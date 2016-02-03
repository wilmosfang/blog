---
layout: post
title:  Zabbix Agent 安装
categories: linux monitoring zabbix 
wc: 443 1534 20654
excerpt: zabbix agent 的安装过程
comments: true
date:   2015-12-14 17:52:00
---


---

# 前言


**[Zabbix][zabbix]** 是一个开源的企业级分布式监控解决方案

非常灵活与强大，有很多优秀 **[特性][features]** ，布署与管理也相对简单，下面分享一下它的基础操作

详细操作可以参考 **[官方文档][zbx_doc]** 

> **Tip:**  当前的最新稳定版是 **Zabbix 2.4**

---


# 概要

* TOC
{:toc}




---


##获取zabbix仓库

{% highlight bash %}
[root@zbx-target src]# wget http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
--2015-12-14 15:23:26--  http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
Resolving repo.zabbix.com... 87.110.183.174
Connecting to repo.zabbix.com|87.110.183.174|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11296 (11K) [application/x-redhat-package-manager]
Saving to: “zabbix-release-2.4-1.el6.noarch.rpm”

100%[==============================================================================>] 11,296      40.2K/s   in 0.3s    

2015-12-14 15:23:27 (40.2 KB/s) - “zabbix-release-2.4-1.el6.noarch.rpm” saved [11296/11296]

[root@zbx-target src]# ll zabbix-release-2.4-1.el6.noarch.rpm
-rw-r--r--. 1 root root 11296 Sep 11  2014 zabbix-release-2.4-1.el6.noarch.rpm
[root@zbx-target src]# rpm -ivh  zabbix-release-2.4-1.el6.noarch.rpm
warning: zabbix-release-2.4-1.el6.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID 79ea5ed4: NOKEY
Preparing...                ########################################### [100%]
   1:zabbix-release         ########################################### [100%]
[root@zbx-target src]# 
{% endhighlight %}


---

##使用yum安装zabbix-agent

{% highlight bash %}
[root@zbx-target src]# yum list all | grep zabbix
zabbix-release.noarch                    2.4-1.el6                      installed
fping.x86_64                             2.4b2-16.el6                   zabbix-non-supported
iksemel.x86_64                           1.4-2.el6                      zabbix-non-supported
iksemel-devel.x86_64                     1.4-2.el6                      zabbix-non-supported
iksemel-utils.x86_64                     1.4-2.el6                      zabbix-non-supported
libssh2.x86_64                           1.4.2-2.el6                    zabbix-non-supported
libssh2-devel.x86_64                     1.4.2-2.el6                    zabbix-non-supported
libssh2-docs.noarch                      1.4.2-2.el6                    zabbix-non-supported
perl-Config-IniFiles.noarch              2.72-2.el6                     zabbix-non-supported
snmptt.noarch                            1.4-1.el6                      zabbix-non-supported
zabbix.x86_64                            2.4.7-1.el6                    zabbix  
zabbix-agent.x86_64                      2.4.7-1.el6                    zabbix  
zabbix-get.x86_64                        2.4.7-1.el6                    zabbix  
zabbix-java-gateway.x86_64               2.4.7-1.el6                    zabbix  
zabbix-proxy.x86_64                      2.4.7-1.el6                    zabbix  
zabbix-proxy-mysql.x86_64                2.4.7-1.el6                    zabbix  
zabbix-proxy-pgsql.x86_64                2.4.7-1.el6                    zabbix  
zabbix-proxy-sqlite3.x86_64              2.4.7-1.el6                    zabbix  
zabbix-sender.x86_64                     2.4.7-1.el6                    zabbix  
zabbix-server.x86_64                     2.4.7-1.el6                    zabbix  
zabbix-server-mysql.x86_64               2.4.7-1.el6                    zabbix  
zabbix-server-pgsql.x86_64               2.4.7-1.el6                    zabbix  
zabbix-web.noarch                        2.4.7-1.el6                    zabbix  
zabbix-web-japanese.noarch               2.4.7-1.el6                    zabbix  
zabbix-web-mysql.noarch                  2.4.7-1.el6                    zabbix  
zabbix-web-pgsql.noarch                  2.4.7-1.el6                    zabbix  
[root@zbx-target src]# yum install zabbix-agent.x86_64 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * extras: mirrors.skyshe.cn
 * updates: mirrors.skyshe.cn
Resolving Dependencies
--> Running transaction check
---> Package zabbix-agent.x86_64 0:2.4.7-1.el6 will be installed
--> Processing Dependency: zabbix for package: zabbix-agent-2.4.7-1.el6.x86_64
--> Running transaction check
---> Package zabbix.x86_64 0:2.4.7-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================
 Package                        Arch                     Version                         Repository                Size
========================================================================================================================
Installing:
 zabbix-agent                   x86_64                   2.4.7-1.el6                     zabbix                   173 k
Installing for dependencies:
 zabbix                         x86_64                   2.4.7-1.el6                     zabbix                   163 k

Transaction Summary
========================================================================================================================
Install       2 Package(s)

Total download size: 336 k
Installed size: 1.1 M
Is this ok [y/N]: y
Downloading Packages:
(1/2): zabbix-2.4.7-1.el6.x86_64.rpm                                                              | 163 kB     00:30     
(2/2): zabbix-agent-2.4.7-1.el6.x86_64.rpm                                                        | 173 kB     00:01     
-------------------------------------------------------------------------------------------------------------------------
Total                                                                                    8.9 kB/s | 336 kB     00:37     
warning: rpmts_HdrFromFdno: Header V4 DSA/SHA1 Signature, key ID 79ea5ed4: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
Importing GPG key 0x79EA5ED4:
 Userid : Zabbix SIA <packager@zabbix.com>
 Package: zabbix-release-2.4-1.el6.noarch (installed)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : zabbix-2.4.7-1.el6.x86_64                                                                             1/2 
  Installing : zabbix-agent-2.4.7-1.el6.x86_64                                                                       2/2 
  Verifying  : zabbix-agent-2.4.7-1.el6.x86_64                                                                       1/2 
  Verifying  : zabbix-2.4.7-1.el6.x86_64                                                                             2/2 

Installed:
  zabbix-agent.x86_64 0:2.4.7-1.el6                                                                                      

Dependency Installed:
  zabbix.x86_64 0:2.4.7-1.el6                                                                                            

Complete!
[root@zbx-target src]# 
{% endhighlight %}

---

##目录结构


{% highlight bash %}
[root@zbx-target etc]# tree /etc/zabbix/
/etc/zabbix/
├── zabbix_agentd.conf
└── zabbix_agentd.d
    └── userparameter_mysql.conf

1 directory, 2 files
[root@zbx-target etc]# 
{% endhighlight %}

其中 **zabbix_agentd.conf** 是agent的配置文件， **userparameter_mysql.conf** 是用户自定义监控插件的地方

只要定义在 **zabbix_agentd.d** 目录下都有效，所以习惯上，一种应用使用一个独立的配置

这个机制非常强大，可以灵活地集成本地脚本，收集任何脚本可以收集到的信息

---

##修改配置

原本的配置 

{% highlight bash %}
[root@zbx-target etc]#  grep -v "^#" /etc/zabbix/zabbix_agentd.conf | grep -v "^$"
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=Zabbix server
Include=/etc/zabbix/zabbix_agentd.d/
[root@zbx-target etc]# 
{% endhighlight %}

我们需要在 **Server** 中加入zabbix server的IP地址

{% highlight bash %}
[root@zbx-target zabbix]# vim zabbix_agentd.conf 
[root@zbx-target zabbix]# grep -v "^#" /etc/zabbix/zabbix_agentd.conf | grep -v "^$"
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=192.168.66.123,127.0.0.1
ServerActive=127.0.0.1
Hostname=Zabbix server
Include=/etc/zabbix/zabbix_agentd.d/
[root@zbx-target zabbix]# 
{% endhighlight %}


---

##打开防火墙


{% highlight bash %}
[root@zbx-target script]# vim /etc/sysconfig/iptables
[root@zbx-target script]# grep 10050 /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10050  -j ACCEPT
[root@zbx-target script]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@zbx-target script]# iptables -L -nv  | grep 10050
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:10050 
[root@zbx-target script]# 
{% endhighlight %}

默认情况下 **zabbix_agentd** 会监听在 **`0.0.0.0:10050`** 上面，所以要将防火墙打开，以方便与zabbix server之间的通信

---

##启动zabbix-agent

{% highlight bash %}
[root@zbx-target zabbix]# /etc/init.d/zabbix-agent start 
Starting Zabbix agent:                                     [  OK  ]
[root@zbx-target zabbix]# ps faux | grep zabbix | grep -v grep 
zabbix   26124  0.0  0.0  77336  1144 ?        S    16:11   0:00 zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
zabbix   26126  0.0  0.0  77336  1564 ?        S    16:11   0:00  \_ zabbix_agentd: collector [idle 1 sec]          
zabbix   26127  0.0  0.0  77336   968 ?        S    16:11   0:00  \_ zabbix_agentd: listener #1 [waiting for connection]
zabbix   26128  0.0  0.0  77336  1108 ?        S    16:11   0:00  \_ zabbix_agentd: listener #2 [waiting for connection]
zabbix   26129  0.0  0.0  77336   968 ?        S    16:11   0:00  \_ zabbix_agentd: listener #3 [waiting for connection]
zabbix   26130  0.0  0.0  77348  1136 ?        S    16:11   0:00  \_ zabbix_agentd: active checks #1 [idle 1 sec]   
[root@zbx-target zabbix]# netstat  -ant | grep 10050
tcp        0      0 0.0.0.0:10050               0.0.0.0:*                   LISTEN      
tcp        0      0 192.168.66.5:10050          192.168.66.123:38010         TIME_WAIT   
tcp        0      0 192.168.66.5:10050          192.168.66.123:38072         TIME_WAIT   
tcp        0      0 :::10050                    :::*                        LISTEN      
[root@zbx-target zabbix]# 
{% endhighlight %}

当它可以开机启动

{% highlight bash %}
[root@zbx-target zabbix]# chkconfig  --list | grep zabbix
zabbix-agent   	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@zbx-target zabbix]# chkconfig  zabbix-agent on 
[root@zbx-target zabbix]# chkconfig  --list | grep zabbix
zabbix-agent   	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@zbx-target zabbix]# 
{% endhighlight %}

---

##使用zabbix-server测试连接

{% highlight bash %}
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "system.cpu.load[all,avg1]"
0.010000
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "system.cpu.load[all,avg5]"
0.020000
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "system.cpu.load[all,avg15]"
0.010000
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "net.tcp.listen[10050]"
1
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "net.tcp.listen[10051]"
0
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "system.boottime"
1449836322
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "agent.version"
2.4.7
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "net.if.discovery"
{"data":[{"{#IFNAME}":"lo"},{"{#IFNAME}":"em1"},{"{#IFNAME}":"em2"},{"{#IFNAME}":"em3"},{"{#IFNAME}":"em4"}]}
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "system.cpu.discovery"
{"data":[{"{#CPU.NUMBER}":0,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":1,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":2,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":3,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":4,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":5,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":6,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":7,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":8,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":9,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":10,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":11,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":12,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":13,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":14,"{#CPU.STATUS}":"online"},{"{#CPU.NUMBER}":15,"{#CPU.STATUS}":"online"}]}
[root@zbx-server script]# zabbix_get -s zbx-target -p 10050 -k "system.sw.arch"
x86_64
[root@zbx-server script]# 
{% endhighlight %}

可以正常获取检测结果，说明连接通畅，更多的监控条目可以参考 **[Zabbix agent items][zbx_items]** ，这些条目的详细解释可以参考 **[Zabbix agent][zabbix_agent]**

Zabbix中已经集成了大量的常用监控条目，不用过多配置就可以直接使用


---

##添加监控脚本


虽然Zabbix直接集成和覆盖了很多我们的监控对象，但有时官方提供的条目无法满足我们的个性化需求，这时需要自定义一些脚本，获取信息以让zabbix可以接受并处理

{% highlight bash %}
[root@zbx-target zabbix]# ls
zabbix_agentd.conf  zabbix_agentd.d
[root@zbx-target zabbix]# cd zabbix_agentd.d/
[root@zbx-target zabbix_agentd.d]# ls
userparameter_mysql.conf
[root@zbx-target zabbix_agentd.d]# mkdir script
[root@zbx-target zabbix_agentd.d]# ls
script  userparameter_mysql.conf
[root@zbx-target zabbix_agentd.d]# cd script/
[root@zbx-target script]# vim  port.discovery.bash 
[root@zbx-target script]# ll 
total 4
-rw-r--r--. 1 root root 212 Dec 14 15:50 port.discovery.bash
[root@zbx-target script]# chmod +x port.discovery.bash 
[root@zbx-target script]# ./port.discovery.bash 
{"data":[{"{#OPENPORT}":"57091"},{"{#OPENPORT}":"55581"},{"{#OPENPORT}":"10050"},{"{#OPENPORT}":"631"},{"{#OPENPORT}":"111"},{"{#OPENPORT}":"25"},{"{#OPENPORT}":"22"},{"{#OPENPORT}":"END"}]}
[root@zbx-target script]# cat port.discovery.bash 
#!/bin/bash

printf '{"data":['


for i in `netstat -tnl| grep  LISTEN|awk '{print $4}'| awk -F ':' '{print $NF}' | sort -run`
do 
	printf "{\"{#OPENPORT}\":\"%d\"}," $i
done


echo -e '{"{#OPENPORT}":"END"}]}' 
[root@zbx-target script]# 
{% endhighlight %}

这个脚本是用来进行端口发现的，作为基础服务提供给其它监控条目使用


> **Note:**  zabbix用户要有这个脚本的执行权限，因为实际信息收集过程中，是以zabbix这个用户的身份进行的

{% highlight bash %}
[root@zbx-target zabbix_agentd.d]# ps faux | grep zabbix  | grep -v "grep"
zabbix   26928  0.0  0.0  77336  1136 ?        S    17:02   0:00 zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
zabbix   26930  0.0  0.0  77388  2028 ?        S    17:02   0:00  \_ zabbix_agentd: collector [idle 1 sec]          
zabbix   26931  0.0  0.0  77388  1372 ?        S    17:02   0:00  \_ zabbix_agentd: listener #1 [waiting for connection]
zabbix   26932  0.0  0.0  77388  1368 ?        S    17:02   0:00  \_ zabbix_agentd: listener #2 [waiting for connection]
zabbix   26933  0.0  0.0  77388  1412 ?        S    17:02   0:00  \_ zabbix_agentd: listener #3 [waiting for connection]
zabbix   26934  0.0  0.0  77344  1128 ?        S    17:02   0:00  \_ zabbix_agentd: active checks #1 [idle 1 sec]   
[root@zbx-target zabbix_agentd.d]# 
{% endhighlight %}

---

##配置监控插件

{% highlight bash %}
[root@zbx-target zabbix_agentd.d]# vim userparameter_DIY.conf
[root@zbx-target zabbix_agentd.d]# cat userparameter_DIY.conf 
#UserParameter=swap.in.ps,/usr/bin/sar -W 1 1  | grep Average | awk {'print $2'}
#UserParameter=swap.out.ps,/usr/bin/sar -W 1 1  | grep Average | awk {'print $3'}
UserParameter=mem.used,/usr/bin/free -k | grep + | awk '{print $3}'
UserParameter=ps.proc.sum[*],/bin/ps faux   | grep "$1" | grep -v 'grep' | awk 'BEGIN{sum=0;}{sum=sum+$$$2;}END{print sum*1024;}'  
UserParameter=ps.proc[*],/bin/ps faux   | grep "$1" | grep -v 'grep' | awk '{print $$$2*1024}' | head -n 1 
UserParameter=ps.proc.psum[*],/bin/ps faux   | grep "$1" | grep -v 'grep' | awk 'BEGIN{sum=0;}{sum=sum+$$$2;}END{print sum;}'  
UserParameter=ps.proc.p[*],/bin/ps faux   | grep "$1" | grep -v 'grep' | awk '{print $$$2}' | head -n 1 
UserParameter=redis.stat[*],/usr/local/bin/redis-cli  -h 127.0.0.1 -p $1  info  $2 | grep $3: | cut -f 2 -d ':' 
UserParameter=port.discovery,/etc/zabbix/zabbix_agentd.d/script/port.discovery.bash
UserParameter=kernal.sysctl[*], (/sbin/sysctl -n $1  2> /dev/null || /sbin/sysctl -n $2  2> /dev/null ) | /bin/awk '{print $$$3}'
UserParameter=mongo.slowlog[*], /usr/bin/tail -n $1 $2  | awk 'BEGIN{sum=0;}{sum= sum+($NF-0)}END{print sum/$1}'
UserParameter=mysql.slowlog[*], /usr/bin/tail -n $1 $2 | grep Query_time   | awk 'BEGIN{sum=0;}{sum= sum+($$3-0)}END{print sum/NR}'
[root@zbx-target zabbix_agentd.d]#
{% endhighlight %}

配置完监控插件后，要重启agent

> **Note:** 如果不重启，就读不到新添的配置，从服务端尝试获取信息，会出现如下报错

{% highlight bash %}
[root@zbx-server zabbix_agentd.d]#  zabbix_get -s zbx-target -p 10050 -k "mem.used"
ZBX_NOTSUPPORTED: Unsupported item key.
[root@zbx-server zabbix_agentd.d]#  zabbix_get -s zbx-target -p 10050 -k "port.discovery"
ZBX_NOTSUPPORTED: Unsupported item key.
[root@zbx-server zabbix_agentd.d]# 
{% endhighlight %}

重启agent

{% highlight bash %}
[root@zbx-target zabbix_agentd.d]# /etc/init.d/zabbix-agent restart 
Shutting down Zabbix agent:                                [  OK  ]
Starting Zabbix agent:                                     [  OK  ]
[root@zbx-target zabbix_agentd.d]# 
{% endhighlight %}

然后再尝试从服务端进行信息采集

{% highlight bash %}
[root@zbx-server zabbix_agentd.d]#  zabbix_get -s zbx-target -p 10050 -k "port.discovery"
{"data":[{"{#OPENPORT}":"57091"},{"{#OPENPORT}":"55581"},{"{#OPENPORT}":"10050"},{"{#OPENPORT}":"10010"},{"{#OPENPORT}":"631"},{"{#OPENPORT}":"111"},{"{#OPENPORT}":"25"},{"{#OPENPORT}":"22"},{"{#OPENPORT}":"END"}]}
[root@zbx-server zabbix_agentd.d]#  zabbix_get -s zbx-target -p 10050 -k "mem.used"
623308
[root@zbx-server zabbix_agentd.d]# 
{% endhighlight %}

一切正常

到此，基础工作已经完成，在此前提下，配置  **[Templates][templates]**，创建 **[Graphs][graphs]** ，拼接 **[Screens][screens]** 就可以展示出非常炫目的dashboard效果

---

#命令总结

* **`wget http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm`**
* **`zabbix_get -s zbx-target -p 10050 -k "system.cpu.load[all,avg1]"`**
* **`zabbix_get -s zbx-target -p 10050 -k "system.cpu.load[all,avg5]"`**
* **`zabbix_get -s zbx-target -p 10050 -k "system.cpu.load[all,avg15]"`**
* **`zabbix_get -s zbx-target -p 10050 -k "net.tcp.listen[10050]"`**
* **`zabbix_get -s zbx-target -p 10050 -k "net.tcp.listen[10051]"`**
* **`zabbix_get -s zbx-target -p 10050 -k "system.boottime"`**
* **`zabbix_get -s zbx-target -p 10050 -k "agent.version"`**
* **`zabbix_get -s zbx-target -p 10050 -k "net.if.discovery"`**
* **`zabbix_get -s zbx-target -p 10050 -k "system.cpu.discovery"`**
* **`zabbix_get -s zbx-target -p 10050 -k "system.sw.arch"`**
* **`cat port.discovery.bash `**
* **`cat userparameter_DIY.conf `**
* **`zabbix_get -s zbx-target -p 10050 -k "port.discovery"`**
* **`zabbix_get -s zbx-target -p 10050 -k "mem.used"`**


---

[zabbix]:http://www.zabbix.com/
[features]:https://www.zabbix.com/documentation/2.4/manual/introduction/features
[zbx_doc]:https://www.zabbix.com/documentation/2.4/
[zbx_items]:https://www.zabbix.com/documentation/2.4/manual/appendix/items/supported_by_platform
[zabbix_agent]:https://www.zabbix.com/documentation/2.4/manual/config/items/itemtypes/zabbix_agent
[templates]:https://www.zabbix.com/documentation/2.4/manual/config/templates
[graphs]:https://www.zabbix.com/documentation/2.4/manual/config/visualisation/graphs
[screens]:https://www.zabbix.com/documentation/2.4/manual/config/visualisation/screens

