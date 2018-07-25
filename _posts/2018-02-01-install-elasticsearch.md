---
layout: post
title: "Install Elasticsearch"
author:  wilmosfang
date: 2018-02-01 10:31:26
image: '/assets/img/'
excerpt: 'Elasticsearch 的安装方法'
main-class: es
color: '#51bcb2'
tags:
 - es
categories:
 - es
twitter_text: 'simple process of Elasticsearch installation'
introduction: 'installation method of Elasticsearch'
---


# 前言


**[Elasticsearch][elasticsearch]** 是一个分布式全文检索引擎

>Elasticsearch is a distributed, JSON-based search and analytics engine designed for horizontal scalability, maximum reliability, and easy management.

具有强大的文本检索与分析能力，非常受欢迎

这里分享一下 **[Elasticsearch][elasticsearch]** 的安装方法

参考 **[Installation][elasticsearch_installation]**

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

## 依赖


Elasticsearch 需要 Java８ 的支持

官方建议使用 Oracle JDK version 1.8.0_131

>Elasticsearch requires at least Java 8. Specifically as of this writing, it is recommended that you use the Oracle JDK version 1.8.0_131

~~~
[root@much ~]# java -version
openjdk version "1.8.0_151"
OpenJDK Runtime Environment (build 1.8.0_151-b12)
OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)
[root@much ~]#
~~~

我本地的是 **openjdk version "1.8.0_151"** ，这个是 OpenJDK，并非 Oracle 版的，不过也没啥大问题，比推荐的 **1.8.0_131** 还新


## 下载安装包

首先下载安装包

下载地址 **[Download Elasticsearch][elasticsearch_dl]**

~~~
[root@much es]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.3.rpm
--2018-02-01 20:22:51--  https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.3.rpm
Resolving artifacts.elastic.co (artifacts.elastic.co)... 23.21.118.61, 184.72.218.26, 184.72.242.47, ...
Connecting to artifacts.elastic.co (artifacts.elastic.co)|23.21.118.61|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 28411530 (27M) [application/octet-stream]
Saving to: ‘elasticsearch-6.1.3.rpm’

100%[======================================>] 28,411,530  59.5KB/s   in 28m 3s

2018-02-01 20:51:02 (16.5 KB/s) - ‘elasticsearch-6.1.3.rpm’ saved [28411530/28411530]

[root@much es]# ls
elasticsearch-6.1.3.rpm
[root@much es]#
~~~


## 安装

直接使用 rpm 进行安装

~~~
[root@much es]# rpm -ivh elasticsearch-6.1.3.rpm
Preparing...                          ################################# [100%]
Creating elasticsearch group... OK
Creating elasticsearch user... OK
Updating / installing...
   1:elasticsearch-0:6.1.3-1          ################################# [100%]
### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
 sudo systemctl daemon-reload
 sudo systemctl enable elasticsearch.service
### You can start elasticsearch service by executing
 sudo systemctl start elasticsearch.service
[root@much es]#
~~~

## 运行

根据提示加载服务，并且使它下次自动运行

~~~
[root@much es]# systemctl daemon-reload
[root@much es]# systemctl enable elasticsearch.service
Created symlink from /etc/systemd/system/multi-user.target.wants/elasticsearch.service to /usr/lib/systemd/system/elasticsearch.service.
[root@much es]# ps faux | grep -i elast
root      5402  0.0  0.0 112648  1028 pts/0    S+   20:59   0:00  |       \_ grep --color=auto -i elast
[root@much es]# netstat  -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.208:22       192.168.56.1:53858      ESTABLISHED
tcp        0      0 192.168.56.208:22       192.168.56.1:53368      ESTABLISHED
tcp        0      0 10.0.2.15:42342         54.225.188.6:443        ESTABLISHED
tcp       32      0 10.0.2.15:42566         54.235.82.130:443       CLOSE_WAIT
tcp        0      0 192.168.56.208:22       192.168.56.1:53128      ESTABLISHED
tcp        0      0 10.0.2.15:36934         184.72.242.47:443       ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN     
[root@much es]# systemctl start elasticsearch.service
[root@much es]# ps faux | grep -i elast
root      5480  0.0  0.0 112648  1028 pts/0    S+   21:00   0:00  |       \_ grep --color=auto -i elast
elastic+  5436  135 29.3 3614912 1187844 ?     Ssl  21:00   0:05 /bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/lib/elasticsearch -Des.path.home=/usr/share/elasticsearch -Des.path.conf=/etc/elasticsearch -cp /usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch -p /var/run/elasticsearch/elasticsearch.pid --quiet
[root@much es]# netstat  -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.208:22       192.168.56.1:53858      ESTABLISHED
tcp        0      0 192.168.56.208:22       192.168.56.1:53368      ESTABLISHED
tcp        0      0 10.0.2.15:42342         54.225.188.6:443        ESTABLISHED
tcp       32      0 10.0.2.15:42566         54.235.82.130:443       CLOSE_WAIT
tcp        0      0 192.168.56.208:22       192.168.56.1:53128      ESTABLISHED
tcp        0      0 10.0.2.15:36934         184.72.242.47:443       ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN     
[root@much es]#
~~~

可见实例已经运行起来了，并且监听在本地的 **9200 9300** 端口

## 查看集群状态

~~~
[root@much es]# curl 'localhost:9200/_cat/health?v'
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1517490762 21:12:42  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
[root@much es]#
~~~


>**Tips:** 可以使用 help 参数来查看 title 的解释

~~~
[root@much es]# curl 'localhost:9200/_cat/health?help'
epoch                 | t,time                                   | seconds since 1970-01-01 00:00:00  
timestamp             | ts,hms,hhmmss                            | time in HH:MM:SS                   
cluster               | cl                                       | cluster name                       
status                | st                                       | health status                      
node.total            | nt,nodeTotal                             | total number of nodes              
node.data             | nd,nodeData                              | number of nodes that can store data
shards                | t,sh,shards.total,shardsTotal            | total number of shards             
pri                   | p,shards.primary,shardsPrimary           | number of primary shards           
relo                  | r,shards.relocating,shardsRelocating     | number of relocating nodes         
init                  | i,shards.initializing,shardsInitializing | number of initializing nodes       
unassign              | u,shards.unassigned,shardsUnassigned     | number of unassigned shards        
pending_tasks         | pt,pendingTasks                          | number of pending tasks            
max_task_wait_time    | mtwt,maxTaskWaitTime                     | wait time of longest task pending  
active_shards_percent | asp,activeShardsPercent                  | active number of shards in percent
[root@much es]#
~~~


## 查看节点

~~~
[root@much es]# curl 'localhost:9200/_cat/nodes?v'
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1            9          93   0    0.00    0.03     0.05 mdi       *      cOdZg7-
[root@much es]#
~~~

其它的 **`_cat`** API 可以参看 **[cat APIs][cat]**



---

# 总结

使用 rpm 包安装的过程非常简单

这是一个不涉及配置的单实例集群环境

* TOC
{:toc}


---

[elasticsearch]:https://www.elastic.co/products/elasticsearch
[elasticsearch_installation]:https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html
[elasticsearch_dl]:https://www.elastic.co/downloads/elasticsearch
[cat]:https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html#cat
