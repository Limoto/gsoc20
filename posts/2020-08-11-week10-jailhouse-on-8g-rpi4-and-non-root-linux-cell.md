---
layout: post
title:  "Week 10: Jailhouse on 8G RPi4 & non-root Linux cell"
date:   2020-08-11 13:00:00 +0200
categories: 
---

In the last report, I announced support for the 4G variant of RPi4, but later I found out that I have a problem with the virtual PCI MMIO space. That is used by Jailhouse to present virtual devices to the host/guest system and allow communication between them.

At that moment, it seemed like a trivial issue, but I faced issues when trying to get it to work. In the original Jailhouse configuration, it was placed at some address about 3,5G in the physical address space. However, at the RPis with 4G or 8G memory, this area is occupied with RAM and thus I was getting errors when enabling Jailhouse and the virtual PCI devices were not showing up.

At first, I tried to move it to address above the 8G of RAM, which is not used by anything, but for some reason, it has to be in the first 4G of physical address space, so that was not a way. 

Then I tried to move it somewhere into the memory area reserved for Jailhouse, but I was getting some crashes, so I finally decided to leave it where it was originally and add another reserved memory region to the device tree to reserve 2 MiB at the original offset. That helped and everything finally started to work.

After moving the reserved memory for Jailhouse use from 0x30000000 (768M) to 0x20000000 (512M) in the last weeks, it doesn't conflict with GPU memory anymore (for gpu_mem setting of up to 256M, which is used on AGL). That made me try to boot Linux in a non-root cell, because I was not able to get that working before. I haven't succeeded with the AGL kernel, but when I tried to use the kernel from jailhouse-images for the non-root cell, it finally booted! The non-root cell took over the serial console and it's also possible to communicate between the cells over the virtual network interface.

So, to run Linux in a non-root cell, the following commands can be used. Note that rootfs.cpio and demo-image-jailhouse-demo-rpi4-vmlinuz are taken from jailhouse-images.

    jailhouse enable /usr/share/jailhouse/cells/rpi4.cell
    ip a a 192.168.19.1/24 dev eth1
    jailhouse cell linux /usr/share/jailhouse/cells/rpi4-linux-demo.cell demo-image-jailhouse-demo-rpi4-vmlinuz -d /usr/share/jailhouse/cells/dts/inmate-rpi4.dtb -i rootfs.cpio -c "console=ttyS0,115200 ip=192.168.19.2"

So for now, I plan to open a pull request today to get this stuff into meta-agl-devel. Meanwhile, I also want to get the stuff into jailhouse and jailhouse-images. Jailhouse-images actually doesn't even boot on the 8G RPi4, it seems there are some old firmwares used, so I want to fix that too. During the next week, I want to start the effort to get Jailhouse merged into meta-virtualization. And if time allows, I will start to implement some nice demo for the non-root cell.




