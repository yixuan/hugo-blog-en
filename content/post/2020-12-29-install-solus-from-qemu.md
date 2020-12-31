---
title: 'Installing Solus from QEMU'
slug: "install-solus-from-qemu"
date: 2020-12-29
categories:
- Programming
tags:
- Solus
- QEMU
- virtual machine
- operating system
---

> This article is mainly used to record my own experience on installing the Solus operating system to
a new machine, during which I encountered troubles using the regular method. It may not be very helpful
for general users, but if you had similar experience as mine, then the method introduced here is one
possible solution.

I typically install three operating systems (OS) side by side on my machine for daily use: Windows,
[Manjaro Linux](https://manjaro.org/), and [Solus](https://getsol.us/). Solus is a very young OS,
but with a strong community for development and maintenance. It has a modern design, and is well
tuned and optimized, making it very suitable for programming and development. It is especially useful
if you want to run GPU-based programs, as it has a nice integration with video card drivers.

Recently I got a new desktop equipped with the RTX 3070 video card, on which I intend to run some deep
learning code. I had a pleasant experience working with Solus on my laptop, so I also decided to install
Solus on the new machine. However, [the regular method](https://getsol.us/articles/installation/preparing-to-install/en/)
introduced in the official guide did not succeed, possibly because the video card is relatively new, and the
live CD could not even boot.

After several rounds of searching on web, I still couldn't find a solution, and my last resort was to
install the OS via a virtual machine. Here I do not mean installing Solus to a virtual machine.
Instead, what I want is to boot the live CD within a virtual machine, and then install the OS to the
real machine.

There are many choices for virtual machine software, and finally I used [QEMU](https://www.qemu.org/)
since it is very lightweight and does not require heavy configuration.
The following command immediately opens a virtual machine that runs the [Solus live CD](https://mirrors.rit.edu/solus/images/4.1/Solus-4.1-Budgie.iso):

```bash
qemu-system-x86_64 -boot d -cdrom Solus-4.1-Budgie.iso -m 4096
```

<div align="center">
    <img src="https://upload.yixuan.blog/en/2020/12/qemu.png" alt="QEMU" />
</div>

But there are two problems here. First, as explained in
[this article](https://getsol.us/articles/troubleshooting/installation-issues/en/),
the live CD will attempt to install Solus by the same method it was booted, which means that
if the live CD was not booted in [EFI mode](https://en.wikipedia.org/wiki/EFI_system_partition),
then it would not provide the EFI option for installation to the real machine. QEMU by default
does not provide the EFI boot option, so the virtual machine above was booted in legacy mode,
which can be verified by running the `ls /sys/firmware/efi` command.

Fortunately, there is a method to add EFI support to QEMU. Following the instructions
[here](https://unix.stackexchange.com/a/57221), we need to extract the file `bios.bin`
from the [OVMF rpm package](http://download.opensuse.org/repositories/home:/jejb1:/UEFI/openSUSE_Tumbleweed/x86_64/OVMF-0.1+20160502+gd0a23f9-2.13.x86_64.rpm),
and then run QEMU with the `-bios` option:

```
qemu-system-x86_64 -bios bios.bin -boot d -cdrom Solus-4.1-Budgie.iso -m 4096
```

<div align="center">
    <img src="https://upload.yixuan.blog/en/2020/12/bios-efi.png" alt="EFI" />
</div>

This time the live CD was indeed booted in the EFI mode.

<div align="center">
    <img src="https://upload.yixuan.blog/en/2020/12/qemu-efi.png" alt="QEMU with EFI" />
</div>

And the second problem is more crucial: we want to install the OS to the real hard disk,
but typically the virtual machine works on a virtual file system. To enable QEMU to
modify the physical hard disk, we need to pass in the `-hda` (or `-hdb` etc.) option.
Suppose we want to map the `/dev/sdb` physical hard disk to the first virtual hard disk, we can do

```
sudo qemu-system-x86_64 -bios bios.bin -boot d -cdrom Solus-4.1-Budgie.iso -hda /dev/sdb -m 4096
```

Note that this command must be run with `sudo`, since modifying `/dev/sdb` is a super user operation.

Finally, I was able to run Solus live CD in EFI mode and install the system to the real machine.
The remaining tasks were just routines: upgrading the whole system using `sudo eopkg upgrade` for better
hardware support, and installing the NVIDIA video card driver with `sudo eopkg install nvidia-glx-driver-current`.
After that I was able to run Solus on the new machine smoothly.
