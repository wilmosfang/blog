---
layout: post
title:  Docker Swarm 基础
author: wilmosfang
categories:   docker 
tags:   cluster docker swarm 
wc: 889 3830 48502
excerpt: swarm 的架构与依赖环境，发现服务的安装，swarm 的下载安装，集群构建，管理节点代理节点的添加，可能产生的问题，swarm 的使用，状态查看，容器运行，故障转移
comments: true
---



# 前言

**[Docker Swarm][swarm]** 是一个原生的 Docker 集群工具

>Docker Swarm is native clustering for Docker. It turns a pool of Docker hosts into a single, virtual Docker host. Because Docker Swarm serves the standard Docker API, any tool that already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts. 

目前的Docker集群解决方案有：

| NAME|COMPANY| +| -|
| :----|:--- | :----: | :---: |
| Swarm|Docker| 原生，简单，集成方便 | 复杂调度支持困难    |
| Fleet|CoreOS| 轻量 | 低级别，较底层 |
| Mesos|Apache| 成熟稳定，架构宏大，强扩展性 | 偏向基层，主要针对的是大型数据中心计算资源分配调度|
| Kubernetes|Google| 成熟稳定，强扩展性|自成体系，复杂，侵入性强，不能简单集成，得对现有应用重新设计|

详细的对比可以参考 **[Swarm v. Fleet v. Kubernetes v. Mesos][compare]**

总体来讲 **Kubernetes** 和 **Mesos** 较为成熟，**Docker Swarm** 还在快速的成长过程中

![swarm2.png](/images/docker/swarm2.png)

由于 **[Docker Swarm][swarm]** 的原生特性，遵循 **“batteries included but removable”的** 原则，所以对现有架构入侵性不强(松耦合)，比较便于集成

这里分享一下 **[Docker Swarm][swarm]** 的相关操作基础，详细内容可以参考 **[官方文档][swarm]** 


> **Tip:** 当前最新的稳定版本为 **Swarm Version: 1.1.3**

---


# 概要

* TOC
{:toc}



---

## 架构

![swarm1.jpg](/images/docker/swarm1.jpg)




---

## 环境

| Host|IP  | manager|node|consul|
| :------- | ----: | :---: |:---: |
| h104| 192.168.100.104 |  manager0|node0|consul0|
| docker|192.168.100.103|  manager1|node1||


---

## 准备

要在每一台服务器上安装Docker 

相关的安装过程可以参考 **[Docker 基础][docker_basic]**

关键是要使用如下配置启动服务

~~~
[root@h104 ~]# docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
WARN[0000] /!\ DON'T BIND ON ANY IP ADDRESS WITHOUT setting -tlsverify IF YOU DON'T KNOW WHAT YOU'RE DOING /!\ 
INFO[0000] API listen on [::]:2375                      
INFO[0000] API listen on /var/run/docker.sock           
WARN[0000] Usage of loopback devices is strongly discouraged for production use. Please use `--storage-opt dm.thinpooldev` or use `man docker` to refer to dm.thinpooldev section. 
INFO[0000] [graphdriver] using prior storage driver "devicemapper" 
INFO[0000] Firewalld running: true                      
INFO[0000] Default bridge (docker0) is assigned with an IP address 172.17.0.1/16. Daemon option --bip can be used to set a preferred IP address 
INFO[0000] Loading containers: start.                   
........
INFO[0000] Loading containers: done.                    
INFO[0000] Daemon has completed initialization          
INFO[0000] Docker daemon                                 commit=a34a1d5 execdriver=native-0.2 graphdriver=devicemapper version=1.9.1
...
...
...
~~~

> **Note:** 如不指定，默认会以 **`/usr/bin/docker daemon -H fd://`** 的方式启动服务，这将导致 **swarm** 无法通过 **2375** 端口接受请求，**node** 会一直都处于 **Pending** 状态


安装完成后，可以使用 **hello-world** 镜像来检测是否运行成功

~~~
[root@h104 ~]# docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/

[root@h104 ~]# 
~~~



---

## 下载 Swarm 镜像



Docker 将 **Swarm** 也做成了镜像，可以通过 **Docker Swarm** 的官方镜像来构建集群

> **Tip:** 第一次尝试使用任何镜像时，Docker 引擎都会去本地的镜像库里找，如果有，就使用本地的，如果没有就去Docker Hub里找，如不指定版本，默认会使用 **`*:latest`** ，如果本地不是最新的，也会从 Docker Hub 下载

使用 **`docker pull swarm`** 的方式下载 Swarm 镜像

~~~
[root@h104 ~]# docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker:5000/ci/jnkns-img                latest              5b825467fc4f        7 weeks ago         708.2 MB
docker:5000/ci/jnkns-img2               latest              efae71df8aca        7 weeks ago         708.2 MB
jenkins                                 latest              5a0e442d31f6        7 weeks ago         708.4 MB
docker:5000/ubuntu                      latest              8693db7e8a00        8 weeks ago         187.9 MB
ubuntu                                  latest              8693db7e8a00        8 weeks ago         187.9 MB
registry                                2                   683f9cd9cf88        10 weeks ago        224.5 MB
daocloud.io/daocloud/daocloud-toolset   latest              1e743a7453e4        11 weeks ago        145.8 MB
hello-world                             latest              0a6ba66e537a        5 months ago        960 B
[root@h104 ~]# docker search swarm
NAME                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
swarm                            Swarm: a Docker-native clustering system.       227       [OK]       
csanchez/jenkins-swarm                                                           5                    [OK]
ambakshi/perforce-swarm          Swarm enables collaboration and code revie...   1                    [OK]
masato/ambari-swarm              Ambari on Docker Swarm                          1                    [OK]
dockerswarm/swarm                Swarm: a Docker-native clustering system        1                    [OK]
nordluf/swarm-discovery          Service discovery designed for Docker Swar...   1                    [OK]
a1017/swarm                      Docker Swarm image with Flocker volume dri...   1                    [OK]
voxxit/swarm                                                                     0                    [OK]
cloudposse/jenkins-swarm-slave   Jenkins swarm slave service                     0                    [OK]
pitkley/python-swarm             Python on a Jenkins Swarm slave                 0                    [OK]
pitkley/maven-swarm              Maven in a Jenkins Swarm Slave                  0                    [OK]
reallyenglish/swarm              Docker swarm                                    0                    [OK]
fabric8/jenkins-swarm-client     Jenkins swarm client docker image               0                    [OK]
blacklabelops/swarm-aws          Jenkins Slave Container with Amazon WS CLI      0                    [OK]
swarmsim/swarm-server-sails                                                      0                    [OK]
mesosphere/swarm                                                                 0                    [OK]
dockerswarm/swarm-test-env                                                       0                    [OK]
blacklabelops/swarm-dockerhost   Test container for accessing the docker de...   0                    [OK]
blacklabelops/swarm-jdk8         Jenkins Swarm Slave With Oracle JDK8            0                    [OK]
blacklabelops/jenkins-swarm      Jenkins Swarm Slave Dockerized and Paramet...   0                    [OK]
blacklabelops/swarm-jdk7         Jenkins Swarm Slave With Java JDK7, Maven,...   0                    [OK]
blacklabelops/swarm-jdk6         Jenkins Swarm Slave With Java JDK6, Maven,...   0                    [OK]
blacklabelops/swarm-docker       Jenkins Swarm Slave with Docker Cli             0                    [OK]
aratto/jenkins-swarm-slave       A Jenkins+Swarm slave base image                0                    [OK]
doronp/swarm                     build swarm                                     0                    [OK]
[root@h104 ~]# docker pull swarm 
Using default tag: latest
latest: Pulling from library/swarm
d621ed4b2fb9: Pull complete 
9e8ac70babff: Pull complete 
0322bf43f34e: Pull complete 
98c28310973a: Pull complete 
c42833df477a: Pull complete 
4a121ce53126: Pull complete 
09934e0ae8d5: Pull complete 
81127fe5e9b4: Pull complete 
Digest: sha256:5f2b4066b2f7e97a326a8bfcfa623be26ce45c26ffa18ea63f01de045d2238f3
Status: Downloaded newer image for swarm:latest
[root@h104 ~]# docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
swarm                                   latest              81127fe5e9b4        2 weeks ago         18.11 MB
docker:5000/ci/jnkns-img                latest              5b825467fc4f        7 weeks ago         708.2 MB
docker:5000/ci/jnkns-img2               latest              efae71df8aca        7 weeks ago         708.2 MB
jenkins                                 latest              5a0e442d31f6        7 weeks ago         708.4 MB
ubuntu                                  latest              8693db7e8a00        8 weeks ago         187.9 MB
docker:5000/ubuntu                      latest              8693db7e8a00        8 weeks ago         187.9 MB
registry                                2                   683f9cd9cf88        10 weeks ago        224.5 MB
daocloud.io/daocloud/daocloud-toolset   latest              1e743a7453e4        11 weeks ago        145.8 MB
hello-world                             latest              0a6ba66e537a        5 months ago        960 B
[root@h104 ~]# docker images | grep swarm 
swarm                                   latest              81127fe5e9b4        2 weeks ago         18.11 MB
[root@h104 ~]#
~~~

除了使用 Swarm 的镜像，还能使用 Swarm binary 的方式，但是官方不推荐这么用，因为有配置编译安装等一系列“脏活”要干(实在是感兴趣的话可以参考 **[Swarm binary][swarm_bin]** ，主要面向贡献代码的开发人员)，相较而言直接使用 Swarm的镜像有如下好处：

* 不必操心源码的编译
* 不必操心版本和升级的问题(直接就可以获得最新的版本)
* 不必操心环境的依赖(隔离了运行环境，内部解决了依赖)

其实这也是绝大多数应用容器化的初衷，省时，省心，省力

---

## 服务发现

Swarm 需要使用到服务发现机制，发现服务是 Swarm 中极其关键的一环，Swarm 依赖它对集群中的其它节点进行感知和交互，集群的高可用也依赖于它完成，如果服务发现工作不正常，集群将无法操作

Swarm 目前支持四种服务发现工具：

* Hosted (用于测试，不要使用到生产)
* Consul
* etcd
* Zookeeper

下面选择 Consul 作为服务发现工具

~~~
[root@h104 ~]# docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap
Unable to find image 'progrium/consul:latest' locally
latest: Pulling from progrium/consul
3b4d28ce80e4: Pull complete 
e5ab901dcf2d: Pull complete 
30ad296c0ea0: Pull complete 
3dba40dec256: Pull complete 
f2ef4387b95e: Pull complete 
53bc8dcc4791: Pull complete 
75ed0b50ba1d: Pull complete 
17c3a7ed5521: Pull complete 
8aca9e0ecf68: Pull complete 
4d1828359d36: Pull complete 
46ed7df7f742: Pull complete 
b5e8ce623ef8: Pull complete 
049dca6ef253: Pull complete 
bdb608bc4555: Pull complete 
8b3d489cfb73: Pull complete 
c74500bbce24: Pull complete 
9f3e605442f6: Pull complete 
d9125e9e799b: Verifying Checksum 
Pulling repository docker.io/progrium/consul
e66fb6787628: Download complete 
31f630c65071: Download complete 
cadd1fc80511: Download complete 
617a0e174acf: Download complete 
8142b9305a34: Download complete 
8e614066f6a6: Download complete 
8ba0a056d1e3: Download complete 
d79f17b5cf8f: Download complete 
1222ac49a0eb: Download complete 
8d3315362a90: Download complete 
8359e65c65c1: Download complete 
53d68e7edf50: Download complete 
e57fb0e987c9: Download complete 
39dd286a1371: Download complete 
0ed95048b811: Download complete 
d9333eedb08a: Download complete 
18d2154179cc: Download complete 
ff1cf6216ab6: Download complete 
Status: Downloaded newer image for progrium/consul:latest
docker.io/progrium/consul: this image was pulled from a legacy registry.  Important: This registry version will not be supported in future versions of docker.
3b12ab97b20fc65a74936fb632317358845c5318ac654d768da2c98754a80906
[root@h104 ~]# echo $?
0
[root@h104 ~]# 
~~~

多了一个 Consul 的镜像，容器也已经运行起来了

~~~
[root@h104 ~]# docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
swarm                                   latest              81127fe5e9b4        2 weeks ago         18.11 MB
docker:5000/ci/jnkns-img                latest              5b825467fc4f        7 weeks ago         708.2 MB
docker:5000/ci/jnkns-img2               latest              efae71df8aca        7 weeks ago         708.2 MB
jenkins                                 latest              5a0e442d31f6        7 weeks ago         708.4 MB
docker:5000/ubuntu                      latest              8693db7e8a00        8 weeks ago         187.9 MB
ubuntu                                  latest              8693db7e8a00        8 weeks ago         187.9 MB
registry                                2                   683f9cd9cf88        11 weeks ago        224.5 MB
daocloud.io/daocloud/daocloud-toolset   latest              1e743a7453e4        11 weeks ago        145.8 MB
hello-world                             latest              0a6ba66e537a        5 months ago        960 B
progrium/consul                         latest              e66fb6787628        8 months ago        69.4 MB
<none>                                  <none>              9f3e605442f6        8 months ago        69.4 MB
[root@h104 ~]#
[root@h104 ~]# docker  ps -a 
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                     PORTS                                                                            NAMES
3b12ab97b20f        progrium/consul             "/bin/start -server -"   4 minutes ago        Up 4 minutes               53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp   consul
236348a3c9ff        docker:5000/ci/jnkns-img2   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                    jenkins01
[root@h104 ~]# 
~~~


---

## 创建 Swarm 集群 


有了发现服务作基础，接下来就要创建 Swarm 管理节点，我们创建两个节点(分别在不同的服务器上)来模拟高可用架构


### 创建第一个管理节点

~~~
[root@h104 ~]# docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 192.168.100.104:4000 consul://192.168.100.104:8500
a6a0adaa76a8771bf373998832deaa236d68513bb5f9de0b3051c49761447e1a
[root@h104 ~]# docker ps -a 
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                     PORTS                                                                            NAMES
a6a0adaa76a8        swarm                       "/swarm manage -H :40"   3 seconds ago       Up 1 seconds               2375/tcp, 0.0.0.0:4000->4000/tcp                                                 sad_mestorf
3b12ab97b20f        progrium/consul             "/bin/start -server -"   16 hours ago        Up 20 minutes              53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 0.0.0.0:8500->8500/tcp, 8301-8302/udp   consul
236348a3c9ff        docker:5000/ci/jnkns-img2   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                    jenkins01
[root@h104 ~]#  
~~~

### 创建第二个管理节点

~~~
[root@docker ~]# docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 192.168.100.103:4000 consul://192.168.100.104:8500
de2669846044ea05851f69a643846d55b0f87c1f1d9abd29bcb90c71fa91bb0f
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                   PORTS                              NAMES
de2669846044        swarm                         "/swarm manage -H :40"   3 seconds ago       Up 2 seconds             2375/tcp, 0.0.0.0:4000->4000/tcp   high_wright
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                                      jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 2 hours               0.0.0.0:5000->5000/tcp             registry
[root@docker ~]# 
~~~

> **Note:**  涉及的网络端口有必要在防火墙上放行，打开方法

~~~
[root@docker ~]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 3306/tcp 80/tcp 40000/tcp 8080/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@docker ~]# firewall-cmd --add-port 4000/tcp
success
[root@docker ~]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 3306/tcp 80/tcp 40000/tcp 8080/tcp 4000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@docker ~]#
~~~

有必要打开的端口为 **4000/tcp、2375/tcp、8500/tcp**

PORT  | Comment
-------- | ---
4000/tcp| 管理节点用来接受请求的端口
2375/tcp| Docker 引擎接受请求的端口
8500/tcp| 发现服务用来接受请求的端口


此时已经可以使用命令对管理节点发送请求

~~~
[root@h104 ~]# docker -H :4000 info 
Containers: 0
Images: 0
Server Version: swarm/1.1.3
Role: replica
Primary: 192.168.100.103:4000
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 0
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 0
Total Memory: 0 B
Name: a6a0adaa76a8
[root@h104 ~]# docker -H :4000 ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@h104 ~]# 
----------
[root@docker ~]# docker -H :4000 info
Containers: 0
Images: 0
Server Version: swarm/1.1.3
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 0
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 0
Total Memory: 0 B
Name: de2669846044
[root@docker ~]# docker -H :4000 ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker ~]#
~~~

可见通过投票自动选举出103为主节点，104为备份节点，主节点是投票选出的而不是谁先加入谁就一定是主节点，103和104上都有运行中的容器，但目前还看不到，因为没有安装swarm代理节点


---


### 添加一个节点

安装完swarm代理节点后就可以通过管理节点使用到该服务器上的资源

~~~
[root@h104 ~]# docker run -d swarm join --advertise=192.168.100.104:2375 consul://192.168.100.104:8500
055469770d50b477642717e3ebcd795eca26806bc1d55a547d60ac4559991b79
[root@h104 ~]# docker ps -a 
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                     PORTS                                                                            NAMES
055469770d50        swarm                       "/swarm join --advert"   3 seconds ago       Up 1 seconds               2375/tcp                                                                         sharp_engelbart
a6a0adaa76a8        swarm                       "/swarm manage -H :40"   50 minutes ago      Up 50 minutes              2375/tcp, 0.0.0.0:4000->4000/tcp                                                 sad_mestorf
3b12ab97b20f        progrium/consul             "/bin/start -server -"   17 hours ago        Up About an hour           53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp   consul
236348a3c9ff        docker:5000/ci/jnkns-img2   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                    jenkins01
[root@h104 ~]# docker -H :4000 info 
Containers: 4
Images: 9
Server Version: swarm/1.1.3
Role: replica
Primary: 192.168.100.103:4000
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 1
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 4
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T06:44:50Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 2
Total Memory: 2.044 GiB
Name: a6a0adaa76a8
[root@h104 ~]# 
~~~

### 添加另一个节点

~~~
[root@docker ~]# docker run -d swarm join --advertise=192.168.100.103:2375 consul://192.168.100.104:8500
592ca6995b4d66344686d588f066db6a6dc7018e45052704bdf9728d36cca807
[root@docker ~]# docker -H :4000 info
Containers: 8
Images: 17
Server Version: swarm/1.1.3
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 docker: 192.168.100.103:2375
  └ Status: Healthy
  └ Containers: 4
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 4.045 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T06:45:32Z
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 4
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T06:45:49Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 4
Total Memory: 6.09 GiB
Name: de2669846044
[root@docker ~]# 
~~~

此时，Swarm 的集群已经构建完成和成功启动，同时符合高可用的架构，并且可以通过添加更多的服务发现节点，swarm管理节点，普通swarm节点来进一步提升系统的稳定性、可用性和负载能力

---

### 可能产生的错误

在VM环境下，如果通过克隆虚拟机或拷贝软件目录的方式创建新的docker实例，可能会遇到下面的问题

~~~
[root@h104 ~]# docker -H :4000 info 
Containers: 7
Images: 9
Server Version: swarm/1.1.3
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 docker: 192.168.100.103:2375
  └ Status: Pending
  └ Containers: 4
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 4.045 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: ID duplicated. TDJN:N3S4:3NPB:URQ6:OFVC:3DYQ:QMKH:LZYW:CBFU:OWZY:GMXK:LPV7 shared by this node 192.168.100.103:2375 and another node 192.168.100.104:2375
  └ UpdatedAt: 2016-03-22T13:33:46Z
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 7
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-22T13:34:16Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 2
Total Memory: 2.044 GiB
Name: a8f16aa3a7d1
[root@h104 ~]#
~~~

其中 **103** 处于 **Pending** 的状态，有 **Error: ID duplicated.** 的报错，表明ID有冲突

解决办法是重新生成Key

首先备份一下 **/etc/docker/key.json**

~~~
[root@docker ~]# mv /etc/docker/key.json  /tmp/
[root@docker ~]# ll /etc/docker/key.json 
ls: cannot access /etc/docker/key.json: No such file or directory
[root@docker ~]#
~~~

然后重启Docker 服务，**/etc/docker/** 目录下会重新生成新的 **key.json** 


~~~
[root@docker ~]# cat /tmp/key.json 
{
    "crv": "P-256",
    "d": "c7V1nUG73Ze-FPV-_50uvRe-LwMSRvMIojSDpLfNk3U",
    "kid": "TDJN:N3S4:3NPB:URQ6:OFVC:3DYQ:QMKH:LZYW:CBFU:OWZY:GMXK:LPV7",
    "kty": "EC",
    "x": "Kw6ybSVcLXedDU3Sab03M3Nz4kLUeu1WOKmqzJCSaMM",
    "y": "RrG9h0OFt5TyRLmyBzPgsfe0ZNH6w9MEt2aN_C5YEDk"
}[root@docker ~]# cat /etc/docker/key.json 
{
    "crv": "P-256",
    "d": "lNojr-xwvXz252t4S97UCL2LDN0dkbuI5zIbZSCMfqY",
    "kid": "7DPV:ENKF:APVK:XISF:HT66:QICK:WTD4:BQAS:B27U:XNGR:2FPU:W4BH",
    "kty": "EC",
    "x": "9W4wJfqvetoJY5VcMz-pJHMqZxDz_u0ZqRKPF4FKegs",
    "y": "MEHvblUpJkCyMD5GTbk7sbl5NgUS5ZTGzB2BXf0Wa-E"
}[root@docker ~]#
~~~

再进行检查，状态就正常了


---

## 使用Swarm

由于Swarm的原生特性，对于Docker引擎的命令大部分都可以直接使用，就像使用单个本地Docker服务一样地使用一群Docker引擎

### 查看容器状态

~~~
[root@docker ~]# docker -H :4000 ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                    NAMES
3b12ab97b20f        progrium/consul     "/bin/start -server -"   18 hours ago        Up About an hour    53/tcp, 53/udp, 8300-8302/tcp, 8301-8302/udp, 192.168.100.104:8500->8500/tcp, 8400/tcp   h104/consul
71de3ba93794        registry:2          "/bin/registry /etc/d"   8 weeks ago         Up 4 hours          192.168.100.103:5000->5000/tcp                                                           docker/registry
[root@docker ~]# docker -H :4000 ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                     PORTS                                                                                    NAMES
592ca6995b4d        swarm                         "/swarm join --advert"   37 minutes ago      Up 37 minutes              2375/tcp                                                                                 docker/suspicious_joliot
055469770d50        swarm                         "/swarm join --advert"   38 minutes ago      Up 38 minutes              2375/tcp                                                                                 h104/sharp_engelbart
de2669846044        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour           2375/tcp, 192.168.100.103:4000->4000/tcp                                                 docker/high_wright
a6a0adaa76a8        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour           2375/tcp, 192.168.100.104:4000->4000/tcp                                                 h104/sad_mestorf
3b12ab97b20f        progrium/consul               "/bin/start -server -"   18 hours ago        Up About an hour           53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 192.168.100.104:8500->8500/tcp   h104/consul
236348a3c9ff        docker:5000/ci/jnkns-img2     "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                            h104/jenkins01
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                                                                                              docker/jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 4 hours                 192.168.100.103:5000->5000/tcp                                                           docker/registry
[root@docker ~]# 
~~~

---

### 运行容器


虽然104是备份管理节点，但是它依然可以接受命令，它会自动将命令路由给主管理节点，主管理节点再将命令发布给合适的代理节点执行

~~~
[root@h104 ~]# docker -H :4000 run hello-world
[root@h104 ~]# docker -H :4000 run hello-world
[root@h104 ~]# docker -H :4000 run hello-world
[root@h104 ~]# docker -H :4000 run hello-world
[root@h104 ~]#
[root@h104 ~]# docker -H :4000 ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                      PORTS                                                                                    NAMES
8d7c107be5fa        hello-world                   "/hello"                 39 seconds ago      Exited (0) 38 seconds ago                                                                                            docker/high_hopper
ff67eb10e45f        hello-world                   "/hello"                 40 seconds ago      Exited (0) 39 seconds ago                                                                                            h104/furious_lovelace
2f5af4ab2bcc        hello-world                   "/hello"                 42 seconds ago      Exited (0) 41 seconds ago                                                                                            docker/stupefied_booth
581f5ffa23f4        hello-world                   "/hello"                 46 seconds ago      Exited (0) 45 seconds ago                                                                                            h104/small_payne
592ca6995b4d        swarm                         "/swarm join --advert"   49 minutes ago      Up 49 minutes               2375/tcp                                                                                 docker/suspicious_joliot
055469770d50        swarm                         "/swarm join --advert"   50 minutes ago      Up 50 minutes               2375/tcp                                                                                 h104/sharp_engelbart
de2669846044        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour            2375/tcp, 192.168.100.103:4000->4000/tcp                                                 docker/high_wright
a6a0adaa76a8        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour            2375/tcp, 192.168.100.104:4000->4000/tcp                                                 h104/sad_mestorf
3b12ab97b20f        progrium/consul               "/bin/start -server -"   18 hours ago        Up 2 hours                  53/tcp, 53/udp, 8300-8302/tcp, 8301-8302/udp, 8400/tcp, 192.168.100.104:8500->8500/tcp   h104/consul
236348a3c9ff        docker:5000/ci/jnkns-img2     "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                             h104/jenkins01
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                                                                                               docker/jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 4 hours                  192.168.100.103:5000->5000/tcp                                                           docker/registry
[root@h104 ~]# 
~~~

我连续创建了四个容器，从输出可以看出，它们是平均分布的(两个在h104上，两个在docker上)，这个策略由 **info** 中的 **Strategy: spread** 决定




---

## 故障转移


由于当前的主管理节点是103，我们删掉它的容器，看看会发生什么


我们使用Swarm自已来删除主管理节点

~~~
[root@h104 ~]# docker -H :4000 info 
Containers: 12
Images: 17
Server Version: swarm/1.1.3
Role: replica
Primary: 192.168.100.103:4000
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 docker: 192.168.100.103:2375
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 4.045 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T07:50:42Z
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T07:50:38Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 4
Total Memory: 6.09 GiB
Name: a6a0adaa76a8
[root@h104 ~]# 
[root@h104 ~]# docker -H :4000 ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                      PORTS                                                                                    NAMES
8d7c107be5fa        hello-world                   "/hello"                 17 minutes ago      Exited (0) 17 minutes ago                                                                                            docker/high_hopper
ff67eb10e45f        hello-world                   "/hello"                 17 minutes ago      Exited (0) 17 minutes ago                                                                                            h104/furious_lovelace
2f5af4ab2bcc        hello-world                   "/hello"                 17 minutes ago      Exited (0) 17 minutes ago                                                                                            docker/stupefied_booth
581f5ffa23f4        hello-world                   "/hello"                 17 minutes ago      Exited (0) 17 minutes ago                                                                                            h104/small_payne
592ca6995b4d        swarm                         "/swarm join --advert"   About an hour ago   Up About an hour            2375/tcp                                                                                 docker/suspicious_joliot
055469770d50        swarm                         "/swarm join --advert"   About an hour ago   Up About an hour            2375/tcp                                                                                 h104/sharp_engelbart
de2669846044        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour            2375/tcp, 192.168.100.103:4000->4000/tcp                                                 docker/high_wright
a6a0adaa76a8        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour            2375/tcp, 192.168.100.104:4000->4000/tcp                                                 h104/sad_mestorf
3b12ab97b20f        progrium/consul               "/bin/start -server -"   18 hours ago        Up 2 hours                  53/tcp, 53/udp, 8300-8302/tcp, 8301-8302/udp, 8400/tcp, 192.168.100.104:8500->8500/tcp   h104/consul
236348a3c9ff        docker:5000/ci/jnkns-img2     "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                             h104/jenkins01
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                                                                                               docker/jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 4 hours                  192.168.100.103:5000->5000/tcp                                                           docker/registry
[root@h104 ~]# docker -H :4000 rm -f de2669846044
An error occurred trying to connect: EOF
Error: failed to remove containers: [de2669846044]
[root@h104 ~]# docker -H :4000 ps -a 
Error response from daemon: Unable to reach primary cluster manager (dial tcp 192.168.100.103:4000: getsockopt: connection refused): 192.168.100.103:4000
[root@h104 ~]# docker -H :4000 ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                      PORTS                                                                                    NAMES
8d7c107be5fa        hello-world                   "/hello"                 18 minutes ago      Exited (0) 18 minutes ago                                                                                            docker/high_hopper
ff67eb10e45f        hello-world                   "/hello"                 18 minutes ago      Exited (0) 18 minutes ago                                                                                            h104/furious_lovelace
2f5af4ab2bcc        hello-world                   "/hello"                 18 minutes ago      Exited (0) 18 minutes ago                                                                                            docker/stupefied_booth
581f5ffa23f4        hello-world                   "/hello"                 18 minutes ago      Exited (0) 18 minutes ago                                                                                            h104/small_payne
592ca6995b4d        swarm                         "/swarm join --advert"   About an hour ago   Up About an hour            2375/tcp                                                                                 docker/suspicious_joliot
055469770d50        swarm                         "/swarm join --advert"   About an hour ago   Up About an hour            2375/tcp                                                                                 h104/sharp_engelbart
a6a0adaa76a8        swarm                         "/swarm manage -H :40"   About an hour ago   Up About an hour            2375/tcp, 192.168.100.104:4000->4000/tcp                                                 h104/sad_mestorf
3b12ab97b20f        progrium/consul               "/bin/start -server -"   18 hours ago        Up 2 hours                  53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 192.168.100.104:8500->8500/tcp   h104/consul
236348a3c9ff        docker:5000/ci/jnkns-img2     "/bin/tini -- /usr/lo"   7 weeks ago         Exited (143) 4 weeks ago                                                                                             h104/jenkins01
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                                                                                               docker/jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 4 hours                  192.168.100.103:5000->5000/tcp                                                           docker/registry
[root@h104 ~]# docker -H :4000 info 
Containers: 11
Images: 17
Server Version: swarm/1.1.3
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 docker: 192.168.100.103:2375
  └ Status: Healthy
  └ Containers: 5
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 4.045 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T07:53:00Z
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T07:53:17Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 4
Total Memory: 6.09 GiB
Name: a6a0adaa76a8
[root@h104 ~]# 
~~~

虽然途中有一个报错，但是还是成功执行了，为什么呢？

那是因为管理节点要处理一条命令，自已删除自己，看起来很矛盾，但是实际上，它只是传达了这条命令，真正执行的是swarm的代理节点，整个过程是这样的：

备份管理节点收到一条干掉主节点容器的命令，然后路由给主管理节点，主管理节点收到后，报了一个错，尼玛这是要干死我呀，不过还是忠实的传给了能正确执行这个命令的代理节点，代理节点很听话地这么做了，这个过程中没有了主节点，consul将原主节点标记为不可用，主节点开始重新投票选举，在新的主节点被选出之前集群是不能被操作的，这时我发了一个ps的查看命令，想知道当前swarm中的容器状态，但是没有主节点，所以报错了，紧接着新的主管理节点选举产生了，我使用了同样的命令就有了当前的反馈结果

最后104自动切换换成了primary ，再去 103上看本地的容器状态

~~~
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                      PORTS                    NAMES
8d7c107be5fa        hello-world                   "/hello"                 18 minutes ago      Exited (0) 18 minutes ago                            high_hopper
2f5af4ab2bcc        hello-world                   "/hello"                 18 minutes ago      Exited (0) 18 minutes ago                            stupefied_booth
592ca6995b4d        swarm                         "/swarm join --advert"   About an hour ago   Up About an hour            2375/tcp                 suspicious_joliot
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                               jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 4 hours                  0.0.0.0:5000->5000/tcp   registry
[root@docker ~]# docker -H :4000 ps 
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
[root@docker ~]#
[root@docker ~]# docker -H :4000 info 
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
[root@docker ~]# docker -H 192.168.100.104:4000 info 
Containers: 11
Images: 17
Server Version: swarm/1.1.3
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 docker: 192.168.100.103:2375
  └ Status: Healthy
  └ Containers: 5
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 4.045 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T08:17:49Z
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T08:17:46Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 4
Total Memory: 6.09 GiB
Name: a6a0adaa76a8
[root@docker ~]#  
~~~

没有了那个管理节点容器，并且对Swarm的管理命令无法执行

现在加回来

~~~
[root@docker ~]# docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 192.168.100.103:4000 consul://192.168.100.104:8500
d563af1475b5cc2f58e3ec2d0e80224472db86785740736cd628e27c6dee8164
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                      PORTS                              NAMES
d563af1475b5        swarm                         "/swarm manage -H :40"   4 seconds ago       Up 3 seconds                2375/tcp, 0.0.0.0:4000->4000/tcp   sharp_carson
8d7c107be5fa        hello-world                   "/hello"                 44 minutes ago      Exited (0) 44 minutes ago                                      high_hopper
2f5af4ab2bcc        hello-world                   "/hello"                 44 minutes ago      Exited (0) 44 minutes ago                                      stupefied_booth
592ca6995b4d        swarm                         "/swarm join --advert"   About an hour ago   Up About an hour            2375/tcp                           suspicious_joliot
f616e3e353bc        ci-infrastructure/jnkns-img   "/bin/tini -- /usr/lo"   7 weeks ago         Exited (0) 7 weeks ago                                         jenkins01
71de3ba93794        registry:2                    "/bin/registry /etc/d"   8 weeks ago         Up 5 hours                  0.0.0.0:5000->5000/tcp             registry
[root@docker ~]# docker -H :4000 info 
Containers: 12
Images: 17
Server Version: swarm/1.1.3
Role: replica
Primary: 192.168.100.104:4000
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 docker: 192.168.100.103:2375
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 4.045 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T08:18:59Z
 h104: 192.168.100.104:2375
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 2.044 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.10.0-327.4.4.el7.x86_64, operatingsystem=CentOS Linux 7 (Core), storagedriver=devicemapper
  └ Error: (none)
  └ UpdatedAt: 2016-03-23T08:18:59Z
Kernel Version: 3.10.0-327.4.4.el7.x86_64
Operating System: linux
CPUs: 4
Total Memory: 6.09 GiB
Name: d563af1475b5
[root@docker ~]# 
~~~

---

# 命令汇总


* **`docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock`**
* **`docker run hello-world`**
* **`docker search swarm`**
* **`docker pull swarm`**
* **`docker images | grep swarm`**
* **`docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap`**
* **`docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 192.168.100.104:4000 consul://192.168.100.104:8500`**
* **`docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 192.168.100.103:4000 consul://192.168.100.104:8500`**
* **`firewall-cmd --list-all`**
* **`firewall-cmd --add-port 4000/tcp`**
* **`docker -H :4000 info`**
* **`docker -H :4000 ps -a`**
* **`docker run -d swarm join --advertise=192.168.100.104:2375 consul://192.168.100.104:8500`**
* **`docker run -d swarm join --advertise=192.168.100.103:2375 consul://192.168.100.104:8500`**
* **`mv /etc/docker/key.json  /tmp/`**
* **`ll /etc/docker/key.json`**
* **`cat /tmp/key.json`**
* **`docker -H :4000 ps`**
* **`docker -H :4000 ps -a`**
* **`docker -H :4000 run hello-world`**
* **`docker -H :4000 rm -f de2669846044`**
* **`docker ps -a`**
* **`docker -H 192.168.100.104:4000 info`**
* **`docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 192.168.100.103:4000 consul://192.168.100.104:8500`**
* **`docker -H :4000 info`**


---


[swarm]:https://docs.docker.com/swarm/overview/
[compare]:https://www.oreilly.com/ideas/swarm-v-fleet-v-kubernetes-v-mesos
[swarm_bin]:https://github.com/docker/swarm/blob/master/CONTRIBUTING.md
[docker_basic]:http://soft.dog/2016/01/20/docker-basic/

