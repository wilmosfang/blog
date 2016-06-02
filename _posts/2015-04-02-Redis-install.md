---
layout: post
title: Redis 安装
author: wilmosfang
tags:  redis nosql
categories:  redis nosql
wc: 418  1391 13516
excerpt: redis 的下载，解压，编译，安装，运行，连接
comments: true
---


# 前言



[Redis][redis] 是一个开源的, BSD 许可的, key-value 缓存或存储. 它也被称作数据结构服务器，因为它可以包含 strings, hashes, lists, sets, sorted sets, bitmaps 和 hyperloglogs这些数据结构。

现在比较火热的几款网络应用都使用到了redis，如：

**Twitter  GitHub    Weibo    Pinterest    Snapchat    Craigslist    Digg    StackOverflow    Flickr**

这里有张[列表][caselist]，对以上网络应用作了描述了，并且也列出了一些其它使用到redis的比较受欢迎的网站

总体来说，redis对现在零碎的网络信息可以进行比较高效快捷的处理和存储，如评论，微博之类的碎片化信息

> **Tip:** 当前的最新版是 **Redis Stable (3.0)**


---


# 概要

* TOC
{:toc}


---

Redis 是日前我遇到过的安装最简单的数据库

## 下载安装包

[Redis的安装包地址][redis download] 

当前的最新版是**Stable (3.0)** 

> **Tip:** Redis uses a standard practice for its versioning: major.minor.patchlevel. An even minor marks a stable release, like 1.2, 2.0, 2.2, 2.4, 2.6, 2.8. Odd minors are used for unstable releases, for example 2.9.x releases are the unstable versions of what will be Redis 3.0 once stable.

~~~
[root@m1 ~]# wget http://download.redis.io/releases/redis-3.0.0.tar.gz
--2015-04-02 16:46:56--  http://download.redis.io/releases/redis-3.0.0.tar.gz
Resolving download.redis.io... 109.74.203.151
Connecting to download.redis.io|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1358081 (1.3M) [application/x-gzip]
Saving to: “redis-3.0.0.tar.gz”
...
...
2015-04-02 16:48:25 (15.0 KB/s) - “redis-3.0.0.tar.gz” saved [1358081/1358081]
 
[root@m1 ~]# ls
anaconda-ks.cfg  Documents  install.log         log    Pictures  redis-3.0.0.tar.gz  tmp
Desktop          Downloads  install.log.syslog  Music  Public    Templates           Videos
[root@m1 ~]# 
~~~

## 解压并编译

~~~
[root@m1 ~]# tar zxvf redis-3.0.0.tar.gz 
redis-3.0.0/
redis-3.0.0/.gitignore
redis-3.0.0/00-RELEASENOTES
redis-3.0.0/BUGS
redis-3.0.0/CONTRIBUTING
...
...
redis-3.0.0/utils/redis_init_script
redis-3.0.0/utils/redis_init_script.tpl
redis-3.0.0/utils/speed-regression.tcl
redis-3.0.0/utils/whatisdoing.sh
[root@m1 ~]# 
[root@m1 ~]# cd redis-3.0.0
[root@m1 redis-3.0.0]# ls
00-RELEASENOTES  COPYING  Makefile   redis.conf       runtest-sentinel  tests
BUGS             deps     MANIFESTO  runtest          sentinel.conf     utils
CONTRIBUTING     INSTALL  README     runtest-cluster  src
[root@m1 redis-3.0.0]# make 
cd src && make all
make[1]: Entering directory `/root/redis-3.0.0/src'
...
...
    CC redis-check-dump.o
    LINK redis-check-dump
    CC redis-check-aof.o
    LINK redis-check-aof

Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory `/root/redis-3.0.0/src'
[root@m1 redis-3.0.0]# echo $?
0
[root@m1 redis-3.0.0]# 
~~~

## 拷贝脚本


src下会多出几个脚本

~~~
[root@m1 redis-3.0.0]# cd src/
[root@m1 src]# ls
adlist.c     config.h       lzf_c.c          rdb.c               rio.c           t_hash.c
adlist.h     config.o       lzf_c.o          rdb.h               rio.h           t_hash.o
adlist.o     crc16.c        lzf_d.c          rdb.o               rio.o           t_list.c
ae.c         crc16.o        lzf_d.o          redisassert.h       scripting.c     t_list.o
ae_epoll.c   crc64.c        lzf.h            redis-benchmark     scripting.o     t_set.c
ae_evport.c  crc64.h        lzfP.h           redis-benchmark.c   sds.c           t_set.o
ae.h         crc64.o        Makefile         redis-benchmark.o   sds.h           t_string.c
ae_kqueue.c  db.c           Makefile.dep     redis.c             sds.o           t_string.o
ae.o         db.o           memtest.c        redis-check-aof     sentinel.c      t_zset.c
ae_select.c  debug.c        memtest.o        redis-check-aof.c   sentinel.o      t_zset.o
anet.c       debug.o        mkreleasehdr.sh  redis-check-aof.o   setproctitle.c  util.c
anet.h       dict.c         multi.c          redis-check-dump    setproctitle.o  util.h
anet.o       dict.h         multi.o          redis-check-dump.c  sha1.c          util.o
aof.c        dict.o         networking.c     redis-check-dump.o  sha1.h          valgrind.sup
aof.o        endianconv.c   networking.o     redis-cli           sha1.o          version.h
asciilogo.h  endianconv.h   notify.c         redis-cli.c         slowlog.c       ziplist.c
bio.c        endianconv.o   notify.o         redis-cli.o         slowlog.h       ziplist.h
bio.h        fmacros.h      object.c         redis.h             slowlog.o       ziplist.o
bio.o        help.h         object.o         redis.o             solarisfixes.h  zipmap.c
bitops.c     hyperloglog.c  pqsort.c         redis-sentinel      sort.c          zipmap.h
bitops.o     hyperloglog.o  pqsort.h         redis-server        sort.o          zipmap.o
blocked.c    intset.c       pqsort.o         redis-trib.rb       sparkline.c     zmalloc.c
blocked.o    intset.h       pubsub.c         release.c           sparkline.h     zmalloc.h
cluster.c    intset.o       pubsub.o         release.h           sparkline.o     zmalloc.o
cluster.h    latency.c      rand.c           release.o           syncio.c
cluster.o    latency.h      rand.h           replication.c       syncio.o
config.c     latency.o      rand.o           replication.o       testhelp.h
[root@m1 src]#  ls  | grep -v "\."
Makefile
redis-benchmark
redis-check-aof
redis-check-dump
redis-cli
redis-sentinel
redis-server
[root@m1 src]# 
~~~

将 **redis-server** 和 **redis-cli** 拷贝到 **/usr/local/bin/** 以便使用

~~~
[root@m1 src]# cp redis-server  /usr/local/bin/
[root@m1 src]# cp redis-cli /usr/local/bin/
~~~


> **Tip:** **redis-3.0.0**中有两个重要文件 **redis.conf** 和 **README** 
> 
>  **redis.conf** 是redis 的配置模板
>
>  **README** 可以用来参考作为快速入门
>
>  >[root@m1 redis-3.0.0]# ls
>  >
>  >00-RELEASENOTES  COPYING  Makefile   redis.conf       runtest-sentinel  tests
>  >
>  >BUGS             deps     MANIFESTO  runtest          sentinel.conf     utils
>  >
>  >CONTRIBUTING     INSTALL  README     runtest-cluster  src
>  >
>  >[root@m1 redis-3.0.0]# 

## 运行redis server


**redis-server** 的使用方法 

~~~
[root@m1 src]# ./redis-server  --help 
Usage: ./redis-server [/path/to/redis.conf] [options]
       ./redis-server - (read config from stdin)
       ./redis-server -v or --version
       ./redis-server -h or --help
       ./redis-server --test-memory <megabytes>

Examples:
       ./redis-server (run the server with default conf)
       ./redis-server /etc/redis/6379.conf
       ./redis-server --port 7777
       ./redis-server --port 7777 --slaveof 127.0.0.1 8888
       ./redis-server /etc/myredis.conf --loglevel verbose

Sentinel mode:
       ./redis-server /etc/sentinel.conf --sentinel
[root@m1 src]# ./redis-server  --version
Redis server v=3.0.0 sha=00000000:0 malloc=jemalloc-3.6.0 bits=64 build=2af66cb8b0604f9c
[root@m1 src]# 
~~~
redis-server 不加配置文件时是以默认配置运行

~~~
[root@m1 ~]# redis-server  
8551:C 02 Apr 20:27:32.302 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
8551:M 02 Apr 20:27:32.304 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.0 (00000000/0) 64 bit
  .-`` .-~~~.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 8551
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

8551:M 02 Apr 20:27:32.307 # Server started, Redis version 3.0.0
8551:M 02 Apr 20:27:32.314 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
8551:M 02 Apr 20:27:32.315 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
8551:M 02 Apr 20:27:32.315 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
8551:M 02 Apr 20:27:32.315 * The server is now ready to accept connections on port 6379
8551:M 02 Apr 21:27:33.008 * 1 changes in 3600 seconds. Saving...
8551:M 02 Apr 21:27:33.012 * Background saving started by pid 8822
8822:C 02 Apr 21:27:33.033 * DB saved on disk
8822:C 02 Apr 21:27:33.033 * RDB: 6 MB of memory used by copy-on-write
~~~

redis服务默认运行在 **6379** 端口

~~~
[root@m1 redis-3.0.0]# ps -ef | grep redis
root      8551  4915  0 20:27 pts/0    00:00:11 redis-server *:6379
root      8991  8562  0 22:17 pts/1    00:00:00 grep redis
[root@m1 redis-3.0.0]# netstat -an | grep 6379
tcp        0      0 0.0.0.0:6379                0.0.0.0:*                   LISTEN      
tcp        0      0 :::6379                     :::*                        LISTEN      
[root@m1 redis-3.0.0]# 
~~~

## 连入redis server

可以使用 **redis-cli** 连接 redis server , 然后完成各种操作

~~~
[root@m1 ~]# redis-cli 
127.0.0.1:6379> set abc xiaoge 
OK
127.0.0.1:6379> get abc 
"xiaoge"
127.0.0.1:6379> append abc jkjk
(integer) 10
127.0.0.1:6379> get abc 
"xiaogejkjk"
127.0.0.1:6379> lpush list perl
(integer) 1
127.0.0.1:6379> lget list
(error) ERR unknown command 'lget'
127.0.0.1:6379> lrange list 0 -1
1) "perl"
127.0.0.1:6379> lpush list python
(integer) 2
127.0.0.1:6379> lpush list ruby 
(integer) 3
127.0.0.1:6379> lpush list c
(integer) 4
127.0.0.1:6379> lpush list c++
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "c++"
2) "c"
3) "ruby"
4) "python"
5) "perl"
127.0.0.1:6379> lpush list R
(integer) 6
127.0.0.1:6379> lpush list Basic
(integer) 7
127.0.0.1:6379> lrange list 0 -1
1) "Basic"
2) "R"
3) "c++"
4) "c"
5) "ruby"
6) "python"
7) "perl"
127.0.0.1:6379> sadd s apple 
(integer) 1
127.0.0.1:6379> sadd s orange
(integer) 1
127.0.0.1:6379> sadd s oppo
(integer) 1
127.0.0.1:6379> sadd s strawbrew
(integer) 1
127.0.0.1:6379> smembers s 
1) "oppo"
2) "orange"
3) "apple"
4) "strawbrew"
127.0.0.1:6379> zadd b 10 webshell
(integer) 1
127.0.0.1:6379> zadd b  1  commend
(integer) 1
127.0.0.1:6379> zadd b  4 attac
(integer) 1
127.0.0.1:6379> zadd b 8 op
(integer) 1
127.0.0.1:6379> zadd b 9 pp
(integer) 1
127.0.0.1:6379> zadd b 3 oo
(integer) 1
127.0.0.1:6379> zrange 0 -1
(error) ERR wrong number of arguments for 'zrange' command
127.0.0.1:6379> zrange b 0 -1
1) "commend"
2) "oo"
3) "attac"
4) "op"
5) "pp"
6) "webshell"
127.0.0.1:6379> hset h name wilmos
(integer) 1
127.0.0.1:6379> hset h city shanghai
(integer) 1
127.0.0.1:6379> hset h gender female
(integer) 1
127.0.0.1:6379>
~~~

当然，还有一种方法，界面比较不友好，但也可以对redis server进行操作，使用 **telnet**

~~~
[root@m1 ~]# telnet localhost 6379
Trying ::1...
Connected to localhost.
Escape character is '^]'.


set a b 
+OK
get a 
$1
b
set c d 
+OK
get c 
$1
d
hgetall h 
*6
$4
name
$6
wilmos
$4
city
$8
shanghai
$6
gender
$6
female
zrange b 0 -1
*6
$7
commend
$2
oo
$5
attac
$2
op
$2
pp
$8
webshell
~~~


使用这个[**链接**][redis try]，可以对redis的基本操作进行学习

---

# 命令汇总

* **`wget http://download.redis.io/releases/redis-3.0.0.tar.gz`**
* **`tar zxvf redis-3.0.0.tar.gz`**
* **`cd redis-3.0.0`**
* **`make`**
* **`cd src/`**
* **`ls  | grep -v "\."`**
* **`cp redis-server  /usr/local/bin/`**
* **`cp redis-cli /usr/local/bin/`**
* **`./redis-server  --help`**
* **`./redis-server  --version`**
* **`telnet localhost 6379`**




---

[redis]:http://redis.io/
[redis try]:http://try.redis.io/
[redis download]:http://redis.io/download
[caselist]:http://techstacks.io/tech/redis



