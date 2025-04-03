# RT-Kernel
How to compile 64-bit RT-kernel for Raspberry Pi 5 for Debian bookworm (i.e., /boot/firmware/)

## Prepare the environment
```bash
sudo apt install git bc bison flex libssl-dev make
sudo apt install libncurses5-dev
sudo apt install raspberrypi-kernel-headers
```
## Clone the git, in this case kernel 6.14, from from https://github.com/raspberrypi/linux/tree/rpi-6.14.y
```bash
cd ~
git clone --depth 1 --branch rpi-6.14.y https://github.com/raspberrypi/linux
```

## Make for Raspberry Pi 5
```bash
make bcm2712_defconfig
```
## Start menuconfig
```bash
cp ~/src/RT-Kernel/bcm2712_config .config
make menuconfig
```

## Build the kernel using all cores (and try gcc optimization level -O3, if you like)
```bash
make prepare
make CFLAGS='-O3 -march=native' -j6 Image.gz modules dtbs # recommendation is 1.5 times the number of cores (=4), which equals 6
sudo make -j6 modules_install # recommendation is 1.5 times the number of cores (=4), which equals 6
```
## Create the required directories once
```bash
sudo mkdir /boot/firmware/RT
sudo mkdir /boot/firmware/RT/overlays-RT
```
## Add this to /boot/firmware/config.txt in order to preserve the standard kernel
```bash
os_prefix=RT/
overlay_prefix=overlays-RT/
kernel=/kernel_2712-RT.img
```
## Copy the file ino the right directories
```bash
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/RT/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/RT/overlays-RT/
sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/RT/overlays-RT/
sudo cp arch/arm64/boot/Image.gz /boot/firmware/kernel_2712-RT.img
```
## Reboot to activate the kernel
```bash
sudo reboot now
```
## Update the firmware (but not the standard kernel)
```bash
sudo SKIP_KERNEL=1 PRUNE_MODULES=1 rpi-update rpi-6.14.y
```

## Build status for official rpi-6.14.y from https://github.com/raspberrypi/linux:
[![Pi kernel build tests](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml/badge.svg?branch=rpi-6.14.y)](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml)

[![dtoverlaycheck](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml/badge.svg?branch=rpi-6.14.y)](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml)
