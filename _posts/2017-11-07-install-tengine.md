---
layout: post
title: "Install Tengine"
author:  wilmosfang
date: 2017-11-07 15:41:10
image: '/assets/img/'
excerpt: Tengine 的安装方法
main-class: nginx
color: '#00943d'
tags:
 - tengine
 - nginx
categories:
 - nginx
twitter_text: Tengine install simple process
introduction: Tengine install method
---


# 前言


**[Tengine][tengine]** 是由淘宝网发起的Web服务器项目，它在 **[Nginx][nginx]** 的基础上，针对大访问量网站的需求，添加了很多高级功能和特性

值得一提的是直接集成了很多实用模块，给管理与监控带来了很大便利

下面分享一下 **[Tengine][tengine]** 的基础操作

> **Tip:** 当前版本 **Tengine-2.2.1**

---

# 操作

## 下载源码包


~~~bash
[root@much tmp]# wget http://tengine.taobao.org/download/tengine-2.2.1.tar.gz
--2017-11-08 00:11:27--  http://tengine.taobao.org/download/tengine-2.2.1.tar.gz
Resolving tengine.taobao.org (tengine.taobao.org)... 120.55.149.135
Connecting to tengine.taobao.org (tengine.taobao.org)|120.55.149.135|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2114640 (2.0M) [application/octet-stream]
Saving to: ‘tengine-2.2.1.tar.gz’

100%[======================================>] 2,114,640   55.4KB/s   in 36s    

2017-11-08 00:12:08 (56.6 KB/s) - ‘tengine-2.2.1.tar.gz’ saved [2114640/2114640]

[root@much tmp]#
~~~

## 检验源码包


~~~bash
[root@much tmp]# md5sum  tengine-2.2.1.tar.gz
c283f55a34817836e380240287e8c57d  tengine-2.2.1.tar.gz
[root@much tmp]#
~~~


## 解压

~~~
[root@much tmp]# tar -zxvf tengine-2.2.1.tar.gz
tengine-2.2.1/
tengine-2.2.1/.gitignore
tengine-2.2.1/.travis.yml
tengine-2.2.1/AUTHORS.te
tengine-2.2.1/auto/
tengine-2.2.1/CHANGES
tengine-2.2.1/CHANGES.cn
tengine-2.2.1/CHANGES.ru
tengine-2.2.1/CHANGES.te
tengine-2.2.1/conf/
...
...
tengine-2.2.1/auto/cc/msvc
tengine-2.2.1/auto/cc/name
tengine-2.2.1/auto/cc/owc
tengine-2.2.1/auto/cc/sunc
[root@much tmp]#
~~~

## 配置

~~~
[root@much tmp]# cd tengine-2.2.1/
[root@much tengine-2.2.1]# ls
AUTHORS.te  CHANGES.ru  contrib  man       README.markdown
auto        CHANGES.te  docs     modules   src
CHANGES     conf        html     packages  tests
CHANGES.cn  configure   LICENSE  README    THANKS.te
[root@much tengine-2.2.1]# pwd
/tmp/tengine-2.2.1
[root@much tengine-2.2.1]# ./configure
checking for OS
 + Linux 3.10.0-514.21.1.el7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.8.5 20150623 (Red Hat 4.8.5-11) (GCC)
checking for gcc -pipe switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
...
...
checking for PCRE JIT support ... found
checking for OpenSSL library ... found
checking for zlib library ... found
creating objs/Makefile

Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + md5: using OpenSSL library
  + sha1: using OpenSSL library
  + using system zlib library
  + jemalloc library is disabled

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx dso module path: "/usr/local/nginx/modules/"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

[root@much tengine-2.2.1]# echo $?
0
[root@much tengine-2.2.1]#
~~~

## 编译

~~~
[root@much tengine-2.2.1]# pwd
/tmp/tengine-2.2.1
[root@much tengine-2.2.1]# ls
AUTHORS.te  CHANGES.ru  contrib  Makefile  packages         tests
auto        CHANGES.te  docs     man       README           THANKS.te
CHANGES     conf        html     modules   README.markdown
CHANGES.cn  configure   LICENSE  objs      src
[root@much tengine-2.2.1]# make
make -f objs/Makefile
make[1]: Entering directory `/tmp/tengine-2.2.1'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
	-o objs/src/core/nginx.o \
	src/core/nginx.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
	-o objs/src/core/ngx_log.o \
	src/core/ngx_log.c
...
...
objs/src/http/modules/ngx_http_reqstat_module.o \
objs/src/http/modules/ngx_http_upstream_keepalive_module.o \
objs/src/http/modules/ngx_http_upstream_dynamic_module.o \
objs/src/http/modules/ngx_http_stub_status_module.o \
objs/ngx_modules.o \
-lpthread -ldl -lcrypt -lpcre -lssl -lcrypto -ldl -lz
make[1]: Leaving directory `/tmp/tengine-2.2.1'
make -f objs/Makefile manpage
make[1]: Entering directory `/tmp/tengine-2.2.1'
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
	-e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
	-e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
	-e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
	< man/nginx.8 > objs/nginx.8
make[1]: Leaving directory `/tmp/tengine-2.2.1'
[root@much tengine-2.2.1]# echo $?
0
[root@much tengine-2.2.1]#
~~~

## 安装

~~~
[root@much tengine-2.2.1]# make install
make -f objs/Makefile install
make[1]: Entering directory `/tmp/tengine-2.2.1'
test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'
test -d '/usr/local/nginx/sbin' 		|| mkdir -p '/usr/local/nginx/sbin'
test ! -f '/usr/local/nginx/sbin/nginx' 		|| mv '/usr/local/nginx/sbin/nginx' 			'/usr/local/nginx/sbin/nginx.old'
cp objs/nginx '/usr/local/nginx/sbin/nginx'
test -d '/usr/local/nginx/conf' 		|| mkdir -p '/usr/local/nginx/conf'
cp conf/koi-win '/usr/local/nginx/conf'
cp conf/koi-utf '/usr/local/nginx/conf'
...
...
test -f 'objs/ngx_auto_headers.h'  && cp 'objs/ngx_auto_headers.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_config.h' && cp 'objs/ngx_auto_config.h' '/usr/local/nginx/include'
make[1]: Leaving directory `/tmp/tengine-2.2.1'
[root@much tengine-2.2.1]# echo $?
0
[root@much tengine-2.2.1]#
~~~

## 启动

~~~
[root@much tengine-2.2.1]# /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@much tengine-2.2.1]# /usr/local/nginx/sbin/nginx  -c /usr/local/nginx/conf/nginx.conf
[root@much tengine-2.2.1]#   
[root@much tengine-2.2.1]# netstat -antp | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      7039/nginx: master  
[root@much tengine-2.2.1]# ps faux | grep nginx
root      7058  0.0  0.0 112648  1016 pts/1    S+   00:34   0:00  |       \_ grep --color=auto nginx
root      7039  0.0  0.0  46996  1144 ?        Ss   00:33   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody    7040  0.0  0.0  49524  1980 ?        S    00:33   0:00  \_ nginx: worker process
[root@much tengine-2.2.1]#
~~~

## 访问

![tengine](/assets/img/nginx/tengine.png)

---

# 总结

总体来讲遵循一个典型的源码安装流程

* TOC
{:toc}


---

[tengine]:http://tengine.taobao.org/
[nginx]:https://www.nginx.com/
