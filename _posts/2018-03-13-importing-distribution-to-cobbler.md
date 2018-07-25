---
layout: post
title: "Importing Distribution to Cobbler"
author:  wilmosfang
date: 2018-03-13 00:22:04
image: '/assets/img/'
excerpt: '导入发行版到 Cobbler'
main-class: cobbler
color: '#262626'
tags:
 - cobbler
categories:
 - cobbler
twitter_text: 'simple process of Importing Distribution to Cobbler'
introduction: 'Importing Distribution to Cobbler'
---



## 前言

**[Cobbler][cobbler]** 是一款 Linux 系统安装与配置软件

>Cobbler is a Linux installation server that allows for rapid setup of network installation environments. It glues together and automates many associated Linux tasks so you do not have to hop between many various commands and applications when deploying new systems, and, in some cases, changing existing ones. Cobbler can help with provisioning, managing DNS and DHCP, package updates, power management, configuration management orchestration, and much more.

可以实现 Linux 的自动化部署与初始化配置，在需要安装大量 OS 的场景下，可以极大提升效率

前面一篇中在本地搭建了一个 **[Cobbler][cobbler]** 服务

这里分享一下 **[Cobbler][cobbler]** 中导入发行版的方法

参考 **[Cobbler Quickstart Guide][cobbler_ins]**

> **Tip:** 当前的版本为 **LATEST VERSION: 2.8.2 released on Sep 16th 2017**

---

# 操作


## 环境

~~~
[root@56-201 ~]# hostnamectl
   Static hostname: 56-201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: f2d8e34edfd34fae951f7404c7e6d928
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@56-201 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0e:38:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84615sec preferred_lft 84615sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:bb:5d:54 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.201/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febb:5d54/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@56-201 ~]# hostname
56-201
[root@56-201 ~]#
~~~


cobbler 的版本与环境信息

~~~
[root@56-201 ~]# cobbler check
No configuration problems found.  All systems go.
[root@56-201 ~]# cobbler version
Cobbler 2.8.2
  source: ?, ?
  build time: Mon Sep 18 15:16:53 2017
[root@56-201 ~]# ps faux | grep httpd
root      2818  0.0  0.0 112648  1016 pts/0    S+   08:51   0:00          \_ grep --color=auto httpd
root      2678  0.0  0.4 261156  9380 ?        Ss   08:49   0:00 /usr/sbin/httpd -DFOREGROUND
apache    2681  0.0  0.1 263168  3892 ?        S    08:49   0:00  \_ /usr/sbin/httpd -DFOREGROUND
apache    2682  0.0  0.4 265512  8252 ?        S    08:49   0:00  \_ /usr/sbin/httpd -DFOREGROUND
apache    2683  0.0  0.4 265512  8252 ?        S    08:49   0:00  \_ /usr/sbin/httpd -DFOREGROUND
apache    2684  0.0  0.4 265512  8252 ?        S    08:49   0:00  \_ /usr/sbin/httpd -DFOREGROUND
apache    2685  0.0  0.4 265512  8252 ?        S    08:49   0:00  \_ /usr/sbin/httpd -DFOREGROUND
apache    2686  0.0  0.4 265512  8252 ?        S    08:49   0:00  \_ /usr/sbin/httpd -DFOREGROUND
[root@56-201 ~]# ps faux | grep rsync
root       791  0.0  0.0 114644  1016 ?        Ss   08:12   0:00 /usr/bin/rsync --daemon --no-detach
root      2822  0.0  0.0 112648  1016 pts/0    S+   08:51   0:00          \_ grep --color=auto rsync
[root@56-201 ~]# ps faux | grep cobbler
root      2824  0.0  0.0 112648  1012 pts/0    S+   08:52   0:00          \_ grep --color=auto cobbler
root      2731  0.2  1.4 438724 29788 ?        Ss   08:49   0:00 /usr/bin/python2 -s /usr/bin/cobblerd -F
[root@56-201 ~]# cobbler check
No configuration problems found.  All systems go.
[root@56-201 ~]#
~~~


## 挂载镜像

下载官方镜像，并且进行挂载

~~~
[root@56-201 ~]# mkdir /mnt/centos6
[root@56-201 ~]# ll /media/sf_usb/*.iso
-rwxrwx--- 1 root vboxsf 4632608768 6月  11 2017 /media/sf_usb/CentOS-6.6-x86_64-bin-DVD1.iso
[root@56-201 ~]# 
[root@56-201 ~]# mount -t iso9660 -o loop,ro /media/sf_usb/CentOS-6.6-x86_64-bin-DVD1.iso  /mnt/centos6/
[root@56-201 ~]# df -h 
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root   39G  6.5G   32G  17% /
devtmpfs             986M     0  986M   0% /dev
tmpfs               1001M   84K 1001M   1% /dev/shm
tmpfs               1001M  8.8M  992M   1% /run
tmpfs               1001M     0 1001M   0% /sys/fs/cgroup
/dev/sr0             7.8G  7.8G     0 100% /media/cdrom
/dev/sda1           1014M  193M  822M  19% /boot
/dev/mapper/cl-home   19G  538M   19G   3% /home
usb                  257G  222G   35G  87% /media/sf_usb
tmpfs                201M   16K  201M   1% /run/user/42
tmpfs                201M     0  201M   0% /run/user/0
/dev/loop0           4.4G  4.4G     0 100% /mnt/centos6
[root@56-201 ~]# ll /mnt/centos6/
total 712
-r--r--r-- 2 root root     14 10月 24 2014 CentOS_BuildTag
dr-xr-xr-x 3 root root   2048 10月 24 2014 EFI
-r--r--r-- 2 root root    212 11月 28 2013 EULA
-r--r--r-- 2 root root  18009 11月 28 2013 GPL
dr-xr-xr-x 3 root root   2048 10月 24 2014 images
dr-xr-xr-x 2 root root   2048 10月 24 2014 isolinux
dr-xr-xr-x 2 root root 686080 10月 24 2014 Packages
-r--r--r-- 2 root root   1354 10月 20 2014 RELEASE-NOTES-en-US.html
dr-xr-xr-x 2 root root   4096 10月 24 2014 repodata
-r--r--r-- 2 root root   1706 11月 28 2013 RPM-GPG-KEY-CentOS-6
-r--r--r-- 2 root root   1730 11月 28 2013 RPM-GPG-KEY-CentOS-Debug-6
-r--r--r-- 2 root root   1730 11月 28 2013 RPM-GPG-KEY-CentOS-Security-6
-r--r--r-- 2 root root   1734 11月 28 2013 RPM-GPG-KEY-CentOS-Testing-6
-r--r--r-- 1 root root   3380 10月 24 2014 TRANS.TBL
[root@56-201 ~]# 
~~~



## 导入发行版

使用 **`cobbler import`** 导入发行版

~~~
[root@56-201 ~]# cobbler --help 
usage
=====
cobbler <distro|profile|system|repo|image|mgmtclass|package|file> ... 
        [add|edit|copy|getks*|list|remove|rename|report] [options|--help]
cobbler <aclsetup|buildiso|import|list|replicate|report|reposync|sync|validateks|version|signature|get-loaders|hardlink> [options|--help]
[root@56-201 ~]# cobbler import --help 
Usage: cobbler import [options]

Options:
  -h, --help            show this help message and exit
  --arch=ARCH           OS architecture being imported
  --breed=BREED         the breed being imported
  --os-version=OS_VERSION
                        the version being imported
  --path=PATH           local path or rsync location
  --name=NAME           name, ex 'RHEL-5'
  --available-as=AVAILABLE_AS
                        tree is here, don't mirror
  --kickstart=KICKSTART_FILE
                        assign this kickstart file
  --rsync-flags=RSYNC_FLAGS
                        pass additional flags to rsync
[root@56-201 ~]# cobbler import --name=centos6 --arch=x86_64 --path=/mnt/centos6 
task started: 2018-03-13_233830_import
task started (id=Media import, time=Tue Mar 13 23:38:30 2018)
Found a candidate signature: breed=redhat, version=rhel6
Found a matching signature: breed=redhat, version=rhel6
Adding distros from path /var/www/cobbler/ks_mirror/centos6-x86_64:
creating new distro: centos6-x86_64
trying symlink: /var/www/cobbler/ks_mirror/centos6-x86_64 -> /var/www/cobbler/links/centos6-x86_64
creating new profile: centos6-x86_64
associating repos
checking for rsync repo(s)
checking for rhn repo(s)
checking for yum repo(s)
starting descent into /var/www/cobbler/ks_mirror/centos6-x86_64 for centos6-x86_64
processing repo at : /var/www/cobbler/ks_mirror/centos6-x86_64
need to process repo/comps: /var/www/cobbler/ks_mirror/centos6-x86_64
looking for /var/www/cobbler/ks_mirror/centos6-x86_64/repodata/*comps*.xml
Keeping repodata as-is :/var/www/cobbler/ks_mirror/centos6-x86_64/repodata
*** TASK COMPLETE ***
[root@56-201 ~]# echo $?
0
[root@56-201 ~]# 
~~~

**`/var/www/cobbler/ks_mirror/`** 会多出一个文件夹，里面存放着发行版的内容

~~~
[root@56-201 ~]# ll /var/www/cobbler/ks_mirror/
total 4
dr-xr-xr-x  7 root root 4096 10月 24 2014 centos6-x86_64
drwxr-xr-x. 2 root root   33 3月  13 23:44 config
[root@56-201 ~]# ll /var/www/cobbler/ks_mirror/centos6-x86_64/
total 352
-r--r--r-- 1 root root     14 10月 24 2014 CentOS_BuildTag
dr-xr-xr-x 3 root root     35 10月 24 2014 EFI
-r--r--r-- 1 root root    212 11月 28 2013 EULA
-r--r--r-- 1 root root  18009 11月 28 2013 GPL
dr-xr-xr-x 3 root root     95 10月 24 2014 images
dr-xr-xr-x 2 root root    198 10月 24 2014 isolinux
dr-xr-xr-x 2 root root 233472 10月 24 2014 Packages
-r--r--r-- 1 root root   1354 10月 20 2014 RELEASE-NOTES-en-US.html
dr-xr-xr-x 2 root root   4096 10月 24 2014 repodata
-r--r--r-- 1 root root   1706 11月 28 2013 RPM-GPG-KEY-CentOS-6
-r--r--r-- 1 root root   1730 11月 28 2013 RPM-GPG-KEY-CentOS-Debug-6
-r--r--r-- 1 root root   1730 11月 28 2013 RPM-GPG-KEY-CentOS-Security-6
-r--r--r-- 1 root root   1734 11月 28 2013 RPM-GPG-KEY-CentOS-Testing-6
-r--r--r-- 1 root root   3380 10月 24 2014 TRANS.TBL
[root@56-201 ~]# 
~~~

## 列出发行版

~~~
[root@56-201 ~]# cobbler distro --help
usage
=====
cobbler distro add
cobbler distro copy
cobbler distro edit
cobbler distro find
cobbler distro list
cobbler distro remove
cobbler distro rename
cobbler distro report
[root@56-201 ~]# cobbler distro list
   centos6-x86_64
[root@56-201 ~]#
~~~

## 列出 profile

~~~
[root@56-201 ~]# cobbler profile --help 
usage
=====
cobbler profile add
cobbler profile copy
cobbler profile dumpvars
cobbler profile edit
cobbler profile find
cobbler profile getks
cobbler profile list
cobbler profile remove
cobbler profile rename
cobbler profile report
[root@56-201 ~]# cobbler profile list
   centos6-x86_64
[root@56-201 ~]#
~~~

一般导入发行版后都会自动创建一个发行配置和一个 profile 配置，名字相同


## 发行版详情

可以查看发行版的详情

~~~
[root@56-201 ~]# cobbler distro report
Name                           : centos6-x86_64
Architecture                   : x86_64
TFTP Boot Files                : {}
Breed                          : redhat
Comment                        : 
Fetchable Files                : {}
Initrd                         : /var/www/cobbler/ks_mirror/centos6-x86_64/images/pxeboot/initrd.img
Kernel                         : /var/www/cobbler/ks_mirror/centos6-x86_64/images/pxeboot/vmlinuz
Kernel Options                 : {}
Kernel Options (Post Install)  : {}
Kickstart Metadata             : {'tree': 'http://@@http_server@@/cblr/links/centos6-x86_64'}
Management Classes             : []
OS Version                     : rhel6
Owners                         : ['admin']
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Template Files                 : {}

[root@56-201 ~]# cobbler distro report --help
Usage: cobbler [options]

Options:
  -h, --help   show this help message and exit
  --name=NAME  name of object
[root@56-201 ~]# 
~~~

由于目前系统中只有一个发行版本，所以只列出了一个，如果要在多个中进行指定可以加 **`--name=NAME`** 参数进行限定

## 创建系统

~~~
[root@56-201 ~]# cobbler system --help
usage
=====
cobbler system add
cobbler system copy
cobbler system dumpvars
cobbler system edit
cobbler system find
cobbler system getks
cobbler system list
cobbler system poweroff
cobbler system poweron
cobbler system powerstatus
cobbler system reboot
cobbler system remove
cobbler system rename
cobbler system report
[root@56-201 ~]# cobbler system list
[root@56-201 ~]# cobbler system report
[root@56-201 ~]# cobbler system add --name=justfortest_pxe --profile=centos6-x86_64
[root@56-201 ~]# cobbler system list
   justfortest_pxe
[root@56-201 ~]# cobbler system report
Name                           : justfortest_pxe
TFTP Boot Files                : {}
Comment                        : 
Enable gPXE?                   : <<inherit>>
Fetchable Files                : {}
Gateway                        : 
Hostname                       : 
Image                          : 
IPv6 Autoconfiguration         : False
IPv6 Default Device            : 
Kernel Options                 : {}
Kernel Options (Post Install)  : {}
Kickstart                      : <<inherit>>
Kickstart Metadata             : {}
LDAP Enabled                   : False
LDAP Management Type           : authconfig
Management Classes             : <<inherit>>
Management Parameters          : <<inherit>>
Monit Enabled                  : False
Name Servers                   : []
Name Servers Search Path       : []
Netboot Enabled                : True
Owners                         : <<inherit>>
Power Management Address       : 
Power Management ID            : 
Power Management Password      : 
Power Management Type          : ipmitool
Power Management Username      : 
Profile                        : centos6-x86_64
Internal proxy                 : <<inherit>>
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Repos Enabled                  : False
Server Override                : <<inherit>>
Status                         : production
Template Files                 : {}
Virt Auto Boot                 : <<inherit>>
Virt CPUs                      : <<inherit>>
Virt Disk Driver Type          : <<inherit>>
Virt File Size(GB)             : <<inherit>>
Virt Path                      : <<inherit>>
Virt PXE Boot                  : 0
Virt RAM (MB)                  : <<inherit>>
Virt Type                      : <<inherit>>

[root@56-201 ~]# 
~~~


现在已经创建了一个系统配置


## 关闭防火墙

为了排除防火墙的影响，先关闭防火墙

生产环境下建议不要关闭，只用针对开放的端口和主机进行放行

~~~
[root@56-201 ~]# systemctl  status  firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 三 2018-03-14 00:38:40 CST; 10s ago
     Docs: man:firewalld(1)
 Main PID: 5879 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─5879 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
Hint: Some lines were ellipsized, use -l to show in full.
[root@56-201 ~]# 
[root@56-201 ~]# systemctl  stop  firewalld.service
[root@56-201 ~]# systemctl  status  firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since 三 2018-03-14 00:40:23 CST; 6s ago
     Docs: man:firewalld(1)
  Process: 5879 ExecStart=/usr/sbin/firewalld --nofork --nopid $FIREWALLD_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 5879 (code=exited, status=0/SUCCESS)

3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:38:41 56-201 firewalld[5879]: WARNING: COMMAND_FAILED: '/usr/sbin...:
3月 14 00:40:22 56-201 systemd[1]: Stopping firewalld - dynamic firewall d.....
3月 14 00:40:23 56-201 systemd[1]: Stopped firewalld - dynamic firewall daemon.
Hint: Some lines were ellipsized, use -l to show in full.
[root@56-201 ~]# 
[root@56-201 ~]# iptables -L -nv
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@56-201 ~]# 
~~~

## 网络引导

开启 cobbler 上 dhcp 的流量监听，然后创建一台 VM，使其网卡挂载相同网络，并且配置为从网络引导

~~~
[root@56-201 cobbler]# tcpdump -vv -i enp0s8 port 67
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 65535 bytes
00:46:54.831879 IP (tos 0x0, ttl 20, id 0, offset 0, flags [none], proto UDP (17), length 576)
    0.0.0.0.bootpc > 255.255.255.255.bootps: [udp sum ok] BOOTP/DHCP, Request from 08:00:27:6b:dc:e9 (oui Unknown), length 548, xid 0x286bdce9, secs 4, Flags [Broadcast] (0x8000)
	  Client-Ethernet-Address 08:00:27:6b:dc:e9 (oui Unknown)
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: Discover
	    Parameter-Request Option 55, length 36: 
	      Subnet-Mask, Time-Zone, Default-Gateway, Time-Server
	      IEN-Name-Server, Domain-Name-Server, RL, Hostname
	      BS, Domain-Name, SS, RP
	      EP, RSZ, TTL, BR
	      YD, YS, NTP, Vendor-Option
	      Requested-IP, Lease-Time, Server-ID, RN
	      RB, Vendor-Class, TFTP, BF
	      Option 128, Option 129, Option 130, Option 131
	      Option 132, Option 133, Option 134, Option 135
	    MSZ Option 57, length 2: 1260
	    GUID Option 97, length 17: 0.124.249.156.78.119.154.79.196.164.8.32.253.120.85.68.6
	    ARCH Option 93, length 2: 0
	    NDI Option 94, length 3: 1.2.1
	    Vendor-Class Option 60, length 32: "PXEClient:Arch:00000:UNDI:002001"
00:46:54.831895 IP (tos 0x0, ttl 255, id 39432, offset 0, flags [none], proto UDP (17), length 576)
    192.168.56.100.bootps > 255.255.255.255.bootpc: [udp sum ok] BOOTP/DHCP, Reply, length 548, xid 0x286bdce9, Flags [none] (0x0000)
	  Client-IP 192.168.56.101
	  Your-IP 192.168.56.101
	  Client-Ethernet-Address 08:00:27:6b:dc:e9 (oui Unknown)
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    Server-ID Option 54, length 4: 192.168.56.100
	    DHCP-Message Option 53, length 1: Offer
	    Lease-Time Option 51, length 4: 1200
	    Subnet-Mask Option 1, length 4: 255.255.255.0
00:46:55.833473 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    56-201.bootps > 255.255.255.255.bootpc: [udp sum ok] BOOTP/DHCP, Reply, length 300, xid 0x286bdce9, secs 4, Flags [Broadcast] (0x8000)
	  Your-IP 192.168.56.230
	  Server-IP 56-201
	  Client-Ethernet-Address 08:00:27:6b:dc:e9 (oui Unknown)
	  file "pxelinux.0"
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: Offer
	    Server-ID Option 54, length 4: 56-201
	    Lease-Time Option 51, length 4: 21600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    Default-Gateway Option 3, length 4: 56-201
	    Domain-Name-Server Option 6, length 4: 56-201
00:46:56.441373 IP (tos 0x0, ttl 20, id 1, offset 0, flags [none], proto UDP (17), length 576)
    0.0.0.0.bootpc > 255.255.255.255.bootps: [udp sum ok] BOOTP/DHCP, Request from 08:00:27:6b:dc:e9 (oui Unknown), length 548, xid 0x286bdce9, secs 4, Flags [Broadcast] (0x8000)
	  Client-Ethernet-Address 08:00:27:6b:dc:e9 (oui Unknown)
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: Request
	    Requested-IP Option 50, length 4: 192.168.56.230
	    Parameter-Request Option 55, length 36: 
	      Subnet-Mask, Time-Zone, Default-Gateway, Time-Server
	      IEN-Name-Server, Domain-Name-Server, RL, Hostname
	      BS, Domain-Name, SS, RP
	      EP, RSZ, TTL, BR
	      YD, YS, NTP, Vendor-Option
	      Requested-IP, Lease-Time, Server-ID, RN
	      RB, Vendor-Class, TFTP, BF
	      Option 128, Option 129, Option 130, Option 131
	      Option 132, Option 133, Option 134, Option 135
	    MSZ Option 57, length 2: 1260
	    Server-ID Option 54, length 4: 56-201
	    GUID Option 97, length 17: 0.124.249.156.78.119.154.79.196.164.8.32.253.120.85.68.6
	    ARCH Option 93, length 2: 0
	    NDI Option 94, length 3: 1.2.1
	    Vendor-Class Option 60, length 32: "PXEClient:Arch:00000:UNDI:002001"
00:46:56.441388 IP (tos 0x0, ttl 255, id 19791, offset 0, flags [none], proto UDP (17), length 576)
    192.168.56.100.bootps > 255.255.255.255.bootpc: [udp sum ok] BOOTP/DHCP, Reply, length 548, xid 0x286bdce9, Flags [none] (0x0000)
	  Client-IP 192.168.56.101
	  Your-IP 192.168.56.101
	  Client-Ethernet-Address 08:00:27:6b:dc:e9 (oui Unknown)
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    Server-ID Option 54, length 4: 192.168.56.100
	    DHCP-Message Option 53, length 1: ACK
	    Lease-Time Option 51, length 4: 1200
	    Subnet-Mask Option 1, length 4: 255.255.255.0
00:46:56.442707 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    56-201.bootps > 255.255.255.255.bootpc: [udp sum ok] BOOTP/DHCP, Reply, length 300, xid 0x286bdce9, secs 4, Flags [Broadcast] (0x8000)
	  Your-IP 192.168.56.230
	  Server-IP 56-201
	  Client-Ethernet-Address 08:00:27:6b:dc:e9 (oui Unknown)
	  file "pxelinux.0"
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: ACK
	    Server-ID Option 54, length 4: 56-201
	    Lease-Time Option 51, length 4: 21600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    Default-Gateway Option 3, length 4: 56-201
	    Domain-Name-Server Option 6, length 4: 56-201
...
...
...
~~~


发现听到了包，是 dhcp 请求，广播用来获取自己的 IP 地址

不过此网段中有另一个 dhcp 服务器(virtual box)，给发了 56.101 的地址，但是 cobbler 的 dhcp 覆盖了它的配置，分配了一个我们之前设定的地址池(192.168.56.230-192.168.56.235)中的第一个地址 192.168.56.230

生产环境中不要有多个 dhcp, 这会带来一定问题

从 console 中看到的确获取到了我们预期的地址

![cobbler](/assets/img/cobbler/cobbler01.png)

但是在连接 tftp 服务器的时候获不到响应

![cobbler](/assets/img/cobbler/cobbler02.png)

最后由于超时而退出 PXE 引导过程

![cobbler](/assets/img/cobbler/cobbler03.png)

为什么会有这个问题呢

接下来解决这个故障

## tftp server

原因是本地的 tftp server 并没有工作

~~~
[root@56-201 cobbler]# cobbler check
No configuration problems found.  All systems go.
[root@56-201 cobbler]# 
[root@56-201 cobbler]# grep disable /etc/xinetd.d/tftp 
        disable                 = no
[root@56-201 cobbler]# 
~~~

即便检查配置没问题，tftp 也没有按照期望运行

~~~
[root@56-201 ~]# nmap -sU localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2018-03-14 01:03 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000017s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 997 closed ports
PORT     STATE         SERVICE
67/udp   open|filtered dhcps
68/udp   open|filtered dhcpc
5353/udp open|filtered zeroconf

Nmap done: 1 IP address (1 host up) scanned in 2.00 seconds
[root@56-201 ~]#
~~~

是因为 tftp 是由 **xinetd** 管理的，系统中缺少 **xinetd**

将 **xinetd** 安装上

~~~
[root@56-201 dhcp]# rpm -qa | grep xinetd
[root@56-201 dhcp]# yum list all | grep xinetd
xinetd.x86_64                           2:2.3.15-13.el7                base     
[root@56-201 dhcp]# yum install xinetd.x86_64
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
centos-openshift-origin15                                | 2.9 kB     00:00     
epel/x86_64/metalink                                     | 6.7 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
epel/x86_64/primary_db         FAILED                                           
https://linuxmirrors.ir/pub/epel/7/x86_64/repodata/09a27c6f2be460803b31afa82627def106e62024409faec50705d3cfbdb2f99d-primary.sqlite.bz2: [Errno 14] HTTPS Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below knowledge base article 

https://access.redhat.com/articles/1320623

If above article doesn't help to resolve this issue please create a bug on https://bugs.centos.org/

(1/2): epel/x86_64/updateinfo                              | 900 kB   00:05     
(2/2): epel/x86_64/primary_db                              | 6.3 MB   00:27     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media: 
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package xinetd.x86_64 2:2.3.15-13.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package         Arch            Version                    Repository     Size
================================================================================
Installing:
 xinetd          x86_64          2:2.3.15-13.el7            base          128 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 128 k
Installed size: 261 k
Is this ok [y/d/N]: y
Downloading packages:
xinetd-2.3.15-13.el7.x86_64.rpm                            | 128 kB   00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 2:xinetd-2.3.15-13.el7.x86_64                                1/1 
  Verifying  : 2:xinetd-2.3.15-13.el7.x86_64                                1/1 

Installed:
  xinetd.x86_64 2:2.3.15-13.el7                                                 

Complete!
[root@56-201 dhcp]# systemctl status xinetd.service
● xinetd.service - Xinetd A Powerful Replacement For Inetd
   Loaded: loaded (/usr/lib/systemd/system/xinetd.service; enabled; vendor preset: enabled)
   Active: inactive (dead)
[root@56-201 dhcp]# systemctl start xinetd.service
[root@56-201 dhcp]# systemctl status xinetd.service
● xinetd.service - Xinetd A Powerful Replacement For Inetd
   Loaded: loaded (/usr/lib/systemd/system/xinetd.service; enabled; vendor preset: enabled)
   Active: active (running) since 三 2018-03-14 00:28:23 CST; 2s ago
  Process: 5661 ExecStart=/usr/sbin/xinetd -stayalive -pidfile /var/run/xinetd.pid $EXTRAOPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 5662 (xinetd)
   CGroup: /system.slice/xinetd.service
           └─5662 /usr/sbin/xinetd -stayalive -pidfile /var/run/xinetd.pid

3月 14 00:28:23 56-201 xinetd[5662]: removing discard
3月 14 00:28:23 56-201 xinetd[5662]: removing discard
3月 14 00:28:23 56-201 xinetd[5662]: removing echo
3月 14 00:28:23 56-201 xinetd[5662]: removing echo
3月 14 00:28:23 56-201 xinetd[5662]: removing tcpmux
3月 14 00:28:23 56-201 xinetd[5662]: removing time
3月 14 00:28:23 56-201 xinetd[5662]: removing time
3月 14 00:28:23 56-201 xinetd[5662]: xinetd Version 2.3.15 started with li...n.
3月 14 00:28:23 56-201 xinetd[5662]: Started working: 1 available service
3月 14 00:28:23 56-201 systemd[1]: Started Xinetd A Powerful Replacement F...d.
Hint: Some lines were ellipsized, use -l to show in full.
[root@56-201 dhcp]# 
~~~

再进行查看，就会发现有 tftp 服务了


~~~
[root@56-201 cobbler]# ps faux | grep tftp
root      6908  0.0  0.0 112648  1016 pts/1    S+   01:15   0:00          \_ grep --color=auto tftp
[root@56-201 cobbler]# nmap -sU localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2018-03-14 01:15 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000050s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 996 closed ports
PORT     STATE         SERVICE
67/udp   open|filtered dhcps
68/udp   open|filtered dhcpc
69/udp   open|filtered tftp
5353/udp open|filtered zeroconf

Nmap done: 1 IP address (1 host up) scanned in 1.26 seconds
[root@56-201 cobbler]# ps faux | grep tftp
root      6914  0.0  0.0 112648  1016 pts/1    S+   01:15   0:00          \_ grep --color=auto tftp
root      6910  0.0  0.0  10940   808 ?        Ss   01:15   0:00  \_ in.tftpd -B 1380 -v -s /var/lib/tftpboot
[root@56-201 cobbler]# 
~~~

(由于 tftp 是被动触发的，由 xinetd 进行管理，要监听到有请求时才会开启，所以即便开启了 xinetd 的情况下，tftp 也不是马上就启动了，要等至少一个请求产生，这里是用的 nmap 的方式去 ping 它的 UDP69)

## 再次引导

就快速获取了 IP, 并且从 tftp 下载了引导文件

直接进入了安装选择界面

![cobbler](/assets/img/cobbler/cobbler04.png)

选择版本

![cobbler](/assets/img/cobbler/cobbler05.png)

就开始获取内核与镜像

![cobbler](/assets/img/cobbler/cobbler06.png)

自动安装软件包

![cobbler](/assets/img/cobbler/cobbler07.png)

成功安装完毕，整个过程除了选择版本，其它过程没有一次键盘敲击

![cobbler](/assets/img/cobbler/cobbler08.png)

## 登录系统

尝试使用 root 和之前配置的密码进行登录

![cobbler](/assets/img/cobbler/cobbler09.png)

---

# 总结

最核心的其实是理解整个 PXE 引导过程中各个角色的交互顺序和作用，这样可以更容易把握这些零散的组件与关系

已经完成了基本的自动化安装，但是如果需要更多定制化的东西，就需要 Kickstart 文件了

* TOC
{:toc}


---


[cobbler]:http://cobbler.github.io/
[cobbler_ins]:http://cobbler.github.io/manuals/quickstart/
