---
layout: post
title: "Install Drupal"
author:  wilmosfang
date: 2018-04-19 15:18:46
image: '/assets/img/'
excerpt: '安装 Drupal'
main-class: 'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
 - drupal
categories:
 - php
twitter_text: 'simple process of Drupal installation'
introduction: 'Installation of Drupal'
---
# 前言

**[Drupal][drupal]** 是一款用 php 实现的开源 CMS 软件

>Drupal is open-source (free) content-management framework , and you can use it to build a wide range of web applications, from basic websites to elaborate API driven monoliths.

因为插件丰富，架构灵活，可以简单而快速实现大部分的网站功能，在国外很受欢迎

这里演示一下如何构建 **[Drupal][drupal]**

参考 **[Installing Drupal 8][drupal_install]**

> **Tip:** 当前的版本为 **Drupal 8.5.2**

---

# 操作

## 依赖

Software |requirements
----| ---
浏览器| **[Browser requirements][drupal_browser]**
Web| **[Web Server][drupal_web]**
php| **[Drupal 8 PHP requirements][drupal_php]**
数据库| **[Database server][drupal_database]**

详细信息可以参考 **[System requirements][drupal_dep]**


## OS 环境

~~~bash
[root@drupal ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@drupal ~]# hostnamectl 
   Static hostname: drupal
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: 086f7e7994e542728868f6f422776936
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@drupal ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:8c:97:19 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83367sec preferred_lft 83367sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ab:2c:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.217/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@drupal ~]# 
~~~

## 软件环境

~~~
[root@drupal ~]# systemctl status mariadb
● mariadb.service - MariaDB 10.2.14 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: active (running) since 五 2018-04-20 23:40:17 EDT; 25s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 2619 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
  Process: 2575 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=exited, status=0/SUCCESS)
  Process: 2573 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
 Main PID: 2587 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─2587 /usr/sbin/mysqld

4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586119506048 [Note] Plugin 'FEEDBACK' is disabled.
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586119506048 [Note] Server socket created on IP: '::'.
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586119506048 [ERROR] Missing system table mysql.roles_mapping; please run mysql_upgrade to create it
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586025043712 [Warning] Failed to load slave replication state from table mysql.gtid_slave_pos: 1146: Table 'm...oesn't exist
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586119506048 [Note] Reading of all Master_info entries succeded
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586119506048 [Note] Added new Master_info '' to hash table
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140586119506048 [Note] /usr/sbin/mysqld: ready for connections.
4月 20 23:40:17 drupal mysqld[2587]: Version: '10.2.14-MariaDB'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MariaDB Server
4月 20 23:40:17 drupal mysqld[2587]: 2018-04-20 23:40:17 140585193162496 [Note] InnoDB: Buffer pool(s) load completed at 180420 23:40:17
4月 20 23:40:17 drupal systemd[1]: Started MariaDB 10.2.14 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@drupal ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2018-04-20 23:40:23 EDT; 27s ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 2627 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)
 Main PID: 2632 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─2632 /usr/sbin/httpd -DFOREGROUND
           ├─2633 /usr/sbin/httpd -DFOREGROUND
           ├─2634 /usr/sbin/httpd -DFOREGROUND
           ├─2635 /usr/sbin/httpd -DFOREGROUND
           ├─2636 /usr/sbin/httpd -DFOREGROUND
           └─2637 /usr/sbin/httpd -DFOREGROUND

4月 20 23:40:23 drupal systemd[1]: Starting The Apache HTTP Server...
4月 20 23:40:23 drupal httpd[2632]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::334c:bc63:1266:56b3. Set the 'Serv...this message
4月 20 23:40:23 drupal systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@drupal ~]# php --version
PHP 7.2.4 (cli) (built: Mar 27 2018 17:23:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.4, Copyright (c) 1999-2018, by Zend Technologies
[root@drupal ~]# rpm -qa | grep httpd
httpd-tools-2.4.6-67.el7.centos.6.x86_64
httpd-2.4.6-67.el7.centos.6.x86_64
[root@drupal ~]# rpm -qa | grep -i mariadb
MariaDB-client-10.2.14-1.el7.centos.x86_64
MariaDB-compat-10.2.14-1.el7.centos.x86_64
MariaDB-common-10.2.14-1.el7.centos.x86_64
MariaDB-server-10.2.14-1.el7.centos.x86_64
[root@drupal ~]# 
~~~

## 下载 drupal 包

~~~
[root@drupal drupal]# ls
[root@drupal drupal]# wget https://ftp.drupal.org/files/projects/drupal-8.5.2.tar.gz
--2018-04-20 23:53:12--  https://ftp.drupal.org/files/projects/drupal-8.5.2.tar.gz
Resolving ftp.drupal.org (ftp.drupal.org)... 151.101.197.175
Connecting to ftp.drupal.org (ftp.drupal.org)|151.101.197.175|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15527091 (15M) [application/octet-stream]
Saving to: ‘drupal-8.5.2.tar.gz’

100%[===============================================================================================================================================>] 15,527,091  3.17MB/s   in 7.6s   

2018-04-20 23:53:21 (1.95 MB/s) - ‘drupal-8.5.2.tar.gz’ saved [15527091/15527091]

[root@drupal drupal]# echo $?
0
[root@drupal drupal]# ls
drupal-8.5.2.tar.gz
[root@drupal drupal]# 
~~~

## 解压到合适的位置 

先解压

~~~bash
[root@drupal drupal]# ls
drupal-8.5.2.tar.gz
[root@drupal drupal]# tar -zxvf drupal-8.5.2.tar.gz
...
...
drupal-8.5.2/vendor/zendframework/zend-feed/src/Writer/Source.php
drupal-8.5.2/vendor/zendframework/zend-feed/src/Writer/StandaloneExtensionManager.php
drupal-8.5.2/vendor/zendframework/zend-feed/src/Writer/Version.php
drupal-8.5.2/vendor/zendframework/zend-feed/src/Writer/Writer.php
drupal-8.5.2/vendor/autoload.php
drupal-8.5.2/vendor/.htaccess
drupal-8.5.2/vendor/web.config
drupal-8.5.2/LICENSE.txt
[root@drupal drupal]# ls
drupal-8.5.2  drupal-8.5.2.tar.gz
[root@drupal drupal]# 
~~~

再拷贝到合适的位置

~~~
[root@drupal drupal]# mkdir /var/www/html/drupal
[root@drupal drupal]# ls drupal-8.5.2
autoload.php  composer.json  composer.lock  core  example.gitignore  index.php  LICENSE.txt  modules  profiles  README.txt  robots.txt  sites  themes  update.php  vendor  web.config
[root@drupal drupal]# cp -r drupal-8.5.2/* /var/www/html/drupal/
[root@drupal drupal]# ls /var/www/html/drupal/
autoload.php  composer.json  composer.lock  core  example.gitignore  index.php  LICENSE.txt  modules  profiles  README.txt  robots.txt  sites  themes  update.php  vendor  web.config
[root@drupal drupal]# 
~~~

顺便调整权限

~~~
[root@drupal drupal]# ll /var/www/html/drupal/
total 224
-rw-r--r--  1 root root    262 4月  21 00:00 autoload.php
-rw-r--r--  1 root root   2740 4月  21 00:00 composer.json
-rw-r--r--  1 root root 161072 4月  21 00:00 composer.lock
drwxr-xr-x 12 root root   4096 4月  21 00:00 core
-rw-r--r--  1 root root   1272 4月  21 00:00 example.gitignore
-rw-r--r--  1 root root    549 4月  21 00:00 index.php
-rw-r--r--  1 root root  18092 4月  21 00:00 LICENSE.txt
drwxr-xr-x  2 root root     24 4月  21 00:00 modules
drwxr-xr-x  2 root root     24 4月  21 00:00 profiles
-rw-r--r--  1 root root   5889 4月  21 00:00 README.txt
-rw-r--r--  1 root root   1596 4月  21 00:00 robots.txt
drwxr-xr-x  3 root root    130 4月  21 00:00 sites
drwxr-xr-x  2 root root     24 4月  21 00:00 themes
-rw-r--r--  1 root root    848 4月  21 00:00 update.php
drwxr-xr-x 17 root root    298 4月  21 00:00 vendor
-rw-r--r--  1 root root   4555 4月  21 00:00 web.config
[root@drupal drupal]# chown -R apache.apache /var/www/html/drupal/
[root@drupal drupal]# ll /var/www/html/drupal/
total 224
-rw-r--r--  1 apache apache    262 4月  21 00:00 autoload.php
-rw-r--r--  1 apache apache   2740 4月  21 00:00 composer.json
-rw-r--r--  1 apache apache 161072 4月  21 00:00 composer.lock
drwxr-xr-x 12 apache apache   4096 4月  21 00:00 core
-rw-r--r--  1 apache apache   1272 4月  21 00:00 example.gitignore
-rw-r--r--  1 apache apache    549 4月  21 00:00 index.php
-rw-r--r--  1 apache apache  18092 4月  21 00:00 LICENSE.txt
drwxr-xr-x  2 apache apache     24 4月  21 00:00 modules
drwxr-xr-x  2 apache apache     24 4月  21 00:00 profiles
-rw-r--r--  1 apache apache   5889 4月  21 00:00 README.txt
-rw-r--r--  1 apache apache   1596 4月  21 00:00 robots.txt
drwxr-xr-x  3 apache apache    130 4月  21 00:00 sites
drwxr-xr-x  2 apache apache     24 4月  21 00:00 themes
-rw-r--r--  1 apache apache    848 4月  21 00:00 update.php
drwxr-xr-x 17 apache apache    298 4月  21 00:00 vendor
-rw-r--r--  1 apache apache   4555 4月  21 00:00 web.config
[root@drupal drupal]# 
~~~

## 创建数据库和用户

~~~
[root@drupal drupal]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 10.2.14-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE drupal_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES ON drupal_db.* TO 'drupal'@'localhost' IDENTIFIED BY 'drupal';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show create database drupal_db;
+-----------+-----------------------------------------------------------------------+
| Database  | Create Database                                                       |
+-----------+-----------------------------------------------------------------------+
| drupal_db | CREATE DATABASE `drupal_db` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+-----------+-----------------------------------------------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> show grants for "drupal"@"localhost";
+------------------------------------------------------------------------------------------------------------------------------------+
| Grants for drupal@localhost                                                                                                        |
+------------------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'drupal'@'localhost' IDENTIFIED BY PASSWORD '*7AFEAE5774E672996251E09B946CB3953FC67656'                      |
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES ON `drupal_db`.* TO 'drupal'@'localhost' |
+------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
[root@drupal drupal]# 
~~~

可以参考 **[Create a database][drupal_create_db]**

## 防火墙

~~~
[root@drupal drupal]# firewall-cmd --list-all
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
	
[root@drupal drupal]# 
~~~

确保开放了 web 端口

## SELINUX

~~~
[root@drupal drupal]# getenforce 
Disabled
[root@drupal drupal]# 
~~~

Selinux 已经放行

## 进行配置

访问 http://192.168.56.217/drupal/core/install.php

进入配置界面

![drupal](/assets/img/drupal/drupal01.png)

选择语言

![drupal](/assets/img/drupal/drupal02.png)

选择安装方式

![drupal](/assets/img/drupal/drupal03.png)

对环境进行评估，看是否有依赖的缺失

![drupal](/assets/img/drupal/drupal04.png)

![drupal](/assets/img/drupal/drupal05.png)

数据库配置

![drupal](/assets/img/drupal/drupal06.png)

安装网站

![drupal](/assets/img/drupal/drupal07.png)

安装翻译

![drupal](/assets/img/drupal/drupal08.png)

设置网站

![drupal](/assets/img/drupal/drupal09.png)

更新配置翻译

![drupal](/assets/img/drupal/drupal11.png)

完成安装，进入站点

![drupal](/assets/img/drupal/drupal12.png)


关于 Drupal 的其它细节操作可以基于这个状态进行更深入地探索

---

# 总结

Drupal 是一个经典的 LAMP 应用 

所以在 Linux Apache Mysql PHP 等环境准备好了的情况下只需要将相应的软件包解压并放在合适的位置就可以了

* TOC
{:toc}

---

[drupal]:https://www.drupal.org/
[drupal_install]:https://www.drupal.org/docs/8/install
[drupal_dep]:https://www.drupal.org/docs/8/system-requirements
[drupal_database]:https://www.drupal.org/docs/8/system-requirements/database-server
[drupal_php]:https://www.drupal.org/docs/8/system-requirements/drupal-8-php-requirements
[drupal_web]:https://www.drupal.org/docs/8/system-requirements/web-server
[drupal_browser]:https://www.drupal.org/docs/8/system-requirements/browser-requirements
[drupal_create_db]:https://www.drupal.org/docs/8/install/step-3-create-a-database
