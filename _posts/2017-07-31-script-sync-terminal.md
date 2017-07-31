---
layout:  post
title:  使用 script 实现 terminal 同屏 
author:  wilmosfang
tags:  linux script
categories:  linux 
wc: 210 603 5616
excerpt:  Linux 中通过 script 实现 terminal 同屏 
comments: true
---


# 前言


在 linux 系统中，有一个神器 **`script`** 命令，可以用来记录所有 CLI 终端的交互记录

>script  makes  a  typescript  of everything printed on your terminal.  It is useful for students who need a hardcopy record of an interactive session as proof of an assignment

~~~
[root@56-201 ~]# script --help 

Usage:
 script [options] [file]

Options:
 -a, --append            append the output
 -c, --command <command> run command rather than interactive shell
 -e, --return            return exit code of the child process
 -f, --flush             run flush after each write
     --force             use output file even when it is a link
 -q, --quiet             be quiet
 -t, --timing[=<file>]   output timing data to stderr (or to FILE)
 -V, --version           output version information and exit
 -h, --help              display this help and exit

[root@56-201 ~]# 
~~~

> **Tip:**  script 是一个可以记录终端行为的命令，生成的日志可以重现当时的交互，日志是以 ASCII 的格式存档的

接合 **`scriptreplay`** 可以重放当时的交互

~~~
[root@56-201 ~]# scriptreplay --help 

Usage:
 scriptreplay [-t] timingfile [typescript] [divisor]

Options:
 -t, --timing <file>     script timing output file
 -s, --typescript <file> script terminal session output file
 -d, --divisor <num>     speed up or slow down execution with time divisor
 -V, --version           output version information and exit
 -h, --help              display this help and exit

[root@56-201 ~]#
~~~

这里不准备演示 **`scriptreplay`** 的使用方法，其实也很简单

这里接合重定向实现文本终端的同屏，这个可以用于同时给很多人演示命令效果，用于教学演示场景

---

# 概要

* TOC
{:toc}

---


## 系统环境


~~~
[root@56-201 ~]# hostnamectl 
   Static hostname: 56-201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 8650880b99aa4e62a56b447efeb0684e
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@56-201 ~]# uname -a 
Linux 56-201 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@56-201 ~]# ip  a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0e:38:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 65696sec preferred_lft 65696sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:bb:5d:54 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.201/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febb:5d54/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@56-201 ~]# 
~~~

---

## 目标

* 通过重定向实现文本终端同屏
* 通过跟踪日志实现文本终端同屏 

---

## 创建日志

~~~
[root@56-201 jail]# ll
total 0
drwxr-xr-x. 2 root root 30 7月  30 19:04 bin
drwxr-xr-x. 2 root root 90 7月  30 19:03 lib64
[root@56-201 jail]# script -f abc
Script started, file is abc
[root@56-201 jail]# ll abc
-rw-r--r--. 1 root root 135 7月  31 15:32 abc
[root@56-201 jail]#
~~~

**`-f`** 就是每次写操作过后都进行刷新，目的是进行实时同步，让其它结点可以读到最新的内容


---

## 获取终端

~~~
wilmos@Nothing:~$ ssh jman@192.168.56.201
jman@192.168.56.201's password: 
Last login: Mon Jul 31 00:42:17 2017
Could not chdir to home directory /home/jman: No such file or directory
-bash-4.2$ 
-bash-4.2$ 
-bash-4.2$ 
~~~

登录以获取终端

---

## 服务端重定向输出

~~~
[root@56-201 jail]# tail -f abc > /dev/pts/2 & 
[1] 8350
[root@56-201 jail]# 
[root@56-201 jail]# 
~~~

之后从客户端 terminal 中看到的就是服务端中的内容，这个方法客户端只管登录，不用做什么别的操作


![script_pipe.gif](/images/script/script_pipe.gif)

---

## 客户端从本地跟踪日志


~~~
-bash-4.2$ 
-bash-4.2$ 
-bash-4.2$ /bin/tail -f abc
[root@56-201 jail]# 
[root@56-201 jail]# 
[root@56-201 jail]# 
[root@56-201 jail]# 
[root@56-201 jail]# 
[root@56-201 jail]# 
[root@56-201 jail]# 
[root@56-201 jail]# ls
abc  bin  lib64
[root@56-201 jail]# 
~~~ 

之后从客户端 terminal 中看到的就是服务端中的内容，这个方法客户端登录后要主动使用 tail 命令，服务端不用做什么别的操作


![script_pipe.gif](/images/script/script_tail.gif)


两种同屏效果，是不是很绚，又多了一招可以用来装B了，开不开心，惊不惊喜


---

# 命令汇总

* **`script --help`**
* **`scriptreplay --help`**
* **`script -f abc`**
* **`tail -f abc > /dev/pts/2 &`**
