---
title: "Linux Container 信息收集"
linkTitle: "信息收集"
weight: 90
date: 2025-04-24
description: >
  Linux Container 信息收集
---


## deepseek 的建议

咨询了一下 deepseek ，请它给出一些学习建议，以下是它的回答：

以下是去掉加粗字体后的推荐书单和资源，保持内容结构清晰：

---

### 1. 容器基础与底层技术
#### 英文书籍
- "Linux Containers and Virtualization: A Kernel Perspective" (by Shashank Mohan Jain)  
  - 从内核视角深入讲解容器技术（命名空间、cgroups、文件系统隔离等），适合想理解底层机制的读者。
- "Container Security" (by Liz Rice)  
  - 聚焦容器安全，涉及运行时隔离、镜像漏洞分析等，适合安全方向的学习者。
- "The Linux Programming Interface" (by Michael Kerrisk)  
  - 虽然不是容器专书，但它是理解 Linux 系统调用、进程管理和命名空间的权威参考，容器技术的基石。

#### 中文书籍
- "Docker 容器与容器云"（浙江大学 SEL 实验室著）  
  - 从 Docker 基础到 Kubernetes 原理，兼顾实践与源码分析，适合中文读者。
- "自己动手写 Docker"（陈显鹭著）  
  - 通过动手实现简易 Docker，理解命名空间、cgroups 和镜像分层等核心概念。

---

### 2. 容器运行时与高级实现
- "Designing Distributed Systems" (by Brendan Burns)  
  - Kubernetes 联合创始人撰写，讲解容器编排系统的设计模式，适合进阶分布式场景。
- "Kubernetes in Action" (by Marko Luksa)  
  - 深入 Kubernetes 内部原理，涵盖 Pod、调度、网络等实现细节。
- "Container Networking" (by Michael Hausenblas)  
  - 专门讲解容器网络模型（CNI）、Service Mesh 等网络实现。

---

### 3. 开源代码与官方文档
- Linux 内核文档  
  - [Namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html) 和 [cgroups](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html) 的官方文档是终极参考。
- Docker/Kubernetes 源码  
  - Docker 的 [containerd](https://github.com/containerd/containerd) 和 Kubernetes 的 [kubelet](https://github.com/kubernetes/kubernetes) 是学习工业级实现的优秀案例。
- OCI 标准文档  
  - [Open Container Initiative (OCI) Runtime Spec](https://github.com/opencontainers/runtime-spec) 定义了容器运行时的标准化实现。

---

### 4. 在线课程与论文
- Linux Foundation 课程  
  - [《Linux Container Internals》](https://training.linuxfoundation.org/) 提供实验环境深入操作。
- 论文阅读  
  - Google 的 Borg 论文（Kubernetes 前身）、Docker 早期架构白皮书等。

---

### 学习路径建议
1. 先理解基础：Linux 命名空间、cgroups、联合文件系统（OverlayFS/AUFS）。  
2. 动手实验：用 lsns、cgcreate 等命令手动创建隔离环境，再对比 Docker 的实现。  
3. 深入源码：从简易容器（如 [LXC](https://linuxcontainers.org/)）过渡到 Docker/containerd。
4. 扩展生态：学习 Kubernetes 或 Service Mesh（如 Istio）的容器编排逻辑。

如果需要更具体的某方面资源（如安全、网络或性能优化），可以进一步补充说明！
