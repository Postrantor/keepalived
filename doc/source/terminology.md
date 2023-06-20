---
tip: translate by openai@2023-06-20 08:32:25
title: Terminology
---

::: todo
put image here
:::

LVS stands for "Linux Virtual Server". LVS is a patched Linux kernel that adds a load balancing facility. For more information on LVS, please refer to the project homepage: [http://www.linux-vs.org](http://www.linux-vs.org). LVS acts as a network bridge (using NAT) to load balance TCP/UDP stream. The LVS router components are:

> LVS 代表“Linux 虚拟服务器”。LVS 是一个补丁 Linux 内核，增加了负载均衡功能。有关 LVS 的更多信息，请参阅项目主页：[http://www.linux-vs.org](http://www.linux-vs.org)。LVS 作为一个网络桥接（使用 NAT）来负载均衡 TCP / UDP 流。LVS 路由器组件有：

- WAN Interface: Ethernet Network Interface Controller that will be accessed by all the clients.
- LAN Interface: Ethernet Network Interface Controller to manage all the load balanced servers.
- Linux kernel: The kernel is patched with the latest LVS and is used as a router OS.

In this document, we will use the following keywords:

> 在本文档中，我们将使用以下关键字：

# LVS Component

::: glossary

IPVS (IP Virtual Server)

> IPVS（IP 虚拟服务器）

: The linux kernel module that provides packet forwarding services for network load balancing (Layer-4 switching).

Director

: The server running IPVS which is responsible for distributing incoming network traffic among a pool of backend servers.

Real server

: A real server hosts the application accessed by client requests. WEB SERVER 1 & WEB SERVER 2 in our synopsis.

Server pool

: A farm of real servers.

RIP (Real IP)

: Real IP address configured at a real server in the LVS cluster.

VIP

: The Virtual IP is the IP address that will be accessed by all the clients. The clients only access this IP address.

DIP (Director IP)

: IP address of the LVS director node. Director uses this when communicating with a Real server.

Virtual server

: The access point to a Server pool.

Virtual Service

: A TCP/UDP service associated with the VIP.
:::

# VRRP Component

::: glossary

VRRP

: The protocol implemented for the directors' failover/virtualization.

IP Address owner

: The VRRP Instance that has the IP address(es) as real interface address(es). This is the VRRP Instance that, when up, will respond to packets addressed to one of these IP address(es) for ICMP, TCP connections, \...

MASTER state

: VRRP Instance state when it is assuming the responsibility of forwarding packets sent to the IP address(es) associated with the VRRP Instance. This state is illustrated on "Case study: Failover" by a red line.

BACKUP state

: VRRP Instance state when it is capable of forwarding packets in the event that the current VRRP Instance MASTER fails.

Real Load Balancer

: An LVS director running one or many VRRP Instances.

Virtual Load balancer

: A set of Real Load balancers.

Synchronized Instance

: VRRP Instance with which we want to be synchronized. This provides VRRP Instance monitoring.

Advertisement

: The name of a simple VRRPv2 packet sent to a set of VRRP Instances while in the MASTER state.
:::

::: todo
:::
