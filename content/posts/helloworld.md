+++
title = "Whynot Blog: Hello AWS and Hugo!"
date = "2021-08-01T16:25:47+02:00"
author = "Andreas Skoglund"
authorTwitter = "" #do not include @
cover = ""
tags = ["blog", "hugo", "aws", "terraform", "s3", "cloudfront", "serverless"]
keywords = ["key", "word"]
description = "AWS & Hugo"
showFullContent = false
+++

## Hello world, and welcome to this brand new blog!
I made this site to act as a playground for some various ideas, and as a place to post some of my projects. First up is this blog itself, because *why not*?

This blog is set up with Terraform on AWS, and uses Hugo to generate static content which gets deployed on S3.

A detailed guide on how Terraform was used can be found here: [HowTo: Making a static webpage like this blog](/posts/making-a-static-website/)

### Hugo 
[Hugo](https://gohugo.io/) is, according to their creators: 
> *"... one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again."*. 

Which seems to be about right so far. This is the first time I've used any sort of static page generator, and I selected Hugo on a whim, no real reason. I also found the [Minimal theme](https://github.com/calintat/minimal) to be quite nice, but I've done some slight modifications on it to make things more functional for my purpose. 

For those unfamiliar with the term, a "static page generator" is a toolset to pre-generate web-pages to be used without any serverside processing like PHP, Django, etc. Some of you might have known about the "Save as Webpage" option in MS Word, this is similar, just that it's usable. 

The Hugo config I've ended up with so far, and all blog pages can be found here: https://github.com/Anderen2/whynot-blog

### Amazon Web Services
To provide the blog a home I selected AWS this time. However I'm likely to experiment around with other providers too, just to get an idea on how they compare. A simplified graph showing the AWS products I've used, and their relationships may be viewed below. 

{{< figure src="/whynot-aws.svg" >}}

As seen, the architecture is quite simple, but it allows for a quite low-cost and performant website. 

#### Terraform
To deploy the AWS products I've used [Terraform](https://www.terraform.io/), which is a piece of software which allows you to define your infrastructure as code. 

The entire Terraform infrastructure-as-code (IaC) for this very blog may be found here: https://github.com/Anderen2/blog-s3

I have a detailed write-up on how you can create your own blog-page in the same manner here: [HowTo: Making a static webpage like this blog](/posts/making-a-static-website/)

### So far ...

- The "serverless" nature of the solution has worked great in conjunction with Hugo, it's easy to deploy, no lost sleep on patching, and is cost effective. 
- Hugo was easy to get started with and seems like a great choice for simple blogs, writing pages in Markdown and not fighting a janky WYSIWYG is (for me) a joy.
- Using AWS with Terraform is much easier to work with than hunting around in the AWS-console, I love the feeling when you can tear things down and set it back up again with just a few simple commands. 
