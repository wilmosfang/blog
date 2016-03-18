---
layout: post
title:  Consul 集群
categories:  linux cluster consul
wc: 638  2047 22482 
excerpt:  consul 的发现机制，集群构建，节点查询，脱离集群，健康检查，检查定义，配置重载，状态查看，键值操作，增删查改，条件更新，触发器与集群构建过程中的注意事项 
comments: true
---



# 前言

**[Consul][consul]** 是一个服务发现和配置工具


它有如下功能和特性：

* 服务发现
* 健康检查
* 健值存储
* 分布式且多数据中心

**Consul** 的作用类似于 **Zookeeper** 或 **etcd** ，和 **etcd** 一样也是使用 **Go** 实现的，也是使用的 **Raft** 算法

Consul 的架构

![consul_arch.png](/images/consul/consul_arch.png)

Docker Swarm 中使用 **[Consul][consul]** 来进行服务发现，这里简单分享一下 **[Consul][consul]** 集群相关的基础操作，详细内容可以参考 **[官方文档][consul_doc]** 


> **Tip:** 当前的最新版本为 **Consul 0.6.4** 

---


# 概要

* TOC
{:toc}



---

## 发现机制

当一个Consul代理启动后，它并不知道其它节点的存在，它是一个孤立的 **单节点集群**，如果想感知到其它节点的存在，它必须加入到一个现存的集群，要加入到一个现存的集群，它只用加入集群中任意一个现存的成员，当加入一个现存的成员后，会通过成员间的通讯很快发现集群中的其它成员，一个Consul代理可以加入任意一个代理，而不仅仅是服务节点

---

## 构建集群

### 启动首个节点

{% highlight bash %}
[root@h104 ~]# consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul -node=a1 -bind=192.168.100.104 -config-dir /etc/consul.d 
==> WARNING: BootstrapExpect Mode is specified as 1; this is the same as Bootstrap mode.
==> WARNING: Bootstrap mode enabled! Do not enable unless necessary
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'a1'
        Datacenter: 'dc1'
            Server: true (bootstrap: true)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 192.168.100.104 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/03/18 21:22:31 [INFO] raft: Node at 192.168.100.104:8300 [Follower] entering Follower state
    2016/03/18 21:22:31 [INFO] serf: EventMemberJoin: a1 192.168.100.104
    2016/03/18 21:22:31 [INFO] serf: EventMemberJoin: a1.dc1 192.168.100.104
    2016/03/18 21:22:31 [INFO] consul: adding WAN server a1.dc1 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 21:22:31 [INFO] consul: adding LAN server a1 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 21:22:31 [ERR] agent: failed to sync remote state: No cluster leader
    2016/03/18 21:22:32 [WARN] raft: Heartbeat timeout reached, starting election
    2016/03/18 21:22:32 [INFO] raft: Node at 192.168.100.104:8300 [Candidate] entering Candidate state
    2016/03/18 21:22:32 [INFO] raft: Election won. Tally: 1
    2016/03/18 21:22:32 [INFO] raft: Node at 192.168.100.104:8300 [Leader] entering Leader state
    2016/03/18 21:22:32 [INFO] consul: cluster leadership acquired
    2016/03/18 21:22:32 [INFO] consul: New leader elected: a1
    2016/03/18 21:22:32 [INFO] raft: Disabling EnableSingleNode (bootstrap)
    2016/03/18 21:22:32 [INFO] consul: member 'a1' joined, marking health alive
    2016/03/18 21:22:34 [INFO] agent: Synced service 'consul'
    2016/03/18 21:22:34 [INFO] agent: Synced service 'web'
...
...
...
{% endhighlight %}


ARG     | Comment
-------- | ---
`-server` | 以服务模式运行
`-bootstrap-expect` | 指定期望加入的节点数
`-data-dir`     | 指定数据存放的位置
`-node`|指定节点名
`-bind`|指定绑定的IP
`-config-dir`|指定配置目录



### 启动第二个节点

{% highlight bash %}
[root@docker consul]# consul agent -data-dir /tmp/consul -node=a2 -bind=192.168.100.103 -config-dir /etc/consul.d
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'a2'
        Datacenter: 'dc1'
            Server: false (bootstrap: false)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 192.168.100.103 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/03/18 21:51:55 [INFO] serf: EventMemberJoin: a2 192.168.100.103
    2016/03/18 21:51:55 [ERR] agent: failed to sync remote state: No known Consul servers
    2016/03/18 21:52:17 [ERR] agent: failed to sync remote state: No known Consul servers
    2016/03/18 21:52:22 [INFO] agent.rpc: Accepted client: 127.0.0.1:42288
    2016/03/18 21:52:38 [ERR] agent: failed to sync remote state: No known Consul servers
...
...
...
{% endhighlight %}


此时已经分别在104和103上启动了两个代理a1和a2，a1准备用来作server ，a2用来作client，但它们彼此还互不认识，都是自己的单节点集群中的唯一节点，可以通过 **`consul members`** 来进行查看


{% highlight bash %}
[root@h104 consul]# consul members
Node  Address               Status  Type    Build  Protocol  DC
a1    192.168.100.104:8301  alive   server  0.6.4  2         dc1
[root@h104 consul]# 
----------
[root@docker ~]# consul members
Node  Address               Status  Type    Build  Protocol  DC
a2    192.168.100.103:8301  alive   client  0.6.4  2         dc1
[root@docker ~]# 
{% endhighlight %}

---

### 加入集群

使用a1来加入a2

{% highlight bash %}
[root@h104 consul]# consul join 192.168.100.103
Successfully joined cluster by contacting 1 nodes.
[root@h104 consul]# 
{% endhighlight %}

> **Note:** 要确保两个节点TCP或UDP的8301是开放的，最好是TCP和UDP都开放，因为节点间的通讯得依赖这个端口，否则无法加入

{% highlight bash %}
[root@h104 consul]# consul join 192.168.100.103
Error joining the cluster: dial tcp 192.168.100.103:8301: getsockopt: no route to host
[root@h104 consul]# 
{% endhighlight %}

> **Tip:** 防火墙端口打开方法：在Centos7中使用 **firewall-cmd** 来管理防火墙

{% highlight bash %}
[root@h104 consul]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 3306/tcp 80/tcp 40000/tcp 8080/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@h104 consul]# firewall-cmd --add-port=8301/tcp
success
[root@h104 consul]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 3306/tcp 80/tcp 40000/tcp 8080/tcp 8301/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@h104 consul]# consul join 192.168.100.103
Successfully joined cluster by contacting 1 nodes.
[root@h104 consul]# 
{% endhighlight %}

加入成功后server节点上就会产生如下日志

{% highlight bash %}
...
...
    2016/03/18 22:00:36 [INFO] agent.rpc: Accepted client: 127.0.0.1:44743
    2016/03/18 22:00:36 [INFO] agent: (LAN) joining: [192.168.100.103]
    2016/03/18 22:00:36 [INFO] agent: (LAN) joined: 0 Err: dial tcp 192.168.100.103:8301: getsockopt: no route to host
    2016/03/18 22:04:06 [INFO] agent.rpc: Accepted client: 127.0.0.1:44778
    2016/03/18 22:04:06 [INFO] agent: (LAN) joining: [192.168.100.103]
    2016/03/18 22:04:06 [INFO] serf: EventMemberJoin: a2 192.168.100.103
    2016/03/18 22:04:06 [INFO] agent: (LAN) joined: 1 Err: <nil>
    2016/03/18 22:04:06 [INFO] consul: member 'a2' joined, marking health alive
    2016/03/18 22:07:12 [INFO] agent.rpc: Accepted client: 127.0.0.1:44813
...
...
{% endhighlight %}


> **Note:** 要确保server节点TCP的8300是开放的，最好是server 和client都开放(没准以后client也会更换角色呢)，因为client向server的RPC得依赖这个端口，不打开无法同步



{% highlight bash %}
...
...
    2016/03/18 22:04:06 [INFO] serf: EventMemberJoin: a1 192.168.100.104
    2016/03/18 22:04:06 [INFO] consul: adding server a1 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 22:04:06 [INFO] consul: New leader elected: a1
    2016/03/18 22:04:07 [ERR] agent: coordinate update error: rpc error: failed to get conn: dial tcp 192.168.100.104:8300: getsockopt: no route to host
    2016/03/18 22:04:09 [ERR] agent: failed to sync remote state: rpc error: failed to get conn: dial tcp 192.168.100.104:8300: getsockopt: no route to host
...
...
...
{% endhighlight %}

打开方式一样

{% highlight bash %}
[root@h104 consul]# firewall-cmd --add-port=8300/tcp
success
[root@h104 consul]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 80/tcp 8080/tcp 8301/tcp 3306/tcp 8300/tcp 40000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@h104 consul]#
{% endhighlight %}

打开后client才能正常同步

{% highlight bash %}
...
...
    2016/03/18 22:06:03 [INFO] agent: Synced node info
...
...
{% endhighlight %}


此时再在两个节点上查看成员状态，彼此都能互识了

{% highlight bash %}
[root@h104 consul]# consul members
Node  Address               Status  Type    Build  Protocol  DC
a1    192.168.100.104:8301  alive   server  0.6.4  2         dc1
a2    192.168.100.103:8301  alive   client  0.6.4  2         dc1
[root@h104 consul]# 
----------
[root@docker ~]# consul members
Node  Address               Status  Type    Build  Protocol  DC
a1    192.168.100.104:8301  alive   server  0.6.4  2         dc1
a2    192.168.100.103:8301  alive   client  0.6.4  2         dc1
[root@docker ~]# 
{% endhighlight %}


> **Tip:** 如果有多个成员，也只用加入一个节点，其它节点会在这个节点加入集群后通过成员间的通讯相互发现


---

### 查询节点

可以通过 DNS API 或 HTTP API 来查询节点

如果使用DNS API，查询结构为 **`NAME.node.consul`** 和 **`NAME.node.DATACENTER.consul`** 

{% highlight bash %}
[root@h104 consul]# dig @127.0.0.1 -p 8600 a2.node.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 a2.node.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57955
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;a2.node.consul.			IN	A

;; ANSWER SECTION:
a2.node.consul.		0	IN	A	192.168.100.103

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 23:03:36 CST 2016
;; MSG SIZE  rcvd: 62

[root@h104 consul]# dig @127.0.0.1 -p 8600 a2.node.dc1.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 a2.node.dc1.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45949
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;a2.node.dc1.consul.		IN	A

;; ANSWER SECTION:
a2.node.dc1.consul.	0	IN	A	192.168.100.103

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 23:03:40 CST 2016
;; MSG SIZE  rcvd: 70

[root@h104 consul]# dig @127.0.0.1 -p 8600 a1.node.dc1.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 a1.node.dc1.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43289
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;a1.node.dc1.consul.		IN	A

;; ANSWER SECTION:
a1.node.dc1.consul.	0	IN	A	192.168.100.104

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 23:03:47 CST 2016
;; MSG SIZE  rcvd: 70

[root@h104 consul]# 
{% endhighlight %}


---

### 脱离集群

可以使用 `Ctrl-C` 来平滑地退出，也可以强行Kill退出，区别是主动告知其它节点自己的离开，和被其它节点标记为失效，被发现离开

---

## 健康检查


健康检查对于避免将请求发送给运行不正常的服务是一个相当关键的机制

和服务一样，有两种方式来定义健康检查

* 通过配置文件
* 使用 HTTP API

### 定义检查

这里使用配置文件的方式来定义健康检查

{% highlight bash %}
[root@docker ~]# echo '{"check": {"name": "ping","script": "ping -c1 soft.dog >/dev/null", "interval": "30s"}}'  > /etc/consul.d/ping.json
[root@docker ~]# echo '{"service": {"name": "web", "tags": ["rails"], "port": 80,"check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}}}' > /etc/consul.d/web.json 
[root@docker ~]# cat /etc/consul.d/ping.json 
{"check": {"name": "ping","script": "ping -c1 soft.dog >/dev/null", "interval": "30s"}}
[root@docker ~]# cat /etc/consul.d/web.json 
{"service": {"name": "web", "tags": ["rails"], "port": 80,"check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}}}
[root@docker ~]# 
{% endhighlight %}

### 重载配置

通过给进程发送 **SIGHUP** 的信号来使配置重载

{% highlight bash %}
[root@docker ~]# ps faux | grep consul
root     22094  1.2  0.3  25084 13756 pts/0    Sl+  21:51   1:07  |       \_ consul agent -data-dir /tmp/consul -node=a2 -bind=192.168.100.103 -config-dir /etc/consul.d
root     25063  0.0  0.0 112644   960 pts/1    S+   23:20   0:00          \_ grep --color=auto consul
[root@docker ~]# kill -s SIGHUP 22094
[root@docker ~]#
{% endhighlight %}

这时可以观察到日志输出

{% highlight bash %}
...
...
==> Caught signal: hangup
==> Reloading configuration...
    2016/03/18 23:21:07 [INFO] agent: Synced service 'web'
    2016/03/18 23:21:07 [INFO] agent: Synced check 'ping'
    2016/03/18 23:21:08 [WARN] agent: Check 'service:web' is now critical
    2016/03/18 23:21:18 [WARN] agent: Check 'service:web' is now critical
    2016/03/18 23:21:28 [WARN] agent: Check 'service:web' is now critical
    2016/03/18 23:21:32 [INFO] agent: Synced check 'ping'
    2016/03/18 23:21:38 [WARN] agent: Check 'service:web' is now critical
...
...
...
{% endhighlight %}

重新加载配置后，两个检查脚本都成功载入了

ping 脚本检查正常，因为我的博客地址是可达的，同时由于我们并没有真正在本地启web服务，80端口不存在，也不提供内容，所以检查结果是状态不正常

---


### 查看状态

可以使用HTTP API来检查配置

{% highlight bash %}
[root@h104 consul]# curl http://localhost:8500/v1/health/state/critical
[{"Node":"a2","CheckID":"service:web","Name":"Service 'web' check","Status":"critical","Notes":"","Output":"","ServiceID":"web","ServiceName":"web","CreateIndex":593,"ModifyIndex":593}][root@h104 consul]# 
[root@h104 consul]#
----------
[root@docker ~]# curl http://localhost:8500/v1/health/state/critical
[{"Node":"a2","CheckID":"service:web","Name":"Service 'web' check","Status":"critical","Notes":"","Output":"","ServiceID":"web","ServiceName":"web","CreateIndex":593,"ModifyIndex":593}][root@docker ~]# 
[root@docker ~]#
{% endhighlight %}

可以在任意一个节点上进行检查

和服务一样，健康检查也可以使用 HTTP API 来动态地进行添加，删除和修改

---

## 键值存储

Consul 提供了一个简单的键值存储机制，可以使用这个特性来存储动态配置，服务协调，主节点选举和其它一些功能


{% highlight bash %}
[root@h104 consul]# curl -v http://localhost:8500/v1/kv/?recurse
* About to connect() to localhost port 8500 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8500 (#0)
> GET /v1/kv/?recurse HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8500
> Accept: */*
> 
< HTTP/1.1 404 Not Found
< X-Consul-Index: 1
< X-Consul-Knownleader: true
< X-Consul-Lastcontact: 0
< Date: Fri, 18 Mar 2016 16:14:49 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
< 
* Connection #0 to host localhost left intact
[root@h104 consul]# 
{% endhighlight %}

因为没有任何值，所以反馈结果为 404

### 存入值

我们先存入一些值，然后再取回

创建用 **PUT** 方法

{% highlight bash %}
[root@h104 consul]# curl -X PUT -d 'soft.dog' http://localhost:8500/v1/kv/web/key1
true[root@h104 consul]# curl -X PUT -d 'soft.dog' http://localhost:8500/v1/kv/web/key2?flags=42
true[root@h104 consul]# curl -X PUT -d 'soft.dog' http://localhost:8500/v1/kv/web/sub/key3
true[root@h104 consul]# curl http://localhost:8500/v1/kv/?recurse
[{"LockIndex":0,"Key":"web/key1","Flags":0,"Value":"c29mdC5kb2c=","CreateIndex":909,"ModifyIndex":909},{"LockIndex":0,"Key":"web/key2","Flags":42,"Value":"c29mdC5kb2c=","CreateIndex":912,"ModifyIndex":912},{"LockIndex":0,"Key":"web/sub/key3","Flags":0,"Value":"c29mdC5kb2c=","CreateIndex":917,"ModifyIndex":917}][root@h104 consul]# 
[root@h104 consul]# 
[root@h104 consul]# 
{% endhighlight %}


### 查询值

查询用 **GET** 方法

**`?recurse`** 参数是递归返回所有KV的意思， 如果要单独返回指定值可以使用指定key的方式

{% highlight bash %}
[root@h104 consul]# curl  http://localhost:8500/v1/kv/web/key2
[{"LockIndex":0,"Key":"web/key2","Flags":42,"Value":"c29mdC5kb2c=","CreateIndex":912,"ModifyIndex":912}][root@h104 consul]# 
[root@h104 consul]#  
{% endhighlight %}

---

### 删除值 

删除用 **DELETE** 方法

{% highlight bash %}
[root@h104 consul]# curl -X DELETE  http://localhost:8500/v1/kv/web/sub?recurse
true[root@h104 consul]# curl http://localhost:8500/v1/kv/web?recurse
[{"LockIndex":0,"Key":"web/key1","Flags":0,"Value":"c29mdC5kb2c=","CreateIndex":909,"ModifyIndex":909},{"LockIndex":0,"Key":"web/key2","Flags":42,"Value":"c29mdC5kb2c=","CreateIndex":912,"ModifyIndex":912}][root@h104 consul]# 
[root@h104 consul]# 
{% endhighlight %}

---

### 更新值

更新和存值一样使用 **PUT** 方法，只是提供一个与原值不同的内容就可以了

{% highlight bash %}
[root@h104 consul]# curl http://localhost:8500/v1/kv/web/key1
[{"LockIndex":0,"Key":"web/key1","Flags":0,"Value":"c29mdC5kb2c=","CreateIndex":909,"ModifyIndex":909}][root@h104 consul]# 
[root@h104 consul]# 
[root@h104 consul]# curl -X PUT -d 'great' http://localhost:8500/v1/kv/web/key1
true[root@h104 consul]# 
[root@h104 consul]# curl http://localhost:8500/v1/kv/web/key1
[{"LockIndex":0,"Key":"web/key1","Flags":0,"Value":"Z3JlYXQ=","CreateIndex":909,"ModifyIndex":1000}][root@h104 consul]# 
[root@h104 consul]# 
{% endhighlight %}

**ModifyIndex** 会增加

###  条件更新

也就是检查更新， Check-And-Set ， 当 **cas** 指定的值与 **ModifyIndex** 相等时，才能成功更新，否则更新失败

{% highlight bash %}
[root@h104 consul]# curl  http://localhost:8500/v1/kv/web/key1
[{"LockIndex":0,"Key":"web/key1","Flags":0,"Value":"Z3JlYXQ=","CreateIndex":909,"ModifyIndex":1061}][root@h104 consul]# 
[root@h104 consul]# 
[root@h104 consul]# curl -X PUT -d 'great' http://localhost:8500/v1/kv/web/key1?cas=1061
true[root@h104 consul]# 
[root@h104 consul]# curl -X PUT -d 'great' http://localhost:8500/v1/kv/web/key1?cas=1061
false[root@h104 consul]# 
[root@h104 consul]# curl  http://localhost:8500/v1/kv/web/key1
[{"LockIndex":0,"Key":"web/key1","Flags":0,"Value":"Z3JlYXQ=","CreateIndex":909,"ModifyIndex":1076}][root@h104 consul]# 
[root@h104 consul]#
{% endhighlight %}

第一次更新成功是因为 **cas** 指定的值 1061 与 **ModifyIndex** 相等，第二次失败是因为，**cas** 指定的值 1061与**ModifyIndex** 的 1076 不相等

---

### 监听

{% highlight bash %}
[root@h104 consul]# time curl "http://localhost:8500/v1/kv/web/key2?index=101&wait=5s"
[{"LockIndex":0,"Key":"web/key2","Flags":42,"Value":"c29mdC5kb2c=","CreateIndex":912,"ModifyIndex":912}]
real	0m0.030s
user	0m0.006s
sys	0m0.021s
[root@h104 consul]# time curl "http://localhost:8500/v1/kv/web/key2?index=10001&wait=5s"
[{"LockIndex":0,"Key":"web/key2","Flags":42,"Value":"c29mdC5kb2c=","CreateIndex":912,"ModifyIndex":912}]
real	0m5.138s
user	0m0.005s
sys	0m0.015s
[root@h104 consul]#
{% endhighlight %}

当数据的 **ModifyIndex** 超过指定值，立刻返回，如果5s之内还没有满足条件，就直接返回原值，如果不加wait，则为一直等下去

这个特性可以用作触发器


---

## 监控

Consul 提供 WEB UI 来对自身状态进行 GUI 展示

有两种方式：

* 使用 Atlas by HashiCorp 的方式来进行托管
* 自建 open-source UI，自己维护

这个还没完全整明白，有机会再研究补全


---



# 命令汇总

* **`consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul -node=a1 -bind=192.168.100.104 -config-dir /etc/consul.d`**
* **`consul agent -data-dir /tmp/consul -node=a2 -bind=192.168.100.103 -config-dir /etc/consul.d`**
* **`consul members`**
* **`firewall-cmd --add-port=8301/tcp`**
* **`consul join 192.168.100.103`**
* **`firewall-cmd --add-port=8300/tcp`**
* **`firewall-cmd --list-all`**
* **`dig @127.0.0.1 -p 8600 a2.node.consul`**
* **`dig @127.0.0.1 -p 8600 a1.node.dc1.consul`**
* **`echo '{"check": {"name": "ping","script": "ping -c1 soft.dog >/dev/null", "interval": "30s"}}'  > /etc/consul.d/ping.json`**
* **`echo '{"service": {"name": "web", "tags": ["rails"], "port": 80,"check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}}}' > /etc/consul.d/web.json`**
* **`cat /etc/consul.d/ping.json`**
* **`cat /etc/consul.d/web.json`**
* **`kill -s SIGHUP 22094`**
* **`curl http://localhost:8500/v1/health/state/critical`**
* **`curl -v http://localhost:8500/v1/kv/?recurse`**
* **`curl -X PUT -d 'soft.dog' http://localhost:8500/v1/kv/web/key1`**
* **`curl  http://localhost:8500/v1/kv/web/key2`**
* **`curl -X DELETE  http://localhost:8500/v1/kv/web/sub?recurse`**
* **`curl http://localhost:8500/v1/kv/web/key1`**
* **`curl -X PUT -d 'great' http://localhost:8500/v1/kv/web/key1`**
* **`curl -X PUT -d 'great' http://localhost:8500/v1/kv/web/key1?cas=1061`**
* **`time curl "http://localhost:8500/v1/kv/web/key2?index=101&wait=5s"`**
* **`time curl "http://localhost:8500/v1/kv/web/key2?index=10001&wait=5s"`**


---


[consul]:https://www.consul.io/
[consul_doc]:https://www.consul.io/docs/index.html
