---
title: X86模拟ARM环境
date: 2024-12-15 19:19:19
permalink: /pages/d0f2dc/
sidebar: auto
categories:
  - 随笔
tags:
  - ARM64
  - 虚拟机
  - Kylin
  - Linux
  - QEMU
author: 
  name: Kevin Zhang
  link: https://github.com/gyzhang
---
到 [https://qemu.weilnetz.de/w64/](https://qemu.weilnetz.de/w64/) 下载最新 Windows 版本的 qemu；

到 [http://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/QEMU_EFI.fd](http://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/QEMU_EFI.fd) 下载启动镜像；

到 [https://build.openvpn.net/downloads/releases/tap-windows-9.24.7-I601-Win10.exe](https://build.openvpn.net/downloads/releases/tap-windows-9.24.7-I601-Win10.exe) 下载虚拟网卡；

到 [https://sx.ygwid.cn:4431/](https://sx.ygwid.cn:4431/) 下载对应的 Kylin 操作系统。

安装完虚拟网卡 `tap-windows-9.24.7-I601-Win10.exe` 后，将网卡命名为 `TAP9247`（后面的脚本要用到，对应上即可）。

本机可上网的网卡（无线/有线均可），共享 `TAP9247` 连接上网（网络地址会固定为 `192.168.137.1`）。

创建虚拟磁盘：

```bash
qemu-img create -f qcow2 KylinV10SP1WithoutGUI-ARM64.qcow2 256G
```

安装系统：

```bash
qemu-system-aarch64 -m 8096 -cpu cortex-a76 -smp 8,cores=4,threads=2,sockets=1 -M virt -bios QEMU_EFI.fd -drive if=none,file=D:\Kevin\MySoftware\Linux\Kylin-Server-10-SP1-Release-Build04-20200711-arm64.iso,id=cdrom,media=cdrom -device virtio-scsi-device -device scsi-cd,drive=cdrom -drive if=none,file=KylinV10SP1WithoutGUI-ARM64.qcow2,id=hd0 -device virtio-blk-device,drive=hd0 -device VGA -device nec-usb-xhci -device usb-mouse -device usb-kbd -net nic -net tap,ifname=TAP9247
```

> 系统内设置 IP 为 192.168.137.101，网关和 DNS 都设置为 192.168.137.1，这样虚拟机 Linux 可访问互联网。

启动操作系统：

```bash
echo password: good@Man#123
echo ip: 192.168.137.101
qemu-system-aarch64 -m 8096 -cpu cortex-a76 -smp 8,cores=4,threads=2,sockets=1 -M virt -bios QEMU_EFI.fd -drive if=none,file=KylinV10SP1WithoutGUI-ARM64.qcow2,id=hd0 -device virtio-blk-device,drive=hd0 -device VGA -device qemu-xhci -device usb-mouse -device usb-kbd -net nic -net tap,ifname=TAP9247
```

