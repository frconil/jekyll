---
title: Recover a client certificate from chrome
tags: SSL chrome Linux
layout: post
summary: After migrating workstations, I realised I forgot to take care of one of my certificates. A client certificate. Now the workstation in question has been dealt with and disposed...
---

After migrating workstations, I realised I forgot to take care of one of my certificates.   
A client certificate.  
The one I use to generate and renew other certificates. Or access my tax file. or a million other things that would require a client certificate.

Now, the workstation in question has been dealt with and disposed (it was a viking funeral, very intense emotionally. Sadly it was only for close friends and the guys from Internal IT), but its innards will remain with us forever, in the form of an image of the old HDD, which can be (and has been numerous times) mounted at will to recover old files/settings/etc. 

Enough drama, and onwards to the meaty bits, aka "how to recover that @#$% client certificate"

Using your package manager of choice, install "libnss3-tools" and do the following:
{% highlight bash lineos %}
    certutil -d sql:$OLDHOME/.pki/nssdb -L
{% endhighlight %}
You should get something looking like this:
{% highlight bash lineos %}
    Certificate Nickname                                         Trust Attributes
                                                               	SSL,S/MIME,JAR/XPI
    
    <Certificate name>						 u,u,u
{% endhighlight %}
Now all you need to do is:
{% highlight bash lineos %}
    pk12util -o certfile.p12 -d sql:$OLDHOME/.pki/nssdb -n "<Certificate name>"
{% endhighlight %}

After creating a password for the certificate, you should be able to import it back in your browser of choice.  


