---
title: "Mount namespace"
linkTitle: "Mount"
weight: 30 
date: 2025-04-30
description: >
  控制进程看到的挂载点
---

Mount namespace 控制进程应该看到哪些挂载点。

如果进程位于命名空间中, 则它只会看到该命名空间内的挂载。

### vfsmount 数据结构

内核中的挂载由名为 vfsmount 的数据结构表示。

所有挂载都形成一个树状结构,其中子挂载结构包含对父挂载结构的引用。

```c
struct vfsmount {
  struct list_head mnt_hash;
  struct vfsmount *mnt_parent;    /* fs we are mounted on */
  struct dentry *mnt_mountpoint;  /* dentry of mountpoint */
  struct dentry *mnt_root;      /* root of the mounted tree*/
  struct super_block *mnt_sb;     /* pointer to superblock */
  struct list_head mnt_mounts;  /* list of children, anchored here */
  struct list_head mnt_child;
  atomic_t mnt_count;
  int mnt_flags;
  char *mnt_devname;
  /* and going through their mnt_child */
  /* Name of device e.g. /dev/dsk/hda1 */
  struct list_head mnt_list;
};                     
```

每当调用挂载操作时,都会创建一个 vfsmount 结构,并填充挂载点的 dentry 以及挂载树的 dentry。dentry 是将 inode 映射到文件名的数据结构。

除了 mount 之外,还有一个 bind mount,它允许将目录(而不是设备)挂载到挂载点。绑定挂载过程会导致创建指向目录的 dentry 的 vfsmount 结构。

容器采用绑定挂载的概念。因此,当为容器创建卷时,它实际上是主机中目录与容器文件系统中的挂载点的绑定挂载。由于挂载发生在 mount 命名空间内,因此 vfsmount 结构的范围限定为 mount 命名空间。

这意味着,通过创建目录的绑定挂载,我们可以在包含容器的命名空间中公开一个卷。


