# Installing generic arm64 distros on rk3588 devices with UEFI and GRUB
## This will (hopefully) be a tutorial on how to successfully install most generic arm64 distros on Rockchip rk3588 based devices.
## This tutorial assumes you know how to use the command line, are familiar with the distro you're trying to install, and how to manipulate disks and their partitions.

## Hardware Prequisites:
* an arm64 device already running linux
* a storage device you want to install the distro to
* an SD card to hold the bootloader(optional, can be flashed to SPI)
* a good internet connection

## Software Prequisites:
* `arch-chroot`, optional, but makes chrooting easier. packaged as `arch-install-scripts` on most distros
* `dosfstools`, for making the boot partition
* `fdisk`, for manipulating disks(shouldd be already installed)
* `bsdtar`,(only if you're using a roofs tarball) packaged as `libarchive-tools` on ubuntu & debian, `bsdtar` on arch.
* `ar`, for extracting the kernel .deb

## Download Prequisites:
* generic arm64 rootfs tarball or image of the distro you want to install
* the kernel deb from [https://launchpad.net/~jjriek/+archive/ubuntu/rockchip/+packages](https://launchpad.net/~jjriek/+archive/ubuntu/rockchip/+packages), it'll be in the `linux-rockchip` entry, and download the `linux-image-5.10.160-rockchip_XXX_arm64.deb` file
* the efi bootloader image for you device, from [https://github.com/edk2-porting/edk2-rk3588/releases](https://github.com/edk2-porting/edk2-rk3588/releases)

### small notes:
* will all be done as root
* `//` and everything after is a comment

# Preparing the disk (only needed if using a rootfs tarball):

```bash
# fdisk -l        //to find the device you want to install to eg. sdX, will be referred as <installdevice> from now on
# fdisk /dev/<installdevice>

fdisk prompt: g        //to create new gpt table
fdisk prompt: o        //make sure there are no more partitions
fdisk prompt: n        //make boot partition
fdisk prompt: 1        //select boot partition number
fdisk prompt: <enter>  //use default start for the boot partition
fdisk prompt: +1G      //set size of boot partition
fdisk prompt: t        //set partition type
fdisk prompt: 1        //to efi system
fdisk prompt: n        //make root partition
fdisk prompt: 2        //select root partition number
fdisk prompt: <enter>  //use default start for the root partition
fdisk prompt: <enter>  //use default end for the root partition
fdisk prompt: w        //write changes to disk and exit

# mkfs.vfat -F 32 /dev/<installdevice><p1>
# mkfs.ext4 /dev/<installdevice><p2>
# sync
```

# Using a disk image:
flash it to your device like you would normally

# Installing the rootfs tarball to the disk:
```bash
# mkdir boot root        //make mount dirs
# mount /dev/<installdevice><p1> boot  //mount boot partiton
# mount /dev/<installdevice><p2> root  //mount root partition
# bsdtar -xpf <your-rootfs-tarball> -C root  //extract tarball to root partition
# sync
# mv root/boot/* boot/  //move boot files to boot partition
# sync
# umount boot         //umount boot 
# mount /dev/<installdevice><p1> root/boot  //remount boot
#
```

