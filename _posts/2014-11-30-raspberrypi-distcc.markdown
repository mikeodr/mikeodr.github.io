---
layout: post
title:  "Cross compiling for Raspberry Pi with distcc"
date:  2014-11-30 12:34:00
categories: [RaspberryPi]
---

# Using distcc to cross compile for Raspberry Pi

My post here is based off the following guide: <http://www.openframeworks.cc/setup/raspberrypi/Raspberry-Pi-DISTCC-guide.html>

This guide assumes you are familiar with compiling and have all required build
tools installed.

## Setup your Ubuntu PC

Install the following:
{% highlight bash %}
sudo apt-get install distcc
{% endhighlight %}

Grab the Raspberry Pi compiler for Linux and remember where you’ve placed it:
{% highlight bash %}
git clone <https://github.com/raspberrypi/tools.git> --depth=1 rpi-tools
{% endhighlight %}
I’ll refer to this path as $RPI_TOOLS from now on.

Edit distcc’s config file to have it start on boot with appropriate options.

Edit /etc/default/distcc and change the following:
{% highlight bash %}
STARTDISTCC=”true”
ALLOWNETS=”127.0.0.1 192.168.1.0/24” # Change this based on your network config.
ZEROCONF=”true”
LISTNER=”” # Listen on all interfaces
{% endhighlight %}

Now edit the PATH variable for distcc so it can find the Raspberry Pi compiler.
Replace $RPI_TOOLS with the path to where you cloned the compiler.
{% highlight bash %}
PATH=$RPI_TOOLS/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
{% endhighlight %}

Start the distcc daemon.
{% highlight bash %}
sudo service distcc start
{% endhighlight %}

To see the log output of distccd while you’re calling compile at the last step
below you can:
{% highlight bash %}
tail -f /var/log/distccd.log
{% endhighlight %}

## On the Raspberry Pi

Install distcc:
{% highlight bash %}
sudo apt-get install distcc
{% endhighlight %}

Set your distcc hosts on your RPi
and add your Ubuntu PC’s ip to the hosts file.
{% highlight bash %}
mkdir $HOME/.distcc/

## Add ip to this file

vim $HOME/.distcc/hosts
{% endhighlight %}

Now set your make flags to compile with distcc
Replace the number 8 below with the number of CPU cores on your distcc 
server for multiple simultaneous compiling threads.
{% highlight bash %}
export MAKEFLAGS=”-j 8 CXX=/usr/lib/distcc/arm-linux-gnueabihf-g++ CC=/usr/lib/distcc/arm-linux-gnueabihf-gcc”
{% endhighlight %}

Now it’s as simple as going to the directory of your projects Makefile and
calling:
{% highlight bash %}
make
{% endhighlight %}
