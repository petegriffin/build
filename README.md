This git-project is a helper project to be able to build U-Boot, TF-A, Linux
kernel etc as standalone components for iMX8MQ-evk boards. I.e., with this you
don't have to use Yocto etc.

# Prerequisites
```
$ apt install coreutils dosfstools e2fsprogs fdisk git kpartx pv sudo \
              util-linux wget
```

# Setup
Sync the git's needed using
[repo](https://source.android.com/setup/build/downloading). Following the link
for installation instructions.

```
$ repo init -u https://github.com/jbech-linaro/manifest.git -b imx8mqevk
```

Get the toolchain (GCC, only 64bit is used, but you'll get a 32bit GCC as well).
```
$ make -j2 toolchains
```

# Compile
This will compile all components, i.e., TF-A, U-Boot, Linux kernel, imx-mkimage.
```
$ make
```

# Flash
There are several ways to flash a device, it depends what you want to flash and
what method you prefer.

Be **super-careful** about the name you use for all `dd` commands. Since if you
use the wrong name you can accidentally wipe another hard drive on your local
computer.

## Create and write a complete image
This will create a complete image ready to flash to a SD-card using `dd`. The
script will require `sudo` access. Note that, you must run a normal `make` once
to compile all the binaries used before being able to run the command below.

```
$ make flash-image
```
This will create an image file called `imx8mqevk.img` that contains the
bootloader (U-boot, put at offset 33), a boot disk (Linux kernel and DTB) and
the rootfs disk. During creation it will map and mount the partitions to
`/media/boot` and `/media/rootfs` on your local machine. So, if you have those
in use already for other purposes, then you need to update the `create_image.sh`
to use other unused paths.

When the image has been created, insert your SD-card into your computer. Via
`sudo dmesg` or `lsblk` you should be able to figure out what device name it
got. I.e., something with `/dev/sdX` or `/dev/mmcblk[n]`. Use that device as
part of the `... of=<device>` in the command below. There we're using
`/dev/sdj`. Note when flashing the complete image you should use the device name
and not device name + partition number. I.e., `/dev/sdj` is correct, `/dev/sdj1`
is incorrect. The `make flash-image` will give an example also.

```
$ sudo dd if=build/imx8mqevk.img | pv | sudo dd of=/dev/sdj bs=1M conv=fsync
```

Note! If you need more space for the rootfs, simply change the size in the
`create_image.sh` script.

## Flash bootloader only
This flash directly to the SD-card. So after inserting the SD-card into your
computer, run:
```
$ make flash-bootloader
```
Type the last row (a `dd` command) to flash the bootloader to your device.


// Joakim Bech
Last updated: 2020-09-01

