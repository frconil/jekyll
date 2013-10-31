---
title: Calculate SLAs using jmeter and Bash (or Python/perl/ruby/insert language of choice) 
tags: Jmeter Bash Linux SLAs 
layout: post
summary: For some time I raked my brains trying to figure an accurate way to report SLAs. Obviously Nagios could tell me when a service was up, but it's not always enough from the user point of view... 
---

For some time I raked my brains trying to figure an accurate way to report SLAs. 

Obviously Nagios could tell me whether a service was up, but it's not enough from the user point of view, and we very quickly go back to the painful problem of adjusting metrics and reporting to match a customer expectation. 

I think we've all experienced an irate call/mail from a stakeholder telling us that "the system is down" while Nagios is trudging along nicely with all services green. 



At first I tried to use curl to pull the login page, send the necessary login information, and display the end result, but curl doesn't really like meta redirects, which the app was using. 

That's when I remembered the testers had a fully fucntional jmeter script used to load test the application. The script would go to the app, follow the redirect to the SSO page, log in, do a couple actions and then log out. That seems like a perfect fit for the customer expectation:

> "The system is up and running because I can login, use the application and log out" 

as opposed to 

> "The system is up and running because [Apache, Mysql, OpenLDAP] is green in nagios"

I had to do a bit of wrapping though, as I couldn't expect someone to log in and run jmeter at regular intervals (duh.)


I assume you have a fully working copy of jmeter in /opt/monitoring/ .  
I have a cron entry running every minute running this script:

{% highlight bash %}
#!/bin/bash

DAY=`date +%d`
MONTH=`date +%m`
YEAR=`date +%Y`

/opt/monitoring/bin/jmeter -n -t /opt/monitoring/login.jmx  -l /var/log/monitoring/log.jtl.$DAY.$MONTH.$YEAR
{% endhighlight %}

This gives me a log that looks like this:

    1382401905857,1168,Login Redirect,200,OK,WebApp 1-1,text,true,10821,1167
    1382401907124,4645,Login,200,OK,WebApp 1-1,text,true,10174,4645

I can then extract the SLA using this bash script:
{% highlight bash  %}
#!/bin/bash


#this report analyses the results from a jmeter login script set to run every minute.
#logs are generated and rolled every day 

DAY=$1
MONTH="$2"
YEAR="$3"

if [ "$#" -ne 3 ] ; then
	echo "Usage is report_sla.sh Day Month Year"
	echo "wrong arguments given, assuming report on last day results"
	DAY=`date --date="yesterday" +%d` 
	MONTH=`date --date="yesterday" +%m` 
	YEAR=`date --date="yesterday" +%Y`
fi
LOG=log.jtl.$DAY.$MONTH.$YEAR


if [ -f $LOG ] ; then 
	TOTAL=`wc -l $LOG | cut -d ' ' -f1`
	ERROR=`grep -cv "302,Moved\|200,OK" $LOG`
	SLA=`echo "($TOTAL-$ERROR)*100/$TOTAL" | bc`
	echo "SLA for `date -d \"$MONTH/$DAY/$YEAR\" +%d\ %b\ %Y` is $SLA %" 	
else
	echo "no data recorded for `date -d \"$MONTH/$DAY/$YEAR\" +%d\ %b\ %Y`"
fi

{% endhighlight %}

Basically, you analyse your log results, and compare the number of errors against the "expected" error codes (200 and 302).
The maths are still a bit flawed as it does not consider partial success, but there is always room for improvement.
