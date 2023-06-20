---
tip: translate by openai@2023-06-20 08:24:35
title: Configuring SNMP Support
---

Keepalived provides an SNMP subsystem that can gather various metrics about the VRRP stack and the health checker system. The keepalived MIB is in the `doc` directory of the project. The base SNMP OID for the MIB is [.1.3.6.1.4.1.9586.100.5]{.title-ref}, which is hosted under the [Debian OID space](https://dsa.debian.org/iana/) assigned by IANA.

> Keepalived 提供了一个 SNMP 子系统，可以收集有关 VRRP 堆栈和健康检查系统的各种指标。Keepalived MIB 位于项目的`doc`目录中。MIB 的基本 SNMP OID 为[.1.3.6.1.4.1.9586.100.5]{.title-ref}，由 IANA 分配的 Debian OID 空间([Debian OID 空间](https://dsa.debian.org/iana/))下提供。

Prerequisites \*\*\*\*\*\*\*\*\*\*\*\*\*

> 前提条件

Install the SNMP protocol tools and libraries onto your system. This requires the installation of a few packages:

> 在你的系统上安装 SNMP 协议工具和库。这需要安装几个软件包：

```
yum install net-snmp net-snmp-utils net-snmp-libs
```

Once SNMP has been installed on your system, configure keepalived with SNMP support. When compiling keepalived, add the `--enable-snmp` configure option. For example:

> 一旦 SNMP 已安装在您的系统上，请配置 keepalived，并带有 SNMP 支持。编译 keepalived 时，请添加`--enable-snmp`配置选项。例如：

```
./configure --enable-snmp
```

During the configure step of the compiling process, you will get a configuration summary before building with `make`. For example, you may see similar output on a CentOS 6 machine:

> 在编译过程的配置步骤中，在使用“make”编译之前，你会得到一个配置摘要。例如，在 CentOS 6 机器上，你可能会看到类似的输出：

```
./configure --prefix=/usr/local/keepalived-1.2.15 --enable-snmp
Keepalived configuration
------------------------
Keepalived version       : 1.2.15
Compiler                 : gcc
Compiler flags           : -g -O2 -I/usr/include/libnl3
Extra Lib                : -Wl,-z,relro -Wl,-z,now -L/usr/lib64
-lnetsnmpagent -lnetsnmphelpers -lnetsnmpmibs -lnetsnmp -Wl,-E
-Wl,-rpath,/usr/lib64/perl5/CORE -lssl -lcrypto -lcrypt  -lnl-genl-3 -lnl-3
Use IPVS Framework       : Yes
IPVS sync daemon support : Yes
IPVS use libnl           : Yes
fwmark socket support    : Yes
Use VRRP Framework       : Yes
Use VRRP VMAC            : Yes
SNMP support             : Yes
SHA1 support             : No
Use Debug flags          : No
```

Notice the _Extra Lib_ section of the configuration summary. It lists various library flags that gcc will use to build keepalived, several of which have to do with SNMP.

> 注意配置摘要中的*额外库*部分。它列出了 GCC 将用于构建 Keepalived 的各种库标志，其中有几个与 SNMP 有关。

# Configuring Support

Enable SNMP AgentX support by including the following line in the SNMP daemon configuration file, typically `/etc/snmp/snmpd.conf` if you installed via RPMs on a CentOS machine:

> 在 CentOS 机器上通过 RPM 安装时，在 SNMP 守护程序配置文件（通常为`/etc/snmp/snmpd.conf`）中包含以下行以启用 SNMP AgentX 支持：

```
master agentx
```

::: note
::: title
Note
:::

Be sure to reload or restart the SNMP service for the configuration change to take effect.

> 确保重新加载或重新启动 SNMP 服务，以使配置更改生效。
> :::

# Adding the MIB

You can query keepalived SNMP managed objects by using the OID. For example:

> 你可以使用 OID 查询 keepalived SNMP 管理的对象。例如：

```
snmpwalk -v2c -c public localhost .1.3.6.1.4.1.9586.100.5.1.1.0
SNMPv2-SMI::enterprises.9586.100.5.1.1.0 = STRING: "Keepalived v1.2.15 (01/10,2015)"
```

Alternatively, with the keepalived MIB, you can query using the MIB available from the project. First, copy the MIB to the system\'s global MIB directory or to the user\'s local MIB directory:

> 另外，可以使用 keepalived MIB 从项目中查询。首先，将 MIB 复制到系统的全局 MIB 目录或用户的本地 MIB 目录：

```
cp /usr/local/src/keepalived-1.2.15/doc/KEEPALIVED-MIB /usr/share/snmp/mibs
```

or:

```
cp /usr/local/src/keepalived-1.2.15/doc/KEEPALIVED-MIB ~/.snmp/mibs
```

The SNMP daemon will check both directories for the existence of the MIB. Once the MIB is in place, the SNMP query can look as follows:

> SNMP 守护进程将检查两个目录中是否存在 MIB。一旦 MIB 到位，SNMP 查询可以如下所示：

```
snmpwalk -v2c -c public localhost KEEPALIVED-MIB::version
KEEPALIVED-MIB::version.0 = STRING: Keepalived v1.2.15 (01/10,2015)
```

# MIB Overview

There are four main sections to the keepalived MIB:

> 有四个主要部分组成 keepalived MIB：

- global
- vrrp
- check
- conformance

## Global

The global section includes objects that contain information about the keepalived instance such as version, router ID and administrative email addresses.

> 全局部分包括包含有关 keepalived 实例的信息的对象，例如版本，路由器 ID 和管理电子邮件地址。

## VRRP

The VRRP section includes objects that contain information about each configured VRRP instance. Within each instance, there are objects that include instance name, current state, and virtual IP addresses.

> VRRP 部分包含了关于每一个配置的 VRRP 实例的信息对象。在每一个实例中，有实例名称、当前状态和虚拟 IP 地址的对象。

## Check

The Check section includes objects that contain information about each configured virtual server. It includes server tables for virtual and real servers and also configured load balancing algorithms, load balancing method, protocol, status, real and virtual server network connection statistics.

> 该检查部分包含有关每个配置的虚拟服务器的信息的对象。它包括虚拟和实际服务器的服务器表，以及配置的负载均衡算法、负载均衡方法、协议、状态、实际和虚拟服务器网络连接统计信息。

## Conformance

::: todo
do conformance
:::

::: note
::: title
Note
:::

Use a MIB browser, such as mbrowse, to see what managed objects are available to query for monitoring the health of your LVS servers.

> 使用 MIB 浏览器，例如 mbrowse，查看可用于查询以监控 LVS 服务器健康状况的管理对象。
> :::
