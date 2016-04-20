---
layout: post
title:  RabbitMQ 的CLI管理工具 rabbitmqadmin
categories:  linux  rabbitmq
wc: 2272   8098 103484 
excerpt: Erlang 的仓库下载配置，Erlang 的升级，RabbitMQ 的升级，服务启动，插件启用 ，RabbitMQ CLI 管理工具 rabbitmqadmin 的获取，RabbitMQ 的架构、概念、消息投递过程，user、vhost、connection、exchange、binding、permission、channels、parameter、consumer、queue、policies、node 的查看删除和定义，格式化输出，消息发布与消费，exchange 三种类型 (fanout、direct、topic) 的特性
comments: true
---



# 前言

**[RabbitMQ][rabbitmq]** 是一个使用 Erlang 编写的开源消息队列中间件，被广泛使用在各种应用场景中

一般对于它的监控和管理可以通过web来完成，详细可以参考 **[RabbitMQ 监控][rabbitmq_monitoring]** 

但生产环境中经常没有访问web管理界面的条件，只提供了CLI界面，或者有些自动化的需求通过web界面无法完成，这时有没有一种直接在CLI环境下进行管理的方法呢，官方提供的 **rabbitmqadmin** 命令正好可以满足这类需求

对于运维来说，个人感觉更倾向使用CLI的方式，因为虽然web的界面更友好，但是明显不如CLI快捷，CLI也可以结合其它命令进行更进一步的处理，比如将关键信息查出来后提供给集中的监控系统以触发报警

目前 **rabbitmqadmin** 可以完成以下任务：

* 列出 exchanges, queues, bindings, vhosts, users, permissions, connections and channels
* 看到汇总信息
* 申明和清除 exchanges, queues, bindings, vhosts, users and permissions
* 发布和获取消息
* 关闭连接和清空队列
* 导入导出配置

这里分享一下 **rabbitmqadmin** 的基本操作，详细可以参考 **[官方文档][management_cli]**

> **Tip:** 当前的最新版本为 **RabbitMQ 3.6.1** 发布于 01 Mar 2016 ，当前最新的 Erlang 版本为 **erlang 18.3** ， **RabbitMQ 3.6.1** 依赖于 **`>= R16B-03`** 的 Erlang


~~~
[root@h102 rabbitmq]# rpm -ivh rabbitmq-server-3.6.1-1.noarch.rpm 
warning: rabbitmq-server-3.6.1-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 056e8e56: NOKEY
error: Failed dependencies:
	erlang >= R16B-03 is needed by rabbitmq-server-3.6.1-1.noarch
[root@h102 rabbitmq]# 
~~~


---


# 概要

* TOC
{:toc}


---

## 环境


~~~
[root@h102 rabbitmq]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 rabbitmq]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 rabbitmq]#
~~~


---


## 升级Erlang

最新版的 Erlang 可以在 **[Erlang下载地址][erlang_down]** 里找到

如果之前没安装过 Erlang ，可以直接下载

~~~
[root@h102 rabbitmq]# wget https://packages.erlang-solutions.com/erlang/esl-erlang/FLAVOUR_1_general/esl-erlang_18.3-1~centos~6_amd64.rpm
--2016-04-18 16:45:35--  https://packages.erlang-solutions.com/erlang/esl-erlang/FLAVOUR_1_general/esl-erlang_18.3-1~centos~6_amd64.rpm
Resolving packages.erlang-solutions.com... 31.172.186.53
Connecting to packages.erlang-solutions.com|31.172.186.53|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 44480824 (42M) [application/x-redhat-package-manager]
Saving to: “esl-erlang_18.3-1~centos~6_amd64.rpm”

100%[===================================================================================================>] 44,480,824   350K/s   in 4m 54s  

2016-04-18 16:50:31 (148 KB/s) - “esl-erlang_18.3-1~centos~6_amd64.rpm” saved [44480824/44480824]

[root@h102 rabbitmq]# 
~~~

但是建议还是使用仓库的方式，特别是在已经安装有Erlang的情况下，因为如果添加删除包的过程中有依赖，让 YUM 自动去解决是最省心的

如下：

~~~
[root@h102 rabbitmq]# rpm -ivh esl-erlang_18.3-1~centos~6_amd64.rpm 
Preparing...                ########################################### [100%]
	file /usr/bin/epmd from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/erl from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/erlc from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/escript from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/run_erl from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/run_test from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/to_erl from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-erts-R14B-04.3.el6.x86_64
	file /usr/bin/dialyzer from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-dialyzer-R14B-04.3.el6.x86_64
	file /usr/bin/typer from install of esl-erlang-18.3-1.x86_64 conflicts with file from package erlang-typer-R14B-04.3.el6.x86_64
[root@h102 rabbitmq]# rpm -e erlang-erts-R14B-04.3.el6.x86_64
error: Failed dependencies:
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-crypto-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-kernel-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-hipe-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-syntax_tools-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-stdlib-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-compiler-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-runtime_tools-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-mnesia-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-snmp-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-xmerl-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-public_key-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-ssl-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-inets-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-orber-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosEvent-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosTime-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosNotification-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosProperty-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-edoc-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-ssh-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-otp_mibs-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-asn1-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-docbuilder-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosFileTransfer-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosEventDomain-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-cosTransactions-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-percept-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-inviso-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-parsetools-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-eunit-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-diameter-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-ic-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-jinterface-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-erl_interface-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-erl_docgen-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-gs-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-pman-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-tv-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-appmon-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-toolbar-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-odbc-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-wx-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-debugger-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-et-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-webtool-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-observer-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-tools-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-sasl-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-test_server-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-dialyzer-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-typer-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-common_test-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-reltool-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-os_mon-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-megaco-R14B-04.3.el6.x86_64
	erlang-erts(x86-64) = R14B-04.3.el6 is needed by (installed) erlang-R14B-04.3.el6.x86_64
[root@h102 rabbitmq]# 
~~~

是不是看着就头大，还是使用仓库的方法吧

---

### 下载仓库

~~~
[root@h102 rabbitmq]#  wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
--2016-04-18 16:47:40--  http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
Resolving packages.erlang-solutions.com... 31.172.186.53
Connecting to packages.erlang-solutions.com|31.172.186.53|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1932 (1.9K) [application/x-redhat-package-manager]
Saving to: “erlang-solutions-1.0-1.noarch.rpm”

100%[===================================================================================================>] 1,932       --.-K/s   in 0s      

2016-04-18 16:47:41 (199 MB/s) - “erlang-solutions-1.0-1.noarch.rpm” saved [1932/1932]

[root@h102 rabbitmq]#
~~~

---

### 安装仓库

~~~
[root@h102 rabbitmq]# rpm -ivh erlang-solutions-1.0-1.noarch.rpm 
Preparing...                ########################################### [100%]
   1:erlang-solutions       ########################################### [100%]
--2016-04-18 16:47:48--  http://packages.erlang-solutions.com/rpm/centos/erlang_solutions.repo
Resolving packages.erlang-solutions.com... 31.172.186.53
Connecting to packages.erlang-solutions.com|31.172.186.53|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 245
Saving to: “erlang_solutions.repo”

100%[===================================================================================================>] 245         --.-K/s   in 0s      

2016-04-18 16:47:48 (30.4 MB/s) - “erlang_solutions.repo” saved [245/245]


[root@h102 rabbitmq]# ll /etc/yum.repos.d/ | grep erl
-rw-r--r--  1 root root  245 Sep 26  2013 erlang_solutions.repo
[root@h102 rabbitmq]# cat /etc/yum.repos.d/erlang_solutions.repo 
[erlang-solutions]
name=Centos $releasever - $basearch - Erlang Solutions
baseurl=http://packages.erlang-solutions.com/rpm/centos/$releasever/$basearch
gpgcheck=0
gpgkey=http://packages.erlang-solutions.com/debian/erlang_solutions.asc
enabled=1
[root@h102 rabbitmq]# 
[root@h102 rabbitmq]# yum list all | grep erlang
erlang.x86_64                              R14B-04.3.el6                @epel   
erlang-appmon.x86_64                       R14B-04.3.el6                @epel   
erlang-asn1.x86_64                         R14B-04.3.el6                @epel   
erlang-common_test.x86_64                  R14B-04.3.el6                @epel   
erlang-compiler.x86_64                     R14B-04.3.el6                @epel   
erlang-cosEvent.x86_64                     R14B-04.3.el6                @epel   
erlang-cosEventDomain.x86_64               R14B-04.3.el6                @epel   
erlang-cosFileTransfer.x86_64              R14B-04.3.el6                @epel   
erlang-cosNotification.x86_64              R14B-04.3.el6                @epel   
erlang-cosProperty.x86_64                  R14B-04.3.el6                @epel   
erlang-cosTime.x86_64                      R14B-04.3.el6                @epel   
erlang-cosTransactions.x86_64              R14B-04.3.el6                @epel   
erlang-crypto.x86_64                       R14B-04.3.el6                @epel   
erlang-debugger.x86_64                     R14B-04.3.el6                @epel   
erlang-dialyzer.x86_64                     R14B-04.3.el6                @epel   
erlang-diameter.x86_64                     R14B-04.3.el6                @epel   
erlang-docbuilder.x86_64                   R14B-04.3.el6                @epel   
erlang-edoc.x86_64                         R14B-04.3.el6                @epel   
erlang-erl_docgen.x86_64                   R14B-04.3.el6                @epel   
erlang-erl_interface.x86_64                R14B-04.3.el6                @epel   
erlang-erts.x86_64                         R14B-04.3.el6                @epel   
erlang-et.x86_64                           R14B-04.3.el6                @epel   
erlang-eunit.x86_64                        R14B-04.3.el6                @epel   
erlang-examples.x86_64                     R14B-04.3.el6                @epel   
erlang-gs.x86_64                           R14B-04.3.el6                @epel   
erlang-hipe.x86_64                         R14B-04.3.el6                @epel   
erlang-ic.x86_64                           R14B-04.3.el6                @epel   
erlang-inets.x86_64                        R14B-04.3.el6                @epel   
erlang-inviso.x86_64                       R14B-04.3.el6                @epel   
erlang-jinterface.x86_64                   R14B-04.3.el6                @epel   
erlang-kernel.x86_64                       R14B-04.3.el6                @epel   
erlang-megaco.x86_64                       R14B-04.3.el6                @epel   
erlang-mnesia.x86_64                       R14B-04.3.el6                @epel   
erlang-observer.x86_64                     R14B-04.3.el6                @epel   
erlang-odbc.x86_64                         R14B-04.3.el6                @epel   
erlang-orber.x86_64                        R14B-04.3.el6                @epel   
erlang-os_mon.x86_64                       R14B-04.3.el6                @epel   
erlang-otp_mibs.x86_64                     R14B-04.3.el6                @epel   
erlang-parsetools.x86_64                   R14B-04.3.el6                @epel   
erlang-percept.x86_64                      R14B-04.3.el6                @epel   
erlang-pman.x86_64                         R14B-04.3.el6                @epel   
erlang-public_key.x86_64                   R14B-04.3.el6                @epel   
erlang-reltool.x86_64                      R14B-04.3.el6                @epel   
erlang-runtime_tools.x86_64                R14B-04.3.el6                @epel   
erlang-sasl.x86_64                         R14B-04.3.el6                @epel   
erlang-snmp.x86_64                         R14B-04.3.el6                @epel   
erlang-solutions.noarch                    1.0-1                        installed
erlang-ssh.x86_64                          R14B-04.3.el6                @epel   
erlang-ssl.x86_64                          R14B-04.3.el6                @epel   
erlang-stdlib.x86_64                       R14B-04.3.el6                @epel   
erlang-syntax_tools.x86_64                 R14B-04.3.el6                @epel   
erlang-test_server.x86_64                  R14B-04.3.el6                @epel   
erlang-toolbar.x86_64                      R14B-04.3.el6                @epel   
erlang-tools.x86_64                        R14B-04.3.el6                @epel   
erlang-tv.x86_64                           R14B-04.3.el6                @epel   
erlang-typer.x86_64                        R14B-04.3.el6                @epel   
erlang-webtool.x86_64                      R14B-04.3.el6                @epel   
erlang-wx.x86_64                           R14B-04.3.el6                @epel   
erlang-xmerl.x86_64                        R14B-04.3.el6                @epel   
emacs-erlang.noarch                        18.3-1.el6                   erlang-solutions
emacs-erlang-el.noarch                     18.3-1.el6                   erlang-solutions
erlang.x86_64                              18.3-1.el6                   erlang-solutions
erlang-appmon.x86_64                       R16B03-0.2.el6               erlang-solutions
erlang-asn1.x86_64                         18.3-1.el6                   erlang-solutions
erlang-common_test.x86_64                  18.3-1.el6                   erlang-solutions
erlang-compiler.x86_64                     18.3-1.el6                   erlang-solutions
erlang-cosEvent.x86_64                     18.3-1.el6                   erlang-solutions
erlang-cosEventDomain.x86_64               18.3-1.el6                   erlang-solutions
erlang-cosFileTransfer.x86_64              18.3-1.el6                   erlang-solutions
erlang-cosNotification.x86_64              18.3-1.el6                   erlang-solutions
erlang-cosProperty.x86_64                  18.3-1.el6                   erlang-solutions
erlang-cosTime.x86_64                      18.3-1.el6                   erlang-solutions
erlang-cosTransactions.x86_64              18.3-1.el6                   erlang-solutions
erlang-crypto.x86_64                       18.3-1.el6                   erlang-solutions
erlang-debugger.x86_64                     18.3-1.el6                   erlang-solutions
erlang-dialyzer.x86_64                     18.3-1.el6                   erlang-solutions
erlang-diameter.x86_64                     18.3-1.el6                   erlang-solutions
erlang-doc.noarch                          18.3-1.el6                   erlang-solutions
erlang-edoc.x86_64                         18.3-1.el6                   erlang-solutions
erlang-eldap.x86_64                        18.3-1.el6                   erlang-solutions
erlang-erl_docgen.x86_64                   18.3-1.el6                   erlang-solutions
erlang-erl_interface.x86_64                18.3-1.el6                   erlang-solutions
erlang-erlsom.x86_64                       1.2.1-12.20120904gitdef76b9.el6
erlang-erts.x86_64                         18.3-1.el6                   erlang-solutions
erlang-et.x86_64                           18.3-1.el6                   erlang-solutions
erlang-eunit.x86_64                        18.3-1.el6                   erlang-solutions
erlang-examples.x86_64                     18.3-1.el6                   erlang-solutions
erlang-gs.x86_64                           18.3-1.el6                   erlang-solutions
erlang-hipe.x86_64                         18.3-1.el6                   erlang-solutions
erlang-ibrowse.x86_64                      2.2.0-4.el6                  epel    
erlang-ic.x86_64                           18.3-1.el6                   erlang-solutions
erlang-inets.x86_64                        18.3-1.el6                   erlang-solutions
erlang-jinterface.x86_64                   18.3-1.el6                   erlang-solutions
erlang-kernel.x86_64                       18.3-1.el6                   erlang-solutions
erlang-megaco.x86_64                       18.3-1.el6                   erlang-solutions
erlang-mnesia.x86_64                       18.3-1.el6                   erlang-solutions
erlang-observer.x86_64                     18.3-1.el6                   erlang-solutions
erlang-odbc.x86_64                         18.3-1.el6                   erlang-solutions
erlang-orber.x86_64                        18.3-1.el6                   erlang-solutions
erlang-os_mon.x86_64                       18.3-1.el6                   erlang-solutions
erlang-ose.x86_64                          18.3-1.el6                   erlang-solutions
erlang-otp_mibs.x86_64                     18.3-1.el6                   erlang-solutions
erlang-parsetools.x86_64                   18.3-1.el6                   erlang-solutions
erlang-percept.x86_64                      18.3-1.el6                   erlang-solutions
erlang-pgsql.x86_64                        0-6.20101203svn.el6          epel    
erlang-pman.x86_64                         R16B03-0.2.el6               erlang-solutions
erlang-public_key.x86_64                   18.3-1.el6                   erlang-solutions
erlang-reltool.x86_64                      18.3-1.el6                   erlang-solutions
erlang-runtime_tools.x86_64                18.3-1.el6                   erlang-solutions
erlang-sasl.x86_64                         18.3-1.el6                   erlang-solutions
erlang-snmp.x86_64                         18.3-1.el6                   erlang-solutions
erlang-ssh.x86_64                          18.3-1.el6                   erlang-solutions
erlang-ssl.x86_64                          18.3-1.el6                   erlang-solutions
erlang-stdlib.x86_64                       18.3-1.el6                   erlang-solutions
erlang-syntax_tools.x86_64                 18.3-1.el6                   erlang-solutions
erlang-test_server.x86_64                  18.3-1.el6                   erlang-solutions
erlang-toolbar.x86_64                      R16B03-0.2.el6               erlang-solutions
erlang-tools.x86_64                        18.3-1.el6                   erlang-solutions
erlang-tv.x86_64                           R16B03-0.2.el6               erlang-solutions
erlang-typer.x86_64                        18.3-1.el6                   erlang-solutions
erlang-webtool.x86_64                      18.3-1.el6                   erlang-solutions
erlang-wx.x86_64                           18.3-1.el6                   erlang-solutions
erlang-xmerl.x86_64                        18.3-1.el6                   erlang-solutions
erlang-xmlrpc.x86_64                       1.13-2.el6                   epel    
esl-erlang.x86_64                          18.3-1                       erlang-solutions
xemacs-erlang.noarch                       R14B-04.3.el6                epel    
xemacs-erlang-el.noarch                    R14B-04.3.el6                epel    
[root@h102 rabbitmq]# 
~~~

---

### 升级Erlang



~~~
[root@h102 rabbitmq]# yum update erlang.x86_64
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Update Process
Loading mirror speeds from cached hostfile
 * base: mirrors.pubyun.com
 * epel: mirrors.opencas.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.pubyun.com
Resolving Dependencies
--> Running transaction check
---> Package erlang.x86_64 0:R14B-04.3.el6 will be updated
--> Processing Dependency: erlang(x86-64) = R14B-04.3.el6 for package: erlang-examples-R14B-04.3.el6.x86_64
---> Package erlang.x86_64 0:18.3-1.el6 will be obsoleting
--> Processing Dependency: erlang-webtool(x86-64) = 18.3-1.el6 for package: erlang-18.3-1.el6.x86_64
...
...
--> Processing Dependency: erlang-runtime_tools(x86-64) = 18.3-1.el6 for package: erlang-18.3-1.el6.x86_64
--> Processing Dependency: erlang-erts(x86-64) = 18.3-1.el6 for package: erlang-18.3-1.el6.x86_64
--> Processing Dependency: erlang-cosFileTransfer(x86-64) = 18.3-1.el6 for package: erlang-18.3-1.el6.x86_64
---> Package erlang-appmon.x86_64 0:R14B-04.3.el6 will be obsoleted
---> Package erlang-docbuilder.x86_64 0:R14B-04.3.el6 will be obsoleted
...
...
---> Package erlang-xmerl.x86_64 0:R14B-04.3.el6 will be updated
---> Package erlang-xmerl.x86_64 0:18.3-1.el6 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================
 Package                                   Arch                      Version                       Repository                           Size
=============================================================================================================================================
Installing:
 erlang                                    x86_64                    18.3-1.el6                    erlang-solutions                     16 k
     replacing  erlang-appmon.x86_64 R14B-04.3.el6
     replacing  erlang-docbuilder.x86_64 R14B-04.3.el6
     replacing  erlang-inviso.x86_64 R14B-04.3.el6
     replacing  erlang-pman.x86_64 R14B-04.3.el6
     replacing  erlang-toolbar.x86_64 R14B-04.3.el6
     replacing  erlang-tv.x86_64 R14B-04.3.el6
Installing for dependencies:
 erlang-eldap                              x86_64                    18.3-1.el6                    erlang-solutions                    123 k
 erlang-ose                                x86_64                    18.3-1.el6                    erlang-solutions                     25 k
Updating for dependencies:
 erlang-asn1                               x86_64                    18.3-1.el6                    erlang-solutions                    916 k
 erlang-common_test                        x86_64                    18.3-1.el6                    erlang-solutions                    976 k
 erlang-compiler                           x86_64                    18.3-1.el6                    erlang-solutions                    1.4 M
 erlang-cosEvent                           x86_64                    18.3-1.el6                    erlang-solutions                    168 k
 erlang-cosEventDomain                     x86_64                    18.3-1.el6                    erlang-solutions                    135 k
 erlang-cosFileTransfer                    x86_64                    18.3-1.el6                    erlang-solutions                    198 k
 erlang-cosNotification                    x86_64                    18.3-1.el6                    erlang-solutions                    840 k
 erlang-cosProperty                        x86_64                    18.3-1.el6                    erlang-solutions                    186 k
 erlang-cosTime                            x86_64                    18.3-1.el6                    erlang-solutions                    122 k
 erlang-cosTransactions                    x86_64                    18.3-1.el6                    erlang-solutions                    195 k
 erlang-crypto                             x86_64                    18.3-1.el6                    erlang-solutions                    188 k
 erlang-debugger                           x86_64                    18.3-1.el6                    erlang-solutions                    486 k
 erlang-dialyzer                           x86_64                    18.3-1.el6                    erlang-solutions                    766 k
 erlang-diameter                           x86_64                    18.3-1.el6                    erlang-solutions                    826 k
 erlang-edoc                               x86_64                    18.3-1.el6                    erlang-solutions                    379 k
 erlang-erl_docgen                         x86_64                    18.3-1.el6                    erlang-solutions                    168 k
 erlang-erl_interface                      x86_64                    18.3-1.el6                    erlang-solutions                    266 k
 erlang-erts                               x86_64                    18.3-1.el6                    erlang-solutions                    2.9 M
 erlang-et                                 x86_64                    18.3-1.el6                    erlang-solutions                    193 k
 erlang-eunit                              x86_64                    18.3-1.el6                    erlang-solutions                    183 k
 erlang-examples                           x86_64                    18.3-1.el6                    erlang-solutions                    1.1 M
 erlang-gs                                 x86_64                    18.3-1.el6                    erlang-solutions                    696 k
 erlang-hipe                               x86_64                    18.3-1.el6                    erlang-solutions                    3.0 M
 erlang-ic                                 x86_64                    18.3-1.el6                    erlang-solutions                    1.0 M
 erlang-inets                              x86_64                    18.3-1.el6                    erlang-solutions                    921 k
 erlang-jinterface                         x86_64                    18.3-1.el6                    erlang-solutions                    178 k
 erlang-kernel                             x86_64                    18.3-1.el6                    erlang-solutions                    1.2 M
 erlang-megaco                             x86_64                    18.3-1.el6                    erlang-solutions                    6.2 M
 erlang-mnesia                             x86_64                    18.3-1.el6                    erlang-solutions                    843 k
 erlang-observer                           x86_64                    18.3-1.el6                    erlang-solutions                    927 k
 erlang-odbc                               x86_64                    18.3-1.el6                    erlang-solutions                     87 k
 erlang-orber                              x86_64                    18.3-1.el6                    erlang-solutions                    1.1 M
 erlang-os_mon                             x86_64                    18.3-1.el6                    erlang-solutions                    133 k
 erlang-otp_mibs                           x86_64                    18.3-1.el6                    erlang-solutions                     32 k
 erlang-parsetools                         x86_64                    18.3-1.el6                    erlang-solutions                    208 k
 erlang-percept                            x86_64                    18.3-1.el6                    erlang-solutions                    178 k
 erlang-public_key                         x86_64                    18.3-1.el6                    erlang-solutions                    655 k
 erlang-reltool                            x86_64                    18.3-1.el6                    erlang-solutions                    418 k
 erlang-runtime_tools                      x86_64                    18.3-1.el6                    erlang-solutions                    223 k
 erlang-sasl                               x86_64                    18.3-1.el6                    erlang-solutions                    349 k
 erlang-snmp                               x86_64                    18.3-1.el6                    erlang-solutions                    1.9 M
 erlang-ssh                                x86_64                    18.3-1.el6                    erlang-solutions                    538 k
 erlang-ssl                                x86_64                    18.3-1.el6                    erlang-solutions                    793 k
 erlang-stdlib                             x86_64                    18.3-1.el6                    erlang-solutions                    2.7 M
 erlang-syntax_tools                       x86_64                    18.3-1.el6                    erlang-solutions                    464 k
 erlang-test_server                        x86_64                    18.3-1.el6                    erlang-solutions                    370 k
 erlang-tools                              x86_64                    18.3-1.el6                    erlang-solutions                    671 k
 erlang-typer                              x86_64                    18.3-1.el6                    erlang-solutions                     74 k
 erlang-webtool                            x86_64                    18.3-1.el6                    erlang-solutions                     55 k
 erlang-wx                                 x86_64                    18.3-1.el6                    erlang-solutions                    4.4 M
 erlang-xmerl                              x86_64                    18.3-1.el6                    erlang-solutions                    1.1 M

Transaction Summary
=============================================================================================================================================
Install       3 Package(s)
Upgrade      51 Package(s)

Total download size: 44 M
Is this ok [y/N]: y
Downloading Packages:
(1/54): erlang-18.3-1.el6.x86_64.rpm                                                                                  |  16 kB     00:00     
(2/54): erlang-asn1-18.3-1.el6.x86_64.rpm                                                                             | 916 kB     00:08     
(3/54): erlang-common_test-18.3-1.el6.x86_64.rpm                                                                      | 976 kB     00:09     
...
...    
(53/54): erlang-wx-18.3-1.el6.x86_64.rpm                                                                              | 4.4 MB     00:48     
(54/54): erlang-xmerl-18.3-1.el6.x86_64.rpm                                                                           | 1.1 MB     00:12     
---------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                         94 kB/s |  44 MB     07:58     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
** Found 6 pre-existing rpmdb problem(s), 'yum check' output follows:
perl-DBD-MySQL-4.013-3.el6.x86_64 has missing requires of libmysqlclient.so.16()(64bit)
perl-DBD-MySQL-4.013-3.el6.x86_64 has missing requires of libmysqlclient.so.16(libmysqlclient_16)(64bit)
ruby-mysql-2.8.2-1.el6.x86_64 has missing requires of libmysqlclient.so.16()(64bit)
ruby-mysql-2.8.2-1.el6.x86_64 has missing requires of libmysqlclient.so.16(libmysqlclient_16)(64bit)
ruby193-rubygem-mysql2-0.3.11-4.el6.x86_64 has missing requires of libmysqlclient_r.so.16()(64bit)
ruby193-rubygem-mysql2-0.3.11-4.el6.x86_64 has missing requires of libmysqlclient_r.so.16(libmysqlclient_16)(64bit)
  Updating   : erlang-syntax_tools-18.3-1.el6.x86_64                                                                                   1/112 
  Updating   : erlang-compiler-18.3-1.el6.x86_64                                                                                       2/112 
  Updating   : erlang-hipe-18.3-1.el6.x86_64                                                                                           3/112 
  Updating   : erlang-erts-18.3-1.el6.x86_64                                                                                           4/112 
  Updating   : erlang-kernel-18.3-1.el6.x86_64                                                                                         5/112 
  ...
  ...
  Verifying  : erlang-gs-R14B-04.3.el6.x86_64                                                                                        111/112 
  Verifying  : erlang-ssl-R14B-04.3.el6.x86_64                                                                                       112/112 

Installed:
  erlang.x86_64 0:18.3-1.el6                                                                                                                 

Dependency Installed:
  erlang-eldap.x86_64 0:18.3-1.el6                                       erlang-ose.x86_64 0:18.3-1.el6                                      

Dependency Updated:
  erlang-asn1.x86_64 0:18.3-1.el6                erlang-common_test.x86_64 0:18.3-1.el6        erlang-compiler.x86_64 0:18.3-1.el6           
  erlang-cosEvent.x86_64 0:18.3-1.el6            erlang-cosEventDomain.x86_64 0:18.3-1.el6     erlang-cosFileTransfer.x86_64 0:18.3-1.el6    
  erlang-cosNotification.x86_64 0:18.3-1.el6     erlang-cosProperty.x86_64 0:18.3-1.el6        erlang-cosTime.x86_64 0:18.3-1.el6            
  erlang-cosTransactions.x86_64 0:18.3-1.el6     erlang-crypto.x86_64 0:18.3-1.el6             erlang-debugger.x86_64 0:18.3-1.el6           
  erlang-dialyzer.x86_64 0:18.3-1.el6            erlang-diameter.x86_64 0:18.3-1.el6           erlang-edoc.x86_64 0:18.3-1.el6               
  erlang-erl_docgen.x86_64 0:18.3-1.el6          erlang-erl_interface.x86_64 0:18.3-1.el6      erlang-erts.x86_64 0:18.3-1.el6               
  erlang-et.x86_64 0:18.3-1.el6                  erlang-eunit.x86_64 0:18.3-1.el6              erlang-examples.x86_64 0:18.3-1.el6           
  erlang-gs.x86_64 0:18.3-1.el6                  erlang-hipe.x86_64 0:18.3-1.el6               erlang-ic.x86_64 0:18.3-1.el6                 
  erlang-inets.x86_64 0:18.3-1.el6               erlang-jinterface.x86_64 0:18.3-1.el6         erlang-kernel.x86_64 0:18.3-1.el6             
  erlang-megaco.x86_64 0:18.3-1.el6              erlang-mnesia.x86_64 0:18.3-1.el6             erlang-observer.x86_64 0:18.3-1.el6           
  erlang-odbc.x86_64 0:18.3-1.el6                erlang-orber.x86_64 0:18.3-1.el6              erlang-os_mon.x86_64 0:18.3-1.el6             
  erlang-otp_mibs.x86_64 0:18.3-1.el6            erlang-parsetools.x86_64 0:18.3-1.el6         erlang-percept.x86_64 0:18.3-1.el6            
  erlang-public_key.x86_64 0:18.3-1.el6          erlang-reltool.x86_64 0:18.3-1.el6            erlang-runtime_tools.x86_64 0:18.3-1.el6      
  erlang-sasl.x86_64 0:18.3-1.el6                erlang-snmp.x86_64 0:18.3-1.el6               erlang-ssh.x86_64 0:18.3-1.el6                
  erlang-ssl.x86_64 0:18.3-1.el6                 erlang-stdlib.x86_64 0:18.3-1.el6             erlang-syntax_tools.x86_64 0:18.3-1.el6       
  erlang-test_server.x86_64 0:18.3-1.el6         erlang-tools.x86_64 0:18.3-1.el6              erlang-typer.x86_64 0:18.3-1.el6              
  erlang-webtool.x86_64 0:18.3-1.el6             erlang-wx.x86_64 0:18.3-1.el6                 erlang-xmerl.x86_64 0:18.3-1.el6              

Replaced:
  erlang-appmon.x86_64 0:R14B-04.3.el6         erlang-docbuilder.x86_64 0:R14B-04.3.el6         erlang-inviso.x86_64 0:R14B-04.3.el6        
  erlang-pman.x86_64 0:R14B-04.3.el6           erlang-toolbar.x86_64 0:R14B-04.3.el6            erlang-tv.x86_64 0:R14B-04.3.el6            

Complete!
[root@h102 rabbitmq]# 
~~~



---

## 下载安装 RabbitMQ 

~~~
[root@h102 rabbitmq]# wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
--2016-04-18 16:41:08--  http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
Resolving www.rabbitmq.com... 192.240.153.117
Connecting to www.rabbitmq.com|192.240.153.117|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5088884 (4.9M) [application/x-redhat-package-manager]
Saving to: “rabbitmq-server-3.6.1-1.noarch.rpm”

100%[===================================================================================================>] 5,088,884    270K/s   in 16s     

2016-04-18 16:41:24 (320 KB/s) - “rabbitmq-server-3.6.1-1.noarch.rpm” saved [5088884/5088884]

[root@h102 rabbitmq]#
[root@h102 rabbitmq]# rpm -ivh rabbitmq-server-3.6.1-1.noarch.rpm 
warning: rabbitmq-server-3.6.1-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 056e8e56: NOKEY
Preparing...                ########################################### [100%]
   1:rabbitmq-server        ########################################### [100%]
[root@h102 rabbitmq]# 
~~~

> **Note:** **rabbitmq-server-3.6.1** 需要 **`>= R16B-03`** 的 Erlang 的支持，否则会有如下报错

~~~
[root@h102 rabbitmq]# rpm -ivh rabbitmq-server-3.6.1-1.noarch.rpm 
warning: rabbitmq-server-3.6.1-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 056e8e56: NOKEY
error: Failed dependencies:
	erlang >= R16B-03 is needed by rabbitmq-server-3.6.1-1.noarch
[root@h102 rabbitmq]#
~~~


---

## 启动服务

**rabbitmqadmin** 是由 **rabbitmq_management** 插件提供的，得启用此插件


首先启动服务

~~~
[root@h102 rabbitmq]# /etc/init.d/rabbitmq-server start 
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@h102 rabbitmq]# 
~~~

查看端口

~~~
[root@h102 rabbitmq]# ps faux | grep -i rabbitmq | grep -v grep 
rabbitmq  4703  0.0  0.0  10832   412 ?        S    15:08   0:00 /usr/lib64/erlang/erts-5.8.5/bin/epmd -daemon
root     32636  0.0  0.0 106364  1200 pts/0    S    17:29   0:00 /bin/sh /etc/init.d/rabbitmq-server start
root     32653  0.0  0.0 106096  1248 pts/0    S    17:29   0:00  \_ /bin/bash -c ulimit -S -c 0 >/dev/null 2>&1 ; /usr/sbin/rabbitmq-server
root     32654  0.0  0.0 106096  1308 pts/0    S    17:29   0:00      \_ /bin/sh /usr/sbin/rabbitmq-server
root     32662  0.0  0.1 163856  2168 pts/0    S    17:29   0:00          \_ su rabbitmq -s /bin/sh -c /usr/lib/rabbitmq/bin/rabbitmq-server 
rabbitmq 32678  0.0  0.0 106100  1424 ?        Ss   17:29   0:00              \_ /bin/sh -e /usr/lib/rabbitmq/bin/rabbitmq-server
rabbitmq 32893  0.7  3.8 1243180 73308 ?       Sl   17:29   0:49                  \_ /usr/lib64/erlang/erts-7.3/bin/beam.smp -W w -A 64 -P 1048576 -K true -B i -- -root /usr/lib64/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.1/ebin -noshell -noinput -s rabbit boot -sname rabbit@h102 -boot start_sasl -kernel inet_default_connect_options [{nodelay,true}] -sasl errlog_type error -sasl sasl_error_logger false -rabbit error_logger {file,"/var/log/rabbitmq/rabbit@h102.log"} -rabbit sasl_error_logger {file,"/var/log/rabbitmq/rabbit@h102-sasl.log"} -rabbit enabled_plugins_file "/etc/rabbitmq/enabled_plugins" -rabbit plugins_dir "/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.1/plugins" -rabbit plugins_expand_dir "/var/lib/rabbitmq/mnesia/rabbit@h102-plugins-expand" -os_mon start_cpu_sup false -os_mon start_disksup false -os_mon start_memsup false -mnesia dir "/var/lib/rabbitmq/mnesia/rabbit@h102" -kernel inet_dist_listen_min 25672 -kernel inet_dist_listen_max 25672
rabbitmq 32976  0.0  0.0  10796   512 ?        Ss   17:29   0:00                      \_ inet_gethost 4
rabbitmq 32977  0.0  0.0  12900   676 ?        S    17:29   0:00                          \_ inet_gethost 4
[root@h102 rabbitmq]# netstat  -an | grep -E "(4369|25672|5672|5671|15672|61613|61614|1883|8883)"
tcp        0      0 0.0.0.0:4369                0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:25672               0.0.0.0:*                   LISTEN      
tcp        0      0 127.0.0.1:40720             127.0.0.1:4369              ESTABLISHED 
tcp        0      0 192.168.100.102:4369        192.168.100.102:44969       TIME_WAIT   
tcp        0      0 127.0.0.1:4369              127.0.0.1:40720             ESTABLISHED 
tcp        0      0 :::5672                     :::*                        LISTEN      
[root@h102 rabbitmq]# 
~~~

---

## 启动插件


启用 **rabbitmq_management** 插件


~~~
[root@h102 rabbitmq]# rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status:   * = running on rabbit@h102
 |/
[  ] amqp_client                       3.6.1
[  ] cowboy                            1.0.3
[  ] cowlib                            1.0.1
[  ] mochiweb                          2.13.0
[  ] rabbitmq_amqp1_0                  3.6.1
[  ] rabbitmq_auth_backend_ldap        3.6.1
[  ] rabbitmq_auth_mechanism_ssl       3.6.1
[  ] rabbitmq_consistent_hash_exchange 3.6.1
[  ] rabbitmq_event_exchange           3.6.1
[  ] rabbitmq_federation               3.6.1
[  ] rabbitmq_federation_management    3.6.1
[  ] rabbitmq_management               3.6.1
[  ] rabbitmq_management_agent         3.6.1
[  ] rabbitmq_management_visualiser    3.6.1
[  ] rabbitmq_mqtt                     3.6.1
[  ] rabbitmq_recent_history_exchange  1.2.1
[  ] rabbitmq_sharding                 0.1.0
[  ] rabbitmq_shovel                   3.6.1
[  ] rabbitmq_shovel_management        3.6.1
[  ] rabbitmq_stomp                    3.6.1
[  ] rabbitmq_tracing                  3.6.1
[  ] rabbitmq_web_dispatch             3.6.1
[  ] rabbitmq_web_stomp                3.6.1
[  ] rabbitmq_web_stomp_examples       3.6.1
[  ] sockjs                            0.3.4
[  ] webmachine                        1.10.3
[root@h102 rabbitmq]# netstat  -ant | grep 15672
[root@h102 rabbitmq]# rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@h102... started 6 plugins.
[root@h102 rabbitmq]# rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status:   * = running on rabbit@h102
 |/
[e*] amqp_client                       3.6.1
[  ] cowboy                            1.0.3
[  ] cowlib                            1.0.1
[e*] mochiweb                          2.13.0
[  ] rabbitmq_amqp1_0                  3.6.1
[  ] rabbitmq_auth_backend_ldap        3.6.1
[  ] rabbitmq_auth_mechanism_ssl       3.6.1
[  ] rabbitmq_consistent_hash_exchange 3.6.1
[  ] rabbitmq_event_exchange           3.6.1
[  ] rabbitmq_federation               3.6.1
[  ] rabbitmq_federation_management    3.6.1
[E*] rabbitmq_management               3.6.1
[e*] rabbitmq_management_agent         3.6.1
[  ] rabbitmq_management_visualiser    3.6.1
[  ] rabbitmq_mqtt                     3.6.1
[  ] rabbitmq_recent_history_exchange  1.2.1
[  ] rabbitmq_sharding                 0.1.0
[  ] rabbitmq_shovel                   3.6.1
[  ] rabbitmq_shovel_management        3.6.1
[  ] rabbitmq_stomp                    3.6.1
[  ] rabbitmq_tracing                  3.6.1
[e*] rabbitmq_web_dispatch             3.6.1
[  ] rabbitmq_web_stomp                3.6.1
[  ] rabbitmq_web_stomp_examples       3.6.1
[  ] sockjs                            0.3.4
[e*] webmachine                        1.10.3
[root@h102 rabbitmq]# netstat  -ant | grep 15672
tcp        0      0 0.0.0.0:15672               0.0.0.0:*                   LISTEN      
[root@h102 rabbitmq]#
~~~


此时已经可以通过web进行访问了


![mq_webui.png](/images/mq_monitoring/mq_webui.png)


---

## **rabbitmqadmin**

**rabbitmqadmin** 可以完成和 Web UI 一样的工作，它其实是使用Python写的一个脚本，实现了一个http client

>The management plugin ships with a command line tool rabbitmqadmin which can perform the same actions as the web-based UI, and which may be more convenient for use when scripting. Note that rabbitmqadmin is just a specialised HTTP client; if you are contemplating invoking rabbitmqadmin from your own program you may want to consider using the HTTP API directly

---

### 获取 **rabbitmqadmin**

在web UI管理界面中，的 **`/cli/`** 下可以下载此脚本

![mq_cli.png](/images/mq_monitoring/mq_cli.png)

点击【here】会呈现脚本内容

![mq_physcript.png](/images/mq_monitoring/mq_physcript.png)


但是有些情况下，并没条件使用Web UI，使用下面的方法一样可以获取这个脚本

~~~
[root@h102 rabbitmq]# wget http://localhost:15672/cli/rabbitmqadmin
--2016-04-18 19:47:50--  http://localhost:15672/cli/rabbitmqadmin
Resolving localhost... ::1, 127.0.0.1
Connecting to localhost|::1|:15672... failed: Connection refused.
Connecting to localhost|127.0.0.1|:15672... connected.
HTTP request sent, awaiting response... 200 OK
Length: 34504 (34K) [text/plain]
Saving to: “rabbitmqadmin”

100%[===================================================================================================>] 34,504      --.-K/s   in 0s      

2016-04-18 19:47:50 (155 MB/s) - “rabbitmqadmin” saved [34504/34504]

[root@h102 rabbitmq]# ll rabbitmqadmin 
-rw-r--r-- 1 root root 34504 Apr 18 19:29 rabbitmqadmin
[root@h102 rabbitmq]# file rabbitmqadmin 
rabbitmqadmin: a /usr/bin/env python script text executable
[root@h102 rabbitmq]# 
[root@h102 rabbitmq]# chmod  +x rabbitmqadmin 
[root@h102 rabbitmq]# ./rabbitmqadmin --help 
Usage
=====
  rabbitmqadmin [options] subcommand

Options
=======
--help, -h              show this help message and exit
--config=CONFIG, -c CONFIG
                        configuration file [default: ~/.rabbitmqadmin.conf]
--node=NODE, -N NODE    node described in the configuration file [default:
                        'default' only if configuration file is specified]
--host=HOST, -H HOST    connect to host HOST [default: localhost]
--port=PORT, -P PORT    connect to port PORT [default: 15672]
--path-prefix=PATH_PREFIX
                        use specific URI path prefix for the RabbitMQ HTTP API
                        (default: blank string) [default: ]
--vhost=VHOST, -V VHOST
                        connect to vhost VHOST [default: all vhosts for list,
                        '/' for declare]
--username=USERNAME, -u USERNAME
                        connect using username USERNAME [default: guest]
--password=PASSWORD, -p PASSWORD
                        connect using password PASSWORD [default: guest]
--quiet, -q             suppress status messages [default: True]
--ssl, -s               connect with ssl [default: False]
--ssl-key-file=SSL_KEY_FILE
                        PEM format key file for SSL
--ssl-cert-file=SSL_CERT_FILE
                        PEM format certificate file for SSL
--format=FORMAT, -f FORMAT
                        format for listing commands - one of [raw_json, long,
                        pretty_json, kvp, tsv, table, bash] [default: table]
--sort=SORT, -S SORT    sort key for listing queries
--sort-reverse, -R      reverse the sort order
--depth=DEPTH, -d DEPTH
                        maximum depth to recurse for listing tables [default:
                        1]
--bash-completion       Print bash completion script [default: False]
--version               Display version and exit

More Help
=========

For more help use the help subcommand:

  rabbitmqadmin help subcommands  # For a list of available subcommands
  rabbitmqadmin help config       # For help with the configuration file
[root@h102 rabbitmq]# 
~~~


---

### 基础概念

#### 架构

**Producer、Exchange、Binding、Queue、Consumer** 之间的关系

![rabbitmq_arc1.jpg](/images/mq_monitoring/rabbitmq_arc1.jpg)

**Routing Key、Binding Key、Exchange Type** 的关系

![rabbitmq_arc2.png](/images/mq_monitoring/rabbitmq_arc2.png)



#### 概念


Item     | Comment
-------- | ---
Exchange|消息交换机，它指定消息按什么规则，路由到哪个队列
Queue|消息队列，每个消息都会被投入到一个或多个队列
Binding|绑定，它的作用就是把exchange和queue按照路由规则绑定起来
Routing Key|路由关键字，exchange根据这个关键字进行消息投递
Vhost|虚拟主机，可以开设多个vhost，用作不同用户的权限分离
Producer|消息生产者，就是投递消息的程序
Consumer|消息消费者，就是接受消息的程序
Channel|消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务



#### 投递过程

消息队列的使用过程大概如下：

* 1.客户端连接到消息队列服务器，打开一个channel
* 2.客户端声明一个exchange，并设置相关属性
* 3.客户端声明一个queue，并设置相关属性
* 4.客户端使用routing key，在exchange和queue之间建立好绑定关系
* 5.客户端投递消息到exchange
* 6.客户端从指定的queue中消费信息


---

### **rabbitmqadmin** 用法

~~~
[root@h102 rabbitmq]# rabbitmqadmin --help 
Usage
=====
  rabbitmqadmin [options] subcommand

Options
=======
--help, -h              show this help message and exit
--config=CONFIG, -c CONFIG
                        configuration file [default: ~/.rabbitmqadmin.conf]
--node=NODE, -N NODE    node described in the configuration file [default:
                        'default' only if configuration file is specified]
--host=HOST, -H HOST    connect to host HOST [default: localhost]
--port=PORT, -P PORT    connect to port PORT [default: 15672]
--path-prefix=PATH_PREFIX
                        use specific URI path prefix for the RabbitMQ HTTP API
                        (default: blank string) [default: ]
--vhost=VHOST, -V VHOST
                        connect to vhost VHOST [default: all vhosts for list,
                        '/' for declare]
--username=USERNAME, -u USERNAME
                        connect using username USERNAME [default: guest]
--password=PASSWORD, -p PASSWORD
                        connect using password PASSWORD [default: guest]
--quiet, -q             suppress status messages [default: True]
--ssl, -s               connect with ssl [default: False]
--ssl-key-file=SSL_KEY_FILE
                        PEM format key file for SSL
--ssl-cert-file=SSL_CERT_FILE
                        PEM format certificate file for SSL
--format=FORMAT, -f FORMAT
                        format for listing commands - one of [raw_json, long,
                        pretty_json, kvp, tsv, table, bash] [default: table]
--sort=SORT, -S SORT    sort key for listing queries
--sort-reverse, -R      reverse the sort order
--depth=DEPTH, -d DEPTH
                        maximum depth to recurse for listing tables [default:
                        1]
--bash-completion       Print bash completion script [default: False]
--version               Display version and exit

More Help
=========

For more help use the help subcommand:

  rabbitmqadmin help subcommands  # For a list of available subcommands
  rabbitmqadmin help config       # For help with the configuration file
[root@h102 rabbitmq]# rabbitmqadmin help subcommands
Usage
=====
  rabbitmqadmin [options] subcommand

  where subcommand is one of:

Display
=======

  list users [<column>...]
  list vhosts [<column>...]
  list connections [<column>...]
  list exchanges [<column>...]
  list bindings [<column>...]
  list permissions [<column>...]
  list channels [<column>...]
  list parameters [<column>...]
  list consumers [<column>...]
  list queues [<column>...]
  list policies [<column>...]
  list nodes [<column>...]
  show overview [<column>...]

Object Manipulation
===================

  declare queue name=... [node=... auto_delete=... durable=... arguments=...]
  declare vhost name=... [tracing=...]
  declare user name=... password=... tags=...
  declare exchange name=... type=... [auto_delete=... internal=... durable=... arguments=...]
  declare policy name=... pattern=... definition=... [priority=... apply-to=...]
  declare parameter component=... name=... value=...
  declare permission vhost=... user=... configure=... write=... read=...
  declare binding source=... destination=... [arguments=... routing_key=... destination_type=...]
  delete queue name=...
  delete vhost name=...
  delete user name=...
  delete exchange name=...
  delete policy name=...
  delete parameter component=... name=...
  delete permission vhost=... user=...
  delete binding source=... destination_type=... destination=... properties_key=...
  close connection name=...
  purge queue name=...

Broker Definitions
==================

  export <file>
  import <file>

Publishing and Consuming
========================

  publish routing_key=... [exchange=... payload=... payload_encoding=... properties=...]
  get queue=... [count=... requeue=... payload_file=... encoding=...]

  * If payload is not specified on publish, standard input is used

  * If payload_file is not specified on get, the payload will be shown on
    standard output along with the message metadata

  * If payload_file is specified on get, count must not be set

[root@h102 rabbitmq]# rabbitmqadmin help config
Usage
=====
rabbitmqadmin [options] subcommand

Configuration File
==================

  It is possible to specify a configuration file from the command line.
  Hosts can be configured easily in a configuration file and called
  from the command line.

Example
=======

  # rabbitmqadmin.conf.example START

  [host_normal]
  hostname = localhost
  port = 15672
  username = guest
  password = guest
  declare_vhost = / # Used as default for declare / delete only
  vhost = /         # Used as default for declare / delete / list

  [host_ssl]
  hostname = otherhost
  port = 15672
  username = guest
  password = guest
  ssl = True
  ssl_key_file = /path/to/key.pem
  ssl_cert_file = /path/to/cert.pem

  # rabbitmqadmin.conf.example END

Use
===

  rabbitmqadmin -c rabbitmqadmin.conf.example -N host_normal ...
[root@h102 rabbitmq]#
~~~

---

###  查看 users

~~~
[root@h102 rabbitmq]# rabbitmqadmin list users
+-------+-----------------------------+------------------------------+---------------+
| name  |      hashing_algorithm      |        password_hash         |     tags      |
+-------+-----------------------------+------------------------------+---------------+
| guest | rabbit_password_hashing_md5 | 7jG486YR6/F0hVLWYfCnqRyxKe4= | administrator |
| test  | rabbit_password_hashing_md5 | /iSnvCNZ5m3kvqoqqH4U3dSEqTM= | administrator |
+-------+-----------------------------+------------------------------+---------------+
[root@h102 rabbitmq]# rabbitmqadmin list users name
+-------+
| name  |
+-------+
| guest |
| test  |
+-------+
[root@h102 rabbitmq]# rabbitmqadmin list users tags
+---------------+
|     tags      |
+---------------+
| administrator |
| administrator |
+---------------+
[root@h102 rabbitmq]#
~~~

---

### 查看 vhosts

~~~
[root@h102 rabbitmq]# rabbitmqadmin list vhosts
+------+----------+
| name | messages |
+------+----------+
| /    | 13       |
+------+----------+
[root@h102 rabbitmq]#
~~~

---

### 查看 connections


~~~
[root@h102 rabbitmq]# rabbitmqadmin list connections
No items
[root@h102 rabbitmq]# 
~~~


---

### 查看 exchanges


~~~
[root@h102 rabbitmq]# rabbitmqadmin list exchanges
+--------------------+---------+
|        name        |  type   |
+--------------------+---------+
|                    | direct  |
| amq.direct         | direct  |
| amq.fanout         | fanout  |
| amq.headers        | headers |
| amq.match          | headers |
| amq.rabbitmq.log   | topic   |
| amq.rabbitmq.trace | topic   |
| amq.topic          | topic   |
| kk                 | fanout  |
| test               | fanout  |
+--------------------+---------+
[root@h102 rabbitmq]#
~~~

---

### 查看 bindings


~~~
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+--------+-------------+-------------+
| source | destination | routing_key |
+--------+-------------+-------------+
|        | hello       | hello       |
|        | test        | test        |
| kk     | hello       | hello       |
| kk     | test        | test        |
+--------+-------------+-------------+
[root@h102 rabbitmq]# 
~~~


---

### 查看 permissions

~~~
[root@h102 rabbitmq]# rabbitmqadmin list permissions
+-------+-----------+------+-------+-------+
| vhost | configure | read | user  | write |
+-------+-----------+------+-------+-------+
| /     | .*        | .*   | guest | .*    |
+-------+-----------+------+-------+-------+
[root@h102 rabbitmq]# rabbitmqadmin list permissions read
+------+
| read |
+------+
| .*   |
+------+
[root@h102 rabbitmq]# 
~~~


---

### 查看 channels


~~~
[root@h102 rabbitmq]# rabbitmqadmin list channels
No items
[root@h102 rabbitmq]# 
~~~


---

### 查看 parameters

~~~
[root@h102 rabbitmq]# rabbitmqadmin list parameters
No items
[root@h102 rabbitmq]#
~~~


---


### 查看 consumers

~~~
[root@h102 rabbitmq]# rabbitmqadmin list consumers
No items
[root@h102 rabbitmq]# 
~~~


---

### 查看 queues

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------+----------+
| name  | messages |
+-------+----------+
| hello | 8        |
| test  | 5        |
+-------+----------+
[root@h102 rabbitmq]# 
~~~


---

### 查看 policies


~~~
[root@h102 rabbitmq]# rabbitmqadmin list policies
No items
[root@h102 rabbitmq]# 
~~~

---

### 查看 nodes

~~~
[root@h102 rabbitmq]# rabbitmqadmin list nodes
+-------------+------+----------+
|    name     | type | mem_used |
+-------------+------+----------+
| rabbit@h102 | disc | 52274376 |
+-------------+------+----------+
[root@h102 rabbitmq]# 
~~~


---

### 查看 overview

~~~
[root@h102 rabbitmq]# rabbitmqadmin show overview
+------------------+------------------+-----------------------+----------------------+
| rabbitmq_version |   cluster_name   | queue_totals.messages | object_totals.queues |
+------------------+------------------+-----------------------+----------------------+
| 3.6.1            | rabbit@h101.temp | 13                    | 2                    |
+------------------+------------------+-----------------------+----------------------+
[root@h102 rabbitmq]# 
~~~

---

### 删除 queue

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------+----------+
| name  | messages |
+-------+----------+
| hello | 8        |
| test  | 5        |
+-------+----------+
[root@h102 rabbitmq]# rabbitmqadmin delete queue name=hello
queue deleted
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 5        |
+------+----------+
[root@h102 rabbitmq]# 
~~~


---

### 删除 user

~~~
[root@h102 rabbitmq]# rabbitmqadmin list users
+-------+-----------------------------+------------------------------+---------------+
| name  |      hashing_algorithm      |        password_hash         |     tags      |
+-------+-----------------------------+------------------------------+---------------+
| guest | rabbit_password_hashing_md5 | 7jG486YR6/F0hVLWYfCnqRyxKe4= | administrator |
| test  | rabbit_password_hashing_md5 | /iSnvCNZ5m3kvqoqqH4U3dSEqTM= | administrator |
+-------+-----------------------------+------------------------------+---------------+
[root@h102 rabbitmq]# rabbitmqadmin delete user name=test
user deleted
[root@h102 rabbitmq]# rabbitmqadmin list users
+-------+-----------------------------+------------------------------+---------------+
| name  |      hashing_algorithm      |        password_hash         |     tags      |
+-------+-----------------------------+------------------------------+---------------+
| guest | rabbit_password_hashing_md5 | 7jG486YR6/F0hVLWYfCnqRyxKe4= | administrator |
+-------+-----------------------------+------------------------------+---------------+
[root@h102 rabbitmq]# 
~~~


---

### 删除 exchange

~~~
[root@h102 rabbitmq]# rabbitmqadmin list exchanges
+--------------------+---------+
|        name        |  type   |
+--------------------+---------+
|                    | direct  |
| amq.direct         | direct  |
| amq.fanout         | fanout  |
| amq.headers        | headers |
| amq.match          | headers |
| amq.rabbitmq.log   | topic   |
| amq.rabbitmq.trace | topic   |
| amq.topic          | topic   |
| kk                 | fanout  |
| test               | fanout  |
+--------------------+---------+
[root@h102 rabbitmq]# rabbitmqadmin delete exchange name=test
exchange deleted
[root@h102 rabbitmq]# rabbitmqadmin list exchanges
+--------------------+---------+
|        name        |  type   |
+--------------------+---------+
|                    | direct  |
| amq.direct         | direct  |
| amq.fanout         | fanout  |
| amq.headers        | headers |
| amq.match          | headers |
| amq.rabbitmq.log   | topic   |
| amq.rabbitmq.trace | topic   |
| amq.topic          | topic   |
| kk                 | fanout  |
+--------------------+---------+
[root@h102 rabbitmq]# 
~~~


---

### 删除 binding


~~~
[root@h102 rabbitmq]# rabbitmqadmin list bindings  source destination_type destination properties_key
+--------+------------------+-------------+----------------+
| source | destination_type | destination | properties_key |
+--------+------------------+-------------+----------------+
|        | queue            | test        | test           |
| kk     | queue            | test        | test           |
+--------+------------------+-------------+----------------+
[root@h102 rabbitmq]# rabbitmqadmin delete binding source='kk'  destination_type=queue  destination=test  properties_key=test
binding deleted
[root@h102 rabbitmq]# rabbitmqadmin list bindings source destination_type destination properties_key
+--------+------------------+-------------+----------------+
| source | destination_type | destination | properties_key |
+--------+------------------+-------------+----------------+
|        | queue            | test        | test           |
+--------+------------------+-------------+----------------+
[root@h102 rabbitmq]# 
~~~

> **Note:**  **source、 destination_type、destination、properties_key** 缺一不可，否则会报错




> **Tip:**  **vhost、policy、parameter、permission** 的删除方法类似 ，**connection** 的关闭方法也类似




---

###  清空队列

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 5        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin purge queue name=test
queue purged
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]#
~~~


---

### 格式化输出

使用 **`-f`** 可以指定格式

有如下几种格式 **raw_json, long, pretty_json, kvp, tsv, table, bash**

默认为 **table**

~~~
[root@h102 rabbitmq]# rabbitmqadmin list users
+-------+-----------------------------+------------------------------+---------------+
| name  |      hashing_algorithm      |        password_hash         |     tags      |
+-------+-----------------------------+------------------------------+---------------+
| guest | rabbit_password_hashing_md5 | 7jG486YR6/F0hVLWYfCnqRyxKe4= | administrator |
+-------+-----------------------------+------------------------------+---------------+
[root@h102 rabbitmq]# rabbitmqadmin -f raw_json list users
[{"name":"guest","password_hash":"7jG486YR6/F0hVLWYfCnqRyxKe4=","hashing_algorithm":"rabbit_password_hashing_md5","tags":"administrator"}]
[root@h102 rabbitmq]# rabbitmqadmin -f long list users

--------------------------------------------------------------------------------

             name: guest
hashing_algorithm: rabbit_password_hashing_md5
    password_hash: 7jG486YR6/F0hVLWYfCnqRyxKe4=
             tags: administrator

--------------------------------------------------------------------------------

[root@h102 rabbitmq]# rabbitmqadmin -f pretty_json list users
[
  {
    "hashing_algorithm": "rabbit_password_hashing_md5", 
    "name": "guest", 
    "password_hash": "7jG486YR6/F0hVLWYfCnqRyxKe4=", 
    "tags": "administrator"
  }
]
[root@h102 rabbitmq]# rabbitmqadmin -f kvp list users
name="guest" hashing_algorithm="rabbit_password_hashing_md5" password_hash="7jG486YR6/F0hVLWYfCnqRyxKe4=" tags="administrator"
[root@h102 rabbitmq]# rabbitmqadmin -f tsv list users
name	hashing_algorithm	password_hash	tags
guest	rabbit_password_hashing_md5	7jG486YR6/F0hVLWYfCnqRyxKe4=	administrator
[root@h102 rabbitmq]# rabbitmqadmin -f table list users
+-------+-----------------------------+------------------------------+---------------+
| name  |      hashing_algorithm      |        password_hash         |     tags      |
+-------+-----------------------------+------------------------------+---------------+
| guest | rabbit_password_hashing_md5 | 7jG486YR6/F0hVLWYfCnqRyxKe4= | administrator |
+-------+-----------------------------+------------------------------+---------------+
[root@h102 rabbitmq]# rabbitmqadmin -f bash list users
guest
[root@h102 rabbitmq]#
~~~


---

### 定义一个 queue

~~~
[root@h102 rabbitmq]# rabbitmqadmin list bindings
No items
[root@h102 rabbitmq]# rabbitmqadmin list queues
No items
[root@h102 rabbitmq]# rabbitmqadmin declare queue name=test  durable=true
queue declared
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+--------+-------------+-------------+
| source | destination | routing_key |
+--------+-------------+-------------+
|        | test        | test        |
+--------+-------------+-------------+
[root@h102 rabbitmq]#
~~~

**durable=true** 代表持久化打开

发现定义一个新的queue后，RabbitMQ会自动为之创建一个 **binding**

---

### 发布一条消息

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=test payload="just for test"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 1        |
+------+----------+
[root@h102 rabbitmq]#
~~~

> **Tip:**  这里有一个细节，我们并未指定任何 exchange ，依旧可以成功发送消息，是不是不需要 exchange 参与整个消息的派送过程呢，这和前面说的信息处理流程貌似有冲突呀，先卖个关子，后面再解释


---

### 消费一条信息


~~~
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=true
+-------------+----------+---------------+---------------+---------------+------------------+------------+-------------+
| routing_key | exchange | message_count |    payload    | payload_bytes | payload_encoding | properties | redelivered |
+-------------+----------+---------------+---------------+---------------+------------------+------------+-------------+
| test        |          | 0             | just for test | 13            | string           |            | False       |
+-------------+----------+---------------+---------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 1        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=false
+-------------+----------+---------------+---------------+---------------+------------------+------------+-------------+
| routing_key | exchange | message_count |    payload    | payload_bytes | payload_encoding | properties | redelivered |
+-------------+----------+---------------+---------------+---------------+------------------+------------+-------------+
| test        |          | 0             | just for test | 13            | string           |            | True        |
+-------------+----------+---------------+---------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]#
~~~


---

### 定义一个 exchange

我们前面发布消息的过程中并未指定exchange，依旧成功发布了，事实上只是未明确指出，系统还是帮我们指了，RabbitMQ 的逻辑中是没法绕过 exchange 而直接给queue发送消息的

>In RabbitMQ a message can never be sent directly to the queue, it always needs to go through an exchange. 


>The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Actually, quite often the producer doesn't even know if a message will be delivered to any queue at all.


exchange 就是用来进行信息交换的逻辑

>An exchange is a very simple thing. On one side it receives messages from producers and the other side it pushes them to queues. 

exchange 的类型决定了其行为特性

>The exchange must know exactly what to do with a message it receives. Should it be appended to a particular queue? Should it be appended to many queues? Or should it get discarded. The rules for that are defined by the exchange type

exchange 有以下几种类型

* direct
* topic
* headers 
* fanout


**Fanout、Direct、Topic** 三种 **Exchange Type** 的区别

![exchange_type.jpg](/images/mq_monitoring/exchange_type.jpg)

* Fanout ：进行最简单的广播
* Direct ： 最直接的根据整个 routing key 对应
* Topic ： 可以使用点分 routing key 的模糊匹配

> \* (star) can substitute for exactly one word
>
> \# (hash) can substitute for zero or more words


**headers** 以后再探讨


系统中默认就有如下 exchange

~~~
[root@h102 rabbitmq]# rabbitmqadmin list exchanges
+--------------------+---------+
|        name        |  type   |
+--------------------+---------+
|                    | direct  |
| amq.direct         | direct  |
| amq.fanout         | fanout  |
| amq.headers        | headers |
| amq.match          | headers |
| amq.rabbitmq.log   | topic   |
| amq.rabbitmq.trace | topic   |
| amq.topic          | topic   |
+--------------------+---------+
[root@h102 rabbitmq]#
~~~

我们创建了一个 queue 后，系统自动为它定义了一个 binding

~~~
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+--------+-------------+-------------+
| source | destination | routing_key |
+--------+-------------+-------------+
|        | test        | test        |
+--------+-------------+-------------+
[root@h102 rabbitmq]#
~~~

从中我们看出了一些端倪，我们不手动指定 exchange时，使用的默认 exchange是空字符串(系统中的第一个exchange，binding中的source部分)，并且这个默认的exchange是 **direct** 类型，这种隐式调用确保了我的消息准确投递



这里再定义三个exchange 分属三种类型

~~~
[root@h102 rabbitmq]# rabbitmqadmin list exchanges
+--------------------+---------+
|        name        |  type   |
+--------------------+---------+
|                    | direct  |
| amq.direct         | direct  |
| amq.fanout         | fanout  |
| amq.headers        | headers |
| amq.match          | headers |
| amq.rabbitmq.log   | topic   |
| amq.rabbitmq.trace | topic   |
| amq.topic          | topic   |
+--------------------+---------+
[root@h102 rabbitmq]# rabbitmqadmin declare exchange name=my.fanout type=fanout
exchange declared
[root@h102 rabbitmq]# rabbitmqadmin declare exchange name=my.direct type=direct
exchange declared
[root@h102 rabbitmq]# rabbitmqadmin declare exchange name=my.topic type=topic
exchange declared
[root@h102 rabbitmq]# rabbitmqadmin list exchanges
+--------------------+---------+
|        name        |  type   |
+--------------------+---------+
|                    | direct  |
| amq.direct         | direct  |
| amq.fanout         | fanout  |
| amq.headers        | headers |
| amq.match          | headers |
| amq.rabbitmq.log   | topic   |
| amq.rabbitmq.trace | topic   |
| amq.topic          | topic   |
| my.direct          | direct  |
| my.fanout          | fanout  |
| my.topic           | topic   |
+--------------------+---------+
[root@h102 rabbitmq]#
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+--------+-------------+-------------+
| source | destination | routing_key |
+--------+-------------+-------------+
|        | test        | test        |
+--------+-------------+-------------+
[root@h102 rabbitmq]# 
~~~

---

### 定义 binding

上面申明了几个 exchange ，尝试发布一条信息

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=test exchange=my.fanout  payload="just for test"
Message published but NOT routed
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=test payload="just for test2"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 1        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+--------+-------------+-------------+
| source | destination | routing_key |
+--------+-------------+-------------+
|        | test        | test        |
+--------+-------------+-------------+
[root@h102 rabbitmq]# 
~~~

指定 **exchange=my.fanout** 后，报 **Message published but NOT routed** ，然后检查queue 发现并产生新消息，而直接不指定却成功发布了，原因是没有binding，exchange 并不知道将数据转发给谁

**binding** 定义了 exchange 与 queue 的关系，并且限定了路由的部分规则

信息路由规则一部分由 exchange的类型决定 ，一部分由 binding 关系决定，binding 的 key 还能起到甄选信息的作用

>A binding is a relationship between an exchange and a queue. This can be simply read as: the queue is interested in messages from this exchange.Bindings can take an extra routing_key parameter. To avoid the confusion with a basic_publish parameter we're going to call it a binding key.


> **Tip:**  对于类型为 **fanout** 的 exchange 比较特别，binding 的 **routing_key** 会被忽略，直接被 **广播**

>The meaning of a binding key depends on the exchange type. The fanout exchanges, which we used previously, simply ignored its value.


我们创建一个 binding


~~~
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+--------+-------------+-------------+
| source | destination | routing_key |
+--------+-------------+-------------+
|        | test        | test        |
+--------+-------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.fanout destination=test routing_key=first
binding declared
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+-----------+-------------+-------------+
|  source   | destination | routing_key |
+-----------+-------------+-------------+
|           | test        | test        |
| my.fanout | test        | first       |
+-----------+-------------+-------------+
[root@h102 rabbitmq]#
~~~


再次尝试发布信息


~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 0        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=true
No items
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=first exchange=my.fanout  payload="just for test1"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 1        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=first payload="just for test2"
Message published but NOT routed
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 1        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| first       | my.fanout | 0             | just for test1 | 14            | string           |            | False       |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# 
~~~

可以成功发布，然后我尝试了不指定 exchange，只指定 **routing_key=first** ，发现报 **Message published but NOT routed** ，说明不指定 exchange 的情况下，系统又默认去找为空的 exchange，并且在它下面去找 **routing_key=first** 的binding ,但是没找到，于是信息路由不成功


---

### **fanout** 的特性

定义第二个queue ，也使用 my.fanout binding 起来，取 routing_key 为 second

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+------+----------+
| name | messages |
+------+----------+
| test | 1        |
+------+----------+
[root@h102 rabbitmq]# rabbitmqadmin declare queue name=test.fanout durable=true
queue declared
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 1        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+-----------+-------------+-------------+
|  source   | destination | routing_key |
+-----------+-------------+-------------+
|           | test        | test        |
|           | test.fanout | test.fanout |
| my.fanout | test        | first       |
+-----------+-------------+-------------+
[root@h102 rabbitmq]# 
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.fanout destination=test.fanout routing_key=second
binding declared
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+-----------+-------------+-------------+
|  source   | destination | routing_key |
+-----------+-------------+-------------+
|           | test        | test        |
|           | test.fanout | test.fanout |
| my.fanout | test        | first       |
| my.fanout | test.fanout | second      |
+-----------+-------------+-------------+
[root@h102 rabbitmq]#
~~~


尝试再发一条数据到 **my.fanout** 中

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 1        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=second exchange=my.fanout payload="just for test3"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 2        |
| test.fanout | 1        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| first       | my.fanout | 1             | just for test1 | 14            | string           |            | True        |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test.fanout requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| second      | my.fanout | 0             | just for test3 | 14            | string           |            | False       |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 2        |
| test.fanout | 1        |
+-------------+----------+
[root@h102 rabbitmq]# 
~~~


这次我们使用 **routing_key=first** 来投递消息

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 2        |
| test.fanout | 1        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin purge queue name=test
queue purged
[root@h102 rabbitmq]# rabbitmqadmin purge queue name=test.fanout
queue purged
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=first exchange=my.fanout payload="just for test4"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 1        |
| test.fanout | 1        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| first       | my.fanout | 0             | just for test4 | 14            | string           |            | False       |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test.fanout requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| first       | my.fanout | 0             | just for test4 | 14            | string           |            | False       |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# 
~~~

发现结果一样，应证了前面说的 **routing_key** 会被忽略的说法，但是不能不指定，否则会报错


~~~
[root@h102 rabbitmq]# rabbitmqadmin publish exchange=my.fanout payload="just for test5"

ERROR: mandatory argument "routing_key" required

rabbitmqadmin --help for help

[root@h102 rabbitmq]#
~~~

---

### **direct** 的特性


定义第三个queue ，使用 my.direct binding 起来


~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin declare queue name=test.direct durable=true
queue declared
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.direct | 0        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.direct destination=test routing_key=third
binding declared
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.direct destination=test.direct routing_key=fourth
binding declared
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+-----------+-------------+-------------+
|  source   | destination | routing_key |
+-----------+-------------+-------------+
|           | test        | test        |
|           | test.direct | test.direct |
|           | test.fanout | test.fanout |
| my.direct | test        | third       |
| my.direct | test.direct | fourth      |
| my.fanout | test        | first       |
| my.fanout | test.fanout | second      |
+-----------+-------------+-------------+
[root@h102 rabbitmq]# 
~~~


尝试分别使用 **third** 和 **fourth** 的 **routing_key** 来发布消息


~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.direct | 0        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=third exchange=my.direct payload="just for test6"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 1        |
| test.direct | 0        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| third       | my.direct | 0             | just for test6 | 14            | string           |            | False       |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=fourth exchange=my.direct payload="just for test7"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 1        |
| test.direct | 1        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin get queue=test.direct requeue=true
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| routing_key | exchange  | message_count |    payload     | payload_bytes | payload_encoding | properties | redelivered |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
| fourth      | my.direct | 0             | just for test7 | 14            | string           |            | False       |
+-------------+-----------+---------------+----------------+---------------+------------------+------------+-------------+
[root@h102 rabbitmq]# 
~~~

从反馈结果来看，**direct** 的 exchange 就像点对点通信，**fanout** 的 exchange 就像是广播


---

### **topic** 的特性


定义第四个queue ，使用 my.topic binding 起来

~~~
[root@h102 rabbitmq]# rabbitmqadmin purge queue name=test
queue purged
[root@h102 rabbitmq]# rabbitmqadmin purge queue name=test.direct
queue purged
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.direct | 0        |
| test.fanout | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin declare queue name=test.topic durable=true
queue declared
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.direct | 0        |
| test.fanout | 0        |
| test.topic  | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.topic destination=test routing_key=*.hard.* 
binding declared
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.topic destination=test.topic routing_key=cheap.# 
binding declared
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.topic destination=test.direct routing_key=*.*.food 
binding declared
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.topic destination=test.fanout routing_key=*.*.food
binding declared
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+-----------+-------------+-------------+
|  source   | destination | routing_key |
+-----------+-------------+-------------+
|           | test        | test        |
|           | test.direct | test.direct |
|           | test.fanout | test.fanout |
|           | test.topic  | test.topic  |
| my.direct | test        | third       |
| my.direct | test.direct | fourth      |
| my.fanout | test        | first       |
| my.fanout | test.fanout | second      |
| my.topic  | test        | *.hard.*    |
| my.topic  | test.direct | *.*.food    |
| my.topic  | test.fanout | *.*.food    |
| my.topic  | test.topic  | cheap.#     |
+-----------+-------------+-------------+
[root@h102 rabbitmq]# 
~~~

如果不使用 **\* 和  \#** ，那么 topic 的特性就和 direct 一样了


~~~
[root@h102 rabbitmq]# rabbitmqadmin declare binding source=my.topic destination=test.fanout routing_key=xtest
binding declared
[root@h102 rabbitmq]# rabbitmqadmin list bindings
+-----------+-------------+-------------+
|  source   | destination | routing_key |
+-----------+-------------+-------------+
|           | test        | test        |
|           | test.direct | test.direct |
|           | test.fanout | test.fanout |
|           | test.topic  | test.topic  |
| my.direct | test        | third       |
| my.direct | test.direct | fourth      |
| my.fanout | test        | first       |
| my.fanout | test.fanout | second      |
| my.topic  | test        | *.hard.*    |
| my.topic  | test.direct | *.*.food    |
| my.topic  | test.fanout | *.*.food    |
| my.topic  | test.fanout | xtest       |
| my.topic  | test.topic  | cheap.#     |
+-----------+-------------+-------------+
[root@h102 rabbitmq]#
~~~


尝试使用以上 **routing_key** 发送消息

~~~
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 0        |
| test.direct | 0        |
| test.fanout | 0        |
| test.topic  | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=a.hard.b  exchange=my.topic payload="just for test8"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 1        |
| test.direct | 0        |
| test.fanout | 0        |
| test.topic  | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=a.hard.food  exchange=my.topic payload="just for test9"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 2        |
| test.direct | 1        |
| test.fanout | 1        |
| test.topic  | 0        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=cheap.soft.food  exchange=my.topic payload="just for test10"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 2        |
| test.direct | 2        |
| test.fanout | 2        |
| test.topic  | 1        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=cheap.hard.drink  exchange=my.topic payload="just for test11"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 3        |
| test.direct | 2        |
| test.fanout | 2        |
| test.topic  | 2        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=xtest exchange=my.topic payload="just for test12"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 3        |
| test.direct | 2        |
| test.fanout | 3        |
| test.topic  | 2        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin publish routing_key=cheap.hard.food  exchange=my.topic payload="just for test13"
Message published
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 4        |
| test.direct | 2        |
| test.fanout | 4        |
| test.topic  | 3        |
+-------------+----------+
[root@h102 rabbitmq]# rabbitmqadmin list queues
+-------------+----------+
|    name     | messages |
+-------------+----------+
| test        | 4        |
| test.direct | 3        |
| test.fanout | 4        |
| test.topic  | 3        |
+-------------+----------+
[root@h102 rabbitmq]# 
~~~

到此为止，我们看到了 **topic** 的组播，异步特性





---

# 命令汇总


* **`rpm -ivh rabbitmq-server-3.6.1-1.noarch.rpm`**
* **`wget https://packages.erlang-solutions.com/erlang/esl-erlang/FLAVOUR_1_general/esl-erlang_18.3-1~centos~6_amd64.rpm`**
* **`rpm -ivh esl-erlang_18.3-1~centos~6_amd64.rpm`**
* **`rpm -e erlang-erts-R14B-04.3.el6.x86_64`**
* **`wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm`**
* **`rpm -ivh erlang-solutions-1.0-1.noarch.rpm`**
* **`cat /etc/yum.repos.d/erlang_solutions.repo`**
* **`yum update erlang.x86_64`**
* **`wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm`**
* **`rpm -ivh rabbitmq-server-3.6.1-1.noarch.rpm`**
* **`/etc/init.d/rabbitmq-server start`**
* **`netstat  -an | grep -E "(4369|25672|5672|5671|15672|61613|61614|1883|8883)"`**
* **`rabbitmq-plugins list`**
* **`netstat  -ant | grep 15672`**
* **`rabbitmq-plugins enable rabbitmq_management`**
* **`wget http://localhost:15672/cli/rabbitmqadmin`**
* **`file rabbitmqadmin`**
* **`chmod  +x rabbitmqadmin`**
* **`rabbitmqadmin --help`**
* **`rabbitmqadmin help subcommands`**
* **`rabbitmqadmin help config`**
* **`rabbitmqadmin list users`**
* **`rabbitmqadmin list users name`**
* **`rabbitmqadmin list users tags`**
* **`rabbitmqadmin list vhosts`**
* **`rabbitmqadmin list connections`**
* **`rabbitmqadmin list exchanges`**
* **`rabbitmqadmin list bindings`**
* **`rabbitmqadmin list permissions`**
* **`rabbitmqadmin list permissions read`**
* **`rabbitmqadmin list channels`**
* **`rabbitmqadmin list parameters`**
* **`rabbitmqadmin list consumers`**
* **`rabbitmqadmin list queues`**
* **`rabbitmqadmin list policies`**
* **`rabbitmqadmin list nodes`**
* **`rabbitmqadmin show overview`**
* **`rabbitmqadmin delete queue name=hello`**
* **`rabbitmqadmin delete user name=test`**
* **`rabbitmqadmin delete exchange name=test`**
* **`rabbitmqadmin delete binding source='kk'  destination_type=queue  destination=test  properties_key=test`**
* **`rabbitmqadmin list bindings source destination_type destination properties_key`**
* **`rabbitmqadmin purge queue name=test`**
* **`rabbitmqadmin -f raw_json list users`**
* **`rabbitmqadmin -f long list users`**
* **`rabbitmqadmin -f pretty_json list users`**
* **`rabbitmqadmin -f kvp list users`**
* **`rabbitmqadmin -f tsv list users`**
* **`rabbitmqadmin -f table list users`**
* **`rabbitmqadmin -f bash list users`**
* **`rabbitmqadmin declare queue name=test  durable=true`**
* **`rabbitmqadmin publish routing_key=test payload="just for test"`**
* **`rabbitmqadmin get queue=test requeue=true`**
* **`rabbitmqadmin get queue=test requeue=false`**
* **`rabbitmqadmin declare exchange name=my.fanout type=fanout`**
* **`rabbitmqadmin declare exchange name=my.direct type=direct`**
* **`rabbitmqadmin declare exchange name=my.topic type=topic`**
* **`rabbitmqadmin publish routing_key=test exchange=my.fanout  payload="just for test"`**
* **`rabbitmqadmin publish routing_key=test payload="just for test2"`**
* **`rabbitmqadmin declare binding source=my.fanout destination=test routing_key=first`**
* **`rabbitmqadmin publish routing_key=first exchange=my.fanout  payload="just for test1"`**
* **`rabbitmqadmin publish routing_key=first payload="just for test2"`**
* **`rabbitmqadmin declare queue name=test.fanout durable=true`**
* **`rabbitmqadmin declare binding source=my.fanout destination=test.fanout routing_key=second`**
* **`rabbitmqadmin publish routing_key=second exchange=my.fanout payload="just for test3"`**
* **`rabbitmqadmin get queue=test.fanout requeue=true`**
* **`rabbitmqadmin purge queue name=test`**
* **`rabbitmqadmin purge queue name=test.fanout`**
* **`rabbitmqadmin publish routing_key=first exchange=my.fanout payload="just for test4"`**
* **`rabbitmqadmin get queue=test requeue=true`**
* **`rabbitmqadmin get queue=test.fanout requeue=true`**
* **`rabbitmqadmin publish exchange=my.fanout payload="just for test5"`**
* **`rabbitmqadmin declare queue name=test.direct durable=true`**
* **`rabbitmqadmin declare binding source=my.direct destination=test routing_key=third`**
* **`rabbitmqadmin declare binding source=my.direct destination=test.direct routing_key=fourth`**
* **`rabbitmqadmin publish routing_key=third exchange=my.direct payload="just for test6"`**
* **`rabbitmqadmin publish routing_key=fourth exchange=my.direct payload="just for test7"`**
* **`rabbitmqadmin get queue=test.direct requeue=true`**
* **`rabbitmqadmin purge queue name=test`**
* **`rabbitmqadmin purge queue name=test.direct`**
* **`rabbitmqadmin declare queue name=test.topic durable=true`**
* **`rabbitmqadmin declare binding source=my.topic destination=test routing_key=*.hard.*`**
* **`rabbitmqadmin declare binding source=my.topic destination=test.topic routing_key=cheap.#`**
* **`rabbitmqadmin declare binding source=my.topic destination=test.direct routing_key=*.*.food`**
* **`rabbitmqadmin declare binding source=my.topic destination=test.fanout routing_key=*.*.food`**
* **`rabbitmqadmin declare binding source=my.topic destination=test.fanout routing_key=xtest`**
* **`rabbitmqadmin publish routing_key=a.hard.b  exchange=my.topic payload="just for test8"`**
* **`rabbitmqadmin publish routing_key=a.hard.food  exchange=my.topic payload="just for test9"`**
* **`rabbitmqadmin publish routing_key=cheap.soft.food  exchange=my.topic payload="just for test10"`**
* **`rabbitmqadmin publish routing_key=cheap.hard.drink  exchange=my.topic payload="just for test11"`**
* **`rabbitmqadmin publish routing_key=xtest exchange=my.topic payload="just for test12"`**
* **`rabbitmqadmin publish routing_key=cheap.hard.food  exchange=my.topic payload="just for test13"`**



---

[rabbitmq]:http://www.rabbitmq.com/
[rabbitmq_monitoring]:http://soft.dog/2016/01/13/rabbitmq-monitoring/
[management_cli]:http://www.rabbitmq.com/management-cli.html
[erlang_down]:https://www.erlang-solutions.com/resources/download.html

