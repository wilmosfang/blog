---
layout: post
title:  Jenkins 基础 
categories: linux  Jenkins 
wc: 283 871 10039
excerpt:  Jenkins 的安装与基础
comments: true
---


---

# 前言


**[Jenkins][jenkins]** 是一个现在使用相当广泛的持续集成，持续交付开源工具，网络公司的快速迭代都会使用到此类工具


>Jenkins is an award-winning, cross-platform, continuous integration and continuous delivery application that increases your productivity. Use Jenkins to build and test your software projects continuously making it easier for developers to integrate changes to the project, and making it easier for users to obtain a fresh build. It also allows you to continuously deliver your software by providing powerful ways to define your build pipelines and integrating with a large number of testing and deployment technologies.

**[jenkins][jenkins]** 是自动运维的经典代表，下面分享一下它的基础操作，详细可以参阅 **[官方文档][jenkins_doc]**

> **Tip:** 当前的最新版本为 **jenkins 1.642**

---


# 概要

* TOC
{:toc}



---

##安装


最新版的安装

{% highlight bash %}
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
{% endhighlight %}

稳定版的安装


{% highlight bash %}
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
{% endhighlight %}

安装过程

{% highlight bash %}
[root@h101 ~]# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
--2015-12-17 19:42:11--  http://pkg.jenkins-ci.org/redhat/jenkins.repo
Resolving pkg.jenkins-ci.org... 199.193.196.24
Connecting to pkg.jenkins-ci.org|199.193.196.24|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 75 [text/plain]
Saving to: “/etc/yum.repos.d/jenkins.repo”

100%[============================================================================================>] 75          --.-K/s   in 0s      

2015-12-17 19:42:12 (6.28 MB/s) - “/etc/yum.repos.d/jenkins.repo” saved [75/75]

[root@h101 ~]# rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
[root@h101 ~]# echo $?
0
[root@h101 ~]# yum install jenkins
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package jenkins.noarch 0:1.642-1.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                        Arch                          Version                            Repository                      Size
======================================================================================================================================
Installing:
 jenkins                        noarch                        1.642-1.1                          jenkins                         61 M

Transaction Summary
======================================================================================================================================
Install       1 Package(s)

Total download size: 61 M
Installed size: 61 M
Is this ok [y/N]: y
Downloading Packages:
jenkins-1.642-1.1.noarch.rpm                         57% [========================                  ]  48 kB/s |  35 MB     09:07 ETA 
jenkins-1.642-1.1.noarch.rpm                         57% [======================
jenkins-1.642-1.1.noarch.rpm                                                                                   |  61 MB     13:52     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : jenkins-1.642-1.1.noarch                                                                                           1/1 
  Verifying  : jenkins-1.642-1.1.noarch                                                                                           1/1 

Installed:
  jenkins.noarch 0:1.642-1.1                                                                                                          

Complete!
[root@h101 ~]# 

{% endhighlight %}

**/etc/yum.repos.d/jenkins.repo** 是一个Jenkins RPM 仓库

---

##依赖

Jenkins 的运行需要 Java 环境 ，**`yum install jenkins`** 并不检查并且强制要求  java 已经正确安装，正常安装完jenkins并不代表其可以正常运行，使用下面的方式来确认当前环境下java的版本

{% highlight bash %}
[root@h101 ~]# java -version
java version "1.7.0_65"
OpenJDK Runtime Environment (rhel-2.5.1.2.el6_5-x86_64 u65-b17)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
[root@h101 ~]# 
{% endhighlight %}


> **Note:**  To further make things difficult for CentOS users, the default CentOS version of Java is not compatible with Jenkins. Jenkins typically works best with a Sun implementation of Java, which is not included in CentOS for licensing reasons

CentOS 版本的java 与Jenkins并不兼容

{% highlight bash %}
java -version
java version "1.5.0"
gij (GNU libgcj) version 4.4.6 20110731 (Red Hat 4.4.6-3)
{% endhighlight %}

EPEL仓库里提供的OpenJDK 可以很好的支持Jenkins

{% highlight bash %}
java -version
java version "1.7.0_79"
OpenJDK Runtime Environment (rhel-2.5.5.1.el6_6-x86_64 u79-b14)
OpenJDK 64-Bit Server VM (build 24.79-b02, mixed mode)
{% endhighlight %}

如果出现java的相关报错，可以先尝试找找java版本的原因

---

##防火墙

{% highlight bash %}
[root@h101 ~]# vim /etc/sysconfig/iptables
[root@h101 ~]# grep 8080 /etc/sysconfig/iptables
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080  -j ACCEPT 
[root@h101 ~]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h101 ~]# iptables -L -nv | grep 8080
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:8080 
[root@h101 ~]# 
{% endhighlight %}

默认情况下Jenkins监听在8080端口，所以要打开此端口，否则服务将无法访问



---

##启停

###启动

{% highlight bash %}
[root@h101 ~]# /etc/init.d/jenkins start 
Starting Jenkins                                           [  OK  ]
[root@h101 ~]# ps faux | grep jenkins
root      4064  0.0  0.0 103256   828 pts/0    S+   20:32   0:00  |       \_ grep jenkins
jenkins   3973 88.6 13.9 2206040 266288 ?      Ssl  20:32   0:17 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --ajp13Port=8009 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
[root@h101 ~]# netstat  -ant | grep 8080
tcp        0      0 :::8080                     :::*                        LISTEN      
tcp        0      0 ::ffff:192.168.100.101:8080 ::ffff:192.168.100.1:58628  TIME_WAIT   
[root@h101 ~]# 
{% endhighlight %}

* Jenkins 会作为一个服务在系统后台运行
* **/etc/init.d/jenkins** 会提供详细信息，包括实际运行了什么，配置文件的位置
* 初始配置在 **/etc/sysconfig/jenkins** 中
* 默认情况下Jenkins会监听在 **8080** 端口(可以使用/etc/sysconfig/jenkins修改)，要打开防火墙 ， 可以使用本地的浏览器进行访问
* **jenkins** 用户会被创建，并且以它的身份运行服务

> **Note:** If you change this to a different user via the config file, you must change the owner of /var/log/jenkins, /var/lib/jenkins, and /var/cache/jenkins

*  日志会被记到 **/var/log/jenkins/jenkins.log** 中


###配置

{% highlight bash %}
[root@h101 ~]# grep -v "^#" /etc/sysconfig/jenkins  | grep -v "^$"
JENKINS_HOME="/var/lib/jenkins"
JENKINS_JAVA_CMD=""
JENKINS_USER="jenkins"
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"
JENKINS_PORT="8080"
JENKINS_LISTEN_ADDRESS=""
JENKINS_HTTPS_PORT=""
JENKINS_HTTPS_KEYSTORE=""
JENKINS_HTTPS_KEYSTORE_PASSWORD=""
JENKINS_HTTPS_LISTEN_ADDRESS=""
JENKINS_AJP_PORT="8009"
JENKINS_AJP_LISTEN_ADDRESS=""
JENKINS_DEBUG_LEVEL="5"
JENKINS_ENABLE_ACCESS_LOG="no"
JENKINS_HANDLER_MAX="100"
JENKINS_HANDLER_IDLE="20"
JENKINS_ARGS=""
[root@h101 ~]# 
{% endhighlight %}


###操作界面

![jenkins1.png](/images/jenkins.png)

###停止

{% highlight bash %}
[root@h101 ~]# /etc/init.d/jenkins stop 
Shutting down Jenkins                                      [  OK  ]
[root@h101 ~]# ps faux | grep jenkins
root      4079  0.0  0.0 103256   828 pts/0    S+   20:34   0:00  |       \_ grep jenkins
[root@h101 ~]# netstat  -ant | grep 8080
[root@h101 ~]#  
{% endhighlight %}

###开机启动


{% highlight bash %}
[root@h101 ~]# chkconfig  --list | grep jenkins
jenkins        	0:off	1:off	2:off	3:on	4:off	5:on	6:off
[root@h101 ~]# chkconfig  jenkins on
[root@h101 ~]# chkconfig  --list | grep jenkins
jenkins        	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@h101 ~]# 
{% endhighlight %}

Jenkins 会作为一个后台服务在系统启动时启动


---

#命令汇总

* **`wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo`**
* **`rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key`**
* **`yum install jenkins`**
* **`java -version`**
* **`vim /etc/sysconfig/iptables`**
* **`grep 8080 /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables  reload`**
* **`iptables -L -nv | grep 8080`**
* **`/etc/init.d/jenkins start`**
* **`grep -v "^#" /etc/sysconfig/jenkins  | grep -v "^$"`**
* **`/etc/init.d/jenkins stop`**
* **`chkconfig  jenkins on`**


---

[jenkins]:http://jenkins-ci.org/
[jenkins_doc]:https://wiki.jenkins-ci.org/display/JENKINS/Use+Jenkins
