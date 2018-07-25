---
layout: post
title: "Config Otrs"
author:  wilmosfang
date: 2018-01-30 14:26:12
image: '/assets/img/'
excerpt: 'Otrs 的配置方法'
main-class: otrs
color: '#66b2c1'
tags:
 - otrs
categories:
 - otrs
twitter_text: 'web configuration of Otrs'
introduction: 'configuration of Otrs'
---


# 前言


**[Otrs][otrs]** 是一个开源的工单系统

使用工单来对接服务请求，跟踪问题，管理事务，协调人力是一种高效的组织方式

这里分享一下 **[Otrs][otrs]** 的配置方法

参考 **[Otrs配置][web_config_otrs]**

> **Tip:** 当前版本 **OTRS 6 Patch Level 4**

---

# 操作

## 系统环境


~~~
[root@much otrs]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 7b4bb229c627449d8860c83a818b2416
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much otrs]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0b:e9:0b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84089sec preferred_lft 84089sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:36:8b:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.209/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much otrs]#
~~~

## 访问配置界面


访问 **http://serverip/otrs/install.pl**

![jenkins](/assets/img/otrs/otrs01.png)

直接进入下一步

下一步进入授权申明

## 授权申明

![jenkins](/assets/img/otrs/otrs02.png)

选择接受

## 选择数据库类型

![jenkins](/assets/img/otrs/otrs03.png)

因为使用的 mysql 所以选择第一个

这里有两种方式:

* 一种是创建新的数据库和账号
* 一种是使用现有的数据库和账号

### 创建用户与数据库

输入正确的 mysql root 账号密码后测试登录

![jenkins](/assets/img/otrs/otrs04.png)

下一步后

会自动创建用用户和密码，还有数据库


### 使用现有的用户和数据库

在 mysql 中手动创建好用户和数据库

>这种情况一般发生在有专门的数据库管理员，而不想给 otrs 暴露太多权限的时候

正确认输入用户和数据库

![jenkins](/assets/img/otrs/otrs05.png)

## 导入数据库与表结构

![jenkins](/assets/img/otrs/otrs06.png)

正常完成后，会有成功提示

## 进行基础系统设置

![jenkins](/assets/img/otrs/otrs07.png)

可以配置 **FQDN,AdminEmial,Organization,LogModule,Default language,CheckMXRecord**

![jenkins](/assets/img/otrs/otrs08.png)

可以配置邮件的出入参数

## 配置完成

![jenkins](/assets/img/otrs/otrs09.png)

会有管理员的账号密码提示和访问链接提示

## 登录 OTRS

![jenkins](/assets/img/otrs/otrs10.png)

## 输入账号密码

![jenkins](/assets/img/otrs/otrs11.png)

## 进入系统

![jenkins](/assets/img/otrs/otrs12.png)

提示不要使用超级用户登录，建议创建服务账户并且使用创建的服务账号登录


# 总结

这个配置过程主要作用就是初始化 otrs 在 DB 中的数据结构

* TOC
{:toc}


---

[otrs]:https://www.otrs.com/
[web_config_otrs]:http://doc.otrs.com/doc/manual/admin/stable/zh_CN/html/web-installer.html
