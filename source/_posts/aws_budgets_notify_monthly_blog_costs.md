---
title: AWS Budgets to notify montly blog costs
date: 2021-01-18 23:07:55
tags: 
- aws
- awsbudgets
- cloudcosts
- terraform
- iam
---

It has been a few weeks since I have made any changes or wired up a blog post, but this has been one that is not only everyone's favourite topic but one I have been meaning to do for a while: AWS Budgets!

## AWS Budgets

If you have been involved with AWS or Cloud computing, you will have a story of your own or a scary anecdote about a very high unexpected bill.  I obviously do not want a high unexpected bill, but I am confident that my lightweight, server-less blog shouldn't cost me anymore than $5-6 a month anyways.  This is a good learning tool for working with the AWS Budgets offering, which I have limited exposure to, but it is kinda cool that you can create 2 AWS Budgets for free, which is perfect for my blog.  All the details for the service are [here](https://aws.amazon.com/aws-cost-management/aws-budgets/)

## IAM
Before you begin, be sure to give [this](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html) and [this](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step1) a solid read.  You need to activate IAM Access in AWS Budgets so that non-root user accounts can access AWS Budgets - even if the users have the correct IAM permissions, this area must be activated first!  

Before enabling it will look like this:

![IAM No Access-y](/images/IAM_Not_Enabled.png)

After enablement:

![IAM has the power!](/images/IAM_Budget_Activated.png)

## Terraform

With this in mind (and with a lot of help from [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/budgets_budget)) and keeping everything IaC related, this was how I setup my AWS Budget and notification:

```
resource "aws_budgets_budget" "blog_cost" {
  name              = "montly-blog-cost"
  budget_type       = "COST"
  limit_amount      = "7.50"
  limit_unit        = "USD"
  time_period_end   = "2087-06-15_00:00"
  time_period_start = "2020-01-01_00:00"
  time_unit         = "MONTHLY"

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "FORECASTED"
    subscriber_email_addresses = ["paulmarsicloud@gmail.com"]
  }
}
```

As you can see above, I've kept the budget very simple:
$7.50 a month for **total** costs
Notify when the **forecasted percentage** is **greater than** 80% (so $6)

And yes I know - email is the death of communication, but I might setup SNS to my phone or something in a later post 🙂

Now the AWS Budgets section in the Console reflects this nicely:

![Let there be budgets!](/images/AWS_Budgets.png)

I am confident this will work well as some small insurance just incase the monthly blog bill is anything higher than usual - which for the past few months has been around $4 CAD, so I'll wait with baited breath to see if I am ever alerted for a crazy bill forecast. I will welcome the notification!

## Recap
* AWS Budgets will help notify you if you are about to break your budget!
* You must activate IAM access for AWS Budgets first!
* The AWS Budget can be controlled in Terraform!