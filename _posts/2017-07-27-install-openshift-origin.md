---
layout:  post
title:  安装 OpenShift Origin
author:  wilmosfang
tags:  openshift docker 
categories:  openshift 
wc: 1169 9588 107307
excerpt:  在 Centos7 中安装 openshift-origin
comments: true
---


# 前言

这两年容器技术太火了，很多大公司后台的应用都完成了容器化的转变，容器化代表了 DevOps 领域的一个未来趋势

就像 kvm 的虚拟化有一个 openstack 的开源 iaas 与之相对应一样，docker 的容器技术也有一个 openshift 的开源 paas 与之相对应

## 什么是 openshift

**[openshift][openshift]** 是一个开源的容器应用平台


![openshift_architecture.png](/images/openshift/openshift_architecture.png)

## 什么是 openshift origin

**[openshift origin][openshift_origin]** 是用来支持 **[openshift][openshift]** 产品的一个上游社区项目，围绕 Docker 容器和 Kubernetes 集群技术，一套来进行应用生命周期管理的 DevOps 工具，它提供了一个完整的开源容器应用平台

>Origin is the upstream community project that powers OpenShift. Built around a core of Docker container packaging and Kubernetes container cluster management, Origin is also augmented by application lifecycle management functionality and DevOps tooling. Origin provides a complete open source container application platform


![openshift_relationship.jpg](/images/openshift/openshift_relationship.jpg)


## DevOps

现在流行的 DevOps 文化就是在当前机器越来越便宜而人力相对而言越来越贵的大前提下，可以节约人力，就尽量使用机器的一种现象 

(伴随的结果就是生产效率越来越高)

个人看来，人类从未停止过最大化使用有限资源的步伐

针对 IT 领域，有限的资源就指硬件资源和人力资源

(硬件资源又可以粗略的划分为计算，存储和网络资源)

硬件资源的池化，诞生了虚拟化技术，进一步地提升了以有硬件的使用效率，节约了资源

虚拟化技术虽然巧妙，但不免有些厚重，放弃深层隔离主动使用更浅的隔离再加上系统层面的资源约束可以更为充分地榨取以有硬件的资源，于是在自然地实践中诞生了容器技术　

>容器技术并不专指 Docker ，容器技术有好几种(LXC libcontainer，都是基于 linux 中 namespace 和 cgroup 的内核特性)，Docker 只是其中一种打包方案，Docker 的意义在于制定了一系列标准，一系列统一接口，从而让容器技术更简单地流行起来

DevOps 过程中最常见的系统就是 CI/CD

CI/CD(持续集成/持续交付) 系统因为有了容器技术一切都变得简单和高效起来，结合前面讲的 **[GitLab][gitlab]** ，就可以逐步构建出一个 DevOps 生态链


## SDX

与此同时，上面这个演进脉络中可以隐约看出一个 SDX 的路径

网络资源的虚拟化叫 SDN (软件定义网络)

服务器硬件资源的虚拟华可以叫 SDC (软件定义计算机)

云平台的资源虚拟化可以叫 SDI (软件定义基础架构，包括了 SDN)

软件运行环境的虚拟化可以叫 SDE (软件定义环境，也就是容器技术)

这个 **SDX** 中的 **X** 代表了 **everything** ，软件定义一切

因为软件定义了一切，于是可以更加标准和简洁地被计算机接受和处理，于是可以解放人类的有限而昂贵的劳动力，同时更加充分地榨取现有计算资源

在这里简单地实现一下 **[openshift origin][openshift_origin]** 的部署

细节的展开，将在后面的文章中慢慢展开

> **Tip:**  当前最新版本为 **OpenShift Origin 1.5**

详细信息可以参考 **[openshift origin 的官方文档][openshift_doc]** ，还可以跟进  **[openshift origin GitHub 项目][openshift_github]**


---

# 概要

* TOC
{:toc}


---


## 系统环境


~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 17c7c182aeda49aa94a14491067ca767
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:04:c7:5a brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85616sec preferred_lft 85616sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:b5:a5:da brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.206/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb5:a5da/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]# uname  -a
Linux much 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@much ~]# 
~~~

---


## 查看系统的包信息


**openshift-origin15** 是当前最新的版本

~~~
[root@much ~]# rpm -qa | grep openshift
[root@much ~]# rpm -qa | grep docker 
[root@much ~]# yum  list all | grep openshift
centos-release-openshift-origin.noarch     1-1.el7.centos              extras   
centos-release-openshift-origin13.noarch   1-1.el7.centos              extras   
centos-release-openshift-origin14.noarch   1-1.el7.centos              extras   
centos-release-openshift-origin15.noarch   1-1.el7.centos              extras   
[root@much ~]# yum  list all | grep docker 
cockpit-docker.x86_64                      141-3.el7.centos            extras   
docker.x86_64                              2:1.12.6-32.git88a4867.el7.centos
docker-client.x86_64                       2:1.12.6-32.git88a4867.el7.centos
docker-client-latest.x86_64                1.13.1-13.gitb303bf6.el7.centos
docker-common.x86_64                       2:1.12.6-32.git88a4867.el7.centos
docker-devel.x86_64                        1.3.2-4.el7.centos          extras   
docker-distribution.x86_64                 2.6.1-1.el7                 extras   
docker-forward-journald.x86_64             1.10.3-44.el7.centos        extras   
docker-latest.x86_64                       1.13.1-13.gitb303bf6.el7.centos
docker-latest-logrotate.x86_64             1.13.1-13.gitb303bf6.el7.centos
docker-latest-v1.10-migrator.x86_64        1.13.1-13.gitb303bf6.el7.centos
docker-logrotate.x86_64                    2:1.12.6-32.git88a4867.el7.centos
docker-lvm-plugin.x86_64                   2:1.12.6-32.git88a4867.el7.centos
docker-novolume-plugin.x86_64              2:1.12.6-32.git88a4867.el7.centos
docker-python.x86_64                       1.4.0-115.el7               extras   
docker-registry.noarch                     0.6.8-8.el7                 extras   
docker-registry.x86_64                     0.9.1-7.el7                 extras   
docker-unit-test.x86_64                    2:1.12.6-32.git88a4867.el7.centos
docker-v1.10-migrator.x86_64               2:1.12.6-32.git88a4867.el7.centos
python-docker-py.noarch                    1.10.6-1.el7                extras   
python-docker-pycreds.noarch               1.10.6-1.el7                extras   
[root@much ~]# yum info centos-release-openshift-origin15.noarch 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.cn99.com
 * c7-media: 
 * extras: mirrors.cn99.com
 * updates: mirrors.cn99.com
Available Packages
Name        : centos-release-openshift-origin15
Arch        : noarch
Version     : 1
Release     : 1.el7.centos
Size        : 11 k
Repo        : extras/7/x86_64
Summary     : Yum configuration for OpenShift Origin 1.5 packages
URL         : https://wiki.centos.org/SpecialInterestGroup/PaaS/OpenShift
License     : GPLv2
Description : yum configuration for OpenShift Origin 1.5 packages as delivered via the
            : CentOS PaaS SIG.

[root@much ~]# 
~~~



---

## 安装 openshift-origin15 库

~~~
[root@much ~]# yum install centos-release-openshift-origin15.noarch
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.cn99.com
 * c7-media: 
 * extras: mirrors.cn99.com
 * updates: mirrors.cn99.com
Resolving Dependencies
--> Running transaction check
---> Package centos-release-openshift-origin15.noarch 0:1-1.el7.centos will be installed
--> Processing Dependency: centos-release-paas-common for package: centos-release-openshift-origin15-1-1.el7.centos.noarch
--> Running transaction check
---> Package centos-release-paas-common.noarch 0:1-1.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================================================
 Package                                                        Arch                                Version                                       Repository                           Size
============================================================================================================================================================================================
Installing:
 centos-release-openshift-origin15                              noarch                              1-1.el7.centos                                extras                               11 k
Installing for dependencies:
 centos-release-paas-common                                     noarch                              1-1.el7.centos                                extras                               11 k

Transaction Summary
============================================================================================================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 22 k
Installed size: 37 k
Is this ok [y/d/N]: y
Downloading packages:
(1/2): centos-release-openshift-origin15-1-1.el7.centos.noarch.rpm                                                                                                   |  11 kB  00:00:00     
(2/2): centos-release-paas-common-1-1.el7.centos.noarch.rpm                                                                                                          |  11 kB  00:00:00     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                        53 kB/s |  22 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : centos-release-paas-common-1-1.el7.centos.noarch                                                                                                                         1/2 
  Installing : centos-release-openshift-origin15-1-1.el7.centos.noarch                                                                                                                  2/2 
  Verifying  : centos-release-paas-common-1-1.el7.centos.noarch                                                                                                                         1/2 
  Verifying  : centos-release-openshift-origin15-1-1.el7.centos.noarch                                                                                                                  2/2 

Installed:
  centos-release-openshift-origin15.noarch 0:1-1.el7.centos                                                                                                                                 

Dependency Installed:
  centos-release-paas-common.noarch 0:1-1.el7.centos                                                                                                                                        

Complete!
[root@much ~]# echo $?
0
[root@much ~]# rpm -ql centos-release-openshift-origin15
/etc/yum.repos.d/CentOS-OpenShift-Origin15.repo
/usr/share/licenses/centos-release-openshift-origin15-1
/usr/share/licenses/centos-release-openshift-origin15-1/LICENSE
[root@much ~]# ll /etc/yum.repos.d/
total 32
-rw-r--r--. 1 root root 1664 11月 30 2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 2016 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 2016 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 2016 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 6月  16 01:08 CentOS-Media.repo
-rw-r--r--. 1 root root  884 5月  25 02:37 CentOS-OpenShift-Origin15.repo
-rw-r--r--. 1 root root 1331 11月 30 2016 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 2016 CentOS-Vault.repo
[root@much ~]# ll /etc/yum.repos.d/ | grep  OpenShift
-rw-r--r--. 1 root root  884 5月  25 02:37 CentOS-OpenShift-Origin15.repo
[root@much ~]# 
~~~

**CentOS-OpenShift-Origin15.repo** 就是安装后产生的库配置文件

---

## 安装 origin

~~~
[root@much ~]# yum install origin
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: mirrors.btte.net
Resolving Dependencies
--> Running transaction check
---> Package origin.x86_64 0:1.5.1-1.el7 will be installed
--> Processing Dependency: origin-clients = 1.5.1-1.el7 for package: origin-1.5.1-1.el7.x86_64
--> Running transaction check
---> Package origin-clients.x86_64 0:1.5.1-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================================================
 Package                                      Arch                                 Version                                    Repository                                               Size
============================================================================================================================================================================================
Installing:
 origin                                       x86_64                               1.5.1-1.el7                                centos-openshift-origin15                                35 M
Installing for dependencies:
 origin-clients                               x86_64                               1.5.1-1.el7                                centos-openshift-origin15                                16 M

Transaction Summary
============================================================================================================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 51 M
Installed size: 334 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/centos-openshift-origin15/packages/origin-clients-1.5.1-1.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID 2f297ecc: NOKEY MB/s |  36 MB  00:00:12 ETA 
Public key for origin-clients-1.5.1-1.el7.x86_64.rpm is not installed
(1/2): origin-clients-1.5.1-1.el7.x86_64.rpm                                                                                                                         |  16 MB  00:00:29     
(2/2): origin-1.5.1-1.el7.x86_64.rpm                                                                                                                                 |  35 MB  00:00:43     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                       1.2 MB/s |  51 MB  00:00:43     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-PaaS
Importing GPG key 0x2F297ECC:
 Userid     : "CentOS PaaS SIG (https://wiki.centos.org/SpecialInterestGroup/PaaS) <security@centos.org>"
 Fingerprint: c5e8 ab44 6fa7 893d 7490 51f1 c34c 5bd4 2f29 7ecc
 Package    : centos-release-paas-common-1-1.el7.centos.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-PaaS
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : origin-clients-1.5.1-1.el7.x86_64                                                                                                                                        1/2 
  Installing : origin-1.5.1-1.el7.x86_64                                                                                                                                                2/2 
  Verifying  : origin-1.5.1-1.el7.x86_64                                                                                                                                                1/2 
  Verifying  : origin-clients-1.5.1-1.el7.x86_64                                                                                                                                        2/2 

Installed:
  origin.x86_64 0:1.5.1-1.el7                                                                                                                                                               

Dependency Installed:
  origin-clients.x86_64 0:1.5.1-1.el7                                                                                                                                                       

Complete!
[root@much ~]# echo $?
0
[root@much ~]#
~~~

---

## 安装 docker

~~~
[root@much ~]# yum install docker
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: mirrors.btte.net
Resolving Dependencies
--> Running transaction check
---> Package docker.x86_64 2:1.12.6-32.git88a4867.el7.centos will be installed
--> Processing Dependency: docker-common = 2:1.12.6-32.git88a4867.el7.centos for package: 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64
--> Processing Dependency: docker-client = 2:1.12.6-32.git88a4867.el7.centos for package: 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64
--> Processing Dependency: oci-systemd-hook >= 1:0.1.4-9 for package: 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64
--> Processing Dependency: oci-register-machine >= 1:0-3.10 for package: 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64
--> Processing Dependency: container-selinux >= 2:2.12-2 for package: 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64
--> Processing Dependency: skopeo-containers for package: 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.19-2.1.el7 will be installed
---> Package docker-client.x86_64 2:1.12.6-32.git88a4867.el7.centos will be installed
---> Package docker-common.x86_64 2:1.12.6-32.git88a4867.el7.centos will be installed
---> Package oci-register-machine.x86_64 1:0-3.11.gitdd0daef.el7 will be installed
---> Package oci-systemd-hook.x86_64 1:0.1.7-4.gite533efa.el7 will be installed
---> Package skopeo-containers.x86_64 1:0.1.20-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================================================
 Package                                          Arch                               Version                                                       Repository                          Size
============================================================================================================================================================================================
Installing:
 docker                                           x86_64                             2:1.12.6-32.git88a4867.el7.centos                             extras                              14 M
Installing for dependencies:
 container-selinux                                noarch                             2:2.19-2.1.el7                                                extras                              28 k
 docker-client                                    x86_64                             2:1.12.6-32.git88a4867.el7.centos                             extras                             3.2 M
 docker-common                                    x86_64                             2:1.12.6-32.git88a4867.el7.centos                             extras                              77 k
 oci-register-machine                             x86_64                             1:0-3.11.gitdd0daef.el7                                       extras                             1.0 M
 oci-systemd-hook                                 x86_64                             1:0.1.7-4.gite533efa.el7                                      extras                              30 k
 skopeo-containers                                x86_64                             1:0.1.20-2.el7                                                extras                             7.8 k

Transaction Summary
============================================================================================================================================================================================
Install  1 Package (+6 Dependent packages)

Total download size: 19 M
Installed size: 64 M
Is this ok [y/d/N]: y
Downloading packages:
(1/7): container-selinux-2.19-2.1.el7.noarch.rpm                                                                                                                     |  28 kB  00:00:00     
(2/7): docker-common-1.12.6-32.git88a4867.el7.centos.x86_64.rpm                                                                                                      |  77 kB  00:00:00     
(3/7): docker-client-1.12.6-32.git88a4867.el7.centos.x86_64.rpm                                                                                                      | 3.2 MB  00:00:02     
(4/7): oci-register-machine-0-3.11.gitdd0daef.el7.x86_64.rpm                                                                                                         | 1.0 MB  00:00:04     
(5/7): skopeo-containers-0.1.20-2.el7.x86_64.rpm                                                                                                                     | 7.8 kB  00:00:00     
(6/7): oci-systemd-hook-0.1.7-4.gite533efa.el7.x86_64.rpm                                                                                                            |  30 kB  00:00:02     
(7/7): docker-1.12.6-32.git88a4867.el7.centos.x86_64.rpm                                                                                                             |  14 MB  00:00:15     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                       1.2 MB/s |  19 MB  00:00:15     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 2:docker-common-1.12.6-32.git88a4867.el7.centos.x86_64                                                                                                                   1/7 
  Installing : 2:docker-client-1.12.6-32.git88a4867.el7.centos.x86_64                                                                                                                   2/7 
  Installing : 1:skopeo-containers-0.1.20-2.el7.x86_64                                                                                                                                  3/7 
  Installing : 1:oci-register-machine-0-3.11.gitdd0daef.el7.x86_64                                                                                                                      4/7 
  Installing : 2:container-selinux-2.19-2.1.el7.noarch                                                                                                                                  5/7 
  Installing : 1:oci-systemd-hook-0.1.7-4.gite533efa.el7.x86_64                                                                                                                         6/7 
Stopping containers...
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
"docker stop" requires at least 1 argument(s).
See 'docker stop --help'.

Usage:  docker stop [OPTIONS] CONTAINER [CONTAINER...]

Stop one or more running containers
  Installing : 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64                                                                                                                          7/7 
  Verifying  : 2:docker-1.12.6-32.git88a4867.el7.centos.x86_64                                                                                                                          1/7 
  Verifying  : 1:oci-systemd-hook-0.1.7-4.gite533efa.el7.x86_64                                                                                                                         2/7 
  Verifying  : 2:container-selinux-2.19-2.1.el7.noarch                                                                                                                                  3/7 
  Verifying  : 2:docker-common-1.12.6-32.git88a4867.el7.centos.x86_64                                                                                                                   4/7 
  Verifying  : 1:oci-register-machine-0-3.11.gitdd0daef.el7.x86_64                                                                                                                      5/7 
  Verifying  : 1:skopeo-containers-0.1.20-2.el7.x86_64                                                                                                                                  6/7 
  Verifying  : 2:docker-client-1.12.6-32.git88a4867.el7.centos.x86_64                                                                                                                   7/7 

Installed:
  docker.x86_64 2:1.12.6-32.git88a4867.el7.centos                                                                                                                                           

Dependency Installed:
  container-selinux.noarch 2:2.19-2.1.el7                     docker-client.x86_64 2:1.12.6-32.git88a4867.el7.centos         docker-common.x86_64 2:1.12.6-32.git88a4867.el7.centos        
  oci-register-machine.x86_64 1:0-3.11.gitdd0daef.el7         oci-systemd-hook.x86_64 1:0.1.7-4.gite533efa.el7               skopeo-containers.x86_64 1:0.1.20-2.el7                       

Complete!
[root@much ~]# echo $?
0
[root@much ~]# 
~~~

openshift 需要 docker 的支持，如果缺少 docker 会有如下报错

~~~
[root@much ~]# openshift start 
W0726 16:37:37.669871    3199 start_master.go:291] Warning: assetConfig.loggingPublicURL: Invalid value: "": required to view aggregated container logs in the console, master start will continue.
W0726 16:37:37.670075    3199 start_master.go:291] Warning: assetConfig.metricsPublicURL: Invalid value: "": required to view cluster metrics in the console, master start will continue.
W0726 16:37:37.670083    3199 start_master.go:291] Warning: auditConfig.auditFilePath: Required value: audit can now be logged to a separate file, master start will continue.
I0726 16:37:37.673799    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:37.674572    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.674664 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:37:37.674822 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.675163    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.675421 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.675716    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.676052 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.676339    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.676651 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
E0726 16:37:37.679630    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/resourcequota/resource_access.go:83: Failed to list *api.ResourceQuota: Get https://10.0.2.15:8443/api/v1/resourcequotas?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:37:37.679716    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:119: Failed to list *api.Secret: Get https://10.0.2.15:8443/api/v1/secrets?fieldSelector=type%3Dkubernetes.io%2Fservice-account-token&resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
I0726 16:37:37.679752    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:37:37.679761    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/storageclass/default/admission.go:75: Failed to list *storage.StorageClass: Get https://10.0.2.15:8443/apis/storage.k8s.io/v1beta1/storageclasses?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:37:37.679829    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:103: Failed to list *api.ServiceAccount: Get https://10.0.2.15:8443/api/v1/serviceaccounts?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
2017-07-26 16:37:37.679950 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.680427    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.680699 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.681076    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:37.681771    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.681824 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:37:37.681964 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.682115    3199 plugins.go:94] No cloud provider specified.
I0726 16:37:37.685328    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.685880 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.685943    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:37.686011    3199 start_master.go:410] Starting master on 0.0.0.0:8443 (v1.5.1)
I0726 16:37:37.686018    3199 start_master.go:411] Public master address is https://10.0.2.15:8443
I0726 16:37:37.686048    3199 start_master.go:415] Using images from "openshift/origin-<component>:v1.5.1"
2017-07-26 16:37:37.686123 I | embed: peerTLS: cert = openshift.local.config/master/etcd.server.crt, key = openshift.local.config/master/etcd.server.key, ca = openshift.local.config/master/ca.crt, trusted-ca = , client-cert-auth = true
2017-07-26 16:37:37.686984 I | embed: listening for peers on https://0.0.0.0:7001
2017-07-26 16:37:37.687040 I | embed: listening for client requests on 0.0.0.0:4001
2017-07-26 16:37:37.688239 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:37:37.697538    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.697631 I | etcdserver: name = openshift.local
2017-07-26 16:37:37.697640 I | etcdserver: data dir = openshift.local.etcd
2017-07-26 16:37:37.697646 I | etcdserver: member dir = openshift.local.etcd/member
2017-07-26 16:37:37.697649 I | etcdserver: heartbeat = 100ms
2017-07-26 16:37:37.697653 I | etcdserver: election = 1000ms
2017-07-26 16:37:37.697657 I | etcdserver: snapshot count = 10000
2017-07-26 16:37:37.697666 I | etcdserver: advertise client URLs = https://10.0.2.15:4001
2017-07-26 16:37:37.697670 I | etcdserver: initial advertise peer URLs = https://10.0.2.15:7001
2017-07-26 16:37:37.697678 I | etcdserver: initial cluster = openshift.local=https://10.0.2.15:7001
2017-07-26 16:37:37.699961 I | etcdserver: starting member abca2d8f0dd66377 in cluster 28cb2054c808baa3
2017-07-26 16:37:37.700000 I | raft: abca2d8f0dd66377 became follower at term 0
2017-07-26 16:37:37.700012 I | raft: newRaft abca2d8f0dd66377 [peers: [], term: 0, commit: 0, applied: 0, lastindex: 0, lastterm: 0]
2017-07-26 16:37:37.700017 I | raft: abca2d8f0dd66377 became follower at term 1
I0726 16:37:37.706089    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:37.706979    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:37.707024 I | etcdserver: starting server... [version: 3.1.0, cluster version: to_be_decided]
2017-07-26 16:37:37.707063 I | embed: ClientTLS: cert = openshift.local.config/master/etcd.server.crt, key = openshift.local.config/master/etcd.server.key, ca = openshift.local.config/master/ca.crt, trusted-ca = , client-cert-auth = true
2017-07-26 16:37:37.709035 I | etcdserver/membership: added member abca2d8f0dd66377 [https://10.0.2.15:7001] to cluster 28cb2054c808baa3
2017-07-26 16:37:38.002535 I | raft: abca2d8f0dd66377 is starting a new election at term 1
2017-07-26 16:37:38.002683 I | raft: abca2d8f0dd66377 became candidate at term 2
2017-07-26 16:37:38.002740 I | raft: abca2d8f0dd66377 received MsgVoteResp from abca2d8f0dd66377 at term 2
2017-07-26 16:37:38.002788 I | raft: abca2d8f0dd66377 became leader at term 2
2017-07-26 16:37:38.002817 I | raft: raft.node: abca2d8f0dd66377 elected leader abca2d8f0dd66377 at term 2
2017-07-26 16:37:38.004306 I | etcdserver: setting up the initial cluster version to 3.1
2017-07-26 16:37:38.005817 N | etcdserver/membership: set the initial cluster version to 3.1
2017-07-26 16:37:38.005922 I | etcdserver/api: enabled capabilities for version 3.1
2017-07-26 16:37:38.006153 I | etcdserver: published {Name:openshift.local ClientURLs:[https://10.0.2.15:4001]} to cluster 28cb2054c808baa3
2017-07-26 16:37:38.006185 I | embed: ready to serve client requests
I0726 16:37:38.006396    3199 run.go:77] Started etcd at 10.0.2.15:4001
2017-07-26 16:37:38.007052 I | embed: serving client requests on [::]:4001
2017-07-26 16:37:38.081593 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:37:38.082226 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:37:38.083322 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:37:38.083346 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:37:38.083364 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
I0726 16:37:38.083469    3199 run_components.go:229] Using default project node label selector: 
I0726 16:37:38.084278    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:37:38.084893 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
I0726 16:37:38.084914    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.085500    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:37:38.085918    3199 reflector.go:199] github.com/openshift/origin/pkg/controller/shared/shared_informer.go:101: Failed to list *api.ClusterResourceQuota: Get https://10.0.2.15:8443/oapi/v1/clusterresourcequotas?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
I0726 16:37:38.086075    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.086640    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.087215    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:37:38.087802    3199 reflector.go:188] github.com/openshift/origin/pkg/project/cache/cache.go:107: Failed to list *api.Namespace: Get https://10.0.2.15:8443/api/v1/namespaces?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
I0726 16:37:38.131526    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.133030    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.135342    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.136188    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.136991    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.137616    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.138205    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.138848    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.139466    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.140196    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.140968    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.141635    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.142915    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.143523    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.144312    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.145109    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.145702    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.146242    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.167770    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.168436    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.169066    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.169626    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.170214    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.170853    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.171428    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.172012    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.172547    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.173266    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.173931    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.174596    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.175623    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.176403    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.177067    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.177690    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.178375    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.179083    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.211275    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.211955    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.212600    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.231420    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.253507    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.254247    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.255287    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.256020    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.256827    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.257566    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.258262    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.259370    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.260097    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.260752    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.261323    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.262034    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.262928    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.263631    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.264211    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.264887    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.265522    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.266320    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.266977    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.267590    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.268187    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.332750    3199 master.go:171] Started OAuth2 API at 0.0.0.0:8443/oauth
I0726 16:37:38.332769    3199 master.go:171] Started Web Console 0.0.0.0:8443/console/
I0726 16:37:38.332773    3199 master.go:171] Started Kubernetes API at 0.0.0.0:8443/api
I0726 16:37:38.332776    3199 master.go:171] Started Kubernetes API apps/v1beta1 at 0.0.0.0:8443/apis
I0726 16:37:38.332780    3199 master.go:171] Started Kubernetes API authentication.k8s.io/v1beta1 at 0.0.0.0:8443/apis
I0726 16:37:38.332783    3199 master.go:171] Started Kubernetes API autoscaling/v1 at 0.0.0.0:8443/apis
I0726 16:37:38.332786    3199 master.go:171] Started Kubernetes API batch/v1 at 0.0.0.0:8443/apis
I0726 16:37:38.332789    3199 master.go:171] Started Kubernetes API batch/v2alpha1 at 0.0.0.0:8443/apis
I0726 16:37:38.332792    3199 master.go:171] Started Kubernetes API certificates.k8s.io/v1alpha1 at 0.0.0.0:8443/apis
I0726 16:37:38.332795    3199 master.go:171] Started Kubernetes API extensions/v1beta1 at 0.0.0.0:8443/apis
I0726 16:37:38.332799    3199 master.go:171] Started Kubernetes API policy/v1beta1 at 0.0.0.0:8443/apis
I0726 16:37:38.332802    3199 master.go:171] Started Kubernetes API storage.k8s.io/v1beta1 at 0.0.0.0:8443/apis
I0726 16:37:38.332806    3199 master.go:171] Started Swagger Schema API at 0.0.0.0:8443/swaggerapi/
I0726 16:37:38.332809    3199 master.go:171] Started OpenAPI Schema at 0.0.0.0:8443/swagger.json
[restful] 2017/07/26 16:37:38 log.go:30: [restful/swagger] listing is available at https://10.0.2.15:8443/swaggerapi/
[restful] 2017/07/26 16:37:38 log.go:30: [restful/swagger] https://10.0.2.15:8443/swaggerui/ is mapped to folder /swagger-ui/
I0726 16:37:38.640829    3199 serve.go:95] Serving securely on 0.0.0.0:8443
W0726 16:37:38.652047    3199 storage_extensions.go:137] third party resource sync failed: User "system:openshift-master" cannot list all extensions.thirdpartyresources in the cluster
E0726 16:37:38.660622    3199 controller.go:136] Unable to perform initial Kubernetes service initialization: User "system:openshift-master" cannot create services in project "default"
E0726 16:37:38.665409    3199 controller.go:162] unable to sync kubernetes service: User "system:openshift-master" cannot create services in project "default"
E0726 16:37:38.724846    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:103: Failed to list *api.ServiceAccount: User "system:openshift-master" cannot list all serviceaccounts in the cluster
E0726 16:37:38.725135    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/resourcequota/resource_access.go:83: Failed to list *api.ResourceQuota: User "system:openshift-master" cannot list all resourcequotas in the cluster
E0726 16:37:38.725214    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:119: Failed to list *api.Secret: User "system:openshift-master" cannot list all secrets in the cluster
E0726 16:37:38.725294    3199 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/storageclass/default/admission.go:75: Failed to list *storage.StorageClass: User "system:openshift-master" cannot list all storage.k8s.io.storageclasses in the cluster
I0726 16:37:38.728274    3199 trace.go:61] Trace "cacher *api.ClusterPolicy: List" (started 2017-07-26 16:37:38.085732478 +0800 CST):
[642.473081ms] [642.473081ms] Ready
[642.49618ms] [23.099µs] watchCache locked acquired
[642.496305ms] [125ns] watchCache fresh enough
[642.49981ms] [3.505µs] Listed 0 items from cache
[642.500666ms] [856ns] Filtered 0 items
"cacher *api.ClusterPolicy: List" [642.505474ms] [4.808µs] END
I0726 16:37:38.730897    3199 trace.go:61] Trace "cacher *api.PolicyBinding: List" (started 2017-07-26 16:37:38.0856151 +0800 CST):
[645.228196ms] [645.228196ms] Ready
[645.249593ms] [21.397µs] watchCache locked acquired
[645.249709ms] [116ns] watchCache fresh enough
[645.253373ms] [3.664µs] Listed 0 items from cache
[645.25442ms] [1.047µs] Filtered 0 items
"cacher *api.PolicyBinding: List" [645.258589ms] [4.169µs] END
I0726 16:37:38.737805    3199 trace.go:61] Trace "cacher *api.Group: List" (started 2017-07-26 16:37:38.084934315 +0800 CST):
[652.792889ms] [652.792889ms] Ready
[652.819255ms] [26.366µs] watchCache locked acquired
[652.819395ms] [140ns] watchCache fresh enough
[652.822491ms] [3.096µs] Listed 0 items from cache
[652.823678ms] [1.187µs] Filtered 0 items
"cacher *api.Group: List" [652.829499ms] [5.821µs] END
I0726 16:37:38.738558    3199 trace.go:61] Trace "cacher *api.Policy: List" (started 2017-07-26 16:37:38.085495761 +0800 CST):
[652.996238ms] [652.996238ms] Ready
[653.032727ms] [36.489µs] watchCache locked acquired
[653.032829ms] [102ns] watchCache fresh enough
[653.035395ms] [2.566µs] Listed 0 items from cache
[653.037113ms] [1.718µs] Filtered 0 items
"cacher *api.Policy: List" [653.041003ms] [3.89µs] END
I0726 16:37:38.738666    3199 trace.go:61] Trace "cacher *api.ClusterPolicyBinding: List" (started 2017-07-26 16:37:38.085847326 +0800 CST):
[652.749642ms] [652.749642ms] Ready
[652.770128ms] [20.486µs] watchCache locked acquired
[652.770246ms] [118ns] watchCache fresh enough
[652.773104ms] [2.858µs] Listed 0 items from cache
[652.77378ms] [676ns] Filtered 0 items
"cacher *api.ClusterPolicyBinding: List" [652.777655ms] [3.875µs] END
I0726 16:37:38.751549    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:37:38.755357    3199 reflector.go:199] github.com/openshift/origin/pkg/controller/shared/shared_informer.go:89: Failed to list *api.SecurityContextConstraints: User "system:openshift-master" cannot list all securitycontextconstraints in the cluster
E0726 16:37:38.755483    3199 reflector.go:199] pkg/controller/informers/factory.go:89: Failed to list *api.LimitRange: User "system:openshift-master" cannot list all limitranges in the cluster
E0726 16:37:38.755697    3199 reflector.go:199] pkg/controller/informers/factory.go:89: Failed to list *api.Namespace: User "system:openshift-master" cannot list all namespaces in the cluster
E0726 16:37:38.755777    3199 reflector.go:199] pkg/controller/informers/factory.go:89: Failed to list *api.ServiceAccount: User "system:openshift-master" cannot list all serviceaccounts in the cluster
E0726 16:37:38.755928    3199 reflector.go:199] github.com/openshift/origin/pkg/controller/shared/shared_informer.go:89: Failed to list *api.ImageStream: User "system:openshift-master" cannot list all imagestreams in the cluster
I0726 16:37:38.766381    3199 ensure.go:222] No cluster policy found.  Creating bootstrap policy based on: openshift.local.config/master/policy.json
I0726 16:37:38.767067    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.767732    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.768351    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:38.768967    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:37:39.087215    3199 reflector.go:199] github.com/openshift/origin/pkg/controller/shared/shared_informer.go:101: Failed to list *api.ClusterResourceQuota: User "system:openshift-master" cannot list all clusterresourcequotas in the cluster
E0726 16:37:39.090756    3199 reflector.go:188] github.com/openshift/origin/pkg/project/cache/cache.go:107: Failed to list *api.Namespace: User "system:openshift-master" cannot list all namespaces in the cluster
I0726 16:37:39.605550    3199 ensure.go:207] Created default security context constraint privileged
I0726 16:37:39.609657    3199 ensure.go:207] Created default security context constraint nonroot
I0726 16:37:39.611893    3199 ensure.go:207] Created default security context constraint hostmount-anyuid
I0726 16:37:39.614022    3199 ensure.go:207] Created default security context constraint hostaccess
I0726 16:37:39.618080    3199 ensure.go:207] Created default security context constraint restricted
I0726 16:37:39.620527    3199 ensure.go:207] Created default security context constraint anyuid
I0726 16:37:39.622632    3199 ensure.go:207] Created default security context constraint hostnetwork
W0726 16:37:40.555185    3199 run_components.go:207] Binding DNS on port 8053 instead of 53, which may not be resolvable from all clients
I0726 16:37:40.555623    3199 logs.go:41] skydns: ready for queries on cluster.local. for tcp4://0.0.0.0:8053 [rcache 0]
I0726 16:37:40.555633    3199 logs.go:41] skydns: ready for queries on cluster.local. for udp4://0.0.0.0:8053 [rcache 0]
I0726 16:37:40.659110    3199 run_components.go:224] DNS listening at 0.0.0.0:8053
I0726 16:37:40.666678    3199 start_master.go:582] Controllers starting (*)
E0726 16:37:40.668021    3199 util.go:45] Metric for serviceaccount_controller already registered
I0726 16:37:40.668907    3199 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:37:40.669100    3199 serviceaccounts_controller.go:120] Starting ServiceAccount controller
I0726 16:37:40.799601    3199 create_dockercfg_secrets.go:218] Dockercfg secret controller initialized, starting.
I0726 16:37:41.398047    3199 docker.go:356] Connecting to docker on unix:///var/run/docker.sock
I0726 16:37:41.398068    3199 docker.go:376] Start docker client with request timeout=2m0s
E0726 16:37:41.398275    3199 kube_docker_client.go:98] failed to retrieve docker version: Cannot connect to the Docker daemon. Is the docker daemon running on this host?
W0726 16:37:41.398290    3199 kube_docker_client.go:99] Using empty version for docker client, this may sometimes cause compatibility issue.
E0726 16:37:41.398369    3199 cni.go:163] error updating cni config: No networks found in /etc/cni/net.d
I0726 16:37:41.493971    3199 node_config.go:338] DNS Bind to 10.0.2.15:53
I0726 16:37:41.493993    3199 start_node.go:343] Starting node much (v1.5.1)
I0726 16:37:41.495149    3199 start_node.go:352] Connecting to API server https://10.0.2.15:8443
I0726 16:37:41.517030    3199 docker.go:356] Connecting to docker on unix:///var/run/docker.sock
I0726 16:37:41.517051    3199 docker.go:376] Start docker client with request timeout=0s
E0726 16:37:41.517188    3199 kube_docker_client.go:98] failed to retrieve docker version: Cannot connect to the Docker daemon. Is the docker daemon running on this host?
W0726 16:37:41.517197    3199 kube_docker_client.go:99] Using empty version for docker client, this may sometimes cause compatibility issue.
F0726 16:37:41.517216    3199 node.go:173] error: No Docker socket found at /var/run/docker.sock. Have you started the Docker daemon?
[root@much ~]# echo $?
255
[root@much ~]# 
~~~


## 启动 docker

~~~
[root@much ~]# service  docker status
Redirecting to /bin/systemctl status  docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: http://docs.docker.com
[root@much ~]# service  docker start 
Redirecting to /bin/systemctl start  docker.service
[root@much ~]# service  docker status
Redirecting to /bin/systemctl status  docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-07-26 16:54:47 CST; 2s ago
     Docs: http://docs.docker.com
 Main PID: 4503 (dockerd-current)
   CGroup: /system.slice/docker.service
           ├─4503 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userl...
           └─4510 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m...

Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.084186855+08:00" level=info msg="devmapper: Successfully created filesystem xfs on device docker-253:0-36170928-base"
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.142557140+08:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.143901310+08:00" level=info msg="Loading containers: start."
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.165721252+08:00" level=info msg="Firewalld running: true"
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.321680923+08:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16.... IP address"
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.466069791+08:00" level=info msg="Loading containers: done."
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.466206492+08:00" level=info msg="Daemon has completed initialization"
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.466222714+08:00" level=info msg="Docker daemon" commit="88a4867/1.12.6" graphdriver=devicemapper version=1.12.6
Jul 26 16:54:47 much systemd[1]: Started Docker Application Container Engine.
Jul 26 16:54:47 much dockerd-current[4503]: time="2017-07-26T16:54:47.472207628+08:00" level=info msg="API listen on /var/run/docker.sock"
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]# 
~~~

---

## 启动 openshift

~~~
[root@much ~]# openshift start 
W0726 16:54:57.840530    4648 start_master.go:291] Warning: assetConfig.loggingPublicURL: Invalid value: "": required to view aggregated container logs in the console, master start will continue.
W0726 16:54:57.840604    4648 start_master.go:291] Warning: assetConfig.metricsPublicURL: Invalid value: "": required to view cluster metrics in the console, master start will continue.
W0726 16:54:57.840614    4648 start_master.go:291] Warning: auditConfig.auditFilePath: Required value: audit can now be logged to a separate file, master start will continue.
I0726 16:54:57.844616    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.845432    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.846030    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.846628    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:54:57.846743 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:54:57.846781 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:54:57.846998 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:54:57.847259    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:54:57.847271 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:54:57.847453 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
E0726 16:54:57.850919    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/resourcequota/resource_access.go:83: Failed to list *api.ResourceQuota: Get https://10.0.2.15:8443/api/v1/resourcequotas?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
I0726 16:54:57.850951    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:54:57.850973    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:119: Failed to list *api.Secret: Get https://10.0.2.15:8443/api/v1/secrets?fieldSelector=type%3Dkubernetes.io%2Fservice-account-token&resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:54:57.851008    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/storageclass/default/admission.go:75: Failed to list *storage.StorageClass: Get https://10.0.2.15:8443/apis/storage.k8s.io/v1beta1/storageclasses?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:54:57.851043    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:103: Failed to list *api.ServiceAccount: Get https://10.0.2.15:8443/api/v1/serviceaccounts?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
2017-07-26 16:54:57.851194 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:54:57.851938    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:54:57.852585 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:54:57.852695    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.853458    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.853845    4648 plugins.go:94] No cloud provider specified.
2017-07-26 16:54:57.854003 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:54:57.854039 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:54:57.856868    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:54:57.857590 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
I0726 16:54:57.857601    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.857797    4648 start_master.go:410] Starting master on 0.0.0.0:8443 (v1.5.1)
I0726 16:54:57.857806    4648 start_master.go:411] Public master address is https://10.0.2.15:8443
I0726 16:54:57.857818    4648 start_master.go:415] Using images from "openshift/origin-<component>:v1.5.1"
2017-07-26 16:54:57.858369 I | etcdserver/api/v3rpc: grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 10.0.2.15:4001: getsockopt: connection refused"; Reconnecting to {10.0.2.15:4001 <nil>}
2017-07-26 16:54:57.858445 I | embed: peerTLS: cert = openshift.local.config/master/etcd.server.crt, key = openshift.local.config/master/etcd.server.key, ca = openshift.local.config/master/ca.crt, trusted-ca = , client-cert-auth = true
2017-07-26 16:54:57.859628 I | embed: listening for peers on https://0.0.0.0:7001
2017-07-26 16:54:57.859705 I | embed: listening for client requests on 0.0.0.0:4001
I0726 16:54:57.862806    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:54:57.863129 I | etcdserver: name = openshift.local
2017-07-26 16:54:57.863139 I | etcdserver: data dir = openshift.local.etcd
2017-07-26 16:54:57.863144 I | etcdserver: member dir = openshift.local.etcd/member
2017-07-26 16:54:57.863148 I | etcdserver: heartbeat = 100ms
2017-07-26 16:54:57.863152 I | etcdserver: election = 1000ms
2017-07-26 16:54:57.863156 I | etcdserver: snapshot count = 10000
2017-07-26 16:54:57.863167 I | etcdserver: advertise client URLs = https://10.0.2.15:4001
2017-07-26 16:54:57.875077 I | etcdserver: restarting member abca2d8f0dd66377 in cluster 28cb2054c808baa3 at commit index 369
2017-07-26 16:54:57.875146 I | raft: abca2d8f0dd66377 became follower at term 3
2017-07-26 16:54:57.875158 I | raft: newRaft abca2d8f0dd66377 [peers: [], term: 3, commit: 369, applied: 0, lastindex: 369, lastterm: 3]
I0726 16:54:57.892386    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:57.892991    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2017-07-26 16:54:57.893017 I | etcdserver: starting server... [version: 3.1.0, cluster version: to_be_decided]
2017-07-26 16:54:57.893059 I | embed: ClientTLS: cert = openshift.local.config/master/etcd.server.crt, key = openshift.local.config/master/etcd.server.key, ca = openshift.local.config/master/ca.crt, trusted-ca = , client-cert-auth = true
2017-07-26 16:54:57.895507 I | etcdserver/membership: added member abca2d8f0dd66377 [https://10.0.2.15:7001] to cluster 28cb2054c808baa3
2017-07-26 16:54:57.895586 N | etcdserver/membership: set the initial cluster version to 3.1
2017-07-26 16:54:57.895608 I | etcdserver/api: enabled capabilities for version 3.1
2017-07-26 16:54:58.475946 I | raft: abca2d8f0dd66377 is starting a new election at term 3
2017-07-26 16:54:58.476008 I | raft: abca2d8f0dd66377 became candidate at term 4
2017-07-26 16:54:58.476025 I | raft: abca2d8f0dd66377 received MsgVoteResp from abca2d8f0dd66377 at term 4
2017-07-26 16:54:58.476039 I | raft: abca2d8f0dd66377 became leader at term 4
2017-07-26 16:54:58.476046 I | raft: raft.node: abca2d8f0dd66377 elected leader abca2d8f0dd66377 at term 4
2017-07-26 16:54:58.477126 I | etcdserver: published {Name:openshift.local ClientURLs:[https://10.0.2.15:4001]} to cluster 28cb2054c808baa3
2017-07-26 16:54:58.477171 I | embed: ready to serve client requests
2017-07-26 16:54:58.477955 I | embed: serving client requests on [::]:4001
I0726 16:54:58.478218    4648 run.go:77] Started etcd at 10.0.2.15:4001
2017-07-26 16:54:58.536860 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:54:58.537027 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:54:58.537160 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:54:58.537872 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
I0726 16:54:58.538515    4648 run_components.go:229] Using default project node label selector: 
I0726 16:54:58.539531    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.540137    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.540724    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.541292    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.541845    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.542400    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
E0726 16:54:58.549963    4648 reflector.go:199] github.com/openshift/origin/pkg/controller/shared/shared_informer.go:101: Failed to list *api.ClusterResourceQuota: Get https://10.0.2.15:8443/oapi/v1/clusterresourcequotas?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:54:58.580889    4648 reflector.go:188] github.com/openshift/origin/pkg/project/cache/cache.go:107: Failed to list *api.Namespace: Get https://10.0.2.15:8443/api/v1/namespaces?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
2017-07-26 16:54:58.609412 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
2017-07-26 16:54:58.652961 I | etcdserver/api/v3rpc: Failed to dial [::]:4001: connection error: desc = "transport: remote error: tls: bad certificate"; please retry.
I0726 16:54:58.667191    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.668045    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.668961    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.669685    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.671189    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.672828    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.673914    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.674644    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.676123    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.676944    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.677677    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.678309    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.679694    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.680393    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.680968    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.681507    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.682004    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.682512    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.705500    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.707063    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.707937    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.709183    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.709914    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.710683    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.711475    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.712140    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.712817    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.713490    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.714168    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.714906    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.715663    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.716395    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.717067    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.717706    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.734757    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.735604    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.815951    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.816708    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.818125    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.819028    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.819834    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.820618    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.821259    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.821934    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.822689    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.823362    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.824013    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.824697    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.825328    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.825940    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.826579    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.827363    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.828302    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.862772    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.863593    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.864254    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.865171    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.865984    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.866671    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.867337    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.868044    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:54:58.929363    4648 master.go:171] Started OAuth2 API at 0.0.0.0:8443/oauth
I0726 16:54:58.929420    4648 master.go:171] Started Web Console 0.0.0.0:8443/console/
I0726 16:54:58.929425    4648 master.go:171] Started Kubernetes API at 0.0.0.0:8443/api
I0726 16:54:58.929429    4648 master.go:171] Started Kubernetes API apps/v1beta1 at 0.0.0.0:8443/apis
I0726 16:54:58.929432    4648 master.go:171] Started Kubernetes API authentication.k8s.io/v1beta1 at 0.0.0.0:8443/apis
I0726 16:54:58.929436    4648 master.go:171] Started Kubernetes API autoscaling/v1 at 0.0.0.0:8443/apis
I0726 16:54:58.929439    4648 master.go:171] Started Kubernetes API batch/v1 at 0.0.0.0:8443/apis
I0726 16:54:58.929442    4648 master.go:171] Started Kubernetes API batch/v2alpha1 at 0.0.0.0:8443/apis
I0726 16:54:58.929465    4648 master.go:171] Started Kubernetes API certificates.k8s.io/v1alpha1 at 0.0.0.0:8443/apis
I0726 16:54:58.929470    4648 master.go:171] Started Kubernetes API extensions/v1beta1 at 0.0.0.0:8443/apis
I0726 16:54:58.929473    4648 master.go:171] Started Kubernetes API policy/v1beta1 at 0.0.0.0:8443/apis
I0726 16:54:58.929476    4648 master.go:171] Started Kubernetes API storage.k8s.io/v1beta1 at 0.0.0.0:8443/apis
I0726 16:54:58.929480    4648 master.go:171] Started Swagger Schema API at 0.0.0.0:8443/swaggerapi/
I0726 16:54:58.929483    4648 master.go:171] Started OpenAPI Schema at 0.0.0.0:8443/swagger.json
E0726 16:54:59.040581    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/resourcequota/resource_access.go:83: Failed to list *api.ResourceQuota: Get https://10.0.2.15:8443/api/v1/resourcequotas?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:54:59.042944    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/storageclass/default/admission.go:75: Failed to list *storage.StorageClass: Get https://10.0.2.15:8443/apis/storage.k8s.io/v1beta1/storageclasses?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
E0726 16:54:59.043103    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:119: Failed to list *api.Secret: Get https://10.0.2.15:8443/api/v1/secrets?fieldSelector=type%3Dkubernetes.io%2Fservice-account-token&resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
[restful] 2017/07/26 16:54:59 log.go:30: [restful/swagger] listing is available at https://10.0.2.15:8443/swaggerapi/
[restful] 2017/07/26 16:54:59 log.go:30: [restful/swagger] https://10.0.2.15:8443/swaggerui/ is mapped to folder /swagger-ui/
E0726 16:54:59.168042    4648 reflector.go:199] github.com/openshift/origin/vendor/k8s.io/kubernetes/plugin/pkg/admission/serviceaccount/admission.go:103: Failed to list *api.ServiceAccount: Get https://10.0.2.15:8443/api/v1/serviceaccounts?resourceVersion=0: dial tcp 10.0.2.15:8443: getsockopt: connection refused
I0726 16:54:59.285777    4648 trace.go:61] Trace "cacher *api.ClusterPolicy: List" (started 2017-07-26 16:54:58.548490848 +0800 CST):
[737.204337ms] [737.204337ms] Ready
[737.224423ms] [20.086µs] watchCache locked acquired
[737.224535ms] [112ns] watchCache fresh enough
[737.228089ms] [3.554µs] Listed 1 items from cache
[737.247761ms] [19.672µs] Resized result
[737.249735ms] [1.974µs] Filtered 1 items
"cacher *api.ClusterPolicy: List" [737.254239ms] [4.504µs] END
I0726 16:54:59.290425    4648 trace.go:61] Trace "cacher *api.PolicyBinding: List" (started 2017-07-26 16:54:58.543977464 +0800 CST):
[746.378375ms] [746.378375ms] Ready
[746.402475ms] [24.1µs] watchCache locked acquired
[746.402583ms] [108ns] watchCache fresh enough
[746.406543ms] [3.96µs] Listed 4 items from cache
[746.415681ms] [9.138µs] Resized result
[746.418173ms] [2.492µs] Filtered 4 items
"cacher *api.PolicyBinding: List" [746.424333ms] [6.16µs] END
I0726 16:54:59.293512    4648 trace.go:61] Trace "cacher *api.ClusterPolicyBinding: List" (started 2017-07-26 16:54:58.543073806 +0800 CST):
[750.355006ms] [750.355006ms] Ready
[750.386879ms] [31.873µs] watchCache locked acquired
[750.386982ms] [103ns] watchCache fresh enough
[750.393874ms] [6.892µs] Listed 1 items from cache
[750.402536ms] [8.662µs] Resized result
[750.404079ms] [1.543µs] Filtered 1 items
"cacher *api.ClusterPolicyBinding: List" [750.408038ms] [3.959µs] END
I0726 16:54:59.295831    4648 trace.go:61] Trace "cacher *api.Policy: List" (started 2017-07-26 16:54:58.543851639 +0800 CST):
[751.904574ms] [751.904574ms] Ready
[751.929563ms] [24.989µs] watchCache locked acquired
[751.929679ms] [116ns] watchCache fresh enough
[751.93323ms] [3.551µs] Listed 1 items from cache
[751.946179ms] [12.949µs] Resized result
[751.948367ms] [2.188µs] Filtered 1 items
"cacher *api.Policy: List" [751.952983ms] [4.616µs] END
I0726 16:54:59.300885    4648 trace.go:61] Trace "cacher *api.Group: List" (started 2017-07-26 16:54:58.548536005 +0800 CST):
[752.248094ms] [752.248094ms] Ready
[752.293547ms] [45.453µs] watchCache locked acquired
[752.293659ms] [112ns] watchCache fresh enough
[752.300871ms] [7.212µs] Listed 0 items from cache
[752.302335ms] [1.464µs] Filtered 0 items
"cacher *api.Group: List" [752.313581ms] [11.246µs] END
I0726 16:54:59.312951    4648 serve.go:95] Serving securely on 0.0.0.0:8443
I0726 16:54:59.418837    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
W0726 16:54:59.673390    4648 lease_endpoint_reconciler.go:174] Resetting endpoints for master service "kubernetes" to [10.0.2.15]
W0726 16:54:59.962591    4648 run_components.go:207] Binding DNS on port 8053 instead of 53, which may not be resolvable from all clients
I0726 16:54:59.963060    4648 logs.go:41] skydns: ready for queries on cluster.local. for tcp4://0.0.0.0:8053 [rcache 0]
I0726 16:54:59.963070    4648 logs.go:41] skydns: ready for queries on cluster.local. for udp4://0.0.0.0:8053 [rcache 0]
I0726 16:55:00.064172    4648 run_components.go:224] DNS listening at 0.0.0.0:8053
I0726 16:55:00.067584    4648 start_master.go:582] Controllers starting (*)
E0726 16:55:00.068550    4648 util.go:45] Metric for serviceaccount_controller already registered
I0726 16:55:00.069261    4648 logs.go:41] warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
I0726 16:55:00.069587    4648 serviceaccounts_controller.go:120] Starting ServiceAccount controller
I0726 16:55:00.172233    4648 create_dockercfg_secrets.go:218] Dockercfg secret controller initialized, starting.
I0726 16:55:00.255228    4648 docker.go:356] Connecting to docker on unix:///var/run/docker.sock
I0726 16:55:00.255247    4648 docker.go:376] Start docker client with request timeout=2m0s
E0726 16:55:00.270820    4648 cni.go:163] error updating cni config: No networks found in /etc/cni/net.d
I0726 16:55:00.365752    4648 node_config.go:338] DNS Bind to 10.0.2.15:53
I0726 16:55:00.365777    4648 start_node.go:343] Starting node much (v1.5.1)
I0726 16:55:00.368790    4648 start_node.go:352] Connecting to API server https://10.0.2.15:8443
I0726 16:55:00.394582    4648 docker.go:356] Connecting to docker on unix:///var/run/docker.sock
I0726 16:55:00.394607    4648 docker.go:376] Start docker client with request timeout=0s
I0726 16:55:00.408704    4648 node.go:142] Connecting to Docker at unix:///var/run/docker.sock
I0726 16:55:00.432997    4648 feature_gate.go:181] feature gates: map[]
I0726 16:55:00.433184    4648 manager.go:143] cAdvisor running in container: "/"
I0726 16:55:00.436307    4648 node.go:390] Using iptables Proxier.
W0726 16:55:00.442910    4648 node.go:509] Failed to retrieve node info: nodes "much" not found
W0726 16:55:00.443079    4648 proxier.go:248] invalid nodeIP, initialize kube-proxy with 127.0.0.1 as nodeIP
W0726 16:55:00.443087    4648 proxier.go:253] clusterCIDR not specified, unable to distinguish between internal and external traffic
I0726 16:55:00.443100    4648 node.go:417] Tearing down userspace rules.
W0726 16:55:00.456118    4648 manager.go:151] unable to connect to Rkt api service: rkt: cannot tcp Dial rkt api service: dial tcp [::1]:15441: getsockopt: connection refused
I0726 16:55:00.469162    4648 fs.go:117] Filesystem partitions: map[/dev/mapper/cl-root:{mountpoint:/ major:253 minor:0 fsType:xfs blockSize:0} /dev/sda1:{mountpoint:/boot major:8 minor:1 fsType:xfs blockSize:0} /dev/mapper/cl-home:{mountpoint:/home major:253 minor:2 fsType:xfs blockSize:0}]
I0726 16:55:00.473938    4648 manager.go:198] Machine: {NumCores:2 CpuFrequency:2591998 MemoryCapacity:4143964160 MachineID:33dc28f7e76c4903ad9b603b77e29a7c SystemUUID:20517223-3047-48EB-8A05-AB7E628A2AC4 BootID:17c7c182-aeda-49aa-94a1-4491067ca767 Filesystems:[{Device:/dev/sda1 Capacity:1063256064 Type:vfs Inodes:524288 HasInodes:true} {Device:/dev/mapper/cl-home Capacity:20063453184 Type:vfs Inodes:9801728 HasInodes:true} {Device:/dev/mapper/cl-root Capacity:41100877824 Type:vfs Inodes:20078592 HasInodes:true}] DiskMap:map[253:0:{Name:dm-0 Major:253 Minor:0 Size:41120956416 Scheduler:none} 253:1:{Name:dm-1 Major:253 Minor:1 Size:2147483648 Scheduler:none} 253:2:{Name:dm-2 Major:253 Minor:2 Size:20073938944 Scheduler:none} 253:3:{Name:dm-3 Major:253 Minor:3 Size:107374182400 Scheduler:none} 8:0:{Name:sda Major:8 Minor:0 Size:64424509440 Scheduler:cfq}] NetworkDevices:[{Name:enp0s3 MacAddress:08:00:27:04:c7:5a Speed:1000 Mtu:1500} {Name:enp0s8 MacAddress:08:00:27:b5:a5:da Speed:1000 Mtu:1500} {Name:virbr0 MacAddress:52:54:00:16:5e:11 Speed:0 Mtu:1500} {Name:virbr0-nic MacAddress:52:54:00:16:5e:11 Speed:0 Mtu:1500}] Topology:[{Id:0 Memory:4294500352 Cores:[{Id:0 Threads:[0] Caches:[{Size:32768 Type:Data Level:1} {Size:32768 Type:Instruction Level:1} {Size:262144 Type:Unified Level:2} {Size:6291456 Type:Unified Level:3}]} {Id:1 Threads:[1] Caches:[{Size:32768 Type:Data Level:1} {Size:32768 Type:Instruction Level:1} {Size:262144 Type:Unified Level:2} {Size:6291456 Type:Unified Level:3}]}] Caches:[]}] CloudProvider:Unknown InstanceType:Unknown InstanceID:None}
I0726 16:55:00.477962    4648 manager.go:204] Version: {KernelVersion:3.10.0-514.21.1.el7.x86_64 ContainerOsVersion:CentOS Linux 7 (Core) DockerVersion:1.12.6 CadvisorVersion: CadvisorRevision:}
W0726 16:55:00.484584    4648 container_manager_linux.go:205] Running with swap on is not supported, please disable swap! This will be a fatal error by default starting in K8s v1.6! In the meantime, you can opt-in to making this a fatal error by enabling --experimental-fail-swap-on.
I0726 16:55:00.484797    4648 kubelet.go:252] Watching apiserver
W0726 16:55:00.509781    4648 kubelet_network.go:69] Hairpin mode set to "promiscuous-bridge" but kubenet is not enabled, falling back to "hairpin-veth"
I0726 16:55:00.509812    4648 kubelet.go:477] Hairpin mode set to "hairpin-veth"
I0726 16:55:00.527284    4648 docker_manager.go:262] Setting dockerRoot to /var/lib/docker
I0726 16:55:00.527307    4648 docker_manager.go:265] Setting cgroupDriver to systemd
I0726 16:55:00.535564    4648 server.go:770] Started kubelet v1.5.2+43a9be4
I0726 16:55:00.535600    4648 server.go:123] Starting to listen on 0.0.0.0:10250
E0726 16:55:00.544741    4648 kubelet.go:1146] Image garbage collection failed: unable to find data for container /
I0726 16:55:00.546171    4648 kubelet_node_status.go:230] Controller attach/detach is disabled for this node; Kubelet will attach and detach volumes
E0726 16:55:00.580389    4648 kubelet.go:1635] Failed to check if disk space is available for the runtime: failed to get fs info for "runtime": unable to find data for container /
E0726 16:55:00.580416    4648 kubelet.go:1643] Failed to check if disk space is available on the root partition: failed to get fs info for "root": unable to find data for container /
I0726 16:55:00.588112    4648 fs_resource_analyzer.go:66] Starting FS ResourceAnalyzer
I0726 16:55:00.588493    4648 status_manager.go:129] Starting to sync pod status with apiserver
I0726 16:55:00.588525    4648 kubelet.go:1715] Starting kubelet main sync loop.
I0726 16:55:00.588541    4648 kubelet.go:1726] skipping pod synchronization - [container runtime is down]
W0726 16:55:00.589870    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4503
W0726 16:55:00.589883    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4503
W0726 16:55:00.590028    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4648
W0726 16:55:00.590035    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4648
I0726 16:55:00.590060    4648 volume_manager.go:244] Starting Kubelet Volume Manager
E0726 16:55:00.612298    4648 factory.go:301] devicemapper filesystem stats will not be reported: usage of thin_ls is disabled to preserve iops
I0726 16:55:00.612355    4648 factory.go:305] Registering Docker factory
W0726 16:55:00.612371    4648 manager.go:247] Registration of the rkt container factory failed: unable to communicate with Rkt api service: rkt: cannot tcp Dial rkt api service: dial tcp [::1]:15441: getsockopt: connection refused
I0726 16:55:00.612376    4648 factory.go:54] Registering systemd factory
I0726 16:55:00.612675    4648 factory.go:86] Registering Raw factory
I0726 16:55:00.612907    4648 manager.go:1106] Started watching for new ooms in manager
I0726 16:55:00.624303    4648 oomparser.go:185] oomparser using systemd
I0726 16:55:00.625282    4648 manager.go:288] Starting recovery of all containers
I0726 16:55:00.625879    4648 manager.go:293] Recovery completed
I0726 16:55:00.698716    4648 kubelet_node_status.go:230] Controller attach/detach is disabled for this node; Kubelet will attach and detach volumes
I0726 16:55:00.717240    4648 kubelet_node_status.go:74] Attempting to register node much
I0726 16:55:00.724159    4648 kubelet_node_status.go:77] Successfully registered node much
I0726 16:55:00.903967    4648 node.go:501] Started Kubernetes Proxy on 0.0.0.0
I0726 16:55:01.004835    4648 node.go:359] Starting DNS on 10.0.2.15:53
I0726 16:55:01.004960    4648 logs.go:41] skydns: ready for queries on cluster.local. for tcp://10.0.2.15:53 [rcache 0]
I0726 16:55:01.004968    4648 logs.go:41] skydns: ready for queries on cluster.local. for udp://10.0.2.15:53 [rcache 0]
I0726 16:55:01.183058    4648 nodecontroller.go:189] Sending events to api server.
I0726 16:55:01.185665    4648 replication_controller.go:219] Starting RC Manager
I0726 16:55:01.185698    4648 replica_set.go:162] Starting ReplicaSet controller
I0726 16:55:01.185725    4648 deployment_controller.go:132] Starting deployment controller
I0726 16:55:01.185768    4648 controller.go:91] Starting CronJob Manager
I0726 16:55:01.186020    4648 horizontal.go:132] Starting HPA Controller
I0726 16:55:01.186331    4648 daemoncontroller.go:196] Starting Daemon Sets controller manager
I0726 16:55:01.186359    4648 disruption.go:317] Starting disruption controller
I0726 16:55:01.186363    4648 disruption.go:319] Sending events to api server.
I0726 16:55:01.269602    4648 start_master.go:723] Started Kubernetes Controllers
I0726 16:55:01.269749    4648 pet_set.go:146] Starting statefulset controller
I0726 16:55:01.270403    4648 attach_detach_controller.go:204] Starting Attach Detach Controller
E0726 16:55:01.283437    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283457    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283463    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283470    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283476    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283482    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283583    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283594    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283600    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283613    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283619    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283625    4648 util.go:45] Metric for replenishment_controller already registered
E0726 16:55:01.283631    4648 util.go:45] Metric for replenishment_controller already registered
I0726 16:55:01.306844    4648 secret_updating_controller.go:108] starting service signing cert update controller
I0726 16:55:01.386280    4648 start_master.go:762] Started Origin Controllers
E0726 16:55:01.395766    4648 actual_state_of_world.go:462] Failed to set statusUpdateNeeded to needed true because nodeName="much"  does not exist
I0726 16:55:01.399595    4648 nodecontroller.go:429] Initializing eviction metric for zone: 
W0726 16:55:01.399731    4648 nodecontroller.go:678] Missing timestamp for Node much. Assuming now as a timestamp.
I0726 16:55:01.399798    4648 nodecontroller.go:608] NodeController detected that zone  is now in state Normal.
I0726 16:55:01.399828    4648 event.go:217] Event(api.ObjectReference{Kind:"Node", Namespace:"", Name:"much", UID:"1b9112e4-71e0-11e7-8ccf-08002704c75a", APIVersion:"", ResourceVersion:"", FieldPath:""}): type: 'Normal' reason: 'RegisteredNode' Node much event: Registered Node much in NodeController
...
...
I0726 16:55:55.379030    4648 logs.go:41] http: TLS handshake error from 127.0.0.1:46104: tls: first record does not look like a TLS handshake
I0726 16:55:55.438528    4648 logs.go:41] http: TLS handshake error from 127.0.0.1:46106: tls: first record does not look like a TLS handshake
I0726 16:55:55.441851    4648 logs.go:41] http: TLS handshake error from 127.0.0.1:46108: tls: first record does not look like a TLS handshake
I0726 16:56:19.946669    4648 logs.go:41] http: TLS handshake error from 192.168.56.206:39728: tls: first record does not look like a TLS handshake
I0726 16:56:20.007942    4648 logs.go:41] http: TLS handshake error from 192.168.56.206:39730: tls: first record does not look like a TLS handshake
I0726 16:56:20.008900    4648 logs.go:41] http: TLS handshake error from 192.168.56.206:39732: tls: first record does not look like a TLS handshake
I0726 16:56:37.879470    4648 logs.go:41] http: TLS handshake error from 192.168.56.206:39738: remote error: tls: unknown certificate authority
I0726 16:56:55.338076    4648 logs.go:41] http: TLS handshake error from 192.168.56.206:39740: remote error: tls: unknown certificate authority
I0726 16:59:01.918429    4648 logs.go:41] http: TLS handshake error from 192.168.56.206:39774: remote error: tls: unknown certificate authority
I0726 16:59:03.982606    4648 logs.go:41] http: TLS handshake error from 10.0.2.15:54020: remote error: tls: unknown certificate authority
I0726 16:59:18.332793    4648 logs.go:41] http: TLS handshake error from 10.0.2.15:54022: remote error: tls: unknown certificate authority
W0726 17:00:00.598445    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4503
W0726 17:00:00.598490    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4503
W0726 17:00:00.598625    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4648
W0726 17:00:00.598641    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4648
2017-07-26 17:04:57.858577 I | mvcc: store.index: compact 513
2017-07-26 17:04:57.861921 I | mvcc: finished scheduled compaction at 513 (took 1.9933ms)
I0726 17:04:57.863459    4648 compact.go:159] etcd: compacted rev (513), endpoints ([https://10.0.2.15:4001])
W0726 17:05:00.608135    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4503
W0726 17:05:00.608175    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4503
W0726 17:05:00.608308    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4648
W0726 17:05:00.608325    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4648
2017-07-26 17:09:57.872319 I | mvcc: store.index: compact 614
2017-07-26 17:09:57.874693 I | mvcc: finished scheduled compaction at 614 (took 746.093µs)
I0726 17:09:57.875211    4648 compact.go:159] etcd: compacted rev (614), endpoints ([https://10.0.2.15:4001])
W0726 17:10:00.615786    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4503
W0726 17:10:00.615808    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4503
W0726 17:10:00.615873    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4648
W0726 17:10:00.615880    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4648
E0726 17:10:14.489212    4648 watcher.go:186] watch chan error: etcdserver: mvcc: required revision has been compacted
W0726 17:10:14.490129    4648 reflector.go:319] github.com/openshift/origin/pkg/unidling/controller/controller.go:195: watch of *api.Event ended with: etcdserver: mvcc: required revision has been compacted
2017-07-26 17:14:57.878335 I | mvcc: store.index: compact 670
2017-07-26 17:14:57.878875 I | mvcc: finished scheduled compaction at 670 (took 165.629µs)
I0726 17:14:57.879368    4648 compact.go:159] etcd: compacted rev (670), endpoints ([https://10.0.2.15:4001])
W0726 17:15:00.617872    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4503
W0726 17:15:00.617888    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4503
W0726 17:15:00.617939    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4648
W0726 17:15:00.617944    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4648
2017-07-26 17:19:57.887223 I | mvcc: store.index: compact 725
2017-07-26 17:19:57.888899 I | mvcc: finished scheduled compaction at 725 (took 346.648µs)
I0726 17:19:57.889593    4648 compact.go:159] etcd: compacted rev (725), endpoints ([https://10.0.2.15:4001])
W0726 17:20:00.630064    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4503
W0726 17:20:00.630105    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4503
W0726 17:20:00.630471    4648 container_manager_linux.go:728] CPUAccounting not enabled for pid: 4648
W0726 17:20:00.630507    4648 container_manager_linux.go:731] MemoryAccounting not enabled for pid: 4648
E0726 17:20:03.519657    4648 watcher.go:186] watch chan error: etcdserver: mvcc: required revision has been compacted
W0726 17:20:03.520077    4648 reflector.go:319] github.com/openshift/origin/pkg/unidling/controller/controller.go:195: watch of *api.Event ended with: etcdserver: mvcc: required revision has been compacted
...
...
...
~~~

---

## 访问 openshift 


使用 https 访问本机 IP 的 8443 端口，由于是自签名证书所以浏览器会提示要手动确认和接受证书

![openshift1.png](/images/openshift/openshift1.png)

登录界面

![openshift2.png](/images/openshift/openshift2.png)

使用 test/test 进行登录

![openshift3.png](/images/openshift/openshift3.png)

欢迎界面，可以创建新的项目

![openshift4.png](/images/openshift/openshift4.png)

填写新项目的信息

![openshift5.png](/images/openshift/openshift5.png)

![openshift6.png](/images/openshift/openshift6.png)

进入管理界面

![openshift7.png](/images/openshift/openshift7.png)

管理界面与客户端浏览器的关系

![openshift_architecture_web.png](/images/openshift/openshift_architecture_web.png)


---

# 命令汇总

* **`rpm -qa | grep openshift`**
* **`rpm -qa | grep docker`**
* **`yum  list all | grep openshift`**
* **`yum  list all | grep docker`**
* **`yum info centos-release-openshift-origin15.noarch`**
* **`yum install centos-release-openshift-origin15.noarch`**
* **`rpm -ql centos-release-openshift-origin15`**
* **`ll /etc/yum.repos.d/ | grep  OpenShift`**
* **`yum install origin`**
* **`yum install docker`**
* **`service  docker start`**
* **`service  docker status`**
* **`openshift start`**

---

[openshift_origin]:https://www.openshift.org/
[openshift]:https://www.openshift.com/
[openshift_github]:https://github.com/openshift/origin/
[openshift_doc]:https://docs.openshift.org/index.html
[gitlab]:https://about.gitlab.com/

