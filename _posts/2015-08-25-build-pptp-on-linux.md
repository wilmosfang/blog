---
layout: post
title: Linux 搭建 VPN
author: wilmosfang
tags:  network vpn
categories:  vpn
wc: 579  2406 24418
excerpt: Linux 下搭建 pptp VPN 的方法
comments: true
---

---

# 前言

VPN (Virtual Private Network) 虚拟私有网络 (也有叫虚拟专用网络的)，是在互联网上使用各种技术构建出一个加密隧道，然后通过这个隧道直达对端，与对端形成一种局域互联的技术，VPN可以实现以局域互联的身份使用到远程网络资源的效果

作为翻墙越货，登堂入室，隐身马甲之必备神器，很有必要了解一点，这里我分享一下在Linux中搭建的方法

---

# 概要

* TOC
{:toc}


---

## 环境

内核版本 **2.6.32-504.el6.x86_64** 

系统版本 **CentOS release 6.6 (Final)**

~~~
[root@pptp-server ~]# uname  -a 
Linux pptp-server.cloud.com 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@pptp-server ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@pptp-server ~]# 
~~~

## 安装ppp软件包

~~~
[root@pptp-server ~]# yum install ppp 
Loaded plugins: fastestmirror, security
Setting up Install Process
Determining fastest mirrors
addons                                                                                                       | 3.4 kB     00:00     
addons/primary_db                                                                                            | 4.4 MB     00:00     
base                                                                                                         | 3.7 kB     00:00     
base/primary_db                                                                                              | 4.6 MB     00:00     
centosplus                                                                                                   | 3.4 kB     00:00     
centosplus/primary_db                                                                                        | 2.0 MB     00:00     
contrib                                                                                                      | 2.9 kB     00:00     
contrib/primary_db                                                                                           | 1.2 kB     00:00     
dag                                                                                                          | 1.9 kB     00:00     
dag/primary_db                                                                                               | 2.7 MB     00:00     
epel                                                                                                         | 4.3 kB     00:00     
epel/primary_db                                                                                              | 5.7 MB     00:00     
extras                                                                                                       | 3.4 kB     00:00     
extras/primary_db                                                                                            |  31 kB     00:00     
update                                                                                                       | 3.4 kB     00:00     
update/primary_db                                                                                            | 4.4 MB     00:00     
Package iptables-1.4.7-14.el6.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package ppp.x86_64 0:2.4.5-5.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                     Arch                           Version                              Repository                    Size
====================================================================================================================================
Installing:
 ppp                         x86_64                         2.4.5-5.el6                          base                         323 k

Transaction Summary
====================================================================================================================================
Install       1 Package(s)

Total download size: 323 k
Installed size: 760 k
Is this ok [y/N]: y
Downloading Packages:
ppp-2.4.5-5.el6.x86_64.rpm                                                                                   | 323 kB     00:00     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID c105b9de: NOKEY
Retrieving key from http://yum.ksyun.cn/centos/RPM-GPG-KEY-CentOS-6
Importing GPG key 0xC105B9DE:
 Userid: "CentOS-6 Key (CentOS 6 Official Signing Key) <centos-6-key@centos.org>"
 From  : http://yum.ksyun.cn/centos/RPM-GPG-KEY-CentOS-6
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : ppp-2.4.5-5.el6.x86_64                                                                                           1/1 
  Verifying  : ppp-2.4.5-5.el6.x86_64                                                                                           1/1 

Installed:
  ppp.x86_64 0:2.4.5-5.el6                                                                                                          

Complete!
[root@pptp-server ~]# 
~~~


## 安装pptpd软件包

~~~
[root@pptp-server ~]# yum list all | grep -i pptp 
NetworkManager-pptp.x86_64                   1:0.8.0-1.git20100411.el6    epel  
pptp.x86_64                                  1.7.2-8.1.el6                base  
pptp-setup.x86_64                            1.7.2-8.1.el6                base  
pptpd.x86_64                                 1.4.0-3.el6                  epel  
[root@pptp-server ~]# yum -y  install  pptpd.x86_64 
Loaded plugins: fastestmirror, security
Setting up Install Process
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package pptpd.x86_64 0:1.4.0-3.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                      Arch                          Version                               Repository                   Size
====================================================================================================================================
Installing:
 pptpd                        x86_64                        1.4.0-3.el6                           epel                         75 k

Transaction Summary
====================================================================================================================================
Install       1 Package(s)

Total download size: 75 k
Installed size: 178 k
Downloading Packages:
pptpd-1.4.0-3.el6.x86_64.rpm                                                                                 |  75 kB     00:00     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Retrieving key from http://yum.ksyun.cn/epel/RPM-GPG-KEY-EPEL-6
Importing GPG key 0x0608B895:
 Userid: "EPEL (6) <epel@fedoraproject.org>"
 From  : http://yum.ksyun.cn/epel/RPM-GPG-KEY-EPEL-6
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : pptpd-1.4.0-3.el6.x86_64                                                                                         1/1 
  Verifying  : pptpd-1.4.0-3.el6.x86_64                                                                                         1/1 

Installed:
  pptpd.x86_64 0:1.4.0-3.el6                                                                                                        

Complete!
[root@pptp-server ~]# 
~~~


## 打开内核参数 **net.ipv4.ip_forward**


~~~
[root@pptp-server ~]# sysctl  -a | grep forward 
net.ipv4.conf.all.forwarding = 0
net.ipv4.conf.all.mc_forwarding = 0
net.ipv4.conf.default.forwarding = 0
net.ipv4.conf.default.mc_forwarding = 0
net.ipv4.conf.lo.forwarding = 0
net.ipv4.conf.lo.mc_forwarding = 0
net.ipv4.conf.eth0.forwarding = 0
net.ipv4.conf.eth0.mc_forwarding = 0
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.all.mc_forwarding = 0
net.ipv6.conf.default.forwarding = 0
net.ipv6.conf.default.mc_forwarding = 0
net.ipv6.conf.lo.forwarding = 0
net.ipv6.conf.lo.mc_forwarding = 0
net.ipv6.conf.eth0.forwarding = 0
net.ipv6.conf.eth0.mc_forwarding = 0
[root@pptp-server ~]# vim /etc/sysctl.conf 
[root@pptp-server ~]# grep forward /etc/sysctl.conf 
net.ipv4.ip_forward = 1
[root@pptp-server ~]# sysctl  -p 
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key
error: "net.bridge.bridge-nf-call-iptables" is an unknown key
error: "net.bridge.bridge-nf-call-arptables" is an unknown key
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.eth0.rp_filter = 0
error: "net.ipv4.conf.eth1.rp_filter" is an unknown key
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.eth0.arp_announce = 2
error: "net.ipv4.conf.eth1.arp_announce" is an unknown key
[root@pptp-server ~]# sysctl  -a | grep forward 
net.ipv4.conf.all.forwarding = 1
net.ipv4.conf.all.mc_forwarding = 0
net.ipv4.conf.default.forwarding = 1
net.ipv4.conf.default.mc_forwarding = 0
net.ipv4.conf.lo.forwarding = 1
net.ipv4.conf.lo.mc_forwarding = 0
net.ipv4.conf.eth0.forwarding = 1
net.ipv4.conf.eth0.mc_forwarding = 0
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.all.mc_forwarding = 0
net.ipv6.conf.default.forwarding = 0
net.ipv6.conf.default.mc_forwarding = 0
net.ipv6.conf.lo.forwarding = 0
net.ipv6.conf.lo.mc_forwarding = 0
net.ipv6.conf.eth0.forwarding = 0
net.ipv6.conf.eth0.mc_forwarding = 0
[root@pptp-server ~]# 
~~~


## 配置pptpd

~~~
[root@pptp-server ~]# grep -v "^#"  /etc/pptpd.conf  | cat -s 

option /etc/ppp/options.pptpd

logwtmp

localip 192.168.123.1 #ip range for ppp0 of pptp server 
remoteip 192.168.123.101-200  # ip range for ppp0 of client
[root@pptp-server ~]# cat /etc/resolv.conf 
search ksyun.cn
nameserver 10.10.10.5
options rotate
options timeout:1
options attempts:2
[root@pptp-server ~]# 
[root@pptp-server ~]#  grep -v "^#"  /etc/ppp/options.pptpd  | cat -s

name pptpd

refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128

proxyarp

lock

nobsdcomp 

novj
novjccomp

nologfd

ms-dns  10.10.10.5  #add local dns server
#add some public dns server
[root@pptp-server ~]# 
~~~



## 配置防火墙

配置之前

~~~
[root@pptp-server ~]# iptables -L -nv 
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
19167   26M ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
  101  4646 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    4   156 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    1    48 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
   11   376 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT 3360 packets, 273K bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@pptp-server ~]# iptables -L -nv -t nat 
Chain PREROUTING (policy ACCEPT 255 packets, 12222 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 26 packets, 1687 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 26 packets, 1687 bytes)
 pkts bytes target     prot opt in     out     source               destination  

~~~

> **Note:** 建议在所有的 **iptables** 变更之前使用 **/etc/init.d/iptables  save** 保存一下，然后将 **/etc/sysconfig/iptables** 拷贝到一个安全的地方，以方便恢复

可以使用命令行的方式修改 **iptables** 

~~~
[root@pptp-server ~]# iptables --flush POSTROUTING --table nat 
[root@pptp-server ~]# iptables --flush FORWARD 
[root@pptp-server ~]# iptables -A INPUT -p gre -j ACCEPT 
[root@pptp-server ~]# iptables -A INPUT -p tcp -m tcp --dport 1723 -j ACCEPT
[root@pptp-server ~]# iptables -t nat -A POSTROUTING -s 192.168.123.0/24 -o eth0 -j MASQUERADE 
[root@pptp-server ~]# iptables -L -nv 
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
19377   26M ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
  103  4738 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    4   156 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    2    88 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
   12   408 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:1723 
    0     0 ACCEPT     47   --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 82 packets, 8240 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@pptp-server ~]# iptables -L -nv -t nat 
Chain PREROUTING (policy ACCEPT 1 packets, 46 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      eth0    192.168.123.0/24     0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination    
~~~


然后 **save** 一下，调整一下规则的顺序，重新 **reload** 一下

~~~     
[root@pptp-server ~]# /etc/init.d/iptables  save 
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
[root@pptp-server ~]# vim /etc/sysconfig/iptables
[root@pptp-server ~]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@pptp-server ~]#  
~~~

不过我习惯直接到 **/etc/sysconfig/iptables** 进行修改，检查无误后直接 **reload**

~~~
[root@pptp-server ~]# vim /etc/sysconfig/iptables
[root@pptp-server ~]# cat /etc/sysconfig/iptables
# Generated by iptables-save v1.4.7 on Tue Aug 25 00:02:45 2015
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [1081:165593]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A INPUT -p icmp -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -p gre -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 1723 -j ACCEPT 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT 
-A INPUT -j REJECT --reject-with icmp-host-prohibited 
COMMIT
# Completed on Tue Aug 25 00:02:45 2015
# Generated by iptables-save v1.4.7 on Tue Aug 25 00:02:45 2015
*nat
:PREROUTING ACCEPT [79:4384]
:POSTROUTING ACCEPT [4:250]
:OUTPUT ACCEPT [4:250]
-A POSTROUTING -s 192.168.123.0/24 -o eth0 -j MASQUERADE 
COMMIT
# Completed on Tue Aug 25 00:02:45 2015
[root@pptp-server ~]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@pptp-server ~]# 
~~~

配置之后

~~~
[root@pptp-server ~]# iptables -L -nv 
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   35  3695 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
    0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     47   --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:1723 
    1    60 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
    1    36 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 39 packets, 5283 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@pptp-server ~]# iptables -L -nv -t nat 
Chain PREROUTING (policy ACCEPT 5 packets, 276 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      eth0    192.168.123.0/24     0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@pptp-server ~]# 
~~~


## 创建vpn账户

~~~
[root@pptp-server ~]# vim /etc/ppp/chap-secrets 
[root@pptp-server ~]# cat /etc/ppp/chap-secrets 
# Secrets for authentication using CHAP
# client	server	secret			IP addresses
#
testvpn  pptpd  testvpnabc *
[root@pptp-server ~]# 
~~~

> **Tip:**  
> 1. 密码是可以在线修改的
> 2. 密码可以使用字母大小写Aa!@$%^&*()-_+={}[];':"<>,.?\|不能使用#\
> (密码纯是人肉测出来的)


## 启动服务

~~~
[root@pptp-server ~]# /etc/init.d/iptables  restart 
iptables: Setting chains to policy ACCEPT: nat filter      [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]
[root@pptp-server ~]# /etc/init.d/pptpd  start 
Starting pptpd:                                            [  OK  ]
[root@pptp-server ~]#  chkconfig  --list | grep -E "(pptp|iptables)" 
pptpd          	0:off	1:off	2:off	3:off	4:off	5:off	6:off
iptables       	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@pptp-server ~]# chkconfig  iptables on 
[root@pptp-server ~]# chkconfig  pptpd on 
[root@pptp-server ~]# chkconfig  --list | grep -E "(pptp|iptables)" 
iptables       	0:off	1:off	2:on	3:on	4:on	5:on	6:off
pptpd          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@pptp-server ~]# 
~~~


客户端成功连接后会出现 **ppp适配器** (这是在windows下)

~~~
PPP 适配器 VPN test for JS:

   连接特定的 DNS 后缀 . . . . . . . :
   描述. . . . . . . . . . . . . . . : VPN test for JS
   物理地址. . . . . . . . . . . . . :
   DHCP 已启用 . . . . . . . . . . . : 否
   自动配置已启用. . . . . . . . . . : 是
   IPv4 地址 . . . . . . . . . . . . : 192.168.123.101(首选)
   子网掩码  . . . . . . . . . . . . : 255.255.255.255
   默认网关. . . . . . . . . . . . . : 0.0.0.0
   DNS 服务器  . . . . . . . . . . . : 10.10.10.5
   TCPIP 上的 NetBIOS  . . . . . . . : 已启用
~~~


连接成功后服务端会出现 **ppp0**

~~~
[root@pptp-server ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether fa:16:3e:57:0f:7c brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.16/24 brd 10.10.10.255 scope global eth0
    inet6 fe80::f816:3eff:fe57:f7c/64 scope link 
       valid_lft forever preferred_lft forever
34: ppp0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1396 qdisc pfifo_fast state UNKNOWN qlen 3
    link/ppp 
    inet 192.168.123.1 peer 192.168.123.101/32 scope global ppp0
[root@pptp-server ~]#
~~~


服务端 **/var/log/messages** 中会出现类似的日志 

~~~
Aug 25 00:26:02 pptp-server pptpd[10177]: CTRL: Client 103.240.124.15 control connection started
Aug 25 00:26:02 pptp-server pptpd[10177]: CTRL: Starting call (launching pppd, opening GRE)
Aug 25 00:26:02 pptp-server pppd[10178]: Plugin /usr/lib64/pptpd/pptpd-logwtmp.so loaded.
Aug 25 00:26:02 pptp-server pppd[10178]: pppd 2.4.5 started by root, uid 0
Aug 25 00:26:02 pptp-server pppd[10178]: Using interface ppp0
Aug 25 00:26:02 pptp-server pppd[10178]: Connect: ppp0 <--> /dev/pts/1
Aug 25 00:26:05 pptp-server pppd[10178]: peer from calling number 103.240.124.15 authorized
Aug 25 00:26:05 pptp-server pppd[10178]: MPPE 128-bit stateless compression enabled
Aug 25 00:26:07 pptp-server pppd[10178]: Cannot determine ethernet address for proxy ARP
Aug 25 00:26:07 pptp-server pppd[10178]: local  IP address 192.168.123.1
Aug 25 00:26:07 pptp-server pppd[10178]: remote IP address 192.168.123.101
~~~

> **Tip:**  **pptpd** 和 **pppd**的日志默认会写到 **/var/log/messages** 中


如果这个时候 **[CheckIP][checkip]** 会发自己已经穿越到了另一个地方


如果断开连接，服务端会出现下列日志

~~~
Aug 25 00:40:35 pptp-server pppd[10178]: LCP terminated by peer (nM-,<=^@<M-Mt^@^@^@^@)
Aug 25 00:40:35 pptp-server pppd[10178]: Connect time 14.5 minutes.
Aug 25 00:40:35 pptp-server pppd[10178]: Sent 3146618 bytes, received 469485 bytes.
Aug 25 00:40:35 pptp-server pppd[10178]: Modem hangup
Aug 25 00:40:35 pptp-server pppd[10178]: Connection terminated.
Aug 25 00:40:35 pptp-server pppd[10178]: Exit.
Aug 25 00:40:35 pptp-server pptpd[10177]: CTRL: Client 103.240.124.15 control connection finished
~~~

所有客户端都断开后 服务端的**ppp0** 也会消失


---

# 命令汇总

* **`yum install ppp`**
* **`yum list all | grep -i pptp`**
* **`yum -y  install  pptpd.x86_64`**
* **`sysctl  -a | grep forward`**
* **`vim /etc/sysctl.conf`**
* **`grep forward /etc/sysctl.conf`**
* **`sysctl  -p`**
* **`sysctl  -a | grep forward`**
* **`grep -v "^#"  /etc/pptpd.conf  | cat -s`**
* **`cat /etc/resolv.conf`**
* **`grep -v "^#"  /etc/ppp/options.pptpd  | cat -s`**
* **`iptables --flush POSTROUTING --table nat`**
* **`iptables --flush FORWARD`**
* **`iptables -A INPUT -p gre -j ACCEPT`**
* **`iptables -A INPUT -p tcp -m tcp --dport 1723 -j ACCEPT`**
* **`iptables -t nat -A POSTROUTING -s 192.168.123.0/24 -o eth0 -j MASQUERADE`**
* **`/etc/init.d/iptables  save`**
* **`vim /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables  reload`**
* **`iptables -L -nv`**
* **`iptables -L -nv -t nat`**
* **`cat /etc/ppp/chap-secrets`**
* **`/etc/init.d/iptables  restart`**
* **`/etc/init.d/pptpd  start`**
* **`chkconfig  --list | grep -E "(pptp|iptables)"`**
* **`chkconfig  iptables on`**
* **`chkconfig  pptpd on`**
* **`ip a`**

---

[checkip]: http://ip.chinaz.com/
