---
title: Managing external repos with the apt puppet module 
tags: Puppet Debian Packaging
layout: post
---
If you're using debian (or ubuntu), it's very likely you're using external repos, moreso if you're using Puppet. Or Nginx. Or Postgres. Or MariaDB. 

And if you do, it's also very likely you're starting to look at [apt pinning](https://wiki.debian.org/AptPreferences#Pinning) to deal with all those cases where the package you want is definitely not what you got (PYWIDNWYG in short).

Pinning is really easy to achieve with the [apt module](https://github.com/puppetlabs/puppetlabs-apt)

You just need to declare your class instance like so:
{% highlight puppet %}
apt::source { 'puppetlabs':
    location   => 'http://apt.puppetlabs.com/',
    release    => $::lsbmajdistrelease ? 
                 '5'  => 'squeeze',
                 '6'  => 'squeeze',
                 '7'  => 'wheezy',
    },
    repos       => 'main dependencies',
    pin         => '600',
  }
{% endhighlight %}


The pref file generated should look like this:

{% highlight text %}
Explanation: apt: puppetlabs_apt
Package: *
Pin: origin apt.puppetlabs.com
Pin-Priority: 600
{% endhighlight %}

Tada, you have puppetlabs repo set up AND pinned to a higher priority! (default priority is 500 FYI)


Now, if you want to make it extra fun, how about caching your repos? You can try Apt-Proxy or Apt Cacher NG, or even a trusty apache or nginx. The result is pretty much the same, you have a cached copy of your packages as they are requested.

So let's say you end up with myaptproxy/pgdg, myaptproxy/puppetlabs, myaptproxy/mariadb, myaptproxy/debian etc. Great. And that's when pinning is going to come back and bite you in the ass.

Remember the origin line?

Now everything is like this:
{% highlight text %}
Explanation: apt: puppetlabs_apt
Package: *
Pin: origin myaptproxy
Pin-Priority: 600
{% endhighlight %}

Which means that **all relevant priorities are ignored**, since origin is the same for both puppetlabs and debian (in our example)

The key is to look at the "Release" file and look for differences.

{% highlight text %}
Origin: Puppetlabs
Label: Puppetlabs
Suite: wheezy
Codename: wheezy
{% endhighlight %}

And the Debian one:

{% highlight text %}
Origin: Debian
Label: Debian
Suite: stable
Version: 7.5
Codename: wheezy
{% endhighlight %}

Origin is a perfect match for what we want to do. Let's edit our puppet declaration to differentiate using it. For this we'll need to actually use apt::pin instead of relying on lazy pin options:

{% highlight puppet %}
apt::source { 'puppetlabs':
    location   => 'http://myaptproxy/puppetlabs',
    release    => $::lsbmajdistrelease ? 
                 '5'  => 'squeeze',
                 '6'  => 'squeeze',
                 '7'  => 'wheezy',
    },
    repos       => 'main dependencies',
  }

apt::pin { 'puppetlabs':
    originator  => Puppetlabs,
    priority    => '600',
}
{% endhighlight %}

Let's now look at our pref file:

{% highlight text %}
Explanation: apt: puppetlabs_apt
Package: *
Pin: release o=Puppetlabs
Pin-Priority: 600
{% endhighlight %}

The o=Puppetlabs force *apt* to only apply the upgraded priority to the packages coming from puppetlabs, as definied in their repo "Release" file.
