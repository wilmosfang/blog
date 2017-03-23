---
layout: post
title: Linux 网关 
author: wilmosfang
tags:  network
categories:  network
wc: 192 507 4772
excerpt: 使用Linux搭建网关服务器
comments: true
---

前言
=

生产环境中，Public IP 经常比较有限，Linux GateWay可以充分利用有限IP为更多的机器提供网络服务，也可以有意识地将某些服务器隐藏在后面，即可以主动获取网络资源，又避免被动访问，更加安全

---

# 概要

* TOC
{:toc}

---


开启内核转发
-

调整内核参数 **net.ipv4.ip_forward** 开启转发

~~~
[root@linux-gateway ~]# grep  forward  /etc/sysctl.conf 
# Controls IP packet forwarding
net.ipv4.ip_forward = 1
[root@linux-gateway ~]# 
~~~

**sysctl -p** 使其生效，然后使用 **sysctl -a** 来进行确认

~~~
[root@linux-gateway ~]# sysctl  -a | grep  forwarding
net.ipv4.conf.all.forwarding = 1
net.ipv4.conf.all.mc_forwarding = 0
net.ipv4.conf.default.forwarding = 1
net.ipv4.conf.default.mc_forwarding = 0
net.ipv4.conf.lo.forwarding = 1
net.ipv4.conf.lo.mc_forwarding = 0
net.ipv4.conf.em4.forwarding = 1
net.ipv4.conf.em4.mc_forwarding = 0
net.ipv4.conf.em2.forwarding = 1
net.ipv4.conf.em2.mc_forwarding = 0
net.ipv4.conf.em3.forwarding = 1
net.ipv4.conf.em3.mc_forwarding = 0
net.ipv4.conf.em1.forwarding = 1
net.ipv4.conf.em1.mc_forwarding = 0
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.all.mc_forwarding = 0
net.ipv6.conf.default.forwarding = 0
net.ipv6.conf.default.mc_forwarding = 0
net.ipv6.conf.lo.forwarding = 0
net.ipv6.conf.lo.mc_forwarding = 0
net.ipv6.conf.em1.forwarding = 0
net.ipv6.conf.em1.mc_forwarding = 0
net.ipv6.conf.em2.forwarding = 0
net.ipv6.conf.em2.mc_forwarding = 0
net.ipv6.conf.em3.forwarding = 0
net.ipv6.conf.em3.mc_forwarding = 0
net.ipv6.conf.em4.forwarding = 0
net.ipv6.conf.em4.mc_forwarding = 0
[root@linux-gateway ~]# 
~~~


---

开启iptables转发
-

查看内网网卡

~~~
[root@linux-gateway ~]# ip a | grep 168
    inet 192.168.1.254/24 brd 192.168.1.255 scope global em1
[root@linux-gateway ~]# 
~~~

内网网卡为 **em1**

查看默认路由，与出口网卡

~~~
[root@linux-gateway ~]# ip route | grep default 
default via 180.140.110.123 dev em2 
[root@linux-gateway ~]# 
~~~

出口网卡为 **em2**

方法一：直接在命令行配置 

* **filter** 表上接受来自 **em1** 的 **FORWARD** 请求
* **nat** 表的 **POSTROUTING** 链上打开来自内网出口为 **em2** 的地址伪装，即 **SNAT** 

~~~
[root@linux-gateway ~]# iptables -A FORWARD -i em1 -j ACCEPT
[root@linux-gateway ~]# iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o em2 -j MASQUERADE 
~~~

方法二：通过配置文件修改

可以在 **/etc/sysconfig/iptables** 中的 **filter** 和 **nat** 部分进行配置


~~~
*nat
:PREROUTING ACCEPT [3370:177472]
:POSTROUTING ACCEPT [32:1664]
:OUTPUT ACCEPT [42:2288]
...
...
-A POSTROUTING -s 192.168.1.0/24 -o em2 -j MASQUERADE 
COMMIT
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [65295:28838004]
...
...
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
-A FORWARD -i em1 -j ACCEPT 
COMMIT
~~~

使用 **/etc/init.d/iptables  reload** 重新加载 **iptables**

---

配置主机默认路由
-

在想要连接外网的服务器上删除原有路由，添加新路由

~~~
[root@db-server ~]# ip route | grep default
default via 192.168.1.1 dev em1 
[root@db-server ~]# ip route del default 
[root@db-server ~]# ip route add default via 192.168.1.254 dev em1
~~~

测试连接

~~~
[root@db-server ~]# ping www.baidu.com
PING www.a.shifen.com (58.217.200.13) 56(84) bytes of data.
64 bytes from 58.217.200.13: icmp_seq=1 ttl=51 time=7.59 ms
64 bytes from 58.217.200.13: icmp_seq=2 ttl=51 time=7.60 ms
64 bytes from 58.217.200.13: icmp_seq=3 ttl=51 time=7.65 ms
64 bytes from 58.217.200.13: icmp_seq=4 ttl=51 time=7.58 ms
64 bytes from 58.217.200.13: icmp_seq=5 ttl=51 time=7.64 ms
^C
--- www.a.shifen.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4621ms
rtt min/avg/max/mdev = 7.585/7.615/7.653/0.113 ms
[root@db-server ~]# 
~~~

---

总结
=

* **`net.ipv4.ip_forward = 1`**
* **`grep  forward  /etc/sysctl.conf`**
* **`sysctl  -a | grep  forwarding`**
* **`ip route | grep default`**
* **`iptables -A FORWARD -i em1 -j ACCEPT`**
* **`iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o em2 -j MASQUERADE`**
* **`-A POSTROUTING -s 192.168.1.0/24 -o em2 -j MASQUERADE`**
* **`-A FORWARD -i em1 -j ACCEPT`**
* **`/etc/init.d/iptables  reload`**
* **`ip route del default`**
* **`ip route add default via 192.168.1.254 dev em1`**

总体分三部
-

* 1.打开内核参数 **net.ipv4.ip_forward** 允许转发
* 2.打开 **filter** 表 **FORWARD** 链内网端口的转发
* 3.打开 **nat** 表 **POSTROUTING** 链的定向地址伪装

---
