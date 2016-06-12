---
layout: post
title:  Mysql binlog 查看方法
author: wilmosfang
tags:   mysql troubleshooting
categories:   mysql 
wc: 775  5326 85771 
excerpt: mysql binlog 的查看方法，show binlog events 和 mysqlbinlog， 这两种方法的操作示例与详解还有注意事项
comments: true
---


# 前言


Mysql 的运维工作中，因为排错的需要，有时我们会对过往的修改操作进行查看，mysql binlog 的机制正好可以应对这类需求

这里我分享一下查看 mysql binlog 的相关基础，详细可以参考 **[SHOW BINLOG EVENTS][show_binlog_events]** 和 **[Utility for Processing Binary Log Files][mysqlbinlog]**


> **Tip:**   当前的最新版本为 **Mysql 5.7** ，这里实验使用 **Percona Server version: 5.6.27-76.0**

---


# 概要

* TOC
{:toc}



---


## 环境


~~~
[root@h105 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h105 ~]# uname -a
Linux h105 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h105 ~]#
[root@h105 ~]# mysql --version
mysql  Ver 14.14 Distrib 5.6.27-76.0, for Linux (x86_64) using  6.0
[root@h105 ~]# 
~~~

---

## 前提

要查看 binlog 的详细信息，必须先打开 binlog 日志

~~~
[root@h105 ~]# grep bin /etc/my.cnf
log-bin=mysql-bin
relay-log=relay-bin
[root@h105 ~]#
[root@h105 ~]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.6.27-76.0-log Percona Server (GPL), Release 76.0, Revision 5498987

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like '%log_bin%';
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             |
| log_bin_basename                | /var/lib/mysql/mysql-bin       |
| log_bin_index                   | /var/lib/mysql/mysql-bin.index |
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
6 rows in set (0.00 sec)

mysql> 
~~~

查看本地的 binlog 日志

~~~
mysql> SHOW BINARY LOGS;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       120 |
+------------------+-----------+
1 row in set (0.00 sec)

mysql>
~~~

也可以直接列出

~~~
mysql> \! ls /var/lib/mysql/*bin*
/var/lib/mysql/mysql-bin.000001  /var/lib/mysql/mysql-bin.index
mysql> \! cat /var/lib/mysql/mysql-bin.index
./mysql-bin.000001
mysql> 
~~~

---

## SHOW BINLOG EVENTS 

~~~
mysql> use testxxx;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_testxxx |
+-------------------+
| test              |
+-------------------+
1 row in set (0.00 sec)

mysql> select * from test;
+------+----------+
| id   | name     |
+------+----------+
|    1 | hello1   |
|    2 | hello2   |
|    3 | hello3   |
|    4 | hello4   |
|    5 | hello5   |
...
...
|   97 | hello97  |
|   98 | hello98  |
|   99 | hello99  |
|  100 | hello100 |
+------+----------+
99 rows in set (0.01 sec)

mysql> SHOW BINLOG EVENTS;
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                       |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4 |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
1 row in set (0.00 sec)

mysql> delete from test where id=100;
Query OK, 1 row affected (0.10 sec)

mysql> SHOW BINLOG EVENTS;
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                         |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4   |
| mysql-bin.000001 | 120 | Query       |         1 |         205 | BEGIN                                        |
| mysql-bin.000001 | 205 | Query       |         1 |         314 | use `testxxx`; delete from test where id=100 |
| mysql-bin.000001 | 314 | Xid         |         1 |         345 | COMMIT /* xid=15 */                          |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
4 rows in set (0.03 sec)

mysql> 
~~~


再更新一条数据

~~~
mysql> update test set name = 'change99' where id=99;
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SHOW BINLOG EVENTS;
+------------------+------+-------------+-----------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name         | Pos  | Event_type  | Server_id | End_log_pos | Info                                                                                                                                                                                                                                                                                                                                   |
+------------------+------+-------------+-----------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| mysql-bin.000001 |    4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4                                                                                                                                                                                                                                                                                             |
| mysql-bin.000001 |  120 | Query       |         1 |         205 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 |  205 | Query       |         1 |         314 | use `testxxx`; delete from test where id=100                                                                                                                                                                                                                                                                                           |
| mysql-bin.000001 |  314 | Xid         |         1 |         345 | COMMIT /* xid=15 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 |  345 | Query       |         1 |         428 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 |  428 | Intvar      |         1 |         460 | INSERT_ID=1584                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 |  460 | Query       |         1 |         773 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612140001', '20160612140001', 1, 'CROND[5870]:')                                                                              |
| mysql-bin.000001 |  773 | Xid         |         1 |         804 | COMMIT /* xid=17 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 |  804 | Query       |         1 |         887 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 |  887 | Intvar      |         1 |         919 | INSERT_ID=1585                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 |  919 | Query       |         1 |        1237 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (run-parts /etc/cron.hourly)', 9, 'h105', 6, '20160612140101', '20160612140101', 1, 'CROND[5877]:')                                                                         |
| mysql-bin.000001 | 1237 | Xid         |         1 |        1268 | COMMIT /* xid=18 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 1268 | Query       |         1 |        1351 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 1351 | Intvar      |         1 |        1383 | INSERT_ID=1586                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 1383 | Query       |         1 |        1701 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' starting 0anacron', 9, 'h105', 5, '20160612140101', '20160612140101', 1, 'run-parts(/etc/cron.hourly)[5877]:')                                                                         |
| mysql-bin.000001 | 1701 | Xid         |         1 |        1732 | COMMIT /* xid=19 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 1732 | Query       |         1 |        1815 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 1815 | Intvar      |         1 |        1847 | INSERT_ID=1587                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 1847 | Query       |         1 |        2165 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' finished 0anacron', 9, 'h105', 5, '20160612140101', '20160612140101', 1, 'run-parts(/etc/cron.hourly)[5886]:')                                                                         |
| mysql-bin.000001 | 2165 | Xid         |         1 |        2196 | COMMIT /* xid=20 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 2196 | Query       |         1 |        2279 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 2279 | Intvar      |         1 |        2311 | INSERT_ID=1588                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 2311 | Query       |         1 |        2624 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612141001', '20160612141001', 1, 'CROND[5902]:')                                                                              |
| mysql-bin.000001 | 2624 | Xid         |         1 |        2655 | COMMIT /* xid=21 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 2655 | Query       |         1 |        2738 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 2738 | Intvar      |         1 |        2770 | INSERT_ID=1589                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 2770 | Query       |         1 |        3083 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612142001', '20160612142001', 1, 'CROND[5922]:')                                                                              |
| mysql-bin.000001 | 3083 | Xid         |         1 |        3114 | COMMIT /* xid=22 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 3114 | Query       |         1 |        3197 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 3197 | Intvar      |         1 |        3229 | INSERT_ID=1590                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 3229 | Query       |         1 |        3542 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612143001', '20160612143001', 1, 'CROND[5942]:')                                                                              |
| mysql-bin.000001 | 3542 | Xid         |         1 |        3573 | COMMIT /* xid=23 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 3573 | Query       |         1 |        3656 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 3656 | Intvar      |         1 |        3688 | INSERT_ID=1591                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 3688 | Query       |         1 |        4001 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612144001', '20160612144001', 1, 'CROND[5962]:')                                                                              |
| mysql-bin.000001 | 4001 | Xid         |         1 |        4032 | COMMIT /* xid=24 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 4032 | Query       |         1 |        4115 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 4115 | Intvar      |         1 |        4147 | INSERT_ID=1592                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 4147 | Query       |         1 |        4537 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' Address 192.168.100.1 maps to localhost, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!', 10, 'h105', 6, '20160612144840', '20160612144840', 1, 'sshd[5979]:') |
| mysql-bin.000001 | 4537 | Xid         |         1 |        4568 | COMMIT /* xid=25 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 4568 | Query       |         1 |        4651 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 4651 | Intvar      |         1 |        4683 | INSERT_ID=1593                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 4683 | Query       |         1 |        5023 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' Accepted password for root from 192.168.100.1 port 51286 ssh2', 10, 'h105', 6, '20160612144840', '20160612144840', 1, 'sshd[5979]:')                                                   |
| mysql-bin.000001 | 5023 | Xid         |         1 |        5054 | COMMIT /* xid=26 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 5054 | Query       |         1 |        5137 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 5137 | Intvar      |         1 |        5169 | INSERT_ID=1594                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 5169 | Query       |         1 |        5511 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' pam_unix(sshd:session): session opened for user root by (uid=0)', 10, 'h105', 6, '20160612144840', '20160612144840', 1, 'sshd[5979]:')                                                 |
| mysql-bin.000001 | 5511 | Xid         |         1 |        5542 | COMMIT /* xid=27 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 5542 | Query       |         1 |        5625 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 5625 | Intvar      |         1 |        5657 | INSERT_ID=1595                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 5657 | Query       |         1 |        5970 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612145001', '20160612145001', 1, 'CROND[6008]:')                                                                              |
| mysql-bin.000001 | 5970 | Xid         |         1 |        6001 | COMMIT /* xid=28 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 6001 | Query       |         1 |        6084 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 6084 | Intvar      |         1 |        6116 | INSERT_ID=1596                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 6116 | Query       |         1 |        6429 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612150001', '20160612150001', 1, 'CROND[6040]:')                                                                              |
| mysql-bin.000001 | 6429 | Xid         |         1 |        6460 | COMMIT /* xid=43 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 6460 | Query       |         1 |        6543 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 6543 | Intvar      |         1 |        6575 | INSERT_ID=1597                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 6575 | Query       |         1 |        6893 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (run-parts /etc/cron.hourly)', 9, 'h105', 6, '20160612150101', '20160612150101', 1, 'CROND[6046]:')                                                                         |
| mysql-bin.000001 | 6893 | Xid         |         1 |        6924 | COMMIT /* xid=44 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 6924 | Query       |         1 |        7007 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 7007 | Intvar      |         1 |        7039 | INSERT_ID=1598                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 7039 | Query       |         1 |        7357 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' starting 0anacron', 9, 'h105', 5, '20160612150101', '20160612150101', 1, 'run-parts(/etc/cron.hourly)[6046]:')                                                                         |
| mysql-bin.000001 | 7357 | Xid         |         1 |        7388 | COMMIT /* xid=45 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 7388 | Query       |         1 |        7471 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 7471 | Intvar      |         1 |        7503 | INSERT_ID=1599                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 7503 | Query       |         1 |        7821 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' finished 0anacron', 9, 'h105', 5, '20160612150101', '20160612150101', 1, 'run-parts(/etc/cron.hourly)[6055]:')                                                                         |
| mysql-bin.000001 | 7821 | Xid         |         1 |        7852 | COMMIT /* xid=46 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 7852 | Query       |         1 |        7937 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 7937 | Query       |         1 |        8062 | use `testxxx`; update test set name = 'change99' where id=99                                                                                                                                                                                                                                                                           |
| mysql-bin.000001 | 8062 | Xid         |         1 |        8093 | COMMIT /* xid=47 */                                                                                                                                                                                                                                                                                                                    |
+------------------+------+-------------+-----------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
71 rows in set (0.01 sec)

mysql> select * from test where id =  99;
+------+----------+
| id   | name     |
+------+----------+
|   99 | change99 |
+------+----------+
1 row in set (0.07 sec)

mysql> SHOW BINLOG EVENTS;
+------------------+------+-------------+-----------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name         | Pos  | Event_type  | Server_id | End_log_pos | Info                                                                                                                                                                                                                                                                                                                                   |
+------------------+------+-------------+-----------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| mysql-bin.000001 |    4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4                                                                                                                                                                                                                                                                                             |
| mysql-bin.000001 |  120 | Query       |         1 |         205 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 |  205 | Query       |         1 |         314 | use `testxxx`; delete from test where id=100                                                                                                                                                                                                                                                                                           |
| mysql-bin.000001 |  314 | Xid         |         1 |         345 | COMMIT /* xid=15 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 |  345 | Query       |         1 |         428 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 |  428 | Intvar      |         1 |         460 | INSERT_ID=1584                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 |  460 | Query       |         1 |         773 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612140001', '20160612140001', 1, 'CROND[5870]:')                                                                              |
| mysql-bin.000001 |  773 | Xid         |         1 |         804 | COMMIT /* xid=17 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 |  804 | Query       |         1 |         887 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 |  887 | Intvar      |         1 |         919 | INSERT_ID=1585                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 |  919 | Query       |         1 |        1237 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (run-parts /etc/cron.hourly)', 9, 'h105', 6, '20160612140101', '20160612140101', 1, 'CROND[5877]:')                                                                         |
| mysql-bin.000001 | 1237 | Xid         |         1 |        1268 | COMMIT /* xid=18 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 1268 | Query       |         1 |        1351 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 1351 | Intvar      |         1 |        1383 | INSERT_ID=1586                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 1383 | Query       |         1 |        1701 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' starting 0anacron', 9, 'h105', 5, '20160612140101', '20160612140101', 1, 'run-parts(/etc/cron.hourly)[5877]:')                                                                         |
| mysql-bin.000001 | 1701 | Xid         |         1 |        1732 | COMMIT /* xid=19 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 1732 | Query       |         1 |        1815 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 1815 | Intvar      |         1 |        1847 | INSERT_ID=1587                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 1847 | Query       |         1 |        2165 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' finished 0anacron', 9, 'h105', 5, '20160612140101', '20160612140101', 1, 'run-parts(/etc/cron.hourly)[5886]:')                                                                         |
| mysql-bin.000001 | 2165 | Xid         |         1 |        2196 | COMMIT /* xid=20 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 2196 | Query       |         1 |        2279 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 2279 | Intvar      |         1 |        2311 | INSERT_ID=1588                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 2311 | Query       |         1 |        2624 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612141001', '20160612141001', 1, 'CROND[5902]:')                                                                              |
| mysql-bin.000001 | 2624 | Xid         |         1 |        2655 | COMMIT /* xid=21 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 2655 | Query       |         1 |        2738 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 2738 | Intvar      |         1 |        2770 | INSERT_ID=1589                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 2770 | Query       |         1 |        3083 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612142001', '20160612142001', 1, 'CROND[5922]:')                                                                              |
| mysql-bin.000001 | 3083 | Xid         |         1 |        3114 | COMMIT /* xid=22 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 3114 | Query       |         1 |        3197 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 3197 | Intvar      |         1 |        3229 | INSERT_ID=1590                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 3229 | Query       |         1 |        3542 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612143001', '20160612143001', 1, 'CROND[5942]:')                                                                              |
| mysql-bin.000001 | 3542 | Xid         |         1 |        3573 | COMMIT /* xid=23 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 3573 | Query       |         1 |        3656 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 3656 | Intvar      |         1 |        3688 | INSERT_ID=1591                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 3688 | Query       |         1 |        4001 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612144001', '20160612144001', 1, 'CROND[5962]:')                                                                              |
| mysql-bin.000001 | 4001 | Xid         |         1 |        4032 | COMMIT /* xid=24 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 4032 | Query       |         1 |        4115 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 4115 | Intvar      |         1 |        4147 | INSERT_ID=1592                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 4147 | Query       |         1 |        4537 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' Address 192.168.100.1 maps to localhost, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!', 10, 'h105', 6, '20160612144840', '20160612144840', 1, 'sshd[5979]:') |
| mysql-bin.000001 | 4537 | Xid         |         1 |        4568 | COMMIT /* xid=25 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 4568 | Query       |         1 |        4651 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 4651 | Intvar      |         1 |        4683 | INSERT_ID=1593                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 4683 | Query       |         1 |        5023 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' Accepted password for root from 192.168.100.1 port 51286 ssh2', 10, 'h105', 6, '20160612144840', '20160612144840', 1, 'sshd[5979]:')                                                   |
| mysql-bin.000001 | 5023 | Xid         |         1 |        5054 | COMMIT /* xid=26 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 5054 | Query       |         1 |        5137 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 5137 | Intvar      |         1 |        5169 | INSERT_ID=1594                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 5169 | Query       |         1 |        5511 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' pam_unix(sshd:session): session opened for user root by (uid=0)', 10, 'h105', 6, '20160612144840', '20160612144840', 1, 'sshd[5979]:')                                                 |
| mysql-bin.000001 | 5511 | Xid         |         1 |        5542 | COMMIT /* xid=27 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 5542 | Query       |         1 |        5625 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 5625 | Intvar      |         1 |        5657 | INSERT_ID=1595                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 5657 | Query       |         1 |        5970 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612145001', '20160612145001', 1, 'CROND[6008]:')                                                                              |
| mysql-bin.000001 | 5970 | Xid         |         1 |        6001 | COMMIT /* xid=28 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 6001 | Query       |         1 |        6084 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 6084 | Intvar      |         1 |        6116 | INSERT_ID=1596                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 6116 | Query       |         1 |        6429 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (/usr/lib64/sa/sa1 1 1)', 9, 'h105', 6, '20160612150001', '20160612150001', 1, 'CROND[6040]:')                                                                              |
| mysql-bin.000001 | 6429 | Xid         |         1 |        6460 | COMMIT /* xid=43 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 6460 | Query       |         1 |        6543 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 6543 | Intvar      |         1 |        6575 | INSERT_ID=1597                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 6575 | Query       |         1 |        6893 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' (root) CMD (run-parts /etc/cron.hourly)', 9, 'h105', 6, '20160612150101', '20160612150101', 1, 'CROND[6046]:')                                                                         |
| mysql-bin.000001 | 6893 | Xid         |         1 |        6924 | COMMIT /* xid=44 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 6924 | Query       |         1 |        7007 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 7007 | Intvar      |         1 |        7039 | INSERT_ID=1598                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 7039 | Query       |         1 |        7357 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' starting 0anacron', 9, 'h105', 5, '20160612150101', '20160612150101', 1, 'run-parts(/etc/cron.hourly)[6046]:')                                                                         |
| mysql-bin.000001 | 7357 | Xid         |         1 |        7388 | COMMIT /* xid=45 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 7388 | Query       |         1 |        7471 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 7471 | Intvar      |         1 |        7503 | INSERT_ID=1599                                                                                                                                                                                                                                                                                                                         |
| mysql-bin.000001 | 7503 | Query       |         1 |        7821 | use `Syslog`; insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values (' finished 0anacron', 9, 'h105', 5, '20160612150101', '20160612150101', 1, 'run-parts(/etc/cron.hourly)[6055]:')                                                                         |
| mysql-bin.000001 | 7821 | Xid         |         1 |        7852 | COMMIT /* xid=46 */                                                                                                                                                                                                                                                                                                                    |
| mysql-bin.000001 | 7852 | Query       |         1 |        7937 | BEGIN                                                                                                                                                                                                                                                                                                                                  |
| mysql-bin.000001 | 7937 | Query       |         1 |        8062 | use `testxxx`; update test set name = 'change99' where id=99                                                                                                                                                                                                                                                                           |
| mysql-bin.000001 | 8062 | Xid         |         1 |        8093 | COMMIT /* xid=47 */                                                                                                                                                                                                                                                                                                                    |
+------------------+------+-------------+-----------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
71 rows in set (0.00 sec)

mysql>
~~~


* 除了我执行的 **update** 操作，还多出了很多插入到 **SystemEvents** 中的操作，这些应该是系统后台自动记录的
* 我的查找操作并没有记录在 binlog 中，binlog 只记录数据变更操作  
* 不加参数直接运行出来的结果，是第一个 binlog 中的所有内容


---

## 指定参数


所有参数的详细解释可以参考 **[SHOW BINLOG EVENTS][show_binlog_events]**



---

### **FROM** and **LIMIT**

~~~
mysql> show binlog events from 4 limit 1;
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                       |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4 |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
1 row in set (0.00 sec)

mysql> show binlog events from 4 limit 2;
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                       |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4 |
| mysql-bin.000001 | 120 | Query       |         1 |         205 | BEGIN                                      |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
2 rows in set (0.00 sec)

mysql> show binlog events from 4 limit 4;
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                         |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4   |
| mysql-bin.000001 | 120 | Query       |         1 |         205 | BEGIN                                        |
| mysql-bin.000001 | 205 | Query       |         1 |         314 | use `testxxx`; delete from test where id=100 |
| mysql-bin.000001 | 314 | Xid         |         1 |         345 | COMMIT /* xid=15 */                          |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
4 rows in set (0.02 sec)

mysql> 
~~~

* 可以通过 **FROM** 和 **LIMIT** 限制输出，在生产环境下，如果不限制输出，会产生一个极其消耗时间和资源的进程，它会默认返回出这个日志文件中的所有内容，这时最好使用 **mysqlbinlog** 工具来完成类似工作，并且将结果重定向到一个文件里，然后慢慢分析这个文件内容


---

### **IN**

指定要查看的日志文件

~~~
mysql> flush logs;
Query OK, 0 rows affected (0.05 sec)

mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |      9976 |
| mysql-bin.000002 |       120 |
+------------------+-----------+
2 rows in set (0.01 sec)

mysql>
mysql> show binlog events in 'mysql-bin.000002' from 4 limit 4;
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                       |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| mysql-bin.000002 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4 |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
1 row in set (0.00 sec)

mysql> 
~~~


* 我们可以使用 **IN** 来指定一个日志文件进行查看

---

###  **OFFSET**

~~~
mysql> show binlog events in 'mysql-bin.000001' from 4  limit 4 ;
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                         |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4   |
| mysql-bin.000001 | 120 | Query       |         1 |         205 | BEGIN                                        |
| mysql-bin.000001 | 205 | Query       |         1 |         314 | use `testxxx`; delete from test where id=100 |
| mysql-bin.000001 | 314 | Xid         |         1 |         345 | COMMIT /* xid=15 */                          |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
4 rows in set (0.00 sec)

mysql> show binlog events in 'mysql-bin.000001' from 4  limit 4  offset 2;
+------------------+-----+------------+-----------+-------------+----------------------------------------------+
| Log_name         | Pos | Event_type | Server_id | End_log_pos | Info                                         |
+------------------+-----+------------+-----------+-------------+----------------------------------------------+
| mysql-bin.000001 | 205 | Query      |         1 |         314 | use `testxxx`; delete from test where id=100 |
| mysql-bin.000001 | 314 | Xid        |         1 |         345 | COMMIT /* xid=15 */                          |
| mysql-bin.000001 | 345 | Query      |         1 |         428 | BEGIN                                        |
| mysql-bin.000001 | 428 | Intvar     |         1 |         460 | INSERT_ID=1584                               |
+------------------+-----+------------+-----------+-------------+----------------------------------------------+
4 rows in set (0.00 sec)

mysql> 
~~~

* **offset** 可以指定跳过几条事件

---

### 格式

下面是 **SHOW BINLOG EVENTS** 的语法与格式

~~~
SHOW BINLOG EVENTS
   [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
~~~

**SHOW BINLOG EVENTS** 的详细用法可以参考 **[SHOW BINLOG EVENTS Syntax][show_binlog_events]**


---

###  SHOW RELAYLOG EVENTS

要查看 relay 日志得使用 **SHOW RELAYLOG EVENTS** ，如果使用 **SHOW BINLOG EVENTS** 会报找不到文件的错误

~~~
mysql> show binlog  events in 'relay-bin.000197'  from 4   limit 4 ;
ERROR 1220 (HY000): Error when executing command SHOW BINLOG EVENTS: Could not find target log
mysql> show relaylog  events in 'relay-bin.000197'  from 4   limit 4 ;
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                       |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
| relay-bin.000197 |   4 | Format_desc |       112 |         120 | Server ver: 5.6.27-75.0-log, Binlog ver: 4 |
| relay-bin.000197 | 120 | Rotate      |        15 |           0 | mysql-bin.000079;pos=4                     |
| relay-bin.000197 | 167 | Format_desc |        15 |         120 | Server ver: 5.6.27-75.0-log, Binlog ver: 4 |
| relay-bin.000197 | 283 | Query       |        15 |         196 | BEGIN                                      |
+------------------+-----+-------------+-----------+-------------+--------------------------------------------+
4 rows in set (0.00 sec)

mysql>
~~~

因为 **SHOW BINLOG EVENTS** 会去 **mysql-bin.index** 里找，找不到自然报错

**SHOW RELAYLOG EVENTS** 与 **SHOW BINLOG EVENTS** 类似 ，详细用法可以参考 **[SHOW RELAYLOG EVENTS Syntax][show_relaylog_events]**


---

### 工具的缺陷

 我们看看下面的情况

~~~
mysql> show binlog events in 'mysql-bin.000001' from 4  limit 4  ;
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                                         |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc |         1 |         120 | Server ver: 5.6.27-76.0-log, Binlog ver: 4   |
| mysql-bin.000001 | 120 | Query       |         1 |         205 | BEGIN                                        |
| mysql-bin.000001 | 205 | Query       |         1 |         314 | use `testxxx`; delete from test where id=100 |
| mysql-bin.000001 | 314 | Xid         |         1 |         345 | COMMIT /* xid=15 */                          |
+------------------+-----+-------------+-----------+-------------+----------------------------------------------+
4 rows in set (0.00 sec)

mysql> show binlog events in 'mysql-bin.000001' from 120  limit 4  ;
+------------------+-----+------------+-----------+-------------+----------------------------------------------+
| Log_name         | Pos | Event_type | Server_id | End_log_pos | Info                                         |
+------------------+-----+------------+-----------+-------------+----------------------------------------------+
| mysql-bin.000001 | 120 | Query      |         1 |         205 | BEGIN                                        |
| mysql-bin.000001 | 205 | Query      |         1 |         314 | use `testxxx`; delete from test where id=100 |
| mysql-bin.000001 | 314 | Xid        |         1 |         345 | COMMIT /* xid=15 */                          |
| mysql-bin.000001 | 345 | Query      |         1 |         428 | BEGIN                                        |
+------------------+-----+------------+-----------+-------------+----------------------------------------------+
4 rows in set (0.00 sec)

mysql> show binlog events in 'mysql-bin.000001' from 100  limit 4  ;
ERROR 1220 (HY000): Error when executing command SHOW BINLOG EVENTS: Wrong offset or I/O error
mysql>
~~~

**4** 和 **120** 正好是 **Pos** 点，于是可以顺利查出结果，但是如果我指定介于它们之间的 **100** ，工具就会报 **Wrong offset or I/O error** 的错误，它并不会智能的找到之后最接近的一个位置并读出数据来，所以在查看日志内容之前一定要首先定位好，而 **POS** 一般都不是连续的，间距相差几十到几百，所以要反复尝试，变得很低效


---

## **mysqlbinlog** 

系统的 binlog 日志是由对数据进行修改的一个个事件组成

系统是以二进制的方式进行读写，mysqlbinlog 可以将它们转化为文本的形式

> **Tip:** 由于 relay log 遵循 binlog 相同的规范，所以也可以被 mysqlbinlog 进行转化


使用 mysqlbinlog 对日志进行查看

~~~
[root@h105 mysql]# mysqlbinlog mysql-bin.000001 | head -n 40
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#160612 13:56:01 server id 1  end_log_pos 120 CRC32 0x6a792fa7 	Start: binlog v 4, server v 5.6.27-76.0-log created 160612 13:56:01 at startup
ROLLBACK/*!*/;
BINLOG '
cflcVw8BAAAAdAAAAHgAAAAAAAQANS42LjI3LTc2LjAtbG9nAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAABx+VxXEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAacv
eWo=
'/*!*/;
# at 120
#160612 13:58:27 server id 1  end_log_pos 205 CRC32 0xe1f9b47e 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1465711107/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1073741824/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 205
#160612 13:58:27 server id 1  end_log_pos 314 CRC32 0xec7455fc 	Query	thread_id=3	exec_time=0	error_code=0
use `testxxx`/*!*/;
SET TIMESTAMP=1465711107/*!*/;
delete from test where id=100
/*!*/;
# at 314
#160612 13:58:27 server id 1  end_log_pos 345 CRC32 0x455b6f62 	Xid = 15
COMMIT/*!*/;
# at 345
#160612 14:00:01 server id 1  end_log_pos 428 CRC32 0x227c2add 	Query	thread_id=4	exec_time=0	error_code=0
SET TIMESTAMP=1465711201/*!*/;
/*!\C latin1 *//*!*/;
SET @@session.character_set_client=8,@@session.collation_connection=8,@@session.collation_server=8/*!*/;
BEGIN
[root@h105 mysql]#
~~~

> **Tip:** 默认情况下，会将指定日志的所有内容都转化为文本形式

从结果来看 mysqlbinlog 可以产生更为详尽的信息


之前删除一条数据的过程在这里

~~~
# at 120
#160612 13:58:27 server id 1  end_log_pos 205 CRC32 0xe1f9b47e 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1465711107/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1073741824/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 205
#160612 13:58:27 server id 1  end_log_pos 314 CRC32 0xec7455fc 	Query	thread_id=3	exec_time=0	error_code=0
use `testxxx`/*!*/;
SET TIMESTAMP=1465711107/*!*/;
delete from test where id=100
/*!*/;
# at 314
#160612 13:58:27 server id 1  end_log_pos 345 CRC32 0x455b6f62 	Xid = 15
COMMIT/*!*/;
# at 345
~~~


关于这些字段的解释可以参考：

>In the first line, the number following at indicates the file offset, or starting position, of the event in the binary log file.
>
>The second line starts with a date and time indicating when the statement started on the server where the event originated. For replication, this timestamp is propagated to slave servers. server id is the server_id value of the server where the event originated. end_log_pos indicates where the next event starts (that is, it is the end position of the current event + 1). thread_id indicates which thread executed the event. exec_time is the time spent executing the event, on a master server. On a slave, it is the difference of the end execution time on the slave minus the beginning execution time on the master. The difference serves as an indicator of how much replication lags behind the master. error_code indicates the result from executing the event. Zero means that no error occurred

---

## 输出限定

各种 **mysqlbinlog** 的参数可以改变输出的特性，详细参数可以参考  **[Utility for Processing Binary Log Files][mysqlbinlog]**


这里演示最常用的方法与参数

---

### 指定起止位置

~~~
[root@h105 mysql]# mysqlbinlog   --start-position=205    --stop-position=314     mysql-bin.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#160612 13:56:01 server id 1  end_log_pos 120 CRC32 0x6a792fa7 	Start: binlog v 4, server v 5.6.27-76.0-log created 160612 13:56:01 at startup
ROLLBACK/*!*/;
BINLOG '
cflcVw8BAAAAdAAAAHgAAAAAAAQANS42LjI3LTc2LjAtbG9nAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAABx+VxXEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAacv
eWo=
'/*!*/;
# at 205
#160612 13:58:27 server id 1  end_log_pos 314 CRC32 0xec7455fc 	Query	thread_id=3	exec_time=0	error_code=0
use `testxxx`/*!*/;
SET TIMESTAMP=1465711107/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1073741824/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
delete from test where id=100
/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@h105 mysql]# 
~~~

---

### 指定起止时间


~~~
[root@h105 mysql]# mysqlbinlog  --start-datetime="2016-06-12 13:58:27"  --stop-datetime="2016-06-12 14:00:01"  mysql-bin.000001 
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#160612 13:56:01 server id 1  end_log_pos 120 CRC32 0x6a792fa7 	Start: binlog v 4, server v 5.6.27-76.0-log created 160612 13:56:01 at startup
ROLLBACK/*!*/;
BINLOG '
cflcVw8BAAAAdAAAAHgAAAAAAAQANS42LjI3LTc2LjAtbG9nAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAABx+VxXEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAacv
eWo=
'/*!*/;
# at 120
#160612 13:58:27 server id 1  end_log_pos 205 CRC32 0xe1f9b47e 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1465711107/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1073741824/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 205
#160612 13:58:27 server id 1  end_log_pos 314 CRC32 0xec7455fc 	Query	thread_id=3	exec_time=0	error_code=0
use `testxxx`/*!*/;
SET TIMESTAMP=1465711107/*!*/;
delete from test where id=100
/*!*/;
# at 314
#160612 13:58:27 server id 1  end_log_pos 345 CRC32 0x455b6f62 	Xid = 15
COMMIT/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@h105 mysql]# 
~~~

> **Tip:** 与 **SHOW BINLOG EVENTS** 不一样的是，不论是 POS 还是时间点都可以不是一个与日志中精确匹配的值，mysqlbinlog  会自动判断，去定位到那个大于或等于指定值的第一条事件

mysqlbinlog 的输出是可以直接在 mysql 里执行的，结合管道可以方便将结果定向到数据库中，相当于重新执行一遍所有变更操作，利用这个特性可以进行定点恢复

>The output from mysqlbinlog can be re-executed (for example, by using it as input to mysql) to redo the statements in the log. This is useful for recovery operations after a server crash

关于定点恢复的操作细节，下次专门开一篇博客，官方文档可以参考 **[Point-in-Time (Incremental) Recovery Using the Binary Log][point_in_time_recovery]**


---

# 命令汇总


* **`grep bin /etc/my.cnf`**
* **`mysqlbinlog mysql-bin.000001 | head -n 40`**
* **`mysqlbinlog   --start-position=205    --stop-position=314     mysql-bin.000001`**
* **`mysqlbinlog  --start-datetime="2016-06-12 13:58:27"  --stop-datetime="2016-06-12 14:00:01"  mysql-bin.000001`**


---


[show_binlog_events]:http://dev.mysql.com/doc/refman/5.6/en/show-binlog-events.html
[mysqlbinlog]:http://dev.mysql.com/doc/refman/5.6/en/mysqlbinlog.html
[show_relaylog_events]:http://dev.mysql.com/doc/refman/5.6/en/show-relaylog-events.html
[point_in_time_recovery]:http://dev.mysql.com/doc/refman/5.6/en/point-in-time-recovery.html
