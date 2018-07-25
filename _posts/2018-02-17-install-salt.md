---
layout: post
title: "Install Saltstack"
author:  wilmosfang
date: 2018-02-17 09:47:59
image: '/assets/img/'
excerpt: '安装 Saltstack'
main-class: saltstack
color: '#313233'
tags:
 - saltstack
categories:
 - saltstack
twitter_text: 'simple process of Saltstack installation'
introduction: 'installation of Saltstack'
---


## 前言

**[SaltStack][saltstack]** 是一款高性能的自动化运维工具

>SaltStack is a revolutionary approach to infrastructure management that replaces complexity with speed. SaltStack is simple enough to get running in minutes, scalable enough to manage tens of thousands of servers, and fast enough to communicate with each system in seconds

类似的工具还有 **Puppet、Chef、Ansible**，他们之间可以相互替代，但是哪一个更好，我就不在此引发圣战了

这里分享一下 **[SaltStack][saltstack]** 的安装方法

参考 **[INSTALL SALTSTACK][saltstack_install]**

> **Tip:** 当前的版本为 **Latest release: 2017.7.3 (February 5, 2018)**

---

# 操作


## 环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: a0fe156a8a034c1e8e48e5b5e3c42a8c
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0b:e9:0b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 78880sec preferred_lft 78880sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:36:8b:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.209/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~

## 组件与机制

强烈建议先看看 **[SALTSTACK COMPONENTS][saltstack_components]**，可以在脑中构建起 **[SaltStack][saltstack]** 里的各种对象，有助于理解其工作机制

根据需要，再看看 **[Understanding SaltStack][saltstack_tutorial]**，这里基于 **SaltStack** 系统中的的各种组件，清楚地描述了它们间的交互与工作机制

有了这些作为基础，对于 **[SaltStack][saltstack]** 的使用就会清晰和容易很多


## 安装 salt master


在 master 上(192.168.56.209)

~~~
[root@much ~]# curl -L https://bootstrap.saltstack.com -o install_salt.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   264  100   264    0     0    134      0  0:00:01  0:00:01 --:--:--   134
100  244k  100  244k    0     0  68655      0  0:00:03  0:00:03 --:--:--  282k
[root@much ~]# sh install_salt.sh -P -M
 *  INFO: Running version: 2017.12.13
 *  INFO: Executed by: sh
 *  INFO: Command line: 'install_salt.sh -P -M'

 *  INFO: System Information:
 *  INFO:   CPU:          GenuineIntel
 *  INFO:   CPU Arch:     x86_64
 *  INFO:   OS Name:      Linux
 *  INFO:   OS Version:   3.10.0-514.21.1.el7.x86_64
 *  INFO:   Distribution: CentOS 7.3

 *  INFO: Installing minion
 *  INFO: Installing master
 *  INFO: Found function install_centos_stable_deps
 *  INFO: Found function config_salt
 *  INFO: Found function preseed_master
 *  INFO: Found function install_centos_stable
 *  INFO: Found function install_centos_stable_post
 *  INFO: Found function install_centos_restart_daemons
 *  INFO: Found function daemons_running
 *  INFO: Found function install_centos_check_services
 *  INFO: Running install_centos_stable_deps()
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package PyYAML.x86_64 0:3.11-1.el7 will be installed
---> Package chkconfig.x86_64 0:1.7.2-1.el7 will be updated
--> Processing Dependency: chkconfig = 1.7.2-1.el7 for package: ntsysv-1.7.2-1.el7.x86_64
---> Package chkconfig.x86_64 0:1.7.4-1.el7 will be an update
---> Package yum-utils.noarch 0:1.1.31-40.el7 will be updated
---> Package yum-utils.noarch 0:1.1.31-42.el7 will be an update
--> Running transaction check
---> Package ntsysv.x86_64 0:1.7.2-1.el7 will be updated
---> Package ntsysv.x86_64 0:1.7.4-1.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch           Version               Repository         Size
================================================================================
Installing:
 PyYAML            x86_64         3.11-1.el7            saltstack         160 k
Updating:
 chkconfig         x86_64         1.7.4-1.el7           base              181 k
 yum-utils         noarch         1.1.31-42.el7         base              117 k
Updating for dependencies:
 ntsysv            x86_64         1.7.4-1.el7           base               38 k

Transaction Summary
================================================================================
Install  1 Package
Upgrade  2 Packages (+1 Dependent package)

Total download size: 497 k
Downloading packages:
No Presto metadata available for base
--------------------------------------------------------------------------------
Total                                              190 kB/s | 497 kB  00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : chkconfig-1.7.4-1.el7.x86_64                                 1/7
  Updating   : ntsysv-1.7.4-1.el7.x86_64                                    2/7
  Installing : PyYAML-3.11-1.el7.x86_64                                     3/7
  Updating   : yum-utils-1.1.31-42.el7.noarch                               4/7
  Cleanup    : yum-utils-1.1.31-40.el7.noarch                               5/7
  Cleanup    : ntsysv-1.7.2-1.el7.x86_64                                    6/7
  Cleanup    : chkconfig-1.7.2-1.el7.x86_64                                 7/7
  Verifying  : yum-utils-1.1.31-42.el7.noarch                               1/7
  Verifying  : chkconfig-1.7.4-1.el7.x86_64                                 2/7
  Verifying  : PyYAML-3.11-1.el7.x86_64                                     3/7
  Verifying  : ntsysv-1.7.4-1.el7.x86_64                                    4/7
  Verifying  : chkconfig-1.7.2-1.el7.x86_64                                 5/7
  Verifying  : yum-utils-1.1.31-40.el7.noarch                               6/7
  Verifying  : ntsysv-1.7.2-1.el7.x86_64                                    7/7

Installed:
  PyYAML.x86_64 0:3.11-1.el7                                                    

Updated:
  chkconfig.x86_64 0:1.7.4-1.el7        yum-utils.noarch 0:1.1.31-42.el7       

Dependency Updated:
  ntsysv.x86_64 0:1.7.4-1.el7                                                   

Complete!
 *  INFO: Running install_centos_stable()
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package salt-master.noarch 0:2017.7.3-1.el7 will be installed
--> Processing Dependency: salt = 2017.7.3-1.el7 for package: salt-master-2017.7.3-1.el7.noarch
---> Package salt-minion.noarch 0:2017.7.3-1.el7 will be installed
--> Running transaction check
---> Package salt.noarch 0:2017.7.3-1.el7 will be installed
--> Processing Dependency: python-msgpack > 0.3 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-tornado >= 4.2.1 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-futures >= 2.0 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-crypto >= 2.6.1 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-zmq for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-psutil for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-jinja2 for package: salt-2017.7.3-1.el7.noarch
--> Running transaction check
---> Package python-jinja2.noarch 0:2.7.2-2.el7 will be installed
--> Processing Dependency: python-babel >= 0.8 for package: python-jinja2-2.7.2-2.el7.noarch
---> Package python-tornado.x86_64 0:4.2.1-1.el7 will be installed
---> Package python-zmq.x86_64 0:15.3.0-2.el7 will be installed
--> Processing Dependency: libzmq.so.5()(64bit) for package: python-zmq-15.3.0-2.el7.x86_64
---> Package python2-crypto.x86_64 0:2.6.1-15.el7 will be installed
--> Processing Dependency: libtomcrypt.so.0()(64bit) for package: python2-crypto-2.6.1-15.el7.x86_64
---> Package python2-futures.noarch 0:3.0.5-1.el7 will be installed
---> Package python2-msgpack.x86_64 0:0.5.1-1.el7 will be installed
---> Package python2-psutil.x86_64 0:2.2.1-3.el7 will be installed
--> Running transaction check
---> Package libtomcrypt.x86_64 0:1.17-26.el7 will be installed
--> Processing Dependency: libtommath >= 0.42.0 for package: libtomcrypt-1.17-26.el7.x86_64
--> Processing Dependency: libtommath.so.0()(64bit) for package: libtomcrypt-1.17-26.el7.x86_64
---> Package python-babel.noarch 0:0.9.6-8.el7 will be installed
---> Package zeromq.x86_64 0:4.1.4-6.el7 will be installed
--> Processing Dependency: libsodium.so.13()(64bit) for package: zeromq-4.1.4-6.el7.x86_64
--> Processing Dependency: libpgm-5.2.so.0()(64bit) for package: zeromq-4.1.4-6.el7.x86_64
--> Running transaction check
---> Package libsodium13.x86_64 0:1.0.5-1.el7 will be installed
---> Package libtommath.x86_64 0:0.42.0-6.el7 will be installed
---> Package openpgm.x86_64 0:5.2.122-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch         Version               Repository       Size
================================================================================
Installing:
 salt-master           noarch       2017.7.3-1.el7        saltstack       2.0 M
 salt-minion           noarch       2017.7.3-1.el7        saltstack        35 k
Installing for dependencies:
 libsodium13           x86_64       1.0.5-1.el7           epel            144 k
 libtomcrypt           x86_64       1.17-26.el7           extras          224 k
 libtommath            x86_64       0.42.0-6.el7          extras           36 k
 openpgm               x86_64       5.2.122-2.el7         epel            171 k
 python-babel          noarch       0.9.6-8.el7           base            1.4 M
 python-jinja2         noarch       2.7.2-2.el7           base            515 k
 python-tornado        x86_64       4.2.1-1.el7           base            636 k
 python-zmq            x86_64       15.3.0-2.el7          saltstack       520 k
 python2-crypto        x86_64       2.6.1-15.el7          extras          477 k
 python2-futures       noarch       3.0.5-1.el7           epel             26 k
 python2-msgpack       x86_64       0.5.1-1.el7           epel             93 k
 python2-psutil        x86_64       2.2.1-3.el7           epel            116 k
 salt                  noarch       2017.7.3-1.el7        saltstack       7.9 M
 zeromq                x86_64       4.1.4-6.el7           saltstack       555 k

Transaction Summary
================================================================================
Install  2 Packages (+14 Dependent packages)

Total download size: 15 M
Installed size: 58 M
Downloading packages:
--------------------------------------------------------------------------------
Total                                              271 kB/s |  15 MB  00:55     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libsodium13-1.0.5-1.el7.x86_64                              1/16
  Installing : python-babel-0.9.6-8.el7.noarch                             2/16
  Installing : python-jinja2-2.7.2-2.el7.noarch                            3/16
  Installing : python-tornado-4.2.1-1.el7.x86_64                           4/16
  Installing : libtommath-0.42.0-6.el7.x86_64                              5/16
  Installing : libtomcrypt-1.17-26.el7.x86_64                              6/16
  Installing : python2-crypto-2.6.1-15.el7.x86_64                          7/16
  Installing : python2-msgpack-0.5.1-1.el7.x86_64                          8/16
  Installing : python2-psutil-2.2.1-3.el7.x86_64                           9/16
  Installing : python2-futures-3.0.5-1.el7.noarch                         10/16
  Installing : openpgm-5.2.122-2.el7.x86_64                               11/16
  Installing : zeromq-4.1.4-6.el7.x86_64                                  12/16
  Installing : python-zmq-15.3.0-2.el7.x86_64                             13/16
  Installing : salt-2017.7.3-1.el7.noarch                                 14/16
  Installing : salt-master-2017.7.3-1.el7.noarch                          15/16
  Installing : salt-minion-2017.7.3-1.el7.noarch                          16/16
  Verifying  : salt-master-2017.7.3-1.el7.noarch                           1/16
  Verifying  : python-jinja2-2.7.2-2.el7.noarch                            2/16
  Verifying  : openpgm-5.2.122-2.el7.x86_64                                3/16
  Verifying  : salt-2017.7.3-1.el7.noarch                                  4/16
  Verifying  : python2-futures-3.0.5-1.el7.noarch                          5/16
  Verifying  : python2-crypto-2.6.1-15.el7.x86_64                          6/16
  Verifying  : python2-psutil-2.2.1-3.el7.x86_64                           7/16
  Verifying  : python-zmq-15.3.0-2.el7.x86_64                              8/16
  Verifying  : python2-msgpack-0.5.1-1.el7.x86_64                          9/16
  Verifying  : libtommath-0.42.0-6.el7.x86_64                             10/16
  Verifying  : python-tornado-4.2.1-1.el7.x86_64                          11/16
  Verifying  : zeromq-4.1.4-6.el7.x86_64                                  12/16
  Verifying  : python-babel-0.9.6-8.el7.noarch                            13/16
  Verifying  : libtomcrypt-1.17-26.el7.x86_64                             14/16
  Verifying  : salt-minion-2017.7.3-1.el7.noarch                          15/16
  Verifying  : libsodium13-1.0.5-1.el7.x86_64                             16/16

Installed:
  salt-master.noarch 0:2017.7.3-1.el7    salt-minion.noarch 0:2017.7.3-1.el7   

Dependency Installed:
  libsodium13.x86_64 0:1.0.5-1.el7       libtomcrypt.x86_64 0:1.17-26.el7      
  libtommath.x86_64 0:0.42.0-6.el7       openpgm.x86_64 0:5.2.122-2.el7        
  python-babel.noarch 0:0.9.6-8.el7      python-jinja2.noarch 0:2.7.2-2.el7    
  python-tornado.x86_64 0:4.2.1-1.el7    python-zmq.x86_64 0:15.3.0-2.el7      
  python2-crypto.x86_64 0:2.6.1-15.el7   python2-futures.noarch 0:3.0.5-1.el7  
  python2-msgpack.x86_64 0:0.5.1-1.el7   python2-psutil.x86_64 0:2.2.1-3.el7   
  salt.noarch 0:2017.7.3-1.el7           zeromq.x86_64 0:4.1.4-6.el7           

Complete!
 *  INFO: Running install_centos_stable_post()
 *  INFO: Running install_centos_check_services()
 *  INFO: Running install_centos_restart_daemons()
 *  INFO: Running daemons_running()
 *  INFO: Salt installed!
[root@much ~]# echo $?
0
[root@much ~]# rpm -qa | grep -i salt
salt-2017.7.3-1.el7.noarch
salt-master-2017.7.3-1.el7.noarch
salt-minion-2017.7.3-1.el7.noarch
[root@much ~]#
~~~


## 安装 salt minion


在 minion 上(192.168.56.208)

~~~
[root@much ~]# curl -L https://bootstrap.saltstack.com -o install_salt.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   264  100   264    0     0    147      0  0:00:01  0:00:01 --:--:--   147
100  244k  100  244k    0     0  71726      0  0:00:03  0:00:03 --:--:--  201k
[root@much ~]# sudo sh install_salt.sh -P
 *  INFO: Running version: 2017.12.13
 *  INFO: Executed by: shell pipe
 *  INFO: Command line: 'install_salt.sh -P'

 *  INFO: System Information:
 *  INFO:   CPU:          GenuineIntel
 *  INFO:   CPU Arch:     x86_64
 *  INFO:   OS Name:      Linux
 *  INFO:   OS Version:   3.10.0-514.21.1.el7.x86_64
 *  INFO:   Distribution: CentOS 7.3

 *  INFO: Installing minion
 *  INFO: Found function install_centos_stable_deps
 *  INFO: Found function config_salt
 *  INFO: Found function preseed_master
 *  INFO: Found function install_centos_stable
 *  INFO: Found function install_centos_stable_post
 *  INFO: Found function install_centos_restart_daemons
 *  INFO: Found function daemons_running
 *  INFO: Found function install_centos_check_services
 *  INFO: Running install_centos_stable_deps()
Loaded plugins: fastestmirror, langpacks
https://ftp.yzu.edu.tw/Linux/Fedora-EPEL/7/x86_64/repodata/bdb4124d91bfddbc7a700e9ee228e462ff7d3652d93aac38b327e14fea76911e-primary.sqlite.bz2: [Errno 14] curl#56 - "TCP connection reset by peer"
Trying other mirror.
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package PyYAML.x86_64 0:3.11-1.el7 will be installed
---> Package chkconfig.x86_64 0:1.7.2-1.el7 will be updated
--> Processing Dependency: chkconfig = 1.7.2-1.el7 for package: ntsysv-1.7.2-1.el7.x86_64
---> Package chkconfig.x86_64 0:1.7.4-1.el7 will be an update
---> Package yum-utils.noarch 0:1.1.31-40.el7 will be updated
---> Package yum-utils.noarch 0:1.1.31-42.el7 will be an update
--> Running transaction check
---> Package ntsysv.x86_64 0:1.7.2-1.el7 will be updated
---> Package ntsysv.x86_64 0:1.7.4-1.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch           Version               Repository         Size
================================================================================
Installing:
 PyYAML            x86_64         3.11-1.el7            saltstack         160 k
Updating:
 chkconfig         x86_64         1.7.4-1.el7           base              181 k
 yum-utils         noarch         1.1.31-42.el7         base              117 k
Updating for dependencies:
 ntsysv            x86_64         1.7.4-1.el7           base               38 k

Transaction Summary
================================================================================
Install  1 Package
Upgrade  2 Packages (+1 Dependent package)

Total download size: 497 k
Downloading packages:
No Presto metadata available for base
--------------------------------------------------------------------------------
Total                                              195 kB/s | 497 kB  00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : chkconfig-1.7.4-1.el7.x86_64                                 1/7
  Updating   : ntsysv-1.7.4-1.el7.x86_64                                    2/7
  Installing : PyYAML-3.11-1.el7.x86_64                                     3/7
  Updating   : yum-utils-1.1.31-42.el7.noarch                               4/7
  Cleanup    : yum-utils-1.1.31-40.el7.noarch                               5/7
  Cleanup    : ntsysv-1.7.2-1.el7.x86_64                                    6/7
  Cleanup    : chkconfig-1.7.2-1.el7.x86_64                                 7/7
  Verifying  : yum-utils-1.1.31-42.el7.noarch                               1/7
  Verifying  : chkconfig-1.7.4-1.el7.x86_64                                 2/7
  Verifying  : PyYAML-3.11-1.el7.x86_64                                     3/7
  Verifying  : ntsysv-1.7.4-1.el7.x86_64                                    4/7
  Verifying  : chkconfig-1.7.2-1.el7.x86_64                                 5/7
  Verifying  : yum-utils-1.1.31-40.el7.noarch                               6/7
  Verifying  : ntsysv-1.7.2-1.el7.x86_64                                    7/7

Installed:
  PyYAML.x86_64 0:3.11-1.el7                                                    

Updated:
  chkconfig.x86_64 0:1.7.4-1.el7        yum-utils.noarch 0:1.1.31-42.el7       

Dependency Updated:
  ntsysv.x86_64 0:1.7.4-1.el7                                                   

Complete!
 *  INFO: Running install_centos_stable()
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package salt-minion.noarch 0:2017.7.3-1.el7 will be installed
--> Processing Dependency: salt = 2017.7.3-1.el7 for package: salt-minion-2017.7.3-1.el7.noarch
--> Running transaction check
---> Package salt.noarch 0:2017.7.3-1.el7 will be installed
--> Processing Dependency: python-msgpack > 0.3 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-tornado >= 4.2.1 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-futures >= 2.0 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-crypto >= 2.6.1 for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-psutil for package: salt-2017.7.3-1.el7.noarch
--> Processing Dependency: python-jinja2 for package: salt-2017.7.3-1.el7.noarch
--> Running transaction check
---> Package python-jinja2.noarch 0:2.7.2-2.el7 will be installed
--> Processing Dependency: python-babel >= 0.8 for package: python-jinja2-2.7.2-2.el7.noarch
---> Package python-tornado.x86_64 0:4.2.1-1.el7 will be installed
---> Package python2-crypto.x86_64 0:2.6.1-15.el7 will be installed
--> Processing Dependency: libtomcrypt.so.0()(64bit) for package: python2-crypto-2.6.1-15.el7.x86_64
---> Package python2-futures.noarch 0:3.0.5-1.el7 will be installed
---> Package python2-msgpack.x86_64 0:0.5.1-1.el7 will be installed
---> Package python2-psutil.x86_64 0:2.2.1-3.el7 will be installed
--> Running transaction check
---> Package libtomcrypt.x86_64 0:1.17-26.el7 will be installed
--> Processing Dependency: libtommath >= 0.42.0 for package: libtomcrypt-1.17-26.el7.x86_64
--> Processing Dependency: libtommath.so.0()(64bit) for package: libtomcrypt-1.17-26.el7.x86_64
---> Package python-babel.noarch 0:0.9.6-8.el7 will be installed
--> Running transaction check
---> Package libtommath.x86_64 0:0.42.0-6.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch         Version               Repository       Size
================================================================================
Installing:
 salt-minion           noarch       2017.7.3-1.el7        saltstack        35 k
Installing for dependencies:
 libtomcrypt           x86_64       1.17-26.el7           extras          224 k
 libtommath            x86_64       0.42.0-6.el7          extras           36 k
 python-babel          noarch       0.9.6-8.el7           base            1.4 M
 python-jinja2         noarch       2.7.2-2.el7           base            515 k
 python-tornado        x86_64       4.2.1-1.el7           base            636 k
 python2-crypto        x86_64       2.6.1-15.el7          extras          477 k
 python2-futures       noarch       3.0.5-1.el7           epel             26 k
 python2-msgpack       x86_64       0.5.1-1.el7           epel             93 k
 python2-psutil        x86_64       2.2.1-3.el7           epel            116 k
 salt                  noarch       2017.7.3-1.el7        saltstack       7.9 M

Transaction Summary
================================================================================
Install  1 Package (+10 Dependent packages)

Total download size: 11 M
Installed size: 52 M
Downloading packages:
--------------------------------------------------------------------------------
Total                                              270 kB/s |  11 MB  00:43     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python-babel-0.9.6-8.el7.noarch                             1/11
  Installing : python-jinja2-2.7.2-2.el7.noarch                            2/11
  Installing : python-tornado-4.2.1-1.el7.x86_64                           3/11
  Installing : libtommath-0.42.0-6.el7.x86_64                              4/11
  Installing : libtomcrypt-1.17-26.el7.x86_64                              5/11
  Installing : python2-crypto-2.6.1-15.el7.x86_64                          6/11
  Installing : python2-msgpack-0.5.1-1.el7.x86_64                          7/11
  Installing : python2-futures-3.0.5-1.el7.noarch                          8/11
  Installing : python2-psutil-2.2.1-3.el7.x86_64                           9/11
  Installing : salt-2017.7.3-1.el7.noarch                                 10/11
  Installing : salt-minion-2017.7.3-1.el7.noarch                          11/11
  Verifying  : salt-2017.7.3-1.el7.noarch                                  1/11
  Verifying  : python-jinja2-2.7.2-2.el7.noarch                            2/11
  Verifying  : python2-psutil-2.2.1-3.el7.x86_64                           3/11
  Verifying  : python2-futures-3.0.5-1.el7.noarch                          4/11
  Verifying  : python2-crypto-2.6.1-15.el7.x86_64                          5/11
  Verifying  : python2-msgpack-0.5.1-1.el7.x86_64                          6/11
  Verifying  : libtommath-0.42.0-6.el7.x86_64                              7/11
  Verifying  : python-tornado-4.2.1-1.el7.x86_64                           8/11
  Verifying  : python-babel-0.9.6-8.el7.noarch                             9/11
  Verifying  : libtomcrypt-1.17-26.el7.x86_64                             10/11
  Verifying  : salt-minion-2017.7.3-1.el7.noarch                          11/11

Installed:
  salt-minion.noarch 0:2017.7.3-1.el7                                           

Dependency Installed:
  libtomcrypt.x86_64 0:1.17-26.el7       libtommath.x86_64 0:0.42.0-6.el7      
  python-babel.noarch 0:0.9.6-8.el7      python-jinja2.noarch 0:2.7.2-2.el7    
  python-tornado.x86_64 0:4.2.1-1.el7    python2-crypto.x86_64 0:2.6.1-15.el7  
  python2-futures.noarch 0:3.0.5-1.el7   python2-msgpack.x86_64 0:0.5.1-1.el7  
  python2-psutil.x86_64 0:2.2.1-3.el7    salt.noarch 0:2017.7.3-1.el7          

Complete!
 *  INFO: Running install_centos_stable_post()
 *  INFO: Running install_centos_check_services()
 *  INFO: Running install_centos_restart_daemons()
 *  INFO: Running daemons_running()
 *  INFO: Salt installed!
[root@much ~]# echo $?
0
[root@much ~]# rpm -qa | grep -i salt
salt-2017.7.3-1.el7.noarch
salt-minion-2017.7.3-1.el7.noarch
[root@much ~]#
~~~

## 脚本参数

**`-P`** 允许使用 pip 进行安装

**`-M`** 安装 salt-master

默认就是安装 salt-minion 的

~~~
[root@much ~]# sh install_salt.sh -h

  Usage :  bootstrap-salt.sh [options] <install-type> [install-type-args]

  Installation types:
    - stable              Install latest stable release. This is the default
                          install type
    - stable [branch]     Install latest version on a branch. Only supported
                          for packages available at repo.saltstack.com
    - stable [version]    Install a specific version. Only supported for
                          packages available at repo.saltstack.com
    - daily               Ubuntu specific: configure SaltStack Daily PPA
    - testing             RHEL-family specific: configure EPEL testing repo
    - git                 Install from the head of the develop branch
    - git [ref]           Install from any git ref (such as a branch, tag, or
                          commit)

  Examples:
    - bootstrap-salt.sh
    - bootstrap-salt.sh stable
    - bootstrap-salt.sh stable 2016.3
    - bootstrap-salt.sh stable 2016.3.1
    - bootstrap-salt.sh daily
    - bootstrap-salt.sh testing
    - bootstrap-salt.sh git
    - bootstrap-salt.sh git 2016.3
    - bootstrap-salt.sh git v2016.3.1
    - bootstrap-salt.sh git 06f249901a2e2f1ed310d58ea3921a129f214358

  Options:
    -h  Display this message
    -v  Display script version
    -n  No colours
    -D  Show debug output
    -c  Temporary configuration directory
    -g  Salt Git repository URL. Default: https://github.com/saltstack/salt.git
    -w  Install packages from downstream package repository rather than
        upstream, saltstack package repository. This is currently only
        implemented for SUSE.
    -k  Temporary directory holding the minion keys which will pre-seed
        the master.
    -s  Sleep time used when waiting for daemons to start, restart and when
        checking for the services running. Default: 3
    -L  Also install salt-cloud and required python-libcloud package
    -M  Also install salt-master
    -S  Also install salt-syndic
    -N  Do not install salt-minion
    -X  Do not start daemons after installation
    -d  Disables checking if Salt services are enabled to start on system boot.
        You can also do this by touching /tmp/disable_salt_checks on the target
        host. Default: ${BS_FALSE}
    -P  Allow pip based installations. On some distributions the required salt
        packages or its dependencies are not available as a package for that
        distribution. Using this flag allows the script to use pip as a last
        resort method. NOTE: This only works for functions which actually
        implement pip based installations.
    -U  If set, fully upgrade the system prior to bootstrapping Salt
    -I  If set, allow insecure connections while downloading any files. For
        example, pass '--no-check-certificate' to 'wget' or '--insecure' to
        'curl'. On Debian and Ubuntu, using this option with -U allows to obtain
        GnuPG archive keys insecurely if distro has changed release signatures.
    -F  Allow copied files to overwrite existing (config, init.d, etc)
    -K  If set, keep the temporary files in the temporary directories specified
        with -c and -k
    -C  Only run the configuration function. Implies -F (forced overwrite).
        To overwrite Master or Syndic configs, -M or -S, respectively, must
        also be specified. Salt installation will be ommitted, but some of the
        dependencies could be installed to write configuration with -j or -J.
    -A  Pass the salt-master DNS name or IP. This will be stored under
        ${BS_SALT_ETC_DIR}/minion.d/99-master-address.conf
    -i  Pass the salt-minion id. This will be stored under
        ${BS_SALT_ETC_DIR}/minion_id
    -p  Extra-package to install while installing Salt dependencies. One package
        per -p flag. You're responsible for providing the proper package name.
    -H  Use the specified HTTP proxy for all download URLs (including https://).
        For example: http://myproxy.example.com:3128
    -Z  Enable additional package repository for newer ZeroMQ
        (only available for RHEL/CentOS/Fedora/Ubuntu based distributions)
    -b  Assume that dependencies are already installed and software sources are
        set up. If git is selected, git tree is still checked out as dependency
        step.
    -f  Force shallow cloning for git installations.
        This may result in an "n/a" in the version number.
    -l  Disable ssl checks. When passed, switches "https" calls to "http" where
        possible.
    -V  Install Salt into virtualenv
        (only available for Ubuntu based distributions)
    -a  Pip install all Python pkg dependencies for Salt. Requires -V to install
        all pip pkgs into the virtualenv.
        (Only available for Ubuntu based distributions)
    -r  Disable all repository configuration performed by this script. This
        option assumes all necessary repository configuration is already present
        on the system.
    -R  Specify a custom repository URL. Assumes the custom repository URL
        points to a repository that mirrors Salt packages located at
        repo.saltstack.com. The option passed with -R replaces the
        "repo.saltstack.com". If -R is passed, -r is also set. Currently only
        works on CentOS/RHEL and Debian based distributions.
    -J  Replace the Master config file with data passed in as a JSON string. If
        a Master config file is found, a reasonable effort will be made to save
        the file with a ".bak" extension. If used in conjunction with -C or -F,
        no ".bak" file will be created as either of those options will force
        a complete overwrite of the file.
    -j  Replace the Minion config file with data passed in as a JSON string. If
        a Minion config file is found, a reasonable effort will be made to save
        the file with a ".bak" extension. If used in conjunction with -C or -F,
        no ".bak" file will be created as either of those options will force
        a complete overwrite of the file.
    -q  Quiet salt installation from git (setup.py install -q)
    -x  Changes the python version used to install a git version of salt. Currently
        this is considered experimental and has only been tested on Centos 6. This
        only works for git installations.
    -y  Installs a different python version on host. Currently this has only been
        tested with Centos 6 and is considered experimental. This will install the
        ius repo on the box if disable repo is false. This must be used in conjunction
        with -x <pythonversion>.  For example:
            sh bootstrap.sh -P -y -x python2.7 git v2016.11.3
        The above will install python27 and install the git version of salt using the
        python2.7 executable. This only works for git and pip installations.

[root@much ~]#
~~~

这个 shell 脚本可以侦测目标系统的环境，并且选择最合适的安装方法

在我的环境中，是使用的 yum 安装的

~~~
[root@much ~]# ll /etc/yum.repos.d/saltstack.repo
-rw-r--r-- 1 root root 295 2月  17 19:20 /etc/yum.repos.d/saltstack.repo
[root@much ~]#
~~~

## 配置 master

默认情况下 master 就监听在了 **`0.0.0.0`** 和 **`4505 4506`** 端口

~~~
[root@much ~]# grep interface: /etc/salt/master
#interface: 0.0.0.0
# the interface option must be adjusted, too. (For example: "interface: '::'")
[root@much ~]# grep 45 /etc/salt/master
#publish_port: 4505
#ret_port: 4506
#syndic_master_port: 4506
[root@much ~]#
~~~

启动服务，查看状态

~~~
[root@much ~]# systemctl | grep master
  salt-master.service                                                                      loaded active running   The Salt Master Server
[root@much ~]# systemctl restart salt-master.service
[root@much ~]# systemctl status salt-master.service
● salt-master.service - The Salt Master Server
   Loaded: loaded (/usr/lib/systemd/system/salt-master.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2018-02-17 20:00:10 CST; 16s ago
     Docs: man:salt-master(1)
           file:///usr/share/doc/salt/html/contents.html
           https://docs.saltstack.com/en/latest/contents.html
 Main PID: 8550 (salt-master)
   CGroup: /system.slice/salt-master.service
           ├─8550 /usr/bin/python /usr/bin/salt-master
           ├─8555 /usr/bin/python /usr/bin/salt-master
           ├─8560 /usr/bin/python /usr/bin/salt-master
           ├─8563 /usr/bin/python /usr/bin/salt-master
           ├─8564 /usr/bin/python /usr/bin/salt-master
           ├─8565 /usr/bin/python /usr/bin/salt-master
           ├─8566 /usr/bin/python /usr/bin/salt-master
           ├─8567 /usr/bin/python /usr/bin/salt-master
           ├─8568 /usr/bin/python /usr/bin/salt-master
           ├─8569 /usr/bin/python /usr/bin/salt-master
           ├─8576 /usr/bin/python /usr/bin/salt-master
           └─8577 /usr/bin/python /usr/bin/salt-master

2月 17 20:00:10 much systemd[1]: Starting The Salt Master Server...
2月 17 20:00:10 much systemd[1]: Started The Salt Master Server.
[root@much ~]#       
[root@much ~]# netstat -antp | grep python
tcp        0      0 0.0.0.0:4505            0.0.0.0:*               LISTEN      8560/python         
tcp        0      0 0.0.0.0:4506            0.0.0.0:*               LISTEN      8566/python         
[root@much ~]#
~~~



## 配置 minion


默认 minion 会去找解析名为 **salt** 的服务器

我们修改为 192.168.56.209，重启服务应用配置

~~~
[root@much ~]# grep master: /etc/salt/minion
#master: salt
#random_master: False
# Location of the repository cache file on the master:
[root@much ~]# vim /etc/salt/minion
[root@much ~]# grep master: /etc/salt/minion
#master: salt
master: 192.168.56.209
#random_master: False
# Location of the repository cache file on the master:
[root@much ~]#    
[root@much ~]# systemctl | grep minion
  salt-minion.service                                                                      loaded active running   The Salt Minion
[root@much ~]# systemctl  restart salt-minion.service    
[root@much ~]# systemctl  status  salt-minion.service    
● salt-minion.service - The Salt Minion
   Loaded: loaded (/usr/lib/systemd/system/salt-minion.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2018-02-17 21:39:19 CST; 6s ago
     Docs: man:salt-minion(1)
           file:///usr/share/doc/salt/html/contents.html
           https://docs.saltstack.com/en/latest/contents.html
 Main PID: 19014 (salt-minion)
   CGroup: /system.slice/salt-minion.service
           ├─19014 /usr/bin/python /usr/bin/salt-minion
           ├─19017 /usr/bin/python /usr/bin/salt-minion
           └─19021 /usr/bin/python /usr/bin/salt-minion

2月 17 21:39:19 much systemd[1]: Starting The Salt Minion...
2月 17 21:39:19 much systemd[1]: Started The Salt Minion.
[root@much ~]#
~~~


salt 是以 root 的身份运行的，所以它有很大的权限在系统中可以为所欲为

~~~
[root@much ~]# ps faux | grep salt
root     17398  0.0  0.0 112652  1016 pts/0    S+   21:49   0:00          \_ grep --color=auto salt
root      4394  0.0  0.4 285024 19504 ?        Ss   19:21   0:00 /usr/bin/python /usr/bin/salt-minion
root      4412  0.0  0.9 517892 39064 ?        Sl   19:21   0:01  \_ /usr/bin/python /usr/bin/salt-minion
root      4432  0.0  0.4 406280 19932 ?        S    19:21   0:00      \_ /usr/bin/python /usr/bin/salt-minion
root      8550  0.0  0.9 392164 37468 ?        Ss   20:00   0:00 /usr/bin/python /usr/bin/salt-master
root      8555  0.0  0.4 312556 19836 ?        S    20:00   0:00  \_ /usr/bin/python /usr/bin/salt-master
root      8560  0.0  0.7 473508 31736 ?        Sl   20:00   0:00  \_ /usr/bin/python /usr/bin/salt-master
root      8563  0.0  0.7 391580 31316 ?        S    20:00   0:00  \_ /usr/bin/python /usr/bin/salt-master
root      8564  0.4  1.4 471208 60380 ?        S    20:00   0:28  \_ /usr/bin/python /usr/bin/salt-master
root      8565  0.0  0.7 392164 31668 ?        S    20:00   0:00  \_ /usr/bin/python /usr/bin/salt-master
root      8566  0.0  0.7 769020 32192 ?        Sl   20:00   0:00      \_ /usr/bin/python /usr/bin/salt-master
root      8567  0.0  1.0 533320 43576 ?        Sl   20:00   0:01      \_ /usr/bin/python /usr/bin/salt-master
root      8568  0.0  1.0 533320 43584 ?        Sl   20:00   0:01      \_ /usr/bin/python /usr/bin/salt-master
root      8569  0.0  1.0 533324 43568 ?        Sl   20:00   0:01      \_ /usr/bin/python /usr/bin/salt-master
root      8576  0.0  1.0 533328 43588 ?        Sl   20:00   0:01      \_ /usr/bin/python /usr/bin/salt-master
root      8577  0.0  1.0 533328 43584 ?        Sl   20:00   0:01      \_ /usr/bin/python /usr/bin/salt-master
[root@much ~]#
------
[root@much ~]# ps faux | grep salt
root     19631  0.0  0.0 112660  1016 pts/0    S+   21:48   0:00          \_ grep --color=auto salt
root     19014  0.0  0.4 285412 19564 ?        Ss   21:39   0:00 /usr/bin/python /usr/bin/salt-minion
root     19017  0.1  1.7 854516 72316 ?        Sl   21:39   0:01  \_ /usr/bin/python /usr/bin/salt-minion
root     19021  0.0  0.5 406036 23736 ?        S    21:39   0:00      \_ /usr/bin/python /usr/bin/salt-minion
[root@much ~]#
~~~


## 打开防火墙


在 master 上

~~~
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 8080/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]#
[root@much ~]# firewall-cmd --add-port 4505/tcp --permanent
success
[root@much ~]# firewall-cmd --add-port 4506/tcp --permanent
success
[root@much ~]# firewall-cmd --reload
success
[root@much ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 4505/tcp 8080/tcp 4506/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much ~]#
~~~


## 密钥管理

由于 master 与 minion 的 id 都叫 **much**

就会出现如下问题

~~~
[root@much ~]# salt-key  -L
Accepted Keys:
Denied Keys:
much
Unaccepted Keys:
much
Rejected Keys:
[root@much ~]#
~~~

我们要把 id 改过来


## 修改主机名

为了区分，将主机名也修改一下

minion

~~~
[root@much ~]# hostnamectl set-hostname  h208
[root@much ~]# su - root
Last login: 六 2月 17 19:19:36 CST 2018 from 192.168.56.1 on pts/0
[root@h208 ~]#
~~~

master

~~~
[root@much ~]# hostnamectl set-hostname h209
[root@much ~]# su - root
Last login: 六 2月 17 19:03:56 CST 2018 from 192.168.56.1 on pts/0
[root@h209 ~]#
~~~

## 配置 minion

~~~
[root@h208 log]# cat /etc/salt/minion_id
much
[root@h208 log]# ll /etc/salt/pki/minion/
total 8
-r-------- 1 root root 1674 2月  17 22:35 minion.pem
-rw-r--r-- 1 root root  450 2月  17 22:35 minion.pub
[root@h208 log]# vim /etc/salt/minion
[root@h208 log]# grep id: /etc/salt/minion
#id:
id: test-minion
[root@h208 log]# echo test-minion > /etc/salt/minion_id
[root@h208 log]# cat /etc/salt/minion_id
test-minion
[root@h208 log]# rm /etc/salt/pki/minion/*
rm: remove regular file ‘/etc/salt/pki/minion/minion.pem’? y
rm: remove regular file ‘/etc/salt/pki/minion/minion.pub’? y
[root@h208 log]# ll /etc/salt/pki/minion/
total 0
[root@h208 log]# systemctl restart salt-minion.service
[root@h208 log]# systemctl status salt-minion.service
● salt-minion.service - The Salt Minion
   Loaded: loaded (/usr/lib/systemd/system/salt-minion.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2018-02-17 22:40:27 CST; 6s ago
     Docs: man:salt-minion(1)
           file:///usr/share/doc/salt/html/contents.html
           https://docs.saltstack.com/en/latest/contents.html
 Main PID: 2559 (salt-minion)
   CGroup: /system.slice/salt-minion.service
           ├─2559 /usr/bin/python /usr/bin/salt-minion
           ├─2562 /usr/bin/python /usr/bin/salt-minion
           └─2566 /usr/bin/python /usr/bin/salt-minion

2月 17 22:40:27 h208 systemd[1]: Starting The Salt Minion...
2月 17 22:40:27 h208 systemd[1]: Started The Salt Minion.
2月 17 22:40:29 h208 salt-minion[2559]: [ERROR   ] The Salt Master has cach...e
Hint: Some lines were ellipsized, use -l to show in full.
[root@h208 log]# ll /etc/salt/pki/minion/
total 8
-r-------- 1 root root 1678 2月  17 22:40 minion.pem
-rw-r--r-- 1 root root  450 2月  17 22:40 minion.pub
[root@h208 log]#
~~~

重启后，相关密钥会自动生成


再在 master 上看一下密钥

~~~
[root@h209 ~]# salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
test-minion
Rejected Keys:
[root@h209 ~]#
~~~


在 master 上也操作一遍

~~~
[root@h209 ~]# cat /etc/salt/minion_id
much[root@h209 ~]# ll /etc/salt/pki/minion/
total 8
-r-------- 1 root root 1674 2月  17 22:06 minion.pem
-rw-r--r-- 1 root root  450 2月  17 22:06 minion.pub
[root@h209 ~]# vim /etc/salt/minion
[root@h209 ~]# grep id: /etc/salt/minion
#id:
id: test-master
[root@h209 ~]# rm /etc/salt/pki/minion/*
rm: remove regular file ‘/etc/salt/pki/minion/minion.pem’? y
rm: remove regular file ‘/etc/salt/pki/minion/minion.pub’? y
[root@h209 ~]# ll /etc/salt/pki/minion/
total 0
[root@h209 ~]# systemctl restart salt-minion.service
[root@h209 ~]# systemctl status salt-minion.service
● salt-minion.service - The Salt Minion
   Loaded: loaded (/usr/lib/systemd/system/salt-minion.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2018-02-17 22:45:50 CST; 6s ago
     Docs: man:salt-minion(1)
           file:///usr/share/doc/salt/html/contents.html
           https://docs.saltstack.com/en/latest/contents.html
 Main PID: 8317 (salt-minion)
   CGroup: /system.slice/salt-minion.service
           ├─8317 /usr/bin/python /usr/bin/salt-minion
           ├─8320 /usr/bin/python /usr/bin/salt-minion
           └─8324 /usr/bin/python /usr/bin/salt-minion

2月 17 22:45:50 h209 systemd[1]: Starting The Salt Minion...
2月 17 22:45:50 h209 systemd[1]: Started The Salt Minion.
2月 17 22:45:51 h209 salt-minion[8317]: [ERROR   ] The Salt Master has cached the public key for this node...icate
Hint: Some lines were ellipsized, use -l to show in full.
[root@h209 ~]# ll /etc/salt/pki/minion/
total 8
-r-------- 1 root root 1674 2月  17 22:45 minion.pem
-rw-r--r-- 1 root root  450 2月  17 22:45 minion.pub
[root@h209 ~]#
[root@h209 ~]# cat  /etc/salt/minion_id ; salt-key -L
muchAccepted Keys:
Denied Keys:
Unaccepted Keys:
test-master
test-minion
Rejected Keys:
[root@h209 ~]#
~~~

对于 **master** 的这个小实验，说明只要配置文件里修改了，下次启动就会生效，貌似跟 **`/etc/salt/minion_id`** 没什么关系


## 接受密钥

从 master 端接受 minion 的密钥是后面所有通讯与管理的基础

~~~
[root@h209 ~]# salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
test-master
test-minion
Rejected Keys:
[root@h209 ~]# salt-key -a test-minion
The following keys are going to be accepted:
Unaccepted Keys:
test-minion
Proceed? [n/Y] y
Key for minion test-minion accepted.
[root@h209 ~]# salt-key -L
Accepted Keys:
test-minion
Denied Keys:
Unaccepted Keys:
test-master
Rejected Keys:
[root@h209 ~]# salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
test-master
Proceed? [n/Y] n
[root@h209 ~]# salt-key -L
Accepted Keys:
test-minion
Denied Keys:
Unaccepted Keys:
test-master
Rejected Keys:
[root@h209 ~]# salt-key -Ay
The following keys are going to be accepted:
Unaccepted Keys:
test-master
Key for minion test-master accepted.
[root@h209 ~]# salt-key -L
Accepted Keys:
test-master
test-minion
Denied Keys:
Unaccepted Keys:
Rejected Keys:
[root@h209 ~]#
~~~

salt 的 CLI 是有色彩的，对于终端用户来讲，十分友好


## 执行命令

使用 **`test.ping`** 可以用来验证 minion 响应

也可以执行一些只有 root 才有权限执行的操作

~~~
[root@h209 ~]# salt '*' test.ping
test-master:
    True
test-minion:
    True
[root@h209 ~]#
[root@h209 ~]# salt '*' cmd.run 'cat /etc/shadow'
test-master:
    root:$6$z5FtmEUU0kH0fuF2$9x/RaANrHhawl8AFzP55BEGDI71ogi7283ZvuZYSnPq7q60XES4N7oQlE92k9jPT86LkJNbK/WbibbVbYYMJs.::0:99999:7:::
    bin:*:17110:0:99999:7:::
    daemon:*:17110:0:99999:7:::
    adm:*:17110:0:99999:7:::
    lp:*:17110:0:99999:7:::
    sync:*:17110:0:99999:7:::
    shutdown:*:17110:0:99999:7:::
    halt:*:17110:0:99999:7:::
    mail:*:17110:0:99999:7:::
    operator:*:17110:0:99999:7:::
    games:*:17110:0:99999:7:::
    ftp:*:17110:0:99999:7:::
    nobody:*:17110:0:99999:7:::
    systemd-bus-proxy:!!:17332::::::
    systemd-network:!!:17332::::::
    dbus:!!:17332::::::
    polkitd:!!:17332::::::
    abrt:!!:17332::::::
    unbound:!!:17332::::::
    usbmuxd:!!:17332::::::
    tss:!!:17332::::::
    apache:!!:17332::::::
    libstoragemgmt:!!:17332::::::
    rpc:!!:17332:0:99999:7:::
    colord:!!:17332::::::
    pcp:!!:17332::::::
    saslauth:!!:17332::::::
    geoclue:!!:17332::::::
    setroubleshoot:!!:17332::::::
    rtkit:!!:17332::::::
    qemu:!!:17332::::::
    radvd:!!:17332::::::
    chrony:!!:17332::::::
    ntp:!!:17332::::::
    sssd:!!:17332::::::
    rpcuser:!!:17332::::::
    nfsnobody:!!:17332::::::
    pulse:!!:17332::::::
    gdm:!!:17332::::::
    gnome-initial-setup:!!:17332::::::
    avahi:!!:17332::::::
    postfix:!!:17332::::::
    sshd:!!:17332::::::
    tcpdump:!!:17332::::::
    oprofile:!!:17332::::::
    bolo:$6$.NPfOPh0dQ6pE6R9$I4M1jgxQFv6xJ0hT0/vGPwt9.jr5rFcY5A8t65Ju0VHe82CVP77KJjEZuccYd5zjq9R27eF.ksmvvQF1YR/66/::0:99999:7:::
    vboxadd:!!:17332::::::
    hadoop:$6$SEdaRKSa$qF8csIu103H.0JOIjeCZJrFkHYg7K1LBZ9AOEOmnPogo.bCJUkPIQxF5zI/pGCE6Pd4xyrJX0/Wf3zmCsH/Z3.:17384:0:99999:7:::
    dockerroot:!!:17534::::::
    etcd:!!:17535::::::
    cc:$6$sCsm0ctQ$ou6j3MYUjKGdJ1QOgSylHoCo7uehA1BClUCndqIVocMthwd.uQVy41EkIj5xS6hMLTI02is3Xl.QIT0tpSGB7/:17549:0:99999:7:::
    yy:$6$5cRJR6IN$qldQv77bP9OVA0C4Uc2UNypCxPmxZVq0Fa76t6p0n9lAWHfhX5WSiVlSafgvXXMZpON3GBUGKNRCa1FvQb.Da/:17549:0:99999:7:::
    cassandra:!!:17553::::::
    mysql:!!:17560::::::
    otrs:!!:17560:0:99999:7:::
test-minion:
    root:$6$z5FtmEUU0kH0fuF2$9x/RaANrHhawl8AFzP55BEGDI71ogi7283ZvuZYSnPq7q60XES4N7oQlE92k9jPT86LkJNbK/WbibbVbYYMJs.::0:99999:7:::
    bin:*:17110:0:99999:7:::
    daemon:*:17110:0:99999:7:::
    adm:*:17110:0:99999:7:::
    lp:*:17110:0:99999:7:::
    sync:*:17110:0:99999:7:::
    shutdown:*:17110:0:99999:7:::
    halt:*:17110:0:99999:7:::
    mail:*:17110:0:99999:7:::
    operator:*:17110:0:99999:7:::
    games:*:17110:0:99999:7:::
    ftp:*:17110:0:99999:7:::
    nobody:*:17110:0:99999:7:::
    systemd-bus-proxy:!!:17332::::::
    systemd-network:!!:17332::::::
    dbus:!!:17332::::::
    polkitd:!!:17332::::::
    abrt:!!:17332::::::
    unbound:!!:17332::::::
    usbmuxd:!!:17332::::::
    tss:!!:17332::::::
    apache:!!:17332::::::
    libstoragemgmt:!!:17332::::::
    rpc:!!:17332:0:99999:7:::
    colord:!!:17332::::::
    pcp:!!:17332::::::
    saslauth:!!:17332::::::
    geoclue:!!:17332::::::
    setroubleshoot:!!:17332::::::
    rtkit:!!:17332::::::
    qemu:!!:17332::::::
    radvd:!!:17332::::::
    chrony:!!:17332::::::
    ntp:!!:17332::::::
    sssd:!!:17332::::::
    rpcuser:!!:17332::::::
    nfsnobody:!!:17332::::::
    pulse:!!:17332::::::
    gdm:!!:17332::::::
    gnome-initial-setup:!!:17332::::::
    avahi:!!:17332::::::
    postfix:!!:17332::::::
    sshd:!!:17332::::::
    tcpdump:!!:17332::::::
    oprofile:!!:17332::::::
    bolo:$6$.NPfOPh0dQ6pE6R9$I4M1jgxQFv6xJ0hT0/vGPwt9.jr5rFcY5A8t65Ju0VHe82CVP77KJjEZuccYd5zjq9R27eF.ksmvvQF1YR/66/::0:99999:7:::
    vboxadd:!!:17332::::::
    hadoop:$6$SEdaRKSa$qF8csIu103H.0JOIjeCZJrFkHYg7K1LBZ9AOEOmnPogo.bCJUkPIQxF5zI/pGCE6Pd4xyrJX0/Wf3zmCsH/Z3.:17384:0:99999:7:::
    dockerroot:!!:17534::::::
    etcd:!!:17535::::::
    kibana:!!:17571::::::
    elasticsearch:!!:17571::::::
    grafana:!!:17573::::::
    influxdb:!!:17575::::::
    telegraf:!!:17575::::::
    chronograf:!!:17576::::::
    kapacitor:!!:17577::::::
[root@h209 ~]#
~~~

致此，已经可以成功在远程执行命令了

批量管理的基础就是远程执行操作

使用此类工具，可以节省很多无意义的时间开销

---

# 总结

使用脚本安装，还是很简单的，这里重在理解 master 和 minion 的通讯机制

* TOC
{:toc}


---


[saltstack]:https://saltstack.com/
[saltstack_install]:https://docs.saltstack.com/en/getstarted/fundamentals/install.html
[saltstack_components]:https://docs.saltstack.com/en/getstarted/overview.html
[saltstack_tutorial]:https://docs.saltstack.com/en/getstarted/system/index.html
