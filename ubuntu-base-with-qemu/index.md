---
title: Ubuntu Base with QEMU
comments: true
---

<div class="alert">Warning! This is an working in progress article and its content may be incomplete and inconsistent.</div>

This procedure shows how to create a QEMU<sup>[1]</sup> virtual machine with Ubuntu Base<sup>[2]</sup>.  There is a great post about Ubuntu Base installation in Ask Ubuntu<sup>[3]</sup>. I am following the same path, but here I am installing on a QEMU virtual machine. It worth mentioning that this procedure will work only if the host and the virtual machine have the same architecture.

## About Ubuntu Base

**Ubuntu Base** is a minimal rootfs. Although It is highly stripped, but can install any package from Ubuntu repositories using the `apt-get` command. Because that, it is a useful tool to build custom systems. Ubuntu Base was previously known as Ubuntu Core, but Canonical renamed it and gave that name to other project. 

## About QEMU

**QEMU** is an open source virtual machine monitor (VMM) or a hypervisor. It can virtualize hardwares from many architectures, including x86, SPARC, ARM, PowerPC, MIPs, Microblaze and others.

## Download Ubuntu Base

Choose the version and download the Ubuntu Base [here](http://cdimage.ubuntu.com/ubuntu-basereleases/). This procedure will work only if the Ubuntu Base has the same architecture than the host machine. In my case, I downloaded the version 16.04 for i386. 

```
wget http://cdimage.ubuntu.com/ubuntu-basereleases/16.04/release/ubuntu-base-16.04-core-i386.tar.gz
```

## Install QEMU

```
sudo apt-get install qemu
```

## Create QEMU image

```
qemu-img create ubuntu-base.img 100M
```

## Log as root

All of the following commands require root right.

```
sudo su
```

## Create a Network Block Device

QEMU can export a disk image as a Network Block Device (NBD). With that, the host can handle the QEMU image as any other block device.  

```
modprobe nbd max_part=16
qemu-nbd -c /dev/nbd0 ubuntu-base.img
```

## Partionion the device

```
fdisk /dev/nbd0
```

Then press `n` `p` `1` `<Return>` `<Return>` `a` `1` `w`. This sequence creates a single partition on the image.

## Format the partition

```
mkfs.ext4 /dev/nbd0p1
```

## Mount the partition

```
mount /dev/nbd0p1 /mnt
cd /mnt
tar xvfz /path/to/where/you/put/ubuntu-base-16.04-core-i386.tar.gz
```

## Install bootloader

```
grub-install --force --root-directory=/mnt /dev/nbd0
```

## Copy DNS configuration

```
cp /etc/resolv.conf /mnt/etc/resolv.conf
```

## Run `chroot`

```
mount --rbind /sys /mnt/sys
mount --rbind /proc /mnt/proc
mount --rbind /dev /mnt/dev
chroot /mnt
```

### Install the kernel

```
apt-get update 
apt-get install linux-image-4.4.0-75-generic
```

### Adjust grub configuration

```
sed -i s/nbd0p1/sda1/g /boot/grub/grub.cfg
```

###  Change the root password

```
passwd root
```

## Close procedure

Exit from `chroot`, dismount the partition, disconnect device and exit from root.

```
exit
umount -l /mnt
qemu-nbd -d /dev/nbd0
exit
```

## Run the virtual machine

```
qemu-system-i386 ubuntu-base.img
```

## References

* [1] [QEMU](http://www.qemu.org/)
* [2] [Ubuntu Base](https://wiki.ubuntu.com/Base)
* [3] [What commands are needed to install Ubuntu Core?](https://askubuntu.com/a/70139/413551)

