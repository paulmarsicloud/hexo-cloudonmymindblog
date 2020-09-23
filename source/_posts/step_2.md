---
title: Step 2 - Setting up Terraform and GitHub
date: 2020-09-22 18:19:38
tags: #aws #terraform #terraformcloud #github
---

This is post will outline the steps taken to setup the cloud infrastructure that will power this website!

## Harness the power of GitHub
I needed a a fresh repo to house my terraform infrastructure code, so I set this up as `terraform-cloudonmymindblog`: https://github.com/paulmarsicloud/terraform-cloudonmymindblog
Some notes on GitHub repo'ing:
* Definitely add a readme - there is an absolutely **brilliant** [article](https://tom.preston-werner.com/2010/08/23/readme-driven-development.html) about why readme's are important by none other than [Tom Preston-Werner](https://twitter.com/mojombo/) (_ya know. . .the co-founder of GitHub and all. . .no big deal_)
* `.gitignore` file - definitely include this file, helps keep the repo clean
* Licensing! Things got interesting for me here. I had a quick gander through https://choosealicense.com/ and decided that the terraform repo in Git will be licensed with MIT (honestly; I am not re-inventing the wheel, and I am happy for folks to use the terraform code even without acknowledging/referencing me), and the git repo that will hold the blog content itself will be licensed with [CC BY 1.0](https://creativecommons.org/licenses/by/1.0/) - which means if someone finds a gold nugget post and wishes to copy it, they can at least acknowledge me.

## Terraform
I then added the S3 bucket and Route 53 scaffolding via terraform files and committed these into the GitHub repo.  Terraform is an amazing cloud-agnostic IaC tool that I have been using for some years now and even though this blog won't really harness the full power/capabilities of Terraform, its still a great tool to use

### Terraform Cloud for IaC CI/CD
So, I had a terraform repo, but now I needed a tool that will push the terraform changes to my AWS environment when merged in successfully.  Enter [Terraform Cloud](https://www.hashicorp.com/products/terraform/)
Terraform Cloud will essentially pull any changes from the git repo and apply them on the infrastructure level; my thinking here is that Terraform Cloud can take care of the CI/CD portion of the Terraform code changes that make their way into the GitHub repo.  I have another tool in mind for the actual blog content updates which I will touch on in the next post!
Here's the steps I took to get the service rolling:
* Created a Terraform Cloud account
* Setup an organization in Terraform Cloud and name it accordingly: I've chosen "thecloudonmymind" as the organization name in Terraform (super original, I know. . . )
* For Workflow; I chose Version Control Workflow as I want any commits to git to push automatically to the remote terraform state
    * For hooking into GitHub - needed to authorize that sucka (disable your ad-blockers to streamline this step!)
    * I then needed to choose a repo - forward planned this with the GitHub repo https://github.com/paulmarsicloud/terraform-cloudonmymindblog created earlier
    * Took a quick spin through the "Configure settings" option - everything looked good, so I went ahead and confirmed to create my workspace
    * Terraform Cloud then checks to make sure everything is kosher, and when it is, you need to configure variables which will authorize Terraform Cloud to access your AWS account via the AWS Access Key ID and AWS Secret Access Key.
    > Note: never, ever, ever expose these keys in a GitHub repo or in the Terraform codebase itself.  It is easy to do so, but you essentially open up the world to accessing your AWS account at that point. . .not a good time.
    * We now **have** to trust Terraform Cloud with these AWS access keys - we are at risk if Terraform Cloud as a service or our account within Terraform Cloud is ever compromised though.  A fair trade off for risk though, as it increases efficiency for CI/CD.
        * You can mark easily these keys/variables as sensitive which is nice as well - no exposure via the api or logs 👍
    * I then queued a plan and let it run!
    * Terraform Cloud will then prompt you for a confirmation if the plan looks gravy - I really like that about the Terraform Cloud interface, I think its super neat and orderly and gives you one more chance to review before saying yes.  You can turn this feature off though and have it auto `terraform apply`; I may/may not change that later if I'm finding the approvals to cumbersome to have to manually approve each time.
        * The **SUPER** neat thing is that the next subsequent PRs that go into the GitHub repo are now also checked by Terraform Cloud (not that I experienced this first hand when I messed up the initial s3 bucket policy and had to recommit so that Terraform Cloud could actually apply the plan. . .)
    * Queued plan ran successfully and I had a Route 53 Hosted Zone and S3 bucket ready to go!

## Recap
* GitHub for Terraform infrastructure repo - rocking and rolling
* Terraform Cloud setup to plan/apply changes that make their way into the GitHub repo for IaC CI/CD
* Terraform Cloud now has programmatic access to my AWS account - I am aware and ok with this being the case.

_Next Post_ : Setting up Hexo and yet another GitHub repo to actually get the site live!