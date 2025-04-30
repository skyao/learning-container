---
title: "PID namespace"
linkTitle: "PID"
weight: 20 
date: 2025-04-30
description: >
  PID 命名空间中的进程具有不同的进程树
---

PID 命名空间中的进程具有不同的进程树。 

他们有一个 PID 为 1 的 init 进程。

但是,在数据结构级别,进程属于一个全局进程树,该树仅在主机级别可见。

ps 之类的工具或从命名空间中直接使用 /proc 文件系统将列出命名空间中进程树的进程及其相关资源。

