---
layout: post
title:  "Final report"
date:   2020-08-25 14:00:00 +0200
categories: 
---

This is the final report for the project I was working for GSoC 2020: Porting the Jailhouse hypervisor into Automotive Grade Linux.

Contributions
===============

During the project, I made contributions into multiple projects. Some of them are already in the corresponding code tree of the projects, some are still in the progress and I will continue with upstreaming efforts in the following weeks.

The main contribution is the meta-agl-jailhouse layer into meta-agl-devel of Automotive Grade Linux ([commit](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/25034)). Later, I updated it with support for the higher memory variants of Rasberry Pi 4 ([commit](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/25108)). In the end, I refactored the layer to work with stock Yocto project (without AGL-specific stuff) and to be compatible with the latest revisions of its dependencies. The result is currently in the [meta-jailhouse](https://github.com/Limoto/meta-jailhouse) repository on my GitHub.

The modified Jailhouse configuration that enables use of more than 1 GiB of memory was already accepted into the next branch of jailhouse ([commit](https://groups.google.com/g/jailhouse-dev/c/L4rjIgemjZY)).

I also contributed support for the Waveshare RS485 CAN HAT boards I was planning to use into meta-raspberrypi ([pull request](https://github.com/agherzan/meta-raspberrypi/pull/685)).

All the work I have done is available in my [gsoc20](https://github.com/Limoto/gsoc20) GitHub repository.

Future plans
===============

To get the modified Jailhouse configuration into the master branch of Jailhouse, I will probably also have to make a patch for jailhouse-images to stay compatible with the changed configuration.

Regarding the meta-jailhouse, I want to discuss it with other layers using Jailhouse (e.g. NXP or TI) and make a common base recipe. Eventually, it might be upstreamed e.g. to meta-virtualization. The RPi4 stuff regarding the ARM Trusted Firmware will go into meta-raspberrypi.

Given up plans
==============

In the original proposal, I also planned to make a Jailhouse inmate , that will enable the cell to use a CAN HAT on the Raspberry Pi and forward the messages to the AGL system over shared memory. Due to many issues on the way, I haven't got to that point.

Acknowledgments
==============

At first, I would like to thank Google for creating this opportunity and supporting me in this work with the stipend. And then, also to the AGL guys for accepting me. It was great to get to know you! And especially to my mentor, Jan-Simon Möller, for helping me with an endless amount of build issues with Yocto/AGL. 

And not to forget, also thanks for supporting me with the Raspberry Pis and access to that powerful machine to build all the stuff.

