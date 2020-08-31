---
layout: post
title:  "Week 2: Making a Jailhouse recipe, part 1"
date:   2020-06-12 18:00:00 +0200
categories: 
---
I started to work on the Jailhouse recipe for Yocto/OpenEmbedded, so that it can be used in AGL. I've looked into the recipes from [Texas Instruments](https://layers.openembedded.org/layerindex/recipe/104375/) and [NXP](https://layers.openembedded.org/layerindex/recipe/131664/) layers, but they are both a bit vendor specific and will need some adjustments.

To get Jailhouse running on the Raspberry Pi, I will also need to build the ARM Trusted Firmware (also called TF or ATF) for it and make the bootloader load it before the Linux kernel. They have [documentation about running it on a Raspberry](https://trustedfirmware-a.readthedocs.io/en/latest/plat/rpi4.html). The firmware is required to allow PSCI support, which is required for Jailhouse to be able to hotplug CPUs when transferring them between the cells. There is already a [recipe for ATF](https://layers.openembedded.org/layerindex/recipe/121436/) in the meta-arm layer, but I will have to somehow propagate it to the /boot partition on the RPi and set the config.txt variables.

I've also looked into how the support for Xen is done in AGL, but everything is done in AGL specific way and that is not my desire, as I would like to upstream as much as I can.

I'm setting up my own OpenEmbedded layer to put the recipes in, but my exams started this week and many more will come next week, so my time is unfortunately quite limited now. 