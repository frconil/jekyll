---
title: Generating and analyzing logs with S3 
tags: AWS S3 Linux Bash
layout: post
summary: I am using Jekyll to generate a website that I then deploy to an S3 bucket repository to host this blog, and while it is relatively easier and requires less effort than hosting a custom solution on your own server (with all that implies, ie. apache, database, setup, configuration, maintenance) you do relinquish some of the things you took for granted, like apache logs and those fancy graphic reports. Or do you?
---
I am using Jekyll to generate a website that I then deploy to an S3 bucket repository to host this blog, and while it is relatively easier and requires less effort than hosting a custom solution on your own server (with all that implies, ie. apache, database, setup, configuration, maintenance) you do relinquish some of the things you took for granted, like apache logs and those fancy graphic reports. Or do you?

While in no way "the best way" to do things, here is how I achieved it: 

####First step: Enabling and synchronising logs.####

From the S3 console, create a bucket with a unique name. I made mine match my domain, but anything goes so long the AWS console accepts the name.
Go back to your original bucket (the one hosting your website) and set up logging to the newly created bucket.


![Enable logging in S3](/assets/images/logs_s3.png)

Now you need to setup [s3cmd](http://s3tools.org/s3cmd) to download your logs. (hint: it's available through apt-get)

Create a new user in AWS IAM, and set up a new policy to access the log bucket:
{% highlight json %}
{
  "Statement": [
    {
      "Action": [
        "s3:ListAllMyBuckets"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Action": ["s3:*"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::logs.conil.id.au", "arn:aws:s3:::logs.conil.id.au/*"]
    }
  ]
}
{% endhighlight %}

Save your keys and go back to your local workstation and type:
{% highlight bash %}
s3cmd --configure
{% endhighlight %}
And enter the credentials for your newly created user.

You can then syncronise your logs with:
{% highlight bash %}
s3cmd sync s3://<logs bucket> <local folder>
{% endhighlight %}


####Second step: format the logs####

You just need to concatenate all your logs to a temporary file. All the logs will be of the form <prefix>YEAR-MONTH-DAY-HOUR-MINUTE-SECOND-HEXADECIMAL_CHAIN, so merging them is as easy as typing
{% highlight bash %}
cat logs/<prefix>* >> tmpfile
{% endhighlight %}

You then need to reformat the s3 logs to combined log format. Using [Apache's combined log format page](http://httpd.apache.org/docs/1.3/logs.html#combined) and [Amazon's S3 Log Format page](http://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html) you get the following command to reorder the log contents:
{% highlight bash %}
#we split the log using white spaces, and re assemble the ones that are between quotes (ie HTTP request and User Agent)

#example before:

# 1ba73c535889696f47f28d7ca143a2c73957140e42bd154fdabfcc610c730aea blog.conil.id.au [10/Nov/2013:21:09:59 +0000] <SOME_IP> - E33230BFE47525EF WEBSITE.GET.OBJECT css/pygments/default.css "GET /css/pygments/default.css HTTP/1.1" 304 - - 923 14 - "http://blog.conil.id.au/" "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:24.0) Gecko/20100101 Firefox/24.0" -

#example after:

# <SOME_IP> - - [10/Nov/2013:21:09:59 +0000] "GET /css/pygments/default.css HTTP/1.1" 304 - "http://blog.conil.id.au/" "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:24.0) Gecko/20100101 Firefox/24.0"

awk '{ printf "%s - - %s %s %s %s %s %s %s %s",$5,$3,$4,$10,$11,$12,$13,$15,$19;for (i=20;i<NF;i++)printf " %s",$i;printf "\n"}' tmplog > combined.log
{% endhighlight %}

####Fourth step: profit####
I'm personally using awstats from the command line, but your mileage may vary. 
Here is the configuration and process I'm using for the sake of it:
{% highlight ini %}
LogFile="/home/fconil/combined.log"
SiteDomain="blog.conil.id.au"
HostAliases="conil.id.au blog.conil.id.au www.conil.id.au"
DNSLookup=1
DirData="/home/fconil/awstats/"
DirIcons="/usr/share/awstats/icon"
CreateDirDataIfNotExists=1
KeepBackupOfHistoricFiles=1
LogType=W
LogFormat=1
ShowSummary=UVPHB
ShowMonthStats=UVPHB
ShowDaysOfMonthStats=VPHB
ShowDaysOfWeekStats=PHB
ShowHoursStats=PHB
ShowDomainsStats=PHB
ShowHostsStats=PHBL
ShowRobotsStats=HBL
ShowSessionsStats=1
ShowPagesStats=PBEX
ShowFileTypesStats=HB
ShowOSStats=1
ShowBrowsersStats=1
ShowOriginStats=PH
ShowKeyphrasesStats=1
ShowKeywordsStats=1
ShowMiscStats=a
ShowHTTPErrorsStats=1
ShowFlagLinks=""
ShowLinksOnUrl=1
{% endhighlight %}

Files get populated just by running awstats_buildstaticpages.pl:
{% highlight bash %}
/usr/share/awstats/tools/awstats_buildstaticpages.pl -config=blog.conil.id.au -update -dir=$HOME/awstats
{% endhighlight %}


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

cat $HOME/$LOGPREFIX* >> tmplog

awk '{ printf "%s - - %s %s %s %s %s %s %s %s",$5,$3,$4,$10,$11,$12,$13,$15,$19;for (i=20;i<NF;i++)printf " %s",$i;printf "\n"}' tmplog > combined.log

/usr/share/awstats/tools/awstats_buildstaticpages.pl -config=$WEBSITE -update -dir=$HOME/awstats

{% endhighlight %}

