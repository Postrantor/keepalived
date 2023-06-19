---
title: Keepalived configuration synopsis
---

The Keepalived configuration file uses the following synopsis (configuration keywords are Bold/Italic):

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
notification_email email accounts that will receive the notification mail List
notification_email_from email to use when processing "MAIL FROM:" SMTP command List
smtp_server remote SMTP server to use for sending mail notifications alphanum
smtp_connect_timeout specify a timeout for SMTP stream processing numerical
router_id specify the name of the LVS director string

Email type: Is a string using charset as specified into the SMTP RFC eg: "<user@domain.com>"

# Virtual Server Definitions Synopsis

::: parsed-literal

**virtual_server** (@IP PORT)\|(_fwmark_ num) {

: **delay_loop** num **lb_algo** _rr\|wrr\|lc\|wlc\|sh\|dh\|lblc_ **lb_kind** _NAT\|DR\|TUN_ **(nat_mask** \@IP) **persistence_timeout** num **persistence_granularity** \@IP **virtualhost** string **protocol** _TCP\|UDP_

```
**sorry_server** \@IP PORT **real_server** \@IP PORT { **weight** num **TCP_CHECK** { **connect_port** num **connect_timeout** num } } **real_server** \@IP PORT { **weight** num **MISC_CHECK** { **misc_path** /path_to_script/script.sh (or **misc_path** " /path_to_script/script.sh \<arg_list\>") } }
```

} **real_server** \@IP PORT { **weight** num **HTTP_GET\|SSL_GET** { **url** { \# You can add multiple url block **path** alphanum **digest** alphanum } **connect_port** num **connect_timeout** num **retry** num **delay_before_retry** num } }
:::

Keyword Definition Type

---

virtual_server fwmark identify a virtual server definition block specify that virtual server is a FWMARK
delay_loop specify in seconds the interval between checks numerical
lb_algo select a specific scheduler (rrlc\|wlc\...) string
lb_kind select a specific forwarding method (NATTUN) string
persistence_timeout persistence_granularity specify a timeout value for persistent connections specify a granularity mask for persistent connections numerical
virtualhost specify a HTTP virtualhost to use for HTTP\|SSL_GET alphanum
protocol sorry_server real_server specify the protocol kind (TCP\|UDP) server to be added to the pool if all real servers are down specify a real server member numerical
weight TCP_CHECK MISC_CHECK specify the real server weight for load balancing decisions check real server availability using TCP connect check real server availability using user defined script numerical
misc_path HTTP_GET SSL_GET url identify the script to run with full path check real server availability using HTTP GET request check real server availability using SSL GET request identify a url definition block path
path specify the url path alphanum
digest specify the digest for a specific url path alphanum
connect_port connect remote server on specified TCP port numerical
connect_timeout connect remote server using timeout numerical
retry maximum number of retries numerical
delay_before_retry delay between two successive retries numerical

::: note
::: title
Note
:::

The \"nat_mask\" keyword is obsolete if you are not using LVS with Linux kernel 2.2 series. This flag give you the ability to define the reverse NAT granularity.
:::

::: note
::: title
Note
:::

Currently, Healthcheck framework, only implements TCP protocol for service monitoring.
:::

::: note
::: title
Note
:::

Type \"path\" refers to the full path of the script being called. Note that for scripts requiring arguments the path and arguments must be enclosed in double quotes (\").
:::

# VRRP Instance Definitions Synopsis

::: parsed-literal

**vrrp_sync_group** string {

:

```
**group** {

:   string string

} **notify_master** /path_to_script/script_master.sh (or **notify_master** " /path_to_script/script_master.sh \<arg_list\>") **notify_backup** /path_to_script/script_backup.sh (or **notify_backup** "/path_to_script/script_backup.sh \<arg_list\>") **notify_fault** /path_to_script/script_fault.sh (or **notify_fault** " /path_to_script/script_fault.sh \<arg_list\>")
```

} **vrrp_instance** string { **state** _MASTER\|BACKUP_ **interface** string **mcast_src_ip** \@IP **lvs_sync_daemon_interface** string **virtual_router_id** num **priority** num **advert_int** num **smtp_alert** **authentication** { **auth_type** _PASS\|AH_ **auth_pass** string } **virtual_ipaddress** { \# Block limited to 20 IP addresses \@IP \@IP \@IP } **virtual_ipaddress_excluded** { \# Unlimited IP addresses \@IP \@IP \@IP } **notify_master** /path_to_script/script_master.sh (or **notify_master** " /path_to_script/script_master.sh \<arg_list\>") **notify_backup** /path_to_script/script_backup.sh (or **notify_backup** " /path_to_script/script_backup.sh \<arg_list\>") **notify_fault** /path_to_script/script_fault.sh (or **notify_fault** " /path_to_script/script_fault.sh \<arg_list\>") }
:::

Keyword Definition Type

---

vrrp_instance state identify a VRRP instance definition block specify the instance state in standard use
Interface mcast_src_ip specify the network interface for the instance to run on specify the src IP address value for VRRP adverts IP header string
lvs_sync_daemon_inteface specify the network interface for the LVS sync_daemon to run on string
virtual_router_id specify to which VRRP router id the instance belongs numerical
priority specify the instance priority in the VRRP router numerical
advert_int smtp_alert authentication auth_type specify the advertisement interval in seconds (set to 1) Activate the SMTP notification for MASTER state transition identify a VRRP authentication definition block specify which kind of authentication to use (PASS\|AH) numerical
auth_pass virtual_ipaddress virtual_ipaddress_excluded specify the password string to use identify a VRRP VIP definition block identify a VRRP VIP excluded definition block (not protocol VIPs) string
notify_master specify a shell script to be executed during transition to master state path
notify_backup specify a shell script to be executed during transition to backup state path
notify_fault specify a shell script to be executed during transition to fault state path
vrrp_sync_group Identify the VRRP synchronization instances group string

Path type: A system path to a script eg: "/usr/local/bin/transit.sh \<arg_list\>"
