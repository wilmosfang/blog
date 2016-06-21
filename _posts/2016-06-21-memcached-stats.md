---
layout: post
title:   Memcached 状态详解
author: wilmosfang
tags:   nosql memcached 
categories:   nosql 
wc: 211  476 6809 
excerpt: 连接 memcached 实例，stats 命令，指标详解 
comments: true
---


# 前言


**[memcached][memcached]** 是一个免费开源的，高性能分布式内存对象缓存系统

在 **[memcached][memcached]** 运维管理过程中，会需要了解缓存系统当前的状态，这里分享一下 **[memcached][memcached]** 状态的详细解释

> **Tip:** **[memcached][memcached]** 相关基础，可以参考之前的一篇博客 **[memcached基础][memcached_basic_operation]**


详细可以参考 **[memcached 源码][protocol]**


> **Tip:**   目前最新版本为 **Memcached  v1.4.26** ，实验中使用 **v1.4.25** 



---


# 概要

* TOC
{:toc}


---

## 连接实例

由于 memcached 的协议非常简单，可以使用 **telnet** 直接连接

~~~
[mtest@meminst ~]$ telnet localhost 11212
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
version
VERSION 1.4.25
~~~

> **Tip:** 使用 telnet 作客户端，是在直接使用 tcp 建立连接和传送数据，所以并没有其它客户端那么友好的提示信息或界面，这个得忍

---

## stats

要获取内部统计数据可以通过 **stats** 命令，最主要的状态信息也都在里面

~~~
[mtest@meminst ~]$ telnet localhost 11212
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
stats 
STAT pid 92579
STAT uptime 15483344
STAT time 1466503609
STAT version 1.4.25
STAT libevent 1.4.13-stable
STAT pointer_size 64
STAT rusage_user 30237.694171
STAT rusage_system 48141.977307
STAT curr_connections 74
STAT total_connections 11487
STAT connection_structures 99
STAT reserved_fds 50
STAT cmd_get 2179473016
STAT cmd_set 1780727826
STAT cmd_flush 0
STAT cmd_touch 0
STAT get_hits 2151718170
STAT get_misses 27754846
STAT delete_misses 523904
STAT delete_hits 609080
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT touch_hits 0
STAT touch_misses 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 234944598773
STAT bytes_written 1788730431727
STAT limit_maxbytes 4294967296
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT time_in_listen_disabled_us 0
STAT threads 10
STAT conn_yields 330
STAT hash_power_level 23
STAT hash_bytes 67108864
STAT hash_is_expanding 0
STAT malloc_fails 0
STAT bytes 3841174015
STAT curr_items 7995364
STAT total_items 1780727826
STAT expired_unfetched 210230
STAT evicted_unfetched 9554685
STAT evictions 17096414
STAT reclaimed 281812
STAT crawler_reclaimed 0
STAT crawler_items_checked 0
STAT lrutail_reflocked 0
END
~~~

---

## 状态详解



Verb     | Comment
-------- | ---
**pid**| 此服务进程号
**uptime**| 此服务的连续运行时间，单位是秒
**time**| 服务器所在主机当前系统的时间，单位是秒
**version**| 服务软件版本
**libevent**| libevent 版本
**pointer_size**| 所在主机操作系统的指针位数，32或64
**rusage_user**| 进程的累计用户时间
**rusage_system**| 进程的累计系统时间
**curr_connections**| 当前系统打开的连接数
**total_connections**| 从启动到现在，系统打开过的连接总数
**connection_structures**| 实例分配的连接构造(结构体)数
**reserved_fds**| 内部使用(保留)的FD数
**cmd_get**| get 命令执行的累计次数 (cmd_get = get_hits + get_misses)
**cmd_set**| set 命令执行的累计次数
**cmd_flush**| flush 命令执行的累计次数
**cmd_touch**| touch 命令执行的累计次数
**get_hits**| get 命令成功返回的累计次数
**get_misses**| get 命令失败返回的累计次数
**delete_misses**| delete 命令失败返回的累计次数
**delete_hits**| delete 命令成功返回的累计次数
**incr_misses**| incr 命令失败返回的累计次数
**incr_hits**| incr 命令成功返回的累计次数
**decr_misses**| decr 命令失败返回的累计次数
**decr_hits**| decr 命令成功返回的累计次数
**cas_misses**| cas 命令失败返回的累计次数
**cas_hits**| cas 命令成功返回的累计次数
**cas_badval**| cas 命令检查值不等(错误的 CAS id)而更新失败的累计次数
**touch_hits**| touch 命令成功返回的累计次数
**touch_misses**| touch 命令失败返回的累计次数
**auth_cmds**| 认证命令执行累计次数
**auth_errors**| 认证命令执行失败累计次数
**bytes_read**| 实例从网络读取的总字节数
**bytes_written**| 实例发送到网络的总字节数
**limit_maxbytes**| 缓存允许使用的最大字节数，等于我们启动服务时设置的大小
**accepting_conns**| 布尔值，实例是否接受连接(1为接受，0为不接受)
**listen_disabled_num**| 统计当前服务器连接数曾经达到最大连接的次数，这个次数应该为0或者接近于0，如果这个数字不断增长， 就要小心我们的服务了
**time_in_listen_disabled_us**| 实例由于到达最大连接数，而拒绝连接(处于监听状态但不接受新连接)的累计时间，单位为ms
**threads**| 可以被请求的工作线程的总数量
**conn_yields**| Number of times any connection yielded to another due to hitting the -R limit
**hash_power_level**| 当前的哈希表翻倍的级别（Current size multiplier for hash table）
**hash_bytes**| 当前哈希表的大小，单位字节
**hash_is_expanding**| 布尔值1或0，表明哈希表是否增涨到过新的量级
**malloc_fails**| 内存分配失败的次数
**bytes**| 当前存储总字节数
**curr_items**| 实例当前存储的items数量
**total_items**| 从服务器启动以后存储过的items总数量
**expired_unfetched**| 已过期但未获取的对象数目
**evicted_unfetched**| 已驱逐但未获取的对象数目
**evictions**| 为获取空闲内存(LRU)而删除的items数,分配给memcache的空间用满后需要删除旧的items来得到空间分配给新的items
**reclaimed**| 因为条目过期而执行的回收次数
**crawler_reclaimed**| 被 LRU 爬虫释放的累计对象数
**crawler_items_checked**| 被 LRU 检查过的对象总数
**lrutail_reflocked**| Times LRU tail was found with active ref .Items can be evicted to avoid OOM errors 


详细可以参考 **[memcached 源码][protocol]**


这些值的组合结果可以获取常见的可用性标准，如：

**`命中率 = get_hits / cmd_get * 100%`**

其它的指标以此类推

有空可以据此写一个 memcached 的状态性能指标展示工具


---

# 命令汇总

* **`telnet localhost 11212`**

---


[memcached]:http://memcached.org/
[memcached_basic_operation]:http://soft.dog/2015/09/23/memcached-basic-operation/
[protocol]:https://github.com/memcached/memcached/blob/master/doc/protocol.txt

