---
layout: post
title: Build Calculix 2.15 on MSYS2 on Windows
author: xpanda
img: https://on.xhost.top:97/images/2019/10/28/3cab49818cb6f58309d48ae78e5642c3.md.png
category: Infor
---

![Calculix](https://on.xhost.top:97/images/2019/10/28/3cab49818cb6f58309d48ae78e5642c3.png)

This is a help  note about building Calculix 2.15 on windows in MSYS2, if you want to build it on Linux, please refer to this [link]( https://www.libremechanics.com/?q=node/9 ).

## Install MSYS2 on Windows

Just go to the [MSYS2 offical webiste](https://www.msys2.org/) and install the windows version of MSYS2. 

After installing MSYS2, you may want to use local mirrors for package installation. Specifically for Chinese users, please refer to [this link](https://lug.ustc.edu.cn/wiki/mirrors/help/msys2) to set the mirrors.

Now we need to update some packages and install some packages for development:

``` sh
# Update everything using
pacman -Syu

# Installing gcc using MSYS2
pacman -S base-devel
pacman -S gcc
pacman -S vim
pacman -S cmake
```

If you are using EmuCon, you can integrate MSYS2 to your EmuCon console taskers by adding a task in the menu: Settings -> Tasks. Add the following settings to the task commands fields:

``` sh
set MSYS2_PATH_TYPE=inherit & set CHERE_INVOKING=1 & set MSYSTEM=MSYS & set MSYSCON=conemu64.exe & "MSYS2_INSTALL_DIR\usr\bin\bash.exe" --login -i -new_console:C:"MSYS2_INSTALL_DIR\msys2.ico"
```

## Install some packages for building Calculix

```sh
# all the packages need to compile Calculix
pacman -S mingw-w64-x86_64-toolchain
pacman -S mingw-w64-x86_64-boost
pacman -S mingw-w64-x86_64-libxml2 mingw-w64-x86_64-eigen3
pacman -S scons base-devel mingw-w64-x86_64-cmake mingw-w64-x86_64-perl mingw-w64-x86_64-make mingw-w64-x86_64-yaml-cpp
pacman -S gcc-fortran
```

## Build Spooles

- Download spooles from the [offical website](http://www.netlib.org/linalg/spooles/spooles.2.2.tgz) to your local dir, e.g. f:/soft/spooles.2.2. 
-  In file f:/soft/pooles.2.2/Tree/src/makeGlobalLib change:   **drawTree.c** to **draw.c** 
-  in file f:/soft/spooles.2.2/Make.inc change:  `CC = /usr/lang-4.0/bin/cc` to `CC = /usr/bin/cc`
-  On spooles.2.2 folder make (in the MSYS2 console): `make lib`

## Build ARPARK

- Download [ARPARK](http://www.caam.rice.edu/software/ARPACK/SRC/arpack96.tar.Z) and unpack to f:/soft/ARPARK.
- Download [ARPARK Patch](http://www.caam.rice.edu/software/ARPACK/SRC/patch.tar.gz) and unpack to f:/soft/ARPARK to overwrite the folder.
- In file f:/soft/ARPARK/ARmake.inc:
- change: `home = $(HOME)/ARPACK` to `home = /f/soft/ARPACK`
- change:  `PLAT = SUN4` to `PLAT = linux`
- change:  `FC = f77` to `FC = gfortran`
- change:  `FFLAGS = -O -cg89` to `FFLAGS = -O2`
- change:  `MAKE = /bin/make` to `MAKE = /usr/bin/make`
- In File f:/soft//ARPACK/UTIL/second.f 
- change: `EXTERNAL ETIME` to `* EXTERNAL ETIME` (Comment the line)
- On ARPAK folder make: `make lib`

## Build Calculix

- Download Calculix source code to f:/soft/ccx.2.15
- In file f:/soft/ccx.2.15/src/Makefile
- Change the dir of spooles and ARPACK accordingly
- Then build: `make`

