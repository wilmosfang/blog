---
layout: post
title: SSDB基础
author: wilmosfang
tags:  nosql ssdb cluster
categories:  ssdb
wc: 696 2140 20109
excerpt: ssdb 下载安装，启动停止，前台后台运行，客户端连接，启动脚本，master-master 集群，内存占用
comments: true
---



# 前言


**[ssdb][ssdb]** 是一个高性能支持丰富数据结构的 NoSQL 数据库, 用于替代 Redis.

它与Redis相比具备明显优势的一个特性就是基于硬盘存储，所以它支撑的数据量不会因为内存的限制而受约束。

至于性能，由于存取效率一部分取决于硬盘的IOPS，所以[官网][ssdb]给的benchmark结果应该是极高IOPS的硬盘(比如固态硬盘)上测试出的结果，有人在正常环境下做过测试，实际效率是redis的42%~57%，CPU负载要高出一倍，并发效率也与redis有明显差距，详细可以参考 [redis和ssdb读取性能对比][ssdb_vs_redis]

虽然SSDB效率上不如REDIS,但基于磁盘存储有着其最大的优势,毕竟很多业务数据远超过服务器内存的容量，即便不超过，相较硬盘，内存也要贵很多，所以时间与空间的取舍就取决于具体的应用场景了

下面分享一下 **[ssdb][ssdb]** 的基本操作，详细可以参阅 [官方文档][ssdb_doc]


 > **Tip:** 目前官方版本是 **SSDB 1.9.2** 
  

---


# 概要

* TOC
{:toc}



---

## 下载和安装


使用下面的方法进行安装

~~~
wget --no-check-certificate https://github.com/ideawu/ssdb/archive/master.zip
unzip master
cd ssdb-master
make
sudo make install
~~~

安装过程

~~~
[root@h101 src]# wget --no-check-certificate https://github.com/ideawu/ssdb/archive/master.zip
--2015-11-24 19:36:34--  https://github.com/ideawu/ssdb/archive/master.zip
Resolving github.com... 192.30.252.130
Connecting to github.com|192.30.252.130|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/ideawu/ssdb/zip/master [following]
--2015-11-24 19:36:35--  https://codeload.github.com/ideawu/ssdb/zip/master
Resolving codeload.github.com... 192.30.252.147
Connecting to codeload.github.com|192.30.252.147|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/zip]
Saving to: “master.zip”

    [                                                                                                     <=>

2015-11-24 19:37:19 (33.7 KB/s) - “master.zip” saved [1460288]

[root@h101 src]# ll master.zip 
-rw-r--r-- 1 root root 1460288 Nov 24 19:37 master.zip
[root@h101 src]# 
[root@h101 src]# unzip master.zip 
Archive:  master.zip
9497637f54da00f364e310de42000284545cfebe
   creating: ssdb-master/
  inflating: ssdb-master/.gitignore  
  inflating: ssdb-master/ChangeLog   
  inflating: ssdb-master/Dockerfile  
  inflating: ssdb-master/LICENSE     
  inflating: ssdb-master/Makefile    
  inflating: ssdb-master/README.md 
...
...
   creating: ssdb-master/tools/ssdb_cli/
  inflating: ssdb-master/tools/ssdb_cli/cluster.cpy  
  inflating: ssdb-master/tools/ssdb_cli/exporter.cpy  
  inflating: ssdb-master/tools/ssdb_cli/flushdb.cpy  
  inflating: ssdb-master/tools/ssdb_cli/importer.cpy  
  inflating: ssdb-master/tools/ssdb_cli/util.cpy  
  inflating: ssdb-master/tools/unittest.php  
 extracting: ssdb-master/version     
[root@h101 src]# 
[root@h101 src]# ll ssdb-master/
total 4912
drwxr-xr-x 6 root root    4096 Nov 23 20:34 api
-rw-r--r-- 1 root root     671 Nov 24 19:41 build_config.mk
-rwxr-xr-x 1 root root    3468 Nov 23 20:34 build.sh
-rw-r--r-- 1 root root    4831 Nov 23 20:34 ChangeLog
drwxr-xr-x 6 root root    4096 Nov 23 20:34 deps
-rw-r--r-- 1 root root     971 Nov 23 20:34 Dockerfile
drwxr-xr-x 2 root root    4096 Nov 23 20:34 docs
-rw-r--r-- 1 root root    1470 Nov 23 20:34 LICENSE
-rw-r--r-- 1 root root    1573 Nov 23 20:34 Makefile
-rw-r--r-- 1 root root    4412 Nov 23 20:34 README.md
drwxr-xr-x 6 root root    4096 Nov 24 19:41 src
-rwxr-xr-x 1 root root     946 Nov 23 20:34 ssdb.conf
-rwxr-xr-x 1 root root 4950089 Nov 24 19:40 ssdb-server
-rwxr-xr-x 1 root root     705 Nov 23 20:34 ssdb_slave.conf
drwxr-xr-x 3 root root    4096 Nov 24 19:40 tools
drwxr-xr-x 2 root root    4096 Nov 24 19:40 var
drwxr-xr-x 2 root root    4096 Nov 24 19:40 var_slave
-rw-r--r-- 1 root root       6 Nov 23 20:34 version
[root@h101 src]# 
[root@h101 src]# cd ssdb-master/
[root@h101 ssdb-master]# make 

##### building snappy... #####
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
...
...
g++ -o ssdb-repair ssdb-repair.o ../src/net/link.o ../src/net/fde.o ../src/util/log.o ../src/util/bytes.o   --1.18/libleveldb.a" "/usr/local/src/ssdb-master/deps/snappy-1.1.0/.libs/libsnappy.a" "/usr/local/src/ssdb-mas
g++ -o leveldb-import leveldb-import.o ../src/net/link.o ../src/net/fde.o ../src/util/log.o ../src/util/byteseveldb-1.18/libleveldb.a" "/usr/local/src/ssdb-master/deps/snappy-1.1.0/.libs/libsnappy.a" "/usr/local/src/ss
g++ -o ssdb-migrate ssdb-migrate.o ../api/cpp/libssdb-client.a ../src/util/libutil.a
make[1]: Leaving directory `/usr/local/src/ssdb-master/tools'
[root@h101 ssdb-master]# echo $?
0
[root@h101 ssdb-master]# make install 
mkdir -p /usr/local/ssdb
mkdir -p /usr/local/ssdb/_cpy_
mkdir -p /usr/local/ssdb/deps
mkdir -p /usr/local/ssdb/var
mkdir -p /usr/local/ssdb/var_slave
cp -f ssdb-server ssdb.conf ssdb_slave.conf /usr/local/ssdb
cp -rf api /usr/local/ssdb
cp -rf \
		tools/ssdb-bench \
		tools/ssdb-cli tools/ssdb_cli \
		tools/ssdb-cli.cpy tools/ssdb-dump \
		tools/ssdb-repair \
		/usr/local/ssdb
cp -rf deps/cpy /usr/local/ssdb/deps
chmod 755 /usr/local/ssdb
chmod -R ugo+rw /usr/local/ssdb/*
rm -f /usr/local/ssdb/Makefile
[root@h101 ssdb-master]#
~~~

默认将安装在 **/usr/local/ssdb** 目录下

~~~
[root@h101 ssdb-master]# ll /usr/local/ssdb/
total 17848
drwxrwxrwx 6 root root    4096 Nov 24 19:41 api
drwxrwxrwx 2 root root    4096 Nov 24 19:41 _cpy_
drwxrwxrwx 3 root root    4096 Nov 24 19:41 deps
-rwxrwxrwx 1 root root 4186529 Nov 24 19:41 ssdb-bench
drwxrwxrwx 2 root root    4096 Nov 24 19:41 ssdb_cli
-rwxrwxrwx 1 root root     149 Nov 24 19:41 ssdb-cli
-rwxrwxrwx 1 root root   12560 Nov 24 19:41 ssdb-cli.cpy
-rwxrwxrwx 1 root root     946 Nov 24 19:41 ssdb.conf
-rwxrwxrwx 1 root root 4531101 Nov 24 19:41 ssdb-dump
-rwxrwxrwx 1 root root 4546317 Nov 24 19:41 ssdb-repair
-rwxrwxrwx 1 root root 4950089 Nov 24 19:41 ssdb-server
-rwxrwxrwx 1 root root     705 Nov 24 19:41 ssdb_slave.conf
drwxrwxrwx 2 root root    4096 Nov 24 19:41 var
drwxrwxrwx 2 root root    4096 Nov 24 19:41 var_slave
[root@h101 ssdb-master]# 
~~~ 

---

## 启动和停止 

### 前台运行

这种模式下会占用当前的terminal

~~~
[root@h101 ssdb]# ./ssdb-server  ssdb.conf 
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io


~~~

查看进程状态

~~~
[root@h101 ssdb]# ps faux | grep ssdb | grep -v grep 
root     10734  0.4  1.8 184860 34508 pts/1    Sl+  20:29   0:07          \_ ./ssdb-server ssdb.conf
[root@h101 ssdb]# 
~~~

查看日志

~~~
[root@h101 ssdb]# cat log.txt 
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(46): ssdb-server 1.9.2
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(47): conf_file        : ssdb.conf
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(48): log_level        : debug
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(49): log_output       : log.txt
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(50): log_rotate_size  : 1000000000
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(52): main_db          : ./var/data
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(53): meta_db          : ./var/meta
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(54): cache_size       : 500 MB
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(55): block_size       : 32 KB
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(56): write_buffer     : 64 MB
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(57): max_open_files   : 500
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(58): compaction_speed : 1000 MB/s
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(59): compression      : yes
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(60): binlog           : yes
2015-11-24 20:29:21.692 [INFO ] ssdb-server.cpp(61): sync_speed       : -1 MB/s
2015-11-24 20:29:21.719 [INFO ] binlog.cpp(175): binlogs capacity: 20000000, min: 0, max: 0,
2015-11-24 20:29:21.729 [INFO ] server.cpp(159): server listen on 127.0.0.1:8888
2015-11-24 20:29:21.729 [INFO ] server.cpp(169): auth: off
2015-11-24 20:29:21.730 [DEBUG] cluster.cpp(13): Cluster init
2015-11-24 20:29:21.730 [INFO ] serv.cpp(329): key_range.kv: "", ""
2015-11-24 20:29:21.730 [INFO ] ssdb-server.cpp(84): pidfile: ./var/ssdb.pid, pid: 10734
2015-11-24 20:29:21.730 [INFO ] ssdb-server.cpp(85): ssdb server started.
2015-11-24 20:29:21.731 [DEBUG] ttl.cpp(112): load 0 keys into fast_keys
2015-11-24 20:29:21.732 [DEBUG] worker.cpp(17): writer 0 init
2015-11-24 20:29:21.732 [DEBUG] worker.cpp(17): reader 0 init
2015-11-24 20:29:21.733 [DEBUG] worker.cpp(17): reader 1 init
2015-11-24 20:29:21.741 [DEBUG] worker.cpp(17): reader 2 init
2015-11-24 20:29:21.742 [DEBUG] worker.cpp(17): reader 3 init
2015-11-24 20:29:21.758 [DEBUG] worker.cpp(17): reader 4 init
2015-11-24 20:29:21.775 [DEBUG] worker.cpp(17): reader 5 init
2015-11-24 20:29:21.794 [DEBUG] worker.cpp(17): reader 6 init
2015-11-24 20:29:21.810 [DEBUG] worker.cpp(17): reader 7 init
2015-11-24 20:29:21.810 [DEBUG] worker.cpp(17): reader 9 init
2015-11-24 20:29:21.811 [DEBUG] worker.cpp(17): reader 8 init
2015-11-24 20:34:22.630 [INFO ] server.cpp(203): server running, links: 0
2015-11-24 20:39:22.630 [INFO ] server.cpp(203): server running, links: 0
2015-11-24 20:44:22.630 [INFO ] server.cpp(203): server running, links: 0
2015-11-24 20:49:22.630 [INFO ] server.cpp(203): server running, links: 0
2015-11-24 20:54:22.630 [INFO ] server.cpp(203): server running, links: 0
2015-11-24 20:59:22.630 [INFO ] server.cpp(203): server running, links: 0
[root@h101 ssdb]# 
~~~

查看线程

~~~
[root@h101 ssdb]# pstree -a 10734
ssdb-server ssdb.conf
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  ├─{ssdb-server}
  └─{ssdb-server}
[root@h101 ssdb]# ps -Lf 10734 
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root     10734 10446 10734  0   14 20:29 pts/1    Sl+    0:01 ./ssdb-server ssdb.conf
root     10734 10446 10735  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10736  0   14 20:29 pts/1    Sl+    0:06 ./ssdb-server ssdb.conf
root     10734 10446 10737  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10738  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10739  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10740  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10741  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10742  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10743  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10744  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10745  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10746  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
root     10734 10446 10747  0   14 20:29 pts/1    Sl+    0:00 ./ssdb-server ssdb.conf
[root@h101 ssdb]# 
~~~

---

### 停止服务

~~~
[root@h101 ssdb]# ./ssdb-server  ssdb.conf  -s stop 
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io

[root@h101 ssdb]# ps faux | grep ssdb | grep -v grep 
[root@h101 ssdb]# 
~~~

---

### 后台运行

这种方式不会占用当前terminal，关闭当前terminal也不会导致服务退出

~~~
[root@h101 ssdb]# ./ssdb-server  -d ssdb.conf 
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io

[root@h101 ssdb]# ps faux | grep ssdb | grep -v grep 
root     10890  1.7  1.8 184860 36164 ?        Ssl  21:05   0:00 ./ssdb-server -d ssdb.conf
[root@h101 ssdb]# pstree 10890
ssdb-server───13*[{ssdb-server}]
[root@h101 ssdb]# 
~~~

---

### 客户端连接


~~~
[root@h101 ssdb]# ./ssdb-cli -p 8888
ssdb (cli) - ssdb command line tool.
Copyright (c) 2012-2015 ssdb.io

'h' or 'help' for help, 'q' to quit.

ssdb-server 1.9.2

ssdb 127.0.0.1:8888> info 
version
	1.9.2
links
	1
total_calls
	1
dbsize
	0
binlogs
	    capacity : 20000000
	    min_seq  : 0
	    max_seq  : 0
serv_key_range
	    kv  : "" - ""
	    hash: "" - ""
	    zset: "" - ""
	    list: "" - ""
data_key_range
	    kv  : "" - ""
	    hash: "" - ""
	    zset: "" - ""
	    list: "" - ""
leveldb.stats
	                               Compactions
	Level  Files Size(MB) Time(sec) Read(MB) Write(MB)
	--------------------------------------------------
	
17 result(s) (0.001 sec)
ssdb 127.0.0.1:8888> 
~~~


---

### 发送信号终止服务

~~~
[root@h101 ssdb]# ps faux | grep ssdb | grep -v grep 
root     10890  0.4  1.8 184860 36164 ?        Ssl  21:05   0:01 ./ssdb-server -d ssdb.conf
[root@h101 ssdb]# cat var/ssdb.pid 
10890[root@h101 ssdb]# kill `cat ./var/ssdb.pid`
[root@h101 ssdb]# ps faux | grep ssdb | grep -v grep 
[root@h101 ssdb]# cat var/ssdb.pid
cat: var/ssdb.pid: No such file or directory
[root@h101 ssdb]#  
~~~

---

### SSDB 启动脚本

修改 **configs** 配置，使其指向正确的位置

~~~
[root@h101 tools]# cp /usr/local/src/ssdb-master/tools/ssdb.sh  /etc/init.d/ssdb
[root@h101 tools]# vim ssdb.sh 
[root@h101 tools]# grep configs=  /etc/init.d/ssdb
# configs="/data/ssdb_data/test/ssdb.conf /data/ssdb_data/test2/ssdb.conf"
#configs="/data/ssdb_data/test/ssdb.conf"
configs="/usr/local/ssdb/ssdb.conf"
[root@h101 tools]# 
~~~

使用脚本启动服务

~~~
[root@h101 ~]# /etc/init.d/ssdb  start 
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io

[root@h101 ~]# ps faux | grep ssdb | grep -v grep 
root      2795  2.6  1.8 184864 36180 ?        Ssl  09:41   0:00 /usr/local/ssdb/ssdb-server /usr/local/ssdb/ssdb.conf -s restart -d
[root@h101 ~]# /etc/init.d/ssdb  stop 
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io

[root@h101 ~]# ps faux | grep ssdb | grep -v grep 
[root@h101 ~]# 
~~~

SSDB相关配置可以参阅[SSDB配置][ssdb_config]

SSDB相关命令可以参阅[SSDB命令][ssdb_command]


---

## master-master

config on h101

~~~
[root@h101 ~]# grep -v "#" /etc/ssdb/ssdb1234.conf 

work_dir = /data/ssdb/ssdb1234
pidfile = /data/ssdb/ssdb1234/ssdb.pid

server:
	port: 1234
	ip: 0.0.0.0
	deny: all
	allow: 127.0.0.1
	allow: 192.168

replication:
	binlog: yes
	sync_speed: -1
	slaveof:
		id: svc_h102_1
		type: mirror
		host: h102
		port: 1234

logger:
	level: debug
	output: /data/ssdb/ssdb1234/log.txt
	rotate:
		size: 1000000000

leveldb:
	cache_size: 500
	block_size: 32
	write_buffer_size: 64
	compaction_speed: 1000
	compression: yes


[root@h101 ~]# 
~~~

config on h102

~~~
[root@h102 ssdb]# grep -v "#" /etc/ssdb/ssdb1234.conf 
work_dir = /data/ssdb/ssdb1234
pidfile = /data/ssdb/ssdb1234/ssdb.pid

server:
	port: 1234
	ip: 0.0.0.0
	deny: all
	allow: 127.0.0.1
	allow: 192.168

replication:
	binlog: yes
	sync_speed: -1
	slaveof:
		id: svc_h101_1
		type: mirror
		host: h101
		port: 1234

logger:
	level: debug
	output: /data/ssdb/ssdb1234/log.txt
	rotate:
		size: 1000000000

leveldb:
	cache_size: 500
	block_size: 32
	write_buffer_size: 64
	compaction_speed: 1000
	compression: yes
[root@h102 ssdb]# 
~~~


**Note:** 注意以下几点


*  1  . 注意防火墙的影响
*  2  . **slaveof.id** 是 **master的id** ，这是从 slave角度来看的, 不要在 master 上配置它自己的id.
*  3  . 对于新版本(1.9.2+), **slaveof.ip** 改成了 **slaveof.host** 用来指定 master 的主机名(域名)
*  4  . type设定为 **mirror** ，默认情况下是sync



---

分别启动服务

~~~
[root@h101 ~]# /usr/local/ssdb/ssdb-server -d /etc/ssdb/ssdb1234.conf
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io

[root@h101 ~]# 
----------
[root@h102 ~]# /usr/local/ssdb/ssdb-server -d /etc/ssdb/ssdb1234.conf
ssdb-server 1.9.2
Copyright (c) 2012-2015 ssdb.io

[root@h102 ~]#
~~~

分别查看状态


~~~
ssdb h101:1234> info 
version
	1.9.2
links
	1
total_calls
	10
dbsize
	240
binlogs
	    capacity : 20000000
	    min_seq  : 1
	    max_seq  : 7
replication
	client 192.168.100.102:38671
	    type     : mirror
	    status   : SYNC
	    last_seq : 7
replication
	slaveof h102:1234
	    id         : svc_h102_1
	    type       : mirror
	    status     : SYNC
	    last_seq   : 9
	    copy_count : 0
	    sync_count : 4
serv_key_range
	    kv  : "" - ""
	    hash: "" - ""
	    zset: "" - ""
	    list: "" - ""
data_key_range
	    kv  : "1" - "z"
	    hash: "" - ""
	    zset: "" - ""
	    list: "" - ""
leveldb.stats
	                               Compactions
	Level  Files Size(MB) Time(sec) Read(MB) Write(MB)
	--------------------------------------------------
	  0        1        0         0        0         0
	
21 result(s) (0.001 sec)
ssdb h101:1234> 
----------
ssdb h102:1234> info 
version
	1.9.2
links
	1
total_calls
	12
dbsize
	234
binlogs
	    capacity : 20000000
	    min_seq  : 3
	    max_seq  : 9
replication
	client 192.168.100.101:43355
	    type     : mirror
	    status   : SYNC
	    last_seq : 9
replication
	slaveof h101:1234
	    id         : svc_h101_1
	    type       : mirror
	    status     : SYNC
	    last_seq   : 7
	    copy_count : 0
	    sync_count : 1
serv_key_range
	    kv  : "" - ""
	    hash: "" - ""
	    zset: "" - ""
	    list: "" - ""
data_key_range
	    kv  : "1" - "z"
	    hash: "" - ""
	    zset: "" - ""
	    list: "" - ""
leveldb.stats
	                               Compactions
	Level  Files Size(MB) Time(sec) Read(MB) Write(MB)
	--------------------------------------------------
	  1        1        0         0        0         0
	
21 result(s) (0.001 sec)
ssdb h102:1234> 
~~~


详细可以参考[同步和复制的配置与监控][ssdb_replication]

---

## 限制

Item     | Comment
-------- | ---
最大 Key 长度	|200 字节
最大 Value 长度	|31MB
最大请求或响应长度	|31MB
单个 HASH 中的元素数量	|9,223,372,036,854,775,807
单个 ZSET 中的元素数量	|9,223,372,036,854,775,807
单个 QUEUE 中的元素数量	|9,223,372,036,854,775,807
命令最多参数个数	|所有参数加起来体积不超过 31MB 大小


### 内存占用

一个 ssdb-server 实例占用的内存瞬时(有可能, 而且即使达到, 也只是持续短时间)最高达到(MB):

对于压缩选项没有开启的情况

~~~
cache_size + write_buffer_size * 66 + 32
~~~

如果压缩选项开启 (compression: yes) , 计算公式是:

~~~
cache_size + 10 * write_buffer_size * 66 + 32
~~~

可以调整配置参数, 限制 ssdb-server 的内存占用.


---

# 命令汇总

* **`wget --no-check-certificate https://github.com/ideawu/ssdb/archive/master.zip`**
* **`unzip master.zip`**
* **`cd ssdb-master/`**
* **`make`**
* **`make install`**
* **`./ssdb-server  ssdb.conf`**
* **`ps faux | grep ssdb | grep -v grep`**
* **`cat log.txt`**
* **`pstree -a 10734`**
* **`ps -Lf 10734`**
* **`./ssdb-server  ssdb.conf  -s stop`**
* **`./ssdb-server  -d ssdb.conf`**
* **`ps faux | grep ssdb | grep -v grep`**
* **`./ssdb-cli -p 8888`**
* **`vim ssdb.sh`**
* **`grep configs=  /etc/init.d/ssdb`**
* **`/etc/init.d/ssdb  start`**
* **`/etc/init.d/ssdb  stop`**
* **`grep -v "#" /etc/ssdb/ssdb1234.conf`**
* **`/usr/local/ssdb/ssdb-server -d /etc/ssdb/ssdb1234.conf`**

---

[ssdb]:http://ssdb.io/zh_cn/
[ssdb_doc]:http://ssdb.io/docs/zh_cn/index.html
[ssdb_vs_redis]:http://www.cnblogs.com/smark/p/3909291.html
[ssdb_config]:http://ssdb.io/docs/zh_cn/config.html
[ssdb_command]:http://ssdb.io/docs/zh_cn/commands/index.html
[ssdb_replication]:http://ssdb.io/docs/zh_cn/replication.html

