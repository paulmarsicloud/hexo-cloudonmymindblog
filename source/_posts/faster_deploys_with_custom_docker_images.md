---
title: Faster deployments with custom Docker images
date: 2020-09-29 19:23:39
tags: 
- docker
- containers
- hexo
- circleci
- cicd
- github
- node
- aws
---

After reviewing the stack and setup for the blog over the last few days, I noticed something interesting the original CircleCI Config file:
```
version: 2.1
jobs:
  release:
    docker:
      - image: node:lts
    working_directory: ~/hexo-cloudonmymindblog
    steps:
      - checkout
      - run:
          name: Install Hexo CLI
          command: npm install -g hexo-cli
      - run:
          name: Install Hexo Deployer
          command: npm install hexo-deployer-aws-s3 --save
      - restore_cache:
          keys:
            - npm-deps-{{ checksum "package.json" }}
      - run:
          name: Install dependencies
          command: npm install
      - save_cache:
          key: npm-deps-{{ checksum "package.json" }}
          paths:
            - node_modules
      - run:
          name: Force theme download
          command: git clone https://github.com/paulmarsicloud/hexo-theme-cactus themes/cactus
      - run:
          name: Generate static website
          command: hexo generate
      - run:
          name: Install AWS CLI
          command: |
            apt-get update
            apt-get install -y awscli
      - run:
          name: Push to S3 bucket
          command: hexo deploy

workflows:
    version: 2
    build_and_deploy:
      jobs:
        - release
```

Basically, the number of commands to build and deploy the server were kinda. . . long.  Both in just the number of commands, but also in the deployment times:
![previous deployment times...not bad, but could be improved](/images/1_min_deploys.png)

Now, ~1 minute deployments to the S3 bucket are perfectly fine for the use case of this blog.  However, for fun, I thought it would be neat to see if I could get that time down.  Enter. . . Docker

## Docker
> Note; I won't go into detail about [Docker](https://www.docker.com/) and containerization as that's outside the scope of this blog.  But images and Containers are our present and future, and we all need to be comfortable with them 🙂

So CircleCI spins up a Docker image and then runs the commands from the config and pushes the code to S3.  To make the code push complete faster, I thought "why not have all the commands pre-ran/built in Docker?".  

As a good technologist, rahter than build something entirely from scratch, I looked to see if there were any Docker images that already had Hexo pre-installed.  I found this awesome [image](https://hub.docker.com/r/spurin/hexo) from [James Spurin](https://github.com/spurin) and read this great [article](https://spurin.com/2020/01/04/Creating-a-Blog-Website-with-Docker-Hexo-Github-Free-Hosting-and-HTTPS/) as well, which put the wheels in motion for my next move.  I could have easily used the Spurin Hexo Docker image, but the use-case for that image is clearly more to build the Hexo website _within_ a Docker Container, whereas my use-case was to use a Docker Container to build **and deploy** the Hexo website.

# Building a "custom" Docker Container
After taking a look at the CircleCI config file, I realized that I need the Docker image to simply do the following:
* Install Hexo
* Download the [GitHub](https://github.com/paulmarsicloud/hexo-cloudonmymindblog) repo
* Generate the thecloudonmymind.com blog
* Push the generated website to AWS S3

I thought it would be a good opportunity to learn a little more Docker and spin up an image myself that does just that.  So after a fair bit of testing on machine, I came up with this [Docker Container](https://hub.docker.com/r/cloudonmymind/hexo-aws)!

To get this rolling, I had to perform the following:
* Setup a [Docker Hub Account](https://hub.docker.com/u/paulmarsicloud)
* Create a [Docker Hub Organization](https://hub.docker.com/r/cloudonmymind/) to house the Container
* Build the Docker image locally based on this [repo](https://github.com/cloudonmymind/docker-hexo-aws-base) I created.
* Tag and push the Docker Container to Docker Hub

The Dockerfile in that repo shows the components that make up and essentially "pre-load" the Container for CircleCI to use.  My initial thoughts were that a Container could house the GitHub codebase too, but then for **every commit** to the [blog repo](https://github.com/paulmarsicloud/hexo-cloudonmymindblog) I would need to re-build the Docker Container so that it has the latest changes on the Container. . .not super efficient and may make the Container startup time longer. . .

I also learned that in order to authorize Docker Hub to build the Containers you should do this within a GitHub organization (so as to not expose your private and public GitHub repos to Docker Hub).  So I setup a [Docker Hub Organization](https://hub.docker.com/r/cloudonmymind/) and [Git Hub Organization](https://github.com/cloudonmymind) based on this blog, and we'll see where the future take us!

With the new pre-installed Hexo with AWS CLI Container now being avaiable, the CircleCI Config file now looks like this:
```
version: 2.1
jobs:
  release:
    docker:
      - image: paulmarsicloud/hexo-aws:1.5
    working_directory: ~/hexo-cloudonmymindblog
    steps:
      - checkout
      - run:
          name: Force theme download
          command: git clone https://github.com/paulmarsicloud/hexo-theme-cactus themes/cactus
      - run:
          name: Generate static website
          command: hexo generate
      - run:
          name: Push to S3 bucket
          command: cd public/ && aws s3 sync . s3://www.thecloudonmymind.com

workflows:
    version: 2
    build_and_deploy:
      jobs:
        - release
```

And now takes around ~30 seconds to deploy. . .not too bad and definitely an improvement! 💪


## Recap
* Built a custom pre-installed Hexo and AWS CLI Docker Container
* Took the CircleCI config steps from 9 commands to 3 commands
* Improved the website deployment from ~1 minute to ~30 seconds!

