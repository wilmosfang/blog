---
layout: post
title: "Simple CI/CD with poll SCM of Jenkins"
author:  wilmosfang
date: 2018-01-20 14:19:06
image: '/assets/img/'
excerpt: '使用Jenkins的pollSCM实现简单的CI/CD'
main-class: 'jenkins'
color: '#1077ad'
tags:
 - jenkins
 - ci
 - cd
categories:
 - jenkins
twitter_text: 'CI/CD with pollSCM of Jenkins'
introduction: 'Simple CI/CD method'
---


# 前言


**[Jenkins][jenkins]** 是一套自动化软件，结合不同的插件可以轻易实现 **CI/CD** 工作流

**[Jenkins][jenkins]** 与 **k8s** 还有 **Gitlab** 常常放在一起构建持续集成系统

下面分享一下 **[Jenkins][jenkins]** 构建 **CI/CD** 流的简单实现


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


## 创建密钥对


**[HomePage]->[Credentials]->[System]->[Global credentials (unrestricted)]->[Add Credentials]**

![jenkins](/assets/img/jenkins/jenkins8.png)

这个密钥对的作用是用来登录目标服务器

代码最终要更新到此服务器中，WEB服务在此服务器中运行

**Username** 和 **Password** 必须手动指定，即为登录账号与密码

**Description** 可以不填，只是为了识别

**ID** 可以不填，会自动生成

## 添加SSH远程主机

**[HomePage]->[Manage Jenkins]->[Configure System]->[SSH remote hosts]->[Add]**

![jenkins](/assets/img/jenkins/jenkins9.png)

配置完成后点击 **[Check connection]** 进行连接测试

**Successfull connection**　表明可以正常联通

**Hostname** 指定远程主机 IP 或主机名，必须网络可达

**Port** 指定远程的 SSH 端口

**Credentials** 选择上一步中设定的密钥对

其它保持默认，这样就配置好了一个远程主机


## 创建项目

![jenkins](/assets/img/jenkins/jenkins10.png)

**[HomePage]->[New Item]->[Freestyle Project]->[OK]**

**Enter an item name** 下输入项目名


## 配置SCM

**SCM** 是 **Source Code Management** 的缩写

![jenkins](/assets/img/jenkins/jenkins11.png)

选择 **Git** (因为我的项目在GitHub上)

然后指定正确的 **Repository URL** 和 **Branch Specifier (blank for 'any')** 分支 (因为我的 Web 只发布于 gh-pages, 所以我只需要让其检查此分支的变化就可以了)

## 配置触发器

**Build Triggers**

![jenkins](/assets/img/jenkins/jenkins12.png)

这里为了简便，就使用了 **Poll SCM**

**`H/2  * * * *`** 代表每两分钟检查一次

编辑框下面会提示下一次执行检查的时间

### Poll SCM 与 Build periodically 区别

**Build periodically** 也会要求输入调动周期

那 **Poll SCM** 和它有什么区别呢

两者都会周期性地调动，但是 **Poll SCM** 只在检查到源码版本有变化的时候才会执行后面的 **build** 操作，而 **Build periodically** 是不论源码版本是否有变化都会执行后面的 **build** 操作


### 主动与被动

如果源代码在公网平台上 (比如 github)，那这两者与其它触发机制有什么不同呢

这两者由于是主动发起的，所以可以没有公网IP而隐藏在 NAT 后面，只要有可以主动访问公网的权限就可以，而其它方式(比如 GitHub hook trigger for GITScm polling),就需要提供一个公网 IP 或从公网 IP 到此 Jenkins 服务端口的 DNAT 映射，无疑后者的网络环境要求要高一些，但是前者的系统开销要大一点，因为事件触发的响应式模型更加有效和节省系统资源

## 配置执行内容

**Build** 作为整个构建过程中最核心的一步，里面定义了所有要做的事情

这里选择 **Excuete shell scrip on remote host using ssh**

![jenkins](/assets/img/jenkins/jenkins13.png)


**SSH site** 中选择在系统配置里设定好的连接串

**Command** 中定义脚本内容

由于我是使用的 jekyll 来构建 web 的，所以可以动态发布，并没额外的 build 步骤，这一步由 jekyll 代劳了，我只需要更新发布代码就可以了

~~~
cd  /home/git/git/biscuits/
git pull
echo `date` > /tmp/date
cat /tmp/date
~~~

前面两步是进入代码根目录，下拉最新代码到本地，后面两步是记录一个更新的时间戳到 tmp 目录


## 提交变更触发发布

从本地 commit 完代码 push 到远程库后，远程仓库的代码版本就会发生变化

等每两分钟的 pollSCM 检查后，发现远程代码版本发生了变化，就会触发一次 build 的过程　

![jenkins](/assets/img/jenkins/jenkins14.png)


## 日志输出

可以点击查看此次构建的 **Console Output**

![jenkins](/assets/img/jenkins/jenkins15.png)


**Console Output**

~~~
Console Output
Started by an SCM change
Building in workspace /var/lib/jenkins/workspace/blog_cicd_test
Cloning the remote Git repository
Cloning repository https://github.com/wilmosfang/biscuits.git
 > git init /var/lib/jenkins/workspace/blog_cicd_test # timeout=10
Fetching upstream changes from https://github.com/wilmosfang/biscuits.git
 > git --version # timeout=10
 > git fetch --tags --progress https://github.com/wilmosfang/biscuits.git +refs/heads/*:refs/remotes/origin/*
 > git config remote.origin.url https://github.com/wilmosfang/biscuits.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/wilmosfang/biscuits.git # timeout=10
Fetching upstream changes from https://github.com/wilmosfang/biscuits.git
 > git fetch --tags --progress https://github.com/wilmosfang/biscuits.git +refs/heads/*:refs/remotes/origin/*
 > git rev-parse refs/remotes/origin/gh-pages^{commit} # timeout=10
 > git rev-parse refs/remotes/origin/origin/gh-pages^{commit} # timeout=10
Checking out Revision 2457bdb4a2ed540109acf164d9974519a5ec43b6 (refs/remotes/origin/gh-pages)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 2457bdb4a2ed540109acf164d9974519a5ec43b6
Commit message: "add  _posts/2018-01-20-simple-cicd-with-poll-scm-of-jenkins.md"
 > git rev-list --no-walk 8b54a92f57c6df0bbef8142887192abe35e646c2 # timeout=10
[SSH] script:

cd  /home/git/git/biscuits/
git pull
echo `date` > /tmp/date
cat /tmp/date


[SSH] executing...
From github.com:wilmosfang/biscuits
   8b54a92..2457bdb  gh-pages   -> origin/gh-pages
Updating 8b54a92..2457bdb
Fast-forward
 ...8-01-20-simple-cicd-with-poll-scm-of-jenkins.md | 196 +++++++++++++++++++++
 _posts/2018-01-20-upgrade-jenkins.md               |   7 +-
 assets/img/jenkins/jenkins10.png                   | Bin 0 -> 121689 bytes
 assets/img/jenkins/jenkins11.png                   | Bin 0 -> 83667 bytes
 assets/img/jenkins/jenkins12.png                   | Bin 0 -> 95810 bytes
 assets/img/jenkins/jenkins13.png                   | Bin 0 -> 70812 bytes
 assets/img/jenkins/jenkins8.png                    | Bin 0 -> 87137 bytes
 assets/img/jenkins/jenkins9.png                    | Bin 0 -> 68035 bytes
 8 files changed, 201 insertions(+), 2 deletions(-)
 create mode 100644 _posts/2018-01-20-simple-cicd-with-poll-scm-of-jenkins.md
 create mode 100644 assets/img/jenkins/jenkins10.png
 create mode 100644 assets/img/jenkins/jenkins11.png
 create mode 100644 assets/img/jenkins/jenkins12.png
 create mode 100644 assets/img/jenkins/jenkins13.png
 create mode 100644 assets/img/jenkins/jenkins8.png
 create mode 100644 assets/img/jenkins/jenkins9.png
Sun Jan 21 00:28:23 CST 2018

[SSH] completed
[SSH] exit-status: 0

Started calculate disk usage of build
Finished Calculation of disk usage of build in 0 seconds
Started calculate disk usage of workspace
Finished Calculation of disk usage of workspace in 0 seconds
Finished: SUCCESS
~~~

从日志中可以看到整个构建过程的详细输出与返回状态，便于进行 debug

构建与发布成功后可以直接到网页中查看最终效果

不难想像，再集成自动测试的若干步骤后，开发人员与价值交付间最终会缩减成了一个 commit


## 其它信息

每触发一次构建都会有一个闪烁的任务进度显示在左边的状态栏中

![jenkins](/assets/img/jenkins/jenkins16.png)

运行过程中的日志是会实时反馈到 **Console Output** 中的

![jenkins](/assets/img/jenkins/jenkins17.png)

可以看到历史任务的分布图与耗时趋势图

![jenkins](/assets/img/jenkins/jenkins18.png)


---

# 总结

Jenkins 非常注重管道(Pipeline)的概念，这篇文档以最简洁的方式演示了管道的过程

从开发，到提交，到推送，到检查更新，到触发操作，到测试，到构建，到发布，到检验就是一个完整的管道流

根据实际项目中的具体情况，其中步骤或多或少，但这是一个很有效的思路，将价值交付的过程管道化，自动化，并且将人的注意力节省下来，用在最有意义的部分

* TOC
{:toc}


---

[jenkins]:https://jenkins.io/
