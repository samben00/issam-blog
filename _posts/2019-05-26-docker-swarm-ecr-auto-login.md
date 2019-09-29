---
layout: post
title: "Docker Swarm ECR Auto-Login"
date: 2019-05-26 18:34:43
image: '/assets/img/'
description: How to auto login to AWS ECR when using Docker Swarm with AWS AutoScaling 
tags:
- Docker
- ECR
- Docker Swarm
- AWS ECR auth
categories:
- Docker
- AWS
twitter_text:
---

## Issue Description

I have been using Docker Swarm for quite some time to manage a cluster of applications running on EC2 instances on AWS. Everything was fine with my docker swarm cluster until I have enabled AWS Auto-Scaling on docker workers to manage the increased load.

The issue I had was whenever a new instance was created Docker swarm could not pull Docker images into the newly launched instances as results docker is unable to run services on the newly created instances.

Of course, the first obvious solution came to my mind is to set a Cron job on the docker workers that login to ECR every day since the ECR login token is valid for 12 hours. Unfortunately, this did not work, and I was getting the same error.

After reading more about how docker-swarm authentication works, I found out that docker swarm doesn’t refresh the Auth tokens unless you update or deploy a service. So what happens is that when a service is created using `--with-registry-auth`, the docker manager pull the tokens stored locally on the manager and send it to all agents so the workers can pull the image from the private registry (ECR in our case). Then docker swarm store this token in the raft storage which is shared among all the Docker swarm agents. 

Also, theses tokens remain stored and only refreshed by the manager. This means even though you run a docker login command on the worker, it won’t use the local tokens; instead, it uses the tokens stored in the docker swarm raft.

According to docker swarm documentation, no command allows refreshing these tokens without running the update or the deploy command.
## Solution

### Simple Solution

In short the command that allows refreshing the auth tokens. 

{% highlight bash%}
#!/bin/bash

SERVICES=$(docker service ls --format '{{.Name}}' 2>/dev/null)
for SERVICE in $SERVICES; do
    docker service update -d -q --with-registry-auth $SERVICE
done

{% endhighlight %}

**Note:** Make sure you run this command from a **manager node** not a worker node.

### Complete solution

Obviously you want the command above to run at least twice a day, since the tokens are valid for 12 hours only.

There are different ways to implement this, either by using:
- Cron job
- Systemd service with a timer
- As docker service that is part of your cluster

I think the first and second solution are the simplest and I don't see a need of creating another docker service just to refresh tokens.

In my case I setup a systemd with timer that runs the command above every hour

**/usr/bin/docker-ecr-login.sh**
{% highlight bash%}
#!/bin/bash
set -eu

# Login to ECR

GET_LOGIN_OUTPUT="$(aws --region=eu-central-1 ecr get-login --no-include-email)"
LOGIN_ARGS="$(echo "$GET_LOGIN_OUTPUT" | sed 's/^docker login \(.*\)/\1/g')"

if [ -n "$LOGIN_ARGS" ]; then
    docker login ${LOGIN_ARGS[@]}
    if [ $? -eq 0 ]; then
        >&2 echo "Docker login succeeded"

        # Refresh auth tokens
        >&2 echo "Running docker update swarm services registry auth"
        SERVICES=$(docker service ls --format '{{.Name}}' 2>/dev/null)
        for SERVICE in $SERVICES; do
            docker service update -d -q --with-registry-auth $SERVICE
        done
    else
        >&2 echo "Docker login failed"
        exit 1
    fi
else
    >&2 echo "Failed to acquire docker login credentials"
    exit 1
fi
{% endhighlight %}

**/etc/systemd/system/docker_ecr_login.service**

{% highlight bash%}
[Unit]
Description=Docker service update (Login to ECR + Refresh registry auth tokens)
Requires=docker.service

[Service]
Type=oneshot
User=root
Group=root
ExecStart=/usr/bin/docker-ecr-login.sh
{% endhighlight %}

**/etc/systemd/system/docker_ecr_login.timer**

{% highlight bash%}
[Unit]
Description=Scheduled Docker ECR Login

[Timer]
OnCalendar=hourly

[Install]
WantedBy=timers.target
{% endhighlight %}


**Start the services**

{% highlight bash %}
$ sudo systemctl start docker_ecr_login.timer

{% endhighlight %}

