---
title: Embedded Linux From Scratch (part I)
comments: true
---

<div class="alert">Warning! This is an working in progress article and its content may be incomplete and inconsistent.</div>

**Embedded Linux From Scratch** will be a series of 3 articles showing how to cross-build an embedded system. The host is a x86 machine and the target is an ARM machine.

## Build Toolchain

### Build GCC

Download GCC:

```
mkdir -p toolchain/sources
cd toolchain/sources
wget https://ftp.gnu.org/pub/gnu/gcc/gcc-6.3.0/gcc-6.3.0.tar.bz2
wget -O config.sub "http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=3d5db9ebe860"
tar jxvf gcc-6.3.0.tar.bz2
```

Apply patches:

```
TODO
```

### Build binutils

Download binutils:

```
wget https://ftp.gnu.org/pub/gnu/binutils/binutils-2.27.tar.bz2
```

## References

* [1] [musl-cross-make](https://github.com/richfelker/musl-cross-make)
* [2] [mkroot](https://github.com/landley/mkroot)
