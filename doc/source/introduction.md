---
tip: translate by openai@2023-06-20 08:19:02
title: Introduction
---

Load balancing is a method of distributing IP traffic across a cluster of real servers, providing one or more highly available virtual services. When designing load-balanced topologies, it is important to account for the availability of the load balancer itself as well as the real servers behind it.

> 负载均衡是一种将 IP 流量分布到一群真实服务器上的方法，提供一个或多个高可用的虚拟服务。在设计负载平衡拓扑时，重要的是要考虑负载均衡器本身以及它背后的真实服务器的可用性。

Keepalived provides frameworks for both load balancing and high availability. The load balancing framework relies on the well-known and widely used Linux Virtual Server (IPVS) kernel module, which provides Layer 4 load balancing. Keepalived implements a set of health checkers to dynamically and adaptively maintain and manage load balanced server pools according to their health. High availability is achieved by the Virtual Redundancy Routing Protocol (VRRP). VRRP is a fundamental brick for router failover. In addition, keepalived implements a set of hooks to the VRRP finite state machine providing low-level and high-speed protocol interactions. Each Keepalived framework can be used independently or together to provide resilient infrastructures.

> Keepalived 提供负载均衡和高可用性的框架。负载均衡框架依赖于众所周知的广泛使用的 Linux 虚拟服务器（IPVS）内核模块，它提供了第 4 层负载均衡。Keepalived 实现了一组健康检查程序，根据服务器池的健康状况，动态地和自适应地维护和管理负载均衡服务器池。高可用性通过虚拟冗余路由协议（VRRP）实现。VRRP 是路由器故障转移的基础砖。此外，Keepalived 实现了一组钩子，用于 VRRP 有限状态机，提供低级别和高速协议交互。每个 Keepalived 框架可以单独使用或一起使用，以提供弹性基础架构。

In this context, load balancer may also be referred to as a _director_ or an _LVS router_.

> 在这种情况下，负载均衡器也可被称为导演或 LVS 路由器。

In short, Keepalived provides two main functions:

> 简而言之，Keepalived 提供两个主要功能：

- Health checking for LVS systems
- Implementation of the VRRPv2 stack to handle load balancer failover
