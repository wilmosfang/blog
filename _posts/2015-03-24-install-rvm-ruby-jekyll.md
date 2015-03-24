---
layout: default
title: 安装jekyll 
---

##前言

我前面有一篇文章可以[21分钟搭建一个GitHub Blog](http://wilmosfang.github.io/blog/2015/03/02/build-a-githubblog-in-21minutes.html)

这里我们深入到jekyll层面看看,如何在本地也生成一个blog,也就是在本地干和github上一样的事情

这里涉及到rvm ruby gem 和jekyll的安装,绝对干货,因为里面已经包含了几个我踩的坑,坑对面的屌丝看过来,看过来,看过来……这里有表演很精彩……


##系统环境

>[root@Test-slave ~]# lsb_release  -a 
>LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
>Distributor ID:	CentOS
>Description:	CentOS release 6.6 (Final)
>Release:	6.6
>Codename:	Final
>[root@Test-slave ~]# cat /etc/issue
>CentOS release 6.6 (Final)
>Kernel \r on an \m
>
>[root@Test-slave ~]# uname -a 
>Linux Test-slave 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
>[root@Test-slave ~]# 

