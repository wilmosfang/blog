---
layout: post
title:  Elasticsearch 监控
author: wilmosfang
tags:  nosql elasticsearch nginx network monitoring
categories:  elasticsearch
wc: 464 1516 15274
excerpt:  elasticsearch 健康监控方法，融合nginx,ssl,auth_basic,plugins,DNAT等相关技术
comments: true
---



# 前言

由于elasticsearch有plugins机制，监控elasticsearch只需要安装一个head插件

但在生产环境下还有安全方面的考虑，只能让有管理权限的人员查看，所以需要加入访问控制，下面分享一下elasticsearch监控涉及的操作及方法


> **Tip:** 当前的版本为 **Elasticsearch-1.2.2** 、 **Tengine-2.1.2** 

---



# 概要

* TOC
{:toc}



---

## 安装插件


~~~
[root@es_node ~]# /data/ES/bin/plugin install mobz/elasticsearch-head
-> Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip...
Downloading ....................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................DONE
Installed mobz/elasticsearch-head into /data/ES/plugins/head
[root@es_node ~]# /data/ES/bin/plugin -l
Installed plugins:
    - head
    - analysis-mmseg
[root@es_node ~]# ll /data/ES/plugins/
total 8
drwxr-xr-x 2 bhuser bhuser 4096 Jul 10  2014 analysis-mmseg
drwxr-xr-x 5 root   root   4096 Jan  6 19:22 head
[root@es_node ~]# chown -R bhuser.bhuser /data/ES/plugins/head/
[root@es_node ~]# ll /data/ES/plugins/
total 8
drwxr-xr-x 2 bhuser bhuser 4096 Jul 10  2014 analysis-mmseg
drwxr-xr-x 5 bhuser bhuser 4096 Jan  6 19:22 head
[root@es_node ~]#
~~~

---

## 安装web服务器

这里使用 **Tengine-2.1.2** 是淘宝在 nginx 基础上打包的开源web服务器，详细可以参阅 **[Tengine][tengine]**

### 依赖包

安装之前要确保以下软件包已经安装

~~~
gcc.x86_64
pcre.x86_64   
pcre-devel.x86_64  
zlib.x86_64  
zlib-devel.x86_64 
openssl.x86_64 
openssl-devel.x86_64
~~~

### 下载Tengine

~~~
[root@es_node src]# wget http://tengine.taobao.org/download/tengine-2.1.2.tar.gz
--2016-01-06 19:30:59--  http://tengine.taobao.org/download/tengine-2.1.2.tar.gz
Resolving tengine.taobao.org... 120.55.149.135
Connecting to tengine.taobao.org|120.55.149.135|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2137295 (2.0M) [application/octet-stream]
Saving to: “tengine-2.1.2.tar.gz”

100%[===============================================================================>] 2,137,295   1.02M/s   in 2.0s    

2016-01-06 19:31:01 (1.02 MB/s) - “tengine-2.1.2.tar.gz” saved [2137295/2137295]

[root@es_node src]# md5sum tengine-2.1.2.tar.gz 
7f898a0dbb5162ff1eb19aeb9d53bec3  tengine-2.1.2.tar.gz
[root@es_node src]# 
~~~

### 解压编译与安装

~~~
[root@es_node src]# tar -zxvf tengine-2.1.2.tar.gz 
tengine-2.1.2/
tengine-2.1.2/good_configure
tengine-2.1.2/configure
tengine-2.1.2/docs/
tengine-2.1.2/docs/modules/
...
...
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/check_interface.t
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/tcp_check.t
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/ssl_hello_check.t
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/http_check.t
[root@es_node src]# 
[root@es_node src]# cd tengine-2.1.2
[root@es_node tengine-2.1.2]# ./configure 
checking for OS
 + Linux 2.6.32-573.12.1.el6.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC) 
checking for gcc -pipe switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
...
...
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

[root@es_node tengine-2.1.2]# echo $?
0
[root@es_node tengine-2.1.2]# make
make -f objs/Makefile
make[1]: Entering directory `/usr/local/src/tengine-2.1.2'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
		-o objs/src/core/nginx.o \
		src/core/nginx.c
...
...
	objs/src/http/modules/ngx_http_upstream_dynamic_module.o \
	objs/src/http/modules/ngx_http_stub_status_module.o \
	objs/ngx_modules.o \
	-lpthread -ldl -lcrypt -lpcre -lssl -lcrypto -ldl -lz
make[1]: Leaving directory `/usr/local/src/tengine-2.1.2'
make -f objs/Makefile manpage
make[1]: Entering directory `/usr/local/src/tengine-2.1.2'
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
		-e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
		-e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
		-e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
		< man/nginx.8 > objs/nginx.8
make[1]: Leaving directory `/usr/local/src/tengine-2.1.2'
[root@es_node tengine-2.1.2]# echo $?
0 
[root@es_node tengine-2.1.2]# make install 
make -f objs/Makefile install
make[1]: Entering directory `/usr/local/src/tengine-2.1.2'
test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'
test -d '/usr/local/nginx/sbin' 		|| mkdir -p '/usr/local/nginx/sbin'
test ! -f '/usr/local/nginx/sbin/nginx' 		|| mv '/usr/local/nginx/sbin/nginx' 			'/usr/local/nginx/sbin/nginx.old'
...
...
test -f 'src/http/modules/ngx_http_reqstat.h' && cp 'src/http/modules/ngx_http_reqstat.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_headers.h'  && cp 'objs/ngx_auto_headers.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_config.h' && cp 'objs/ngx_auto_config.h' '/usr/local/nginx/include'
make[1]: Leaving directory `/usr/local/src/tengine-2.1.2'
[root@es_node tengine-2.1.2]# echo $?
0
[root@es_node tengine-2.1.2]# ll /usr/local/nginx/
total 24
drwxr-xr-x 2 root root 4096 Jan  6 19:46 conf
drwxr-xr-x 2 root root 4096 Jan  6 19:46 html
drwxr-xr-x 2 root root 4096 Jan  6 19:46 include
drwxr-xr-x 2 root root 4096 Jan  6 19:46 logs
drwxr-xr-x 2 root root 4096 Jan  6 19:46 modules
drwxr-xr-x 2 root root 4096 Jan  6 19:46 sbin
[root@es_node tengine-2.1.2]# 
~~~

---

## 创建自签名证书

~~~
[root@es_node tengine-2.1.2]# cd /usr/local/nginx/
[root@es_node nginx]# ls
conf  html  include  logs  modules  sbin
[root@es_node nginx]# mkdir cert
[root@es_node nginx]# cd cert/
[root@es_node cert]# openssl genrsa -out es.key 2048
Generating RSA private key, 2048 bit long modulus
...................+++
...........+++
e is 65537 (0x10001)
[root@es_node cert]# openssl req -new -key es.key -out es.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:Shanghai
Locality Name (eg, city) [Default City]:Shanghai
Organization Name (eg, company) [Default Company Ltd]:es
Organizational Unit Name (eg, section) []:es
Common Name (eg, your name or your server's hostname) []:es
Email Address []:ok@es.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
[root@es_node cert]# openssl x509 -req -days 365 -in es.csr -signkey es.key -out es.crt
Signature ok
subject=/C=CN/ST=Shanghai/L=Shanghai/O=es/OU=es/CN=es/emailAddress=ok@es.com
Getting Private key
[root@es_node cert]# ll
total 12
-rw-r--r-- 1 root root 1249 Jan  6 19:54 es.crt
-rw-r--r-- 1 root root 1025 Jan  6 19:53 es.csr
-rw-r--r-- 1 root root 1675 Jan  6 19:49 es.key
[root@es_node cert]# chmod 600 *
[root@es_node cert]# ll 
total 12
-rw------- 1 root root 1249 Jan  6 19:54 es.crt
-rw------- 1 root root 1025 Jan  6 19:53 es.csr
-rw------- 1 root root 1675 Jan  6 19:49 es.key
[root@es_node cert]#
~~~

---

## 创建nginx用户

~~~
[root@es_node cert]# useradd nginx 
[root@es_node cert]# grep nginx /etc/passwd
nginx:x:505:505::/home/nginx:/bin/bash
[root@es_node cert]# chown -R nginx.nginx /usr/local/nginx/
[root@es_node cert]# ll 
total 12
-rw------- 1 nginx nginx 1249 Jan  6 19:54 es.crt
-rw------- 1 nginx nginx 1025 Jan  6 19:53 es.csr
-rw------- 1 nginx nginx 1675 Jan  6 19:49 es.key
[root@es_node cert]# 
~~~

---

## 生成认证密码

~~~
[nginx@es_node nginx]$ ll
total 28
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:54 cert
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:46 conf
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:46 html
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:46 include
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:46 logs
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:46 modules
drwxr-xr-x 2 nginx nginx 4096 Jan  6 19:46 sbin
[nginx@es_node nginx]$ mkdir pass
[nginx@es_node nginx]$ cd pass/
[nginx@es_node pass]$ perl -e 'print  crypt(espass,espass)'
esPl/R9yBFjpA[nginx@es_node pass]$ vim es.passwd
[nginx@es_node pass]$ cat es.passwd 
test:esPl/R9yBFjpA
[nginx@es_node pass]$
~~~

---

## 修改nginx配置

修改nginx配置文件

~~~
[root@es_node conf]# vim nginx.conf
[root@es_node conf]# grep -v "#" nginx.conf | grep -v "^$"
user  nginx;
worker_processes  1;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    upstream es {
        server 192.168.66.66:9200 ;
	}
    server {
        listen      443; 
        server_name  localhost;
        ssl on;
	ssl_certificate /usr/local/nginx/cert/es.crt;
	ssl_certificate_key /usr/local/nginx/cert/es.key;
        location / {
            root   html;
            index  index.html index.htm;
            auth_basic      "input your name and passsword";
	    auth_basic_user_file  /usr/local/nginx/pass/es.passwd;
	    allow all;
	    proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://es;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
[root@es_node conf]# cd ..
[root@es_node nginx]# sbin/nginx  -t -c conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@es_node nginx]# 
~~~

> **Tip:** 打开日志不是必要的，因为会浪费掉部分磁盘空间，也会影响一点性能，但是会给排查问题，行为跟踪带来方便，相对而言此类非应用型系统，打开日志带来的方便明显比产生的开销要更有价值


---

## 打开防火墙

~~~
[root@es_node nginx]# iptables -L -nv | grep 443
[root@es_node nginx]# vim /etc/sysconfig/iptables
[root@es_node nginx]# grep 443 /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
[root@es_node nginx]# /etc/init.d/iptables  reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@es_node nginx]# iptables -L -nv | grep 443
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:443 
[root@es_node nginx]# 
~~~

---

## 配置DNAT


作为边界的网关服务器，要打开内核转发和iptables转发

也就是 **`net.ipv4.ip_forward`** 和 **`filter` 表 `FORWARD` 链**

然后开启 **NAT** **PREROUTING** 链的 **DNAT**

~~~
[root@net_border ~]# iptables -L -nv  -t nat | grep 443
[root@net_border ~]# vim /etc/sysconfig/iptables
[root@net_border ~]# grep 443 /etc/sysconfig/iptables
-A PREROUTING -p tcp -m tcp --dport 2443 -j DNAT --to-destination 192.168.66.66:443
[root@net_border ~]# /etc/init.d/iptables reload
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@net_border ~]# iptables -L -nv  -t nat | grep 443
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:2443 to:192.168.66.66:443 
[root@net_border ~]# 
~~~

---

## 开启nginx服务


~~~
[root@es_node nginx]# sbin/nginx  -t -c conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@es_node nginx]# sbin/nginx  -c conf/nginx.conf
[root@es_node nginx]# ps faux | grep nginx 
root      4629  0.0  0.0 103308   832 pts/0    S+   22:18   0:00          \_ grep nginx
root      4623  0.0  0.0  46320  1172 ?        Ss   22:17   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
nginx     4624  0.0  0.0  46688  1836 ?        S    22:17   0:00  \_ nginx: worker process        
[root@es_node nginx]# 
~~~

## 进行访问

**`https://ip:2443/_plugin/head/`**

此ip为边界服务器的ip

因为这是使用的自签名证书，所以访问过程中会提示此证书不可信，是否继续，我们要选择继续

![cert.png](/images/es_mon/cert.png)

选择继续后，提示输入密码，就是前面设定的基本认证密码

![auth1.png](/images/es_mon/auth1.png)

![auth2.png](/images/es_mon/auth2.png)

Chrome也是一样

![auth3.png](/images/es_mon/auth3.png)

![auth4.png](/images/es_mon/auth4.png)

认证通过后就可以看到管理界面

![esview.png](/images/es_mon/esview.png)


---



# 命令汇总

* **`/data/ES/bin/plugin install mobz/elasticsearch-head`**
* **`/data/ES/bin/plugin -l`**
* **`ll /data/ES/plugins/`**
* **`chown -R bhuser.bhuser /data/ES/plugins/head/`**
* **`wget http://tengine.taobao.org/download/tengine-2.1.2.tar.gz`**
* **`md5sum tengine-2.1.2.tar.gz`**
* **`tar -zxvf tengine-2.1.2.tar.gz`**
* **`cd tengine-2.1.2`**
* **`./configure`**
* **`make`**
* **`make install`**
* **`mkdir cert`**
* **`cd cert/`**
* **`openssl genrsa -out es.key 2048`**
* **`openssl req -new -key es.key -out es.csr`**
* **`openssl x509 -req -days 365 -in es.csr -signkey es.key -out es.crt`**
* **`chmod 600 *`**
* **`useradd nginx`**
* **`grep nginx /etc/passwd`**
* **`chown -R nginx.nginx /usr/local/nginx/`**
* **`mkdir pass`**
* **`cd pass/`**
* **`perl -e 'print  crypt(espass,espass)'`**
* **`cat es.passwd`**
* **`vim nginx.conf`**
* **`grep -v "#" nginx.conf | grep -v "^$"`**
* **`iptables -L -nv | grep 443`**
* **`vim /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables  reload`**
* **`grep 443 /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv  -t nat | grep 443`**
* **`sbin/nginx  -t -c conf/nginx.conf`**
* **`sbin/nginx  -c conf/nginx.conf`**

---

[tengine]:http://tengine.taobao.org/index_cn.html

