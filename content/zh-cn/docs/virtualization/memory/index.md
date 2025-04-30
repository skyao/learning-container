---
title: "内存虚拟化"
linkTitle: "内存"
weight: 30
date: 2025-04-30
description: >
  内存虚拟化技术介绍
---

## 概述

虚拟化的关键挑战之一是如何虚拟化内存。

Guest 操作系统应具有与非虚拟化作系统相同的行为。 这意味着至少应该使 guest 操作系统感觉它控制着内存。

Guest 不应直接访问物理内存,但应由 VMM 拦截和处理。

在运行 guest 操作系统 时,涉及三个内存抽象:

1. Guest 虚拟内存（Guest Virtual memory）:这是在 guest 操作系统上运行的进程所看到的内容。

2. 客户机物理内存（Guest Physical memory）:这是 guest 操作系统看到的内存。

3. 系统物理内存（System Physical memory）:这是 VMM 看到的

有两种可能的方法可以处理此问题:

- 影子页表（Shadow Page Table）

- 具有硬件支持的嵌套页表（Nested page tables with hardware support）

## 影子页表

影子页表是 VMM 中的一种技术,用于将 guest 虚拟内存映射到物理内存。

对于影子页表, guest 虚拟内存通过 VMM 直接映射到系统物理内存。 这通过避免一个额外的转换层来提高性能。

## 具有硬件支持的嵌套页表

具有硬件支持的嵌套页表是 VMM 中的一种技术,用于将 guest 虚拟内存映射到物理内存。

Intel 和 AMD 通过硬件扩展为这个问题提供了解决方案。

- Intel 提供了一种称为 Extended Page Table (EPT) 的东西,它允许 MMU 遍历两个页表。



