---
layout: post
title:  general_log 问题处理 
categories: linux  troubleshooting upgrade mysql 
wc: 707 3886 33288
excerpt: mysql 升级过程中出现关于 general_log 缺失的故障处理
comments: true
---


---

#前言


mysql 升级过程中出现了general_log的缺失，下面分享一下处理过程


---


#概要

* TOC
{:toc}


---

##什么是general_log

目前的mysql提供了两种查询日志， 这两种查询日志为 **普通日志（general log）** 和 **慢速日志（slow log）**

**慢速日志（slow log）** 可以提供一种机制，将执行时间超过指定长度的语句记录下来

{% highlight bash %}
mysql> show variables like "%slow%";
+------------------------------------+-------------------------+
| Variable_name                      | Value                   |
+------------------------------------+-------------------------+
| log_slow_admin_statements          | OFF                     |
| log_slow_filter                    |                         |
| log_slow_rate_limit                | 1                       |
| log_slow_rate_type                 | session                 |
| log_slow_slave_statements          | OFF                     |
| log_slow_sp_statements             | ON                      |
| log_slow_verbosity                 |                         |
| max_slowlog_files                  | 0                       |
| max_slowlog_size                   | 0                       |
| slow_launch_time                   | 2                       |
| slow_query_log                     | ON                      |
| slow_query_log_always_write_time   | 10.000000               |
| slow_query_log_file                | slow.log                |
| slow_query_log_timestamp_always    | OFF                     |
| slow_query_log_timestamp_precision | second                  |
| slow_query_log_use_global_control  |                         |
+------------------------------------+-------------------------+
16 rows in set (0.00 sec)

mysql>
{% endhighlight %}


**普通日志（general log）** 可以提供一种机制，能将打开日志期间所有的语句记录下来，由于开销比较大，所以正常情况下是关闭的，只在进行深度分析时打开



>General query log
>
>A type of log used for diagnosis and troubleshooting of SQL statements processed by the MySQL server. Can be stored in a file or in a database table. You must enable this feature through the general\_log configuration option to use it. You can disable it for a specific connection through the sql\_log\_off configuration option.
>
>Records a broader range of queries than the slow query log. Unlike the binary log, which is used for replication, the general query log contains SELECT statements and does not maintain strict ordering





{% highlight bash %}
mysql> show variables like "%general%";
+------------------+-----------------------------------+
| Variable_name    | Value                             |
+------------------+-----------------------------------+
| general_log      | OFF                               |
| general_log_file | /var/lib/mysql/general.log        |
+------------------+-----------------------------------+
2 rows in set (0.00 sec)

mysql> 
{% endhighlight %}

如果 **`log_output`** 设定为 **FILE** ，则会记录到 **`general_log_file`** 中去

{% highlight bash %}
mysql> show variables like "%log_output%";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+
1 row in set (0.00 sec)

mysql> 
{% endhighlight %}

如果为 **TABLE** 则会写到 **mysql.general_log** 中，关于general_log的详细机制，可以参考附录中的相关资料



---

##报错

在5.1->5.6升级过程中，执行upgrade之后产生如下报错

{% highlight bash %}
[root@upgrade-slave ~]# time mysql_upgrade -u root -p 
Enter password: 
Looking for 'mysql' as: mysql
Looking for 'mysqlcheck' as: mysqlcheck
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.event                                        OK
mysql.func                                         OK
mysql.general_log
Error    : Can't get stat of './mysql/general_log.CSV' (Errcode: 2 - No such file or directory)
Error    : Out of memory; check if mysqld or some other process uses all available memory; if not, you may have to use 'ulimit' to allow mysqld to use more memory or you can add more swap space
error    : Corrupt
mysql.help_category                                OK
mysql.help_keyword                                 OK
mysql.help_relation                                OK
mysql.help_topic                                   OK
mysql.host                                         OK
mysql.innodb_index_stats                           OK
mysql.innodb_table_stats                           OK
mysql.ndb_binlog_index                             OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.servers                                      OK
mysql.slave_master_info                            OK
mysql.slave_relay_log_info                         OK
mysql.slave_worker_info                            OK
mysql.slow_log                                     OK
mysql.tables_priv                                  OK
mysql.time_zone                                    OK
mysql.time_zone_leap_second                        OK
mysql.time_zone_name                               OK
mysql.time_zone_transition                         OK
mysql.time_zone_transition_type                    OK
mysql.user                                         OK

Repairing tables
mysql.general_log
Error    : Can't get stat of './mysql/general_log.CSV' (Errcode: 2 - No such file or directory)
Error    : Out of memory; check if mysqld or some other process uses all available memory; if not, you may have to use 'ulimit' to allow mysqld to use more memory or you can add more swap space
error    : Corrupt
Running 'mysql_fix_privilege_tables'...
Warning: Using a password on the command line interface can be insecure.
ERROR 13 (HY000) at line 25: Can't get stat of './mysql/general_log.CSV' (Errcode: 2 - No such file or directory)
ERROR 1243 (HY000) at line 26: Unknown prepared statement handler (stmt) given to EXECUTE
ERROR 1243 (HY000) at line 27: Unknown prepared statement handler (stmt) given to DEALLOCATE PREPARE
ERROR 13 (HY000) at line 1591: Can't get stat of './mysql/general_log.CSV' (Errcode: 2 - No such file or directory)
ERROR 13 (HY000) at line 1598: Can't get stat of './mysql/general_log.CSV' (Errcode: 2 - No such file or directory)
FATAL ERROR: Upgrade failed

real	0m5.161s
user	0m0.051s
sys	0m0.049s
[root@upgrade-slave ~]# 
{% endhighlight %}

由于缺少 **general_log.CSV** 从而报错

为了确认是否在备份过程中丢失的，进入master中查看

{% highlight bash %}
[root@old-master mysql]# ll  ./mysql/general_log.CSV
ls: ./mysql/general_log.CSV: No such file or directory
[root@old-master mysql]# cd mysql
[root@old-master mysql]# ll general_log.* 
-rw------- 1 mysql mysql 8776 Apr 21  2011 general_log.frm
[root@old-master mysql]# 
----------
mysql> use mysql
Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| servers                   |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
23 rows in set (0.00 sec)

mysql> show create table general_log;
ERROR 13 (HY000): Can't get stat of './mysql/general_log.CSV' (Errcode: 2)
mysql>
{% endhighlight %}

发现master中也没有，也许在很久以前损坏的

正常情况下是这样的

{% highlight bash %}
[root@normal-instancek mysql]# ll general_log.
general_log.CSM  general_log.CSV  general_log.frm  
[root@normal-instancek mysql]# du -sh general_log.*
4.0K	general_log.CSM
0	general_log.CSV
12K	general_log.frm
[root@normal-instancek mysql]# file  general_log.CSV
general_log.CSV: empty
[root@normal-instancek mysql]# cat general_log.CSV
[root@normal-instancek mysql]#
{% endhighlight %}

表结构如下

{% highlight bash %}
mysql> desc general_log;
+--------------+---------------------+------+-----+-------------------+-----------------------------+
| Field        | Type                | Null | Key | Default           | Extra                       |
+--------------+---------------------+------+-----+-------------------+-----------------------------+
| event_time   | timestamp           | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| user_host    | mediumtext          | NO   |     | NULL              |                             |
| thread_id    | bigint(21) unsigned | NO   |     | NULL              |                             |
| server_id    | int(10) unsigned    | NO   |     | NULL              |                             |
| command_type | varchar(64)         | NO   |     | NULL              |                             |
| argument     | mediumtext          | NO   |     | NULL              |                             |
+--------------+---------------------+------+-----+-------------------+-----------------------------+
6 rows in set (0.01 sec)

mysql> show create table general_log\G
*************************** 1. row ***************************
       Table: general_log
Create Table: CREATE TABLE `general_log` (
  `event_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `user_host` mediumtext NOT NULL,
  `thread_id` bigint(21) unsigned NOT NULL,
  `server_id` int(10) unsigned NOT NULL,
  `command_type` varchar(64) NOT NULL,
  `argument` mediumtext NOT NULL
) ENGINE=CSV DEFAULT CHARSET=utf8 COMMENT='General log'
1 row in set (0.00 sec)

mysql> select count(*) general_log;
+----------+
| count(*) |
+----------+
|        0 |
+----------+
1 row in set (0.00 sec)

mysql>
{% endhighlight %}


---

##进行手动修复

{% highlight bash %}
[root@upgrade-slave mysql]# touch general_log.CSV
[root@upgrade-slave mysql]# ll general_log.*
-rw-r--r--. 1 root  root     0 Dec 15 00:18 general_log.CSV
-rw-------. 1 mysql mysql 8776 Dec 14 23:37 general_log.frm
[root@upgrade-slave mysql]# chown  mysql.mysql general_log.CSV 
[root@upgrade-slave mysql]# chmod 660 general_log.CSV
[root@upgrade-slave mysql]# ll general_log.*
-rw-rw----. 1 mysql mysql    0 Dec 15 00:18 general_log.CSV
-rw-------. 1 mysql mysql 8776 Dec 14 23:37 general_log.frm
[root@upgrade-slave mysql]# 
{% endhighlight %}

---

##再次尝试升级


{% highlight bash %}
[root@upgrade-slave ~]# time mysql_upgrade -u root -p 
Enter password: 
Looking for 'mysql' as: mysql
Looking for 'mysqlcheck' as: mysqlcheck
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.event                                        OK
mysql.func                                         OK
mysql.general_log                                  OK
mysql.help_category                                OK
mysql.help_keyword                                 OK
mysql.help_relation                                OK
mysql.help_topic                                   OK
mysql.host                                         OK
mysql.innodb_index_stats                           OK
mysql.innodb_table_stats                           OK
mysql.ndb_binlog_index                             OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.servers                                      OK
mysql.slave_master_info                            OK
mysql.slave_relay_log_info                         OK
mysql.slave_worker_info                            OK
mysql.slow_log                                     OK
mysql.tables_priv                                  OK
mysql.time_zone                                    OK
mysql.time_zone_leap_second                        OK
mysql.time_zone_name                               OK
mysql.time_zone_transition                         OK
mysql.time_zone_transition_type                    OK
mysql.user                                         OK
Running 'mysql_fix_privilege_tables'...
Warning: Using a password on the command line interface can be insecure.
ERROR 1194 (HY000) at line 1591: Table 'general_log' is marked as crashed and should be repaired
ERROR 1194 (HY000) at line 1598: Table 'general_log' is marked as crashed and should be repaired
FATAL ERROR: Upgrade failed

real	0m5.853s
user	0m0.055s
sys	0m0.066s
[root@upgrade-slave ~]#
{% endhighlight %}

这时general_log 被认为已经crashed了，需要修复一下


---

##repair table

{% highlight bash %}
[root@upgrade-slave ~]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 5.6.27-75.0-log Percona Server (GPL), Release 75.0, Revision 8bb53b6

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysql
Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
29 rows in set (0.00 sec)

mysql> repair table `general_log`;
+-------------------+--------+----------+----------+
| Table             | Op     | Msg_type | Msg_text |
+-------------------+--------+----------+----------+
| mysql.general_log | repair | status   | OK       |
+-------------------+--------+----------+----------+
1 row in set (0.03 sec)

mysql> quit
Bye
[root@upgrade-slave ~]# 
{% endhighlight %}

此时多出来一个文件 **general_log.CSM** 

{% highlight bash %}
[root@upgrade-slave mysql]# ll general_log.*
-rw-rw----. 1 mysql mysql   35 Dec 15 00:22 general_log.CSM
-rw-rw----. 1 mysql mysql    0 Dec 15 00:18 general_log.CSV
-rw-------. 1 mysql mysql 8776 Dec 15 00:22 general_log.frm
[root@upgrade-slave mysql]# 
{% endhighlight %}

再次尝试升级

{% highlight bash %}
[root@upgrade-slave ~]# time mysql_upgrade -u root -p
Enter password: 
Looking for 'mysql' as: mysql
Looking for 'mysqlcheck' as: mysqlcheck
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.event                                        OK
mysql.func                                         OK
mysql.general_log                                  OK
mysql.help_category                                OK
mysql.help_keyword                                 OK
mysql.help_relation                                OK
mysql.help_topic                                   OK
mysql.host                                         OK
mysql.innodb_index_stats                           OK
mysql.innodb_table_stats                           OK
mysql.ndb_binlog_index                             OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.servers                                      OK
mysql.slave_master_info                            OK
mysql.slave_relay_log_info                         OK
mysql.slave_worker_info                            OK
mysql.slow_log                                     OK
mysql.tables_priv                                  OK
mysql.time_zone                                    OK
mysql.time_zone_leap_second                        OK
mysql.time_zone_name                               OK
mysql.time_zone_transition                         OK
mysql.time_zone_transition_type                    OK
mysql.user                                         OK
Running 'mysql_fix_privilege_tables'...
Warning: Using a password on the command line interface can be insecure.
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
Running 'mysqlcheck' with connection arguments: '--port=3306' '--socket=/var/lib/mysql/mysql.sock' 
Warning: Using a password on the command line interface can be insecure.
azheng_db.answers                                  OK
azheng_db.feedbacks                                OK
azheng_db.logged_exceptions                        OK
azheng_db.question_answers                         OK
azheng_db.questions                                OK
azheng_db.rule_answers                             OK
...
...
text_db.categories                                 OK
text_db.exception_logger_logged_exceptions         OK
text_db.missions                                   OK
text_db.schema_migrations                          OK
text_db.taggings                                   OK
text_db.tags                                       OK
text_db.users                                      OK
OK

real	0m59.267s
user	0m0.098s
sys	0m0.099s
[root@upgrade-slave ~]# 
{% endhighlight %}

执行正常，顺利完成升级


重启服务，查看error日志里的信息，关注有无报错

{% highlight bash %}
[root@upgrade-slave ~]# /etc/init.d/mysql  stop 
Shutting down MySQL (Percona Server)..[  OK  ]

[root@upgrade-slave ~]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server)...[  OK  ]

[root@upgrade-slave ~]#
{% endhighlight %}

检查无误后，配置master，启动同步就可以了


---

#命令汇总


* **`time mysql_upgrade -u root -p`**
* **`ll  ./mysql/general_log.CSV`**
* **`ll general_log.`**
* **`du -sh general_log.*`**
* **`file  general_log.CSV`**
* **`cat general_log.CSV`**
* **`touch general_log.CSV`**
* **`chown  mysql.mysql general_log.CSV`**
* **`chmod 660 general_log.CSV`**

---

#附

##The General Query Log

**[The General Query Log][query-log]**

{% highlight bash %}
5.2.3 The General Query Log

The general query log is a general record of what mysqld is doing. The server writes information to this log when clients connect or disconnect, and it logs each SQL statement received from clients. The general query log can be very useful when you suspect an error in a client and want to know exactly what the client sent to mysqld.

mysqld writes statements to the query log in the order that it receives them, which might differ from the order in which they are executed. This logging order is in contrast with that of the binary log, for which statements are written after they are executed but before any locks are released. In addition, the query log may contain statements that only select data while such statements are never written to the binary log.

When using statement-based binary logging on a replication master server, statements received by its slaves are written to the query log of each slave. Statements are written to the query log of the master server if a client reads events with the mysqlbinlog utility and passes them to the server.

However, when using row-based binary logging, updates are sent as row changes rather than SQL statements, and thus these statements are never written to the query log when binlog_format is ROW. A given update also might not be written to the query log when this variable is set to MIXED, depending on the statement used. See Section 17.1.2.1, “Advantages and Disadvantages of Statement-Based and Row-Based Replication”, for more information.

By default, the general query log is disabled. To specify the initial general query log state explicitly, use --general_log[={0|1}]. With no argument or an argument of 1, --general_log enables the log. With an argument of 0, this option disables the log. To specify a log file name, use --general_log_file=file_name. To specify the log destination, use --log-output (as described in Section 5.2.1, “Selecting General Query and Slow Query Log Output Destinations”).

If you specify no name for the general query log file, the default name is host_name.log. The server creates the file in the data directory unless an absolute path name is given to specify a different directory.

To disable or enable the general query log or change the log file name at runtime, use the global general_log and general_log_file system variables. Set general_log to 0 (or OFF) to disable the log or to 1 (or ON) to enable it. Set general_log_file to specify the name of the log file. If a log file already is open, it is closed and the new file is opened.

When the general query log is enabled, the server writes output to any destinations specified by the --log-output option or log_output system variable. If you enable the log, the server opens the log file and writes startup messages to it. However, further logging of queries to the file does not occur unless the FILE log destination is selected. If the destination is NONE, the server writes no queries even if the general log is enabled. Setting the log file name has no effect on logging if the log destination value does not contain FILE.

Server restarts and log flushing do not cause a new general query log file to be generated (although flushing closes and reopens it). To rename the file and create a new one, use the following commands:

shell> mv host_name.log host_name-old.log
shell> mysqladmin flush-logs
shell> mv host_name-old.log backup-directory

On Windows, use rename rather than mv.

You can also rename the general query log file at runtime by disabling the log:

SET GLOBAL general_log = 'OFF';

With the log disabled, rename the log file externally; for example, from the command line. Then enable the log again:

SET GLOBAL general_log = 'ON';

This method works on any platform and does not require a server restart.

The session sql_log_off variable can be set to ON or OFF to disable or enable general query logging for the current connection.

As of MySQL 5.6.3, passwords in statements written to the general query log are rewritten by the server not to occur literally in plain text. Password rewriting can be suppressed for the general query log by starting the server with the --log-raw option. This option may be useful for diagnostic purposes, to see the exact text of statements as received by the server, but for security reasons is not recommended for production use.

Before MySQL 5.6.3, passwords in statements are not rewritten and the general query log should be protected. See Section 6.1.2.3, “Passwords and Logging”.

One implication of the introduction of password rewriting in MySQL 5.6.3 is that statements that cannot be parsed (due, for example, to syntax errors) are no longer written to the general query log because they cannot be known to be password free. Use cases that require logging of all statements including those with errors should use the --log-raw option, bearing in mind that this also bypasses password writing.
{% endhighlight %}


##log-output


| `Command-Line Format`     |`--log-output=name` | |
| :------- | :---- | :--- |
|System Variable|	Name	|log_output|
||Variable Scope	|Global|
||Dynamic Variable|	Yes|
|Permitted Values	|Type	|set|
||Default|	FILE|
||Valid Values|	TABLE|
|||FILE|
|||NONE|


##Selecting General Query and Slow Query Log Output Destinations

**[Selecting General Query and Slow Query Log Output Destinations][log-destinations]**

{% highlight bash %}
5.2.1 Selecting General Query and Slow Query Log Output Destinations

MySQL Server provides flexible control over the destination of output to the general query log and the slow query log, if those logs are enabled. Possible destinations for log entries are log files or the general_log and slow_log tables in the mysql database. Either or both destinations can be selected.

Before MySQL 5.5.7, logging to tables incurs significantly more server overhead than logging to files. If you enable the general log or slow query log and require highest performance, you should use file logging, not table logging.

Log control at server startup. The --log-output option specifies the destination for log output. This option does not in itself enable the logs. Its syntax is --log-output[=value,...]:

If --log-output is given with a value, the value should be a comma-separated list of one or more of the words TABLE (log to tables), FILE (log to files), or NONE (do not log to tables or files). NONE, if present, takes precedence over any other specifiers.

If --log-output is omitted, the default logging destination is FILE.

The general_log system variable controls logging to the general query log for the selected log destinations. If specified at server startup, general_log takes an optional argument of 1 or 0 to enable or disable the log. To specify a file name other than the default for file logging, set the general_log_file variable. Similarly, the slow_query_log variable controls logging to the slow query log for the selected destinations and setting slow_query_log_file specifies a file name for file logging. If either log is enabled, the server opens the corresponding log file and writes startup messages to it. However, further logging of queries to the file does not occur unless the FILE log destination is selected.

Examples:

To write general query log entries to the log table and the log file, use --log-output=TABLE,FILE to select both log destinations and --general_log to enable the general query log.

To write general and slow query log entries only to the log tables, use --log-output=TABLE to select tables as the log destination and --general_log and --slow_query_log to enable both logs.

To write slow query log entries only to the log file, use --log-output=FILE to select files as the log destination and --slow_query_log to enable the slow query log. (In this case, because the default log destination is FILE, you could omit the --log-output option.)

Log control at runtime. The system variables associated with log tables and files enable runtime control over logging:

The global log_output system variable indicates the current logging destination. It can be modified at runtime to change the destination.

The global general_log and slow_query_log variables indicate whether the general query log and slow query log are enabled (ON) or disabled (OFF). You can set these variables at runtime to control whether the logs are enabled.

The global general_log_file and slow_query_log_file variables indicate the names of the general query log and slow query log files. You can set these variables at server startup or at runtime to change the names of the log files.

To disable or enable general query logging for the current connection, set the session sql_log_off variable to ON or OFF.

The use of tables for log output offers the following benefits:

Log entries have a standard format. To display the current structure of the log tables, use these statements:

SHOW CREATE TABLE mysql.general_log;
SHOW CREATE TABLE mysql.slow_log;

Log contents are accessible through SQL statements. This enables the use of queries that select only those log entries that satisfy specific criteria. For example, to select log contents associated with a particular client (which can be useful for identifying problematic queries from that client), it is easier to do this using a log table than a log file.

Logs are accessible remotely through any client that can connect to the server and issue queries (if the client has the appropriate log table privileges). It is not necessary to log in to the server host and directly access the file system.

The log table implementation has the following characteristics:

In general, the primary purpose of log tables is to provide an interface for users to observe the runtime execution of the server, not to interfere with its runtime execution.

CREATE TABLE, ALTER TABLE, and DROP TABLE are valid operations on a log table. For ALTER TABLE and DROP TABLE, the log table cannot be in use and must be disabled, as described later.

By default, the log tables use the CSV storage engine that writes data in comma-separated values format. For users who have access to the .CSV files that contain log table data, the files are easy to import into other programs such as spreadsheets that can process CSV input.

The log tables can be altered to use the MyISAM storage engine. You cannot use ALTER TABLE to alter a log table that is in use. The log must be disabled first. No engines other than CSV or MyISAM are legal for the log tables.

To disable logging so that you can alter (or drop) a log table, you can use the following strategy. The example uses the general query log; the procedure for the slow query log is similar but uses the slow_log table and slow_query_log system variable.

SET @old_log_state = @@global.general_log;
SET GLOBAL general_log = 'OFF';
ALTER TABLE mysql.general_log ENGINE = MyISAM;
SET GLOBAL general_log = @old_log_state;

TRUNCATE TABLE is a valid operation on a log table. It can be used to expire log entries.

RENAME TABLE is a valid operation on a log table. You can atomically rename a log table (to perform log rotation, for example) using the following strategy:

USE mysql;
DROP TABLE IF EXISTS general_log2;
CREATE TABLE general_log2 LIKE general_log;
RENAME TABLE general_log TO general_log_backup, general_log2 TO general_log;

As of MySQL 5.5.7, CHECK TABLE is a valid operation on a log table.

LOCK TABLES cannot be used on a log table.

INSERT, DELETE, and UPDATE cannot be used on a log table. These operations are permitted only internally to the server itself.

FLUSH TABLES WITH READ LOCK and the state of the read_only system variable have no effect on log tables. The server can always write to the log tables.

Entries written to the log tables are not written to the binary log and thus are not replicated to slave servers.

To flush the log tables or log files, use FLUSH TABLES or FLUSH LOGS, respectively.

Partitioning of log tables is not permitted.

Before MySQL 5.5.25, mysqldump does not dump the general_log or slow_query_log tables for dumps of the mysql database. As of 5.5.25, the dump includes statements to recreate those tables so that they are not missing after reloading the dump file. Log table contents are not dumped.
{% endhighlight %}


---

[query-log]:http://dev.mysql.com/doc/refman/5.6/en/query-log.html
[log-destinations]:http://dev.mysql.com/doc/refman/5.5/en/log-destinations.html

