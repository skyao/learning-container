---
title: "Time namespace"
linkTitle: "Time"
weight: 70 
date: 2025-04-30
description: >
  Time
---


time namespace 有两个主要用例:

- 更改容器内的日期和时间
- 调整从 checkpoint 恢复的容器的时钟

内核提供对几个 clocks 的访问: CLOCK_REALTIME, CLOCK_MONOTONIC, 和 CLOCK_BOOTTIME。

最后两个 clocks 是 monotonous （单调的？）, 但它们的起点没有明确定义(目前是系统启动时间,但 POSIX 说“since an unspecified point in past”), 并且每个系统都不同。

当容器从一个节点迁移到另一个节点时,所有 clocks 都会恢复到其一致状态。




