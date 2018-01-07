---
layout: post
title: compiler environment
---

So there I was. Starting from scratch.

## arm-none-eabi

A good thing : arm-none-eabi the *standard* compiler toolchain is in the Ubuntu 16.04 repos.
(we will see we will need some headers from this toolchain)

```bash
sudo apt-get install binutils-arm-none-eabi gcc-arm-none-eabi gdb-arm-none-eabi  libnewlib-arm-none-eabi libnewlib-arm-none-eabi
```

## LLVM

But I want to use LLVM as a compiler. Just as a personal preference.
And here follows the first problem.
The linker of LLVM (ld.lld) for the cortex-m controllers is only above version 5.0 useful.

We have to add the apt-packages from apt.llvm.org to our package list.

```bash
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
sudo add-apt-repository -y 'deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-5.0 main'
sudo add-apt-repository -y 'deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-5.0 main'
```

after that we can update the Ubuntu repos and install LLVM / Clang

```bash
sudo apt-get update
sudo apt-get -y clang-5.0 clang-5.0-doc libclang-common-5.0-dev libclang-5.0-dev libclang1-5.0 libclang1-5.0-dbg libllvm-5.0-ocaml-dev libllvm5.0 libllvm5.0-dbg lldb-5.0 llvm-5.0 llvm-5.0-dev llvm-5.0-doc llvm-5.0-examples llvm-5.0-runtime clang-format-5.0 python-clang-5.0 lldb-5.0-dev lld-5.0 libfuzzer-5.0-dev
```

## Meson Build

After that we of course need the [meson build](http://mesonbuild.com/) system.
Also Python3 is needed.

```bash
sudo apt-get -y install python3 python3-pip python3-setuptools ninja-build
sudo pip3 install --upgrade pip
sudo pip3 install --user meson
```

##  openocd
We will later want to program and debug the microcontroller. So we need openocd

```bash
sudo apt-get install openocd
```

You maybe have to add the so called udev rules. (They should get installed with the openocd package). After updating the udev rules use this command to reload the new configs

```bash
sudo udevadm control --reload-rules
```

### never openocd version

For the STM32L0-Series we need a never version of openocd.
At the moment Version 0.10.0 is the latest version. So my install script here will install version 0.10.0

```bash
cd /tmp/
wget https://sourceforge.net/projects/openocd/files/openocd/0.10.0/openocd-0.10.0.zip/download
unzip download -d /tmp/openocd
rm download
cd openocd/openocd-0.10.0
./configure
make
sudo make install
```

this installs version 0.10 besides your current openocd installation. you have to call `/usr/local/bin/openocd` instead of just `openocd` to use the newer version

a quick and dirty alternative is to overwrite the old openocd binary in `/usr/bin/openocd`

```bash
cp /usr/local/bin/openocd /usr/bin/openocd
```
