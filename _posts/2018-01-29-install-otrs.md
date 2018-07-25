---
layout: post
title: "Install Otrs"
author:  wilmosfang
date: 2018-01-29 14:38:16
image: '/assets/img/'
excerpt: 'Otrs 的安装方法'
main-class: otrs
color: '66b2c1'
tags:
 - otrs
 - perl
 - mysql
categories:
 - otrs
twitter_text: 'simple process of Otrs installation'
introduction: 'installation method of Otrs'
---


# 前言


**[Otrs][otrs]** 是一个开源的工单系统

>OTRS is one of the most flexible web-based ticketing systems used for Customer Service, Help Desk, IT Service Management. With a fast implementation and easy customization to your needs it helps you reducing costs and increasing the efficiency and transparency of your business communication.  

使用工单来对接请求服务，跟踪问题，管理事务，协调人力是一种高效的组织方式

这里分享一下 **[Otrs][otrs]** 的安装方法

参考 **[Otrs安装文档][otrs_installation]**

> **Tip:** 当前版本 **OTRS 6 Patch Level 4**

---

# 操作

## 系统环境


~~~
[root@much otrs]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 7b4bb229c627449d8860c83a818b2416
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much otrs]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0b:e9:0b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84089sec preferred_lft 84089sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:36:8b:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.209/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much otrs]#
~~~


## 下载相关软件包

~~~
[root@much otrs]# pwd
/root/otrs
[root@much otrs]# ls
mysql57-community-release-el7-11.noarch.rpm  otrs-6.0.4-03.noarch.rpm
[root@much otrs]# md5sum otrs-6.0.4-03.noarch.rpm
fd11a980d5ca7b09681f10ef7091890e  otrs-6.0.4-03.noarch.rpm
[root@much otrs]#
~~~

**[Otrs的下载地址][otrs_dl]**

## 禁用 SELinux

~~~
[root@much otrs]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
#SELINUX=enforcing
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted


[root@much otrs]# getenforce
Disabled
[root@much otrs]#
~~~

## 安装 mysql

### 安装 mysql 仓库

~~~
[root@much otrs]# ls
mysql57-community-release-el7-11.noarch.rpm  otrs-6.0.4-03.noarch.rpm
[root@much otrs]# rpm -ivh mysql57-community-release-el7-11.noarch.rpm
warning: mysql57-community-release-el7-11.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-11 ################################# [100%]
[root@much otrs]# ll /etc/yum.repos.d/
total 48
-rw-r--r--  1 root root  170 1月  22 21:10 cassandra.repo
-rw-r--r--. 1 root root 1664 11月 30 2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 2016 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 2016 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 2016 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 6月  16 2017 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 2016 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 2016 CentOS-Vault.repo
-rw-r--r--. 1 root root  957 12月 28 2016 epel.repo
-rw-r--r--. 1 root root 1056 12月 28 2016 epel-testing.repo
-rw-r--r--  1 root root 1838 4月  27 2017 mysql-community.repo
-rw-r--r--  1 root root 1885 4月  27 2017 mysql-community-source.repo
[root@much otrs]#
~~~

### 软件列表

~~~
[root@much otrs]# yum list  all | grep  mysql57
mysql57-community-release.noarch        el7-11                         installed
mysql-community-client.i686             5.7.21-1.el7                   mysql57-community
mysql-community-client.x86_64           5.7.21-1.el7                   mysql57-community
mysql-community-common.i686             5.7.21-1.el7                   mysql57-community
mysql-community-common.x86_64           5.7.21-1.el7                   mysql57-community
mysql-community-devel.i686              5.7.21-1.el7                   mysql57-community
mysql-community-devel.x86_64            5.7.21-1.el7                   mysql57-community
mysql-community-embedded.i686           5.7.21-1.el7                   mysql57-community
mysql-community-embedded.x86_64         5.7.21-1.el7                   mysql57-community
mysql-community-embedded-compat.i686    5.7.21-1.el7                   mysql57-community
mysql-community-embedded-compat.x86_64  5.7.21-1.el7                   mysql57-community
mysql-community-embedded-devel.i686     5.7.21-1.el7                   mysql57-community
mysql-community-embedded-devel.x86_64   5.7.21-1.el7                   mysql57-community
mysql-community-libs.i686               5.7.21-1.el7                   mysql57-community
mysql-community-libs.x86_64             5.7.21-1.el7                   mysql57-community
mysql-community-libs-compat.i686        5.7.21-1.el7                   mysql57-community
mysql-community-libs-compat.x86_64      5.7.21-1.el7                   mysql57-community
mysql-community-release.noarch          el7-7                          mysql57-community
mysql-community-server.x86_64           5.7.21-1.el7                   mysql57-community
mysql-community-test.x86_64             5.7.21-1.el7                   mysql57-community
                                        1-20170320                     mysql57-community
mysql-ref-manual-5.5-en-pdf.noarch      1-20170320                     mysql57-community
                                        1-20171228                     mysql57-community
mysql-ref-manual-5.7-en-pdf.noarch      1-20171228                     mysql57-community
[root@much otrs]#
~~~


### 安装 mysql

~~~
[root@much otrs]# yum install  mysql-community-server.x86_64     
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package mysql-community-server.x86_64 0:5.7.21-1.el7 will be installed
--> Processing Dependency: mysql-community-common(x86-64) = 5.7.21-1.el7 for package: mysql-community-server-5.7.21-1.el7.x86_64
--> Processing Dependency: mysql-community-client(x86-64) >= 5.7.9 for package: mysql-community-server-5.7.21-1.el7.x86_64
--> Running transaction check
---> Package mysql-community-client.x86_64 0:5.7.21-1.el7 will be installed
--> Processing Dependency: mysql-community-libs(x86-64) >= 5.7.9 for package: mysql-community-client-5.7.21-1.el7.x86_64
---> Package mysql-community-common.x86_64 0:5.7.21-1.el7 will be installed
--> Running transaction check
---> Package mariadb-libs.x86_64 1:5.5.52-1.el7 will be obsoleted
--> Processing Dependency: libmysqlclient.so.18()(64bit) for package: 2:postfix-2.10.1-6.el7.x86_64
--> Processing Dependency: libmysqlclient.so.18(libmysqlclient_18)(64bit) for package: 2:postfix-2.10.1-6.el7.x86_64
---> Package mysql-community-libs.x86_64 0:5.7.21-1.el7 will be obsoleting
--> Running transaction check
---> Package mysql-community-libs-compat.x86_64 0:5.7.21-1.el7 will be obsoleting
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================
 Package                                        Arch                      Version                         Repository                            Size
=====================================================================================================================================================
Installing:
 mysql-community-libs                           x86_64                    5.7.21-1.el7                    mysql57-community                    2.1 M
     replacing  mariadb-libs.x86_64 1:5.5.52-1.el7
 mysql-community-libs-compat                    x86_64                    5.7.21-1.el7                    mysql57-community                    2.0 M
     replacing  mariadb-libs.x86_64 1:5.5.52-1.el7
 mysql-community-server                         x86_64                    5.7.21-1.el7                    mysql57-community                    164 M
Installing for dependencies:
 mysql-community-client                         x86_64                    5.7.21-1.el7                    mysql57-community                     24 M
 mysql-community-common                         x86_64                    5.7.21-1.el7                    mysql57-community                    272 k

Transaction Summary
=====================================================================================================================================================
Install  3 Packages (+2 Dependent packages)

Total download size: 192 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/mysql57-community/packages/mysql-community-common-5.7.21-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Public key for mysql-community-common-5.7.21-1.el7.x86_64.rpm is not installed
(1/5): mysql-community-common-5.7.21-1.el7.x86_64.rpm                                                                         | 272 kB  00:00:01     
(2/5): mysql-community-libs-5.7.21-1.el7.x86_64.rpm                                                                           | 2.1 MB  00:00:13     
(3/5): mysql-community-libs-compat-5.7.21-1.el7.x86_64.rpm                                                                    | 2.0 MB  00:00:10     
(4/5): mysql-community-client-5.7.21-1.el7.x86_64.rpm                                                                         |  24 MB  00:02:18     
(5/5): mysql-community-server-5.7.21-1.el7.x86_64.rpm                                                                         | 164 MB  00:08:59     
-----------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                348 kB/s | 192 MB  00:09:25     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Importing GPG key 0x5072E1F5:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: a4a9 4068 76fc bd3c 4567 70c8 8c71 8d3b 5072 e1f5
 Package    : mysql57-community-release-el7-11.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:
ipa-client-4.4.0-14.el7.centos.7.x86_64 has installed conflicts freeipa-client: ipa-client-4.4.0-14.el7.centos.7.x86_64
ipa-client-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-client-common: ipa-client-common-4.4.0-14.el7.centos.7.noarch
ipa-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-common: ipa-common-4.4.0-14.el7.centos.7.noarch
  Installing : mysql-community-common-5.7.21-1.el7.x86_64                                                                                        1/6
  Installing : mysql-community-libs-5.7.21-1.el7.x86_64                                                                                          2/6
  Installing : mysql-community-client-5.7.21-1.el7.x86_64                                                                                        3/6
  Installing : mysql-community-server-5.7.21-1.el7.x86_64                                                                                        4/6
  Installing : mysql-community-libs-compat-5.7.21-1.el7.x86_64                                                                                   5/6
  Erasing    : 1:mariadb-libs-5.5.52-1.el7.x86_64                                                                                                6/6
  Verifying  : mysql-community-client-5.7.21-1.el7.x86_64                                                                                        1/6
  Verifying  : mysql-community-libs-compat-5.7.21-1.el7.x86_64                                                                                   2/6
  Verifying  : mysql-community-libs-5.7.21-1.el7.x86_64                                                                                          3/6
  Verifying  : mysql-community-server-5.7.21-1.el7.x86_64                                                                                        4/6
  Verifying  : mysql-community-common-5.7.21-1.el7.x86_64                                                                                        5/6
  Verifying  : 1:mariadb-libs-5.5.52-1.el7.x86_64                                                                                                6/6

Installed:
  mysql-community-libs.x86_64 0:5.7.21-1.el7    mysql-community-libs-compat.x86_64 0:5.7.21-1.el7    mysql-community-server.x86_64 0:5.7.21-1.el7   

Dependency Installed:
  mysql-community-client.x86_64 0:5.7.21-1.el7                              mysql-community-common.x86_64 0:5.7.21-1.el7                             

Replaced:
  mariadb-libs.x86_64 1:5.5.52-1.el7                                                                                                                 

Complete!
[root@much otrs]#
~~~


### 启动 mysql

~~~
[root@much otrs]# systemctl  start mysqld
[root@much otrs]# systemctl  status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2018-01-29 23:40:24 CST; 6s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 3040 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 2960 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 3043 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─3043 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

1月 29 23:40:19 much systemd[1]: Starting MySQL Server...
1月 29 23:40:24 much systemd[1]: Started MySQL Server.
[root@much otrs]#
~~~


### 获取密码

~~~
[root@much mysql]# grep password /var/log/mysqld.log
2018-01-29T15:40:21.149869Z 1 [Note] A temporary password is generated for root@localhost: gis77wkd<pkL
[root@much mysql]#
~~~


### 修改密码


~~~
[root@much mysql]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> alter user 'root'@'localhost'  IDENTIFIED BY  'x1iAOA8A0ub2tWX_2L3s';
Query OK, 0 rows affected (0.00 sec)

mysql>
~~~


## 安装 otrs

~~~
[root@much ~]# cd
[root@much ~]# cd otrs/
[root@much otrs]# ls
mysql57-community-release-el7-11.noarch.rpm  otrs-6.0.4-03.noarch.rpm
[root@much otrs]# yum install otrs-6.0.4-03.noarch.rpm
Loaded plugins: fastestmirror, langpacks
Examining otrs-6.0.4-03.noarch.rpm: otrs-6.0.4-03.noarch
Marking otrs-6.0.4-03.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package otrs.noarch 0:6.0.4-03 will be installed
--> Processing Dependency: perl(Archive::Zip) for package: otrs-6.0.4-03.noarch
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
--> Processing Dependency: perl(DateTime) for package: otrs-6.0.4-03.noarch
--> Processing Dependency: perl(Net::DNS) for package: otrs-6.0.4-03.noarch
--> Processing Dependency: perl(Net::LDAP) for package: otrs-6.0.4-03.noarch
--> Processing Dependency: perl(Template) for package: otrs-6.0.4-03.noarch
--> Processing Dependency: perl(XML::LibXML) for package: otrs-6.0.4-03.noarch
--> Processing Dependency: perl(XML::LibXSLT) for package: otrs-6.0.4-03.noarch
--> Processing Dependency: procmail for package: otrs-6.0.4-03.noarch
--> Running transaction check
---> Package perl-Archive-Zip.noarch 0:1.30-11.el7 will be installed
---> Package perl-DateTime.x86_64 2:1.04-6.el7 will be installed
--> Processing Dependency: perl(Params::Validate) >= 0.76 for package: 2:perl-DateTime-1.04-6.el7.x86_64
--> Processing Dependency: perl(DateTime::TimeZone) >= 1.09 for package: 2:perl-DateTime-1.04-6.el7.x86_64
--> Processing Dependency: perl(DateTime::Locale) >= 0.41 for package: 2:perl-DateTime-1.04-6.el7.x86_64
--> Processing Dependency: perl(Try::Tiny) for package: 2:perl-DateTime-1.04-6.el7.x86_64
---> Package perl-LDAP.noarch 1:0.56-5.el7 will be installed
--> Processing Dependency: perl(Convert::ASN1) >= 0.2 for package: 1:perl-LDAP-0.56-5.el7.noarch
--> Processing Dependency: perl(Authen::SASL) >= 2.00 for package: 1:perl-LDAP-0.56-5.el7.noarch
--> Processing Dependency: perl(XML::SAX::Writer) for package: 1:perl-LDAP-0.56-5.el7.noarch
--> Processing Dependency: perl(XML::SAX::Base) for package: 1:perl-LDAP-0.56-5.el7.noarch
--> Processing Dependency: perl(JSON) for package: 1:perl-LDAP-0.56-5.el7.noarch
---> Package perl-Net-DNS.x86_64 0:0.72-6.el7 will be installed
--> Processing Dependency: perl(Digest::HMAC_MD5) >= 1 for package: perl-Net-DNS-0.72-6.el7.x86_64
---> Package perl-Template-Toolkit.x86_64 0:2.24-5.el7 will be installed
--> Processing Dependency: perl(Pod::POM) for package: perl-Template-Toolkit-2.24-5.el7.x86_64
--> Processing Dependency: perl(Image::Info) for package: perl-Template-Toolkit-2.24-5.el7.x86_64
--> Processing Dependency: perl(AppConfig) for package: perl-Template-Toolkit-2.24-5.el7.x86_64
---> Package perl-XML-LibXML.x86_64 1:2.0018-5.el7 will be installed
--> Processing Dependency: perl(XML::SAX::DocumentLocator) for package: 1:perl-XML-LibXML-2.0018-5.el7.x86_64
--> Processing Dependency: perl(XML::NamespaceSupport) for package: 1:perl-XML-LibXML-2.0018-5.el7.x86_64
---> Package perl-XML-LibXSLT.x86_64 0:1.80-4.el7 will be installed
---> Package procmail.x86_64 0:3.22-36.el7_4.1 will be installed
--> Running transaction check
---> Package perl-AppConfig.noarch 0:1.66-20.el7 will be installed
---> Package perl-Authen-SASL.noarch 0:2.15-10.el7 will be installed
--> Processing Dependency: perl(GSSAPI) for package: perl-Authen-SASL-2.15-10.el7.noarch
---> Package perl-Convert-ASN1.noarch 0:0.26-4.el7 will be installed
---> Package perl-DateTime-Locale.noarch 0:0.45-6.el7 will be installed
--> Processing Dependency: perl(List::MoreUtils) for package: perl-DateTime-Locale-0.45-6.el7.noarch
---> Package perl-DateTime-TimeZone.noarch 0:1.63-2.el7 will be installed
--> Processing Dependency: perl(Class::Singleton) >= 1.03 for package: perl-DateTime-TimeZone-1.63-2.el7.noarch
--> Processing Dependency: perl(Class::Load) for package: perl-DateTime-TimeZone-1.63-2.el7.noarch
---> Package perl-Digest-HMAC.noarch 0:1.03-5.el7 will be installed
---> Package perl-Image-Info.noarch 0:1.33-3.el7 will be installed
--> Processing Dependency: perl(Image::Xpm) >= 1.09 for package: perl-Image-Info-1.33-3.el7.noarch
--> Processing Dependency: perl(Image::Xbm) >= 1.07 for package: perl-Image-Info-1.33-3.el7.noarch
--> Processing Dependency: perl(XML::Simple) for package: perl-Image-Info-1.33-3.el7.noarch
---> Package perl-JSON.noarch 0:2.59-2.el7 will be installed
---> Package perl-Params-Validate.x86_64 0:1.08-4.el7 will be installed
--> Processing Dependency: perl(Module::Implementation) for package: perl-Params-Validate-1.08-4.el7.x86_64
---> Package perl-Pod-POM.noarch 0:0.27-10.el7 will be installed
---> Package perl-Try-Tiny.noarch 0:0.12-2.el7 will be installed
---> Package perl-XML-NamespaceSupport.noarch 0:1.11-10.el7 will be installed
---> Package perl-XML-SAX.noarch 0:0.99-9.el7 will be installed
---> Package perl-XML-SAX-Base.noarch 0:1.08-7.el7 will be installed
---> Package perl-XML-SAX-Writer.noarch 0:0.53-4.el7 will be installed
--> Processing Dependency: perl(XML::Filter::BufferText) for package: perl-XML-SAX-Writer-0.53-4.el7.noarch
--> Running transaction check
---> Package perl-Class-Load.noarch 0:0.20-3.el7 will be installed
--> Processing Dependency: perl(Package::Stash) >= 0.14 for package: perl-Class-Load-0.20-3.el7.noarch
--> Processing Dependency: perl(Module::Runtime) >= 0.012 for package: perl-Class-Load-0.20-3.el7.noarch
--> Processing Dependency: perl(Module::Runtime) for package: perl-Class-Load-0.20-3.el7.noarch
--> Processing Dependency: perl(Data::OptList) for package: perl-Class-Load-0.20-3.el7.noarch
---> Package perl-Class-Singleton.noarch 0:1.4-14.el7 will be installed
---> Package perl-GSSAPI.x86_64 0:0.28-9.el7 will be installed
---> Package perl-Image-Xbm.noarch 0:1.08-21.el7 will be installed
--> Processing Dependency: perl(Image::Base) for package: perl-Image-Xbm-1.08-21.el7.noarch
---> Package perl-Image-Xpm.noarch 0:1.09-21.el7 will be installed
---> Package perl-List-MoreUtils.x86_64 0:0.33-9.el7 will be installed
---> Package perl-Module-Implementation.noarch 0:0.06-6.el7 will be installed
---> Package perl-XML-Filter-BufferText.noarch 0:1.01-17.el7 will be installed
---> Package perl-XML-Simple.noarch 0:2.20-5.el7 will be installed
--> Running transaction check
---> Package perl-Data-OptList.noarch 0:0.107-9.el7 will be installed
--> Processing Dependency: perl(Sub::Install) >= 0.921 for package: perl-Data-OptList-0.107-9.el7.noarch
--> Processing Dependency: perl(Params::Util) for package: perl-Data-OptList-0.107-9.el7.noarch
---> Package perl-Image-Base.noarch 0:1.07-23.el7 will be installed
---> Package perl-Module-Runtime.noarch 0:0.013-4.el7 will be installed
---> Package perl-Package-Stash.noarch 0:0.34-2.el7 will be installed
--> Processing Dependency: perl(Package::Stash::XS) >= 0.26 for package: perl-Package-Stash-0.34-2.el7.noarch
--> Processing Dependency: perl(Package::DeprecationManager) for package: perl-Package-Stash-0.34-2.el7.noarch
--> Running transaction check
---> Package perl-Package-DeprecationManager.noarch 0:0.13-7.el7 will be installed
---> Package perl-Package-Stash-XS.x86_64 0:0.26-3.el7 will be installed
---> Package perl-Params-Util.x86_64 0:1.07-6.el7 will be installed
---> Package perl-Sub-Install.noarch 0:0.926-6.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================
 Package                                         Arch                   Version                          Repository                             Size
=====================================================================================================================================================
Installing:
 otrs                                            noarch                 6.0.4-03                         /otrs-6.0.4-03.noarch                 150 M
Installing for dependencies:
 perl-AppConfig                                  noarch                 1.66-20.el7                      base                                   88 k
 perl-Archive-Zip                                noarch                 1.30-11.el7                      base                                  107 k
 perl-Authen-SASL                                noarch                 2.15-10.el7                      base                                   57 k
 perl-Class-Load                                 noarch                 0.20-3.el7                       base                                   27 k
 perl-Class-Singleton                            noarch                 1.4-14.el7                       base                                   18 k
 perl-Convert-ASN1                               noarch                 0.26-4.el7                       base                                   54 k
 perl-Data-OptList                               noarch                 0.107-9.el7                      base                                   23 k
 perl-DateTime                                   x86_64                 2:1.04-6.el7                     base                                  112 k
 perl-DateTime-Locale                            noarch                 0.45-6.el7                       base                                  1.6 M
 perl-DateTime-TimeZone                          noarch                 1.63-2.el7                       base                                  417 k
 perl-Digest-HMAC                                noarch                 1.03-5.el7                       base                                   16 k
 perl-GSSAPI                                     x86_64                 0.28-9.el7                       base                                   59 k
 perl-Image-Base                                 noarch                 1.07-23.el7                      base                                   14 k
 perl-Image-Info                                 noarch                 1.33-3.el7                       base                                   83 k
 perl-Image-Xbm                                  noarch                 1.08-21.el7                      base                                   17 k
 perl-Image-Xpm                                  noarch                 1.09-21.el7                      base                                   19 k
 perl-JSON                                       noarch                 2.59-2.el7                       base                                   96 k
 perl-LDAP                                       noarch                 1:0.56-5.el7                     base                                  411 k
 perl-List-MoreUtils                             x86_64                 0.33-9.el7                       base                                   58 k
 perl-Module-Implementation                      noarch                 0.06-6.el7                       base                                   17 k
 perl-Module-Runtime                             noarch                 0.013-4.el7                      base                                   19 k
 perl-Net-DNS                                    x86_64                 0.72-6.el7                       base                                  308 k
 perl-Package-DeprecationManager                 noarch                 0.13-7.el7                       base                                   18 k
 perl-Package-Stash                              noarch                 0.34-2.el7                       base                                   34 k
 perl-Package-Stash-XS                           x86_64                 0.26-3.el7                       base                                   31 k
 perl-Params-Util                                x86_64                 1.07-6.el7                       base                                   38 k
 perl-Params-Validate                            x86_64                 1.08-4.el7                       base                                   69 k
 perl-Pod-POM                                    noarch                 0.27-10.el7                      base                                  101 k
 perl-Sub-Install                                noarch                 0.926-6.el7                      base                                   21 k
 perl-Template-Toolkit                           x86_64                 2.24-5.el7                       base                                  1.4 M
 perl-Try-Tiny                                   noarch                 0.12-2.el7                       base                                   23 k
 perl-XML-Filter-BufferText                      noarch                 1.01-17.el7                      base                                   11 k
 perl-XML-LibXML                                 x86_64                 1:2.0018-5.el7                   base                                  373 k
 perl-XML-LibXSLT                                x86_64                 1.80-4.el7                       base                                   59 k
 perl-XML-NamespaceSupport                       noarch                 1.11-10.el7                      base                                   18 k
 perl-XML-SAX                                    noarch                 0.99-9.el7                       base                                   63 k
 perl-XML-SAX-Base                               noarch                 1.08-7.el7                       base                                   32 k
 perl-XML-SAX-Writer                             noarch                 0.53-4.el7                       base                                   25 k
 perl-XML-Simple                                 noarch                 2.20-5.el7                       base                                   74 k
 procmail                                        x86_64                 3.22-36.el7_4.1                  updates                               171 k

Transaction Summary
=====================================================================================================================================================
Install  1 Package (+40 Dependent packages)

Total size: 156 M
Total download size: 6.1 M
Installed size: 175 M
Is this ok [y/d/N]: y
Downloading packages:
(1/40): perl-Class-Singleton-1.4-14.el7.noarch.rpm                                                                            |  18 kB  00:00:00     
(2/40): perl-Convert-ASN1-0.26-4.el7.noarch.rpm                                                                               |  54 kB  00:00:00     
(3/40): perl-Archive-Zip-1.30-11.el7.noarch.rpm                                                                               | 107 kB  00:00:01     
(4/40): perl-Class-Load-0.20-3.el7.noarch.rpm                                                                                 |  27 kB  00:00:01     
(5/40): perl-Data-OptList-0.107-9.el7.noarch.rpm                                                                              |  23 kB  00:00:00     
(6/40): perl-AppConfig-1.66-20.el7.noarch.rpm                                                                                 |  88 kB  00:00:01     
(7/40): perl-Digest-HMAC-1.03-5.el7.noarch.rpm                                                                                |  16 kB  00:00:00     
(8/40): perl-DateTime-1.04-6.el7.x86_64.rpm                                                                                   | 112 kB  00:00:01     
(9/40): perl-Authen-SASL-2.15-10.el7.noarch.rpm                                                                               |  57 kB  00:00:02     
(10/40): perl-Image-Base-1.07-23.el7.noarch.rpm                                                                               |  14 kB  00:00:00     
(11/40): perl-Image-Xbm-1.08-21.el7.noarch.rpm                                                                                |  17 kB  00:00:00     
(12/40): perl-GSSAPI-0.28-9.el7.x86_64.rpm                                                                                    |  59 kB  00:00:01     
(13/40): perl-Image-Xpm-1.09-21.el7.noarch.rpm                                                                                |  19 kB  00:00:01     
(14/40): perl-DateTime-TimeZone-1.63-2.el7.noarch.rpm                                                                         | 417 kB  00:00:03     
(15/40): perl-Image-Info-1.33-3.el7.noarch.rpm                                                                                |  83 kB  00:00:02     
(16/40): perl-List-MoreUtils-0.33-9.el7.x86_64.rpm                                                                            |  58 kB  00:00:00     
(17/40): perl-JSON-2.59-2.el7.noarch.rpm                                                                                      |  96 kB  00:00:02     
(18/40): perl-Module-Runtime-0.013-4.el7.noarch.rpm                                                                           |  19 kB  00:00:00     
(19/40): perl-Module-Implementation-0.06-6.el7.noarch.rpm                                                                     |  17 kB  00:00:00     
(20/40): perl-Package-DeprecationManager-0.13-7.el7.noarch.rpm                                                                |  18 kB  00:00:00     
(21/40): perl-Package-Stash-XS-0.26-3.el7.x86_64.rpm                                                                          |  31 kB  00:00:00     
(22/40): perl-Params-Util-1.07-6.el7.x86_64.rpm                                                                               |  38 kB  00:00:00     
(23/40): perl-Package-Stash-0.34-2.el7.noarch.rpm                                                                             |  34 kB  00:00:00     
(24/40): perl-Params-Validate-1.08-4.el7.x86_64.rpm                                                                           |  69 kB  00:00:00     
(25/40): perl-Sub-Install-0.926-6.el7.noarch.rpm                                                                              |  21 kB  00:00:00     
(26/40): perl-Pod-POM-0.27-10.el7.noarch.rpm                                                                                  | 101 kB  00:00:01     
(27/40): perl-Try-Tiny-0.12-2.el7.noarch.rpm                                                                                  |  23 kB  00:00:00     
(28/40): perl-XML-Filter-BufferText-1.01-17.el7.noarch.rpm                                                                    |  11 kB  00:00:00     
(29/40): perl-Net-DNS-0.72-6.el7.x86_64.rpm                                                                                   | 308 kB  00:00:04     
(30/40): perl-LDAP-0.56-5.el7.noarch.rpm                                                                                      | 411 kB  00:00:06     
(31/40): perl-XML-NamespaceSupport-1.11-10.el7.noarch.rpm                                                                     |  18 kB  00:00:00     
(32/40): perl-XML-LibXSLT-1.80-4.el7.x86_64.rpm                                                                               |  59 kB  00:00:01     
(33/40): perl-XML-SAX-0.99-9.el7.noarch.rpm                                                                                   |  63 kB  00:00:01     
(34/40): perl-XML-SAX-Writer-0.53-4.el7.noarch.rpm                                                                            |  25 kB  00:00:00     
(35/40): perl-XML-SAX-Base-1.08-7.el7.noarch.rpm                                                                              |  32 kB  00:00:01     
(36/40): perl-XML-Simple-2.20-5.el7.noarch.rpm                                                                                |  74 kB  00:00:01     
(37/40): perl-XML-LibXML-2.0018-5.el7.x86_64.rpm                                                                              | 373 kB  00:00:06     
(38/40): procmail-3.22-36.el7_4.1.x86_64.rpm                                                                                  | 171 kB  00:00:03     
(39/40): perl-Template-Toolkit-2.24-5.el7.x86_64.rpm                                                                          | 1.4 MB  00:00:08     
(40/40): perl-DateTime-Locale-0.45-6.el7.noarch.rpm                                                                           | 1.6 MB  00:00:18     
-----------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                307 kB/s | 6.1 MB  00:00:20     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : perl-XML-SAX-Base-1.08-7.el7.noarch                                                                                              1/41
  Installing : perl-XML-NamespaceSupport-1.11-10.el7.noarch                                                                                     2/41
  Installing : perl-Try-Tiny-0.12-2.el7.noarch                                                                                                  3/41
  Installing : perl-XML-SAX-0.99-9.el7.noarch                                                                                                   4/41
  Installing : 1:perl-XML-LibXML-2.0018-5.el7.x86_64                                                                                            5/41
  Installing : perl-Sub-Install-0.926-6.el7.noarch                                                                                              6/41
  Installing : perl-Image-Base-1.07-23.el7.noarch                                                                                               7/41
  Installing : perl-Digest-HMAC-1.03-5.el7.noarch                                                                                               8/41
  Installing : perl-Params-Util-1.07-6.el7.x86_64                                                                                               9/41
  Installing : perl-List-MoreUtils-0.33-9.el7.x86_64                                                                                           10/41
  Installing : perl-Module-Runtime-0.013-4.el7.noarch                                                                                          11/41
  Installing : perl-Module-Implementation-0.06-6.el7.noarch                                                                                    12/41
  Installing : perl-Params-Validate-1.08-4.el7.x86_64                                                                                          13/41
  Installing : perl-DateTime-Locale-0.45-6.el7.noarch                                                                                          14/41
  Installing : perl-Package-DeprecationManager-0.13-7.el7.noarch                                                                               15/41
  Installing : perl-Data-OptList-0.107-9.el7.noarch                                                                                            16/41
  Installing : perl-Net-DNS-0.72-6.el7.x86_64                                                                                                  17/41
  Installing : perl-Image-Xpm-1.09-21.el7.noarch                                                                                               18/41
  Installing : perl-Image-Xbm-1.08-21.el7.noarch                                                                                               19/41
  Installing : perl-XML-LibXSLT-1.80-4.el7.x86_64                                                                                              20/41
  Installing : perl-XML-Simple-2.20-5.el7.noarch                                                                                               21/41
  Installing : perl-Image-Info-1.33-3.el7.noarch                                                                                               22/41
  Installing : perl-XML-Filter-BufferText-1.01-17.el7.noarch                                                                                   23/41
  Installing : perl-XML-SAX-Writer-0.53-4.el7.noarch                                                                                           24/41
  Installing : perl-Archive-Zip-1.30-11.el7.noarch                                                                                             25/41
  Installing : perl-GSSAPI-0.28-9.el7.x86_64                                                                                                   26/41
  Installing : perl-Authen-SASL-2.15-10.el7.noarch                                                                                             27/41
  Installing : perl-Package-Stash-XS-0.26-3.el7.x86_64                                                                                         28/41
  Installing : perl-Package-Stash-0.34-2.el7.noarch                                                                                            29/41
  Installing : perl-Class-Load-0.20-3.el7.noarch                                                                                               30/41
  Installing : perl-Pod-POM-0.27-10.el7.noarch                                                                                                 31/41
  Installing : procmail-3.22-36.el7_4.1.x86_64                                                                                                 32/41
  Installing : perl-AppConfig-1.66-20.el7.noarch                                                                                               33/41
  Installing : perl-Template-Toolkit-2.24-5.el7.x86_64                                                                                         34/41
  Installing : perl-Class-Singleton-1.4-14.el7.noarch                                                                                          35/41
  Installing : 2:perl-DateTime-1.04-6.el7.x86_64                                                                                               36/41
  Installing : perl-DateTime-TimeZone-1.63-2.el7.noarch                                                                                        37/41
  Installing : perl-JSON-2.59-2.el7.noarch                                                                                                     38/41
  Installing : perl-Convert-ASN1-0.26-4.el7.noarch                                                                                             39/41
  Installing : 1:perl-LDAP-0.56-5.el7.noarch                                                                                                   40/41
Check OTRS user ... otrs added.
  Installing : otrs-6.0.4-03.noarch                                                                                                            41/41

Next steps:

[restart web server]
 systemctl restart apache2.service

[install the OTRS database]
 Make sure your database server is running.
 Use a web browser and open this link:
 http://much/otrs/installer.pl

[start OTRS daemon and corresponding watchdog cronjob]
 /opt/otrs/bin/otrs.Daemon.pl start
 /opt/otrs/bin/Cron.sh start

((enjoy))
 Your OTRS Team

  Verifying  : perl-Convert-ASN1-0.26-4.el7.noarch                                                                                              1/41
  Verifying  : perl-JSON-2.59-2.el7.noarch                                                                                                      2/41
  Verifying  : perl-Module-Runtime-0.013-4.el7.noarch                                                                                           3/41
  Verifying  : 2:perl-DateTime-1.04-6.el7.x86_64                                                                                                4/41
  Verifying  : perl-Try-Tiny-0.12-2.el7.noarch                                                                                                  5/41
  Verifying  : perl-Class-Singleton-1.4-14.el7.noarch                                                                                           6/41
  Verifying  : perl-XML-SAX-Writer-0.53-4.el7.noarch                                                                                            7/41
  Verifying  : perl-XML-SAX-0.99-9.el7.noarch                                                                                                   8/41
  Verifying  : perl-Image-Info-1.33-3.el7.noarch                                                                                                9/41
  Verifying  : perl-AppConfig-1.66-20.el7.noarch                                                                                               10/41
  Verifying  : procmail-3.22-36.el7_4.1.x86_64                                                                                                 11/41
  Verifying  : otrs-6.0.4-03.noarch                                                                                                            12/41
  Verifying  : perl-Data-OptList-0.107-9.el7.noarch                                                                                            13/41
  Verifying  : perl-XML-NamespaceSupport-1.11-10.el7.noarch                                                                                    14/41
  Verifying  : perl-List-MoreUtils-0.33-9.el7.x86_64                                                                                           15/41
  Verifying  : perl-Package-DeprecationManager-0.13-7.el7.noarch                                                                               16/41
  Verifying  : perl-XML-Filter-BufferText-1.01-17.el7.noarch                                                                                   17/41
  Verifying  : perl-Authen-SASL-2.15-10.el7.noarch                                                                                             18/41
  Verifying  : perl-Pod-POM-0.27-10.el7.noarch                                                                                                 19/41
  Verifying  : perl-Image-Xpm-1.09-21.el7.noarch                                                                                               20/41
  Verifying  : perl-Params-Util-1.07-6.el7.x86_64                                                                                              21/41
  Verifying  : perl-Net-DNS-0.72-6.el7.x86_64                                                                                                  22/41
  Verifying  : perl-Package-Stash-0.34-2.el7.noarch                                                                                            23/41
  Verifying  : 1:perl-XML-LibXML-2.0018-5.el7.x86_64                                                                                           24/41
  Verifying  : perl-DateTime-Locale-0.45-6.el7.noarch                                                                                          25/41
  Verifying  : perl-Package-Stash-XS-0.26-3.el7.x86_64                                                                                         26/41
  Verifying  : perl-Digest-HMAC-1.03-5.el7.noarch                                                                                              27/41
  Verifying  : perl-XML-LibXSLT-1.80-4.el7.x86_64                                                                                              28/41
  Verifying  : perl-Params-Validate-1.08-4.el7.x86_64                                                                                          29/41
  Verifying  : perl-GSSAPI-0.28-9.el7.x86_64                                                                                                   30/41
  Verifying  : perl-Class-Load-0.20-3.el7.noarch                                                                                               31/41
  Verifying  : perl-Archive-Zip-1.30-11.el7.noarch                                                                                             32/41
  Verifying  : perl-Image-Base-1.07-23.el7.noarch                                                                                              33/41
  Verifying  : perl-Sub-Install-0.926-6.el7.noarch                                                                                             34/41
  Verifying  : perl-Template-Toolkit-2.24-5.el7.x86_64                                                                                         35/41
  Verifying  : perl-Module-Implementation-0.06-6.el7.noarch                                                                                    36/41
  Verifying  : perl-Image-Xbm-1.08-21.el7.noarch                                                                                               37/41
  Verifying  : 1:perl-LDAP-0.56-5.el7.noarch                                                                                                   38/41
  Verifying  : perl-XML-SAX-Base-1.08-7.el7.noarch                                                                                             39/41
  Verifying  : perl-XML-Simple-2.20-5.el7.noarch                                                                                               40/41
  Verifying  : perl-DateTime-TimeZone-1.63-2.el7.noarch                                                                                        41/41

Installed:
  otrs.noarch 0:6.0.4-03                                                                                                                             

Dependency Installed:
  perl-AppConfig.noarch 0:1.66-20.el7           perl-Archive-Zip.noarch 0:1.30-11.el7                  perl-Authen-SASL.noarch 0:2.15-10.el7       
  perl-Class-Load.noarch 0:0.20-3.el7           perl-Class-Singleton.noarch 0:1.4-14.el7               perl-Convert-ASN1.noarch 0:0.26-4.el7       
  perl-Data-OptList.noarch 0:0.107-9.el7        perl-DateTime.x86_64 2:1.04-6.el7                      perl-DateTime-Locale.noarch 0:0.45-6.el7    
  perl-DateTime-TimeZone.noarch 0:1.63-2.el7    perl-Digest-HMAC.noarch 0:1.03-5.el7                   perl-GSSAPI.x86_64 0:0.28-9.el7             
  perl-Image-Base.noarch 0:1.07-23.el7          perl-Image-Info.noarch 0:1.33-3.el7                    perl-Image-Xbm.noarch 0:1.08-21.el7         
  perl-Image-Xpm.noarch 0:1.09-21.el7           perl-JSON.noarch 0:2.59-2.el7                          perl-LDAP.noarch 1:0.56-5.el7               
  perl-List-MoreUtils.x86_64 0:0.33-9.el7       perl-Module-Implementation.noarch 0:0.06-6.el7         perl-Module-Runtime.noarch 0:0.013-4.el7    
  perl-Net-DNS.x86_64 0:0.72-6.el7              perl-Package-DeprecationManager.noarch 0:0.13-7.el7    perl-Package-Stash.noarch 0:0.34-2.el7      
  perl-Package-Stash-XS.x86_64 0:0.26-3.el7     perl-Params-Util.x86_64 0:1.07-6.el7                   perl-Params-Validate.x86_64 0:1.08-4.el7    
  perl-Pod-POM.noarch 0:0.27-10.el7             perl-Sub-Install.noarch 0:0.926-6.el7                  perl-Template-Toolkit.x86_64 0:2.24-5.el7   
  perl-Try-Tiny.noarch 0:0.12-2.el7             perl-XML-Filter-BufferText.noarch 0:1.01-17.el7        perl-XML-LibXML.x86_64 1:2.0018-5.el7       
  perl-XML-LibXSLT.x86_64 0:1.80-4.el7          perl-XML-NamespaceSupport.noarch 0:1.11-10.el7         perl-XML-SAX.noarch 0:0.99-9.el7            
  perl-XML-SAX-Base.noarch 0:1.08-7.el7         perl-XML-SAX-Writer.noarch 0:0.53-4.el7                perl-XML-Simple.noarch 0:2.20-5.el7         
  procmail.x86_64 0:3.22-36.el7_4.1            

Complete!
[root@much otrs]#
~~~



## 启动 apache 服务

~~~
[root@much ~]# systemctl restart httpd
[root@much ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since 二 2018-01-30 00:05:36 CST; 9s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 3491 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─3491 /usr/sbin/httpd -DFOREGROUND
           ├─3492 /usr/sbin/httpd -DFOREGROUND
           ├─3493 /usr/sbin/httpd -DFOREGROUND
           ├─3494 /usr/sbin/httpd -DFOREGROUND
           ├─3495 /usr/sbin/httpd -DFOREGROUND
           ├─3496 /usr/sbin/httpd -DFOREGROUND
           └─3497 /usr/sbin/httpd -DFOREGROUND

1月 30 00:05:35 much systemd[1]: Starting The Apache HTTP Server...
1月 30 00:05:36 much httpd[3491]: AH00558: httpd: Could not reliably deter...ge
1月 30 00:05:36 much systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]#
~~~


otrs 的安装位置


~~~
[root@much ~]# ll /opt/otrs/
total 1320
-rw-rw----  1 otrs apache 455481 1月  13 03:21 ARCHIVE
-rw-rw----  1 otrs apache   4056 1月  13 03:21 AUTHORS.md
drwxrwsr-x  4 otrs apache    205 1月  29 23:54 bin
-rw-rw----  1 otrs apache 814884 1月  13 03:21 CHANGES.md
-rw-rw----  1 otrs apache     74 1月  13 03:21 CONTRIBUTING.md
-rw-rw----  1 otrs apache  34520 1月  13 03:21 COPYING
-rw-rw----  1 otrs apache  15839 1月  13 03:21 COPYING-Third-Party
drwxrwsr-x  2 otrs apache     20 1月  29 23:54 Custom
drwxrwsr-x  3 otrs apache     43 1月  29 23:54 doc
drwxr-xr-x  3 root root       18 1月  29 23:54 i18n
-rw-rw----  1 otrs apache    142 1月  13 03:21 INSTALL.md
drwxrwsr-x 10 otrs apache    211 1月  29 23:54 Kernel
-rw-rw----  1 otrs apache   2738 1月  13 03:21 README.md
-rw-rw----  1 otrs apache    103 1月  13 03:21 RELEASE
drwxrwsr-x  8 otrs apache    305 1月  29 23:54 scripts
-rw-rw----  1 otrs apache    132 1月  13 03:21 UPDATING.md
drwxrwsr-x 13 otrs apache    180 1月  29 23:54 var
[root@much ~]#
~~~


## 检查额外包

使用 otrs 自带的工具进行检查

~~~
[root@much ~]# ll /opt/otrs/bin/otrs.CheckModules.pl
-rwxrwx--- 1 otrs apache 24965 1月  13 03:21 /opt/otrs/bin/otrs.CheckModules.pl
[root@much ~]# /opt/otrs/bin/otrs.CheckModules.pl
  o Apache::DBI......................ok (v1.12)
  o Apache2::Reload..................ok (v0.13)
  o Archive::Tar.....................ok (v1.92)
  o Archive::Zip.....................ok (v1.30)
  o Crypt::Eksblowfish::Bcrypt.......Not installed! Use: 'yum install "perl(Crypt::Eksblowfish::Bcrypt)"' (optional - For strong password hashing.)
  o Crypt::SSLeay....................ok (v0.64)
  o Date::Format.....................ok (v2.24)
  o DateTime.........................ok (v1.04)
  o DBI..............................ok (v1.627)
  o DBD::mysql.......................Not installed! Use: 'yum install "perl(DBD::mysql)"' (optional - Required to connect to a MySQL database.)
  o DBD::ODBC........................Not installed! (optional - Required to connect to a MS-SQL database.)
  o DBD::Oracle......................Not installed! (optional - Required to connect to a Oracle database.)
  o DBD::Pg..........................Not installed! Use: 'yum install "perl(DBD::Pg)"' (optional - Required to connect to a PostgreSQL database.)
  o Digest::SHA......................ok (v5.85)
  o Encode::HanExtra.................Not installed! Use: 'yum install "perl(Encode::HanExtra)"' (optional - Required to handle mails with several Chinese character sets.)
  o IO::Socket::SSL..................ok (v1.94)
  o JSON::XS.........................Not installed! Use: 'yum install "perl(JSON::XS)"' (optional - Recommended for faster AJAX/JavaScript handling.)
  o List::Util::XS...................ok (v1.27)
  o LWP::UserAgent...................ok (v6.26)
  o Mail::IMAPClient.................Not installed! Use: 'yum install "perl(Mail::IMAPClient)"' (optional - Required for IMAP TLS connections.)
    o IO::Socket::SSL................ok (v1.94)
    o Authen::SASL...................ok (v2.15)
    o Authen::NTLM...................Not installed! Use: 'yum install "perl(Authen::NTLM)"' (optional - Required for NTLM authentication mechanism in IMAP connections.)
  o ModPerl::Util....................Not installed! Use: 'yum install "perl(ModPerl::Util)"' (optional - Improves Performance on Apache webservers dramatically.)
  o Net::DNS.........................ok (v0.72)
  o Net::LDAP........................ok (v0.56)
  o Template.........................ok (v2.24)
  o Template::Stash::XS..............ok (undef)
  o Text::CSV_XS.....................Not installed! Use: 'yum install "perl(Text::CSV_XS)"' (optional - Recommended for faster CSV handling.)
  o Time::HiRes......................ok (v1.9725)
  o XML::LibXML......................ok (v2.0018)
  o XML::LibXSLT.....................ok (v1.80)
  o XML::Parser......................ok (v2.41)
  o YAML::XS.........................Not installed! Use: 'yum install "perl(YAML::XS)"' (required - Required for fast YAML processing.)
[root@much ~]#
~~~

## 使用 epel 库来补全包

~~~
[root@much ~]# rpm -qa | grep epel
epel-release-7-9.noarch
[root@much ~]# yum install "perl(Crypt::Eksblowfish::Bcrypt)" "perl(DBD::mysql)" "perl(Encode::HanExtra)"  "perl(JSON::XS)"  "perl(Mail::IMAPClient)"  "perl(Authen::NTLM)"  "perl(ModPerl::Util)" "perl(Text::CSV_XS)"  "perl(YAML::XS)"  
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package mod_perl.x86_64 0:2.0.10-2.el7 will be installed
--> Processing Dependency: perl(BSD::Resource) for package: mod_perl-2.0.10-2.el7.x86_64
--> Processing Dependency: perl(Linux::Pid) for package: mod_perl-2.0.10-2.el7.x86_64
---> Package perl-Crypt-Eksblowfish.x86_64 0:0.009-11.el7 will be installed
--> Processing Dependency: perl(Class::Mix) >= 0.001 for package: perl-Crypt-Eksblowfish-0.009-11.el7.x86_64
---> Package perl-DBD-MySQL.x86_64 0:4.023-5.el7 will be installed
---> Package perl-Encode-HanExtra.x86_64 0:0.23-10.el7 will be installed
---> Package perl-JSON-XS.x86_64 1:3.01-2.el7 will be installed
--> Processing Dependency: perl(Types::Serialiser) for package: 1:perl-JSON-XS-3.01-2.el7.x86_64
--> Processing Dependency: perl(common::sense) for package: 1:perl-JSON-XS-3.01-2.el7.x86_64
---> Package perl-Mail-IMAPClient.noarch 0:3.37-1.el7 will be installed
--> Processing Dependency: perl(Parse::RecDescent) for package: perl-Mail-IMAPClient-3.37-1.el7.noarch
---> Package perl-NTLM.noarch 0:1.09-5.el7 will be installed
---> Package perl-Text-CSV_XS.x86_64 0:1.00-3.el7 will be installed
---> Package perl-YAML-LibYAML.x86_64 0:0.54-1.el7 will be installed
--> Running transaction check
---> Package perl-BSD-Resource.x86_64 0:1.29.07-1.el7 will be installed
---> Package perl-Class-Mix.noarch 0:0.005-10.el7 will be installed
--> Processing Dependency: perl(Params::Classify) >= 0.000 for package: perl-Class-Mix-0.005-10.el7.noarch
---> Package perl-Linux-Pid.x86_64 0:0.04-18.el7 will be installed
---> Package perl-Parse-RecDescent.noarch 0:1.967009-5.el7 will be installed
---> Package perl-Types-Serialiser.noarch 0:1.0-1.el7 will be installed
---> Package perl-common-sense.noarch 0:3.6-4.el7 will be installed
--> Running transaction check
---> Package perl-Params-Classify.x86_64 0:0.013-7.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                     Arch        Version                Repository
                                                                           Size
================================================================================
Installing:
 mod_perl                    x86_64      2.0.10-2.el7           epel      3.0 M
 perl-Crypt-Eksblowfish      x86_64      0.009-11.el7           epel       54 k
 perl-DBD-MySQL              x86_64      4.023-5.el7            base      140 k
 perl-Encode-HanExtra        x86_64      0.23-10.el7            epel      1.7 M
 perl-JSON-XS                x86_64      1:3.01-2.el7           epel      103 k
 perl-Mail-IMAPClient        noarch      3.37-1.el7             epel      221 k
 perl-NTLM                   noarch      1.09-5.el7             epel       19 k
 perl-Text-CSV_XS            x86_64      1.00-3.el7             base       78 k
 perl-YAML-LibYAML           x86_64      0.54-1.el7             epel       87 k
Installing for dependencies:
 perl-BSD-Resource           x86_64      1.29.07-1.el7          epel       38 k
 perl-Class-Mix              noarch      0.005-10.el7           epel       16 k
 perl-Linux-Pid              x86_64      0.04-18.el7            epel       14 k
 perl-Params-Classify        x86_64      0.013-7.el7            epel       27 k
 perl-Parse-RecDescent       noarch      1.967009-5.el7         base      203 k
 perl-Types-Serialiser       noarch      1.0-1.el7              epel       17 k
 perl-common-sense           noarch      3.6-4.el7              epel       28 k

Transaction Summary
================================================================================
Install  9 Packages (+7 Dependent packages)

Total download size: 5.7 M
Installed size: 20 M
Is this ok [y/d/N]: y
Downloading packages:
(1/16): perl-DBD-MySQL-4.023-5.el7.x86_64.rpm              | 140 kB   00:00     
(2/16): perl-Crypt-Eksblowfish-0.009-11.el7.x86_64.rpm     |  54 kB   00:01     
perl-BSD-Resource-1.29.07-1.el FAILED                                          
http://mirror.smartmedia.net.id/epel/7/x86_64/Packages/p/perl-BSD-Resource-1.29.07-1.el7.x86_64.rpm: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below knowledge base article

https://access.redhat.com/articles/1320623

If above article doesn't help to resolve this issue please create a bug on https://bugs.centos.org/

(3/16): perl-Class-Mix-0.005-10.el7.noarch.rpm             |  16 kB   00:02     
(4/16): perl-JSON-XS-3.01-2.el7.x86_64.rpm                 | 103 kB   00:04     
(5/16): perl-Linux-Pid-0.04-18.el7.x86_64.rpm              |  14 kB   00:06     
(6/16): perl-Encode-HanExtra-0.23-10.el7.x86_64.rpm        | 1.7 MB   00:08     
(7/16): perl-NTLM-1.09-5.el7.noarch.rpm                    |  19 kB   00:04     
(8/16): perl-Mail-IMAPClient-3.37-1.el7.noarch.rpm         | 221 kB   00:07     
(9/16): mod_perl-2.0.10-2.el7.x86_64.rpm                   | 3.0 MB   00:23     
(10/16): perl-Types-Serialiser-1.0-1.el7.noarch.rpm        |  17 kB   00:00     
(11/16): perl-Text-CSV_XS-1.00-3.el7.x86_64.rpm            |  78 kB   00:01     
(12/16): perl-Parse-RecDescent-1.967009-5.el7.noarch.rpm   | 203 kB   00:01     
(13/16): perl-common-sense-3.6-4.el7.noarch.rpm            |  28 kB   00:01     
(14/16): perl-BSD-Resource-1.29.07-1.el7.x86_64.rpm        |  38 kB   00:00     
(15/16): perl-Params-Classify-0.013-7.el7.x86_64.rpm       |  27 kB   00:01     
(16/16): perl-YAML-LibYAML-0.54-1.el7.x86_64.rpm           |  87 kB   00:02     
--------------------------------------------------------------------------------
Total                                              227 kB/s | 5.7 MB  00:25     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : perl-common-sense-3.6-4.el7.noarch                          1/16
  Installing : perl-Types-Serialiser-1.0-1.el7.noarch                      2/16
  Installing : perl-Params-Classify-0.013-7.el7.x86_64                     3/16
  Installing : perl-Class-Mix-0.005-10.el7.noarch                          4/16
  Installing : perl-Parse-RecDescent-1.967009-5.el7.noarch                 5/16
  Installing : perl-BSD-Resource-1.29.07-1.el7.x86_64                      6/16
  Installing : perl-Linux-Pid-0.04-18.el7.x86_64                           7/16
  Installing : mod_perl-2.0.10-2.el7.x86_64                                8/16
  Installing : perl-Mail-IMAPClient-3.37-1.el7.noarch                      9/16
  Installing : perl-Crypt-Eksblowfish-0.009-11.el7.x86_64                 10/16
  Installing : 1:perl-JSON-XS-3.01-2.el7.x86_64                           11/16
  Installing : perl-DBD-MySQL-4.023-5.el7.x86_64                          12/16
  Installing : perl-NTLM-1.09-5.el7.noarch                                13/16
  Installing : perl-Text-CSV_XS-1.00-3.el7.x86_64                         14/16
  Installing : perl-YAML-LibYAML-0.54-1.el7.x86_64                        15/16
  Installing : perl-Encode-HanExtra-0.23-10.el7.x86_64                    16/16
  Verifying  : perl-common-sense-3.6-4.el7.noarch                          1/16
  Verifying  : perl-Linux-Pid-0.04-18.el7.x86_64                           2/16
  Verifying  : perl-BSD-Resource-1.29.07-1.el7.x86_64                      3/16
  Verifying  : perl-Encode-HanExtra-0.23-10.el7.x86_64                     4/16
  Verifying  : perl-YAML-LibYAML-0.54-1.el7.x86_64                         5/16
  Verifying  : perl-Text-CSV_XS-1.00-3.el7.x86_64                          6/16
  Verifying  : perl-Parse-RecDescent-1.967009-5.el7.noarch                 7/16
  Verifying  : perl-Params-Classify-0.013-7.el7.x86_64                     8/16
  Verifying  : perl-NTLM-1.09-5.el7.noarch                                 9/16
  Verifying  : perl-Mail-IMAPClient-3.37-1.el7.noarch                     10/16
  Verifying  : perl-Class-Mix-0.005-10.el7.noarch                         11/16
  Verifying  : 1:perl-JSON-XS-3.01-2.el7.x86_64                           12/16
  Verifying  : perl-DBD-MySQL-4.023-5.el7.x86_64                          13/16
  Verifying  : perl-Types-Serialiser-1.0-1.el7.noarch                     14/16
  Verifying  : mod_perl-2.0.10-2.el7.x86_64                               15/16
  Verifying  : perl-Crypt-Eksblowfish-0.009-11.el7.x86_64                 16/16

Installed:
  mod_perl.x86_64 0:2.0.10-2.el7                                                
  perl-Crypt-Eksblowfish.x86_64 0:0.009-11.el7                                  
  perl-DBD-MySQL.x86_64 0:4.023-5.el7                                           
  perl-Encode-HanExtra.x86_64 0:0.23-10.el7                                     
  perl-JSON-XS.x86_64 1:3.01-2.el7                                              
  perl-Mail-IMAPClient.noarch 0:3.37-1.el7                                      
  perl-NTLM.noarch 0:1.09-5.el7                                                 
  perl-Text-CSV_XS.x86_64 0:1.00-3.el7                                          
  perl-YAML-LibYAML.x86_64 0:0.54-1.el7                                         

Dependency Installed:
  perl-BSD-Resource.x86_64 0:1.29.07-1.el7                                      
  perl-Class-Mix.noarch 0:0.005-10.el7                                          
  perl-Linux-Pid.x86_64 0:0.04-18.el7                                           
  perl-Params-Classify.x86_64 0:0.013-7.el7                                     
  perl-Parse-RecDescent.noarch 0:1.967009-5.el7                                 
  perl-Types-Serialiser.noarch 0:1.0-1.el7                                      
  perl-common-sense.noarch 0:3.6-4.el7                                          

Complete!
[root@much ~]#
~~~


再次检查确认

~~~
[root@much ~]# /opt/otrs/bin/otrs.CheckModules.pl
  o Apache::DBI......................ok (v1.12)
  o Apache2::Reload..................ok (v0.13)
  o Archive::Tar.....................ok (v1.92)
  o Archive::Zip.....................ok (v1.30)
  o Crypt::Eksblowfish::Bcrypt.......ok (v0.009)
  o Crypt::SSLeay....................ok (v0.64)
  o Date::Format.....................ok (v2.24)
  o DateTime.........................ok (v1.04)
  o DBI..............................ok (v1.627)
  o DBD::mysql.......................ok (v4.023)
  o DBD::ODBC........................Not installed! (optional - Required to connect to a MS-SQL database.)
  o DBD::Oracle......................Not installed! (optional - Required to connect to a Oracle database.)
  o DBD::Pg..........................Not installed! Use: 'yum install "perl(DBD::Pg)"' (optional - Required to connect to a PostgreSQL database.)
  o Digest::SHA......................ok (v5.85)
  o Encode::HanExtra.................ok (v0.23)
  o IO::Socket::SSL..................ok (v1.94)
  o JSON::XS.........................ok (v3.01)
  o List::Util::XS...................ok (v1.27)
  o LWP::UserAgent...................ok (v6.26)
  o Mail::IMAPClient.................ok (v3.37)
    o IO::Socket::SSL................ok (v1.94)
    o Authen::SASL...................ok (v2.15)
    o Authen::NTLM...................ok (v1.09)
  o ModPerl::Util....................ok (v2.000010)
  o Net::DNS.........................ok (v0.72)
  o Net::LDAP........................ok (v0.56)
  o Template.........................ok (v2.24)
  o Template::Stash::XS..............ok (undef)
  o Text::CSV_XS.....................ok (v1.00)
  o Time::HiRes......................ok (v1.9725)
  o XML::LibXML......................ok (v2.0018)
  o XML::LibXSLT.....................ok (v1.80)
  o XML::Parser......................ok (v2.41)
  o YAML::XS.........................ok (v0.54)
[root@much ~]#
~~~

发现除了几个不需要的，其它都安装了

> 这里使用 **mysql** 自然就不需要连接 MS-SQL Oracle PostgrSQL 的 driver 了


## 配置 mysql

~~~
[root@much ~]# vim /etc/my.cnf
[root@much ~]# grep -v "^#" /etc/my.cnf | sed -r 's/^\s*$//'

[mysqld]
max_allowed_packet   = 64M
query_cache_size     = 32M
innodb_log_file_size = 256M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[root@much ~]# rpm -qa | grep  DBD
perl-DBD-MySQL-4.023-5.el7.x86_64
perl-DBD-SQLite-1.39-3.el7.x86_64
[root@much ~]#
[root@much ~]# systemctl  restart mysqld
[root@much ~]#
~~~


## 配置 utf8 字符集

~~~
[root@much my.cnf.d]# vim /etc/my.cnf
[root@much my.cnf.d]# grep -v "^#" /etc/my.cnf | sed -r 's/^\s*$//'
[client]
default-character-set=utf8

[mysqld]
max_allowed_packet   = 64M
query_cache_size     = 32M
innodb_log_file_size = 256M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
character-set-server=utf8
collation-server=utf8_general_ci

[mysql]
default-character-set=utf8
[root@much my.cnf.d]# mysql -uroot -px1iAOA8A0ub2tWX_2L3s
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)

mysql>
~~~

如果不是 utf8 的字符集，会在 web 页面配置过程中，检查连接的那一步报错


## 加强 mysql 安全


> 使用下面的工具可以进一步提升 mysql 安全

~~~
[root@much ~]# /usr/bin/mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root:
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration
of the plugin.
Using existing password for root.

Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password:
Sorry, you can't use an empty password here.

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
[root@much ~]#
~~~


## 启动服务

~~~
[root@much ~]# /opt/otrs/bin/otrs.Daemon.pl start

Manage the OTRS daemon process.

Error: You cannot run otrs.Daemon.pl as root. Please run it as the 'otrs' user or with the help of su:
  su -c "bin/otrs.Daemon.pl ..." -s /bin/bash otrs
[root@much ~]# su - otrs
Last login: 二 1月 30 00:21:28 CST 2018 on pts/1
[otrs@much ~]$ /opt/otrs/bin/otrs.Daemon.pl start

Manage the OTRS daemon process.

Daemon started
[Tue Jan 30 00:43:32 2018] otrs.Daemon.pl: DBI connect('database=otrs;host=127.0.0.1;','otrs',...) failed: Access denied for user 'otrs'@'localhost' (using password: YES) at /opt/otrs/Kernel/System/DB.pm line 204.
ERROR: OTRS-otrs.Daemon.pl - Daemon Kernel::System::Daemon::DaemonModules::SystemConfigurationSyncManager-10 Perl: 5.16.3 OS: linux Time: Tue Jan 30 00:43:32 2018

 Message: Access denied for user 'otrs'@'localhost' (using password: YES)

 Traceback (4495):
   Module: Kernel::System::DB::Ping Line: 1782
   Module: Kernel::System::Daemon::DaemonModules::SystemConfigurationSyncManager::PreRun Line: 90
   Module: (eval) Line: 316
   Module: main::Start Line: 316
   Module: /opt/otrs/bin/otrs.Daemon.pl Line: 137

~~~

>会提示不能使用 root 启动进程

~~~
[otrs@much ~]$ /opt/otrs/bin/Cron.sh start
(using /opt/otrs) done
[otrs@much ~]$
~~~

之后就可以使用 web 进行登录了配置了

访问 **http://serverip/otrs/install.pl**

![jenkins](/assets/img/otrs/otrs01.png)

---

# 总结

perl mysql 还有相关配置与依赖的处理是顺利安装的关键

接下来再分享一下相关的页面配置

* TOC
{:toc}


---

[otrs]:https://www.otrs.com/
[otrs_installation]:http://doc.otrs.com/doc/manual/admin/stable/zh_CN/html/installation.html
[otrs_dl]:https://www.otrs.com/%E4%B8%8B%E8%BD%BD-otrs-%E5%BC%80%E6%BA%90%E5%B8%AE%E5%8A%A9%E5%8F%B0%E8%BD%AF%E4%BB%B6%E5%92%8C%E5%85%8D%E8%B4%B9%E5%8A%9F%E8%83%BD/?lang=zh-hans
