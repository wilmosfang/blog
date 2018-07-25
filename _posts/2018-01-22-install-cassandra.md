---
layout: post
title: "Install Cassandra"
author:  wilmosfang
date: 2018-01-22 12:11:18
image: '/assets/img/'
excerpt: 'Cassandra 的安装方法'
main-class: cassandra
color: '#90bd50'
tags:
 - cassandra
 - nosql
categories:
 - cassandra
twitter_text: 'simple process of cassandra installation '
introduction: 'installation method of Cassandra'
---


# 前言


**[Cassandra][cassandra]** 是一套开源分布式数据库软件，可以提供高容错，高性能，高可用，高弹性，可线性扩展的特性

>The Apache Cassandra database is the right choice when you need scalability and high availability without compromising performance. Linear scalability and proven fault-tolerance on commodity hardware or cloud infrastructure make it the perfect platform for mission-critical data

> **Note:** 它已经具备了那么多其它数据库羡慕的特性，那它牺牲了什么呢，在 **CAP** 理论中，它很好地实践了 **AP** 牺牲了 **C** , 它是一个最终一致性数据库，什么叫最终一致性呢，一个夸张的比喻就是 DNS

由于它的开源性，可以运行于廉价的硬件之上，高可用，强容错，可线性扩展，比较对当前互联网应用的胃口，近年来越来越受到重视，我也不免俗地想对它加深一些了解

> **Tip:** 此刻 **[Cassandra][cassandra]** 在全球数据库排名中进入了前十，在第八位上下

下面分享一下 **[Cassandra][cassandra]** 的安装方法

参考 **[Downloading Cassandra][cassandra_dl]** 和 **[Getting Started][cassandra_doc]**

> **Tip:** 当前版本 **Cassandra 3.11.1**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: b18825a04fdc473797014251a5c53648
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85656sec preferred_lft 85656sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link tentative dadfailed
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 依赖

* Java 8

>The latest version of Java 8, either the Oracle Java Standard Edition 8 or OpenJDK 8

~~~
[root@much ~]# java -version
openjdk version "1.8.0_131"
OpenJDK Runtime Environment (build 1.8.0_131-b12)
OpenJDK 64-Bit Server VM (build 25.131-b12, mixed mode)
[root@much ~]#
~~~

### 报错

> **Note:** 不要升级到最新版本 Java, 我在使用 **`openjdk version "1.8.0_161"`** 的过程中出现了问题,　无法正常启动


在 **openjdk version "1.8.0_161"** 下尝试启动服务会无法启动并且伴随如下报错

~~~
[root@much ~]# java -version
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
[root@much ~]#
~~~

**/var/log/cassandra/cassandra.log** 中有如下异常

~~~
Exception (java.lang.AbstractMethodError) encountered during startup: org.apache.cassandra.utils.JMXServerUtils$Exporter.exportObject(Ljava/rmi/Remote;ILjava/rmi/server/RMIClientSocketFactory;Ljava/rmi/server/RMIServerSocketFactory;Lsun/misc/ObjectInputFilter;)Ljava/rmi/Remote;
java.lang.AbstractMethodError: org.apache.cassandra.utils.JMXServerUtils$Exporter.exportObject(Ljava/rmi/Remote;ILjava/rmi/server/RMIClientSocketFactory;Ljava/rmi/server/RMIServerSocketFactory;Lsun/misc/ObjectInputFilter;)Ljava/rmi/Remote;
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:150)
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:135)
	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:405)
	at org.apache.cassandra.utils.JMXServerUtils.createJMXServer(JMXServerUtils.java:104)
	at org.apache.cassandra.service.CassandraDaemon.maybeInitJmx(CassandraDaemon.java:143)
	at org.apache.cassandra.service.CassandraDaemon.setup(CassandraDaemon.java:188)
	at org.apache.cassandra.service.CassandraDaemon.activate(CassandraDaemon.java:600)
	at org.apache.cassandra.service.CassandraDaemon.main(CassandraDaemon.java:689)
ERROR [main] 2018-01-22 21:49:52,241 CassandraDaemon.java:706 - Exception encountered during startup
java.lang.AbstractMethodError: org.apache.cassandra.utils.JMXServerUtils$Exporter.exportObject(Ljava/rmi/Remote;ILjava/rmi/server/RMIClientSocketFactory;Ljava/rmi/server/RMIServerSocketFactory;Lsun/misc/ObjectInputFilter;)Ljava/rmi/Remote;
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:150) ~[na:1.8.0_161]
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:135) ~[na:1.8.0_161]
	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:405) ~[na:1.8.0_161]
	at org.apache.cassandra.utils.JMXServerUtils.createJMXServer(JMXServerUtils.java:104) ~[apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.maybeInitJmx(CassandraDaemon.java:143) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.setup(CassandraDaemon.java:188) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.activate(CassandraDaemon.java:600) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.main(CassandraDaemon.java:689) [apache-cassandra-3.11.1.jar:3.11.1]
~~~

**/var/log/cassandra/debug.log** 中有如下警告和异常


~~~
WARN  [main] 2018-01-22 21:49:51,597 DatabaseDescriptor.java:546 - Only 31.900GiB free across all data volumes. Consider adding more capacity to your cluster or removing obsolete snapshots
INFO  [main] 2018-01-22 21:49:51,618 RateBasedBackPressure.java:123 - Initialized back-pressure with high ratio: 0.9, factor: 5, flow: FAST, window size: 2000.
INFO  [main] 2018-01-22 21:49:51,619 DatabaseDescriptor.java:725 - Back-pressure is disabled with strategy org.apache.cassandra.net.RateBasedBackPressure{high_ratio=0.9, factor=5, flow=FAST}.
DEBUG [main] 2018-01-22 21:49:51,640 YamlConfigurationLoader.java:108 - Loading settings from file:/etc/cassandra/default.conf/cassandra.yaml
ERROR [main] 2018-01-22 21:49:52,241 CassandraDaemon.java:706 - Exception encountered during startup
java.lang.AbstractMethodError: org.apache.cassandra.utils.JMXServerUtils$Exporter.exportObject(Ljava/rmi/Remote;ILjava/rmi/server/RMIClientSocketFactory;Ljava/rmi/server/RMIServerSocketFactory;Lsun/misc/ObjectInputFilter;)Ljava/rmi/Remote;
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:150) ~[na:1.8.0_161]
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:135) ~[na:1.8.0_161]
	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:405) ~[na:1.8.0_161]
	at org.apache.cassandra.utils.JMXServerUtils.createJMXServer(JMXServerUtils.java:104) ~[apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.maybeInitJmx(CassandraDaemon.java:143) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.setup(CassandraDaemon.java:188) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.activate(CassandraDaemon.java:600) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.main(CassandraDaemon.java:689) [apache-cassandra-3.11.1.jar:3.11.1]
~~~

**/var/log/cassandra/system.log** 中有如下警告与异常

~~~
WARN  [main] 2018-01-22 21:49:51,597 DatabaseDescriptor.java:546 - Only 31.900GiB free across all data volumes. Consider adding more capacity to your cluster or removing obsolete snapshots
INFO  [main] 2018-01-22 21:49:51,618 RateBasedBackPressure.java:123 - Initialized back-pressure with high ratio: 0.9, factor: 5, flow: FAST, window size: 2000.
INFO  [main] 2018-01-22 21:49:51,619 DatabaseDescriptor.java:725 - Back-pressure is disabled with strategy org.apache.cassandra.net.RateBasedBackPressure{high_ratio=0.9, factor=5, flow=FAST}.
ERROR [main] 2018-01-22 21:49:52,241 CassandraDaemon.java:706 - Exception encountered during startup
java.lang.AbstractMethodError: org.apache.cassandra.utils.JMXServerUtils$Exporter.exportObject(Ljava/rmi/Remote;ILjava/rmi/server/RMIClientSocketFactory;Ljava/rmi/server/RMIServerSocketFactory;Lsun/misc/ObjectInputFilter;)Ljava/rmi/Remote;
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:150) ~[na:1.8.0_161]
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:135) ~[na:1.8.0_161]
	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:405) ~[na:1.8.0_161]
	at org.apache.cassandra.utils.JMXServerUtils.createJMXServer(JMXServerUtils.java:104) ~[apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.maybeInitJmx(CassandraDaemon.java:143) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.setup(CassandraDaemon.java:188) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.activate(CassandraDaemon.java:600) [apache-cassandra-3.11.1.jar:3.11.1]
	at org.apache.cassandra.service.CassandraDaemon.main(CassandraDaemon.java:689) [apache-cassandra-3.11.1.jar:3.11.1]
~~~

system.log 中的报错与 debug.log 中的一致，大体意思是本地磁盘空间太小，**[Not enough space for compaction messages](https://github.com/docker-library/cassandra/issues/103#issuecomment-294247158)**

这里有更深层的讲解

**[Cassandra - compaction stuck
](https://stackoverflow.com/questions/33432779/cassandra-compaction-stuck/33433755#33433755)**

不过我的解决办法是，降级 Java JDK 版本

* Python 2.7

>For using cqlsh, the latest version of Python 2.7

~~~
[root@much ~]# python -V
Python 2.7.5
[root@much ~]#
~~~

## 构建仓库

~~~
[root@much ~]# vim /etc/yum.repos.d/cassandra.repo
[root@much ~]# cat /etc/yum.repos.d/cassandra.repo
[cassandra]
name=Apache Cassandra
baseurl=https://www.apache.org/dist/cassandra/redhat/311x/
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://www.apache.org/dist/cassandra/KEYS
[root@much ~]#
~~~

这里我就去掉了 gpgcheck, 生产环境下还是建议检查一下的，我这是图省事儿


## 安装软件

~~~
[root@much ~]# yum list all | grep -i cassandra
cassandra.noarch                            3.11.1-1                   cassandra
cassandra-tools.noarch                      3.11.1-1                   cassandra
[root@much ~]# yum install cassandra.noarch
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package cassandra.noarch 0:3.11.1-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package            Arch            Version            Repository          Size
================================================================================
Installing:
 cassandra          noarch          3.11.1-1           cassandra           28 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 28 M
Installed size: 37 M
Is this ok [y/d/N]: y
Downloading packages:
cassandra-3.11.1-1.noarch.rpm                              |  28 MB   01:31     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : cassandra-3.11.1-1.noarch                                    1/1
  Verifying  : cassandra-3.11.1-1.noarch                                    1/1

Installed:
  cassandra.noarch 0:3.11.1-1                                                   

Complete!
[root@much ~]#
~~~


可以顺便看看此软件包里有些什么内容

~~~
[root@much ~]# rpm -ql cassandra.noarch
/etc/cassandra
/etc/cassandra/default.conf
/etc/cassandra/default.conf/README.txt
/etc/cassandra/default.conf/cassandra-env.sh
/etc/cassandra/default.conf/cassandra-env.sh.orig
/etc/cassandra/default.conf/cassandra-jaas.config
/etc/cassandra/default.conf/cassandra-rackdc.properties
/etc/cassandra/default.conf/cassandra-topology.properties
/etc/cassandra/default.conf/cassandra.yaml
/etc/cassandra/default.conf/cassandra.yaml.orig
/etc/cassandra/default.conf/commitlog_archiving.properties
/etc/cassandra/default.conf/cqlshrc.sample
/etc/cassandra/default.conf/hotspot_compiler
/etc/cassandra/default.conf/jvm.options
/etc/cassandra/default.conf/logback-tools.xml
/etc/cassandra/default.conf/logback.xml
/etc/cassandra/default.conf/metrics-reporter-config-sample.yaml
/etc/cassandra/default.conf/triggers
/etc/cassandra/default.conf/triggers/README.txt
/etc/default/cassandra
/etc/rc.d/init.d/cassandra
/etc/security/limits.d/cassandra.conf
/usr/bin/cassandra-stress
/usr/bin/cqlsh
/usr/bin/cqlsh.py
/usr/bin/debug-cql
/usr/bin/nodetool
/usr/bin/sstableloader
/usr/bin/sstablescrub
/usr/bin/sstableupgrade
/usr/bin/sstableutil
/usr/bin/sstableverify
/usr/bin/stop-server
/usr/lib/python2.7/site-packages/cassandra_pylib-0.0.0-py2.7.egg-info
/usr/lib/python2.7/site-packages/cqlshlib
/usr/lib/python2.7/site-packages/cqlshlib/__init__.py
/usr/lib/python2.7/site-packages/cqlshlib/copyutil.py
/usr/lib/python2.7/site-packages/cqlshlib/cql3handling.py
/usr/lib/python2.7/site-packages/cqlshlib/cqlhandling.py
/usr/lib/python2.7/site-packages/cqlshlib/cqlshhandling.py
/usr/lib/python2.7/site-packages/cqlshlib/displaying.py
/usr/lib/python2.7/site-packages/cqlshlib/formatting.py
/usr/lib/python2.7/site-packages/cqlshlib/helptopics.py
/usr/lib/python2.7/site-packages/cqlshlib/pylexotron.py
/usr/lib/python2.7/site-packages/cqlshlib/saferscanner.py
/usr/lib/python2.7/site-packages/cqlshlib/sslhandling.py
/usr/lib/python2.7/site-packages/cqlshlib/tracing.py
/usr/lib/python2.7/site-packages/cqlshlib/util.py
/usr/lib/python2.7/site-packages/cqlshlib/wcwidth.py
/usr/sbin/cassandra
/usr/share/cassandra
/usr/share/cassandra/apache-cassandra-3.11.1.jar
/usr/share/cassandra/apache-cassandra-thrift-3.11.1.jar
/usr/share/cassandra/cassandra.in.sh
/usr/share/cassandra/lib
/usr/share/cassandra/lib/HdrHistogram-2.1.9.jar
/usr/share/cassandra/lib/ST4-4.0.8.jar
/usr/share/cassandra/lib/airline-0.6.jar
/usr/share/cassandra/lib/antlr-runtime-3.5.2.jar
/usr/share/cassandra/lib/asm-5.0.4.jar
/usr/share/cassandra/lib/caffeine-2.2.6.jar
/usr/share/cassandra/lib/cassandra-driver-core-3.0.1-shaded.jar
/usr/share/cassandra/lib/cassandra-driver-internal-only-3.10.zip
/usr/share/cassandra/lib/commons-cli-1.1.jar
/usr/share/cassandra/lib/commons-codec-1.9.jar
/usr/share/cassandra/lib/commons-lang3-3.1.jar
/usr/share/cassandra/lib/commons-math3-3.2.jar
/usr/share/cassandra/lib/compress-lzf-0.8.4.jar
/usr/share/cassandra/lib/concurrent-trees-2.4.0.jar
/usr/share/cassandra/lib/concurrentlinkedhashmap-lru-1.4.jar
/usr/share/cassandra/lib/disruptor-3.0.1.jar
/usr/share/cassandra/lib/ecj-4.4.2.jar
/usr/share/cassandra/lib/futures-2.1.6-py2.py3-none-any.zip
/usr/share/cassandra/lib/guava-18.0.jar
/usr/share/cassandra/lib/high-scale-lib-1.0.6.jar
/usr/share/cassandra/lib/hppc-0.5.4.jar
/usr/share/cassandra/lib/jackson-core-asl-1.9.2.jar
/usr/share/cassandra/lib/jackson-mapper-asl-1.9.2.jar
/usr/share/cassandra/lib/jamm-0.3.0.jar
/usr/share/cassandra/lib/javax.inject.jar
/usr/share/cassandra/lib/jbcrypt-0.3m.jar
/usr/share/cassandra/lib/jcl-over-slf4j-1.7.7.jar
/usr/share/cassandra/lib/jctools-core-1.2.1.jar
/usr/share/cassandra/lib/jflex-1.6.0.jar
/usr/share/cassandra/lib/jna-4.2.2.jar
/usr/share/cassandra/lib/joda-time-2.4.jar
/usr/share/cassandra/lib/json-simple-1.1.jar
/usr/share/cassandra/lib/jstackjunit-0.0.1.jar
/usr/share/cassandra/lib/libthrift-0.9.2.jar
/usr/share/cassandra/lib/licenses
/usr/share/cassandra/lib/licenses/ST4-4.0.8.txt
/usr/share/cassandra/lib/licenses/airline-0.6.txt
/usr/share/cassandra/lib/licenses/antlr-runtime-3.5.2.txt
/usr/share/cassandra/lib/licenses/asm-5.0.4.txt
/usr/share/cassandra/lib/licenses/caffeine-2.2.6.txt
/usr/share/cassandra/lib/licenses/cassandra-driver-3.0.1.txt
/usr/share/cassandra/lib/licenses/commons-cli-1.1.txt
/usr/share/cassandra/lib/licenses/commons-codec-1.9.txt
/usr/share/cassandra/lib/licenses/commons-lang3-3.1.txt
/usr/share/cassandra/lib/licenses/commons-math3-3.2.txt
/usr/share/cassandra/lib/licenses/compress-lzf-0.8.4.txt
/usr/share/cassandra/lib/licenses/concurrent-trees-2.4.0.txt
/usr/share/cassandra/lib/licenses/concurrentlinkedhashmap-lru-1.4.txt
/usr/share/cassandra/lib/licenses/disruptor-3.0.1.txt
/usr/share/cassandra/lib/licenses/ecj-4.4.2.txt
/usr/share/cassandra/lib/licenses/futures-2.1.6.txt
/usr/share/cassandra/lib/licenses/guava-18.0.txt
/usr/share/cassandra/lib/licenses/hdrhistogram-2.1.9.txt
/usr/share/cassandra/lib/licenses/high-scale-lib-1.0.6.txt
/usr/share/cassandra/lib/licenses/hppc-0.5.4.txt
/usr/share/cassandra/lib/licenses/jackson-core-asl-1.9.2.txt
/usr/share/cassandra/lib/licenses/jackson-mapper-asl-1.9.2.txt
/usr/share/cassandra/lib/licenses/jamm-0.3.0.txt
/usr/share/cassandra/lib/licenses/javax.inject.txt
/usr/share/cassandra/lib/licenses/jbcrypt-0.3m.txt
/usr/share/cassandra/lib/licenses/jcl-over-slf4j-1.7.7.txt
/usr/share/cassandra/lib/licenses/jctools-core-1.2.1.txt
/usr/share/cassandra/lib/licenses/jflex-1.6.0.txt
/usr/share/cassandra/lib/licenses/jna-4.2.2.txt
/usr/share/cassandra/lib/licenses/joda-time-2.4.txt
/usr/share/cassandra/lib/licenses/json-simple-1.1.txt
/usr/share/cassandra/lib/licenses/jstackjunit-0.0.1.txt
/usr/share/cassandra/lib/licenses/libthrift-0.9.2.txt
/usr/share/cassandra/lib/licenses/log4j-over-slf4j-1.7.7.txt
/usr/share/cassandra/lib/licenses/logback-classic-1.1.3.txt
/usr/share/cassandra/lib/licenses/logback-core-1.1.3.txt
/usr/share/cassandra/lib/licenses/lz4-1.3.0.txt
/usr/share/cassandra/lib/licenses/metrics-core-3.1.0.txt
/usr/share/cassandra/lib/licenses/metrics-jvm-3.1.0.txt
/usr/share/cassandra/lib/licenses/metrics-logback-3.1.0.txt
/usr/share/cassandra/lib/licenses/netty-all-4.0.44.Final.txt
/usr/share/cassandra/lib/licenses/ohc-0.4.4.txt
/usr/share/cassandra/lib/licenses/reporter-config-base-3.0.3.txt
/usr/share/cassandra/lib/licenses/reporter-config3-3.0.3.txt
/usr/share/cassandra/lib/licenses/sigar-1.6.4.txt
/usr/share/cassandra/lib/licenses/six-1.7.3.txt
/usr/share/cassandra/lib/licenses/slf4j-api-1.7.7.txt
/usr/share/cassandra/lib/licenses/snakeyaml-1.11.txt
/usr/share/cassandra/lib/licenses/snappy-java-1.1.1.7.txt
/usr/share/cassandra/lib/licenses/snowball-stemmer-1.3.0.581.1.txt
/usr/share/cassandra/lib/licenses/stream-2.5.2.txt
/usr/share/cassandra/lib/licenses/thrift-server-0.3.7.txt
/usr/share/cassandra/lib/log4j-over-slf4j-1.7.7.jar
/usr/share/cassandra/lib/logback-classic-1.1.3.jar
/usr/share/cassandra/lib/logback-core-1.1.3.jar
/usr/share/cassandra/lib/lz4-1.3.0.jar
/usr/share/cassandra/lib/metrics-core-3.1.0.jar
/usr/share/cassandra/lib/metrics-jvm-3.1.0.jar
/usr/share/cassandra/lib/metrics-logback-3.1.0.jar
/usr/share/cassandra/lib/netty-all-4.0.44.Final.jar
/usr/share/cassandra/lib/ohc-core-0.4.4.jar
/usr/share/cassandra/lib/ohc-core-j8-0.4.4.jar
/usr/share/cassandra/lib/reporter-config-base-3.0.3.jar
/usr/share/cassandra/lib/reporter-config3-3.0.3.jar
/usr/share/cassandra/lib/sigar-1.6.4.jar
/usr/share/cassandra/lib/sigar-bin
/usr/share/cassandra/lib/sigar-bin/libsigar-amd64-freebsd-6.so
/usr/share/cassandra/lib/sigar-bin/libsigar-amd64-linux.so
/usr/share/cassandra/lib/sigar-bin/libsigar-amd64-solaris.so
/usr/share/cassandra/lib/sigar-bin/libsigar-ia64-hpux-11.sl
/usr/share/cassandra/lib/sigar-bin/libsigar-ia64-linux.so
/usr/share/cassandra/lib/sigar-bin/libsigar-pa-hpux-11.sl
/usr/share/cassandra/lib/sigar-bin/libsigar-ppc-aix-5.so
/usr/share/cassandra/lib/sigar-bin/libsigar-ppc-linux.so
/usr/share/cassandra/lib/sigar-bin/libsigar-ppc64-aix-5.so
/usr/share/cassandra/lib/sigar-bin/libsigar-ppc64-linux.so
/usr/share/cassandra/lib/sigar-bin/libsigar-s390x-linux.so
/usr/share/cassandra/lib/sigar-bin/libsigar-sparc-solaris.so
/usr/share/cassandra/lib/sigar-bin/libsigar-sparc64-solaris.so
/usr/share/cassandra/lib/sigar-bin/libsigar-universal-macosx.dylib
/usr/share/cassandra/lib/sigar-bin/libsigar-universal64-macosx.dylib
/usr/share/cassandra/lib/sigar-bin/libsigar-x86-freebsd-5.so
/usr/share/cassandra/lib/sigar-bin/libsigar-x86-freebsd-6.so
/usr/share/cassandra/lib/sigar-bin/libsigar-x86-linux.so
/usr/share/cassandra/lib/sigar-bin/libsigar-x86-solaris.so
/usr/share/cassandra/lib/six-1.7.3-py2.py3-none-any.zip
/usr/share/cassandra/lib/slf4j-api-1.7.7.jar
/usr/share/cassandra/lib/snakeyaml-1.11.jar
/usr/share/cassandra/lib/snappy-java-1.1.1.7.jar
/usr/share/cassandra/lib/snowball-stemmer-1.3.0.581.1.jar
/usr/share/cassandra/lib/stream-2.5.2.jar
/usr/share/cassandra/lib/thrift-server-0.3.7.jar
/usr/share/cassandra/stress.jar
/usr/share/doc/cassandra-3.11.1
/usr/share/doc/cassandra-3.11.1/CHANGES.txt
/usr/share/doc/cassandra-3.11.1/LICENSE.txt
/usr/share/doc/cassandra-3.11.1/NEWS.txt
/usr/share/doc/cassandra-3.11.1/NOTICE.txt
/usr/share/doc/cassandra-3.11.1/README.asc
/var/lib/cassandra/commitlog
/var/lib/cassandra/data
/var/lib/cassandra/hints
/var/lib/cassandra/saved_caches
/var/log/cassandra
/var/run/cassandra
[root@much ~]#
~~~

## 启动服务

~~~
[root@much ~]# /etc/init.d/cassandra start
Reloading systemd:                                         [  OK  ]
Starting cassandra (via systemctl):                        [  OK  ]
[root@much ~]# systemctl status cassandra
● cassandra.service - LSB: distributed storage system for structured data
   Loaded: loaded (/etc/rc.d/init.d/cassandra; bad; vendor preset: disabled)
   Active: active (running) since 一 2018-01-22 22:39:58 CST; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 2704 ExecStart=/etc/rc.d/init.d/cassandra start (code=exited, status=0/SUCCESS)
 Main PID: 2783 (java)
   CGroup: /system.slice/cassandra.service
           ‣ 2783 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_6...

1月 22 22:39:47 much systemd[1]: Starting LSB: distributed storage system .....
1月 22 22:39:47 much su[2712]: (to cassandra) root on none
1月 22 22:39:58 much cassandra[2704]: Starting Cassandra: OK
1月 22 22:39:58 much systemd[1]: Started LSB: distributed storage system f...a.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]#
[root@much ~]# ps faux | grep cassandra
root      3343  0.0  0.0 112648  1016 pts/1    S+   23:13   0:00          \_ grep --color=auto cassandra
cassand+  2783  1.5 67.3 2949792 1380100 ?     SLl  22:39   0:32 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/jre/bin/java -Xloggc:/var/log/cassandra/gc.log -ea -XX:+UseThreadPriorities -XX:ThreadPriorityPolicy=42 -XX:+HeapDumpOnOutOfMemoryError -Xss256k -XX:StringTableSize=1000003 -XX:+AlwaysPreTouch -XX:-UseBiasedLocking -XX:+UseTLAB -XX:+ResizeTLAB -XX:+UseNUMA -XX:+PerfDisableSharedMem -Djava.net.preferIPv4Stack=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=1 -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSWaitDuration=10000 -XX:+CMSParallelInitialMarkEnabled -XX:+CMSEdenChunksRecordAlways -XX:+CMSClassUnloadingEnabled -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -XX:+PrintPromotionFailure -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=10M -Xms1000M -Xmx1000M -Xmn200M -XX:CompileCommandFile=/etc/cassandra/conf/hotspot_compiler -javaagent:/usr/share/cassandra/lib/jamm-0.3.0.jar -Dcassandra.jmx.local.port=7199 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.password.file=/etc/cassandra/jmxremote.password -Djava.library.path=/usr/share/cassandra/lib/sigar-bin -Dlogback.configurationFile=logback.xml -Dcassandra.logdir=/var/log/cassandra -Dcassandra.storagedir= -Dcassandra-pidfile=/var/run/cassandra/cassandra.pid -cp /etc/cassandra/conf:/usr/share/cassandra/lib/airline-0.6.jar:/usr/share/cassandra/lib/antlr-runtime-3.5.2.jar:/usr/share/cassandra/lib/asm-5.0.4.jar:/usr/share/cassandra/lib/caffeine-2.2.6.jar:/usr/share/cassandra/lib/cassandra-driver-core-3.0.1-shaded.jar:/usr/share/cassandra/lib/commons-cli-1.1.jar:/usr/share/cassandra/lib/commons-codec-1.9.jar:/usr/share/cassandra/lib/commons-lang3-3.1.jar:/usr/share/cassandra/lib/commons-math3-3.2.jar:/usr/share/cassandra/lib/compress-lzf-0.8.4.jar:/usr/share/cassandra/lib/concurrentlinkedhashmap-lru-1.4.jar:/usr/share/cassandra/lib/concurrent-trees-2.4.0.jar:/usr/share/cassandra/lib/disruptor-3.0.1.jar:/usr/share/cassandra/lib/ecj-4.4.2.jar:/usr/share/cassandra/lib/guava-18.0.jar:/usr/share/cassandra/lib/HdrHistogram-2.1.9.jar:/usr/share/cassandra/lib/high-scale-lib-1.0.6.jar:/usr/share/cassandra/lib/hppc-0.5.4.jar:/usr/share/cassandra/lib/jackson-core-asl-1.9.2.jar:/usr/share/cassandra/lib/jackson-mapper-asl-1.9.2.jar:/usr/share/cassandra/lib/jamm-0.3.0.jar:/usr/share/cassandra/lib/javax.inject.jar:/usr/share/cassandra/lib/jbcrypt-0.3m.jar:/usr/share/cassandra/lib/jcl-over-slf4j-1.7.7.jar:/usr/share/cassandra/lib/jctools-core-1.2.1.jar:/usr/share/cassandra/lib/jflex-1.6.0.jar:/usr/share/cassandra/lib/jna-4.2.2.jar:/usr/share/cassandra/lib/joda-time-2.4.jar:/usr/share/cassandra/lib/json-simple-1.1.jar:/usr/share/cassandra/lib/jstackjunit-0.0.1.jar:/usr/share/cassandra/lib/libthrift-0.9.2.jar:/usr/share/cassandra/lib/log4j-over-slf4j-1.7.7.jar:/usr/share/cassandra/lib/logback-classic-1.1.3.jar:/usr/share/cassandra/lib/logback-core-1.1.3.jar:/usr/share/cassandra/lib/lz4-1.3.0.jar:/usr/share/cassandra/lib/metrics-core-3.1.0.jar:/usr/share/cassandra/lib/metrics-jvm-3.1.0.jar:/usr/share/cassandra/lib/metrics-logback-3.1.0.jar:/usr/share/cassandra/lib/netty-all-4.0.44.Final.jar:/usr/share/cassandra/lib/ohc-core-0.4.4.jar:/usr/share/cassandra/lib/ohc-core-j8-0.4.4.jar:/usr/share/cassandra/lib/reporter-config3-3.0.3.jar:/usr/share/cassandra/lib/reporter-config-base-3.0.3.jar:/usr/share/cassandra/lib/sigar-1.6.4.jar:/usr/share/cassandra/lib/slf4j-api-1.7.7.jar:/usr/share/cassandra/lib/snakeyaml-1.11.jar:/usr/share/cassandra/lib/snappy-java-1.1.1.7.jar:/usr/share/cassandra/lib/snowball-stemmer-1.3.0.581.1.jar:/usr/share/cassandra/lib/ST4-4.0.8.jar:/usr/share/cassandra/lib/stream-2.5.2.jar:/usr/share/cassandra/lib/thrift-server-0.3.7.jar:/usr/share/cassandra/apache-cassandra-3.11.1.jar:/usr/share/cassandra/apache-cassandra-thrift-3.11.1.jar:/usr/share/cassandra/stress.jar: org.apache.cassandra.service.CassandraDaemon
[root@much ~]#
[root@much ~]# netstat  -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 127.0.0.1:9042          0.0.0.0:*               LISTEN      2783/java           
tcp        0      0 127.0.0.1:44210         0.0.0.0:*               LISTEN      2783/java           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1746/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1526/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1530/cupsd          
tcp        0      0 127.0.0.1:7000          0.0.0.0:*               LISTEN      2783/java           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1644/master         
tcp        0      0 0.0.0.0:7199            0.0.0.0:*               LISTEN      2783/java           
tcp        0      0 192.168.56.210:22       192.168.56.1:52284      ESTABLISHED 2497/sshd: root@pts
tcp        0      0 192.168.56.210:22       192.168.56.1:52354      ESTABLISHED 2875/sshd: root@pts
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::22                   :::*                    LISTEN      1526/sshd           
tcp6       0      0 ::1:631                 :::*                    LISTEN      1530/cupsd          
tcp6       0      0 ::1:25                  :::*                    LISTEN      1644/master         
[root@much ~]#
~~~

## 停止服务

~~~
[root@much ~]# systemctl status cassandra
● cassandra.service - LSB: distributed storage system for structured data
   Loaded: loaded (/etc/rc.d/init.d/cassandra; bad; vendor preset: disabled)
   Active: active (running) since 一 2018-01-22 22:39:58 CST; 35min ago
     Docs: man:systemd-sysv-generator(8)
  Process: 2704 ExecStart=/etc/rc.d/init.d/cassandra start (code=exited, status=0/SUCCESS)
 Main PID: 2783 (java)
   CGroup: /system.slice/cassandra.service
           ‣ 2783 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_6...

1月 22 22:39:47 much systemd[1]: Starting LSB: distributed storage system .....
1月 22 22:39:47 much su[2712]: (to cassandra) root on none
1月 22 22:39:58 much cassandra[2704]: Starting Cassandra: OK
1月 22 22:39:58 much systemd[1]: Started LSB: distributed storage system f...a.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]# systemctl stop  cassandra
[root@much ~]# ps faux | grep cassandra
root      3406  0.0  0.0 112648  1016 pts/0    S+   23:15   0:00  |       \_ grep --color=auto cassandra
[root@much ~]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1746/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1526/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1530/cupsd          
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1644/master         
tcp        0      0 192.168.56.210:22       192.168.56.1:52284      ESTABLISHED 2497/sshd: root@pts
tcp        0      0 192.168.56.210:22       192.168.56.1:52354      ESTABLISHED 2875/sshd: root@pts
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::22                   :::*                    LISTEN      1526/sshd           
tcp6       0      0 ::1:631                 :::*                    LISTEN      1530/cupsd          
tcp6       0      0 ::1:25                  :::*                    LISTEN      1644/master         
[root@much ~]# systemctl status cassandra
● cassandra.service - LSB: distributed storage system for structured data
   Loaded: loaded (/etc/rc.d/init.d/cassandra; bad; vendor preset: disabled)
   Active: failed (Result: exit-code) since 一 2018-01-22 23:15:40 CST; 39s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3371 ExecStop=/etc/rc.d/init.d/cassandra stop (code=exited, status=0/SUCCESS)
  Process: 2704 ExecStart=/etc/rc.d/init.d/cassandra start (code=exited, status=0/SUCCESS)
 Main PID: 2783 (code=exited, status=143)

1月 22 22:39:47 much su[2712]: (to cassandra) root on none
1月 22 22:39:58 much cassandra[2704]: Starting Cassandra: OK
1月 22 22:39:58 much systemd[1]: Started LSB: distributed storage system for structured data.
1月 22 23:15:36 much systemd[1]: Stopping LSB: distributed storage system for structured data...
1月 22 23:15:36 much su[3379]: (to cassandra) root on none
1月 22 23:15:39 much cassandra[3371]: Shutdown Cassandra: OK
1月 22 23:15:40 much systemd[1]: cassandra.service: main process exited, code=exited, status=143/n/a
1月 22 23:15:40 much systemd[1]: Stopped LSB: distributed storage system for structured data.
1月 22 23:15:40 much systemd[1]: Unit cassandra.service entered failed state.
1月 22 23:15:40 much systemd[1]: cassandra.service failed.
[root@much ~]#
~~~

## 连接服务

使用自带的 **cqlsh** 客户端进行连接

~~~
[root@much ~]# cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> select cluster_name from system.local;

 cluster_name
--------------
 Test Cluster

(1 rows)
cqlsh> select listen_address from system.local;

 listen_address
----------------
      127.0.0.1

(1 rows)
cqlsh> select cluster_name,listen_address from system.local;

 cluster_name | listen_address
--------------+----------------
 Test Cluster |      127.0.0.1

(1 rows)
cqlsh>
~~~

这个客户端相较其它客户端更友好，因为是彩色输出

---

# 总结

总体来讲安装过程是一个最典型的 YUM 源安装

* TOC
{:toc}


---

[cassandra]:http://cassandra.apache.org/
[cassandra_dl]:http://cassandra.apache.org/download/
[cassandra_doc]:http://cassandra.apache.org/doc/latest/getting_started/index.html
