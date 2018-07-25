---
layout: post
title: "Install MediaWiki"
author:  wilmosfang
date: 2018-04-15 10:00:32
image: '/assets/img/'
excerpt: '安装 MediaWiki'
main-class: 'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
 - mediawiki
categories:
 - php
twitter_text: 'simple process of MediaWiki installation'
introduction: 'Installation of MediaWiki'
---
# 前言

**[MediaWiki][mediawiki]** 是一款用 php 实现的开源 **wiki** 软件

>MediaWiki is a free software open source wiki package written in PHP, originally for use on Wikipedia.
>
>It is now also used by several other projects of the non-profit Wikimedia Foundation and by many other wikis, including this website, the home of MediaWiki.

作为一种具备开源精神的知识共享平台，**wiki** 在全世界都广受欢迎

使用 **[MediaWiki][mediawiki]** 可以在组织内部构建一个信息共享平台，用于知识分享

在技术型组织里，可以使用它来构建一个词条文档管理系统

这里演示一下如何构建 **[MediaWiki][mediawiki]**

参考: 

**[Running MediaWiki on Red Hat Linux][mediawiki_install]** 

**[ Installation guide][mediawiki_install_guide]**

**[Installing MediaWiki][mediawiki_install_brif]**

> **Tip:** 当前的版本为 **MediaWiki 1.30.0**

---

# 操作

## 依赖

* Web server such as Apache or IIS
  - Local or command line access is needed for running maintenance scripts
* PHP version 5.5.9 or later
  - with Perl Compatible Regular Expressions
  - with Standard PHP Library
  - with JSON support
* Database Server, one of the following:
  - MySQL 5.5.8+ (*)
  - MariaDB
  - PostgreSQL 8.3+
  - SQLite

详细依赖可以参考 **[Installation requirements][mediawiki_dep]**

## 环境

~~~
[root@wiki ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@wiki ~]# hostnamectl 
   Static hostname: wiki
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: d7231d0112bb46d88ed5bc34302d88e1
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@wiki ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:8c:97:19 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 72852sec preferred_lft 72852sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ab:2c:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.214/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@wiki ~]# 
~~~

## 软件环境

~~~
[root@wiki ~]# systemctl status mariadb.service
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-04-15 06:00:03 EDT; 3h 46min ago
  Process: 1230 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 1162 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 1229 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─1229 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─1446 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log...

4月 15 06:00:01 wiki systemd[1]: Starting MariaDB database server...
4月 15 06:00:01 wiki mariadb-prepare-db-dir[1162]: Database MariaDB is probably initialized in /var/lib/mysql alre...done.
4月 15 06:00:01 wiki mariadb-prepare-db-dir[1162]: If this is not the case, make sure the /var/lib/mysql is empty ...-dir.
4月 15 06:00:01 wiki mysqld_safe[1229]: 180415 06:00:01 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
4月 15 06:00:02 wiki mysqld_safe[1229]: 180415 06:00:02 mysqld_safe Starting mysqld daemon with databases from /v.../mysql
4月 15 06:00:03 wiki systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wiki ~]# systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-04-15 06:00:02 EDT; 3h 46min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1180 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1180 /usr/sbin/httpd -DFOREGROUND
           ├─1419 /usr/sbin/httpd -DFOREGROUND
           ├─1420 /usr/sbin/httpd -DFOREGROUND
           ├─1421 /usr/sbin/httpd -DFOREGROUND
           ├─1422 /usr/sbin/httpd -DFOREGROUND
           └─1423 /usr/sbin/httpd -DFOREGROUND

4月 15 06:00:01 wiki systemd[1]: Starting The Apache HTTP Server...
4月 15 06:00:01 wiki httpd[1180]: AH00558: httpd: Could not reliably determine the server's fully qualified domai...essage
4月 15 06:00:02 wiki systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wiki ~]# php --version
PHP 7.2.4 (cli) (built: Mar 27 2018 17:23:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.4, Copyright (c) 1999-2018, by Zend Technologies
[root@wiki ~]# 
~~~

## 依赖包安装

确认已经成功安装以下包

~~~
[root@wiki ~]# yum install httpd php php-mysql php-gd php-xml mariadb-server mariadb php-mbstring
Loaded plugins: fastestmirror, langpacks
base                                                                                                | 3.6 kB  00:00:00     
epel/x86_64/metalink                                                                                | 7.4 kB  00:00:00     
extras                                                                                              | 3.4 kB  00:00:00     
remi-php72                                                                                          | 2.9 kB  00:00:00     
remi-safe                                                                                           | 2.9 kB  00:00:00     
updates                                                                                             | 3.4 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.smartmedia.net.id
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Package httpd-2.4.6-67.el7.centos.6.x86_64 already installed and latest version
Package php-7.2.4-1.el7.remi.x86_64 already installed and latest version
Package php-mysql-5.4.16-43.el7_4.1.x86_64 is obsoleted by php-mysqlnd-7.2.4-1.el7.remi.x86_64 which is already installed
Package php-gd-7.2.4-1.el7.remi.x86_64 already installed and latest version
Package php-xml-7.2.4-1.el7.remi.x86_64 already installed and latest version
Package 1:mariadb-server-5.5.56-2.el7.x86_64 already installed and latest version
Package 1:mariadb-5.5.56-2.el7.x86_64 already installed and latest version
Package php-mbstring-7.2.4-1.el7.remi.x86_64 already installed and latest version
Nothing to do
[root@wiki ~]# 
~~~

## 创建数据库和为用户分配权限

~~~
[root@wiki ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'wiki'@'localhost' IDENTIFIED BY 'wiki';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> CREATE DATABASE wiki_db;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON wiki_db.* TO 'wiki'@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wiki_db            |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> show create database wiki_db;
+----------+--------------------------------------------------------------------+
| Database | Create Database                                                    |
+----------+--------------------------------------------------------------------+
| wiki_db  | CREATE DATABASE `wiki_db` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+--------------------------------------------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> SHOW GRANTS FOR 'wiki'@'localhost';
+-------------------------------------------------------------------------------------------------------------+
| Grants for wiki@localhost                                                                                   |
+-------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'wiki'@'localhost' IDENTIFIED BY PASSWORD '*A5DB2D927D6DF94DA5E1CE4B293AEAAB4D8304EA' |
| GRANT ALL PRIVILEGES ON `wiki_db`.* TO 'wiki'@'localhost'                                                   |
+-------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> exit
Bye
[root@wiki ~]# 
~~~

## 下载 wikimedia 软件包

~~~
[root@wiki ~]# cd wiki/
[root@wiki wiki]# ls
[root@wiki wiki]# wget http://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
--2018-04-15 10:13:59--  http://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
Resolving releases.wikimedia.org (releases.wikimedia.org)... 208.80.153.248, 2620:0:860:ed1a::3:d
Connecting to releases.wikimedia.org (releases.wikimedia.org)|208.80.153.248|:80... connected.
HTTP request sent, awaiting response... 301 TLS Redirect
Location: https://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz [following]
--2018-04-15 10:14:00--  https://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
Connecting to releases.wikimedia.org (releases.wikimedia.org)|208.80.153.248|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 42680041 (41M) [application/x-gzip]
Saving to: ‘mediawiki-1.30.0.tar.gz’

100%[=================================================================================>] 42,680,041  2.59MB/s   in 28s    

2018-04-15 10:14:29 (1.44 MB/s) - ‘mediawiki-1.30.0.tar.gz’ saved [42680041/42680041]

[root@wiki wiki]# ls
mediawiki-1.30.0.tar.gz
[root@wiki wiki]# 
~~~

## 解压 mediawiki 到合适的路径

~~~
[root@wiki wiki]# ls
mediawiki-1.30.0.tar.gz
[root@wiki wiki]# tar -zxf  mediawiki-1.30.0.tar.gz 
[root@wiki wiki]# ls
mediawiki-1.30.0  mediawiki-1.30.0.tar.gz
[root@wiki wiki]# cp -r mediawiki-1.30.0 /var/www/html/mediawiki
[root@wiki wiki]# 
[root@wiki wiki]# chown -R apache.apache /var/www/html/
[root@wiki wiki]# ll /var/www/html/mediawiki/
total 1184
-rw-r--r--  1 apache apache   4697 4月  15 10:22 api.php
-rw-r--r--  1 apache apache 131511 4月  15 10:22 autoload.php
drwxr-xr-x  2 apache apache     23 4月  15 10:22 cache
-rw-r--r--  1 apache apache    116 4月  15 10:22 CODE_OF_CONDUCT.md
-rw-r--r--  1 apache apache   3595 4月  15 10:22 composer.json
-rw-r--r--  1 apache apache    102 4月  15 10:22 composer.local.json-sample
-rw-r--r--  1 apache apache  19419 4月  15 10:22 COPYING
-rw-r--r--  1 apache apache  10924 4月  15 10:22 CREDITS
drwxr-xr-x  8 apache apache   4096 4月  15 10:22 docs
drwxr-xr-x 19 apache apache    334 4月  15 10:22 extensions
-rw-r--r--  1 apache apache     95 4月  15 10:22 FAQ
-rw-r--r--  1 apache apache   3735 4月  15 10:22 Gruntfile.js
-rw-r--r--  1 apache apache 867797 4月  15 10:22 HISTORY
drwxr-xr-x  2 apache apache     37 4月  15 10:22 images
-rw-r--r--  1 apache apache   7705 4月  15 10:22 img_auth.php
drwxr-xr-x 66 apache apache   8192 4月  15 10:22 includes
-rw-r--r--  1 apache apache   1623 4月  15 10:22 index.php
-rw-r--r--  1 apache apache   3663 4月  15 10:22 INSTALL
-rw-r--r--  1 apache apache   2025 4月  15 10:22 jsduck.json
drwxr-xr-x  6 apache apache    229 4月  15 10:22 languages
-rw-r--r--  1 apache apache   1965 4月  15 10:22 load.php
drwxr-xr-x 17 apache apache   8192 4月  15 10:22 maintenance
drwxr-xr-x  4 apache apache    110 4月  15 10:22 mw-config
-rw-r--r--  1 apache apache   4059 4月  15 10:22 opensearch_desc.php
-rw-r--r--  1 apache apache   3274 4月  15 10:22 phpcs.xml
-rw-r--r--  1 apache apache  12013 4月  15 10:22 profileinfo.php
-rw-r--r--  1 apache apache   1529 4月  15 10:22 README
-rw-r--r--  1 apache apache  16790 4月  15 10:22 RELEASE-NOTES-1.30
drwxr-xr-x  5 apache apache     63 4月  15 10:22 resources
drwxr-xr-x  2 apache apache    144 4月  15 10:22 serialized
drwxr-xr-x  6 apache apache     83 4月  15 10:22 skins
-rw-r--r--  1 apache apache   1703 4月  15 10:22 StartProfiler.sample
drwxr-xr-x  9 apache apache    126 4月  15 10:22 tests
-rw-r--r--  1 apache apache   1087 4月  15 10:22 thumb_handler.php
-rw-r--r--  1 apache apache  21335 4月  15 10:22 thumb.php
-rw-r--r--  1 apache apache  12244 4月  15 10:22 UPGRADE
drwxr-xr-x 22 apache apache   4096 4月  15 10:22 vendor
[root@wiki wiki]#
~~~

## 确认防火墙开放

~~~
[root@wiki wiki]# firewall-cmd --list-all
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
	
[root@wiki wiki]#
~~~

## 确认 SELINUX 放行

~~~
[root@wiki wiki]# getenforce 
Disabled
[root@wiki wiki]# 
~~~


### SELINUX

如果开启了 SELINUX

则通过类似以下的操作来调整目录的 context

~~~
[root@wiki www]# getenforce 
Enforcing
[root@wiki www]# ls -lZ /var/www/
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_content_t:s0 html
lrwxrwxrwx. root   root   unconfined_u:object_r:httpd_sys_content_t:s0 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. apache apache unconfined_u:object_r:httpd_sys_content_t:s0 mediawiki-1.30.0
[root@wiki www]# restorecon -FR /var/www/mediawiki-1.30.0/
[root@wiki www]# ls -lZ /var/www/
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_content_t:s0 html
lrwxrwxrwx. root   root   unconfined_u:object_r:httpd_sys_content_t:s0 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. apache apache system_u:object_r:httpd_sys_content_t:s0 mediawiki-1.30.0
[root@wiki www]# restorecon -FR /var/www/mediawiki
[root@wiki www]# ls -lZ /var/www/
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_content_t:s0 html
lrwxrwxrwx. root   root   system_u:object_r:httpd_sys_content_t:s0 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. apache apache system_u:object_r:httpd_sys_content_t:s0 mediawiki-1.30.0
[root@wiki www]# 
~~~

## 进行访问

浏览器中打开 http://192.168.56.214/mediawiki/

![mediawiki](/assets/img/mediawiki/mediawiki01.png)

## 配置 wiki

首先选择语言

![mediawiki](/assets/img/mediawiki/mediawiki02.png)

进行安装前检查

![mediawiki](/assets/img/mediawiki/mediawiki03.png)

选择数据库

配置数据库连接信息

![mediawiki](/assets/img/mediawiki/mediawiki04.png)

选择存储引擎

![mediawiki](/assets/img/mediawiki/mediawiki05.png)

创建管理账号

![mediawiki](/assets/img/mediawiki/mediawiki06.png)

安装前的确认

![mediawiki](/assets/img/mediawiki/mediawiki07.png)

安装过程

![mediawiki](/assets/img/mediawiki/mediawiki08.png)

生成配置文件

![mediawiki](/assets/img/mediawiki/mediawiki09.png)

这个配置文件需要下载，并且放到正确的位置

~~~
wilmos@Nothing:~/download$ scp LocalSettings.php root@h214:/tmp/
LocalSettings.php                             100% 4369     4.3KB/s   00:00    
wilmos@Nothing:~/download$ 
~~~

放到 mediawiki 的根目录中

~~~
[root@wiki ~]# cd /var/www/html/mediawiki/
[root@wiki mediawiki]# ls
api.php                     images               README
autoload.php                img_auth.php         RELEASE-NOTES-1.30
cache                       includes             resources
CODE_OF_CONDUCT.md          index.php            serialized
composer.json               INSTALL              skins
composer.local.json-sample  jsduck.json          StartProfiler.sample
COPYING                     languages            tests
CREDITS                     load.php             thumb_handler.php
docs                        maintenance          thumb.php
extensions                  mw-config            UPGRADE
FAQ                         opensearch_desc.php  vendor
Gruntfile.js                phpcs.xml
HISTORY                     profileinfo.php
[root@wiki mediawiki]# cp /tmp/LocalSettings.php .
[root@wiki mediawiki]# ll LocalSettings.php 
-rw-r--r-- 1 root root 4369 4月  15 11:03 LocalSettings.php
[root@wiki mediawiki]#
[root@wiki mediawiki]# grep '^[^# ]' LocalSettings.php 
<?php
if ( !defined( 'MEDIAWIKI' ) ) {
	exit;
}
$wgSitename = "wiki_doc";
$wgMetaNamespace = "Wiki_doc";
$wgScriptPath = "/mediawiki";
$wgServer = "http://192.168.56.214";
$wgResourceBasePath = $wgScriptPath;
$wgLogo = "$wgResourceBasePath/resources/assets/wiki.png";
$wgEnableEmail = true;
$wgEnableUserEmail = true; # UPO
$wgEmergencyContact = "apache@192.168.56.214";
$wgPasswordSender = "apache@192.168.56.214";
$wgEnotifUserTalk = false; # UPO
$wgEnotifWatchlist = false; # UPO
$wgEmailAuthentication = true;
$wgDBtype = "mysql";
$wgDBserver = "localhost";
$wgDBname = "wiki_db";
$wgDBuser = "wiki";
$wgDBpassword = "wiki";
$wgDBprefix = "wiki_";
$wgDBTableOptions = "ENGINE=InnoDB, DEFAULT CHARSET=utf8";
$wgDBmysql5 = false;
$wgMainCacheType = CACHE_NONE;
$wgMemCachedServers = [];
$wgEnableUploads = false;
$wgUseInstantCommons = false;
$wgPingback = true;
$wgShellLocale = "en_US.utf8";
$wgLanguageCode = "zh-hans";
$wgSecretKey = "52280a1ee8f1b2455264059e26e4218b1796060920ef9a7fa5db2994e4a0186d";
$wgAuthenticationTokenVersion = "1";
$wgUpgradeKey = "4520946ac1a9e321";
$wgRightsPage = ""; # Set to the title of a wiki page that describes your license/copyright
$wgRightsUrl = "";
$wgRightsText = "";
$wgRightsIcon = "";
$wgDiff3 = "/usr/bin/diff3";
$wgDefaultSkin = "vector";
wfLoadSkin( 'CologneBlue' );
wfLoadSkin( 'Modern' );
wfLoadSkin( 'MonoBook' );
wfLoadSkin( 'Vector' );
[root@wiki mediawiki]# 
~~~

## 进入 wiki

![mediawiki](/assets/img/mediawiki/mediawiki10.png)

登录后右上角就会有提示

![mediawiki](/assets/img/mediawiki/mediawiki11.png)

编辑与预览

![mediawiki](/assets/img/mediawiki/mediawiki12.png)

从 phpMyAdmin 中可以看到表结构

![mediawiki](/assets/img/mediawiki/mediawiki13.png)

详细的编辑技巧可以参考 **[编辑页面][mediawiki_edit]**

---

# 总结

MediaWiki 是一个经典的 LAMP 应用 

* TOC
{:toc}

---

[mediawiki]:https://www.mediawiki.org/wiki/MediaWiki
[mediawiki_install]:https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Red_Hat_Linux
[mediawiki_install_guide]:https://www.mediawiki.org/wiki/Manual:Installation_guide
[mediawiki_install_brif]:https://www.mediawiki.org/wiki/Manual:Installing_MediaWiki
[mediawiki_dep]:https://www.mediawiki.org/wiki/Manual:Installation_requirements
[mediawiki_edit]:https://zh.wikipedia.org/wiki/Help:%E7%BC%96%E8%BE%91%E9%A1%B5%E9%9D%A2#%E5%BC%80%E5%A7%8B%E7%BC%96%E8%BE%91