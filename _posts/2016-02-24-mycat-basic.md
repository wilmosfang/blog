---
layout: post
title:  Mycat 基础
categories:  linux mysql mycat
wc: 604  1967 23792 
excerpt: mycat运行的环境依赖，mycat下载，安装，配置，连接，启动与停止等相关基础
comments: true
---



# 前言


网络应用持续扩张的过程中，为了处理海量数据往往首先遇到的挑战就是数据存储的扩展

数据存储的扩展一般以切分来实现，切分的技术实现又可分为垂直切分和水平切分：

* 以表(或Schema)为切分粒度的是垂直切分
* 以表内行为切分粒度的是水平切分

初期扩展使用垂直切分就可以基本解决问题，垂直切分也相对简单，但随着数据行成量级的持续增长，针对这张表的各层面操作性能都会显著降低，此时就不得不进行水平切分了，水平切分就要复杂很多

为了应对此类挑战，就产生了一类新的数据库 **NoSQL** (Not Only SQL) ，此类的代表有 **MongoDB、Redis、Memcached、Elasticsearch**  ，这类数据库可以轻松应对数据库的扩展(不论是水平还是垂直切分都非常高效)，在海量数据的情况下，也能有着极佳的读写性能，NoSQL的根本优势就在于大数据时代易于进行大规模分布式扩展

但是由于NoSQL固有的分布式架构，目前对事务的支持非常弱，存储也是弱结构的，join等复杂操作能力很差，应用场景比较局限

数据库弱了，就意味着，如果要实现一样的特性，应用层面的设计得更复杂才能弥补，这也无形地引入了架构风险

为了使用到关系模型的一些特性(交易或支付的场景，前几年非常火热的去IOE)，还是绕不过关系型数据库，但是关系型数据库先天就对分布式支持很弱

简单点可以这么理解：

* **事务性就是序列化，高并发就是分布式，序列化与并行相背离，所以事务性和分布式很难和谐共处** 

在事务性和高并发中得有取舍，所以市面上又出现了很多用来进行协调的中间件

**[Mycat][mycat]** 是一个数据库分库分表中间件，就是用来协调这类问题的

> **Tip:** **[Mycat][mycat]** 是一款国人贡献的非常优秀的开源软件，近两年来，国人贡献出越来越多优秀的开源项目，对于IT技术界真是一种可喜局面

与 **[Mycat][mycat]** 类似的还有 **TDDL、Amoeba、Cobar**，它们的架构如下：

* **TDDL**

![tddl.png](/images/mycat/tddl.png)

* **Amoeba**

![amoeba.png](/images/mycat/amoeba.png)


* **Cobar**

![cobar.png](/images/mycat/cobar.png)

* **Mycat**

![mycat.png](/images/mycat/mycat.png)


**TDDL** 相对异类，而 **Amoeba、Cobar、MyCAT** 却是一脉相承，每个项目都是脱胎于上一个项目

相对于 **TDDL、Amoeba、Cobar** 这几个几乎停滞的项目，目前 **MyCAT** 最为活跃

这里简单分享一下 **[Mycat][mycat]** 的相关基础 ，详细内容可以参考 **[官方文档][mycat_doc]** 和 **[Mycat-Server][mycat_git]**


> **Tip:** 当前的最新版本为 **Mycat server 1.5 GA** 

---


# 概要

* TOC
{:toc}



---

## 环境

MyCAT 是使用JAVA开发的，必须运行于JRE的环境中，MyCAT的运行依赖不低于JDK7版本的环境


{% highlight bash %}
[root@h102 ~]# java -version
java version "1.7.0_65"
OpenJDK Runtime Environment (rhel-2.5.1.2.el6_5-x86_64 u65-b17)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
[root@h102 ~]# 
{% endhighlight %}

环境符合要求

---

## 下载

MyCAT的 **[下载地址][mycat_dl]**

{% highlight bash %}
[root@h102 mycat]# wget https://github.com/MyCATApache/Mycat-download/raw/master/1.5-GA/Mycat-server-1.5-GA-20160217103036-linux.tar.gz
--2016-02-24 17:00:41--  https://github.com/MyCATApache/Mycat-download/raw/master/1.5-GA/Mycat-server-1.5-GA-20160217103036-linux.tar.gz
Resolving github.com... 192.30.252.129
Connecting to github.com|192.30.252.129|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/MyCATApache/Mycat-download/master/1.5-GA/Mycat-server-1.5-GA-20160217103036-linux.tar.gz [following]
--2016-02-24 17:00:45--  https://raw.githubusercontent.com/MyCATApache/Mycat-download/master/1.5-GA/Mycat-server-1.5-GA-20160217103036-linux.tar.gz
Resolving raw.githubusercontent.com... 103.245.222.133
Connecting to raw.githubusercontent.com|103.245.222.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11477321 (11M) [application/octet-stream]
Saving to: “Mycat-server-1.5-GA-20160217103036-linux.tar.gz”

100%[============================================================================================>] 11,477,321   196K/s   in 87s     

2016-02-24 17:02:26 (129 KB/s) - “Mycat-server-1.5-GA-20160217103036-linux.tar.gz” saved [11477321/11477321]

[root@h102 mycat]#
{% endhighlight %}

---

## 安装


解压就可以直接使用

{% highlight bash %}
[root@h102 mycat]# ls
Mycat-server-1.5-GA-20160217103036-linux.tar.gz
[root@h102 mycat]# tar -xzvf Mycat-server-1.5-GA-20160217103036-linux.tar.gz 
mycat/bin/wrapper-linux-ppc-64
mycat/bin/wrapper-linux-x86-64
mycat/bin/wrapper-linux-x86-32
mycat/bin/mycat
mycat/lib/zookeeper-3.4.6.jar
mycat/lib/jline-0.9.94.jar
mycat/lib/ehcache-core-2.6.11.jar
mycat/lib/log4j-1.2.17.jar
mycat/lib/guava-18.0.jar
mycat/lib/libwrapper-linux-x86-32.so
mycat/lib/netty-3.7.0.Final.jar
mycat/lib/mapdb-1.0.7.jar
mycat/lib/xml-apis-1.0.b2.jar
mycat/lib/slf4j-api-1.7.12.jar
mycat/lib/leveldb-api-0.7.jar
mycat/lib/wrapper.jar
mycat/lib/slf4j-log4j12-1.7.12.jar
mycat/lib/json-20151123.jar
mycat/lib/curator-framework-2.9.0.jar
mycat/lib/mongo-java-driver-2.11.4.jar
mycat/lib/druid-1.0.14.jar
mycat/lib/libwrapper-linux-ppc-64.so
mycat/lib/leveldb-0.7.jar
mycat/lib/fastjson-1.2.7.jar
mycat/lib/sequoiadb-java-driver-1.0-20150615.070208-1.jar
mycat/lib/Mycat-server-1.5-GA.jar
mycat/lib/snakeyaml-1.16.jar
mycat/lib/univocity-parsers-1.5.4.jar
mycat/lib/curator-client-2.9.0.jar
mycat/lib/libwrapper-linux-x86-64.so
mycat/lib/dom4j-1.6.1.jar
mycat/conf/wrapper.conf
mycat/conf/
mycat/conf/sequence_time_conf.properties
mycat/conf/ehcache.xml
mycat/conf/index_to_charset.properties
mycat/conf/partition-range-mod.txt
mycat/conf/sequence_db_conf.properties
mycat/conf/cacheservice.properties
mycat/conf/partition-hash-int.txt
mycat/conf/autopartition-long.txt
mycat/conf/rule.xml
mycat/conf/router.xml
mycat/conf/sequence_conf.properties
mycat/conf/myid.properties
mycat/conf/schema.xml
mycat/conf/zk-create.yaml
mycat/conf/server.xml
mycat/version.txt
mycat/conf/log4j.xml
mycat/bin/init_zk_data.sh
mycat/bin/startup_nowrap.sh
mycat/bin/rehash.sh
mycat/bin/xml_to_yaml.sh
mycat/logs/
mycat/catlet/
[root@h102 mycat]# ll 
total 11216
drwxr-xr-x 7 root root     4096 Feb 24 17:12 mycat
-rw-r--r-- 1 root root 11477321 Feb 24 17:02 Mycat-server-1.5-GA-20160217103036-linux.tar.gz
[root@h102 mycat]# ll mycat/
total 24
drwxr-xr-x 2 root root 4096 Feb 24 17:12 bin
drwxrwxrwx 2 root root 4096 Dec 13 16:55 catlet
drwxrwxrwx 2 root root 4096 Feb 24 17:12 conf
drwxr-xr-x 2 root root 4096 Feb 24 17:12 lib
drwxrwxrwx 2 root root 4096 Dec 13 16:55 logs
-rwxrwxrwx 1 root root  212 Feb 17 10:30 version.txt
[root@h102 mycat]# 
[root@h102 mycat]# tree mycat/
mycat/
├── bin
│   ├── init_zk_data.sh
│   ├── mycat
│   ├── rehash.sh
│   ├── startup_nowrap.sh
│   ├── wrapper-linux-ppc-64
│   ├── wrapper-linux-x86-32
│   ├── wrapper-linux-x86-64
│   └── xml_to_yaml.sh
├── catlet
├── conf
│   ├── autopartition-long.txt
│   ├── cacheservice.properties
│   ├── ehcache.xml
│   ├── index_to_charset.properties
│   ├── log4j.xml
│   ├── myid.properties
│   ├── partition-hash-int.txt
│   ├── partition-range-mod.txt
│   ├── router.xml
│   ├── rule.xml
│   ├── schema.xml
│   ├── sequence_conf.properties
│   ├── sequence_db_conf.properties
│   ├── sequence_time_conf.properties
│   ├── server.xml
│   ├── wrapper.conf
│   └── zk-create.yaml
├── lib
│   ├── curator-client-2.9.0.jar
│   ├── curator-framework-2.9.0.jar
│   ├── dom4j-1.6.1.jar
│   ├── druid-1.0.14.jar
│   ├── ehcache-core-2.6.11.jar
│   ├── fastjson-1.2.7.jar
│   ├── guava-18.0.jar
│   ├── jline-0.9.94.jar
│   ├── json-20151123.jar
│   ├── leveldb-0.7.jar
│   ├── leveldb-api-0.7.jar
│   ├── libwrapper-linux-ppc-64.so
│   ├── libwrapper-linux-x86-32.so
│   ├── libwrapper-linux-x86-64.so
│   ├── log4j-1.2.17.jar
│   ├── mapdb-1.0.7.jar
│   ├── mongo-java-driver-2.11.4.jar
│   ├── Mycat-server-1.5-GA.jar
│   ├── netty-3.7.0.Final.jar
│   ├── sequoiadb-java-driver-1.0-20150615.070208-1.jar
│   ├── slf4j-api-1.7.12.jar
│   ├── slf4j-log4j12-1.7.12.jar
│   ├── snakeyaml-1.16.jar
│   ├── univocity-parsers-1.5.4.jar
│   ├── wrapper.jar
│   ├── xml-apis-1.0.b2.jar
│   └── zookeeper-3.4.6.jar
├── logs
└── version.txt

5 directories, 53 files
[root@h102 mycat]# 
{% endhighlight %}


---


## 修改配置


主要调整内存使用大小，因为是测试环境，尽量调小一点，生产环境得根据具体情况评估

{% highlight bash %}
[root@h102 mycat]# ll conf/wrapper.conf 
-rwxrwxrwx 1 root root 4244 Feb 24 20:58 conf/wrapper.conf
[root@h102 mycat]# vim conf/wrapper.conf 
[root@h102 mycat]# grep Xm conf/wrapper.conf 
#wrapper.java.additional.10=-Xmx4G
wrapper.java.additional.10=-Xmx512m
#wrapper.java.additional.11=-Xms1G
wrapper.java.additional.11=-Xms128m
[root@h102 mycat]# grep MaxDirectMemorySize conf/wrapper.conf 
#wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G
wrapper.java.additional.5=-XX:MaxDirectMemorySize=256m
[root@h102 mycat]#
[root@h102 mycat]# grep -v "^#" conf/wrapper.conf | grep -v "^$"
wrapper.java.command=java
wrapper.working.dir=..
wrapper.java.mainclass=org.tanukisoftware.wrapper.WrapperSimpleApp
set.default.REPO_DIR=lib
set.APP_BASE=.
wrapper.java.classpath.1=lib/wrapper.jar
wrapper.java.classpath.2=conf
wrapper.java.classpath.3=%REPO_DIR%/*
wrapper.java.library.path.1=lib
wrapper.java.additional.1=-DMYCAT_HOME=.
wrapper.java.additional.2=-server
wrapper.java.additional.3=-XX:MaxPermSize=64M
wrapper.java.additional.4=-XX:+AggressiveOpts
wrapper.java.additional.5=-XX:MaxDirectMemorySize=256m
wrapper.java.additional.6=-Dcom.sun.management.jmxremote
wrapper.java.additional.7=-Dcom.sun.management.jmxremote.port=1984
wrapper.java.additional.8=-Dcom.sun.management.jmxremote.authenticate=false
wrapper.java.additional.9=-Dcom.sun.management.jmxremote.ssl=false
wrapper.java.additional.10=-Xmx512m
wrapper.java.additional.11=-Xms128m
wrapper.app.parameter.1=org.opencloudb.MycatStartup
wrapper.app.parameter.2=start
wrapper.console.format=PM
wrapper.console.loglevel=INFO
wrapper.logfile=logs/wrapper.log
wrapper.logfile.format=LPTM
wrapper.logfile.loglevel=INFO
wrapper.logfile.maxsize=0
wrapper.logfile.maxfiles=0
wrapper.syslog.loglevel=NONE
wrapper.console.title=Mycat-server
wrapper.ntservice.name=mycat
wrapper.ntservice.displayname=Mycat-server
wrapper.ntservice.description=The project of Mycat-server
wrapper.ntservice.dependency.1=
wrapper.ntservice.starttype=AUTO_START
wrapper.ntservice.interactive=false
wrapper.ping.timeout=120
configuration.directory.in.classpath.first=conf
[root@h102 mycat]# 
{% endhighlight %}


下面是几个java程序的常用且容易混淆的配置

Args | Comment
-------- | ---
**`-Xmx`**| 设置JVM最大可用内存
**`-Xms`**| 设置JVM初始内存
**`-Xmn`**| 设置年轻代内存大小，**整个JVM内存大小=年轻代大小 + 年老代大小 + 持久代大小** ，持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，官方推荐配置为整个堆的3/8
**`-Xss`**| 设置每个线程的堆栈大小



---

## 启动

{% highlight bash %}
[root@h102 bin]# ./mycat start 
Starting Mycat-server...
[root@h102 bin]# 
{% endhighlight %}

查看进程状态

{% highlight bash %}
[root@h102 bin]# ps faux | grep mycat
root     32813  0.0  0.0 103256   828 pts/0    S+   21:25   0:00  |       \_ grep mycat
root     32758  0.1  0.0  19124   780 ?        Sl   21:24   0:00 /usr/local/src/mycat/mycat/bin/./wrapper-linux-x86- /usr/local/src/mycat/mycat/conf/wrapper.conf wrapper.syslog.ident=mycat wrapper.pidfile=/usr/local/src/mycat/mycat/gs/mycat.pid wrapper.daemonize=TRUE wrapper.lockfile=/var/lock/subsys/mycat
[root@h102 bin]# ps faux | grep java | grep MYCAT
root     32760  4.6  4.3 1842812 84088 ?       Sl   21:24   0:02  \_ java -DMYCAT_HOME=. -server -XX:MaxPermSize=64MXX:+AggressiveOpts -XX:MaxDirectMemorySize=256m -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=14 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Xmx512m -Xms128m -Djavlibrary.path=lib -classpath lib/wrapper.jar:conf:lib/Mycat-server-1.5-GA.jar:lib/curator-framework-2.9.0.jar:lib/slf-log4j12-1.7.12.jar:lib/libwrapper-linux-ppc-64.so:lib/sequoiadb-java-driver-1.0-20150615.070208-1.jar:lib/guava-18.jar:lib/wrapper.jar:lib/mongo-java-driver-2.11.4.jar:lib/jline-0.9.94.jar:lib/libwrapper-linux-x86-32.so:lib/xml-api1.0.b2.jar:lib/log4j-1.2.17.jar:lib/ehcache-core-2.6.11.jar:lib/snakeyaml-1.16.jar:lib/libwrapper-linux-x86-64.so:lislf4j-api-1.7.12.jar:lib/leveldb-0.7.jar:lib/curator-client-2.9.0.jar:lib/netty-3.7.0.Final.jar:lib/druid-1.0.14.jarib/json-20151123.jar:lib/dom4j-1.6.1.jar:lib/zookeeper-3.4.6.jar:lib/mapdb-1.0.7.jar:lib/univocity-parsers-1.5.4.jarib/leveldb-api-0.7.jar:lib/fastjson-1.2.7.jar -Dwrapper.key=9RaInG87OR4ajDV5 -Dwrapper.port=32000 -Dwrapper.jvm.portin=31000 -Dwrapper.jvm.port.max=31999 -Dwrapper.pid=32758 -Dwrapper.version=3.2.3 -Dwrapper.native_library=wrapper -rapper.service=TRUE -Dwrapper.cpu.timeout=10 -Dwrapper.jvmid=1 org.tanukisoftware.wrapper.WrapperSimpleApp org.opencudb.MycatStartup start
[root@h102 bin]# 
[root@h102 bin]# ps -Lf 32758
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root     32758     1 32758  0    2 21:24 ?        Sl     0:00 /usr/local/src/mycat/mycat/bin/./wrapper-linux-x86-64 
root     32758     1 32759  0    2 21:24 ?        Sl     0:00 /usr/local/src/mycat/mycat/bin/./wrapper-linux-x86-64 
[root@h102 bin]# ps -Lf 32760
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root     32760 32758 32760  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32761  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32762  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32763  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32764  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32765  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32766  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32767  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32768  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32769  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32770  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32771  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32772  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32773  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32774  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32775  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32777  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32779  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32780  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32781  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32782  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32784  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32785  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32786  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32787  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32788  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32789  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32790  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32791  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32792  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32793  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32794  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
root     32760 32758 32795  0   33 21:24 ?        Sl     0:00 java -DMYCAT_HOME=. -server -XX:MaxPermSize=64M -XX:+A
[root@h102 bin]# 
{% endhighlight %}


端口开启情况(同时打开防火墙)

{% highlight bash %}
[root@h102 mycat]# netstat  -ant | grep 8066
tcp        0      0 :::8066                     :::*                        LISTEN      
[root@h102 mycat]# lsof -i :8066
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    32760 root   60u  IPv6 166556      0t0  TCP *:8066 (LISTEN)
[root@h102 mycat]# 
[root@h102 mycat]# iptables -L -nv | grep 8066
    0   0   ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8066 
[root@h102 mycat]# 
{% endhighlight %}


---

## 连接


启动服务后确保防火墙开启的情况下，可以直接使用mysql客户端进行连接



{% highlight bash %}
[root@h101 ~]# mysql -utest -ptest  -P8066 -h 192.168.100.102
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.8-mycat-1.5-GA-20160217103036 MyCat Server (OpenCloundDB)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+----------+
| DATABASE |
+----------+
| TESTDB   |
+----------+
1 row in set (0.01 sec)

mysql> use TESTDB
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables in TESTDB |
+------------------+
| company          |
| customer         |
| customer_addr    |
| employee         |
| goods            |
| hotnews          |
| orders           |
| order_items      |
| travelrecord     |
+------------------+
9 rows in set (0.01 sec)

mysql> desc company;
ERROR 3009 (HY000): java.lang.IllegalArgumentException: Invalid DataSource:0
mysql> desc goods;
ERROR 3009 (HY000): java.lang.IllegalArgumentException: Invalid DataSource:0
mysql> 
{% endhighlight %}

这是由于没指定数据源产生的报错

虽然没有连接任何数据库，但从操作体验上来看，与直接操作一个mysql没有什么区别


> **Tip:** 为什么用户名和密码是 **test** 呢，因为 **conf/server.xml** 有定义


{% highlight bash %}
<user name="test">
	<property name="password">test</property>
	<property name="schemas">TESTDB</property>
</user>
<user name="user">
	<property name="password">user</property>
	<property name="schemas">TESTDB</property>
	<property name="readOnly">true</property>
</user>
{% endhighlight %}

所以user/user也可以登录

{% highlight bash %}
[root@h101 ~]# mysql -uuser -puser  -P8066 -h 192.168.100.102
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.8-mycat-1.5-GA-20160217103036 MyCat Server (OpenCloundDB)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+----------+
| DATABASE |
+----------+
| TESTDB   |
+----------+
1 row in set (0.00 sec)

mysql> use TESTDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables in TESTDB |
+------------------+
| company          |
| customer         |
| customer_addr    |
| employee         |
| goods            |
| hotnews          |
| orders           |
| order_items      |
| travelrecord     |
+------------------+
9 rows in set (0.00 sec)

mysql> desc hotnews;
ERROR 1495 (HY000): User readonly
mysql> desc commpany;
ERROR 1495 (HY000): User readonly
mysql> desc customer;
ERROR 1495 (HY000): User readonly
mysql> 
{% endhighlight %}



---

## 停止

{% highlight bash %}
[root@h102 bin]# ./mycat  stop 
Stopping Mycat-server...
Stopped Mycat-server.
[root@h102 bin]# 
{% endhighlight %}

---

# 命令汇总

* **`java -version`**
* **`wget https://github.com/MyCATApache/Mycat-download/raw/master/1.5-GA/Mycat-server-1.5-GA-20160217103036-linux.tar.gz`**
* **`tar -xzvf Mycat-server-1.5-GA-20160217103036-linux.tar.gz`**
* **`tree mycat/`**
* **`vim conf/wrapper.conf`**
* **`grep Xm conf/wrapper.conf`**
* **`grep MaxDirectMemorySize conf/wrapper.conf`**
* **`grep -v "^#" conf/wrapper.conf | grep -v "^$"`**
* **`./mycat start`**
* **`ps faux | grep mycat`**
* **`ps faux | grep java | grep MYCAT`**
* **`ps -Lf 32758`**
* **`ps -Lf 32760`**
* **`netstat  -ant | grep 8066`**
* **`lsof -i :8066`**
* **`iptables -L -nv | grep 8066`**
* **`mysql -utest -ptest  -P8066 -h 192.168.100.102`**
* **`mysql -uuser -puser  -P8066 -h 192.168.100.102`**
* **`./mycat  stop`**


---



[mycat]:http://www.mycat.org.cn/
[mycat_doc]:http://www.mycat.org.cn/document/mycat1.5.2.pdf
[mycat_git]:https://github.com/MyCATApache/Mycat-Server
[mycat_dl]:https://github.com/MyCATApache/Mycat-download
