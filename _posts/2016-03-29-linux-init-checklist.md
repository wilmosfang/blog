---
layout: post
title:  Linux 初始化检查列表
author: wilmosfang
categories:  linux 
tags:    linux
wc:  751  2339 28173 
excerpt:  linux 的初始化检查列表，包括环境，网络联通，基础配置，主机名，系统更新，时间同步，安全配置，口令更改，创建用户，防火墙配置，SELINUX，RSA Key，参数调整，文件系统调优，文件句柄数调整，软件相关，软件仓库，监控和其它注意事项 
comments: true
---



# 前言

一台新开的云主机，我们往往需要对其进行初始化，或加入一些简单的调优参数，以适应测试或生产的基本需求

这里简要分享一下Linux初始化的检查列表，以帮忙更为高效地进行检查确认

---


# 概要

* TOC
{:toc}



---

## 环境

这里以一台Centos6的服务器进行演示

~~~
[root@localhost ~]# uname -a 
Linux localhost.localdomain 2.6.32-573.el6.x86_64 #1 SMP Thu Jul 23 15:44:03 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# lsb_release 
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
[root@localhost ~]# cat /etc/issue
CentOS release 6.7 (Final)
Kernel \r on an \m

[root@localhost ~]#
~~~


---

## 网络

网络是最先要解决的问题

### 外网联通

首先要配通网络，以使其可以获得各种网络资源

网络包含：

* IPADDR/NETMASK
* GATEWAY
* DNS

~~~
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth1 
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth1  
DEVICE=eth1
TYPE=Ethernet
UUID=c3a6fb7b-0970-4d76-8871-26e18d500ceb
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.22.121
NETMASK=255.255.255.0
GATEWAY=192.168.22.10
DNS1=210.22.84.3
DNS2=114.114.114.114
[root@localhost ~]# /etc/init.d/network restart 
Shutting down interface eth1:                              [  OK  ]
Shutting down loopback interface:                          [  OK  ]
Bringing up loopback interface:                            [  OK  ]
Bringing up interface eth1:  Determining if ip address 192.168.22.121 is already in use for device eth1...
                                                           [  OK  ]
[root@localhost ~]# ping www.baidu.com
PING www.a.shifen.com (115.239.210.27) 56(84) bytes of data.
64 bytes from 115.239.210.27: icmp_seq=1 ttl=52 time=5.48 ms
64 bytes from 115.239.210.27: icmp_seq=2 ttl=52 time=5.42 ms
^C
--- www.a.shifen.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1155ms
rtt min/avg/max/mdev = 5.427/5.456/5.486/0.079 ms
[root@localhost ~]#
~~~

### 内网联通


内网的联通需要域名解析(或主机名解析)，大的网络可以自己构建DNS服务器，小的网络可以使用 **/etc/hosts** 替代

添加主机列表到里面

~~~
[root@localhost ~]# cat  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
[root@localhost ~]# 
~~~

---

## 基础配置

### 主机名

每台主机最好都配置上主机名以示区别，否则都是 **localhost** ，敲错命令后果很难估量

只用修改两个地方 

* /etc/hosts
* /etc/networks

~~~
[root@localhost ~]# vim /etc/networks 
[root@localhost ~]# cat /etc/networks 
default 0.0.0.0
loopback 127.0.0.0
link-local 169.254.0.0
HOSTNAME=check-list
[root@localhost ~]#
~~~

### 更新系统


更新系统可以确保打上了新的补丁，使系统更为健壮

使用 **yum update** 对系统进行更新

~~~
[root@localhost ~]# yum update 
Loaded plugins: fastestmirror, security
Setting up Update Process
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * extras: mirrors.skyshe.cn
 * updates: mirrors.skyshe.cn
base                                                                                | 3.7 kB     00:00     
base/primary_db                                                                     | 4.6 MB     00:04     
extras                                                                              | 3.4 kB     00:00     
extras/primary_db                                                                   |  35 kB     00:00     
updates                                                                             | 3.4 kB     00:00     
updates/primary_db                                                                  | 4.6 MB     00:04     
Resolving Dependencies
--> Running transaction check
---> Package SDL.x86_64 0:1.2.14-6.el6 will be updated
---> Package SDL.x86_64 0:1.2.14-7.el6_7.1 will be an update
...
...
 zip                                   x86_64       3.0-1.el6_7.1                      updates       259 k
Installing for dependencies:
 pcsc-lite-libs                        x86_64       1.5.2-15.el6                       base           28 k

Transaction Summary
===========================================================================================================
Install       2 Package(s)
Upgrade     170 Package(s)

Total download size: 243 M
Is this ok [y/N]: y
Downloading Packages:
(1/172): SDL-1.2.14-7.el6_7.1.x86_64.rpm                                            | 193 kB     00:00     
(2/172): bash-4.1.2-33.el6_7.1.x86_64.rpm 
...
...
(171/172): udev-147-2.63.el6_7.1.x86_64.rpm                                         | 355 kB     00:00     
(172/172): zip-3.0-1.el6_7.1.x86_64.rpm                                             | 259 kB     00:00     
-----------------------------------------------------------------------------------------------------------
Total                                                                      1.0 MB/s | 243 MB     04:00     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
Importing GPG key 0xC105B9DE:
 Userid : CentOS-6 Key (CentOS 6 Official Signing Key) <centos-6-key@centos.org>
 Package: centos-release-6-7.el6.centos.12.3.x86_64 (@anaconda-CentOS-201508042137.x86_64/6.7)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : tzdata-java-2016c-1.el6.noarch                                                        1/342 
  Updating   : kernel-firmware-2.6.32-573.22.1.el6.noarch
...
...
  tmpwatch.x86_64 0:2.9.16-5.el6_7                                                                         
  tzdata.noarch 0:2016c-1.el6                                                                              
  tzdata-java.noarch 0:2016c-1.el6                                                                         
  udev.x86_64 0:147-2.63.el6_7.1                                                                           
  zip.x86_64 0:3.0-1.el6_7.1                                                                               

Complete!
[root@localhost ~]# yum update 
Loaded plugins: fastestmirror, security
Setting up Update Process
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * extras: mirrors.skyshe.cn
 * updates: mirrors.skyshe.cn
No Packages marked for Update
[root@localhost ~]# 
~~~

> **Tip:** 更新完成后，对服务器进行重启 **init 6**


### 同步时间

在同一个网络中，最好将时间进行统一，否则日志信息都会误导分析，更不用说一些对时间非常敏感的服务了

~~~
[root@check-list ~]# cp /etc/ntp.conf /etc/ntp.conf.bak.160329
[root@check-list ~]# vim /etc/ntp.conf
[root@check-list ~]# ntpdate  ntp-server
29 Mar 16:07:09 ntpdate[6657]: step time server 192.168.22.123 offset 29060.498313 sec
[root@check-list ~]# date 
Tue Mar 29 16:07:13 CST 2016
[root@check-list ~]# chkconfig --list | grep ntp 
ntpd           	0:off	1:off	2:off	3:off	4:off	5:off	6:off
ntpdate        	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@check-list ~]# chkconfig ntpd on 
[root@check-list ~]# /etc/init.d/ntpd start
Starting ntpd:                                             [  OK  ]
[root@check-list ~]# chkconfig --list | grep ntp 
ntpd           	0:off	1:off	2:on	3:on	4:on	5:on	6:off
ntpdate        	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@check-list ~]# 
~~~

---

## 安全

### 更改root口令

云主机服务商提供了初始登录密码，但显然不是一个安全的密码，需要进行修改

~~~
[root@check-list ~]# passwd
Changing password for user root.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@check-list ~]#
~~~

### 禁止root ssh登录

禁止root的ssh登录可以有效防止通过直接破解root密码来获取系统最高权限，或者通过多次的尝试失败来进行登录的DOS攻击

~~~
[root@check-list ~]# grep RootLogin /etc/ssh/sshd_config 
#PermitRootLogin yes
# the setting of "PermitRootLogin without-password".
[root@check-list ~]# vim /etc/ssh/sshd_config 
[root@check-list ~]# grep RootLogin /etc/ssh/sshd_config 
#PermitRootLogin yes
PermitRootLogin no
# the setting of "PermitRootLogin without-password".
[root@check-list ~]#
~~~

要使生效，得重启sshd服务


### 创建管理用户

不能直接使用root登录，就得创建管理员用户，来登录管理(不能登录系统，就没法管)

并且要赋予sudo权限

~~~
[root@check-list ~]# useradd saops 
[root@check-list ~]# passwd saops
Changing password for user saops.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@check-list ~]# visudo 
----------
User_Alias USERSU = saops
USERSU  ALL=(root)  ALL
~~~

### 防火墙设置

防火墙是安全领域中的重要环节，能够有效过滤掉非法访问

确认防火墙是开启的，并且只有22号端口是开放的，以后随着业务的扩展会逐步更新防火墙配置

~~~
[root@check-list ~]# chkconfig --list | grep ipta
iptables       	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@check-list ~]# iptables -L -nv 
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 2120  171K ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
    1    84 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    2   120 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
    3   494 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT 1480 packets, 171K bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@check-list ~]# 
~~~

> **Tip:** 前端等特殊情况下最好也安装一下 **dynfw** ， 它提供的 **ipdrop、tcplimit、host-tcplimit** 能够有效抵抗恶意访问或攻击



### SELINUX


一般选择关闭SELINUX，虽然SELINUX会提升系统安全级别，但是会给很多应用的运行造成困扰，也有很大的性能开销，如果不是极其注重安全的领域，建议关闭SELINUX

~~~
[root@check-list ~]# getenforce 
Enforcing
[root@check-list ~]# vim /etc/sysconfig/selinux 
[root@check-list ~]# grep SELINUX /etc/sysconfig/selinux
# SELINUX= can take one of these three values:
#SELINUX=enforcing
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
SELINUXTYPE=targeted 
[root@check-list ~]#
~~~
 
 重启后就可以生效，如果要立刻生效，可以使用 **setenforce 0**

~~~
[root@check-list ~]# getenforce 
Enforcing
[root@check-list ~]# setenforce 0
[root@check-list ~]# getenforce 
Permissive
[root@check-list ~]# sestatus -v
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   permissive
Mode from config file:          disabled
Policy version:                 24
Policy from config file:        targeted

Process contexts:
Current context:                unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
Init context:                   system_u:system_r:init_t:s0
/sbin/mingetty                  system_u:system_r:getty_t:s0
/usr/sbin/sshd                  system_u:system_r:sshd_t:s0-s0:c0.c1023

File contexts:
Controlling term:               unconfined_u:object_r:user_devpts_t:s0
/etc/passwd                     system_u:object_r:etc_t:s0
/etc/shadow                     system_u:object_r:shadow_t:s0
/bin/bash                       system_u:object_r:shell_exec_t:s0
/bin/login                      system_u:object_r:login_exec_t:s0
/bin/sh                         system_u:object_r:bin_t:s0 -> system_u:object_r:shell_exec_t:s0
/sbin/agetty                    system_u:object_r:getty_exec_t:s0
/sbin/init                      system_u:object_r:init_exec_t:s0
/sbin/mingetty                  system_u:object_r:getty_exec_t:s0
/usr/sbin/sshd                  system_u:object_r:sshd_exec_t:s0
/lib/libc.so.6                  system_u:object_r:lib_t:s0 -> system_u:object_r:lib_t:s0
/lib/ld-linux.so.2              system_u:object_r:lib_t:s0 -> system_u:object_r:ld_so_t:s0
[root@check-list ~]#
~~~


### RSA key

一般建议不使用密码，而是使用RSA 证书进行登录，并且 RSA证书本身再加密

将有权限登入的公钥添加到 **authorized_keys**

~~~
[saops@check-list ~]$ ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/saops/.ssh/id_rsa): 
Created directory '/home/saops/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/saops/.ssh/id_rsa.
Your public key has been saved in /home/saops/.ssh/id_rsa.pub.
The key fingerprint is:
3f:05:e8:af:c8:f3:42:3b:0b:d5:c6:63:75:a9:9c:6d saops@check-list
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .   .   |
|        . o o    |
|       + o *     |
|      . S + E    |
|     ..o + o     |
|    .. .  +      |
|     o=. . .     |
|      +*o        |
+-----------------+
[saops@check-list ~]$ 
[saops@check-list ~]$ cd .ssh/
[saops@check-list .ssh]$ ls
id_rsa  id_rsa.pub
[saops@check-list .ssh]$ vim  authorized_keys
[saops@check-list .ssh]$ ll authorized_keys 
-rw-rw-r--. 1 saops saops 1209 Mar 29 17:11 authorized_keys
[saops@check-list .ssh]$ chmod 600 authorized_keys 
[saops@check-list .ssh]$ ll 
total 12
-rw-------. 1 saops saops 1209 Mar 29 17:11 authorized_keys
-rw-------. 1 saops saops 1675 Mar 29 17:07 id_rsa
-rw-r--r--. 1 saops saops  395 Mar 29 17:07 id_rsa.pub
[saops@check-list .ssh]$ 
~~~

---

##  参数调整

可以调整部分参数使系统有较好的表现，或放开某些因为安全考虑而显得过于保守的设置，还有一些是基于特定应用场景的定向调优

### 文件系统调优

系统的默认属性是会将最近的读请求时间记录到文件系统的元数据里，这样一次读请求会产生至少一次写请求，在很多场景下，这种特性没有应用价值，所以可以关掉来减少IO开销

在挂载选项里加入 **noatime** 可以提升磁盘读写效率

~~~
[root@check-list ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Tue Mar 29 01:38:43 2016
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup-LogVol01 /                       ext4    defaults        1 1
UUID=1f5f10d2-1ae5-4595-b931-1dfe944d8b0f /boot                   ext4    defaults        1 2
/dev/mapper/VolGroup-LogVol00 swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/sdb1               /data                         ext4    defaults      0 0
[root@check-list ~]# vim /etc/fstab 
[root@check-list ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Tue Mar 29 01:38:43 2016
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup-LogVol01 /                       ext4    defaults,noatime        1 1
UUID=1f5f10d2-1ae5-4595-b931-1dfe944d8b0f /boot                   ext4    defaults        1 2
/dev/mapper/VolGroup-LogVol00 swap                    swap    defaults,noatime        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/sdb1               /data                         ext4    defaults,noatime      0 0
[root@check-list ~]#
~~~


> **Tip:** **noatime** 包含了 **nodiratime** ，所以不必重复指定



### 放开句柄数


默认情况下一个用户只能打开1024个文件句柄，这是出于安全的考虑，linux中一切都是文件，安全的同时也限制了用户能同时操作对象数的上限，但是很多场景中(比如web前端)，会需要打开很多个连接，以对外提供服务，高并发的情形下很容易耗尽这个配额，这时就会产生 **Too many open files** 的报错，如果适当放开这个限制，就可以提供更多的服务


**/proc/sys/fs/file-max、/proc/sys/fs/file-nr** 分别记录了系统中可以打开的最大文件数和当前已经打开的文件数


**/etc/security/limits.conf** 可以配置打开文件句柄数的软硬限制，它是被 **PAM** 模块调用，所以它在每个用户登录时会生效

~~~
[root@check-list ~]# cat /proc/sys/fs/file-max 
3264717
[root@check-list ~]# cat /proc/sys/fs/file-nr 
800	0	3264717
[root@check-list ~]# ulimit -n 
1024
[root@check-list ~]# vim /etc/security/limits.conf 
[root@check-list ~]# su - root 
[root@check-list ~]# ulimit  -n
32768
[root@check-list ~]# tail -n 4 /etc/security/limits.conf
#
#
* soft nofile 32768
* hard nofile 65536
[root@check-list ~]#
~~~

---

## 软件

除了以上的基础配置，有时还要定向安装一些软件包，以提供增强服务

### 软件仓库

**epel** 是一个非常好用的扩展仓库，一般都建议配置上

~~~
[root@check-list ~]# rpm -qa | grep epel
[root@check-list ~]# yum list all | grep "^epel"
epel-release.noarch                      6-8                            extras  
[root@check-list ~]# yum install epel-release.noarch
Loaded plugins: fastestmirror, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * extras: mirrors.skyshe.cn
 * updates: mirrors.skyshe.cn
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:6-8 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                       Arch                    Version               Repository               Size
===========================================================================================================
Installing:
 epel-release                  noarch                  6-8                   extras                   14 k

Transaction Summary
===========================================================================================================
Install       1 Package(s)

Total download size: 14 k
Installed size: 22 k
Is this ok [y/N]: y
Downloading Packages:
epel-release-6-8.noarch.rpm                                                         |  14 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : epel-release-6-8.noarch                                                                 1/1 
  Verifying  : epel-release-6-8.noarch                                                                 1/1 

Installed:
  epel-release.noarch 0:6-8                                                                                

Complete!
[root@check-list ~]# ll /etc/yum.repos.d/
total 32
-rw-r--r--. 1 root root 1991 Aug  4  2015 CentOS-Base.repo
-rw-r--r--. 1 root root  647 Aug  4  2015 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  289 Aug  4  2015 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Aug  4  2015 CentOS-Media.repo
-rw-r--r--. 1 root root 6259 Aug  4  2015 CentOS-Vault.repo
-rw-r--r--. 1 root root  957 Nov  5  2012 epel.repo
-rw-r--r--. 1 root root 1056 Nov  5  2012 epel-testing.repo
[root@check-list ~]# rpm -qa | grep epel 
epel-release-6-8.noarch
[root@check-list ~]# rpm -ql epel-release-6-8.noarch
/etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
/etc/rpm/macros.ghc-srpm
/etc/yum.repos.d/epel-testing.repo
/etc/yum.repos.d/epel.repo
/usr/share/doc/epel-release-6
/usr/share/doc/epel-release-6/GPL
[root@check-list ~]#
~~~




### denyhosts

**denyhosts** 是一款能有效防止通过暴力破解登录系统的软件

>DenyHosts is a Python script that analyzes the sshd server log
messages to determine which hosts are attempting to hack into your
system. It also determines what user accounts are being targeted. It
keeps track of the frequency of attempts from each host and, upon
discovering a repeated attack host, updates the /etc/hosts.deny file
to prevent future break-in attempts from that host.  Email reports can
be sent to a system admin.

~~~
[root@check-list ~]# yum list all | grep -i denyhost
denyhosts.noarch                           2.6-20.el6                   epel    
[root@check-list ~]# yum install denyhosts.noarch
Loaded plugins: fastestmirror, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * epel: mirrors.opencas.cn
 * extras: mirrors.skyshe.cn
 * updates: mirrors.skyshe.cn
Resolving Dependencies
--> Running transaction check
---> Package denyhosts.noarch 0:2.6-20.el6 will be installed
--> Processing Dependency: libselinux-python for package: denyhosts-2.6-20.el6.noarch
--> Running transaction check
---> Package libselinux-python.x86_64 0:2.0.94-5.8.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================
 Package                        Arch                Version                        Repository         Size
===========================================================================================================
Installing:
 denyhosts                      noarch              2.6-20.el6                     epel               90 k
Installing for dependencies:
 libselinux-python              x86_64              2.0.94-5.8.el6                 base              203 k

Transaction Summary
===========================================================================================================
Install       2 Package(s)

Total download size: 292 k
Installed size: 921 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): denyhosts-2.6-20.el6.noarch.rpm                                              |  90 kB     00:00     
(2/2): libselinux-python-2.0.94-5.8.el6.x86_64.rpm                                  | 203 kB     00:00     
-----------------------------------------------------------------------------------------------------------
Total                                                                      479 kB/s | 292 kB     00:00     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Importing GPG key 0x0608B895:
 Userid : EPEL (6) <epel@fedoraproject.org>
 Package: epel-release-6-8.noarch (@extras)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : libselinux-python-2.0.94-5.8.el6.x86_64                                                 1/2 
  Installing : denyhosts-2.6-20.el6.noarch                                                             2/2 
  Verifying  : denyhosts-2.6-20.el6.noarch                                                             1/2 
  Verifying  : libselinux-python-2.0.94-5.8.el6.x86_64                                                 2/2 

Installed:
  denyhosts.noarch 0:2.6-20.el6                                                                            

Dependency Installed:
  libselinux-python.x86_64 0:2.0.94-5.8.el6                                                                

Complete!
[root@check-list ~]# 
[root@check-list ~]# /etc/init.d/denyhosts start
Starting denyhosts:                                        [  OK  ]
[root@check-list ~]#
[root@check-list ~]# ps faux | grep denyho
root      7961  0.0  0.0 103304   876 pts/0    S+   20:31   0:00                  \_ grep denyho
root      7958  0.0  0.0 189188  6972 ?        S    20:31   0:00 /usr/bin/python /usr/bin/denyhosts.py --daemon --config=/etc/denyhosts.conf
[root@check-list ~]# chkconfig --list | grep deny
denyhosts      	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@check-list ~]# chkconfig denyhosts on
[root@check-list ~]# chkconfig --list | grep deny
denyhosts      	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@check-list ~]#
~~~


---

## 监控

确保一些性能查看和系统监控的软件已经安装

例如：**sysstat、iotop、vmstat、netstat、zabbix-agent** 


---

## 重启

最后为了让所有配置生效，也为了检查重启后所有服务如期启动，得重启一次


---

# 命令汇总

* **`uname -a`**
* **`lsb_release`**
* **`cat /etc/issue`**
* **`cat /etc/sysconfig/network-scripts/ifcfg-eth1`**
* **`ping www.baidu.com`**
* **`cat  /etc/hosts`**
* **`cat /etc/networks`**
* **`yum update`**
* **`cp /etc/ntp.conf /etc/ntp.conf.bak.160329`**
* **`vim /etc/ntp.conf`**
* **`ntpdate  ntp-server`**
* **`chkconfig ntpd on`**
* **`/etc/init.d/ntpd start`**
* **`chkconfig --list | grep ntp`**
* **`passwd`**
* **`vim /etc/ssh/sshd_config`**
* **`grep RootLogin /etc/ssh/sshd_config`**
* **`useradd saops`**
* **`passwd saops`**
* **`visudo`**
* **`chkconfig --list | grep ipta`**
* **`iptables -L -nv`**
* **`getenforce`**
* **`vim /etc/sysconfig/selinux`**
* **`grep SELINUX /etc/sysconfig/selinux`**
* **`setenforce 0`**
* **`getenforce`**
* **`sestatus -v`**
* **`ssh-keygen -t rsa`**
* **`vim  authorized_keys`**
* **`chmod 600 authorized_keys`**
* **`cat /etc/fstab`**
* **`cat /proc/sys/fs/file-max`**
* **`cat /proc/sys/fs/file-nr`**
* **`vim /etc/security/limits.conf`**
* **`su - root`**
* **`ulimit  -n`**
* **`tail -n 4 /etc/security/limits.conf`**
* **`rpm -qa | grep epel`**
* **`yum list all | grep "^epel"`**
* **`yum install epel-release.noarch`**
* **`ll /etc/yum.repos.d/`**
* **`rpm -ql epel-release-6-8.noarch`**
* **`yum list all | grep -i denyhost`**
* **`yum install denyhosts.noarch`**
* **`/etc/init.d/denyhosts start`**
* **`ps faux | grep denyho`**
* **`chkconfig denyhosts on`**
* **`chkconfig --list | grep deny`**


