---
title: "使用 KVM"
linkTitle: "KVM"
weight: 40
date: 2025-04-30
description: >
  使用 KVM 模块创建 VM
---

KVM 是内核虚拟机管理程序, 它使用内核模块来管理虚拟机。

### 创建 VM

要创建 VM,必须对内核 KVM 模块进行一组 ioctl 调用,这将向客户机公开 /dev/kvm 设备。简单来说,这些是来自用户空间的创建和启动 VM 的调用:

1. KVM CREATE VM: 此命令将创建一个没有虚拟 CPU 和内存的新 VM。
2. KVM SET USER MEMORY REGION: 此命令映射 VM 的用户空间内存。
3. KVM CREATE IRQCHIP / KVM CREATE VCPU:此命令创建一个硬件组件(如虚拟 CPU),并将它们与 vt-x 功能进行映射。
4. KVM SET REGS / SREGS / KVM SET FPU / KVM SET CPUID / KVM SET MSRS / KVM SET VCPU EVENTS / KVM SET LAPIC:这些命令是硬件配置。
5. KVM RUN: 此命令启动 VM。

