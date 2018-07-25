---
layout: post
title: "Install SSR (ShadowsocksR)"
author:  wilmosfang
date: 2018-03-23 16:30:11
image: '/assets/img/'
excerpt: '安装 SSR (ShadowsocksR)'
main-class:  'tools'
color: '#808080'
tags:
 - ssr
 - ss
 - tools
 - pip
categories:
 - tools
twitter_text: 'ShadowsocksR install simple process'
introduction: 'installation method of ShadowsocksR'
---

# 前言

**[ShadowsocksR][shadowsocksr]** 是一款开源的用来穿透 **GFW** 的软件

**[ShadowsocksR][shadowsocksr]** 是 **SS(Shadowsocks)** 的一个分支，在 **SS(Shadowsocks)** 的基础上又添加了一些干扰识别(进行伪装)的特性，使其更具有穿透力

它是 **C/S** 架构的，需要安装服务端软件，然后通过客户端与之连接

成功连接后，其后的通讯就可以进行加密与伪装来穿透 **GFW**，达到科学上网的效果

>仅代表个人观点: 不允许自由了解外面的世界，对一些信息有意加以过滤与封锁，是否等价于给门窗糊上报纸，不让风景透过窗户，也不让自己成为外面人的风景，若干年后的人们再次回望这段历史的时候，会不会觉得愚昧和荒谬
>
>堤坝可以用来防洪，本有其积极的意义，但若借机建得太高，以至于阻断了自由交流，可能就成了思想的牢笼
>
>有一个人说过，可以永远欺骗一部分人或者暂时欺骗所有人，但却无法永远欺骗所有人

好在还有那么一小片镜子，可以透过一些微小的缝隙，反射出外面的光芒

(不见得外面的世界一定就代表着美丽，也可能充斥着混乱与丑恶，但关键是请让我们自己来作判断，让我们拥有这份自主选择的权利，而非直接剥夺)

废话了那么多，回归正题，这里演示一下如何构建 **[ShadowsocksR][shadowsocksr]** 服务的过程

> **Tip:** 当前的版本为 **Shadowsocks 2.8.2**

---

# 操作


## 环境

~~~
[root@ci ~]# hostnamectl 
   Static hostname: ci
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 8d13a50988cc5c4972347415eddf7d47
           Boot ID: 10b2ba6eee6941b78a7e0b2fa9c42e8c
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@ci ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:b7:37:f1 brd ff:ff:ff:ff:ff:ff
    inet 10.144.126.20/18 brd 10.144.127.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@ci ~]# 
~~~


## 安装 pip


~~~
[root@ci ~]# yum install python-setuptools && easy_install pip
Loaded plugins: fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
jenkins                                                  | 2.9 kB     00:00     
os                                                       | 3.6 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/5): epel/7/x86_64/updateinfo                            | 904 kB   00:00     
(2/5): extras/7/x86_64/primary_db                          | 181 kB   00:00     
(3/5): updates/7/x86_64/primary_db                         | 6.9 MB   00:00     
(4/5): epel/7/x86_64/primary_db                            | 6.3 MB   00:00     
(5/5): jenkins/primary_db                                  |  23 kB   00:00     
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package python-setuptools.noarch 0:0.9.8-7.el7 will be installed
--> Processing Dependency: python-backports-ssl_match_hostname for package: python-setuptools-0.9.8-7.el7.noarch
--> Running transaction check
---> Package python-backports-ssl_match_hostname.noarch 0:3.4.0.2-4.el7 will be installed
--> Processing Dependency: python-backports for package: python-backports-ssl_match_hostname-3.4.0.2-4.el7.noarch
--> Running transaction check
---> Package python-backports.x86_64 0:1.0-8.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                                Arch      Version           Repository
                                                                           Size
================================================================================
Installing:
 python-setuptools                      noarch    0.9.8-7.el7       os    397 k
Installing for dependencies:
 python-backports                       x86_64    1.0-8.el7         os    5.8 k
 python-backports-ssl_match_hostname    noarch    3.4.0.2-4.el7     os     12 k

Transaction Summary
================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 415 k
Installed size: 2.0 M
Is this ok [y/d/N]: y
Downloading packages:
(1/3): python-backports-1.0-8.el7.x86_64.rpm               | 5.8 kB   00:00     
(2/3): python-backports-ssl_match_hostname-3.4.0.2-4.el7.n |  12 kB   00:00     
(3/3): python-setuptools-0.9.8-7.el7.noarch.rpm            | 397 kB   00:00     
--------------------------------------------------------------------------------
Total                                              2.1 MB/s | 415 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python-backports-1.0-8.el7.x86_64                            1/3 
  Installing : python-backports-ssl_match_hostname-3.4.0.2-4.el7.noarch     2/3 
  Installing : python-setuptools-0.9.8-7.el7.noarch                         3/3 
  Verifying  : python-setuptools-0.9.8-7.el7.noarch                         1/3 
  Verifying  : python-backports-1.0-8.el7.x86_64                            2/3 
  Verifying  : python-backports-ssl_match_hostname-3.4.0.2-4.el7.noarch     3/3 

Installed:
  python-setuptools.noarch 0:0.9.8-7.el7                                        

Dependency Installed:
  python-backports.x86_64 0:1.0-8.el7                                           
  python-backports-ssl_match_hostname.noarch 0:3.4.0.2-4.el7                    

Complete!
Searching for pip
Reading https://pypi.python.org/simple/pip/
Best match: pip 9.0.3
Downloading https://pypi.python.org/packages/c4/44/e6b8056b6c8f2bfd1445cc9990f478930d8e3459e9dbf5b8e2d2922d64d3/pip-9.0.3.tar.gz#md5=b15b33f9aad61f88d0f8c866d16c55d8
Processing pip-9.0.3.tar.gz
Writing /tmp/easy_install-MeC501/pip-9.0.3/setup.cfg
Running pip-9.0.3/setup.py -q bdist_egg --dist-dir /tmp/easy_install-MeC501/pip-9.0.3/egg-dist-tmp-0Q5YaT
/usr/lib64/python2.7/distutils/dist.py:267: UserWarning: Unknown distribution option: 'python_requires'
  warnings.warn(msg)
warning: no previously-included files found matching '.coveragerc'
warning: no previously-included files found matching '.mailmap'
warning: no previously-included files found matching '.travis.yml'
warning: no previously-included files found matching '.landscape.yml'
warning: no previously-included files found matching 'pip/_vendor/Makefile'
warning: no previously-included files found matching 'tox.ini'
warning: no previously-included files found matching 'dev-requirements.txt'
warning: no previously-included files found matching 'appveyor.yml'
no previously-included directories found matching '.github'
no previously-included directories found matching '.travis'
no previously-included directories found matching 'docs/_build'
no previously-included directories found matching 'contrib'
no previously-included directories found matching 'tasks'
no previously-included directories found matching 'tests'
Adding pip 9.0.3 to easy-install.pth file
Installing pip script to /usr/bin
Installing pip2.7 script to /usr/bin
Installing pip2 script to /usr/bin

Installed /usr/lib/python2.7/site-packages/pip-9.0.3-py2.7.egg
Processing dependencies for pip
Finished processing dependencies for pip
[root@ci ~]# echo $?
0
[root@ci ~]# pip install shadowsocks
Collecting shadowsocks
  Downloading shadowsocks-2.8.2.tar.gz
Installing collected packages: shadowsocks
  Running setup.py install for shadowsocks ... done
Successfully installed shadowsocks-2.8.2
[root@ci ~]# echo $?
0
[root@ci ~]# 
~~~

## 安装 shadowsocks

~~~
[root@ci ~]# pip install shadowsocks
Collecting shadowsocks
  Downloading shadowsocks-2.8.2.tar.gz
Installing collected packages: shadowsocks
  Running setup.py install for shadowsocks ... done
Successfully installed shadowsocks-2.8.2
[root@ci ~]# echo $?
0
[root@ci ~]#
~~~

## 查看帮助

~~~
[root@ci ~]# ssserver --help 
usage: ssserver [OPTION]...
A fast tunnel proxy that helps you bypass firewalls.

You can supply configurations via either config file or command line arguments.

Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address, default: 0.0.0.0
  -p SERVER_PORT         server port, default: 8388
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
  -t TIMEOUT             timeout in seconds, default: 300
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+
  --workers WORKERS      number of workers, available on Unix/Linux
  --forbidden-ip IPLIST  comma seperated IP list forbidden to connect
  --manager-address ADDR optional server manager UDP address, see wiki

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information

Online help: <https://github.com/shadowsocks/shadowsocks>

[root@ci ~]# ssserver --version
Shadowsocks 2.8.2
[root@ci ~]#
~~~

## 运行服务

查看当前运行的端口

~~~
[root@ci ~]# netstat -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN     
tcp        0     36 10.144.126.20:443       119.94.94.14:44208      ESTABLISHED
tcp        0      0 10.144.126.20:443       119.94.94.14:43946      ESTABLISHED
tcp        0      0 10.144.126.20:41304     10.249.93.85:5574       ESTABLISHED
tcp     1051      0 10.144.126.20:8080      112.211.20.186:60782    CLOSE_WAIT 
tcp6       0      0 :::443                  :::*                    LISTEN     
[root@ci ~]# 
~~~

前台运行服务(日志会直接在终端中输出)

~~~
[root@ci ~]# ssserver -p 22 -k just_for_test -m aes-256-cfb
2018-03-24 01:18:04 INFO     loading libcrypto from libcrypto.so.10
2018-03-24 01:18:04 INFO     starting server at 0.0.0.0:22
...
...
...
~~~

在另外一个终端中查看端口

~~~
[root@ci ~]# ps faux | grep ssserver
root     13294  0.0  0.5 206840 10800 pts/0    S+   01:18   0:00  |       \_ /usr/bin/python /usr/bin/ssserver -p 22 -k just_for_test -m aes-256-cfb
root     13817  0.0  0.0 112652  1016 pts/1    S+   01:29   0:00          \_ grep --color=auto ssserver
[root@ci ~]# netstat  -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN     
tcp        0     36 10.144.126.20:443       119.94.94.14:44208      ESTABLISHED
tcp        0      0 10.144.126.20:443       119.94.94.14:43946      ESTABLISHED
tcp        0      0 10.144.126.20:41304     10.249.93.85:5574       ESTABLISHED
tcp     1051      0 10.144.126.20:8080      112.211.20.186:60782    CLOSE_WAIT 
tcp6       0      0 :::443                  :::*                    LISTEN     
[root@ci ~]# lsof -i :22
COMMAND    PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
ssserver 13294 root    3u  IPv4 27377492      0t0  TCP *:ssh (LISTEN)
ssserver 13294 root    4u  IPv4 27377493      0t0  UDP *:ssh 
[root@ci ~]#
~~~

服务在指定的端口进行了监听，并且同时有 TCP 和 UDP (正常的 ssh 是基于 TCP 的，不会监听在 UDP 上的)

虽然名称显示是 SSH，但其实是由 ssserver 命令生成的

到此 ssr 服务端已经搭建完成，并且也已经运行了


---

# 总结

使用 pip 来进行安装，十分便捷

ssserver 也就是一条命令，直接指定参数就可立刻运行，也很简单和直观 (它还可以通过指定 JSON 配置来后台运行)


* TOC
{:toc}


---



[shadowsocksr]:https://github.com/shadowsocksr-backup/shadowsocksr




