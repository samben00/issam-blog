---
layout: post
title: "How to setup up AWS CloudWatch Logs"
date: 2017-05-01 14:56:42
image: '/assets/img/'
description:
comments: true
tags:
- CloudWatch
- AWS
- Logs
- CloudWatch Logs
- AWS CloudWatch
categories: 
- aws
twitter_text:
---

I find AWS CloudWatch very useful when it comes to monitor system/application logs. CloudWatch service provides a friendly UI to search inside your logs. Also it offers very flexible way to select logs of specific date or date range.

## What I m trying to do

I will try to explain how it s easy to install and configure AWS CloudWatch on my EC2 instance and create Alarms on the logs.


## Enviroment 
- EC2 instance running ubuntu machine 16.04

## Prerequisite 
- [awscli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) must be installed and configured.


## Basic Steps
1. Download and Install aws CloudWatch agent on our machine
2. Setup the Cloudwatch Agent
3. Configure an alarm

## Configuration

### 1. Instalation

Amazon makes the installation of the CloudWatch agent very easy. 
{% highlight bash %}
wget https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py
{% endhighlight %}

{% highlight bash %}
python awslogs-agent-setup.py -n -r MY_REGION
{% endhighlight %}

MY_REGION : is the region of your running EC2 instance ( for exmaple : us-east-1 )

