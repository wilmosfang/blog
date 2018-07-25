---
layout: post
title: "Install InfluxDB"
author:  wilmosfang
date: 2018-02-11 16:00:36
image: '/assets/img/'
excerpt: '安装 InfluxDB'
main-class: influxdb
color: '#4591ed'
tags:
 - influxdb
categories:
 - influxdb
twitter_text: 'simple process of InfluxDB installation'
introduction: 'installation of InfluxDB'
---



## 前言

**[InfluxDB][influxdb]** 是一款高性能的时序数据库

>InfluxDB is a time series database built from the ground up to handle high write and query loads. It is the second piece of the TICK stack. InfluxDB is meant to be used as a backing store for any use case involving large amounts of timestamped data, including DevOps monitoring, application metrics, IoT sensor data, and real-time analytics

**[InfluxDB][influxdb]** 见长于标量的时序存储

当前最主要的应用场景有以下两个方面

* 监控数据
* IoT 数据流存储

这两方面的特性 **Elasticsearch** 也有覆盖，那它们两者的区别是什么呢，可以参考下面的文章

**[InfluxDB vs. Elasticsearch for Time Series Data & Metrics Benchmark][Influxdb_vs_es]**

总体来讲，在日志与全文检索方面 **Elasticsearch** 还是无可匹敌的，但是在纯标量的存储检索压缩和响应方面 **InfluxDB** 会有更明显的优势，所以事实上各有侧重

同时，类似于 **Elasticsearch** 的 **ELK** 技术栈，**InfluxDB** 也有一套 **[TICK][tick]** 技术栈

这里分享一下 **[InfluxDB][influxdb]** 的安装方法

参考 **[Installing InfluxDB][grafana_install]**

> **Tip:** 当前的版本为 **InfluxDB v1.4.3**

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
[root@much ~]# yum install influxdb
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 6.8 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
influxdb                                                 | 2.5 kB     00:00     
kibana-6.x                                               | 1.3 kB     00:00     
updates                                                  | 3.4 kB     00:00     
epel/x86_64/primary_db         FAILED                                           
http://mirror1.ku.ac.th/fedora/epel/7/x86_64/repodata/7dedd75641340b89a13c2b5be3489c3d408b5661c7bf6172baf4157e668d2179-primary.sqlite.bz2: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below knowledge base article

https://access.redhat.com/articles/1320623

If above article doesn't help to resolve this issue please create a bug on https://bugs.centos.org/

(1/3): influxdb/7/x86_64/primary_db                        |  25 kB   00:01     
(2/3): epel/x86_64/updateinfo                              | 879 kB   00:03     
(3/3): epel/x86_64/primary_db                              | 6.2 MB   00:27     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package influxdb.x86_64 0:1.4.2-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package            Arch             Version           Repository          Size
================================================================================
Installing:
 influxdb           x86_64           1.4.2-1           influxdb            20 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 20 M
Installed size: 20 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/influxdb/packages/influxdb-1.4.2.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 2582e0c5: NOKEY=======================================================================-] 336 kB/s |  20 MB  00:00:00 ETA
Public key for influxdb-1.4.2.x86_64.rpm is not installed
influxdb-1.4.2.x86_64.rpm                                                                                                                                                                                              |  20 MB  00:00:58     
Retrieving key from https://repos.influxdata.com/influxdb.key
Importing GPG key 0x2582E0C5:
 Userid     : "InfluxDB Packaging Service <support@influxdb.com>"
 Fingerprint: 05ce 1508 5fc0 9d18 e99e fb22 684a 14cf 2582 e0c5
 From       : https://repos.influxdata.com/influxdb.key
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : influxdb-1.4.2-1.x86_64                                      1/1
Created symlink from /etc/systemd/system/influxd.service to /usr/lib/systemd/system/influxdb.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/influxdb.service to /usr/lib/systemd/system/influxdb.service.
  Verifying  : influxdb-1.4.2-1.x86_64                                      1/1

Installed:
  influxdb.x86_64 0:1.4.2-1                                                     

Complete!
[root@much ~]# echo $?
0
[root@much ~]#
~~~

可以看看有些什么东西

~~~
[root@much ~]# rpm -qa | grep influ
influxdb-1.4.2-1.x86_64
[root@much ~]# rpm -ql  influxdb-1.4.2-1.x86_64
/etc/influxdb/influxdb.conf
/etc/logrotate.d/influxdb
/usr/bin/influx
/usr/bin/influx_inspect
/usr/bin/influx_stress
/usr/bin/influx_tsm
/usr/bin/influxd
/usr/lib/influxdb/scripts/influxdb.service
/usr/lib/influxdb/scripts/init.sh
/usr/share/man/man1/influx.1.gz
/usr/share/man/man1/influx_inspect.1.gz
/usr/share/man/man1/influx_stress.1.gz
/usr/share/man/man1/influx_tsm.1.gz
/usr/share/man/man1/influxd-backup.1.gz
/usr/share/man/man1/influxd-config.1.gz
/usr/share/man/man1/influxd-restore.1.gz
/usr/share/man/man1/influxd-run.1.gz
/usr/share/man/man1/influxd-version.1.gz
/usr/share/man/man1/influxd.1.gz
/var/lib/influxdb
/var/log/influxdb
[root@much ~]#
~~~

特别少的内容，并且一目了然


## 启动服务

~~~
[root@much ~]# systemctl status influxdb
● influxdb.service - InfluxDB is an open-source, distributed, time series database
   Loaded: loaded (/usr/lib/systemd/system/influxdb.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.influxdata.com/influxdb/
[root@much ~]# systemctl start influxdb
[root@much ~]# systemctl status influxdb
● influxdb.service - InfluxDB is an open-source, distributed, time series database
   Loaded: loaded (/usr/lib/systemd/system/influxdb.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-02-13 22:01:51 CST; 1s ago
     Docs: https://docs.influxdata.com/influxdb/
 Main PID: 12892 (influxd)
   CGroup: /system.slice/influxdb.service
           └─12892 /usr/bin/influxd -config /etc/influxdb/influxdb.conf

2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Starting pre...on
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Starting sna...ot
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Starting con...er
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Starting HTT...pd
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Authenticati...pd
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Listening on...pd
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Starting ret...on
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Storing stat...or
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Sending usag...om
2月 13 22:01:51 much influxd[12892]: [I] 2018-02-13T14:01:51Z Listening fo...ls
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]# ps faux | grep influxdb
root     12902  0.0  0.0 112660  1016 pts/0    S+   22:02   0:00          \_ grep --color=auto influxdb
influxdb 12892  1.0  0.2 282824 11896 ?        Ssl  22:01   0:00 /usr/bin/influxd -config /etc/influxdb/influxdb.conf
[root@much ~]#   
[root@much ~]# netstat  -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN      1310/java           
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      1310/java           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1517/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1316/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1313/cupsd          
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN      12892/influxd       
tcp        0      0 10.0.2.15:46188         104.131.151.204:443     ESTABLISHED 12892/influxd       
tcp        0      0 192.168.56.208:22       192.168.56.1:44464      ESTABLISHED 12566/sshd: root@pt
tcp6       0      0 :::8086                 :::*                    LISTEN      12892/influxd       
tcp6       0      0 :::22                   :::*                    LISTEN      1316/sshd           
tcp6       0      0 :::3000                 :::*                    LISTEN      1311/grafana-server
[root@much ~]#
~~~

可以看到 **influxd** 在 **8088** 和 **8086** 上都有监听

## 配置文件

~~~
[root@much ~]# grep -v "#" /etc/influxdb/influxdb.conf  | cat -s

[meta]
  dir = "/var/lib/influxdb/meta"

[data]
  dir = "/var/lib/influxdb/data"

  wal-dir = "/var/lib/influxdb/wal"

[coordinator]

[retention]

[shard-precreation]

[monitor]

[http]

[ifql]

[subscriber]

[[graphite]]

[[collectd]]

[[opentsdb]]

[[udp]]

[continuous_queries]

[root@much ~]#
~~~

## 目录结构

**influxdb** 会创建一个用户，并且有着简洁的目录结构

如果自己创建一定要留意权限问题

~~~
[root@much ~]# tail /etc/passwd -n 3
elasticsearch:x:982:977:elasticsearch user:/home/elasticsearch:/sbin/nologin
grafana:x:981:976:grafana user:/usr/share/grafana:/sbin/nologin
influxdb:x:980:975::/var/lib/influxdb:/bin/false
[root@much ~]# ll /var/lib/influxdb -d
drwxr-xr-x 5 influxdb influxdb 41 2月  13 22:02 /var/lib/influxdb
[root@much ~]# ll /var/lib/influxdb
total 0
drwxr-xr-x 3 influxdb influxdb 23 2月  13 22:02 data
drwxr-xr-x 2 influxdb influxdb 21 2月  13 22:02 meta
drwx------ 3 influxdb influxdb 23 2月  13 22:02 wal
[root@much ~]# tree /var/lib/influxdb
/var/lib/influxdb
├── data
│   └── _internal
│       └── monitor
│           └── 1
├── meta
│   └── meta.db
└── wal
    └── _internal
        └── monitor
            └── 1
                └── _00001.wal

9 directories, 2 files
[root@much ~]#
~~~

**wal** 用来存放 **write ahead log**

**meta** 用来存放元数据，也就是用来管理数据的数据

**data** 用来存放真实的数据


## influx 命令

~~~
[root@much ~]# influx --help
Usage of influx:
  -version
       Display the version and exit.
  -host 'host name'
       Host to connect to.
  -port 'port #'
       Port to connect to.
  -socket 'unix domain socket'
       Unix socket to connect to.
  -database 'database name'
       Database to connect to the server.
  -password 'password'
      Password to connect to the server.  Leaving blank will prompt for password (--password '').
  -username 'username'
       Username to connect to the server.
  -ssl
        Use https for requests.
  -unsafeSsl
        Set this when connecting to the cluster using https and not use SSL verification.
  -execute 'command'
       Execute command and quit.
  -format 'json|csv|column'
       Format specifies the format of the server responses:  json, csv, or column.
  -precision 'rfc3339|h|m|s|ms|u|ns'
       Precision specifies the format of the timestamp:  rfc3339, h, m, s, ms, u or ns.
  -consistency 'any|one|quorum|all'
       Set write consistency level: any, one, quorum, or all
  -pretty
       Turns on pretty print for the json format.
  -import
       Import a previous database export from file
  -pps
       How many points per second the import will allow.  By default it is zero and will not throttle importing.
  -path
       Path to file to import
  -compressed
       Set to true if the import file is compressed

Examples:

    # Use influx in a non-interactive mode to query the database "metrics" and pretty print json:
    $ influx -database 'metrics' -execute 'select * from cpu' -format 'json' -pretty

    # Connect to a specific database on startup and set database context:
    $ influx -database 'metrics' -host 'localhost' -port '8086'

[root@much ~]#
~~~


## 连接数据库

~~~
[root@much ~]# influx -precision rfc3339
Connected to http://localhost:8086 version 1.4.2
InfluxDB shell version: 1.4.2
> help
Usage:
        connect <host:port>   connects to another node specified by host:port
        auth                  prompts for username and password
        pretty                toggles pretty print for the json format
        chunked               turns on chunked responses from server
        chunk size <size>     sets the size of the chunked responses.  Set to 0 to reset to the default chunked size
        use <db_name>         sets current database
        format <format>       specifies the format of the server responses: json, csv, or column
        precision <format>    specifies the format of the timestamp: rfc3339, h, m, s, ms, u or ns
        consistency <level>   sets write consistency level: any, one, quorum, or all
        history               displays command history
        settings              outputs the current settings for the shell
        clear                 clears settings such as database or retention policy.  run 'clear' for help
        exit/quit/ctrl+d      quits the influx shell

        show databases        show database names
        show series           show series information
        show measurements     show measurement information
        show tag keys         show tag key information
        show field keys       show field key information

        A full list of influxql commands can be found at:
        https://docs.influxdata.com/influxdb/latest/query_language/spec/

>
~~~

**`-precision rfc3339`** 是在指时间格式

默认会连接 **http://localhost:8086**

## 常用命令

~~~
> pretty
Pretty print enabled
> pretty
Pretty print disabled
> history
help
quit
help
history
settings
show databases
pretty
> settings
Setting           Value
--------          --------
Host              localhost:8086
Username          
Database          
RetentionPolicy   
Pretty            false
Format            column
Write Consistency all
Chunked           true
Chunk Size        0

> show databases
name: databases
name
----
_internal
>
~~~

## 创建数据库

~~~
> create  database testdb
> show databases
name: databases
name
----
_internal
testdb
> use testdb
Using database testdb
>
> show series
> show measurements
> show tag keys
> show field keys
>
~~~


---

# 总结

**influxdb** 是一个非常简洁的数据库，从目录结构就可以看出来，使用也很简单，从命令集就可以看出来

简单是高效的基础

并且随着大数据的进一步发展，可以预见，此类时序数据库会越来越受到欢迎


* TOC
{:toc}


---


[influxdb]:https://portal.influxdata.com/
[Influxdb_vs_es]:https://www.influxdata.com/blog/influxdb-markedly-elasticsearch-in-time-series-data-metrics-benchmark/
[tick]:https://www.influxdata.com/time-series-platform/
[grafana_install]:https://docs.influxdata.com/influxdb/v1.4/introduction/installation/
