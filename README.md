# bcm2711-kernel
Automated build of the latest 64-bit `bcm2711_defconfig` Linux kernel for the Raspberry Pi 4

## Description

<img src="https://raw.githubusercontent.com/sakaki-/resources/master/raspberrypi/pi4/Raspberry_Pi_4_B.jpg" alt="Raspberry Pi 4 B" width="250px" align="right"/>

This project contains a weekly autobuild of the default branch (currently, `rpi-4.19.y`) of the [official Raspberry Pi Linux source tree](https://github.com/raspberrypi/linux), for the [64-bit Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/).

Builds are performed with the standard `bcm2711_defconfig`, with the only change being that the first 12 hex digits of the tip commit SHA1 hash plus `-p4` are appended to `CONFIG_LOCALVERSION` (with a separating hyphen) before building.

A new build tarball is automatically created and uploaded as a release asset each week (unless the tip of the default branch is unchanged from the prior week, or an error occurs during the build process).

> The default branch is used, as that is generally given most attention by RPF upstream.

Each kernel release tarball currently provides the following files:
* `/boot/kernel8-64bit.img` (this is the bootable 64-bit kernel);
* `/boot/COPYING.linux` (the kernel's license file);
* `/boot/config-p4` (the configuration used to build the kernel);
* `/boot/Module-p4.symvers.xz` (a table mapping exported symbols to provider, compressed);
* `/boot/System-p4.map.xz` (the kernel's symbol table, compressed);
* `/boot/bcm2711-rpi-4-b.dtb` (the device tree blob; currently only one);
* `/boot/armstub8-gic.bin` (stubs required for the GIC);
* `/boot/overlays/...` (the device tree blob overlays);
* `/lib/modules/<kernel release name>/...` (the module set for the kernel).

> The `/boot/Module-p4.symvers.xz` file is only included in more recent builds. The `/boot/System-p4.map.xz` is supplied in compressed form only in recent builds.

The current kernel tarball may be downloaded from the link below (or via `wget`):

Variant | Version | Most Recent Image
:--- | ---: | ---:
Kernel, dtbs, modules and GIC stub | 4.19.102 | [kernel-latest.tar.xz](https://github.com/TwinUsers/pi-kernel/releases/download/4.19.x-64bit/kernel-latest.tar.xz)

## <a name="installation"></a>Installation

You can simply untar a kernel release tarball from this project into an existing (32 or 64-bit) OS image to deploy it.

For example, to allow the current 32-bit userland Raspbian (with desktop) image to be booted under a 64-bit kernel on a Pi4, proceed as follows.

> For simplicity, I will assume you are working on a Linux PC, as root, here.

Begin by downloading and writing the Raspbian Buster image onto a _unused_ microSD card, and mounting it. Assuming the card appears as `/dev/mmcblk0` on your PC, and your OS image has two partitions (bootfs on the first, rootfs on the second, as Raspbian does), issue:

```console
linuxpc ~ # wget -cO- https://downloads.raspberrypi.org/raspbian_latest | bsdtar -xOf- > /dev/mmcblk0
linuxpc ~ # sync && partprobe /dev/mmcblk0
linuxpc ~ # mkdir -pv /mnt/piroot
linuxpc ~ # mount -v /dev/mmcblk0p2 /mnt/piroot
linuxpc ~ # mount -v /dev/mmcblk0p1 /mnt/piroot/boot
```

> NB: you **must** take care to substitute the correct path for your microSD card (which may appear as something completely different from `/dev/mmcblk0`, depending on your system) in these instructions, as the contents of the target drive will be irrevocably overwritten by the above operation.

Next, fetch the the current kernel tarball, and untar it into the mounted image. Issue:

```console
linuxpc ~ # wget -cO- https://github.com/TwinUsers/pi-kernel/releases/download/4.19.x-64bit/kernel-latest.tar.xz | tar -xJf- -C /mnt/piroot/
```

Then, edit the image's `/boot/config.txt`:

```console
linuxpc ~ # nano -w /mnt/piroot/boot/config.txt
```

Modify the [pi4] section of this file (it appears near the end of the file), so it reads as follows:
```ini
[pi4]
# Enable DRM VC4 V3D driver on top of the dispmanx display stack
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
# memory must be clamped at 1GiB for current 64-bit Pi4 kernels
# restriction will hopefully be removed shortly
total_mem=1024
arm_64bit=1
enable_gic=1
armstub=armstub8-gic.bin
# differentiate from Pi3 64-bit kernels
kernel=kernel8-p4.img
```

Leave the rest of the file as-is. Save, and exit `nano`. Then unmount the image:

```console
linuxpc ~ # sync
linuxpc ~ # umount -v /mnt/piroot/{boot,}
```

If you now remove the microSD card, insert it into a RPi4, and power on, you should find it starts up under the 64-bit kernel! 
