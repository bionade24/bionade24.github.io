---
title:       "Extend SailfishOS rootfs with a hidden, unused partition."
subtitle:    "The best way to get space when you can't install an OS update."
description: "How to guide to extend the rootfs' logical volume with an unused Android partition."
date:        2022-04-13
author:      "Oskar Roesler"
image:       ""
tags:        ["SailfishOS", "Xperia", "Android", "Smartphone"]
categories:  ["Tech" ]
---

The intial problem
-

To update to the latest SailfishOS, you'll need a bit over 500MB free space. Shouldn't be a problem oon a modern phone, should it? Well, for whatever reason Jolla gives the rootfs only 2.5Gb of space and adresses the rest to the home partition. So when you've installed a few more apps, you'll sooner or later run out of space. You could remove a bunch of apps and try to install them again after the update, but you'll face the same problem with the next update again. Shrinking the home logical volume would be an option, but maybe we also run low on space on this partition.

Luckily, most Sailfish X devices have a partition left over from Android which is completely unused. Newer Android phones have partitions labled `system_a` and `system_b`, which were used by deliver OS updates as full partitions without overwriting the used one. On next reboot, Android just switches the partition to boot from. SailfishOS only uses `system_b`, as it doesn't provide kernel updates.  So let's use `system_a` instead of wasting its space.


But does this work on all SailfishOS devices?
-

No, it doesn't. If your phone only has a `/dev/disk/by-partlabel/system` partition. If it has unused spaces, you could shrink it and create another partition to do the same, but I don't know for sure. If your system has a `/dev/disk/by-partlabel/system_a` and a `/dev/disk/by-partlabel/system_b`, this guide will work one by one for you. For example, the Xperia X has only a `system` partition and the Xperia XA2 does have a `system_a` and a `system_b` partition. so it won't work on the X but will work on the XA2. I don't know how it's on newer devices, if you find out, please let me know.

Additionally, you wont't be able to reflash SailfishOS without reflashing Android first. I'll release a blogpost how you can do that without having to install Windows and the Sony EMMA tool soon. Updating SailfishOS without reflashing it is absolutely fine.

Preperations
-

As always, it's good to have a backup first, in case you miss up somehow anyway. Since SailfishOS is effectively Linux, I'm using [restic](https://restic.org), but you can use whatever tool you like. Additionally, you need shell access on your Sailfish device which you get by enabling the development mode, but that's it.


Let's do it.
-

First of all, change to root user, which is reuquired for all operations. Do that by entering `devel-su` in the command line. Afterwards, you'll need to enter your root password, which is changable in the same place as where the development mode can be activated.


### Add the partition to LVM ###

To start, you have to add `/dev/disk/by-partlabel/system_a` to the [Logical Volume Manager](https://wiki.archlinux.org/title/LVM).
1. Enter `pvcreate /dev/disk/by-partlabel/system_a` to prepare the partition for LVM.
2. Secondly, type `vgextend sailfish /dev/disk/by-partlabel/system_a` to add the partition to the `sailfish` volume group.

### Extend the sailfish root filesystem ##

Now, the space of the sailfish volume group is extended by the size of the `system_a` partition and we can continue with adding the new free space of the volume group to the root volume.
1. This is done with `lvextend sailfish/root -l +100%Free`, which tells LVM to assign all free space in the `sailfish` volume group to the `root` volume.
2. In the end, we resize the root filesystem to the new size of the `root` volume by entering `resize2fs /dev/mapper/sailfish-root`. This can be done while the filesystem is mounted, which is a nice feature of ext4.

Now you can proceed with installing apps or starting the system update!

