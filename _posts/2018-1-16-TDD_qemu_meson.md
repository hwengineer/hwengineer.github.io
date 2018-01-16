---
layout: post
title: Qemu - Meson
---

Now its time to marry qemu and meson.build

### Example project

Of course I did an examplem project for this:  
[STM32-L0-qemu-example](https://github.com/hwengineer/STM32-L0-qemu-example)

I did this with the [B-L072Z-LRWAN1](http://www.st.com/en/evaluation-tools/b-l072z-lrwan1.html) development board from ST. But it will also work with the STM32F3Discovery board.

![B-L072Z-LRWAN1](http://www.st.com/content/ccc/fragment/product_related/rpn_information/board_photo/group0/53/9e/4a/b9/c2/5d/4d/00/b-l072z-lrwan1.jpg/files/b-l072z-lrwan1.jpg/_jcr_content/translations/en.b-l072z-lrwan1.jpg)


### folder structure

```
├── STM32-ldscripts      # uC dependend linker files
|   ├──...
├── STM32-startup        # uC dependend startup files
|   ├──...
├── STM32Cube-F0-meson   # official ST HAL / CMIS Driver
|   ├──...
├── tests                # qemu test files
|   ├──...
├── lib                  # arm_semihosting files
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
```
we now have a lib folder with the *arm_semihosting* files inside. there is also a `meson.build` file inside which get called by the root `meson.build` file to import the arm_semihosting sources.  
Also we have a `tests` folder which contains the qemu test files. And also e `meson.build` file

we can configure meson to call the tests automatically.

### ninja test

firstly we have to add a new line to the `cross_file.build` file

`cross_file.build`
```
exe_wrapper = 'qemu-system-arm'
```

but that will fail. I will explain that later.

after that we can define test executables in the `meson.build` file

`tests/meson.build`
```
#==============================================================================#
# define tests : minimal test that has to be succesfull
test1 = executable(
          'test1.elf',
          [qemu_startupfile, 'helloWorld.c',  stm32cube_srcs, srcs] ,
          c_args              : [c_args],
          link_args           : [qemu_linkargs, '-Wl,--gc-sections'],
          dependencies        : link_deps,
          include_directories : [incdirs, stm32cube_incdirs])

test(  'helloWorld',
       test1,
       args : qemu_args,
       timeout: 2)
```

The executable is defined the same as the `main.elf` executable.  
We have to add a different startup file, because the qemu machine defines another memory-layout as the STM chips.  
thats why we have to define a specific `qemu_startupfile` and `qemu_linksargs`.

For more information look at this link [tests/meson.build](https://github.com/hwengineer/STM32-L0-qemu-example/blob/master/tests/meson.build)

And more interestingly we take a look to the `test` function:  
We have to define a name, an executable and the arguments which are getting called with the `exe_wrapper` defined application.

then we can just type `ninja tests` in the build directory and meson-build will compile the *normal* executable and all test executables.  
After that it will run each test and shows a short summary (how much passed, how much failed)

### problems

This example will not work...  
Because meson will first call the `exe_wrapper` concat the executable file name and after that the `qemu_args`.

But we need to define the `-kernel` flag before the executable file name. Otherwise it will not work!

Thats why we need a bash-script, which calls the `-kernel` parameter before the exe-filename.

`meson_exe_wrapper.sh`
```bash
#!/bin/sh
if [ $# = 1 ]; then
  qemu-system-arm -version #at meson startup it makes a testrun with exact 1 parameter
  exit $?
else
  qemu-system-arm -kernel "$@"
  exit $?
fi
```

this bash script need to be stored in a class path where meson-build can find it and call it.  
A meson specialty : At meson build creation it will check all tools if there are found. It calls every tool and if this call failes, the tool is not found and probably force a crash.
When we call `qemu-system-arm` call the wrong way it will return a false and lead meson to fail also...

thats why we need ther first `qemu-system-arm -version` line.

and the second call `qemu-system-arm -kernel "$@"` does what we intended to do.

Thats why we actualy do not call `qemu-system-arm` directly in the `cross_file.build` file.
But instead call the bash script

`cross_file.build`
```
exe_wrapper = 'meson_exe_wrapper.sh'
```

### integration

We can now integrate the tests folder in the main build project.

We have to write a [`tests/meson.build`](https://github.com/hwengineer/STM32-L0-qemu-example/blob/master/tests/meson.build) file inside the tests folder and call it in the *root meson.build* file:

`meson.build`
```
# add tests
subdir('tests')
```

with `ninja tests` we now can use meson to build our tests, run them and creates a summary to the results.
