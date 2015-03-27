---
layout: default
title: mysql HA 搭建(一) percona5.1 安装
comments: true
---

#前言


高可用非常重要，任何系统架构都要去考虑，数据库尤为如此

开源的系统架构中使用mysql的份额很大

mysql的各种高可用方案里mha比较高效和快捷

下面我分享一下前段时间研究MHA过程中积累下来的经验





##环境

* 三个虚拟机:

m1:192.168.75.11/24

m2:192.168.75.12/24

s:192.168.75.13/24

* 一个浮动ip:

192.168.66.66/24


* 操作系统版本:

CentOS release 6.6 (Final)

* Mha软件版本:

MHA 0.53 for RHEL6

* Keepalived软件版本:

Keepalived v1.2.13 (10/15,2014)



>目前[MHA](http://code.google.com/p/mysql-master-ha/)的最新版本是0.56
>
>[Percona Server](http://www.percona.com/doc/percona-server/5.6/)的最新版本是5.6
>
>之所以使用这两个老版本是为了刻意模拟某些环境，其实老版本带来的不便要多于新版本，实际工作中建议使用最新版本


##系统架构

正常状态 : m1为主库，m2与s作为备库并列从m1那里进行复制，浮动ip挂在m1上


|host|ip |role| keepalived|vip|OS|mha|
|:--- | ---:| :---: |:---:| :---:|:---:|:---:|
|m1|192.168.75.11 |master|v1.2.13|192.168.66.66|CentOS6.6|node|
|m2|192.168.75.11|slave|v1.2.13|null|CentOS6.6|node|
|s|192.168.75.13|slave|null|null|CentOS6.6|node,manager|

~~~
m1 	192.168.75.11 	192.168.66.66
|-m2	192.168.75.12
`-s	192.168.75.13
~~~

切换后状态 : 切换后m1从集群中移出，原来的后备master m2 升级为主master，原来的s将主指向m2，继续作为库，浮动IP飘到m2上，这个过程是由mha软件自动来完成


|host|ip |role| keepalived|vip|OS|mha|
|:--- | ---:| :---: |:---:| :---:|:---:|:---:|
|m1|192.168.75.11 |null|v1.2.13|null|CentOS6.6|node|
|m2|192.168.75.11|master|v1.2.13|192.168.66.66|CentOS6.6|node|
|s|192.168.75.13|slave|null|null|CentOS6.6|node,manager|

~~~
m2	192.168.75.12 	192.168.66.66
`s	192.168.75.13
~~~



