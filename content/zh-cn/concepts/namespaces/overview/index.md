---
title: "namespaces 概述"
linkTitle: "概述"
weight: 1 
date: 2025-04-30
description: >
  namespace 是 Linux 内核中的逻辑隔离
---

命名空间用于隔离全局系统资源，使得一组进程拥有独立的资源视图。

通过命名空间，进程可以拥有独立的进程ID、主机名、用户ID、网络接口、文件系统挂载点等。

## namespaces

namespace 是 Linux 内核中的逻辑隔离。namespace 控制内核中的可见性。

所有控制都在流程级别定义。这意味着命名空间控制进程可以看到内核中的哪些资源。将 Linux 内核视为一个守卫,保护 OS 内存、特权 CPU 指令、磁盘和其他只有内核才能访问的资源。

在用户空间内运行的应用程序只能通过 trap 访问这些资源,在这种情况下,内核会接管控制权并代表用户空间应用程序执行这些指令。例如,想要访问磁盘上的文件的应用程序必须通过系统调用(在内部捕获到内核中)将此调用委托给内核,然后 Linux 内核代表应用程序执行此请求。

由于可能有许多用户空间应用程序在单个 Linux 内核上并行运行,因此我们需要一种方法在这些基于用户空间的应用程序之间提供隔离。

我们所说的隔离是指应该有一种单独的应用程序的沙箱,以便应用程序中的某些资源被限制在该沙箱中。

实现这种沙箱的技术是通过 Linux 内核中的特定数据结构(称为namespace)完成的。

## 内核数据结构

### task_struct

内核将每个进程表示为 task_struct 数据结构：

```c
/* task_struct member predeclarations (sorted alphabetically):
*/
struct audit_context;
struct backing_dev_info;
struct bio_list;
struct blk_plug;
struct capture_control;
struct cfs_rq;
struct fs_struct;
struct futex_pi_state;
struct io_context;
struct mempolicy;
struct nameidata;
struct nsproxy;
struct perf_event_context;
struct pid_namespace;
struct pipe_inode_info;
struct rcu_node;
struct reclaim_state;
struct robust_list_head;
struct root_domain;
struct rq;
struct sched_attr;
struct sched_param;
struct seq_file;
struct sighand_struct;
struct signal_struct;
struct task_delay_info;
struct task_group;
```

### nsproxy

nsproxy 结构体是 task (进程)所属的不同命名空间的持有者结构。

```c
struct nsproxy {
       atomic_t count;
       struct uts_namespace *uts_ns;
       struct ipc_namespace *ipc_ns;
       struct mnt_namespace *mnt_ns;
       struct pid_namespace *pid_ns_for_children;
       struct net           *net_ns;
       struct time_namespace *time_ns;
       struct time_namespace *time_ns_for_children;
       struct cgroup_namespace *cgroup_ns;
};
extern struct nsproxy init_nsproxy;
```

nsproxy 包含八个 namespace 数据结构。缺少的是 user namespace, 它是 task_struct 中 cred 数据结构的一部分。

有三个系统调用可用于将 task 放入特定命名空间:

1. clone
2. unshare
3. setns

clone 和 setns 调用会导致创建一个 nsproxy 对象,然后添加 task 所需的特定命名空间。

## 发展历史

对命名空间的支持最初于内核 2.4.19（挂载命名空间）和 2.6.19（PID 命名空间）引入，并在内核 3.8 后逐步完善（用户命名空间）。现代 Linux 内核（5.6+）支持以下类型的命名空间：

| 命名空间类型 | 隔离内容                     | 相关标志          | 内核版本 |
| :----------- | :--------------------------- | :---------------- | :------- |
| Cgroup       | Cgroup 根目录                | `CLONE_NEWCGROUP` | 4.6      |
| IPC          | System V IPC、POSIX 消息队列 | `CLONE_NEWIPC`    | 2.6.19   |
| Network      | 网络设备、协议栈、端口等     | `CLONE_NEWNET`    | 2.6.24   |
| Mount        | 挂载点                       | `CLONE_NEWNS`     | 2.4.19   |
| PID          | 进程ID                       | `CLONE_NEWPID`    | 2.6.24   |
| Time         | 时钟                         | `CLONE_NEWTIME`   | 5.6      |
| User         | 用户和组ID                   | `CLONE_NEWUSER`   | 3.8      |
| UTS          | 主机名和域名                 | `CLONE_NEWUTS`    | 2.6.19   |









