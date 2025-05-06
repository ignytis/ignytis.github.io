---
title: "QEMU + Ubuntu Server + GDB. Debugging the linux server using a virtual machine"
date: 2025-05-05T22:11:23
description: This guide demonstrates how to compile and debug the Linux Kernel using QEMU
tags: ["debugging", "reverse-engineering", "linux", "kernel", "virtualization", "qemu"]
draft: false
---

# Considerations

- Ubuntu Server 24.04.2 (x64) is used as a guest system

# Compilation of Linux Kernel

## How to obtain the code

The code can be pulled from [Github mirror](https://github.com/torvalds/linux)
```bash
git clone git@github.com:torvalds/linux.git
```
...or from [kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/), if you prefer the original source:
```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

It will take a while, because Kernel has quite a huge history. This would be the __Kernel source directory__.

## Prerequisites

You will need:
- C compiler (`gcc`), of course
- Development version of `ncurses` library to render the configuration menu when running `make menuconfig`.
  In Fedora in can be installed via `dnf install ncurses-devel`
- GNU `make` utility

And that's most likely all. The benefit of Linux kernel development is not being dependent on any other system libraries.
On the opposite, other libraries might depend on Linux Kernel :)

## Configuration

We will need to compile a kernel which is compatible with QEMU and which has debug information for GDB.
Navigate to __Kernel source directory__ and run:
```bash
make x86_64_defconfig
make kvm_guest.config
```
These commands will create a basic `.config` file which is configured for x64 system
and which has components for KVM virtualization that we will use in QEMU.
We only need to enable the debug information in the kernel:

```bash
make menuconfig
```

This will display a TUI menu where you need to navigate to `Kernel hacking` (it must be the last item).
Verify if `Kernel debugging` is enabled. Then go to `Compile-time checks and compiler options` &gt; 
`Debug information (Generate DWARF Version 5 debuginfo)` sub-level and set the `Generate DWARF Version 5 debuginfo` option.
Now you can save the config and exit from configuration utility.

### Setting the kernel version (optional)

You would like to be sure that you are running YOUR kernel, right?
To set a custom version name, you can edit the `Makefile` and assign some suffix to `EXTRAVERSION` parameter in beginning of the file.
The initial value is something like `-rc4`, so you can make it `-john-doe-edition`.
It will be observable via `uname -a` in the future.


## Building the kernel

```bash
make -j$(nproc)
```

The `-j$(nproc)` argument will resolve to `-j<number_of_your_cpu_cores>` which will significantly improve the time of compilation.
Compilation might be time consuming, especially if your CPU doesn't have manu cores.
It might take even hours if you skip the CPU number parameter. But in my case (16 cores) it took probably 10 minutes.


When the compilation completes, there should be output like
```
Kernel: arch/x86/boot/bzImage is ready
```

This is your kernel. To avoid any confusion - there is also `arch/x86/boot/bzImage` file, but it is just a symlink to `arch/x86/boot/bzImage`.


# Making the initial RAM disk

Linux kernel needs an initial filesystem to bootstrap, and we have to build it or to borrow it from somewhere else.
Let's try to build it.

## Compilation of Busybox

# Setting up the virtual machine

# Debugging the kernel

# Kudos
- https://www.youtube.com/@johannes4gnu_linux96 for this [video about compilation of Kernel, Busybox and building the rootfs](https://www.youtube.com/watch?v=LyWlpuntdU4)
- https://www.youtube.com/@hexdump1337 for [one more video about kernel, busybox and qemu](https://www.youtube.com/watch?v=4PdMZd0Bt7c)
- https://www.youtube.com/@ghostinthehive2027 for [video about debugging the kernel in GDB](https://www.youtube.com/watch?v=2VcA5Wj7IvU)
- [ChatGPT](https://chatgpt.com/) for solving my networking issues inside qemu VM