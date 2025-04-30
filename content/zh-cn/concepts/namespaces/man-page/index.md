---
title: "namespaces 官方文档"
linkTitle: "官方文档"
weight: 99 
date: 2025-04-30
description: >
  namespace 的官方文档
---

https://man7.org/linux/man-pages/man7/namespaces.7.html

## 描述

命名空间将全局系统资源抽象化，使命名空间内的进程看起来拥有自己的全局资源独立实例。作为命名空间成员的其他进程可以看到对全局资源的更改，但其他进程则看不到。命名空间的一个用途是实现容器。

本页提供了有关各种命名空间类型信息的指针，描述了相关的 /proc 文件，并总结了使用命名空间的 API。

## Namespace 类型

下表列出了 Linux 上可用的命名空间类型。

表中第二列显示了用于在各种 API 中指定命名空间类型的标志值。最后一列是被命名空间类型隔离的资源摘要。

| Namespace | Flag            | Isolates                             |
| --------- | --------------- | ------------------------------------ |
| Cgroup    | CLONE_NEWCGROUP | Cgroup root directory                |
| IPC       | CLONE_NEWIPC    | System V IPC, POSIX message queues   |
| Network   | CLONE_NEWNET    | Network devices, stacks, ports, etc. |
| Mount     | CLONE_NEWNS     | Mount points                         |
| PID       | CLONE_NEWPID    | Process IDs                          |
| Time      | CLONE_NEWTIME   | Boot and monotonic clocks            |
| User      | CLONE_NEWUSER   | User and group IDs                   |
| UTS       | CLONE_NEWUTS    | Hostname and NIS domain name         |

## Namespace API

除了下面描述的各种 `/proc` 文件外，namespace API 包括以下系统调用：

- clone

  clone 系统调用会创建一个新进程。 
  
  如果调用的 flags 参数指定了一个或多个上面列出的 CLONE_NEW* 标志，那么将为每个标志创建新的命名空间，子进程将成为这些命名空间的成员。 
  
  (该系统调用还实现了许多与命名空间无关的功能）。

- setns

  setns 系统调用允许调用进程加入一个现有的命名空间。
  
  要加入的命名空间是通过文件描述符指定的，该文件描述符指向下面描述的 `/proc/pid/ns` 文件中的一个文件描述符来指定要加入的命名空间。

- unshare

  unshare 系统调用会将调用进程移动到一个新的命名空间。
  
  如果调用的 flags 参数指定了上述 CLONE_NEW* 标志中的一个或多个，那么将为每个标志创建新的命名空间，调用进程将成为这些命名空间的成员。 
  
  (该系统调用还实现了许多与命名空间无关的功能）。

- ioctl

  各种 ioctl 操作可用于发现命名空间的信息。 这些操作在 ioctl_nsfs 中进行了描述。
  
在大多数情况下，使用 clone 和 unshare 创建新命名空间需要具备 CAP_SYS_ADMIN 能力，因为在新命名空间中，创建者将拥有修改全局资源的权限，这些资源对后续在命名空间内创建或加入的进程可见。user 命名空间是一个例外：自 Linux 3.8 起，创建 users 命名空间无需任何特权。

## `/proc/pid/ns/` 目录

每个进程都有一个 `/proc/pid/ns/` 子目录，其中包含每个支持 setns 操作的命名空间的一个条目:

```bash
$ ls -l /proc/$$/ns | awk '{print $1, $9, $10, $11}'
total 0
lrwxrwxrwx. cgroup -> cgroup:[4026531835]
lrwxrwxrwx. ipc -> ipc:[4026531839]
lrwxrwxrwx. mnt -> mnt:[4026531840]
lrwxrwxrwx. net -> net:[4026531969]
lrwxrwxrwx. pid -> pid:[4026531836]
lrwxrwxrwx. pid_for_children -> pid:[4026531834]
lrwxrwxrwx. time -> time:[4026531834]
lrwxrwxrwx. time_for_children -> time:[4026531834]
lrwxrwxrwx. user -> user:[4026531837]
lrwxrwxrwx. uts -> uts:[4026531838]
```

将该目录下的某个文件绑定挂载到文件系统的其他位置，可以保持由 `pid` 指定的进程对应的命名空间存活，即使该命名空间内的所有进程均已终止。  

打开该目录下的某个文件（或绑定挂载到这些文件的某个文件），会返回由 `pid` 指定的进程对应命名空间的文件句柄。只要该文件描述符保持打开状态，即使命名空间内的所有进程终止，命名空间也会继续存在。该文件描述符可以传递给 `setns` 使用。  

在 Linux 3.7 及更早版本中，这些文件以硬链接形式显示。自 Linux 3.8 起，它们以符号链接形式出现。如果两个进程位于同一命名空间，它们的 `/proc/pid/ns/xxx` 符号链接的设备 ID 和 inode 号将相同；应用程序可以使用 `stat` 返回的 `stat.st_dev` 和 `stat.st_ino` 字段来验证这一点。该符号链接的内容是一个字符串，包含命名空间类型和 inode 号，示例如下：

```bash
$ readlink /proc/$$/ns/uts
uts:[4026531838]
```

此目录中的符号链接如下：

- `/proc/pid/ns/cgroup`（自 Linux 4.6 起） : 此文件是进程 cgroup 命名空间的句柄。
- `/proc/pid/ns/ipc`（自 Linux 3.0 起）： 此文件是进程 IPC 命名空间的句柄。
- `/proc/pid/ns/mnt`（自 Linux 3.8 起）： 此文件是进程 mount 命名空间的句柄。
- `/proc/pid/ns/net`（自 Linux 3.0 起）： 此文件是进程 network 命名空间的句柄。
- `/proc/pid/ns/pid`（自 Linux 3.8 起）： 此文件是进程 PID 命名空间的句柄。此句柄在进程的生存期内是永久性的 (i.e.,进程的 PID 命名空间成员永远不会改变）。
- `/proc/pid/ns/pid_for_children`（自 Linux 4.12 起）： 此文件是当前进程所创建子进程的PID命名空间句柄。由于可能调用`unshare`和`setns`（参见`pid_namespaces`），该文件可能会发生变化，因此可能与 `/proc/pid/ns/pid` 不同。该符号链接只有在命名空间中创建第一个子进程后才会获得有效值（在此之前，对该符号链接执行`readlink`将返回空缓冲区）。
- `/proc/pid/ns/time`（自 Linux 5.6 起）： 此文件是进程 time 命名空间的句柄。
- `/proc/pid/ns/time_for_children`（自 Linux 5.6 起）： 此文件是当前进程所创建子进程的 time 命名空间句柄。调用 unshare 和 setns 后，时间命名空间可能会发生变化（参见 time_namespaces），因此该文件可能与 `/proc/pid/ns/time` 不同。
- `/proc/pid/ns/user`（自 Linux 3.8 起）： 此文件是进程 user 命名空间的句柄。
- `/proc/pid/ns/uts`（自 Linux 3.0 起）： 此文件是进程 UTS 命名空间的句柄。

取消引用或读取 (readlink) 这些符号链接的权限受 ptrace 访问模式 PTRACE_MODE_READ_FSCREDS 检查的制约；请参阅 ptrace。

## `/proc/sys/user` 目录

`/proc/sys/user` 目录中的文件（自 Linux 4.9）暴露了各种 可以创建的类型。文件如下：

- max_cgroup_namespaces： 该文件中的值定义了用户命名空间内可创建的 cgroup 命名空间数量的单用户上限。

- max_ipc_namespaces： 该文件中的值定义了用户命名空间内可创建的 ipc 命名空间数量的单用户上限。

- max_mount_namespaces： 该文件中的值定义了用户命名空间内可创建的 mount 命名空间数量的单用户上限。

- max_net_namespaces： 该文件中的值定义了用户命名空间内可创建的 network 命名空间数量的单用户上限。

- max_pid_namespaces： 该文件中的值定义了用户命名空间内可创建的 PID 命名空间数量的单用户上限。

- max_time_namespaces： 该文件中的值（自Linux 5.7起）定义了用户命名空间内可创建的 time 命名空间数量的单用户上限。

- max_user_namespaces： 该文件中的值定义了用户命名空间内可创建的 user 命名空间数量的单用户上限。

- max_uts_namespaces： 该文件中的值定义了用户命名空间内可创建的 uts 命名空间数量的单用户上限。



关于这些文件需注意以下细节：

- 特权进程可修改这些文件中的数值  

- 这些文件显示的数值是打开进程所在用户命名空间的限制  

- 限制是按用户计算的。同一用户命名空间内的每个用户均可创建数量达到定义上限的命名空间  

- 限制适用于所有用户（包括UID 0）  

- 这些限制是在其他可能存在的按命名空间限制（如PID和用户命名空间的限制）之外额外实施的  

- 达到这些限制时，clone 和 unshare 会失败并返回ENOSPC错误  

- 在初始用户命名空间中，这些文件的默认值是允许创建线程数上限（`/proc/sys/kernel/threads-max`）的一半；在所有子用户命名空间中，默认值为 MAXINT  

- 创建命名空间时，该对象也会计入祖先命名空间。具体而言：  

  - 每个用户命名空间都有创建者UID  

  - 创建命名空间时，会将其计入每个祖先用户命名空间的创建者UID，内核会确保不超过祖先命名空间中该创建者UID对应的命名空间限制  

  - 上述机制确保创建新用户命名空间不能作为逃避当前用户命名空间强制限制的手段

## namespace 生命周期

如果没有其他因素影响，当一个命名空间中的最后一个进程终止或离开时，该命名空间会自动销毁。但即使没有成员进程，仍可能存在多种因素使命名空间保持存在，这些因素包括：

- 对应的 `/proc/pid/ns/*` 文件存在打开的文件描述符或绑定挂载

- 命名空间是层级式的（如 PID 或用户命名空间）且拥有子命名空间

- 作为用户命名空间，拥有一个或多个非用户命名空间  

- 作为 PID 命名空间，存在进程通过 `/proc/pid/ns/pid_for_children` 符号链接引用该命名空间  

- 作为时间命名空间，存在进程通过 `/proc/pid/ns/time_for_children` 符号链接引用该命名空间  

- 作为 IPC 命名空间，对应的 mqueue 文件系统挂载（参见 `mq_overview`）引用了该命名空间  

- 作为 PID 命名空间，对应的 proc 文件系统挂载引用了该命名空间  





