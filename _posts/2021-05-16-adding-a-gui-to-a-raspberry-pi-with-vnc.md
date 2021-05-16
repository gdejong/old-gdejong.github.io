---
title: Adding a GUI to a Raspberry Pi running Raspberry Pi OS Lite
author: Gijs
date: 2021-05-16 12:00:00 +0100
categories: [linux]
tags: [linux, raspberry-pi]
pin: true
image: /assets/img/gui/desktop.png
---

You may have installed **Raspberry Pi OS Lite** on your Raspberry Pi, which does not ship with a GUI.
If you change your mind and want a GUI after all, you can just install one.

In this exercise we will install the PIXEL desktop, which is what you would get when installing the non-Lite version of Raspberry Pi OS.

# Preparation

I assume you have a Raspberry Pi running Raspberry Pi OS Lite, and you have SSH access (or a monitor/keyboard attached).

# Installing the GUI

First, make sure to update your packages:

```terminal
sudo apt-get update
```

Now install the GUI:

```terminal
sudo apt-get install raspberrypi-ui-mods
```

Simple, right?

Now attach a monitor to your Raspberry Pi and reboot.

```terminal
sudo reboot
```

You will now have a GUI to work with.

# Adding VNC support

A monitor is not always convenient. Let's work around that by adding VNC support, which will allow us to use the GUI
from our own computer remotely.

We will use `raspi-config` noninteractively to install and enable the VNC server:

```terminal
sudo raspi-config nonint do_vnc 0
```

Now reboot your Raspberry Pi.

```terminal
sudo reboot
```

Install [Real VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) on your computer and connect to your Raspberry Pi.

# Fixing the resolution

When you unplug the HDMI cable and reboot, you might be surprised by the resolution. It is only using a resolution of 656 x 416 pixels.

```terminal
xdpyinfo | grep dimensions
```

![Low resolution](/assets/img/gui/low-res.png)

Let's change that to a more HD resolution. We can control the resolution by changing a few settings in `/boot/config.txt`.

Before you start, backup the file before we make any modifications to it:
```terminal
sudo cp /boot/config.txt /boot/config.txt.bak
```

The [Raspberry Pi documentation on video settings](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md) covers this is great detail.

We want to change these settings:

| Setting                      | Value             | Effect                                           |
|:-----------------------------|:------------------|-------------------------------------------------:|
| hdmi_force_hotplug           | 1                 | Force HDMI output even when there is no attached |
| hdmi_group                   | 2                 | Use the DMT (Display Monitor Timings) standard   |
| hdmi_mode                    | 82                | Choose 1080p HDMI output mode                 |


![config changes](/assets/img/gui/config.png)

Now reboot, and you will see a 1080p resolution!

```terminal
sudo reboot
```

![1080p resolution](/assets/img/gui/high-res.png)

# Resources
- [https://www.raspberrypi.org/software/operating-systems/](https://www.raspberrypi.org/software/operating-systems/)
- [https://www.raspberrypi.org/documentation/remote-access/vnc/](https://www.raspberrypi.org/documentation/remote-access/vnc/)
- [https://www.raspberrypi.org/forums/viewtopic.php?t=133691](https://www.raspberrypi.org/forums/viewtopic.php?t=133691)
- [https://www.raspberrypi.org/documentation/configuration/config-txt/video.md](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md)
- [https://github.com/RPi-Distro/raspberrypi-ui-mods](https://github.com/RPi-Distro/raspberrypi-ui-mods)
- [https://www.realvnc.com/en/connect/download/viewer/](https://www.realvnc.com/en/connect/download/viewer/)
- [https://github.com/RPi-Distro/raspi-config/blob/master/raspi-config](https://github.com/RPi-Distro/raspi-config/blob/master/raspi-config)

[^footnote]: The footnote source.
