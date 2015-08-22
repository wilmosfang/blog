---
layout: default
title: Redis 复制
comments: true
---

---

前言
=

[Redis][redis]有一个和Mysql一样的特性:[复制][replication]

下面是我根据官方文档的学习笔记，进行的整理与分享


---

[redis]:http://redis.io/
[replication]:http://redis.io/topics/replication

安全隐患
-

* 1.强烈建议开启master的持久特性
* 2.如果由于性能或延时方面的考虑，没有条件开启持久，必须确保避免master的自动重启

>自动重启会有两个后果
>
> * 1.丢失数据
> 
> * 2.同步丢失数据后的数据库 
> 
> >如果开启了持久，则丢失未保存的数据，如果没开启持久，则丢失所有数据，并且抹掉了备库中的数据
> 
> 任何时候数据安全是最重要的，master的自动重启必须关掉

---

复制原理
-

* 1.如果设置了一个Slave，无论是第一次连接还是重连到Master，它都会发出一个SYNC命令；
* 2.当Master收到SYNC命令之后，会做两件事：

> * a) Master执行BGSAVE，即在后台保存数据到磁盘（rdb快照文件）；
> * b) Master同时将新收到的写入和修改数据集的命令存入缓冲区（非查询类）；

* 3.当Master在后台把数据保存到快照文件完成之后，Master会把这个快照文件传送给Slave，而Slave则把内存清空后，加载该文件到内存中；
* 4.而Master也会把此前收集到缓冲区中的命令，通过Reids命令协议形式转发给Slave，Slave执行这些命令，实现和Master的同步；
* 5.Master/Slave此后会不断通过异步方式进行命令的同步，达到最终数据的同步一致；
* 6.需要注意的是Master和Slave之间一旦发生重连都会引发全量同步操作。但在2.8之后版本，也可能是部分同步操作。

---

增量同步
-

从 Redis 2.8 开始， 在网络连接短暂性失效之后， 主从服务器可以尝试继续执行原有的复制进程（process）， 而不一定要执行完整重同步操作。

这个特性需要主服务器为被发送的复制流创建一个内存缓冲区（in-memory backlog）， 并且主服务器和所有从服务器之间都记录一个复制偏移量（replication offset）和一个主服务器 ID （master run id）， 当出现网络连接断开时， 从服务器会重新连接， 并且向主服务器请求继续执行原来的复制进程：

* 如果从服务器记录的主服务器 ID 和当前要连接的主服务器的 ID 相同， 并且从服务器记录的偏移量所指定的数据仍然保存在主服务器的复制流缓冲区里面， 那么主服务器会向从服务器发送断线时缺失的那部分数据， 然后复制工作可以继续执行。
* 否则的话， 从服务器就要执行完整重同步操作。

Redis 2.8 的这个部分重同步特性会用到一个新增的 PSYNC 内部命令， 而 Redis 2.8 以前的旧版本只有 SYNC 命令， 不过， 只要从服务器是 Redis 2.8 或以上的版本， 它就会根据主服务器的版本来决定到底是使用 PSYNC 还是 SYNC ：

* 如果主服务器是 Redis 2.8 或以上版本，那么从服务器使用 PSYNC 命令来进行同步。
* 如果主服务器是 Redis 2.8 之前的版本，那么从服务器使用 SYNC 命令来进行同步。

---

无盘同步
-

正常情况下同步操作都会在master本地磁盘中创建一个RDB文件，然后把这个RDB文件传给slave以完成同步操作

这个操作会给master造成很大的io压力，2.8.18之后支持一种特性，可以省掉存盘的中间动作，而直接通过网络传输RDB给slave，不过目前这个特性还处于实验性阶段

关于无盘同步，目前只有两个参数进行控制：

**repl-diskless-sync**

**repl-diskless-sync-delay**

关于这两个参数的说明, **redis.conf** 中已经有了详细的说明：

{% highlight bash %}
# Replication SYNC strategy: disk or socket.
#
# -------------------------------------------------------
# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
# -------------------------------------------------------
#
# New slaves and reconnecting slaves that are not able to continue the replication
# process just receiving differences, need to do what is called a "full
# synchronization". An RDB file is transmitted from the master to the slaves.
# The transmission can happen in two different ways:
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the slaves incrementally.
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to slave sockets, without touching the disk at all.
#
# With disk-backed replication, while the RDB file is generated, more slaves
# can be queued and served with the RDB file as soon as the current child producing
# the RDB file finishes its work. With diskless replication instead once
# the transfer starts, new slaves arriving will be queued and a new transfer
# will start when the current one terminates.
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple slaves
# will arrive and the transfer can be parallelized.
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better.
repl-diskless-sync no

# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the slaves.
#
# This is important since once the transfer starts, it is not possible to serve
# new slaves arriving, that will be queued for the next RDB transfer, so the server
# waits a delay in order to let more slaves arrive.
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP.
repl-diskless-sync-delay 5

{% endhighlight %}


---

配置方法 
-

配置方法有两种：

* 1.直接使用slaveof 命令
* 2.写在在配置文件中


### 使用 **slaveof** 命令


**master** 上没有 **rdb** 文件

{% highlight bash %}
[root@m1 ~]# ls
anaconda-ks.cfg  Downloads           log       Public              redis.conf      redis_slave_on_m1.conf  Videos
Desktop          install.log         Music     redis-3.0.0         redis.log       Templates
Documents        install.log.syslog  Pictures  redis-3.0.0.tar.gz  redis-new.conf  tmp
[root@m1 ~]# redis-cli 
127.0.0.1:6379> keys * 
1) "b"
2) "a"
3) "c"
4) "8"
5) "9"
6) "d"
127.0.0.1:6379>

{% endhighlight %}

在 **slave** 上执行 **slaveof 192.168.75.11 6379** 命令，可以看到空白的库有了数据 

{% highlight bash %}
[root@m2 tmp]# telnet localhost 6379 
Trying ::1...
Connected to localhost.
Escape character is '^]'.
keys * 
*0
keys * 
*0
slaveof 192.168.75.11 6379
+OK
keys * 
*6
$1
d
$1
c
$1
9
$1
b
$1
a
$1
8
{% endhighlight %}

**master** 上也多了一个 **dump.rdb** 文件

{% highlight bash %}
127.0.0.1:6379> keys * 
1) "b"
2) "a"
3) "c"
4) "8"
5) "9"
6) "d"
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> quit
[root@m1 ~]# ls
anaconda-ks.cfg  Downloads    install.log.syslog  Pictures     redis-3.0.0.tar.gz  redis-new.conf          tmp
Desktop          dump.rdb     log                 Public       redis.conf          redis_slave_on_m1.conf  Videos
Documents        install.log  Music               redis-3.0.0  redis.log           Templates
[root@m1 ~]# 
{% endhighlight %}

**slave** 在退出后，也会在本地产生一个 **rdb** 文件，这个文件就是库的一个快照，下次启动后用来恢复内存状态

{% highlight bash %}
shutdown   
Connection closed by foreign host.
[root@m2 tmp]# ps -ef | grep redis
root      5052  2119  0 14:50 pts/0    00:00:00 grep redis
[root@m2 tmp]# ls
dump.rdb                                 my.cnf              percona-release-0.1-3.noarch.rpm
epel-release-6-8.noarch2.rpm             my.cnf.leopard      redis-3.0.0
mha4mysql-manager-0.53-0.el6.noarch.rpm  my.cnf.leopard.bak  redis-3.0.0.tar.gz
mha4mysql-manager-0.53.tar.gz            my.cnfrrrccc        redis.conf
mha4mysql-node-0.53-0.el6.noarch.rpm     my.cnf.tt           redisnew.conf
[root@m2 tmp]# 
{% endhighlight %}

如果删掉这个 **rdb** 文件，数据就会丢失

> 同步实际上是从 **client** 端发送了一个 **SYNC** 命令

{% highlight bash %}
[root@m2 tmp]# rm dump.rdb 
rm: remove regular file `dump.rdb'? y
[root@m2 tmp]# ls
epel-release-6-8.noarch2.rpm             my.cnf              my.cnf.tt                         redis.conf
mha4mysql-manager-0.53-0.el6.noarch.rpm  my.cnf.leopard      percona-release-0.1-3.noarch.rpm  redisnew.conf
mha4mysql-manager-0.53.tar.gz            my.cnf.leopard.bak  redis-3.0.0
mha4mysql-node-0.53-0.el6.noarch.rpm     my.cnfrrrccc        redis-3.0.0.tar.gz
[root@m2 tmp]# redis-server redisnew.conf 
[root@m2 tmp]# telnet localhost 6379
Trying ::1...
Connected to localhost.
Escape character is '^]'.
keys * 
*0
keys *        
*0
sync 
$18
REDIS0006³C띲V


keys *    
*0
*1
$4
PING
keys * 
*0
*1
$4
PING
*1
$4
PING
*1
$4
PING
keys * 
*0
*1
$4
PING
quit
+OK
Connection closed by foreign host.
[root@m2 tmp]# ls
dump.rdb                                 my.cnf              percona-release-0.1-3.noarch.rpm
epel-release-6-8.noarch2.rpm             my.cnf.leopard      redis-3.0.0
mha4mysql-manager-0.53-0.el6.noarch.rpm  my.cnf.leopard.bak  redis-3.0.0.tar.gz
mha4mysql-manager-0.53.tar.gz            my.cnfrrrccc        redis.conf
mha4mysql-node-0.53-0.el6.noarch.rpm     my.cnf.tt           redisnew.conf
[root@m2 tmp]# 
{% endhighlight %}


### 写在配置文件中

上面一种方法会在当前运行状态中生效，一旦重启，将不再同步，要想在重启后依然有效，只用在配置文件中加下面一行

{% highlight bash %}
[root@m2 tmp]# grep slaveof redis.conf 
# Master-Slave replication. Use slaveof to make a Redis instance a copy of
slaveof m1 6379
[root@m2 tmp]# 
{% endhighlight %}

---

Slave的只读特性
-

可能是出于安全方面的考，从 Redis 2.6 开始， 从服务器支持只读模式， 并且该模式为从服务器的默认模式。

只读从服务器会拒绝执行任何写命令， 所以不会出现因为操作失误而将数据不小心写入到了从服务器的情况。

### 只读特性

{% highlight bash %}
[root@m2 tmp]# redis-cli 
127.0.0.1:6379> KEYS * 
(empty list or set)
127.0.0.1:6379> SLAVEOF m1 6379
OK 
127.0.0.1:6379> KEYS * 
1) "b"
2) "8"
3) "d"
4) "c"
5) "a"
6) "9"
127.0.0.1:6379> set jj jj 
(error) READONLY You can't write against a read only slave.
127.0.0.1:6379> 
{% endhighlight %}

该模式为从服务器的默认模式

{% highlight bash %}
[root@m2 tmp]# grep slave-read-only redisnew.conf
slave-read-only yes
[root@m2 tmp]# 
{% endhighlight %}

停止同步

{% highlight bash %}
127.0.0.1:6379> config get slave*
1) "slave-priority"
2) "100"
3) "slave-serve-stale-data"
4) "yes"
5) "slave-read-only"
6) "yes"
7) "slaveof"
8) "m1 6379"
127.0.0.1:6379> CONFIG SET slaveof nil
(error) ERR Unsupported CONFIG parameter: slaveof
127.0.0.1:6379> SLAVEOF no one
OK
127.0.0.1:6379> config get slaveof
1) "slaveof"
2) ""
127.0.0.1:6379> 
{% endhighlight %}

停止同步后又可以写入了

{% highlight bash %}
127.0.0.1:6379> config get slaveof
1) "slaveof"
2) ""
127.0.0.1:6379> set abc iii
OK
127.0.0.1:6379> get abc 
"iii"
127.0.0.1:6379>  
{% endhighlight %}

但是这个特性可以被在线修改

### 在线关闭Slave只读

{% highlight bash %}
127.0.0.1:6379> SLAVEOF m1 6379
OK
127.0.0.1:6379> set iii abc 
(error) READONLY You can't write against a read only slave.
127.0.0.1:6379> CONFIG GET slave-read-only
1) "slave-read-only"
2) "yes"
127.0.0.1:6379> CONFIG SET slave-read-only no
OK
127.0.0.1:6379> CONFIG GET slave-read-only
1) "slave-read-only"
2) "no"
127.0.0.1:6379> set iii abc 
OK
127.0.0.1:6379> get iii 
"abc"
127.0.0.1:6379> 
{% endhighlight %}

> 既然从服务器上的写数据会被重同步数据覆盖， 也可能在从服务器重启时丢失， 那么为什么要让一个从服务器变得可写呢？
原因是， 一些不重要的临时数据， 仍然是可以保存在从服务器上面的。 比如说， 客户端可以在从服务器上保存主服务器的可达性（reachability）信息， 从而实现故障转移（failover）策略。



### 配置关闭只读

和复制一样，运行时的配置在重启后将丢失，要想重启后依然生效，得修改配置文件

{% highlight bash %}
[root@m2 tmp]# grep slave-read-only redisnew.conf
slave-read-only no
[root@m2 tmp]# 
{% endhighlight %}

---

连接认证
-

redis设计之初就没有过多的考虑安全问题，所以默认情况下，客户端的登录和slave的连接是不用密码认证的


### 在线设置密码

但是可以通过 **requirepass** 配置一下简单的认证

{% highlight bash %}
127.0.0.1:6379> CONFIG GET requ*
1) "requirepass"
2) ""
127.0.0.1:6379> exit
[root@m1 ~]# redis-cli 
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> CONFIG SET requirepass 123456
OK
127.0.0.1:6379> CONFIG GET requirepass
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 654321
(error) ERR invalid password
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> KEYS * 
1) "b"
2) "a"
3) "c"
4) "8"
5) "9"
6) "d"
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> 
{% endhighlight %}

这时master进行一个更新，slave无法同步

**master side**

{% highlight bash %}
127.0.0.1:6379> set ui ui 
OK
127.0.0.1:6379> 
{% endhighlight %}

**slave side**

{% highlight bash %}
127.0.0.1:6379> get ui 
(nil)
127.0.0.1:6379> CONFIG set masterauth 123456
OK
127.0.0.1:6379> get ui 
"ui"
127.0.0.1:6379> 
{% endhighlight %}

> 通过 **CONFIG set masterauth 123456** 可以配置master密码，以顺利进行连接

### 设置密码到配置文件

跟复制和只读设置一样，运行时的配置在重启后将丢失，要想重启后依然生效，得修改配置文件

**master side**

{% highlight bash %}
[root@m1 ~]# grep requirepass redis.conf 
# If the master is password protected (using the "requirepass" configuration
requirepass 123456
[root@m1 ~]# 
{% endhighlight %}

**slave side**

{% highlight bash %}
[root@m2 tmp]# grep masterauth redis.conf 
masterauth 123456
[root@m2 tmp]# 
{% endhighlight %}

---

限制写操作
-

下面两个参数是用来限制 **master** 写操作的:

{% highlight bash %}
127.0.0.1:6379> CONFIG GET min*
1) "min-slaves-to-write"
2) "0"
3) "min-slaves-max-lag"
4) "10"
127.0.0.1:6379> 
{% endhighlight %}

由于暂时用不到这个特性，所以，只是对文档进行翻译，没有用试验验证

从 Redis 2.8 开始， 为了保证数据的安全性， 可以通过配置， 让主服务器只在有至少 N 个当前已连接从服务器的情况下， 才执行写命令; 不过， 因为 Redis 使用异步复制， 所以主服务器发送的写数据并不一定会被从服务器接收到， 因此， 数据丢失的可能性仍然是存在的。

以下是这个特性的运作逻辑：

* 从服务器以每秒一次的频率 PING 主服务器一次， 并报告复制流的处理情况。
* 主服务器会记录各个从服务器最后一次向它发送 PING 的时间。
* 用户可以通过配置， 指定网络延迟的最大值 min-slaves-max-lag ， 以及执行写操作所需的至少从服务器数量 min-slaves-to-write 。
* 如果至少有 min-slaves-to-write 个从服务器， 并且这些服务器的延迟值都少于 min-slaves-max-lag 秒， 那么主服务器就会执行客户端请求的写操作。
* 另一方面， 如果条件达不到 min-slaves-to-write 和 min-slaves-max-lag 所指定的条件， 那么写操作就不会被执行， 主服务器会向请求执行写操作的客户端返回一个错误。

> 可以将这个特性看作 CAP 理论中的 C 的条件放宽版本： 尽管不能保证写操作的持久性， 但起码丢失数据的窗口会被严格限制在指定的秒数中。


以下是这个特性的两个选项和它们所需的参数：

* min-slaves-to-write <number of slaves>

* min-slaves-max-lag <number of seconds>

详细的信息可以参考 Redis 源码中附带的 redis.conf 示例文件。


---

总结
=

[Redis][redis]有一个和mysql类似的[复制][replication]机制，可以进行如下同步方式：

* (SYNC)完全同步
* (PSYNC)增量同步(2.8之后的特性)
* (diskless)无盘同步(2.8.18之后支持一种特性，目前还处于实验性阶段)

**Note:** 同步设置后，避免master的自动重启


几个重要命令:

* slaveof 192.168.75.11 6379
* SLAVEOF no one
* telnet localhost 6379 
* sync
* ping
* keys *
* config get slave*
* CONFIG SET slave-read-only no
* CONFIG SET requirepass 123456
* CONFIG set masterauth 123456




