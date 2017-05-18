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

Download config

```
cd ~/ELFS/toolchain/sources/gcc-6.3.0
wget -O config.sub "http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=3d5db9ebe860"
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

## References

* [1] [musl-cross-make](https://github.com/richfelker/musl-cross-make)
* [2] [mkroot](https://github.com/landley/mkroot)
