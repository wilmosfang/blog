---
layout: post
title: "Install Gitea"
author:  wilmosfang
date: 2018-05-20 16:56:27
image: '/assets/img/'
excerpt: 'Gitea 的安装'
main-class: git
color: '#ef4c2c'
tags:
 - go
 - gitea
 - git
categories:
 - git
twitter_text: 'simple process of Gitea installation'
introduction: 'Installation of Gitea'
---


# 前言

**[Gitea][gitea]** 是一款使用 Go 编写的，轻量的代码版本管理软件，开源于 MIT 许可

>A painless self-hosted Git service
>Gitea is a community managed fork of Gogs, lightweight code hosting solution written in Go and published under the MIT license

**[Gitea][gitea]** forks 自 Gogs 的代码

**[Gitea][gitea]** 具备简单、跨平台、轻量、开源的特性

要快速构建一个 git 管理平台，**[Gitea][gitea]** 可以作为一个备选考虑

如果需要更全面和更完善的代码管理特性，可以考虑 Gitlab

这里演示一下如何构建 **[Gitea][gitea]**

参考 **[从二进制安装][gitea_install]**

> **Tip:** 当前的版本为 **gitea 1.4.1**

---

## 运行环境

~~~
[vagrant@gitea ~]$ hostnamectl 
   Static hostname: gitea
         Icon name: computer-vm
           Chassis: vm
        Machine ID: e97ba38a8492475f83866069702c4cb8
           Boot ID: facce7ece11c4a4ea9b5be685c171c36
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.2.2.el7.x86_64
      Architecture: x86-64
[vagrant@gitea ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:8a:70:6a brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83221sec preferred_lft 83221sec
    inet6 fe80::a00:27ff:fe8a:706a/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:19:48:79 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.150/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe19:4879/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@gitea ~]$ 
~~~

## 下载软件包

参考 **[gitea bin][gitea_dl]** 下载对应系统版本的软件包

所有下载均包括 SQLite, MySQL 和 PostgreSQL 的支持，同时所有驱动均已嵌入到可执行程序中

~~~
[vagrant@gitea ~]$ wget https://dl.gitea.io/gitea/1.4.1/gitea-1.4.1-linux-amd64
--2018-05-20 17:52:22--  https://dl.gitea.io/gitea/1.4.1/gitea-1.4.1-linux-amd64
Resolving dl.gitea.io (dl.gitea.io)... 104.27.143.155, 104.27.142.155, 2400:cb00:2048:1::681b:8e9b, ...
Connecting to dl.gitea.io (dl.gitea.io)|104.27.143.155|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 53663640 (51M) [application/octet-stream]
Saving to: ‘gitea-1.4.1-linux-amd64’

100%[======================================>] 53,663,640  2.82MB/s   in 22s    

2018-05-20 17:52:45 (2.36 MB/s) - ‘gitea-1.4.1-linux-amd64’ saved [53663640/53663640]

[vagrant@gitea ~]$ sha256sum gitea-1.4.1-linux-amd64 
d8cfa0d39da70497f1f75e519e4fee33e5ee7c0de88919bdfe46a8b0d38af851  gitea-1.4.1-linux-amd64
[vagrant@gitea ~]$ 
~~~

下载包的 hash 可以参考 **[gitea-1.4.1-linux-amd64.sha256][gitea_hash]**

具体值随版本不同而不同，这个方式可以简单而有效地确认下载包的一致性

## 添加权限

~~~
[vagrant@gitea ~]$ ll gitea-1.4.1-linux-amd64 
-rw-rw-r--. 1 vagrant vagrant 53663640 5月   3 06:01 gitea-1.4.1-linux-amd64
[vagrant@gitea ~]$ chmod +x gitea-1.4.1-linux-amd64 
[vagrant@gitea ~]$ ll gitea-1.4.1-linux-amd64 
-rwxrwxr-x. 1 vagrant vagrant 53663640 5月   3 06:01 gitea-1.4.1-linux-amd64
[vagrant@gitea ~]$ 
~~~

## 依赖

gitea 的运行是依赖 git 的

如果事先没有安装 git 会有如下报错

~~~
[vagrant@gitea ~]$ ./gitea-1.4.1-linux-amd64 web
panic: Git version missing: exec: "git": executable file not found in $PATH

goroutine 1 [running]:
code.gitea.io/gitea/vendor/code.gitea.io/git.init.0()
	/srv/app/src/code.gitea.io/gitea/vendor/code.gitea.io/git/git.go:76 +0x1d1
[vagrant@gitea ~]$ 
~~~

解决办法是安装 git 

~~~
[vagrant@gitea ~]$ sudo yum install git.x86_64
Loaded plugins: fastestmirror
base                                                     | 3.6 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
updates/7/x86_64/primary_db                                | 1.2 MB   00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package git.x86_64 0:1.8.3.1-13.el7 will be installed
--> Processing Dependency: perl-Git = 1.8.3.1-13.el7 for package: git-1.8.3.1-13.el7.x86_64
--> Processing Dependency: perl(Term::ReadKey) for package: git-1.8.3.1-13.el7.x86_64
--> Processing Dependency: perl(Git) for package: git-1.8.3.1-13.el7.x86_64
--> Processing Dependency: perl(Error) for package: git-1.8.3.1-13.el7.x86_64
--> Processing Dependency: libgnome-keyring.so.0()(64bit) for package: git-1.8.3.1-13.el7.x86_64
--> Running transaction check
---> Package libgnome-keyring.x86_64 0:3.12.0-1.el7 will be installed
---> Package perl-Error.noarch 1:0.17020-2.el7 will be installed
---> Package perl-Git.noarch 0:1.8.3.1-13.el7 will be installed
---> Package perl-TermReadKey.x86_64 0:2.30-20.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch          Version                Repository   Size
================================================================================
Installing:
 git                     x86_64        1.8.3.1-13.el7         base        4.4 M
Installing for dependencies:
 libgnome-keyring        x86_64        3.12.0-1.el7           base        109 k
 perl-Error              noarch        1:0.17020-2.el7        base         32 k
 perl-Git                noarch        1.8.3.1-13.el7         base         54 k
 perl-TermReadKey        x86_64        2.30-20.el7            base         31 k

Transaction Summary
================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 4.6 M
Installed size: 23 M
Is this ok [y/d/N]: y
Downloading packages:
(1/5): libgnome-keyring-3.12.0-1.el7.x86_64.rpm            | 109 kB   00:00     
(2/5): perl-Error-0.17020-2.el7.noarch.rpm                 |  32 kB   00:00     
(3/5): perl-Git-1.8.3.1-13.el7.noarch.rpm                  |  54 kB   00:00     
(4/5): perl-TermReadKey-2.30-20.el7.x86_64.rpm             |  31 kB   00:00     
(5/5): git-1.8.3.1-13.el7.x86_64.rpm                       | 4.4 MB   00:02     
--------------------------------------------------------------------------------
Total                                              1.8 MB/s | 4.6 MB  00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:perl-Error-0.17020-2.el7.noarch                            1/5 
  Installing : libgnome-keyring-3.12.0-1.el7.x86_64                         2/5 
  Installing : perl-TermReadKey-2.30-20.el7.x86_64                          3/5 
  Installing : perl-Git-1.8.3.1-13.el7.noarch                               4/5 
  Installing : git-1.8.3.1-13.el7.x86_64                                    5/5 
  Verifying  : git-1.8.3.1-13.el7.x86_64                                    1/5 
  Verifying  : perl-TermReadKey-2.30-20.el7.x86_64                          2/5 
  Verifying  : libgnome-keyring-3.12.0-1.el7.x86_64                         3/5 
  Verifying  : 1:perl-Error-0.17020-2.el7.noarch                            4/5 
  Verifying  : perl-Git-1.8.3.1-13.el7.noarch                               5/5 

Installed:
  git.x86_64 0:1.8.3.1-13.el7                                                   

Dependency Installed:
  libgnome-keyring.x86_64 0:3.12.0-1.el7  perl-Error.noarch 1:0.17020-2.el7     
  perl-Git.noarch 0:1.8.3.1-13.el7        perl-TermReadKey.x86_64 0:2.30-20.el7 

Complete!
[vagrant@gitea ~]$ echo $?
0
[vagrant@gitea ~]$ 
~~~

## 测试运行

~~~
[vagrant@gitea ~]$ ls -l gitea-1.4.1-linux-amd64 
-rwxrwxr-x. 1 vagrant vagrant 53663640 5月   3 06:01 gitea-1.4.1-linux-amd64
[vagrant@gitea ~]$ ./gitea-1.4.1-linux-amd64  --help 
NAME:
   Gitea - A painless self-hosted Git service

USAGE:
   gitea-1.4.1-linux-amd64 [global options] command [command options] [arguments...]

VERSION:
   1.4.1 built with: bindata, sqlite

DESCRIPTION:
   By default, gitea will start serving using the webserver with no
arguments - which can alternatively be run by running the subcommand web.

COMMANDS:
     web      Start Gitea web server
     serv     This command should only be called by SSH shell
     hook     Delegate commands to corresponding Git hooks
     dump     Dump Gitea files and database
     cert     Generate self-signed certificate
     admin    Command line interface to perform common administrative operations
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
[vagrant@gitea ~]$ 
[vagrant@gitea ~]$ ./gitea-1.4.1-linux-amd64 web
2018/05/20 17:59:43 [W] Custom config '/home/vagrant/custom/conf/app.ini' not found, ignore this if you're running first time
2018/05/20 17:59:43 [T] AppPath: /home/vagrant/gitea-1.4.1-linux-amd64
2018/05/20 17:59:43 [T] AppWorkPath: /home/vagrant
2018/05/20 17:59:43 [T] Custom path: /home/vagrant/custom
2018/05/20 17:59:43 [T] Log path: /home/vagrant/log
2018/05/20 17:59:43 [I] Gitea v1.4.1 built with: bindata, sqlite
2018/05/20 17:59:43 [I] Log Mode: Console(Info)
2018/05/20 17:59:43 [I] XORM Log Mode: Console(Info)
2018/05/20 17:59:43 [I] Cache Service Enabled
2018/05/20 17:59:43 [I] Session Service Enabled
2018/05/20 17:59:43 [I] SQLite3 Supported
2018/05/20 17:59:43 [I] Run Mode: Development
2018/05/20 17:59:43 Serving [::]:3000 with pid 2694
2018/05/20 17:59:43 [I] Listen: http://0.0.0.0:3000
...
...
...
~~~

会在本地开放一个 3000 的端口以提供服务

~~~
[vagrant@gitea ~]$ netstat  -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp        0      0 10.0.2.15:22            10.0.2.2:35408          ESTABLISHED
tcp        0      0 10.0.2.15:22            10.0.2.2:57710          ESTABLISHED
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 :::3000                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
[vagrant@gitea ~]$ 
~~~

## 进行访问

首先要确认一下防火墙对 3000 端口进行了放行，排除网络阻断

访问服务地址 **`http://192.168.56.150:3000/`** 可以看到直接跳转到了安装界面

![gitea](/assets/img/gitea/gitea01.png)

![gitea](/assets/img/gitea/gitea02.png)

## 进行配置

这里有几种数据库可选，因为我懒，这边没有安装 mysql，所以我选择一个 SQLite3 作为后端数据库

(所有这里面列举的数据，gitea 都事先安装好了驱动，所以不用操心数据库驱动的问题)

![gitea](/assets/img/gitea/gitea03.png)

下面可以填写管理员的创建信息，也可以不填，因为第一个注册的用户会自动获取管理员的权限

![gitea](/assets/img/gitea/gitea04.png)

点击完 **[Install Gitea]** 后，过一会儿再登录，就能见到 gitea 主界面

![gitea](/assets/img/gitea/gitea05.png)

登录界面

![gitea](/assets/img/gitea/gitea06.png)

注册界面

![gitea](/assets/img/gitea/gitea07.png)

使用注册的用户登录

![gitea](/assets/img/gitea/gitea08.png)

登录成功后的管理界面

![gitea](/assets/img/gitea/gitea09.png)

到此 Gitea 的构建工作就已经完成了

如果需要管理大量的数据，可以考虑将 SQLite3 替换成 mysql 或 postgresql

---

因为 Gitea 已经将软件包打包成了各平台下的 bin 包，所以安装过程其实就是下载过程

而运行和配置都相对简单，所以总体来讲，单纯的构建过程还是十分容易的

后面有空再讲讲使用技巧

# 总结


* TOC
{:toc}

---

[gitea]:https://gitea.io/en-US/
[gitea_install]:https://docs.gitea.io/zh-cn/install-from-binary/
[gitea_dl]:https://dl.gitea.io/gitea/
[gitea_hash]:https://dl.gitea.io/gitea/1.4.1/gitea-1.4.1-linux-amd64.sha256


