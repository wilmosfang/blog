---
layout: post
title: RabbitMQ管理
categories: linux
excerpt: follow me
comments: true
---


---

#前言


**[RabbitMQ][rabbitmq]** 是一款开源的消息代理服务器，用来进行信息路由。

MQ可以使架构变得松耦合，从而更有弹性，更灵活，是SOA架构不可或缺的组成部分，担当服务总线或信息总线的角色。

下面分享一下 RabbitMQ 的基本管理操作，详细可以参阅 [官方文档][rabbitmq_doc]


> **Tip:** 目前官方版本是 **RabbitMQ 3.5.6 release** 
  

---


#概要

* TOC
{:toc}




---


##用户管理

###列出用户


**list_users** 会返回所有用户

{% highlight bash %}
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
[root@h101 ~]# 
{% endhighlight %}

---

###添加用户


{% highlight bash %}
[root@h101 ~]# rabbitmqctl  add_user test testpass
Creating user "test" ...
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
test	[]
[root@h101 ~]# 
{% endhighlight %}

---

###修改用户密码

{% highlight bash %}
[root@h101 ~]# rabbitmqctl change_password test changetonew
Changing password for user "test" ...
[root@h101 ~]# 
{% endhighlight %}

---

###清除密码


**clear_password** 可以清除指定用户密码，被清除密码的用户将无法使用密码登录

{% highlight bash %}
[root@h101 ~]# rabbitmqctl clear_password test
Clearing password for user "test" ...
[root@h101 ~]# 
{% endhighlight %}


---

###给用户打标


**set_user_tags** 可以将用户设定为管理员

{% highlight bash %}
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
test	[]
[root@h101 ~]# rabbitmqctl set_user_tags test administrator
Setting tags for user "test" to [administrator] ...
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
test	[administrator]
[root@h101 ~]# 
{% endhighlight %}

回收标记

{% highlight bash %}
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
test	[administrator]
[root@h101 ~]# rabbitmqctl set_user_tags test 
Setting tags for user "test" to [] ...
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
test	[]
[root@h101 ~]#
{% endhighlight %}

> **Note:** 可以一次设定多个标记，此命令只会以最新一次的设定为准，之前的设置会被覆盖，所以要作好记录，以便恢复

{% highlight bash %}
[root@h101 ~]# rabbitmqctl set_user_tags test  ui,ii,uiui
Setting tags for user "test" to ['ui,ii,uiui'] ...
[root@h101 ~]# rabbitmqctl  list_users
Listing users ...
guest	[administrator]
test	[ui,ii,uiui]
[root@h101 ~]# 
{% endhighlight %}

---

###删除用户

{% highlight bash %}
[root@h101 ~]# rabbitmqctl list_users
Listing users ...
guest	[administrator]
test	[]
[root@h101 ~]# rabbitmqctl delete_user test
Deleting user "test" ...
[root@h101 ~]# rabbitmqctl list_users
Listing users ...
guest	[administrator]
[root@h101 ~]#
{% endhighlight %}

---

##访问控制


RabbitMQ里有一个vhost的概念，和其它软件中的vhost不太一样，在Apache中是表示一个虚拟的站点，而在这里是表示一个命名空间和权限集合

一个vhost中包含有一堆的exchange，binding，queue，permission，parameter 和policie元素，对一个vhost拥有权限，就意味着对其下的这些元素有相应操作权限，它的设定是为了方便权限分配和隔离

系统中默认带有一个名为 `/` 的vhost

不同应用，最好使用不同的vhost进行隔离


---


###列出vhost

{% highlight bash %}
[root@h102 ~]# rabbitmqctl  list_vhosts
Listing vhosts ...
/
[root@h102 ~]# rabbitmqctl  list_vhosts tracing name
Listing vhosts ...
false	/
[root@h102 ~]# 
{% endhighlight %}

---

###添加vhost

{% highlight bash %}
[root@h102 ~]# rabbitmqctl list_vhosts
Listing vhosts ...
/
[root@h102 ~]# rabbitmqctl add_vhost mq_test
Creating vhost "mq_test" ...
[root@h102 ~]# rabbitmqctl  list_vhosts
Listing vhosts ...
/
mq_test
[root@h102 ~]# rabbitmqctl add_vhost /abc
Creating vhost "/abc" ...
[root@h102 ~]# rabbitmqctl  list_vhosts
Listing vhosts ...
/
/abc
mq_test
[root@h102 ~]#
{% endhighlight %}

---

###查看vhost中的权限分配


不使用 **`-p`** 指定vhost时，默认会使用 **`/`**

{% highlight bash %}
[root@h102 ~]# rabbitmqctl list_permissions
Listing permissions in vhost "/" ...
guest	.*	.*	.*
[root@h102 ~]# rabbitmqctl list_permissions -p /abc
Listing permissions in vhost "/abc" ...
[root@h102 ~]# rabbitmqctl list_permissions -p mq_test
Listing permissions in vhost "mq_test" ...
[root@h102 ~]# 
{% endhighlight %}


---

###查看用户权限


**list_user_permissions** 可以查看指定用户在不同vhost中的权限

{% highlight bash %}
[root@h102 ~]# rabbitmqctl list_users
Listing users ...
guest	[administrator]
mq	[]
[root@h102 ~]# rabbitmqctl list_user_permissions 
Error: list_user_permissions expects a username argument, but none provided.
[root@h102 ~]# rabbitmqctl list_user_permissions guest
Listing permissions for user "guest" ...
/	.*	.*	.*
[root@h102 ~]# rabbitmqctl list_user_permissions mq
Listing permissions for user "mq" ...
[root@h102 ~]# 
{% endhighlight %}

---

###分配权限

{% highlight bash %}
[root@h102 ~]# rabbitmqctl set_permissions -p mq_test mq ".*" ".*" ".*"
Setting permissions for user "mq" in vhost "mq_test" ...
[root@h102 ~]# rabbitmqctl set_permissions -p / mq "^mq.*" ".*" ".*"
Setting permissions for user "mq" in vhost "/" ...
[root@h102 ~]# rabbitmqctl list_user_permissions mq
Listing permissions for user "mq" ...
/	^mq.*	.*	.*
mq_test	.*	.*	.*
[root@h102 ~]# rabbitmqctl list_permissions -p /
Listing permissions in vhost "/" ...
guest	.*	.*	.*
mq	^mq.*	.*	.*
[root@h102 ~]# 
{% endhighlight %}


---

###收回权限


不使用 **`-p`** 指定vhost时，默认会使用 **`/`** ，而不是清除所有

{% highlight bash %}
[root@h102 ~]# rabbitmqctl list_user_permissions mq
Listing permissions for user "mq" ...
/	^mq.*	.*	.*
mq_test	.*	.*	.*
[root@h102 ~]# rabbitmqctl clear_permissions -p / mq 
Clearing permissions for user "mq" in vhost "/" ...
[root@h102 ~]# rabbitmqctl list_permissions -p /
Listing permissions in vhost "/" ...
guest	.*	.*	.*
[root@h102 ~]# rabbitmqctl list_permissions -p mq_test
Listing permissions in vhost "mq_test" ...
mq	.*	.*	.*
[root@h102 ~]# rabbitmqctl list_user_permissions mq
Listing permissions for user "mq" ...
mq_test	.*	.*	.*
[root@h102 ~]# rabbitmqctl clear_permissions  mq
Clearing permissions for user "mq" in vhost "/" ...
[root@h102 ~]# rabbitmqctl list_user_permissions mq
Listing permissions for user "mq" ...
mq_test	.*	.*	.*
[root@h102 ~]# 
{% endhighlight %}


---

###删除vhost

最彻底直接方便也是最危险的权限清除方式就是直接删掉vhost

{% highlight bash %}
[root@h102 ~]# rabbitmqctl list_vhosts
Listing vhosts ...
/
/abc
mq_test
[root@h102 ~]# rabbitmqctl delete_vhost mq_test
Deleting vhost "mq_test" ...
[root@h102 ~]# rabbitmqctl list_vhosts
Listing vhosts ...
/
/abc
[root@h102 ~]# rabbitmqctl list_user_permissions mq
Listing permissions for user "mq" ...
[root@h102 ~]# 
{% endhighlight %}

**`/`** 也是可以被删除的

{% highlight bash %}
[root@h101 ~]# rabbitmqctl list_vhosts
Listing vhosts ...
/
[root@h101 ~]# rabbitmqctl delete_vhost /
Deleting vhost "/" ...
[root@h101 ~]# rabbitmqctl list_vhosts
Listing vhosts ...
[root@h101 ~]#
{% endhighlight %}


---

##连接RabbitMQ

###[python连接RabbitMQ][python_api]

生产脚本

{% highlight bash %}
[root@h102 python]# cat p.py 
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='mq_learning_q')

channel.basic_publish(exchange='',
                      routing_key='mq_learning_q',
                      body='Hello World!')
print " [x] Sent 'Hello World!'"
connection.close()
[root@h102 python]# 
{% endhighlight %}

消费脚本

{% highlight bash %}
[root@h102 python]# cat c.py 
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='mq_learning_q')

print ' [*] Waiting for messages. To exit press CTRL+C'

def callback(ch, method, properties, body):
    print " [x] Received %r" % (body,)

channel.basic_consume(callback,
                      queue='mq_learning_q',
                      no_ack=True)

channel.start_consuming()
[root@h102 python]# 
{% endhighlight %}

运行生产脚本

{% highlight bash %}
[root@h102 python]# python p.py
Traceback (most recent call last):
  File "p.py", line 2, in <module>
    import pika
ImportError: No module named pika
[root@h102 python]# 
{% endhighlight %}

####报错：缺少 **pika** 模块

解决办法，安装相应的包来解决依赖，建议使用pip，比较方便

{% highlight bash %}
[root@h102 python]# pip install pika
-bash: pip: command not found
[root@h102 python]# yum install python-pip
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: ftp.riken.jp
 * extras: mirrors.aliyun.com
 * updates: mirrors.163.com
Resolving Dependencies
--> Running transaction check
---> Package python-pip.noarch 0:7.1.0-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                           Arch                          Version                            Repository                   Size
======================================================================================================================================
Installing:
 python-pip                        noarch                        7.1.0-1.el6                        epel                        1.5 M

Transaction Summary
======================================================================================================================================
Install       1 Package(s)

Total download size: 1.5 M
Installed size: 6.6 M
Is this ok [y/N]: y
Downloading Packages:
python-pip-7.1.0-1.el6.noarch.rpm                                                                              | 1.5 MB     00:31     
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
  Installing : python-pip-7.1.0-1.el6.noarch                                                                                      1/1 
  Verifying  : python-pip-7.1.0-1.el6.noarch                                                                                      1/1 

Installed:
  python-pip.noarch 0:7.1.0-1.el6                                                                                                     

Complete!
[root@h102 python]# pip install pika
/usr/lib/python2.6/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
You are using pip version 7.1.0, however version 7.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Collecting pika
/usr/lib/python2.6/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
  Downloading pika-0.10.0-py2.py3-none-any.whl (92kB)
    100% |████████████████████████████████| 94kB 252kB/s 
Installing collected packages: pika
Successfully installed pika-0.10.0
[root@h102 python]# echo $?
0
[root@h102 python]# 
{% endhighlight %}

再次测试发送

{% highlight bash %}
[root@h102 python]# python p.py 
 [x] Sent 'Hello World!'
[root@h102 python]# echo $?
0
[root@h102 python]#
{% endhighlight %}

查看队列

{% highlight bash %}
[root@h102 python]# rabbitmqctl list_queues
Listing queues ...
mq_learning_q	1
[root@h102 python]# 
{% endhighlight %}

消费队列里的内容(这个进程消费完队列里的内容后，会挂起，等待接收队列里新的内容)

{% highlight bash %}
[root@h102 python]# python c.py 
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'Hello World!'

{% endhighlight %}


> **Tip:** 尝试多发几次，可以在消费端不断看到新的内容

{% highlight bash %}
[root@h102 python]# python p.py 
 [x] Sent 'Hello World!'
[root@h102 python]# python p.py 
 [x] Sent 'Hello World!'
[root@h102 python]# python p.py 
 [x] Sent 'Hello World!'
[root@h102 python]# python p.py 
 [x] Sent 'Hello World!'
[root@h102 python]# python p.py 
 [x] Sent 'Hello World!'
[root@h102 python]# 
----------
[root@h102 python]# python c.py 
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'Hello World!'
 [x] Received 'Hello World!'
 [x] Received 'Hello World!'
 [x] Received 'Hello World!'
 [x] Received 'Hello World!'
 [x] Received 'Hello World!'
{% endhighlight %}



---

###[ruby连接RabbitMQ][ruby_api]


生产脚本

{% highlight bash %}
[root@h102 ruby]# cat p.rb 
#!/usr/bin/env ruby
## encoding: utf-8

require "bunny"
conn = Bunny.new
conn.start
conn = Bunny.new(:hostname => "localhost")
conn.start
ch   = conn.create_channel
q    = ch.queue("ruby_test_q")
ch.default_exchange.publish("I am a handsome guy!", :routing_key => q.name)
puts " [x] Sent 'Done!'"
conn.close
[root@h102 ruby]# 
{% endhighlight %}

> **Tip:** 要连接远程的服务器只用修改下面的代码就可以了,相关的配置可以参考 **[bunny的API文档][bunny_api]**

{% highlight bash %}
conn = Bunny.new(:host => "192.168.1.20",:user => "test", :password => "test")
{% endhighlight %}

消费脚本

{% highlight bash %}
[root@h102 ruby]# cat c.rb 
#!/usr/bin/env ruby
## encoding: utf-8

require "bunny"

conn = Bunny.new
conn.start

ch   = conn.create_channel
q    = ch.queue("ruby_test_q")
puts " [*] Waiting for messages in #{q.name}. To exit press CTRL+C"
q.subscribe(:block => true) do |delivery_info, properties, body|
	puts " [x] Received #{body}"
	#cancel the consumer to exit
	#delivery_info.consumer.cancel
	end
[root@h102 ruby]# 
{% endhighlight %}

运行生产脚本


{% highlight bash %}
[root@h102 ruby]# ruby p.rb 
/usr/local/rvm/rubies/ruby-2.2.1/lib/ruby/site_ruby/2.2.0/rubygems/core_ext/kernel_require.rb:54:in `require': cannot load such file -- bunny (LoadError)
	from /usr/local/rvm/rubies/ruby-2.2.1/lib/ruby/site_ruby/2.2.0/rubygems/core_ext/kernel_require.rb:54:in `require'
	from p.rb:4:in `<main>'
[root@h102 ruby]#
{% endhighlight %}

####报错：缺少 **bunny** 模块

解决办法，安装相应的包来解决依赖，建议使用gem，比较方便

{% highlight bash %}
[root@h102 ruby]# gem sources -l 
*** CURRENT SOURCES ***

https://ruby.taobao.org/
[root@h102 ruby]# time gem install bunny
Fetching: amq-protocol-2.0.0.gem (100%)
Successfully installed amq-protocol-2.0.0
Fetching: bunny-2.2.1.gem (100%)
Successfully installed bunny-2.2.1
Parsing documentation for amq-protocol-2.0.0
Installing ri documentation for amq-protocol-2.0.0
Parsing documentation for bunny-2.2.1
Installing ri documentation for bunny-2.2.1
Done installing documentation for amq-protocol, bunny after 5 seconds
2 gems installed

real	0m10.577s
user	0m5.172s
sys	0m0.350s
[root@h102 ruby]# echo $?
0
[root@h102 ruby]# 
{% endhighlight %}


再次尝试发送

{% highlight bash %}
[root@h102 ruby]# ruby p.rb 
 [x] Sent 'Done!'
[root@h102 ruby]#
{% endhighlight %}

查看队列

{% highlight bash %}
[root@h102 ruby]# rabbitmqctl list_queues
Listing queues ...
mq_learning_q	0
ruby_test_q	1
[root@h102 ruby]# 
{% endhighlight %}


消费队列里的内容(这个进程消费完队列里的内容后，会挂起，等待接收队列里新的内容)

{% highlight bash %}
[root@h102 ruby]# ruby c.rb 
 [*] Waiting for messages in ruby_test_q. To exit press CTRL+C
 [x] Received I am a handsome guy!


{% endhighlight %}


> **Tip:** 尝试多发几次，可以在消费端不断看到新的内容

{% highlight bash %}
[root@h102 ruby]# ruby p.rb 
 [x] Sent 'Done!'
[root@h102 ruby]# ruby p.rb 
 [x] Sent 'Done!'
[root@h102 ruby]# ruby p.rb 
 [x] Sent 'Done!'
[root@h102 ruby]# ruby p.rb 
 [x] Sent 'Done!'
[root@h102 ruby]# 
----------
[root@h102 ruby]# ruby c.rb 
 [*] Waiting for messages in ruby_test_q. To exit press CTRL+C
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!
 [x] Received I am a handsome guy!


{% endhighlight %}


---

##日志

rabbitmq的日志默认存放在 **/var/log/rabbitmq/** 中

{% highlight bash %}
[root@h102 ruby]# ll /var/log/rabbitmq/
total 64
-rw-r--r-- 1 rabbitmq rabbitmq 25009 Nov 18 20:59 rabbit@h102.log
-rw-r--r-- 1 rabbitmq rabbitmq 15882 Oct 23 17:20 rabbit@h102.log.1
-rw-r--r-- 1 rabbitmq rabbitmq  2064 Nov 18 17:11 rabbit@h102.log-20151028.gz
-rw-r--r-- 1 rabbitmq rabbitmq  1945 Nov 18 17:11 rabbit@h102.log-20151118
-rw-r--r-- 1 rabbitmq rabbitmq     0 Nov 18 17:11 rabbit@h102-sasl.log
-rw-r--r-- 1 rabbitmq rabbitmq     0 Oct 23 17:20 rabbit@h102-sasl.log.1
-rw-r--r-- 1 root     root         0 Oct 24 00:22 shutdown_err
-rw-r--r-- 1 root     root        42 Oct 24 00:22 shutdown_log
-rw-r--r-- 1 root     root         0 Nov 18 16:13 startup_err
-rw-r--r-- 1 root     root       340 Nov 18 16:13 startup_log
[root@h102 ruby]# 
{% endhighlight %}

查看日志 

{% highlight bash %}
[root@h102 ruby]# tail /var/log/rabbitmq/rabbit@h102.log
=WARNING REPORT==== 18-Nov-2015::20:58:38 ===
closing AMQP connection <0.3341.0> (127.0.0.1:41681 -> 127.0.0.1:5672):
connection_closed_abruptly

=WARNING REPORT==== 18-Nov-2015::20:58:40 ===
closing AMQP connection <0.3104.0> (127.0.0.1:41662 -> 127.0.0.1:5672):
connection_closed_abruptly

=INFO REPORT==== 18-Nov-2015::20:59:46 ===
accepting AMQP connection <0.3366.0> (127.0.0.1:41683 -> 127.0.0.1:5672)
[root@h102 ruby]# 
{% endhighlight %}

> **Tip:** 一般可以使用 **tail -f /var/log/rabbitmq/rabbit@h102.log** 的方式来实时跟踪当前的变化

---



##状态查看


灵活使用下面的list命令可以更好了解当前MQ状态，详细用法可以参考 [官方文档][rabbitmqctl]


{% highlight bash %}
[root@h102 ruby]# rabbitmqctl list_policies
Listing policies ...
[root@h102 ruby]# rabbitmqctl list_queues
Listing queues ...
mq_learning_q	0
ruby_test_q	0
[root@h102 ruby]# rabbitmqctl list_exchanges
Listing exchanges ...
	direct
amq.direct	direct
amq.fanout	fanout
amq.headers	headers
amq.match	headers
amq.rabbitmq.log	topic
amq.rabbitmq.trace	topic
amq.topic	topic
[root@h102 ruby]# rabbitmqctl list_bindings
Listing bindings ...
	exchange	mq_learning_q	queue	mq_learning_q	[]
	exchange	ruby_test_q	queue	ruby_test_q	[]
[root@h102 ruby]# rabbitmqctl list_connections
Listing connections ...
guest	127.0.0.1	41683	running
[root@h102 ruby]# rabbitmqctl list_channels
Listing channels ...
<rabbit@h102.1.3374.0>	guest	1	0
[root@h102 ruby]# rabbitmqctl list_consumers
Listing consumers ...
ruby_test_q	<rabbit@h102.1.3374.0>	bunny-1447851586000-713632469015	false	0	[]
[root@h102 ruby]# 
{% endhighlight %}


---

[rabbitmq]:http://www.rabbitmq.com/
[rabbitmq_doc]:http://www.rabbitmq.com/documentation.html
[python_api]:http://www.rabbitmq.com/tutorials/tutorial-one-python.html
[ruby_api]:http://www.rabbitmq.com/tutorials/tutorial-one-ruby.html
[rabbitmqctl]:http://www.rabbitmq.com/man/rabbitmqctl.1.man.html
[bunny_api]:http://rubybunny.info/articles/connecting.html




