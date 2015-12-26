---
title: pcDuino3Nano - configuring wireless networking
layout: post
permalink: pcduino3nano-configuring-wireless-networking
comments: true
categories: [pcduino, linux]
---
In my [previous post][nav-installarch] I've covered the installation procedure for [Arch Linux] on the [pcDuino3Nano] board. The procedure covers wired connectivity, but not wireless. This post will show how to enable wireless networking in [Arch Linux] quickly and painlessly.

<!--more-->

* TOC
{:toc}

# What you'll need

You need an USB wireless adapter which is supported by your kernel. I'm using an adapter based on the RTL8188 chip, namely [this one](http://www.ebay.com/itm/EDUP-EP-N8508-150Mbps-802-11n-Wireless-LAN-USB-2-0-Ultra-Mini-Adapter-small-/331126556199?pt=US_USB_Wi_Fi_Adapters_Dongles&hash=item4d18ae4627), which works very well. Other adapters based on the same chip will likely work equally well. Adapters based on other chips might work too, but the chip must be supported by the kernel.

You also need a wireless access point to connect to, preferably one that is configured to use WPA encryption, for security reasons.

# Setting up

Plug the adapter into one of the USB ports on the board and check if it's detected:

{% highlight sh %}
[root@alarm ~]# lsusb
Bus 003 Device 002: ID 0bda:8176 Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter
[truncated]
{% endhighlight %}

If the adapter is not detected at this point, but you can verify that it works (for example, by checking if it is detected on a PC), your power supply might not supply enough current to the board. Get a power supply that can output more current and retry.

Check that a network interface was created for your adapter by running **ifconfig -a**. It should list a *wlan0* interface. If that's not the case, you need to fix this problem before continuing this tutorial. The most likely cause is that **udev** can't load the corresponding kernel module for your adapter, but that isn't always the case. I'm not going to cover fixing this particular problem here (remember, google is your friend).

If the wireless access point you're going to connect to is secured (and hopefully it is), you must install **wpa_supplicant**:

{% highlight sh %}
# pacman -S wpa_supplicant
{% endhighlight %}

Next, you need to create a **netctl** profile for your wireless connection. There are a number of example profiles provided in **/etc/netctl/examples**, covering open network (no authentication), WEP and WPA authentication. I'll assume that your access point uses WPA authentication and runs a DHCP server, so copy the corresponding example to **/etc/netctl**:

{% highlight sh %}
# cp /etc/netctl/examples/wireless-wpa /etc/netctl/wirelesswpa
{% endhighlight %}

Edit the **/etc/netctl/wirelesswpa** to reflect the name (ESSID) of you access point and the WPA password, then check your configuration:

{% highlight sh %}
# netctl start wirelesswpa
{% endhighlight %}

If everything goes well, at this point you'll be connected to your access point and the interface will have a DHCP assigned address. If that's the case, you can made this change permanent (persistent after reboot):

{% highlight sh %}
# netctl enable wirelesswpa
{% endhighlight %}

That's all, your wireless connection is now properly configured.

# Dealing with two network interfaces

At this point, there are two network interfaces which are automatically configured on startup: wired (Ethernet, we configured it in the [previous post][nav-installarch]) and wireless. It makes little sense for them to be active at the same time. There are a few ways to fix this:

1. disable the automatic configuration of the wired connection using **netctl stop ethdhcp** and **netctl disable ethdhcp**. This works fine, but if you remove the wireless adapter and plug in the Ethernet cable, you'll need to re-enable the interface manually using **netctl start ethdhcp** and **netctl enable ethdhcp**.
2. use **ifplugd**. This service will automatically configure the Ethernet interface when a cable is plugged and will automatically disable the interface when the cable is removed.

Using **ifplugd** is very easy:

{% highlight sh %}
# netctl disable ethdhcp
# systemctl enable netctl-ifplugd@eth0.service
{% endhighlight %}

# Where to go from here

This tutorial covers only the simplest methods one can use to configure the network connections. Much more complex configurations are possible. For more details on the subject, the links below are a good starting point:

* [Network configuration page on ArchWiki](https://wiki.archlinux.org/index.php/Network_configuration)
* [netctl page on ArchWiki](https://wiki.archlinux.org/index.php/netctl)

[pcDuino3Nano]: http://www.linksprite.com/linksprite-pcduino3-nano/
[nav-installarch]: /pcduino3nano-installing-arch-linux
[nav-firstimpressions]: /pcduino3nano-first-impressions
[Arch Linux]: http://www.archlinux.org

