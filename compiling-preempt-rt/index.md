# Compile Preempt RT kernel

## Environment and requirements

I ran this procedure on my Ubuntu Desktop 16.04.2 box. The following packages were needed:

```
sudo apt-get install git
sudo apt-get install libssl-dev
sudo apt-get install dpkg-dev
```

## Create the directory for the kernel source

The directory ```prempt-rt``` is the root of kernel source.

```
mkdir prempt-rt
cd prempt-rt
```

## Get PREMPT RT Patch

### Method 1

This method first gets the mainline kernel.
 
```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
git fetch origin v4.9.18
git reset --hard FETCH_HEAD

# Get and apply the prempt-rt patch.
git branch rt14
git checkout rt14
wget -O - https://www.kernel.org/pub/linux/kernel/projects/rt/4.9/older/patch-4.9.18-rt14.patch.gz | gunzip -c > /tmp/patch-4.9.18-rt14.patch
patch -p1 < /tmp/patch-4.9.18-rt14.patch
git add -A .
git commit -m "PREEMP_RT 14"
```

### Method 2

This method gets prempt-rt from its oficial git repositories ([stable](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git) or [develepment](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git/)).

```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git
git fetch origin v4.9.18-rt14-patches
git reset --hard FETCH_HEAD
```

## Configure the kernel

```
cp /boot/config-`uname -r` .config
make olddefconfig
make menuconfig
```

Config options:

* PREEMPT_RT_FULL = y
* HIGH_RES_TIMERS = y

## Compile kernel and create packages

```
make deb-pkg LOCALVERSION=-rt14
```

Created packages:

* linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14-dbg_4.9.18-rt14-rt14-5_amd64.deb
* linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-libc-dev_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb


## Install packages

```
sudo dpkg -i linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
```
