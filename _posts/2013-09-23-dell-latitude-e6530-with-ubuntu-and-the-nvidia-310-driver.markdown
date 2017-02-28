---
layout: post
title: Dell Latitude E6530 with Ubuntu and the Nvidia 310 driver
date: '2013-09-23 10:00:00'
tags:
- ubuntu-linux-dell-latitude-nvidia
---

![Dell 6430](/images/latitude_6430_backlit_keyboard.jpg)

Here’s a list of some of the fixes I made to overcome most of the hassles encountered so people can use their system optimally with that driver.


Installing the Nvidia Driver (310 Experimental)
---

Before installing the driver, I highly recommend installing the kernel headers of your fully updated system. Not doing so will result in resolution problems and Unity not starting.

You can find out what kernel version your are currently running by issuing the command in the terminal:

```
~$ uname -a
Linux neon-flower 3.5.0-25-generic #39-Ubuntu SMP Mon Feb 25 18:26:58 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
```

And installing the appropriate kernel headers:

```
~$ sudo apt-get install linux-headers-3.5.0-25
```

Now you can proceed with installing the experimental Nvidia driver through *Software Sources >> Additional Drivers*.

**Note: if you happen to be stuck after a reboot because you didn’t install the kernel headers, open a terminal with CTRL+ALT+T, and do the following:**

1. sudo apt-get remove **nvidia-experimental-310**
2. follow the normal steps above to install the kernel headers
3. sudo apt-get install **nvidia-experimental-310**
4. sudo reboot

<p>&nbsp;</p>

Fix the Brightness keys not working with the 310 Nvidia Driver
---

Open the Nvidia X Server Settings app (nvidia-settings from a terminal).

![Nvidia Settings](/images/nvidia_settings.png)

You need to let the Nvidia program write the xorg.conf file for you, as by default none is generated or present in /etc/X11/

Now you can open that file as root, either by using

```
sudo nano /etc/X11/xorg.conf
```

or

```
gksu gedit /etc/X11/xorg.conf
```

At the **Section "Screen"**, add the following line:

```
Option            "RegistryDwords" "EnableBrightnessControl=1"
```

Either you logout and back in, or reboot, and the Brightness keys should work again.


Disable the Nvidia logo after boot and before the login screen
---

Take a look at the previous section if you haven’t generated an xorg.conf file yet.

Done that? Ok, find **Section "Device"** and add the following line:

```
Option           "NoLogo" "True"
```

The Nvidia logo should not appear any more when booting.


Fix boot resolution (Plymouth)
---

The script in the article found [here](http://www.webupd8.org/2010/10/script-to-fix-ubuntu-plymouth-for.html) (courtesy of d0rkye and kyleabaker) does all the necessary steps to fix the resolution problem when booting.

All you need to do is follow the instructions and reboot:

```
wget http://launchpadlibrarian.net/121675171/fixplymouth
chmod +x fixplymouth
./fixplymouth
```
