---
layout: post
title: "Install Telegraf"
author:  wilmosfang
date: 2018-02-13 15:14:57
image: '/assets/img/'
excerpt: '安装 Telegraf'
main-class: influxdb
color: '#4591ed'
tags:
 - telegraf
 - influxdb
categories:
 - influxdb
twitter_text: 'simple process of Telegraf installation'
introduction: 'installation of Telegraf'
---


## 前言

**[InfluxDB][influxdb]** 是一款高性能的时序数据库，见长于标量的时序存储

类似于 **Elasticsearch** 的 **ELK** 技术栈，**InfluxDB** 也有一套 **[TICK][tick]** 技术栈

其中 **[Telegraf][telegraf]** 是前端收集数据的插件，与 **[InfluxDB][influxdb]** 一样也是用 Go 编写的

这里分享一下 **[Telegraf][telegraf]** 的安装方法

参考 **[Installing Telegraf][telegraf_install]**

> **Tip:** 当前的版本为 **telegraf-1.5.2-1**

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


## 安装软件

~~~
[root@much ~]# yum list all | grep influxdb
influxdb.x86_64                         1.4.2-1                        @influxdb
chronograf.x86_64                       1.4.0.0-1                      influxdb
kapacitor.x86_64                        1.4.0-1                        influxdb
pcp-export-pcp2influxdb.x86_64          3.11.8-7.el7                   base     
telegraf.x86_64                         1.5.2-1                        influxdb
[root@much ~]# yum install telegraf
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package telegraf.x86_64 0:1.5.2-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===============================================================================================
 Package               Arch                Version                 Repository             Size
===============================================================================================
Installing:
 telegraf              x86_64              1.5.2-1                 influxdb              8.7 M

Transaction Summary
===============================================================================================
Install  1 Package

Total download size: 8.7 M
Installed size: 27 M
Is this ok [y/d/N]: y
Downloading packages:
telegraf-1.5.2-1.x86_64.rpm                                             | 8.7 MB  00:00:26     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : telegraf-1.5.2-1.x86_64                                                     1/1
Created symlink from /etc/systemd/system/multi-user.target.wants/telegraf.service to /usr/lib/systemd/system/telegraf.service.
  Verifying  : telegraf-1.5.2-1.x86_64                                                     1/1

Installed:
  telegraf.x86_64 0:1.5.2-1                                                                    

Complete!
[root@much ~]# echo $?
0
[root@much ~]#
~~~

可以看看有些什么东西

~~~
[root@much ~]# rpm -ql telegraf-1.5.2-1.x86_64
/etc/logrotate.d/telegraf
/etc/telegraf/telegraf.conf
/etc/telegraf/telegraf.d
/usr/bin/telegraf
/usr/lib/telegraf/scripts/init.sh
/usr/lib/telegraf/scripts/telegraf.service
/var/log/telegraf
[root@much ~]#
~~~

特别少的内容，并且一目了然


## 启动服务

~~~
[root@much ~]# systemctl status telegraf
● telegraf.service - The plugin-driven server agent for reporting metrics into InfluxDB
   Loaded: loaded (/usr/lib/systemd/system/telegraf.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://github.com/influxdata/telegraf
[root@much ~]# systemctl start  telegraf
[root@much ~]# systemctl status telegraf
● telegraf.service - The plugin-driven server agent for reporting metrics into InfluxDB
   Loaded: loaded (/usr/lib/systemd/system/telegraf.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-02-13 23:40:36 CST; 2s ago
     Docs: https://github.com/influxdata/telegraf
 Main PID: 14181 (telegraf)
   CGroup: /system.slice/telegraf.service
           └─14181 /usr/bin/telegraf -config /etc/telegraf/telegraf.conf -config-directory /...

2月 13 23:40:36 much systemd[1]: Started The plugin-driven server agent for reporting m...xDB.
2月 13 23:40:36 much systemd[1]: Starting The plugin-driven server agent for reporting ...B...
2月 13 23:40:36 much telegraf[14181]: 2018-02-13T15:40:36Z I! Starting Telegraf v1.5.2
2月 13 23:40:36 much telegraf[14181]: 2018-02-13T15:40:36Z I! Loaded outputs: influxdb
2月 13 23:40:36 much telegraf[14181]: 2018-02-13T15:40:36Z I! Loaded inputs: inputs.proc...mem
2月 13 23:40:36 much telegraf[14181]: 2018-02-13T15:40:36Z I! Tags enabled: host=much
2月 13 23:40:36 much telegraf[14181]: 2018-02-13T15:40:36Z I! Agent Config: Interval:10s...10s
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]# ps faux | grep telegraf
root     14208  0.0  0.0 112660  1016 pts/0    S+   23:40   0:00          \_ grep --color=auto telegraf
telegraf 14181  0.4  0.4 234184 19588 ?        Ssl  23:40   0:00 /usr/bin/telegraf -config /etc/telegraf/telegraf.conf -config-directory /etc/telegraf/telegraf.d
[root@much ~]# netstat  -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN      1310/java           
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      1310/java           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1517/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1316/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1313/cupsd          
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN      12892/influxd       
tcp        0      0 127.0.0.1:41198         127.0.0.1:8086          ESTABLISHED 14181/telegraf      
tcp        0      0 192.168.56.208:22       192.168.56.1:44464      ESTABLISHED 12566/sshd: root@pt
tcp6       0      0 :::8086                 :::*                    LISTEN      12892/influxd       
tcp6       0      0 :::22                   :::*                    LISTEN      1316/sshd           
tcp6       0      0 :::3000                 :::*                    LISTEN      1311/grafana-server
tcp6       0      0 127.0.0.1:8086          127.0.0.1:41198         ESTABLISHED 12892/influxd       
[root@much ~]#
~~~

可以看到 **telegraf** 连接到了本地的 **8086** 端口

## 配置文件

~~~
[root@much ~]# grep -v "#" /etc/telegraf/telegraf.conf | cat -s

[global_tags]

[agent]
  interval = "10s"
  round_interval = true

  metric_batch_size = 1000

  metric_buffer_limit = 10000

  collection_jitter = "0s"

  flush_interval = "10s"
  flush_jitter = "0s"

  precision = ""

  debug = false
  quiet = false
  logfile = ""

  hostname = ""
  omit_hostname = false

[[outputs.influxdb]]

  retention_policy = ""
  write_consistency = "any"

  timeout = "5s"

[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

[[inputs.disk]]

  ignore_fs = ["tmpfs", "devtmpfs", "devfs"]

[[inputs.diskio]]

[[inputs.kernel]]

[[inputs.mem]]

[[inputs.processes]]

[[inputs.swap]]

[[inputs.system]]

[root@much ~]#
~~~



修改配置文件，收集 cpu 和 mem 信息，存储到 influxdb 中


~~~
[root@much telegraf]# cp telegraf.conf telegraf.conf.bak
[root@much telegraf]# ls
telegraf.conf  telegraf.conf.bak  telegraf.d
[root@much telegraf]# telegraf -sample-config -input-filter cpu:mem -output-filter influxdb > telegraf.conf
[root@much telegraf]#
[root@much telegraf]# grep -v "#" telegraf.conf | cat -s

[global_tags]

[agent]
  interval = "10s"
  round_interval = true

  metric_batch_size = 1000

  metric_buffer_limit = 10000

  collection_jitter = "0s"

  flush_interval = "10s"
  flush_jitter = "0s"

  precision = ""

  debug = false
  quiet = false
  logfile = ""

  hostname = ""
  omit_hostname = false

[[outputs.influxdb]]

  retention_policy = ""
  write_consistency = "any"

  timeout = "5s"

[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

[[inputs.mem]]

[root@much telegraf]#
[root@much telegraf]# systemctl  | grep tele
  telegraf.service                                                                         loaded active running   The plugin-driven server agent for reporting metrics into InfluxDB
[root@much telegraf]# systemctl  restart  telegraf.service
[root@much telegraf]#
~~~

## 查询数据

~~~
[root@much telegraf]# influx -precision rfc3339
Connected to http://localhost:8086 version 1.4.2
InfluxDB shell version: 1.4.2
> show databases
name: databases
name
----
_internal
testdb
telegraf
> use telegraf
Using database telegraf
> show MEASUREMENTS
name: measurements
name
----
cpu
disk
diskio
kernel
mem
processes
swap
system
> show field keys
name: cpu
fieldKey         fieldType
--------         ---------
usage_guest      float
usage_guest_nice float
usage_idle       float
usage_iowait     float
usage_irq        float
usage_nice       float
usage_softirq    float
usage_steal      float
usage_system     float
usage_user       float

name: disk
fieldKey     fieldType
--------     ---------
free         integer
inodes_free  integer
inodes_total integer
inodes_used  integer
total        integer
used         integer
used_percent float

name: diskio
fieldKey         fieldType
--------         ---------
io_time          integer
iops_in_progress integer
read_bytes       integer
read_time        integer
reads            integer
weighted_io_time integer
write_bytes      integer
write_time       integer
writes           integer

name: kernel
fieldKey         fieldType
--------         ---------
boot_time        integer
context_switches integer
interrupts       integer
processes_forked integer

name: mem
fieldKey          fieldType
--------          ---------
active            integer
available         integer
available_percent float
buffered          integer
cached            integer
free              integer
inactive          integer
slab              integer
total             integer
used              integer
used_percent      float

name: processes
fieldKey      fieldType
--------      ---------
blocked       integer
dead          integer
idle          integer
paging        integer
running       integer
sleeping      integer
stopped       integer
total         integer
total_threads integer
unknown       integer
zombies       integer

name: swap
fieldKey     fieldType
--------     ---------
free         integer
in           integer
out          integer
total        integer
used         integer
used_percent float

name: system
fieldKey      fieldType
--------      ---------
load1         float
load15        float
load5         float
n_cpus        integer
n_users       integer
uptime        integer
uptime_format string
> select usage_idle from cpu where  cpu = 'cpu-total' LIMIT 10
name: cpu
time                 usage_idle
----                 ----------
2018-02-13T15:40:50Z 99.64806435394816
2018-02-13T15:41:00Z 99.74924774323333
2018-02-13T15:41:10Z 99.74912192674725
2018-02-13T15:41:20Z 99.89944695826829
2018-02-13T15:41:30Z 99.74912192674725
2018-02-13T15:41:40Z 99.79939819457935
2018-02-13T15:41:50Z 99.79909593168827
2018-02-13T15:42:00Z 99.89959839359034
2018-02-13T15:42:10Z 99.7992975413905
2018-02-13T15:42:20Z 99.74937343358762
>
~~~


---

# 总结

**telegraf** 作为 influxdb 的前端数据采集插件，与 influxdb 的对接非常简单(默认不作配置都是直接连接的本地 influxdb)


* TOC
{:toc}


---


[influxdb]:https://portal.influxdata.com/
[telegraf]:https://www.influxdata.com/time-series-platform/telegraf/
[tick]:https://www.influxdata.com/time-series-platform/
[telegraf_install]:https://docs.influxdata.com/telegraf/v1.5/introduction/installation/
