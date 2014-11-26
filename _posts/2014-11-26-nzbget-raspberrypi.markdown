---
layout: post
title:  "NZBGet for Raspberry Pi"
date:   2014-11-26 00:34:00
categories: [RaspberryPi, nzbget]
---

I’ve created a repo for hosting a raspberry pi nzbget deb.
If you’d like to use it please follow the following instructions:

{% highlight bash %}
# Add keys
gpg --recv-keys --keyserver keyserver.ubuntu.com 0E50BF67
gpg -a --export 0E50BF67 |sudo apt-key add -
{% endhighlight %}

Then add the following to your /etc/apt/sources.list
{% highlight bash %}
#NZBGet repo
deb http://packages.unusedbytes.ca wheezy main
{% endhighlight %}

Then it’s as simple as:
{% highlight bash %}
sudo apt-get update
sudo apt-get install nzbget
{% endhighlight %}
