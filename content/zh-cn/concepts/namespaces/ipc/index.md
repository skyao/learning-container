---
title: "IPC namespace"
linkTitle: "IPC"
weight: 50 
date: 2025-04-30
description: >
  限定 IPC 结构体的范围,如 POSIX 消息队列
---

IPC  namespace 限定 IPC 结构体的范围,如 POSIX 消息队列。 

在同一命名空间内的两个进程之间, 将启用 IPC, 但如果两个不同命名空间中的两个进程尝试通过 IPC 进行通信,则 IPC 将受到限制。
