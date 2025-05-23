---
title: "CPU虚拟化"
linkTitle: "CPU"
weight: 40
date: 2025-04-30
description: >
  CPU虚拟化技术介绍
---

## 概念

### x86保护环

保护环 protection rings 是 CPU 中的一种技术,用于将 CPU 的权限级别划分为不同的环。

x86 架构使用 protection rings 的概念,内核在最高特权模式(ring 0)下运行,用于运行进程运行的用户空间在 ring 3 中。

硬件要求所有特权指令都在 ring 0 中执行。如果尝试在环 3 中运行特权指令,则 CPU 将生成错误。在基于 VM 的虚拟化的情况下,VM 在主机作系统上作为进程运行, 因此如果未处理故障,则整个 VM 可能会被终止。

在高级别上,来自 Ring 3 的特权指令执行由代码段寄存器通过 CPL (代码特权级别) 位控制。来自 ring 3 的所有呼叫都被限制到 ring 0。例如,系统调用可以通过 syscall 之类的指令(来自用户空间)进行,


### 保护环下的虚拟机

同样的概念也适用于虚拟化作系统。 

在这种情况下,guest 被取消特权并在ring 1 中运行,而 guest 的进程在 ring 3 中运行。VMM 本身在 ring 0 中运行。

对于完全虚拟化的 Guest,任何特权指令都必须被捕获和模拟。


## 问题

旧版本的 x86 CPU 不可虚拟化,这意味着并非所有敏感指令都具有特权。

SGDT、SIDT 等指令可以在 Ring 1 中执行而不会被困住。

这在运行 guest 操作系统时可能有害,因为这可能允许 guest 查看主机内核数据结构。

可以通过两种方式解决此问题:

1. 完全虚拟化情况下的二进制转换
2. 使用 hypercalls 的 XEN 半虚拟化

### 完全虚拟化情况下的二进制转换

在这种情况下,使用来 guest 操作系统时无需进行任何更改。

这些指令针对目标环境进行捕获和模拟。这会导致大量的性能开销,因为必须将大量指令捕获到主机/虚拟机管理程序中并进行模拟。

### 半虚拟化

为了避免在使用完全虚拟化时与二进制转换相关的性能问题,我们使用了半虚拟化, 其中 guest 知道它正在虚拟化环境中运行,并且它与主机的交互经过优化以避免过多的陷阱。

设备驱动程序代码已更改并分为两部分：

1. 后端： 与虚拟机管理程序在一起
2. 前端： 与 guest 在一起

guest 和 host 驱动程序现在通过环形缓冲区（ring buffers）进行通信。

2005 年,x86 终于实现了虚拟化。他们引入了另一个环,称为 Ring-1,也称为 VMX (virtual machine extensions) root mode。

VMM 在 VMX root 模式下运行, guest 在 non-root 模式下运行。

这意味着 Guest 可以在 Ring 0 中运行,并且对于大多数说明,没有陷阱。客户机所需的特权/敏感指令由 VMM 在 root 模式下通过陷阱执行。

我们将这些开关称为：

- VM Exits： VMM 接管 guest 的指令执行

- VM Entries： VM 从 VMM 获得控制权

硬件辅助虚拟化的优势是双重的:

- 无二进制转换

- 无需修改操作系统

### 遗留问题

遗留问题： VM Entry 和 Exits 仍然是涉及大量 CPU 周期的繁重调用, 因为必须保存和还原完整的 VM 状态。

为了减少这些进入和退出的周期,已经做了大量工作，如使用半虚拟化驱动程序。











