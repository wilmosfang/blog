---
layout:  post
title:  安装 PostgreSQL 
author:  wilmosfang
tags: postgresql
categories:  postgresql 
wc: 934 3716 40830
excerpt:  Centos7 下安装 PostgreSQL 
comments: true
---


# 前言


**[PostgreSQL][postgresql]** 号称是这个世界上最高级的开源数据库

作为一个运维人员是怎样也无法忽视的

由于特性丰富，很多 CMDB 都是基于它开发的，再加上当前的物联网热潮，IOT 场景中大量涉及时空数据的处理，这些方面都是它的专长

>之前的 gitlab 可以对接 mysql 也可以对接 **[PostgreSQL][postgresql]** ，但是官网推荐使用 **[PostgreSQL][postgresql]** 作为其后端数据库，因为使用 **[PostgreSQL][postgresql]** 就可以使用所有的 gitlab 特性，而如果使用 mysql ，部分特性将会无法正常工作，我想应该是数据库层面的特性导致的这种差异吧，**[PostgreSQL][postgresql]** 有更为丰富的特性支持


这里对 **[PostgreSQL][postgresql]** 的安装做一个简单的演示，详细特性可以参考 **[PostgreSQL Documentation][postgresql_doc]** ，后期关于它的细节特性，再一点点展开

> **Tip:** 当前的最新稳定版为 **Aug. 10, 2017** 发布的 **PostgreSQL 9.6.4** ，同时 **PostgreSQL 10 Beta 3** 也在 **2017-08-10** 进行了发布

---

# 概要

* TOC
{:toc}



---


## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 902c440d59844169b696287b0d63d1e4
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:04:c7:5a brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85861sec preferred_lft 85861sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:b5:a5:da brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.206/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb5:a5da/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]# uname -a
Linux much 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@much ~]# 
~~~

---

## 目标

* Centos7 下安装 PostgreSQL 


---

## 安装 postgresql repo

**[PostgreSQL Core Distribution][postgresql_dl]** 里有不同平台下的 PostgreSQL 版本

这里根据我的具体环境选择 **[Linux downloads (Red Hat family)][postgresql_dl_redhat]**

根据提示选择合适的 **版本，平台和架构**

~~~
[root@much ~]# ll /etc/yum.repos.d/
total 32
-rw-r--r--. 1 root root 1664 11月 30 2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 2016 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 2016 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 2016 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 6月  16 01:08 CentOS-Media.repo
-rw-r--r--. 1 root root  884 5月  25 02:37 CentOS-OpenShift-Origin15.repo
-rw-r--r--. 1 root root 1331 11月 30 2016 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 2016 CentOS-Vault.repo
[root@much ~]# yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
Loaded plugins: fastestmirror, langpacks
pgdg-centos96-9.6-3.noarch.rpm                           | 4.7 kB     00:00     
Examining /var/tmp/yum-root-9Au8QL/pgdg-centos96-9.6-3.noarch.rpm: pgdg-centos96-9.6-3.noarch
Marking /var/tmp/yum-root-9Au8QL/pgdg-centos96-9.6-3.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package pgdg-centos96.noarch 0:9.6-3 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch       Version     Repository                       Size
================================================================================
Installing:
 pgdg-centos96     noarch     9.6-3       /pgdg-centos96-9.6-3.noarch     2.7 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 2.7 k
Installed size: 2.7 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : pgdg-centos96-9.6-3.noarch                                   1/1 
  Verifying  : pgdg-centos96-9.6-3.noarch                                   1/1 

Installed:
  pgdg-centos96.noarch 0:9.6-3                                                  

Complete!
[root@much ~]# ll /etc/yum.repos.d/
total 36
-rw-r--r--. 1 root root 1664 11月 30 2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 2016 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 2016 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 2016 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 6月  16 01:08 CentOS-Media.repo
-rw-r--r--. 1 root root  884 5月  25 02:37 CentOS-OpenShift-Origin15.repo
-rw-r--r--. 1 root root 1331 11月 30 2016 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 2016 CentOS-Vault.repo
-rw-r--r--. 1 root root 1012 9月  21 2016 pgdg-96-centos.repo
[root@much ~]# cat /etc/yum.repos.d/pgdg-96-centos.repo 
[pgdg96]
name=PostgreSQL 9.6 $releasever - $basearch
baseurl=https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-96

[pgdg96-source]
name=PostgreSQL 9.6 $releasever - $basearch - Source
failovermethod=priority
baseurl=https://download.postgresql.org/pub/repos/yum/srpms/9.6/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-96

[pgdg96-updates-testing]
name=PostgreSQL 9.6 $releasever - $basearch
baseurl=https://download.postgresql.org/pub/repos/yum/testing/9.6/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-96

[pgdg96-source-updates-testing]
name=PostgreSQL 9.6 $releasever - $basearch - Source
failovermethod=priority
baseurl=https://download.postgresql.org/pub/repos/yum/srpms/testing/9.6/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-96

[root@much ~]# 
~~~

之后就可以列出新版的 pgsql


~~~
[root@much ~]# yum list all | grep postgresql
freeradius-postgresql.x86_64                3.0.4-8.el7_3              updates  
libreoffice-postgresql.x86_64               1:5.0.6.2-5.el7_3.1        updates  
pcp-pmda-postgresql.x86_64                  3.11.3-4.el7               base     
postgresql.i686                             9.2.18-1.el7               base     
postgresql.x86_64                           9.2.18-1.el7               base     
postgresql-contrib.x86_64                   9.2.18-1.el7               base     
postgresql-devel.i686                       9.2.18-1.el7               base     
postgresql-devel.x86_64                     9.2.18-1.el7               base     
postgresql-docs.x86_64                      9.2.18-1.el7               base     
postgresql-jdbc.noarch                      42.1.4-1.rhel7             pgdg96   
postgresql-jdbc-javadoc.noarch              42.1.4-1.rhel7             pgdg96   
postgresql-libs.i686                        9.2.18-1.el7               base     
postgresql-libs.x86_64                      9.2.18-1.el7               base     
postgresql-odbc.x86_64                      09.03.0100-2.el7           base     
postgresql-plperl.x86_64                    9.2.18-1.el7               base     
postgresql-plpython.x86_64                  9.2.18-1.el7               base     
postgresql-pltcl.x86_64                     9.2.18-1.el7               base     
postgresql-server.x86_64                    9.2.18-1.el7               base     
postgresql-test.x86_64                      9.2.18-1.el7               base     
postgresql-unit96.x86_64                    3.1-1.rhel7                pgdg96   
postgresql-unit96-debuginfo.x86_64          3.1-1.rhel7                pgdg96   
postgresql-upgrade.x86_64                   9.2.18-1.el7               base     
postgresql96.x86_64                         9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-contrib.x86_64                 9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-debuginfo.x86_64               9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-devel.x86_64                   9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-docs.x86_64                    9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-libs.x86_64                    9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-odbc.x86_64                    09.06.0410-1PGDG.rhel7     pgdg96   
postgresql96-plperl.x86_64                  9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-plpython.x86_64                9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-pltcl.x86_64                   9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-server.x86_64                  9.6.4-1PGDG.rhel7          pgdg96   
postgresql96-tcl.x86_64                     2.3.1-1.rhel7              pgdg96   
postgresql96-tcl-debuginfo.x86_64           2.3.1-1.rhel7              pgdg96   
postgresql96-test.x86_64                    9.6.4-1PGDG.rhel7          pgdg96   
qt-postgresql.i686                          1:4.8.5-13.el7             base     
qt-postgresql.x86_64                        1:4.8.5-13.el7             base     
qt5-qtbase-postgresql.i686                  5.6.1-10.el7               base     
qt5-qtbase-postgresql.x86_64                5.6.1-10.el7               base     
[root@much ~]# 
~~~



---

## 安装客户端

~~~
[root@much ~]# yum install postgresql96
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * c7-media: 
 * extras: mirrors.163.com
 * updates: mirrors.sohu.com
Resolving Dependencies
--> Running transaction check
---> Package postgresql96.x86_64 0:9.6.4-1PGDG.rhel7 will be installed
--> Processing Dependency: postgresql96-libs(x86-64) = 9.6.4-1PGDG.rhel7 for package: postgresql96-9.6.4-1PGDG.rhel7.x86_64
--> Processing Dependency: libpq.so.5()(64bit) for package: postgresql96-9.6.4-1PGDG.rhel7.x86_64
--> Running transaction check
---> Package postgresql96-libs.x86_64 0:9.6.4-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch        Version                   Repository   Size
================================================================================
Installing:
 postgresql96           x86_64      9.6.4-1PGDG.rhel7         pgdg96      1.4 M
Installing for dependencies:
 postgresql96-libs      x86_64      9.6.4-1PGDG.rhel7         pgdg96      312 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 1.7 M
Installed size: 8.1 M
Is this ok [y/d/N]: y
Downloading packages:
(1/2): postgresql96-libs-9.6.4-1PGDG.rhel7.x86_64.rpm      | 312 kB   00:03     
(2/2): postgresql96-9.6.4-1PGDG.rhel7.x86_64.rpm           | 1.4 MB   00:04     
--------------------------------------------------------------------------------
Total                                              414 kB/s | 1.7 MB  00:04     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql96-libs-9.6.4-1PGDG.rhel7.x86_64                   1/2 
  Installing : postgresql96-9.6.4-1PGDG.rhel7.x86_64                        2/2 
  Verifying  : postgresql96-9.6.4-1PGDG.rhel7.x86_64                        1/2 
  Verifying  : postgresql96-libs-9.6.4-1PGDG.rhel7.x86_64                   2/2 

Installed:
  postgresql96.x86_64 0:9.6.4-1PGDG.rhel7                                       

Dependency Installed:
  postgresql96-libs.x86_64 0:9.6.4-1PGDG.rhel7                                  

Complete!
[root@much ~]# 
~~~

查看 **psql** 的版本

~~~
[root@much ~]# psql --help 
psql is the PostgreSQL interactive terminal.

Usage:
  psql [OPTION]... [DBNAME [USERNAME]]

General options:
  -c, --command=COMMAND    run only single command (SQL or internal) and exit
  -d, --dbname=DBNAME      database name to connect to (default: "root")
  -f, --file=FILENAME      execute commands from file, then exit
  -l, --list               list available databases, then exit
  -v, --set=, --variable=NAME=VALUE
                           set psql variable NAME to VALUE
                           (e.g., -v ON_ERROR_STOP=1)
  -V, --version            output version information, then exit
  -X, --no-psqlrc          do not read startup file (~/.psqlrc)
  -1 ("one"), --single-transaction
                           execute as a single transaction (if non-interactive)
  -?, --help[=options]     show this help, then exit
      --help=commands      list backslash commands, then exit
      --help=variables     list special variables, then exit

Input and output options:
  -a, --echo-all           echo all input from script
  -b, --echo-errors        echo failed commands
  -e, --echo-queries       echo commands sent to server
  -E, --echo-hidden        display queries that internal commands generate
  -L, --log-file=FILENAME  send session log to file
  -n, --no-readline        disable enhanced command line editing (readline)
  -o, --output=FILENAME    send query results to file (or |pipe)
  -q, --quiet              run quietly (no messages, only query output)
  -s, --single-step        single-step mode (confirm each query)
  -S, --single-line        single-line mode (end of line terminates SQL command)

Output format options:
  -A, --no-align           unaligned table output mode
  -F, --field-separator=STRING
                           field separator for unaligned output (default: "|")
  -H, --html               HTML table output mode
  -P, --pset=VAR[=ARG]     set printing option VAR to ARG (see \pset command)
  -R, --record-separator=STRING
                           record separator for unaligned output (default: newline)
  -t, --tuples-only        print rows only
  -T, --table-attr=TEXT    set HTML table tag attributes (e.g., width, border)
  -x, --expanded           turn on expanded table output
  -z, --field-separator-zero
                           set field separator for unaligned output to zero byte
  -0, --record-separator-zero
                           set record separator for unaligned output to zero byte

Connection options:
  -h, --host=HOSTNAME      database server host or socket directory (default: "local socket")
  -p, --port=PORT          database server port (default: "5432")
  -U, --username=USERNAME  database user name (default: "root")
  -w, --no-password        never prompt for password
  -W, --password           force password prompt (should happen automatically)

For more information, type "\?" (for internal commands) or "\help" (for SQL
commands) from within psql, or consult the psql section in the PostgreSQL
documentation.

Report bugs to <pgsql-bugs@postgresql.org>.
[root@much ~]# psql --version
psql (PostgreSQL) 9.6.4
[root@much ~]# 
~~~

查看一下多安装了哪些命令

~~~
[root@much ~]# rpm -ql  postgresql96-9.6.4-1PGDG.rhel7.x86_64 | grep bin
/usr/pgsql-9.6/bin/clusterdb
/usr/pgsql-9.6/bin/createdb
/usr/pgsql-9.6/bin/createlang
/usr/pgsql-9.6/bin/createuser
/usr/pgsql-9.6/bin/dropdb
/usr/pgsql-9.6/bin/droplang
/usr/pgsql-9.6/bin/dropuser
/usr/pgsql-9.6/bin/pg_archivecleanup
/usr/pgsql-9.6/bin/pg_basebackup
/usr/pgsql-9.6/bin/pg_config
/usr/pgsql-9.6/bin/pg_dump
/usr/pgsql-9.6/bin/pg_dumpall
/usr/pgsql-9.6/bin/pg_isready
/usr/pgsql-9.6/bin/pg_receivexlog
/usr/pgsql-9.6/bin/pg_restore
/usr/pgsql-9.6/bin/pg_rewind
/usr/pgsql-9.6/bin/pg_test_fsync
/usr/pgsql-9.6/bin/pg_test_timing
/usr/pgsql-9.6/bin/pg_upgrade
/usr/pgsql-9.6/bin/pg_xlogdump
/usr/pgsql-9.6/bin/pgbench
/usr/pgsql-9.6/bin/psql
/usr/pgsql-9.6/bin/reindexdb
/usr/pgsql-9.6/bin/vacuumdb
/usr/pgsql-9.6/share/man/man3/SPI_getbinval.3
[root@much ~]#
~~~


---

## 安装服务端

~~~
[root@much ~]# yum install postgresql96-server
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * c7-media: 
 * extras: mirrors.163.com
 * updates: mirrors.sohu.com
Resolving Dependencies
--> Running transaction check
---> Package postgresql96-server.x86_64 0:9.6.4-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                  Arch        Version                 Repository   Size
================================================================================
Installing:
 postgresql96-server      x86_64      9.6.4-1PGDG.rhel7       pgdg96      4.3 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 4.3 M
Installed size: 18 M
Is this ok [y/d/N]: y
Downloading packages:
postgresql96-server-9.6.4-1PGDG.rhel7.x86_64.rpm           | 4.3 MB   00:28     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql96-server-9.6.4-1PGDG.rhel7.x86_64                 1/1 
  Verifying  : postgresql96-server-9.6.4-1PGDG.rhel7.x86_64                 1/1 

Installed:
  postgresql96-server.x86_64 0:9.6.4-1PGDG.rhel7                                

Complete!
[root@much ~]# echo $?
0
[root@much ~]# 
~~~

查看一下多安装了哪些命令

~~~
[root@much ~]# rpm -qa | grep -i postgresql
postgresql96-9.6.4-1PGDG.rhel7.x86_64
postgresql96-libs-9.6.4-1PGDG.rhel7.x86_64
postgresql96-server-9.6.4-1PGDG.rhel7.x86_64
[root@much ~]# rpm -ql postgresql96-server-9.6.4-1PGDG.rhel7.x86_64 | grep bin
/usr/pgsql-9.6/bin/initdb
/usr/pgsql-9.6/bin/pg_controldata
/usr/pgsql-9.6/bin/pg_ctl
/usr/pgsql-9.6/bin/pg_resetxlog
/usr/pgsql-9.6/bin/postgres
/usr/pgsql-9.6/bin/postgresql96-check-db-dir
/usr/pgsql-9.6/bin/postgresql96-setup
/usr/pgsql-9.6/bin/postmaster
[root@much ~]# 
~~~

---

## 初始化数据库

~~~
[root@much ~]# /usr/pgsql-9.6/bin/postgresql96-setup initdb
Initializing database ... OK

[root@much ~]# 
~~~


---

## 开启服务


~~~
[root@much ~]# systemctl enable postgresql-9.6
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-9.6.service to /usr/lib/systemd/system/postgresql-9.6.service.
[root@much ~]# systemctl status postgresql-9.6
● postgresql-9.6.service - PostgreSQL 9.6 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-9.6.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
[root@much ~]# systemctl start postgresql-9.6
[root@much ~]# systemctl status postgresql-9.6
● postgresql-9.6.service - PostgreSQL 9.6 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-9.6.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2017-08-22 18:30:22 CST; 2s ago
  Process: 2855 ExecStartPre=/usr/pgsql-9.6/bin/postgresql96-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 2861 (postmaster)
   CGroup: /system.slice/postgresql-9.6.service
           ├─2861 /usr/pgsql-9.6/bin/postmaster -D /var/lib/pgsql/9.6/data/
           ├─2863 postgres: logger process   
           ├─2865 postgres: checkpointer process   
           ├─2866 postgres: writer process   
           ├─2867 postgres: wal writer process   
           ├─2868 postgres: autovacuum launcher process   
           └─2869 postgres: stats collector process   

8月 22 18:30:22 much systemd[1]: Starting PostgreSQL 9.6 database server...
8月 22 18:30:22 much postmaster[2861]: < 2017-08-22 18:30:22.410 CST > LOG:...s
8月 22 18:30:22 much postmaster[2861]: < 2017-08-22 18:30:22.410 CST > HINT....
8月 22 18:30:22 much systemd[1]: Started PostgreSQL 9.6 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]# 
~~~

此时系统里 PostgreSQL 已经运行起来了

~~~
[root@much ~]# ps faux | grep postgres
root      2886  0.0  0.0 112648  1008 pts/1    S+   18:31   0:00          \_ grep --color=auto postgres
postgres  2861  0.0  0.3 357632 15252 ?        Ss   18:30   0:00 /usr/pgsql-9.6/bin/postmaster -D /var/lib/pgsql/9.6/data/
postgres  2863  0.0  0.0 212636  1568 ?        Ss   18:30   0:00  \_ postgres: logger process   
postgres  2865  0.0  0.0 357632  1908 ?        Ss   18:30   0:00  \_ postgres: checkpointer process   
postgres  2866  0.0  0.0 357632  2440 ?        Ss   18:30   0:00  \_ postgres: writer process   
postgres  2867  0.0  0.0 357632  1680 ?        Ss   18:30   0:00  \_ postgres: wal writer process   
postgres  2868  0.0  0.0 357948  2796 ?        Ss   18:30   0:00  \_ postgres: autovacuum launcher process   
postgres  2869  0.0  0.0 212632  1812 ?        Ss   18:30   0:00  \_ postgres: stats collector process   
[root@much ~]#
~~~

从以上输出中可以获知以下信息

* 数据文件在 **`/var/lib/pgsql/9.6/data/`**
* 启动了一个日志进程
* 启动了一个检查点进程
* 启动了一个写操作进程
* 启动了一个 WAL 写进程
* 启动了一个 autovacuum 进程
* 启动了一个统计信息收集进程

查看一下数据文件　

~~~
[root@much data]# ll /var/lib/pgsql/9.6/data/
total 56
drwx------. 5 postgres postgres    41 8月  22 18:28 base
drwx------. 2 postgres postgres  4096 8月  22 18:30 global
drwx------. 2 postgres postgres    18 8月  22 18:28 pg_clog
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_commit_ts
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_dynshmem
-rw-------. 1 postgres postgres  4224 8月  22 18:28 pg_hba.conf
-rw-------. 1 postgres postgres  1636 8月  22 18:28 pg_ident.conf
drwx------. 2 postgres postgres    32 8月  22 18:30 pg_log
drwx------. 4 postgres postgres    39 8月  22 18:28 pg_logical
drwx------. 4 postgres postgres    36 8月  22 18:28 pg_multixact
drwx------. 2 postgres postgres    18 8月  22 18:30 pg_notify
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_replslot
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_serial
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_snapshots
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_stat
drwx------. 2 postgres postgres    25 8月  22 18:37 pg_stat_tmp
drwx------. 2 postgres postgres    18 8月  22 18:28 pg_subtrans
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_tblspc
drwx------. 2 postgres postgres     6 8月  22 18:28 pg_twophase
-rw-------. 1 postgres postgres     4 8月  22 18:28 PG_VERSION
drwx------. 3 postgres postgres    60 8月  22 18:28 pg_xlog
-rw-------. 1 postgres postgres    88 8月  22 18:28 postgresql.auto.conf
-rw-------. 1 postgres postgres 22305 8月  22 18:28 postgresql.conf
-rw-------. 1 postgres postgres    60 8月  22 18:30 postmaster.opts
-rw-------. 1 postgres postgres    95 8月  22 18:30 postmaster.pid
[root@much data]# tail -n 3 /etc/passwd
vboxadd:x:987:1::/var/run/vboxadd:/bin/false
dockerroot:x:986:981:Docker User:/var/lib/docker:/sbin/nologin
postgres:x:26:26:PostgreSQL Server:/var/lib/pgsql:/bin/bash
[root@much data]# 
~~~

安装的过程中还顺便创建了一个叫 postgres 的用户，用于进程的运行身份


---

## 创建数据库


~~~
[root@much data]# pwd
/var/lib/pgsql/9.6/data
[root@much data]# ls
base          pg_hba.conf    pg_notify     pg_stat_tmp  pg_xlog
global        pg_ident.conf  pg_replslot   pg_subtrans  postgresql.auto.conf
pg_clog       pg_log         pg_serial     pg_tblspc    postgresql.conf
pg_commit_ts  pg_logical     pg_snapshots  pg_twophase  postmaster.opts
pg_dynshmem   pg_multixact   pg_stat       PG_VERSION   postmaster.pid
[root@much data]# createdb mydb
createdb: could not connect to database template1: FATAL:  role "root" does not exist
[root@much data]# su - postgres
-bash-4.2$ createdb mydb
-bash-4.2$ cd /var/lib/pgsql/9.6/data
-bash-4.2$ ls
base          pg_hba.conf    pg_notify     pg_stat_tmp  pg_xlog
global        pg_ident.conf  pg_replslot   pg_subtrans  postgresql.auto.conf
pg_clog       pg_log         pg_serial     pg_tblspc    postgresql.conf
pg_commit_ts  pg_logical     pg_snapshots  pg_twophase  postmaster.opts
pg_dynshmem   pg_multixact   pg_stat       PG_VERSION   postmaster.pid
-bash-4.2$ 
~~~

pgsql 里的用户体系是独立的，由于识别不了 root 用户所以报错


---


## 访问数据库


~~~
-bash-4.2$ psql mydb
psql (9.6.4)
Type "help" for help.

mydb=# 
~~~

**`#`** 代表我现在是数据库的超级管理员

执行一些简单的查询

~~~
mydb=# select version();
                                                 version                        
                          
--------------------------------------------------------------------------------
--------------------------
 PostgreSQL 9.6.4 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (
Red Hat 4.8.5-11), 64-bit
(1 row)

mydb=# select current_date;
    date    
------------
 2017-08-22
(1 row)

mydb=# select 3+3;
 ?column? 
----------
        6
(1 row)

mydb=# 
~~~


**`\h`**  可以获取帮助

退出 psql 用 **`\q`**

~~~
mydb=# \h
Available help:
  ABORT                            ALTER SERVER                     CREATE COLLATION                 CREATE TEXT SEARCH CONFIGURATION DROP FOREIGN TABLE               DROP USER                        ROLLBACK TO SAVEPOINT
  ALTER AGGREGATE                  ALTER SYSTEM                     CREATE CONVERSION                CREATE TEXT SEARCH DICTIONARY    DROP FUNCTION                    DROP USER MAPPING                SAVEPOINT
  ALTER COLLATION                  ALTER TABLE                      CREATE DATABASE                  CREATE TEXT SEARCH PARSER        DROP GROUP                       DROP VIEW                        SECURITY LABEL
  ALTER CONVERSION                 ALTER TABLESPACE                 CREATE DOMAIN                    CREATE TEXT SEARCH TEMPLATE      DROP INDEX                       END                              SELECT
  ALTER DATABASE                   ALTER TEXT SEARCH CONFIGURATION  CREATE EVENT TRIGGER             CREATE TRANSFORM                 DROP LANGUAGE                    EXECUTE                          SELECT INTO
  ALTER DEFAULT PRIVILEGES         ALTER TEXT SEARCH DICTIONARY     CREATE EXTENSION                 CREATE TRIGGER                   DROP MATERIALIZED VIEW           EXPLAIN                          SET
  ALTER DOMAIN                     ALTER TEXT SEARCH PARSER         CREATE FOREIGN DATA WRAPPER      CREATE TYPE                      DROP OPERATOR                    FETCH                            SET CONSTRAINTS
  ALTER EVENT TRIGGER              ALTER TEXT SEARCH TEMPLATE       CREATE FOREIGN TABLE             CREATE USER                      DROP OPERATOR CLASS              GRANT                            SET ROLE
  ALTER EXTENSION                  ALTER TRIGGER                    CREATE FUNCTION                  CREATE USER MAPPING              DROP OPERATOR FAMILY             IMPORT FOREIGN SCHEMA            SET SESSION AUTHORIZATION
  ALTER FOREIGN DATA WRAPPER       ALTER TYPE                       CREATE GROUP                     CREATE VIEW                      DROP OWNED                       INSERT                           SET TRANSACTION
  ALTER FOREIGN TABLE              ALTER USER                       CREATE INDEX                     DEALLOCATE                       DROP POLICY                      LISTEN                           SHOW
  ALTER FUNCTION                   ALTER USER MAPPING               CREATE LANGUAGE                  DECLARE                          DROP ROLE                        LOAD                             START TRANSACTION
  ALTER GROUP                      ALTER VIEW                       CREATE MATERIALIZED VIEW         DELETE                           DROP RULE                        LOCK                             TABLE
  ALTER INDEX                      ANALYZE                          CREATE OPERATOR                  DISCARD                          DROP SCHEMA                      MOVE                             TRUNCATE
  ALTER LANGUAGE                   BEGIN                            CREATE OPERATOR CLASS            DO                               DROP SEQUENCE                    NOTIFY                           UNLISTEN
  ALTER LARGE OBJECT               CHECKPOINT                       CREATE OPERATOR FAMILY           DROP ACCESS METHOD               DROP SERVER                      PREPARE                          UPDATE
  ALTER MATERIALIZED VIEW          CLOSE                            CREATE POLICY                    DROP AGGREGATE                   DROP TABLE                       PREPARE TRANSACTION              VACUUM
  ALTER OPERATOR                   CLUSTER                          CREATE ROLE                      DROP CAST                        DROP TABLESPACE                  REASSIGN OWNED                   VALUES
  ALTER OPERATOR CLASS             COMMENT                          CREATE RULE                      DROP COLLATION                   DROP TEXT SEARCH CONFIGURATION   REFRESH MATERIALIZED VIEW        WITH
  ALTER OPERATOR FAMILY            COMMIT                           CREATE SCHEMA                    DROP CONVERSION                  DROP TEXT SEARCH DICTIONARY      REINDEX                          
  ALTER POLICY                     COMMIT PREPARED                  CREATE SEQUENCE                  DROP DATABASE                    DROP TEXT SEARCH PARSER          RELEASE SAVEPOINT                
  ALTER ROLE                       COPY                             CREATE SERVER                    DROP DOMAIN                      DROP TEXT SEARCH TEMPLATE        RESET                            
  ALTER RULE                       CREATE ACCESS METHOD             CREATE TABLE                     DROP EVENT TRIGGER               DROP TRANSFORM                   REVOKE                           
  ALTER SCHEMA                     CREATE AGGREGATE                 CREATE TABLE AS                  DROP EXTENSION                   DROP TRIGGER                     ROLLBACK                         
  ALTER SEQUENCE                   CREATE CAST                      CREATE TABLESPACE                DROP FOREIGN DATA WRAPPER        DROP TYPE                        ROLLBACK PREPARED                
mydb=# 
mydb=# \q
-bash-4.2$ 
~~~

其它的一些帮助信息

~~~
-bash-4.2$ psql mydb
psql (9.6.4)
Type "help" for help.

mydb=# help
You are using psql, the command-line interface to PostgreSQL.
Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit
mydb=# \copyright
PostgreSQL Database Management System
(formerly known as Postgres, then as Postgres95)

Portions Copyright (c) 1996-2016, PostgreSQL Global Development Group

Portions Copyright (c) 1994, The Regents of the University of California

Permission to use, copy, modify, and distribute this software and its
documentation for any purpose, without fee, and without a written agreement
is hereby granted, provided that the above copyright notice and this
paragraph and the following two paragraphs appear in all copies.

IN NO EVENT SHALL THE UNIVERSITY OF CALIFORNIA BE LIABLE TO ANY PARTY FOR
DIRECT, INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES, INCLUDING
LOST PROFITS, ARISING OUT OF THE USE OF THIS SOFTWARE AND ITS
DOCUMENTATION, EVEN IF THE UNIVERSITY OF CALIFORNIA HAS BEEN ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.

THE UNIVERSITY OF CALIFORNIA SPECIFICALLY DISCLAIMS ANY WARRANTIES,
INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
AND FITNESS FOR A PARTICULAR PURPOSE.  THE SOFTWARE PROVIDED HEREUNDER IS
ON AN "AS IS" BASIS, AND THE UNIVERSITY OF CALIFORNIA HAS NO OBLIGATIONS TO
PROVIDE MAINTENANCE, SUPPORT, UPDATES, ENHANCEMENTS, OR MODIFICATIONS.

mydb=# \?
General
  \copyright             show PostgreSQL usage and distribution terms
  \errverbose            show most recent error message at maximum verbosity
  \g [FILE] or ;         execute query (and send results to file or |pipe)
  \gexec                 execute query, then execute each value in its result
  \gset [PREFIX]         execute query and store results in psql variables
  \q                     quit psql
  \crosstabview [COLUMNS] execute query and display results in crosstab
  \watch [SEC]           execute query every SEC seconds

Help
  \? [commands]          show help on backslash commands
  \? options             show help on psql command-line options
  \? variables           show help on special variables
  \h [NAME]              help on syntax of SQL commands, * for all commands

Query Buffer
  \e [FILE] [LINE]       edit the query buffer (or file) with external editor
  \ef [FUNCNAME [LINE]]  edit function definition with external editor
  \ev [VIEWNAME [LINE]]  edit view definition with external editor
  \p                     show the contents of the query buffer
  \r                     reset (clear) the query buffer
  \s [FILE]              display history or save it to file
  \w FILE                write query buffer to file

Input/Output
  \copy ...              perform SQL COPY with data stream to the client host
  \echo [STRING]         write string to standard output
  \i FILE                execute commands from file
  \ir FILE               as \i, but relative to location of current script
  \o [FILE]              send all query results to file or |pipe
  \qecho [STRING]        write string to query output stream (see \o)

Informational
  (options: S = show system objects, + = additional detail)
  \d[S+]                 list tables, views, and sequences
  \d[S+]  NAME           describe table, view, sequence, or index
  \da[S]  [PATTERN]      list aggregates
  \dA[+]  [PATTERN]      list access methods
  \db[+]  [PATTERN]      list tablespaces
  \dc[S+] [PATTERN]      list conversions
  \dC[+]  [PATTERN]      list casts
  \dd[S]  [PATTERN]      show object descriptions not displayed elsewhere
  \ddp    [PATTERN]      list default privileges
  \dD[S+] [PATTERN]      list domains
  \det[+] [PATTERN]      list foreign tables
  \des[+] [PATTERN]      list foreign servers
  \deu[+] [PATTERN]      list user mappings
  \dew[+] [PATTERN]      list foreign-data wrappers
  \df[antw][S+] [PATRN]  list [only agg/normal/trigger/window] functions
  \dF[+]  [PATTERN]      list text search configurations
  \dFd[+] [PATTERN]      list text search dictionaries
  \dFp[+] [PATTERN]      list text search parsers
  \dFt[+] [PATTERN]      list text search templates
  \dg[S+] [PATTERN]      list roles
  \di[S+] [PATTERN]      list indexes
  \dl                    list large objects, same as \lo_list
  \dL[S+] [PATTERN]      list procedural languages
  \dm[S+] [PATTERN]      list materialized views
  \dn[S+] [PATTERN]      list schemas
  \do[S]  [PATTERN]      list operators
  \dO[S+] [PATTERN]      list collations
  \dp     [PATTERN]      list table, view, and sequence access privileges
  \drds [PATRN1 [PATRN2]] list per-database role settings
  \ds[S+] [PATTERN]      list sequences
  \dt[S+] [PATTERN]      list tables
  \dT[S+] [PATTERN]      list data types
  \du[S+] [PATTERN]      list roles
  \dv[S+] [PATTERN]      list views
  \dE[S+] [PATTERN]      list foreign tables
  \dx[+]  [PATTERN]      list extensions
  \dy     [PATTERN]      list event triggers
  \l[+]   [PATTERN]      list databases
  \sf[+]  FUNCNAME       show a function's definition
  \sv[+]  VIEWNAME       show a view's definition
  \z      [PATTERN]      same as \dp

Formatting
  \a                     toggle between unaligned and aligned output mode
  \C [STRING]            set table title, or unset if none
  \f [STRING]            show or set field separator for unaligned query output
  \H                     toggle HTML output mode (currently off)
  \pset [NAME [VALUE]]   set table output option
                         (NAME := {format|border|expanded|fieldsep|fieldsep_zero|footer|null|
                         numericlocale|recordsep|recordsep_zero|tuples_only|title|tableattr|pager|
                         unicode_border_linestyle|unicode_column_linestyle|unicode_header_linestyle})
  \t [on|off]            show only rows (currently off)
  \T [STRING]            set HTML <table> tag attributes, or unset if none
  \x [on|off|auto]       toggle expanded output (currently off)

Connection
  \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "mydb")
  \encoding [ENCODING]   show or set client encoding
  \password [USERNAME]   securely change the password for a user
  \conninfo              display information about current connection

Operating System
  \cd [DIR]              change the current working directory
  \setenv NAME [VALUE]   set or unset environment variable
  \timing [on|off]       toggle timing of commands (currently off)
  \! [COMMAND]           execute command in shell or start interactive shell

Variables
  \prompt [TEXT] NAME    prompt user to set internal variable
  \set [NAME [VALUE]]    set internal variable, or list all if no parameters
  \unset NAME            unset (delete) internal variable

Large Objects
  \lo_export LOBOID FILE
  \lo_import FILE [COMMENT]
  \lo_list
  \lo_unlink LOBOID      large object operations
mydb=# 
mydb=# \g
~~~

要获取某一个命令的帮助信息

~~~
mydb=# \h copy
Command:     COPY
Description: copy data between a file and a table
Syntax:
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | PROGRAM 'command' | STDIN }
    [ [ WITH ] ( option [, ...] ) ]

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
    TO { 'filename' | PROGRAM 'command' | STDOUT }
    [ [ WITH ] ( option [, ...] ) ]

where option can be one of:

    FORMAT format_name
    OIDS [ boolean ]
    FREEZE [ boolean ]
    DELIMITER 'delimiter_character'
    NULL 'null_string'
    HEADER [ boolean ]
    QUOTE 'quote_character'
    ESCAPE 'escape_character'
    FORCE_QUOTE { ( column_name [, ...] ) | * }
    FORCE_NOT_NULL ( column_name [, ...] )
    FORCE_NULL ( column_name [, ...] )
    ENCODING 'encoding_name'

mydb=# \h show
Command:     SHOW
Description: show the value of a run-time parameter
Syntax:
SHOW name
SHOW ALL

mydb=#
~~~



---

# 命令汇总

* **`yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm`**
* **`ll /etc/yum.repos.d/`**
* **`cat /etc/yum.repos.d/pgdg-96-centos.repo`**
* **`yum list all | grep postgresql`**
* **`yum install postgresql96`**
* **`psql --help`**
* **`psql --version`**
* **`rpm -ql  postgresql96-9.6.4-1PGDG.rhel7.x86_64 | grep bin`**
* **`yum install postgresql96-server`**
* **`rpm -qa | grep -i postgresql`**
* **`rpm -ql postgresql96-server-9.6.4-1PGDG.rhel7.x86_64 | grep bin`**
* **`/usr/pgsql-9.6/bin/postgresql96-setup initdb`**
* **`systemctl enable postgresql-9.6`**
* **`systemctl start postgresql-9.6`**
* **`systemctl status postgresql-9.6`**
* **`ps faux | grep postgres`**
* **`ll /var/lib/pgsql/9.6/data/`**
* **`tail -n 3 /etc/passwd`**
* **`createdb mydb`**
* **`su - postgres`**


---

[postgresql]:https://www.postgresql.org/
[postgresql_dl]:https://www.postgresql.org/download/
[postgresql_dl_redhat]:https://www.postgresql.org/download/linux/redhat/
[postgresql_doc]:https://www.postgresql.org/docs/
