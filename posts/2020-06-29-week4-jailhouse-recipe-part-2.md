---
layout: post
title:  "Week 4: Jailhouse recipe, part 2"
date:   2020-06-29 18:00:00 +0200
categories: 
---
Last week I continued to work on the Jailhouse recipe. I published my Yocto layer [meta-jailhouse](https://github.com/Limoto/meta-jailhouse) with the current code. I have a working Jailhouse recipe that should give all the necessary stuff. I don't have a recipe that would set up the ARM Trusted Firmware on the Raspberry yet, so I had to copy the *bl31.bin* manually to the boot partition of the SD card and set `armstub=bl31.bin` in *config.txt*. I've got the binary from the build of [jailhouse-images](https://github.com/siemens/jailhouse-images). However, when enabling Jailhouse, I get this error message:

    raspberrypi4-64:~# jailhouse enable /usr/share/jailhouse/cells/rpi4.cell 

    Initializing Jailhouse hypervisor v0.12 (59-g4ce7658d-dirty) on CPU 2
    Code location: 0x0000ffffc0200800
    Page pool usage after early setup: mem 39/994, remap 0/131072
    Initializing processors:
    CPU 2... OK
    CPU 1... OK
    CPU 3... OK
    CPU 0... OK
    Initializing unit: irqchip
    Initializing unit: ARM SMMU v3
    Initializing unit: PVU IOMMU
    Initializing unit: PCI
    Adding virtual PCI device 00:00.0 to cell "Raspberry-Pi4"
    Adding virtual PCI device 00:01.0 to cell "Raspberry-Pi4"
    Page pool usage after late setup: mem 61/994, remap 5/131072
    FATAL: instruction abort at 0xfbfff7d0

    FATAL: forbidden access (exception class 0x20)
    Cell state before exception:
    pc: ffffffc008c147d0   lr: ffffffc008c147d0 spsr: 20000085     EL1
    sp: ffffffc01190ba70  esr: 20 1 0000086
    x0: 0000000000000000   x1: 0000000000000000   x2: 0000000000000000
    x3: 0000000000000000   x4: 0000000000000000   x5: 0000000000000000
    x6: 0000000000000000   x7: 0000000000000000   x8: 0000000000000000
    x9: 0000000000000000  x10: 0000000000000000  x11: 0000000000000000
    x12: 0000000000000000  x13: 0000000000000000  x14: 0000000000000000
    x15: 0000000000000000  x16: 0000000000000000  x17: 0000000000000000
    x18: 0000000000000000  x19: ffffffc008c1bc28  x20: ffffffc014400000
    x21: 0000000000000000  x22: 0000000000000000  x23: 0000000000000000
    x24: 0000000000000420  x25: ffffffc008c1b010  x26: 0000000000000004
    x27: ffffffc008c1b880  x28: ffffffc008c1bc28  x29: ffffffc01190ba70

    Parking CPU 1 (Cell: "Raspberry-Pi4")

I tried to solve that by using the kernel patches from [Jan Kiszska's kernel repository](http://git.kiszka.org/?p=linux.git;a=shortlog;h=refs/heads/jailhouse-enabling/5.4), but it didn't help. So now I have to dive into this further.