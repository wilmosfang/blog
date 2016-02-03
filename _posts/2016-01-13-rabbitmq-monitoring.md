---
layout: post
title:  RabbitMQ 监控
categories:  linux network nginx rabbitmq monitoring 
wc: 789  2321 28166 
excerpt:  RabbitMQ  的rabbitmq-management管理插件，https安全防护，基本认证，DNAT，访问控制，集群创建和web GUI监控方法 
comments: true
---



# 前言


**[RabbitMQ ][rabbitmq]** 有灵活的插件机制，启用 **[rabbitmq-management][rabbitmq_management]** 就可以对服务进行监控和管理

**[RabbitMQ ][rabbitmq]** 监控管理是基于 **HTTP API** 的 **WEB GUI** 服务，默认开放在 **15672** 端口，它可以实现以下功能：

* 声明显示和删除 exchanges, queues, bindings, users, virtual hosts and permissions.
* 监控队列长度, 全局和通道上的消息速率, 连接的数据率
* 发送和接收 messages.
* 监控Erlang 进程, 文件描述符, 内存使用情况.
* 导入导出对象的定义到 JSON.
* 强制关闭连接, 清空队列.

下面分享一下 **[RabbitMQ ][rabbitmq]** 监控的基础操作，详细可以参阅 **[官方文档][rabbitmq_management]**

> **Tip:** 当前的最新版本为 **RabbitMQ 3.6.0 release** ,  但是示例是 **RabbitMQ 3.5.6 release**

---


# 概要

* TOC
{:toc}


---

## 启用插件


RabbitMQ 有插件机制，从而可以动态灵活地扩展功能和特性

主要是通过 **rabbitmq-plugins** 来管理的

{% highlight bash %}
[root@rabbitmq ~]# rabbitmq-plugins -h
Error: could not recognise command
Usage:
rabbitmq-plugins [-n <node>] <command> [<command options>] 

Commands:
    list [-v] [-m] [-E] [-e] [<pattern>]
    enable [--offline] [--online] <plugin> ...
    disable [--offline] [--online] <plugin> ...
    set [--offline] [--online] <plugin> ...


[root@rabbitmq ~]# 
{% endhighlight %}

启用 **rabbitmq_management** 插件

{% highlight bash %}
[root@rabbitmq ~]# rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status:   * = running on rabbit@rabbitmq
 |/
[  ] amqp_client                       3.5.6
[  ] cowboy                            0.5.0-rmq3.5.6-git4b93c2d
[  ] eldap                             3.5.6-gite309de4
[  ] mochiweb                          2.7.0-rmq3.5.6-git680dba8
[  ] rabbitmq_amqp1_0                  3.5.6
[  ] rabbitmq_auth_backend_ldap        3.5.6
[  ] rabbitmq_auth_mechanism_ssl       3.5.6
[  ] rabbitmq_consistent_hash_exchange 3.5.6
[  ] rabbitmq_federation               3.5.6
[  ] rabbitmq_federation_management    3.5.6
[  ] rabbitmq_management               3.5.6
[  ] rabbitmq_management_agent         3.5.6
[  ] rabbitmq_management_visualiser    3.5.6
[  ] rabbitmq_mqtt                     3.5.6
[  ] rabbitmq_shovel                   3.5.6
[  ] rabbitmq_shovel_management        3.5.6
[  ] rabbitmq_stomp                    3.5.6
[  ] rabbitmq_test                     3.5.6
[  ] rabbitmq_tracing                  3.5.6
[  ] rabbitmq_web_dispatch             3.5.6
[  ] rabbitmq_web_stomp                3.5.6
[  ] rabbitmq_web_stomp_examples       3.5.6
[  ] sockjs                            0.3.4-rmq3.5.6-git3132eb9
[  ] webmachine                        1.10.3-rmq3.5.6-gite9359c7
[root@rabbitmq ~]# netstat  -ant | grep 15672
[root@rabbitmq ~]# rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@rabbitmq... started 6 plugins.
[root@rabbitmq ~]# rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status:   * = running on rabbit@rabbitmq
 |/
[e*] amqp_client                       3.5.6
[  ] cowboy                            0.5.0-rmq3.5.6-git4b93c2d
[  ] eldap                             3.5.6-gite309de4
[e*] mochiweb                          2.7.0-rmq3.5.6-git680dba8
[  ] rabbitmq_amqp1_0                  3.5.6
[  ] rabbitmq_auth_backend_ldap        3.5.6
[  ] rabbitmq_auth_mechanism_ssl       3.5.6
[  ] rabbitmq_consistent_hash_exchange 3.5.6
[  ] rabbitmq_federation               3.5.6
[  ] rabbitmq_federation_management    3.5.6
[E*] rabbitmq_management               3.5.6
[e*] rabbitmq_management_agent         3.5.6
[  ] rabbitmq_management_visualiser    3.5.6
[  ] rabbitmq_mqtt                     3.5.6
[  ] rabbitmq_shovel                   3.5.6
[  ] rabbitmq_shovel_management        3.5.6
[  ] rabbitmq_stomp                    3.5.6
[  ] rabbitmq_test                     3.5.6
[  ] rabbitmq_tracing                  3.5.6
[e*] rabbitmq_web_dispatch             3.5.6
[  ] rabbitmq_web_stomp                3.5.6
[  ] rabbitmq_web_stomp_examples       3.5.6
[  ] sockjs                            0.3.4-rmq3.5.6-git3132eb9
[e*] webmachine                        1.10.3-rmq3.5.6-gite9359c7
[root@rabbitmq ~]# netstat  -ant | grep 15672
tcp        0      0 0.0.0.0:15672               0.0.0.0:*                   LISTEN      
[root@rabbitmq ~]# curl http://localhost:15672
<html>
  <head>
    <title>RabbitMQ Management</title>
    <script src="js/ejs.min.js" type="text/javascript"></script>
    <script src="js/jquery-1.6.4.min.js" type="text/javascript"></script>
    <script src="js/jquery.flot.min.js" type="text/javascript"></script>
    <script src="js/jquery.flot.time.min.js" type="text/javascript"></script>
    <script src="js/sammy-0.6.0.min.js" type="text/javascript"></script>
    <script src="js/json2.js" type="text/javascript"></script>
    <script src="js/base64.js" type="text/javascript"></script>
    <script src="js/global.js" type="text/javascript"></script>
    <script src="js/main.js" type="text/javascript"></script>
    <script src="js/prefs.js" type="text/javascript"></script>
    <script src="js/help.js" type="text/javascript"></script>
    <script src="js/formatters.js" type="text/javascript"></script>
    <script src="js/charts.js" type="text/javascript"></script>

    <link href="css/main.css" rel="stylesheet" type="text/css"/>
    <link href="favicon.ico" rel="shortcut icon" type="image/x-icon"/>

<!--[if lte IE 8]>
    <script src="js/excanvas.min.js" type="text/javascript"></script>
    <link href="css/evil.css" rel="stylesheet" type="text/css"/>
<![endif]-->
  </head>
  <body>
    <div id="outer"></div>
    <div id="debug"></div>
    <div id="scratch"></div>
  </body>
</html>
[root@rabbitmq ~]#
{% endhighlight %}

---

## https安全防护

### 创建认证密码

{% highlight bash %}
[nginx@new-mq-node pass]$ pwd
/usr/local/nginx/pass
[nginx@new-mq-node pass]$ perl -e 'print  crypt(mqpass,mqpass)'
mqdhK69Oo2JQA[nginx@new-mq-node pass]$ vim mq.passwd
[nginx@new-mq-node pass]$ cat mq.passwd 
mqmonitor:mqdhK69Oo2JQA
[nginx@new-mq-node pass]$ 
{% endhighlight %}


> **Note:** 这里的认证密码要和mq中监控用户的一样，否则第一次正确输入后可以看到MQ的认证窗口，但在第二次认证过程中，nginx会到自己的基本认证文件中去找对应用户从而导致跳转失败，可以在错误日志中看到对应信息


### 修改nginx配置


{% highlight bash %}
[nginx@new-mq-node conf]$ tail nginx.conf | grep include
    include apps/mq.conf;
[nginx@new-mq-node conf]$ cat apps/mq.conf 
    upstream mq {
        server 192.168.66.123:15672;
	}

    server {
        listen      1443; 
        server_name  localhost;

        ssl on;
	ssl_certificate /usr/local/nginx/cert/es.crt;
	ssl_certificate_key /usr/local/nginx/cert/es.key;

        location / {
            root   html;
            index  index.html index.htm;
            auth_basic      "input your name and passsword for mq";
	    auth_basic_user_file  /usr/local/nginx/pass/mq.passwd;
	    allow all;
	    proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://mq;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
[nginx@new-mq-node conf]$
{% endhighlight %}



---

### 打开防火墙


打开本地(RabbitMQ Server)防火墙

{% highlight bash %}
[root@rabbitmq ~]# netstat  -ant | grep 15672
tcp        0      0 0.0.0.0:15672               0.0.0.0:*                   LISTEN      
[root@rabbitmq ~]# iptables -L -nv | grep 15672
[root@rabbitmq ~]# grep 15672 /etc/sysconfig/iptables
[root@rabbitmq ~]# vim /etc/sysconfig/iptables
[root@rabbitmq ~]# grep 15672 /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 15672 -j ACCEPT
[root@rabbitmq ~]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@rabbitmq ~]# iptables -L -nv | grep 15672
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:15672 
[root@rabbitmq ~]# 
{% endhighlight %}

打开nginx server防火墙

{% highlight bash %}
[root@new-mq-node nginx]# iptables -L -nv | grep 1443
[root@new-mq-node nginx]# vim /etc/sysconfig/iptables
[root@new-mq-node nginx]# grep 1443 /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 1443 -j ACCEPT
[root@new-mq-node nginx]# /etc/init.d/iptables  reload
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@new-mq-node nginx]# iptables -L -nv | grep 1443
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:1443 
[root@new-mq-node nginx]# 
{% endhighlight %}


---


### 检查和重载nginx服务

{% highlight bash %}
[root@new-mq-node nginx]# netstat  -ant | grep 1443
[root@new-mq-node nginx]# sbin/nginx -t -c conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@new-mq-node nginx]# ps faux | grep nginx | grep -v grep 
root      4623  0.0  0.0  46460  3632 ?        Ss   Jan07   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
nginx    15681  0.0  0.0  46956  2028 ?        S    10:16   0:00  \_ nginx: worker process        
[root@new-mq-node nginx]# kill -HUP 4623
[root@new-mq-node nginx]# netstat  -ant | grep 1443
tcp        0      0 0.0.0.0:1443                0.0.0.0:*                   LISTEN      
[root@new-mq-node nginx]# 
{% endhighlight %}


---

### 配置DNAT

{% highlight bash %}
[root@net-border ~]#  iptables -L -nv  -t nat | grep 1443
[root@net-border ~]# vim /etc/sysconfig/iptables
[root@net-border ~]# grep 1443 /etc/sysconfig/iptables
-A PREROUTING -p tcp -m tcp --dport 21443 -j DNAT --to-destination 192.168.66.111:1443
[root@net-border ~]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@net-border ~]#  iptables -L -nv  -t nat | grep 1443
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:21443 to:192.168.66.111:1443 
[root@net-border ~]#
{% endhighlight %}

---

## 访问控制

管理插件在已有的RabbitMQ权限模型上进行了扩展，通过分配标签来为用户赋权

目前有以下几种标签和角色:

<table>
          <tr>
            <th>Tag</th>
            <th>Capabilities</th>
          </tr>
          <tr>
            <td>(None)</td>
            <td>
              No access to the management plugin
            </td>
          </tr>
          <tr>
            <td>management</td>
            <td>
              Anything the user could do via AMQP plus:
              <ul>
                <li>List virtual hosts to which they can log in via AMQP</li>
                <li>
                  View all queues, exchanges and bindings in "their"
                  virtual hosts
                </li>
                <li>View and close their own channels and connections</li>
                <li>
                  View "global" statistics covering all their
                  virtual hosts, including activity by other users
                  within them
                </li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>policymaker</td>
            <td>
              Everything "management" can plus:
              <ul>
                <li>
                  View, create and delete policies and parameters for virtual
                  hosts to which they can log in via AMQP
                </li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>monitoring</td>
            <td>
              Everything "management" can plus:
              <ul>
                <li>
                  List all virtual hosts, including ones they could
                  not log in to via AMQP
                </li>
                <li>View other users's connections and channels</li>
                <li>View node-level data such as memory use and clustering</li>
                <li>View truly global statistics for all virtual hosts</li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>administrator</td>
            <td>
              Everything "policymaker" and "monitoring" can plus:
              <ul>
                <li>Create and delete virtual hosts</li>
                <li>View, create and delete users</li>
                <li>View, create and delete permissions</li>
                <li>Close other users's connections</li>
              </ul>
            </td>
          </tr>
        </table>



### 创建用户

{% highlight bash %}
[root@rabbitmq ~]# rabbitmqctl  list_users
Listing users ...
cooper	[]
guest	[administrator]
[root@rabbitmq ~]# rabbitmqctl add_user mqmonitor mqpass
Creating user "mqmonitor" ...
[root@rabbitmq ~]# rabbitmqctl  list_users
Listing users ...
cooper	[]
guest	[administrator]
mqmonitor	[]
[root@rabbitmq ~]#
{% endhighlight %}

### 用户赋权

{% highlight bash %}
[root@rabbitmq ~]# rabbitmqctl set_user_tags mqmonitor monitoring
Setting tags for user "mqmonitor" to [monitoring] ...
[root@rabbitmq ~]# rabbitmqctl  list_users
Listing users ...
cooper	[]
guest	[administrator]
mqmonitor	[monitoring]
[root@rabbitmq ~]# 
{% endhighlight %}


---

## 进行访问


因为是自签名证书，所以第一次访问时会弹出警告

要点继续接受来进行访问

然后会出现基础认证的提示框

![mq1.png](/images/mq_monitoring/mq1.png)


输入帐号密码后，会出现RabbitMQ的认证窗口

![mq2.png](/images/mq_monitoring/mq2.png)

![mq3.png](/images/mq_monitoring/mq3.png)

正确输入密码后，就可以看到管理界面

![mq4.png](/images/mq_monitoring/mq4.png)

![mq5.png](/images/mq_monitoring/mq5.png)


---

## 创建集群


当前的集群为单节点

{% highlight bash %}
[root@rabbitmq ~]# rabbitmqctl  cluster_status
Cluster status of node 'rabbit@rabbitmq' ...
[{nodes,[{disc,['rabbit@rabbitmq']}]},
 {running_nodes,['rabbit@rabbitmq']},
 {cluster_name,<<"rabbit@rabbitmq">>},
 {partitions,[]}]
[root@rabbitmq ~]# 
{% endhighlight %}

> **Tip:** 也可以在管理界面里看得到

### 安装Rabbitmq

{% highlight bash %}
[root@new-mq-node nginx]# yum install erlang
...
...
[root@new-mq-node src]# wget  http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.6/rabbitmq-server-3.5.6-1.noarch.rpm
--2016-01-13 13:57:46--  http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.6/rabbitmq-server-3.5.6-1.noarch.rpm
Resolving www.rabbitmq.com... 192.240.153.117
Connecting to www.rabbitmq.com|192.240.153.117|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4239866 (4.0M) [application/x-redhat-package-manager]
Saving to: “rabbitmq-server-3.5.6-1.noarch.rpm”

100%[===============================================================================>] 4,239,866    177K/s   in 2m 7s   

2016-01-13 13:59:54 (32.7 KB/s) - “rabbitmq-server-3.5.6-1.noarch.rpm” saved [4239866/4239866]

[root@new-mq-node src]# 
[root@new-mq-node src]# rpm -ivh rabbitmq-server-3.5.6-1.noarch.rpm 
warning: rabbitmq-server-3.5.6-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 056e8e56: NOKEY
Preparing...                ########################################### [100%]
   1:rabbitmq-server        ########################################### [100%]
[root@new-mq-node src]# rpm -qa | grep mq 
rabbitmq-server-3.5.6-1.noarch
[root@new-mq-node src]# 


要与原来节点的版本保持一致


{% endhighlight %}

### 同步 Erlang cookie

集群中node必须使用相同的cookie才能相互通讯

在Linux中cookie的位置一般在 **/var/lib/rabbitmq/.erlang.cookie**

{% highlight bash %}
[root@new-mq-node rabbitmq]# vim .erlang.cookie
[root@new-mq-node rabbitmq]# cat .erlang.cookie 
ABCDEFGGTESTGNUMPXYZ
[root@new-mq-node rabbitmq]# 
{% endhighlight %}

> **Note:** 一定要确保所有node上的cookie内容相同，并且为所有者只读


#### 报错


如果 **.erlang.cookie** 不为所有者只读，启动时会报如下错误


{% highlight bash %}
[root@new-mq-node rabbitmq]# ll .erlang.cookie 
-rw-r--r--   1 rabbitmq rabbitmq   21 Jan 13 14:08 .erlang.cookie
[root@new-mq-node rabbitmq]# /etc/init.d/rabbitmq-server  start 
Starting rabbitmq-server: FAILED - check /var/log/rabbitmq/startup_{log, _err}
rabbitmq-server.
[root@new-mq-node rabbitmq]# cd /var/log/rabbitmq/
[root@new-mq-node rabbitmq]# ls
startup_err  startup_log
[root@new-mq-node rabbitmq]# tail startup_err 

Crash dump was written to: erl_crash.dump
Kernel pid terminated (application_controller) ({application_start_failure,kernel,{shutdown,{kernel,start,[normal,[]]\}\}})
[root@new-mq-node rabbitmq]# tail startup_log 
{error_logger,\{\{2016,1,13},{14,22,17\}\},"Cookie file /var/lib/rabbitmq/.erlang.cookie must be accessible by owner only",[]}
{error_logger,\{\{2016,1,13},{14,22,17\}\},crash_report,[[{initial_call,{auth,init,['Argument__1']\}\},{pid,<0.19.0>},{registered_name,[]},{error_info,{exit,{"Cookie file /var/lib/rabbitmq/.erlang.cookie must be accessible by owner only",[{auth,init_cookie,0},{auth,init,1},{gen_server,init_it,6},{proc_lib,init_p_do_apply,3}]},[{gen_server,init_it,6},{proc_lib,init_p_do_apply,3}]\}\},{ancestors,[net_sup,kernel_sup,<0.9.0>]},{messages,[]},{links,[<0.17.0>]},{dictionary,[]},{trap_exit,true},{status,running},{heap_size,987},{stack_size,24},{reductions,428}],[]]}
{error_logger,\{\{2016,1,13},{14,22,17\}\},supervisor_report,[{supervisor,{local,net_sup\}\},{errorContext,start_error},{reason,{"Cookie file /var/lib/rabbitmq/.erlang.cookie must be accessible by owner only",[{auth,init_cookie,0},{auth,init,1},{gen_server,init_it,6},{proc_lib,init_p_do_apply,3}]\}\},{offender,[{pid,undefined},{name,auth},{mfargs,{auth,start_link,[]\}\},{restart_type,permanent},{shutdown,2000},{child_type,worker}]}]}
{error_logger,\{\{2016,1,13},{14,22,17\}\},supervisor_report,[{supervisor,{local,kernel_sup\}\},{errorContext,start_error},{reason,shutdown},{offender,[{pid,undefined},{name,net_sup},{mfargs,{erl_distribution,start_link,[]\}\},{restart_type,permanent},{shutdown,infinity},{child_type,supervisor}]}]}
{error_logger,\{\{2016,1,13},{14,22,17\}\},std_info,[{application,kernel},{exited,{shutdown,{kernel,start,[normal,[]]\}\}},{type,permanent}]}
{"Kernel pid terminated",application_controller,"{application_start_failure,kernel,{shutdown,{kernel,start,[normal,[]]\}\}}"}
[root@new-mq-node rabbitmq]#
{% endhighlight %}

解决办法是修改权限成为 **`-r--------`**


{% highlight bash %}
[root@new-mq-node rabbitmq]# chmod 400 .erlang.cookie 
[root@new-mq-node rabbitmq]# ll .erlang.cookie 
-r-------- 1 rabbitmq rabbitmq 21 Jan 13 14:08 .erlang.cookie
[root@new-mq-node rabbitmq]# 
{% endhighlight %}

再次尝试就能成功启动

{% highlight bash %}
[root@new-mq-node rabbitmq]# /etc/init.d/rabbitmq-server  start 
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@new-mq-node rabbitmq]# /etc/init.d/rabbitmq-server  status
Status of node 'rabbit@new-mq-node' ...
...
...
[root@new-mq-node rabbitmq]# netstat  -ant | grep 5672
tcp        0      0 0.0.0.0:25672               0.0.0.0:*                   LISTEN      
tcp        0      0 :::5672                     :::*                        LISTEN      
[root@new-mq-node rabbitmq]#
{% endhighlight %}


---

### 打开防火墙

需要打开以下端口以供访问

* **5672**  : for amqp
* **25672**  : for clustering
* **15672** : RabbitMQ Management for web

{% highlight bash %}
[root@new-mq-node rabbitmq]# iptables -L -nv | grep 5672
[root@new-mq-node rabbitmq]# grep 5672 /etc/sysconfig/iptables 
[root@new-mq-node rabbitmq]# vim /etc/sysconfig/iptables
[root@new-mq-node rabbitmq]# grep 5672 /etc/sysconfig/iptables 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 25672 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 15672 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 5672 -j ACCEPT
[root@new-mq-node rabbitmq]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@new-mq-node rabbitmq]# iptables -L -nv | grep 5672
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:25672 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:15672 
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:5672 
[root@new-mq-node rabbitmq]#
{% endhighlight %}

> **Tip:** 其实只要目标节点，也就是指向(join_cluster to xx)的那个节点(xx) 25672打开了，就可以加入了，也能正常运行，只是这种情况下，自己就不能被加入，也无法提供服务

---

### 加入集群

当前状态

{% highlight bash %}
[root@new-mq-node rabbitmq]# rabbitmqctl  cluster_status
Cluster status of node 'rabbit@new-mq-node' ...
[{nodes,[{disc,['rabbit@new-mq-node']}]},
 {running_nodes,['rabbit@new-mq-node']},
 {cluster_name,<<"rabbit@new-mq-node">>},
 {partitions,[]}]
[root@new-mq-node rabbitmq]# 
{% endhighlight %}

停止应用

{% highlight bash %}
[root@new-mq-node rabbitmq]# rabbitmqctl  stop_app
Stopping node 'rabbit@new-mq-node' ...
[root@new-mq-node rabbitmq]#
{% endhighlight %}

加入集群

{% highlight bash %}
[root@new-mq-node rabbitmq]# rabbitmqctl join_cluster rabbit@rabbitmq
Clustering node 'rabbit@new-mq-node' with 'rabbit@rabbitmq' ...
[root@new-mq-node rabbitmq]#
[root@new-mq-node rabbitmq]# rabbitmqctl  cluster_status
Cluster status of node 'rabbit@new-mq-node' ...
[{nodes,[{disc,['rabbit@new-mq-node','rabbit@rabbitmq']}]}]
[root@new-mq-node rabbitmq]# 
{% endhighlight %}

启动应用

{% highlight bash %}
[root@new-mq-node rabbitmq]# rabbitmqctl start_app
Starting node 'rabbit@new-mq-node' ...
[root@new-mq-node rabbitmq]# rabbitmqctl  cluster_status
Cluster status of node 'rabbit@new-mq-node' ...
[{nodes,[{disc,['rabbit@new-mq-node','rabbit@rabbitmq']}]},
 {running_nodes,['rabbit@rabbitmq','rabbit@new-mq-node']},
 {cluster_name,<<"rabbit@rabbitmq">>},
 {partitions,[]}]
[root@new-mq-node rabbitmq]#
{% endhighlight %}


---

### 开启管理插件

{% highlight bash %}
[root@new-mq-node rabbitmq]# rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status:   * = running on rabbit@new-mq-node
 |/
[  ] amqp_client                       3.5.6
[  ] cowboy                            0.5.0-rmq3.5.6-git4b93c2d
[  ] eldap                             3.5.6-gite309de4
[  ] mochiweb                          2.7.0-rmq3.5.6-git680dba8
[  ] rabbitmq_amqp1_0                  3.5.6
[  ] rabbitmq_auth_backend_ldap        3.5.6
[  ] rabbitmq_auth_mechanism_ssl       3.5.6
[  ] rabbitmq_consistent_hash_exchange 3.5.6
[  ] rabbitmq_federation               3.5.6
[  ] rabbitmq_federation_management    3.5.6
[  ] rabbitmq_management               3.5.6
[  ] rabbitmq_management_agent         3.5.6
[  ] rabbitmq_management_visualiser    3.5.6
[  ] rabbitmq_mqtt                     3.5.6
[  ] rabbitmq_shovel                   3.5.6
[  ] rabbitmq_shovel_management        3.5.6
[  ] rabbitmq_stomp                    3.5.6
[  ] rabbitmq_test                     3.5.6
[  ] rabbitmq_tracing                  3.5.6
[  ] rabbitmq_web_dispatch             3.5.6
[  ] rabbitmq_web_stomp                3.5.6
[  ] rabbitmq_web_stomp_examples       3.5.6
[  ] sockjs                            0.3.4-rmq3.5.6-git3132eb9
[  ] webmachine                        1.10.3-rmq3.5.6-gite9359c7
[root@new-mq-node rabbitmq]# rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@new-mq-node... started 6 plugins.
[root@new-mq-node rabbitmq]# rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status:   * = running on rabbit@new-mq-node
 |/
[e*] amqp_client                       3.5.6
[  ] cowboy                            0.5.0-rmq3.5.6-git4b93c2d
[  ] eldap                             3.5.6-gite309de4
[e*] mochiweb                          2.7.0-rmq3.5.6-git680dba8
[  ] rabbitmq_amqp1_0                  3.5.6
[  ] rabbitmq_auth_backend_ldap        3.5.6
[  ] rabbitmq_auth_mechanism_ssl       3.5.6
[  ] rabbitmq_consistent_hash_exchange 3.5.6
[  ] rabbitmq_federation               3.5.6
[  ] rabbitmq_federation_management    3.5.6
[E*] rabbitmq_management               3.5.6
[e*] rabbitmq_management_agent         3.5.6
[  ] rabbitmq_management_visualiser    3.5.6
[  ] rabbitmq_mqtt                     3.5.6
[  ] rabbitmq_shovel                   3.5.6
[  ] rabbitmq_shovel_management        3.5.6
[  ] rabbitmq_stomp                    3.5.6
[  ] rabbitmq_test                     3.5.6
[  ] rabbitmq_tracing                  3.5.6
[e*] rabbitmq_web_dispatch             3.5.6
[  ] rabbitmq_web_stomp                3.5.6
[  ] rabbitmq_web_stomp_examples       3.5.6
[  ] sockjs                            0.3.4-rmq3.5.6-git3132eb9
[e*] webmachine                        1.10.3-rmq3.5.6-gite9359c7
[root@new-mq-node rabbitmq]# netstat  -ant | grep 15672
tcp        0      0 0.0.0.0:15672               0.0.0.0:*                   LISTEN      
[root@new-mq-node rabbitmq]#
{% endhighlight %}

 **Note:** 如果不启用 **rabbitmq_management** 那么在管理界面里是看不到新节点 **File descriptors** 、**Socket descriptors** 、 **Erlang processes** 、 **Memory** 、 **Disk space** 、**Info** 等相关状态的


再次访问管理界面，就可以看到新加入的节点信息


![mq6.png](/images/mq_monitoring/mq6.png)

![mq7.png](/images/mq_monitoring/mq7.png)



---

# 命令汇总

* **`rabbitmq-plugins -h`**
* **`rabbitmq-plugins enable rabbitmq_management`**
* **`rabbitmq-plugins list`**
* **`netstat  -ant | grep 15672`**
* **`curl http://localhost:15672`**
* **`perl -e 'print  crypt(mqpass,mqpass)'`**
* **`cat mq.passwd`**
* **`tail nginx.conf | grep include`**
* **`cat apps/mq.conf`**
* **`iptables -L -nv | grep 15672`**
* **`grep 15672 /etc/sysconfig/iptables`**
* **`grep 1443 /etc/sysconfig/iptables`**
* **`sbin/nginx -t -c conf/nginx.conf`**
* **`kill -HUP 4623`**
* **`netstat  -ant | grep 1443`**
* **`grep 1443 /etc/sysconfig/iptables`**
* **`iptables -L -nv  -t nat | grep 1443`**
* **`rabbitmqctl add_user mqmonitor mqpass`**
* **`rabbitmqctl set_user_tags mqmonitor monitoring`**
* **`rabbitmqctl  list_users`**
* **`rabbitmqctl  cluster_status`**
* **`yum install erlang`**
* **`wget  http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.6/rabbitmq-server-3.5.6-1.noarch.rpm`**
* **`rpm -ivh rabbitmq-server-3.5.6-1.noarch.rpm`**
* **`vim .erlang.cookie`**
* **`cat .erlang.cookie`**
* **`/etc/init.d/rabbitmq-server  start`**
* **`cd /var/log/rabbitmq/`**
* **`tail startup_err`**
* **`tail startup_log`**
* **`chmod 400 .erlang.cookie`**
* **`ll .erlang.cookie`**
* **`/etc/init.d/rabbitmq-server  start`**
* **`/etc/init.d/rabbitmq-server  status`**
* **`grep 5672 /etc/sysconfig/iptables`**
* **`iptables -L -nv | grep 5672`**
* **`rabbitmqctl  stop_app`**
* **`rabbitmqctl join_cluster rabbit@rabbitmq`**
* **`rabbitmqctl start_app`**
* **`rabbitmqctl  cluster_status`**
* **`rabbitmq-plugins enable rabbitmq_management`**
* **`rabbitmq-plugins list`**


---

[rabbitmq]:http://www.rabbitmq.com/
[rabbitmq_management]:http://www.rabbitmq.com/management.html


