---
layout: post
title:  mysql 清理 relay log 和 bin log
author: wilmosfang
categories:    mysql
tags:    mysql mha 
wc: 255   982 13311 
excerpt: mysql relay log 和 bin log 的清理方法 
comments: true
---



# 前言

使用过 **Mysql mha** 的都知道，为了确保在故障切换的时候，有尽量多的数据用于恢复，**mha** 是建议关闭 **relay_log** 自动清理功能的

这个功能默认是开启的，因为一般情况下已经被 **SQL Thread** 执行过的 **Relay** 日志是没有价值的，但是对于 **mha** 来说有用，因为它可以从多个 **slave** 的 **Relay** 日志中提取更接近 **原master** 的操作加以重放来尽量减少数据的丢失，如果自动清理 **Relay** 日志的状态为开启的，那么在进行 **mha** 集群构建的过程中是会产生警告的，所以为了安全，还是要关闭

~~~
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
~~~

但是关闭自动清理是有代价的，最主要的就是，太消耗磁盘空间了，需要定期清理

如果手动来清理，就很麻烦，幸好这里有一个很好用的工具 **purge_relay_logs**，下面简单介绍一下它的用法

> **Tip:** mha当前的最新版本为 **MHA 0.56**

---


# 概要

* TOC
{:toc}


---

## 日志很多


这里除了relay log 外，还有很多 bin log

>**Tip:** 可以通过 **mysqladmin flush-logs** 来生成很多日志

~~~
[root@h102 data]# cd mysql/
[root@h102 mysql]# ls
taobao_db         mysql-bin.000033    relay-bin.000043  relay-bin.000093  relay-bin.000143  relay-bin.000193
auto.cnf          mysql-bin.000034    relay-bin.000044  relay-bin.000094  relay-bin.000144  relay-bin.000194
stack_db          mysql-bin.000035    relay-bin.000045  relay-bin.000095  relay-bin.000145  relay-bin.000195
pptv_db           mysql-bin.000036    relay-bin.000046  relay-bin.000096  relay-bin.000146  relay-bin.000196
h102.err          mysql-bin.000037    relay-bin.000047  relay-bin.000097  relay-bin.000147  relay-bin.000197
h102.pid          mysql-bin.index     relay-bin.000048  relay-bin.000098  relay-bin.000148  relay-bin.000198
h102-slow.log     mysql.sock          relay-bin.000049  relay-bin.000099  relay-bin.000149  relay-bin.000199
guanglan_db       my_test             relay-bin.000050  relay-bin.000100  relay-bin.000150  relay-bin.000200
sale_db           jingdong_db         relay-bin.000051  relay-bin.000101  relay-bin.000151  relay-bin.000201
ibdata1           performance_schema  relay-bin.000052  relay-bin.000102  relay-bin.000152  relay-bin.000202
ib_logfile0       relay-bin.000003    relay-bin.000053  relay-bin.000103  relay-bin.000153  relay-bin.000203
ib_logfile1       relay-bin.000004    relay-bin.000054  relay-bin.000104  relay-bin.000154  relay-bin.000204
ib_logfile2       relay-bin.000005    relay-bin.000055  relay-bin.000105  relay-bin.000155  relay-bin.000205
duang_db          relay-bin.000006    relay-bin.000056  relay-bin.000106  relay-bin.000156  relay-bin.000206
51job_db          relay-bin.000007    relay-bin.000057  relay-bin.000107  relay-bin.000157  relay-bin.000207
master.info       relay-bin.000008    relay-bin.000058  relay-bin.000108  relay-bin.000158  relay-bin.000208
gateway_db        relay-bin.000009    relay-bin.000059  relay-bin.000109  relay-bin.000159  relay-bin.000209
mysql             relay-bin.000010    relay-bin.000060  relay-bin.000110  relay-bin.000160  relay-bin.000210
mysql-bin.000001  relay-bin.000011    relay-bin.000061  relay-bin.000111  relay-bin.000161  relay-bin.000211
mysql-bin.000002  relay-bin.000012    relay-bin.000062  relay-bin.000112  relay-bin.000162  relay-bin.000212
...
...
mysql-bin.000023  relay-bin.000033    relay-bin.000083  relay-bin.000133  relay-bin.000183  relay-bin.000233
mysql-bin.000024  relay-bin.000034    relay-bin.000084  relay-bin.000134  relay-bin.000184  relay-bin.index
mysql-bin.000025  relay-bin.000035    relay-bin.000085  relay-bin.000135  relay-bin.000185  relay-log.info
mysql-bin.000026  relay-bin.000036    relay-bin.000086  relay-bin.000136  relay-bin.000186  sina_db
mysql-bin.000027  relay-bin.000037    relay-bin.000087  relay-bin.000137  relay-bin.000187  test
mysql-bin.000028  relay-bin.000038    relay-bin.000088  relay-bin.000138  relay-bin.000188  xtrabackup_binlog_pos_innodb
mysql-bin.000029  relay-bin.000039    relay-bin.000089  relay-bin.000139  relay-bin.000189  xtrabackup_info
mysql-bin.000030  relay-bin.000040    relay-bin.000090  relay-bin.000140  relay-bin.000190
mysql-bin.000031  relay-bin.000041    relay-bin.000091  relay-bin.000141  relay-bin.000191
mysql-bin.000032  relay-bin.000042    relay-bin.000092  relay-bin.000142  relay-bin.000192
[root@h102 mysql]#
~~~

---

##  清理 bin log


清理 bin log 相对简单，我之前有写过一篇专门介绍以各种姿势清 bin log 的博客，有兴趣的可以翻一翻

~~~
[root@h102 mysql]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 159467
Server version: 5.6.27-75.0-log Percona Server (GPL), Release 75.0, Revision 8bb53b6

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  show binary logs;
+------------------+------------+
| Log_name         | File_size  |
+------------------+------------+
| mysql-bin.000001 | 1108156504 |
| mysql-bin.000002 | 1107993599 |
...
...
| mysql-bin.000036 | 1097494872 |
| mysql-bin.000037 |   80418017 |
+------------------+------------+
37 rows in set (0.00 sec)

mysql> purge master logs before date_sub(now(),interval 14 day);
Query OK, 0 rows affected (13.06 sec)

mysql>  show binary logs;
+------------------+------------+
| Log_name         | File_size  |
+------------------+------------+
| mysql-bin.000032 | 1149743087 |
| mysql-bin.000033 | 1150147671 |
| mysql-bin.000034 | 1153148442 |
| mysql-bin.000035 | 1409805642 |
| mysql-bin.000036 | 1097494872 |
| mysql-bin.000037 |   80418017 |
+------------------+------------+
6 rows in set (0.00 sec)

mysql> quit
Bye
[root@h102 mysql]# ls
taobao_db           relay-bin.000019  relay-bin.000064  relay-bin.000109  relay-bin.000154  relay-bin.000199
auto.cnf            relay-bin.000020  relay-bin.000065  relay-bin.000110  relay-bin.000155  relay-bin.000200
stack_db            relay-bin.000021  relay-bin.000066  relay-bin.000111  relay-bin.000156  relay-bin.000201
pptv_db             relay-bin.000022  relay-bin.000067  relay-bin.000112  relay-bin.000157  relay-bin.000202
h102.err            relay-bin.000023  relay-bin.000068  relay-bin.000113  relay-bin.000158  relay-bin.000203
h102.pid            relay-bin.000024  relay-bin.000069  relay-bin.000114  relay-bin.000159  relay-bin.000204
h102-slow.log       relay-bin.000025  relay-bin.000070  relay-bin.000115  relay-bin.000160  relay-bin.000205
guanglan_db         relay-bin.000026  relay-bin.000071  relay-bin.000116  relay-bin.000161  relay-bin.000206
sale_db             relay-bin.000027  relay-bin.000072  relay-bin.000117  relay-bin.000162  relay-bin.000207
ibdata1             relay-bin.000028  relay-bin.000073  relay-bin.000118  relay-bin.000163  relay-bin.000208
ib_logfile0         relay-bin.000029  relay-bin.000074  relay-bin.000119  relay-bin.000164  relay-bin.000209
ib_logfile1         relay-bin.000030  relay-bin.000075  relay-bin.000120  relay-bin.000165  relay-bin.000210
ib_logfile2         relay-bin.000031  relay-bin.000076  relay-bin.000121  relay-bin.000166  relay-bin.000211
duang_db            relay-bin.000032  relay-bin.000077  relay-bin.000122  relay-bin.000167  relay-bin.000212
51job_db            relay-bin.000033  relay-bin.000078  relay-bin.000123  relay-bin.000168  relay-bin.000213
master.info         relay-bin.000034  relay-bin.000079  relay-bin.000124  relay-bin.000169  relay-bin.000214
gateway_db          relay-bin.000035  relay-bin.000080  relay-bin.000125  relay-bin.000170  relay-bin.000215
mysql               relay-bin.000036  relay-bin.000081  relay-bin.000126  relay-bin.000171  relay-bin.000216
mysql-bin.000032    relay-bin.000037  relay-bin.000082  relay-bin.000127  relay-bin.000172  relay-bin.000217
...
...
mysql-bin.000037    relay-bin.000042  relay-bin.000087  relay-bin.000132  relay-bin.000177  relay-bin.000222
mysql-bin.index     relay-bin.000043  relay-bin.000088  relay-bin.000133  relay-bin.000178  relay-bin.000223
mysql.sock          relay-bin.000044  relay-bin.000089  relay-bin.000134  relay-bin.000179  relay-bin.000224
my_test             relay-bin.000045  relay-bin.000090  relay-bin.000135  relay-bin.000180  relay-bin.000225
jingdong_db         relay-bin.000046  relay-bin.000091  relay-bin.000136  relay-bin.000181  relay-bin.000226
performance_schema  relay-bin.000047  relay-bin.000092  relay-bin.000137  relay-bin.000182  relay-bin.000227
relay-bin.000003    relay-bin.000048  relay-bin.000093  relay-bin.000138  relay-bin.000183  relay-bin.000228
...
...
relay-bin.000008    relay-bin.000053  relay-bin.000098  relay-bin.000143  relay-bin.000188  relay-bin.000233
relay-bin.000009    relay-bin.000054  relay-bin.000099  relay-bin.000144  relay-bin.000189  relay-bin.index
relay-bin.000010    relay-bin.000055  relay-bin.000100  relay-bin.000145  relay-bin.000190  relay-log.info
relay-bin.000011    relay-bin.000056  relay-bin.000101  relay-bin.000146  relay-bin.000191  sina_db
relay-bin.000012    relay-bin.000057  relay-bin.000102  relay-bin.000147  relay-bin.000192  test
relay-bin.000013    relay-bin.000058  relay-bin.000103  relay-bin.000148  relay-bin.000193  xtrabackup_binlog_pos_innodb
relay-bin.000014    relay-bin.000059  relay-bin.000104  relay-bin.000149  relay-bin.000194  xtrabackup_info
relay-bin.000015    relay-bin.000060  relay-bin.000105  relay-bin.000150  relay-bin.000195
relay-bin.000016    relay-bin.000061  relay-bin.000106  relay-bin.000151  relay-bin.000196
relay-bin.000017    relay-bin.000062  relay-bin.000107  relay-bin.000152  relay-bin.000197
relay-bin.000018    relay-bin.000063  relay-bin.000108  relay-bin.000153  relay-bin.000198
[root@h102 mysql]#
~~~

现在只剩下 relay log 要清理了

---

## 清理 relay log


~~~
[root@h102 data]# purge_relay_logs --user=root --password=xxxxxx --workdir=/data/relay_tmp/
2016-04-14 22:31:59: purge_relay_logs script started.
 Found relay_log.info: /var/lib/mysql/relay-log.info
 Removing hard linked relay log files relay-bin* under /data/relay_tmp/.. done.
 Current relay log file: /var/lib/mysql/relay-bin.000233
 Archiving unused relay log files (up to /var/lib/mysql/relay-bin.000232) ...
 Creating hard link for /var/lib/mysql/relay-bin.000003 under /data/relay_tmp//relay-bin.000003 .. ok.
 Creating hard link for /var/lib/mysql/relay-bin.000004 under /data/relay_tmp//relay-bin.000004 .. ok.
 ...
 ...
 Creating hard link for /var/lib/mysql/relay-bin.000231 under /data/relay_tmp//relay-bin.000231 .. ok.
 Creating hard link for /var/lib/mysql/relay-bin.000232 under /data/relay_tmp//relay-bin.000232 .. ok.
 Creating hard links for unused relay log files completed.
 Executing SET GLOBAL relay_log_purge=1; FLUSH LOGS; sleeping a few seconds so that SQL thread can delete older relay log files (if it keeps up); SET GLOBAL relay_log_purge=0; .. ok.
 Removing hard linked relay log files relay-bin* under /data/relay_tmp/.. done.
2016-04-14 22:32:25: All relay log purging operations succeeded.
[root@h102 data]#
[root@h102 data]# cd mysql/
[root@h102 mysql]# ls
taobao_db   h102-slow.log    ib_logfile2  mysql-bin.000032  mysql-bin.index     relay-bin.000234              xtrabackup_info
auto.cnf    guanglan_db      duang_db     mysql-bin.000033  mysql.sock          relay-bin.index
stack_db    sale_db          51job_db     mysql-bin.000034  my_test             relay-log.info
pptv_db     ibdata1          master.info  mysql-bin.000035  jingdong_db         sina_db
h102.err    ib_logfile0      gateway_db   mysql-bin.000036  performance_schema  test
h102.pid    ib_logfile1      mysql        mysql-bin.000037  relay-bin.000233    xtrabackup_binlog_pos_innodb
[root@h102 mysql]#
~~~


> **Note:**  这里有一点要非常注意，根据 **purge_relay_logs** 的工作原理 **workdir** 必须得和 **mysql** 的**datadir** 在同一个分区上，因为它是通过硬链接的方式来进行中转操作的，而硬链接是要求在相同分区上的


> **Tip:** **purge_relay_logs** 只是一个perl 脚本，由 **mha** 的包提供的


~~~
[root@h102 mysql]# which purge_relay_logs
/usr/bin/purge_relay_logs
[root@h102 mysql]# rpm -qf /usr/bin/purge_relay_logs
mha4mysql-node-0.53-0.el6.noarch
[root@h102 mysql]# file /usr/bin/purge_relay_logs
/usr/bin/purge_relay_logs: a /usr/bin/env perl script text executable
[root@h102 mysql]# wc /usr/bin/purge_relay_logs
 252  809 7401 /usr/bin/purge_relay_logs
[root@h102 mysql]# 
~~~



---


# 命令汇总


* **`purge_relay_logs --user=root --password=xxxxxx --workdir=/data/relay_tmp/`**
* **`which purge_relay_logs`**
* **`rpm -qf /usr/bin/purge_relay_logs`**
* **`file /usr/bin/purge_relay_logs`**


