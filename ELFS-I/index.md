---
title: Embedded Linux From Scratch (part I)
comments: true
---

<div class="alert">Warning! This is an working in progress article and its content may be incomplete and inconsistent.</div>

**Embedded Linux From Scratch** (ELFS) will be a series of 3 articles showing how to cross-build an embedded system. The host is a x86 machine and the target is an ARM machine.

## Create directory structure

```
mkdir -p ~/ELFS/toolchain/sources
```

## Build Toolchain

### Get musl-cross-make

```
cd ~/ELFS/toolchain/sources
git clone https://github.com/richfelker/musl-cross-make.git
```

### Get config.sub

```
cd ~/ELFS/toolchain/sources/
wget -O config.sub "http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=3d5db9ebe860"
```

### Get GCC

Download and uncompress GCC:

```
cd ~/ELFS/toolchain/sources
wget https://ftp.gnu.org/pub/gnu/gcc/gcc-6.3.0/gcc-6.3.0.tar.bz2
tar jxvf gcc-6.3.0.tar.bz2
```

Apply patches:

```
cd ~/ELFS/toolchain/sources/gcc-6.3.0
cat ~/ELFS/toolchain/sources/musl-cross-make/patches/gcc-6.3.0/* | patch -p1
```

### Get binutils

Download and uncompress binutils:

```
cd ~/ELFS/toolchain/sources
wget https://ftp.gnu.org/pub/gnu/binutils/binutils-2.27.tar.bz2
tar jxvf binutils-2.27.tar.bz2
```

Apply patches:

```
cd ~/ELFS/toolchain/sources/binutils-2.27/
cat ~/ELFS/toolchain/sources/musl-cross-make/patches/binutils-2.27/* | patch -p1
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/sources/config.sub ~/ELFS/toolchain/sources/binutils-2.27
```

### Get musl

Download and uncompress musl

```
cd ~/ELFS/toolchain/sources
wget https://www.musl-libc.org/releases/musl-1.1.16.tar.gz
tar zxvf musl-1.1.16.tar.gz
```

### Get gmp

Download and uncompress gmp

```
cd ~/ELFS/toolchain/sources
wget https://ftp.gnu.org/pub/gnu/gmp/gmp-6.1.1.tar.bz2
tar jxvf gmp-6.1.1.tar.bz2
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/sources/config.sub ~/ELFS/toolchain/sources/gmp-6.1.1
```

### Get mpc

Download and uncompress mpc

```
cd ~/ELFS/toolchain/sources
wget https://ftp.gnu.org/pub/gnu/mpc/mpc-1.0.3.tar.gz
tar zxvf mpc-1.0.3.tar.gz
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/sources/config.sub ~/ELFS/toolchain/sources/mpc-1.0.3
```

### Get mpfr

Download and uncompress mpfr

```
cd ~/ELFS/toolchain/sources
wget https://ftp.gnu.org/pub/gnu/mpfr/mpfr-3.1.4.tar.bz2
tar jxvf mpfr-3.1.4.tar.bz2
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/sources/config.sub ~/ELFS/toolchain/sources/mpfr-3.1.4
```

### Get Linux
 
Download and uncompress Linux:

```
cd ~/ELFS/toolchain/sources
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.10.tar.xz
tar Jxvf linux-4.4.10.tar.xz
```

Apply patches:

```
cd ~/ELFS/toolchain/sources/linux-4.4.10/
cat ~/ELFS/toolchain/sources/musl-cross-make/patches/linux-4.4.10/* | patch -p1
```

## References

* [1] [musl-cross-make](https://github.com/richfelker/musl-cross-make)
* [2] [mkroot](https://github.com/landley/mkroot)
