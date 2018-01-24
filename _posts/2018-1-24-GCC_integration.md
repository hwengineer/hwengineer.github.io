---
layout: post
title: GCC integration
---

Due to some request I added gcc support  
It was nice to see that only minor changes had to be made.

I added a new cross-file : `cross_gcc.build`  
Firstly you have to change all compiler commands to the gcc related binaries.

`cross_gcc.build`
```
[binaries]
c       = 'arm-none-eabi-gcc'
cpp     = 'arm-none-eabi-g++'
ld      = 'arm-none-eabi-ld'
ar      = 'arm-none-eabi-ar'
as      = 'arm-none-eabi-as'
size    = 'arm-none-eabi-size'
objdump = 'arm-none-eabi-objdump'
objcopy = 'arm-none-eabi-objcopy'
strip   = 'arm-none-eabi-strip'
gdb     = 'arm-none-eabi-gdb'
```

And I had to strip some llvm specific compile and link flags.

```
[properties]
c_args      = [
               #'--target=arm-none-eabi',   # llvm specific
               '-mthumb',                   # define language
               #------------------------------------
               '-fshort-enums',             # otherwise errors at linking...
               '-fmessage-length=0',        # all error warnings in a single line (default 72)
               '-fsigned-char',             # char is per default unsigned
               '-ffunction-sections',       # each function to a seperate section ==> Code-optimization / deletion
               '-fdata-sections',           # each variable to a seperate section ==> Code-optimization / deletion

               #'-Weverything',             # llvm specific
               '-Wall',
               '-ffreestanding',
               ]

c_link_args = [
               #'--target=arm-none-eabi', # llvm specific
               '-nostdlib',               # do not import the standard library's
              ]
```

done


## examples

I added and test the files on my examples repo.

```
git clone https://github.com/hwengineer/STM32F3Discovery-meson-example.git
cd STM32F3Discovery-meson-example
mkdir gccbuild
meson gccbuild --cross-file=cross_gcc.build
cd gccbuild
ninja
```

```
git clone https://github.com/hwengineer/STM32-L0-qemu-example.git
cd STM32-L0-qemu-example
mkdir gccbuild
meson gccbuild --cross-file=cross_gcc.build
cd gccbuild
ninja
```
