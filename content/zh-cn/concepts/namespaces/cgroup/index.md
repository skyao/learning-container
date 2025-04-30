---
title: "Cgroup namespace"
linkTitle: "Cgroup"
weight: 60 
date: 2025-04-30
description: >
  将 cgroup 文件系统的可见性限制为进程所属的 cgroup
---

cgroup namespace 将 cgroup 文件系统的可见性限制为进程所属的 cgroup。

如果没有此限制, 进程可以通过 `/proc/self/cgroup` 层次结构查看全局 cgroup。

此命名空间有效地虚拟化了 cgroup 本身。

