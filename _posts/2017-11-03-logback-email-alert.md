---
layout: post
title: "Logback email alert using Gmail"
date: 2017-11-03 19:24:18
image: '/assets/img/'
description:'Sending email alerts when error occurs using Gmail account and Logback'
comments: true
tags:
- java
- logback
- logback SMTPAppender
- SMTPAppender logback
categories:
- java
twitter_text:
---
Recently I automated a java application that runs everyday at certain times. The java application was running as Docker container on AWS ECS.

Since the java application was communicating with 3party service through REST API, there was a chance that an error might happen. I needed to be always up to date and receive emails on my Gmail once something fails to execute.
  
Since I was using [logback](https://logback.qos.ch/), it was fairly simple to implement email alerts using SMTPAppender. However, it did not work out as it was described in the [documentation](https://logback.qos.ch/manual/appenders.html#gmailSTARTTLS).
  
In this post I am going to share with you how easy to setup logback in order to receive automatic emails using your Gmail account. 


# What I'm trying to accomplish

1. Setup: Integrating sending mail functionality with LogBack
2. Make it work
3. Extra security step

## 1. Setup

To achieve this, there are 2 ways :
- Use the email sending code at every place wherever i have used logger.error.
- Use the SMTPAppender of logback which sends email by itself whenever logger.error will be called.

Since I have many places where `logger.error` is used obviously the second way is better.

Add the following appender to your logback file.

{% highlight xml %}
<appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>smtp.gmail.com</smtpHost>
    <smtpPort>587</smtpPort>
    <STARTTLS>true</STARTTLS>
    <asynchronousSending>false</asynchronousSending>
    <username>youremail@gmail.com</username>
    <password>your_password</password>
    <to>destination@email.com</to>
    <from>sender@gmail.com</from>
    <subject>TESTING: %logger{20} - %m</subject>
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%date %-5level %logger{35} - %message%n</pattern>
    </layout>
</appender>
{% endhighlight %} 
   
<b>Note:</b> Don't forget to add the appender to the root element.

{% highlight xml %}
<root level="DEBUG">
    <appender-ref ref="EMAIL" />
</root>
{% endhighlight %} 

## 2. Make it work

According to the documentation, this is supposed to work. Well not entirely true. since Google are tight on their security, the email will never reach your inbox.

Since I am using a Gmail account to send emails, Google flag it as non secure App. Therefore my application was not able to send any email.

The answer to solve this issue is by simply changing your account parameters. according to this [documentation](https://support.google.com/accounts/answer/6010255).

<b>Note:</b> if you are using your own email serve, most likely you will not run into this issue(depending on your settings).

## 3. Security

Well I don't feel comfortable putting my email's password in the xml configuration file without any encryption. 

That's why I have used git-crypt tool to encrypt my logback.xml file using gpg key.

You can refer to [git-crypt](https://github.com/AGWA/git-crypt) project to read more about how to use it.

In order to crypt your `logback.xml` file which contains secerts. add the following line to `.gitattributes` file. create the file in case does not exists.

{% highlight bash %}
src/main/resources/logback.xml filter=git-crypt diff=git-crypt
{% endhighlight %}

Run the following commands to encrypt your file, before committing your changes.
{% highlight bash %}
git-crypt add-gpg-user my-gpg-key
git-crypt lock
{% endhighlight %}





    