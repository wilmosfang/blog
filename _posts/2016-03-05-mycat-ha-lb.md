---
layout: post
title:  Mycat HA(高可用) 与 LB(负载均衡)
author: wilmosfang
categories:  mycat
tags:   cluster mysql mycat  keepalived haproxy rsyslog
wc: 1524 5439 63228
excerpt: keepalived、haproxy、mycat的下载安装配置运行，rsyslog的配置运行与测试，简单的haproxy健康检查，haproxy监控，系统访问测试，切换过程中的影响，和部署过程中的注意事项
comments: true
---



# 前言

**[Mycat][mycat]** 是一款开源的数据库分库分表中间件

一般而言在生产环境下，任何基础架构都有必要考虑 **高可用，扩展性和监控方案** :

* 监控：使用 **Mycat web** (也有叫 **Mycat-eye** )可以实现对mycat的 **监控** 
* 高可用：**[Mycat][mycat]** 目前没有官方的高可用解决方案 ，但配合使用 **keepalived** 和 **haproxy** 也可以实现mycat的 **高可用** 
* 可扩展：由于 **[Mycat][mycat]** 本身是无状态的，可以通过添加 **Mycat** 节点来实现 **水平扩展** ，从而分摊访问压力

下面分享一下 **Mycat 高可用与负载均衡** 的实现方法，详细内容可以参考 **[官方文档][mycat_doc]** (但是由于官方文档比较老，有不少坑，这篇分享里会将这些坑填平)


> **Tip:** 当前的最新版本为 **Mycat server 1.5 GA** 、 **Keepalived for Linux - Version 1.2.19** 、 **HAProxy 1.6.3**

---


# 概要

* TOC
{:toc}



---

## Mycat 高可用架构


Mycat 是无状态的，可以使用 HAProxy 或四层交换机等设备构建 Mycat 高可用集群，后端 Mysql 则配置为主从同步 (replication 、mha 或 galary 集群)，那么Mycat层和Mysql层就都是高可用的，整个分布式数据库系统就是高可用的


![mycat_ha1.png](/images/mycat_ha/mycat_ha1.png)


![mycat_ha2.png](/images/mycat_ha/mycat_ha2.png)


没有四层交换机的环境下，为了实现系统架构的高扩展性，可以使用 **LVS** 或 **HAProxy** 替代 (开源软件最显著的好处就是便宜) ，不过引入了四层(TCP层)交换逻辑或服务后，又会增加此层的单点风险，为了有效规避，可以使用 **keepalived** 来进行故障转移(故障判断和服务漂移)


![mycat_ha3.png](/images/mycat_ha/mycat_ha3.png)


![mycat_ha4.png](/images/mycat_ha/mycat_ha4.png)


这样，HAProxy 层和 Mycat 层任意宕机一半，整个系统依旧可以正常运转

由于对业务层透明，个别请求在实际IP切换的过程中可能会有一次timeout，但自动重发请求就能恢复正常，一般应用也都有重发机制

---

## 下载安装keepalived


**[keepalived][keepalived]** 项目是为了结合 **LVS** 在Linux平台上构建简单而健壮的高可用负载均衡系统而产生的，不过 **[keepalived][keepalived]** 也可以单独出来作为构建高可用系统的基础服务，对 **浮动IP** (VIP，也有叫服务IP)进行管理

**[keepalived][keepalived]** 的 **[下载地址][keepalived_dl]**


### 下载

~~~
[root@h101 keepalived]# wget  http://www.keepalived.org/software/keepalived-1.2.19.tar.gz
--2016-03-02 15:26:58--  http://www.keepalived.org/software/keepalived-1.2.19.tar.gz
Resolving www.keepalived.org... 37.59.63.157, 2001:41d0:8:7a9d::1
Connecting to www.keepalived.org|37.59.63.157|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 330164 (322K) [application/x-gzip]
Saving to: “keepalived-1.2.19.tar.gz”

100%[============================================================================>] 330,164     35.7K/s   in 10s     

2016-03-02 15:27:11 (32.1 KB/s) - “keepalived-1.2.19.tar.gz” saved [330164/330164]

[root@h101 keepalived]# 
~~~

### 解压

~~~
[root@h101 keepalived]# tar -zxvf keepalived-1.2.19.tar.gz 
keepalived-1.2.19/
keepalived-1.2.19/keepalived.spec.in
keepalived-1.2.19/Makefile.in
keepalived-1.2.19/genhash/
...
...
keepalived-1.2.19/lib/vector.c
keepalived-1.2.19/lib/scheduler.h
keepalived-1.2.19/lib/signals.c
keepalived-1.2.19/TODO
[root@h101 keepalived]# ls
keepalived-1.2.19  keepalived-1.2.19.tar.gz
[root@h101 keepalived]# cd keepalived-1.2.19
[root@h101 keepalived-1.2.19]# ls
AUTHOR  ChangeLog  configure.in  COPYING  genhash  install-sh  keepalived.spec.in  Makefile.in  TODO
bin     configure  CONTRIBUTORS  doc      INSTALL  keepalived  lib                 README       VERSION
[root@h101 keepalived-1.2.19]# 
~~~

### 安装

使用下列命令进行安装

~~~
./configure --prefix=/usr/local/keepalived
make 
make install
~~~

> **Tip:** 可以使用 **`echo $?`** 来确认执行结果

#### 详细安装过程

~~~
[root@h101 keepalived-1.2.19]# ls
AUTHOR  ChangeLog  configure.in  COPYING  genhash  install-sh  keepalived.spec.in  Makefile.in  TODO
bin     configure  CONTRIBUTORS  doc      INSTALL  keepalived  lib                 README       VERSION
[root@h101 keepalived-1.2.19]# ./configure --prefix=/usr/local/keepalived  
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking for a BSD-compatible install... /usr/bin/install -c
checking for strip... strip
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for ANSI C header files... yes
checking for sys/wait.h that is POSIX.1 compatible... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking fcntl.h usability... yes
checking fcntl.h presence... yes
checking for fcntl.h... yes
checking syslog.h usability... yes
checking syslog.h presence... yes
checking for syslog.h... yes
checking for unistd.h... (cached) yes
checking sys/ioctl.h usability... yes
checking sys/ioctl.h presence... yes
checking for sys/ioctl.h... yes
checking sys/time.h usability... yes
checking sys/time.h presence... yes
checking for sys/time.h... yes
checking openssl/ssl.h usability... yes
checking openssl/ssl.h presence... yes
checking for openssl/ssl.h... yes
checking openssl/md5.h usability... yes
checking openssl/md5.h presence... yes
checking for openssl/md5.h... yes
checking openssl/err.h usability... yes
checking openssl/err.h presence... yes
checking for openssl/err.h... yes
checking whether ETHERTYPE_IPV6 is declared... yes
checking for crypt in -lcrypt... yes
checking for MD5_Init in -lcrypto... yes
checking for SSL_CTX_new in -lssl... yes
checking for nl_socket_alloc in -lnl-3... no
checking for nl_socket_modify_cb in -lnl... no
configure: WARNING: keepalived will be built without libnl support.
checking for kernel version... 2.6.32
checking for IPVS syncd support... yes
checking for kernel macvlan support... yes
checking whether SO_MARK is declared... yes
checking for an ANSI C-conforming const... yes
checking for pid_t... yes
checking whether time.h and sys/time.h may both be included... yes
checking whether gcc needs -traditional... no
checking for working memcmp... yes
checking return type of signal handlers... void
checking for gettimeofday... yes
checking for select... yes
checking for socket... yes
checking for strerror... yes
checking for strtol... yes
checking for uname... yes
configure: creating ./config.status
config.status: creating Makefile
config.status: creating genhash/Makefile
config.status: creating keepalived/core/Makefile
config.status: creating lib/config.h
config.status: creating keepalived.spec
config.status: creating keepalived/Makefile
config.status: creating lib/Makefile
config.status: creating keepalived/vrrp/Makefile
config.status: creating keepalived/check/Makefile
config.status: creating keepalived/libipvs-2.6/Makefile

Keepalived configuration
------------------------
Keepalived version       : 1.2.19
Compiler                 : gcc
Compiler flags           : -g -O2
Extra Lib                : -lssl -lcrypto -lcrypt 
Use IPVS Framework       : Yes
IPVS sync daemon support : Yes
IPVS use libnl           : No
fwmark socket support    : Yes
Use VRRP Framework       : Yes
Use VRRP VMAC            : Yes
SNMP support             : No
SHA1 support             : No
Use Debug flags          : No
[root@h101 keepalived-1.2.19]# echo $?
0
[root@h101 keepalived-1.2.19]# make 
make -C lib || exit 1;
make[1]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/lib'
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c memory.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c utils.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c notify.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c timer.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c scheduler.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c vector.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c list.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c html.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c parser.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c signals.c
gcc -I. -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_WITHOUT_SNMP_ -c logger.c
make[1]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/lib'
make -C keepalived
make[1]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived'
make[2]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/core'
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c main.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c daemon.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c pidfile.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c layer4.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c smtp.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c global_data.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c global_parser.c
make[2]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/core'
make[2]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/check'
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_daemon.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_data.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_parser.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_api.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_tcp.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_http.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_ssl.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_smtp.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c check_misc.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c ipwrapper.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_WITH_VRRP_ -D_WITHOUT_SNMP_ -D_WITH_SO_MARK_  -c ipvswrapper.c
make[2]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/check'
make[2]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/vrrp'
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_daemon.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_print.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_data.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_parser.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_notify.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_scheduler.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_sync.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_index.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_netlink.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_arp.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_if.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_track.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_ipaddress.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_iproute.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_ipsecah.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_ndisc.c
gcc -I../include -I../../lib -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes -D_KRNL_2_6_ -D_WITH_LVS_ -D_HAVE_IPVS_SYNCD_ -D_HAVE_VRRP_VMAC_ -D_WITHOUT_SNMP_  -c vrrp_vmac.c
make[2]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/vrrp'
make[2]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/libipvs-2.6'
gcc -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -DLIBIPVS_DONTUSE_NL -Wall -Wunused -c -o libipvs.o libipvs.c
gcc -g -O2  -I/usr/src/linux/include -I/usr/src/linux/include -DLIBIPVS_DONTUSE_NL -Wall -Wunused -c -o ip_vs_nl_policy.o ip_vs_nl_policy.c
ar rv libipvs.a libipvs.o ip_vs_nl_policy.o
ar: creating libipvs.a
a - libipvs.o
a - ip_vs_nl_policy.o
rm libipvs.o ip_vs_nl_policy.o
make[2]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived/libipvs-2.6'
Building ../bin/keepalived
strip ../bin/keepalived

Make complete
make[1]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived'
make -C genhash
make[1]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/genhash'
gcc -I../lib -g -O2 -D_WITH_SO_MARK_  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes   -c -o main.o main.c
gcc -I../lib -g -O2 -D_WITH_SO_MARK_  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes   -c -o sock.o sock.c
gcc -I../lib -g -O2 -D_WITH_SO_MARK_  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes   -c -o layer4.o layer4.c
gcc -I../lib -g -O2 -D_WITH_SO_MARK_  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes   -c -o http.o http.c
gcc -I../lib -g -O2 -D_WITH_SO_MARK_  -I/usr/src/linux/include -I/usr/src/linux/include -Wall -Wunused -Wstrict-prototypes   -c -o ssl.o ssl.c
Building ../bin/genhash
strip ../bin/genhash

Make complete
make[1]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/genhash'

Make complete
[root@h101 keepalived-1.2.19]# echo $?
0
[root@h101 keepalived-1.2.19]# make install 
make -C keepalived install
make[1]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived'
install -d /usr/local/keepalived/sbin
install -m 700 ../bin/keepalived /usr/local/keepalived/sbin/
install -d /usr/local/keepalived/etc/rc.d/init.d
install -m 755 etc/init.d/keepalived.init /usr/local/keepalived/etc/rc.d/init.d/keepalived
install -d /usr/local/keepalived/etc/sysconfig
install -m 644 etc/init.d/keepalived.sysconfig /usr/local/keepalived/etc/sysconfig/keepalived
install -d /usr/local/keepalived/etc/keepalived/samples
install -m 644 etc/keepalived/keepalived.conf /usr/local/keepalived/etc/keepalived/
install -m 644 ../doc/samples/* /usr/local/keepalived/etc/keepalived/samples/
install -d /usr/local/keepalived/share/man/man5
install -d /usr/local/keepalived/share/man/man8
install -m 644 ../doc/man/man5/keepalived.conf.5 /usr/local/keepalived/share/man/man5
install -m 644 ../doc/man/man8/keepalived.8 /usr/local/keepalived/share/man/man8
make[1]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/keepalived'
make -C genhash install
make[1]: Entering directory `/usr/local/src/keepalived/keepalived-1.2.19/genhash'
install -d /usr/local/keepalived/bin
install -m 755 ../bin/genhash /usr/local/keepalived/bin/
install -d /usr/local/keepalived/share/man/man1
install -m 644 ../doc/man/man1/genhash.1 /usr/local/keepalived/share/man/man1
make[1]: Leaving directory `/usr/local/src/keepalived/keepalived-1.2.19/genhash'
[root@h101 keepalived-1.2.19]# echo $?
0
[root@h101 keepalived-1.2.19]# ll /usr/local/keepalived/
total 16
drwxr-xr-x 2 root root 4096 Mar  2 15:58 bin
drwxr-xr-x 5 root root 4096 Mar  2 15:58 etc
drwxr-xr-x 2 root root 4096 Mar  2 15:58 sbin
drwxr-xr-x 3 root root 4096 Mar  2 15:58 share
[root@h101 keepalived-1.2.19]# 
~~~

#### 目录结构

~~~
[root@h101 sbin]# tree /usr/local/keepalived/
/usr/local/keepalived/
├── bin
│   └── genhash
├── etc
│   ├── keepalived
│   │   ├── keepalived.conf
│   │   └── samples
│   │       ├── client.pem
│   │       ├── dh1024.pem
│   │       ├── keepalived.conf.fwmark
│   │       ├── keepalived.conf.HTTP_GET.port
│   │       ├── keepalived.conf.inhibit
│   │       ├── keepalived.conf.IPv6
│   │       ├── keepalived.conf.misc_check
│   │       ├── keepalived.conf.misc_check_arg
│   │       ├── keepalived.conf.quorum
│   │       ├── keepalived.conf.sample
│   │       ├── keepalived.conf.SMTP_CHECK
│   │       ├── keepalived.conf.SSL_GET
│   │       ├── keepalived.conf.status_code
│   │       ├── keepalived.conf.track_interface
│   │       ├── keepalived.conf.virtualhost
│   │       ├── keepalived.conf.virtual_server_group
│   │       ├── keepalived.conf.vrrp
│   │       ├── keepalived.conf.vrrp.localcheck
│   │       ├── keepalived.conf.vrrp.lvs_syncd
│   │       ├── keepalived.conf.vrrp.routes
│   │       ├── keepalived.conf.vrrp.scripts
│   │       ├── keepalived.conf.vrrp.static_ipaddress
│   │       ├── keepalived.conf.vrrp.sync
│   │       ├── root.pem
│   │       └── sample.misccheck.smbcheck.sh
│   ├── rc.d
│   │   └── init.d
│   │       └── keepalived
│   └── sysconfig
│       └── keepalived
├── sbin
│   └── keepalived
└── share
    └── man
        ├── man1
        │   └── genhash.1
        ├── man5
        │   └── keepalived.conf.5
        └── man8
            └── keepalived.8

13 directories, 33 files
[root@h101 sbin]# 
~~~

### 版本确认

~~~
[root@h101 sbin]# /usr/local/keepalived/sbin/keepalived  -v 
Keepalived v1.2.19 (03/02,2016)
[root@h101 sbin]# 
~~~

---


## 下载安装HAProxy

**[HAProxy][haproxy]** 是一个稳定且开源的高性能 TCP/HTTP 负载均衡代理软件

**[HAProxy][haproxy]** 的 **[下载地址][haproxy_dl]**

> **Tip:** 访问 **[HAProxy][haproxy]** 和 **[下载地址][haproxy_dl]**，可能会被墙，所以准备好VPN或代理

### 解压

~~~
[root@h101 haproxy]# ls
haproxy-1.6.3.tar.gz
[root@h101 haproxy]# md5sum haproxy-1.6.3.tar.gz 
3362d1e268c78155c2474cb73e7f03f9  haproxy-1.6.3.tar.gz
[root@h101 haproxy]# tar -zxvf haproxy-1.6.3.tar.gz 
haproxy-1.6.3/
haproxy-1.6.3/.gitignore
haproxy-1.6.3/CHANGELOG
haproxy-1.6.3/CONTRIBUTING
haproxy-1.6.3/LICENSE
haproxy-1.6.3/MAINTAINERS
haproxy-1.6.3/Makefile
haproxy-1.6.3/README
...
...
haproxy-1.6.3/tests/test.c
haproxy-1.6.3/tests/test_hashes.c
haproxy-1.6.3/tests/test_pools.c
haproxy-1.6.3/tests/testinet.c
haproxy-1.6.3/tests/uri_hash.c
[root@h101 haproxy]# ls
haproxy-1.6.3  haproxy-1.6.3.tar.gz
[root@h101 haproxy]# ll haproxy-1.6.3
total 488
-rw-rw-r--  1 root root 351146 Dec 27 22:04 CHANGELOG
drwxrwxr-x 10 root root   4096 Dec 27 22:04 contrib
-rw-rw-r--  1 root root  36238 Dec 27 22:04 CONTRIBUTING
drwxrwxr-x  5 root root   4096 Dec 27 22:04 doc
drwxrwxr-x  2 root root   4096 Dec 27 22:04 ebtree
drwxrwxr-x  3 root root   4096 Dec 27 22:04 examples
drwxrwxr-x  6 root root   4096 Dec 27 22:04 include
-rw-rw-r--  1 root root   2029 Dec 27 22:04 LICENSE
-rw-rw-r--  1 root root   2216 Dec 27 22:04 MAINTAINERS
-rw-rw-r--  1 root root  31547 Dec 27 22:04 Makefile
-rw-rw-r--  1 root root  22419 Dec 27 22:04 README
-rw-rw-r--  1 root root   3305 Dec 27 22:04 ROADMAP
drwxrwxr-x  2 root root   4096 Dec 27 22:04 src
-rw-rw-r--  1 root root     14 Dec 27 22:04 SUBVERS
drwxrwxr-x  2 root root   4096 Dec 27 22:04 tests
-rw-rw-r--  1 root root     24 Dec 27 22:04 VERDATE
-rw-rw-r--  1 root root      6 Dec 27 22:04 VERSION
[root@h101 haproxy]# 
~~~


### 编译安装

使用下面命令进行编译安装

~~~
make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy
make install PREFIX=/usr/local/haproxy
~~~

#### 详细安装过程

~~~
[root@h101 haproxy]# cd haproxy-1.6.3
[root@h101 haproxy-1.6.3]# ls
CHANGELOG  CONTRIBUTING  ebtree    include  MAINTAINERS  README   src      tests    VERSION
contrib    doc           examples  LICENSE  Makefile     ROADMAP  SUBVERS  VERDATE
[root@h101 haproxy-1.6.3]# make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy
gcc -Iinclude -Iebtree -Wall -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement       -DCONFIG_HAP_LINUX_SPLICE -DTPROXY -DCONFIG_HAP_LINUX_TPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DUSE_CPU_AFFINITY -DASSUME_SPLICE_WORKS -DUSE_ACCEPT4 -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.6.3\" -DCONFIG_HAPROXY_DATE=\"2015/12/25\" \
	      -DBUILD_TARGET='"linux2628"' \
	      -DBUILD_ARCH='"x86_64"' \
	      -DBUILD_CPU='"generic"' \
	      -DBUILD_CC='"gcc"' \
	      -DBUILD_CFLAGS='"-m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement"' \
	      -DBUILD_OPTIONS='""' \
	       -c -o src/haproxy.o src/haproxy.c
gcc -Iinclude -Iebtree -Wall -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement       -DCONFIG_HAP_LINUX_SPLICE -DTPROXY -DCONFIG_HAP_LINUX_TPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DUSE_CPU_AFFINITY -DASSUME_SPLICE_WORKS -DUSE_ACCEPT4 -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.6.3\" -DCONFIG_HAPROXY_DATE=\"2015/12/25\" -c -o src/base64.o src/base64.c
...
...
gcc -Iinclude -Iebtree -Wall -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement       -DCONFIG_HAP_LINUX_SPLICE -DTPROXY -DCONFIG_HAP_LINUX_TPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DUSE_CPU_AFFINITY -DASSUME_SPLICE_WORKS -DUSE_ACCEPT4 -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.6.3\" -DCONFIG_HAPROXY_DATE=\"2015/12/25\" \
	      -DSBINDIR='"/usr/local/haproxy/sbin"' \
	       -c -o src/haproxy-systemd-wrapper.o src/haproxy-systemd-wrapper.c
gcc -m64 -march=x86-64 -g -o haproxy-systemd-wrapper src/haproxy-systemd-wrapper.o   -lcrypt -ldl 
[root@h101 haproxy-1.6.3]# echo $?
0
[root@h101 haproxy-1.6.3]# make install PREFIX=/usr/local/haproxy
install -d "/usr/local/haproxy/sbin"
install haproxy  "/usr/local/haproxy/sbin"
install -d "/usr/local/haproxy/share/man"/man1
install -m 644 doc/haproxy.1 "/usr/local/haproxy/share/man"/man1
install -d "/usr/local/haproxy/doc/haproxy"
for x in architecture close-options configuration cookie-options intro linux-syn-cookies lua management network-namespaces proxy-protocol; do \
		install -m 644 doc/$x.txt "/usr/local/haproxy/doc/haproxy" ; \
	done
[root@h101 haproxy-1.6.3]# echo $?
0
[root@h101 haproxy-1.6.3]#
~~~

#### 目录结构

~~~
[root@h101 haproxy-1.6.3]# ll /usr/local/haproxy/
total 12
drwxr-xr-x 3 root root 4096 Mar  2 16:23 doc
drwxr-xr-x 2 root root 4096 Mar  2 16:23 sbin
drwxr-xr-x 3 root root 4096 Mar  2 16:23 share
[root@h101 haproxy-1.6.3]# tree /usr/local/haproxy/
/usr/local/haproxy/
├── doc
│   └── haproxy
│       ├── architecture.txt
│       ├── close-options.txt
│       ├── configuration.txt
│       ├── cookie-options.txt
│       ├── intro.txt
│       ├── linux-syn-cookies.txt
│       ├── lua.txt
│       ├── management.txt
│       ├── network-namespaces.txt
│       └── proxy-protocol.txt
├── sbin
│   └── haproxy
└── share
    └── man
        └── man1
            └── haproxy.1

6 directories, 12 files
[root@h101 haproxy-1.6.3]#
~~~


### 版本确认

~~~
[root@h101 ~]# /usr/local/haproxy/sbin/haproxy -v
HA-Proxy version 1.6.3 2015/12/25
Copyright 2000-2015 Willy Tarreau <willy@haproxy.org>

[root@h101 ~]# 
~~~


---


## 配置rsyslog日志

日志是可选的，因为日志并不是系统正常运转的必要基础，但是有了日志可以更有效理解系统当前的状态，出现问题后通过日志可以高效定位，所以是间接提升了系统的可用性(通过人力间接提高)，系统的高可用，不能只考虑到服务器，运维人员同样是考虑对象，任何可以帮助运维人员减少误操作，或提升恢复效率的努力都是值得的


确保系统中有 **rsyslog** 包

~~~
[root@h101 ~]# rpm -qa | grep rsyslog
rsyslog-5.8.10-8.el6.x86_64
[root@h101 ~]# 
~~~

> **Tip:** Centos6 以后系统都默认使用 rsyslog 来管理日志，当前的最新版为 **rsyslog-8.16.0**


### 添加haproxy配置


当前配置

~~~
[root@h101 ~]# grep -v "^#" /etc/rsyslog.conf  | grep -v "^$"
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$IncludeConfig /etc/rsyslog.d/*.conf
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
authpriv.*                                              /var/log/secure
mail.*                                                  -/var/log/maillog
cron.*                                                  /var/log/cron
*.emerg                                                 *
uucp,news.crit                                          /var/log/spooler
local7.*                                                /var/log/boot.log
$template SpiceTmpl,"%TIMESTAMP%.%TIMESTAMP:::date-subseconds% %syslogtag% %syslogseverity-text%:%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
:programname, startswith, "spice-vdagent"	/var/log/spice-vdagent.log;SpiceTmpl
[root@h101 ~]#
~~~

当前配置中有一条 **`$IncludeConfig /etc/rsyslog.d/*.conf`** , 代表所有在 **/etc/rsyslog.d/** 中以 **conf** 结尾的配置会被合并进来，于是为了便于管理，我们单独为 haproxy 创建一个配置文件

~~~
[root@h101 ~]# vim /etc/rsyslog.d/haproxy.conf 
[root@h101 ~]# cat /etc/rsyslog.d/haproxy.conf 
$ModLoad imudp
$UDPServerRun 514

local0.* /var/log/haproxy.log
[root@h101 ~]# 
~~~

Item     | Comment
-------- | ---
`$ModLoad imudp`| 加载UDP输入模块 **imudp**
`$UDPServerRun 514`| 使用UDP的514端口(一般默认是使用这个端口，也就是其它程序不明确指定的情况下也是尝试连接这个端口，如果改为其它端口，写日志的程序要在配置里明确指出，否则没法成功写入)
`local0.* /var/log/haproxy.log`| 自定义一个 **local0** 类别，这个类别的所有级别报警都记录到 **/var/log/haproxy.log** 文件中




### 重启rsyslog服务

~~~
[root@h101 ~]# ll /var/log/ha*
ls: cannot access /var/log/ha*: No such file or directory
[root@h101 ~]# /etc/init.d/rsyslog restart 
Shutting down system logger:                               [  OK  ]
Starting system logger:                                    [  OK  ]
[root@h101 ~]# ll /var/log/ha*
-rw------- 1 root root 0 Mar  3 17:32 /var/log/haproxy.log
[root@h101 ~]# netstat  -antulp | grep 514
tcp        0      0 192.168.100.101:22          192.168.100.1:49514         ESTABLISHED 4491/sshd           
udp        0      0 0.0.0.0:514                 0.0.0.0:*                               43095/rsyslogd      
udp        0      0 :::514                      :::*                                    43095/rsyslogd      
[root@h101 ~]# 
~~~

可以看到多出了一个日志文件 **/var/log/haproxy.log** ，同时也打开了 UDP 的 514 端口


### 测试写日志

我们可以使用 **logger**  命令来测试配置

~~~
[root@h101 ~]# logger -it test -p local0.info "test"
[root@h101 ~]# 
----------
[root@h101 ~]# tail -f /var/log/haproxy.log 
Mar  4 17:40:21 h101 test[44940]: test
...
...
...
~~~

在一个窗口中输入 **`logger -it test -p local0.info "test"`** ， 跟踪 **/var/log/haproxy.log** 文件可以看到产生了我定制的信息


---

## 安装mycat

### 下载

~~~
[root@h101 mycat]# rsync  -av root@192.168.100.102:/usr/local/src/mycat/Mycat-server-1.5-GA-20160217103036-linux.tar.gz  . 
root@192.168.100.102's password: 
receiving incremental file list
Mycat-server-1.5-GA-20160217103036-linux.tar.gz

sent 30 bytes  received 11478842 bytes  3279677.71 bytes/sec
total size is 11477321  speedup is 1.00
[root@h101 mycat]# ls
Mycat-server-1.5-GA-20160217103036-linux.tar.gz
[root@h101 mycat]# 
~~~

### 解压

~~~
[root@h101 mycat]# tar -zxvf Mycat-server-1.5-GA-20160217103036-linux.tar.gz 
mycat/bin/wrapper-linux-ppc-64
mycat/bin/wrapper-linux-x86-64
mycat/bin/wrapper-linux-x86-32
mycat/bin/mycat
...
...
mycat/bin/rehash.sh
mycat/bin/xml_to_yaml.sh
mycat/logs/
mycat/catlet/
[root@h101 mycat]#
~~~

### 环境确认

~~~
[root@h101 mycat]# java -version
java version "1.7.0_65"
OpenJDK Runtime Environment (rhel-2.5.1.2.el6_5-x86_64 u65-b17)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
[root@h101 mycat]# 
~~~



---

## 配置mycat


### **wrapper.conf**

~~~
[root@h101 conf]# cat wrapper.conf | egrep "(Xm|MaxDirectMemorySize)"
#wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G
wrapper.java.additional.5=-XX:MaxDirectMemorySize=256m
#wrapper.java.additional.10=-Xmx4G
wrapper.java.additional.10=-Xmx512m
#wrapper.java.additional.11=-Xms1G
wrapper.java.additional.11=-Xms128m
[root@h101 conf]# 
~~~

### **server.xml、schema.xml、rule.xml**

~~~
--[server.xml]--------
        <user name="cc">
                <property name="password">cc</property>
                <property name="schemas">cctest</property>
        </user>
--[schema.xml]--------
        <schema name="cctest" checkSQLschema="false" sqlMaxLimit="100">
                <table name="catworld"  dataNode="sd1,sd2,sd3"  rule="mod-long" />
                <table name="abc"  dataNode="sd1,sd2,sd3,sd5"  rule="mod4-long" />
        </schema>

        <dataNode name="sd1" dataHost="h101" database="my1" />
        <dataNode name="sd2" dataHost="h101" database="my2" />
        <dataNode name="sd3" dataHost="h101" database="my3" />
        <dataNode name="sd5" dataHost="h101" database="my5" />

       ...
       ...

        <dataHost name="h101" maxCon="100" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <writeHost host="h101M1" url="192.168.100.101:3306" user="root" password="mysql">
                <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>
--[rule.xml]--------
	<tableRule name="mod-long">
		<rule>
			<columns>id</columns>
			<algorithm>mod-long</algorithm>
		</rule>
	</tableRule>
	<tableRule name="mod4-long">
		<rule>
			<columns>id</columns>
			<algorithm>mod4-long</algorithm>
		</rule>
	</tableRule>
	...
	...
	<function name="mod-long" class="org.opencloudb.route.function.PartitionByMod">
		<!-- how many data nodes -->
		<property name="count">3</property>
	</function>
	<function name="mod4-long" class="org.opencloudb.route.function.PartitionByMod">
		<!-- how many data nodes -->
		<property name="count">4</property>
	</function>
~~~

### 打开防火墙

确保 **8066** 开启

~~~
[root@h101 conf]# iptables -L -nv  | grep 8066
[root@h101 conf]# vim /etc/sysconfig/iptables
[root@h101 conf]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h101 conf]# iptables -L -nv  | grep 8066
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8066 
[root@h101 conf]# 
~~~

以相同的方式打开 **9066、8888、9999**


Port | Comment
-------- | ---
8066 | 默认服务端口 `<property name="serverPort">8066</property>`
9066 | 默认管理端口 `<property name="managerPort">9066</property>`
8888 | haproxy对外的mycat服务端口
9999 | haproxy对外的mycat管理端口


~~~
[root@h101 ~]# iptables -L -nv | grep -E "(8066|9066|8888|9999)"
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8066 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:9066 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8888 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:9999 
[root@h101 ~]# 
~~~

---

## 启动mycat

~~~
[root@h101 bin]# ./mycat  start 
Starting Mycat-server...
[root@h101 bin]#
[root@h101 bin]# mysql -u cc -p -P 8066 -p -h 192.168.100.101
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.8-mycat-1.5-GA-20160217103036 MyCat Server (OpenCloundDB)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+----------+
| DATABASE |
+----------+
| cctest   |
+----------+
1 row in set (0.00 sec)

mysql> use cctest;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables in cctest |
+------------------+
| abc              |
| catworld         |
+------------------+
2 rows in set (0.00 sec)

mysql> 
mysql> select * from abc;
+----+------+
| id | name |
+----+------+
|  2 | abc  |
|  6 | abc  |
| 10 | abc  |
|  4 | abc  |
|  8 | abc  |
|  1 | abc  |
|  5 | abc  |
|  9 | xxx  |
+----+------+
8 rows in set (0.53 sec)

mysql>
~~~

> **Tip:**  密切关注 **mycat.log** 和 **wrapper.log** ，根据日志信息确认启动成功，如果有错误进行相应调整

---


## 配置haproxy


### 添加haproxy用户

添加一个 **haproxy** 用户，并赋权

~~~
[root@h101 haproxy]# grep proxy /etc/passwd
[root@h101 haproxy]# useradd haproxy
[root@h101 haproxy]# grep proxy /etc/passwd
haproxy:x:505:506::/home/haproxy:/bin/bash
[root@h101 haproxy]# chown -R haproxy.haproxy /usr/local/haproxy/
[root@h101 haproxy]# ll /usr/local/haproxy/
total 16
drwxr-xr-x 3 haproxy haproxy 4096 Mar  2 16:23 doc
drwxr-xr-x 2 haproxy haproxy 4096 Mar  2 16:23 sbin
drwxr-xr-x 3 haproxy haproxy 4096 Mar  2 16:23 share
[root@h101 haproxy]#
~~~

### 配置haproxy


~~~
[root@h101 ~]# cd /usr/local/haproxy/
[root@h101 haproxy]# vim haproxy.cfg 
[root@h101 haproxy]# grep -v "^#" haproxy.cfg 
global
 log 127.0.0.1 local0
 maxconn 512
 chroot /usr/local/haproxy
 user haproxy
 group haproxy
 daemon

defaults
  log global 
  option dontlognull
  retries 3 
  option redispatch
  maxconn 512
  timeout connect 5000
  timeout client 50000
  timeout server 50000

listen admin_status 
  bind *:1234
  stats uri /admin
  stats auth admin:admin
  mode http
  option httplog

listen all_mycat 
  bind *:8888
  mode tcp
  option tcplog
  balance roundrobin
    server mycat_101 192.168.100.101:8066 check port 8066 inter 5s rise 2 fall 3 
    server mycat_102 192.168.100.102:8066 check port 8066 inter 5s rise 2 fall 3 
  timeout server 20000

listen all_mycat_admin 
  bind *:9999
  mode tcp
  option tcplog
  balance roundrobin
    server mycat_101 192.168.100.101:9066 check port 9066 inter 5s rise 2 fall 3
    server mycat_102 192.168.100.102:9066 check port 9066 inter 5s rise 2 fall 3 
  timeout server 20000

[root@h101 haproxy]# 
~~~

Haproxy的配置有三个来源：

* 运行时通过命令行指定
* 配置文件的 **global** 区域
* 代理区域，包含 **defaults、listen、frontend、backend**

时间单位：

Units | Comment
-------- | ---
us|microseconds 1/1000000 second
ms|milliseconds 1/1000 second ，这是默认配置，不指定单位时用这个单位
s |seconds.  1s = 1000ms
m | minutes. 1m = 60s = 60000ms
h | hours.   1h = 60m = 3600s = 3600000ms
d | days.    1d = 24h = 1440m = 86400s = 86400000ms



CONF| Comment
-------- | ---
log 127.0.0.1 local0 | 记录到本机的 **local0** 类别中(之前rsyslog中自定义的类别)，详细可参考 **[log][c_log]**
maxconn 512 | 限制 **最大并行连接数** 为512，详细可参考 **[maxconn ][c_maxconn]**
chroot /usr/local/haproxy| 限定进程的 **视域** ，详细可参考 **[chroot][c_chroot]** ，官方文档说 `It is important to ensure that dir is both empty and unwritable to anyone` ，然而实际并无此限制，强调这个只是为了安全考虑 
user haproxy|以haproxy的身份运行 详细可参考 **[user][c_user]**
group haproxy|以haproxy的组运行 详细可参考 **[group][c_group]**
daemon |后台服务的方式运行， 详细可参考 **[daemon][c_daemon]**
log global| 日志配置沿袭global的设定, 详细可参考 **[log global][c_logglobal]**
option dontlognull|不记录无数据的操作，比如监控的侦测包 ,详细可参考 **[dontlognull][c_dontlognull]**
retries 3 | 失败后最多重试3次,  详细可参考 **[retries][c_retries]**
option redispatch|跳转的设定，-1代表最后一次失败尝试就直接跳转，1代表每一次失败尝试都是跳转，0代表失败后不进行跳转,  详细可参考 **[redispatch][c_redispatch]**
timeout connect 5000|一般性的超时时间为5s , 详细可参考 **[timeout connect][c_connect]**
timeout client 50000|与客户端之间的超时时间为50s ,详细可参考 **[timeout client][c_client]**
timeout server 50000|与后端服务器之间的超时时间为50s ,详细可参考 **[timeout server][c_server]**
listen admin_status| 定义一个包含前后端的完整监听
bind \*:1234| 监听1234端口, \* 代表任意IP, 详细可参考 **[bind][c_bind]**
stats uri /admin|定义/admin为统计uri， 详细可参考 **[stats uri][c_uri]**
stats auth admin:admin| 定义认证用户名和密码为 admin:admin  ， 详细可参考 **[stats auth][c_auth]**
mode http|设定代理模式为http，  详细可参考 **[mode][c_mode]**
option httplog|指定日志模式为http模式 ， 详细可参考 **[option httplog][c_httplog]**
balance roundrobin| 以轮询的方式进行负载均衡， 详细可参考 **[balance][c_balance]**
server mycat_101 192.168.100.101:8066 check port 8066 inter 5s rise 2 fall 3 | 定义一个后端服务器为mycat_101，主机IP为192.168.100.101 , 端口为 8066，检查8066是否开放，检查周期为5s，连续两次检查成功算正常，连续三次检查失败算异常




> **Note:**   **Mycat** 官方文档中的配置是使用的 **contimeout、clitimeout、srvtimeout** ，这种写法已经不被支持，如果在配置中这样指定会有如下报错

~~~
[root@h101 haproxy]# /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
[WARNING] 063/215627 (16321) : parsing [/usr/local/haproxy/haproxy.cfg:16] : the 'contimeout' directive is now deprecated in favor of 'timeout connect', and will not be supported in future versions.
[WARNING] 063/215627 (16321) : parsing [/usr/local/haproxy/haproxy.cfg:18] : the 'clitimeout' directive is now deprecated in favor of 'timeout client', and will not be supported in future versions.
[WARNING] 063/215627 (16321) : parsing [/usr/local/haproxy/haproxy.cfg:20] : the 'srvtimeout' directive is now deprecated in favor of 'timeout server', and will not be supported in future versions.
[root@h101 haproxy]# 
~~~

正确的写法是 

~~~
#contimeout 5000
  timeout connect 5000
#clitimeout 50000
  timeout client 50000
#srvtimeout 50000
  timeout server 50000
~~~

> **Note:** mycat官方文档中是使用 **listen all_mycat  192.168.100.101:8888** 的方式对ip进行绑定，但这种方式已经不被支持，如果使用这种方式，会有如下报错

~~~
[root@h101 haproxy]# /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
[ALERT] 063/223814 (22295) : parsing [/usr/local/haproxy/haproxy.cfg:29] : 'listen' cannot handle unexpected argument '192.168.100.101:8888'.
[ALERT] 063/223814 (22295) : parsing [/usr/local/haproxy/haproxy.cfg:29] : please use the 'bind' keyword for listening addresses.
[WARNING] 063/223814 (22295) : parsing [/usr/local/haproxy/haproxy.cfg:36] : overwriting 'timeout server' which was already specified
[ALERT] 063/223814 (22295) : Error(s) found in configuration file : /usr/local/haproxy/haproxy.cfg
[WARNING] 063/223814 (22295) : config : proxy 'all_mycat' has no 'bind' directive. Please declare it as a backend if this was intended.
[WARNING] 063/223814 (22295) : config : missing timeouts for proxy 'all_mycat'.
   | While not properly invalid, you will certainly encounter various problems
   | with such a configuration. To fix this, please ensure that all following
   | timeouts are set to a non-zero value: 'client', 'connect', 'server'.
[WARNING] 063/223814 (22295) : config : log format ignored for proxy 'all_mycat' since it has no log address.
[ALERT] 063/223814 (22295) : Fatal errors found in configuration.
[root@h101 haproxy]# 
~~~

正确的写法是使用 **bind**

~~~
listen all_mycat
  bind *:8888
  mode tcp
  option tcplog
  balance roundrobin
    server mycat_101 192.168.100.101:8066 check port 8066 inter 5s rise 2 fall 3
    server mycat_102 192.168.100.102:8066 check port 8066 inter 5s rise 2 fall 3
  timeout server 20000
~~~


详细内容可以参考 **[Haproxy 配置][haproxy_conf]**

---

## 启动haproxy

~~~
[root@h101 haproxy]# /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
[root@h101 haproxy]# ps faux | grep -v grep | grep haproxy
haproxy  23083  0.0  0.0  14260   936 ?        Ss   22:43   0:00 /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
[root@h101 haproxy]# netstat -ant |  grep -E "(8066|9066|8888|9999|1234)"
tcp        0      0 0.0.0.0:1234                0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:8888                0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:9999                0.0.0.0:*                   LISTEN        
tcp        0      0 :::8066                     :::*                        LISTEN      
tcp        0      0 :::9066                     :::*                        LISTEN      
[root@h101 haproxy]# 
~~~

查看 **/var/log/haproxy.log** 日志，会多出如下记录

~~~
Mar  4 22:43:40 localhost haproxy[23081]: Proxy admin_status started.
Mar  4 22:43:40 localhost haproxy[23081]: Proxy all_mycat started.
Mar  4 22:43:40 localhost haproxy[23081]: Proxy all_mycat_admin started.
~~~


> **Tip:**  这里只演示了其中一台的配置方法，以上这些在另一台服务器上也要作同样的设置


---



## 监控haproxy

浏览器中输入 **http://192.168.100.101:1234/admin** ，进行查看


![mycat_haproxy.png](/images/mycat_ha/mycat_haproxy.png)


如果检测到其中一个 mycat异常，会是如下界面

~~~
[root@h101 ~]# ps faux | grep mycat
root     33274  0.0  0.0 103256   828 pts/2    S+   23:55   0:00  |       \_ grep mycat
root      3980  0.1  0.0  19124   784 ?        Sl   13:17   0:50 /usr/local/src/mycat/mycat/bin/./wrapper-linux-x86-64 /usr/local/src/mycat/mycat/conf/wrapper.conf wrapper.syslog.ident=mycat wrapper.pidfile=/usr/local/src/mycat/mycat/logs/mycat.pid wrapper.daemonize=TRUE wrapper.lockfile=/var/lock/subsys/mycat
[root@h101 ~]# kill 3980
[root@h101 ~]# ps faux | grep mycat
root     33292  0.0  0.0 103256   828 pts/2    S+   23:55   0:00  |       \_ grep mycat
[root@h101 ~]# 
~~~

![mycat_haproxy2.png](/images/mycat_ha/mycat_haproxy2.png)

![mycat_haproxy3.png](/images/mycat_ha/mycat_haproxy3.png)


> **Tip:** 要确保在本地防火墙上开放了 **1234** 端口



---

## 配置keepalived


###  简单的haproxy检查脚本

keepalived要对本机运行的haproxy健康状态进行检查，当发现haproxy不能正常工作的情况下，将IP交由另一台服务器进行管理

下面是一个最简单的 haproxy 健康检查脚本，能实现对haproxy运行状态的粗略判断(当然这个脚本有很大的精进打磨空间)

~~~
[root@h101 script]# cat /usr/local/keepalived/script/chk_haproxy.bash 
#!/bin/bash

count=`ps aux | grep -v grep | grep haproxy.cfg | wc -l`

if [ $count -gt 0 ]; then
    exit 0
else
    exit 1
fi
[root@h101 script]# chmod +x /usr/local/keepalived/script/chk_haproxy.bash 
[root@h101 script]# 
~~~

它进行的判断就是，如果系统中有命令包含 **haproxy.cfg** 的进程(假定这种情况就代表haproxy正在运行)，就反馈 **0** ， 否则反馈 **1**

~~~
[root@h101 script]# ps faux | grep -v grep | grep haproxy 
haproxy  23083  0.0  0.0  14260  1408 ?        Ss   22:43   0:00 /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
[root@h101 script]# /usr/local/keepalived/script/chk_haproxy.bash 
[root@h101 script]# echo $?
0
[root@h101 script]# kill 23083
[root@h101 script]# ps faux | grep -v grep | grep haproxy 
[root@h101 script]# /usr/local/keepalived/script/chk_haproxy.bash 
[root@h101 script]# echo $?
1
[root@h101 script]# 
~~~

---

###  配置keepalived


~~~
[root@h101 script]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id LVS_101
}

vrrp_script checkhaproxy {
        script "/usr/local/keepalived/script/chk_haproxy.bash"
        weight -20
        interval 3
}

vrrp_instance VI_222 {
    state BACKUP 
    interface eth2
    virtual_router_id 222
    priority 108
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
            checkhaproxy
    }
    virtual_ipaddress {
        192.168.100.222/24
    }
}
[root@h101 script]# 
----------
[root@h102 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id LVS_102
}

vrrp_script checkhaproxy {
	script "/usr/local/keepalived/script/chk_haproxy.bash"
	weight -20 
	interval 3
}


vrrp_instance VI_222 {
    state BACKUP 
    interface eth2
    virtual_router_id 222
    priority 115
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
            checkhaproxy
    }

    virtual_ipaddress {
        192.168.100.222/24
    }
}
[root@h102 ~]# 
~~~

其中的核心部分在这里


~~~
vrrp_script checkhaproxy {
	script "/usr/local/keepalived/script/chk_haproxy.bash"
	weight -20 
	interval 3
}
...
...
    track_script {
            checkhaproxy
    }
~~~

**track_script**  中调用 **checkhaproxy**

**checkhaproxy** 的意思是：

每3秒种执行一次 **/usr/local/keepalived/script/chk_haproxy.bash** 脚本
如果反馈结果是 **0**，就保持原优先级 **priority** ，如果是 **1** ，就将优先级降低 **20**，也就是检查到 haproxy 状态异常后，就降级，以便让另一台服务器的keepalived进程可以抢到IP

为了避免网络的不稳定还可以加入 **fall N** (代表连续N次检查失败才算异常) 和 **rise N** (代表连续N次检查成功就算正常)

优先级改变的算法是这样的：

* 如果脚本执行结果为0，并且weight配置的值大于0，则优先级相应的增加
* 如果脚本执行结果非0，并且weight配置的值小于0，则优先级相应的减少
* 其他情况，维持原本配置的优先级，即配置文件中priority对应的值


> **Note:** keepalived 相互之间的通讯要使用到组播，如果没打开，会出现几个实例同时抢占着IP的情况，打开方式是在iptables中加入  **`-A INPUT -d 224.0.0.18 -j ACCEPT`**，然后重载iptables配置

---

## 启动keepalived

先确保两边的haproxy都是正常运行的

~~~
[root@h101 script]# /usr/local/keepalived/sbin/keepalived -f /etc/keepalived/keepalived.conf
[root@h101 script]#
~~~

两边的keepalived启动后，以初始设定优先级高的keepalived为Master

当优先级高的keeaplived检测到haproxy异常后，会自动降级20，然后重新选举Master，这时另一台服务器的优先级就相对较高，更有优势，于是抢到IP，成为新的Master


在原master上执行以下命令，可以看到IP的漂移过程 

~~~
[root@h102 mycat-web]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b6:a8:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.143/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:feb6:a8f8/64 scope link 
       valid_lft forever preferred_lft forever
3: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b6:a8:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.102/24 brd 192.168.100.255 scope global eth2
    inet 192.168.100.222/24 scope global secondary eth2
    inet6 fe80::20c:29ff:feb6:a802/64 scope link 
       valid_lft forever preferred_lft forever
[root@h102 mycat-web]# ps faux | grep haproxy 
root     12707  0.0  0.0 103256   828 pts/1    S+   23:48   0:00  |       \_ grep haproxy
haproxy  12118  0.0  0.0  14260   936 ?        Ss   23:42   0:00 /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
[root@h102 mycat-web]# kill 12118; watch -n .5 ip a 
[root@h102 mycat-web]# 
~~~

这个过程中 **192.168.100.222** 会消失，在另一台服务器上，就能看到这个IP被挂载了


反过来也一样，会看到IP被挂载的过程

~~~
[root@h102 mycat-web]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b6:a8:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.143/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:feb6:a8f8/64 scope link 
       valid_lft forever preferred_lft forever
3: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b6:a8:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.102/24 brd 192.168.100.255 scope global eth2
    inet6 fe80::20c:29ff:feb6:a802/64 scope link 
       valid_lft forever preferred_lft forever
[root@h102 mycat-web]# /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg; watch -n .5 ip a 
[root@h102 mycat-web]# 
~~~



---

## 访问测试 

~~~
[root@h101 ~]# mysql -u cc -p -P 8888 -h 192.168.100.222 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 174
Server version: 5.5.8-mycat-1.5-GA-20160217103036 MyCat Server (OpenCloundDB)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+----------+
| DATABASE |
+----------+
| cctest   |
+----------+
1 row in set (0.03 sec)

mysql> use cctest;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables in cctest |
+------------------+
| abc              |
| catworld         |
+------------------+
2 rows in set (0.00 sec)

mysql> 
~~~


## 切换过程中的影响

切换过程并非完全没有任何影响，一般会产生一次中断，但当再次发起请求时(重试时)就能恢复正常

下面的过程就是在切换中进行操作的

~~~
mysql> show tables;
ERROR 2013 (HY000): Lost connection to MySQL server during query
mysql> show tables;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    175
Current database: cctest

+------------------+
| Tables in cctest |
+------------------+
| abc              |
| catworld         |
+------------------+
2 rows in set (0.07 sec)

mysql> show databases;
+----------+
| DATABASE |
+----------+
| cctest   |
+----------+
1 row in set (0.00 sec)

mysql> 
~~~


---


# 命令汇总

* **`wget  http://www.keepalived.org/software/keepalived-1.2.19.tar.gz`**
* **`tar -zxvf keepalived-1.2.19.tar.gz`**
* **`cd keepalived-1.2.19`**
* **`./configure --prefix=/usr/local/keepalived`**
* **`make`**
* **`make install`**
* **`tree /usr/local/keepalived/`**
* **`/usr/local/keepalived/sbin/keepalived  -v`**
* **`md5sum haproxy-1.6.3.tar.gz`**
* **`tar -zxvf haproxy-1.6.3.tar.gz`**
* **`cd haproxy-1.6.3`**
* **`make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy`**
* **`make install PREFIX=/usr/local/haproxy`**
* **`tree /usr/local/haproxy/`**
* **`/usr/local/haproxy/sbin/haproxy -v`**
* **`rpm -qa | grep rsyslog`**
* **`grep -v "^#" /etc/rsyslog.conf  | grep -v "^$"`**
* **`vim /etc/rsyslog.d/haproxy.conf`**
* **`cat /etc/rsyslog.d/haproxy.conf`**
* **`/etc/init.d/rsyslog restart`**
* **`ll /var/log/ha*`**
* **`netstat  -antulp | grep 514`**
* **`logger -it test -p local0.info "test"`**
* **`tail -f /var/log/haproxy.log`**
* **`rsync  -av root@192.168.100.102:/usr/local/src/mycat/Mycat-server-1.5-GA-20160217103036-linux.tar.gz  .`**
* **`tar -zxvf Mycat-server-1.5-GA-20160217103036-linux.tar.gz`**
* **`java -version`**
* **`cat wrapper.conf | egrep "(Xm|MaxDirectMemorySize)"`**
* **`vim /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv | grep -E "(8066|9066|8888|9999)"`**
* **`./mycat  start`**
* **`mysql -u cc -p -P 8066 -p -h 192.168.100.101`**
* **`useradd haproxy`**
* **`grep proxy /etc/passwd`**
* **`chown -R haproxy.haproxy /usr/local/haproxy/`**
* **`vim haproxy.cfg`**
* **`grep -v "^#" haproxy.cfg`**
* **`/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg`**
* **`ps faux | grep -v grep | grep haproxy`**
* **`netstat -ant |  grep -E "(8066|9066|8888|9999|1234)"`**
* **`cat /usr/local/keepalived/script/chk_haproxy.bash`**
* **`chmod +x /usr/local/keepalived/script/chk_haproxy.bash`**
* **`ps faux | grep -v grep | grep haproxy`**
* **`/usr/local/keepalived/script/chk_haproxy.bash`**
* **`cat /etc/keepalived/keepalived.conf`**
* **`/usr/local/keepalived/sbin/keepalived -f /etc/keepalived/keepalived.conf`**
* **`kill 12118; watch -n .5 ip a`**
* **`/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg; watch -n .5 ip a`**
* **`mysql -u cc -p -P 8888 -h 192.168.100.222`**


---





[mycat]:http://www.mycat.org.cn/
[mycat_doc]:http://www.mycat.org.cn/document/mycat1.5.2.pdf
[keepalived]:http://www.keepalived.org/index.html
[keepalived_dl]:http://www.keepalived.org/download.html
[haproxy]:http://www.haproxy.org/
[haproxy_dl]:http://www.haproxy.org/#down
[haproxy_doc]:http://cbonte.github.io/haproxy-dconv/index.html
[haproxy_conf]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html
[c_log]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#log
[c_maxconn]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#maxconn
[c_chroot]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#chroot
[c_user]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#user
[c_group]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#group
[c_daemon]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#daemon
[c_logglobal]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#log%20global
[c_dontlognull]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-option%20dontlognull
[c_retries]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-retries
[c_redispatch]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#option%20redispatch
[c_connect]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#timeout%20connect
[c_client]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#timeout%20client
[c_server]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#timeout%20server
[c_bind]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-bind
[c_uri]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-stats%20uri
[c_auth]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-stats%20auth
[c_mode]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4.2-mode
[c_httplog]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4.2-option%20httplog
[c_balance]:http://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-balance

