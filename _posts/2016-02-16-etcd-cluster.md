---
layout: post
title:  etcd 集群
author: wilmosfang
categories:  linux cluster etcd  
wc: 494  1412 20562 
excerpt: 配置、构建、启动etcd集群 ，成员信息查看，在线添加删除节点
comments: true
---



# 前言

**[etcd][etcd]** 在生产环境下一般都以集群的形式出现

构建 etcd 集群有以下三种方法

* **[Static][static]**
* **[etcd Discovery][etcd_discovery]**
* **[DNS Discovery][dns_discovery]**

这里简单分享一下使用静态方法构建 **[etcd][etcd]** 集群的操作 ，详细过程可以参考 **[Etcd Clustering Guide][etcd_cluster]**


> **Tip:** 当前的最新版本为 **etcd v2.2.5**
> 
> **Note:**  The master branch may be in an unstable or even broken state during development. Please use **[releases][releases]** instead of the master branch in order to get stable binaries

---


# 概要

* TOC
{:toc}



---

## 环境

~~~
[root@docker etcd]# hostnamectl 
   Static hostname: docker
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 12a02f8ee88d4b8e91d54d1390b0b275
           Boot ID: 42988a5820ea4d038a3a277920dcfff8
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.4.4.el7.x86_64
      Architecture: x86-64
[root@docker etcd]# cd etcd-v2.2.4-linux-amd64/
[root@docker etcd-v2.2.4-linux-amd64]# ./etcd --version
etcd Version: 2.2.4
Git SHA: bdee27b
Go Version: go1.5.3
Go OS/Arch: linux/amd64
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~



Name     | Address
-------- | ---
h102| 192.168.100.102
docker| 192.168.100.103
h104| 192.168.100.104


---

## 安装etcd

安装过程非常简单，就是下载解压，然后运行

详细可以参考 **[releases][releases]**

---

## etcd的配置方法


etcd是直接在命令行或环境变量中进行配置的，优先使用命令行中指定的设置

>etcd is configurable through command-line flags and environment variables. Options set on the command line take precedence over those from the environment.

详细配置方法可以参考 **[configuration][configuration]**

命令行配置和环境变量配置对应比较工整，如

CLI | ENV
-------- | ---
`-name`| `ETCD_NAME`
`-data-dir`| `ETCD_DATA_DIR`
`-wal-dir`| `ETCD_DATA_DIR`
`-snapshot-count`|`ETCD_SNAPSHOT_COUNT`

基本上都是小写转换为大写，在前面加上 **`ETCD`** ，然后将 **`-`**  换成 **`_`**


配置使用方法为：

~~~
ETCD_INITIAL_CLUSTER="infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380" ETCD_INITIAL_CLUSTER_STATE=new etcd
~~~

或

~~~
etcd -initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 -initial-cluster-state new
~~~

上面两种方法是一样的效果，也可以两种结合一起使用(重复指定的情况下，以命令行中的优先)


---

## 打开防火墙端口

~~~
[root@docker etcd-v2.2.4-linux-amd64]# firewall-cmd  --list-all
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 3306/tcp 80/tcp 40000/tcp 8080/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@docker etcd-v2.2.4-linux-amd64]# firewall-cmd  --add-port=2379/tcp
success
[root@docker etcd-v2.2.4-linux-amd64]# firewall-cmd  --add-port=2380/tcp
success
[root@docker etcd-v2.2.4-linux-amd64]# firewall-cmd  --list-all
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 2379/tcp 80/tcp 2380/tcp 8080/tcp 3306/tcp 40000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@docker etcd-v2.2.4-linux-amd64]#
~~~

确保 **2379** 和 **2380** 端口被打开，默认情况下

* **2379** 用来监听客户端请求
* **2380** 用来进行节点间通讯

> **Tip:** CentOS Linux 7 中使用的 **firewalld** 来管理防火墙设置


> **Note:** 其它节点也要确保这两个端口是开放的，否则无法正常工作

---

## 配置启动集群


~~~
[root@docker etcd-v2.2.4-linux-amd64]# ./etcd -name docker -initial-advertise-peer-urls http://192.168.100.103:2380 \
>   -listen-peer-urls http://192.168.100.103:2380 \
>   -listen-client-urls http://192.168.100.103:2379,http://127.0.0.1:2379 \
>   -advertise-client-urls http://192.168.100.103:2379 \
>   -initial-cluster-token etcd-cluster-test \
>   -initial-cluster docker=http://192.168.100.103:2380,h102=http://192.168.100.102:2380,h104=http://192.168.100.104:2380 \
>   -initial-cluster-state new
2016-02-16 15:35:49.278204 I | etcdmain: etcd Version: 2.2.4
2016-02-16 15:35:49.278496 I | etcdmain: Git SHA: bdee27b
2016-02-16 15:35:49.278504 I | etcdmain: Go Version: go1.5.3
2016-02-16 15:35:49.278510 I | etcdmain: Go OS/Arch: linux/amd64
...
...
...
----------
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcd -name h104 -initial-advertise-peer-urls http://192.168.100.104:2380 \
>   -listen-peer-urls http://192.168.100.104:2380 \
>   -listen-client-urls http://192.168.100.104:2379,http://127.0.0.1:2379 \
>   -advertise-client-urls http://192.168.100.104:2379 \
>   -initial-cluster-token etcd-cluster-test \
>   -initial-cluster docker=http://192.168.100.103:2380,h102=http://192.168.100.102:2380,h104=http://192.168.100.104:2380 \
>   -initial-cluster-state new
2016-02-16 15:36:13.898374 I | etcdmain: etcd Version: 2.2.4
2016-02-16 15:36:13.898513 I | etcdmain: Git SHA: bdee27b
2016-02-16 15:36:13.898522 I | etcdmain: Go Version: go1.5.3
2016-02-16 15:36:13.898530 I | etcdmain: Go OS/Arch: linux/amd64
...
...
...
----------
[root@h102 etcd-v2.2.4-linux-amd64]# ./etcd -name h102 -initial-advertise-peer-urls http://192.168.100.102:2380  -listen-peer-urls http://192.168.100.102:2380  -listen-client-urls http://192.168.100.102:2379,http://127.0.0.1:2379  -advertise-client-urls http://192.168.100.102:2379  -initial-cluster-token etcd-cluster-test  -initial-cluster docker=http://192.168.100.103:2380,h102=http://192.168.100.102:2380,h104=http://192.168.100.104:2380  -initial-cluster-state new
2016-02-16 15:37:38.592028 I | etcdmain: etcd Version: 2.2.4
2016-02-16 15:37:38.592139 I | etcdmain: Git SHA: bdee27b
2016-02-16 15:37:38.592155 I | etcdmain: Go Version: go1.5.3
2016-02-16 15:37:38.592163 I | etcdmain: Go OS/Arch: linux/amd64
...
...
...
~~~



Item     | Comment
-------- | ---
`-name`| 指定此节点的名字
`-initial-advertise-peer-urls`| 指定广播给其它节点的此节点地址
`-listen-peer-urls`| 指定此节点在集群中监听(接受)其它节点通信的地址
`-listen-client-urls`|指定用于监听客户端请求的地址
`-advertise-client-urls`| 指定广播给其它节点的此节点用于监听客户端请求的地址 
`-initial-cluster-token`| 指定此集群的统一token
`-initial-cluster`| 初始化集群,指定包含所有节点的一个列表
`-initial-cluster-state`| 初始集群的状态，可以是  ("new" or "existing")


配置的详细信息可以参考 **[configuration][configuration]**

---

## 查看成员信息

~~~
[root@h104 ~]# curl http://localhost:2379/v2/members
{"members":[{"id":"1b80a88a471eb4b8","name":"h104","peerURLs":["http://192.168.100.104:2380"],"clientURLs":["http://192.168.100.104:2379"]},{"id":"84099faad6d56427","name":"h102","peerURLs":["http://192.168.100.102:2380"],"clientURLs":["http://192.168.100.102:2379"]},{"id":"940f6e83e019a03f","name":"docker","peerURLs":["http://192.168.100.103:2380"],"clientURLs":["http://192.168.100.103:2379"]}]}
[root@h104 ~]# 
~~~


> **Note:**  要确保节点之间时间的一致，如果时间有太大偏差，etcd日志是会报警的

~~~
2016-02-16 15:49:38.674505 W | rafthttp: the clock difference against peer 940f6e83e019a03f is too high [4.377659814s > 1s]
2016-02-16 15:49:38.674572 W | rafthttp: the clock difference against peer 1b80a88a471eb4b8 is too high [4.372660776s > 1s]
~~~

节点间的时间有偏差

~~~
[root@docker etcd-v2.2.4-linux-amd64]# date
Tue Feb 16 15:45:38 CST 2016
[root@docker etcd-v2.2.4-linux-amd64]# date +%s
1455608804
[root@docker etcd-v2.2.4-linux-amd64]#
----------
[root@h102 tmp]# date
Tue Feb 16 15:45:43 CST 2016
[root@h102 tmp]# date +%s
1455608808
[root@h102 tmp]#
----------
[root@h104 ~]# date
Tue Feb 16 15:45:38 CST 2016
[root@h104 ~]# date +%s
1455608804
[root@h104 ~]#
~~~

时间同步后，警告就会消失


> **Tip:** 同步有很多种方式，ntp是生产中常用的方法，测试环境下，可以直接使用 **`date -s "2016-2-16 15:49:50"`** 的方式同时在每个节点中执行，有些shell工具提供了 **to all session** 的命令运行功能，时间点的选择可以是几个节点上跑得最快的那一个


---

## 简单测试

~~~
[root@h104 ~]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="set by h104"
{"action":"set","node":{"key":"/message","value":"set by h104","modifiedIndex":11,"createdIndex":11},"prevNode":{"key":"/message","value":"abc","modifiedIndex":10,"createdIndex":10}}
[root@h104 ~]# curl http://127.0.0.1:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"set by h104","modifiedIndex":11,"createdIndex":11}}
[root@h104 ~]# 
----------
[root@h102 tmp]# curl http://127.0.0.1:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"set by h104","modifiedIndex":11,"createdIndex":11}}
[root@h102 tmp]# 
----------
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"set by h104","modifiedIndex":11,"createdIndex":11}}
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

在h104上设定的数据自动同步到了h102和docker上，简单测试通过


---

## 添加删除节点

实际使用场景中，必然会遇到添加删除节点的情况

**[etcdctl][etcdctl]** 可以帮忙完成相关工作 

~~~
[root@h104 etcd-v2.2.4-linux-amd64]# curl http://localhost:2379/v2/members
{"members":[{"id":"1b80a88a471eb4b8","name":"h104","peerURLs":["http://192.168.100.104:2380"],"clientURLs":["http://192.168.100.104:2379"]},{"id":"84099faad6d56427","name":"h102","peerURLs":["http://192.168.100.102:2380"],"clientURLs":["http://192.168.100.102:2379"]},{"id":"940f6e83e019a03f","name":"docker","peerURLs":["http://192.168.100.103:2380"],"clientURLs":["http://192.168.100.103:2379"]}]}
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member list
1b80a88a471eb4b8: name=h104 peerURLs=http://192.168.100.104:2380 clientURLs=http://192.168.100.104:2379
84099faad6d56427: name=h102 peerURLs=http://192.168.100.102:2380 clientURLs=http://192.168.100.102:2379
940f6e83e019a03f: name=docker peerURLs=http://192.168.100.103:2380 clientURLs=http://192.168.100.103:2379
[root@h104 etcd-v2.2.4-linux-amd64]#
~~~


加减节点相关的操作

~~~
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member --help 
NAME:
   etcdctl member - member add, remove and list subcommands

USAGE:
   etcdctl member command [command options] [arguments...]

COMMANDS:
   list		enumerate existing cluster members
   add		add a new member to the etcd cluster
   remove	remove an existing member from the etcd cluster
   update	update an existing member in the etcd cluster
   help, h	Shows a list of commands or help for one command
   
OPTIONS:
   --help, -h	show help
   
[root@h104 etcd-v2.2.4-linux-amd64]#
~~~


下面演示一下在线添加删除节点的操作

---

### 删除一个节点 


删除节点相对简单

~~~
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member  list
1b80a88a471eb4b8: name=h104 peerURLs=http://192.168.100.104:2380 clientURLs=http://192.168.100.104:2379
84099faad6d56427: name=h102 peerURLs=http://192.168.100.102:2380 clientURLs=http://192.168.100.102:2379
940f6e83e019a03f: name=docker peerURLs=http://192.168.100.103:2380 clientURLs=http://192.168.100.103:2379
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member  remove 84099faad6d56427
Removed member 84099faad6d56427 from cluster
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member  list
1b80a88a471eb4b8: name=h104 peerURLs=http://192.168.100.104:2380 clientURLs=http://192.168.100.104:2379
940f6e83e019a03f: name=docker peerURLs=http://192.168.100.103:2380 clientURLs=http://192.168.100.103:2379
[root@h104 etcd-v2.2.4-linux-amd64]# curl http://localhost:2379/v2/members
{"members":[{"id":"1b80a88a471eb4b8","name":"h104","peerURLs":["http://192.168.100.104:2380"],"clientURLs":["http://192.168.100.104:2379"]},{"id":"940f6e83e019a03f","name":"docker","peerURLs":["http://192.168.100.103:2380"],"clientURLs":["http://192.168.100.103:2379"]}]}
[root@h104 etcd-v2.2.4-linux-amd64]# 
~~~

同时 h102 的日志中出现如下信息，最后退出


~~~
...
...
...
2016-02-16 17:42:21.320955 I | rafthttp: the connection with 1b80a88a471eb4b8 became inactive
2016-02-16 17:42:21.321053 E | rafthttp: failed to dial 1b80a88a471eb4b8 on stream MsgApp v2 (the member has been permanently removed from the cluster)
2016-02-16 17:42:21.321402 E | etcdserver: the member has been permanently removed from the cluster
2016-02-16 17:42:21.321452 I | etcdserver: the data-dir used by this member must be removed.
2016-02-16 17:42:21.341262 I | rafthttp: the connection with 940f6e83e019a03f became inactive
2016-02-16 17:42:21.341313 E | rafthttp: failed to dial 940f6e83e019a03f on stream MsgApp v2 (the member has been permanently removed from the cluster)
2016-02-16 17:42:21.342089 E | rafthttp: failed to dial 940f6e83e019a03f on stream Message (net/http: request canceled)
2016-02-16 17:42:21.342314 E | etcdhttp: error removing member 84099faad6d56427 (etcdserver: server stopped)
2016-02-16 17:42:21.342338 E | etcdhttp: got unexpected response error (etcdserver: server stopped)
[root@h102 etcd-v2.2.4-linux-amd64]#
~~~

> **Tip:**  直接删除leader也是安全的，只是在选举出新的leader前集群是不可用状态，时长相当于一个选举timeout周期加上投票过程耗费的总时间


---

### 添加一个节点

添加节点相对麻烦一点，分作两步：

* 使用 **`etcdctl member add`** 或 [members API][members_api] 添加节点
* 使用新的集群配置启动新加入的节点，包含一份所有当前成员的列表

~~~
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member  list
1b80a88a471eb4b8: name=h104 peerURLs=http://192.168.100.104:2380 clientURLs=http://192.168.100.104:2379
940f6e83e019a03f: name=docker peerURLs=http://192.168.100.103:2380 clientURLs=http://192.168.100.103:2379
[root@h104 etcd-v2.2.4-linux-amd64]# 
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member add new-h102 http://192.168.100.102:2380
Added member named new-h102 with ID cdc1e5e338e27adc to cluster

ETCD_NAME="new-h102"
ETCD_INITIAL_CLUSTER="h104=http://192.168.100.104:2380,docker=http://192.168.100.103:2380,new-h102=http://192.168.100.102:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
[root@h104 etcd-v2.2.4-linux-amd64]#
~~~

执行完后，终端反馈出几个关键的环境变量

要使用这些环境变量来运行新加入的节点，当前情况下新节点还没运行

~~~
[root@h104 etcd-v2.2.4-linux-amd64]# ./etcdctl  member  list
1b80a88a471eb4b8: name=h104 peerURLs=http://192.168.100.104:2380 clientURLs=http://192.168.100.104:2379
940f6e83e019a03f: name=docker peerURLs=http://192.168.100.103:2380 clientURLs=http://192.168.100.103:2379
cdc1e5e338e27adc[unstarted]: peerURLs=http://192.168.100.102:2380
[root@h104 etcd-v2.2.4-linux-amd64]#
~~~

使用环境变量运行新节点，将节点加入集群

~~~
[root@h102 etcd-v2.2.4-linux-amd64]# ETCD_NAME="new-h102" ETCD_INITIAL_CLUSTER="h104=http://192.168.100.104:2380,docker=http://192.168.100.103:2380,new-h102=http://192.168.100.102:2380"   ETCD_INITIAL_CLUSTER_STATE="existing"   /usr/local/src/etcd/etcd-v2.2.4-linux-amd64/etcd -listen-client-urls http://192.168.100.102:2379,http://127.0.0.1:2379   -advertise-client-urls http://192.168.100.102:2379 -listen-peer-urls http://192.168.100.102:2380   -initial-advertise-peer-urls http://192.168.100.102:2380
2016-02-16 18:46:13.884181 I | flags: recognized and used environment variable ETCD_INITIAL_CLUSTER=h104=http://192.168.100.104:2380,docker=http://192.168.100.103:2380,new-h102=http://192.168.100.102:2380
2016-02-16 18:46:13.884325 I | flags: recognized and used environment variable ETCD_INITIAL_CLUSTER_STATE=existing
2016-02-16 18:46:13.884363 I | flags: recognized and used environment variable ETCD_NAME=new-h102
2016-02-16 18:46:13.884512 I | etcdmain: etcd Version: 2.2.4
2016-02-16 18:46:13.884528 I | etcdmain: Git SHA: bdee27b
2016-02-16 18:46:13.884541 I | etcdmain: Go Version: go1.5.3
2016-02-16 18:46:13.884554 I | etcdmain: Go OS/Arch: linux/amd64
...
...
...
~~~

节点可以正常工作

~~~
[root@h102 etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"set by h104","modifiedIndex":11,"createdIndex":11}}
[root@h102 etcd-v2.2.4-linux-amd64]# 
~~~

再次查看成员状态

~~~
[root@h102 etcd-v2.2.4-linux-amd64]# ./etcdctl  member list
1b80a88a471eb4b8: name=h104 peerURLs=http://192.168.100.104:2380 clientURLs=http://192.168.100.104:2379
940f6e83e019a03f: name=docker peerURLs=http://192.168.100.103:2380 clientURLs=http://192.168.100.103:2379
cdc1e5e338e27adc: name=new-h102 peerURLs=http://192.168.100.102:2380 clientURLs=http://192.168.100.102:2379
[root@h102 etcd-v2.2.4-linux-amd64]# 
~~~


> **Note:** 如果要添加多个节点，建议一次只添加一个，然后检查节点和集群运行状态正常后再依次逐个添加其它节点



---

# 命令汇总

* **`hostnamectl`**
* **`cd etcd-v2.2.4-linux-amd64/`**
* **`./etcd --version`**
* **`firewall-cmd  --add-port=2379/tcp`**
* **`firewall-cmd  --add-port=2380/tcp`**
* **`firewall-cmd  --list-all`**
* **`./etcd -name docker -initial-advertise-peer-urls http://192.168.100.103:2380 \`**
* **`./etcd -name h104 -initial-advertise-peer-urls http://192.168.100.104:2380 \`**
* **`./etcd -name h102 -initial-advertise-peer-urls http://192.168.100.102:2380  -listen-peer-urls http://192.168.100.102:2380  -listen-client-urls http://192.168.100.102:2379,http://127.0.0.1:2379  -advertise-client-urls http://192.168.100.102:2379  -initial-cluster-token etcd-cluster-test  -initial-cluster docker=http://192.168.100.103:2380,h102=http://192.168.100.102:2380,h104=http://192.168.100.104:2380  -initial-cluster-state new`**
* **`date`**
* **`date +%s`**
* **`curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="set by h104"`**
* **`curl http://127.0.0.1:2379/v2/keys/message`**
* **`./etcdctl  member --help`**
* **`./etcdctl  member  remove 84099faad6d56427`**
* **`curl http://localhost:2379/v2/members`**
* **`./etcdctl  member add new-h102 http://192.168.100.102:2380`**
* **`./etcdctl  member  list`**
* **`ETCD_NAME="new-h102" ETCD_INITIAL_CLUSTER="h104=http://192.168.100.104:2380,docker=http://192.168.100.103:2380,new-h102=http://192.168.100.102:2380"   ETCD_INITIAL_CLUSTER_STATE="existing"   /usr/local/src/etcd/etcd-v2.2.4-linux-amd64/etcd -listen-client-urls http://192.168.100.102:2379,http://127.0.0.1:2379   -advertise-client-urls http://192.168.100.102:2379 -listen-peer-urls http://192.168.100.102:2380   -initial-advertise-peer-urls http://192.168.100.102:2380`**
* **`curl http://127.0.0.1:2379/v2/keys/message`**


---

[releases]:https://github.com/coreos/etcd/releases/
[etcd]:https://github.com/coreos/etcd
[etcd_cluster]:https://github.com/coreos/etcd/blob/master/Documentation/clustering.md
[static]:https://github.com/coreos/etcd/blob/master/Documentation/clustering.md#static
[etcd_discovery]:https://github.com/coreos/etcd/blob/master/Documentation/clustering.md#etcd-discovery
[dns_discovery]:https://github.com/coreos/etcd/blob/master/Documentation/clustering.md#dns-discovery
[configuration]:https://github.com/coreos/etcd/blob/master/Documentation/configuration.md
[etcdctl]:https://github.com/coreos/etcd/tree/master/etcdctl
[members_api]:https://github.com/coreos/etcd/blob/master/Documentation/members_api.md#post-v2members

