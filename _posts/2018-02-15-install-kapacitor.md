---
layout: post
title: "Install Kapacitor"
author:  wilmosfang
date: 2018-02-15 15:08:30
image: '/assets/img/'
excerpt: '安装 Kapacitor'
main-class: influxdb
color: '#4591ed'
tags:
 - kapacitor
 - tick
 - influxdb
categories:
 - influxdb
twitter_text: 'simple process of Kapacitor installation'
introduction: 'installation of Kapacitor'
---


## 前言

**[InfluxDB][influxdb]** 是一款高性能的时序数据库，见长于标量的时序存储

类似于 **Elasticsearch** 的 **ELK** 技术栈，**InfluxDB** 也有一套 **[TICK][tick]** 技术栈

其中 **[Kapacitor][kapacitor]** 是一个实时的流处理引擎，可以对流数据或批量数据进行计算和处理

>Kapacitor is a native data processing engine in the **[TICK][tick]** Stack. It can process both stream and batch data from InfluxDB. It lets you plug in your own custom logic or user-defined functions to process alerts with dynamic thresholds, match metrics for patterns, compute statistical anomalies, and perform specific actions based on these alerts like dynamic load rebalancing

这里分享一下 **[Kapacitor][kapacitor]** 的安装方法

参考 **[Installing Kapacitor][kapacitor_install]**

> **Tip:** 当前的版本为 **kapacitor-1.4.0**

---

# 操作


## 环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 1f9d9f1fc29440c8874b993d9455c898
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
       valid_lft 83032sec preferred_lft 83032sec
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


## 配置仓库

~~~
[root@much ~]# cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
> [influxdb]
> name = InfluxDB Repository - RHEL \$releasever
> baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
> enabled = 1
> gpgcheck = 1
> gpgkey = https://repos.influxdata.com/influxdb.key
> EOF
[influxdb]
name = InfluxDB Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
[root@much ~]# cat /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
[root@much ~]#
~~~

## 依赖

在 **[TICK][tick]** 技术栈中，**[Kapacitor][kapacitor]** 和其它组件的关系如下

![tick](/assets/img/influxdb/TICK.png)

**[InfluxDB][influxdb]** 和 **[Chronograf][chronograf]** 还有 **[Telegraf][telegraf]** 的版本要求

* InfluxDB - While Kapacitor does not require InfluxDB, it is the easiest integration to setup and so it will be used in this guide. InfluxDB >= 1.3.x will be needed.
* Telegraf - Telegraf >= 1.4.x will be required.
* Kapacitor - The latest Kapacitor binary and installation packages can be found at the downloads page.
* Terminal - The Kapacitor client application works using the CLI and so a basic terminal will be needed to issue commands.

## 安装软件

~~~
[root@much ~]# yum list all | grep kapacitor
kapacitor.x86_64                        1.4.0-1                        influxdb
[root@much ~]# yum list all | grep influxdb
chronograf.x86_64                       1.4.0.0-1                      @influxdb
influxdb.x86_64                         1.4.2-1                        @influxdb
telegraf.x86_64                         1.5.2-1                        @influxdb
kapacitor.x86_64                        1.4.0-1                        influxdb
pcp-export-pcp2influxdb.x86_64          3.11.8-7.el7                   base     
[root@much ~]# yum install kapacitor.x86_64
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 7.7 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
influxdb                                                 | 2.5 kB     00:00     
kibana-6.x                                               | 1.3 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/2): epel/x86_64/updateinfo                              | 881 kB   00:06     
(2/2): epel/x86_64/primary_db                              | 6.2 MB   00:25     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package kapacitor.x86_64 0:1.4.0-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package            Arch            Version             Repository         Size
================================================================================
Installing:
 kapacitor          x86_64          1.4.0-1             influxdb           21 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 21 M
Installed size: 21 M
Is this ok [y/d/N]: y
Downloading packages:
kapacitor-1.4.0.x86_64.rpm                                 |  21 MB   01:03     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : kapacitor-1.4.0-1.x86_64                                     1/1
  Verifying  : kapacitor-1.4.0-1.x86_64                                     1/1

Installed:
  kapacitor.x86_64 0:1.4.0-1                                                    

Complete!
[root@much ~]#
[root@much ~]# echo $?
0
[root@much ~]# rpm -qa | grep kapacitor
kapacitor-1.4.0-1.x86_64
[root@much ~]#
~~~

可以看看有些什么东西

~~~
[root@much ~]# rpm -ql kapacitor-1.4.0-1.x86_64
/etc/kapacitor
/etc/kapacitor/kapacitor.conf
/etc/logrotate.d/kapacitor
/usr/bin/kapacitor
/usr/bin/kapacitord
/usr/bin/tickfmt
/usr/lib/kapacitor
/usr/lib/kapacitor/scripts
/usr/lib/kapacitor/scripts/init.sh
/usr/lib/kapacitor/scripts/kapacitor.service
/usr/share/bash-completion/completions/kapacitor
/var/lib/kapacitor
/var/log/kapacitor
[root@much ~]#
~~~

特别少的内容，并且一目了然


## 启动服务

~~~
[root@much ~]# systemctl status kapacitor
● kapacitor.service - Time series data processing engine.
   Loaded: loaded (/usr/lib/systemd/system/kapacitor.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://github.com/influxdb/kapacitor
[root@much ~]# systemctl start kapacitor
[root@much ~]#
[root@much ~]# systemctl status kapacitor
● kapacitor.service - Time series data processing engine.
   Loaded: loaded (/usr/lib/systemd/system/kapacitor.service; disabled; vendor preset: disabled)
   Active: active (running) since 五 2018-02-16 01:49:32 CST; 23s ago
     Docs: https://github.com/influxdb/kapacitor
 Main PID: 13530 (kapacitord)
   CGroup: /system.slice/kapacitor.service
           └─13530 /usr/bin/kapacitord -config /etc/kapacitor/kapacitor.conf

2月 16 01:49:32 much systemd[1]: Starting Time series data processing engine....
2月 16 01:49:32 much kapacitord[13530]: '##:::'##::::'###::::'########:::::'###:::::'######::'####:'########::'#######::'########::
2月 16 01:49:32 much kapacitord[13530]: ##::'##::::'## ##::: ##.... ##:::'## ##:::'##... ##:. ##::... ##..::'##.... ##: ##.... ##:
2月 16 01:49:32 much kapacitord[13530]: ##:'##::::'##:. ##:: ##:::: ##::'##:. ##:: ##:::..::: ##::::: ##:::: ##:::: ##: ##:::: ##:
2月 16 01:49:32 much kapacitord[13530]: #####::::'##:::. ##: ########::'##:::. ##: ##:::::::: ##::::: ##:::: ##:::: ##: ########::
2月 16 01:49:32 much kapacitord[13530]: ##. ##::: #########: ##.....::: #########: ##:::::::: ##::::: ##:::: ##:::: ##: ##.. ##:::
2月 16 01:49:32 much kapacitord[13530]: ##:. ##:: ##.... ##: ##:::::::: ##.... ##: ##::: ##:: ##::::: ##:::: ##:::: ##: ##::. ##::
2月 16 01:49:32 much kapacitord[13530]: ##::. ##: ##:::: ##: ##:::::::: ##:::: ##:. ######::'####:::: ##::::. #######:: ##:::. ##:
2月 16 01:49:32 much kapacitord[13530]: ..::::..::..:::::..::..:::::::::..:::::..:::......:::....:::::..::::::.......:::..:::::..::
2月 16 01:49:32 much kapacitord[13530]: 2018/02/16 01:49:32 Using configuration at: /etc/kapacitor/kapacitor.conf
[root@much ~]# ps faux | grep kapacitor
root     13578  0.0  0.0 112660  1020 pts/0    S+   01:50   0:00  |       \_ grep --color=auto kapacitor
kapacit+ 13530  0.4  0.6 187780 25156 ?        Ssl  01:49   0:00 /usr/bin/kapacitord -config /etc/kapacitor/kapacitor.conf
[root@much ~]#
[root@much ~]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN      1322/java           
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      1322/java           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1555/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1316/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1324/cupsd          
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN      1319/influxd        
tcp        0      0 127.0.0.1:44566         127.0.0.1:9092          ESTABLISHED 1319/influxd        
tcp        0      0 127.0.0.1:54082         127.0.0.1:8086          ESTABLISHED 1323/telegraf       
tcp        0      0 127.0.0.1:44568         127.0.0.1:9092          ESTABLISHED 1319/influxd        
tcp        0      0 127.0.0.1:54628         127.0.0.1:8086          ESTABLISHED 1321/chronograf     
tcp        0      0 127.0.0.1:54632         127.0.0.1:8086          ESTABLISHED 13530/kapacitord    
tcp        0      0 127.0.0.1:54630         127.0.0.1:8086          ESTABLISHED 1321/chronograf     
tcp        0      0 192.168.56.208:22       192.168.56.1:50012      ESTABLISHED 2634/sshd: root@pts
tcp        0      0 192.168.56.208:22       192.168.56.1:50096      ESTABLISHED 3125/sshd: root@pts
tcp        0      0 10.0.2.15:47048         104.131.151.204:443     ESTABLISHED 13530/kapacitord    
tcp6       0      0 :::8086                 :::*                    LISTEN      1319/influxd        
tcp6       0      0 :::22                   :::*                    LISTEN      1316/sshd           
tcp6       0      0 :::3000                 :::*                    LISTEN      1320/grafana-server
tcp6       0      0 :::8888                 :::*                    LISTEN      1321/chronograf     
tcp6       0      0 :::9092                 :::*                    LISTEN      13530/kapacitord    
tcp6       0      0 192.168.56.208:8888     192.168.56.1:39824      ESTABLISHED 1321/chronograf     
tcp6       0      0 127.0.0.1:9092          127.0.0.1:44566         ESTABLISHED 13530/kapacitord    
tcp6       0      0 192.168.56.208:8888     192.168.56.1:39814      ESTABLISHED 1321/chronograf     
tcp6       0      0 127.0.0.1:9092          127.0.0.1:44568         ESTABLISHED 13530/kapacitord    
tcp6       0      0 127.0.0.1:8086          127.0.0.1:54628         ESTABLISHED 1319/influxd        
tcp6       0      0 127.0.0.1:8086          127.0.0.1:54082         ESTABLISHED 1319/influxd        
tcp6       0      0 192.168.56.208:8888     192.168.56.1:39812      ESTABLISHED 1321/chronograf     
tcp6       0      0 192.168.56.208:8888     192.168.56.1:39826      ESTABLISHED 1321/chronograf     
tcp6       0      0 127.0.0.1:8086          127.0.0.1:54630         ESTABLISHED 1319/influxd        
tcp6       0      0 127.0.0.1:8086          127.0.0.1:54632         ESTABLISHED 1319/influxd        
tcp6       0      0 192.168.56.208:8888     192.168.56.1:39854      ESTABLISHED 1321/chronograf     
[root@much ~]#
~~~

可以看到 **kapacitord** 监听在了本地的 **9092** 端口

## 打开防火墙

~~~
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 3000/tcp 8888/tcp 8080/tcp 5601/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]# firewall-cmd --add-port 9092/tcp --permanent
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
  ports: 3000/tcp 5601/tcp 9092/tcp 8888/tcp 8080/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]#
~~~


## 进行访问



![kapacitor](/assets/img/kapacitor/kapacitor01.png)


**Username** 和 **Password** 可以为空

![kapacitor](/assets/img/kapacitor/kapacitor02.png)

配置

![kapacitor](/assets/img/kapacitor/kapacitor03.png)

**Kapacitor** 使用一种叫做 **TICKscript** 的 DSL 来定义流处理任务

>Kapacitor uses a DSL called TICKscript to define tasks. Each TICKscript defines a pipeline that tells Kapacitor which data to process and how

可以参考 **[etting started with Kapacitor][kapacitor_getting_started]**

关于 **TICKscript** 的用法，后面再作详细说明

---

# 总结

**Kapacitor** 作为 influxdb 的数据处理组件，与 influxdb 的对接非常简单(默认不作配置都是直接连接的本地 influxdb)

结合 **TICKscript** 可以实现复杂的流数据处理，这方面的内容，后面逐步展开

* TOC
{:toc}


---


[influxdb]:https://portal.influxdata.com/
[kapacitor]:https://www.influxdata.com/time-series-platform/kapacitor/
[chronograf]:https://www.influxdata.com/time-series-platform/chronograf/
[telegraf]:https://www.influxdata.com/time-series-platform/telegraf/
[tick]:https://www.influxdata.com/time-series-platform/
[kapacitor_install]:http://docs.influxdata.com/kapacitor/v1.4/introduction/installation/
[kapacitor_getting_started]:http://docs.influxdata.com/kapacitor/v1.4/introduction/getting_started/
