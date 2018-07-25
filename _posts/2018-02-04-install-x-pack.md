---
layout: post
title: "Install X-Pack"
author:  wilmosfang
date: 2018-02-04 13:29:59
image: '/assets/img/'
excerpt: 'X-Pack 的安装方法'
main-class: es
color: '#51bcb2'
tags:
 - x-pack
 - es
categories:
 - es
twitter_text: 'simple process of X-Pack installation'
introduction: 'installation method of X-Pack'
---


# 前言

**[X-Pack][x-pack]** 是 **Elasticsearch** 的一个插件集合

>X-Pack takes it to a new level by bundling powerful features into a single pack.

当前的 **[X-Pack][x-pack]** 集成了以下功能

* Alerting
* Monitoring
* Reporting
* Graph
* Machine Learning

为 ELK 技术栈的使用带来了很大便利

这里分享一下 **[X-Pack][x-pack]** 的安装方法

参考 **[Installing X-Pack in Elasticsearch][x-pack_install]** 和 **[Install X-Pack][install_x-pack]**

> **Tip:** 当前版本 **Version:6.1.3**



---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 71a5a14bde634bfc8c5bafb7d9442f9e
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
       valid_lft 85051sec preferred_lft 85051sec
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


## elasticsearch-plugin

~~~
[root@much es]# /usr/share/elasticsearch/bin/elasticsearch-plugin -h
A tool for managing installed elasticsearch plugins

Commands
--------
list - Lists installed elasticsearch plugins
install - Install a plugin
remove - removes a plugin from Elasticsearch

Non-option arguments:
command              

Option         Description        
------         -----------        
-h, --help     show help          
-s, --silent   show minimal output
-v, --verbose  show verbose output
[root@much es]#
[root@much es]# /usr/share/elasticsearch/bin/elasticsearch-plugin list
[root@much es]#
~~~

## ES 中安装 x-pack

~~~
[root@much es]# /usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack
-> Downloading x-pack from elastic
[=================================================] 100%  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.io.FilePermission \\.\pipe\* read,write
* java.lang.RuntimePermission accessClassInPackage.com.sun.activation.registries
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.RuntimePermission setFactory
* java.net.SocketPermission * connect,accept,resolve
* java.security.SecurityPermission createPolicy.JavaPolicy
* java.security.SecurityPermission getPolicy
* java.security.SecurityPermission putProviderProperty.BC
* java.security.SecurityPermission setPolicy
* java.util.PropertyPermission * read,write
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@        WARNING: plugin forks a native controller        @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
This plugin launches a native controller that is not subject to the Java
security manager nor to system call filters.

Continue with installation? [y/N]y
Elasticsearch keystore is required by plugin [x-pack], creating...
-> Installed x-pack
[root@much es]# /usr/share/elasticsearch/bin/elasticsearch-plugin list
x-pack
[root@much es]#
~~~

安装过程中会提示有额外权限需求，需要同意

使用 **`elasticsearch-plugin list`** 来确认本地安装的 ES 插件


## 重启 ES

~~~
[root@much es]# systemctl restart elasticsearch
[root@much es]#
~~~

## 使用 x-pack 设定密码

~~~
[root@much es]# ll  /usr/share/elasticsearch/bin/x-pack/setup-passwords auto
ls: cannot access auto: No such file or directory
-rwxr-xr-x 1 root root 303 2月   4 16:11 /usr/share/elasticsearch/bin/x-pack/setup-passwords
[root@much es]#  /usr/share/elasticsearch/bin/x-pack/setup-passwords auto
Initiating the setup of passwords for reserved users elastic,kibana,logstash_system.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y


Changed password for user kibana
PASSWORD kibana = nZkrE27ZXjP5-6CLgrI0

Changed password for user logstash_system
PASSWORD logstash_system = %A&TN@6OWqWK4t%o7kjM

Changed password for user elastic
PASSWORD elastic = rlziMTdf-+cFW4mN0&pO

[root@much es]#
~~~

**X-Pack** 可以加强 ELK 的安全

## 修改 kibana 配置

~~~
[root@much es]# vim /etc/kibana/kibana.yml
[root@much es]# grep -v "^#" /etc/kibana/kibana.yml  | cat -s

server.host: "0.0.0.0"

elasticsearch.username: "kibana"
elasticsearch.password: "nZkrE27ZXjP5-6CLgrI0"

[root@much es]#
~~~

## kibana-plugin

~~~
[root@much ~]# rpm -ql  kibana | grep -i kibana-plugin
/usr/share/kibana/bin/kibana-plugin
[root@much ~]# /usr/share/kibana/bin/kibana-plugin --help

  Usage: bin/kibana-plugin [command] [options]

  The Kibana plugin manager enables you to install and remove plugins that provide additional functionality to Kibana

  Commands:
    list  [options]                 list installed plugins
    install  [options] <plugin/url> install a plugin
    remove  [options] <plugin>      remove a plugin
    help  <command>                 get the help for a specific command


[root@much ~]# /usr/share/kibana/bin/kibana-plugin list

[root@much ~]#
~~~


## 安装 kibana x-pack 插件

~~~
[root@much ~]# /usr/share/kibana/bin/kibana-plugin install x-pack
Attempting to transfer from x-pack
Attempting to transfer from https://artifacts.elastic.co/downloads/kibana-plugins/x-pack/x-pack-6.1.3.zip
Transferring 269818640 bytes....................
Transfer complete
Retrieving metadata from plugin archive
Extracting plugin archive
Extraction complete
Optimizing and caching browser bundles...
Plugin installation complete
[root@much ~]# echo $?
0
[root@much ~]#
[root@much ~]# /usr/share/kibana/bin/kibana-plugin list
x-pack@6.1.3

[root@much ~]#
~~~

## 重启 kibana

~~~
[root@much ~]# systemctl restart kibana
[root@much ~]#
~~~

## 访问

再次访问 kibana 会提示输入密码

![kibana](/assets/img/kibana/kibana08.png)

输入正确密码后，主界面中多出很多功能

![kibana](/assets/img/kibana/kibana09.png)

在 **Monitoring** 中可以看到当前系统的监控状态

![kibana](/assets/img/kibana/kibana10.png)


![kibana](/assets/img/kibana/kibana12.png)


![kibana](/assets/img/kibana/kibana13.png)


默认有 14 天的功能体验时间

14 天后完全的功能需要相应授权

![kibana](/assets/img/kibana/kibana11.png)

* Register for a Basic License

> Monitoring is available for free with a Basic License. Simply register, check your inbox, and enjoy!

* Unlock the Full Functionality

> Get all X-Pack has to offer: security, monitoring, alerting, reporting, and Graph, plus support from the engineers behind the Elastic Stack





---

# 总结

自带的插件管理工具安装的过程非常简单

分别要在 es 中和 kibana 中安装 x-pack 插件

* TOC
{:toc}


---

[x-pack]:https://www.elastic.co/cn/products/x-pack
[x-pack_install]:https://www.elastic.co/guide/en/elasticsearch/reference/6.1/installing-xpack-es.html
[install_x-pack]:https://www.elastic.co/downloads/x-pack
