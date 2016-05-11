---
layout: post
title:  日志服务器
author: wilmosfang
categories:  linux rsyslog loganalyzer mysql php
wc: 1514  4989 60666 
excerpt:  安装 httpd，安装 mysql，安装 php，安装 rsyslog，创建 schema，rsyslog 的服务端和客户端配置，审批本地所有操作，安装 LogAnalyzer，配置 LogAnalyzer，查看日志统计，LogAnalyzer 登录测试，搜索日志，修改配置、报错和处理办法，相关注意事项
comments: true
---


# 前言

**[LogAnalyzer][loganalyzer]** 是一款syslog日志和其他网络事件数据的Web前端


>Adiscon LogAnalyzer is a web interface to syslog and other network event data. It provides easy browsing, analysis of realtime network events and reporting services.


>对于任何一个系统而言，日志都是致关重要的，通过日志，系统管理员可以查看系统的运行状况，开发人员可以快速定位问题、分析问题

当系统或应用很分散时，日志就会很分散，给日志分析带来一定不便，awk，sed，grep 等工具的局限性愈发明显，ELK 可以很好解决这个问题，感兴趣可以参考之前的 **[ELK 搭建][elk]** ，ELK 可以高效且有针对性地解决这类问题，同时也有其复杂度和相应的基础开销，有时对于一套相对较小的系统用起来会有点重，这时使用系统自带的 rsyslog 结合 LogAnalyzer 就可以很方便的满足需求


这里分享一下使用 **loganalyzer、rsyslog、mysql、apache** 搭建一个简单日志服务器的操作过程，详细可以参考 **David Tang** 的 **[CentOS 6.5下利用Rsyslog+LogAnalyzer+MySQL部署日志服务器][ref_doc]** (这篇文章准确来说不算原创，是参考他博客的一次实践) 和 **[官方文档][loganalyzer_doc]**


> **Tip:** 当前的 LogAnalyzer 最新版本为 **LogAnalyzer v4.1.3 (v4-beta)** ，最新稳定版为 **LogAnalyzer v3.6.6 (v3-stable)** 


---


# 概要

* TOC
{:toc}



---

## 环境

~~~
[root@h105 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h105 ~]# uname -a 
Linux h105 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h105 ~]# 
~~~



---

## 安装 httpd

~~~
[root@h105 log]# yum install httpd -y
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
epel/metalink                                                                                                | 4.3 kB     00:00     
 * epel: ftp.cuhk.edu.hk
 * extras: mirror.bit.edu.cn
 * updates: mirrors.pubyun.com
epel                                                                                                         | 4.3 kB     00:00     
epel/primary_db                                                                                              | 5.9 MB     00:04     
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.2.15-39.el6.centos will be updated
---> Package httpd.x86_64 0:2.2.15-47.el6.centos.4 will be an update
--> Processing Dependency: httpd-tools = 2.2.15-47.el6.centos.4 for package: httpd-2.2.15-47.el6.centos.4.x86_64
--> Running transaction check
---> Package httpd-tools.x86_64 0:2.2.15-39.el6.centos will be updated
---> Package httpd-tools.x86_64 0:2.2.15-47.el6.centos.4 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                       Arch                     Version                                     Repository                 Size
====================================================================================================================================
Updating:
 httpd                         x86_64                   2.2.15-47.el6.centos.4                      updates                   831 k
Updating for dependencies:
 httpd-tools                   x86_64                   2.2.15-47.el6.centos.4                      updates                    77 k

Transaction Summary
====================================================================================================================================
Upgrade       2 Package(s)

Total download size: 908 k
Downloading Packages:
(1/2): httpd-2.2.15-47.el6.centos.4.x86_64.rpm                                                               | 831 kB     00:00     
(2/2): httpd-tools-2.2.15-47.el6.centos.4.x86_64.rpm                                                         |  77 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               1.3 MB/s | 908 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : httpd-tools-2.2.15-47.el6.centos.4.x86_64                                                                        1/4 
  Updating   : httpd-2.2.15-47.el6.centos.4.x86_64                                                                              2/4 
  Cleanup    : httpd-2.2.15-39.el6.centos.x86_64                                                                                3/4 
  Cleanup    : httpd-tools-2.2.15-39.el6.centos.x86_64                                                                          4/4 
  Verifying  : httpd-2.2.15-47.el6.centos.4.x86_64                                                                              1/4 
  Verifying  : httpd-tools-2.2.15-47.el6.centos.4.x86_64                                                                        2/4 
  Verifying  : httpd-2.2.15-39.el6.centos.x86_64                                                                                3/4 
  Verifying  : httpd-tools-2.2.15-39.el6.centos.x86_64                                                                          4/4 

Updated:
  httpd.x86_64 0:2.2.15-47.el6.centos.4                                                                                             

Dependency Updated:
  httpd-tools.x86_64 0:2.2.15-47.el6.centos.4                                                                                       

Complete!
[root@h105 log]# 
~~~

---

### 启动httpd服务并设置开机启动

~~~
[root@h105 log]# /etc/init.d/httpd start 
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.100.105 for ServerName
                                                           [  OK  ]
[root@h105 log]# chkconfig httpd --list
httpd          	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@h105 log]# chkconfig httpd on
[root@h105 log]# chkconfig httpd --list
httpd          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@h105 log]# vim /etc/sysconfig/iptables
[root@h105 log]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h105 log]# 
[root@h105 log]# iptables -L -nv  | grep 80
    4   200 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:80 
[root@h105 log]# 
~~~

此时已经可以通过 **`http://192.168.100.105/`** 从外部进行访问了

展示的是 apache 的页面


---

## 安装 mysql


这里我使用 percona 版本的mysql



~~~
[root@h105 mysql]# ls
Percona-Server-client-56-5.6.27-rel76.0.el6.x86_64.rpm  Percona-Server-server-56-5.6.27-rel76.0.el6.x86_64.rpm
Percona-Server-devel-56-5.6.27-rel76.0.el6.x86_64.rpm   Percona-Server-shared-56-5.6.27-rel76.0.el6.x86_64.rpm
[root@h105 mysql]# rpm -ivh Percona-Server-*
warning: Percona-Server-client-56-5.6.27-rel76.0.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:Percona-Server-shared-5########################################### [ 25%]
   2:Percona-Server-client-5########################################### [ 50%]
   3:Percona-Server-server-5########################################### [ 75%]
2016-05-10 20:59:42 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2016-05-10 20:59:42 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-76.0) starting as process 2661 ...
2016-05-10 20:59:42 2661 [Note] InnoDB: Using atomics to ref count buffer pool pages
2016-05-10 20:59:42 2661 [Note] InnoDB: The InnoDB memory heap is disabled
2016-05-10 20:59:42 2661 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2016-05-10 20:59:42 2661 [Note] InnoDB: Memory barrier is not used
2016-05-10 20:59:42 2661 [Note] InnoDB: Compressed tables use zlib 1.2.3
2016-05-10 20:59:42 2661 [Note] InnoDB: Using Linux native AIO
2016-05-10 20:59:42 2661 [Note] InnoDB: Using CPU crc32 instructions
2016-05-10 20:59:42 2661 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2016-05-10 20:59:42 2661 [Note] InnoDB: Completed initialization of buffer pool
2016-05-10 20:59:42 2661 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2016-05-10 20:59:42 2661 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2016-05-10 20:59:42 2661 [Note] InnoDB: Database physically writes the file full: wait...
2016-05-10 20:59:42 2661 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2016-05-10 20:59:44 2661 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2016-05-10 20:59:46 2661 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2016-05-10 20:59:46 2661 [Warning] InnoDB: New log files created, LSN=45781
2016-05-10 20:59:46 2661 [Note] InnoDB: Doublewrite buffer not found: creating new
2016-05-10 20:59:46 2661 [Note] InnoDB: Doublewrite buffer created
2016-05-10 20:59:46 2661 [Note] InnoDB: 128 rollback segment(s) are active.
2016-05-10 20:59:46 2661 [Warning] InnoDB: Creating foreign key constraint system tables.
2016-05-10 20:59:46 2661 [Note] InnoDB: Foreign key constraint system tables created
2016-05-10 20:59:46 2661 [Note] InnoDB: Creating tablespace and datafile system tables.
2016-05-10 20:59:46 2661 [Note] InnoDB: Tablespace and datafile system tables created.
2016-05-10 20:59:46 2661 [Note] InnoDB: Waiting for purge to start
2016-05-10 20:59:46 2661 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-76.0 started; log sequence number 0
2016-05-10 20:59:46 2661 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
2016-05-10 20:59:46 2661 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
2016-05-10 20:59:47 2661 [Note] Binlog end
2016-05-10 20:59:47 2661 [Note] InnoDB: FTS optimize thread exiting.
2016-05-10 20:59:47 2661 [Note] InnoDB: Starting shutdown...
2016-05-10 20:59:48 2661 [Note] InnoDB: Shutdown completed; log sequence number 1625977


2016-05-10 20:59:50 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2016-05-10 20:59:50 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-76.0) starting as process 2686 ...
2016-05-10 20:59:50 2686 [Note] InnoDB: Using atomics to ref count buffer pool pages
2016-05-10 20:59:50 2686 [Note] InnoDB: The InnoDB memory heap is disabled
2016-05-10 20:59:50 2686 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2016-05-10 20:59:50 2686 [Note] InnoDB: Memory barrier is not used
2016-05-10 20:59:50 2686 [Note] InnoDB: Compressed tables use zlib 1.2.3
2016-05-10 20:59:50 2686 [Note] InnoDB: Using Linux native AIO
2016-05-10 20:59:50 2686 [Note] InnoDB: Using CPU crc32 instructions
2016-05-10 20:59:50 2686 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2016-05-10 20:59:50 2686 [Note] InnoDB: Completed initialization of buffer pool
2016-05-10 20:59:50 2686 [Note] InnoDB: Highest supported file format is Barracuda.
2016-05-10 20:59:53 2686 [Note] InnoDB: 128 rollback segment(s) are active.
2016-05-10 20:59:53 2686 [Note] InnoDB: Waiting for purge to start
2016-05-10 20:59:53 2686 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-76.0 started; log sequence number 1625977
2016-05-10 20:59:53 2686 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
2016-05-10 20:59:53 2686 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
2016-05-10 20:59:53 2686 [Note] Binlog end
2016-05-10 20:59:53 2686 [Note] InnoDB: FTS optimize thread exiting.
2016-05-10 20:59:53 2686 [Note] InnoDB: Starting shutdown...
2016-05-10 20:59:55 2686 [Note] InnoDB: Shutdown completed; log sequence number 1625987




PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  /usr/bin/mysqladmin -u root password 'new-password'
  /usr/bin/mysqladmin -u root -h h105 password 'new-password'

Alternatively you can run:

  /usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

Please report any problems at
 https://bugs.launchpad.net/percona-server/+filebug

The latest information about Percona Server is available on the web at
  http://www.percona.com/software/percona-server

Support Percona by buying support at
 http://www.percona.com/products/mysql-support

Percona Server is distributed with several useful UDF (User Defined Function) from Percona Toolkit.
Run the following commands to create these functions:
mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"
mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"
mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"
See http://www.percona.com/doc/percona-server/5.6/management/udf_percona_toolkit.html for more details
   4:Percona-Server-devel-56########################################### [100%]
[root@h105 mysql]# 
~~~

### 设置mysql密码

~~~
[root@h105 mysql]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server)...                         [  OK  ]
[root@h105 mysql]# mysqladmin -uroot password 'mysql'
Warning: Using a password on the command line interface can be insecure.
[root@h105 mysql]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.27-76.0 Percona Server (GPL), Release 76.0, Revision 5498987

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)

mysql> quit
Bye
[root@h105 mysql]# 
~~~

---

## 安装 php


~~~
[root@h105 mysql]# yum install php 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * epel: ftp.cuhk.edu.hk
 * extras: mirror.bit.edu.cn
 * updates: mirrors.pubyun.com
Resolving Dependencies
--> Running transaction check
---> Package php.x86_64 0:5.3.3-46.el6_7.1 will be installed
--> Processing Dependency: php-common(x86-64) = 5.3.3-46.el6_7.1 for package: php-5.3.3-46.el6_7.1.x86_64
--> Processing Dependency: php-cli(x86-64) = 5.3.3-46.el6_7.1 for package: php-5.3.3-46.el6_7.1.x86_64
--> Running transaction check
---> Package php-cli.x86_64 0:5.3.3-46.el6_7.1 will be installed
---> Package php-common.x86_64 0:5.3.3-46.el6_7.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                        Arch                       Version                                Repository                   Size
====================================================================================================================================
Installing:
 php                            x86_64                     5.3.3-46.el6_7.1                       updates                     1.1 M
Installing for dependencies:
 php-cli                        x86_64                     5.3.3-46.el6_7.1                       updates                     2.2 M
 php-common                     x86_64                     5.3.3-46.el6_7.1                       updates                     529 k

Transaction Summary
====================================================================================================================================
Install       3 Package(s)

Total download size: 3.8 M
Installed size: 13 M
Is this ok [y/N]: y
Downloading Packages:
(1/3): php-5.3.3-46.el6_7.1.x86_64.rpm                                                                       | 1.1 MB     00:00     
(2/3): php-cli-5.3.3-46.el6_7.1.x86_64.rpm                                                                   | 2.2 MB     00:01     
(3/3): php-common-5.3.3-46.el6_7.1.x86_64.rpm                                                                | 529 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               1.4 MB/s | 3.8 MB     00:02     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : php-common-5.3.3-46.el6_7.1.x86_64                                                                               1/3 
  Installing : php-cli-5.3.3-46.el6_7.1.x86_64                                                                                  2/3 
  Installing : php-5.3.3-46.el6_7.1.x86_64                                                                                      3/3 
  Verifying  : php-common-5.3.3-46.el6_7.1.x86_64                                                                               1/3 
  Verifying  : php-cli-5.3.3-46.el6_7.1.x86_64                                                                                  2/3 
  Verifying  : php-5.3.3-46.el6_7.1.x86_64                                                                                      3/3 

Installed:
  php.x86_64 0:5.3.3-46.el6_7.1                                                                                                     

Dependency Installed:
  php-cli.x86_64 0:5.3.3-46.el6_7.1                               php-common.x86_64 0:5.3.3-46.el6_7.1                              

Complete!
[root@h105 mysql]#
~~~


---


### 测试 php 运行环境


~~~
[root@h105 mysql]# cd /var/www/html/
[root@h105 html]# cat >index.php  <<EOF
> <?php
> phpinfo();
> ?>
> EOF
[root@h105 html]# cat index.php 
<?php
phpinfo();
?>
[root@h105 html]# getenforce 
Enforcing
[root@h105 html]# setenforce 0 
[root@h105 html]# getenforce 
Permissive
[root@h105 html]#
~~~

要关掉 SElinux ，否则它会捣乱


加载新环境，重启httpd 服务

~~~
[root@h105 html]# su - root 
[root@h105 ~]# /etc/init.d/httpd restart 
Stopping httpd:                                            [  OK  ]
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.100.105 for ServerName
                                                           [  OK  ]
[root@h105 ~]# 
~~~

再次访问主页

![php_test.png](/images/loganalyzer/php_test.png)


---

## 安装 rsyslog 

其实系统已经自带了

~~~
[root@h105 ~]# rpm -qa | grep rsyslog
rsyslog-5.8.10-8.el6.x86_64
[root@h105 ~]#
~~~

### 安装 rsyslog 连接 mysql 数据库的模块

因为数据最后是写到 mysql 里，所以要安装 rsyslog 操作 mysql 的模块


~~~
[root@h105 ~]# yum -y install rsyslog-mysql
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * epel: ftp.cuhk.edu.hk
 * extras: mirror.bit.edu.cn
 * updates: mirrors.pubyun.com
Resolving Dependencies
--> Running transaction check
---> Package rsyslog-mysql.x86_64 0:5.8.10-8.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                            Arch                        Version                             Repository                 Size
====================================================================================================================================
Installing:
 rsyslog-mysql                      x86_64                      5.8.10-8.el6                        base                       21 k

Transaction Summary
====================================================================================================================================
Install       1 Package(s)

Total download size: 21 k
Installed size: 15 k
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : rsyslog-mysql-5.8.10-8.el6.x86_64                                                                                1/1 
  Verifying  : rsyslog-mysql-5.8.10-8.el6.x86_64                                                                                1/1 

Installed:
  rsyslog-mysql.x86_64 0:5.8.10-8.el6                                                                                               

Complete!
[root@h105 ~]# 
~~~


---

## 创建schema


根据提供的脚本，要预先定义好 schema 和相应表结构

~~~
[root@h105 ~]# cd /usr/share/doc/rsyslog-mysql-5.8.10/
[root@h105 rsyslog-mysql-5.8.10]# ls
createDB.sql
[root@h105 rsyslog-mysql-5.8.10]# wc -l createDB.sql 
37 createDB.sql
[root@h105 rsyslog-mysql-5.8.10]# cat createDB.sql 
CREATE DATABASE Syslog;
USE Syslog;
CREATE TABLE SystemEvents
(
        ID int unsigned not null auto_increment primary key,
        CustomerID bigint,
        ReceivedAt datetime NULL,
        DeviceReportedTime datetime NULL,
        Facility smallint NULL,
        Priority smallint NULL,
        FromHost varchar(60) NULL,
        Message text,
        NTSeverity int NULL,
        Importance int NULL,
        EventSource varchar(60),
        EventUser varchar(60) NULL,
        EventCategory int NULL,
        EventID int NULL,
        EventBinaryData text NULL,
        MaxAvailable int NULL,
        CurrUsage int NULL,
        MinUsage int NULL,
        MaxUsage int NULL,
        InfoUnitID int NULL ,
        SysLogTag varchar(60),
        EventLogType varchar(60),
        GenericFileName VarChar(60),
        SystemID int NULL
);

CREATE TABLE SystemEventsProperties
(
        ID int unsigned not null auto_increment primary key,
        SystemEventID int NULL ,
        ParamName varchar(255) NULL ,
        ParamValue text NULL
);
[root@h105 rsyslog-mysql-5.8.10]# mysql -u root -pmysql < createDB.sql 
Warning: Using a password on the command line interface can be insecure.
[root@h105 rsyslog-mysql-5.8.10]# 
~~~

> **Tip:** 注意到这里并没有索引，应该是和日志的 append only 属性相关


---

### 查看表结构

~~~
[root@h105 rsyslog-mysql-5.8.10]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 5.6.27-76.0 Percona Server (GPL), Release 76.0, Revision 5498987

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| Syslog             |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

mysql> use Syslog;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------------+
| Tables_in_Syslog       |
+------------------------+
| SystemEvents           |
| SystemEventsProperties |
+------------------------+
2 rows in set (0.00 sec)

mysql> desc SystemEvents;
+--------------------+------------------+------+-----+---------+----------------+
| Field              | Type             | Null | Key | Default | Extra          |
+--------------------+------------------+------+-----+---------+----------------+
| ID                 | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| CustomerID         | bigint(20)       | YES  |     | NULL    |                |
| ReceivedAt         | datetime         | YES  |     | NULL    |                |
| DeviceReportedTime | datetime         | YES  |     | NULL    |                |
| Facility           | smallint(6)      | YES  |     | NULL    |                |
| Priority           | smallint(6)      | YES  |     | NULL    |                |
| FromHost           | varchar(60)      | YES  |     | NULL    |                |
| Message            | text             | YES  |     | NULL    |                |
| NTSeverity         | int(11)          | YES  |     | NULL    |                |
| Importance         | int(11)          | YES  |     | NULL    |                |
| EventSource        | varchar(60)      | YES  |     | NULL    |                |
| EventUser          | varchar(60)      | YES  |     | NULL    |                |
| EventCategory      | int(11)          | YES  |     | NULL    |                |
| EventID            | int(11)          | YES  |     | NULL    |                |
| EventBinaryData    | text             | YES  |     | NULL    |                |
| MaxAvailable       | int(11)          | YES  |     | NULL    |                |
| CurrUsage          | int(11)          | YES  |     | NULL    |                |
| MinUsage           | int(11)          | YES  |     | NULL    |                |
| MaxUsage           | int(11)          | YES  |     | NULL    |                |
| InfoUnitID         | int(11)          | YES  |     | NULL    |                |
| SysLogTag          | varchar(60)      | YES  |     | NULL    |                |
| EventLogType       | varchar(60)      | YES  |     | NULL    |                |
| GenericFileName    | varchar(60)      | YES  |     | NULL    |                |
| SystemID           | int(11)          | YES  |     | NULL    |                |
+--------------------+------------------+------+-----+---------+----------------+
24 rows in set (0.00 sec)

mysql> desc SystemEventsProperties;
+---------------+------------------+------+-----+---------+----------------+
| Field         | Type             | Null | Key | Default | Extra          |
+---------------+------------------+------+-----+---------+----------------+
| ID            | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| SystemEventID | int(11)          | YES  |     | NULL    |                |
| ParamName     | varchar(255)     | YES  |     | NULL    |                |
| ParamValue    | text             | YES  |     | NULL    |                |
+---------------+------------------+------+-----+---------+----------------+
4 rows in set (0.02 sec)

mysql> 
mysql> select * from  SystemEventsProperties;
Empty set (0.00 sec)

mysql> select * from  SystemEvents;
Empty set (0.01 sec)

mysql>
~~~


---

### 创建 logger 用户并赋予相应权限

~~~
mysql> grant all on Syslog.* to logger@localhost identified by '123456';
Query OK, 0 rows affected (0.02 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>
~~~

---

## 配置 rsyslog 服务端

~~~
[root@h105 rsyslog-mysql-5.8.10]# grep -v "^#" /etc/rsyslog.conf | grep -v "^$"
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$IncludeConfig /etc/rsyslog.d/*.conf
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
authpriv.*                                              /var/log/secure
mail.*                                                  -/var/log/maillog
cron.*                                                  /var/log/cron
*.emerg                                                 *
uucp,news.crit                                          /var/log/spooler
local7.*                                                /var/log/boot.log
$template SpiceTmpl,"%TIMESTAMP%.%TIMESTAMP:::date-subseconds% %syslogtag% %syslogseverity-text%:%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
:programname, startswith, "spice-vdagent"	/var/log/spice-vdagent.log;SpiceTmpl
[root@h105 rsyslog-mysql-5.8.10]# vim /etc/rsyslog.conf 
[root@h105 rsyslog-mysql-5.8.10]# grep -v "^#" /etc/rsyslog.conf | grep -v "^$"
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
$ModLoad immark  # provides --MARK-- message capability
$ModLoad imudp
$UDPServerRun 514
$ModLoad ommysql
*.* :ommysql:localhost,Syslog,logger,123456
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$IncludeConfig /etc/rsyslog.d/*.conf
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
authpriv.*                                              /var/log/secure
mail.*                                                  -/var/log/maillog
cron.*                                                  /var/log/cron
*.emerg                                                 *
uucp,news.crit                                          /var/log/spooler
local7.*                                                /var/log/boot.log
$template SpiceTmpl,"%TIMESTAMP%.%TIMESTAMP:::date-subseconds% %syslogtag% %syslogseverity-text%:%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
:programname, startswith, "spice-vdagent"	/var/log/spice-vdagent.log;SpiceTmpl
[root@h105 rsyslog-mysql-5.8.10]#
~~~

前后的差异如下：

~~~
[root@h105 rsyslog-mysql-5.8.10]# diff /tmp/before  /tmp/after
2a3,7
> $ModLoad immark  # provides --MARK-- message capability
> $ModLoad imudp
> $UDPServerRun 514
> $ModLoad ommysql
> *.* :ommysql:localhost,Syslog,logger,123456
[root@h105 rsyslog-mysql-5.8.10]#
~~~

主要就是打开了 **udp 514** 端口以接受其它服务器传来的日志，打开了往 mysql 中写数据的通道，然后打开一个产生 **`-- MARK --`** 标记信息的特性


### 重启服务

~~~
[root@h105 rsyslog-mysql-5.8.10]# /etc/init.d/rsyslog restart 
Shutting down system logger:                               [  OK  ]
Starting system logger:                                    [  OK  ]
[root@h105 rsyslog-mysql-5.8.10]# 
~~~

### 打开防火墙

~~~
[root@h105 rsyslog-mysql-5.8.10]# netstat  -an | grep 514
udp        0      0 0.0.0.0:514                 0.0.0.0:*                               
udp        0      0 :::514                      :::*                                    
[root@h105 rsyslog-mysql-5.8.10]# iptables -L -nv | grep 514
[root@h105 rsyslog-mysql-5.8.10]# vim /etc/sysconfig/iptables
[root@h105 rsyslog-mysql-5.8.10]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h105 rsyslog-mysql-5.8.10]# iptables -L -nv | grep 514
    0     0 ACCEPT     udp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW udp dpt:514 
[root@h105 rsyslog-mysql-5.8.10]# 
~~~

---

## 客户端配置

~~~
[root@h202 ~]# grep -v "^#" /etc/rsyslog.conf | grep -v "^$"
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$IncludeConfig /etc/rsyslog.d/*.conf
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
authpriv.*                                              /var/log/secure
mail.*                                                  -/var/log/maillog
cron.*                                                  /var/log/cron
*.emerg                                                 *
uucp,news.crit                                          /var/log/spooler
local7.*                                                /var/log/boot.log
$template SpiceTmpl,"%TIMESTAMP%.%TIMESTAMP:::date-subseconds% %syslogtag% %syslogseverity-text%:%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
:programname, startswith, "spice-vdagent"	/var/log/spice-vdagent.log;SpiceTmpl
[root@h202 ~]# vim /etc/rsyslog.conf 
[root@h202 ~]# grep -v "^#" /etc/rsyslog.conf | grep -v "^$"
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$IncludeConfig /etc/rsyslog.d/*.conf
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
authpriv.*                                              /var/log/secure
mail.*                                                  -/var/log/maillog
cron.*                                                  /var/log/cron
*.emerg                                                 *
uucp,news.crit                                          /var/log/spooler
local7.*                                                /var/log/boot.log
*.* 							@192.168.100.105
$template SpiceTmpl,"%TIMESTAMP%.%TIMESTAMP:::date-subseconds% %syslogtag% %syslogseverity-text%:%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
:programname, startswith, "spice-vdagent"	/var/log/spice-vdagent.log;SpiceTmpl
[root@h202 ~]#
~~~

前后的差异主要为

~~~
[root@h202 ~]# diff /tmp/before /tmp/after 
11a12
> *.* 							@192.168.100.105
[root@h202 ~]#
~~~

增加了一条，将本地的日志记录到远程的服务器 192.168.100.105 ， 不指定端口就是默认的 **udp 514**


### 重启客户端服务


~~~
[root@h202 ~]# /etc/init.d/rsyslog restart 
Shutting down system logger:                               [  OK  ]
Starting system logger:                                    [  OK  ]
[root@h202 ~]#
~~~

---

## 审计本地所有操作

将客户端执行的所有命令写入系统日志/var/log/messages中

~~~
[root@h202 ~]# vim /etc/bashrc 
[root@h202 ~]# tail -n 3 /etc/bashrc 


export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; });logger "[euid=$(whoami)]":$(who am i):[`pwd`]"$msg"; }'
[root@h202 ~]# 
~~~

在当前环境下生效

~~~
[root@h202 ~]# source /etc/bashrc 
[root@h202 ~]# 
~~~

---

### 客户端操作测试

~~~
[root@h202 ~]# ls
anaconda-ks.cfg  Downloads           ip.log  Music     plot    Templates                         vmware-tools-distrib
Desktop          install.log         logger  packages  Public  Videos                            zk
Documents        install.log.syslog  mtools  Pictures  ruby    VMwareTools-9.6.2-1688356.tar.gz
[root@h202 ~]# echo abc 
abc
[root@h202 ~]# crontab -l 
no crontab for root
[root@h202 ~]# date
Tue May 10 22:03:59 CST 2016
[root@h202 ~]# pwd
/root
[root@h202 ~]# cd xxxxx
-bash: cd: xxxxx: No such file or directory
[root@h202 ~]# cat /etc/passwd | grep root 
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@h202 ~]# grep root /etc/shadow
root:$6$Y7oPl.HJqPuiLgcO$.SEke/qishToW6PlZC.UewgjQaLp9YPPTFqvLbh47F6QUhHqPhrLT6fqdEfqYr6TIGyOl0XuAiUnlvJflixfO/:16545:0:99999:7:::
[root@h202 ~]# 
~~~

---

### 服务端检查日志

~~~
[root@h105 ~]# tailf /var/log/messages
May 10 22:03:21 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]source /etc/bashrc
May 10 22:03:21 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]source /etc/bashrc
May 10 22:03:23 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]ls
May 10 22:03:40 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]echo abc
May 10 22:03:47 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]crontab -l
May 10 22:03:59 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]date
May 10 22:04:02 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]pwd
May 10 22:04:05 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]cd xxxxx
May 10 22:04:13 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]cat /etc/passwd | grep root
May 10 22:04:23 h202 root: [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]grep root /etc/shadow
...
...
...
~~~

通过这种方式已经可以实现操作审记了


---

### 查看服务端数据库中的日志


检查数据库确保数据也写了一份到mysql中

~~~
[root@h105 ~]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 5.6.27-76.0 Percona Server (GPL), Release 76.0, Revision 5498987

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use Syslog
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------------+
| Tables_in_Syslog       |
+------------------------+
| SystemEvents           |
| SystemEventsProperties |
+------------------------+
2 rows in set (0.00 sec)

mysql> select count(*) from SystemEvents;
+----------+
| count(*) |
+----------+
|       54 |
+----------+
1 row in set (0.00 sec)

mysql> select *  from SystemEvents limit 5\G;
*************************** 1. row ***************************
                ID: 1
        CustomerID: NULL
        ReceivedAt: 2016-05-10 21:39:29
DeviceReportedTime: 2016-05-10 21:39:29
          Facility: 0
          Priority: 6
          FromHost: h105
           Message: imklog 5.8.10, log source = /proc/kmsg started.
        NTSeverity: NULL
        Importance: NULL
       EventSource: NULL
         EventUser: NULL
     EventCategory: NULL
           EventID: NULL
   EventBinaryData: NULL
      MaxAvailable: NULL
         CurrUsage: NULL
          MinUsage: NULL
          MaxUsage: NULL
        InfoUnitID: 1
         SysLogTag: kernel:
      EventLogType: NULL
   GenericFileName: NULL
          SystemID: NULL
*************************** 2. row ***************************
                ID: 2
        CustomerID: NULL
        ReceivedAt: 2016-05-10 21:39:29
DeviceReportedTime: 2016-05-10 21:39:29
          Facility: 5
          Priority: 6
          FromHost: h105
           Message:  [origin software="rsyslogd" swVersion="5.8.10" x-pid="3230" x-info="http://www.rsyslog.com"] start
        NTSeverity: NULL
        Importance: NULL
       EventSource: NULL
         EventUser: NULL
     EventCategory: NULL
           EventID: NULL
   EventBinaryData: NULL
      MaxAvailable: NULL
         CurrUsage: NULL
          MinUsage: NULL
          MaxUsage: NULL
        InfoUnitID: 1
         SysLogTag: rsyslogd:
      EventLogType: NULL
   GenericFileName: NULL
          SystemID: NULL
*************************** 3. row ***************************
                ID: 3
        CustomerID: NULL
        ReceivedAt: 2016-05-10 21:40:01
DeviceReportedTime: 2016-05-10 21:40:01
          Facility: 9
          Priority: 6
          FromHost: h105
           Message:  (root) CMD (/usr/lib64/sa/sa1 1 1)
        NTSeverity: NULL
        Importance: NULL
       EventSource: NULL
         EventUser: NULL
     EventCategory: NULL
           EventID: NULL
   EventBinaryData: NULL
      MaxAvailable: NULL
         CurrUsage: NULL
          MinUsage: NULL
          MaxUsage: NULL
        InfoUnitID: 1
         SysLogTag: CROND[3246]:
      EventLogType: NULL
   GenericFileName: NULL
          SystemID: NULL
*************************** 4. row ***************************
                ID: 4
        CustomerID: NULL
        ReceivedAt: 2016-05-10 21:40:02
DeviceReportedTime: 2016-05-10 21:40:02
          Facility: 9
          Priority: 5
          FromHost: h105
           Message:  Job `cron.daily' started
        NTSeverity: NULL
        Importance: NULL
       EventSource: NULL
         EventUser: NULL
     EventCategory: NULL
           EventID: NULL
   EventBinaryData: NULL
      MaxAvailable: NULL
         CurrUsage: NULL
          MinUsage: NULL
          MaxUsage: NULL
        InfoUnitID: 1
         SysLogTag: anacron[2878]:
      EventLogType: NULL
   GenericFileName: NULL
          SystemID: NULL
*************************** 5. row ***************************
                ID: 5
        CustomerID: NULL
        ReceivedAt: 2016-05-10 21:40:02
DeviceReportedTime: 2016-05-10 21:40:02
          Facility: 9
          Priority: 5
          FromHost: h105
           Message:  starting cups
        NTSeverity: NULL
        Importance: NULL
       EventSource: NULL
         EventUser: NULL
     EventCategory: NULL
           EventID: NULL
   EventBinaryData: NULL
      MaxAvailable: NULL
         CurrUsage: NULL
          MinUsage: NULL
          MaxUsage: NULL
        InfoUnitID: 1
         SysLogTag: run-parts(/etc/cron.daily)[3249]:
      EventLogType: NULL
   GenericFileName: NULL
          SystemID: NULL
5 rows in set (0.00 sec)

ERROR: 
No query specified

mysql> 
mysql> select *  from SystemEvents where id=51 \G;
*************************** 1. row ***************************
                ID: 51
        CustomerID: NULL
        ReceivedAt: 2016-05-10 22:04:23
DeviceReportedTime: 2016-05-10 22:04:23
          Facility: 1
          Priority: 5
          FromHost: h202
           Message:  [euid=root]:root pts/1 2016-05-10 15:47 (192.168.100.1):[/root]grep root /etc/shadow
        NTSeverity: NULL
        Importance: NULL
       EventSource: NULL
         EventUser: NULL
     EventCategory: NULL
           EventID: NULL
   EventBinaryData: NULL
      MaxAvailable: NULL
         CurrUsage: NULL
          MinUsage: NULL
          MaxUsage: NULL
        InfoUnitID: 1
         SysLogTag: root:
      EventLogType: NULL
   GenericFileName: NULL
          SystemID: NULL
1 row in set (0.01 sec)

ERROR: 
No query specified

mysql> 
~~~

---

## 安装 LogAnalyzer

**[LogAnalyzer][loganalyzer]** 的下载地址可以参考 **[下载][loganalyzer_dl]** ，安装过程可以参考 **[安装][loganalyzer_inst]**

---

### 下载 LogAnalyzer


~~~
[root@h105 src]# wget http://download.adiscon.com/loganalyzer/loganalyzer-3.6.6.tar.gz
--2016-05-10 22:15:18--  http://download.adiscon.com/loganalyzer/loganalyzer-3.6.6.tar.gz
Resolving download.adiscon.com... 176.9.39.152
Connecting to download.adiscon.com|176.9.39.152|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1047243 (1023K) [application/x-gzip]
Saving to: “loganalyzer-3.6.6.tar.gz”

38% [=================================>                                                         ] 402,243     15.6K/s   in 2m 0s   

2016-05-10 22:17:20 (3.27 KB/s) - Connection closed at byte 402243. Retrying.

--2016-05-10 22:17:21--  (try: 2)  http://download.adiscon.com/loganalyzer/loganalyzer-3.6.6.tar.gz
Connecting to download.adiscon.com|176.9.39.152|:80... connected.
HTTP request sent, awaiting response... 206 Partial Content
Length: 1047243 (1023K), 645000 (630K) remaining [application/x-gzip]
Saving to: “loganalyzer-3.6.6.tar.gz”

100%[++++++++++++++++++++++++++++++++++========================================================>] 1,047,243   11.6K/s   in 35s     

2016-05-10 22:17:57 (17.8 KB/s) - “loganalyzer-3.6.6.tar.gz” saved [1047243/1047243]

[root@h105 src]# ls
loganalyzer-3.6.6.tar.gz  mysql
[root@h105 src]# 
~~~


### 解压与安装

~~~
[root@h105 src]# tar zxf loganalyzer-3.6.6.tar.gz 
[root@h105 src]# cd loganalyzer-3.6.6
[root@h105 loganalyzer-3.6.6]# mkdir -p /var/www/html/loganalyzer
[root@h105 loganalyzer-3.6.6]# rsync -a src/* /var/www/html/loganalyzer/
[root@h105 loganalyzer-3.6.6]#
~~~


---

## 浏览器中配置 loganalyzer

访问 **`http://192.168.100.105/loganalyzer/`**

提示缺少主配置文件

![loganalyzer1.png](/images/loganalyzer/loganalyzer1.png)

点击 【here】

进行预检查

![loganalyzer2.png](/images/loganalyzer/loganalyzer2.png)

点击 【Next】

提示创建一个配置文件

![loganalyzer3.png](/images/loganalyzer/loganalyzer3.png)

根据提示创建配置文件，并赋予指定权限

~~~
[root@h105 loganalyzer-3.6.6]# ls
ChangeLog  contrib  COPYING  doc  INSTALL  src
[root@h105 loganalyzer-3.6.6]# cat contrib/configure.sh 
#!/bin/sh

touch config.php
chmod 666 config.php
[root@h105 loganalyzer-3.6.6]# touch /var/www/html/loganalyzer/config.php
[root@h105 loganalyzer-3.6.6]# chmod 666 /var/www/html/loganalyzer/config.php 
[root@h105 loganalyzer-3.6.6]# 
~~~

点击 【ReCheck】


![loganalyzer4.png](/images/loganalyzer/loganalyzer4.png)

检查通过

点击 【Next】

![loganalyzer5.png](/images/loganalyzer/loganalyzer5.png)

保持默认配置，**Show message details popup** 有日志概要弹出效果 ，选择使用数据库 **Enable User Database** ，目前只支持 mysql，填充正确信息

![loganalyzer6.png](/images/loganalyzer/loganalyzer6.png)

点击 【Next】

数据库连接正常，并且准备创建相应表

![loganalyzer8.png](/images/loganalyzer/loganalyzer8.png)



> **Note:** 这个过程中要确保 **php-mysql** 包存在，否则无法与mysql 连接，会出现如下的界面

![loganalyzer7.png](/images/loganalyzer/loganalyzer7.png)

> **Tip:** 遇到这种情况，先检查一下 **php-mysql** ，然后重新加载环境变量，重启一下 httpd 服务

安装 **php-mysql** 的过程

~~~
[root@h105 loganalyzer-3.6.6]# yum clean all 
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository base is listed more than once in the configuration
Cleaning repos: base epel extras updates
Cleaning up Everything
Cleaning up list of fastest mirrors
[root@h105 loganalyzer-3.6.6]# yum install  php-mysql
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Determining fastest mirrors
epel/metalink                                                                                                | 4.9 kB     00:00     
 * epel: ftp.cuhk.edu.hk
 * extras: mirrors.pubyun.com
 * updates: mirrors.pubyun.com
base                                                                                                         | 4.0 kB     00:00 ... 
base/primary_db                                                                                              | 4.5 MB     00:00 ... 
epel                                                                                                         | 4.3 kB     00:00     
epel/primary_db                                                                                              | 5.9 MB     00:04     
extras                                                                                                       | 3.4 kB     00:00     
extras/primary_db                                                                                            |  37 kB     00:00     
updates                                                                                                      | 3.4 kB     00:00     
updates/primary_db                                                                                           | 5.2 MB     00:03     
Resolving Dependencies
--> Running transaction check
---> Package php-mysql.x86_64 0:5.3.3-46.el6_7.1 will be installed
--> Processing Dependency: php-pdo(x86-64) for package: php-mysql-5.3.3-46.el6_7.1.x86_64
--> Running transaction check
---> Package php-pdo.x86_64 0:5.3.3-46.el6_7.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                       Arch                       Version                                 Repository                   Size
====================================================================================================================================
Installing:
 php-mysql                     x86_64                     5.3.3-46.el6_7.1                        updates                      86 k
Installing for dependencies:
 php-pdo                       x86_64                     5.3.3-46.el6_7.1                        updates                      80 k

Transaction Summary
====================================================================================================================================
Install       2 Package(s)

Total download size: 165 k
Installed size: 384 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): php-mysql-5.3.3-46.el6_7.1.x86_64.rpm                                                                 |  86 kB     00:00     
(2/2): php-pdo-5.3.3-46.el6_7.1.x86_64.rpm                                                                   |  80 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               862 kB/s | 165 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : php-pdo-5.3.3-46.el6_7.1.x86_64                                                                                  1/2 
  Installing : php-mysql-5.3.3-46.el6_7.1.x86_64                                                                                2/2 
  Verifying  : php-mysql-5.3.3-46.el6_7.1.x86_64                                                                                1/2 
  Verifying  : php-pdo-5.3.3-46.el6_7.1.x86_64                                                                                  2/2 

Installed:
  php-mysql.x86_64 0:5.3.3-46.el6_7.1                                                                                               

Dependency Installed:
  php-pdo.x86_64 0:5.3.3-46.el6_7.1                                                                                                 

Complete!
[root@h105 loganalyzer-3.6.6]#
~~~

正常后，会出现先前界面

![loganalyzer8.png](/images/loganalyzer/loganalyzer8.png)

点击 【Next】

成功创建相关表

![loganalyzer9.png](/images/loganalyzer/loganalyzer9.png)

点击 【Next】

创建管理用户

![loganalyzer10.png](/images/loganalyzer/loganalyzer10.png)

指定管理员名，设置管理员的密码，点击 【Next】

指定数据源

![loganalyzer11.png](/images/loganalyzer/loganalyzer11.png)

修改数据源为数据库(也可以为文件，默认是文件)

![loganalyzer12.png](/images/loganalyzer/loganalyzer12.png)

指定正确的数据库连接参数

点击 【Next】

![loganalyzer13.png](/images/loganalyzer/loganalyzer13.png)

完成

点击 【Finish】

![loganalyzer14.png](/images/loganalyzer/loganalyzer14.png)


进入了首页，由于我们配置中启用了弹出信息，所以鼠标指到哪条信息上，就有一个小窗口显示日志概要，点击进一条信息可以查看详细信息

![loganalyzer15.png](/images/loganalyzer/loganalyzer15.png)

> **Tip:** 如果想修改配置可以清空 **config.php** 重来

~~~
[root@h105 loganalyzer]# pwd
/var/www/html/loganalyzer
[root@h105 loganalyzer]# > config.php 
[root@h105 loganalyzer]#
~~~

---

## 查看日志统计

点击 【Statistics】 选项卡

![loganalyzer17.png](/images/loganalyzer/loganalyzer17.png)


![loganalyzer18.png](/images/loganalyzer/loganalyzer18.png)


> **Note:** 统计的图形界面需要 **php-gd** 如果没有会产生如下空白和报错

![loganalyzer16.png](/images/loganalyzer/loganalyzer16.png)

> **Tip:** 解决办法是 安装 **php-gd** ，切换加载环境，重启httpd服务

~~~
[root@h105 ~]# yum clean all 
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository base is listed more than once in the configuration
Cleaning repos: base epel extras updates
Cleaning up Everything
Cleaning up list of fastest mirrors
[root@h105 ~]# yum install  php-gd
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Determining fastest mirrors
epel/metalink                                                                                                | 4.9 kB     00:00     
 * epel: ftp.cuhk.edu.hk
 * extras: mirrors.pubyun.com
 * updates: mirrors.pubyun.com
base                                                                                                         | 4.0 kB     00:00 ... 
base/primary_db                                                                                              | 4.5 MB     00:00 ... 
epel                                                                                                         | 4.3 kB     00:00     
epel/primary_db                                                                                              | 5.9 MB     00:04     
extras                                                                                                       | 3.4 kB     00:00     
extras/primary_db                                                                                            |  37 kB     00:00     
updates                                                                                                      | 3.4 kB     00:00     
updates/primary_db                                                                                           | 5.2 MB     00:03     
Resolving Dependencies
--> Running transaction check
---> Package php-gd.x86_64 0:5.3.3-46.el6_7.1 will be installed
--> Processing Dependency: libXpm.so.4()(64bit) for package: php-gd-5.3.3-46.el6_7.1.x86_64
--> Running transaction check
---> Package libXpm.x86_64 0:3.5.10-2.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                     Arch                        Version                                 Repository                    Size
====================================================================================================================================
Installing:
 php-gd                      x86_64                      5.3.3-46.el6_7.1                        updates                      111 k
Installing for dependencies:
 libXpm                      x86_64                      3.5.10-2.el6                            base                          51 k

Transaction Summary
====================================================================================================================================
Install       2 Package(s)

Total download size: 162 k
Installed size: 426 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): php-gd-5.3.3-46.el6_7.1.x86_64.rpm                                                                    | 111 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               528 kB/s | 162 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : libXpm-3.5.10-2.el6.x86_64                                                                                       1/2 
  Installing : php-gd-5.3.3-46.el6_7.1.x86_64                                                                                   2/2 
  Verifying  : libXpm-3.5.10-2.el6.x86_64                                                                                       1/2 
  Verifying  : php-gd-5.3.3-46.el6_7.1.x86_64                                                                                   2/2 

Installed:
  php-gd.x86_64 0:5.3.3-46.el6_7.1                                                                                                  

Dependency Installed:
  libXpm.x86_64 0:3.5.10-2.el6                                                                                                      

Complete!
[root@h105 ~]# 
[root@h105 ~]# exit
logout
[root@h105 ~]# su - root 
[root@h105 ~]# /etc/init.d/httpd restart 
Stopping httpd:                                            [  OK  ]
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.100.105 for ServerName
                                                           [  OK  ]
[root@h105 ~]#
~~~

---



## loganalyzer 登录测试

![loganalyzer19.png](/images/loganalyzer/loganalyzer19.png)


---

## 搜索日志

点击 【Search】 选项卡



可以根据时间范围和日志级别进行检索

![loganalyzer20.png](/images/loganalyzer/loganalyzer20.png)

---

## 修改配置

点击 【Admin Center】 选项卡

![loganalyzer21.png](/images/loganalyzer/loganalyzer21.png)

---

# 命令汇总

* **`yum install httpd -y`**
* **`/etc/init.d/httpd start`**
* **`chkconfig httpd on`**
* **`chkconfig httpd --list`**
* **`vim /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv  | grep 80`**
* **`rpm -ivh Percona-Server-*`**
* **`/etc/init.d/mysql  start`**
* **`mysqladmin -uroot password 'mysql'`**
* **`yum install php`**
* **`cd /var/www/html/`**
* **`cat >index.php  <<EOF`**
* **`cat index.php`**
* **`setenforce 0`**
* **`getenforce`**
* **`su - root`**
* **`rpm -qa | grep rsyslog`**
* **`yum -y install rsyslog-mysql`**
* **`cd /usr/share/doc/rsyslog-mysql-5.8.10/`**
* **`wc -l createDB.sql`**
* **`cat createDB.sql`**
* **`mysql -u root -pmysql < createDB.sql`**
* **`vim /etc/rsyslog.conf`**
* **`grep -v "^#" /etc/rsyslog.conf | grep -v "^$"`**
* **`/etc/init.d/rsyslog restart`**
* **`netstat  -an | grep 514`**
* **`vim /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv | grep 514`**
* **`vim /etc/rsyslog.conf`**
* **`grep -v "^#" /etc/rsyslog.conf | grep -v "^$"`**
* **`/etc/init.d/rsyslog restart`**
* **`vim /etc/bashrc`**
* **`tail -n 3 /etc/bashrc`**
* **`source /etc/bashrc`**
* **`tailf /var/log/messages`**
* **`wget http://download.adiscon.com/loganalyzer/loganalyzer-3.6.6.tar.gz`**
* **`tar zxf loganalyzer-3.6.6.tar.gz`**
* **`cd loganalyzer-3.6.6`**
* **`mkdir -p /var/www/html/loganalyzer`**
* **`rsync -a src/* /var/www/html/loganalyzer/`**
* **`cat contrib/configure.sh`**
* **`touch /var/www/html/loganalyzer/config.php`**
* **`chmod 666 /var/www/html/loganalyzer/config.php`**
* **`yum clean all`**
* **`yum install  php-mysql`**
* **`> config.php`**
* **`yum clean all`**
* **`yum install  php-gd`**
* **`/etc/init.d/httpd restart`**

---


[loganalyzer]:http://loganalyzer.adiscon.com/
[loganalyzer_dl]:http://loganalyzer.adiscon.com/downloads/
[ref_doc]:http://www.cnblogs.com/mchina/p/linux-centos-rsyslog-loganalyzer-mysql-log-server.html
[loganalyzer_doc]:http://loganalyzer.adiscon.com/doc/manual.html
[loganalyzer_inst]:http://loganalyzer.adiscon.com/doc/install.html
[elk]:http://soft.dog/2015/12/22/elk-basic/
