---
title: Step 3 / 3 - Setting up Hexo and pushing to S3 using CircleCI
date: 2020-09-23 12:41:35
tags: 
- aws
- hexo
- circleci
- github
- route53
- s3
---

I thought it apt to delve deeper into the reasoning behind the _why_ of this blog in this post 🙂

After a bit of a _"penny drop"_ moment in my career; I feel it appropriate to start putting some time and effort into blogging as it helps reinforce the lessons learned along my cloud journey and it may help others as well!  As a cloud/tech professional, practitioner and enthusiast; what better way than to share some thoughts and learnings than through a Server-less blog!

> A note on Server-less; I am a huge fan and proponent of Server-less.  I think there are far too many services that currently are not Server-less that definitely should be Server-less.  I feel it helps emphasize the "pay as you go" Cloud model with its nature, and it is an area that I hope to become an expert in!

## Hexo
So then, why Hexo?
Truthfully, I wanted to up my node.js game and experience and looking through some "Server-less blog" entries, there are quite nice templates on Hexo.
> Note: there is no way I'm dedicating time and effort to creating my own, personal, custom Hexo theme.  I will leave that to the design experts - I am after all a Cloud Engineer, not a Front-End Engineer. . .but I'll make it look nice and will credit accordingly!

I did a brief comparison of [Ghost](https://ghost.org/), [Jekyll](https://jekyllrb.com/), [Hugo](https://gohugo.io/) and [Hexo](https://hexo.io/), but honestly after looking through the themes for each, I really dug this [theme](https://github.com/V-Vincen/hexo-theme-livemylife) and that's kinda what made the decision for me.  I am not opposed to changing up the blog in the future to learn a different tool; all four will play nicely with the Server-less design anyway.

Also - Hexo is **damn** quick to setup and get going!

### How to Hexo
* Follow ze steps [here](https://hexo.io/) to get your environment setup.
* As I said earlier, the theme for this site was intended to be [this](https://github.com/V-Vincen/hexo-theme-livemylife) - this theme has a legit readme that helps spell out how to setup a proper looking blog.  However, after toying with it locally; it felt a little too much like an old school Wordpress site that just wasn't jiving with me.
* I then decided to go with this [cactcus theme](https://github.com/probberechts/hexo-theme-cactus) for the blog - again, another straightforward readme, and a design that definitely is a bit more. . .me 🙂

Setting up a blog locally in Hexo is very easy and straightforward, but now it was about configuring the theme to my liking!
* Honestly, it took me a few good hours of playing around to get the hang of the theme and feel; I ended up forking this repo to [here](https://github.com/paulmarsicloud/hexo-cloudonmymindblog) so that I could truly customize this.
* After previewing the blog locally, my theme changes were to my liking, so now I just had to push this to S3.

Enter CircleCI

## CircleCI
After reading this cool [artcile](https://hackernoon.com/build-a-serverless-production-ready-blog-b1583c0a5ac2) by [Mohamed Labouardy](https://twitter.com/mlabouardy) I thought setting up CircleCI would be an excellent candidate to push the Hexo repo code and blog posts to my S3 bucket.  This was an easy setup:
* Signed up for a free CircleCI account with my GitHub account
* Authorized CircleCI to have access to the [hexo repo](https://github.com/paulmarsicloud/hexo-cloudonmymindblog)
* Added a similar `config.yml` file from [here](https://github.com/slow-coder/slowcoder.com/blob/master/.circleci/config.yml)
* Added my AWS programmatic keys to my CircleCI settings
> Note - like with Terraform Cloud, CircleCI has access to my AWS account.  I am aware of the risks that CircleCI or my account on CircleCI could become compromised, but it is a trade-off I am willing to make
* Here's where things got interesting. . .The [`config.yml`](https://github.com/slow-coder/slowcoder.com/blob/master/.circleci/config.yml) here was erroring on upload to my S3 bucket with a `seek() takes 2 positional arguments but 3 were given` error.
    * This was amusing, and after a little bit of review, I decided to have CircleCI call run a **Hexo** command to push the files to S3, rather than an AWS S3 command.  This then required a few minor changes to the [hexo repo](https://github.com/paulmarsicloud/hexo-cloudonmymindblog) but after some testing, the code is now being pushed via CircleCI!
* CircleCI is super neat because it pushes the changes quick and it pushes based on PR - Terraform Cloud you need to merge the change in before can push the infrastructure change, so its a teeny tiny bit more cumbersome

## Recap
* Setup Hexo on my local machine; super lightweight and got the feel of it pretty quickly
* Pushed the local code to a fresh [repo](https://github.com/paulmarsicloud/hexo-cloudonmymindblog) in GitHub - this will be used lots going forward
* Setup CirclCI to be the de-facto CI/CD pipleline for new blog content changes in future