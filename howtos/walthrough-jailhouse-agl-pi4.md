# Jailhouse with AGL on RPi4 Walk-through

This guide shows all the necessary steps from downloading and building Automotive Grade Linux to actually running a simple inmate (bare metal code) in a non-root cell, next to the running Linux root cell.

## Downloading AGL

To download AGL, the repo tool is required. To check all the necessary dependencies, read the article [Preparing Your Build Host](https://docs.automotivelinux.org/docs/en/master/getting_started/reference/getting-started/image-workflow-prep-host.html) in the AGL documentation.

Make an empty directory on the build machine and change into it. Then, initialize the repository by running:

    repo init -b master -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo

If you want to rather use a version that is proven to work, use Jellyfish RC3 version instead:

    repo init -m jellyfish_9.99.3.xml -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo

Then, download all the AGL layers and its dependencies by running:

    repo sync

## Building AGL

Note: the complete build requires about 100 GiB of disk space and takes 2-3 hours on an 8-core machine.

First, ininitize the build environment by running:

    source meta-agl/scripts/aglsetup.sh -m raspberrypi4 agl-demo agl-netboot agl-appfw-smack agl-jailhouse

Then, run the actual build:

    bitbake agl-demo-platform

After the build is finished, insert the SD card for the Raspberry Pi into the reader on the build machine and flash the build image onto it.
Note that the following command expects the SD card to show up as /dev/mmcblk0. If that's not your case, following the AGL documentation chapter [Deploying the AGL Demo Image](https://docs.automotivelinux.org/docs/en/master/getting_started/reference/getting-started/machines/raspberrypi.html#deploying-the-agl-demo-image) to find out. Also, make sure the SD card filesystem is not mounted.

    xzcat tmp/deploy/images/raspberrypi4-64/agl-demo-platform-raspberrypi4.wic.xz | sudo dd of=/dev/mmcblk0 bs=8M

To make sure all the data is flushed onto the SD card, run:

    sync

After that, you can plug the SD card into the Raspberry, connect a screen or any other peripherals and it will boot into the AGL demo system.

## Running Jailhouse

To play with Jailhouse, the best way is to connect the serial console. To do so, follow the [Debugging](https://docs.automotivelinux.org/docs/en/master/getting_started/reference/getting-started/machines/raspberrypi.html#debugging) section of the AGL docs.

When connected to the serial console, you can log in as root without a password.

To enable Jailhouse hypervisor and move the running system into the root cell, execute:

    jailhouse enable /usr/share/jailhouse/cells/rpi4.cell

Now, you can create a non-root cell to run an inmate demo. This will remove one CPU core from the Linux root cell:

    jailhouse cell create /usr/share/jailhouse/cells/rpi4-inmate-demo.cell

For the demonstration, the gic-demo inmate can be used. It measures the jitter of timer interrupts and spams the serial console with results. To load the actual executable code into memory, run:

    jailhouse cell load inmate-demo /usr/share/jailhouse/inmates/gic-demo.bin

Finally, start the cel by issuing:

    jailhouse cell start inmate-demo

Now, you should see the results of jitter measurement of the serial console.
