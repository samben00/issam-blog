---
layout: post
title: "How to setup up logroate with aws s3"
date: 2017-04-09 20:09:18
image: '/assets/img/postlogroate-aws.jpg'
description: 'This is a tutorial explain how to rotate files and upload back them up in a S3 bucket.'
comments: true
tags:
- logrotate
- AWS s3
- linux ubuntu
- ubuntu logrotate 
- centralized logging 
categories:
- aws
twitter_text:
---

## What I m trying to do

Rotate my logs everyday at midnight, archive the rotated files. then upload them to an S3 bucket.
all archived files on S3 will be referenced with a timestamp, to make the search process easy

## Enviroment 
- EC2 instance running ubuntu machine 14.04

## Tools 
- [awscli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) Tool
- logrotate 

## Basic Steps
1. configure logrotate 
2. create bash script to upload files to S3
3. setup init.d to upload files in case of (restart or shutdown)

## Configuration

### 1. logrotate

- first make sure that `/etc/logrotate.conf` file has this line `include /etc/logrotate.d`. 
this line tells logrotate to pickup any configuration sitting in this directory
- cd `/etc/logrotate.d/` and create new config file, in this demo I will use apache logs as an example . you can replicate the same configuration for other types of logs (syslogs, mail logs ..).
- this is what my configuration file `/etc/logrotate.d/apache2` looks like
{% highlight bash %}
/var/log/apache2/*.log {
        daily
        missingok
        rotate 10
        compress
        delaycompress
        ifempty
        create 640 root adm
        prerotate
               /bin/bash /home/ubuntu/backup_current_logs.sh
        endscript
        postrotate
                if /etc/init.d/apache2 status > /dev/null ; then \
                    /etc/init.d/apache2 reload > /dev/null; \
                fi;
        endscript

}
{% endhighlight %}
- let me explain what the config does
    - fitst I tell logrotate to rotate any file ends with `.log` inside the directory `/var/log/apache2/`
    - `daily` : rotate my files everyday 
    - `missingok` : to avoid error messages in case the file is missing
    - `rotate 10` : keep 10 rotated files, for example I will have error.log error.log.1 error.log.3 ... error.log.10
    - `delaycompress` : start compressing files from third file, so error.log and error.log.1 won't be compressed
    - `ifempty` :  Rotate the log file even if it is empty
    - `prerotate ` : the action I want to execute before rotation, in my case I want to upload files to S3
    - `postrotate` : then I reload apache2

### 2. Uploading to S3 

- this is my script file that uploads logs to S3, I m appending the date and instance ID for better reference.
Also if you have multiple environment you can replace "environment" with the adequate env name. 
{% highlight bash %}
# this script will create an archive file of current apache logs
tar -C/var --warning=no-file-changed  -zcf /tmp/log.tar.gz log/apache2/error.log log/apache2/access.log

# then upload the archived file to S3 bucket
EC2_INSTANCE_ID="`wget -q -O - http://instance-data/latest/meta-data/instance-id`"
su - ubuntu -c "aws s3 cp /tmp/log.tar.gz s3://my-bucket-name/environment/`date +%Y-%m-%d`_${EC2_INSTANCE_ID}.tar.gz "
{% endhighlight %}

### 3. Managing shutdown/restart
- just copy `/home/ubuntu/backup_current_logs.sh` to `/etc/init.d/`