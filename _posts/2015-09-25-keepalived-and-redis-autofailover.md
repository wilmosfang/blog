---
layout: post
title: Keepalived 实现 Redis AutoFailover (RedisHA)
author: wilmosfang
tags:  redis keepalived nosql
categories:   redis
wc: 1030 3223 37050
excerpt: follow me
comments: true
---

---

# 前言

**[Redis][redis]** 是一个开源的, BSD 许可的, key-value 缓存和存储

关于它的HA，目前有三种方式:

* [Sentinel][sentinel]
* [Redis Cluster][cluster]
* Keepalived + Redis


前两种都是官方的HA方式，它们各有利弊:

[Sentinel][sentinel]使用一个守护进程对正在运行的[Redis][redis]实例进行监控和管理，可以有效实现分布式监控与投票，但是failover过后，新的master对于client来说不可知，需要问sentinel，所以这种机制需要client端的逻辑支持

[Redis Cluster][cluster]非常完整高效地构建了一个分布式集群，[Redis][redis]实例间相互监控和投票，自举与故障切换，官方建议至少需要三个节点，然后最好再各自带一个slave，于是六个节点就是起步价(否则集群很脆弱，很容易进入失效状态)，节点要使用cluster模式启动，并在分配数据之初就提前构建好集群关系，所以这种机制需要重新部署，并且为了避免多次收到MOVED转向导致的开消，也需要client端的逻辑优化与支持

Keepalived + Redis 的实现方式并非官方HA方案，在监控与失效切换方面也并不显得比上面两种更加智能(靠自定义脚本实现，相对来说low很多)，正是因为它足够传统足够老旧，所以它有一个上面两种都不具备的特性，就是对于客户端几乎是透明的，不必作任何修改，对当前正在运行的redis实例也不必作任何修改。


这里分享一下 **Keepalived + Redis** 的配置方法

> **Tip:** 当前版本 **Redis 3.0.4**

---

# 概要

* TOC
{:toc}


---

## 下载redis源码包


~~~
[root@temp src]# wget http://download.redis.io/releases/redis-3.0.4.tar.gz
--2015-09-24 14:21:10--  http://download.redis.io/releases/redis-3.0.4.tar.gz
Resolving download.redis.io... 109.74.203.151
Connecting to download.redis.io|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1364993 (1.3M) [application/x-gzip]
Saving to: “redis-3.0.4.tar.gz”

100%[============================================================================================>] 1,364,993    180K/s   in 8.6s    

2015-09-24 14:21:20 (155 KB/s) - “redis-3.0.4.tar.gz” saved [1364993/1364993]

[root@temp src]# ls
redis-3.0.4.tar.gz
[root@temp src]# 
~~~

---

## 安装redis


解压redis源码包

~~~
[root@temp src]# tar -zxvf redis-3.0.4.tar.gz 
redis-3.0.4/
redis-3.0.4/.gitignore
redis-3.0.4/00-RELEASENOTES
redis-3.0.4/BUGS
redis-3.0.4/CONTRIBUTING
redis-3.0.4/COPYING
redis-3.0.4/INSTALL
redis-3.0.4/MANIFESTO
...
...
redis-3.0.4/utils/redis-copy.rb
redis-3.0.4/utils/redis-sha1.rb
redis-3.0.4/utils/redis_init_script
redis-3.0.4/utils/redis_init_script.tpl
redis-3.0.4/utils/speed-regression.tcl
redis-3.0.4/utils/whatisdoing.sh
[root@temp src]# ls
redis-3.0.4  redis-3.0.4.tar.gz
[root@temp src]# 
~~~


### 报错一


~~~
[root@temp redis-3.0.4]# pwd
/usr/local/src/redis-3.0.4
[root@temp redis-3.0.4]# make 
cd src && make all
make[1]: Entering directory `/usr/local/src/redis-3.0.4/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-dump redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html
(cd ../deps && make distclean)
make[2]: Entering directory `/usr/local/src/redis-3.0.4/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[2]: Leaving directory `/usr/local/src/redis-3.0.4/deps'
(rm -f .make-*)
echo STD=-std=c99 -pedantic >> .make-settings
echo WARN=-Wall -W >> .make-settings
echo OPT=-O2 >> .make-settings
echo MALLOC=jemalloc >> .make-settings
echo CFLAGS= >> .make-settings
echo LDFLAGS= >> .make-settings
echo REDIS_CFLAGS= >> .make-settings
echo REDIS_LDFLAGS= >> .make-settings
echo PREV_FINAL_CFLAGS=-std=c99 -pedantic -Wall -W -O2 -g -ggdb   -I../deps/hiredis -I../deps/linenoise -I../deps/lua/src -DUSE_JEMALLOC -I../deps/jemalloc/include >> .make-settings
echo PREV_FINAL_LDFLAGS=  -g -ggdb -rdynamic >> .make-settings
(cd ../deps && make hiredis linenoise lua jemalloc)
make[2]: Entering directory `/usr/local/src/redis-3.0.4/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
(echo "" > .make-ldflags)
(echo "" > .make-cflags)
MAKE hiredis
cd hiredis && make static
make[3]: Entering directory `/usr/local/src/redis-3.0.4/deps/hiredis'
gcc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
make[3]: gcc: Command not found
make[3]: *** [net.o] Error 127
make[3]: Leaving directory `/usr/local/src/redis-3.0.4/deps/hiredis'
make[2]: *** [hiredis] Error 2
make[2]: Leaving directory `/usr/local/src/redis-3.0.4/deps'
make[1]: [persist-settings] Error 2 (ignored)
    CC adlist.o
/bin/sh: cc: command not found
make[1]: *** [adlist.o] Error 127
make[1]: Leaving directory `/usr/local/src/redis-3.0.4/src'
make: *** [all] Error 2
[root@temp redis-3.0.4]# echo $?
2
[root@temp redis-3.0.4]#
~~~

**make[3]: gcc: Command not found**  此问题是没有安装 **gcc** 导致的

解决办法安装 **gcc**

~~~
[root@temp redis-3.0.4]# rpm -qa | grep gcc 
libgcc-4.4.7-11.el6.x86_64
[root@temp redis-3.0.4]# yum -y install gcc 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Determining fastest mirrors
 * extras: mirrors.opencas.cn
 * updates: mirror.bit.edu.cn
base                                                                                                           | 4.0 kB     00:00 ... 
extras                                                                                                         | 3.4 kB     00:00     
extras/primary_db                                                                                              |  32 kB     00:00     
percona-release-noarch                                                                                         |  951 B     00:00     
percona-release-noarch/primary                                                                                 | 5.0 kB     00:00     
percona-release-noarch                                                                                                          33/33
percona-release-x86_64                                                                                         |  951 B     00:00     
percona-release-x86_64/primary                                                                                 | 181 kB     00:01     
percona-release-x86_64                                                                                                        611/611
updates                                                                                                        | 3.4 kB     00:00     
updates/primary_db                                                                                             | 1.9 MB     00:01     
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.4.7-11.el6 will be installed
--> Processing Dependency: cpp = 4.4.7-11.el6 for package: gcc-4.4.7-11.el6.x86_64
...
...
---> Package mpfr.x86_64 0:2.4.1-6.el6 will be installed
---> Package ppl.x86_64 0:0.10.2-11.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                         Arch                         Version                                Repository                  Size
======================================================================================================================================
Installing:
 gcc                             x86_64                       4.4.7-11.el6                           base                        10 M
Installing for dependencies:
 cloog-ppl                       x86_64                       0.15.7-1.2.el6                         base                        93 k
 cpp                             x86_64                       4.4.7-11.el6                           base                       3.7 M
 mpfr                            x86_64                       2.4.1-6.el6                            base                       157 k
 ppl                             x86_64                       0.10.2-11.el6                          base                       1.3 M

Transaction Summary
======================================================================================================================================
Install       5 Package(s)

Total download size: 15 M
Installed size: 33 M
Downloading Packages:
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                  13 MB/s |  15 MB     00:01     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded

...
...

Installed:
  gcc.x86_64 0:4.4.7-11.el6                                                                                                           

Dependency Installed:
  cloog-ppl.x86_64 0:0.15.7-1.2.el6      cpp.x86_64 0:4.4.7-11.el6      mpfr.x86_64 0:2.4.1-6.el6      ppl.x86_64 0:0.10.2-11.el6     

Complete!
[root@temp redis-3.0.4]# 
~~~

---

### 报错二

~~~
[root@temp redis-3.0.4]# pwd
/usr/local/src/redis-3.0.4
[root@temp redis-3.0.4]# make 
cd src && make all
make[1]: Entering directory `/usr/local/src/redis-3.0.4/src'
    CC adlist.o
In file included from adlist.c:34:
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/usr/local/src/redis-3.0.4/src'
make: *** [all] Error 2
[root@temp redis-3.0.4]# echo $?
2
[root@temp redis-3.0.4]#
~~~

此问题是 **Redis** 默认会使用 **jemalloc** , 因为 **jemalloc** 被证明比 **libc** 有更少的碎片问题，但是我的系统中没有 **jemalloc** ，所以会出错

下面是来自源码中 **README** 的申明

~~~
Allocator
---------

Selecting a non-default memory allocator when building Redis is done by setting
the `MALLOC` environment variable. Redis is compiled and linked against libc
malloc by default, with the exception of jemalloc being the default on Linux
systems. This default was picked because jemalloc has proven to have fewer
fragmentation problems than libc malloc.

To force compiling against libc malloc, use:

    % make MALLOC=libc

To compile against jemalloc on Mac OS X systems, use:

    % make MALLOC=jemalloc
~~~

解决办法是手动指定系统中已有的 **libc**

~~~
[root@temp redis-3.0.4]# make MALLOC=libc
cd src && make all
make[1]: Entering directory `/usr/local/src/redis-3.0.4/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-dump redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html
(cd ../deps && make distclean)
make[2]: Entering directory `/usr/local/src/redis-3.0.4/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[2]: Leaving directory `/usr/local/src/redis-3.0.4/deps'
(rm -f .make-*)
echo STD=-std=c99 -pedantic >> .make-settings
echo WARN=-Wall -W >> .make-settings
echo OPT=-O2 >> .make-settings
echo MALLOC=libc >> .make-settings
echo CFLAGS= >> .make-settings
echo LDFLAGS= >> .make-settings
echo REDIS_CFLAGS= >> .make-settings
echo REDIS_LDFLAGS= >> .make-settings
echo PREV_FINAL_CFLAGS=-std=c99 -pedantic -Wall -W -O2 -g -ggdb   -I../deps/hiredis -I../deps/linenoise -I../deps/lua/src >> .make-settings
echo PREV_FINAL_LDFLAGS=  -g -ggdb -rdynamic >> .make-settings
(cd ../deps && make hiredis linenoise lua)
make[2]: Entering directory `/usr/local/src/redis-3.0.4/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
(echo "" > .make-ldflags)
(echo "" > .make-cflags)
MAKE hiredis
cd hiredis && make static
make[3]: Entering directory `/usr/local/src/redis-3.0.4/deps/hiredis'
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  hiredis.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  sds.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  async.c
ar rcs libhiredis.a net.o hiredis.o sds.o async.o
make[3]: Leaving directory `/usr/local/src/redis-3.0.4/deps/hiredis'
MAKE linenoise
cd linenoise && make
make[3]: Entering directory `/usr/local/src/redis-3.0.4/deps/linenoise'
cc  -Wall -Os -g  -c linenoise.c
make[3]: Leaving directory `/usr/local/src/redis-3.0.4/deps/linenoise'
MAKE lua
cd lua/src && make all CFLAGS="-O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL " MYLDFLAGS="" AR="ar rcu"
make[3]: Entering directory `/usr/local/src/redis-3.0.4/deps/lua/src'
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lapi.o lapi.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lcode.o lcode.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ldebug.o ldebug.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ldo.o ldo.c
ldo.c: In function ‘f_parser’:
ldo.c:496: warning: unused variable ‘c’
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ldump.o ldump.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lfunc.o lfunc.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lgc.o lgc.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o llex.o llex.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lmem.o lmem.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lobject.o lobject.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lopcodes.o lopcodes.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lparser.o lparser.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lstate.o lstate.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lstring.o lstring.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ltable.o ltable.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ltm.o ltm.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lundump.o lundump.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lvm.o lvm.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lzio.o lzio.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o strbuf.o strbuf.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o fpconv.o fpconv.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lauxlib.o lauxlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lbaselib.o lbaselib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ldblib.o ldblib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o liolib.o liolib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lmathlib.o lmathlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o loslib.o loslib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o ltablib.o ltablib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lstrlib.o lstrlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o loadlib.o loadlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o linit.o linit.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lua_cjson.o lua_cjson.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lua_struct.o lua_struct.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lua_cmsgpack.o lua_cmsgpack.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lua_bit.o lua_bit.c
ar rcu liblua.a lapi.o lcode.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o ltm.o lundump.o lvm.o lzio.o strbuf.o fpconv.o lauxlib.o lbaselib.o ldblib.o liolib.o lmathlib.o loslib.o ltablib.o lstrlib.o loadlib.o linit.o lua_cjson.o lua_struct.o lua_cmsgpack.o lua_bit.o	# DLL needs all object files
ranlib liblua.a
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o lua.o lua.c
cc -o lua  lua.o liblua.a -lm 
liblua.a(loslib.o): In function `os_tmpname':
loslib.c:(.text+0x35): warning: the use of `tmpnam' is dangerous, better use `mkstemp'
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o luac.o luac.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL    -c -o print.o print.c
cc -o luac  luac.o print.o liblua.a -lm 
make[3]: Leaving directory `/usr/local/src/redis-3.0.4/deps/lua/src'
make[2]: Leaving directory `/usr/local/src/redis-3.0.4/deps'
    CC adlist.o
    CC ae.o
    CC anet.o
anet.c: In function ‘anetSockName’:
anet.c:623: warning: dereferencing pointer ‘s’ does break strict-aliasing rules
anet.c:621: note: initialized from here
anet.c:627: warning: dereferencing pointer ‘s’ does break strict-aliasing rules
anet.c:625: note: initialized from here
anet.c: In function ‘anetPeerToString’:
anet.c:584: warning: dereferencing pointer ‘s’ does break strict-aliasing rules
anet.c:582: note: initialized from here
anet.c:588: warning: dereferencing pointer ‘s’ does break strict-aliasing rules
anet.c:586: note: initialized from here
anet.c: In function ‘anetTcpAccept’:
anet.c:555: warning: dereferencing pointer ‘s’ does break strict-aliasing rules
anet.c:553: note: initialized from here
anet.c:559: warning: dereferencing pointer ‘s’ does break strict-aliasing rules
anet.c:557: note: initialized from here
    CC dict.o
    CC redis.o
    CC sds.o
    CC zmalloc.o
    CC lzf_c.o
    CC lzf_d.o
    CC pqsort.o
    CC zipmap.o
    CC sha1.o
    CC ziplist.o
    CC release.o
    CC networking.o
    CC util.o
    CC object.o
    CC db.o
db.c: In function ‘scanGenericCommand’:
db.c:432: warning: ‘pat’ may be used uninitialized in this function
db.c:433: warning: ‘patlen’ may be used uninitialized in this function
    CC replication.o
    CC rdb.o
    CC t_string.o
    CC t_list.o
    CC t_set.o
    CC t_zset.o
    CC t_hash.o
    CC config.o
    CC aof.o
    CC pubsub.o
    CC multi.o
    CC debug.o
    CC sort.o
    CC intset.o
    CC syncio.o
    CC cluster.o
    CC crc16.o
    CC endianconv.o
    CC slowlog.o
    CC scripting.o
    CC bio.o
    CC rio.o
    CC rand.o
    CC memtest.o
    CC crc64.o
    CC bitops.o
    CC sentinel.o
    CC notify.o
    CC setproctitle.o
    CC blocked.o
    CC hyperloglog.o
    CC latency.o
    CC sparkline.o
    LINK redis-server
    INSTALL redis-sentinel
    CC redis-cli.o
    LINK redis-cli
    CC redis-benchmark.o
    LINK redis-benchmark
    CC redis-check-dump.o
    LINK redis-check-dump
    CC redis-check-aof.o
    LINK redis-check-aof

Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory `/usr/local/src/redis-3.0.4/src'
[root@temp redis-3.0.4]# echo $?
0
[root@temp redis-3.0.4]# 
~~~

---

## 运行redis

修改配置并运行 **redis**

~~~
[root@temp redis-3.0.4]# mkdir /etc/redis
[root@temp redis-3.0.4]# cp redis.conf  /etc/redis/redis.conf
BUGS             COPYING       INSTALL  MANIFESTO  redis.conf  runtest-cluster  sentinel.conf     tests
[root@temp redis-3.0.4]# cd /etc/redis/
[root@temp redis]# vim redis.conf 
[root@temp redis]# mv redis.conf  redis6379.conf 
[root@temp redis]# cd /usr/local/src/redis-3.0.4
[root@temp redis-3.0.4]# grep -v "^#" /etc/redis/redis6379.conf  | grep -v "^$"
daemonize yes
pidfile /data/redis/redis6379.pid
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 0
loglevel notice
logfile "/data/redis/redis6379.log"
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump6379.rdb
dir /data/redis
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly6379.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
[root@temp redis-3.0.4]# src/redis-server /etc/redis/redis6379.conf 
[root@temp redis-3.0.4]# ps faux | grep redis 
root      6989  0.0  0.0 103252   828 pts/0    S+   16:02   0:00  |       \_ grep redis
root      6985  0.4  0.0 127840  1912 ?        Ssl  16:01   0:00 src/redis-server *:6379                   
[root@temp redis-3.0.4]# 
~~~

---


## 安装keepalived

~~~
[root@temp ~]# yum -y install  keepalived.x86_64   
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * extras: mirrors.opencas.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package keepalived.x86_64 0:1.2.13-4.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                          Arch                         Version                             Repository                  Size
====================================================================================================================================
Installing:
 keepalived                       x86_64                       1.2.13-4.el6                        base                       214 k

Transaction Summary
====================================================================================================================================
Install       1 Package(s)

Total download size: 214 k
Installed size: 625 k
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : keepalived-1.2.13-4.el6.x86_64                                                                                   1/1 
  Verifying  : keepalived-1.2.13-4.el6.x86_64                                                                                   1/1 

Installed:
  keepalived.x86_64 0:1.2.13-4.el6                                                                                                  

Complete!
[root@temp ~]# 
~~~

## 修改/etc/hosts



~~~
[root@redis-a ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.100.201  redis-a
192.168.100.202  redis-b
[root@redis-a ~]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=redis-a.temp
[root@redis-a ~]# 
[root@redis-b ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.100.201  redis-a
192.168.100.202  redis-b
[root@redis-b ~]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=redis-b.temp
[root@redis-b ~]# 
~~~


## 故障检查与切换脚本

### redis_check.sh

redis状态检查脚本，两个节点上的一样

~~~
[root@redis-a scripts]# cat /etc/keepalived/scripts/redis_check.sh 
#!/bin/bash


ALIVE=`/usr/local/bin/redis-cli PING`
if [ "$ALIVE" == "PONG" ] ; then 
 echo $ALIVE
 exit 0 
else 
 echo $ALIVE
 exit 1
fi
[root@redis-a scripts]# 
~~~

---

### redis_master.sh

master状态切换脚本，两个节点上一样

~~~
[root@redis-a scripts]# cat /etc/keepalived/scripts/redis_master.sh 
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$

echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[master] read change to master" >> $LOGFILE
echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[master] wait 10 sec for data sync from old master" >> $LOGFILE
sleep 10
echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[master] data rsync from old mater ok..." >> $LOGFILE
echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[master] Run slaveof no one,close master/slave" >> $LOGFILE
$REDISCLI SLAVEOF NO ONE >> $LOGFILE 2>&1
echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[master] wait other slave connect...." >> $LOGFILE
[root@redis-a scripts]# 
~~~

---

### redis_backup.sh

backup状态切换脚本，不同之处在于互指对方为master

~~~
[root@redis-a scripts]# cat /etc/keepalived/scripts/redis_backup.sh 
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$



echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] being backup status" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] ready to sync with master" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] Run 'SLAVEOF redis-b  6379'" >> $LOGFILE
#$REDISCLI SLAVEOF redis-b  6379 >> $LOGFILE  2>&1
echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] wait other connect...." >> $LOGFILE
[root@redis-a scripts]#
[root@redis-b scripts]# cat /etc/keepalived/scripts/redis_backup.sh 
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$



echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] being backup status" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] ready to sync with master" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] Run 'SLAVEOF redis-a  6379'" >> $LOGFILE
#$REDISCLI SLAVEOF redis-a  6379 >> $LOGFILE  2>&1
echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[backup] wait other connect...." >> $LOGFILE
[root@redis-b scripts]# 
~~~

> **Note:** 我注释掉了同步代码，因为生产中，在没了解实例的当前内存使用状况，服务器实际负载的状况下，贸然自动同步，会对服务器造成很大压力，对其它应用也会有很大影响，所以这一步由人工来确认，在此只作日志记录


> **Tip:**测试环境中 ，或轻量实例的生产环境中，可以将以上注释去掉


---

### redis_fault.sh

fault状态切换脚本，不同之处在于互指对方为master

~~~

[root@redis-a scripts]# cat /etc/keepalived/scripts/redis_fault.sh 
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$



echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] being fault status" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] ready to sync with master" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] Run 'SLAVEOF redis-b  6379'" >> $LOGFILE
#$REDISCLI SLAVEOF redis-b  6379 >> $LOGFILE  2>&1
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] wait other connect...." >> $LOGFILE
[root@redis-a scripts]# 
[root@redis-b scripts]# cat /etc/keepalived/scripts/redis_fault.sh 
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$



echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] being fault status" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] ready to sync with master" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] Run 'SLAVEOF redis-a  6379'" >> $LOGFILE
#$REDISCLI SLAVEOF redis-a  6379 >> $LOGFILE  2>&1
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[fault] wait other connect...." >> $LOGFILE
[root@redis-b scripts]# 
~~~

> **Note:** 我注释掉了同步代码，原因同上


---

### redis_stop.sh

stop状态切换脚本，不同之处在于互指对方为master


~~~
[root@redis-a scripts]# cat /etc/keepalived/scripts/redis_stop.sh 
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$



echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] being slave status" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] ready to sync with master" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] Run 'SLAVEOF redis-b  6379'" >> $LOGFILE
#$REDISCLI SLAVEOF redis-b  6379 >> $LOGFILE  2>&1
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] wait other  connect...." >> $LOGFILE
[root@redis-a scripts]# 
[root@redis-b scripts]# cat /etc/keepalived/scripts/redis_stop.sh
#!/bin/bash 

REDISCLI="/usr/local/bin/redis-cli"
LOGFILE="/data/redis/keepalived-redis-state.log"
pid=$$



echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] being slave status" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] ready to sync with master" >> $LOGFILE
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] Run 'SLAVEOF redis-a  6379'" >> $LOGFILE
#$REDISCLI SLAVEOF redis-a  6379 >> $LOGFILE  2>&1
#echo "`date +'%Y-%m-%d:%H:%M:%S'`|$pid|state:[stop] wait other  connect...." >> $LOGFILE
[root@redis-b scripts]#
~~~


---

## keepalived.conf 

keepalived 配置

~~~
[root@redis-a keepalived]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   router_id LVS_redis-a
}

vrrp_script chk_redis {
        script "/etc/keepalived/scripts/redis_check.sh"  
        weight  -10
        interval 2                                      
}



vrrp_instance VI_123 {
    state BACKUP
    interface eth2
    virtual_router_id 123
    priority 138
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
            chk_redis                    
    }


    virtual_ipaddress {
	192.168.100.123
    }
        notify_master /etc/keepalived/scripts/redis_master.sh
        notify_backup /etc/keepalived/scripts/redis_backup.sh
        notify_fault  /etc/keepalived/scripts/redis_fault.sh
        notify_stop   /etc/keepalived/scripts/redis_stop.sh

}
[root@redis-a keepalived]# 
[root@redis-b keepalived]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   router_id LVS_redis-b
}

vrrp_script chk_redis {
        script "/etc/keepalived/scripts/redis_check.sh"  
        weight  -10
        interval 2                                      
}



vrrp_instance VI_123 {
    state BACKUP
    interface eth2
    virtual_router_id 123
    priority 139
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
            chk_redis
    }
    virtual_ipaddress {
	192.168.100.123
    }
        notify_master /etc/keepalived/scripts/redis_master.sh
        notify_backup /etc/keepalived/scripts/redis_backup.sh
        notify_fault  /etc/keepalived/scripts/redis_fault.sh
        notify_stop   /etc/keepalived/scripts/redis_stop.sh

}
[root@redis-b keepalived]# 
~~~

下面是它的区别

~~~
4c4
<    router_id LVS_redis-b
---
>    router_id LVS_redis-a
19c19
<     priority 139
---
>     priority 138
~~~

priority 的范围是 1-255

> **Note:**  priority 值高的会自动变成 master , 所以在分配优先级时master 要设置一个相对高一点的值，但不能比 backup 高过weight 调整值，否则keepalived检查发现服务不可用了，将相应的priority 减小了之后，发现还是比backup的优先级高，于是还会继续充当master ，不放开 VIP，这时客户端访问就不会获得期望的结果


---

## 启动顺序

这个redis+keepalived 集群的启动顺序相当有讲究，否则会出意外


1. 启动充当master的redis (或者生产中正好有一台正在运行的redis)
2. 启动充当slave的redis
3. 在slave上执行 SLAVEOF 与master进行同步 (选择一个业务低峰点)
4. 调整master上的keepalived priority 使它的值  s.priority < m.priority < s.priority+weight (是为了master被keepalived检查并认定失效后，slave可以通过自已的优先级成功竞选成为新的master)
5. 启动master上的keepalived
6. 启动slave上的keepliaved


> **Note:** 这种方式只可以抗击一次master的非计划故障切换，或计划性切换，如果要再次使用，得重新手动按照上面顺序进行构建，之所以不自动化，是因为，SLAVEOF是一个有杀伤性的命令，正常情况会对服务器造成显著压力，意外情况会毁掉master上的数据，比如和一个空的redis进行同步，将导致自己的数据被清掉


---

## 一次模拟切换

### 当前状态


redis-b 是master 

~~~
[root@redis-b keepalived]# redis-cli info replication 
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.100.201,port=6379,state=online,offset=8107,lag=0
master_repl_offset:8107
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:8106
[root@redis-b keepalived]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:ab:e8:95 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.185/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:feab:e895/64 scope link 
       valid_lft forever preferred_lft forever
3: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:ab:e8:9f brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.202/24 brd 192.168.100.255 scope global eth2
    inet 192.168.100.123/32 scope global eth2
    inet6 fe80::20c:29ff:feab:e89f/64 scope link 
       valid_lft forever preferred_lft forever
[root@redis-b keepalived]# 
~~~

redis-a 是slave

~~~
[root@redis-a keepalived]# redis-cli info replication 
# Replication
role:slave
master_host:redis-b
master_port:6379
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:8093
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
[root@redis-a keepalived]# 
~~~


### 模拟故障发生

~~~
[root@redis-b keepalived]# redis-cli shutdown 
[root@redis-b keepalived]# redis-cli info replication
Could not connect to Redis at 127.0.0.1:6379: Connection refused
[root@redis-b keepalived]# 
~~~

### 发生自动切换

redis-a 的日志中产生了数据

~~~
2015-09-25:00:53:36|25366|state:[master] read change to master
2015-09-25:00:53:36|25366|state:[master] wait 10 sec for data sync from old master
2015-09-25:00:53:46|25366|state:[master] data rsync from old mater ok...
2015-09-25:00:53:46|25366|state:[master] Run slaveof no one,close master/slave
OK
2015-09-25:00:53:46|25366|state:[master] wait other slave connect....
~~~

redis-a 自动升为master

~~~
[root@redis-a keepalived]# redis-cli info replication 
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
[root@redis-a keepalived]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:6e:13:48 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.184/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe6e:1348/64 scope link 
       valid_lft forever preferred_lft forever
3: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:6e:13:52 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.201/24 brd 192.168.100.255 scope global eth2
    inet 192.168.100.123/32 scope global eth2
    inet6 fe80::20c:29ff:fe6e:1352/64 scope link 
       valid_lft forever preferred_lft forever
[root@redis-a keepalived]# 
~~~

---


[redis]:http://redis.io/
[sentinel]:http://redis.io/topics/sentinel
[cluster]:http://redis.io/topics/cluster-tutorial
