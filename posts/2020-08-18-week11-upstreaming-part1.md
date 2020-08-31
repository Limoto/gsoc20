---
layout: post
title:  "Week 11: Upstreaming, part 1"
date:   2020-08-18 13:00:00 +0200
categories: 
---

This week I focused on upstreaming of the configuration files for Jailhouse into the upstream repository. First I tried to already integrate all the stuff into jailhouse-images repository, but then I got a bit stuck on the fact that they don't use the device tree overlays built with the kernel, but just unpack the ones supplied with Raspberry Pi's firmware. So I would need to change that to be able to build the device tree overlay that I use for reserving the memory.

But I rather decided to send out just the patches for Jailhouse configs and see what the discussion brings up. It took me a while to get git send-email to work and to be able to use my Gmail account, but I managed to contribute the patch and currently I'm resolving the feedback.

I've also looked into meta-virt and currently I'm figuring out how to integrate the Jailhouse stuff in there. 



