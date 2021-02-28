---
title: How the Raspberry Pi enables SSH when /boot/ssh.txt is present
author: Gijs 
date: 2021-02-28 13:00:00 +0100
categories: [linux]
tags: [linux, raspberry-pi]
pin: true
---

# Note on security

If you decide to enable SSH, remember to **change the default password!**

Preferably you disable password-based authentication and instead setup key-based authentication.

See [the official Raspberry Pi documentation on Security](https://www.raspberrypi.org/documentation/configuration/security.md)

# What does it do?

In Raspberry Pi OS, SSH is **disabled** by default. You can manually enable SSH for example by using `raspi-config`.
There is also a way to automatically enable SSH, this can come in handy when you don't have a monitor at hand.

This is done by putting a special file called `ssh` or `ssh.txt` on the boot partition. This partition will be
mounted as `/boot` when the Raspberry Pi boots. The boot partition is formatted as a FAT file system and can be read by
and written to by all operating systems.

When the Raspberry Pi boots it looks for a file called `/boot/ssh` or `/boot/ssh.txt`. If found, SSH will be enabled,
and the file will be deleted. Note that the contents of the file don't matter, it can be left empty.

# How exactly is this implemented?

Now that we know what the Raspberry Pi looks for and what it does when it finds the file, we can look at how this is implemented.

The 'enable SSH when /boot/ssh exists' behavior is actually a Systemd
service: [`/lib/systemd/system/sshswitch.service`](https://github.com/RPi-Distro/raspberrypi-sys-mods/blob/master/debian/raspberrypi-sys-mods.sshswitch.service).

```terminal 
[Unit]
Description=Turn on SSH if /boot/ssh is present
ConditionPathExistsGlob=/boot/ssh{,.txt}
After=regenerate_ssh_host_keys.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c "systemctl enable --now ssh && rm -f /boot/ssh ; rm -f /boot/ssh.txt"

[Install]
WantedBy=multi-user.target
```

Some interesting details:

[`ConditionPathExistsGlob`](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#ConditionPathExistsGlob=)
is used to check if at least one file is present that matches the given globbing pattern. The globbing pattern used
is `{,.txt}`. The terms of a globbing pattern are separated by commas, so this is actually a globbing pattern with 2
terms: the first one is empty, and the second is `.txt`.

[`ExecStart`](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=) is only run
when `ConditionPathExistsGlob` succeeds, it executes the actual command that enables SSH (`systemctl enable --now ssh`),
afterwards it removes the
`/boot/ssh` and `/boot/ssh.txt` files.

# Resources

- [https://www.raspberrypi.org/documentation/configuration/boot_folder.md](https://www.raspberrypi.org/documentation/configuration/boot_folder.md)
- [https://www.raspberrypi.org/documentation/remote-access/ssh](https://www.raspberrypi.org/documentation/remote-access/ssh)
- [https://www.raspberrypi.org/documentation/configuration/security.md](https://www.raspberrypi.org/documentation/configuration/security.md)
- [https://github.com/RPi-Distro/raspberrypi-sys-mods/blob/master/debian/raspberrypi-sys-mods.sshswitch.service](https://github.com/RPi-Distro/raspberrypi-sys-mods/blob/master/debian/raspberrypi-sys-mods.sshswitch.service)
- [https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm)

[^footnote]: The footnote source.
