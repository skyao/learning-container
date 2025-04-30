---
title: "cgroups 概述"
linkTitle: "概述"
weight: 1 
date: 2025-04-30
description: >
  cgroups 用于限制进程的资源使用
---

## 背景

需要一种方法来为命名空间中的进程引入资源控制。Linux 使用一种称为 control groups 的机制来实现,通常称为 cgroups。

cgroups 基于 cgroup 控制器的概念工作，由 Linux 内核中名为 cgroupfs 的文件系统表示。

当前使用的 cgroup 版本为 cgroup v2。

> cgroup 版本 v1 和 v2 之间的区别在于: 
>
> 在 v1 中挂载时,我们可以指定挂载选项来指定要启用的控制器,而在 cgroup v2 中,不能传递此类挂载选项。

## 操作示例

```bash
mkdir ~/mygroup
sudo mount -t cgroup2 none ~/mygroup
cd ~/mygroup
ls
```

能看到如下的文件，这些是挂载的 cgroup v2 的文件系统：

```bash
0 -r--r--r--  1 root root 0 Apr 30 17:53 cgroup.controllers
0 -rw-r--r--  1 root root 0 Apr 30 17:55 cgroup.max.depth
0 -rw-r--r--  1 root root 0 Apr 30 17:55 cgroup.max.descendants
0 -rw-r--r--  1 root root 0 Apr 30 17:55 cgroup.pressure
0 -rw-r--r--  1 root root 0 Apr 30 17:53 cgroup.procs
0 -r--r--r--  1 root root 0 Apr 30 17:55 cgroup.stat
0 -rw-r--r--  1 root root 0 Apr 30 17:53 cgroup.subtree_control
0 -rw-r--r--  1 root root 0 Apr 30 17:55 cgroup.threads
0 -rw-r--r--  1 root root 0 Apr 30 17:55 cpu.pressure
0 -r--r--r--  1 root root 0 Apr 30 17:55 cpuset.cpus.effective
0 -r--r--r--  1 root root 0 Apr 30 17:55 cpuset.mems.effective
0 -r--r--r--  1 root root 0 Apr 30 17:55 cpu.stat
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 dev-hugepages.mount
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 dev-mqueue.mount
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 init.scope
0 -rw-r--r--  1 root root 0 Apr 30 17:55 io.cost.model
0 -rw-r--r--  1 root root 0 Apr 30 17:55 io.cost.qos
0 -rw-r--r--  1 root root 0 Apr 30 17:55 io.pressure
0 -r--r--r--  1 root root 0 Apr 30 17:55 io.stat
0 -r--r--r--  1 root root 0 Apr 30 17:55 memory.numa_stat
0 -rw-r--r--  1 root root 0 Apr 30 17:55 memory.pressure
0 --w-------  1 root root 0 Apr 30 17:55 memory.reclaim
0 -r--r--r--  1 root root 0 Apr 30 17:55 memory.stat
0 -r--r--r--  1 root root 0 Apr 30 17:55 misc.capacity
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 proc-sys-fs-binfmt_misc.mount
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 sys-fs-fuse-connections.mount
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 sys-kernel-config.mount
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 sys-kernel-debug.mount
0 drwxr-xr-x  2 root root 0 Apr 30 17:53 sys-kernel-tracing.mount
0 drwxr-xr-x 17 root root 0 Apr 30 17:53 system.slice
0 drwxr-xr-x  3 root root 0 Apr 30 17:53 user.slice
```

## Cgroup 类型

根据我们想要控制的资源,有不同类型的 cgroup。我们将介绍的两个 cgroup 如下:

- CPU: 为用户空间进程提供 CPU 限制

- Block I/O： 为用户空间进程提供块设备的 I/O 限制


















