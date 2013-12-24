---
layout: post
title: "Configure Time Machine to Save Backups to a TrueCrypt Volume"
date: 2013-05-13 11:19:00 -0600
comments: true
categories: 
alias: /2013/05/23/configure-time-machine-to-save-backups-to-a-truecrypt-volume/
---
I have an encrypted 1TB portable hard drive, and I wanted to use that as a Time Machine backup.

TL;DR:
```
$ sudo tmutil setdestination /Volumes/TC\ Backup/
```
<!-- more -->

{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-23-at-1-17-15-am.png %}

But for some reason, Time Machine doesn't like my TrueCrypt Volume.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-23-at-1-17-18-am.png %}

I have the TrueCrypt Volume mounted.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-23-at-11-30-24-am.png %}

{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-2-57-37-pm.png %}

Here is what it looks like, it's just a regular volume.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-2-56-24-pm.png %}

Why doesn't Time Machine allow me to use it?
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/timemachineyuno.jpg %}

Since the GUI won't let me choose the TrueCrypt volume, we can use Time Machine's command line tool to set the backup drive.
You can see what commands `tmutil` supports by looking at its manual:
```
$ man tmutil
```

The commands we're interested in are:
```
setdestination
destinationinfo
removedestination
```

We're going to use `setdestination` to force Time Machine to use the TrueCrypt Volume to store backups, `destinationinfo` to make sure the correct drive is set, and `removedestination` if the wrong destination was specified.

Let's find out where our volume is mounted:
```
clay@laptop:1.9.3: ~ $ mount
/dev/disk1 on / (hfs, local, journaled)
devfs on /dev (devfs, local, nobrowse)
map -hosts on /net (autofs, nosuid, automounted, nobrowse)
map auto_home on /home (autofs, automounted, nobrowse)
TrueCrypt@osxfuse0 on /private/tmp/.truecrypt_aux_mnt1 (osxfusefs, synchronous, nobrowse)
/dev/disk3 on /Volumes/TC Backup (hfs, local, nodev, nosuid, journaled, noowners)
```

My volume is named `TC Backup`, so I'm going to issue the command:
```
richardsonclay@yo:1.9.3: ~ $ sudo tmutil setdestination /Volumes/TC\ Backup/
```

Replace `TC\ Backup` with the name of your mounted TrueCrypt volume.

We can check to see if the information is correct:
```
richardsonclay@yo:1.9.3: ~ $ tmutil destinationinfo
====================================================
Name          : TC Backup
Kind          : Local
Mount Point   : /Volumes/TC Backup
ID            : 12345678-1234-ABCD-ABCD-1234ABCD1234
```

If it's not, you can use `removedestination` to remove the destination:
```
richardsonclay@yo:1.9.3: ~ $ tmutil removedestination 12345678-1234-ABCD-ABCD-1234ABCD1234
```

You should be able to see the drive in the Time Machine Preferences.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-06-05-pm.png %}

Turn Time Machine on.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-06-27-pm.png %}

And start your backup!
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-06-47-pm.png %}

{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-12-18-pm.png %}

Time Machine has created the `Backups.backupdb` folder, and is copying stuff over to it!
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-08-09-pm.png %}

Here's what the drive information looks like now that it is being used for Time Machine backups.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-07-49-pm.png %}

Be aware that backups might be CPU intensive, as TrueCrypt needs to transparently encrypt all of the data being copied over.
{% img /images/configure-time-machine-to-save-backups-to-a-truecrypt-volume/screen-shot-2013-05-22-at-3-14-49-pm.png %}

