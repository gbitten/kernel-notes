---
title: Ubuntu Base with QEMU
comments: true
---

<div class="alert">Warning! This is an working in progress article and its content may be incomplete and inconsistent.</div>

## About Ubuntu Base

**Ubuntu Base** is a minimal rootfs. It is highly stripped, but can install any package from Ubuntu repositories using the `apt-get` command. Because that, it is a useful tool to build custom systems. Ubuntu Base was previously known as Ubuntu Core, but Canonical renamed it and gave that name to other project. There is a great tutorial about its installation in Ask Ubuntu<sup>[2]</sup>. I am following the same path from this Ask Ubuntu page, but here I am installing on a QEMU virtual machine. 

## About QEMU

**QEMU** is an open source virtual machine monitor (VMM) or a hypervisor. It can virtualize hardwares from many architectures, including x86, SPARC, ARM, PowerPC, MIPs, Microblaze and others.

## Prepare the environment

Install QEMU

```
sudo apt-get install qemu
```

Download an Live CD

```
wget http://releases.ubuntu.com/16.04.2/ubuntu-16.04.2-desktop-amd64.iso
```

## Install 

Create QEMU image

```
qemu-img create ubuntu-base.img 100M
```

Boot from Ubuntu Live CD 

```
qemu-system-x86_64 -cdrom ubuntu-16.04.2-desktop-amd64.iso -hda ubuntu-base.img -boot d -m 512
```

After the Ubuntu boot , click o "Try Ubuntu" and open a terminal in the virtual machine.


## References

* [1] [Ubuntu Base](https://wiki.ubuntu.com/Base)
* [2] [What commands are needed to install Ubuntu Core?](https://askubuntu.com/a/70139/413551)
* [3] [QEMU](http://www.qemu.org/)
* [4] [Ubuntu Base releases](http://cdimage.ubuntu.com/ubuntu-base/releases/)

