---
title: "CPU cgroups"
linkTitle: "CPU"
weight: 10
date: 2025-04-30
description: >
  CPU cgroups 用于限制进程的 CPU 使用
---

## 概述

CPU cgroups 可以在两个调度器之上实现:

- 完全公平的调度程序(Completely fair scheduler / CFS)
- 实时调度器(Real-Time scheduler / RRS)

CPU cgroup 提供两种类型的 CPU 资源控制:

- cpu.shares: 指定 cgroup 中 task 可用的 CPU 时间的相对份额

- cpu.cfs_period_us: 指定 cgroup 中的所有任务在一个时间段(如 cpu.cfs_period_us 定义)内可以运行的总时间量(以微秒为单位,μs,此处表示为“us”)

- cpu.cfs_quota_us: 这是从 cgroups (cpu.cfs_quota_us) 的 CPU 配额开始划分的时间段,配额和时间段参数基于每个 CPU 运行。

## CFS 任务调度程序

CFS 计划程序的目标是为系统上运行的所有任务授予公平份额的 CPU 资源。

CFS 使用一个称为虚拟时钟的抽象来实现公平性。虚拟时钟是系统中所有任务共享的单一时钟,它以恒定速率递增,不受系统负载的影响。

任务分为两种类型: 

- CPU 密集型任务: 加密、机器学习、查询处理等任务
- I/O 密集型任务: 使用磁盘或网络 I/O 的任务,如数据库客户端

计划程序负责计划这两种任务。

CFS 使用 vruntime 的概念。vruntime 是 sched_entity 结构体的成员,而 sched_entity 结构体是 task_struct 结构体的成员

### task_struct

```c
struct task_struct {
  int prio, static_prio, normal_prio;
  unsigned int rt_priority;
  struct list_head run_list;
  const struct sched_class *sched_class;
  struct sched_entity se;
  unsigned int policy;
  cpumask_t cpus_allowed;
  unsigned int time_slice;
  int time_slice;
}
```

### sched_entity

```c
struct sched_entity {
  /* For load-balancing: */
  struct load_weight load;
  struct rb_node run_node;
  struct list_head group_node;
  unsigned int on_rq;
  u64 exec_start;
  u64 sum_exec_runtime;
  u64 vruntime;
  u64 prev_sum_exec_runtime;
  u64 nr_migrations;
  struct sched_statistics statistics;
#ifdef CONFIG_FAIR_GROUP_SCHED
  int depth;
  struct sched_entity *parent;
  /* rq on which this entity is (to be) queued: */
  struct cfs_rq
  *cfs_rq;
  /* rq "owned" by this entity/group: */
  struct cfs_rq *my_q;
  /* cached value of my_q->h_nr_running */
  unsigned long runnable_weight;
}
```

task_struct 具有对 sched_entity 的引用,而 sched_entity 包含对 vruntime 的引用。

### vruntime

vruntime使用以下步骤进行计算:

1. 计算进程在 CPU 上花费的时间
2. 权衡计算出的运行时间与可运行进程的数量

内核使用 https://elixir.bootlin.com/linux/latest/source/kernel/sched/fair.c 文件中定义的 update_curr 函数。

```c
/*
* 更新当前任务的运行时间统计信息。
*/
static void update_curr(struct cfs_rq *cfs_rq)
{
  struct sched_entity *curr = cfs_rq-> curr;
  u64 now = rq_clock_task(rq_of(cfs_rq));
  u64 delta_exec;

  if (unlikely(!curr))
    return;

  delta_exec = now - curr->exec_start;
  if (unlikely((s64)delta_exec <= 0))
    return;


  curr->exec_start = now;

  schedstat_set(curr->statistics.exec_max,
    max(delta_exec, curr->statistics.exec_max));

  curr->sum_exec_runtime += delta_exec;
  schedstat_add(cfs_rq->exec_clock, delta_exec);

  curr->vruntime += calc_delta_fair(delta_exec, curr);
  update_min_vruntime(cfs_rq);

  if (entity_is_task(curr)) {
    struct task_struct *curtask = task_of(curr);
    trace_sched_stat_runtime(curtask, delta_exec,
    curr->vruntime);
    cgroup_account_cputime(curtask, delta_exec);
    account_group_exec_runtime(curtask, delta_exec);
  }
  account_cfs_rq_runtime(cfs_rq, delta_exec);
}
```

calc_delta_fair 函数计算进程的 vruntime。

### 分组调度

CFS 使用分组调度来实现公平性。分组调度允许将任务组织成层次结构,从而实现公平性。





