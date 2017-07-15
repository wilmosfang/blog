---
layout:  post
title:  安装 jekyll
author:  wilmosfang
tags:   jekyll gcc rvm ruby nodejs
categories:  jekyll
wc: 1203 6117 92532
excerpt:  安装新版本 jekyll
comments: true
---


# 前言

很早以前我写过一篇安装 **[jekyll][jekyll]** 的文章，这次作为一个简单的梳理，再对安装过程作一个更新


> **Tip:** 
> 当前最近版本为
> 
> **ruby 2.4.0**  
>
> **gem 2.6.12**
>
> **rvm 1.29.2** 
>
> **jekyll 3.5.0** 
>
> **nodejs v6.10.3**


---

# 概要

* TOC
{:toc}



---


## 系统环境


~~~
[root@much ~]# hostnamectl 
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 99395058b7b94c72a1e753b553a3d690
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# uname -a 
Linux much 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@much ~]# 
~~~

---

## 安装 GCC

~~~
[root@much ~]# yum install gcc 
Loaded plugins: fastestmirror, langpacks
base                                                                                                                                                                 | 3.6
 kB  00:00:00     
c7-media                                                                                                                                                             | 3.6
 kB  00:00:00     
extras                                                                                                                                                               | 3.4
 kB  00:00:00     
updates                                                                                                                                                              | 3.4
 kB  00:00:00     
(2/2): updates/7/x86_64/primary_db                                              2% [=-                                                                    ]  0.0 B/s | 212
(1/2): extras/7/x86_64/primary_db                                                                                                                                    | 190
 kB  00:00:00     
(2/2): updates/7/x86_64/primary_db                                              8% [======                                                                ] 632 kB/s | 726
(2/2): updates/7/x86_64/primary_db                                              12% [========-                                                            ] 657 kB/s | 1.0
...
...
(2/2): updates/7/x86_64/primary_db                                              88% [=============================================================        ] 1.2 MB/s | 7.0
(2/2): updates/7/x86_64/primary_db                                              98% [==================================================================== ] 1.2 MB/s | 7.8
(2/2): updates/7/x86_64/primary_db                                                                                                                                   | 7.7
 MB  00:00:05     
Determining fastest mirrors
 * base: centos.ustc.edu.cn
 * c7-media: 
 * extras: mirror.bit.edu.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.8.5-11.el7 will be installed
--> Processing Dependency: cpp = 4.8.5-11.el7 for package: gcc-4.8.5-11.el7.x86_64
--> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-4.8.5-11.el7.x86_64
--> Running transaction check
---> Package cpp.x86_64 0:4.8.5-11.el7 will be installed
---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================================================
==================
 Package                                     Arch                                        Version                                            Repository                    
             Size
==========================================================================================================================================================================
==================
Installing:
 gcc                                         x86_64                                      4.8.5-11.el7                                       base                          
             16 M
Installing for dependencies:
 cpp                                         x86_64                                      4.8.5-11.el7                                       base                          
            5.9 M
 libmpc                                      x86_64                                      1.0.1-3.el7                                        base                          
             51 k

Transaction Summary
==========================================================================================================================================================================
==================
Install  1 Package (+2 Dependent packages)

Total download size: 22 M
Installed size: 52 M
Is this ok [y/d/N]: y
Downloading packages:
(2/3): gcc-4.8.5-11.el7.x86_64.rpm                                              9% [======-                                                               ]  0.0 B/s | 2.2
(1/3): libmpc-1.0.1-3.el7.x86_64.rpm                                                                                                                                 |  51
 kB  00:00:00     
(3/3): gcc-4.8.5-11.el7.x86_64.rpm                                              24% [=================                                                    ] 4.4 MB/s | 5.5
(2/3): cpp-4.8.5-11.el7.x86_64.rpm                                                                                                                                   | 5.9
 MB  00:00:00     
(3/3): gcc-4.8.5-11.el7.x86_64.rpm                                              60% [=========================================-                           ] 5.3 MB/s |  13
(3/3): gcc-4.8.5-11.el7.x86_64.rpm                                              75% [====================================================                 ] 5.7 MB/s |  17
(3/3): gcc-4.8.5-11.el7.x86_64.rpm                                              91% [===============================================================      ] 6.0 MB/s |  20
(3/3): gcc-4.8.5-11.el7.x86_64.rpm                                                                                                                                   |  16
 MB  00:00:02     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------
Total                                                                                                                                                       9.5 MB/s |  22
 MB  00:00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libmpc-1.0.1-3.el7.x86_64 [                                                                                                                                
  Installing : libmpc-1.0.1-3.el7.x86_64 [###############################################################################                                                 
  Installing : libmpc-1.0.1-3.el7.x86_64 [#########################################################################################################################       
  Installing : libmpc-1.0.1-3.el7.x86_64 [################################################################################################################################
  Installing : libmpc-1.0.1-3.el7.x86_64 [################################################################################################################################
  Installing : libmpc-1.0.1-3.el7.x86_64                                                                                                                                  
              1/3 
  Installing : cpp-4.8.5-11.el7.x86_64 [                                                                                                                                  
  Installing : cpp-4.8.5-11.el7.x86_64 [#                                                                                                                                 
  Installing : cpp-4.8.5-11.el7.x86_64 [##                                                                                                                                
  Installing : cpp-4.8.5-11.el7.x86_64 [####                                                                                                                              
  Installing : cpp-4.8.5-11.el7.x86_64 [#####                                                                                                                             
 ...
 ...
  Installing : cpp-4.8.5-11.el7.x86_64 [##################################################################################################################################
  Installing : cpp-4.8.5-11.el7.x86_64                                                                                                                                    
              2/3 
  Installing : gcc-4.8.5-11.el7.x86_64 [                                                                                                                                  
  Installing : gcc-4.8.5-11.el7.x86_64 [#                                                                                                                                 
  Installing : gcc-4.8.5-11.el7.x86_64 [##                                                                                                                                
 ...
 ...
  Installing : gcc-4.8.5-11.el7.x86_64 [##################################################################################################################################
  Installing : gcc-4.8.5-11.el7.x86_64 [##################################################################################################################################
  Installing : gcc-4.8.5-11.el7.x86_64                                                                                                                                    
              3/3 
  Verifying  : gcc-4.8.5-11.el7.x86_64                                                                                                                                    
              1/3 
  Verifying  : cpp-4.8.5-11.el7.x86_64                                                                                                                                    
              2/3 
  Verifying  : libmpc-1.0.1-3.el7.x86_64                                                                                                                                  
              3/3 

Installed:
  gcc.x86_64 0:4.8.5-11.el7                                                                                                                                               
                  

Dependency Installed:
  cpp.x86_64 0:4.8.5-11.el7                                                                   libmpc.x86_64 0:1.0.1-3.el7                                                 
                 

Complete!
[root@much ~]# 
~~~


---

## 安装 RVM

~~~
[root@much ~]# rvm -v 
bash: rvm: command not found...
[root@much ~]# curl -L get.rvm.io | bash -s stable  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0     29      0  0:00:06  0:00:06 --:--:--    47
100 24028  100 24028    0     0   3203      0  0:00:07  0:00:07 --:--:-- 35026
Downloading https://github.com/rvm/rvm/archive/1.29.2.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.2/1.29.2.tar.gz.asc
gpg: Signature made 2017年06月23日 星期五 00时18分38秒 CST using RSA key ID BF04FF17
gpg: Can't check signature: No public key
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found. Assuming you trust Michal Papis import the mpapis public key (do
wnloading the signatures).

GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.29.2.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.2/1.29.2.tar.gz.asc'! Try to install G
PG v2 and then fetch the public key:

    gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

the key can be compared with:

    https://rvm.io/mpapis.asc
    https://keybase.io/mpapis

NOTE: GPG version 2.1.17 have a bug which cause failures during fetching keys from remote server. Please downgrade or upgrade to newer version (if available) or use the s
econd method described above.

[root@much ~]# gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: requesting key D39DC0E3 from hkp server keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[root@much ~]# echo $?
0
[root@much ~]# curl -L get.rvm.io | bash -s stable  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    127      0  0:00:01  0:00:01 --:--:--   127
100 24028  100 24028    0     0  10648      0  0:00:02  0:00:02 --:--:-- 46656
Downloading https://github.com/rvm/rvm/archive/1.29.2.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.2/1.29.2.tar.gz.asc
gpg: Signature made 2017年06月23日 星期五 00时18分38秒 CST using RSA key ID BF04FF17
gpg: Good signature from "Michal Papis (RVM signing) <mpapis@gmail.com>"
gpg:                 aka "Michal Papis <michal.papis@toptal.com>"
gpg:                 aka "[jpeg image of size 5015]"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
     Subkey fingerprint: 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/rvm/archives/rvm-1.29.2.tgz'
Creating group 'rvm'

Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open shell windows, in rare cases you need to reopen all shell windows.

# Administrator,
#
#   Thank you for using RVM!
#   We sincerely hope that RVM helps to make your life easier and more enjoyable!!!
#
# ~Wayne, Michal & team.

In case of problems: https://rvm.io/help and https://twitter.com/rvm_io
[root@much ~]# echo $?
0
[root@much ~]# rvm -v 
bash: rvm: command not found...
[root@much ~]# su - 
Last login: 六 7月 15 23:49:55 CST 2017 from 192.168.56.1 on pts/1
Last failed login: 六 7月 15 23:56:12 CST 2017 from 192.168.56.1 on ssh:notty
There were 16 failed login attempts since the last successful login.
[root@much ~]# rvm -v 
rvm 1.29.2 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io/]
[root@much ~]# 
~~~



---

## 安装 Ruby 与 gem

~~~
[root@much ~]# rvm list known 
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.6]
[ruby-]2.3[.3]
[ruby-]2.4[.0]
ruby-head

# for forks use: rvm install ruby-head-<name> --url https://github.com/github/ruby.git --branch 2.2

# JRuby
jruby-1.6[.8]
jruby-1.7[.26]
jruby[-9.1.7.0]
jruby-head

# Rubinius
rbx-1[.4.3]
rbx-2.3[.0]
rbx-2.4[.1]
rbx-2[.5.8]
rbx[-3.71]
rbx-head

# Opal
opal

# Minimalistic ruby implementation - ISO 30170:2012
mruby-1.0.0
mruby-1.1.0
mruby-1[.2.0]
mruby[-head]

# Ruby Enterprise Edition
ree-1.8.6
ree[-1.8.7][-2012.02]

# Topaz
topaz

# MagLev
maglev[-head]
maglev-1.0.0

# Mac OS X Snow Leopard Or Newer
macruby-0.10
macruby-0.11
macruby[-0.12]
macruby-nightly
macruby-head

# IronRuby
ironruby[-1.1.3]
ironruby-head
[root@much ~]# rvm  install ruby
Searching for binary rubies, this might take some time.
No binary rubies available for: centos/7/x86_64/ruby-2.4.0.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: patch, libyaml-devel, autoconf, gcc-c++, patch, readline-devel, zlib-devel, libffi-devel, openssl-devel, automake, libtool, bison, sqlit
e-devel.................
Requirements installation successful.
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-2.4.0, this may take a while depending on your cpu(s)...
ruby-2.4.0 - #downloading ruby-2.4.0, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  1 11.9M    1  224k    0     0  30734      0  0:06:58  0:00:06  0:06:52 39102
  4 11
  5 11.9M    5  699k    0     0  29265      0  0:07:03  0:00:23  0:06:40 30500
  7 11.9M    7  925k    0     0  29212      0  0:07:10  0:00:32  0:06:38 33923
  9 11.9M    9 1142k    0     0  28061      0  0:07:28  0:00:40  0:06:47 28575
 11 11.9M   11 1403k    0     0  28405      0  0:07:22  0:00:49  0:06:33 30751
 13 11.9M   13 1609k    0     0  28185      0  0:07:26  0:00:58  0:06:30 22647
 14 11.9M   14 1807k    0     0  27488      0  0:07:37  0:01:06  0:06:31 17855
 16 11.9M   16 1970k    0     0  26646      0  0:07:51  0:01:15  0:06:34 18848
 18 11.9M   18 2277k    0     0  27650      0  0:07:34  0:01:23  0:06:11 36714
 20 11.9M   20 2469k    0     0  27347      0  0:07:39  0:01:32  0:06:07 29504
 21 11.9M   21 2669k    0     0  26921      0  0:07:48  0:01:40  0:06:08 29006
 24 1
 26 11.9M   26 3306k    0     0  28573      0  0:07:20  0:01:58  0:05:22 42975
 28 11.9M   28 3488k    0     0  28208      0  0:07:25  0:02:06  0:05:19 26391
 30 11.9M   30 3720k    0     0  28121      0  0:07:27  0:02:14  0:05:15 20133
 32 11.9M   32 4038k    0     0  28552      0  0:07:20  0:02:23  0:04:57 28038
 35 11.9M   35 4302k    0     0  28851      0  0:07:15  0:02:31  0:04:45 30350
 36 11.9M   36 4515k    0     0  28716      0  0:07:17  0:02:40  0:04:37 30581
 38 11.9M   38 4676k    0     0  28255      0  0:07:24  0:02:49  0:04:35 21476
 39 11.9M   39 4905k    0     0  28214      0  0:07:25  0:02:57  0:04:28 26739
 40 11.9M   40 5028k    0     0  27531      0  0:07:36  0:03:07  0:04:29  8127
 42 11.9M   42 5188k    0     0  27164      0  0:07:41  0:03:14  0:04:27 19458
 43 
 44 11.9M   44 5506k    0     0  26542      0  0:07:53  0:03:31  0:04:22 13722
 47 11.9M   47 5773k    0     0  26810      0  0:07:48  0:03:40  0:04:08 34601
 49 11.9M   49 6085k    0     0  27144      0  0:07:43  0:03:48  0:03:56 40461
 52 11.9M   52 6447k    0     0  27706      0  0:07:33  0:03:57  0:03:36 42853
 54 11.9M   54 6683k    0     0  27750      0  0:07:33  0:04:05  0:03:28 27443
 56 11.9M   56 6880k    0     0  27541      0  0:07:36  0:04:14  0:03:22 20785
 58 11.9M   58 7166k    0     0  27832      0  0:07:31  0:04:23  0:03:10 41462
 61 11.9M   61 7529k    0     0  28276      0  0:07:24  0:04:31  0:02:53 52316
 62 11.9M   62 7726k    0     0  28208      0  0:07:25  0:04:40  0:02:45 33770
 65 11.9M   65 8009k    0     0  28313      0  0:07:24  0:04:48  0:02:36 32529
 66
 68 11.9M   68 8357k    0     0  27922      0  0:07:30  0:05:05  0:02:25 25904
 69 11.9M   69 8584k    0     0  27947      0  0:07:29  0:05:14  0:02:15 30804
 71 11.9M   71 8730k    0     0  27640      0  0:07:34  0:05:22  0:02:12 18392
 73 11.9M   73 9058k    0     0  27671      0  0:07:34  0:05:31  0:02:03 24845
 74 11.9M   74 9166k    0     0  27562      0  0:07:36  0:05:39  0:01:57 17958
 76 11.9M   76 9422k    0     0  27632      0  0:07:34  0:05:48  0:01:46 31347
 78 11.9M   78 9577k    0     0  27431      0  0:07:38  0:05:57  0:01:41 19397
 79 11.9M   79 9759k    0     0  27287      0  0:07:40  0:06:05  0:01:35 21294
 81 11.9M   81 9951k    0     0  27203      0  0:07:42  0:06:14  0:01:28 20593
 83 11.9M   83 10.0M    0     0  27458      0  0:07:37  0:06:22  0:01:15 33667
 8
 87 11.9M   87 10.4M    0     0  27336      0  0:07:40  0:06:39  0:01:01 20614
 89 11.9M   88 10.6M    0     0  27371      0  0:07:39  0:06:48  0:00:51 33213
 91 11.9M   91 10.9M    0     0  27587      0  0:07:35  0:06:56  0:00:39 39160
 93 11.9M   93 11.1M    0     0  27458      0  0:07:37  0:07:05  0:00:32 20381
 94 11.9M   94 11.3M    0     0  27429      0  0:07:38  0:07:13  0:00:25 21191
 96 11.9M   96 11.6M    0     0  27462      0  0:07:37  0:07:22  0:00:15 27681
 98 11.9M   98 11.7M    0     0  27398      0  0:07:38  0:07:31  0:00:09 21184
100 11.9M  100 11.9M    0     0  27595      0  0:07:35  0:07:35 --:--:-- 49088
ruby-2.4.0 - #extracting ruby-2.4.0 to /usr/local/rvm/src/ruby-2.4.0....
ruby-2.4.0 - #configuring.................................|
.................................
ruby-2.4.0 - #post-configuration..
ruby-2.4.0 - #compiling................................./
..................................../
............
ruby-2.4.0 - #installing..........................
ruby-2.4.0 - #making binaries executable..
ruby-2.4.0 - #downloading rubygems-2.6.12
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  749k    0  4491    0     0    571      0  0:50:47  0:00:06  0:50:41   378
  8  7
 11  749k   11 91929    0     0   3666      0  0:03:25  0:00:23  0:03:02  1653
 14  749k   14  105k    0     0   3251      0  0:03:56  0:00:33  0:03:23  1197
 18  749k   18  140k    0     0   3420      0  0:03:44  0:00:40  0:03:06  5035
 24  749k   24  180k    0     0   3650      0  0:03:30  0:00:49  0:02:41  3208
 27  749k   27  204k    0     0   3564      0  0:03:35  0:00:58  0:02:37  3265
 31  749k   31  239k    0     0   3399      0  0:03:45  0:01:07  0:02:38  2010
 37  749k   37  282k    0     0   3803      0  0:03:21  0:01:15  0:02:06  4811
 40  749k   40  303k    0     0   3706      0  0:03:27  0:01:23  0:02:04  3494
 45  749k   45  338k    0     0   3722      0  0:03:26  0:01:33  0:01:53  6922
 46  749k   46  351k    0     0   3513      0  0:03:36  0:01:41  0:01:55   950
 50  
 51  749k   51  386k    0     0   3329      0  0:03:49  0:01:57  0:01:52   597
 57  749k   56  426k    0     0   3443      0  0:03:43  0:02:06  0:01:37  4467
 62  749k   62  466k    0     0   3517      0  0:03:38  0:02:15  0:01:22  5225
 67  749k   67  509k    0     0   3586      0  0:03:34  0:02:23  0:01:11  2817
 72  749k   72  546k    0     0   3661      0  0:03:29  0:02:31  0:00:58  6524
 75  749k   75  562k    0     0   3479      0  0:03:40  0:02:40  0:01:00     0
 77  749k   77  581k    0     0   3503      0  0:03:39  0:02:49  0:00:50  2731
 80  749k   80  602k    0     0   3470      0  0:03:41  0:02:57  0:00:44  2979
 82  749k   82  618k    0     0   3392      0  0:03:46  0:03:06  0:00:40  2695
 86  749k   86  645k    0     0   3372      0  0:03:46  0:03:15  0:00:31  3617
 87 
 90  749k   90  677k    0     0   3259      0  0:03:55  0:03:31  0:00:24  1308
 93  749k   92  696k    0     0   3228      0  0:03:57  0:03:40  0:00:17  2147
 96  749k   96  723k    0     0   3219      0  0:03:58  0:03:48  0:00:10  3272
 99  749k   99  741k    0     0   3181      0  0:04:01  0:03:58  0:00:03  2755
100  749k  100  749k    0     0   3187      0  0:04:00  0:04:00 --:--:--  2112
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.4.0 - #extracting rubygems-2.6.12....
ruby-2.4.0 - #removing old rubygems.........
ruby-2.4.0 - #installing rubygems-2.6.12.........................
ruby-2.4.0 - #gemset created /usr/local/rvm/gems/ruby-2.4.0@global
ruby-2.4.0 - #importing gemset /usr/local/rvm/gemsets/global.gems...............................\
................
ruby-2.4.0 - #generating global wrappers........
ruby-2.4.0 - #gemset created /usr/local/rvm/gems/ruby-2.4.0
ruby-2.4.0 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.4.0 - #generating default wrappers........
ruby-2.4.0 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.4.0 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
[root@much ~]# echo $?
0
[root@much ~]# gem -v
2.6.12
[root@much ~]# 
~~~

---

## 更新源

~~~
[root@much ~]# gem source -l
*** CURRENT SOURCES ***

https://rubygems.org/
[root@much ~]# gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/ 
https://gems.ruby-china.org/ added to sources
https://rubygems.org/ removed from sources
[root@much ~]# gem source -l
*** CURRENT SOURCES ***

https://gems.ruby-china.org/
[root@much ~]# 
~~~

> **Note:**  由于众所周知的原因，在我们伟大的 GrateWall 介入之下 ，如果不更新源，安装 gem 将会是一种折磨

---

## 安装jekyll

~~~
[root@much ~]# gem install jekyll
Fetching: safe_yaml-1.0.4.gem (100%)
Successfully installed safe_yaml-1.0.4
Fetching: rouge-1.11.1.gem (100%)
Successfully installed rouge-1.11.1
Fetching: forwardable-extended-2.6.0.gem (100%)
Successfully installed forwardable-extended-2.6.0
Fetching: pathutil-0.14.0.gem (100%)
Successfully installed pathutil-0.14.0
Fetching: mercenary-0.3.6.gem (100%)
Successfully installed mercenary-0.3.6
Fetching: liquid-4.0.0.gem (100%)
Successfully installed liquid-4.0.0
Fetching: kramdown-1.14.0.gem (100%)
Successfully installed kramdown-1.14.0
Fetching: ffi-1.9.18.gem ( 37%)
Fetching: ffi-1.9.18.gem ( 76%)
Fetching: ffi-1.9.18.gem (100%)
Building native extensions.  This could take a while...
Successfully installed ffi-1.9.18
Fetching: rb-inotify-0.9.10.gem (100%)
Successfully installed rb-inotify-0.9.10
Fetching: rb-fsevent-0.10.2.gem (100%)
Successfully installed rb-fsevent-0.10.2
Fetching: listen-3.0.8.gem (100%)
Successfully installed listen-3.0.8
Fetching: jekyll-watch-1.5.0.gem (100%)
Successfully installed jekyll-watch-1.5.0
Fetching: sass-listen-4.0.0.gem (100%)
Successfully installed sass-listen-4.0.0
Fetching: sass-3.5.1.gem (100%)
sass-3.5.1.gem (100%)
Successfully installed sass-3.5.1
Fetching: jekyll-sass-converter-1.5.0.gem (100%)
Successfully installed jekyll-sass-converter-1.5.0
Fetching: colorator-1.1.0.gem (100%)
Successfully installed colorator-1.1.0
Fetching: public_suffix-2.0.5.gem (100%)
Successfully installed public_suffix-2.0.5
Fetching: addressable-2.5.1.gem (100%)
Successfully installed addressable-2.5.1
Fetching: jekyll-3.5.0.gem (100%)
Successfully installed jekyll-3.5.0
Parsing documentation for safe_yaml-1.0.4
Installing ri documentation for safe_yaml-1.0.4
Parsing documentation for rouge-1.11.1
Installing ri documentation for rouge-1.11.1
Parsing documentation for forwardable-extended-2.6.0
Installing ri documentation for forwardable-extended-2.6.0
Parsing documentation for pathutil-0.14.0
Installing ri documentation for pathutil-0.14.0
Parsing documentation for mercenary-0.3.6
Installing ri documentation for mercenary-0.3.6
Parsing documentation for liquid-4.0.0
Installing ri documentation for liquid-4.0.0
Parsing documentation for kramdown-1.14.0
Installing ri documentation for kramdown-1.14.0
Parsing documentation for ffi-1.9.18
Installing ri documentation for ffi-1.9.18
Parsing documentation for rb-inotify-0.9.10
Installing ri documentation for rb-inotify-0.9.10
Parsing documentation for rb-fsevent-0.10.2
Installing ri documentation for rb-fsevent-0.10.2
Parsing documentation for listen-3.0.8
Installing ri documentation for listen-3.0.8
Parsing documentation for jekyll-watch-1.5.0
Installing ri documentation for jekyll-watch-1.5.0
Parsing documentation for sass-listen-4.0.0
Installing ri documentation for sass-listen-4.0.0
Parsing documentation for sass-3.5.1
Installing ri documentation for sass-3.5.1
Parsing documentation for jekyll-sass-converter-1.5.0
Installing ri documentation for jekyll-sass-converter-1.5.0
Parsing documentation for colorator-1.1.0
Installing ri documentation for colorator-1.1.0
Parsing documentation for public_suffix-2.0.5
Installing ri documentation for public_suffix-2.0.5
Parsing documentation for addressable-2.5.1
Installing ri documentation for addressable-2.5.1
Parsing documentation for jekyll-3.5.0
Installing ri documentation for jekyll-3.5.0
Done installing documentation for safe_yaml, rouge, forwardable-extended, pathutil, mercenary, liquid, kramdown, ffi, rb-inotify, rb-fsevent, listen, jekyll-watch, sass-l
isten, sass, jekyll-sass-converter, colorator, public_suffix, addressable, jekyll after 18 seconds
19 gems installed
[root@much ~]# echo $?
0
[root@much ~]# jekyll  --help 
jekyll 3.5.0 -- Jekyll is a blog-aware, static site generator in Ruby

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
[root@much ~]# jekyll  -v
jekyll 3.5.0
[root@much ~]# gem -v 
2.6.12
[root@much ~]# 
~~~

---


## 安装nodejs

~~~
[root@much ~]# yum install epel-release 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * c7-media: 
 * extras: mirror.bit.edu.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-9 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================================================
==================
 Package                                           Arch                                        Version                                    Repository                      
             Size
==========================================================================================================================================================================
==================
Installing:
 epel-release                                      noarch                                      7-9                                        extras                          
             14 k

Transaction Summary
==========================================================================================================================================================================
==================
Install  1 Package

Total download size: 14 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-9.noarch.rpm                                                                                                                                          |  14
 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-9.noarch [                                                                                                                                  
  Installing : epel-release-7-9.noarch [########                                                                                                                          
  Installing : epel-release-7-9.noarch [###############                                                                                                                   
  Installing : epel-release-7-9.noarch [#####################                                                                                                             
  Installing : epel-release-7-9.noarch [######################################                                                                                            
  Installing : epel-release-7-9.noarch [##################################################################################################################################
  Installing : epel-release-7-9.noarch                                                                                                                                    
              1/1 
  Verifying  : epel-release-7-9.noarch                                                                                                                                    
              1/1 

Installed:
  epel-release.noarch 0:7-9                                                                                                                                               
                  

Complete!
[root@much ~]# yum -y install nodejs 
Loaded plugins: fastestmirror, langpacks
epel/x86_64/metalink                                                                                                                                                 | 4.8
 kB  00:00:00     
epel                                                                                                                                                                 | 4.3
 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                   0% [                                                                      ]  0.0 B/s |    
(1/3): epel/x86_64/group_gz                                                                                                                                          | 170
 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                                                                                                        | 793
 kB  00:00:00     
(3/3): epel/x86_64/primary_db                                                   16% [===========                                                          ]  0.0 B/s | 965
(3/3): epel/x86_64/primary_db                                                   16% [===========                                                          ] 3.0 kB/s | 969
...
...
(3/3): epel/x86_64/primary_db                                                   99% [====================================================================-]  44 kB/s | 5.7
(3/3): epel/x86_64/primary_db                                                                                                                                        | 4.8
 MB  00:01:45     
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * c7-media: 
 * epel: mirrors.neusoft.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package nodejs.x86_64 1:6.10.3-1.el7 will be installed
--> Processing Dependency: npm = 1:3.10.10-1.6.10.3.1.el7 for package: 1:nodejs-6.10.3-1.el7.x86_64
--> Processing Dependency: libuv >= 1:1.9.1 for package: 1:nodejs-6.10.3-1.el7.x86_64
--> Processing Dependency: libuv.so.1()(64bit) for package: 1:nodejs-6.10.3-1.el7.x86_64
--> Processing Dependency: libhttp_parser.so.2()(64bit) for package: 1:nodejs-6.10.3-1.el7.x86_64
--> Running transaction check
---> Package http-parser.x86_64 0:2.7.1-3.el7 will be installed
---> Package libuv.x86_64 1:1.10.2-1.el7 will be installed
---> Package npm.x86_64 1:3.10.10-1.6.10.3.1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================================================
==================
 Package                                      Arch                                    Version                                                   Repository                
             Size
==========================================================================================================================================================================
==================
Installing:
 nodejs                                       x86_64                                  1:6.10.3-1.el7                                            epel                      
            4.6 M
Installing for dependencies:
 http-parser                                  x86_64                                  2.7.1-3.el7                                               epel                      
             30 k
 libuv                                        x86_64                                  1:1.10.2-1.el7                                            epel                      
            109 k
 npm                                          x86_64                                  1:3.10.10-1.6.10.3.1.el7                                  epel                      
            2.5 M

Transaction Summary
==========================================================================================================================================================================
==================
Install  1 Package (+3 Dependent packages)

Total download size: 7.3 M
Installed size: 26 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/epel/packages/http-parser-2.7.1-3.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for http-parser-2.7.1-3.el7.x86_64.rpm is not installed
(1/4): http-parser-2.7.1-3.el7.x86_64.rpm                                                                                                                            |  30
 kB  00:00:00     
(3/4): nodejs-6.10.3-1.el7.x86_64.rpm                                           0% [                                                                      ]  0.0 B/s |  30
(2/4): libuv-1.10.2-1.el7.x86_64.rpm                                                                                                                                 | 109
 kB  00:00:00     
(4/4): npm-3.10.10-1.6.10.3.1.el7.x86_64.rpm                                    2% [=-                                                                    ] 193 kB/s | 192
(3/4): nodejs-6.10.3-1.el7.x86_64.rpm                                           2% [=-                                                                    ] 177 kB/s | 212
(3/4): nodejs-6.10.3-1.el7.x86_64.rpm                                           3% [==                                                                    ] 159 kB/s | 226
(3/4): nodejs-6.10.3-1.el7.x86_64.rpm                                           3% [==                                                                    ] 150 kB/s | 247
...
...
(4/4): npm-3.10.10-1.6.10.3.1.el7.x86_64.rpm                                    99% [====================================================================-] 9.8 kB/s | 7.3
(4/4): npm-3.10.10-1.6.10.3.1.el7.x86_64.rpm                                    99% [====================================================================-]  10 kB/s | 7.3
(4/4): npm-3.10.10-1.6.10.3.1.el7.x86_64.rpm                                                                                                                         | 2.5
 MB  00:02:48     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------
Total                                                                                                                                                        44 kB/s | 7.3
 MB  00:02:48     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : http-parser-2.7.1-3.el7.x86_64 [                                                                                                                           
  Installing : http-parser-2.7.1-3.el7.x86_64 [####################################################                                                                       
  ...
  ...
  Installing : http-parser-2.7.1-3.el7.x86_64 [###########################################################################################################################
  Installing : http-parser-2.7.1-3.el7.x86_64                                                                                                                             
              1/4 
  Installing : 1:libuv-1.10.2-1.el7.x86_64 [                                                                                                                              
  Installing : 1:libuv-1.10.2-1.el7.x86_64 [###############################                                                                                               
  Installing : 1:libuv-1.10.2-1.el7.x86_64 [###############################################################                                                               
...
...
  Installing : 1:libuv-1.10.2-1.el7.x86_64 [##############################################################################################################################
  Installing : 1:libuv-1.10.2-1.el7.x86_64 [##############################################################################################################################
  Installing : 1:libuv-1.10.2-1.el7.x86_64                                                                                                                                
              2/4 
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [                                                                                                                             
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [#                                                                                                                            
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [##                                                                                                                           
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [####                                                                                                                         
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [#####                                                                                                                        
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [######                                                                                                                       
...
...
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [#############################################################################################################################
  Installing : 1:nodejs-6.10.3-1.el7.x86_64 [#############################################################################################################################
  Installing : 1:nodejs-6.10.3-1.el7.x86_64                                                                                                                               
              3/4 
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64 [                                                                                                                      
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64 [#                                                                                                                     
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64 [##                                                                                                                    
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64 [### 
  ...
  ...
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64 [######################################################################################################################
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64 [######################################################################################################################
  Installing : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64                                                                                                                        
              4/4 
  Verifying  : 1:npm-3.10.10-1.6.10.3.1.el7.x86_64                                                                                                                        
              1/4 
  Verifying  : 1:libuv-1.10.2-1.el7.x86_64                                                                                                                                
              2/4 
  Verifying  : http-parser-2.7.1-3.el7.x86_64                                                                                                                             
              3/4 
  Verifying  : 1:nodejs-6.10.3-1.el7.x86_64                                                                                                                               
              4/4 

Installed:
  nodejs.x86_64 1:6.10.3-1.el7                                                                                                                                            
                  

Dependency Installed:
  http-parser.x86_64 0:2.7.1-3.el7                               libuv.x86_64 1:1.10.2-1.el7                               npm.x86_64 1:3.10.10-1.6.10.3.1.el7            
                  

Complete!
[root@much ~]# echo $?
0
[root@much ~]# no
node         nohup        nologin      notify-send  
[root@much ~]# node -v 
v6.10.3
[root@much ~]# 
~~~

---

## 报错１

由于我是用老版本的 jekyll 起的博客，现在切换到新版的过程中产生了如下报错

~~~
[bolo@much blog]$ alias runblog 
alias runblog='jekyll server  --host 0.0.0.0 --port 8080'
[bolo@much blog]$
[bolo@much blog]$ runblog
Configuration file: /home/bolo/git/blog/blog/_config.yml
       Deprecation: The 'gems' configuration option has been renamed to 'plugins'. Please update your config file accordingly.
  Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- jekyll-paginate' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/! 
jekyll 3.5.0 | Error:  jekyll-paginate
[bolo@much blog]$ 
~~~

原因是 **gems** 配置参数在新版本中改为了 **plugins** 

解决办法就是将参数更名为如下

~~~
# paginate
#gems: [jekyll-paginate]
plugins: [jekyll-paginate]
~~~


---

## 报错２

在紧接着启动博客的过程中又遇到如下报错


~~~
[bolo@much blog]$ runblog
Configuration file: /home/bolo/git/blog/blog/_config.yml
  Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- jekyll-paginate' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/! 
jekyll 3.5.0 | Error:  jekyll-paginate
[bolo@much blog]$
~~~

原因是缺少 **jekyll-paginate** 的 gem

解决办法是安装这个 gem

~~~
[root@much ~]# gem install  jekyll-paginate 
Fetching: jekyll-paginate-1.1.0.gem (100%)
Successfully installed jekyll-paginate-1.1.0
Parsing documentation for jekyll-paginate-1.1.0
Installing ri documentation for jekyll-paginate-1.1.0
Done installing documentation for jekyll-paginate after 0 seconds
1 gem installed
[root@much ~]# echo $?
0
[root@much ~]#
~~~

再次启动就一切正常

~~~
[bolo@much blog]$ runblog
Configuration file: /home/bolo/git/blog/blog/_config.yml
            Source: /home/bolo/git/blog/blog
       Destination: /home/bolo/git/blog/blog/_site
 Incremental build: disabled. Enable with --incremental
      Generating...    
                      done in 17.078 seconds.
 Auto-regeneration: enabled for '/home/bolo/git/blog/blog'
    Server address: http://0.0.0.0:8080/
  Server running... press ctrl-c to stop.
...
...
...
~~~

博客的详细内容可以参看 **[soft.dog][softdog]**

## 命令汇总

* **`hostnamectl`**
* **`uname -a`**
* **`yum install gcc`**
* **`rvm -v`**
* **`curl -L get.rvm.io | bash -s stable`**
* **`gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`**
* **`curl -L get.rvm.io | bash -s stable`**
* **`rvm -v`**
* **`rvm list known`**
* **`rvm  install ruby`**
* **`gem -v`**
* **`gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/`**
* **`gem source -l`**
* **`gem install jekyll`**
* **`jekyll  --help`**
* **`jekyll  -v`**
* **`yum install epel-release`**
* **`yum -y install nodejs`**
* **`echo $?`**
* **`node -v`**
* **`alias runblog`**
* **`gem install  jekyll-paginate`**
* **`runblog`**

---


[jekyll]:http://jekyll.com.cn/
[softdog]:http://soft.dog/
