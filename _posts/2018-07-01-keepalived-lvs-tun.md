---
layout: post
title: "Keepalived LVS TUN"
author:  wilmosfang
date: 2018-07-01 23:52:34
image: '/assets/img/'
excerpt: 'Keepalived LVS TUN 配置'
main-class: 'lvs'
color: '#3c4a6a'
tags:
 - lvs
 - nginx
 - keepalived
categories: 
 - lvs
twitter_text: 'simple process of Keepalived LVS TUN config'
introduction: 'Configration of Keepalived LVS TUN'
---

# 前言

**[Keepalived][keepalived]** 作为 LVS 的有效补充可以构建一个高可用的 LB 前端

>Keepalived is a routing software written in C. The main goal of this project is to provide simple and robust facilities for loadbalancing and high-availability to Linux system and Linux based infrastructures

**[Keepalived][keepalived]** 主要使用 VRRP 实现 VIP 的管理

>On the other hand high-availability is achieved by VRRP protocol. VRRP is a fundamental brick for router failover. In addition, Keepalived implements a set of hooks to the VRRP finite state machine providing low-level and high-speed protocol interactions. Keepalived frameworks can be used independently or all together to provide resilient infrastructures

LVS 只实现到了四层，**[Keepalived][keepalived]** 可以实现七层的简单检查，**[Keepalived][keepalived]** 可以通过预设的检查逻辑来管理 LVS 配置，从而实现对 LVS 自动且动态的调配，让整个 LB 系统更加灵活且健壮

这里演示一下如何配置 **[Keepalived][keepalived]** 加 **[LVS][lvs]** 的 TUN 模式

> **Tip:** 当前的版本为 **IPVS 1.2.1** 和 **Keepalived Version 2.0.5** (但是实验环境下，没有使用最新的版本)

---

# 操作

## 系统环境

**DS**

~~~
[root@ds1 ~]# hostnamectl 
   Static hostname: ds1
         Icon name: computer-vm
           Chassis: vm
        Machine ID: fce1d1d9ca0345dca5e300f4ee8f6b0a
           Boot ID: 04693ae27f67491ea0d1f23fd456904f
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[root@ds1 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86231sec preferred_lft 86231sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:fc:bf:30 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.181/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fefc:bf30/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b3:ae:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.181/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb3:ae0b/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds1 ~]# cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core) 
[root@ds1 ~]# 
~~~

**RS**

~~~
[vagrant@rs1 ~]$ hostnamectl 
   Static hostname: rs1
         Icon name: computer-vm
           Chassis: vm
        Machine ID: f7f1e12e7c2b48cd90739bb41d24c97d
           Boot ID: f50c7cf156a24573a9be96f998dc8d51
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[vagrant@rs1 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86240sec preferred_lft 86240sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ab:52:cb brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.183/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feab:52cb/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@rs1 ~]$ cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core) 
[vagrant@rs1 ~]$ 
~~~

rs3

~~~
[vagrant@rs3 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85835sec preferred_lft 85835sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:1f:90:b7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.183/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1f:90b7/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@rs3 ~]$ 
~~~



## 网络环境

~~~
ds1:
    192.168.1.181
    192.168.56.181
ds2:
    192.168.1.182
    192.168.56.182
rs1:
    192.168.56.183
rs2:
    192.168.56.184
rs3:
    192.168.1.183
~~~

VIP 为 192.168.1.184

CIP 为 192.168.1.6

**拓扑**

~~~
client--->ds1
      `-->ds2
      `<--rs1
      `<--rs2
      `<--rs3
~~~


## 概念

* DS：Director Server 指的是前端负载均衡器节点
* RS：Real Server 后端真实的工作服务器
* VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址
* DIP：Director Server IP，主要用于和内部主机通讯的IP地址
* RIP：Real Server IP，后端服务器的IP地址
* CIP：Client IP，访问客户端的IP地址


## RS 上本地安装 Nginx

~~~
[root@rs1 ~]# curl localhost:80
rs1
[root@rs1 ~]# 
~~~

~~~
[root@rs2 ~]# curl localhost:80
rs2
[root@rs2 ~]# 
~~~

~~~
[root@rs3 ~]# curl localhost:80
rs3
[root@rs3 ~]# 
~~~

## 安装 keepalived

~~~
[root@ds1 ~]# yum list all | grep keep 
keepalived.x86_64                           1.3.5-6.el7                base     
[root@ds1 ~]# 
[root@ds1 ~]# yum install keepalived.x86_64
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package keepalived.x86_64 0:1.3.5-6.el7 will be installed
--> Processing Dependency: libnetsnmpmibs.so.31()(64bit) for package: keepalived-1.3.5-6.el7.x86_64
--> Processing Dependency: libnetsnmpagent.so.31()(64bit) for package: keepalived-1.3.5-6.el7.x86_64
--> Processing Dependency: libnetsnmp.so.31()(64bit) for package: keepalived-1.3.5-6.el7.x86_64
--> Running transaction check
---> Package net-snmp-agent-libs.x86_64 1:5.7.2-33.el7_5.2 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: 1:net-snmp-agent-libs-5.7.2-33.el7_5.2.x86_64
--> Processing Dependency: libsensors.so.4()(64bit) for package: 1:net-snmp-agent-libs-5.7.2-33.el7_5.2.x86_64
--> Processing Dependency: libperl.so()(64bit) for package: 1:net-snmp-agent-libs-5.7.2-33.el7_5.2.x86_64
---> Package net-snmp-libs.x86_64 1:5.7.2-33.el7_5.2 will be installed
--> Running transaction check
---> Package lm_sensors-libs.x86_64 0:3.4.0-4.20160601gitf9185e5.el7 will be installed
---> Package perl.x86_64 4:5.16.3-292.el7 will be installed
--> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl-macros for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-292.el7.x86_64
---> Package perl-libs.x86_64 4:5.16.3-292.el7 will be installed
--> Running transaction check
---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
--> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
--> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
--> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
--> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
---> Package perl-Socket.x86_64 0:2.010-4.el7 will be installed
---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
---> Package perl-macros.x86_64 4:5.16.3-292.el7 will be installed
---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
--> Running transaction check
---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
---> Package perl-Pod-Escapes.noarch 1:1.04-292.el7 will be installed
---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
--> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
--> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
--> Running transaction check
---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
--> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
--> Running transaction check
---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch    Version                         Repository
                                                                           Size
================================================================================
Installing:
 keepalived              x86_64  1.3.5-6.el7                     base     329 k
Installing for dependencies:
 lm_sensors-libs         x86_64  3.4.0-4.20160601gitf9185e5.el7  base      41 k
 net-snmp-agent-libs     x86_64  1:5.7.2-33.el7_5.2              updates  705 k
 net-snmp-libs           x86_64  1:5.7.2-33.el7_5.2              updates  749 k
 perl                    x86_64  4:5.16.3-292.el7                base     8.0 M
 perl-Carp               noarch  1.26-244.el7                    base      19 k
 perl-Encode             x86_64  2.51-7.el7                      base     1.5 M
 perl-Exporter           noarch  5.68-3.el7                      base      28 k
 perl-File-Path          noarch  2.09-2.el7                      base      26 k
 perl-File-Temp          noarch  0.23.01-3.el7                   base      56 k
 perl-Filter             x86_64  1.49-3.el7                      base      76 k
 perl-Getopt-Long        noarch  2.40-3.el7                      base      56 k
 perl-HTTP-Tiny          noarch  0.033-3.el7                     base      38 k
 perl-PathTools          x86_64  3.40-5.el7                      base      82 k
 perl-Pod-Escapes        noarch  1:1.04-292.el7                  base      51 k
 perl-Pod-Perldoc        noarch  3.20-4.el7                      base      87 k
 perl-Pod-Simple         noarch  1:3.28-4.el7                    base     216 k
 perl-Pod-Usage          noarch  1.63-3.el7                      base      27 k
 perl-Scalar-List-Utils  x86_64  1.27-248.el7                    base      36 k
 perl-Socket             x86_64  2.010-4.el7                     base      49 k
 perl-Storable           x86_64  2.45-3.el7                      base      77 k
 perl-Text-ParseWords    noarch  3.29-4.el7                      base      14 k
 perl-Time-HiRes         x86_64  4:1.9725-3.el7                  base      45 k
 perl-Time-Local         noarch  1.2300-2.el7                    base      24 k
 perl-constant           noarch  1.27-2.el7                      base      19 k
 perl-libs               x86_64  4:5.16.3-292.el7                base     688 k
 perl-macros             x86_64  4:5.16.3-292.el7                base      43 k
 perl-parent             noarch  1:0.225-244.el7                 base      12 k
 perl-podlators          noarch  2.5.1-3.el7                     base     112 k
 perl-threads            x86_64  1.87-4.el7                      base      49 k
 perl-threads-shared     x86_64  1.43-6.el7                      base      39 k

Transaction Summary
================================================================================
Install  1 Package (+30 Dependent packages)

Total download size: 13 M
Installed size: 42 M
Is this ok [y/d/N]: y
Downloading packages:
(1/31): lm_sensors-libs-3.4.0-4.20160601gitf9185e5.el7.x86 |  41 kB   00:00     
(2/31): perl-Carp-1.26-244.el7.noarch.rpm                  |  19 kB   00:00     
(3/31): keepalived-1.3.5-6.el7.x86_64.rpm                  | 329 kB   00:00     
(4/31): perl-Exporter-5.68-3.el7.noarch.rpm                |  28 kB   00:00     
(5/31): net-snmp-agent-libs-5.7.2-33.el7_5.2.x86_64.rpm    | 705 kB   00:00     
(6/31): perl-File-Path-2.09-2.el7.noarch.rpm               |  26 kB   00:00     
(7/31): perl-Filter-1.49-3.el7.x86_64.rpm                  |  76 kB   00:00     
(8/31): perl-Getopt-Long-2.40-3.el7.noarch.rpm             |  56 kB   00:00     
(9/31): perl-HTTP-Tiny-0.033-3.el7.noarch.rpm              |  38 kB   00:00     
(10/31): net-snmp-libs-5.7.2-33.el7_5.2.x86_64.rpm         | 749 kB   00:01     
(11/31): perl-PathTools-3.40-5.el7.x86_64.rpm              |  82 kB   00:00     
(12/31): perl-File-Temp-0.23.01-3.el7.noarch.rpm           |  56 kB   00:00     
(13/31): perl-Pod-Perldoc-3.20-4.el7.noarch.rpm            |  87 kB   00:00     
(14/31): perl-Pod-Simple-3.28-4.el7.noarch.rpm             | 216 kB   00:00     
(15/31): perl-Scalar-List-Utils-1.27-248.el7.x86_64.rpm    |  36 kB   00:00     
(16/31): perl-Pod-Escapes-1.04-292.el7.noarch.rpm          |  51 kB   00:00     
(17/31): perl-Socket-2.010-4.el7.x86_64.rpm                |  49 kB   00:00     
(18/31): perl-Storable-2.45-3.el7.x86_64.rpm               |  77 kB   00:00     
(19/31): perl-Text-ParseWords-3.29-4.el7.noarch.rpm        |  14 kB   00:00     
(20/31): perl-Time-HiRes-1.9725-3.el7.x86_64.rpm           |  45 kB   00:00     
(21/31): perl-Pod-Usage-1.63-3.el7.noarch.rpm              |  27 kB   00:00     
(22/31): perl-Time-Local-1.2300-2.el7.noarch.rpm           |  24 kB   00:00     
(23/31): perl-macros-5.16.3-292.el7.x86_64.rpm             |  43 kB   00:00     
(24/31): perl-constant-1.27-2.el7.noarch.rpm               |  19 kB   00:00     
(25/31): perl-parent-0.225-244.el7.noarch.rpm              |  12 kB   00:00     
(26/31): perl-podlators-2.5.1-3.el7.noarch.rpm             | 112 kB   00:00     
(27/31): perl-threads-shared-1.43-6.el7.x86_64.rpm         |  39 kB   00:00     
(28/31): perl-threads-1.87-4.el7.x86_64.rpm                |  49 kB   00:00     
(29/31): perl-libs-5.16.3-292.el7.x86_64.rpm               | 688 kB   00:00     
(30/31): perl-Encode-2.51-7.el7.x86_64.rpm                 | 1.5 MB   00:03     
(31/31): perl-5.16.3-292.el7.x86_64.rpm                    | 8.0 MB   00:05     
--------------------------------------------------------------------------------
Total                                              2.5 MB/s |  13 MB  00:05     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:net-snmp-libs-5.7.2-33.el7_5.2.x86_64                     1/31 
  Installing : 1:perl-parent-0.225-244.el7.noarch                          2/31 
  Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                           3/31 
  Installing : perl-podlators-2.5.1-3.el7.noarch                           4/31 
  Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                          5/31 
  Installing : 1:perl-Pod-Escapes-1.04-292.el7.noarch                      6/31 
  Installing : perl-Text-ParseWords-3.29-4.el7.noarch                      7/31 
  Installing : perl-Encode-2.51-7.el7.x86_64                               8/31 
  Installing : perl-Pod-Usage-1.63-3.el7.noarch                            9/31 
  Installing : 4:perl-macros-5.16.3-292.el7.x86_64                        10/31 
  Installing : 4:perl-libs-5.16.3-292.el7.x86_64                          11/31 
  Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      12/31 
  Installing : perl-Exporter-5.68-3.el7.noarch                            13/31 
  Installing : perl-constant-1.27-2.el7.noarch                            14/31 
  Installing : perl-Time-Local-1.2300-2.el7.noarch                        15/31 
  Installing : perl-Socket-2.010-4.el7.x86_64                             16/31 
  Installing : perl-Carp-1.26-244.el7.noarch                              17/31 
  Installing : perl-Storable-2.45-3.el7.x86_64                            18/31 
  Installing : perl-PathTools-3.40-5.el7.x86_64                           19/31 
  Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 20/31 
  Installing : perl-File-Temp-0.23.01-3.el7.noarch                        21/31 
  Installing : perl-File-Path-2.09-2.el7.noarch                           22/31 
  Installing : perl-threads-shared-1.43-6.el7.x86_64                      23/31 
  Installing : perl-threads-1.87-4.el7.x86_64                             24/31 
  Installing : perl-Filter-1.49-3.el7.x86_64                              25/31 
  Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        26/31 
  Installing : perl-Getopt-Long-2.40-3.el7.noarch                         27/31 
  Installing : 4:perl-5.16.3-292.el7.x86_64                               28/31 
  Installing : lm_sensors-libs-3.4.0-4.20160601gitf9185e5.el7.x86_64      29/31 
  Installing : 1:net-snmp-agent-libs-5.7.2-33.el7_5.2.x86_64              30/31 
  Installing : keepalived-1.3.5-6.el7.x86_64                              31/31 
  Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           1/31 
  Verifying  : perl-threads-shared-1.43-6.el7.x86_64                       2/31 
  Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                       3/31 
  Verifying  : perl-Exporter-5.68-3.el7.noarch                             4/31 
  Verifying  : perl-constant-1.27-2.el7.noarch                             5/31 
  Verifying  : perl-PathTools-3.40-5.el7.x86_64                            6/31 
  Verifying  : 4:perl-macros-5.16.3-292.el7.x86_64                         7/31 
  Verifying  : 1:net-snmp-libs-5.7.2-33.el7_5.2.x86_64                     8/31 
  Verifying  : 1:perl-parent-0.225-244.el7.noarch                          9/31 
  Verifying  : 4:perl-5.16.3-292.el7.x86_64                               10/31 
  Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                        11/31 
  Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        12/31 
  Verifying  : keepalived-1.3.5-6.el7.x86_64                              13/31 
  Verifying  : 4:perl-libs-5.16.3-292.el7.x86_64                          14/31 
  Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        15/31 
  Verifying  : perl-Socket-2.010-4.el7.x86_64                             16/31 
  Verifying  : perl-Carp-1.26-244.el7.noarch                              17/31 
  Verifying  : perl-Storable-2.45-3.el7.x86_64                            18/31 
  Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 19/31 
  Verifying  : 1:perl-Pod-Escapes-1.04-292.el7.noarch                     20/31 
  Verifying  : lm_sensors-libs-3.4.0-4.20160601gitf9185e5.el7.x86_64      21/31 
  Verifying  : 1:net-snmp-agent-libs-5.7.2-33.el7_5.2.x86_64              22/31 
  Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           23/31 
  Verifying  : perl-Encode-2.51-7.el7.x86_64                              24/31 
  Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         25/31 
  Verifying  : perl-podlators-2.5.1-3.el7.noarch                          26/31 
  Verifying  : perl-File-Path-2.09-2.el7.noarch                           27/31 
  Verifying  : perl-threads-1.87-4.el7.x86_64                             28/31 
  Verifying  : perl-Filter-1.49-3.el7.x86_64                              29/31 
  Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         30/31 
  Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     31/31 

Installed:
  keepalived.x86_64 0:1.3.5-6.el7                                               

Dependency Installed:
  lm_sensors-libs.x86_64 0:3.4.0-4.20160601gitf9185e5.el7                       
  net-snmp-agent-libs.x86_64 1:5.7.2-33.el7_5.2                                 
  net-snmp-libs.x86_64 1:5.7.2-33.el7_5.2                                       
  perl.x86_64 4:5.16.3-292.el7                                                  
  perl-Carp.noarch 0:1.26-244.el7                                               
  perl-Encode.x86_64 0:2.51-7.el7                                               
  perl-Exporter.noarch 0:5.68-3.el7                                             
  perl-File-Path.noarch 0:2.09-2.el7                                            
  perl-File-Temp.noarch 0:0.23.01-3.el7                                         
  perl-Filter.x86_64 0:1.49-3.el7                                               
  perl-Getopt-Long.noarch 0:2.40-3.el7                                          
  perl-HTTP-Tiny.noarch 0:0.033-3.el7                                           
  perl-PathTools.x86_64 0:3.40-5.el7                                            
  perl-Pod-Escapes.noarch 1:1.04-292.el7                                        
  perl-Pod-Perldoc.noarch 0:3.20-4.el7                                          
  perl-Pod-Simple.noarch 1:3.28-4.el7                                           
  perl-Pod-Usage.noarch 0:1.63-3.el7                                            
  perl-Scalar-List-Utils.x86_64 0:1.27-248.el7                                  
  perl-Socket.x86_64 0:2.010-4.el7                                              
  perl-Storable.x86_64 0:2.45-3.el7                                             
  perl-Text-ParseWords.noarch 0:3.29-4.el7                                      
  perl-Time-HiRes.x86_64 4:1.9725-3.el7                                         
  perl-Time-Local.noarch 0:1.2300-2.el7                                         
  perl-constant.noarch 0:1.27-2.el7                                             
  perl-libs.x86_64 4:5.16.3-292.el7                                             
  perl-macros.x86_64 4:5.16.3-292.el7                                           
  perl-parent.noarch 1:0.225-244.el7                                            
  perl-podlators.noarch 0:2.5.1-3.el7                                           
  perl-threads.x86_64 0:1.87-4.el7                                              
  perl-threads-shared.x86_64 0:1.43-6.el7                                       

Complete!
[root@ds1 ~]# 
~~~

在 ds2 上也进行相同的操作，以准备好 keepalived 环境

## 配置 keepalived

~~~
[root@ds1 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     #acassen@firewall.loc
     #failover@firewall.loc
     #sysadmin@firewall.loc
   }
   #notification_email_from Alexandre.Cassen@firewall.loc
   #smtp_server 192.168.200.1
   #smtp_connect_timeout 30
   router_id LVS_ds1
   vrrp_skip_check_adv_addr
   #vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 50
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.184/24
    }
}

virtual_server 192.168.1.184 80 {
    delay_loop 6
    lb_algo wrr
    lb_kind TUN
    persistence_timeout 0
    protocol TCP

    real_server 192.168.56.183 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 192.168.56.184 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            delay_before_retry 3
            connect_port 80
        }
    }
   real_server 192.168.1.183 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            delay_before_retry 3
            connect_port 80
        }
    }
}

[root@ds1 ~]# 
~~~

ds2 的配置为

~~~
[root@ds2 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     #acassen@firewall.loc
     #failover@firewall.loc
     #sysadmin@firewall.loc
   }
   #notification_email_from Alexandre.Cassen@firewall.loc
   #smtp_server 192.168.200.1
   #smtp_connect_timeout 30
   router_id LVS_ds2
   vrrp_skip_check_adv_addr
   #vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 50
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.184/24
    }
}

virtual_server 192.168.1.184 80 {
    delay_loop 6
    lb_algo wrr
    lb_kind TUN
    persistence_timeout 0
    protocol TCP

    real_server 192.168.56.183 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 192.168.56.184 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            delay_before_retry 3
            connect_port 80
        }
    }
   real_server 192.168.1.183 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            delay_before_retry 3
            connect_port 80
        }
    }
}

[root@ds2 ~]# 
~~~

master 与 slave 的配配置差异为

~~~
[root@ds1 ~]# diff /etc/keepalived/keepalived.conf /tmp/slave 
12c12
<    router_id LVS_ds1
---
>    router_id LVS_ds2
20c20
<     state MASTER
---
>     state BACKUP
23c23
<     priority 100
---
>     priority 90
[root@ds1 ~]#
~~~

## 启动 keepalived 实例

先启动 slave 看看情况

~~~
[root@ds2 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83927sec preferred_lft 83927sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d5:11:3f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.182/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed5:113f/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:93:b1:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.182/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe93:b138/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds2 ~]# systemctl  start keepalived
[root@ds2 ~]# systemctl  status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-07-03 00:13:38 UTC; 7s ago
  Process: 3415 ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 3416 (keepalived)
   CGroup: /system.slice/keepalived.service
           ├─3416 /usr/sbin/keepalived -D
           ├─3417 /usr/sbin/keepalived -D
           └─3418 /usr/sbin/keepalived -D

Jul 03 00:13:38 ds2 Keepalived_healthcheckers[3417]: Activating healthchecker...
Jul 03 00:13:42 ds2 Keepalived_vrrp[3418]: VRRP_Instance(VI_1) Transition to...E
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: VRRP_Instance(VI_1) Entering MAST...E
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: VRRP_Instance(VI_1) setting proto....
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: VRRP_Instance(VI_1) Sending/queue...4
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:13:43 ds2 Keepalived_vrrp[3418]: Sending gratuitous ARP on eth1 fo...4
Hint: Some lines were ellipsized, use -l to show in full.
[root@ds2 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83903sec preferred_lft 83903sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d5:11:3f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.182/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet 192.168.1.184/24 scope global secondary eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed5:113f/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:93:b1:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.182/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe93:b138/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds2 ~]# 
~~~

再启动 master 发现 IP 就被抢占了

~~~
[root@ds1 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83830sec preferred_lft 83830sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:fc:bf:30 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.181/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fefc:bf30/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b3:ae:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.181/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb3:ae0b/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds1 ~]# systemctl start keepalived
[root@ds1 ~]# systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-07-03 00:14:41 UTC; 6s ago
  Process: 3429 ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 3430 (keepalived)
   CGroup: /system.slice/keepalived.service
           ├─3430 /usr/sbin/keepalived -D
           ├─3431 /usr/sbin/keepalived -D
           └─3432 /usr/sbin/keepalived -D

Jul 03 00:14:43 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:43 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:43 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:43 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:48 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:48 ds1 Keepalived_vrrp[3432]: VRRP_Instance(VI_1) Sending/queue...4
Jul 03 00:14:48 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:48 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:48 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Jul 03 00:14:48 ds1 Keepalived_vrrp[3432]: Sending gratuitous ARP on eth1 fo...4
Hint: Some lines were ellipsized, use -l to show in full.
[root@ds1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83794sec preferred_lft 83794sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:fc:bf:30 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.181/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet 192.168.1.184/24 scope global secondary eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fefc:bf30/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b3:ae:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.181/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb3:ae0b/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds1 ~]# 
~~~

这时看看本地就产生了 LVS 的配置

~~~
[root@ds1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.184:80 wrr
  -> 192.168.1.183:80             Tunnel  1      0          0         
  -> 192.168.56.183:80            Tunnel  1      0          0         
  -> 192.168.56.184:80            Tunnel  1      0          0         
[root@ds1 ~]# 
~~~

~~~
[root@ds2 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.184:80 wrr
  -> 192.168.1.183:80             Tunnel  1      0          0         
  -> 192.168.56.183:80            Tunnel  1      0          0         
  -> 192.168.56.184:80            Tunnel  1      0          0         
[root@ds2 ~]# 
~~~

此时还无法访问,这是因为 RS 上还没通过隧道绑定 VIP

## RS 调整默认路由

~~~
[root@rs1 ~]# ip route
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.183 metric 101 
[root@rs1 ~]# ping  192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=63 time=0.227 ms
64 bytes from 192.168.1.6: icmp_seq=2 ttl=63 time=0.603 ms
^C
--- 192.168.1.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.227/0.415/0.603/0.188 ms
[root@rs1 ~]# tracepath -n 192.168.1.6
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.0.2.2                                              0.442ms 
 1:  10.0.2.2                                              0.913ms 
 2:  192.168.1.6                                           0.662ms reached
     Resume: pmtu 1500 hops 2 back 64 
[root@rs1 ~]# ip route del default
[root@rs1 ~]# ip route
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.183 metric 101 
[root@rs1 ~]# ping  192.168.1.6
connect: Network is unreachable
[root@rs1 ~]# ip route add 192.168.1.6 dev eth1
[root@rs1 ~]# ip route
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.1.6 dev eth1 scope link 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.183 metric 101 
[root@rs1 ~]# ping  192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=64 time=0.486 ms
64 bytes from 192.168.1.6: icmp_seq=2 ttl=64 time=0.318 ms
^C
--- 192.168.1.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1007ms
rtt min/avg/max/mdev = 0.318/0.402/0.486/0.084 ms
[root@rs1 ~]# tracepath -n 192.168.1.6
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.1.6                                           0.314ms reached
 1:  192.168.1.6                                           0.529ms reached
     Resume: pmtu 1500 hops 1 back 1 
[root@rs1 ~]# 
~~~

本来都可达，为什么要调整路由呢

这就涉及到了 LVS TUN 的工作原理

如果根据原来的默认路由，是会发生 SNAT 的，而一旦数据包经过了 SNAT, 源 IP 也就是 RS 的 VIP 回包的源地址就被替换了，这样客户端在发出一个请求后收到一个不知是谁发过来的回包，就是蒙逼的，它会丢弃掉

这样就无法构建正常的数据通讯

所以在 rs2 上执行相同操作

~~~
[root@rs2 ~]# ip route 
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.184 metric 101 
[root@rs2 ~]# ping  192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=63 time=0.626 ms
64 bytes from 192.168.1.6: icmp_seq=2 ttl=63 time=0.355 ms
^C
--- 192.168.1.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.355/0.490/0.626/0.137 ms
[root@rs2 ~]# tracepath -n 192.168.1.6
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.0.2.2                                              0.452ms 
 1:  10.0.2.2                                              0.396ms 
 2:  192.168.1.6                                           0.563ms reached
     Resume: pmtu 1500 hops 2 back 64 
[root@rs2 ~]# ip route del default
[root@rs2 ~]# ip route 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.184 metric 101 
[root@rs2 ~]# ping  192.168.1.6
connect: Network is unreachable
[root@rs2 ~]# ip route add 192.168.1.6 dev eth1
[root@rs2 ~]# ip route 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.1.6 dev eth1 scope link 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.184 metric 101 
[root@rs2 ~]# ping  192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=64 time=0.592 ms
^C
--- 192.168.1.6 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.592/0.592/0.592/0.000 ms
[root@rs2 ~]# tracepath -n 192.168.1.6
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.1.6                                           0.538ms reached
 1:  192.168.1.6                                           0.423ms reached
     Resume: pmtu 1500 hops 1 back 1 
[root@rs2 ~]# 
~~~

r3 上不用特特殊操作，因为是直连的

~~~
[root@rs3 ~]# ip route 
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.1.0/24 dev eth1 proto kernel scope link src 192.168.1.183 metric 101 
[root@rs3 ~]# ping 192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=64 time=0.287 ms
64 bytes from 192.168.1.6: icmp_seq=2 ttl=64 time=0.467 ms
^C
--- 192.168.1.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.287/0.377/0.467/0.090 ms
[root@rs3 ~]# tracepath -n 192.168.1.6
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.1.6                                           0.175ms reached
 1:  192.168.1.6                                           0.146ms reached
     Resume: pmtu 1500 hops 1 back 1 
[root@rs3 ~]# 
~~~


## RS 脚本

~~~
[root@rs1 ~]# ls
anaconda-ks.cfg  original-ks.cfg  rs_init  rs_init_tun
[root@rs1 ~]# cat rs_init_tun 
VIP=192.168.1.184
case "$1" in
start)
    /sbin/ifconfig tunl0 $VIP broadcast $VIP netmask 255.255.255.255 up
    /sbin/route add -host $VIP dev tunl0
    /bin/echo 1 > /proc/sys/net/ipv4/conf/tunl0/arp_ignore
    /bin/echo 2 > /proc/sys/net/ipv4/conf/tunl0/arp_announce
    /bin/echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
    /bin/echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
    /bin/echo 0 > /proc/sys/net/ipv4/conf/tunl0/rp_filter
    /bin/echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter
    /bin/echo "rs start ok"
    ;;
stop)
    /sbin/ifconfig tunl0 down
    /sbin/route del $VIP >/dev/null 2>&1
    /bin/echo 0 > /proc/sys/net/ipv4/conf/tunl0/arp_ignore
    /bin/echo 0 > /proc/sys/net/ipv4/conf/tunl0/arp_announce
    /bin/echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
    /bin/echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
    /bin/echo 1 > /proc/sys/net/ipv4/conf/tunl0/rp_filter
    /bin/echo 1 > /proc/sys/net/ipv4/conf/all/rp_filter
    /bin/echo "rs stop ok"
    ;;
*)
    echo "Usage: $0 {start|stop}"
    exit 1
esac
exit 0
[root@rs1 ~]# 
~~~

执行脚本

r1 r2 上都执行相同操作

~~~
[root@rs1 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84917sec preferred_lft 84917sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ab:52:cb brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.183/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feab:52cb/64 scope link 
       valid_lft forever preferred_lft forever
[root@rs1 ~]# ./rs_init_tun 
Usage: ./rs_init_tun {start|stop}
[root@rs1 ~]# ./rs_init_tun  start
rs start ok
[root@rs1 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84896sec preferred_lft 84896sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ab:52:cb brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.183/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feab:52cb/64 scope link 
       valid_lft forever preferred_lft forever
4: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 192.168.1.184/32 brd 192.168.1.184 scope global tunl0
       valid_lft forever preferred_lft forever
[root@rs1 ~]# 
[root@rs1 ~]# cat /proc/sys/net/ipv4/conf/tunl0/arp_ignore
1
[root@rs1 ~]# cat /proc/sys/net/ipv4/conf/tunl0/arp_announce
2
[root@rs1 ~]# cat /proc/sys/net/ipv4/conf/all/arp_ignore
1
[root@rs1 ~]# cat /proc/sys/net/ipv4/conf/all/arp_announce
2
[root@rs1 ~]# cat /proc/sys/net/ipv4/conf/tunl0/rp_filter
0
[root@rs1 ~]# cat /proc/sys/net/ipv4/conf/all/rp_filter
0
[root@rs1 ~]# 
~~~


rs3 上也执行相同的操作

~~~
[root@rs3 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 81474sec preferred_lft 81474sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:1f:90:b7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.183/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1f:90b7/64 scope link 
       valid_lft forever preferred_lft forever
[root@rs3 ~]# ./rs_init_tun 
Usage: ./rs_init_tun {start|stop}
[root@rs3 ~]# ./rs_init_tun start
rs start ok
[root@rs3 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 81466sec preferred_lft 81466sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:1f:90:b7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.183/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1f:90b7/64 scope link 
       valid_lft forever preferred_lft forever
4: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 192.168.1.184/32 brd 192.168.1.184 scope global tunl0
       valid_lft forever preferred_lft forever
[root@rs3 ~]# 
~~~


## 客户端上先关掉源 mac 的过滤

因为这是实验环境，RS 与 Client 是直连的，Client 发请求的和收请求的都不是同一个网卡

这时如果不关闭过滤， client 会认为是垃圾数据而将响应包丢弃，于是 rs1 和 rs2 的数据就无法获取

>**Note:** 在实际环境中，是无法干预客户端行为的，但是由于 RS 离 client 比较远，client 收取数据都是通过自己的网关，所以不会有这种实验环境下要特殊处理的情况

不关掉效果就是这样的，给到 rs1 和 rs2 的包，被丢弃了，rs3 的包可以正常获取

~~~
wilmos@Nothing:~$ curl 192.168.1.184
^C
wilmos@Nothing:~$ curl 192.168.1.184
^C
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
^C
wilmos@Nothing:~$ curl 192.168.1.184
^C
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$
~~~

处理方法也很简单，就是关掉过滤

~~~
root@Nothing:~# cat /proc/sys/net/ipv4/conf/vboxnet0/rp_filter
1
root@Nothing:~# cat /proc/sys/net/ipv4/conf/all/rp_filter
1
root@Nothing:~# echo 0 > /proc/sys/net/ipv4/conf/vboxnet0/rp_filter
root@Nothing:~# echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter
root@Nothing:~# cat /proc/sys/net/ipv4/conf/all/rp_filter
0
root@Nothing:~# cat /proc/sys/net/ipv4/conf/all/rp_filter
0
root@Nothing:~# 
~~~


## 进行访问

~~~
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ 
~~~

达到了预期效果


## 停止一个后端

~~~
[root@rs1 ~]# curl localhost
rs1
[root@rs1 ~]# systemctl stop nginx
[root@rs1 ~]# curl localhost
curl: (7) Failed connect to localhost:80; Connection refused
[root@rs1 ~]# curl localhost
curl: (7) Failed connect to localhost:80; Connection refused
[root@rs1 ~]# 
~~~

keepalived 侦测到了这个变化，日志中有如下输出

~~~
...
...
Jul  3 01:11:22 ds1 Keepalived_healthcheckers[3431]: TCP connection to [192.168.56.183]:80 failed.
Jul  3 01:11:25 ds1 Keepalived_healthcheckers[3431]: TCP connection to [192.168.56.183]:80 failed.
Jul  3 01:11:25 ds1 Keepalived_healthcheckers[3431]: Check on service [192.168.56.183]:80 failed after 1 retry.
Jul  3 01:11:25 ds1 Keepalived_healthcheckers[3431]: Removing service [192.168.56.183]:80 from VS [192.168.1.184]:80
...
...
~~~

查看 IPVS 配置

~~~
[root@ds1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.184:80 wrr
  -> 192.168.1.183:80             Tunnel  1      0          0         
  -> 192.168.56.184:80            Tunnel  1      0          0         
[root@ds1 ~]# 
~~~

少了 rs1 的 IP

客户端访问

~~~
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ 
~~~

并不影响客户端访问，只是所有请求都转发给了另外两台

## 停止 keepalived master

~~~
[root@ds1 ~]# systemctl stop keepalived
[root@ds1 ~]# systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Jul 03 01:13:55 ds1 Keepalived_healthcheckers[3431]: TCP connection to [192.1...
Jul 03 01:13:55 ds1 Keepalived_healthcheckers[3431]: Adding service [192.168....
Jul 03 01:14:20 ds1 Keepalived[3430]: Stopping
Jul 03 01:14:20 ds1 systemd[1]: Stopping LVS and VRRP High Availability Mon.....
Jul 03 01:14:20 ds1 Keepalived_vrrp[3432]: VRRP_Instance(VI_1) sent 0 priority
Jul 03 01:14:20 ds1 Keepalived_vrrp[3432]: VRRP_Instance(VI_1) removing prot....
Jul 03 01:14:20 ds1 Keepalived_healthcheckers[3431]: Removing service [192.16...
Jul 03 01:14:21 ds1 Keepalived_vrrp[3432]: Stopped
Jul 03 01:14:21 ds1 Keepalived[3430]: Stopped Keepalived v1.3.5 (03/19,2017...f2
Jul 03 01:14:21 ds1 systemd[1]: Stopped LVS and VRRP High Availability Monitor.
Hint: Some lines were ellipsized, use -l to show in full.
[root@ds1 ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
[root@ds1 ~]# 
~~~

客户端访问

~~~
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ 
~~~

依旧可以正常访问

所以在 LB 层面任意故障一台，同时 server 层面任意故障一台，整个集群依旧是可用的

## 恢复一台后端

~~~
[root@rs1 ~]# curl localhost
curl: (7) Failed connect to localhost:80; Connection refused
[root@rs1 ~]# curl localhost
curl: (7) Failed connect to localhost:80; Connection refused
[root@rs1 ~]# systemctl start nginx 
[root@rs1 ~]# curl localhost
rs1
[root@rs1 ~]# curl localhost
rs1
[root@rs1 ~]# 
~~~

keepalived 侦测到了这个变化，日志中有如下输出

~~~
...
...
Jul  3 14:05:11 ds2 Keepalived_healthcheckers[3458]: TCP connection to [192.168.56.183]:80 success.
Jul  3 14:05:11 ds2 Keepalived_healthcheckers[3458]: Adding service [192.168.56.183]:80 to VS [192.168.1.184]:80
...
...
~~~

查看 IPVS 配置

~~~
[root@ds2 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.184:80 wrr
  -> 192.168.1.183:80             Tunnel  1      0          4         
  -> 192.168.56.183:80            Tunnel  1      0          0         
  -> 192.168.56.184:80            Tunnel  1      0          4         
[root@ds2 ~]# 
~~~

自动加入了 rs1 的 IP

客户端访问

~~~
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ 
~~~

这时所有请求都自动转发给了所有后端，用户对这个过程是无感知的

对于运维来讲，这整个过程也是自动的，无需干预

## 恢复 keepalived 实例

~~~
[root@ds1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
[root@ds1 ~]# systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Jul 03 14:03:09 ds1 Keepalived[3407]: Stopping
Jul 03 14:03:09 ds1 systemd[1]: Stopping LVS and VRRP High Availability Mon.....
Jul 03 14:03:09 ds1 Keepalived_healthcheckers[3408]: Removing service [192.16...
Jul 03 14:03:09 ds1 Keepalived_healthcheckers[3408]: Removing service [192.16...
Jul 03 14:03:09 ds1 Keepalived_healthcheckers[3408]: Stopped
Jul 03 14:03:09 ds1 Keepalived_vrrp[3409]: VRRP_Instance(VI_1) sent 0 priority
Jul 03 14:03:09 ds1 Keepalived_vrrp[3409]: VRRP_Instance(VI_1) removing prot....
Jul 03 14:03:10 ds1 Keepalived_vrrp[3409]: Stopped
Jul 03 14:03:10 ds1 Keepalived[3407]: Stopped Keepalived v1.3.5 (03/19,2017...f2
Jul 03 14:03:10 ds1 systemd[1]: Stopped LVS and VRRP High Availability Monitor.
Hint: Some lines were ellipsized, use -l to show in full.
[root@ds1 ~]# systemctl start keepalived
[root@ds1 ~]# systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-07-03 14:11:32 UTC; 3s ago
  Process: 3463 ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 3464 (keepalived)
   CGroup: /system.slice/keepalived.service
           ├─3464 /usr/sbin/keepalived -D
           ├─3465 /usr/sbin/keepalived -D
           └─3466 /usr/sbin/keepalived -D

Jul 03 14:11:32 ds1 Keepalived_vrrp[3466]: VRRP sockpool: [ifindex(3), proto...]
Jul 03 14:11:33 ds1 Keepalived_vrrp[3466]: VRRP_Instance(VI_1) Transition to...E
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: VRRP_Instance(VI_1) Entering MAST...E
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: VRRP_Instance(VI_1) setting proto....
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: Sending gratuitous ARP on eth1 fo...4
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: VRRP_Instance(VI_1) Sending/queue...4
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: Sending gratuitous ARP on eth1 fo...4
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: Sending gratuitous ARP on eth1 fo...4
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: Sending gratuitous ARP on eth1 fo...4
Jul 03 14:11:34 ds1 Keepalived_vrrp[3466]: Sending gratuitous ARP on eth1 fo...4
Hint: Some lines were ellipsized, use -l to show in full.
[root@ds1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.184:80 wrr
  -> 192.168.1.183:80             Tunnel  1      0          0         
  -> 192.168.56.183:80            Tunnel  1      0          0         
  -> 192.168.56.184:80            Tunnel  1      0          0         
[root@ds1 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 77809sec preferred_lft 77809sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:fc:bf:30 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.181/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet 192.168.1.184/24 scope global secondary eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fefc:bf30/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b3:ae:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.181/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb3:ae0b/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds1 ~]# 
~~~

客户访问

~~~
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ curl 192.168.1.184
rs2
wilmos@Nothing:~$ curl 192.168.1.184
rs1
wilmos@Nothing:~$ curl 192.168.1.184
rs3
wilmos@Nothing:~$ 
~~~

对于客户来讲，是透明的，他并不知道现在 VIP 已经发生了迁移

事实上，我的连续测试，发现是会掉一个包的

但是一般而言对于线上的业务来讲，只要不是特别频繁的来回切换，掉一个包是完全可以接受的

协议层面和应用层面是会有重传机制来保障这个包的数据不丢失

---

# 总结

不得不说 keepalived + lvs 是一个优秀的开源 LB 解决方案

廉价且易于管理

LVS TUN 结合 keepalived 的调度方式，可以跨越机房限制，在很大范围里调度计算资源

其实 LVS TUN 的资源开销是比 LVS DR 要略大的，但是因为其可以轻易的进行大范围资源调度，所以可以实现特别大的流量承载

应该是目前开源解决方案中最大流量的前端 LB 调度器了

* TOC
{:toc}

---

[keepalived]:http://www.keepalived.org/
[lvs]:http://www.linuxvirtualserver.org/index.html


