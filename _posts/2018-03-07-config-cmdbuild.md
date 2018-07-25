---
layout: post
title: "Config CMDBuild"
author:  wilmosfang
date: 2018-03-07 14:01:27
image: '/assets/img/'
excerpt: '配置 CMDBuild'
main-class: cmdbuild
color: '#f79a30'
tags:
 - cmdbuild
 - postgresql
 - tomcat
categories:
 - cmdbuild
twitter_text: 'simple config of CMDBuild'
introduction: 'configration of CMDBuild'
---



## 前言

**[CMDBuild][cmdbuild]** 是一款优秀的开源 **CMDB** 软件

>A configuration management database (CMDB) is a repository that acts as a data warehouse for information technology (IT) installations. It holds data relating to a collection of IT assets (commonly referred to as configuration items (CI)), as well as to descriptive relationships between such assets. The repository provides a means of understanding
>
> * the composition of critical assets such as information systems
> * the upstream sources or dependencies of assets
> * the downstream targets of assets

**CMDB** 是 IT 信息扩张过程中工具革新的一个必经之路

准确来说 **CMDB** 应该算作一种 IT 信息管理理念，对信息处理工具的信息通过信息系统进行管理的一种理念

**[CMDBuild][cmdbuild]** 是这种理念的一个开源实现

这里分享一下 **[CMDBuild][cmdbuild]** 的配置方法

参考 **[Technical Manual][cmdbuild_manual]**

> **Tip:** 当前的版本为 **cmdbuild-2.5.0**

---

# 操作


## 环境

~~~
[root@h210 ~]# hostnamectl
   Static hostname: h210
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 739de39e0b1440618015b8dcd595f9f7
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@h210 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 82748sec preferred_lft 82748sec
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
[root@h210 ~]#
~~~

## 依赖

### 硬件依赖

* 主流架构的服务器
* 一个 CMDBuild 实例配置 4G 内存，生产环境下推荐 6-8G
* 一个 CMDBuild 实例配置 120G 磁盘

### 软件依赖

* 支持以下软件的任何操作系统(但是更推荐Linux)
* PostgreSQL 9.4 to 9.6
* Apache Tomcat 6.0 or 7.0 or 8.0 (推荐7.068)
* JDK 1.8
* (可选) PostGIS 1.5.2 or 2.0
* (可选) Alfresco 3.4 用于卡片文档管理，或者 DMS 支持 CMIS 协议

>前面的两篇文章中已经交代了 JDK Tomcat PostgreSQL 的安装方法，这里有不明白的可以翻阅前面的博客进行了解

其它相关细节可以参考 **[System requirements][cmdbuild_dl]**


## 安装过程

* 下载最新项目压缩包　(http://www.cmdbuild.org/download)
* 拷贝根目录的 CMDBuild-{version}.war 拷贝到 Tomcat 的 webapps 中，然后重命名为 cmdbuild.war
* 拷贝 extras 中的 CMDBuild-shark 到 Tomcat 的 webapps 中
* 拷贝 extras/tomcat-libs 中相应版本的依赖库到 Tomcat 的 lib 中
* 启动 tomcat 进程

## 配置 CMDBuild

### 访问配置界面

**`http://192.168.56.210:8080/cmdbuild/`**

![cmdbuild](/assets/img/cmdbuild/cmdbuild01.png)

选中 **Show language choice in the login window** 然后下一步

![cmdbuild](/assets/img/cmdbuild/cmdbuild02.png)

### 配置 pg_hba.conf

需要对默认的 **`pg_hba.conf`** 进行调整，以允许来自本地的密码访问

~~~
[root@h210 data]# pwd
/var/lib/pgsql/10/data
[root@h210 data]# ls
base              pg_ident.conf  pg_stat      pg_xact
current_logfiles  pg_logical     pg_stat_tmp  postgresql.auto.conf
global            pg_multixact   pg_subtrans  postgresql.conf
log               pg_notify      pg_tblspc    postmaster.opts
pg_commit_ts      pg_replslot    pg_twophase  postmaster.pid
pg_dynshmem       pg_serial      PG_VERSION
pg_hba.conf       pg_snapshots   pg_wal
[root@h210 data]# vim pg_hba.conf
[root@h210 data]# grep -v "#" pg_hba.conf  | cat -s

local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
[root@h210 data]#
~~~

添加了以下一条配置，以允许来自本地的密码访问

**`host    all             all             127.0.0.1/32            md5`**

测试连接

~~~
[root@h210 data]# psql -U postgres -h 127.0.0.1  -W
Password for user postgres:
psql (10.3)
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileg
es   
-----------+----------+----------+-------------+-------------+------------------
-----
 cmdbuild  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres      
    +
           |          |          |             |             | postgres=CTc/post
gres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres      
    +
           |          |          |             |             | postgres=CTc/post
gres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(5 rows)

postgres=#
~~~

>如果不加以上配置会怎样呢

~~~
[root@h210 data]# psql -U postgres -h 127.0.0.1  -W
Password for user postgres:
psql: FATAL:  Ident authentication failed for user "postgres"
[root@h210 data]# echo $?
2
[root@h210 data]#
~~~

日志中会有如下报错

~~~
2018-03-07 23:07:58.008 CST [3955] LOG:  could not connect to Ident server at address "127.0.0.1", port 113: Connection refused
2018-03-07 23:07:58.008 CST [3955] FATAL:  Ident authentication failed for user "postgres"
2018-03-07 23:07:58.008 CST [3955] DETAIL:  Connection matched pg_hba.conf line 83: "host    all             all             127.0.0.1/32            ident"
~~~

即便我的密码没错，连接还是被拒绝了，因为匹配上了 **pg_hba.conf** 中的一条策略，在本地尝试使用 TCP 进行连接的时候，会使用 **pg_ident.conf** 中的映射关系，将本地用户映射成数据库中的用户进行登录，而 **pg_ident.conf** 中并没有此映射，所以登录被拒绝了

>**Tip:** 修改完 **pg_hba.conf** 后，需要对服务进行重载，以使变更后的配置生效

~~~
[root@h210 data]# vim pg_hba.conf
[root@h210 data]# systemctl reload postgresql-10
[root@h210 data]#
~~~


### 数据库配置

#### CMDBuild Database section

* 创建一个空库
* 选择一个兼容 CMDBuild 1.0 的已经存在的库
* 创建一个有测试数据库的新库
* 库名


#### Database connection

* PostgreSQL 数据库所在服务器的 IP (host name or IP address)
* PostgreSQL 数据库服务所开放的端口 (the default port is 5432)
* 访问 PostgreSQL 的用户名 (for DBA activities)
* 访问 PostgreSQL 的密码 (for DBA activities)

点击 **[Test connection]** 来测试数据库的联通性


![cmdbuild](/assets/img/cmdbuild/cmdbuild03.png)

### 创建账号

![cmdbuild](/assets/img/cmdbuild/cmdbuild04.png)


![cmdbuild](/assets/img/cmdbuild/cmdbuild05.png)

### 登录界面

![cmdbuild](/assets/img/cmdbuild/cmdbuild06.png)

**CMDBuild** 支持很多种语言，可以选择自己熟悉的语言

![cmdbuild](/assets/img/cmdbuild/cmdbuild07.png)

### CMDB分两个模块

数据管理模块和系统管理模块

![cmdbuild](/assets/img/cmdbuild/cmdbuild08.png)


![cmdbuild](/assets/img/cmdbuild/cmdbuild09.png)

切换成中文后

![cmdbuild](/assets/img/cmdbuild/cmdbuild10.png)


![cmdbuild](/assets/img/cmdbuild/cmdbuild11.png)

虽然还有很多处没有翻译过来，不过这个界面对于英语基础偏弱一点的同学来说，已经容易很多了

---

# 总结

从整个过程来看，**CMDBuild** 的配置还是十分简单的

已经被此软件封装了很多细节，包括 schema 的导入

关于 **CMDBuild** 的使用后面有机会再进行展开

* TOC
{:toc}


---


[cmdbuild]:http://www.cmdbuild.org/en/
[cmdbuild_manual]:http://www.cmdbuild.org/en/documentazione/manuali/technical-manual
[cmdbuild_dl]:http://www.cmdbuild.org/en/download
