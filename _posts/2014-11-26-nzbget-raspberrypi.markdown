---
layout: post
title:  "NZBGet for Raspberry Pi"
date:   2014-11-26 00:34:00
categories: [RaspberryPi, nzbget]
---

I’ve created a repo for hosting armhf debs for Raspberry Pi.
As of this posting it only contains NZBGet deb.

If you’d like to use it please use the following instructions:

Add the signing keys:
{% highlight bash %}
# Add keys
gpg --recv-keys --keyserver keyserver.ubuntu.com 0E50BF67
gpg -a --export 0E50BF67 |sudo apt-key add -
{% endhighlight %}

Then add the following to your /etc/apt/sources.list:
{% highlight bash %}
#NZBGet repo
deb http://packages.unusedbytes.ca wheezy main
{% endhighlight %}

Then it’s as simple as:
{% highlight bash %}
sudo apt-get update
sudo apt-get install nzbget
{% endhighlight %}

For nzbget to start copy the conf file:
{% highlight bash %}
cp /usr/share/nzbget/nzbget.conf ~/.nzbget
{% endhighlight %}
