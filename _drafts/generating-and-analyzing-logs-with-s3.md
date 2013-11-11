---
title: Generating and analyzing logs with S3 
tags: S3 Linux Reporting 
layout: post
summary: I used Jekyll deployed to an S3 bucket repository to host this blog, and while it is relatively easier and requires less effort than hosting a custom solution on your own server (with all that implies, ie. apache, database, setup, configuration, maintenance) you do relinquish some of the things you took for granted, like apache logs. Or do you?
---

First you need to enable s3 logging like so:
- create log bucket
- enable logs to log bucket (and set up accordingly in s3)

I'm then using s3cmd, bash and awstats to generate an HTML file

set up s3cmd.

Recombine logs using [Apache's combined log format page](http://httpd.apache.org/docs/1.3/logs.html#combined) and [Amazon's S3 LogFormat page](http://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html) as referencei

Bash script to download,recombine and generate log report.
{% highlight bash %}

{% endhighlight %}

