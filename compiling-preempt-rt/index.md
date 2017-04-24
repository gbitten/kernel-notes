---
title: Compile Preempt RT
comments: true
---

I usually run this procedure (with some variations if necessary) to compile and install the Linux kernel with Preempt RT<sup>[1]</sup> on my Ubuntu box.

## Prepare the build environment

```
sudo apt-get install git
sudo apt-get install libssl-dev
sudo apt-get install dpkg-devi
sudo apt-get build-dep linux-image-$(uname -r)
```

## Create the directory for the kernel source

The directory ```prempt-rt``` is the root of kernel source.

```
mkdir prempt-rt
cd prempt-rt
```

## Get PREMPT RT Patch

### Method 1

First, download the mainline kernel.
 
```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
git fetch origin v4.9.18
git reset --hard FETCH_HEAD
```

Then, apply the prempt-rt patch.

```
git branch rt14
git checkout rt14
wget -O - https://www.kernel.org/pub/linux/kernel/projects/rt/4.9/older/patch-4.9.18-rt14.patch.gz | gunzip -c > /tmp/patch-4.9.18-rt14.patch
patch -p1 < /tmp/patch-4.9.18-rt14.patch
git add -A .
git commit -m "PREEMP_RT 14"
```

### Method 2

Get prempt-rt from its oficial git repositories ([stable](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git) or [develepment](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git/)).

```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git
git fetch origin v4.9.18-rt14-patches
git reset --hard FETCH_HEAD
```

## Configure the kernel


### Create the base ```.config```

There are many alternatives to define the base ```.config```. 

```
cp /boot/config-`uname -r` .config
make olddefconfig
```

### Set some important paramentes for real-time systems

```
make menuconfig
```

Config options:

* PREEMPT_RT_FULL = y
* HIGH_RES_TIMERS = y

## Compile and install the kernel

Those are the main methods (with some variations when is needed) that I use to compile and install the kernel. 

### Method 1

This is the traditional compilation/instalation method. It installs the kernel locally.

```
make -j $(nproc) LOCALVERSION=-rt14
sudo make modules_install install
```

### Method 2

Instead of a local instalation, this method creates packages which could be installed on other machines. The kernel's ```make``` has options for different kind of packages <sup>[2]</sup>, the option ```deb-pkg``` generates debian packages.

```
make -j $(nproc) deb-pkg LOCALVERSION=-rt14
```

The above command creates 5 packages:

* linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14-dbg_4.9.18-rt14-rt14-5_amd64.deb
* linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-libc-dev_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb

Install the kernel packages.

```
sudo dpkg -i linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
```

## References

* [1] [Preempt RT patch](https://rt.wiki.kernel.org/index.php/CONFIG_PREEMPT_RT_Patch)
* [2] [Output of kernel's "make help"](https://www.kernel.org/doc/makehelp.txt)
