---
layout: post
title:  "Introduction"
date:   2020-05-18 10:00:00 +0200
categories: 
---
In this blog, I’m going to write about progress on my GSoC project. The project is called “Porting of Jailhouse partitioning hypervisor into Automotive Grade Linux” and my mentor is Jan-Simon Möller (dl9pf).

So, the task is about porting the Jailhouse hypervisor into Automotive Grade Linux. That means to make it build together, and make it working under QEMU and on a Raspberry Pi 4. If the time allows, a communication mechanism between the Linux system running in one Jalihouse cell and another, “real-time” cell running bare-metal code or an RTOS will be implemented.

Jailhouse is a partitioning hypervisor based on Linux. It allows to run bare-metal or adapted operating systems next to Linux on the same machine. It uses the hardware virtualization features of the platform to create independent domains called ‘cells’, that are given some part of the system to operate on (CPUs, memory, devices). The cells can’t interfere with each other in an unacceptable way.

Compared to traditional virtualization like KVM or VirtualBox, it doesn’t allow overcommitment of the hardware resources, e.g. only one cell can run code on one CPU core. It doesn’t run inside the Linux kernel as the traditional virtualization does. Instead, it is started on a running Linux kernel and moves that running kernel into a cell, so Jailhouse runs beneath the Linux kernel. Then, Jailhouse can be controlled from the Linux system.

The integration of Jailhouse into the AGL platform will allow us to run the critical code beside the current AGL stack. That means that the AGL stack running in one cell can’t interfere with the critical code running in the second cell. A diagram of how it will fit into the AGL is in the picture below.

![AGL and Jailhouse integration diagram]({{site.baseurl}}/assets/img/agl-jailhouse-diagram.svg)
