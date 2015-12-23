---
layout: post
title: Mysql MHA 搭建 (四) mha failover 测试
categories: linux mha mysql  keepalived cluster
wc: 1046 5200 49647
excerpt: follow me
comments: true
---


---

前言
=

继前一篇[Mysql MHA 搭建 (三) keepalived 1.2.13 安装][mha3] , 这篇将演示如何操作mha进行master failover


---


#概要

* TOC
{:toc}


---

##在线迁移测试 

使用**masterha_master_switch**进行在线切换

####迁移前检查

on m1

{% highlight bash %}
[mysql@m1 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.11/24 brd 192.168.75.255 scope global eth4
    inet6 fe80::20c:29ff:fe25:abec/64 scope link 
       valid_lft forever preferred_lft forever
3: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.115/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe25:abf6/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m1 ~]$ F_rs.bash 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.12
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000015
          Read_Master_Log_Pos: 106
               Relay_Log_File: m1-relay-bin.000008
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000015
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 106
              Relay_Log_Space: 845
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000023 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
[mysql@m1 ~]$ 
{% endhighlight %}
on m2

{% highlight bash %}
[mysql@m2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:c7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.12/24 brd 192.168.75.255 scope global eth5
    inet 192.168.66.66/24 scope global eth5
    inet6 fe80::20c:29ff:fed8:a1c7/64 scope link 
       valid_lft forever preferred_lft forever
3: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:d1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.94/24 brd 192.168.1.255 scope global eth6
    inet6 fe80::20c:29ff:fed8:a1d1/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m2 ~]$ F_rs.bash 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000015 |      106 |              |                  |
+------------------+----------+--------------+------------------+
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
[mysql@m2 ~]$ 
{% endhighlight %}

on s

{% highlight bash %}
[mysql@s script]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:8b:41:c5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.13/24 brd 192.168.75.255 scope global eth8
    inet6 fe80::20c:29ff:fe8b:41c5/64 scope link 
       valid_lft forever preferred_lft forever
3: eth7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:8b:41:cf brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.95/24 brd 192.168.1.255 scope global eth7
    inet6 fe80::20c:29ff:fe8b:41cf/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@s script]$ F_rs.bash 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.12
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000015
          Read_Master_Log_Pos: 106
               Relay_Log_File: s-relay-bin.000024
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000015
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 106
              Relay_Log_Space: 843
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000005 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
[mysql@s script]$ 
{% endhighlight %}

在s上进行复制检查

{% highlight bash %}
[mysql@s script]$ masterha_check_repl --conf=/etc/app1.cnf 
Tue Mar 31 19:03:18 2015 - [info] Reading default configuratoins from /etc/masterha_default.cnf..
Tue Mar 31 19:03:18 2015 - [info] Reading application default configurations from /etc/app1.cnf..
Tue Mar 31 19:03:18 2015 - [info] Reading server configurations from /etc/app1.cnf..
Tue Mar 31 19:03:18 2015 - [info] MHA::MasterMonitor version 0.53.
Tue Mar 31 19:03:18 2015 - [info] Dead Servers:
Tue Mar 31 19:03:18 2015 - [info] Alive Servers:
Tue Mar 31 19:03:18 2015 - [info]   m1(192.168.75.11:3306)
Tue Mar 31 19:03:18 2015 - [info]   m2(192.168.75.12:3306)
Tue Mar 31 19:03:18 2015 - [info]   s(192.168.75.13:3306)
Tue Mar 31 19:03:18 2015 - [info] Alive Slaves:
Tue Mar 31 19:03:18 2015 - [info]   m1(192.168.75.11:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:03:18 2015 - [info]     Replicating from 192.168.75.12(192.168.75.12:3306)
Tue Mar 31 19:03:18 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Tue Mar 31 19:03:18 2015 - [info]   s(192.168.75.13:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:03:18 2015 - [info]     Replicating from 192.168.75.12(192.168.75.12:3306)
Tue Mar 31 19:03:18 2015 - [info]     Not candidate for the new Master (no_master is set)
Tue Mar 31 19:03:18 2015 - [info] Current Alive Master: m2(192.168.75.12:3306)
Tue Mar 31 19:03:18 2015 - [info] Checking slave configurations..
Tue Mar 31 19:03:18 2015 - [info] Checking replication filtering settings..
Tue Mar 31 19:03:18 2015 - [info]  binlog_do_db= , binlog_ignore_db= 
Tue Mar 31 19:03:18 2015 - [info]  Replication filtering check ok.
Tue Mar 31 19:03:18 2015 - [info] Starting SSH connection tests..
Tue Mar 31 19:03:23 2015 - [info] All SSH connection tests passed successfully.
Tue Mar 31 19:03:23 2015 - [info] Checking MHA Node version..
Tue Mar 31 19:03:25 2015 - [info]  Version check ok.
Tue Mar 31 19:03:25 2015 - [info] Checking SSH publickey authentication settings on the current master..
Tue Mar 31 19:03:25 2015 - [info] HealthCheck: SSH to m2 is reachable.
Tue Mar 31 19:03:26 2015 - [info] Master MHA Node version is 0.53.
Tue Mar 31 19:03:26 2015 - [info] Checking recovery script configurations on the current master..
Tue Mar 31 19:03:26 2015 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/home/mysql/mha/save_binary_logs_test --manager_version=0.53 --start_file=mysql-bin.000015 
Tue Mar 31 19:03:26 2015 - [info]   Connecting to mysql@m2(m2).. 
  Creating /home/mysql/mha if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin.000015
Tue Mar 31 19:03:27 2015 - [info] Master setting check done.
Tue Mar 31 19:03:27 2015 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Tue Mar 31 19:03:27 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=m1 --slave_ip=192.168.75.11 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Tue Mar 31 19:03:27 2015 - [info]   Connecting to mysql@192.168.75.11(m1:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to m1-relay-bin.000008
    Temporary relay log file is /var/lib/mysql/m1-relay-bin.000008
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Tue Mar 31 19:03:28 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=s --slave_ip=192.168.75.13 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Tue Mar 31 19:03:28 2015 - [info]   Connecting to mysql@192.168.75.13(s:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to s-relay-bin.000024
    Temporary relay log file is /var/lib/mysql/s-relay-bin.000024
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Tue Mar 31 19:03:29 2015 - [info] Slaves settings check done.
Tue Mar 31 19:03:29 2015 - [info] 
m2 (current master)
 +--m1
 +--s

Tue Mar 31 19:03:29 2015 - [info] Checking replication health on m1..
Tue Mar 31 19:03:29 2015 - [info]  ok.
Tue Mar 31 19:03:29 2015 - [info] Checking replication health on s..
Tue Mar 31 19:03:29 2015 - [info]  ok.
Tue Mar 31 19:03:29 2015 - [info] Checking master_ip_failover_script status:
Tue Mar 31 19:03:29 2015 - [info]   /home/mysql/mha/script/master_ip_failover --command=status --ssh_user=mysql --orig_master_host=m2 --orig_master_ip=192.168.75.12 --orig_master_port=3306 
Tue Mar 31 19:03:29 2015 - [info]  OK.
Tue Mar 31 19:03:29 2015 - [warning] shutdown_script is not defined.
Tue Mar 31 19:03:29 2015 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.
[mysql@s script]$ 
{% endhighlight %}

####进行在线切换


确认无误后进行手动切换

{% highlight bash %}
[mysql@s script]$ masterha_master_switch --master_state=alive --conf=/etc/app1.cnf  --new_master_host=m1 --interactive=0
Tue Mar 31 19:04:58 2015 - [info] MHA::MasterRotate version 0.53.
Tue Mar 31 19:04:58 2015 - [info] Starting online master switch..
Tue Mar 31 19:04:58 2015 - [info] 
Tue Mar 31 19:04:58 2015 - [info] * Phase 1: Configuration Check Phase..
Tue Mar 31 19:04:58 2015 - [info] 
Tue Mar 31 19:04:58 2015 - [info] Reading default configuratoins from /etc/masterha_default.cnf..
Tue Mar 31 19:04:58 2015 - [info] Reading application default configurations from /etc/app1.cnf..
Tue Mar 31 19:04:58 2015 - [info] Reading server configurations from /etc/app1.cnf..
Tue Mar 31 19:04:58 2015 - [info] Current Alive Master: m2(192.168.75.12:3306)
Tue Mar 31 19:04:58 2015 - [info] Alive Slaves:
Tue Mar 31 19:04:58 2015 - [info]   m1(192.168.75.11:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:04:58 2015 - [info]     Replicating from 192.168.75.12(192.168.75.12:3306)
Tue Mar 31 19:04:58 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Tue Mar 31 19:04:58 2015 - [info]   s(192.168.75.13:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:04:58 2015 - [info]     Replicating from 192.168.75.12(192.168.75.12:3306)
Tue Mar 31 19:04:58 2015 - [info]     Not candidate for the new Master (no_master is set)
Tue Mar 31 19:04:58 2015 - [info] Executing FLUSH NO_WRITE_TO_BINLOG TABLES. This may take long time..
Tue Mar 31 19:04:58 2015 - [error][/usr/share/perl5/vendor_perl/MHA/Server.pm, ln606]  Failed! Access denied; you need the RELOAD privilege for this operation
Tue Mar 31 19:04:58 2015 - [info] Checking MHA is not monitoring or doing failover..
Tue Mar 31 19:04:58 2015 - [info] Checking replication health on m1..
Tue Mar 31 19:04:58 2015 - [info]  ok.
Tue Mar 31 19:04:58 2015 - [info] Checking replication health on s..
Tue Mar 31 19:04:58 2015 - [info]  ok.
Tue Mar 31 19:04:58 2015 - [info] m1 can be new master.
Tue Mar 31 19:04:58 2015 - [info] 
From:
m2 (current master)
 +--m1
 +--s

To:
m1 (new master)
 +--s
Tue Mar 31 19:04:58 2015 - [info] Checking whether m1(192.168.75.11:3306) is ok for the new master..
Tue Mar 31 19:04:58 2015 - [info]  ok.
Tue Mar 31 19:04:58 2015 - [info] ** Phase 1: Configuration Check Phase completed.
Tue Mar 31 19:04:58 2015 - [info] 
Tue Mar 31 19:04:58 2015 - [info] * Phase 2: Rejecting updates Phase..
Tue Mar 31 19:04:58 2015 - [info] 
Tue Mar 31 19:04:58 2015 - [info] Executing master ip online change script to disable write on the current master:
Tue Mar 31 19:04:58 2015 - [info]   /home/mysql/mha/script/master_ip_online_change --command=stop --orig_master_host=m2 --orig_master_ip=192.168.75.12 --orig_master_port=3306 --new_master_host=m1 --new_master_ip=192.168.75.11 --new_master_port=3306  
Tue Mar 31 19:04:58 2015 948119 Set read_only on the new master.. ok.
Tue Mar 31 19:04:58 2015 953302 Drpping app user on the orig master..
Tue Mar 31 19:04:58 2015 954113 Set read_only=1 on the orig master.. ok.
Tue Mar 31 19:04:58 2015 957489 Killing all application threads..
Tue Mar 31 19:04:58 2015 958115 done.
Tue Mar 31 19:04:58 2015 - [info]  ok.
Tue Mar 31 19:04:58 2015 - [info] Locking all tables on the orig master to reject updates from everybody (including root):
Tue Mar 31 19:04:58 2015 - [info] Executing FLUSH TABLES WITH READ LOCK..
Tue Mar 31 19:04:58 2015 - [error][/usr/share/perl5/vendor_perl/MHA/Server.pm, ln621]  Failed! Access denied; you need the RELOAD privilege for this operation
Tue Mar 31 19:04:58 2015 - [info] Checking binlog writes are stopped or not..
Tue Mar 31 19:04:59 2015 - [info]  ok.
Tue Mar 31 19:04:59 2015 - [info] Orig master binlog:pos is mysql-bin.000015:106.
Tue Mar 31 19:04:59 2015 - [info]  Waiting to execute all relay logs on m1(192.168.75.11:3306)..
Tue Mar 31 19:04:59 2015 - [info]  master_pos_wait(mysql-bin.000015:106) completed on m1(192.168.75.11:3306). Executed 0 events.
Tue Mar 31 19:04:59 2015 - [info]   done.
Tue Mar 31 19:04:59 2015 - [info] Getting new master's binlog name and position..
Tue Mar 31 19:04:59 2015 - [info]  mysql-bin.000023:106
Tue Mar 31 19:04:59 2015 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='m1 or 192.168.75.11', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000023', MASTER_LOG_POS=106, MASTER_USER='repl', MASTER_PASSWORD='xxx';
Tue Mar 31 19:04:59 2015 - [info] Executing master ip online change script to allow write on the new master:
Tue Mar 31 19:04:59 2015 - [info]   /home/mysql/mha/script/master_ip_online_change --command=start --orig_master_host=m2 --orig_master_ip=192.168.75.12 --orig_master_port=3306 --new_master_host=m1 --new_master_ip=192.168.75.11 --new_master_port=3306  
Tue Mar 31 19:05:00 2015 094952 Set read_only=0 on the new master.
Tue Mar 31 19:05:00 2015 096016 Creating app user on the new master..
Connection to 192.168.75.12 closed.
Tue Mar 31 19:05:01 2015 - [info]  ok.
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info] * Switching slaves in parallel..
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info] -- Slave switch on host s(192.168.75.13:3306) started, pid: 4109
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info] Log messages from s ...
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info]  Waiting to execute all relay logs on s(192.168.75.13:3306)..
Tue Mar 31 19:05:01 2015 - [info]  master_pos_wait(mysql-bin.000015:106) completed on s(192.168.75.13:3306). Executed 0 events.
Tue Mar 31 19:05:01 2015 - [info]   done.
Tue Mar 31 19:05:01 2015 - [info]  Resetting slave s(192.168.75.13:3306) and starting replication from the new master m1(192.168.75.11:3306)..
Tue Mar 31 19:05:01 2015 - [info]  Executed CHANGE MASTER.
Tue Mar 31 19:05:01 2015 - [info]  Slave started.
Tue Mar 31 19:05:01 2015 - [info] End of log messages from s ...
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info] -- Slave switch on host s(192.168.75.13:3306) succeeded.
Tue Mar 31 19:05:01 2015 - [info] Unlocking all tables on the orig master:
Tue Mar 31 19:05:01 2015 - [info] Executing UNLOCK TABLES..
Tue Mar 31 19:05:01 2015 - [info]  ok.
Tue Mar 31 19:05:01 2015 - [info] All new slave servers switched successfully.
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info] * Phase 5: New master cleanup phease..
Tue Mar 31 19:05:01 2015 - [info] 
Tue Mar 31 19:05:01 2015 - [info]  m1: Resetting slave info succeeded.
Tue Mar 31 19:05:01 2015 - [info] Switching master to m1(192.168.75.11:3306) completed successfully.
[mysql@s script]$ 
{% endhighlight %}

切换成功

检查发现s已经自动指向了新的master m1

{% highlight bash %}
[mysql@s script]$ F_rs.bash 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.11
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000023
          Read_Master_Log_Pos: 106
               Relay_Log_File: s-relay-bin.000002
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000023
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 106
              Relay_Log_Space: 402
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000005 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
[mysql@s script]$ 
{% endhighlight %}



m2的ip已经没有了

{% highlight bash %}
[mysql@m2 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:c7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.12/24 brd 192.168.75.255 scope global eth5
    inet6 fe80::20c:29ff:fed8:a1c7/64 scope link 
       valid_lft forever preferred_lft forever
3: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:d1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.94/24 brd 192.168.1.255 scope global eth6
    inet6 fe80::20c:29ff:fed8:a1d1/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m2 ~]$ ps -ef | grep keep 
mysql     3644  3097  0 19:10 pts/0    00:00:00 grep keep
[mysql@m2 ~]$
{% endhighlight %}

m1获得ip

{% highlight bash %}
[mysql@m1 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.11/24 brd 192.168.75.255 scope global eth4
    inet 192.168.66.66/24 scope global eth4
    inet6 fe80::20c:29ff:fe25:abec/64 scope link 
       valid_lft forever preferred_lft forever
3: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.115/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe25:abf6/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m1 ~]$ 
{% endhighlight %}



---

##故障转移测试 

使用**masterha_manager**进行故障转移测试 


####迁移前检查

>**Note :** 要将候选master的keepalived优先级降低，然后启动，以免夺走master ip

on m1

{% highlight bash %}
[mysql@m1 ~]$ ps -ef | grep keep 
root     12605     1  0 18:50 ?        00:00:00 /usr/sbin/keepalived -D
root     12606 12605  0 18:50 ?        00:00:00 /usr/sbin/keepalived -D
root     12607 12605  0 18:50 ?        00:00:00 /usr/sbin/keepalived -D
mysql    14615 12622  0 19:22 pts/2    00:00:00 grep keep
[mysql@m1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.11/24 brd 192.168.75.255 scope global eth4
    inet 192.168.66.66/24 scope global eth4
    inet6 fe80::20c:29ff:fe25:abec/64 scope link 
       valid_lft forever preferred_lft forever
3: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.115/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe25:abf6/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m1 ~]$ F_rs.bash 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000023 |      106 |              |                  |
+------------------+----------+--------------+------------------+
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
[mysql@m1 ~]$ 
{% endhighlight %}

on m2

{% highlight bash %}
[mysql@m2 ~]$ ps -ef | grep keep 
root      3751     1  0 19:21 ?        00:00:00 /usr/sbin/keepalived -D
root      3752  3751  0 19:21 ?        00:00:00 /usr/sbin/keepalived -D
root      3753  3751  0 19:21 ?        00:00:00 /usr/sbin/keepalived -D
mysql     3766  3097  0 19:21 pts/0    00:00:00 grep keep
[mysql@m2 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:c7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.12/24 brd 192.168.75.255 scope global eth5
    inet6 fe80::20c:29ff:fed8:a1c7/64 scope link 
       valid_lft forever preferred_lft forever
3: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:d1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.94/24 brd 192.168.1.255 scope global eth6
    inet6 fe80::20c:29ff:fed8:a1d1/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m2 ~]$ F_rs.bash 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.11
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000023
          Read_Master_Log_Pos: 106
               Relay_Log_File: m2-relay-bin.000002
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000023
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 106
              Relay_Log_Space: 403
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000015 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
[mysql@m2 ~]$ 
{% endhighlight %}

on s

{% highlight bash %}
[mysql@s ~]$ F_rs.bash 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.11
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000023
          Read_Master_Log_Pos: 106
               Relay_Log_File: s-relay-bin.000002
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000023
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 106
              Relay_Log_Space: 402
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000005 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
[mysql@s ~]$ masterha_check_repl --conf=/etc/app1.cnf 
Tue Mar 31 19:24:17 2015 - [info] Reading default configuratoins from /etc/masterha_default.cnf..
Tue Mar 31 19:24:17 2015 - [info] Reading application default configurations from /etc/app1.cnf..
Tue Mar 31 19:24:17 2015 - [info] Reading server configurations from /etc/app1.cnf..
Tue Mar 31 19:24:17 2015 - [info] MHA::MasterMonitor version 0.53.
Tue Mar 31 19:24:17 2015 - [info] Dead Servers:
Tue Mar 31 19:24:17 2015 - [info] Alive Servers:
Tue Mar 31 19:24:17 2015 - [info]   m1(192.168.75.11:3306)
Tue Mar 31 19:24:17 2015 - [info]   m2(192.168.75.12:3306)
Tue Mar 31 19:24:17 2015 - [info]   s(192.168.75.13:3306)
Tue Mar 31 19:24:17 2015 - [info] Alive Slaves:
Tue Mar 31 19:24:17 2015 - [info]   m2(192.168.75.12:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:24:17 2015 - [info]     Replicating from 192.168.75.11(192.168.75.11:3306)
Tue Mar 31 19:24:17 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Tue Mar 31 19:24:17 2015 - [info]   s(192.168.75.13:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:24:17 2015 - [info]     Replicating from 192.168.75.11(192.168.75.11:3306)
Tue Mar 31 19:24:17 2015 - [info]     Not candidate for the new Master (no_master is set)
Tue Mar 31 19:24:17 2015 - [info] Current Alive Master: m1(192.168.75.11:3306)
Tue Mar 31 19:24:17 2015 - [info] Checking slave configurations..
Tue Mar 31 19:24:17 2015 - [info] Checking replication filtering settings..
Tue Mar 31 19:24:17 2015 - [info]  binlog_do_db= , binlog_ignore_db= 
Tue Mar 31 19:24:17 2015 - [info]  Replication filtering check ok.
Tue Mar 31 19:24:17 2015 - [info] Starting SSH connection tests..
Tue Mar 31 19:24:22 2015 - [info] All SSH connection tests passed successfully.
Tue Mar 31 19:24:22 2015 - [info] Checking MHA Node version..
Tue Mar 31 19:24:24 2015 - [info]  Version check ok.
Tue Mar 31 19:24:24 2015 - [info] Checking SSH publickey authentication settings on the current master..
Tue Mar 31 19:24:25 2015 - [info] HealthCheck: SSH to m1 is reachable.
Tue Mar 31 19:24:26 2015 - [info] Master MHA Node version is 0.53.
Tue Mar 31 19:24:26 2015 - [info] Checking recovery script configurations on the current master..
Tue Mar 31 19:24:26 2015 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/home/mysql/mha/save_binary_logs_test --manager_version=0.53 --start_file=mysql-bin.000023 
Tue Mar 31 19:24:26 2015 - [info]   Connecting to mysql@m1(m1).. 
  Creating /home/mysql/mha if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin.000023
Tue Mar 31 19:24:27 2015 - [info] Master setting check done.
Tue Mar 31 19:24:27 2015 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Tue Mar 31 19:24:27 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=m2 --slave_ip=192.168.75.12 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Tue Mar 31 19:24:27 2015 - [info]   Connecting to mysql@192.168.75.12(m2:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to m2-relay-bin.000002
    Temporary relay log file is /var/lib/mysql/m2-relay-bin.000002
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Tue Mar 31 19:24:28 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=s --slave_ip=192.168.75.13 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Tue Mar 31 19:24:28 2015 - [info]   Connecting to mysql@192.168.75.13(s:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to s-relay-bin.000002
    Temporary relay log file is /var/lib/mysql/s-relay-bin.000002
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Tue Mar 31 19:24:28 2015 - [info] Slaves settings check done.
Tue Mar 31 19:24:28 2015 - [info] 
m1 (current master)
 +--m2
 +--s

Tue Mar 31 19:24:28 2015 - [info] Checking replication health on m2..
Tue Mar 31 19:24:28 2015 - [info]  ok.
Tue Mar 31 19:24:28 2015 - [info] Checking replication health on s..
Tue Mar 31 19:24:28 2015 - [info]  ok.
Tue Mar 31 19:24:28 2015 - [info] Checking master_ip_failover_script status:
Tue Mar 31 19:24:28 2015 - [info]   /home/mysql/mha/script/master_ip_failover --command=status --ssh_user=mysql --orig_master_host=m1 --orig_master_ip=192.168.75.11 --orig_master_port=3306 
Tue Mar 31 19:24:29 2015 - [info]  OK.
Tue Mar 31 19:24:29 2015 - [warning] shutdown_script is not defined.
Tue Mar 31 19:24:29 2015 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.
[mysql@s ~]$ 
{% endhighlight %}


在s上启动后台监控，监视master的健康状态

{% highlight bash %}
[mysql@s ~]$ nohup   masterha_manager --conf=/etc/app1.cnf  --ignore_last_failover  & 
[1] 4446
[mysql@s ~]$ nohup: ignoring input and appending output to `nohup.out'

[mysql@s ~]$ 
{% endhighlight %}

> **Note:**  **--ignore_last_failover** 一定要加 ，否则即便系统状态是正常的也不能切换，因为mha的机制是发现8小时内有切换，就不会再次切换，这个机制是为避免新的master短时间内再次切换的一种保护措施 ，也可以不使用这个参数 而将**~/mha/app1/app1.failover.complete**这个文件手动删除


> **Tip:** 可以使用**~/mha/app1/manager.log**这个日志文件来监视状态，也可以获知当前做了什么事

{% highlight bash %}
[mysql@s ~]$ cat  mha/app1/manager.log
Tue Mar 31 19:27:56 2015 - [info] MHA::MasterMonitor version 0.53.
Tue Mar 31 19:27:56 2015 - [info] Dead Servers:
Tue Mar 31 19:27:56 2015 - [info] Alive Servers:
Tue Mar 31 19:27:56 2015 - [info]   m1(192.168.75.11:3306)
Tue Mar 31 19:27:56 2015 - [info]   m2(192.168.75.12:3306)
Tue Mar 31 19:27:56 2015 - [info]   s(192.168.75.13:3306)
Tue Mar 31 19:27:56 2015 - [info] Alive Slaves:
Tue Mar 31 19:27:56 2015 - [info]   m2(192.168.75.12:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:27:56 2015 - [info]     Replicating from 192.168.75.11(192.168.75.11:3306)
Tue Mar 31 19:27:56 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Tue Mar 31 19:27:56 2015 - [info]   s(192.168.75.13:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Tue Mar 31 19:27:56 2015 - [info]     Replicating from 192.168.75.11(192.168.75.11:3306)
Tue Mar 31 19:27:56 2015 - [info]     Not candidate for the new Master (no_master is set)
Tue Mar 31 19:27:56 2015 - [info] Current Alive Master: m1(192.168.75.11:3306)
Tue Mar 31 19:27:56 2015 - [info] Checking slave configurations..
Tue Mar 31 19:27:56 2015 - [info] Checking replication filtering settings..
Tue Mar 31 19:27:56 2015 - [info]  binlog_do_db= , binlog_ignore_db= 
Tue Mar 31 19:27:56 2015 - [info]  Replication filtering check ok.
Tue Mar 31 19:27:56 2015 - [info] Starting SSH connection tests..
Tue Mar 31 19:28:01 2015 - [info] All SSH connection tests passed successfully.
Tue Mar 31 19:28:01 2015 - [info] Checking MHA Node version..
Tue Mar 31 19:28:02 2015 - [info]  Version check ok.
Tue Mar 31 19:28:02 2015 - [info] Checking SSH publickey authentication settings on the current master..
Tue Mar 31 19:28:03 2015 - [info] HealthCheck: SSH to m1 is reachable.
Tue Mar 31 19:28:04 2015 - [info] Master MHA Node version is 0.53.
Tue Mar 31 19:28:04 2015 - [info] Checking recovery script configurations on the current master..
Tue Mar 31 19:28:04 2015 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/home/mysql/mha/save_binary_logs_test --manager_version=0.53 --start_file=mysql-bin.000023 
Tue Mar 31 19:28:04 2015 - [info]   Connecting to mysql@m1(m1).. 
  Creating /home/mysql/mha if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin.000023
Tue Mar 31 19:28:05 2015 - [info] Master setting check done.
Tue Mar 31 19:28:05 2015 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Tue Mar 31 19:28:05 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=m2 --slave_ip=192.168.75.12 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Tue Mar 31 19:28:05 2015 - [info]   Connecting to mysql@192.168.75.12(m2:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to m2-relay-bin.000002
    Temporary relay log file is /var/lib/mysql/m2-relay-bin.000002
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Tue Mar 31 19:28:06 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=s --slave_ip=192.168.75.13 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Tue Mar 31 19:28:06 2015 - [info]   Connecting to mysql@192.168.75.13(s:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to s-relay-bin.000002
    Temporary relay log file is /var/lib/mysql/s-relay-bin.000002
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Tue Mar 31 19:28:07 2015 - [info] Slaves settings check done.
Tue Mar 31 19:28:07 2015 - [info] 
m1 (current master)
 +--m2
 +--s

Tue Mar 31 19:28:07 2015 - [info] Checking master_ip_failover_script status:
Tue Mar 31 19:28:07 2015 - [info]   /home/mysql/mha/script/master_ip_failover --command=status --ssh_user=mysql --orig_master_host=m1 --orig_master_ip=192.168.75.11 --orig_master_port=3306 
Tue Mar 31 19:28:07 2015 - [info]  OK.
Tue Mar 31 19:28:07 2015 - [warning] shutdown_script is not defined.
Tue Mar 31 19:28:07 2015 - [info] Set master ping interval 3 seconds.
Tue Mar 31 19:28:07 2015 - [warning] secondary_check_script is not defined. It is highly recommended setting it to check master reachability from two or more routes.
Tue Mar 31 19:28:07 2015 - [info] Starting ping health check on m1(192.168.75.11:3306)..
Tue Mar 31 19:28:07 2015 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
[mysql@s ~]$ 
{% endhighlight %}


####停库切换

停止m1上的mysql，模拟一次数据库宕机

{% highlight bash %}
[root@m1 log]# /etc/init.d/mysql  stop 
Shutting down MySQL...                                     [  OK  ]
[root@m1 log]# 
{% endhighlight %}

发现m1的ip已经移除

{% highlight bash %}
[root@m1 log]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.11/24 brd 192.168.75.255 scope global eth4
    inet6 fe80::20c:29ff:fe25:abec/64 scope link 
       valid_lft forever preferred_lft forever
3: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.115/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe25:abf6/64 scope link 
       valid_lft forever preferred_lft forever
[root@m1 log]# ps -ef | grep keep 
root     16448  2438  0 19:47 pts/0    00:00:00 grep keep
[root@m1 log]#
{% endhighlight %}


ip已经飘移到了m2上

{% highlight bash %}
[mysql@m2 ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:c7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.12/24 brd 192.168.75.255 scope global eth5
    inet 192.168.66.66/24 scope global eth5
    inet6 fe80::20c:29ff:fed8:a1c7/64 scope link 
       valid_lft forever preferred_lft forever
3: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d8:a1:d1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.94/24 brd 192.168.1.255 scope global eth6
    inet6 fe80::20c:29ff:fed8:a1d1/64 scope link 
       valid_lft forever preferred_lft forever
[mysql@m2 ~]$ F_rs.bash 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000015 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | ON    |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | OFF   |
+---------------+-------+
[mysql@m2 ~]$ 
{% endhighlight %}

s也自动指向了新的master 

{% highlight bash %}
[mysql@s ~]$ F_rs.bash 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.12
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000015
          Read_Master_Log_Pos: 106
               Relay_Log_File: s-relay-bin.000002
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000015
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 106
              Relay_Log_Space: 402
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000005 |      106 |              |                  |
+------------------+----------+--------------+------------------+
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| relay_log_purge | OFF   |
+-----------------+-------+
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
[mysql@s ~]$ 
{% endhighlight %}

这是从日志中截取的failover 报告

{% highlight bash %}
----- Failover Report -----

app1: MySQL Master failover m1 to m2 succeeded

Master m1 is down!

Check MHA Manager logs at s:/home/mysql/mha/app1/manager.log for details.

Started automated(non-interactive) failover.
Invalidated master IP address on m1.
The latest slave m2(192.168.75.12:3306) has all relay logs for recovery.
Selected m2 as a new master.
m2: OK: Applying all logs succeeded.
m2: OK: Activated master IP address.
s: This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
s: OK: Applying all logs succeeded. Slave started, replicating from m2.
m2: Resetting slave info succeeded.
Master failover to m2(192.168.75.12:3306) completed successfully.
{% endhighlight %}

致此mha的两种failover模式已经通过检测，事实上生产环境非常复杂，在具体环境中还要具体分析，比如流量大到一定程度时，迁移如何，不断有大量新的数据正在写入时，迁移又如何，还得进行更充分的测试

在此只是将mha的基本框架进行了搭建，基本功能进行了展示


[mha3]:http://soft.dog/2015/03/30/mysql-HA-build-keepalived-install-and-config.html



