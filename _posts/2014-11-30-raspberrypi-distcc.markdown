---
layout: post
title:  "Cross compiling for Raspberry Pi with distcc"
date:  2014-11-30 12:34:00
categories: ["Raspberry Pi"]
tags: ["distcc", "compiling", "cross compiling"]
---

Typically it can be a bit hard to compile for one architecture on CPU
while using another. One way around that is to use `distcc` to take
advantage of distributed complication. Thus using other machines you
have handy to do the compiling for you natively. Use your x86 machine
to do the coding, but your armhf device to do the compilation.

## Setup

My post here is based off the following guide: <http://www.openframeworks.cc/setup/raspberrypi/Raspberry-Pi-DISTCC-guide.html>

> Update 2024: the above link is no longer valid.
> The remainder of the doc should be reasonably valid.
{: .prompt-warning }

This guide assumes you are familiar with compiling and have all required build
tools installed.

### Setup your Ubuntu PC

Install the following:

```bash
sudo apt-get install distcc
```

Grab the Raspberry Pi compiler for Linux and remember where you’ve placed it:

```bash
git clone <https://github.com/raspberrypi/tools.git> --depth=1 rpi-tools
```

I’ll refer to this path as $RPI_TOOLS from now on.

Edit distcc’s config file to have it start on boot with appropriate options.

Edit /etc/default/distcc and change the following:

```bash
STARTDISTCC=”true”
ALLOWNETS=”127.0.0.1 192.168.1.0/24” # Change this based on your network config.
ZEROCONF=”true”
LISTNER=”” # Listen on all interfaces
```

Now edit the PATH variable for distcc so it can find the Raspberry Pi compiler.
Replace $RPI_TOOLS with the path to where you cloned the compiler.

```bash
PATH=$RPI_TOOLS/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

Start the distcc daemon.

```bash
sudo service distcc start
```

To see the log output of distccd while you’re calling compile at the last step
below you can:

```bash
tail -f /var/log/distccd.log
```

### On the Raspberry Pi

Install distcc:

```bash
sudo apt-get install distcc
```

Set your distcc hosts on your RPi
and add your Ubuntu PC’s ip to the hosts file.

```bash
mkdir $HOME/.distcc/

## Add ip to this file

vim $HOME/.distcc/hosts
```

Now set your make flags to compile with distcc
Replace the number 8 below with the number of CPU cores on your distcc
server for multiple simultaneous compiling threads.

```bash
export MAKEFLAGS=”-j 8 CXX=/usr/lib/distcc/arm-linux-gnueabihf-g++ CC=/usr/lib/distcc/arm-linux-gnueabihf-gcc”
```

Now it’s as simple as going to the directory of your projects Makefile and
calling:

```bash
make
```
