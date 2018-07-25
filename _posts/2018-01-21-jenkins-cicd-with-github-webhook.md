---
layout: post
title: "Jenkins CI/CD with GitHub webhook"
author:  wilmosfang
date: 2018-01-21 09:31:24
image: '/assets/img/'
excerpt: '使用 GitHub webhook 来实现 Jenkins CI/CD 流'
main-class: 'jenkins'
color: '#1077ad'
tags:
 - jenkins
 - ci
 - cd
categories:
 - jenkins
twitter_text: 'Jenkins CI/CD with GitHub webhook'
introduction: 'simple CI/CD with webhook'
---


# 前言


**[Jenkins][jenkins]** 是一套自动化软件，结合不同的插件可以轻易实现 **CI/CD** 工作流

**[Jenkins][jenkins]** 与 **k8s** 还有 **Gitlab** 常常放在一起构建持续集成系统

下面分享一下 **[Jenkins][jenkins]** 结合 **GitHub webhook** 构建 **CI/CD** 流的简单实现


> **Tip:** 当前版本 **Jenkins 2.89.3 LTS**

---

# 操作

## 系统环境

~~~
[root@ci ~]# hostnamectl
   Static hostname: ci
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 8d13a50988cc5c4972347415eddf7d47
           Boot ID: 10b2ba6eee6941b78a7e0b2fa9c42e8c
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@ci ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:b7:37:f1 brd ff:ff:ff:ff:ff:ff
    inet 10.144.126.20/18 brd 10.144.127.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@ci ~]#
~~~

## 查出口IP

很多云服务商并不会直接给一台 VM 分配一个公网 IP 绑定到本地网卡，而是通过 DNAT 的方式进行分配

这时从本地就无法直接看到被分配的公网 IP

这里提供两种查本地出回 IP 的方法

{% highlight bash %}
[root@ci ~]# curl  ifconfig.me
119.28.xx.xx
[root@ci ~]# curl http://members.3322.org/dyndns/getip  
119.28.xx.xx
[root@ci ~]#
{% endhighlight %}


当然如果不嫌麻烦，也可以直接登录控制台查看

这个方法也可以用来查看宽带在访问外网过程中运营商给我们分配的公网 IP


## 安装插件

**Manage Jenkins -> Manage Plugins -> Available -> GitHub plugin**


![jenkins](/assets/img/jenkins/jenkins19.png)


同样的方式安装上 **SSH plugin** 插件


## GitHub 配置

**GitHub -> repositories -> repo -> settings -> Integrations＆Service -> Add service -> Jenkins (GitHub plugin)**


![jenkins](/assets/img/jenkins/jenkins20.png)

输入 Jenkins 的地址

![jenkins](/assets/img/jenkins/jenkins21.png)

保存

![jenkins](/assets/img/jenkins/jenkins22.png)

测试连接

![jenkins](/assets/img/jenkins/jenkins23.png)


## 创建密钥对


**Credentials -> System -> Global credentials (unrestricted) -> Add Credentials**


![jenkins](/assets/img/jenkins/jenkins24.png)


![jenkins](/assets/img/jenkins/jenkins25.png)


这个密钥对的作用是用来登录目标服务器

代码最终要更新到此服务器中，WEB服务在此服务器中运行

**Username** 和 **Password** 必须手动指定，即为登录账号与密码

**Description** 可以不填，只是为了识别

**ID** 可以不填，会自动生成


## 添加SSH远程主机

**Manage Jenkins -> Configure System -> SSH remote hosts -> Add**

![jenkins](/assets/img/jenkins/jenkins26.png)

配置完成后点击 **[Check connection]** 进行连接测试

**Successfull connection**　表明可以正常联通

**Hostname** 指定远程主机 IP 或主机名，必须网络可达

**Port** 指定远程的 SSH 端口

**Credentials** 选择上一步中设定的密钥对

其它保持默认，这样就配置好了一个远程主机


## 创建项目

![jenkins](/assets/img/jenkins/jenkins27.png)

**New Item -> Freestyle Project -> OK**

**Enter an item name** 下输入项目名

## 配置SCM

**SCM** 是 **Source Code Management** 的缩写

![jenkins](/assets/img/jenkins/jenkins28.png)

选择 **Git** (因为我的项目在GitHub上)

然后指定正确的 **Repository URL** 和 **Branch Specifier (blank for 'any')** 分支 (因为我的 Web 只发布于 gh-pages, 所以我只需要让其检查此分支的变化就可以了)


## 配置触发器

**Build Triggers**

![jenkins](/assets/img/jenkins/jenkins29.png)


**GitHub hook trigger for GITScm polling** 会在代码发生变化的时候，由 GitHub 发出一个请求给之前我们配置的 Jenkins 链接

Jenkins 监听到这个请求后就会触发构建的过程，相较于周期性轮询，这种方式更为高效　

## 配置执行内容

**Build** 作为整个构建过程中最核心的一步，里面定义了所有要做的事情

这里选择 **Excuete shell scrip on remote host using ssh**

![jenkins](/assets/img/jenkins/jenkins30.png)


**SSH site** 中选择在系统配置里设定好的连接串

**Command** 中定义脚本内容

由于我是使用的 jekyll 来构建 web 的，所以可以动态发布，并没额外的 build 步骤，这一步由 jekyll 代劳了，我只需要更新发布代码就可以了

{% highlight bash %}
cd  /home/git/git/biscuits/
git pull
{% endhighlight %}

这两步是进入代码根目录，下拉最新代码到本地


## 提交变更触发发布

从本地 commit 完代码 push 到远程库后，远程仓库的代码版本就会发生变化

远程代码版本发生了变化，GitHub 就会给 Jenkins 发送一个请求，Jenkins 收到请求就会触发一次 build 的过程　

![jenkins](/assets/img/jenkins/jenkins31.png)

在提交代码后，左下角会自动产生一个任务进度条，显示当前的构建进度和状态

## 日志输出

可以点击查看此次构建的 **Console Output**

![jenkins](/assets/img/jenkins/jenkins32.png)


**Console Output**

{% highlight bash %}
Console Output
Started by GitHub push by wilmosfang
Building in workspace /var/lib/jenkins/workspace/github_webhook_test
Cloning the remote Git repository
Cloning repository https://github.com/wilmosfang/biscuits.git
 > git init /var/lib/jenkins/workspace/github_webhook_test # timeout=10
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
Checking out Revision 0d7d73a63ba76c6d36177132594967c4b2992016 (refs/remotes/origin/gh-pages)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 0d7d73a63ba76c6d36177132594967c4b2992016
Commit message: "add  2018-01-21-jenkins-cicd-with-github-webhook"
First time build. Skipping changelog.
[SSH] script:

cd ~/git/biscuits/
git pull

[SSH] executing...
From github.com:wilmosfang/biscuits
   952a9ac..0d7d73a  gh-pages   -> origin/gh-pages
Updating 952a9ac..0d7d73a
Fast-forward
 .../2018-01-21-jenkins-cicd-with-github-webhook.md | 254 +++++++++++++++++++++
 assets/css/main.css                                |   2 +-
 assets/img/jenkins/jenkins19.png                   | Bin 0 -> 164709 bytes
 assets/img/jenkins/jenkins20.png                   | Bin 0 -> 119027 bytes
 assets/img/jenkins/jenkins21.png                   | Bin 0 -> 126164 bytes
 assets/img/jenkins/jenkins22.png                   | Bin 0 -> 112304 bytes
 assets/img/jenkins/jenkins23.png                   | Bin 0 -> 120453 bytes
 assets/img/jenkins/jenkins24.png                   | Bin 0 -> 82907 bytes
 assets/img/jenkins/jenkins25.png                   | Bin 0 -> 89182 bytes
 assets/img/jenkins/jenkins26.png                   | Bin 0 -> 67115 bytes
 assets/img/jenkins/jenkins27.png                   | Bin 0 -> 103769 bytes
 assets/img/jenkins/jenkins28.png                   | Bin 0 -> 78133 bytes
 assets/img/jenkins/jenkins29.png                   | Bin 0 -> 70974 bytes
 assets/img/jenkins/jenkins30.png                   | Bin 0 -> 61959 bytes
 category/jenkins.html                              |   2 +-
 src/styl/_theme-colors.styl                        |   2 +-
 16 files changed, 257 insertions(+), 3 deletions(-)
 create mode 100644 _posts/2018-01-21-jenkins-cicd-with-github-webhook.md
 create mode 100644 assets/img/jenkins/jenkins19.png
 create mode 100644 assets/img/jenkins/jenkins20.png
 create mode 100644 assets/img/jenkins/jenkins21.png
 create mode 100644 assets/img/jenkins/jenkins22.png
 create mode 100644 assets/img/jenkins/jenkins23.png
 create mode 100644 assets/img/jenkins/jenkins24.png
 create mode 100644 assets/img/jenkins/jenkins25.png
 create mode 100644 assets/img/jenkins/jenkins26.png
 create mode 100644 assets/img/jenkins/jenkins27.png
 create mode 100644 assets/img/jenkins/jenkins28.png
 create mode 100644 assets/img/jenkins/jenkins29.png
 create mode 100644 assets/img/jenkins/jenkins30.png

[SSH] completed
[SSH] exit-status: 0

Finished: SUCCESS
{% endhighlight %}

从日志中可以看到整个构建过程的详细输出与返回状态，便于 debug

构建与发布成功后可以直接到网页中查看最终效果

不难想像，再集成自动测试的若干步骤后，开发人员与价值交付间最终会缩减成了一个 commit



---

# 总结

Jenkins 非常注重管道(Pipeline)的概念，这篇文档以最简洁的方式演示了管道的过程

从开发，到提交，到推送，到检查更新，到触发操作，到测试，到构建，到发布，到检验就是一个完整的管道流

根据实际项目中的具体情况，其中步骤或多或少，但这是一个很有效的思路，将价值交付的过程管道化，自动化，并且将人的注意力节省下来，用在最有意义的部分

* TOC
{:toc}


---

[jenkins]:https://jenkins.io/
