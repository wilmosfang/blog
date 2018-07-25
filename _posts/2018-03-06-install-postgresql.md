---
layout: post
title: "Install PostgreSQL"
author:  wilmosfang
date: 2018-03-06 13:42:51
image: '/assets/img/'
excerpt: '安装 PostgreSQL'
main-class: cmdbuild
color: '#f79a30'
tags:
 - cmdbuild
 - postgresql
categories:
 - cmdbuild
twitter_text: 'simple process of PostgreSQL installation'
introduction: 'installation of PostgreSQL'
---



## 前言

**[PostgreSQL][postgresql]** 号称是这个世界上最高级的开源数据库

**[PostgreSQL][postgresql]** 的影响力越来越大了，虽然长期居于数据库排行榜的第四名(前三分别为 oracle, mysql, sqlserver)，不过近三年来(2015-2018年)，却是受关注涨幅最大的数据库，并且长期保持稳步增涨的态势，可能与其丰富的特性迎合了现代互联网的发展需求有一定关联

**CMDBuild** 也是通过 **[PostgreSQL][postgresql]** 来提供数据存储的, 在研究 **CMDBuild** 之前，需要对 **[PostgreSQL][postgresql]** 进行简单的了解

这里分享一下 **[PostgreSQL][postgresql]** 的安装方法

参考 **[Linux downloads (Red Hat family)][postgresql_dl]**


> **Tip:** 当前的版本为 **Version 10.3**

---

# 操作


## 环境

~~~
[root@h210 ~]# hostnamectl
   Static hostname: h210
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 739de39e0b1440618015b8dcd595f9f7
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@h210 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 82748sec preferred_lft 82748sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h210 ~]#
~~~


## 安装仓库

~~~
[root@h210 ~]# ll /etc/yum.repos.d/
total 36
-rw-r--r--. 1 root root 1664 11月 30 2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 2016 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 2016 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 2016 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 6月  16 2017 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 2016 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 2016 CentOS-Vault.repo
-rw-r--r--. 1 root root  957 12月 28 2016 epel.repo
-rw-r--r--. 1 root root 1056 12月 28 2016 epel-testing.repo
[root@h210 ~]# yum install yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
Loaded plugins: fastestmirror, langpacks
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 5.9 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/4): epel/x86_64/updateinfo                              | 899 kB   00:04     
(2/4): extras/7/x86_64/primary_db                          | 167 kB   00:04     
(3/4): epel/x86_64/primary_db                              | 6.3 MB   00:32     
(4/4): updates/7/x86_64/primary_db                         | 6.0 MB   00:48     
Determining fastest mirrors
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirrors.vinahost.vn
 * updates: mirrors.vinahost.vn
No package install available.
pgdg-centos10-10-2.noarch.rpm                            | 4.6 kB     00:00     
Examining /var/tmp/yum-root-RfPIvE/pgdg-centos10-10-2.noarch.rpm: pgdg-centos10-10-2.noarch
Marking /var/tmp/yum-root-RfPIvE/pgdg-centos10-10-2.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package pgdg-centos10.noarch 0:10-2 will be installed
---> Package yum.noarch 0:3.4.3-150.el7.centos will be updated
---> Package yum.noarch 0:3.4.3-154.el7.centos.1 will be an update
--> Processing Dependency: rpm >= 4.11.3-22 for package: yum-3.4.3-154.el7.centos.1.noarch
--> Running transaction check
---> Package rpm.x86_64 0:4.11.3-21.el7 will be updated
--> Processing Dependency: rpm = 4.11.3-21.el7 for package: rpm-libs-4.11.3-21.el7.x86_64
--> Processing Dependency: rpm = 4.11.3-21.el7 for package: rpm-python-4.11.3-21.el7.x86_64
---> Package rpm.x86_64 0:4.11.3-25.el7 will be an update
--> Running transaction check
---> Package rpm-libs.x86_64 0:4.11.3-21.el7 will be updated
--> Processing Dependency: rpm-libs(x86-64) = 4.11.3-21.el7 for package: rpm-build-libs-4.11.3-21.el7.x86_64
---> Package rpm-libs.x86_64 0:4.11.3-25.el7 will be an update
---> Package rpm-python.x86_64 0:4.11.3-21.el7 will be updated
---> Package rpm-python.x86_64 0:4.11.3-25.el7 will be an update
--> Running transaction check
---> Package rpm-build-libs.x86_64 0:4.11.3-21.el7 will be updated
---> Package rpm-build-libs.x86_64 0:4.11.3-25.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package        Arch   Version                 Repository                  Size
================================================================================
Installing:
 pgdg-centos10  noarch 10-2                    /pgdg-centos10-10-2.noarch 2.7 k
Updating:
 yum            noarch 3.4.3-154.el7.centos.1  updates                    1.2 M
Updating for dependencies:
 rpm            x86_64 4.11.3-25.el7           base                       1.2 M
 rpm-build-libs x86_64 4.11.3-25.el7           base                       104 k
 rpm-libs       x86_64 4.11.3-25.el7           base                       275 k
 rpm-python     x86_64 4.11.3-25.el7           base                        81 k

Transaction Summary
================================================================================
Install  1 Package
Upgrade  1 Package (+4 Dependent packages)

Total size: 2.8 M
Total download size: 2.8 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for base
updates/7/x86_64/prestodelta                               | 775 kB   00:03     
Delta RPMs reduced 1.2 M of updates to 86 k (93% saved)
(1/5): rpm-build-libs-4.11.3-25.el7.x86_64.rpm             | 104 kB   00:01     
(2/5): rpm-python-4.11.3-25.el7.x86_64.rpm                 |  81 kB   00:01     
(3/5): yum-3.4.3-150.el7.centos_3.4.3-154.el7.centos.1.noa |  86 kB   00:02     
(4/5): rpm-libs-4.11.3-25.el7.x86_64.rpm                   | 275 kB   00:03     
(5/5): rpm-4.11.3-25.el7.x86_64.rpm                        | 1.2 MB   00:06     
--------------------------------------------------------------------------------
Total                                              277 kB/s | 1.7 MB  00:06     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : rpm-libs-4.11.3-25.el7.x86_64                               1/11
  Updating   : rpm-4.11.3-25.el7.x86_64                                    2/11
  Updating   : rpm-build-libs-4.11.3-25.el7.x86_64                         3/11
  Updating   : rpm-python-4.11.3-25.el7.x86_64                             4/11
  Updating   : yum-3.4.3-154.el7.centos.1.noarch                           5/11
  Installing : pgdg-centos10-10-2.noarch                                   6/11
  Cleanup    : yum-3.4.3-150.el7.centos.noarch                             7/11
  Cleanup    : rpm-python-4.11.3-21.el7.x86_64                             8/11
  Cleanup    : rpm-build-libs-4.11.3-21.el7.x86_64                         9/11
  Cleanup    : rpm-4.11.3-21.el7.x86_64                                   10/11
  Cleanup    : rpm-libs-4.11.3-21.el7.x86_64                              11/11
  Verifying  : rpm-build-libs-4.11.3-25.el7.x86_64                         1/11
  Verifying  : rpm-4.11.3-25.el7.x86_64                                    2/11
  Verifying  : rpm-libs-4.11.3-25.el7.x86_64                               3/11
  Verifying  : yum-3.4.3-154.el7.centos.1.noarch                           4/11
  Verifying  : pgdg-centos10-10-2.noarch                                   5/11
  Verifying  : rpm-python-4.11.3-25.el7.x86_64                             6/11
  Verifying  : rpm-python-4.11.3-21.el7.x86_64                             7/11
  Verifying  : rpm-build-libs-4.11.3-21.el7.x86_64                         8/11
  Verifying  : rpm-4.11.3-21.el7.x86_64                                    9/11
  Verifying  : rpm-libs-4.11.3-21.el7.x86_64                              10/11
  Verifying  : yum-3.4.3-150.el7.centos.noarch                            11/11

Installed:
  pgdg-centos10.noarch 0:10-2                                                   

Updated:
  yum.noarch 0:3.4.3-154.el7.centos.1                                           

Dependency Updated:
  rpm.x86_64 0:4.11.3-25.el7          rpm-build-libs.x86_64 0:4.11.3-25.el7    
  rpm-libs.x86_64 0:4.11.3-25.el7     rpm-python.x86_64 0:4.11.3-25.el7        

Complete!
[root@h210 ~]# yum list all | grep postgresql
calligra-kexi-driver-postgresql.x86_64  2.9.10-1.el7                   epel     
collectd-postgresql.x86_64              5.8.0-1.el7                    epel     
freeradius-postgresql.x86_64            3.0.13-8.el7_4                 updates  
libreoffice-postgresql.x86_64           1:5.0.6.2-14.el7               base     
mingw32-postgresql.noarch               9.3.4-2.el7                    epel     
mingw64-postgresql.noarch               9.3.4-2.el7                    epel     
nextcloud-postgresql.noarch             10.0.4-2.el7                   epel     
opendbx-postgresql.x86_64               1.4.6-6.el7                    epel     
opensips-postgresql.x86_64              1.10.5-3.el7                   epel     
owncloud-postgresql.noarch              9.1.5-1.el7                    epel     
pcp-pmda-postgresql.x86_64              3.11.8-7.el7                   base     
pdns-backend-postgresql.x86_64          3.4.11-4.el7                   epel     
perdition-postgresql.x86_64             2.1-7.el7                      epel     
postgresql.i686                         9.2.23-3.el7_4                 updates  
postgresql.x86_64                       9.2.23-3.el7_4                 updates  
postgresql-contrib.x86_64               9.2.23-3.el7_4                 updates  
postgresql-devel.i686                   9.2.23-3.el7_4                 updates  
postgresql-devel.x86_64                 9.2.23-3.el7_4                 updates  
postgresql-docs.x86_64                  9.2.23-3.el7_4                 updates  
postgresql-jdbc.noarch                  42.2.1-1.rhel7                 pgdg10   
postgresql-jdbc-javadoc.noarch          42.2.1-1.rhel7                 pgdg10   
postgresql-libs.i686                    9.2.23-3.el7_4                 updates  
postgresql-libs.x86_64                  9.2.23-3.el7_4                 updates  
postgresql-odbc.x86_64                  09.03.0100-2.el7               base     
postgresql-pgpool-II.x86_64             3.4.6-1.el7                    epel     
postgresql-pgpool-II-devel.x86_64       3.4.6-1.el7                    epel     
postgresql-pgpool-II-extensions.x86_64  3.4.6-1.el7                    epel     
postgresql-plperl.x86_64                9.2.23-3.el7_4                 updates  
postgresql-plpython.x86_64              9.2.23-3.el7_4                 updates  
postgresql-plruby.x86_64                0.5.3-13.el7                   epel     
postgresql-plruby-doc.x86_64            0.5.3-13.el7                   epel     
postgresql-pltcl.x86_64                 9.2.23-3.el7_4                 updates  
postgresql-server.x86_64                9.2.23-3.el7_4                 updates  
postgresql-static.i686                  9.2.23-3.el7_4                 updates  
postgresql-static.x86_64                9.2.23-3.el7_4                 updates  
postgresql-test.x86_64                  9.2.23-3.el7_4                 updates  
postgresql-unit10.x86_64                3.1-1.rhel7                    pgdg10   
postgresql-unit10-debuginfo.x86_64      3.1-1.rhel7                    pgdg10   
postgresql-upgrade.x86_64               9.2.23-3.el7_4                 updates  
postgresql10.x86_64                     10.3-1PGDG.rhel7               pgdg10   
postgresql10-contrib.x86_64             10.3-1PGDG.rhel7               pgdg10   
postgresql10-debuginfo.x86_64           10.3-1PGDG.rhel7               pgdg10   
postgresql10-devel.x86_64               10.3-1PGDG.rhel7               pgdg10   
postgresql10-docs.x86_64                10.3-1PGDG.rhel7               pgdg10   
postgresql10-libs.x86_64                10.3-1PGDG.rhel7               pgdg10   
postgresql10-odbc.x86_64                10.01.0000-1PGDG.rhel7         pgdg10   
postgresql10-plperl.x86_64              10.3-1PGDG.rhel7               pgdg10   
postgresql10-plpython.x86_64            10.3-1PGDG.rhel7               pgdg10   
postgresql10-pltcl.x86_64               10.3-1PGDG.rhel7               pgdg10   
postgresql10-server.x86_64              10.3-1PGDG.rhel7               pgdg10   
postgresql10-tcl.x86_64                 2.4.0-1.rhel7                  pgdg10   
postgresql10-tcl-debuginfo.x86_64       2.3.1-1.rhel7                  pgdg10   
postgresql10-test.x86_64                10.3-1PGDG.rhel7               pgdg10   
proftpd-postgresql.x86_64               1.3.5e-4.el7                   epel     
python-testing.postgresql.noarch        1.1.0-2.el7                    epel     
qt-postgresql.i686                      1:4.8.5-15.el7_4               updates  
qt-postgresql.x86_64                    1:4.8.5-15.el7_4               updates  
qt5-qtbase-postgresql.i686              5.6.2-1.el7                    base     
qt5-qtbase-postgresql.x86_64            5.6.2-1.el7                    base     
soci-postgresql.x86_64                  3.2.3-1.el7                    epel     
soci-postgresql-devel.x86_64            3.2.3-1.el7                    epel     
[root@h210 ~]# ll /etc/yum.repos.d/
total 40
-rw-r--r--. 1 root root 1664 11月 30 2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 2016 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 2016 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 2016 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 6月  16 2017 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 2016 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 2016 CentOS-Vault.repo
-rw-r--r--. 1 root root  957 12月 28 2016 epel.repo
-rw-r--r--. 1 root root 1056 12月 28 2016 epel-testing.repo
-rw-r--r--. 1 root root 1004 9月  24 17:54 pgdg-10-centos.repo
[root@h210 ~]#
~~~


## 安装客户端

~~~
[root@h210 ~]# yum install postgresql10
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirrors.vinahost.vn
 * updates: mirrors.vinahost.vn
Resolving Dependencies
--> Running transaction check
---> Package postgresql10.x86_64 0:10.3-1PGDG.rhel7 will be installed
--> Processing Dependency: postgresql10-libs(x86-64) = 10.3-1PGDG.rhel7 for package: postgresql10-10.3-1PGDG.rhel7.x86_64
--> Processing Dependency: libpq.so.5()(64bit) for package: postgresql10-10.3-1PGDG.rhel7.x86_64
--> Running transaction check
---> Package postgresql10-libs.x86_64 0:10.3-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch         Version                Repository    Size
================================================================================
Installing:
 postgresql10            x86_64       10.3-1PGDG.rhel7       pgdg10       1.5 M
Installing for dependencies:
 postgresql10-libs       x86_64       10.3-1PGDG.rhel7       pgdg10       354 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 1.9 M
Installed size: 9.3 M
Is this ok [y/d/N]: y
Downloading packages:
(1/2): postgresql10-libs-10.3-1PGDG.rhel7.x86_64.rpm       | 354 kB   00:03     
(2/2): postgresql10-10.3-1PGDG.rhel7.x86_64.rpm            | 1.5 MB   00:09     
--------------------------------------------------------------------------------
Total                                              195 kB/s | 1.9 MB  00:09     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql10-libs-10.3-1PGDG.rhel7.x86_64                    1/2
  Installing : postgresql10-10.3-1PGDG.rhel7.x86_64                         2/2
  Verifying  : postgresql10-libs-10.3-1PGDG.rhel7.x86_64                    1/2
  Verifying  : postgresql10-10.3-1PGDG.rhel7.x86_64                         2/2

Installed:
  postgresql10.x86_64 0:10.3-1PGDG.rhel7                                        

Dependency Installed:
  postgresql10-libs.x86_64 0:10.3-1PGDG.rhel7                                   

Complete!
[root@h210 ~]# echo $?
0
[root@h210 ~]#
~~~



## 安装服务端

~~~
[root@h210 ~]# yum install postgresql10-server
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirrors.vinahost.vn
 * updates: mirrors.vinahost.vn
Resolving Dependencies
--> Running transaction check
---> Package postgresql10-server.x86_64 0:10.3-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                  Arch        Version                 Repository   Size
================================================================================
Installing:
 postgresql10-server      x86_64      10.3-1PGDG.rhel7        pgdg10      4.4 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 4.4 M
Installed size: 17 M
Is this ok [y/d/N]: y
Downloading packages:
postgresql10-server-10.3-1PGDG.rhel7.x86_64.rpm            | 4.4 MB   00:19     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql10-server-10.3-1PGDG.rhel7.x86_64                  1/1
  Verifying  : postgresql10-server-10.3-1PGDG.rhel7.x86_64                  1/1

Installed:
  postgresql10-server.x86_64 0:10.3-1PGDG.rhel7                                 

Complete!
[root@h210 ~]# echo $?
0
[root@h210 ~]#
~~~


## 初始化数据库

~~~
[root@h210 ~]# /usr/pgsql-10/bin/postgresql-10-setup initdb
Initializing database ... OK

[root@h210 ~]#
~~~


## 启动服务

~~~
[root@h210 ~]# systemctl status postgresql-10
● postgresql-10.service - PostgreSQL 10 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-10.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://www.postgresql.org/docs/10/static/
[root@h210 ~]# systemctl start postgresql-10
[root@h210 ~]# systemctl status postgresql-10
● postgresql-10.service - PostgreSQL 10 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-10.service; disabled; vendor preset: disabled)
   Active: active (running) since 二 2018-03-06 22:14:43 CST; 2s ago
     Docs: https://www.postgresql.org/docs/10/static/
  Process: 2478 ExecStartPre=/usr/pgsql-10/bin/postgresql-10-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 2484 (postmaster)
   CGroup: /system.slice/postgresql-10.service
           ├─2484 /usr/pgsql-10/bin/postmaster -D /var/lib/pgsql/10/data/
           ├─2486 postgres: logger process   
           ├─2488 postgres: checkpointer process   
           ├─2489 postgres: writer process   
           ├─2490 postgres: wal writer process   
           ├─2491 postgres: autovacuum launcher process   
           ├─2492 postgres: stats collector process   
           └─2493 postgres: bgworker: logical replication launcher   

3月 06 22:14:42 h210 systemd[1]: Starting PostgreSQL 10 database server...
3月 06 22:14:43 h210 postmaster[2484]: 2018-03-06 22:14:43.018 CST [2484] L...2
3月 06 22:14:43 h210 postmaster[2484]: 2018-03-06 22:14:43.018 CST [2484] L...2
3月 06 22:14:43 h210 postmaster[2484]: 2018-03-06 22:14:43.020 CST [2484] L..."
3月 06 22:14:43 h210 postmaster[2484]: 2018-03-06 22:14:43.023 CST [2484] L..."
3月 06 22:14:43 h210 postmaster[2484]: 2018-03-06 22:14:43.030 CST [2484] L...s
3月 06 22:14:43 h210 postmaster[2484]: 2018-03-06 22:14:43.030 CST [2484] H....
3月 06 22:14:43 h210 systemd[1]: Started PostgreSQL 10 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@h210 ~]#
[root@h210 ~]# ps faux | grep -i post
root      2498  0.0  0.0 112648  1028 pts/0    S+   22:15   0:00          \_ grep --color=auto -i post
root      1654  0.0  0.1  91120  2168 ?        Ss   21:38   0:00 /usr/libexec/postfix/master -w
postfix   1655  0.0  0.1  91224  3984 ?        S    21:38   0:00  \_ pickup -l -t unix -u
postfix   1656  0.0  0.1  91292  4004 ?        S    21:38   0:00  \_ qmgr -l -t unix -u
postgres  2484  0.0  0.8 391500 16488 ?        Ss   22:14   0:00 /usr/pgsql-10/bin/postmaster -D /var/lib/pgsql/10/data/
postgres  2486  0.0  0.0 246436  1944 ?        Ss   22:14   0:00  \_ postgres: logger process   
postgres  2488  0.0  0.1 391500  2168 ?        Ss   22:14   0:00  \_ postgres: checkpointer process   
postgres  2489  0.0  0.1 391500  2672 ?        Ss   22:14   0:00  \_ postgres: writer process   
postgres  2490  0.0  0.3 391500  6360 ?        Ss   22:14   0:00  \_ postgres: wal writer process   
postgres  2491  0.0  0.1 391908  3000 ?        Ss   22:14   0:00  \_ postgres: autovacuum launcher process   
postgres  2492  0.0  0.0 246432  2024 ?        Ss   22:14   0:00  \_ postgres: stats collector process   
postgres  2493  0.0  0.1 391792  2524 ?        Ss   22:14   0:00  \_ postgres: bgworker: logical replication launcher   
[root@h210 ~]#
[root@h210 ~]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1741/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1524/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1526/cupsd          
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      2484/postmaster     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1654/master         
tcp        0      0 192.168.56.210:22       192.168.56.1:45640      ESTABLISHED 2109/sshd: root@pts
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::22                   :::*                    LISTEN      1524/sshd           
tcp6       0      0 ::1:631                 :::*                    LISTEN      1526/cupsd          
tcp6       0      0 ::1:5432                :::*                    LISTEN      2484/postmaster     
tcp6       0      0 ::1:25                  :::*                    LISTEN      1654/master         
[root@h210 ~]#
~~~


## 访问数据库


~~~
[root@h210 ~]# createdb testdb
createdb: could not connect to database template1: FATAL:  role "root" does not exist
[root@h210 ~]# pwd
/root
[root@h210 ~]# su - postgres
-bash-4.2$ cd /var/lib/pgsql/10/data/
-bash-4.2$ ls
base              pg_commit_ts   pg_logical    pg_serial     pg_subtrans  pg_wal                postmaster.opts
current_logfiles  pg_dynshmem    pg_multixact  pg_snapshots  pg_tblspc    pg_xact               postmaster.pid
global            pg_hba.conf    pg_notify     pg_stat       pg_twophase  postgresql.auto.conf
log               pg_ident.conf  pg_replslot   pg_stat_tmp   PG_VERSION   postgresql.conf
-bash-4.2$ createdb testdb
-bash-4.2$ psql testdb
psql (10.3)
Type "help" for help.

testdb=# select version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 10.3 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-16), 64-bit
(1 row)

testdb=# \h
Available help:
  ABORT                            CREATE FOREIGN TABLE             DROP SCHEMA
  ALTER AGGREGATE                  CREATE FUNCTION                  DROP SEQUENCE
  ALTER COLLATION                  CREATE GROUP                     DROP SERVER
  ALTER CONVERSION                 CREATE INDEX                     DROP STATISTICS
  ALTER DATABASE                   CREATE LANGUAGE                  DROP SUBSCRIPTION
  ALTER DEFAULT PRIVILEGES         CREATE MATERIALIZED VIEW         DROP TABLE
  ALTER DOMAIN                     CREATE OPERATOR                  DROP TABLESPACE
  ALTER EVENT TRIGGER              CREATE OPERATOR CLASS            DROP TEXT SEARCH CONFIGURATION
  ALTER EXTENSION                  CREATE OPERATOR FAMILY           DROP TEXT SEARCH DICTIONARY
  ALTER FOREIGN DATA WRAPPER       CREATE POLICY                    DROP TEXT SEARCH PARSER
  ALTER FOREIGN TABLE              CREATE PUBLICATION               DROP TEXT SEARCH TEMPLATE
  ALTER FUNCTION                   CREATE ROLE                      DROP TRANSFORM
  ALTER GROUP                      CREATE RULE                      DROP TRIGGER
  ALTER INDEX                      CREATE SCHEMA                    DROP TYPE
  ALTER LANGUAGE                   CREATE SEQUENCE                  DROP USER
  ALTER LARGE OBJECT               CREATE SERVER                    DROP USER MAPPING
  ALTER MATERIALIZED VIEW          CREATE STATISTICS                DROP VIEW
  ALTER OPERATOR                   CREATE SUBSCRIPTION              END
  ALTER OPERATOR CLASS             CREATE TABLE                     EXECUTE
  ALTER OPERATOR FAMILY            CREATE TABLE AS                  EXPLAIN
  ALTER POLICY                     CREATE TABLESPACE                FETCH
  ALTER PUBLICATION                CREATE TEXT SEARCH CONFIGURATION GRANT
  ALTER ROLE                       CREATE TEXT SEARCH DICTIONARY    IMPORT FOREIGN SCHEMA
  ALTER RULE                       CREATE TEXT SEARCH PARSER        INSERT
  ALTER SCHEMA                     CREATE TEXT SEARCH TEMPLATE      LISTEN
  ALTER SEQUENCE                   CREATE TRANSFORM                 LOAD
  ALTER SERVER                     CREATE TRIGGER                   LOCK
  ALTER STATISTICS                 CREATE TYPE                      MOVE
  ALTER SUBSCRIPTION               CREATE USER                      NOTIFY
  ALTER SYSTEM                     CREATE USER MAPPING              PREPARE
  ALTER TABLE                      CREATE VIEW                      PREPARE TRANSACTION
  ALTER TABLESPACE                 DEALLOCATE                       REASSIGN OWNED
  ALTER TEXT SEARCH CONFIGURATION  DECLARE                          REFRESH MATERIALIZED VIEW
  ALTER TEXT SEARCH DICTIONARY     DELETE                           REINDEX
  ALTER TEXT SEARCH PARSER         DISCARD                          RELEASE SAVEPOINT
  ALTER TEXT SEARCH TEMPLATE       DO                               RESET
  ALTER TRIGGER                    DROP ACCESS METHOD               REVOKE
  ALTER TYPE                       DROP AGGREGATE                   ROLLBACK
  ALTER USER                       DROP CAST                        ROLLBACK PREPARED
  ALTER USER MAPPING               DROP COLLATION                   ROLLBACK TO SAVEPOINT
  ALTER VIEW                       DROP CONVERSION                  SAVEPOINT
  ANALYZE                          DROP DATABASE                    SECURITY LABEL
  BEGIN                            DROP DOMAIN                      SELECT
  CHECKPOINT                       DROP EVENT TRIGGER               SELECT INTO
  CLOSE                            DROP EXTENSION                   SET
  CLUSTER                          DROP FOREIGN DATA WRAPPER        SET CONSTRAINTS
  COMMENT                          DROP FOREIGN TABLE               SET ROLE
  COMMIT                           DROP FUNCTION                    SET SESSION AUTHORIZATION
  COMMIT PREPARED                  DROP GROUP                       SET TRANSACTION
  COPY                             DROP INDEX                       SHOW
  CREATE ACCESS METHOD             DROP LANGUAGE                    START TRANSACTION
  CREATE AGGREGATE                 DROP MATERIALIZED VIEW           TABLE
  CREATE CAST                      DROP OPERATOR                    TRUNCATE
  CREATE COLLATION                 DROP OPERATOR CLASS              UNLISTEN
  CREATE CONVERSION                DROP OPERATOR FAMILY             UPDATE
  CREATE DATABASE                  DROP OWNED                       VACUUM
  CREATE DOMAIN                    DROP POLICY                      VALUES
  CREATE EVENT TRIGGER             DROP PUBLICATION                 WITH
  CREATE EXTENSION                 DROP ROLE                        
  CREATE FOREIGN DATA WRAPPER      DROP RULE                        
testdb=#
testdb=#
testdb=# \q
-bash-4.2$
~~~



---

# 总结

由于 **[PostgreSQL][postgresql]** 准备好了相应的 yum 仓库，所以整个过程就是一个 rpm 包的安装过程

关于 PostgreSQL 的配置，后面有需要，再进行展开

* TOC
{:toc}


---


[postgresql]:https://www.postgresql.org/
[postgresql_dl]:https://www.postgresql.org/download/linux/redhat/
