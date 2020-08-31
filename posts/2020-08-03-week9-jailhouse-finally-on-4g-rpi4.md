---
layout: post
title:  "Week 9: Jailhouse finally on 4G RPi4"
date:   2020-08-04 12:00:00 +0200
categories: 
---

After accidentally sending my last weekly report to the wrong mailing list (jailhouse-dev instead of agl-dev-community), I've got a reply on my post by Jan Kiszka and that helped me to figure out the problem. I was adding a memory region for the additional memory into Jailhouse cell configuration, but I didn't increase the number of memory regions in the configuration. I can't believe how I couldn't notice that. But at least I learned something about Jailhouse internals when figuring this out.

I also tested the configuration on the 2G variant of the RPi4 and it works fine, it's not a problem to have a memory region specified in the Jailhouse configuration that is not present on the board. The device tree supplied by the bootloader to the Linux kernel specifies the memory available and nothing will access memory behind it. 

This means that it is possible to make a universal Jailhouse configuration for all the memory variants of the Raspberry. I just don't have an 8G version to verify that (it has another 4G memory region added).

So this week I want to update the layer to reflect this and try to get the updated layer into meta-agl-devel. In the meanwhile, my pull request for setting the CAN board crystal frequency (required to support the Waveshare RS485 CAN board) got [merged into meta-raspberrypi](https://github.com/agherzan/meta-raspberrypi/pull/685#event-3598469886).




