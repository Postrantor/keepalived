---
tip: translate by openai@2023-06-20 08:17:50
title: Installing Keepalived
---

Install keepalived from the distribution\'s repositories or, alternatively, compile from source. Although installing from the repositories is generally the fastest way to get keepalived running on a system, the version of keepalived available in the repositories are typically a few releases behind the latest available stable version.

> 安装 keepalived 可以从发行版的仓库中安装，或者从源代码编译。虽然从仓库安装通常是让系统上的 keepalived 快速运行的最快方法，但仓库中可用的 keepalived 版本通常比最新的稳定版本要旧几个版本。

# Installing from the Repositories

## Installing on Red Hat Enterprise Linux

As of Red Hat 6.4, Red Hat and the clones have included the keepalived package in the base repository. Therefore, run the following to install the keepalived package and all the required dependencies using dnf (or yum on older systems):

> 从 Red Hat 6.4 开始，Red Hat 和克隆版本已将 keepalived 包包含在基础存储库中。因此，使用 dnf（或旧系统上的 yum）运行以下命令安装 keepalived 包及所有所需的依赖项：

```
dnf install keepalived
```

## Installing on Debian

Run the following to install the keepalived package and all the required dependencies using Debian\'s APT package handling utility:

> 运行以下命令，使用 Debian 的 APT 包管理实用程序安装 keepalived 包及所有所需的依赖项：

```
apt-get install keepalived
```

# Compiling and Building from Source

In order to run the latest stable version, compile keepalived from source. Compiling keepalived requires a compiler, OpenSSL and the Netlink Library. You may optionally install Net-SNMP, which is required for SNMP support.

> 为了运行最新的稳定版本，需要从源码编译 keepalived。编译 keepalived 需要编译器，OpenSSL 和 Netlink 库。您可以选择安装 Net-SNMP，这是 SNMP 支持所必需的。

## Install Prerequisites on RHEL/CentOS/Fedora

On RHEL, Centos, Fedora etc install the following prerequisites (on older systems replace dnf with yum):

> 在 RHEL、Centos、Fedora 等系统上安装以下先决条件（在较旧的系统上用 yum 替换 dnf）：

```
dnf install gcc make autoconf automake openssl-devel libnl3-devel \
    iptables-devel ipset-devel net-snmp-devel libnfnetlink-devel file-devel \
    glib2-devel pcre2-revel libnftnl-devel libmnl-devel systemd-devel kmod-devel
```

For DBUS support:

```
dnf install glib2-devel
```

For JSON support:

```
dnf install json-c-devel
```

Note: On RHEL the codeready-builder-for-rhel-8-x86_64-rpms (or equivalent) repo

: needs to be enabled, and on CentOS the PowerTools repo is needed.

## Install Prerequisites on Debian/Ubuntu

On Debian/Ubuntu, install the following prerequisites:

> 在 Debian/Ubuntu 上安装以下先决条件：

```
apt-get install build-essential pkg-config curl gcc autoconf automake libssl-dev \
    libnl-3-dev libnl-genl-3-dev libsnmp-dev libnl-route-3-dev libnfnetlink-dev \
    iptables-dev* libipset-dev libsnmp-dev libmagic-dev libglib2.0-dev libpcre2-dev \
    libnftnl-dev libmnl-dev libsystemd-dev libkmod-dev

* on more recent versions replace iptables-dev with libxtables-dev libip4tc-dev libip6tc-dev
```

For DBUS support:

```
dnf install libglib2.0-dev
```

## Install Prerequisites on Alpine Linux

On Alpine Linux install the following prerequisites:

> 在 Alpine Linux 上安装以下先决条件：

```
autoconf automake iptables-dev ipset-dev libnfnetlink-dev libnl3-dev musl-dev
    libnftnl-dev file-dev pcre2-dev
  and
    openssl-dev or libressl-dev
```

For SNMP support:

```
net-snmp-dev (requires libressl-dev and not openssl-dev)
```

## Install Prerequisites on Archlinux

On Archlinux run the following to install the required libraries:

> 在 Archlinux 上运行以下命令安装所需的库：

```
pacman -S ipset libnfnetlink libnl1 pcre-2
```

For SNMP support:

```
pacman -S net-snmp
```

## Build and Install

Use _curl_ or any other transfer tool such as _wget_ to download keepalived. The software is available at [http://www.keepalived.org/download.html](http://www.keepalived.org/download.html) or [https://github.com/acassen/keepalived](https://github.com/acassen/keepalived). Then, compile the package:

> 使用 curl 或其他传输工具，如 wget，下载 keepalived。该软件可在[http://www.keepalived.org/download.html](http://www.keepalived.org/download.html) 或 [https://github.com/acassen/keepalived](https://github.com/acassen/keepalived)处获得。然后，编译该包：

```
curl --location --progress http://keepalived.org/software/keepalived-1.2.15.tar.gz | tar xz
cd keepalived-1.2.15
./build_setup
./configure
make
sudo make install
```

It is a general recommendation when compiling from source to specify a PREFIX. For example:

> 建议编译源码时指定 PREFIX。例如：

```
./configure --prefix=/usr/local/keepalived-1.2.15
```

This makes it easy to uninstall a compiled version of keepalived simply by deleting the parent directory. Additionally, this method of installation allows for multiple versions of Keepalived installed without overwriting each other. Use a symlink to point to the desired version. For example, your directory layout could look like this:

> 这使得只需删除父目录就可以轻松卸载编译版本的 keepalived。此外，此安装方法允许安装多个版本的 Keepalived 而不会互相覆盖。使用符号链接指向所需的版本。例如，您的目录布局可能如下：

```
[root@lvs1 ~]# cd /usr/local
[root@lvs1 local]# ls -l
total 12
lrwxrwxrwx. 1 root root   17 Feb 24 20:23 keepalived -> keepalived-1.2.15
drwxr-xr-x. 2 root root 4096 Feb 24 20:22 keepalived-1.2.13
drwxr-xr-x. 2 root root 4096 Feb 24 20:22 keepalived-1.2.14
drwxr-xr-x. 2 root root 4096 Feb 24 20:22 keepalived-1.2.15
```

## Setup Init Scripts

After compiling, create an init script in order to control the keepalived daemon.

> 编译完成后，创建一个初始脚本以控制 keepalived 守护进程。

On RHEL:

```
ln -s /etc/rc.d/init.d/keepalived.init /etc/rc.d/rc3.d/S99keepalived
```

On Debian:

```
ln -s /etc/init.d/keepalived.init /etc/rc2.d/S99keepalived
```

Note: The link should be added in your default run level directory.
