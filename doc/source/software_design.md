---
tip: translate by openai@2023-06-20 08:26:12
title: Software Design
---

Keepalived is written in pure ANSI/ISO C. The software is articulated around a central I/O multiplexer that provides realtime networking design. The main design focus is to provide a homogenous modularity between all elements. This is why a core library was created to remove code duplication. The goal is to produce a safe and secure code, ensuring production robustness and stability.

> 保活（Keepalived）是用纯 ANSI/ISO C 编写的。该软件围绕一个中央 I/O 多路复用器构建，提供实时网络设计。主要设计重点是在所有元素之间提供同质化的模块化。这就是为什么要创建一个核心库来消除代码重复。目标是产生安全可靠的代码，确保生产稳健和稳定性。

To ensure robustness and stability, daemon is split into 4 distinct processes:

> 为了确保健壮性和稳定性，守护进程被分成 4 个不同的进程：

- A minimalistic parent process in charge with forked children process monitoring.

- Up to three child processes, one responsible for VRRP framework, one for healthchecking and IPVS configuration, and one for BFD.

> 最多三个子进程，一个负责 VRRP 框架，一个负责健康检查和 IPVS 配置，一个负责 BFD。

Each child process has its own scheduling I/O multiplexer, that way VRRP scheduling jitter is optimized since VRRP and BFD scheduling are more sensitive/critical than healthcheckers. This split design minimalizes for healthchecking the usage of foreign libraries and minimalizes its own action down to an idle mainloop in order to avoid malfunctions caused by itself.

> 每个子进程都有自己的调度 I/O 复用器，这样 VRRP 调度抖动就得到了优化，因为 VRRP 和 BFD 调度比健康检查更敏感/关键。这种分离式设计将健康检查的外部库使用降到最低，将自身行为最小化到空闲的主循环，以避免由自身引起的故障。

The parent process monitoring framework is called watchdog. If the parent process detects that a child has terminated it simply restarts child process:

> 监视父进程的框架称为看门狗。如果父进程检测到子进程终止，它会重新启动子进程。

```
PID         111     Keepalived  <-- Parent process monitoring children
            112     \_ Keepalived   <-- VRRP child
            113     \_ Keepalived   <-- Healthchecking child
            114     \_ Keepalived   <-- BFD child
```

# Kernel Components

Keepalived uses four Linux kernel components:

> Keepalived 使用四个 Linux 内核组件：

1. LVS Framework: Uses the getsockopt and setsockopt calls to get and set options on sockets.

> LVS 框架：使用 getsockopt 和 setsockopt 调用来获取和设置套接字上的选项。

2. Netfilter Framework: IPVS code that supports NAT and Masquerading.

> 网络过滤框架：支持 NAT 和伪装的 IPVS 代码。

3. Netlink Interface: Sets and removes VRRP virtual IPs on network interfaces.

> 设置和删除网络接口上的 VRRP 虚拟 IP：Netlink 接口。

4. Multicast: VRRP advertisements are sent to the reserved VRRP MULTICAST group (224.0.0.18).

> 4. 多播：VRRP 广播发送到保留的 VRRP 多播组（224.0.0.18）。

# Atomic Elements

![keepalived software design](images/software_design.png){.align-center}

## Control Plane

Keepalived configuration is done through the file keepalived.conf. A compiler design is used for parsing. Parser work with a keyword tree hierarchy for mapping each configuration keyword with specifics handler. A central multi-level recursive function reads the configuration file and traverses the keyword tree. During parsing, configuration file is translated into an internal memory representation.

> 配置 Keepalived 通过 keepalived.conf 文件完成。采用编译器设计来解析。解析器使用关键字树层次结构来映射每个配置关键字的特定处理程序。中央多级递归函数读取配置文件并遍历关键字树。在解析过程中，配置文件被转换为内部存储表示。

## Scheduler - I/O Multiplexer

For each process, all the events are scheduled into the same process. Keepalived is network routing software, it is so close to I/O. The design used here is a central epoll_wait(\...) that is in charge of scheduling all internal tasks. POSIX thread libs are NOT used. This framework provides its own thread abstraction optimized for networking purpose.

> 对于每个进程，所有事件都被安排到同一个进程中。Keepalived 是网络路由软件，它离 I/O 很近。在这里使用的设计是一个负责调度所有内部任务的中央 epoll_wait（\...）。不使用 POSIX 线程库。该框架提供了自己的线程抽象，针对网络目的进行了优化。

## Memory Management

This framework provides access to some generic memory management functions like allocation, reallocation, release,\... This framework can be used in two modes: normal_mode & debug_mode. When using debug_mode it provides a strong way to eradicate and track memory leaks. This low-level env provides buffer under-run protection by tracking allocation and release of memory. All the buffers used are length fixed to prevent against eventual buffer-overflow.

> 这个框架提供访问一些通用的内存管理功能，如分配、重新分配、释放等。它可以在正常模式和调试模式下使用。当使用调试模式时，它可以有效地消除和跟踪内存泄漏。这个低级环境通过跟踪内存的分配和释放来提供缓冲区溢出保护。所有使用的缓冲区大小都是固定的，以防止最终的缓冲区溢出。

## Core Components

This framework defines some common and global libraries that are used in all the code. Those libraries are html parsing, link-list, timer, vector, string formating, buffer dump, networking utils, daemon management, pid handling, low-level TCP layer4. The goal here is to factorize code to the max to limit as much as possible code duplication to increase modularity.

> 这个框架定义了一些常见的和全局的库，它们被用于所有的代码。这些库包括 html 解析、链接列表、定时器、向量、字符串格式化、缓冲区转储、网络实用程序、守护管理、pid 处理和低层 TCP 四层。这里的目标是尽可能多地因子化代码，以最大限度地减少代码重复，以提高模块化。

## Checkers

This is one of the main Keepalived functionality. Checkers are in charge of adding, removing and changing the weight of realservers. There are several types of checkers, most of which relate to realserver healthchecking. A checker tests if realserver is alive, this test either ends on a binary decision: remove or add realserver from/into the LVS topology, or changing the weight of the realserver. The internal checker design is realtime networking software, it uses a fully multi-threaded FSM design (Finite State Machine). This checker stack provides LVS topology manipulation according to layer4 to layer5/7 test results. It\'s run in an independent process monitored by the parent process.

> 这是 Keepalived 的主要功能之一。检查器负责添加、删除和更改实际服务器的权重。有几种类型的检查器，其中大多数与实际服务器健康检查有关。检查器测试实际服务器是否存活，该测试以二进制决定结束：从/进入 LVS 拓扑中移除或添加实际服务器，或更改实际服务器的权重。内部检查器设计是实时网络软件，它使用完全多线程 FSM 设计（有限状态机）。该检查器堆栈根据四层到五/七层测试结果来提供 LVS 拓扑操作。它由父进程监视的独立进程运行。

## VRRP Stack

The other most important Keepalived functionality. VRRP (Virtual Router Redundancy Protocol: RFC2338/RFC3768/RFC5798) is focused on director takeover, it provides low-level design for router backup. It implements full IETF RFC5798 standard with some provisions and extensions for LVS and Firewall design (with legacy support for RFC2338, i.e. authentication). It implements the vrrp_sync_group extension that guarantees persistence routing path after protocol takeover. It implements IPSEC-AH using MD5-96bit crypto provision for securing protocol adverts exchange. For more information on VRRP please read the RFC. Important things: VRRP code can be used without the LVS support, it has been designed for independent use. It\'s run in an independent process monitored by parent process.

> 其他最重要的 Keepalived 功能是 VRRP（虚拟路由器冗余协议：RFC2338/RFC3768/RFC5798），它专注于导演接管，为路由器备份提供了低级设计。它实现了完整的 IETF RFC5798 标准，并为 LVS 和防火墙设计提供了一些规定和扩展（带有对 RFC2338 的遗留支持，即身份验证）。它实现了 vrrp_sync_group 扩展，可以在协议接管后保证路由路径的持久性。它使用 MD5-96 位加密提供程序实现 IPSEC-AH，以保护协议广告交换。有关 VRRP 的更多信息，请阅读 RFC。重要的是：VRRP 代码可以在没有 LVS 支持的情况下使用，它是专门为独立使用而设计的。它在由父进程监视的独立进程中运行。

## BFD Stack

An implementation of BFD (Bidirectional Forwarding and Detection: RFC5880). This can be used by both the VRRP process as a tracker for VRRP instance(s) and by the checker process as a checker for realserver. It\'s run in an independent process monitored by parent process.

> 一个 BFD（双向转发和检测：RFC5880）的实现。它可以被 VRRP 进程用作 VRRP 实例的跟踪器，也可以被检查进程用作 realserver 的检查器。它是由父进程监控的独立进程运行的。

## System Call

This framework offers the ability to launch extra system script. It is mainly used in the MISC checker. In VRRP framework it provides the ability to launch extra script during protocol state transition. The system call is done into a forked process to not pertube the global scheduling timer.

> 这个框架提供了启动额外系统脚本的能力。它主要用于 MISC 检查器。在 VRRP 框架中，它提供了在协议状态转换期间启动额外脚本的能力。系统调用是在一个分叉进程中完成的，以不扰乱全局调度定时器。

## Netlink Reflector

Same as IPVS wrapper. Keepalived works with its own network interface representation. IP address and interface flags are set and monitored through kernel Netlink channel. The Netlink messaging sub-system is used for setting VRRP VIPs. On the other hand, the Netlink kernel messaging broadcast capability is used to reflect into our userspace Keepalived internal data representation any events related to interfaces. So any other userspace (others program) netlink manipulation is reflected our Keepalived data representation via Netlink Kernel broadcast (RTMGRP_LINK & RTMGRP_IPV4_IFADDR).

> Keepalived 使用自己的网络接口表示法，与 IPVS 封装相同。IP 地址和接口标志通过内核 Netlink 通道设置和监视。Netlink 消息子系统用于设置 VRRP VIPs。另一方面，Netlink 内核消息广播功能用于反映我们的用户空间 Keepalived 内部数据表示中与接口相关的任何事件。因此，任何其他用户空间（其他程序）的 Netlink 操作都会通过 Netlink 内核广播（RTMGRP_LINK＆RTMGRP_IPV4_IFADDR）反映到我们的 Keepalived 数据表示中。

## SMTP

The SMTP protocol is used for administration notification. It implements the IETF RFC821 using a multi-threaded FSM design. Administration notifications are sent for healthcheckers activities and VRRP protocol state transition. SMTP is commonly used and can be interfaced with any other notification sub-system such as GSM-SMS, pagers, etc.

> SMTP 协议用于管理通知。它使用多线程 FSM 设计实现 IETF RFC821。管理通知用于健康检查器活动和 VRRP 协议状态转换。SMTP 通常被使用，可以与其他通知子系统（如 GSM-SMS、寻呼机等）进行接口。

## IPVS Wrapper

This framework is used for sending rules to the Kernel IPVS code. It provides translation between Keepalived internal data representation and IPVS rule_user representation. It uses the IPVS libipvs to keep generic integration with IPVS code.

> 这个框架用于向内核 IPVS 代码发送规则。它提供了 Keepalived 内部数据表示和 IPVS rule_user 表示之间的转换。它使用 IPVS libipvs 来保持与 IPVS 代码的通用集成。

## IPVS

The Linux Kernel code provided by Wensong from LinuxVirtualServer.org OpenSource Project. IPVS (IP Virtual Server) implements transport-layer load balancing inside the Linux kernel, also referred to as Layer-4 switching.

> 温松从 LinuxVirtualServer.org 开源项目提供的 Linux 内核代码。IPVS（IP 虚拟服务器）在 Linux 内核中实现了传输层负载均衡，也称为四层交换。

## NETLINK

The Linux Kernel code provided by Alexey Kuznetov with its very nice advanced routing framework and sub-system capabilities. Netlink is used to transfer information between kernel and user-space processes. It consists of a standard sockets-based interface for userspace processes and an internal kernel API for kernel modules.

> 阿列克西·库兹尼科夫提供的 Linux 内核代码具有非常出色的高级路由框架和子系统功能。Netlink 用于在内核和用户空间进程之间传输信息。它由用户空间进程的标准套接字接口和内核模块的内部内核 API 组成。

## Syslog

All keepalived daemon notification messages are logged using the syslog service.

> 所有 keepalived 守护进程通知消息都使用 syslog 服务记录。

# Healthcheck Framework

Each health check is registered to the global scheduling framework. These health check worker threads implement the following types of health checks:

> 每次健康检查都会被注册到全局调度框架中。这些健康检查工作线程实现以下类型的健康检查：

::: glossary

TCP_CHECK

: Working at layer4. To ensure this check, we use a TCP Vanilla check using nonblocking/timed-out TCP connections. If the remote server does not reply to this request (timed-out), then the test is wrong and the server is removed from the server pool.

HTTP_GET

: Working at layer5. Performs a HTTP GET to a specified URL. The HTTP GET result is then summed using the MD5 algorithm. If this sum does not match with the expected value, the test is wrong and the server is removed from the server pool. This module implements a multi-URL get check on the same service. This functionality is useful if you are using a server hosting more than one application servers. This functionality gives you the ability to check if an application server is working properly. The MD5 digests are generated using the genhash utility (included in the keepalived package).

SSL_GET

: Same as HTTP_GET but uses a SSL connection to the remote webservers.

MISC_CHECK

: This check allows a user-defined script to be run as the health checker. The result must be 0 or 1. The script is run on the director box and this is an ideal way to test in-house applications. Scripts that can be run without arguments can be called using the full path (i.e. /path_to_script/script.sh). Those requiring arguments need to be enclosed in double quotes (i.e. "/path_to_script/script.sh arg 1 \... arg n ")

SMTP_CHECK

: This check ensures that an SMTP server can be connected to and the initial SMTP handshake completed.

DNS_CHECK

: This check queries a DNS server for the configured name of the specified type (e.g. A, AAAA, MX record).

BFD_CHECK

: This is updated by the BFD process, and allows a realserver to be removed if the BFD session goes down.

UDP_CHECK

: This check sends a UDP packet to the specified remote host/port. It can be configured to require a specific response, or to fail if an ICMP error is returned.

PING_CHECK

: This check sends and ICMP echo request and will fail if an appropriate ICMP echo response is not received.

FILE_CHECK

: This check monitors a file using inotify(). If the file is modified or created, its contents are read and interpreted as a numeric value. This can either indicate the realserver should be removed, or its weight changed, depending on the configuration.
:::

The goal for Keepalived is to define a generic framework easily extensible for adding new checkers modules. If you are interested in the development of existing or new checkers, have a look at the _keepalived/check_ and _keepalived/trackers_ directories in the source:

> Keepalived 的目标是定义一个易于扩展以添加新检查器模块的通用框架。如果您有兴趣开发现有或新的检查器，请查看源中的*keepalived/check*和*keepalived/trackers*目录。

[https://github.com/acassen/keepalived/tree/master/keepalived/check](https://github.com/acassen/keepalived/tree/master/keepalived/check)

> [https://github.com/acassen/keepalived/tree/master/keepalived/check](https://github.com/acassen/keepalived/tree/master/keepalived/check) 的简体中文翻译：Keepalived 检查

# Failover (VRRP) Framework

Keepalived implements the VRRP protocol for director failover. Within the implemented VRRP stack, the VRRP Packet dispatcher is responsible for demultiplexing specific I/O for each VRRP instance.

> Keepalived 实现了 VRRP 协议以实现导演故障转移。在实现的 VRRP 堆栈中，VRRP 数据包调度程序负责为每个 VRRP 实例解复用特定的 I/O。

From RFC5798, VRRP is defined as:

> 根据 RFC5798，VRRP 定义为：

```
“VRRP specifies an election protocol that dynamically assigns
responsibility for a virtual router to one of the VRRP routers on a LAN.
The VRRP router controlling the IPv4 or IPv6 address(es) associated with
a virtual router is called the Master, and it forwards packets sent to
these IPv4 or IPv6 addresses.  VRRP Master routers are configured with
virtual IPv4 or IPv6 addresses, and VRRP Backup routers infer the
address family of the virtual addresses being carried based on the
transport protocol.  Within a VRRP router, the virtual routers in
each of the IPv4 and IPv6 address families are a domain unto
themselves and do not overlap.  The election process provides dynamic
failover in the forwarding responsibility should the Master become
unavailable.  For IPv4, the advantage gained from using VRRP is a
higher-availability default path without requiring configuration of
dynamic routing or router discovery protocols on every end-host.  For
IPv6, the advantage gained from using VRRP for IPv6 is a quicker
switchover to Backup routers than can be obtained with standard IPv6
Neighbor Discovery mechanisms.” [rfc5798]
```

::: note
::: title
Note
:::

This framework is LVS independent, so you can use it for LVS director failover, even for other Linux routers needing a Hot-Standby protocol. This framework has been completely integrated in the Keepalived daemon for design & robustness reasons.

> 这个框架是独立于 LVS 的，因此您可以将其用于 LVS 管理器故障转移，甚至用于其他需要热备协议的 Linux 路由器。出于设计和稳健性的原因，这个框架已经完全集成到 Keepalived 守护程序中。
> :::

The main functionalities provided by this framework are:

> 此框架提供的主要功能有：

- Failover: The native VRRP protocol purpose, based on a roaming set of VRRP VIPs.

- VRRP Instance synchronization: We can specify a state monitoring between 2 or more VRRP Instances, also known as a _VRRP sync group_. It guarantees that the VRRP Instances remain in the same state. The synchronized instances monitor each other.

> 可以在两个或更多的 VRRP 实例之间指定状态监测，也称为 VRRP 同步组。它保证 VRRP 实例保持相同的状态。同步的实例彼此监控。

- Nice Fallback
- Advert Packet integrity: Using IPSEC-AH ICV.
- System call: During a VRRP state transition, an external script/program may be called.

## Note on Using VRRP with Virtual MAC Address

To reduce takeover impact, some networking environment would require using VRRP with VMAC address. To reach that goal Keepalived VRRP framework implements VMAC support by the invocation of \'use_vmac\' keyword in configuration file.

> 为了减少接管的影响，一些网络环境需要使用 VRRP 和 VMAC 地址。为了达到这一目标，Keepalived VRRP 框架通过在配置文件中调用'use_vmac'关键字来实现 VMAC 支持。

Internally, Keepalived code will bring up virtual interfaces, each interface dedicated to a specific virtual_router. Keepalived uses Linux kernel macvlan driver to defines these interfaces. It is then mandatory to use kernel compiled with macvlan support.

> 内部，Keepalived 代码会启动虚拟接口，每个接口专用于特定的虚拟路由器。Keepalived 使用 Linux 内核 macvlan 驱动程序来定义这些接口。因此，必须使用带有 macvlan 支持的内核进行编译。

By default MACVLAN interface are in VEPA mode which filters out received packets whose MAC source address matches that of the MACVLAN interface. Setting MACVLAN interface in private mode will not filter based on source MAC address.

> 默认情况下，MACVLAN 接口处于 VEPA 模式，过滤掉接收到的源 MAC 地址与 MACVLAN 接口相匹配的数据包。将 MACVLAN 接口设置为私有模式时，将不再基于源 MAC 地址进行过滤。

Alternatively, you can specify \'vmac_xmit_base\' which will cause the VRRP messages to be transmitted and received on the underlying interface whilst ARP will happen from the VMAC interface.

> 另外，您可以指定'vmac_xmit_base'，这将导致 VRRP 消息在基础接口上发送和接收，而 ARP 将从 VMAC 接口发生。

You may also need to tweak your physical interfaces to play around with well known ARP issues. Keepalived sets the following configuration when using VMACs:

> 您可能还需要调整物理接口以玩弄众所周知的 ARP 问题。 使用 VMAC 时，Keepalived 设置以下配置：

1. Global configuration:

> 1. 全局配置：

```

net.ipv4.conf.all.arp_ignore = 1

> net.ipv4.conf.all.arp_ignore = 1（所有）忽略ARP请求

net.ipv4.conf.all.arp_announce = 1

> net.ipv4.conf.all.arp_announce = 1（简体中文）：net.ipv4.conf.all.arp_announce = 1

net.ipv4.conf.all.arp_filter = 0

> net.ipv4.conf.all.arp_filter = 0（简体中文）：net.ipv4.conf.all.arp_filter = 0
```

2. Physical interface configuration

> 2. 物理接口配置

For the physical ethernet interface running VRRP instance use:

> 对于运行 VRRP 实例的物理以太网接口使用：

```
net.ipv4.conf.eth0.arp_filter = 1
```

3. VMAC interface

consider the following VRRP configuration:

> 考虑以下 VRRP 配置：

```
vrrp_instance instance1 {
    state BACKUP
    interface eth0
    virtual_router_id 250
    use_vmac
        vmac_xmit_base         # Transmit VRRP adverts over physical interface
    priority 150
    advert_int 1
    virtual_ipaddress {
        10.0.0.254
    }
}
```

The `use_vmac` keyword will drive keepalived code to create a macvlan interface named _vrrp.250_ (default internal paradigm is vrrp.{virtual_router_id}, you can override this naming by giving an argument to \'use_vmac\' keyword, eg: use_vmac vrrp250).

> 使用`use_vmac`关键字将会推动 keepalived 代码创建一个名为*vrrp.250*的 Macvlan 接口（默认内部范式是 vrrp.{virtual_router_id}，您可以通过给'use_vmac'关键字提供一个参数来覆盖这个命名，例如：use_vmac vrrp250）。
