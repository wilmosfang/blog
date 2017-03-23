---
layout: post
title: RabbitMQ集群II
author: wilmosfang
tags:  rabbitmq cluster
categories:  rabbitmq
wc: 471 1884 21044
excerpt: follow me
comments: true
---


---

# 前言

**[RabbitMQ][rabbitmq]** 是一款开源的消息代理服务器，用来进行信息路由。

MQ可以使架构变得松耦合，从而更有弹性，更灵活，是SOA架构不可或缺的组成部分，担当服务总线或信息总线的角色。

可用性在生产系统中是非常重要的指标， **[RabbitMQ][rabbitmq]** 对集群进行了很好的支持

下面分享一下 **[RabbitMQ][rabbitmq]** 的集群操作，详细可以参阅 [官方文档][cluster]

> **Tip:** 当前版本 **RabbitMQ 3.5.6 release**

---

# 概要

* TOC
{:toc}



---

## 升级集群

升级Erlang或RabbitMQ版本，必须停止集群，因为集群中不能容忍不同版本协同工作

在集群升级之前最好确认哪一个Node是第一个升级的，这个node必须是最后一个关闭，然后第一个启动。否则在这个node和实际最后一个关闭的node之前的配置变更都会丢失

在集群中，要使用DISC node来主导升级，而不能使用RAM node，会报错，从安全层面也可以理解这样做的用意


>When upgrading from one major or minor version of RabbitMQ to another (i.e. from 3.0.x to 3.1.x, or from 2.x.x to 3.x.x), or when upgrading Erlang, the whole cluster must be taken down for the upgrade (since clusters cannot run mixed versions like this). This will not be the case when upgrading from one patch version to another (i.e. from 3.0.x to 3.0.y); these versions can be mixed in a cluster (with the exception that 3.0.0 cannot be mixed with later versions from the 3.0.x series).
>
>RabbitMQ will automatically update its persistent data structures if necessary when upgrading between major / minor versions. In a cluster, this task is performed by the first disc node to be started (the "upgrader" node). Therefore when upgrading a RabbitMQ cluster, you should not attempt to start any RAM nodes first; any RAM nodes started will emit an error message and fail to start up.
>
>While not strictly necessary, it is a good idea to decide ahead of time which disc node will be the upgrader, stop that node last, and start it first. Otherwise changes to the cluster configuration that were made between the upgrader node stopping and the last node stopping will be lost.
>
>Automatic upgrades are only possible from RabbitMQ versions 2.1.1 and later. If you have an earlier cluster, you will need to rebuild it to upgrade.


---

## 单机集群


在同一个OS中运行多个RabbitMQ node主要要满足以下两个条件：

* 1 每一个node使用的名字不能重复
* 2 每一个node使用的port / IP不能重复


~~~
[root@h101 ~]# rabbitmqctl  status
Status of node rabbit@h101 ...
Error: unable to connect to node rabbit@h101: nodedown

DIAGNOSTICS
===========

attempted to contact: [rabbit@h101]

rabbit@h101:
  * connected to epmd (port 4369) on h101
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on h101
  * suggestion: start the node

current node details:
- node name: 'rabbitmq-cli-2840@h101'
- home dir: /var/lib/rabbitmq
- cookie hash: FkPvQU6k3zuamsY9Ow+9Og==

[root@h101 ~]# 
[root@h101 ~]# free -m 
             total       used       free     shared    buffers     cached
Mem:          3824        659       3165          0         20         84
-/+ buffers/cache:        553       3271
Swap:         3999          0       3999
[root@h101 ~]# 
~~~

使用不同名字，不同端口，分别从后台启动两个服务进程

~~~
[root@h101 ~]# RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit rabbitmq-server -detached
Warning: PID file not written; -detached was passed.
[root@h101 ~]# ps faux | grep mq 
root      3054  0.0  0.0 103252   828 pts/0    S+   14:21   0:00          \_ grep mq
rabbitmq  2888  0.0  0.0  10828   468 ?        S    14:14   0:00 /usr/lib64/erlang/erts-5.8.5/bin/epmd -daemon
rabbitmq  2975 22.7  0.8 1088040 33292 ?       Sl   14:21   0:02 /usr/lib64/erlang/erts-5.8.5/bin/beam.smp -W w -A 64 -P 1048576 -K erlang -progname erl -- -home /var/lib/rabbitmq -- -pa /usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../ebin -noshell -noinput -st -boot start_sasl -kernel inet_default_connect_options [{nodelay,true}] -rabbit tcp_listeners [{"auto",5672}] -sasl errlog_type errr false -rabbit error_logger {file,"/var/log/rabbitmq/rabbit.log"} -rabbit sasl_error_logger {file,"/var/log/rabbitmq/rabbit-sasl.lons_file "/etc/rabbitmq/enabled_plugins" -rabbit plugins_dir "/usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../plugins" -rabbit plb/rabbitmq/mnesia/rabbit-plugins-expand" -os_mon start_cpu_sup false -os_mon start_disksup false -os_mon start_memsup false -mnesia esia/rabbit" -kernel inet_dist_listen_min 25672 -kernel inet_dist_listen_max 25672 -noshell -noinput
rabbitmq  3050  0.0  0.0  10792   508 ?        Ss   14:21   0:00  \_ inet_gethost 4
rabbitmq  3051  0.0  0.0  12896   648 ?        S    14:21   0:00      \_ inet_gethost 4
[root@h101 ~]# RABBITMQ_NODE_PORT=5673 RABBITMQ_NODENAME=hare rabbitmq-server -detached
Warning: PID file not written; -detached was passed.
[root@h101 ~]# ps faux | grep mq 
root      3196  0.0  0.0 103252   824 pts/0    S+   14:22   0:00          \_ grep mq
rabbitmq  2888  0.0  0.0  10828   468 ?        S    14:14   0:00 /usr/lib64/erlang/erts-5.8.5/bin/epmd -daemon
rabbitmq  2975  8.6  0.8 1088040 33292 ?       Sl   14:21   0:02 /usr/lib64/erlang/erts-5.8.5/bin/beam.smp -W w -A 64 -P 1048576 -K erlang -progname erl -- -home /var/lib/rabbitmq -- -pa /usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../ebin -noshell -noinput -st -boot start_sasl -kernel inet_default_connect_options [{nodelay,true}] -rabbit tcp_listeners [{"auto",5672}] -sasl errlog_type errr false -rabbit error_logger {file,"/var/log/rabbitmq/rabbit.log"} -rabbit sasl_error_logger {file,"/var/log/rabbitmq/rabbit-sasl.lons_file "/etc/rabbitmq/enabled_plugins" -rabbit plugins_dir "/usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../plugins" -rabbit plb/rabbitmq/mnesia/rabbit-plugins-expand" -os_mon start_cpu_sup false -os_mon start_disksup false -os_mon start_memsup false -mnesia esia/rabbit" -kernel inet_dist_listen_min 25672 -kernel inet_dist_listen_max 25672 -noshell -noinput
rabbitmq  3050  0.0  0.0  10792   508 ?        Ss   14:21   0:00  \_ inet_gethost 4
rabbitmq  3051  0.0  0.0  12896   648 ?        S    14:21   0:00      \_ inet_gethost 4
rabbitmq  3116 29.8  0.8 1087524 33684 ?       Sl   14:21   0:01 /usr/lib64/erlang/erts-5.8.5/bin/beam.smp -W w -A 64 -P 1048576 -K erlang -progname erl -- -home /var/lib/rabbitmq -- -pa /usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../ebin -noshell -noinput -s-boot start_sasl -kernel inet_default_connect_options [{nodelay,true}] -rabbit tcp_listeners [{"auto",5673}] -sasl errlog_type errorfalse -rabbit error_logger {file,"/var/log/rabbitmq/hare.log"} -rabbit sasl_error_logger {file,"/var/log/rabbitmq/hare-sasl.log"} -re "/etc/rabbitmq/enabled_plugins" -rabbit plugins_dir "/usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../plugins" -rabbit plugins_itmq/mnesia/hare-plugins-expand" -os_mon start_cpu_sup false -os_mon start_disksup false -os_mon start_memsup false -mnesia dir "/vae" -kernel inet_dist_listen_min 25673 -kernel inet_dist_listen_max 25673 -noshell -noinput
rabbitmq  3191  0.0  0.0  10792   508 ?        Ss   14:21   0:00  \_ inet_gethost 4
rabbitmq  3192  0.0  0.0  12896   648 ?        S    14:21   0:00      \_ inet_gethost 4
[root@h101 ~]#
~~~

查看node状态

~~~
[root@h101 ~]# rabbitmqctl  -n hare status
Status of node hare@h101 ...
[{pid,3116},
 {running_applications,[{rabbit,"RabbitMQ","3.5.6"},
                        {mnesia,"MNESIA  CXC 138 12","4.5"},
                        {os_mon,"CPO  CXC 138 46","2.2.7"},
                        {xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,28139112},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2704},
          {queue_procs,2704},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9342280},
          {mnesia,60144},
          {mgmt_db,0},
          {msg_index,33256},
          {other_ets,794360},
          {binary,8712},
          {code,14756074},
          {atom,1366585},
          {other_system,1772293}]},
 {alarms,[]},
 {listeners,[{clustering,25673,"::"},{amqp,5673,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,1604293427},
 {disk_free_limit,50000000},
 {disk_free,43578609664},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,124}]},
 {run_queue,0},
 {uptime,26}]
[root@h101 ~]# rabbitmqctl  -n rabbit  status
Status of node rabbit@h101 ...
[{pid,2975},
 {running_applications,[{rabbit,"RabbitMQ","3.5.6"},
                        {mnesia,"MNESIA  CXC 138 12","4.5"},
                        {os_mon,"CPO  CXC 138 46","2.2.7"},
                        {xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,28100032},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2704},
          {queue_procs,2704},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9303984},
          {mnesia,60144},
          {mgmt_db,0},
          {msg_index,33480},
          {other_ets,794816},
          {binary,8808},
          {code,14756074},
          {atom,1366585},
          {other_system,1770733}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,1604293427},
 {disk_free_limit,50000000},
 {disk_free,43578597376},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,124}]},
 {run_queue,0},
 {uptime,56}]
[root@h101 ~]# netstat  -ant | grep 567
tcp        0      0 0.0.0.0:25672               0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:25673               0.0.0.0:*                   LISTEN      
tcp        0      0 192.168.100.101:36794       192.168.100.101:25673       TIME_WAIT   
tcp        0      0 192.168.100.101:57271       192.168.100.101:25672       TIME_WAIT   
tcp        0      0 :::5672                     :::*                        LISTEN      
tcp        0      0 :::5673                     :::*                        LISTEN      
[root@h101 ~]# ll /var/lib/rabbitmq/.erlang.cookie 
-r-------- 1 rabbitmq rabbitmq 20 Oct 23 00:00 /var/lib/rabbitmq/.erlang.cookie
[root@h101 ~]# 
~~~

查看集群状态


~~~
[root@h101 mnesia]# rabbitmqctl  -n hare cluster_status
Cluster status of node hare@h101 ...
[{nodes,[{disc,[hare@h101]}]},
 {running_nodes,[hare@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 mnesia]# rabbitmqctl  -n rabbit cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 mnesia]#
~~~

创建集群

~~~
[root@h101 mnesia]# rabbitmqctl  -n rabbit stop_app 
Stopping node rabbit@h101 ...
[root@h101 mnesia]# rabbitmqctl  -n rabbit cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]}]}]
[root@h101 mnesia]# rabbitmqctl  -n rabbit status
Status of node rabbit@h101 ...
[{pid,2975},
 {running_applications,[{xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,27473232},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,0},
          {queue_procs,0},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,8922352},
          {mnesia,0},
          {mgmt_db,0},
          {msg_index,0},
          {other_ets,643008},
          {binary,3416},
          {code,14756074},
          {atom,1367393},
          {other_system,1780989}]},
 {alarms,[]},
 {listeners,[]},
 {processes,[{limit,1048576},{used,46}]},
 {run_queue,0},
 {uptime,251}]
[root@h101 mnesia]# rabbitmqctl  -n rabbit join_cluster hare@h101
Clustering node rabbit@h101 with hare@h101 ...
[root@h101 mnesia]# rabbitmqctl  -n rabbit start_app 
Starting node rabbit@h101 ...
[root@h101 mnesia]# rabbitmqctl  -n rabbit cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[hare@h101,rabbit@h101]}]},
 {running_nodes,[hare@h101,rabbit@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 mnesia]#
~~~

其它端口也可以手动指定来避免冲突

> **Note:** that if you have RabbitMQ opening any ports other than AMQP, you'll need to configure those not to clash as well - for example:
>
>> $ RABBITMQ_NODE_PORT=5672 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]" RABBITMQ_NODENAME=rabbit rabbitmq-server -detached
>>
>> $ RABBITMQ_NODE_PORT=5673 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME=hare rabbitmq-server -detached
>
>will start two nodes (which can then be clustered) when the management plugin is installed.


---


## 防火墙问题

集群中的node之间可能会有防火墙阻隔，为了确保集群正常工作，node之间必须开放一些端口用于互相通信，打开 **4369** 和 **25672** 

>The case for firewalled clustered nodes exists when nodes are in a data center or on a reliable network, but separated by firewalls. Again, clustering is not recommended over a WAN or when network links between nodes are unreliable.
>
>In the most common configuration you will need to open ports 4369 and 25672 for clustering to work.
>
>Erlang makes use of a Port Mapper Daemon (epmd) for resolution of node names in a cluster. The default epmd port is 4369, but this can be changed using the ERL_EPMD_PORT environment variable. All nodes must use the same port. For further details see the Erlang epmd manpage.
>
>Once a distributed Erlang node address has been resolved via epmd, other nodes will attempt to communicate directly with that address using the Erlang distributed node protocol. The default port for this traffic in RabbitMQ is 20000 higher than RABBITMQ_NODE_PORT (i.e. 25672 by default). This can be explicitly configured using the RABBITMQ_DIST_PORT variable - see the configuration guide


---

## 集群中Erlang的版本


集群中所有node的Erlang版本必须一致

>All nodes in a cluster must run the same version of Erlang.


---

## 客户端的连接

集群中没有主备概念，每一个node都是对等的，所以客户端可以连入任意一个node进行操作，如果一个node出现故障，客户端会返回错误，直接连接另一个node就可以继续进行操作

我们可以使用第三方工具确保连接的高可用(如 keepalived)

---

## 内存节点集群

内存node是将所有元数据保存在内存中的node，是以一定安全风险为代价交换性能的选择，由于不保存数据到硬盘，所以断电或重启后数据将会丢失，也正因为不必与硬盘打交道，所以速度会非常快

一般使用它来动态地扩展集群性能(只使用RAM node的集群是脆弱的)

>RAM nodes keep their metadata only in memory. As RAM nodes don't have to write to disc as much as disc nodes, they can perform better. However, note that since persistent queue data is always stored on disc, the performance improvements will affect only resource management (e.g. adding/removing queues, exchanges, or vhosts), but not publishing or consuming speed.
>
>RAM nodes are an advanced use case; when setting up your first cluster you should simply not use them. You should have enough disc nodes to handle your redundancy requirements, then if necessary add additional RAM nodes for scale.
>
>A cluster containing only RAM nodes is fragile; if the cluster stops you will not be able to start it again and will lose all data. RabbitMQ will prevent the creation of a RAM-node-only cluster in many situations, but it can't absolutely prevent it.
>
>The examples here show a cluster with one disc and one RAM node for simplicity only; such a cluster is a poor design choice.


---

## 创建内存node


使用下面方法创建内存node

~~~
[root@h101 ~]# rabbitmqctl  -n rabbit  cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# rabbitmqctl  -n hare  cluster_status
Cluster status of node hare@h101 ...
[{nodes,[{disc,[hare@h101]}]},
 {running_nodes,[hare@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# rabbitmqctl  -n rabbit stop_app 
Stopping node rabbit@h101 ...
[root@h101 ~]# rabbitmqctl  -n rabbit join_cluster --ram hare@h101
Clustering node rabbit@h101 with hare@h101 ...
[root@h101 ~]# rabbitmqctl  -n rabbit  start_app 
Starting node rabbit@h101 ...
[root@h101 ~]# rabbitmqctl  -n rabbit  cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[hare@h101]},{ram,[rabbit@h101]}]},
 {running_nodes,[hare@h101,rabbit@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
~~~




---


## 修改node类型


一个集群中运行着的node，可以动态地切换类型

~~~
[root@h101 ~]# rabbitmqctl  -n rabbit  cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[hare@h101]},{ram,[rabbit@h101]}]},
 {running_nodes,[hare@h101,rabbit@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# rabbitmqctl  -n rabbit  stop_app 
Stopping node rabbit@h101 ...
[root@h101 ~]# rabbitmqctl  -n hare cluster_status
Cluster status of node hare@h101 ...
[{nodes,[{disc,[hare@h101]},{ram,[rabbit@h101]}]},
 {running_nodes,[hare@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# rabbitmqctl  -n rabbit  change_cluster_node_type disc
Turning rabbit@h101 into a disc node ...
[root@h101 ~]# rabbitmqctl  -n rabbit  start_app 
Starting node rabbit@h101 ...
[root@h101 ~]# rabbitmqctl  -n rabbit  cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[hare@h101,rabbit@h101]}]},
 {running_nodes,[hare@h101,rabbit@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# rabbitmqctl  -n hare stop_app
Stopping node hare@h101 ...
[root@h101 ~]# rabbitmqctl  -n hare change_cluster_node_type ram
Turning hare@h101 into a ram node ...
[root@h101 ~]# rabbitmqctl  -n hare start_app
Starting node hare@h101 ...
[root@h101 ~]# rabbitmqctl  -n rabbit  cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]},{ram,[hare@h101]}]},
 {running_nodes,[hare@h101,rabbit@h101]},
 {cluster_name,<<"hare@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
~~~

一个node 停止应用后，会对集群中剩余node的负载产生一定的影响，所以最好是在业务低峰进行以上操作

---



# 命令汇总


* rabbitmqctl  -n hare  stop_app
* rabbitmqctl  -n hare  reset 
* rabbitmqctl  -n hare  start_app
* rabbitmqctl  -n hare  status
* rabbitmqctl  -n hare  cluster_status
* rabbitmqctl  -n rabbit join_cluster --ram hare@h101
* rabbitmqctl  -n rabbit  change\_cluster\_node\_type disc
* rabbitmqctl  -n hare change\_cluster\_node\_type ram


---

[rabbitmq]:http://www.rabbitmq.com/
[rabbitdoc]:http://www.rabbitmq.com/documentation.html
[install]:http://www.rabbitmq.com/download.html
[epel]:http://fedoraproject.org/wiki/EPEL/FAQ#howtouse
[erlang]:https://www.erlang-solutions.com/downloads/download-erlang-otp
[installrepo]:https://packagecloud.io/rabbitmq/rabbitmq-server/install
[install-rpm]:http://www.rabbitmq.com/install-rpm.html
[rabbitmqctl]:http://www.rabbitmq.com/man/rabbitmqctl.1.man.html
[rabbitmq-server]:http://www.rabbitmq.com/man/rabbitmq-server.1.man.html
[cluster]:http://www.rabbitmq.com/clustering.html

