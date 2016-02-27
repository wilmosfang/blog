---
layout: post
title: Mysql MHA 搭建 (一) percona5.1 安装
author: wilmosfang
categories: linux mysql mha cluster
wc: 562 1390 15045
excerpt: follow me
comments: true
---

---

# 前言


高可用非常重要，任何系统架构都要去考虑，数据库尤为如此

开源的系统架构中使用mysql的份额很大

mysql的各种高可用方案里mha比较高效和快捷

下面我分享一下前段时间研究MHA过程中积累下来的经验

---


# 概要

* TOC
{:toc}


---


## 环境

* 三个虚拟机:

m1:192.168.75.11/24

m2:192.168.75.12/24

s:192.168.75.13/24

* 一个浮动ip:

192.168.66.66/24


* 操作系统版本:

CentOS release 6.6 (Final)

* Mha软件版本:

MHA 0.53 for RHEL6

* Keepalived软件版本:

Keepalived v1.2.13 (10/15,2014)



>目前[MHA](http://code.google.com/p/mysql-master-ha/)的最新版本是0.56
>
>[Percona Server](http://www.percona.com/doc/percona-server/5.6/)的最新版本是5.6
>
>之所以使用这两个老版本是为了刻意模拟具体环境，其实老版本带来的不便要多于新版本，实际工作中建议使用最新版本


---

## 系统架构

正常状态 : m1为主库，m2与s作为备库并列从m1那里进行复制，浮动ip挂在m1上


|host|ip |role| keepalived|vip|OS|mha|
|:--- |:---:| :---: |:---:| :---:|:---:|---:|
|m1|192.168.75.11 |master|v1.2.13|192.168.66.66|CentOS6.6|node|
|m2|192.168.75.11|slave|v1.2.13|null|CentOS6.6|node|
|s|192.168.75.13|slave|null|null|CentOS6.6|node,manager|

{% highlight bash %}
m1 	192.168.75.11 	192.168.66.66
|-m2	192.168.75.12
`-s	192.168.75.13
{% endhighlight %}

切换后状态 : 切换后m1从集群中移出，原来的后备master m2 升级为主master，原来的s将主指向m2，继续作为库，浮动IP飘到m2上，这个过程是由mha软件自动来完成


|host|ip |role| keepalived|vip|OS|mha|
|:--- |:---:| :---: |:---:| :---:|:---:|---:|
|m1|192.168.75.11 |null|v1.2.13|null|CentOS6.6|node|
|m2|192.168.75.11|master|v1.2.13|192.168.66.66|CentOS6.6|node|
|s|192.168.75.13|slave|null|null|CentOS6.6|node,manager|

{% highlight bash %}
m2	192.168.75.12 	192.168.66.66
`s	192.168.75.13
{% endhighlight %}



---

## 安装Percona Server 5.1

系统环境

{% highlight bash %}
[root@m1 ~]# lsb_release  -a 
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID:	CentOS
Description:	CentOS release 6.6 (Final)
Release:	6.6
Codename:	Final
[root@m1 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@m1 ~]# uname -a 
Linux m1 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@m1 ~]#
{% endhighlight %}

检查系统防火墙

{% highlight bash %}
[root@m1 tmp]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:ssh
REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
[root@m1 tmp]#
{% endhighlight %}

这个防火墙设置会使数据库主备复制失败，为了简便，直接关掉

>更安全的做法是开3306端口

{% highlight bash %}
[root@m1 tmp]# /etc/init.d/iptables  stop
iptables: Setting chains to policy ACCEPT: filter [  OK  ]
iptables: Flushing firewall rules: [  OK  ]
iptables: Unloading modules: [  OK  ]
[root@m1 tmp]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
[root@m1 tmp]#
[root@m1 tmp]# chkconfig  --list | grep iptables
iptables        0:off   1:off   2:on    3:on    4:on    5:on    6:off
[root@m1 tmp]# chkconfig iptables off
[root@m1 tmp]#
{% endhighlight %}

同样检查SELINUX，关闭SELINUX

{% highlight bash %}
[root@m1 tmp]# getenforce
Enforcing
[root@m1 tmp]# setenforce 0
[root@m1 tmp]# getenforce
Permissive
[root@m1 tmp]# 
[root@m1 tmp]# cat /etc/rc.local 
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
setenforce 0 
[root@m1 tmp]#
{% endhighlight %}

>关闭selinux也可以修改**/etc/selinux/config**,设置**SELINUX=disabled**,这个要重启后才可以生效


参照[Percona Yum Repo][percona yum]配置本地yum库

{% highlight bash %}
[root@m1 tmp]# wget http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm 
[root@m1 tmp]# ls
percona-release-0.1-3.noarch.rpm
[root@m1 tmp]# rpm -ivh percona-release-0.1-3.noarch.rpm 
warning: percona-release-0.1-3.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:percona-release        ########################################### [100%]
[root@m1 tmp]#
{% endhighlight %}

安装percona server 5.1 ,系统自动解决了依赖关系，安装了Percona-Server-client-51

这两个包，都需要安装

{% highlight bash %}
[root@m1 tmp]# yum install  Percona-Server-server-51.x86_64
...
...
Installed:
  Percona-Server-server-51.x86_64 0:5.1.73-rel14.12.624.rhel6                                      
  Percona-Server-shared-51.x86_64 0:5.1.73-rel14.12.624.rhel6                                      

Dependency Installed:
  Percona-Server-client-51.x86_64 0:5.1.73-rel14.12.624.rhel6                                      

Replaced:
  mysql-libs.x86_64 0:5.1.73-3.el6_5  

Complete!
[root@m1 tmp]# 
{% endhighlight %}

配置/etc/my.cnf文件

 配置my.cnf文件过程中，有几点需要注意
 
 *  配置log-bin=mysql-bin以防止主机名发生变化后找不到bin log
 *  配置relay-log=relay-bin以防止主机名发生变化后找不到relay log
 *  配置relay\_log\_purge=off以防止中继日志被删除，也可以后期手动设置
 *  建议配置read_only=on以进一步保障安全，这一步不是必须，可以后期，手动设置
 *  使用/var/lib/mysql/*.err 来进行排错，对于直接使用其它数据库my.cnf的情况特别有效
 


初始化数据库

{% highlight bash %}
[root@m1 tmp]# mysql_install_db --defaults-file=/etc/my.cnf
Installing MySQL system tables...
OK
Filling help tables...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/bin/mysqladmin -u root password 'new-password'
/usr/bin/mysqladmin -u root -h localhost.localdomain password 'new-password'

Alternatively you can run:
/usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr ; /usr/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/mysql-test ; perl mysql-test-run.pl

Please report any problems with the /usr/bin/mysqlbug script!

Percona recommends that all production deployments be protected with a support
contract (http://www.percona.com/mysql-suppport/) to ensure the highest uptime,
be eligible for hot fixes, and boost your team's productivity.

[root@m1 tmp]# echo $?
0
[root@m1 tmp]#
{% endhighlight %}



使用 **/etc/init.d/mysql  start** 启动数据库,然后使用下面命令设定root密码

{% highlight bash %}
[root@m1 mysql]# mysqladmin  -u root password 'mysql'
{% endhighlight %}

重新启动mysql，使用之前设置的密码进行登录

{% highlight bash %}
[root@m1 mysql]# /etc/init.d/mysql  start
Starting MySQL (Percona Server)[  OK  ]
[root@m1 mysql]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.1.73-14.12 Percona Server (GPL), Release 14.12, Revision 624

Copyright (c) 2009-2013 Percona LLC and/or its affiliates
Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

mysql> show grants\G
*************************** 1. row ***************************
Grants for root@localhost: GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*E74858DB86EBA20BC33D0AECAE8A8108C56B17FA' WITH GRANT OPTION
1 row in set (0.00 sec)

mysql> 
{% endhighlight %}

导入一些数据到库中


{% highlight bash %}
mysql> use test;
Database changed
mysql> \! ls
bin  demo.sql  mha
mysql> source demo.sql
Query OK, 0 rows affected (0.00 sec)
...
...
Query OK, 0 rows affected (0.01 sec)

Query OK, 9990 rows affected (0.16 sec)
Records: 9990  Duplicates: 0  Warnings: 0
...
...

Query OK, 0 rows affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| test               |
+--------------------+
3 rows in set (0.00 sec)

mysql> use test;
Database changed
mysql> show tables;
+-------------------+
| Tables_in_test    |
+-------------------+
| books             |
| schema_migrations |
+-------------------+
2 rows in set (0.00 sec)

mysql> select count(*) from books ;
+----------+
| count(*) |
+----------+
|     9990 |
+----------+
1 row in set (0.00 sec)

mysql>
{% endhighlight %}

创建用户repl，用来进行主备同步，这个操作在m1和m2上都要做

{% highlight bash %}
mysql> grant replication slave on *.* to 'repl@'192.168.75.%' identified by 'repl';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>
{% endhighlight %}

设置主备复制

安装好percona server 5.1 后，登入mysql 在m2和s上执行如下操作

{% highlight bash %}
mysql> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='192.168.75.11',
    -> MASTER_USER='repl',
    -> MASTER_PASSWORD='repl',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=0;
Query OK, 0 rows affected (0.12 sec) 

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State:
                  Master_Host: 192.168.75.11
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 4
               Relay_Log_File: localhost-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 4
              Relay_Log_Space: 106
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
1 row in set (0.00 sec)

mysql> start slave ;
Query OK, 0 rows affected (0.01 sec)

mysql>


{% endhighlight %}

创建用户mhauser，来进行主备切换的必要操作，由mha来调用

{% highlight bash %}
mysql> grant select , insert , update , delete , create, drop , super , process on *.* to 'mhauser@'192.168.75.%' identified by 'xxx';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> 
{% endhighlight %}

因为同步的原因，此时m1,m2,s上都有mhauser这个用户，可以登入到m2与s上进行检查，也顺便检查主备同步的效果

{% highlight bash %}
mysql> select * from mysql.user where user='mhauser'\G
*************************** 1. row ***************************
                 Host: 192.168.75.%
                 User: mhauser
             Password: *9FD52E1EF06D98F0E523B9AA495DAE60480CA19C
          Select_priv: Y
          Insert_priv: Y
          Update_priv: Y
          Delete_priv: Y
          Create_priv: Y
            Drop_priv: Y
          Reload_priv: N
        Shutdown_priv: N
         Process_priv: Y
            File_priv: N
           Grant_priv: N
      References_priv: N
           Index_priv: N
           Alter_priv: N
         Show_db_priv: N
           Super_priv: Y
Create_tmp_table_priv: N
     Lock_tables_priv: N
         Execute_priv: N
      Repl_slave_priv: N
     Repl_client_priv: N
     Create_view_priv: N
       Show_view_priv: N
  Create_routine_priv: N
   Alter_routine_priv: N
     Create_user_priv: N
           Event_priv: N
         Trigger_priv: N
             ssl_type: 
           ssl_cipher: 
          x509_issuer: 
         x509_subject: 
        max_questions: 0
          max_updates: 0
      max_connections: 0
 max_user_connections: 0
1 row in set (0.00 sec)

mysql> 
{% endhighlight %}

确保m1配置如下

{% highlight bash %}
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | OFF   |
+---------------+-------+
{% endhighlight %}

m2和s配置如下

{% highlight bash %}
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     |  ON   |
+---------------+-------+
{% endhighlight %}

致此，mysql这边的配置已经完成


接下来进行MHA与keepalived，还有系统方面的部分配置 

[percona yum]: http://www.percona.com/doc/percona-xtrabackup/2.2/installation/yum_repo.html






