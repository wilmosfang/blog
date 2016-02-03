---
layout: post
title: ZooKeeper 集群
categories: linux zookeeper cluster
wc: 701 2633 32235
excerpt: follow me
comments: true
---


---

# 前言


**[ZooKeeper][zookeeper]** 本身就是为分布式应用服务的，为了确保高可用所以很少使用 **Standalone** 模式，而更多是使用集群模式运行


一般而言使用3个或大于3个的奇数个server

>For replicated mode, a minimum of three servers are required, and it is strongly recommended that you have an odd number of servers . If you only have two servers, then you are in a situation where if one of them fails, there are not enough machines to form a majority quorum. Two servers is inherently less stable than a single server, because there are two single points of failure.


下面分享一下它的集群操作，详细可以参考 [官方文档][zookeeper]

> **Tip:** 当前版本为 **Release 3.4.6(stable)**


---


# 概要

* TOC
{:toc}



---

##伪集群模式

所谓 **伪集群** 其实就是在同一台机器上运行多个server，从而构成集群，这类集群可以展示集群的逻辑特性

但是由于其固有的架构缺乏实际的物理冗余，所以并不抗风险，不是真正意义上的高可用集群

---

###拷贝目录

停掉应用后将 **zookeeper-3.4.6** 目录拷贝两份

{% highlight bash %}
[root@h101 zk]# ll  -d zookeeper-3.4.6*
drwxr-xr-x 10 1000 1000     4096 Dec  2 21:58 zookeeper-3.4.6
drwxr-xr-x 10 root root     4096 Dec  3 19:24 zookeeper-3.4.6.1
drwxr-xr-x 10 root root     4096 Dec  3 19:24 zookeeper-3.4.6.2
-rw-r--r--  1 root root 17699306 Oct 31  2014 zookeeper-3.4.6.tar.gz
[root@h101 zk]# 
{% endhighlight %}

---

###修改配置

{% highlight bash %}
[root@h101 zk]# cat zookeeper-3.4.6/conf/zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper0
dataLogDir=/tmp/zookeeper0
clientPort=2180
server.0=127.0.0.1:8000:8100
server.1=127.0.0.1:8001:8101
server.2=127.0.0.1:8002:8102
[root@h101 zk]# ll zookeeper-3.4.6*/conf/zoo.cfg
-rw-r--r-- 1 root root 193 Dec  3 19:24 zookeeper-3.4.6.1/conf/zoo.cfg
-rw-r--r-- 1 root root 193 Dec  3 19:25 zookeeper-3.4.6.2/conf/zoo.cfg
-rw-r--r-- 1 root root 193 Dec  3 19:23 zookeeper-3.4.6/conf/zoo.cfg
[root@h101 zk]# cat zookeeper-3.4.6*/conf/zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper1
dataLogDir=/tmp/zookeeper1
clientPort=2181
server.0=127.0.0.1:8000:8100
server.1=127.0.0.1:8001:8101
server.2=127.0.0.1:8002:8102
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper2
dataLogDir=/tmp/zookeeper2
clientPort=2182
server.0=127.0.0.1:8000:8100
server.1=127.0.0.1:8001:8101
server.2=127.0.0.1:8002:8102
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper0
dataLogDir=/tmp/zookeeper0
clientPort=2180
server.0=127.0.0.1:8000:8100
server.1=127.0.0.1:8001:8101
server.2=127.0.0.1:8002:8102
[root@h101 zk]# 
{% endhighlight %}

Item     | Comment
-------- | ---
tickTime | zk的单位时长，单位ms
initLimit| 初始化连接时，follower和leader之间的最长心跳时间，tickTime的倍数
syncLimit| leader和follower之间发送消息, 请求和应答的最大时间长度，tickTime的倍数
dataDir|数据存放目录
dataLogDir|日志存放目录
clientPort|服务监听端口
server.X=A:B:C|X代表serverid，要求dataDir/myid(需要另外手动创建)里包含相同数字;A代表server所在的IP地址;B代表server之间交换信息使用的端口; C代表server之间选举leader时所使用的端口


###建数据目录

{% highlight bash %}
[root@h101 zk]# ll -d /tmp/zookeeper*
drwxr-xr-x 3 root root 4096 Dec  3 19:33 /tmp/zookeeper0
drwxr-xr-x 3 root root 4096 Dec  3 19:33 /tmp/zookeeper1
drwxr-xr-x 3 root root 4096 Dec  3 19:33 /tmp/zookeeper2
[root@h101 zk]# 
{% endhighlight %}

---

###创建 **myid** 文件

{% highlight bash %}
[root@h101 zk]# ll /tmp/zookeeper*/myid
-rw-r--r-- 1 root root 2 Dec  3 19:22 /tmp/zookeeper0/myid
-rw-r--r-- 1 root root 2 Dec  3 19:22 /tmp/zookeeper1/myid
-rw-r--r-- 1 root root 2 Dec  3 19:23 /tmp/zookeeper2/myid
[root@h101 zk]# cat /tmp/zookeeper*/myid
0
1
2
[root@h101 zk]# 
{% endhighlight %}

---

###启动集群

分别启动服务

{% highlight bash %}
[root@h101 zk]#  zookeeper-3.4.6/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zk]#  zookeeper-3.4.6.1/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zk]#  zookeeper-3.4.6.2/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zk]# 
{% endhighlight %}

查看状态


{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6.2/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]# zookeeper-3.4.6.1/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Mode: leader
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]#
{% endhighlight %}

可知此时每个服务分别的角色



查看进程

{% highlight bash %}
[root@h101 zk]# ps faux | grep zookeeper | grep -v grep  
root      5474  0.2  2.8 2103548 55168 pts/0   Sl   19:33   0:06 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zk/zookeeper-3.4.6/bin/../build/classes:/root/zk/zookeeper-3.4.6/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
root      5505  0.4  3.1 2109716 61216 pts/0   Sl   19:33   0:12 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zk/zookeeper-3.4.6.1/bin/../build/classes:/root/zk/zookeeper-3.4.6.1/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6.1/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6.1/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6.1/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6.1/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6.1/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6.1/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6.1/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6.1/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
root      5549  0.3  2.8 2103548 53900 pts/0   Sl   19:33   0:10 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zk/zookeeper-3.4.6.2/bin/../build/classes:/root/zk/zookeeper-3.4.6.2/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6.2/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6.2/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6.2/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6.2/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6.2/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6.2/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6.2/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6.2/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
[root@h101 zk]# 
{% endhighlight %}

{% highlight bash %}
[root@h101 zk]# ll /tmp/zookeeper*/zookeeper_server.pid 
-rw-r--r-- 1 root root 4 Dec  3 19:33 /tmp/zookeeper0/zookeeper_server.pid
-rw-r--r-- 1 root root 4 Dec  3 19:33 /tmp/zookeeper1/zookeeper_server.pid
-rw-r--r-- 1 root root 4 Dec  3 19:33 /tmp/zookeeper2/zookeeper_server.pid
[root@h101 zk]# cat  /tmp/zookeeper*/zookeeper_server.pid 
547455055549[root@h101 zk]#
{% endhighlight %}

---

###连接服务

{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6/bin/zkCli.sh  -server localhost:2180
Connecting to localhost:2180
2015-12-03 20:21:25,745 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2015-12-03 20:21:25,752 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=h101.temp
2015-12-03 20:21:25,752 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_65
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.65.x86_64/jre
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/root/zk/zookeeper-3.4.6/bin/../build/classes:/root/zk/zookeeper-3.4.6/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../conf:
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2015-12-03 20:21:25,757 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2015-12-03 20:21:25,758 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2015-12-03 20:21:25,758 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-504.el6.x86_64
2015-12-03 20:21:25,758 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2015-12-03 20:21:25,758 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2015-12-03 20:21:25,758 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/root/zk
2015-12-03 20:21:25,761 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2180 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@3243a52c
Welcome to ZooKeeper!
2015-12-03 20:21:25,821 [myid:] - INFO  [main-SendThread(localhost:2180):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/127.0.0.1:2180. Will not attempt to authenticate using SASL (unknown error)
2015-12-03 20:21:25,837 [myid:] - INFO  [main-SendThread(localhost:2180):ClientCnxn$SendThread@852] - Socket connection established to localhost/127.0.0.1:2180, initiating session
JLine support is enabled
2015-12-03 20:21:25,861 [myid:] - INFO  [main-SendThread(localhost:2180):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/127.0.0.1:2180, sessionid = 0x51679e16b90001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2180(CONNECTED) 0] 
[zk: localhost:2180(CONNECTED) 0] 
[zk: localhost:2180(CONNECTED) 0] 
[zk: localhost:2180(CONNECTED) 0] 
[zk: localhost:2180(CONNECTED) 0] ls /
[abc, zookeeper]
[zk: localhost:2180(CONNECTED) 1] create /defg defg
Created /defg
[zk: localhost:2180(CONNECTED) 2] get /defg
defg
cZxid = 0x100000009
ctime = Thu Dec 03 20:21:43 CST 2015
mZxid = 0x100000009
mtime = Thu Dec 03 20:21:43 CST 2015
pZxid = 0x100000009
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: localhost:2180(CONNECTED) 3] connect localhost:2181
2015-12-03 20:24:18,536 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x51679e16b90001 closed
2015-12-03 20:24:18,537 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@d5c4abf
2015-12-03 20:24:18,538 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down
[zk: localhost:2181(CONNECTING) 4] 2015-12-03 20:24:18,559 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2015-12-03 20:24:18,561 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2015-12-03 20:24:18,575 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x151679e16cc0001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: localhost:2181(CONNECTED) 4] ls /
[abc, defg, zookeeper]
[zk: localhost:2181(CONNECTED) 5] get /defg
defg
cZxid = 0x100000009
ctime = Thu Dec 03 20:21:43 CST 2015
mZxid = 0x100000009
mtime = Thu Dec 03 20:21:43 CST 2015
pZxid = 0x100000009
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: localhost:2181(CONNECTED) 6] connect localhost:2182
2015-12-03 20:24:38,749 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down
2015-12-03 20:24:38,750 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x151679e16cc0001 closed
2015-12-03 20:24:38,751 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2182 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@2e3fe12e
2015-12-03 20:24:38,756 [myid:] - INFO  [main-SendThread(localhost:2182):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2182. Will not attempt to authenticate using SASL (unknown error)
2015-12-03 20:24:38,757 [myid:] - INFO  [main-SendThread(localhost:2182):ClientCnxn$SendThread@852] - Socket connection established to localhost/0:0:0:0:0:0:0:1:2182, initiating session
[zk: localhost:2182(CONNECTING) 7] 2015-12-03 20:24:38,803 [myid:] - INFO  [main-SendThread(localhost:2182):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2182, sessionid = 0x251679e27ea0001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: localhost:2182(CONNECTED) 7] ls /
[abc, defg, zookeeper]
[zk: localhost:2182(CONNECTED) 8] get /defg
defg
cZxid = 0x100000009
ctime = Thu Dec 03 20:21:43 CST 2015
mZxid = 0x100000009
mtime = Thu Dec 03 20:21:43 CST 2015
pZxid = 0x100000009
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: localhost:2182(CONNECTED) 9] 
{% endhighlight %}


###依次关掉服务

当前状态

{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]# zookeeper-3.4.6.1/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Mode: leader
[root@h101 zk]# zookeeper-3.4.6.2/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]#
{% endhighlight %}

关掉当前leader

{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6.1/bin/zkServer.sh  stop
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
[root@h101 zk]# zookeeper-3.4.6.2/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
Mode: leader
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]#
{% endhighlight %}

关掉新选出的leader(关掉一大半server)

{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6.2/bin/zkServer.sh  stop
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@h101 zk]# zookeeper-3.4.6.1/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@h101 zk]# zookeeper-3.4.6.2/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.2/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@h101 zk]# ps faux | grep zookeeper | grep -v grep 
root      6117  0.9  2.4 2103548 46112 pts/0   Sl   20:27   0:01 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zk/zookeeper-3.4.6/bin/../build/classes:/root/zk/zookeeper-3.4.6/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@h101 zk]# 
{% endhighlight %}

发现剩下一台，但服务已经不可用了


随便启动一台

{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh  status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@h101 zk]# zookeeper-3.4.6.1/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zk]# zookeeper-3.4.6.1/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6.1/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]# zookeeper-3.4.6/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: leader
[root@h101 zk]# 
{% endhighlight %}

剩下的那个成为了leader，新启动的成为了follower，服务变得可用




---

##集群模式


集群模式在配置上与之前的没有本质区别，唯一区别就是server分布在了不同的物理服务器上

###修改配置

{% highlight bash %}
[root@h101 zk]# cat zookeeper-3.4.6-real/conf/zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper101
dataLogDir=/tmp/zookeeper101
clientPort=2180
server.101=192.168.100.101:8000:8100
server.102=192.168.100.102:8000:8100
server.202=192.168.100.202:8000:8100
[root@h101 zk]# 
{% endhighlight %}

---

###拷贝目录

{% highlight bash %}
[root@h101 zk]# rsync  -av zookeeper-3.4.6-real root@192.168.100.102:/root/zk/zookeeper-3.4.6-real/
root@192.168.100.102's password: 
sending incremental file list
created directory /root/zk/zookeeper-3.4.6-real
zookeeper-3.4.6-real/
zookeeper-3.4.6-real/CHANGES.txt
zookeeper-3.4.6-real/LICENSE.txt
zookeeper-3.4.6-real/NOTICE.txt
...
...
zookeeper-3.4.6-real/src/recipes/queue/test/org/apache/zookeeper/recipes/queue/
zookeeper-3.4.6-real/src/recipes/queue/test/org/apache/zookeeper/recipes/queue/DistributedQueueTest.java

sent 38977410 bytes  received 29989 bytes  11144971.14 bytes/sec
total size is 38865680  speedup is 1.00
[root@h101 zk]#
[root@h101 zk]# rsync  -av zookeeper-3.4.6-real root@192.168.100.202:/root/zk/zookeeper-3.4.6-real/
The authenticity of host '192.168.100.202 (192.168.100.202)' can't be established.
RSA key fingerprint is 78:c4:6f:3f:08:43:d1:2a:02:bf:ec:f3:9f:e3:89:76.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.100.202' (RSA) to the list of known hosts.
Address 192.168.100.202 maps to localhost, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
root@192.168.100.202's password: 
sending incremental file list
created directory /root/zk/zookeeper-3.4.6-real
zookeeper-3.4.6-real/
zookeeper-3.4.6-real/CHANGES.txt
zookeeper-3.4.6-real/LICENSE.txt
zookeeper-3.4.6-real/NOTICE.txt
...
...
zookeeper-3.4.6-real/src/recipes/queue/test/org/apache/zookeeper/recipes/queue/
zookeeper-3.4.6-real/src/recipes/queue/test/org/apache/zookeeper/recipes/queue/DistributedQueueTest.java

sent 38977410 bytes  received 29989 bytes  6001138.31 bytes/sec
total size is 38865680  speedup is 1.00
[root@h101 zk]#
{% endhighlight %}

> **Note:** 拷贝后，注意修改 **dataDir** 成正确的路径，也可以不修改，那在其它服务器上创建的绝对路径就要相同


---

###创建dataDir和myid

{% highlight bash %}
[root@h101 zk]# mkdir  /tmp/zookeeper101
[root@h101 zk]# echo 101 > /tmp/zookeeper101/myid
[root@h101 zk]# cat /tmp/zookeeper101/myid
101
[root@h101 zk]# 
----------
[root@h102 ~]# mkdir  /tmp/zookeeper102
[root@h102 ~]# echo 102 > /tmp/zookeeper102/myid
----------
[root@redis-b ~]# mkdir  /tmp/zookeeper202
[root@redis-b ~]# echo 202 > /tmp/zookeeper202/myid
{% endhighlight %}


---

###开启防火墙


在每台服务器的 **/etc/sysconfig/iptables** 中加入以下几行

{% highlight bash %}
-A INPUT -p tcp -m state --state NEW -m tcp --dport 2180  -j ACCEPT 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8000  -j ACCEPT 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8100  -j ACCEPT 
{% endhighlight %}

然后重载一下

{% highlight bash %}
[root@h101 zk]# vim /etc/sysconfig/iptables
[root@h101 zk]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h101 zk]#
{% endhighlight %}


可以使用 **`iptables -L -nv`** 进行检查，filter 表中包含以下几行的，为已经生效

{% highlight bash %}
0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:2180 
0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8000 
0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8100
{% endhighlight %}

---

###启动服务

{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6-real/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6-real/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h101 zk]# 
----------
[root@h102 zookeeper-3.4.6-real]# zookeeper-3.4.6-real/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h102 zookeeper-3.4.6-real]# 
----------
[root@redis-b zookeeper-3.4.6-real]# zookeeper-3.4.6-real/bin/zkServer.sh start 
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@redis-b zookeeper-3.4.6-real]#
{% endhighlight %}

服务状态


{% highlight bash %}
[root@h101 zk]# zookeeper-3.4.6-real/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6-real/bin/../conf/zoo.cfg
Mode: follower
[root@h101 zk]#
----------
[root@h102 zookeeper-3.4.6-real]# zookeeper-3.4.6-real/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../conf/zoo.cfg
Mode: follower
[root@h102 zookeeper-3.4.6-real]# 
----------
[root@redis-b zookeeper-3.4.6-real]# zookeeper-3.4.6-real/bin/zkServer.sh status
JMX enabled by default
Using config: /root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../conf/zoo.cfg
Mode: leader
[root@redis-b zookeeper-3.4.6-real]#
{% endhighlight %}



---

###连接测试 


{% highlight bash %}
[root@redis-b zookeeper-3.4.6-real]# zookeeper-3.4.6-real/bin/zkCli.sh -server 192.168.100.101:2180
Connecting to 192.168.100.101:2180
2015-12-03 21:17:07,041 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2015-12-03 21:17:07,046 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=220.250.64.225
2015-12-03 21:17:07,046 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_65
2015-12-03 21:17:07,049 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2015-12-03 21:17:07,050 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.65.x86_64/jre
2015-12-03 21:17:07,050 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../build/classes:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../build/lib/*.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../lib/slf4j-log4j12-1.6.1.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../lib/slf4j-api-1.6.1.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../lib/netty-3.7.0.Final.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../lib/log4j-1.2.16.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../lib/jline-0.9.94.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../zookeeper-3.4.6.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../src/java/lib/*.jar:/root/zk/zookeeper-3.4.6-real/zookeeper-3.4.6-real/bin/../conf:
2015-12-03 21:17:07,050 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2015-12-03 21:17:07,050 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2015-12-03 21:17:07,050 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2015-12-03 21:17:07,050 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2015-12-03 21:17:07,051 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2015-12-03 21:17:07,051 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-504.el6.x86_64
2015-12-03 21:17:07,051 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2015-12-03 21:17:07,051 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2015-12-03 21:17:07,051 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/root/zk/zookeeper-3.4.6-real
2015-12-03 21:17:07,054 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=192.168.100.101:2180 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@3243a52c
Welcome to ZooKeeper!
2015-12-03 21:17:07,222 [myid:] - INFO  [main-SendThread(192.168.100.101:2180):ClientCnxn$SendThread@975] - Opening socket connection to server 192.168.100.101/192.168.100.101:2180. Will not attempt to authenticate using SASL (unknown error)
2015-12-03 21:17:07,232 [myid:] - INFO  [main-SendThread(192.168.100.101:2180):ClientCnxn$SendThread@852] - Socket connection established to 192.168.100.101/192.168.100.101:2180, initiating session
JLine support is enabled
2015-12-03 21:17:07,370 [myid:] - INFO  [main-SendThread(192.168.100.101:2180):ClientCnxn$SendThread@1235] - Session establishment complete on server 192.168.100.101/192.168.100.101:2180, sessionid = 0x655167f10adf0000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: 192.168.100.101:2180(CONNECTED) 0] 
[zk: 192.168.100.101:2180(CONNECTED) 0] 
[zk: 192.168.100.101:2180(CONNECTED) 0] ls /
[zookeeper]
[zk: 192.168.100.101:2180(CONNECTED) 1] create /ui ui
Created /ui
[zk: 192.168.100.101:2180(CONNECTED) 3] get /ui
ui
cZxid = 0x100000002
ctime = Thu Dec 03 21:17:30 CST 2015
mZxid = 0x100000002
mtime = Thu Dec 03 21:17:30 CST 2015
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 2
numChildren = 0
[zk: 192.168.100.101:2180(CONNECTED) 4] connect 192.168.100.102:2180
2015-12-03 21:17:51,662 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x655167f10adf0000 closed
2015-12-03 21:17:51,662 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=192.168.100.102:2180 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@339bf2ac
2015-12-03 21:17:51,662 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down
2015-12-03 21:17:51,668 [myid:] - INFO  [main-SendThread(192.168.100.102:2180):ClientCnxn$SendThread@975] - Opening socket connection to server 192.168.100.102/192.168.100.102:2180. Will not attempt to authenticate using SASL (unknown error)
[zk: 192.168.100.102:2180(CONNECTING) 5] 2015-12-03 21:17:51,693 [myid:] - INFO  [main-SendThread(192.168.100.102:2180):ClientCnxn$SendThread@852] - Socket connection established to 192.168.100.102/192.168.100.102:2180, initiating session
2015-12-03 21:17:51,714 [myid:] - INFO  [main-SendThread(192.168.100.102:2180):ClientCnxn$SendThread@1235] - Session establishment complete on server 192.168.100.102/192.168.100.102:2180, sessionid = 0x665167f1562b0000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: 192.168.100.102:2180(CONNECTED) 5] ls /
[ui, zookeeper]
[zk: 192.168.100.102:2180(CONNECTED) 6] get /ui
ui
cZxid = 0x100000002
ctime = Thu Dec 03 21:17:30 CST 2015
mZxid = 0x100000002
mtime = Thu Dec 03 21:17:30 CST 2015
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 2
numChildren = 0
[zk: 192.168.100.102:2180(CONNECTED) 7] connect 192.168.100.202:2180
2015-12-03 21:18:07,300 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x665167f1562b0000 closed
2015-12-03 21:18:07,300 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@512] - EventThread shut down
2015-12-03 21:18:07,300 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=192.168.100.202:2180 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@76fb7505
[zk: 192.168.100.202:2180(CONNECTING) 8] 2015-12-03 21:18:07,327 [myid:] - INFO  [main-SendThread(192.168.100.202:2180):ClientCnxn$SendThread@975] - Opening socket connection to server 192.168.100.202/192.168.100.202:2180. Will not attempt to authenticate using SASL (unknown error)
2015-12-03 21:18:07,328 [myid:] - INFO  [main-SendThread(192.168.100.202:2180):ClientCnxn$SendThread@852] - Socket connection established to 192.168.100.202/192.168.100.202:2180, initiating session
2015-12-03 21:18:07,379 [myid:] - INFO  [main-SendThread(192.168.100.202:2180):ClientCnxn$SendThread@1235] - Session establishment complete on server 192.168.100.202/192.168.100.202:2180, sessionid = 0xca5167f10f260000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: 192.168.100.202:2180(CONNECTED) 8] ls /   
[ui, zookeeper]
[zk: 192.168.100.202:2180(CONNECTED) 9] get /ui
ui
cZxid = 0x100000002
ctime = Thu Dec 03 21:17:30 CST 2015
mZxid = 0x100000002
mtime = Thu Dec 03 21:17:30 CST 2015
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 2
numChildren = 0
[zk: 192.168.100.202:2180(CONNECTED) 10] 
{% endhighlight %}


---

[zookeeper]:http://zookeeper.apache.org/

