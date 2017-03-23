---
layout: post
title:  Redis 容器与配置
author: wilmosfang
categories:   docker  
tags:   docker nosql redis 
wc: 688 1980 22669
excerpt: redis 容器启动，使用容器连接redis容器，redis配置，挂载本地卷，查看日志，查看数据文件，使用客户端容器进行访问，以及需要注意的事项 
comments: true
---



# 前言

**Redis** 是C语言编写的开源内存型KV存储

> **Tip:** The name Redis means **REmote DIctionary Server**

目前互联网应用中大量使用着 **Redis**

**Docker** 又是 **DevOps** 神器

两者结合使用可以给开发带来极大便利

但是目前来讲，容器更适合运行无状态的服务，因为这样可以更方便地进行水平扩展，而存储一类属于典型的有状态应用，所以处理起来要有更多注意

主要得注意以下三方面:

* 配置 : 使用默认配置有时满足不了客制化需求
* 数据 : 做缓存时可以不用关心，如果开启持久化，显然不便于放在镜像中 
* 日志 : 默认情况下日志会写到标准输出，然后由Docker log 收集，但有时这不便于日志管理(也只是有了Docker后才大量使用这种处理方式，对于现有的日志收集系统侵入性比较强)，应该写到一个指定位置，便于后期处理和分析

> **Tip:** 其实还有网络，由于Redis异步非阻塞的事件驱动特性，接受处理网络请求非常快，这时docker的转发网络就变成了性能瓶颈(系统内核会多把一层关)，直接使用主机网络可以有效缓解这个问题，但这里用于开发环境，暂时不用考虑

这里分享一下 **[Redis][redis]** 容器的相关操作和基础，详细可以参考 **[Docker Hub][redis]** 和 **[官方文档][redis_soft]** 


> **Tip:** 当前的最新版本为 **Redis 3.0.7**


---


# 概要

* TOC
{:toc}



---

## 环境

~~~
[root@h104 ~]# hostnamectl 
   Static hostname: h104
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 12a02f8ee88d4b8e91d54d1390b0b275
           Boot ID: 6109315d5e854747b7732bb2d163ed34
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.4.4.el7.x86_64
      Architecture: x86-64
[root@h104 ~]# uname -a 
Linux h104 3.10.0-327.4.4.el7.x86_64 #1 SMP Tue Jan 5 16:07:00 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
[root@h104 ~]# docker --version
Docker version 1.9.1, build a34a1d5
[root@h104 ~]#
~~~

---

## 启动一个redis容器


由于本地并没有 redis 镜像，所以它会自动去 dockerhub 上拉取

~~~
[root@h104 ~]# docker run --name test-redis -d redis
Unable to find image 'redis:latest' locally
latest: Pulling from library/redis
70e9a6907f10: Pull complete 
32f2a4cccab8: Pull complete 
34446d9c0a72: Pull complete 
145623ef4dd4: Pull complete 
8d9771e0b6ee: Pull complete 
018a9e995b7a: Pull complete 
0deb375d521a: Pull complete 
c152ff671a13: Pull complete 
aa64b511d09c: Pull complete 
23c6ea3916be: Pull complete 
9cd603438a28: Pull complete 
8be59f114d61: Pull complete 
70aeeb507933: Pull complete 
840927008ad0: Pull complete 
970117e3ce7f: Pull complete 
9d6240c55f04: Pull complete 
b2f46bd01929: Pull complete 
1f4ff6e27d64: Pull complete 
Digest: sha256:68524efa50a33d595d7484de3939a476b38667c7b4789f193761899ca125d016
Status: Downloaded newer image for redis:latest
71c69b5bf62cc4521a43f91869114d598eab5d4826372371909c461e6f024f71
[root@h104 ~]#
[root@h104 ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
71c69b5bf62c        redis               "docker-entrypoint.sh"   49 seconds ago      Up 48 seconds       6379/tcp            test-reds
[root@h104 ~]#
[root@h104 ~]# docker logs 71c69b5bf62c
1:C 28 Apr 02:41:05.459 # Warning: no config file specified, using the default config. In order to specify a config file use redis-sever /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
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

1:M 28 Apr 02:41:05.461 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to te lower value of 128.
1:M 28 Apr 02:41:05.461 # Server started, Redis version 3.0.7
1:M 28 Apr 02:41:05.461 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this isse add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to tke effect.
1:M 28 Apr 02:41:05.461 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and emory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, nd add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 28 Apr 02:41:05.461 * The server is now ready to accept connections on port 6379
[root@h104 ~]# sysctl -a | grep  vm.overcommit_memory
vm.overcommit_memory = 0
[root@h104 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@h104 ~]#
[root@h104 ~]# docker images  | grep redis
redis                                   latest              1f4ff6e27d64        6 days ago          177.4 MB
[root@h104 ~]# 
~~~

从日志中可以看到，redis通过检查系统配置给出了一些优化建议，以便更好的发挥其性能

这种状态下的redis在后台运行，没有公开端口可以让外面的客户端直连

但是可以通过link的方式，实现容器间的对接


---

## 使用客户端容器连接

~~~
[root@h104 ~]# docker  ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
71c69b5bf62c        redis               "docker-entrypoint.sh"   6 hours ago         Up 7 minutes        6379/tcp            test-redis
[root@h104 ~]# docker run -it --link test-redis:redis --rm redis redis-cli -h redis -p 6379
redis:6379> info 
# Server
redis_version:3.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:6f8b503a2787e3a6
redis_mode:standalone
os:Linux 3.10.0-327.4.4.el7.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.9.2
process_id:1
run_id:9515a58ed6b97720e31a6a7ecafa4d8f7dbde630
tcp_port:6379
uptime_in_seconds:584
uptime_in_days:0
hz:10
lru_clock:2216150
config_file:

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:815912
used_memory_human:796.79K
used_memory_rss:7909376
used_memory_peak:815912
used_memory_peak_human:796.79K
used_memory_lua:36864
mem_fragmentation_ratio:9.69
mem_allocator:jemalloc-3.6.0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1461833358
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
total_commands_processed:0
instantaneous_ops_per_sec:0
total_net_input_bytes:14
total_net_output_bytes:0
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
used_cpu_sys:2.94
used_cpu_user:0.16
used_cpu_sys_children:0.01
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
redis:6379> 
redis:6379> 
redis:6379> 
redis:6379> CONFIG get *data*
1) "databases"
2) "16"
3) "slave-serve-stale-data"
4) "yes"
redis:6379> exit
[root@h104 ~]#
[root@h104 ~]# docker  ps -a  | grep redis
71c69b5bf62c        redis                       "docker-entrypoint.sh"   6 hours ago         Up 31 minutes               6379/tcp                                                                         test-redis
[root@h104 ~]# 
~~~

这里对 **`docker run -it --link test-redis:redis --rm redis redis-cli -h redis -p 6379`** 进行一下解析


Option| Comment
-------- | ---
`docker run`| 调用 docker 命令的 run 子命令
`-i`| 打开 STDIN ，进入交互模式
`-t`| 分配一个伪终端，一般都和 `-i` 一起使用
`--link test-redis:redis`| 连接 test-redis 容器，并且为这个容器定义一个别名，叫 redis (`redis-cli -h redis -p 6379` 中指定的 redis 就是用的这个别名)
`--rm`| 此容器用完就删掉，不留存，一般用在短期前台交互的情况下(默认特性是不删的)


这里稍微就 **`-p、-P、--link、--expose、EXPOSE`** 进行一下区分


Item| Comment
-------- | ---
`EXPOSE`| 记录服务可用的端口，但是并不创建和宿主机之间的映射，只出现在Dockerfile中
`--expose`|运行时暴露端口，但是并不创建和主机之间的映射，同 `EXPOSE` 功能一样，但只出现在 CLI 中
`-p`| 创建端口映射规则，如`-p ip:hostPort:containerPort`, 必须指定 containerPort ，如果没有指定 hostPort, Docker会自动分配端口
`-P`| 将Dockerfile 里暴露的所有容器端口映射到动态分配的宿主机端口上
`--link`| 在容器之间创建链接，如 `--link name:alias`,这会创建一系列环境变量，并在消费者容器的 `/etc/hosts` 文件里添加入口项，必须暴露或发布端口


操作和正常使用客户端一样

~~~
[root@h104 ~]# docker run -it --link test-redis:redis --rm redis redis-cli -h redis -p 6379
redis:6379> get a 
(nil)
redis:6379> set a b 
OK
redis:6379> get a 
"b"
redis:6379>
~~~


---

## 指定配置

指定配置后，映射本地卷，就可以对数据文件和日志文件的读写位置进行控制


---

### 创建配置

~~~
[root@h104 x]# vim redis6379.conf
[root@h104 x]# cat redis6379.conf 
daemonize no 
pidfile /data/redis6379.pid
port 6379
tcp-backlog 511
timeout 1800
tcp-keepalive 0
loglevel notice
logfile "/data/redis6379.log"
databases 50 
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump6379.rdb
dir /data/
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
[root@h104 x]# 
[root@h104 x]# diff /tmp/redis.conf  ./redis6379.conf 
1,2c1,2
< daemonize yes
< pidfile /var/run/redis.pid
---
> daemonize no 
> pidfile /data/redis6379.pid
5c5
< timeout 0
---
> timeout 1800
8,9c8,9
< logfile ""
< databases 20 
---
> logfile "/data/redis6379.log"
> databases 50 
16,17c16,17
< dbfilename dump.rdb
< dir ./
---
> dbfilename dump6379.rdb
> dir /data/
25c25
< appendfilename "appendonly.aof"
---
> appendfilename "appendonly6379.aof"
[root@h104 x]#
~~~

后面是与默认配置的差异，这里稍微解释一下

Option| Comment
-------- | ---
`daemonize no` | 不以后台服务的形式运行
`timeout 1800`| 将超时时间限定为半小时，默认是不限的
`logfile "/data/redis6379.log"`|指定日志的路径
`databases 50`|将数据库设为50个
`dir /data/`|指定存储目录
`dbfilename dump6379.rdb`|指定数据文件名
`appendfilename "appendonly6379.aof"`|指定append文件名


> **Note:** 为什么普通情况下使用redis都是开启后台服务模式，而这里要使用前台模式呢，那是因为，容器化后，必须终结于一个前台进程，否则容器就直接退出了，这涉及容器交互模式运行和后台运行的一些特性

---

###  挂载本地卷到容器

~~~
[root@h104 x]# pwd
/tmp/x
[root@h104 x]# ls
redis6379.conf
[root@h104 x]# docker run --name myredis -d -v /tmp/x:/data redis  redis-server /data/redis6379.conf 
a1a4cfe7e7498827aeeb770744a9443a89212017282b41233cb3c4258a9a0b61
[root@h104 x]# docker ps -l
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
a1a4cfe7e749        redis               "docker-entrypoint.sh"   12 seconds ago      Up 11 seconds       6379/tcp            myredis
[root@h104 x]#
~~~

这里对 **`docker run --name myredis -d -v /tmp/x:/data redis  redis-server /data/redis6379.conf`** 进行一下解析


Option| Comment
-------- | ---
`docker run`| 调用 docker 命令的 run 子命令
`--name myredis`| 给这个容器取名为 **myredis**
`-d`| 后台模式运行
`-v /tmp/x:/data`| 将本地的 **`/tmp/x`** 目录挂载到容器中的 **`/data`** 目录
`redis`|使用redis镜像
`redis-server /data/redis6379.conf`| 使用指定的配置初始化并启动redis服务


---

### 查看日志

因为本地目录挂载到了容器中，那么日志根据映射就直接记录到了本地

~~~
[root@h104 x]# pwd
/tmp/x
[root@h104 x]# ls
redis6379.conf  redis6379.log
[root@h104 x]# cat /tmp/x/redis6379.log 
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
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

1:M 28 Apr 14:47:24.523 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 28 Apr 14:47:24.523 # Server started, Redis version 3.0.7
1:M 28 Apr 14:47:24.523 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 28 Apr 14:47:24.524 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 28 Apr 14:47:24.524 * The server is now ready to accept connections on port 6379
[root@h104 x]#
~~~


---

###  使用容器的方式访问redis容器


~~~
[root@h104 x]# docker ps -l 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
a1a4cfe7e749        redis               "docker-entrypoint.sh"   19 minutes ago      Up 19 minutes       6379/tcp            myredis
[root@h104 x]# docker run -it --link myredis:redis --rm redis redis-cli -h redis -p 6379
redis:6379> CONFIG GET *pidfile*
1) "pidfile"
2) "/data/redis6379.pid"
redis:6379> CONFIG GET timeout 
1) "timeout"
2) "1800"
redis:6379> CONFIG GET *logfile*
1) "logfile"
2) "/data/redis6379.log"
redis:6379> CONFIG GET *dbfilename*
1) "dbfilename"
2) "dump6379.rdb"
redis:6379> CONFIG GET *dir*
1) "dir"
2) "/data"
redis:6379> CONFIG GET *databases*
1) "databases"
2) "50"
redis:6379> info 
# Server
redis_version:3.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:6f8b503a2787e3a6
redis_mode:standalone
os:Linux 3.10.0-327.4.4.el7.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.9.2
process_id:1
run_id:280352487e387d2f70c9358b763dab833094943c
tcp_port:6379
uptime_in_seconds:1281
uptime_in_days:0
hz:10
lru_clock:2238333
config_file:/data/redis6379.conf

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:843568
used_memory_human:823.80K
used_memory_rss:7929856
used_memory_peak:843568
used_memory_peak_human:823.80K
used_memory_lua:36864
mem_fragmentation_ratio:9.40
mem_allocator:jemalloc-3.6.0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1461854844
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
total_connections_received:4
total_commands_processed:19
instantaneous_ops_per_sec:0
total_net_input_bytes:757
total_net_output_bytes:1178
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
used_cpu_sys:5.66
used_cpu_user:0.23
used_cpu_sys_children:0.03
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
redis:6379> select 49
OK
redis:6379[49]> select 50
(error) ERR invalid DB index
redis:6379> select 49
OK
redis:6379[49]> get a 
(nil)
redis:6379[49]> set a b 
OK
redis:6379[49]> get a 
"b"
redis:6379[49]> save 
OK
redis:6379[49]> set b c 
OK
redis:6379[49]> get b 
"c"
redis:6379[49]> save 
OK
redis:6379[49]> quit
[root@h104 x]# 
[root@h104 x]# ls
dump6379.rdb  redis6379.conf  redis6379.log
[root@h104 x]# tail redis6379.log
1:M 28 Apr 14:47:24.523 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 28 Apr 14:47:24.524 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 28 Apr 14:47:24.524 * The server is now ready to accept connections on port 6379
1:M 28 Apr 15:09:11.241 * 1 changes in 900 seconds. Saving...
1:M 28 Apr 15:09:11.275 * Background saving started by pid 14
14:C 28 Apr 15:09:11.460 * DB saved on disk
14:C 28 Apr 15:09:11.461 * RDB: 6 MB of memory used by copy-on-write
1:M 28 Apr 15:09:11.515 * Background saving terminated with success
1:M 28 Apr 15:09:15.457 * DB saved on disk
1:M 28 Apr 15:09:27.514 * DB saved on disk
[root@h104 x]# 
[root@h104 x]# ls
dump6379.rdb  redis6379.conf  redis6379.log
[root@h104 x]# 
~~~

对其中的部分配置进行了校验，做了一些基本操作，然后保存

从日志中可以看到输出符合预期

数据文件也生成在了我们指定的位置

如果这个容器出现故障，我们可以将数据文件提供给其它redis容器使用，日志文件也可以使用ELK进行管理

通过以上的方式，我们成功对一个redis容器进行了客制化的修改



---

# 命令汇总

* **`hostnamectl`**
* **`docker --version`**
* **`docker run --name test-redis -d redis`**
* **`docker ps`**
* **`docker logs 71c69b5bf62c`**
* **`sysctl -a | grep  vm.overcommit_memory`**
* **`cat /sys/kernel/mm/transparent_hugepage/enabled`**
* **`docker images  | grep redis`**
* **`docker run -it --link test-redis:redis --rm redis redis-cli -h redis -p 6379`**
* **`docker  ps -a  | grep redis`**
* **`cat redis6379.conf`**
* **`diff /tmp/redis.conf  ./redis6379.conf`**
* **`docker run --name myredis -d -v /tmp/x:/data redis  redis-server /data/redis6379.conf`**
* **`docker ps -l`**
* **`cat /tmp/x/redis6379.log`**
* **`docker ps -l`**
* **`docker run -it --link myredis:redis --rm redis redis-cli -h redis -p 6379`**
* **`tail redis6379.log`**



---

[redis]:https://hub.docker.com/_/redis/
[redis_soft]:http://redis.io/

