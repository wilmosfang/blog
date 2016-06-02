---
layout: post
title:  Mycat 管理命令
author: wilmosfang
categories:   mysql mycat 
tags:   mysql mycat 
wc: 887  4580 52439 
excerpt: mycat的管理，与命令总览 
comments: true
---



# 前言

**[Mycat][mycat]** 是一款开源的数据库分库分表中间件

可以通过 Mysql 命令行，登录 **9066** 端口执行 SQL 的方式来进行管理

事实上目前的管理命令集更多是用来进行查看和分析，里面包含很多有价值的统计数据，但是对 **Mycat** 实例的操作命令还相对较少

这里分享一下 **[Mycat][mycat]** 相关管理操作，详细内容可以参考 **[官方文档][mycat_doc]** 


> **Tip:** 当前的最新版本为 **Mycat server 1.5 GA**

---


# 概要

* TOC
{:toc}

---

## 管理端口

正常启动 mycat 后，会在本地打开 **8066** 和 **9066** 两个端口，其中：

* **8066** 用作数据交互
* **9066** 用作mycat管理

~~~
[root@h102 conf]# grep 66 server.xml 
			<property name="serverPort">8066</property> <property name="managerPort">9066</property> 
[root@h102 conf]# netstat  -ant | grep 66
tcp        0      0 :::8066                     :::*                        LISTEN      
tcp        0      0 :::9066                     :::*                        LISTEN      
[root@h102 conf]# 
~~~

这个端口可以在启动前自定义

---

## 登录管理口


管理台的登录方式与普通mysql登录无异

~~~
[root@h102 conf]# mysql -u cc -p -P 9066 -h 192.168.100.102
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.8-mycat-1.5-GA-20160217103036 MyCat Server (monitor)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
~~~

Arg | Comment
-------- | ---
`-u`| **server.xml** 中配置的逻辑用户名
`-p`| **server.xml** 中与逻辑用户名应对的密码
`-P`| 指定连接端口
`-h`| 指定mycat实例的主机名

可以看到和mysql的登录没有不同


---


## 命令总览


使用 **`show @@help`** 能够查看所有可用命令

~~~
mysql> show @@help;
+------------------------------------------+--------------------------------------------+
| STATEMENT                                | DESCRIPTION                                |
+------------------------------------------+--------------------------------------------+
| show @@time.current                      | Report current timestamp                   |
| show @@time.startup                      | Report startup timestamp                   |
| show @@version                           | Report Mycat Server version                |
| show @@server                            | Report server status                       |
| show @@threadpool                        | Report threadPool status                   |
| show @@database                          | Report databases                           |
| show @@datanode                          | Report dataNodes                           |
| show @@datanode where schema = ?         | Report dataNodes                           |
| show @@datasource                        | Report dataSources                         |
| show @@datasource where dataNode = ?     | Report dataSources                         |
| show @@datasource.synstatus              | Report datasource data synchronous         |
| show @@datasource.syndetail where name=? | Report datasource data synchronous detail  |
| show @@datasource.cluster                | Report datasource galary cluster variables |
| show @@processor                         | Report processor status                    |
| show @@command                           | Report commands status                     |
| show @@connection                        | Report connection status                   |
| show @@cache                             | Report system cache usage                  |
| show @@backend                           | Report backend connection status           |
| show @@session                           | Report front session details               |
| show @@connection.sql                    | Report connection sql                      |
| show @@sql.execute                       | Report execute status                      |
| show @@sql.detail where id = ?           | Report execute detail status               |
| show @@sql                               | Report SQL list                            |
| show @@sql.high                          | Report Hight Frequency SQL                 |
| show @@sql.slow                          | Report slow SQL                            |
| show @@sql.sum                           | Report  User RW Stat                       |
| show @@sql.sum.user                      | Report  User RW Stat                       |
| show @@sql.sum.table                     | Report  Table RW Stat                      |
| show @@parser                            | Report parser status                       |
| show @@router                            | Report router status                       |
| show @@heartbeat                         | Report heartbeat status                    |
| show @@heartbeat.detail where name=?     | Report heartbeat current detail            |
| show @@slow where schema = ?             | Report schema slow sql                     |
| show @@slow where datanode = ?           | Report datanode slow sql                   |
| show @@sysparam                          | Report system param                        |
| show @@syslog limit=?                    | Report system mycat.log                    |
| show @@white                             | show mycat white host                      |
| show @@white.set=?,?                     | set mycat white host,[ip,user]             |
| switch @@datasource name:index           | Switch dataSource                          |
| kill @@connection id1,id2,...            | Kill the specified connections             |
| stop @@heartbeat name:time               | Pause dataNode heartbeat                   |
| reload @@config                          | Reload basic config from file              |
| reload @@config_all                      | Reload all config from file                |
| reload @@route                           | Reload route config from file              |
| reload @@user                            | Reload user config from file               |
| reload @@sqlslow=                        | Set Slow SQL Time(ms)                      |
| reload @@user_stat                       | Reset show @@sql  @@sql.sum @@sql.slow     |
| rollback @@config                        | Rollback all config from memory            |
| rollback @@route                         | Rollback route config from memory          |
| rollback @@user                          | Rollback user config from memory           |
| reload @@sqlstat=open                    | Open real-time sql stat analyzer           |
| reload @@sqlstat=close                   | Close real-time sql stat analyzer          |
| offline                                  | Change MyCat status to OFF                 |
| online                                   | Change MyCat status to ON                  |
| clear @@slow where schema = ?            | Clear slow sql by schema                   |
| clear @@slow where datanode = ?          | Clear slow sql by datanode                 |
+------------------------------------------+--------------------------------------------+
56 rows in set (0.01 sec)

mysql> 
~~~


---

### 查看当前时间

~~~
mysql> show @@time.current;
+---------------+
| TIMESTAMP     |
+---------------+
| 1457440508666 |
+---------------+
1 row in set (0.00 sec)

mysql> 
~~~

### 查看启动时间

~~~
mysql> show @@time.startup;
+---------------+
| TIMESTAMP     |
+---------------+
| 1457439259286 |
+---------------+
1 row in set (0.00 sec)

mysql> 
~~~

> **Note:**  这个时间是以ms为单位的Unix时间 


---

### 查看当前版本


~~~
mysql> show @@version;
+-----------------------------------+
| VERSION                           |
+-----------------------------------+
| 5.5.8-mycat-1.5-GA-20160217103036 |
+-----------------------------------+
1 row in set (0.00 sec)

mysql> 
~~~


### 查看系统状态


~~~
mysql>  show @@server;
+---------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
| UPTIME        | USED_MEMORY | TOTAL_MEMORY | MAX_MEMORY | RELOAD_TIME   | ROLLBACK_TIME | CHARSET | STATUS | AVG_BUFPOOL_ITEM_SIZE |
+---------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
| 25m 18s 521ms |    11423568 |    129499136 |  477102080 | 1457439259286 |            -1 | utf8    | ON     |                   873 |
+---------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
1 row in set (0.00 sec)

mysql> 
~~~


### 查看线程池


~~~
mysql> show @@threadpool ;
+------------------+-----------+--------------+-----------------+----------------+------------+
| NAME             | POOL_SIZE | ACTIVE_COUNT | TASK_QUEUE_SIZE | COMPLETED_TASK | TOTAL_TASK |
+------------------+-----------+--------------+-----------------+----------------+------------+
| Timer            |         2 |            0 |               0 |           3304 |       3304 |
| BusinessExecutor |         4 |            0 |               0 |           2024 |       2024 |
+------------------+-----------+--------------+-----------------+----------------+------------+
2 rows in set (0.00 sec)

mysql> 
~~~


### 查看逻辑数据库


~~~
mysql> show @@database;
+----------+
| DATABASE |
+----------+
| TESTDB   |
| cctest   |
+----------+
2 rows in set (0.00 sec)

mysql>
~~~


### 查看数据节点


#### 查看所有数据节点

~~~
mysql> show @@datanode;
+------+----------------+-------+-------+--------+------+------+---------+------------+----------+---------+---------------+
| NAME | DATHOST        | INDEX | TYPE  | ACTIVE | IDLE | SIZE | EXECUTE | TOTAL_TIME | MAX_TIME | MAX_SQL | RECOVERY_TIME |
+------+----------------+-------+-------+--------+------+------+---------+------------+----------+---------+---------------+
| dn1  | localhost1/db1 |     0 | mysql |      0 |    0 | 1000 |       0 |          0 |        0 |       0 |            -1 |
| dn2  | localhost1/db2 |     0 | mysql |      0 |    0 | 1000 |       0 |          0 |        0 |       0 |            -1 |
| dn3  | localhost1/db3 |     0 | mysql |      0 |    0 | 1000 |       0 |          0 |        0 |       0 |            -1 |
| sd1  | h101/my1       |     0 | mysql |      0 |    4 |  100 |       4 |          0 |        0 |       0 |            -1 |
| sd2  | h101/my2       |     0 | mysql |      0 |    0 |  100 |       2 |          0 |        0 |       0 |            -1 |
| sd3  | h101/my3       |     0 | mysql |      0 |    6 |  100 |     175 |          0 |        0 |       0 |            -1 |
| sd4  | h202/my4       |     0 | mysql |      0 |    0 |  100 |       0 |          0 |        0 |       0 |            -1 |
| sd5  | h101/my5       |     0 | mysql |      0 |    0 |  100 |       2 |          0 |        0 |       0 |            -1 |
+------+----------------+-------+-------+--------+------+------+---------+------------+----------+---------+---------------+
8 rows in set (0.01 sec)

mysql> 
~~~

#### 查看某个schema的数据节点


~~~
mysql> show @@datanode where schema = cctest;
+------+----------+-------+-------+--------+------+------+---------+------------+----------+---------+---------------+
| NAME | DATHOST  | INDEX | TYPE  | ACTIVE | IDLE | SIZE | EXECUTE | TOTAL_TIME | MAX_TIME | MAX_SQL | RECOVERY_TIME |
+------+----------+-------+-------+--------+------+------+---------+------------+----------+---------+---------------+
| sd1  | h101/my1 |     0 | mysql |      0 |    4 |  100 |       4 |          0 |        0 |       0 |            -1 |
| sd2  | h101/my2 |     0 | mysql |      0 |    0 |  100 |       2 |          0 |        0 |       0 |            -1 |
| sd3  | h101/my3 |     0 | mysql |      0 |    6 |  100 |     180 |          0 |        0 |       0 |            -1 |
| sd5  | h101/my5 |     0 | mysql |      0 |    0 |  100 |       2 |          0 |        0 |       0 |            -1 |
+------+----------+-------+-------+--------+------+------+---------+------------+----------+---------+---------------+
4 rows in set (0.01 sec)

mysql> 
~~~


### 查看数据源

#### 查看所有数据源

~~~
mysql> show @@datasource;
+----------+--------+-------+-----------------+------+------+--------+------+------+---------+
| DATANODE | NAME   | TYPE  | HOST            | PORT | W/R  | ACTIVE | IDLE | SIZE | EXECUTE |
+----------+--------+-------+-----------------+------+------+--------+------+------+---------+
| sd3      | h101M1 | mysql | 192.168.100.101 | 3306 | W    |      0 |   10 |  100 |     194 |
| sd4      | h202M1 | mysql | 192.168.100.202 | 3306 | W    |      0 |    0 |  100 |       0 |
| sd1      | h101M1 | mysql | 192.168.100.101 | 3306 | W    |      0 |   10 |  100 |     194 |
| sd2      | h101M1 | mysql | 192.168.100.101 | 3306 | W    |      0 |   10 |  100 |     194 |
| sd5      | h101M1 | mysql | 192.168.100.101 | 3306 | W    |      0 |   10 |  100 |     194 |
| dn2      | hostM1 | mysql | localhost       | 3306 | W    |      0 |    0 | 1000 |       0 |
| dn2      | hostS1 | mysql | localhost       | 3316 | W    |      0 |    0 | 1000 |       0 |
| dn3      | hostM1 | mysql | localhost       | 3306 | W    |      0 |    0 | 1000 |       0 |
| dn3      | hostS1 | mysql | localhost       | 3316 | W    |      0 |    0 | 1000 |       0 |
| dn1      | hostM1 | mysql | localhost       | 3306 | W    |      0 |    0 | 1000 |       0 |
| dn1      | hostS1 | mysql | localhost       | 3316 | W    |      0 |    0 | 1000 |       0 |
+----------+--------+-------+-----------------+------+------+--------+------+------+---------+
11 rows in set (0.00 sec)

mysql> 
~~~


#### 查看某一个数据节点的数据源

~~~
mysql> show @@datasource where dataNode = sd2;
+----------+--------+-------+-----------------+------+------+--------+------+------+---------+
| DATANODE | NAME   | TYPE  | HOST            | PORT | W/R  | ACTIVE | IDLE | SIZE | EXECUTE |
+----------+--------+-------+-----------------+------+------+--------+------+------+---------+
| sd2      | h101M1 | mysql | 192.168.100.101 | 3306 | W    |      0 |   10 |  100 |     203 |
+----------+--------+-------+-----------------+------+------+--------+------+------+---------+
1 row in set (0.00 sec)

mysql> 
~~~


---

### 查看处理器状态

~~~
mysql> show @@processor;
+------------+--------+---------+-------------+---------+---------+-------------+--------------+------------+----------+----------+----------+
| NAME       | NET_IN | NET_OUT | REACT_COUNT | R_QUEUE | W_QUEUE | FREE_BUFFER | TOTAL_BUFFER | BU_PERCENT | BU_WARNS | FC_COUNT | BC_COUNT |
+------------+--------+---------+-------------+---------+---------+-------------+--------------+------------+----------+----------+----------+
| Processor0 |  14212 |    3804 |           0 |       0 |       0 |         186 |          200 |          7 |      277 |        0 |        7 |
| Processor1 |   8229 |   25989 |           0 |       0 |       0 |         186 |          200 |          7 |      277 |        2 |        3 |
+------------+--------+---------+-------------+---------+---------+-------------+--------------+------------+----------+----------+----------+
2 rows in set (0.00 sec)

mysql> 
~~~


### 查看命令状态(统计)

~~~
mysql> show @@command;
+------------+---------+-------+--------------+--------------+------------+------+------+------+-------+
| PROCESSOR  | INIT_DB | QUERY | STMT_PREPARE | STMT_EXECUTE | STMT_CLOSE | PING | KILL | QUIT | OTHER |
+------------+---------+-------+--------------+--------------+------------+------+------+------+-------+
| Processor0 |       0 |     0 |            0 |            0 |          0 |    0 |    0 |    0 |     0 |
| Processor1 |       0 |    32 |            0 |            0 |          0 |    0 |    0 |    0 |     0 |
+------------+---------+-------+--------------+--------------+------------+------+------+------+-------+
2 rows in set (0.00 sec)

mysql> 
~~~


### 查看连接状态

~~~
mysql> show @@connection;
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
| PROCESSOR  | ID   | HOST            | PORT | LOCAL_PORT | USER | SCHEMA | CHARSET | NET_IN | NET_OUT | ALIVE_TIME(S) | RECV_BUFFER | SEND_QUEUE | txlevel | autocommit |
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
| Processor1 |    2 | 192.168.100.101 | 9066 |      41013 | cc   | NULL   | utf8:33 |    113 |    3108 |           190 |       40960 |          0 |         |            |
| Processor1 |    1 | 192.168.100.102 | 9066 |      43190 | cc   | NULL   | utf8:45 |    846 |   21755 |          1981 |       40960 |          0 |         |            |
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
2 rows in set (0.01 sec)

mysql> 
~~~


### 查看缓存状态


~~~
mysql> show @@cache;
+-------------------------------------+-------+------+--------+------+------+-------------+----------+
| CACHE                               | MAX   | CUR  | ACCESS | HIT  | PUT  | LAST_ACCESS | LAST_PUT |
+-------------------------------------+-------+------+--------+------+------+-------------+----------+
| SQLRouteCache                       | 10000 |    0 |      0 |    0 |    0 |           0 |        0 |
| TableID2DataNodeCache.TESTDB_ORDERS | 50000 |    0 |      0 |    0 |    0 |           0 |        0 |
| ER_SQL2PARENTID                     |  1000 |    0 |      0 |    0 |    0 |           0 |        0 |
+-------------------------------------+-------+------+--------+------+------+-------------+----------+
3 rows in set (0.01 sec)

mysql>
~~~

### 查看后端连接状态


~~~
mysql> show @@backend;
+------------+------+---------+-----------------+------+--------+--------+---------+------+--------+----------+------------+--------+----------+---------+------------+
| processor  | id   | mysqlId | host            | port | l_port | net_in | net_out | life | closed | borrowed | SEND_QUEUE | schema | charset  | txlevel | autocommit |
+------------+------+---------+-----------------+------+--------+--------+---------+------+--------+----------+------------+--------+----------+---------+------------+
| Processor0 |   12 |      12 | 192.168.100.101 | 3306 |  34828 |   3127 |     804 | 2338 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor0 |   13 |      14 | 192.168.100.101 | 3306 |  35206 |   2609 |     678 | 2038 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor0 |    8 |       2 | 192.168.100.101 | 3306 |  34416 |   3867 |     984 | 2657 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor0 |    2 |       8 | 192.168.100.101 | 3306 |  34422 |    463 |     156 | 2657 | false  | false    |          0 | my1    | latin1:8 | 0       | true       |
| Processor0 |   10 |      11 | 192.168.100.101 | 3306 |  34425 |    463 |     156 | 2657 | false  | false    |          0 | my1    | latin1:8 | 0       | true       |
| Processor0 |   15 |      16 | 192.168.100.101 | 3306 |  37083 |    611 |     192 |  538 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor0 |    4 |       7 | 192.168.100.101 | 3306 |  34421 |   3941 |    1002 | 2657 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor1 |   14 |      15 | 192.168.100.101 | 3306 |  35582 |   2091 |     552 | 1738 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor1 |    3 |       3 | 192.168.100.101 | 3306 |  34417 |   3941 |    1002 | 2657 | false  | false    |          0 | my3    | latin1:8 | 0       | true       |
| Processor1 |   11 |      13 | 192.168.100.101 | 3306 |  34829 |    315 |     120 | 2338 | false  | false    |          0 | my1    | latin1:8 | 0       | true       |
+------------+------+---------+-----------------+------+--------+--------+---------+------+--------+----------+------------+--------+----------+---------+------------+
10 rows in set (0.00 sec)

mysql> 
~~~

### 查看会话状态

~~~
mysql> show @@session;
Empty set (0.00 sec)

mysql> 
~~~


### 查看连接SQL

~~~
mysql>  show @@connection.sql;
+------+-----------------+------+--------+---------------+--------------+-----------------------+
| ID   | HOST            | USER | SCHEMA | START_TIME    | EXECUTE_TIME | SQL                   |
+------+-----------------+------+--------+---------------+--------------+-----------------------+
|    2 | 192.168.100.101 | cc   | NULL   | 1457441641104 |       452380 | show @@help           |
|    1 | 192.168.100.102 | cc   | NULL   | 1457442093484 |            0 | show @@connection.sql |
|    3 | 192.168.100.102 | cc   | cctest | 1457442089977 |           42 | desc catworld         |
+------+-----------------+------+--------+---------------+--------------+-----------------------+
3 rows in set (0.00 sec)

mysql> 
~~~


### 查看SQL执行状态

~~~
mysql> show @@sql.execute;
+--------+---------+-------+----------+----------+
| SQL_ID | EXECUTE | TIME  | MAX_TIME | MIN_TIME |
+--------+---------+-------+----------+----------+
|   1000 |     100 | 898.9 |      8.8 |        1 |
|   2000 |     100 | 898.9 |      8.8 |        1 |
|   3000 |     100 | 898.9 |      8.8 |        1 |
+--------+---------+-------+----------+----------+
3 rows in set (0.01 sec)

mysql> 
~~~

### 查看SQL详细

~~~
mysql> show @@sql.detail where id = 3;
+-------------+---------+------+------------------------+-----------+
| DATA_SOURCE | EXECUTE | TIME | LAST_EXECUTE_TIMESTAMP | LAST_TIME |
+-------------+---------+------+------------------------+-----------+
| mysql_1     |     123 |  2.3 |          1279188420682 |      3.42 |
| mysql_1     |     123 |  2.3 |          1279188420682 |      3.42 |
| mysql_1     |     123 |  2.3 |          1279188420682 |      3.42 |
+-------------+---------+------+------------------------+-----------+
3 rows in set (0.00 sec)

mysql>
~~~


### 查看SQL


~~~
mysql> show @@sql;
+------+------+---------------+--------------+-------------------------------+
| ID   | USER | START_TIME    | EXECUTE_TIME | SQL                           |
+------+------+---------------+--------------+-------------------------------+
|   49 | cc   | 1457442294270 |           27 | select count(*) from catworld |
+------+------+---------------+--------------+-------------------------------+
1 row in set (0.00 sec)

mysql> 
~~~


### 查看高频SQL


~~~
mysql> show @@sql.high ;
+------+------+-----------+----------+----------+----------+--------------+---------------+-------------------------------+
| ID   | USER | FREQUENCY | AVG_TIME | MAX_TIME | MIN_TIME | EXECUTE_TIME | LAST_TIME     | SQL                           |
+------+------+-----------+----------+----------+----------+--------------+---------------+-------------------------------+
|    0 | cc   |         1 |       13 |       27 |       27 |           27 | 1457442294297 | select count(*) from catworld |
+------+------+-----------+----------+----------+----------+--------------+---------------+-------------------------------+
1 row in set (0.00 sec)

mysql> 
~~~


### 查看慢查询

~~~
mysql> show @@sql.slow;
Empty set (0.00 sec)

mysql>
~~~

### 查看SQL统计

~~~
mysql> show @@sql.sum;
+------+------+------+------+------+------+--------------+--------------+---------------+
| ID   | USER | R    | W    | R%   | MAX  | TIME_COUNT   | TTL_COUNT    | LAST_TIME     |
+------+------+------+------+------+------+--------------+--------------+---------------+
|    1 | cc   |    8 |    0 | 1.00 | 1    | [0, 0, 0, 8] | [5, 3, 0, 0] | 1457442532080 |
+------+------+------+------+------+------+--------------+--------------+---------------+
1 row in set (0.00 sec)

mysql> 
mysql> show @@sql.sum.user;
+------+------+------+------+------+------+--------------+--------------+---------------+
| ID   | USER | R    | W    | R%   | MAX  | TIME_COUNT   | TTL_COUNT    | LAST_TIME     |
+------+------+------+------+------+------+--------------+--------------+---------------+
|    1 | cc   |    8 |    0 | 1.00 | 1    | [0, 0, 0, 8] | [5, 3, 0, 0] | 1457442532080 |
+------+------+------+------+------+------+--------------+--------------+---------------+
1 row in set (0.00 sec)

mysql> 
~~~


### 查看表统计


~~~
mysql> show @@sql.sum.table;
+------+----------+------+------+------+-----------+-----------+---------------+
| ID   | TABLE    | R    | W    | R%   | RELATABLE | RELACOUNT | LAST_TIME     |
+------+----------+------+------+------+-----------+-----------+---------------+
|    0 | catworld |    5 |    0 | 1.00 | NULL      | NULL      | 1457442518224 |
|    1 | abc      |    3 |    0 | 1.00 | NULL      | NULL      | 1457442532080 |
+------+----------+------+------+------+-----------+-----------+---------------+
2 rows in set (0.01 sec)

mysql> 
~~~

### 查看分析器状态

~~~
mysql> show @@parser;
+----------------+-------------+------------+----------------+------------------+--------------+------------+
| PROCESSOR_NAME | PARSE_COUNT | TIME_COUNT | MAX_PARSE_TIME | MAX_PARSE_SQL_ID | CACHED_COUNT | CACHE_SIZE |
+----------------+-------------+------------+----------------+------------------+--------------+------------+
| NULL           |        NULL |       NULL |           NULL |             NULL |         NULL |       NULL |
+----------------+-------------+------------+----------------+------------------+--------------+------------+
1 row in set (0.01 sec)

mysql> 
~~~


### 查看路由状态


~~~
mysql> show @@router;
+----------------+-------------+------------+----------------+------------------+
| PROCESSOR_NAME | ROUTE_COUNT | TIME_COUNT | MAX_ROUTE_TIME | MAX_ROUTE_SQL_ID |
+----------------+-------------+------------+----------------+------------------+
| Processor0     |        NULL |       NULL |           NULL |             NULL |
| Processor1     |        NULL |       NULL |           NULL |             NULL |
+----------------+-------------+------------+----------------+------------------+
2 rows in set (0.00 sec)

mysql> 
~~~

### 查看心跳状态


~~~
mysql> show @@heartbeat;
+--------+-------+-----------------+------+---------+-------+----------+---------+--------------+---------------------+-------+
| NAME   | TYPE  | HOST            | PORT | RS_CODE | RETRY | STATUS   | TIMEOUT | EXECUTE_TIME | LAST_ACTIVE_TIME    | STOP  |
+--------+-------+-----------------+------+---------+-------+----------+---------+--------------+---------------------+-------+
| h202M1 | mysql | 192.168.100.202 | 3306 |      -1 |     0 | checking |       0 | 750,600,610  | 1970-01-01 08:00:00 | false |
| h101M1 | mysql | 192.168.100.101 | 3306 |       1 |     0 | idle     |       0 | 2,2,2        | 2016-03-08 21:11:58 | false |
| hostM1 | mysql | localhost       | 3306 |      -1 |     0 | idle     |       0 | 0,0,0        | 2016-03-08 21:11:58 | false |
| hostS1 | mysql | localhost       | 3316 |      -1 |     0 | idle     |       0 | 0,0,0        | 2016-03-08 21:11:58 | false |
+--------+-------+-----------------+------+---------+-------+----------+---------+--------------+---------------------+-------+
4 rows in set (0.01 sec)

mysql> 
~~~


### 查看某一台主机的心跳记录

~~~
mysql> show @@heartbeat.detail where name= h101M1;
+--------+-------+-----------------+------+---------------------+--------------+
| NAME   | TYPE  | HOST            | PORT | TIME                | EXECUTE_TIME |
+--------+-------+-----------------+------+---------------------+--------------+
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 20:14:48 | 2            |
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 20:14:58 | 1            |
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 20:15:08 | 4            |
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 20:15:18 | 1            |
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 20:15:28 | 2            |
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 20:15:38 | 2            |
...
...
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 21:13:18 | 1            |
| h101M1 | mysql | 192.168.100.101 | 3306 | 2016-03-08 21:13:28 | 2            |
+--------+-------+-----------------+------+---------------------+--------------+
353 rows in set (0.03 sec)

mysql> 

~~~

### 查看系统参数

~~~
mysql> show @@sysparam ;
+-------------------------------+--------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PARAM_NAME                    | PARAM_VALUE        | PARAM_DESCR                                                                                                                                                                                                                                                                                                                       |
+-------------------------------+--------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| processors                    | 2                  | 主要用于指定系统可用的线程数，默认值为Runtime.getRuntime().availableProcessors()方法返回的值。主要影响processorBufferPool、processorBufferLocalPercent、processorExecutor属性。NIOProcessor的个数也是由这个属性定义的，所以调优的时候可以适当的调高这个属性。                                                                     |
| processorBufferChunk          | 40960B             | 指定每次分配Socket Direct Buffer的大小，默认是4096个字节。这个属性也影响buffer pool的长度。                                                                                                                                                                                                                                       |
| processorBufferPool           | 8192000B           | 指定bufferPool计算 比例值。由于每次执行NIO读、写操作都需要使用到buffer，系统初始化的时候会建立一定长度的buffer池来加快读、写的效率，减少建立buffer的时间                                                                                                                                                                          |
| processorBufferLocalPercent   | 100                | 就是用来控制分配这个pool的大小用的，但其也并不是一个准确的值，也是一个比例值。这个属性默认值为100。线缓存百分比 = bufferLocalPercent / processors属性。                                                                                                                                                                         |
| processorExecutor             | 4                  | 主要用于指定NIOProcessor上共享的businessExecutor固定线程池大小。mycat在需要处理一些异步逻辑的时候会把务提交到这个线程池中。新版本中这个连接池的使用频率不是很大了，可以设置一个较小的值。                                                                                                                                       |
| sequnceHandlerType            | 本地文件方式       | 指定使用Mycat全局序列的类型。                                                                                                                                                                                                                                                                                                     |
| Mysql_packetHeaderSize        | 4B                 | 指定Mysql协议中的报文头长度。默认4                                                                                                                                                                                                                                                                                                |
| Mysql_maxPacketSize           | 16M                | 指定Mysql协议可以携带的数据最大长度。默认16M                                                                                                                                                                                                                                                                                      |
| Mysql_idleTimeout             | 30分钟             | 指定连接的空闲超时时间。某连接在发起空闲检查下，发现距离上次使用超过了空闲时间，那么这个连接会被回收，就是被直接的关闭掉。默认30分钟                                                                                                                                                                                              |
| Mysql_charset                 | utf8               | 连接的初始化字符集。默认为utf8                                                                                                                                                                                                                                                                                                    |
| Mysql_txIsolation             | REPEATED_READ      | 前端连接的初始化事务隔离级别，只在初始化的时候使用，后续会根据客户端传递过来的属性对后端数据库连接进行同步。默认为REPEATED_READ                                                                                                                                                                                                   |
| Mysql_sqlExecuteTimeout       | 300秒              | SQL执行超时的时间，Mycat会检查连接上最后一次执行SQL的时间，若超过这个时间则会直接关闭这连接。默认时间300秒                                                                                                                                                                                                                      |
| Mycat_processorCheckPeriod    | 1秒                | 清理NIOProcessor上前后端空闲、超时和关闭连接的间隔时间。默认是1秒                                                                                                                                                                                                                                                                 |
| Mycat_dataNodeIdleCheckPeriod | 300秒              | 对后端连接进行空闲、超时检查的时间间隔，默认是300秒                                                                                                                                                                                                                                                                               |
| Mycat_dataNodeHeartbeatPeriod | 10秒               | 对后端所有读、写库发起心跳的间隔时间，默认是10秒                                                                                                                                                                                                                                                                                  |
| Mycat_bindIp                  | 0.0.0.0            | mycat服务监听的IP地址，默认值为0.0.0.0                                                                                                                                                                                                                                                                                            |
| Mycat_serverPort              | 8066               | mycat的使用端口，默认值为8066                                                                                                                                                                                                                                                                                                     |
| Mycat_managerPort             | 9066               | mycat的管理端口，默认值为9066                                                                                                                                                                                                                                                                                                     |
+-------------------------------+--------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
18 rows in set (0.01 sec)

mysql> 
~~~


### 查看mycat.log日志


~~~
mysql> show @@syslog limit=10;
+---------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| DATE                | LOG                                                                                                                                                                                                                                                                                                                                                                                                                                               |
+---------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 03/08 21:27:44.784  |   INFO [$_NIOConnector] (PhysicalDatasource.java:373) -not ilde connection in pool,create new connection for h202M1 of schema my4                                                                                                                                                                                                                                                                                                                 |
| 03/08 21:27:44.784  |   INFO [$_NIOConnector] (SQLJob.java:111) -can't get connection for sql :select user()                                                                                                                                                                                                                                                                                                                                                            |
| 03/08 21:27:44.784  |   INFO [$_NIOConnector] (AbstractConnection.java:458) -close connection,reason:java.net.NoRouteToHostException: No route to host ,MySQLConnection [id=0, lastTime=1457443661777, user=root, schema=my4, old shema=my4, borrowed=false, fromSlaveDB=false, threadId=0, charset=utf8, txIsolation=0, autocommit=true, attachment=null, respHandler=null, host=192.168.100.202, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false] |
| 03/08 21:27:41.783  |   INFO [$_NIOConnector] (PhysicalDatasource.java:373) -not ilde connection in pool,create new connection for h202M1 of schema my4                                                                                                                                                                                                                                                                                                                 |
| 03/08 21:27:41.783  |   INFO [$_NIOConnector] (SQLJob.java:111) -can't get connection for sql :select user()                                                                                                                                                                                                                                                                                                                                                            |
| 03/08 21:27:41.783  |   INFO [$_NIOConnector] (AbstractConnection.java:458) -close connection,reason:java.net.NoRouteToHostException: No route to host ,MySQLConnection [id=0, lastTime=1457443658778, user=root, schema=my4, old shema=my4, borrowed=false, fromSlaveDB=false, threadId=0, charset=utf8, txIsolation=0, autocommit=true, attachment=null, respHandler=null, host=192.168.100.202, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false] |
| 03/08 21:27:38.789  |   INFO [$_NIOConnector] (SQLJob.java:111) -can't get connection for sql :select user()                                                                                                                                                                                                                                                                                                                                                            |
| 03/08 21:27:38.789  |   INFO [$_NIOConnector] (AbstractConnection.java:458) -close connection,reason:java.net.ConnectException: Connection refused ,MySQLConnection [id=0, lastTime=1457443658778, user=root, schema=db2, old shema=db2, borrowed=false, fromSlaveDB=false, threadId=0, charset=utf8, txIsolation=0, autocommit=true, attachment=null, respHandler=null, host=localhost, port=3316, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]           |
| 03/08 21:27:38.789  |   INFO [$_NIOConnector] (SQLJob.java:111) -can't get connection for sql :select user()                                                                                                                                                                                                                                                                                                                                                            |
| 03/08 21:27:38.789  |   INFO [$_NIOConnector] (AbstractConnection.java:458) -close connection,reason:java.net.ConnectException: Connection refused ,MySQLConnection [id=0, lastTime=1457443658778, user=root, schema=db2, old shema=db2, borrowed=false, fromSlaveDB=false, threadId=0, charset=utf8, txIsolation=0, autocommit=true, attachment=null, respHandler=null, host=localhost, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]           |
+---------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
10 rows in set (0.06 sec)

mysql> 
~~~


### 查看白名单

~~~
mysql> show @@white;
Empty set (0.01 sec)

mysql> 
~~~


### 强制断开连接

~~~
mysql> show @@connection;
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
| PROCESSOR  | ID   | HOST            | PORT | LOCAL_PORT | USER | SCHEMA | CHARSET | NET_IN | NET_OUT | ALIVE_TIME(S) | RECV_BUFFER | SEND_QUEUE | txlevel | autocommit |
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
| Processor1 |    1 | 192.168.100.102 | 9066 |      43190 | cc   | NULL   | utf8:45 |   1998 |  143920 |          4021 |       40960 |          0 |         |            |
| Processor1 |    3 | 192.168.100.102 | 8066 |      33145 | cc   | cctest | utf8:45 |    603 |    3621 |          1863 |       40960 |          0 | 3       | true       |
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
2 rows in set (0.00 sec)

mysql> kill @@connection 3;
Query OK, 1 row affected (0.00 sec)

mysql> show @@connection;
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
| PROCESSOR  | ID   | HOST            | PORT | LOCAL_PORT | USER | SCHEMA | CHARSET | NET_IN | NET_OUT | ALIVE_TIME(S) | RECV_BUFFER | SEND_QUEUE | txlevel | autocommit |
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
| Processor1 |    1 | 192.168.100.102 | 9066 |      43190 | cc   | NULL   | utf8:45 |   2044 |  144628 |          4046 |       40960 |          0 |         |            |
+------------+------+-----------------+------+------------+------+--------+---------+--------+---------+---------------+-------------+------------+---------+------------+
1 row in set (0.01 sec)

mysql> 
~~~


### 重载配置

~~~
mysql> reload @@config;
Query OK, 1 row affected (0.15 sec)
Reload config success

mysql> 
~~~

重载的是 **schema.xml** 配置


### 重置SQL统计

~~~
mysql> reload @@user_stat;
Query OK, 1 row affected (0.03 sec)
Reset show @@sql  @@sql.sum @@sql.slow success

mysql> show @@sql;
Empty set (0.00 sec)

mysql> show @@sql.sum;
+------+------+------+------+------+------+--------------+--------------+-----------+
| ID   | USER | R    | W    | R%   | MAX  | TIME_COUNT   | TTL_COUNT    | LAST_TIME |
+------+------+------+------+------+------+--------------+--------------+-----------+
|    1 | cc   |    0 |    0 | 0    | 0    | [0, 0, 0, 0] | [0, 0, 0, 0] |         0 |
+------+------+------+------+------+------+--------------+--------------+-----------+
1 row in set (0.00 sec)

mysql> show @@sql.slow;
Empty set (0.00 sec)

mysql> 
~~~


### 关闭SQL统计

~~~
mysql> reload @@sqlstat=close ;
Query OK, 1 row affected (0.00 sec)
Set sql stat module isclosed=close, to succeed by manager. 

mysql> 
~~~

关闭后，`@@sql  @@sql.sum @@sql.slow` 就不会变化


### 打开SQL统计

~~~
mysql> reload @@sqlstat=open;
Query OK, 1 row affected (0.01 sec)
Set sql stat module isclosed=open, to fail by manager. 

mysql> 
~~~

### 离线上线

~~~
mysql> offline;
Query OK, 1 row affected (0.00 sec)

mysql> show @@server;
+--------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
| UPTIME       | USED_MEMORY | TOTAL_MEMORY | MAX_MEMORY | RELOAD_TIME   | ROLLBACK_TIME | CHARSET | STATUS | AVG_BUFPOOL_ITEM_SIZE |
+--------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
| 8m 52s 140ms |    26609688 |    129499136 |  477102080 | 1457444457222 |            -1 | utf8    | OFF    |                   575 |
+--------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
1 row in set (0.00 sec)

mysql> online;
Query OK, 1 row affected (0.00 sec)

mysql> show @@server;
+-------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
| UPTIME      | USED_MEMORY | TOTAL_MEMORY | MAX_MEMORY | RELOAD_TIME   | ROLLBACK_TIME | CHARSET | STATUS | AVG_BUFPOOL_ITEM_SIZE |
+-------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
| 9m 0s 630ms |    26972784 |    129499136 |  477102080 | 1457444457222 |            -1 | utf8    | ON     |                   572 |
+-------------+-------------+--------------+------------+---------------+---------------+---------+--------+-----------------------+
1 row in set (0.00 sec)

mysql> 
~~~

然而我并没变有发现前后操作有什么异常，依旧可以正常连接操作

### 定向清理慢SQL

~~~
mysql> clear @@slow where schema = cctest;
Query OK, 0 rows affected (0.01 sec)

mysql> clear @@slow where datanode = sd1;
Query OK, 0 rows affected (0.01 sec)

mysql> 
~~~


> **Tip:** 还有好多，虽然 `show @@help` 中有列出，但目前还不支持，也许还没实现(正在开发中) 如

~~~
mysql> reload @@route;
ERROR 1003 (HY000): Unsupported statement
mysql> reload @@user;
ERROR 1003 (HY000): Unsupported statement
mysql> rollback @@route ;
ERROR 1003 (HY000): Unsupported statement
mysql> rollback @@user ;
ERROR 1003 (HY000): Unsupported statement
mysql> show @@slow where datanode = sd1;
ERROR 1003 (HY000): Unsupported statement
mysql> show @@slow where schema =cctest;
ERROR 1003 (HY000): Unsupported statement
mysql> 
~~~

Mycat还是一个成长中的项目，还需要一些时间将这些功能完善，但目前来看，主体功能已经可以满足大部分需求

---


# 命令汇总

* **`grep 66 server.xml`**
* **`netstat  -ant | grep 66`**
* **`mysql -u cc -p -P 9066 -h 192.168.100.102`**

---


[mycat]:http://www.mycat.org.cn/
[mycat_doc]:http://www.mycat.org.cn/document/mycat1.5.2.pdf
