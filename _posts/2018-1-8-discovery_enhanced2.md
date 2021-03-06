---
layout: post
title: STM32F3Discovery-meson-example (meson)
---

We have to get deeper into meson!

We start with an examination of the meson `cross-file`

#### cross-file

We have to tell meson which compilers it should use: In our case the llvm-compiler

```
[binaries]
c       = 'clang-5.0'
cpp     = 'clang++-5.0'
ld      = 'llvm-link-5.0'
ar      = 'llvm-ar-5.0'
as      = 'llvm-as-5.0'
size    = 'llvm-size-5.0'
objdump = 'llvm-objdump-5.0'
objcopy = 'arm-none-eabi-objcopy'
```

after that we have to define some compiler flags.
These are copied directly at the compilation to the specific compilers / linkers

```
[properties]
c_args      = [
               '--target=arm-none-eabi',    # define target for clang
               '-mthumb',                   # define language
               #------------------------------------
               '-fshort-enums',             # otherwise errors at linking...
               '-fmessage-length=0',        # all error warnings in a single line (default 72)
               '-fsigned-char',             # char is per default unsigned
               '-ffunction-sections',       # each function to a seperate section ==> Code-optimization / deletion
               '-fdata-sections',           # each variable to a seperate section ==> Code-optimization / deletion

               '-Weverything',
               '-ffreestanding',
               '-v', # option for more verbose output
               ]

c_link_args = [
              '--target=arm-none-eabi', # define target for linker
              '-v',                     # option for more verbose output
               '-nostdlib',             # do not import the standard library's
]
```

these are the bare minimum what we have to define for a compilation with llvm.

*okay i lied*...

you need also the `-mcpu` flag, the linker and startup files and also link the `thumbv6` or `thumbv7` runtimes.
But this will follow shortly

**back** to the cross-file.

```
[host_machine]
system     = 'none'
cpu_family = 'arm'
cpu        = 'cortex-m4'
endian     = 'little'
```

At the end we have to define our `host_machine`. Meson define the `host_machine` as the machine that runs the compilated code. And in most cases this is equal to the `target_machine`.

#### meson.build
The `meson.build` file defines our executable and I also added some nice features for developing with STM32 Devices.

First we define the project name and default options.

```
project('blink', 'c',
          default_options : ['b_lto=false',
                             'b_asneeded=false'])
```

And after that we define some variables.

```
# uController / HAL Driver dependend options
c_args     += '-DSTM32F303xC'    # HAL Driver define
linkfiles   = files(['STM32-ldscripts/STM32F3/STM32F303VC.ld', 'STM32-ldscripts/simple.ld'])
startupfile = files(['STM32-startup/STM32F3/stm32f303xc.s'])
```
We have to add define the `STM32F303xC` flag for the compilation with the STM32Cube library.
this is equivalent to

```
#define(STM32F303xC)
```

in a C file.

Also we have to add the linkfiles and startup files to a variable. When we define paths with the `files` function, meson will save the absolute file path.
So we don't have to mess around with filenames when we concatenate `meson.build` files to create more complex projects.

#### create linker flags

Now we can use a string replace function and a `foreach` loop to create our linker-flags with an absolute path to our linker files

```
foreach linkfile : linkfiles
 link_args += ['-Wl,-T,@0@/@1@'.format(meson.current_source_dir(), linkfile)]
endforeach
```

#### a convenience function

Now follows a convenience function:

```
cpu = host_machine.cpu() == 'cortex-m0+' ? 'cortex-m0plus' : host_machine.cpu()
c_args += '-mcpu=@0@'.format( cpu )
```

we take the already define variable from the `host_machine` definition and define our compiler flag `-mcpu`.
Sadly the `Cortex-M0+` has another syntax. It can't use the plus sign. So we have to make this `if else`

#### link runtime  and header
It follows another a convenience function

```
arch       = (host_machine.cpu() == 'cortex-m0') or (host_machine.cpu() == 'cortex-m0+') or (host_machine.cpu() == 'cortex-m1') ? 'armv6-m'  : ''
arch      += (host_machine.cpu() == 'cortex-m3') ?                                                                                'armv7-m'  : ''
arch      += (host_machine.cpu() == 'cortex-m4') or (host_machine.cpu() == 'cortex-m7') ?                                         'armv7e-m' : ''

link_deps +=  meson.get_compiler('c').find_library('/usr/lib/gcc/arm-none-eabi/4.9.3/@0@/libgcc'.format(arch))
```

dependent on the cpu architecture we have to load different runtimes from the arm-none-eabi-gcc toolchain.
Here we just mangling around path names.

And for the llvm-toolchain only I had to link manually the right header files to our project.

```
if meson.get_compiler('c').get_id() == 'clang'
  incdirs += include_directories('/usr/lib/arm-none-eabi/include/') # bare-metal : std header includes
endif
```

Also the `include_directories` function will mangling around the path to an absolute path for easier usage.

**Why did I had to do this?**
I didn't compile the so called llvm runtime `compiler-rt` for our Cortex-M targets. This would have created the runtime from the standard llvm toolchain.
Something I want to do in the future.


#### include STM32Cube library

In the folder `STM32Cube-F3-meson` there is also a `meson.build` file.

```
# add STM library
subdir('STM32Cube-F3-meson')
```
with this command we tell meson to look in that directory for another `meson.build` file and execute it.
All variables will be present in the *root meson.build* file

#### define executable

```
main = executable(
            'main.elf',
            [srcs, stm32cube_srcs, 'main.c', startupfile] ,
            c_args              : [c_args ],
            link_args           : [link_args, '-Wl,--gc-sections'],
            dependencies        : link_deps,
            include_directories : [incdirs, stm32cube_incdirs] )
```

finally we can define our executable.

It's mostly self explanatory. We create an output file with the name `main.elf`.
We include all defined source files, the STM32Cube sources, the `main.c` and startupfiles.

We use the compiler with the flags defined in `c_args` and afterwards link it with the flags from `link_args`.
I added also the `-Wl,--gc-sections` flag. This tells the compiler to use `garbage collection` and strips out unused code.

Then we add the runtime dependencies saved in the `link_deps` variable. And last but not leas we tell the compiler where to look for header files.


#### custom targets

At the end of the file I added some custom targets.
These command can be called by meson directly or are just called in every `ninja` run if the `build_by_default` flag is set to true

```
...
maindump = custom_target(
                        'dump',
      capture          : true,
      output           : ['main.dump'],
      command          : [objdump, '-triple=thumbv6m-none-eabi', '-disassemble-all', '-S', '-t', 'main.elf', '>', 'main.dump'],
      depends          : [main])
...
```

I used this custom target make copies of the executable to different file formats and dump files.
