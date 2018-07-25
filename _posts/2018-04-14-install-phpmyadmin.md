---
layout: post
title: "Install phpMyAdmin"
author:  wilmosfang
date: 2018-04-14 16:17:09
image: '/assets/img/'
excerpt: 'phpMyAdmin 的安装配置方法'
main-class:  'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
 - phpmyadmin
categories:
 - php
twitter_text: 'simple process of phpMyAdmin installation'
introduction: 'Installation of phpMyAdmin'
---

# 前言

**[phpMyAdmin][phpmyadmin]** 是一个 php 写的 web 版 mysql 管理工具

>phpMyAdmin is a free software tool written in PHP, intended to handle the administration of MySQL over the Web. phpMyAdmin supports a wide range of operations on MySQL and MariaDB. Frequently used operations (managing databases, tables, columns, relations, indexes, users, permissions, etc) can be performed via the user interface, while you still have the ability to directly execute any SQL statement

**[phpMyAdmin][phpmyadmin]** 可以完成常见的 mysql 管理工作

这里演示一下如何构建 **[phpMyAdmin][lamp_install]** 的过程

参考 **[phpMyAdmin文档][phpmyadmin_doc]** 和 **[How to install Apache, PHP 7.2 and MySQL on CentOS 7.4 (LAMP)][lamp_install]**

> **Tip:** 当前的最新版为 **phpMyAdmin 4.8.0**

---

# 操作

## OS 环境

~~~bash
[root@phpmyadmin ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@phpmyadmin ~]# hostnamectl 
   Static hostname: phpmyadmin
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: 0d7d6055ed704fb6a02a08745c3e5e91
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@phpmyadmin ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:8c:97:19 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84865sec preferred_lft 84865sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ab:2c:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.213/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@phpmyadmin ~]# 
~~~

## 软件环境

~~~bash
[root@phpmyadmin ~]# systemctl status mariadb.service
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-04-15 02:40:09 EDT; 43min ago
  Process: 1228 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 1164 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 1227 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─1227 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─1456 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log...

4月 15 02:40:06 phpmyadmin systemd[1]: Starting MariaDB database server...
4月 15 02:40:07 phpmyadmin mariadb-prepare-db-dir[1164]: Database MariaDB is probably initialized in /var/lib/mysql...one.
4月 15 02:40:07 phpmyadmin mariadb-prepare-db-dir[1164]: If this is not the case, make sure the /var/lib/mysql is e...dir.
4月 15 02:40:07 phpmyadmin mysqld_safe[1227]: 180415 02:40:07 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
4月 15 02:40:07 phpmyadmin mysqld_safe[1227]: 180415 02:40:07 mysqld_safe Starting mysqld daemon with databases fr...mysql
4月 15 02:40:09 phpmyadmin systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@phpmyadmin ~]# systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-04-15 02:40:07 EDT; 43min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1181 (httpd)
   Status: "Total requests: 11; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1181 /usr/sbin/httpd -DFOREGROUND
           ├─1414 /usr/sbin/httpd -DFOREGROUND
           ├─1415 /usr/sbin/httpd -DFOREGROUND
           ├─1416 /usr/sbin/httpd -DFOREGROUND
           ├─1417 /usr/sbin/httpd -DFOREGROUND
           ├─1418 /usr/sbin/httpd -DFOREGROUND
           ├─3305 /usr/sbin/httpd -DFOREGROUND
           ├─3306 /usr/sbin/httpd -DFOREGROUND
           └─3307 /usr/sbin/httpd -DFOREGROUND

4月 15 02:40:06 phpmyadmin systemd[1]: Starting The Apache HTTP Server...
4月 15 02:40:07 phpmyadmin httpd[1181]: AH00558: httpd: Could not reliably determine the server's fully qualified...essage
4月 15 02:40:07 phpmyadmin systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@phpmyadmin ~]#
[root@phpmyadmin ~]# php --version
PHP 7.2.4 (cli) (built: Mar 27 2018 17:23:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.4, Copyright (c) 1999-2018, by Zend Technologies
[root@phpmyadmin ~]#
~~~

其运行环境需要一个完整的 LAMP 环境，准确来讲，是一个 LAP 环境，而其中的 M(mysql) 就是被管理的对象

## 安装软件包

~~~
[root@phpmyadmin ~]#  rpm -qa | grep epel
epel-release-7-9.noarch
[root@phpmyadmin ~]# yum list all | grep phpMyAdmin
phpMyAdmin.noarch                        4.4.15.10-2.el7              epel      
[root@phpmyadmin ~]# yum install phpMyAdmin.noarch
Loaded plugins: fastestmirror, langpacks
base                                                                                                | 3.6 kB  00:00:00     
epel/x86_64/metalink                                                                                | 7.8 kB  00:00:00     
extras                                                                                              | 3.4 kB  00:00:00     
remi-php72                                                                                          | 2.9 kB  00:00:00     
remi-safe                                                                                           | 2.9 kB  00:00:00     
updates                                                                                             | 3.4 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: ftp.riken.jp
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package phpMyAdmin.noarch 0:4.4.15.10-2.el7 will be installed
--> Processing Dependency: php-gd >= 5.3.7 for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-mbstring >= 5.3.7 for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-php-gettext for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-simplexml for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-tcpdf for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-tcpdf-dejavu-sans-fonts for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-xmlwriter for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Processing Dependency: php-zip for package: phpMyAdmin-4.4.15.10-2.el7.noarch
--> Running transaction check
---> Package php-gd.x86_64 0:7.2.4-1.el7.remi will be installed
--> Processing Dependency: gd-last(x86-64) >= 2.1.1 for package: php-gd-7.2.4-1.el7.remi.x86_64
--> Processing Dependency: libgd.so.3()(64bit) for package: php-gd-7.2.4-1.el7.remi.x86_64
---> Package php-mbstring.x86_64 0:7.2.4-1.el7.remi will be installed
---> Package php-pecl-zip.x86_64 0:1.15.2-1.el7.remi.7.2 will be installed
--> Processing Dependency: libzip5(x86-64) >= 1.3.2 for package: php-pecl-zip-1.15.2-1.el7.remi.7.2.x86_64
--> Processing Dependency: libzip.so.5()(64bit) for package: php-pecl-zip-1.15.2-1.el7.remi.7.2.x86_64
---> Package php-php-gettext.noarch 0:1.0.12-1.el7 will be installed
---> Package php-tcpdf.noarch 0:6.2.13-1.el7 will be installed
--> Processing Dependency: php-bcmath for package: php-tcpdf-6.2.13-1.el7.noarch
--> Processing Dependency: php-composer(fedora/autoloader) for package: php-tcpdf-6.2.13-1.el7.noarch
--> Processing Dependency: php-posix for package: php-tcpdf-6.2.13-1.el7.noarch
--> Processing Dependency: php-tidy for package: php-tcpdf-6.2.13-1.el7.noarch
---> Package php-tcpdf-dejavu-sans-fonts.noarch 0:6.2.13-1.el7 will be installed
---> Package php-xml.x86_64 0:7.2.4-1.el7.remi will be installed
--> Running transaction check
---> Package gd-last.x86_64 0:2.2.5-2.el7.remi will be installed
---> Package libzip5.x86_64 0:1.5.1-1.el7.remi will be installed
---> Package php-bcmath.x86_64 0:7.2.4-1.el7.remi will be installed
---> Package php-fedora-autoloader.noarch 0:1.0.0-1.el7 will be installed
---> Package php-process.x86_64 0:7.2.4-1.el7.remi will be installed
---> Package php-tidy.x86_64 0:7.2.4-1.el7.remi will be installed
--> Processing Dependency: libtidy.so.5()(64bit) for package: php-tidy-7.2.4-1.el7.remi.x86_64
--> Running transaction check
---> Package libtidy.x86_64 0:5.4.0-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================================
 Package                                 Arch               Version                           Repository              Size
===========================================================================================================================
Installing:
 phpMyAdmin                              noarch             4.4.15.10-2.el7                   epel                   4.7 M
Installing for dependencies:
 gd-last                                 x86_64             2.2.5-2.el7.remi                  remi-safe              133 k
 libtidy                                 x86_64             5.4.0-1.el7                       epel                   174 k
 libzip5                                 x86_64             1.5.1-1.el7.remi                  remi-safe               55 k
 php-bcmath                              x86_64             7.2.4-1.el7.remi                  remi-php72              68 k
 php-fedora-autoloader                   noarch             1.0.0-1.el7                       epel                   9.6 k
 php-gd                                  x86_64             7.2.4-1.el7.remi                  remi-php72              75 k
 php-mbstring                            x86_64             7.2.4-1.el7.remi                  remi-php72             620 k
 php-pecl-zip                            x86_64             1.15.2-1.el7.remi.7.2             remi-php72              50 k
 php-php-gettext                         noarch             1.0.12-1.el7                      epel                    23 k
 php-process                             x86_64             7.2.4-1.el7.remi                  remi-php72              77 k
 php-tcpdf                               noarch             6.2.13-1.el7                      epel                   2.1 M
 php-tcpdf-dejavu-sans-fonts             noarch             6.2.13-1.el7                      epel                   257 k
 php-tidy                                x86_64             7.2.4-1.el7.remi                  remi-php72              61 k
 php-xml                                 x86_64             7.2.4-1.el7.remi                  remi-php72             203 k

Transaction Summary
===========================================================================================================================
Install  1 Package (+14 Dependent packages)

Total download size: 8.5 M
Installed size: 42 M
Is this ok [y/d/N]: y
Downloading packages:
(1/15): php-fedora-autoloader-1.0.0-1.el7.noarch.rpm                                                | 9.6 kB  00:00:00     
(2/15): libtidy-5.4.0-1.el7.x86_64.rpm                                                              | 174 kB  00:00:00     
(3/15): libzip5-1.5.1-1.el7.remi.x86_64.rpm                                                         |  55 kB  00:00:00     
(4/15): php-bcmath-7.2.4-1.el7.remi.x86_64.rpm                                                      |  68 kB  00:00:00     
(5/15): php-php-gettext-1.0.12-1.el7.noarch.rpm                                                     |  23 kB  00:00:00     
(6/15): php-gd-7.2.4-1.el7.remi.x86_64.rpm                                                          |  75 kB  00:00:00     
(7/15): gd-last-2.2.5-2.el7.remi.x86_64.rpm                                                         | 133 kB  00:00:01     
(8/15): php-process-7.2.4-1.el7.remi.x86_64.rpm                                                     |  77 kB  00:00:00     
(9/15): php-tcpdf-6.2.13-1.el7.noarch.rpm                                                           | 2.1 MB  00:00:00     
(10/15): php-tidy-7.2.4-1.el7.remi.x86_64.rpm                                                       |  61 kB  00:00:00     
(11/15): php-pecl-zip-1.15.2-1.el7.remi.7.2.x86_64.rpm                                              |  50 kB  00:00:01     
(12/15): php-xml-7.2.4-1.el7.remi.x86_64.rpm                                                        | 203 kB  00:00:01     
(13/15): phpMyAdmin-4.4.15.10-2.el7.noarch.rpm                                                      | 4.7 MB  00:00:01     
(14/15): php-tcpdf-dejavu-sans-fonts-6.2.13-1.el7.noarch.rpm                                        | 257 kB  00:00:02     
(15/15): php-mbstring-7.2.4-1.el7.remi.x86_64.rpm                                                   | 620 kB  00:00:03     
---------------------------------------------------------------------------------------------------------------------------
Total                                                                                      1.9 MB/s | 8.5 MB  00:00:04     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : php-mbstring-7.2.4-1.el7.remi.x86_64                                                                   1/15 
  Installing : php-xml-7.2.4-1.el7.remi.x86_64                                                                        2/15 
  Installing : php-php-gettext-1.0.12-1.el7.noarch                                                                    3/15 
  Installing : libtidy-5.4.0-1.el7.x86_64                                                                             4/15 
  Installing : php-tidy-7.2.4-1.el7.remi.x86_64                                                                       5/15 
  Installing : php-process-7.2.4-1.el7.remi.x86_64                                                                    6/15 
  Installing : gd-last-2.2.5-2.el7.remi.x86_64                                                                        7/15 
  Installing : php-gd-7.2.4-1.el7.remi.x86_64                                                                         8/15 
  Installing : libzip5-1.5.1-1.el7.remi.x86_64                                                                        9/15 
  Installing : php-pecl-zip-1.15.2-1.el7.remi.7.2.x86_64                                                             10/15 
  Installing : php-fedora-autoloader-1.0.0-1.el7.noarch                                                              11/15 
  Installing : php-bcmath-7.2.4-1.el7.remi.x86_64                                                                    12/15 
  Installing : php-tcpdf-6.2.13-1.el7.noarch                                                                         13/15 
  Installing : php-tcpdf-dejavu-sans-fonts-6.2.13-1.el7.noarch                                                       14/15 
  Installing : phpMyAdmin-4.4.15.10-2.el7.noarch                                                                     15/15 
  Verifying  : php-tcpdf-6.2.13-1.el7.noarch                                                                          1/15 
  Verifying  : php-xml-7.2.4-1.el7.remi.x86_64                                                                        2/15 
  Verifying  : php-bcmath-7.2.4-1.el7.remi.x86_64                                                                     3/15 
  Verifying  : phpMyAdmin-4.4.15.10-2.el7.noarch                                                                      4/15 
  Verifying  : php-pecl-zip-1.15.2-1.el7.remi.7.2.x86_64                                                              5/15 
  Verifying  : php-gd-7.2.4-1.el7.remi.x86_64                                                                         6/15 
  Verifying  : php-tidy-7.2.4-1.el7.remi.x86_64                                                                       7/15 
  Verifying  : php-tcpdf-dejavu-sans-fonts-6.2.13-1.el7.noarch                                                        8/15 
  Verifying  : php-fedora-autoloader-1.0.0-1.el7.noarch                                                               9/15 
  Verifying  : php-mbstring-7.2.4-1.el7.remi.x86_64                                                                  10/15 
  Verifying  : libzip5-1.5.1-1.el7.remi.x86_64                                                                       11/15 
  Verifying  : gd-last-2.2.5-2.el7.remi.x86_64                                                                       12/15 
  Verifying  : php-process-7.2.4-1.el7.remi.x86_64                                                                   13/15 
  Verifying  : php-php-gettext-1.0.12-1.el7.noarch                                                                   14/15 
  Verifying  : libtidy-5.4.0-1.el7.x86_64                                                                            15/15 

Installed:
  phpMyAdmin.noarch 0:4.4.15.10-2.el7                                                                                      

Dependency Installed:
  gd-last.x86_64 0:2.2.5-2.el7.remi                        libtidy.x86_64 0:5.4.0-1.el7                                   
  libzip5.x86_64 0:1.5.1-1.el7.remi                        php-bcmath.x86_64 0:7.2.4-1.el7.remi                           
  php-fedora-autoloader.noarch 0:1.0.0-1.el7               php-gd.x86_64 0:7.2.4-1.el7.remi                               
  php-mbstring.x86_64 0:7.2.4-1.el7.remi                   php-pecl-zip.x86_64 0:1.15.2-1.el7.remi.7.2                    
  php-php-gettext.noarch 0:1.0.12-1.el7                    php-process.x86_64 0:7.2.4-1.el7.remi                          
  php-tcpdf.noarch 0:6.2.13-1.el7                          php-tcpdf-dejavu-sans-fonts.noarch 0:6.2.13-1.el7              
  php-tidy.x86_64 0:7.2.4-1.el7.remi                       php-xml.x86_64 0:7.2.4-1.el7.remi                              

Complete!
[root@phpmyadmin ~]# 
~~~

## 修改 Apache 的配置

放开连接限制

~~~
[root@phpmyadmin ~]# rpm -ql  phpMyAdmin.noarch | grep etc
/etc/httpd/conf.d/phpMyAdmin.conf
/etc/phpMyAdmin
/etc/phpMyAdmin/config.inc.php
[root@phpmyadmin ~]# vim /etc/httpd/conf.d/phpMyAdmin.conf 
[root@phpmyadmin ~]# head -n 20 /etc/httpd/conf.d/phpMyAdmin.conf
# phpMyAdmin - Web based MySQL browser written in php
# 
# Allows only localhost by default
#
# But allowing phpMyAdmin to anyone other than localhost should be considered
# dangerous unless properly secured by SSL

Alias /phpMyAdmin /usr/share/phpMyAdmin
Alias /phpmyadmin /usr/share/phpMyAdmin

<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8

   <IfModule mod_authz_core.c>
     # Apache 2.4
#     <RequireAny>
#       Require ip 127.0.0.1
#       Require ip ::1
#     </RequireAny>
     Require all granted
[root@phpmyadmin ~]# 
[root@phpmyadmin ~]# grep -v "#" /etc/httpd/conf.d/phpMyAdmin.conf

Alias /phpMyAdmin /usr/share/phpMyAdmin
Alias /phpmyadmin /usr/share/phpMyAdmin

<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8

   <IfModule mod_authz_core.c>
     Require all granted
   </IfModule>
   <IfModule !mod_authz_core.c>
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>

<Directory /usr/share/phpMyAdmin/setup/>
   <IfModule mod_authz_core.c>
     <RequireAny>
       Require ip 127.0.0.1
       Require ip ::1
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>

<Directory /usr/share/phpMyAdmin/libraries/>
    Order Deny,Allow
    Deny from All
    Allow from None
</Directory>

<Directory /usr/share/phpMyAdmin/setup/lib/>
    Order Deny,Allow
    Deny from All
    Allow from None
</Directory>

<Directory /usr/share/phpMyAdmin/setup/frames/>
    Order Deny,Allow
    Deny from All
    Allow from None
</Directory>

[root@phpmyadmin ~]# 
~~~

## 修改 php 的配置

~~~
[root@phpmyadmin ~]# vim /etc/phpMyAdmin/config.inc.php
[root@phpmyadmin ~]# grep '^\$' /etc/phpMyAdmin/config.inc.php
$cfg['blowfish_secret'] = 'cuMQvacGFKf4aAjlebR1STNfJNgCreot'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
$i = 0;
$i++;
$cfg['Servers'][$i]['host']          = 'localhost'; // MySQL hostname or IP address
$cfg['Servers'][$i]['port']          = '';          // MySQL port - leave blank for default port
$cfg['Servers'][$i]['socket']        = '';          // Path to the socket - leave blank for default socket
$cfg['Servers'][$i]['connect_type']  = 'tcp';       // How to connect to MySQL server ('tcp' or 'socket')
$cfg['Servers'][$i]['extension']     = 'mysqli';    // The php MySQL extension to use ('mysql' or 'mysqli')
$cfg['Servers'][$i]['compress']      = FALSE;       // Use compressed protocol for the MySQL connection
$cfg['Servers'][$i]['controluser']   = '';          // MySQL control user settings
$cfg['Servers'][$i]['controlpass']   = '';          // access to the "mysql/user"
$cfg['Servers'][$i]['auth_type']     = 'http';    // Authentication method (config, http or cookie based)?
$cfg['Servers'][$i]['user']          = '';          // MySQL user
$cfg['Servers'][$i]['password']      = '';          // MySQL password (only needed
$cfg['Servers'][$i]['only_db']       = '';          // If set to a db-name, only
$cfg['Servers'][$i]['hide_db']       = '';          // Database name to be hidden from listings
$cfg['Servers'][$i]['verbose']       = '';          // Verbose name for this host - leave blank to show the hostname
$cfg['Servers'][$i]['pmadb']         = '';          // Database used for Relation, Bookmark and PDF Features
$cfg['Servers'][$i]['bookmarktable'] = '';          // Bookmark table
$cfg['Servers'][$i]['relation']      = '';          // table to describe the relation between links (see doc)
$cfg['Servers'][$i]['table_info']    = '';          // table to describe the display fields
$cfg['Servers'][$i]['table_coords']  = '';          // table to describe the tables position for the PDF schema
$cfg['Servers'][$i]['pdf_pages']     = '';          // table to describe pages of relationpdf
$cfg['Servers'][$i]['column_info']   = '';          // table to store column information
$cfg['Servers'][$i]['history']       = '';          // table to store SQL history
$cfg['Servers'][$i]['verbose_check'] = TRUE;        // set to FALSE if you know that your pma_* tables
$cfg['Servers'][$i]['AllowRoot']     = TRUE;        // whether to allow root login
$cfg['Servers'][$i]['AllowDeny']['order']           // Host authentication order, leave blank to not use
$cfg['Servers'][$i]['AllowDeny']['rules']           // Host authentication rules, leave blank for defaults
$cfg['Servers'][$i]['AllowNoPassword']              // Allow logins without a password. Do not change the FALSE
$cfg['Servers'][$i]['designer_coords']              // Leave blank (default) for no Designer support, otherwise
$cfg['Servers'][$i]['bs_garbage_threshold']         // Blobstreaming: Recommented default value from upstream
$cfg['Servers'][$i]['bs_repository_threshold']      // Blobstreaming: Recommented default value from upstream
$cfg['Servers'][$i]['bs_temp_blob_timeout']         // Blobstreaming: Recommented default value from upstream
$cfg['Servers'][$i]['bs_temp_log_threshold']        // Blobstreaming: Recommented default value from upstream
$cfg['UploadDir'] = '/var/lib/phpMyAdmin/upload';
$cfg['SaveDir']   = '/var/lib/phpMyAdmin/save';
$cfg['PmaNoRelation_DisableWarning'] = TRUE;
[root@phpmyadmin ~]# 
~~~

## 进行访问

登录 http://192.168.56.213/phpmyadmin

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin01.png)

输入数据库正确的密码后，第一个展示的界面

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin02.png)

可以选择使用的语言

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin03.png)

可以在这个界面中直接输入和执行 SQL 语句　(界面还是非常友好的，有语句提醒)

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin04.png)

可以在这里管理用户

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin05.png)

## 状态信息

这个状态栏里，可以获得很多有用的信息

通过建议栏可以获取一些优化方向

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin06.png)

通过监控栏可以获取一些实时的性能参数

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin08.png)

通过查询统计可以查看到各语句的使用占比

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin09.png)

还有当前的所有变量值

![phpmyadmin](/assets/img/phpmyadmin/phpmyadmin10.png)


通过状态功能，可以对 mysql 完成基本的监控与调优

---


# 总结

这是一个经典的 LAMP 结构，通过 phpMyAdmin 可以覆盖到 mysql 日常的大部分管理操作

相对于命令行更加友好和简单

个人觉得，状态功能非常好，给出了一些优化建议，还给出了优化建议依据的计算方法

* TOC
{:toc}

---

[phpmyadmin]:https://www.phpmyadmin.net/
[phpmyadmin_doc]:https://docs.phpmyadmin.net/zh_CN/latest/index.html
[lamp_install]:https://www.howtoforge.com/tutorial/centos-lamp-server-apache-mysql-php/
