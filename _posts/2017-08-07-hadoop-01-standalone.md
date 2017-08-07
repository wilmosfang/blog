---
layout:  post
title:  Hadoop 在 Centos7 下的单机布署(一). Standalone Operation
author:  wilmosfang
tags:  hadoop cluster
categories:  hadoop
wc: 1262 6603 72655
excerpt:  Centos7 下布署 Standalone 模式的 Hadoop
comments: true
---


# 前言


**[Apache Hadoop][hadoop]** 是一个专注于可靠，弹性，分布式计算框架的开源软件项目

>The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

主要包含以下几个模块：

* **Hadoop Common** : 支持其它 **hadoop** 模块的通用工具
* **Hadoop Distributed File System** : 简称 **HDFS** , 给应用数据提供高吞吐性能的分布式文件系统
* **Hadoop YARN** : 工作调度与集群资源管理的框架
* **Hadoop MapReduce** : 大数据集的并行处理系统

Hadoop 生态圈中的其它项目可以参考 **[Hadoop-related projects][hadoop]**

> **Tip:**  当前的最新稳定版为 **Hadoop Release 2.8.1** 发布于 **08 June, 2017**

这里根据官方的文档给出最新版 hadoop  在 Centos7 下的单机布署方案，详细可以参考 **[Setting up a Single Node Cluster][singlecluster]**

此文章借鉴了 **[Hadoop Wiki][hadoop_doc]** 中的部分内容和  **[Running Hadoop on Ubuntu Linux (Single-Node Cluster)][install_hadoop_on_ubuntu]** 中部分操作


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
           Boot ID: 8cfd2a8aec4f4235b4776f7cd2fdfdb1
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
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84190sec preferred_lft 84190sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.207/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed3:ece7/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]# uname -a 
Linux much 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@much ~]# 
~~~

---

## 目标

* 构建一个 hadoop 的单点伪集群
* hdfs 的基本操作

---

## 基础依赖

* Java 
* ssh

由于 Hadoop 是使用 Java 语言开发出来的项目，所以 Java 运行环境是必要的, Hadoop 支持的 Java 版本可以参考 **[Hadoop Java Versions][HadoopJavaVersions]**

之所以需要 sshd , 是因为 Hadoop 是通过 ssh 执行脚本来管理远程节点

---

## 安装 Java

~~~
[root@much ~]# rpm  -qa | grep epel 
[root@much ~]# yum  list all | grep -i epel 
epel-release.noarch                        7-9                         extras   
[root@much ~]# yum  install epel-release.noarch 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.yun-idc.com
 * c7-media: 
 * extras: mirrors.163.com
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-9 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================
 Package                  Arch               Version           Repository          Size
========================================================================================
Installing:
 epel-release             noarch             7-9               extras              14 k

Transaction Summary
========================================================================================
Install  1 Package

Total download size: 14 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-9.noarch.rpm                                      |  14 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-9.noarch                                              1/1 
  Verifying  : epel-release-7-9.noarch                                              1/1 

Installed:
  epel-release.noarch 0:7-9                                                             

Complete!
[root@much ~]# 
[root@much ~]# rpm  -qa | grep epel 
epel-release-7-9.noarch
[root@much ~]# 
[root@much ~]# yum list all | grep -i java | grep -i  jdk
java-1.7.0-openjdk.x86_64               1:1.7.0.141-2.6.10.1.el7_3     @updates 
java-1.7.0-openjdk-headless.x86_64      1:1.7.0.141-2.6.10.1.el7_3     @updates 
java-1.8.0-openjdk.x86_64               1:1.8.0.131-3.b12.el7_3        @updates 
java-1.8.0-openjdk-headless.x86_64      1:1.8.0.131-3.b12.el7_3        @updates 
java-1.6.0-openjdk.x86_64               1:1.6.0.41-1.13.13.1.el7_3     updates  
java-1.6.0-openjdk-demo.x86_64          1:1.6.0.41-1.13.13.1.el7_3     updates  
java-1.6.0-openjdk-devel.x86_64         1:1.6.0.41-1.13.13.1.el7_3     updates  
java-1.6.0-openjdk-javadoc.x86_64       1:1.6.0.41-1.13.13.1.el7_3     updates  
java-1.6.0-openjdk-src.x86_64           1:1.6.0.41-1.13.13.1.el7_3     updates  
java-1.7.0-openjdk-accessibility.x86_64 1:1.7.0.141-2.6.10.1.el7_3     updates  
java-1.7.0-openjdk-demo.x86_64          1:1.7.0.141-2.6.10.1.el7_3     updates  
java-1.7.0-openjdk-devel.x86_64         1:1.7.0.141-2.6.10.1.el7_3     updates  
java-1.7.0-openjdk-javadoc.noarch       1:1.7.0.141-2.6.10.1.el7_3     updates  
java-1.7.0-openjdk-src.x86_64           1:1.7.0.141-2.6.10.1.el7_3     updates  
java-1.8.0-openjdk.i686                 1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk.x86_64               1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-accessibility.x86_64 1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-accessibility-debug.x86_64
java-1.8.0-openjdk-debug.i686           1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-debug.x86_64         1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-debuginfo.i686       1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-debuginfo.x86_64     1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-demo.x86_64          1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-demo-debug.x86_64    1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-devel.i686           1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-devel.x86_64         1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-devel-debug.i686     1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-devel-debug.x86_64   1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-headless.i686        1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-headless.x86_64      1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-headless-debug.i686  1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-headless-debug.x86_64
java-1.8.0-openjdk-javadoc.noarch       1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-javadoc-debug.noarch 1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-javadoc-zip.noarch   1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-javadoc-zip-debug.noarch
java-1.8.0-openjdk-src.x86_64           1:1.8.0.141-1.b16.el7_3        updates  
java-1.8.0-openjdk-src-debug.x86_64     1:1.8.0.141-1.b16.el7_3        updates  
ldapjdk-javadoc.noarch                  4.18-16.el7_3                  updates  
[root@much ~]# yum install java-1.8.0-openjdk.x86_64  java-1.8.0-openjdk-devel.x86_64  java-1.8.0-openjdk-headless.x86_64  
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.yun-idc.com
 * c7-media: 
 * epel: mirrors.ustc.edu.cn
 * extras: mirrors.163.com
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.131-3.b12.el7_3 will be updated
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.141-1.b16.el7_3 will be an update
---> Package java-1.8.0-openjdk-devel.x86_64 1:1.8.0.141-1.b16.el7_3 will be installed
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.131-3.b12.el7_3 will be updated
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.141-1.b16.el7_3 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================
 Package                        Arch      Version                      Repository  Size
========================================================================================
Installing:
 java-1.8.0-openjdk-devel       x86_64    1:1.8.0.141-1.b16.el7_3      updates    9.7 M
Updating:
 java-1.8.0-openjdk             x86_64    1:1.8.0.141-1.b16.el7_3      updates    234 k
 java-1.8.0-openjdk-headless    x86_64    1:1.8.0.141-1.b16.el7_3      updates     31 M

Transaction Summary
========================================================================================
Install  1 Package
Upgrade  2 Packages

Total download size: 41 M
Is this ok [y/d/N]: y
Downloading packages:
updates/7/x86_64/prestodelta                                     | 954 kB  00:00:00     
Delta RPMs reduced 234 k of updates to 58 k (75% saved)
(1/3): java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3_1.8.0.141-1.b16. |  58 kB  00:00:00     
(2/3): java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64.rpm | 9.7 MB  00:00:03     
(3/3): java-1.8.0-openjdk-headless-1.8.0.141-1.b16.el7_3.x86_64. |  31 MB  00:00:03     
----------------------------------------------------------------------------------------
Total                                                       10 MB/s |  41 MB  00:04     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:java-1.8.0-openjdk-headless-1.8.0.141-1.b16.el7_3.x86_64           1/5 
warning: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/lib/security/java.security created as /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/lib/security/java.security.rpmnew
  Updating   : 1:java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64                    2/5 
  Installing : 1:java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64              3/5 
  Cleanup    : 1:java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64                    4/5 
  Cleanup    : 1:java-1.8.0-openjdk-headless-1.8.0.131-3.b12.el7_3.x86_64           5/5 
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.141-1.b16.el7_3.x86_64           1/5 
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64                    2/5 
  Verifying  : 1:java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64              3/5 
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64                    4/5 
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.131-3.b12.el7_3.x86_64           5/5 

Installed:
  java-1.8.0-openjdk-devel.x86_64 1:1.8.0.141-1.b16.el7_3                               

Updated:
  java-1.8.0-openjdk.x86_64 1:1.8.0.141-1.b16.el7_3                                     
  java-1.8.0-openjdk-headless.x86_64 1:1.8.0.141-1.b16.el7_3                            

Complete!
[root@much ~]# 
~~~


检查 **JAVA_HOME**

~~~
[root@much ~]# java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-b16)
OpenJDK 64-Bit Server VM (build 25.141-b16, mixed mode)
[root@much ~]# which java 
/usr/bin/java
[root@much ~]# ll /usr/bin/java
lrwxrwxrwx. 1 root root 22 8月   6 23:55 /usr/bin/java -> /etc/alternatives/java
[root@much ~]# ll /etc/alternatives/java
lrwxrwxrwx. 1 root root 73 8月   6 23:55 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/bin/java
[root@much ~]# ll /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/bin/java
-rwxr-xr-x. 1 root root 7336 7月  21 06:37 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/bin/java
[root@much ~]# ll /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/
total 4
drwxr-xr-x. 2 root root 4096 8月   6 23:55 bin
drwxr-xr-x. 3 root root  132 8月   6 23:55 include
drwxr-xr-x. 4 root root   28 8月   6 23:55 jre
drwxr-xr-x. 3 root root  144 8月   6 23:55 lib
drwxr-xr-x. 2 root root  204 8月   6 23:55 tapset
[root@much ~]# ll /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/
total 420
-rwxr-xr-x. 1 root root   7400 7月  21 06:37 appletviewer
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 extcheck
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 idlj
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jar
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 jarsigner
-rwxr-xr-x. 1 root root   7336 7月  21 06:37 java
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 javac
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 javadoc
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 javah
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 javap
-rwxr-xr-x. 1 root root   2806 7月  21 06:01 java-rmi.cgi
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jcmd
-rwxr-xr-x. 1 root root   7416 7月  21 06:37 jconsole
-rwxr-xr-x. 1 root root   7400 7月  21 06:37 jdb
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jdeps
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jhat
-rwxr-xr-x. 1 root root   7448 7月  21 06:37 jinfo
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jjs
-rwxr-xr-x. 1 root root   7448 7月  21 06:37 jmap
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jps
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 jrunscript
-rwxr-xr-x. 1 root root   7408 7月  21 06:37 jsadebugd
-rwxr-xr-x. 1 root root   7456 7月  21 06:37 jstack
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 jstat
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 jstatd
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 keytool
-rwxr-xr-x. 1 root root   7400 7月  21 06:37 native2ascii
-rwxr-xr-x. 1 root root   7456 7月  21 06:37 orbd
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 pack200
-rwxr-xr-x. 1 root root   7400 7月  21 06:37 policytool
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 rmic
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 rmid
-rwxr-xr-x. 1 root root   7400 7月  21 06:37 rmiregistry
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 schemagen
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 serialver
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 servertool
-rwxr-xr-x. 1 root root   7464 7月  21 06:37 tnameserv
-rwxr-xr-x. 1 root root 103424 7月  21 06:37 unpack200
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 wsgen
-rwxr-xr-x. 1 root root   7392 7月  21 06:37 wsimport
-rwxr-xr-x. 1 root root   7384 7月  21 06:37 xjc
[root@much ~]#
~~~

---

## 确保 ssh rsync 已经安装

~~~
[root@much ~]# yum install openssh rsync 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.yun-idc.com
 * c7-media: 
 * epel: mirrors.ustc.edu.cn
 * extras: mirrors.163.com
 * updates: centos.ustc.edu.cn
Package openssh-6.6.1p1-35.el7_3.x86_64 already installed and latest version
Package rsync-3.0.9-17.el7.x86_64 already installed and latest version
Nothing to do
[root@much ~]#
~~~

---

## 关闭 selinux

~~~
[root@much ~]# getenforce 
Enforcing
[root@much ~]# setenforce  0 
[root@much ~]# getenforce 
Permissive
[root@much ~]#
~~~

---

## 创建 hadoop 用户　

~~~
[root@much ~]# useradd hadoop 
[root@much ~]# passwd hadoop
Changing password for user hadoop.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@much ~]# 
~~~

---

## 配置用户免密码登录

~~~
[root@much ~]# su - hadoop 
[hadoop@much ~]$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Created directory '/home/hadoop/.ssh'.
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
cb:02:3a:db:ce:eb:e3:d0:72:be:54:84:8f:cb:d7:c1 hadoop@much
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|     .           |
|    . .          |
|     + .         |
|    o o E        |
|   + + o o       |
|  = * o +        |
|   @.. .         |
|  .+X+           |
+-----------------+
[hadoop@much ~]$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
[hadoop@much ~]$ chmod 600 $HOME/.ssh/authorized_keys
[hadoop@much ~]$ ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is 0c:52:20:0a:00:e3:1a:5d:c6:fc:79:b3:e8:6e:d6:f1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
Last login: Mon Aug  7 00:04:13 2017
[hadoop@much ~]$ exit
logout
Connection to localhost closed.
[hadoop@much ~]$ ssh localhost
Last login: Mon Aug  7 00:05:48 2017 from localhost
[hadoop@much ~]$ exit
logout
Connection to localhost closed.
[hadoop@much ~]$ 
~~~

> **Tip:**  配置证书登录的过程中，要注意 **authorized_keys** 文件的权限，如果没有达到预期效果可以通过 **`ssh -vvv localhost`** 来进行 debug

~~~
[hadoop@much ~]$ ssh -vvv localhost
OpenSSH_6.6.1, OpenSSL 1.0.1e-fips 11 Feb 2013
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 56: Applying options for *
debug2: ssh_connect: needpriv 0
debug1: Connecting to localhost [::1] port 22.
debug1: Connection established.
debug3: Incorrect RSA1 identifier
debug3: Could not load "/home/hadoop/.ssh/id_rsa" as a RSA1 public key
debug1: identity file /home/hadoop/.ssh/id_rsa type 1
debug1: identity file /home/hadoop/.ssh/id_rsa-cert type -1
debug1: identity file /home/hadoop/.ssh/id_dsa type -1
debug1: identity file /home/hadoop/.ssh/id_dsa-cert type -1
debug1: identity file /home/hadoop/.ssh/id_ecdsa type -1
debug1: identity file /home/hadoop/.ssh/id_ecdsa-cert type -1
debug1: identity file /home/hadoop/.ssh/id_ed25519 type -1
debug1: identity file /home/hadoop/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.6.1
debug1: Remote protocol version 2.0, remote software version OpenSSH_6.6.1
debug1: match: OpenSSH_6.6.1 pat OpenSSH_6.6.1* compat 0x04000000
debug2: fd 3 setting O_NONBLOCK
debug3: load_hostkeys: loading entries for host "localhost" from file "/home/hadoop/.ssh/known_hosts"
debug3: load_hostkeys: found key type ECDSA in file /home/hadoop/.ssh/known_hosts:1
debug3: load_hostkeys: loaded 1 keys
debug3: order_hostkeyalgs: prefer hostkeyalgs: ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug2: kex_parse_kexinit: curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
debug2: kex_parse_kexinit: ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-ed25519-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ssh-dss-cert-v01@openssh.com,ssh-rsa-cert-v00@openssh.com,ssh-dss-cert-v00@openssh.com,ssh-ed25519,ssh-rsa,ssh-dss
debug2: kex_parse_kexinit: aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-cbc,3des-cbc,blowfish-cbc,cast128-cbc,aes192-cbc,aes256-cbc,arcfour,rijndael-cbc@lysator.liu.se
debug2: kex_parse_kexinit: aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-cbc,3des-cbc,blowfish-cbc,cast128-cbc,aes192-cbc,aes256-cbc,arcfour,rijndael-cbc@lysator.liu.se
debug2: kex_parse_kexinit: hmac-md5-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-ripemd160-etm@openssh.com,hmac-sha1-96-etm@openssh.com,hmac-md5-96-etm@openssh.com,hmac-md5,hmac-sha1,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-ripemd160,hmac-ripemd160@openssh.com,hmac-sha1-96,hmac-md5-96
debug2: kex_parse_kexinit: hmac-md5-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-ripemd160-etm@openssh.com,hmac-sha1-96-etm@openssh.com,hmac-md5-96-etm@openssh.com,hmac-md5,hmac-sha1,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-ripemd160,hmac-ripemd160@openssh.com,hmac-sha1-96,hmac-md5-96
debug2: kex_parse_kexinit: none,zlib@openssh.com,zlib
debug2: kex_parse_kexinit: none,zlib@openssh.com,zlib
debug2: kex_parse_kexinit: 
debug2: kex_parse_kexinit: 
debug2: kex_parse_kexinit: first_kex_follows 0 
debug2: kex_parse_kexinit: reserved 0 
debug2: kex_parse_kexinit: curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
debug2: kex_parse_kexinit: ssh-rsa,ecdsa-sha2-nistp256,ssh-ed25519
debug2: kex_parse_kexinit: aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-cbc,3des-cbc,blowfish-cbc,cast128-cbc,aes192-cbc,aes256-cbc,arcfour,rijndael-cbc@lysator.liu.se
debug2: kex_parse_kexinit: aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-cbc,3des-cbc,blowfish-cbc,cast128-cbc,aes192-cbc,aes256-cbc,arcfour,rijndael-cbc@lysator.liu.se
debug2: kex_parse_kexinit: hmac-md5-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-ripemd160-etm@openssh.com,hmac-sha1-96-etm@openssh.com,hmac-md5-96-etm@openssh.com,hmac-md5,hmac-sha1,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-ripemd160,hmac-ripemd160@openssh.com,hmac-sha1-96,hmac-md5-96
debug2: kex_parse_kexinit: hmac-md5-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-ripemd160-etm@openssh.com,hmac-sha1-96-etm@openssh.com,hmac-md5-96-etm@openssh.com,hmac-md5,hmac-sha1,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-ripemd160,hmac-ripemd160@openssh.com,hmac-sha1-96,hmac-md5-96
debug2: kex_parse_kexinit: none,zlib@openssh.com
debug2: kex_parse_kexinit: none,zlib@openssh.com
debug2: kex_parse_kexinit: 
debug2: kex_parse_kexinit: 
debug2: kex_parse_kexinit: first_kex_follows 0 
debug2: kex_parse_kexinit: reserved 0 
debug2: mac_setup: setup hmac-md5-etm@openssh.com
debug1: kex: server->client aes128-ctr hmac-md5-etm@openssh.com none
debug2: mac_setup: setup hmac-md5-etm@openssh.com
debug1: kex: client->server aes128-ctr hmac-md5-etm@openssh.com none
debug1: kex: curve25519-sha256@libssh.org need=16 dh_need=16
debug1: kex: curve25519-sha256@libssh.org need=16 dh_need=16
debug1: sending SSH2_MSG_KEX_ECDH_INIT
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: Server host key: ECDSA 0c:52:20:0a:00:e3:1a:5d:c6:fc:79:b3:e8:6e:d6:f1
debug3: load_hostkeys: loading entries for host "localhost" from file "/home/hadoop/.ssh/known_hosts"
debug3: load_hostkeys: found key type ECDSA in file /home/hadoop/.ssh/known_hosts:1
debug3: load_hostkeys: loaded 1 keys
debug1: Host 'localhost' is known and matches the ECDSA host key.
debug1: Found key in /home/hadoop/.ssh/known_hosts:1
debug1: ssh_ecdsa_verify: signature correct
debug2: kex_derive_keys
debug2: set_newkeys: mode 1
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug2: set_newkeys: mode 0
debug1: SSH2_MSG_NEWKEYS received
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug2: service_accept: ssh-userauth
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug2: key: /home/hadoop/.ssh/id_rsa (0x7f284841f2c0),
debug2: key: /home/hadoop/.ssh/id_dsa ((nil)),
debug2: key: /home/hadoop/.ssh/id_ecdsa ((nil)),
debug2: key: /home/hadoop/.ssh/id_ed25519 ((nil)),
debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
debug3: start over, passed a different list publickey,gssapi-keyex,gssapi-with-mic,password
debug3: preferred gssapi-keyex,gssapi-with-mic,publickey,keyboard-interactive,password
debug3: authmethod_lookup gssapi-keyex
debug3: remaining preferred: gssapi-with-mic,publickey,keyboard-interactive,password
debug3: authmethod_is_enabled gssapi-keyex
debug1: Next authentication method: gssapi-keyex
debug1: No valid Key exchange context
debug2: we did not send a packet, disable method
debug3: authmethod_lookup gssapi-with-mic
debug3: remaining preferred: publickey,keyboard-interactive,password
debug3: authmethod_is_enabled gssapi-with-mic
debug1: Next authentication method: gssapi-with-mic
debug1: Unspecified GSS failure.  Minor code may provide more information
No Kerberos credentials available (default cache: KEYRING:persistent:1001)

debug1: Unspecified GSS failure.  Minor code may provide more information
No Kerberos credentials available (default cache: KEYRING:persistent:1001)

debug2: we did not send a packet, disable method
debug3: authmethod_lookup publickey
debug3: remaining preferred: keyboard-interactive,password
debug3: authmethod_is_enabled publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /home/hadoop/.ssh/id_rsa
debug3: send_pubkey_test
debug2: we sent a publickey packet, wait for reply
debug1: Server accepts key: pkalg ssh-rsa blen 279
debug2: input_userauth_pk_ok: fp cb:02:3a:db:ce:eb:e3:d0:72:be:54:84:8f:cb:d7:c1
debug3: sign_and_send_pubkey: RSA cb:02:3a:db:ce:eb:e3:d0:72:be:54:84:8f:cb:d7:c1
debug1: key_parse_private2: missing begin marker
debug1: read PEM private key done: type RSA
debug1: Authentication succeeded (publickey).
Authenticated to localhost ([::1]:22).
debug1: channel 0: new [client-session]
debug3: ssh_session2_open: channel_new: 0
debug2: channel 0: send open
debug1: Requesting no-more-sessions@openssh.com
debug1: Entering interactive session.
debug2: callback start
debug2: fd 3 setting TCP_NODELAY
debug3: packet_set_tos: set IPV6_TCLASS 0x10
debug2: client_session2_setup: id 0
debug2: channel 0: request pty-req confirm 1
debug1: Sending environment.
debug3: Ignored env XDG_SESSION_ID
debug3: Ignored env HOSTNAME
debug3: Ignored env SHELL
debug3: Ignored env TERM
debug3: Ignored env HISTSIZE
debug3: Ignored env USER
debug3: Ignored env LS_COLORS
debug3: Ignored env MAIL
debug3: Ignored env PATH
debug3: Ignored env PWD
debug1: Sending env LANG = en_US.UTF-8
debug2: channel 0: request env confirm 0
debug3: Ignored env HISTCONTROL
debug3: Ignored env SHLVL
debug3: Ignored env HOME
debug3: Ignored env LOGNAME
debug3: Ignored env LESSOPEN
debug3: Ignored env _
debug2: channel 0: request shell confirm 1
debug2: callback done
debug2: channel 0: open confirm rwindow 0 rmax 32768
debug2: channel_input_status_confirm: type 99 id 0
debug2: PTY allocation request accepted on channel 0
debug2: channel 0: rcvd adjust 2097152
debug2: channel_input_status_confirm: type 99 id 0
debug2: shell request accepted on channel 0
Last login: Mon Aug  7 00:05:55 2017 from localhost
[hadoop@much ~]$ 
~~~

---

## 关闭 IPv6

~~~
[root@much ~]# sysctl -a | grep -i net.ipv6.conf.all.disable_ipv6
net.ipv6.conf.all.disable_ipv6 = 0
[root@much ~]# cat /etc/sysctl.conf
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
[root@much ~]#
[root@much ~]# vim  /etc/sysctl.conf
[root@much ~]# sysctl -p 
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
[root@much ~]# sysctl -a | grep -i net.ipv6.conf.all.disable_ipv6
net.ipv6.conf.all.disable_ipv6 = 1
[root@much ~]# 
[root@much ~]# cat /etc/sysctl.conf
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

[root@much ~]# 
[root@much ~]#  cat /proc/sys/net/ipv6/conf/all/disable_ipv6
1
[root@much ~]# 
~~~

> **Note:** 如果不关闭 IPv６ 会在后面的 MapReduce 任务调度过程中出错，因为 **`:8032`** 端口会绑定到 IPv6 的地址上面，从而使本地的 IPv4 的访问方式访问不到服务，而不断重试


---

## 下载 Hadoop 包

**Hadoop** 包的下载地址可以参考 **[Apache Hadoop Releases Download][hadoop_dl]**


这里我们下载最新的稳定版 **[hadoop-2.8.1.tar.gz][hadoop_2.8.1]**


~~~
[hadoop@much ~]$ ls
[hadoop@much ~]$ wget http://apache.fayea.com/hadoop/common/hadoop-2.8.1/hadoop-2.8.1.tar.gz
--2017-08-07 00:26:32--  http://apache.fayea.com/hadoop/common/hadoop-2.8.1/hadoop-2.8.1.tar.gz
Resolving apache.fayea.com (apache.fayea.com)... 119.6.242.164
Connecting to apache.fayea.com (apache.fayea.com)|119.6.242.164|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 424555111 (405M) [application/x-gzip]
Saving to: ‘hadoop-2.8.1.tar.gz’

100%[==============================================>] 424,555,111 34.3KB/s   in 13m 4s 

2017-08-07 00:39:37 (529 KB/s) - ‘hadoop-2.8.1.tar.gz’ saved [424555111/424555111]

[hadoop@much ~]$ ls
hadoop-2.8.1.tar.gz
[hadoop@much ~]$ du -sh hadoop-2.8.1.tar.gz 
405M	hadoop-2.8.1.tar.gz
[hadoop@much ~]$ md5sum  hadoop-2.8.1.tar.gz 
def2211ee1561871794e1b0eec8cb628  hadoop-2.8.1.tar.gz
[hadoop@much ~]$
~~~

进行简单的校验是为了确保数据的完整

可以在下载页面里找到 **[hadoop-2.8.1.tar.gz 的校验和](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.8.1/hadoop-2.8.1.tar.gz.mds)**

---

## 解压 Hadoop 包

~~~
[hadoop@much ~]$ ls
hadoop-2.8.1.tar.gz
[hadoop@much ~]$ tar -xzf hadoop-2.8.1.tar.gz 
[hadoop@much ~]$ ls
hadoop-2.8.1  hadoop-2.8.1.tar.gz
[hadoop@much ~]$ du -sh hadoop-2.8.1
2.3G	hadoop-2.8.1
[hadoop@much ~]$ ll
total 414608
drwxrwxr-x. 9 hadoop hadoop       149 Jun  2 14:24 hadoop-2.8.1
-rw-rw-r--. 1 hadoop hadoop 424555111 Jul 20 02:58 hadoop-2.8.1.tar.gz
[hadoop@much ~]$
~~~

---

## 修改 Hadoop 的 JAVA_HOME

~~~
[hadoop@much ~]$ ls
hadoop-2.8.1  hadoop-2.8.1.tar.gz
[hadoop@much ~]$ cd hadoop-2.8.1/
[hadoop@much hadoop-2.8.1]$ ls
bin  etc  include  lib  libexec  LICENSE.txt  NOTICE.txt  README.txt  sbin  share
[hadoop@much hadoop-2.8.1]$ bin/hadoop
Error: JAVA_HOME is not set and could not be found.
[hadoop@much hadoop-2.8.1]$ vim  etc/hadoop/hadoop-env.sh 
[hadoop@much hadoop-2.8.1]$ grep  JAVA_HOME etc/hadoop/hadoop-env.sh
# The only required environment variable is JAVA_HOME.  All others are
# set JAVA_HOME in this file, so that it is correctly defined on
#export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64
[hadoop@much hadoop-2.8.1]$ bin/hadoop
Usage: hadoop [--config confdir] [COMMAND | CLASSNAME]
  CLASSNAME            run the class named CLASSNAME
 or
  where COMMAND is one of:
  fs                   run a generic filesystem user client
  version              print the version
  jar <jar>            run a jar file
                       note: please use "yarn jar" to launch
                             YARN applications, not this command.
  checknative [-a|-h]  check native hadoop and compression libraries availability
  distcp <srcurl> <desturl> copy file or directories recursively
  archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
  classpath            prints the class path needed to get the
                       Hadoop jar and the required libraries
  credential           interact with credential providers
  daemonlog            get/set the log level for each daemon
  trace                view and modify Hadoop tracing settings

Most commands print help when invoked w/o parameters.
[hadoop@much hadoop-2.8.1]$ bin/hadoop version
Hadoop 2.8.1
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 20fe5304904fc2f5a18053c389e43cd26f7a70fe
Compiled by vinodkv on 2017-06-02T06:14Z
Compiled with protoc 2.5.0
From source with checksum 60125541c2b3e266cbf3becc5bda666
This command was run using /home/hadoop/hadoop-2.8.1/share/hadoop/common/hadoop-common-2.8.1.jar
[hadoop@much hadoop-2.8.1]$
~~~

---

## 脱机模式


默认情况下，Hadoop 是作为一个 Java 进程运行在非分布式模式下，以方便排错

下面一个例子就是利用了 Hadoop 的单进程模式

~~~
[hadoop@much hadoop-2.8.1]$ ls
bin  etc  include  lib  libexec  LICENSE.txt  NOTICE.txt  README.txt  sbin  share
[hadoop@much hadoop-2.8.1]$ mkdir input
[hadoop@much hadoop-2.8.1]$ cp etc/hadoop/*.xml input/
[hadoop@much hadoop-2.8.1]$ ll input/
total 48
-rw-rw-r--. 1 hadoop hadoop 4942 Aug  7 01:17 capacity-scheduler.xml
-rw-rw-r--. 1 hadoop hadoop  774 Aug  7 01:17 core-site.xml
-rw-rw-r--. 1 hadoop hadoop 9683 Aug  7 01:17 hadoop-policy.xml
-rw-rw-r--. 1 hadoop hadoop  775 Aug  7 01:17 hdfs-site.xml
-rw-rw-r--. 1 hadoop hadoop  620 Aug  7 01:17 httpfs-site.xml
-rw-rw-r--. 1 hadoop hadoop 3518 Aug  7 01:17 kms-acls.xml
-rw-rw-r--. 1 hadoop hadoop 5546 Aug  7 01:17 kms-site.xml
-rw-rw-r--. 1 hadoop hadoop  690 Aug  7 01:17 yarn-site.xml
[hadoop@much hadoop-2.8.1]$ ll output
ls: cannot access output: No such file or directory
[hadoop@much hadoop-2.8.1]$ bin/hadoop   jar  share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar    grep ./input/   ./output    'dfs[a-z.]+'
17/08/07 01:18:34 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
17/08/07 01:18:34 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
17/08/07 01:18:34 INFO input.FileInputFormat: Total input files to process : 8
17/08/07 01:18:34 INFO mapreduce.JobSubmitter: number of splits:8
17/08/07 01:18:35 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local1783062321_0001
17/08/07 01:18:35 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
17/08/07 01:18:35 INFO mapreduce.Job: Running job: job_local1783062321_0001
17/08/07 01:18:35 INFO mapred.LocalJobRunner: OutputCommitter set in config null
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Waiting for map tasks
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000000_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/hadoop-policy.xml:0+9683
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.MapTask: Spilling map output
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufend = 17; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214396(104857584); length = 1/6553600
17/08/07 01:18:35 INFO mapred.MapTask: Finished spill 0
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000000_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000000_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000000_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000001_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/kms-site.xml:0+5546
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000001_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000001_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000001_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000002_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/capacity-scheduler.xml:0+4942
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000002_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000002_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000002_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000003_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/kms-acls.xml:0+3518
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000003_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000003_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000003_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000004_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/hdfs-site.xml:0+775
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000004_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000004_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000004_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000005_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/core-site.xml:0+774
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000005_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000005_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000005_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000006_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/yarn-site.xml:0+690
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000006_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000006_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000006_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_m_000007_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/input/httpfs-site.xml:0+620
17/08/07 01:18:35 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:35 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:35 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:35 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:35 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:35 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 
17/08/07 01:18:35 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_m_000007_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_m_000007_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_m_000007_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: map task executor complete.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Waiting for reduce tasks
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Starting task: attempt_local1783062321_0001_r_000000_0
17/08/07 01:18:35 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:35 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:35 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:35 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@6240b29f
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=370304608, maxSingleShuffleLimit=92576152, mergeThreshold=244401056, ioSortFactor=10, memToMemMergeOutputsThreshold=10
17/08/07 01:18:35 INFO reduce.EventFetcher: attempt_local1783062321_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000003_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000003_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->2
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000006_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000006_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 2, commitMemory -> 2, usedMemory ->4
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000000_0 decomp: 21 len: 25 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 21 bytes from map-output for attempt_local1783062321_0001_m_000000_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 21, inMemoryMapOutputs.size() -> 3, commitMemory -> 4, usedMemory ->25
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000002_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000002_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 4, commitMemory -> 25, usedMemory ->27
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000005_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000005_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 5, commitMemory -> 27, usedMemory ->29
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000007_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000007_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 6, commitMemory -> 29, usedMemory ->31
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000001_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000001_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 7, commitMemory -> 31, usedMemory ->33
17/08/07 01:18:35 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1783062321_0001_m_000004_0 decomp: 2 len: 6 to MEMORY
17/08/07 01:18:35 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1783062321_0001_m_000004_0
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 8, commitMemory -> 33, usedMemory ->35
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:35 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 8 / 8 copied.
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: finalMerge called with 8 in-memory map-outputs and 0 on-disk map-outputs
17/08/07 01:18:35 INFO mapred.Merger: Merging 8 sorted segments
17/08/07 01:18:35 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 10 bytes
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: Merged 8 segments, 35 bytes to disk to satisfy reduce memory limit
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: Merging 1 files, 25 bytes from disk
17/08/07 01:18:35 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
17/08/07 01:18:35 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 01:18:35 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 10 bytes
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 8 / 8 copied.
17/08/07 01:18:35 INFO Configuration.deprecation: mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
17/08/07 01:18:35 INFO mapred.Task: Task:attempt_local1783062321_0001_r_000000_0 is done. And is in the process of committing
17/08/07 01:18:35 INFO mapred.LocalJobRunner: 8 / 8 copied.
17/08/07 01:18:35 INFO mapred.Task: Task attempt_local1783062321_0001_r_000000_0 is allowed to commit now
17/08/07 01:18:35 INFO output.FileOutputCommitter: Saved output of task 'attempt_local1783062321_0001_r_000000_0' to file:/home/hadoop/hadoop-2.8.1/grep-temp-1237275612/_temporary/0/task_local1783062321_0001_r_000000
17/08/07 01:18:35 INFO mapred.LocalJobRunner: reduce > reduce
17/08/07 01:18:35 INFO mapred.Task: Task 'attempt_local1783062321_0001_r_000000_0' done.
17/08/07 01:18:35 INFO mapred.LocalJobRunner: Finishing task: attempt_local1783062321_0001_r_000000_0
17/08/07 01:18:35 INFO mapred.LocalJobRunner: reduce task executor complete.
17/08/07 01:18:36 INFO mapreduce.Job: Job job_local1783062321_0001 running in uber mode : false
17/08/07 01:18:36 INFO mapreduce.Job:  map 100% reduce 100%
17/08/07 01:18:36 INFO mapreduce.Job: Job job_local1783062321_0001 completed successfully
17/08/07 01:18:36 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=2954121
		FILE: Number of bytes written=5645783
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=757
		Map output records=1
		Map output bytes=17
		Map output materialized bytes=67
		Input split bytes=933
		Combine input records=1
		Combine output records=1
		Reduce input groups=1
		Reduce shuffle bytes=67
		Reduce input records=1
		Reduce output records=1
		Spilled Records=2
		Shuffled Maps =8
		Failed Shuffles=0
		Merged Map outputs=8
		GC time elapsed (ms)=32
		Total committed heap usage (bytes)=4065329152
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=26548
	File Output Format Counters 
		Bytes Written=123
17/08/07 01:18:36 INFO jvm.JvmMetrics: Cannot initialize JVM Metrics with processName=JobTracker, sessionId= - already initialized
17/08/07 01:18:36 INFO input.FileInputFormat: Total input files to process : 1
17/08/07 01:18:36 INFO mapreduce.JobSubmitter: number of splits:1
17/08/07 01:18:36 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local1111349493_0002
17/08/07 01:18:36 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
17/08/07 01:18:36 INFO mapreduce.Job: Running job: job_local1111349493_0002
17/08/07 01:18:36 INFO mapred.LocalJobRunner: OutputCommitter set in config null
17/08/07 01:18:36 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:36 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:36 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
17/08/07 01:18:36 INFO mapred.LocalJobRunner: Waiting for map tasks
17/08/07 01:18:36 INFO mapred.LocalJobRunner: Starting task: attempt_local1111349493_0002_m_000000_0
17/08/07 01:18:36 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:36 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:36 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:36 INFO mapred.MapTask: Processing split: file:/home/hadoop/hadoop-2.8.1/grep-temp-1237275612/part-r-00000:0+111
17/08/07 01:18:36 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
17/08/07 01:18:36 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
17/08/07 01:18:36 INFO mapred.MapTask: soft limit at 83886080
17/08/07 01:18:36 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
17/08/07 01:18:36 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
17/08/07 01:18:36 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
17/08/07 01:18:36 INFO mapred.LocalJobRunner: 
17/08/07 01:18:36 INFO mapred.MapTask: Starting flush of map output
17/08/07 01:18:36 INFO mapred.MapTask: Spilling map output
17/08/07 01:18:36 INFO mapred.MapTask: bufstart = 0; bufend = 17; bufvoid = 104857600
17/08/07 01:18:36 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214396(104857584); length = 1/6553600
17/08/07 01:18:36 INFO mapred.MapTask: Finished spill 0
17/08/07 01:18:36 INFO mapred.Task: Task:attempt_local1111349493_0002_m_000000_0 is done. And is in the process of committing
17/08/07 01:18:36 INFO mapred.LocalJobRunner: map
17/08/07 01:18:36 INFO mapred.Task: Task 'attempt_local1111349493_0002_m_000000_0' done.
17/08/07 01:18:36 INFO mapred.LocalJobRunner: Finishing task: attempt_local1111349493_0002_m_000000_0
17/08/07 01:18:36 INFO mapred.LocalJobRunner: map task executor complete.
17/08/07 01:18:36 INFO mapred.LocalJobRunner: Waiting for reduce tasks
17/08/07 01:18:36 INFO mapred.LocalJobRunner: Starting task: attempt_local1111349493_0002_r_000000_0
17/08/07 01:18:36 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
17/08/07 01:18:36 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
17/08/07 01:18:36 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
17/08/07 01:18:36 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@199b7cfa
17/08/07 01:18:36 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=370304608, maxSingleShuffleLimit=92576152, mergeThreshold=244401056, ioSortFactor=10, memToMemMergeOutputsThreshold=10
17/08/07 01:18:36 INFO reduce.EventFetcher: attempt_local1111349493_0002_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
17/08/07 01:18:36 INFO reduce.LocalFetcher: localfetcher#2 about to shuffle output of map attempt_local1111349493_0002_m_000000_0 decomp: 21 len: 25 to MEMORY
17/08/07 01:18:36 INFO reduce.InMemoryMapOutput: Read 21 bytes from map-output for attempt_local1111349493_0002_m_000000_0
17/08/07 01:18:36 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 21, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->21
17/08/07 01:18:36 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
17/08/07 01:18:36 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:208)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
17/08/07 01:18:36 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 01:18:36 INFO reduce.MergeManagerImpl: finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
17/08/07 01:18:36 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 01:18:36 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 11 bytes
17/08/07 01:18:36 INFO reduce.MergeManagerImpl: Merged 1 segments, 21 bytes to disk to satisfy reduce memory limit
17/08/07 01:18:36 INFO reduce.MergeManagerImpl: Merging 1 files, 25 bytes from disk
17/08/07 01:18:36 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
17/08/07 01:18:36 INFO mapred.Merger: Merging 1 sorted segments
17/08/07 01:18:36 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 11 bytes
17/08/07 01:18:36 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 01:18:36 INFO mapred.Task: Task:attempt_local1111349493_0002_r_000000_0 is done. And is in the process of committing
17/08/07 01:18:36 INFO mapred.LocalJobRunner: 1 / 1 copied.
17/08/07 01:18:36 INFO mapred.Task: Task attempt_local1111349493_0002_r_000000_0 is allowed to commit now
17/08/07 01:18:36 INFO output.FileOutputCommitter: Saved output of task 'attempt_local1111349493_0002_r_000000_0' to file:/home/hadoop/hadoop-2.8.1/output/_temporary/0/task_local1111349493_0002_r_000000
17/08/07 01:18:36 INFO mapred.LocalJobRunner: reduce > reduce
17/08/07 01:18:36 INFO mapred.Task: Task 'attempt_local1111349493_0002_r_000000_0' done.
17/08/07 01:18:36 INFO mapred.LocalJobRunner: Finishing task: attempt_local1111349493_0002_r_000000_0
17/08/07 01:18:36 INFO mapred.LocalJobRunner: reduce task executor complete.
17/08/07 01:18:37 INFO mapreduce.Job: Job job_local1111349493_0002 running in uber mode : false
17/08/07 01:18:37 INFO mapreduce.Job:  map 100% reduce 100%
17/08/07 01:18:37 INFO mapreduce.Job: Job job_local1111349493_0002 completed successfully
17/08/07 01:18:37 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=1274768
		FILE: Number of bytes written=2505222
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=1
		Map output records=1
		Map output bytes=17
		Map output materialized bytes=25
		Input split bytes=129
		Combine input records=0
		Combine output records=0
		Reduce input groups=1
		Reduce shuffle bytes=25
		Reduce input records=1
		Reduce output records=1
		Spilled Records=2
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=0
		Total committed heap usage (bytes)=1058013184
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=123
	File Output Format Counters 
		Bytes Written=23
[hadoop@much hadoop-2.8.1]$ echo $?
0
[hadoop@much hadoop-2.8.1]$ ll output/
total 4
-rw-r--r--. 1 hadoop hadoop 11 Aug  7 01:18 part-r-00000
-rw-r--r--. 1 hadoop hadoop  0 Aug  7 01:18 _SUCCESS
[hadoop@much hadoop-2.8.1]$ cat output/*
1	dfsadmin
[hadoop@much hadoop-2.8.1]$ 
~~~

> **Note:**  **output** 目录是在执行过程中被 Hadoop 自动创建的，不用事先自己创建，如果事先自己创建了，会出错，解决办法是在一个有写权限的位置指定一个未被创建的子目录


---

# 命令汇总

* **`rpm  -qa | grep epel`**
* **`yum  list all | grep -i epel`**
* **`yum  install epel-release.noarch`**
* **`yum  list all | grep -i java | grep -i  jdk`**
* **`yum install java-1.8.0-openjdk.x86_64  java-1.8.0-openjdk-devel.x86_64  java-1.8.0-openjdk-headless.x86_64`**
* **`java -version`**
* **`ll /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/bin/java`**
* **`ll /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/`**
* **`yum install openssh rsync`**
* **`setenforce  0`**
* **`getenforce`**
* **`useradd hadoop`**
* **`passwd hadoop`**
* **`su - hadoop`**
* **`ssh-keygen -t rsa -P ""`**
* **`cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys`**
* **`chmod 600 $HOME/.ssh/authorized_keys`**
* **`ssh localhost`**
* **`ssh -vvv localhost`**
* **`sysctl -a | grep -i net.ipv6.conf.all.disable_ipv6`**
* **`vim  /etc/sysctl.conf`**
* **`sysctl -p`**
* **`sysctl -a | grep -i net.ipv6.conf.all.disable_ipv6`**
* **`cat /etc/sysctl.conf`**
* **`cat /proc/sys/net/ipv6/conf/all/disable_ipv6`**
* **`wget http://apache.fayea.com/hadoop/common/hadoop-2.8.1/hadoop-2.8.1.tar.gz`**
* **`du -sh hadoop-2.8.1.tar.gz`**
* **`md5sum  hadoop-2.8.1.tar.gz`**
* **`tar -xzf hadoop-2.8.1.tar.gz`**
* **`du -sh hadoop-2.8.1`**
* **`cd hadoop-2.8.1/`**
* **`bin/hadoop`**
* **`vim  etc/hadoop/hadoop-env.sh`**
* **`grep  JAVA_HOME etc/hadoop/hadoop-env.sh`**
* **`bin/hadoop`**
* **`bin/hadoop version`**
* **`mkdir input`**
* **`cp etc/hadoop/*.xml input/`**
* **`ll output`**
* **`bin/hadoop   jar  share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar    grep ./input/   ./output    'dfs[a-z.]+'`**
* **`ll output/`**
* **`cat output/*`**

---

[hadoop]:http://hadoop.apache.org/
[singlecluster]:https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
[hadoop_doc]:https://wiki.apache.org/hadoop
[install_hadoop_on_ubuntu]:http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/
[hadoop_dl]:http://hadoop.apache.org/releases.html
[hadoop_2.8.1]:http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.8.1/hadoop-2.8.1.tar.gz
[HadoopJavaVersions]:https://wiki.apache.org/hadoop/HadoopJavaVersions
