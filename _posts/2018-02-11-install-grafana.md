---
layout: post
title: "Install Grafana"
author:  wilmosfang
date: 2018-02-11 12:16:29
image: '/assets/img/'
excerpt: '安装 Grafana'
main-class: grafana
color: '#f07a2d'
tags:
 - grafana
categories:
 - grafana
twitter_text: 'simple process of Grafana installation'
introduction: 'installation of Grafana'
---



## 前言

**[Grafana][grafana]** 是一款颜值极高的数据展示软件，可以对接多种数据源，对数据进行可视化分析与展示

>The open platform for beautiful
analytics and monitoring, No matter where your data is, or what kind of database it lives in, you can bring it together with Grafana. Beautifully.

在这个大数据时代，对数据进行可视化分析的确是一种减轻信息负载，提高数据理解，提炼高价值知识的的有效手段

这里分享一下 **[Grafana][grafana]** 的安装方法

参考 **[Installing on RPM-based Linux (CentOS, Fedora, OpenSuse, RedHat)][grafana_install]**

> **Tip:** 当前的稳定版为 **grafana-4.6.3-1** 最新版本为 **grafana-5.0.0-beta1**

---

# 操作


## 环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 15bc89d01d6c4e6284a91ca246a75d4d
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
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 54859sec preferred_lft 54859sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
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


## 安装 grafana

~~~
[root@much ~]# sudo yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.6.3-1.x86_64.rpm
Loaded plugins: fastestmirror, langpacks
grafana-4.6.3-1.x86_64.rpm                               |  45 MB     02:40     
Examining /var/tmp/yum-root-iJmrTZ/grafana-4.6.3-1.x86_64.rpm: grafana-4.6.3-1.x86_64
Marking /var/tmp/yum-root-iJmrTZ/grafana-4.6.3-1.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package grafana.x86_64 0:4.6.3-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package       Arch         Version         Repository                     Size
================================================================================
Installing:
 grafana       x86_64       4.6.3-1         /grafana-4.6.3-1.x86_64       133 M

Transaction Summary
================================================================================
Install  1 Package

Total size: 133 M
Installed size: 133 M
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : grafana-4.6.3-1.x86_64                                       1/1
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server.service
### You can start grafana-server by executing
 sudo /bin/systemctl start grafana-server.service
POSTTRANS: Running script
  Verifying  : grafana-4.6.3-1.x86_64                                       1/1

Installed:
  grafana.x86_64 0:4.6.3-1                                                      

Complete!
[root@much ~]#
[root@much ~]# rpm -qa | grep grafa
grafana-4.6.3-1.x86_64
[root@much ~]#
~~~


## 启动 grafana 服务

~~~
[root@much ~]# service grafana-server start
Starting grafana-server (via systemctl):                   [  OK  ]
[root@much ~]# systemctl daemon-reload
[root@much ~]# systemctl start grafana-server
[root@much ~]# systemctl status grafana-server
● grafana-server.service - Grafana instance
   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: disabled)
   Active: active (running) since 日 2018-02-11 19:01:00 CST; 21s ago
     Docs: http://docs.grafana.org
 Main PID: 6897 (grafana-server)
   CGroup: /system.slice/grafana-server.service
           └─6897 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini ...

2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i..."
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i..."
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i..."
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i...s
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=w...s
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i...s
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i...e
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i...p
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i..."
2月 11 19:01:00 much grafana-server[6897]: t=2018-02-11T19:01:00+0800 lvl=i...=
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]#
~~~

服务默认监听在 **3000** 端口

~~~
[root@much ~]# netstat  -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN      1314/java           
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      1314/java           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1523/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1320/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1318/cupsd          
tcp        0      0 0.0.0.0:5601            0.0.0.0:*               LISTEN      1666/node           
tcp        0      0 127.0.0.1:35654         127.0.0.1:9200          ESTABLISHED 4290/python2        
tcp        0      0 127.0.0.1:9200          127.0.0.1:35588         ESTABLISHED 1314/java           
tcp        0      0 127.0.0.1:35588         127.0.0.1:9200          ESTABLISHED 1666/node           
tcp        0      0 192.168.56.208:22       192.168.56.1:50386      ESTABLISHED 5072/sshd: root@pts
tcp        0      0 192.168.56.208:22       192.168.56.1:49312      ESTABLISHED 3874/sshd: root@pts
tcp        0      0 192.168.56.208:22       192.168.56.1:53160      ESTABLISHED 6613/sshd: root@pts
tcp        0      0 192.168.56.208:22       192.168.56.1:48986      ESTABLISHED 1576/sshd: root@pts
tcp        0      0 127.0.0.1:9200          127.0.0.1:35654         ESTABLISHED 1314/java           
tcp6       0      0 :::22                   :::*                    LISTEN      1320/sshd           
tcp6       0      0 :::3000                 :::*                    LISTEN      6897/grafana-server
[root@much ~]#
~~~


## 配置开机启动

~~~
[root@much ~]# systemctl enable grafana-server.service
Created symlink from /etc/systemd/system/multi-user.target.wants/grafana-server.service to /usr/lib/systemd/system/grafana-server.service.
[root@much ~]#
~~~



## 打开放火墙

~~~
[root@much ~]# firewall-cmd  --list-all
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

[root@much ~]#
[root@much ~]# firewall-cmd --add-port 3000/tcp --permanent  
success
[root@much ~]# firewall-cmd  --reload
success
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 3000/tcp 8080/tcp 5601/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]#
~~~



## 访问 grafana


![grafana](/assets/img/grafana/grafana01.png)

默认账号密码为 **admin/admin**

![grafana](/assets/img/grafana/grafana02.png)


此窗口一出，已经可以隐约感受到不凡的颜值，随便点击进入，界面和交互都很赏心悦目


---

# 总结

在颜值即正义的年代，好看也是一个让人无法拒绝的理由

随着数据可视化需求的增长，像 **grafana** 还有 **kibana** 一样的数据展示系统一定会越来越有市场

* TOC
{:toc}


---


[grafana]:https://grafana.com/
[grafana_install]:http://docs.grafana.org/installation/rpm/
