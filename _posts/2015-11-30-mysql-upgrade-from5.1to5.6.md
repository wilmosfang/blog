---
layout: post
title: mysql 5.1升级到5.6
author: wilmosfang
tags:  mysql upgrade
categories:  mysql upgrade
wc: 1238 5231 59037
excerpt: follow me
comments: true
---


---

# 前言


优秀的开源软件，由于社区的力量，会经常更新，下面分享一下mysql升级的基本操作

> **Tip:** 我使用的是 **[percona][percona]** 版的mysql，当前最新版本为 **Percona-Server-server-56-5.6.27** ，社区版最新为 **mysql-community-server-5.7.9** , 升级方法是一样的


---


# 概要

* TOC
{:toc}



---


## 备份
	

### 挂载备份目录

挂载目标服务器(就是要创建成slave的mysql服务器)NFS

> **Tip:** 以方便存放备份文件，这样省去了一次单独的网络间拷贝，还可以方便的给其它服务器使用，用来创建更多slave

~~~
[root@upgrade-master ~]# mount -t nfs -o intr upgrade-slave:/data/nfs /mnt/mysqlbak/
[root@upgrade-master ~]# df -h 
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda5              95G   22G   69G  25% /
/dev/sda6             429G  214G  193G  53% /opt
/dev/sda3             284G  102G  168G  38% /var
/dev/sda1             494M   36M  434M   8% /boot
tmpfs                  24G     0   24G   0% /dev/shm
upgrade-slave:/data/nfs
                      1.7T   73M  1.7T   1% /mnt/mysqlbak
[root@upgrade-master ~]#
~~~


---

### 停止集群

如果有集群如mha，要停止集群，因为备份结束时会产生一个长锁，集群软件侦测到一定时间里没响应会认为master发生了故障，从而触发迁移，这不是我们想看到的~因为这会导致新恢复出来的数据不一致

所以有集群的情况下，最好停止集群

> **Tip:** 如果是mha，可以参考以下命令

~~~
masterha_check_status --conf=/etc/app1.cnf
masterha_stop  --conf=/etc/app1.cnf
~~~

---

### 开始备份


~~~
[root@upgrade-master ~]# time nohup /usr/bin/innobackupex --defaults-file=/etc/my.cnf  --user=root --password=xxxxxxxx  /mnt/mysqlbak/masterdb.full.backup  >> /tmp/masterdb.full.backup.log  2>&1   & 
[1] 890
[root@upgrade-master ~]# 
~~~

---

## 下载两种版本mysql

分别下载同版本mysql，和目标版本(高版本)mysql

~~~
[root@upgrade-slave src]# yumdownloader  Percona-Server-client-51.x86_64   Percona-Server-devel-51.x86_64 Percona-Server-server-51.x86_64  Percona-Server-shared-51.x86_64  ; yumdownloader Percona-Server-client-56.x86_64 Percona-Server-devel-56.x86_64  Percona-Server-server-56.x86_64  Percona-Server-shared-56.x86_64  
Loaded plugins: fastestmirror, refresh-packagekit
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * extras: mirrors.opencas.cn
 * updates: mirror.bit.edu.cn
Percona-Server-client-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm                                     | 920 kB     00:18     
Percona-Server-devel-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm                                      | 6.3 MB     01:01     
Percona-Server-server-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm                                     |  12 MB     02:16     
Percona-Server-shared-51-5.1.73-rel14.12.625.rhel6.x86_64.rpm                                     | 2.1 MB     00:12     
Loaded plugins: fastestmirror, refresh-packagekit
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * extras: mirrors.opencas.cn
 * updates: mirror.bit.edu.cn
Percona-Server-client-56-5.6.27-rel75.0.el6.x86_64.rpm                                            | 6.4 MB     01:27     
Percona-Server-devel-56-5.6.27-rel75.0.el6.x86_64.rpm                                             | 1.0 MB     00:06     
Percona-Server-server-56-5.6.27-rel75.0.el6.x86_64.rpm                                            |  20 MB     03:51     
Percona-Server-shared-56-5.6.27-rel75.0.el6.x86_64.rpm                                            | 725 kB     00:16     
[root@upgrade-slave src]# echo $?
0
[root@upgrade-slave src]#   
~~~
	

> **Tip:** 可以对卷进行快照，以方便恢复，但是要注意的是:
>
> * 快照大小一定要大于变化数据的总量，否则会被撑爆，
> * 一般根据需求创建，在自己觉得有危险的操作之前(有可能对数据造成不可逆改变的地方)进行创建
> * 快照得在同一个卷组中，跨卷组无法创建，所以也要保证当前卷组中有足够空余空间
> * 快照使用完一般都是立即删除，因为cow的机制会给io带来额外开销，删除快照使用 **lvremove**
> * 创建快照的方法如下

~~~
[root@upgrade-slave ~]# lvs
  LV      VG             Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data vg_test_for_mysqlupgrade -wi-ao----   1.73t                                                    
  lv_home vg_test_for_mysqlupgrade -wi-ao----  50.00g                                                    
  lv_root vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                    
  lv_swap vg_test_for_mysqlupgrade -wi-ao----  16.00g                                                    
  lv_var  vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                    
[root@upgrade-slave ~]# lvcreate  -L 150G -s -n lv_s_data /dev/vg_test_for_mysqlupgrade/lv_data  
  Logical volume "lv_s_data" created.
[root@upgrade-slave ~]# lvs
  LV        VG             Attr       LSize   Pool Origin  Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data   vg_test_for_mysqlupgrade owi-aos---   1.73t                                                     
  lv_home   vg_test_for_mysqlupgrade -wi-ao----  50.00g                                                     
  lv_root   vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                     
  lv_s_data vg_test_for_mysqlupgrade swi-a-s--- 150.00g      lv_data 0.00                                   
  lv_swap   vg_test_for_mysqlupgrade -wi-ao----  16.00g                                                     
  lv_var    vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                     
[root@upgrade-slave ~]# 
~~~

---

## 安装同版本mysql

安装与原数据库相同版本的mysql

~~~
[root@upgrade-slave src]# rpm -ivh  Percona-Server-server-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm  Percona-Server-client-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm 
Preparing...                ########################################### [100%]
   1:Percona-Server-client-5########################################### [ 50%]
   2:Percona-Server-server-5########################################### [100%]
151126 15:54:42 [ERROR] /usr/sbin/mysqld: unknown option '--myisam_recover_options'
151126 15:54:42 [ERROR] Aborting

151126 15:54:42 [Note] /usr/sbin/mysqld: Shutdown complete


Installation of system tables failed!  Examine the logs in
/var/lib/mysql for more information.

You can try to start the mysqld daemon with:

    shell> /usr/sbin/mysqld --skip-grant &

and use the command line tool /usr/bin/mysql
to connect to the mysql database and look at the grant tables:

    shell> /usr/bin/mysql -u root mysql
    mysql> show tables

Try 'mysqld --help' if you have problems with paths.  Using --log
gives you a log in /var/lib/mysql that may be helpful.

Please consult the MySQL manual section
'Problems running mysql_install_db', and the manual section that
describes problems on your OS.  Another information source are the
MySQL email archives available at http://lists.mysql.com/.

Please check all of the above before mailing us!  And remember, if
you do mail us, you MUST use the /usr/bin/mysqlbug script!

Starting MySQL (Percona Server).........................Manager of pid-file quit without updating file.[FAILED]
Giving mysqld 2 seconds to start
Percona Server is distributed with several useful UDF (User Defined Function) from Percona Toolkit.
Run the following commands to create these functions:
mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"
mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"
mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"
See http://www.percona.com/doc/percona-server/5.1/management/udf_percona_toolkit.html for more details
[root@upgrade-slave src]# echo $?
0
[root@upgrade-slave src]# 
~~~

---

## 备份并清空数据目录

备份并清空 **/var/lib/mysql**  (也就是mysql的数据目录)，不清空在之后的恢复过程中会报错

~~~
[root@upgrade-slave data]# cp -r mysql mysql.20151126  
[root@upgrade-slave data]# ls
benchmark  lost+found  mysql  mysql.20151125  mysql.20151126  mysql.bak  nfs  redis
[root@upgrade-slave data]# cd /var/lib/mysql
[root@upgrade-slave mysql]# ls
upgrade-slave.err  ibdata1  ib_logfile0  ib_logfile1  ib_logfile2  mysql  mysql-bin.index  test
[root@upgrade-slave mysql]# rm -rf *
[root@upgrade-slave mysql]# ls
[root@upgrade-slave mysql]# 
~~~

---

## 修改配置文件

将原来的配置文件进行局部修改，主要为以下几点

其目的是为了适应新的主机环境，并且避免与master的server-id冲突

~~~
[root@upgrade-slave etc]# diff /tmp/new.mysql.cnf /tmp/old.mysql.cnf 
27d26
< relay-log=relay-bin
30c29
< slow_query_log_file = upgrade-slave-slow.log
---
> slow_query_log_file = upgrade-master-slow.log
32c31
< server-id = 10
---
> server-id = 3
[root@upgrade-slave etc]#
~~~

> **Tip:**  根据具体情况，有时 **tmpdir** 也要根据环境进行设置，修改完成后，最好进行再次确认，合适的配置可以减少errlog里的报错，和重新调试的时间


---

## 备份完成

通过观察 **masterdb.full.backup.log** 可以知道备份是否完成

通常备份完成会产生 **innobackupex: completed OK!** 的输出

---

## 恢复集群

是时候恢复集群了，让集群重新担当起失效检查和故障转移的职责

> **Tip:** 如果是mha，可以参考以下命令

~~~
nohup  masterha_manager --conf=/etc/app1.cnf  --ignore_last_failover  &
~~~


---

## 安装xtrabackup


~~~
[root@upgrade-slave percona.51]# rpm -ivh  xtrabackup-1.6.7-356.rhel6.x86_64.rpm 
warning: xtrabackup-1.6.7-356.rhel6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:xtrabackup             ########################################### [100%]
[root@upgrade-slave percona.51]# 
~~~

尽量安装版本相近的xtrabackup,可以有效避免一些兼容性问题

---

## 应用日志(恢复准备)

准备恢复

~~~
[root@upgrade-slave nfs]# time nohup  innobackupex --apply-log  masterdb.full.backup/2015-11-25_21-25-02/
nohup: ignoring input and appending output to `nohup.out'

real	2m49.336s
user	0m18.968s
sys	0m5.878s
[root@upgrade-slave nfs]# 
~~~

---

## 恢复数据

~~~
[root@upgrade-slave nfs]# time nohup  innobackupex  --copy-back  masterdb.full.backup/2015-11-25_21-25-02/  >> /tmp/fullcopyback.log  2>&1   & 
[1] 117341
[root@upgrade-slave nfs]# 
[root@upgrade-slave nfs]# tail -f /tmp/fullcopyback.log   
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/mysql/general_log.CSV' to '/var/lib/mysql/mysql/general_log.CSV'
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/mysql/func.MYI' to '/var/lib/mysql/mysql/func.MYI'
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/mysql/time_zone_leap_second.MYD' to '/var/lib/mysql/mysql/time_zone_leap_second.MYD'
innobackupex: Creating directory '/var/lib/mysql/key_db'
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/key_db/user_connections.ibd' to '/var/lib/mysql/key_db/user_connections.ibd'
...
...
innobackupex: Starting to copy InnoDB log files
innobackupex: in '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02'
innobackupex: back to original InnoDB log directory '/var/lib/mysql'
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/ib_logfile1' to '/var/lib/mysql/ib_logfile1'
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/ib_logfile2' to '/var/lib/mysql/ib_logfile2'
innobackupex: Copying '/data/nfs/masterdb.full.backup/2015-11-25_21-25-02/ib_logfile0' to '/var/lib/mysql/ib_logfile0'
innobackupex: Finished copying back files.

151126 20:41:40  innobackupex: completed OK!

real	33m3.933s
user	0m0.963s
sys	4m54.254s


[1]+  Done                    time nohup innobackupex --copy-back masterdb.full.backup/2015-11-25_21-25-02/ >> /tmp/fullcopyback.log 2>&1
[root@upgrade-slave nfs]#
~~~

---

## 修改权限

~~~
[root@upgrade-slave ~]# cd /var/lib/mysql/
[root@upgrade-slave mysql]# ls
huodongdb  something_more  ib_logfile0  istuffdb    mysql        feeddb
historydb   stuffdb      ib_logfile1  caldb    mytempdb      test
testdb    ibdata1      ib_logfile2  mupgradetest_pd  key_db  xtrabackup_binlog_pos_innodb
[root@upgrade-slave mysql]# ll 
total 5916788
drwxr-xr-x. 2 root root       4096 Nov 26 20:10 huodongdb
drwxr-xr-x. 2 root root       4096 Nov 26 20:12 historydb
drwxr-xr-x. 2 root root      53248 Nov 26 20:41 testdb
drwxr-xr-x. 2 root root       4096 Nov 26 20:18 something_more
drwxr-xr-x. 2 root root       4096 Nov 26 20:11 stuffdb
-rw-r--r--. 1 root root 5253365760 Nov 26 20:41 ibdata1
-rw-r--r--. 1 root root  268435456 Nov 26 20:41 ib_logfile0
-rw-r--r--. 1 root root  268435456 Nov 26 20:41 ib_logfile1
-rw-r--r--. 1 root root  268435456 Nov 26 20:41 ib_logfile2
drwxr-xr-x. 2 root root       4096 Nov 26 20:11 istuffdb
drwxr-xr-x. 2 root root      12288 Nov 26 20:18 caldb
drwxr-xr-x. 2 root root       4096 Nov 26 20:18 mupgradetest_pd
drwxr-xr-x. 2 root root       4096 Nov 26 20:08 mysql
drwxr-xr-x. 2 root root       4096 Nov 26 20:10 mytempdb
drwxr-xr-x. 2 root root       4096 Nov 26 20:10 key_db
drwxr-xr-x. 2 root root       4096 Nov 26 20:11 feeddb
drwxr-xr-x. 2 root root       4096 Nov 26 20:11 test
-rw-r--r--. 1 root root         29 Nov 26 20:08 xtrabackup_binlog_pos_innodb
[root@upgrade-slave mysql]# cd ..
[root@upgrade-slave lib]# chown  -R mysql.mysql /var/lib/mysql/
[root@upgrade-slave lib]# ll /var/lib/mysql/
total 5916788
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:10 huodongdb
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:12 historydb
drwxr-xr-x. 2 mysql mysql      53248 Nov 26 20:41 testdb
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:18 something_more
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:11 stuffdb
-rw-r--r--. 1 mysql mysql 5253365760 Nov 26 20:41 ibdata1
-rw-r--r--. 1 mysql mysql  268435456 Nov 26 20:41 ib_logfile0
-rw-r--r--. 1 mysql mysql  268435456 Nov 26 20:41 ib_logfile1
-rw-r--r--. 1 mysql mysql  268435456 Nov 26 20:41 ib_logfile2
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:11 istuffdb
drwxr-xr-x. 2 mysql mysql      12288 Nov 26 20:18 caldb
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:18 mupgradetest_pd
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:08 mysql
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:10 mytempdb
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:10 key_db
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:11 feeddb
drwxr-xr-x. 2 mysql mysql       4096 Nov 26 20:11 test
-rw-r--r--. 1 mysql mysql         29 Nov 26 20:08 xtrabackup_binlog_pos_innodb
[root@upgrade-slave lib]#
~~~

---

## 启动服务

~~~
[root@upgrade-slave lib]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server)..............              [  OK  ]
[root@upgrade-slave lib]# mysql -u root -p 
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[root@upgrade-slave lib]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.1.73-14.12-log Percona Server (GPL), Release 14.12, Revision 624

Copyright (c) 2009-2013 Percona LLC and/or its affiliates
Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| huodongdb          |
| historydb          |
| testdb             |
| something_more     |
| stuffdb            |
| istuffdb           |
| caldb              |
| mupgradetest_pd    |
| mytempdb           |
| mysql              |
| key_db             |
| feeddb             |
| test               |
+--------------------+
14 rows in set (0.04 sec)

mysql> quit
Bye
[root@upgrade-slave lib]#
[hunter@upgrade-slave ~]$ tail -f /var/lib/mysql/upgrade-slave.err 
tail: cannot open `/var/lib/mysql/upgrade-slave.err' for reading: Permission denied
tail: no files remaining
[hunter@upgrade-slave ~]$ sudo tail -f /var/lib/mysql/upgrade-slave.err 
[sudo] password for hunter: 
InnoDB: The InnoDB memory heap is disabled
InnoDB: Mutexes and rw_locks use GCC atomic builtins
InnoDB: Compressed tables use zlib 1.2.3
151126 20:46:11  InnoDB: Initializing buffer pool, size = 20.0G
151126 20:46:18  InnoDB: Completed initialization of buffer pool
151126 20:46:18  InnoDB: highest supported file format is Barracuda.
...
...
~~~

启动过程中如何有任何问题，可以通过追踪 **/var/lib/mysql/upgrade-slave.err** 文件来获知


---

## 停止mysql

~~~
[root@upgrade-slave mysql]# /etc/init.d/mysql stop 
Shutting down MySQL.                                       [  OK  ]
[root@upgrade-slave mysql]# 
~~~

---

## 创建当前快照

便于恢复到当前状态

~~~
[root@upgrade-slave ~]# df  -h 
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_test_for_mysqlupgrade-lv_root
                       99G  3.4G   90G   4% /
tmpfs                  63G     0   63G   0% /dev/shm
/dev/sda1             477M   62M  390M  14% /boot
/dev/mapper/vg_test_for_mysqlupgrade-lv_data
                      1.7T  927G  726G  57% /data
/dev/mapper/vg_test_for_mysqlupgrade-lv_home
                       50G   53M   47G   1% /home
/dev/mapper/vg_test_for_mysqlupgrade-lv_var
                       99G  302M   94G   1% /var
[root@upgrade-slave ~]# lvs
  LV      VG             Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data vg_test_for_mysqlupgrade -wi-ao----   1.73t                                                    
  lv_home vg_test_for_mysqlupgrade -wi-ao----  50.00g                                                    
  lv_root vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                    
  lv_swap vg_test_for_mysqlupgrade -wi-ao----  16.00g                                                    
  lv_var  vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                    
[root@upgrade-slave ~]# vgs
  VG             #PV #LV #SN Attr   VSize VFree  
  vg_test_for_mysqlupgrade   1   5   0 wz--n- 2.18t 200.00g
[root@upgrade-slave ~]# lvcreate  -L 150G -s -n lv_s_data /dev/vg_test_for_mysqlupgrade/lv_data 
  Logical volume "lv_s_data" created.
[root@upgrade-slave ~]# lvs
  LV        VG             Attr       LSize   Pool Origin  Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data   vg_test_for_mysqlupgrade owi-aos---   1.73t                                                     
  lv_home   vg_test_for_mysqlupgrade -wi-ao----  50.00g                                                     
  lv_root   vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                     
  lv_s_data vg_test_for_mysqlupgrade swi-a-s--- 150.00g      lv_data 0.00                                   
  lv_swap   vg_test_for_mysqlupgrade -wi-ao----  16.00g                                                     
  lv_var    vg_test_for_mysqlupgrade -wi-ao---- 100.00g                                                     
[root@upgrade-slave ~]# 
~~~

---

## 检查mysql状态

~~~
[root@upgrade-slave src]# ps faux | grep mysql 
root      11270  0.0  0.0 103308   824 pts/1    S+   21:22   0:00  |                       \_ grep mysql
root     136775  0.0  0.0 189236  3732 pts/2    S+   20:58   0:00  |           \_ sudo tail -f /var/lib/mysql/upgrade-slave.err
root     136889  0.0  0.0 100944   612 pts/2    S+   20:58   0:00  |               \_ tail -f /var/lib/mysql/upgrade-slave.err
root     137921  0.0  0.0 189236  3736 pts/3    S+   20:59   0:00              \_ sudo tail -f /var/lib/mysql/upgrade-slave.err
root     137993  0.0  0.0 100944   608 pts/3    S+   20:59   0:00                  \_ tail -f /var/lib/mysql/upgrade-slave.err
[root@upgrade-slave src]# /etc/init.d/mysql  status
MySQL is not running                                       [FAILED]
[root@upgrade-slave src]# 
~~~

---

## 升级新版本mysql

其实就是替换成新版本的软件

因为有依赖关系所以要使用 **`--nodeps`** 来忽略掉依赖

~~~
[root@upgrade-slave src]# rpm -e Percona-Server-client-51-5.1.73-rel14.12.624.rhel6.x86_64 Percona-Server-server-51-5.1.73-rel14.12.624.rhel6.x86_64 Percona-Server-shared-51-5.1.73-rel14.12.624.rhel6.x86_64 
error: Failed dependencies:
	libmysqlclient.so.16()(64bit) is needed by (installed) postfix-2:2.6.6-6.el6_5.x86_64
	libmysqlclient.so.16()(64bit) is needed by (installed) perl-DBD-MySQL-4.013-3.el6.x86_64
	libmysqlclient.so.16()(64bit) is needed by (installed) php-mysql-5.3.3-46.el6_6.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) postfix-2:2.6.6-6.el6_5.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) perl-DBD-MySQL-4.013-3.el6.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) php-mysql-5.3.3-46.el6_6.x86_64
	libmysqlclient_r.so.16()(64bit) is needed by (installed) sysbench-0.5-6.el6.x86_64
	libmysqlclient_r.so.16(libmysqlclient_16)(64bit) is needed by (installed) sysbench-0.5-6.el6.x86_64
	mysql-libs is needed by (installed) postfix-2:2.6.6-6.el6_5.x86_64
[root@upgrade-slave src]# rpm -e Percona-Server-client-51-5.1.73-rel14.12.624.rhel6.x86_64 Percona-Server-server-51-5.1.73-rel14.12.624.rhel6.x86_64 Percona-Server-shared-51-5.1.73-rel14.12.624.rhel6.x86_64  --nodeps
[root@upgrade-slave src]# ll | grep 56
-rw-r--r--. 1 root   root    6726712 Nov  3 19:31 Percona-Server-client-56-5.6.27-rel75.0.el6.x86_64.rpm
-rw-r--r--. 1 root   root    6566688 Jul 31  2014 Percona-Server-devel-51-5.1.73-rel14.12.624.rhel6.x86_64.rpm
-rw-r--r--. 1 root   root    1032200 Nov  3 19:31 Percona-Server-devel-56-5.6.27-rel75.0.el6.x86_64.rpm
-rw-r--r--. 1 root   root   20480132 Nov  3 19:31 Percona-Server-server-56-5.6.27-rel75.0.el6.x86_64.rpm
-rw-r--r--. 1 root   root     742712 Nov  3 19:31 Percona-Server-shared-56-5.6.27-rel75.0.el6.x86_64.rpm
[root@upgrade-slave src]# rpm -ivh  Percona-Server-client-56-5.6.27-rel75.0.el6.x86_64.rpm  Percona-Server-server-56-5.6.27-rel75.0.el6.x86_64.rpm Percona-Server-shared-56-5.6.27-rel75.0.el6.x86_64.rpm 
Preparing...                ########################################### [100%]
   1:Percona-Server-shared-5########################################### [ 33%]
   2:Percona-Server-client-5########################################### [ 67%]
   3:Percona-Server-server-5########################################### [100%]
Percona Server is distributed with several useful UDF (User Defined Function) from Percona Toolkit.
Run the following commands to create these functions:
mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"
mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"
mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"
See http://www.percona.com/doc/percona-server/5.6/management/udf_percona_toolkit.html for more details
[root@upgrade-slave src]# echo $?
0
[root@upgrade-slave src]# 
~~~

---

## 尝试启动数据库

~~~
[root@upgrade-slave mysql]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server)............The server quit [FAILED]updating PID file (/var/lib/mysql/upgrade-slave.pid).
[root@upgrade-slave mysql]# 
~~~


**/var/lib/mysql/upgrade-slave.err** 日志报错


~~~
151126 21:27:16 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2015-11-26 21:27:16 0 [Warning] 'THREAD_CONCURRENCY' is deprecated and will be removed in a future release.
2015-11-26 21:27:16 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-11-26 21:27:16 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-75.0-log) starting as process 12493 ...
2015-11-26 21:27:16 12493 [Warning] Using unique option prefix myisam_recover instead of myisam-recover-options is deprecated and will be removed in a future release. Please use the full name instead.
2015-11-26 21:27:16 12493 [Warning] option 'innodb-autoextend-increment': unsigned value 33554432 adjusted to 1000
2015-11-26 21:27:16 12493 [Note] Plugin 'FEDERATED' is disabled.
2015-11-26 21:27:16 7f6ec219f7e0 InnoDB: Warning: Using innodb_additional_mem_pool_size is DEPRECATED. This option may be removed in future releases, together with the option innodb_use_sys_malloc and with the InnoDB's internal memory allocator.
2015-11-26 21:27:16 12493 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-11-26 21:27:16 12493 [Note] InnoDB: The InnoDB memory heap is disabled
2015-11-26 21:27:16 12493 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-11-26 21:27:16 12493 [Note] InnoDB: Memory barrier is not used
2015-11-26 21:27:16 12493 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-11-26 21:27:16 12493 [Note] InnoDB: Using Linux native AIO
2015-11-26 21:27:16 12493 [Note] InnoDB: Using CPU crc32 instructions
2015-11-26 21:27:16 12493 [Note] InnoDB: Initializing buffer pool, size = 20.0G
2015-11-26 21:27:21 12493 [Note] InnoDB: Completed initialization of buffer pool
2015-11-26 21:27:22 12493 [Note] InnoDB: Highest supported file format is Barracuda.
2015-11-26 21:27:24 12493 [Note] InnoDB: 128 rollback segment(s) are active.
2015-11-26 21:27:24 12493 [Note] InnoDB: Creating tablespace and datafile system tables.
2015-11-26 21:27:24 12493 [Note] InnoDB: Tablespace and datafile system tables created.
2015-11-26 21:27:24 12493 [Note] InnoDB: Waiting for purge to start
2015-11-26 21:27:24 12493 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-75.0 started; log sequence number 4538181520918
2015-11-26 21:27:24 12493 [ERROR] /usr/sbin/mysqld: unknown variable 'table_cache=2048'
2015-11-26 21:27:24 12493 [ERROR] Aborting

2015-11-26 21:27:24 12493 [Note] Binlog end
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'partition'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'ARCHIVE'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'BLACKHOLE'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'PERFORMANCE_SCHEMA'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CHANGED_PAGES'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_DATAFILES'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_TABLESPACES'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_FOREIGN_COLS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_FOREIGN'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_FIELDS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_COLUMNS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_INDEXES'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_TABLESTATS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_SYS_TABLES'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_FT_INDEX_TABLE'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_FT_INDEX_CACHE'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_FT_CONFIG'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_FT_BEING_DELETED'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_FT_DELETED'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_FT_DEFAULT_STOPWORD'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_METRICS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_BUFFER_POOL_STATS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_BUFFER_PAGE_LRU'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_BUFFER_PAGE'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CMP_PER_INDEX_RESET'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CMP_PER_INDEX'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CMPMEM_RESET'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CMPMEM'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CMP_RESET'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_CMP'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_LOCK_WAITS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_LOCKS'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'INNODB_TRX'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'XTRADB_RSEG'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'XTRADB_INTERNAL_HASH_TABLES'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'XTRADB_READ_VIEW'
2015-11-26 21:27:24 12493 [Note] Shutting down plugin 'InnoDB'
2015-11-26 21:27:24 12493 [Note] InnoDB: FTS optimize thread exiting.
2015-11-26 21:27:24 12493 [Note] InnoDB: Starting shutdown...
2015-11-26 21:27:27 12493 [Note] InnoDB: Shutdown completed; log sequence number 4538183224168
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'CSV'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'MyISAM'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'MRG_MYISAM'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'MEMORY'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'sha256_password'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'mysql_old_password'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'mysql_native_password'
2015-11-26 21:27:27 12493 [Note] Shutting down plugin 'binlog'
2015-11-26 21:27:27 12493 [Note] /usr/sbin/mysqld: Shutdown complete

151126 21:27:27 mysqld_safe mysqld from pid file /var/lib/mysql/upgrade-slave.pid ended
~~~

原因为某些参数在新的版本里已经不被支持，或者将要被废弃，或者已经改成了新的名字，解决方法是查阅文档，修改配置文件

---

## 修改配置文件my.cnf

~~~
[root@upgrade-slave ~]# diff /tmp/old.my.cnf /tmp/new.my.cnf
11c11
< table_cache = 2048
---
> table_open_cache = 2048
18d17
< thread_concurrency = 8
22c21
< default_table_type = INNODB
---
> default_storage_engine = INNODB
44c43
< myisam_recover
---
> myisam_recover_options
48d46
< innodb_ibuf_max_size = 1G
[root@upgrade-slave ~]# 
~~~

---

## 尝试启动mysql


~~~
[root@upgrade-slave ~]# /etc/init.d/mysql  start 
Starting MySQL (Percona Server)............                [  OK  ]
[root@upgrade-slave ~]# 
~~~


**/var/lib/mysql/upgrade-slave.err** 日志仍然报错

~~~
151126 21:35:58 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2015-11-26 21:35:59 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-11-26 21:35:59 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-75.0-log) starting as process 15776 ...
2015-11-26 21:35:59 15776 [Warning] option 'innodb-autoextend-increment': unsigned value 33554432 adjusted to 1000
2015-11-26 21:35:59 15776 [Note] Plugin 'FEDERATED' is disabled.
2015-11-26 21:35:59 7f104092f7e0 InnoDB: Warning: Using innodb_additional_mem_pool_size is DEPRECATED. This option may be removed in future releases, together with the option innodb_use_sys_malloc and with the InnoDB's internal memory allocator.
2015-11-26 21:35:59 15776 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-11-26 21:35:59 15776 [Note] InnoDB: The InnoDB memory heap is disabled
2015-11-26 21:35:59 15776 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-11-26 21:35:59 15776 [Note] InnoDB: Memory barrier is not used
2015-11-26 21:35:59 15776 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-11-26 21:35:59 15776 [Note] InnoDB: Using Linux native AIO
2015-11-26 21:35:59 15776 [Note] InnoDB: Using CPU crc32 instructions
2015-11-26 21:35:59 15776 [Note] InnoDB: Initializing buffer pool, size = 20.0G
2015-11-26 21:36:06 15776 [Note] InnoDB: Completed initialization of buffer pool
2015-11-26 21:36:07 15776 [Note] InnoDB: Highest supported file format is Barracuda.
2015-11-26 21:36:09 15776 [Note] InnoDB: 128 rollback segment(s) are active.
2015-11-26 21:36:09 15776 [Note] InnoDB: Waiting for purge to start
2015-11-26 21:36:09 15776 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-75.0 started; log sequence number 4538183224168
2015-11-26 21:36:09 15776 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: a6f64eac-9442-11e5-944e-44a842184bd8.
2015-11-26 21:36:09 15776 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
2015-11-26 21:36:09 15776 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
2015-11-26 21:36:09 15776 [Note] Server hostname (bind-address): '*'; port: 3306
2015-11-26 21:36:09 15776 [Note] IPv6 is available.
2015-11-26 21:36:09 15776 [Note]   - '::' resolves to '::';
2015-11-26 21:36:09 15776 [Note] Server socket created on IP: '::'.
2015-11-26 21:36:09 15776 [ERROR] Missing system table mysql.proxies_priv; please run mysql_upgrade to create it
2015-11-26 21:36:09 15776 [Warning] Info table is not ready to be used. Table 'mysql.slave_master_info' cannot be opened.
2015-11-26 21:36:09 15776 [Warning] Info table is not ready to be used. Table 'mysql.slave_relay_log_info' cannot be opened.
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'cond_instances' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_current' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_history' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_history_long' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_summary_by_host_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_summary_by_instance' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_summary_by_thread_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_summary_by_user_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_summary_by_account_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_waits_summary_global_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'file_instances' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'file_summary_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'file_summary_by_instance' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'host_cache' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'mutex_instances' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'objects_summary_global_by_type' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'performance_timers' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'rwlock_instances' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'setup_actors' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'setup_consumers' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'setup_instruments' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'setup_objects' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'setup_timers' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'table_io_waits_summary_by_index_usage' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'table_io_waits_summary_by_table' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'table_lock_waits_summary_by_table' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'threads' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_current' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_history' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_history_long' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_summary_by_thread_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_summary_by_account_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_summary_by_user_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_summary_by_host_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_stages_summary_global_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_current' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_history' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_history_long' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_summary_by_thread_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_summary_by_account_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_summary_by_user_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_summary_by_host_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_summary_global_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'events_statements_summary_by_digest' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'users' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'accounts' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'hosts' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'socket_instances' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'socket_summary_by_instance' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'socket_summary_by_event_name' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'session_connect_attrs' has the wrong structure
2015-11-26 21:36:09 15776 [ERROR] Native table 'performance_schema'.'session_account_connect_attrs' has the wrong structure
2015-11-26 21:36:09 15776 [Note] Event Scheduler: Loaded 0 events
2015-11-26 21:36:09 15776 [Note] /usr/sbin/mysqld: ready for connections.
Version: '5.6.27-75.0-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  Percona Server (GPL), Release 75.0, Revision 8bb53b6
~~~

报错表明schema的结构有问题

但是尝试登录已经可以正常登录

并且发现已经是新的版本了

~~~
[root@upgrade-slave ~]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.27-75.0-log Percona Server (GPL), Release 75.0, Revision 8bb53b6

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
~~~

解决schema的结构错误的方法是进行upgrade


---

## 进行mysql_upgrade

必须是实例正在运行的状态下使用upgrade

~~~
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
mysql.ndb_binlog_index                             OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.servers                                      OK
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
huodongdb.delivered_emails                         OK
huodongdb.prepared_emails                          OK
testdb.space_diaries                               OK
...
...
feeddb.ux_order_stats                              OK
feeddb.ux_product_stats                            OK
feeddb.weight_changes                              OK
test.checksum                                      OK
OK

real	2m4.418s
user	0m0.049s
sys	0m0.038s
[root@upgrade-slave ~]# 
~~~

修复完成

---

## 重启mysql

~~~
[root@upgrade-slave ~]# /etc/init.d/mysql  stop 
Shutting down MySQL (Percona Server)...                    [  OK  ]
[root@upgrade-slave ~]# /etc/init.d/mysql  start
Starting MySQL (Percona Server)...........                 [  OK  ]
[root@upgrade-slave ~]# 
~~~

---

## 检查日志

~~~
151126 21:49:21 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2015-11-26 21:49:21 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-11-26 21:49:21 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-75.0-log) starting as process 20186 ...
2015-11-26 21:49:21 20186 [Warning] option 'innodb-autoextend-increment': unsigned value 33554432 adjusted to 1000
2015-11-26 21:49:21 20186 [Note] Plugin 'FEDERATED' is disabled.
2015-11-26 21:49:21 7f8f2869e7e0 InnoDB: Warning: Using innodb_additional_mem_pool_size is DEPRECATED. This option may be removed in future releases, together with the option innodb_use_sys_malloc and with the InnoDB's internal memory allocator.
2015-11-26 21:49:21 20186 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-11-26 21:49:21 20186 [Note] InnoDB: The InnoDB memory heap is disabled
2015-11-26 21:49:21 20186 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-11-26 21:49:21 20186 [Note] InnoDB: Memory barrier is not used
2015-11-26 21:49:21 20186 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-11-26 21:49:21 20186 [Note] InnoDB: Using Linux native AIO
2015-11-26 21:49:21 20186 [Note] InnoDB: Using CPU crc32 instructions
2015-11-26 21:49:21 20186 [Note] InnoDB: Initializing buffer pool, size = 20.0G
2015-11-26 21:49:25 20186 [Note] InnoDB: Completed initialization of buffer pool
2015-11-26 21:49:29 20186 [Note] InnoDB: Highest supported file format is Barracuda.
2015-11-26 21:49:30 20186 [Note] InnoDB: 128 rollback segment(s) are active.
2015-11-26 21:49:30 20186 [Note] InnoDB: Waiting for purge to start
2015-11-26 21:49:30 20186 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-75.0 started; log sequence number 4538185053276
2015-11-26 21:49:30 20186 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
2015-11-26 21:49:30 20186 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
2015-11-26 21:49:30 20186 [Note] Server hostname (bind-address): '*'; port: 3306
2015-11-26 21:49:30 20186 [Note] IPv6 is available.
2015-11-26 21:49:30 20186 [Note]   - '::' resolves to '::';
2015-11-26 21:49:30 20186 [Note] Server socket created on IP: '::'.
2015-11-26 21:49:30 20186 [Note] Event Scheduler: Loaded 0 events
2015-11-26 21:49:30 20186 [Note] /usr/sbin/mysqld: ready for connections.
Version: '5.6.27-75.0-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  Percona Server (GPL), Release 75.0, Revision 8bb53b6
~~~

schema的结构报错已经消失了，启动正常

---

## 同步数据库

~~~
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |      120 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> show slave status\G
Empty set (0.00 sec)

mysql> 
mysql>  CHANGE MASTER TO MASTER_HOST='192.168.100.123', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000899',MASTER_LOG_POS=451678342,MASTER_USER='repl',MASTER_PASSWORD='xxxxxx';
Query OK, 0 rows affected, 2 warnings (0.33 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.100.123
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000899
          Read_Master_Log_Pos: 451678342
               Relay_Log_File: relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000899
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,feeddb,mytempdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 451678342
              Relay_Log_Space: 120
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
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.03 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Queueing master event to the relay log
                  Master_Host: 192.168.100.123
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000899
          Read_Master_Log_Pos: 465602930
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 585
        Relay_Master_Log_File: mysql-bin.000899
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,feeddb,mytempdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 451678658
              Relay_Log_Space: 13925024
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 82821
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 3
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Opening tables
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.100.123
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000900
          Read_Master_Log_Pos: 745342806
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 51863048
        Relay_Master_Log_File: mysql-bin.000899
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,feeddb,mytempdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 503541121
              Relay_Log_Space: 1368280295
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 77965
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 3
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: System lock
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> 
~~~

同步一段时间过后


~~~
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.100.123
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000900
          Read_Master_Log_Pos: 766875783
               Relay_Log_File: relay-bin.000004
                Relay_Log_Pos: 766875942
        Relay_Master_Log_File: mysql-bin.000900
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,feeddb,mytempdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 766875783
              Relay_Log_Space: 766876148
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
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 3
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> 
~~~


---

## 检查数据一致性

同步完成后，检查一致性，抽查一个关键表

~~~
[root@upgrade-slave ~]# pt-table-checksum --nocheck-replication-filters --nocheck-binlog-format --replicate=test.checksum --tables key_db.users  h=upgrade-master,u=root --ask-pass 
Enter MySQL password: 
Checksumming key_db.users:   8% 05:38 remain
Checksumming key_db.users:  16% 05:13 remain
Checksumming key_db.users:  25% 04:29 remain
Checksumming key_db.users:  35% 03:48 remain
Checksumming key_db.users:  43% 03:20 remain
Checksumming key_db.users:  50% 03:02 remain
Checksumming key_db.users:  55% 02:55 remain
Checksumming key_db.users:  59% 02:45 remain
Checksumming key_db.users:  63% 02:37 remain
Checksumming key_db.users:  68% 02:23 remain
Checksumming key_db.users:  73% 02:04 remain
Checksumming key_db.users:  78% 01:40 remain
Checksumming key_db.users:  84% 01:14 remain
Checksumming key_db.users:  90% 00:48 remain
Checksumming key_db.users:  95% 00:22 remain
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
11-26T22:56:42      0     11 14940167     153       0 562.858 key_db.users
[root@upgrade-slave ~]#
[root@upgrade-slave ~]# pt-table-sync --replicate test.checksum  h=upgrade-slave,u=root --ask-pass  --sync-to-master --databases=key_db  --tables=users  --print > /tmp/users.sql
Enter password for upgrade-slave: 
[root@upgrade-slave ~]# 
[root@upgrade-slave ~]# wc -l /tmp/users.sql 
1 /tmp/users.sql
[root@upgrade-slave ~]# cat /tmp/users.sql 

[root@upgrade-slave ~]#
~~~

没有发现不一致数据，一致性检查通过

然后找一个业务低点进行业务切换



---


[percona]:https://www.percona.com/software/mysql-database/percona-server

