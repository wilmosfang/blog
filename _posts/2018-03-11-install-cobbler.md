---
layout: post
title: "Install Cobbler"
author:  wilmosfang
date: 2018-03-11 15:18:25
image: '/assets/img/'
excerpt: '安装 Cobbler'
main-class: cobbler
color: '#262626'
tags:
 - cobbler
categories:
 - cobbler
twitter_text: 'simple process of Cobbler installation'
introduction: 'installation of Cobbler'
---


## 前言

**[Cobbler][cobbler]** 是一款 Linux 系统安装与配置软件

>Cobbler is a Linux installation server that allows for rapid setup of network installation environments. It glues together and automates many associated Linux tasks so you do not have to hop between many various commands and applications when deploying new systems, and, in some cases, changing existing ones. Cobbler can help with provisioning, managing DNS and DHCP, package updates, power management, configuration management orchestration, and much more.

可以实现 Linux 的自动化部署与初始化配置，在需要安装大量 OS 的场景下，可以极大提升效率

这里分享一下 **[Cobbler][cobbler]** 的安装方法

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



## 查看包

~~~
[root@56-201 ~]# yum list all | grep cobbler
cobbler.x86_64                          2.8.2-1.el7                    epel     
cobbler-web.noarch                      2.8.2-1.el7                    epel     
[root@56-201 ~]#
~~~

默认情况下 **cobbler** 已经被添加到了 **epel-release** 的软件仓库中


## 查看依赖

**cobbler-web** 依赖于 **cobbler**

~~~
[root@56-201 ~]# yum deplist cobbler-web.noarch
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
package: cobbler-web.noarch 2.8.2-1.el7
  dependency: /bin/sh
   provider: bash.x86_64 4.2.46-29.el7_4
  dependency: cobbler
   provider: cobbler.x86_64 2.8.2-1.el7
  dependency: mod_ssl
   provider: mod_ssl.x86_64 1:2.4.6-67.el7.centos.6
  dependency: mod_wsgi
   provider: mod_wsgi.x86_64 3.4-12.el7_0
  dependency: openssl
   provider: openssl.x86_64 1:1.0.2k-8.el7
  dependency: python-django
   provider: python2-django.noarch 1.6.11.6-16.el7
   provider: python-django.noarch 1.6.11.6-1.el7
[root@56-201 ~]#
~~~

**cobbler-web** 是 **cobbler** 的 web 管理界面，是基于 python-django 构建的，需要 ssl 的支持


**cobbler** 依赖于 **httpd** **rsync**  **tftp-server** **yum-utils**

~~~
[root@56-201 ~]# yum deplist cobbler.x86_64
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
package: cobbler.x86_64 2.8.2-1.el7
  dependency: /bin/sh
   provider: bash.x86_64 4.2.46-29.el7_4
  dependency: /usr/bin/python
   provider: python.x86_64 2.7.5-58.el7
  dependency: /usr/bin/python2
   provider: python.x86_64 2.7.5-58.el7
  dependency: PyYAML
   provider: PyYAML.x86_64 3.10-11.el7
  dependency: createrepo
   provider: createrepo.noarch 0.9.9-28.el7
  dependency: genisoimage
   provider: genisoimage.x86_64 1.1.11-23.el7
  dependency: httpd
   provider: httpd.x86_64 2.4.6-67.el7.centos.6
  dependency: mod_wsgi
   provider: mod_wsgi.x86_64 3.4-12.el7_0
  dependency: python(abi) = 2.7
   provider: python.x86_64 2.7.5-58.el7
  dependency: python-cheetah
   provider: python-cheetah.x86_64 2.4.4-5.el7.centos
  dependency: python-netaddr
   provider: python-netaddr.noarch 0.7.5-7.el7
  dependency: python-simplejson
   provider: python2-simplejson.x86_64 3.10.0-1.el7
  dependency: python-urlgrabber
   provider: python-urlgrabber.noarch 3.10-8.el7
  dependency: rsync
   provider: rsync.x86_64 3.0.9-18.el7
  dependency: syslinux
   provider: syslinux.x86_64 4.05-13.el7
  dependency: systemd
   provider: systemd.x86_64 219-42.el7_4.10
  dependency: tftp-server
   provider: tftp-server.x86_64 5.2-13.el7
  dependency: yum-utils
   provider: yum-utils.noarch 1.1.31-42.el7
[root@56-201 ~]#
~~~

**cobbler** 的核心功能需要以上软件的支持

**rsync yum-utils** 用来进行软件库的更新同步

**tftp-server** 用来提供 bootstrap 和 相关镜像的下载

**httpd** 来提供软件库的下载

**createrepo** 用来构建本地仓库


## 关闭 SELINUX

~~~
[root@56-201 ~]# getenforce
Enforcing
[root@56-201 ~]# setenforce 0
[root@56-201 ~]# getenforce
Permissive
[root@56-201 ~]#
~~~

修改 **`/etc/selinux/config`** 配置文件可以在下次 OS 重启禁用掉 selinux

~~~
[root@56-201 ~]# vim /etc/selinux/config
[root@56-201 ~]# grep  disabled /etc/selinux/config
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
[root@56-201 ~]#
~~~


## 安装 cobbler

~~~
[root@56-201 ~]# yum install cobbler
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
centos-openshift-origin15                                | 2.9 kB     00:00     
epel/x86_64/metalink                                     | 6.3 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/2): epel/x86_64/updateinfo                              | 902 kB   00:18     
(2/2): epel/x86_64/primary_db                              | 6.3 MB   00:26     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package cobbler.x86_64 0:2.8.2-1.el7 will be installed
--> Processing Dependency: PyYAML for package: cobbler-2.8.2-1.el7.x86_64
--> Processing Dependency: mod_wsgi for package: cobbler-2.8.2-1.el7.x86_64
--> Processing Dependency: python-cheetah for package: cobbler-2.8.2-1.el7.x86_64
--> Processing Dependency: python-simplejson for package: cobbler-2.8.2-1.el7.x86_64
--> Processing Dependency: tftp-server for package: cobbler-2.8.2-1.el7.x86_64
--> Running transaction check
---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
---> Package mod_wsgi.x86_64 0:3.4-12.el7_0 will be installed
---> Package python-cheetah.x86_64 0:2.4.4-5.el7.centos will be installed
--> Processing Dependency: python-pygments for package: python-cheetah-2.4.4-5.el7.centos.x86_64
--> Processing Dependency: python-markdown for package: python-cheetah-2.4.4-5.el7.centos.x86_64
---> Package python2-simplejson.x86_64 0:3.10.0-1.el7 will be installed
---> Package tftp-server.x86_64 0:5.2-13.el7 will be installed
--> Running transaction check
---> Package python-markdown.noarch 0:2.4.1-2.el7 will be installed
---> Package python-pygments.noarch 0:1.4-10.el7 will be installed
--> Processing Dependency: python-imaging for package: python-pygments-1.4-10.el7.noarch
--> Running transaction check
---> Package python-pillow.x86_64 0:2.0.0-19.gitd1c6db8.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch       Version                     Repository  Size
================================================================================
Installing:
 cobbler                x86_64     2.8.2-1.el7                 epel       574 k
Installing for dependencies:
 PyYAML                 x86_64     3.10-11.el7                 base       153 k
 mod_wsgi               x86_64     3.4-12.el7_0                base        76 k
 python-cheetah         x86_64     2.4.4-5.el7.centos          extras     341 k
 python-markdown        noarch     2.4.1-2.el7                 epel       186 k
 python-pillow          x86_64     2.0.0-19.gitd1c6db8.el7     base       438 k
 python-pygments        noarch     1.4-10.el7                  base       599 k
 python2-simplejson     x86_64     3.10.0-1.el7                epel       188 k
 tftp-server            x86_64     5.2-13.el7                  base        44 k

Transaction Summary
================================================================================
Install  1 Package (+8 Dependent packages)

Total download size: 2.5 M
Installed size: 11 M
Is this ok [y/d/N]: y
Downloading packages:
(1/9): PyYAML-3.10-11.el7.x86_64.rpm                       | 153 kB   00:00     
(2/9): python-cheetah-2.4.4-5.el7.centos.x86_64.rpm        | 341 kB   00:00     
(3/9): mod_wsgi-3.4-12.el7_0.x86_64.rpm                    |  76 kB   00:00     
(4/9): cobbler-2.8.2-1.el7.x86_64.rpm                      | 574 kB   00:10     
(5/9): python-pillow-2.0.0-19.gitd1c6db8.el7.x86_64.rpm    | 438 kB   00:00     
(6/9): python-pygments-1.4-10.el7.noarch.rpm               | 599 kB   00:01     
python-markdown-2.4.1-2.el7.no FAILED                                          
https://mirrors.ustc.edu.cn/epel/7/x86_64/Packages/p/python-markdown-2.4.1-2.el7.noarch.rpm: [Errno 12] Timeout on https://mirrors.ustc.edu.cn/epel/7/x86_64/Packages/p/python-markdown-2.4.1-2.el7.noarch.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(7/9): tftp-server-5.2-13.el7.x86_64.rpm                   |  44 kB   00:00     
(8/9): python-markdown-2.4.1-2.el7.noarch.rpm              | 186 kB   00:02     
python2-simplejson-3.10.0-1.el FAILED                                          
https://mirrors.ustc.edu.cn/epel/7/x86_64/Packages/p/python2-simplejson-3.10.0-1.el7.x86_64.rpm: [Errno 12] Timeout on https://mirrors.ustc.edu.cn/epel/7/x86_64/Packages/p/python2-simplejson-3.10.0-1.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(9/9): python2-simplejson-3.10.0-1.el7.x86_64.rpm          | 188 kB   00:02     
--------------------------------------------------------------------------------
Total                                               33 kB/s | 2.5 MB  01:19     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : tftp-server-5.2-13.el7.x86_64                                1/9
  Installing : PyYAML-3.10-11.el7.x86_64                                    2/9
  Installing : python-pillow-2.0.0-19.gitd1c6db8.el7.x86_64                 3/9
  Installing : python-pygments-1.4-10.el7.noarch                            4/9
  Installing : python-markdown-2.4.1-2.el7.noarch                           5/9
  Installing : python-cheetah-2.4.4-5.el7.centos.x86_64                     6/9
  Installing : python2-simplejson-3.10.0-1.el7.x86_64                       7/9
  Installing : mod_wsgi-3.4-12.el7_0.x86_64                                 8/9
  Installing : cobbler-2.8.2-1.el7.x86_64                                   9/9
  Verifying  : mod_wsgi-3.4-12.el7_0.x86_64                                 1/9
  Verifying  : python-pygments-1.4-10.el7.noarch                            2/9
  Verifying  : python-cheetah-2.4.4-5.el7.centos.x86_64                     3/9
  Verifying  : cobbler-2.8.2-1.el7.x86_64                                   4/9
  Verifying  : python2-simplejson-3.10.0-1.el7.x86_64                       5/9
  Verifying  : python-markdown-2.4.1-2.el7.noarch                           6/9
  Verifying  : python-pillow-2.0.0-19.gitd1c6db8.el7.x86_64                 7/9
  Verifying  : PyYAML-3.10-11.el7.x86_64                                    8/9
  Verifying  : tftp-server-5.2-13.el7.x86_64                                9/9

Installed:
  cobbler.x86_64 0:2.8.2-1.el7                                                  

Dependency Installed:
  PyYAML.x86_64 0:3.10-11.el7                                                   
  mod_wsgi.x86_64 0:3.4-12.el7_0                                                
  python-cheetah.x86_64 0:2.4.4-5.el7.centos                                    
  python-markdown.noarch 0:2.4.1-2.el7                                          
  python-pillow.x86_64 0:2.0.0-19.gitd1c6db8.el7                                
  python-pygments.noarch 0:1.4-10.el7                                           
  python2-simplejson.x86_64 0:3.10.0-1.el7                                      
  tftp-server.x86_64 0:5.2-13.el7                                               

Complete!
[root@56-201 ~]# echo $?
0
[root@56-201 ~]#
~~~



## 修改密码

~~~
[root@56-201 cobbler]# openssl passwd -1
Password:
Verifying - Password:
$1$BAj5Akia$3PiP6Kl10RpuRuY/B25fV.
[root@56-201 cobbler]# vim settings
[root@56-201 cobbler]# grep  default_password_crypted settings
#default_password_crypted: "$1$mF86/UHC$WvcIcX2t6crBz2onWxyac."
default_password_crypted: "$1$BAj5Akia$3PiP6Kl10RpuRuY/B25fV."
[root@56-201 cobbler]#
~~~

使用 **`openssl passwd -1`** 生成密码来替代原有的密码

这个密码是 kickstart 中的 root 密码

## 修改服务地址

修改 cobbler 服务地址

~~~
[root@56-201 cobbler]# vim settings
[root@56-201 cobbler]# grep -B 8 '^server' settings

# this is the address of the cobbler server -- as it is used
# by systems during the install process, it must be the address
# or hostname of the system as those systems can see the server.
# if you have a server that appears differently to different subnets
# (dual homed, etc), you need to read the --server-override section
# of the manpage for how that works.
#server: 127.0.0.1
server: 192.168.56.201
[root@56-201 cobbler]#
~~~

修改 dhcp 服务地址

~~~
[root@56-201 cobbler]# vim settings
[root@56-201 cobbler]# grep -B 5 '^next_server' settings

# if using cobbler with manage_dhcp, put the IP address
# of the cobbler server here so that PXE booting guests can find it
# if you do not set this correctly, this will be manifested in TFTP open timeouts.
#next_server: 127.0.0.1
next_server: 192.168.56.201
[root@56-201 cobbler]#
~~~

由于在 PXE 引导的过程中 dhcp 服务器可以作为一个独立的角色与 cobbler 分别布到不同的节点上，因此这里的 IP 不一定是本机 IP，但是为了方便，我希望 cobbler 一同管理 dhcp，所以我也准备将 dhcp 服务与 cobbler 放在一起 (cobbler 已经集成了直接管理 dhcp 的功能)


## 管理 dhcp

~~~
[root@56-201 cobbler]# vim settings
[root@56-201 cobbler]# grep -B 4 '^manage_dhcp' settings

# set to 1 to enable Cobbler's DHCP management features.
# the choice of DHCP management engine is in /etc/cobbler/modules.conf
#manage_dhcp: 0
manage_dhcp: 1
[root@56-201 cobbler]#
~~~

默认 cobbler 是不管理 dhcp 的

但是我们通过配置 **manage_dhcp** 选项，可以打开管理

## 修改模板

~~~
[root@56-201 cobbler]# grep -v "#" dhcp.template

ddns-update-style interim;

allow booting;
allow bootp;

ignore client-updates;
set vendorclass = option vendor-class-identifier;

option pxe-system-type code 93 = unsigned integer 16;

subnet 192.168.1.0 netmask 255.255.255.0 {
     option routers             192.168.1.5;
     option domain-name-servers 192.168.1.1;
     option subnet-mask         255.255.255.0;
     range dynamic-bootp        192.168.1.100 192.168.1.254;
     default-lease-time         21600;
     max-lease-time             43200;
     next-server                $next_server;
     class "pxeclients" {
          match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
          if option pxe-system-type = 00:02 {
                  filename "ia64/elilo.efi";
          } else if option pxe-system-type = 00:06 {
                  filename "grub/grub-x86.efi";
          } else if option pxe-system-type = 00:07 {
                  filename "grub/grub-x86_64.efi";
          } else if option pxe-system-type = 00:09 {
                  filename "grub/grub-x86_64.efi";
          } else {
                  filename "pxelinux.0";
          }
     }

}

group {
    host $iface.name {
        option dhcp-client-identifier = $mac;
        hardware ethernet $mac;
        fixed-address $iface.ip_address;
        option host-name "$iface.hostname";
        option subnet-mask $iface.netmask;
        option routers $iface.gateway;
        if exists user-class and option user-class = "gPXE" {
            filename "http://$cobbler_server/cblr/svc/op/gpxe/system/$iface.owner";
        } else if exists user-class and option user-class = "iPXE" {
            filename "http://$cobbler_server/cblr/svc/op/gpxe/system/$iface.owner";
        } else {
            filename "undionly.kpxe";
        }
        filename "$iface.filename";
        next-server $next_server;
    }
}

[root@56-201 cobbler]# vim dhcp.template
[root@56-201 cobbler]# grep -v "#" dhcp.template

ddns-update-style interim;

allow booting;
allow bootp;

ignore client-updates;
set vendorclass = option vendor-class-identifier;

option pxe-system-type code 93 = unsigned integer 16;

subnet 192.168.56.0 netmask 255.255.255.0 {
     option routers             192.168.56.201;
     option domain-name-servers 192.168.56.201;
     option subnet-mask         255.255.255.0;
     range dynamic-bootp        192.168.56.230 192.168.56.235;
     default-lease-time         21600;
     max-lease-time             43200;
     next-server                $next_server;
     class "pxeclients" {
          match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
          if option pxe-system-type = 00:02 {
                  filename "ia64/elilo.efi";
          } else if option pxe-system-type = 00:06 {
                  filename "grub/grub-x86.efi";
          } else if option pxe-system-type = 00:07 {
                  filename "grub/grub-x86_64.efi";
          } else if option pxe-system-type = 00:09 {
                  filename "grub/grub-x86_64.efi";
          } else {
                  filename "pxelinux.0";
          }
     }

}

group {
    host $iface.name {
        option dhcp-client-identifier = $mac;
        hardware ethernet $mac;
        fixed-address $iface.ip_address;
        option host-name "$iface.hostname";
        option subnet-mask $iface.netmask;
        option routers $iface.gateway;
        if exists user-class and option user-class = "gPXE" {
            filename "http://$cobbler_server/cblr/svc/op/gpxe/system/$iface.owner";
        } else if exists user-class and option user-class = "iPXE" {
            filename "http://$cobbler_server/cblr/svc/op/gpxe/system/$iface.owner";
        } else {
            filename "undionly.kpxe";
        }
        filename "$iface.filename";
        next-server $next_server;
    }
}

[root@56-201 cobbler]#
~~~

## 安装 dhcp 服务端

系统默认情况下只安装了 dhcp 客户端需要的包和库，我们需要明确地安装 dhcp 服务端软件

~~~
[root@56-201 cobbler]# rpm -qa | grep dhcp
dhcp-libs-4.2.5-47.el7.centos.x86_64
dhcp-common-4.2.5-47.el7.centos.x86_64
[root@56-201 cobbler]# yum install dhcp.x86_64
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package dhcp.x86_64 12:4.2.5-58.el7.centos.1 will be installed
--> Processing Dependency: dhcp-libs(x86-64) = 12:4.2.5-58.el7.centos.1 for package: 12:dhcp-4.2.5-58.el7.centos.1.x86_64
--> Processing Dependency: dhcp-common = 12:4.2.5-58.el7.centos.1 for package: 12:dhcp-4.2.5-58.el7.centos.1.x86_64
--> Running transaction check
---> Package dhcp-common.x86_64 12:4.2.5-47.el7.centos will be updated
--> Processing Dependency: dhcp-common = 12:4.2.5-47.el7.centos for package: 12:dhclient-4.2.5-47.el7.centos.x86_64
---> Package dhcp-common.x86_64 12:4.2.5-58.el7.centos.1 will be an update
---> Package dhcp-libs.x86_64 12:4.2.5-47.el7.centos will be updated
---> Package dhcp-libs.x86_64 12:4.2.5-58.el7.centos.1 will be an update
--> Running transaction check
---> Package dhclient.x86_64 12:4.2.5-47.el7.centos will be updated
---> Package dhclient.x86_64 12:4.2.5-58.el7.centos.1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch        Version                        Repository    Size
================================================================================
Installing:
 dhcp             x86_64      12:4.2.5-58.el7.centos.1       updates      513 k
Updating for dependencies:
 dhclient         x86_64      12:4.2.5-58.el7.centos.1       updates      282 k
 dhcp-common      x86_64      12:4.2.5-58.el7.centos.1       updates      174 k
 dhcp-libs        x86_64      12:4.2.5-58.el7.centos.1       updates      130 k

Transaction Summary
================================================================================
Install  1 Package
Upgrade             ( 3 Dependent packages)

Total download size: 1.1 M
Is this ok [y/d/N]: y
Downloading packages:
Not downloading deltainfo for updates, MD is 954 k and rpms are 586 k
(1/4): dhclient-4.2.5-58.el7.centos.1.x86_64.rpm           | 282 kB   00:00     
(2/4): dhcp-libs-4.2.5-58.el7.centos.1.x86_64.rpm          | 130 kB   00:00     
(3/4): dhcp-common-4.2.5-58.el7.centos.1.x86_64.rpm        | 174 kB   00:00     
(4/4): dhcp-4.2.5-58.el7.centos.1.x86_64.rpm               | 513 kB   00:01     
--------------------------------------------------------------------------------
Total                                              982 kB/s | 1.1 MB  00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 12:dhcp-libs-4.2.5-58.el7.centos.1.x86_64                    1/7
  Updating   : 12:dhcp-common-4.2.5-58.el7.centos.1.x86_64                  2/7
  Updating   : 12:dhclient-4.2.5-58.el7.centos.1.x86_64                     3/7
  Installing : 12:dhcp-4.2.5-58.el7.centos.1.x86_64                         4/7
  Cleanup    : 12:dhclient-4.2.5-47.el7.centos.x86_64                       5/7
  Cleanup    : 12:dhcp-common-4.2.5-47.el7.centos.x86_64                    6/7
  Cleanup    : 12:dhcp-libs-4.2.5-47.el7.centos.x86_64                      7/7
  Verifying  : 12:dhcp-common-4.2.5-58.el7.centos.1.x86_64                  1/7
  Verifying  : 12:dhcp-libs-4.2.5-58.el7.centos.1.x86_64                    2/7
  Verifying  : 12:dhclient-4.2.5-58.el7.centos.1.x86_64                     3/7
  Verifying  : 12:dhcp-4.2.5-58.el7.centos.1.x86_64                         4/7
  Verifying  : 12:dhcp-libs-4.2.5-47.el7.centos.x86_64                      5/7
  Verifying  : 12:dhclient-4.2.5-47.el7.centos.x86_64                       6/7
  Verifying  : 12:dhcp-common-4.2.5-47.el7.centos.x86_64                    7/7

Installed:
  dhcp.x86_64 12:4.2.5-58.el7.centos.1                                          

Dependency Updated:
  dhclient.x86_64 12:4.2.5-58.el7.centos.1                                      
  dhcp-common.x86_64 12:4.2.5-58.el7.centos.1                                   
  dhcp-libs.x86_64 12:4.2.5-58.el7.centos.1                                     

Complete!
[root@56-201 cobbler]# rpm -qa | grep dhcp
dhcp-common-4.2.5-58.el7.centos.1.x86_64
dhcp-4.2.5-58.el7.centos.1.x86_64
dhcp-libs-4.2.5-58.el7.centos.1.x86_64
[root@56-201 cobbler]#
~~~


## 启动 cobbler 服务


~~~
[root@56-201 cobbler]# systemctl status cobblerd.service
● cobblerd.service - Cobbler Helper Daemon
   Loaded: loaded (/usr/lib/systemd/system/cobblerd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@56-201 cobbler]# systemctl start cobblerd.service
[root@56-201 cobbler]# systemctl status cobblerd.service
● cobblerd.service - Cobbler Helper Daemon
   Loaded: loaded (/usr/lib/systemd/system/cobblerd.service; disabled; vendor preset: disabled)
   Active: active (running) since 二 2018-03-13 00:57:46 CST; 3s ago
  Process: 4096 ExecStartPost=/usr/bin/touch /usr/share/cobbler/web/cobbler.wsgi (code=exited, status=1/FAILURE)
 Main PID: 4095 (cobblerd)
   CGroup: /system.slice/cobblerd.service
           └─4095 /usr/bin/python2 -s /usr/bin/cobblerd -F

3月 13 00:57:46 56-201 systemd[1]: Starting Cobbler Helper Daemon...
3月 13 00:57:46 56-201 touch[4096]: /usr/bin/touch: cannot touch ‘/usr/sha…tory
3月 13 00:57:46 56-201 systemd[1]: Started Cobbler Helper Daemon.
Hint: Some lines were ellipsized, use -l to show in full.
[root@56-201 cobbler]#
[root@56-201 cobbler]# ps faux | grep cobbler
root      4202  0.0  0.0 112648  1016 pts/0    S+   01:00   0:00          \_ grep --color=auto cobbler
root      4095  0.1  1.4 364992 29468 ?        Ss   00:57   0:00 /usr/bin/python2 -s /usr/bin/cobblerd -F
[root@56-201 cobbler]#
~~~


## 检查环境与配置

使用 **`cobbler check`** 来进行环境与配置检查

~~~
[root@56-201 cobbler]# cobbler check
httpd does not appear to be running and proxying cobbler, or SELinux is in the way. Original traceback:
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/cobbler/cli.py", line 251, in check_setup
    s.ping()
  File "/usr/lib64/python2.7/xmlrpclib.py", line 1233, in __call__
    return self.__send(self.__name, args)
  File "/usr/lib64/python2.7/xmlrpclib.py", line 1587, in __request
    verbose=self.__verbose
  File "/usr/lib64/python2.7/xmlrpclib.py", line 1273, in request
    return self.single_request(host, handler, request_body, verbose)
  File "/usr/lib64/python2.7/xmlrpclib.py", line 1301, in single_request
    self.send_content(h, request_body)
  File "/usr/lib64/python2.7/xmlrpclib.py", line 1448, in send_content
    connection.endheaders(request_body)
  File "/usr/lib64/python2.7/httplib.py", line 1013, in endheaders
    self._send_output(message_body)
  File "/usr/lib64/python2.7/httplib.py", line 864, in _send_output
    self.send(msg)
  File "/usr/lib64/python2.7/httplib.py", line 826, in send
    self.connect()
  File "/usr/lib64/python2.7/httplib.py", line 807, in connect
    self.timeout, self.source_address)
  File "/usr/lib64/python2.7/socket.py", line 571, in create_connection
    raise err
error: [Errno 111] Connection refused
[root@56-201 cobbler]#
[root@56-201 cobbler]# systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
[root@56-201 cobbler]# systemctl start httpd.service
[root@56-201 cobbler]# systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since 二 2018-03-13 01:07:07 CST; 3s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 4298 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─4298 /usr/sbin/httpd -DFOREGROUND
           ├─4301 /usr/sbin/httpd -DFOREGROUND
           ├─4302 /usr/sbin/httpd -DFOREGROUND
           ├─4303 /usr/sbin/httpd -DFOREGROUND
           ├─4304 /usr/sbin/httpd -DFOREGROUND
           ├─4305 /usr/sbin/httpd -DFOREGROUND
           └─4306 /usr/sbin/httpd -DFOREGROUND

3月 13 01:07:06 56-201 systemd[1]: Starting The Apache HTTP Server...
3月 13 01:07:07 56-201 httpd[4298]: AH00558: httpd: Could not reliably det...ge
3月 13 01:07:07 56-201 systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : change 'disable' to 'no' in /etc/xinetd.d/tftp
3 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
4 : enable and start rsyncd.service with systemctl
5 : debmirror package is not installed, it will be required to manage debian deployments and repositories
6 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]#
~~~

这里会有一些建议，我们根据这些进行调整，然后再检查，会发现已经修复过的问题，不会再次提醒

~~~
[root@56-201 cobbler]# vim /etc/xinetd.d/tftp
[root@56-201 cobbler]# grep disable /etc/xinetd.d/tftp
#	disable			= yes
	disable			= no
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
3 : enable and start rsyncd.service with systemctl
4 : debmirror package is not installed, it will be required to manage debian deployments and repositories
5 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]#
~~~


下面是我执行和完善这些建议的过程

~~~
[root@56-201 cobbler]# vim /etc/xinetd.d/tftp
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
3 : enable and start rsyncd.service with systemctl
4 : debmirror package is not installed, it will be required to manage debian deployments and repositories
5 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]# grep disable /etc/xinetd.d/tftp
#	disable			= yes
	disable			= no
[root@56-201 cobbler]# cobbler get-loaders
task started: 2018-03-13_012542_get_loaders
task started (id=Download Bootloader Content, time=Tue Mar 13 01:25:42 2018)
downloading https://cobbler.github.io/loaders/README to /var/lib/cobbler/loaders/README
downloading https://cobbler.github.io/loaders/COPYING.elilo to /var/lib/cobbler/loaders/COPYING.elilo
downloading https://cobbler.github.io/loaders/COPYING.yaboot to /var/lib/cobbler/loaders/COPYING.yaboot
downloading https://cobbler.github.io/loaders/COPYING.syslinux to /var/lib/cobbler/loaders/COPYING.syslinux
downloading https://cobbler.github.io/loaders/elilo-3.8-ia64.efi to /var/lib/cobbler/loaders/elilo-ia64.efi
downloading https://cobbler.github.io/loaders/yaboot-1.3.17 to /var/lib/cobbler/loaders/yaboot
downloading https://cobbler.github.io/loaders/pxelinux.0-3.86 to /var/lib/cobbler/loaders/pxelinux.0
downloading https://cobbler.github.io/loaders/menu.c32-3.86 to /var/lib/cobbler/loaders/menu.c32
downloading https://cobbler.github.io/loaders/grub-0.97-x86.efi to /var/lib/cobbler/loaders/grub-x86.efi
downloading https://cobbler.github.io/loaders/grub-0.97-x86_64.efi to /var/lib/cobbler/loaders/grub-x86_64.efi
*** TASK COMPLETE ***
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : enable and start rsyncd.service with systemctl
3 : debmirror package is not installed, it will be required to manage debian deployments and repositories
4 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]# yum list all | grep  debmirror
debmirror.noarch                        2.16-4.el7                     epel     
[root@56-201 cobbler]# yum install debmirror
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package debmirror.noarch 0:2.16-4.el7 will be installed
--> Processing Dependency: patch for package: debmirror-2.16-4.el7.noarch
--> Processing Dependency: perl(LockFile::Simple) for package: debmirror-2.16-4.el7.noarch
--> Running transaction check
---> Package patch.x86_64 0:2.7.1-8.el7 will be installed
---> Package perl-LockFile-Simple.noarch 0:0.208-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                     Arch          Version            Repository   Size
================================================================================
Installing:
 debmirror                   noarch        2.16-4.el7         epel         61 k
Installing for dependencies:
 patch                       x86_64        2.7.1-8.el7        base        110 k
 perl-LockFile-Simple        noarch        0.208-1.el7        epel         24 k

Transaction Summary
================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 195 k
Installed size: 410 k
Is this ok [y/d/N]: y
Downloading packages:
(1/3): patch-2.7.1-8.el7.x86_64.rpm                        | 110 kB   00:00     
(2/3): perl-LockFile-Simple-0.208-1.el7.noarch.rpm         |  24 kB   00:00     
(3/3): debmirror-2.16-4.el7.noarch.rpm                     |  61 kB   00:01     
--------------------------------------------------------------------------------
Total                                              104 kB/s | 195 kB  00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : patch-2.7.1-8.el7.x86_64                                     1/3
  Installing : perl-LockFile-Simple-0.208-1.el7.noarch                      2/3
  Installing : debmirror-2.16-4.el7.noarch                                  3/3
  Verifying  : debmirror-2.16-4.el7.noarch                                  1/3
  Verifying  : perl-LockFile-Simple-0.208-1.el7.noarch                      2/3
  Verifying  : patch-2.7.1-8.el7.x86_64                                     3/3

Installed:
  debmirror.noarch 0:2.16-4.el7                                                 

Dependency Installed:
  patch.x86_64 0:2.7.1-8.el7      perl-LockFile-Simple.noarch 0:0.208-1.el7     

Complete!
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : enable and start rsyncd.service with systemctl
3 : comment out 'dists' on /etc/debmirror.conf for proper debian support
4 : comment out 'arches' on /etc/debmirror.conf for proper debian support
5 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]# vim /etc/debmirror.conf
[root@56-201 cobbler]# grep  dists /etc/debmirror.conf
@dists="sid";
# @di_dists="dists";
[root@56-201 cobbler]# grep arches /etc/debmirror.conf
@arches="i386";
# @di_archs="arches";
[root@56-201 cobbler]#
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : enable and start rsyncd.service with systemctl
3 : comment out 'dists' on /etc/debmirror.conf for proper debian support
4 : comment out 'arches' on /etc/debmirror.conf for proper debian support
5 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]#
[root@56-201 cobbler]# vim /etc/debmirror.conf
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : enable and start rsyncd.service with systemctl
3 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]# grep 'dists' /etc/debmirror.conf
#@dists="sid";
# @di_dists="dists";
[root@56-201 cobbler]# grep arches /etc/debmirror.conf
#@arches="i386";
# @di_archs="arches";
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : enable and start rsyncd.service with systemctl
3 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]# yum install fence-agents
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package fence-agents-all.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: fence-virt for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-wti for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-vmware-soap for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-scsi for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-sbd for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-rsb for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-rsa for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-rhevm for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-mpath for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-kdump for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ipmilan for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ipdu for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-intelmodular for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ilo2 for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ilo-ssh for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ilo-mp for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ilo-moonshot for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ifmib for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-ibmblade for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-hpblade for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-eps for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-emerson for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-eaton-snmp for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-drac5 for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-compute for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-cisco-ucs for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-cisco-mds for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-brocade for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-bladecenter for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-apc-snmp for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: fence-agents-apc for package: fence-agents-all-4.0.11-66.el7_4.4.x86_64
--> Running transaction check
---> Package fence-agents-apc.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: fence-agents-common >= 4.0.11-66.el7_4.4 for package: fence-agents-apc-4.0.11-66.el7_4.4.x86_64
--> Processing Dependency: telnet for package: fence-agents-apc-4.0.11-66.el7_4.4.x86_64
---> Package fence-agents-apc-snmp.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: net-snmp-utils for package: fence-agents-apc-snmp-4.0.11-66.el7_4.4.x86_64
---> Package fence-agents-bladecenter.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-brocade.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-cisco-mds.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-cisco-ucs.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-compute.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-drac5.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-eaton-snmp.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-emerson.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-eps.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-hpblade.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ibmblade.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ifmib.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ilo-moonshot.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ilo-mp.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ilo-ssh.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ilo2.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-intelmodular.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ipdu.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-ipmilan.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: /usr/bin/ipmitool for package: fence-agents-ipmilan-4.0.11-66.el7_4.4.x86_64
---> Package fence-agents-kdump.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-mpath.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-rhevm.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-rsa.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-rsb.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-sbd.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-agents-scsi.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: sg3_utils for package: fence-agents-scsi-4.0.11-66.el7_4.4.x86_64
---> Package fence-agents-vmware-soap.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: python-suds for package: fence-agents-vmware-soap-4.0.11-66.el7_4.4.x86_64
---> Package fence-agents-wti.x86_64 0:4.0.11-66.el7_4.4 will be installed
---> Package fence-virt.x86_64 0:0.3.2-12.el7 will be installed
--> Running transaction check
---> Package fence-agents-common.x86_64 0:4.0.11-66.el7_4.4 will be installed
--> Processing Dependency: pexpect for package: fence-agents-common-4.0.11-66.el7_4.4.x86_64
---> Package ipmitool.x86_64 0:1.8.18-5.el7 will be installed
--> Processing Dependency: OpenIPMI-modalias for package: ipmitool-1.8.18-5.el7.x86_64
---> Package net-snmp-utils.x86_64 1:5.7.2-28.el7_4.1 will be installed
--> Processing Dependency: net-snmp-libs = 1:5.7.2-28.el7_4.1 for package: 1:net-snmp-utils-5.7.2-28.el7_4.1.x86_64
---> Package python-suds.noarch 0:0.4.1-5.el7 will be installed
---> Package sg3_utils.x86_64 0:1.37-12.el7 will be installed
--> Processing Dependency: sg3_utils-libs = 1.37-12.el7 for package: sg3_utils-1.37-12.el7.x86_64
---> Package telnet.x86_64 1:0.17-64.el7 will be installed
--> Running transaction check
---> Package OpenIPMI-modalias.x86_64 0:2.0.19-15.el7 will be installed
---> Package net-snmp-libs.x86_64 1:5.7.2-24.el7_3.2 will be updated
---> Package net-snmp-libs.x86_64 1:5.7.2-28.el7_4.1 will be an update
---> Package pexpect.noarch 0:2.3-11.el7 will be installed
---> Package sg3_utils-libs.x86_64 0:1.37-9.el7 will be updated
---> Package sg3_utils-libs.x86_64 0:1.37-12.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                      Arch      Version                Repository  Size
================================================================================
Installing:
 fence-agents-all             x86_64    4.0.11-66.el7_4.4      updates     18 k
Installing for dependencies:
 OpenIPMI-modalias            x86_64    2.0.19-15.el7          base        15 k
 fence-agents-apc             x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-apc-snmp        x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-bladecenter     x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-brocade         x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-cisco-mds       x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-cisco-ucs       x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-common          x86_64    4.0.11-66.el7_4.4      updates     65 k
 fence-agents-compute         x86_64    4.0.11-66.el7_4.4      updates     27 k
 fence-agents-drac5           x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-eaton-snmp      x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-emerson         x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-eps             x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-hpblade         x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-ibmblade        x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-ifmib           x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-ilo-moonshot    x86_64    4.0.11-66.el7_4.4      updates     20 k
 fence-agents-ilo-mp          x86_64    4.0.11-66.el7_4.4      updates     20 k
 fence-agents-ilo-ssh         x86_64    4.0.11-66.el7_4.4      updates     24 k
 fence-agents-ilo2            x86_64    4.0.11-66.el7_4.4      updates     23 k
 fence-agents-intelmodular    x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-ipdu            x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-ipmilan         x86_64    4.0.11-66.el7_4.4      updates     30 k
 fence-agents-kdump           x86_64    4.0.11-66.el7_4.4      updates     31 k
 fence-agents-mpath           x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-rhevm           x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-rsa             x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-rsb             x86_64    4.0.11-66.el7_4.4      updates     21 k
 fence-agents-sbd             x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-agents-scsi            x86_64    4.0.11-66.el7_4.4      updates     25 k
 fence-agents-vmware-soap     x86_64    4.0.11-66.el7_4.4      updates     23 k
 fence-agents-wti             x86_64    4.0.11-66.el7_4.4      updates     22 k
 fence-virt                   x86_64    0.3.2-12.el7           base        41 k
 ipmitool                     x86_64    1.8.18-5.el7           base       441 k
 net-snmp-utils               x86_64    1:5.7.2-28.el7_4.1     updates    198 k
 pexpect                      noarch    2.3-11.el7             base       142 k
 python-suds                  noarch    0.4.1-5.el7            base       204 k
 sg3_utils                    x86_64    1.37-12.el7            base       644 k
 telnet                       x86_64    1:0.17-64.el7          base        64 k
Updating for dependencies:
 net-snmp-libs                x86_64    1:5.7.2-28.el7_4.1     updates    748 k
 sg3_utils-libs               x86_64    1.37-12.el7            base        64 k

Transaction Summary
================================================================================
Install  1 Package  (+39 Dependent packages)
Upgrade             (  2 Dependent packages)

Total download size: 3.2 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for base
Not downloading deltainfo for updates, MD is 954 k and rpms are 748 k
(1/42): fence-agents-apc-4.0.11-66.el7_4.4.x86_64.rpm      |  22 kB   00:00     
(2/42): fence-agents-bladecenter-4.0.11-66.el7_4.4.x86_64. |  21 kB   00:00     
(3/42): OpenIPMI-modalias-2.0.19-15.el7.x86_64.rpm         |  15 kB   00:00     
(4/42): fence-agents-all-4.0.11-66.el7_4.4.x86_64.rpm      |  18 kB   00:00     
(5/42): fence-agents-apc-snmp-4.0.11-66.el7_4.4.x86_64.rpm |  22 kB   00:00     
(6/42): fence-agents-cisco-mds-4.0.11-66.el7_4.4.x86_64.rp |  21 kB   00:00     
(7/42): fence-agents-brocade-4.0.11-66.el7_4.4.x86_64.rpm  |  21 kB   00:00     
(8/42): fence-agents-cisco-ucs-4.0.11-66.el7_4.4.x86_64.rp |  22 kB   00:00     
(9/42): fence-agents-drac5-4.0.11-66.el7_4.4.x86_64.rpm    |  22 kB   00:00     
(10/42): fence-agents-emerson-4.0.11-66.el7_4.4.x86_64.rpm |  21 kB   00:00     
(11/42): fence-agents-compute-4.0.11-66.el7_4.4.x86_64.rpm |  27 kB   00:00     
(12/42): fence-agents-hpblade-4.0.11-66.el7_4.4.x86_64.rpm |  21 kB   00:00     
(13/42): fence-agents-eaton-snmp-4.0.11-66.el7_4.4.x86_64. |  22 kB   00:00     
(14/42): fence-agents-eps-4.0.11-66.el7_4.4.x86_64.rpm     |  21 kB   00:00     
(15/42): fence-agents-common-4.0.11-66.el7_4.4.x86_64.rpm  |  65 kB   00:00     
(16/42): fence-agents-ifmib-4.0.11-66.el7_4.4.x86_64.rpm   |  21 kB   00:00     
(17/42): fence-agents-ilo-mp-4.0.11-66.el7_4.4.x86_64.rpm  |  20 kB   00:00     
(18/42): fence-agents-ilo-ssh-4.0.11-66.el7_4.4.x86_64.rpm |  24 kB   00:00     
(19/42): fence-agents-ibmblade-4.0.11-66.el7_4.4.x86_64.rp |  21 kB   00:00     
(20/42): fence-agents-ipdu-4.0.11-66.el7_4.4.x86_64.rpm    |  21 kB   00:00     
(21/42): fence-agents-ilo2-4.0.11-66.el7_4.4.x86_64.rpm    |  23 kB   00:00     
(22/42): fence-agents-intelmodular-4.0.11-66.el7_4.4.x86_6 |  21 kB   00:00     
(23/42): fence-agents-mpath-4.0.11-66.el7_4.4.x86_64.rpm   |  22 kB   00:00     
(24/42): fence-agents-kdump-4.0.11-66.el7_4.4.x86_64.rpm   |  31 kB   00:00     
(25/42): fence-agents-ilo-moonshot-4.0.11-66.el7_4.4.x86_6 |  20 kB   00:00     
(26/42): fence-agents-ipmilan-4.0.11-66.el7_4.4.x86_64.rpm |  30 kB   00:00     
(27/42): fence-agents-rhevm-4.0.11-66.el7_4.4.x86_64.rpm   |  21 kB   00:00     
(28/42): fence-agents-rsb-4.0.11-66.el7_4.4.x86_64.rpm     |  21 kB   00:00     
(29/42): fence-agents-rsa-4.0.11-66.el7_4.4.x86_64.rpm     |  21 kB   00:00     
(30/42): fence-agents-sbd-4.0.11-66.el7_4.4.x86_64.rpm     |  22 kB   00:00     
(31/42): fence-agents-wti-4.0.11-66.el7_4.4.x86_64.rpm     |  22 kB   00:00     
(32/42): fence-agents-vmware-soap-4.0.11-66.el7_4.4.x86_64 |  23 kB   00:00     
(33/42): fence-virt-0.3.2-12.el7.x86_64.rpm                |  41 kB   00:00     
(34/42): fence-agents-scsi-4.0.11-66.el7_4.4.x86_64.rpm    |  25 kB   00:00     
(35/42): pexpect-2.3-11.el7.noarch.rpm                     | 142 kB   00:00     
(36/42): net-snmp-utils-5.7.2-28.el7_4.1.x86_64.rpm        | 198 kB   00:00     
(37/42): net-snmp-libs-5.7.2-28.el7_4.1.x86_64.rpm         | 748 kB   00:00     
(38/42): sg3_utils-1.37-12.el7.x86_64.rpm                  | 644 kB   00:00     
(39/42): telnet-0.17-64.el7.x86_64.rpm                     |  64 kB   00:00     
(40/42): python-suds-0.4.1-5.el7.noarch.rpm                | 204 kB   00:00     
(41/42): ipmitool-1.8.18-5.el7.x86_64.rpm                  | 441 kB   00:01     
(42/42): sg3_utils-libs-1.37-12.el7.x86_64.rpm             |  64 kB   00:01     
--------------------------------------------------------------------------------
Total                                              1.1 MB/s | 3.2 MB  00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:telnet-0.17-64.el7.x86_64                                 1/44
  Installing : OpenIPMI-modalias-2.0.19-15.el7.x86_64                      2/44
  Installing : ipmitool-1.8.18-5.el7.x86_64                                3/44
  Installing : fence-virt-0.3.2-12.el7.x86_64                              4/44
  Updating   : 1:net-snmp-libs-5.7.2-28.el7_4.1.x86_64                     5/44
  Installing : 1:net-snmp-utils-5.7.2-28.el7_4.1.x86_64                    6/44
  Updating   : sg3_utils-libs-1.37-12.el7.x86_64                           7/44
  Installing : sg3_utils-1.37-12.el7.x86_64                                8/44
  Installing : python-suds-0.4.1-5.el7.noarch                              9/44
  Installing : pexpect-2.3-11.el7.noarch                                  10/44
  Installing : fence-agents-common-4.0.11-66.el7_4.4.x86_64               11/44
  Installing : fence-agents-eps-4.0.11-66.el7_4.4.x86_64                  12/44
  Installing : fence-agents-rsb-4.0.11-66.el7_4.4.x86_64                  13/44
  Installing : fence-agents-ipmilan-4.0.11-66.el7_4.4.x86_64              14/44
  Installing : fence-agents-mpath-4.0.11-66.el7_4.4.x86_64                15/44
  Installing : fence-agents-sbd-4.0.11-66.el7_4.4.x86_64                  16/44
  Installing : fence-agents-cisco-ucs-4.0.11-66.el7_4.4.x86_64            17/44
  Installing : fence-agents-ibmblade-4.0.11-66.el7_4.4.x86_64             18/44
  Installing : fence-agents-ilo-mp-4.0.11-66.el7_4.4.x86_64               19/44
  Installing : fence-agents-eaton-snmp-4.0.11-66.el7_4.4.x86_64           20/44
  Installing : fence-agents-cisco-mds-4.0.11-66.el7_4.4.x86_64            21/44
  Installing : fence-agents-scsi-4.0.11-66.el7_4.4.x86_64                 22/44
  Installing : fence-agents-compute-4.0.11-66.el7_4.4.x86_64              23/44
  Installing : fence-agents-rsa-4.0.11-66.el7_4.4.x86_64                  24/44
  Installing : fence-agents-hpblade-4.0.11-66.el7_4.4.x86_64              25/44
  Installing : fence-agents-drac5-4.0.11-66.el7_4.4.x86_64                26/44
  Installing : fence-agents-wti-4.0.11-66.el7_4.4.x86_64                  27/44
  Installing : fence-agents-intelmodular-4.0.11-66.el7_4.4.x86_64         28/44
  Installing : fence-agents-ilo-ssh-4.0.11-66.el7_4.4.x86_64              29/44
  Installing : fence-agents-emerson-4.0.11-66.el7_4.4.x86_64              30/44
  Installing : fence-agents-apc-snmp-4.0.11-66.el7_4.4.x86_64             31/44
  Installing : fence-agents-ifmib-4.0.11-66.el7_4.4.x86_64                32/44
  Installing : fence-agents-apc-4.0.11-66.el7_4.4.x86_64                  33/44
  Installing : fence-agents-ilo2-4.0.11-66.el7_4.4.x86_64                 34/44
  Installing : fence-agents-rhevm-4.0.11-66.el7_4.4.x86_64                35/44
  Installing : fence-agents-kdump-4.0.11-66.el7_4.4.x86_64                36/44
  Installing : fence-agents-bladecenter-4.0.11-66.el7_4.4.x86_64          37/44
  Installing : fence-agents-vmware-soap-4.0.11-66.el7_4.4.x86_64          38/44
  Installing : fence-agents-ipdu-4.0.11-66.el7_4.4.x86_64                 39/44
  Installing : fence-agents-ilo-moonshot-4.0.11-66.el7_4.4.x86_64         40/44
  Installing : fence-agents-brocade-4.0.11-66.el7_4.4.x86_64              41/44
  Installing : fence-agents-all-4.0.11-66.el7_4.4.x86_64                  42/44
  Cleanup    : 1:net-snmp-libs-5.7.2-24.el7_3.2.x86_64                    43/44
  Cleanup    : sg3_utils-libs-1.37-9.el7.x86_64                           44/44
  Verifying  : pexpect-2.3-11.el7.noarch                                   1/44
  Verifying  : fence-agents-eps-4.0.11-66.el7_4.4.x86_64                   2/44
  Verifying  : fence-agents-rsb-4.0.11-66.el7_4.4.x86_64                   3/44
  Verifying  : fence-agents-ipmilan-4.0.11-66.el7_4.4.x86_64               4/44
  Verifying  : fence-agents-mpath-4.0.11-66.el7_4.4.x86_64                 5/44
  Verifying  : fence-agents-sbd-4.0.11-66.el7_4.4.x86_64                   6/44
  Verifying  : fence-agents-cisco-ucs-4.0.11-66.el7_4.4.x86_64             7/44
  Verifying  : fence-agents-ibmblade-4.0.11-66.el7_4.4.x86_64              8/44
  Verifying  : fence-agents-ilo-mp-4.0.11-66.el7_4.4.x86_64                9/44
  Verifying  : fence-agents-eaton-snmp-4.0.11-66.el7_4.4.x86_64           10/44
  Verifying  : fence-agents-cisco-mds-4.0.11-66.el7_4.4.x86_64            11/44
  Verifying  : fence-agents-scsi-4.0.11-66.el7_4.4.x86_64                 12/44
  Verifying  : ipmitool-1.8.18-5.el7.x86_64                               13/44
  Verifying  : fence-agents-compute-4.0.11-66.el7_4.4.x86_64              14/44
  Verifying  : fence-agents-rsa-4.0.11-66.el7_4.4.x86_64                  15/44
  Verifying  : python-suds-0.4.1-5.el7.noarch                             16/44
  Verifying  : fence-agents-hpblade-4.0.11-66.el7_4.4.x86_64              17/44
  Verifying  : sg3_utils-libs-1.37-12.el7.x86_64                          18/44
  Verifying  : fence-agents-drac5-4.0.11-66.el7_4.4.x86_64                19/44
  Verifying  : fence-agents-wti-4.0.11-66.el7_4.4.x86_64                  20/44
  Verifying  : 1:telnet-0.17-64.el7.x86_64                                21/44
  Verifying  : fence-agents-intelmodular-4.0.11-66.el7_4.4.x86_64         22/44
  Verifying  : fence-agents-ilo-ssh-4.0.11-66.el7_4.4.x86_64              23/44
  Verifying  : 1:net-snmp-utils-5.7.2-28.el7_4.1.x86_64                   24/44
  Verifying  : sg3_utils-1.37-12.el7.x86_64                               25/44
  Verifying  : fence-agents-emerson-4.0.11-66.el7_4.4.x86_64              26/44
  Verifying  : fence-agents-apc-snmp-4.0.11-66.el7_4.4.x86_64             27/44
  Verifying  : 1:net-snmp-libs-5.7.2-28.el7_4.1.x86_64                    28/44
  Verifying  : fence-agents-ifmib-4.0.11-66.el7_4.4.x86_64                29/44
  Verifying  : fence-agents-apc-4.0.11-66.el7_4.4.x86_64                  30/44
  Verifying  : fence-agents-ilo2-4.0.11-66.el7_4.4.x86_64                 31/44
  Verifying  : fence-agents-rhevm-4.0.11-66.el7_4.4.x86_64                32/44
  Verifying  : fence-agents-all-4.0.11-66.el7_4.4.x86_64                  33/44
  Verifying  : fence-agents-kdump-4.0.11-66.el7_4.4.x86_64                34/44
  Verifying  : fence-agents-bladecenter-4.0.11-66.el7_4.4.x86_64          35/44
  Verifying  : fence-agents-vmware-soap-4.0.11-66.el7_4.4.x86_64          36/44
  Verifying  : fence-virt-0.3.2-12.el7.x86_64                             37/44
  Verifying  : fence-agents-ipdu-4.0.11-66.el7_4.4.x86_64                 38/44
  Verifying  : fence-agents-ilo-moonshot-4.0.11-66.el7_4.4.x86_64         39/44
  Verifying  : fence-agents-brocade-4.0.11-66.el7_4.4.x86_64              40/44
  Verifying  : fence-agents-common-4.0.11-66.el7_4.4.x86_64               41/44
  Verifying  : OpenIPMI-modalias-2.0.19-15.el7.x86_64                     42/44
  Verifying  : 1:net-snmp-libs-5.7.2-24.el7_3.2.x86_64                    43/44
  Verifying  : sg3_utils-libs-1.37-9.el7.x86_64                           44/44

Installed:
  fence-agents-all.x86_64 0:4.0.11-66.el7_4.4                                   

Dependency Installed:
  OpenIPMI-modalias.x86_64 0:2.0.19-15.el7                                      
  fence-agents-apc.x86_64 0:4.0.11-66.el7_4.4                                   
  fence-agents-apc-snmp.x86_64 0:4.0.11-66.el7_4.4                              
  fence-agents-bladecenter.x86_64 0:4.0.11-66.el7_4.4                           
  fence-agents-brocade.x86_64 0:4.0.11-66.el7_4.4                               
  fence-agents-cisco-mds.x86_64 0:4.0.11-66.el7_4.4                             
  fence-agents-cisco-ucs.x86_64 0:4.0.11-66.el7_4.4                             
  fence-agents-common.x86_64 0:4.0.11-66.el7_4.4                                
  fence-agents-compute.x86_64 0:4.0.11-66.el7_4.4                               
  fence-agents-drac5.x86_64 0:4.0.11-66.el7_4.4                                 
  fence-agents-eaton-snmp.x86_64 0:4.0.11-66.el7_4.4                            
  fence-agents-emerson.x86_64 0:4.0.11-66.el7_4.4                               
  fence-agents-eps.x86_64 0:4.0.11-66.el7_4.4                                   
  fence-agents-hpblade.x86_64 0:4.0.11-66.el7_4.4                               
  fence-agents-ibmblade.x86_64 0:4.0.11-66.el7_4.4                              
  fence-agents-ifmib.x86_64 0:4.0.11-66.el7_4.4                                 
  fence-agents-ilo-moonshot.x86_64 0:4.0.11-66.el7_4.4                          
  fence-agents-ilo-mp.x86_64 0:4.0.11-66.el7_4.4                                
  fence-agents-ilo-ssh.x86_64 0:4.0.11-66.el7_4.4                               
  fence-agents-ilo2.x86_64 0:4.0.11-66.el7_4.4                                  
  fence-agents-intelmodular.x86_64 0:4.0.11-66.el7_4.4                          
  fence-agents-ipdu.x86_64 0:4.0.11-66.el7_4.4                                  
  fence-agents-ipmilan.x86_64 0:4.0.11-66.el7_4.4                               
  fence-agents-kdump.x86_64 0:4.0.11-66.el7_4.4                                 
  fence-agents-mpath.x86_64 0:4.0.11-66.el7_4.4                                 
  fence-agents-rhevm.x86_64 0:4.0.11-66.el7_4.4                                 
  fence-agents-rsa.x86_64 0:4.0.11-66.el7_4.4                                   
  fence-agents-rsb.x86_64 0:4.0.11-66.el7_4.4                                   
  fence-agents-sbd.x86_64 0:4.0.11-66.el7_4.4                                   
  fence-agents-scsi.x86_64 0:4.0.11-66.el7_4.4                                  
  fence-agents-vmware-soap.x86_64 0:4.0.11-66.el7_4.4                           
  fence-agents-wti.x86_64 0:4.0.11-66.el7_4.4                                   
  fence-virt.x86_64 0:0.3.2-12.el7                                              
  ipmitool.x86_64 0:1.8.18-5.el7                                                
  net-snmp-utils.x86_64 1:5.7.2-28.el7_4.1                                      
  pexpect.noarch 0:2.3-11.el7                                                   
  python-suds.noarch 0:0.4.1-5.el7                                              
  sg3_utils.x86_64 0:1.37-12.el7                                                
  telnet.x86_64 1:0.17-64.el7                                                   

Dependency Updated:
  net-snmp-libs.x86_64 1:5.7.2-28.el7_4.1  sg3_utils-libs.x86_64 0:1.37-12.el7

Complete!
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : enable and start rsyncd.service with systemctl

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]# systemctl enable rsyncd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/rsyncd.service to /usr/lib/systemd/system/rsyncd.service.
[root@56-201 cobbler]# systemctl start rsyncd.service
[root@56-201 cobbler]# systemctl status rsyncd.service
● rsyncd.service - fast remote file copy program daemon
   Loaded: loaded (/usr/lib/systemd/system/rsyncd.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-03-13 01:33:42 CST; 5s ago
 Main PID: 5022 (rsync)
   CGroup: /system.slice/rsyncd.service
           └─5022 /usr/bin/rsync --daemon --no-detach

3月 13 01:33:42 56-201 systemd[1]: Started fast remote file copy program d...n.
3月 13 01:33:42 56-201 systemd[1]: Starting fast remote file copy program .....
3月 13 01:33:42 56-201 rsyncd[5022]: rsyncd version 3.0.9 starting, listen...73
Hint: Some lines were ellipsized, use -l to show in full.
[root@56-201 cobbler]# cobbler check
The following are potential configuration items that you may want to fix:

1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux

Restart cobblerd and then run 'cobbler sync' to apply changes.
[root@56-201 cobbler]#
~~~


这些多是建议，服务类型的比如 **tftp** 和 **rsync** 是需要遵从的，否则 cobbler 可能会工作不正常，但是其它，比如 **debmirror** 和 **fence** 如果用不上，是可以不用理会的

**SELinux** 在重启 OS 后会自动满足条件，可以忽略掉


## 同步配置

~~~
[root@56-201 cobbler]# systemctl restart cobblerd.service
[root@56-201 cobbler]# cobbler sync
task started: 2018-03-13_014304_sync
task started (id=Sync, time=Tue Mar 13 01:43:04 2018)
running pre-sync triggers
cleaning trees
removing: /var/lib/tftpboot/pxelinux.cfg/default
removing: /var/lib/tftpboot/grub/images
removing: /var/lib/tftpboot/grub/grub-x86.efi
removing: /var/lib/tftpboot/grub/grub-x86_64.efi
removing: /var/lib/tftpboot/grub/efidefault
removing: /var/lib/tftpboot/s390x/profile_list
copying bootloaders
copying: /var/lib/cobbler/loaders/pxelinux.0 -> /var/lib/tftpboot/pxelinux.0
copying: /var/lib/cobbler/loaders/menu.c32 -> /var/lib/tftpboot/menu.c32
copying: /var/lib/cobbler/loaders/yaboot -> /var/lib/tftpboot/yaboot
copying: /usr/share/syslinux/memdisk -> /var/lib/tftpboot/memdisk
copying: /var/lib/cobbler/loaders/grub-x86.efi -> /var/lib/tftpboot/grub/grub-x86.efi
copying: /var/lib/cobbler/loaders/grub-x86_64.efi -> /var/lib/tftpboot/grub/grub-x86_64.efi
copying distros to tftpboot
copying images
generating PXE configuration files
generating PXE menu structure
rendering DHCP files
generating /etc/dhcp/dhcpd.conf
rendering TFTPD files
generating /etc/xinetd.d/tftp
cleaning link caches
running post-sync triggers
running python triggers from /var/lib/cobbler/triggers/sync/post/*
running python trigger cobbler.modules.sync_post_restart_services
running: dhcpd -t -q
received on stdout:
received on stderr:
running: service dhcpd restart
received on stdout:
received on stderr: Redirecting to /bin/systemctl restart  dhcpd.service

running shell triggers from /var/lib/cobbler/triggers/sync/post/*
running python triggers from /var/lib/cobbler/triggers/change/*
running python trigger cobbler.modules.scm_track
running shell triggers from /var/lib/cobbler/triggers/change/*
*** TASK COMPLETE ***
[root@56-201 cobbler]# echo $?
0
[root@56-201 cobbler]#
~~~

到此为止 cobbler 的安装就已经完成了


---

# 总结

虽然是典型的 rpm 包安装，也有现成的 yum 库，但是由于 cobbler 依赖关系很复杂，所以配置与调试还是需要一定的耐心与经验

最核心的其实是理解整个 PXE 引导过程中各个角色的交互顺序和作用，这样可以更容易把握这些零散的组件与关系

* TOC
{:toc}


---


[cobbler]:http://cobbler.github.io/
[cobbler_ins]:http://cobbler.github.io/manuals/quickstart/
