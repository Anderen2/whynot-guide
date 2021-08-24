+++
title = "HowTo: Making a static webpage like this blog"
date = "2021-08-01T16:25:47+02:00"
author = "Andreas Skoglund"
authorTwitter = "" #do not include @
cover = ""
tags = ["HowTos","blog", "aws", "terraform", "s3", "cloudfront", "serverless"]
keywords = ["key", "word"]
description = "Using Terraform and AWS to create a easy to maintain and low-cost website"
showFullContent = false
+++

Making of the blog
===================
So, building on the previous entry here: [Whynot Blog: Hello AWS and Hugo!](/posts/helloworld/), lets explain and deepdive into how this Terraform-magic actually works. 

First of all, everything used for creating this blog can be found here: [github.com/anderen2/blog-s3](https://github.com/Anderen2/blog-s3/tree/5ab4b563f227d099f7a83d3d75b27bba96f8b436). If you simply want to make a quick and dirty copy, feel free to do so. All you need to change is [line 3 in variables.tf](https://github.com/Anderen2/blog-s3/blob/main/variables.tf#L3) to match your own domain name, then come back for the deploy-part further down on this very page. 

> NB: If you find me doing something weird/unnecessary or outright stupid in my Terraform code then you are very welcome to point it out by [creating a issue](https://github.com/Anderen2/blog-s3/issues/new?assignees=Anderen2&labels=&template=somethingweird.md&title=Hey+Anderen2%21+you+did+something+stupid.+). There's always more to learn. 

## Behind the magic
To start out, lets explain what Terraform actually does in short terms. 
- It allows you to write how you want things to be set up, not as filling in small textboxes between hundreds of mouseclicks in the AWS console, nor as a bunch of seperate commands, but more like a script or a document. Not unlike how Puppet or Ansible works if you are familiar with those tools. This is also often referred to as **IaC (Infrastructure-as-Code)**.
- You write this IaC in the HCL-language, a domain-specific language claiming to be ["*... structured configuration languages that are both human- and machine-friendly, for use with command-line tools*"](https://github.com/hashicorp/hcl/blob/main/README.md). For a script-ish language inspired by the Nginx config, it's actually suprisingly easy to get into. 
- It uses the HCL you write to calculate a "desired state" of your solution, and calls on the API's of the provider (in this case, AWS) to get the actual state. Then it attempts to figure out how to get those two to match the best it can. This is called the *"plan"* stage. 
- If you were satisfied with whatever plan it churned out, then you can *"apply"* the plan, making your IaC lose two letters and become actual infrastructure, using the providers API's again to realize it. 

That was somewhat short at least. 

{{< figure src="/terraform-work-cycle.svg" >}}
*A normal Terraform workflow*

## Diving into it
This will be the long part, I'll here attempt to explain each section of the Terraform code, file by file, starting with main.tf. I'll be commenting all the lines of interest in the code-sniplets too. 

This walkthrough assumes that you have [configured your AWS CLI](https://aws.amazon.com/cli/), and checked that it does work. 

Terraform can be downloaded for free from https://www.terraform.io/downloads.html

**Note**, none of the following filenames are magical and can be named whatever you'd prefer, you can even keep everything in a singular file if you'd really want to. 

### main.tf
This file is quite short, and consists of three sections. 

The first section tells terraform which providers it should use (in our case AWS, but it could very well be Azure or any other provider too.), as well as which version it should use. 
```hcl
terraform {
  required_providers {
    aws = { # The provider we want to use
      source  = "hashicorp/aws" # Where to fetch the provider 
      # see also: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
      version = "~> 3.27" # Which version we want to lock into
    }
  }

  required_version = ">= 0.14.9" # Required Terraform version (Later, or equal to 0.14.9 in this case)
}

```

The next section defines the AWS provider that we want to use with some essential information. Note! this is where you specify the AWS region you want to use. 
```hcl
provider "aws" {
  profile = "default" # Which AWS CLI profile to use, normally default. 
  region  = "us-east-1" # Which region we want to provision to
}
```

### output.tf
The output.tf file contains "output" statements. These will cause Terraform to provide some extra information at the end of a "terraform apply", like so: 
```shell
~/blog-s3$ ~/bin/terraform apply
aws_acm_certificate.cert: Refreshing state... 
aws_route53_zone.main: Refreshing state... 
aws_s3_bucket.www: Refreshing state... 
aws_s3_bucket.site: Refreshing state... 
aws_route53_record.cert_validation: Refreshing state... 
aws_acm_certificate_validation.cert: Refreshing state... 
aws_route53_record.www: Refreshing state... 
aws_cloudfront_distribution.main: Refreshing state... 
aws_route53_record.main: Refreshing state... 

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

website_endpoint = "whynot.guide.s3-website-us-east-1.amazonaws.com"
```

I found that the direct link to the S3 bucket was nice for troubleshooting at times (especially when waiting for DNS.)

Any variable can be output here, so feel free to add more if you want to know the contents of any. 
```hcl
output "website_endpoint" {
  value = aws_s3_bucket.site.website_endpoint
}
```

### variables.tf
Here I've specified a single variable, in our case this should match the domain-name of your page. We will be using this variable in multiple different places as you will see. 
```hcl
variable "site_name" {
  type = string
  default = "whynot.guide"
}
```

### s3.tf
So, where in this "serverless" world of ours, does our actual site exist? The answer is on S3. AWS S3 is a so-called object-storage, meaning that the everything is stored as "objects". alexwlchan has a great article on this if you are curious about what seperates that from normal file-storage: [S3 keys are not file paths](https://alexwlchan.net/2020/08/s3-keys-are-not-file-paths/). Anyway, I digress, back to Terraform!

So in s3.tf we create our S3-buckets which will contain all the files we want to host on our website. The first section here creates the main site bucket that will store this. 
```hcl
resource "aws_s3_bucket" "site" {
  # This will be the name of the bucket, and should match the domain-name of your site. Note that this has to be world unique!
  bucket = var.site_name 

  # The ACL to use within AWS, we have no need to share this with other AWS accounts therefore the ACL is private
  acl    = "private" 

  # The access-policy to use for this bucket, we will see this again two sections down. 
  policy = data.aws_iam_policy_document.bucket_policy.json 

  # This allows Terraform to DELETE ALL FILES in your bucket at will. This is necessary for some scenarios, but be sure that you are not storing anything you cannot afford to lose when using this. 
  force_destroy = true 

  # This allows the S3 bucket to be used as a website, which is exactly what we want. 
  website { 
    index_document = "index.html"
    error_document = "404.html"
  }

  # Not required, but makes it easier to view in the AWS console. 
  tags = { 
    "Name" = var.site_name
  }
  
}
```

Next up is a nice-to-have function, this allows old-fashion visitors that expect websites to be accessed with www.* to get pushed along to the correct place. 
```hcl
resource "aws_s3_bucket" "www" {
  # Our website domain, prefixed with www.
  bucket = "www.${var.site_name}"

  # Identical to our main site
  acl    = "private"

  # No policy attached, as there are no objects in this bucket that we want to serve. 
  policy = ""
  force_destroy = true

  # This causes all web-requests to this bucket to be redirected to our main domain, without www.
  website {
    redirect_all_requests_to = "https://${var.site_name}"
  }

  tags = {
    "Name" = "www.${var.site_name}"
  }
}
```

Last up is the policy we saw on the bucket for our main site. This is whats allowing strangers to look into our bucket and fetch objects. Without this our page would simply return a sad "403 Forbidden". 
```hcl
data "aws_iam_policy_document" "bucket_policy" {
  statement {
    # The Sid (statement ID) is an identifier for our policy.
    sid = "AllowReadFromAll"

    # What actions we want to allow
    actions = [
      # We only want to allow strangers to fetch objects
      "s3:GetObject", 
    ]

    # Which AWS resources this should affect
    resources = [
      # We only want this to affect our main site bucket
      "arn:aws:s3:::${var.site_name}/*",
    ]

    # As we want non-AWS users to access this, we must allow public access
    principals {
      type        = "*"
      identifiers = ["*"]
    }
  }
}
```

### cloudfront.tf
To allow for using SSL (https) with a custom domain, we have to use Cloudfront in-front of our S3 bucket. Cloudfront is a Content Delivery Network (CDN) which helps you accelerate your website by caching your website at several edge-locations around the world. 
```hcl
resource "aws_cloudfront_distribution" "main" {
  # Whether or not our CF should accept requests or not, which we definitly want. 
  enabled             = true

  # Which object to return if the end user requests https://ourdomain.com
  default_root_object = "index.html"

  # What price class we want to limit ourselves to, this may affect performance in some regions. 
  # See: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PriceClass.html
  price_class = "PriceClass_100"

  # Which domains this distribution covers, in our case it's only the base domain. 
  aliases = concat([var.site_name])

  # What which backs this CF distribution
  origin {
    # A unique identifier for the origin
    origin_id   = "origin-${var.site_name}"

    # The domain name of our S3 bucket that we want Cloudfront to host
    domain_name = aws_s3_bucket.site.website_endpoint

    # Origin specific configuration. 
    custom_origin_config {
      # As mentioned, S3 only supports http traffic, therefore we need to set http-only here. This only affect traffic between Cloudfront and S3.
      origin_protocol_policy = "http-only"
      http_port              = "80"

      # These options are required, but not in effect as we've set "http-only"
      https_port             = "443"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  # Whether or not we want to restrict our website to a certain geographical location. Maybe exclude the pesky norwegians? 
  restrictions {
    geo_restriction {
      # We want our page to be viewable from the entire world, so no restriction here. 
      restriction_type = "none"
    }
  }

  # Cache configuration
  default_cache_behavior {
    # The Origin we want to target, see "origin_id" above. 
    target_origin_id = "origin-${var.site_name}"

    # Which HTTP methods we want to allow. We only need "GET" and "HEAD" in our case. 
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]

    # Whether or not we want to automatically compress pages for clients that request it. 
    compress         = true

    # Which parts of the client request we want to forward along to S3 when querying. 
    forwarded_values {
      # We have no need for query strings for this setup
      query_string = false

      # Nor do we use any cookies
      cookies {
        forward = "none"
      }
    }

    # Whether we allow non-https clients, or if we want to redirect them. In this case we want to redirect them. 
    viewer_protocol_policy = "redirect-to-https"

    # How long a page on your site should be cached before refreshed. If you want to view changes closer to real time, you might want these low.
    # More details on https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Expiration.html
    min_ttl                = 0
    default_ttl            = 300
    max_ttl                = 1200
  }

  # SSL certificate configuration
  viewer_certificate {
    # The AWS ID of the certificate we want to use, see acm.tf
    acm_certificate_arn      = "${aws_acm_certificate_validation.cert.certificate_arn}"

    # In what way we want to serve HTTPS, sni-only or vip. We have no need for anything other than sni-only. 
    ssl_support_method       = "sni-only"

    # Which minimum SSL/TLS protocol version we want to support. 
    minimum_protocol_version = "TLSv1"
  }
}


```

### r53.tf
Next up is the DNS setup, one thing to note here is that I bought my domain-name outside of AWS. You can also buy your domain in AWS, however this has to be done outside of Terraform. 

Back to r53.tf, first thing we need to do is to create our DNS Zone.
```hcl
resource "aws_route53_zone" "main" {
  name = "${var.site_name}" # Note that we use the variable we specified earlier here!
}
```

Next up is the records for our little website:
```hcl
resource "aws_route53_record" "main" {
  # The zone we want this record to live under, see above
  zone_id = "${aws_route53_zone.main.zone_id}"

  # The name of the record, in this case it's for the base domain
  name    = "${var.site_name}"

  # The type of record, we want this to point to an IP, therefore A
  type    = "A"

  # This causes the record to be an "alias-record". 
  # An alias-record is a AWS-specific functionality, and allows you to easier connect the records to AWS services. 
  # See also: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html
  alias {
    # We want this to point towards our Cloudfront distribution
    name                   = "${aws_cloudfront_distribution.main.domain_name}"
    zone_id                = "${aws_cloudfront_distribution.main.hosted_zone_id}"

    # No need to evaluate the health of the endpoint, if Cloudfront is down we are too anyway. 
    evaluate_target_health = false
  }
}

resource "aws_route53_record" "www" {
  zone_id = "${aws_route53_zone.main.zone_id}"
  # This would be the record for the www.* subdomain
  name    = "www.${var.site_name}"

  # I noticed a quirk here. 
  # When using A-record the Terraform Apply works and the record gets seemingly provisioned successfully, but www.whynot.guide never resolves. Only seems to affect alias-records towards S3. 
  # I've therefore set this to CNAME, which makes the record point to a the S3 record instead. 
  type    = "CNAME"

  alias {
    # We want this to point towards our www-redirect S3 bucket directly. 
    name                   = "${aws_s3_bucket.www.website_endpoint}"
    zone_id                = "${aws_s3_bucket.www.hosted_zone_id}"
    evaluate_target_health = false
  }
}
```

Last record to configure is a special one. This is a "certificate validation record", which we need to be able to use Amazon Certificate Manager to get a SSL certificate for our page. 
```hcl
resource "aws_route53_record" "cert_validation" {
  # All of these parameters are fetched from our certificate that we will generate with acm.tf
  name    = "${tolist(aws_acm_certificate.cert.domain_validation_options).0.resource_record_name}"
  type    = "${tolist(aws_acm_certificate.cert.domain_validation_options).0.resource_record_type}"
  zone_id = "${aws_route53_zone.main.id}"
  records = ["${tolist(aws_acm_certificate.cert.domain_validation_options).0.resource_record_value}"]
  ttl     = 60
}
```

### acm.tf
Here we will configure our free SSL certificate that we get through Amazon Certificate Manager (ACM) for our domain. 
```hcl
# This will define our certificate
resource "aws_acm_certificate" "cert" {
  # Our main domain name
  domain_name       = "${var.site_name}"

  # As to ensure that we own the domain that the certificate is for, we need to validate it in some way. 
  # The easiest here is to use DNS as a validation method, which is what we configured in the last part of r53.tf
  validation_method = "DNS"
}

# This resource is not strictly necessary in all cases, and is not actually creating anything.
# It simply makes Terraform wait until the certificate validation has succeeded. 
# See also https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate_validation
resource "aws_acm_certificate_validation" "cert" {
  certificate_arn         = "${aws_acm_certificate.cert.arn}"
  validation_record_fqdns = ["${aws_route53_record.cert_validation.fqdn}"]
}

```

## Deploy!
Finally, we can attempt to deploy! 

While in the directory containing the .tf files, run `terraform apply`. This should return quite a lot of information on what Terraform plans to perform, do a quick readthrough to ensure that it seems some-what sane, and if so press `y` and enter to start provisioning the solution. 

**Note!** You might get an error as the ACM DNS certificate validation fails, this happens if you've yet to point your domain towards AWS' nameservers. To fix this, you need find the list of AWS nameservers on your domain here: https://console.aws.amazon.com/route53/v2/hostedzones#ListRecordSets/ and change your nameservers (NS-records) at your registrar to match those. When that is done, you may simply re-run `terraform apply` to continue the provisioning. 

## Adding some content
So, we now have a website, but as we haven't uploaded any content yet it might be somewhat lame. Lets fix that!

We can upload content to the website by putting it in the bucket named after our domain. This can be done via the AWS web-interface by navigating to https://s3.console.aws.amazon.com/s3/home. Or even better, we can use the AWS CLI like so:
```shell
# To upload a single file
aws s3 cp <file> s3://<domain>/

# Example:
whynot-guide$ aws s3 cp index.html s3://whynot.guide/ 
upload: ./index.html to s3://whynot.guide/index.html

# To upload an entire folder, and its subfolders
aws s3 cp <folder> s3://<domain>/ --recursive

# Example:
whynot-guide$ aws s3 cp ./public/ s3://whynot.guide/ --recursive
upload: public/css/main.css to s3://whynot.guide/css/main.css    
upload: public/categories/index.xml to s3://whynot.guide/categories/index.xml
upload: public/404.html to s3://whynot.guide/404.html            
upload: public/index.html to s3://whynot.guide/index.html          
upload: public/index.xml to s3://whynot.guide/index.xml            
upload: public/page/1/index.html to s3://whynot.guide/page/1/index.html
...
```

And there we have it, you now have a simple static webpage to play with! 🎉

**Note!** Depending on the TTL-settings you set in cloudfront.tf, it might take some time before the content you uploaded actually appears on the page. 

## Known issues / exercises for the reader
No good guide without some thought-exercise for the reader ;) 

#### Https on www.*
You might have already noticed, but `https://www.<domain-here>` does not work, this may or may not be a problem for you. The reason for this is that we only used a S3-bucket for our redirect, and S3 does not support SSL/HTTPS. This can easily be fixed by creating another Cloudfront distribution to cover it. 

#### ACM DNS certificate validation fails on first apply
As mentioned in the note under `Deploy!`, our first round of `terraform apply` has a little catch-22. We cannot move our domain before the zone is created because then we don't know which nameservers to use, but if we don't the apply will fail because the validation step in ACM expects it to be done. We can avoid this by removing the "cert_validation" resource from acm.tf, or we could restructure our code to work in a two-stage approach. 

## Conclusion
We made it to the end at last, *puh*. I hope that this was of help to you, the reader, in some way. However, if you find yourself totally stuck, or you have a question/feedback on something here, feel free to send me an email @ andreas@whynot.guide. 

Arrivederci!