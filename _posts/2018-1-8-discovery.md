---
layout: post
title: STM32F3Discovery-meson-example
---

We now look at the first example project [STM32F3Discovery-meson-example](https://github.com/hwengineer/STM32F3Discovery-meson-example).
It's for the [STM32F3Discovery](http://www.st.com/en/evaluation-tools/stm32f3discovery.html) Development Board from ST Microelectronics

![STM32F3Discovery Development Board](http://www.st.com/content/ccc/fragment/product_related/rpn_information/board_photo/8e/9b/f4/fd/3f/3b/4a/e7/stm32f3discovery.jpg/files/stm32f3discovery.jpg/_jcr_content/translations/en.stm32f3discovery.jpg)

## Example Project

```
├── STM32-ldscripts      # uC dependend linker files
|   ├──...
├── STM32-startup        # uC dependend startup files
|   ├──...
├── STM32Cube-F3-meson   # official ST HAL / CMIS Driver
|   ├──...
├── .gdbinit             # init file for gdb
├── .gitignore
├── .gitmodules
├── meson.build          # meson configuration file
├── cross_file.build     # meson cross-compilation configuration file
├── main.c
├── main.h
├── LICENSE
├── README.md
├── Toolchain.md
```

### linker, startup files

In the folders STM32-ldscripts and STM32-startup are microcontroller dependend
configuration files saved.
For a running application we need to specify the right linker and startup scripts.
I will discuss that topic later.

### STM32 HAL Driver

In this folder we find the STM32 HAL / CMSIS Drivers from ST.
Sadly ST doesn't host a github repo at the moment. So I had to copy the files to github.
The files are under the license from ST.

### meson files
To configure meson we need two files.

Normally the `meson.build` file is sufficient. But when we do a cross-compilation build
we need also a `cross-file` for defining the right compilers and special target flags.

### usage

1.  cloning repo and change to project folder

        git clone https://github.com/hwengineer/STM32F3Discovery-meson-example
        cd STM32F3Discovery-meson-example

2.  create a build directory

        mkdir llvmbuild

3.  init the build system

        meson llvmbuild --cross-file=cross_file.build

4.  change to build directory

        cd llvmbuild

5.  start compilation process (with ninja backend)

        ninja

### flash uC

1.  start openocd (for the STM32F3Discovery use this parameters)

        openocd -f interface/stlink-v2.cfg -f target/stm32f3x.cfg

2.  start a gdb session

        arm-none-eabi-gdb -q main.elf

3.  configure and load the uC with gdb

        (gdb)target remote:3333 # connect to openocd gdb server (standard port 3333)
        (gdb)load               # load binary
        (gdb)continue           # start execution


now the led should blink on the STM32F3Discovery board
