---
layout: post
title: "Install WordPress"
author:  wilmosfang
date: 2018-04-15 16:32:14
image: '/assets/img/'
excerpt: '安装 WordPress'
main-class: 'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
 - wordpress
categories:
 - php
twitter_text: 'simple process of WordPress installation'
introduction: 'Installation of WordPress'
---
# 前言

**[WordPress][wordpress]** 是一款用 php 实现的开源 CMS 软件

>WordPress is open source software you can use to create a beautiful website, blog, or app

因为特性丰富，使用简单，它在个人博客领域非常受欢迎

在技术型组织里，可以使用它来构建一个文档管理系统或者信息发布平台

这里演示一下如何构建 **[WordPress][wordpress]**

参考 **[Installing WordPress][wordpress_install]**

> **Tip:** 当前的版本为 **WordPress ４.9.5**

---

# 操作

## 依赖

* PHP version 7.2 or greater.
* MySQL version 5.6 or greater OR MariaDB version 10.0 or greater.
* HTTPS support

当前版本的 WordPress 需要满足以上的运行环境

>**Tip:** 详细可以参考 **[Requirements][wordpress_dep]**

## OS 环境

~~~bash
[root@wp ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@wp ~]# hostnamectl 
   Static hostname: wp
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: 3298948d4df94faf88cf158bfaedc570
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@wp ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:8c:97:19 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86257sec preferred_lft 86257sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ab:2c:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.215/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@wp ~]# 
~~~

## 软件环境

~~~
[root@wp ~]# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2018-04-16 08:58:23 EDT; 12min ago
  Process: 1252 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 1164 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 1251 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─1251 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─1525 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql...

4月 16 08:58:20 wp systemd[1]: Starting MariaDB database server...
4月 16 08:58:21 wp mariadb-prepare-db-dir[1164]: Database MariaDB is probabl...
4月 16 08:58:21 wp mariadb-prepare-db-dir[1164]: If this is not the case, ma...
4月 16 08:58:21 wp mysqld_safe[1251]: 180416 08:58:21 mysqld_safe Logging ...'.
4月 16 08:58:21 wp mysqld_safe[1251]: 180416 08:58:21 mysqld_safe Starting...ql
4月 16 08:58:23 wp systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wp ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2018-04-16 08:58:21 EDT; 12min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1175 (httpd)
   Status: "Total requests: 3; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1175 /usr/sbin/httpd -DFOREGROUND
           ├─1432 /usr/sbin/httpd -DFOREGROUND
           ├─1433 /usr/sbin/httpd -DFOREGROUND
           ├─1434 /usr/sbin/httpd -DFOREGROUND
           ├─1435 /usr/sbin/httpd -DFOREGROUND
           ├─1436 /usr/sbin/httpd -DFOREGROUND
           └─1861 /usr/sbin/httpd -DFOREGROUND

4月 16 08:58:20 wp systemd[1]: Starting The Apache HTTP Server...
4月 16 08:58:21 wp httpd[1175]: AH00558: httpd: Could not reliably determi...ge
4月 16 08:58:21 wp systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wp ~]# php --version
PHP 7.2.4 (cli) (built: Mar 27 2018 17:23:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.4, Copyright (c) 1999-2018, by Zend Technologies
[root@wp ~]# rpm -qa | grep httpd
httpd-tools-2.4.6-67.el7.centos.6.x86_64
httpd-2.4.6-67.el7.centos.6.x86_64
[root@wp ~]# rpm -qa | grep mariadb
mariadb-server-5.5.56-2.el7.x86_64
mariadb-libs-5.5.56-2.el7.x86_64
mariadb-5.5.56-2.el7.x86_64
[root@wp ~]# 
~~~

本地的 mariadb 并不最新的，这里我们对 mariadb 进行一个升级

### 停止并删除当前的 mariadb

~~~
[root@wp ~]# systemctl stop mariadb
[root@wp ~]# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since 一 2018-04-16 09:13:36 EDT; 3s ago
  Process: 1252 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 1251 ExecStart=/usr/bin/mysqld_safe --basedir=/usr (code=exited, status=0/SUCCESS)
  Process: 1164 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 1251 (code=exited, status=0/SUCCESS)

4月 16 08:58:20 wp systemd[1]: Starting MariaDB database server...
4月 16 08:58:21 wp mariadb-prepare-db-dir[1164]: Database MariaDB is probabl...
4月 16 08:58:21 wp mariadb-prepare-db-dir[1164]: If this is not the case, ma...
4月 16 08:58:21 wp mysqld_safe[1251]: 180416 08:58:21 mysqld_safe Logging ...'.
4月 16 08:58:21 wp mysqld_safe[1251]: 180416 08:58:21 mysqld_safe Starting...ql
4月 16 08:58:23 wp systemd[1]: Started MariaDB database server.
4月 16 09:13:33 wp systemd[1]: Stopping MariaDB database server...
4月 16 09:13:36 wp systemd[1]: Stopped MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wp ~]# 
[root@wp ~]# yum remove mariadb-server-5.5.56-2.el7.x86_64 mariadb-libs-5.5.56-2.el7.x86_64 mariadb-5.5.56-2.el7.x86_64
Loaded plugins: fastestmirror, langpacks
Resolving Dependencies
--> Running transaction check
---> Package mariadb.x86_64 1:5.5.56-2.el7 will be erased
---> Package mariadb-libs.x86_64 1:5.5.56-2.el7 will be erased
--> Processing Dependency: libmysqlclient.so.18()(64bit) for package: perl-DBD-MySQL-4.023-5.el7.x86_64
--> Processing Dependency: libmysqlclient.so.18()(64bit) for package: 2:postfix-2.10.1-6.el7.x86_64
--> Processing Dependency: libmysqlclient.so.18(libmysqlclient_18)(64bit) for package: perl-DBD-MySQL-4.023-5.el7.x86_64
--> Processing Dependency: libmysqlclient.so.18(libmysqlclient_18)(64bit) for package: 2:postfix-2.10.1-6.el7.x86_64
---> Package mariadb-server.x86_64 1:5.5.56-2.el7 will be erased
--> Running transaction check
---> Package perl-DBD-MySQL.x86_64 0:4.023-5.el7 will be erased
---> Package postfix.x86_64 2:2.10.1-6.el7 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package              Arch         Version                Repository       Size
================================================================================
Removing:
 mariadb              x86_64       1:5.5.56-2.el7         @base            49 M
 mariadb-libs         x86_64       1:5.5.56-2.el7         @anaconda       4.4 M
 mariadb-server       x86_64       1:5.5.56-2.el7         @base            58 M
Removing for dependencies:
 perl-DBD-MySQL       x86_64       4.023-5.el7            @base           323 k
 postfix              x86_64       2:2.10.1-6.el7         @anaconda        12 M

Transaction Summary
================================================================================
Remove  3 Packages (+2 Dependent packages)

Installed size: 124 M
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : 1:mariadb-server-5.5.56-2.el7.x86_64                         1/5 
warning: /var/log/mariadb/mariadb.log saved as /var/log/mariadb/mariadb.log.rpmsave
  Erasing    : 1:mariadb-5.5.56-2.el7.x86_64                                2/5 
  Erasing    : perl-DBD-MySQL-4.023-5.el7.x86_64                            3/5 
  Erasing    : 2:postfix-2.10.1-6.el7.x86_64                                4/5 
  Erasing    : 1:mariadb-libs-5.5.56-2.el7.x86_64                           5/5 
  Verifying  : 1:mariadb-server-5.5.56-2.el7.x86_64                         1/5 
  Verifying  : perl-DBD-MySQL-4.023-5.el7.x86_64                            2/5 
  Verifying  : 1:mariadb-libs-5.5.56-2.el7.x86_64                           3/5 
  Verifying  : 2:postfix-2.10.1-6.el7.x86_64                                4/5 
  Verifying  : 1:mariadb-5.5.56-2.el7.x86_64                                5/5 

Removed:
  mariadb.x86_64 1:5.5.56-2.el7           mariadb-libs.x86_64 1:5.5.56-2.el7   
  mariadb-server.x86_64 1:5.5.56-2.el7   

Dependency Removed:
  perl-DBD-MySQL.x86_64 0:4.023-5.el7       postfix.x86_64 2:2.10.1-6.el7      

Complete!
[root@wp ~]#
~~~

### 配置 mariadb 仓库

mariadb 仓库配置可以参考 **[Setting up MariaDB Repositories][mariadb_repo]**

~~~
[root@wp yum.repos.d]# vim mariadb.repo
[root@wp yum.repos.d]# cat mariadb.repo 
# MariaDB 10.2 CentOS repository list - created 2018-04-16 13:06 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
[root@wp yum.repos.d]# yum list all | grep  -i mariadb
MariaDB-aws-key-management.x86_64        10.2.14-1.el7.centos         mariadb   
MariaDB-backup.x86_64                    10.2.14-1.el7.centos         mariadb   
MariaDB-cassandra-engine.x86_64          10.2.14-1.el7.centos         mariadb   
MariaDB-client.x86_64                    10.2.14-1.el7.centos         mariadb   
MariaDB-common.x86_64                    10.2.14-1.el7.centos         mariadb   
MariaDB-compat.x86_64                    10.2.14-1.el7.centos         mariadb   
MariaDB-connect-engine.x86_64            10.2.14-1.el7.centos         mariadb   
MariaDB-cracklib-password-check.x86_64   10.2.14-1.el7.centos         mariadb   
MariaDB-devel.x86_64                     10.2.14-1.el7.centos         mariadb   
MariaDB-gssapi-server.x86_64             10.2.14-1.el7.centos         mariadb   
MariaDB-oqgraph-engine.x86_64            10.2.14-1.el7.centos         mariadb   
MariaDB-rocksdb-engine.x86_64            10.2.14-1.el7.centos         mariadb   
MariaDB-server.x86_64                    10.2.14-1.el7.centos         mariadb   
MariaDB-shared.x86_64                    10.2.14-1.el7.centos         mariadb   
MariaDB-test.x86_64                      10.2.14-1.el7.centos         mariadb   
MariaDB-tokudb-engine.x86_64             10.2.14-1.el7.centos         mariadb   
galera.x86_64                            25.3.23-1.rhel7.el7.centos   mariadb   
mariadb.x86_64                           1:5.5.56-2.el7               base      
mariadb-bench.x86_64                     1:5.5.56-2.el7               base      
mariadb-devel.i686                       1:5.5.56-2.el7               base      
mariadb-devel.x86_64                     1:5.5.56-2.el7               base      
mariadb-embedded.i686                    1:5.5.56-2.el7               base      
mariadb-embedded.x86_64                  1:5.5.56-2.el7               base      
mariadb-embedded-devel.i686              1:5.5.56-2.el7               base      
mariadb-embedded-devel.x86_64            1:5.5.56-2.el7               base      
mariadb-libs.i686                        1:5.5.56-2.el7               base      
mariadb-libs.x86_64                      1:5.5.56-2.el7               base      
mariadb-server.x86_64                    1:5.5.56-2.el7               base      
mariadb-test.x86_64                      1:5.5.56-2.el7               base      
[root@wp yum.repos.d]# 
~~~

### 安装最新版本 mariadb

~~~
[root@wp yum.repos.d]# yum install MariaDB-server MariaDB-client
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.smartmedia.net.id
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package MariaDB-client.x86_64 0:10.2.14-1.el7.centos will be installed
--> Processing Dependency: MariaDB-common for package: MariaDB-client-10.2.14-1.el7.centos.x86_64
---> Package MariaDB-server.x86_64 0:10.2.14-1.el7.centos will be installed
--> Processing Dependency: galera for package: MariaDB-server-10.2.14-1.el7.centos.x86_64
--> Running transaction check
---> Package MariaDB-common.x86_64 0:10.2.14-1.el7.centos will be installed
--> Processing Dependency: MariaDB-compat for package: MariaDB-common-10.2.14-1.el7.centos.x86_64
---> Package galera.x86_64 0:25.3.23-1.rhel7.el7.centos will be installed
--> Processing Dependency: libboost_program_options.so.1.53.0()(64bit) for package: galera-25.3.23-1.rhel7.el7.centos.x86_64
--> Running transaction check
---> Package MariaDB-compat.x86_64 0:10.2.14-1.el7.centos will be installed
---> Package boost-program-options.x86_64 0:1.53.0-27.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch     Version                       Repository
                                                                           Size
================================================================================
Installing:
 MariaDB-client          x86_64   10.2.14-1.el7.centos          mariadb    48 M
 MariaDB-server          x86_64   10.2.14-1.el7.centos          mariadb   109 M
Installing for dependencies:
 MariaDB-common          x86_64   10.2.14-1.el7.centos          mariadb   157 k
 MariaDB-compat          x86_64   10.2.14-1.el7.centos          mariadb   2.8 M
 boost-program-options   x86_64   1.53.0-27.el7                 base      156 k
 galera                  x86_64   25.3.23-1.rhel7.el7.centos    mariadb   8.0 M

Transaction Summary
================================================================================
Install  2 Packages (+4 Dependent packages)

Total download size: 168 M
Installed size: 716 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/mariadb/packages/MariaDB-10.2.14-centos73-x86_64-common.rpm: Header V4 DSA/SHA1 Signature, key ID 1bb943db: NOKEY
Public key for MariaDB-10.2.14-centos73-x86_64-common.rpm is not installed
(1/6): MariaDB-10.2.14-centos73-x86_64-common.rpm          | 157 kB   00:02     
(2/6): MariaDB-10.2.14-centos73-x86_64-compat.rpm          | 2.8 MB   00:30     
(3/6): boost-program-options-1.53.0-27.el7.x86_64.rpm      | 156 kB   00:00     
(4/6): MariaDB-10.2.14-centos73-x86_64-client.rpm          |  48 MB   03:33     
(5/6): galera-25.3.23-1.rhel7.el7.centos.x86_64.rpm        | 8.0 MB   00:06     
(6/6): MariaDB-10.2.14-centos73-x86_64-server.rpm          | 109 MB   04:40     
--------------------------------------------------------------------------------
Total                                              549 kB/s | 168 MB  05:13     
Retrieving key from https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
Importing GPG key 0x1BB943DB:
 Userid     : "MariaDB Package Signing Key <package-signing-key@mariadb.org>"
 Fingerprint: 1993 69e5 404b d5fc 7d2f e43b cbcb 082a 1bb9 43db
 From       : https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : MariaDB-compat-10.2.14-1.el7.centos.x86_64                   1/6 
  Installing : MariaDB-common-10.2.14-1.el7.centos.x86_64                   2/6 
  Installing : MariaDB-client-10.2.14-1.el7.centos.x86_64                   3/6 
  Installing : boost-program-options-1.53.0-27.el7.x86_64                   4/6 
  Installing : galera-25.3.23-1.rhel7.el7.centos.x86_64                     5/6 
  Installing : MariaDB-server-10.2.14-1.el7.centos.x86_64                   6/6 
  Verifying  : MariaDB-common-10.2.14-1.el7.centos.x86_64                   1/6 
  Verifying  : MariaDB-client-10.2.14-1.el7.centos.x86_64                   2/6 
  Verifying  : MariaDB-server-10.2.14-1.el7.centos.x86_64                   3/6 
  Verifying  : MariaDB-compat-10.2.14-1.el7.centos.x86_64                   4/6 
  Verifying  : galera-25.3.23-1.rhel7.el7.centos.x86_64                     5/6 
  Verifying  : boost-program-options-1.53.0-27.el7.x86_64                   6/6 

Installed:
  MariaDB-client.x86_64 0:10.2.14-1.el7.centos                                  
  MariaDB-server.x86_64 0:10.2.14-1.el7.centos                                  

Dependency Installed:
  MariaDB-common.x86_64 0:10.2.14-1.el7.centos                                  
  MariaDB-compat.x86_64 0:10.2.14-1.el7.centos                                  
  boost-program-options.x86_64 0:1.53.0-27.el7                                  
  galera.x86_64 0:25.3.23-1.rhel7.el7.centos                                    

Complete!
[root@wp yum.repos.d]# echo $?
0
[root@wp yum.repos.d]#
~~~

## 启动 mysql

~~~
[root@wp ~]# systemctl start mariadb
[root@wp ~]# 
[root@wp ~]# systemctl status mariadb
● mariadb.service - MariaDB 10.2.14 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: active (running) since 一 2018-04-16 10:33:31 EDT; 1min 58s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 4654 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
  Process: 4610 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=exited, status=0/SUCCESS)
  Process: 4608 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
 Main PID: 4622 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─4622 /usr/sbin/mysqld

4月 16 10:33:31 wp mysqld[4622]: 2018-04-16 10:33:31 140055682467968 [Note...'.
4月 16 10:33:31 wp mysqld[4622]: 2018-04-16 10:33:31 140055682467968 [ERRO...it
4月 16 10:33:31 wp mysqld[4622]: 2018-04-16 10:33:31 140055595734784 [Warn...st
4月 16 10:33:31 wp mysqld[4622]: 2018-04-16 10:33:31 140055682467968 [Note...ed
4月 16 10:33:31 wp mysqld[4622]: 2018-04-16 10:33:31 140055682467968 [Note...le
4月 16 10:33:31 wp mysqld[4622]: 2018-04-16 10:33:31 140055682467968 [Note...s.
4月 16 10:33:31 wp mysqld[4622]: Version: '10.2.14-MariaDB'  socket: '/var...er
4月 16 10:33:31 wp systemd[1]: Started MariaDB 10.2.14 database server.
4月 16 10:34:18 wp mysqld[4622]: 2018-04-16 10:34:18 140055194482432 [ERRO...it
4月 16 10:34:20 wp mysqld[4622]: 2018-04-16 10:34:20 140055194482432 [ERRO...it
Hint: Some lines were ellipsized, use -l to show in full.
[root@wp ~]# 
~~~

进行安全配置

~~~
[root@wp ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] 
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[root@wp ~]# 
~~~

## 创建数据库和用户

~~~
[root@wp ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 17
Server version: 10.2.14-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE wp_db;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON wp_db.* TO "wp"@"localhost" IDENTIFIED BY "wp";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> EXIT
Bye
[root@wp ~]# 
~~~

## 下载 wordpress 包

~~~
[root@wp wordpress]# wget https://wordpress.org/latest.zip
--2018-04-16 10:11:49--  https://wordpress.org/latest.zip
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9333392 (8.9M) [application/zip]
Saving to: ‘latest.zip’

100%[======================================>] 9,333,392   1.78MB/s   in 5.0s   

2018-04-16 10:11:55 (1.78 MB/s) - ‘latest.zip’ saved [9333392/9333392]

[root@wp wordpress]# echo $?
0
[root@wp wordpress]# ls
latest.zip
[root@wp wordpress]# 
~~~

## 解压 wordpress 包

~~~
[root@wp wordpress]# unzip latest.zip 
Archive:  latest.zip
   creating: wordpress/
  inflating: wordpress/xmlrpc.php    
  inflating: wordpress/wp-blog-header.php  
  inflating: wordpress/readme.html   
  inflating: wordpress/wp-signup.php  
  inflating: wordpress/index.php     
  inflating: wordpress/wp-cron.php
  ...
  ...
  inflating: wordpress/wp-admin/themes.php  
  inflating: wordpress/wp-admin/options-reading.php  
  inflating: wordpress/wp-trackback.php  
  inflating: wordpress/wp-comments-post.php  
[root@wp wordpress]# echo $?
0
[root@wp wordpress]# 
~~~

## 拷贝到合适的位置

同时修改权限

~~~
[root@wp wordpress]# pwd
/root/wordpress
[root@wp wordpress]# ls
latest.zip  wordpress
[root@wp wordpress]# cp -r wordpress/ /var/www/html/
[root@wp wordpress]# ll /var/www/html/
total 12
-rw-r--r--  1 apache apache   17 4月  14 11:33 info.php
drwxr-xr-x 15 apache apache 4096 4月  15 11:03 mediawiki
drwxr-xr-x  5 root   root   4096 4月  16 10:48 wordpress
[root@wp wordpress]# chown -R apache.apache /var/www/html/wordpress/
[root@wp wordpress]# ll /var/www/html/
total 12
-rw-r--r--  1 apache apache   17 4月  14 11:33 info.php
drwxr-xr-x 15 apache apache 4096 4月  15 11:03 mediawiki
drwxr-xr-x  5 apache apache 4096 4月  16 10:48 wordpress
[root@wp wordpress]# 
~~~

## 运行安装脚本

访问链接:

http://192.168.56.215/wordpress/wp-admin/setup-config.php

这个配置过程的结果就是生成一个 wp-config.php 文件

这个配置文件里包含了数据库连接的相关信息

登录后第一个界面

![wordpress](/assets/img/wordpress/wordpress01.png)

可以在这里选择语言

![wordpress](/assets/img/wordpress/wordpress02.png)

说明这个配置过程是为了获取数据库连接信息

![wordpress](/assets/img/wordpress/wordpress03.png)

如实填入数据库的相关信息

![wordpress](/assets/img/wordpress/wordpress04.png)

进行安装

![wordpress](/assets/img/wordpress/wordpress05.png)

配置站点信息

它会自动生成一个用户密码

![wordpress](/assets/img/wordpress/wordpress06.png)

安装成功提示

![wordpress](/assets/img/wordpress/wordpress07.png)

## 进行登录

通过 http://192.168.56.215/wordpress/wp-login.php 可以进行登录

![wordpress](/assets/img/wordpress/wordpress08.png)

简洁而美丽的管理界面

![wordpress](/assets/img/wordpress/wordpress09.png)

这里可以进行各种管理操作

关于 wordpress 的其它细节操作可以基于这个状态进行探索

---

# 总结

WordPress 是一个经典的 LAMP 应用 

所以在 Linux Apache Mysql PHP 等环境准备好了的情况下只需要将相应的软件包解压并放在合适的位置就可以了

不得不说当前的 wordpress 版本颜值极高，插件支持，媒体库的托拽，编辑窗口的易用，内容版本差异的管理，评论支持，主题更换，界面的自定义都有着十分友好的接口

一个开源内容发布系统，真的很喜欢

* TOC
{:toc}

---

[wordpress]:https://wordpress.org/
[wordpress_dep]:https://wordpress.org/about/requirements/
[wordpress_install]:https://codex.wordpress.org/Installing_WordPress
[mariadb_repo]:https://downloads.mariadb.org/mariadb/repositories/#mirror=tuna&distro=CentOS&distro_release=centos7-amd64--centos7&version=10.2