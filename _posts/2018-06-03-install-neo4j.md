---
layout: post
title: "Install neo4j"
author:  wilmosfang
date: 2018-06-03 03:16:05
image: '/assets/img/'
excerpt: '安装 neo4j'
main-class: 'neo4j'
color: '#63b345'
tags:
 - neo4j
 - docker
categories: 
 - neo4j
twitter_text: 'simple process of neo4j installation'
introduction: 'Installation of neo4j'
---

# 前言

**[neo4j][neo4j]** 是一款开源的图式数据库

什么是图：

由节点与关系构成的集合

>A graph is composed of two elements: a node and a relationship. Each node represents an entity (a person, place, thing, category or other piece of data), and each relationship represents how two nodes are associated. This general-purpose structure allows you to model all kinds of scenarios – from a system of roads, to a network of devices, to a population’s medical history or anything else defined by relationships

什么是图式数据库:

管理图元素 CRUD 的软件系统

>A graph database is an online database management system with Create, Read, Update and Delete (CRUD) operations working on a graph data model

**[neo4j][neo4j]** 是一款优秀的图式数据库

>Neo4j is a highly scalable, robust, native graph database

根据图数据库处理的对象特性，就很容易知道它的应用场景，最常见的就是人物关系的数据管理

这里演示一下如何构建 **[neo4j][neo4j]**

参考 **[Get Started][neo4j_started]**

> **Tip:** 当前的版本为 **neo4j-community-3.4.0**

---

# 操作

## 系统环境

~~~
[root@h171 ~]# hostnamectl 
   Static hostname: h171
         Icon name: computer-vm
           Chassis: vm
        Machine ID: d46f9440d4be429ea66b726977adf233
           Boot ID: b285f5a2d43e4b02849c7b267e307993
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[root@h171 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 79729sec preferred_lft 79729sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:48:f4:2c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.171/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe48:f42c/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:85:ac:18:8b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:85ff:feac:188b/64 scope link 
       valid_lft forever preferred_lft forever
[root@h171 ~]# cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core) 
[root@h171 ~]# docker version
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:20:16 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:23:58 2018
  OS/Arch:      linux/amd64
  Experimental: false
[root@h171 ~]# 
~~~


## 安装 neo4j


这里我准备直接使用 docker 安装

参考 **[Start an instance of neo4j][neo4j_docker]**

~~~
[root@h171 ~]# docker run \
>     --publish=7474:7474 --publish=7687:7687 \
>     --volume=$HOME/neo4j/data:/data \
>     neo4j
Unable to find image 'neo4j:latest' locally
latest: Pulling from library/neo4j
ff3a5c916c92: Pull complete 
5de5f69f42d7: Pull complete 
fa7536dd895a: Pull complete 
70a9fb6cb502: Pull complete 
ec1eefcd2625: Pull complete 
98725ec0d2cc: Pull complete 
d6ec8050656b: Pull complete 
Digest: sha256:abe3297efdcbb683f8a7624a1eab908f738ffcc8769916e02ed7abb464a0d9cb
Status: Downloaded newer image for neo4j:latest
Active database: graph.db
Directories in use:
  home:         /var/lib/neo4j
  config:       /var/lib/neo4j/conf
  logs:         /var/lib/neo4j/logs
  plugins:      /var/lib/neo4j/plugins
  import:       /var/lib/neo4j/import
  data:         /var/lib/neo4j/data
  certificates: /var/lib/neo4j/certificates
  run:          /var/lib/neo4j/run
Starting Neo4j.
2018-06-03 04:07:51.785+0000 WARN  Unknown config option: causal_clustering.discovery_listen_address
2018-06-03 04:07:51.788+0000 WARN  Unknown config option: causal_clustering.raft_advertised_address
2018-06-03 04:07:51.789+0000 WARN  Unknown config option: causal_clustering.raft_listen_address
2018-06-03 04:07:51.789+0000 WARN  Unknown config option: ha.host.coordination
2018-06-03 04:07:51.789+0000 WARN  Unknown config option: causal_clustering.transaction_advertised_address
2018-06-03 04:07:51.789+0000 WARN  Unknown config option: causal_clustering.discovery_advertised_address
2018-06-03 04:07:51.789+0000 WARN  Unknown config option: ha.host.data
2018-06-03 04:07:51.790+0000 WARN  Unknown config option: causal_clustering.transaction_listen_address
2018-06-03 04:07:51.805+0000 INFO  ======== Neo4j 3.4.0 ========
2018-06-03 04:07:51.841+0000 INFO  Starting...
2018-06-03 04:07:54.246+0000 INFO  Bolt enabled on 0.0.0.0:7687.
2018-06-03 04:07:56.874+0000 INFO  Started.
2018-06-03 04:07:57.821+0000 INFO  Remote interface available at http://localhost:7474/
2018-06-03 04:11:50.372+0000 ERROR Unexpected error detected in bolt session '0242acfffe110002-00000007-00000001-756de5b38b381bf1-8b9ab9f7'. The client is unauthorized due to authentication failure.
org.neo4j.bolt.v1.runtime.BoltConnectionFatality: The client is unauthorized due to authentication failure.
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.handleFailure(BoltStateMachine.java:740)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.handleFailure(BoltStateMachine.java:726)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.access$300(BoltStateMachine.java:62)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine$State$1.init(BoltStateMachine.java:434)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.init(BoltStateMachine.java:140)
	at org.neo4j.bolt.v1.messaging.BoltMessageRouter.lambda$onInit$0(BoltMessageRouter.java:70)
	at org.neo4j.bolt.runtime.DefaultBoltConnection.processNextBatch(DefaultBoltConnection.java:193)
	at org.neo4j.bolt.runtime.DefaultBoltConnection.processNextBatch(DefaultBoltConnection.java:143)
	at org.neo4j.bolt.runtime.ExecutorBoltScheduler.executeBatch(ExecutorBoltScheduler.java:163)
	at org.neo4j.bolt.runtime.ExecutorBoltScheduler.lambda$null$0(ExecutorBoltScheduler.java:145)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1590)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
2018-06-03 04:30:03.561+0000 ERROR Unexpected error detected in bolt session '0242acfffe110002-00000007-00000002-b826c79acbf8cb12-f0d52ae4'. The client is unauthorized due to authentication failure.
org.neo4j.bolt.v1.runtime.BoltConnectionFatality: The client is unauthorized due to authentication failure.
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.handleFailure(BoltStateMachine.java:740)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.handleFailure(BoltStateMachine.java:726)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.access$300(BoltStateMachine.java:62)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine$State$1.init(BoltStateMachine.java:434)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.init(BoltStateMachine.java:140)
	at org.neo4j.bolt.v1.messaging.BoltMessageRouter.lambda$onInit$0(BoltMessageRouter.java:70)
	at org.neo4j.bolt.runtime.DefaultBoltConnection.processNextBatch(DefaultBoltConnection.java:193)
	at org.neo4j.bolt.runtime.DefaultBoltConnection.processNextBatch(DefaultBoltConnection.java:143)
	at org.neo4j.bolt.runtime.ExecutorBoltScheduler.executeBatch(ExecutorBoltScheduler.java:163)
	at org.neo4j.bolt.runtime.ExecutorBoltScheduler.lambda$null$0(ExecutorBoltScheduler.java:145)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1590)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
2018-06-03 04:30:13.131+0000 ERROR Unexpected error detected in bolt session '0242acfffe110002-00000007-00000003-ed874af88bf8f074-384516ca'. The client is unauthorized due to authentication failure.
org.neo4j.bolt.v1.runtime.BoltConnectionFatality: The client is unauthorized due to authentication failure.
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.handleFailure(BoltStateMachine.java:740)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.handleFailure(BoltStateMachine.java:726)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.access$300(BoltStateMachine.java:62)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine$State$1.init(BoltStateMachine.java:434)
	at org.neo4j.bolt.v1.runtime.BoltStateMachine.init(BoltStateMachine.java:140)
	at org.neo4j.bolt.v1.messaging.BoltMessageRouter.lambda$onInit$0(BoltMessageRouter.java:70)
	at org.neo4j.bolt.runtime.DefaultBoltConnection.processNextBatch(DefaultBoltConnection.java:193)
	at org.neo4j.bolt.runtime.DefaultBoltConnection.processNextBatch(DefaultBoltConnection.java:143)
	at org.neo4j.bolt.runtime.ExecutorBoltScheduler.executeBatch(ExecutorBoltScheduler.java:163)
	at org.neo4j.bolt.runtime.ExecutorBoltScheduler.lambda$null$0(ExecutorBoltScheduler.java:145)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1590)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

...
...
...
~~~

## 进行访问 

需要密码进行登录

![neo4j](/assets/img/neo4j/neo4j01.png)

因为没配置密码，所以无法登录

![neo4j](/assets/img/neo4j/neo4j02.png)


## 配置密码重装

容器的好处就体现出来了，不满意干掉重来的成本特别低

在生产和测试环境中就这一点好处就可以优化掉很多不必要的 troubleshooting 时间

先清理残存

~~~
[root@h171 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
17fad3e36cea        neo4j               "/sbin/tini -g -- /d…"   29 minutes ago      Exited (0) 5 seconds ago                       dazzling_babbage
9141a1590781        hello-world         "/hello"                 2 hours ago         Exited (0) 2 hours ago                         relaxed_ardinghelli
[root@h171 ~]# docker rm  17fad3e36cea
17fad3e36cea
[root@h171 ~]# ls
anaconda-ks.cfg  neo4j  original-ks.cfg
[root@h171 ~]# rm neo4j/ -rf 
[root@h171 ~]# ls
anaconda-ks.cfg  original-ks.cfg
[root@h171 ~]#
~~~

再一次带入密码配置

~~~
[root@h171 ~]# docker run --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --env NEO4J_AUTH=neo4j/abc123 neo4j
Changed password for user 'neo4j'.
Active database: graph.db
Directories in use:
  home:         /var/lib/neo4j
  config:       /var/lib/neo4j/conf
  logs:         /var/lib/neo4j/logs
  plugins:      /var/lib/neo4j/plugins
  import:       /var/lib/neo4j/import
  data:         /var/lib/neo4j/data
  certificates: /var/lib/neo4j/certificates
  run:          /var/lib/neo4j/run
Starting Neo4j.
2018-06-03 04:47:08.156+0000 WARN  Unknown config option: causal_clustering.discovery_listen_address
2018-06-03 04:47:08.160+0000 WARN  Unknown config option: causal_clustering.raft_advertised_address
2018-06-03 04:47:08.160+0000 WARN  Unknown config option: causal_clustering.raft_listen_address
2018-06-03 04:47:08.160+0000 WARN  Unknown config option: ha.host.coordination
2018-06-03 04:47:08.160+0000 WARN  Unknown config option: causal_clustering.transaction_advertised_address
2018-06-03 04:47:08.160+0000 WARN  Unknown config option: causal_clustering.discovery_advertised_address
2018-06-03 04:47:08.160+0000 WARN  Unknown config option: ha.host.data
2018-06-03 04:47:08.161+0000 WARN  Unknown config option: causal_clustering.transaction_listen_address
2018-06-03 04:47:08.178+0000 INFO  ======== Neo4j 3.4.0 ========
2018-06-03 04:47:08.227+0000 INFO  Starting...
2018-06-03 04:47:10.872+0000 INFO  Bolt enabled on 0.0.0.0:7687.
2018-06-03 04:47:13.187+0000 INFO  Started.
2018-06-03 04:47:14.157+0000 INFO  Remote interface available at http://localhost:7474/
...
...
...
~~~

关于 neo4j 容器的简单配置可以参考 **[This section describes how to run Neo4j in a Docker container][neo4j_conf]**

## 再次登录

![neo4j](/assets/img/neo4j/neo4j03.png)

输入密码后

![neo4j](/assets/img/neo4j/neo4j04.png)


这就代表 neo4j 已经正常安装了

---

# 总结

使用容器构建应用

是目前所有构建方式中最简单高效的

这代表着软件基础架构的未来

* TOC
{:toc}

---

[neo4j]:https://neo4j.com/
[neo4j_started]:https://neo4j.com/developer/get-started/
[neo4j_docker]:https://hub.docker.com/_/neo4j/
[neo4j_conf]:https://neo4j.com/docs/operations-manual/current/installation/docker/

