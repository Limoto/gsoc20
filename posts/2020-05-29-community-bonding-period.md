---
layout: post
title:  "Community Bonding Period: Figuring out the goal"
date:   2020-05-29 12:00:00 +0200
categories: 
---
During the community bonding period, I joined the AGL mailing list and introduced myself and my project. I also started to attend the regular Tuesday's AGL Weekly Developer Calls and biweekly AGL Virtualization Expert Group calls.

During the calls with my mentor, we found out a new goal for the project. The goal is to make a real-time Jailhouse cell that will be given access to the CAN hardware peripheral. It will be able to reply to CAN messages that need a quick reply and pass the other to the Linux cell with AGL.

To implement that, I have got two Raspberry Pi 4s (4 GB and 2 GB version) and two pieces of the [Waveshare CAN HAT](https://www.waveshare.com/rs485-can-hat.htm). One of them will be used to run the AGL system with Jailhouse and one will be used as the counterpart on the CAN bus.

During the Virtualization EG call, I was asked if I was thinking about using virtio. It is a specification that allows virtual machines access to simplified virtual devices, so there is no need to emulate more complicated real hardware peripherals. I haven't thought about that before, but it would make sense to use it. There would be a virtual CAN link between the real-time cell and the AGL cell. I was linked to [a paper](https://www.researchgate.net/publication/337760284_Virtio_Front-end_Network_Driver_for_RTEMS_Operating_System) about using virtio on RTEMS real-time operating system. It implements networking over virtio, so it might serve as an example for implementing CAN bus. It seems that there is no specification for CAN over virtio, but there is some code of a Linux driver for [virtio-can](https://github.com/ork/virtio-can) on GitHub.

Another thing to consider is the hardware support in RTEMS. There is a [BSP for Raspberry Pi](https://git.rtems.org/rtems/tree/bsps/arm/raspberrypi), but according to the comments in source code, it's made for the original Raspberry Pi and Raspberry Pi 2. So I will have to try in on a RPi 4 and eventually port it. There are already drivers for SPI and UART in the RPi RTEMS BSP, but I haven't found any RTEMS code for the MCP2515 controller used on the CAN HAT. So I will have to implement that.

There was a [GSoC project](https://devel.rtems.org/wiki/GSoC/2016/RTEMS%20improvement%20for%20Jailhouse%20hypervisor) about running RTEMS in Jailhouse, but I'm not sure how much usable is the [outcome](http://wonjunhwang.blogspot.com/2016/08/final-report-of-gsoc-2016-rtems.html). So that is another thing to dive into.

So, the goal for next week is to compile and run AGL and Jailhouse both in QEMU and on the Raspberry. Then I will continue integrating them together.

