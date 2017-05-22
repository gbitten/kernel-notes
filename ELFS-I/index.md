---
title: Embedded Linux From Scratch (part I)
comments: true
---

<div class="alert">Warning! This is an working in progress article and its content may be incomplete and inconsistent.</div>

**Embedded Linux From Scratch** (ELFS) will be a series of 3 articles showing how to cross-build an embedded system. The host is a x86 machine and the target is an ARM machine.

## Create directory structure

```
mkdir -p ~/ELFS/toolchain/srcs
mkdir -p ~/ELFS/toolchain/objs
mkdir -p ~/ELFS/toolchain/sysroot
```

## Get toolchain

### Get musl-cross-make

```
cd ~/ELFS/toolchain/srcs
git clone https://github.com/richfelker/musl-cross-make.git
```

### Get config.sub

```
cd ~/ELFS/toolchain/srcs
wget -O config.sub "http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=3d5db9ebe860"
```

### Get GCC

Download and uncompress GCC:

```
cd ~/ELFS/toolchain/srcs
wget https://ftp.gnu.org/pub/gnu/gcc/gcc-6.3.0/gcc-6.3.0.tar.bz2
tar jxvf gcc-6.3.0.tar.bz2
```

Apply patches:

```
cd ~/ELFS/toolchain/srcs/gcc-6.3.0
cat ~/ELFS/toolchain/srcs/musl-cross-make/patches/gcc-6.3.0/* | patch -p1
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/srcs/config.sub ~/ELFS/toolchain/srcs/gcc-6.3.0
```

### Get binutils

Download and uncompress binutils:

```
cd ~/ELFS/toolchain/srcs
wget https://ftp.gnu.org/pub/gnu/binutils/binutils-2.27.tar.bz2
tar jxvf binutils-2.27.tar.bz2
```

Apply patches:

```
cd ~/ELFS/toolchain/srcs/binutils-2.27/
cat ~/ELFS/toolchain/srcs/musl-cross-make/patches/binutils-2.27/* | patch -p1
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/srcs/config.sub ~/ELFS/toolchain/srcs/binutils-2.27
```

### Get musl

Download and uncompress musl

```
cd ~/ELFS/toolchain/srcs
wget https://www.musl-libc.org/releases/musl-1.1.16.tar.gz
tar zxvf musl-1.1.16.tar.gz
```

### Get gmp

Download and uncompress gmp

```
cd ~/ELFS/toolchain/srcs
wget https://ftp.gnu.org/pub/gnu/gmp/gmp-6.1.1.tar.bz2
tar jxvf gmp-6.1.1.tar.bz2
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/srcs/config.sub ~/ELFS/toolchain/srcs/gmp-6.1.1
```

### Get mpc

Download and uncompress mpc

```
cd ~/ELFS/toolchain/srcs
wget https://ftp.gnu.org/pub/gnu/mpc/mpc-1.0.3.tar.gz
tar zxvf mpc-1.0.3.tar.gz
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/srcs/config.sub ~/ELFS/toolchain/srcs/mpc-1.0.3
```

### Get mpfr

Download and uncompress mpfr

```
cd ~/ELFS/toolchain/srcs
wget https://ftp.gnu.org/pub/gnu/mpfr/mpfr-3.1.4.tar.bz2
tar jxvf mpfr-3.1.4.tar.bz2
```

Copy config.sub:

```
cp -f ~/ELFS/toolchain/srcs/config.sub ~/ELFS/toolchain/srcs/mpfr-3.1.4
```

### Get Linux
 
Download and uncompress Linux:

```
cd ~/ELFS/toolchain/srcs
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.10.tar.xz
tar Jxvf linux-4.4.10.tar.xz
```

Apply patches:

```
cd ~/ELFS/toolchain/srcs/linux-4.4.10/
cat ~/ELFS/toolchain/srcs/musl-cross-make/patches/linux-4.4.10/* | patch -p1
```

## Build toolchain

### Build GCC

Create links for gmp mpc and mpfr sources

```
cd ~/ELFS/toolchain/srcs/gcc-6.3
ln -sf ../gmp-6.1.1 gmp
ln -sf ../mpc-1.0.3 mpc
ln -sf ../mpfr-3.1.4 mpfr
```

Configure Makefile with the following options<sup>[3]</sup>:

* `--enable-languages`: Specify that only a particular subset of compilers and their runtime libraries should be built.
* `--with-float`: Set the compiler option `-mhard-float` as default. The `-mhard-float` flag sets the GCC compiler to generates the output containing floating point instructions.


```
cd ~/ELFS/toolchain/objs
../srcs/gcc-6.3.0/configure --enable-languages=c,c++  --with-float=hard \
  --disable-werror --target=arm-linux-musleabihf --prefix= --libdir=/lib \
  --disable-multilib --with-sysroot=/arm-linux-musleabihf --enable-tls \
  --disable-libmudflap --disable-libsanitizer --disable-gnu-indirect-function \
  --disable-libmpx --enable-libstdcxx-time \
  --with-build-sysroot=~/ELFS/toolchain/sysroot
```

Build GCC:

```
ln -sf . ../sysroot/usr
ln -sf lib ../sysroot/lib64
mkdir -p ~/ELFS/toolchain/ssysroot/include
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false MAKE="make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= \
  ac_cv_prog_lex_root=lex.yy.c MAKEINFO=false" all-gcc
```
## References

* [1] [musl-cross-make](https://github.com/richfelker/musl-cross-make)
* [2] [mkroot](https://github.com/landley/mkroot)
* [3] [Installing GCC](https://gcc.gnu.org/install/configure.html)
