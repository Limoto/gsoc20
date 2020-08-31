---
layout: post
title:  "Week 8: Jailhouse & RPi4 with more than 1G of memory"
date:   2020-07-27 23:00:00 +0200
categories: 
---

This week I was working on enabling Jailhouse to use more than 1 GiB of memory on the Raspberry Pi 4. However, with little success so far.

Jailhouse comes with a configuration that is made for the 1 GiB variant of the RPi4 (that is actually not even sold anymore). It also works on the variants with more memory, but then it stays limited to 1 GiB. To illustrate how the memory layout of the Raspberry Pi looks like, I took the following map from the [BCM2711 ARM Peripherals documentation](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2711/rpi_DATA_2711_1p0.pdf). I took the address map in the "Low Peripheral" mode, as it seems to reflect the configuration supplied with Jailhouse. Note the memory regions are not in scale. 

![Raspberry Pi 4 memory layout]({{site.baseurl}}/assets/img/rpi4-memlayout.png)

In the [default configuration](https://github.com/siemens/jailhouse/blob/master/configs/arm64/rpi4.c), Jailhouse allocates the memory range 0x0 - 0x3fa10000 (~1018M) for the cells. The rest of the first 0x40000000 (1024M) long memory space is used for the hypervisor itself and various shared memory regions. To avoid the Linux kernel using all of the memory, `mem=768M` kernel parameter [is used](https://github.com/siemens/jailhouse-images/blob/master/recipes-bsp/rpi-firmware/files/cmdline.txt). That makes the memory above 768M available for the non-root cells. Regarding the GPU memory, the `gpu_mem` parameter in Raspberry's config.txt [is not set](https://github.com/siemens/jailhouse-images/blob/master/recipes-bsp/rpi-firmware/files/config.txt), so it defaults to 64M. Then, the area of 0x3b400000 (948M) - 0x40000000 (1024M) is reserved by the Raspberry firmware. I don't know how they make sure that Jailhouse's shared memory regions don't interfere with the GPU, but it seems that they know what they're doing.

To extend the support to the 4GiB version, I added a section describing the additional memory region to the Jailhouse configuration:

	/* RAM (1024M-4032M) */ {
		.phys_start = 0x40000000,
		.virt_start = 0x40000000,
		.size = 0xbc000000,
		.flags = JAILHOUSE_MEM_READ | JAILHOUSE_MEM_WRITE |
			JAILHOUSE_MEM_EXECUTE,
	},

To simulate the behavior of the previous `mem=768M` kernel parameter, I decided to reserve the memory area of 0x30000000 (768M) - 0x40000000 (1024M). At first, I tried that by using the `memmap` kernel parameter, but I found out it's x86 only and I have to define the reservation in the device tree. So I had to make my own device tree overlay (DTBO). After some time of figuring out the actual format, this one is doing what I want:

    // Reserved memory for Jailhouse use
    /dts-v1/;
    /plugin/;

    / {
    	compatible = "brcm,bcm2835";

    	fragment@0 {
		    target-path = "/";
		    __overlay__ {
    			reserved-memory {			
				    #address-cells = <2>;
				    #size-cells = <1>;
				    ranges;

				    jailhouse_reserved: jailhouse@30000000 {
    					reg = <0x0 0x30000000 0x10000000>;
					    no-map;
				    };
			    };
		    };
	    };
    };

However, even after all of this, I can't get Jailhouse to work with this additional memory. When I run `jailhouse enable` with this modified configuration, it crashes immediately:

    root@demo:~# jailhouse enable rpi4-4g.cell  

    Initializing Jailhouse hypervisor v0.12 (59-g4ce7658d) on CPU 3
    Code location: 0x0000ffffc0200800
    Page pool usage after early setup: mem 39/994, remap 0/131072
    Initializing processors:
    CPU 3... OK
    CPU 0... OK
    CPU 2... OK
    CPU 1... OK
    Initializing unit: irqchip
    Initializing unit: ARM SMMU v3
    Initializing unit: PVU IOMMU
    Initializing unit: PCI
    Adding virtual PCI device 00:00.0 to cell "Raspberry-Pi4 4G"
    Adding virtual PCI device 00:01.0 to cell "Raspberry-Pi4 4G"
    Page pool usage after late setup: mem 61/994, remap 5/131072
    FATAL: instruction abort at 0xfbfff7c0

    FATAL: forbidden access (exception class 0x20)
    Cell state before exception:
    pc: ffffffc0089fd7c0   lr: ffffffc0089fd7c0 spsr: 20000085     EL1
    sp: ffffffc01000bef0  esr: 20 1 0000086
    x0: 0000000000000000   x1: 0000000000000000   x2: 0000000000000000
    x3: 0000000000000000   x4: 0000000000000000   x5: 0000000000000000
    x6: 0000000000000000   x7: 0000000000000000   x8: 0000000000000000
    x9: 0000000000000000  x10: 0000000000000000  x11: 0000000000000000
    x12: 0000000000000000  x13: 0000000000000000  x14: 0000000000000000
    x15: 0000000000000000  x16: 0000000000000000  x17: 0000000000000000
    x18: 0000000000000000  x19: ffffffc008a04c28  x20: ffffffc014800000
    x21: 0000000000000000  x22: 0000000000000001  x23: 0000000000000000
    x24: 0000000000000001  x25: 0000000000000001  x26: ffffffc010fe3dc0
    x27: 0000000000000000  x28: ffffff80f6da5940  x29: ffffffc01000bef0

    Parking CPU 1 (Cell: "Raspberry-Pi4 4G")

The address 0xfbfff7c0 of the instruction abort is 0x840 (2112) bytes before the end of the defined memory region and the start of the peripheral space. When I limit the kernel memory by using the `mem=` kernel parameter, e.g. `mem=2G`, the address moves, but it stays at the same offset from the end of the memory space. I can't figure out why it's crashing. I tried [to ask in the jailhouse-dev mailing list](https://groups.google.com/g/jailhouse-dev/c/iY84ebxz2hI), but I haven't got any answers. 

In the meanwhile, my meta-agl-jailhouse layer [was accepted into meta-agl-devel](https://gerrit.automotivelinux.org/gerrit/c/AGL/meta-agl-devel/+/25034). In the next week, while trying to figure out this issue, I want to make a recipe for compiling additional cell configurations independently from the jailhouse recipe.




