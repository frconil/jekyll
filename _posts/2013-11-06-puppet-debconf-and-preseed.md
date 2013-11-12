---
title: Puppet, Debconf and Preseed
tags: Puppet Debian Packaging
layout: post
summary: "For better (and mostly for worse), ops people like to reinvent the wheel. Sometimes it's because we don't have the resources available to us (see: reinventing disk_check with df because you are not allowed to install nagios), sometimes it's just a quick and dirty hack that ends up becoming part of the furniture..."
---


For better (and mostly for worse), ops people like to reinvent the wheel. Sometimes it's because we don't have the resources available to us (see: reinventing disk_check with df because you are not allowed to install nagios), sometimes it's just a quick and dirty hack that ends up becoming part of the furniture.

The important bit is to actually clean up once in a while. 

My issue started when we were streamlining/automating some of our builds. One of the package requirements was sun-java6-*, which involve accepting a licence. Personally I chaffed at the fact that we had a mostly automated build, but we still had to log on individual servers, install java by hand like peasants and then only could we start puppet. It would also make things like unattended self-service deploys (my personal end goal :3 )

Now I was getting started with puppet, so I threw together a quick and very dirty hack to accept the licence:
{% highlight puppet %}
# Base Java class

class sun-java6-jre {
        include sun-java-base
        package { "sun-java6-jre":
                ensure => present,
                require => Exec["has-java-licence"];
        }
}

class sun-java-base {
        exec { "has-java-licence":
                command => "grep -q accepted-sun-dlj-v1-1 /var/cache/debconf/config.dat",
        }
        exec { "set java-licence":
                command => "echo sun-java6-jre   shared/accepted-sun-dlj-v1-1    boolean true | debconf-set-selections",
                unless => "grep -q accepted-sun-dlj-v1-1 /var/cache/debconf/config.dat",
        }
}

{% endhighlight %}

That's quite cringe worthy. I know, because I'm cringing pretty hard myself. (notice the use of debconf-set-selections, but not debconf-get-selections). It also generates a lot of useless chatter in the logs, as the licence check runs at every catalog refresh. 

Now we can do it in a much cleaner way, by grabbing the actual info we need from an already installed package with 
{% highlight bash %}
localhost:~# debconf-get-selections | grep accepted-sun
sun-java6-bin shared/accepted-sun-dlj-v1-1  boolean true
sun-java6-jdk shared/accepted-sun-dlj-v1-1  boolean true
sun-java6-jre shared/accepted-sun-dlj-v1-1  boolean true
{% endhighlight %}

Armed with this, we can generate a new, cleaner puppet module:
{% highlight ruby %}
# Base Java class

class sun-java6-jre {
  file { "/var/cache/debconf/sun-java6-jre.preseed":
    ensure       => present,
    content      => "sun-java6-jre   shared/accepted-sun-dlj-v1-1    boolean true
sun-java6-bin shared/accepted-sun-dlj-v1-1  boolean true
sun-java6-jdk shared/accepted-sun-dlj-v1-1  boolean true",
  }

  package { "sun-java6-jre":
    ensure       => present,
    responsefile => "/var/cache/debconf/sun-java6-jre.preseed", 
    require      => File["/var/cache/debconf/sun-java6-jre.preseed"],
  }

}
{% endhighlight %}

I could also call the file directly from the puppet server itself using source instead of content, like so:
{% highlight ruby %}
file { "/var/cache/debconf/sun-java6-jre.preseed":
    ensure       => present,
    source       => "puppet://puppet/files/sun-java6-jre.preseed",
  }
{% endhighlight %}

Looks already so much better, and cut down on the chatter in the logs.
