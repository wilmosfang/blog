---
layout: post
title: RabbitMQ基础 
categories: linux rabbitmq
wc: 775 2425 30842
excerpt: follow me
comments: true
---

---

#前言

**[RabbitMQ][rabbitmq]** 是一款开源的消息代理服务器，用来进行信息路由。

MQ可以使架构变得松耦合，从而更有弹性，更灵活，是SOA架构不可或缺的组成部分，担当服务总线或信息总线的角色。

下面分享一下 **[RabbitMQ][rabbitmq]** 的基础操作，详细可以参阅 [官方文档][rabbitdoc]

> **Tip:** 当前版本 **RabbitMQ 3.5.6 release**

---

#概要

* TOC
{:toc}



---

##安装

**[RabbitMQ][rabbitmq]** 是由 **[Erlang][erlang]** 语言构建的，所以先要安装 **Erlang** 


### 安装 Erlang

最方便的方式是使用 **[epel][epel]** 库

配置好了epel 软件仓库后，使用如下命令进行安装

{% highlight bash %}
[root@h102 conf]# yum install erlang.x86_64 
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.pubyun.com
 * epel: ftp.riken.jp
 * extras: centos.ustc.edu.cn
 * updates: mirrors.pubyun.com
Resolving Dependencies
--> Running transaction check
---> Package erlang.x86_64 0:R14B-04.3.el6 will be installed
--> Processing Dependency: erlang-xmerl(x86-64) = R14B-04.3.el6 for package: erlang-R14B-04.3.el6.x86_64
--> Processing Dependency: erlang-wx(x86-64) = R14B-04.3.el6 for package: erlang-R14B-04.3.el6.x86_64
--> Processing Dependency: erlang-webtool(x86-64) = R14B-04.3.el6 for package: erlang-R14B-04.3.el6.x86_64
...
...
--> Processing Dependency: erlang-asn1(x86-64) = R14B-04.3.el6 for package: erlang-R14B-04.3.el6.x86_64
--> Processing Dependency: erlang-appmon(x86-64) = R14B-04.3.el6 for package: erlang-R14B-04.3.el6.x86_64
--> Running transaction check
---> Package erlang-appmon.x86_64 0:R14B-04.3.el6 will be installed
---> Package erlang-asn1.x86_64 0:R14B-04.3.el6 will be installed
---> Package erlang-common_test.x86_64 0:R14B-04.3.el6 will be installed
...
...
--> Processing Dependency: libtcl8.5.so()(64bit) for package: 1:tk-8.5.7-5.el6.x86_64
---> Package unixODBC.x86_64 0:2.2.14-14.el6 will be installed
---> Package wxGTK-gl.x86_64 0:2.8.12-1.el6.centos will be installed
--> Running transaction check
---> Package tcl.x86_64 1:8.5.7-6.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                                 Arch                    Version                                Repository               Size
======================================================================================================================================
Installing:
 erlang                                  x86_64                  R14B-04.3.el6                          epel                     26 k
Installing for dependencies:
 erlang-appmon                           x86_64                  R14B-04.3.el6                          epel                    145 k
 erlang-asn1                             x86_64                  R14B-04.3.el6                          epel                    993 k
 erlang-common_test                      x86_64                  R14B-04.3.el6                          epel                    508 k
 ...
 ...
 tcl                                     x86_64                  1:8.5.7-6.el6                          base                    1.9 M
 tk                                      x86_64                  1:8.5.7-5.el6                          base                    1.4 M
 unixODBC                                x86_64                  2.2.14-14.el6                          base                    378 k
 wxGTK-gl                                x86_64                  2.8.12-1.el6.centos                    extras                   31 k

Transaction Summary
======================================================================================================================================
Install      62 Package(s)

Total download size: 38 M
Installed size: 72 M
Is this ok [y/N]: y
Downloading Packages:
(1/62): erlang-R14B-04.3.el6.x86_64.rpm                                                                        |  26 kB     00:00     
(2/62): erlang-appmon-R14B-04.3.el6.x86_64.rpm                                                                 | 145 kB     00:02     
(3/62): erlang-asn1-R14B-04.3.el6.x86_64.rpm                                                                   | 993 kB     00:18     
...
...
(60/62): tk-8.5.7-5.el6.x86_64.rpm                                                                             | 1.4 MB     00:00     
(61/62): unixODBC-2.2.14-14.el6.x86_64.rpm                                                                     | 378 kB     00:00     
(62/62): wxGTK-gl-2.8.12-1.el6.centos.x86_64.rpm                                                               |  31 kB     00:00     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                  76 kB/s |  38 MB     08:34     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : erlang-crypto-R14B-04.3.el6.x86_64                                                                                1/62 
  Installing : erlang-erts-R14B-04.3.el6.x86_64                                                                                  2/62 
  Installing : erlang-kernel-R14B-04.3.el6.x86_64                                                                                3/62 
...
...
  Verifying  : erlang-dialyzer-R14B-04.3.el6.x86_64                                                                             60/62 
  Verifying  : erlang-snmp-R14B-04.3.el6.x86_64                                                                                 61/62 
  Verifying  : erlang-cosEvent-R14B-04.3.el6.x86_64                                                                             62/62 

Installed:
  erlang.x86_64 0:R14B-04.3.el6                                                                                                       

Dependency Installed:
  erlang-appmon.x86_64 0:R14B-04.3.el6                              erlang-asn1.x86_64 0:R14B-04.3.el6                               
  erlang-common_test.x86_64 0:R14B-04.3.el6                         erlang-compiler.x86_64 0:R14B-04.3.el6                           
 ...
 ...                               
  erlang-xmerl.x86_64 0:R14B-04.3.el6                               tcl.x86_64 1:8.5.7-6.el6                                         
  tk.x86_64 1:8.5.7-5.el6                                           unixODBC.x86_64 0:2.2.14-14.el6                                  
  wxGTK-gl.x86_64 0:2.8.12-1.el6.centos                            

Complete!
[root@h102 conf]# echo $?
0
[root@h102 conf]# 
{% endhighlight %}

---

###安装 RabbitMQ Server

下载 **[RabbitMQ Server][install-rpm]** 安装包

{% highlight bash %}
[root@h102 ~]# wget  http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.6/rabbitmq-server-3.5.6-1.noarch.rpm 
--2015-10-21 21:57:41--  http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.6/rabbitmq-server-3.5.6-1.noarch.rpm
Resolving www.rabbitmq.com... 192.240.153.117
Connecting to www.rabbitmq.com|192.240.153.117|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4239866 (4.0M) [application/x-redhat-package-manager]
Saving to: “rabbitmq-server-3.5.6-1.noarch.rpm”

100%[==========================================================================================>] 4,239,866    540K/s   in 15s     

2015-10-21 21:57:57 (271 KB/s) - “rabbitmq-server-3.5.6-1.noarch.rpm” saved [4239866/4239866]

[root@h102 ~]#
{% endhighlight %}

---

安装 **[RabbitMQ Server][install-rpm]**

{% highlight bash %}
[root@h102 ~]# rpm -ivh rabbitmq-server-3.5.6-1.noarch.rpm  
warning: rabbitmq-server-3.5.6-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 056e8e56: NOKEY
Preparing...                ########################################### [100%]
   1:rabbitmq-server        ########################################### [100%]
[root@h102 ~]# 
{% endhighlight %}


---

##基础操作

###启动服务

{% highlight bash %}
[root@h102 ~]# /etc/init.d/rabbitmq-server  status 
Status of node rabbit@h102 ...
Error: unable to connect to node rabbit@h102: nodedown

DIAGNOSTICS
===========

attempted to contact: [rabbit@h102]

rabbit@h102:
  * connected to epmd (port 4369) on h102
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on h102
  * suggestion: start the node

current node details:
- node name: 'rabbitmq-cli-3191@h102'
- home dir: /var/lib/rabbitmq
- cookie hash: N3kEGl2Jm7amHtg0ViAg8w==

[root@h102 ~]# /etc/init.d/rabbitmq-server  start 
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@h102 ~]# /etc/init.d/rabbitmq-server  status
Status of node rabbit@h102 ...
[{pid,3416},
 {running_applications,[{rabbit,"RabbitMQ","3.5.6"},
                        {mnesia,"MNESIA  CXC 138 12","4.5"},
                        {os_mon,"CPO  CXC 138 46","2.2.7"},
                        {xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,28066048},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2704},
          {queue_procs,2704},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9268312},
          {mnesia,60144},
          {mgmt_db,0},
          {msg_index,34152},
          {other_ets,795288},
          {binary,8712},
          {code,14756074},
          {atom,1366585},
          {other_system,1771373}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,784151347},
 {disk_free_limit,50000000},
 {disk_free,26819747840},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,124}]},
 {run_queue,0},
 {uptime,10}]
[root@h102 ~]# 
{% endhighlight %}

---

###停止服务

{% highlight bash %}
[root@h102 ~]# /etc/init.d/rabbitmq-server stop 
Stopping rabbitmq-server: rabbitmq-server.
[root@h102 ~]# /etc/init.d/rabbitmq-server status
Status of node rabbit@h102 ...
Error: unable to connect to node rabbit@h102: nodedown

DIAGNOSTICS
===========

attempted to contact: [rabbit@h102]

rabbit@h102:
  * connected to epmd (port 4369) on h102
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on h102
  * suggestion: start the node

current node details:
- node name: 'rabbitmq-cli-4036@h102'
- home dir: /var/lib/rabbitmq
- cookie hash: N3kEGl2Jm7amHtg0ViAg8w==

[root@h102 ~]# 
{% endhighlight %}

---

###查看限制


使用 **cat /proc/$RABBITMQ_BEAM_PROCESS_PID/limits** 可以看到限制

{% highlight bash %}
[root@h102 rabbitmq]# ps faux | grep rabbit 
root      4627  0.0  0.0 103256   828 pts/0    S+   22:08   0:00          \_ grep rabbit
rabbitmq  2995  0.0  0.0  10828   476 ?        S    21:55   0:00 /usr/lib64/erlang/erts-5.8.5/bin/epmd -daemon
root      4251  0.0  0.0 106364  1092 pts/0    S    22:07   0:00 /bin/sh /etc/init.d/rabbitmq-server start
root      4255  0.0  0.0 106096  1176 pts/0    S    22:07   0:00  \_ /bin/bash -c ulimit -S -c 0 >/dev/null 2>&1 ; /usr/sbin/rabbitmq-server
root      4259  0.0  0.0 106096  1248 pts/0    S    22:07   0:00      \_ /bin/sh /usr/sbin/rabbitmq-server
root      4276  0.0  0.1 163856  2172 pts/0    S    22:07   0:00          \_ su rabbitmq -s /bin/sh -c /usr/lib/rabbitmq/bin/rabbitmq-server 
rabbitmq  4280  0.0  0.0 106100  1336 ?        Ss   22:07   0:00              \_ /bin/sh -e /usr/lib/rabbitmq/bin/rabbitmq-server
rabbitmq  4384  2.5  1.7 1087528 33464 ?       Sl   22:07   0:01                  \_ /usr/lib64/erlang/erts-5.8.5/bin/beam.smp -W w -A 64 -P 1048576 -K true -B i -- -root /usr/lib64/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa /usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../ebin -noshell -noinput -s rabbit boot -sname rabbit@h102 -boot start_sasl -kernel inet_default_connect_options [{nodelay,true}] -sasl errlog_type error -sasl sasl_error_logger false -rabbit error_logger {file,"/var/log/rabbitmq/rabbit@h102.log"} -rabbit sasl_error_logger {file,"/var/log/rabbitmq/rabbit@h102-sasl.log"} -rabbit enabled_plugins_file "/etc/rabbitmq/enabled_plugins" -rabbit plugins_dir "/usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../plugins" -rabbit plugins_expand_dir "/var/lib/rabbitmq/mnesia/rabbit@h102-plugins-expand" -os_mon start_cpu_sup false -os_mon start_disksup false -os_mon start_memsup false -mnesia dir "/var/lib/rabbitmq/mnesia/rabbit@h102" -kernel inet_dist_listen_min 25672 -kernel inet_dist_listen_max 25672
rabbitmq  4470  0.0  0.0  10792   508 ?        Ss   22:07   0:00                      \_ inet_gethost 4
rabbitmq  4471  0.0  0.0  12896   644 ?        S    22:07   0:00                          \_ inet_gethost 4
[root@h102 rabbitmq]# cat /proc/2995/limits 
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            10485760             unlimited            bytes     
Max core file size        0                    unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             1024                 14779                processes 
Max open files            1024                 4096                 files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       14779                14779                signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us        
[root@h102 rabbitmq]#
{% endhighlight %}

查看端口运行情况

{% highlight bash %}
[root@h102 rabbitmq]# netstat  -an | grep -E "(4369|25672|5672|5671|15672|61613|61614|1883|8883)"
tcp        0      0 0.0.0.0:4369                0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:25672               0.0.0.0:*                   LISTEN      
tcp        0      0 127.0.0.1:49602             127.0.0.1:4369              ESTABLISHED 
tcp        0      0 127.0.0.1:4369              127.0.0.1:49602             ESTABLISHED 
tcp        0      0 192.168.100.102:4369        192.168.100.102:52333       TIME_WAIT   
tcp        0      0 :::5672                     :::*                        LISTEN      
[root@h102 rabbitmq]# 
{% endhighlight %}

 > **Tip:** 在 **/etc/security/limits.conf** 中修改系统的文件句柄数，这个参数对会产生大量连接，需要打开很多文件的服务有制约作用，系统的默认1024比较保守，可以满足开发需求，但无法满足生产需求


---

##rabbitmqctl基础操作

日常管理主要使用 **[rabbitmqctl][rabbitmqctl]**  

###关闭node

{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  stop 
Stopping and halting node rabbit@h102 ...
[root@h102 rabbitmq]# rabbitmqctl  status
Status of node rabbit@h102 ...
Error: unable to connect to node rabbit@h102: nodedown

DIAGNOSTICS
===========

attempted to contact: [rabbit@h102]

rabbit@h102:
  * connected to epmd (port 4369) on h102
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on h102
  * suggestion: start the node

current node details:
- node name: 'rabbitmq-cli-5175@h102'
- home dir: /var/lib/rabbitmq
- cookie hash: N3kEGl2Jm7amHtg0ViAg8w==

[root@h102 rabbitmq]# 
{% endhighlight %}

>This command instructs the RabbitMQ node to terminate.

---

###关闭RabbitMQ 应用

{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  status
Status of node rabbit@h102 ...
[{pid,5596},
 {running_applications,[{rabbit,"RabbitMQ","3.5.6"},
                        {os_mon,"CPO  CXC 138 46","2.2.7"},
                        {xmerl,"XML parser","1.2.10"},
                        {mnesia,"MNESIA  CXC 138 12","4.5"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,28114136},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2704},
          {queue_procs,2704},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9318008},
          {mnesia,59584},
          {mgmt_db,0},
          {msg_index,34088},
          {other_ets,790424},
          {binary,11136},
          {code,14756074},
          {atom,1366585},
          {other_system,1772829}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,784151347},
 {disk_free_limit,50000000},
 {disk_free,26749317120},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,124}]},
 {run_queue,0},
 {uptime,13}]
[root@h102 rabbitmq]# rabbitmqctl  stop_app
Stopping node rabbit@h102 ...
[root@h102 rabbitmq]#
[root@h102 rabbitmq]# ps faux | grep -i mq 
root      5917  0.0  0.0 103256   844 pts/0    S+   16:38   0:00          \_ grep -i mq
rabbitmq  4169  0.0  0.0  10832   412 ?        S    16:00   0:00 /usr/lib64/erlang/erts-5.8.5/bin/epmd -daemon
root      5462  0.0  0.0 106364  1092 pts/0    S    16:37   0:00 /bin/sh /etc/init.d/rabbitmq-server start
root      5466  0.0  0.0 106096  1176 pts/0    S    16:37   0:00  \_ /bin/bash -c ulimit -S -c 0 >/dev/null 2>&1 ; /usr/sbin/rabbitmq-server
root      5470  0.0  0.0 106096  1244 pts/0    S    16:37   0:00      \_ /bin/sh /usr/sbin/rabbitmq-server
root      5487  0.0  0.1 163856  2168 pts/0    S    16:37   0:00          \_ su rabbitmq -s /bin/sh -c /usr/lib/rabbitmq/bin/rabbitmq-server 
rabbitmq  5491  0.0  0.0 106100  1336 ?        Ss   16:37   0:00              \_ /bin/sh -e /usr/lib/rabbitmq/bin/rabbitmq-server
rabbitmq  5596  1.9  1.6 1086600 32388 ?       Sl   16:37   0:01                  \_ /usr/lib64/erlang/erts-5.8.5/bin/beam.smp -W w -A 64 -P 1048576 -K true -B i -- -root /usr/lib64/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa /usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../ebin -noshell -noinput -s rabbit boot -sname rabbit@h102 -boot start_sasl -kernel inet_default_connect_options [{nodelay,true}] -sasl errlog_type error -sasl sasl_error_logger false -rabbit error_logger {file,"/var/log/rabbitmq/rabbit@h102.log"} -rabbit sasl_error_logger {file,"/var/log/rabbitmq/rabbit@h102-sasl.log"} -rabbit enabled_plugins_file "/etc/rabbitmq/enabled_plugins" -rabbit plugins_dir "/usr/lib/rabbitmq/lib/rabbitmq_server-3.5.6/sbin/../plugins" -rabbit plugins_expand_dir "/var/lib/rabbitmq/mnesia/rabbit@h102-plugins-expand" -os_mon start_cpu_sup false -os_mon start_disksup false -os_mon start_memsup false -mnesia dir "/var/lib/rabbitmq/mnesia/rabbit@h102" -kernel inet_dist_listen_min 25672 -kernel inet_dist_listen_max 25672
rabbitmq  5686  0.0  0.0  10792   496 ?        Ss   16:37   0:00                      \_ inet_gethost 4
rabbitmq  5687  0.0  0.0  12896   624 ?        S    16:37   0:00                          \_ inet_gethost 4
[root@h102 rabbitmq]# rabbitmqctl  status
Status of node rabbit@h102 ...
[{pid,5596},
 {running_applications,[{xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,27448088},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,0},
          {queue_procs,0},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,8902024},
          {mnesia,0},
          {mgmt_db,0},
          {msg_index,0},
          {other_ets,643344},
          {binary,3992},
          {code,14756074},
          {atom,1366585},
          {other_system,1776069}]},
 {alarms,[]},
 {listeners,[]},
 {processes,[{limit,1048576},{used,46}]},
 {run_queue,0},
 {uptime,89}]
[root@h102 rabbitmq]# 
{% endhighlight %}

节点还在运行，应用已经停了

注意到 **{listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]}** 变成了 **{listeners,[]}** , 内存磁盘还有文件句柄部分也消失了

>This command instructs the RabbitMQ node to stop the RabbitMQ application.

日志信息

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::16:52:30 ===
Stopping RabbitMQ

=INFO REPORT==== 23-Oct-2015::16:52:30 ===
stopped TCP Listener on [::]:5672

=INFO REPORT==== 23-Oct-2015::16:52:30 ===
Stopped RabbitMQ application
{% endhighlight %}

> **Tip:** 使用 **tail -f  /var/log/rabbitmq/rabbit@h102.log** 查看日志

{% highlight bash %}
[root@h102 rabbitmq]# netstat  -ant | grep 5672
tcp        0      0 0.0.0.0:25672               0.0.0.0:*                   LISTEN      
tcp        0      0 192.168.100.102:54922       192.168.100.102:25672       TIME_WAIT   
[root@h102 rabbitmq]#
{% endhighlight %}

---

###开启RabbitMQ 应用


{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  status
Status of node rabbit@h102 ...
[{pid,5596},
 {running_applications,[{xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,27465424},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,0},
          {queue_procs,0},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,8917816},
          {mnesia,0},
          {mgmt_db,0},
          {msg_index,0},
          {other_ets,643344},
          {binary,3992},
          {code,14756074},
          {atom,1367393},
          {other_system,1776805}]},
 {alarms,[]},
 {listeners,[]},
 {processes,[{limit,1048576},{used,46}]},
 {run_queue,0},
 {uptime,420}]
[root@h102 rabbitmq]# rabbitmqctl  start_app 
Starting node rabbit@h102 ...
[root@h102 rabbitmq]# rabbitmqctl  status
Status of node rabbit@h102 ...
[{pid,5596},
 {running_applications,[{rabbit,"RabbitMQ","3.5.6"},
                        {os_mon,"CPO  CXC 138 46","2.2.7"},
                        {mnesia,"MNESIA  CXC 138 12","4.5"},
                        {xmerl,"XML parser","1.2.10"},
                        {sasl,"SASL  CXC 138 11","2.1.10"},
                        {stdlib,"ERTS  CXC 138 10","1.17.5"},
                        {kernel,"ERTS  CXC 138 10","2.14.5"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,28067792},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2704},
          {queue_procs,2704},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,9270424},
          {mnesia,59584},
          {mgmt_db,0},
          {msg_index,34120},
          {other_ets,790272},
          {binary,11136},
          {code,14756074},
          {atom,1367393},
          {other_system,1773381}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,784151347},
 {disk_free_limit,50000000},
 {disk_free,26749280256},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,124}]},
 {run_queue,0},
 {uptime,516}]
[root@h102 rabbitmq]# 
{% endhighlight %}

>This command instructs the RabbitMQ node to start the RabbitMQ application.

日志信息


{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::16:52:50 ===
Starting RabbitMQ 3.5.6 on Erlang R14B04
Copyright (C) 2007-2015 Pivotal Software, Inc.
Licensed under the MPL.  See http://www.rabbitmq.com/

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
node           : rabbit@h102
home dir       : /var/lib/rabbitmq
config file(s) : /etc/rabbitmq/rabbitmq.config (not found)
cookie hash    : N3kEGl2Jm7amHtg0ViAg8w==
log            : /var/log/rabbitmq/rabbit@h102.log
sasl log       : /var/log/rabbitmq/rabbit@h102-sasl.log
database dir   : /var/lib/rabbitmq/mnesia/rabbit@h102

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
Memory limit set to 747MB of 1869MB total.

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
Disk free limit set to 50MB

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
Limiting to approx 924 file handles (829 sockets)

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
FHC read buffering:  ON
FHC write buffering: ON

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
msg_store_transient: using rabbit_msg_store_ets_index to provide index

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
msg_store_persistent: using rabbit_msg_store_ets_index to provide index

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
started TCP Listener on [::]:5672

=INFO REPORT==== 23-Oct-2015::16:52:50 ===
Server startup complete; 0 plugins started.
{% endhighlight %}



{% highlight bash %}
[root@h102 rabbitmq]# netstat  -ant | grep 5672
tcp        0      0 0.0.0.0:25672               0.0.0.0:*                   LISTEN      
tcp        0      0 192.168.100.102:39082       192.168.100.102:25672       TIME_WAIT   
tcp        0      0 :::5672                     :::*                        LISTEN      
[root@h102 rabbitmq]# 
{% endhighlight %}

---

###重置node

{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  reset 
Resetting node rabbit@h102 ...
[root@h102 rabbitmq]# 
{% endhighlight %}

让节点恢复到初始状态(原文是返回处女状态 )

>This command resets the RabbitMQ node . Return a RabbitMQ node to its virgin state

必须先停掉RabbitMQ应用，才能成功执行,否则会报错

{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  reset
Resetting node rabbit@h102 ...
Error: mnesia_unexpectedly_running
[root@h102 rabbitmq]# echo $?
2
[root@h102 rabbitmq]# 
{% endhighlight %}

停掉后，可以正常执行

{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  stop_app
Stopping node rabbit@h102 ...
[root@h102 rabbitmq]# rabbitmqctl  reset
Resetting node rabbit@h102 ...
[root@h102 rabbitmq]# 
{% endhighlight %}

日志

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::17:08:47 ===
Resetting Rabbit
{% endhighlight %}


---

###强制重置node


也必须先停掉应用，否则无法成功

{% highlight bash %}
[root@h102 rabbitmq]# rabbitmqctl  start_app
Starting node rabbit@h102 ...
[root@h102 rabbitmq]# rabbitmqctl  force_reset
Forcefully resetting node rabbit@h102 ...
Error: mnesia_unexpectedly_running
[root@h102 rabbitmq]# rabbitmqctl  stop_app
Stopping node rabbit@h102 ...
[root@h102 rabbitmq]# rabbitmqctl  force_reset
Forcefully resetting node rabbit@h102 ...
[root@h102 rabbitmq]# 
{% endhighlight %}

日志

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::17:11:19 ===
Resetting Rabbit forcefully
{% endhighlight %}


---

###轮转日志 

当前状态

{% highlight bash %}
[root@h102 ~]# ll /var/log/rabbitmq/
total 24
-rw-r--r-- 1 rabbitmq rabbitmq 15807 Oct 23 17:11 rabbit@h102.log
-rw-r--r-- 1 rabbitmq rabbitmq     0 Oct 21 21:58 rabbit@h102-sasl.log
-rw-r--r-- 1 root     root         0 Oct 21 22:18 shutdown_err
-rw-r--r-- 1 root     root        42 Oct 21 22:18 shutdown_log
-rw-r--r-- 1 root     root         0 Oct 23 16:37 startup_err
-rw-r--r-- 1 root     root      2040 Oct 23 17:11 startup_log
[root@h102 ~]# 
{% endhighlight %}

轮转日志 

{% highlight bash %}
[root@h102 ~]# rabbitmqctl  rotate_logs .1
Rotating logs to files with suffix ".1" ...
[root@h102 ~]# 
{% endhighlight %}

日志

{% highlight bash %}
=INFO REPORT==== 23-Oct-2015::17:20:23 ===
Rotating logs with suffix '.1'
{% endhighlight %}

之后的状态

{% highlight bash %}
[root@h102 ~]# ll /var/log/rabbitmq/
total 24
-rw-r--r-- 1 rabbitmq rabbitmq     0 Oct 23 17:20 rabbit@h102.log
-rw-r--r-- 1 rabbitmq rabbitmq 15882 Oct 23 17:20 rabbit@h102.log.1
-rw-r--r-- 1 rabbitmq rabbitmq     0 Oct 23 17:20 rabbit@h102-sasl.log
-rw-r--r-- 1 rabbitmq rabbitmq     0 Oct 23 17:20 rabbit@h102-sasl.log.1
-rw-r--r-- 1 root     root         0 Oct 21 22:18 shutdown_err
-rw-r--r-- 1 root     root        42 Oct 21 22:18 shutdown_log
-rw-r--r-- 1 root     root         0 Oct 23 16:37 startup_err
-rw-r--r-- 1 root     root      2040 Oct 23 17:11 startup_log
[root@h102 ~]# 
{% endhighlight %}

发现原来的日志已经加上了 **.1** 的后缀

>Instruct the RabbitMQ node to rotate the log files.




---

[rabbitmq]:http://www.rabbitmq.com/
[rabbitdoc]:http://www.rabbitmq.com/documentation.html
[install]:http://www.rabbitmq.com/download.html
[epel]:http://fedoraproject.org/wiki/EPEL/FAQ#howtouse
[erlang]:https://www.erlang-solutions.com/downloads/download-erlang-otp
[installrepo]:https://packagecloud.io/rabbitmq/rabbitmq-server/install
[install-rpm]:http://www.rabbitmq.com/install-rpm.html
[rabbitmqctl]:http://www.rabbitmq.com/man/rabbitmqctl.1.man.html


