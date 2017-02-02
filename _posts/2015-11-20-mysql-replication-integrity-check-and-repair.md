---
layout: post
title: Mysql复制数据一致性检查
author: wilmosfang
tags:  mysql admintools 
categories:  mysql 
wc: 310 1193 16168
excerpt: follow me
comments: true
---


---

# 前言


**[Percona Toolkit][percona-toolkit]** 是一系列进行mysql管理的工具，强大而高效，可以完成很多复杂的工作，是mysql数据库运维工程师居家旅行必备的良品。

> **Tip:** 它们是由 **Maatkit** 和 **Aspersa** 演化而来，由 **[percona][percona]** 收集整理和维护而成

其中有两个特别有用的工具 **[pt-table-checksum][pt-table-checksum]** 和 **[pt-table-sync][pt-table-sync]** ，分别可以用来进行主从一致性检查，和不一致数据修复

下面分享一下Mysql复制数据一致性检查的基本操作，详细可以参阅 [官方文档][percona-toolkit]


 > **Tip:** 目前官方版本是 **Percona Toolkit 2.2.16** 
  

---


# 概要

* TOC
{:toc}




---

## 下载安装Percona Toolkit

~~~
[root@replication-check-vm src]# wget  http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm
--2015-11-19 21:21:06--  http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm
Resolving www.percona.com... 74.121.199.234
Connecting to www.percona.com|74.121.199.234|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm [following]
--2015-11-19 21:21:08--  https://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm
Connecting to www.percona.com|74.121.199.234|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6566 (6.4K) [application/x-redhat-package-manager]
Saving to: `percona-release-0.1-3.noarch.rpm'


100%[===============================================================================>] 6,566       --.-K/s   in 0.002s  

2015-11-19 21:21:09 (4.12 MB/s) - `percona-release-0.1-3.noarch.rpm' saved [6566/6566]

[root@replication-check-vm src]# rpm -ivh percona-release-0.1-3.noarch.rpm 
warning: percona-release-0.1-3.noarch.rpm: Header V4 DSA signature: NOKEY, key ID cd2efd2a
Preparing...                ########################################### [100%]
   1:percona-release        ########################################### [100%]
[root@replication-check-vm src]# 
[root@replication-check-vm src]# yum list all | grep toolkit
percona-toolkit.noarch                     2.2.9-1                     installed
percona-toolkit.noarch                     2.2.16-1                    percona-release-noarch
translate-toolkit.noarch                   1.9.0-1.el5                 epel     
translate-toolkit-devel.noarch             1.9.0-1.el5                 epel     
[root@replication-check-vm src]# yum update  percona-toolkit.noarch  
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * epel: mirrors.opencas.cn
 * extras: mirrors.163.com
 * updates: mirrors.163.com
Setting up Update Process
Resolving Dependencies
--> Running transaction check
---> Package percona-toolkit.noarch 0:2.2.16-1 set to be updated
--> Processing Dependency: perl(Term::ReadKey) for package: percona-toolkit
--> Running transaction check
---> Package perl-TermReadKey.x86_64 0:2.30-4.el5 set to be updated
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================
 Package                        Arch                 Version                  Repository                            Size
=========================================================================================================================
Updating:
 percona-toolkit                noarch               2.2.16-1                 percona-release-noarch               1.6 M
Installing for dependencies:
 perl-TermReadKey               x86_64               2.30-4.el5               epel                                  32 k

Transaction Summary
=========================================================================================================================
Install       1 Package(s)
Upgrade       1 Package(s)

Total download size: 1.7 M
Is this ok [y/N]: y
Downloading Packages:

(1/2): perl-TermReadKey-2.30-4.el5.x86_64.rpm                                                     |  32 kB     00:00     
(2/2): percona-toolkit-2.2.16-1.noarch.rpm  (3%)  1% [                                 ]  0.0 B/s |  24 kB     --:-- ETA 
(2/2): percona-toolkit-2.2.16-1.noarch.rpm  (8%)  7% [==                               ] 115 kB/s | 120 kB     00:13 ETA 
(2/2): percona-toolkit-2.2.16-1.noarch.rpm (20%) 18% [======                           ] 143 kB/s | 312 kB     00:09 ETA 
(2/2): percona-toolkit-2.2.16-1.noarch.rpm (43%) 42% [=============-                   ] 213 kB/s | 704 kB     00:04 ETA 
(2/2): percona-toolkit-2.2.16-1.noarch.rpm (62%) 61% [====================             ] 265 kB/s | 1.0 MB     00:02 ETA 

(2/2): percona-toolkit-2.2.16-1.noarch.rpm                                                        | 1.6 MB     00:02     
-------------------------------------------------------------------------------------------------------------------------
Total                                                                                    173 kB/s | 1.7 MB     00:09     
warning: rpmts_HdrFromFdno: Header V4 DSA signature: NOKEY, key ID cd2efd2a

percona-release-noarch/gpgkey                                                                     | 1.7 kB     00:00     
Importing GPG key 0xCD2EFD2A "Percona MySQL Development Team <mysql-dev@percona.com>" from /etc/pki/rpm-gpg/RPM-GPG-KEY-Percona
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Finished Transaction Test
Transaction Test Succeeded
Running Transaction

  Installing     : perl-TermReadKey [                                                                              ] 1/3
...
...
  Installing     : perl-TermReadKey [############################################################################# ] 1/3
  Installing     : perl-TermReadKey                                                                                  1/3 

  Updating       : percona-toolkit [                                                                               ] 2/3
...
...
  Updating       : percona-toolkit [############################################################################## ] 2/3
  Updating       : percona-toolkit                                                                                   2/3 

  Cleanup        : percona-toolkit                                                                                   3/3 

Dependency Installed:
  perl-TermReadKey.x86_64 0:2.30-4.el5                                                                                   

Updated:
  percona-toolkit.noarch 0:2.2.16-1                                                                                      

Complete!
[root@replication-check-vm src]#
~~~

---

## 检查数据不一致


使用 **[pt-table-checksum][pt-table-checksum]** 进行不一致数据检查


> **pt-table-checksum** performs an online replication consistency check by executing checksum queries on the master, which produces different results on replicas that are inconsistent with the master. The optional DSN specifies the master host. The tool’s “EXIT STATUS” is non-zero if any differences are found, or if any warnings or errors occur.
> This tool is focused on finding data differences efficiently. If any data is different, you can resolve the problem with **pt-table-sync**.

检查示例

~~~
[mysql@replication-check-vm ~]$ pt-table-checksum --nocheck-replication-filters --nocheck-binlog-format --replicate=test.checksum --tables abc_test_db.users  h=replication-check-vm,u=root --ask-pass  
Enter MySQL password: 
Checksumming abc_test_db.users:   9% 05:00 remain
Checksumming abc_test_db.users:  18% 04:42 remain
Checksumming abc_test_db.users:  29% 03:48 remain
Checksumming abc_test_db.users:  37% 03:26 remain
Checksumming abc_test_db.users:  45% 03:03 remain
Checksumming abc_test_db.users:  51% 02:57 remain
Checksumming abc_test_db.users:  56% 02:51 remain
Checksumming abc_test_db.users:  61% 02:36 remain
Checksumming abc_test_db.users:  65% 02:30 remain
Checksumming abc_test_db.users:  69% 02:18 remain
Checksumming abc_test_db.users:  74% 01:59 remain
Checksumming abc_test_db.users:  80% 01:34 remain
Checksumming abc_test_db.users:  85% 01:11 remain
Checksumming abc_test_db.users:  90% 00:47 remain
Checksumming abc_test_db.users:  94% 00:25 remain
Checksumming abc_test_db.users:  99% 00:01 remain
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
11-19T22:02:30      0    103 14879748     154       0 594.225 abc_test_db.users
[mysql@replication-check-vm ~]$
~~~

Option	| Comment
-------- | ---
`--nocheck-replication-filters` | 如果有复制过滤就不进行检查
`--nocheck-binlog-format`  | 不对binlog_format进行检查
`--replicate`| 将结果存入指定的表中
`--tables` | 指定要检查的表，可以是一个列表，使用逗号分割
`--databases` | 指定要检查的库，可以是一个列表，使用逗号分割
`h=xxx,u=xxx` | DSN选项,h代表host,u代表用户名,使用逗号分割
`--ask-pass`| 使用提示密码的方式连接数据库，而不是使用DSN指定，更安全


> **Note:** 此时的DSN要指定master，并且首先要解决对于数据库的读写权限问题
> 
> **Tip:** 可以一次指定多个表进行检查，中间使用逗号分隔

~~~
[mysql@replication-check-vm ~]$ pt-table-checksum --nocheck-replication-filters --nocheck-binlog-format --replicate=test.checksum --tables user_key_db.user_schema,user_key_db.log_records    h=replication-check-vm,u=root --ask-pass  
Enter MySQL password: 
Checksumming user_key_db.log_records:   3% 14:14 remain
Checksumming user_key_db.log_records:   7% 13:18 remain
Checksumming user_key_db.log_records:  10% 13:07 remain
Checksumming user_key_db.log_records:  13% 12:51 remain
Checksumming user_key_db.log_records:  17% 12:18 remain
Checksumming user_key_db.log_records:  20% 11:53 remain
Checksumming user_key_db.log_records:  24% 11:13 remain
Checksumming user_key_db.log_records:  27% 10:30 remain
Checksumming user_key_db.log_records:  31% 09:53 remain
Checksumming user_key_db.log_records:  35% 09:21 remain
Checksumming user_key_db.log_records:  38% 08:49 remain
Checksumming user_key_db.log_records:  42% 08:16 remain
Checksumming user_key_db.log_records:  45% 07:48 remain
Checksumming user_key_db.log_records:  48% 07:29 remain
Checksumming user_key_db.log_records:  51% 07:11 remain
Checksumming user_key_db.log_records:  54% 06:45 remain
Checksumming user_key_db.log_records:  57% 06:16 remain
Checksumming user_key_db.log_records:  60% 05:55 remain
Checksumming user_key_db.log_records:  63% 05:36 remain
Checksumming user_key_db.log_records:  66% 05:13 remain
Checksumming user_key_db.log_records:  68% 04:50 remain
Checksumming user_key_db.log_records:  71% 04:22 remain
Checksumming user_key_db.log_records:  74% 03:55 remain
Checksumming user_key_db.log_records:  78% 03:23 remain
Checksumming user_key_db.log_records:  81% 02:53 remain
Checksumming user_key_db.log_records:  84% 02:25 remain
Checksumming user_key_db.log_records:  87% 01:55 remain
Checksumming user_key_db.log_records:  90% 01:26 remain
Checksumming user_key_db.log_records:  93% 00:58 remain
Checksumming user_key_db.log_records:  97% 00:26 remain
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
11-19T23:01:39      0     20 57822684     448       0 943.371 user_key_db.log_records
Checksumming user_key_db.user_schema:   7% 06:04 remain
Checksumming user_key_db.user_schema:  14% 06:20 remain
Checksumming user_key_db.user_schema:  18% 06:50 remain
Checksumming user_key_db.user_schema:  24% 06:21 remain
Checksumming user_key_db.user_schema:  30% 05:58 remain
Checksumming user_key_db.user_schema:  35% 05:30 remain
Checksumming user_key_db.user_schema:  42% 04:55 remain
Checksumming user_key_db.user_schema:  46% 04:37 remain
Checksumming user_key_db.user_schema:  52% 04:12 remain
Checksumming user_key_db.user_schema:  57% 03:45 remain
Checksumming user_key_db.user_schema:  62% 03:22 remain
Checksumming user_key_db.user_schema:  66% 03:06 remain
Checksumming user_key_db.user_schema:  70% 02:44 remain
Checksumming user_key_db.user_schema:  74% 02:28 remain
Checksumming user_key_db.user_schema:  77% 02:12 remain
Checksumming user_key_db.user_schema:  80% 01:57 remain
Checksumming user_key_db.user_schema:  83% 01:43 remain
Checksumming user_key_db.user_schema:  86% 01:24 remain
Checksumming user_key_db.user_schema:  90% 00:58 remain
Checksumming user_key_db.user_schema:  96% 00:22 remain
11-19T23:12:05      0    133 13877382     212       0 626.470 user_key_db.user_schema
[mysql@replication-check-vm ~]$
~~~


---

## 修复数据不一致

使用 **[pt-table-sync][pt-table-sync]** 来进行数据不一致修复

> **pt-table-sync** synchronizes data efficiently between MySQL tables.
>
> This tool changes data, so for maximum safety,  **you should back up your data before using it** . When synchronizing a server that is a replication slave with the --replicate or --sync-to-master methods,  **it always makes the changes on the replication master, never the replication slave directly** . This is in general the only safe way to bring a replica back in sync with its master; changes to the replica are usually the source of the problems in the first place. However,  **the changes it makes on the master should be no-op changes that set the data to their current values, and actually affect only the replica**.

**pt-table-sync** 是有可能改变数据的，所以要非常慎重，三思而后行

它并非在slave上执行操作，修改不一致数据，而是在master上重新插入当前值，通过同步而影响slave，从而解决数据的不一致(指定用户需要足读写权限)


> **Note:**  **pt-table-sync** is mature, proven in the real world, and well tested, but if used improperly it can have adverse consequences. Always test syncing first with `--dry-run` and `--print`.


~~~
[mysql@replication-check-vm ~]$ pt-table-sync --replicate test.checksum  h=slave-vm,u=root --ask-pass  --sync-to-master --databases=abc_test_db  --tables=users  --print > /tmp/users.sql
Enter password for slave-vm:       
[mysql@replication-check-vm ~]$ 
~~~

Option	| Comment
-------- | ---
`--replicate`| 参考指定检查表中的结果来进行修复，通常是pt-table-checksum生成的
`h=xxx,u=xxx` | DSN选项,h代表host,u代表用户名,使用逗号分割
`--ask-pass`| 使用提示密码的方式连接数据库，而不是使用DSN指定，更安全
`--sync-to-master`  | 将指定的DSN当作slave，尝试与它的master同步
`--databases` | 限定检查的DB范围，可以使用逗号分割的方式指定一个db列表
`--tables` |限定检查的table范围，可以使用逗号分割的方式指定一个table列表
`--print`| 不直接执行(相对于`--execute`)，而是输出解决不一致的SQL语句
`--execute`| 直接执行变更，使数据一致，会导致数据改变


一般我都会使用 `--print` 将结果导出到一个文本中，查看修改的内容进行确认，然后使用`--execute`自动执行，或手动执行其中部分SQL

执行完修改后，再重复上面的操作检查一次

这个工具好在，业务在线时可以执行，不会对系统造成很大影响(会产生一定量的读IO，但不会产生有明显业务影响的锁)，特别是在大表，核心表数据一致性检查时太能解渴了

---

[percona-toolkit]:https://www.percona.com/doc/percona-toolkit/2.2/index.html
[percona]:https://www.percona.com/
[pt-table-checksum]:https://www.percona.com/doc/percona-toolkit/2.2/pt-table-checksum.html
[pt-table-sync]:https://www.percona.com/doc/percona-toolkit/2.2/pt-table-sync.html

