---
layout: post
title:  Zabbix 监控系统搭建
categories:  linux monitoring zabbix http mysql php
wc: 576  1805 26491  
excerpt: zabbix软件包下载，安装，数据库配置，zabbix服务配置，防火墙配置，访问设置，selinux问题
comments: true
---



#前言

**[Zabbix][zabbix]** 是业内知名的开源监控系统，详细特性就不冗述了，我觉得相较其它监控软件它做得更优秀的一点就是提供了一个完整的解决方案，从收集数据进行展示到报警通知

>Zabbix is the ultimate enterprise-level software designed for monitoring availability and performance of IT infrastructure components. Zabbix is open source and comes at no cost.


下面分享一下 **[Zabbix][zabbix]** 监控系统搭建的基础操作，详细可以参阅 **[官方文档][zabbix_doc]**

> **Tip:** 当前的最新版本为 **Zabbix 2.4.7**

---


#概要

* TOC
{:toc}



---

##环境要求

相关的软硬件需求相对琐碎可以参考 **[Requirements] [requirements]**

---

##安装zabbix软件仓库

{% highlight bash %}
[root@zabbix-server zabbix]# wget http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
--2016-01-15 16:26:48--  http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
Resolving repo.zabbix.com... 87.110.183.174
Connecting to repo.zabbix.com|87.110.183.174|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11296 (11K) [application/x-redhat-package-manager]
Saving to: “zabbix-release-2.4-1.el6.noarch.rpm”

100%[==========================================================================================>] 11,296      30.1K/s   in 0.4s    

2016-01-15 16:26:49 (30.1 KB/s) - “zabbix-release-2.4-1.el6.noarch.rpm” saved [11296/11296]

[root@zabbix-server zabbix]# rpm -ivh zabbix-release-2.4-1.el6.noarch.rpm 
warning: zabbix-release-2.4-1.el6.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 79ea5ed4: NOKEY
Preparing...                ########################################### [100%]
   1:zabbix-release         ########################################### [100%]
[root@zabbix-server zabbix]# ll /etc/yum.repos.d/zabbix.repo 
-rw-r--r--. 1 root root 401 Sep 11  2014 /etc/yum.repos.d/zabbix.repo
[root@zabbix-server zabbix]#
{% endhighlight %}


---

##安装Zabbix软件包


{% highlight bash %}
[root@zabbix-server zabbix]# yum install zabbix-server-mysql zabbix-web-mysql 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * extras: mirrors.pubyun.com
 * updates: mirrors.163.com
Resolving Dependencies
--> Running transaction check
---> Package zabbix-server-mysql.x86_64 0:2.4.7-1.el6 will be installed
--> Processing Dependency: zabbix-server = 2.4.7-1.el6 for package: zabbix-server-mysql-2.4.7-1.el6.x86_64
--> Processing Dependency: libiksemel.so.3()(64bit) for package: zabbix-server-mysql-2.4.7-1.el6.x86_64
--> Processing Dependency: libOpenIPMIposix.so.0()(64bit) for package: zabbix-server-mysql-2.4.7-1.el6.x86_64
--> Processing Dependency: libodbc.so.2()(64bit) for package: zabbix-server-mysql-2.4.7-1.el6.x86_64
--> Processing Dependency: libOpenIPMI.so.0()(64bit) for package: zabbix-server-mysql-2.4.7-1.el6.x86_64
---> Package zabbix-web-mysql.noarch 0:2.4.7-1.el6 will be installed
--> Processing Dependency: zabbix-web = 2.4.7-1.el6 for package: zabbix-web-mysql-2.4.7-1.el6.noarch
--> Processing Dependency: php-mysql for package: zabbix-web-mysql-2.4.7-1.el6.noarch
--> Running transaction check
---> Package OpenIPMI-libs.x86_64 0:2.0.16-14.el6 will be installed
---> Package iksemel.x86_64 0:1.4-2.el6 will be installed
---> Package php-mysql.x86_64 0:5.3.3-46.el6_6 will be installed
--> Processing Dependency: php-common(x86-64) = 5.3.3-46.el6_6 for package: php-mysql-5.3.3-46.el6_6.x86_64
--> Processing Dependency: php-pdo(x86-64) for package: php-mysql-5.3.3-46.el6_6.x86_64
---> Package unixODBC.x86_64 0:2.2.14-14.el6 will be installed
---> Package zabbix-server.x86_64 0:2.4.7-1.el6 will be installed
--> Processing Dependency: zabbix for package: zabbix-server-2.4.7-1.el6.x86_64
--> Processing Dependency: fping for package: zabbix-server-2.4.7-1.el6.x86_64
---> Package zabbix-web.noarch 0:2.4.7-1.el6 will be installed
--> Processing Dependency: php >= 5.3 for package: zabbix-web-2.4.7-1.el6.noarch
--> Processing Dependency: php-gd for package: zabbix-web-2.4.7-1.el6.noarch
--> Processing Dependency: php-mbstring for package: zabbix-web-2.4.7-1.el6.noarch
--> Processing Dependency: php-bcmath for package: zabbix-web-2.4.7-1.el6.noarch
--> Processing Dependency: php-xml for package: zabbix-web-2.4.7-1.el6.noarch
--> Running transaction check
---> Package fping.x86_64 0:2.4b2-16.el6 will be installed
---> Package php.x86_64 0:5.3.3-46.el6_6 will be installed
--> Processing Dependency: php-cli(x86-64) = 5.3.3-46.el6_6 for package: php-5.3.3-46.el6_6.x86_64
---> Package php-bcmath.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package php-common.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package php-gd.x86_64 0:5.3.3-46.el6_6 will be installed
--> Processing Dependency: libXpm.so.4()(64bit) for package: php-gd-5.3.3-46.el6_6.x86_64
---> Package php-mbstring.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package php-pdo.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package php-xml.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package zabbix.x86_64 0:2.4.7-1.el6 will be installed
--> Running transaction check
---> Package libXpm.x86_64 0:3.5.10-2.el6 will be installed
---> Package php-cli.x86_64 0:5.3.3-46.el6_6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                            Arch                  Version                         Repository                           Size
====================================================================================================================================
Installing:
 zabbix-server-mysql                x86_64                2.4.7-1.el6                     zabbix                              1.5 M
 zabbix-web-mysql                   noarch                2.4.7-1.el6                     zabbix                               15 k
Installing for dependencies:
 OpenIPMI-libs                      x86_64                2.0.16-14.el6                   base                                473 k
 fping                              x86_64                2.4b2-16.el6                    zabbix-non-supported                 31 k
 iksemel                            x86_64                1.4-2.el6                       zabbix-non-supported                 47 k
 libXpm                             x86_64                3.5.10-2.el6                    base                                 51 k
 php                                x86_64                5.3.3-46.el6_6                  updates                             1.1 M
 php-bcmath                         x86_64                5.3.3-46.el6_6                  updates                              39 k
 php-cli                            x86_64                5.3.3-46.el6_6                  updates                             2.2 M
 php-common                         x86_64                5.3.3-46.el6_6                  updates                             529 k
 php-gd                             x86_64                5.3.3-46.el6_6                  updates                             111 k
 php-mbstring                       x86_64                5.3.3-46.el6_6                  updates                             459 k
 php-mysql                          x86_64                5.3.3-46.el6_6                  updates                              86 k
 php-pdo                            x86_64                5.3.3-46.el6_6                  updates                              79 k
 php-xml                            x86_64                5.3.3-46.el6_6                  updates                             107 k
 unixODBC                           x86_64                2.2.14-14.el6                   base                                378 k
 zabbix                             x86_64                2.4.7-1.el6                     zabbix                              163 k
 zabbix-server                      x86_64                2.4.7-1.el6                     zabbix                               22 k
 zabbix-web                         noarch                2.4.7-1.el6                     zabbix                              4.5 M

Transaction Summary
====================================================================================================================================
Install      19 Package(s)

Total download size: 12 M
Installed size: 50 M
Is this ok [y/N]: y
Downloading Packages:
(1/19): fping-2.4b2-16.el6.x86_64.rpm                                                                        |  31 kB     00:00     
(2/19): iksemel-1.4-2.el6.x86_64.rpm                                                                         |  47 kB     00:00     
(3/19): php-5.3.3-46.el6_6.x86_64.rpm                                                                        | 1.1 MB     00:01     
(4/19): php-bcmath-5.3.3-46.el6_6.x86_64.rpm                                                                 |  39 kB     00:00     
(5/19): php-cli-5.3.3-46.el6_6.x86_64.rpm                                                                    | 2.2 MB     00:02     
(6/19): php-common-5.3.3-46.el6_6.x86_64.rpm                                                                 | 529 kB     00:00     
(7/19): php-gd-5.3.3-46.el6_6.x86_64.rpm                                                                     | 111 kB     00:00     
(8/19): php-mbstring-5.3.3-46.el6_6.x86_64.rpm                                                               | 459 kB     00:00     
(9/19): php-mysql-5.3.3-46.el6_6.x86_64.rpm                                                                  |  86 kB     00:00     
(10/19): php-pdo-5.3.3-46.el6_6.x86_64.rpm                                                                   |  79 kB     00:00     
(11/19): php-xml-5.3.3-46.el6_6.x86_64.rpm                                                                   | 107 kB     00:00     
(12/19): zabbix-2.4.7-1.el6.x86_64.rpm                                                                       | 163 kB     00:06     
(13/19): zabbix-server-2.4.7-1.el6.x86_64.rpm                                                                |  22 kB     00:01     
(14/19): zabbix-server-mysql-2.4.7-1.el6.x86_64.rpm                                                          | 1.5 MB     00:35     
(15/19): zabbix-web-2.4.7-1.el6.noarch.rpm                                                                   | 4.5 MB     01:37     
(16/19): zabbix-web-mysql-2.4.7-1.el6.noarch.rpm                                                             |  15 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                78 kB/s |  12 MB     02:35     
warning: rpmts_HdrFromFdno: Header V4 DSA/SHA1 Signature, key ID 79ea5ed4: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
Importing GPG key 0x79EA5ED4:
 Userid : Zabbix SIA <packager@zabbix.com>
 Package: zabbix-release-2.4-1.el6.noarch (installed)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : php-common-5.3.3-46.el6_6.x86_64                                                                                1/19 
  Installing : unixODBC-2.2.14-14.el6.x86_64                                                                                   2/19 
  Installing : OpenIPMI-libs-2.0.16-14.el6.x86_64                                                                              3/19 
  Installing : iksemel-1.4-2.el6.x86_64                                                                                        4/19 
  Installing : php-xml-5.3.3-46.el6_6.x86_64                                                                                   5/19 
  Installing : php-pdo-5.3.3-46.el6_6.x86_64                                                                                   6/19 
  Installing : php-mysql-5.3.3-46.el6_6.x86_64                                                                                 7/19 
  Installing : php-cli-5.3.3-46.el6_6.x86_64                                                                                   8/19 
  Installing : php-5.3.3-46.el6_6.x86_64                                                                                       9/19 
  Installing : php-bcmath-5.3.3-46.el6_6.x86_64                                                                               10/19 
  Installing : php-mbstring-5.3.3-46.el6_6.x86_64                                                                             11/19 
  Installing : fping-2.4b2-16.el6.x86_64                                                                                      12/19 
  Installing : libXpm-3.5.10-2.el6.x86_64                                                                                     13/19 
  Installing : php-gd-5.3.3-46.el6_6.x86_64                                                                                   14/19 
  Installing : zabbix-web-mysql-2.4.7-1.el6.noarch                                                                            15/19 
  Installing : zabbix-web-2.4.7-1.el6.noarch                                                                                  16/19 
  Installing : zabbix-2.4.7-1.el6.x86_64                                                                                      17/19 
  Installing : zabbix-server-2.4.7-1.el6.x86_64                                                                               18/19 
  Installing : zabbix-server-mysql-2.4.7-1.el6.x86_64                                                                         19/19 
  Verifying  : php-xml-5.3.3-46.el6_6.x86_64                                                                                   1/19 
  Verifying  : iksemel-1.4-2.el6.x86_64                                                                                        2/19 
  Verifying  : php-pdo-5.3.3-46.el6_6.x86_64                                                                                   3/19 
  Verifying  : zabbix-server-2.4.7-1.el6.x86_64                                                                                4/19 
  Verifying  : zabbix-2.4.7-1.el6.x86_64                                                                                       5/19 
  Verifying  : php-cli-5.3.3-46.el6_6.x86_64                                                                                   6/19 
  Verifying  : php-5.3.3-46.el6_6.x86_64                                                                                       7/19 
  Verifying  : OpenIPMI-libs-2.0.16-14.el6.x86_64                                                                              8/19 
  Verifying  : libXpm-3.5.10-2.el6.x86_64                                                                                      9/19 
  Verifying  : unixODBC-2.2.14-14.el6.x86_64                                                                                  10/19 
  Verifying  : php-common-5.3.3-46.el6_6.x86_64                                                                               11/19 
  Verifying  : zabbix-server-mysql-2.4.7-1.el6.x86_64                                                                         12/19 
  Verifying  : zabbix-web-2.4.7-1.el6.noarch                                                                                  13/19 
  Verifying  : zabbix-web-mysql-2.4.7-1.el6.noarch                                                                            14/19 
  Verifying  : php-bcmath-5.3.3-46.el6_6.x86_64                                                                               15/19 
  Verifying  : php-mysql-5.3.3-46.el6_6.x86_64                                                                                16/19 
  Verifying  : php-mbstring-5.3.3-46.el6_6.x86_64                                                                             17/19 
  Verifying  : php-gd-5.3.3-46.el6_6.x86_64                                                                                   18/19 
  Verifying  : fping-2.4b2-16.el6.x86_64                                                                                      19/19 

Installed:
  zabbix-server-mysql.x86_64 0:2.4.7-1.el6                           zabbix-web-mysql.noarch 0:2.4.7-1.el6                          

Dependency Installed:
  OpenIPMI-libs.x86_64 0:2.0.16-14.el6         fping.x86_64 0:2.4b2-16.el6                iksemel.x86_64 0:1.4-2.el6                
  libXpm.x86_64 0:3.5.10-2.el6                 php.x86_64 0:5.3.3-46.el6_6                php-bcmath.x86_64 0:5.3.3-46.el6_6        
  php-cli.x86_64 0:5.3.3-46.el6_6              php-common.x86_64 0:5.3.3-46.el6_6         php-gd.x86_64 0:5.3.3-46.el6_6            
  php-mbstring.x86_64 0:5.3.3-46.el6_6         php-mysql.x86_64 0:5.3.3-46.el6_6          php-pdo.x86_64 0:5.3.3-46.el6_6           
  php-xml.x86_64 0:5.3.3-46.el6_6              unixODBC.x86_64 0:2.2.14-14.el6            zabbix.x86_64 0:2.4.7-1.el6               
  zabbix-server.x86_64 0:2.4.7-1.el6           zabbix-web.noarch 0:2.4.7-1.el6           

Complete!
[root@zabbix-server zabbix]# 
{% endhighlight %}


---

##初始化数据库

zabbix的数据需要存到数据库

我选择mysql进行存储，mysql的安装过程就不在这里浪费篇幅了


###创建zabbix数据库

{% highlight bash %}
[root@zabbix-server zabbix]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.6.25-73.1-log Percona Server (GPL), Release 73.1, Revision 07b797f

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
| bhtest             |
| mysql              |
| performance_schema |
| pt                 |
| test               |
+--------------------+
6 rows in set (0.00 sec)

mysql> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.01 sec)

mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
Query OK, 0 rows affected (0.00 sec)

mysql> quit;
Bye
[root@zabbix-server zabbix]# 
{% endhighlight %}

###导入初始schema与数据


初始化数据在 **`/usr/share/doc/zabbix-server-mysql-2.4.7/create/`** 中

{% highlight bash %}
[root@zabbix-server zabbix]# ll /usr/share/doc/zabbix-server-mysql-2.4.7/create/
total 2988
-rw-r--r--. 1 root root  972942 Nov 13 18:31 data.sql
-rw-r--r--. 1 root root 1978341 Nov 12 18:12 images.sql
-rw-r--r--. 1 root root  104816 Nov 12 18:39 schema.sql
[root@zabbix-server zabbix]# cd /usr/share/doc/zabbix-server-mysql-2.4.7/create/
[root@zabbix-server create]# ll
total 2988
-rw-r--r--. 1 root root  972942 Nov 13 18:31 data.sql
-rw-r--r--. 1 root root 1978341 Nov 12 18:12 images.sql
-rw-r--r--. 1 root root  104816 Nov 12 18:39 schema.sql
[root@zabbix-server create]# mysql -u root -p zabbix < schema.sql 
Enter password: 
[root@zabbix-server create]# mysql -u root -p zabbix < images.sql 
Enter password: 
[root@zabbix-server create]# mysql -u root -p zabbix < data.sql 
Enter password: 
[root@zabbix-server create]#
{% endhighlight %}

---

##配置zabbix server

确保如下参数已经正确配置

{% highlight bash %}
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
{% endhighlight %}

操作过程

{% highlight bash %}
[root@zabbix-server create]# grep -v "^#" /etc/zabbix/zabbix_server.conf | cat -s 

LogFile=/var/log/zabbix/zabbix_server.log

LogFileSize=0

PidFile=/var/run/zabbix/zabbix_server.pid

DBName=zabbix

DBUser=zabbix

DBSocket=/var/lib/mysql/mysql.sock

SNMPTrapperFile=/var/log/snmptt/snmptt.log

AlertScriptsPath=/usr/lib/zabbix/alertscripts

ExternalScripts=/usr/lib/zabbix/externalscripts

[root@zabbix-server create]# vim /etc/zabbix/zabbix_server.conf
[root@zabbix-server create]# grep -v "^#" /etc/zabbix/zabbix_server.conf | cat -s 

LogFile=/var/log/zabbix/zabbix_server.log

LogFileSize=0

PidFile=/var/run/zabbix/zabbix_server.pid

DBHost=localhost

DBName=zabbix

DBUser=zabbix

DBPassword=zabbix

DBSocket=/var/lib/mysql/mysql.sock

SNMPTrapperFile=/var/log/snmptt/snmptt.log

AlertScriptsPath=/usr/lib/zabbix/alertscripts

ExternalScripts=/usr/lib/zabbix/externalscripts

[root@zabbix-server create]# diff /tmp/old  /tmp/new 
7a8,9
> DBHost=localhost
> 
11a14,15
> DBPassword=zabbix
> 
[root@zabbix-server create]# 
{% endhighlight %}

---

##配置前端php


将 **date.timezone** 配置成正确的时区 **Asia/Shanghai**

{% highlight bash %}
[root@zabbix-server conf.d]# vim /etc/httpd/conf.d/zabbix.conf 
[root@zabbix-server conf.d]# grep  php_value  /etc/httpd/conf.d/zabbix.conf 
        php_value max_execution_time 300
        php_value memory_limit 128M
        php_value post_max_size 16M
        php_value upload_max_filesize 2M
        php_value max_input_time 300
        # php_value date.timezone Europe/Riga
        php_value date.timezone Asia/Shanghai
[root@zabbix-server conf.d]#  
{% endhighlight %}

---

##打开防火墙


允许web访问

{% highlight bash %}
[root@zabbix-server conf.d]# iptables -L -nv | grep 80
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:2180 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8000 
[root@zabbix-server conf.d]# vim /etc/sysconfig/iptables
[root@zabbix-server conf.d]# grep 80 /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2180 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8000 -j ACCEPT
[root@zabbix-server conf.d]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@zabbix-server conf.d]# iptables -L -nv | grep 80
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:80 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:2180 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8000 
[root@zabbix-server conf.d]# 
{% endhighlight %}

---

##启动服务


分别启动 **httpd** 和 **zabbix-server**

{% highlight bash %}
[root@zabbix-server conf.d]# /etc/init.d/httpd start 
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using zabbix-server.temp for ServerName
                                                           [  OK  ]
[root@zabbix-server conf.d]#
[root@zabbix-server conf.d]# /etc/init.d/zabbix-server start 
Starting Zabbix server:                                    [  OK  ]
[root@zabbix-server conf.d]# 
{% endhighlight %}

---

##进行访问


* 1.欢迎界面

![zabbix1.png](/images/zabbix/zabbix1.png)

* 2.预检

![zabbix2.png](/images/zabbix/zabbix2.png)

* 3.配置数据连接

![zabbix3.png](/images/zabbix/zabbix3.png)

测试连接

![zabbix4.png](/images/zabbix/zabbix4.png)


* 4.服务器详情

![zabbix5.png](/images/zabbix/zabbix5.png)

* 5.预装概要

![zabbix6.png](/images/zabbix/zabbix6.png)

* 6.执行安装

![zabbix7.png](/images/zabbix/zabbix7.png)


###登录界面


![zabbix8.png](/images/zabbix/zabbix8.png)

默认密码为 Admin/zabbix

![zabbix9.png](/images/zabbix/zabbix9.png)


---

###报错


![zabbix10.png](/images/zabbix/zabbix10.png)


原因是 SELinux 的干扰，解决办法是关掉

{% highlight bash %}
[root@zabbix-server conf.d]# getenforce 
Enforcing
[root@zabbix-server conf.d]# setenforce 0
[root@zabbix-server conf.d]# getenforce 
Permissive
[root@zabbix-server conf.d]# 
{% endhighlight %}

报警就消失了

但这是暂时的，服务器重启后会失效，为避免重启后这个问题复现，可以将 SELINUX 关闭

方法如下

{% highlight bash %}
[root@zabbix-server conf.d]# grep  "^SELI" /etc/sysconfig/selinux 
SELINUX=enforcing
SELINUXTYPE=targeted 
[root@zabbix-server conf.d]# vim /etc/sysconfig/selinux 
[root@zabbix-server conf.d]# grep  "^SELI" /etc/sysconfig/selinux 
SELINUX=disabled
SELINUXTYPE=targeted 
[root@zabbix-server conf.d]# 
{% endhighlight %}

---


#命令汇总

* **`wget http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm`**
* **`rpm -ivh zabbix-release-2.4-1.el6.noarch.rpm`**
* **`ll /etc/yum.repos.d/zabbix.repo`**
* **`yum install zabbix-server-mysql zabbix-web-mysql`**
* **`mysql -u root -p`**
* **`cd /usr/share/doc/zabbix-server-mysql-2.4.7/create/`**
* **`mysql -u root -p zabbix < schema.sql`**
* **`mysql -u root -p zabbix < images.sql`**
* **`mysql -u root -p zabbix < data.sql`**
* **`vim /etc/zabbix/zabbix_server.conf`**
* **`grep -v "^#" /etc/zabbix/zabbix_server.conf | cat -s`**
* **`diff /tmp/old  /tmp/new`**
* **`vim /etc/httpd/conf.d/zabbix.conf`**
* **`grep  php_value  /etc/httpd/conf.d/zabbix.conf`**
* **`vim /etc/sysconfig/iptables`**
* **`grep 80 /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv | grep 80`**
* **`/etc/init.d/httpd start`**
* **`/etc/init.d/zabbix-server start`**
* **`setenforce 0`**
* **`getenforce`**
* **`vim /etc/sysconfig/selinux`**
* **`grep  "^SELI" /etc/sysconfig/selinux`**


---

[zabbix]:http://www.zabbix.com/
[zabbix_doc]:http://www.zabbix.com/documentation.php
[requirements]:https://www.zabbix.com/documentation/2.4/manual/installation/requirements







