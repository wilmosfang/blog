---
layout: post
title: "Install Docker"
author:  wilmosfang
date: 2018-06-03 02:05:04
image: '/assets/img/'
excerpt: '安装 Docker'
main-class: 'docker'
color: '#067cbd'
tags:
 - docker
categories: 
 - docker
twitter_text: 'simple process of Docker installation'
introduction: 'Installation of Docker'
---

# 前言

**[Docker][docker]** 是一款开源的容器引擎与管理套件

>Docker is the company driving the container movement and the only container platform provider to address every application across the hybrid cloud. Today’s businesses are under pressure to digitally transform but are constrained by existing applications and infrastructure while rationalizing an increasingly diverse portfolio of clouds, datacenters and application architectures. Docker enables true independence between applications and infrastructure and developers and IT ops to unlock their potential and creates a model for better collaboration and innovation.

**[Docker][docker]** 的设计目标的是简化应用的管理操作，提升应用组织协作与生命周期的管理效率

几乎已经成了 paas 的基础组件与事实标准，是 k8s 与 rancher 的基础

这里演示一下如何构建 **[Docker][docker]**

参考 **[Get Docker CE for CentOS][docker_install]**

> **Tip:** 当前的版本为 **Docker 18.03.1**

---

# 操作

## 系统环境

~~~
[root@h171 ~]# hostnamectl 
   Static hostname: h171
         Icon name: computer-vm
           Chassis: vm
        Machine ID: d46f9440d4be429ea66b726977adf233
           Boot ID: b285f5a2d43e4b02849c7b267e307993
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[root@h171 ~]# cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core) 
[root@h171 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84654sec preferred_lft 84654sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:48:f4:2c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.171/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe48:f42c/64 scope link 
       valid_lft forever preferred_lft forever
[root@h171 ~]# 
~~~

## 删除旧版本 docker

~~~
[root@h171 ~]# yum remove docker \
>                   docker-client \
>                   docker-client-latest \
>                   docker-common \
>                   docker-latest \
>                   docker-latest-logrotate \
>                   docker-logrotate \
>                   docker-selinux \
>                   docker-engine-selinux \
>                   docker-engine
Loaded plugins: fastestmirror
No Match for argument: docker
No Match for argument: docker-client
No Match for argument: docker-client-latest
No Match for argument: docker-common
No Match for argument: docker-latest
No Match for argument: docker-latest-logrotate
No Match for argument: docker-logrotate
No Match for argument: docker-selinux
No Match for argument: docker-engine-selinux
No Match for argument: docker-engine
No Packages marked for removal
[root@h171 ~]# 
~~~

## 安装依赖软件

~~~
[root@h171 ~]# yum install -y yum-utils \
>   device-mapper-persistent-data \
>   lvm2
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
base                                                     | 3.6 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/4): base/7/x86_64/group_gz                              | 166 kB   00:00     
(2/4): extras/7/x86_64/primary_db                          | 147 kB   00:00     
(3/4): updates/7/x86_64/primary_db                         | 2.0 MB   00:00     
(4/4): base/7/x86_64/primary_db                            | 5.9 MB   00:02     
Package yum-utils-1.1.31-45.el7.noarch already installed and latest version
Package device-mapper-persistent-data-0.7.3-3.el7.x86_64 already installed and latest version
Package 7:lvm2-2.02.177-4.el7.x86_64 already installed and latest version
Nothing to do
[root@h171 ~]# 
~~~


## 安装库

~~~
[root@h171 ~]# yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@h171 ~]# ll /etc/yum.repos.d/
total 36
-rw-r--r--. 1 root root 1664 Apr 28 16:35 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Apr 28 16:35 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Apr 28 16:35 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Apr 28 16:35 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Apr 28 16:35 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Apr 28 16:35 CentOS-Sources.repo
-rw-r--r--. 1 root root 4768 Apr 28 16:35 CentOS-Vault.repo
-rw-r--r--. 1 root root 2424 Jun  1 23:42 docker-ce.repo
[root@h171 ~]# cat /etc/yum.repos.d/docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://download.docker.com/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://download.docker.com/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://download.docker.com/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://download.docker.com/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
[root@h171 ~]# 
~~~

>**Tip:** 默认情况下 **edge** 和 **test** 的库是被关闭的，可以使用下面的方式打开

~~~
yum-config-manager --enable docker-ce-edge
yum-config-manager --enable docker-ce-test
~~~

关闭

~~~
yum-config-manager --disable docker-ce-edge
~~~


## 安装 Docker ce

先更新仓库然后看一下有哪些包

~~~
[root@h171 ~]# yum list all | grep docker 
cockpit-docker.x86_64                       165-3.el7.centos           extras   
docker.x86_64                               2:1.13.1-63.git94f4240.el7.centos
docker-ce.x86_64                            18.03.1.ce-1.el7.centos    docker-ce-stable
docker-ce-selinux.noarch                    17.03.2.ce-1.el7.centos    docker-ce-stable
docker-client.x86_64                        2:1.13.1-63.git94f4240.el7.centos
docker-client-latest.x86_64                 1.13.1-58.git87f2fab.el7.centos
docker-common.x86_64                        2:1.13.1-63.git94f4240.el7.centos
docker-devel.x86_64                         1.3.2-4.el7.centos         extras   
docker-distribution.x86_64                  2.6.2-2.git48294d9.el7     extras   
docker-forward-journald.x86_64              1.10.3-44.el7.centos       extras   
docker-latest.x86_64                        1.13.1-58.git87f2fab.el7.centos
docker-latest-logrotate.x86_64              1.13.1-58.git87f2fab.el7.centos
docker-latest-v1.10-migrator.x86_64         1.13.1-58.git87f2fab.el7.centos
docker-logrotate.x86_64                     2:1.13.1-63.git94f4240.el7.centos
docker-lvm-plugin.x86_64                    2:1.13.1-63.git94f4240.el7.centos
docker-novolume-plugin.x86_64               2:1.13.1-63.git94f4240.el7.centos
docker-python.x86_64                        1.4.0-115.el7              extras   
docker-registry.x86_64                      0.9.1-7.el7                extras   
docker-unit-test.x86_64                     2:1.13.1-63.git94f4240.el7.centos
docker-v1.10-migrator.x86_64                2:1.13.1-63.git94f4240.el7.centos
pcp-pmda-docker.x86_64                      3.12.2-5.el7               base     
python-docker-py.noarch                     1.10.6-3.el7               extras   
python-docker-pycreds.noarch                1.10.6-3.el7               extras   
[root@h171 ~]# 
~~~

安装 docker ce

~~~
[root@h171 ~]# yum install docker-ce
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 0:18.03.1.ce-1.el7.centos will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-18.03.1.ce-1.el7.centos.x86_64
--> Processing Dependency: libcgroup for package: docker-ce-18.03.1.ce-1.el7.centos.x86_64
--> Processing Dependency: pigz for package: docker-ce-18.03.1.ce-1.el7.centos.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-ce-18.03.1.ce-1.el7.centos.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.55-1.el7 will be installed
--> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.55-1.el7.noarch
---> Package libcgroup.x86_64 0:0.41-15.el7 will be installed
---> Package libtool-ltdl.x86_64 0:2.4.2-22.el7_3 will be installed
---> Package pigz.x86_64 0:2.3.3-1.el7.centos will be installed
--> Running transaction check
---> Package policycoreutils-python.x86_64 0:2.5-22.el7 will be installed
--> Processing Dependency: setools-libs >= 3.3.8-2 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-9 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.1-3.el7 will be installed
---> Package checkpolicy.x86_64 0:2.5-6.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-11.el7 will be installed
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.8-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================
 Package                                      Arch                         Version                                          Repository                              Size
=========================================================================================================================================================================
Installing:
 docker-ce                                    x86_64                       18.03.1.ce-1.el7.centos                          docker-ce-stable                        35 M
Installing for dependencies:
 audit-libs-python                            x86_64                       2.8.1-3.el7                                      base                                    75 k
 checkpolicy                                  x86_64                       2.5-6.el7                                        base                                   294 k
 container-selinux                            noarch                       2:2.55-1.el7                                     extras                                  34 k
 libcgroup                                    x86_64                       0.41-15.el7                                      base                                    65 k
 libsemanage-python                           x86_64                       2.5-11.el7                                       base                                   112 k
 libtool-ltdl                                 x86_64                       2.4.2-22.el7_3                                   base                                    49 k
 pigz                                         x86_64                       2.3.3-1.el7.centos                               extras                                  68 k
 policycoreutils-python                       x86_64                       2.5-22.el7                                       base                                   454 k
 python-IPy                                   noarch                       0.75-6.el7                                       base                                    32 k
 setools-libs                                 x86_64                       3.3.8-2.el7                                      base                                   619 k

Transaction Summary
=========================================================================================================================================================================
Install  1 Package (+10 Dependent packages)

Total download size: 36 M
Installed size: 156 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/extras/packages/container-selinux-2.55-1.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for container-selinux-2.55-1.el7.noarch.rpm is not installed
(1/11): container-selinux-2.55-1.el7.noarch.rpm                                                                                                   |  34 kB  00:00:00     
Public key for audit-libs-python-2.8.1-3.el7.x86_64.rpm is not installed
(2/11): audit-libs-python-2.8.1-3.el7.x86_64.rpm                                                                                                  |  75 kB  00:00:00     
(3/11): libcgroup-0.41-15.el7.x86_64.rpm                                                                                                          |  65 kB  00:00:00     
(4/11): libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm                                                                                                    |  49 kB  00:00:00     
(5/11): pigz-2.3.3-1.el7.centos.x86_64.rpm                                                                                                        |  68 kB  00:00:00     
(6/11): python-IPy-0.75-6.el7.noarch.rpm                                                                                                          |  32 kB  00:00:00     
(7/11): checkpolicy-2.5-6.el7.x86_64.rpm                                                                                                          | 294 kB  00:00:00     
(8/11): libsemanage-python-2.5-11.el7.x86_64.rpm                                                                                                  | 112 kB  00:00:00     
(9/11): policycoreutils-python-2.5-22.el7.x86_64.rpm                                                                                              | 454 kB  00:00:00     
(10/11): setools-libs-3.3.8-2.el7.x86_64.rpm                                                                                                      | 619 kB  00:00:00     
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY00:00 ETA 
Public key for docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm is not installed
(11/11): docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm                                                                                             |  35 MB  00:00:14     
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                    2.5 MB/s |  36 MB  00:00:14     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-5.1804.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Is this ok [y/N]: y
Retrieving key from https://download.docker.com/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libcgroup-0.41-15.el7.x86_64                                                                                                                         1/11 
  Installing : pigz-2.3.3-1.el7.centos.x86_64                                                                                                                       2/11 
  Installing : audit-libs-python-2.8.1-3.el7.x86_64                                                                                                                 3/11 
  Installing : libtool-ltdl-2.4.2-22.el7_3.x86_64                                                                                                                   4/11 
  Installing : python-IPy-0.75-6.el7.noarch                                                                                                                         5/11 
  Installing : checkpolicy-2.5-6.el7.x86_64                                                                                                                         6/11 
  Installing : libsemanage-python-2.5-11.el7.x86_64                                                                                                                 7/11 
  Installing : setools-libs-3.3.8-2.el7.x86_64                                                                                                                      8/11 
  Installing : policycoreutils-python-2.5-22.el7.x86_64                                                                                                             9/11 
  Installing : 2:container-selinux-2.55-1.el7.noarch                                                                                                               10/11 
  Installing : docker-ce-18.03.1.ce-1.el7.centos.x86_64                                                                                                            11/11 
  Verifying  : libcgroup-0.41-15.el7.x86_64                                                                                                                         1/11 
  Verifying  : docker-ce-18.03.1.ce-1.el7.centos.x86_64                                                                                                             2/11 
  Verifying  : setools-libs-3.3.8-2.el7.x86_64                                                                                                                      3/11 
  Verifying  : policycoreutils-python-2.5-22.el7.x86_64                                                                                                             4/11 
  Verifying  : libsemanage-python-2.5-11.el7.x86_64                                                                                                                 5/11 
  Verifying  : 2:container-selinux-2.55-1.el7.noarch                                                                                                                6/11 
  Verifying  : checkpolicy-2.5-6.el7.x86_64                                                                                                                         7/11 
  Verifying  : python-IPy-0.75-6.el7.noarch                                                                                                                         8/11 
  Verifying  : libtool-ltdl-2.4.2-22.el7_3.x86_64                                                                                                                   9/11 
  Verifying  : audit-libs-python-2.8.1-3.el7.x86_64                                                                                                                10/11 
  Verifying  : pigz-2.3.3-1.el7.centos.x86_64                                                                                                                      11/11 

Installed:
  docker-ce.x86_64 0:18.03.1.ce-1.el7.centos                                                                                                                             

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.1-3.el7   checkpolicy.x86_64 0:2.5-6.el7         container-selinux.noarch 2:2.55-1.el7   libcgroup.x86_64 0:0.41-15.el7              
  libsemanage-python.x86_64 0:2.5-11.el7   libtool-ltdl.x86_64 0:2.4.2-22.el7_3   pigz.x86_64 0:2.3.3-1.el7.centos        policycoreutils-python.x86_64 0:2.5-22.el7  
  python-IPy.noarch 0:0.75-6.el7           setools-libs.x86_64 0:3.3.8-2.el7     

Complete!
[root@h171 ~]# rpm -qa | grep docker -i 
docker-ce-18.03.1.ce-1.el7.centos.x86_64
[root@h171 ~]# 
~~~

**Tip:** 可以使用下面的方式查看历史版本

~~~
[root@h171 ~]#  yum list docker-ce --showduplicates | sort -r
 * updates: mirror.pregi.net
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
 * extras: mirror.pregi.net
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            18.03.1.ce-1.el7.centos            @docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable 
 * base: mirror.pregi.net
Available Packages
[root@h171 ~]#
~~~

查看安装了些什么

~~~
[root@h171 ~]# rpm -ql docker-ce-18.03.1.ce-1.el7.centos.x86_64
/etc/udev/rules.d/80-docker.rules
/usr/bin/docker
/usr/bin/docker-containerd
/usr/bin/docker-containerd-ctr
/usr/bin/docker-containerd-shim
/usr/bin/docker-init
/usr/bin/docker-proxy
/usr/bin/docker-runc
/usr/bin/dockerd
/usr/lib/systemd/system/docker.service
/usr/share/bash-completion/completions/docker
/usr/share/doc/docker-ce-18.03.1.ce
/usr/share/doc/docker-ce-18.03.1.ce/cli-LICENSE
/usr/share/doc/docker-ce-18.03.1.ce/cli-MAINTAINERS
/usr/share/doc/docker-ce-18.03.1.ce/cli-NOTICE
/usr/share/doc/docker-ce-18.03.1.ce/cli-README.md
/usr/share/doc/docker-ce-18.03.1.ce/engine-AUTHORS
/usr/share/doc/docker-ce-18.03.1.ce/engine-CHANGELOG.md
/usr/share/doc/docker-ce-18.03.1.ce/engine-CONTRIBUTING.md
/usr/share/doc/docker-ce-18.03.1.ce/engine-LICENSE
/usr/share/doc/docker-ce-18.03.1.ce/engine-MAINTAINERS
/usr/share/doc/docker-ce-18.03.1.ce/engine-NOTICE
/usr/share/doc/docker-ce-18.03.1.ce/engine-README.md
/usr/share/fish/vendor_completions.d/docker.fish
/usr/share/man/man1/docker-attach.1.gz
/usr/share/man/man1/docker-build.1.gz
/usr/share/man/man1/docker-checkpoint-create.1.gz
/usr/share/man/man1/docker-checkpoint-ls.1.gz
/usr/share/man/man1/docker-checkpoint-rm.1.gz
/usr/share/man/man1/docker-checkpoint.1.gz
/usr/share/man/man1/docker-commit.1.gz
/usr/share/man/man1/docker-config-create.1.gz
/usr/share/man/man1/docker-config-inspect.1.gz
/usr/share/man/man1/docker-config-ls.1.gz
/usr/share/man/man1/docker-config-rm.1.gz
/usr/share/man/man1/docker-config.1.gz
/usr/share/man/man1/docker-container-attach.1.gz
/usr/share/man/man1/docker-container-commit.1.gz
/usr/share/man/man1/docker-container-cp.1.gz
/usr/share/man/man1/docker-container-create.1.gz
/usr/share/man/man1/docker-container-diff.1.gz
/usr/share/man/man1/docker-container-exec.1.gz
/usr/share/man/man1/docker-container-export.1.gz
/usr/share/man/man1/docker-container-inspect.1.gz
/usr/share/man/man1/docker-container-kill.1.gz
/usr/share/man/man1/docker-container-logs.1.gz
/usr/share/man/man1/docker-container-ls.1.gz
/usr/share/man/man1/docker-container-pause.1.gz
/usr/share/man/man1/docker-container-port.1.gz
/usr/share/man/man1/docker-container-prune.1.gz
/usr/share/man/man1/docker-container-rename.1.gz
/usr/share/man/man1/docker-container-restart.1.gz
/usr/share/man/man1/docker-container-rm.1.gz
/usr/share/man/man1/docker-container-run.1.gz
/usr/share/man/man1/docker-container-start.1.gz
/usr/share/man/man1/docker-container-stats.1.gz
/usr/share/man/man1/docker-container-stop.1.gz
/usr/share/man/man1/docker-container-top.1.gz
/usr/share/man/man1/docker-container-unpause.1.gz
/usr/share/man/man1/docker-container-update.1.gz
/usr/share/man/man1/docker-container-wait.1.gz
/usr/share/man/man1/docker-container.1.gz
/usr/share/man/man1/docker-cp.1.gz
/usr/share/man/man1/docker-create.1.gz
/usr/share/man/man1/docker-deploy.1.gz
/usr/share/man/man1/docker-diff.1.gz
/usr/share/man/man1/docker-events.1.gz
/usr/share/man/man1/docker-exec.1.gz
/usr/share/man/man1/docker-export.1.gz
/usr/share/man/man1/docker-history.1.gz
/usr/share/man/man1/docker-image-build.1.gz
/usr/share/man/man1/docker-image-history.1.gz
/usr/share/man/man1/docker-image-import.1.gz
/usr/share/man/man1/docker-image-inspect.1.gz
/usr/share/man/man1/docker-image-load.1.gz
/usr/share/man/man1/docker-image-ls.1.gz
/usr/share/man/man1/docker-image-prune.1.gz
/usr/share/man/man1/docker-image-pull.1.gz
/usr/share/man/man1/docker-image-push.1.gz
/usr/share/man/man1/docker-image-rm.1.gz
/usr/share/man/man1/docker-image-save.1.gz
/usr/share/man/man1/docker-image-tag.1.gz
/usr/share/man/man1/docker-image.1.gz
/usr/share/man/man1/docker-images.1.gz
/usr/share/man/man1/docker-import.1.gz
/usr/share/man/man1/docker-info.1.gz
/usr/share/man/man1/docker-inspect.1.gz
/usr/share/man/man1/docker-kill.1.gz
/usr/share/man/man1/docker-load.1.gz
/usr/share/man/man1/docker-login.1.gz
/usr/share/man/man1/docker-logout.1.gz
/usr/share/man/man1/docker-logs.1.gz
/usr/share/man/man1/docker-manifest-annotate.1.gz
/usr/share/man/man1/docker-manifest-create.1.gz
/usr/share/man/man1/docker-manifest-inspect.1.gz
/usr/share/man/man1/docker-manifest-push.1.gz
/usr/share/man/man1/docker-manifest.1.gz
/usr/share/man/man1/docker-network-connect.1.gz
/usr/share/man/man1/docker-network-create.1.gz
/usr/share/man/man1/docker-network-disconnect.1.gz
/usr/share/man/man1/docker-network-inspect.1.gz
/usr/share/man/man1/docker-network-ls.1.gz
/usr/share/man/man1/docker-network-prune.1.gz
/usr/share/man/man1/docker-network-rm.1.gz
/usr/share/man/man1/docker-network.1.gz
/usr/share/man/man1/docker-node-demote.1.gz
/usr/share/man/man1/docker-node-inspect.1.gz
/usr/share/man/man1/docker-node-ls.1.gz
/usr/share/man/man1/docker-node-promote.1.gz
/usr/share/man/man1/docker-node-ps.1.gz
/usr/share/man/man1/docker-node-rm.1.gz
/usr/share/man/man1/docker-node-update.1.gz
/usr/share/man/man1/docker-node.1.gz
/usr/share/man/man1/docker-pause.1.gz
/usr/share/man/man1/docker-plugin-create.1.gz
/usr/share/man/man1/docker-plugin-disable.1.gz
/usr/share/man/man1/docker-plugin-enable.1.gz
/usr/share/man/man1/docker-plugin-inspect.1.gz
/usr/share/man/man1/docker-plugin-install.1.gz
/usr/share/man/man1/docker-plugin-ls.1.gz
/usr/share/man/man1/docker-plugin-push.1.gz
/usr/share/man/man1/docker-plugin-rm.1.gz
/usr/share/man/man1/docker-plugin-set.1.gz
/usr/share/man/man1/docker-plugin-upgrade.1.gz
/usr/share/man/man1/docker-plugin.1.gz
/usr/share/man/man1/docker-port.1.gz
/usr/share/man/man1/docker-ps.1.gz
/usr/share/man/man1/docker-pull.1.gz
/usr/share/man/man1/docker-push.1.gz
/usr/share/man/man1/docker-rename.1.gz
/usr/share/man/man1/docker-restart.1.gz
/usr/share/man/man1/docker-rm.1.gz
/usr/share/man/man1/docker-rmi.1.gz
/usr/share/man/man1/docker-run.1.gz
/usr/share/man/man1/docker-save.1.gz
/usr/share/man/man1/docker-search.1.gz
/usr/share/man/man1/docker-secret-create.1.gz
/usr/share/man/man1/docker-secret-inspect.1.gz
/usr/share/man/man1/docker-secret-ls.1.gz
/usr/share/man/man1/docker-secret-rm.1.gz
/usr/share/man/man1/docker-secret.1.gz
/usr/share/man/man1/docker-service-create.1.gz
/usr/share/man/man1/docker-service-inspect.1.gz
/usr/share/man/man1/docker-service-logs.1.gz
/usr/share/man/man1/docker-service-ls.1.gz
/usr/share/man/man1/docker-service-ps.1.gz
/usr/share/man/man1/docker-service-rm.1.gz
/usr/share/man/man1/docker-service-rollback.1.gz
/usr/share/man/man1/docker-service-scale.1.gz
/usr/share/man/man1/docker-service-update.1.gz
/usr/share/man/man1/docker-service.1.gz
/usr/share/man/man1/docker-stack-deploy.1.gz
/usr/share/man/man1/docker-stack-ls.1.gz
/usr/share/man/man1/docker-stack-ps.1.gz
/usr/share/man/man1/docker-stack-rm.1.gz
/usr/share/man/man1/docker-stack-services.1.gz
/usr/share/man/man1/docker-stack.1.gz
/usr/share/man/man1/docker-start.1.gz
/usr/share/man/man1/docker-stats.1.gz
/usr/share/man/man1/docker-stop.1.gz
/usr/share/man/man1/docker-swarm-ca.1.gz
/usr/share/man/man1/docker-swarm-init.1.gz
/usr/share/man/man1/docker-swarm-join-token.1.gz
/usr/share/man/man1/docker-swarm-join.1.gz
/usr/share/man/man1/docker-swarm-leave.1.gz
/usr/share/man/man1/docker-swarm-unlock-key.1.gz
/usr/share/man/man1/docker-swarm-unlock.1.gz
/usr/share/man/man1/docker-swarm-update.1.gz
/usr/share/man/man1/docker-swarm.1.gz
/usr/share/man/man1/docker-system-df.1.gz
/usr/share/man/man1/docker-system-events.1.gz
/usr/share/man/man1/docker-system-info.1.gz
/usr/share/man/man1/docker-system-prune.1.gz
/usr/share/man/man1/docker-system.1.gz
/usr/share/man/man1/docker-tag.1.gz
/usr/share/man/man1/docker-top.1.gz
/usr/share/man/man1/docker-trust-inspect.1.gz
/usr/share/man/man1/docker-trust-key-generate.1.gz
/usr/share/man/man1/docker-trust-key-load.1.gz
/usr/share/man/man1/docker-trust-key.1.gz
/usr/share/man/man1/docker-trust-revoke.1.gz
/usr/share/man/man1/docker-trust-sign.1.gz
/usr/share/man/man1/docker-trust-signer-add.1.gz
/usr/share/man/man1/docker-trust-signer-remove.1.gz
/usr/share/man/man1/docker-trust-signer.1.gz
/usr/share/man/man1/docker-trust.1.gz
/usr/share/man/man1/docker-unpause.1.gz
/usr/share/man/man1/docker-update.1.gz
/usr/share/man/man1/docker-version.1.gz
/usr/share/man/man1/docker-volume-create.1.gz
/usr/share/man/man1/docker-volume-inspect.1.gz
/usr/share/man/man1/docker-volume-ls.1.gz
/usr/share/man/man1/docker-volume-prune.1.gz
/usr/share/man/man1/docker-volume-rm.1.gz
/usr/share/man/man1/docker-volume.1.gz
/usr/share/man/man1/docker-wait.1.gz
/usr/share/man/man1/docker.1.gz
/usr/share/man/man5/Dockerfile.5.gz
/usr/share/man/man5/docker-config-json.5.gz
/usr/share/man/man8/dockerd.8.gz
/usr/share/nano/Dockerfile.nanorc
/usr/share/vim/vimfiles/doc/dockerfile.txt
/usr/share/vim/vimfiles/ftdetect/dockerfile.vim
/usr/share/vim/vimfiles/syntax/dockerfile.vim
/usr/share/zsh/vendor-completions/_docker
[root@h171 ~]# rpm -ql docker-ce-18.03.1.ce-1.el7.centos.x86_64 | grep -v share
/etc/udev/rules.d/80-docker.rules
/usr/bin/docker
/usr/bin/docker-containerd
/usr/bin/docker-containerd-ctr
/usr/bin/docker-containerd-shim
/usr/bin/docker-init
/usr/bin/docker-proxy
/usr/bin/docker-runc
/usr/bin/dockerd
/usr/lib/systemd/system/docker.service
[root@h171 ~]# 
~~~

## 启动 Docker

~~~
[root@h171 ~]# systemctl start docker
[root@h171 ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2018-06-03 02:53:38 UTC; 12s ago
     Docs: https://docs.docker.com
 Main PID: 3755 (dockerd)
    Tasks: 21
   Memory: 39.2M
   CGroup: /system.slice/docker.service
           ├─3755 /usr/bin/dockerd
           └─3762 docker-containerd --config /var/run/docker/containerd/containerd.toml

Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38Z" level=info msg=serving... address="/var/run/docker/containerd/docker-containerd.sock" modul...inerd/grpc"
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38Z" level=info msg="containerd successfully booted in 0.008033s" module=containerd
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.429964055Z" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.430829101Z" level=info msg="Loading containers: start."
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.587215806Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0....IP address"
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.635052743Z" level=info msg="Loading containers: done."
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.656867690Z" level=info msg="Docker daemon" commit=9ee9f40 graphdriver(s)=overlay2 version=18.03.1-ce
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.657123874Z" level=info msg="Daemon has completed initialization"
Jun 03 02:53:38 h171 dockerd[3755]: time="2018-06-03T02:53:38.663502365Z" level=info msg="API listen on /var/run/docker.sock"
Jun 03 02:53:38 h171 systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
[root@h171 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@h171 ~]# 
[root@h171 ~]# ps faux | grep -i docker
root      3909  0.0  0.0  12520  1000 pts/0    S+   02:54   0:00                          \_ grep --color=auto -i docker
root      3755  0.5  0.7 552456 62392 ?        Ssl  02:53   0:00 /usr/bin/dockerd
root      3762  0.9  0.2 440596 22956 ?        Ssl  02:53   0:00  \_ docker-containerd --config /var/run/docker/containerd/containerd.toml
[root@h171 ~]# 
~~~

## 检查安装

~~~
[root@h171 ~]# docker run hello-world
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

[root@h171 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
9141a1590781        hello-world         "/hello"            7 seconds ago       Exited (0) 6 seconds ago                       relaxed_ardinghelli
[root@h171 ~]# 
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

因为是使用的 yum 工具配置 repo, 然后使用 yum 安装的

所以相应容易

只需要保证网络可达，其它就都很简单

* TOC
{:toc}

---

[docker]:https://www.docker.com/
[docker_install]:https://docs.docker.com/install/linux/docker-ce/centos/