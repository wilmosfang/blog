---
layout: post
title:   Redis 状态信息详解
author: wilmosfang
tags:   nosql redis
categories:   redis 
wc: 574 1357 17442
excerpt:  Redis 的下载，解压，安装，启动，连接，info的用法，Redis 状态信息的详解 
comments: true
---


# 前言


**[Redis][redis]** 是一个遵循 BSD 许可的开源软件，是一个内存型数据结构存储，可以用作为数据库，缓存和消息中间件

在 **[Redis][redis]** 的运维管理过程中，会需要了解系统当前的状态

> **Tip:** **[Redis][redis]** 相关基础，可以参考之前的一篇博客 **[Redis 基础][redis_install]**

这里分享一下 **[Redis][redis]** 状态信息的详细解释，详细可以参考 **[INFO [section]][info]**

> **Tip:**   目前最新版本为 **Redis 3.2.1** 



---


# 概要

* TOC
{:toc}


---

## 环境

~~~
[root@h102 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 ~]# uname  -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 ~]# 
~~~


---

## 下载解压并编译

~~~
[root@h102 redis]# wget http://download.redis.io/releases/redis-3.2.1.tar.gz
--2016-06-23 16:46:16--  http://download.redis.io/releases/redis-3.2.1.tar.gz
Resolving download.redis.io... 109.74.203.151
Connecting to download.redis.io|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1534696 (1.5M) [application/x-gzip]
Saving to: “redis-3.2.1.tar.gz”

100%[============================================================================================>] 1,534,696   12.5K/s   in 92s     

2016-06-23 16:47:48 (16.4 KB/s) - “redis-3.2.1.tar.gz” saved [1534696/1534696]

[root@h102 redis]# ls
redis-3.2.1.tar.gz
[root@h102 redis]# tar -zxvf redis-3.2.1.tar.gz 
redis-3.2.1/
redis-3.2.1/.gitignore
redis-3.2.1/00-RELEASENOTES
redis-3.2.1/BUGS
redis-3.2.1/CONTRIBUTING
redis-3.2.1/COPYING
redis-3.2.1/INSTALL
...
...
redis-3.2.1/utils/releasetools/
redis-3.2.1/utils/releasetools/01_create_tarball.sh
redis-3.2.1/utils/releasetools/02_upload_tarball.sh
redis-3.2.1/utils/releasetools/03_test_release.sh
redis-3.2.1/utils/releasetools/04_release_hash.sh
redis-3.2.1/utils/speed-regression.tcl
redis-3.2.1/utils/whatisdoing.sh
[root@h102 redis]# cd redis-3.2.1
[root@h102 redis-3.2.1]# make 
cd src && make all
make[1]: Entering directory `/usr/local/src/redis/redis-3.2.1/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html
(cd ../deps && make distclean)
make[2]: Entering directory `/usr/local/src/redis/redis-3.2.1/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd geohash-int && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[2]: Leaving directory `/usr/local/src/redis/redis-3.2.1/deps'
(rm -f .make-*)
...
...
    CC redis-cli.o
    LINK redis-cli
    CC redis-benchmark.o
    LINK redis-benchmark
    INSTALL redis-check-rdb
    CC redis-check-aof.o
    LINK redis-check-aof

Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory `/usr/local/src/redis/redis-3.2.1/src'
[root@h102 redis-3.2.1]# echo $?
0
[root@h102 redis-3.2.1]# 
~~~

查看生成的可执行文件，并且验证版本

~~~
[root@h102 redis-3.2.1]# cd src/
[root@h102 src]# ls  | grep -v "\."
Makefile
redis-benchmark
redis-check-aof
redis-check-rdb
redis-cli
redis-sentinel
redis-server
[root@h102 src]# ./redis-server --help
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
[root@h102 src]# ./redis-server --version
Redis server v=3.2.1 sha=00000000:0 malloc=jemalloc-4.0.3 bits=64 build=588e5be7450e0a73
[root@h102 src]#
~~~


---

## 启动

不加参数直接启动服务，会前台运行

~~~
[root@h102 src]# redis-server 
36305:C 23 Jun 17:00:28.456 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
36305:M 23 Jun 17:00:28.457 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.2.1 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 36305
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

36305:M 23 Jun 17:00:28.459 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
36305:M 23 Jun 17:00:28.467 # Server started, Redis version 3.2.1
36305:M 23 Jun 17:00:28.468 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
36305:M 23 Jun 17:00:28.468 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
36305:M 23 Jun 17:00:28.468 * The server is now ready to accept connections on port 6379
~~~


---

## 连接

使用客户端连接，

~~~
[root@h102 ~]# redis-cli -p 6379
127.0.0.1:6379>
~~~

---

## info

info 命令会反馈出服务的统计信息

并且是以分组的形式进行展现

~~~
[root@h102 ~]# redis-cli -p 6379
127.0.0.1:6379> info
# Server
redis_version:3.2.1
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:588e5be7450e0a73
redis_mode:standalone
os:Linux 2.6.32-504.el6.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.4.7
process_id:36305
run_id:3235dccf9b8bdcbdb0abaee1b9250d9393f23e6b
tcp_port:6379
uptime_in_seconds:37
uptime_in_days:0
hz:10
lru_clock:7054673
executable:/usr/local/src/redis/redis-3.2.1/src/redis-server
config_file:

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:822280
used_memory_human:803.01K
used_memory_rss:8040448
used_memory_rss_human:7.67M
used_memory_peak:822280
used_memory_peak_human:803.01K
total_system_memory:1960378368
total_system_memory_human:1.83G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:9.78
mem_allocator:jemalloc-4.0.3

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1466672428
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:1
total_commands_processed:1
instantaneous_ops_per_sec:0
total_net_input_bytes:31
total_net_output_bytes:5889902
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.09
used_cpu_user:0.02
used_cpu_sys_children:0.00
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
127.0.0.1:6379> 
~~~


> **Tip:** 新版客户端有一个小改进，添加了命令提示

![redis1.png](/images/redis/redis1.png)

![redis2.png](/images/redis/redis2.png)


---

## info 用法


info 命令有如下几种用法

Usage| Comment
-------- | ---
**info [section]**| 定向显示一组信息
**info** | 显示所有默认信息
**info default**| 同上
**info all**| 显示所有(全部)信息



---


## 信息详解

反馈信息包含以下几个分组


Section | Comment
-------- | ---
**Server**| Redis 服务的基础信息
**Clients**| 客户端连接信息
**Memory**| 内存开销相关信息
**Persistence**| 持久化相关信息
**Stats**| 基础统计信息
**Replication**| 主备复制信息
**CPU**| CPU 开销相关信息
**Cluster**| 集群相关信息
**Keyspace**| 数据库相关统计
**Commandstats**| redis命令相关统计


除了信息分组，其它的都是各分组中的属性与值，遵循 **field:value** 的格式

### Server section

Property| Value
-------- | ---
**redis_version**| redis 服务的版本
**redis_git_sha1**|  Git SHA1 hash 值 
**redis_git_dirty**| Git dirty flag
**redis_build_id**|? 构建ID 
**redis_mode**| 运行模式
**os**| 宿主机内核版本
**arch_bits**| 架构(32位 64位)
**multiplexing_api**| 事件驱动的模式
**gcc_version**| 用来编译源码的gcc版本
**process_id**| 实例的进程号
**run_id**| 一个随机产生的值，用于服务在集群中的标识
**tcp_port**| TCP/IP 的监听端口
**uptime_in_seconds**| 实例连续运行的时间(秒数)
**uptime_in_days**| 实例连续运行的时间(天数)
**hz**|? 
**lru_clock**| 用于LRU管理的自增时钟
**executable**| 可执行程序的完整路径
**config_file**| 配置文件路径



---

### Clients section


Property| Value
-------- | ---
**connected_clients**| 排除了 slave 连接后的所有客户端连接数
**client_longest_output_list**|？ longest output list among current client connections 
**client_biggest_input_buf**| 所有当前客户端中最大的输入缓冲大小
**blocked_clients**| 被阻塞的客户端数量


---

### Memory section

Property| Value
-------- | ---
**used_memory**| 为Redis 分配的内存总量，单位为 byte
**used_memory_human**| 与 **used_memory** 相同，只是以更便于阅读的方式进行展示
**used_memory_rss**| OS分配的常驻内存大小，单位为 byte
**used_memory_rss_human**| 与 **used_memory_rss** 相同，只是以更便于阅读的方式进行展示
**used_memory_peak**| Redis 曾使用过的内存峰值
**used_memory_peak_human**| 与 **used_memory_peak** 相同，只是以更便于阅读的方式进行展示
**total_system_memory**| 系统总内存，单位为 byte
**total_system_memory_human**| 与 **total_system_memory** 相同，只是以更便于阅读的方式进行展示
**used_memory_lua**| lua 引擎占用的内存大小
**used_memory_lua_human**| 与 **used_memory_lua** 相同，只是以更便于阅读的方式进行展示
**maxmemory**|  ?
**maxmemory_human**| ?
**maxmemory_policy**| ?
**mem_fragmentation_ratio**| 内存碎片率，是 **used_memory_rss** 与  **used_memory** 的比值
**mem_allocator**| 内存的分配方法

理想情况下 **used_memory_rss** 只会比 **used_memory** 大一点点

* 当 rss >> used 时，代表有大量的内存碎片
* 当 used >> rss 时，代表有大量的内存被换出到了交换空间

当 Redis 释放了内存后，内存被交还给了内存分配器，但是内存分配器不一定会将内存交还给系统，这时可能导致 **used_memory** 的值和从系统角度看到的 Redis 消耗不符，这时可以拿 **used_memory_peak** 作一下参考

---

### Persistence section

Property | Value
-------- | ---
**loading**| 用来表明是否正在加载dump数据
**rdb_changes_since_last_save**| 距离上一次dump数据后产生的变更数量
**rdb_bgsave_in_progress**| 用来表明是否有 RDB save 正在执行
**rdb_last_save_time**| 上一次成功进行 RDB save 的时间点
**rdb_last_bgsave_status**| 上一次 RDB save 的最终状态
**rdb_last_bgsave_time_sec**| 上一次 RDB save 持续的时间，单位是秒
**rdb_current_bgsave_time_sec**| 正在进行中的 RDB save 持续的时间
**aof_enabled**| 用来表明 AOF 日志是否打开
**aof_rewrite_in_progress**| 标明 AOF 重写操作是否正在进行
**aof_rewrite_scheduled**| 标明 一旦 RDB save 完成，是否继续进行 AOF 重写操作
**aof_last_rewrite_time_sec**| 上一次 AOF 重写操作的持续时间
**aof_current_rewrite_time_sec**| 当前正在执行的 AOF 操作的持续时间
**aof_last_bgrewrite_status**|  上一次的 AOF 最终反馈状态
**aof_last_write_status**| 

如果 AOF 打开了，还会有如下多出来的信息

Property | Value
-------- | ---
**aof_current_size**| AOF current file size
**aof_base_size**| AOF file size on latest startup or rewrite
**aof_pending_rewrite**|Flag indicating an AOF rewrite operation will be scheduled once the on-going RDB save is complete.
**aof_buffer_length**| Size of the AOF buffer
**aof_rewrite_buffer_length**|Size of the AOF rewrite buffer
**aof_pending_bio_fsync**| Number of fsync pending jobs in background I/O queue
**aof_delayed_fsync**| Delayed fsync counter

如果加载操作正在进行，还会多出如下的信息

Property | Value
-------- | ---
**loading_start_time**|Epoch-based timestamp of the start of the load operation
**loading_total_bytes**|Total file size
**loading_loaded_bytes**|Number of bytes already loaded
**loading_loaded_perc**| Same value expressed as a percentage
**loading_eta_seconds**|ETA in seconds for the load to be complete

---

### Stats section

Property | Value
-------- | ---
**total_connections_received**| 实例接收到的总连接数
**total_commands_processed**| 实例处理过的总命令数
**instantaneous_ops_per_sec**| 每秒总处理的命令数
**total_net_input_bytes**| 网络总输入，单位byte
**total_net_output_bytes**| 网络总输出，单位byte
**instantaneous_input_kbps**| 
**instantaneous_output_kbps**|
**rejected_connections**| 因为到达最大连接数而拒绝的连接总量
**sync_full**| 
**sync_partial_ok**|
**sync_partial_err**| 
**expired_keys**| 过期Key累计总量
**evicted_keys**| 由于到到最大内存限制而回收的Key总量
**keyspace_hits**| 数据库中成功找到指定Key的次数
**keyspace_misses**| 数据库中没有成功找到指定Key的次数
**pubsub_channels**| 全局订阅频道的个数
**pubsub_patterns**| 全局订阅频道特征模式的个数
**latest_fork_usec**| 最近的一次 Fork 操作的持续时间，单位是ms
**migrate_cached_sockets**| 

---

### Replication section

Property | Value
-------- | ---
**role**| 角色，如果复制源，就为 slave,其它情况下都为master
**connected_slaves**| slave 的连接数
**master_repl_offset**| 
**repl_backlog_active**|
**repl_backlog_size**|
**repl_backlog_first_byte_offset**|
**repl_backlog_histlen**|


---

Property | Value
-------- | ---
**used_cpu_sys**| 实例消耗的内核时间
**used_cpu_user**| 实例消耗的用户时间
**used_cpu_sys_children**| 后台处理消耗的内核时间
**used_cpu_user_children**| 后台处理消耗的用户时间


---

### Commandstats section


Property | Value
-------- | ---
**cmdstat_info**| 调用总量，时间开销，和平均每个调用的时间花费
**cmdstat_command**| 调用总量，时间开销，和平均每个调用的时间花费

---

### Cluster section

Property | Value
-------- | ---
**cluster_enabled**| 标示有没有开启集群

---

### Keyspace section

这里会显示每个数据库中的 Key 统计信息，Key 数量，过期次数，平均TTL时长


> **Note:** 还有很多的不完善，相关信息在持续补充中


---

# 命令汇总

* **`wget http://download.redis.io/releases/redis-3.2.1.tar.gz`**
* **`tar -zxvf redis-3.2.1.tar.gz`**
* **`cd redis-3.2.1`**
* **`make`**
* **`echo $?`**
* **`cd src/`**
* **`ls  | grep -v "\."`**
* **`./redis-server --help`**
* **`./redis-server --version`**
* **`redis-server`**
* **`redis-cli -p 6379`**

---


[redis]:http://redis.io/
[redis_install]:/2015/04/02/Redis-install/
[info]:http://redis.io/commands/info

