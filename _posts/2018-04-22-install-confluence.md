---
layout: post
title: "Install Confluence"
author:  wilmosfang
date: 2018-04-22 16:00:30
image: '/assets/img/'
excerpt: '安装 Confluence'
main-class: 'tools'
color: '#808080'
tags:
 - tools
 - mysql
 - tomcat
 - confluence
categories:
 - tools
twitter_text: 'simple process of Confluence installation'
introduction: 'Installation of Confluence'
---
# 前言

**[Confluence][confluence]** 是一款 ATLASSIAN 公司出品的文档管理工具

>Confluence is content collaboration software that changes how modern teams work

可以用来作企业的内部知识共享平台

ATLASSIAN 出了很多优秀的产品，大名鼎鼎的 Jira 就是这个公司的产品，Jira 在开发团队中算是比较知名的项目管理软件了

这里演示一下如何构建 **[Confluence][confluence]**

参考 **[Confluence Installation Guide][confluence_installation_guide]**

> **Tip:** 当前的版本为 **confluence 6.8.1**

---

# 操作

## 依赖

每个软件都有它的运行环境，可以通过 **[System Requirements][confluence_system_req]** 确认 **[Confluence][confluence]** 的运行环境

目前的兼容列表：

Desktop browsers:

* Internet Explorer 11
* Microsoft Edge
* Chrome
* Firefox
* Safari (Mac only)

Mobile browsers:

* Chrome
* Firefox
* Safari (iOS only)
* Android 4.4 (KitKat) or later

Mobile operating system: 
(required for mobile app)

* iOS 11 or later
* Android 4.4 (KitKat) or later


Operating systems:

* Microsoft Windows
* Linux (most distributions)
* MacOS / OSX (evaluation only)

PostgreSQL:

* PostgreSQL 9.3
* PostgreSQL 9.4
* PostgreSQL 9.5
* PostgreSQL 9.6

MySQL:

* MySQL 5.6
* MySQL 5.7 

Oracle:

* Oracle 12c (Release 1)

Microsoft SQL Server:

* SQL Server 2012
* SQL Server 2014
* Azure SQL

Embedded database:

* H2 (evaluation only)

Oracle Java JRE / JDK:

* Java 1.8

软件依赖的细节可以参考 **[Supported Platforms][confluence_dep]**

>**Note:**
>* Confluence 只支持软件包中的 Apache Tomcat 版本
>* Confluence 软件包中已经打包好了 Java Runtime Environment (JRE),不需要额外安装
>* 杀毒软件会明显影响 Confluence 的运行速度，所以最好在杀毒软件中排除掉 Confluence 的家目录，索引目录，数据库目录
>* Confluence 只支持在 64 位的 x85 硬件架构之上


>**Tip:** 详细可以参考 **[System Requirements][confluence_system_req]**

## OS 环境

~~~bash
[root@confluence ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@confluence ~]# hostnamectl 
   Static hostname: confluence
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: 54ca7cef2a2e48239ecbd18e8c7a0f8d
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@confluence ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:93:64:5f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 81118sec preferred_lft 81118sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:5c:09:2e brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.218/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@confluence ~]# 
~~~

## 下载与安装 mysql repo

因为 confluence 官方只支持 oracle 的 mysql

>**Note:** Confluence will not work on MySQL variants such as MariaDB (CONFSERVER-29060) and Percona Server (CONFSERVER-36471)

关于 mysql 的支持可以参考 **[Database Setup For MySQL][confluence_mysql]**

所以我得下载和使用 Oracle 版本的 mysql

通过以下链接来下载 mysql 的软件库

**[Download MySQL Yum Repository][mysql_repo]**

~~~
[root@confluence ~]# cd cf/
[root@confluence cf]# ls mysql80-community-release-el7-1.noarch.rpm 
mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# 
[root@confluence cf]# rpm -ivh mysql80-community-release-el7-1.noarch.rpm 
warning: mysql80-community-release-el7-1.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql80-community-release-el7-1  ################################# [100%]
[root@confluence cf]#
[root@confluence cf]# ll /etc/yum.repos.d/ | grep mysql
-rw-r--r--  1 root root 1864 2月  22 03:49 mysql-community.repo
-rw-r--r--  1 root root 1885 2月  22 03:49 mysql-community-source.repo
[root@confluence cf]#
[root@confluence cf]# yum list all | grep mysql80
mysql80-community-release.noarch         el7-1                        installed 
mysql-community-client.i686              8.0.11-1.el7                 mysql80-community
mysql-community-client.x86_64            8.0.11-1.el7                 mysql80-community
mysql-community-common.i686              8.0.11-1.el7                 mysql80-community
mysql-community-common.x86_64            8.0.11-1.el7                 mysql80-community
mysql-community-devel.i686               8.0.11-1.el7                 mysql80-community
mysql-community-devel.x86_64             8.0.11-1.el7                 mysql80-community
mysql-community-embedded-compat.i686     8.0.11-1.el7                 mysql80-community
mysql-community-embedded-compat.x86_64   8.0.11-1.el7                 mysql80-community
mysql-community-libs.i686                8.0.11-1.el7                 mysql80-community
mysql-community-libs.x86_64              8.0.11-1.el7                 mysql80-community
mysql-community-libs-compat.i686         8.0.11-1.el7                 mysql80-community
mysql-community-libs-compat.x86_64       8.0.11-1.el7                 mysql80-community
mysql-community-server.x86_64            8.0.11-1.el7                 mysql80-community
mysql-community-test.x86_64              8.0.11-1.el7                 mysql80-community
                                         1-20180420                   mysql80-community
mysql-ref-manual-8.0-en-pdf.noarch       1-20180420                   mysql80-community
[root@confluence cf]# 
~~~

>**Tip:** 因为官方刚刚发布了 mysql 8 的 GA 版，所以这里就尝尝鲜，其实 confluence 的文档里并没有明文支持 mysql 8, 我这里只是作一个尝试，看看 mysql 能否向下兼容，不过生产中还是建议严格根据官方发布的兼容列表来选择软件版本

## 安装 mysql

~~~
[root@confluence cf]# yum install mysql-community-server.x86_64 --enablerepo=mysql80-community
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package mysql-community-server.x86_64 0:8.0.11-1.el7 will be installed
--> Processing Dependency: mysql-community-common(x86-64) = 8.0.11-1.el7 for package: mysql-community-server-8.0.11-1.el7.x86_64
--> Processing Dependency: mysql-community-client(x86-64) >= 8.0.0 for package: mysql-community-server-8.0.11-1.el7.x86_64
--> Running transaction check
---> Package mysql-community-client.x86_64 0:8.0.11-1.el7 will be installed
--> Processing Dependency: mysql-community-libs(x86-64) >= 8.0.0 for package: mysql-community-client-8.0.11-1.el7.x86_64
---> Package mysql-community-common.x86_64 0:8.0.11-1.el7 will be installed
--> Running transaction check
---> Package mysql-community-libs.x86_64 0:8.0.11-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================================
 Package                                 Arch                    Version                       Repository                          Size
========================================================================================================================================
Installing:
 mysql-community-server                  x86_64                  8.0.11-1.el7                  mysql80-community                  341 M
Installing for dependencies:
 mysql-community-client                  x86_64                  8.0.11-1.el7                  mysql80-community                   26 M
 mysql-community-common                  x86_64                  8.0.11-1.el7                  mysql80-community                  537 k
 mysql-community-libs                    x86_64                  8.0.11-1.el7                  mysql80-community                  2.2 M

Transaction Summary
========================================================================================================================================
Install  1 Package (+3 Dependent packages)

Total download size: 369 M
Installed size: 1.7 G
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/mysql80-community/packages/mysql-community-common-8.0.11-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Public key for mysql-community-common-8.0.11-1.el7.x86_64.rpm is not installed
(1/4): mysql-community-common-8.0.11-1.el7.x86_64.rpm                                                            | 537 kB  00:00:01     
(2/4): mysql-community-libs-8.0.11-1.el7.x86_64.rpm                                                              | 2.2 MB  00:00:04     
(3/4): mysql-community-client-8.0.11-1.el7.x86_64.rpm                                                            |  26 MB  00:00:37     
(4/4): mysql-community-server-8.0.11-1.el7.x86_64.rpm                                                            | 341 MB  00:07:18     
----------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                   852 kB/s | 369 MB  00:07:23     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Importing GPG key 0x5072E1F5:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: a4a9 4068 76fc bd3c 4567 70c8 8c71 8d3b 5072 e1f5
 Package    : mysql80-community-release-el7-1.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : mysql-community-common-8.0.11-1.el7.x86_64                                                                           1/4 
  Installing : mysql-community-libs-8.0.11-1.el7.x86_64                                                                             2/4 
  Installing : mysql-community-client-8.0.11-1.el7.x86_64                                                                           3/4 
  Installing : mysql-community-server-8.0.11-1.el7.x86_64                                                                           4/4 
  Verifying  : mysql-community-server-8.0.11-1.el7.x86_64                                                                           1/4 
  Verifying  : mysql-community-client-8.0.11-1.el7.x86_64                                                                           2/4 
  Verifying  : mysql-community-libs-8.0.11-1.el7.x86_64                                                                             3/4 
  Verifying  : mysql-community-common-8.0.11-1.el7.x86_64                                                                           4/4 

Installed:
  mysql-community-server.x86_64 0:8.0.11-1.el7                                                                                          

Dependency Installed:
  mysql-community-client.x86_64 0:8.0.11-1.el7 mysql-community-common.x86_64 0:8.0.11-1.el7 mysql-community-libs.x86_64 0:8.0.11-1.el7

Complete!
[root@confluence cf]#
~~~

## 配置 mysql

参考 **[Database Setup For MySQL][confluence_mysql]**

对 mysql 进行配置

~~~
[root@confluence mysql]# grep -v "^#" /etc/my.cnf |sed '/^$/d'
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[root@confluence mysql]# vim /etc/my.cnf
[root@confluence mysql]# grep -v "^#" /etc/my.cnf |sed '/^$/d'
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
character-set-server=utf8
collation-server=utf8_bin
default-storage-engine=INNODB
max_allowed_packet=256M
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
[root@confluence mysql]# 
~~~

## 启动 mysql

~~~
[root@confluence mysql]# systemctl start mysqld.service
[root@confluence mysql]# 
[root@confluence mysql]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2018-04-23 13:14:11 EDT; 1min 20s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 5382 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 5456 (mysqld)
   Status: "SERVER_OPERATING"
   CGroup: /system.slice/mysqld.service
           └─5456 /usr/sbin/mysqld

4月 23 13:14:01 confluence systemd[1]: Starting MySQL Server...
4月 23 13:14:11 confluence systemd[1]: Started MySQL Server.
[root@confluence mysql]#
~~~

以下为日志中的内容，里面包含了初始的 root 密码

~~~
2018-04-23T17:14:01.942059Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.11) initializing of server in progress as process 5407
2018-04-23T17:14:01.944317Z 0 [Warning] [MY-010161] [Server] You need to use --log-bin to make --binlog-format work.
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000
2018-04-23T17:14:07.423830Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: Q*tRckwhs9tp
2018-04-23T17:14:09.161078Z 0 [System] [MY-013170] [Server] /usr/sbin/mysqld (mysqld 8.0.11) initializing of server has completed
2018-04-23T17:14:11.304693Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.11) starting as process 5456
2018-04-23T17:14:11.716330Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2018-04-23T17:14:11.774901Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.11'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server - GPL.
~~~

## mysql 安全配置

~~~
[root@confluence mysql]# mysql_secure_installation 

Securing the MySQL server deployment.

Enter password for user root: 
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration
of the plugin.
Using existing password for root.

Estimated strength of the password: 100 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password: 

Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
[root@confluence mysql]# echo $?
0
[root@confluence mysql]# 
~~~

在新版本的 mysql 中都加入了这个脚本　(mariadb 10+ 中也有此脚本)，对于基本的安全加固很有必要

## 创建数据库和用户并且赋权

~~~
[root@confluence mysql]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 8.0.11 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

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
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> CREATE DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
Query OK, 1 row affected, 1 warning (0.04 sec)

mysql>
mysql> CREATE USER 'confluenceuser'@'localhost' IDENTIFIED BY 'confluenceuser';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> 
mysql> CREATE USER 'confluenceuser'@'localhost' IDENTIFIED BY 'M6z#C#LzgJ26ysj3!S';
Query OK, 0 rows affected (0.10 sec)

mysql> GRANT ALL PRIVILEGES ON confluence.* TO 'confluenceuser'@'localhost';
Query OK, 0 rows affected (0.08 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| confluence         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> show create database confluence;
+------------+--------------------------------------------------------------------------------------+
| Database   | Create Database                                                                      |
+------------+--------------------------------------------------------------------------------------+
| confluence | CREATE DATABASE `confluence` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */ |
+------------+--------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> show grants for 'confluenceuser'@'localhost';
+------------------------------------------------------------------------+
| Grants for confluenceuser@localhost                                    |
+------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `confluenceuser`@`localhost`                     |
| GRANT ALL PRIVILEGES ON `confluence`.* TO `confluenceuser`@`localhost` |
+------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> 
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@confluence mysql]# 
~~~


## 下载 Confluence

首先要下载安装包

下载 **[Download Confluence Server][confluence_dl]**

~~~
[root@confluence ~]# cd cf/
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# 
[root@confluence cf]# du -sh atlassian-confluence-6.8.1-x64.bin 
563M	atlassian-confluence-6.8.1-x64.bin
[root@confluence cf]# md5sum atlassian-confluence-6.8.1-x64.bin 
04a95ebd8072785d41d41963ca32f559  atlassian-confluence-6.8.1-x64.bin
[root@confluence cf]# 
~~~

## 安装 Confluence

参考以下链接安装 Confluence

**[Installing Confluence on Linux][confluence_installation_on_linux]**

~~~
[root@confluence ~]# cd cf
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# chmod +x atlassian-confluence-6.8.1-x64.bin 
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# ./atlassian-confluence-6.8.1-x64.bin 
Unpacking JRE ...
Starting Installer ...
四月 23, 2018 1:49:54 下午 java.util.prefs.FileSystemPreferences$1 run
INFO: Created user preferences directory.
四月 23, 2018 1:49:54 下午 java.util.prefs.FileSystemPreferences$2 run
INFO: Created system preferences directory in java.home.

This will install Confluence 6.8.1 on your computer.
OK [o, Enter], Cancel [c]

Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (uses default settings) [1], 
Custom Install (recommended for advanced users) [2, Enter], 
Upgrade an existing Confluence installation [3]


Where should Confluence 6.8.1 be installed?
[/opt/atlassian/confluence]

Default location for Confluence data
[/var/atlassian/application-data/confluence]

Configure which ports Confluence will use.
Confluence requires two TCP ports that are not being used by any other
applications on this machine. The HTTP port is where you will access
Confluence through your browser. The Control port is used to Startup and
Shutdown Confluence.
Use default ports (HTTP: 8090, Control: 8000) - Recommended [1, Enter], Set custom value for HTTP and Control ports [2]

Confluence can be run in the background.
You may choose to run Confluence as a service, which means it will start
automatically whenever the computer restarts.
Install Confluence as Service?
Yes [y, Enter], No [n]


Extracting files ...
                                                                           

Please wait a few moments while we configure Confluence.
Installation of Confluence 6.8.1 is complete
Start Confluence now?
Yes [y, Enter], No [n]


Please wait a few moments while Confluence starts up.
Launching Confluence ...
Installation of Confluence 6.8.1 is complete
Your installation of Confluence 6.8.1 is now ready and can be accessed via
your browser.
Confluence 6.8.1 can be accessed at http://localhost:8090
Finishing installation ...
[root@confluence cf]# echo $?
0
[root@confluence cf]# netstat -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.218:22       192.168.56.1:57706      ESTABLISHED
tcp        0      0 192.168.56.218:22       192.168.56.1:59402      ESTABLISHED
tcp        0      0 192.168.56.218:22       192.168.56.1:59492      ESTABLISHED
tcp6       0      0 :::3306                 :::*                    LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 :::8090                 :::*                    LISTEN     
tcp6       0      0 127.0.0.1:8000          :::*                    LISTEN     
tcp6       0      0 :::33060                :::*                    LISTEN     
tcp6       0      0 127.0.0.1:45156         127.0.0.1:8000          TIME_WAIT  
[root@confluence cf]# ps faux | grep -i confluence
avahi      698  0.0  0.0  30196  1708 ?        Ss   10:29   0:00 avahi-daemon: running [confluence.local]
root     18225  0.0  0.0 112660  1036 pts/0    S+   13:52   0:00  |       \_ grep --color=auto -i confluence
conflue+ 18146 36.5 22.5 3674912 911844 ?      Sl   13:51   0:21 /opt/atlassian/confluence/jre//bin/java -Djava.util.logging.config.file=/opt/atlassian/confluence/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dconfluence.context.path= -Datlassian.plugins.startup.options= -Dorg.apache.tomcat.websocket.DEFAULT_BUFFER_SIZE=32768 -Dsynchrony.enable.xhr.fallback=true -Xms1024m -Xmx1024m -XX:+UseG1GC -Datlassian.plugins.enable.wait=300 -Djava.awt.headless=true -XX:G1ReservePercent=20 -Xloggc:/opt/atlassian/confluence/logs/gc-2018-04-23_13-51-33.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=2M -XX:-PrintGCDetails -XX:+PrintGCDateStamps -XX:-PrintTenuringDistribution -Dignore.endorsed.dirs= -classpath /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar -Dcatalina.base=/opt/atlassian/confluence -Dcatalina.home=/opt/atlassian/confluence -Djava.io.tmpdir=/opt/atlassian/confluence/temp org.apache.catalina.startup.Bootstrap start
[root@confluence cf]# /opt/atlassian/confluence/jre//bin/java -version
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
[root@confluence cf]# java -version
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
[root@confluence cf]# 
~~~

## 防火墙

~~~
[root@confluence cf]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: ssh dhcpv6-client http https
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@confluence cf]# 
[root@confluence cf]# firewall-cmd --add-port 8090/tcp --permanent
success
[root@confluence cf]# firewall-cmd --reload
success
[root@confluence cf]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: ssh dhcpv6-client http https
  ports: 8090/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@confluence cf]#
~~~

确保开放了 8090 端口

## SELINUX

~~~
[root@confluence cf]# getenforce 
Disabled
[root@confluence cf]# 
~~~

Selinux 已经放行

## 破解

下载破解工具

下载confluence_keygen.jar注册机，见附件

链接：https://pan.baidu.com/s/1gg85p4Z 密码：3t5b

~~~
[root@confluence cf]# cp /tmp/confluence_x.zip  . 
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  confluence_x.zip  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# unzip confluence_x.zip 
Archive:  confluence_x.zip
  inflating: confluence╞╞╜т╣д╛▀/confluence_keygen.jar  
  inflating: confluence╞╞╜т╣д╛▀/keygen.bat  
  inflating: confluence╞╞╜т╣д╛▀/keygen.sh  
  inflating: confluence╞╞╜т╣д╛▀/keygen_MacOSX.sh  
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  confluence_x.zip  confluence╞╞╜т╣д╛▀  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# 
[root@confluence cf]# mv confluence╞╞╜т╣д╛▀/ cf_x
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  cf_x  confluence_x.zip  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# cd cf_x/
[root@confluence cf_x]# ls
confluence_keygen.jar  keygen.bat  keygen_MacOSX.sh  keygen.sh
[root@confluence cf_x]#
~~~

将 **`atlassian-extras-decoder-v2-3.3.0.jar`** 拷贝出来

~~~
[root@confluence lib]# pwd
/opt/atlassian/confluence/confluence/WEB-INF/lib
[root@confluence lib]# ll atlassian-extras-decoder-v2*
-rw-r--r-- 1 root root 6684 Apr  4 06:51 atlassian-extras-decoder-v2-3.3.0.jar
[root@confluence lib]# cp atlassian-extras-decoder-v2-3.3.0.jar /root/Desktop/atlassian-extras-2.4.jar
[root@confluence lib]# 
~~~

运行破解程序

![confluence](/assets/img/confluence/confluence05.png)

打上补丁

![confluence](/assets/img/confluence/confluence06.png)

拷贝回源位置

~~~
[root@confluence cf_x]# cp /root/Desktop/atlassian-extras-2.4.jar  /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.3.0.jar 
cp: overwrite ‘/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.3.0.jar’? y
[root@confluence cf_x]# 
~~~

重启 confluence 服务

~~~
[root@confluence cf_x]# sh /opt/atlassian/confluence/bin/stop-confluence.sh
executing using dedicated user
If you encounter issues starting up Confluence, please see the Installation guide at http://confluence.atlassian.com/display/DOC/Confluence+Installation+Guide
/opt/atlassian/confluence/bin/setenv.sh: line 33: cd: /root/cf/cf_x: Not a directory

Server startup logs are located in /opt/atlassian/confluence/logs/catalina.out
---------------------------------------------------------------------------
Using Java: /opt/atlassian/confluence/jre//bin/java
2018-04-23 14:25:34,722 INFO [main] [atlassian.confluence.bootstrap.SynchronyProxyWatchdog] A Context element for ${confluence.context.path}/synchrony-proxy is found in /opt/atlassian/confluence/conf/server.xml. No further action is required
---------------------------------------------------------------------------
Using CATALINA_BASE:   /opt/atlassian/confluence
Using CATALINA_HOME:   /opt/atlassian/confluence
Using CATALINA_TMPDIR: /opt/atlassian/confluence/temp
Using JRE_HOME:        /opt/atlassian/confluence/jre/
Using CLASSPATH:       /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/atlassian/confluence/work/catalina.pid
Tomcat stopped.
[root@confluence cf_x]# echo $?
0
[root@confluence cf_x]# sh /opt/atlassian/confluence/bin/start-confluence.sh

To run Confluence in the foreground, start the server with start-confluence.sh -fg
executing using dedicated user: confluence
If you encounter issues starting up Confluence, please see the Installation guide at http://confluence.atlassian.com/display/DOC/Confluence+Installation+Guide
/opt/atlassian/confluence/bin/setenv.sh: line 33: cd: /root/cf/cf_x: Not a directory

Server startup logs are located in /opt/atlassian/confluence/logs/catalina.out
---------------------------------------------------------------------------
Using Java: /opt/atlassian/confluence/jre//bin/java
2018-04-23 14:25:48,314 INFO [main] [atlassian.confluence.bootstrap.SynchronyProxyWatchdog] A Context element for ${confluence.context.path}/synchrony-proxy is found in /opt/atlassian/confluence/conf/server.xml. No further action is required
---------------------------------------------------------------------------
Using CATALINA_BASE:   /opt/atlassian/confluence
Using CATALINA_HOME:   /opt/atlassian/confluence
Using CATALINA_TMPDIR: /opt/atlassian/confluence/temp
Using JRE_HOME:        /opt/atlassian/confluence/jre/
Using CLASSPATH:       /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/atlassian/confluence/work/catalina.pid
Tomcat started.
[root@confluence cf_x]# echo $?
0
[root@confluence cf_x]#
~~~


## 进行 confluence 配置

关于 confluence 的配置，可以参考 **[Confluence Setup Guide][confluence_stup]**

访问

http://192.168.56.218:8090/

进入配置界面

![confluence](/assets/img/confluence/confluence01.png)

选择语言

![confluence](/assets/img/confluence/confluence02.png)

![confluence](/assets/img/confluence/confluence03.png)

选择产品安装

![confluence](/assets/img/confluence/confluence04.png)

勾选插件 (如果没有 licence 就不用勾选了)

![confluence](/assets/img/confluence/confluence07.png)

随便输入名称，输入 Server ID，生成 Key

![confluence](/assets/img/confluence/confluence08.png)

填入 Key, 下一步

![confluence](/assets/img/confluence/confluence09.png)

选择自己的数据库

![confluence](/assets/img/confluence/confluence10.png)


![confluence](/assets/img/confluence/confluence11.png)

选择 mysql

![confluence](/assets/img/confluence/confluence12.png)

下载 JDBC 可以参考 [Database JDBC Drivers][mysql_driver]

下载地址 [Download Connector][mysql_driver_dl]

下载安装相应版本的 **mysql-connector-java**

~~~
[root@confluence ~]# cd cf
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  cf_x  confluence_x.zip  mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# cp /tmp/mysql-connector-java-8.0.11-1.el7.noarch.rpm  . 
[root@confluence cf]# ls
atlassian-confluence-6.8.1-x64.bin  confluence_x.zip                            mysql-connector-java-8.0.11-1.el7.noarch.rpm
cf_x                                mysql80-community-release-el7-1.noarch.rpm
[root@confluence cf]# rpm -ivh mysql-connector-java-8.0.11-1.el7.noarch.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-connector-java-8.0.11-1.el7################################# [100%]
[root@confluence cf]# echo $?
0
[root@confluence cf]# 
[root@confluence cf]# rpm -ql mysql-connector-java
/usr/share/doc/mysql-connector-java-8.0.11
/usr/share/doc/mysql-connector-java-8.0.11/CHANGES
/usr/share/doc/mysql-connector-java-8.0.11/LICENSE
/usr/share/doc/mysql-connector-java-8.0.11/README
/usr/share/java/mysql-connector-java-8.0.11.jar
[root@confluence cf]#
~~~

拷贝到 **`/opt/atlassian/confluence/confluence/WEB-INF/lib/`** 中并且重启 confluence 服务

~~~
[root@confluence cf]# cp /usr/share/java/mysql-connector-java-8.0.11.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
[root@confluence cf]# sh /opt/atlassian/confluence/bin/stop-confluence.sh
executing using dedicated user
If you encounter issues starting up Confluence, please see the Installation guide at http://confluence.atlassian.com/display/DOC/Confluence+Installation+Guide
/opt/atlassian/confluence/bin/setenv.sh: line 33: cd: /root/cf: Not a directory

Server startup logs are located in /opt/atlassian/confluence/logs/catalina.out
---------------------------------------------------------------------------
Using Java: /opt/atlassian/confluence/jre//bin/java
2018-04-23 14:53:56,828 INFO [main] [atlassian.confluence.bootstrap.SynchronyProxyWatchdog] A Context element for ${confluence.context.path}/synchrony-proxy is found in /opt/atlassian/confluence/conf/server.xml. No further action is required
---------------------------------------------------------------------------
Using CATALINA_BASE:   /opt/atlassian/confluence
Using CATALINA_HOME:   /opt/atlassian/confluence
Using CATALINA_TMPDIR: /opt/atlassian/confluence/temp
Using JRE_HOME:        /opt/atlassian/confluence/jre/
Using CLASSPATH:       /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/atlassian/confluence/work/catalina.pid
Tomcat stopped.
[root@confluence cf]# sh /opt/atlassian/confluence/bin/start-confluence.sh

To run Confluence in the foreground, start the server with start-confluence.sh -fg
executing using dedicated user: confluence
If you encounter issues starting up Confluence, please see the Installation guide at http://confluence.atlassian.com/display/DOC/Confluence+Installation+Guide
/opt/atlassian/confluence/bin/setenv.sh: line 33: cd: /root/cf: Not a directory

Server startup logs are located in /opt/atlassian/confluence/logs/catalina.out
---------------------------------------------------------------------------
Using Java: /opt/atlassian/confluence/jre//bin/java
2018-04-23 14:54:14,378 INFO [main] [atlassian.confluence.bootstrap.SynchronyProxyWatchdog] A Context element for ${confluence.context.path}/synchrony-proxy is found in /opt/atlassian/confluence/conf/server.xml. No further action is required
---------------------------------------------------------------------------
Using CATALINA_BASE:   /opt/atlassian/confluence
Using CATALINA_HOME:   /opt/atlassian/confluence
Using CATALINA_TMPDIR: /opt/atlassian/confluence/temp
Using JRE_HOME:        /opt/atlassian/confluence/jre/
Using CLASSPATH:       /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/atlassian/confluence/work/catalina.pid
Tomcat started.
[root@confluence cf]# echo $?
0
[root@confluence cf]#
~~~

再次访问配置界面，就可以看到 Mysql 的配置项

![confluence](/assets/img/confluence/confluence13.png)

填入相应的参数，然后测试连接

![confluence](/assets/img/confluence/confluence14.png)

此报错为时区问题

~~~
[root@confluence cf]# date 
2018年 04月 23日 星期一 15:05:39 EDT
[root@confluence cf]# timedatectl status
      Local time: 一 2018-04-23 15:05:41 EDT
  Universal time: 一 2018-04-23 19:05:41 UTC
        RTC time: 一 2018-04-23 19:05:39
       Time zone: America/New_York (EDT, -0400)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: yes
 Last DST change: DST began at
                  日 2018-03-11 01:59:59 EST
                  日 2018-03-11 03:00:00 EDT
 Next DST change: DST ends (the clock jumps one hour backwards) at
                  日 2018-11-04 01:59:59 EDT
                  日 2018-11-04 01:00:00 EST
[root@confluence cf]#
[root@confluence cf]# timedatectl list-timezones | grep -i UTC
UTC
[root@confluence cf]#
[root@confluence cf]# timedatectl 
list-timezones  set-local-rtc   set-ntp         set-time        set-timezone    status          
[root@confluence cf]# timedatectl set-timezone UTC
[root@confluence cf]# date
2018年 04月 23日 星期一 19:07:36 UTC
[root@confluence cf]# 
~~~

调整了时区，还是有问题

![confluence](/assets/img/confluence/confluence15.png)

这边考虑可能是 mysql８ 的兼容问题

## 决定重装 mysql5.7

### 删掉 mysql8 

~~~
yum remove  mysql-community-client-8.0.11-1.el7.x86_64 mysql-community-server-8.0.11-1.el7.x86_64 mysql-connector-java-8.0.11-1.el7.noarch mysql-community-libs-8.0.11-1.el7.x86_64  mysql-community-common-8.0.11-1.el7.x86_64
~~~

### 启用 mysql5.7 repo

在 mysql repo 中禁用掉 mysql8 的库， 然后启用 mysql5.7 的库

~~~
[root@confluence cf]# vim /etc/yum.repos.d/mysql-community.repo
[root@confluence cf]# egrep '(name|enabled)' /etc/yum.repos.d/mysql-community.repo
name=MySQL 5.5 Community Server
enabled=0
name=MySQL 5.6 Community Server
enabled=0
name=MySQL 5.7 Community Server
enabled=1
name=MySQL 8.0 Community Server
enabled=0
name=MySQL Connectors Community
enabled=1
name=MySQL Tools Community
enabled=1
name=MySQL Tools Preview
enabled=0
name=MySQL Cluster 7.5 Community
enabled=0
name=MySQL Cluster 7.6 Community
enabled=0
[root@confluence cf]#
[root@confluence cf]# yum list all | grep mysql57
mysql-community-client.i686              5.7.22-1.el7                 mysql57-community
mysql-community-client.x86_64            5.7.22-1.el7                 mysql57-community
mysql-community-common.i686              5.7.22-1.el7                 mysql57-community
mysql-community-common.x86_64            5.7.22-1.el7                 mysql57-community
mysql-community-devel.i686               5.7.22-1.el7                 mysql57-community
mysql-community-devel.x86_64             5.7.22-1.el7                 mysql57-community
mysql-community-embedded.i686            5.7.22-1.el7                 mysql57-community
mysql-community-embedded.x86_64          5.7.22-1.el7                 mysql57-community
mysql-community-embedded-compat.i686     5.7.22-1.el7                 mysql57-community
mysql-community-embedded-compat.x86_64   5.7.22-1.el7                 mysql57-community
mysql-community-embedded-devel.i686      5.7.22-1.el7                 mysql57-community
mysql-community-embedded-devel.x86_64    5.7.22-1.el7                 mysql57-community
mysql-community-libs.i686                5.7.22-1.el7                 mysql57-community
mysql-community-libs.x86_64              5.7.22-1.el7                 mysql57-community
mysql-community-libs-compat.i686         5.7.22-1.el7                 mysql57-community
mysql-community-libs-compat.x86_64       5.7.22-1.el7                 mysql57-community
mysql-community-release.noarch           el7-7                        mysql57-community
mysql-community-server.x86_64            5.7.22-1.el7                 mysql57-community
mysql-community-test.x86_64              5.7.22-1.el7                 mysql57-community
                                         1-20170320                   mysql57-community
mysql-ref-manual-5.5-en-pdf.noarch       1-20170320                   mysql57-community
                                         1-20180304                   mysql57-community
mysql-ref-manual-5.7-en-pdf.noarch       1-20180304                   mysql57-community
mysql57-community-release.noarch         el7-10                       mysql57-community
[root@confluence cf]# 
~~~

### 清理磁盘

~~~
[root@confluence cf]# cd /var/lib/mysql/
[root@confluence mysql]# ls
auto.cnf       binlog.000004  client-cert.pem  ibdata1      mysql.ibd           server-cert.pem  undo_002
binlog.000001  binlog.index   client-key.pem   ib_logfile0  performance_schema  server-key.pem
binlog.000002  ca-key.pem     confluence       ib_logfile1  private_key.pem     sys
binlog.000003  ca.pem         ib_buffer_pool   mysql        public_key.pem      undo_001
[root@confluence mysql]# rm -rf *
[root@confluence mysql]# ls
[root@confluence mysql]# ll /var/lib/mysql/
total 0
[root@confluence mysql]#
~~~

### 安装 mysql5.7

~~~
[root@confluence ~]# yum install mysql-community-server.x86_64
Loaded plugins: fastestmirror, langpacks
mysql-connectors-community                                                               | 2.5 kB  00:00:00     
mysql-tools-community                                                                    | 2.5 kB  00:00:00     
mysql57-community                                                                        | 2.5 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package mysql-community-server.x86_64 0:5.7.22-1.el7 will be installed
--> Processing Dependency: mysql-community-common(x86-64) = 5.7.22-1.el7 for package: mysql-community-server-5.7.22-1.el7.x86_64
--> Processing Dependency: mysql-community-client(x86-64) >= 5.7.9 for package: mysql-community-server-5.7.22-1.el7.x86_64
--> Running transaction check
---> Package mysql-community-client.x86_64 0:5.7.22-1.el7 will be installed
--> Processing Dependency: mysql-community-libs(x86-64) >= 5.7.9 for package: mysql-community-client-5.7.22-1.el7.x86_64
---> Package mysql-community-common.x86_64 0:5.7.22-1.el7 will be installed
--> Running transaction check
---> Package mysql-community-libs.x86_64 0:5.7.22-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================
 Package                           Arch              Version                 Repository                    Size
================================================================================================================
Installing:
 mysql-community-server            x86_64            5.7.22-1.el7            mysql57-community            165 M
Installing for dependencies:
 mysql-community-client            x86_64            5.7.22-1.el7            mysql57-community             24 M
 mysql-community-common            x86_64            5.7.22-1.el7            mysql57-community            274 k
 mysql-community-libs              x86_64            5.7.22-1.el7            mysql57-community            2.1 M

Transaction Summary
================================================================================================================
Install  1 Package (+3 Dependent packages)

Total download size: 191 M
Installed size: 862 M
Is this ok [y/d/N]: y
Downloading packages:
(1/4): mysql-community-common-5.7.22-1.el7.x86_64.rpm                                    | 274 kB  00:00:00     
(2/4): mysql-community-libs-5.7.22-1.el7.x86_64.rpm                                      | 2.1 MB  00:00:01     
(3/4): mysql-community-client-5.7.22-1.el7.x86_64.rpm                                    |  24 MB  00:00:12     
(4/4): mysql-community-server-5.7.22-1.el7.x86_64.rpm                                    | 165 MB  00:01:31     
----------------------------------------------------------------------------------------------------------------
Total                                                                           2.0 MB/s | 191 MB  00:01:33     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : mysql-community-common-5.7.22-1.el7.x86_64                                                   1/4 
  Installing : mysql-community-libs-5.7.22-1.el7.x86_64                                                     2/4 
  Installing : mysql-community-client-5.7.22-1.el7.x86_64                                                   3/4 
  Installing : mysql-community-server-5.7.22-1.el7.x86_64                                                   4/4 
  Verifying  : mysql-community-client-5.7.22-1.el7.x86_64                                                   1/4 
  Verifying  : mysql-community-libs-5.7.22-1.el7.x86_64                                                     2/4 
  Verifying  : mysql-community-common-5.7.22-1.el7.x86_64                                                   3/4 
  Verifying  : mysql-community-server-5.7.22-1.el7.x86_64                                                   4/4 

Installed:
  mysql-community-server.x86_64 0:5.7.22-1.el7                                                                  

Dependency Installed:
  mysql-community-client.x86_64 0:5.7.22-1.el7           mysql-community-common.x86_64 0:5.7.22-1.el7          
  mysql-community-libs.x86_64 0:5.7.22-1.el7            

Complete!
[root@confluence ~]# echo $?
0
[root@confluence ~]# 
~~~

### 配置 mysql

~~~
[root@confluence ~]# grep -v "^#" /etc/my.cnf |sed '/^$/d'
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[root@confluence ~]# vim /etc/my.cnf
[root@confluence ~]# grep -v "^#" /etc/my.cnf |sed '/^$/d'
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
character-set-server=utf8
collation-server=utf8_bin
default-storage-engine=INNODB
max_allowed_packet=256M
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
[root@confluence ~]# 
~~~

### 启动 mysql

~~~
[root@confluence ~]# systemctl start mysqld.service
[root@confluence ~]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-04-24 15:32:54 UTC; 4min 48s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 3129 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 3049 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 3132 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─3132 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

4月 24 15:32:47 confluence systemd[1]: Starting MySQL Server...
4月 24 15:32:54 confluence systemd[1]: Started MySQL Server.
[root@confluence ~]# ps faux | grep mysql
root      3257  0.0  0.0 112660  1028 pts/0    S+   15:37   0:00  |       \_ grep --color=auto mysql
mysql     3132  0.1  4.3 1119416 174584 ?      Sl   15:32   0:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
[root@confluence ~]# netstat  -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.218:22       192.168.56.1:36012      ESTABLISHED
tcp        0      0 192.168.56.218:22       192.168.56.1:35660      ESTABLISHED
tcp6       0      0 :::3306                 :::*                    LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 :::8090                 :::*                    LISTEN     
tcp6       0      0 127.0.0.1:8000          :::*                    LISTEN     
[root@confluence ~]# 
~~~

以下为日志输出

~~~
2018-04-24T15:32:48.281081Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-04-24T15:32:48.283478Z 0 [Warning] You need to use --log-bin to make --binlog-format work.
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000
2018-04-24T15:32:51.436982Z 0 [Warning] InnoDB: New log files created, LSN=45790
2018-04-24T15:32:51.469663Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2018-04-24T15:32:51.559992Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: c00e4512-47d4-11e8-bed2-08002793645f.
2018-04-24T15:32:51.572440Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2018-04-24T15:32:51.575667Z 1 [Note] A temporary password is generated for root@localhost: Xr7s6WQtO/,Q
2018-04-24T15:32:54.619486Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-04-24T15:32:54.623555Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.22) starting as process 3132 ...
2018-04-24T15:32:54.628108Z 0 [Warning] You need to use --log-bin to make --binlog-format work.
2018-04-24T15:32:54.629440Z 0 [Note] InnoDB: PUNCH HOLE support available
2018-04-24T15:32:54.629461Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2018-04-24T15:32:54.629466Z 0 [Note] InnoDB: Uses event mutexes
2018-04-24T15:32:54.629470Z 0 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
2018-04-24T15:32:54.629473Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.3
2018-04-24T15:32:54.629476Z 0 [Note] InnoDB: Using Linux native AIO
2018-04-24T15:32:54.629835Z 0 [Note] InnoDB: Number of pools: 1
2018-04-24T15:32:54.630034Z 0 [Note] InnoDB: Using CPU crc32 instructions
2018-04-24T15:32:54.632224Z 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2018-04-24T15:32:54.640229Z 0 [Note] InnoDB: Completed initialization of buffer pool
2018-04-24T15:32:54.642029Z 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2018-04-24T15:32:54.663453Z 0 [Note] InnoDB: Highest supported file format is Barracuda.
2018-04-24T15:32:54.697210Z 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2018-04-24T15:32:54.697354Z 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2018-04-24T15:32:54.711913Z 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2018-04-24T15:32:54.712895Z 0 [Note] InnoDB: 96 redo rollback segment(s) found. 96 redo rollback segment(s) are active.
2018-04-24T15:32:54.712942Z 0 [Note] InnoDB: 32 non-redo rollback segment(s) are active.
2018-04-24T15:32:54.713502Z 0 [Note] InnoDB: Waiting for purge to start
2018-04-24T15:32:54.763818Z 0 [Note] InnoDB: 5.7.22 started; log sequence number 2589318
2018-04-24T15:32:54.764441Z 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2018-04-24T15:32:54.765179Z 0 [Note] Plugin 'FEDERATED' is disabled.
2018-04-24T15:32:54.768274Z 0 [Note] InnoDB: Buffer pool(s) load completed at 180424 15:32:54
2018-04-24T15:32:54.778809Z 0 [Note] Found ca.pem, server-cert.pem and server-key.pem in data directory. Trying to enable SSL support using them.
2018-04-24T15:32:54.779529Z 0 [Warning] CA certificate ca.pem is self signed.
2018-04-24T15:32:54.786317Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2018-04-24T15:32:54.786439Z 0 [Note] IPv6 is available.
2018-04-24T15:32:54.786467Z 0 [Note]   - '::' resolves to '::';
2018-04-24T15:32:54.786501Z 0 [Note] Server socket created on IP: '::'.
2018-04-24T15:32:54.804728Z 0 [Note] Event Scheduler: Loaded 0 events
2018-04-24T15:32:54.805167Z 0 [Note] /usr/sbin/mysqld: ready for connections.
Version: '5.7.22'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
~~~

### mysql 安全配置

~~~
[root@confluence ~]# mysql_secure_installation 

Securing the MySQL server deployment.

Enter password for user root: 

The existing password for the user account root has expired. Please set a new password.

New password: 

Re-enter new password: 
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration
of the plugin.
Using existing password for root.

Estimated strength of the password: 100 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password: 

Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
[root@confluence ~]# echo $?
0
[root@confluence ~]# 
~~~

### 创建数据库和用户并且赋权

~~~
[root@confluence ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.22 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

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
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> CREATE DATABASE confluence_db CHARACTER SET utf8 COLLATE utf8_bin;
Query OK, 1 row affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON confluence_db.* TO 'confluenceuser'@'localhost' IDENTIFIED BY 'M6z#C#LzgJ26ysj3!S';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| confluence_db      |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> show create database confluence_db;
+---------------+-----------------------------------------------------------------------------------------+
| Database      | Create Database                                                                         |
+---------------+-----------------------------------------------------------------------------------------+
| confluence_db | CREATE DATABASE `confluence_db` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */ |
+---------------+-----------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> show grants for 'confluenceuser'@'localhost';
+---------------------------------------------------------------------------+
| Grants for confluenceuser@localhost                                       |
+---------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'confluenceuser'@'localhost'                        |
| GRANT ALL PRIVILEGES ON `confluence_db`.* TO 'confluenceuser'@'localhost' |
+---------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
[root@confluence ~]#
~~~

### 重启 confluence 服务

~~~
[root@confluence ~]# sh /opt/atlassian/confluence/bin/stop-confluence.sh
executing using dedicated user
If you encounter issues starting up Confluence, please see the Installation guide at http://confluence.atlassian.com/display/DOC/Confluence+Installation+Guide
/opt/atlassian/confluence/bin/setenv.sh: line 33: cd: /root: Permission denied

Server startup logs are located in /opt/atlassian/confluence/logs/catalina.out
---------------------------------------------------------------------------
Using Java: /opt/atlassian/confluence/jre//bin/java
2018-04-24 15:47:52,835 INFO [main] [atlassian.confluence.bootstrap.SynchronyProxyWatchdog] A Context element for ${confluence.context.path}/synchrony-proxy is found in /opt/atlassian/confluence/conf/server.xml. No further action is required
---------------------------------------------------------------------------
Using CATALINA_BASE:   /opt/atlassian/confluence
Using CATALINA_HOME:   /opt/atlassian/confluence
Using CATALINA_TMPDIR: /opt/atlassian/confluence/temp
Using JRE_HOME:        /opt/atlassian/confluence/jre/
Using CLASSPATH:       /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/atlassian/confluence/work/catalina.pid
Tomcat stopped.
[root@confluence ~]# echo $?
0
[root@confluence ~]# sh /opt/atlassian/confluence/bin/start-confluence.sh

To run Confluence in the foreground, start the server with start-confluence.sh -fg
executing using dedicated user: confluence
If you encounter issues starting up Confluence, please see the Installation guide at http://confluence.atlassian.com/display/DOC/Confluence+Installation+Guide
/opt/atlassian/confluence/bin/setenv.sh: line 33: cd: /root: Permission denied

Server startup logs are located in /opt/atlassian/confluence/logs/catalina.out
---------------------------------------------------------------------------
Using Java: /opt/atlassian/confluence/jre//bin/java
2018-04-24 15:48:07,509 INFO [main] [atlassian.confluence.bootstrap.SynchronyProxyWatchdog] A Context element for ${confluence.context.path}/synchrony-proxy is found in /opt/atlassian/confluence/conf/server.xml. No further action is required
---------------------------------------------------------------------------
Using CATALINA_BASE:   /opt/atlassian/confluence
Using CATALINA_HOME:   /opt/atlassian/confluence
Using CATALINA_TMPDIR: /opt/atlassian/confluence/temp
Using JRE_HOME:        /opt/atlassian/confluence/jre/
Using CLASSPATH:       /opt/atlassian/confluence/bin/bootstrap.jar:/opt/atlassian/confluence/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/atlassian/confluence/work/catalina.pid
Tomcat started.
[root@confluence ~]# echo $?
0
[root@confluence ~]# 
~~~

再次进入 confluence 配置界面，输入数据库连接参数，测试数据库连接

测试通过

>**Tip:** 看来目前只能支持 mysql5.7,支持不了 5.8

![confluence](/assets/img/confluence/confluence16.png)

下一步后，选择 示范站点

![confluence](/assets/img/confluence/confluence17.png)

在 confluece 中管理用户和组

![confluence](/assets/img/confluence/confluence18.png)

配置账号

![confluence](/assets/img/confluence/confluence19.png)

设置成功

![confluence](/assets/img/confluence/confluence20.png)

进行欢迎界面

>**Tip:** 这个视频可以跳过

![confluence](/assets/img/confluence/confluence21.png)

观看完视频后右下角会有可以点击的继续按钮

![confluence](/assets/img/confluence/confluence22.png)

提示上传头像，可以直接拖拽

(不得不说，confluence 在写技术文档过程中直接拖拽就可以插入图片是一个特别方便和友好的特性)

![confluence](/assets/img/confluence/confluence23.png)

创建第一个空间

![confluence](/assets/img/confluence/confluence24.png)

进入后的第一个界面，可以开始在空间中创建第一个文档

![confluence](/assets/img/confluence/confluence25.png)

在管理后台中可以选择语言

登录管理后台的过程，会再次确认管理员身份，要求需要输入管理员密码

![confluence](/assets/img/confluence/confluence26.png)

第一个空间的主页

![confluence](/assets/img/confluence/confluence27.png)

从管理后台中可以获取授权信息

![confluence](/assets/img/confluence/confluence28.png)

自己的界面语言可以在个人的设置界面中进行配置

![confluence](/assets/img/confluence/confluence29.png)

中文的空间展示效果

![confluence](/assets/img/confluence/confluence30.png)


关于 Confluence 的其它细节操作可以基于这个状态进行更深入地探索

---

# 总结

Confluence 是一款优秀的团队协作软件，可以方便地进行知识共享 

只不过它是闭源的商业软件，需要花很多钱，并不适合初创型的技术公司使用

但是如果软件费用不是问题，那么它和 jira 的对接也非常方便

它的安装 bin 内集成了 jre 和 tomcat，安装起来十分方便

也可以与大部分主流的关系型数据库进行对接

只是对于 mysql，官方文档里强调只支持 Oracle 版本的 mysql，而 mysql 的变种 percona 和 mariadb 都不被支持

* TOC
{:toc}

---

[confluence]:https://www.atlassian.com/software/confluence
[confluence_installation_guide]:https://confluence.atlassian.com/doc/confluence-installation-guide-135681.html
[confluence_system_req]:https://confluence.atlassian.com/doc/system-requirements-126517514.html
[confluence_dep]:https://confluence.atlassian.com/doc/supported-platforms-207488198.html
[mysql_repo]:https://dev.mysql.com/downloads/repo/yum/
[confluence_mysql]:https://confluence.atlassian.com/doc/database-setup-for-mysql-128747.html
[confluence_installation_on_linux]:https://confluence.atlassian.com/doc/installing-confluence-on-linux-143556824.html#
[confluence_dl]:https://www.atlassian.com/software/confluence/download
[mysql_driver]:https://confluence.atlassian.com/doc/database-jdbc-drivers-171742.html
[mysql_driver_dl]:https://dev.mysql.com/downloads/connector/j/
[confluence_stup]:https://confluence.atlassian.com/doc/confluence-setup-guide-135691.html
