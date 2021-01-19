---
title: Certificates for your S3 powered blog
date: 2020-10-30 19:00:06
tags:
- s3
- certificates
- hexo
- acm
- cloudfront
- https
- route53
- aws
- terraform
---

So the blog is looking pretty snazzy to, but there is one elephant in the url I wanted to address:

![connection looking insecure](/images/why_no_https.png)

## HTTPS

This [article](https://web.dev/why-https-matters/) by [Kayce Basques](https://twitter.com/kaycebasques) sums up perfectly *why* you want HTTPS everywhere.  For me it was really a no-brainer; I want the blog to be secure and I want the blog to look professional and not deter any folks reading the content.

So in order to HTTPS, we'll need a certificate.  Enter Amazon Certificate Manager

## Amazon Certificate Manager

Amazon's native certificate manager service is super nice to use with other AWS resources (as most AWS services tend to be), but I have also found it **very** straightforward to create a certificate and associate it to a resource.  Gone are days of manual handcrafted certificate creations that always have _something_ wrong with them (I, for one, do not miss those days).

The certificate is simple to setup in terraform using the following: 

```
resource "aws_acm_certificate" "www-certificate" {
  domain_name       = "thecloudonmymind.com"
  validation_method = "DNS"
  subject_alternative_names = ["www.thecloudonmymind.com", "*.thecloudonmymind.com"]
}
```

This [article](https://medium.com/runatlantis/hosting-our-static-site-over-ssl-with-s3-acm-cloudfront-and-terraform-513b799aec0f) by [Luke Kysow](https://twitter.com/lkysow) goes over most steps nicely - which is basically using Terraform to create the cert and using CloudFront to serve the HTTPS (and as such the certificate) to the users.

## CloudFront

Unfortunately, due to hosting a static site on S3, you **must** use a CloudFront distribution in order to serve the site with https **and** with your custom domain name.  You can serve the site with https using the AWS S3 generated website end-point, but where's the fun/learning in that!

So I needed a CloudFront distribution which was wired up in terraform with [this](https://github.com/paulmarsicloud/terraform-cloudonmymindblog/blob/236618caab7b8dbeda237d762548af44463fe5fb/cloudfront.tf#L1-L46)

The key portion of the code is actually this:

```
  viewer_certificate {
    acm_certificate_arn = aws_acm_certificate.www-certificate.arn
    ssl_support_method  = "sni-only"
  }
```

This is where the viewer certificate is associated with the CloudFront distribution.  After that, the final update was to redirect the Route 53 A record from the S3 end-point to the CloudFront end-point, so that the certificate can be referenced in the browser request:

```
resource "aws_route53_record" "blog_www_zone" {
  zone_id = aws_route53_zone.blog_zone.zone_id
  name    = "www.thecloudonmymind.com"
  type    = "A"
  alias {
    name                   = aws_cloudfront_distribution.www_distribution.domain_name
    zone_id                = aws_cloudfront_distribution.www_distribution.hosted_zone_id
    evaluate_target_health = false
  }
  depends_on = [aws_s3_bucket.www_bucket]
}
```

Now, the certificate serves the request via https and lo and behold:

![nice looking cert there!](/images/https_is_achieved.png)

The blog is secure yet again!

There are some extra neat terraform tricks in the [repo](https://github.com/paulmarsicloud/terraform-cloudonmymindblog) to "validate" the certificate in AWS and to have both www and the * top level FQDN references be requested with https, there must be a second cloudfront distribution.  All things that can be found in the repo 🙂

## Recap
* No HTTPS for custom domain names for S3 website hosting - this is by design
* ACM to create a certificate
* Cloudfront to _serve_ the certificate for your S3 website with a custom domain name 💪
