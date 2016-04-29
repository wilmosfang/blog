---
layout: post
title:  Consul 基础
categories:  linux consul
wc: 639  2349 23520 
excerpt:  consul 的下载安装运行，成员查看，HTTP API，DNS API ，停止consul，服务定义，服务注册，查看服务和注意事项 
comments: true
---



# 前言

**[Consul][consul]** 是一个服务发现和配置工具


它有如下功能和特性：

* 服务发现
* 健康检查
* 健值存储
* 多数据中心

**Consul** 的作用类似于 **Zookeeper** 或 **etcd** ，和 **etcd** 一样也是使用 **Go** 实现的，也是使用的 **Raft** 算法

Consul 的架构

![consul_arch.png](/images/consul/consul_arch.png)



Docker Swarm 中使用 **[Consul][consul]** 来进行服务发现，这里简单分享一下 **[Consul][consul]** 操作的相关基础，详细内容可以参考 **[官方文档][consul_doc]** 


> **Tip:** 当前的最新版本为 **Consul 0.6.4** 

---


# 概要

* TOC
{:toc}


---

## 下载

**Consul** 的 **[下载地址][consul_dl]**


~~~
[root@h104 consul]# wget https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_linux_amd64.zip
--2016-03-18 15:25:40--  https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_linux_amd64.zip
Resolving releases.hashicorp.com (releases.hashicorp.com)... 103.245.224.69
Connecting to releases.hashicorp.com (releases.hashicorp.com)|103.245.224.69|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6206903 (5.9M) [application/zip]
Saving to: ‘consul_0.6.4_linux_amd64.zip’

100%[==========================================================================>] 6,206,903    200KB/s   in 24s    

2016-03-18 15:26:05 (258 KB/s) - ‘consul_0.6.4_linux_amd64.zip’ saved [6206903/6206903]

[root@h104 consul]# ll 
total 6064
-rw-r--r-- 1 root root 6206903 Mar 17 00:55 consul_0.6.4_linux_amd64.zip
[root@h104 consul]# sha256sum consul_0.6.4_linux_amd64.zip 
abdf0e1856292468e2c9971420d73b805e93888e006c76324ae39416edcf0627  consul_0.6.4_linux_amd64.zip
[root@h104 consul]# 
~~~

> **Tip:** 建议下载完成后进行一下 **校验** 以确保包的完整性，**Consul 0.6.4** 各平台版本可以参考  **[SHA256 checksums for Consul 0.6.4][checksum]**

---

## 解压安装

解压后就是一个可以直接使用的bin文件，拷贝到合适目录就行了

~~~
[root@h104 consul]# ls
consul_0.6.4_linux_amd64.zip
[root@h104 consul]# unzip consul_0.6.4_linux_amd64.zip 
Archive:  consul_0.6.4_linux_amd64.zip
  inflating: consul                  
[root@h104 consul]# ll 
total 28528
-rwxr-xr-x 1 root root 23002736 Mar 17 00:52 consul
-rw-r--r-- 1 root root  6206903 Mar 17 00:55 consul_0.6.4_linux_amd64.zip
[root@h104 consul]# file consul
consul: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
[root@h104 consul]# ./consul --help 
usage: consul [--version] [--help] <command> [<args>]

Available commands are:
    agent          Runs a Consul agent
    configtest     Validate config file
    event          Fire a new event
    exec           Executes a command on Consul nodes
    force-leave    Forces a member of the cluster to enter the "left" state
    info           Provides debugging information for operators
    join           Tell Consul agent to join cluster
    keygen         Generates a new encryption key
    keyring        Manages gossip layer encryption keys
    leave          Gracefully leaves the Consul cluster and shuts down
    lock           Execute a command holding a lock
    maint          Controls node or service maintenance mode
    members        Lists the members of a Consul cluster
    monitor        Stream logs from a Consul agent
    reload         Triggers the agent to reload configuration files
    rtt            Estimates network round trip time between nodes
    version        Prints the Consul version
    watch          Watch for changes in Consul

[root@h104 consul]# ./consul version
Consul v0.6.4
Consul Protocol: 3 (Understands back to: 1)
[root@h104 consul]# cp consul /usr/local/bin/
[root@h104 consul]# consul version
Consul v0.6.4
Consul Protocol: 3 (Understands back to: 1)
[root@h104 consul]# 

~~~

---

## 运行Consul代理

**Consul** 是典型的C/S架构，可以运行在 **服务模式** 或 **客户模式**

每一个数据中心必须有至少一个服务节点，3到5个服务节点最好，非常不建议只运行一个服务节点，因为在节点失效的情况下数据有极大的丢失风险

其它的所有节点都运行在客户端模式下，一个客户节点是非常轻量的注册服务进程，用来处理健康检查，转发请求到服务节点等事务，代理必须在集群中的每个节点上运行


以开发模式运行 **Consul**，这个模式可以快速启动一个单节点集群，但是由于并不保存状态，所以不要在生产环境中使用

~~~
[root@h104 consul]# consul agent -dev -bind=0.0.0.0
==> Starting Consul agent...
==> Error starting agent: Failed to get advertise address: Multiple private IPs found. Please configure one.
[root@h104 consul]# consul agent -dev -bind=192.168.100.104
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'h104'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 192.168.100.104 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/03/18 16:18:56 [INFO] serf: EventMemberJoin: h104 192.168.100.104
    2016/03/18 16:18:56 [INFO] serf: EventMemberJoin: h104.dc1 192.168.100.104
    2016/03/18 16:18:56 [INFO] raft: Node at 192.168.100.104:8300 [Follower] entering Follower state
    2016/03/18 16:18:56 [INFO] consul: adding LAN server h104 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 16:18:56 [INFO] consul: adding WAN server h104.dc1 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 16:18:56 [ERR] agent: failed to sync remote state: No cluster leader
    2016/03/18 16:18:57 [WARN] raft: Heartbeat timeout reached, starting election
    2016/03/18 16:18:57 [INFO] raft: Node at 192.168.100.104:8300 [Candidate] entering Candidate state
    2016/03/18 16:18:57 [DEBUG] raft: Votes needed: 1
    2016/03/18 16:18:57 [DEBUG] raft: Vote granted from 192.168.100.104:8300. Tally: 1
    2016/03/18 16:18:57 [INFO] raft: Election won. Tally: 1
    2016/03/18 16:18:57 [INFO] raft: Node at 192.168.100.104:8300 [Leader] entering Leader state
    2016/03/18 16:18:57 [INFO] raft: Disabling EnableSingleNode (bootstrap)
    2016/03/18 16:18:57 [INFO] consul: cluster leadership acquired
    2016/03/18 16:18:57 [DEBUG] raft: Node 192.168.100.104:8300 updated peer set (2): [192.168.100.104:8300]
    2016/03/18 16:18:57 [DEBUG] consul: reset tombstone GC to index 2
    2016/03/18 16:18:57 [INFO] consul: member 'h104' joined, marking health alive
    2016/03/18 16:18:57 [INFO] consul: New leader elected: h104
    2016/03/18 16:18:59 [INFO] agent: Synced service 'consul'
    2016/03/18 16:20:48 [DEBUG] agent: Service 'consul' in sync
    2016/03/18 16:21:53 [DEBUG] agent: Service 'consul' in sync
    2016/03/18 16:23:02 [INFO] agent.rpc: Accepted client: 127.0.0.1:41645
    2016/03/18 16:23:32 [DEBUG] agent: Service 'consul' in sync
...
...
...
~~~

> **Note:**  在有多个IP的环境下，必须指定IP，否则会失败

~~~
[root@h104 consul]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:5e:57:a3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.125/24 brd 192.168.1.255 scope global dynamic eno16777736
       valid_lft 25200sec preferred_lft 25200sec
    inet6 fe80::20c:29ff:fe5e:57a3/64 scope link 
       valid_lft forever preferred_lft forever
3: eno33554960: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:5e:57:ad brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.104/24 brd 192.168.100.255 scope global eno33554960
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe5e:57ad/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:52:c8:a6:60 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
[root@h104 consul]# consul agent -dev
==> Starting Consul agent...
==> Error starting agent: Failed to get advertise address: Multiple private IPs found. Please configure one.
[root@h104 consul]# 
~~~

启动 **consul** 后，系统会监听在一些端口

~~~
[root@h104 ~]# netstat  -tunpea | grep consul
tcp        0      0 192.168.100.104:8300    0.0.0.0:*               LISTEN      0          47240      11552/consul        
tcp        0      0 192.168.100.104:8301    0.0.0.0:*               LISTEN      0          47241      11552/consul        
tcp        0      0 192.168.100.104:8302    0.0.0.0:*               LISTEN      0          47243      11552/consul        
tcp        0      0 127.0.0.1:8400          0.0.0.0:*               LISTEN      0          47245      11552/consul        
tcp        0      0 127.0.0.1:8500          0.0.0.0:*               LISTEN      0          47246      11552/consul        
tcp        0      0 127.0.0.1:8600          0.0.0.0:*               LISTEN      0          47253      11552/consul        
udp        0      0 192.168.100.104:8301    0.0.0.0:*                           0          47242      11552/consul        
udp        0      0 192.168.100.104:8302    0.0.0.0:*                           0          47244      11552/consul        
udp        0      0 127.0.0.1:8600          0.0.0.0:*                           0          47252      11552/consul        
[root@h104 ~]# 
~~~

---

## 查看成员

### 使用member命令

~~~
[root@h104 ~]# consul members --help 
Usage: consul members [options]

  Outputs the members of a running Consul agent.

Options:

  -detailed                 Provides detailed information about nodes

  -rpc-addr=127.0.0.1:8400  RPC address of the Consul agent.

  -status=<regexp>          If provided, output is filtered to only nodes matching
                            the regular expression for status

  -wan                      If the agent is in server mode, this can be used to return
                            the other peers in the WAN pool
[root@h104 ~]# consul members -detailed 
Node  Address               Status  Tags
h104  192.168.100.104:8301  alive   build=0.6.4:26a0ef8c,dc=dc1,port=8300,role=consul,vsn=2,vsn_max=3,vsn_min=1
[root@h104 ~]# 
~~~

---

### 使用HTTP API

还可以使用 **HTTP API** (RESTful API) 的方式查看信息

~~~
[root@h104 ~]# curl localhost:8500/v1/catalog/nodes?pretty
[
    {
        "Node": "h104",
        "Address": "192.168.100.104",
        "TaggedAddresses": {
            "wan": "192.168.100.104"
        },
        "CreateIndex": 3,
        "ModifyIndex": 5
    }
][root@h104 ~]# 
[root@h104 ~]#
~~~

---

## 使用DNS API

启动服务后，本地的 **8600** 一直处于监听状态，可以接受DNS请求

~~~
[root@h104 ~]# netstat  -tunpea | grep consul | grep 8600
tcp        0      0 127.0.0.1:8600          0.0.0.0:*               LISTEN      0          52147      13641/consul        
udp        0      0 127.0.0.1:8600          0.0.0.0:*                           0          52979      13641/consul        
[root@h104 ~]# 
~~~

使用 dig 查看节点IP

~~~
[root@h104 ~]# dig @127.0.0.1 -p 8600  h104.node.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 h104.node.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43696
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;h104.node.consul.		IN	A

;; ANSWER SECTION:
h104.node.consul.	0	IN	A	192.168.100.104

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 17:22:33 CST 2016
;; MSG SIZE  rcvd: 66

[root@h104 ~]# 
~~~



---

## 停止服务

终端前台运行的情况下 **`Ctrl-C`** 可以平滑地停止服务

~~~
...
...
...
    2016/03/18 16:41:30 [DEBUG] agent: Service 'consul' in sync
    2016/03/18 16:43:12 [DEBUG] agent: Service 'consul' in sync
    2016/03/18 16:44:22 [DEBUG] agent: Service 'consul' in sync
    2016/03/18 16:45:46 [DEBUG] agent: Service 'consul' in sync
^C==> Caught signal: interrupt
==> Gracefully shutting down agent...
    2016/03/18 16:46:58 [INFO] consul: server starting leave
    2016/03/18 16:46:58 [INFO] serf: EventMemberLeave: h104.dc1 192.168.100.104
    2016/03/18 16:46:58 [INFO] serf: EventMemberLeave: h104 192.168.100.104
    2016/03/18 16:46:58 [DEBUG] http: Shutting down http server (127.0.0.1:8500)
    2016/03/18 16:46:58 [INFO] agent: requesting shutdown
    2016/03/18 16:46:58 [INFO] consul: shutting down server
    2016/03/18 16:46:58 [ERR] dns: error starting tcp server: accept tcp 127.0.0.1:8600: use of closed network connection
    2016/03/18 16:46:58 [INFO] agent: shutdown complete
[root@h104 consul]#
[root@h104 consul]# consul members 
Error connecting to Consul agent: dial tcp 127.0.0.1:8400: getsockopt: connection refused
[root@h104 consul]# netstat  -tunpea | grep consul
[root@h104 consul]# 
~~~

---

## 注册服务

### 定义服务

定义服务可以使用两种方法：

* 使用配置文件进行服务定义
* 调用 **HTTP API** 进行定义

Consul 会加载配置目录中的所有配置文件，配置文件是以 **.json** 结尾的，并且以字典顺序加载

创建配置文件

~~~
[root@h104 consul]# mkdir /etc/consul.d
[root@h104 consul]# echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' > /etc/consul.d/web.json
[root@h104 consul]# cat /etc/consul.d/web.json 
{"service": {"name": "web", "tags": ["rails"], "port": 80}}
[root@h104 consul]#
~~~

指定配置文件启动服务

~~~
[root@h104 consul]# consul agent -dev -bind=192.168.100.104 -config-dir /etc/consul.d/
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'h104'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 192.168.100.104 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/03/18 17:07:20 [INFO] serf: EventMemberJoin: h104 192.168.100.104
    2016/03/18 17:07:20 [INFO] serf: EventMemberJoin: h104.dc1 192.168.100.104
    2016/03/18 17:07:20 [INFO] raft: Node at 192.168.100.104:8300 [Follower] entering Follower state
    2016/03/18 17:07:20 [INFO] consul: adding LAN server h104 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 17:07:20 [INFO] consul: adding WAN server h104.dc1 (Addr: 192.168.100.104:8300) (DC: dc1)
    2016/03/18 17:07:20 [ERR] agent: failed to sync remote state: No cluster leader
    2016/03/18 17:07:22 [WARN] raft: Heartbeat timeout reached, starting election
    2016/03/18 17:07:22 [INFO] raft: Node at 192.168.100.104:8300 [Candidate] entering Candidate state
    2016/03/18 17:07:22 [DEBUG] raft: Votes needed: 1
    2016/03/18 17:07:22 [DEBUG] raft: Vote granted from 192.168.100.104:8300. Tally: 1
    2016/03/18 17:07:22 [INFO] raft: Election won. Tally: 1
    2016/03/18 17:07:22 [INFO] raft: Node at 192.168.100.104:8300 [Leader] entering Leader state
    2016/03/18 17:07:22 [INFO] raft: Disabling EnableSingleNode (bootstrap)
    2016/03/18 17:07:22 [DEBUG] raft: Node 192.168.100.104:8300 updated peer set (2): [192.168.100.104:8300]
    2016/03/18 17:07:22 [INFO] consul: cluster leadership acquired
    2016/03/18 17:07:22 [DEBUG] consul: reset tombstone GC to index 2
    2016/03/18 17:07:22 [INFO] consul: member 'h104' joined, marking health alive
    2016/03/18 17:07:22 [INFO] consul: New leader elected: h104
    2016/03/18 17:07:25 [INFO] agent: Synced service 'consul'
    2016/03/18 17:07:25 [INFO] agent: Synced service 'web'
    2016/03/18 17:08:44 [DEBUG] dns: request for {h104.node.consul. 1 1} (396.33µs) from client 127.0.0.1:48961 (udp)
    2016/03/18 17:09:15 [DEBUG] agent: Service 'consul' in sync
    2016/03/18 17:09:15 [DEBUG] agent: Service 'web' in sync
...
...
...
~~~

输出中表示 **Synced service 'web'**，说明定义的服务已经成功被注册进来了，如果要以配置文件的形式注册更多的服务，可以在配置目录中添加其它的服务定义

---

## 查看服务

### 使用DNS API查看

可以使用 DNS API的方式查看服务对应的IP



~~~
[root@h104 ~]# dig @127.0.0.1 -p 8600  web.service.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 web.service.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59452
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;web.service.consul.		IN	A

;; ANSWER SECTION:
web.service.consul.	0	IN	A	192.168.100.104

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 17:38:04 CST 2016
;; MSG SIZE  rcvd: 70

[root@h104 ~]# 
~~~

也可以使用 DNS API 获取整个 address/port 信息，只要加上 SRV

~~~
[root@h104 ~]# dig @127.0.0.1 -p 8600  web.service.consul SRV

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 web.service.consul SRV
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32826
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;web.service.consul.		IN	SRV

;; ANSWER SECTION:
web.service.consul.	0	IN	SRV	1 1 80 h104.node.dc1.consul.

;; ADDITIONAL SECTION:
h104.node.dc1.consul.	0	IN	A	192.168.100.104

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 17:40:00 CST 2016
;; MSG SIZE  rcvd: 130

[root@h104 ~]#
~~~

我们还可以使用 DNS API 结合 tag 来过滤服务

~~~
[root@h104 ~]# dig @127.0.0.1 -p 8600  rails.web.service.consul 

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.1 <<>> @127.0.0.1 -p 8600 rails.web.service.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15147
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;rails.web.service.consul.	IN	A

;; ANSWER SECTION:
rails.web.service.consul. 0	IN	A	192.168.100.104

;; Query time: 1 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Mar 18 17:43:27 CST 2016
;; MSG SIZE  rcvd: 82

[root@h104 ~]# 
~~~

---

### 使用HTTP API查看 

~~~
[root@h104 ~]# curl http://localhost:8500/v1/catalog/service/web?pretty
[
    {
        "Node": "h104",
        "Address": "192.168.100.104",
        "ServiceID": "web",
        "ServiceName": "web",
        "ServiceTags": [
            "rails"
        ],
        "ServiceAddress": "",
        "ServicePort": 80,
        "ServiceEnableTagOverride": false,
        "CreateIndex": 5,
        "ModifyIndex": 5
    }
][root@h104 ~]# 
[root@h104 ~]# 
~~~

查看服务的健康状态

~~~
[root@h104 ~]# curl http://localhost:8500/v1/health/service/web?passing
[{"Node":{"Node":"h104","Address":"192.168.100.104","TaggedAddresses":{"wan":"192.168.100.104"},"CreateIndex":3,"ModifyIndex":5},"Service":{"ID":"web","Service":"web","Tags":["rails"],"Address":"","Port":80,"EnableTagOverride":false,"CreateIndex":5,"ModifyIndex":5},"Checks":[{"Node":"h104","CheckID":"serfHealth","Name":"Serf Health Status","Status":"passing","Notes":"","Output":"Agent alive and reachable","ServiceID":"","ServiceName":"","CreateIndex":3,"ModifyIndex":3}]}][root@h104 ~]# 
[root@h104 ~]# 
[root@h104 ~]# curl http://localhost:8500/v1/health/service/web?pretty
[
    {
        "Node": {
            "Node": "h104",
            "Address": "192.168.100.104",
            "TaggedAddresses": {
                "wan": "192.168.100.104"
            },
            "CreateIndex": 3,
            "ModifyIndex": 5
        },
        "Service": {
            "ID": "web",
            "Service": "web",
            "Tags": [
                "rails"
            ],
            "Address": "",
            "Port": 80,
            "EnableTagOverride": false,
            "CreateIndex": 5,
            "ModifyIndex": 5
        },
        "Checks": [
            {
                "Node": "h104",
                "CheckID": "serfHealth",
                "Name": "Serf Health Status",
                "Status": "passing",
                "Notes": "",
                "Output": "Agent alive and reachable",
                "ServiceID": "",
                "ServiceName": "",
                "CreateIndex": 3,
                "ModifyIndex": 3
            }
        ]
    }
][root@h104 ~]# 
[root@h104 ~]# 
~~~

服务是可以使用 HTTP API 进行动态修改 (HTTP API 可以用来进行动态的添加，删除，修改服务)

---

# 命令汇总

* **`wget https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_linux_amd64.zip`**
* **`sha256sum consul_0.6.4_linux_amd64.zip`**
* **`unzip consul_0.6.4_linux_amd64.zip`**
* **`file consul`**
* **`./consul --help`**
* **`./consul version`**
* **`cp consul /usr/local/bin/`**
* **`consul version`**
* **`consul agent -dev -bind=192.168.100.104`**
* **`netstat  -tunpea | grep consul`**
* **`consul members --help`**
* **`consul members -detailed`**
* **`curl localhost:8500/v1/catalog/nodes?pretty`**
* **`netstat  -tunpea | grep consul | grep 8600`**
* **`dig @127.0.0.1 -p 8600  h104.node.consul`**
* **`consul members`**
* **`mkdir /etc/consul.d`**
* **`echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' > /etc/consul.d/web.json`**
* **`cat /etc/consul.d/web.json`**
* **`consul agent -dev -bind=192.168.100.104 -config-dir /etc/consul.d/`**
* **`dig @127.0.0.1 -p 8600  web.service.consul`**
* **`dig @127.0.0.1 -p 8600  web.service.consul SRV`**
* **`dig @127.0.0.1 -p 8600  rails.web.service.consul`**
* **`curl http://localhost:8500/v1/health/service/web?passing`**
* **`curl http://localhost:8500/v1/health/service/web?pretty`**



---





[consul]:https://www.consul.io/
[consul_doc]:https://www.consul.io/docs/index.html
[consul_dl]:https://www.consul.io/downloads.html
[checksum]:https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_SHA256SUMS

