---
layout: default
title: Tengine基础
comments: true
---


---

#前言

**[Tengine][tengine]** 是由淘宝网发起的Web服务器项目，它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性

值得一提的是直接集成了很多实用模块，给管理与监控带来了很大便利

下面分享一下 **[Tengine][tengine]** 的基础操作，详细可以参阅 [官方文档][doc] 

> **Tip:** 当前版本 **Tengine 2.1.1**  

---

#概要

* TOC
{:toc}


---

##安装


###下载软件包


**[Tengine][tengine]** 的 **[下载地址][download]**

{% highlight bash %}
[root@i-1avyrt2d src]# wget  http://tengine.taobao.org/download/tengine-2.1.1.tar.gz 
--2015-11-04 13:24:28--  http://tengine.taobao.org/download/tengine-2.1.1.tar.gz
Resolving tengine.taobao.org... 120.55.149.135
Connecting to tengine.taobao.org|120.55.149.135|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2062650 (2.0M) [application/octet-stream]
Saving to: “tengine-2.1.1.tar.gz”

100%[===============================================================================>] 2,062,650   1.07M/s   in 1.8s    

2015-11-04 13:24:30 (1.07 MB/s) - “tengine-2.1.1.tar.gz” saved [2062650/2062650]

[root@i-1avyrt2d src]# md5sum  tengine-2.1.1.tar.gz 
357ec313735bce0b75fedd4662f6208c  tengine-2.1.1.tar.gz
[root@i-1avyrt2d src]# 
{% endhighlight %}

---


###安装软件包

####依赖包

下面是可能的依赖包

{% highlight bash %}
pcre.x86_64   
pcre-devel.x86_64  
zlib.x86_64  
zlib-devel.x86_64 
openssl.x86_64 
openssl-devel.x86_64
{% endhighlight %}

> **Note:** 如果缺少以上的包，在配置检查过程中就可能会报错 ， 可以使用 **yum** 


---

####解压

{% highlight bash %}
[root@i-1avyrt2d src]# ls
nginx-1.9.6  nginx-1.9.6.tar.gz  tengine-2.1.1.tar.gz
[root@i-1avyrt2d src]# tar -zxvf tengine-2.1.1.tar.gz 
tengine-2.1.1/
tengine-2.1.1/configure
tengine-2.1.1/docs/
tengine-2.1.1/docs/modules/
tengine-2.1.1/docs/modules/ngx_http_tfs_module_cn.md
tengine-2.1.1/docs/modules/ngx_http_upstream_session_sticky_module.md
tengine-2.1.1/docs/modules/ngx_http_sysguard.md
...
...
tengine-2.1.1/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/
tengine-2.1.1/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/check_interface.t
tengine-2.1.1/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/tcp_check.t
tengine-2.1.1/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/ssl_hello_check.t
tengine-2.1.1/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/http_check.t
[root@i-1avyrt2d src]# ls
nginx-1.9.6  nginx-1.9.6.tar.gz  tengine-2.1.1  tengine-2.1.1.tar.gz
[root@i-1avyrt2d src]# 
{% endhighlight %}

---

####配置

{% highlight bash %}
[root@i-1avyrt2d tengine-2.1.1]# ./configure 
checking for OS
 + Linux 2.6.32-573.7.1.el6.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC) 
checking for gcc -pipe switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for compiler structure-packing pragma ... found
checking for unistd.h ... found
...
...
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

[root@i-1avyrt2d tengine-2.1.1]# echo $?
0
[root@i-1avyrt2d tengine-2.1.1]#
{% endhighlight %}


---

####编译

{% highlight bash %}
[root@i-1avyrt2d tengine-2.1.1]# make 
make -f objs/Makefile
make[1]: Entering directory `/usr/local/src/tengine-2.1.1'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
		-o objs/src/core/nginx.o \
		src/core/nginx.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
		-o objs/src/core/ngx_log.o \
		src/core/ngx_log.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
		-o objs/src/core/ngx_palloc.o \
		src/core/ngx_palloc.c
...
...
	objs/src/http/modules/ngx_http_upstream_dynamic_module.o \
	objs/src/http/modules/ngx_http_stub_status_module.o \
	objs/ngx_modules.o \
	-lpthread -ldl -lcrypt -lpcre -lssl -lcrypto -ldl -lz
make[1]: Leaving directory `/usr/local/src/tengine-2.1.1'
make -f objs/Makefile manpage
make[1]: Entering directory `/usr/local/src/tengine-2.1.1'
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
		-e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
		-e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
		-e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
		< man/nginx.8 > objs/nginx.8
make[1]: Leaving directory `/usr/local/src/tengine-2.1.1'
[root@i-1avyrt2d tengine-2.1.1]# echo $?
0
[root@i-1avyrt2d tengine-2.1.1]#
{% endhighlight %}

---

####安装

{% highlight bash %}
[root@i-1avyrt2d tengine-2.1.1]# make install 
make -f objs/Makefile install
make[1]: Entering directory `/usr/local/src/tengine-2.1.1'
test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'
test -d '/usr/local/nginx/sbin' 		|| mkdir -p '/usr/local/nginx/sbin'
test ! -f '/usr/local/nginx/sbin/nginx' 		|| mv '/usr/local/nginx/sbin/nginx' 			'/usr/local/nginx/sbin/nginx.old'
cp objs/nginx '/usr/local/nginx/sbin/nginx'
test -d '/usr/local/nginx/conf' 		|| mkdir -p '/usr/local/nginx/conf'
cp conf/koi-win '/usr/local/nginx/conf'
cp conf/koi-utf '/usr/local/nginx/conf'
cp conf/win-utf '/usr/local/nginx/conf'
...
...
test -f 'src/http/ngx_http_upstream_round_robin.h' && cp 'src/http/ngx_http_upstream_round_robin.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_busy_lock.h' && cp 'src/http/ngx_http_busy_lock.h' '/usr/local/nginx/include'
test -f 'src/http/modules/ngx_http_ssi_filter_module.h' && cp 'src/http/modules/ngx_http_ssi_filter_module.h' '/usr/local/nginx/include'
test -f 'src/http/modules/ngx_http_ssl_module.h' && cp 'src/http/modules/ngx_http_ssl_module.h' '/usr/local/nginx/include'
test -f 'src/http/modules/ngx_http_reqstat.h' && cp 'src/http/modules/ngx_http_reqstat.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_headers.h'  && cp 'objs/ngx_auto_headers.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_config.h' && cp 'objs/ngx_auto_config.h' '/usr/local/nginx/include'
make[1]: Leaving directory `/usr/local/src/tengine-2.1.1'
[root@i-1avyrt2d tengine-2.1.1]# echo $?
0
[root@i-1avyrt2d tengine-2.1.1]# 
{% endhighlight %}

---

##配置检查与启动


{% highlight bash %}
[root@i-1avyrt2d nginx]# sbin/nginx -t -c conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@i-1avyrt2d nginx]# sbin/nginx  -c conf/nginx.conf
[root@i-1avyrt2d nginx]# 
{% endhighlight %}

之后就可以访问了

![Image_201511041423251.png](/images/tengine/Image_201511041423251.png)


---

##模块


Tengine 将检查和监模块都集成了进来，非常方便


模块查看方法


{% highlight bash %}
[root@i-1avyrt2d nginx.old]# /usr/local/nginx/sbin/nginx  -h 
Tengine version: Tengine/2.1.1 (nginx/1.6.2)
Usage: nginx [-?hvmVtdq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -m            : show all modules and exit
  -l            : show all directives and exit
  -V            : show version, modules and configure options then exit
  -t            : test configuration and exit
  -d            : dump configuration and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file

[root@i-1avyrt2d nginx.old]# /usr/local/nginx/sbin/nginx  -m
Tengine version: Tengine/2.1.1 (nginx/1.6.2)
loaded modules:
    ngx_core_module (static)
    ngx_errlog_module (static)
    ngx_conf_module (static)
    ngx_dso_module (static)
    ngx_syslog_module (static)
    ngx_events_module (static)
    ngx_event_core_module (static)
    ngx_epoll_module (static)
    ngx_procs_module (static)
    ngx_proc_core_module (static)
    ngx_openssl_module (static)
    ngx_regex_module (static)
    ngx_http_module (static)
    ngx_http_core_module (static)
    ngx_http_log_module (static)
    ngx_http_upstream_module (static)
    ngx_http_static_module (static)
    ngx_http_autoindex_module (static)
    ngx_http_index_module (static)
    ngx_http_auth_basic_module (static)
    ngx_http_access_module (static)
    ngx_http_limit_conn_module (static)
    ngx_http_limit_req_module (static)
    ngx_http_geo_module (static)
    ngx_http_map_module (static)
    ngx_http_split_clients_module (static)
    ngx_http_referer_module (static)
    ngx_http_rewrite_module (static)
    ngx_http_ssl_module (static)
    ngx_http_proxy_module (static)
    ngx_http_fastcgi_module (static)
    ngx_http_uwsgi_module (static)
    ngx_http_scgi_module (static)
    ngx_http_memcached_module (static)
    ngx_http_empty_gif_module (static)
    ngx_http_browser_module (static)
    ngx_http_user_agent_module (static)
    ngx_http_upstream_ip_hash_module (static)
    ngx_http_upstream_consistent_hash_module (static)
    ngx_http_upstream_check_module (static)
    ngx_http_upstream_least_conn_module (static)
    ngx_http_reqstat_module (static)
    ngx_http_upstream_keepalive_module (static)
    ngx_http_upstream_dynamic_module (static)
    ngx_http_stub_status_module (static)
    ngx_http_write_filter_module (static)
    ngx_http_header_filter_module (static)
    ngx_http_chunked_filter_module (static)
    ngx_http_range_header_filter_module (static)
    ngx_http_gzip_filter_module (static)
    ngx_http_postpone_filter_module (static)
    ngx_http_ssi_filter_module (static)
    ngx_http_charset_filter_module (static)
    ngx_http_userid_filter_module (static)
    ngx_http_footer_filter_module (static)
    ngx_http_trim_filter_module (static)
    ngx_http_headers_filter_module (static)
    ngx_http_upstream_session_sticky_module (static)
    ngx_http_copy_filter_module (static)
    ngx_http_range_body_filter_module (static)
    ngx_http_not_modified_filter_module (static)
[root@i-1avyrt2d nginx.old]# 
{% endhighlight %}

这些模块都很实用， **ngx_http_upstream_check_module** 可以检查后端服务器的状态

{% highlight bash %}
[root@i-1avyrt2d nginx.old]# /usr/local/nginx/sbin/nginx  -m  2>&1  | grep upstream 
    ngx_http_upstream_module (static)
    ngx_http_upstream_ip_hash_module (static)
    ngx_http_upstream_consistent_hash_module (static)
    ngx_http_upstream_check_module (static)
    ngx_http_upstream_least_conn_module (static)
    ngx_http_upstream_keepalive_module (static)
    ngx_http_upstream_dynamic_module (static)
    ngx_http_upstream_session_sticky_module (static)
[root@i-1avyrt2d nginx.old]# /usr/local/nginx/sbin/nginx  -m  2>&1  | grep upstream | grep check 
    ngx_http_upstream_check_module (static)
[root@i-1avyrt2d nginx.old]# 
{% endhighlight %}


> **Tip:**  官方版本的没有 **-m** 选项，不能方便的列出加载的模块

{% highlight bash %}
[root@i-1avyrt2d nginx.old]# sbin/nginx -h 
nginx version: nginx/1.9.6
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file

[root@i-1avyrt2d nginx.old]# sbin/nginx -v
nginx version: nginx/1.9.6
[root@i-1avyrt2d nginx.old]# sbin/nginx -V
nginx version: nginx/1.9.6
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC) 
configure arguments:
[root@i-1avyrt2d nginx.old]# 
{% endhighlight %}

从目录结构可以看到，多了不少东西， **nginx.old** 是官方版本

{% highlight bash %}
[root@i-1avyrt2d nginx]# ll /usr/local/nginx
total 44
drwx------ 2 nginx root 4096 Nov  4 14:17 client_body_temp
drwxr-xr-x 3 root  root 4096 Nov  4 16:09 conf
drwx------ 2 nginx root 4096 Nov  4 14:17 fastcgi_temp
drwxr-xr-x 2 root  root 4096 Nov  4 14:15 html
drwxr-xr-x 2 root  root 4096 Nov  4 14:15 include
drwxr-xr-x 2 root  root 4096 Nov  4 16:10 logs
drwxr-xr-x 2 root  root 4096 Nov  4 14:15 modules
drwx------ 2 nginx root 4096 Nov  4 14:17 proxy_temp
drwxr-xr-x 2 root  root 4096 Nov  4 14:15 sbin
drwx------ 2 nginx root 4096 Nov  4 14:17 scgi_temp
drwx------ 2 nginx root 4096 Nov  4 14:17 uwsgi_temp
[root@i-1avyrt2d nginx]# ll /usr/local/nginx.old/
total 16
drwxr-xr-x 3 root root 4096 Nov  4 10:37 conf
drwxr-xr-x 2 root root 4096 Nov  4 10:22 html
drwxr-xr-x 2 root root 4096 Nov  4 13:09 logs
drwxr-xr-x 2 root root 4096 Nov  4 10:22 sbin
[root@i-1avyrt2d nginx]# 
{% endhighlight %}

---

##案例分析

Nginx 很大的一个作用就是作为web前端进行负载均衡和反向代理，但负载均衡的一个关键点就是状态检查，因为不能把请求分配给有故障的后端服务器

下面对一个案例进行分析

{% highlight bash %}
[root@i-1avyrt2d conf]# cat nginx.conf | grep -v "#" | grep -v "^$"  
user nginx nginx;
worker_processes  4;
error_log  /var/log/nginx/error.log error;
worker_rlimit_nofile 65535;
events {
	worker_connections  65535;
	use epoll;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    include apps/proxy_tengine.conf;
}
[root@i-1avyrt2d conf]# cat apps/proxy_tengine.conf | grep -v "#" | grep -v "^$"  
    upstream test_apps {
        server x.x.x.x:80  max_fails=1 fail_timeout=10s weight=25;
        server y.y.y.y:80  max_fails=1 fail_timeout=10s  weight=25;
        server x.x.x.x:80  max_fails=1 fail_timeout=10s  weight=25;
        server y.y.y.y:80  max_fails=1 fail_timeout=10s  weight=25;
        check interval=5000 fall=5 rise=2 timeout=2000 default_down=false type=http;
        check_http_send "HEAD /health_status HTTP/1.0\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
    }
    server {
        listen  80;
        server_name  *.boohee.com   _;
        keepalive_timeout  30; 
        location /nginx-status {
            stub_status on;
            access_log  off;
            allow z.z.z.z;
            allow i.i.i.i/28;
            deny all;
        }
	location /status {
        auth_basic      "input your name and passsword";
        auth_basic_user_file  apps/status.passwd;
        check_status;
        access_log off;
        allow all;
	}	
	location / {
	    proxy_pass http://test_apps;
            proxy_set_header X-Forwarded-For  $remote_addr;
	
	}
     }
[root@i-1avyrt2d conf]# ../sbin/nginx  -t -c /usr/local/nginx/conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@i-1avyrt2d conf]# kill  -HUP `cat ../logs/nginx.pid `
[root@i-1avyrt2d conf]# 
{% endhighlight %}

后面对这个配置的不同部分进行详细分析

---


###负载均衡

nginx可以很简单的配置成http负载均衡服务器，对前端的请求进行转发

{% highlight bash %}
    upstream test_apps {
        server x.x.x.x:80  max_fails=1 fail_timeout=10s weight=25;
        server y.y.y.y:80  max_fails=1 fail_timeout=10s  weight=25;
        server x.x.x.x:80  max_fails=1 fail_timeout=10s  weight=25;
        server y.y.y.y:80  max_fails=1 fail_timeout=10s  weight=25;
    }
{% endhighlight %}

upstream 是nginx 负载均衡的主要模块，它提供了一个简单方法来轮询后端的服务器

server 使用于 upstream  环境，服务名称可以是一个域名，一个ip地址，ip地址加端口，也可以是UNIX Socket。

---


| Item     | Type  | comment   |
| :------- | :----: | :---|
|max_fails| NUMBER |参数fail\_timeout指定的时间里允许对服务器请求失败的次数(默认为1)|
|weight|NUMBER|权重，值越高，被分配的请求数越多(默认为1)|
|fail_timeout|TIME|失效超时时间|
|down|-|标记服务器离线|
|backup|-|标记服务器，在所有非backup服务器宕机的时候才启用|


###反向代理

{% highlight bash %}
	location / {
	    proxy_pass http://test_apps;
            proxy_set_header X-Forwarded-For  $remote_addr;
	
	}
{% endhighlight %}


---

###状态检查

单纯轮询，其中一台出问题后，会影响一定比例请求的正常响应，加入检查模块就可以避免此问题

这个检查逻辑就是 **ngx_http_upstream_check_module** 模块提供的，如果使用官方版，需要额外编译加入此模块

{% highlight bash %}
    upstream test_apps {
        server x.x.x.x:80  max_fails=1 fail_timeout=10s weight=25;
        server y.y.y.y:80  max_fails=1 fail_timeout=10s  weight=25;
        server x.x.x.x:80  max_fails=1 fail_timeout=10s  weight=25;
        server y.y.y.y:80  max_fails=1 fail_timeout=10s  weight=25;
        check interval=5000 fall=5 rise=2 timeout=2000 default_down=false type=http;
        check_http_send "HEAD /health_status HTTP/1.0\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
    }
{% endhighlight %}

创建 **apps/status.passwd** 文件，创建方法(用户设为test，密码设为tengine) 

{% highlight bash %}
[root@i-1avyrt2d apps]#  perl -e 'print  crypt(tengine,tengine)';
tejMqaZALnkgk[root@i-1avyrt2d apps]# vim status.passwd 
[root@i-1avyrt2d apps]# cat status.passwd 
test:tejMqaZALnkgk
[root@i-1avyrt2d apps]# 
{% endhighlight %}


下面代码的作用就是可以使用 **http://ip/status** 监控后端服务器的状态

{% highlight bash %}
	location /status {
        auth_basic      "input your name and passsword";
        auth_basic_user_file  apps/status.passwd;
        check_status;
        access_log off;
        allow all;
	}	
{% endhighlight %}

> **Tip:** 在启动前可以使用 nginx check 一下配置语法 

{% highlight bash %}
[root@i-1avyrt2d conf]# ../sbin/nginx  -t -c /usr/local/nginx/conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@i-1avyrt2d conf]# kill  -HUP `cat ../logs/nginx.pid `
[root@i-1avyrt2d conf]# 
{% endhighlight %}


使用浏览器访问 **http://103.21.118.104/**

![Image_201511042047144.png](/images/tengine/Image_201511042047144.png)

使用浏览器访问 **http://103.21.118.104/status**

![Image_201511042051527.png](/images/tengine/Image_201511042051527.png)

认证成功后 (test:tengine)

![Image_201511042102329.png](/images/tengine/Image_201511042102329.png)



---

[tengine]:http://tengine.taobao.org/index_cn.html
[download]:http://tengine.taobao.org/download_cn.html
[doc]:http://tengine.taobao.org/documentation_cn.html


