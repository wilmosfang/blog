---
layout: post
title: "Install Jekyll"
author:  wilmosfang
date: 2018-07-05 12:59:42
image: '/assets/img/'
excerpt: '快速构建一个 Jekyll 博客'
main-class: jekyll
color: '#B31917'
tags:
 - jekyll
 - gcc
 - ruby
 - rvm
 - nodejs
 - gem
 - gulp
categories:
 - jekyll
twitter_text: 'Simple process of Jekyll installation'
introduction: 'Installation of Jekyll'
---

# 前言

**[Jekyll][jekyll]** 可以将 markdown 格式的文本快速转换成博客

>Transform your plain text into static websites and blogs

这里演示一下如何用十分钟搭建一个 **[Jekyll][jekyll]** 博客

> **Tip:** 当前最新版本 **Jekyll 3.8.3**

---

# 操作

## 环境

~~~
[root@h105 ~]# hostnamectl 
   Static hostname: h105
         Icon name: computer-vm
           Chassis: vm
        Machine ID: d027b43535bc4dbe83ef5a6f935dffdb
           Boot ID: b8c9c157cc6e423b933e2a6a641c0910
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[root@h105 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85841sec preferred_lft 85841sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:4c:ad:6a brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.105/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4c:ad6a/64 scope link 
       valid_lft forever preferred_lft forever
[root@h105 ~]# 
~~~

## 安装 gcc

~~~
[root@h105 ~]# yum install gcc
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
base                                                     | 3.6 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/4): extras/7/x86_64/primary_db                          | 150 kB   00:00     
(2/4): base/7/x86_64/group_gz                              | 166 kB   00:00     
(3/4): base/7/x86_64/primary_db                            | 5.9 MB   00:02     
(4/4): updates/7/x86_64/primary_db                         | 3.6 MB   00:03     
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.8.5-28.el7_5.1 will be installed
--> Processing Dependency: libgomp = 4.8.5-28.el7_5.1 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: cpp = 4.8.5-28.el7_5.1 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: libgcc >= 4.8.5-28.el7_5.1 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: glibc-devel >= 2.2.90-12 for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: libmpfr.so.4()(64bit) for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-4.8.5-28.el7_5.1.x86_64
--> Running transaction check
---> Package cpp.x86_64 0:4.8.5-28.el7_5.1 will be installed
---> Package glibc-devel.x86_64 0:2.17-222.el7 will be installed
--> Processing Dependency: glibc-headers = 2.17-222.el7 for package: glibc-devel-2.17-222.el7.x86_64
--> Processing Dependency: glibc-headers for package: glibc-devel-2.17-222.el7.x86_64
---> Package libgcc.x86_64 0:4.8.5-28.el7 will be updated
---> Package libgcc.x86_64 0:4.8.5-28.el7_5.1 will be an update
---> Package libgomp.x86_64 0:4.8.5-28.el7 will be updated
---> Package libgomp.x86_64 0:4.8.5-28.el7_5.1 will be an update
---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
---> Package mpfr.x86_64 0:3.1.1-4.el7 will be installed
--> Running transaction check
---> Package glibc-headers.x86_64 0:2.17-222.el7 will be installed
--> Processing Dependency: kernel-headers >= 2.2.1 for package: glibc-headers-2.17-222.el7.x86_64
--> Processing Dependency: kernel-headers for package: glibc-headers-2.17-222.el7.x86_64
--> Running transaction check
---> Package kernel-headers.x86_64 0:3.10.0-862.6.3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package              Arch         Version                  Repository     Size
================================================================================
Installing:
 gcc                  x86_64       4.8.5-28.el7_5.1         updates        16 M
Installing for dependencies:
 cpp                  x86_64       4.8.5-28.el7_5.1         updates       5.9 M
 glibc-devel          x86_64       2.17-222.el7             base          1.1 M
 glibc-headers        x86_64       2.17-222.el7             base          678 k
 kernel-headers       x86_64       3.10.0-862.6.3.el7       updates       7.1 M
 libmpc               x86_64       1.0.1-3.el7              base           51 k
 mpfr                 x86_64       3.1.1-4.el7              base          203 k
Updating for dependencies:
 libgcc               x86_64       4.8.5-28.el7_5.1         updates       101 k
 libgomp              x86_64       4.8.5-28.el7_5.1         updates       156 k

Transaction Summary
================================================================================
Install  1 Package  (+6 Dependent packages)
Upgrade             ( 2 Dependent packages)

Total download size: 31 M
Is this ok [y/d/N]: y
Downloading packages:
Not downloading deltainfo for updates, MD is 370 k and rpms are 257 k
warning: /var/cache/yum/x86_64/7/base/packages/glibc-headers-2.17-222.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for glibc-headers-2.17-222.el7.x86_64.rpm is not installed
(1/9): glibc-headers-2.17-222.el7.x86_64.rpm               | 678 kB   00:01     
(2/9): glibc-devel-2.17-222.el7.x86_64.rpm                 | 1.1 MB   00:01     
Public key for libgcc-4.8.5-28.el7_5.1.x86_64.rpm is not installed
(3/9): libgcc-4.8.5-28.el7_5.1.x86_64.rpm                  | 101 kB   00:00     
(4/9): libgomp-4.8.5-28.el7_5.1.x86_64.rpm                 | 156 kB   00:00     
(5/9): libmpc-1.0.1-3.el7.x86_64.rpm                       |  51 kB   00:00     
(6/9): mpfr-3.1.1-4.el7.x86_64.rpm                         | 203 kB   00:00     
(7/9): cpp-4.8.5-28.el7_5.1.x86_64.rpm                     | 5.9 MB   00:08     
(8/9): kernel-headers-3.10.0-862.6.3.el7.x86_64.rpm        | 7.1 MB   00:09     
(9/9): gcc-4.8.5-28.el7_5.1.x86_64.rpm                     |  16 MB   00:16     
--------------------------------------------------------------------------------
Total                                              1.9 MB/s |  31 MB  00:16     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-5.1804.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : mpfr-3.1.1-4.el7.x86_64                                     1/11 
  Installing : libmpc-1.0.1-3.el7.x86_64                                   2/11 
  Installing : cpp-4.8.5-28.el7_5.1.x86_64                                 3/11 
  Installing : kernel-headers-3.10.0-862.6.3.el7.x86_64                    4/11 
  Installing : glibc-headers-2.17-222.el7.x86_64                           5/11 
  Installing : glibc-devel-2.17-222.el7.x86_64                             6/11 
  Updating   : libgcc-4.8.5-28.el7_5.1.x86_64                              7/11 
  Updating   : libgomp-4.8.5-28.el7_5.1.x86_64                             8/11 
  Installing : gcc-4.8.5-28.el7_5.1.x86_64                                 9/11 
  Cleanup    : libgcc-4.8.5-28.el7.x86_64                                 10/11 
  Cleanup    : libgomp-4.8.5-28.el7.x86_64                                11/11 
  Verifying  : libgomp-4.8.5-28.el7_5.1.x86_64                             1/11 
  Verifying  : libgcc-4.8.5-28.el7_5.1.x86_64                              2/11 
  Verifying  : gcc-4.8.5-28.el7_5.1.x86_64                                 3/11 
  Verifying  : glibc-devel-2.17-222.el7.x86_64                             4/11 
  Verifying  : mpfr-3.1.1-4.el7.x86_64                                     5/11 
  Verifying  : cpp-4.8.5-28.el7_5.1.x86_64                                 6/11 
  Verifying  : glibc-headers-2.17-222.el7.x86_64                           7/11 
  Verifying  : libmpc-1.0.1-3.el7.x86_64                                   8/11 
  Verifying  : kernel-headers-3.10.0-862.6.3.el7.x86_64                    9/11 
  Verifying  : libgomp-4.8.5-28.el7.x86_64                                10/11 
  Verifying  : libgcc-4.8.5-28.el7.x86_64                                 11/11 

Installed:
  gcc.x86_64 0:4.8.5-28.el7_5.1                                                 

Dependency Installed:
  cpp.x86_64 0:4.8.5-28.el7_5.1                                                 
  glibc-devel.x86_64 0:2.17-222.el7                                             
  glibc-headers.x86_64 0:2.17-222.el7                                           
  kernel-headers.x86_64 0:3.10.0-862.6.3.el7                                    
  libmpc.x86_64 0:1.0.1-3.el7                                                   
  mpfr.x86_64 0:3.1.1-4.el7                                                     

Dependency Updated:
  libgcc.x86_64 0:4.8.5-28.el7_5.1       libgomp.x86_64 0:4.8.5-28.el7_5.1      

Complete!
[root@h105 ~]# 
~~~

## 安装 rvm

~~~
[root@h105 ~]# rvm -v 
-bash: rvm: command not found
[root@h105 ~]# curl -L get.rvm.io | bash -s stable
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    170      0  0:00:01  0:00:01 --:--:--   170
100 24361  100 24361    0     0  14485      0  0:00:01  0:00:01 --:--:-- 14485
Downloading https://github.com/rvm/rvm/archive/1.29.4.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.4/1.29.4.tar.gz.asc
gpg: directory `/root/.gnupg' created
gpg: new configuration file `/root/.gnupg/gpg.conf' created
gpg: WARNING: options in `/root/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/root/.gnupg/pubring.gpg' created
gpg: Signature made Sun 01 Jul 2018 07:41:26 PM UTC using RSA key ID BF04FF17
gpg: Can't check signature: No public key
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found. Assuming you trust Michal Papis import the mpapis public key (downloading the signatures).

GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.29.4.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.4/1.29.4.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

the key can be compared with:

    https://rvm.io/mpapis.asc
    https://keybase.io/mpapis

NOTE: GPG version 2.1.17 have a bug which cause failures during fetching keys from remote server. Please downgrade or upgrade to newer version (if available) or use the second method described above.

[root@h105 ~]# gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: requesting key D39DC0E3 from hkp server keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[root@h105 ~]# echo $?
0
[root@h105 ~]# curl -L get.rvm.io | bash -s stable
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    372      0 --:--:-- --:--:-- --:--:--   373
100 24361  100 24361    0     0  35805      0 --:--:-- --:--:-- --:--:-- 35805
Downloading https://github.com/rvm/rvm/archive/1.29.4.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.4/1.29.4.tar.gz.asc
gpg: Signature made Sun 01 Jul 2018 07:41:26 PM UTC using RSA key ID BF04FF17
gpg: Good signature from "Michal Papis (RVM signing) <mpapis@gmail.com>"
gpg:                 aka "Michal Papis <michal.papis@toptal.com>"
gpg:                 aka "[jpeg image of size 5015]"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
     Subkey fingerprint: 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/rvm/archives/rvm-1.29.4.tgz'
Creating group 'rvm'
Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
  * Please do NOT forget to add your users to the rvm group.
     The installer no longer auto-adds root or users to the rvm group. Admins must do this.
     Also, please note that group memberships are ONLY evaluated at login time.
     This means that users must log out then back in before group membership takes effect!
[root@h105 ~]# echo $?
0
[root@h105 ~]# su - root 
Last login: Wed Jul  4 23:54:35 UTC 2018 on pts/0
[root@h105 ~]# rmv -v 
-bash: rmv: command not found
[root@h105 ~]# rvm -v 
rvm 1.29.4 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
[root@h105 ~]# 
~~~

## 安装 ruby


~~~
[root@h105 ~]# rvm install ruby
Searching for binary rubies, this might take some time.
No binary rubies available for: centos/7/x86_64/ruby-2.5.1.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: patch, autoconf, automake, bison, gcc-c++, libffi-devel, libtool, patch, readline-devel, sqlite-devel, zlib-devel, openssl-devel.-
Requirements installation successful.
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-2.5.1, this may take a while depending on your cpu(s)...
ruby-2.5.1 - #downloading ruby-2.5.1, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13.3M  100 13.3M    0     0  2696k      0  0:00:05  0:00:05 --:--:-- 3048k
ruby-2.5.1 - #extracting ruby-2.5.1 to /usr/local/rvm/src/ruby-2.5.1.....
ruby-2.5.1 - #configuring......................................................|
ruby-2.5.1 - #post-configuration..
ruby-2.5.1 - #compiling........................................................-
ruby-2.5.1 - #installing..............................
ruby-2.5.1 - #making binaries executable..
ruby-2.5.1 - #downloading rubygems-2.7.7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  894k  100  894k    0     0   984k      0 --:--:-- --:--:-- --:--:--  983k
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.5.1 - #extracting rubygems-2.7.7........................................-
ruby-2.5.1 - #removing old rubygems........
ruby-2.5.1 - #installing rubygems-2.7.7................................
ruby-2.5.1 - #gemset created /usr/local/rvm/gems/ruby-2.5.1@global
ruby-2.5.1 - #importing gemset /usr/local/rvm/gemsets/global.gems..............|
ruby-2.5.1 - #generating global wrappers.......
ruby-2.5.1 - #gemset created /usr/local/rvm/gems/ruby-2.5.1
ruby-2.5.1 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.5.1 - #generating default wrappers.......
ruby-2.5.1 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.5.1 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
[root@h105 ~]# echo $?
0
[root@h105 ~]# 
~~~


## 安装 jekyll

~~~
[root@h105 ~]# gem install jekyll
Fetching: public_suffix-3.0.2.gem (100%)gem
Successfully installed public_suffix-3.0.2
Fetching: addressable-2.5.2.gem (100%)
Successfully installed addressable-2.5.2
Fetching: colorator-1.1.0.gem (100%)
Successfully installed colorator-1.1.0
Fetching: http_parser.rb-0.6.0.gem (100%)
Building native extensions. This could take a while...
Successfully installed http_parser.rb-0.6.0
Fetching: eventmachine-1.2.7.gem (100%)
Building native extensions. This could take a while...
Successfully installed eventmachine-1.2.7
Fetching: em-websocket-0.5.1.gem (100%)
Successfully installed em-websocket-0.5.1
Fetching: concurrent-ruby-1.0.5.gem (100%)
Successfully installed concurrent-ruby-1.0.5
Fetching: i18n-0.9.5.gem (100%)
Successfully installed i18n-0.9.5
Fetching: rb-fsevent-0.10.3.gem (100%)
Successfully installed rb-fsevent-0.10.3
Fetching: ffi-1.9.25.gem (100%)
Building native extensions. This could take a while...
Successfully installed ffi-1.9.25
Fetching: rb-inotify-0.9.10.gem (100%)
Successfully installed rb-inotify-0.9.10
Fetching: sass-listen-4.0.0.gem (100%)
Successfully installed sass-listen-4.0.0
Fetching: sass-3.5.6.gem (100%)
Successfully installed sass-3.5.6
Fetching: jekyll-sass-converter-1.5.2.gem (100%)
Successfully installed jekyll-sass-converter-1.5.2
Fetching: ruby_dep-1.5.0.gem (100%)
Successfully installed ruby_dep-1.5.0
Fetching: listen-3.1.5.gem (100%)
Successfully installed listen-3.1.5
Fetching: jekyll-watch-2.0.0.gem (100%)
Successfully installed jekyll-watch-2.0.0
Fetching: kramdown-1.17.0.gem (100%)
Successfully installed kramdown-1.17.0
Fetching: liquid-4.0.0.gem (100%)
Successfully installed liquid-4.0.0
Fetching: mercenary-0.3.6.gem (100%)
Successfully installed mercenary-0.3.6
Fetching: forwardable-extended-2.6.0.gem (100%)
Successfully installed forwardable-extended-2.6.0
Fetching: pathutil-0.16.1.gem (100%)
Successfully installed pathutil-0.16.1
Fetching: rouge-3.1.1.gem (100%)
Successfully installed rouge-3.1.1
Fetching: safe_yaml-1.0.4.gem (100%)
Successfully installed safe_yaml-1.0.4
Fetching: jekyll-3.8.3.gem (100%)
Successfully installed jekyll-3.8.3
Parsing documentation for public_suffix-3.0.2
Installing ri documentation for public_suffix-3.0.2
Parsing documentation for addressable-2.5.2
Installing ri documentation for addressable-2.5.2
Parsing documentation for colorator-1.1.0
Installing ri documentation for colorator-1.1.0
Parsing documentation for http_parser.rb-0.6.0
Installing ri documentation for http_parser.rb-0.6.0
Parsing documentation for eventmachine-1.2.7
Installing ri documentation for eventmachine-1.2.7
Parsing documentation for em-websocket-0.5.1
Installing ri documentation for em-websocket-0.5.1
Parsing documentation for concurrent-ruby-1.0.5
Installing ri documentation for concurrent-ruby-1.0.5
Parsing documentation for i18n-0.9.5
Installing ri documentation for i18n-0.9.5
Parsing documentation for rb-fsevent-0.10.3
Installing ri documentation for rb-fsevent-0.10.3
Parsing documentation for ffi-1.9.25
Installing ri documentation for ffi-1.9.25
Parsing documentation for rb-inotify-0.9.10
Installing ri documentation for rb-inotify-0.9.10
Parsing documentation for sass-listen-4.0.0
Installing ri documentation for sass-listen-4.0.0
Parsing documentation for sass-3.5.6
Installing ri documentation for sass-3.5.6
Parsing documentation for jekyll-sass-converter-1.5.2
Installing ri documentation for jekyll-sass-converter-1.5.2
Parsing documentation for ruby_dep-1.5.0
Installing ri documentation for ruby_dep-1.5.0
Parsing documentation for listen-3.1.5
Installing ri documentation for listen-3.1.5
Parsing documentation for jekyll-watch-2.0.0
Installing ri documentation for jekyll-watch-2.0.0
Parsing documentation for kramdown-1.17.0
Installing ri documentation for kramdown-1.17.0
Parsing documentation for liquid-4.0.0
Installing ri documentation for liquid-4.0.0
Parsing documentation for mercenary-0.3.6
Installing ri documentation for mercenary-0.3.6
Parsing documentation for forwardable-extended-2.6.0
Installing ri documentation for forwardable-extended-2.6.0
Parsing documentation for pathutil-0.16.1
Installing ri documentation for pathutil-0.16.1
Parsing documentation for rouge-3.1.1
Installing ri documentation for rouge-3.1.1
Parsing documentation for safe_yaml-1.0.4
Installing ri documentation for safe_yaml-1.0.4
Parsing documentation for jekyll-3.8.3
Installing ri documentation for jekyll-3.8.3
Done installing documentation for public_suffix, addressable, colorator, http_parser.rb, eventmachine, em-websocket, concurrent-ruby, i18n, rb-fsevent, ffi, rb-inotify, sass-listen, sass, jekyll-sass-converter, ruby_dep, listen, jekyll-watch, kramdown, liquid, mercenary, forwardable-extended, pathutil, rouge, safe_yaml, jekyll after 29 seconds
25 gems installed
[root@h105 ~]# echo $?
0
[root@h105 ~]# 
~~~

查看帮助文档

~~~
[root@h105 ~]# jekyll  --help 
jekyll 3.8.3 -- Jekyll is a blog-aware, static site generator in Ruby

Usage:

  jekyll <subcommand> [options]

Options:
        -s, --source [DIR]  Source directory (defaults to ./)
        -d, --destination [DIR]  Destination directory (defaults to ./_site)
            --safe         Safe mode (defaults to false)
        -p, --plugins PLUGINS_DIR1[,PLUGINS_DIR2[,...]]  Plugins directory (defaults to ./_plugins)
            --layouts DIR  Layouts directory (defaults to ./_layouts)
            --profile      Generate a Liquid rendering profile
        -h, --help         Show this message
        -v, --version      Print the name and version
        -t, --trace        Show the full backtrace when an error occurs

Subcommands:
  docs                  
  import                
  build, b              Build your site
  clean                 Clean the site (removes site output and metadata file) without building.
  doctor, hyde          Search site and print specific deprecation warnings
  help                  Show the help message, optionally for a given subcommand.
  new                   Creates a new Jekyll site scaffold in PATH
  new-theme             Creates a new Jekyll theme scaffold
  serve, server, s      Serve your site locally
[root@h105 ~]# 
~~~

## 安装 nodejs

~~~
[root@h105 ~]# yum install epel-release -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch             Version         Repository        Size
================================================================================
Installing:
 epel-release           noarch           7-11            extras            15 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Downloading packages:
epel-release-7-11.noarch.rpm                               |  15 kB   00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-11.noarch                                     1/1 
  Verifying  : epel-release-7-11.noarch                                     1/1 

Installed:
  epel-release.noarch 0:7-11                                                    

Complete!
[root@h105 ~]# yum -y install nodejs
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                     | 5.1 kB     00:00     
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
epel                                                     | 3.2 kB     00:00     
(1/3): epel/x86_64/group_gz                                |  88 kB   00:00     
(2/3): epel/x86_64/updateinfo                              | 929 kB   00:00     
(3/3): epel/x86_64/primary                                 | 3.5 MB   00:04     
epel                                                                12604/12604
Resolving Dependencies
--> Running transaction check
---> Package nodejs.x86_64 1:6.14.3-1.el7 will be installed
--> Processing Dependency: npm = 1:3.10.10-1.6.14.3.1.el7 for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: libuv >= 1:1.9.1 for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: http-parser >= 2.7.0 for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: libuv.so.1()(64bit) for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: libicuuc.so.50()(64bit) for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: libicui18n.so.50()(64bit) for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: libicudata.so.50()(64bit) for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Processing Dependency: libhttp_parser.so.2()(64bit) for package: 1:nodejs-6.14.3-1.el7.x86_64
--> Running transaction check
---> Package http-parser.x86_64 0:2.7.1-5.el7_4 will be installed
---> Package libicu.x86_64 0:50.1.2-15.el7 will be installed
---> Package libuv.x86_64 1:1.19.2-1.el7 will be installed
---> Package npm.x86_64 1:3.10.10-1.6.14.3.1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch         Version                        Repository  Size
================================================================================
Installing:
 nodejs            x86_64       1:6.14.3-1.el7                 epel       4.7 M
Installing for dependencies:
 http-parser       x86_64       2.7.1-5.el7_4                  base        28 k
 libicu            x86_64       50.1.2-15.el7                  base       6.9 M
 libuv             x86_64       1:1.19.2-1.el7                 epel       121 k
 npm               x86_64       1:3.10.10-1.6.14.3.1.el7       epel       2.5 M

Transaction Summary
================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 14 M
Installed size: 51 M
Downloading packages:
(1/5): http-parser-2.7.1-5.el7_4.x86_64.rpm                |  28 kB   00:00     
warning: /var/cache/yum/x86_64/7/epel/packages/libuv-1.19.2-1.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for libuv-1.19.2-1.el7.x86_64.rpm is not installed
(2/5): libuv-1.19.2-1.el7.x86_64.rpm                       | 121 kB   00:00     
(3/5): npm-3.10.10-1.6.14.3.1.el7.x86_64.rpm               | 2.5 MB   00:02     
(4/5): nodejs-6.14.3-1.el7.x86_64.rpm                      | 4.7 MB   00:03     
(5/5): libicu-50.1.2-15.el7.x86_64.rpm                     | 6.9 MB   00:06     
--------------------------------------------------------------------------------
Total                                              2.1 MB/s |  14 MB  00:06     
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
  Installing : libicu-50.1.2-15.el7.x86_64                                  1/5 
  Installing : http-parser-2.7.1-5.el7_4.x86_64                             2/5 
  Installing : 1:libuv-1.19.2-1.el7.x86_64                                  3/5 
  Installing : 1:npm-3.10.10-1.6.14.3.1.el7.x86_64                          4/5 
  Installing : 1:nodejs-6.14.3-1.el7.x86_64                                 5/5 
  Verifying  : 1:libuv-1.19.2-1.el7.x86_64                                  1/5 
  Verifying  : http-parser-2.7.1-5.el7_4.x86_64                             2/5 
  Verifying  : 1:nodejs-6.14.3-1.el7.x86_64                                 3/5 
  Verifying  : 1:npm-3.10.10-1.6.14.3.1.el7.x86_64                          4/5 
  Verifying  : libicu-50.1.2-15.el7.x86_64                                  5/5 

Installed:
  nodejs.x86_64 1:6.14.3-1.el7                                                  

Dependency Installed:
  http-parser.x86_64 0:2.7.1-5.el7_4     libicu.x86_64 0:50.1.2-15.el7          
  libuv.x86_64 1:1.19.2-1.el7            npm.x86_64 1:3.10.10-1.6.14.3.1.el7    

Complete!
[root@h105 ~]# 
~~~


## 安装 jekyll-paginate

这是一个分页插件

~~~
[root@h105 ~]# gem install  jekyll-paginate
Fetching: jekyll-paginate-1.1.0.gem (100%)
Successfully installed jekyll-paginate-1.1.0
Parsing documentation for jekyll-paginate-1.1.0
Installing ri documentation for jekyll-paginate-1.1.0
Done installing documentation for jekyll-paginate after 0 seconds
1 gem installed
[root@h105 ~]# 
~~~

## 安装 git

~~~
[root@h105 ~]# yum install git -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package git.x86_64 0:1.8.3.1-14.el7_5 will be installed
--> Processing Dependency: perl-Git = 1.8.3.1-14.el7_5 for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(Term::ReadKey) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(Git) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: perl(Error) for package: git-1.8.3.1-14.el7_5.x86_64
--> Processing Dependency: libgnome-keyring.so.0()(64bit) for package: git-1.8.3.1-14.el7_5.x86_64
--> Running transaction check
---> Package libgnome-keyring.x86_64 0:3.12.0-1.el7 will be installed
---> Package perl-Error.noarch 1:0.17020-2.el7 will be installed
---> Package perl-Git.noarch 0:1.8.3.1-14.el7_5 will be installed
---> Package perl-TermReadKey.x86_64 0:2.30-20.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch         Version                Repository     Size
================================================================================
Installing:
 git                    x86_64       1.8.3.1-14.el7_5       updates       4.4 M
Installing for dependencies:
 libgnome-keyring       x86_64       3.12.0-1.el7           base          109 k
 perl-Error             noarch       1:0.17020-2.el7        base           32 k
 perl-Git               noarch       1.8.3.1-14.el7_5       updates        54 k
 perl-TermReadKey       x86_64       2.30-20.el7            base           31 k

Transaction Summary
================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 4.6 M
Installed size: 23 M
Downloading packages:
(1/5): perl-Error-0.17020-2.el7.noarch.rpm                 |  32 kB   00:00     
(2/5): perl-Git-1.8.3.1-14.el7_5.noarch.rpm                |  54 kB   00:00     
(3/5): perl-TermReadKey-2.30-20.el7.x86_64.rpm             |  31 kB   00:00     
(4/5): libgnome-keyring-3.12.0-1.el7.x86_64.rpm            | 109 kB   00:00     
(5/5): git-1.8.3.1-14.el7_5.x86_64.rpm                     | 4.4 MB   00:03     
--------------------------------------------------------------------------------
Total                                              1.2 MB/s | 4.6 MB  00:03     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:perl-Error-0.17020-2.el7.noarch                            1/5 
  Installing : perl-TermReadKey-2.30-20.el7.x86_64                          2/5 
  Installing : libgnome-keyring-3.12.0-1.el7.x86_64                         3/5 
  Installing : git-1.8.3.1-14.el7_5.x86_64                                  4/5 
  Installing : perl-Git-1.8.3.1-14.el7_5.noarch                             5/5 
  Verifying  : 1:perl-Error-0.17020-2.el7.noarch                            1/5 
  Verifying  : git-1.8.3.1-14.el7_5.x86_64                                  2/5 
  Verifying  : libgnome-keyring-3.12.0-1.el7.x86_64                         3/5 
  Verifying  : perl-Git-1.8.3.1-14.el7_5.noarch                             4/5 
  Verifying  : perl-TermReadKey-2.30-20.el7.x86_64                          5/5 

Installed:
  git.x86_64 0:1.8.3.1-14.el7_5                                                 

Dependency Installed:
  libgnome-keyring.x86_64 0:3.12.0-1.el7  perl-Error.noarch 1:0.17020-2.el7     
  perl-Git.noarch 0:1.8.3.1-14.el7_5      perl-TermReadKey.x86_64 0:2.30-20.el7 

Complete!
[root@h105 ~]# 
~~~

## clone 一个博客项目

~~~
[vagrant@h105 ~]$ cd /vagrant/
[vagrant@h105 vagrant]$ ls
Vagrantfile
[vagrant@h105 vagrant]$ 
[vagrant@h105 vagrant]$ mkdir git
[vagrant@h105 vagrant]$ cd git/
[vagrant@h105 git]$ pwd
/vagrant/git
[vagrant@h105 git]$ ls
[vagrant@h105 git]$ git clone https://github.com/wilmosfang/biscuits.git
Cloning into 'biscuits'...
remote: Counting objects: 1142, done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 1142 (delta 9), reused 12 (delta 2), pack-reused 1118
Receiving objects: 100% (1142/1142), 33.50 MiB | 2.25 MiB/s, done.
Resolving deltas: 100% (488/488), done.
Checking out files: 100% (457/457), done.
[vagrant@h105 git]$ echo $?
0
[vagrant@h105 git]$ du -sh biscuits/
72M	biscuits/
[vagrant@h105 git]$ cd biscuits/
[vagrant@h105 biscuits]$ ls
CNAME        _layouts    category       gulpfile.js   robots.txt   src
README.md    _posts      category.html  index.html    search.json  tags.html
_config.yml  about.html  favicon.ico    initpost.sh   series.html
_includes    assets      feed.xml       package.json  sitemap.xml
[vagrant@h105 biscuits]$ git pull 
Already up-to-date.
[vagrant@h105 biscuits]$
~~~

## 配置 alias

简化启动操作

~~~
[root@h105 ~]# vi ~/.bashrc 
[root@h105 ~]# source ~/.bashrc
[root@h105 ~]# alias runblog
alias runblog='jekyll server  --host 0.0.0.0 --port 80'
[root@h105 ~]# 
~~~

## 运行博客

~~~
[root@h105 biscuits]# runblog
Configuration file: /vagrant/git/biscuits/_config.yml
            Source: /vagrant/git/biscuits
       Destination: /vagrant/git/biscuits/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 23.275 seconds.
 Auto-regeneration: enabled for '/vagrant/git/biscuits'
    Server address: http://0.0.0.0:80/
  Server running... press ctrl-c to stop.

...
...
...
~~~


# 附

## 安装 gulp

这是一个本地的基于流的自动化构建工具

在前端领域有着广泛的使用

~~~
[root@h105 ~]# npm install --global gulp
npm WARN deprecated gulp-util@3.0.8: gulp-util is deprecated - replace it, following the guidelines at https://medium.com/gulpjs/gulp-util-ca3b1f9f9ac5
npm WARN deprecated graceful-fs@3.0.11: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
npm WARN deprecated minimatch@2.0.10: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated minimatch@0.2.14: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated graceful-fs@1.2.3: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
/usr/bin/gulp -> /usr/lib/node_modules/gulp/bin/gulp.js
/usr/lib
└─┬ gulp@3.9.1 
  ├── archy@1.0.0 
  ├─┬ chalk@1.1.3 
  │ ├── ansi-styles@2.2.1 
  │ ├── escape-string-regexp@1.0.5 
  │ ├─┬ has-ansi@2.0.0 
  │ │ └── ansi-regex@2.1.1 
  │ ├── strip-ansi@3.0.1 
  │ └── supports-color@2.0.0 
  ├── deprecated@0.0.1 
  ├─┬ gulp-util@3.0.8 
  │ ├── array-differ@1.0.0 
  │ ├── array-uniq@1.0.3 
  │ ├── beeper@1.1.1 
  │ ├── dateformat@2.2.0 
  │ ├─┬ fancy-log@1.3.2 
  │ │ ├─┬ ansi-gray@0.1.1 
  │ │ │ └── ansi-wrap@0.1.0 
  │ │ ├── color-support@1.1.3 
  │ │ └── time-stamp@1.1.0 
  │ ├─┬ gulplog@1.0.0 
  │ │ └── glogg@1.0.1 
  │ ├─┬ has-gulplog@0.1.0 
  │ │ └── sparkles@1.0.1 
  │ ├── lodash._reescape@3.0.0 
  │ ├── lodash._reevaluate@3.0.0 
  │ ├── lodash._reinterpolate@3.0.0 
  │ ├─┬ lodash.template@3.6.2 
  │ │ ├── lodash._basecopy@3.0.1 
  │ │ ├── lodash._basetostring@3.0.1 
  │ │ ├── lodash._basevalues@3.0.0 
  │ │ ├── lodash._isiterateecall@3.0.9 
  │ │ ├─┬ lodash.escape@3.2.0 
  │ │ │ └── lodash._root@3.0.1 
  │ │ ├─┬ lodash.keys@3.1.2 
  │ │ │ ├── lodash._getnative@3.9.1 
  │ │ │ ├── lodash.isarguments@3.1.0 
  │ │ │ └── lodash.isarray@3.0.4 
  │ │ ├── lodash.restparam@3.6.1 
  │ │ └── lodash.templatesettings@3.1.1 
  │ ├─┬ multipipe@0.1.2 
  │ │ └─┬ duplexer2@0.0.2 
  │ │   └── readable-stream@1.1.14 
  │ ├── object-assign@3.0.0 
  │ ├── replace-ext@0.0.1 
  │ ├─┬ through2@2.0.3 
  │ │ ├─┬ readable-stream@2.3.6 
  │ │ │ ├── core-util-is@1.0.2 
  │ │ │ ├── inherits@2.0.3 
  │ │ │ ├── isarray@1.0.0 
  │ │ │ ├── process-nextick-args@2.0.0 
  │ │ │ ├── safe-buffer@5.1.2 
  │ │ │ ├── string_decoder@1.1.1 
  │ │ │ └── util-deprecate@1.0.2 
  │ │ └── xtend@4.0.1 
  │ └─┬ vinyl@0.5.3 
  │   ├── clone@1.0.4 
  │   └── clone-stats@0.0.1 
  ├── interpret@1.1.0 
  ├─┬ liftoff@2.5.0 
  │ ├── extend@3.0.1 
  │ ├─┬ findup-sync@2.0.0 
  │ │ ├── detect-file@1.0.0 
  │ │ ├─┬ is-glob@3.1.0 
  │ │ │ └── is-extglob@2.1.1 
  │ │ ├─┬ micromatch@3.1.10 
  │ │ │ ├── arr-diff@4.0.0 
  │ │ │ ├── array-unique@0.3.2 
  │ │ │ ├─┬ braces@2.3.2 
  │ │ │ │ ├── arr-flatten@1.1.0 
  │ │ │ │ ├─┬ extend-shallow@2.0.1 
  │ │ │ │ │ └── is-extendable@0.1.1 
  │ │ │ │ ├─┬ fill-range@4.0.0 
  │ │ │ │ │ ├── extend-shallow@2.0.1 
  │ │ │ │ │ ├─┬ is-number@3.0.0 
  │ │ │ │ │ │ └─┬ kind-of@3.2.2 
  │ │ │ │ │ │   └── is-buffer@1.1.6 
  │ │ │ │ │ ├── repeat-string@1.6.1 
  │ │ │ │ │ └── to-regex-range@2.1.1 
  │ │ │ │ ├── repeat-element@1.1.2 
  │ │ │ │ ├─┬ snapdragon-node@2.1.1 
  │ │ │ │ │ ├─┬ define-property@1.0.0 
  │ │ │ │ │ │ └─┬ is-descriptor@1.0.2 
  │ │ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
  │ │ │ │ │ │   └── is-data-descriptor@1.0.0 
  │ │ │ │ │ └─┬ snapdragon-util@3.0.1 
  │ │ │ │ │   └── kind-of@3.2.2 
  │ │ │ │ └── split-string@3.1.0 
  │ │ │ ├─┬ define-property@2.0.2 
  │ │ │ │ └─┬ is-descriptor@1.0.2 
  │ │ │ │   ├── is-accessor-descriptor@1.0.0 
  │ │ │ │   └── is-data-descriptor@1.0.0 
  │ │ │ ├─┬ extend-shallow@3.0.2 
  │ │ │ │ ├── assign-symbols@1.0.0 
  │ │ │ │ └── is-extendable@1.0.1 
  │ │ │ ├─┬ extglob@2.0.4 
  │ │ │ │ ├─┬ define-property@1.0.0 
  │ │ │ │ │ └─┬ is-descriptor@1.0.2 
  │ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
  │ │ │ │ │   └── is-data-descriptor@1.0.0 
  │ │ │ │ ├─┬ expand-brackets@2.1.4 
  │ │ │ │ │ ├── define-property@0.2.5 
  │ │ │ │ │ ├── extend-shallow@2.0.1 
  │ │ │ │ │ └── posix-character-classes@0.1.1 
  │ │ │ │ └── extend-shallow@2.0.1 
  │ │ │ ├── fragment-cache@0.2.1 
  │ │ │ ├── kind-of@6.0.2 
  │ │ │ ├─┬ nanomatch@1.2.13 
  │ │ │ │ └── is-windows@1.0.2 
  │ │ │ ├─┬ regex-not@1.0.2 
  │ │ │ │ └─┬ safe-regex@1.1.0 
  │ │ │ │   └── ret@0.1.15 
  │ │ │ ├─┬ snapdragon@0.8.2 
  │ │ │ │ ├─┬ base@0.11.2 
  │ │ │ │ │ ├─┬ cache-base@1.0.1 
  │ │ │ │ │ │ ├─┬ collection-visit@1.0.0 
  │ │ │ │ │ │ │ ├── map-visit@1.0.0 
  │ │ │ │ │ │ │ └── object-visit@1.0.1 
  │ │ │ │ │ │ ├── get-value@2.0.6 
  │ │ │ │ │ │ ├─┬ has-value@1.0.0 
  │ │ │ │ │ │ │ └─┬ has-values@1.0.0 
  │ │ │ │ │ │ │   └── kind-of@4.0.0 
  │ │ │ │ │ │ ├─┬ set-value@2.0.0 
  │ │ │ │ │ │ │ └── extend-shallow@2.0.1 
  │ │ │ │ │ │ ├─┬ to-object-path@0.3.0 
  │ │ │ │ │ │ │ └── kind-of@3.2.2 
  │ │ │ │ │ │ ├─┬ union-value@1.0.0 
  │ │ │ │ │ │ │ └─┬ set-value@0.4.3 
  │ │ │ │ │ │ │   └── extend-shallow@2.0.1 
  │ │ │ │ │ │ └─┬ unset-value@1.0.0 
  │ │ │ │ │ │   └─┬ has-value@0.3.1 
  │ │ │ │ │ │     ├── has-values@0.1.4 
  │ │ │ │ │ │     └─┬ isobject@2.1.0 
  │ │ │ │ │ │       └── isarray@1.0.0 
  │ │ │ │ │ ├─┬ class-utils@0.3.6 
  │ │ │ │ │ │ ├── arr-union@3.1.0 
  │ │ │ │ │ │ ├── define-property@0.2.5 
  │ │ │ │ │ │ └─┬ static-extend@0.1.2 
  │ │ │ │ │ │   ├── define-property@0.2.5 
  │ │ │ │ │ │   └─┬ object-copy@0.1.0 
  │ │ │ │ │ │     ├── copy-descriptor@0.1.1 
  │ │ │ │ │ │     ├── define-property@0.2.5 
  │ │ │ │ │ │     └── kind-of@3.2.2 
  │ │ │ │ │ ├── component-emitter@1.2.1 
  │ │ │ │ │ ├─┬ define-property@1.0.0 
  │ │ │ │ │ │ └─┬ is-descriptor@1.0.2 
  │ │ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
  │ │ │ │ │ │   └── is-data-descriptor@1.0.0 
  │ │ │ │ │ ├─┬ mixin-deep@1.3.1 
  │ │ │ │ │ │ └── is-extendable@1.0.1 
  │ │ │ │ │ └── pascalcase@0.1.1 
  │ │ │ │ ├─┬ debug@2.6.9 
  │ │ │ │ │ └── ms@2.0.0 
  │ │ │ │ ├─┬ define-property@0.2.5 
  │ │ │ │ │ └─┬ is-descriptor@0.1.6 
  │ │ │ │ │   ├─┬ is-accessor-descriptor@0.1.6 
  │ │ │ │ │   │ └── kind-of@3.2.2 
  │ │ │ │ │   ├─┬ is-data-descriptor@0.1.4 
  │ │ │ │ │   │ └── kind-of@3.2.2 
  │ │ │ │ │   └── kind-of@5.1.0 
  │ │ │ │ ├── extend-shallow@2.0.1 
  │ │ │ │ ├── source-map@0.5.7 
  │ │ │ │ ├─┬ source-map-resolve@0.5.2 
  │ │ │ │ │ ├── atob@2.1.1 
  │ │ │ │ │ ├── decode-uri-component@0.2.0 
  │ │ │ │ │ ├── resolve-url@0.2.1 
  │ │ │ │ │ ├── source-map-url@0.4.0 
  │ │ │ │ │ └── urix@0.1.0 
  │ │ │ │ └── use@3.1.0 
  │ │ │ └── to-regex@3.0.2 
  │ │ └─┬ resolve-dir@1.0.1 
  │ │   └─┬ global-modules@1.0.0 
  │ │     └─┬ global-prefix@1.0.2 
  │ │       ├── ini@1.3.5 
  │ │       └─┬ which@1.3.1 
  │ │         └── isexe@2.0.0 
  │ ├─┬ fined@1.1.0 
  │ │ ├─┬ expand-tilde@2.0.2 
  │ │ │ └─┬ homedir-polyfill@1.0.1 
  │ │ │   └── parse-passwd@1.0.0 
  │ │ ├─┬ object.defaults@1.1.0 
  │ │ │ ├── array-each@1.0.1 
  │ │ │ └── array-slice@1.1.0 
  │ │ ├── object.pick@1.3.0 
  │ │ └─┬ parse-filepath@1.0.2 
  │ │   ├─┬ is-absolute@1.0.0 
  │ │   │ └─┬ is-relative@1.0.0 
  │ │   │   └─┬ is-unc-path@1.0.0 
  │ │   │     └── unc-path-regex@0.1.2 
  │ │   ├── map-cache@0.2.2 
  │ │   └─┬ path-root@0.1.1 
  │ │     └── path-root-regex@0.1.2 
  │ ├── flagged-respawn@1.0.0 
  │ ├─┬ is-plain-object@2.0.4 
  │ │ └── isobject@3.0.1 
  │ ├─┬ object.map@1.0.1 
  │ │ ├─┬ for-own@1.0.0 
  │ │ │ └── for-in@1.0.2 
  │ │ └── make-iterator@1.0.1 
  │ ├── rechoir@0.6.2 
  │ └─┬ resolve@1.8.1 
  │   └── path-parse@1.0.5 
  ├── minimist@1.2.0 
  ├─┬ orchestrator@0.3.8 
  │ ├─┬ end-of-stream@0.1.5 
  │ │ └─┬ once@1.3.3 
  │ │   └── wrappy@1.0.2 
  │ ├── sequencify@0.0.7 
  │ └── stream-consume@0.1.1 
  ├── pretty-hrtime@1.0.3 
  ├── semver@4.3.6 
  ├─┬ tildify@1.2.0 
  │ └── os-homedir@1.0.2 
  ├─┬ v8flags@2.1.1 
  │ └── user-home@1.1.1 
  └─┬ vinyl-fs@0.3.14 
    ├── defaults@1.0.3 
    ├─┬ glob-stream@3.1.18 
    │ ├─┬ glob@4.5.3 
    │ │ └── inflight@1.0.6 
    │ ├─┬ glob2base@0.0.12 
    │ │ └── find-index@0.1.1 
    │ ├─┬ minimatch@2.0.10 
    │ │ └─┬ brace-expansion@1.1.11 
    │ │   ├── balanced-match@1.0.0 
    │ │   └── concat-map@0.0.1 
    │ ├── ordered-read-streams@0.1.0 
    │ ├─┬ through2@0.6.5 
    │ │ └── readable-stream@1.0.34 
    │ └── unique-stream@1.0.0 
    ├─┬ glob-watcher@0.0.6 
    │ └─┬ gaze@0.5.2 
    │   └─┬ globule@0.1.0 
    │     ├─┬ glob@3.1.21 
    │     │ ├── graceful-fs@1.2.3 
    │     │ └── inherits@1.0.2 
    │     ├── lodash@1.0.2 
    │     └─┬ minimatch@0.2.14 
    │       ├── lru-cache@2.7.3 
    │       └── sigmund@1.0.1 
    ├─┬ graceful-fs@3.0.11 
    │ └── natives@1.1.4 
    ├─┬ mkdirp@0.5.1 
    │ └── minimist@0.0.8 
    ├─┬ strip-bom@1.0.0 
    │ ├── first-chunk-stream@1.0.0 
    │ └── is-utf8@0.2.1 
    ├─┬ through2@0.6.5 
    │ └─┬ readable-stream@1.0.34 
    │   ├── isarray@0.0.1 
    │   └── string_decoder@0.10.31 
    └─┬ vinyl@0.4.6 
      └── clone@0.2.0 

[root@h105 ~]# 
~~~

上面的全局的安装

在本地要安装一次

~~~
[vagrant@h105 biscuits]$ gulp -f gulpfile.js 
[00:27:44] Local gulp not found in /vagrant/git/biscuits
[00:27:44] Try running: npm install gulp
[vagrant@h105 biscuits]$ npm install gulp
npm WARN deprecated gulp-util@3.0.8: gulp-util is deprecated - replace it, following the guidelines at https://medium.com/gulpjs/gulp-util-ca3b1f9f9ac5
npm WARN deprecated graceful-fs@3.0.11: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
npm WARN deprecated minimatch@2.0.10: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated minimatch@0.2.14: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated graceful-fs@1.2.3: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
cardsjekyll@1.0.0 /vagrant/git/biscuits
├─┬ gulp@3.9.1 
│ ├── archy@1.0.0 
│ ├─┬ chalk@1.1.3 
│ │ ├── ansi-styles@2.2.1 
│ │ ├── escape-string-regexp@1.0.5 
│ │ ├─┬ has-ansi@2.0.0 
│ │ │ └── ansi-regex@2.1.1 
│ │ ├── strip-ansi@3.0.1 
│ │ └── supports-color@2.0.0 
│ ├── deprecated@0.0.1 
│ ├─┬ gulp-util@3.0.8 
│ │ ├── array-differ@1.0.0 
│ │ ├── array-uniq@1.0.3 
│ │ ├── beeper@1.1.1 
│ │ ├── dateformat@2.2.0 
│ │ ├─┬ fancy-log@1.3.2 
│ │ │ ├─┬ ansi-gray@0.1.1 
│ │ │ │ └── ansi-wrap@0.1.0 
│ │ │ ├── color-support@1.1.3 
│ │ │ └── time-stamp@1.1.0 
│ │ ├─┬ gulplog@1.0.0 
│ │ │ └── glogg@1.0.1 
│ │ ├─┬ has-gulplog@0.1.0 
│ │ │ └── sparkles@1.0.1 
│ │ ├── lodash._reescape@3.0.0 
│ │ ├── lodash._reevaluate@3.0.0 
│ │ ├── lodash._reinterpolate@3.0.0 
│ │ ├─┬ lodash.template@3.6.2 
│ │ │ ├── lodash._basecopy@3.0.1 
│ │ │ ├── lodash._basetostring@3.0.1 
│ │ │ ├── lodash._basevalues@3.0.0 
│ │ │ ├── lodash._isiterateecall@3.0.9 
│ │ │ ├─┬ lodash.escape@3.2.0 
│ │ │ │ └── lodash._root@3.0.1 
│ │ │ ├─┬ lodash.keys@3.1.2 
│ │ │ │ ├── lodash._getnative@3.9.1 
│ │ │ │ ├── lodash.isarguments@3.1.0 
│ │ │ │ └── lodash.isarray@3.0.4 
│ │ │ ├── lodash.restparam@3.6.1 
│ │ │ └── lodash.templatesettings@3.1.1 
│ │ ├─┬ multipipe@0.1.2 
│ │ │ └─┬ duplexer2@0.0.2 
│ │ │   └── readable-stream@1.1.14 
│ │ ├── object-assign@3.0.0 
│ │ ├── replace-ext@0.0.1 
│ │ ├─┬ through2@2.0.3 
│ │ │ ├─┬ readable-stream@2.3.6 
│ │ │ │ ├── core-util-is@1.0.2 
│ │ │ │ ├── inherits@2.0.3 
│ │ │ │ ├── isarray@1.0.0 
│ │ │ │ ├── process-nextick-args@2.0.0 
│ │ │ │ ├── safe-buffer@5.1.2 
│ │ │ │ ├── string_decoder@1.1.1 
│ │ │ │ └── util-deprecate@1.0.2 
│ │ │ └── xtend@4.0.1 
│ │ └─┬ vinyl@0.5.3 
│ │   ├── clone@1.0.4 
│ │   └── clone-stats@0.0.1 
│ ├── interpret@1.1.0 
│ ├─┬ liftoff@2.5.0 
│ │ ├── extend@3.0.1 
│ │ ├─┬ findup-sync@2.0.0 
│ │ │ ├── detect-file@1.0.0 
│ │ │ ├─┬ is-glob@3.1.0 
│ │ │ │ └── is-extglob@2.1.1 
│ │ │ ├─┬ micromatch@3.1.10 
│ │ │ │ ├── arr-diff@4.0.0 
│ │ │ │ ├── array-unique@0.3.2 
│ │ │ │ ├─┬ braces@2.3.2 
│ │ │ │ │ ├── arr-flatten@1.1.0 
│ │ │ │ │ ├─┬ extend-shallow@2.0.1 
│ │ │ │ │ │ └── is-extendable@0.1.1 
│ │ │ │ │ ├─┬ fill-range@4.0.0 
│ │ │ │ │ │ ├── extend-shallow@2.0.1 
│ │ │ │ │ │ ├─┬ is-number@3.0.0 
│ │ │ │ │ │ │ └─┬ kind-of@3.2.2 
│ │ │ │ │ │ │   └── is-buffer@1.1.6 
│ │ │ │ │ │ ├── repeat-string@1.6.1 
│ │ │ │ │ │ └── to-regex-range@2.1.1 
│ │ │ │ │ ├── repeat-element@1.1.2 
│ │ │ │ │ ├─┬ snapdragon-node@2.1.1 
│ │ │ │ │ │ ├─┬ define-property@1.0.0 
│ │ │ │ │ │ │ └─┬ is-descriptor@1.0.2 
│ │ │ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
│ │ │ │ │ │ │   └── is-data-descriptor@1.0.0 
│ │ │ │ │ │ └─┬ snapdragon-util@3.0.1 
│ │ │ │ │ │   └── kind-of@3.2.2 
│ │ │ │ │ └── split-string@3.1.0 
│ │ │ │ ├─┬ define-property@2.0.2 
│ │ │ │ │ └─┬ is-descriptor@1.0.2 
│ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
│ │ │ │ │   └── is-data-descriptor@1.0.0 
│ │ │ │ ├─┬ extend-shallow@3.0.2 
│ │ │ │ │ ├── assign-symbols@1.0.0 
│ │ │ │ │ └── is-extendable@1.0.1 
│ │ │ │ ├─┬ extglob@2.0.4 
│ │ │ │ │ ├─┬ define-property@1.0.0 
│ │ │ │ │ │ └─┬ is-descriptor@1.0.2 
│ │ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
│ │ │ │ │ │   └── is-data-descriptor@1.0.0 
│ │ │ │ │ ├─┬ expand-brackets@2.1.4 
│ │ │ │ │ │ ├── define-property@0.2.5 
│ │ │ │ │ │ ├── extend-shallow@2.0.1 
│ │ │ │ │ │ └── posix-character-classes@0.1.1 
│ │ │ │ │ └── extend-shallow@2.0.1 
│ │ │ │ ├── fragment-cache@0.2.1 
│ │ │ │ ├── kind-of@6.0.2 
│ │ │ │ ├─┬ nanomatch@1.2.13 
│ │ │ │ │ └── is-windows@1.0.2 
│ │ │ │ ├─┬ regex-not@1.0.2 
│ │ │ │ │ └─┬ safe-regex@1.1.0 
│ │ │ │ │   └── ret@0.1.15 
│ │ │ │ ├─┬ snapdragon@0.8.2 
│ │ │ │ │ ├─┬ base@0.11.2 
│ │ │ │ │ │ ├─┬ cache-base@1.0.1 
│ │ │ │ │ │ │ ├─┬ collection-visit@1.0.0 
│ │ │ │ │ │ │ │ ├── map-visit@1.0.0 
│ │ │ │ │ │ │ │ └── object-visit@1.0.1 
│ │ │ │ │ │ │ ├── get-value@2.0.6 
│ │ │ │ │ │ │ ├─┬ has-value@1.0.0 
│ │ │ │ │ │ │ │ └─┬ has-values@1.0.0 
│ │ │ │ │ │ │ │   └── kind-of@4.0.0 
│ │ │ │ │ │ │ ├─┬ set-value@2.0.0 
│ │ │ │ │ │ │ │ └── extend-shallow@2.0.1 
│ │ │ │ │ │ │ ├─┬ to-object-path@0.3.0 
│ │ │ │ │ │ │ │ └── kind-of@3.2.2 
│ │ │ │ │ │ │ ├─┬ union-value@1.0.0 
│ │ │ │ │ │ │ │ └─┬ set-value@0.4.3 
│ │ │ │ │ │ │ │   └── extend-shallow@2.0.1 
│ │ │ │ │ │ │ └─┬ unset-value@1.0.0 
│ │ │ │ │ │ │   └─┬ has-value@0.3.1 
│ │ │ │ │ │ │     ├── has-values@0.1.4 
│ │ │ │ │ │ │     └─┬ isobject@2.1.0 
│ │ │ │ │ │ │       └── isarray@1.0.0 
│ │ │ │ │ │ ├─┬ class-utils@0.3.6 
│ │ │ │ │ │ │ ├── arr-union@3.1.0 
│ │ │ │ │ │ │ ├── define-property@0.2.5 
│ │ │ │ │ │ │ └─┬ static-extend@0.1.2 
│ │ │ │ │ │ │   ├── define-property@0.2.5 
│ │ │ │ │ │ │   └─┬ object-copy@0.1.0 
│ │ │ │ │ │ │     ├── copy-descriptor@0.1.1 
│ │ │ │ │ │ │     ├── define-property@0.2.5 
│ │ │ │ │ │ │     └── kind-of@3.2.2 
│ │ │ │ │ │ ├── component-emitter@1.2.1 
│ │ │ │ │ │ ├─┬ define-property@1.0.0 
│ │ │ │ │ │ │ └─┬ is-descriptor@1.0.2 
│ │ │ │ │ │ │   ├── is-accessor-descriptor@1.0.0 
│ │ │ │ │ │ │   └── is-data-descriptor@1.0.0 
│ │ │ │ │ │ ├─┬ mixin-deep@1.3.1 
│ │ │ │ │ │ │ └── is-extendable@1.0.1 
│ │ │ │ │ │ └── pascalcase@0.1.1 
│ │ │ │ │ ├─┬ debug@2.6.9 
│ │ │ │ │ │ └── ms@2.0.0 
│ │ │ │ │ ├─┬ define-property@0.2.5 
│ │ │ │ │ │ └─┬ is-descriptor@0.1.6 
│ │ │ │ │ │   ├─┬ is-accessor-descriptor@0.1.6 
│ │ │ │ │ │   │ └── kind-of@3.2.2 
│ │ │ │ │ │   ├─┬ is-data-descriptor@0.1.4 
│ │ │ │ │ │   │ └── kind-of@3.2.2 
│ │ │ │ │ │   └── kind-of@5.1.0 
│ │ │ │ │ ├── extend-shallow@2.0.1 
│ │ │ │ │ ├── source-map@0.5.7 
│ │ │ │ │ ├─┬ source-map-resolve@0.5.2 
│ │ │ │ │ │ ├── atob@2.1.1 
│ │ │ │ │ │ ├── decode-uri-component@0.2.0 
│ │ │ │ │ │ ├── resolve-url@0.2.1 
│ │ │ │ │ │ ├── source-map-url@0.4.0 
│ │ │ │ │ │ └── urix@0.1.0 
│ │ │ │ │ └── use@3.1.0 
│ │ │ │ └── to-regex@3.0.2 
│ │ │ └─┬ resolve-dir@1.0.1 
│ │ │   └─┬ global-modules@1.0.0 
│ │ │     └─┬ global-prefix@1.0.2 
│ │ │       ├── ini@1.3.5 
│ │ │       └─┬ which@1.3.1 
│ │ │         └── isexe@2.0.0 
│ │ ├─┬ fined@1.1.0 
│ │ │ ├─┬ expand-tilde@2.0.2 
│ │ │ │ └─┬ homedir-polyfill@1.0.1 
│ │ │ │   └── parse-passwd@1.0.0 
│ │ │ ├─┬ object.defaults@1.1.0 
│ │ │ │ ├── array-each@1.0.1 
│ │ │ │ └── array-slice@1.1.0 
│ │ │ ├── object.pick@1.3.0 
│ │ │ └─┬ parse-filepath@1.0.2 
│ │ │   ├─┬ is-absolute@1.0.0 
│ │ │   │ └─┬ is-relative@1.0.0 
│ │ │   │   └─┬ is-unc-path@1.0.0 
│ │ │   │     └── unc-path-regex@0.1.2 
│ │ │   ├── map-cache@0.2.2 
│ │ │   └─┬ path-root@0.1.1 
│ │ │     └── path-root-regex@0.1.2 
│ │ ├── flagged-respawn@1.0.0 
│ │ ├─┬ is-plain-object@2.0.4 
│ │ │ └── isobject@3.0.1 
│ │ ├─┬ object.map@1.0.1 
│ │ │ ├─┬ for-own@1.0.0 
│ │ │ │ └── for-in@1.0.2 
│ │ │ └── make-iterator@1.0.1 
│ │ ├── rechoir@0.6.2 
│ │ └─┬ resolve@1.8.1 
│ │   └── path-parse@1.0.5 
│ ├─┬ orchestrator@0.3.8 
│ │ ├─┬ end-of-stream@0.1.5 
│ │ │ └─┬ once@1.3.3 
│ │ │   └── wrappy@1.0.2 
│ │ ├── sequencify@0.0.7 
│ │ └── stream-consume@0.1.1 
│ ├── pretty-hrtime@1.0.3 
│ ├── semver@4.3.6 
│ ├─┬ tildify@1.2.0 
│ │ └── os-homedir@1.0.2 
│ ├─┬ v8flags@2.1.1 
│ │ └── user-home@1.1.1 
│ └─┬ vinyl-fs@0.3.14 
│   ├── defaults@1.0.3 
│   ├─┬ glob-stream@3.1.18 
│   │ ├─┬ glob@4.5.3 
│   │ │ └── inflight@1.0.6 
│   │ ├─┬ glob2base@0.0.12 
│   │ │ └── find-index@0.1.1 
│   │ ├─┬ minimatch@2.0.10 
│   │ │ └─┬ brace-expansion@1.1.11 
│   │ │   ├── balanced-match@1.0.0 
│   │ │   └── concat-map@0.0.1 
│   │ ├── ordered-read-streams@0.1.0 
│   │ ├─┬ through2@0.6.5 
│   │ │ └── readable-stream@1.0.34 
│   │ └── unique-stream@1.0.0 
│   ├─┬ glob-watcher@0.0.6 
│   │ └─┬ gaze@0.5.2 
│   │   └─┬ globule@0.1.0 
│   │     ├─┬ glob@3.1.21 
│   │     │ ├── graceful-fs@1.2.3 
│   │     │ └── inherits@1.0.2 
│   │     ├── lodash@1.0.2 
│   │     └─┬ minimatch@0.2.14 
│   │       ├── lru-cache@2.7.3 
│   │       └── sigmund@1.0.1 
│   ├─┬ graceful-fs@3.0.11 
│   │ └── natives@1.1.4 
│   ├─┬ mkdirp@0.5.1 
│   │ └── minimist@0.0.8 
│   ├─┬ strip-bom@1.0.0 
│   │ ├── first-chunk-stream@1.0.0 
│   │ └── is-utf8@0.2.1 
│   ├─┬ through2@0.6.5 
│   │ └─┬ readable-stream@1.0.34 
│   │   ├── isarray@0.0.1 
│   │   └── string_decoder@0.10.31 
│   └─┬ vinyl@0.4.6 
│     └── clone@0.2.0 
└── minimist@1.2.0 

npm WARN cardsjekyll@1.0.0 No repository field.
[vagrant@h105 biscuits]$ 
[vagrant@h105 biscuits]$ gulp -f gulpfile.js 
module.js:478
    throw err;
    ^

Error: Cannot find module 'gulp-plumber'
    at Function.Module._resolveFilename (module.js:476:15)
    at Function.Module._load (module.js:424:25)
    at Module.require (module.js:504:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/vagrant/git/biscuits/gulpfile.js:3:16)
    at Module._compile (module.js:577:32)
    at Object.Module._extensions..js (module.js:586:10)
    at Module.load (module.js:494:32)
    at tryModuleLoad (module.js:453:12)
    at Function.Module._load (module.js:445:3)
[vagrant@h105 biscuits]$ vim gulpfile.js 
[vagrant@h105 biscuits]$ grep require gulpfile.js 
var env         = require('minimist')(process.argv.slice(2)),
	gulp        = require('gulp'),
	plumber     = require('gulp-plumber'),
	browserSync = require('browser-sync'),
	stylus      = require('gulp-stylus'),
	uglify      = require('gulp-uglify'),
	concat      = require('gulp-concat'),
	jeet        = require('jeet'),
	rupture     = require('rupture'),
	koutoSwiss  = require('kouto-swiss'),
	prefixer    = require('autoprefixer-stylus'),
	imagemin    = require('gulp-imagemin'),
	cp          = require('child_process');
[vagrant@h105 biscuits]$ npm install gulp-plumber
cardsjekyll@1.0.0 /vagrant/git/biscuits
└─┬ gulp-plumber@0.6.6 
  └─┬ through2@0.6.5 
    └── readable-stream@1.0.34 

npm WARN cardsjekyll@1.0.0 No repository field.
[vagrant@h105 biscuits]$ gulp -f gulpfile.js 
module.js:478
    throw err;
    ^

Error: Cannot find module 'browser-sync'
    at Function.Module._resolveFilename (module.js:476:15)
    at Function.Module._load (module.js:424:25)
    at Module.require (module.js:504:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/vagrant/git/biscuits/gulpfile.js:4:16)
    at Module._compile (module.js:577:32)
    at Object.Module._extensions..js (module.js:586:10)
    at Module.load (module.js:494:32)
    at tryModuleLoad (module.js:453:12)
    at Function.Module._load (module.js:445:3)
[vagrant@h105 biscuits]$ npm install gulp gulp-plumber browser-sync gulp-stylus gulp-uglify gulp-concat jeet rupture kouto-swiss autoprefixer-stylus gulp-imagemin child_process 
npm WARN deprecated minimatch@2.0.10: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated minimatch@0.2.14: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated graceful-fs@1.2.3: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
npm WARN deprecated gulp-util@3.0.8: gulp-util is deprecated - replace it, following the guidelines at https://medium.com/gulpjs/gulp-util-ca3b1f9f9ac5
npm WARN deprecated graceful-fs@3.0.11: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
npm WARN deprecated minimatch@0.3.0: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue

> gifsicle@3.0.4 postinstall /vagrant/git/biscuits/node_modules/gifsicle
> node lib/install.js

  ✔ gifsicle pre-build test passed successfully

> jpegtran-bin@3.2.0 postinstall /vagrant/git/biscuits/node_modules/jpegtran-bin
> node lib/install.js

  ✔ jpegtran pre-build test passed successfully

> optipng-bin@3.1.4 postinstall /vagrant/git/biscuits/node_modules/optipng-bin
> node lib/install.js

  ✔ optipng pre-build test passed successfully
- graceful-fs@1.2.3 node_modules/globule/node_modules/graceful-fs
- inherits@1.0.2 node_modules/globule/node_modules/inherits
- glob@3.1.21 node_modules/globule/node_modules/glob
cardsjekyll@1.0.0 /vagrant/git/biscuits
├─┬ autoprefixer-stylus@0.4.0 
│ └─┬ autoprefixer-core@4.0.2 
│   ├── caniuse-db@1.0.30000862 
│   └─┬ postcss@3.0.7 
│     └── js-base64@2.1.9 
├─┬ browser-sync@1.9.2 
│ ├── browser-sync-client@1.0.2 
│ ├── commander@2.16.0 
│ ├─┬ connect@3.6.6 
│ │ ├─┬ finalhandler@1.1.0 
│ │ │ ├─┬ on-finished@2.3.0 
│ │ │ │ └── ee-first@1.1.1 
│ │ │ ├── statuses@1.3.1 
│ │ │ └── unpipe@1.0.0 
│ │ ├── parseurl@1.3.2 
│ │ └── utils-merge@1.0.1 
│ ├─┬ dev-ip@0.1.7 
│ │ └── lodash@2.2.1 
│ ├─┬ easy-extender@2.3.2 
│ │ └── lodash@3.10.1 
│ ├─┬ eazy-logger@2.1.3 
│ │ └─┬ lodash.clonedeep@4.3.1 
│ │   └── lodash._baseclone@4.5.7 
│ ├── emitter-steward@0.0.1 
│ ├─┬ foxy@7.1.0 
│ │ ├── cookie@0.1.5 
│ │ ├── dev-ip@1.0.1 
│ │ ├─┬ http-proxy@1.17.0 
│ │ │ ├── eventemitter3@3.1.0 
│ │ │ ├─┬ follow-redirects@1.5.0 
│ │ │ │ └── debug@3.1.0 
│ │ │ └── requires-port@1.0.0 
│ │ ├── immutable@3.8.2 
│ │ └─┬ meow@2.1.0 
│ │   ├── camelcase-keys@1.0.0 
│ │   ├─┬ indent-string@1.2.2 
│ │   │ └─┬ repeating@1.1.3 
│ │   │   └── is-finite@1.0.2 
│ │   └── object-assign@2.1.1 
│ ├─┬ glob-watcher@0.0.7 
│ │ └─┬ gaze@0.5.2
│ │   └─┬ globule@0.1.0
│ │     ├─┬ glob@3.1.21 
│ │     │ ├── graceful-fs@1.2.3 
│ │     │ ├── inherits@1.0.2 
│ │     │ └── minimatch@0.2.14 
│ │     └── lodash@1.0.2 
│ ├─┬ localtunnel@1.9.0 
│ │ ├── axios@0.17.1 
│ │ ├── debug@2.6.8 
│ │ ├── openurl@1.1.1 
│ │ └─┬ yargs@6.6.0 
│ │   ├── camelcase@3.0.0 
│ │   ├─┬ cliui@3.2.0 
│ │   │ └── wrap-ansi@2.1.0 
│ │   ├── decamelize@1.2.0 
│ │   ├── get-caller-file@1.0.2 
│ │   ├─┬ os-locale@1.4.0 
│ │   │ └─┬ lcid@1.0.0 
│ │   │   └── invert-kv@1.0.0 
│ │   ├─┬ read-pkg-up@1.0.1 
│ │   │ ├─┬ find-up@1.1.2 
│ │   │ │ ├── path-exists@2.1.0 
│ │   │ │ └─┬ pinkie-promise@2.0.1 
│ │   │ │   └── pinkie@2.0.4 
│ │   │ └─┬ read-pkg@1.1.0 
│ │   │   ├─┬ load-json-file@1.1.0 
│ │   │   │ ├── graceful-fs@4.1.11 
│ │   │   │ ├─┬ parse-json@2.2.0 
│ │   │   │ │ └─┬ error-ex@1.3.2 
│ │   │   │ │   └── is-arrayish@0.2.1 
│ │   │   │ └── pify@2.3.0 
│ │   │   └─┬ path-type@1.1.0 
│ │   │     └── graceful-fs@4.1.11 
│ │   ├── require-directory@2.1.1 
│ │   ├── require-main-filename@1.0.1 
│ │   ├── set-blocking@2.0.0 
│ │   ├─┬ string-width@1.0.2 
│ │   │ ├── code-point-at@1.1.0 
│ │   │ └── is-fullwidth-code-point@1.0.0 
│ │   ├── which-module@1.0.0 
│ │   ├── y18n@3.2.1 
│ │   └─┬ yargs-parser@4.2.1 
│ │     └── camelcase@3.0.0 
│ ├── lodash@2.4.2 
│ ├── object-path@0.8.1 
│ ├── opn@1.0.2 
│ ├─┬ opt-merger@1.1.1 
│ │ └── lodash@3.10.1 
│ ├─┬ portscanner-plus@0.2.1 
│ │ ├─┬ portscanner@1.2.0 
│ │ │ └── async@1.5.2 
│ │ └── q@1.5.1 
│ ├── resp-modifier@1.0.2 
│ ├─┬ serve-index@1.9.1 
│ │ ├─┬ accepts@1.3.5 
│ │ │ └── negotiator@0.6.1 
│ │ ├── batch@0.6.1 
│ │ ├── escape-html@1.0.3 
│ │ ├─┬ http-errors@1.6.3 
│ │ │ ├── depd@1.1.2 
│ │ │ ├── inherits@2.0.3 
│ │ │ ├── setprototypeof@1.1.0 
│ │ │ └── statuses@1.5.0 
│ │ └─┬ mime-types@2.1.18 
│ │   └── mime-db@1.33.0 
│ ├─┬ serve-static@1.13.2 
│ │ ├── encodeurl@1.0.2 
│ │ └─┬ send@0.16.2 
│ │   ├── destroy@1.0.4 
│ │   ├── etag@1.8.1 
│ │   ├── fresh@0.5.2 
│ │   ├── mime@1.4.1 
│ │   ├── range-parser@1.2.0 
│ │   └── statuses@1.4.0 
│ ├─┬ socket.io@1.7.4 
│ │ ├─┬ debug@2.3.3 
│ │ │ └── ms@0.7.2 
│ │ ├─┬ engine.io@1.8.5 
│ │ │ ├── accepts@1.3.3 
│ │ │ ├── base64id@1.0.0 
│ │ │ ├── cookie@0.3.1 
│ │ │ ├─┬ debug@2.3.3 
│ │ │ │ └── ms@0.7.2 
│ │ │ ├─┬ engine.io-parser@1.3.2 
│ │ │ │ ├── after@0.8.2 
│ │ │ │ ├── arraybuffer.slice@0.0.6 
│ │ │ │ ├── base64-arraybuffer@0.1.5 
│ │ │ │ ├── blob@0.0.4 
│ │ │ │ └── wtf-8@1.0.0 
│ │ │ └─┬ ws@1.1.5 
│ │ │   ├── options@0.0.6 
│ │ │   └── ultron@1.0.2 
│ │ ├── has-binary@0.1.7 
│ │ ├── object-assign@4.1.0 
│ │ ├─┬ socket.io-adapter@0.5.0 
│ │ │ └─┬ debug@2.3.3 
│ │ │   └── ms@0.7.2 
│ │ ├─┬ socket.io-client@1.7.4 
│ │ │ ├── backo2@1.0.2 
│ │ │ ├── component-bind@1.0.0 
│ │ │ ├── component-emitter@1.2.1 
│ │ │ ├─┬ debug@2.3.3 
│ │ │ │ └── ms@0.7.2 
│ │ │ ├─┬ engine.io-client@1.8.5 
│ │ │ │ ├── component-emitter@1.2.1 
│ │ │ │ ├── component-inherit@0.0.3 
│ │ │ │ ├─┬ debug@2.3.3 
│ │ │ │ │ └── ms@0.7.2 
│ │ │ │ ├── has-cors@1.1.0 
│ │ │ │ ├── parsejson@0.0.3 
│ │ │ │ ├── parseqs@0.0.5 
│ │ │ │ ├── xmlhttprequest-ssl@1.5.3 
│ │ │ │ └── yeast@0.1.2 
│ │ │ ├── indexof@0.0.1 
│ │ │ ├── object-component@0.0.3 
│ │ │ ├─┬ parseuri@0.0.5 
│ │ │ │ └─┬ better-assert@1.0.2 
│ │ │ │   └── callsite@1.0.0 
│ │ │ └── to-array@0.1.4 
│ │ └─┬ socket.io-parser@2.3.1 
│ │   ├── component-emitter@1.1.2 
│ │   ├─┬ debug@2.2.0 
│ │   │ └── ms@0.7.1 
│ │   └── json3@3.3.2 
│ ├─┬ tfunk@3.1.0 
│ │ └── object-path@0.9.2 
│ └── ua-parser-js@0.7.18 
├── child_process@1.0.2 
├─┬ gulp@3.9.1 
│ ├─┬ gulp-util@3.0.8
│ │ ├─┬ multipipe@0.1.2
│ │ │ └─┬ duplexer2@0.0.2
│ │ │   └─┬ readable-stream@1.1.14
│ │ │     └── inherits@2.0.3 
│ │ └── object-assign@3.0.0 
│ ├─┬ liftoff@2.5.0
│ │ └─┬ findup-sync@2.0.0
│ │   └─┬ micromatch@3.1.10
│ │     └─┬ snapdragon@0.8.2
│ │       ├─┬ base@0.11.2
│ │       │ ├─┬ cache-base@1.0.1
│ │       │ │ └── component-emitter@1.2.1 
│ │       │ └── component-emitter@1.2.1 
│ │       └── source-map@0.5.7 
│ ├── semver@4.3.6 
│ └─┬ vinyl-fs@0.3.14
│   ├─┬ glob-stream@3.1.18
│   │ └─┬ glob@4.5.3 
│   │   └── inherits@2.0.3 
│   ├── glob-watcher@0.0.6 
│   ├── graceful-fs@3.0.11 
│   ├── strip-bom@1.0.0 
│   └─┬ through2@0.6.5
│     └─┬ readable-stream@1.0.34
│       └── inherits@2.0.3 
├─┬ gulp-concat@2.6.1 
│ ├─┬ concat-with-sourcemaps@1.1.0 
│ │ └── source-map@0.6.1 
│ ├─┬ through2@2.0.3
│ │ └─┬ readable-stream@2.3.6
│ │   └── inherits@2.0.3 
│ └─┬ vinyl@2.2.0 
│   ├── clone@2.1.1 
│   ├── clone-buffer@1.0.0 
│   ├── clone-stats@1.0.0 
│   ├─┬ cloneable-readable@1.1.2 
│   │ ├── inherits@2.0.3 
│   │ └─┬ readable-stream@2.3.6 
│   │   ├── isarray@1.0.0 
│   │   └── string_decoder@1.1.1 
│   ├── remove-trailing-separator@1.1.0 
│   └── replace-ext@1.0.0 
├─┬ gulp-imagemin@2.4.0 
│ ├─┬ imagemin@4.0.0 
│ │ ├─┬ buffer-to-vinyl@1.1.0 
│ │ │ ├── file-type@3.9.0 
│ │ │ ├─┬ readable-stream@2.3.6 
│ │ │ │ ├── inherits@2.0.3 
│ │ │ │ ├── isarray@1.0.0 
│ │ │ │ └── string_decoder@1.1.1 
│ │ │ ├── uuid@2.0.3 
│ │ │ └── vinyl@1.2.0 
│ │ ├─┬ concat-stream@1.6.2 
│ │ │ ├── buffer-from@1.1.0 
│ │ │ ├── inherits@2.0.3 
│ │ │ ├─┬ readable-stream@2.3.6 
│ │ │ │ ├── isarray@1.0.0 
│ │ │ │ └── string_decoder@1.1.1 
│ │ │ └── typedarray@0.0.6 
│ │ ├─┬ imagemin-gifsicle@4.2.0 
│ │ │ ├─┬ gifsicle@3.0.4 
│ │ │ │ ├─┬ bin-build@2.2.0 
│ │ │ │ │ ├── archive-type@3.2.0 
│ │ │ │ │ ├─┬ decompress@3.0.0 
│ │ │ │ │ │ ├─┬ decompress-tar@3.1.0 
│ │ │ │ │ │ │ ├── is-tar@1.0.0 
│ │ │ │ │ │ │ ├─┬ strip-dirs@1.1.1 
│ │ │ │ │ │ │ │ ├─┬ is-absolute@0.1.7 
│ │ │ │ │ │ │ │ │ └── is-relative@0.1.3 
│ │ │ │ │ │ │ │ ├── is-natural-number@2.1.1 
│ │ │ │ │ │ │ │ └── sum-up@1.0.3 
│ │ │ │ │ │ │ ├─┬ tar-stream@1.6.1 
│ │ │ │ │ │ │ │ ├─┬ bl@1.2.2 
│ │ │ │ │ │ │ │ │ └─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │ │ │   ├── inherits@2.0.3 
│ │ │ │ │ │ │ │ │   ├── isarray@1.0.0 
│ │ │ │ │ │ │ │ │   └── string_decoder@1.1.1 
│ │ │ │ │ │ │ │ ├─┬ buffer-alloc@1.2.0 
│ │ │ │ │ │ │ │ │ ├── buffer-alloc-unsafe@1.1.0 
│ │ │ │ │ │ │ │ │ └── buffer-fill@1.0.0 
│ │ │ │ │ │ │ │ ├─┬ end-of-stream@1.4.1 
│ │ │ │ │ │ │ │ │ └── once@1.4.0 
│ │ │ │ │ │ │ │ ├── fs-constants@1.0.0 
│ │ │ │ │ │ │ │ ├─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │ │ │ ├── inherits@2.0.3 
│ │ │ │ │ │ │ │ │ ├── isarray@1.0.0 
│ │ │ │ │ │ │ │ │ └── string_decoder@1.1.1 
│ │ │ │ │ │ │ │ └── to-buffer@1.1.1 
│ │ │ │ │ │ │ ├─┬ through2@0.6.5 
│ │ │ │ │ │ │ │ └─┬ readable-stream@1.0.34 
│ │ │ │ │ │ │ │   └── inherits@2.0.3 
│ │ │ │ │ │ │ └─┬ vinyl@0.4.6 
│ │ │ │ │ │ │   └── clone@0.2.0 
│ │ │ │ │ │ ├─┬ decompress-tarbz2@3.1.0 
│ │ │ │ │ │ │ ├── is-bzip2@1.0.0 
│ │ │ │ │ │ │ ├─┬ seek-bzip@1.0.5 
│ │ │ │ │ │ │ │ └─┬ commander@2.8.1 
│ │ │ │ │ │ │ │   └── graceful-readlink@1.0.1 
│ │ │ │ │ │ │ ├─┬ through2@0.6.5 
│ │ │ │ │ │ │ │ └─┬ readable-stream@1.0.34 
│ │ │ │ │ │ │ │   └── inherits@2.0.3 
│ │ │ │ │ │ │ └─┬ vinyl@0.4.6 
│ │ │ │ │ │ │   └── clone@0.2.0 
│ │ │ │ │ │ ├─┬ decompress-targz@3.1.0 
│ │ │ │ │ │ │ ├── is-gzip@1.0.0 
│ │ │ │ │ │ │ ├─┬ through2@0.6.5 
│ │ │ │ │ │ │ │ └─┬ readable-stream@1.0.34 
│ │ │ │ │ │ │ │   └── inherits@2.0.3 
│ │ │ │ │ │ │ └─┬ vinyl@0.4.6 
│ │ │ │ │ │ │   └── clone@0.2.0 
│ │ │ │ │ │ ├─┬ decompress-unzip@3.4.0 
│ │ │ │ │ │ │ ├── is-zip@1.0.0 
│ │ │ │ │ │ │ ├── stat-mode@0.2.2 
│ │ │ │ │ │ │ ├── vinyl@1.2.0 
│ │ │ │ │ │ │ └─┬ yauzl@2.10.0 
│ │ │ │ │ │ │   ├── buffer-crc32@0.2.13 
│ │ │ │ │ │ │   └─┬ fd-slicer@1.1.0 
│ │ │ │ │ │ │     └── pend@1.2.0 
│ │ │ │ │ │ ├─┬ vinyl-assign@1.2.1 
│ │ │ │ │ │ │ ├── object-assign@4.1.1 
│ │ │ │ │ │ │ └─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │   ├── inherits@2.0.3 
│ │ │ │ │ │ │   ├── isarray@1.0.0 
│ │ │ │ │ │ │   └── string_decoder@1.1.1 
│ │ │ │ │ │ └─┬ vinyl-fs@2.4.4 
│ │ │ │ │ │   ├─┬ glob-stream@5.3.5 
│ │ │ │ │ │   │ ├── glob@5.0.15 
│ │ │ │ │ │   │ ├─┬ micromatch@2.3.11 
│ │ │ │ │ │   │ │ ├── arr-diff@2.0.0 
│ │ │ │ │ │   │ │ ├── array-unique@0.2.1 
│ │ │ │ │ │   │ │ ├── braces@1.8.5 
│ │ │ │ │ │   │ │ ├── expand-brackets@0.1.5 
│ │ │ │ │ │   │ │ ├── extglob@0.3.2 
│ │ │ │ │ │   │ │ ├── is-extglob@1.0.0 
│ │ │ │ │ │   │ │ ├── is-glob@2.0.1 
│ │ │ │ │ │   │ │ └── kind-of@3.2.2 
│ │ │ │ │ │   │ ├── ordered-read-streams@0.3.0 
│ │ │ │ │ │   │ ├─┬ through2@0.6.5 
│ │ │ │ │ │   │ │ └─┬ readable-stream@1.0.34 
│ │ │ │ │ │   │ │   ├── isarray@0.0.1 
│ │ │ │ │ │   │ │   └── string_decoder@0.10.31 
│ │ │ │ │ │   │ └── unique-stream@2.2.1 
│ │ │ │ │ │   ├── graceful-fs@4.1.11 
│ │ │ │ │ │   ├── object-assign@4.1.1 
│ │ │ │ │ │   ├─┬ readable-stream@2.3.6 
│ │ │ │ │ │   │ ├── inherits@2.0.3 
│ │ │ │ │ │   │ ├── isarray@1.0.0 
│ │ │ │ │ │   │ └── string_decoder@1.1.1 
│ │ │ │ │ │   └── vinyl@1.2.0 
│ │ │ │ │ ├─┬ download@4.4.3 
│ │ │ │ │ │ ├─┬ caw@1.2.0 
│ │ │ │ │ │ │ ├─┬ get-proxy@1.1.0 
│ │ │ │ │ │ │ │ └─┬ rc@1.2.8 
│ │ │ │ │ │ │ │   ├── deep-extend@0.6.0 
│ │ │ │ │ │ │ │   └── strip-json-comments@2.0.1 
│ │ │ │ │ │ │ ├── is-obj@1.0.1 
│ │ │ │ │ │ │ ├── object-assign@3.0.0 
│ │ │ │ │ │ │ └── tunnel-agent@0.4.3 
│ │ │ │ │ │ ├─┬ filenamify@1.2.1 
│ │ │ │ │ │ │ ├── filename-reserved-regex@1.0.0 
│ │ │ │ │ │ │ ├── strip-outer@1.0.1 
│ │ │ │ │ │ │ └── trim-repeated@1.0.0 
│ │ │ │ │ │ ├─┬ got@5.7.1 
│ │ │ │ │ │ │ ├─┬ create-error-class@3.0.2 
│ │ │ │ │ │ │ │ └── capture-stack-trace@1.0.0 
│ │ │ │ │ │ │ ├── duplexer2@0.1.4 
│ │ │ │ │ │ │ ├── is-redirect@1.0.0 
│ │ │ │ │ │ │ ├── is-retry-allowed@1.1.0 
│ │ │ │ │ │ │ ├── lowercase-keys@1.0.1 
│ │ │ │ │ │ │ ├── node-status-codes@1.0.0 
│ │ │ │ │ │ │ ├── object-assign@4.1.1 
│ │ │ │ │ │ │ ├─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │ │ ├── inherits@2.0.3 
│ │ │ │ │ │ │ │ ├── isarray@1.0.0 
│ │ │ │ │ │ │ │ └── string_decoder@1.1.1 
│ │ │ │ │ │ │ ├── timed-out@3.1.3 
│ │ │ │ │ │ │ ├── unzip-response@1.0.2 
│ │ │ │ │ │ │ └─┬ url-parse-lax@1.0.0 
│ │ │ │ │ │ │   └── prepend-http@1.0.4 
│ │ │ │ │ │ ├─┬ gulp-decompress@1.2.0 
│ │ │ │ │ │ │ └─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │   ├── inherits@2.0.3 
│ │ │ │ │ │ │   ├── isarray@1.0.0 
│ │ │ │ │ │ │   └── string_decoder@1.1.1 
│ │ │ │ │ │ ├── gulp-rename@1.3.0 
│ │ │ │ │ │ ├── is-url@1.2.4 
│ │ │ │ │ │ ├── object-assign@4.1.1 
│ │ │ │ │ │ ├─┬ read-all-stream@3.1.0 
│ │ │ │ │ │ │ └─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │   ├── inherits@2.0.3 
│ │ │ │ │ │ │   ├── isarray@1.0.0 
│ │ │ │ │ │ │   └── string_decoder@1.1.1 
│ │ │ │ │ │ ├─┬ readable-stream@2.3.6 
│ │ │ │ │ │ │ ├── inherits@2.0.3 
│ │ │ │ │ │ │ ├── isarray@1.0.0 
│ │ │ │ │ │ │ └── string_decoder@1.1.1 
│ │ │ │ │ │ ├── vinyl@1.2.0 
│ │ │ │ │ │ ├─┬ vinyl-fs@2.4.4 
│ │ │ │ │ │ │ ├─┬ glob-stream@5.3.5 
│ │ │ │ │ │ │ │ ├── glob@5.0.15 
│ │ │ │ │ │ │ │ ├─┬ micromatch@2.3.11 
│ │ │ │ │ │ │ │ │ ├── arr-diff@2.0.0 
│ │ │ │ │ │ │ │ │ ├── array-unique@0.2.1 
│ │ │ │ │ │ │ │ │ ├── braces@1.8.5 
│ │ │ │ │ │ │ │ │ ├── expand-brackets@0.1.5 
│ │ │ │ │ │ │ │ │ ├── extglob@0.3.2 
│ │ │ │ │ │ │ │ │ ├── is-extglob@1.0.0 
│ │ │ │ │ │ │ │ │ ├── is-glob@2.0.1 
│ │ │ │ │ │ │ │ │ └── kind-of@3.2.2 
│ │ │ │ │ │ │ │ ├── ordered-read-streams@0.3.0 
│ │ │ │ │ │ │ │ ├─┬ through2@0.6.5 
│ │ │ │ │ │ │ │ │ └─┬ readable-stream@1.0.34 
│ │ │ │ │ │ │ │ │   ├── isarray@0.0.1 
│ │ │ │ │ │ │ │ │   └── string_decoder@0.10.31 
│ │ │ │ │ │ │ │ └── unique-stream@2.2.1 
│ │ │ │ │ │ │ └── graceful-fs@4.1.11 
│ │ │ │ │ │ └─┬ ware@1.3.0 
│ │ │ │ │ │   └─┬ wrap-fn@0.1.5 
│ │ │ │ │ │     └── co@3.1.0 
│ │ │ │ │ ├─┬ exec-series@1.0.3 
│ │ │ │ │ │ ├── async-each-series@1.1.0 
│ │ │ │ │ │ └── object-assign@4.1.1 
│ │ │ │ │ └─┬ url-regex@3.2.0 
│ │ │ │ │   └── ip-regex@1.0.3 
│ │ │ │ ├─┬ bin-wrapper@3.0.2 
│ │ │ │ │ ├─┬ bin-check@2.0.0 
│ │ │ │ │ │ └─┬ executable@1.1.0 
│ │ │ │ │ │   └─┬ meow@3.7.0 
│ │ │ │ │ │     ├─┬ camelcase-keys@2.1.0 
│ │ │ │ │ │     │ └── camelcase@2.1.1 
│ │ │ │ │ │     └── object-assign@4.1.1 
│ │ │ │ │ ├─┬ bin-version-check@2.1.0 
│ │ │ │ │ │ ├─┬ bin-version@1.0.4 
│ │ │ │ │ │ │ └─┬ find-versions@1.2.1 
│ │ │ │ │ │ │   ├─┬ meow@3.7.0 
│ │ │ │ │ │ │   │ ├─┬ camelcase-keys@2.1.0 
│ │ │ │ │ │ │   │ │ └── camelcase@2.1.1 
│ │ │ │ │ │ │   │ └── object-assign@4.1.1 
│ │ │ │ │ │ │   └── semver-regex@1.0.0 
│ │ │ │ │ │ ├── semver@4.3.6 
│ │ │ │ │ │ └── semver-truncate@1.1.2 
│ │ │ │ │ ├─┬ each-async@1.1.1 
│ │ │ │ │ │ ├── onetime@1.1.0 
│ │ │ │ │ │ └── set-immediate-shim@1.0.1 
│ │ │ │ │ ├── lazy-req@1.1.0 
│ │ │ │ │ └── os-filter-obj@1.0.3 
│ │ │ │ └─┬ logalot@2.1.0 
│ │ │ │   ├─┬ figures@1.7.0 
│ │ │ │   │ └── object-assign@4.1.1 
│ │ │ │   └─┬ squeak@1.3.0 
│ │ │ │     ├── console-stream@0.1.1 
│ │ │ │     └─┬ lpad-align@1.1.2 
│ │ │ │       ├─┬ indent-string@2.1.0 
│ │ │ │       │ └── repeating@2.0.1 
│ │ │ │       └─┬ meow@3.7.0 
│ │ │ │         ├─┬ camelcase-keys@2.1.0 
│ │ │ │         │ └── camelcase@2.1.1 
│ │ │ │         └── object-assign@4.1.1 
│ │ │ ├── is-gif@1.0.0 
│ │ │ └─┬ through2@0.6.5 
│ │ │   └─┬ readable-stream@1.0.34 
│ │ │     └── inherits@2.0.3 
│ │ ├─┬ imagemin-jpegtran@4.3.2 
│ │ │ ├── is-jpg@1.0.1 
│ │ │ └── jpegtran-bin@3.2.0 
│ │ ├─┬ imagemin-optipng@4.3.0 
│ │ │ ├─┬ exec-buffer@2.0.1 
│ │ │ │ ├─┬ rimraf@2.6.2 
│ │ │ │ │ └─┬ glob@7.1.2 
│ │ │ │ │   ├── inherits@2.0.3 
│ │ │ │ │   └── minimatch@3.0.4 
│ │ │ │ └─┬ tempfile@1.1.1 
│ │ │ │   └── os-tmpdir@1.0.2 
│ │ │ ├── is-png@1.1.0 
│ │ │ ├── optipng-bin@3.1.4 
│ │ │ └─┬ through2@0.6.5 
│ │ │   └─┬ readable-stream@1.0.34 
│ │ │     └── inherits@2.0.3 
│ │ ├─┬ imagemin-svgo@4.2.1 
│ │ │ ├── is-svg@1.1.1 
│ │ │ └─┬ svgo@0.6.6 
│ │ │   ├── coa@1.0.4 
│ │ │   ├── colors@1.1.2 
│ │ │   ├─┬ csso@2.0.0 
│ │ │   │ ├── clap@1.2.3 
│ │ │   │ └── source-map@0.5.7 
│ │ │   ├─┬ js-yaml@3.6.1 
│ │ │   │ ├─┬ argparse@1.0.10 
│ │ │   │ │ └── sprintf-js@1.0.3 
│ │ │   │ └── esprima@2.7.3 
│ │ │   ├── sax@1.2.4 
│ │ │   └── whet.extend@0.9.9 
│ │ ├── optional@0.1.4 
│ │ ├─┬ readable-stream@2.3.6 
│ │ │ ├── inherits@2.0.3 
│ │ │ ├── isarray@1.0.0 
│ │ │ └── string_decoder@1.1.1 
│ │ ├─┬ stream-combiner2@1.1.1 
│ │ │ ├── duplexer2@0.1.4 
│ │ │ └─┬ readable-stream@2.3.6 
│ │ │   ├── inherits@2.0.3 
│ │ │   ├── isarray@1.0.0 
│ │ │   └── string_decoder@1.1.1 
│ │ └─┬ vinyl-fs@2.4.4 
│ │   ├─┬ duplexify@3.6.0 
│ │   │ ├─┬ end-of-stream@1.4.1 
│ │   │ │ └── once@1.4.0 
│ │   │ ├── inherits@2.0.3 
│ │   │ ├─┬ readable-stream@2.3.6 
│ │   │ │ ├── isarray@1.0.0 
│ │   │ │ └── string_decoder@1.1.1 
│ │   │ └── stream-shift@1.0.0 
│ │   ├─┬ glob-stream@5.3.5 
│ │   │ ├── glob@5.0.15 
│ │   │ ├─┬ glob-parent@3.1.0 
│ │   │ │ └── path-dirname@1.0.2 
│ │   │ ├─┬ micromatch@2.3.11 
│ │   │ │ ├── arr-diff@2.0.0 
│ │   │ │ ├── array-unique@0.2.1 
│ │   │ │ ├─┬ braces@1.8.5 
│ │   │ │ │ ├─┬ expand-range@1.8.2 
│ │   │ │ │ │ └─┬ fill-range@2.2.4 
│ │   │ │ │ │   ├─┬ is-number@2.1.0 
│ │   │ │ │ │   │ └── kind-of@3.2.2 
│ │   │ │ │ │   ├─┬ isobject@2.1.0 
│ │   │ │ │ │   │ └── isarray@1.0.0 
│ │   │ │ │ │   └─┬ randomatic@3.0.0 
│ │   │ │ │ │     ├── is-number@4.0.0 
│ │   │ │ │ │     └── math-random@1.0.1 
│ │   │ │ │ └── preserve@0.2.0 
│ │   │ │ ├─┬ expand-brackets@0.1.5 
│ │   │ │ │ └── is-posix-bracket@0.1.1 
│ │   │ │ ├── extglob@0.3.2 
│ │   │ │ ├── filename-regex@2.0.1 
│ │   │ │ ├── is-extglob@1.0.0 
│ │   │ │ ├── is-glob@2.0.1 
│ │   │ │ ├── kind-of@3.2.2 
│ │   │ │ ├── normalize-path@2.1.1 
│ │   │ │ ├─┬ object.omit@2.0.1 
│ │   │ │ │ └── for-own@0.1.5 
│ │   │ │ ├─┬ parse-glob@3.0.4 
│ │   │ │ │ ├─┬ glob-base@0.3.0 
│ │   │ │ │ │ ├── glob-parent@2.0.0 
│ │   │ │ │ │ └─┬ is-glob@2.0.1 
│ │   │ │ │ │   └── is-extglob@1.0.0 
│ │   │ │ │ ├── is-dotfile@1.0.3 
│ │   │ │ │ ├── is-extglob@1.0.0 
│ │   │ │ │ └── is-glob@2.0.1 
│ │   │ │ └─┬ regex-cache@0.4.4 
│ │   │ │   └─┬ is-equal-shallow@0.1.3 
│ │   │ │     └── is-primitive@2.0.0 
│ │   │ ├─┬ ordered-read-streams@0.3.0 
│ │   │ │ └── is-stream@1.1.0 
│ │   │ ├─┬ through2@0.6.5 
│ │   │ │ └─┬ readable-stream@1.0.34 
│ │   │ │   ├── isarray@0.0.1 
│ │   │ │   └── string_decoder@0.10.31 
│ │   │ ├─┬ to-absolute-glob@0.1.1 
│ │   │ │ └── extend-shallow@2.0.1 
│ │   │ └─┬ unique-stream@2.2.1 
│ │   │   └─┬ json-stable-stringify@1.0.1 
│ │   │     └── jsonify@0.0.0 
│ │   ├── graceful-fs@4.1.11 
│ │   ├─┬ gulp-sourcemaps@1.6.0 
│ │   │ ├── convert-source-map@1.5.1 
│ │   │ ├── graceful-fs@4.1.11 
│ │   │ └── vinyl@1.2.0 
│ │   ├── is-valid-glob@0.3.0 
│ │   ├─┬ lazystream@1.0.0 
│ │   │ └─┬ readable-stream@2.3.6 
│ │   │   ├── inherits@2.0.3 
│ │   │   ├── isarray@1.0.0 
│ │   │   └── string_decoder@1.1.1 
│ │   ├── lodash.isequal@4.5.0 
│ │   ├─┬ merge-stream@1.0.1 
│ │   │ └─┬ readable-stream@2.3.6 
│ │   │   ├── inherits@2.0.3 
│ │   │   ├── isarray@1.0.0 
│ │   │   └── string_decoder@1.1.1 
│ │   ├── object-assign@4.1.1 
│ │   ├── strip-bom@2.0.0 
│ │   ├── strip-bom-stream@1.0.0 
│ │   ├── through2-filter@2.0.0 
│ │   ├── vali-date@1.0.0 
│ │   └── vinyl@1.2.0 
│ ├── object-assign@4.1.1 
│ ├─┬ plur@2.1.2 
│ │ └── irregular-plurals@1.4.0 
│ ├─┬ pretty-bytes@2.0.1 
│ │ ├── get-stdin@4.0.1 
│ │ ├─┬ meow@3.7.0 
│ │ │ ├─┬ camelcase-keys@2.1.0 
│ │ │ │ └── camelcase@2.1.1 
│ │ │ ├─┬ loud-rejection@1.6.0 
│ │ │ │ ├─┬ currently-unhandled@0.4.1 
│ │ │ │ │ └── array-find-index@1.0.2 
│ │ │ │ └── signal-exit@3.0.2 
│ │ │ ├── map-obj@1.0.1 
│ │ │ ├─┬ normalize-package-data@2.4.0 
│ │ │ │ ├── hosted-git-info@2.6.1 
│ │ │ │ ├─┬ is-builtin-module@1.0.0 
│ │ │ │ │ └── builtin-modules@1.1.1 
│ │ │ │ └─┬ validate-npm-package-license@3.0.3 
│ │ │ │   ├─┬ spdx-correct@3.0.0 
│ │ │ │   │ └── spdx-license-ids@3.0.0 
│ │ │ │   └─┬ spdx-expression-parse@3.0.0 
│ │ │ │     └── spdx-exceptions@2.1.0 
│ │ │ ├── object-assign@4.1.1 
│ │ │ ├─┬ redent@1.0.0 
│ │ │ │ ├─┬ indent-string@2.1.0 
│ │ │ │ │ └── repeating@2.0.1 
│ │ │ │ └── strip-indent@1.0.1 
│ │ │ └── trim-newlines@1.0.0 
│ │ └── number-is-nan@1.0.1 
│ └── through2-concurrent@1.1.1 
├─┬ gulp-plumber@0.6.6 
│ └─┬ through2@0.6.5
│   └─┬ readable-stream@1.0.34
│     └── inherits@2.0.3 
├─┬ gulp-stylus@1.3.7 
│ ├─┬ accord@0.12.0 
│ │ ├─┬ fobject@0.0.4 
│ │ │ ├── graceful-fs@4.1.11 
│ │ │ └── semver@5.5.0 
│ │ ├─┬ glob@4.5.3 
│ │ │ └── inherits@2.0.3 
│ │ ├── indx@0.2.3 
│ │ ├── resolve@0.7.4 
│ │ ├─┬ uglify-js@2.8.29 
│ │ │ ├── source-map@0.5.7 
│ │ │ └─┬ yargs@3.10.0 
│ │ │   └── cliui@2.1.0 
│ │ └── when@3.7.8 
│ └─┬ through2@0.6.5 
│   └─┬ readable-stream@1.0.34 
│     └── inherits@2.0.3 
├─┬ gulp-uglify@1.5.4 
│ ├── deap@1.0.1 
│ ├─┬ isobject@2.1.0 
│ │ └── isarray@1.0.0 
│ ├─┬ uglify-js@2.6.4 
│ │ ├── async@0.2.10 
│ │ ├── source-map@0.5.7 
│ │ ├── uglify-to-browserify@1.0.2 
│ │ └─┬ yargs@3.10.0 
│ │   ├── camelcase@1.2.1 
│ │   ├─┬ cliui@2.1.0 
│ │   │ ├─┬ center-align@0.1.3 
│ │   │ │ ├─┬ align-text@0.1.4 
│ │   │ │ │ ├── kind-of@3.2.2 
│ │   │ │ │ └── longest@1.0.1 
│ │   │ │ └── lazy-cache@1.0.4 
│ │   │ ├── right-align@0.1.3 
│ │   │ └── wordwrap@0.0.2 
│ │   └── window-size@0.1.0 
│ ├── uglify-save-license@0.4.1 
│ └─┬ vinyl-sourcemaps-apply@0.2.1 
│   └── source-map@0.5.7 
├── jeet@6.1.2 
├─┬ kouto-swiss@0.11.14 
│ ├─┬ prefiks@0.3.3 
│ │ └── semver@4.3.6 
│ └─┬ stylus@0.53.0 
│   ├─┬ glob@3.2.11 
│   │ ├── inherits@2.0.3 
│   │ └── minimatch@0.3.0 
│   └── sax@0.5.8 
├── rupture@0.6.2 
└─┬ stylus@0.54.5 
  ├── css-parse@1.7.0 
  ├─┬ glob@7.0.6 
  │ ├── fs.realpath@1.0.0 
  │ ├── inherits@2.0.3 
  │ ├── minimatch@3.0.4 
  │ └── path-is-absolute@1.0.1 
  ├── sax@0.5.8 
  └─┬ source-map@0.1.43 
    └── amdefine@1.0.1 

npm WARN cardsjekyll@1.0.0 No repository field.
[vagrant@h105 biscuits]$ 
[vagrant@h105 biscuits]$ 
[vagrant@h105 biscuits]$ gulp -f gulpfile.js [00:34:05] Using gulpfile /vagrant/git/biscuits/gulpfile.js
[00:34:05] Starting 'js'...
[00:34:05] Starting 'stylus'...
[00:34:05] Finished 'stylus' after 6.53 ms
[00:34:05] Starting 'watch'...
[00:34:09] Finished 'watch' after 4.73 s
...
...
...
~~~

并且在这个过程中顺便解决一些插件的依赖

---

# 总结

整个过程行云流水，一气呵成(因为我已经做过五次类似搭建了^_^)

* TOC
{:toc}

---

[jekyll]:https://jekyllrb.com/