---
layout: post
title: "Install Rancher 2"
author:  wilmosfang
date: 2018-04-26 16:40:52
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
introduction: 'Installation of Rancher 2.0 bata'
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

前一篇中准备好了 Docker 环境

继续上一篇的内容，这里继续演示如何构建 **[Rancher][rancher]**

参考 **[Quick Start Guide][rancher_guide]**

> **Tip:** 当前的版本为 **RANCHER 2.0 bata**

---

# 操作

## 系统环境

~~~
wilmos@Nothing:~$ ssh bolo@h152
The authenticity of host 'h152 (192.168.56.152)' can't be established.
ECDSA key fingerprint is SHA256:R2LjKHhqC2SPVRzTeLkg/qySIgcKgfnNHxwqXNJcBYI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'h152,192.168.56.152' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Thu Apr 26 09:57:55 2018
bolo@rancher:~$ 
bolo@rancher:~$ hostnamectl 
   Static hostname: rancher
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4bae573cc9bc7f877992f1885ae0c42b
           Boot ID: 0b7b1ad848234f76848ad4f150a41640
    Virtualization: oracle
  Operating System: Ubuntu 16.04.4 LTS
            Kernel: Linux 4.4.0-116-generic
      Architecture: x86-64
bolo@rancher:~$ 
bolo@rancher:~$ ip a 
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
bolo@rancher:~$ 
~~~

## Docker 版本

~~~
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           17.03.2~ce-0~ubuntu-xenial                 amd64        Docker: the open-source application container engine
root@rancher:~# 
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
root@rancher:~# sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/server:preview
Unable to find image 'rancher/server:preview' locally
preview: Pulling from rancher/server
f56c16792d1e: Pull complete 
e2f3fcbcaacd: Pull complete 
29f1503c00f8: Pull complete 
b5ce6675ff39: Pull complete 
a29a9bdc981f: Pull complete 
4758641d8c60: Pull complete 
a30a6a05c4ed: Pull complete 
1ec93119bf5e: Pull complete 
74bc3b84fcb6: Pull complete 
567d9245728b: Pull complete 
Digest: sha256:09507919e8c16395393ab170b3129d3919ab6797d8f7fc1bdfc94cb4f989c151
Status: Downloaded newer image for rancher/server:preview
e4bfdf4beb0d2640afc120524a3c8c2a310bd4e9d5320701576eaebb7f86bd78
root@rancher:~# echo $?
0
root@rancher:~# docker ps -a 
CONTAINER ID        IMAGE                    COMMAND                  CREATED              STATUS                    PORTS                                      NAMES
e4bfdf4beb0d        rancher/server:preview   "rancher --http-li..."   About a minute ago   Up About a minute         0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   hardcore_brattain
dfaf2be1374d        hello-world              "/hello"                 13 hours ago         Exited (0) 13 hours ago                                              angry_montalcini
root@rancher:~# netstat -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.152:22       192.168.56.1:39870      ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 :::443                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
root@rancher:~# 
~~~

基于容器就是这么任性，没有任何配置过程，连安装到运行一条命令解决问题

对于这个版本，只用这条命令

**`sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/server:preview`**

## 进行访问

使用 http 进行访问

![rancher](/assets/img/rancher/rancher01.png)

被自动重定向到了 https

![rancher](/assets/img/rancher/rancher02.png)

因为是自签名证书，所以浏览器提示让我们自己确认

>**Note:** Rancher v2.0 beta:
> * Supports only the HTTPS protocol.
> * Uses a self-signed certificate. Due to this signature, the browser prompts you to trust the certificate before login. Following GA, you’ll be able to use your own certificate.


设定 Rancher 的服务 URL

这个 URL 可以是 IP 或者主机名，但不论是哪一个，后面加入的集群节点都要能解析和连接得上

我这里就直接使用 IP 地址了

![rancher](/assets/img/rancher/rancher03.png)

然后就是配置 admin 用户的密码

可以手动指定，也可以自动生成

我选择自动生成，这个密码复杂度还是蛮强的

![rancher](/assets/img/rancher/rancher04.png)

接下来就进入到了创建集群的界面

>**Tip:** 一个集群就是一组物理的或虚拟的服务器，它们可以共享资源执行任务，对外表现得就好像是一个单一的系统一样

![rancher](/assets/img/rancher/rancher05.png)

点击完添加集群后进入到如下界面

我没有可以对接的云服务，就直接自定义 

选择自定义

>**Note:** Cluster Name 当中不能有空格，连 **`_`** 下划线都不能有，否则会报无效的 API 的错误 

![rancher](/assets/img/rancher/rancher06.png)

配置集群选项

主要是一些集群的属性

这里我留默认值

![rancher](/assets/img/rancher/rancher07.png)

下一步后进入

![rancher](/assets/img/rancher/rancher08.png)

填写对外地址和内网地址

会自动生成节点命令

只需要在工作节点上运行此命令，就可以加入到集群中来

![rancher](/assets/img/rancher/rancher09.png)

确认后，集群会进入布署状态

![rancher](/assets/img/rancher/rancher10.png)

## 创建节点

依据所给的命令行，接合 docker 的基础环境，就可以很方便地创建工作节点

~~~
root@node153:~# docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
dfaf2be1374d        hello-world         "/hello"            14 hours ago        Exited (0) 14 hours ago                       angry_montalcini
root@node153:~# sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/agent:v2.0.0-beta4 --server https://192.168.56.152 --token rg974vs2gss2dgnc8nt7f7d95fbdvhdqmfkt9qdgbdd2lkrgnjkg6q --ca-checksum 70d081b62581b8b81705606b81ba8618482b137feeadba90d27efd1f0e007bd8 --address 10.0.0.1 --internal-address 192.168.56.153 --etcd --worker
Unable to find image 'rancher/agent:v2.0.0-beta4' locally
v2.0.0-beta4: Pulling from rancher/agent
f56c16792d1e: Already exists 
e2f3fcbcaacd: Already exists 
29f1503c00f8: Already exists 
b5ce6675ff39: Already exists 
a29a9bdc981f: Already exists 
7df2ca5bd2b8: Pull complete 
f5a8acd5ddf7: Pull complete 
03bae7f6bd91: Pull complete 
63b0fc3fb97d: Pull complete 
Digest: sha256:dc49864bdc1e959900d7af640c6de640dff8d516c5e9bf5a12bb38b0338f2870
Status: Downloaded newer image for rancher/agent:v2.0.0-beta4
b20e674f692fa16b3a0d9aef52442804d0a7b30af6cd804c95290c3db77fc837
root@node153:~# echo $?
0
root@node153:~# docker ps -a 
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS                    PORTS               NAMES
a9824fdeeaed        rancher/agent:v2.0.0-beta4   "run.sh -- share-r..."   6 seconds ago       Up 5 seconds                                  share-mnt
b20e674f692f        rancher/agent:v2.0.0-beta4   "run.sh --server h..."   9 seconds ago       Up 8 seconds                                  wizardly_ardinghelli
dfaf2be1374d        hello-world                  "/hello"                 14 hours ago        Exited (0) 14 hours ago                       angry_montalcini
root@node153:~# 
~~~

再看 rancher 的管理界面

![rancher](/assets/img/rancher/rancher11.png)

就多出了一个节点


---

# 总结

因为是使用的 Docker 安装

所以相对容易

* TOC
{:toc}

---

[rancher]:https://rancher.com/
[rancher_guide]:https://rancher.com/docs/rancher/v2.0/en/quick-start-guide/
[rancher_requirements]:https://rancher.com/docs/rancher/v2.0/en/quick-start-guide/#host-and-node-requirements
[docker_install]:https://docs.docker.com/install/linux/docker-ce/ubuntu/