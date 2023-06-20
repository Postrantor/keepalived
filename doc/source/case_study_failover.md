---
tip: translate by openai@2023-06-20 08:09:18
title: "Case Study: Failover using VRRP"
---

As an example we can introduce the following LVS topology:

> 例如，我们可以介绍以下 LVS 拓扑结构：

# Architecture Specification

To create a virtual LVS director using the VRRPv2 protocol, we define the following architecture:

> 要使用 VRRPv2 协议创建虚拟 LVS 导演，我们定义以下架构：

- 2 LVS directors in active-active configuration.

- 4 VRRP Instances per LVS director: 2 VRRP Instance in the MASTER state and 2 in BACKUP state. We use a symmetric state on each LVS directors.

> 每个 LVS 导演有 4 个 VRRP 实例：2 个处于主状态的 VRRP 实例和 2 个处于备份状态的 VRRP 实例。我们在每个 LVS 导演上使用对称状态。

- 2 VRRP Instances in the same state are to be synchronized to define a persistent virtual routing path.

> 两个处于同一状态的 VRRP 实例必须同步以定义持久的虚拟路由路径。

- Strong authentication: IPSEC-AH is used to protect our VRRP advertisements from spoofed and reply attacks.

> 强有力的身份验证：使用 IPSEC-AH 来保护我们的 VRRP 广告免受伪造和回复攻击。

The VRRP Instances are compounded with the following IP addresses:

> VRRP 实例由以下 IP 地址组成：

- VRRP Instance VI_1: owning VRRIP VIPs VIP1 & VIP2. This instance defaults to the MASTER state on LVS director 1. It stays synchronized with VI_2.

> 实例 VI_1：拥有 VRRIP VIP VIP1 和 VIP2。该实例默认在 LVS 导演 1 处处于主节点状态。它与 VI_2 保持同步。

- VRRP Instance VI_2: owning DIP1. This instance is by default in MASTER state on LVS director 1. It stays synchronized with VI_1.

> 实例 VI_2：拥有 DIP1。此实例默认在 LVS 导演 1 上处于主状态。它与 VI_1 保持同步。

- VRRP Instance VI_3: owning VRRIP VIPs VIP3 & VIP4. This instance is in default MASTER state on LVS director 2. It stays synchronized with VI_4.

> VRRP 实例 VI_3：拥有 VRRIP VIPs VIP3 和 VIP4。此实例处于 LVS 导演 2 上的默认主状态。它与 VI_4 保持同步。

- VRRP Instance VI_4: owning DIP2. This instance is in default MASTER state on LVS director 2. It stays synchronized with VI_3.

> VRRP 实例 VI_4：拥有 DIP2。该实例在 LVS 导演 2 上处于默认的主状态。它与 VI_3 保持同步。

# Keepalived Configuration

The whole configuration is done in the /etc/keepalived/keepalived.conf file. In our case study this file on LVS director 1 looks like:

> 整个配置都在/etc/keepalived/keepalived.conf 文件中完成。在我们的案例研究中，LVS director 1 上的这个文件看起来像：

```
vrrp_sync_group VG1 {
    group {
        VI_1
        VI_2
    }
}
vrrp_sync_group VG2 {
    group {
        VI_3
        VI_4
    }
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type AH
        auth_pass k@l!ve1
    }
    virtual_ipaddress {
        192.168.200.10
        192.168.200.11
    }
}
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 52
    priority 150
    advert_int 1
    authentication {
        auth_type AH
        auth_pass k@l!ve2
    }
    virtual_ipaddress {
        192.168.100.10
    }
}

vrrp_instance VI_3 {
    state BACKUP
    interface eth0
    virtual_router_id 53
    priority 100
    advert_int 1
    authentication {
        auth_type AH
        auth_pass k@l!ve3
    }
    virtual_ipaddress {
        192.168.200.12
        192.168.200.13
    }
}
vrrp_instance VI_4 {
    state BACKUP
    interface eth1
    virtual_router_id 54
    priority 100
    advert_int 1
    authentication {
        auth_type AH
        auth_pass k@l!ve4
    }
    virtual_ipaddress {
        192.168.100.11
    }
}
```

Then we define the symmetric configuration file on LVS director 2. This means that VI_3 & VI_4 on LVS director 2 are in MASTER state with a higher priority 150 to start with a stable state. Symmetrically VI_1 & VI_2 on LVS director 2 are in default BACKUP state with lower priority of 100. \| This configuration file specifies 2 VRRP Instances per physical NIC. When you run Keepalived on LVS director 1 without running it on LVS director 2, LVS director 1 will own all the VRRP VIP. So if you use the ip utility you may see something like: (On Debian the ip utility is part of iproute):

> 然后，我们在 LVS director 2 上定义对称的配置文件。这意味着 LVS director 2 上的 VI_3 和 VI_4 处于拥有更高优先级 150 的主状态，以便以稳定的状态开始。对称地，LVS director 2 上的 VI_1 和 VI_2 处于具有较低优先级 100 的默认备份状态。 \|此配置文件为每个物理 NIC 指定 2 个 VRRP 实例。当您仅在 LVS director 1 上运行 Keepalived 而不在 LVS director 2 上运行它时，LVS director 1 将拥有所有 VRRP VIP。因此，如果您使用 ip 实用程序，您可能会看到类似的东西：（在 Debian 中，ip 实用程序是 iproute 的一部分）：

```
[root@lvs1 tmp]# ip address list
1: lo: <LOOPBACK,UP> mtu 3924 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 brd 127.255.255.255 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
    link/ether 00:00:5e:00:01:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.5/24 brd 192.168.200.255 scope global eth0
    inet 192.168.200.10/32 scope global eth0
    inet 192.168.200.11/32 scope global eth0
    inet 192.168.200.12/32 scope global eth0
    inet 192.168.200.13/32 scope global eth0
3: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
    link/ether 00:00:5e:00:01:32 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.5/24 brd 192.168.201.255 scope global eth1
    inet 192.168.100.10/32 scope global eth1
    inet 192.168.100.11/32 scope global eth1
```

Then simply start Keepalived on the LVS director 2 and you will see:

> 然后在 LVS director 2 上启动 Keepalived，您将会看到：

```
[root@lvs1 tmp]# ip address list
1: lo: <LOOPBACK,UP> mtu 3924 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 brd 127.255.255.255 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
    link/ether 00:00:5e:00:01:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.5/24 brd 192.168.200.255 scope global eth0
    inet 192.168.200.10/32 scope global eth0
    inet 192.168.200.11/32 scope global eth0
3: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
    link/ether 00:00:5e:00:01:32 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.5/24 brd 192.168.201.255 scope global eth1
    inet 192.168.100.10/32 scope global eth1
```

Symmetrically on LVS director 2 you will see:

> 在 LVS director 2 上对称地看到：

```
[root@lvs2 tmp]# ip address list
1: lo: <LOOPBACK,UP> mtu 3924 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 brd 127.255.255.255 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
    link/ether 00:00:5e:00:01:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.5/24 brd 192.168.200.255 scope global eth0
    inet 192.168.200.12/32 scope global eth0
    inet 192.168.200.13/32 scope global eth0
3: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
    link/ether 00:00:5e:00:01:32 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.5/24 brd 192.168.201.255 scope global eth1
    inet 192.168.100.11/32 scope global eth1
```

The VRRP VIPs are:

- VIP1 = 192.168.200.10
- VIP2 = 192.168.200.11
- VIP3 = 192.168.200.12
- VIP4 = 192.168.200.13
- DIP1 = 192.168.100.10
- DIP2 = 192.168.100.11

The use of VRRP keyword "sync_instance" imply that we have defined a pair of MASTER VRRP Instance per LVS directors ó (VI_1,VI_2) & (VI_3,VI_4). This means that if eth0 on LVS director 1 fails then VI_1 enters the MASTER state on LVS director 2 so the MASTER Instance distribution on both directors will be: (VI_2) on director 1 & (VI_1,VI_3,VI_4) on director 2. We use "sync_instance" so VI_2 is forced to BACKUP the state on LVS director 1. The final VRRP MASTER instance distribution will be: (none) on LVS director 1 & (VI_1,VI_2,VI_3,VI_4) on LVS director 2. If eth0 on LVS director 1 became available the distribution will transition back to the initial state.

> 使用 VRRP 关键字“sync_instance”意味着我们为每个 LVS 导演定义了一对 MASTER VRRP 实例（VI_1，VI_2）和（VI_3，VI_4）。这意味着，如果 LVS 导演 1 上的 eth0 失败，则 VI_1 进入 LVS 导演 2 上的 MASTER 状态，因此两个导演上的 MASTER 实例分布将是：（VI_2）在导演 1 上，（VI_1，VI_3，VI_4）在导演 2 上。我们使用“sync_instance”，以便 VI_2 强制进入 LVS 导演 1 的备份状态。最终的 VRRP MASTER 实例分布将是：（无）在 LVS 导演 1 上，（VI_1，VI_2，VI_3，VI_4）在 LVS 导演 2 上。如果 LVS 导演 1 上的 eth0 可用，分布将恢复到初始状态。

For more details on this state transition please refer to the "Linux Virtual Server High Availability using VRRPv2" paper (available at [http://www.linux-vs.org/~acassen/](http://www.linux-vs.org/~acassen/)), which explains the implementation of this functionality.

> 详细了解该状态转换，请参考"Linux 虚拟服务器高可用性使用 VRRPv2"论文（可在[http://www.linux-vs.org/~acassen/]（http://www.linux-vs.org/~acassen/）上获取），该论文解释了该功能的实现。

Using this configuration both LVS directors are active at a time, thus sharing LVS directors for a global director. That way we introduce a virtual LVS director.

> 使用这种配置，两个 LVS 导演同时处于活动状态，从而为全局导演共享 LVS 导演。这样，我们就引入了虚拟 LVS 导演。

::: note
::: title
Note
:::

This VRRP configuration sample is an illustration for a high availability router (not LVS specific). It can be used for many more common/simple needs.

> 这个 VRRP 配置示例是一个高可用路由器（不是 LVS 特定的）的说明。它可以用于更多常见/简单的需求。
> :::
