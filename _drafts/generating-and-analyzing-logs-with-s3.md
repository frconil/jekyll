---
title: Generating and analyzing logs with S3 
tags: AWS S3 Linux Bash
layout: post
summary: I am using Jekyll to generate a website that I then deploy to an S3 bucket repository to host this blog, and while it is relatively easier and requires less effort than hosting a custom solution on your own server (with all that implies, ie. apache, database, setup, configuration, maintenance) you do relinquish some of the things you took for granted, like apache logs and those fancy graphic reports. Or do you?
---
I am using Jekyll to generate a website that I then deploy to an S3 bucket repository to host this blog, and while it is relatively easier and requires less effort than hosting a custom solution on your own server (with all that implies, ie. apache, database, setup, configuration, maintenance) you do relinquish some of the things you took for granted, like apache logs and those fancy graphic reports. Or do you?

To get reports out of an s3 bucket, using awstats, I used the following
First you need to enable s3 logging like so:
- create log bucket
- attribute right permissions in AWS IAM
- enable logs to log bucket (and set up accordingly in s3)


I'm then using s3cmd, bash and awstats to generate an HTML file

<set up s3cmd, and download logs.>

I then reformat the s3 logs to combined log format using [Apache's combined log format page](http://httpd.apache.org/docs/1.3/logs.html#combined) and [Amazon's S3 Log Format page](http://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html) as reference. Here's how to do it with AWK/Python for reference.


As a conclusion, here is the bash script I use to download the logs, convert them to combined, and then generate an HTML report using awstats:
{% highlight bash %}
#!/bin/bash

LOGS=$HOME/logs
LOGBUCKET=logs.conil.id.au
#Log prefix, as set in the S3 console
LOGPREFIX=blog-
#website as configured in awstats
WEBSITE=blog.conil.id.au


s3cmd sync s3://$LOGBUCKET $LOGS

rm tmplog

cat logs/$LOGPREFIX* >> tmplog

#we split the log using white spaces, and re assemble the ones that are between quotes like requests or user agents
#example before:
#
#example after:
#
awk '{ printf "%s - - %s %s %s %s %s %s %s %s",$5,$3,$4,$10,$11,$12,$13,$15,$19;for (i=20;i<NF;i++)printf " %s",$i;printf "\n"}' tmplog > combinedlog

awstats_buildhtml.pl -config=$WEBSITE
{% endhighlight %}

