---
tip: translate by openai@2023-06-20 07:58:16
...

# keepalived: Loadbalancing & High-Availability

[![Keepalived CI](https://github.com/acassen/keepalived/actions/workflows/build.yml/badge.svg)](https://github.com/acassen/keepalived/actions/workflows/build.yml)
[![Coverity Status](https://scan.coverity.com/projects/22678/badge.svg)](https://scan.coverity.com/projects/acassen-keepalived)
[![Language grade: C/C++](https://img.shields.io/lgtm/grade/cpp/g/acassen/keepalived.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/acassen/keepalived/context:cpp)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/acassen/keepalived.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/acassen/keepalived/alerts/)
[![keepalived](https://snapcraft.io/keepalived/badge.svg)](https://snapcraft.io/keepalived)
[![Twitter Follow](https://img.shields.io/twitter/url/http/shields.io.svg?style=social&label=Follow)](https://twitter.com/keepalived)

The main goal of this project is to provide simple and robust facilities for loadbalancing and high-availability to Linux system and Linux based infrastructures. Loadbalancing framework relies on well-known and widely used Linux Virtual Server (IPVS) kernel module providing Layer4 loadbalancing. Keepalived implements a set of checkers to dynamically and adaptively maintain and manage loadbalanced server pool according their health. On the other hand high-availability is achieved by the Virtual Router Redundancy Protocol (VRRP). VRRP is a fundamental brick for router failover. In addition, Keepalived implements a set of hooks to the VRRP finite state machine providing low-level and high-speed protocol interactions. In order to offer fastest network failure detection, Keepalived implements the Bidirectional Forwarding Detection (BFD) protocol. VRRP state transition can take into account BFD hints to drive fast state transition. Keepalived frameworks can be used independently or all together to provide resilient infrastructures.

> 这个项目的主要目标是为 Linux 系统和基于 Linux 的基础架构提供简单而可靠的负载平衡和高可用性设施。负载平衡框架依赖于众所周知且广泛使用的 Linux 虚拟服务器（IPVS）内核模块，提供第四层负载平衡。Keepalived 实现了一组检查器，可以动态和自适应地维护和管理负载平衡服务器池，其根据健康状况。另一方面，通过虚拟路由器冗余协议（VRRP）实现高可用性。VRRP 是路由器故障转移的基础砖块。此外，Keepalived 实现了一组钩子，用于 VRRP 有限状态机，以提供低级别和高速协议交互。为了提供最快的网络故障检测，Keepalived 实现了双向转发检测（BFD）协议。VRRP 状态转换可以考虑 BFD 提示来驱动快速状态转换。Keepalived 框架可以独立使用，也可以一起使用，以提供弹性基础架构。

Keepalived implementation is based on an I/O multiplexer to handle a strong multi-threading framework. All the events process use this I/O multiplexer.

> 实现 Keepalived 基于 I/O 多路复用器来处理强大的多线程框架。所有事件处理都使用这个 I/O 多路复用器。

To build keepalived from the git source tree, you will need to have autoconf, automake and various libraries installed. See the INSTALL file for details of what needs to be installed and what needs to be executed before building keepalived.

> 要从 Git 源树构建 Keepalived，您需要安装 autoconf、automake 和各种库。有关在构建 Keepalived 之前需要安装和执行的内容，请参阅 INSTALL 文件。

Keepalived is free software, Copyright (C) Alexandre Cassen. See the file COPYING for copying conditions.

> Keepalived 是免费软件，版权所有 (C) Alexandre Cassen。请参阅 COPYING 文件以获取复制条件。

OPENSSL TOOLKIT LICENCE EXCEPTION

> 开放 SSL 工具包许可异常

In addition, as the copyright holder of Keepalived, I, Alexandre Cassen, <acassen@linux-vs.org>, grant the following special exception:

> 此外，作为 Keepalived 的版权持有人，我 Alexandre Cassen，<acassen@linux-vs.org>，授予以下特殊例外：

I, Alexandre Cassen, <acassen@linux-vs.org>, explicitly allow the compilation and distribution of the Keepalived software with the OpenSSL Toolkit.

> 我，Alexandre Cassen，<acassen@linux-vs.org>，明确允许使用 OpenSSL Toolkit 编译和分发 Keepalived 软件。
