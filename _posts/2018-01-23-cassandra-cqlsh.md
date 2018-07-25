---
layout: post
title: "Cassandra Cqlsh"
author:  wilmosfang
date: 2018-01-23 14:59:52
image: '/assets/img/'
excerpt: 'Cqlsh 的简单用法'
main-class: cassandra
color: '#90bd50'
tags:
 - cassandra
 - cqlsh
categories:
 - cassandra
twitter_text: 'the usage of Cqlsh'
introduction: 'client of cassandra'
---


# 前言


**[Cassandra][cassandra]** 是一款开源分布式数据库软件，可以提供高容错，高性能，高可用，高弹性，可线性扩展的特性

在 **CAP** 理论中，它很好地实践了 **AP** 牺牲了 **C**, 它是一个最终一致性数据库

**[Cqlsh][cqlsh]** 是 **[Cassandra][cassandra]** 的客户端

下面分享一下 **[Cqlsh][cqlsh]** 的简单使用方法

参考 **[Cassandra Tools][cassandra_tools]**

> **Tip:** 当前版本 **Cassandra 3.11.1** 和 **cqlsh 5.0.1**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: f13f6477c2db43f9a7febda448d0468e
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
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84011sec preferred_lft 84011sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link
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

* Python 2.7

>For using cqlsh, the latest version of Python 2.7

~~~
[root@much ~]# python -V
Python 2.7.5
[root@much ~]#
~~~

## 路径

~~~
[root@much ~]# rpm -ql cassandra | grep cqlsh | grep bin
/usr/bin/cqlsh
/usr/bin/cqlsh.py
[root@much ~]#
~~~

在安装 **cassandra** 的过程中就已经自带了客户端


## 连接

~~~
[root@much ~]# cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh>
~~~

默认会连接本地的 9042 号端口

## 常用命令

### CONSISTENCY

获取或设定一致性级别

~~~
cqlsh> CONSISTENCY
Current consistency level is ONE.
cqlsh>
~~~

有以下几种级别

* ANY
* ONE
* TWO
* THREE
* QUORUM
* ALL
* LOCAL_QUORUM
* LOCAL_ONE
* SERIAL
* LOCAL_SERIAL


### SERIAL CONSISTENCY

获取或设定串行一致性级别

~~~
cqlsh> SERIAL CONSISTENCY
Current serial consistency level is SERIAL.
cqlsh>
~~~

有以下几种级别

* SERIAL
* LOCAL_SERIAL

此串行一致性级别主要用于更新操作

>The serial consistency level is only used by conditional updates (INSERT, UPDATE and DELETE with an IF condition)

### show VERSION

查看当前版本

~~~
cqlsh> show VERSION
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
cqlsh>
~~~

### show HOST

查看当前连接的集群主机和端口

~~~
cqlsh> show HOST
Connected to Test Cluster at localhost:9042.
cqlsh>
~~~

### SHOW SESSION

打印出一个 session 的详细内容

~~~
cqlsh> SHOW
HOST     SESSION  VERSION  
cqlsh> SHOW SESSION
Improper SHOW command.
cqlsh> SHOW SESSION 95ac6470-327e-11e6-beca-dfb660d92ad8
Session 95ac6470-327e-11e6-beca-dfb660d92ad8 wasn't found.
cqlsh>
~~~

### SOURCE

执行指定文件中的 CQL 语句集

~~~
[root@much ~]# echo 'SHOW VERSION' > abc.cql
[root@much ~]# cat abc.cql
SHOW VERSION
[root@much ~]# cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> SOURCE '~/abc.cql'
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
cqlsh>
~~~

### CAPTURE

抓取查询结果

特殊命令的输出将不会被抓取

~~~
[root@much ~]# > abc.log
[root@much ~]# cat abc.log
[root@much ~]# cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> CAPTURE 'abc.log'
Now capturing query output to 'abc.log'.
cqlsh> SELECT cluster_name from system.local ;
cqlsh> SELECT listen_address from system.local ;
cqlsh> show VERSION
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
cqlsh> CAPTURE off;
cqlsh> CAPTURE
Currently not capturing query output.
cqlsh> CAPTURE ;
Currently not capturing query output.
cqlsh> quit
[root@much ~]# cat abc.log

 cluster_name
--------------
 Test Cluster

(1 rows)

 listen_address
----------------
      127.0.0.1

(1 rows)
[root@much ~]#
~~~

>Begins capturing command output and appending it to a specified file. Output will not be shown at the console while it is captured.
>Only query result output is captured. Errors and output from cqlsh-only commands will still be shown in the cqlsh session
>o inspect the current capture configuration, use CAPTURE with no arguments.


### HELP

帮助命令

不加参数显现所有的命令

加上命令就对指定的命令进行解释

~~~
cqlsh> HELP

Documented shell commands:
===========================
CAPTURE  CLS          COPY  DESCRIBE  EXPAND  LOGIN   SERIAL  SOURCE   UNICODE
CLEAR    CONSISTENCY  DESC  EXIT      HELP    PAGING  SHOW    TRACING

CQL help topics:
================
AGGREGATES               CREATE_KEYSPACE           DROP_TRIGGER      TEXT     
ALTER_KEYSPACE           CREATE_MATERIALIZED_VIEW  DROP_TYPE         TIME     
ALTER_MATERIALIZED_VIEW  CREATE_ROLE               DROP_USER         TIMESTAMP
ALTER_TABLE              CREATE_TABLE              FUNCTIONS         TRUNCATE
ALTER_TYPE               CREATE_TRIGGER            GRANT             TYPES    
ALTER_USER               CREATE_TYPE               INSERT            UPDATE   
APPLY                    CREATE_USER               INSERT_JSON       USE      
ASCII                    DATE                      INT               UUID     
BATCH                    DELETE                    JSON            
BEGIN                    DROP_AGGREGATE            KEYWORDS        
BLOB                     DROP_COLUMNFAMILY         LIST_PERMISSIONS
BOOLEAN                  DROP_FUNCTION             LIST_ROLES      
COUNTER                  DROP_INDEX                LIST_USERS      
CREATE_AGGREGATE         DROP_KEYSPACE             PERMISSIONS     
CREATE_COLUMNFAMILY      DROP_MATERIALIZED_VIEW    REVOKE          
CREATE_FUNCTION          DROP_ROLE                 SELECT          
CREATE_INDEX             DROP_TABLE                SELECT_JSON     

cqlsh> HELP HELP

        HELP [cqlsh only]

        Gives information about cqlsh commands. To see available topics,
        enter "HELP" without any arguments. To see help on a topic,
        use "HELP <topic>".

cqlsh> HELP SELECT_JSON
*** No browser to display CQL help. URL for help topic SELECT_JSON : https://cassandra.apache.org/doc/cql3/CQL-3.2.html#selectJson
cqlsh> HELP SELECT
*** No browser to display CQL help. URL for help topic SELECT : https://cassandra.apache.org/doc/cql3/CQL-3.2.html#selectStmt
cqlsh> HELP CAPTURE

        CAPTURE [cqlsh only]

        Begins capturing command output and appending it to a specified file.
        Output will not be shown at the console while it is captured.

        Usage:

          CAPTURE '<file>';
          CAPTURE OFF;
          CAPTURE;

        That is, the path to the file to be appended to must be given inside a
        string literal. The path is interpreted relative to the current working
        directory. The tilde shorthand notation ('~/mydir') is supported for
        referring to $HOME.

        Only query result output is captured. Errors and output from cqlsh-only
        commands will still be shown in the cqlsh session.

        To stop capturing output and show it in the cqlsh session again, use
        CAPTURE OFF.

        To inspect the current capture configuration, use CAPTURE with no
        arguments.

cqlsh> HELP APPLY
*** No browser to display CQL help. URL for help topic APPLY : https://cassandra.apache.org/doc/cql3/CQL-3.2.html#batchStmt
cqlsh> HELP EXIT

        EXIT/QUIT [cqlsh only]

        Exits cqlsh.

cqlsh>
~~~

---

# 总结

CQL 的这些命令还是比较简单好用的，但目前为止还没涉及到操作数据

* TOC
{:toc}


---

[cassandra]:http://cassandra.apache.org/
[cqlsh]:http://cassandra.apache.org/doc/latest/tools/cqlsh.html
[cassandra_tools]:http://cassandra.apache.org/doc/latest/tools/index.html
