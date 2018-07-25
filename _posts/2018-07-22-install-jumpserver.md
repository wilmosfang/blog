---
layout: post
title:  "Install Jumpserver"
author:  wilmosfang
date:  2018-07-22 10:37:26
image:  '/assets/img/'
excerpt:  'Jumpserver 的安装方法'
main-class:  'tools'
color: 
tags: 
  - tools
  - jumpserver
categories:
  - tools
twitter_text: 'Simple process of Jumpserver Installation'
introduction: 'Installation of Jumpserver'
---

# 前言 #

**[Jumpserver][jumpserver]** 是一款使用广泛的开源堡垒机软件

> Jumpserver 是完全开源的堡垒机，使用 GNU GPL v2.0 开源协议，符合 4A 的运维审计系统

Jumpserver 基于 Python / Django 进行开发，遵循 Web 2.0 规范，配备了 Web Terminal 解决方案

Jumpserver 采纳分布式架构，支持多机房跨区域部署，中心节点提供 API，各机房部署登录节点，可横向扩展、无并发访问限制

这里就 **[Jumpserver][jumpserver]** 的安装作一个简单的演示

参考 [一步一步安装(CentOS)][jumpserver_doc]

> **Tip:** 当前的最新版本为 **jumpserver 1.3.3**

-------------------------------------------------------------------------------

# 操作 #

## 系统环境 ##

~~~
[root@h165 ~]# hostnamectl 
   Static hostname: h165
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 0b945835fbb54426b6f67a179adc93cf
           Boot ID: 3605e751c5cb495ea414ef44dec6526f
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[root@h165 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 82235sec preferred_lft 82235sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:55:8b:d3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.165/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe55:8bd3/64 scope link 
       valid_lft forever preferred_lft forever
[root@h165 ~]# 
~~~

## 关闭 Selinux ##

~~~
[root@h165 ~]# getenforce 
Enforcing
[root@h165 ~]# setenforce 0
[root@h165 ~]# getenforce 
Permissive
[root@h165 ~]# 
~~~


>**Note:** 如果 SELINUX 不关闭会无法访问 web 界面


## 关闭防火墙 ##

~~~
[root@h165 ~]# systemctl stop firewalld.service
[root@h165 ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
[root@h165 ~]# 
~~~

## 修改字符集 ##

~~~
[root@h165 ~]# localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8
[root@h165 ~]# export LC_ALL=zh_CN.UTF-8
-bash: warning: setlocale: LC_ALL: cannot change locale (zh_CN.UTF-8): No such file or directory
-bash: warning: setlocale: LC_ALL: cannot change locale (zh_CN.UTF-8)
[root@h165 ~]# echo 'LANG="zh_CN.UTF-8"' > /etc/locale.conf
[root@h165 ~]# env | grep LC
LC_ALL=zh_CN.UTF-8
[root@h165 ~]# 
~~~

## 安装依赖包 ##

~~~
[root@h165 tmp]# yum -y install wget sqlite-devel xz gcc automake zlib-devel openssl-devel epel-release git
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Package xz-5.2.2-1.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package automake.noarch 0:1.13.4-3.el7 will be installed
--> Processing Dependency: perl >= 5.006 for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: autoconf >= 2.65 for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(warnings) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(vars) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(threads) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(strict) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(constant) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(Thread::Queue) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(TAP::Parser) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(POSIX) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(IO::File) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(Getopt::Long) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(File::stat) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(File::Spec) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(File::Path) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(File::Copy) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(File::Compare) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(File::Basename) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(Exporter) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(Errno) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(DynaLoader) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(Class::Struct) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: perl(Carp) for package: automake-1.13.4-3.el7.noarch
--> Processing Dependency: /usr/bin/perl for package: automake-1.13.4-3.el7.noarch
---> Package epel-release.noarch 0:7-11 will be installed
---> Package gcc.x86_64 0:4.8.5-28.el7_5.1 will be installed
--> Processing Dependency: libgomp = 4.8.5-28.el7_5.1 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: cpp = 4.8.5-28.el7_5.1 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: libgcc >= 4.8.5-28.el7_5.1 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: glibc-devel >= 2.2.90-12 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: libmpfr.so.4()(64bit) for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-4.8.5-28.el7_5.1.x86_64
---> Package git.x86_64 0:1.8.3.1-14.el7_5 will be installed
--> Processing Dependency: perl-Git = 1.8.3.1-14.el7_5 for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(Term::ReadKey) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(Git) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(File::Temp) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(Error) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: libgnome-keyring.so.0()(64bit) for package: git-1.8.3.1-14.el7_5.x86_64
---> Package openssl-devel.x86_64 1:1.0.2k-12.el7 will be installed
--> Processing Dependency: krb5-devel(x86-64) for package: 1:openssl-devel-1.0.2k-12.el7.x86_64
---> Package sqlite-devel.x86_64 0:3.7.17-8.el7 will be installed
---> Package wget.x86_64 0:1.14-15.el7_4.1 will be installed
---> Package zlib-devel.x86_64 0:1.2.7-17.el7 will be installed
--> Running transaction check
---> Package autoconf.noarch 0:2.69-11.el7 will be installed
--> Processing Dependency: m4 >= 1.4.14 for package: autoconf-2.69-11.el7.noarch
--> Processing Dependency: perl(Text::ParseWords) for package: autoconf-2.69-11.el7.noarch
--> Processing Dependency: perl(Data::Dumper) for package: autoconf-2.69-11.el7.noarch
---> Package cpp.x86_64 0:4.8.5-28.el7_5.1 will be installed
---> Package glibc-devel.x86_64 0:2.17-222.el7 will be installed
--> Processing Dependency: glibc-headers = 2.17-222.el7 for package: glibc-devel-2.17-222.el7.x86_64
--> Processing Dependency: glibc-headers for package: glibc-devel-2.17-222.el7.x86_64
---> Package krb5-devel.x86_64 0:1.15.1-19.el7 will be installed
--> Processing Dependency: libkadm5(x86-64) = 1.15.1-19.el7 for package: krb5-devel-1.15.1-19.el7.x86_64
--> Processing Dependency: libverto-devel for package: krb5-devel-1.15.1-19.el7.x86_64
--> Processing Dependency: libselinux-devel for package: krb5-devel-1.15.1-19.el7.x86_64
--> Processing Dependency: libcom_err-devel for package: krb5-devel-1.15.1-19.el7.x86_64
--> Processing Dependency: keyutils-libs-devel for package: krb5-devel-1.15.1-19.el7.x86_64
---> Package libgcc.x86_64 0:4.8.5-28.el7 will be updated
---> Package libgcc.x86_64 0:4.8.5-28.el7_5.1 will be an update
---> Package libgnome-keyring.x86_64 0:3.12.0-1.el7 will be installed
---> Package libgomp.x86_64 0:4.8.5-28.el7 will be updated
---> Package libgomp.x86_64 0:4.8.5-28.el7_5.1 will be an update
---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
---> Package mpfr.x86_64 0:3.1.1-4.el7 will be installed
---> Package perl.x86_64 4:5.16.3-292.el7 will be installed
--> Processing Dependency: perl-libs = 4:5.16.3-292.el7 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl-macros for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl-libs for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: libperl.so()(64bit) for package: 4:perl-5.16.3-292.el7.x86_64
---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
---> Package perl-Error.noarch 1:0.17020-2.el7 will be installed
---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
--> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
---> Package perl-Git.noarch 0:1.8.3.1-14.el7_5 will be installed
---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
---> Package perl-TermReadKey.x86_64 0:2.30-20.el7 will be installed
---> Package perl-Test-Harness.noarch 0:3.28-3.el7 will be installed
---> Package perl-Thread-Queue.noarch 0:3.02-2.el7 will be installed
---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
--> Running transaction check
---> Package glibc-headers.x86_64 0:2.17-222.el7 will be installed
--> Processing Dependency: kernel-headers >= 2.2.1 for package: glibc-headers-2.17-222.el7.x86_64
--> Processing Dependency: kernel-headers for package: glibc-headers-2.17-222.el7.x86_64
---> Package keyutils-libs-devel.x86_64 0:1.5.8-3.el7 will be installed
---> Package libcom_err-devel.x86_64 0:1.42.9-12.el7_5 will be installed
--> Processing Dependency: libcom_err(x86-64) = 1.42.9-12.el7_5 for package: libcom_err-devel-1.42.9-12.el7_5.x86_64
---> Package libkadm5.x86_64 0:1.15.1-19.el7 will be installed
---> Package libselinux-devel.x86_64 0:2.5-12.el7 will be installed
--> Processing Dependency: libsepol-devel(x86-64) >= 2.5-6 for package: libselinux-devel-2.5-12.el7.x86_64
--> Processing Dependency: pkgconfig(libsepol) for package: libselinux-devel-2.5-12.el7.x86_64
--> Processing Dependency: pkgconfig(libpcre) for package: libselinux-devel-2.5-12.el7.x86_64
---> Package libverto-devel.x86_64 0:0.2.5-4.el7 will be installed
---> Package m4.x86_64 0:1.4.16-10.el7 will be installed
---> Package perl-Data-Dumper.x86_64 0:2.145-3.el7 will be installed
---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
--> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
--> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
--> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
--> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
---> Package perl-Socket.x86_64 0:2.010-4.el7 will be installed
---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
---> Package perl-libs.x86_64 4:5.16.3-292.el7 will be installed
---> Package perl-macros.x86_64 4:5.16.3-292.el7 will be installed
---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
--> Running transaction check
---> Package kernel-headers.x86_64 0:3.10.0-862.9.1.el7 will be installed
---> Package libcom_err.x86_64 0:1.42.9-11.el7 will be updated
--> Processing Dependency: libcom_err(x86-64) = 1.42.9-11.el7 for package: libss-1.42.9-11.el7.x86_64
--> Processing Dependency: libcom_err(x86-64) = 1.42.9-11.el7 for package: e2fsprogs-libs-1.42.9-11.el7.x86_64
--> Processing Dependency: libcom_err(x86-64) = 1.42.9-11.el7 for package: e2fsprogs-1.42.9-11.el7.x86_64
---> Package libcom_err.x86_64 0:1.42.9-12.el7_5 will be an update
---> Package libsepol-devel.x86_64 0:2.5-8.1.el7 will be installed
---> Package pcre-devel.x86_64 0:8.32-17.el7 will be installed
---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
---> Package perl-Pod-Escapes.noarch 1:1.04-292.el7 will be installed
---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
--> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
--> Running transaction check
---> Package e2fsprogs.x86_64 0:1.42.9-11.el7 will be updated
---> Package e2fsprogs.x86_64 0:1.42.9-12.el7_5 will be an update
---> Package e2fsprogs-libs.x86_64 0:1.42.9-11.el7 will be updated
---> Package e2fsprogs-libs.x86_64 0:1.42.9-12.el7_5 will be an update
---> Package libss.x86_64 0:1.42.9-11.el7 will be updated
---> Package libss.x86_64 0:1.42.9-12.el7_5 will be an update
---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                    Arch       Version                Repository   Size
================================================================================
Installing:
 automake                   noarch     1.13.4-3.el7           base        679 k
 epel-release               noarch     7-11                   extras       15 k
 gcc                        x86_64     4.8.5-28.el7_5.1       updates      16 M
 git                        x86_64     1.8.3.1-14.el7_5       updates     4.4 M
 openssl-devel              x86_64     1:1.0.2k-12.el7        base        1.5 M
 sqlite-devel               x86_64     3.7.17-8.el7           base        104 k
 wget                       x86_64     1.14-15.el7_4.1        base        547 k
 zlib-devel                 x86_64     1.2.7-17.el7           base         50 k
Installing for dependencies:
 autoconf                   noarch     2.69-11.el7            base        701 k
 cpp                        x86_64     4.8.5-28.el7_5.1       updates     5.9 M
 glibc-devel                x86_64     2.17-222.el7           base        1.1 M
 glibc-headers              x86_64     2.17-222.el7           base        678 k
 kernel-headers             x86_64     3.10.0-862.9.1.el7     updates     7.1 M
 keyutils-libs-devel        x86_64     1.5.8-3.el7            base         37 k
 krb5-devel                 x86_64     1.15.1-19.el7          updates     269 k
 libcom_err-devel           x86_64     1.42.9-12.el7_5        updates      31 k
 libgnome-keyring           x86_64     3.12.0-1.el7           base        109 k
 libkadm5                   x86_64     1.15.1-19.el7          updates     175 k
 libmpc                     x86_64     1.0.1-3.el7            base         51 k
 libselinux-devel           x86_64     2.5-12.el7             base        186 k
 libsepol-devel             x86_64     2.5-8.1.el7            base         77 k
 libverto-devel             x86_64     0.2.5-4.el7            base         12 k
 m4                         x86_64     1.4.16-10.el7          base        256 k
 mpfr                       x86_64     3.1.1-4.el7            base        203 k
 pcre-devel                 x86_64     8.32-17.el7            base        480 k
 perl                       x86_64     4:5.16.3-292.el7       base        8.0 M
 perl-Carp                  noarch     1.26-244.el7           base         19 k
 perl-Data-Dumper           x86_64     2.145-3.el7            base         47 k
 perl-Encode                x86_64     2.51-7.el7             base        1.5 M
 perl-Error                 noarch     1:0.17020-2.el7        base         32 k
 perl-Exporter              noarch     5.68-3.el7             base         28 k
 perl-File-Path             noarch     2.09-2.el7             base         26 k
 perl-File-Temp             noarch     0.23.01-3.el7          base         56 k
 perl-Filter                x86_64     1.49-3.el7             base         76 k
 perl-Getopt-Long           noarch     2.40-3.el7             base         56 k
 perl-Git                   noarch     1.8.3.1-14.el7_5       updates      54 k
 perl-HTTP-Tiny             noarch     0.033-3.el7            base         38 k
 perl-PathTools             x86_64     3.40-5.el7             base         82 k
 perl-Pod-Escapes           noarch     1:1.04-292.el7         base         51 k
 perl-Pod-Perldoc           noarch     3.20-4.el7             base         87 k
 perl-Pod-Simple            noarch     1:3.28-4.el7           base        216 k
 perl-Pod-Usage             noarch     1.63-3.el7             base         27 k
 perl-Scalar-List-Utils     x86_64     1.27-248.el7           base         36 k
 perl-Socket                x86_64     2.010-4.el7            base         49 k
 perl-Storable              x86_64     2.45-3.el7             base         77 k
 perl-TermReadKey           x86_64     2.30-20.el7            base         31 k
 perl-Test-Harness          noarch     3.28-3.el7             base        302 k
 perl-Text-ParseWords       noarch     3.29-4.el7             base         14 k
 perl-Thread-Queue          noarch     3.02-2.el7             base         17 k
 perl-Time-HiRes            x86_64     4:1.9725-3.el7         base         45 k
 perl-Time-Local            noarch     1.2300-2.el7           base         24 k
 perl-constant              noarch     1.27-2.el7             base         19 k
 perl-libs                  x86_64     4:5.16.3-292.el7       base        688 k
 perl-macros                x86_64     4:5.16.3-292.el7       base         43 k
 perl-parent                noarch     1:0.225-244.el7        base         12 k
 perl-podlators             noarch     2.5.1-3.el7            base        112 k
 perl-threads               x86_64     1.87-4.el7             base         49 k
 perl-threads-shared        x86_64     1.43-6.el7             base         39 k
Updating for dependencies:
 e2fsprogs                  x86_64     1.42.9-12.el7_5        updates     699 k
 e2fsprogs-libs             x86_64     1.42.9-12.el7_5        updates     167 k
 libcom_err                 x86_64     1.42.9-12.el7_5        updates      41 k
 libgcc                     x86_64     4.8.5-28.el7_5.1       updates     101 k
 libgomp                    x86_64     4.8.5-28.el7_5.1       updates     156 k
 libss                      x86_64     1.42.9-12.el7_5        updates      45 k

Transaction Summary
================================================================================
Install  8 Packages (+50 Dependent packages)
Upgrade             (  6 Dependent packages)

Total download size: 54 M
Downloading packages:
updates/7/x86_64/prestodelta                               | 383 kB   00:00     
Delta RPMs reduced 1.2 M of updates to 477 k (60% saved)
(1/64): libcom_err-1.42.9-11.el7_1.42.9-12.el7_5.x86_64.dr |  26 kB   00:00     
(2/64): libss-1.42.9-11.el7_1.42.9-12.el7_5.x86_64.drpm    |  27 kB   00:00     
/usr/share/doc/libcom_err-1.42.9/COPYING: No such file or directory
cannot reconstruct rpm from disk files
/usr/share/doc/libss-1.42.9/COPYING: No such file or directory
cannot reconstruct rpm from disk files
(3/64): e2fsprogs-libs-1.42.9-11.el7_1.42.9-12.el7_5.x86_6 |  39 kB   00:00     
(4/64): libgomp-4.8.5-28.el7_4.8.5-28.el7_5.1.x86_64.drpm  |  47 kB   00:00     
(5/64): libgcc-4.8.5-28.el7_4.8.5-28.el7_5.1.x86_64.drpm   |  44 kB   00:00     
/usr/share/doc/libgomp-4.8.5/ChangeLog.bz2: No such file or directory
cannot reconstruct rpm from disk files
/usr/share/doc/e2fsprogs-libs-1.42.9/COPYING: No such file or directory-:-- ETA 
cannot reconstruct rpm from disk files
(6/64): e2fsprogs-1.42.9-11.el7_1.42.9-12.el7_5.x86_64.drp | 293 kB   00:00     
/usr/share/doc/e2fsprogs-1.42.9/COPYING: No such file or directory
cannot reconstruct rpm from disk files
/usr/share/doc/libgcc-4.8.5/COPYING: No such file or directory
cannot reconstruct rpm from disk files
warning: /var/cache/yum/x86_64/7/extras/packages/epel-release-7-11.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for epel-release-7-11.noarch.rpm is not installed
(7/64): epel-release-7-11.noarch.rpm                       |  15 kB   00:00     
Public key for autoconf-2.69-11.el7.noarch.rpm is not installed7 MB   01:02 ETA 
(8/64): autoconf-2.69-11.el7.noarch.rpm                    | 701 kB   00:01     
(9/64): automake-1.13.4-3.el7.noarch.rpm                   | 679 kB   00:01     
(10/64): glibc-devel-2.17-222.el7.x86_64.rpm               | 1.1 MB   00:00     
(11/64): glibc-headers-2.17-222.el7.x86_64.rpm             | 678 kB   00:01     
(12/64): keyutils-libs-devel-1.5.8-3.el7.x86_64.rpm        |  37 kB   00:00     
Public key for krb5-devel-1.15.1-19.el7.x86_64.rpm is not installed   00:21 ETA 
(13/64): krb5-devel-1.15.1-19.el7.x86_64.rpm               | 269 kB   00:01     
(14/64): libcom_err-devel-1.42.9-12.el7_5.x86_64.rpm       |  31 kB   00:00     
(15/64): libgnome-keyring-3.12.0-1.el7.x86_64.rpm          | 109 kB   00:00     
(16/64): git-1.8.3.1-14.el7_5.x86_64.rpm                   | 4.4 MB   00:04     
(17/64): libmpc-1.0.1-3.el7.x86_64.rpm                     |  51 kB   00:00     
(18/64): libselinux-devel-2.5-12.el7.x86_64.rpm            | 186 kB   00:00     
(19/64): libkadm5-1.15.1-19.el7.x86_64.rpm                 | 175 kB   00:00     
(20/64): libverto-devel-0.2.5-4.el7.x86_64.rpm             |  12 kB   00:00     
(21/64): libsepol-devel-2.5-8.1.el7.x86_64.rpm             |  77 kB   00:00     
(22/64): m4-1.4.16-10.el7.x86_64.rpm                       | 256 kB   00:00     
(23/64): mpfr-3.1.1-4.el7.x86_64.rpm                       | 203 kB   00:00     
(24/64): pcre-devel-8.32-17.el7.x86_64.rpm                 | 480 kB   00:00     
(25/64): openssl-devel-1.0.2k-12.el7.x86_64.rpm            | 1.5 MB   00:01     
(26/64): perl-Carp-1.26-244.el7.noarch.rpm                 |  19 kB   00:00     
(27/64): perl-Data-Dumper-2.145-3.el7.x86_64.rpm           |  47 kB   00:00     
(28/64): perl-Encode-2.51-7.el7.x86_64.rpm                 | 1.5 MB   00:01     
(29/64): perl-Error-0.17020-2.el7.noarch.rpm               |  32 kB   00:00     
(30/64): perl-Exporter-5.68-3.el7.noarch.rpm               |  28 kB   00:00     
(31/64): perl-File-Path-2.09-2.el7.noarch.rpm              |  26 kB   00:00     
(32/64): perl-File-Temp-0.23.01-3.el7.noarch.rpm           |  56 kB   00:00     
(33/64): perl-Filter-1.49-3.el7.x86_64.rpm                 |  76 kB   00:00     
(34/64): perl-Getopt-Long-2.40-3.el7.noarch.rpm            |  56 kB   00:00     
(35/64): perl-Git-1.8.3.1-14.el7_5.noarch.rpm              |  54 kB   00:00     
(36/64): perl-HTTP-Tiny-0.033-3.el7.noarch.rpm             |  38 kB   00:00     
(37/64): perl-PathTools-3.40-5.el7.x86_64.rpm              |  82 kB   00:00     
(38/64): perl-Pod-Escapes-1.04-292.el7.noarch.rpm          |  51 kB   00:01     
(39/64): perl-Pod-Perldoc-3.20-4.el7.noarch.rpm            |  87 kB   00:00     
(40/64): perl-Pod-Simple-3.28-4.el7.noarch.rpm             | 216 kB   00:00     
(41/64): perl-Pod-Usage-1.63-3.el7.noarch.rpm              |  27 kB   00:00     
(42/64): perl-Scalar-List-Utils-1.27-248.el7.x86_64.rpm    |  36 kB   00:00     
(43/64): perl-Socket-2.010-4.el7.x86_64.rpm                |  49 kB   00:00     
(44/64): perl-Storable-2.45-3.el7.x86_64.rpm               |  77 kB   00:00     
(45/64): perl-TermReadKey-2.30-20.el7.x86_64.rpm           |  31 kB   00:00     
(46/64): perl-Test-Harness-3.28-3.el7.noarch.rpm           | 302 kB   00:00     
(47/64): perl-Text-ParseWords-3.29-4.el7.noarch.rpm        |  14 kB   00:00     
(48/64): perl-Thread-Queue-3.02-2.el7.noarch.rpm           |  17 kB   00:00     
(49/64): perl-Time-HiRes-1.9725-3.el7.x86_64.rpm           |  45 kB   00:00     
(50/64): perl-Time-Local-1.2300-2.el7.noarch.rpm           |  24 kB   00:00     
(51/64): perl-constant-1.27-2.el7.noarch.rpm               |  19 kB   00:00     
(52/64): perl-libs-5.16.3-292.el7.x86_64.rpm               | 688 kB   00:00     
(53/64): perl-macros-5.16.3-292.el7.x86_64.rpm             |  43 kB   00:00     
(54/64): perl-parent-0.225-244.el7.noarch.rpm              |  12 kB   00:00     
(55/64): perl-podlators-2.5.1-3.el7.noarch.rpm             | 112 kB   00:00     
(56/64): perl-threads-1.87-4.el7.x86_64.rpm                |  49 kB   00:00     
(57/64): perl-threads-shared-1.43-6.el7.x86_64.rpm         |  39 kB   00:00     
(58/64): sqlite-devel-3.7.17-8.el7.x86_64.rpm              | 104 kB   00:00     
(59/64): wget-1.14-15.el7_4.1.x86_64.rpm                   | 547 kB   00:00     
(60/64): zlib-devel-1.2.7-17.el7.x86_64.rpm                |  50 kB   00:00     
(61/64): cpp-4.8.5-28.el7_5.1.x86_64.rpm                   | 5.9 MB   00:18     
(62/64): perl-5.16.3-292.el7.x86_64.rpm                    | 8.0 MB   00:14     
(63/64): kernel-headers-3.10.0-862.9.1.el7.x86_64.rpm      | 7.1 MB   00:18     
(64/64): gcc-4.8.5-28.el7_5.1.x86_64.rpm                   |  16 MB   00:25     
Some delta RPMs failed to download or rebuild. Retrying..
Public key for libcom_err-1.42.9-12.el7_5.x86_64.rpm is not installed
(1/6): libcom_err-1.42.9-12.el7_5.x86_64.rpm               |  41 kB   00:00     
(2/6): libss-1.42.9-12.el7_5.x86_64.rpm                    |  45 kB   00:00     
(3/6): e2fsprogs-libs-1.42.9-12.el7_5.x86_64.rpm           | 167 kB   00:00     
(4/6): libgcc-4.8.5-28.el7_5.1.x86_64.rpm                  | 101 kB   00:00     
(5/6): libgomp-4.8.5-28.el7_5.1.x86_64.rpm                 | 156 kB   00:00     
(6/6): e2fsprogs-1.42.9-12.el7_5.x86_64.rpm                | 699 kB   00:01     
--------------------------------------------------------------------------------
Total                                              2.0 MB/s |  54 MB  00:27     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-5.1804.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : libcom_err-1.42.9-12.el7_5.x86_64                           1/70 
  Installing : mpfr-3.1.1-4.el7.x86_64                                     2/70 
  Installing : libmpc-1.0.1-3.el7.x86_64                                   3/70 
  Installing : cpp-4.8.5-28.el7_5.1.x86_64                                 4/70 
  Updating   : libss-1.42.9-12.el7_5.x86_64                                5/70 
  Installing : libcom_err-devel-1.42.9-12.el7_5.x86_64                     6/70 
  Updating   : e2fsprogs-libs-1.42.9-12.el7_5.x86_64                       7/70 
  Installing : libkadm5-1.15.1-19.el7.x86_64                               8/70 
  Installing : 1:perl-parent-0.225-244.el7.noarch                          9/70 
  Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                          10/70 
  Installing : perl-podlators-2.5.1-3.el7.noarch                          11/70 
  Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                         12/70 
  Installing : perl-Encode-2.51-7.el7.x86_64                              13/70 
  Installing : perl-Text-ParseWords-3.29-4.el7.noarch                     14/70 
  Installing : 1:perl-Pod-Escapes-1.04-292.el7.noarch                     15/70 
  Installing : perl-Pod-Usage-1.63-3.el7.noarch                           16/70 
  Installing : 4:perl-macros-5.16.3-292.el7.x86_64                        17/70 
  Installing : perl-Exporter-5.68-3.el7.noarch                            18/70 
  Installing : perl-constant-1.27-2.el7.noarch                            19/70 
  Installing : perl-Time-Local-1.2300-2.el7.noarch                        20/70 
  Installing : perl-Socket-2.010-4.el7.x86_64                             21/70 
  Installing : perl-Carp-1.26-244.el7.noarch                              22/70 
  Installing : 4:perl-libs-5.16.3-292.el7.x86_64                          23/70 
  Installing : perl-PathTools-3.40-5.el7.x86_64                           24/70 
  Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 25/70 
  Installing : perl-Storable-2.45-3.el7.x86_64                            26/70 
  Installing : perl-File-Temp-0.23.01-3.el7.noarch                        27/70 
  Installing : perl-File-Path-2.09-2.el7.noarch                           28/70 
  Installing : perl-threads-shared-1.43-6.el7.x86_64                      29/70 
  Installing : perl-threads-1.87-4.el7.x86_64                             30/70 
  Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      31/70 
  Installing : perl-Filter-1.49-3.el7.x86_64                              32/70 
  Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        33/70 
  Installing : perl-Getopt-Long-2.40-3.el7.noarch                         34/70 
  Installing : 4:perl-5.16.3-292.el7.x86_64                               35/70 
  Installing : 1:perl-Error-0.17020-2.el7.noarch                          36/70 
  Installing : perl-TermReadKey-2.30-20.el7.x86_64                        37/70 
  Installing : perl-Test-Harness-3.28-3.el7.noarch                        38/70 
  Installing : perl-Thread-Queue-3.02-2.el7.noarch                        39/70 
  Installing : perl-Data-Dumper-2.145-3.el7.x86_64                        40/70 
  Installing : libsepol-devel-2.5-8.1.el7.x86_64                          41/70 
  Installing : pcre-devel-8.32-17.el7.x86_64                              42/70 
  Installing : libselinux-devel-2.5-12.el7.x86_64                         43/70 
  Updating   : libgcc-4.8.5-28.el7_5.1.x86_64                             44/70 
  Installing : libverto-devel-0.2.5-4.el7.x86_64                          45/70 
  Installing : kernel-headers-3.10.0-862.9.1.el7.x86_64                   46/70 
  Installing : glibc-headers-2.17-222.el7.x86_64                          47/70 
  Installing : glibc-devel-2.17-222.el7.x86_64                            48/70 
  Updating   : libgomp-4.8.5-28.el7_5.1.x86_64                            49/70 
  Installing : m4-1.4.16-10.el7.x86_64                                    50/70 
  Installing : autoconf-2.69-11.el7.noarch                                51/70 
  Installing : keyutils-libs-devel-1.5.8-3.el7.x86_64                     52/70 
  Installing : krb5-devel-1.15.1-19.el7.x86_64                            53/70 
  Installing : libgnome-keyring-3.12.0-1.el7.x86_64                       54/70 
  Installing : perl-Git-1.8.3.1-14.el7_5.noarch                           55/70 
  Installing : git-1.8.3.1-14.el7_5.x86_64                                56/70 
  Installing : zlib-devel-1.2.7-17.el7.x86_64                             57/70 
  Installing : 1:openssl-devel-1.0.2k-12.el7.x86_64                       58/70 
  Installing : automake-1.13.4-3.el7.noarch                               59/70 
  Installing : gcc-4.8.5-28.el7_5.1.x86_64                                60/70 
  Updating   : e2fsprogs-1.42.9-12.el7_5.x86_64                           61/70 
  Installing : sqlite-devel-3.7.17-8.el7.x86_64                           62/70 
  Installing : wget-1.14-15.el7_4.1.x86_64                                63/70 
  Installing : epel-release-7-11.noarch                                   64/70 
  Cleanup    : e2fsprogs-1.42.9-11.el7.x86_64                             65/70 
  Cleanup    : e2fsprogs-libs-1.42.9-11.el7.x86_64                        66/70 
  Cleanup    : libss-1.42.9-11.el7.x86_64                                 67/70 
  Cleanup    : libcom_err-1.42.9-11.el7.x86_64                            68/70 
  Cleanup    : libgcc-4.8.5-28.el7.x86_64                                 69/70 
  Cleanup    : libgomp-4.8.5-28.el7.x86_64                                70/70 
  Verifying  : krb5-devel-1.15.1-19.el7.x86_64                             1/70 
  Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           2/70 
  Verifying  : zlib-devel-1.2.7-17.el7.x86_64                              3/70 
  Verifying  : libgnome-keyring-3.12.0-1.el7.x86_64                        4/70 
  Verifying  : keyutils-libs-devel-1.5.8-3.el7.x86_64                      5/70 
  Verifying  : libss-1.42.9-12.el7_5.x86_64                                6/70 
  Verifying  : perl-threads-shared-1.43-6.el7.x86_64                       7/70 
  Verifying  : glibc-devel-2.17-222.el7.x86_64                             8/70 
  Verifying  : mpfr-3.1.1-4.el7.x86_64                                     9/70 
  Verifying  : perl-Exporter-5.68-3.el7.noarch                            10/70 
  Verifying  : perl-constant-1.27-2.el7.noarch                            11/70 
  Verifying  : perl-PathTools-3.40-5.el7.x86_64                           12/70 
  Verifying  : glibc-headers-2.17-222.el7.x86_64                          13/70 
  Verifying  : 4:perl-macros-5.16.3-292.el7.x86_64                        14/70 
  Verifying  : automake-1.13.4-3.el7.noarch                               15/70 
  Verifying  : m4-1.4.16-10.el7.x86_64                                    16/70 
  Verifying  : libgomp-4.8.5-28.el7_5.1.x86_64                            17/70 
  Verifying  : kernel-headers-3.10.0-862.9.1.el7.x86_64                   18/70 
  Verifying  : 1:openssl-devel-1.0.2k-12.el7.x86_64                       19/70 
  Verifying  : libverto-devel-0.2.5-4.el7.x86_64                          20/70 
  Verifying  : perl-TermReadKey-2.30-20.el7.x86_64                        21/70 
  Verifying  : libselinux-devel-2.5-12.el7.x86_64                         22/70 
  Verifying  : libcom_err-1.42.9-12.el7_5.x86_64                          23/70 
  Verifying  : perl-Test-Harness-3.28-3.el7.noarch                        24/70 
  Verifying  : gcc-4.8.5-28.el7_5.1.x86_64                                25/70 
  Verifying  : perl-Thread-Queue-3.02-2.el7.noarch                        26/70 
  Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                        27/70 
  Verifying  : perl-Data-Dumper-2.145-3.el7.x86_64                        28/70 
  Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        29/70 
  Verifying  : libcom_err-devel-1.42.9-12.el7_5.x86_64                    30/70 
  Verifying  : e2fsprogs-libs-1.42.9-12.el7_5.x86_64                      31/70 
  Verifying  : epel-release-7-11.noarch                                   32/70 
  Verifying  : perl-Socket-2.010-4.el7.x86_64                             33/70 
  Verifying  : e2fsprogs-1.42.9-12.el7_5.x86_64                           34/70 
  Verifying  : git-1.8.3.1-14.el7_5.x86_64                                35/70 
  Verifying  : perl-Carp-1.26-244.el7.noarch                              36/70 
  Verifying  : libgcc-4.8.5-28.el7_5.1.x86_64                             37/70 
  Verifying  : wget-1.14-15.el7_4.1.x86_64                                38/70 
  Verifying  : 1:perl-Error-0.17020-2.el7.noarch                          39/70 
  Verifying  : 4:perl-libs-5.16.3-292.el7.x86_64                          40/70 
  Verifying  : 1:perl-parent-0.225-244.el7.noarch                         41/70 
  Verifying  : sqlite-devel-3.7.17-8.el7.x86_64                           42/70 
  Verifying  : cpp-4.8.5-28.el7_5.1.x86_64                                43/70 
  Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 44/70 
  Verifying  : libmpc-1.0.1-3.el7.x86_64                                  45/70 
  Verifying  : pcre-devel-8.32-17.el7.x86_64                              46/70 
  Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           47/70 
  Verifying  : perl-Git-1.8.3.1-14.el7_5.noarch                           48/70 
  Verifying  : 4:perl-5.16.3-292.el7.x86_64                               49/70 
  Verifying  : perl-Encode-2.51-7.el7.x86_64                              50/70 
  Verifying  : perl-Storable-2.45-3.el7.x86_64                            51/70 
  Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         52/70 
  Verifying  : perl-podlators-2.5.1-3.el7.noarch                          53/70 
  Verifying  : autoconf-2.69-11.el7.noarch                                54/70 
  Verifying  : perl-File-Path-2.09-2.el7.noarch                           55/70 
  Verifying  : perl-threads-1.87-4.el7.x86_64                             56/70 
  Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      57/70 
  Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        58/70 
  Verifying  : perl-Filter-1.49-3.el7.x86_64                              59/70 
  Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         60/70 
  Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     61/70 
  Verifying  : libkadm5-1.15.1-19.el7.x86_64                              62/70 
  Verifying  : libsepol-devel-2.5-8.1.el7.x86_64                          63/70 
  Verifying  : 1:perl-Pod-Escapes-1.04-292.el7.noarch                     64/70 
  Verifying  : libgomp-4.8.5-28.el7.x86_64                                65/70 
  Verifying  : libcom_err-1.42.9-11.el7.x86_64                            66/70 
  Verifying  : libgcc-4.8.5-28.el7.x86_64                                 67/70 
  Verifying  : libss-1.42.9-11.el7.x86_64                                 68/70 
  Verifying  : e2fsprogs-libs-1.42.9-11.el7.x86_64                        69/70 
  Verifying  : e2fsprogs-1.42.9-11.el7.x86_64                             70/70 

Installed:
  automake.noarch 0:1.13.4-3.el7          epel-release.noarch 0:7-11           
  gcc.x86_64 0:4.8.5-28.el7_5.1           git.x86_64 0:1.8.3.1-14.el7_5        
  openssl-devel.x86_64 1:1.0.2k-12.el7    sqlite-devel.x86_64 0:3.7.17-8.el7   
  wget.x86_64 0:1.14-15.el7_4.1           zlib-devel.x86_64 0:1.2.7-17.el7     

Dependency Installed:
  autoconf.noarch 0:2.69-11.el7                                                 
  cpp.x86_64 0:4.8.5-28.el7_5.1                                                 
  glibc-devel.x86_64 0:2.17-222.el7                                             
  glibc-headers.x86_64 0:2.17-222.el7                                           
  kernel-headers.x86_64 0:3.10.0-862.9.1.el7                                    
  keyutils-libs-devel.x86_64 0:1.5.8-3.el7                                      
  krb5-devel.x86_64 0:1.15.1-19.el7                                             
  libcom_err-devel.x86_64 0:1.42.9-12.el7_5                                     
  libgnome-keyring.x86_64 0:3.12.0-1.el7                                        
  libkadm5.x86_64 0:1.15.1-19.el7                                               
  libmpc.x86_64 0:1.0.1-3.el7                                                   
  libselinux-devel.x86_64 0:2.5-12.el7                                          
  libsepol-devel.x86_64 0:2.5-8.1.el7                                           
  libverto-devel.x86_64 0:0.2.5-4.el7                                           
  m4.x86_64 0:1.4.16-10.el7                                                     
  mpfr.x86_64 0:3.1.1-4.el7                                                     
  pcre-devel.x86_64 0:8.32-17.el7                                               
  perl.x86_64 4:5.16.3-292.el7                                                  
  perl-Carp.noarch 0:1.26-244.el7                                               
  perl-Data-Dumper.x86_64 0:2.145-3.el7                                         
  perl-Encode.x86_64 0:2.51-7.el7                                               
  perl-Error.noarch 1:0.17020-2.el7                                             
  perl-Exporter.noarch 0:5.68-3.el7                                             
  perl-File-Path.noarch 0:2.09-2.el7                                            
  perl-File-Temp.noarch 0:0.23.01-3.el7                                         
  perl-Filter.x86_64 0:1.49-3.el7                                               
  perl-Getopt-Long.noarch 0:2.40-3.el7                                          
  perl-Git.noarch 0:1.8.3.1-14.el7_5                                            
  perl-HTTP-Tiny.noarch 0:0.033-3.el7                                           
  perl-PathTools.x86_64 0:3.40-5.el7                                            
  perl-Pod-Escapes.noarch 1:1.04-292.el7                                        
  perl-Pod-Perldoc.noarch 0:3.20-4.el7                                          
  perl-Pod-Simple.noarch 1:3.28-4.el7                                           
  perl-Pod-Usage.noarch 0:1.63-3.el7                                            
  perl-Scalar-List-Utils.x86_64 0:1.27-248.el7                                  
  perl-Socket.x86_64 0:2.010-4.el7                                              
  perl-Storable.x86_64 0:2.45-3.el7                                             
  perl-TermReadKey.x86_64 0:2.30-20.el7                                         
  perl-Test-Harness.noarch 0:3.28-3.el7                                         
  perl-Text-ParseWords.noarch 0:3.29-4.el7                                      
  perl-Thread-Queue.noarch 0:3.02-2.el7                                         
  perl-Time-HiRes.x86_64 4:1.9725-3.el7                                         
  perl-Time-Local.noarch 0:1.2300-2.el7                                         
  perl-constant.noarch 0:1.27-2.el7                                             
  perl-libs.x86_64 4:5.16.3-292.el7                                             
  perl-macros.x86_64 4:5.16.3-292.el7                                           
  perl-parent.noarch 1:0.225-244.el7                                            
  perl-podlators.noarch 0:2.5.1-3.el7                                           
  perl-threads.x86_64 0:1.87-4.el7                                              
  perl-threads-shared.x86_64 0:1.43-6.el7                                       

Dependency Updated:
  e2fsprogs.x86_64 0:1.42.9-12.el7_5   e2fsprogs-libs.x86_64 0:1.42.9-12.el7_5 
  libcom_err.x86_64 0:1.42.9-12.el7_5  libgcc.x86_64 0:4.8.5-28.el7_5.1        
  libgomp.x86_64 0:4.8.5-28.el7_5.1    libss.x86_64 0:1.42.9-12.el7_5          

Complete!
[root@h165 tmp]# echo $?
0
[root@h165 tmp]# 
~~~

## 下载 python 源码 ##

~~~
[root@h165 tmp]# wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tar.xz
--2018-07-22 12:46:12--  https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tar.xz
正在解析主机 www.python.org (www.python.org)... 151.101.24.223, 2a04:4e42:6::223
正在连接 www.python.org (www.python.org)|151.101.24.223|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：16872064 (16M) [application/octet-stream]
正在保存至: “Python-3.6.1.tar.xz”

100%[===============================================>] 16,872,064  3.26MB/s 用时 8.4s   

2018-07-22 12:46:22 (1.92 MB/s) - 已保存 “Python-3.6.1.tar.xz” [16872064/16872064])

[root@h165 tmp]# 
~~~


## 解压源码包 ##

~~~
[root@h165 tmp]# tar xvf Python-3.6.1.tar.xz  && cd Python-3.6.1
Python-3.6.1/
Python-3.6.1/Doc/
Python-3.6.1/Doc/c-api/
Python-3.6.1/Doc/c-api/sys.rst
Python-3.6.1/Doc/c-api/conversion.rst
Python-3.6.1/Doc/c-api/marshal.rst
Python-3.6.1/Doc/c-api/coro.rst
Python-3.6.1/Doc/c-api/method.rst
Python-3.6.1/Doc/c-api/index.rst
Python-3.6.1/Doc/c-api/bytearray.rst
Python-3.6.1/Doc/library/html.entities.rst
...
...
Python-3.6.1/Objects/methodobject.c
Python-3.6.1/Objects/tupleobject.c
Python-3.6.1/Objects/obmalloc.c
Python-3.6.1/Objects/object.c
Python-3.6.1/Objects/abstract.c
Python-3.6.1/Objects/listobject.c
Python-3.6.1/Objects/bytes_methods.c
Python-3.6.1/Objects/dictnotes.txt
Python-3.6.1/Objects/typeslots.inc
[root@h165 Python-3.6.1]# 
~~~

## 进行编译安装 ##

~~~
[root@h165 Python-3.6.1]# ./configure && make && make install
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking for python3.6... no
checking for python3... no
checking for python... python
checking for --enable-universalsdk... no
checking for --with-universal-archs... no
checking MACHDEP... linux
checking for --without-gcc... no
checking for --with-icc... no
checking for gcc... gcc
...
...
rm -f /usr/local/bin/python3-config
(cd /usr/local/bin; ln -s python3.6-config python3-config)
rm -f /usr/local/lib/pkgconfig/python3.pc
(cd /usr/local/lib/pkgconfig; ln -s python-3.6.pc python3.pc)
rm -f /usr/local/bin/idle3
(cd /usr/local/bin; ln -s idle3.6 idle3)
rm -f /usr/local/bin/pydoc3
(cd /usr/local/bin; ln -s pydoc3.6 pydoc3)
rm -f /usr/local/bin/2to3
(cd /usr/local/bin; ln -s 2to3-3.6 2to3)
rm -f /usr/local/bin/pyvenv
(cd /usr/local/bin; ln -s pyvenv-3.6 pyvenv)
if test "x" != "x" ; then \
	rm -f /usr/local/bin/python3-32; \
	(cd /usr/local/bin; ln -s python3.6-32 python3-32) \
fi
rm -f /usr/local/share/man/man1/python3.1
(cd /usr/local/share/man/man1; ln -s python3.6.1 python3.1)
if test "xupgrade" != "xno"  ; then \
	case upgrade in \
		upgrade) ensurepip="--upgrade" ;; \
		install|*) ensurepip="" ;; \
	esac; \
	 ./python -E -m ensurepip \
		$ensurepip --root=/ ; \
fi
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-9.0.1 setuptools-28.8.0
[root@h165 Python-3.6.1]# echo $?
0
[root@h165 Python-3.6.1]# 
~~~


## 建立 Python 虚拟环境 ##

为了不扰乱原来的环境我们来使用 Python 虚拟环境

~~~
[root@h165 Python-3.6.1]# cd /opt/
[root@h165 opt]# 
[root@h165 opt]# python3 -m venv py3
[root@h165 opt]# source /opt/py3/bin/activate
(py3) [root@h165 opt]# 
(py3) [root@h165 opt]# 
(py3) [root@h165 opt]#
~~~

看到上面的提示符代表成功，以后运行 Jumpserver 都要先运行以上 source 命令，以下所有命令均在该虚拟环境中运行

~~~
[vagrant@h165 ~]$ cd /opt/
[vagrant@h165 opt]$ git clone git://github.com/kennethreitz/autoenv.git
fatal: could not create work tree dir 'autoenv'.: Permission denied
[vagrant@h165 opt]$ sudo git clone git://github.com/kennethreitz/autoenv.git
Cloning into 'autoenv'...
remote: Counting objects: 671, done.
remote: Total 671 (delta 0), reused 0 (delta 0), pack-reused 671
Receiving objects: 100% (671/671), 103.92 KiB | 0 bytes/s, done.
Resolving deltas: 100% (356/356), done.
[vagrant@h165 opt]$ echo 'source /opt/autoenv/activate.sh' >> ~/.bashrc
[vagrant@h165 opt]$ source ~/.bashrc
[vagrant@h165 opt]$
~~~

之后进入此目录，会有提醒

~~~
[vagrant@h165 ~]$ cd /opt/jumpserver/
autoenv:
autoenv: WARNING:
autoenv: This is the first time you are about to source /opt/jumpserver/.env:
autoenv:
autoenv:   --- (begin contents) ---------------------------------------
autoenv:     source /opt/py3/bin/activate$
autoenv:
autoenv:   --- (end contents) -----------------------------------------
autoenv:
autoenv: Are you sure you want to allow this? (y/N) y
(py3) [vagrant@h165 jumpserver]$ 
(py3) [vagrant@h165 jumpserver]$ 
(py3) [vagrant@h165 jumpserver]$ 
~~~


## 下载 Jumpserver 项目 ##

~~~
[root@h165 ~]# cd /opt/
[root@h165 opt]# git clone https://github.com/jumpserver/jumpserver.git && cd jumpserver && git checkout master
正克隆到 'jumpserver'...
remote: Counting objects: 30587, done.
remote: Compressing objects: 100% (121/121), done.
remote: Total 30587 (delta 77), reused 95 (delta 42), pack-reused 30420
接收对象中: 100% (30587/30587), 40.98 MiB | 2.30 MiB/s, done.
处理 delta 中: 100% (21128/21128), done.
已经位于 'master'
[root@h165 jumpserver]# 
~~~



## 下载安装依赖包 ##


~~~
[root@h165 ~]# cd /opt/jumpserver/requirements
[root@h165 requirements]# cat rpm_requirements.txt 
libtiff-devel libjpeg-devel libzip-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel sshpass openldap-devel mysql-devel libffi-devel openssh-clients
[root@h165 requirements]# yum -y install $(cat rpm_requirements.txt) 
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                              | 6.4 kB  00:00:00     
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
epel                                                              | 3.2 kB  00:00:01     
epel/x86_64/primary            FAILED                                          01:12 ETA 
http://mirror.smartmedia.net.id/epel/7/x86_64/repodata/2ee09d0b2a6f689166bc88e06103a893d8d841678883f3fde6ce77841da598ba-primary.xml.gz: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below wiki article 

https://wiki.centos.org/yum-errors

If above article doesn't help to resolve this issue please use https://bugs.centos.org/.

epel/x86_64/primary            FAILED                                          00:33 ETA 
http://ftp.cuhk.edu.hk/pub/linux/fedora-epel/7/x86_64/repodata/2ee09d0b2a6f689166bc88e06103a893d8d841678883f3fde6ce77841da598ba-primary.xml.gz: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
(1/3): epel/x86_64/updateinfo                                     | 930 kB  00:00:02     
(2/3): epel/x86_64/group_gz                                       |  88 kB  00:00:09     
(3/3): epel/x86_64/primary                                        | 3.5 MB  00:00:14     
epel                                                                         12618/12618
Package openssh-clients-7.4p1-16.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package freetype-devel.x86_64 0:2.4.11-15.el7 will be installed
---> Package lcms2-devel.x86_64 0:2.6-3.el7 will be installed
--> Processing Dependency: lcms2 = 2.6-3.el7 for package: lcms2-devel-2.6-3.el7.x86_64
--> Processing Dependency: liblcms2.so.2()(64bit) for package: lcms2-devel-2.6-3.el7.x86_64
---> Package libffi-devel.x86_64 0:3.0.13-18.el7 will be installed
---> Package libjpeg-turbo-devel.x86_64 0:1.2.90-5.el7 will be installed
--> Processing Dependency: libjpeg-turbo(x86-64) = 1.2.90-5.el7 for package: libjpeg-turbo-devel-1.2.90-5.el7.x86_64
--> Processing Dependency: libjpeg.so.62()(64bit) for package: libjpeg-turbo-devel-1.2.90-5.el7.x86_64
---> Package libtiff-devel.x86_64 0:4.0.3-27.el7_3 will be installed
--> Processing Dependency: libtiff(x86-64) = 4.0.3-27.el7_3 for package: libtiff-devel-4.0.3-27.el7_3.x86_64
--> Processing Dependency: libtiffxx.so.5()(64bit) for package: libtiff-devel-4.0.3-27.el7_3.x86_64
--> Processing Dependency: libtiff.so.5()(64bit) for package: libtiff-devel-4.0.3-27.el7_3.x86_64
---> Package libwebp-devel.x86_64 0:0.3.0-7.el7 will be installed
--> Processing Dependency: libwebp(x86-64) = 0.3.0-7.el7 for package: libwebp-devel-0.3.0-7.el7.x86_64
--> Processing Dependency: libwebpmux.so.0()(64bit) for package: libwebp-devel-0.3.0-7.el7.x86_64
--> Processing Dependency: libwebp.so.4()(64bit) for package: libwebp-devel-0.3.0-7.el7.x86_64
---> Package libzip-devel.x86_64 0:0.10.1-8.el7 will be installed
--> Processing Dependency: libzip(x86-64) = 0.10.1-8.el7 for package: libzip-devel-0.10.1-8.el7.x86_64
--> Processing Dependency: libzip.so.2()(64bit) for package: libzip-devel-0.10.1-8.el7.x86_64
---> Package mariadb-devel.x86_64 1:5.5.56-2.el7 will be installed
---> Package openldap-devel.x86_64 0:2.4.44-15.el7_5 will be installed
--> Processing Dependency: openldap(x86-64) = 2.4.44-15.el7_5 for package: openldap-devel-2.4.44-15.el7_5.x86_64
--> Processing Dependency: cyrus-sasl-devel(x86-64) for package: openldap-devel-2.4.44-15.el7_5.x86_64
---> Package sshpass.x86_64 0:1.06-2.el7 will be installed
---> Package tcl-devel.x86_64 1:8.5.13-8.el7 will be installed
--> Processing Dependency: tcl = 1:8.5.13-8.el7 for package: 1:tcl-devel-8.5.13-8.el7.x86_64
---> Package tk-devel.x86_64 1:8.5.13-6.el7 will be installed
--> Processing Dependency: tk = 1:8.5.13-6.el7 for package: 1:tk-devel-8.5.13-6.el7.x86_64
--> Processing Dependency: libXft-devel for package: 1:tk-devel-8.5.13-6.el7.x86_64
--> Processing Dependency: libX11-devel for package: 1:tk-devel-8.5.13-6.el7.x86_64
--> Running transaction check
---> Package cyrus-sasl-devel.x86_64 0:2.1.26-23.el7 will be installed
--> Processing Dependency: cyrus-sasl(x86-64) = 2.1.26-23.el7 for package: cyrus-sasl-devel-2.1.26-23.el7.x86_64
---> Package lcms2.x86_64 0:2.6-3.el7 will be installed
---> Package libX11-devel.x86_64 0:1.6.5-1.el7 will be installed
--> Processing Dependency: libX11 = 1.6.5-1.el7 for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(xcb) >= 1.11.1 for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(xproto) for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(xcb) for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(kbproto) for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: libX11.so.6()(64bit) for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: libX11-xcb.so.1()(64bit) for package: libX11-devel-1.6.5-1.el7.x86_64
---> Package libXft-devel.x86_64 0:2.3.2-2.el7 will be installed
--> Processing Dependency: libXft = 2.3.2-2.el7 for package: libXft-devel-2.3.2-2.el7.x86_64
--> Processing Dependency: pkgconfig(xrender) for package: libXft-devel-2.3.2-2.el7.x86_64
--> Processing Dependency: pkgconfig(fontconfig) for package: libXft-devel-2.3.2-2.el7.x86_64
--> Processing Dependency: libXft.so.2()(64bit) for package: libXft-devel-2.3.2-2.el7.x86_64
---> Package libjpeg-turbo.x86_64 0:1.2.90-5.el7 will be installed
---> Package libtiff.x86_64 0:4.0.3-27.el7_3 will be installed
--> Processing Dependency: libjbig.so.2.0()(64bit) for package: libtiff-4.0.3-27.el7_3.x86_64
---> Package libwebp.x86_64 0:0.3.0-7.el7 will be installed
---> Package libzip.x86_64 0:0.10.1-8.el7 will be installed
---> Package openldap.x86_64 0:2.4.44-13.el7 will be updated
---> Package openldap.x86_64 0:2.4.44-15.el7_5 will be an update
---> Package tcl.x86_64 1:8.5.13-8.el7 will be installed
---> Package tk.x86_64 1:8.5.13-6.el7 will be installed
--> Running transaction check
---> Package cyrus-sasl.x86_64 0:2.1.26-23.el7 will be installed
---> Package fontconfig-devel.x86_64 0:2.10.95-11.el7 will be installed
--> Processing Dependency: fontconfig(x86-64) = 2.10.95-11.el7 for package: fontconfig-devel-2.10.95-11.el7.x86_64
--> Processing Dependency: pkgconfig(expat) for package: fontconfig-devel-2.10.95-11.el7.x86_64
--> Processing Dependency: libfontconfig.so.1()(64bit) for package: fontconfig-devel-2.10.95-11.el7.x86_64
---> Package jbigkit-libs.x86_64 0:2.0-11.el7 will be installed
---> Package libX11.x86_64 0:1.6.5-1.el7 will be installed
--> Processing Dependency: libX11-common >= 1.6.5-1.el7 for package: libX11-1.6.5-1.el7.x86_64
--> Processing Dependency: libxcb.so.1()(64bit) for package: libX11-1.6.5-1.el7.x86_64
---> Package libXft.x86_64 0:2.3.2-2.el7 will be installed
--> Processing Dependency: libXrender.so.1()(64bit) for package: libXft-2.3.2-2.el7.x86_64
---> Package libXrender-devel.x86_64 0:0.9.10-1.el7 will be installed
---> Package libxcb-devel.x86_64 0:1.12-1.el7 will be installed
--> Processing Dependency: pkgconfig(xau) >= 0.99.2 for package: libxcb-devel-1.12-1.el7.x86_64
---> Package xorg-x11-proto-devel.noarch 0:7.7-20.el7 will be installed
--> Running transaction check
---> Package expat-devel.x86_64 0:2.1.0-10.el7_3 will be installed
---> Package fontconfig.x86_64 0:2.10.95-11.el7 will be installed
--> Processing Dependency: fontpackages-filesystem for package: fontconfig-2.10.95-11.el7.x86_64
--> Processing Dependency: font(:lang=en) for package: fontconfig-2.10.95-11.el7.x86_64
---> Package libX11-common.noarch 0:1.6.5-1.el7 will be installed
---> Package libXau-devel.x86_64 0:1.0.8-2.1.el7 will be installed
--> Processing Dependency: libXau = 1.0.8-2.1.el7 for package: libXau-devel-1.0.8-2.1.el7.x86_64
--> Processing Dependency: libXau.so.6()(64bit) for package: libXau-devel-1.0.8-2.1.el7.x86_64
---> Package libXrender.x86_64 0:0.9.10-1.el7 will be installed
---> Package libxcb.x86_64 0:1.12-1.el7 will be installed
--> Running transaction check
---> Package fontpackages-filesystem.noarch 0:1.44-8.el7 will be installed
---> Package libXau.x86_64 0:1.0.8-2.1.el7 will be installed
---> Package lyx-fonts.noarch 0:2.2.3-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================
 Package                       Arch         Version                  Repository     Size
=========================================================================================
Installing:
 freetype-devel                x86_64       2.4.11-15.el7            base          356 k
 lcms2-devel                   x86_64       2.6-3.el7                base          2.0 M
 libffi-devel                  x86_64       3.0.13-18.el7            base           23 k
 libjpeg-turbo-devel           x86_64       1.2.90-5.el7             base           98 k
 libtiff-devel                 x86_64       4.0.3-27.el7_3           base          473 k
 libwebp-devel                 x86_64       0.3.0-7.el7              base           23 k
 libzip-devel                  x86_64       0.10.1-8.el7             base           77 k
 mariadb-devel                 x86_64       1:5.5.56-2.el7           base          752 k
 openldap-devel                x86_64       2.4.44-15.el7_5          updates       803 k
 sshpass                       x86_64       1.06-2.el7               extras         21 k
 tcl-devel                     x86_64       1:8.5.13-8.el7           base          165 k
 tk-devel                      x86_64       1:8.5.13-6.el7           base          488 k
Installing for dependencies:
 cyrus-sasl                    x86_64       2.1.26-23.el7            base           88 k
 cyrus-sasl-devel              x86_64       2.1.26-23.el7            base          310 k
 expat-devel                   x86_64       2.1.0-10.el7_3           base           57 k
 fontconfig                    x86_64       2.10.95-11.el7           base          229 k
 fontconfig-devel              x86_64       2.10.95-11.el7           base          128 k
 fontpackages-filesystem       noarch       1.44-8.el7               base          9.9 k
 jbigkit-libs                  x86_64       2.0-11.el7               base           46 k
 lcms2                         x86_64       2.6-3.el7                base          150 k
 libX11                        x86_64       1.6.5-1.el7              base          606 k
 libX11-common                 noarch       1.6.5-1.el7              base          164 k
 libX11-devel                  x86_64       1.6.5-1.el7              base          980 k
 libXau                        x86_64       1.0.8-2.1.el7            base           29 k
 libXau-devel                  x86_64       1.0.8-2.1.el7            base           14 k
 libXft                        x86_64       2.3.2-2.el7              base           58 k
 libXft-devel                  x86_64       2.3.2-2.el7              base           19 k
 libXrender                    x86_64       0.9.10-1.el7             base           26 k
 libXrender-devel              x86_64       0.9.10-1.el7             base           17 k
 libjpeg-turbo                 x86_64       1.2.90-5.el7             base          134 k
 libtiff                       x86_64       4.0.3-27.el7_3           base          170 k
 libwebp                       x86_64       0.3.0-7.el7              base          170 k
 libxcb                        x86_64       1.12-1.el7               base          211 k
 libxcb-devel                  x86_64       1.12-1.el7               base          1.0 M
 libzip                        x86_64       0.10.1-8.el7             base           48 k
 lyx-fonts                     noarch       2.2.3-1.el7              epel          159 k
 tcl                           x86_64       1:8.5.13-8.el7           base          1.9 M
 tk                            x86_64       1:8.5.13-6.el7           base          1.4 M
 xorg-x11-proto-devel          noarch       7.7-20.el7               base          284 k
Updating for dependencies:
 openldap                      x86_64       2.4.44-15.el7_5          updates       355 k

Transaction Summary
=========================================================================================
Install  12 Packages (+27 Dependent packages)
Upgrade              (  1 Dependent package)

Total download size: 14 M
Downloading packages:
Delta RPMs reduced 355 k of updates to 85 k (76% saved)
(1/40): openldap-2.4.44-13.el7_2.4.44-15.el7_5.x86_64.drpm        |  85 kB  00:00:00     
(2/40): cyrus-sasl-2.1.26-23.el7.x86_64.rpm                       |  88 kB  00:00:00     
(3/40): cyrus-sasl-devel-2.1.26-23.el7.x86_64.rpm                 | 310 kB  00:00:00     
(4/40): fontpackages-filesystem-1.44-8.el7.noarch.rpm             | 9.9 kB  00:00:00     
/usr/share/doc/openldap-2.4.44/ANNOUNCEMENT: No such file or directory
cannot reconstruct rpm from disk files
(5/40): jbigkit-libs-2.0-11.el7.x86_64.rpm                        |  46 kB  00:00:00     
(6/40): fontconfig-2.10.95-11.el7.x86_64.rpm                      | 229 kB  00:00:00     
(7/40): freetype-devel-2.4.11-15.el7.x86_64.rpm                   | 356 kB  00:00:00     
(8/40): lcms2-2.6-3.el7.x86_64.rpm                                | 150 kB  00:00:00     
(9/40): libX11-1.6.5-1.el7.x86_64.rpm                             | 606 kB  00:00:00     
(10/40): libX11-common-1.6.5-1.el7.noarch.rpm                     | 164 kB  00:00:00     
(11/40): libXau-1.0.8-2.1.el7.x86_64.rpm                          |  29 kB  00:00:00     
(12/40): libX11-devel-1.6.5-1.el7.x86_64.rpm                      | 980 kB  00:00:00     
(13/40): libXau-devel-1.0.8-2.1.el7.x86_64.rpm                    |  14 kB  00:00:00     
(14/40): libXft-2.3.2-2.el7.x86_64.rpm                            |  58 kB  00:00:00     
(15/40): libXft-devel-2.3.2-2.el7.x86_64.rpm                      |  19 kB  00:00:00     
(16/40): libXrender-0.9.10-1.el7.x86_64.rpm                       |  26 kB  00:00:00     
(17/40): libXrender-devel-0.9.10-1.el7.x86_64.rpm                 |  17 kB  00:00:00     
(18/40): libffi-devel-3.0.13-18.el7.x86_64.rpm                    |  23 kB  00:00:00     
(19/40): fontconfig-devel-2.10.95-11.el7.x86_64.rpm               | 128 kB  00:00:01     
(20/40): libjpeg-turbo-devel-1.2.90-5.el7.x86_64.rpm              |  98 kB  00:00:00     
(21/40): libjpeg-turbo-1.2.90-5.el7.x86_64.rpm                    | 134 kB  00:00:00     
(22/40): libtiff-devel-4.0.3-27.el7_3.x86_64.rpm                  | 473 kB  00:00:00     
(23/40): libwebp-devel-0.3.0-7.el7.x86_64.rpm                     |  23 kB  00:00:00     
(24/40): libwebp-0.3.0-7.el7.x86_64.rpm                           | 170 kB  00:00:00     
(25/40): libxcb-1.12-1.el7.x86_64.rpm                             | 211 kB  00:00:00     
(26/40): libtiff-4.0.3-27.el7_3.x86_64.rpm                        | 170 kB  00:00:00     
(27/40): libzip-0.10.1-8.el7.x86_64.rpm                           |  48 kB  00:00:00     
(28/40): libzip-devel-0.10.1-8.el7.x86_64.rpm                     |  77 kB  00:00:00     
(29/40): mariadb-devel-5.5.56-2.el7.x86_64.rpm                    | 752 kB  00:00:00     
(30/40): libxcb-devel-1.12-1.el7.x86_64.rpm                       | 1.0 MB  00:00:00     
(31/40): sshpass-1.06-2.el7.x86_64.rpm                            |  21 kB  00:00:00     
(32/40): openldap-devel-2.4.44-15.el7_5.x86_64.rpm                | 803 kB  00:00:01     
(33/40): tcl-devel-8.5.13-8.el7.x86_64.rpm                        | 165 kB  00:00:00     
(34/40): tcl-8.5.13-8.el7.x86_64.rpm                              | 1.9 MB  00:00:01     
(35/40): tk-devel-8.5.13-6.el7.x86_64.rpm                         | 488 kB  00:00:00     
(36/40): tk-8.5.13-6.el7.x86_64.rpm                               | 1.4 MB  00:00:00     
(37/40): xorg-x11-proto-devel-7.7-20.el7.noarch.rpm               | 284 kB  00:00:00     
(38/40): expat-devel-2.1.0-10.el7_3.x86_64.rpm                    |  57 kB  00:00:06     
(39/40): lcms2-devel-2.6-3.el7.x86_64.rpm                         | 2.0 MB  00:00:06     
warning: /var/cache/yum/x86_64/7/epel/packages/lyx-fonts-2.2.3-1.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for lyx-fonts-2.2.3-1.el7.noarch.rpm is not installed
(40/40): lyx-fonts-2.2.3-1.el7.noarch.rpm                         | 159 kB  00:00:04     
Some delta RPMs failed to download or rebuild. Retrying..
openldap-2.4.44-15.el7_5.x86_64.rpm                               | 355 kB  00:00:00     
-----------------------------------------------------------------------------------------
Total                                                       1.7 MB/s |  14 MB  00:08     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-11.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : xorg-x11-proto-devel-7.7-20.el7.noarch                               1/41 
  Updating   : openldap-2.4.44-15.el7_5.x86_64                                      2/41 
  Installing : libXau-1.0.8-2.1.el7.x86_64                                          3/41 
  Installing : libxcb-1.12-1.el7.x86_64                                             4/41 
  Installing : freetype-devel-2.4.11-15.el7.x86_64                                  5/41 
  Installing : fontpackages-filesystem-1.44-8.el7.noarch                            6/41 
  Installing : libjpeg-turbo-1.2.90-5.el7.x86_64                                    7/41 
  Installing : 1:tcl-8.5.13-8.el7.x86_64                                            8/41 
  Installing : 1:tcl-devel-8.5.13-8.el7.x86_64                                      9/41 
  Installing : lyx-fonts-2.2.3-1.el7.noarch                                        10/41 
  Installing : fontconfig-2.10.95-11.el7.x86_64                                    11/41 
  Installing : libXau-devel-1.0.8-2.1.el7.x86_64                                   12/41 
  Installing : libxcb-devel-1.12-1.el7.x86_64                                      13/41 
  Installing : cyrus-sasl-2.1.26-23.el7.x86_64                                     14/41 
  Installing : cyrus-sasl-devel-2.1.26-23.el7.x86_64                               15/41 
  Installing : libzip-0.10.1-8.el7.x86_64                                          16/41 
  Installing : lcms2-2.6-3.el7.x86_64                                              17/41 
  Installing : libX11-common-1.6.5-1.el7.noarch                                    18/41 
  Installing : libX11-1.6.5-1.el7.x86_64                                           19/41 
  Installing : libXrender-0.9.10-1.el7.x86_64                                      20/41 
  Installing : libXft-2.3.2-2.el7.x86_64                                           21/41 
  Installing : libX11-devel-1.6.5-1.el7.x86_64                                     22/41 
  Installing : libXrender-devel-0.9.10-1.el7.x86_64                                23/41 
  Installing : 1:tk-8.5.13-6.el7.x86_64                                            24/41 
  Installing : jbigkit-libs-2.0-11.el7.x86_64                                      25/41 
  Installing : libtiff-4.0.3-27.el7_3.x86_64                                       26/41 
  Installing : libwebp-0.3.0-7.el7.x86_64                                          27/41 
  Installing : expat-devel-2.1.0-10.el7_3.x86_64                                   28/41 
  Installing : fontconfig-devel-2.10.95-11.el7.x86_64                              29/41 
  Installing : libXft-devel-2.3.2-2.el7.x86_64                                     30/41 
  Installing : 1:tk-devel-8.5.13-6.el7.x86_64                                      31/41 
  Installing : libwebp-devel-0.3.0-7.el7.x86_64                                    32/41 
  Installing : libtiff-devel-4.0.3-27.el7_3.x86_64                                 33/41 
  Installing : lcms2-devel-2.6-3.el7.x86_64                                        34/41 
  Installing : libzip-devel-0.10.1-8.el7.x86_64                                    35/41 
  Installing : openldap-devel-2.4.44-15.el7_5.x86_64                               36/41 
  Installing : libjpeg-turbo-devel-1.2.90-5.el7.x86_64                             37/41 
  Installing : libffi-devel-3.0.13-18.el7.x86_64                                   38/41 
  Installing : sshpass-1.06-2.el7.x86_64                                           39/41 
  Installing : 1:mariadb-devel-5.5.56-2.el7.x86_64                                 40/41 
  Cleanup    : openldap-2.4.44-13.el7.x86_64                                       41/41 
  Verifying  : libXft-devel-2.3.2-2.el7.x86_64                                      1/41 
  Verifying  : libX11-1.6.5-1.el7.x86_64                                            2/41 
  Verifying  : 1:tcl-8.5.13-8.el7.x86_64                                            3/41 
  Verifying  : libtiff-4.0.3-27.el7_3.x86_64                                        4/41 
  Verifying  : libXrender-devel-0.9.10-1.el7.x86_64                                 5/41 
  Verifying  : libjpeg-turbo-1.2.90-5.el7.x86_64                                    6/41 
  Verifying  : libXrender-0.9.10-1.el7.x86_64                                       7/41 
  Verifying  : lcms2-devel-2.6-3.el7.x86_64                                         8/41 
  Verifying  : 1:mariadb-devel-5.5.56-2.el7.x86_64                                  9/41 
  Verifying  : xorg-x11-proto-devel-7.7-20.el7.noarch                              10/41 
  Verifying  : sshpass-1.06-2.el7.x86_64                                           11/41 
  Verifying  : fontpackages-filesystem-1.44-8.el7.noarch                           12/41 
  Verifying  : expat-devel-2.1.0-10.el7_3.x86_64                                   13/41 
  Verifying  : libwebp-devel-0.3.0-7.el7.x86_64                                    14/41 
  Verifying  : libffi-devel-3.0.13-18.el7.x86_64                                   15/41 
  Verifying  : libjpeg-turbo-devel-1.2.90-5.el7.x86_64                             16/41 
  Verifying  : libtiff-devel-4.0.3-27.el7_3.x86_64                                 17/41 
  Verifying  : 1:tk-devel-8.5.13-6.el7.x86_64                                      18/41 
  Verifying  : 1:tk-8.5.13-6.el7.x86_64                                            19/41 
  Verifying  : cyrus-sasl-devel-2.1.26-23.el7.x86_64                               20/41 
  Verifying  : fontconfig-devel-2.10.95-11.el7.x86_64                              21/41 
  Verifying  : libwebp-0.3.0-7.el7.x86_64                                          22/41 
  Verifying  : libxcb-1.12-1.el7.x86_64                                            23/41 
  Verifying  : libXft-2.3.2-2.el7.x86_64                                           24/41 
  Verifying  : openldap-devel-2.4.44-15.el7_5.x86_64                               25/41 
  Verifying  : libxcb-devel-1.12-1.el7.x86_64                                      26/41 
  Verifying  : jbigkit-libs-2.0-11.el7.x86_64                                      27/41 
  Verifying  : libX11-devel-1.6.5-1.el7.x86_64                                     28/41 
  Verifying  : libX11-common-1.6.5-1.el7.noarch                                    29/41 
  Verifying  : freetype-devel-2.4.11-15.el7.x86_64                                 30/41 
  Verifying  : libzip-devel-0.10.1-8.el7.x86_64                                    31/41 
  Verifying  : lcms2-2.6-3.el7.x86_64                                              32/41 
  Verifying  : libXau-1.0.8-2.1.el7.x86_64                                         33/41 
  Verifying  : fontconfig-2.10.95-11.el7.x86_64                                    34/41 
  Verifying  : libzip-0.10.1-8.el7.x86_64                                          35/41 
  Verifying  : cyrus-sasl-2.1.26-23.el7.x86_64                                     36/41 
  Verifying  : lyx-fonts-2.2.3-1.el7.noarch                                        37/41 
  Verifying  : openldap-2.4.44-15.el7_5.x86_64                                     38/41 
  Verifying  : 1:tcl-devel-8.5.13-8.el7.x86_64                                     39/41 
  Verifying  : libXau-devel-1.0.8-2.1.el7.x86_64                                   40/41 
  Verifying  : openldap-2.4.44-13.el7.x86_64                                       41/41 

Installed:
  freetype-devel.x86_64 0:2.4.11-15.el7      lcms2-devel.x86_64 0:2.6-3.el7              
  libffi-devel.x86_64 0:3.0.13-18.el7        libjpeg-turbo-devel.x86_64 0:1.2.90-5.el7   
  libtiff-devel.x86_64 0:4.0.3-27.el7_3      libwebp-devel.x86_64 0:0.3.0-7.el7          
  libzip-devel.x86_64 0:0.10.1-8.el7         mariadb-devel.x86_64 1:5.5.56-2.el7         
  openldap-devel.x86_64 0:2.4.44-15.el7_5    sshpass.x86_64 0:1.06-2.el7                 
  tcl-devel.x86_64 1:8.5.13-8.el7            tk-devel.x86_64 1:8.5.13-6.el7              

Dependency Installed:
  cyrus-sasl.x86_64 0:2.1.26-23.el7         cyrus-sasl-devel.x86_64 0:2.1.26-23.el7     
  expat-devel.x86_64 0:2.1.0-10.el7_3       fontconfig.x86_64 0:2.10.95-11.el7          
  fontconfig-devel.x86_64 0:2.10.95-11.el7  fontpackages-filesystem.noarch 0:1.44-8.el7 
  jbigkit-libs.x86_64 0:2.0-11.el7          lcms2.x86_64 0:2.6-3.el7                    
  libX11.x86_64 0:1.6.5-1.el7               libX11-common.noarch 0:1.6.5-1.el7          
  libX11-devel.x86_64 0:1.6.5-1.el7         libXau.x86_64 0:1.0.8-2.1.el7               
  libXau-devel.x86_64 0:1.0.8-2.1.el7       libXft.x86_64 0:2.3.2-2.el7                 
  libXft-devel.x86_64 0:2.3.2-2.el7         libXrender.x86_64 0:0.9.10-1.el7            
  libXrender-devel.x86_64 0:0.9.10-1.el7    libjpeg-turbo.x86_64 0:1.2.90-5.el7         
  libtiff.x86_64 0:4.0.3-27.el7_3           libwebp.x86_64 0:0.3.0-7.el7                
  libxcb.x86_64 0:1.12-1.el7                libxcb-devel.x86_64 0:1.12-1.el7            
  libzip.x86_64 0:0.10.1-8.el7              lyx-fonts.noarch 0:2.2.3-1.el7              
  tcl.x86_64 1:8.5.13-8.el7                 tk.x86_64 1:8.5.13-6.el7                    
  xorg-x11-proto-devel.noarch 0:7.7-20.el7 

Dependency Updated:
  openldap.x86_64 0:2.4.44-15.el7_5                                                      

Complete!
[root@h165 requirements]#
~~~

## 安装 Python 库依赖 ##


~~~
(py3) [root@h165 requirements]# pwd
/opt/jumpserver/requirements
(py3) [root@h165 requirements]# ls
deb_requirements.txt  mac_requirements.txt  rpm_requirements.txt
issues.txt            requirements.txt
(py3) [root@h165 requirements]# cat rpm_requirements.txt 
libtiff-devel libjpeg-devel libzip-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel sshpass openldap-devel mysql-devel libffi-devel openssh-clients
(py3) [root@h165 requirements]# cat requirements.txt 
amqp==2.1.4
ansible==2.4.2.0
asn1crypto==0.24.0
bcrypt==3.1.4
billiard==3.5.0.3
boto3==1.6.5
botocore==1.9.5
celery==4.1.0
certifi==2018.1.18
cffi==1.11.2
chardet==3.0.4
configparser==3.5.0
coreapi==2.3.3
coreschema==0.0.4
cryptography==2.1.4
decorator==4.1.2
Django==1.11
django-auth-ldap==1.3.0
django-bootstrap3==9.1.0
django-celery-beat==1.1.1
django-filter==1.1.0
django-formtools==2.1
django-ranged-response==0.2.0
django-redis-cache==1.7.1
django-rest-swagger==2.1.2
django-simple-captcha==0.5.6
djangorestframework==3.7.3
djangorestframework-bulk==0.2.1
docutils==0.14
ecdsa==0.13
elasticsearch==6.1.1
enum-compat==0.0.2
ephem==3.7.6.0
eventlet==0.22.1
ForgeryPy==0.1
greenlet==0.4.12
gunicorn==19.7.1
idna==2.6
itsdangerous==0.24
itypes==1.1.0
Jinja2==2.10
jmespath==0.9.3
kombu==4.0.2
ldap3==2.4
MarkupSafe==1.0
mysqlclient==1.3.12
olefile==0.44
openapi-codec==1.3.2
paramiko==2.4.0
passlib==1.7.1
Pillow==4.3.0
pyasn1==0.4.2
pycparser==2.18
pycrypto==2.6.1
pyldap==2.4.45
pyotp==2.2.6
PyNaCl==1.2.1
python-dateutil==2.6.1
python-gssapi==0.6.4
pytz==2018.3
PyYAML==3.12
redis==2.10.6
requests==2.18.4
jms-storage==0.0.18
s3transfer==0.1.13
simplejson==3.13.2
six==1.11.0
sshpubkeys==2.2.0
uritemplate==3.0.0
urllib3==1.22
vine==1.1.4
(py3) [root@h165 requirements]# pip install -r requirements.txt 
Collecting amqp==2.1.4 (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7e/4b/ac7afb11b57f237e3c1c64b5408c5d229bf5d4b42af6cb6e683c7690ca4f/amqp-2.1.4-py2.py3-none-any.whl (49kB)
    100% |████████████████████████████████| 51kB 123kB/s 
Collecting ansible==2.4.2.0 (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/4f/65/ae3ad8589c38f9e04ebc8a824c2880eb4f9e603a1f62b5f5a3f938e524b0/ansible-2.4.2.0.tar.gz (6.5MB)
    100% |████████████████████████████████| 6.5MB 205kB/s 
Collecting asn1crypto==0.24.0 (from -r requirements.txt (line 3))
  Downloading https://files.pythonhosted.org/packages/ea/cd/35485615f45f30a510576f1a56d1e0a7ad7bd8ab5ed7cdc600ef7cd06222/asn1crypto-0.24.0-py2.py3-none-any.whl (101kB)
    100% |████████████████████████████████| 102kB 62kB/s 
Collecting bcrypt==3.1.4 (from -r requirements.txt (line 4))
  Downloading https://files.pythonhosted.org/packages/b8/09/905ec939994e2c49dcffff72f823802557f166b3815ea54c1db3671eed42/bcrypt-3.1.4-cp36-cp36m-manylinux1_x86_64.whl (54kB)
    100% |████████████████████████████████| 61kB 2.4MB/s 
Collecting billiard==3.5.0.3 (from -r requirements.txt (line 5))
  Downloading https://files.pythonhosted.org/packages/82/55/76f4e786141b7174926cdffa7a155aeea316b729118fb48ec548f3c6754f/billiard-3.5.0.3-py3-none-any.whl (89kB)
    100% |████████████████████████████████| 92kB 110kB/s 
Collecting boto3==1.6.5 (from -r requirements.txt (line 6))
  Downloading https://files.pythonhosted.org/packages/ba/3c/ba15b7165982e39f411e2b37f12667f508c7d623f339ec0ac5482d2f8502/boto3-1.6.5-py2.py3-none-any.whl (128kB)
    100% |████████████████████████████████| 133kB 1.5MB/s 
Collecting botocore==1.9.5 (from -r requirements.txt (line 7))
  Downloading https://files.pythonhosted.org/packages/54/b9/4c04a9b75f36ec467a630af705d38f75e65805ab30abcd83f4428686bdf0/botocore-1.9.5-py2.py3-none-any.whl (4.1MB)
    100% |████████████████████████████████| 4.1MB 284kB/s 
Collecting celery==4.1.0 (from -r requirements.txt (line 8))
  Downloading https://files.pythonhosted.org/packages/22/9b/88ef5cc7edf5d43215f383ae0a2b1cdeb33f5f07886386c7e4691b2eba0c/celery-4.1.0-py2.py3-none-any.whl (400kB)
    100% |████████████████████████████████| 409kB 910kB/s 
Collecting certifi==2018.1.18 (from -r requirements.txt (line 9))
  Downloading https://files.pythonhosted.org/packages/fa/53/0a5562e2b96749e99a3d55d8c7df91c9e4d8c39a9da1f1a49ac9e4f4b39f/certifi-2018.1.18-py2.py3-none-any.whl (151kB)
    100% |████████████████████████████████| 153kB 166kB/s 
Collecting cffi==1.11.2 (from -r requirements.txt (line 10))
  Downloading https://files.pythonhosted.org/packages/dc/1e/b383fde1f0a14b6ef5a60f71797c778ea1ef8bb34b726cb57061c0542c58/cffi-1.11.2-cp36-cp36m-manylinux1_x86_64.whl (419kB)
    100% |████████████████████████████████| 430kB 611kB/s 
Collecting chardet==3.0.4 (from -r requirements.txt (line 11))
  Downloading https://files.pythonhosted.org/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl (133kB)
    100% |████████████████████████████████| 143kB 661kB/s 
Collecting configparser==3.5.0 (from -r requirements.txt (line 12))
  Downloading https://files.pythonhosted.org/packages/7c/69/c2ce7e91c89dc073eb1aa74c0621c3eefbffe8216b3f9af9d3885265c01c/configparser-3.5.0.tar.gz
Collecting coreapi==2.3.3 (from -r requirements.txt (line 13))
  Downloading https://files.pythonhosted.org/packages/fc/3a/9dedaad22962770edd334222f2b3c3e7ad5e1c8cab1d6a7992c30329e2e5/coreapi-2.3.3-py2.py3-none-any.whl
Collecting coreschema==0.0.4 (from -r requirements.txt (line 14))
  Downloading https://files.pythonhosted.org/packages/93/08/1d105a70104e078718421e6c555b8b293259e7fc92f7e9a04869947f198f/coreschema-0.0.4.tar.gz
Collecting cryptography==2.1.4 (from -r requirements.txt (line 15))
  Downloading https://files.pythonhosted.org/packages/4e/e0/4959b48f04c879414972048fe2bedc96825e39c5413ae241c230fba58783/cryptography-2.1.4-cp36-cp36m-manylinux1_x86_64.whl (2.2MB)
    100% |████████████████████████████████| 2.2MB 344kB/s 
Collecting decorator==4.1.2 (from -r requirements.txt (line 16))
  Downloading https://files.pythonhosted.org/packages/a1/4e/c42167ba5c3192bed633726d39d7896cc55d4fa3ec4a1fb60cd3a53fc4c7/decorator-4.1.2-py2.py3-none-any.whl
Collecting Django==1.11 (from -r requirements.txt (line 17))
  Downloading https://files.pythonhosted.org/packages/47/a6/078ebcbd49b19e22fd560a2348cfc5cec9e5dcfe3c4fad8e64c9865135bb/Django-1.11-py2.py3-none-any.whl (6.9MB)
    100% |████████████████████████████████| 6.9MB 198kB/s 
Collecting django-auth-ldap==1.3.0 (from -r requirements.txt (line 18))
  Downloading https://files.pythonhosted.org/packages/df/3b/aac713e0f351aa2672f74017e0e23e175515fac1342b0dd17431da44716a/django_auth_ldap-1.3.0-py2.py3-none-any.whl
Collecting django-bootstrap3==9.1.0 (from -r requirements.txt (line 19))
  Downloading https://files.pythonhosted.org/packages/ca/b8/d91efa12368f28aa95cd3709c4f25f79b2c9e8735c1f87a69b45977375b1/django-bootstrap3-9.1.0.tar.gz
Collecting django-celery-beat==1.1.1 (from -r requirements.txt (line 20))
  Downloading https://files.pythonhosted.org/packages/45/4b/4d9dfd63ebd119e7340526d023248896626e9454edba981b331858b1b5bf/django_celery_beat-1.1.1-py2.py3-none-any.whl
Collecting django-filter==1.1.0 (from -r requirements.txt (line 21))
  Downloading https://files.pythonhosted.org/packages/ee/99/eb6f20b0ca4e2800279963599971e70c71767b9d151f44fcbcd1caa19f32/django_filter-1.1.0-py2.py3-none-any.whl (45kB)
    100% |████████████████████████████████| 51kB 2.6MB/s 
Collecting django-formtools==2.1 (from -r requirements.txt (line 22))
  Downloading https://files.pythonhosted.org/packages/97/3f/b8e04c41c028d5cdad651393abea1f686d846c717d8ab5d5ebe2974f711c/django_formtools-2.1-py2.py3-none-any.whl (132kB)
    100% |████████████████████████████████| 133kB 1.8MB/s 
Collecting django-ranged-response==0.2.0 (from -r requirements.txt (line 23))
  Downloading https://files.pythonhosted.org/packages/70/e3/9372fcdca8e9c3205e7979528ccd1a14354a9a24d38efff11c1846ff8bf1/django-ranged-response-0.2.0.tar.gz
Collecting django-redis-cache==1.7.1 (from -r requirements.txt (line 24))
  Downloading https://files.pythonhosted.org/packages/e9/50/13d72d9d3a3c8ffd3c66dc91f2918f9f6bc8454c3d869dd86576e2e02aaa/django-redis-cache-1.7.1.tar.gz
Collecting django-rest-swagger==2.1.2 (from -r requirements.txt (line 25))
  Downloading https://files.pythonhosted.org/packages/2c/12/28d0677756283d913f8371ad7590e16f5e6c8444044f5b1b1586f956e610/django_rest_swagger-2.1.2-py2.py3-none-any.whl (747kB)
    100% |████████████████████████████████| 757kB 670kB/s 
Collecting django-simple-captcha==0.5.6 (from -r requirements.txt (line 26))
  Downloading https://files.pythonhosted.org/packages/4d/46/44aff307e370e873ebb44dd8e9a1bcde10ddd1e55779361a5654d273d939/django-simple-captcha-0.5.6.zip (226kB)
    100% |████████████████████████████████| 235kB 2.4MB/s 
Collecting djangorestframework==3.7.3 (from -r requirements.txt (line 27))
  Downloading https://files.pythonhosted.org/packages/f5/2c/3c3c81b8ecd60952c20ccd5fc1d19c0b22731c80c75649dd2e3fef361aaf/djangorestframework-3.7.3-py2.py3-none-any.whl (1.5MB)
    100% |████████████████████████████████| 1.5MB 490kB/s 
Collecting djangorestframework-bulk==0.2.1 (from -r requirements.txt (line 28))
  Downloading https://files.pythonhosted.org/packages/32/f8/79fd8c1919fd1e033eedc04fea3396e978eb20c9cda49a0c538e9c5d8127/djangorestframework-bulk-0.2.1.tar.gz
Collecting docutils==0.14 (from -r requirements.txt (line 29))
  Downloading https://files.pythonhosted.org/packages/36/fa/08e9e6e0e3cbd1d362c3bbee8d01d0aedb2155c4ac112b19ef3cae8eed8d/docutils-0.14-py3-none-any.whl (543kB)
    100% |████████████████████████████████| 552kB 618kB/s 
Collecting ecdsa==0.13 (from -r requirements.txt (line 30))
  Downloading https://files.pythonhosted.org/packages/63/f4/73669d51825516ce8c43b816c0a6b64cd6eb71d08b99820c00792cb42222/ecdsa-0.13-py2.py3-none-any.whl (86kB)
    100% |████████████████████████████████| 92kB 1.1MB/s 
Collecting elasticsearch==6.1.1 (from -r requirements.txt (line 31))
  Downloading https://files.pythonhosted.org/packages/67/15/80db00582d1d6286c5c8f0e18e444481e0fc2bc1fa2391935f10358c5f2d/elasticsearch-6.1.1-py2.py3-none-any.whl (59kB)
    100% |████████████████████████████████| 61kB 2.2MB/s 
Collecting enum-compat==0.0.2 (from -r requirements.txt (line 32))
  Downloading https://files.pythonhosted.org/packages/95/6e/26bdcba28b66126f66cf3e4cd03bcd63f7ae330d29ee68b1f6b623550bfa/enum-compat-0.0.2.tar.gz
Collecting ephem==3.7.6.0 (from -r requirements.txt (line 33))
  Downloading https://files.pythonhosted.org/packages/c3/2c/9e1a815add6c222a0d4bf7c644e095471a934a39bc90c201f9550a8f7f14/ephem-3.7.6.0.tar.gz (739kB)
    100% |████████████████████████████████| 747kB 556kB/s 
Collecting eventlet==0.22.1 (from -r requirements.txt (line 34))
  Downloading https://files.pythonhosted.org/packages/61/1a/d1ff6e4f1dc652dfdda4a674f807c842eaa15f1ed9b76938a3be313bbac9/eventlet-0.22.1-py2.py3-none-any.whl (409kB)
    100% |████████████████████████████████| 409kB 558kB/s 
Collecting ForgeryPy==0.1 (from -r requirements.txt (line 35))
  Downloading https://files.pythonhosted.org/packages/de/66/8a29d7163b528d2d5c3ccc98e621959723b7eeb010e38c11f1404313e2b7/ForgeryPy-0.1.tar.gz
Collecting greenlet==0.4.12 (from -r requirements.txt (line 36))
  Downloading https://files.pythonhosted.org/packages/20/ea/e47c2fff6e91b20c05107411fa25fb93e66bd76ecd27f04e2224e7806f41/greenlet-0.4.12-cp36-cp36m-manylinux1_x86_64.whl (42kB)
    100% |████████████████████████████████| 51kB 2.4MB/s 
Collecting gunicorn==19.7.1 (from -r requirements.txt (line 37))
  Downloading https://files.pythonhosted.org/packages/64/32/becbd4089a4c06f0f9f538a76e9fe0b19a08f010bcb47dcdbfbc640cdf7d/gunicorn-19.7.1-py2.py3-none-any.whl (111kB)
    100% |████████████████████████████████| 112kB 605kB/s 
Collecting idna==2.6 (from -r requirements.txt (line 38))
  Downloading https://files.pythonhosted.org/packages/27/cc/6dd9a3869f15c2edfab863b992838277279ce92663d334df9ecf5106f5c6/idna-2.6-py2.py3-none-any.whl (56kB)
    100% |████████████████████████████████| 61kB 1.9MB/s 
Collecting itsdangerous==0.24 (from -r requirements.txt (line 39))
  Downloading https://files.pythonhosted.org/packages/dc/b4/a60bcdba945c00f6d608d8975131ab3f25b22f2bcfe1dab221165194b2d4/itsdangerous-0.24.tar.gz (46kB)
    100% |████████████████████████████████| 51kB 1.2MB/s 
Collecting itypes==1.1.0 (from -r requirements.txt (line 40))
  Downloading https://files.pythonhosted.org/packages/d3/24/5e511590f95582efe64b8ad2f6dadd85c5563c9dcf40171ea5a70adbf5a9/itypes-1.1.0.tar.gz
Collecting Jinja2==2.10 (from -r requirements.txt (line 41))
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
    100% |████████████████████████████████| 133kB 486kB/s 
Collecting jmespath==0.9.3 (from -r requirements.txt (line 42))
  Downloading https://files.pythonhosted.org/packages/b7/31/05c8d001f7f87f0f07289a5fc0fc3832e9a57f2dbd4d3b0fee70e0d51365/jmespath-0.9.3-py2.py3-none-any.whl
Collecting kombu==4.0.2 (from -r requirements.txt (line 43))
  Downloading https://files.pythonhosted.org/packages/43/ac/567da3770c8f1d2c44262b485130d6a12efc1bdb4ba79f1f328e1015ecd0/kombu-4.0.2-py2.py3-none-any.whl (178kB)
    100% |████████████████████████████████| 184kB 428kB/s 
Collecting ldap3==2.4 (from -r requirements.txt (line 44))
  Downloading https://files.pythonhosted.org/packages/5e/8b/e10470924ffa0ddc4a8ea9733facb474aebbae188eef16157279810e18a6/ldap3-2.4-py2.py3-none-any.whl (373kB)
    100% |████████████████████████████████| 378kB 602kB/s 
Collecting MarkupSafe==1.0 (from -r requirements.txt (line 45))
  Downloading https://files.pythonhosted.org/packages/4d/de/32d741db316d8fdb7680822dd37001ef7a448255de9699ab4bfcbdf4172b/MarkupSafe-1.0.tar.gz
Collecting mysqlclient==1.3.12 (from -r requirements.txt (line 46))
  Downloading https://files.pythonhosted.org/packages/6f/86/bad31f1c1bb0cc99e88ca2adb7cb5c71f7a6540c1bb001480513de76a931/mysqlclient-1.3.12.tar.gz (89kB)
    100% |████████████████████████████████| 92kB 974kB/s 
Collecting olefile==0.44 (from -r requirements.txt (line 47))
  Downloading https://files.pythonhosted.org/packages/35/17/c15d41d5a8f8b98cc3df25eb00c5cee76193114c78e5674df6ef4ac92647/olefile-0.44.zip (74kB)
    100% |████████████████████████████████| 81kB 1.1MB/s 
Collecting openapi-codec==1.3.2 (from -r requirements.txt (line 48))
  Downloading https://files.pythonhosted.org/packages/78/e5/e0b5aba60c645dde952bc8a9df1f2b0bef27302908839b0a29284c9245d4/openapi-codec-1.3.2.tar.gz
Collecting paramiko==2.4.0 (from -r requirements.txt (line 49))
  Downloading https://files.pythonhosted.org/packages/be/9f/2b899b028aec1f3973253c0cf8dda6fbff65f4930f7ebedc43033e9f1b18/paramiko-2.4.0-py2.py3-none-any.whl (192kB)
    100% |████████████████████████████████| 194kB 440kB/s 
Collecting passlib==1.7.1 (from -r requirements.txt (line 50))
  Downloading https://files.pythonhosted.org/packages/ee/a7/d6d238d927df355d4e4e000670342ca4705a72f0bf694027cf67d9bcf5af/passlib-1.7.1-py2.py3-none-any.whl (498kB)
    100% |████████████████████████████████| 501kB 523kB/s 
Collecting Pillow==4.3.0 (from -r requirements.txt (line 51))
  Downloading https://files.pythonhosted.org/packages/3c/5c/44a8f05da34cbb495e5330825c2204b9fa761357c87bc0bc1785b1d76e41/Pillow-4.3.0-cp36-cp36m-manylinux1_x86_64.whl (5.8MB)
    100% |████████████████████████████████| 5.8MB 212kB/s 
Collecting pyasn1==0.4.2 (from -r requirements.txt (line 52))
  Downloading https://files.pythonhosted.org/packages/ba/fe/02e3e2ee243966b143657fb8bd6bc97595841163b6d8c26820944acaec4d/pyasn1-0.4.2-py2.py3-none-any.whl (71kB)
    100% |████████████████████████████████| 71kB 3.2MB/s 
Collecting pycparser==2.18 (from -r requirements.txt (line 53))
  Downloading https://files.pythonhosted.org/packages/8c/2d/aad7f16146f4197a11f8e91fb81df177adcc2073d36a17b1491fd09df6ed/pycparser-2.18.tar.gz (245kB)
    100% |████████████████████████████████| 256kB 893kB/s 
Collecting pycrypto==2.6.1 (from -r requirements.txt (line 54))
  Downloading https://files.pythonhosted.org/packages/60/db/645aa9af249f059cc3a368b118de33889219e0362141e75d4eaf6f80f163/pycrypto-2.6.1.tar.gz (446kB)
    100% |████████████████████████████████| 450kB 861kB/s 
Collecting pyldap==2.4.45 (from -r requirements.txt (line 55))
  Downloading https://files.pythonhosted.org/packages/68/94/291eb5e4dbf804dd57125c8309aee5678caf470acbe077151d711e185034/pyldap-2.4.45.tar.gz (304kB)
    100% |████████████████████████████████| 307kB 870kB/s 
Collecting pyotp==2.2.6 (from -r requirements.txt (line 56))
  Downloading https://files.pythonhosted.org/packages/12/1f/0b27e24456a9bb89b418602f61cbcad4861b55cf872469643ed13b38ff9b/pyotp-2.2.6-py2.py3-none-any.whl
Collecting PyNaCl==1.2.1 (from -r requirements.txt (line 57))
  Downloading https://files.pythonhosted.org/packages/77/03/927e4cdbd821f929392608ddb2220a9548ce164c52047e90fadd20786fd8/PyNaCl-1.2.1-cp36-cp36m-manylinux1_x86_64.whl (692kB)
    100% |████████████████████████████████| 696kB 608kB/s 
Collecting python-dateutil==2.6.1 (from -r requirements.txt (line 58))
  Downloading https://files.pythonhosted.org/packages/4b/0d/7ed381ab4fe80b8ebf34411d14f253e1cf3e56e2820ffa1d8844b23859a2/python_dateutil-2.6.1-py2.py3-none-any.whl (194kB)
    100% |████████████████████████████████| 194kB 2.5MB/s 
Collecting python-gssapi==0.6.4 (from -r requirements.txt (line 59))
  Downloading https://files.pythonhosted.org/packages/a4/9e/648b4e85235097edcee561c986f7075cb1606be24c514cfcdd2930e35c5e/python-gssapi-0.6.4.tar.gz
Collecting pytz==2018.3 (from -r requirements.txt (line 60))
  Downloading https://files.pythonhosted.org/packages/3c/80/32e98784a8647880dedf1f6bf8e2c91b195fe18fdecc6767dcf5104598d6/pytz-2018.3-py2.py3-none-any.whl (509kB)
    100% |████████████████████████████████| 512kB 575kB/s 
Collecting PyYAML==3.12 (from -r requirements.txt (line 61))
  Downloading https://files.pythonhosted.org/packages/4a/85/db5a2df477072b2902b0eb892feb37d88ac635d36245a72a6a69b23b383a/PyYAML-3.12.tar.gz (253kB)
    100% |████████████████████████████████| 256kB 666kB/s 
Collecting redis==2.10.6 (from -r requirements.txt (line 62))
  Downloading https://files.pythonhosted.org/packages/3b/f6/7a76333cf0b9251ecf49efff635015171843d9b977e4ffcf59f9c4428052/redis-2.10.6-py2.py3-none-any.whl (64kB)
    100% |████████████████████████████████| 71kB 950kB/s 
Collecting requests==2.18.4 (from -r requirements.txt (line 63))
  Downloading https://files.pythonhosted.org/packages/49/df/50aa1999ab9bde74656c2919d9c0c085fd2b3775fd3eca826012bef76d8c/requests-2.18.4-py2.py3-none-any.whl (88kB)
    100% |████████████████████████████████| 92kB 1.7MB/s 
Collecting jms-storage==0.0.18 (from -r requirements.txt (line 64))
  Downloading https://files.pythonhosted.org/packages/db/74/1f9ae797c970c76bb5e1a959beedfa72ea50dbf954daa91f4ce957d9fa41/jms-storage-0.0.18.tar.gz
Collecting s3transfer==0.1.13 (from -r requirements.txt (line 65))
  Downloading https://files.pythonhosted.org/packages/d7/14/2a0004d487464d120c9fb85313a75cd3d71a7506955be458eebfe19a6b1d/s3transfer-0.1.13-py2.py3-none-any.whl (59kB)
    100% |████████████████████████████████| 61kB 2.0MB/s 
Collecting simplejson==3.13.2 (from -r requirements.txt (line 66))
  Downloading https://files.pythonhosted.org/packages/0d/3f/3a16847fe5c010110a8f54dd8fe7b091b4e22922def374fe1cce9c1cb7e9/simplejson-3.13.2.tar.gz (79kB)
    100% |████████████████████████████████| 81kB 919kB/s 
Collecting six==1.11.0 (from -r requirements.txt (line 67))
  Downloading https://files.pythonhosted.org/packages/67/4b/141a581104b1f6397bfa78ac9d43d8ad29a7ca43ea90a2d863fe3056e86a/six-1.11.0-py2.py3-none-any.whl
Collecting sshpubkeys==2.2.0 (from -r requirements.txt (line 68))
  Downloading https://files.pythonhosted.org/packages/a7/59/7012b9a50caf1085cdda138bb66c502759bc3950fc3270380a2981486441/sshpubkeys-2.2.0-py2.py3-none-any.whl
Collecting uritemplate==3.0.0 (from -r requirements.txt (line 69))
  Downloading https://files.pythonhosted.org/packages/e5/7d/9d5a640c4f8bf2c8b1afc015e9a9d8de32e13c9016dcc4b0ec03481fb396/uritemplate-3.0.0-py2.py3-none-any.whl
Collecting urllib3==1.22 (from -r requirements.txt (line 70))
  Downloading https://files.pythonhosted.org/packages/63/cb/6965947c13a94236f6d4b8223e21beb4d576dc72e8130bd7880f600839b8/urllib3-1.22-py2.py3-none-any.whl (132kB)
    100% |████████████████████████████████| 133kB 545kB/s 
Collecting vine==1.1.4 (from -r requirements.txt (line 71))
  Downloading https://files.pythonhosted.org/packages/10/50/5b1ebe42843c19f35edb15022ecae339fbec6db5b241a7a13c924dabf2a3/vine-1.1.4-py2.py3-none-any.whl
Requirement already satisfied: setuptools in /opt/py3/lib/python3.6/site-packages (from ansible==2.4.2.0->-r requirements.txt (line 2))
Collecting crcmod==1.7 (from jms-storage==0.0.18->-r requirements.txt (line 64))
  Downloading https://files.pythonhosted.org/packages/6b/b0/e595ce2a2527e169c3bcd6c33d2473c1918e0b7f6826a043ca1245dd4e5b/crcmod-1.7.tar.gz (89kB)
    100% |████████████████████████████████| 92kB 971kB/s 
Collecting oss2==2.4.0 (from jms-storage==0.0.18->-r requirements.txt (line 64))
  Downloading https://files.pythonhosted.org/packages/57/c2/6ea84c56d398988482610e835b92f2f990e3b439a56f2e7c3fba9c5ae51b/oss2-2.4.0.tar.gz (102kB)
    100% |████████████████████████████████| 112kB 483kB/s 
Installing collected packages: vine, amqp, MarkupSafe, Jinja2, PyYAML, pyasn1, pycparser, cffi, six, PyNaCl, bcrypt, asn1crypto, idna, cryptography, paramiko, ansible, billiard, python-dateutil, jmespath, docutils, botocore, s3transfer, boto3, pytz, kombu, celery, certifi, chardet, configparser, urllib3, requests, uritemplate, itypes, coreschema, coreapi, decorator, Django, pyldap, django-auth-ldap, django-bootstrap3, django-celery-beat, django-filter, django-formtools, django-ranged-response, redis, django-redis-cache, openapi-codec, djangorestframework, simplejson, django-rest-swagger, olefile, Pillow, django-simple-captcha, djangorestframework-bulk, ecdsa, elasticsearch, enum-compat, ephem, greenlet, eventlet, ForgeryPy, gunicorn, itsdangerous, ldap3, mysqlclient, passlib, pycrypto, pyotp, python-gssapi, crcmod, oss2, jms-storage, sshpubkeys
  Running setup.py install for MarkupSafe ... done
  Running setup.py install for PyYAML ... done
  Running setup.py install for pycparser ... done
  Running setup.py install for ansible ... done
  Running setup.py install for configparser ... done
  Running setup.py install for itypes ... done
  Running setup.py install for coreschema ... done
  Running setup.py install for pyldap ... done
  Running setup.py install for django-bootstrap3 ... done
  Running setup.py install for django-ranged-response ... done
  Running setup.py install for django-redis-cache ... done
  Running setup.py install for openapi-codec ... done
  Running setup.py install for simplejson ... done
  Running setup.py install for olefile ... done
  Running setup.py install for django-simple-captcha ... done
  Running setup.py install for djangorestframework-bulk ... done
  Running setup.py install for enum-compat ... done
  Running setup.py install for ephem ... done
  Running setup.py install for ForgeryPy ... done
  Running setup.py install for itsdangerous ... done
  Running setup.py install for mysqlclient ... done
  Running setup.py install for pycrypto ... done
  Running setup.py install for python-gssapi ... done
  Running setup.py install for crcmod ... done
  Running setup.py install for oss2 ... done
  Running setup.py install for jms-storage ... done
Successfully installed Django-1.11 ForgeryPy-0.1 Jinja2-2.10 MarkupSafe-1.0 Pillow-4.3.0 PyNaCl-1.2.1 PyYAML-3.12 amqp-2.1.4 ansible-2.4.2.0 asn1crypto-0.24.0 bcrypt-3.1.4 billiard-3.5.0.3 boto3-1.6.5 botocore-1.9.5 celery-4.1.0 certifi-2018.1.18 cffi-1.11.2 chardet-3.0.4 configparser-3.5.0 coreapi-2.3.3 coreschema-0.0.4 crcmod-1.7 cryptography-2.1.4 decorator-4.1.2 django-auth-ldap-1.3.0 django-bootstrap3-9.1.0 django-celery-beat-1.1.1 django-filter-1.1.0 django-formtools-2.1 django-ranged-response-0.2.0 django-redis-cache-1.7.1 django-rest-swagger-2.1.2 django-simple-captcha-0.5.6 djangorestframework-3.7.3 djangorestframework-bulk-0.2.1 docutils-0.14 ecdsa-0.13 elasticsearch-6.1.1 enum-compat-0.0.2 ephem-3.7.6.0 eventlet-0.22.1 greenlet-0.4.12 gunicorn-19.7.1 idna-2.6 itsdangerous-0.24 itypes-1.1.0 jmespath-0.9.3 jms-storage-0.0.18 kombu-4.0.2 ldap3-2.4 mysqlclient-1.3.12 olefile-0.44 openapi-codec-1.3.2 oss2-2.4.0 paramiko-2.4.0 passlib-1.7.1 pyasn1-0.4.2 pycparser-2.18 pycrypto-2.6.1 pyldap-2.4.45 pyotp-2.2.6 python-dateutil-2.6.1 python-gssapi-0.6.4 pytz-2018.3 redis-2.10.6 requests-2.18.4 s3transfer-0.1.13 simplejson-3.13.2 six-1.11.0 sshpubkeys-2.2.0 uritemplate-3.0.0 urllib3-1.22 vine-1.1.4
You are using pip version 9.0.1, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
(py3) [root@h165 requirements]# echo $?
0
(py3) [root@h165 requirements]# pip install --upgrade pip
Collecting pip
  Downloading https://files.pythonhosted.org/packages/5f/25/e52d3f31441505a5f3af41213346e5b6c221c9e086a166f3703d2ddaf940/pip-18.0-py2.py3-none-any.whl (1.3MB)
    100% |████████████████████████████████| 1.3MB 543kB/s 
Installing collected packages: pip
  Found existing installation: pip 9.0.1
    Uninstalling pip-9.0.1:
      Successfully uninstalled pip-9.0.1
Successfully installed pip-18.0
(py3) [root@h165 requirements]# 
~~~

**Note:** 不要指定-i参数，因为镜像上可能没有最新的包

## 安装 Redis ##

Jumpserver 使用 Redis 做 cache 和 celery broke

~~~
(py3) [root@h165 requirements]# yum -y install redis
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package redis.x86_64 0:3.2.10-2.el7 will be installed
--> Processing Dependency: libjemalloc.so.1()(64bit) for package: redis-3.2.10-2.el7.x86_64
--> Running transaction check
---> Package jemalloc.x86_64 0:3.6.0-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================
 Package              Arch               Version                  Repository        Size
=========================================================================================
Installing:
 redis                x86_64             3.2.10-2.el7             epel             545 k
Installing for dependencies:
 jemalloc             x86_64             3.6.0-1.el7              epel             105 k

Transaction Summary
=========================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 650 k
Installed size: 1.7 M
Downloading packages:
(1/2): jemalloc-3.6.0-1.el7.x86_64.rpm                            | 105 kB  00:00:00     
(2/2): redis-3.2.10-2.el7.x86_64.rpm                              | 545 kB  00:00:08     
-----------------------------------------------------------------------------------------
Total                                                        81 kB/s | 650 kB  00:08     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : jemalloc-3.6.0-1.el7.x86_64                                           1/2 
  Installing : redis-3.2.10-2.el7.x86_64                                             2/2 
  Verifying  : redis-3.2.10-2.el7.x86_64                                             1/2 
  Verifying  : jemalloc-3.6.0-1.el7.x86_64                                           2/2 

Installed:
  redis.x86_64 0:3.2.10-2.el7                                                            

Dependency Installed:
  jemalloc.x86_64 0:3.6.0-1.el7                                                          

Complete!
(py3) [root@h165 requirements]# systemctl enable redis
Created symlink from /etc/systemd/system/multi-user.target.wants/redis.service to /usr/lib/systemd/system/redis.service.
(py3) [root@h165 requirements]# systemctl start redis
(py3) [root@h165 requirements]# systemctl status redis
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since 日 2018-07-22 13:57:35 UTC; 5s ago
 Main PID: 21683 (redis-server)
   CGroup: /system.slice/redis.service
           └─21683 /usr/bin/redis-server 127.0.0.1:6379

7月 22 13:57:35 h165 systemd[1]: Started Redis persistent key-value database.
7月 22 13:57:35 h165 systemd[1]: Starting Redis persistent key-value database...
(py3) [root@h165 requirements]# 
~~~


## 安装 MySQL ##

使用 Mysql 作为数据库

~~~
(py3) [root@h165 ~]# yum -y install mariadb mariadb-devel mariadb-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Package 1:mariadb-devel-5.5.56-2.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package mariadb.x86_64 1:5.5.56-2.el7 will be installed
---> Package mariadb-server.x86_64 1:5.5.56-2.el7 will be installed
--> Processing Dependency: perl-DBI for package: 1:mariadb-server-5.5.56-2.el7.x86_64
--> Processing Dependency: perl-DBD-MySQL for package: 1:mariadb-server-5.5.56-2.el7.x86_64
--> Processing Dependency: perl(DBI) for package: 1:mariadb-server-5.5.56-2.el7.x86_64
--> Running transaction check
---> Package perl-DBD-MySQL.x86_64 0:4.023-6.el7 will be installed
---> Package perl-DBI.x86_64 0:1.627-4.el7 will be installed
--> Processing Dependency: perl(RPC::PlServer) >= 0.2001 for package: perl-DBI-1.627-4.el7.x86_64
--> Processing Dependency: perl(RPC::PlClient) >= 0.2000 for package: perl-DBI-1.627-4.el7.x86_64
--> Running transaction check
---> Package perl-PlRPC.noarch 0:0.2020-14.el7 will be installed
--> Processing Dependency: perl(Net::Daemon) >= 0.13 for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Net::Daemon::Test) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Net::Daemon::Log) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Compress::Zlib) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Running transaction check
---> Package perl-IO-Compress.noarch 0:2.061-2.el7 will be installed
--> Processing Dependency: perl(Compress::Raw::Zlib) >= 2.061 for package: perl-IO-Compress-2.061-2.el7.noarch
--> Processing Dependency: perl(Compress::Raw::Bzip2) >= 2.061 for package: perl-IO-Compress-2.061-2.el7.noarch
---> Package perl-Net-Daemon.noarch 0:0.48-5.el7 will be installed
--> Running transaction check
---> Package perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7 will be installed
---> Package perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================
 Package                        Arch          Version                  Repository   Size
=========================================================================================
Installing:
 mariadb                        x86_64        1:5.5.56-2.el7           base        8.7 M
 mariadb-server                 x86_64        1:5.5.56-2.el7           base         11 M
Installing for dependencies:
 perl-Compress-Raw-Bzip2        x86_64        2.061-3.el7              base         32 k
 perl-Compress-Raw-Zlib         x86_64        1:2.061-4.el7            base         57 k
 perl-DBD-MySQL                 x86_64        4.023-6.el7              base        140 k
 perl-DBI                       x86_64        1.627-4.el7              base        802 k
 perl-IO-Compress               noarch        2.061-2.el7              base        260 k
 perl-Net-Daemon                noarch        0.48-5.el7               base         51 k
 perl-PlRPC                     noarch        0.2020-14.el7            base         36 k

Transaction Summary
=========================================================================================
Install  2 Packages (+7 Dependent packages)

Total download size: 21 M
Installed size: 110 M
Downloading packages:
(1/9): perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm             |  32 kB  00:00:00     
(2/9): perl-DBD-MySQL-4.023-6.el7.x86_64.rpm                      | 140 kB  00:00:00     
(3/9): perl-DBI-1.627-4.el7.x86_64.rpm                            | 802 kB  00:00:00     
(4/9): perl-Net-Daemon-0.48-5.el7.noarch.rpm                      |  51 kB  00:00:00     
(5/9): perl-PlRPC-0.2020-14.el7.noarch.rpm                        |  36 kB  00:00:00     
(6/9): perl-IO-Compress-2.061-2.el7.noarch.rpm                    | 260 kB  00:00:01     
(7/9): mariadb-server-5.5.56-2.el7.x86_64.rpm                     |  11 MB  00:00:08     
(8/9): mariadb-5.5.56-2.el7.x86_64.rpm                            | 8.7 MB  00:00:08     
(9/9): perl-Compress-Raw-Zlib-2.061-4.el7.x86_64.rpm              |  57 kB  00:00:09     
-----------------------------------------------------------------------------------------
Total                                                       2.2 MB/s |  21 MB  00:09     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:mariadb-5.5.56-2.el7.x86_64                                         1/9 
  Installing : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                           2/9 
  Installing : perl-Net-Daemon-0.48-5.el7.noarch                                     3/9 
  Installing : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                            4/9 
  Installing : perl-IO-Compress-2.061-2.el7.noarch                                   5/9 
  Installing : perl-PlRPC-0.2020-14.el7.noarch                                       6/9 
  Installing : perl-DBI-1.627-4.el7.x86_64                                           7/9 
  Installing : perl-DBD-MySQL-4.023-6.el7.x86_64                                     8/9 
  Installing : 1:mariadb-server-5.5.56-2.el7.x86_64                                  9/9 
  Verifying  : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                            1/9 
  Verifying  : perl-Net-Daemon-0.48-5.el7.noarch                                     2/9 
  Verifying  : perl-DBD-MySQL-4.023-6.el7.x86_64                                     3/9 
  Verifying  : perl-PlRPC-0.2020-14.el7.noarch                                       4/9 
  Verifying  : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                           5/9 
  Verifying  : 1:mariadb-server-5.5.56-2.el7.x86_64                                  6/9 
  Verifying  : perl-IO-Compress-2.061-2.el7.noarch                                   7/9 
  Verifying  : perl-DBI-1.627-4.el7.x86_64                                           8/9 
  Verifying  : 1:mariadb-5.5.56-2.el7.x86_64                                         9/9 

Installed:
  mariadb.x86_64 1:5.5.56-2.el7           mariadb-server.x86_64 1:5.5.56-2.el7          

Dependency Installed:
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7                                           
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7                                            
  perl-DBD-MySQL.x86_64 0:4.023-6.el7                                                    
  perl-DBI.x86_64 0:1.627-4.el7                                                          
  perl-IO-Compress.noarch 0:2.061-2.el7                                                  
  perl-Net-Daemon.noarch 0:0.48-5.el7                                                    
  perl-PlRPC.noarch 0:0.2020-14.el7                                                      

Complete!
(py3) [root@h165 ~]# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
(py3) [root@h165 ~]# systemctl start mariadb
(py3) [root@h165 ~]# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-07-22 14:01:17 UTC; 4s ago
  Process: 21881 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 21800 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 21880 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─21880 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─22041 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugi...

7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: MySQL manual for more instructions.
7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: Please report any problems at ht...a
7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: The latest information about Mar....
7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: You can find additional informat...:
7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: http://dev.mysql.com
7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: Consider joining MariaDB's stron...:
7月 22 14:01:15 h165 mariadb-prepare-db-dir[21800]: https://mariadb.org/get-involved/
7月 22 14:01:15 h165 mysqld_safe[21880]: 180722 14:01:15 mysqld_safe Logging to '/v...'.
7月 22 14:01:15 h165 mysqld_safe[21880]: 180722 14:01:15 mysqld_safe Starting mysql...ql
7月 22 14:01:17 h165 systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
(py3) [root@h165 ~]# 
~~~


## 创建数据库 Jumpserver 并授权 ##

~~~
(py3) [root@h165 ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database jumpserver default charset 'utf8';
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by 'somepassword';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> 
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jumpserver         |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

MariaDB [(none)]> show create database jumpserver;
+------------|---------------------------------------------------------------------+
| Database   | Create Database                                                     |
+------------|---------------------------------------------------------------------+
| jumpserver | CREATE DATABASE `jumpserver` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+------------|---------------------------------------------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> exit
Bye
(py3) [root@h165 ~]#
~~~


## 修改 Jumpserver 配置文件 ##


~~~
(py3) [root@h165 ~]# cd /opt/jumpserver/
(py3) [root@h165 jumpserver]# cp config_example.py  config.py
(py3) [root@h165 jumpserver]#
(py3) [root@h165 jumpserver]# vi config.py 
(py3) [root@h165 jumpserver]#
(py3) [root@h165 jumpserver]# cat config.py 
xxx
    jumpserver.config
   

    Jumpserver project setting file

    :copyright: (c) 2014-2017 by Jumpserver Team
    :license: GPL v2, see LICENSE for more details.
xxx
import os

BASE_DIR = os.path.dirname(os.path.abspath(__file__))


class Config:
    # Use it to encrypt or decrypt data
    # SECURITY WARNING: keep the secret key used in production secret!
    #SECRET_KEY = os.environ.get('SECRET_KEY') or '2vym+ky!997d5kkcc64mnz06y1mmui3lut#(^wd=%s_qj$1%x'
    SECRET_KEY = '2vym+ky!997d5kkcc64mnz06y1mmui3lut#(^wd=%s_qj$1%x'

    # Django security setting, if your disable debug model, you should setting that
    ALLOWED_HOSTS = ['*']

    # Development env open this, when error occur display the full process track, Production disable it
    DEBUG = os.environ.get("DEBUG") or False

    # DEBUG, INFO, WARNING, ERROR, CRITICAL can set. See https://docs.djangoproject.com/en/1.10/topics/logging/
    LOG_LEVEL = os.environ.get("LOG_LEVEL") or 'DEBUG'
    LOG_DIR = os.path.join(BASE_DIR, 'logs')

    # Database setting, Support sqlite3, mysql, postgres ....
    # See https://docs.djangoproject.com/en/1.10/ref/settings/#databases

    # SQLite setting:
    #DB_ENGINE = 'sqlite3'
    #DB_NAME = os.path.join(BASE_DIR, 'data', 'db.sqlite3')

    # MySQL or postgres setting like:
    DB_ENGINE = os.environ.get("DB_ENGINE") or 'mysql'
    DB_HOST = os.environ.get("DB_HOST") or '127.0.0.1'
    DB_PORT = os.environ.get("DB_PORT") or 3306
    DB_USER = os.environ.get("DB_USER") or 'jumpserver'
    DB_PASSWORD = os.environ.get("DB_PASSWORD") or 'weakPassword'
    DB_NAME = os.environ.get("DB_NAME") or 'jumpserver'

    # When Django start it will bind this host and port
    # ./manage.py runserver 127.0.0.1:8080
    HTTP_BIND_HOST = '0.0.0.0'
    HTTP_LISTEN_PORT = 8080

    # Use Redis as broker for celery and web socket
    REDIS_HOST = os.environ.get("REDIS_HOST") or '127.0.0.1'
    REDIS_PORT = os.environ.get("REDIS_PORT") or 6379
    REDIS_PASSWORD = os.environ.get("REDIS_PASSWORD") or ''
    REDIS_DB_CELERY = os.environ.get('REDIS_DB') or 3
    REDIS_DB_CACHE = os.environ.get('REDIS_DB') or 4

    def __init__(self):
        pass

    def __getattr__(self, item):
        return None


class DevelopmentConfig(Config):
    pass


class TestConfig(Config):
    pass


class ProductionConfig(Config):
    pass


# Default using Config settings, you can write if/else for different env
config = DevelopmentConfig()

(py3) [root@h165 jumpserver]# 
~~~

## 生成数据库表结构和初始化数据 ##


~~~
(py3) [root@h165 ~]# cd /opt/jumpserver/utils
(py3) [root@h165 utils]# bash make_migrations.sh
2018-07-23 01:50:39 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:50:39 [signals_handler DEBUG]   - fresh all settings
Migrations for 'assets':
  /opt/jumpserver/apps/assets/migrations/0002_auto_20180723_0150.py
    - Create model Domain
    - Create model Gateway
    - Create model Label
    - Create model Node
    - Change Meta options on adminuser
    - Change Meta options on asset
    - Change Meta options on assetgroup
    - Change Meta options on cluster
    - Change Meta options on systemuser
    - Remove field cabinet_no from asset
    - Remove field cabinet_pos from asset
    - Remove field cluster from asset
    - Remove field env from asset
    - Remove field groups from asset
    - Remove field remote_card_ip from asset
    - Remove field status from asset
    - Remove field type from asset
    - Remove field cluster from systemuser
    - Add field protocol to asset
    - Add field assets to systemuser
    - Add field login_mode to systemuser
    - Alter field created_by on adminuser
    - Alter field username on adminuser
    - Alter field admin_user on asset
    - Alter field platform on asset
    - Alter field created_by on assetgroup
    - Alter field created_by on systemuser
    - Alter field protocol on systemuser
    - Alter field sudo on systemuser
    - Alter field username on systemuser
    - Alter unique_together for label (1 constraint(s))
    - Add field domain to asset
    - Add field labels to asset
    - Add field nodes to asset
    - Add field nodes to systemuser
Migrations for 'audits':
  /opt/jumpserver/apps/audits/migrations/0001_initial.py
    - Create model FTPLog
Migrations for 'common':
  /opt/jumpserver/apps/common/migrations/0001_initial.py
    - Create model Setting
Migrations for 'ops':
  /opt/jumpserver/apps/ops/migrations/0002_celerytask.py
    - Create model CeleryTask
Migrations for 'terminal':
  /opt/jumpserver/apps/terminal/migrations/0002_auto_20180723_0150.py
    - Add field date_last_active to session
    - Add field protocol to session
    - Add field remote_addr to session
    - Add field terminal to session
    - Add field terminal to status
    - Add field terminal to task
    - Add field command_storage to terminal
    - Add field replay_storage to terminal
    - Add field user to terminal
    - Alter field asset on command
    - Alter field system_user on command
    - Alter field user on command
    - Alter field date_start on session
    - Alter field name on terminal
Migrations for 'users':
  /opt/jumpserver/apps/users/migrations/0003_auto_20180723_0150.py
    - Change Meta options on user
    - Change Meta options on usergroup
    - Remove field enable_otp from user
    - Remove field secret_key_otp from user
    - Remove field discard_time from usergroup
    - Remove field is_discard from usergroup
    - Add field mfa to loginlog
    - Add field reason to loginlog
    - Add field status to loginlog
    - Add field _otp_secret_key to user
    - Add field otp_level to user
    - Add field source to user
    - Alter field date_expired on user
    - Alter field is_first_login on user
    - Alter field created_by on usergroup
    - Alter field name on usergroup
Migrations for 'perms':
  /opt/jumpserver/apps/perms/migrations/0002_auto_20180723_0150.py
    - Create model NodePermission
    - Remove field asset_groups from assetpermission
    - Add field date_start to assetpermission
    - Add field nodes to assetpermission
    - Add field user_groups to assetpermission
    - Add field users to assetpermission
    - Alter field date_expired on assetpermission
    - Alter unique_together for nodepermission (1 constraint(s))
2018-07-23 01:50:41 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:50:41 [signals_handler DEBUG]   - fresh all settings
System check identified some issues:

WARNINGS:
?: (mysql.W002) MySQL Strict Mode is not set for database connection 'default'
	HINT: MySQL's Strict Mode fixes many data integrity problems in MySQL, such as data truncation upon insertion, by escalating warnings into errors. It is strongly recommended you activate it. See: https://docs.djangoproject.com/en/1.11/ref/databases/#mysql-sql-mode
Operations to perform:
  Apply all migrations: assets, audits, auth, captcha, common, contenttypes, django_celery_beat, ops, perms, sessions, terminal, users
Running migrations:
  Applying assets.0001_initial... OK
  Applying assets.0002_auto_20180723_0150... OK
  Applying audits.0001_initial... OK
  Applying contenttypes.0001_initial... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0001_initial... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying captcha.0001_initial... OK
  Applying common.0001_initial... OK
  Applying django_celery_beat.0001_initial... OK
  Applying django_celery_beat.0002_auto_20161118_0346... OK
  Applying django_celery_beat.0003_auto_20161209_0049... OK
  Applying django_celery_beat.0004_auto_20170221_0000... OK
  Applying django_celery_beat.0005_add_solarschedule_events_choices... OK
  Applying django_celery_beat.0006_auto_20180210_1226... OK
  Applying ops.0001_initial... OK
  Applying ops.0002_celerytask... OK
  Applying users.0001_initial... OK
  Applying users.0002_auto_20171225_1157... OK
  Applying users.0003_auto_20180723_0150... OK
  Applying perms.0001_initial... OK
  Applying perms.0002_auto_20180723_0150... OK
  Applying sessions.0001_initial... OK
  Applying terminal.0001_initial... OK
  Applying terminal.0002_auto_20180723_0150... OK
2018-07-23 01:50:45 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:50:45 [signals_handler DEBUG]   - fresh all settings
No conflicts detected to merge.
(py3) [root@h165 utils]# echo $?
0
(py3) [root@h165 utils]# 
~~~

## 运行 Jumpserver ##

~~~
(py3) [root@h165 utils]# cd /opt/jumpserver
(py3) [root@h165 jumpserver]# ./jms start all
Sun Jul 22 17:52:27 2018
Jumpserver version 1.3.3, more see https://www.jumpserver.org

- Start Gunicorn WSGI HTTP Server
Check database structure change ...
2018-07-23 01:52:29 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:29 [signals_handler DEBUG]   - fresh all settings
System check identified some issues:

WARNINGS:
?: (mysql.W002) MySQL Strict Mode is not set for database connection 'default'
	HINT: MySQL's Strict Mode fixes many data integrity problems in MySQL, such as data truncation upon insertion, by escalating warnings into errors. It is strongly recommended you activate it. See: https://docs.djangoproject.com/en/1.11/ref/databases/#mysql-sql-mode
Operations to perform:
  Apply all migrations: assets, audits, auth, captcha, common, contenttypes, django_celery_beat, ops, perms, sessions, terminal, users
Running migrations:
  No migrations to apply.
Collect static files
2018-07-23 01:52:31 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:31 [signals_handler DEBUG]   - fresh all settings
Copying '/opt/jumpserver/apps/static/css/animate.css'
Copying '/opt/jumpserver/apps/static/css/bootstrap.min.css'
Copying '/opt/jumpserver/apps/static/css/colorbox.css'
Copying '/opt/jumpserver/apps/static/css/font-awesome.min.css'
Copying '/opt/jumpserver/apps/static/css/jumpserver.css'
Copying '/opt/jumpserver/apps/static/css/otp.css'
Copying '/opt/jumpserver/apps/static/css/style.css'
Copying '/opt/jumpserver/apps/static/css/images/jbox-button1.png'
Copying '/opt/jumpserver/apps/static/css/images/jbox-close.gif'
Copying '/opt/jumpserver/apps/static/css/images/jbox-icons.png'
Copying '/opt/jumpserver/apps/static/css/patterns/congruent_pentagon.png'
Copying '/opt/jumpserver/apps/static/css/patterns/header-profile-skin-1.png'
Copying '/opt/jumpserver/apps/static/css/patterns/header-profile-skin-2.png'
Copying '/opt/jumpserver/apps/static/css/patterns/header-profile-skin-3.png'
Copying '/opt/jumpserver/apps/static/css/patterns/header-profile.png'
Copying '/opt/jumpserver/apps/static/css/patterns/otis_redding.png'
Copying '/opt/jumpserver/apps/static/css/patterns/shattered.png'
Copying '/opt/jumpserver/apps/static/css/patterns/triangular.png'
Copying '/opt/jumpserver/apps/static/css/plugins/bootstrap.min.css'
Copying '/opt/jumpserver/apps/static/css/plugins/inputTags.css'
Copying '/opt/jumpserver/apps/static/css/plugins/awesome-bootstrap-checkbox/awesome-bootstrap-checkbox.css'
Copying '/opt/jumpserver/apps/static/css/plugins/cropper/cropper.min.css'
Copying '/opt/jumpserver/apps/static/css/plugins/datatables/datatables.min.css'
Copying '/opt/jumpserver/apps/static/css/plugins/datatables/datatables.min.css.bak'
Copying '/opt/jumpserver/apps/static/css/plugins/datepicker/datepicker3.css'
Copying '/opt/jumpserver/apps/static/css/plugins/dropzone/basic.css'
Copying '/opt/jumpserver/apps/static/css/plugins/dropzone/dropzone.css'
Copying '/opt/jumpserver/apps/static/css/plugins/footable/footable.core.css'
Copying '/opt/jumpserver/apps/static/css/plugins/footable/fonts/footable.eot'
Copying '/opt/jumpserver/apps/static/css/plugins/footable/fonts/footable.svg'
Copying '/opt/jumpserver/apps/static/css/plugins/footable/fonts/footable.ttf'
Copying '/opt/jumpserver/apps/static/css/plugins/footable/fonts/footable.woff'
Copying '/opt/jumpserver/apps/static/css/plugins/fullcalendar/fullcalendar.css'
Copying '/opt/jumpserver/apps/static/css/plugins/fullcalendar/fullcalendar.print.css'
Copying '/opt/jumpserver/apps/static/css/plugins/iCheck/custom.css'
Copying '/opt/jumpserver/apps/static/css/plugins/iCheck/green.png'
Copying '/opt/jumpserver/apps/static/css/plugins/iCheck/green@2x.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sort.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sort_asc.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sort_desc.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sprite-skin-flat.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sprite-skin-flat2.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sprite-skin-nice.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/sprite-skin-simple.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/spritemap.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/spritemap@2x.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/bootstrap-colorpicker/alpha-horizontal.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/bootstrap-colorpicker/alpha.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/bootstrap-colorpicker/hue-horizontal.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/bootstrap-colorpicker/hue.png'
Copying '/opt/jumpserver/apps/static/css/plugins/images/bootstrap-colorpicker/saturation.png'
Copying '/opt/jumpserver/apps/static/css/plugins/jstree/32px.png'
Copying '/opt/jumpserver/apps/static/css/plugins/jstree/39px.png'
Copying '/opt/jumpserver/apps/static/css/plugins/jstree/40px.png'
Copying '/opt/jumpserver/apps/static/css/plugins/jstree/style.css'
Copying '/opt/jumpserver/apps/static/css/plugins/jstree/style.min.css'
Copying '/opt/jumpserver/apps/static/css/plugins/jstree/throbber.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/layer/layer.css'
Copying '/opt/jumpserver/apps/static/css/plugins/layer/default/icon-ext.png'
Copying '/opt/jumpserver/apps/static/css/plugins/layer/default/icon.png'
Copying '/opt/jumpserver/apps/static/css/plugins/layer/default/loading-0.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/layer/default/loading-1.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/layer/default/loading-2.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/select2/select2.min.css'
Copying '/opt/jumpserver/apps/static/css/plugins/steps/jquery.steps.css'
Copying '/opt/jumpserver/apps/static/css/plugins/sweetalert/sweetalert.css'
Copying '/opt/jumpserver/apps/static/css/plugins/toastr/toastr.min.css'
Copying '/opt/jumpserver/apps/static/css/plugins/vaildator/jquery.validator.css'
Copying '/opt/jumpserver/apps/static/css/plugins/vaildator/images/loading.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/vaildator/images/validator_default.png'
Copying '/opt/jumpserver/apps/static/css/plugins/vaildator/images/validator_simple.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/demo.css'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/awesomeStyle/awesome.css'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/awesomeStyle/awesome.less'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/awesomeStyle/fa.css'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/awesomeStyle/fa.less'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/awesomeStyle/img/loading.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/metroStyle/metroStyle.css'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/metroStyle/img/line_conn.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/metroStyle/img/loading.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/metroStyle/img/metro.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/metroStyle/img/metro.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/ztreestyle.css'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/line_conn.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/loading.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/zTreeStandard.gif'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/zTreeStandard.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/1_close.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/1_open.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/2.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/3.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/4.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/5.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/6.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/7.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/8.png'
Copying '/opt/jumpserver/apps/static/css/plugins/ztree/ztreestyle/img/diy/9.png'
Copying '/opt/jumpserver/apps/static/fonts/FontAwesome.otf'
Copying '/opt/jumpserver/apps/static/fonts/fontawesome-webfont.eot'
Copying '/opt/jumpserver/apps/static/fonts/fontawesome-webfont.svg'
Copying '/opt/jumpserver/apps/static/fonts/fontawesome-webfont.ttf'
Copying '/opt/jumpserver/apps/static/fonts/fontawesome-webfont.woff'
Copying '/opt/jumpserver/apps/static/fonts/fontawesome-webfont.woff2'
Copying '/opt/jumpserver/apps/static/fonts/glyphicons-halflings-regular.eot'
Copying '/opt/jumpserver/apps/static/fonts/glyphicons-halflings-regular.svg'
Copying '/opt/jumpserver/apps/static/fonts/glyphicons-halflings-regular.ttf'
Copying '/opt/jumpserver/apps/static/fonts/glyphicons-halflings-regular.woff'
Copying '/opt/jumpserver/apps/static/fonts/glyphicons-halflings-regular.woff2'
Copying '/opt/jumpserver/apps/static/fonts/font_otp/iconfont.css'
Copying '/opt/jumpserver/apps/static/fonts/font_otp/iconfont.eot'
Copying '/opt/jumpserver/apps/static/fonts/font_otp/iconfont.js'
Copying '/opt/jumpserver/apps/static/fonts/font_otp/iconfont.svg'
Copying '/opt/jumpserver/apps/static/fonts/font_otp/iconfont.ttf'
Copying '/opt/jumpserver/apps/static/fonts/font_otp/iconfont.woff'
Copying '/opt/jumpserver/apps/static/img/authenticator_android.png'
Copying '/opt/jumpserver/apps/static/img/authenticator_iphone.png'
Copying '/opt/jumpserver/apps/static/img/facio.ico'
Copying '/opt/jumpserver/apps/static/img/logo-text.png'
Copying '/opt/jumpserver/apps/static/img/logo.png'
Copying '/opt/jumpserver/apps/static/img/otp_auth.png'
Copying '/opt/jumpserver/apps/static/img/root.png'
Copying '/opt/jumpserver/apps/static/img/avatar/admin.png'
Copying '/opt/jumpserver/apps/static/img/avatar/user.png'
Copying '/opt/jumpserver/apps/static/js/angular-route.min.js'
Copying '/opt/jumpserver/apps/static/js/angular.min.js'
Copying '/opt/jumpserver/apps/static/js/bootstrap-dialog.js'
Copying '/opt/jumpserver/apps/static/js/bootstrap.min.js'
Copying '/opt/jumpserver/apps/static/js/inspinia.js'
Copying '/opt/jumpserver/apps/static/js/jquery-2.1.1.js'
Copying '/opt/jumpserver/apps/static/js/jquery-ui-1.10.4.min.js'
Copying '/opt/jumpserver/apps/static/js/jquery-ui.custom.min.js'
Copying '/opt/jumpserver/apps/static/js/jquery.colorbox.js'
Copying '/opt/jumpserver/apps/static/js/jquery.form.min.js'
Copying '/opt/jumpserver/apps/static/js/jquery.shiftcheckbox.js'
Copying '/opt/jumpserver/apps/static/js/jumpserver.js'
Copying '/opt/jumpserver/apps/static/js/mindmup-editabletable.js'
Copying '/opt/jumpserver/apps/static/js/pwstrength-bootstrap.js'
Copying '/opt/jumpserver/apps/static/js/record.js'
Copying '/opt/jumpserver/apps/static/js/term.js'
Copying '/opt/jumpserver/apps/static/js/webterminal.js'
Copying '/opt/jumpserver/apps/static/js/wssh.js'
Copying '/opt/jumpserver/apps/static/js/plugins/inputTags.jquery.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/cropper/cropper.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/datatables/datatables.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/datatables/pdfmake.min.js.map'
Copying '/opt/jumpserver/apps/static/js/plugins/datatables/i18n/English.lang'
Copying '/opt/jumpserver/apps/static/js/plugins/datatables/i18n/zh-hans.json'
Copying '/opt/jumpserver/apps/static/js/plugins/datepicker/bootstrap-datepicker.js'
Copying '/opt/jumpserver/apps/static/js/plugins/demo/peity-demo.js'
Copying '/opt/jumpserver/apps/static/js/plugins/dropzone/dropzone.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/echarts-all.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/echarts.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/bar.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/chord.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/eventRiver.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/force.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/funnel.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/gauge.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/heatmap.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/k.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/line.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/map.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/pie.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/radar.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/scatter.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/tree.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/treemap.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/venn.js'
Copying '/opt/jumpserver/apps/static/js/plugins/echarts/chart/wordCloud.js'
Copying '/opt/jumpserver/apps/static/js/plugins/footable/footable.all.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/footable/footable.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/fullcalendar/fullcalendar.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/fullcalendar/moment.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts-3d.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts-3d.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts-all.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts-more.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts-more.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/highcharts.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/adapters/standalone-framework.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/adapters/standalone-framework.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/canvas-tools.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/canvas-tools.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/data.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/data.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/drilldown.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/drilldown.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/exporting.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/exporting.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/funnel.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/funnel.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/heatmap.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/heatmap.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/no-data-to-display.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/no-data-to-display.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/solid-gauge.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/modules/solid-gauge.src.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/dark-blue.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/dark-green.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/dark-unica.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/gray.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/grid-light.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/grid.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/sand-signika.js'
Copying '/opt/jumpserver/apps/static/js/plugins/highcharts/themes/skies.js'
Copying '/opt/jumpserver/apps/static/js/plugins/iCheck/icheck.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/jstree/jstree.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/layer.js'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/skin/layer.css'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/skin/default/icon-ext.png'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/skin/default/icon.png'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/skin/default/loading-0.gif'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/skin/default/loading-1.gif'
Copying '/opt/jumpserver/apps/static/js/plugins/layer/skin/default/loading-2.gif'
Copying '/opt/jumpserver/apps/static/js/plugins/magnific/jquery.magnific-popup.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/metisMenu/jquery.metisMenu.js'
Copying '/opt/jumpserver/apps/static/js/plugins/pace/pace.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/peity/jquery.peity.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/qrcode/qrcode.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/select2/select2.full.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/slimscroll/jquery.slimscroll.js'
Copying '/opt/jumpserver/apps/static/js/plugins/slimscroll/jquery.slimscroll.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/steps/jquery.steps.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/sweetalert/sweetalert.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/toastr/toastr.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/validate/jquery.validate.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/validator/jquery.validator.js'
Copying '/opt/jumpserver/apps/static/js/plugins/validator/zh_CN.js'
Copying '/opt/jumpserver/apps/static/js/plugins/validator/images/loading.gif'
Copying '/opt/jumpserver/apps/static/js/plugins/validator/images/validator_default.png'
Copying '/opt/jumpserver/apps/static/js/plugins/validator/images/validator_simple.png'
Copying '/opt/jumpserver/apps/static/js/plugins/xterm/xterm.css'
Copying '/opt/jumpserver/apps/static/js/plugins/xterm/xterm.js'
Copying '/opt/jumpserver/apps/static/js/plugins/xterm/xterm.js.map'
Copying '/opt/jumpserver/apps/static/js/plugins/ztree/jquery-1.4.4.min.js'
Copying '/opt/jumpserver/apps/static/js/plugins/ztree/jquery.ztree.all.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/css/bootstrap-tweaks.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/css/bootstrap.min.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/css/default.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/css/prettify.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/css/base.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/css/bootstrap-theme.min.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/css/bootstrap.min.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/css/font-awesome-4.0.3.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/css/highlight.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/css/jquery.json-view.min.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/fontawesome-webfont.eot'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/fontawesome-webfont.svg'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/fontawesome-webfont.ttf'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/fontawesome-webfont.woff'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/glyphicons-halflings-regular.eot'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/glyphicons-halflings-regular.svg'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/glyphicons-halflings-regular.ttf'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/glyphicons-halflings-regular.woff'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/fonts/glyphicons-halflings-regular.woff2'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/img/favicon.ico'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/img/grid.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/js/api.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/js/bootstrap.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/js/highlight.pack.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/js/jquery-1.10.2.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/docs/js/jquery.json-view.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/fonts/glyphicons-halflings-regular.eot'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/fonts/glyphicons-halflings-regular.svg'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/fonts/glyphicons-halflings-regular.ttf'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/fonts/glyphicons-halflings-regular.woff'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/fonts/glyphicons-halflings-regular.woff2'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/img/glyphicons-halflings-white.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/img/glyphicons-halflings.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/img/grid.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/ajax-form.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/bootstrap.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/coreapi-0.1.1.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/csrf.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/default.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/jquery-1.12.4.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework/static/rest_framework/js/prettify-min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/init.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/o2c.html'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/swagger-ui.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/swagger-ui.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/css/print.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/css/reset.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/css/screen.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/css/typography.css'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/fonts/DroidSans-Bold.ttf'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/fonts/DroidSans.ttf'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/collapse.gif'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/expand.gif'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/explorer_icons.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/favicon-16x16.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/favicon-32x32.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/favicon.ico'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/logo_small.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/pet_store_api.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/throbber.gif'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/images/wordnik_api.png'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/en.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/es.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/fr.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/geo.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/it.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/ja.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/ko-kr.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/pl.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/pt.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/ru.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/tr.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/translator.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lang/zh-cn.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/backbone-min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/handlebars-2.0.0.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/highlight.9.1.0.pack.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/highlight.9.1.0.pack_extended.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/jquery-1.8.0.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/jquery.ba-bbq.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/jquery.slideto.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/jquery.wiggle.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/js-yaml.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/jsoneditor.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/lodash.min.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/marked.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/object-assign-pollyfill.js'
Copying '/opt/py3/lib/python3.6/site-packages/rest_framework_swagger/static/rest_framework_swagger/lib/swagger-oauth.js'

325 static files copied to '/opt/jumpserver/data/static'.

- Start Celery as Distributed Task Queue

- Start Beat as Periodic Task Scheduler
[2018-07-22 17:52:31 +0000] [22571] [INFO] Starting gunicorn 19.7.1
[2018-07-22 17:52:31 +0000] [22571] [INFO] Listening at: http://0.0.0.0:8080 (22571)
[2018-07-22 17:52:31 +0000] [22571] [INFO] Using worker: eventlet
[2018-07-22 17:52:31 +0000] [22582] [INFO] Booting worker with pid: 22582
[2018-07-22 17:52:31 +0000] [22583] [INFO] Booting worker with pid: 22583
[2018-07-22 17:52:31 +0000] [22584] [INFO] Booting worker with pid: 22584
[2018-07-22 17:52:32 +0000] [22586] [INFO] Booting worker with pid: 22586
2018-07-23 01:52:33 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:33 [signals_handler DEBUG]   - fresh all settings
2018-07-23 01:52:33 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:33 [signals_handler DEBUG]   - fresh all settings
celery beat v4.1.0 (latentcall) is starting.
2018-07-23 01:52:33 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:33 [signals_handler DEBUG]   - fresh all settings
2018-07-23 01:52:33 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:33 [signals_handler DEBUG]   - fresh all settings
2018-07-23 01:52:35 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:35 [signals_handler DEBUG]   - fresh all settings
2018-07-23 01:52:35 [signals_handler DEBUG] Receive django ready signal
2018-07-23 01:52:35 [signals_handler DEBUG]   - fresh all settings
/opt/py3/lib/python3.6/site-packages/celery/platforms.py:795: RuntimeWarning: You're running the worker with superuser privileges: this is
absolutely not recommended!

Please specify a different user using the -u option.

User information: uid=0 euid=0 gid=0 egid=0

  uid=uid, euid=euid, gid=gid, egid=egid,
__    -    ... __   -        _
LocalTime -> 2018-07-23 01:52:36
Configuration ->
    . broker -> redis://127.0.0.1:6379/3
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> django_celery_beat.schedulers.DatabaseScheduler

    . logfile -> [stderr]@%DEBUG
    . maxinterval -> 1.00 minute (60.0s)
Setting default socket timeout to 30
beat: Starting...
DatabaseScheduler: initial read
Writing entries...
DatabaseScheduler: Fetching database schedule
Current schedule:
<ModelEntry: terminal.tasks.delete_terminal_status_period terminal.tasks.delete_terminal_status_period(*[], **{}) <freq: 1.00 hour>>
<ModelEntry: terminal.tasks.clean_orphan_session terminal.tasks.clean_orphan_session(*[], **{}) <freq: 1.00 hour>>
beat: Ticking with max interval->1.00 minute
beat: Waking up in 1.00 minute.
| Worker: Preparing bootsteps.
| Worker: Building graph...
| Worker: New boot order: {Beat, StateDB, Timer, Hub, Pool, Autoscaler, Consumer}
| Consumer: Preparing bootsteps.
| Consumer: Building graph...
| Consumer: New boot order: {Connection, Agent, Events, Mingle, Tasks, Control, Gossip, Heart, event loop}
 
 -------------- celery@h165 v4.1.0 (latentcall)
---- **** ----- 
--- * ***  * -- Linux-3.10.0-862.2.3.el7.x86_64-x86_64-with-centos-7.5.1804-Core 2018-07-23 01:52:36
-- * - **** --- 
- ** ---------- [config]
- ** ---------- .> app:         jumpserver:0x7faa2793f978
- ** ---------- .> transport:   redis://127.0.0.1:6379/3
- ** ---------- .> results:     redis://127.0.0.1:6379/3
- *** --- * --- .> concurrency: 4 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery
                

[tasks]
  . assets.tasks.push_system_user_to_assets
  . assets.tasks.push_system_user_to_assets_manual
  . assets.tasks.push_system_user_util
  . assets.tasks.set_admin_user_connectability_info
  . assets.tasks.set_assets_hardware_info
  . assets.tasks.set_system_user_connectablity_info
  . assets.tasks.test_admin_user_connectability_manual
  . assets.tasks.test_admin_user_connectability_period
  . assets.tasks.test_admin_user_connectability_util
  . assets.tasks.test_asset_connectability_manual
  . assets.tasks.test_asset_connectability_util
  . assets.tasks.test_system_user_connectability_manual
  . assets.tasks.test_system_user_connectability_period
  . assets.tasks.test_system_user_connectability_util
  . assets.tasks.update_asset_hardware_info_manual
  . assets.tasks.update_assets_hardware_info_period
  . assets.tasks.update_assets_hardware_info_util
  . celery.accumulate
  . celery.backend_cleanup
  . celery.chain
  . celery.chord
  . celery.chord_unlock
  . celery.chunks
  . celery.group
  . celery.map
  . celery.starmap
  . common.tasks.send_mail_async
  . ops.tasks.hello
  . ops.tasks.hello_callback
  . ops.tasks.run_ansible_task
  . terminal.tasks.clean_orphan_session
  . terminal.tasks.delete_terminal_status_period
  . users.tasks.write_login_log_async

| Worker: Starting Hub
^-- substep ok
| Worker: Starting Pool
^-- substep ok
| Worker: Starting Consumer
| Consumer: Starting Connection
Connected to redis://127.0.0.1:6379/3
^-- substep ok
| Consumer: Starting Events
^-- substep ok
| Consumer: Starting Mingle
mingle: searching for neighbors
mingle: all alone
^-- substep ok
| Consumer: Starting Tasks
^-- substep ok
| Consumer: Starting Control
^-- substep ok
| Consumer: Starting Gossip
^-- substep ok
| Consumer: Starting Heart
^-- substep ok
| Consumer: Starting event loop
| Worker: Hub.register Pool...
2018-07-23 01:52:37 [signal_handler DEBUG] App ready signal recv
App ready signal recv
2018-07-23 01:52:37 [signal_handler DEBUG] Start need start task: [assets.tasks.update_assets_hardware_info_period, assets.tasks.test_admin_user_connectability_period, assets.tasks.test_system_user_connectability_period, terminal.tasks.delete_terminal_status_period, terminal.tasks.clean_orphan_session]
Start need start task: [assets.tasks.update_assets_hardware_info_period, assets.tasks.test_admin_user_connectability_period, assets.tasks.test_system_user_connectability_period, terminal.tasks.delete_terminal_status_period, terminal.tasks.clean_orphan_session]
celery@h165 ready.
basic.qos: prefetch_count->16
Received task: assets.tasks.update_assets_hardware_info_period[aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85]  
TaskPool: Apply <function _fast_trace_task at 0x7faa22f8aea0> (args:('assets.tasks.update_assets_hardware_info_period', 'aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85', {'lang': 'py', 'task': 'assets.tasks.update_assets_hardware_info_period', 'id': 'aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85', 'eta': None, 'expires': None, 'group': None, 'retries': 0, 'timelimit': [None, None], 'root_id': 'aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85', 'parent_id': None, 'argsrepr': '()', 'kwargsrepr': '{}', 'origin': 'gen22572@h165', 'reply_to': '07fc10f1-4f00-3fe0-b8d1-c98b94e70bb3', 'correlation_id': 'aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85', 'delivery_info': {'exchange': '', 'routing_key': 'celery', 'priority': 0, 'redelivered': None}}, b'\x80\x02)}q\x00}q\x01(X\t\x00\x00\x00callbacksq\x02NX\x08\x00\x00\x00errbacksq\x03NX\x05\x00\x00\x00chainq\x04NX\x05\x00\x00\x00chordq\x05Nu\x87q\x06.', 'application/x-python-serialize', 'binary') kwargs:{})
Received task: assets.tasks.test_admin_user_connectability_period[28cbd1e8-7543-404e-b177-09d94155e31f]  
TaskPool: Apply <function _fast_trace_task at 0x7faa22f8aea0> (args:('assets.tasks.test_admin_user_connectability_period', '28cbd1e8-7543-404e-b177-09d94155e31f', {'lang': 'py', 'task': 'assets.tasks.test_admin_user_connectability_period', 'id': '28cbd1e8-7543-404e-b177-09d94155e31f', 'eta': None, 'expires': None, 'group': None, 'retries': 0, 'timelimit': [None, None], 'root_id': '28cbd1e8-7543-404e-b177-09d94155e31f', 'parent_id': None, 'argsrepr': '()', 'kwargsrepr': '{}', 'origin': 'gen22572@h165', 'reply_to': '07fc10f1-4f00-3fe0-b8d1-c98b94e70bb3', 'correlation_id': '28cbd1e8-7543-404e-b177-09d94155e31f', 'delivery_info': {'exchange': '', 'routing_key': 'celery', 'priority': 0, 'redelivered': None}}, b'\x80\x02)}q\x00}q\x01(X\t\x00\x00\x00callbacksq\x02NX\x08\x00\x00\x00errbacksq\x03NX\x05\x00\x00\x00chainq\x04NX\x05\x00\x00\x00chordq\x05Nu\x87q\x06.', 'application/x-python-serialize', 'binary') kwargs:{})
Task accepted: assets.tasks.update_assets_hardware_info_period[aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85] pid:22610
Received task: assets.tasks.test_system_user_connectability_period[ce3795ce-96e2-475f-944f-4cb9cc462cbd]  
TaskPool: Apply <function _fast_trace_task at 0x7faa22f8aea0> (args:('assets.tasks.test_system_user_connectability_period', 'ce3795ce-96e2-475f-944f-4cb9cc462cbd', {'lang': 'py', 'task': 'assets.tasks.test_system_user_connectability_period', 'id': 'ce3795ce-96e2-475f-944f-4cb9cc462cbd', 'eta': None, 'expires': None, 'group': None, 'retries': 0, 'timelimit': [None, None], 'root_id': 'ce3795ce-96e2-475f-944f-4cb9cc462cbd', 'parent_id': None, 'argsrepr': '()', 'kwargsrepr': '{}', 'origin': 'gen22572@h165', 'reply_to': '07fc10f1-4f00-3fe0-b8d1-c98b94e70bb3', 'correlation_id': 'ce3795ce-96e2-475f-944f-4cb9cc462cbd', 'delivery_info': {'exchange': '', 'routing_key': 'celery', 'priority': 0, 'redelivered': None}}, b'\x80\x02)}q\x00}q\x01(X\t\x00\x00\x00callbacksq\x02NX\x08\x00\x00\x00errbacksq\x03NX\x05\x00\x00\x00chainq\x04NX\x05\x00\x00\x00chordq\x05Nu\x87q\x06.', 'application/x-python-serialize', 'binary') kwargs:{})
Task accepted: assets.tasks.test_admin_user_connectability_period[28cbd1e8-7543-404e-b177-09d94155e31f] pid:22609
Received task: terminal.tasks.delete_terminal_status_period[10a7c66d-0877-4fda-bd24-2a9ac3a96e34]  
TaskPool: Apply <function _fast_trace_task at 0x7faa22f8aea0> (args:('terminal.tasks.delete_terminal_status_period', '10a7c66d-0877-4fda-bd24-2a9ac3a96e34', {'lang': 'py', 'task': 'terminal.tasks.delete_terminal_status_period', 'id': '10a7c66d-0877-4fda-bd24-2a9ac3a96e34', 'eta': None, 'expires': None, 'group': None, 'retries': 0, 'timelimit': [None, None], 'root_id': '10a7c66d-0877-4fda-bd24-2a9ac3a96e34', 'parent_id': None, 'argsrepr': '()', 'kwargsrepr': '{}', 'origin': 'gen22572@h165', 'reply_to': '07fc10f1-4f00-3fe0-b8d1-c98b94e70bb3', 'correlation_id': '10a7c66d-0877-4fda-bd24-2a9ac3a96e34', 'delivery_info': {'exchange': '', 'routing_key': 'celery', 'priority': 0, 'redelivered': None}}, b'\x80\x02)}q\x00}q\x01(X\t\x00\x00\x00callbacksq\x02NX\x08\x00\x00\x00errbacksq\x03NX\x05\x00\x00\x00chainq\x04NX\x05\x00\x00\x00chordq\x05Nu\x87q\x06.', 'application/x-python-serialize', 'binary') kwargs:{})
Received task: terminal.tasks.clean_orphan_session[962fa5c4-9e38-47f8-a9e1-d0d49990d691]  
Task accepted: assets.tasks.test_system_user_connectability_period[ce3795ce-96e2-475f-944f-4cb9cc462cbd] pid:22612
Task accepted: terminal.tasks.delete_terminal_status_period[10a7c66d-0877-4fda-bd24-2a9ac3a96e34] pid:22611
Signal handler <function pre_run_task_signal_handler at 0x7faa203da7b8> raised: FileExistsError(17, 'File exists')
Traceback (most recent call last):
  File "/opt/py3/lib/python3.6/site-packages/celery/utils/dispatch/signal.py", line 227, in send
    response = receiver(signal=self, sender=sender, **named)
  File "/opt/jumpserver/apps/ops/celery/signal_handler.py", line 79, in pre_run_task_signal_handler
    os.makedirs(os.path.dirname(full_path))
  File "/usr/local/lib/python3.6/os.py", line 220, in makedirs
    mkdir(name, mode)
FileExistsError: [Errno 17] File exists: '/opt/jumpserver/data/celery/2018-07-23'
2018-07-23 01:52:38 [tasks DEBUG] Period task disabled, test system user connectability pass
Period task disabled, test system user connectability pass
Task assets.tasks.test_system_user_connectability_period[ce3795ce-96e2-475f-944f-4cb9cc462cbd] succeeded in 0.13772281799901975s: None
2018-07-23 01:52:38 [tasks DEBUG] Period task disabled, update assets hardware info pass
Period task disabled, update assets hardware info pass
2018-07-23 01:52:38 [tasks DEBUG] Period task disabled, test admin user connectability pass
Period task disabled, test admin user connectability pass
Task assets.tasks.update_assets_hardware_info_period[aa7c1dc8-54b6-4c6c-94ea-ac60514ecf85] succeeded in 0.14983265699993353s: None
Task assets.tasks.test_admin_user_connectability_period[28cbd1e8-7543-404e-b177-09d94155e31f] succeeded in 0.1483927599983872s: None
Signal handler <function post_run_task_signal_handler at 0x7faa203da840> raised: AttributeError("'test_system_user_connectability_period' object has no attribute 'log_f'",)
Traceback (most recent call last):
  File "/opt/py3/lib/python3.6/site-packages/celery/utils/dispatch/signal.py", line 227, in send
    response = receiver(signal=self, sender=sender, **named)
  File "/opt/jumpserver/apps/ops/celery/signal_handler.py", line 101, in post_run_task_signal_handler
    task.log_f.flush()
AttributeError: 'test_system_user_connectability_period' object has no attribute 'log_f'
Task terminal.tasks.delete_terminal_status_period[10a7c66d-0877-4fda-bd24-2a9ac3a96e34] succeeded in 0.1456284380001307s: None
TaskPool: Apply <function _fast_trace_task at 0x7faa22f8aea0> (args:('terminal.tasks.clean_orphan_session', '962fa5c4-9e38-47f8-a9e1-d0d49990d691', {'lang': 'py', 'task': 'terminal.tasks.clean_orphan_session', 'id': '962fa5c4-9e38-47f8-a9e1-d0d49990d691', 'eta': None, 'expires': None, 'group': None, 'retries': 0, 'timelimit': [None, None], 'root_id': '962fa5c4-9e38-47f8-a9e1-d0d49990d691', 'parent_id': None, 'argsrepr': '()', 'kwargsrepr': '{}', 'origin': 'gen22572@h165', 'reply_to': '07fc10f1-4f00-3fe0-b8d1-c98b94e70bb3', 'correlation_id': '962fa5c4-9e38-47f8-a9e1-d0d49990d691', 'delivery_info': {'exchange': '', 'routing_key': 'celery', 'priority': 0, 'redelivered': None}}, b'\x80\x02)}q\x00}q\x01(X\t\x00\x00\x00callbacksq\x02NX\x08\x00\x00\x00errbacksq\x03NX\x05\x00\x00\x00chainq\x04NX\x05\x00\x00\x00chordq\x05Nu\x87q\x06.', 'application/x-python-serialize', 'binary') kwargs:{})
Task accepted: terminal.tasks.clean_orphan_session[962fa5c4-9e38-47f8-a9e1-d0d49990d691] pid:22612
Task terminal.tasks.clean_orphan_session[962fa5c4-9e38-47f8-a9e1-d0d49990d691] succeeded in 0.12453231399922515s: None
beat: Synchronizing schedule...
Writing entries...
beat: Waking up in 1.00 minute.
...
...
...
~~~


## 进行访问 ##

![jumpserver](/assets/img/jumpserver/jumpserver01.png)

![jumpserver](/assets/img/jumpserver/jumpserver02.png)

## 安装 SSH Server 和 WebSocket Server ##

~~~
[root@h165 ~]# cd /opt
[root@h165 opt]# source /opt/py3/bin/activate
(py3) [root@h165 opt]# git clone https://github.com/jumpserver/coco.git && cd coco && git checkout master
正克隆到 'coco'...
remote: Counting objects: 1588, done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 1588 (delta 11), reused 21 (delta 10), pack-reused 1558
接收对象中: 100% (1588/1588), 334.00 KiB | 302.00 KiB/s, done.
处理 delta 中: 100% (1114/1114), done.
已经位于 'master'
(py3) [root@h165 coco]# echo $?
0
(py3) [root@h165 coco]# echo "source /opt/py3/bin/activate" > /opt/coco/.env
(py3) [root@h165 coco]# 
~~~

## 安装依赖 ##


~~~
(py3) [root@h165 coco]# cd /opt/coco/requirements
(py3) [root@h165 requirements]# ls
requirements.txt  rpm_requirements.txt
(py3) [root@h165 requirements]# yum -y  install $(cat rpm_requirements.txt)
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Package libffi-devel-3.0.13-18.el7.x86_64 already installed and latest version
Package sshpass-1.06-2.el7.x86_64 already installed and latest version
Nothing to do
(py3) [root@h165 requirements]# pip install -r requirements.txt -i https://pypi.org/simple
Requirement already satisfied: asn1crypto==0.24.0 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 1)) (0.24.0)
Requirement already satisfied: bcrypt==3.1.4 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 2)) (3.1.4)
Requirement already satisfied: boto3==1.6.5 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 3)) (1.6.5)
Requirement already satisfied: botocore==1.9.5 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 4)) (1.9.5)
Collecting cachetools==2.0.1 (from -r requirements.txt (line 5))
  Downloading https://files.pythonhosted.org/packages/ac/e8/5492fd5ada0b05a1bc485bcb634b559acdec59383eef5c4203b5e22be296/cachetools-2.0.1-py2.py3-none-any.whl
Requirement already satisfied: certifi==2018.1.18 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 6)) (2018.1.18)
Requirement already satisfied: cffi==1.11.2 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 7)) (1.11.2)
Requirement already satisfied: chardet==3.0.4 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 8)) (3.0.4)
Collecting click==6.7 (from -r requirements.txt (line 9))
  Downloading https://files.pythonhosted.org/packages/34/c1/8806f99713ddb993c5366c362b2f908f18269f8d792aff1abfd700775a77/click-6.7-py2.py3-none-any.whl (71kB)
    100% |████████████████████████████████| 71kB 229kB/s 
Requirement already satisfied: crcmod==1.7 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 10)) (1.7)
Requirement already satisfied: cryptography==2.1.4 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 11)) (2.1.4)
Requirement already satisfied: docutils==0.14 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 12)) (0.14)
Collecting dotmap==1.2.20 (from -r requirements.txt (line 13))
  Downloading https://files.pythonhosted.org/packages/de/1b/797c50035901e9bf7df2e539c7aa81b06e406963bf4f391b532f119bc327/dotmap-1.2.20.tar.gz
Requirement already satisfied: elasticsearch==6.1.1 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 14)) (6.1.1)
Collecting Flask==1.0.2 (from -r requirements.txt (line 15))
  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
    100% |████████████████████████████████| 92kB 517kB/s 
Collecting Flask-SocketIO==2.9.2 (from -r requirements.txt (line 16))
  Downloading https://files.pythonhosted.org/packages/63/76/5aac107e14c5d680b60a312345502ddea101011218492c6d65b919051add/Flask_SocketIO-2.9.2-py2.py3-none-any.whl
Requirement already satisfied: idna==2.6 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 17)) (2.6)
Requirement already satisfied: itsdangerous==0.24 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 18)) (0.24)
Requirement already satisfied: Jinja2==2.10 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 19)) (2.10)
Requirement already satisfied: jmespath==0.9.3 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 20)) (0.9.3)
Requirement already satisfied: jms-storage==0.0.18 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 21)) (0.0.18)
Collecting jumpserver-python-sdk==0.0.44 (from -r requirements.txt (line 22))
  Downloading https://files.pythonhosted.org/packages/ae/6e/9050fc817d4d62fa2189a75d1d95249396106844393715bbb261fef703e2/jumpserver-python-sdk-0.0.44.tar.gz
Requirement already satisfied: MarkupSafe==1.0 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 23)) (1.0)
Requirement already satisfied: oss2==2.4.0 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 24)) (2.4.0)
Requirement already satisfied: paramiko==2.4.0 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 25)) (2.4.0)
Collecting psutil==5.4.1 (from -r requirements.txt (line 26))
  Downloading https://files.pythonhosted.org/packages/fe/17/0f0bf5792b2dfe6003efc5175c76225f7d3426f88e2bf8d360cfab870cd8/psutil-5.4.1.tar.gz (408kB)
    100% |████████████████████████████████| 409kB 637kB/s 
Requirement already satisfied: pyasn1==0.4.2 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 27)) (0.4.2)
Requirement already satisfied: pycparser==2.18 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 28)) (2.18)
Requirement already satisfied: PyNaCl==1.2.1 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 29)) (1.2.1)
Collecting pyte==0.8.0 (from -r requirements.txt (line 30))
  Downloading https://files.pythonhosted.org/packages/66/37/6fed89b484c8012a0343117f085c92df8447a18af4966d25599861cd5aa0/pyte-0.8.0.tar.gz (50kB)
    100% |████████████████████████████████| 51kB 9.0MB/s 
Requirement already satisfied: python-dateutil==2.6.1 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 31)) (2.6.1)
Collecting python-engineio==2.1.0 (from -r requirements.txt (line 32))
  Downloading https://files.pythonhosted.org/packages/ab/87/a43279428b8ac8ebd0f7d4959c675ec48c9aed5f6a90e5befbc7c8252b6a/python_engineio-2.1.0-py2.py3-none-any.whl
Requirement already satisfied: python-gssapi==0.6.4 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 33)) (0.6.4)
Collecting python-socketio==1.8.3 (from -r requirements.txt (line 34))
  Downloading https://files.pythonhosted.org/packages/54/80/ee4dab6e14c749f8591b6aabc7e503147bac35791c6638626f0fd39cda3f/python_socketio-1.8.3-py2.py3-none-any.whl
Requirement already satisfied: pytz==2018.3 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 35)) (2018.3)
Requirement already satisfied: requests==2.18.4 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 36)) (2.18.4)
Requirement already satisfied: s3transfer==0.1.13 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 37)) (0.1.13)
Requirement already satisfied: simplejson==3.13.2 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 38)) (3.13.2)
Requirement already satisfied: six==1.11.0 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 39)) (1.11.0)
Collecting tornado==4.5.2 (from -r requirements.txt (line 40))
  Downloading https://files.pythonhosted.org/packages/fa/14/52e2072197dd0e63589e875ebf5984c91a027121262aa08f71a49b958359/tornado-4.5.2.tar.gz (483kB)
    100% |████████████████████████████████| 491kB 863kB/s 
Requirement already satisfied: urllib3==1.22 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 41)) (1.22)
Collecting wcwidth==0.1.7 (from -r requirements.txt (line 42))
  Downloading https://files.pythonhosted.org/packages/7e/9f/526a6947247599b084ee5232e4f9190a38f398d7300d866af3ab571a5bfe/wcwidth-0.1.7-py2.py3-none-any.whl
Requirement already satisfied: eventlet==0.22.1 in /opt/py3/lib/python3.6/site-packages (from -r requirements.txt (line 43)) (0.22.1)
Collecting Werkzeug==0.14.1 (from -r requirements.txt (line 44))
  Downloading https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
    100% |████████████████████████████████| 327kB 891kB/s 
Requirement already satisfied: greenlet>=0.3 in /opt/py3/lib/python3.6/site-packages (from eventlet==0.22.1->-r requirements.txt (line 43)) (0.4.12)
Installing collected packages: cachetools, click, dotmap, Werkzeug, Flask, python-engineio, python-socketio, Flask-SocketIO, wcwidth, pyte, jumpserver-python-sdk, psutil, tornado
  Running setup.py install for dotmap ... done
  Running setup.py install for pyte ... done
  Running setup.py install for jumpserver-python-sdk ... done
  Running setup.py install for psutil ... done
  Running setup.py install for tornado ... done
Successfully installed Flask-1.0.2 Flask-SocketIO-2.9.2 Werkzeug-0.14.1 cachetools-2.0.1 click-6.7 dotmap-1.2.20 jumpserver-python-sdk-0.0.44 psutil-5.4.1 pyte-0.8.0 python-engineio-2.1.0 python-socketio-1.8.3 tornado-4.5.2 wcwidth-0.1.7
(py3) [root@h165 requirements]# echo $?
0
(py3) [root@h165 requirements]#
~~~

## 修改 coco 配置文件 ##

~~~
(py3) [root@h165 requirements]# cd /opt/coco
(py3) [root@h165 coco]# cp conf_example.py conf.py
(py3) [root@h165 coco]# vi conf.py 
(py3) [root@h165 coco]# cat conf.py 
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
#

import os

BASE_DIR = os.path.dirname(__file__)


class Config:
    """
    Coco config file, coco also load config from server update setting below
    """
    # 项目名称, 会用来向Jumpserver注册, 识别而已, 不能重复
    # NAME = "localhost"
    NAME = "coco"

    # Jumpserver项目的url, api请求注册会使用
    # CORE_HOST = os.environ.get("CORE_HOST") or 'http://127.0.0.1:8080'
    CORE_HOST = os.environ.get("CORE_HOST") or 'http://127.0.0.1:8080'

    # 启动时绑定的ip, 默认 0.0.0.0
    # BIND_HOST = '0.0.0.0'

    # 监听的SSH端口号, 默认2222
    # SSHD_PORT = 2222

    # 监听的HTTP/WS端口号，默认5000
    # HTTPD_PORT = 5000

    # 项目使用的ACCESS KEY, 默认会注册,并保存到 ACCESS_KEY_STORE中,
    # 如果有需求, 可以写到配置文件中, 格式 access_key_id:access_key_secret
    # ACCESS_KEY = None

    # ACCESS KEY 保存的地址, 默认注册后会保存到该文件中
    # ACCESS_KEY_STORE = os.path.join(BASE_DIR, 'keys', '.access_key')

    # 加密密钥
    # SECRET_KEY = None

    # 设置日志级别 ['DEBUG', 'INFO', 'WARN', 'ERROR', 'FATAL', 'CRITICAL']
    # LOG_LEVEL = 'INFO'

    # 日志存放的目录
    # LOG_DIR = os.path.join(BASE_DIR, 'logs')

    # Session录像存放目录
    # SESSION_DIR = os.path.join(BASE_DIR, 'sessions')

    # 资产显示排序方式, ['ip', 'hostname']
    # ASSET_LIST_SORT_BY = 'ip'

    # 登录是否支持密码认证
    # PASSWORD_AUTH = True

    # 登录是否支持秘钥认证
    # PUBLIC_KEY_AUTH = True

    # 和Jumpserver 保持心跳时间间隔
    # HEARTBEAT_INTERVAL = 5

    # Admin的名字，出问题会提示给用户
    # ADMINS = ''
    COMMAND_STORAGE = {
        "TYPE": "server"
    }
    REPLAY_STORAGE = {
        "TYPE": "server"
    }


config = Config()
(py3) [root@h165 coco]# 
~~~

## 运行 coco ##

~~~
(py3) [root@h165 coco]# ./cocod start
Start coco process
2018-07-22 18:12:23 [service DEBUG] Initial app service
2018-07-22 18:12:23 [service DEBUG] Load access key
2018-07-22 18:12:23 [service INFO] No access key found, register it
2018-07-22 18:12:23 [service INFO] "Terminal was not accepted yet"
2018-07-22 18:12:26 [service INFO] "Terminal was not accepted yet"
...
...
...
~~~



![jumpserver](/assets/img/jumpserver/jumpserver03.png)


## 安装 Web Terminal 前端 ##

~~~
[root@h165 ~]# cd /opt
[root@h165 opt]# wget https://github.com/jumpserver/luna/releases/download/1.3.3/luna.tar.gz
--2018-07-22 18:16:49--  https://github.com/jumpserver/luna/releases/download/1.3.3/luna.tar.gz
正在解析主机 github.com (github.com)... 52.74.223.119, 13.229.188.59, 13.250.177.223
正在连接 github.com (github.com)|52.74.223.119|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 302 Found
位置：https://github-production-release-asset-2e65be.s3.amazonaws.com/83748317/c44beea0-8b4f-11e8-8056-75da7b314d1f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20180722%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180722T181649Z&X-Amz-Expires=300&X-Amz-Signature=630ec697ea90ec5185482908192bef07ffe33d4e3430868db47b17c7ccf9f61b&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dluna.tar.gz&response-content-type=application%2Foctet-stream [跟随至新的 URL]
--2018-07-22 18:16:50--  https://github-production-release-asset-2e65be.s3.amazonaws.com/83748317/c44beea0-8b4f-11e8-8056-75da7b314d1f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20180722%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180722T181649Z&X-Amz-Expires=300&X-Amz-Signature=630ec697ea90ec5185482908192bef07ffe33d4e3430868db47b17c7ccf9f61b&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dluna.tar.gz&response-content-type=application%2Foctet-stream
正在解析主机 github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 54.231.97.232
正在连接 github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|54.231.97.232|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：5335554 (5.1M) [application/octet-stream]
正在保存至: “luna.tar.gz”

100%[======================================>] 5,335,554    229KB/s 用时 27s    

2018-07-22 18:17:18 (192 KB/s) - 已保存 “luna.tar.gz” [5335554/5335554])

[root@h165 opt]# tar xvf luna.tar.gz
luna/
luna/fontawesome-webfont.912ec66d7572ff821749.svg
luna/OpenSans-Semibold.08952b029e4decbc8ef9.woff2
luna/OpenSans-Semibold.bb100c995f1d20b8a964.svg
luna/OpenSans-Light.39d27e13dce3dfe4cdc7.woff2
luna/OpenSans-BoldItalic.5aaceea2d60ddb477c6a.woff2
luna/styles.c46837e1b0ec984edfd7.bundle.css
luna/OpenSans-ExtraBoldItalic.4595d7f8ce0e7b381abb.ttf
luna/favicon.ico
luna/index.html
luna/OpenSans-Bold.892667349c5cff6fcf7e.woff
luna/OpenSans-Regular.a35546eef3ea0de0d473.eot
luna/OpenSans-Italic.e487b7cb072550896dde.eot
luna/data-table.bce071e976865da51100.eot
luna/fontawesome-webfont.af7ae505a9eed503f8b8.woff2
luna/OpenSans-BoldItalic.7be88e73fea7b64568a4.woff
luna/OpenSans-BoldItalic.a54aba83b3d5d7702890.svg
luna/OpenSans-ExtraBoldItalic.4f44077586ec12a35ce6.woff
luna/OpenSans-SemiboldItalic.1c0b4eb93fcf561eec03.ttf
luna/3rdpartylicenses.txt
luna/OpenSans-Bold.d6291f88056601e360ce.svg
luna/OpenSans-ExtraBold.12e2ed7a180e601bff44.woff
luna/MaterialIcons-Regular.e79bfd88537def476913.eot
luna/OpenSans-Light.ecb4572a5e478b107dfc.ttf
luna/fontawesome-webfont.674f50d287a8c48dc19b.eot
luna/OpenSans-Semibold.33f225b8f5f7d6b34a09.ttf
luna/polyfills.a77ad0f66aa4cf1a6a23.bundle.js
luna/OpenSans-Semibold.9f2144213fad53d4e0fd.woff
luna/OpenSans-Italic.383eba0e55ed778006d7.woff2
luna/OpenSans-ExtraBold.8c5c497a47304f276f99.svg
luna/OpenSans-SemiboldItalic.ec55f263e2b86bc0f28f.woff
luna/OpenSans-LightItalic.6725fc490942895a65f5.eot
luna/OpenSans-LightItalic.b64e9910811cdcc8df89.svg
luna/OpenSans-Regular.ac327c4db6284ef64ebe.woff
luna/OpenSans-Light.963eb32907744d9a0d6b.woff
luna/OpenSans-LightItalic.e7cc7120e670a8073073.woff2
luna/OpenSans-SemiboldItalic.3343e54368719e3786f7.woff2
luna/OpenSans-BoldItalic.c36b5ac7c2dddf6f525c.ttf
luna/OpenSans-BoldItalic.ea07932c5245dd421e3d.eot
luna/OpenSans-Regular.cd7296352d159532b66c.ttf
luna/OpenSans-ExtraBoldItalic.5f467e780ed0aead6614.eot
luna/OpenSans-ExtraBold.19b56cfcb97fbcc24524.ttf
luna/fontawesome-webfont.b06871f281fee6b241d6.ttf
luna/OpenSans-LightItalic.97534dd409492b05b11a.woff
luna/OpenSans-Semibold.0ea04502930623aa3de1.eot
luna/OpenSans-SemiboldItalic.da061416028fc9a66fbc.eot
luna/OpenSans-LightItalic.26f1e68dfbd8b8621e5d.ttf
luna/OpenSans-Light.804037562eabaa5dbefa.eot
luna/theme/
luna/static/
luna/MaterialIcons-Regular.570eb83859dc23dd0eec.woff2
luna/OpenSans-Regular.f641a7d4e80fd6321135.svg
luna/OpenSans-Italic.9b30f13428e1b4a659ae.ttf
luna/OpenSans-ExtraBoldItalic.bc511bacf828ac9e833e.woff2
luna/OpenSans-SemiboldItalic.ddc348f204283c4f4090.svg
luna/OpenSans-ExtraBold.561e4b63e9119235465e.eot
luna/OpenSans-ExtraBoldItalic.9704305e6fd8184b40d5.svg
luna/OpenSans-Light.d79f021974b1f6bc5c21.svg
luna/OpenSans-Bold.3326e4d74d3924ee1c88.woff2
luna/main.a3c75c0a3047ed1eef01.bundle.js
luna/OpenSans-Italic.d6671d41dde41d355619.svg
luna/OpenSans-ExtraBold.5211065d7cf88c28086d.woff2
luna/OpenSans-Bold.5a100916f94b0babde0c.ttf
luna/OpenSans-Bold.7ae9b8ba7886341831bf.eot
luna/scripts.56fb3235bc9c0e75506e.bundle.js
luna/i18n/
luna/fontawesome-webfont.fee66e712a8a08eef580.woff
luna/inline.6602a1f82e32b48c37e4.bundle.js
luna/OpenSans-Regular.55835483c304eaa8477f.woff2
luna/data-table.b0aebd744ce7adb780a9.svg
luna/OpenSans-Italic.525074686dfb8aa36b1b.woff
luna/MaterialIcons-Regular.012cf6a10129e2275d79.woff
luna/MaterialIcons-Regular.a37b0c01c0baf1888ca8.ttf
luna/i18n/zh.json
luna/i18n/cn.json
luna/i18n/zh-CN.json
luna/static/imgs/
luna/static/imgs/inspinia/
luna/static/imgs/logo.png
luna/static/imgs/logo-text.png
luna/static/imgs/inspinia/header-profile-skin-2.png
luna/static/imgs/inspinia/header-profile-skin-3.png
luna/static/imgs/inspinia/header-profile-skin-1.png
luna/static/imgs/inspinia/header-profile.png
luna/static/imgs/inspinia/4.png
luna/static/imgs/inspinia/5.png
luna/static/imgs/inspinia/7.png
luna/static/imgs/inspinia/shattered.png
luna/static/imgs/inspinia/6.png
luna/static/imgs/inspinia/2.png
luna/static/imgs/inspinia/3.png
luna/static/imgs/inspinia/1.png
luna/theme/default/
luna/theme/default/layer.css
[root@h165 opt]# chown -R root:root luna
[root@h165 opt]# 
~~~


## 配置 Nginx 整合各组件 ##

~~~
[root@h165 opt]# yum -y install nginx
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package nginx.x86_64 1:1.12.2-2.el7 will be installed
--> Processing Dependency: nginx-filesystem = 1:1.12.2-2.el7 for package: 1:nginx-1.12.2-2.el7.x86_64
--> Processing Dependency: nginx-all-modules = 1:1.12.2-2.el7 for package: 1:nginx-1.12.2-2.el7.x86_64
--> Processing Dependency: nginx-filesystem for package: 1:nginx-1.12.2-2.el7.x86_64
--> Processing Dependency: libprofiler.so.0()(64bit) for package: 1:nginx-1.12.2-2.el7.x86_64
--> Running transaction check
---> Package gperftools-libs.x86_64 0:2.6.1-1.el7 will be installed
---> Package nginx-all-modules.noarch 1:1.12.2-2.el7 will be installed
--> Processing Dependency: nginx-mod-stream = 1:1.12.2-2.el7 for package: 1:nginx-all-modules-1.12.2-2.el7.noarch
--> Processing Dependency: nginx-mod-mail = 1:1.12.2-2.el7 for package: 1:nginx-all-modules-1.12.2-2.el7.noarch
--> Processing Dependency: nginx-mod-http-xslt-filter = 1:1.12.2-2.el7 for package: 1:nginx-all-modules-1.12.2-2.el7.noarch
--> Processing Dependency: nginx-mod-http-perl = 1:1.12.2-2.el7 for package: 1:nginx-all-modules-1.12.2-2.el7.noarch
--> Processing Dependency: nginx-mod-http-image-filter = 1:1.12.2-2.el7 for package: 1:nginx-all-modules-1.12.2-2.el7.noarch
--> Processing Dependency: nginx-mod-http-geoip = 1:1.12.2-2.el7 for package: 1:nginx-all-modules-1.12.2-2.el7.noarch
---> Package nginx-filesystem.noarch 1:1.12.2-2.el7 will be installed
--> Running transaction check
---> Package nginx-mod-http-geoip.x86_64 1:1.12.2-2.el7 will be installed
---> Package nginx-mod-http-image-filter.x86_64 1:1.12.2-2.el7 will be installed
--> Processing Dependency: gd for package: 1:nginx-mod-http-image-filter-1.12.2-2.el7.x86_64
--> Processing Dependency: libgd.so.2()(64bit) for package: 1:nginx-mod-http-image-filter-1.12.2-2.el7.x86_64
---> Package nginx-mod-http-perl.x86_64 1:1.12.2-2.el7 will be installed
---> Package nginx-mod-http-xslt-filter.x86_64 1:1.12.2-2.el7 will be installed
--> Processing Dependency: libxslt.so.1(LIBXML2_1.0.18)(64bit) for package: 1:nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64
--> Processing Dependency: libxslt.so.1(LIBXML2_1.0.11)(64bit) for package: 1:nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64
--> Processing Dependency: libxslt.so.1()(64bit) for package: 1:nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64
--> Processing Dependency: libexslt.so.0()(64bit) for package: 1:nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64
---> Package nginx-mod-mail.x86_64 1:1.12.2-2.el7 will be installed
---> Package nginx-mod-stream.x86_64 1:1.12.2-2.el7 will be installed
--> Running transaction check
---> Package gd.x86_64 0:2.0.35-26.el7 will be installed
--> Processing Dependency: libpng15.so.15(PNG15_0)(64bit) for package: gd-2.0.35-26.el7.x86_64
--> Processing Dependency: libpng15.so.15()(64bit) for package: gd-2.0.35-26.el7.x86_64
--> Processing Dependency: libXpm.so.4()(64bit) for package: gd-2.0.35-26.el7.x86_64
---> Package libxslt.x86_64 0:1.1.28-5.el7 will be installed
--> Running transaction check
---> Package libXpm.x86_64 0:3.5.12-1.el7 will be installed
---> Package libpng.x86_64 2:1.5.13-7.el7_2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                         Arch       Version              Repository
                                                                           Size
================================================================================
Installing:
 nginx                           x86_64     1:1.12.2-2.el7       epel     530 k
Installing for dependencies:
 gd                              x86_64     2.0.35-26.el7        base     146 k
 gperftools-libs                 x86_64     2.6.1-1.el7          base     272 k
 libXpm                          x86_64     3.5.12-1.el7         base      55 k
 libpng                          x86_64     2:1.5.13-7.el7_2     base     213 k
 libxslt                         x86_64     1.1.28-5.el7         base     242 k
 nginx-all-modules               noarch     1:1.12.2-2.el7       epel      16 k
 nginx-filesystem                noarch     1:1.12.2-2.el7       epel      17 k
 nginx-mod-http-geoip            x86_64     1:1.12.2-2.el7       epel      23 k
 nginx-mod-http-image-filter     x86_64     1:1.12.2-2.el7       epel      26 k
 nginx-mod-http-perl             x86_64     1:1.12.2-2.el7       epel      36 k
 nginx-mod-http-xslt-filter      x86_64     1:1.12.2-2.el7       epel      26 k
 nginx-mod-mail                  x86_64     1:1.12.2-2.el7       epel      54 k
 nginx-mod-stream                x86_64     1:1.12.2-2.el7       epel      76 k

Transaction Summary
================================================================================
Install  1 Package (+13 Dependent packages)

Total download size: 1.7 M
Installed size: 4.9 M
Downloading packages:
(1/14): libXpm-3.5.12-1.el7.x86_64.rpm                     |  55 kB   00:00     
(2/14): gd-2.0.35-26.el7.x86_64.rpm                        | 146 kB   00:00     
(3/14): gperftools-libs-2.6.1-1.el7.x86_64.rpm             | 272 kB   00:00     
(4/14): libxslt-1.1.28-5.el7.x86_64.rpm                    | 242 kB   00:00     
(5/14): nginx-filesystem-1.12.2-2.el7.noarch.rpm           |  17 kB   00:00     
(6/14): nginx-mod-http-image-filter-1.12.2-2.el7.x86_64.rp |  26 kB   00:00     
(7/14): nginx-mod-http-perl-1.12.2-2.el7.x86_64.rpm        |  36 kB   00:00     
(8/14): nginx-1.12.2-2.el7.x86_64.rpm                      | 530 kB   00:00     
(9/14): nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64.rpm |  26 kB   00:00     
(10/14): nginx-mod-mail-1.12.2-2.el7.x86_64.rpm            |  54 kB   00:00     
(11/14): nginx-mod-stream-1.12.2-2.el7.x86_64.rpm          |  76 kB   00:00     
(12/14): nginx-mod-http-geoip-1.12.2-2.el7.x86_64.rpm      |  23 kB   00:00     
(13/14): nginx-all-modules-1.12.2-2.el7.noarch.rpm         |  16 kB   00:03     
(14/14): libpng-1.5.13-7.el7_2.x86_64.rpm                  | 213 kB   00:05     
--------------------------------------------------------------------------------
Total                                              298 kB/s | 1.7 MB  00:05     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : gperftools-libs-2.6.1-1.el7.x86_64                          1/14 
  Installing : libxslt-1.1.28-5.el7.x86_64                                 2/14 
  Installing : 2:libpng-1.5.13-7.el7_2.x86_64                              3/14 
  Installing : 1:nginx-filesystem-1.12.2-2.el7.noarch                      4/14 
  Installing : libXpm-3.5.12-1.el7.x86_64                                  5/14 
  Installing : gd-2.0.35-26.el7.x86_64                                     6/14 
  Installing : 1:nginx-mod-mail-1.12.2-2.el7.x86_64                        7/14 
  Installing : 1:nginx-mod-http-geoip-1.12.2-2.el7.x86_64                  8/14 
  Installing : 1:nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64            9/14 
  Installing : 1:nginx-mod-http-perl-1.12.2-2.el7.x86_64                  10/14 
  Installing : 1:nginx-mod-stream-1.12.2-2.el7.x86_64                     11/14 
  Installing : 1:nginx-1.12.2-2.el7.x86_64                                12/14 
  Installing : 1:nginx-mod-http-image-filter-1.12.2-2.el7.x86_64          13/14 
  Installing : 1:nginx-all-modules-1.12.2-2.el7.noarch                    14/14 
  Verifying  : libXpm-3.5.12-1.el7.x86_64                                  1/14 
  Verifying  : 1:nginx-filesystem-1.12.2-2.el7.noarch                      2/14 
  Verifying  : gd-2.0.35-26.el7.x86_64                                     3/14 
  Verifying  : 2:libpng-1.5.13-7.el7_2.x86_64                              4/14 
  Verifying  : libxslt-1.1.28-5.el7.x86_64                                 5/14 
  Verifying  : gperftools-libs-2.6.1-1.el7.x86_64                          6/14 
  Verifying  : 1:nginx-1.12.2-2.el7.x86_64                                 7/14 
  Verifying  : 1:nginx-mod-mail-1.12.2-2.el7.x86_64                        8/14 
  Verifying  : 1:nginx-all-modules-1.12.2-2.el7.noarch                     9/14 
  Verifying  : 1:nginx-mod-http-geoip-1.12.2-2.el7.x86_64                 10/14 
  Verifying  : 1:nginx-mod-http-xslt-filter-1.12.2-2.el7.x86_64           11/14 
  Verifying  : 1:nginx-mod-http-image-filter-1.12.2-2.el7.x86_64          12/14 
  Verifying  : 1:nginx-mod-http-perl-1.12.2-2.el7.x86_64                  13/14 
  Verifying  : 1:nginx-mod-stream-1.12.2-2.el7.x86_64                     14/14 

Installed:
  nginx.x86_64 1:1.12.2-2.el7                                                   

Dependency Installed:
  gd.x86_64 0:2.0.35-26.el7                                                     
  gperftools-libs.x86_64 0:2.6.1-1.el7                                          
  libXpm.x86_64 0:3.5.12-1.el7                                                  
  libpng.x86_64 2:1.5.13-7.el7_2                                                
  libxslt.x86_64 0:1.1.28-5.el7                                                 
  nginx-all-modules.noarch 1:1.12.2-2.el7                                       
  nginx-filesystem.noarch 1:1.12.2-2.el7                                        
  nginx-mod-http-geoip.x86_64 1:1.12.2-2.el7                                    
  nginx-mod-http-image-filter.x86_64 1:1.12.2-2.el7                             
  nginx-mod-http-perl.x86_64 1:1.12.2-2.el7                                     
  nginx-mod-http-xslt-filter.x86_64 1:1.12.2-2.el7                              
  nginx-mod-mail.x86_64 1:1.12.2-2.el7                                          
  nginx-mod-stream.x86_64 1:1.12.2-2.el7                                        

Complete!
[root@h165 opt]# 
~~~


## 配置 nginx ##

~~~
[root@h165 ~]# vi /etc/nginx/nginx.conf
[root@h165 ~]# nginx -t -c /etc/nginx/nginx.conf
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@h165 ~]# cat /etc/nginx/nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {

    listen 80;  # 代理端口，以后将通过此端口进行访问，不再通过8080端口

    location /luna/ {
        try_files $uri / /index.html;
        alias /opt/luna/;  # luna 路径，如果修改安装目录，此处需要修改
    }

    location /media/ {
        add_header Content-Encoding gzip;
        root /opt/jumpserver/data/;  # 录像位置，如果修改安装目录，此处需要修改
    }

    location /static/ {
        root /opt/jumpserver/data/;  # 静态资源，如果修改安装目录，此处需要修改
    }

    location /socket.io/ {
        proxy_pass       http://localhost:5000/socket.io/;  # 如果coco安装在别的服务器，请填写它的ip
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /guacamole/ {
        proxy_pass       http://localhost:8081/;  # 如果guacamole安装在别的服务器，请填写它的ip
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
        client_max_body_size 100m;  # Windows 文件上传大小限制
    }

    location / {
        proxy_pass http://localhost:8080;  # 如果jumpserver安装在别的服务器，请填写它的ip
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }        










    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}

[root@h165 ~]# 
~~~

## 运行 Nginx ##

~~~
[root@h165 ~]# systemctl start nginx
[root@h165 ~]# systemctl enable nginx
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
[root@h165 ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-07-22 18:27:24 UTC; 9s ago
 Main PID: 23715 (nginx)
   CGroup: /system.slice/nginx.service
           ├─23715 nginx: master process /usr/sbin/nginx
           ├─23716 nginx: worker process
           ├─23717 nginx: worker process
           ├─23718 nginx: worker process
           └─23719 nginx: worker process

7月 22 18:27:24 h165 systemd[1]: Starting The nginx HTTP and reverse proxy.....
7月 22 18:27:24 h165 nginx[23710]: nginx: the configuration file /etc/ngin...ok
7月 22 18:27:24 h165 nginx[23710]: nginx: configuration file /etc/nginx/ng...ul
7月 22 18:27:24 h165 systemd[1]: Started The nginx HTTP and reverse proxy ...r.
Hint: Some lines were ellipsized, use -l to show in full.
[root@h165 ~]#
~~~

## 再次进行登录 ##

![jumpserver](/assets/img/jumpserver/jumpserver04.png)

## 接受连接 ##


![jumpserver](/assets/img/jumpserver/jumpserver05.png)

![jumpserver](/assets/img/jumpserver/jumpserver06.png)

![jumpserver](/assets/img/jumpserver/jumpserver07.png)


~~~
...
...
...
2018-07-22 18:36:09 [service INFO] "Terminal was not accepted yet"
2018-07-22 18:36:12 [service INFO] "Terminal was not accepted yet"
2018-07-22 18:36:15 [service INFO] "Terminal was not accepted yet"
2018-07-22 18:36:18 [service DEBUG] Set app service auth: cd39df5d-a40b-4d8c-bd4c-f5617bbaa5e2
2018-07-22 18:36:18 [service DEBUG] Service http auth: <jms.auth.AccessKeyAuth object at 0x7fdf3c677cf8>
2018-07-22 18:36:18 [app DEBUG] Loading config from server: {"COMMAND_STORAGE": {"TYPE": "server"}, "REPLAY_STORAGE": {"TYPE": "server"}}
Sun Jul 22 18:36:18 2018
Coco version 1.3.3, more see https://www.jumpserver.org
Quit the server with CONTROL-C.
Starting ssh server at 0.0.0.0:2222
Starting websocket server at 0.0.0.0:5000
...
...
...
~~~

## 通过 cli 进行管理 ##

~~~
[vagrant@h105 ~]$ ssh -p2222 admin@192.168.56.165
admin@192.168.56.165's password: 
...
...
...
~~~

默认密码为 admin/admin

~~~
    Administrator, 欢迎使用Jumpserver开源跳板机系统  

    1) 输入 ID 直接登录 或 输入部分 IP,主机名,备注 进行搜索登录(如果唯一).
    2) 输入 / + IP, 主机名 or 备注 搜索. 如: /ip
    3) 输入 p 显示您有权限的主机.
    4) 输入 g 显示您有权限的节点
    5) 输入 g + 组ID 显示节点下主机. 如: g1
    6) 输入 h 帮助.
    0) 输入 q 退出.

Opt> 
~~~

密码输入正确后，就有相应的登录界面了



## 安装 Windows 支持组件 ##


### 首先安装 docker ###

~~~
[root@h165 ~]# yum remove docker-latest-logrotate  docker-logrotate  docker-selinux dockdocker-engine
Loaded plugins: fastestmirror
No Match for argument: docker-latest-logrotate
No Match for argument: docker-logrotate
No Match for argument: docker-selinux
No Match for argument: dockdocker-engine
No Packages marked for removal
[root@h165 ~]# yum install -y yum-utils   device-mapper-persistent-data   lvm2
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Package yum-utils-1.1.31-45.el7.noarch already installed and latest version
Package device-mapper-persistent-data-0.7.3-3.el7.x86_64 already installed and latest version
Package 7:lvm2-2.02.177-4.el7.x86_64 already installed and latest version
Nothing to do
[root@h165 ~]# yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@h165 ~]# yum makecache fast
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                     | 6.1 kB     00:00     
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
base                                                     | 3.6 kB     00:00     
docker-ce-stable                                         | 2.9 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
docker-ce-stable/x86_64/primary_db                         |  14 kB   00:00     
Metadata Cache Created
[root@h165 ~]# yum install docker-ce
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 0:18.06.0.ce-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-18.06.0.ce-3.el7.x86_64
--> Processing Dependency: libcgroup for package: docker-ce-18.06.0.ce-3.el7.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-ce-18.06.0.ce-3.el7.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.66-1.el7 will be installed
--> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.66-1.el7.noarch
---> Package libcgroup.x86_64 0:0.41-15.el7 will be installed
---> Package libtool-ltdl.x86_64 0:2.4.2-22.el7_3 will be installed
--> Running transaction check
---> Package policycoreutils-python.x86_64 0:2.5-22.el7 will be installed
--> Processing Dependency: setools-libs >= 3.3.8-2 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-9 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.1-3.el7 will be installed
---> Package checkpolicy.x86_64 0:2.5-6.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-11.el7 will be installed
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.8-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                  Arch     Version             Repository          Size
================================================================================
Installing:
 docker-ce                x86_64   18.06.0.ce-3.el7    docker-ce-stable    41 M
Installing for dependencies:
 audit-libs-python        x86_64   2.8.1-3.el7         base                75 k
 checkpolicy              x86_64   2.5-6.el7           base               294 k
 container-selinux        noarch   2:2.66-1.el7        extras              35 k
 libcgroup                x86_64   0.41-15.el7         base                65 k
 libsemanage-python       x86_64   2.5-11.el7          base               112 k
 libtool-ltdl             x86_64   2.4.2-22.el7_3      base                49 k
 policycoreutils-python   x86_64   2.5-22.el7          base               454 k
 python-IPy               noarch   0.75-6.el7          base                32 k
 setools-libs             x86_64   3.3.8-2.el7         base               619 k

Transaction Summary
================================================================================
Install  1 Package (+9 Dependent packages)

Total download size: 42 M
Installed size: 174 M
Is this ok [y/d/N]: y
Downloading packages:
(1/10): container-selinux-2.66-1.el7.noarch.rpm            |  35 kB   00:00     
(2/10): audit-libs-python-2.8.1-3.el7.x86_64.rpm           |  75 kB   00:00     
(3/10): libcgroup-0.41-15.el7.x86_64.rpm                   |  65 kB   00:00     
(4/10): libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm             |  49 kB   00:00     
(5/10): python-IPy-0.75-6.el7.noarch.rpm                   |  32 kB   00:00     
(6/10): checkpolicy-2.5-6.el7.x86_64.rpm                   | 294 kB   00:00     
(7/10): libsemanage-python-2.5-11.el7.x86_64.rpm           | 112 kB   00:00     
(8/10): policycoreutils-python-2.5-22.el7.x86_64.rpm       | 454 kB   00:01     
(9/10): setools-libs-3.3.8-2.el7.x86_64.rpm                | 619 kB   00:01     
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-18.06.0.ce-3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Public key for docker-ce-18.06.0.ce-3.el7.x86_64.rpm is not installed
(10/10): docker-ce-18.06.0.ce-3.el7.x86_64.rpm             |  41 MB   00:21     
--------------------------------------------------------------------------------
Total                                              2.0 MB/s |  42 MB  00:21     
Retrieving key from https://download.docker.com/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libcgroup-0.41-15.el7.x86_64                                1/10 
  Installing : checkpolicy-2.5-6.el7.x86_64                                2/10 
  Installing : audit-libs-python-2.8.1-3.el7.x86_64                        3/10 
  Installing : libtool-ltdl-2.4.2-22.el7_3.x86_64                          4/10 
  Installing : python-IPy-0.75-6.el7.noarch                                5/10 
  Installing : libsemanage-python-2.5-11.el7.x86_64                        6/10 
  Installing : setools-libs-3.3.8-2.el7.x86_64                             7/10 
  Installing : policycoreutils-python-2.5-22.el7.x86_64                    8/10 
  Installing : 2:container-selinux-2.66-1.el7.noarch                       9/10 
  Installing : docker-ce-18.06.0.ce-3.el7.x86_64                          10/10 
  Verifying  : libcgroup-0.41-15.el7.x86_64                                1/10 
  Verifying  : docker-ce-18.06.0.ce-3.el7.x86_64                           2/10 
  Verifying  : setools-libs-3.3.8-2.el7.x86_64                             3/10 
  Verifying  : policycoreutils-python-2.5-22.el7.x86_64                    4/10 
  Verifying  : libsemanage-python-2.5-11.el7.x86_64                        5/10 
  Verifying  : python-IPy-0.75-6.el7.noarch                                6/10 
  Verifying  : libtool-ltdl-2.4.2-22.el7_3.x86_64                          7/10 
  Verifying  : audit-libs-python-2.8.1-3.el7.x86_64                        8/10 
  Verifying  : 2:container-selinux-2.66-1.el7.noarch                       9/10 
  Verifying  : checkpolicy-2.5-6.el7.x86_64                               10/10 

Installed:
  docker-ce.x86_64 0:18.06.0.ce-3.el7                                           

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.1-3.el7                                        
  checkpolicy.x86_64 0:2.5-6.el7                                                
  container-selinux.noarch 2:2.66-1.el7                                         
  libcgroup.x86_64 0:0.41-15.el7                                                
  libsemanage-python.x86_64 0:2.5-11.el7                                        
  libtool-ltdl.x86_64 0:2.4.2-22.el7_3                                          
  policycoreutils-python.x86_64 0:2.5-22.el7                                    
  python-IPy.noarch 0:0.75-6.el7                                                
  setools-libs.x86_64 0:3.3.8-2.el7                                             

Complete!
[root@h165 ~]# echo $?
0
[root@h165 ~]# 
~~~


### 然后启动 docker ###

~~~
[root@h165 ~]# systemctl start docker
[root@h165 ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since 日 2018-07-22 18:58:56 UTC; 4s ago
     Docs: https://docs.docker.com
 Main PID: 24622 (dockerd)
    Tasks: 26
   Memory: 46.6M
   CGroup: /system.slice/docker.service
           ├─24622 /usr/bin/dockerd
           └─24630 docker-containerd --config /var/run/docker/containerd/cont...

7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.421854451Z"...pc
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.421897175Z"...pc
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.423786369Z"...pc
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.423851725Z"...."
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.574771896Z"...s"
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.623893983Z"...."
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.643650687Z"...ce
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.644194232Z"...n"
7月 22 18:58:56 h165 dockerd[24622]: time="2018-07-22T18:58:56.663163741Z"...k"
7月 22 18:58:56 h165 systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
[root@h165 ~]# 
~~~

### 创建 Guacamole 的容器 ###


~~~
[root@h165 ~]# docker run --name jms_guacamole -d -p 8081:8080 -v /opt/guacamole/key:/config/guacamole/key -e JUMPSERVER_KEY_DIR=/config/guacamole/key -e JUMPSERVER_SERVER=http://127.0.0.1:8080 registry.jumpserver.org/public/guacamole:latest
Unable to find image 'registry.jumpserver.org/public/guacamole:latest' locally
latest: Pulling from public/guacamole
723254a2c089: Downloading  458.2kB/45.12MB
abe15a44e12f: Downloading  458.2kB/11.11MB
409a28e3cc3d: Downloading  392.7kB/4.335MB
a9511c68044a: Waiting 
9d1b16e30bc8: Waiting 
0fc5a09c9242: Waiting 
d34976006493: Waiting 
3b70003f0c10: Waiting 
bc7887582e2e: Waiting 
d2ab4f165865: Waiting 
3882b23577d6: Waiting 
9f8b758ebfa6: Waiting 
ef5d2d838878: Waiting 
310fa32446d6: Waiting 
a23204f32cd2: Waiting 
f3cba08c8ef8: Waiting 
59073672f2e3: Waiting 
86d50039bf5c: Waiting 
7041bb4312f0: Waiting 
4a7a284e984f: Waiting 
2da6caf16c59: Waiting 
c41fb67653ac: Waiting 
1a457b98f2f8: Waiting 
...
...
...
~~~


这个过程有些慢长


完成后，遵循 coco 一样的方法，在 jumpserver 中接受注册



---

# 总结

总体来将 jumpserver 涉及的组件有点多

这些组件拼凑起来一起协同完成管理任务

所以要整体是否运行正常要考虑以下几个服务

* nginx
* mysql
* jumpserver
* coco
* guacamole


这几个组件的用途这里作一个简要的说明　

* Jumpserver：为管理后台，管理员可以通过Web页面进行资产管理、用户管理、资产授权等操作
* Coco: 为 SSH Server 和 Web Terminal Server 用户可以通过使用自己的账户登录 SSH 或者 Web Terminal 直接访问被授权的资产, 不需要知道服务器的账户密码
* Luna: 为 Web Terminal Server 前端页面，用户使用 Web Terminal 方式登录所需要的组件
* Guacamole: 为 Windows 组件，用户可以通过 Web Terminal 来连接 Windows 资产 ，暂时只能通过 Web Terminal 来访问


端口说明

* Jumpserver 默认端口为 8080/tcp 配置文件在 jumpserver/config.py
* Coco 默认 SSH 端口为 2222/tcp ，默认 Web Terminal 端口为 5000/tcp 配置文件在 coco/conf.py
* Guacamole 默认端口为 8081/tcp 在 docker run 时指定
* Nginx 默认端口为 80/tcp 配置在 nginx/nginx.conf 中指定





* TOC
{:toc}

---

[jumpserver]:http://www.jumpserver.org/
[jumpserver_doc]:http://docs.jumpserver.org/zh/docs/step_by_step.html