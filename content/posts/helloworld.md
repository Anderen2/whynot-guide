+++
title = "Whynot Blog: Hello AWS and Hugo!"
date = "2021-08-01T16:25:47+02:00"
author = "Andreas Skoglund"
authorTwitter = "" #do not include @
cover = ""
tags = ["blog", "hugo", "aws", "terraform", "s3", "cloudfront"]
keywords = ["key", "word"]
description = "AWS & Hugo"
showFullContent = false
+++

## Hello world, and welcome to this brand new blog!
This site will act as my playground for various ideas, and a place to post some of my projects. First up is this blog itself, because why not?

This blog is set up with Terraform on AWS, and uses Hugo to generate static content which gets deployed on S3. 

### Hugo 
[Hugo](https://gohugo.io/) is, according to their creators: 
> *"... one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again."*. 

Which seems to be about right so far. This is the first time I've used any sort of static page generator, and I selected Hugo on a whim, no real reason. I found the [Minimal theme](https://github.com/calintat/minimal) to be quite nice, but I've already done some slight modifications on it to make things more functional for my purpose. 

The Hugo config I've ended up with so far, and all blog pages can be found here: https://github.com/Anderen2/whynot-blog

### Amazon Web Services
To provide the blog a home, I've selected AWS this time. However I'm likely to experiment around with other providers too, just to get an idea on how they compare. A simplified graph showing the AWS products I've used, and their relationships may be viewed below. 

{{< figure src="/whynot-aws.svg" >}}

As seen, the architecture is quite simple, but it allows for a quite low-cost and performant website. 

#### Terraform
To deploy the AWS products I've used [Terraform](https://www.terraform.io/), which is a piece of software which allows you to define your infrastructure as code. 

The entire Terraform infrastructure-as-code (IaC) for this very blog may be found here: https://github.com/Anderen2/blog-s3

### So far ...

- Using Hugo seems like a great choice for simple blogs, writing pages in Markdown and not fighting a janky WYSIWYG is for me a joy.
- AWS with Terraform has worked great, it's much easier to work with AWS when you can tear things down and set it back up again with just a few simple commands. 

I'll do a more detailed write-up on how you can create your own blog-page in this manner, and how it measures in cost later on. 