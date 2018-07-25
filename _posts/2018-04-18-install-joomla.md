---
layout: post
title: "Install Joomla"
author:  wilmosfang
date: 2018-04-18 03:29:34
image: '/assets/img/'
excerpt: '安装 Joomla'
main-class: 'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
 - joomla
categories:
 - php
twitter_text: 'simple process of Joomla installation'
introduction: 'Installation of Joomla'
---
# 前言

**[Joomla][joomla]** 是一款用 php 实现的开源 CMS 软件

>The Flexible Platform Empowering Website Creators
>
>Joomla! is an award-winning content management system (CMS), which enables you to build web sites and powerful online applications

因为插件丰富，架构灵活，可以简单而快速实现大部分的网站功能，在国外很受欢迎，连续好几年都获得开源 CMS 的冠军

这里演示一下如何构建 **[Joomla][joomla]**

参考 **[Installing Joomla][joomla_install]**

> **Tip:** 当前的版本为 **Joomla 3.8.6**

---

# 操作

## 依赖

### 软件


Software | Recommended|Minimum|More information
----| ---|---|---|
PHP |5.6 + or 7 +|5.3.10 +|www.php.net|

(Magic Quotes GPC, MB String Overload = off / Zlib Compression Support, XML Support, INI Parser Support, JSON Support, Mcrypt Support, MB Language = Default)

### 数据库

Supported Databases | Recommended|Minimum|More information
----| ---|---|---|
MySQL(InnoDB support required)|5.5.3 +|5.1 +|www.mysql.com|
SQL Server|10.50.1600.1 +|10.50.1600.1 +|www.microsoft.com/sql
PostgreSQL|9.1 +|8.3.18 +|www.postgresql.org

### Web

Supported Web Servers | Recommended|Minimum|More information
----| ---|---|---|
Apache(with mod_mysql, mod_xml, and mod_zlib)|2.4 +|2.x +|www.apache.org
Nginx|1.8 +|1.0 +|wiki.nginx.org
Microsoft IIS|7|7|www.iis.net

当前版本的 **[Joomla][joomla]** 需要满足以上的运行环境


### PHP.ini 推荐配置

* memory_limit - Minimum: 64M Recommended: 128M or better
* upload_max_filesize - Minimum: 20M
* post_max_size - Minimum: 20M
* max_execution_time: At Least 120 Recommended: 300

>**Tip:** 详细可以参考 **[Requirements][joomla_dep]**

## OS 环境

~~~bash
[root@joomla ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@joomla ~]# hostnamectl 
   Static hostname: joomla
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: b1640ef597174e3eb9d093c09535ae13
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@joomla ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:8c:97:19 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84352sec preferred_lft 84352sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ab:2c:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.216/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@joomla ~]# 
~~~

## 软件环境

~~~
[root@joomla ~]# systemctl status mariadb
● mariadb.service - MariaDB 10.2.14 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: active (running) since 二 2018-04-17 23:25:32 EDT; 33min ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 2134 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
  Process: 2090 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=exited, status=0/SUCCESS)
  Process: 2088 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
 Main PID: 2102 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─2102 /usr/sbin/mysqld

4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757285865600 [Note] Plugin 'FEEDBACK' is disabled.
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139756012824320 [Note] InnoDB: Buffer pool(s) l...25:32
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757285865600 [Note] Server socket created on...'::'.
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757285865600 [ERROR] Missing system table my...te it
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757162718976 [Warning] Failed to load slave ...exist
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757285865600 [Note] Reading of all Master_in...ceded
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757285865600 [Note] Added new Master_info ''...table
4月 17 23:25:32 joomla mysqld[2102]: 2018-04-17 23:25:32 139757285865600 [Note] /usr/sbin/mysqld: ready ...ions.
4月 17 23:25:32 joomla mysqld[2102]: Version: '10.2.14-MariaDB'  socket: '/var/lib/mysql/mysql.sock'  po...erver
4月 17 23:25:32 joomla systemd[1]: Started MariaDB 10.2.14 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@joomla ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-04-17 23:21:47 EDT; 37min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1174 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1174 /usr/sbin/httpd -DFOREGROUND
           ├─1215 /usr/sbin/httpd -DFOREGROUND
           ├─1216 /usr/sbin/httpd -DFOREGROUND
           ├─1217 /usr/sbin/httpd -DFOREGROUND
           ├─1218 /usr/sbin/httpd -DFOREGROUND
           └─1219 /usr/sbin/httpd -DFOREGROUND

4月 17 23:21:47 wp systemd[1]: Starting The Apache HTTP Server...
4月 17 23:21:47 wp httpd[1174]: AH00558: httpd: Could not reliably determine the server's fully qualifie...ssage
4月 17 23:21:47 wp systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@joomla ~]# php --version
PHP 7.2.4 (cli) (built: Mar 27 2018 17:23:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.4, Copyright (c) 1999-2018, by Zend Technologies
[root@joomla ~]# rpm -qa | grep httpd
httpd-tools-2.4.6-67.el7.centos.6.x86_64
httpd-2.4.6-67.el7.centos.6.x86_64
[root@joomla ~]# rpm -qa | grep -i mariadb
MariaDB-client-10.2.14-1.el7.centos.x86_64
MariaDB-compat-10.2.14-1.el7.centos.x86_64
MariaDB-common-10.2.14-1.el7.centos.x86_64
MariaDB-server-10.2.14-1.el7.centos.x86_64
[root@joomla ~]#
~~~

## 下载 Joomla 包

~~~
[root@joomla joomla]# ls
[root@joomla joomla]# wget https://downloads.joomla.org/cms/joomla3/3-8-6/Joomla_3-8-6-Stable-Full_Package.zip
--2018-04-18 00:10:24--  https://downloads.joomla.org/cms/joomla3/3-8-6/Joomla_3-8-6-Stable-Full_Package.zip
Resolving downloads.joomla.org (downloads.joomla.org)... 72.29.124.146
Connecting to downloads.joomla.org (downloads.joomla.org)|72.29.124.146|:443... connected.
HTTP request sent, awaiting response... 303 See Other
Location: https://s3-us-west-2.amazonaws.com/joomla-official-downloads/joomladownloads/joomla3/Joomla_3.8.6-Stable-Full_Package.zip?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIZ6S3Q3YQHG57ZRA%2F20180418%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20180418T041024Z&X-Amz-Expires=60&X-Amz-SignedHeaders=host&X-Amz-Signature=8dbc08ccbbe47d114cc58dce84cfcdb2a3e78a8c6e1bf2f984b65b650650a77b [following]
--2018-04-18 00:10:25--  https://s3-us-west-2.amazonaws.com/joomla-official-downloads/joomladownloads/joomla3/Joomla_3.8.6-Stable-Full_Package.zip?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIZ6S3Q3YQHG57ZRA%2F20180418%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20180418T041024Z&X-Amz-Expires=60&X-Amz-SignedHeaders=host&X-Amz-Signature=8dbc08ccbbe47d114cc58dce84cfcdb2a3e78a8c6e1bf2f984b65b650650a77b
Resolving s3-us-west-2.amazonaws.com (s3-us-west-2.amazonaws.com)... 52.218.212.48
Connecting to s3-us-west-2.amazonaws.com (s3-us-west-2.amazonaws.com)|52.218.212.48|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13502576 (13M) [application/zip]
Saving to: ‘Joomla_3-8-6-Stable-Full_Package.zip’

100%[=======================================================================>] 13,502,576   531KB/s   in 38s    

2018-04-18 00:11:04 (348 KB/s) - ‘Joomla_3-8-6-Stable-Full_Package.zip’ saved [13502576/13502576]

[root@joomla joomla]# ls
Joomla_3-8-6-Stable-Full_Package.zip
[root@joomla joomla]# 
~~~

## 解压到合适的位置 

顺便调整权限

~~~
[root@joomla joomla]# ls
Joomla_3-8-6-Stable-Full_Package.zip
[root@joomla joomla]# mkdir /var/www/html/joomla
[root@joomla joomla]# ll /var/www/html/joomla/
total 0
[root@joomla joomla]# 
[root@joomla joomla]# unzip Joomla_3-8-6-Stable-Full_Package.zip  -d /var/www/html/joomla/
...
...
 extracting: /var/www/html/joomla/templates/system/images/selector-arrow.png  
  inflating: /var/www/html/joomla/templates/system/index.php  
  inflating: /var/www/html/joomla/templates/system/offline.php  
   creating: /var/www/html/joomla/tmp/
  inflating: /var/www/html/joomla/tmp/index.html  
  inflating: /var/www/html/joomla/web.config.txt  
[root@joomla joomla]#
[root@joomla joomla]# ll /var/www/html/joomla/
total 56
drwxr-xr-x 11 root root   159 3月  12 18:25 administrator
drwxr-xr-x  2 root root    44 3月  12 18:25 bin
drwxr-xr-x  2 root root    24 3月  12 18:25 cache
drwxr-xr-x  2 root root   169 3月  12 18:25 cli
drwxr-xr-x 19 root root  4096 3月  12 18:25 components
-rw-r--r--  1 root root  3005 3月  12 18:25 htaccess.txt
drwxr-xr-x  5 root root   118 3月  12 18:25 images
drwxr-xr-x  2 root root    64 3月  12 18:25 includes
-rw-r--r--  1 root root  1420 3月  12 18:25 index.php
drwxr-xr-x 14 root root   327 3月  12 18:25 installation
drwxr-xr-x  4 root root    54 3月  12 18:25 language
drwxr-xr-x  5 root root    70 3月  12 18:25 layouts
drwxr-xr-x 12 root root   266 3月  12 18:25 libraries
-rw-r--r--  1 root root 18092 3月  12 18:25 LICENSE.txt
drwxr-xr-x 27 root root  4096 3月  12 18:25 media
drwxr-xr-x 27 root root  4096 3月  12 18:25 modules
drwxr-xr-x 17 root root   268 3月  12 18:25 plugins
-rw-r--r--  1 root root  4872 3月  12 18:25 README.txt
-rw-r--r--  1 root root   836 3月  12 18:25 robots.txt.dist
drwxr-xr-x  5 root root    68 3月  12 18:25 templates
drwxr-xr-x  2 root root    24 3月  12 18:25 tmp
-rw-r--r--  1 root root  1690 3月  12 18:25 web.config.txt
[root@joomla joomla]#
[root@joomla joomla]# chown -R apache.apache /var/www/html/joomla/
[root@joomla joomla]# ll /var/www/html/joomla/
total 56
drwxr-xr-x 11 apache apache   159 3月  12 18:25 administrator
drwxr-xr-x  2 apache apache    44 3月  12 18:25 bin
drwxr-xr-x  2 apache apache    24 3月  12 18:25 cache
drwxr-xr-x  2 apache apache   169 3月  12 18:25 cli
drwxr-xr-x 19 apache apache  4096 3月  12 18:25 components
-rw-r--r--  1 apache apache  3005 3月  12 18:25 htaccess.txt
drwxr-xr-x  5 apache apache   118 3月  12 18:25 images
drwxr-xr-x  2 apache apache    64 3月  12 18:25 includes
-rw-r--r--  1 apache apache  1420 3月  12 18:25 index.php
drwxr-xr-x 14 apache apache   327 3月  12 18:25 installation
drwxr-xr-x  4 apache apache    54 3月  12 18:25 language
drwxr-xr-x  5 apache apache    70 3月  12 18:25 layouts
drwxr-xr-x 12 apache apache   266 3月  12 18:25 libraries
-rw-r--r--  1 apache apache 18092 3月  12 18:25 LICENSE.txt
drwxr-xr-x 27 apache apache  4096 3月  12 18:25 media
drwxr-xr-x 27 apache apache  4096 3月  12 18:25 modules
drwxr-xr-x 17 apache apache   268 3月  12 18:25 plugins
-rw-r--r--  1 apache apache  4872 3月  12 18:25 README.txt
-rw-r--r--  1 apache apache   836 3月  12 18:25 robots.txt.dist
drwxr-xr-x  5 apache apache    68 3月  12 18:25 templates
drwxr-xr-x  2 apache apache    24 3月  12 18:25 tmp
-rw-r--r--  1 apache apache  1690 3月  12 18:25 web.config.txt
[root@joomla joomla]# 
~~~

## 创建数据库和用户

~~~
[root@joomla joomla]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.2.14-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE joomla_db;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON joomla_db.* TO "joomla"@"localhost" IDENTIFIED BY "joomla";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| joomla_db          |
| mysql              |
| performance_schema |
| wiki_db            |
| wp_db              |
+--------------------+
6 rows in set (0.01 sec)

MariaDB [(none)]> show grants for "joomla"@"localhost";
+---------------------------------------------------------------------------------------------------------------+
| Grants for joomla@localhost                                                                                   |
+---------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'joomla'@'localhost' IDENTIFIED BY PASSWORD '*F70658E9BDD2910AC33ACDA164605DFC1DA70A68' |
| GRANT ALL PRIVILEGES ON `joomla_db`.* TO 'joomla'@'localhost'                                                 |
+---------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> exit
Bye
[root@joomla joomla]# 
~~~

可以参考 **[Creating a Database for Joomla][joomla_db]**

## 防火墙

~~~
[root@joomla ~]# firewall-cmd --list-all
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
	
[root@joomla ~]#
~~~

确保开放了 web 端口

## SELINUX

~~~
[root@joomla ~]# getenforce 
Disabled
[root@joomla ~]# 
~~~

Selinux 已经放行

## 进行配置

访问

http://192.168.56.216/joomla/installation/index.php

进入配置界面

![joomla](/assets/img/joomla/joomla01.png)

选择配置过程的语言

![joomla](/assets/img/joomla/joomla02.png)

要填写以下信息

* Site Name: 站点名 — 这个还可以后期在全局系统配置里调整.
* Description: 站点的描述信息. 这个信息可以提供给搜索引擎优化，也可以后期在全局系统配置里调整
* Admin Email Address: 管理员地址，可以用于接收密码确认邮件.
* Admin Username: 管理员名字
* Admin Password: 管理员密码
* Site Offline: 站点的启用状态

![joomla](/assets/img/joomla/joomla03.png)

* Database Type: MySQLi is the common database used
* Hostname: 数据库所在的主机
* Username: 数据库用户名
* Password: 数据库用户密码
* Database Name: 数据库名
* Table Prefix: 表前缀，需要加 (_) 在末尾
* Old Database Process: 对于老表的处置方法，是选择删除还是备份

![joomla](/assets/img/joomla/joomla04.png)

示范数据和邮件信息配置

![joomla](/assets/img/joomla/joomla05.png)

* Main Configuration: 网站配置
Database Configuration: 数据库配置
* Pre-Installation Check: 安装前的依赖检查
* Typical PHP settings: PHP 配置检查
* Recommended Settings: 推荐配置

![joomla](/assets/img/joomla/joomla06.png)

点完安装后，就有一个进度条

安装成功，后可以选择安装语言

![joomla](/assets/img/joomla/joomla07.png)

选择要安装的语言

![joomla](/assets/img/joomla/joomla08.png)

选择中文和英文

![joomla](/assets/img/joomla/joomla10.png)

选择多语言的支持

和前后端使用的语言

![joomla](/assets/img/joomla/joomla11.png)

删除安装目录

![joomla](/assets/img/joomla/joomla12.png)


![joomla](/assets/img/joomla/joomla13.png)

进入前台，可以看到示例网页

![joomla](/assets/img/joomla/joomla14.png)

文章的编辑界面

![joomla](/assets/img/joomla/joomla15.png)

进入后台，可以看到登录的界面

![joomla](/assets/img/joomla/joomla16.png)

进入界面后，有新版本的提醒

![joomla](/assets/img/joomla/joomla17.png)

跟随提醒，可以看到新版本的信息

![joomla](/assets/img/joomla/joomla18.png)

点击安装升级，一会儿就升级完成了

![joomla](/assets/img/joomla/joomla20.png)

版本已经是最新的了，可见进行升级是相当容易的

![joomla](/assets/img/joomla/joomla21.png)

安装插件是相当方便的

![joomla](/assets/img/joomla/joomla19.png)

后台可以方便地进行各种管理操作

关于 Joomla 的其它细节操作可以基于这个状态进行更深入地探索

---

# 总结

Joomla 是一个经典的 LAMP 应用 

所以在 Linux Apache Mysql PHP 等环境准备好了的情况下只需要将相应的软件包解压并放在合适的位置就可以了

不得不说当前的 Joomla 十分强大，可以管控的粒度很细，界面的自定义都有着十分友好的接口

作为一个开源内容发布系统，真的可以满足绝大部分的需求

* TOC
{:toc}

---

[joomla]:https://www.joomla.org/
[joomla_install]:https://docs.joomla.org/Special:MyLanguage/J3.x:Installing_Joomla
[joomla_dep]:https://docs.joomla.org/Special:MyLanguage/J3.x:Installing_Joomla#Requirements
[joomla_db]:https://docs.joomla.org/Creating_a_Database_for_Joomla!