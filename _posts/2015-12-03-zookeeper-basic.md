---
layout: post
title: ZooKeeper 基础
author: wilmosfang
tags:  zookeeper
categories:  zookeeper
wc: 661 1965 19747
excerpt: zookeeper的下载，安装，服务启停，客户端连接，和常用命令
comments: true
---

# 前言


**[ZooKeeper][zookeeper]** 是一个高性能，开源分布式应用协调服务。实现了一个配置管理中心，可以将配置信息分发到各个服务节点上，并保证信息的正确性和一致性

>ZooKeeper is a distributed, open-source coordination service for distributed applications

Zookeeper 特点：

* 顺序一致性：按照客户端发送请求的顺序更新数据。
* 原子性：更新要么成功，要么失败，不会出现部分更新。
* 单一性 ：无论客户端连接哪个server，都会看到同一个视图。
* 可靠性：一旦数据更新成功，将一直保持，直到新的更新。
* 及时性：客户端会在一个确定的时间内得到最新的数据。

应用场景：

* 数据发布与订阅（配置中心）
* 负载均衡
* 命名服务(Naming Service)
* 分布式通知/协调
* 分布式锁
* 分布式队列
* 集群管理与Master选举

分布式应用中，**[ZooKeeper][zookeeper]** 使用得非常广泛，下面分享一下它的基本操作，详细可以参考 [官方文档][zookeeper]

> **Tip:** 当前版本为 **Release 3.4.6(stable)**


---


# 概要

* TOC
{:toc}




---

## 下载

这是 **[版本信息][zookeeper_download]**  和 **[镜像站点][zookeeper_mirror]**

使用下列方法进行下载

~~~
[root@h101 zk]# wget  http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
--2015-12-02 21:04:50--  http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
Resolving apache.fayea.com... 119.6.56.18, 119.6.56.17
Connecting to apache.fayea.com|119.6.56.18|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 17699306 (17M) [application/x-gzip]
Saving to: “zookeeper-3.4.6.tar.gz”

100%[============================================================================================>] 17,699,306  1.74M/s   in 13s     

2015-12-02 21:05:04 (1.30 MB/s) - “zookeeper-3.4.6.tar.gz” saved [17699306/17699306]

[root@h101 zk]# ll 
total 17288
-rw-r--r-- 1 root root 17699306 Oct 31  2014 zookeeper-3.4.6.tar.gz
[root@h101 zk]# 
~~~

---

## 安装


解压

> **Tip:** 其实解压就可以直接运行了，确保有JVM
>
> ZooKeeper runs in Java, release 1.6 or greater (JDK 6 or greater)

~~~
[root@h101 zk]# tar -zxvf zookeeper-3.4.6.tar.gz 
zookeeper-3.4.6/
zookeeper-3.4.6/src/
zookeeper-3.4.6/src/lastRevision.sh
zookeeper-3.4.6/src/zookeeper.jute
zookeeper-3.4.6/src/c/
zookeeper-3.4.6/src/c/missing
zookeeper-3.4.6/src/c/ChangeLog
zookeeper-3.4.6/src/c/src/
...
...
zookeeper-3.4.6/lib/jdiff/
zookeeper-3.4.6/lib/jdiff/zookeeper_3.1.1.xml
zookeeper-3.4.6/lib/jdiff/zookeeper_3.4.6-SNAPSHOT.xml
zookeeper-3.4.6/lib/jdiff/zookeeper_3.4.6.xml
zookeeper-3.4.6/lib/slf4j-api-1.6.1.jar
zookeeper-3.4.6/lib/log4j-1.2.16.jar
zookeeper-3.4.6/lib/netty-3.7.0.Final.jar
zookeeper-3.4.6/lib/jline-0.9.94.LICENSE.txt
[root@h101 zk]# ls
zookeeper-3.4.6  zookeeper-3.4.6.tar.gz
[root@h101 zk]# 
~~~

默认配置

~~~
[root@h101 zk]# cd zookeeper-3.4.6
[root@h101 zookeeper-3.4.6]# ls
bin          conf        docs             lib          README_packaging.txt  src                      zookeeper-3.4.6.jar.md5
build.xml    contrib     ivysettings.xml  LICENSE.txt  README.txt            zookeeper-3.4.6.jar      zookeeper-3.4.6.jar.sha1
CHANGES.txt  dist-maven  ivy.xml          NOTICE.txt   recipes               zookeeper-3.4.6.jar.asc
[root@h101 zookeeper-3.4.6]# grep -v "^#" conf/zoo_sample.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper
clientPort=2181
[root@h101 zookeeper-3.4.6]# 
~~~

---

## 服务操作


### 启动服务

~~~
[root@h101 zookeeper-3.4.6]# cp conf/zoo_sample.cfg conf/zoo.cfg
[root@h101 zookeeper-3.4.6]# bin/zkServer.sh  start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zookeeper-3.4.6]# 
[root@h101 zookeeper-3.4.6]# netstat  -ant | grep 2181
tcp        0      0 :::2181                     :::*                        LISTEN      
[root@h101 zookeeper-3.4.6]# ps faux | grep zookeeper
root      5544  0.0  0.0 103256   832 pts/2    S+   21:59   0:00          \_ grep zookeeper
root      5516  1.5  2.4 2094296 46252 pts/2   Sl   21:58   0:01 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zk/zookeeper-3.4.6/bin/../build/classes:/root/zk/zookeeper-3.4.6/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
[root@h101 zookeeper-3.4.6]# 
~~~

---

### 服务状态


~~~
[root@h101 zookeeper-3.4.6]# bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: standalone
[root@h101 zookeeper-3.4.6]# 
~~~

### 重启服务


~~~
[root@h101 zookeeper-3.4.6]# bin/zkServer.sh restart
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zookeeper-3.4.6]# 
~~~



### 停止服务

~~~
[root@h101 zookeeper-3.4.6]# bin/zkServer.sh stop
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
[root@h101 zookeeper-3.4.6]# bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@h101 zookeeper-3.4.6]# ps faux | grep zookeeper
root      5853  0.0  0.0 103256   836 pts/2    S+   22:11   0:00          \_ grep zookeeper
[root@h101 zookeeper-3.4.6]#
~~~


---

## 客户端连接



使用 **`bin/zkCli.sh  -server 127.0.0.1:2181`** 进行连接

如果都是默认配置可以不用加

~~~
[root@h101 zookeeper-3.4.6]# bin/zkCli.sh  
Connecting to localhost:2181
2015-12-02 22:13:42,629 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2015-12-02 22:13:42,633 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=h101.temp
2015-12-02 22:13:42,634 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_65
2015-12-02 22:13:42,638 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2015-12-02 22:13:42,638 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.65.x86_64/jre
2015-12-02 22:13:42,639 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/root/zk/zookeeper-3.4.6/bin/../build/classes:/root/zk/zookeeper-3.4.6/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../conf:
2015-12-02 22:13:42,639 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2015-12-02 22:13:42,639 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2015-12-02 22:13:42,639 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2015-12-02 22:13:42,639 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2015-12-02 22:13:42,640 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2015-12-02 22:13:42,640 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-504.el6.x86_64
2015-12-02 22:13:42,640 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2015-12-02 22:13:42,640 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2015-12-02 22:13:42,641 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/root/zk/zookeeper-3.4.6
2015-12-02 22:13:42,661 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@7e7f0b4e
Welcome to ZooKeeper!
2015-12-02 22:13:42,713 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
2015-12-02 22:13:42,733 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session
JLine support is enabled
2015-12-02 22:13:42,770 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x151630a4f7e0001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
	connect host:port
	get path [watch]
	ls path [watch]
	set path data [version]
	rmr path
	delquota [-n|-b] path
	quit 
	printwatches on|off
	create [-s] [-e] path data acl
	stat path [watch]
	close 
	ls2 path [watch]
	history 
	listquota path
	setAcl path acl
	getAcl path
	sync path
	redo cmdno
	addauth scheme auth
	delete path [version]
	setquota -n|-b val path
[zk: localhost:2181(CONNECTED) 1] 
~~~

---

## 常用命令

### ls

类似linux里的ls

~~~
[zk: localhost:2181(CONNECTED) 15] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 16] 
~~~

---

### creat

创建znode

> **Note:** 必须以 **`/`** 开头

~~~
[zk: localhost:2181(CONNECTED) 17] create abc abc 
Command failed: java.lang.IllegalArgumentException: Path must start with / character
[zk: localhost:2181(CONNECTED) 18] create /abc abc
Created /abc
[zk: localhost:2181(CONNECTED) 19] ls / 
[abc, zookeeper]
[zk: localhost:2181(CONNECTED) 20] 
~~~

---

### get


获取znode内容

~~~
[zk: localhost:2181(CONNECTED) 30] get /

cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0xa
cversion = 2
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
[zk: localhost:2181(CONNECTED) 31] get /zookeeper

cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
[zk: localhost:2181(CONNECTED) 32] get /abc
abc
cZxid = 0xa
ctime = Thu Dec 03 14:28:05 CST 2015
mZxid = 0xa
mtime = Thu Dec 03 14:28:05 CST 2015
pZxid = 0xa
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 33] 
~~~

Attribute     | Comment
-------- | ---
czxid|The zxid of the change that caused this znode to be created.
mzxid|The zxid of the change that last modified this znode.
ctime|The time in milliseconds from epoch when this znode was created.
mtime|The time in milliseconds from epoch when this znode was last modified.
version|The number of changes to the data of this znode.
cversion|The number of changes to the children of this znode.
aversion|The number of changes to the ACL of this znode.
ephemeralOwner|The session id of the owner of this znode if the znode is an ephemeral node. If it is not an ephemeral node, it will be zero.
dataLength|The length of the data field of this znode.
numChildren|The number of children of this znode.




### set

设置znode内容

~~~
[zk: localhost:2181(CONNECTED) 33] set /abc def
cZxid = 0xa
ctime = Thu Dec 03 14:28:05 CST 2015
mZxid = 0xb
mtime = Thu Dec 03 14:35:59 CST 2015
pZxid = 0xa
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 34] get /abc
def
cZxid = 0xa
ctime = Thu Dec 03 14:28:05 CST 2015
mZxid = 0xb
mtime = Thu Dec 03 14:35:59 CST 2015
pZxid = 0xa
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 35] 
~~~

---

### delete

删除znode


> **Note:** **`/`** 无法被删除

~~~
[zk: localhost:2181(CONNECTED) 35] ls / 
[abc, zookeeper]
[zk: localhost:2181(CONNECTED) 36] delete /
Arguments are not valid : /
[zk: localhost:2181(CONNECTED) 37] delete /abc
[zk: localhost:2181(CONNECTED) 38] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 39] 
~~~

---

### history

可以像shell一样上下翻命令，也可使用history查看历史命令


~~~
[zk: localhost:2181(CONNECTED) 40] history
30 - get /
31 - get /zookeeper
32 - get /abc
33 - set /abc def
34 - get /abc
35 - ls / 
36 - delete /
37 - delete /abc
38 - ls /
39 - help 
40 - history
[zk: localhost:2181(CONNECTED) 41] 
~~~

---

### stat

感觉和get没有太大差别，只是少了内容数据

~~~
[zk: localhost:2181(CONNECTED) 43] stat /def
cZxid = 0xe
ctime = Thu Dec 03 14:57:27 CST 2015
mZxid = 0xe
mtime = Thu Dec 03 14:57:27 CST 2015
pZxid = 0xe
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 44] get /def 
def
cZxid = 0xe
ctime = Thu Dec 03 14:57:27 CST 2015
mZxid = 0xe
mtime = Thu Dec 03 14:57:27 CST 2015
pZxid = 0xe
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 45]
~~~


---

### ls2

ls2 就是携带了stat信息的ls

~~~
[zk: localhost:2181(CONNECTED) 60] ls /def
[]
[zk: localhost:2181(CONNECTED) 61] ls2 /def
[]
cZxid = 0xe
ctime = Thu Dec 03 14:57:27 CST 2015
mZxid = 0xf
mtime = Thu Dec 03 15:42:21 CST 2015
pZxid = 0xe
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 62]
~~~

---

### rmr 

递归删除

~~~
[zk: localhost:2181(CONNECTED) 74] ls /def
[abc]
[zk: localhost:2181(CONNECTED) 75] ls2 /def
[abc]
cZxid = 0xe
ctime = Thu Dec 03 14:57:27 CST 2015
mZxid = 0xf
mtime = Thu Dec 03 15:42:21 CST 2015
pZxid = 0x10
cversion = 1
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 1
[zk: localhost:2181(CONNECTED) 76] delete /def
Node not empty: /def
[zk: localhost:2181(CONNECTED) 77] rmr /def
[zk: localhost:2181(CONNECTED) 78] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 79] 
~~~

---

### close

主动断开当前连接

~~~
[zk: localhost:2181(CONNECTED) 14] ls /
[test, zookeeper]
[zk: localhost:2181(CONNECTED) 15] close 
2015-12-03 15:50:51,211 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x151665fb7430003 closed
[zk: localhost:2181(CLOSED) 16] 2015-12-03 15:50:51,211 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down

[zk: localhost:2181(CLOSED) 16] 
[zk: localhost:2181(CLOSED) 16] ls /
Not connected
[zk: localhost:2181(CLOSED) 17] 
~~~

---

### connect

连接服务

~~~
[zk: localhost:2181(CLOSED) 16] ls /
Not connected
[zk: localhost:2181(CLOSED) 17] connect localhost:2181
2015-12-03 16:40:11,981 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@7a450ea8
2015-12-03 16:40:11,988 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
[zk: localhost:2181(CONNECTING) 18] 2015-12-03 16:40:12,014 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session
2015-12-03 16:40:12,028 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x151665fb7430004, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: localhost:2181(CONNECTED) 18] ls /
[test, zookeeper]
[zk: localhost:2181(CONNECTED) 19] 
~~~


---

### quit

退出

~~~
[zk: localhost:2181(CONNECTED) 80] quit
Quitting...
2015-12-03 15:48:27,285 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x151665fb7430001 closed
2015-12-03 15:48:27,294 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down
[root@h101 bin]# 
~~~


---

### redo

再次执行之前命令

> **Tip:** 历史命令的保持是在当前会话中，不论此时是否已经成功连接服务

~~~
[zk: localhost:2181(CONNECTED) 20] history
10 - ls
11 - ls /
12 - create /test test
13 - get /test
14 - ls /
15 - close 
16 - ls /
17 - connect localhost:2181
18 - ls /
19 - help 
20 - history
[zk: localhost:2181(CONNECTED) 21] redo 11
[test, zookeeper]
[zk: localhost:2181(CONNECTED) 22] redo 15
2015-12-03 16:43:43,109 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x151665fb7430004 closed
Not connected
[zk: localhost:2181(CLOSED) 23] 2015-12-03 16:43:43,111 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down

[zk: localhost:2181(CLOSED) 23] 
[zk: localhost:2181(CLOSED) 23] redo 17
2015-12-03 16:43:51,288 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@5a4a6010
[zk: localhost:2181(CONNECTING) 24] 2015-12-03 16:43:51,309 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2015-12-03 16:43:51,311 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2015-12-03 16:43:51,333 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x151665fb7430005, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: localhost:2181(CONNECTED) 24] 
~~~

还有一些关于权限管控和监听的命令在以后深入使用过程中再仔细研究


---


#  命令汇总

* **`wget  http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz`**
* **`tar -zxvf zookeeper-3.4.6.tar.gz`**
* **`cd zookeeper-3.4.6`**
* **`grep -v "^#" conf/zoo_sample.cfg`**
* **`cp conf/zoo_sample.cfg conf/zoo.cfg`**
* **`bin/zkServer.sh  start`**
* **`netstat  -ant | grep 2181`**
* **`bin/zkServer.sh restart`**
* **`bin/zkServer.sh stop`**
* **`bin/zkServer.sh status`**
* **`bin/zkCli.sh`**



---

[zookeeper]:http://zookeeper.apache.org/
[zookeeper_download]:http://zookeeper.apache.org/releases.html#download
[zookeeper_mirror]:http://www.apache.org/dyn/closer.cgi/zookeeper/


