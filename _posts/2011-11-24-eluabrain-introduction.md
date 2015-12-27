---
title: eLuaBrain - introduction
layout: post
permalink: eluabrain-introduction
comments: true
categories: [eLua]
---
*eLuaBrain* is a fully autonomous computer built around [eLua] on the software side and a [STM3210E-EVAL] (an excellent development board for a STM32 (Cortex-M3) CPU from ST Microelectronics). In short, it is a 32-bit computer with a PS/2 keyboard input and a VGA (text only for now) video output that can be used for interactive program development.

<!--more-->

Obligatory picture:

<div align="center"><img src="/images/eluabrain/app_logo.jpg" alt="eLuaBrain startup screen" style="width: 60%;" align="middle" /></div>

*eLuaBrain* is the little board with green and red LEDs on the right of the image. It connects to the black PS/2 keyboard and outputs VGA text to the monitor. The oscilloscope (gray box on the left side) is not part of *eLuaBrain*.

I built it with two main ideas in mind:

* to fulfill an old dream of mine (some insist to call it an "obsession" for some reason): designing and implementing a fully functional, reasonably featured computer based on a MCU (rather than a "classical" CPU system). The recent explosion of 32-bit MCUs made this finally possible.
* enter it in a EBV/ST Microelectonics design contest. I did that, but I didn't get any prize. On the bright side, the contest rules forced me to write quite a bit of documentation for the project, so whoever decides to take a closer look at *eLuaBrain* will probably have a good starting point. You can download the full entry [here](https://github.com/bogdanm/eLuaBrain/raw/master/entry00138/abstract.doc) (file size is about 10M).

The complete source code, as well as the schematics and the documentation are available [here](https://github.com/bogdanm/eluabrain).

<div class="info">
    <h4>A quick note</h4>
    <p> If you open the entry files, you'll notice that the "official" name of entry is STMBrain, not eLuaBrain. This was just a publicity stunt (since one of the contest organizers was ST Microelectronics). eLuaBrain is not "tied" to a particular hardware, it can be implemented and run equally well on a large variety of MCUs and development boards. eLuaBrain is also a bit more descriptive (from a semantic standpoint), so I'm going to stick to that from now on.</p>
</div>

I believe *eLuaBrain* has a number of features that could make it interesting for some people (then again, I'm obviously biased). To quote from the entry's abstract.doc file: being a generic development platform, *eLuaBrain* can be used in lots of applications, but it was design around two main usage scenarios:

* **educational computer**: *eLuaBrain* can be an excellent tool for students trying to learn about computer science and/or hardware design. Modern computer hardware is simply too hard to understand even for people with some computer knowledge, while most modern programming languages exhibit a growing trend of hiding as much about the hardware as possible. By contrast, *eLuaBrain* offers the full experience. Its hardware is quite easy to understand and it can be easily interfaced with other hardware (which might be a problem on a PC); at the same time, [eLua] offers APIs that can access the hardware at very low level. Learning about the software and the hardware in parallel gives students a much better idea about the software-hardware interactions, which in turn tends to develop better professionals. At the same time, learning about programming on a platform with relatively low resources forces the future programmers to consider resource allocation and optimization techniques, so again, better programmers. All these make *eLuaBrain* a good tool for education.
* **automation controller**: *eLuaBrain* can act as a controller for a large number of automation tasks, ranging from home automation to industrial automation. Its main strengths are the power of the STM32 CPU, the ability of being programmed on-the-fly, the built-in help system, the radio interface and the large number of peripherals available via its extension connector. It can replace a PC as an automation controller in the many cases where the full power of a PC isn't actually needed, but a better alternative is either not available or very expensive. Compared to a PC, it brings simplicity, reliability (a simpler system is almost always more reliable than a complex system), lower cost and lower power consumption.

Phew, lots of text here. I bet everybody's waiting for some actual action by now, so here it comes. First you have a demo of interactive program development on *eLuaBrain*, featuring the integrated text editor, online help system and other goodies:

<div align="center"><iframe src="https://player.vimeo.com/video/28887627" width="500" height="375" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe></div>

Following is a video of most of the demo programs I wrote for *eLuaBrain* as part of the contest entry (there are a few not included here, but still you should get a pretty good image about the capabilities of *eLuaBrain* after seeing this):

<div align="center"><iframe src="https://player.vimeo.com/video/28886601" width="500" height="375" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe></div>

If this made you curious, you can find below a complete list of the *eLuaBrain* platform capabilities:

* built around the STM32F103ZET6 MCU from ST, an extremely powerful Cortex-M3 MCU with an impressive list of on-chip communication interfaces and peripherals.
* 1MB of external RAM memory allows large, complex programs to run on *eLuaBrain*.
* PS/2 connector for any PS/2 compatible AT keyboard.
* high speed VGA video interface: 80x30 characters (original DOS CP437 charset), 16 colors, built around a Propeller microcontroller.
* TCP/IP connectivity using the ENC28J60 Ethernet to SPI bridge.
* high speed, low power wireless interface (based on the nRF24L01 chip) for easy wireless communication with remote equipment.
* microSD storage for storing *eLuaBrain* programs and data.
* an extension connector for external daughter boards that allows connections to GPIO pins, ADC, SPI, I2C, UART, CAN, timer compare outputs and others.
* UART connector with standard RS232 logic levels.
* simple sound generation on a small speaker using PWM signals.

And yet another list of *eLuaBrain*'s awesome features (taken directly from the contest entry's abstract.doc file):

* low cost: the *eLuaBrain* prototype comes as a "shield" over a STM3210E-EVAL board, but it can be produced on a dedicated PCB at low costs in large quantities, making it a compelling option for emerging economies and educational facilities in general.
* low power consumption: the STM32 along with its peripherals and the *eLuaBrain* additional hardware consumes much less power than a desktop PC or even a laptop.
* completely open source design: both the hardware and the software are open sourced, released under the very permissive MIT license which allows unrestricted use in both commercial and non-commercial projects.
* optimized for developing: based on the [eLua] project that optimizes the popular [Lua] for resource constrained systems (like MCUs), *eLuaBrain* is capable of running Lua source files directly on the STM32 CPU. Also, [eLua] is optimized for low memory devices, so it will make sure that the memory on the STM3210E-EVAL board is put to good use.
* built-in editor allows for very easy editing of source files.
* built-in interactive help browser makes programming a breeze. API help is available from the shell, from the editor or from the Lua interpreter. Whenever you forget the name, arguments or return result of a function that you need in your program, just ask the integrated help system.
* instant boot: forget about having to wait for an OS to load before being able to use your desktop. *eLuaBrain* boots in less than one second.
* access to a large number of peripherals directly from Lua: GPIOs, I2C, UART, SPI, timers, CAN and others.
* optimized TCP/IP stack lets network applications run on little memory.
* easy to interface with other hardware via the extension connector.
* multiple file systems for easy storage: FAT on SD/MMC card, ROM file system inside Flash,  UDP-based network file system with automatic server detection.
* runs on cheap hardware: *eLuaBrain* needs a PS/2 keyboard and a standard VGA (640x480) monitor to run. These can be found at very low prices today. With a low cost VGA to PAL/NTSC adapter it can also use an old TV instead of a VGA monitor.
* comes with an extensive set of demos that covers a lot of topics (network connectivity, peripheral communication, algorithms, a Web server, radio controlled RGB lights and of course games) and can be a good starting point for future applications.
* good community support via the [eLua] communication channels (mailing list, IRC, wiki).

Hopefully you'll find all this at least remotely interesting. If I get enough feedback, I'll continue posting and working on this. I'm very curious to find out what people think about this concept. Also, please keep in mind that *eLuaBrain* is far from being complete; it can be improved and extended in countless ways.

One more thing before the end of this (long) post: the software and hardware are fully open source, no restrictions at all (not even commercial ones). The software is already licensed under MIT/BSD, I don't know what license to choose for the hardware (I'll take a closer look on that), but it will definitely be a very liberal one. If MIT is also appliable to hardware design, MIT it is.

[eLua]: http://www.eluaproject.net
[STM3210E-EVAL]: http://www.st.com/internet/evalboard/product/204176.jsp
[Lua]: http://www.lua.org
