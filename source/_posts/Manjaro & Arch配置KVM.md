---
title: Manjaro & Arch 配置 KVM
tags: Virtalization
categories: Linux
cover: /img/cover/8fa8ffbb.webp
abbrlink: 8fa8ffbb
date: 2021-10-24 16:43:00
description:  Linux GUI KVM 虚拟化基础配置
sticky: 
comments:
katex: 
aplayer: 
---
![封面图](/img/cover/8fa8ffbb.webp)

## 什么是 KVM

KVM (Kernel-based Virtual Machine)  由 Quramnet 开发，此公司于 2008 年被 Red Hat 收购；自 Linux 2.6.20 后整合到内核，这个模块使得 Linux 变成了一个 Hypervisor 层；运行虚拟机仅需要加载相应的 KVM 模块，依托于 PU 虚拟化指令集，性能、安全性、兼容性、稳定性表现良好，每一个虚拟化操作系统表现为单个系统进程，于 Linux 安全模块 SELinux 安全模块很好结合。
在 KVM 中，可以运行各种 GUN/Linux,  Windows 或者其他系统镜像（例如： FreeBSD ）；每一个虚拟机都可以提供独享的虚拟硬件，包括网卡、硬盘等；虚拟机也可以直通主机设备硬件（例如： GPU PCI pass through ）；但是 KVM 需要硬件支持虚拟化技术，需要使用 Intel 处理器 ( 含 VT 虚拟化技术 ) 或 AMD 处理器 ( 含 SVM 安全虚拟机技术的 AMD 处理器 ,  也叫 AMD-V ).

## 检查 KVM 支持
### 硬件检查（主要）
KVM 需要虚拟机宿主 (host) 的处理器带有虚拟化支持（对于 Intel 处理器来说是VT-x，对于AMD处理器来说是AMD-V），你可以通过以下命令来检查你的处理器是否支持虚拟化： 
```shell
$ LC_ALL=C lscpu | grep Virtualization
```
例如在此处我的电脑输出为：
```shell
Virtualization:                  VT-x
```
如果运行后没有输出，那么你的处理器可能不支持硬件虚拟化，不能使用 KVM。
**注意：** 部分品牌机，会默认关闭虚拟化支持，请前往 BIOS 检查启用

### 内核检查
通常情况下，相对较新的 Linux 内核都提供了相应的内核模块来支持 KVM.

- 你可以通过以下命令来检查内核是否包含了支持虚拟化的模块：
```
$ zgrep CONFIG_KVM /proc/config.gz
```

## 安装
此处需要的软件包：
- qemu （向虚拟主机提供模拟硬件）
- libvirt （提供管理虚拟机和其他虚拟化功能的工具和 API ）
- ovmf （为虚拟机启用 UEFI 支持）
- virt-manager （管理虚拟机的 GUI ）
**注：** 实际情况下，安装完 qemu 就可以使用虚拟机，但是 qemu-kvm 的接口相对复杂， libvirt 和 virt-manager 可以让配置和管理虚拟机更加便捷简单。

以下是用于网络连接的软件包：
- iptables-nft （和 dnsmasq 一起用于 default 的 NAT/DHCP 网络）
- dnsmasq 
- bridge-utils （用于桥接网络）
- openbsd-netcat （通过 SSH 远程管理）

```
$ pacman -S packages
```
## 将用户加入 KVM 与 libvirt 组
```
$ usermod -a -G group username
```
**注：** root用户此处跳过

## 启动 libvirtd 服务并设置开机启动
```shell
$ systemctl start libvirted
$ systemctl enable libvirted
```
现在，主机上已经配置完成 KVM 环境了，可以使用 QEMU/virt-manager 来安装与管理虚拟机了。

![Virtual Machine Manager](/img/posts/manjaro-and-arch-configuration-kvm.webp)

## 我遇到的问题

> （此部分可能会更新）

### default 网络不活动
如果出现`Requested operation is not valid: network 'default' is not active`的问题。
检查虚拟网络状态：
```shell
sudo virsh net-list -all
```
若输出状态为inactive，autostart显示为no，使用一下命令：
```shell
sudo virsh net-start default
sudo virsh net-autostart default
```

### 性能监控工具 （Performance Monitoring Tools）
Virtual Machine Manager 默认情况下只会显示 `CPU usage` ，当点击View想启用其他参数显示时，均为灰色不可操作选项，提示信息为： `Disable in perferences dialog.` 参考RedHat官方文档：[Monitoring Performance in Virtual Machine Manager](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_tuning_and_optimization_guide/sect-virtualization_tuning_optimization_guide-monitoring_in_virt_manager). 
步骤为：

1. 点击 **Edit** 菜单，选择 **Perferences**.
2. 在 **Perferences** 窗口中，选择 **Polling** 页面下，修改更新时间与stats polling options.
![Performance Monitoring Tools](/img/posts/manjaro-and-arch-configuration-kvm-Performance-Monitoring-Tools.webp)

## 结语

我使用 Linux 桌面系统已经快两年，在 Linux 上对虚拟机的需求在此前一直都是延续了在 Windows 上的习惯，直接使用 VMware Workstaion，为什么在 Linux 抛弃了使用了半年多的 VMware Workstation.
在这里不可绕开的一点是 Workstation 是商业私有软件，而 KVM 是开源软件，除此之外对于我来说还有：

- 在性能上 KVM 由于内核集成接近于原生速度
- 作为 Linux 内核的一部分

