## Running Linux in a non-root cell on a Raspberry Pi 4

To get Linux run in a non-root cell, a compatible kernel and initial ramdisk image is required. For simplicity, they can be taken from the [jailhouse-images](https://github.com/siemens/jailhouse-images) project. To do so, clone the repository and build the image for RPi4 target using:

    ./build-images.sh
    
and selecting the right target on the command line. After the build is successful, you can search for files called `rootfs.cpio` and `demo-image-jailhouse-demo-rpi4-vmlinuz` in the `tmp/work` subdirectory. These need to be copied over to the target.

Then, Jailhouse hypervisor can be enabled on the target:

    jailhouse enable /usr/share/jailhouse/cells/rpi4.cell

After doing that, eth1 network interface will show up. That is a virtual interface to communicate between the cells using shared memory. Let's give it an IP address and set it up:

    ip a a 192.168.19.1/24 dev eth1
    ip link set up dev eth1

Now, the Linux cell can be started:

    jailhouse cell linux /usr/share/jailhouse/cells/rpi4-linux-demo.cell demo-image-jailhouse-demo-rpi4-vmlinuz -d /usr/share/jailhouse/cells/dts/inmate-rpi4.dtb -i rootfs.cpio -c "console=ttyS0,115200 ip=192.168.19.2"

The boot process (Linux kernel boot) can be seen on the serial console of the RPi and after it finishes, a shell from the non-root Linux cell will pop up. The root cell is still running, that can be verified by simply pinging it:

    ping 192.168.19.1