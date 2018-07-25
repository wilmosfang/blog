---
layout: post
title: "Install LAMP"
author:  wilmosfang
date: 2018-04-13 16:21:38
image: '/assets/img/'
excerpt: 'LAMP(Linux+Apache+Mysql+php) 的安装配置方法'
main-class:  'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
categories:
 - php
twitter_text: 'simple process of LAMP configuration'
introduction: 'Configuration of LAMP'
---

# 前言

**[LAMP][lamp]** 是一组开源软件拼接成的 **Web** 技术栈

由 **Linux** 提供运行环境，**Mysql** 提供关系型存储，**Apache** 提供 Web 服务，**PHP** 完成应用逻辑

>LAMP is an archetypal model of web service stacks, named as an acronym of the names of its original four open-source components: the Linux operating system, the Apache HTTP Server, the MySQL relational database management system (RDBMS), and the PHP programming language. The LAMP components are largely interchangeable and not limited to the original selection. As a solution stack, LAMP is suitable for building dynamic web sites and web applications

下图可以比较直观地看到各模块的层级和相互之间的关系

![lamp](/assets/img/lamp/lamp_software_bundle.png)

**[LAMP][lamp]** 为廉价而快速地构建一整套 Web 应用提供了一种可能，于是在互联网初期形成了一股风潮，事实上此技术栈的余热致今也未散去，很多流行的应用依旧沿用此架构

>个人愚见，这也是 Mysql(和Mysql系) 如今为何那么风靡的一个重要原因，以我的视角，Postgresql 是一个比 Mysql 更为高级，拥有更多功能特性的数据库软件，但是由于早期的 Mysql 因为简单而被广泛接受，并且经年累月逐步形成了自己的生态圈，于是这就成为了目前其排名与地位无法轻易被撼动的最主要原因

LAMP 的 **变体**

技术栈的不同层面进行替换就可以形成不同的变体

* LAPP（以PostgreSQL替代MySQL）
* LAMP（最后两个字母意味着Middleware和PostgreSQL）
* LNMP或LEMP（以Nginx替代Apache）
* WAMP（以Microsoft Windows替代Linux）
* MAMP（以Macintosh替代Linux）
* LAMJ（以JSP/servlet替代PHP）
* BAMP（以BSD替代Linux）
* WIMP（指Microsoft Windows, Microsoft IIS，MySQL, PHP）
* AMP（单指Apache, MySQL和PHP）
* XAMP（以XML替代Linux）

回归正题，这里演示一下如何构建 **[LAMP][lamp_install]** 的过程

参考 **[How to install Apache, PHP 7.2 and MySQL on CentOS 7.4 (LAMP)][lamp_install]**

> **Tip:** 当前的系统环境为 **CentOS 7.4**

---

# 操作

## 环境

~~~
[root@lamp ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core) 
[root@lamp ~]# hostnamectl 
   Static hostname: lamp
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 316348df30744c9c91b9202baf3915a6
           Boot ID: aa43c7a9ee774322b705837ba281b11d
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.21.1.el7.x86_64
      Architecture: x86-64
[root@lamp ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:16:d7:4f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85098sec preferred_lft 85098sec
    inet6 fe80::334c:bc63:1266:56b3/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:6c:7c:2c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.212/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::2be7:a317:cc4b:666b/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:14:54:5c brd ff:ff:ff:ff:ff:ff
[root@lamp ~]# 
~~~

## 安装 epel 仓库

~~~
[root@lamp ~]# ll /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-*
-rw-r--r--. 1 root root 1690 8月  30 2017 /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
-rw-r--r--. 1 root root 1004 8月  30 2017 /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-Debug-7
-rw-r--r--. 1 root root 1690 8月  30 2017 /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-Testing-7
[root@lamp ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY*
[root@lamp ~]# yum -y install epel-release
Loaded plugins: fastestmirror, langpacks
base                                                                                | 3.6 kB  00:00:00     
extras                                                                              | 3.4 kB  00:00:00     
updates                                                                             | 3.4 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-9 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                       Arch                    Version               Repository               Size
===========================================================================================================
Installing:
 epel-release                  noarch                  7-9                   extras                   14 k

Transaction Summary
===========================================================================================================
Install  1 Package

Total download size: 14 k
Installed size: 24 k
Downloading packages:
epel-release-7-9.noarch.rpm                                                         |  14 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-9.noarch                                                                 1/1 
  Verifying  : epel-release-7-9.noarch                                                                 1/1 

Installed:
  epel-release.noarch 0:7-9                                                                                

Complete!
[root@lamp ~]# 
~~~

## 安装 mysql

~~~
[root@lamp ~]# yum -y install mariadb-server mariadb
Loaded plugins: fastestmirror, langpacks
epel/x86_64/metalink                                                                | 7.8 kB  00:00:00     
epel                                                                                | 4.7 kB  00:00:00     
(1/3): epel/x86_64/group_gz                                                         | 266 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                       | 908 kB  00:00:00     
(3/3): epel/x86_64/primary_db                                                       | 6.3 MB  00:00:26     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.vinahost.vn
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package mariadb.x86_64 1:5.5.56-2.el7 will be installed
---> Package mariadb-server.x86_64 1:5.5.56-2.el7 will be installed
--> Processing Dependency: perl-DBD-MySQL for package: 1:mariadb-server-5.5.56-2.el7.x86_64
--> Running transaction check
---> Package perl-DBD-MySQL.x86_64 0:4.023-5.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                      Arch                 Version                        Repository          Size
===========================================================================================================
Installing:
 mariadb                      x86_64               1:5.5.56-2.el7                 base               8.7 M
 mariadb-server               x86_64               1:5.5.56-2.el7                 base                11 M
Installing for dependencies:
 perl-DBD-MySQL               x86_64               4.023-5.el7                    base               140 k

Transaction Summary
===========================================================================================================
Install  2 Packages (+1 Dependent package)

Total download size: 20 M
Installed size: 107 M
Downloading packages:
(1/3): perl-DBD-MySQL-4.023-5.el7.x86_64.rpm                                        | 140 kB  00:00:00     
(2/3): mariadb-5.5.56-2.el7.x86_64.rpm                                              | 8.7 MB  00:00:05     
(3/3): mariadb-server-5.5.56-2.el7.x86_64.rpm                                       |  11 MB  00:00:08     
-----------------------------------------------------------------------------------------------------------
Total                                                                      2.2 MB/s |  20 MB  00:00:09     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:mariadb-5.5.56-2.el7.x86_64                                                           1/3 
  Installing : perl-DBD-MySQL-4.023-5.el7.x86_64                                                       2/3 
  Installing : 1:mariadb-server-5.5.56-2.el7.x86_64                                                    3/3 
  Verifying  : 1:mariadb-server-5.5.56-2.el7.x86_64                                                    1/3 
  Verifying  : perl-DBD-MySQL-4.023-5.el7.x86_64                                                       2/3 
  Verifying  : 1:mariadb-5.5.56-2.el7.x86_64                                                           3/3 

Installed:
  mariadb.x86_64 1:5.5.56-2.el7                    mariadb-server.x86_64 1:5.5.56-2.el7                   

Dependency Installed:
  perl-DBD-MySQL.x86_64 0:4.023-5.el7                                                                      

Complete!
[root@lamp ~]# 
~~~

## 启用 mysql

~~~
[root@lamp ~]# systemctl start mariadb.service
[root@lamp ~]# systemctl enable mariadb.service
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@lamp ~]# systemctl status mariadb.service
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2018-04-14 10:51:36 EDT; 11s ago
 Main PID: 2055 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─2055 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─2217 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/my...

4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: MySQL manual for more instructions.
4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: Please report any problems at http://mariadb.org/jira
4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: The latest information about MariaDB is available...g/.
4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: You can find additional information about the MyS...at:
4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: http://dev.mysql.com
4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: Consider joining MariaDB's strong and vibrant com...ty:
4月 14 10:51:34 lamp mariadb-prepare-db-dir[1977]: https://mariadb.org/get-involved/
4月 14 10:51:34 lamp mysqld_safe[2055]: 180414 10:51:34 mysqld_safe Logging to '/var/log/mariadb/ma...og'.
4月 14 10:51:34 lamp mysqld_safe[2055]: 180414 10:51:34 mysqld_safe Starting mysqld daemon with dat...ysql
4月 14 10:51:36 lamp systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@lamp ~]# 
~~~

## 配置 mysql

~~~
[root@lamp ~]# mysql_secure_installation

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

Set root password? [Y/n] Y
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
[root@lamp ~]# echo $?
0
[root@lamp ~]# 
~~~

## 安装 apache

~~~
[root@lamp ~]# yum -y install httpd
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.smartmedia.net.id
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.6-67.el7.centos.6 will be installed
--> Processing Dependency: httpd-tools = 2.4.6-67.el7.centos.6 for package: httpd-2.4.6-67.el7.centos.6.x86_64
--> Processing Dependency: /etc/mime.types for package: httpd-2.4.6-67.el7.centos.6.x86_64
--> Processing Dependency: libaprutil-1.so.0()(64bit) for package: httpd-2.4.6-67.el7.centos.6.x86_64
--> Processing Dependency: libapr-1.so.0()(64bit) for package: httpd-2.4.6-67.el7.centos.6.x86_64
--> Running transaction check
---> Package apr.x86_64 0:1.4.8-3.el7_4.1 will be installed
---> Package apr-util.x86_64 0:1.5.2-6.el7 will be installed
---> Package httpd-tools.x86_64 0:2.4.6-67.el7.centos.6 will be installed
---> Package mailcap.noarch 0:2.1.41-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                 Arch               Version                              Repository           Size
===========================================================================================================
Installing:
 httpd                   x86_64             2.4.6-67.el7.centos.6                updates             2.7 M
Installing for dependencies:
 apr                     x86_64             1.4.8-3.el7_4.1                      updates             103 k
 apr-util                x86_64             1.5.2-6.el7                          base                 92 k
 httpd-tools             x86_64             2.4.6-67.el7.centos.6                updates              88 k
 mailcap                 noarch             2.1.41-2.el7                         base                 31 k

Transaction Summary
===========================================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 3.0 M
Installed size: 10 M
Downloading packages:
(1/5): apr-util-1.5.2-6.el7.x86_64.rpm                                              |  92 kB  00:00:00     
(2/5): apr-1.4.8-3.el7_4.1.x86_64.rpm                                               | 103 kB  00:00:00     
(3/5): httpd-tools-2.4.6-67.el7.centos.6.x86_64.rpm                                 |  88 kB  00:00:00     
(4/5): mailcap-2.1.41-2.el7.noarch.rpm                                              |  31 kB  00:00:00     
(5/5): httpd-2.4.6-67.el7.centos.6.x86_64.rpm                                       | 2.7 MB  00:00:02     
-----------------------------------------------------------------------------------------------------------
Total                                                                      1.1 MB/s | 3.0 MB  00:00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : apr-1.4.8-3.el7_4.1.x86_64                                                              1/5 
  Installing : apr-util-1.5.2-6.el7.x86_64                                                             2/5 
  Installing : httpd-tools-2.4.6-67.el7.centos.6.x86_64                                                3/5 
  Installing : mailcap-2.1.41-2.el7.noarch                                                             4/5 
  Installing : httpd-2.4.6-67.el7.centos.6.x86_64                                                      5/5 
  Verifying  : mailcap-2.1.41-2.el7.noarch                                                             1/5 
  Verifying  : httpd-2.4.6-67.el7.centos.6.x86_64                                                      2/5 
  Verifying  : apr-util-1.5.2-6.el7.x86_64                                                             3/5 
  Verifying  : apr-1.4.8-3.el7_4.1.x86_64                                                              4/5 
  Verifying  : httpd-tools-2.4.6-67.el7.centos.6.x86_64                                                5/5 

Installed:
  httpd.x86_64 0:2.4.6-67.el7.centos.6                                                                     

Dependency Installed:
  apr.x86_64 0:1.4.8-3.el7_4.1   apr-util.x86_64 0:1.5.2-6.el7  httpd-tools.x86_64 0:2.4.6-67.el7.centos.6 
  mailcap.noarch 0:2.1.41-2.el7 

Complete!
[root@lamp ~]# 
~~~

## 启用 Apache

~~~
[root@lamp ~]# systemctl start httpd.service
[root@lamp ~]# systemctl enable httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@lamp ~]# systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2018-04-14 10:56:21 EDT; 11s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 2483 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─2483 /usr/sbin/httpd -DFOREGROUND
           ├─2484 /usr/sbin/httpd -DFOREGROUND
           ├─2485 /usr/sbin/httpd -DFOREGROUND
           ├─2486 /usr/sbin/httpd -DFOREGROUND
           ├─2487 /usr/sbin/httpd -DFOREGROUND
           └─2488 /usr/sbin/httpd -DFOREGROUND

4月 14 10:56:20 lamp systemd[1]: Starting The Apache HTTP Server...
4月 14 10:56:21 lamp httpd[2483]: AH00558: httpd: Could not reliably determine the server's fully ...ssage
4月 14 10:56:21 lamp systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@lamp ~]# 
~~~

## 打开防火墙

~~~
[root@lamp ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: ssh dhcpv6-client
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@lamp ~]# firewall-cmd --permanent --zone=public --add-service=http 
success
[root@lamp ~]# firewall-cmd --permanent --zone=public --add-service=https
success
[root@lamp ~]# firewall-cmd --reload
success
[root@lamp ~]# firewall-cmd --list-all
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
	
[root@lamp ~]# 
~~~

## 进行访问测试

http://192.168.56.212/

![lamp](/assets/img/lamp/lamp01.png)

出现了这个界面，说明 Apache 已经安装成功了

## 安装 remi 仓库

因为系统自带的 php 版本太低，所以要使用其它的软件仓库来提供软件包

~~~
[root@lamp ~]# yum list all | grep php.x86_64 
geos-php.x86_64                         3.4.2-2.el7                    epel     
graphviz-php.x86_64                     2.30.1-19.el7                  base     
mlt-php.x86_64                          6.4.1-4.el7                    epel     
php.x86_64                              5.4.16-43.el7_4.1              updates  
remctl-php.x86_64                       3.9-2.el7                      epel     
rrdtool-php.x86_64                      1.4.8-9.el7                    base     
uuid-php.x86_64                         1.6.2-26.el7                   base     
uwsgi-plugin-php.x86_64                 2.0.16-1.el7                   epel     
[root@lamp ~]# rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
Retrieving http://rpms.remirepo.net/enterprise/remi-release-7.rpm
warning: /var/tmp/rpm-tmp.mUqy4y: Header V4 DSA/SHA1 Signature, key ID 00f97f56: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:remi-release-7.4-2.el7.remi      ################################# [100%]
[root@lamp ~]#
~~~

可以看看安装了些什么东西

~~~
[root@lamp ~]# rpm -ql remi-release
/etc/pki/rpm-gpg/RPM-GPG-KEY-remi
/etc/pki/rpm-gpg/RPM-GPG-KEY-remi2017
/etc/pki/rpm-gpg/RPM-GPG-KEY-remi2018
/etc/yum.repos.d/remi-php54.repo
/etc/yum.repos.d/remi-php70.repo
/etc/yum.repos.d/remi-php71.repo
/etc/yum.repos.d/remi-php72.repo
/etc/yum.repos.d/remi-safe.repo
/etc/yum.repos.d/remi.repo
[root@lamp ~]#
[root@lamp ~]# ll /etc/yum.repos.d/
total 60
-rw-r--r--. 1 root root 1664 8月  30 2017 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 8月  30 2017 CentOS-CR.repo
-rw-r--r--. 1 root root  649 8月  30 2017 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 8月  30 2017 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 8月  30 2017 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 8月  30 2017 CentOS-Sources.repo
-rw-r--r--. 1 root root 3830 8月  30 2017 CentOS-Vault.repo
-rw-r--r--  1 root root  957 12月 27 2016 epel.repo
-rw-r--r--  1 root root 1056 12月 27 2016 epel-testing.repo
-rw-r--r--  1 root root  456 3月  21 09:28 remi-php54.repo
-rw-r--r--  1 root root 1314 3月  21 09:28 remi-php70.repo
-rw-r--r--  1 root root 1314 3月  21 09:28 remi-php71.repo
-rw-r--r--  1 root root 1314 3月  21 09:28 remi-php72.repo
-rw-r--r--  1 root root 2605 3月  21 09:28 remi.repo
-rw-r--r--  1 root root  750 3月  21 09:28 remi-safe.repo
[root@lamp ~]# ll /etc/yum.repos.d/ | grep remi
-rw-r--r--  1 root root  456 3月  21 09:28 remi-php54.repo
-rw-r--r--  1 root root 1314 3月  21 09:28 remi-php70.repo
-rw-r--r--  1 root root 1314 3月  21 09:28 remi-php71.repo
-rw-r--r--  1 root root 1314 3月  21 09:28 remi-php72.repo
-rw-r--r--  1 root root 2605 3月  21 09:28 remi.repo
-rw-r--r--  1 root root  750 3月  21 09:28 remi-safe.repo
[root@lamp ~]# 
~~~


## 启用指定仓库

这里我启用最新的 php 版本软件库

~~~
[root@lamp ~]# yum-config-manager --enable remi-php72
Loaded plugins: fastestmirror, langpacks
============================================ repo: remi-php72 =============================================
[remi-php72]
async = True
bandwidth = 0
base_persistdir = /var/lib/yum/repos/x86_64/7
baseurl = 
cache = 0
cachedir = /var/cache/yum/x86_64/7/remi-php72
check_config_file_age = True
compare_providers_priority = 80
cost = 1000
deltarpm_metadata_percentage = 100
deltarpm_percentage = 
enabled = 1
enablegroups = True
exclude = 
failovermethod = priority
ftp_disable_epsv = False
gpgcadir = /var/lib/yum/repos/x86_64/7/remi-php72/gpgcadir
gpgcakey = 
gpgcheck = True
gpgdir = /var/lib/yum/repos/x86_64/7/remi-php72/gpgdir
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
hdrdir = /var/cache/yum/x86_64/7/remi-php72/headers
http_caching = all
includepkgs = 
ip_resolve = 
keepalive = True
keepcache = False
mddownloadpolicy = sqlite
mdpolicy = group:small
mediaid = 
metadata_expire = 21600
metadata_expire_filter = read-only:present
metalink = 
minrate = 0
mirrorlist = http://cdn.remirepo.net/enterprise/7/php72/mirror
mirrorlist_expire = 86400
name = Remi's PHP 7.2 RPM repository for Enterprise Linux 7 - x86_64
old_base_cache_dir = 
password = 
persistdir = /var/lib/yum/repos/x86_64/7/remi-php72
pkgdir = /var/cache/yum/x86_64/7/remi-php72/packages
proxy = False
proxy_dict = 
proxy_password = 
proxy_username = 
repo_gpgcheck = False
retries = 10
skip_if_unavailable = False
ssl_check_cert_permissions = True
sslcacert = 
sslclientcert = 
sslclientkey = 
sslverify = True
throttle = 0
timeout = 30.0
ui_id = remi-php72
ui_repoid_vars = releasever,
   basearch
username = 

[root@lamp ~]# 
~~~

## 安装软件包

这里我安装最新的 php

~~~
[root@lamp ~]# yum -y install php php-opcache
Loaded plugins: fastestmirror, langpacks
remi-php72                                                                          | 2.9 kB  00:00:00     
remi-safe                                                                           | 2.9 kB  00:00:00     
(1/2): remi-php72/primary_db                                                        | 185 kB  00:00:03     
(2/2): remi-safe/primary_db                                                         | 1.2 MB  00:00:10     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.smartmedia.net.id
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package php.x86_64 0:7.2.4-1.el7.remi will be installed
--> Processing Dependency: php-common(x86-64) = 7.2.4-1.el7.remi for package: php-7.2.4-1.el7.remi.x86_64
--> Processing Dependency: php-cli(x86-64) = 7.2.4-1.el7.remi for package: php-7.2.4-1.el7.remi.x86_64
--> Processing Dependency: libargon2.so.0()(64bit) for package: php-7.2.4-1.el7.remi.x86_64
---> Package php-opcache.x86_64 0:7.2.4-1.el7.remi will be installed
--> Running transaction check
---> Package libargon2.x86_64 0:20161029-2.el7 will be installed
---> Package php-cli.x86_64 0:7.2.4-1.el7.remi will be installed
---> Package php-common.x86_64 0:7.2.4-1.el7.remi will be installed
--> Processing Dependency: php-json(x86-64) = 7.2.4-1.el7.remi for package: php-common-7.2.4-1.el7.remi.x86_64
--> Running transaction check
---> Package php-json.x86_64 0:7.2.4-1.el7.remi will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                  Arch                Version                        Repository               Size
===========================================================================================================
Installing:
 php                      x86_64              7.2.4-1.el7.remi               remi-php72              3.2 M
 php-opcache              x86_64              7.2.4-1.el7.remi               remi-php72              279 k
Installing for dependencies:
 libargon2                x86_64              20161029-2.el7                 epel                     23 k
 php-cli                  x86_64              7.2.4-1.el7.remi               remi-php72              4.8 M
 php-common               x86_64              7.2.4-1.el7.remi               remi-php72              1.1 M
 php-json                 x86_64              7.2.4-1.el7.remi               remi-php72               61 k

Transaction Summary
===========================================================================================================
Install  2 Packages (+4 Dependent packages)

Total download size: 9.4 M
Installed size: 38 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/epel/packages/libargon2-20161029-2.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for libargon2-20161029-2.el7.x86_64.rpm is not installed
(1/6): libargon2-20161029-2.el7.x86_64.rpm                                          |  23 kB  00:00:00     
warning: /var/cache/yum/x86_64/7/remi-php72/packages/php-opcache-7.2.4-1.el7.remi.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 00f97f56: NOKEY
Public key for php-opcache-7.2.4-1.el7.remi.x86_64.rpm is not installed
(2/6): php-opcache-7.2.4-1.el7.remi.x86_64.rpm                                      | 279 kB  00:00:01     
(3/6): php-json-7.2.4-1.el7.remi.x86_64.rpm                                         |  61 kB  00:00:02     
(4/6): php-cli-7.2.4-1.el7.remi.x86_64.rpm                                          | 4.8 MB  00:00:04     
(5/6): php-common-7.2.4-1.el7.remi.x86_64.rpm                                       | 1.1 MB  00:00:04     
(6/6): php-7.2.4-1.el7.remi.x86_64.rpm                                              | 3.2 MB  00:00:10     
-----------------------------------------------------------------------------------------------------------
Total                                                                      910 kB/s | 9.4 MB  00:00:10     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
Importing GPG key 0x00F97F56:
 Userid     : "Remi Collet <RPMS@FamilleCollet.com>"
 Fingerprint: 1ee0 4cce 88a4 ae4a a29a 5df5 004e 6f47 00f9 7f56
 Package    : remi-release-7.4-2.el7.remi.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : libargon2-20161029-2.el7.x86_64                                                         1/6 
  Installing : php-common-7.2.4-1.el7.remi.x86_64                                                      2/6 
  Installing : php-json-7.2.4-1.el7.remi.x86_64                                                        3/6 
  Installing : php-cli-7.2.4-1.el7.remi.x86_64                                                         4/6 
  Installing : php-7.2.4-1.el7.remi.x86_64                                                             5/6 
  Installing : php-opcache-7.2.4-1.el7.remi.x86_64                                                     6/6 
  Verifying  : php-cli-7.2.4-1.el7.remi.x86_64                                                         1/6 
  Verifying  : php-7.2.4-1.el7.remi.x86_64                                                             2/6 
  Verifying  : php-opcache-7.2.4-1.el7.remi.x86_64                                                     3/6 
  Verifying  : libargon2-20161029-2.el7.x86_64                                                         4/6 
  Verifying  : php-json-7.2.4-1.el7.remi.x86_64                                                        5/6 
  Verifying  : php-common-7.2.4-1.el7.remi.x86_64                                                      6/6 

Installed:
  php.x86_64 0:7.2.4-1.el7.remi                    php-opcache.x86_64 0:7.2.4-1.el7.remi                   

Dependency Installed:
  libargon2.x86_64 0:20161029-2.el7                     php-cli.x86_64 0:7.2.4-1.el7.remi                  
  php-common.x86_64 0:7.2.4-1.el7.remi                  php-json.x86_64 0:7.2.4-1.el7.remi                 

Complete!
[root@lamp ~]# 
~~~

## 测试 php 的安装

安装完 php 后重启 Apache 服务

然后构建一个测试页面

~~~
[root@lamp ~]# systemctl restart httpd.service
[root@lamp ~]# vim /var/www/html/info.php
[root@lamp ~]# cat /var/www/html/info.php
<?php
phpinfo();
[root@lamp ~]# 
~~~

访问 http://192.168.56.212/info.php

如果出现下面的页面，就代表安装正常

![lamp](/assets/img/lamp/lamp02.png)

这个页面里其实可以反馈很多信息，这些信息都是 php 运行时收集起来的



## 加入对 mysql 的支持

安装 php 的 mysql driver

~~~
[root@lamp ~]# yum -y install php-mysqlnd php-pdo
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.rise.ph
 * extras: mirror.pregi.net
 * remi-php72: mirrors.thzhost.com
 * remi-safe: mirrors.thzhost.com
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package php-mysqlnd.x86_64 0:7.2.4-1.el7.remi will be installed
---> Package php-pdo.x86_64 0:7.2.4-1.el7.remi will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                  Arch                Version                        Repository               Size
===========================================================================================================
Installing:
 php-mysqlnd              x86_64              7.2.4-1.el7.remi               remi-php72              230 k
 php-pdo                  x86_64              7.2.4-1.el7.remi               remi-php72              123 k

Transaction Summary
===========================================================================================================
Install  2 Packages

Total download size: 353 k
Installed size: 1.2 M
Downloading packages:
(1/2): php-pdo-7.2.4-1.el7.remi.x86_64.rpm                                          | 123 kB  00:00:01     
(2/2): php-mysqlnd-7.2.4-1.el7.remi.x86_64.rpm                                      | 230 kB  00:00:02     
-----------------------------------------------------------------------------------------------------------
Total                                                                      158 kB/s | 353 kB  00:00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : php-pdo-7.2.4-1.el7.remi.x86_64                                                         1/2 
  Installing : php-mysqlnd-7.2.4-1.el7.remi.x86_64                                                     2/2 
  Verifying  : php-pdo-7.2.4-1.el7.remi.x86_64                                                         1/2 
  Verifying  : php-mysqlnd-7.2.4-1.el7.remi.x86_64                                                     2/2 

Installed:
  php-mysqlnd.x86_64 0:7.2.4-1.el7.remi                  php-pdo.x86_64 0:7.2.4-1.el7.remi                 

Complete!
[root@lamp ~]# systemctl restart httpd.service
[root@lamp ~]# 
~~~

就可以在 phpinfo 中看到相关信息了

![lamp](/assets/img/lamp/lamp03.png)

到此 LAMP 这个框架就算是已经搭建起来了

后面基于这个框架，可以有一批有趣的应用能快速的进行构建

---

# 总结

各种 rpm 包的拼接就可以搭建出 LAMP，总体来讲还是很简单的

* TOC
{:toc}

---

[lamp]:https://en.wikipedia.org/wiki/LAMP_(software_bundle)
[lamp_install]:https://www.howtoforge.com/tutorial/centos-lamp-server-apache-mysql-php/
