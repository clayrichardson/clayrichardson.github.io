---
layout: post
title: "TI LaunchPad development on Ubuntu 11.10"
date: 2012-03-15 15:43:00 -0600
comments: true
categories: 
alias: [/2012/03/15/ti-launchpad-development-on-ubuntu-11-10/]
---
I got my hands on a couple of [TI LaunchPads](http://www.ti.com/launchpad) and was surprised there is no official support for Linux. Sad day. Regardless, at $4.30 each + shipping, they're a steal.

Interested in getting development running on Ubuntu, I was reading [Hack A Day's LaunchPad on Ubuntu article](http://hackaday.com/2010/08/11/how-to-launchpad-programming-with-linux/) until I got to the [mspgcc4 project page](http://mspgcc4.sourceforge.net/) and noticed
>mspgcc4 is no longer supported. All contributions have been incorporated into [mspgcc](http://sourceforge.net/projects/mspgcc/), which has newer versions of all components.
Following the link to the [GCC toolchain for MSP430 web site](http://sourceforge.net/apps/mediawiki/mspgcc/index.php?title=MSPGCC_Wiki), you'll find install instructions for "[Ubuntu](http://sourceforge.net/apps/mediawiki/mspgcc/index.php?title=Install:debian) and other Debian-derived distributions" which will tell you "mspgcc will be available in Ubuntu Oneiric. Packages are listed [here](https://launchpad.net/ubuntu/oneiric/+search?text=msp430), and were shepherded by Luca Bruno."

[tl;dr](http://en.wikipedia.org/wiki/Wikipedia:Too_long;_didn't_read): Seeing the Hack A Day article was from August of 2010, I figured I would try to go with the more updated toolchain.
<!-- more -->

Install the necessary packages:
```
sudo apt-get install msp430-libc mspdebug msp430mcu binutils-msp430 gcc-msp430 gdb-msp430
```

You will get an error:
```
dpkg: error processing /var/cache/apt/archives/gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_amd64.deb (--unpack):
trying to overwrite '/usr/share/gdb/python/gdb/__init__.py', which is also in package gdb 7.3-0ubuntu2
```

Substitute i386 for amd64 if you're running on a 32 bit system.

There are bug reports for this [here](https://bugs.launchpad.net/ubuntu/+source/gdb-msp430/+bug/860045), [here](https://bugs.launchpad.net/ubuntu/+source/gdb-msp430/+bug/868317), and [here](https://bugs.launchpad.net/gdb-linaro/+bug/891970/comments/7). In the [last comment](https://bugs.launchpad.net/ubuntu/+source/gdb-msp430/+bug/860045/comments/3) of the first linked bug report, [Marek Burza (mkburza)](https://launchpad.net/~mkburza) wrote on 2011-11-23:

>The package tries to overwrite:
>/usr/share/gdb/python/gdb/__init__.py
>/usr/share/gdb/python/gdb/types.py
>/usr/share/gdb/python/gdb/printing.py
>/usr/share/gdb/python/gdb/command/__init__.py
>/usr/share/gdb/python/gdb/command/pretty\_printers.py

>These files are identical (in my case) to the ones already installed so to work around the problem I ran the installation with an override instead:
>```sudo apt-get -o Dpkg::Options::="--force-overwrite" install gdb-msp430```

Let's see if that's the case. We'll take the md5sum of the specific files mentioned to make sure they're truly identical and safe to overwrite. To skip the tedious steps because you know everything, there's a bash one liner further down that prints each file's md5sum.

First, make a directory to unpack the debian package in: (cd without specifying a directory defaults to ~/)
```
cd
mkdir ./gdb-msp430
```

Copy the .deb package that apt-get has downloaded to the temp directory in your home directory:
```
cp /var/cache/apt/archives/gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_amd64.deb ~/gdb-msp430/
cd ./gdb-msp430/
```

Again, substituting i386 for amd64 if you're running a 32 bit system.

Unpack it:
```
ar vx ./gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_amd64.deb
```

Untar the data.tar.gz file:
```
tar -xvpzf ./data.tar.gz
```

Now, get the md5sum of all the files in that directory:
```
md5sum ./usr/share/gdb/python/gdb/*
md5sum /usr/share/gdb/python/gdb/*
md5sum ./usr/share/gdb/python/gdb/command/*
md5sum /usr/share/gdb/python/gdb/command/*
```

You should see something like:
```
$ md5sum ./usr/share/gdb/python/gdb/*
md5sum: ./usr/share/gdb/python/gdb/command: Is a directory
7b88259f8fa8e24bbe87796a43740e4c ./usr/share/gdb/python/gdb/__init__.py
bbd6a17b9c37fb605ecdb5d6bdb621d0 ./usr/share/gdb/python/gdb/printing.py
8648e656c6d3c96fdfdee21f3d3ef149 ./usr/share/gdb/python/gdb/types.py

$ md5sum /usr/share/gdb/python/gdb/*
md5sum: /usr/share/gdb/python/gdb/command: Is a directory
7b88259f8fa8e24bbe87796a43740e4c /usr/share/gdb/python/gdb/__init__.py
bbd6a17b9c37fb605ecdb5d6bdb621d0 /usr/share/gdb/python/gdb/printing.py
8648e656c6d3c96fdfdee21f3d3ef149 /usr/share/gdb/python/gdb/types.py

$ md5sum ./usr/share/gdb/python/gdb/command/*
5c52536fb16cd5770660441d4f787928 ./usr/share/gdb/python/gdb/command/__init__.py
66179835aee61f2a3c948ef95363c1fc ./usr/share/gdb/python/gdb/command/pretty_printers.py

$ md5sum /usr/share/gdb/python/gdb/command/*
5c52536fb16cd5770660441d4f787928 /usr/share/gdb/python/gdb/command/__init__.py
66179835aee61f2a3c948ef95363c1fc /usr/share/gdb/python/gdb/command/pretty_printers.py
```

Bash one liner-ish:
```
cd; mkdir ./gdb-msp430; cp /var/cache/apt/archives/gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_amd64.deb ~/gdb-msp430/; cd ./gdb-msp430/; ar vx ./gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_amd64.deb; tar -xvpzf ./data.tar.gz; echo "package files:"; md5sum ./usr/share/gdb/python/gdb/*; echo "installed files:"; md5sum /usr/share/gdb/python/gdb/*; echo "package files:"; md5sum ./usr/share/gdb/python/gdb/command/*; echo "installed files:"; md5sum /usr/share/gdb/python/gdb/command/*; echo "Cleaning up..."; cd ../; rm -r ./gdb-msp430/
```

Or, if you're running 32 bit:
```
cd; mkdir ./gdb-msp430; cp /var/cache/apt/archives/gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_i386.deb ~/gdb-msp430/; cd ./gdb-msp430/; ar vx ./gdb-msp430_7.2~mspgcc-7.2-20110612-1ubuntu1_i386.deb; tar -xvpzf ./data.tar.gz; echo "package files:"; md5sum ./usr/share/gdb/python/gdb/*; echo "installed files:"; md5sum /usr/share/gdb/python/gdb/*; echo "package files:"; md5sum ./usr/share/gdb/python/gdb/command/*; echo "installed files:"; md5sum /usr/share/gdb/python/gdb/command/*; echo "Cleaning up..."; cd ../; rm -r ./gdb-msp430/
```

Well, looks like everything is identical, so now we can force an overwrite of the files:
```
sudo apt-get -o Dpkg::Options::="--force-overwrite" install gdb-msp430
```

Delete the temp directory you created in your home directory to clean up:
```
rm -r ~/gdb-msp430/
```

Now, test the toolchain. Grab a copy of a simple blink program:

(I usually keep my sources in ~/sources/[version control system]/[project name] so: ~/sources/git/had_launchpad-blink/)
```
git clone git://github.com/richardsonclay/had_launchpad-blink.git
```

Change into the newly created directory:
```
cd ./had_launchpad-blink/
```

And make:
```
make
```

It should output main.elf along with a main.o file.

From here, you should be able to follow the rest of the [Hack A Day article](http://hackaday.com/2010/08/11/how-to-launchpad-programming-with-linux/) to transfer the binary file over to the LaunchPad.

As for me, I'm currently having trouble getting mspdebug to talk to the LaunchPad from [VirtualBox](https://www.virtualbox.org/). [This](https://forums.virtualbox.org/viewtopic.php?f=5&t=3355) post from the VirtualBox forums mentions something about USB timings not working properly in virtualization, but I am determined and will post a walkthrough if I am able to get mspdebug to talk properly using a VM.

Happy hacking!

Edit: [I was able to get it work, aw yeah!](http://clayrichardson.me/2012/03/16/ti-launchpad-development-on-ubuntu-11-10-running-in-virtualbox/)

