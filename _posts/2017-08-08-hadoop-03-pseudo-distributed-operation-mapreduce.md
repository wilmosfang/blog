---
layout:  post
title:  Hadoop 在 Centos7 下的单机布署(三).MapReduce.Pseudo-Distributed Operation
author:  wilmosfang
tags:  hadoop mapreduce
categories:  hadoop 
wc: 405 1370 14453
excerpt:  Centos7 下布署 Pseudo-Distributed 模式的 MapReduce
comments: true
---


# 前言

随着人工智能在各领域的大面积普及，数据分析在各个层面的广泛应用

其底层的技术在圈子里越来越受到重视和热捧

**[Apache Hadoop][hadoop]** 技术栈或者说技术生态圈绝对是一个不容忽视的中坚力量

**[Apache Hadoop][hadoop]** 主要包含以下几个模块：

* **Hadoop Common** : 支持其它 **hadoop** 模块的通用工具
* **Hadoop Distributed File System** : 简称 **HDFS** , 给应用数据提供高吞吐性能的分布式文件系统
* **Hadoop YARN** : 工作调度与集群资源管理的框架
* **Hadoop MapReduce** : 大数据集的并行处理系统

Hadoop 生态圈中的其它项目可以参考 **[Hadoop-related projects][hadoop]**

> **Tip:**  当前的最新稳定版为 **Hadoop Release 2.8.1** 发布于 **08 June, 2017**

前面根据官方的文档给出最新版 Hadoop  在 Centos7 下的单机布署方案，详细可以参考 **[Hadoop 在 Centos7 下的单机布署(一).standalone][hadoop_01_standalone]** 和 **[hadoop 在 Centos7 下的单机布署(二).HDFS][hadoop_02__hdfs]**

这里基于前面的操作，给出最新版 Hadoop  在 Centos7 下 **MapReduce** 的伪分布模式(单机)的布署方案

此文章借鉴了 **[Hadoop Wiki][hadoop_doc]** 中的部分内容和  **[Running Hadoop on Ubuntu Linux (Single-Node Cluster)][install_hadoop_on_ubuntu]** 中的部分操作

---

# 概要

* TOC
{:toc}



---


## 系统环境

~~~
[root@much ~]# hostnamectl 
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 8cfd2a8aec4f4235b4776f7cd2fdfdb1
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
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84190sec preferred_lft 84190sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.207/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed3:ece7/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]# uname -a 
Linux much 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@much ~]# 
~~~

---

## 目标

* 构建一个 hadoop 的单点伪分布集群
* mapreduce 的基本操作

---

## 启动 hdfs

参考 **[Hadoop 在 Centos7 下的单机布署(二).hdfs][hadoop_02__hdfs]** 中的操作构建出脱机版的 Hadoop hdfs

启动 hdfs

~~~
[hadoop@much hadoop-2.8.1]$ jps
7413 Jps
[hadoop@much hadoop-2.8.1]$ sbin/start-dfs.sh 
Starting namenodes on [localhost]
localhost: starting namenode, logging to /home/hadoop/tmp/hadoop/hadoop-2.8.1/logs/hadoop-hadoop-namenode-much.out
localhost: starting datanode, logging to /home/hadoop/tmp/hadoop/hadoop-2.8.1/logs/hadoop-hadoop-datanode-much.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /home/hadoop/tmp/hadoop/hadoop-2.8.1/logs/hadoop-hadoop-secondarynamenode-much.out
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /abc
Found 2 items
-rw-r--r--   1 hadoop supergroup       2637 2017-08-06 20:13 /abc/21centry
-rw-r--r--   1 hadoop supergroup    1580879 2017-08-06 19:23 /abc/4300-0.txt
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /
Found 6 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-06 20:13 /abc
drwxr-xr-x   - hadoop supergroup          0 2017-08-06 19:15 /input
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 23:26 /output
drwxr-xr-x   - hadoop supergroup          0 2017-08-08 00:29 /test
drwxr-xr-x   - hadoop supergroup          0 2017-08-08 00:02 /test_output
drwx------   - hadoop supergroup          0 2017-08-06 19:16 /tmp
[hadoop@much hadoop-2.8.1]$ jps
7683 DataNode
7540 NameNode
7862 SecondaryNameNode
8523 Jps
[hadoop@much hadoop-2.8.1]$
~~~

---

## 配置 YARN 和 MapReduce

在单点系统中 (伪分布式hadoop) 可以通过 YARN 来管理 MapReduce 

只用配置少数几个参数，然后运行资源管理和节点管理进程

配置 **`etc/hadoop/mapred-site.xml`**

~~~
[hadoop@much hadoop-2.8.1]$ cp etc/hadoop/mapred-site.xml.template  etc/hadoop/mapred-site.xml
[hadoop@much hadoop-2.8.1]$ vim etc/hadoop/mapred-site.xml
[hadoop@much hadoop-2.8.1]$ cat etc/hadoop/mapred-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
[hadoop@much hadoop-2.8.1]$
~~~

配置 **`etc/hadoop/yarn-site.xml`**


~~~
[hadoop@much hadoop-2.8.1]$ vim  etc/hadoop/yarn-site.xml 
[hadoop@much hadoop-2.8.1]$ cat  etc/hadoop/yarn-site.xml 
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>  
        <name>yarn.resourcemanager.address</name>  
        <value>localhost:8032</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.scheduler.address</name>  
        <value>localhost:8030</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.resource-tracker.address</name>  
        <value>localhost:8031</value>  
    </property> 
</configuration>
[hadoop@much hadoop-2.8.1]$ 
~~~

> **Tip:** 8030-8032 的端口也可以不用配置，因为默认也会使用这几个端口

~~~
    <property>  
        <name>yarn.resourcemanager.address</name>  
        <value>localhost:8032</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.scheduler.address</name>  
        <value>localhost:8030</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.resource-tracker.address</name>  
        <value>localhost:8031</value>  
    </property> 
~~~



---

## 启动 YARN

~~~
[hadoop@much hadoop-2.8.1]$ sbin/start-yarn.sh 
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/tmp/hadoop/hadoop-2.8.1/logs/yarn-hadoop-resourcemanager-much.out
localhost: starting nodemanager, logging to /home/hadoop/tmp/hadoop/hadoop-2.8.1/logs/yarn-hadoop-nodemanager-much.out
[hadoop@much hadoop-2.8.1]$ jps
7683 DataNode
7540 NameNode
7862 SecondaryNameNode
8727 NodeManager
9063 Jps
8602 ResourceManager
[hadoop@much hadoop-2.8.1]$
~~~

这个过程中启动了一个 **NodeManager** 和一个 **ResourceManager**


---

## 使用 mapreduce 的方式运行 wordcount 小程序

~~~
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /abc
Found 2 items
-rw-r--r--   1 hadoop supergroup       2637 2017-08-06 20:13 /abc/21centry
-rw-r--r--   1 hadoop supergroup    1580879 2017-08-06 19:23 /abc/4300-0.txt
[hadoop@much hadoop-2.8.1]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar  wordcount /abc/21centry  /x_put
17/08/08 01:30:09 INFO client.RMProxy: Connecting to ResourceManager at localhost/127.0.0.1:8032
17/08/08 01:30:10 INFO input.FileInputFormat: Total input files to process : 1
17/08/08 01:30:11 INFO mapreduce.JobSubmitter: number of splits:1
17/08/08 01:30:12 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1502126797087_0001
17/08/08 01:30:12 INFO impl.YarnClientImpl: Submitted application application_1502126797087_0001
17/08/08 01:30:12 INFO mapreduce.Job: The url to track the job: http://much:8088/proxy/application_1502126797087_0001/
17/08/08 01:30:12 INFO mapreduce.Job: Running job: job_1502126797087_0001
17/08/08 01:30:18 INFO mapreduce.Job: Job job_1502126797087_0001 running in uber mode : false
17/08/08 01:30:18 INFO mapreduce.Job:  map 0% reduce 0%
17/08/08 01:30:22 INFO mapreduce.Job:  map 100% reduce 0%
17/08/08 01:30:28 INFO mapreduce.Job:  map 100% reduce 100%
17/08/08 01:30:30 INFO mapreduce.Job: Job job_1502126797087_0001 completed successfully
17/08/08 01:30:31 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=3378
		FILE: Number of bytes written=279039
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=2736
		HDFS: Number of bytes written=2298
		HDFS: Number of read operations=6
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=2067
		Total time spent by all reduces in occupied slots (ms)=3295
		Total time spent by all map tasks (ms)=2067
		Total time spent by all reduce tasks (ms)=3295
		Total vcore-milliseconds taken by all map tasks=2067
		Total vcore-milliseconds taken by all reduce tasks=3295
		Total megabyte-milliseconds taken by all map tasks=2116608
		Total megabyte-milliseconds taken by all reduce tasks=3374080
	Map-Reduce Framework
		Map input records=17
		Map output records=487
		Map output bytes=4577
		Map output materialized bytes=3378
		Input split bytes=99
		Combine input records=487
		Combine output records=270
		Reduce input groups=270
		Reduce shuffle bytes=3378
		Reduce input records=270
		Reduce output records=270
		Spilled Records=540
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=119
		CPU time spent (ms)=710
		Physical memory (bytes) snapshot=443097088
		Virtual memory (bytes) snapshot=4255436800
		Total committed heap usage (bytes)=293601280
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=2637
	File Output Format Counters 
		Bytes Written=2298
[hadoop@much hadoop-2.8.1]$ echo $?
0
[hadoop@much hadoop-2.8.1]$ ls
21centry  4300-0.txt  bin  etc  include  input  lib  libexec  LICENSE.txt  logs  NOTICE.txt  README.txt  sbin  share
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /x_put
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2017-08-08 01:30 /x_put/_SUCCESS
-rw-r--r--   1 hadoop supergroup       2298 2017-08-08 01:30 /x_put/part-r-00000
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -get /x_put/part-r-00000 . 
[hadoop@much hadoop-2.8.1]$ ls
21centry    bin  include  lib      LICENSE.txt  NOTICE.txt    README.txt  share
4300-0.txt  etc  input    libexec  logs         part-r-00000  sbin
[hadoop@much hadoop-2.8.1]$ head part-r-00000 
A	1
But	1
But,	1
Buy	1
Confine	1
Consider	1
Eiffel	1
Every	2
For	2
Grouping	1
[hadoop@much hadoop-2.8.1]$ 
~~~

可以如预期正常工作了

下面的日志中说明了，首先进行 map 操作，完成后再进行 reduce 操作

~~~
17/08/08 01:30:18 INFO mapreduce.Job:  map 0% reduce 0%
17/08/08 01:30:22 INFO mapreduce.Job:  map 100% reduce 0%
17/08/08 01:30:28 INFO mapreduce.Job:  map 100% reduce 100%
~~~


---

# 命令汇总

* **`sbin/start-dfs.sh`**
* **`bin/hdfs dfs -ls /abc`**
* **`bin/hdfs dfs -ls /`**
* **`jps`**
* **`cp etc/hadoop/mapred-site.xml.template  etc/hadoop/mapred-site.xml`**
* **`cat etc/hadoop/mapred-site.xml`**
* **`cat  etc/hadoop/yarn-site.xml`**
* **`sbin/start-yarn.sh`**
* **`bin/hdfs dfs -ls /abc`**
* **`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar  wordcount /abc/21centry  /x_put`**
* **`bin/hdfs dfs -ls /x_put`**
* **`bin/hdfs dfs -get /x_put/part-r-00000 .`**
* **`head part-r-00000`**



---

[hadoop]:http://hadoop.apache.org/
[singlecluster]:https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
[hadoop_doc]:https://wiki.apache.org/hadoop
[install_hadoop_on_ubuntu]:http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/
[hadoop_01_standalone]:/2017/08/07/hadoop-01-standalone/
[hadoop_02__hdfs]:/2017/08/07/hadoop-02-pseudo-distributed-operation-hdfs/
