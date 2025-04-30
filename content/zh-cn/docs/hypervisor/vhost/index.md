---
title: "Vhost 通讯"
linkTitle: "Vhost"
weight: 50
date: 2025-04-30
description: >
  基于 Vhost 的数据通信 
---

vhost 是 KVM 内核虚拟机管理程序的虚拟设备管理程序, 它使用内核模块来管理虚拟设备。

### 网络数据包处理流程

当我们使用 vhost 机制时, QEMU 不在数据平面之外,客户机和主机之间通过 virtqueues 进行直接通信。

![network-package-flow](./images/network-package-flow.png)