---
layout: post
title: "Install OpenResty"
author:  wilmosfang
date: 2018-03-18 14:20:54
image: '/assets/img/'
excerpt: '安装 OpenResty'
main-class: nginx
color: '#00943d'
tags:
 - nginx
 - openresty
 - ab
categories:
 - nginx
twitter_text: 'simple process of OpenResty installation'
introduction: 'installation of OpenResty'
---


## 前言

**[OpenResty][openresty]** 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项

用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关

**[OpenResty][openresty]** 的目标是让Web服务直接跑在 Nginx 服务内部，充分利用 Nginx 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求，甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应

OpenResty 团队主要的贡献在于自主开发了一系列模块，与标准的 Nginx 集成，从而将 Nginx 有效地变成一个强大的通用 Web 应用平台，OpenResty 并非 Nginx 的一个分支，而是标准 Nginx 加上一组模块的集合

这里分享一下 **[OpenResty][openresty]** 的安装方法

参考 **[Getting Started][openresty_install]**

> **Tip:** 当前的版本为 **Version 1.13.6.1** 发布于 **13 November 2017**

---

# 操作


## 环境

~~~
[root@56-201 ~]# hostnamectl 
   Static hostname: 56-201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 1ec9480da2544ea78f153ff176e46736
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
       valid_lft 82832sec preferred_lft 82832sec
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
[root@56-201 ~]# 
~~~



## 构建仓库

~~~
[root@56-201 ~]# rpm -qa | grep yum-utils
yum-utils-1.1.31-40.el7.noarch
[root@56-201 ~]# yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://openresty.org/package/centos/openresty.repo
grabbing file https://openresty.org/package/centos/openresty.repo to /etc/yum.repos.d/openresty.repo
repo saved to /etc/yum.repos.d/openresty.repo
[root@56-201 ~]# ll /etc/yum.repos.d/openresty.repo 
-rw-r--r-- 1 root root 267 5月  23 2017 /etc/yum.repos.d/openresty.repo
[root@56-201 ~]# cat  /etc/yum.repos.d/openresty.repo 
[openresty]
name=Official OpenResty Open Source Repository for CentOS
baseurl=https://openresty.org/package/centos/$releasever/$basearch
skip_if_unavailable=False
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://openresty.org/package/pubkey.gpg
enabled=1
enabled_metadata=1
[root@56-201 ~]# 
~~~


## 软件列表

~~~
[root@56-201 ~]# yum list all | grep openresty
openresty.x86_64                        1.13.6.1-1.el7.centos          openresty
openresty-asan.x86_64                   1.13.6.1-1.el7.centos          openresty
openresty-asan-debuginfo.x86_64         1.13.6.1-1.el7.centos          openresty
openresty-debug.x86_64                  1.13.6.1-1.el7.centos          openresty
openresty-debug-debuginfo.x86_64        1.13.6.1-1.el7.centos          openresty
openresty-debuginfo.x86_64              1.13.6.1-1.el7.centos          openresty
openresty-doc.noarch                    1.13.6.1-1.el7.centos          openresty
openresty-openssl.x86_64                1.0.2k-1.el7.centos            openresty
openresty-openssl-asan.x86_64           1.0.2k-2.el7.centos            openresty
openresty-openssl-asan-debuginfo.x86_64 1.0.2k-2.el7.centos            openresty
openresty-openssl-asan-devel.x86_64     1.0.2k-2.el7.centos            openresty
openresty-openssl-debug.x86_64          1.0.2k-2.el7.centos            openresty
openresty-openssl-debug-debuginfo.x86_64
                                        1.0.2k-2.el7.centos            openresty
openresty-openssl-debug-devel.x86_64    1.0.2k-2.el7.centos            openresty
openresty-openssl-debuginfo.x86_64      1.0.2k-1.el7.centos            openresty
openresty-openssl-devel.x86_64          1.0.2k-1.el7.centos            openresty
openresty-opm.noarch                    1.13.6.1-1.el7.centos          openresty
openresty-pcre.x86_64                   8.41-1.el7.centos              openresty
openresty-pcre-asan.x86_64              8.41-1.el7.centos              openresty
openresty-pcre-asan-debuginfo.x86_64    8.41-1.el7.centos              openresty
openresty-pcre-asan-devel.x86_64        8.41-1.el7.centos              openresty
openresty-pcre-debuginfo.x86_64         8.41-1.el7.centos              openresty
openresty-pcre-devel.x86_64             8.41-1.el7.centos              openresty
openresty-resty.noarch                  1.13.6.1-1.el7.centos          openresty
openresty-valgrind.x86_64               1.13.6.1-1.el7.centos          openresty
openresty-valgrind-debuginfo.x86_64     1.13.6.1-1.el7.centos          openresty
openresty-zlib.x86_64                   1.2.11-3.el7.centos            openresty
openresty-zlib-asan.x86_64              1.2.11-6.el7.centos            openresty
openresty-zlib-asan-debuginfo.x86_64    1.2.11-6.el7.centos            openresty
openresty-zlib-asan-devel.x86_64        1.2.11-6.el7.centos            openresty
openresty-zlib-debuginfo.x86_64         1.2.11-3.el7.centos            openresty
openresty-zlib-devel.x86_64             1.2.11-3.el7.centos            openresty
perl-Lemplate.noarch                    0.15-1.el7.centos              openresty
perl-Spiffy.noarch                      0.46-3.el7.centos              openresty
perl-Test-Base.noarch                   0.88-2.el7.centos              openresty
perl-Test-LongString.noarch             0.17-1.el7.centos              openresty
perl-Test-Nginx.noarch                  0.26-1.el7.centos              openresty
[root@56-201 ~]#
~~~


## 安装软件


~~~
[root@56-201 ~]# yum install openresty
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media: 
 * epel: mirror.vinahost.vn
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package openresty.x86_64 0:1.13.6.1-1.el7.centos will be installed
--> Processing Dependency: openresty-zlib >= 1.2.11-3 for package: openresty-1.13.6.1-1.el7.centos.x86_64
--> Processing Dependency: openresty-pcre >= 8.40-1 for package: openresty-1.13.6.1-1.el7.centos.x86_64
--> Processing Dependency: openresty-openssl >= 1.0.2k-1 for package: openresty-1.13.6.1-1.el7.centos.x86_64
--> Running transaction check
---> Package openresty-openssl.x86_64 0:1.0.2k-1.el7.centos will be installed
---> Package openresty-pcre.x86_64 0:8.41-1.el7.centos will be installed
---> Package openresty-zlib.x86_64 0:1.2.11-3.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================
 Package                                                      Arch                                              Version                                                            Repository                                            Size
==============================================================================================================================================================================================================================================
Installing:
 openresty                                                    x86_64                                            1.13.6.1-1.el7.centos                                              openresty                                            1.1 M
Installing for dependencies:
 openresty-openssl                                            x86_64                                            1.0.2k-1.el7.centos                                                openresty                                            1.3 M
 openresty-pcre                                               x86_64                                            8.41-1.el7.centos                                                  openresty                                            154 k
 openresty-zlib                                               x86_64                                            1.2.11-3.el7.centos                                                openresty                                             54 k

Transaction Summary
==============================================================================================================================================================================================================================================
Install  1 Package (+3 Dependent packages)

Total download size: 2.6 M
Installed size: 7.4 M
Is this ok [y/d/N]: y
Downloading packages:
(1/4): openresty-openssl-1.0.2k-1.el7.centos.x86_64.rpm                                                                                                                                                                | 1.3 MB  00:00:01     
(2/4): openresty-pcre-8.41-1.el7.centos.x86_64.rpm                                                                                                                                                                     | 154 kB  00:00:00     
(3/4): openresty-1.13.6.1-1.el7.centos.x86_64.rpm                                                                                                                                                                      | 1.1 MB  00:00:01     
(4/4): openresty-zlib-1.2.11-3.el7.centos.x86_64.rpm                                                                                                                                                                   |  54 kB  00:00:00     
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                         1.7 MB/s | 2.6 MB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : openresty-zlib-1.2.11-3.el7.centos.x86_64                                                                                                                                                                                  1/4 
  Installing : openresty-openssl-1.0.2k-1.el7.centos.x86_64                                                                                                                                                                               2/4 
  Installing : openresty-pcre-8.41-1.el7.centos.x86_64                                                                                                                                                                                    3/4 
  Installing : openresty-1.13.6.1-1.el7.centos.x86_64                                                                                                                                                                                     4/4 
  Verifying  : openresty-1.13.6.1-1.el7.centos.x86_64                                                                                                                                                                                     1/4 
  Verifying  : openresty-pcre-8.41-1.el7.centos.x86_64                                                                                                                                                                                    2/4 
  Verifying  : openresty-zlib-1.2.11-3.el7.centos.x86_64                                                                                                                                                                                  3/4 
  Verifying  : openresty-openssl-1.0.2k-1.el7.centos.x86_64                                                                                                                                                                               4/4 

Installed:
  openresty.x86_64 0:1.13.6.1-1.el7.centos                                                                                                                                                                                                    

Dependency Installed:
  openresty-openssl.x86_64 0:1.0.2k-1.el7.centos                                   openresty-pcre.x86_64 0:8.41-1.el7.centos                                   openresty-zlib.x86_64 0:1.2.11-3.el7.centos                                  

Complete!
[root@56-201 ~]# 
~~~


查看一下 **openresty** 所包含的内容

~~~
[root@56-201 test]# rpm -ql openresty
/etc/init.d/openresty
/usr/bin/openresty
/usr/local/openresty/COPYRIGHT
/usr/local/openresty/bin/openresty
/usr/local/openresty/luajit/bin
/usr/local/openresty/luajit/bin/luajit
/usr/local/openresty/luajit/bin/luajit-2.1.0-beta3
/usr/local/openresty/luajit/include
/usr/local/openresty/luajit/include/luajit-2.1
/usr/local/openresty/luajit/include/luajit-2.1/lauxlib.h
/usr/local/openresty/luajit/include/luajit-2.1/lua.h
/usr/local/openresty/luajit/include/luajit-2.1/lua.hpp
/usr/local/openresty/luajit/include/luajit-2.1/luaconf.h
/usr/local/openresty/luajit/include/luajit-2.1/luajit.h
/usr/local/openresty/luajit/include/luajit-2.1/lualib.h
/usr/local/openresty/luajit/lib
/usr/local/openresty/luajit/lib/libluajit-5.1.so
/usr/local/openresty/luajit/lib/libluajit-5.1.so.2
/usr/local/openresty/luajit/lib/libluajit-5.1.so.2.1.0
/usr/local/openresty/luajit/lib/lua
/usr/local/openresty/luajit/lib/lua/5.1
/usr/local/openresty/luajit/lib/pkgconfig
/usr/local/openresty/luajit/lib/pkgconfig/luajit.pc
/usr/local/openresty/luajit/share
/usr/local/openresty/luajit/share/lua
/usr/local/openresty/luajit/share/lua/5.1
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/bc.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/bcsave.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_arm.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_arm64.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_arm64be.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_mips.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_mips64.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_mips64el.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_mipsel.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_ppc.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_x64.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dis_x86.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/dump.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/p.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/v.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/vmdef.lua
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/jit/zone.lua
/usr/local/openresty/lualib/cjson.so
/usr/local/openresty/lualib/ngx
/usr/local/openresty/lualib/ngx/balancer.lua
/usr/local/openresty/lualib/ngx/errlog.lua
/usr/local/openresty/lualib/ngx/ocsp.lua
/usr/local/openresty/lualib/ngx/process.lua
/usr/local/openresty/lualib/ngx/re.lua
/usr/local/openresty/lualib/ngx/semaphore.lua
/usr/local/openresty/lualib/ngx/ssl
/usr/local/openresty/lualib/ngx/ssl.lua
/usr/local/openresty/lualib/ngx/ssl/session.lua
/usr/local/openresty/lualib/redis
/usr/local/openresty/lualib/redis/parser.so
/usr/local/openresty/lualib/resty
/usr/local/openresty/lualib/resty/aes.lua
/usr/local/openresty/lualib/resty/core
/usr/local/openresty/lualib/resty/core.lua
/usr/local/openresty/lualib/resty/core/base.lua
/usr/local/openresty/lualib/resty/core/base64.lua
/usr/local/openresty/lualib/resty/core/ctx.lua
/usr/local/openresty/lualib/resty/core/exit.lua
/usr/local/openresty/lualib/resty/core/hash.lua
/usr/local/openresty/lualib/resty/core/misc.lua
/usr/local/openresty/lualib/resty/core/regex.lua
/usr/local/openresty/lualib/resty/core/request.lua
/usr/local/openresty/lualib/resty/core/response.lua
/usr/local/openresty/lualib/resty/core/shdict.lua
/usr/local/openresty/lualib/resty/core/time.lua
/usr/local/openresty/lualib/resty/core/uri.lua
/usr/local/openresty/lualib/resty/core/var.lua
/usr/local/openresty/lualib/resty/core/worker.lua
/usr/local/openresty/lualib/resty/dns
/usr/local/openresty/lualib/resty/dns/resolver.lua
/usr/local/openresty/lualib/resty/limit
/usr/local/openresty/lualib/resty/limit/conn.lua
/usr/local/openresty/lualib/resty/limit/count.lua
/usr/local/openresty/lualib/resty/limit/req.lua
/usr/local/openresty/lualib/resty/limit/traffic.lua
/usr/local/openresty/lualib/resty/lock.lua
/usr/local/openresty/lualib/resty/lrucache
/usr/local/openresty/lualib/resty/lrucache.lua
/usr/local/openresty/lualib/resty/lrucache/pureffi.lua
/usr/local/openresty/lualib/resty/md5.lua
/usr/local/openresty/lualib/resty/memcached.lua
/usr/local/openresty/lualib/resty/mysql.lua
/usr/local/openresty/lualib/resty/random.lua
/usr/local/openresty/lualib/resty/redis.lua
/usr/local/openresty/lualib/resty/sha.lua
/usr/local/openresty/lualib/resty/sha1.lua
/usr/local/openresty/lualib/resty/sha224.lua
/usr/local/openresty/lualib/resty/sha256.lua
/usr/local/openresty/lualib/resty/sha384.lua
/usr/local/openresty/lualib/resty/sha512.lua
/usr/local/openresty/lualib/resty/string.lua
/usr/local/openresty/lualib/resty/upload.lua
/usr/local/openresty/lualib/resty/upstream
/usr/local/openresty/lualib/resty/upstream/healthcheck.lua
/usr/local/openresty/lualib/resty/websocket
/usr/local/openresty/lualib/resty/websocket/client.lua
/usr/local/openresty/lualib/resty/websocket/protocol.lua
/usr/local/openresty/lualib/resty/websocket/server.lua
/usr/local/openresty/nginx/conf/fastcgi.conf
/usr/local/openresty/nginx/conf/fastcgi.conf.default
/usr/local/openresty/nginx/conf/fastcgi_params
/usr/local/openresty/nginx/conf/fastcgi_params.default
/usr/local/openresty/nginx/conf/koi-utf
/usr/local/openresty/nginx/conf/koi-win
/usr/local/openresty/nginx/conf/mime.types
/usr/local/openresty/nginx/conf/mime.types.default
/usr/local/openresty/nginx/conf/nginx.conf
/usr/local/openresty/nginx/conf/nginx.conf.default
/usr/local/openresty/nginx/conf/scgi_params
/usr/local/openresty/nginx/conf/scgi_params.default
/usr/local/openresty/nginx/conf/uwsgi_params
/usr/local/openresty/nginx/conf/uwsgi_params.default
/usr/local/openresty/nginx/conf/win-utf
/usr/local/openresty/nginx/html/50x.html
/usr/local/openresty/nginx/html/index.html
/usr/local/openresty/nginx/logs
/usr/local/openresty/nginx/sbin/nginx
/usr/local/openresty/nginx/sbin/stap-nginx
/usr/local/openresty/nginx/tapset/nginx.stp
/usr/local/openresty/nginx/tapset/ngx_lua.stp
/usr/local/openresty/site/lualib
[root@56-201 test]# 
~~~


## 目录结构

~~~
[root@56-201 ~]# tree /usr/local/openresty/
/usr/local/openresty/
├── bin
│   └── openresty -> /usr/local/openresty/nginx/sbin/nginx
├── COPYRIGHT
├── luajit
│   ├── bin
│   │   ├── luajit -> luajit-2.1.0-beta3
│   │   └── luajit-2.1.0-beta3
│   ├── include
│   │   └── luajit-2.1
│   │       ├── lauxlib.h
│   │       ├── luaconf.h
│   │       ├── lua.h
│   │       ├── lua.hpp
│   │       ├── luajit.h
│   │       └── lualib.h
│   ├── lib
│   │   ├── libluajit-5.1.so -> libluajit-5.1.so.2.1.0
│   │   ├── libluajit-5.1.so.2 -> libluajit-5.1.so.2.1.0
│   │   ├── libluajit-5.1.so.2.1.0
│   │   ├── lua
│   │   │   └── 5.1
│   │   └── pkgconfig
│   │       └── luajit.pc
│   └── share
│       ├── lua
│       │   └── 5.1
│       └── luajit-2.1.0-beta3
│           └── jit
│               ├── bc.lua
│               ├── bcsave.lua
│               ├── dis_arm64be.lua
│               ├── dis_arm64.lua
│               ├── dis_arm.lua
│               ├── dis_mips64el.lua
│               ├── dis_mips64.lua
│               ├── dis_mipsel.lua
│               ├── dis_mips.lua
│               ├── dis_ppc.lua
│               ├── dis_x64.lua
│               ├── dis_x86.lua
│               ├── dump.lua
│               ├── p.lua
│               ├── v.lua
│               ├── vmdef.lua
│               └── zone.lua
├── lualib
│   ├── cjson.so
│   ├── ngx
│   │   ├── balancer.lua
│   │   ├── errlog.lua
│   │   ├── ocsp.lua
│   │   ├── process.lua
│   │   ├── re.lua
│   │   ├── semaphore.lua
│   │   ├── ssl
│   │   │   └── session.lua
│   │   └── ssl.lua
│   ├── redis
│   │   └── parser.so
│   └── resty
│       ├── aes.lua
│       ├── core
│       │   ├── base64.lua
│       │   ├── base.lua
│       │   ├── ctx.lua
│       │   ├── exit.lua
│       │   ├── hash.lua
│       │   ├── misc.lua
│       │   ├── regex.lua
│       │   ├── request.lua
│       │   ├── response.lua
│       │   ├── shdict.lua
│       │   ├── time.lua
│       │   ├── uri.lua
│       │   ├── var.lua
│       │   └── worker.lua
│       ├── core.lua
│       ├── dns
│       │   └── resolver.lua
│       ├── limit
│       │   ├── conn.lua
│       │   ├── count.lua
│       │   ├── req.lua
│       │   └── traffic.lua
│       ├── lock.lua
│       ├── lrucache
│       │   └── pureffi.lua
│       ├── lrucache.lua
│       ├── md5.lua
│       ├── memcached.lua
│       ├── mysql.lua
│       ├── random.lua
│       ├── redis.lua
│       ├── sha1.lua
│       ├── sha224.lua
│       ├── sha256.lua
│       ├── sha384.lua
│       ├── sha512.lua
│       ├── sha.lua
│       ├── string.lua
│       ├── upload.lua
│       ├── upstream
│       │   └── healthcheck.lua
│       └── websocket
│           ├── client.lua
│           ├── protocol.lua
│           └── server.lua
├── nginx
│   ├── conf
│   │   ├── fastcgi.conf
│   │   ├── fastcgi.conf.default
│   │   ├── fastcgi_params
│   │   ├── fastcgi_params.default
│   │   ├── koi-utf
│   │   ├── koi-win
│   │   ├── mime.types
│   │   ├── mime.types.default
│   │   ├── nginx.conf
│   │   ├── nginx.conf.default
│   │   ├── scgi_params
│   │   ├── scgi_params.default
│   │   ├── uwsgi_params
│   │   ├── uwsgi_params.default
│   │   └── win-utf
│   ├── html
│   │   ├── 50x.html
│   │   └── index.html
│   ├── logs
│   ├── sbin
│   │   ├── nginx
│   │   └── stap-nginx
│   └── tapset
│       ├── nginx.stp
│       └── ngx_lua.stp
├── openssl
│   ├── bin
│   │   └── openssl
│   ├── lib
│   │   ├── engines
│   │   │   ├── lib4758cca.so
│   │   │   ├── libaep.so
│   │   │   ├── libatalla.so
│   │   │   ├── libcapi.so
│   │   │   ├── libchil.so
│   │   │   ├── libcswift.so
│   │   │   ├── libgmp.so
│   │   │   ├── libgost.so
│   │   │   ├── libnuron.so
│   │   │   ├── libpadlock.so
│   │   │   ├── libsureware.so
│   │   │   └── libubsec.so
│   │   ├── libcrypto.so -> libcrypto.so.1.0.0
│   │   ├── libcrypto.so.1.0.0
│   │   ├── libssl.so -> libssl.so.1.0.0
│   │   └── libssl.so.1.0.0
│   └── openssl.cnf
├── pcre
│   └── lib
│       ├── libpcre.so -> libpcre.so.1.2.9
│       ├── libpcre.so.1 -> libpcre.so.1.2.9
│       └── libpcre.so.1.2.9
├── site
│   └── lualib
└── zlib
    └── lib
        ├── libz.so -> libz.so.1.2.11
        ├── libz.so.1 -> libz.so.1.2.11
        └── libz.so.1.2.11

41 directories, 127 files
[root@56-201 ~]# 
~~~


## 创建项目

~~~
[root@56-201 ~]# mkdir /tmp/test
[root@56-201 ~]# cd /tmp/test/
[root@56-201 test]# mkdir logs/ conf/
[root@56-201 test]# ll 
total 0
drwxr-xr-x 2 root root 6 3月  19 00:26 conf
drwxr-xr-x 2 root root 6 3月  19 00:26 logs
[root@56-201 test]# 
~~~


## 创建配置

~~~
[root@56-201 test]# vim conf/nginx.conf
[root@56-201 test]# cat conf/nginx.conf
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world !!! just for test</p>")
            ';
        }
    }
}
[root@56-201 test]# 
~~~

配置格式与 nginx 一样

## 运行

~~~
[root@56-201 ~]# PATH=/usr/local/openresty/nginx/sbin:$PATH
[root@56-201 ~]# export PATH
[root@56-201 ~]# echo $PATH
/usr/local/openresty/nginx/sbin:/usr/local/go/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/go/bin:/root/bin
[root@56-201 ~]# nginx -p /tmp/test/ -c /tmp/test/conf/nginx.conf 
[root@56-201 ~]# ps faux | grep nginx 
root      4595  0.0  0.0 112648  1016 pts/0    S+   00:44   0:00          \_ grep --color=auto nginx
root      4592  0.0  0.0  36564  1160 ?        Ss   00:44   0:00 nginx: master process nginx -p /tmp/test/ -c /tmp/test/conf/nginx.conf
nobody    4593  0.0  0.0  39028  1972 ?        S    00:44   0:00  \_ nginx: worker process
[root@56-201 ~]# netstat  -ant 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:873             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25151         0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.201:22       192.168.56.1:46212      ESTABLISHED
tcp6       0      0 :::873                  :::*                    LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
tcp6       0      0 :::443                  :::*                    LISTEN     
[root@56-201 ~]# 
~~~


## 访问

~~~
[root@56-201 ~]# curl http://localhost:8080
<p>hello, world !!! just for test</p>
[root@56-201 ~]# 
~~~

与预期一致


## 日志　

~~~
[root@56-201 ~]# tree /tmp/test/logs/
/tmp/test/logs/
├── access.log
├── error.log
└── nginx.pid

0 directories, 3 files
[root@56-201 ~]# cat  /tmp/test/logs/access.log
127.0.0.1 - - [19/Mar/2018:00:44:53 +0800] "GET / HTTP/1.1" 200 49 "-" "curl/7.29.0"
[root@56-201 ~]# cat  /tmp/test/logs/error.log
[root@56-201 ~]# cat  /tmp/test/logs/nginx.pid 
4592
[root@56-201 ~]# 
~~~



## 性能测试

使用 **ab** 来进行静态页面的访问测试

~~~
[root@56-201 ~]# ab -c10 -n50000 http://localhost:8080/ 
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 5000 requests
Completed 10000 requests
Completed 15000 requests
Completed 20000 requests
Completed 25000 requests
Completed 30000 requests
Completed 35000 requests
Completed 40000 requests
Completed 45000 requests
Completed 50000 requests
Finished 50000 requests


Server Software:        openresty/1.13.6.1
Server Hostname:        localhost
Server Port:            8080

Document Path:          /
Document Length:        38 bytes

Concurrency Level:      10
Time taken for tests:   2.235 seconds
Complete requests:      50000
Failed requests:        0
Write errors:           0
Total transferred:      9300000 bytes
HTML transferred:       1900000 bytes
Requests per second:    22366.55 [#/sec] (mean)
Time per request:       0.447 [ms] (mean)
Time per request:       0.045 [ms] (mean, across all concurrent requests)
Transfer rate:          4062.67 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       2
Processing:     0    0   0.1      0       2
Waiting:        0    0   0.1      0       2
Total:          0    0   0.1      0       3

Percentage of the requests served within a certain time (ms)
  50%      0
  66%      0
  75%      0
  80%      0
  90%      0
  95%      1
  98%      1
  99%      1
 100%      3 (longest request)
[root@56-201 ~]# 
~~~

这是在本地笔记本的 VM 中进行的测试

页面内容有点简单，从结果来看，这个速度已经是很多其它框架或 web 容器无法企及的程度了，为进一步压榨机器的效率提供了可能

(当前这个效率的来源是 nginx 的事件触发异步非阻塞架构的结果，而 OpenResty 想做的就是充分使用这个架构的效能来服务于更为复杂的应用逻辑场景)

到此为止 OpenResty 的安装就已经完成了


---

# 总结

虽然是典型的 rpm 包安装，也有现成的 yum 库，整个过程十分简单

* TOC
{:toc}


---


[openresty]:https://openresty.org/en/
[openresty_install]:https://openresty.org/cn/getting-started.html
