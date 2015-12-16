---
title: pcDuino3Nano - installing Arch Linux
layout: post
permalink: pcduino3nano-installing-arch-linux
---
In my [previous post][nav-firstimpressions], we had a first look at the [pcDuino3Nano] board and booted it up using the pre-programmed firmware, which turned out to be a simple and intuitive process. In today's post, I'll explain how to install [Arch Linux] on this board.

<!--more-->

* TOC
{:toc}


<div class="info">
    <h4>Update 14.12.2015</h4>
    <p>This post was updated on 14.12.2015. Look for "Update" paragraph and text boxes (like this one).</p>
</div>

# Update 14.12.2015

It is now possible to install Arch Linux by following the instruction on the [Arch Linux site][arch-pcduino-install], just replace `pcduino3` with `pcduino3-nano` in the URLs from the installation instructions. However, those instructions use the mainline kernel, which is missing some drivers and modules; notably, the audio driver is missing, which is a show stopper for me. If the missing drivers don't bothre you, it's much easier to follow the instructions from the [Arch Linux site][arch-pcduino-install]. Otherwise, read on.

# Motivation

The reason for installing [Arch Linux] has to do with the intended functionality of this board: a headless media server/player, as small and fast as possible. The default Ubuntu image works very nice, but it's way too "heavy" for this task (as seen in my [previous post][nav-firstimpressions], the factory shipped Ubuntu image has a 1.4G root file system) . It has a graphical user interface that we're not going to use and other packages that we don't need. There are a number of things we can do to fix this:

1. Remove the packages we don't need from the Ubuntu image. This is possible, but definitely harder than it sounds. I've actually tried it and ended up with a fairly small image, but I wouldn't recommend it. You need a good understanding of the packaging system and the dependencies between packages can often be counterintuitive. This is not surprising, since Ubuntu's main goal is to be a complete and easy to use Linux distribution, rather than a minimalistic one.
2. Start from scratch or almost from scratch. One could use [Yocto] or a similar project to generate a really small and highly customizable distribution. However, in my experience, using Yocto is quite an involved process, with a steep learning curve. I'd rather focus on creating the functionality of the server than spending a lot of time generating the base image for the board, so this doesn't sound like the right choice.
3. Use a Linux distribution which is designed with minimalism in mind as the base image.

By now, you probably gussed that I went with 3 above and chose [Arch Linux] as the base for my headless server. It requires more experience with a Linux environment than Ubuntu does, but it's small and has very good documentation. So let's go ahead and install Arch on the pcDuino3Nano board.

<div class="info">
  <h4>Note</h4>
  <p>My experience with Arch is actually fairly limited, I'm mostly an Ubuntu and Mint user. If parts of what I'm writing here seem wrong or just downright stupid from the perspective of an experienced Arch user, please let me know in the comments.</p>
</div>

<div class="error">
  <h4>Warning</h4>
  <p>If you decide to follow this tutorial, keep in mind that you might mess up the data on your local HDD in a way that's very difficult or impossible to recover (at least without a backup) if you don't pay close attention to the commands you're typing, especially the ones that write directly to a physical storage device. Keep focused and double check your command line. Obviously, I won't take any responsability if something goes wrong in the process.</p>
</div>

# What you'll need

You'll still need to use the default pcDuino3Nano Ubuntu image, so the Arch Linux image will be installed on a microSD card (luckily, the board will try to boot from SD before booting from the internal NAND). I'm using a 4GB Class 4 microSD, which is fine, but I'd advise to use a Class 6 or faster card. 4GB on the other hand is more than enough, 2GB would also be fine. I'm also using a "generic" USB card reader on my laptop.

I'm using Linux Mint on my laptop to prepare the SD card because that's my environment of choice for development of embedded Linux applications (other desktop Linux distributions will work equally well). Windows is required for one of the installation steps, which is fortunately optional.

# Step 1 (optional): update kernel

This step is optional, but highly recommended: update the kernel to the latest official one. This is also the only step that requires Windows. For [pcDuino3Nano], the kernel and Ubuntu images can be found [here][board-images]. There is already a [video][board-howtoflash] that covers the update process, so I won't insist on that.

# Step 2: prepare the SD card

This section is based on [this article](http://www.pcduino.com/chapter-4-bootable-microsd-with-ubuntu-os/), with some parts modified to actually work on the nano board (the article only covers the original pcDuino3 board and the process is slightly different for the nano).

<div class="notify notify-yellow"><span class="symbol icon-excl"></span>The data on the SD card will be destroyed after this step is finished, so make sure to backup the card first.</div>

Note that all the commands in this step will run on the pcDuino3Nano board, so you'll need to boot the board using the default Ubuntu image and connect to the board using **ssh** (this is described in my [previous post][nav-firstimpressions]). After the SSH connection is successfully established, plug the microSD card into the nano board. Start *fdisk* to partition the microSD:

{% highlight sh %}
$ sudo fdisk /dev/mmcblk0
{% endhighlight %}

Type **p** to list the partitions on the SD card. If there are some partitions present, delete all of them by using the **d** command repeteadly. After the partition table is empty, create the partitions we need:

{% highlight sh %}
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-7626751, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-7626751, default 7626751): +64M

Command (m for help): t
Selected partition 1
Hex code (type L to list codes): c
Changed system type of partition 1 to c (W95 FAT32 (LBA))

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (1-4, default 2): 2
First sector (133120-7626751, default 133120):
Using default value 133120
Last sector, +sectors or +size{K,M,G} (133120-7626751, default 7626751):
Using default value 7626751

Command (m for help): w
{% endhighlight %}

Now  create a FAT file system on the first partition and an EXT3 file system on the second one:

{% highlight sh %}
$ sudo mkdosfs /dev/mmcblk0p1
$ sudo mkfs.ext3 /dev/mmcblk0p2
{% endhighlight %}

Copy the files need for booting (including the kernel) to the first partition and copy the bootloader directly:

{% highlight sh %}
$ mkdir /tmp/{boot,nanda}
$ sudo mount /dev/mmcblk0p1 /tmp/boot
$ sudo mount /dev/nanda /tmp/nanda
$ sudo cp /tmp/nanda/uImage /tmp/boot
$ sudo cp /tmp/nanda/script.bin /tmp/boot
$ sudo cp /boot-mmc/uEnv.txt /tmp/boot
$ sudo umount /tmp/{boot,nanda}
$ sudo dd if=/boot-mmc/u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
$ sync
{% endhighlight %}

Since we're not building a new kernel, we'll want to save the kernel modules and their configuration and restore it later in Arch Linux. After that we shut down the board, since we're done with it for now:

{% highlight sh %}
$ mkdir /tmp/root
$ sudo mount /dev/mmcblk0p2 /tmp/root
$ sudo mkdir /tmp/root/modsave
$ sudo cp -a /lib/modules/3.4.79+/ /tmp/root/modsave/
$ sudo cp -a /etc/modprobe.d/ /tmp/root/modsave/
$ sudo cp -a /etc/modules /tmp/root/modsave/
$ sudo umount /tmp/root
$ sudo poweroff
{% endhighlight %}

# Step 3: download and unpack the root file system

The commands in this step will run on the PC, not on the board, as opposed to the previous step. After the pcDuino3Nano board shuts down, remove the microSD card, plug it into your USB card reader and plug the USB reader in your PC running Linux. It should be automatically recognized and possibly automatically mounted. In my case, the USB reader is assigned to the **/dev/sdc** device node.

<div class="warning">
  <h4>Be careful</h4>
  <p>On your PC, the USB reader might have another node assigned under <strong>/dev</strong>. Make sure to check what's the correct node (for example by running <strong>dmesg</strong> right after you plug the reader in your PC and taking note of the kernel messages) and substitute the correct node in the commands below. Failure to do so might result in serious data corruption on your PC.</p>
</div>

We are only interested in the EXT3 partition now (which is **/dev/sdc2**) and we'll mount it under **/tmp/root** for simplicity:

{% highlight sh %}
$ sudo umount /dev/sdc{1,2}
$ mkdir /tmp/root
$ sudo mount /dev/sdc /tmp/root
{% endhighlight %}

Now we need to download  the pcDuino Arch Linux root file system image from [this link][rootfs]. After the download finishes, simply unpack the image to **/tmp/root**:

{% highlight sh %}
$ sudo bsdtar -xpf ArchLinuxARM-sun7i-latest.tar.gz -C /tmp/root
$ sync
{% endhighlight %}

At this point, you have a bootable SD card that you can plug into your pcDuino3Nano and it will boot Arch Linux. However, the image is still missing a critical feature: out of the box Ethernet connectivity via DHCP. Which means you need to connect it to a HDMI monitor and an USB keyboard to use it, which doesn't go well with our "headless server" goal. Let's see what we can do about it.

# Step 4: enable DHCP for Ethernet

<div class="info">
  <h4>Update 14.12.2015</h4>
  <p>The procedure below is recommended only if you can't or don't want to connect directly to the board (with a monitor and an USB keyboard). If you can connect directly to the board, it's probably easier to follow the procedure described in the <a href="https://wiki.archlinux.org/index.php/Netctl" style="text-decoration: none">Arch Linux wiki</a>. See below for more details.</p>
</div>

The default Arch Linux pcDuino image has all the packages needed to run a DHCP client on the Ethernet interface, but the board needs extra configuration to make this happen. I believe this is wrong; on an embedded board like the pcDuino, out of the box Ethernet connectivity is very important, and the easiest way to do that is to run a DHCP client on the Ethernet interface automatically. Fortunately, the fix is farily simple. First, copy/paste the text below into your favourite text editor:

{% highlight sh %}
.include /usr/lib/systemd/system/netctl@.service

[Unit]
Description=A basic dhcp ethernet connection
BindsTo=sys-subsystem-net-devices-eth0.device
After=sys-subsystem-net-devices-eth0.device
{% endhighlight %}

Save this file as **/tmp/root/etc/systemd/system/netctl@ethdhcp.service** (you'll probably need superuser permissions for that). Then execute these commands:

{% highlight sh %}
$ cd /tmp/root/etc/netctl
$ sudo cp examples/ethernet-dhcp ethdhcp
$ cd ../systemd/system/multi-user.target.wants
$ sudo ln -s ../netctl@ethdhcp.service .
$ cd
$ sudo umount /tmp/root
{% endhighlight %}

(check [this link](https://wiki.archlinux.org/index.php/netctl) if you want to know more about how to use **netctl** in Arch Linux).

This time, you're really done. Plug the microSD card into the pcDuino3Nano board, power it up, figure out the IP address, ssh to it (just like we did in the [previous article][nav-firstimpressions], but this time the default username and password are **root** and **root** respectively) and enjoy your new root prompt!

## Update 14.12.2015

If you can connect to the board using a display and an USB keyboard, you can enable networking easier by executing these commands as root:

{% highlight sh %}
# cd /etc/netctl
# cp examples/ethernet-dhcp .
# netctl start ethernet-dhcp
# netctl enable ethernet-dhcp
{% endhighlight %}

# Step 5: restore kernel modules

We're going to restore the kernel modules and their configuration as it was in Ubuntu. The kernel modules are configured a little differently in Arch Linux, but it's still fairly easy to restore our configuration. One more thing: the root filesystem image was built for a different board and it contains a  bootloader package which is not relevant to the nano board, which needs to be removed (the last command in the list below does just that):

{% highlight sh %}
# rm -rf /usr/lib/modules/*
# cp -a /modsave/3.4.79+ /usr/lib/modules
# cp -a /modsave/modprobe.d/* /etc/modprobe.d
# rm -rf /modsave
# pacman -Rs uboot-cubieboard2
{% endhighlight %}

For now, I chose not to force Arch Linux to load the same modules that Ubuntu does on startup (**/etc/modules** in Ubuntu). I'm going to use manual module configuration only if **udev** fails to load the required modules automatically. For more information about how to configure the kernel modules in Arch Linux, check [this link](https://wiki.archlinux.org/index.php/Kernel_modules).

# Step 6 (optional): customize the Arch Linux image

<div class="info">
    <h4>Update 14.12.2015</h4>
    <p>These numbers reflect older versions of the Arch Linux root file system. The numbers you'll get will likely be higher.</p>
</div>

First, let's inspect the image a bit:

{% highlight sh %}
[root@alarm ~]# uname -a
Linux alarm 3.4.79+ #5 SMP PREEMPT Wed Oct 15 14:06:46 CST 2014 armv7l GNU/Linux
{% endhighlight %}

This confirms that we're using the same kernel as the Ubuntu image.

{% highlight sh %}
[root@alarm ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p2  3.6G  584M  2.9G  17% /
[truncated]
{% endhighlight %}

The root file system size is much smaller than the Ubuntu one (584M vs 1.4G!), so it looks like our efforts to install Arch Linux have payed off. But 584M still sounds quite large, maybe we can make it smaller? In my case, it turns out that I can remove some packages that I don't need, so the answer is "yes" (you can skip this step if you're not sure if you need these packages or not):

{% highlight sh %}
# pacman -Rs man-db man-pages
# pacman -Rs texinfo lvm2 cryptsetup device-mapper
# pacman -Rs xfsprogs mdadm file
# pacman -Rs linux-armv7m
{% endhighlight %}

(yes, it's really OK to uninstall the kernel, since we've manually installed a separate kernel and its modules in a previous step).

I'm also going to get rid of the locales that I don't need:

1. Install **localepurge** by running **pacman -S localepurge**
2. List the locales you want to **keep** in **/etc/locale.nopurge** (**localepurge** will delete all the locales **except** the ones you list in **/etc/locale.nopurge**). When you're done, comment the *NEEDSCONFIGFIRST* line (near the beginning of the file) by adding a '#' in front of it.
3. Run **localepurge**

In my case, localepurge reported that it freed 50888 KB of space, which is quite a significant save.

Finally, clean the package cache:

{% highlight sh %}
# paccache -r
# paccache -ruk0
{% endhighlight %}

Was it worth the effort ?

{% highlight sh %}
[root@alarm ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p2  3.6G  483M  3.0G  15 % /
[truncated]
{% endhighlight %}

We got rid of about 100M, which is not bad at all. Going lower than this is possible, but probably quite difficult. An obvious candidate for removal is *perl*, but in turns out that in Arch Linux [openssl has a weird dependency on perl](https://bugs.archlinux.org/task/14903). Many people (me included) consider this a bug, but the Arch Linux maintainers seem to be really reluctant to do anything about it.

# Step 7 (optional): backup the root filesystem

Since you've put quite a bit of effort into building a good root filesystem, it makes sense to back it up somewhere, in case you need to start from scratch again later or you want to share it with someone. There are different ways to do this, I'm going to use **partclone**. So power down your pcDuino3Nano board, put the SD into the SD card reader, plug the reader into the PC and back up the image:

{% highlight sh %}
$ sudo apt-get install partclone
$ sudo umount /dev/sdc{1,2}
$ sudo partclone.ext3 -c -s /dev/sdc2 -o archlinux_base.img
$ sudo bzip2 -9 archlinux_base.img
{% endhighlight %}

You can restore this image later using **partclone.ext3 -r** and swapping the source and destination arguments.

# Step 8 (optional): update the image

Updating [Arch Linux] is easy, but if you removed the kernel package in step 6 above, you need to tell `pacman` to ignore updates to the kernel. To do that, edit **/etc/pacman.conf** and add this line to it:

{% highlight sh %}
IgnorePkg   = linux-armv7
{% endhighlight %}

Once that's done, updating the image is really reasy:

{% highlight sh %}
# pacman -Syu
{% endhighlight %}

# Conclusion

The process of installing Arch Linux on the pcDuino3Nano board is a bit involved, but I believe the results are well worth the effort. Besides getting a smaller image (which also feels slightly more responsive during normal use), the thing I appreciate the most is Arch Linux's clean design. In particular, I find **pacman** to be a significantly better package management system than **dpkg/apt-get**. It's also a bit faster, which is important in this kind of resource constrained environment. On the other hand, I'm not a big fan of **systemd**, but I'll get used to it eventually.

The first step above (updating the kernel) can be done from Linux with a bit more effort. In a future post, I might cover how to build a custom kernel image for this board and at that point we'll be in a good position to update the kernel image using only Linux.

[pcDuino3Nano]: http://www.linksprite.com/linksprite-pcduino3-nano/
[nav-firstimpressions]: /pcduino3nano-first-impressions
[Arch Linux]: http://www.archlinux.org
[Arch Linux ARM]: http://www.archlinuxarm.org
[Yocto]: https://www.yoctoproject.org/
[arch-pcduino]: http://archlinuxarm.org/platforms/armv7/allwinner/pcduino
[arch-pcduino-install]: http://archlinuxarm.org/platforms/armv7/allwinner/pcduino3#qt-platform_tabs-ui-tabs2
[board-images]: http://www.linksprite.com/image-for-pcduino3-nano-pcduino3b/
[board-howtoflash]: http://learn.linksprite.com/pcduino/quick-start/pcduino3/video-run-built-in-arduino-ide-on-pcduino3/
[rootfs]: http://archlinuxarm.org/os/ArchLinuxARM-armv7-latest.tar.gz

