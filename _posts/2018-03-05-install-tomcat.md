---
layout: post
title: "Install Tomcat"
author:  wilmosfang
date: 2018-03-05 14:47:25
image: '/assets/img/'
excerpt: '安装 Tomcat'
main-class: cmdbuild
color: '#f79a30'
tags:
 - cmdbuild
 - tomcat
 - java
categories:
 - cmdbuild
twitter_text: 'simple process of Tomcat installation'
introduction: 'installation of Tomcat'
---



## 前言

**[Tomcat][tomcat]** 是一款开源的 Java Servlet 实现，简单来说就是一个 java 应用解析容器

>The Apache Tomcat® software is an open source implementation of the Java Servlet, JavaServer Pages, Java Expression Language and Java WebSocket technologies

常与 Apache 一起配合实现 WebApp, Apache 提供静态内容的服务，Tomcat 提供动态内容的服务

**CMDBuild** 也是通过 **[Tomcat][tomcat]** 来提供 WEB 服务, 在研究 **CMDBuild** 之前，需要对 **[Tomcat][tomcat]** 进行简单的了解

**[Tomcat][tomcat]** 依赖 java 运行环境，所以正常运行前的提是需要有相应版本的 JDK

这里分享一下 **[Tomcat][tomcat]** 的安装方法


> **Tip:** 当前的版本为 **9.0.5** , 但是在此演示的版本为 **8.5.28**, 原因为 **CMDBuild** 官方推荐此版本，为了考虑兼容性，就没有使用最新的版本

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


## 安装仓库

~~~
[root@h210 ~]# yum clean all
Loaded plugins: fastestmirror, langpacks
Cleaning repos: base c7-media extras updates
Cleaning up everything
[root@h210 ~]# yum list all | grep -i epel
epel-release.noarch                         7-9                        extras   
[root@h210 ~]# yum install  epel-release.noarch
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-9 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================================================
 Package                                Arch                             Version                         Repository                        Size
================================================================================================================================================
Installing:
 epel-release                           noarch                           7-9                             extras                            14 k

Transaction Summary
================================================================================================================================================
Install  1 Package

Total download size: 14 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-9.noarch.rpm                                                                                              |  14 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-9.noarch                                                                                                      1/1
  Verifying  : epel-release-7-9.noarch                                                                                                      1/1

Installed:
  epel-release.noarch 0:7-9                                                                                                                     

Complete!
[root@h210 ~]# echo $?
0
[root@h210 ~]#
~~~

## 安装 JDK

~~~
[root@h210 ~]# yum install java-1.8.0-openjdk.x86_64
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.131-3.b12.el7_3 will be updated
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.161-0.b14.el7_4 will be an update
--> Processing Dependency: java-1.8.0-openjdk-headless(x86-64) = 1:1.8.0.161-0.b14.el7_4 for package: 1:java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64
--> Running transaction check
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.131-3.b12.el7_3 will be updated
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.161-0.b14.el7_4 will be an update
--> Processing Dependency: nss-softokn(x86-64) >= 3.28.3 for package: 1:java-1.8.0-openjdk-headless-1.8.0.161-0.b14.el7_4.x86_64
--> Processing Dependency: copy-jdk-configs >= 2.2 for package: 1:java-1.8.0-openjdk-headless-1.8.0.161-0.b14.el7_4.x86_64
--> Running transaction check
---> Package copy-jdk-configs.noarch 0:1.2-1.el7 will be updated
---> Package copy-jdk-configs.noarch 0:2.2-5.el7_4 will be an update
---> Package nss-softokn.x86_64 0:3.16.2.3-14.4.el7 will be updated
---> Package nss-softokn.x86_64 0:3.28.3-8.el7_4 will be an update
--> Processing Dependency: nss-softokn-freebl(x86-64) >= 3.28.3-8.el7_4 for package: nss-softokn-3.28.3-8.el7_4.x86_64
--> Running transaction check
---> Package nss-softokn-freebl.x86_64 0:3.16.2.3-14.4.el7 will be updated
---> Package nss-softokn-freebl.x86_64 0:3.28.3-8.el7_4 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================================================
 Package                                      Arch                    Version                                    Repository                Size
================================================================================================================================================
Updating:
 java-1.8.0-openjdk                           x86_64                  1:1.8.0.161-0.b14.el7_4                    updates                  243 k
Updating for dependencies:
 copy-jdk-configs                             noarch                  2.2-5.el7_4                                updates                   19 k
 java-1.8.0-openjdk-headless                  x86_64                  1:1.8.0.161-0.b14.el7_4                    updates                   32 M
 nss-softokn                                  x86_64                  3.28.3-8.el7_4                             updates                  310 k
 nss-softokn-freebl                           x86_64                  3.28.3-8.el7_4                             updates                  214 k

Transaction Summary
================================================================================================================================================
Upgrade  1 Package (+4 Dependent packages)

Total download size: 32 M
Is this ok [y/d/N]: y
Downloading packages:
updates/7/x86_64/prestodelta                                                                                             | 771 kB  00:00:02     
Delta RPMs reduced 766 k of updates to 349 k (54% saved)
(1/5): nss-softokn-3.16.2.3-14.4.el7_3.28.3-8.el7_4.x86_64.drpm                                                          | 174 kB  00:00:01     
(2/5): copy-jdk-configs-2.2-5.el7_4.noarch.rpm                                                                           |  19 kB  00:00:01     
(3/5): java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3_1.8.0.161-0.b14.el7_4.x86_64.drpm                                        |  80 kB  00:00:01     
(4/5): nss-softokn-freebl-3.16.2.3-14.4.el7_3.28.3-8.el7_4.x86_64.drpm                                                   |  95 kB  00:00:02     
(5/5): java-1.8.0-openjdk-headless-1.8.0.161-0.b14.el7_4.x86_64.rpm                                                      |  32 MB  00:02:25     
------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                           225 kB/s |  32 MB  00:02:25     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : nss-softokn-freebl-3.28.3-8.el7_4.x86_64                                                                                    1/10
  Updating   : nss-softokn-3.28.3-8.el7_4.x86_64                                                                                           2/10
  Updating   : copy-jdk-configs-2.2-5.el7_4.noarch                                                                                         3/10
  Updating   : 1:java-1.8.0-openjdk-headless-1.8.0.161-0.b14.el7_4.x86_64                                                                  4/10
warning: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/java.security created as /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/java.security.rpmnew
warning: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/nss.cfg created as /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/nss.cfg.rpmnew
restored /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/java.security.rpmnew to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/java.security
restored /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/nss.cfg.rpmnew to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/security/nss.cfg
  Updating   : 1:java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64                                                                           5/10
  Cleanup    : 1:java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64                                                                           6/10
  Cleanup    : 1:java-1.8.0-openjdk-headless-1.8.0.131-3.b12.el7_3.x86_64                                                                  7/10
  Cleanup    : nss-softokn-3.16.2.3-14.4.el7.x86_64                                                                                        8/10
  Cleanup    : copy-jdk-configs-1.2-1.el7.noarch                                                                                           9/10
  Cleanup    : nss-softokn-freebl-3.16.2.3-14.4.el7.x86_64                                                                                10/10
  Verifying  : nss-softokn-3.28.3-8.el7_4.x86_64                                                                                           1/10
  Verifying  : copy-jdk-configs-2.2-5.el7_4.noarch                                                                                         2/10
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.161-0.b14.el7_4.x86_64                                                                  3/10
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64                                                                           4/10
  Verifying  : nss-softokn-freebl-3.28.3-8.el7_4.x86_64                                                                                    5/10
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64                                                                           6/10
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.131-3.b12.el7_3.x86_64                                                                  7/10
  Verifying  : nss-softokn-3.16.2.3-14.4.el7.x86_64                                                                                        8/10
  Verifying  : nss-softokn-freebl-3.16.2.3-14.4.el7.x86_64                                                                                 9/10
  Verifying  : copy-jdk-configs-1.2-1.el7.noarch                                                                                          10/10

Updated:
  java-1.8.0-openjdk.x86_64 1:1.8.0.161-0.b14.el7_4                                                                                             

Dependency Updated:
  copy-jdk-configs.noarch 0:2.2-5.el7_4       java-1.8.0-openjdk-headless.x86_64 1:1.8.0.161-0.b14.el7_4  nss-softokn.x86_64 0:3.28.3-8.el7_4
  nss-softokn-freebl.x86_64 0:3.28.3-8.el7_4

Complete!
[root@h210 ~]# java -version
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
[root@h210 ~]#
~~~



## 获取 tomcat

~~~
[root@h210 tomcat]# wget http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.28/bin/apache-tomcat-8.5.28.tar.gz
--2017-07-04 02:32:49--  http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.28/bin/apache-tomcat-8.5.28.tar.gz
Resolving www-us.apache.org (www-us.apache.org)... 140.211.11.105
Connecting to www-us.apache.org (www-us.apache.org)|140.211.11.105|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9544774 (9.1M) [application/x-gzip]
Saving to: ‘apache-tomcat-8.5.28.tar.gz’

100%[======================================>] 9,544,774    301KB/s   in 28s    

2017-07-04 02:33:18 (327 KB/s) - ‘apache-tomcat-8.5.28.tar.gz’ saved [9544774/9544774]

[root@h210 tomcat]#
~~~


## 解压 tomcat

~~~
[root@h210 tomcat]# mkdir /data
[root@h210 tomcat]# tar -xvf apache-tomcat-8.5.28.tar.gz -C /data/
apache-tomcat-8.5.28/conf/
apache-tomcat-8.5.28/conf/catalina.policy
tar: apache-tomcat-8.5.28/conf/catalina.policy: time stamp 2018-02-07 07:12:49 is 18851057.64465244 s in the future
apache-tomcat-8.5.28/conf/catalina.properties
tar: apache-tomcat-8.5.28/conf/catalina.properties: time stamp 2018-02-07 07:12:49 is 18851057.643874223 s in the future
apache-tomcat-8.5.28/conf/context.xml
...
...
tar: apache-tomcat-8.5.28/bin/shutdown.sh: time stamp 2018-02-07 07:10:57 is 18850945.452612035 s in the future
apache-tomcat-8.5.28/bin/startup.sh
tar: apache-tomcat-8.5.28/bin/startup.sh: time stamp 2018-02-07 07:10:57 is 18850945.452535395 s in the future
apache-tomcat-8.5.28/bin/tool-wrapper.sh
tar: apache-tomcat-8.5.28/bin/tool-wrapper.sh: time stamp 2018-02-07 07:10:57 is 18850945.452471234 s in the future
apache-tomcat-8.5.28/bin/version.sh
tar: apache-tomcat-8.5.28/bin/version.sh: time stamp 2018-02-07 07:10:57 is 18850945.452402751 s in the future
[root@h210 tomcat]# echo $?
0
[root@h210 tomcat]#
~~~


## 启动服务

~~~
[root@h210 ~]# cd /data/apache-tomcat-8.5.28/bin/
[root@h210 bin]# pwd
/data/apache-tomcat-8.5.28/bin
[root@h210 bin]# ./catalina.sh start
Using CATALINA_BASE:   /data/apache-tomcat-8.5.28
Using CATALINA_HOME:   /data/apache-tomcat-8.5.28
Using CATALINA_TMPDIR: /data/apache-tomcat-8.5.28/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /data/apache-tomcat-8.5.28/bin/bootstrap.jar:/data/apache-tomcat-8.5.28/bin/tomcat-juli.jar
Tomcat started.
[root@h210 bin]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1723/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1525/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1526/cupsd          
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1643/master         
tcp        0      0 192.168.56.210:22       192.168.56.1:47770      ESTABLISHED 4218/sshd: root@pts
tcp        0      0 192.168.56.210:22       192.168.56.1:47762      ESTABLISHED 4144/sshd: root@pts
tcp6       0      0 :::8009                 :::*                    LISTEN      6073/java           
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::8080                 :::*                    LISTEN      6073/java           
tcp6       0      0 :::22                   :::*                    LISTEN      1525/sshd           
tcp6       0      0 ::1:631                 :::*                    LISTEN      1526/cupsd          
tcp6       0      0 ::1:25                  :::*                    LISTEN      1643/master         
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      6073/java           
tcp6       0      0 ::1:41270               ::1:8080                TIME_WAIT   -                   
tcp6       0      0 ::1:57594               ::1:8009                TIME_WAIT   -                   
tcp6       0      0 127.0.0.1:52082         127.0.0.1:8005          TIME_WAIT   -                   
[root@h210 bin]#
[root@h210 bin]# ps faux | grep tomcat
root      6130  0.0  0.0 112648  1016 pts/0    S+   03:12   0:00  |       \_ grep --color=auto tomcat
root      6073  4.4  4.9 3072396 100460 pts/0  Sl   03:11   0:02 /usr/bin/java -Djava.util.logging.config.file=/data/apache-tomcat-8.5.28/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /data/apache-tomcat-8.5.28/bin/bootstrap.jar:/data/apache-tomcat-8.5.28/bin/tomcat-juli.jar -Dcatalina.base=/data/apache-tomcat-8.5.28 -Dcatalina.home=/data/apache-tomcat-8.5.28 -Djava.io.tmpdir=/data/apache-tomcat-8.5.28/temp org.apache.catalina.startup.Bootstrap start
[root@h210 bin]#
~~~


## 打开防火墙

~~~
[root@h210 ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@h210 ~]# firewall-cmd --add-port 8080/tcp --permanent
success
[root@h210 ~]# firewall-cmd --reload
success
[root@h210 ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 8080/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@h210 ~]#
~~~


## 进行访问


**`http://192.168.56.210:8080/`**


![tomcat](/assets/img/tomcat/tomcat01.png)


---

# 总结

由于 JAVA 的跨平台性，tomcat 的安装其实就是一个解压运行的过程

关于 tomcat 的配置，后面有需要，再进行展开

* TOC
{:toc}


---


[tomcat]:http://tomcat.apache.org/
