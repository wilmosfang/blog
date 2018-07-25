---
layout: post
title: "Install Linux Client of ShadowSocks"
author:  wilmosfang
date: 2018-03-23 17:52:42
image: '/assets/img/'
excerpt: '安装 SSR (ShadowsocksR) linux 客户端'
main-class: 'tools'
color: '#808080'
tags:
 - ssr
 - ss
 - tools
categories:
 - tools
twitter_text: 'Shadowsocks-Qt5 install simple process'
introduction: 'installation method of Shadowsocks-Qt5'
---



## 前言

**[ShadowsocksR][shadowsocksr]** 是一款开源的用来穿透 **GFW** 的软件

**[ShadowsocksR][shadowsocksr]** 是 **SS(Shadowsocks)** 的一个分支，在 **SS(Shadowsocks)** 的基础上又添加了一些干扰识别(进行伪装)的特性，使其更具有穿透力

它是 **C/S** 架构的，需要安装服务端软件，然后通过客户端与之连接

成功连接后，其后的通讯就可以进行加密与伪装来穿透 **GFW**，达到科学上网的效果

前面演示了如何搭建 **[ShadowsocksR][shadowsocksr]** 服务，这里演示一下如何安装 **[Shadowsocks-QT5][shadowsocks-qt5]** 客户端，让 Linux 可以连接 SSR 服务

参考 **[shadowsocks-qt5-wiki][shadowsocks-qt5-wiki]** 和 **[安装指南][install_ref]**

> **Tip:** 当前的版本为 **Shadowsocks 2.8.2**

>以下仅代表个人观点: 不允许自由了解外面的世界，对一些信息有意加以过滤与封锁，是否等价于给门窗糊上报纸，不让风景透过窗户，也不让自己成为外面人的风景，若干年后的人们再次回望这段历史的时候，会不会觉得愚昧和荒谬

>堤坝可以用来防洪，本有其积极的意义，但若借机建得太高，以至于阻断了自由交流，可能就成了思想的牢笼

>有一个人说过，可以永远欺骗一部分人或者暂时欺骗所有人，但却无法永远欺骗所有人

好在还有那么一小片镜子，可以透过一些微小的缝隙，反射出外面的光芒

(不见得外面的世界一定就代表着美丽，也可能充斥着混乱与丑恶，但关键是请让我们自己来作判断，让我们拥有这份自主选择的权利，而非直接剥夺)


---

# 操作


## 环境

~~~
[root@h210 yum.repos.d]# hostnamectl 
   Static hostname: h210
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: e691b5c929a6418cbb90be99cd4acfbb
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@h210 yum.repos.d]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:69:49:17 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 79129sec preferred_lft 79129sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:da:55:a7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feda:55a7/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h210 yum.repos.d]# 
~~~


## 配置仓库


可以下载客户端的 repo 

链接地址 **[COPR][ss_repo]**

其中 Centos7 版本的仓库为 **[librehat-shadowsocks][ss_centos7_client]**

~~~
[root@h210 yum.repos.d]# vim ss.repo 
[root@h210 yum.repos.d]# cat ss.repo 
[librehat-shadowsocks]
name=Copr repo for shadowsocks owned by librehat
baseurl=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/epel-7-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
[root@h210 yum.repos.d]# 
~~~


## 安装 shadowsocks-qt5

~~~
[root@h210 yum.repos.d]# yum clean all 
Loaded plugins: fastestmirror, langpacks
Cleaning repos: base c7-media epel extras librehat-shadowsocks pgdg10 updates
Cleaning up everything
Maybe you want: rm -rf /var/cache/yum, to also free up space taken by orphaned data from disabled or removed repos
Cleaning up list of fastest mirrors
[root@h210 yum.repos.d]# yum install shadowsocks-qt5
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 7.3 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
librehat-shadowsocks                                     | 3.5 kB     00:00     
pgdg10                                                   | 4.1 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/12): c7-media/group_gz                                  | 155 kB   00:00     
(2/12): c7-media/primary_db                                | 5.6 MB   00:00     
(3/12): epel/x86_64/group_gz                               | 266 kB   00:00     
(4/12): base/7/x86_64/group_gz                             | 156 kB   00:00     
(5/12): extras/7/x86_64/primary_db                         | 184 kB   00:00     
(6/12): librehat-shadowsocks/x86_64/primary_db             |  24 kB   00:01     
(7/12): pgdg10/7/x86_64/group_gz                           |  245 B   00:01     
(8/12): base/7/x86_64/primary_db                           | 5.7 MB   00:05     
(9/12): pgdg10/7/x86_64/primary_db                         | 160 kB   00:03     
(10/12): epel/x86_64/updateinfo                            | 904 kB   00:08     
(11/12): updates/7/x86_64/primary_db                       | 6.9 MB   00:09     
(12/12): epel/x86_64/primary_db                            | 6.3 MB   00:35     
Determining fastest mirrors
 * base: centos-hcm.viettelidc.com.vn
 * c7-media: 
 * epel: mirror.ehost.vn
 * extras: mirror.nus.edu.sg
 * updates: mirror.nus.edu.sg
Resolving Dependencies
--> Running transaction check
---> Package shadowsocks-qt5.x86_64 0:2.9.0-2.el7.centos will be installed
--> Processing Dependency: libQtShadowsocks >= 1.10.0 for package: shadowsocks-qt5-2.9.0-2.el7.centos.x86_64
--> Processing Dependency: qt5-qtbase for package: shadowsocks-qt5-2.9.0-2.el7.centos.x86_64
--> Processing Dependency: qt5-qtbase-gui for package: shadowsocks-qt5-2.9.0-2.el7.centos.x86_64
--> Processing Dependency: botan for package: shadowsocks-qt5-2.9.0-2.el7.centos.x86_64
--> Processing Dependency: zbar for package: shadowsocks-qt5-2.9.0-2.el7.centos.x86_64
--> Processing Dependency: libappindicator for package: shadowsocks-qt5-2.9.0-2.el7.centos.x86_64
--> Running transaction check
---> Package botan.x86_64 0:1.10.17-1.el7 will be installed
---> Package libQtShadowsocks.x86_64 0:1.11.0-2.el7.centos will be installed
---> Package libappindicator.x86_64 0:12.10.0-11.el7 will be installed
--> Processing Dependency: libdbusmenu-glib.so.4()(64bit) for package: libappindicator-12.10.0-11.el7.x86_64
--> Processing Dependency: libdbusmenu-gtk.so.4()(64bit) for package: libappindicator-12.10.0-11.el7.x86_64
--> Processing Dependency: libindicator.so.7()(64bit) for package: libappindicator-12.10.0-11.el7.x86_64
---> Package qt5-qtbase.x86_64 0:5.6.2-1.el7 will be installed
--> Processing Dependency: qt5-qtbase-common = 5.6.2-1.el7 for package: qt5-qtbase-5.6.2-1.el7.x86_64
--> Processing Dependency: libcrypto.so.10(OPENSSL_1.0.2)(64bit) for package: qt5-qtbase-5.6.2-1.el7.x86_64
---> Package qt5-qtbase-gui.x86_64 0:5.6.2-1.el7 will be installed
--> Processing Dependency: libxcb-render-util.so.0()(64bit) for package: qt5-qtbase-gui-5.6.2-1.el7.x86_64
--> Processing Dependency: libxcb-keysyms.so.1()(64bit) for package: qt5-qtbase-gui-5.6.2-1.el7.x86_64
--> Processing Dependency: libxcb-image.so.0()(64bit) for package: qt5-qtbase-gui-5.6.2-1.el7.x86_64
--> Processing Dependency: libxcb-icccm.so.4()(64bit) for package: qt5-qtbase-gui-5.6.2-1.el7.x86_64
---> Package zbar.x86_64 0:0.10-27.el7 will be installed
--> Processing Dependency: libGraphicsMagick-Q16.so.3()(64bit) for package: zbar-0.10-27.el7.x86_64
--> Processing Dependency: libGraphicsMagickWand-Q16.so.2()(64bit) for package: zbar-0.10-27.el7.x86_64
--> Running transaction check
---> Package GraphicsMagick.x86_64 0:1.3.28-1.el7 will be installed
--> Processing Dependency: libwmflite-0.2.so.7()(64bit) for package: GraphicsMagick-1.3.28-1.el7.x86_64
---> Package libdbusmenu.x86_64 0:16.04.0-2.el7 will be installed
---> Package libdbusmenu-gtk2.x86_64 0:16.04.0-2.el7 will be installed
---> Package libindicator.x86_64 0:12.10.1-5.el7 will be installed
---> Package openssl-libs.x86_64 1:1.0.1e-60.el7_3.1 will be updated
--> Processing Dependency: openssl-libs(x86-64) = 1:1.0.1e-60.el7_3.1 for package: 1:openssl-1.0.1e-60.el7_3.1.x86_64
---> Package openssl-libs.x86_64 1:1.0.2k-8.el7 will be an update
---> Package qt5-qtbase-common.noarch 0:5.6.2-1.el7 will be installed
---> Package xcb-util-image.x86_64 0:0.4.0-2.el7 will be installed
---> Package xcb-util-keysyms.x86_64 0:0.4.0-1.el7 will be installed
---> Package xcb-util-renderutil.x86_64 0:0.3.9-3.el7 will be installed
---> Package xcb-util-wm.x86_64 0:0.4.1-5.el7 will be installed
--> Running transaction check
---> Package libwmf-lite.x86_64 0:0.2.8.4-41.el7_1 will be installed
---> Package openssl.x86_64 1:1.0.1e-60.el7_3.1 will be updated
---> Package openssl.x86_64 1:1.0.2k-8.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package              Arch    Version               Repository             Size
================================================================================
Installing:
 shadowsocks-qt5      x86_64  2.9.0-2.el7.centos    librehat-shadowsocks  289 k
Installing for dependencies:
 GraphicsMagick       x86_64  1.3.28-1.el7          epel                  1.4 M
 botan                x86_64  1.10.17-1.el7         epel                  909 k
 libQtShadowsocks     x86_64  1.11.0-2.el7.centos   librehat-shadowsocks   74 k
 libappindicator      x86_64  12.10.0-11.el7        epel                   36 k
 libdbusmenu          x86_64  16.04.0-2.el7         epel                  132 k
 libdbusmenu-gtk2     x86_64  16.04.0-2.el7         epel                   34 k
 libindicator         x86_64  12.10.1-5.el7         epel                   63 k
 libwmf-lite          x86_64  0.2.8.4-41.el7_1      base                   66 k
 qt5-qtbase           x86_64  5.6.2-1.el7           base                  2.9 M
 qt5-qtbase-common    noarch  5.6.2-1.el7           base                   25 k
 qt5-qtbase-gui       x86_64  5.6.2-1.el7           base                  5.4 M
 xcb-util-image       x86_64  0.4.0-2.el7           base                   15 k
 xcb-util-keysyms     x86_64  0.4.0-1.el7           base                   10 k
 xcb-util-renderutil  x86_64  0.3.9-3.el7           base                   12 k
 xcb-util-wm          x86_64  0.4.1-5.el7           base                   25 k
 zbar                 x86_64  0.10-27.el7           epel                  146 k
Updating for dependencies:
 openssl              x86_64  1:1.0.2k-8.el7        base                  492 k
 openssl-libs         x86_64  1:1.0.2k-8.el7        base                  1.2 M

Transaction Summary
================================================================================
Install  1 Package  (+16 Dependent packages)
Upgrade             (  2 Dependent packages)

Total download size: 13 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for base
warning: /var/cache/yum/x86_64/7/epel/packages/libdbusmenu-16.04.0-2.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for libdbusmenu-16.04.0-2.el7.x86_64.rpm is not installed
(1/19): libdbusmenu-16.04.0-2.el7.x86_64.rpm               | 132 kB   00:01     
(2/19): GraphicsMagick-1.3.28-1.el7.x86_64.rpm             | 1.4 MB   00:01     
(3/19): libdbusmenu-gtk2-16.04.0-2.el7.x86_64.rpm          |  34 kB   00:00     
(4/19): libindicator-12.10.1-5.el7.x86_64.rpm              |  63 kB   00:00     
(5/19): libwmf-lite-0.2.8.4-41.el7_1.x86_64.rpm            |  66 kB   00:00     
(6/19): botan-1.10.17-1.el7.x86_64.rpm                     | 909 kB   00:01     
warning: /var/cache/yum/x86_64/7/librehat-shadowsocks/packages/libQtShadowsocks-1.11.0-2.el7.centos.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID 753d8f08: NOKEY
Public key for libQtShadowsocks-1.11.0-2.el7.centos.x86_64.rpm is not installed
(7/19): libQtShadowsocks-1.11.0-2.el7.centos.x86_64.rpm    |  74 kB   00:02     
(8/19): qt5-qtbase-common-5.6.2-1.el7.noarch.rpm           |  25 kB   00:00     
(9/19): openssl-1.0.2k-8.el7.x86_64.rpm                    | 492 kB   00:01     
(10/19): libappindicator-12.10.0-11.el7.x86_64.rpm         |  36 kB   00:03     
(11/19): xcb-util-image-0.4.0-2.el7.x86_64.rpm             |  15 kB   00:00     
(12/19): xcb-util-keysyms-0.4.0-1.el7.x86_64.rpm           |  10 kB   00:00     
(13/19): xcb-util-renderutil-0.3.9-3.el7.x86_64.rpm        |  12 kB   00:00     
(14/19): xcb-util-wm-0.4.1-5.el7.x86_64.rpm                |  25 kB   00:00     
(15/19): openssl-libs-1.0.2k-8.el7.x86_64.rpm              | 1.2 MB   00:02     
(16/19): zbar-0.10-27.el7.x86_64.rpm                       | 146 kB   00:00     
(17/19): shadowsocks-qt5-2.9.0-2.el7.centos.x86_64.rpm     | 289 kB   00:02     
(18/19): qt5-qtbase-5.6.2-1.el7.x86_64.rpm                 | 2.9 MB   00:04     
(19/19): qt5-qtbase-gui-5.6.2-1.el7.x86_64.rpm             | 5.4 MB   00:15     
--------------------------------------------------------------------------------
Total                                              726 kB/s |  13 MB  00:18     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Is this ok [y/N]: y
Retrieving key from https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/pubkey.gpg
Importing GPG key 0x753D8F08:
 Userid     : "librehat_shadowsocks (None) <librehat#shadowsocks@copr.fedorahosted.org>"
 Fingerprint: 260e 3142 3f6f dcfd e43b 1f86 f876 b86a 753d 8f08
 From       : https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/pubkey.gpg
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:openssl-libs-1.0.2k-8.el7.x86_64                          1/21 
  Installing : qt5-qtbase-common-5.6.2-1.el7.noarch                        2/21 
  Installing : qt5-qtbase-5.6.2-1.el7.x86_64                               3/21 
  Installing : botan-1.10.17-1.el7.x86_64                                  4/21 
  Installing : libdbusmenu-16.04.0-2.el7.x86_64                            5/21 
  Installing : libdbusmenu-gtk2-16.04.0-2.el7.x86_64                       6/21 
  Installing : libQtShadowsocks-1.11.0-2.el7.centos.x86_64                 7/21 
  Installing : xcb-util-keysyms-0.4.0-1.el7.x86_64                         8/21 
  Installing : libwmf-lite-0.2.8.4-41.el7_1.x86_64                         9/21 
  Installing : GraphicsMagick-1.3.28-1.el7.x86_64                         10/21 
  Installing : zbar-0.10-27.el7.x86_64                                    11/21 
  Installing : libindicator-12.10.1-5.el7.x86_64                          12/21 
  Installing : libappindicator-12.10.0-11.el7.x86_64                      13/21 
  Installing : xcb-util-renderutil-0.3.9-3.el7.x86_64                     14/21 
  Installing : xcb-util-image-0.4.0-2.el7.x86_64                          15/21 
  Installing : xcb-util-wm-0.4.1-5.el7.x86_64                             16/21 
  Installing : qt5-qtbase-gui-5.6.2-1.el7.x86_64                          17/21 
  Installing : shadowsocks-qt5-2.9.0-2.el7.centos.x86_64                  18/21 
  Updating   : 1:openssl-1.0.2k-8.el7.x86_64                              19/21 
  Cleanup    : 1:openssl-1.0.1e-60.el7_3.1.x86_64                         20/21 
  Cleanup    : 1:openssl-libs-1.0.1e-60.el7_3.1.x86_64                    21/21 
  Verifying  : xcb-util-wm-0.4.1-5.el7.x86_64                              1/21 
  Verifying  : xcb-util-image-0.4.0-2.el7.x86_64                           2/21 
  Verifying  : xcb-util-renderutil-0.3.9-3.el7.x86_64                      3/21 
  Verifying  : libdbusmenu-gtk2-16.04.0-2.el7.x86_64                       4/21 
  Verifying  : libindicator-12.10.1-5.el7.x86_64                           5/21 
  Verifying  : libwmf-lite-0.2.8.4-41.el7_1.x86_64                         6/21 
  Verifying  : libQtShadowsocks-1.11.0-2.el7.centos.x86_64                 7/21 
  Verifying  : botan-1.10.17-1.el7.x86_64                                  8/21 
  Verifying  : qt5-qtbase-5.6.2-1.el7.x86_64                               9/21 
  Verifying  : libdbusmenu-16.04.0-2.el7.x86_64                           10/21 
  Verifying  : 1:openssl-libs-1.0.2k-8.el7.x86_64                         11/21 
  Verifying  : libappindicator-12.10.0-11.el7.x86_64                      12/21 
  Verifying  : GraphicsMagick-1.3.28-1.el7.x86_64                         13/21 
  Verifying  : xcb-util-keysyms-0.4.0-1.el7.x86_64                        14/21 
  Verifying  : shadowsocks-qt5-2.9.0-2.el7.centos.x86_64                  15/21 
  Verifying  : 1:openssl-1.0.2k-8.el7.x86_64                              16/21 
  Verifying  : qt5-qtbase-common-5.6.2-1.el7.noarch                       17/21 
  Verifying  : qt5-qtbase-gui-5.6.2-1.el7.x86_64                          18/21 
  Verifying  : zbar-0.10-27.el7.x86_64                                    19/21 
  Verifying  : 1:openssl-1.0.1e-60.el7_3.1.x86_64                         20/21 
  Verifying  : 1:openssl-libs-1.0.1e-60.el7_3.1.x86_64                    21/21 

Installed:
  shadowsocks-qt5.x86_64 0:2.9.0-2.el7.centos                                   

Dependency Installed:
  GraphicsMagick.x86_64 0:1.3.28-1.el7                                          
  botan.x86_64 0:1.10.17-1.el7                                                  
  libQtShadowsocks.x86_64 0:1.11.0-2.el7.centos                                 
  libappindicator.x86_64 0:12.10.0-11.el7                                       
  libdbusmenu.x86_64 0:16.04.0-2.el7                                            
  libdbusmenu-gtk2.x86_64 0:16.04.0-2.el7                                       
  libindicator.x86_64 0:12.10.1-5.el7                                           
  libwmf-lite.x86_64 0:0.2.8.4-41.el7_1                                         
  qt5-qtbase.x86_64 0:5.6.2-1.el7                                               
  qt5-qtbase-common.noarch 0:5.6.2-1.el7                                        
  qt5-qtbase-gui.x86_64 0:5.6.2-1.el7                                           
  xcb-util-image.x86_64 0:0.4.0-2.el7                                           
  xcb-util-keysyms.x86_64 0:0.4.0-1.el7                                         
  xcb-util-renderutil.x86_64 0:0.3.9-3.el7                                      
  xcb-util-wm.x86_64 0:0.4.1-5.el7                                              
  zbar.x86_64 0:0.10-27.el7                                                     

Dependency Updated:
  openssl.x86_64 1:1.0.2k-8.el7        openssl-libs.x86_64 1:1.0.2k-8.el7       

Complete!
[root@h210 yum.repos.d]# echo $?
0
[root@h210 yum.repos.d]# 
~~~


## 进行连接


通过 **`startx`** 进入图形界面(如果本来就在图形界面就不用管)

通过 **`ss-qt5`** 可以弹出客户端界面

~~~
ss-qt5
~~~

![ssr](/assets/img/ssr/ssr01.png)


在 **[Connection]** 中新建连接

![ssr](/assets/img/ssr/ssr02.png)

构建完成后，就多出了一个连接

![ssr](/assets/img/ssr/ssr03.png)

选中连接

![ssr](/assets/img/ssr/ssr04.png)

![ssr](/assets/img/ssr/ssr05.png)

进行连接后

可以看到本地打开了一个 1080 的回环监听

~~~
[root@h210 yum.repos.d]# netstat  -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:1080          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.210:22       192.168.56.1:41910      ESTABLISHED
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::6000                 :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
[root@h210 yum.repos.d]# 
~~~

测试延时

![ssr](/assets/img/ssr/ssr06.png)

如果以 **`-vv`** 模式，可以在服务端看到如下的 DEBUG 和 VERBOSE 信息

~~~
[root@ci ~]# ssserver -p 22 -k just_for_test -m aes-256-cfb -vv
2018-03-24 02:36:08 INFO     loading libcrypto from libcrypto.so.10
2018-03-24 02:36:08 INFO     starting server at 0.0.0.0:22
2018-03-24 02:36:08 DEBUG    using event model: epoll
2018-03-24 02:36:37 VERBOSE  fd 3 POLL_IN
2018-03-24 02:36:37 DEBUG    accept
2018-03-24 02:36:37 VERBOSE  fd 9 POLL_IN
2018-03-24 02:36:37 INFO     connecting www.google.com:80 from 119.94.94.14:44710
2018-03-24 02:36:37 DEBUG    resolving www.google.com with type 1 using server 10.243.28.52
2018-03-24 02:36:37 DEBUG    resolving www.google.com with type 1 using server 10.225.30.178
2018-03-24 02:36:37 VERBOSE  fd 10 POLL_OUT
2018-03-24 02:36:37 VERBOSE  fd 10 POLL_IN
2018-03-24 02:36:37 VERBOSE  fd 9 POLL_IN
2018-03-24 02:36:37 DEBUG    destroy: www.google.com:80
2018-03-24 02:36:37 DEBUG    destroying remote
2018-03-24 02:36:37 DEBUG    destroying local
2018-03-24 02:36:47 VERBOSE  sweeping timeouts
2018-03-24 02:36:57 VERBOSE  sweeping timeouts
2018-03-24 02:37:07 VERBOSE  sweeping timeouts
...
...
...
~~~

到此 ssr 客户端已经与服务端进行了连接

其它的功能，后续再进行探索


---

# 总结

总体来讲，配置客户端与服务端的连接就是将在服务端配置的参数再填一次到客户端中

SSR 的加密是对称加密


* TOC
{:toc}


---


[shadowsocksr]:https://github.com/shadowsocksr-backup/shadowsocksr
[shadowsocks-qt5]:https://github.com/shadowsocks/shadowsocks-qt5
[shadowsocks-qt5-wiki]:https://github.com/shadowsocks/shadowsocks-qt5/wiki
[install_ref]:https://github.com/shadowsocks/shadowsocks-qt5/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97
[ss_repo]:https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/
[ss_centos7_client]:https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo



