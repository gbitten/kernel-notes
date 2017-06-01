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
PATH=/bin:/usr/bin
export LC_ALL PATH
EOF
```

```
source ~/.bash_profile
```

## Get toolchain

### Create source directory

```
mkdir -p ~/srcs
```

### Get musl-cross-make

```
cd ~/srcs
git clone https://github.com/richfelker/musl-cross-make.git
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

### Get mpc

Download and uncompress mpc

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/mpc/mpc-1.0.3.tar.gz
tar zxvf mpc-1.0.3.tar.gz
```

### Get mpfr

Download and uncompress mpfr

```
cd ~/srcs
wget https://ftp.gnu.org/pub/gnu/mpfr/mpfr-3.1.4.tar.bz2
tar jxvf mpfr-3.1.4.tar.bz2
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

### Create a combined tree<sup>[4]</sup>

```
mkdir -p ~/srcs/combined
ln -sf ~/srcs/binutils-2.27/* ~/srcs/combined/
ln -sf ~/srcs/gcc-6.3.0/* ~/srcs/combined/
ln -sf ~/srcs/gmp-6.1.1 ~/srcs/combined/gmp
ln -sf ~/srcs/mpc-1.0.3 ~/srcs/combined/mpc
ln -sf ~/srcs/mpfr-3.1.4 ~/srcs/combined/mpfr
```

## Build toolchain

### Create toolchain directory

```
mkdir -p ~/toolchain
```

### Configure GCC's Makefile 

Configure options<sup>[5]</sup>:

* `--enable-languages`: Specify that only a particular subset of compilers and their runtime libraries should be built.
* `--with-float`: Set the compiler option `-mhard-float` as default. The `-mhard-float` flag sets the GCC compiler to generates the output containing floating point instructions.
* `--with-build-sysroot`: Set a directory to be consider as the system root while building target libraries, instead of the directory specified with `--with-sysroot`. The `readlink` command is used to convert the directory to a full path.

```
mkdir -p ~/toolchain/build/obj_toolchain
cd ~/toolchain/build/obj_toolchain
~/srcs/combined/configure --enable-languages=c,c++  --with-float=hard \
  --disable-werror --target=arm-linux-musleabihf --prefix= --libdir=/lib \
  --disable-multilib --with-sysroot=/arm-linux-musleabihf --enable-tls \
  --disable-libmudflap --disable-libsanitizer --disable-gnu-indirect-function \
  --disable-libmpx --enable-deterministic-archives --enable-libstdcxx-time \
  --with-build-sysroot=$(readlink -f ~/toolchain/build/obj_sysroot)
```

### Build GCC's cross-compilers only

```
cd ~/toolchain/build/
mkdir -p obj_sysroot
ln -sf . obj_sysroot/usr
ln -sf lib obj_sysroot/lib64
mkdir -p obj_sysroot/include
cd ~/toolchain/build/obj_toolchain
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false MAKE="make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c MAKEINFO=false" \
  all-gcc
```

### Configure musl's Makefile

```
mkdir -p ~/toolchain/build/obj_musl
cd ~/toolchain/build/obj_musl
~/srcs/musl-1.1.16/configure --prefix= --host=arm-linux-musleabihf \
  CC="../obj_toolchain/gcc/xgcc -B ../obj_toolchain/gcc" \
  LIBCC="../obj_toolchain/arm-linux-musleabihf/libgcc/libgcc.a" 
```

### Install musl's headers

```
cd ~/toolchain/build/obj_musl
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false \
  DESTDIR=~/toolchain/build/obj_sysroot/usr \
  install-headers
```

### Build target libgcc

```
cd ~/toolchain/build/obj_toolchain
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false MAKE="make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c MAKEINFO=false enable_shared=no" \
  all-target-libgcc
```

### Build musl

```
cd ~/toolchain/build/obj_musl
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false AR=../obj_toolchain/binutils/ar \
  RANLIB=../obj_toolchain/binutils/ranlib
```

### Install musl

```
cd ~/toolchain/build/obj_musl
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false AR=../obj_toolchain/binutils/ar \
  RANLIB=../obj_toolchain/binutils/ranlib \
  DESTDIR=~/toolchain/build/obj_sysroot \
  install
```

### Build GCC

```
cd ~/toolchain/build/obj_toolchain
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false MAKE="make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= \
  ac_cv_prog_lex_root=lex.yy.c MAKEINFO=false"
```

### Install kernel headers

```
mkdir -p ~/toolchain/build/obj_kernel_headers/staged
cd ~/srcs/linux-4.4.10/
make MULTILIB_OSDIRNAMES= INFO_DEPS= infodir= ac_cv_prog_lex_root=lex.yy.c \
  MAKEINFO=false ARCH=arm \
  O=~/toolchain/build/obj_kernel_headers \
  INSTALL_HDR_PATH=~/toolchain/build/obj_kernel_headers/staged \
  headers_install
```

## References

* [1] [Cross-Compiled Linux From Scratch - Embedded](http://clfs.org/view/clfs-embedded/arm/index.html)
* [2] [musl-cross-make](https://github.com/richfelker/musl-cross-make)
* [3] [mkroot](https://github.com/landley/mkroot)
* [4] [How to test GCC on a simulator](https://gcc.gnu.org/wiki/Building_Cross_Toolchains_with_gcc)
* [5] [Installing GCC](https://gcc.gnu.org/install/configure.html)
