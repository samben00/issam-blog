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

### 2. Configuration

- make sure you run the command as root.

MY_REGION : is the region of your running EC2 instance ( for example : us-east-1 )

- there are two way of configuring the CloudWatch agent, either using the interactive mode or non interactive mode. let's start with the interactive mode.

#### Interactive Mode:

{% highlight bash %}
sudo python awslogs-agent-setup.py -r MY_REGION
{% endhighlight %}

You will prompted to fill some details

1. <b> AWS Access Key ID : </b> write or paste your Aws Access ID and hit enter.
2. <b> AWS Secret Access Key : </b> the same goes for secret keys. (you can get always get news keys from the IAM console).
3. Default region name :  region name in this case it will be the name of region we put in the command.
4. Default output format [None]: leave it blank unless you have another format.
5. Path of log file to upload: the absolute path of the file you want to monitor, for example [/var/log/syslog].
6. Destination Log Group name : This allows you to group your logs by name, you can you create different groups for your logs for instance ( staging, production, ... ).
7. Last thing is the stream name,  you can you instance ID or custom.

#### Non Interactive Mode:

- first you need to create a file with the following configuration, we you use this file in the next command. 

{% highlight bash %}
[general]
state_file = /var/awslogs/state/agent-state

[/var/log/syslog]
file = /var/log/syslog
log_stream_name = syslog
log_group_name = stage
datetime_format = %b %d %H:%M:%S
initial_position = start_of_file
{% endhighlight %}

{% highlight bash %}
sudo python awslogs-agent-setup.py -n -r MY_REGION -c CONFIGURATION_FILE
{% endhighlight %}

- after the instalation go the following direcotry [/var/awslogs/etc] and make sure that [aws.conf] has your access ID and secret key.
aws.conf must look something like this

{% highlight bash %}
[default]
aws_access_key_id = ASDFGHJKJGFDSDFGHJKJH
aws_secret_access_key = dtfgyuhbjnkjfyghjk4567895678hgjhhjlkjh
region = us-east-1
[plugins]
cwlogs = cwlogs
{% endhighlight %}

- then restart the aws logs agent
{% highlight bash %}
sudo service awslogs restart
{% endhighlight %}