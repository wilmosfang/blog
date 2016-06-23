---
layout: post
title: redis-benchmark 基础
author: wilmosfang
tags:  redis nosql benchmark
categories:   nosql 
wc: 739 2451 17797
excerpt: follow me
comments: true
---


---

# 前言

**[redis-benchmark][redis-benchmark]** 是官方自带的 [Redis][redis]性能测试工具，可以有效的测试[Redis][redis]服务的性能。

下面分享一下 **[redis-benchmark][redis-benchmark]** 的基础操作，详细可以参阅 [官方文档][redis-benchmark] 

> **Tip:** 当前版本 **[Redis 3.0.5]**  

---

# 概要

* TOC
{:toc}


---

## 安装redis


下载，解压然后编译

~~~
wget http://download.redis.io/releases/redis-3.0.5.tar.gz
tar zxvf redis-3.0.5.tar.gz 
cd redis-3.0.5
make
~~~

此时编译好的命令已经存在于 **src** 目录下面

~~~
[root@h102 src]# pwd
/usr/local/src/redis-3.0.5/src
[root@h102 src]# ls  | grep -v "\." 
Makefile
redis-benchmark
redis-check-aof
redis-check-dump
redis-cli
redis-sentinel
redis-server
[root@h102 src]# 
~~~

---

## 运行redis服务


~~~
[root@h102 ~]# /usr/local/src/redis-3.0.5/src/redis-server  /etc/redis/redis6379.conf 
[root@h102 ~]# ps faux | grep redis 
root      5385  0.0  0.0 103256   828 pts/0    S+   20:46   0:00          \_ grep redis
root      5380  0.4  0.3 137444  7480 ?        Ssl  20:45   0:00 /usr/local/src/redis-3.0.5/src/redis-server *:6379                   
[root@h102 ~]# 
~~~

---


## redis-benchmark


~~~
[root@h102 src]# ./redis-benchmark -h 
Invalid option "-h" or option argument missing

Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 -dbnum <db>        SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.

Examples:

 Run the benchmark with the default configuration against 127.0.0.1:6379:
   $ redis-benchmark

 Use 20 parallel clients, for a total of 100k requests, against 192.168.1.1:
   $ redis-benchmark -h 192.168.1.1 -p 6379 -n 100000 -c 20

 Fill 127.0.0.1:6379 with about 1 million keys only using the SET test:
   $ redis-benchmark -t set -n 1000000 -r 100000000

 Benchmark 127.0.0.1:6379 for a few commands producing CSV output:
   $ redis-benchmark -t ping,set,get -n 100000 --csv

 Benchmark a specific command line:
   $ redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0

 Fill a list with 10000 random elements:
   $ redis-benchmark -r 10000 -n 10000 lpush mylist __rand_int__

 On user specified command lines __rand_int__ is replaced with a random integer
 with a range of values selected by the -r option.
[root@h102 src]# 
~~~


Option     | Comment
--------   | ---
 `-h <hostname>` |   服务器名 (default 127.0.0.1)
 `-p <port>`     |   服务端口 (default 6379)
 `-s <socket>`   |   Unix socket (overrides host and port)
 `-a <password>` |   Redis 认证密码
 `-c <clients>`  |   并行连接数 (default 50)
 `-n <requests>` |  总请求数 (default 100000)
 `-d <size>`     |      SET/GET 的数据大小，单位为bytes (default 2)
 `-dbnum <db>`   |  指定数据库编号 (default 0)
 `-k <boolean>`  | 1=keep alive 0=reconnect (default 1)
 `-r <keyspacelen>`  | 指定随机key的长度  Use random keys for SET/GET/INCR, random values for SADD Using this option the benchmark will expand the string __rand_int__ inside an argument with a 12 digits number in the specified range from 0 to keyspacelen-1. The substitution changes every time a command is executed. Default tests use this to hit random keys in the specified range.
 `-P <numreq>`   |   Pipeline <numreq> requests. Default 1 (no pipeline).
 `-q`            | 安静模式只显示 query/sec 值作为结果
 `--csv`         | 以 CSV 格式输出
 `-l`            |     循环，一直运行下去，一般作压力测试 
 `-t <tests>`    |   指定测试类型，以逗号分割  一种类型输出一行结果
 `-I`            |      空闲模式，只是打开 N 个空连接然后一直等待



---

## 性能测试

### 默认测试

默认配置是

总共100000个请求

50个并发

3字节的负载

分别对以下方法进行测试

~~~
PING_INLINE
PING_BULK
SET
GET
INCR
LPUSH
LPOP
SADD
SPOP
LPUSH (needed to benchmark LRANGE)
LRANGE_100 (first 100 elements)
LRANGE_300 (first 300 elements)
LRANGE_500 (first 450 elements)
LRANGE_600 (first 600 elements)
MSET (10 keys)
~~~

运行测试

~~~
[root@h102 src]# ./redis-benchmark 
====== PING_INLINE ======
  100000 requests completed in 1.74 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

96.77% <= 1 milliseconds
99.81% <= 2 milliseconds
99.90% <= 3 milliseconds
99.93% <= 4 milliseconds
99.94% <= 5 milliseconds
99.99% <= 6 milliseconds
100.00% <= 6 milliseconds
57537.40 requests per second

====== PING_BULK ======
  100000 requests completed in 1.83 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

95.53% <= 1 milliseconds
99.88% <= 2 milliseconds
99.99% <= 3 milliseconds
100.00% <= 3 milliseconds
54555.38 requests per second

====== SET ======
  100000 requests completed in 1.87 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

93.67% <= 1 milliseconds
99.09% <= 2 milliseconds
99.89% <= 3 milliseconds
99.92% <= 5 milliseconds
99.95% <= 7 milliseconds
99.95% <= 8 milliseconds
99.95% <= 9 milliseconds
100.00% <= 9 milliseconds
53390.28 requests per second

====== GET ======
  100000 requests completed in 1.83 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

96.77% <= 1 milliseconds
99.95% <= 2 milliseconds
99.98% <= 3 milliseconds
99.99% <= 4 milliseconds
100.00% <= 4 milliseconds
54674.69 requests per second

====== INCR ======
  100000 requests completed in 1.80 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

97.42% <= 1 milliseconds
99.96% <= 2 milliseconds
100.00% <= 2 milliseconds
55648.30 requests per second

====== LPUSH ======
  100000 requests completed in 1.70 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

98.50% <= 1 milliseconds
99.95% <= 2 milliseconds
100.00% <= 2 milliseconds
58754.41 requests per second

====== LPOP ======
  100000 requests completed in 1.80 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

97.38% <= 1 milliseconds
99.85% <= 2 milliseconds
100.00% <= 3 milliseconds
100.00% <= 3 milliseconds
55617.35 requests per second

====== SADD ======
  100000 requests completed in 1.81 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

96.70% <= 1 milliseconds
99.97% <= 2 milliseconds
100.00% <= 2 milliseconds
55126.79 requests per second

====== SPOP ======
  100000 requests completed in 1.83 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

96.79% <= 1 milliseconds
99.88% <= 2 milliseconds
100.00% <= 3 milliseconds
100.00% <= 3 milliseconds
54704.60 requests per second

====== LPUSH (needed to benchmark LRANGE) ======
  100000 requests completed in 1.77 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

97.71% <= 1 milliseconds
99.69% <= 2 milliseconds
99.76% <= 3 milliseconds
99.90% <= 4 milliseconds
99.90% <= 5 milliseconds
99.94% <= 6 milliseconds
99.96% <= 7 milliseconds
100.00% <= 7 milliseconds
56369.79 requests per second

====== LRANGE_100 (first 100 elements) ======
  100000 requests completed in 3.17 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

86.79% <= 1 milliseconds
99.58% <= 2 milliseconds
100.00% <= 2 milliseconds
31555.70 requests per second

====== LRANGE_300 (first 300 elements) ======
  100000 requests completed in 7.53 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

0.00% <= 1 milliseconds
76.90% <= 2 milliseconds
98.08% <= 3 milliseconds
99.79% <= 4 milliseconds
99.93% <= 5 milliseconds
99.96% <= 6 milliseconds
100.00% <= 7 milliseconds
100.00% <= 7 milliseconds
13276.69 requests per second

====== LRANGE_500 (first 450 elements) ======
  100000 requests completed in 10.67 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

0.01% <= 1 milliseconds
2.44% <= 2 milliseconds
84.68% <= 3 milliseconds
95.67% <= 4 milliseconds
99.04% <= 5 milliseconds
99.67% <= 6 milliseconds
99.85% <= 7 milliseconds
99.89% <= 8 milliseconds
99.94% <= 9 milliseconds
99.96% <= 10 milliseconds
99.98% <= 11 milliseconds
100.00% <= 11 milliseconds
9371.19 requests per second

====== LRANGE_600 (first 600 elements) ======
  100000 requests completed in 12.90 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

0.00% <= 1 milliseconds
0.18% <= 2 milliseconds
32.05% <= 3 milliseconds
93.59% <= 4 milliseconds
98.63% <= 5 milliseconds
99.52% <= 6 milliseconds
99.71% <= 7 milliseconds
99.81% <= 8 milliseconds
99.90% <= 9 milliseconds
99.97% <= 10 milliseconds
100.00% <= 10 milliseconds
7754.34 requests per second

====== MSET (10 keys) ======
  100000 requests completed in 2.16 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

66.62% <= 1 milliseconds
98.67% <= 2 milliseconds
99.71% <= 3 milliseconds
99.89% <= 4 milliseconds
100.00% <= 5 milliseconds
100.00% <= 5 milliseconds
46210.72 requests per second


[root@h102 src]# 
~~~

---

### 指定主机，端口，请求数，并发数测试

~~~
[root@h102 src]# ./redis-benchmark -h localhost -p 6379 -n 100000 -c 20
====== PING_INLINE ======
  100000 requests completed in 1.61 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.86% <= 1 milliseconds
100.00% <= 2 milliseconds
100.00% <= 2 milliseconds
62111.80 requests per second

====== PING_BULK ======
  100000 requests completed in 1.73 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.93% <= 1 milliseconds
100.00% <= 1 milliseconds
57836.90 requests per second

====== SET ======
  100000 requests completed in 1.80 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.74% <= 1 milliseconds
99.88% <= 2 milliseconds
99.96% <= 3 milliseconds
99.98% <= 4 milliseconds
99.98% <= 6 milliseconds
99.98% <= 10 milliseconds
100.00% <= 10 milliseconds
55463.12 requests per second

====== GET ======
  100000 requests completed in 1.66 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.91% <= 1 milliseconds
99.99% <= 2 milliseconds
100.00% <= 2 milliseconds
60277.27 requests per second

====== INCR ======
  100000 requests completed in 1.76 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.90% <= 1 milliseconds
99.97% <= 2 milliseconds
99.98% <= 4 milliseconds
100.00% <= 5 milliseconds
100.00% <= 5 milliseconds
56721.50 requests per second

====== LPUSH ======
  100000 requests completed in 1.67 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.97% <= 1 milliseconds
100.00% <= 1 milliseconds
59916.12 requests per second

====== LPOP ======
  100000 requests completed in 1.68 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.80% <= 1 milliseconds
99.99% <= 2 milliseconds
100.00% <= 2 milliseconds
59665.87 requests per second

====== SADD ======
  100000 requests completed in 1.70 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 1 milliseconds
99.98% <= 2 milliseconds
99.99% <= 3 milliseconds
100.00% <= 3 milliseconds
58927.52 requests per second

====== SPOP ======
  100000 requests completed in 1.74 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.92% <= 1 milliseconds
99.97% <= 2 milliseconds
100.00% <= 3 milliseconds
57339.45 requests per second

====== LPUSH (needed to benchmark LRANGE) ======
  100000 requests completed in 1.93 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.52% <= 1 milliseconds
99.89% <= 2 milliseconds
99.97% <= 3 milliseconds
100.00% <= 3 milliseconds
51679.59 requests per second

====== LRANGE_100 (first 100 elements) ======
  100000 requests completed in 3.38 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

98.35% <= 1 milliseconds
99.73% <= 2 milliseconds
99.89% <= 3 milliseconds
99.93% <= 4 milliseconds
99.96% <= 5 milliseconds
99.97% <= 6 milliseconds
99.97% <= 8 milliseconds
99.97% <= 9 milliseconds
99.99% <= 14 milliseconds
100.00% <= 14 milliseconds
29542.10 requests per second

====== LRANGE_300 (first 300 elements) ======
  100000 requests completed in 7.37 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

93.16% <= 1 milliseconds
99.72% <= 2 milliseconds
99.90% <= 3 milliseconds
99.96% <= 4 milliseconds
100.00% <= 4 milliseconds
13561.16 requests per second

====== LRANGE_500 (first 450 elements) ======
  100000 requests completed in 10.17 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

56.63% <= 1 milliseconds
99.41% <= 2 milliseconds
99.86% <= 3 milliseconds
99.95% <= 4 milliseconds
99.98% <= 5 milliseconds
100.00% <= 6 milliseconds
100.00% <= 7 milliseconds
100.00% <= 7 milliseconds
9828.98 requests per second

====== LRANGE_600 (first 600 elements) ======
  100000 requests completed in 12.77 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

6.56% <= 1 milliseconds
97.97% <= 2 milliseconds
99.68% <= 3 milliseconds
99.89% <= 4 milliseconds
99.96% <= 5 milliseconds
99.96% <= 6 milliseconds
99.98% <= 7 milliseconds
99.99% <= 8 milliseconds
99.99% <= 9 milliseconds
100.00% <= 10 milliseconds
100.00% <= 10 milliseconds
7833.31 requests per second

====== MSET (10 keys) ======
  100000 requests completed in 2.17 seconds
  20 parallel clients
  3 bytes payload
  keep alive: 1

99.35% <= 1 milliseconds
99.94% <= 2 milliseconds
99.96% <= 3 milliseconds
99.99% <= 6 milliseconds
99.99% <= 7 milliseconds
100.00% <= 7 milliseconds
46040.52 requests per second


[root@h102 src]# 
~~~

---

### 使用SET测试来填充1000000keys

~~~
[root@h102 src]# ./redis-benchmark -t set -n 1000000 -r 100000000
====== SET ======
  1000000 requests completed in 19.13 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

92.04% <= 1 milliseconds
99.47% <= 2 milliseconds
99.84% <= 3 milliseconds
99.91% <= 4 milliseconds
99.93% <= 5 milliseconds
99.93% <= 6 milliseconds
99.94% <= 8 milliseconds
99.95% <= 9 milliseconds
99.98% <= 10 milliseconds
99.98% <= 11 milliseconds
99.99% <= 12 milliseconds
99.99% <= 13 milliseconds
99.99% <= 14 milliseconds
99.99% <= 15 milliseconds
99.99% <= 32 milliseconds
100.00% <= 37 milliseconds
100.00% <= 69 milliseconds
100.00% <= 70 milliseconds
100.00% <= 70 milliseconds
52271.18 requests per second


[root@h102 src]# 
~~~

---

### 使用CSV的输出来测试本地服务的几种调用


~~~
[root@h102 src]# ./redis-benchmark -t ping,set,get -n 100000 --csv
"PING_INLINE","59559.26"
"PING_BULK","54555.38"
"SET","37078.23"
"GET","57670.13"
[root@h102 src]# 
~~~

---

### 测试指定的命令

~~~
[root@h102 src]# ./redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0
====== eval return redis.call("ping") 0 ======
  10000 requests completed in 0.28 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

30.19% <= 1 milliseconds
95.13% <= 2 milliseconds
98.26% <= 3 milliseconds
98.97% <= 4 milliseconds
99.51% <= 7 milliseconds
100.00% <= 7 milliseconds
36101.08 requests per second

[root@h102 src]# 
~~~

---

### 使用10000个随机元素填充一个列表

~~~
[root@h102 src]# ./redis-benchmark -r 10000 -n 10000 lpush mylist __rand_int__
====== lpush mylist __rand_int__ ======
  10000 requests completed in 0.19 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

95.37% <= 1 milliseconds
98.58% <= 2 milliseconds
98.94% <= 3 milliseconds
99.08% <= 5 milliseconds
99.47% <= 6 milliseconds
99.51% <= 12 milliseconds
99.75% <= 13 milliseconds
100.00% <= 13 milliseconds
52910.05 requests per second

[root@h102 src]# 
~~~


---

# 命令汇总


* wget http://download.redis.io/releases/redis-3.0.5.tar.gz
* tar zxvf redis-3.0.5.tar.gz 
* cd redis-3.0.5
* make
* ./redis-benchmark -h
* ./redis-benchmark
* ./redis-benchmark -t set -r 100000 -n 1000000
* ./redis-benchmark -n 1000000 -t set,get -P 16 -q
* ./redis-benchmark -q 
* ./redis-benchmark -h localhost -p 6379 -n 100000 -c 20
* ./redis-benchmark -t set -n 1000000 -r 100000000
* ./redis-cli  info \| grep db0
* ./redis-benchmark -t ping,set,get -n 100000 --csv
* ./redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0
* ./redis-benchmark -r 10000 -n 10000 lpush mylist `__rand_int__`




---

[redis-benchmark]:http://redis.io/topics/benchmarks
[redis]:http://redis.io/


