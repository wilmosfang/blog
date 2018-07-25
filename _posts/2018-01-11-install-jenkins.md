---
layout: post
title: "Install Jenkins"
author:  wilmosfang
date: 2018-01-11 15:08:26
image: '/assets/img/'
excerpt: 'Jenkins 的安装方法'
main-class: 'jenkins'
color: '#1077ad'
tags:
 - jenkins
 - ci
 - cd
categories:
 - jenkins
twitter_text: 'jenkins install simple process'
introduction: 'installation method of Jenkins'
---


# 前言


**[Jenkins][jenkins]** 是一套自动化软件，结合不同的插件可以轻易实现 **CI/CD** 工作流


>Build great things at any scale
>
>The leading open source automation server, Jenkins provides hundreds of plugins to support building, deploying and automating any project.

**[Jenkins][jenkins]** 与 **k8s** 还有 **Gitlab** 常常放在一起构建持续集成系统

下面分享一下 **[Jenkins][jenkins]** 的基础安装操作

参考 **[Installing Jenkins on Red Hat distributions][jenkins_doc]**

> **Tip:** 当前版本 **Jenkins 2.89.2 LTS**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: f91fb2eedcd345cf9cd7e6d80aab5115
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
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d1:5d:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83782sec preferred_lft 83782sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:47:20:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 依赖


以下版本的 OpenJDK 是 **[Jenkins][jenkins]** 的运行环境

>You will need to explicitly install a Java runtime environment, because Oracle's Java RPMs are incorrect and fail to register as providing a java dependency. Thus, adding an explicit dependency requirement on Java would force installation of the OpenJDK JVM.

* 2.54 (2017-04) and newer: Java 8
* 1.612 (2015-05) and newer: Java 7

~~~
[root@much ~]# java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-b16)
OpenJDK 64-Bit Server VM (build 25.141-b16, mixed mode)
[root@much ~]#
[root@much ~]# rpm -qa | grep -i jdk
java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64
java-1.7.0-openjdk-headless-1.7.0.141-2.6.10.1.el7_3.x86_64
java-1.7.0-openjdk-1.7.0.141-2.6.10.1.el7_3.x86_64
java-1.8.0-openjdk-headless-1.8.0.141-1.b16.el7_3.x86_64
java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64
copy-jdk-configs-1.2-1.el7.noarch
[root@much ~]# yum list all | grep -i openjdk
java-1.7.0-openjdk.x86_64               1:1.7.0.141-2.6.10.1.el7_3     @updates
java-1.7.0-openjdk-headless.x86_64      1:1.7.0.141-2.6.10.1.el7_3     @updates
java-1.8.0-openjdk.x86_64               1:1.8.0.141-1.b16.el7_3        @updates
java-1.8.0-openjdk-devel.x86_64         1:1.8.0.141-1.b16.el7_3        @updates
java-1.8.0-openjdk-headless.x86_64      1:1.8.0.141-1.b16.el7_3        @updates
java-1.6.0-openjdk.x86_64               1:1.6.0.41-1.13.13.1.el7_3     base     
java-1.6.0-openjdk-demo.x86_64          1:1.6.0.41-1.13.13.1.el7_3     base     
java-1.6.0-openjdk-devel.x86_64         1:1.6.0.41-1.13.13.1.el7_3     base     
java-1.6.0-openjdk-javadoc.x86_64       1:1.6.0.41-1.13.13.1.el7_3     base     
java-1.6.0-openjdk-src.x86_64           1:1.6.0.41-1.13.13.1.el7_3     base     
java-1.7.0-openjdk.x86_64               1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.7.0-openjdk-accessibility.x86_64 1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.7.0-openjdk-demo.x86_64          1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.7.0-openjdk-devel.x86_64         1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.7.0-openjdk-headless.x86_64      1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.7.0-openjdk-javadoc.noarch       1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.7.0-openjdk-src.x86_64           1:1.7.0.161-2.6.12.0.el7_4     updates  
java-1.8.0-openjdk.i686                 1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk.x86_64               1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-accessibility.i686   1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-accessibility.x86_64 1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-accessibility-debug.i686
java-1.8.0-openjdk-accessibility-debug.x86_64
java-1.8.0-openjdk-debug.i686           1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-debug.x86_64         1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-demo.i686            1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-demo.x86_64          1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-demo-debug.i686      1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-demo-debug.x86_64    1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-devel.i686           1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-devel.x86_64         1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-devel-debug.i686     1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-devel-debug.x86_64   1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-headless.i686        1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-headless.x86_64      1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-headless-debug.i686  1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-headless-debug.x86_64
java-1.8.0-openjdk-javadoc.noarch       1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-javadoc-debug.noarch 1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-javadoc-zip.noarch   1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-javadoc-zip-debug.noarch
java-1.8.0-openjdk-src.i686             1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-src.x86_64           1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-src-debug.i686       1:1.8.0.151-5.b12.el7_4        updates  
java-1.8.0-openjdk-src-debug.x86_64     1:1.8.0.151-5.b12.el7_4        updates  
[root@much ~]# yum install java-1.8.0-openjdk.x86_64
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 7.4 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/4): extras/7/x86_64/primary_db                          | 145 kB   00:01     
(2/4): updates/7/x86_64/primary_db                         | 5.2 MB   00:25     
(3/4): epel/x86_64/updateinfo                              | 869 kB   00:39     
(4/4): epel/x86_64/primary_db                              | 6.2 MB   00:44     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror1.ku.ac.th
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.141-1.b16.el7_3 will be updated
--> Processing Dependency: java-1.8.0-openjdk = 1:1.8.0.141-1.b16.el7_3 for package: 1:java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.151-5.b12.el7_4 will be an update
--> Processing Dependency: java-1.8.0-openjdk-headless(x86-64) = 1:1.8.0.151-5.b12.el7_4 for package: 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64
--> Running transaction check
---> Package java-1.8.0-openjdk-devel.x86_64 1:1.8.0.141-1.b16.el7_3 will be updated
---> Package java-1.8.0-openjdk-devel.x86_64 1:1.8.0.151-5.b12.el7_4 will be an update
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.141-1.b16.el7_3 will be updated
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.151-5.b12.el7_4 will be an update
--> Processing Dependency: nss-softokn(x86-64) >= 3.28.3 for package: 1:java-1.8.0-openjdk-headless-1.8.0.151-5.b12.el7_4.x86_64
--> Processing Dependency: copy-jdk-configs >= 2.2 for package: 1:java-1.8.0-openjdk-headless-1.8.0.151-5.b12.el7_4.x86_64
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

================================================================================
 Package                      Arch    Version                    Repository
                                                                           Size
================================================================================
Updating:
 java-1.8.0-openjdk           x86_64  1:1.8.0.151-5.b12.el7_4    updates  241 k
Updating for dependencies:
 copy-jdk-configs             noarch  2.2-5.el7_4                updates   19 k
 java-1.8.0-openjdk-devel     x86_64  1:1.8.0.151-5.b12.el7_4    updates  9.8 M
 java-1.8.0-openjdk-headless  x86_64  1:1.8.0.151-5.b12.el7_4    updates   32 M
 nss-softokn                  x86_64  3.28.3-8.el7_4             updates  310 k
 nss-softokn-freebl           x86_64  3.28.3-8.el7_4             updates  214 k

Transaction Summary
================================================================================
Upgrade  1 Package (+5 Dependent packages)

Total download size: 42 M
Is this ok [y/d/N]: y
Downloading packages:
updates/7/x86_64/prestodelta                               | 646 kB   00:03     
Delta RPMs reduced 11 M of updates to 946 k (91% saved)
(1/6): copy-jdk-configs-2.2-5.el7_4.noarch.rpm             |  19 kB   00:00     
(2/6): java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3_1.8.0.151- |  69 kB   00:01     
(3/6): nss-softokn-3.16.2.3-14.4.el7_3.28.3-8.el7_4.x86_64 | 174 kB   00:01     
(4/6): nss-softokn-freebl-3.16.2.3-14.4.el7_3.28.3-8.el7_4 |  95 kB   00:01     
(5/6): java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3_1.8. | 608 kB   00:04     
(6/6): java-1.8.0-openjdk-headless-1.8.0.151-5.b12.el7_4.x |  32 MB   01:36     
--------------------------------------------------------------------------------
Total                                              342 kB/s |  33 MB  01:37     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : nss-softokn-freebl-3.28.3-8.el7_4.x86_64                    1/12
  Updating   : nss-softokn-3.28.3-8.el7_4.x86_64                           2/12
  Updating   : copy-jdk-configs-2.2-5.el7_4.noarch                         3/12
  Updating   : 1:java-1.8.0-openjdk-headless-1.8.0.151-5.b12.el7_4.x86_    4/12
warning: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/java.security created as /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/java.security.rpmnew
warning: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/nss.cfg created as /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/nss.cfg.rpmnew
restored /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/java.security.rpmnew to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/java.security
restored /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/nss.cfg.rpmnew to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/security/nss.cfg
  Updating   : 1:java-1.8.0-openjdk-devel-1.8.0.151-5.b12.el7_4.x86_64     5/12
  Updating   : 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64           6/12
  Cleanup    : 1:java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64     7/12
  Cleanup    : 1:java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64           8/12
  Cleanup    : 1:java-1.8.0-openjdk-headless-1.8.0.141-1.b16.el7_3.x86_    9/12
  Cleanup    : nss-softokn-3.16.2.3-14.4.el7.x86_64                       10/12
  Cleanup    : copy-jdk-configs-1.2-1.el7.noarch                          11/12
  Cleanup    : nss-softokn-freebl-3.16.2.3-14.4.el7.x86_64                12/12
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64           1/12
  Verifying  : 1:java-1.8.0-openjdk-devel-1.8.0.151-5.b12.el7_4.x86_64     2/12
  Verifying  : nss-softokn-3.28.3-8.el7_4.x86_64                           3/12
  Verifying  : copy-jdk-configs-2.2-5.el7_4.noarch                         4/12
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.151-5.b12.el7_4.x86_    5/12
  Verifying  : nss-softokn-freebl-3.28.3-8.el7_4.x86_64                    6/12
  Verifying  : 1:java-1.8.0-openjdk-devel-1.8.0.141-1.b16.el7_3.x86_64     7/12
  Verifying  : nss-softokn-freebl-3.16.2.3-14.4.el7.x86_64                 8/12
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64           9/12
  Verifying  : nss-softokn-3.16.2.3-14.4.el7.x86_64                       10/12
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.141-1.b16.el7_3.x86_   11/12
  Verifying  : copy-jdk-configs-1.2-1.el7.noarch                          12/12

Updated:
  java-1.8.0-openjdk.x86_64 1:1.8.0.151-5.b12.el7_4                             

Dependency Updated:
  copy-jdk-configs.noarch 0:2.2-5.el7_4                                         
  java-1.8.0-openjdk-devel.x86_64 1:1.8.0.151-5.b12.el7_4                       
  java-1.8.0-openjdk-headless.x86_64 1:1.8.0.151-5.b12.el7_4                    
  nss-softokn.x86_64 0:3.28.3-8.el7_4                                           
  nss-softokn-freebl.x86_64 0:3.28.3-8.el7_4                                    

Complete!
[root@much ~]# java -version
openjdk version "1.8.0_151"
OpenJDK Runtime Environment (build 1.8.0_151-b12)
OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)
[root@much ~]#
~~~


## 安装库文件

~~~
[root@much ~]# sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
--2018-01-11 23:44:53--  https://pkg.jenkins.io/redhat-stable/jenkins.repo
Resolving pkg.jenkins.io (pkg.jenkins.io)... 52.202.51.185
Connecting to pkg.jenkins.io (pkg.jenkins.io)|52.202.51.185|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 85
Saving to: ‘/etc/yum.repos.d/jenkins.repo’

100%[======================================>] 85          --.-K/s   in 0s      

2018-01-11 23:44:55 (3.30 MB/s) - ‘/etc/yum.repos.d/jenkins.repo’ saved [85/85]

[root@much ~]# ll /etc/yum.repos.d/jenkins.repo
-rw-r--r-- 1 root root 85 11月 30 2016 /etc/yum.repos.d/jenkins.repo
[root@much ~]# cat /etc/yum.repos.d/jenkins.repo
[jenkins]
name=Jenkins-stable
baseurl=http://pkg.jenkins.io/redhat-stable
gpgcheck=1
[root@much ~]# sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
[root@much ~]#
~~~



## 下载与安装软件包


~~~
[root@much ~]# yum install jenkins
Loaded plugins: fastestmirror, langpacks
jenkins                                                  | 2.9 kB     00:00     
jenkins/primary_db                                         |  22 kB   00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: ftp.jaist.ac.jp
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package jenkins.noarch 0:2.89.2-1.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch            Version                Repository        Size
================================================================================
Installing:
 jenkins          noarch          2.89.2-1.1             jenkins           71 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 71 M
Installed size: 71 M
Is this ok [y/d/N]: y
Downloading packages:
jenkins-2.89.2-1.1.noarch.rpm                              |  71 MB   03:45     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : jenkins-2.89.2-1.1.noarch                                    1/1
  Verifying  : jenkins-2.89.2-1.1.noarch                                    1/1

Installed:
  jenkins.noarch 0:2.89.2-1.1                                                   

Complete!
[root@much ~]#
~~~

## 服务的启停

~~~
[root@much ~]# service jenkins status
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:systemd-sysv-generator(8)
[root@much ~]# service jenkins start
Starting jenkins (via systemctl):                          [  OK  ]
[root@much ~]# service jenkins status
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since Thu 2018-01-11 23:57:39 CST; 3s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 2981 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/jenkins.service
           └─2998 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -D...

Jan 11 23:57:39 much systemd[1]: Starting LSB: Jenkins Automation Server...
Jan 11 23:57:39 much runuser[2982]: pam_unix(runuser:session): session open...0)
Jan 11 23:57:39 much jenkins[2981]: Starting Jenkins [  OK  ]
Jan 11 23:57:39 much systemd[1]: Started LSB: Jenkins Automation Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]# service jenkins stop
Stopping jenkins (via systemctl):                          [  OK  ]
[root@much ~]# service jenkins status
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: inactive (dead) since Thu 2018-01-11 23:57:49 CST; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3086 ExecStop=/etc/rc.d/init.d/jenkins stop (code=exited, status=0/SUCCESS)
  Process: 2981 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)

Jan 11 23:57:39 much systemd[1]: Starting LSB: Jenkins Automation Server...
Jan 11 23:57:39 much runuser[2982]: pam_unix(runuser:session): session open...0)
Jan 11 23:57:39 much jenkins[2981]: Starting Jenkins [  OK  ]
Jan 11 23:57:39 much systemd[1]: Started LSB: Jenkins Automation Server.
Jan 11 23:57:48 much systemd[1]: Stopping LSB: Jenkins Automation Server...
Jan 11 23:57:49 much jenkins[3086]: Shutting down Jenkins [  OK  ]
Jan 11 23:57:49 much systemd[1]: Stopped LSB: Jenkins Automation Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much ~]#
~~~

**[Jenkins 的安装地址][jenkins_dl]**


## 附加信息

* **Jenkins** 会作为一个守护进程在后台启动. **/etc/init.d/jenkins** 里有详细信息.

~~~
[root@much ~]# cat /etc/init.d/jenkins
#!/bin/sh
#
#     SUSE system statup script for Jenkins
#     Copyright (C) 2007  Pascal Bleser
#
#     This library is free software; you can redistribute it and/or modify it
#     under the terms of the GNU Lesser General Public License as published by
#     the Free Software Foundation; either version 2.1 of the License, or (at
#     your option) any later version.
#
#     This library is distributed in the hope that it will be useful, but
#     WITHOUT ANY WARRANTY; without even the implied warranty of
#     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
#     Lesser General Public License for more details.
#
#     You should have received a copy of the GNU Lesser General Public
#     License along with this library; if not, write to the Free Software
#     Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307,
#     USA.
#
### BEGIN INIT INFO
# Provides:          jenkins
# Required-Start:    $local_fs $remote_fs $network $time $named
# Should-Start: $time sendmail
# Required-Stop:     $local_fs $remote_fs $network $time $named
# Should-Stop: $time sendmail
# Default-Start:     3 5
# Default-Stop:      0 1 2 6
# Short-Description: Jenkins Automation Server
# Description:       Jenkins Automation Server
### END INIT INFO

# Check for missing binaries (stale symlinks should not happen)
JENKINS_WAR="/usr/lib/jenkins/jenkins.war"
test -r "$JENKINS_WAR" || { echo "$JENKINS_WAR not installed";
	if [ "$1" = "stop" ]; then exit 0;
	else exit 5; fi; }

# Check for existence of needed config file and read it
JENKINS_CONFIG=/etc/sysconfig/jenkins
test -e "$JENKINS_CONFIG" || { echo "$JENKINS_CONFIG not existing";
	if [ "$1" = "stop" ]; then exit 0;
	else exit 6; fi; }
test -r "$JENKINS_CONFIG" || { echo "$JENKINS_CONFIG not readable. Perhaps you forgot 'sudo'?";
	if [ "$1" = "stop" ]; then exit 0;
	else exit 6; fi; }

JENKINS_PID_FILE="/var/run/jenkins.pid"

# Source function library.
. /etc/init.d/functions

# Read config
[ -f "$JENKINS_CONFIG" ] && . "$JENKINS_CONFIG"

# Set up environment accordingly to the configuration settings
[ -n "$JENKINS_HOME" ] || { echo "JENKINS_HOME not configured in $JENKINS_CONFIG";
	if [ "$1" = "stop" ]; then exit 0;
	else exit 6; fi; }
[ -d "$JENKINS_HOME" ] || { echo "JENKINS_HOME directory does not exist: $JENKINS_HOME";
	if [ "$1" = "stop" ]; then exit 0;
	else exit 1; fi; }

# Search usable Java as /usr/bin/java might not point to minimal version required by Jenkins.
# see http://www.nabble.com/guinea-pigs-wanted-----Hudson-RPM-for-RedHat-Linux-td25673707.html
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/bin/java
"
for candidate in $candidates
do
  [ -x "$JENKINS_JAVA_CMD" ] && break
  JENKINS_JAVA_CMD="$candidate"
done

JAVA_CMD="$JENKINS_JAVA_CMD $JENKINS_JAVA_OPTIONS -DJENKINS_HOME=$JENKINS_HOME -jar $JENKINS_WAR"
PARAMS="--logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon"
[ -n "$JENKINS_PORT" ] && PARAMS="$PARAMS --httpPort=$JENKINS_PORT"
[ -n "$JENKINS_LISTEN_ADDRESS" ] && PARAMS="$PARAMS --httpListenAddress=$JENKINS_LISTEN_ADDRESS"
[ -n "$JENKINS_HTTPS_PORT" ] && PARAMS="$PARAMS --httpsPort=$JENKINS_HTTPS_PORT"
[ -n "$JENKINS_HTTPS_KEYSTORE" ] && PARAMS="$PARAMS --httpsKeyStore=$JENKINS_HTTPS_KEYSTORE"
[ -n "$JENKINS_HTTPS_KEYSTORE_PASSWORD" ] && PARAMS="$PARAMS --httpsKeyStorePassword='$JENKINS_HTTPS_KEYSTORE_PASSWORD'"
[ -n "$JENKINS_HTTPS_LISTEN_ADDRESS" ] && PARAMS="$PARAMS --httpsListenAddress=$JENKINS_HTTPS_LISTEN_ADDRESS"
[ -n "$JENKINS_DEBUG_LEVEL" ] && PARAMS="$PARAMS --debug=$JENKINS_DEBUG_LEVEL"
[ -n "$JENKINS_HANDLER_STARTUP" ] && PARAMS="$PARAMS --handlerCountStartup=$JENKINS_HANDLER_STARTUP"
[ -n "$JENKINS_HANDLER_MAX" ] && PARAMS="$PARAMS --handlerCountMax=$JENKINS_HANDLER_MAX"
[ -n "$JENKINS_HANDLER_IDLE" ] && PARAMS="$PARAMS --handlerCountMaxIdle=$JENKINS_HANDLER_IDLE"
[ -n "$JENKINS_ARGS" ] && PARAMS="$PARAMS $JENKINS_ARGS"

if [ "$JENKINS_ENABLE_ACCESS_LOG" = "yes" ]; then
    PARAMS="$PARAMS --accessLoggerClassName=winstone.accesslog.SimpleAccessLogger --simpleAccessLogger.format=combined --simpleAccessLogger.file=/var/log/jenkins/access_log"
fi

RETVAL=0

case "$1" in
    start)
	echo -n "Starting Jenkins "
	daemon --user "$JENKINS_USER" --pidfile "$JENKINS_PID_FILE" $JAVA_CMD $PARAMS > /dev/null
	RETVAL=$?
	if [ $RETVAL = 0 ]; then
	    success
	    echo > "$JENKINS_PID_FILE"  # just in case we fail to find it
            MY_SESSION_ID=`/bin/ps h -o sess -p $$`
            # get PID
            /bin/ps hww -u "$JENKINS_USER" -o sess,ppid,pid,cmd | \
            while read sess ppid pid cmd; do
		[ "$ppid" = 1 ] || continue
		# this test doesn't work because Jenkins sets a new Session ID
                # [ "$sess" = "$MY_SESSION_ID" ] || continue
	       	echo "$cmd" | grep $JENKINS_WAR > /dev/null
		[ $? = 0 ] || continue
		# found a PID
		echo $pid > "$JENKINS_PID_FILE"
	    done
	else
	    failure
	fi
	echo
	;;
    stop)
	echo -n "Shutting down Jenkins "
	killproc jenkins
	RETVAL=$?
	echo
	;;
    try-restart|condrestart)
	if test "$1" = "condrestart"; then
		echo "${attn} Use try-restart ${done}(LSB)${attn} rather than condrestart ${warn}(RH)${norm}"
	fi
	$0 status
	if test $? = 0; then
		$0 restart
	else
		: # Not running is not a failure.
	fi
	;;
    restart)
	$0 stop
	$0 start
	;;
    force-reload)
	echo -n "Reload service Jenkins "
	$0 try-restart
	;;
    reload)
    	$0 restart
	;;
    status)
    	status jenkins
	RETVAL=$?
	;;
    probe)
	## Optional: Probe for the necessity of a reload, print out the
	## argument to this init script which is required for a reload.
	## Note: probe is not (yet) part of LSB (as of 1.9)

	test "$JENKINS_CONFIG" -nt "$JENKINS_PID_FILE" && echo reload
	;;
    *)
	echo "Usage: $0 {start|stop|status|try-restart|restart|force-reload|reload|probe}"
	exit 1
	;;
esac
exit $RETVAL
[root@much ~]#
~~~


* 服务是以 **jenkins** 的用户身份运行的 . 如果要在配置文件中改成另一个用户身份　, 必须同时改变以下几个文件的所有者 **/var/log/jenkins, /var/lib/jenkins, and /var/cache/jenkins**.

~~~
[root@much ~]# tail -n5 /etc/passwd
vboxadd:x:987:1::/var/run/vboxadd:/bin/false
hadoop:x:1001:1001::/home/hadoop:/bin/bash
dockerroot:x:986:981:Docker User:/var/lib/docker:/sbin/nologin
etcd:x:985:980:etcd user:/var/lib/etcd:/sbin/nologin
jenkins:x:984:979:Jenkins Automation Server:/var/lib/jenkins:/bin/false
[root@much ~]# ll /var/log/jenkins/
total 8
-rw-r--r-- 1 jenkins jenkins 5499 1月  11 23:57 jenkins.log
[root@much ~]# ll /var/lib/jenkins/
total 40
-rw-r--r-- 1 jenkins jenkins 1719 1月  11 23:57 config.xml
-rw-r--r-- 1 jenkins jenkins   29 1月  11 23:57 failed-boot-attempts.txt
-rw-r--r-- 1 jenkins jenkins  156 1月  11 23:57 hudson.model.UpdateCenter.xml
-rw------- 1 jenkins jenkins 1712 1月  11 23:57 identity.key.enc
-rw-r--r-- 1 jenkins jenkins   94 1月  11 23:57 jenkins.CLI.xml
-rw-r--r-- 1 jenkins jenkins    6 1月  11 23:57 jenkins.install.UpgradeWizard.state
drwxr-xr-x 2 jenkins jenkins    6 1月  11 23:57 jobs
drwxr-xr-x 3 jenkins jenkins   19 1月  11 23:57 logs
-rw-r--r-- 1 jenkins jenkins  907 1月  11 23:57 nodeMonitors.xml
drwxr-xr-x 2 jenkins jenkins    6 1月  11 23:57 nodes
drwxr-xr-x 2 jenkins jenkins    6 1月  11 23:57 plugins
-rw-r--r-- 1 jenkins jenkins  129 1月  11 23:57 queue.xml
-rw-r--r-- 1 jenkins jenkins   64 1月  11 23:57 secret.key
-rw-r--r-- 1 jenkins jenkins    0 1月  11 23:57 secret.key.not-so-secret
drwxr-xr-x 4 jenkins jenkins 4096 1月  11 23:57 secrets
drwxr-xr-x 2 jenkins jenkins   24 1月  11 23:57 userContent
drwxr-xr-x 3 jenkins jenkins   19 1月  11 23:57 users
[root@much ~]# ll /var/cache/jenkins/
total 4
drwxr-xr-x 10 jenkins jenkins 4096 1月  11 23:57 war
[root@much ~]#
~~~

* 日志文件的位置为 /var/log/jenkins/jenkins.log　可以使用这个文件来跟踪进程状态.

~~~
[root@much ~]# tail /var/log/jenkins/jenkins.log
Please use the following password to proceed to installation:

7b4967b63df148daa7e4cebf17f1b6d7

This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

[root@much ~]# ll /var/log/jenkins/jenkins.log
-rw-r--r-- 1 jenkins jenkins 5499 1月  11 23:57 /var/log/jenkins/jenkins.log
[root@much ~]#
~~~

* /etc/sysconfig/jenkins 是进程的启动配置文件.　默认情况下服务监听在　8080　端口. 通过访问此端口进行服务配置.  

~~~
[root@much ~]# service  jenkins  start
Starting jenkins (via systemctl):                          [  OK  ]
[root@much ~]# netstat -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 10.0.2.15:55536         52.202.51.185:443       ESTABLISHED
tcp        0      0 192.168.56.208:22       192.168.56.1:58640      ESTABLISHED
tcp        0      0 192.168.56.208:22       192.168.56.1:52348      ESTABLISHED
tcp        0      0 192.168.56.208:22       192.168.56.1:58726      ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN     
[root@much ~]#
~~~

* **Jenkins** 的 RPM 库为 /etc/yum.repos.d/jenkins.repo

~~~
[root@much ~]# cat /etc/yum.repos.d/jenkins.repo
[jenkins]
name=Jenkins-stable
baseurl=http://pkg.jenkins.io/redhat-stable
gpgcheck=1
[root@much ~]#
~~~

>**Note:** that the built-in firewall may have to be opened to access this port from other computers.  (See http://www.cyberciti.biz/faq/disable-linux-firewall-under-centos-rhel-fedora/ for instructions how to disable the firewall permanently)


## 打开防火墙

~~~
[root@much ~]# firewall-cmd --list-all
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

[root@much ~]#
~~~

可以使用下面方式打开

~~~
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
~~~

## 进行访问

出现登录界面

![jenkins](/assets/img/jenkins/jenkins1.png)

新版本中对安全有所加强

第一次的登录密码是输出到 **/var/lib/jenkins/secrets/initialAdminPassword** 文件中的

~~~
[root@much jenkins]# ll /var/log/jenkins/jenkins.log
-rw-r--r-- 1 jenkins jenkins 10652 1月  12 00:17 /var/log/jenkins/jenkins.log
[root@much jenkins]#
[root@much jenkins]# grep -C 6  password  /var/log/jenkins/jenkins.log
INFO:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

7b4967b63df148daa7e4cebf17f1b6d7

This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword

*************************************************************
[root@much jenkins]# cat /var/lib/jenkins/secrets/initialAdminPassword
7b4967b63df148daa7e4cebf17f1b6d7
[root@much jenkins]#
~~~

## 管理界面

![jenkins](/assets/img/jenkins/jenkins2.png)

---

# 总结

总体来讲遵循一个典型的软件包安装流程

* TOC
{:toc}


---

[jenkins]:https://jenkins.io/
[jenkins_doc]:https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions
[jenkins_dl]:https://pkg.jenkins.io/redhat-stable/
