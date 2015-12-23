---
layout: post
title:  mysql 迁移
categories: linux mysql upgrade keepalived
wc: 1093 3654 41156
excerpt: 安全高效地进行mysql迁移与slave升级，重在迁移过程与检查流程
comments: true
---


---

#前言


有了keepalived和mysql upgrade的技术作基础，可以结合两者完成无逢升级


下面分享一下我的 **Best Practice Of Mysql Migration**

		
		
---


#概要

* TOC
{:toc}



---

##准备工作

下面步骤最好作为准备工作，提前完成，这样可以更有效进行切换，和减少窗口期

* **挂载NFS**
* **安装软件包的收集(mysql,xtrabackup,keepalived)**
* **安装keepalived**
* **窗口选择与提前通知**
* **确认备份数据**
* **监控代理安装与配置**
* **代理(zabbix)权限准备**
* **监控工具准备**
* **操作命令准备**
* **安装xtrabackup**
* **准备新版配置文件my.cnf**

---

##挂载NFS

用于备份重建slave

{% highlight bash %}
[root@new-slave ~]# yum clean all 
Loaded plugins: fastestmirror
Cleaning repos: base epel extras newrelic updates
Cleaning up Everything
Cleaning up list of fastest mirrors
[root@new-slave ~]# yum install nfs-utils rpcbind
...
...
Installed:
  nfs-utils.x86_64 1:1.2.3-64.el6                              rpcbind.x86_64 0:0.2.0-11.el6                             

Dependency Installed:
  keyutils.x86_64 0:1.4-5.el6        libevent.x86_64 0:1.4.13-4.el6          libgssglue.x86_64 0:0.1-11.el6            
  libtirpc.x86_64 0:0.2.1-10.el6     nfs-utils-lib.x86_64 0:1.1.5-11.el6     python-argparse.noarch 0:1.2.1-2.1.el6    

Dependency Updated:
  keyutils-libs.x86_64 0:1.4-5.el6                         keyutils-libs-devel.x86_64 0:1.4-5.el6                        

Complete!
[root@new-slave ~]# showmount  -e new-master
Export list for new-master:
/data/nfs 10.0.0.10,10.0.0.11,10.0.0.13
[root@new-slave ~]# cd /data/
[root@new-slave data]# mount -t nfs -o intr new-master:/data/nfs /data/nfs/ 
[root@new-slave data]# df -h | grep nfs 
new-master:/data/nfs
                      548G   46G  475G   9% /data/nfs
[root@new-slave data]# 
{% endhighlight %}

---


##新版master上安装并启动keepalived

安装并启动keepalived

{% highlight bash %}
[root@new-master ~]# yum -y install  keepalived.x86_64
...
...
[root@new-master ~]# cd /etc/keepalived/
[root@new-master keepalived]# cp keepalived.conf keepalived.conf.bak.201512xx
[root@new-master keepalived]# > keepalived.conf
[root@new-master keepalived]# vim keepalived.conf
[root@new-master keepalived]# vim /etc/sysconfig/iptables
[root@new-master keepalived]# grep 18 /etc/sysconfig/iptables 
-A INPUT -d 224.0.0.18 -j ACCEPT
[root@new-master keepalived]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@new-master keepalived]# iptables -L -nv  | grep 224 
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0            224.0.0.18          
[root@new-master keepalived]# /etc/init.d/keepalived start 
Starting keepalived:                                       [  OK  ]
[root@new-master keepalived]# 
{% endhighlight %}

确认配置

{% highlight bash %}
[testuser@new-master ~]$ ps faux | grep keep  | grep -v grep 
root      73609  0.0  0.0 110276  1144 ?        Ss   Sep25   2:17 /usr/sbin/keepalived -D
root      73610  0.0  0.0 112500  2908 ?        S    Sep25   2:21  \_ /usr/sbin/keepalived -D
root      73611  0.0  0.0 112484  2064 ?        S    Sep25  18:15  \_ /usr/sbin/keepalived -D
[testuser@new-master ~]$ cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id LVS_new_master
}


vrrp_instance VI_3 {
    state BACKUP 
    interface em1
    virtual_router_id 3
    priority 150 
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
	192.168.66.6/24
    }
}
[testuser@new-master ~]$ 
{% endhighlight %}


**Note:**

* 优先级相对原master要低，否则会抢ip
* 两个keepalived 上 **advert_int** 要设为 **1** 为了尽快完成切换

---

##切换时间选择在业务低点

一般选择业务低点进行数据库操作，目的是为了降低业务风险，和数据丢失的风险

通过监控历史数据可以确定这个时间窗口

---

##关闭原集群mha

由于集群软件在侦测到主服务器失效后会干预相关资源，造成备机身份切换和IP飘移，为了避免这种影响，要关掉集群


{% highlight bash %}
[mysql@slave02 bin]$ masterha_check_status --conf=/etc/app1.cnf
app1 (pid:18911) is running(0:PING_OK), master:origin-master
[mysql@slave02 bin]$ masterha_stop --conf=/etc/app1.cnf
Stopped app1 successfully.
[mysql@slave02 bin]$ masterha_check_status --conf=/etc/app1.cnf
app1 is stopped(2:NOT_RUNNING).
[mysql@slave02 bin]$ ps faux | grep manager
mysql    27192  0.0  0.0 103244   864 pts/2    S+   00:23   0:00                                  \_ grep manager
[mysql@slave02 bin]$
{% endhighlight %}

---

##关闭原slave上keepalived


此目的是为了减少三个keepalived之间协商优先级的时间

{% highlight bash %}
[root@slave01 tmp]# ps faux | grep keep 
root     25745  0.0  0.0 103244   864 pts/0    S+   00:25   0:00                          \_ grep keep
root     17798  0.0  0.0 110196   432 ?        Ss   Aug12  12:49 /usr/sbin/keepalived -D
root     17799  0.0  0.0 112300  1564 ?        S    Aug12  13:18  \_ /usr/sbin/keepalived -D
root     17800  0.0  0.0 112300  1036 ?        S    Aug12  88:32  \_ /usr/sbin/keepalived -D
[root@slave01 tmp]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:50:56:93:7c:78 brd ff:ff:ff:ff:ff:ff
    inet 192.168.66.123/24 brd 192.168.66.255 scope global eth0
    inet6 fe80::250:56ff:fe93:7c78/64 scope link 
       valid_lft forever preferred_lft forever
[root@slave01 tmp]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   router_id LVS_slave01
}

vrrp_instance VI_3 {
    state MASTER 
    interface eth0
    virtual_router_id 3
    priority 85 
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
	192.168.66.6/24
    }
}
[root@slave01 tmp]# /etc/init.d/keepalived stop 
Stopping keepalived:                                       [  OK  ]
[root@slave01 tmp]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:50:56:93:7c:78 brd ff:ff:ff:ff:ff:ff
    inet 192.168.66.123/24 brd 192.168.66.255 scope global eth0
    inet6 fe80::250:56ff:fe93:7c78/64 scope link 
       valid_lft forever preferred_lft forever
[root@slave01 tmp]# ps faux |  grep keep 
root     28544  0.0  0.0 103244   864 pts/0    S+   00:28   0:00                          \_ grep keep
[root@slave01 tmp]# 
{% endhighlight %}

---

##再次检查,确认备份数据

这是最后一次备份原数据的机会 


---

##切换keepalived ip

变更新master keepalived优先级，重载的方式切换

{% highlight bash %}
[root@new-master ~]# vim /etc/keepalived/keepalived.conf
[root@new-master ~]# /etc/init.d/keepalived reload ; watch -n .2 ip a 
{% endhighlight %}

使用给新master keepalived 升优先级重载的方式切IP

使用 **watch** 来观察ip变化

---

##从两边密切监控观察检查应用与数据库状态


使用netstat 观察到数据库的连接比如 **:3306**

在数据库里可以使用 **show processlist** 来看连接

(必要的时候可以停止原master数据库)


{% highlight bash %}
[root@origin-master ~]# /etc/init.d/mysql  stop 
Shutting down MySQL........................................[  OK  ].
[root@origin-master ~]# 
{% endhighlight %}


主要观察DB层APP层有无报错，确认无报错，正常后再进行下步


---

##确认备份数据 

在销毁slave和原master前，这是最后一次可以备份原库统计数据的机会

生产数据已经陈旧，不一致了

确认后可以进行下一步

---

##备份新master以便重建


{% highlight bash %}
[root@new-master nfs]# time nohup /usr/bin/innobackupex --defaults-file=/etc/my.cnf --user=root --password=xxxxxxxxxx /data/nfs/test_full_backup    >>  /data/nfs/full_backup.log 2>&1  &
[1] 80736
[root@new-master nfs]# 
[root@new-master nfs]# 
[root@new-master nfs]# tail -f full_backup.log 
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 3
xtrabackup:   innodb_log_file_size = 268435456
xtrabackup: using O_DIRECT
>> log scanned up to (4998975642548)
xtrabackup: Generating a list of tablespaces
>> log scanned up to (4998975644454)
{% endhighlight %}

---

##销毁slave数据库

如果有足够空间，可以备到一个目录，没有则可以直接删


{% highlight bash %}
[root@slave01 data]# /etc/init.d/mysql  stop 
Shutting down MySQL.............. SUCCESS! 
[root@slave01 data]# cd /var/lib/mysql/
[root@slave01 mysql]# ls
livedb      slave01-relay-bin.000308  javadb      ijavadb     mysql-bin.000154  mysql-bin.000159  relay-log.info
backup-my.cnf  slave01-relay-bin.000309  ibdata1      wavedb     mysql-bin.000155  mysql-bin.000160  functiondb
mysqltest_his       slave01-relay-bin.index   ib_logfile0  master.info  mysql-bin.000156  mysql-bin.index   test
mysqltestt_db        slave01-slow.log          ib_logfile1  mobildb   mysql-bin.000157  testdb
slave01.err     stuff_on              ib_logfile2  mysql        mysql-bin.000158  keydb
[root@slave01 mysql]# rm -rf * 
[root@slave01 mysql]# ls
[root@slave01 mysql]# df -h 
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda3              16G  3.7G   12G  25% /
tmpfs                  16G     0   16G   0% /dev/shm
/dev/sda1             194M   66M  119M  36% /boot
/dev/sdb1             493G  1.3G  466G   1% /data
new-master:/data/nfs
                      1.7T  300G  1.4T  19% /data/nfs
[root@slave01 mysql]# 
{% endhighlight %}

如果有多个slave ,重复上面操作

---

##更新slave mysql版本

{% highlight bash %}
[root@slave02 src]# rpm -e Percona-Server-client-51-5.1.73-rel14.11.603.rhel6.x86_64 Percona-Server-server-51-5.1.73-rel14.11.603.rhel6.x86_64 Percona-Server-shared-51-5.1.73-rel14.11.603.rhel6.x86_64  
error: Failed dependencies:
	mysql is needed by (installed) xtrabackup-1.6.7-356.rhel6.x86_64
	libmysqlclient.so.16()(64bit) is needed by (installed) postfix-2:2.6.6-6.el6_5.x86_64
	libmysqlclient.so.16()(64bit) is needed by (installed) Percona-Server-devel-51-5.1.73-rel14.11.603.rhel6.x86_64
	libmysqlclient.so.16()(64bit) is needed by (installed) perl-DBD-MySQL-4.013-3.el6.x86_64
	libmysqlclient.so.16()(64bit) is needed by (installed) php-mysql-5.3.3-40.el6_6.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) postfix-2:2.6.6-6.el6_5.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) perl-DBD-MySQL-4.013-3.el6.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) php-mysql-5.3.3-40.el6_6.x86_64
	libmysqlclient_r.so.16()(64bit) is needed by (installed) sysbench-0.4.12-5.el6.x86_64
	libmysqlclient_r.so.16()(64bit) is needed by (installed) Percona-Server-devel-51-5.1.73-rel14.11.603.rhel6.x86_64
	libmysqlclient_r.so.16(libmysqlclient_16)(64bit) is needed by (installed) sysbench-0.4.12-5.el6.x86_64
	mysql-libs is needed by (installed) postfix-2:2.6.6-6.el6_5.x86_64
[root@slave02 src]# rpm -e Percona-Server-client-51-5.1.73-rel14.11.603.rhel6.x86_64 Percona-Server-server-51-5.1.73-rel14.11.603.rhel6.x86_64 Percona-Server-shared-51-5.1.73-rel14.11.603.rhel6.x86_64   --nodeps
[root@slave02 percona]# rpm -ivh Percona-Server-client-56-5.6.27-rel76.0.el6.x86_64.rpm Percona-Server-server-56-5.6.27-rel76.0.el6.x86_64.rpm Percona-Server-shared-56-5.6.27-rel76.0.el6.x86_64.rpm 
warning: Percona-Server-client-56-5.6.27-rel76.0.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
error: Failed dependencies:
	libnuma.so.1()(64bit) is needed by Percona-Server-server-56-5.6.27-rel76.0.el6.x86_64
	libnuma.so.1(libnuma_1.1)(64bit) is needed by Percona-Server-server-56-5.6.27-rel76.0.el6.x86_64
	libnuma.so.1(libnuma_1.2)(64bit) is needed by Percona-Server-server-56-5.6.27-rel76.0.el6.x86_64
[root@slave02 percona]#
[root@slave02 percona56]# wget http://mirror.centos.org/centos/6/os/x86_64/Packages/numactl-2.0.9-2.el6.x86_64.rpm
--2015-12-09 01:22:50--  http://mirror.centos.org/centos/6/os/x86_64/Packages/numactl-2.0.9-2.el6.x86_64.rpm
Resolving mirror.centos.org... 103.27.60.52
Connecting to mirror.centos.org|103.27.60.52|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 75912 (74K) [application/x-rpm]
Saving to: “numactl-2.0.9-2.el6.x86_64.rpm”

100%[===============================================================================>] 75,912       197K/s   in 0.4s    

2015-12-09 01:22:51 (197 KB/s) - “numactl-2.0.9-2.el6.x86_64.rpm” saved [75912/75912]

[root@slave02 percona56]# 
[root@slave02 percona56]# rpm -ivh  numactl-2.0.9-2.el6.x86_64.rpm 
Preparing...                ########################################### [100%]
   1:numactl                ########################################### [100%]
[root@slave02 percona56]# 
[root@slave02 percona56]# rpm -ivh  Percona-Server-client-56-5.6.27-rel75.0.el6.x86_64.rpm Percona-Server-server-56-5.6.27-rel75.0.el6.x86_64.rpm Percona-Server-shared-56-5.6.27-rel75.0.el6.x86_64.rpm 
warning: Percona-Server-client-56-5.6.27-rel75.0.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:Percona-Server-shared-5########################################### [ 33%]
ln: creating symbolic link `/usr/lib64/libmysqlclient.so': File exists
ln: creating symbolic link `/usr/lib64/libmysqlclient_r.so': File exists
   2:Percona-Server-client-5########################################### [ 67%]
   3:Percona-Server-server-5########################################### [100%]
2015-12-09 01:23:47 0 [Warning] 'THREAD_CONCURRENCY' is deprecated and will be removed in a future release.
2015-12-09 01:23:47 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-12-09 01:23:47 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-75.0-log) starting as process 1855 ...
2015-12-09 01:23:47 7fdbea6e67e0 InnoDB: Warning: Using innodb_additional_mem_pool_size is DEPRECATED. This option may be removed in future releases, together with the option innodb_use_sys_malloc and with the InnoDB's internal memory allocator.
2015-12-09 01:23:47 1855 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-12-09 01:23:47 1855 [Note] InnoDB: The InnoDB memory heap is disabled
2015-12-09 01:23:47 1855 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-12-09 01:23:47 1855 [Note] InnoDB: Memory barrier is not used
2015-12-09 01:23:47 1855 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-12-09 01:23:47 1855 [Note] InnoDB: Using Linux native AIO
2015-12-09 01:23:47 1855 [Note] InnoDB: Using CPU crc32 instructions
2015-12-09 01:23:47 1855 [Note] InnoDB: Initializing buffer pool, size = 4.0G
2015-12-09 01:23:47 1855 [Note] InnoDB: Completed initialization of buffer pool
2015-12-09 01:23:47 1855 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2015-12-09 01:23:47 1855 [Note] InnoDB: Setting file ./ibdata1 size to 10 MB
2015-12-09 01:23:47 1855 [Note] InnoDB: Database physically writes the file full: wait...
2015-12-09 01:23:47 1855 [Note] InnoDB: Setting log file ./ib_logfile101 size to 256 MB
InnoDB: Progress in MB: 100 200
2015-12-09 01:23:48 1855 [Note] InnoDB: Setting log file ./ib_logfile1 size to 256 MB
InnoDB: Progress in MB: 100 200
2015-12-09 01:23:48 1855 [Note] InnoDB: Setting log file ./ib_logfile2 size to 256 MB
InnoDB: Progress in MB: 100 200
2015-12-09 01:23:49 1855 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2015-12-09 01:23:49 1855 [Warning] InnoDB: New log files created, LSN=45781
2015-12-09 01:23:49 1855 [Note] InnoDB: Doublewrite buffer not found: creating new
2015-12-09 01:23:49 1855 [Note] InnoDB: Doublewrite buffer created
2015-12-09 01:23:49 1855 [Note] InnoDB: 128 rollback segment(s) are active.
2015-12-09 01:23:49 1855 [Warning] InnoDB: Creating foreign key constraint system tables.
2015-12-09 01:23:49 1855 [Note] InnoDB: Foreign key constraint system tables created
2015-12-09 01:23:49 1855 [Note] InnoDB: Creating tablespace and datafile system tables.
2015-12-09 01:23:49 1855 [Note] InnoDB: Tablespace and datafile system tables created.
2015-12-09 01:23:49 1855 [Note] InnoDB: Waiting for purge to start
2015-12-09 01:23:50 1855 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-75.0 started; log sequence number 0
2015-12-09 01:23:50 1855 [ERROR] /usr/sbin/mysqld: unknown variable 'table_cache=2048'
2015-12-09 01:23:50 1855 [ERROR] Aborting

2015-12-09 01:23:50 1855 [Note] Binlog end
2015-12-09 01:23:50 1855 [Note] InnoDB: FTS optimize thread exiting.
2015-12-09 01:23:50 1855 [Note] InnoDB: Starting shutdown...
2015-12-09 01:23:52 1855 [Note] InnoDB: Shutdown completed; log sequence number 1600615
2015-12-09 01:23:52 1855 [Note] /usr/sbin/mysqld: Shutdown complete

Percona Server is distributed with several useful UDF (User Defined Function) from Percona Toolkit.
Run the following commands to create these functions:
mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"
mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"
mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"
See http://www.percona.com/doc/percona-server/5.6/management/udf_percona_toolkit.html for more details
[root@slave02 percona56]# echo $?
0
[root@slave02 percona56]# 
{% endhighlight %}

多个slave，而重复上面操作

---

##备份替换my.cnf配置文件

{% highlight bash %}
[root@slave02 etc]# mv my.cnf my.old.2015.12.09.backup
[root@slave02 etc]# mv mynew02.cnf  my.cnf
[root@slave02 etc]# ll my*
-rw-r--r-- 1 root root  1928 Dec  8 21:18 my.cnf
-rw-r--r-- 1 root root 21146 Jul  4  2014 my.old.2015.12.09.backup
[root@slave02 etc]# 
{% endhighlight %}

部分参数已经在新版本中不用，或更名，要进行替换

{% highlight bash %}
[testuser@slave01 etc]$ diff /tmp/old /tmp/new
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
23a23
> relay-log=relay-bin
31c31
< server-id = 123
---
> server-id = 321 
41,42c41
< myisam_max_sort_file_size = 5G
< myisam_max_extra_sort_file_size = 5G
---
> myisam_max_sort_file_size = 1G
44c43
< myisam_recover
---
> myisam_recover_options
[testuser@slave01 etc]$ 
{% endhighlight %}

innodb_additional_mem_pool_size  也已经被弃用了,如果有要注释掉


---

##将zabbit加入mysql组以方便监控


{% highlight bash %}
[root@new-master mysql]# vim /etc/group
[root@new-master mysql]# id zabbix
uid=496(zabbix) gid=493(zabbix) groups=493(zabbix),492(mysql)
[root@new-master mysql]# /etc/init.d/zabbix-agent restart 
Shutting down Zabbix agent:                                [  OK  ]
Starting Zabbix agent:                                     [  OK  ]
[root@new-master mysql]# 
----------
[root@zabbix-server ~]# zabbix_get -s new-master -p 10050 -k "mysql.slowlog[100,/var/lib/mysql/new-master-slow.log]"
2.98465
[root@zabbix-server ~]# 
{% endhighlight %}

---

##修改zabbix统计数据过期时间

{% highlight bash %}
[root@new-master mysql]# vim  /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh
[root@new-master mysql]# grep 120  /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh
    if [ `expr $TIMENOW - $TIMEFLM` -gt 120 ]; then
[root@new-master mysql]# 
{% endhighlight %}

修改之前是300,也就是5分钟,这个监控粒度太粗,所以改为120

---

##安装percona-xtrabackup


{% highlight bash %}
[root@slave01 percona56]# rpm -ivh  percona-xtrabackup-2.3.2-1.el6.x86_64.rpm
warning: percona-xtrabackup-2.3.2-1.el6.x86_64.rpm: Header V4 DSA/SHA1
Signature, key ID cd2efd2a: NOKEY
error: Failed dependencies:
	libev.so.4()(64bit) is needed by percona-xtrabackup-2.3.2-1.el6.x86_64
[root@slave01 percona56]# yum install libev
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.opencas.cn
 * epel: mirrors.opencas.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.opencas.cn
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package libev.x86_64 0:4.03-3.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================
 Package                    Arch                        Version
Repository                 Size
=========================================================================================================================
Installing:
 libev                      x86_64                      4.03-3.el6
epel                      113 k

Transaction Summary
=========================================================================================================================
Install       1 Package(s)

Total download size: 113 k
Installed size: 151 k
Is this ok [y/N]: y
Downloading Packages:
libev-4.03-3.el6.x86_64.rpm
| 113 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
** Found 11 pre-existing rpmdb problem(s), 'yum check' output follows:
Percona-Server-devel-51-5.1.73-rel14.11.603.rhel6.x86_64 has missing requires
of libmysqlclient.so.16()(64bit)
Percona-Server-devel-51-5.1.73-rel14.11.603.rhel6.x86_64 has missing requires
of libmysqlclient_r.so.16()(64bit)
perl-DBD-MySQL-4.013-3.el6.x86_64 has missing requires of
libmysqlclient.so.16()(64bit)
perl-DBD-MySQL-4.013-3.el6.x86_64 has missing requires of
libmysqlclient.so.16(libmysqlclient_16)(64bit)
php-mysql-5.3.3-40.el6_6.x86_64 has missing requires of
libmysqlclient.so.16()(64bit)
php-mysql-5.3.3-40.el6_6.x86_64 has missing requires of
libmysqlclient.so.16(libmysqlclient_16)(64bit)
2:postfix-2.6.6-6.el6_5.x86_64 has missing requires of
libmysqlclient.so.16()(64bit)
2:postfix-2.6.6-6.el6_5.x86_64 has missing requires of
libmysqlclient.so.16(libmysqlclient_16)(64bit)
2:postfix-2.6.6-6.el6_5.x86_64 has missing requires of mysql-libs
sysbench-0.4.12-5.el6.x86_64 has missing requires of
libmysqlclient_r.so.16()(64bit)
sysbench-0.4.12-5.el6.x86_64 has missing requires of
libmysqlclient_r.so.16(libmysqlclient_16)(64bit)
  Installing : libev-4.03-3.el6.x86_64
1/1 
  Verifying  : libev-4.03-3.el6.x86_64
1/1 

Installed:
  libev.x86_64 0:4.03-3.el6                                                                                              

Complete!
[root@slave01 percona56]# rpm -ivh  percona-xtrabackup-2.3.2-1.el6.x86_64.rpm
warning: percona-xtrabackup-2.3.2-1.el6.x86_64.rpm: Header V4 DSA/SHA1
Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
	file /usr/bin/innobackupex from install of
percona-xtrabackup-2.3.2-1.el6.x86_64 conflicts with file from package
xtrabackup-1.6.7-356.rhel6.x86_64
	file /usr/bin/xtrabackup from install of
percona-xtrabackup-2.3.2-1.el6.x86_64 conflicts with file from package
xtrabackup-1.6.7-356.rhel6.x86_64
[root@slave01 percona56]# rpm -qa | grep backup
xtrabackup-1.6.7-356.rhel6.x86_64
[root@slave01 percona56]# rpm -e xtrabackup-1.6.7-356.rhel6.x86_64 
[root@slave01 percona56]# rpm -ivh  percona-xtrabackup-2.3.2-1.el6.x86_64.rpm
warning: percona-xtrabackup-2.3.2-1.el6.x86_64.rpm: Header V4 DSA/SHA1
Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:percona-xtrabackup     ########################################### [100%]
[root@slave01 percona56]# 
[root@slave01 percona56]# innobackupex --version
innobackupex version 2.3.2 Linux (x86_64) (revision id: 306a2e0)
[root@slave01 percona56]# 
{% endhighlight %}

slave上确保安装好 **percona-xtrabackup** 软件

---


##备份完成

{% highlight bash %}
xtrabackup: Creating suspend file '/data/nfs/test_full_backup/2015-12-09_00-53-03/xtrabackup_log_copied' with pid '80799'
xtrabackup: Transaction log of lsn (4998915938330) to (4998984695861) was copied.
151209 02:06:09  innobackupex: Executing UNLOCK BINLOG
151209 02:06:09  innobackupex: Executing UNLOCK TABLES
151209 02:06:09  innobackupex: All tables unlocked

innobackupex: Backup created in directory '/data/nfs/test_full_backup/2015-12-09_00-53-03'
innobackupex: MySQL binlog position: filename 'mysql-bin.000004', position 8299670
151209 02:06:09  innobackupex: Connection to database server closed
151209 02:06:09  innobackupex: completed OK!

real	73m8.390s
user	6m33.032s
sys	7m44.089s

[1]+  Done                    time nohup /usr/bin/innobackupex --defaults-file=/etc/my.cnf --user=root --password=xxxxxxxxxx /data/nfs/test_full_backup >> /data/nfs/full_backup.log 2>&1
[root@new-master nfs]# 
{% endhighlight %}

---

##准备恢复

{% highlight bash %}
[root@slave01 nfs]# which innobackupex 
/usr/bin/innobackupex
[root@slave01 nfs]# time nohup /usr/bin/innobackupex --apply-log /data/nfs/test_full_backup/2015-12-09_00-53-03/
nohup: ignoring input and appending output to `nohup.out'

real	1m43.335s
user	0m20.049s
sys	0m9.147s
[root@slave01 nfs]# 
{% endhighlight %}

---

##恢复数据

{% highlight bash %}
[root@slave01 data]# time nohup /usr/bin/innobackupex --copy-back /data/nfs/test_full_backup/2015-12-09_00-53-03/ >> restore.log  2>&1   &  
[1] 12635
[root@slave01 data]# 
[root@slave01 data]# tail -f restore.log 
...
...
{% endhighlight %}


要确保mysql 数据库的 datadir是清空的，否则会报错


{% highlight bash %}
[root@slave02 data]# cat restore.log 
nohup: ignoring input
Warning: option 'innodb_autoextend_increment': unsigned value 33554432
adjusted to 1000
151209 02:40:21 innobackupex: Starting the copy-back operation

IMPORTANT: Please check that the copy-back run completes successfully.
           At the end of a successful copy-back run innobackupex
           prints "completed OK!".

/usr/bin/innobackupex version 2.3.2 based on MySQL server 5.6.24 Linux
(x86_64) (revision id: 306a2e0)
Original data directory /var/lib/mysql is not empty!
nohup: ignoring input
Warning: option 'innodb_autoextend_increment': unsigned value 33554432
adjusted to 1000
151209 02:41:08 innobackupex: Starting the copy-back operation

IMPORTANT: Please check that the copy-back run completes successfully.
           At the end of a successful copy-back run innobackupex
           prints "completed OK!".

/usr/bin/innobackupex version 2.3.2 based on MySQL server 5.6.24 Linux
(x86_64) (revision id: 306a2e0)
Original data directory /var/lib/mysql is not empty!
[root@slave02 data]#
{% endhighlight %}

一般而言，**`rm -rf *`** 并不会删除以 `.` 开头的文件 如： **`.bash_history  .lesshst  .mysql_history  .viminfo`** 

要指明删，如 **`rm -rf  .bash_history  .lesshst  .mysql_history  .viminfo`**

或 **`rm -rf .*`** 

使用 **`ls -a`** 以确认

---

##监测进展

{% highlight bash %}
[root@slave02 data]# watch -n 2 du -sh /data/mysql/
{% endhighlight %}

每两秒看一下数据目录大小

---

##恢复完成


{% highlight bash %}
151209 03:57:34 [01] Copying ./mysqltestt_db/kqmobile_payments.ibd to
/var/lib/mysql/mysqltestt_db/kqmobile_payments.ibd
151209 03:57:34 [01]        ...done
151209 03:57:34 [01] Copying ./mysqltestt_db/member_orders.frm to
/var/lib/mysql/mysqltestt_db/member_orders.frm
151209 03:57:34 [01]        ...done
151209 03:57:34 [01] Copying ./mysqltestt_db/smsg_runner_logs.ibd to
/var/lib/mysql/mysqltestt_db/smsg_runner_logs.ibd
151209 03:57:34 [01]        ...done
151209 03:57:34 completed OK!

real	73m17.260s
user	0m0.782s
sys	19m4.516s
^C
[1]+  Done                    time nohup /usr/bin/innobackupex --copy-back
/data/nfs/test_full_backup/2015-12-09_00-53-03/ >> restore.log 2>&1
[root@slave02 data]# 
{% endhighlight %}


---

##修改权限


{% highlight bash %}
[root@slave02 mysql]# cat xtrabackup_binlog_pos_innodb 
mysql-bin.000004	8299670
[root@slave02 mysql]# ll 
total 5916780
drwx------ 2 root root       4096 Dec  9 02:49 livedb
drwx------ 2 root root       4096 Dec  9 02:57 mysqltest_his
drwx------ 2 root root      36864 Dec  9 03:57 mysqltestt_db
drwx------ 2 root root       4096 Dec  9 03:08 stuff_on
drwx------ 2 root root       4096 Dec  9 02:52 javadb
-rw-r----- 1 root root 5253365760 Dec  9 02:45 ibdata1
-rw-r----- 1 root root  268435456 Dec  9 02:44 ib_logfile0
-rw-r----- 1 root root  268435456 Dec  9 02:44 ib_logfile1
-rw-r----- 1 root root  268435456 Dec  9 02:44 ib_logfile2
drwx------ 2 root root       4096 Dec  9 02:52 ijavadb
drwx------ 2 root root      12288 Dec  9 03:08 wavedb
drwx------ 2 root root       4096 Dec  9 03:08 mobildb
drwx------ 2 root root       4096 Dec  9 02:45 mysql
drwx------ 2 root root       4096 Dec  9 02:49 testdb
drwx------ 2 root root       4096 Dec  9 02:49 keydb
drwx------ 2 root root       4096 Dec  9 03:08 performance_schema
drwx------ 2 root root       4096 Dec  9 02:52 functiondb
drwx------ 2 root root       4096 Dec  9 02:52 test
-rw-r----- 1 root root         25 Dec  9 03:08 xtrabackup_binlog_pos_innodb
-rw-r----- 1 root root        710 Dec  9 03:08 xtrabackup_info
[root@slave02 mysql]# chown  -R mysql.mysql /var/lib/mysql/
[root@slave02 mysql]# ll 
total 5916780
drwx------ 2 mysql mysql       4096 Dec  9 02:49 livedb
drwx------ 2 mysql mysql       4096 Dec  9 02:57 mysqltest_his
drwx------ 2 mysql mysql      36864 Dec  9 03:57 mysqltestt_db
drwx------ 2 mysql mysql       4096 Dec  9 03:08 stuff_on
drwx------ 2 mysql mysql       4096 Dec  9 02:52 javadb
-rw-r----- 1 mysql mysql 5253365760 Dec  9 02:45 ibdata1
-rw-r----- 1 mysql mysql  268435456 Dec  9 02:44 ib_logfile0
-rw-r----- 1 mysql mysql  268435456 Dec  9 02:44 ib_logfile1
-rw-r----- 1 mysql mysql  268435456 Dec  9 02:44 ib_logfile2
drwx------ 2 mysql mysql       4096 Dec  9 02:52 ijavadb
drwx------ 2 mysql mysql      12288 Dec  9 03:08 wavedb
drwx------ 2 mysql mysql       4096 Dec  9 03:08 mobildb
drwx------ 2 mysql mysql       4096 Dec  9 02:45 mysql
drwx------ 2 mysql mysql       4096 Dec  9 02:49 testdb
drwx------ 2 mysql mysql       4096 Dec  9 02:49 keydb
drwx------ 2 mysql mysql       4096 Dec  9 03:08 performance_schema
drwx------ 2 mysql mysql       4096 Dec  9 02:52 functiondb
drwx------ 2 mysql mysql       4096 Dec  9 02:52 test
-rw-r----- 1 mysql mysql         25 Dec  9 03:08 xtrabackup_binlog_pos_innodb
-rw-r----- 1 mysql mysql        710 Dec  9 03:08 xtrabackup_info
[root@slave02 mysql]# ll -d /data/mysql/
drwxr-xr-x 16 mysql mysql 20480 Dec  9 03:08 /data/mysql/
[root@slave02 mysql]# 
{% endhighlight %}


---

##启动mysql并且开启同步

{% highlight bash %}
[root@slave02 mysql]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.27-75.0-log Percona Server (GPL), Release 75.0, Revision
8bb53b6

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show slave status\G
Empty set (0.00 sec)

mysql> CHANGE MASTER TO
MASTER_HOST='192.168.66.100',MASTER_USER='replication',MASTER_PASSWORD='xxxxxxxxxx',MASTER_LOG_FILE='mysql-bin.000004',MASTER_LOG_POS=8299670;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.66.100
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 8299670
               Relay_Log_File: relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,functiondb,testdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 8299670
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

mysql> start slave ;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.66.100
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 13985464
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 972
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,functiondb,testdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 8300359
              Relay_Log_Space: 5686244
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 8800
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 10
                  Master_UUID: a6f64eac-9442-11e5-944e-44a842184bd8
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

mysql> 
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.66.100
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 14031757
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 5732370
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,functiondb,testdb
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 14031757
              Relay_Log_Space: 5732537
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
             Master_Server_Id: 10
                  Master_UUID: a6f64eac-9442-11e5-944e-44a842184bd8
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the
slave I/O thread to update it
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

mysql> \! hostname
slave02
mysql> 
{% endhighlight %}

---

##重建mha

---



#命令汇总

* **`yum clean all`**
* **`yum install nfs-utils rpcbind`**
* **`showmount  -e new-master`**
* **`mount -t nfs -o intr new-master:/data/nfs /data/nfs/`**
* **`df -h | grep nfs`**
* **`yum -y install  keepalived.x86_64`**
* **`vim keepalived.conf`**
* **`vim /etc/sysconfig/iptables`**
* **`grep 18 /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables  reload`**
* **`iptables -L -nv  | grep 224`**
* **`/etc/init.d/keepalived start`**
* **`cat /etc/keepalived/keepalived.conf`**
* **`masterha_check_status --conf=/etc/app1.cnf`**
* **`masterha_stop --conf=/etc/app1.cnf`**
* **`/etc/init.d/keepalived reload ; watch -n .2 ip a`**
* **`time nohup /usr/bin/innobackupex --defaults-file=/etc/my.cnf --user=root --password=xxxxxxxxxx /data/nfs/test_full_backup    >>  /data/nfs/full_backup.log 2>&1  &`**
* **`tail -f full_backup.log`**
* **`rm -rf *`**
* **`rpm -e Percona-Server-client-51-5.1.73-rel14.11.603.rhel6.x86_64 Percona-Server-server-51-5.1.73-rel14.11.603.rhel6.x86_64 Percona-Server-shared-51-5.1.73-rel14.11.603.rhel6.x86_64   --nodeps`**
* **`wget http://mirror.centos.org/centos/6/os/x86_64/Packages/numactl-2.0.9-2.el6.x86_64.rpm`**
* **`rpm -ivh  numactl-2.0.9-2.el6.x86_64.rpm`**
* **`rpm -ivh  Percona-Server-client-56-5.6.27-rel75.0.el6.x86_64.rpm Percona-Server-server-56-5.6.27-rel75.0.el6.x86_64.rpm Percona-Server-shared-56-5.6.27-rel75.0.el6.x86_64.rpm`**
* **`mv my.cnf my.old.2015.12.09.backup`**
* **`diff /tmp/old /tmp/new`**
* **`id zabbix`**
* **`/etc/init.d/zabbix-agent restart`**
* **`zabbix_get -s new-master -p 10050 -k "mysql.slowlog[100,/var/lib/mysql/new-master-slow.log]"`**
* **`vim  /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh`**
* **`grep 120  /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh`**
* **`yum install libev`**
* **`rpm -e xtrabackup-1.6.7-356.rhel6.x86_64`**
* **`rpm -ivh  percona-xtrabackup-2.3.2-1.el6.x86_64.rpm`**
* **`innobackupex --version`**
* **`time nohup /usr/bin/innobackupex --apply-log /data/nfs/test_full_backup/2015-12-09_00-53-03/`**
* **`time nohup /usr/bin/innobackupex --copy-back /data/nfs/test_full_backup/2015-12-09_00-53-03/ >> restore.log  2>&1   &`**
* **`tail -f restore.log`**
* **`watch -n 2 du -sh /data/mysql/`**
* **`cat xtrabackup_binlog_pos_innodb`**
* **`chown  -R mysql.mysql /var/lib/mysql/`**
