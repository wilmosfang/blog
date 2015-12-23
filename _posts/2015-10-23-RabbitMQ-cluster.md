---
layout: post
title: RabbitMQ集群I
categories: linux rabbitmq cluster
wc: 541 1398 16216
excerpt: follow me
comments: true
---


---

#前言

**[RabbitMQ][rabbitmq]** 是一款开源的消息代理服务器，用来进行信息路由。

MQ可以使架构变得松耦合，从而更有弹性，更灵活，是SOA架构不可或缺的组成部分，担当服务总线或信息总线的角色。

可用性在生产系统中是非常重要的指标， **[RabbitMQ][rabbitmq]** 对集群进行了很好的支持

下面分享一下 **[RabbitMQ][rabbitmq]** 的集群操作，详细可以参阅 [官方文档][rabbitdoc]

> **Tip:** 当前版本 **RabbitMQ 3.5.6 release**

---

#概要

* TOC
{:toc}




---

##准备备用节点


具体安装过程可以参考 **[RabbitMQ安装][install]** 

先安装 **[epel][epel]** 库，然后按照下面方式安装 **[RabbitMQ][rabbitmq]**

{% highlight bash %}
yum install erlang.x86_64  
wget  http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.6/rabbitmq-server-3.5.6-1.noarch.rpm
rpm -ivh rabbitmq-server-3.5.6-1.noarch.rpm 

{% endhighlight %}

##启动节点

{% highlight bash %}
[root@h101 ~]# /etc/init.d/rabbitmq-server start 
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@h101 ~]# /etc/init.d/rabbitmq-server status
Status of node rabbit@h101 ...
[{pid,7593},
 {running_applications,[{rabbit,"RabbitMQ","3.5.6"},
                        {mnesia,"MNESIA  CXC 138 12","4.5"},
                        {os_mon,"CPO  CXC 138 46","2.2.7"},
                        {xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,28062816},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2704},
          {queue_procs,2704},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9265232},
          {mnesia,60144},
          {mgmt_db,0},
          {msg_index,34152},
          {other_ets,795152},
          {binary,8712},
          {code,14756074},
          {atom,1366585},
          {other_system,1771357}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,1604293427},
 {disk_free_limit,50000000},
 {disk_free,43576463360},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,124}]},
 {run_queue,0},
 {uptime,6}]
[root@h101 ~]#
{% endhighlight %}


---

###还有两种启动方式

####前台启动


{% highlight bash %}
[root@h102 ~]# rabbitmq-server  

              RabbitMQ 3.5.6. Copyright (C) 2007-2015 Pivotal Software, Inc.
  ##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
  ##  ##
  ##########  Logs: /var/log/rabbitmq/rabbit@h102.log
  ######  ##        /var/log/rabbitmq/rabbit@h102-sasl.log
  ##########
              Starting broker... completed with 0 plugins.
              
              
{% endhighlight %}

如果当前窗口终结，则这个服务会退出

####后台启动


{% highlight bash %}
[root@h102 ~]# rabbitmq-server  -detached 
Warning: PID file not written; -detached was passed.
[root@h102 ~]# echo $?
0
[root@h102 ~]# 
{% endhighlight %}

详细可参考 **[rabbitmq-server][rabbitmq-server]** 使用方法


---


##同步 Erlang cookie


集群中node必须使用相同的cookie才能相互通讯

在Linux中cookie的位置一般在 **/var/lib/rabbitmq/.erlang.cookie**

{% highlight bash %}
[root@h101 ~]# scp  /var/lib/rabbitmq/.erlang.cookie  root@h102:/var/lib/rabbitmq/.erlang.cookie
The authenticity of host 'h102 (192.168.100.102)' can't be established.
RSA key fingerprint is 78:c4:6f:3f:08:43:d1:2a:02:bf:ec:f3:9f:e3:89:76.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'h102,192.168.100.102' (RSA) to the list of known hosts.
root@h102's password: 
.erlang.cookie                                                                                      100%   20     0.0KB/s   00:00    
[root@h101 ~]# 
{% endhighlight %}


>Erlang nodes use a cookie to determine whether they are allowed to communicate with each other - for two nodes to be able to communicate they must have the same cookie. The cookie is just a string of alphanumeric characters. It can be as long or short as you like. Every cluster node must have the same cookie. The cookie is also used for tools such as rabbitmqctl and rabbitmq-plugins.
>
>Erlang will automatically create a random cookie file when the RabbitMQ server starts up. The easiest way to proceed is to allow one node to create the file, and then copy it to all the other nodes in the cluster.
>
>On Unix systems, the cookie will be typically located in /var/lib/rabbitmq/.erlang.cookie or $HOME/.erlang.cookie.



---

##查看集群状态


{% highlight bash %}
[root@h101 ~]# rabbitmqctl  cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
[root@h102 ~]# rabbitmqctl  cluster_status
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h102]}]},
 {running_nodes,[rabbit@h102]},
 {cluster_name,<<"rabbit@h102.temp">>},
 {partitions,[]}]
[root@h102 ~]# 
{% endhighlight %}

---

##创建集群

首先关掉本地应用 **rabbitmqctl  stop_app**

{% highlight bash %}
[root@h102 ~]# /etc/init.d/rabbitmq-server stop 
Stopping rabbitmq-server: RabbitMQ is not running
rabbitmq-server.
[root@h102 ~]# /etc/init.d/rabbitmq-server start 
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@h102 ~]# rabbitmqctl  stop_app
Stopping node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl  status
Status of node rabbit@h102 ...
[{pid,16425},
 {running_applications,[{xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,27691920},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,0},
          {queue_procs,0},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9143408},
          {mnesia,0},
          {mgmt_db,0},
          {msg_index,0},
          {other_ets,643384},
          {binary,4672},
          {code,14756074},
          {atom,1366585},
          {other_system,1777797}]},
 {alarms,[]},
 {listeners,[]},
 {processes,[{limit,1048576},{used,46}]},
 {run_queue,0},
 {uptime,21}]
[root@h102 ~]# 
{% endhighlight %}

加入集群 **rabbitmqctl join_cluster  rabbit@h101**

{% highlight bash %}
[root@h102 ~]# rabbitmqctl join_cluster  rabbit@h101
Clustering node rabbit@h102 with rabbit@h101 ...
[root@h102 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]}]
[root@h101 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
{% endhighlight %}

启动应用 **rabbitmqctl  start_app**

{% highlight bash %}
[root@h102 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]}]
[root@h102 ~]# rabbitmqctl  start_app
Starting node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h101,rabbit@h102]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h102 ~]# 
[root@h101 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h102,rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
{% endhighlight %}

更多node的加入也是使用相同的办法，并且集群中node是平等的，新node可以选择任意一个节点加入


加入集群分三步

* 1 停应用
* 2 加入集群
* 3 启应用

{% highlight bash %}
[root@h102 ~]# rabbitmqctl  stop_app 
Stopping node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl join_cluster  rabbit@h101
Clustering node rabbit@h102 with rabbit@h101 ...
[root@h102 ~]# rabbitmqctl  start_app 
Starting node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl cluster_status  
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h101,rabbit@h102]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h102 ~]# 
{% endhighlight %}

---

##重启集群node

加入集群的节点可以任意关停、下线或宕机

>Nodes that have been joined to a cluster can be stopped at any time. It is also ok for them to crash. In both cases the rest of the cluster continues operating unaffected, and the nodes automatically "catch up" with the other cluster nodes when they start up again.

{% highlight bash %}
[root@h101 ~]# rabbitmqctl  stop 
Stopping and halting node rabbit@h101 ...
[root@h101 ~]# 
{% endhighlight %}

h101上日志

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::22:19:33 ===
Stopping RabbitMQ

=INFO REPORT==== 23-Oct-2015::22:19:33 ===
stopped TCP Listener on [::]:5672

=INFO REPORT==== 23-Oct-2015::22:19:33 ===
Stopped RabbitMQ application

=INFO REPORT==== 23-Oct-2015::22:19:33 ===
Halting Erlang VM
{% endhighlight %}

h102上日志 

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::22:19:33 ===
rabbit on node rabbit@h101 down

=INFO REPORT==== 23-Oct-2015::22:19:33 ===
Keep rabbit@h101 listeners: the node is already back

=INFO REPORT==== 23-Oct-2015::22:19:34 ===
node rabbit@h101 down: connection_closed
{% endhighlight %}

h102上集群状态

{% highlight bash %}
[root@h102 ~]# rabbitmqctl cluster_status 
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h102]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h102 ~]# 
{% endhighlight %}

启动h101

{% highlight bash %}
[root@h101 ~]# /etc/init.d/rabbitmq-server start 
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@h101 ~]#
{% endhighlight %}

h102上日志

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::22:26:04 ===
node rabbit@h101 up

=INFO REPORT==== 23-Oct-2015::22:26:05 ===
rabbit on node rabbit@h101 up
{% endhighlight %}

集群状态

{% highlight bash %}
[root@h102 ~]# rabbitmqctl cluster_status 
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h101,rabbit@h102]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h102 ~]# 
[root@h101 ~]#  rabbitmqctl cluster_status 
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h102,rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
{% endhighlight %}

> **Tip:** There are some important caveats:
>
>When the entire cluster is brought down, the last node to go down must be the first node to be brought online. If this doesn't happen, the nodes will wait 30 seconds for the last disc node to come back online, and fail afterwards. If the last node to go offline cannot be brought back up, it can be removed from the cluster using the forget_cluster_node command - consult the rabbitmqctl manpage for more information.
>
>If all cluster nodes stop in a simultaneous and uncontrolled manner (for example with a power cut) you can be left with a situation in which all nodes think that some other node stopped after them. In this case you can use the force_boot command on one node to make it bootable again - consult the rabbitmqctl manpage for more information.

---

##拆分集群


###本地退出

有时我们有必要从集群中移出某个node，分三步

* 1 停应用 
* 2 重置node
* 3 起应用

{% highlight bash %}
[root@h102 ~]# rabbitmqctl cluster_status 
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h101,rabbit@h102]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h102 ~]# rabbitmqctl stop_app 
Stopping node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl reset
Resetting node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl start_app
Starting node rabbit@h102 ...
[root@h102 ~]# rabbitmqctl cluster_status 
Cluster status of node rabbit@h102 ...
[{nodes,[{disc,[rabbit@h102]}]},
 {running_nodes,[rabbit@h102]},
 {cluster_name,<<"rabbit@h102.temp">>},
 {partitions,[]}]
[root@h102 ~]# 
{% endhighlight %}


h101上的集群状态

{% highlight bash %}
[root@h101 ~]#  rabbitmqctl cluster_status 
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h102,rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]#  rabbitmqctl cluster_status 
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101,rabbit@h102]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]#  rabbitmqctl cluster_status 
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
{% endhighlight %}

###远程剔除

有些不响应的node,一段时间里无法交互，可以远程的方式，从集群中剔除

停掉h101上应用，模拟不响应

{% highlight bash %}
[root@h101 ~]# rabbitmqctl stop_app
Stopping node rabbit@h101 ...
[root@h101 ~]# 
{% endhighlight %}

从h102上对h101进行剔除

{% highlight bash %}
[root@h102 ~]# rabbitmqctl forget_cluster_node rabbit@h101
Removing node rabbit@h101 from cluster ...
[root@h102 ~]# 
{% endhighlight %}

此时h102上信息已经一致了，但是h101还认为自己是集群的一部分，一旦它的应用恢复，它会尝试与集群联络，从而产生报错


{% highlight bash %}
[root@h101 ~]# rabbitmqctl start_app
Starting node rabbit@h101 ...


BOOT FAILED
===========

Error description:
   {error,{inconsistent_cluster,"Node rabbit@h101 thinks it's clustered with node rabbit@h102, but rabbit@h102 disagrees"}}

Log files (may contain more information):
   /var/log/rabbitmq/rabbit@h101.log
   /var/log/rabbitmq/rabbit@h101-sasl.log

Stack trace:
   [{rabbit_mnesia,check_cluster_consistency,0},
    {rabbit,'-start/0-fun-0-',0},
    {rabbit,start_it,1},
    {rpc,'-handle_call_call/6-fun-0-',5}]

Error: {error,{inconsistent_cluster,"Node rabbit@h101 thinks it's clustered with node rabbit@h102, but rabbit@h102 disagrees"}}
[root@h101 ~]#
{% endhighlight %}

解决办法是一旦有机会连上h101 ,就对它进行重置

{% highlight bash %}
[root@h101 ~]# rabbitmqctl reset
Resetting node rabbit@h101 ...
[root@h101 ~]# rabbitmqctl start_app 
Starting node rabbit@h101 ...
[root@h101 ~]# rabbitmqctl cluster_status  
Cluster status of node rabbit@h101 ...
[{nodes,[{disc,[rabbit@h101]}]},
 {running_nodes,[rabbit@h101]},
 {cluster_name,<<"rabbit@h101.temp">>},
 {partitions,[]}]
[root@h101 ~]# 
{% endhighlight %}



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
