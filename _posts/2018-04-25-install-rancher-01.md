---
layout: post
title: "Install Rancher 1"
author:  wilmosfang
date: 2018-04-25 17:26:34
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
introduction: 'Installation of Docker'
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

这里演示一下如何构建 **[Rancher][rancher]**

参考 **[Quick Start Guide][rancher_guide]**

> **Tip:** 当前的版本为 **RANCHER 2.0 bata**

---

# 操作

## 依赖

### 硬件需求

* 内存: 4GB

### 软件需求

* 操作系统: Ubuntu 16.04 (64-bit)
* 软件: Docker (1.12.6|1.13.1|17.03.2)

>**Note:** 所有的节点上都要安装 Docker

### 端口需求

在不同的节点角色上，为了让调用信息通行，相应的端口必须开放

管理节点 (ETCD AND CONTROLPLANE NODES)

Protocol |Direction|Port Range|Purpose
---|---|---|---|---
TCP|Inbound|22	   |SSH server
TCP|Inbound|80	   |Canal
TCP|Inbound|443	   |Canal
TCP|Inbound|6443   |Kubernetes API server
TCP|Inbound|2379-2380	|etcd server client API
TCP|Inbound|10250	|kubelet API
TCP|Inbound|10251	|scheduler
TCP|Inbound|10252	|controller
TCP|Inbound|10256	|kubeproxy

工作节点

Protocol |Direction|Port Range|Purpose
---|---|---|---|---
TCP|Inbound|22	|SSH Server
TCP|Inbound|80	|Canal
TCP|Inbound|443	|Canal
TCP|Inbound|10250	|kubelet API
TCP|Inbound|10256	|kubeproxy
TCP|Inbound|30000-32767	|NodePort Services

>**Tip:** 关于安装前的依赖可以参考 **[HOST AND NODE REQUIREMENTS][rancher_requirements]**

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

## 安装 Docker

安装 Ubuntu 版本的 Docker 可以参考

**[Get Docker CE for Ubuntu][docker_install]**

### 系统要求

* Artful 17.10 (Docker CE 17.11 Edge and higher only)
* Xenial 16.04 (LTS)
* Trusty 14.04 (LTS)

OS 我这边已经满足了

### 删除旧版本 Docker

~~~
bolo@rancher:~$ sudo apt-get remove docker docker-engine docker.io
sudo: unable to resolve host rancher
[sudo] password for bolo: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package 'docker-engine' is not installed, so not removed
Package 'docker' is not installed, so not removed
Package 'docker.io' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
bolo@rancher:~$ 
~~~

### aufs

从 Ubuntu 16.04 开始， Linux 内核支持 OverlayFS

如果要使用 aufs 就得另外手动配置

>For Ubuntu 16.04 and higher, the Linux kernel includes support for OverlayFS, and Docker CE uses the overlay2 storage driver by default. If you need to use aufs instead, you need to configure it manually. See aufs

这里我先不配，后期优化的过程，再来研究与配置 aufs

### 安装 Docker 仓库

* 更新仓库
* 允许 apt 使用 https 仓库
* 添加 GPG key
* 检查确认 GPG key
* 安装稳定版 Docker 仓库

~~~
root@rancher:~# sudo apt-get update
sudo: unable to resolve host rancher
Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [107 kB]   
Hit:2 http://us.archive.ubuntu.com/ubuntu xenial InRelease                   
Get:3 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]    
Get:4 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]
Fetched 323 kB in 2s (132 kB/s)    
Reading package lists... Done
root@rancher:~# sudo apt-get install \
>     apt-transport-https \
>     ca-certificates \
>     curl \
>     software-properties-common
sudo: unable to resolve host rancher
Reading package lists... Done
Building dependency tree       
Reading state information... Done
apt-transport-https is already the newest version (1.2.26).
ca-certificates is already the newest version (20170717~16.04.1).
curl is already the newest version (7.47.0-1ubuntu2.7).
software-properties-common is already the newest version (0.96.20.7).
0 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
root@rancher:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo: unable to resolve host rancher
OK
root@rancher:~# echo $?
0
root@rancher:~# sudo apt-key fingerprint 0EBFCD88
sudo: unable to resolve host rancher
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

root@rancher:~# lsb_release -cs
xenial
root@rancher:~# sudo add-apt-repository \
>    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
>    $(lsb_release -cs) \
>    stable
> ^C
root@rancher:~# sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo: unable to resolve host rancher
root@rancher:~# echo $?
0
root@rancher:~# 
~~~

>**Tip:** 报 **`sudo: unable to resolve host rancher`** 是 hosts 文件中没有加入 rancher 的解析，加到 127.0.0.1 这一行就不会报了

### 安装 Docker ce

先更新仓库

~~~
root@rancher:~# sudo apt-get update
sudo: unable to resolve host rancher
Get:1 https://download.docker.com/linux/ubuntu xenial InRelease [65.8 kB]   
Hit:2 http://us.archive.ubuntu.com/ubuntu xenial InRelease                     
Hit:3 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease          
Get:4 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages [3,539 B]
Hit:5 http://security.ubuntu.com/ubuntu xenial-security InRelease              
Hit:6 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease    
Fetched 69.3 kB in 1s (49.8 kB/s)
Reading package lists... Done
root@rancher:~# 
root@rancher:~# vim /etc/hosts
root@rancher:~# grep ran /etc/hosts
127.0.1.1	U16X64Base rancher
root@rancher:~# 
root@rancher:~# ping rancher
PING U16X64Base (127.0.1.1) 56(84) bytes of data.
64 bytes from U16X64Base (127.0.1.1): icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from U16X64Base (127.0.1.1): icmp_seq=2 ttl=64 time=0.045 ms
^C
--- U16X64Base ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.041/0.043/0.045/0.002 ms
root@rancher:~# 
root@rancher:~# sudo apt-get update
Hit:1 http://security.ubuntu.com/ubuntu xenial-security InRelease
Hit:2 https://download.docker.com/linux/ubuntu xenial InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu xenial InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease
Hit:5 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease
Reading package lists... Done
root@rancher:~# 
~~~

安装 docker ce

~~~
root@rancher:~# sudo apt-get install docker-ce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  aufs-tools cgroupfs-mount libltdl7 pigz
Suggested packages:
  mountall
The following NEW packages will be installed:
  aufs-tools cgroupfs-mount docker-ce libltdl7 pigz
0 upgraded, 5 newly installed, 0 to remove and 3 not upgraded.
Need to get 92.9 kB/34.1 MB of archives.
After this operation, 182 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://us.archive.ubuntu.com/ubuntu xenial/universe amd64 aufs-tools amd64 1:3.2+20130722-1.1ubuntu1 [92.9 kB]
Fetched 92.9 kB in 1s (65.2 kB/s)     
Selecting previously unselected package pigz.
(Reading database ... 60032 files and directories currently installed.)
Preparing to unpack .../pigz_2.3.1-2_amd64.deb ...
Unpacking pigz (2.3.1-2) ...
Selecting previously unselected package aufs-tools.
Preparing to unpack .../aufs-tools_1%3a3.2+20130722-1.1ubuntu1_amd64.deb ...
Unpacking aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../cgroupfs-mount_1.2_all.deb ...
Unpacking cgroupfs-mount (1.2) ...
Selecting previously unselected package libltdl7:amd64.
Preparing to unpack .../libltdl7_2.4.6-0.1_amd64.deb ...
Unpacking libltdl7:amd64 (2.4.6-0.1) ...
Selecting previously unselected package docker-ce.
Preparing to unpack .../docker-ce_18.03.0~ce-0~ubuntu_amd64.deb ...
Unpacking docker-ce (18.03.0~ce-0~ubuntu) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Setting up pigz (2.3.1-2) ...
Setting up aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Setting up cgroupfs-mount (1.2) ...
Setting up libltdl7:amd64 (2.4.6-0.1) ...
Setting up docker-ce (18.03.0~ce-0~ubuntu) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Processing triggers for ureadahead (0.100.0-19) ...
root@rancher:~# echo $?
0
root@rancher:~# 
~~~

结果默认安装成了最新版本

~~~
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           18.03.0~ce-0~ubuntu                        amd64        Docker: the open-source application container engine
root@rancher:~# 
~~~

因为 rancher 文档里明确规定了 Docker 的版本为 **`17.03.2`**，所以我得降级版本

~~~
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           18.03.0~ce-0~ubuntu                        amd64        Docker: the open-source application container engine
root@rancher:~# 
root@rancher:~# sudo apt-get remove docker docker-engine docker.io docker-ce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package 'docker-engine' is not installed, so not removed
Package 'docker' is not installed, so not removed
Package 'docker.io' is not installed, so not removed
The following packages were automatically installed and are no longer required:
  aufs-tools cgroupfs-mount libltdl7 pigz
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  docker-ce
0 upgraded, 0 newly installed, 1 to remove and 3 not upgraded.
After this operation, 181 MB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 60327 files and directories currently installed.)
Removing docker-ce (18.03.0~ce-0~ubuntu) ...
Processing triggers for man-db (2.7.5-1) ...
root@rancher:~# echo $?
0
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
rc  docker-ce                           18.03.0~ce-0~ubuntu                        amd64        Docker: the open-source application container engine
root@rancher:~# 
root@rancher:~# apt-cache madison docker-ce
 docker-ce | 18.03.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.12.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.09.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.2~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.2~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
root@rancher:~# 
root@rancher:~# sudo apt-get install docker-ce=17.03.2~ce-0~ubuntu-xenial
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  pigz
Use 'sudo apt autoremove' to remove it.
The following NEW packages will be installed:
  docker-ce
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 19.2 MB of archives.
After this operation, 89.0 MB of additional disk space will be used.
Get:1 https://download.docker.com/linux/ubuntu xenial/stable amd64 docker-ce amd64 17.03.2~ce-0~ubuntu-xenial [19.2 MB]
Fetched 19.2 MB in 12s (1,557 kB/s)                                            
Selecting previously unselected package docker-ce.
(Reading database ... 60112 files and directories currently installed.)
Preparing to unpack .../docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb ...
Unpacking docker-ce (17.03.2~ce-0~ubuntu-xenial) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Processing triggers for ureadahead (0.100.0-19) ...
Setting up docker-ce (17.03.2~ce-0~ubuntu-xenial) ...
Installing new version of config file /etc/init.d/docker ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Processing triggers for ureadahead (0.100.0-19) ...
root@rancher:~# echo $?
0
root@rancher:~# 
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           17.03.2~ce-0~ubuntu-xenial                 amd64        Docker: the open-source application container engine
root@rancher:~# 
~~~

### 检查 Docker 安装

~~~
root@rancher:~# ps faux | grep -i docker
root      8292  0.0  0.0  14224   924 pts/0    S+   11:30   0:00  |                       \_ grep --color=auto -i docker
root      8136  0.2  1.0 432756 40604 ?        Ssl  11:23   0:01 /usr/bin/dockerd -H fd://
root      8146  0.1  0.3 292088 13488 ?        Ssl  11:23   0:00  \_ docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
root@rancher:~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete 
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

root@rancher:~# echo $?
0
root@rancher:~# 
~~~

这就代表 Docker 已经正常安装了

这个小镜像能成功执行就代表完成了以下几点检查

* Docker 客户端可以正常连接 Docker 守护进程
* Docker 守护进程可以从 Docker Hub 中下拉镜像
* Docker 守护进程可以根据镜像在本地创建容器
* 容器可以正常运行执行指定逻辑
* Docker 守护进程可以给 Docker 客户端传递信息并且在本地的终端显示

---

# 总结

因为是使用的 apt-get 加上仓库的方式进行 Docker 安装

所以相应容易

只需要保证网络可达，指定自确的版本，其它就都很简单

基于 Docker 后面再进行 Rancher 的安装

* TOC
{:toc}

---

[rancher]:https://rancher.com/
[rancher_guide]:https://rancher.com/docs/rancher/v2.0/en/quick-start-guide/
[rancher_requirements]:https://rancher.com/docs/rancher/v2.0/en/quick-start-guide/#host-and-node-requirements
[docker_install]:https://docs.docker.com/install/linux/docker-ce/ubuntu/