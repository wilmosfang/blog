---
layout: post
title: "Install CMDBuild"
author:  wilmosfang
date: 2018-03-06 14:58:06
image: '/assets/img/'
excerpt: '安装 CMDBuild'
main-class: cmdbuild
color: '#f79a30'
tags:
 - cmdbuild
 - postgresql
 - tomcat
categories:
 - cmdbuild
twitter_text: 'simple process of CMDBuild installation'
introduction: 'installation of CMDBuild'
---



## 前言

**[CMDBuild][cmdbuild]** 是一款优秀的开源 **CMDB** 软件

在生产实践中 **CMDB** 可以用来记录与管理计算相关的资源信息，协调与管理服务信息(事实上并不局限些这个领域的信息)

在服务信息规模从小到大的演进历程中(或者一个项目从初生到成熟的演化过程中)，对信息的管理大多会经历以下几个阶段

* 人脑记
* 文本记
* 表格记
* 领域工具软件记
* 数据库记
* CMDB(与其翻译成配置管理数据，我觉得更恰当的应该是信息中心)
* 基于CMDB的更高层应用

可以说 **CMDB** 是管理信息扩张过程中工具革新的一个必经之路

准确来说 **CMDB** 应该算作一种 IT 信息管理理念，对信息处理工具的信息通过信息系统进行管理的一种理念，而并不局限于某一特定工具或对象

**[CMDBuild][cmdbuild]** 是这种理念的一个开源实现

其融合很多开源工具，实现了一个可以方便使用的信息管理通用平台，并且留有一些自定义空间来适应不同场景中具体的特异的需求

* CMDBuild is the ERP of the Information Technology Department
* CMDBuild is the integrated solution for the Service Desk management
* CMDBuild is the control system of the IT infrastructure

这里分享一下 **[CMDBuild][cmdbuild]** 的安装方法

参考 **[Technical Manual][cmdbuild_manual]**

> **Tip:** 当前的版本为 **cmdbuild-2.5.0**

---

# 操作


## 环境

~~~
[root@h210 ~]# hostnamectl
   Static hostname: h210
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 739de39e0b1440618015b8dcd595f9f7
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@h210 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 82748sec preferred_lft 82748sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h210 ~]#
~~~

## 依赖

### 硬件依赖

* 主流架构的服务器
* 一个 CMDBuild 实例配置 4G 内存，生产环境下推荐 6-8G
* 一个 CMDBuild 实例配置 120G 磁盘

### 软件依赖

* 支持以下软件的任何操作系统(但是更推荐Linux)
* PostgreSQL 9.4 to 9.6
* Apache Tomcat 6.0 or 7.0 or 8.0 (推荐7.068)
* JDK 1.8
* (可选) PostGIS 1.5.2 or 2.0
* (可选) Alfresco 3.4 用于卡片文档管理，或者 DMS 支持 CMIS 协议

>前面的两篇文章中已经交代了 JDK Tomcat PostgreSQL 的安装方法，这里有不明白的可以翻阅前面的博客进行了解

其它相关细节可以参考 **[System requirements][cmdbuild_dl]**


## 下载软件包

下载地址可以参考　**[Installing and using CMDBuild][cmdbuild_dl]**

~~~
[root@h210 ~]# ll /tmp/
total 171876
-rw-r--r--. 1 root root 175999643 3月   6 23:59 cmdbuild-2.5.0.zip
drwx------. 3 root root        17 3月   6 21:38 systemd-private-936f769f7360491681becf9dd3663886-cups.service-5nh2ZV
[root@h210 ~]#
~~~


## 创建数据库

~~~
[root@h210 ~]# su - postgres
Last login: 二 3月  6 22:25:08 CST 2018 on pts/0
-bash-4.2$ psql
psql (10.3)
Type "help" for help.

postgres=# create database cmdbuild with owner postgres encoding = 'UTF8';
CREATE DATABASE
postgres=# alter user postgres with password 'postgres';
ALTER ROLE
postgres=# \q
-bash-4.2$
~~~

顺便修改了 **postgres** 用户的密码

## 解压 cmdbuild

~~~
[root@h210 tmp]# ls
cmdbuild-2.5.0.zip  systemd-private-936f769f7360491681becf9dd3663886-cups.service-5nh2ZV
[root@h210 tmp]# mkdir cmdbuild
[root@h210 tmp]# mv cmdbuild-2.5.0.zip  cmdbuild
[root@h210 tmp]# cd cmdbuild/
[root@h210 cmdbuild]# ls
cmdbuild-2.5.0.zip
[root@h210 cmdbuild]# unzip cmdbuild-2.5.0.zip
Archive:  cmdbuild-2.5.0.zip
   creating: cmdbuild-2.5.0/
  inflating: cmdbuild-2.5.0/cmdbuild-2.5.0.war  
   creating: cmdbuild-2.5.0/extras/
  inflating: cmdbuild-2.5.0/extras/cmdbuild-distribution-old-shark-overlay-2.5.0.zip  
  inflating: cmdbuild-2.5.0/extras/cmdbuild-distribution-shark-overlay-2.5.0.zip  
   creating: cmdbuild-2.5.0/extras/tomcat-libs/
   creating: cmdbuild-2.5.0/extras/tomcat-libs/5.5/
  inflating: cmdbuild-2.5.0/extras/tomcat-libs/5.5/dbcp-6.0.45.jar  
  inflating: cmdbuild-2.5.0/extras/tomcat-libs/5.5/postgresql-9.4.1207.jar  
   creating: cmdbuild-2.5.0/extras/tomcat-libs/6.0 or higher/
  inflating: cmdbuild-2.5.0/extras/tomcat-libs/6.0 or higher/postgresql-9.4.1207.jar  
   creating: cmdbuild-2.5.0/extras/workflow/
   creating: cmdbuild-2.5.0/extras/workflow/RFC/
   creating: cmdbuild-2.5.0/extras/geoserver/
  inflating: cmdbuild-2.5.0/INSTALL.txt  
  inflating: cmdbuild-2.5.0/extras/workflow/RFC/RequestForChange.xpdl  
  inflating: cmdbuild-2.5.0/extras/geoserver/config.yaml  
  inflating: cmdbuild-2.5.0/extras/tomcat-libs/5.5/INSTALL.txt  
  inflating: cmdbuild-2.5.0/extras/tomcat-libs/6.0 or higher/INSTALL.txt  
  inflating: cmdbuild-2.5.0/COPYING  
  inflating: cmdbuild-2.5.0/CHANGELOG  
  inflating: cmdbuild-2.5.0/README   
[root@h210 cmdbuild]#
~~~

## 拷贝 cmdbuild.war

~~~
[root@h210 cmdbuild]# cp /tmp/cmdbuild/cmdbuild-2.5.0/cmdbuild-2.5.0.war /data/apache-tomcat-8.5.28/webapps/cmdbuild.war
[root@h210 cmdbuild]#
[root@h210 cmdbuild]# ll /data/apache-tomcat-8.5.28/webapps/cmdbuild.war
-rw-r--r--. 1 root root 177969167 3月   7 00:39 /data/apache-tomcat-8.5.28/webapps/cmdbuild.war
[root@h210 cmdbuild]#
~~~


## 拷贝依赖库

~~~
[root@h210 cmdbuild]# cp /tmp/cmdbuild/cmdbuild-2.5.0/extras/tomcat-libs/6.0\ or\ higher/postgresql-9.4.1207.jar  /data/apache-tomcat-8.5.28/lib/
[root@h210 cmdbuild]#
[root@h210 cmdbuild]# ll /data/apache-tomcat-8.5.28/lib/postgresql-9.4.1207.jar
-rw-r--r--. 1 root root 607093 3月   7 00:40 /data/apache-tomcat-8.5.28/lib/postgresql-9.4.1207.jar
[root@h210 cmdbuild]#
~~~

## 解压拷贝 shark

~~~
[root@h210 cmdbuild]# cp /tmp/cmdbuild/cmdbuild-2.5.0/extras/cmdbuild-distribution-shark-overlay-2.5.0.zip .
[root@h210 cmdbuild]# ls
cmdbuild-2.5.0  cmdbuild-2.5.0.zip  cmdbuild-distribution-shark-overlay-2.5.0.zip
[root@h210 cmdbuild]# unzip cmdbuild-distribution-shark-overlay-2.5.0.zip
Archive:  cmdbuild-distribution-shark-overlay-2.5.0.zip
   creating: cmdbuild-shark-overlay-2.5.0/
   creating: cmdbuild-shark-overlay-2.5.0/WEB-INF/
   creating: cmdbuild-shark-overlay-2.5.0/WEB-INF/lib/
  inflating: cmdbuild-shark-overlay-2.5.0/WEB-INF/lib/cmdbuild-ws-client-2.5.0.jar  
  inflating: cmdbuild-shark-overlay-2.5.0/WEB-INF/lib/cmdbuild-commons-2.5.0.jar  
  inflating: cmdbuild-shark-overlay-2.5.0/WEB-INF/lib/cmdbuild-shark-commons-2.5.0.jar  
  inflating: cmdbuild-shark-overlay-2.5.0/WEB-INF/lib/cmdbuild-shark-extensions-2.5.0.jar  
   creating: cmdbuild-shark-overlay-2.5.0/conf/
  inflating: cmdbuild-shark-overlay-2.5.0/conf/Shark.conf  
[root@h210 cmdbuild]#
[root@h210 cmdbuild]# ls
cmdbuild-2.5.0  cmdbuild-2.5.0.zip  cmdbuild-distribution-shark-overlay-2.5.0.zip  cmdbuild-shark-overlay-2.5.0
[root@h210 cmdbuild]# cp -r cmdbuild-shark-overlay-2.5.0/ /data/apache-tomcat-8.5.28/webapps/shark
[root@h210 cmdbuild]# du -sh  /data/apache-tomcat-8.5.28/webapps/shark
532K	/data/apache-tomcat-8.5.28/webapps/shark
[root@h210 cmdbuild]# du -sh cmdbuild-shark-overlay-2.5.0/
532K	cmdbuild-shark-overlay-2.5.0/
[root@h210 cmdbuild]#
~~~

## 启动 tomcat

~~~
[root@h210 cmdbuild]# cd /data/apache-tomcat-8.5.28/bin/
[root@h210 bin]# ls
bootstrap.jar       commons-daemon.jar            daemon.sh         setclasspath.sh  startup.sh            tool-wrapper.sh
catalina.bat        commons-daemon-native.tar.gz  digest.bat        shutdown.bat     tomcat-juli.jar       version.bat
catalina.sh         configtest.bat                digest.sh         shutdown.sh      tomcat-native.tar.gz  version.sh
catalina-tasks.xml  configtest.sh                 setclasspath.bat  startup.bat      tool-wrapper.bat
[root@h210 bin]# ps faux | grep tomcat
root      5286  0.0  0.0 112648  1016 pts/0    S+   00:46   0:00          \_ grep --color=auto tomcat
[root@h210 bin]# ./catalina.sh start
Using CATALINA_BASE:   /data/apache-tomcat-8.5.28
Using CATALINA_HOME:   /data/apache-tomcat-8.5.28
Using CATALINA_TMPDIR: /data/apache-tomcat-8.5.28/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /data/apache-tomcat-8.5.28/bin/bootstrap.jar:/data/apache-tomcat-8.5.28/bin/tomcat-juli.jar
Tomcat started.
[root@h210 bin]#
[root@h210 cmdbuild]# ps fuax | grep tomcat
root      5701  0.0  0.0 112648  1016 pts/0    S+   01:05   0:00          \_ grep --color=auto tomcat
root      5300  4.1 21.4 3130692 438816 pts/0  Sl   00:46   0:47 /usr/bin/java -Djava.util.logging.config.file=/data/apache-tomcat-8.5.28/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /data/apache-tomcat-8.5.28/bin/bootstrap.jar:/data/apache-tomcat-8.5.28/bin/tomcat-juli.jar -Dcatalina.base=/data/apache-tomcat-8.5.28 -Dcatalina.home=/data/apache-tomcat-8.5.28 -Djava.io.tmpdir=/data/apache-tomcat-8.5.28/temp org.apache.catalina.startup.Bootstrap start
[root@h210 cmdbuild]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1741/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1524/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1526/cupsd          
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      2484/postmaster     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1654/master         
tcp        0      0 192.168.56.210:22       192.168.56.1:47170      ESTABLISHED 4143/sshd: root@pts
tcp6       0      0 :::8009                 :::*                    LISTEN      5300/java           
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::8080                 :::*                    LISTEN      5300/java           
tcp6       0      0 :::22                   :::*                    LISTEN      1524/sshd           
tcp6       0      0 ::1:631                 :::*                    LISTEN      1526/cupsd          
tcp6       0      0 ::1:5432                :::*                    LISTEN      2484/postmaster     
tcp6       0      0 ::1:25                  :::*                    LISTEN      1654/master         
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      5300/java           
[root@h210 cmdbuild]#
~~~


## 访问

**`http://192.168.56.210:8080/cmdbuild/`**


![cmdbuild](/assets/img/cmdbuild/cmdbuild01.png)

---

# 总结

* 下载最新项目压缩包　(http://www.cmdbuild.org/download)
* 拷贝根目录的 CMDBuild-{version}.war 拷贝到 Tomcat 的 webapps 中，然后重命名为 cmdbuild.war
* 拷贝 extras 中的 CMDBuild-shark 到 Tomcat 的 webapps 中
* 拷贝 extras/tomcat-libs 中相应版本的依赖库到 Tomcat 的 lib 中
* 启动 tomcat 进程

java 的大部分应用都是只需要把相应文件拷贝到正确的目录中即可

关于 **CMDBuild** 的配置后面进行展开

* TOC
{:toc}


---


[cmdbuild]:http://www.cmdbuild.org/en/
[cmdbuild_manual]:http://www.cmdbuild.org/en/documentazione/manuali/technical-manual
[cmdbuild_dl]:http://www.cmdbuild.org/en/download
