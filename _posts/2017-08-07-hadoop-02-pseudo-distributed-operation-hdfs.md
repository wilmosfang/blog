---
layout:  post
title:  Hadoop 在 Centos7 下的单机布署(二).HDFS.Pseudo-Distributed Operation
author:  wilmosfang
tags:  hadoop hdfs
categories:  hadoop 
wc: 1947 12556 150753
excerpt:  Centos7 下布署 Pseudo-Distributed 模式的 HDFS
comments: true
---


# 前言

随着人工智能在各领域的大面积普及，数据分析在各个层面的广泛应用

其底层的技术在圈子里越来越受到重视和热捧

**[Apache Hadoop][hadoop]** 技术栈或者说技术生态圈绝对是一个不容忽视的中坚力量

**[Apache Hadoop][hadoop]** 是一个专注于可靠，弹性，分布式计算框架的开源软件项目

>The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

主要包含以下几个模块：

* **Hadoop Common** : 支持其它 **hadoop** 模块的通用工具
* **Hadoop Distributed File System** : 简称 **HDFS** , 给应用数据提供高吞吐性能的分布式文件系统
* **Hadoop YARN** : 工作调度与集群资源管理的框架
* **Hadoop MapReduce** : 大数据集的并行处理系统

Hadoop 生态圈中的其它项目可以参考 **[Hadoop-related projects][hadoop]**

> **Tip:**  当前的最新稳定版为 **Hadoop Release 2.8.1** 发布于 **08 June, 2017**

前面根据官方的文档给出最新版 Hadoop  在 Centos7 下的单机布署方案，详细可以参考 **[Hadoop 在 Centos7 下的单机布署(一). Standalone Operation][hadoop_01_standalone]**

这里基于前面的操作，给出最新版 Hadoop  在 Centos7 下 HDFS 的伪分布模式(单机)的布署方案

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

* 构建一个 hadoop 的单点伪集群
* hdfs 的基本操作

---

## 单机测试　

参考 **[Hadoop 在 Centos7 下的单机布署(一). Standalone Operation][hadoop_01_standalone]** 中的操作构建出脱机版的 Hadoop

我们在本地尝试执行一个 wordcount 的脱机任务

~~~
[hadoop@much hadoop-2.8.1]$ rm -rf output/
[hadoop@much hadoop-2.8.1]$ ls
21centry  etc      input  libexec      NOTICE.txt  sbin
bin       include  lib    LICENSE.txt  README.txt  share
[hadoop@much hadoop-2.8.1]$ cat 21centry 
In the middle of a vacation, you’re often having so much fun you want a memento to remember it. But that cute market trinket can quickly turn into a dust magnet when you get home. Here are some ideas that put the “fun” in functional.

Grouping similar items

Consider collections- This has the potential to get out of control, but if you travel a limited amount, think about one item that you could buy in every location. It might be a small basket, a serving bowl, or a textile. Over time, you can build up your collection slowly and it will show off your personality as you build it.

Confine trinkets to one space- If you stick to a theme when it comes to choosing your souvenirs, try to be thoughtful about how you place them. For example, if you decide you love masks, put them all on one wall as a group of artwork. If you have one mask in each room, it will just look like someone is always watching you! On the other hand, if you collect blankets or throws, you may want to strategically place a different one in each room where you might want to curl up.

Know when to stop- The goal here is to not turn your space into one of those freakish sideshows you see on the internet where every inch of space is covered. Every few trips, reassess your needs and whether one more of the same item could be overkill. When a collection is complete, appreciate it and move on to something else you’ve been eyeing on your travels.

A selective approach

Select wisely- T-shirts, shot glasses, magnets, and snow globes can be a default purchase, but are rarely appreciated once they’re out of a suitcase. Look for more unique items that may not scream tourist, but instead remind you of a city. For example, you may not want a keychain with the Eiffel Tower on it in Paris, but a unique watering can in the window of a gardening store brings a smile to your face. Buy it! Every time you use it you’ll be reminded of your trip, but the item won’t be cluttering your home.

Keep your colour scheme in mind- Sometimes when you go on vacation, especially to a tropical destination, you’ll fall in love a vibrant blue or orange. But, when you get home, you wonder what was wrong with your eyes as the hue is way too much for your décor. Keep that in mind as you shop and perhaps select a more subdued tone unless you are completely confident that bold is the look you’re after.

Some of the most interesting homes have carefully chosen items that incorporate a story into the overall décor. Your vacations make up the story of your life and the items you bring home can help create a unique look, as interesting as your life.
[hadoop@much hadoop-2.8.1]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar    wordcount  ./21centry   ./test_output
17/08/07 12:05:22 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
17/08/07 12:05:22 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
17/08/07 12:05:28 INFO input.FileInputFormat: Total input files to process : 1
17/08/07 12:05:28 INFO mapreduce.JobSubmitter: number of splits:1
17/08/07 12:05:28 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local279813974_0001
17/08/07 12:05:28 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
17/08/07 12:05:28 INFO mapreduce.Job: Running job: job_local279813974_0001
17/08/07 12:05:28 INFO mapred.LocalJobRunner: OutputCommitter set in config null
17/08/07 12:05:28 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 12:05:28 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 12:05:28 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
17/08/07 12:05:28 INFO mapred.LocalJobRunner: Waiting for map tasks
17/08/07 12:05:28 INFO mapred.LocalJobRunner: Starting task: attempt_local279813974_0001_m_000000_0
17/08/07 12:05:28 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 12:05:28 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 12:05:28 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 12:05:28 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/21centry:0+2637
17/08/07 12:05:28 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 12:05:28 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 12:05:28 INFO mapred.MapTask: soft limit at 83886080
17/08/07 12:05:28 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 12:05:28 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 12:05:28 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 12:05:28 INFO mapred.LocalJobRunner: 
17/08/07 12:05:28 INFO mapred.MapTask: Starting flush of map output
17/08/07 12:05:28 INFO mapred.MapTask: Spilling map output
17/08/07 12:05:28 INFO mapred.MapTask: bufstart = 0; bufend = 4577; bufvoid = 104857600
17/08/07 12:05:28 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26212452(104849808); length = 1945/6553600
17/08/07 12:05:28 INFO mapred.MapTask: Finished spill 0
17/08/07 12:05:28 INFO mapred.Task: Task:attempt_local279813974_0001_m_000000_0 is done. And is in the process of committing
17/08/07 12:05:28 INFO mapred.LocalJobRunner: map
17/08/07 12:05:28 INFO mapred.Task: Task 'attempt_local279813974_0001_m_000000_0' done.
17/08/07 12:05:28 INFO mapred.LocalJobRunner: Finishing task: attempt_local279813974_0001_m_000000_0
17/08/07 12:05:28 INFO mapred.LocalJobRunner: map task executor complete.
17/08/07 12:05:28 INFO mapred.LocalJobRunner: Waiting for reduce tasks
17/08/07 12:05:28 INFO mapred.LocalJobRunner: Starting task: attempt_local279813974_0001_r_000000_0
17/08/07 12:05:28 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 12:05:28 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 12:05:28 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 12:05:28 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@76f76d2b
17/08/07 12:05:28 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=334338464, maxSingleShuffleLimit=83584616, mergeThreshold=220663392, ioSortFactor=10, memToMemMergeOutputsThreshold=10
17/08/07 12:05:28 INFO reduce.EventFetcher: attempt_local279813974_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
17/08/07 12:05:28 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local279813974_0001_m_000000_0 decomp: 3374 len: 3378 to MEMORY
17/08/07 12:05:28 INFO reduce.InMemoryMapOutput: Read 3374 bytes from map-output for attempt_local279813974_0001_m_000000_0
17/08/07 12:05:28 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 3374, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->3374
17/08/07 12:05:28 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
17/08/07 12:05:28 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 12:05:28 INFO reduce.MergeManagerImpl: finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
17/08/07 12:05:28 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 12:05:28 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 3370 bytes
17/08/07 12:05:28 INFO reduce.MergeManagerImpl: Merged 1 segments, 3374 bytes to disk to satisfy reduce memory limit
17/08/07 12:05:28 INFO reduce.MergeManagerImpl: Merging 1 files, 3378 bytes from disk
17/08/07 12:05:28 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
17/08/07 12:05:28 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 12:05:28 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 3370 bytes
17/08/07 12:05:28 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 12:05:28 INFO Configuration.deprecation: mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
17/08/07 12:05:28 INFO mapred.Task: Task:attempt_local279813974_0001_r_000000_0 is done. And is in the process of committing
17/08/07 12:05:28 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 12:05:28 INFO mapred.Task: Task attempt_local279813974_0001_r_000000_0 is allowed to commit now
17/08/07 12:05:28 INFO output.FileOutputCommitter: Saved output of task 'attempt_local279813974_0001_r_000000_0' to file:/home/hadoop/hadoop-2.8.1/test_output/_temporary/0/task_local279813974_0001_r_000000
17/08/07 12:05:28 INFO mapred.LocalJobRunner: reduce > reduce
17/08/07 12:05:28 INFO mapred.Task: Task 'attempt_local279813974_0001_r_000000_0' done.
17/08/07 12:05:28 INFO mapred.LocalJobRunner: Finishing task: attempt_local279813974_0001_r_000000_0
17/08/07 12:05:28 INFO mapred.LocalJobRunner: reduce task executor complete.
17/08/07 12:05:29 INFO mapreduce.Job: Job job_local279813974_0001 running in uber mode : false
17/08/07 12:05:29 INFO mapreduce.Job:  map 100% reduce 100%
17/08/07 12:05:29 INFO mapreduce.Job: Job job_local279813974_0001 completed successfully
17/08/07 12:05:29 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=616258
		FILE: Number of bytes written=1259654
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=17
		Map output records=487
		Map output bytes=4577
		Map output materialized bytes=3378
		Input split bytes=104
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
		GC time elapsed (ms)=0
		Total committed heap usage (bytes)=440401920
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
		Bytes Written=2326
[hadoop@much hadoop-2.8.1]$ echo $?
0
[hadoop@much hadoop-2.8.1]$ ll test_output/
total 4
-rw-r--r--. 1 hadoop hadoop 2298 Aug  7 12:05 part-r-00000
-rw-r--r--. 1 hadoop hadoop    0 Aug  7 12:05 _SUCCESS
[hadoop@much hadoop-2.8.1]$ head -n 20 test_output/part-r-00000 
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
Here	1
If	2
In	1
It	1
Keep	2
Know	1
Look	1
On	1
Over	1
Paris,	1
[hadoop@much hadoop-2.8.1]$ 
~~~

> **Note:**  **test_output** 目录是在执行过程中被 Hadoop 自动创建的，不用事先自己创建，如果事先自己创建了，会出错，解决办法是在一个有写权限的位置指定一个未被创建的子目录

---

## 创建数据存放目录

~~~
[hadoop@much hadoop-2.8.1]$ ls
21centry    bin  include  lib      LICENSE.txt  README.txt  share
4300-0.txt  etc  input    libexec  NOTICE.txt   sbin
[hadoop@much hadoop-2.8.1]$ mkdir hdfs
[hadoop@much hadoop-2.8.1]$ ll hdfs/
total 0
[hadoop@much hadoop-2.8.1]$ ll hdfs/ -d 
drwxrwxr-x. 2 hadoop hadoop 6 Aug  7 12:19 hdfs/
[hadoop@much hadoop-2.8.1]$ cd hdfs/
[hadoop@much hdfs]$ pwd
/home/hadoop/hadoop-2.8.1/hdfs
[hadoop@much hdfs]$ 
~~~

---

## 测试证书登录

~~~
[hadoop@much hadoop-2.8.1]$ ssh localhost
Last login: Mon Aug  7 11:03:56 2017
[hadoop@much ~]$ exit
logout
Connection to localhost closed.
[hadoop@much hadoop-2.8.1]$ ssh localhost
Last login: Mon Aug  7 13:35:02 2017 from localhost
[hadoop@much ~]$ exit
logout
Connection to localhost closed.
[hadoop@much hadoop-2.8.1]$
~~~

---

## HDFS 的架构

HDFS 的架构可以参考 **[Overview of the HDFS Architecture][hdfs_arch]**

这是 Application 与 NameNode 还有 DataNode 之间的交互架构

![hdfs_runtime.jpg](/images/hadoop/hdfs_runtime.jpg)

这是 DataNodes 之间的复制架构

![hdfs_blockreplication.gif](/images/hadoop/hdfs_blockreplication.gif)


---

## HDFS 的配置


修改 **`etc/hadoop/core-site.xml`**

~~~
[hadoop@much hadoop-2.8.1]$ vim  etc/hadoop/core-site.xml 
[hadoop@much hadoop-2.8.1]$ cat etc/hadoop/core-site.xml 
<?xml version="1.0" encoding="UTF-8"?>
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
        <name>hadoop.tmp.dir</name>  
        <value>file:/home/hadoop/hadoop-2.8.1/hdfs</value>  
    </property>  
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
[hadoop@much hadoop-2.8.1]$
~~~

修改 **`etc/hadoop/hdfs-site.xml`**

~~~
[hadoop@much hadoop-2.8.1]$ vim  etc/hadoop/hdfs-site.xml 
[hadoop@much hadoop-2.8.1]$ cat etc/hadoop/hdfs-site.xml 
<?xml version="1.0" encoding="UTF-8"?>
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
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>  
        <name>dfs.namenode.name.dir</name>  
        <value>file:/home/hadoop/hadoop-2.8.1/hdfs/dfs/name</value>  
    </property>  
    <property>  
        <name>dfs.datanode.data.dir</name>  
        <value>file:/home/hadoop/hadoop-2.8.1/hdfs/dfs/data</value>  
    </property>
</configuration>
[hadoop@much hadoop-2.8.1]$ 
~~~


---

## 格式化 hdfs

~~~
[hadoop@much hadoop-2.8.1]$ ll hdfs/
total 0
[hadoop@much hadoop-2.8.1]$ bin/hdfs namenode -format
17/08/07 14:04:21 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   user = hadoop
STARTUP_MSG:   host = much/10.0.2.15
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.8.1
STARTUP_MSG:   classpath = /home/hadoop/hadoop-2.8.1/etc/hadoop:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jsp-api-2.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jersey-core-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jersey-json-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jettison-1.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/stax-api-1.0-2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/activation-1.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jackson-core-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jackson-mapper-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jackson-jaxrs-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jackson-xc-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jersey-server-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/asm-3.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/log4j-1.2.17.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jets3t-0.9.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-lang-2.6.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-configuration-1.6.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-digester-1.8.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-beanutils-1.7.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/slf4j-api-1.7.10.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/avro-1.7.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/paranamer-2.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-compress-1.4.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/xz-1.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/gson-2.2.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/hadoop-auth-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/nimbus-jose-jwt-3.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jcip-annotations-1.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/json-smart-1.1.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/zookeeper-3.4.6.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/netty-3.6.2.Final.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/curator-framework-2.7.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/curator-client-2.7.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jsch-0.1.51.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/htrace-core4-4.0.1-incubating.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/junit-4.11.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/hamcrest-core-1.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/mockito-all-1.8.5.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/hadoop-annotations-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/guava-11.0.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jsr305-3.0.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-cli-1.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-math3-3.1.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/xmlenc-0.52.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/httpclient-4.5.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/httpcore-4.4.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-logging-1.1.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-codec-1.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-io-2.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-net-3.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/commons-collections-3.2.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/servlet-api-2.5.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jetty-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jetty-util-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/lib/jetty-sslengine-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/hadoop-common-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/hadoop-common-2.8.1-tests.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/common/hadoop-nfs-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/commons-io-2.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/netty-3.6.2.Final.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/guava-11.0.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jetty-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jetty-util-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jackson-core-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jackson-mapper-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/asm-3.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/htrace-core4-4.0.1-incubating.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/hadoop-hdfs-client-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/okhttp-2.4.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/okio-1.4.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/netty-all-4.0.23.Final.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-2.8.1-tests.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-client-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-client-2.8.1-tests.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-native-client-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-native-client-2.8.1-tests.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/hdfs/hadoop-hdfs-nfs-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/fst-2.24.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/javassist-3.18.1-GA.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/objenesis-2.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jetty-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/curator-client-2.7.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/curator-test-2.7.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-math-2.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/zookeeper-3.4.6-tests.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-lang-2.6.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/guava-11.0.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-cli-1.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/log4j-1.2.17.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/activation-1.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/xz-1.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/servlet-api-2.5.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-codec-1.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jetty-util-6.1.26.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jersey-core-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jersey-client-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jackson-core-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jackson-mapper-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jackson-jaxrs-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jackson-xc-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/guice-3.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/javax.inject-1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/aopalliance-1.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/commons-io-2.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jersey-server-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/asm-3.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jersey-json-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jettison-1.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/zookeeper-3.4.6.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/lib/netty-3.6.2.Final.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-api-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-common-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-common-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-tests-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-client-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-sharedcachemanager-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-server-timeline-pluginstorage-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/yarn/hadoop-yarn-registry-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/avro-1.7.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/jackson-core-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.9.13.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/xz-1.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/hadoop-annotations-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/asm-3.2.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/netty-3.6.2.Final.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/guice-3.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/javax.inject-1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/junit-4.11.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar:/home/hadoop/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.8.1-tests.jar:/contrib/capacity-scheduler/*.jar
STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hadoop.git -r 20fe5304904fc2f5a18053c389e43cd26f7a70fe; compiled by 'vinodkv' on 2017-06-02T06:14Z
STARTUP_MSG:   java = 1.8.0_141
************************************************************/
17/08/07 14:04:21 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
17/08/07 14:04:21 INFO namenode.NameNode: createNameNode [-format]
Formatting using clusterid: CID-dabe1865-130e-430c-99e4-d54608b2dad3
17/08/07 14:04:22 INFO namenode.FSEditLog: Edit logging is async:false
17/08/07 14:04:22 INFO namenode.FSNamesystem: KeyProvider: null
17/08/07 14:04:22 INFO namenode.FSNamesystem: fsLock is fair: true
17/08/07 14:04:22 INFO namenode.FSNamesystem: Detailed lock hold time metrics enabled: false
17/08/07 14:04:22 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
17/08/07 14:04:22 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
17/08/07 14:04:22 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
17/08/07 14:04:22 INFO blockmanagement.BlockManager: The block deletion will start around 2017 Aug 07 14:04:22
17/08/07 14:04:22 INFO util.GSet: Computing capacity for map BlocksMap
17/08/07 14:04:22 INFO util.GSet: VM type       = 64-bit
17/08/07 14:04:22 INFO util.GSet: 2.0% max memory 889 MB = 17.8 MB
17/08/07 14:04:22 INFO util.GSet: capacity      = 2^21 = 2097152 entries
17/08/07 14:04:22 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
17/08/07 14:04:22 INFO blockmanagement.BlockManager: defaultReplication         = 1
17/08/07 14:04:22 INFO blockmanagement.BlockManager: maxReplication             = 512
17/08/07 14:04:22 INFO blockmanagement.BlockManager: minReplication             = 1
17/08/07 14:04:22 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
17/08/07 14:04:22 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
17/08/07 14:04:22 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
17/08/07 14:04:22 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
17/08/07 14:04:22 INFO namenode.FSNamesystem: fsOwner             = hadoop (auth:SIMPLE)
17/08/07 14:04:22 INFO namenode.FSNamesystem: supergroup          = supergroup
17/08/07 14:04:22 INFO namenode.FSNamesystem: isPermissionEnabled = true
17/08/07 14:04:22 INFO namenode.FSNamesystem: HA Enabled: false
17/08/07 14:04:22 INFO namenode.FSNamesystem: Append Enabled: true
17/08/07 14:04:22 INFO util.GSet: Computing capacity for map INodeMap
17/08/07 14:04:22 INFO util.GSet: VM type       = 64-bit
17/08/07 14:04:22 INFO util.GSet: 1.0% max memory 889 MB = 8.9 MB
17/08/07 14:04:22 INFO util.GSet: capacity      = 2^20 = 1048576 entries
17/08/07 14:04:22 INFO namenode.FSDirectory: ACLs enabled? false
17/08/07 14:04:22 INFO namenode.FSDirectory: XAttrs enabled? true
17/08/07 14:04:22 INFO namenode.NameNode: Caching file names occurring more than 10 times
17/08/07 14:04:22 INFO util.GSet: Computing capacity for map cachedBlocks
17/08/07 14:04:22 INFO util.GSet: VM type       = 64-bit
17/08/07 14:04:22 INFO util.GSet: 0.25% max memory 889 MB = 2.2 MB
17/08/07 14:04:22 INFO util.GSet: capacity      = 2^18 = 262144 entries
17/08/07 14:04:22 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
17/08/07 14:04:22 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
17/08/07 14:04:22 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
17/08/07 14:04:22 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
17/08/07 14:04:22 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
17/08/07 14:04:22 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
17/08/07 14:04:22 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
17/08/07 14:04:22 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
17/08/07 14:04:22 INFO util.GSet: Computing capacity for map NameNodeRetryCache
17/08/07 14:04:22 INFO util.GSet: VM type       = 64-bit
17/08/07 14:04:22 INFO util.GSet: 0.029999999329447746% max memory 889 MB = 273.1 KB
17/08/07 14:04:22 INFO util.GSet: capacity      = 2^15 = 32768 entries
17/08/07 14:04:22 INFO namenode.FSImage: Allocated new BlockPoolId: BP-788595051-10.0.2.15-1502085862480
17/08/07 14:04:22 INFO common.Storage: Storage directory /home/hadoop/hadoop-2.8.1/hdfs/dfs/name has been successfully formatted.
17/08/07 14:04:22 INFO namenode.FSImageFormatProtobuf: Saving image file /home/hadoop/hadoop-2.8.1/hdfs/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
17/08/07 14:04:22 INFO namenode.FSImageFormatProtobuf: Image file /home/hadoop/hadoop-2.8.1/hdfs/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 323 bytes saved in 0 seconds.
17/08/07 14:04:22 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
17/08/07 14:04:22 INFO util.ExitUtil: Exiting with status 0
17/08/07 14:04:22 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at much/10.0.2.15
************************************************************/
[hadoop@much hadoop-2.8.1]$ echo $?
0
[hadoop@much hadoop-2.8.1]$ 
~~~

格式化完成后可以看到新创建的文件夹 hdfs 里多出来了东西　

~~~
[hadoop@much hadoop-2.8.1]$ ll hdfs/
total 0
drwxrwxr-x. 3 hadoop hadoop 18 Aug  7 14:04 dfs
[hadoop@much hadoop-2.8.1]$ tree hdfs/
hdfs/
└── dfs
    └── name
        └── current
            ├── fsimage_0000000000000000000
            ├── fsimage_0000000000000000000.md5
            ├── seen_txid
            └── VERSION

3 directories, 4 files
[hadoop@much hadoop-2.8.1]$ 
~~~

---

## 启动 NameNode 与 DataNode 守护进程

~~~
[hadoop@much hadoop-2.8.1]$ jps
8070 Jps
[hadoop@much hadoop-2.8.1]$ sbin/start-dfs.sh 
Starting namenodes on [localhost]
localhost: starting namenode, logging to /home/hadoop/hadoop-2.8.1/logs/hadoop-hadoop-namenode-much.out
localhost: starting datanode, logging to /home/hadoop/hadoop-2.8.1/logs/hadoop-hadoop-datanode-much.out
Starting secondary namenodes [0.0.0.0]
The authenticity of host '0.0.0.0 (0.0.0.0)' can't be established.
ECDSA key fingerprint is 0c:52:20:0a:00:e3:1a:5d:c6:fc:79:b3:e8:6e:d6:f1.
Are you sure you want to continue connecting (yes/no)? yes
0.0.0.0: Warning: Permanently added '0.0.0.0' (ECDSA) to the list of known hosts.
0.0.0.0: starting secondarynamenode, logging to /home/hadoop/hadoop-2.8.1/logs/hadoop-hadoop-secondarynamenode-much.out
[hadoop@much hadoop-2.8.1]$ jps
7364 DataNode
7270 NameNode
7576 SecondaryNameNode
7693 Jps
[hadoop@much hadoop-2.8.1]$
~~~

---

## web 访问 NameNode

打开放火墙的 50070 端口

~~~
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 8080/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@much ~]# firewall-cmd --add-port 50070/tcp
success
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 8080/tcp 50070/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@much ~]# 
~~~

访问 **`http://192.168.56.207:50070`**

DataNode 的概要

![hadoop_hdfs_01.png](/images/hadoop/hadoop_hdfs_01.png)

DataNode 的信息

![hadoop_hdfs_02.png](/images/hadoop/hadoop_hdfs_02.png)

DataNode 的日志信息

![hadoop_hdfs_03.png](/images/hadoop/hadoop_hdfs_03.png)


---

## hdfs 的常用命令

~~~
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -mkdir /user
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:34 /user
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -mkdir /user/hduser
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /user
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:35 /user/hduser
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -mkdir /input
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:35 /input
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:35 /user
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -put etc/hadoop/  input
put: `input': No such file or directory: `hdfs://localhost:9000/user/hadoop/input'
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -put etc/hadoop/  /input
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /input
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:37 /input/hadoop
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /input/hadoop
Found 29 items
-rw-r--r--   1 hadoop supergroup       4942 2017-08-07 14:37 /input/hadoop/capacity-scheduler.xml
-rw-r--r--   1 hadoop supergroup       1335 2017-08-07 14:37 /input/hadoop/configuration.xsl
-rw-r--r--   1 hadoop supergroup        318 2017-08-07 14:37 /input/hadoop/container-executor.cfg
-rw-r--r--   1 hadoop supergroup       1018 2017-08-07 14:37 /input/hadoop/core-site.xml
-rw-r--r--   1 hadoop supergroup       3719 2017-08-07 14:37 /input/hadoop/hadoop-env.cmd
-rw-r--r--   1 hadoop supergroup       4745 2017-08-07 14:37 /input/hadoop/hadoop-env.sh
-rw-r--r--   1 hadoop supergroup       2490 2017-08-07 14:37 /input/hadoop/hadoop-metrics.properties
-rw-r--r--   1 hadoop supergroup       2598 2017-08-07 14:37 /input/hadoop/hadoop-metrics2.properties
-rw-r--r--   1 hadoop supergroup       9683 2017-08-07 14:37 /input/hadoop/hadoop-policy.xml
-rw-r--r--   1 hadoop supergroup       1165 2017-08-07 14:37 /input/hadoop/hdfs-site.xml
-rw-r--r--   1 hadoop supergroup       1449 2017-08-07 14:37 /input/hadoop/httpfs-env.sh
-rw-r--r--   1 hadoop supergroup       1657 2017-08-07 14:37 /input/hadoop/httpfs-log4j.properties
-rw-r--r--   1 hadoop supergroup         21 2017-08-07 14:37 /input/hadoop/httpfs-signature.secret
-rw-r--r--   1 hadoop supergroup        620 2017-08-07 14:37 /input/hadoop/httpfs-site.xml
-rw-r--r--   1 hadoop supergroup       3518 2017-08-07 14:37 /input/hadoop/kms-acls.xml
-rw-r--r--   1 hadoop supergroup       1611 2017-08-07 14:37 /input/hadoop/kms-env.sh
-rw-r--r--   1 hadoop supergroup       1631 2017-08-07 14:37 /input/hadoop/kms-log4j.properties
-rw-r--r--   1 hadoop supergroup       5546 2017-08-07 14:37 /input/hadoop/kms-site.xml
-rw-r--r--   1 hadoop supergroup      13661 2017-08-07 14:37 /input/hadoop/log4j.properties
-rw-r--r--   1 hadoop supergroup        931 2017-08-07 14:37 /input/hadoop/mapred-env.cmd
-rw-r--r--   1 hadoop supergroup       1383 2017-08-07 14:37 /input/hadoop/mapred-env.sh
-rw-r--r--   1 hadoop supergroup       4113 2017-08-07 14:37 /input/hadoop/mapred-queues.xml.template
-rw-r--r--   1 hadoop supergroup        758 2017-08-07 14:37 /input/hadoop/mapred-site.xml.template
-rw-r--r--   1 hadoop supergroup         10 2017-08-07 14:37 /input/hadoop/slaves
-rw-r--r--   1 hadoop supergroup       2316 2017-08-07 14:37 /input/hadoop/ssl-client.xml.example
-rw-r--r--   1 hadoop supergroup       2697 2017-08-07 14:37 /input/hadoop/ssl-server.xml.example
-rw-r--r--   1 hadoop supergroup       2191 2017-08-07 14:37 /input/hadoop/yarn-env.cmd
-rw-r--r--   1 hadoop supergroup       4567 2017-08-07 14:37 /input/hadoop/yarn-env.sh
-rw-r--r--   1 hadoop supergroup        690 2017-08-07 14:37 /input/hadoop/yarn-site.xml
[hadoop@much hadoop-2.8.1]$
~~~

关于 hdfs 的文件系统操作命令可以参考 **[File System Shell Guide][filesystemshell]**

这时再看 hdfs 目录里的内容

~~~
[hadoop@much hadoop-2.8.1]$ tree hdfs/
hdfs/
└── dfs
    ├── data
    │   ├── current
    │   │   ├── BP-788595051-10.0.2.15-1502085862480
    │   │   │   ├── current
    │   │   │   │   ├── dfsUsed
    │   │   │   │   ├── finalized
    │   │   │   │   │   └── subdir0
    │   │   │   │   │       └── subdir0
    │   │   │   │   │           ├── blk_1073741825
    │   │   │   │   │           ├── blk_1073741825_1001.meta
    │   │   │   │   │           ├── blk_1073741826
    │   │   │   │   │           ├── blk_1073741826_1002.meta
    │   │   │   │   │           ├── blk_1073741827
    │   │   │   │   │           ├── blk_1073741827_1003.meta
    │   │   │   │   │           ├── blk_1073741828
    │   │   │   │   │           ├── blk_1073741828_1004.meta
    │   │   │   │   │           ├── blk_1073741829
    │   │   │   │   │           ├── blk_1073741829_1005.meta
    │   │   │   │   │           ├── blk_1073741830
    │   │   │   │   │           ├── blk_1073741830_1006.meta
    │   │   │   │   │           ├── blk_1073741831
    │   │   │   │   │           ├── blk_1073741831_1007.meta
    │   │   │   │   │           ├── blk_1073741832
    │   │   │   │   │           ├── blk_1073741832_1008.meta
    │   │   │   │   │           ├── blk_1073741833
    │   │   │   │   │           ├── blk_1073741833_1009.meta
    │   │   │   │   │           ├── blk_1073741834
    │   │   │   │   │           ├── blk_1073741834_1010.meta
    │   │   │   │   │           ├── blk_1073741835
    │   │   │   │   │           ├── blk_1073741835_1011.meta
    │   │   │   │   │           ├── blk_1073741836
    │   │   │   │   │           ├── blk_1073741836_1012.meta
    │   │   │   │   │           ├── blk_1073741837
    │   │   │   │   │           ├── blk_1073741837_1013.meta
    │   │   │   │   │           ├── blk_1073741838
    │   │   │   │   │           ├── blk_1073741838_1014.meta
    │   │   │   │   │           ├── blk_1073741839
    │   │   │   │   │           ├── blk_1073741839_1015.meta
    │   │   │   │   │           ├── blk_1073741840
    │   │   │   │   │           ├── blk_1073741840_1016.meta
    │   │   │   │   │           ├── blk_1073741841
    │   │   │   │   │           ├── blk_1073741841_1017.meta
    │   │   │   │   │           ├── blk_1073741842
    │   │   │   │   │           ├── blk_1073741842_1018.meta
    │   │   │   │   │           ├── blk_1073741843
    │   │   │   │   │           ├── blk_1073741843_1019.meta
    │   │   │   │   │           ├── blk_1073741844
    │   │   │   │   │           ├── blk_1073741844_1020.meta
    │   │   │   │   │           ├── blk_1073741845
    │   │   │   │   │           ├── blk_1073741845_1021.meta
    │   │   │   │   │           ├── blk_1073741846
    │   │   │   │   │           ├── blk_1073741846_1022.meta
    │   │   │   │   │           ├── blk_1073741847
    │   │   │   │   │           ├── blk_1073741847_1023.meta
    │   │   │   │   │           ├── blk_1073741848
    │   │   │   │   │           ├── blk_1073741848_1024.meta
    │   │   │   │   │           ├── blk_1073741849
    │   │   │   │   │           ├── blk_1073741849_1025.meta
    │   │   │   │   │           ├── blk_1073741850
    │   │   │   │   │           ├── blk_1073741850_1026.meta
    │   │   │   │   │           ├── blk_1073741851
    │   │   │   │   │           ├── blk_1073741851_1027.meta
    │   │   │   │   │           ├── blk_1073741852
    │   │   │   │   │           ├── blk_1073741852_1028.meta
    │   │   │   │   │           ├── blk_1073741853
    │   │   │   │   │           └── blk_1073741853_1029.meta
    │   │   │   │   ├── rbw
    │   │   │   │   └── VERSION
    │   │   │   ├── scanner.cursor
    │   │   │   └── tmp
    │   │   └── VERSION
    │   └── in_use.lock
    ├── name
    │   ├── current
    │   │   ├── edits_0000000000000000001-0000000000000000001
    │   │   ├── edits_0000000000000000002-0000000000000000003
    │   │   ├── edits_inprogress_0000000000000000004
    │   │   ├── fsimage_0000000000000000000
    │   │   ├── fsimage_0000000000000000000.md5
    │   │   ├── fsimage_0000000000000000003
    │   │   ├── fsimage_0000000000000000003.md5
    │   │   ├── seen_txid
    │   │   └── VERSION
    │   └── in_use.lock
    └── namesecondary
        ├── current
        │   ├── edits_0000000000000000001-0000000000000000001
        │   ├── edits_0000000000000000002-0000000000000000003
        │   ├── fsimage_0000000000000000000
        │   ├── fsimage_0000000000000000000.md5
        │   ├── fsimage_0000000000000000003
        │   ├── fsimage_0000000000000000003.md5
        │   └── VERSION
        └── in_use.lock

14 directories, 81 files
[hadoop@much hadoop-2.8.1]$
~~~

---

## 测试 hdfs 的读写


~~~
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /input
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:37 /input/hadoop
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:37 /input
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:44 /user
[hadoop@much hadoop-2.8.1]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar  grep  /input/hadoop  /output 'dfs[a-z.]+'
17/08/07 14:47:25 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
17/08/07 14:47:25 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
17/08/07 14:47:25 INFO input.FileInputFormat: Total input files to process : 29
17/08/07 14:47:25 INFO mapreduce.JobSubmitter: number of splits:29
17/08/07 14:47:25 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local223365185_0001
17/08/07 14:47:25 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
17/08/07 14:47:25 INFO mapreduce.Job: Running job: job_local223365185_0001
17/08/07 14:47:25 INFO mapred.LocalJobRunner: OutputCommitter set in config null
17/08/07 14:47:25 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:25 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:25 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
17/08/07 14:47:25 INFO mapred.LocalJobRunner: Waiting for map tasks
17/08/07 14:47:25 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000000_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/log4j.properties:0+13661
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufend = 352; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214348(104857392); length = 49/6553600
17/08/07 14:47:26 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000000_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000000_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000000_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000001_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/hadoop-policy.xml:0+9683
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufend = 17; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214396(104857584); length = 1/6553600
17/08/07 14:47:26 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000001_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000001_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000001_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000002_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/kms-site.xml:0+5546
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000002_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000002_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000002_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000003_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/capacity-scheduler.xml:0+4942
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000003_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000003_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000003_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000004_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/hadoop-env.sh:0+4745
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufend = 50; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214392(104857568); length = 5/6553600
17/08/07 14:47:26 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000004_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000004_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000004_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000005_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/yarn-env.sh:0+4567
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000005_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000005_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000005_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000006_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/mapred-queues.xml.template:0+4113
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000006_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000006_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000006_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000007_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/hadoop-env.cmd:0+3719
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufend = 50; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214392(104857568); length = 5/6553600
17/08/07 14:47:26 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000007_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000007_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000007_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000008_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/kms-acls.xml:0+3518
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000008_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000008_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000008_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000009_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/ssl-server.xml.example:0+2697
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000009_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000009_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000009_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000010_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/hadoop-metrics2.properties:0+2598
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000010_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000010_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000010_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000011_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/hadoop-metrics.properties:0+2490
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufend = 170; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214364(104857456); length = 33/6553600
17/08/07 14:47:26 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000011_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000011_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000011_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000012_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/ssl-client.xml.example:0+2316
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000012_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000012_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000012_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000013_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/yarn-env.cmd:0+2191
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000013_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000013_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000013_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000014_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/httpfs-log4j.properties:0+1657
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000014_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000014_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000014_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000015_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/kms-log4j.properties:0+1631
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000015_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000015_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000015_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000016_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/kms-env.sh:0+1611
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000016_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000016_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000016_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000017_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/httpfs-env.sh:0+1449
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000017_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000017_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000017_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000018_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/mapred-env.sh:0+1383
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000018_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000018_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000018_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000019_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/configuration.xsl:0+1335
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000019_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000019_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000019_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000020_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/hdfs-site.xml:0+1165
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufend = 84; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214388(104857552); length = 9/6553600
17/08/07 14:47:26 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000020_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000020_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000020_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000021_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/core-site.xml:0+1018
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000021_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000021_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000021_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000022_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/mapred-env.cmd:0+931
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000022_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000022_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000022_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000023_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/mapred-site.xml.template:0+758
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000023_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000023_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000023_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000024_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/yarn-site.xml:0+690
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000024_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000024_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000024_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000025_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/httpfs-site.xml:0+620
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000025_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000025_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000025_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000026_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/container-executor.cfg:0+318
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000026_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000026_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000026_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000027_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/httpfs-signature.secret:0+21
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapreduce.Job: Job job_local223365185_0001 running in uber mode : false
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapreduce.Job:  map 100% reduce 0%
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000027_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000027_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000027_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_m_000028_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/input/hadoop/slaves:0+10
17/08/07 14:47:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:26 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:26 INFO mapred.LocalJobRunner: 
17/08/07 14:47:26 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:26 INFO mapred.Task: Task:attempt_local223365185_0001_m_000028_0 is done. And is in the process of committing
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map
17/08/07 14:47:26 INFO mapred.Task: Task 'attempt_local223365185_0001_m_000028_0' done.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_m_000028_0
17/08/07 14:47:26 INFO mapred.LocalJobRunner: map task executor complete.
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Waiting for reduce tasks
17/08/07 14:47:26 INFO mapred.LocalJobRunner: Starting task: attempt_local223365185_0001_r_000000_0
17/08/07 14:47:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:26 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:26 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@1608f31e
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=372139616, maxSingleShuffleLimit=93034904, mergeThreshold=245612160, ioSortFactor=10, memToMemMergeOutputsThreshold=10
17/08/07 14:47:27 INFO reduce.EventFetcher: attempt_local223365185_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000001_0 decomp: 21 len: 25 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 21 bytes from map-output for attempt_local223365185_0001_m_000001_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 21, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->21
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000026_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000026_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 2, commitMemory -> 21, usedMemory ->23
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000000_0 decomp: 174 len: 178 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 174 bytes from map-output for attempt_local223365185_0001_m_000000_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 174, inMemoryMapOutputs.size() -> 3, commitMemory -> 23, usedMemory ->197
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000013_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000013_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 4, commitMemory -> 197, usedMemory ->199
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000028_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000028_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 5, commitMemory -> 199, usedMemory ->201
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000002_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000002_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 6, commitMemory -> 201, usedMemory ->203
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000015_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000015_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 7, commitMemory -> 203, usedMemory ->205
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000014_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000014_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 8, commitMemory -> 205, usedMemory ->207
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000027_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000027_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 9, commitMemory -> 207, usedMemory ->209
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000004_0 decomp: 29 len: 33 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 29 bytes from map-output for attempt_local223365185_0001_m_000004_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 29, inMemoryMapOutputs.size() -> 10, commitMemory -> 209, usedMemory ->238
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000017_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000017_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 11, commitMemory -> 238, usedMemory ->240
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000016_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000016_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 12, commitMemory -> 240, usedMemory ->242
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000003_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000003_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 13, commitMemory -> 242, usedMemory ->244
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000018_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000018_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 14, commitMemory -> 244, usedMemory ->246
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000005_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000005_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 15, commitMemory -> 246, usedMemory ->248
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000020_0 decomp: 92 len: 96 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 92 bytes from map-output for attempt_local223365185_0001_m_000020_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 92, inMemoryMapOutputs.size() -> 16, commitMemory -> 248, usedMemory ->340
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000007_0 decomp: 29 len: 33 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 29 bytes from map-output for attempt_local223365185_0001_m_000007_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 29, inMemoryMapOutputs.size() -> 17, commitMemory -> 340, usedMemory ->369
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000006_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000006_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 18, commitMemory -> 369, usedMemory ->371
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000019_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000019_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 19, commitMemory -> 371, usedMemory ->373
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000009_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000009_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 20, commitMemory -> 373, usedMemory ->375
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000008_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000008_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 21, commitMemory -> 375, usedMemory ->377
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000021_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000021_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 22, commitMemory -> 377, usedMemory ->379
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000010_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000010_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 23, commitMemory -> 379, usedMemory ->381
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000023_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000023_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 24, commitMemory -> 381, usedMemory ->383
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000022_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000022_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 25, commitMemory -> 383, usedMemory ->385
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000012_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000012_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 26, commitMemory -> 385, usedMemory ->387
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000025_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000025_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 27, commitMemory -> 387, usedMemory ->389
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000024_0 decomp: 2 len: 6 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local223365185_0001_m_000024_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 28, commitMemory -> 389, usedMemory ->391
17/08/07 14:47:27 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local223365185_0001_m_000011_0 decomp: 109 len: 113 to MEMORY
17/08/07 14:47:27 INFO reduce.InMemoryMapOutput: Read 109 bytes from map-output for attempt_local223365185_0001_m_000011_0
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 109, inMemoryMapOutputs.size() -> 29, commitMemory -> 391, usedMemory ->500
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:27 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
17/08/07 14:47:27 INFO mapred.LocalJobRunner: 29 / 29 copied.
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: finalMerge called with 29 in-memory map-outputs and 0 on-disk map-outputs
17/08/07 14:47:27 INFO mapred.Merger: Merging 29 sorted segments
17/08/07 14:47:27 INFO mapred.Merger: Down to the last merge-pass, with 6 segments left of total size: 338 bytes
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: Merged 29 segments, 500 bytes to disk to satisfy reduce memory limit
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: Merging 1 files, 448 bytes from disk
17/08/07 14:47:27 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
17/08/07 14:47:27 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 14:47:27 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 413 bytes
17/08/07 14:47:27 INFO mapred.LocalJobRunner: 29 / 29 copied.
17/08/07 14:47:27 INFO Configuration.deprecation: mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
17/08/07 14:47:27 INFO mapred.Task: Task:attempt_local223365185_0001_r_000000_0 is done. And is in the process of committing
17/08/07 14:47:27 INFO mapred.LocalJobRunner: 29 / 29 copied.
17/08/07 14:47:27 INFO mapred.Task: Task attempt_local223365185_0001_r_000000_0 is allowed to commit now
17/08/07 14:47:27 INFO output.FileOutputCommitter: Saved output of task 'attempt_local223365185_0001_r_000000_0' to hdfs://localhost:9000/user/hadoop/grep-temp-410115572/_temporary/0/task_local223365185_0001_r_000000
17/08/07 14:47:27 INFO mapred.LocalJobRunner: reduce > reduce
17/08/07 14:47:27 INFO mapred.Task: Task 'attempt_local223365185_0001_r_000000_0' done.
17/08/07 14:47:27 INFO mapred.LocalJobRunner: Finishing task: attempt_local223365185_0001_r_000000_0
17/08/07 14:47:27 INFO mapred.LocalJobRunner: reduce task executor complete.
17/08/07 14:47:27 INFO mapreduce.Job:  map 100% reduce 100%
17/08/07 14:47:27 INFO mapreduce.Job: Job job_local223365185_0001 completed successfully
17/08/07 14:47:27 INFO mapreduce.Job: Counters: 35
	File System Counters
		FILE: Number of bytes read=10242034
		FILE: Number of bytes written=18885357
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=1879201
		HDFS: Number of bytes written=564
		HDFS: Number of read operations=1021
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=32
	Map-Reduce Framework
		Map input records=2176
		Map output records=30
		Map output bytes=723
		Map output materialized bytes=616
		Input split bytes=3389
		Combine input records=30
		Combine output records=17
		Reduce input groups=15
		Reduce shuffle bytes=616
		Reduce input records=17
		Reduce output records=15
		Spilled Records=34
		Shuffled Maps =29
		Failed Shuffles=0
		Merged Map outputs=29
		GC time elapsed (ms)=98
		Total committed heap usage (bytes)=15390998528
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=81383
	File Output Format Counters 
		Bytes Written=564
17/08/07 14:47:27 INFO jvm.JvmMetrics: Cannot initialize JVM Metrics with processName=JobTracker, sessionId= - already initialized
17/08/07 14:47:28 INFO input.FileInputFormat: Total input files to process : 1
17/08/07 14:47:28 INFO mapreduce.JobSubmitter: number of splits:1
17/08/07 14:47:28 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local391964061_0002
17/08/07 14:47:28 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
17/08/07 14:47:28 INFO mapreduce.Job: Running job: job_local391964061_0002
17/08/07 14:47:28 INFO mapred.LocalJobRunner: OutputCommitter set in config null
17/08/07 14:47:28 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:28 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:28 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
17/08/07 14:47:28 INFO mapred.LocalJobRunner: Waiting for map tasks
17/08/07 14:47:28 INFO mapred.LocalJobRunner: Starting task: attempt_local391964061_0002_m_000000_0
17/08/07 14:47:28 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:28 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:28 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:28 INFO mapred.MapTask: Processing split: hdfs://localhost:9000/user/hadoop/grep-temp-410115572/part-r-00000:0+564
17/08/07 14:47:28 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 14:47:28 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 14:47:28 INFO mapred.MapTask: soft limit at 83886080
17/08/07 14:47:28 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 14:47:28 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 14:47:28 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 14:47:28 INFO mapred.LocalJobRunner: 
17/08/07 14:47:28 INFO mapred.MapTask: Starting flush of map output
17/08/07 14:47:28 INFO mapred.MapTask: Spilling map output
17/08/07 14:47:28 INFO mapred.MapTask: bufstart = 0; bufend = 358; bufvoid = 104857600
17/08/07 14:47:28 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214340(104857360); length = 57/6553600
17/08/07 14:47:28 INFO mapred.MapTask: Finished spill 0
17/08/07 14:47:28 INFO mapred.Task: Task:attempt_local391964061_0002_m_000000_0 is done. And is in the process of committing
17/08/07 14:47:28 INFO mapred.LocalJobRunner: map
17/08/07 14:47:28 INFO mapred.Task: Task 'attempt_local391964061_0002_m_000000_0' done.
17/08/07 14:47:28 INFO mapred.LocalJobRunner: Finishing task: attempt_local391964061_0002_m_000000_0
17/08/07 14:47:28 INFO mapred.LocalJobRunner: map task executor complete.
17/08/07 14:47:28 INFO mapred.LocalJobRunner: Waiting for reduce tasks
17/08/07 14:47:28 INFO mapred.LocalJobRunner: Starting task: attempt_local391964061_0002_r_000000_0
17/08/07 14:47:28 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 14:47:28 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 14:47:28 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 14:47:28 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@41b0aba1
17/08/07 14:47:28 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=372139616, maxSingleShuffleLimit=93034904, mergeThreshold=245612160, ioSortFactor=10, memToMemMergeOutputsThreshold=10
17/08/07 14:47:28 INFO reduce.EventFetcher: attempt_local391964061_0002_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
17/08/07 14:47:28 INFO reduce.LocalFetcher: localfetcher#2 about to shuffle output of map attempt_local391964061_0002_m_000000_0 decomp: 390 len: 394 to MEMORY
17/08/07 14:47:28 INFO reduce.InMemoryMapOutput: Read 390 bytes from map-output for attempt_local391964061_0002_m_000000_0
17/08/07 14:47:28 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 390, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->390
17/08/07 14:47:28 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 14:47:28 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
17/08/07 14:47:28 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 14:47:28 INFO reduce.MergeManagerImpl: finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
17/08/07 14:47:28 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 14:47:28 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 380 bytes
17/08/07 14:47:28 INFO reduce.MergeManagerImpl: Merged 1 segments, 390 bytes to disk to satisfy reduce memory limit
17/08/07 14:47:28 INFO reduce.MergeManagerImpl: Merging 1 files, 394 bytes from disk
17/08/07 14:47:28 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
17/08/07 14:47:28 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 14:47:28 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 380 bytes
17/08/07 14:47:28 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 14:47:28 INFO mapred.Task: Task:attempt_local391964061_0002_r_000000_0 is done. And is in the process of committing
17/08/07 14:47:28 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 14:47:28 INFO mapred.Task: Task attempt_local391964061_0002_r_000000_0 is allowed to commit now
17/08/07 14:47:28 INFO output.FileOutputCommitter: Saved output of task 'attempt_local391964061_0002_r_000000_0' to hdfs://localhost:9000/output/_temporary/0/task_local391964061_0002_r_000000
17/08/07 14:47:28 INFO mapred.LocalJobRunner: reduce > reduce
17/08/07 14:47:28 INFO mapred.Task: Task 'attempt_local391964061_0002_r_000000_0' done.
17/08/07 14:47:28 INFO mapred.LocalJobRunner: Finishing task: attempt_local391964061_0002_r_000000_0
17/08/07 14:47:28 INFO mapred.LocalJobRunner: reduce task executor complete.
17/08/07 14:47:29 INFO mapreduce.Job: Job job_local391964061_0002 running in uber mode : false
17/08/07 14:47:29 INFO mapreduce.Job:  map 100% reduce 100%
17/08/07 14:47:29 INFO mapreduce.Job: Job job_local391964061_0002 completed successfully
17/08/07 14:47:29 INFO mapreduce.Job: Counters: 35
	File System Counters
		FILE: Number of bytes read=1330778
		FILE: Number of bytes written=2508916
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=163894
		HDFS: Number of bytes written=1396
		HDFS: Number of read operations=151
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=16
	Map-Reduce Framework
		Map input records=15
		Map output records=15
		Map output bytes=358
		Map output materialized bytes=394
		Input split bytes=131
		Combine input records=0
		Combine output records=0
		Reduce input groups=5
		Reduce shuffle bytes=394
		Reduce input records=15
		Reduce output records=15
		Spilled Records=30
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=0
		Total committed heap usage (bytes)=1063256064
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=564
	File Output Format Counters 
		Bytes Written=268
[hadoop@much hadoop-2.8.1]$ echo $?
0
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /
Found 3 items
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:37 /input
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:47 /output
drwxr-xr-x   - hadoop supergroup          0 2017-08-07 14:44 /user
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -ls /output
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2017-08-07 14:47 /output/_SUCCESS
-rw-r--r--   1 hadoop supergroup        268 2017-08-07 14:47 /output/part-r-00000
[hadoop@much hadoop-2.8.1]$ bin/hdfs dfs -cat /output/part-r-00000
6	dfs.audit.logger
4	dfs.class
3	dfs.logger
3	dfs.server.namenode.
2	dfs.audit.log.maxbackupindex
2	dfs.period
2	dfs.audit.log.maxfilesize
1	dfs.replication
1	dfs.log
1	dfs.file
1	dfs.datanode.data.dir
1	dfs.servers
1	dfsadmin
1	dfsmetrics.log
1	dfs.namenode.name.dir
[hadoop@much hadoop-2.8.1]$ 
~~~

可见脱机调用 hadoop 小任务也可以对 hdfs 进行正常读写

![hadoop_hdfs_04.png](/images/hadoop/hadoop_hdfs_04.png)

---


## 停止 NameNode 与 DataNode 守护进程


~~~
[hadoop@much hadoop-2.8.1]$ ps faux | grep java 
hadoop   10873  0.0  0.0 112648   964 pts/0    S+   15:16   0:00                  \_ grep --color=auto java
hadoop    8201  0.5  8.2 2884040 333712 ?      Sl   14:11   0:21 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/java -Dproc_namenode -Xmx1000m -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/home/hadoop/hadoop-2.8.1/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/home/hadoop/hadoop-2.8.1 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Djava.library.path=/home/hadoop/hadoop-2.8.1/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/home/hadoop/hadoop-2.8.1/logs -Dhadoop.log.file=hadoop-hadoop-namenode-much.log -Dhadoop.home.dir=/home/hadoop/hadoop-2.8.1 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,RFA -Djava.library.path=/home/hadoop/hadoop-2.8.1/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS org.apache.hadoop.hdfs.server.namenode.NameNode
hadoop    8338  0.4  5.0 2859052 203272 ?      Sl   14:11   0:15 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/java -Dproc_datanode -Xmx1000m -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/home/hadoop/hadoop-2.8.1/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/home/hadoop/hadoop-2.8.1 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Djava.library.path=/home/hadoop/hadoop-2.8.1/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/home/hadoop/hadoop-2.8.1/logs -Dhadoop.log.file=hadoop-hadoop-datanode-much.log -Dhadoop.home.dir=/home/hadoop/hadoop-2.8.1 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,RFA -Djava.library.path=/home/hadoop/hadoop-2.8.1/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -server -Dhadoop.security.logger=ERROR,RFAS -Dhadoop.security.logger=ERROR,RFAS -Dhadoop.security.logger=ERROR,RFAS -Dhadoop.security.logger=INFO,RFAS org.apache.hadoop.hdfs.server.datanode.DataNode
hadoop    8511  0.2  6.5 2828548 263600 ?      Sl   14:11   0:08 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/java -Dproc_secondarynamenode -Xmx1000m -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/home/hadoop/hadoop-2.8.1/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/home/hadoop/hadoop-2.8.1 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Djava.library.path=/home/hadoop/hadoop-2.8.1/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/home/hadoop/hadoop-2.8.1/logs -Dhadoop.log.file=hadoop-hadoop-secondarynamenode-much.log -Dhadoop.home.dir=/home/hadoop/hadoop-2.8.1 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,RFA -Djava.library.path=/home/hadoop/hadoop-2.8.1/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
[hadoop@much hadoop-2.8.1]$ jps 
8338 DataNode
8201 NameNode
10874 Jps
8511 SecondaryNameNode
[hadoop@much hadoop-2.8.1]$ sbin/stop-dfs.sh 
Stopping namenodes on [localhost]
localhost: stopping namenode
localhost: stopping datanode
Stopping secondary namenodes [0.0.0.0]
0.0.0.0: stopping secondarynamenode
[hadoop@much hadoop-2.8.1]$ jps 
11247 Jps
[hadoop@much hadoop-2.8.1]$ 
[hadoop@much hadoop-2.8.1]$ ps faux | grep java 
hadoop   11260  0.0  0.0 112648   960 pts/0    S+   15:17   0:00                  \_ grep --color=auto java
[hadoop@much hadoop-2.8.1]$ 
~~~

web 管理界面将会无法访问


---

# 命令汇总

* **`rm -rf output/`**
* **`cat 21centry`**
* **`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar    wordcount  ./21centry   ./test_output`**
* **`head -n 20 test_output/part-r-00000`**
* **`mkdir hdfs`**
* **`ll hdfs/ -d`**
* **`cd hdfs/`**
* **`pwd`**
* **`ssh localhost`**
* **`cat etc/hadoop/core-site.xml`**
* **`cat etc/hadoop/hdfs-site.xml`**
* **`bin/hdfs namenode -format`**
* **`tree hdfs/`**
* **`jps`**
* **`sbin/start-dfs.sh`**
* **`firewall-cmd --add-port 50070/tcp`**
* **`firewall-cmd --list-all`**
* **`bin/hdfs dfs -mkdir /user`**
* **`bin/hdfs dfs -mkdir /user/hduser`**
* **`bin/hdfs dfs -mkdir /input`**
* **`bin/hdfs dfs -ls /`**
* **`bin/hdfs dfs -put etc/hadoop/  /input`**
* **`bin/hdfs dfs -ls /input`**
* **`bin/hdfs dfs -ls /input/hadoop`**
* **`tree hdfs/`**
* **`bin/hdfs dfs -ls /input`**
* **`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar  grep  /input/hadoop  /output 'dfs[a-z.]+'`**
* **`bin/hdfs dfs -ls /output`**
* **`bin/hdfs dfs -cat /output/part-r-00000`**
* **`ps faux | grep java`**
* **`sbin/stop-dfs.sh`**
* **`jps`**
* **`ps faux | grep java`**


---

[hadoop]:http://hadoop.apache.org/
[singlecluster]:https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
[hadoop_doc]:https://wiki.apache.org/hadoop
[install_hadoop_on_ubuntu]:http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/
[hadoop_01_standalone]:/2017/08/07/hadoop-01-standalone/
[hdfs_arch]:http://itm-vm.shidler.hawaii.edu/HDFS/ArchDocOverview.html
[filesystemshell]:https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html
