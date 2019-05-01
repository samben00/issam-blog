---
layout: post
title: "How to find the default active ethernet interface using Salt State"
date: 2019-03-17 20:13:43
image: '/assets/img/'
description: A salt state that will help you find the default active ethernet interface
tags: 
- saltstack
- salt state
- Interface salt state
- linux
categories:
- Saltstack
twitter_text:
---

## Issue

Recently I was working on project where I needed to get the default active ethernet interface using SaltStack. Finding the default interface is quite important when it comes to configuring networks or applications.

I wanted something similar to `route | grep 'default' | awk '{print $8}'` which returns the name of the default interface. Of course I want the same result but using saltstack so I can use the returned value to configure some files. In my case It I needed to configure listener interface on applications.

## Solution 

The solution was very simple and clean using salt. This is what I run 

{% highlight python %}

$ sudo salt "minion" network.default_route inet

minion:
    |_
      ----------
      addr_family:
          inet
      destination:
          0.0.0.0
      flags:
          UG
      gateway:
          10.0.2.2
      interface:
          enp0s3
      netmask:
          0.0.0.0
{% endhighlight %}

<b> The command above is equivalent to </b>


{% highlight bash %}
vagrant@saltmaster ~]$ route | grep '^default' 
{% endhighlight %}


To utilize the command on your sls files you can save the output of the command into a variable.

{% highlight python %}

{ % set default_interface = salt['network.default_route']('inet')[0]['interface'] % }

/etc/app/conf/app.conf:
  file.managed:
    - source: salt://app.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - defaults:
        listen_on: { { default_interface } }

{% endhighlight %}


