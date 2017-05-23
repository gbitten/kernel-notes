---
title: Embedded Linux From Scratch (part I)
comments: true
---

<div class="alert">Warning! This is an working in progress article and its content may be incomplete and inconsistent.</div>

**Embedded Linux From Scratch** (ELFS) will be a series of 3 articles showing how to cross-build an embedded system. The host is a x86 machine and the target is an ARM machine.

## Add the ELFS user

### Create the user and set its password:

```
sudo groupadd elfs
sudo useradd -s /bin/bash -g elfs -m -k /dev/null elfs
sudo passwd elfs
```

### Login the user

```
su - elfs
```

### Set up the user enviroment

```
cat > ~/.bash_profile << "EOF"
exec env -i HOME=${HOME} TERM=${TERM} PS1='\u:\w\$ ' /bin/bash
EOF
```

```
cat > ~/.bashrc << "EOF"
set +h
umask 022
LC_ALL=POSIX
PATH=~/toolchain/sysroot/bin:/bin:/usr/bin
export LC_ALL PATH
EOF
```

```
source ~/.bash_profile
```

## Get toolchain

### Create directory structure

```
mkdir -p ~/srcs
mkdir -p ~/toolchain/objs
mkdir -p ~/toolchain/sysroot
```

### Get musl-cross-make

```
cd ~/srcs
git clone https://github.com/richfelker/musl-cross-make.git
```

### Get config.sub

```
cd ~/srcs
wget -O config.sub "http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=3d5db9ebe860"
```

### Get GCC

Download and uncompress GCC:

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/gcc/gcc-6.3.0/gcc-6.3.0.tar.bz2
tar jxvf gcc-6.3.0.tar.bz2
```

Apply patches:

```
cd ~/srcs/gcc-6.3.0
cat ~/srcs/musl-cross-make/patches/gcc-6.3.0/* | patch -p1
```

Copy config.sub:

```
cp -f ~/srcs/config.sub ~/srcs/gcc-6.3.0
```

### Get binutils

Download and uncompress binutils:

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/binutils/binutils-2.27.tar.bz2
tar jxvf binutils-2.27.tar.bz2
```

Apply patches:

```
cd ~/srcs/binutils-2.27/
cat ~/srcs/musl-cross-make/patches/binutils-2.27/* | patch -p1
```

Copy config.sub:

```
cp -f ~/srcs/config.sub ~/srcs/binutils-2.27
```

### Get musl

Download and uncompress musl

```
cd ~/srcs
wget https://www.musl-libc.org/releases/musl-1.1.16.tar.gz
tar zxvf musl-1.1.16.tar.gz
```

### Get gmp

Download and uncompress gmp

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/gmp/gmp-6.1.1.tar.bz2
tar jxvf gmp-6.1.1.tar.bz2
```

Copy config.sub:

```
cp -f ~/srcs/config.sub ~/srcs/gmp-6.1.1
```

### Get mpc

Download and uncompress mpc

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/mpc/mpc-1.0.3.tar.gz
tar zxvf mpc-1.0.3.tar.gz
```

Copy config.sub:

```
cp -f ~/srcs/config.sub ~/srcs/mpc-1.0.3
```

### Get mpfr

Download and uncompress mpfr

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/mpfr/mpfr-3.1.4.tar.bz2
tar jxvf mpfr-3.1.4.tar.bz2
```

Copy config.sub:

```
cp -f ~/srcs/config.sub ~/srcs/mpfr-3.1.4
```

### Get Linux
 
Download and uncompress Linux:

```
cd ~/srcs
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.10.tar.xz
tar Jxvf linux-4.4.10.tar.xz
```

Apply patches:

```
cd ~/srcs/linux-4.4.10/
cat ~/srcs/musl-cross-make/patches/linux-4.4.10/* | patch -p1
```

## Build toolchain

### Build GCC

Create links for gmp mpc and mpfr sources

```
cd ~/srcs/gcc-6.3
ln -sf ../gmp-6.1.1 gmp
ln -sf ../mpc-1.0.3 mpc
ln -sf ../mpfr-3.1.4 mpfr
```

Configure Makefile with the following options<sup>[4]</sup>:

* `--enable-languages`: Specify that only a particular subset of compilers and their runtime libraries should be built.
* `--with-float`: Set the compiler option `-mhard-float` as default. The `-mhard-float` flag sets the GCC compiler to generates the output containing floating point instructions.


```
cd ~/toolchain/objs
../srcs/gcc-6.3.0/configure --enable-languages=c,c++  --with-float=hard \
  --disable-werror --target=arm-linux-musleabihf --prefix= --libdir=/lib \
  --disable-multilib --with-sysroot=/arm-linux-musleabihf --enable-tls \
  --disable-libmudflap --disable-libsanitizer --disable-gnu-indirect-function \
  --disable-libmpx --enable-libstdcxx-time \
  --with-build-sysroot=~/toolchain/sysroot
```

Build GCC:

```
cd ~/toolchain/objs
ln -sf . ../sysroot/usr
ln -sf lib ../sysroot/lib64
mkdir -p ~/toolchain/ssysroot/include
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false MAKE="make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= \
  ac_cv_prog_lex_root=lex.yy.c MAKEINFO=false" all-gcc
```
## References

* [1] [Cross-Compiled Linux From Scratch - Embedded](http://clfs.org/view/clfs-embedded/arm/index.html)
* [2] [musl-cross-make](https://github.com/richfelker/musl-cross-make)
* [3] [mkroot](https://github.com/landley/mkroot)
* [4] [Installing GCC](https://gcc.gnu.org/install/configure.html)
