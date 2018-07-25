---
layout: post
title: "Install Rancher 3"
author:  wilmosfang
date: 2018-05-06 16:01:34
image: '/assets/img/'
excerpt: '安装 Rancher'
main-class: 'rancher'
color: '#2980b8'
tags:
 - docker
 - rancher
categories: 
 - rancher
twitter_text: 'simple process of Rancher installation'
introduction: 'Installation of Rancher 2.0 GA'
---


# 前言

**[Rancher][rancher]** 是一款开源的容器管理软件

>Rancher is open source software that combines everything an organization needs to adopt and run containers in production
>
>Rancher is enterprise management for Kubernetes. Every distro. Every cluster. Every cloud

**[Rancher][rancher]** 的设计目标的是简化容器的管理操作，提升容器应用的操作效率

>Built on Kubernetes, Rancher makes it easy for DevOps teams to test, deploy and manage their applications. Operations teams use Rancher to deploy, manage and secure every Kubernetes deployment regardless of where it is running

因为整合了 k8s 的编排功能, 并且有着非常友好的操作界面，所以在目前的容器技术圈中有着很大的影响力

如果要快速构建一套 **CI/CD** 发布平台， **[Rancher][rancher]** 是一个不错的选择

因为在写文档期间 Rancher 从 2.0 的 bata 版本升级到了 GA 版本，所以决定基于 2.0 的 GA 版本重新布署一遍

这里演示一下如何构建 **[Rancher][rancher]**

参考 **[Quick Start Guide][rancher_guide]**

> **Tip:** 当前的版本为 **Rancher 2.0 GA**

---

## 依赖

### 操作系统

* Ubuntu 16.04 (64-bit)
* Red Hat Enterprise Linux 7.5 (64-bit)
* RancherOS 1.3.0 (64-bit)

### 硬件

* Memory: 4GB

### 软件

需要 Docker， 以下版本都是支持的

* 1.12.6
* 1.13.1
* 17.03.2

**Tip:** Docker 的安装方法可以参考 **[Install Docker][docker_install]** ，也可以参考前面写的文章 

---

## 运行环境

~~~
root@rancher:~# hostnamectl 
   Static hostname: rancher
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4bae573cc9bc7f877992f1885ae0c42b
           Boot ID: c672b090f61f40cbbcb68ec25d0b99ab
    Virtualization: oracle
  Operating System: Ubuntu 16.04.4 LTS
            Kernel: Linux 4.4.0-116-generic
      Architecture: x86-64
root@rancher:~# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:87:b2:4d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe87:b24d/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ae:1b:a6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.152/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feae:1ba6/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:3a:9c:d0:e3 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:3aff:fe9c:d0e3/64 scope link 
       valid_lft forever preferred_lft forever
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           17.03.2~ce-0~ubuntu-xenial                 amd64        Docker: the open-source application container engine
root@rancher:~# docker version
Client:
 Version:      17.03.2-ce
 API version:  1.27
 Go version:   go1.7.5
 Git commit:   f5ec1e2
 Built:        Tue Jun 27 03:35:14 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.2-ce
 API version:  1.27 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   f5ec1e2
 Built:        Tue Jun 27 03:35:14 2017
 OS/Arch:      linux/amd64
 Experimental: false
root@rancher:~# 
~~~

## 安装 Rancher

~~~
root@rancher:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                   PORTS               NAMES
dfaf2be1374d        hello-world         "/hello"            10 days ago         Exited (0) 10 days ago                       angry_montalcini
root@rancher:~# sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
Unable to find image 'rancher/rancher:latest' locally
latest: Pulling from rancher/rancher
68393378db12: Pull complete 
9e3366501e0e: Pull complete 
156ec05da9a5: Pull complete 
281cba1133d9: Pull complete 
0acdc2cc8ed1: Pull complete 
a8cef3d8a877: Pull complete 
3e968117f1c2: Pull complete 
cf62fef10dfd: Pull complete 
098edd097869: Pull complete 
77a837c0bf2d: Pull complete 
Digest: sha256:38839bb19bdcac084a413a4edce7efb97ab99b6d896bda2f433dfacfd27f8770
Status: Downloaded newer image for rancher/rancher:latest
e470b6fb6d273b461638b6a310d9fb3a094c735c7bc14b5beb558b22937cc96f
root@rancher:~# echo $?
0
root@rancher:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                   PORTS                                      NAMES
e470b6fb6d27        rancher/rancher     "rancher --http-li..."   About a minute ago   Up About a minute        0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   eloquent_bhabha
dfaf2be1374d        hello-world         "/hello"                 10 days ago          Exited (0) 10 days ago                                              angry_montalcini
root@rancher:~# 
~~~

## 登录

访问 http://h152

会进行自动跳转到 https

![rancher](/assets/img/rancher/rancher12.png)

配置密码

![rancher](/assets/img/rancher/rancher13.png)

![rancher](/assets/img/rancher/rancher14.png)

配置链接

![rancher](/assets/img/rancher/rancher15.png)

创建集群

![rancher](/assets/img/rancher/rancher16.png)

![rancher](/assets/img/rancher/rancher17.png)

将命令复制出来

在系统中执行

~~~
root@rancher:~# sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.0 --server https://192.168.56.152 --token dk4t7dn6zj5n8ktq64tssgsvqz88sjc5qfxmdbqkrndj8m4gnlxh9t --ca-checksum af311e47c591f3d89ca209ce6e20ea9d4c98aad6d745db7fd8fada67d3aae997 --etcd --controlplane --worker
Unable to find image 'rancher/rancher-agent:v2.0.0' locally
v2.0.0: Pulling from rancher/rancher-agent
68393378db12: Already exists 
9e3366501e0e: Already exists 
156ec05da9a5: Already exists 
281cba1133d9: Already exists 
0acdc2cc8ed1: Already exists 
e35932568ac7: Pull complete 
a41629a1241c: Pull complete 
38b1ed3b441c: Pull complete 
1d629d93c823: Pull complete 
Digest: sha256:adb59796bd9c1528eebd0357debf6f25457b42a58b1270b74883aff07e02b356
Status: Downloaded newer image for rancher/rancher-agent:v2.0.0
70d47a38ea9769731e705b67b9ec2e38d286ca4f2900dc7b2f6fd1b638de7146
root@rancher:~# echo $?
0
root@rancher:~# docker ps -a 
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                   PORTS                                      NAMES
627036dedbbf        rancher/rancher-agent:v2.0.0   "run.sh -- share-r..."   11 seconds ago      Up 10 seconds                                                       share-mnt
70d47a38ea97        rancher/rancher-agent:v2.0.0   "run.sh --server h..."   12 seconds ago      Up 11 seconds                                                       clever_curran
4ffb830d13cc        rancher/rancher                "rancher --http-li..."   21 minutes ago      Up 21 minutes            0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   peaceful_hodgkin
dfaf2be1374d        hello-world                    "/hello"                 11 days ago         Exited (0) 11 days ago                                              angry_montalcini
root@rancher:~# 
~~~


一段时间后，集群状态为 Active

![rancher](/assets/img/rancher/rancher18.png)


![rancher](/assets/img/rancher/rancher19.png)

---

# 总结

安装 Rancher 的过程都是容器化的，并且十分高效和简捷


* TOC
{:toc}

---

[rancher]:https://rancher.com/
[rancher_guide]:https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/
[rancher_requirements]:https://rancher.com/docs/rancher/v2.0/en/quick-start-guide/#host-and-node-requirements
[docker_install]:https://docs.docker.com/install/linux/docker-ce/ubuntu/