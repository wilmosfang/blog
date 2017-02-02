---
layout: post
title:  Redis 多数据库
author: wilmosfang
categories:   redis 
tags:    redis  ruby nosql
wc: 326  844 9104 
excerpt:  redis 多数据的支持，多数据库切换，配置，清空数据，CLI 操作，ruby API 操作，注意事项 
comments: true
---



# 前言

**[Redis][redis]** 是一款非常好用的KV缓存数据库，生产中会大量使用到

为了区分应用，常规的做法是通过增加redis实例，监听在不同端口上以进行区分，这样在体量小的时候问题不大，当体量大了，就会有产生如下问题：

* redis实例多了
* 每个redis实例的潜能并未发挥充分
* 应用与redis之间的对应容易混乱
* 日志与数据文件分散，配置修改麻烦
* slave数量爆增

总体来说，就是明显提升了运维管理成本

那有没有很好的解决办法？

我们之所以需要区别对待，很大一部分原因是希望获得一个干净的名称空间，不被其它应用意外篡改

redis本身具备的多数据库特性就可以很好的满足这类需求，完全不必运行多个实例

(当然，由于redis的单线程特性，如果应用单个操作平均耗时过长，使用实例区分开来还是很有必要的)

这里分享一下redis的多数据特性与简单操作


> **Tip:**当前的最新版本为 **redis-3.0.7**

---


# 概要

* TOC
{:toc}


---

## 环境


~~~
[root@h102 redis-3.0.7]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 redis-3.0.7]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 redis-3.0.7]# ./src/redis-server --version
Redis server v=3.0.7 sha=00000000:0 malloc=jemalloc-3.6.0 bits=64 build=db8110605b24fc73
[root@h102 redis-3.0.7]#
~~~

---

##  切换数据库

登录后默认是连接到0号数据库

数据的下标是从0开始的，代表第一个数据库，默认数据数据库没有下标，其实就是0号数据库


~~~
[root@h102 redis-3.0.7]# ./src/redis-cli -p 6379
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> 
~~~

可以使用 **select** 加上数据进行数据库切换

~~~
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> select 0
OK
127.0.0.1:6379> select 3
OK
127.0.0.1:6379[3]> select 4
OK
127.0.0.1:6379[4]>
~~~

切换后下标会变

---

## 配置


我们尝试切换到16

~~~
127.0.0.1:6379> select 16
(error) ERR invalid DB index
127.0.0.1:6379> select 15
OK
127.0.0.1:6379[15]> select 14
OK
127.0.0.1:6379[14]> select 13
OK
127.0.0.1:6379[13]> select 12
OK
127.0.0.1:6379[12]> 
...
...
127.0.0.1:6379[3]> select 2
OK
127.0.0.1:6379[2]> select 1
OK
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> 
~~~

但是15-0号数据库有，说明默认有16个数据库可用

这个配置可以在这里看

~~~
127.0.0.1:6379> CONFIG GET databa*
1) "databases"
2) "16"
127.0.0.1:6379>
127.0.0.1:6379> CONFIG set databases 20
(error) ERR Unsupported CONFIG parameter: databases
127.0.0.1:6379>
~~~

并且发现不能修改

这个初始化配置是在启动redis之前在 redis.conf文件中指定的

~~~
[root@h102 redis-3.0.7]# cat redis.conf | grep -v "^#" | grep -v "^$"| grep databa
databases 16
[root@h102 redis-3.0.7]# 
~~~

我们可以修改配置，指定20个库，重启服务

~~~
127.0.0.1:6379> CONFIG GET databa*
1) "databases"
2) "20"
127.0.0.1:6379> select 16
OK
127.0.0.1:6379[16]> select 18
OK
127.0.0.1:6379[18]> select 19
OK
127.0.0.1:6379[19]> select 20
(error) ERR invalid DB index
127.0.0.1:6379>
~~~

多出来了几个库，符合我们的预期


---

## 名称空间

在不同库中设定的内容，并不会冲突

~~~
127.0.0.1:6379> select 10
OK
127.0.0.1:6379[10]> set a test
OK
127.0.0.1:6379[10]> select 11
OK
127.0.0.1:6379[11]> set a in11
OK
127.0.0.1:6379[11]> get a
"in11"
127.0.0.1:6379[11]> select 10
OK
127.0.0.1:6379[10]> get a
"test"
127.0.0.1:6379[10]> 
~~~

---

## 清空


**FLUSHALL** 是一个非常有破坏力的命令，因为它会清掉所有库中的数据

~~~
127.0.0.1:6379[10]> get a
"test"
127.0.0.1:6379[10]> select 11
OK
127.0.0.1:6379[11]> get a
"in11"
127.0.0.1:6379[11]> FLUSHALL
OK
127.0.0.1:6379[11]> get a
(nil)
127.0.0.1:6379[11]> select 10
OK
127.0.0.1:6379[10]> get a
(nil)
127.0.0.1:6379[10]>
~~~


---

## 使用CLI


~~~
[root@h102 redis-3.0.7]# ./src/redis-cli -p 6379 -n 13 set a in13 
OK
[root@h102 redis-3.0.7]# ./src/redis-cli -p 6379 -n 13 get a  
"in13"
[root@h102 redis-3.0.7]# ./src/redis-cli -p 6379 -n 11 get a  
(nil)
[root@h102 redis-3.0.7]#
~~~

---

## 使用API


使用API来操作，这里以ruby演示


先要安装一下gem

~~~
[root@h102 ruby]# ruby -v 
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
[root@h102 ruby]# gem install redis
Fetching: redis-3.2.2.gem (100%)
Successfully installed redis-3.2.2
Parsing documentation for redis-3.2.2
Installing ri documentation for redis-3.2.2
Done installing documentation for redis after 2 seconds
1 gem installed
[root@h102 ruby]#
~~~

在irb中进行操作

~~~
[root@h102 ruby]# irb 
2.3.0 :001 > require 'redis'
 => true 
2.3.0 :002 > redis = Redis.new(:host=>"localhost",:port=>6379,:db=>12)
 => #<Redis client v3.2.2 for redis://localhost:6379/12> 
2.3.0 :003 > redis.class
 => Redis 
2.3.0 :004 >
2.3.0 :005 > redis.methods
 => [:client, :with_reconnect, :without_reconnect, :connected?, :disconnect!, :auth, :ping, :bgrewriteaof, :bgsave, :get, :dbsize, :debug, :flushall, :flushdb, :quit, :info, :lastsave, :rename, :save, :shutdown, :slaveof, :slowlog, :persist, :expire, :expireat, :ttl, :pexpire, :pexpireat, :pttl, :migrate, :del, :exists, :move, :object, :randomkey, :renamenx, :decr, :decrby, :incr, :incrby, :incrbyfloat, :set, :setex, :psetex, :setnx, :mset, :mapped_mset, :msetnx, :mapped_msetnx, :mget, :mapped_mget, :setrange, :getrange, :setbit, :getbit, :append, :bitcount, :bitop, :bitpos, :getset, :strlen, :llen, :lpush, :lpushx, :rpush, :rpushx, :lpop, :rpop, :rpoplpush, :_bpop, :blpop, :brpop, :brpoplpush, :lindex, :linsert, :lrange, :lrem, :lset, :ltrim, :scard, :sadd, :srem, :[], :[]=, :monitor, :spop, :srandmember, :smove, :sismember, :smembers, :sdiff, :sdiffstore, :sinter, :sinterstore, :zcard, :sunionstore, :sunion, :zincrby, :zrem, :zadd, :zrange, :zscore, :zrevrange, :zrank, :zrevrank, :zremrangebyrank, :zrangebylex, :zrevrangebylex, :zrangebyscore, :zrevrangebyscore, :zremrangebyscore, :zcount, :zinterstore, :inspect, :method_missing, :zunionstore, :hlen, :hset, :hsetnx, :hmset, :mapped_hmset, :hget, :hmget, :mapped_hmget, :hdel, :hexists, :hincrby, :hincrbyfloat, :hkeys, :hvals, :hgetall, :publish, :subscribed?, :subscribe, :unsubscribe, :psubscribe, :punsubscribe, :pubsub, :watch, :unwatch, :pipelined, :multi, :discard, :script, :_eval, :evalsha, :_scan, :scan_each, :hscan, :hscan_each, :zscan, :zscan_each, :sscan, :sscan_each, :pfadd, :pfcount, :pfmerge, :dup, :sentinel, :keys, :type, :sort, :id, :select, :synchronize, :dump, :config, :restore, :scan, :exec, :sync, :echo, :time, :eval, :mon_try_enter, :try_mon_enter, :mon_enter, :mon_exit, :mon_synchronize, :new_cond, :instance_of?, :public_send, :instance_variable_get, :instance_variable_set, :instance_variable_defined?, :remove_instance_variable, :private_methods, :kind_of?, :instance_variables, :tap, :define_singleton_method, :is_a?, :singleton_method, :extend, :to_enum, :enum_for, :<=>, :===, :=~, :!~, :eql?, :respond_to?, :freeze, :display, :object_id, :send, :to_s, :method, :public_method, :nil?, :hash, :class, :singleton_class, :clone, :itself, :taint, :tainted?, :untaint, :untrust, :trust, :untrusted?, :methods, :protected_methods, :frozen?, :public_methods, :singleton_methods, :!, :==, :!=, :__send__, :equal?, :instance_eval, :instance_exec, :__id__] 
2.3.0 :006 > redis.methods.grep /set/
 => [:set, :setex, :psetex, :setnx, :mset, :mapped_mset, :msetnx, :mapped_msetnx, :setrange, :setbit, :getset, :lset, :hset, :hsetnx, :hmset, :mapped_hmset, :instance_variable_set] 
2.3.0 :007 > redis.set("testredis","rb")
 => "OK" 
2.3.0 :008 > redis.methods.grep /get/
 => [:get, :mget, :mapped_mget, :getrange, :getbit, :getset, :hget, :hmget, :mapped_hmget, :hgetall, :instance_variable_get] 
2.3.0 :009 > redis.get("testredis")
 => "rb" 
2.3.0 :010 > 
~~~

尝试在cli检索一下

~~~
[root@h102 redis-3.0.7]# ./src/redis-cli -p 6379 -n 12 get testredis
"rb"
[root@h102 redis-3.0.7]# ./src/redis-cli -p 6379 -n 13 get testredis
(nil)
[root@h102 redis-3.0.7]# 
~~~


---

## 注意

目前redis没法逆向关联数据，意思就是只能知道库里有哪些key，但没法知道一个key来自于哪个库，

这就要求应用自己去记录将数据放到了哪个库中，到时候依然得去那个库中取




---


# 命令汇总


* **`./src/redis-server --version`**
* **`./src/redis-cli -p 6379`**
* **`cat redis.conf | grep -v "^#" | grep -v "^$"| grep databa`**
* **`./src/redis-cli -p 6379 -n 13 set a in13`**
* **`./src/redis-cli -p 6379 -n 13 get a`**
* **`gem install redis`**
* **`irb`**
* **`./src/redis-cli -p 6379 -n 12 get testredis`**


---

[redis]:http://redis.io/
[installation]:http://redis.io/download#installation

