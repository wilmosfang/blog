---
layout: post
title: "Install Kibana"
author:  wilmosfang
date: 2018-02-01 12:33:19
image: '/assets/img/'
excerpt: 'Kibana 的安装方法'
main-class: es
color: '#51bcb2'
tags:
 - kibana
 - es
categories:
 - es
twitter_text: 'simple process of Kibana installation'
introduction: 'installation method of Kibana'
---


# 前言

**[Kibana][kibana]** 是 Elasticsearch 的前端展示组件

>Kibana gives shape to your data and is the extensible user interface for configuring and managing all aspects of the Elastic Stack.

具有强大的数据展示能力，可以形成漂亮直观的表格与图形，操作简单，给数据分析与报告带来很大便利

这里分享一下 **[Kibana][kibana]** 的安装方法

参考 **[Install Kibana with RPM][kibana_installation]**

> **Tip:** 当前版本 **Version:6.1.3**



---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: f49ec5c0cb7940328cbc5b9d6ca0a526
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d1:5d:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 80858sec preferred_lft 80858sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:47:20:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 导入 GPG key


~~~
[root@much yum.repos.d]# rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
[root@much yum.repos.d]#
~~~


## 配置 repo

配置 kibana 仓库

~~~
[root@much yum.repos.d]# vim kibana.repo
[root@much yum.repos.d]# cat kibana.repo
[kibana-6.x]
name=Kibana repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
[root@much yum.repos.d]#
~~~

>**Tips:** 也可以在这里直接下载 **[Download Kibana][kibana_dl]**


## 安装软件

~~~
[root@much yum.repos.d]# yum install kibana
Loaded plugins: fastestmirror, langpacks
kibana-6.x                                               | 1.3 kB     00:00     
kibana-6.x/primary                                         |  31 kB   00:04     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
kibana-6.x                                                                90/90
Resolving Dependencies
--> Running transaction check
---> Package kibana.x86_64 0:6.1.3-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch             Version           Repository            Size
================================================================================
Installing:
 kibana           x86_64           6.1.3-1           kibana-6.x            63 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 63 M
Installed size: 232 M
Is this ok [y/d/N]: y
Downloading packages:
kibana-6.1.3-x86_64.rpm                                    |  63 MB   36:10     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : kibana-6.1.3-1.x86_64                                        1/1
  Verifying  : kibana-6.1.3-1.x86_64                                        1/1

Installed:
  kibana.x86_64 0:6.1.3-1                                                       

Complete!
[root@much yum.repos.d]#
~~~



## 启动实例


同时配置开机可以自启动

~~~
[root@much yum.repos.d]# systemctl daemon-reload
[root@much yum.repos.d]# systemctl enable kibana.service
Created symlink from /etc/systemd/system/multi-user.target.wants/kibana.service to /etc/systemd/system/kibana.service.
[root@much yum.repos.d]# ps faux | grep -i  kibana
root      6918  0.0  0.0 112648  1028 pts/2    S+   22:14   0:00          \_ grep --color=auto -i kibana
[root@much yum.repos.d]# netstat  -ant | grep 5601
[root@much yum.repos.d]# systemctl start kibana.service
[root@much yum.repos.d]# netstat  -ant | grep 5601
tcp        0      0 127.0.0.1:5601          0.0.0.0:*               LISTEN     
[root@much yum.repos.d]# ps faux | grep -i  kibana
root      6951  0.0  0.0 112648  1028 pts/2    S+   22:15   0:00          \_ grep --color=auto -i kibana
kibana    6927 20.5  3.1 1197004 129328 ?      Ssl  22:14   0:02 /usr/share/kibana/bin/../node/bin/node --no-warnings /usr/share/kibana/bin/../src/cli -c /etc/kibana/kibana.yml
[root@much yum.repos.d]#
~~~


## 配置 kibana

### 修改监听

默认情况下 kibana 监听在 **localhost:5601**

只能本地访问，但是如果想提供给其它客户端访问，可以修改默认配置

~~~
[root@much ~]# vim /etc/kibana/kibana.yml
[root@much ~]# grep -v "#" /etc/kibana/kibana.yml  | cat -s

server.host: "0.0.0.0"

[root@much ~]# netstat -ant | grep 5601
tcp        0      0 127.0.0.1:5601          0.0.0.0:*               LISTEN     
[root@much ~]# systemctl restart kibana
[root@much ~]# netstat -ant | grep 5601
tcp        0      0 0.0.0.0:5601            0.0.0.0:*               LISTEN     
[root@much ~]#
~~~

重启服务后，已经监听在了 **0.0.0.0**


### 打开放火墙

~~~
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 8080/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]# firewall-cmd --add-port 5601/tcp --permanent  
success
[root@much ~]# firewall-cmd --reload
success
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 8080/tcp 5601/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]# iptables -L -nv
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:53
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:67
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:67
   76  8521 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    0     0 INPUT_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 INPUT_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 INPUT_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  *      virbr0  0.0.0.0/0            192.168.122.0/24     ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  virbr0 *       192.168.122.0/24     0.0.0.0/0           
    0     0 ACCEPT     all  --  virbr0 virbr0  0.0.0.0/0            0.0.0.0/0           
    0     0 REJECT     all  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 REJECT     all  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    0     0 FORWARD_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FORWARD_IN_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FORWARD_IN_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FORWARD_OUT_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FORWARD_OUT_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT 56 packets, 7945 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     udp  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            udp dpt:68
   58  8161 OUTPUT_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain FORWARD_IN_ZONES (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 FWDI_public  all  --  enp0s8 *       0.0.0.0/0            0.0.0.0/0           [goto]
    0     0 FWDI_public  all  --  enp0s3 *       0.0.0.0/0            0.0.0.0/0           [goto]
    0     0 FWDI_public  all  --  +      *       0.0.0.0/0            0.0.0.0/0           [goto]

Chain FORWARD_IN_ZONES_SOURCE (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD_OUT_ZONES (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 FWDO_public  all  --  *      enp0s8  0.0.0.0/0            0.0.0.0/0           [goto]
    0     0 FWDO_public  all  --  *      enp0s3  0.0.0.0/0            0.0.0.0/0           [goto]
    0     0 FWDO_public  all  --  *      +       0.0.0.0/0            0.0.0.0/0           [goto]

Chain FORWARD_OUT_ZONES_SOURCE (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD_direct (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FWDI_public (3 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 FWDI_public_log  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FWDI_public_deny  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FWDI_public_allow  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain FWDI_public_allow (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FWDI_public_deny (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FWDI_public_log (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FWDO_public (3 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 FWDO_public_log  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FWDO_public_deny  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 FWDO_public_allow  all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain FWDO_public_allow (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FWDO_public_deny (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain FWDO_public_log (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT_ZONES (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 IN_public  all  --  enp0s8 *       0.0.0.0/0            0.0.0.0/0           [goto]
    0     0 IN_public  all  --  enp0s3 *       0.0.0.0/0            0.0.0.0/0           [goto]
    0     0 IN_public  all  --  +      *       0.0.0.0/0            0.0.0.0/0           [goto]

Chain INPUT_ZONES_SOURCE (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT_direct (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain IN_public (3 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 IN_public_log  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 IN_public_deny  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 IN_public_allow  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain IN_public_allow (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 ctstate NEW
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:5601 ctstate NEW

Chain IN_public_deny (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain IN_public_log (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT_direct (1 references)
 pkts bytes target     prot opt in     out     source               destination         
[root@much ~]#
~~~


## 访问

访问 **http://serverip:5601**

这里我访问 **http://192.168.56.208:5601**

![kibana](/assets/img/kibana/kibana01.png)

在 **Management** 中配置好 **Index Patterns** , 就可以开始漫游 Elasticsearch 中的数据了

![kibana](/assets/img/kibana/kibana02.png)

不得不说 ELK 技术栈，让日志分析与数据可视变得简单而高效了

---

# 总结

使用 rpm 包安装的过程非常简单

* TOC
{:toc}


---

[kibana]:https://www.elastic.co/products/kibana
[kibana_dl]:https://www.elastic.co/downloads/kibana
[kibana_installation]:https://www.elastic.co/guide/en/kibana/current/rpm.html
