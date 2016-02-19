---
layout: post
title:  HAproxy 基础
categories:  linux cluster haproxy 
wc: 526  1846 17410 
excerpt: haproxy的下载与安装，运行与监控 
comments: true
---




# 前言

**[HAProxy][haproxy]** 是一个稳定的开源的高性能 **TCP/HTTP** 负载均衡软件

>The Reliable, High Performance TCP/HTTP Load Balancer

生产环境下多使用它在前端作负载均衡，提高系统的扩展性，它的作用类似于 **LVS (Linux Virtual Servers)** 和 **Nginx ("engine X")**  ( **LVS** 主要作用在网络的第 **3/4** 层也就是 **ip:port** ， **Nginx** 主要作用在顶层应用层，其本身就是一个 **webserver** )

**[HAProxy][haproxy]** 只专注于 **TCP/HTTP** ，所以相较于 **Nginx** ，它可以作mysql的前端，相较于 **LVS** ，它可以直接代理web请求

>HAProxy is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for very high traffic web sites and powers quite a number of the world's most visited ones.

> **Tip:**  关于 **LB** 基础概念可以参考 **[LB概要][lb]**

这里简单分享一下 **[HAProxy][haproxy]** 的相关基础 ，详细内容可以参考 **[官方文档][doc]**


> **Tip:** 当前的最新稳定版为 **HAProxy 1.6.3**


---


# 概要

* TOC
{:toc}



---

## 环境

{% highlight bash %}
[root@h102 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 ~]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 ~]# cat /proc/cpuinfo 
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 37
model name	: Intel(R) Core(TM) i5 CPU       M 480  @ 2.67GHz
stepping	: 5
microcode	: 2
cpu MHz		: 2660.529
cache size	: 3072 KB
physical id	: 0
siblings	: 2
core id		: 0
cpu cores	: 2
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 11
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 ss ht syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts xtopology tsc_reliable nonstop_tsc aperfmperf unfair_spinlock pni ssse3 cx16 sse4_1 sse4_2 x2apic popcnt hypervisor lahf_lm ida arat dts
bogomips	: 5321.05
clflush size	: 64
cache_alignment	: 64
address sizes	: 40 bits physical, 48 bits virtual
power management:

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 37
model name	: Intel(R) Core(TM) i5 CPU       M 480  @ 2.67GHz
stepping	: 5
microcode	: 2
cpu MHz		: 2660.529
cache size	: 3072 KB
physical id	: 0
siblings	: 2
core id		: 1
cpu cores	: 2
apicid		: 1
initial apicid	: 1
fpu		: yes
fpu_exception	: yes
cpuid level	: 11
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 ss ht syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts xtopology tsc_reliable nonstop_tsc aperfmperf unfair_spinlock pni ssse3 cx16 sse4_1 sse4_2 x2apic popcnt hypervisor lahf_lm ida arat dts
bogomips	: 5321.05
clflush size	: 64
cache_alignment	: 64
address sizes	: 40 bits physical, 48 bits virtual
power management:

[root@h102 ~]# 
{% endhighlight %}

---

## 源码下载与安装


### 下载

**[HAProxy][haproxy]** 的 **[下载地址][dl]**

我们使用当前最新版本 **[haproxy-1.6.3.tar.gz][haproxy_1.6.3]**

{% highlight bash %}
[root@h102 haproxy]# ls
haproxy-1.6.3.tar.gz
[root@h102 haproxy]# md5sum  haproxy-1.6.3.tar.gz
3362d1e268c78155c2474cb73e7f03f9  haproxy-1.6.3.tar.gz
[root@h102 haproxy]# 
{% endhighlight %}

---

### 解压

{% highlight bash %}
[root@h102 haproxy]# tar -xzvf haproxy-1.6.3.tar.gz 
haproxy-1.6.3/
haproxy-1.6.3/.gitignore
haproxy-1.6.3/CHANGELOG
haproxy-1.6.3/CONTRIBUTING
haproxy-1.6.3/LICENSE
haproxy-1.6.3/MAINTAINERS
haproxy-1.6.3/Makefile
haproxy-1.6.3/README
...
...
haproxy-1.6.3/tests/test.c
haproxy-1.6.3/tests/test_hashes.c
haproxy-1.6.3/tests/test_pools.c
haproxy-1.6.3/tests/testinet.c
haproxy-1.6.3/tests/uri_hash.c
[root@h102 haproxy]# echo $?
0
[root@h102 haproxy]# ls
haproxy-1.6.3  haproxy-1.6.3.tar.gz
[root@h102 haproxy]# ls haproxy-1.6.3
CHANGELOG  CONTRIBUTING  ebtree    include  MAINTAINERS  README   src      tests    VERSION
contrib    doc           examples  LICENSE  Makefile     ROADMAP  SUBVERS  VERDATE
[root@h102 haproxy]# 
{% endhighlight %}


---

### 编译安装

#### 配置

源码的目录下有一个 **README** 文件

{% highlight bash %}
[root@h102 haproxy-1.6.3]# ls
CHANGELOG  CONTRIBUTING  ebtree    include  MAINTAINERS  README   src      tests    VERSION
contrib    doc           examples  LICENSE  Makefile     ROADMAP  SUBVERS  VERDATE
[root@h102 haproxy-1.6.3]# wc -l README 
500 README
[root@h102 haproxy-1.6.3]# 
{% endhighlight %}

这个文件里对安装进行了说明，其中关于优化有以下三点需要注意

* **TARGET**

{% highlight bash %}
To build haproxy, you have to choose your target OS amongst the following ones
and assign it to the TARGET variable :

  - linux22     for Linux 2.2
  - linux24     for Linux 2.4 and above (default)
  - linux24e    for Linux 2.4 with support for a working epoll (> 0.21)
  - linux26     for Linux 2.6 and above
  - linux2628   for Linux 2.6.28, 3.x, and above (enables splice and tproxy)
  - solaris     for Solaris 8 or 10 (others untested)
  - freebsd     for FreeBSD 5 to 10 (others untested)
  - netbsd      for NetBSD
  - osx         for Mac OS/X
  - openbsd     for OpenBSD 3.1 and above
  - aix51       for AIX 5.1
  - aix52       for AIX 5.2
  - cygwin      for Cygwin
  - generic     for any other OS or version.
  - custom      to manually adjust every setting
{% endhighlight %}

* **CPU**

{% highlight bash %}
You may also choose your CPU to benefit from some optimizations. This is
particularly important on UltraSparc machines. For this, you can assign
one of the following choices to the CPU variable :

  - i686 for intel PentiumPro, Pentium 2 and above, AMD Athlon
  - i586 for intel Pentium, AMD K6, VIA C3.
  - ultrasparc : Sun UltraSparc I/II/III/IV processor
  - native : use the build machine's specific processor optimizations. Use with
    extreme care, and never in virtualized environments (known to break).
  - generic : any other processor or no CPU-specific optimization. (default)
{% endhighlight %}

* **ARCH**

{% highlight bash %}
You may want to build specific target binaries which do not match your native
compiler's target. This is particularly true on 64-bit systems when you want
to build a 32-bit binary. Use the ARCH variable for this purpose. Right now
it only knows about a few x86 variants (i386,i486,i586,i686,x86_64), two
generic ones (32,64) and sets -m32/-m64 as well as -march=<arch> accordingly.
{% endhighlight %}


其它编译配置是针对 **PCRE** (正则匹配)、**OpenSSL** 、 **ZLIB** 的优化和开关


根据我们的具体环境，使用如下配置进行编译


> **Tip:** 不指定 **CPU** 的情况下，默认会使用 **generic**


---

#### 编译

{% highlight bash %}
[root@h102 haproxy-1.6.3]# make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy
gcc -Iinclude -Iebtree -Wall -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement       -DCONFIG_HAP_LINUX_SPLICE -DTPROXY -DCONFIG_HAP_LINUX_TPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DUSE_CPU_AFFINITY -DASSUME_SPLICE_WORKS -DUSE_ACCEPT4 -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.6.3\" -DCONFIG_HAPROXY_DATE=\"2015/12/25\" \
	      -DBUILD_TARGET='"linux2628"' \
	      -DBUILD_ARCH='"x86_64"' \
	      -DBUILD_CPU='"generic"' \
	      -DBUILD_CC='"gcc"' \
	      -DBUILD_CFLAGS='"-m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement"' \
	      -DBUILD_OPTIONS='""' \
	       -c -o src/haproxy.o src/haproxy.c
gcc -Iinclude -Iebtree -Wall -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement       -DCONFIG_HAP_LINUX_SPLICE -DTPROXY -DCONFIG_HAP_LINUX_TPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DUSE_CPU_AFFINITY -DASSUME_SPLICE_WORKS -DUSE_ACCEPT4 -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.6.3\" -DCONFIG_HAPROXY_DATE=\"2015/12/25\" -c -o src/base64.o src/base64.c
...
...
gcc -m64 -march=x86-64 -g -o haproxy src/haproxy.o src/base64.o src/protocol.o src/uri_auth.o src/standard.o src/buffer.o src/log.o src/task.o src/chunk.o src/channel.o src/listener.o src/lru.o src/xxhash.o src/time.o src/fd.o src/pipe.o src/regex.o src/cfgparse.o src/server.o src/checks.o src/queue.o src/frontend.o src/proxy.o src/peers.o src/arg.o src/stick_table.o src/proto_uxst.o src/connection.o src/proto_http.o src/raw_sock.o src/backend.o src/lb_chash.o src/lb_fwlc.o src/lb_fwrr.o src/lb_map.o src/lb_fas.o src/stream_interface.o src/dumpstats.o src/proto_tcp.o src/applet.o src/session.o src/stream.o src/hdr_idx.o src/ev_select.o src/signal.o src/acl.o src/sample.o src/memory.o src/freq_ctr.o src/auth.o src/proto_udp.o src/compression.o src/payload.o src/hash.o src/pattern.o src/map.o src/namespace.o src/mailers.o src/dns.o src/vars.o src/ev_poll.o src/ev_epoll.o ebtree/ebtree.o ebtree/eb32tree.o ebtree/eb64tree.o ebtree/ebmbtree.o ebtree/ebsttree.o ebtree/ebimtree.o ebtree/ebistree.o   -lcrypt -ldl 
gcc -Iinclude -Iebtree -Wall -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement       -DCONFIG_HAP_LINUX_SPLICE -DTPROXY -DCONFIG_HAP_LINUX_TPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DUSE_CPU_AFFINITY -DASSUME_SPLICE_WORKS -DUSE_ACCEPT4 -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.6.3\" -DCONFIG_HAPROXY_DATE=\"2015/12/25\" \
	      -DSBINDIR='"/usr/local/haproxy/sbin"' \
	       -c -o src/haproxy-systemd-wrapper.o src/haproxy-systemd-wrapper.c
gcc -m64 -march=x86-64 -g -o haproxy-systemd-wrapper src/haproxy-systemd-wrapper.o   -lcrypt -ldl 
[root@h102 haproxy-1.6.3]# echo $?
0
[root@h102 haproxy-1.6.3]# 
{% endhighlight %}

---

#### 安装

{% highlight bash %}
[root@h102 haproxy-1.6.3]# make install PREFIX=/usr/local/haproxy
install -d "/usr/local/haproxy/sbin"
install haproxy  "/usr/local/haproxy/sbin"
install -d "/usr/local/haproxy/share/man"/man1
install -m 644 doc/haproxy.1 "/usr/local/haproxy/share/man"/man1
install -d "/usr/local/haproxy/doc/haproxy"
for x in architecture close-options configuration cookie-options intro linux-syn-cookies lua management network-namespaces proxy-protocol; do \
		install -m 644 doc/$x.txt "/usr/local/haproxy/doc/haproxy" ; \
	done
[root@h102 haproxy-1.6.3]# echo $?
0
[root@h102 haproxy-1.6.3]# ll /usr/local/haproxy/
total 12
drwxr-xr-x 3 root root 4096 Feb 19 17:03 doc
drwxr-xr-x 2 root root 4096 Feb 19 17:03 sbin
drwxr-xr-x 3 root root 4096 Feb 19 17:03 share
[root@h102 haproxy-1.6.3]# tree /usr/local/haproxy/
/usr/local/haproxy/
├── doc
│   └── haproxy
│       ├── architecture.txt
│       ├── close-options.txt
│       ├── configuration.txt
│       ├── cookie-options.txt
│       ├── intro.txt
│       ├── linux-syn-cookies.txt
│       ├── lua.txt
│       ├── management.txt
│       ├── network-namespaces.txt
│       └── proxy-protocol.txt
├── sbin
│   └── haproxy
└── share
    └── man
        └── man1
            └── haproxy.1

6 directories, 12 files
[root@h102 haproxy-1.6.3]# 
{% endhighlight %}

版本查看

**`-vv`** 可以查看编译的配置选项

{% highlight bash %}
[root@h102 haproxy-1.6.3]# /usr/local/haproxy/sbin/haproxy  -vv
HA-Proxy version 1.6.3 2015/12/25
Copyright 2000-2015 Willy Tarreau <willy@haproxy.org>

Build options :
  TARGET  = linux2628
  CPU     = generic
  CC      = gcc
  CFLAGS  = -m64 -march=x86-64 -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement
  OPTIONS = 

Default settings :
  maxconn = 2000, bufsize = 16384, maxrewrite = 1024, maxpollevents = 200

Encrypted password support via crypt(3): yes
Built without compression support (neither USE_ZLIB nor USE_SLZ are set)
Compression algorithms supported : identity("identity")
Built without OpenSSL support (USE_OPENSSL not set)
Built without PCRE support (using libc's regex instead)
Built without Lua support
Built with transparent proxy support using: IP_TRANSPARENT IPV6_TRANSPARENT IP_FREEBIND

Available polling systems :
      epoll : pref=300,  test result OK
       poll : pref=200,  test result OK
     select : pref=150,  test result OK
Total: 3 (3 usable), will use epoll.

[root@h102 haproxy-1.6.3]# 
{% endhighlight %}


---

## 运行


准备配置文件

源码包中有一些示例，可以作为配置模板

{% highlight bash %}
[root@h102 ~]# ll /usr/local/src/haproxy/haproxy-1.6.3/examples/*.cfg
-rw-rw-r-- 1 root root 3740 Dec 27 22:04 /usr/local/src/haproxy/haproxy-1.6.3/examples/acl-content-sw.cfg
-rw-rw-r-- 1 root root 3042 Dec 27 22:04 /usr/local/src/haproxy/haproxy-1.6.3/examples/auth.cfg
-rw-rw-r-- 1 root root 2499 Dec 27 22:04 /usr/local/src/haproxy/haproxy-1.6.3/examples/content-sw-sample.cfg
-rw-rw-r-- 1 root root 1234 Dec 27 22:04 /usr/local/src/haproxy/haproxy-1.6.3/examples/option-http_proxy.cfg
-rw-rw-r-- 1 root root  619 Dec 27 22:04 /usr/local/src/haproxy/haproxy-1.6.3/examples/ssl.cfg
-rw-rw-r-- 1 root root 2274 Dec 27 22:04 /usr/local/src/haproxy/haproxy-1.6.3/examples/transparent_proxy.cfg
[root@h102 ~]# 
[root@h102 ~]# mkdir /etc/haproxy
[root@h102 ~]# cp /usr/local/src/haproxy/haproxy-1.6.3/examples/transparent_proxy.cfg  /etc/haproxy/
[root@h102 ~]# grep -v "^#" /etc/haproxy/transparent_proxy.cfg 

global
defaults
	timeout client		30s
	timeout server		30s
	timeout connect		30s

frontend MyFrontend
	bind	192.168.1.22:80
	default_backend		TransparentBack_http

backend TransparentBack_http
	mode			http
	source 0.0.0.0 usesrc client
	server			MyWebServer 192.168.0.40:80

[root@h102 ~]# 
{% endhighlight %}


启动nginx，作为一个后端的web服务器

{% highlight bash %}
[root@h102 nginx]# sbin/nginx  -t -c conf/nginx.conf
the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@h102 nginx]# sbin/nginx   -c conf/nginx.conf
[root@h102 ~]# netstat  -ant | grep 80
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      
[root@h102 ~]# 
{% endhighlight %}


![nginx.png](/images/haproxy/nginx.png)


修改haproxy配置

{% highlight bash %}
[root@h102 ~]# vim /etc/haproxy/transparent_proxy.cfg
[root@h102 ~]# grep -v "^#" /etc/haproxy/transparent_proxy.cfg 

global
defaults
	timeout client		30s
	timeout server		30s
	timeout connect		30s

frontend MyFrontend
	bind	0.0.0.0:1234
	default_backend		TransparentBack_http

backend TransparentBack_http
	mode			http
	server			MyWebServer 127.0.0.1:80

[root@h102 ~]#
{% endhighlight %}

启动haproxy

{% highlight bash %}
[root@h102 ~]# /usr/local/haproxy/sbin/haproxy -f /etc/haproxy/transparent_proxy.cfg
...
...
...
----------
[root@h102 ~]# netstat  -ant | grep 1234
tcp        0      0 0.0.0.0:1234                0.0.0.0:*                   LISTEN      
[root@h102 ~]# 
{% endhighlight %}


进行访问

![haproxy.png](/images/haproxy/haproxy.png)



---

## 启动监控



{% highlight bash %}
[root@h102 ~]# vim  /etc/haproxy/transparent_proxy.cfg
[root@h102 ~]# grep -v "^#" /etc/haproxy/transparent_proxy.cfg

global
defaults
	timeout client		30s
	timeout server		30s
	timeout connect		30s
	stats  enable
	stats uri /admin
	stats auth admin:admin

frontend MyFrontend
	bind	0.0.0.0:1234
	default_backend		TransparentBack_http

backend TransparentBack_http
	mode			http
	server			MyWebServer 127.0.0.1:80

[root@h102 ~]# /usr/local/haproxy/sbin/haproxy -f /etc/haproxy/transparent_proxy.cfg
[WARNING] 049/222700 (33939) : config : 'stats' statement ignored for frontend 'MyFrontend' as it requires HTTP mode.
...
...
...
{% endhighlight %}

进行访问

过程中会弹出对话框，输入帐号密码后，就可以看到监控界面

![haproxy-mon.png](/images/haproxy/haproxy-mon.png)


---

# 命令汇总


* **`md5sum  haproxy-1.6.3.tar.gz`**
* **`tar -xzvf haproxy-1.6.3.tar.gz`**
* **`make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy`**
* **`make install PREFIX=/usr/local/haproxy`**
* **`ll /usr/local/haproxy/`**
* **`tree /usr/local/haproxy/`**
* **`/usr/local/haproxy/sbin/haproxy  -vv`**
* **`ll /usr/local/src/haproxy/haproxy-1.6.3/examples/*.cfg`**
* **`cp /usr/local/src/haproxy/haproxy-1.6.3/examples/transparent_proxy.cfg  /etc/haproxy/`**
* **`grep -v "^#" /etc/haproxy/transparent_proxy.cfg`**
* **`sbin/nginx  -t -c conf/nginx.conf`**
* **`sbin/nginx   -c conf/nginx.conf`**
* **`netstat  -ant | grep 80`**
* **`vim /etc/haproxy/transparent_proxy.cfg`**
* **`grep -v "^#" /etc/haproxy/transparent_proxy.cfg`**
* **`/usr/local/haproxy/sbin/haproxy -f /etc/haproxy/transparent_proxy.cfg`**
* **`netstat  -ant | grep 1234`**
* **`vim  /etc/haproxy/transparent_proxy.cfg`**
* **`grep -v "^#" /etc/haproxy/transparent_proxy.cfg`**
* **`/usr/local/haproxy/sbin/haproxy -f /etc/haproxy/transparent_proxy.cfg`**



---



[haproxy]:http://www.haproxy.org/
[doc]:http://cbonte.github.io/haproxy-dconv/intro-1.6.html
[lb]:http://cbonte.github.io/haproxy-dconv/intro-1.6.html#2
[dl]:http://www.haproxy.org/#down
[haproxy_1.6.3]:http://www.haproxy.org/download/1.6/src/haproxy-1.6.3.tar.gz

