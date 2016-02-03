---
layout: post
title: Redis 迁移(操作流程建议)
categories: linux redis nosql keepalived
wc: 163 230 3602
excerpt: follow me
comments: true
---

---

# 前言

生产环境下由于服务器系统负载不均，服务器调动等各方面的考虑会遇到有必要对 **[Redis][redis]** 进行迁移的情况


这里分享一下 **Redis迁移** 的过程

> **Tip:** 当前版本 **Redis 3.0.4**

---

# 概要

* TOC
{:toc}


---

## 准备

由于redis的内存特性，迁移过程中避免数据丢失，最好准备两台服务器作为备用

master a (running pd redis)

backup b (backup of a)

backup c (backup of b)


---

## 启动一个新redis实例b

> **Tip:**  假定当前master为实例a

使用相同版本

拷贝一份master配置

创建一个用户redis (不用赋密码)

根据配置文件创建相关目录和赋予redis用户以相应权限

使用redis用户身份执行如下代码

{% highlight bash %}
redis-server  /etc/redis/redis6379.conf
{% endhighlight %}

> **Note:**  使用相同的方法在c上也建立一个redis实例，准备作为b的备份实例

---


## 安装keepalived

在 a  b 上安装keepalived

使用相同版本

设定一个未使用的IP作为VIP

{% highlight bash %}
nmap -R -vv -T4 -p 1  192.168.1.20/24   | grep 'host down'
{% endhighlight %}

除了下面两条，其它配置都一样

* router_id：不同节点不一样
* priority：master要高于slave

---

## 修改iptables

加入如下配置到b和c 的filter表，然后reload

{% highlight bash %}
-A INPUT -d 224.0.0.18 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6379 -j ACCEPT
{% endhighlight %}

> **Note:**  加载完iptables配置 ，最好在其它机器上进行一个访问测试，避免出现网络问题


---

## b同步到a

选择业务低峰点操作

{% highlight bash %}
SLAVEOF a 6379
{% endhighlight %}

可以使用以下命令观察同步状态 **master_sync_in_progress**

{% highlight bash %}
info replication 
{% endhighlight %}

> **Note:**  b的同步完成之后，使用相同的方法让c同步b



---

## 关闭slave只读

在b上关闭slave只读

> **Tip:**  这个设置本来是为了安全，避免对slave的写操作而导致的数据不一致，但为了平滑切换，需要牺牲一点这方面的安全考虑，如果不打开此设置，会出现一些写入报错

{% highlight bash %}
CONFIG SET slave-read-only no 
{% endhighlight %}

---

## 切换keepalived，发布到新VIP


1.切换keepalived，把ip切换到b (可以通过调整优先级，然后reload)

2.然后发布应用到新的VIP(修改配置，将新的VIP作为对redis的访问IP)

> **Note:**  如果先进行的步骤 **2** ，然后进行的步骤 **1** ，这样有一段时间由于arp缓存的缘故，同时会对a和b进行读写操作，如果应用层面对于数据的读写划分得比较好，短时间内，不会造成太大的问题(a b数据不一致)，但时间久了就不行，最彻底的解决办法，还是对新VIP再重新发布一下

---

## 断开同步

观察一段时间，注意以下几个方面

* 1.a上的redis网络连结数 (连接数降为0)
* 2.b上的redis网络连接数 (连接数逐渐稳定)
* 3.日志与报错 (没有特别值得注意的报错信息)

在b上使用如下命令断开与a的同步

{% highlight bash %}
slaveof no one
{% endhighlight %}

此时切换完成，b已经成为了c的master

> **Tip:** 可以考虑重新将a 作为 b的slave，而原来a的slave也可以考虑回收(数据陈旧已经不能用)，注意keepalived实例的优先级，避免飘回

---


[redis]:http://redis.io/
