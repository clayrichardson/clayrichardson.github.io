---
layout: post
title: "TI LaunchPad development on Ubuntu 11.10 running in VirtualBox"
date: 2012-03-16 01:45:00 -0600
comments: true
categories: 
alias: /2012/03/16/ti-launchpad-development-on-ubuntu-11-10-running-in-virtualbox/
---
In my previous post, I was having trouble debugging and writing the binary files to the [TI LaunchPad](http://www.ti.com/launchpad) from Ubuntu 11.10 running in [VirtualBox](https://www.virtualbox.org/) on Mac OS X Lion. I also tried using the VirtualBox ExtensionPack with USB 2.0 support to no avail.

This is an extension of the [previous post](http://clayrichardson.me/2012/03/15/ti-launchpad-development-on-ubuntu-11-10/) if you're running Ubuntu 11.10 in VirtualBox 4.1.10 on OS X Lion.

So I've figured out how to program and debug the Launchpad from within the guest OS. Albeit with a couple of caveats:

1. The driver from [osx-launchpad](http://code.google.com/p/osx-launchpad/) will need to be installed on the host OS.
2. You have to use remote debugging via the GDB remote stub that mspdebug creates.
3. When you use "target remote" from msp430-gdb, it will segfault, yeay!
4. There will be some compiling from source to avoid the segfault party.

<!-- more -->

### On OSX Lion:

Go to http://code.google.com/p/osx-launchpad/ and install the LaunchPad OSX USB VCP CDC Driver. For some reason, I couldn't get [TI's driver](http://e2e.ti.com/support/interface/digital_interface/m/videos__files/198722.aspx) working. Restart, and the LaunchPad should be recognized and properly connected.

Clone the [mspdebug](http://mspdebug.sourceforge.net/) source into your sources directory:
```
git clone git://mspdebug.git.sourceforge.net/gitroot/mspdebug/mspdebug
```

Compile the code:
```
cd ./mspdebug
make 2>&1
```

You'll get a couple of warnings, but it the binary should now be there. I usually use brew on OS X or [CheckInstall](http://en.wikipedia.org/wiki/CheckInstall) on Linux when I have to install from source, as I don't want to lose track of files littered through my systems. Clutter is bad. The brew formula for mspdebug wouldn't work properly, so I've resorted to compiling from source, and editing the makefile for installation.

Change line 21 of ./Makefile from:
```
PREFIX ?= /usr/local
```

To:
```
PREFIX ?= /usr/from_source
```

You can make yours whatever you want, I just figured /usr/from\_source would be a good place.
```
sudo mkdir /usr/from_source
```

Now, when you do a:
```
make install
```

The files will be moved into /usr/from\_source

Add /usr/from\_source/bin to the /etc/paths file so you'll be able to execute it easily.

Reload the environment:
```
$ mspdebug rf2500
MSPDebug version 0.19 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2012 Daniel Beer
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

Trying to open interface 1 on 008
Initializing FET...
FET protocol version is 30001000
Configured for Spy-Bi-Wire
Set Vcc: 3000 mV
fet: FET returned error code 4 (Could not find device (or device not supported))
fet: command C_IDENT1 failed
fet: identify failed
Trying again...
Initializing FET...
FET protocol version is 30001000
Configured for Spy-Bi-Wire
Sending reset...
Set Vcc: 3000 mV
Device ID: 0xf201
  Code start address: 0xf800
  Code size         : 2048 byte = 2 kb
  RAM  start address: 0x200
  RAM  end   address: 0x27f
  RAM  size         : 128 byte = 0 kb
Device: MSP430F2012
Code memory starts at 0xf800
Number of breakpoints: 2

Available commands:
    =         delbreak  gdb       load      opt       reset     simio
    alias     dis       help      locka     prog      run       step
    break     erase     hexout    md        read      set       sym
    cgraph    exit      isearch   mw        regs      setbreak

Available options:
    color           gdb_loop        iradix
    fet_block_size  gdbc_xfer_size  quiet

Type "help " for more information.
Press Ctrl+D to quit.

(mspdebug)
```

Awesome, let's move on to the next step!

### On the Ubuntu 11.10 guest VM:

A bunch of searching led me to [this bug on Launchpad](https://bugs.launchpad.net/ubuntu/+source/gdb-msp430/+bug/891970) where [Andrzej Bieniek (andyhelp+ubuntu)](https://launchpad.net/~andyhelp+ubuntu) mentioned that compiling from [7.2a](ftp://ftp.gnu.org/pub/gnu/gdb/) resulted in no errors, so I figured I'd might as well try that.

You'll need these packages installed in order to build GDB:
```
sudo apt-get install texinfo libncurses5-dev zlib1g-dev
```

Otherwise, you'll get errors like these:
```
WARNING: `makeinfo' is missing on your system.
configure: error: no termcap library found
/usr/bin/ld: cannot find -lz
```

Clone the [mspgcc project](http://sourceforge.net/projects/mspgcc/) into your sources directory: (~/sources/git/)
```
git clone git://mspgcc.git.sourceforge.net/gitroot/mspgcc/mspgcc
```

This project contains the patches you'll need to build the toolchain from source. In this case, we'll only be needing: msp430-gdb-7.2a-20111205.patch

Download GDB 7.2 source and extract it:
```
wget ftp://ftp.gnu.org/pub/gnu/gdb/gdb-7.2a.tar.bz2
tar -xvpjf ./gdb-7.2a.tar.bz2
```

cd into the directory:
```
cd ./gdb-7.2/
```

Looking at the instructions in the patch file:
```
vim ../mspgcc/msp430-gdb-7.2a-20111205.patch
```

We can see:
```
tar xjf gdb-7.2a.tar.bz2
( cd gdb-7.2 ; patch -p1 < ../msp430-gdb-7.2a-20111205.patch )
mkdir -p BUILD/gdb
cd BUILD/gdb
../../gdb-7.2/configure \
--target=msp430 \
--prefix=/usr/local/msp430 \
2>&1 | tee co
make 2>&1 | tee mo
make install 2>&1 | tee moi
```

But we're only going to follow some of the instructions.

Patch the files:
```
patch -p1 < ../mspgcc/msp430-gdb-7.2a-20111205.patch
```

You'll see a bunch of 'patching file x' messages scroll by. Awesome, you've patched it. Make the necessary directories:
```
mkdir -p BUILD/gdb cd BUILD/gdb
```

Configure:
```
../../configure --target=msp430 --prefix=/usr/local/msp430 2>&1
```

If you're on a multicore machine, you can specify -j [number of cores] to compile multiple files simultaneously:
```
make -j4 2>&1
```

Otherwise, just leave the -j switch out:
```
make 2>&1
```

Once that is done, find where the msp430-gdb executable is:
```
$ which msp430-gdb
/usr/bin/msp430-gdb
```

Replace the executable:
```
sudo cp ./gdb/gdb /usr/bin/msp430-gdb
```

If all is well, you should be able to run it:
```
$ msp430-gdb
GNU gdb (GDB) 7.2
Copyright (C) 2010 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law. Type "show copying"
and "show warranty" for details.
This GDB was configured as "--host=i686-pc-linux-gnu --target=msp430".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
(gdb)
```

Press [Ctrl+D](http://en.wikipedia.org/wiki/End-of-transmission_character) to send an [EOF](http://en.wikipedia.org/wiki/End-of-file) and quit gdb.

Great, so we've got both sides of the puzzle working. Let's put them together!

### On OSX Lion:

mspdebug has this cool thing where it can act as a [GDB remote stub](http://davis.lbl.gov/Manuals/GDB/gdb_17.html#SEC140).

From the [mspdebug manual](http://mspdebug.sourceforge.net/manual.html):
```
gdb [port]
  Start  a  GDB  remote  stub, optionally specifying a TCP port to
  listen on.  If no port is given, the default port is 2000.

  MSPDebug will wait for a connection on this port, and  then  act
  as a GDB remote stub until GDB disconnects.

  GDB's  "monitor"  command can be used to issue MSPDebug commands
  via the GDB interface. Supplied commands are executed non-inter-
  actively, and the output is sent back to be displayed in GDB.
```

So start mspdebug on port 2000 (default if no port is specified):
```
mspdebug rf2500 gdb 2000
```

You should see something similar to:
```
$ mspdebug rf2500 gdb 2000
MSPDebug version 0.19 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2012 Daniel Beer <dlbeer@gmail.com>
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Trying to open interface 1 on 008
Initializing FET...
FET protocol version is 30001000
Configured for Spy-Bi-Wire
Set Vcc: 3000 mV
fet: FET returned error code 4 (Could not find device (or device not supported))
fet: command C_IDENT1 failed
fet: identify failed
Trying again...
Initializing FET...
FET protocol version is 30001000
Configured for Spy-Bi-Wire
Sending reset...
Set Vcc: 3000 mV
Device ID: 0xf201
 Code start address: 0xf800
 Code size : 2048 byte = 2 kb
 RAM start address: 0x200
 RAM end address: 0x27f
 RAM size : 128 byte = 0 kb
Device: MSP430F2012
Code memory starts at 0xf800
Number of breakpoints: 2
Bound to port 2000. Now waiting for connection...
```

### On the Ubuntu 11.10 guest VM:

To be able to talk to the LaunchPad, we have to connect to the host machine on whatever port was specified when starting mspdebug.
From the [mspdebug manual](http://mspdebug.sourceforge.net/manual.html):
```
gdbc
  GDB  client  mode.  Connect to a server which implements the GDB
  remote protocol and provide an interface  to  it.  To  use  this
  driver, specify the remote address in hostname:port format using
  the -d option.
```

To do this when networking is configured as NAT, we need to find the default gateway:
```
route
```

You should see something similar to:
```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.2.2        0.0.0.0         UG    0      0        0 eth0
10.0.2.0        *               255.255.255.0   U     1      0        0 eth0
link-local      *               255.255.0.0     U     1000   0        0 eth0
```

We're interested in the line that begins with 'default'.
Start mspdebug with the gbdc driver:
```
$ mspdebug gdbc -d 10.0.2.2:2000
MSPDebug version 0.16 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2011 Daniel Beer
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

Looking up 10.0.2.2...
Connecting to 10.0.2.2:2000...

Available commands:
    =         delbreak  gdb       load      opt       reset     simio
    alias     dis       help      locka     prog      run       step
    break     erase     hexout    md        read      set       sym
    cgraph    exit      isearch   mw        regs      setbreak

Available options:
    color           gdb_loop        iradix
    fet_block_size  gdbc_xfer_size  quiet

Type "help " for more information.
Press Ctrl+D to quit.

(mspdebug)
```

Sweet action, we've connected to our LaunchPad through the network! You can now do everything just like normal.
For debugging, start msp430-gdb:
```
msp430-gdb
```

At the prompt enter:
```
(gdb) target remote 10.0.2.2:2000
```

You can now do all the regular GDB stuff.
If you've got a better way of going about this, I'd love to hear it!

### Automation:

It can be quite annoying to leave the VM and reconnect mspdebug to the LaunchPad from the host, so I've got a little script that will wait 1 second before reconnecting, so you'll be able to kill it with Ctrl + C
```
#!/bin/bash

while [ 1 ]
do
    mspdebug rf2500 gdb 2000
    sleep 1
done
```

I saved it as /usr/from\_source/bin/tilaunchpad right next to the mspdebug binary. You can stay constantly connected to it through mspdebug or msp430-gdb and type 'run' or 'continue' to execute like normal.

Happy Hacking!

### Appendix:

Here is what syslog output looks like when connected directly to the VM:
```
Mar 16 17:07:57 linux kernel: [ 318.149193] usb 2-2: new full speed USB device number 7 using ohci_hcd
Mar 16 17:08:12 linux kernel: [ 333.296388] usb 2-2: device descriptor read/64, error -110
Mar 16 17:08:27 linux kernel: [ 348.565460] usb 2-2: device descriptor read/64, error -110
Mar 16 17:08:28 linux kernel: [ 348.804490] usb 2-2: new full speed USB device number 8 using ohci_hcd
Mar 16 17:08:43 linux kernel: [ 363.952590] usb 2-2: device descriptor read/64, error -110
Mar 16 17:08:58 linux kernel: [ 379.225328] usb 2-2: device descriptor read/64, error -110
Mar 16 17:08:58 linux kernel: [ 379.464388] usb 2-2: new full speed USB device number 9 using ohci_hcd
Mar 16 17:09:03 linux kernel: [ 384.499668] usb 2-2: device descriptor read/8, error -110
Mar 16 17:09:09 linux kernel: [ 389.617160] usb 2-2: device descriptor read/8, error -110
Mar 16 17:09:09 linux kernel: [ 389.856417] usb 2-2: new full speed USB device number 10 using ohci_hcd
Mar 16 17:09:14 linux kernel: [ 394.903343] usb 2-2: device descriptor read/8, error -110
Mar 16 17:09:19 linux kernel: [ 400.022096] usb 2-2: device descriptor read/8, error -110
Mar 16 17:09:19 linux kernel: [ 400.124228] hub 2-0:1.0: unable to enumerate USB device on port 2
```

Here is what mspdebug said after the enumeration failure:
```
$ sudo ./mspdebug rf2500
MSPDebug version 0.19 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2012 Daniel Beer
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

usbutil: unable to find a device matching 0451:f432
```

I can't remember what I did to get it to show up: (I'll post an update if I do)
```
Mar 16 17:28:09 linux kernel: [  532.781053] usb 1-2: new full speed USB device number 5 using ohci_hcd
Mar 16 17:28:09 linux mtp-probe: checking bus 1, device 5: "/sys/devices/pci0000:00/0000:00:06.0/usb1/1-2"
Mar 16 17:28:09 linux kernel: [  533.090225] cdc_acm 1-2:1.0: This device cannot do calls on its own. It is not a modem.
Mar 16 17:28:09 linux kernel: [  533.090232] cdc_acm 1-2:1.0: No union descriptor, testing for castrated device
Mar 16 17:28:09 linux kernel: [  533.090255] cdc_acm 1-2:1.0: ttyACM0: USB ACM device
Mar 16 17:28:19 linux kernel: [  543.174047] generic-usb 0003:0451:F432.0004: usb_submit_urb(ctrl) failed
Mar 16 17:28:19 linux kernel: [  543.174114] generic-usb 0003:0451:F432.0004: timeout initializing reports
Mar 16 17:28:19 linux kernel: [  543.174276] generic-usb 0003:0451:F432.0004: hiddev0,hidraw1: USB HID v1.01 Device [Texas Instruments Texas Instruments MSP-FET430UIF] on usb-0000:00:06.0-2/input1
Mar 16 17:28:20 linux mtp-probe: bus: 1, device: 5 was not an MTP device
Mar 16 17:28:20 linux kernel: [  544.173466] usb 1-2: usbfs: USBDEVFS_CONTROL failed cmd mtp-probe rqt 128 rq 6 len 255 ret -110
```

Once the LaunchPad showed up in /dev/ttyACM0 here's the mspdebug output:
```
$ sudo ./mspdebug rf2500
MSPDebug version 0.19 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2012 Daniel Beer
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

Trying to open interface 1 on 003
rf2500: warning: can't detach kernel driver: No data available
Initializing FET...
rf2500: can't send data: Resource temporarily unavailable
fet: open failed
Trying again...
Initializing FET...
rf2500: can't send data: Resource temporarily unavailable
fet: open failed
```

Here's the segfault using msp430-gdb from the repository:
```
$ msp430-gdb
GNU gdb (Linaro GDB) 7.3
Copyright (C) 2011 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "--host=x86_64-linux-gnu --target=msp430".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>.
(gdb) target remote 10.0.2.2:2000
Remote debugging using 10.0.2.2:2000
Segmentation fault
```


















