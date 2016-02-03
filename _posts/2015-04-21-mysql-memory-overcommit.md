---
layout: post
title: Percona Server  5.1 内存过量消耗分析
categories: linux mysql troubleshooting
wc: 301 894 10541
excerpt: follow me
comments: true
---

---

前言
=

一次巡检过程中发现数据库使用内存有些过量，**innodb_buffer_pool_size** 设置值为 **20G** ，但实际物理内存消耗为 **37G** ，总虚拟内存消耗达 **42G** ，直接导致监控报警，于是开启了一次内存使用探究之旅，整理出来和大家分享一下


---

# 概要

* TOC
{:toc}

---


发现问题
-

巡检过程中发现mysql内存使用过量

下面是 **top** 出来的信息，只截取了关键部分

{% highlight bash %}
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                   
14769 mysql     15   0 42.1g  37g 4792 S 58.1 79.5  25660:41 mysqld  
{% endhighlight %}

**innodb_buffer_pool_size** 只设定了 **20G**

{% highlight bash %}
[root@abc ~]# grep  innodb_buffer_pool_size /etc/my.cnf 
innodb_buffer_pool_size = 20G
[root@abc ~]# 
{% endhighlight %}

加上其它七七八八的参数总量也绝不会超过25G，那怎么会有那么大的差距呢？


---


分析问题
-

初步推断有两种情况：


* 参数配置不当
* 内存泄漏


> 关于参数配置不当，我分析完各种buffer，cache参数配置后没有发现异常或特别严重的错误，于是尝试从内存泄漏的角度来寻找突破口

---

分析工具
-

**pmap** : 用来生成一个进程的内存使用报表

> The pmap command reports the memory map of a process or processes.

**[pt-config-diff][pt-config-diff]** : 用来比较Mysql 配置文件的差异

>pt-config-diff diffs MySQL configuration files and server variables. CONFIG can
be a filename or a DSN.  At least two CONFIG sources must be given.  Like
standard Unix diff, there is no output if there are no differences. 


### 使用 **pmap** 生成mysql内存使用报表

{% highlight bash %}
[root@abc ~]# pmap -x 14769 
14769:   /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --user=mysql --log-error=/var/lib/mysql/abc.err --open-files-limit=8192 --pid-file=/var/lib/mysql/abc.pid --socket=/var/lib/mysql/mysql.sock --port=3306
Address           Kbytes     RSS   Dirty Mode   Mapping
0000000000400000    7484    3704       0 r-x--  mysqld
0000000000d4e000     412     216     116 rw---  mysqld
0000000000db5000     112     108     108 rw---    [ anon ]
0000000000fb4000     516       4       0 rw---  mysqld
000000000adbf000 9286784 7807156 7807116 rw---    [ anon ]
00000035b3a00000     112     100       0 r-x--  ld-2.5.so
00000035b3c1c000       4       4       4 r----  ld-2.5.so
00000035b3c1d000       4       4       4 rw---  ld-2.5.so
00000035b3e00000    1340     528       0 r-x--  libc-2.5.so
00000035b3f4f000    2048       0       0 -----  libc-2.5.so
00000035b414f000      16      16       4 r----  libc-2.5.so
00000035b4153000       4       4       4 rw---  libc-2.5.so
...
...
00002ae84c000000   63804    2524    2524 rw---    [ anon ]
00002ae84fe4f000    1732       0       0 -----    [ anon ]
00002ae850000000   26976    2788    2788 rw---    [ anon ]
00002ae851a58000   38560       0       0 -----    [ anon ]
00007fffe4cbd000      84      16      16 rw---    [ stack ]
00007fffe4ce1000      12       4       0 r-x--    [ anon ]
ffffffffff600000    8192       0       0 -----    [ anon ]
----------------  ------  ------  ------
total kB        44114104 39346676 39341556
{% endhighlight %}

对这个报表作一个排序，会获得更多信息

{% highlight bash %}
[root@abc ~]# pmap -x 14769 | sort -nk 2 
----------------  ------  ------  ------
14769:   /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --user=mysql --log-error=/var/lib/mysql/abc.err --open-files-limit=8192 --pid-file=/var/lib/mysql/abc.pid --socket=/var/lib/mysql/mysql.sock --port=3306
Address           Kbytes     RSS   Dirty Mode   Mapping
total kB        44112060 39341308 39336188
00000035b3c1c000       4       4       4 r----  ld-2.5.so
00000035b3c1d000       4       4       4 rw---  ld-2.5.so
00000035b4153000       4       4       4 rw---  libc-2.5.so
...
...
00002ae830000000   65536   65508   65508 rw---    [ anon ]
00002ae838000000   65536   65144   65144 rw---    [ anon ]
00002ae4f23f5000   85088   29052   29028 rw---    [ anon ]
00002ae6c0000000  131052   90044   90044 rw---    [ anon ]
00002ae73c000000  131064   86360   86360 rw---    [ anon ]
000000000adbf000 9284740 7801584 7801544 rw---    [ anon ]
00002adf8a981000 22243496 22221844 22221652 rw---    [ anon ]
[root@abc ~]# 
{% endhighlight %}

我注意到最后两项(内存消耗最大两项)，分别是 **21.19G** (22221844K/1024/1024) 和 **7.44G** (7801584K/1024/1024)怎么会有这么多的内存消耗呢~~非常奇怪!

第一项 **21.19G** 合理的解释是配置了 **20G** 的 **innodb_buffer_pool_size**

第二项 **7.44G** 从何解释呢？


### 使用 **pt-config-diff** 进行配置比较

还有一个数据库可以正常工作，于是我打算拿它作为基准进行对比

{% highlight bash %}
[root@abc ~]# pt-config-diff h=10.0.0.1 h=10.0.0.2 --user=testuser --password=testuser
12 config differences
Variable                  abc            def
========================= ========================= =========================
general_log_file          /var/lib/mysql/x-... /var/lib/mysql/x-...
hostname                  abc            def
innodb_buffer_pool_size   21474836480               9663676416
innodb_ibuf_max_size      10737418240               4831838208
log_error                 /var/lib/mysql/x-... /var/lib/mysql/x-...
pid_file                  /var/lib/mysql/x-... /var/lib/mysql/x-...
query_cache_size          268435456                 134217728
server_id                 3                         16
slave_load_tmpdir         /tmp                      /mysqldata/mysqltmp
slave_skip_errors         ALL                       OFF
slow_query_log_file       abc-slow.log   def-slow.log
tmpdir                    /tmp                      /mysqldata/mysqltmp
[root@abc ~]#
{% endhighlight %}

由于 **innodb_buffer_pool_size** 和 **query_cache_size** 都是我手动配置的，所以这个差异报告让我立刻注意到了 **innodb_ibuf_max_size**

{% highlight bash %}
[root@abc ~]# grep  query_cache_size /etc/my.cnf 
query_cache_size = 256M
[root@abc ~]# grep  innodb_buffer_pool_size /etc/my.cnf
innodb_buffer_pool_size = 20G
[root@abc ~]# grep  innodb_ibuf_max_size  /etc/my.cnf
[root@abc ~]# 
...
[root@def ~]# grep  query_cache_size /etc/my.cnf
query_cache_size = 128M
[root@def ~]# grep  innodb_buffer_pool_size /etc/my.cnf
#innodb_buffer_pool_size = 12G
innodb_buffer_pool_size = 9G
[root@def ~]# grep  innodb_ibuf_max_size  /etc/my.cnf
[root@def ~]# 
{% endhighlight %}

**innodb_ibuf_max_size** 这个参数我并没配置，但它默认存在于我的系统中

{% highlight bash %}
mysql> show variables like "%innodb_ibuf_max_size%";
+----------------------+-------------+
| Variable_name        | Value       |
+----------------------+-------------+
| innodb_ibuf_max_size | 10737418240 |
+----------------------+-------------+
1 row in set (0.00 sec)

mysql> 
{% endhighlight %}

---

**innodb_ibuf_max_size** 参数意义
-

因为这个参数占用了那么多的内存

{% highlight bash %}
21474836480/10737418240
2.000000000
9663676416/4831838208
2.000000000
{% endhighlight %}

发现都正好是 **innodb_buffer_pool_size** 的一半，于是开始查文档弄清楚 **innodb_ibuf_max_size** 参数的意义

Mysql的官方手册中没有找到关于这个参数的任何描述信息，其实根本就不存在这个参数

去百度里google一把，总会有收获，这次也不例外

发现，原来 **[innodb_ibuf_max_size][innodb_ibuf_max_size]** 是 **[percona][percona]** 自己加的参数，用来提升 **insert** 操作性能的


Variable **[innodb_ibuf_max_size][innodb_ibuf_max_size]**


Item     | Value
-------- | ---
Command Line|Yes
Config File|Yes
Scope|Global
Dynamic|No
Variable Type|Numeric
Default Value|Half the size of the InnoDB buffer pool
Range|0 - Half the size of the InnoDB buffer pool
Units|Bytes


>This variable specifies the maximum size of the insert buffer. By default the insert buffer is half the size of the buffer pool so if you have a very large buffer pool, the insert buffer will be very large too and you may want to restrict its size with this variable.
Setting this variable to 0 is equivalent to disabling the insert buffer. But then all changes to secondary indexes will be performed synchronously which will probably cause performance degradation. Likewise a too small value can hurt performance.
If you have very fast storage (ie storage with RAM-level speed, not just a RAID with fast disks), a value of a few MB may be the best choice for maximum performance.

---

解决办法
-

由于它并不能动态进行调整，所以必须安排一次数据库的启停，在配置文件中对 **innodb_ibuf_max_size** 进行限定就可以有效解决此问题



---
[pt-config-diff]:http://www.percona.com/doc/percona-toolkit/2.2/pt-config-diff.html
[innodb_ibuf_max_size]:http://www.percona.com/doc/percona-server/5.5/scalability/innodb_insert_buffer.html
[percona]:http://www.percona.com/


后记
=

其实这个原因的定位并不像这篇文档中的流程一样那么顺利，先后我尝试了几个方面：

* 使用工具来查看有没有严重的参数配置错误

{% highlight bash %}
pt-variable-advisor 10.0.0.1   --user testuser --password testuser
pt-mysql-summary  --user=testuser --password=testuser
{% endhighlight %}

* 查看分析各种buffer，cache ，Qcache ，connections ，Thread ，sort 参数配置与比值，企图找出不合理的地方

>不得不吐槽一下，网上太多复制粘贴的文档都不具备指导意义，或者也跟本没讲出什么所以然来)

* 大量对比不同库的配置文件，想找出不同配置的不同影响






---

总结
=

几个重要命令：

* pmap -x 14769 \| sort -nk 2
* pt-config-diff h=10.0.0.1 h=10.0.0.2 --user=testuser --password=testuser
* pt-variable-advisor 10.0.0.1   --user testuser --password testuser

几个重要参数：


* **innodb_buffer_pool_size** 
* **innodb_ibuf_max_size**



虽然最开始我的问题定位，锁定在配置上，但是我实在是找不出有什么参数配置问题，转而开始寻求内存泄漏方向的突破，但是最后的结果，还是回到了参数配置上，有点天意弄人的感觉，不过我从中的确学到了不少新的东西



