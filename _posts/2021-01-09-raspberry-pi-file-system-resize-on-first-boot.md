---
title: How the Raspberry Pi automatic file system expansion works
author: Gijs
date: 2021-01-09 20:00:00 +0200
categories: [linux]
tags: [linux, raspberry-pi]
pin: true
image: /assets/img/rpi-fs-resize/rpi-resize.png
---

When you boot Raspberry Pi OS for the first time, it performs an automatic file system expansion. This process makes sure that the file system is expanded to the size of the SD card.
Because of this process, it doesn't matter to what size SD card you write the Raspberry Pi OS image.

But exactly how does this process work?

As far as I can tell the Raspberry Pi OS (formerly knows as Raspbian) expands its file system [since 2016](https://www.raspberrypi.org/blog/another-update-raspbian).

> When flashing a new Raspbian image, the file system will automatically be expanded to use all the space on the card when it is first booted.

Let's see how this is implemented.

# Mounting the Raspberry Pi OS image locally

We'll be mounting the latest Raspberry Pi OS image locally, so we can inspect it before the file system expansion has run.

You can download the latest Raspberry Pi OS image [here](https://www.raspberrypi.org/software/operating-systems) (release date December 2nd 2020 at the time of writing).

You'll need sudo access to follow along. Also, you will need [kpartx](https://linux.die.net/man/8/kpartx) installed.

```terminal
$ sudo su
# kpartx -av 2020-12-02-raspios-buster-armhf-lite.img
add map loop24p1 (253:2): 0 524288 linear 7:24 8192
add map loop24p2 (253:3): 0 3096576 linear 7:24 532480
```

Note: the number after `loop` (in the example `24`) can be different.

At this point two new loop devices will have been created in `/dev/mapper/`:

```terminal
# ls /dev/mapper/loop24*
/dev/mapper/loop24p1  /dev/mapper/loop24p2
```

The first loop device (`loop24p1`) is the `/boot` partition, the second loop device is the `/` (root) partition.

You can list the partitions on a running Raspberry Pi using (for example) this command:

```terminal
$ mount -l | grep mmc
/dev/mmcblk0p2 on / type ext4 (rw,noatime) [rootfs]
/dev/mmcblk0p1 on /boot type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro) [boot]
```

This confirms that partition 1 is the boot partition, and partition 2 is the root partition.

We can now mount both these loop devices locally:

```terminal
# mkdir /tmp/rpi-boot
# mount /dev/mapper/loop25p1 /tmp/rpi-boot/

# mkdir /tmp/rpi-root
# mount /dev/mapper/loop25p2 /tmp/rpi-root/
```

When you are done, remember to reverse the steps we just took:

```terminal
# umount /tmp/rpi-boot/ /tmp/rpi-root/
# kpartx -d 2020-12-02-raspios-buster-armhf-lite.img
```

# How the automatic file system expansion is implemented

Now we can inspect the Raspberry Pi OS file system, without it having been booted for the first time.

The file called `cmdline.txt` is what we want.

The [Raspberry PI documentation](https://www.raspberrypi.org/documentation/configuration/cmdline-txt.md) offers some information about this file.

> The Linux kernel accepts a command line of parameters during boot. On the Raspberry Pi, this command line is defined in a file in the boot partition, called cmdline.txt.

Let's have a look at its contents:

```terminal
# cat /tmp/rpi-boot/cmdline.txt 
console=serial0,115200 console=tty1 root=PARTUUID=067e19d7-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait quiet init=/usr/lib/raspi-config/init_resize.sh
```

The interesting part is `init=/usr/lib/raspi-config/init_resize.sh`.

The [Linux documentation](https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt) has the following to say about the `init` parameter:

```text
init=	[KNL]
	Format: <full_path>
	Run specified binary instead of /sbin/init as init
	process.
```

The Raspberry Pi OS image is telling the Linux kernel to run a specific script (`/usr/lib/raspi-config/init_resize.sh`) instead of the regular init process `/sbin/init` (which is actually a symlink to `/lib/systemd/systemd`).

The script can be found at [github.com/RPi-Distro/raspi-config](https://github.com/RPi-Distro/raspi-config/blob/master/usr/lib/raspi-config/init_resize.sh)

In essence, it uses `parted` to perform the partition resize.

Near the end of the script something interesting happens, the script removes the `init=` parameter from `/boot/cmdline.txt` using `sed -i 's| init=/usr/lib/raspi-config/init_resize\.sh||' /boot/cmdline.txt`.
This makes perfect sense, since if this is not done the Raspberry Pi would reboot and run the `init_resize.sh` again, and again, and again...

As we read in the Linux documentation, `/sbin/init` (which is a symlink to systemd) will be run when the `init=` parameter is omitted.
After the Raspberry Pi is rebooted it will start up as usual, but now with its file system having been expanded to make full use of the available space on the SD card.

# Resources
- [https://www.raspberrypi.org/blog/another-update-raspbian](https://www.raspberrypi.org/blog/another-update-raspbian)
- [https://www.raspberrypi.org/software/operating-systems](https://www.raspberrypi.org/software/operating-systems)
- [https://linux.die.net/man/8/kpartx](https://linux.die.net/man/8/kpartx)
- [https://www.raspberrypi.org/documentation/configuration/cmdline-txt.md](https://www.raspberrypi.org/documentation/configuration/cmdline-txt.md)
- [https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt](https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt)
- [https://github.com/RPi-Distro/raspi-config/blob/master/usr/lib/raspi-config/init_resize.sh](https://github.com/RPi-Distro/raspi-config/blob/master/usr/lib/raspi-config/init_resize.sh)

[^footnote]: The footnote source.
