---
layout: post
title:  Docker 基础
author: wilmosfang
tags:   docker
categories:   docker
wc: 810  3055 34393 
excerpt: docker基础，yum 安装 docker，script 安装 docker，启动，检测，docker组，卸载，systemctl用法
comments: true
---



# 前言

**[Docker][docker]** 是目前最为火热的开源技术之一，它在应用层面(用户空间)将相关依赖进行了打包，构建出一个个轻量而完备的功能模块(镜像)，能够跨平台运行，低开销地创建传递销毁和重建，实在是开发运维居家旅行必备良品

>Docker allows you to package an application with all of its dependencies into a standardized unit for software development.

目前通用的应用场景中，**[Docker][docker]** 可以明显提升开发和运维效率

>Docker containers wrap up a piece of software in a complete filesystem that contains everything it needs to run: code, runtime, system tools, system libraries – anything you can install on a server. This guarantees that it will always run the same, regardless of the environment it is running in.

它的隔离性确保了应用的模块化，轻量性使得系资源被更为有效的使用，只是安全性还在持续的提升过程中

以下是容器和虚拟机的区别

![what-is-vm-diagram.png](/images/docker/what-is-vm-diagram.png)

每一个虚拟机除了必要的应用和它依赖的库还包含了一整个操作系统


![what-is-docker-diagram.png](/images/docker/what-is-docker-diagram.png)

每一个容器只包含必要的应用和其依赖的库，操作系统的内核是共享的(其它实例并不拥有独享内核)


这里分享一下 **[Docker][docker]** 的相关基础，详细可以参阅 **[官方文档][docker_doc]** 

> **Tip:** 当前的最新版本为 **Docker 1.10** Released on January 15, 2016

---


# 概要

* TOC
{:toc}


---

## 依赖

Docker需要运行在 **CentOS 7.X** 上 (这是以CentOS为演示平台)

* **64位** 操作系统
* 内核版本至少为 **3.10**

检查方法

~~~
[root@h103 ~]# hostnamectl 
   Static hostname: h103
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 12a02f8ee88d4b8e91d54d1390b0b275
           Boot ID: 3232f3779bf34f68959ac017c214f268
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.4.4.el7.x86_64
      Architecture: x86-64
[root@h103 ~]# 
~~~

符合要求

> **Tip:** CentOS 7 开始使用使用 **hostnamectl** 管理主机名，更为简洁方便



另外最好将系统进行升级，打上所有最新的补丁

~~~
[root@h103 ~]# yum update 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * extras: mirrors.pubyun.com
 * updates: mirrors.163.com
No packages marked for update
[root@h103 ~]# 
~~~

准备工作完成


---


## 使用yum安装Docker


### 添加yum仓库

~~~
[root@h103 ~]# ll /etc/yum.repos.d/
total 28
drwxr-xr-x 2 root root   24 Jan 19 15:18 bak
-rw-r--r-- 1 root root 1664 Dec  9 17:59 CentOS-Base.repo
-rw-r--r-- 1 root root 1309 Dec  9 17:59 CentOS-CR.repo
-rw-r--r-- 1 root root  649 Dec  9 17:59 CentOS-Debuginfo.repo
-rw-r--r-- 1 root root  290 Dec  9 17:59 CentOS-fasttrack.repo
-rw-r--r-- 1 root root  630 Dec  9 17:59 CentOS-Media.repo
-rw-r--r-- 1 root root 1331 Dec  9 17:59 CentOS-Sources.repo
-rw-r--r-- 1 root root 1952 Dec  9 17:59 CentOS-Vault.repo
[root@h103 ~]# tee /etc/yum.repos.d/docker.repo <<-'EOF'
> [dockerrepo]
> name=Docker Repository
> baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
> enabled=1
> gpgcheck=1
> gpgkey=https://yum.dockerproject.org/gpg
> EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
[root@h103 ~]# ll /etc/yum.repos.d/
total 32
drwxr-xr-x 2 root root   24 Jan 19 15:18 bak
-rw-r--r-- 1 root root 1664 Dec  9 17:59 CentOS-Base.repo
-rw-r--r-- 1 root root 1309 Dec  9 17:59 CentOS-CR.repo
-rw-r--r-- 1 root root  649 Dec  9 17:59 CentOS-Debuginfo.repo
-rw-r--r-- 1 root root  290 Dec  9 17:59 CentOS-fasttrack.repo
-rw-r--r-- 1 root root  630 Dec  9 17:59 CentOS-Media.repo
-rw-r--r-- 1 root root 1331 Dec  9 17:59 CentOS-Sources.repo
-rw-r--r-- 1 root root 1952 Dec  9 17:59 CentOS-Vault.repo
-rw-r--r-- 1 root root  166 Jan 19 17:12 docker.repo
[root@h103 ~]# cat /etc/yum.repos.d/docker.repo 
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
[root@h103 ~]# 
~~~


### 安装Docker

~~~
[root@h103 ~]# yum install docker-engine
Loaded plugins: fastestmirror, langpacks
dockerrepo                                                                                                     | 2.9 kB  00:00:00     
dockerrepo/7/primary_db                                                                                        | 8.2 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * extras: mirrors.pubyun.com
 * updates: mirrors.163.com
Resolving Dependencies
--> Running transaction check
---> Package docker-engine.x86_64 0:1.9.1-1.el7.centos will be installed
--> Processing Dependency: docker-engine-selinux >= 1.9.1-1.el7.centos for package: docker-engine-1.9.1-1.el7.centos.x86_64
--> Running transaction check
---> Package docker-engine-selinux.noarch 0:1.9.1-1.el7.centos will be installed
--> Processing Dependency: policycoreutils-python for package: docker-engine-selinux-1.9.1-1.el7.centos.noarch
--> Running transaction check
---> Package policycoreutils-python.x86_64 0:2.2.5-20.el7 will be installed
--> Processing Dependency: libsemanage-python >= 2.1.10-1 for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.2.5-20.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.4.1-5.el7 will be installed
---> Package checkpolicy.x86_64 0:2.1.12-6.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.1.10-18.el7 will be installed
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.7-46.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                                Arch                   Version                               Repository                  Size
======================================================================================================================================
Installing:
 docker-engine                          x86_64                 1.9.1-1.el7.centos                    dockerrepo                 8.2 M
Installing for dependencies:
 audit-libs-python                      x86_64                 2.4.1-5.el7                           base                        69 k
 checkpolicy                            x86_64                 2.1.12-6.el7                          base                       247 k
 docker-engine-selinux                  noarch                 1.9.1-1.el7.centos                    dockerrepo                  21 k
 libsemanage-python                     x86_64                 2.1.10-18.el7                         base                        94 k
 policycoreutils-python                 x86_64                 2.2.5-20.el7                          base                       435 k
 python-IPy                             noarch                 0.75-6.el7                            base                        32 k
 setools-libs                           x86_64                 3.3.7-46.el7                          base                       485 k

Transaction Summary
======================================================================================================================================
Install  1 Package (+7 Dependent packages)

Total download size: 9.5 M
Installed size: 40 M
Is this ok [y/d/N]: y
Downloading packages:
(1/8): audit-libs-python-2.4.1-5.el7.x86_64.rpm                                                                |  69 kB  00:00:00     
(2/8): libsemanage-python-2.1.10-18.el7.x86_64.rpm                                                             |  94 kB  00:00:00     
(3/8): python-IPy-0.75-6.el7.noarch.rpm                                                                        |  32 kB  00:00:00     
(4/8): policycoreutils-python-2.2.5-20.el7.x86_64.rpm                                                          | 435 kB  00:00:00     
(5/8): docker-engine-selinux-1.9.1-1.el7.centos.noarch.rpm                                                     |  21 kB  00:00:01     
(6/8): setools-libs-3.3.7-46.el7.x86_64.rpm                                                                    | 485 kB  00:00:00     
(7/8): checkpolicy-2.1.12-6.el7.x86_64.rpm                                                                     | 247 kB  00:00:02     
(8/8): docker-engine-1.9.1-1.el7.centos.x86_64.rpm                                                             | 8.2 MB  00:01:06     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 147 kB/s | 9.5 MB  00:01:06     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : audit-libs-python-2.4.1-5.el7.x86_64                                                                               1/8 
  Installing : libsemanage-python-2.1.10-18.el7.x86_64                                                                            2/8 
  Installing : checkpolicy-2.1.12-6.el7.x86_64                                                                                    3/8 
  Installing : python-IPy-0.75-6.el7.noarch                                                                                       4/8 
  Installing : setools-libs-3.3.7-46.el7.x86_64                                                                                   5/8 
  Installing : policycoreutils-python-2.2.5-20.el7.x86_64                                                                         6/8 
  Installing : docker-engine-selinux-1.9.1-1.el7.centos.noarch                                                                    7/8 
setsebool:  SELinux is disabled.
  Installing : docker-engine-1.9.1-1.el7.centos.x86_64                                                                            8/8 
  Verifying  : setools-libs-3.3.7-46.el7.x86_64                                                                                   1/8 
  Verifying  : python-IPy-0.75-6.el7.noarch                                                                                       2/8 
  Verifying  : checkpolicy-2.1.12-6.el7.x86_64                                                                                    3/8 
  Verifying  : docker-engine-selinux-1.9.1-1.el7.centos.noarch                                                                    4/8 
  Verifying  : docker-engine-1.9.1-1.el7.centos.x86_64                                                                            5/8 
  Verifying  : libsemanage-python-2.1.10-18.el7.x86_64                                                                            6/8 
  Verifying  : policycoreutils-python-2.2.5-20.el7.x86_64                                                                         7/8 
  Verifying  : audit-libs-python-2.4.1-5.el7.x86_64                                                                               8/8 

Installed:
  docker-engine.x86_64 0:1.9.1-1.el7.centos                                                                                           

Dependency Installed:
  audit-libs-python.x86_64 0:2.4.1-5.el7                                checkpolicy.x86_64 0:2.1.12-6.el7                            
  docker-engine-selinux.noarch 0:1.9.1-1.el7.centos                     libsemanage-python.x86_64 0:2.1.10-18.el7                    
  policycoreutils-python.x86_64 0:2.2.5-20.el7                          python-IPy.noarch 0:0.75-6.el7                               
  setools-libs.x86_64 0:3.3.7-46.el7                                   

Complete!
[root@h103 ~]# 
~~~

---

### 启动Docker

~~~
[root@h103 ~]# service docker start
Redirecting to /bin/systemctl start  docker.service
[root@h103 ~]# ps faux | grep docker
root      3315  0.0  0.0 112644   960 pts/1    S+   17:20   0:00  |       \_ grep --color=auto docker
root      3200  1.5  0.5 395368 22640 ?        Ssl  17:20   0:00 /usr/bin/docker daemon -H fd://
[root@h103 ~]# ps -Lf 3200
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root      3200     1  3200  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      3200     1  3201  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      3200     1  3202  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      3200     1  3203  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      3200     1  3205  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      3200     1  3206  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      3200     1  3242  0    7 17:20 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
[root@h103 ~]# 
[root@h103 ~]# systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2016-01-19 17:20:10 CST; 10min ago
     Docs: https://docs.docker.com
 Main PID: 3200 (docker)
   CGroup: /system.slice/docker.service
           └─3200 /usr/bin/docker daemon -H fd://

Jan 19 17:20:05 h103 systemd[1]: Starting Docker Application Container Engine...
Jan 19 17:20:05 h103 docker[3200]: time="2016-01-19T17:20:05.746819485+08:00" level=info msg="API listen on /var/run/docker.sock"
Jan 19 17:20:05 h103 docker[3200]: time="2016-01-19T17:20:05.917640473+08:00" level=warning msg="Usage of loopback devices i...ction."
Jan 19 17:20:09 h103 docker[3200]: time="2016-01-19T17:20:09.967104253+08:00" level=info msg="Firewalld running: true"
Jan 19 17:20:10 h103 docker[3200]: time="2016-01-19T17:20:10.147349577+08:00" level=info msg="Default bridge (docker0) is as...ddress"
Jan 19 17:20:10 h103 docker[3200]: time="2016-01-19T17:20:10.461970275+08:00" level=info msg="Loading containers: start."
Jan 19 17:20:10 h103 docker[3200]: time="2016-01-19T17:20:10.462289951+08:00" level=info msg="Loading containers: done."
Jan 19 17:20:10 h103 docker[3200]: time="2016-01-19T17:20:10.462313472+08:00" level=info msg="Daemon has completed initialization"
Jan 19 17:20:10 h103 docker[3200]: time="2016-01-19T17:20:10.462336163+08:00" level=info msg="Docker daemon" commit=a34a1d5 ...n=1.9.1
Jan 19 17:20:10 h103 systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
[root@h103 ~]#
~~~

> **Tip:** CentOS 7 开始使用 **systemd** 来管理服务

~~~
[root@h103 ~]# which systemctl 
/usr/bin/systemctl
[root@h103 ~]# rpm -qf /usr/bin/systemctl
systemd-219-19.el7.x86_64
[root@h103 ~]# 
[root@h103 ~]# rpm -qi systemd
Name        : systemd
Version     : 219
Release     : 19.el7
Architecture: x86_64
Install Date: Tue 19 Jan 2016 04:31:19 PM CST
Group       : Unspecified
Size        : 22289573
License     : LGPLv2+ and MIT and GPLv2+
Signature   : RSA/SHA256, Wed 25 Nov 2015 11:42:22 PM CST, Key ID 24c6a8a7f4a80eb5
Source RPM  : systemd-219-19.el7.src.rpm
Build Date  : Fri 20 Nov 2015 12:49:31 PM CST
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://www.freedesktop.org/wiki/Software/systemd
Summary     : A System and Service Manager
Description :
systemd is a system and service manager for Linux, compatible with
SysV and LSB init scripts. systemd provides aggressive parallelization
capabilities, uses socket and D-Bus activation for starting services,
offers on-demand starting of daemons, keeps track of processes using
Linux cgroups, supports snapshotting and restoring of the system
state, maintains mount and automount points and implements an
elaborate transactional dependency-based service control logic. It can
work as a drop-in replacement for sysvinit.
[root@h103 ~]# 
~~~


---

### 检查Docker

~~~
[root@h103 ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b901d36b6f2f: Pull complete 
0a6ba66e537a: Pull complete 
Digest: sha256:8be990ef2aeb16dbcb9271ddfe2610fa6658d13f6dfb8bc72074cc1ca36966a7
Status: Downloaded newer image for hello-world:latest

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

[root@h103 ~]# 
~~~

---

## 使用脚本安装Docker


确保Docker已经删除的情况下，执行如下命令

~~~
[root@h103 ~]#  curl -sSL https://get.docker.com/ | sh
+ sh -c 'sleep 3; yum -y -q install docker-engine'

If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

[root@h103 ~]#
~~~

启动Docker

~~~
[root@h103 ~]# service docker start
Redirecting to /bin/systemctl start  docker.service
[root@h103 ~]# service docker status
Redirecting to /bin/systemctl status  docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2016-01-19 17:49:51 CST; 5s ago
     Docs: https://docs.docker.com
 Main PID: 4514 (docker)
   CGroup: /system.slice/docker.service
           └─4514 /usr/bin/docker daemon -H fd://

Jan 19 17:49:48 h103 systemd[1]: Starting Docker Application Container Engine...
Jan 19 17:49:48 h103 docker[4514]: time="2016-01-19T17:49:48.493901017+08:00" level=info msg="API listen on /var/run/docker.sock"
Jan 19 17:49:48 h103 docker[4514]: time="2016-01-19T17:49:48.523287426+08:00" level=warning msg="Usage of loopback devices i...ction."
Jan 19 17:49:51 h103 docker[4514]: time="2016-01-19T17:49:51.065772044+08:00" level=info msg="Firewalld running: true"
Jan 19 17:49:51 h103 docker[4514]: time="2016-01-19T17:49:51.152288274+08:00" level=info msg="Default bridge (docker0) is as...ddress"
Jan 19 17:49:51 h103 docker[4514]: time="2016-01-19T17:49:51.279193343+08:00" level=info msg="Loading containers: start."
Jan 19 17:49:51 h103 docker[4514]: time="2016-01-19T17:49:51.279523520+08:00" level=info msg="Loading containers: done."
Jan 19 17:49:51 h103 docker[4514]: time="2016-01-19T17:49:51.279547046+08:00" level=info msg="Daemon has completed initialization"
Jan 19 17:49:51 h103 docker[4514]: time="2016-01-19T17:49:51.279578666+08:00" level=info msg="Docker daemon" commit=a34a1d5 ...n=1.9.1
Jan 19 17:49:51 h103 systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
[root@h103 ~]# ps faux | grep docker 
root      4586  0.0  0.0 112644   956 pts/1    S+   17:50   0:00  |       \_ grep --color=auto docker
root      4514  1.2  0.5 387160 22600 ?        Ssl  17:49   0:00 /usr/bin/docker daemon -H fd://
[root@h103 ~]# ps -Lf 4514
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root      4514     1  4514  0    6 17:49 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      4514     1  4515  0    6 17:49 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      4514     1  4516  0    6 17:49 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      4514     1  4517  0    6 17:49 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      4514     1  4521  0    6 17:49 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
root      4514     1  4526  0    6 17:49 ?        Ssl    0:00 /usr/bin/docker daemon -H fd://
[root@h103 ~]# 
~~~

使用相同的方式检验Docker

~~~
[root@h103 ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b901d36b6f2f: Pull complete 
0a6ba66e537a: Pull complete 
Digest: sha256:8be990ef2aeb16dbcb9271ddfe2610fa6658d13f6dfb8bc72074cc1ca36966a7
Status: Downloaded newer image for hello-world:latest

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

[root@h103 ~]# 
~~~

> **Tip:** 脚本自动创建了一个docker的软件仓库，所以其实是将上面的手动过程使用脚本自动完成了

~~~
[root@h103 ~]# ll /etc/yum.repos.d/docker-main.repo 
-rw-r--r-- 1 root root 166 Jan 19 17:46 /etc/yum.repos.d/docker-main.repo
[root@h103 ~]# cat /etc/yum.repos.d/docker-main.repo 
[docker-main-repo]
name=Docker main Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
[root@h103 ~]# 
~~~

---

## 创建docker组

* Docker不是使用的TCP端口，而是使用的Unix Socket来监听请求
* 默认情况下Docker Socket的拥有者是root
* Docker的进程一般也是以root的身份运行
* 用户如果想调用得使用sudo

为了避免只能使用sudo来调用Docker，在软件安装过程中自动创建了docker组，并且在docker进程启动时赋权给了这个组的用户以docker socket的读写权限，所以只用将管理用户加入到docker组，就可以对docker进行使用了


> **Note:** 使用docker group的方式解决了不用sudo的问题，但仍然有很大的安全隐患，因为它的操作依然相当于root，对运行在容器中的其它镜像实例有破坏潜力，相关详情可以参考 **[Docker Daemon Attack Surface][attack_surface]** 


普通用户没有docker操作权限

~~~
[root@h103 ~]# id cc
uid=1000(cc) gid=1000(cc) groups=1000(cc)
[root@h103 ~]# su - cc
Last login: Tue Jan 19 23:00:16 CST 2016 on pts/1
[cc@h103 ~]$ docker run hello-world
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
[cc@h103 ~]$ 
~~~

将普通用户添加到docker组

~~~
[root@h103 ~]# usermod -aG docker cc
[root@h103 ~]# id cc
uid=1000(cc) gid=1000(cc) groups=1000(cc),993(docker)
[root@h103 ~]#
~~~

再次尝试使用普通用户的身份执行docker命令


~~~
[root@h103 ~]# su - cc
Last login: Tue Jan 19 23:23:04 CST 2016 on pts/1
[cc@h103 ~]$ docker run hello-world

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

[cc@h103 ~]$ 
~~~


---

## 设定开机启动


~~~
[root@h103 ~]# systemctl list-unit-files| grep docker
docker.service                              disabled
docker.socket                               disabled
[root@h103 ~]# systemctl enable docker.service
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@h103 ~]# systemctl list-unit-files | grep docker
docker.service                              enabled 
docker.socket                               disabled
[root@h103 ~]# 
~~~

> **Tip:** CentOS 7 里服务的开机启动也是由 **systemctl** 来进行管理了


我们看到还有一个 **docker.socket** 不是开机启动的，它和 **docker.socket** 的关系如下


~~~
[root@h103 ~]# systemctl list-dependencies docker.service
docker.service
● ├─docker.socket
● ├─system.slice
● └─basic.target
●   ├─firewalld.service
●   ├─microcode.service
●   ├─rhel-autorelabel-mark.service
●   ├─rhel-autorelabel.service
●   ├─rhel-configure.service
●   ├─rhel-dmesg.service
●   ├─rhel-loadmodules.service
●   ├─paths.target
●   ├─slices.target
●   │ ├─-.slice
●   │ └─system.slice
●   ├─sockets.target
●   │ ├─dbus.socket
●   │ ├─dm-event.socket
●   │ ├─iscsid.socket
●   │ ├─iscsiuio.socket
●   │ ├─rpcbind.socket
●   │ ├─systemd-initctl.socket
●   │ ├─systemd-journald.socket
●   │ ├─systemd-shutdownd.socket
●   │ ├─systemd-udevd-control.socket
●   │ └─systemd-udevd-kernel.socket
[root@h103 ~]# 
[root@h103 ~]# cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/docker daemon -H fd://
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
[root@h103 ~]# cat /usr/lib/systemd/system/docker.socket
[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
[root@h103 ~]# ll /var/run/docker.sock
srw-rw---- 1 root docker 0 Jan 20 11:21 /var/run/docker.sock
[root@h103 ~]# 
~~~

可见 **docker.service** 是依赖于 **docker.socket** 的，但是并不必要开启

> **Tip:** 其实上面的步骤完成，就已经能保证docker会开机启动，原因是它依赖的 **docker.socket** 虽然本身设定为不要开机启动，但开机时会被systemctl检查然后触发启动以支持 **docker.service** 的运行


可以用上面方法也将 **docker.socket** 设为开机启动(但这一步不是非常必要)

~~~
[root@h103 ~]# systemctl list-unit-files| grep docker
docker.service                              enabled 
docker.socket                               disabled
[root@h103 ~]# systemctl enable docker.socket
Created symlink from /etc/systemd/system/sockets.target.wants/docker.socket to /usr/lib/systemd/system/docker.socket.
[root@h103 ~]# systemctl list-unit-files| grep docker
docker.service                              enabled 
docker.socket                               enabled 
[root@h103 ~]# 
~~~



---

## 卸载Docker

### 列出安装包


~~~
[root@h103 ~]# yum list installed | grep docker
docker-engine.x86_64                  1.9.1-1.el7.centos             @dockerrepo
docker-engine-selinux.noarch          1.9.1-1.el7.centos             @dockerrepo
[root@h103 ~]# 
~~~


### 删除软件包


~~~
[root@h103 ~]# yum -y remove docker-engine.x86_64
Loaded plugins: fastestmirror, langpacks
Resolving Dependencies
--> Running transaction check
---> Package docker-engine.x86_64 0:1.9.1-1.el7.centos will be erased
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                         Arch                     Version                                 Repository                     Size
======================================================================================================================================
Removing:
 docker-engine                   x86_64                   1.9.1-1.el7.centos                      @dockerrepo                    36 M

Transaction Summary
======================================================================================================================================
Remove  1 Package

Installed size: 36 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : docker-engine-1.9.1-1.el7.centos.x86_64                                                                            1/1 
  Verifying  : docker-engine-1.9.1-1.el7.centos.x86_64                                                                            1/1 

Removed:
  docker-engine.x86_64 0:1.9.1-1.el7.centos                                                                                           

Complete!
[root@h103 ~]# 
~~~

这种情况下，只删除了软件包，但是没有删除镜像，容器，卷和自己创建的本地配置



### 删除数据


~~~
[root@h103 ~]# ll /var/lib/docker
total 16
drwx------ 3 root root   77 Jan 19 17:37 containers
drwx------ 5 root root   50 Jan 19 17:37 devicemapper
drwx------ 5 root root 4096 Jan 19 17:37 graph
-rw-r--r-- 1 root root 5120 Jan 19 17:37 linkgraph.db
drwxr-x--- 3 root root   18 Jan 19 17:20 network
-rw------- 1 root root  110 Jan 19 17:37 repositories-devicemapper
drwx------ 2 root root    6 Jan 19 17:37 tmp
drwx------ 2 root root    6 Jan 19 17:20 trust
drwx------ 2 root root    6 Jan 19 17:20 volumes
[root@h103 ~]# du -sh /var/lib/docker
61M	/var/lib/docker
[root@h103 ~]# rm -rf /var/lib/docker
[root@h103 ~]# du -sh /var/lib/docker
du: cannot access ‘/var/lib/docker’: No such file or directory
[root@h103 ~]# 
~~~

其它配置文件可以根据具体项目进行定位和清理





---


# 命令汇总


* **`hostnamectl`**
* **`yum update`**
* **`tee /etc/yum.repos.d/docker.repo <<-'EOF'`**
* **`cat /etc/yum.repos.d/docker.repo`**
* **`yum install docker-engine`**
* **`service docker start`**
* **`systemctl status docker.service`**
* **`rpm -qi systemd`**
* **`docker run hello-world`**
* **`curl -sSL https://get.docker.com/ | sh`**
* **`cat /etc/yum.repos.d/docker-main.repo`**
* **`usermod -aG docker cc`**
* **`id cc`**
* **`su - cc`**
* **`docker run hello-world`**
* **`systemctl enable docker.service`**
* **`systemctl list-unit-files | grep docker`**
* **`systemctl list-dependencies docker.service`**
* **`cat /usr/lib/systemd/system/docker.service`**
* **`cat /usr/lib/systemd/system/docker.socket`**
* **`ll /var/run/docker.sock`**
* **`systemctl enable docker.socket`**
* **`yum list installed | grep docker`**
* **`yum -y remove docker-engine.x86_64`**
* **`rm -rf /var/lib/docker`**

---

# 附

## systemctl用法小结


CLI     | COMMENT
-------- | ---
systemctl is-enabled *.service | 查询服务是否开机启动
systemctl enable *.service    | 开机运行服务
systemctl disable *.service     | 取消开机运行
systemctl start *.service | 启动服务
systemctl stop *.service |停止服务
systemctl restart *.service |重启服务
systemctl reload *.service|重新加载服务配置文件
systemctl status *.service|查询服务运行状态
systemctl --failed |显示启动失败的服务
systemctl list-unit-files | 查看所有服务及开机启动状态
systemctl list-dependencies *.service |查看服务依赖




---

[docker]:https://www.docker.com/
[docker_doc]:https://docs.docker.com/
[attack_surface]:https://docs.docker.com/engine/articles/security/#docker-daemon-attack-surface

