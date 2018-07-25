---
layout: post
title: "Upgrade Jenkins"
author:  wilmosfang
date: 2018-01-20 09:11:34
image: '/assets/img/'
excerpt: 'Jenkins 的升级方法'
main-class: 'jenkins'
color: '#4799d6'
tags:
 - jenkins
categories:
 - jenkins
twitter_text: 'jenkins upgrade simple process'
introduction: 'upgrade method of Jenkins'
---


# 前言


**[Jenkins][jenkins]** 是一套自动化软件，结合不同的插件可以轻易实现 **CI/CD** 工作流

**[Jenkins][jenkins]** 更新很快，分为两个版本

## Long-term Support (LTS)

每十二周升级一个版本

>LTS (Long-Term Support) releases are chosen every 12 weeks from the stream of regular releases as the stable release for that time period

## Weekly

每周升级一个版本

>A new release is produced weekly to deliver bug fixes and features to users and plugin developers.

## 区别

>The weekly Jenkins releases deliver bug fixes and new features rapidly to users and plugin developers who need them. But for more conservative users, it’s preferable to stick to a release line which changes less often and only receives important bug fixes, even if such a release line lags behind in terms of features.

**[Jenkins][jenkins]** 与 **k8s** 还有 **Gitlab** 常常放在一起构建持续集成系统

下面分享一下 **[Jenkins][jenkins]** 升级过程


> **Tip:** 当前版本 **Jenkins 2.89.3 LTS**

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


## 发现新版本

![jenkins](/assets/img/jenkins/jenkins3.png)

可见当前版本为 **Jenkins ver. 2.89.2** 系统提示有新版本 **New version of Jenkins (2.89.3) is available for download (changelog)**

## 更新日志　

通过 **[变更日志][changelog]** 我们可以看到新版本有哪些变更

也可以通过 **[升级指导][upgrade_guide]** 来看看官方的建议


## 下载更新版本

[下载更新版本][jenkins_dl]

~~~
[root@much tmp]# ll *.war
-rw-r--r-- 1 root root 74292096 1月  20 16:58 jenkins.war
[root@much tmp]#
~~~


## 确认安装路径

![jenkins](/assets/img/jenkins/jenkins4.png)

**[主页]->[Manage Jenkins]->[System Information]**

![jenkins](/assets/img/jenkins/jenkins5.png)

可以看到安装路径为 **/usr/lib/jenkins/jenkins.war**



## 停止服务

![jenkins](/assets/img/jenkins/jenkins6.png)

主页面会提示 **Jenkins is going to shut down**　

然后在没有运行任务的情况下安全地停止 Jenkins 服务


{% highlight shell %}
[root@much tmp]# ps faux | grep jenkins  -i
root      3269  0.0  0.0 112648  1028 pts/0    S+   18:03   0:00  |       \_ grep --color=auto jenkins -i
jenkins   1630  2.7 17.6 3720188 714756 ?      Ssl  16:36   2:24 /etc/alternatives/java -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
[root@much tmp]# systemctl stop jenkins
[root@much tmp]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: inactive (dead) since 六 2018-01-20 18:03:47 CST; 8s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3276 ExecStop=/etc/rc.d/init.d/jenkins stop (code=exited, status=0/SUCCESS)
  Process: 1519 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)

1月 20 16:36:28 much systemd[1]: Starting LSB: Jenkins Automation Server...
1月 20 16:36:28 much runuser[1524]: pam_unix(runuser:session): session ope...0)
1月 20 16:36:30 much systemd[1]: Started LSB: Jenkins Automation Server.
1月 20 16:36:30 much jenkins[1519]: Starting Jenkins [  OK  ]
1月 20 18:03:47 much systemd[1]: Stopping LSB: Jenkins Automation Server...
1月 20 18:03:47 much jenkins[3276]: Shutting down Jenkins [  OK  ]
1月 20 18:03:47 much systemd[1]: Stopped LSB: Jenkins Automation Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much tmp]#
{% endhighlight %}



## 备份和替换 WAR

~~~
[root@much tmp]# ll *.war
-rw-r--r-- 1 root root 74292096 1月  20 16:58 jenkins.war
[root@much tmp]# cd /usr/lib/jenkins/
[root@much jenkins]# ls
jenkins.war
[root@much jenkins]# mv jenkins.war jenkins.war.old
[root@much jenkins]# mv jenkins.war.old jenkins.war.old.20180120
[root@much jenkins]# ls
jenkins.war.old.20180120
[root@much jenkins]# cp /tmp/jenkins.war .
[root@much jenkins]# ll
total 145108
-rw-r--r-- 1 root root 74292096 1月  20 18:08 jenkins.war
-rw-r--r-- 1 root root 74294776 12月 14 10:23 jenkins.war.old.20180120
[root@much jenkins]#
~~~


## 启动服务

~~~
[root@much jenkins]# systemctl start jenkins
[root@much jenkins]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since 六 2018-01-20 18:10:06 CST; 7s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3276 ExecStop=/etc/rc.d/init.d/jenkins stop (code=exited, status=0/SUCCESS)
  Process: 3372 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/jenkins.service
           └─3389 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -D...

1月 20 18:10:05 much systemd[1]: Starting LSB: Jenkins Automation Server...
1月 20 18:10:05 much runuser[3373]: pam_unix(runuser:session): session ope...0)
1月 20 18:10:06 much jenkins[3372]: Starting Jenkins [  OK  ]
1月 20 18:10:06 much systemd[1]: Started LSB: Jenkins Automation Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@much jenkins]#
~~~


## 进行访问

出现登录界面,输入登录密码

![jenkins](/assets/img/jenkins/jenkins7.png)

右下角显示已经是新版本了

---

# 总结

因为 Jenkins 被打成了一个 war 包，并且所有的配置与代码都进行了解耦，所以升级过程特别简单

简而言之就是只用替换掉 war 包，其它都不变

* TOC
{:toc}


---

[jenkins]:https://jenkins.io/
[upgrade_guide]:https://jenkins.io/doc/upgrade-guide/
[changelog]:https://jenkins.io/changelog-stable/
[jenkins_dl]:https://pkg.jenkins.io/redhat-stable/
