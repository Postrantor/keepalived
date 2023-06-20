---
tip: translate by openai@2023-06-20 08:13:48
title: Keepalived configuration synopsis
---

The Keepalived configuration file uses the following synopsis (configuration keywords are Bold/Italic):

> Keepalived 配置文件使用以下概要（配置关键字用粗体/斜体表示）：

# Global Definitions Synopsis

::: parsed-literal

**global_defs** {

:

```
**notification_email** {

:   email email

} **notification_email_from** email **smtp_server** host **smtp_connect_timeout** num **router_id** string
```

}
:::

Keyword Definition Type

---

global_defs identify the global def configuration block

> 全局定义识别全局定义配置块

notification_email email accounts that will receive the notification mail List

> 通知邮件账户列表：将收到通知邮件的邮箱账户列表

notification_email_from email to use when processing "MAIL FROM:" SMTP command List

> 使用处理"MAIL FROM:"SMTP 命令时要使用的通知电子邮件发件人邮箱列表

smtp_server remote SMTP server to use for sending mail notifications alphanum

> 远程 SMTP 服务器用于发送邮件通知的字母数字

smtp_connect_timeout specify a timeout for SMTP stream processing numerical

> 指定 SMTP 流处理的超时时间，以数字表示

router_id specify the name of the LVS director string

> 路由器 ID 指定 LVS 导演字符串的名称

Email type: Is a string using charset as specified into the SMTP RFC eg: "<user@domain.com>"

> 电子邮件类型：是一个使用 SMTP RFC 中指定的字符集的字符串，例如：“<user@domain.com>”

# Virtual Server Definitions Synopsis

::: parsed-literal

**virtual_server** (@IP PORT)\|(_fwmark_ num) {

> **虚拟服务器**（@IP 端口）|（_fwmark_ num）{

: **delay_loop** num **lb_algo** _rr\|wrr\|lc\|wlc\|sh\|dh\|lblc_ **lb_kind** _NAT\|DR\|TUN_ **(nat_mask** \@IP) **persistence_timeout** num **persistence_granularity** \@IP **virtualhost** string **protocol** _TCP\|UDP_

```
**sorry_server** \@IP PORT **real_server** \@IP PORT { **weight** num **TCP_CHECK** { **connect_port** num **connect_timeout** num } } **real_server** \@IP PORT { **weight** num **MISC_CHECK** { **misc_path** /path_to_script/script.sh (or **misc_path** " /path_to_script/script.sh \<arg_list\>") } }
```

} **real_server** \@IP PORT { **weight** num **HTTP_GET\|SSL_GET** { **url** { \# You can add multiple url block **path** alphanum **digest** alphanum } **connect_port** num **connect_timeout** num **retry** num **delay_before_retry** num } }

> 真实服务器@IP 端口 {权重数字 HTTP_GET|SSL_GET { URL {#您可以添加多个 URL 块路径字母数字摘要字母数字} 连接端口数字连接超时数字重试次数数字重试前延迟数字} }
> :::

Keyword Definition Type

---

virtual_server fwmark identify a virtual server definition block specify that virtual server is a FWMARK

> 虚拟服务器 fwmark 标识一个虚拟服务器定义块，指定该虚拟服务器为 FWMARK。

delay_loop specify in seconds the interval between checks numerical

> 延迟循环指定检查之间的间隔，以秒为单位。

lb_algo select a specific scheduler (rrlc\|wlc\...) string

> 选择特定的调度器（rrlc\|wlc\...）字符串

lb_kind select a specific forwarding method (NATTUN) string

> 选择特定的转发方法（NATTUN）字符串

persistence_timeout persistence_granularity specify a timeout value for persistent connections specify a granularity mask for persistent connections numerical

> 持久连接的超时值指定持久连接的粒度掩码，用数字表示。

virtualhost specify a HTTP virtualhost to use for HTTP\|SSL_GET alphanum

> 指定一个 HTTP 虚拟主机以用于 HTTP\|SSL_GET alphanum。

protocol sorry_server real_server specify the protocol kind (TCP\|UDP) server to be added to the pool if all real servers are down specify a real server member numerical

> 协议 sorry_server real_server 指定协议类型（TCP \ | UDP）服务器，如果所有真实服务器都关闭，则添加到池中指定一个真实服务器成员数字。

weight TCP_CHECK MISC_CHECK specify the real server weight for load balancing decisions check real server availability using TCP connect check real server availability using user defined script numerical

> 重量 TCP_CHECK MISC_CHECK 为负载均衡决策指定真实服务器的权重。使用 TCP 连接检查真实服务器的可用性，使用用户定义的脚本检查真实服务器的可用性，数值。

misc_path HTTP_GET SSL_GET url identify the script to run with full path check real server availability using HTTP GET request check real server availability using SSL GET request identify a url definition block path

> 检查真实服务器可用性使用 HTTP GET 请求：misc_path HTTP_GET
> 检查真实服务器可用性使用 SSL GET 请求：SSL_GET
> 使用完整路径鉴定脚本：url
> 定义块路径：identify a url definition block path

path specify the url path alphanum

> 指定 URL 路径的路径是字母数字

digest specify the digest for a specific url path alphanum

> 指定特定 URL 路径的摘要（alphanum）

connect_port connect remote server on specified TCP port numerical

> 连接指定 TCP 端口号的远程服务器

connect_timeout connect remote server using timeout numerical

> 连接远程服务器时使用超时数字设定超时

retry maximum number of retries numerical

> 重试最大重试次数数值

delay_before_retry delay between two successive retries numerical

> 延迟重试：两次连续重试之间的延迟数字

::: note
::: title
Note
:::

The \"nat_mask\" keyword is obsolete if you are not using LVS with Linux kernel 2.2 series. This flag give you the ability to define the reverse NAT granularity.

> “nat_mask”关键字在您不使用 Linux 内核 2.2 系列的 LVS 时已经过时了。此标志可以让您定义反向 NAT 的粒度。
> :::

::: note
::: title
Note
:::

Currently, Healthcheck framework, only implements TCP protocol for service monitoring.

> 目前，Healthcheck 框架仅实现了用于服务监控的 TCP 协议。
> :::

::: note
::: title
Note
:::

Type \"path\" refers to the full path of the script being called. Note that for scripts requiring arguments the path and arguments must be enclosed in double quotes (\").

> 类型“路径”指的是被调用脚本的完整路径。注意，对于需要参数的脚本，路径和参数必须用双引号（“）括起来。
> :::

# VRRP Instance Definitions Synopsis

::: parsed-literal

**vrrp_sync_group** string {

> **VRRP 同步组**字符串{

:

```
**group** {

:   string string

} **notify_master** /path_to_script/script_master.sh (or **notify_master** " /path_to_script/script_master.sh \<arg_list\>") **notify_backup** /path_to_script/script_backup.sh (or **notify_backup** "/path_to_script/script_backup.sh \<arg_list\>") **notify_fault** /path_to_script/script_fault.sh (or **notify_fault** " /path_to_script/script_fault.sh \<arg_list\>")
```

} **vrrp_instance** string { **state** _MASTER\|BACKUP_ **interface** string **mcast_src_ip** \@IP **lvs_sync_daemon_interface** string **virtual_router_id** num **priority** num **advert_int** num **smtp_alert** **authentication** { **auth_type** _PASS\|AH_ **auth_pass** string } **virtual_ipaddress** { \# Block limited to 20 IP addresses \@IP \@IP \@IP } **virtual_ipaddress_excluded** { \# Unlimited IP addresses \@IP \@IP \@IP } **notify_master** /path_to_script/script_master.sh (or **notify_master** " /path_to_script/script_master.sh \<arg_list\>") **notify_backup** /path_to_script/script_backup.sh (or **notify_backup** " /path_to_script/script_backup.sh \<arg_list\>") **notify_fault** /path_to_script/script_fault.sh (or **notify_fault** " /path_to_script/script_fault.sh \<arg_list\>") }

> vrrp_instance：字符串状态：MASTER | BACKUP 接口：字符串 mcast_src_ip：@IP lvs_sync_daemon_interface：字符串 virtual_router_id：数字优先级：数字 advert_int：数字 smtp_alert：认证：{auth_type：PASS | AH auth_pass：字符串}virtual_ipaddress：{#块限制为 20 个 IP 地址@IP @IP @IP}virtual_ipaddress_excluded：{#无限 IP 地址@IP @IP @IP}notify_master：/path_to_script/script_master.sh（或 notify_master“/path_to_script/script_master.sh<arg_list>”）notify_backup：/path_to_script/script_backup.sh（或 notify_backup“/path_to_script/script_backup.sh<arg_list>”）notify_fault：/path_to_script/script_fault.sh（或 notify_fault“/path_to_script/script_fault.sh<arg_list>”）}
> :::

Keyword Definition Type

---

vrrp_instance state identify a VRRP instance definition block specify the instance state in standard use

> 实例状态：标识 VRRP 实例定义块，指定标准使用的实例状态。

Interface mcast_src_ip specify the network interface for the instance to run on specify the src IP address value for VRRP adverts IP header string

> 接口 mcast_src_ip 指定实例运行的网络接口，指定 VRRP 广告 IP 头字符串的源 IP 地址值。

lvs_sync_daemon_inteface specify the network interface for the LVS sync_daemon to run on string

> LVS_sync_daemon_inteface 指定 LVS sync_daemon 运行所使用的网络接口字符串。

virtual_router_id specify to which VRRP router id the instance belongs numerical

> 虚拟路由器 ID 指定实例属于哪个 VRRP 路由器 ID，数值。

priority specify the instance priority in the VRRP router numerical

> 指定 VRRP 路由器中实例优先级的数值

advert_int smtp_alert authentication auth_type specify the advertisement interval in seconds (set to 1) Activate the SMTP notification for MASTER state transition identify a VRRP authentication definition block specify which kind of authentication to use (PASS\|AH) numerical

> 设置广告间隔为秒（设置为 1）激活 MASTER 状态转换的 SMTP 通知，指定 VRRP 认证定义块，指定要使用的认证类型（PASS\|AH）数值。

auth_pass virtual_ipaddress virtual_ipaddress_excluded specify the password string to use identify a VRRP VIP definition block identify a VRRP VIP excluded definition block (not protocol VIPs) string

> 认证密码 virtual_ipaddress 指定要用来标识 VRRP VIP 定义块的密码字符串，virtual_ipaddress_excluded 标识 VRRP VIP 排除定义块（不是协议 VIPs）的字符串。

notify_master specify a shell script to be executed during transition to master state path

> 通知主人指定在转换到主状态路径时要执行的 shell 脚本

notify_backup specify a shell script to be executed during transition to backup state path

> 通知备份指定在过渡到备份状态路径时要执行的 shell 脚本

notify_fault specify a shell script to be executed during transition to fault state path

> 通知故障指定在转换到故障状态路径时要执行的 shell 脚本

vrrp_sync_group Identify the VRRP synchronization instances group string

> VRRP 同步组识别 VRRP 同步实例组字符串

Path type: A system path to a script eg: "/usr/local/bin/transit.sh \<arg_list\>"

> 路径类型：一个指向脚本的系统路径，例如：“/usr/local/bin/transit.sh \<arg_list\>”
