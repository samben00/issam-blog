---
layout: post
title: "How to monitor a remote JVM over SSH with JVisualVM"
date: 2017-05-09 14:58:21
image: '/assets/img/'
description:
- VisualVM is a great tool for monitoring JVM regarding memory usage, threads, GC, MBeans etc. Let’s see how to use it over SSH to monitor (or even profile, using its sampler) a remote JVM with JMX
tags:
- JVisualVM
- jvm
- jmx
- monitor remote jvm
categories:
- java
---

JVisualVM is a great tool for monitoring JVM regarding memory issues, threads and GC ... etc.
There are two modes of communication between VisualVM and the JVM either over JMX or jstatd.

In this post I will only go through over JMX.

## What I m trying to do

I will explain how to monitor a remote application through JMX by establishing an SSH tunnel.

## Enviroment 

- JAVA Application running on ubuntu machine 16.04

## Prerequisite 

- JVisualVM comes with the the JVM, so you don't have to install anything.so make sure that your local machine has visualvm otherwise download it. 
- in case you want the latest version you can grab it from this link [Dwnload VisualVM](https://visualvm.github.io/)

## Basic Steps

1. Prepare JAVA application 
2. Establish ssh tunnel to the remote machine
3. Run and Configure JVisualVM
 
### 1. JAVA application
 
- The only disadvantage about JMX is you need to start the JVM with some properties.
- So make sure you have the following arguments added to your jvm then restart your application.

{% highlight bash%}
yourJavaCommand...  -Dcom.sun.management.jmxremote.ssl=false 
                    -Dcom.sun.management.jmxremote.authenticate=false 
                    -Dcom.sun.management.jmxremote.port=9010
                    -Dcom.sun.management.jmxremote.rmi.port=9011
                    -Djava.rmi.server.hostname=localhost
                    -Dcom.sun.management.jmxremote.local.only=false
{% endhighlight %}

FWI : it's important to set both ports jmx and rmi port.

For more info regarding jmx connection, see [Remote JMX Connections](http://docs.oracle.com/javase/6/docs/technotes/guides/visualvm/jmx_connections.html).

### 2. SSH TUNNEL

- Now it's time to establish an ssh connection. 
- Change the pem file, username and your remote ip address.
 
{% highlight bash %}
ssh -i yourPermissionFile.pem -l username 101.101.101.101 -L 9010:localhost:9010 -L 9011:localhost:9011
{% endhighlight %}

### 3. JVisualVM

- Open VisualVM application
- on the left pannel click right on local then add new jmx connection.
- for the connection config put <b>localhost:9010</b>

![run-VisualVM](/assets/img/post/jvisualvm/1.png){:class="img-responsive"}

![configure-VisualVM](/assets/img/post/jvisualvm/2.png){:class="img-responsive"}

