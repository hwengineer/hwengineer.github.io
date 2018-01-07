---
layout: post
title: compiler environment
---

So there I was. Starting from scratch.

## arm-none-eabi

A good thing : arm-none-eabi the *standard* compiler toolchain is in the Ubuntu 16.04 repos.

'''bash
sudo apt-get install binutils-arm-none-eabi gcc-arm-none-eabi gdb-arm-none-eabi  libnewlib-arm-none-eabi libnewlib-arm-none-eabi
'''

## LLVM

But I want to use LLVM as a compiler. Just as a personal preference.
And here follows the first problem.
The linker of LLVM (ld.lld) for the cortex-m controllers is only above version 5.0 useful.

We have to add the ppa from apt.llvm.org to our package list.

'''bash
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
sudo add-apt-repository -y 'deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-5.0 main'
sudo add-apt-repository -y 'deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-5.0 main'
'''

after that we can update the Ubuntu repos and install LLVM / Clang
'''bash
sudo apt-get update
sudo apt-get -y clang-5.0 clang-5.0-doc libclang-common-5.0-dev libclang-5.0-dev libclang1-5.0 libclang1-5.0-dbg libllvm-5.0-ocaml-dev libllvm5.0 libllvm5.0-dbg lldb-5.0 llvm-5.0 llvm-5.0-dev llvm-5.0-doc llvm-5.0-examples llvm-5.0-runtime clang-format-5.0 python-clang-5.0 lldb-5.0-dev lld-5.0 libfuzzer-5.0-dev
'''
