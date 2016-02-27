---
layout: post
title: 使用XtraBackup创建mysql slave 
author: wilmosfang
categories: linux  mysql admintools
wc: 1206 4769 52759
excerpt: follow me
comments: true
---

---

# 前言

**[XtraBackup][percona-xtrabackup]** 是 **[percona][percona]** 出的一款mysql备份工具，可以使用它对mysql进行高效备份

下面分享一下使用 **[XtraBackup][percona-xtrabackup]** 创建mysql slave的基础操作，详细可以参阅 [官方文档][xtrabackupdoc]

> **Tip:** 当前版本 **Percona XtraBackup 2.2**

---

# 概要

* TOC
{:toc}




---

## 准备slave软件环境

### 下载安装percona repo

{% highlight bash %}
[root@slave-test src]# wget  http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm 
--2015-10-12 14:03:35--  http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm
Resolving www.percona.com... 74.121.199.234
Connecting to www.percona.com|74.121.199.234|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm [following]
--2015-10-12 14:03:36--  https://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm
Connecting to www.percona.com|74.121.199.234|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6566 (6.4K) [application/x-redhat-package-manager]
Saving to: “percona-release-0.1-3.noarch.rpm”

100%[===================================================================================================================>] 6,566    

2015-10-12 14:03:37 (115 MB/s) - “percona-release-0.1-3.noarch.rpm” saved [6566/6566]

[root@slave-test src]# ls
percona-release-0.1-3.noarch.rpm
[root@slave-test src]# rpm -ivh percona-release-0.1-3.noarch.rpm 
warning: percona-release-0.1-3.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:percona-release        ########################################### [100%]
[root@slave-test src]# 
{% endhighlight %}

可以在系统中进行一下检查

{% highlight bash %}
[root@slave-test src]# rpm -qlp percona-release-0.1-3.noarch.rpm 
warning: percona-release-0.1-3.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
/etc/pki/rpm-gpg/RPM-GPG-KEY-Percona
/etc/yum.repos.d/percona-release.repo
/usr/share/doc/percona-release-0.1
/usr/share/doc/percona-release-0.1/RPM-GPG-KEY-Percona
[root@slave-test src]# ll /etc/yum.repos.d/percona-release.repo 
-rw-r--r-- 1 root root 2501 Sep 22  2014 /etc/yum.repos.d/percona-release.repo
[root@slave-test src]# 
{% endhighlight %}

---

### 安装percona-xtrabackup

{% highlight bash %}
[root@slave-test src]# yum -y install percona-xtrabackup.x86_64    
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: mirrors.pubyun.com
 * extras: mirrors.pubyun.com
 * updates: mirrors.pubyun.com
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package percona-xtrabackup.x86_64 0:2.2.12-1.el6 will be installed
--> Processing Dependency: perl(DBD::mysql) for package: percona-xtrabackup-2.2.12-1.el6.x86_64
--> Running transaction check
---> Package perl-DBD-MySQL.x86_64 0:4.013-3.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                           Arch                  Version                        Repository                             Size
====================================================================================================================================
Installing:
 percona-xtrabackup                x86_64                2.2.12-1.el6                   percona-release-x86_64                4.8 M
Installing for dependencies:
 perl-DBD-MySQL                    x86_64                4.013-3.el6                    base                                  134 k

Transaction Summary
====================================================================================================================================
Install       2 Package(s)

Total download size: 5.0 M
Installed size: 19 M
Downloading Packages:
(1/2): percona-xtrabackup-2.2.12-1.el6.x86_64.rpm                                                            | 4.8 MB     00:51     
(2/2): perl-DBD-MySQL-4.013-3.el6.x86_64.rpm                                                                 | 134 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                96 kB/s | 5.0 MB     00:52     
warning: rpmts_HdrFromFdno: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Percona
Importing GPG key 0xCD2EFD2A:
 Userid : Percona MySQL Development Team <mysql-dev@percona.com>
 Package: percona-release-0.1-3.noarch (installed)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-Percona
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : perl-DBD-MySQL-4.013-3.el6.x86_64                                                                                1/2 
  Installing : percona-xtrabackup-2.2.12-1.el6.x86_64                                                                           2/2 
  Verifying  : perl-DBD-MySQL-4.013-3.el6.x86_64                                                                                1/2 
  Verifying  : percona-xtrabackup-2.2.12-1.el6.x86_64                                                                           2/2 

Installed:
  percona-xtrabackup.x86_64 0:2.2.12-1.el6                                                                                          

Dependency Installed:
  perl-DBD-MySQL.x86_64 0:4.013-3.el6                                                                                               

Complete!
[root@slave-test src]# 
{% endhighlight %}


---

### 安装mysql

{% highlight bash %}
[root@slave-test src]# yum install Percona-Server-server-56
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: mirrors.pubyun.com
 * extras: mirrors.pubyun.com
 * updates: mirrors.pubyun.com
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package Percona-Server-server-56.x86_64 0:5.6.26-rel74.0.el6 will be installed
--> Processing Dependency: Percona-Server-client-56 for package: Percona-Server-server-56-5.6.26-rel74.0.el6.x86_64
--> Processing Dependency: Percona-Server-shared-56 for package: Percona-Server-server-56-5.6.26-rel74.0.el6.x86_64
--> Running transaction check
---> Package Percona-Server-client-56.x86_64 0:5.6.26-rel74.0.el6 will be installed
---> Package Percona-Server-shared-56.x86_64 0:5.6.26-rel74.0.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                               Arch                Version                          Repository                           Size
======================================================================================================================================
Installing:
 Percona-Server-server-56              x86_64              5.6.26-rel74.0.el6               percona-release-x86_64               20 M
Installing for dependencies:
 Percona-Server-client-56              x86_64              5.6.26-rel74.0.el6               percona-release-x86_64              6.4 M
 Percona-Server-shared-56              x86_64              5.6.26-rel74.0.el6               percona-release-x86_64              725 k

Transaction Summary
======================================================================================================================================
Install       3 Package(s)

Total download size: 27 M
Installed size: 123 M
Is this ok [y/N]: y
Downloading Packages:
(1/3): Percona-Server-client-56-5.6.26-rel74.0.el6.x86_64.rpm                                                  | 6.4 MB     01:19     
(2/3): Percona-Server-server-56-5.6.26-rel74.0.el6.x86_64.rpm                                                  |  20 MB     03:44     
(3/3): Percona-Server-shared-56-5.6.26-rel74.0.el6.x86_64.rpm                                                  | 725 kB     00:02     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                  89 kB/s |  27 MB     05:07     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : Percona-Server-shared-56-5.6.26-rel74.0.el6.x86_64                                                                 1/3 
  Installing : Percona-Server-client-56-5.6.26-rel74.0.el6.x86_64                                                                 2/3 
  Installing : Percona-Server-server-56-5.6.26-rel74.0.el6.x86_64                                                                 3/3 
2015-10-12 14:51:21 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-10-12 14:51:21 0 [Note] /usr/sbin/mysqld (mysqld 5.6.26-74.0) starting as process 720 ...
2015-10-12 14:51:21 720 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-10-12 14:51:21 720 [Note] InnoDB: The InnoDB memory heap is disabled
2015-10-12 14:51:21 720 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-10-12 14:51:21 720 [Note] InnoDB: Memory barrier is not used
2015-10-12 14:51:21 720 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-10-12 14:51:21 720 [Note] InnoDB: Using Linux native AIO
2015-10-12 14:51:21 720 [Note] InnoDB: Using CPU crc32 instructions
2015-10-12 14:51:21 720 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2015-10-12 14:51:21 720 [Note] InnoDB: Completed initialization of buffer pool
2015-10-12 14:51:21 720 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2015-10-12 14:51:21 720 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2015-10-12 14:51:21 720 [Note] InnoDB: Database physically writes the file full: wait...
2015-10-12 14:51:21 720 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2015-10-12 14:51:22 720 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2015-10-12 14:51:22 720 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2015-10-12 14:51:22 720 [Warning] InnoDB: New log files created, LSN=45781
2015-10-12 14:51:22 720 [Note] InnoDB: Doublewrite buffer not found: creating new
2015-10-12 14:51:22 720 [Note] InnoDB: Doublewrite buffer created
2015-10-12 14:51:22 720 [Note] InnoDB: 128 rollback segment(s) are active.
2015-10-12 14:51:22 720 [Warning] InnoDB: Creating foreign key constraint system tables.
2015-10-12 14:51:22 720 [Note] InnoDB: Foreign key constraint system tables created
2015-10-12 14:51:22 720 [Note] InnoDB: Creating tablespace and datafile system tables.
2015-10-12 14:51:22 720 [Note] InnoDB: Tablespace and datafile system tables created.
2015-10-12 14:51:22 720 [Note] InnoDB: Waiting for purge to start
2015-10-12 14:51:22 720 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.26-74.0 started; log sequence number 0
2015-10-12 14:51:22 720 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
2015-10-12 14:51:22 720 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
2015-10-12 14:51:26 720 [Note] Binlog end
2015-10-12 14:51:26 720 [Note] InnoDB: FTS optimize thread exiting.
2015-10-12 14:51:26 720 [Note] InnoDB: Starting shutdown...
2015-10-12 14:51:28 720 [Note] InnoDB: Shutdown completed; log sequence number 1625977


2015-10-12 14:51:28 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-10-12 14:51:28 0 [Note] /usr/sbin/mysqld (mysqld 5.6.26-74.0) starting as process 744 ...
2015-10-12 14:51:28 744 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-10-12 14:51:28 744 [Note] InnoDB: The InnoDB memory heap is disabled
2015-10-12 14:51:28 744 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-10-12 14:51:28 744 [Note] InnoDB: Memory barrier is not used
2015-10-12 14:51:28 744 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-10-12 14:51:28 744 [Note] InnoDB: Using Linux native AIO
2015-10-12 14:51:28 744 [Note] InnoDB: Using CPU crc32 instructions
2015-10-12 14:51:28 744 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2015-10-12 14:51:28 744 [Note] InnoDB: Completed initialization of buffer pool
2015-10-12 14:51:28 744 [Note] InnoDB: Highest supported file format is Barracuda.
2015-10-12 14:51:28 744 [Note] InnoDB: 128 rollback segment(s) are active.
2015-10-12 14:51:28 744 [Note] InnoDB: Waiting for purge to start
2015-10-12 14:51:28 744 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.26-74.0 started; log sequence number 1625977
2015-10-12 14:51:28 744 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
2015-10-12 14:51:28 744 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
2015-10-12 14:51:28 744 [Note] Binlog end
2015-10-12 14:51:28 744 [Note] InnoDB: FTS optimize thread exiting.
2015-10-12 14:51:28 744 [Note] InnoDB: Starting shutdown...
2015-10-12 14:51:30 744 [Note] InnoDB: Shutdown completed; log sequence number 1625987




PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  /usr/bin/mysqladmin -u root password 'new-password'
  /usr/bin/mysqladmin -u root -h slave-test password 'new-password'

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

WARNING: Default config file /etc/my.cnf exists on the system
This file will be read by default by the MySQL server
If you do not want to use this, either remove it, or use the
--defaults-file argument to mysqld_safe when starting the server

Percona Server is distributed with several useful UDF (User Defined Function) from Percona Toolkit.
Run the following commands to create these functions:
mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"
mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"
mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"
See http://www.percona.com/doc/percona-server/5.6/management/udf_percona_toolkit.html for more details
  Verifying  : Percona-Server-shared-56-5.6.26-rel74.0.el6.x86_64                                                                 1/3 
  Verifying  : Percona-Server-server-56-5.6.26-rel74.0.el6.x86_64                                                                 2/3 
  Verifying  : Percona-Server-client-56-5.6.26-rel74.0.el6.x86_64                                                                 3/3 

Installed:
  Percona-Server-server-56.x86_64 0:5.6.26-rel74.0.el6                                                                                

Dependency Installed:
  Percona-Server-client-56.x86_64 0:5.6.26-rel74.0.el6              Percona-Server-shared-56.x86_64 0:5.6.26-rel74.0.el6             

Complete!
[root@slave-test src]# echo $?
0
[root@slave-test src]# 
{% endhighlight %}

启动mysql，设定密码

{% highlight bash %}
[root@slave-test src]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server).                           [  OK  ]
[root@slave-test src]#  /usr/bin/mysqladmin -u root password 'xxxxx'
Warning: Using a password on the command line interface can be insecure.
[root@slave-test src]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.26-74.0 Percona Server (GPL), Release 74.0, Revision 32f8dfd

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql> 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

mysql> 
{% endhighlight %}

到此salve的软件环境就已经准备好了

---

### 注意事项

* 1.slave上的数据存储位置有足够的空间，如果没有最好链接到一个有空间的位置
* 2.slave上使用master的配置文件，可以将有些大内存使用参数酌情改小
* 3.注意修改 **server-id** ,不能和master一样


---

## 备份master数据库

使用前面的方法在master上安装xtrabackup

### 报错1


{% highlight bash %}
[root@master-qa ~]# /usr/bin/innobackupex --defaults-file=/etc/my.cnf --user=root --password=xxxxxxxx /data/fullbackup/ 

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

Get the latest version of Percona XtraBackup, documentation, and help resources:
http://www.percona.com/xb/p

151012 15:22:14  innobackupex: Executing a version check against the server...
151012 15:22:14  innobackupex: Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_file=/etc/my.cnf;mysql_read_default_group=xtrabackup' as 'root'  (using password: YES).
innobackupex: got a fatal error with the following stacktrace: at /usr/bin/innobackupex line 3006
	main::mysql_connect('abort_on_error', 1) called at /usr/bin/innobackupex line 1551
innobackupex: Error: Failed to connect to MySQL server as DBD::mysql module is not installed at /usr/bin/innobackupex line 3006.
151012 15:22:14  innobackupex: Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_file=/etc/my.cnf;mysql_read_default_group=xtrabackup' as 'root'  (using password: YES).
innobackupex: got a fatal error with the following stacktrace: at /usr/bin/innobackupex line 3006
	main::mysql_connect('abort_on_error', 1) called at /usr/bin/innobackupex line 1570
innobackupex: Error: Failed to connect to MySQL server as DBD::mysql module is not installed at /usr/bin/innobackupex line 3006.
[root@master-qa ~]# rpm -qa | grep -i dbd 
perl-DBD-SQLite-1.27-3.el6.x86_64
perl-DBD-MySQL-4.013-3.el6.x86_64
[root@master-qa ~]#
{% endhighlight %}

可知 **perl-DBD-MySQL** 已经安装了，和报错不符合


解决办法：

重装 **perl-DBD-MySQL**

{% highlight bash %}
[root@master-qa ~]# yum list all | grep  perl-DBD-MySQL 
perl-DBD-MySQL.x86_64                      4.013-3.el6                   @base  
[root@master-qa ~]# yum reinstall  perl-DBD-MySQL.x86_64  
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Reinstall Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * extras: mirrors.pubyun.com
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package perl-DBD-MySQL.x86_64 0:4.013-3.el6 will be reinstalled
--> Processing Dependency: libmysqlclient.so.16(libmysqlclient_16)(64bit) for package: perl-DBD-MySQL-4.013-3.el6.x86_64
--> Processing Dependency: libmysqlclient.so.16()(64bit) for package: perl-DBD-MySQL-4.013-3.el6.x86_64
--> Running transaction check
---> Package Percona-Server-shared-51.x86_64 0:5.1.73-rel14.12.624.rhel6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                             Arch              Version                                Repository                         Size
======================================================================================================================================
Reinstalling:
 perl-DBD-MySQL                      x86_64            4.013-3.el6                            base                              134 k
Installing for dependencies:
 Percona-Server-shared-51            x86_64            5.1.73-rel14.12.624.rhel6              percona-release-x86_64            2.1 M

Transaction Summary
======================================================================================================================================
Install       1 Package(s)
Reinstall     1 Package(s)

Total download size: 2.3 M
Installed size: 6.2 M
Is this ok [y/N]: y
Downloading Packages:
(1/2): Percona-Server-shared-51-5.1.73-rel14.12.6 (1%)  1% [-                                       ]  0.0 B/s |  43 kB     --:-- ETA 
(1/2): Percona-Server-shared-51-5.1.73-rel14.12.6 (91%) 96% [=====================================- ] 177 kB/s | 2.1 MB     00:00 ETA 
(1/2): Percona-Server-shared-51-5.1.73-rel14.12.6 (93%) 99% [======================================-] 174 kB/s | 2.1 MB     00:00 ETA 

(1/2): Percona-Server-shared-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm                                           | 2.1 MB     00:10     

(2/2): perl-DBD-MySQL-4.013-3.el6.x86_64.rpm                                                                   | 134 kB     00:00     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 208 kB/s | 2.3 MB     00:11     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction

  Installing : Percona-Server-shared-51-5.1.73-rel14.12.624.rhel6.x86_ [####################################################### ] 1/2
  Installing : Percona-Server-shared-51-5.1.73-rel14.12.624.rhel6.x86_64                                                          1/2 
  
  Installing : perl-DBD-MySQL-4.013-3.el6.x86_64 [############################################################################# ] 2/2
  Installing : perl-DBD-MySQL-4.013-3.el6.x86_64                                                                                  2/2 

  Verifying  : perl-DBD-MySQL-4.013-3.el6.x86_64                                                                                  1/2 

  Verifying  : Percona-Server-shared-51-5.1.73-rel14.12.624.rhel6.x86_64                                                          2/2 

Installed:
  perl-DBD-MySQL.x86_64 0:4.013-3.el6                                                                                                 

Dependency Installed:
  Percona-Server-shared-51.x86_64 0:5.1.73-rel14.12.624.rhel6                                                                         

Complete!
[root@master-qa ~]# 
{% endhighlight %}

再次尝试备份master数据库

{% highlight bash %}
[root@master-qa ~]#  /usr/bin/innobackupex --defaults-file=/etc/my.cnf --user=root --password=xxxxxxx /data/fullbackup/ 

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

Get the latest version of Percona XtraBackup, documentation, and help resources:
http://www.percona.com/xb/p

151012 15:24:06  innobackupex: Executing a version check against the server...
151012 15:24:06  innobackupex: Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_file=/etc/my.cnf;mysql_read_default_group=xtrabackup' as 'root'  (using password: YES).
151012 15:24:06  innobackupex: Connected to MySQL server
151012 15:24:06  innobackupex: Done.
151012 15:24:06  innobackupex: Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_file=/etc/my.cnf;mysql_read_default_group=xtrabackup' as 'root'  (using password: YES).
151012 15:24:06  innobackupex: Connected to MySQL server
151012 15:24:06  innobackupex: Starting the backup operation

IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

innobackupex:  Using server version 5.6.25-73.0-log

innobackupex: Created backup directory /data/fullbackup/2015-10-12_15-24-06

151012 15:24:06  innobackupex: Starting ibbackup with command: xtrabackup  --defaults-file="/etc/my.cnf"  --defaults-group="mysqld" --backup --suspend-at-end --target-dir=/data/fullbackup/2015-10-12_15-24-06 --innodb_data_file_path="ibdata1:12M:autoextend" --tmpdir=/var/tmp --extra-lsndir='/var/tmp'
innobackupex: Waiting for ibbackup (pid=3577) to suspend
innobackupex: Suspend file '/data/fullbackup/2015-10-12_15-24-06/xtrabackup_suspended_2'

xtrabackup version 2.2.12 based on MySQL server 5.6.24 Linux (x86_64) (revision id: 8726828)
xtrabackup: uses posix_fadvise().
xtrabackup: cd to /var/lib/mysql
xtrabackup: open files limit requested 0, set to 65536
xtrabackup: using the following InnoDB configuration:
xtrabackup:   innodb_data_home_dir = /var/lib/mysql/
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 2
xtrabackup:   innodb_log_file_size = 134217728
xtrabackup: using O_DIRECT
>> log scanned up to (76232231155)
xtrabackup: Generating a list of tablespaces
[01] Copying /var/lib/mysql/ibdata1 to /data/fullbackup/2015-10-12_15-24-06/ibdata1
>> log scanned up to (76232231155)
>> log scanned up to (76232231155)
>> log scanned up to (76232231155)
>> log scanned up to (76232231155)
>> log scanned up to (76232231155)
>> log scanned up to (76232231155)
...
...
>> log scanned up to (76232238365)
>> log scanned up to (76232238365)
>> log scanned up to (76232238365)
>> log scanned up to (76232238365)
[01]        ...done
[01] Copying ./food_qa/topic_pages.ibd to /data/fullbackup/2015-10-12_15-24-06/food_qa/topic_pages.ibd
[01]        ...done
>> log scanned up to (76232238365)
[01] Copying ./food_qa/food_compares.ibd to /data/fullbackup/2015-10-12_15-24-06/food_qa/food_compares.ibd
[01]        ...done
...
...
>> log scanned up to (76232242511)
151012 15:31:21  innobackupex: Finished backing up non-InnoDB tables and files

151012 15:31:21  innobackupex: Executing LOCK BINLOG FOR BACKUP...
151012 15:31:21  innobackupex: Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...
151012 15:31:21  innobackupex: Waiting for log copying to finish

xtrabackup: The latest check point (for incremental): '76232242511'
xtrabackup: Stopping log copying thread.
.>> log scanned up to (76232242511)

xtrabackup: Creating suspend file '/data/fullbackup/2015-10-12_15-24-06/xtrabackup_log_copied' with pid '3577'
xtrabackup: Transaction log of lsn (76232231155) to (76232242511) was copied.
151012 15:31:22  innobackupex: Executing UNLOCK BINLOG
151012 15:31:22  innobackupex: Executing UNLOCK TABLES
151012 15:31:22  innobackupex: All tables unlocked

innobackupex: Backup created in directory '/data/fullbackup/2015-10-12_15-24-06'
innobackupex: MySQL binlog position: filename 'mysql-bin.000009', position 1509223
151012 15:31:22  innobackupex: Connection to database server closed
151012 15:31:22  innobackupex: completed OK!
[root@master-qa ~]# echo $?
0
[root@master-qa ~]# 
{% endhighlight %}

---

## 拷贝备份数据到slave

{% highlight bash %}
[root@master-qa data]# rsync  -av  fullbackup/  root@192.168.1.45:/data/fullbackup/   
The authenticity of host '192.168.1.45 (192.168.1.45)' can't be established.
RSA key fingerprint is bf:ad:20:64:d2:9e:7d:25:a7:bd:8d:7c:a5:de:04:fc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.45' (RSA) to the list of known hosts.
root@192.168.1.45's password: 
sending incremental file list
./
2015-10-12_15-24-06/
2015-10-12_15-24-06/backup-my.cnf
2015-10-12_15-24-06/ibdata1


2015-10-12_15-24-06/xtrabackup_binlog_info
2015-10-12_15-24-06/xtrabackup_checkpoints
2015-10-12_15-24-06/xtrabackup_info
2015-10-12_15-24-06/xtrabackup_logfile
2015-10-12_15-24-06/active_qa/
2015-10-12_15-24-06/active_qa/db.opt
2015-10-12_15-24-06/active_qa/delivered_emails.frm
2015-10-12_15-24-06/active_qa/delivered_emails.ibd
2015-10-12_15-24-06/active_qa/prepared_emails.frm
2015-10-12_15-24-06/active_qa/prepared_emails.ibd
2015-10-12_15-24-06/active_qa/schema_migrations.frm
2015-10-12_15-24-06/active_qa/schema_migrations.ibd
2015-10-12_15-24-06/active_qa/tanita_walkers.frm
2015-10-12_15-24-06/active_qa/tanita_walkers.ibd
2015-10-12_15-24-06/active_qa/tanita_walks.frm
2015-10-12_15-24-06/active_qa/tanita_walks.ibd
2015-10-12_15-24-06/active_qa/temp_prepared_emails.frm
2015-10-12_15-24-06/active_qa/temp_prepared_emails.ibd
2015-10-12_15-24-06/active_qa/unsubscribings.frm
2015-10-12_15-24-06/active_qa/unsubscribings.ibd
2015-10-12_15-24-06/aidiet_qa/
...
...
2015-10-12_15-24-06/zabbix/user_history.ibd
2015-10-12_15-24-06/zabbix/users.frm
2015-10-12_15-24-06/zabbix/users.ibd
2015-10-12_15-24-06/zabbix/users_groups.frm
2015-10-12_15-24-06/zabbix/users_groups.ibd
2015-10-12_15-24-06/zabbix/usrgrp.frm
2015-10-12_15-24-06/zabbix/usrgrp.ibd
2015-10-12_15-24-06/zabbix/valuemaps.frm
2015-10-12_15-24-06/zabbix/valuemaps.ibd

sent 16408401594 bytes  received 62857 bytes  11633083.62 bytes/sec
total size is 16406176319  speedup is 1.00
[root@master-qa data]# 
{% endhighlight %}



---

## 恢复数据库


备份目录里有几个文件,里面有一些重要信息

恢复之前，我习惯将它们进行备份


{% highlight bash %}
[root@slave-test 2015-10-12_15-24-06]# file xtrabackup_*
xtrabackup_binlog_info: ASCII text
xtrabackup_checkpoints: ASCII text
xtrabackup_info:        ASCII text
xtrabackup_logfile:     data
[root@slave-test 2015-10-12_15-24-06]# cat xtrabackup_binlog_info
mysql-bin.000009	1509223
[root@slave-test 2015-10-12_15-24-06]# cat xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 76232242511
last_lsn = 76232242511
compact = 0
[root@slave-test 2015-10-12_15-24-06]# cat xtrabackup_info
uuid = 3cf59860-70b3-11e5-95db-0024213a7622
name = 
tool_name = innobackupex
tool_command = --defaults-file=/etc/my.cnf --user=root --password=... /data/fullbackup/
tool_version = 1.5.1-xtrabackup
ibbackup_version = xtrabackup version 2.2.12 based on MySQL server 5.6.24 Linux (x86_64) (revision id: 8726828)
server_version = 5.6.25-73.0-log
start_time = 2015-10-12 15:24:06
end_time = 2015-10-12 15:31:22
lock_time = 12
binlog_pos = filename 'mysql-bin.000009', position 1509223
innodb_from_lsn = 0
innodb_to_lsn = 76232242511
partial = N
incremental = N
format = file
compact = N
compressed = N
encrypted = N
[root@slave-test 2015-10-12_15-24-06]#
{% endhighlight %}

---

### 准备数据


{% highlight bash %}
[root@slave-test ~]# /usr/bin/innobackupex  --apply-log  /data/fullbackup/2015-10-12_15-24-06/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

Get the latest version of Percona XtraBackup, documentation, and help resources:
http://www.percona.com/xb/p

151012 20:23:21  innobackupex: Starting the apply-log operation

IMPORTANT: Please check that the apply-log run completes successfully.
           At the end of a successful apply-log run innobackupex
           prints "completed OK!".


151012 20:23:22  innobackupex: Starting ibbackup with command: xtrabackup  --defaults-file="/data/fullbackup/2015-10-12_15-24-06/backup-my.cnf"  --defaults-group="mysqld" --prepare --target-dir=/data/fullbackup/2015-10-12_15-24-06

xtrabackup version 2.2.12 based on MySQL server 5.6.24 Linux (x86_64) (revision id: 8726828)
xtrabackup: cd to /data/fullbackup/2015-10-12_15-24-06
xtrabackup: This target seems to be not prepared yet.
xtrabackup: xtrabackup_logfile detected: size=2097152, start_lsn=(76232231155)
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = ./
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 2097152
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = ./
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 2097152
xtrabackup: Starting InnoDB instance for recovery.
xtrabackup: Using 104857600 bytes for buffer pool (set by --use-memory parameter)
InnoDB: Using atomics to ref count buffer pool pages
InnoDB: The InnoDB memory heap is disabled
InnoDB: Mutexes and rw_locks use GCC atomic builtins
InnoDB: Memory barrier is not used
InnoDB: Compressed tables use zlib 1.2.3
InnoDB: Using CPU crc32 instructions
InnoDB: Initializing buffer pool, size = 100.0M
InnoDB: Completed initialization of buffer pool
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 76232231155
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: Reading tablespace information from the .ibd files...
InnoDB: Restoring possible half-written data pages 
InnoDB: from the doublewrite buffer...
InnoDB: Doing recovery: scanned up to log sequence number 76232242511 (0%)
InnoDB: Starting an apply batch of log records to the database...
InnoDB: Progress in percent: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 
InnoDB: Apply batch completed
InnoDB: In a MySQL replication slave the last master binlog file
InnoDB: position 0 221675975, file name mysql-bin.000238
InnoDB: Last MySQL binlog file position 0 1509223, file name mysql-bin.000009
InnoDB: 128 rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.6.24 started; log sequence number 76232242511

[notice (again)]
  If you use binary log and don't use any hack of group commit,
  the binary log position seems to be:
InnoDB: Last MySQL binlog file position 0 1509223, file name mysql-bin.000009

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 76232248223

151012 20:23:25  innobackupex: Restarting xtrabackup with command: xtrabackup  --defaults-file="/data/fullbackup/2015-10-12_15-24-06/backup-my.cnf"  --defaults-group="mysqld" --prepare --target-dir=/data/fullbackup/2015-10-12_15-24-06
for creating ib_logfile*

xtrabackup version 2.2.12 based on MySQL server 5.6.24 Linux (x86_64) (revision id: 8726828)
xtrabackup: cd to /data/fullbackup/2015-10-12_15-24-06
xtrabackup: This target seems to be already prepared.
xtrabackup: notice: xtrabackup_logfile was already used to '--prepare'.
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = ./
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 2
xtrabackup:   innodb_log_file_size = 134217728
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = ./
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 2
xtrabackup:   innodb_log_file_size = 134217728
xtrabackup: Starting InnoDB instance for recovery.
xtrabackup: Using 104857600 bytes for buffer pool (set by --use-memory parameter)
InnoDB: Using atomics to ref count buffer pool pages
InnoDB: The InnoDB memory heap is disabled
InnoDB: Mutexes and rw_locks use GCC atomic builtins
InnoDB: Memory barrier is not used
InnoDB: Compressed tables use zlib 1.2.3
InnoDB: Using CPU crc32 instructions
InnoDB: Initializing buffer pool, size = 100.0M
InnoDB: Completed initialization of buffer pool
InnoDB: Setting log file ./ib_logfile101 size to 128 MB
InnoDB: Progress in MB: 100
InnoDB: Setting log file ./ib_logfile1 size to 128 MB
InnoDB: Progress in MB: 100
InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
InnoDB: New log files created, LSN=76232248223
InnoDB: Highest supported file format is Barracuda.
InnoDB: 128 rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.6.24 started; log sequence number 76232248332

[notice (again)]
  If you use binary log and don't use any hack of group commit,
  the binary log position seems to be:
InnoDB: Last MySQL binlog file position 0 1509223, file name mysql-bin.000009

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 76232248582
151012 20:23:29  innobackupex: completed OK!
[root@slave-test ~]# echo $?
0
[root@slave-test ~]# 
{% endhighlight %}


---


### 恢复数据库


准备完数据后，会多出来一个文件 

{% highlight bash %}
[root@slave-test 2015-10-12_15-24-06]# ll /tmp/xtrabackup_*
-rw-r--r-- 1 root root    25 Oct 12 20:02 /tmp/xtrabackup_binlog_info
-rw-r----- 1 root root    97 Oct 12 20:02 /tmp/xtrabackup_checkpoints
-rw-r--r-- 1 root root   610 Oct 12 20:02 /tmp/xtrabackup_info
-rw-r----- 1 root root 13824 Oct 12 20:02 /tmp/xtrabackup_logfile
[root@slave-test 2015-10-12_15-24-06]# ll xtrabackup_*
-rw-r--r-- 1 root root      25 Oct 12 15:31 xtrabackup_binlog_info
-rw-r--r-- 1 root root      25 Oct 12 20:23 xtrabackup_binlog_pos_innodb
-rw-r----- 1 root root      97 Oct 12 20:23 xtrabackup_checkpoints
-rw-r--r-- 1 root root     610 Oct 12 15:31 xtrabackup_info
-rw-r----- 1 root root 2097152 Oct 12 20:23 xtrabackup_logfile
[root@slave-test 2015-10-12_15-24-06]# cat  /tmp/xtrabackup_binlog_info
mysql-bin.000009	1509223
[root@slave-test 2015-10-12_15-24-06]# cat xtrabackup_binlog_info 
mysql-bin.000009	1509223
[root@slave-test 2015-10-12_15-24-06]# cat xtrabackup_binlog_pos_innodb 
mysql-bin.000009	1509223
[root@slave-test 2015-10-12_15-24-06]# 
{% endhighlight %}

在合适的位置创建一个空文件夹，用来存放数据文件

{% highlight bash %}
[root@slave-test lib]# mv mysql/ mysql.old
[root@slave-test lib]# ln -s /data/mysql/  /var/lib/mysql
[root@slave-test lib]# cd /var/lib/mysql
[root@slave-test mysql]# ls
[root@slave-test mysql]# 
[root@slave-test ~]# ll /var/lib/mysql
lrwxrwxrwx 1 root root 12 Oct 12 15:00 /var/lib/mysql -> /data/mysql/
[root@slave-test ~]# ll /data/mysql/
total 0
[root@slave-test ~]# 
{% endhighlight %}

> **Note:**  The  **datadir must be empty** ; Percona XtraBackup innobackupex --copy-back option will not copy over existing files unless innobackupex --force-non-empty-directories option is specified. Also it’s important to note that **MySQL server needs to be shut down** before restore is performed. You can’t restore to a datadir of a running mysqld instance (except when importing a partial backup). 

恢复数据库

{% highlight bash %}
[root@slave-test fullbackup]# innobackupex  --copy-back /data/fullbackup/2015-10-12_15-24-06/ 

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

Get the latest version of Percona XtraBackup, documentation, and help resources:
http://www.percona.com/xb/p

151012 20:40:09  innobackupex: Starting the copy-back operation

IMPORTANT: Please check that the copy-back run completes successfully.
           At the end of a successful copy-back run innobackupex
           prints "completed OK!".

innobackupex: Starting to copy files in '/data/fullbackup/2015-10-12_15-24-06'
innobackupex: back to original data directory '/var/lib/mysql'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/xtrabackup_info' to '/var/lib/mysql/xtrabackup_info'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/xtrabackup_binlog_pos_innodb' to '/var/lib/mysql/xtrabackup_binlog_pos_innodb'
innobackupex: Creating directory '/var/lib/mysql/bhdw_qa'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/bhdw_qa/db.opt' to '/var/lib/mysql/bhdw_qa/db.opt'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/bhdw_qa/user_logins.ibd' to '/var/lib/mysql/bhdw_qa/user_logins.ibd'
...
...
innobackupex: Starting to copy InnoDB system tablespace
innobackupex: in '/data/fullbackup/2015-10-12_15-24-06'
innobackupex: back to original InnoDB data directory '/var/lib/mysql/'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/ibdata1' to '/var/lib/mysql/ibdata1'

innobackupex: Starting to copy InnoDB undo tablespaces
innobackupex: in '/data/fullbackup/2015-10-12_15-24-06'
innobackupex: back to '/var/lib/mysql'

innobackupex: Starting to copy InnoDB log files
innobackupex: in '/data/fullbackup/2015-10-12_15-24-06'
innobackupex: back to original InnoDB log directory '/var/lib/mysql'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/ib_logfile0' to '/var/lib/mysql/ib_logfile0'
innobackupex: Copying '/data/fullbackup/2015-10-12_15-24-06/ib_logfile1' to '/var/lib/mysql/ib_logfile1'
innobackupex: Finished copying back files.

151012 20:44:21  innobackupex: completed OK!
[root@slave-test fullbackup]# echo $?
0
[root@slave-test fullbackup]# 
{% endhighlight %}

恢复完后， **datadir** 里也有一个位置文件

{% highlight bash %}
[root@slave-test mysql]# cat /data/mysql/xtrabackup_binlog_pos_innodb 
mysql-bin.000009	1509223
[root@slave-test mysql]# cat /data/mysql/xtrabackup_info 
uuid = 3cf59860-70b3-11e5-95db-0024213a7622
name = 
tool_name = innobackupex
tool_command = --defaults-file=/etc/my.cnf --user=root --password=... /data/fullbackup/
tool_version = 1.5.1-xtrabackup
ibbackup_version = xtrabackup version 2.2.12 based on MySQL server 5.6.24 Linux (x86_64) (revision id: 8726828)
server_version = 5.6.25-73.0-log
start_time = 2015-10-12 15:24:06
end_time = 2015-10-12 15:31:22
lock_time = 12
binlog_pos = filename 'mysql-bin.000009', position 1509223
innodb_from_lsn = 0
innodb_to_lsn = 76232242511
partial = N
incremental = N
format = file
compact = N
compressed = N
encrypted = N[root@slave-test mysql]# 
{% endhighlight %}

---

### 修改数据权限

{% highlight bash %}
[root@slave-test mysql]# chown -R mysql:mysql /var/lib/mysql 
[root@slave-test mysql]#
[root@slave-test data]# chown -R mysql:mysql  /data/mysql/
[root@slave-test data]# 
{% endhighlight %}

---

### 启动数据库

{% highlight bash %}
[root@slave-test data]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server)...                         [  OK  ]
[root@slave-test data]# 
[root@slave-test data]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.26-74.0-log Percona Server (GPL), Release 74.0, Revision 32f8dfd

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show slave status\G
Empty set (0.00 sec)

mysql> 
{% endhighlight %}

---

## 创建复制用户

在master上创建一个复制用户

{% highlight bash %}
mysql> grant replication slave,replication client on *.* to repl@'192.168.1.%' identified by 'xxxxxx' ;
Query OK, 0 rows affected (0.04 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.08 sec)

mysql> 
{% endhighlight %}

---

## 执行同步

{% highlight bash %}
[root@slave-test mysql]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26-74.0-log Percona Server (GPL), Release 74.0, Revision 32f8dfd

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show slave status\G
Empty set (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='192.168.1.66', MASTER_USER='repl',MASTER_PASSWORD='xxxxxxxx', MASTER_LOG_FILE='mysql-bin.000009', MASTER_LOG_POS=1509223;
Query OK, 0 rows affected, 2 warnings (0.46 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.1.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000009
          Read_Master_Log_Pos: 1509223
               Relay_Log_File: slave-test-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000009
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1509223
              Relay_Log_Space: 120
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.10 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000009
          Read_Master_Log_Pos: 1669199
               Relay_Log_File: slave-test-relay-bin.000002
                Relay_Log_Pos: 19048
        Relay_Master_Log_File: mysql-bin.000009
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1527988
              Relay_Log_Space: 160438
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 20039
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 18
                  Master_UUID: adf0b5b2-26fb-11e5-8bba-0024213a7622
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: System lock
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> 
{% endhighlight %}

稳定后的状态

{% highlight bash %}
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000009
          Read_Master_Log_Pos: 1669482
               Relay_Log_File: slave-test-relay-bin.000002
                Relay_Log_Pos: 160542
        Relay_Master_Log_File: mysql-bin.000009
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1669482
              Relay_Log_Space: 160721
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 18
                  Master_UUID: adf0b5b2-26fb-11e5-8bba-0024213a7622
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> 
{% endhighlight %}




---
[percona-xtrabackup]:https://www.percona.com/software/mysql-database/percona-xtrabackup
[percona]:https://www.percona.com/
[xtrabackupdoc]:https://www.percona.com/doc/percona-xtrabackup/2.2/index.html

