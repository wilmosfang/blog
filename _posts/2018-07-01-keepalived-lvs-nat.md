---
layout: post
title: "Keepalived LVS NAT"
author:  wilmosfang
date: 2018-07-01 13:33:35
image: '/assets/img/'
excerpt: 'Keepalived LVS NAT 配置'
main-class: 'lvs'
color: '#3c4a6a'
tags:
 - lvs
 - nginx
 - keepalived
categories: 
 - lvs
twitter_text: 'simple process of Keepalived LVS NAT config'
introduction: 'Configration of Keepalived LVS NAT'
---

# 前言

**[Keepalived][keepalived]** 作为 LVS 的有效补充可以构建一个高可用的 LB 前端

>Keepalived is a routing software written in C. The main goal of this project is to provide simple and robust facilities for loadbalancing and high-availability to Linux system and Linux based infrastructures

**[Keepalived][keepalived]** 主要使用 VRRP 实现 VIP 的管理

>On the other hand high-availability is achieved by VRRP protocol. VRRP is a fundamental brick for router failover. In addition, Keepalived implements a set of hooks to the VRRP finite state machine providing low-level and high-speed protocol interactions. Keepalived frameworks can be used independently or all together to provide resilient infrastructures

LVS 只实现到了四层，**[Keepalived][keepalived]** 可以实现七层的简单检查，**[Keepalived][keepalived]** 可以通过预设的检查逻辑来管理 LVS 配置，从而实现对 LVS 自动且动态的调配，让整个 LB 系统更加灵活且健壮

这里演示一下如何配置 **[Keepalived][keepalived]** 加 **[LVS][lvs]** 的 NAT 模式

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
~~~

VIP 为 192.168.1.183

VRoute 为 192.168.56.185


**拓扑**

~~~
client--->ds1<-|->rs1
      `-->ds2<-|->rs2
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

## 开启 DS 转发

~~~
[root@ds1 ~]# cat /proc/sys/net/ipv4/ip_forward
0
[root@ds1 ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
[root@ds1 ~]# cat /proc/sys/net/ipv4/ip_forward
1
[root@ds1 ~]# 
~~~

ds2 上进行相同操作

~~~
[root@ds2 ~]# cat /proc/sys/net/ipv4/ip_forward
0
[root@ds2 ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
[root@ds2 ~]# cat /proc/sys/net/ipv4/ip_forward
1
[root@ds2 ~]# 
~~~

## 开启防火墙转发

~~~
[root@ds1 ~]# iptables -L -nv 
Chain INPUT (policy ACCEPT 5620 packets, 336K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 13942 packets, 732K bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@ds1 ~]# 
[root@ds1 ~]# iptables -A FORWARD -i eth2 -j ACCEPT
[root@ds1 ~]# iptables -t nat -A POSTROUTING -s 192.168.56.0/24  -j MASQUERADE 
[root@ds1 ~]# iptables -L -nv 
Chain INPUT (policy ACCEPT 30 packets, 2024 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  eth2   *       0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 54 packets, 3160 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@ds1 ~]# iptables -t nat -L -nv 
Chain PREROUTING (policy ACCEPT 1 packets, 84 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 1 packets, 84 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 22 packets, 1303 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 3 packets, 183 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   19  1120 MASQUERADE  all  --  *      *       192.168.56.0/24      0.0.0.0/0           
[root@ds1 ~]# 
~~~

ds2 上执行相同操作

~~~
[root@ds2 ~]# iptables -L -nv
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@ds2 ~]# iptables -t nat -L -nv
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@ds2 ~]# iptables -A FORWARD -i eth2 -j ACCEPT
[root@ds2 ~]# iptables -t nat -A POSTROUTING -s 192.168.56.0/24  -j MASQUERADE
[root@ds2 ~]# iptables -L -nv
Chain INPUT (policy ACCEPT 72 packets, 3144 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  eth2   *       0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 24 packets, 1504 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@ds2 ~]#  iptables -t nat -L -nv
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 16 packets, 976 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 1 packets, 76 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   15   900 MASQUERADE  all  --  *      *       192.168.56.0/24      0.0.0.0/0           
[root@ds2 ~]# 
~~~


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
        192.168.1.183/24
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface eth2
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 2222
    }
    virtual_ipaddress {
        192.168.56.185/24
    }
}

virtual_server 192.168.1.183 8800 {
    delay_loop 6
    lb_algo wrr
    lb_kind NAT
    persistence_timeout 0
    protocol TCP

    real_server 192.168.56.183 80 {
        weight 2
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
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.183/24
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface eth2
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 2222
    }
    virtual_ipaddress {
        192.168.56.185/24
    }
}

virtual_server 192.168.1.183 8800 {
    delay_loop 6
    lb_algo wrr
    lb_kind NAT
    persistence_timeout 0
    protocol TCP

    real_server 192.168.56.183 80 {
        weight 2
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
>     priority 99
35c35
<     state MASTER
---
>     state BACKUP
38c38
<     priority 100
---
>     priority 99
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
       valid_lft 69119sec preferred_lft 69119sec
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
   Active: active (running) since Sun 2018-07-01 16:12:53 UTC; 7s ago
  Process: 4248 ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 4249 (keepalived)
   CGroup: /system.slice/keepalived.service
           ├─4249 /usr/sbin/keepalived -D
           ├─4250 /usr/sbin/keepalived -D
           └─4251 /usr/sbin/keepalived -D

Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth2 fo...5
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth2 fo...5
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: VRRP_Instance(VI_1) Entering MAST...E
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: VRRP_Instance(VI_1) setting proto....
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth1 fo...3
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: VRRP_Instance(VI_1) Sending/queue...3
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth1 fo...3
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth1 fo...3
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth1 fo...3
Jul 01 16:12:58 ds2 Keepalived_vrrp[4251]: Sending gratuitous ARP on eth1 fo...3
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
       valid_lft 69093sec preferred_lft 69093sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d5:11:3f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.182/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet 192.168.1.183/24 scope global secondary eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed5:113f/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:93:b1:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.182/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet 192.168.56.185/24 scope global secondary eth2
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
       valid_lft 69003sec preferred_lft 69003sec
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
   Active: active (running) since Sun 2018-07-01 16:14:14 UTC; 5s ago
  Process: 4432 ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 4433 (keepalived)
   CGroup: /system.slice/keepalived.service
           ├─4433 /usr/sbin/keepalived -D
           ├─4434 /usr/sbin/keepalived -D
           └─4435 /usr/sbin/keepalived -D

Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth1 fo...3
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth1 fo...3
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_2) Entering MAST...E
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_2) setting proto....
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth2 fo...5
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_2) Sending/queue...5
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth2 fo...5
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth2 fo...5
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth2 fo...5
Jul 01 16:14:15 ds1 Keepalived_vrrp[4435]: Sending gratuitous ARP on eth2 fo...5
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
       valid_lft 68985sec preferred_lft 68985sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:fc:bf:30 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.181/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet 192.168.1.183/24 scope global secondary eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fefc:bf30/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b3:ae:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.181/24 brd 192.168.56.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet 192.168.56.185/24 scope global secondary eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb3:ae0b/64 scope link 
       valid_lft forever preferred_lft forever
[root@ds1 ~]# 
~~~

这时看看本地就产生了 LVS 的配置

~~~
[root@ds1 ~]# ipvsadm -L -n 
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.183:8800 wrr
  -> 192.168.56.183:80            Masq    2      0          0         
  -> 192.168.56.184:80            Masq    1      0          0         
[root@ds1 ~]# 
~~~

~~~
[root@ds2 ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.183:8800 wrr
  -> 192.168.56.183:80            Masq    2      0          0         
  -> 192.168.56.184:80            Masq    1      0          0         
[root@ds2 ~]#
~~~

此时还无法访问,这是因为 RS 上还没指定默认路由到 DS VRIP

## RS 添加默认路由

~~~
[root@rs1 ~]# ip route 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.183 metric 101 
[root@rs1 ~]# ping 192.168.1.6
connect: Network is unreachable
[root@rs1 ~]# 
[root@rs1 ~]# ip route 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.183 metric 101 
[root@rs1 ~]# ip route add default via 192.168.56.185 dev eth1
[root@rs1 ~]# ping 192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=63 time=1.13 ms
64 bytes from 192.168.1.6: icmp_seq=2 ttl=63 time=1.01 ms
^C
--- 192.168.1.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1004ms
rtt min/avg/max/mdev = 1.011/1.073/1.135/0.062 ms
[root@rs1 ~]# 
[root@rs1 ~]# ip route 
default via 192.168.56.185 dev eth1 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.183 metric 101 
[root@rs1 ~]#
~~~

rs2 上执行相同操作

~~~
[root@rs2 ~]# ip route 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 101 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.184 metric 100 
[root@rs2 ~]# ping 192.168.1.6
connect: Network is unreachable
[root@rs2 ~]# ip route add default via 192.168.56.185 dev eth1
[root@rs2 ~]# ping 192.168.1.6
PING 192.168.1.6 (192.168.1.6) 56(84) bytes of data.
64 bytes from 192.168.1.6: icmp_seq=1 ttl=63 time=0.414 ms
64 bytes from 192.168.1.6: icmp_seq=2 ttl=63 time=1.12 ms
^C
--- 192.168.1.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1007ms
rtt min/avg/max/mdev = 0.414/0.770/1.126/0.356 ms
[root@rs2 ~]# ip route 
default via 192.168.56.185 dev eth1 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 101 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.184 metric 100 
[root@rs2 ~]#
~~~

## 进行访问

~~~
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs1
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs1
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs1
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs1
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs1
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
Jul  1 16:25:21 ds1 Keepalived_healthcheckers[4434]: TCP connection to [192.168.56.183]:80 failed.
Jul  1 16:25:24 ds1 Keepalived_healthcheckers[4434]: TCP connection to [192.168.56.183]:80 failed.
Jul  1 16:25:24 ds1 Keepalived_healthcheckers[4434]: Check on service [192.168.56.183]:80 failed after 1 retry.
Jul  1 16:25:24 ds1 Keepalived_healthcheckers[4434]: Removing service [192.168.56.183]:80 from VS [192.168.1.183]:8800
...
...
~~~

查看 IPVS 配置

~~~
[root@ds1 ~]# ipvsadm -L -n 
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.183:8800 wrr
  -> 192.168.56.184:80            Masq    1      0          0         
[root@ds1 ~]# 
~~~

少了 rs1 的 IP

客户端访问

~~~
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ 
~~~

并不影响客户端访问，只是所有请求都转发给了 rs2

## 停止 keepalived master

~~~
[root@ds1 ~]# systemctl stop keepalived
[root@ds1 ~]# systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Jul 01 16:25:24 ds1 Keepalived_healthcheckers[4434]: Check on service [192.16...
Jul 01 16:25:24 ds1 Keepalived_healthcheckers[4434]: Removing service [192.16...
Jul 01 16:28:36 ds1 Keepalived[4433]: Stopping
Jul 01 16:28:36 ds1 systemd[1]: Stopping LVS and VRRP High Availability Mon.....
Jul 01 16:28:36 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_1) sent 0 priority
Jul 01 16:28:36 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_1) removing prot....
Jul 01 16:28:36 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_2) sent 0 priority
Jul 01 16:28:36 ds1 Keepalived_vrrp[4435]: VRRP_Instance(VI_2) removing prot....
Jul 01 16:28:37 ds1 Keepalived[4433]: Stopped Keepalived v1.3.5 (03/19,2017...f2
Jul 01 16:28:37 ds1 systemd[1]: Stopped LVS and VRRP High Availability Monitor.
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
       valid_lft 68116sec preferred_lft 68116sec
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
[root@ds1 ~]# 
~~~

客户端访问

~~~
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ curl  192.168.1.183:8800
rs2
wilmos@Nothing:~$ 
~~~

依旧可以正常访问

所以在 LB 层面任意故障一台，同时 server 层面任意故障一台，整个集群依旧是可用的

---

# 总结

keepalived + lvs 的确是一个优秀的开源 LB 解决方案

廉价且易于管理

后面再演示 LVS TUN 结合 keepalived 的调度方式


* TOC
{:toc}

---

[keepalived]:http://www.keepalived.org/
[lvs]:http://www.linuxvirtualserver.org/index.html
