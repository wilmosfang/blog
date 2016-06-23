---
layout: post
title: Mysql MHA 搭建 (三) keepalived 1.2.13 安装 
author: wilmosfang
tags:  mha mysql keepalived cluster 
categories: mysql
wc: 426  1026 15053 
excerpt: mha 切换前后架构图，keepalived 的安装，测试，sudo 配置，修改 master_ip_failover 和 master_ip_online_change 脚本
comments: true
---



# 前言


继前一篇[Mysql MHA 搭建 (二) mha0.53 安装 ][mha2] , 这篇继续[keepalived][keepalived] 的安装与配置部分

keepalived和LVS配合使用是开源界比较流行的LB解决方案，keepalived使用VRRP和心跳对VIP进行管理，可以有效进行failover，我们的mha方案中要使用到这个特性，因此要在两台候选主节点上进行安装

>当前最新版本是[1.2.15][keepalived download] 发布于2014.12.21 


# 概要

* TOC
{:toc}


---

## 架构图


#### 切换前

|Host| IP | VIP   |Role|
|:---|:---:|:---:|---:|
|m1|192.168.75.11|192.168.66.66/24|Master|
|m2|192.168.75.12|standby|Backup|
|s|192.168.75.13|null|null|

#### 切换后

|Host| IP | VIP   |Role|
|:---|:---:|:---:|---:|
|m1|192.168.75.11|-|Stoped|
|m2|192.168.75.12|192.168.66.66/24|Master|
|s|192.168.75.13|null|null|


---

## 安装keepalived 1.2.13


由于系统软件中已经有相应rpm包，所以安装非常简单 , m1 m2都要进行安装

~~~
[root@localhost tmp]# yum -y install keepalived.x86_64
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirror.neu.edu.cn
 * updates: mirrors.btte.net
Resolving Dependencies
--> Running transaction check
---> Package keepalived.x86_64 0:1.2.13-4.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================
 Package                       Arch                      Version                           Repository               Size
=========================================================================================================================
Installing:
 keepalived                    x86_64                    1.2.13-4.el6                      base                    214 k

Transaction Summary
=========================================================================================================================
Install       1 Package(s)

Total download size: 214 k
Installed size: 625 k
...
...
Installed:
  keepalived.x86_64 0:1.2.13-4.el6

Complete!
[root@localhost tmp]#
~~~

**keepalived**的默认配置文件是**/etc/keepalived/keepalived.conf**，里面包含了LVS的配置部分，由于我们的环境里没有用到LVS , 所以直接把相关配置删掉，最后修改完成的效果是这样的

~~~
[root@m1 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
#   notification_email {
#     acassen@firewall.loc
#     failover@firewall.loc
#     sysadmin@firewall.loc
#   }
#   notification_email_from Alexandre.Cassen@firewall.loc
#   smtp_server 192.168.200.1
#   smtp_connect_timeout 30
   router_id LVS_m1      # route id , 用来区分不同主机的
}

vrrp_instance VI_66 {    #实例号，多个实例环境中，用来区分配置的
    state MASTER         #这个是初始化的状态，几乎不起作用，可以随便设置
    interface eth4       #IP绑定的网卡口，非常重要，绑错，问题就大了
    virtual_router_id 66 #这个相同组的实例必须有相同id
    priority 93          #非常重要，决定了谁是master,范围是1-255
    advert_int 1 
    authentication {
        auth_type PASS
        auth_pass 1111 
    }
    virtual_ipaddress {
	192.168.66.66/24 #VIP,可以设置多个
    }
}
[root@m1 ~]# 
~~~

**router_id,interface,virtual_router_id,priority,virtual_ipaddress** , 这几个参数非常重要，一定要配置正确

将m1的keepalived配置文件同步到m2上

~~~
[root@localhost keepalived]# rsync  -v keepalived.conf m2:/etc/keepalived/
root@m2's password:
keepalived.conf

sent 623 bytes  received 67 bytes  197.14 bytes/sec
total size is 544  speedup is 0.79
[root@localhost keepalived]#
~~~

在m2上将**router_id,interface,priority**进行相应修改

---

## 测试keepalived


keepalived启停测试，观察浮动IP的挂载情况

~~~
[root@localhost keepalived]# /etc/init.d/keepalived  start
Starting keepalived: [  OK  ]
[root@localhost keepalived]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.11/24 brd 192.168.75.255 scope global eth4
    inet 192.168.66.66/24 scope global eth4
    inet6 fe80::20c:29ff:fe25:abec/64 scope link
       valid_lft forever preferred_lft forever
3: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.92/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe25:abf6/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost keepalived]# /etc/init.d/keepalived  stop
Stopping keepalived: [  OK  ]
[root@localhost keepalived]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.11/24 brd 192.168.75.255 scope global eth4
    inet6 fe80::20c:29ff:fe25:abec/64 scope link
       valid_lft forever preferred_lft forever
3: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:ab:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.92/24 brd 192.168.1.255 scope global eth3
    inet6 fe80::20c:29ff:fe25:abf6/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost keepalived]#
~~~

再进行主备测试，将m1与m2的keepalived都启动，观察是否优先级高的获得了VIP的挂载权，然后关掉master，观察VIP有没有飘移到backup上，然后主备反过来测试一遍

这一步中最容易出问题的是两个都挂载VIP，完全意识不到另一个的存在，或者是VIP不能按照预期进行飘移，这个时候可以检查**/var/log/messages**系统日志进行排查，检查上面几个参数有没有配置正确，防火墙有没有阻挡VRRP的组播

>**Tip:** 
> 可以通过下面两种方法解决防火墙的问题
>
>  *  关闭防火墙
>
>  *  防火墙里有这一条**ACCEPT all -- anywhere             vrrp.mcast.net** 设置方法 **-A INPUT -d 224.0.0.18 -j ACCEPT**

---

## 配置sudo


配置好keepalived，确保它可以按照预期工作后，开始给mysql提升权限，让它可以操作keepalived的启停

~~~
[root@m1 ~]# visudo
~~~

在末尾添加如下几行

~~~
#mysql sudo keepalived
Host_Alias HOSTKEEP =192.168.75.11
Cmnd_Alias COMKEEP = /etc/init.d/keepalived stop , /etc/init.d/keepalived start , /etc/init.d/keepalived restart , /etc/init.d/keepalived reload ## reload is used to check connection error
User_Alias USERKEEP = mysql
USERKEEP  HOSTKEEP=(ALL)  NOPASSWD:COMKEEP
~~~

同理，在m2上也加上如上几行，只是把IP改为m2自己的ip

强烈推荐使用**visudo**而不是直接使用文本编辑器直接修改**/etc/sudoers**，就像这样

~~~
[root@m1 ~]# vim /etc/sudoers
~~~
因为visudo会进行语法检查，而其它方法不会

同时，注释掉配置中下面这行

~~~
#Defaults    requiretty
~~~
或者也可以使用下面方式，使之无效

~~~
Defaults    !requiretty
~~~

>这个坑浪费掉了我半天的时间，都是因为不想使用高权限的root直接操作，而使用低权限的mysql进行操作产生的问题。低权限得提权，提权我选择使用sudo，而sudo在远程操作的时候，默认是要求有tty的，而mha在非计划failover情况下，是一个后台进程进行监视与控制，没有tty，于是对IP的操作部分会失败，解决办法就是如上所述

致此，keepalived的安装与配置已经完成

下面对mha与keepalived进行结合，让它们可以一起配合完成mysql的故障切换和ip漂移

---

## 修改脚本


前一篇中提到两个脚本：

**master_ip_failover_script= /home/mysql/mha/script/master_ip_failover**

**master_ip_online_change_script= /home/mysql/mha/script/master_ip_online_change**

它们分别是进行**非计划故障切换**和**计划性故障切换**时被mha调用的脚本

所谓**非计划故障切换**，就是作为一个后台进程运行在**manager**上，对当前的数据库运行状态进行监控，如果发现**master**工作不正常了，立即执行切换操作，利用binlog同步数据库，使候选master升级为新的master，slave指向新的master，然后把ip飘移到新的master上，设定新master可写

>**Tip:** 完成切换后，自己会退出，下次运行，得手动修复，手动启动，和我们平时的HA不太一样，它把计划和非计划切换分成了两种情况区别对待，并且只确保第一次的打击

这个过程中，数据库的监控，同步与迁移工作是由mha软件来完成，但是它完成不了VIP的迁移，但是我们可以根据它提供的脚本来和keepalived进行对接完成VIP的迁移，这个脚本就是**master_ip_failover**

---

### master\_ip\_failover

这个脚本在**mha4mysql-manager-0.53.tar.gz**中有模板

~~~
[root@s tmp]# ll  mha4mysql-manager-0.53.tar.gz
-rw-r--r--. 1 root root 105797 Mar 18 17:28 mha4mysql-manager-0.53.tar.gz
[root@s tmp]# ll  mha4mysql-manager-0.53
total 76
-rw-r--r--. 1 root root    78 Oct 26  2011 AUTHORS
drwxr-xr-x. 2 root root  4096 Jan  8  2012 bin
-rw-r--r--. 1 root root 17987 Oct 26  2011 COPYING
drwxr-xr-x. 2 root root  4096 Jan  8  2012 debian
drwxr-xr-x. 3 root root  4096 Jan  8  2012 inc
drwxr-xr-x. 3 root root  4096 Jan  8  2012 lib
-rwxr-xr-x. 1 root root   406 Oct 26  2011 Makefile.PL
-rw-r--r--. 1 root root  4299 Jan  8  2012 MANIFEST
-rw-r--r--. 1 root root   619 Jan  8  2012 META.yml
-rw-r--r--. 1 root root   258 Oct 26  2011 README
drwxr-xr-x. 2 root root  4096 Jan  8  2012 rpm
drwxr-xr-x. 4 root root  4096 Jan  8  2012 samples
drwxr-xr-x. 2 root root  4096 Jan  8  2012 t
drwxr-xr-x. 3 root root  4096 Jan  8  2012 tests
[root@s tmp]# ll mha4mysql-manager-0.53/samples/scripts/
master_ip_failover       power_manager            
master_ip_online_change  send_report              
[root@s tmp]# ll mha4mysql-manager-0.53/samples/scripts/
total 32
-rwxr-xr-x. 1 root root  3443 Jan  8  2012 master_ip_failover
-rwxr-xr-x. 1 root root  9186 Jan  8  2012 master_ip_online_change
-rwxr-xr-x. 1 root root 11867 Jan  8  2012 power_manager
-rwxr-xr-x. 1 root root  1360 Jan  8  2012 send_report
[root@s tmp]# 
~~~

>**Tip:** 注意 , 不是manager的rpm包，而是tar.gz的源码包

因为我们不必要创建新用户，故注释掉第**83**行和**88**行

~~~
 83       #FIXME_xxx_create_user( $new_master_handler->{dbh} );
 84       $new_master_handler->enable_log_bin_local();
 85       $new_master_handler->disconnect();
 86 
 87       ## Update master ip on the catalog database, etc
 88       #FIXME_xxx;
~~~

修改第**73**和**74**行

~~~
 73       $new_master_handler->connect( $new_master_ip, $new_master_port, "root",
 74         "rootpass", 1 );
~~~


改为我们之前在数据库中设置的**mhauser**的用户名和密码


~~~
 73       $new_master_handler->connect( $new_master_ip, $new_master_port, "mhauser",
 74         "xxx", 1 );
~~~

>**Note:**  mha0.55和mha0.56不必要进行这一步，因为脚本作了升级，这个用户的信息可以直接通过全局或应用配置文件直接传过来，这个版本的差异也trouble了我好长时间

在原来**89**行的地方加入一条脚本

~~~
 88       #FIXME_xxx;
 89       `/usr/bin/ssh -t  mysql\@${orig_master_ip} "sudo /etc/init.d/keepalived stop"`;
 90       $exit_code = 0;
~~~

**\`/usr/bin/ssh -t  mysql\@${orig_master_ip} "sudo /etc/init.d/keepalived stop"\`;** 的作用就是用来远程停止keepalived服务

>**Note:**  **-t**这个参数很重要，也是和tty有关，不加它，会提示没有终端的错误

MHA Manager 会调用 **master_ip_failover_script** 三次

* 1.使用**masterha_check_repl**进行复制检查，或进入master监控时 (for script validity checking)
* 2.在执行**shutdown_script**之前
* 3.在应用了所有的 relay logs到新的master后

我们的自定义脚本就是在第三次的位置执行，用来进行IP迁移操作，我们也可以利用这个顺序在**status**的方法中插入一些代码，进行进一步自定义地检查


---

### master\_ip\_online\_change

与**master_ip_failover**类似，只是**master_ip_online_change**是用来负责计划性切换的，比如硬件升级，数据库升级，例行停机等等，这时mysql master还是正常工作的状态，所以也叫**alive failover**

注释掉第**142**行**237**行

~~~
142       #FIXME_xxx_drop_app_user($orig_master_handler);
...
...
237       #FIXME_xxx_create_app_user($new_master_handler);
~~~

修改第**122，123，136，137，227，228**行，使用mhauser的用户名**mhauser**和他的密码**xxx**替换掉**root**和**rootpass**


~~~
122       $new_master_handler->connect( $new_master_ip, $new_master_port, "root",
123         "rootpass", 1 );
...
...
136       $orig_master_handler->connect( $orig_master_ip, $orig_master_port, "root",
137         "rootpass", 1 );
...
...
227       $new_master_handler->connect( $new_master_ip, $new_master_port, "root",
228         "rootpass", 1 );
~~~

在**241**行下面插入**\`/usr/bin/ssh -t  mysql\@${orig_master_ip} "sudo /etc/init.d/keepalived stop"\`;**

~~~
241       ## Update master ip on the catalog database, etc
242       `/usr/bin/ssh -t  mysql\@${orig_master_ip} "sudo /etc/init.d/keepalived stop"`;
243       $exit_code = 0;
~~~

致此，failover脚本已经配置完毕

下一篇将演示如何进行故障切换

---

# 命令汇总

* **`yum -y install keepalived.x86_64`**
* **`cat /etc/keepalived/keepalived.conf`**
* **`rsync  -v keepalived.conf m2:/etc/keepalived/`**
* **`/etc/init.d/keepalived  start`**
* **`/etc/init.d/keepalived  stop`**
* **`ip a`**
* **`visudo`**
* **`vim /etc/sudoers`**
* **`ll  mha4mysql-manager-0.53.tar.gz`**
* **`ll  mha4mysql-manager-0.53`**
* **`ll mha4mysql-manager-0.53/samples/scripts/`**


---

[mha2]:http://soft.dog/2015/03/27/mysql-HA-build-mha-install-and-config/
[keepalived]:http://www.keepalived.org/
[keepalived download]:http://www.keepalived.org/download.html



