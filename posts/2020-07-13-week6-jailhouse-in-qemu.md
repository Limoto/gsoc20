---
layout: post
title:  "Week 6: Jailhouse in QEMU, CAN on RPi"
date:   2020-07-13 20:00:00 +0200
categories: 
---

This week I was working on two tasks. The first was to get CAN bus working on the Raspberries I have, so at first, I've got it running under Raspbian (well.. they call it Raspberry Pi OS now), and then I was able to do the same on AGL. It was actually pretty straight-forward, as I only had to bbappend the rpi-config do add the CAN dtoverlay.

Then I continued on getting Jailhouse to the QEMU image. At first, I've got a build error about missing .bbs, because I didn't know that I need to have a matching .bb for every .bbappend (I've thought bitbake will simply ignore non-matching bbappends). I had this problem because of the board-specific stuff, so I started to split the layers into two: `meta-agl-jailhouse` and `meta-agl-jailhouse-raspberrypi`. The only problem is to actually enable the right layers when you enable the agl-jailhouse feature. I tried to write a python conditional to check the MACHINE variable, but it seems that Python code is not evaluated in that part of the configuration. Today, Jan-Simon proposed that I can use dynamic-layer for that, so next week I will rework that again.

So I had an AGL image for qemux86_64 target, with Jailhouse built-in, but that's just the first step. The problem is that the cell configuration that is used for the root cell (the AGL one) and that is supplied when Jailhouse is being enabled, strongly depends on the configuration of the system. That means, when I add or remove some devices to the QEMU machine, or just change the kernel a bit, things get remapped and the configuration doesn't work anymore.

So, the way to go is to generate the cell configuration after booting into AGL using:

    jailhouse config create agl.c

and then copy over generated `agl.c` into Jailhouse source tree (outside of the Yocto tree) into `configs/x86` subdirectory and run make in the root directory to build Jailhouse. Then, copy over `configs/x86/agl.cell` into the running virtual machine and enable Jailhouse using:

    jailhouse enable agl.cell

It's also necessary to add the kernel parameters into `/boot/loader/entries/boot.conf`:

    memmap=82M$0x3a000000 intel_iommu=off

The memory map parameter may differ in another setup and is listed in the comments at the beginning of the generated agl.c file.

So, the task for the next week is to try to polish the QEMU setup and to improve the Jailhouse layers so that they can be upstreamed. 