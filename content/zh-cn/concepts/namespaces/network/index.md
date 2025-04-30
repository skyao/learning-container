---
title: "Network namespace"
linkTitle: "Network"
weight: 40 
date: 2025-04-30
description: >
  为容器提供了一组单独的网络子系统
---

## 介绍

network namespace 为容器提供了一组单独的网络子系统。

这意味着 network namespace 中的进程将看到不同的网络接口、路由和 iptables。

这将容器网络与主机网络分开。

## 内核数据结构

### net

net 结构体是内核中表示 net namespace 的结构体:

```c
struct net {
       /* First cache line can be often dirtied.
        * Do not place read-mostly fields here.
        */
       refcount_t           passive;       /* To decide when the network namespace should be freed. */
       refcount_t           count;        /* To decided when the network namespace should be shut down. */
       spinlock_t           rules_mod_lock;
       unsigned int         dev_unreg_count;
       unsigned int         dev_base_seq;   /* protected by rtnl_mutex */
       int                  ifindex;
       spinlock_t          nsid_lock;
       atomic_t            fnhe_genid;
       struct list_head    list;            /* list of network namespaces */  
       struct list_head    exit_list;       /* To linked to call pernet exit methods on dead net (pernet_ops_rwsem read locked), or to unregister pernet ops (pernet_ops_rwsem write locked). */
       struct llist_node   cleanup_list;   /* namespaces on death row */
#ifdef CONFIG_KEYS
       struct key_tag          *key_domain; /* Key domain of operation tag */
#endif
       struct user_namespace   *user_ns;    /* Owning user namespace */
       struct ucounts           *ucounts;
       struct idr               netns_ids;
       struct ns_common    ns;
       struct list_head    dev_base_head;
       struct proc_dir_entry    *proc_net;
       struct proc_dir_entry    *proc_net_stat;
#ifdef CONFIG_SYSCTL
       struct ctl_table_set      sysctls;
#endif
       struct sock            *rtnl;        /* rtnetlink socket */
       struct sock            *genl_sock;
       struct uevent_sock     *uevent_sock;  /* uevent socket */
       struct hlist_head      *dev_name_head;
       struct hlist_head      *dev_index_head;
       struct raw_notifier_head     netdev_chain;
};
```

此数据结构体的元素之一是此 network namespace 所属的 user namespace (user_ns字段)。

### netns_ipv4

netns_ipv4 结构体是内核中表示 ipv4 namespace 的结构体，其中包括路由表、网络过滤规则等:

```c
struct netns_ipv4 {
#ifdef CONFIG_SYSCTL
       struct ctl_table_header *forw_hdr;
       struct ctl_table_header *frags_hdr;
       struct ctl_table_header *ipv4_hdr;
       struct ctl_table_header *route_hdr;
       struct ctl_table_header *xfrm4_hdr;
#endif
       struct ipv4_devconf *devconf_all;
       struct ipv4_devconf *devconf_dflt;
       struct ip_ra_chain __rcu *ra_chain;
       struct mutex ra_mutex;
#ifdef CONFIG_IP_MULTIPLE_TABLES
       struct fib_rules_ops *rules_ops;
       bool fib_has_custom_rules;
       unsigned int fib_rules_require_fldissect;
       struct fib_table __rcu *fib_main;
       struct fib_table __rcu *fib_default;
#endif
       bool fib_has_custom_local_routes;
#ifdef CONFIG_IP_ROUTE_CLASSID
       int fib_num_tclassid_users;
#endif
       struct hlist_head *fib_table_hash;
       bool fib_offload_disabled;
       struct sock *fibnl;
       struct sock * __percpu *icmp_sk;
       struct sock *mc_autojoin_sk;
       struct inet_peer_base *peers;
       struct sock * __percpu *tcp_sk;
       struct fqdir *fqdir;
#ifdef CONFIG_NETFILTER
       struct xt_table *iptable_filter;
       struct xt_table *iptable_mangle;
       struct xt_table *iptable_raw;
       struct xt_table *arptable_filter;
#ifdef CONFIG_SECURITY
       struct xt_table *iptable_security;
#endif
       struct xt_table *nat_table;
#endif
       int sysctl_icmp_echo_ignore_all;
       int sysctl_icmp_echo_ignore_broadcasts;
      int sysctl_icmp_ignore_bogus_error_responses;
      int sysctl_icmp_ratelimit;
      int sysctl_icmp_ratemask;
      int sysctl_icmp_errors_use_inbound_ifaddr;
      struct local_ports ip_local_ports;
      int sysctl_tcp_ecn;
      int sysctl_tcp_ecn_fallback;
      int sysctl_ip_default_ttl;
      int sysctl_ip_no_pmtu_disc;
      int sysctl_ip_fwd_use_pmtu;
      int sysctl_ip_fwd_update_priority;
      int sysctl_ip_nonlocal_bind;
      int sysctl_ip_autobind_reuse;
      /* Shall we try to damage output packets if routing dev changes? */
      int sysctl_ip_dynaddr;
};
```

这就是 iptables 和路由规则都被划分到 network 命名空间中的方式。


