---
layout: post
title: pt-online-schema-change 使用基础 
author: wilmosfang
tags:  admintools mysql
categories:  mysql
wc: 611 3376 23432
excerpt: follow me
comments: true
---




# 前言

由于业务发展，DB表结构难免因为需求发生变动，这时会产生修改字段类型，增加字段等变更需求。

但是MySQL中对表进行ddl时，会锁表，当表记录数小于一万时，对前端影响较小，当时遇到千万级别的表 就会影响前端应用对表的写操作。

目前InnoDB引擎是通过以下步骤来进行DDL的：

* 1 按照原始表（original_table）的表结构和DDL语句，新建一个不可见的临时表（tmp_table）
* 2 在原表上加write lock，阻塞所有更新操作（insert、delete、update等）
* 3 执行insert into tmp_table select * from original_table
* 4 rename original_table和tmp_table，最后drop original_table
* 5 释放 write lock。

当锁表时间过长时业务无法接受

于是有以下两种处理方式

* 1 用standby来操作，然后和master做切换
* 2 使用 **[pt-online-schema-change][pt-online-schema-change]** 工具直接在master上做修改


前一种方式比较折腾，特别是在有多个slave的情况下，由于表结构发生了变化，master发生了变化，重新构建slave很耗时 

**[pt-online-schema-change][pt-online-schema-change]** 是一款开源的 **[percona-toolkit][percona-toolkit]** ，用来进行ALTER tables的工具(不会锁表)。

下面分享一下 **[pt-online-schema-change][pt-online-schema-change]** 的基础操作，详细可以参阅 [官方文档][pt-online-schema-change]

> **Tip:** 当前版本 **pt-online-schema-change 2.2.14**

---

# 概要

* TOC
{:toc}

---

## 构建测试表

~~~
mysql> create table forpttest( id int(6), name char(10), comment char(10), abc char(10));
Query OK, 0 rows affected (0.10 sec)

mysql> show create table forpttest\G
*************************** 1. row ***************************
       Table: forpttest
Create Table: CREATE TABLE `forpttest` (
  `id` int(6) DEFAULT NULL,
  `name` char(10) DEFAULT NULL,
  `comment` char(10) DEFAULT NULL,
  `abc` char(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

mysql> 
mysql> desc forpttest;
+---------+----------+------+-----+---------+-------+
| Field   | Type     | Null | Key | Default | Extra |
+---------+----------+------+-----+---------+-------+
| id      | int(6)   | YES  |     | NULL    |       |
| name    | char(10) | YES  |     | NULL    |       |
| comment | char(10) | YES  |     | NULL    |       |
| abc     | char(10) | YES  |     | NULL    |       |
+---------+----------+------+-----+---------+-------+
4 rows in set (0.01 sec)


mysql> insert into pt.forpttest (id,name,comment,abc) values(1,1,1,1);
Query OK, 1 row affected (0.01 sec)

mysql> 

~~~


填入一些数据

~~~
[root@h101 ~]# i=1
[root@h101 ~]#  while true; do  echo $i; let "i=$i+1"; sleep 1 ; done
1
2
3
4
5
6
7
8
^C
[root@h101 ~]#  while true; do  echo $i; mysql -u root -pmysql -e "insert into pt.forpttest values($i,$i,$i,$i)" ;let "i=$i+1"; sleep 1 ; done
9
Warning: Using a password on the command line interface can be insecure.
10
Warning: Using a password on the command line interface can be insecure.
11
Warning: Using a password on the command line interface can be insecure.
12
Warning: Using a password on the command line interface can be insecure.
...
...
47
Warning: Using a password on the command line interface can be insecure.
48
Warning: Using a password on the command line interface can be insecure.
^C
[root@h101 ~]# 
[root@h101 ~]# echo $i
49
[root@h101 ~]# 
~~~

---

## 添加新列

### 无主键报错

~~~
[root@h101 ~]# pt-online-schema-change -u root -h localhost  -pmysql  --alter='add column newid char(20)  ' --execute D=pt,t=forptte
No slaves found.  See --recursion-method if host h101.temp has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `pt`.`forpttest`...
Creating new table...
Created new table pt._forpttest_new OK.
Altering new table...
Altered `pt`.`_forpttest_new` OK.
2015-10-23T13:18:00 Dropping new table...
2015-10-23T13:18:00 Dropped new table OK.
`pt`.`forpttest` was not altered.
The new table `pt`.`_forpttest_new` does not have a PRIMARY KEY or a unique index which is required for the DELETE trigger.
[root@h101 ~]# echo $?
255
[root@h101 ~]# 
~~~

至少需要一个主键或唯一索引

> **Note:**  In almost all cases a PRIMARY KEY or UNIQUE INDEX needs to be present in the table. This is necessary because the tool creates a DELETE trigger to keep the new table updated while the process is running.

添加主键

~~~
mysql> show create table forpttest\G
*************************** 1. row ***************************
       Table: forpttest
Create Table: CREATE TABLE `forpttest` (
  `id` int(6) DEFAULT NULL,
  `name` char(10) DEFAULT NULL,
  `comment` char(10) DEFAULT NULL,
  `abc` char(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

mysql> alter table `forpttest` add primary key (`id`);
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show create table forpttest\G
*************************** 1. row ***************************
       Table: forpttest
Create Table: CREATE TABLE `forpttest` (
  `id` int(6) NOT NULL DEFAULT '0',
  `name` char(10) DEFAULT NULL,
  `comment` char(10) DEFAULT NULL,
  `abc` char(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

mysql> 
mysql> desc forpttest;
+---------+----------+------+-----+---------+-------+
| Field   | Type     | Null | Key | Default | Extra |
+---------+----------+------+-----+---------+-------+
| id      | int(6)   | NO   | PRI | 0       |       |
| name    | char(10) | YES  |     | NULL    |       |
| comment | char(10) | YES  |     | NULL    |       |
| abc     | char(10) | YES  |     | NULL    |       |
+---------+----------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql>
~~~

---

### 添加主键成功

~~~
[root@h101 ~]# pt-online-schema-change -u root -h localhost  -pmysql  --alter='add column newid char(20)  ' --execute D=pt,t=forpttest  
No slaves found.  See --recursion-method if host h101.temp has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `pt`.`forpttest`...
Creating new table...
Created new table pt._forpttest_new OK.
Altering new table...
Altered `pt`.`_forpttest_new` OK.
2015-10-23T13:53:01 Creating triggers...
2015-10-23T13:53:01 Created triggers OK.
2015-10-23T13:53:01 Copying approximately 41 rows...
2015-10-23T13:53:01 Copied rows OK.
2015-10-23T13:53:01 Swapping tables...
2015-10-23T13:53:01 Swapped original and new tables OK.
2015-10-23T13:53:01 Dropping old table...
2015-10-23T13:53:01 Dropped old table `pt`.`_forpttest_old` OK.
2015-10-23T13:53:01 Dropping triggers...
2015-10-23T13:53:01 Dropped triggers OK.
Successfully altered `pt`.`forpttest`.
[root@h101 ~]# echo $?
0
[root@h101 ~]# 
~~~

已经添加好新列

~~~
mysql> desc forpttest;
+---------+----------+------+-----+---------+-------+
| Field   | Type     | Null | Key | Default | Extra |
+---------+----------+------+-----+---------+-------+
| id      | int(6)   | NO   | PRI | 0       |       |
| name    | char(10) | YES  |     | NULL    |       |
| comment | char(10) | YES  |     | NULL    |       |
| abc     | char(10) | YES  |     | NULL    |       |
| newid   | char(20) | YES  |     | NULL    |       |
+---------+----------+------+-----+---------+-------+
5 rows in set (0.00 sec)

mysql> 
~~~


~~~
mysql> select * from forpttest;
+----+------+---------+------+-------+
| id | name | comment | abc  | newid |
+----+------+---------+------+-------+
|  1 | 1    | 1       | 1    | NULL  |
|  9 | 9    | 9       | 9    | NULL  |
| 10 | 10   | 10      | 10   | NULL  |
| 11 | 11   | 11      | 11   | NULL  |
| 12 | 12   | 12      | 12   | NULL  |
| 13 | 13   | 13      | 13   | NULL  |
| 14 | 14   | 14      | 14   | NULL  |
| 15 | 15   | 15      | 15   | NULL  |
| 16 | 16   | 16      | 16   | NULL  |
| 17 | 17   | 17      | 17   | NULL  |
| 18 | 18   | 18      | 18   | NULL  |
| 19 | 19   | 19      | 19   | NULL  |
| 20 | 20   | 20      | 20   | NULL  |
| 21 | 21   | 21      | 21   | NULL  |
| 22 | 22   | 22      | 22   | NULL  |
| 23 | 23   | 23      | 23   | NULL  |
| 24 | 24   | 24      | 24   | NULL  |
| 25 | 25   | 25      | 25   | NULL  |
| 26 | 26   | 26      | 26   | NULL  |
| 27 | 27   | 27      | 27   | NULL  |
| 28 | 28   | 28      | 28   | NULL  |
| 29 | 29   | 29      | 29   | NULL  |
| 30 | 30   | 30      | 30   | NULL  |
| 31 | 31   | 31      | 31   | NULL  |
| 32 | 32   | 32      | 32   | NULL  |
| 33 | 33   | 33      | 33   | NULL  |
| 34 | 34   | 34      | 34   | NULL  |
| 35 | 35   | 35      | 35   | NULL  |
| 36 | 36   | 36      | 36   | NULL  |
| 37 | 37   | 37      | 37   | NULL  |
| 38 | 38   | 38      | 38   | NULL  |
| 39 | 39   | 39      | 39   | NULL  |
| 40 | 40   | 40      | 40   | NULL  |
| 41 | 41   | 41      | 41   | NULL  |
| 42 | 42   | 42      | 42   | NULL  |
| 43 | 43   | 43      | 43   | NULL  |
| 44 | 44   | 44      | 44   | NULL  |
| 45 | 45   | 45      | 45   | NULL  |
| 46 | 46   | 46      | 46   | NULL  |
| 47 | 47   | 47      | 47   | NULL  |
| 48 | 48   | 48      | 48   | NULL  |
+----+------+---------+------+-------+
41 rows in set (0.00 sec)

mysql> 
~~~


可知新添加的列默认为空

---

### 非空报错


~~~
[root@h101 ~]# pt-online-schema-change -u root -h localhost  -pmysql  --alter='add column newid2 char(20) not null ' --execute D=pt,t=forpttest  
No slaves found.  See --recursion-method if host h101.temp has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `pt`.`forpttest`...
Creating new table...
Created new table pt._forpttest_new OK.
Altering new table...
Altered `pt`.`_forpttest_new` OK.
2015-10-23T14:04:18 Creating triggers...
2015-10-23T14:04:18 Created triggers OK.
2015-10-23T14:04:18 Copying approximately 41 rows...
2015-10-23T14:04:18 Dropping triggers...
2015-10-23T14:04:18 Dropped triggers OK.
2015-10-23T14:04:18 Dropping new table...
2015-10-23T14:04:18 Dropped new table OK.
`pt`.`forpttest` was not altered.
2015-10-23T14:04:18 Error copying rows from `pt`.`forpttest` to `pt`.`_forpttest_new`: 2015-10-23T14:04:18 Copying rows caused a MySQL error 1364:
    Level: Warning
     Code: 1364
  Message: Field 'newid2' doesn't have a default value
    Query: INSERT LOW_PRIORITY IGNORE INTO `pt`.`_forpttest_new` (`id`, `name`, `comment`, `abc`, `newid`) SELECT `id`, `name`, `comment`, `abc`, `newid` FROM `pt`.`forpttest` LOCK IN SHARE MODE /*pt-online-schema-change 6366 copy table*/

[root@h101 ~]# 
~~~

如果新添列有非空约束，但不给定默认值，就会报错

> **Note:**  If you add a column without a default value and make it NOT NULL, the tool will fail, as it will not try to guess a default value for you; You must specify the default.


### 添加默认值成功


~~~
[root@h101 ~]# pt-online-schema-change -u root -h localhost  -pmysql  --alter='add column newid2 char(20) not null default "fucktest" ' --execute D=pt,t=forpttest  
No slaves found.  See --recursion-method if host h101.temp has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `pt`.`forpttest`...
Creating new table...
Created new table pt._forpttest_new OK.
Altering new table...
Altered `pt`.`_forpttest_new` OK.
2015-10-23T14:08:56 Creating triggers...
2015-10-23T14:08:56 Created triggers OK.
2015-10-23T14:08:56 Copying approximately 41 rows...
2015-10-23T14:08:56 Copied rows OK.
2015-10-23T14:08:56 Swapping tables...
2015-10-23T14:08:56 Swapped original and new tables OK.
2015-10-23T14:08:56 Dropping old table...
2015-10-23T14:08:56 Dropped old table `pt`.`_forpttest_old` OK.
2015-10-23T14:08:56 Dropping triggers...
2015-10-23T14:08:56 Dropped triggers OK.
Successfully altered `pt`.`forpttest`.
[root@h101 ~]# echo $?
0
[root@h101 ~]# 

~~~

表结构已经变化 


~~~
mysql> desc forpttest;
+---------+----------+------+-----+----------+-------+
| Field   | Type     | Null | Key | Default  | Extra |
+---------+----------+------+-----+----------+-------+
| id      | int(6)   | NO   | PRI | 0        |       |
| name    | char(10) | YES  |     | NULL     |       |
| comment | char(10) | YES  |     | NULL     |       |
| abc     | char(10) | YES  |     | NULL     |       |
| newid   | char(20) | YES  |     | NULL     |       |
| newid2  | char(20) | NO   |     | fucktest |       |
+---------+----------+------+-----+----------+-------+
6 rows in set (0.01 sec)

mysql> select * from forpttest;
+----+------+---------+------+-------+----------+
| id | name | comment | abc  | newid | newid2   |
+----+------+---------+------+-------+----------+
|  1 | 1    | 1       | 1    | NULL  | fucktest |
|  9 | 9    | 9       | 9    | NULL  | fucktest |
| 10 | 10   | 10      | 10   | NULL  | fucktest |
| 11 | 11   | 11      | 11   | NULL  | fucktest |
...
...
| 47 | 47   | 47      | 47   | NULL  | fucktest |
| 48 | 48   | 48      | 48   | NULL  | fucktest |
+----+------+---------+------+-------+----------+
41 rows in set (0.00 sec)

mysql> 
~~~

> **Note:** 还有其它一些 **[注意事项][limitations]**


~~~
--alter
type: string

The schema modification, without the ALTER TABLE keywords. You can perform multiple modifications to the table by specifying them with commas. Please refer to the MySQL manual for the syntax of ALTER TABLE.

The following limitations apply which, if attempted, will cause the tool to fail in unpredictable ways:

* In almost all cases a PRIMARY KEY or UNIQUE INDEX needs to be present in the table. This is necessary because the tool creates a DELETE trigger to keep the new table updated while the process is running.

A notable exception is when a PRIMARY KEY or UNIQUE INDEX is being created from existing columns as part of the ALTER clause; in that case it will use these column(s) for the DELETE trigger.

* The RENAME clause cannot be used to rename the table.

* Columns cannot be renamed by dropping and re-adding with the new name. The tool will not copy the original column’s data to the new column.

* If you add a column without a default value and make it NOT NULL, the tool will fail, as it will not try to guess a default value for you; You must specify the default.

* DROP FOREIGN KEY constraint_name requires specifying _constraint_name rather than the real constraint_name. Due to a limitation in MySQL, pt-online-schema-change adds a leading underscore to foreign key constraint names when creating the new table. For example, to drop this constraint:

CONSTRAINT `fk_foo` FOREIGN KEY (`foo_id`) REFERENCES `bar` (`foo_id`)
You must specify --alter "DROP FOREIGN KEY _fk_foo".

* The tool does not use LOCK IN SHARE MODE with MySQL 5.0 because it can cause a slave error which breaks replication:

Query caused different errors on master and slave. Error on master:
'Deadlock found when trying to get lock; try restarting transaction' (1213),
Error on slave: 'no error' (0). Default database: 'pt_osc'.
Query: 'INSERT INTO pt_osc.t (id, c) VALUES ('730', 'new row')'

The error happens when converting a MyISAM table to InnoDB because MyISAM is non-transactional but InnoDB is transactional. MySQL 5.1 and newer handle this case correctly, but testing reproduces the error 5% of the time with MySQL 5.0.

This is a MySQL bug, similar to http://bugs.mysql.com/bug.php?id=45694, but there is no fix or workaround in MySQL 5.0. Without LOCK IN SHARE MODE, tests pass 100% of the time, so the risk of data loss or breaking replication should be negligible.

Be sure to verify the new table if using MySQL 5.0 and converting from MyISAM to InnoDB!
~~~



---

### 业务在线测试


打开一个session，不断往表里插入新数据


~~~
[root@h101 ~]# echo $i
49
[root@h101 ~]#  while true; do  echo $i; mysql -u root -pmysql -e "insert into pt.forpttest(id,name,comment,abc)  values($i,$i,$i,$i)" ;let "i=$i+1"; sleep 1 ; done
49
Warning: Using a password on the command line interface can be insecure.
50
Warning: Using a password on the command line interface can be insecure.
51
Warning: Using a password on the command line interface can be insecure.
52
Warning: Using a password on the command line interface can be insecure.
53
Warning: Using a password on the command line interface can be insecure.
54
Warning: Using a password on the command line interface can be insecure.
55
Warning: Using a password on the command line interface can be insecure.
56
Warning: Using a password on the command line interface can be insecure.
...
...
77
Warning: Using a password on the command line interface can be insecure.
78
Warning: Using a password on the command line interface can be insecure.
79
Warning: Using a password on the command line interface can be insecure.
80
Warning: Using a password on the command line interface can be insecure.
81
~~~


同时在另一个session里修改表结构


~~~
[root@h101 ~]# pt-online-schema-change -u root -h localhost  -pmysql  --alter='add column newcolumn char(20) not null default "duang" ' --execute D=pt,t=forpttest  
No slaves found.  See --recursion-method if host h101.temp has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `pt`.`forpttest`...
Creating new table...
Created new table pt._forpttest_new OK.
Altering new table...
Altered `pt`.`_forpttest_new` OK.
2015-10-23T14:27:33 Creating triggers...
2015-10-23T14:27:33 Created triggers OK.
2015-10-23T14:27:33 Copying approximately 47 rows...
2015-10-23T14:27:33 Copied rows OK.
2015-10-23T14:27:33 Swapping tables...
2015-10-23T14:27:33 Swapped original and new tables OK.
2015-10-23T14:27:33 Dropping old table...
2015-10-23T14:27:33 Dropped old table `pt`.`_forpttest_old` OK.
2015-10-23T14:27:33 Dropping triggers...
2015-10-23T14:27:33 Dropped triggers OK.
Successfully altered `pt`.`forpttest`.
[root@h101 ~]#
~~~

表结构按预期发生了变化，正在执行的操作也没有中断

~~~
mysql> desc forpttest;
+-----------+----------+------+-----+----------+-------+
| Field     | Type     | Null | Key | Default  | Extra |
+-----------+----------+------+-----+----------+-------+
| id        | int(6)   | NO   | PRI | 0        |       |
| name      | char(10) | YES  |     | NULL     |       |
| comment   | char(10) | YES  |     | NULL     |       |
| abc       | char(10) | YES  |     | NULL     |       |
| newid     | char(20) | YES  |     | NULL     |       |
| newid2    | char(20) | NO   |     | fucktest |       |
| newcolumn | char(20) | NO   |     | duang    |       |
+-----------+----------+------+-----+----------+-------+
7 rows in set (0.00 sec)

mysql> 
~~~

可知DML不会被锁






---

# 高级:外键


Foreign keys complicate the tool’s operation and introduce additional risk. The technique of atomically renaming the original and new tables does not work when foreign keys refer to the table. The tool must update foreign keys to refer to the new table after the schema change is complete. The tool supports two methods for accomplishing this. You can read more about this in the documentation for --alter-foreign-keys-method.

Foreign keys also cause some side effects. The final table will have the same foreign keys and indexes as the original table (unless you specify differently in your ALTER statement), but the names of the objects may be changed slightly to avoid object name collisions in MySQL and InnoDB.

For safety, the tool does not modify the table unless you specify the --execute option, which is not enabled by default. The tool supports a variety of other measures to prevent unwanted load or other problems, including automatically detecting replicas, connecting to them, and using the following safety checks:

* In most cases the tool will refuse to operate unless a PRIMARY KEY or UNIQUE INDEX is present in the table. See --alter for details.
* The tool refuses to operate if it detects replication filters. See --[no]check-replication-filters for details.
* The tool pauses the data copy operation if it observes any replicas that are delayed in replication. See --max-lag for details.
* The tool pauses or aborts its operation if it detects too much load on the server. See --max-load and --critical-load for details.
* The tool sets innodb_lock_wait_timeout=1 and (for MySQL 5.5 and newer) lock_wait_timeout=60 so that it is more likely to be the victim of any lock contention, and less likely to disrupt other transactions. These values can be changed by specifying --set-vars.
* The tool refuses to alter the table if foreign key constraints reference it, unless you specify --alter-foreign-keys-method.
* The tool cannot alter MyISAM tables on “Percona XtraDB Cluster” nodes.

---


# 附:pt-online-schema-change 工作原理



**pt-online-schema-change** emulates the way that MySQL alters tables internally, but it works on a copy of the table you wish to alter. This means that the original table is not locked, and clients may continue to read and change data in it.

**pt-online-schema-change** works by creating an empty copy of the table to alter, modifying it as desired, and then copying rows from the original table into the new table. When the copy is complete, it moves away the original table and replaces it with the new one. By default, it also drops the original table.

The data copy process is performed in small chunks of data, which are varied to attempt to make them execute in a specific amount of time (see --chunk-time). This process is very similar to how other tools, such as pt-table-checksum, work. Any modifications to data in the original tables during the copy will be reflected in the new table, because the tool creates triggers on the original table to update the corresponding rows in the new table. The use of triggers means that the tool will not work if any triggers are already defined on the table.

When the tool finishes copying data into the new table, it uses an atomic RENAME TABLE operation to simultaneously rename the original and new tables. After this is complete, the tool drops the original table.



---

[pt-online-schema-change]:https://www.percona.com/doc/percona-toolkit/2.2/pt-online-schema-change.html
[percona-toolkit]:https://www.percona.com/doc/percona-toolkit/2.2/index.html
[limitations]:https://www.percona.com/doc/percona-toolkit/2.2/pt-online-schema-change.html#cmdoption-pt-online-schema-change--alter
