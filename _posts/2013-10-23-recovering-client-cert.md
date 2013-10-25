---
title: How to recover a client cert from chrome
tags: SSL chrome Linux
layout: post
summary: Sometimes after migrating workstations, I realised I forgot to take care of one of my certs. A client cert. Now the workstation in question has been dealt with and disposed...
---

Sometimes after migrating workstations, I realised I forgot to take care of one of my certs.   
A client cert. 

Now, the workstation in question has been dealt with and disposed (it was a viking funeral, very intense emotionally and with only close friends), but its innards will remain with us forever, in the form of an image of the old HDD, which can (and has been numerous times) mounted at will to recover old files/settings/etc. 

Enough drama, and onwards to the meaty bits, aka "how to recover that @#$% client certificate"

Using your package manager of choice, install "libnss3-tools" and do the following:
{% highlight bash %}
    certutil -d sql:$OLD-HOME/.pki/nssdb -L
{% endhighlight %}
You should get something looking like this:
{% highlight bash %}
    Certificate Nickname                                         Trust Attributes
                                                               	SSL,S/MIME,JAR/XPI
    
    <Certificate name>						 u,u,u
{% endhighlight %}
Now all you need to do is:
{% highlight bash %}
    pk12util -o certfile.p12 -d sql:$OLD-HOME/.pki/nssdb -n "<Certificate name>"
{% endhighlight %}

You can now import that certificate in your browser of choice!
