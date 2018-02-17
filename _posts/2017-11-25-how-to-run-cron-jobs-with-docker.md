---
layout: post
title: "How to run cron jobs with Docker and AWS ECS"
date: 2018-11-25 20:46:51
image: '/assets/img/'
description:
tags:
- Docker
- cron
- cron jobs
- docker container cron
categories:
- Docker
- AWS
twitter_text:
---

I am big fun of Docker, it makes development fast and reliable. Since I am a big fan, I am trying to use docker whenever I get the chance.
 
I have a many automated jobs running on daily basis. I have used to setup these cron jobs by running ansible scripts against my EC2 instance(s).
 
Lately I have tried to switch these cron jobs to a docker container and run them on ECS service.

`ECS` is an AWS service that let you run your docker containers on EC2 instance. Also they provide another service called `ECR`, which allows you to store your docker images, it is very similar to docker hub.
 
 The setup is fairly easy I will share with you my Docker file, All you have to do, is to change the cron jobs upload your docker image and run it.
 
## Setup
`Dockerfile`

{% highlight docker %}

# Pull ubuntu image
FROM ubuntu:16.04
MAINTAINER ISSAM BEN <bensirhir.issam@gmail.com>

# Update aptitude
RUN apt-get update

# INSTALL syslog for debugging my cron jobs
RUN apt-get install -y rsyslog

# Optional: depending on your environment 
RUN apt-get install -y default-jdk

# Install cron
RUN apt-get install cron

# Build project
WORKDIR /root/

# Copy cron job file to cron directory
COPY my-cron-file /etc/cron.d/my-cron-file

# Give execution rights on the cron job
RUN chmod 0755 /etc/cron.d/my-cron-file

# Run the command on container startup and keep the container running as service
CMD /usr/sbin/cron -f | service rsyslog restart

{% endhighlight %}

<b> Note : </b> Make sure your cron job exists and has all the jobs you want to run

{% highlight bash %}
touch my-cron-file
echo " # This file contains cron jobs that runs every day at 08 o'clock.
00 08 * * 1-7  root your-command" > my-cron-file
{% endhighlight %}

## Build & Push

Now let's build the docker image.

{% highlight bash %}
docker build -t my-cron-container .
{% endhighlight %}

Now the image has been built we have to push it to [ECR](https://aws.amazon.com/ecr/). Assuming you are familiar with AWS. Otherwise you can run the docker image anywhere and you are ready to go.

First Make sure you have AWS cli installed and have your AWS profile setup (check ~/.aws/).

1. Login to ECR service (depending on your region)
{% highlight bash %}
eval $(aws ecr get-login --no-include-email --region us-east-1)
{% endhighlight %}

2. Create your repository
{% highlight bash %}
aws ecr create-repository --repository-name project-x/my-cron
{% endhighlight %}

2. Tag your image so you can push the image to this repository
{% highlight bash %}
docker tag my-cron-container:latest aws_account_id.dkr.ecr.region.amazonaws.com/project-a/my-cron:latest
docker push aws_account_id.dkr.ecr.region.amazonaws.com/project-x/my-cron:latest
{% endhighlight %}

Now your image on ECR, you can easily run it using ECS tasks. I will explain this part in another post.

