---
title: pcDuino3Nano - first impressions
layout: post
---
I recently purchased a [pcDuino3Nano] board, because I find it to be a better overall board than a Raspberry Pi or other similar boards, and very reasonably priced. Plus, it has SATA (which is a big plus for me) and accepts Arduino shields. It's still quite new and documentation is not great, but most of the documentation for its bigger brother ([pcDuino3]) also applies to the "nano" version.

<!--more-->

* TOC
{:toc}

# What you get

Pretty much this:

![pcDuino3Nano](http://www.pcduino.com/wp-content/uploads/2014/07/pcduino3nano.jpg)

The board comes in a simple package, without an USB cable, an Ethernet cable or a power supply (which is perfectly reasonable given its price).

# What you'll need

 Required items:

* a power supply . The board gets its power via a microUSB connector, clearly labeled **DC_5V** on the back of the board. [Official specs][pcDuino3Nano] suggest a 5V, 2A power supply, but you won't need that kind of juice unless you connect a lot of peripherals (for example a HDD). With only Ethernet or a WiFi USB dongle connected, I was able to power the board from one of my laptop's USB ports without any issues.
* a micro USB cable for the power supply. If powering from a PC or laptop, this will be your standard micro USB cable. 
* an Ethernet cable, if you want to connect your board to the Internet using the Ethernet connection (also recommended for the initial setup of the board).
* a way to connect your board to the network via Ethernet. This will usually be an Ethernet socket in your home router. Your home router must also be able to function as a DHCP server (most of them are by default).
 
Optional items:

* a HDMI-HDMI cable if you want to connect the board to a TV or a monitor. I won't be covering this part; my aim is to use the board as a headless media server, so I'll ignore the HDMI connection as much as possible.
* The board comes with 4GB of built-in storage, which is pre-programmed with an Ubuntu image, so you don't need to prepare an image on a microSD card and boot from that (although that's also possible). However, it's very likely that you'll need a SD card anyway, for things like updating the kernel or installing another distro, so:
    * A microSD card, at least 4GB (but up to 32GB). I recommend at least a Class 6 card, although Class 4 or even Class 2 might work.
    * An USB card reader/writer for your PC (plenty of options to choose from here).
* Speaking of storage: if you think you'll want to connect a SATA HDD to the board, you'll want to get the [special SATA cable].

# Booting the board

1. Connect the board to your router and power it up
2. Wait until **LED3** and **LED4** (close to the audio connector) are both lit. 
3. The board is pre-programmed to get an IP address from your router via DHCP, so use your router's administrative interface to find this IP (the board name is "ubuntu").

At this point you should be able to use ssh to connect to the board (the default username and password are **ubuntu** and **ubuntu**):

{% highlight sh %}
$ ssh ubuntu@ip.of.the.board -v
{% endhighlight %}
Congrats! Your board is now up and running, and you didn't even have to do much to make that happen.

<div class="info">
  <h4>Reminder</h4>
  <p>Getting to a command line via ssh is not the "regular" way to use this board. That would be to hook the board to your TV using a HDMI cable, connect an USB keyboard and/or mice and enjoy the nice graphical interface, which is quite user-friendly. But again, I am not interested in this, so I won't be covering the user interface.</p>
</div>

# Getting more information

The [official specs][pcDuino3Nano] already have quite a bit of information about the board, but let's see what Linux can tell us:

{% highlight sh %}
ubuntu@ubuntu:~$ uname -a
Linux ubuntu 3.4.79+ #5 SMP PREEMPT Wed Oct 15 14:06:46 CST 2014 armv7l armv7l armv7l GNU/Linux
ubuntu@ubuntu:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Linaro
Description:	Linaro 12.07
Release:	12.07
Codename:	precise
{% endhighlight %}

Ubuntu 12.07 with kernel 3.4.79. These can probably be updated with some effort.

{% highlight sh %}
ubuntu@ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/nandd      3.6G  1.4G  2.1G  39% /
none            438M  4.0K  438M   1% /dev
none            438M  4.0K  438M   1% /tmp
none             88M  200K   88M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            438M     0  438M   0% /run/shm
none            438M     0  438M   0% /var/tmp
none            438M  624K  438M   1% /var/log
{% endhighlight %}

The internal 4G NAND memory has 3.6G reserved for the file system (**/dev/nandd**). The default Ubuntu instalation uses 1.4G of that, which is understandable, given that it comes with a graphical interface (however, we'll work on getting a smaller file system in a later post). 

{% highlight sh %}
ubuntu@ubuntu:~$ cat /etc/apt/sources.list
deb http://ports.ubuntu.com/ubuntu-ports/ precise main universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ precise main universe
deb http://ports.ubuntu.com/ubuntu-ports/ precise-security main universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ precise-security main universe
deb http://ports.ubuntu.com/ubuntu-ports/ precise-updates main universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ precise-updates main universe
deb http://www.wiimu.com:8020/pcduino/ pcduino main
deb-src http://www.wiimu.com:8020/pcduino/ pcduino main
{% endhighlight %}

Pacakges coming mostly from the Ubuntu ports repositories. The last 2 lines that reference **wiimu.com** never worked for me, but that doesn't seem to have much of an impact.

{% highlight sh %}
ubuntu@ubuntu:~$ ll /allwinner/
total 12
drwxrwxrwx  3 root root 4096 Aug  7  2014 ./
drwxr-xr-x 25 root root 4096 Jan  1 00:00 ../
drwxr-xr-x  5 root root 4096 Aug  7  2014 xbmc-pvr-binhf/
{% endhighlight %}

There seems to be a board-specific [XBMC] installation in the above directory. I haven't looked at it yet, but I'll probably do that in the future.

{% highlight sh %}
ubuntu@ubuntu:~$ ps aux | grep vnc
root       745  0.0  0.6  21536  5488 ?        Ss   00:00   0:02 x11vnc -display :0 -auth /var/run/lightdm/root/:0 -forever -bg -o /var/log/x11vnc.log -rfbauth /etc/x11vnc.pass -rfbport 5900
{% endhighlight %}

A VNC server is started by default on boot, so you can access the GUI even if you don't use the HDMI connection. Default port is **5900**, default password is **ubuntu**. More details can be found [here][vnc].

# Wrap-up

Overall, I'm quite impressed with this board. It offers a lot of bang for the buck (I got it for $35 from [LinkSprite] with a discount of a few dollars), it's easy to use and it feels quick. Basically, for the same price as a Raspberry Pi, you get a board with a better CPU, more RAM, integrated NAND memory, SATA, proper audio output, IR receiver and handy Arduino compatible expansion headers. Of course, price and hardware aren't the only things to be considered when looking for an embedded Linux board, but for my needs, the choice between this board and the Raspberry Pi was a pretty obvious one.

[pcDuino3Nano]: http://www.linksprite.com/linksprite-pcduino3-nano/
[pcDuino3]:http://www.linksprite.com/linksprite-pcduino3/
[Volumio]: http://volumio.org
[special SATA cable]: http://store.linksprite.com/sata-cable-with-power-connector-for-pcduio3/
[xbmc]: http://xbmc.org/
[vnc]: http://www.element14.com/community/thread/26532/l/quick-start-of-pcduino-without-a-hdmi-monitor-and-serial-debug-cable?displayFullThread=true
[LinkSprite]: http://store.linksprite.com/pcduino3-nano/

