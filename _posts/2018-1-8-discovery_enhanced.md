---
layout: post
title: STM32F3Discovery-meson-example (enhanced commands)
---

With meson it is easy to extend some useful bash command

usually I forgot the commands to type in the terminal for debugging in about a few days...  
Thats why I decided to use a nice feature from meson.

## run_target
On the last few lines of the `meson.build` file I defined some commands:

`meson.build`
```
run_target('gdb',
         command : [terminal, '--working-directory=@0@'.format(meson.source_root()), '-e', gdb, '-q', '@0@/main.elf'.format(meson.current_build_dir())])

run_target('openocd',
         command : [terminal, '-e', openocd, '-f', 'interface/stlink-v2.cfg', '-f' , 'target/stm32f3x.cfg'])
```

I can now run in the build folder the following command : `ninja openocd` that starts an `openocd` session.  
And with `ninja gdb` I start a new `gdb` session.

Both commands get executed in a new terminal and `gdb` loads the `.gdbinit` file automatically.

`.gdbinit`
```
target remote :3333

layout src
load
```

I just have to type in `(gdb)continue` in the `gdb` shell to start my program.

## gdb auto-load save path

for security reasons you maybe have to enable your working directory to load a `.gdbinit` file.  
the brute force variant is to enable all paths.

`~/.gdbinit`
```
set auto-load safe-path /
```
