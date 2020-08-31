---
layout: post
title:  "Week 5: Jailhouse running on RPi4!"
date:   2020-07-07 0:00:00 +0200
categories: 
---

Last week I solved the issues with running Jailhouse and that means that Jailhouse is finally running on AGL on Raspberry Pi 4, i.e. I am able to build an image that can run another cell next to AGL without any manual modifications.

The crash from last week was solved by adding `mem=768M` to the kernel command line. That limits the memory used by the AGL cell to 768 MiB, so it basically limits any Raspberry to 1 GiB of usable memory. It is because the cell configurations supplied with Jailhouse are made for the 1 GiB variant of RPi4. It should be possible to make more cell configurations for different memory variants, but they would need to be selected somehow. It cannot be done at build time, as the image build is universal for all the variants. That means that I would have to supply all the configurations in the image and the user would choose the right one for the machine. I decided that I don't want to hassle with that now, so I will stick to the 1G limit for now.

I've also made recipes/bbappends for ARM Trusted Firmware and `rpi-config`, that generates the `config.txt` for RPi. It's all in my [meta-jailhouse](https://github.com/Limoto/meta-jailhouse) layer.

The layer now depends on the `meta-arm` layer, or more precisely on its `meta-arm` and `meta-arm-bsp` parts. To build the image, you need to add those two layers together with my `meta-jailhouse`. Then, run `aglsetup.sh` with `agl-jailhouse` feature added.

Jailhouse can be tested this way:

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
    Activating hypervisor
    [   82.998239] pci-host-generic e0000000.pci: host bridge /pci@0 ranges:
    [   83.004913] pci-host-generic e0000000.pci:      MEM 0x00e0100000..0x00e0103fff -> 0x00e0100000
    [   83.013867] pci-host-generic e0000000.pci: ECAM at [mem 0xe0000000-0xe00fffff] for [bus 00]
    [   83.022405] Node /pci@0 has 
    [   83.022410] Inconsistent "linux,pci-domain" property in DT
    [   83.031161] pci-host-generic e0000000.pci: PCI host bridge to bus ffffffff:00
    [   83.038461] pci_bus ffffffff:00: root bus resource [bus 00]
    [   83.044167] pci_bus ffffffff:00: root bus resource [mem 0xe0100000-0xe0103fff]
    [   83.051581] pci ffffffff:00:00.0: [110a:4106] type 00 class 0xff0000
    [   83.058130] pci ffffffff:00:00.0: reg 0x10: [mem 0x00000000-0x00000fff]
    [   83.065490] pci ffffffff:00:01.0: [110a:4106] type 00 class 0xff0001
    [   83.072074] pci ffffffff:00:01.0: reg 0x10: [mem 0x00000000-0x00000fff]
    [   83.083098] pci ffffffff:00:00.0: BAR 0: assigned [mem 0xe0100000-0xe0100fff]
    [   83.090502] pci ffffffff:00:01.0: BAR 0: assigned [mem 0xe0101000-0xe0101fff]
    [   83.098148] uio_ivshmem ffffffff:00:00.0: enabling device (0000 -> 0002)
    [   83.105098] uio_ivshmem ffffffff:00:00.0: state_table at 0x000000003faf0000, size 0x0000000000001000
    [   83.114446] uio_ivshmem ffffffff:00:00.0: rw_section at 0x000000003faf1000, size 0x0000000000009000
    [   83.123686] uio_ivshmem ffffffff:00:00.0: input_sections at 0x000000003fafa000, size 0x0000000000006000
    [   83.133259] uio_ivshmem ffffffff:00:00.0: output_section at 0x000000003fafa000, size 0x0000000000002000
    [   83.143742] ivshmem-net ffffffff:00:01.0: enabling device (0000 -> 0002)
    [   83.150784] ivshmem-net ffffffff:00:01.0: TX memory at 0x000000003fb01000, size 0x000000000007f000
    [   83.159935] ivshmem-net ffffffff:00:01.0: RX memory at 0x000000003fb80000, size 0x000000000007f000
    [   83.170311] The Jailhouse is opening.
    raspberrypi4-64:~# jailhouse cell create /usr/share/jailhouse/cells/rpi4-inmate-demo.cell 
    [43412.966826] IRQ42: set affinity failed(-22).
    [43412.971291] CPU1: shutdown
    [43412.974050] psci: CPU1 killed (polled 0 ms)
    Adding virtual PCI device 00:00.0 to cell "inmate-demo"
    Shared memory connection established, peer cells:
    "Raspberry-Pi4"
    Created cell "inmate-demo"
    Page pool usage after cell creation: mem 75/994, remap 5/131072
    [43412.998084] Created Jailhouse cell "inmate-demo"
    raspberrypi4-64:~# jailhouse cell list                                                    
    ID      Name                    State             Assigned CPUs           Failed CPUs             
    0       Raspberry-Pi4           running           0,2-3                                           
    1       inmate-demo             shut down         1                                               
    raspberrypi4-64:~# jailhouse cell load 1 /usr/share/jailhouse/inmates/gic-demo.bin 
    Cell "inmate-demo" can be loaded
    raspberrypi4-64:~# jailhouse cell start 1
    Started cell "inmate-demo"
    Initializing the GIC...
    Initializing the timer...
    raspberrypi4-64:~# Timer fired, jitter:   2592 ns, min:   2592 ns, max:   2592 ns
    Timer fired, jitter:   1259 ns, min:   1259 ns, max:   2592 ns
    Timer fired, jitter:   1259 ns, min:   1259 ns, max:   2592 ns
    ...


For next week, I plan two tasks. The first is to get Jailhouse running also in QEMU and the second is to make two Raspberrys communicate with each other over CAN.