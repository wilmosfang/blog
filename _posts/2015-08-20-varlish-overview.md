---
layout: default
title: Varnish 基础概览
comments: true
---

前言
=

[Varnish][varnish]是一个高效的代理服务器，可以进行网站缓存，CDN中常使用它来进行网站加速。

---

安装
-

下载并安装 **varnish** 的 **[repo][varnish.repo]**

~~~
[root@h101 varnish]# wget  https://repo.varnish-cache.org/redhat/varnish-4.0.el6.rpm 
--2015-08-19 22:11:24--  https://repo.varnish-cache.org/redhat/varnish-4.0.el6.rpm
Resolving repo.varnish-cache.org... 194.31.39.155
Connecting to repo.varnish-cache.org|194.31.39.155|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7132 (7.0K) [application/x-redhat-package-manager]
Saving to: “varnish-4.0.el6.rpm”

100%[==========================================================================================>] 7,132       --.-K/s   in 0s      

2015-08-19 22:11:28 (242 MB/s) - “varnish-4.0.el6.rpm” saved [7132/7132]

[root@h101 varnish]# ls
varnish-4.0.el6.rpm
[root@h101 varnish]# rpm -ivh varnish-4.0.el6.rpm 
warning: varnish-4.0.el6.rpm: Header V4 RSA/SHA1 Signature, key ID 8f2d409f: NOKEY
Preparing...                ########################################### [100%]
   1:varnish-release        ########################################### [100%]
[root@h101 varnish]# rpm -qa | grep varnish 
varnish-release-4.0-3.el6.noarch
[root@h101 varnish]# rpm -ql varnish-release-4.0-3.el6.noarch 
/etc/pki/rpm-gpg/RPM-GPG-KEY-VARNISH
/etc/pki/rpm-gpg/RPM-GPG-KEY-VARNISH-SOFTWARE
/etc/yum.repos.d/varnish.repo
[root@h101 varnish]# yum list all | grep varnish 
Repository base is listed more than once in the configuration
http://mirror01.idc.hinet.net/EPEL/6/x86_64/repodata/repomd.xml: [Errno -1] repomd.xml does not match metalink for epel
Trying other mirror.
varnish-release.noarch                      4.0-3.el6                    installed
varnish.x86_64                              4.0.3-1.el6                  varnish-4.0
varnish-agent.x86_64                        4.0.1-1.el6                  varnish-4.0
varnish-agent-debuginfo.x86_64              4.0.1-1.el6                  varnish-4.0
varnish-debuginfo.x86_64                    4.0.3-1.el6                  varnish-4.0
varnish-docs.x86_64                         4.0.3-1.el6                  varnish-4.0
varnish-libs.i686                           2.1.5-5.el6                  epel   
varnish-libs.x86_64                         4.0.3-1.el6                  varnish-4.0
varnish-libs-devel.i686                     2.1.5-5.el6                  epel   
varnish-libs-devel.x86_64                   4.0.3-1.el6                  varnish-4.0
[root@h101 varnish]# 
~~~

安装 **varnish**

~~~
[root@h101 varnish]# yum -y install  varnish 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * epel: mirror01.idc.hinet.net
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package varnish.x86_64 0:4.0.3-1.el6 will be installed
--> Processing Dependency: varnish-libs = 4.0.3-1.el6 for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnishapi.so.1(LIBVARNISHAPI_1.1)(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: jemalloc for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnishapi.so.1(LIBVARNISHAPI_1.3)(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnishapi.so.1(LIBVARNISHAPI_1.2)(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: gcc for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnishapi.so.1(LIBVARNISHAPI_1.0)(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libjemalloc.so.1()(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvgz.so()(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnish.so()(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnishcompat.so()(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvarnishapi.so.1()(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Processing Dependency: libvcc.so()(64bit) for package: varnish-4.0.3-1.el6.x86_64
--> Running transaction check
---> Package gcc.x86_64 0:4.4.7-11.el6 will be installed
--> Processing Dependency: cpp = 4.4.7-11.el6 for package: gcc-4.4.7-11.el6.x86_64
--> Processing Dependency: cloog-ppl >= 0.15 for package: gcc-4.4.7-11.el6.x86_64
---> Package jemalloc.x86_64 0:3.6.0-1.el6 will be installed
---> Package varnish-libs.x86_64 0:4.0.3-1.el6 will be installed
--> Running transaction check
---> Package cloog-ppl.x86_64 0:0.15.7-1.2.el6 will be installed
--> Processing Dependency: libppl_c.so.2()(64bit) for package: cloog-ppl-0.15.7-1.2.el6.x86_64
--> Processing Dependency: libppl.so.7()(64bit) for package: cloog-ppl-0.15.7-1.2.el6.x86_64
---> Package cpp.x86_64 0:4.4.7-11.el6 will be installed
--> Processing Dependency: libmpfr.so.1()(64bit) for package: cpp-4.4.7-11.el6.x86_64
--> Running transaction check
---> Package mpfr.x86_64 0:2.4.1-6.el6 will be installed
---> Package ppl.x86_64 0:0.10.2-11.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                          Arch                       Version                            Repository                       Size
======================================================================================================================================
Installing:
 varnish                          x86_64                     4.0.3-1.el6                        varnish-4.0                     371 k
Installing for dependencies:
 cloog-ppl                        x86_64                     0.15.7-1.2.el6                     base                             93 k
 cpp                              x86_64                     4.4.7-11.el6                       base                            3.7 M
 gcc                              x86_64                     4.4.7-11.el6                       base                             10 M
 jemalloc                         x86_64                     3.6.0-1.el6                        epel                            100 k
 mpfr                             x86_64                     2.4.1-6.el6                        base                            157 k
 ppl                              x86_64                     0.10.2-11.el6                      base                            1.3 M
 varnish-libs                     x86_64                     4.0.3-1.el6                        varnish-4.0                     184 k

Transaction Summary
======================================================================================================================================
Install       8 Package(s)

Total download size: 16 M
Installed size: 35 M
Downloading Packages:
(1/8): jemalloc-3.6.0-1.el6.x86_64.rpm                                                                         | 100 kB     00:00     
(2/8): varnish-4.0.3-1.el6.x86_64.rpm                                                                          | 371 kB     00:14     
(3/8): varnish-libs-4.0.3-1.el6.x86_64.rpm                                                                     | 184 kB     00:14     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 469 kB/s |  16 MB     00:34     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : ppl-0.10.2-11.el6.x86_64                                                                                           1/8 
  Installing : cloog-ppl-0.15.7-1.2.el6.x86_64                                                                                    2/8 
  Installing : jemalloc-3.6.0-1.el6.x86_64                                                                                        3/8 
  Installing : mpfr-2.4.1-6.el6.x86_64                                                                                            4/8 
  Installing : cpp-4.4.7-11.el6.x86_64                                                                                            5/8 
  Installing : gcc-4.4.7-11.el6.x86_64                                                                                            6/8 
  Installing : varnish-libs-4.0.3-1.el6.x86_64                                                                                    7/8 
  Installing : varnish-4.0.3-1.el6.x86_64                                                                                         8/8 
  Verifying  : varnish-libs-4.0.3-1.el6.x86_64                                                                                    1/8 
  Verifying  : gcc-4.4.7-11.el6.x86_64                                                                                            2/8 
  Verifying  : varnish-4.0.3-1.el6.x86_64                                                                                         3/8 
  Verifying  : mpfr-2.4.1-6.el6.x86_64                                                                                            4/8 
  Verifying  : jemalloc-3.6.0-1.el6.x86_64                                                                                        5/8 
  Verifying  : cpp-4.4.7-11.el6.x86_64                                                                                            6/8 
  Verifying  : ppl-0.10.2-11.el6.x86_64                                                                                           7/8 
  Verifying  : cloog-ppl-0.15.7-1.2.el6.x86_64                                                                                    8/8 

Installed:
  varnish.x86_64 0:4.0.3-1.el6                                                                                                        

Dependency Installed:
  cloog-ppl.x86_64 0:0.15.7-1.2.el6   cpp.x86_64 0:4.4.7-11.el6    gcc.x86_64 0:4.4.7-11.el6           jemalloc.x86_64 0:3.6.0-1.el6  
  mpfr.x86_64 0:2.4.1-6.el6           ppl.x86_64 0:0.10.2-11.el6   varnish-libs.x86_64 0:4.0.3-1.el6  

Complete!
[root@h101 varnish]# 
~~~

---

核心配置与工具
-

**varnish** 提供的内容

~~~
[root@h101 varnish]# rpm -ql varnish
/etc/logrotate.d/varnish
/etc/rc.d/init.d/varnish
/etc/rc.d/init.d/varnishlog
/etc/rc.d/init.d/varnishncsa
/etc/sysconfig/varnish
/etc/varnish
/etc/varnish/default.vcl
/usr/bin/varnishadm
/usr/bin/varnishhist
/usr/bin/varnishlog
/usr/bin/varnishncsa
/usr/bin/varnishstat
/usr/bin/varnishtest
/usr/bin/varnishtop
/usr/sbin/varnish_reload_vcl
/usr/sbin/varnishd
/usr/share/doc/varnish
/usr/share/doc/varnish-4.0.3
/usr/share/doc/varnish-4.0.3/ChangeLog
/usr/share/doc/varnish-4.0.3/LICENSE
/usr/share/doc/varnish-4.0.3/README
/usr/share/doc/varnish-4.0.3/README.redhat
/usr/share/doc/varnish/builtin.vcl
/usr/share/doc/varnish/example.vcl
/usr/share/man/man1/varnishadm.1.gz
/usr/share/man/man1/varnishd.1.gz
/usr/share/man/man1/varnishhist.1.gz
/usr/share/man/man1/varnishlog.1.gz
/usr/share/man/man1/varnishncsa.1.gz
/usr/share/man/man1/varnishstat.1.gz
/usr/share/man/man1/varnishtest.1.gz
/usr/share/man/man1/varnishtop.1.gz
/usr/share/man/man3/vmod_directors.3.gz
/usr/share/man/man3/vmod_std.3.gz
/usr/share/man/man7/varnish-cli.7.gz
/usr/share/man/man7/varnish-counters.7.gz
/usr/share/man/man7/vcl.7.gz
/usr/share/man/man7/vsl-query.7.gz
/usr/share/man/man7/vsl.7.gz
/var/lib/varnish
/var/log/varnish
[root@h101 varnish]# 
~~~

**varnish** 的核心内容

~~~
/etc/sysconfig/varnish
/etc/varnish/default.vcl
/usr/bin/varnishadm
/usr/bin/varnishhist
/usr/bin/varnishlog
/usr/bin/varnishncsa
/usr/bin/varnishstat
/usr/bin/varnishtest
/usr/bin/varnishtop
/usr/sbin/varnish_reload_vcl
/usr/sbin/varnishd
~~~

Item     | Explain
-------- | ---
/etc/sysconfig/varnish | 主配置文件
/etc/varnish/default.vcl | 主VCL文件
/usr/bin/varnishadm | 管理软件
/usr/bin/varnishhist | 直方图显示响应统计
/usr/bin/varnishlog | 以本地格式显示事务日志
/usr/bin/varnishncsa | NCSA格式显示事务日志
/usr/bin/varnishstat | 统计计数器
/usr/bin/varnishtest | 测试工具
/usr/bin/varnishtop | 实时top n 事物日志
/usr/sbin/varnishd | 主程序



基本配置与启动
-


修改 **/etc/varnish/default.vcl** 

~~~
[root@h101 varnish]# vim /etc/varnish/default.vcl
[root@h101 varnish]# grep -v "#" /etc/varnish/default.vcl  

vcl 4.0;

backend default {
    .host = "www.boohee.com";
    .port = "80";
}

sub vcl_recv {
}

sub vcl_backend_response {
}

sub vcl_deliver {
}
[root@h101 varnish]# 
~~~

重新加载 **VCL**

~~~
[root@h101 varnish]# /etc/init.d/varnish reload 
Loading vcl from /etc/varnish/default.vcl
Current running config name is reload_2015-08-19T22:45:05
Using new config name reload_2015-08-19T22:46:38
VCL compiled.
VCL 'reload_2015-08-19T22:46:38' now active
available       0 boot
available       0 reload_2015-08-19T22:43:00
available       0 reload_2015-08-19T22:43:13
available       0 reload_2015-08-19T22:45:05
active          0 reload_2015-08-19T22:46:38

Done
[root@h101 varnish]# 
~~~

使用浏览器访问 **http://192.168.100.101:6081/** 或 **http://127.0.0.1:6081/**

就可以获得 **www.boohee.com** 的主页内容

随便点击主页上的链接发现都可以访问，说明 **varnish** 已经成功进行了代理


---

高级主配置
-

**/etc/sysconfig/varnish** 是 **varnish** 的主配置文件

将其中的 **VARNISH_LISTEN_PORT=6081** 改为 **VARNISH_LISTEN_PORT=80**

~~~
[root@h101 varnish]# vim /etc/sysconfig/varnish
[root@h101 varnish]# grep -v "^#"  /etc/sysconfig/varnish  | cat -s 

NFILES=131072

MEMLOCK=82000

NPROCS="unlimited"

RELOAD_VCL=1

VARNISH_VCL_CONF=/etc/varnish/default.vcl
VARNISH_LISTEN_PORT=80
VARNISH_ADMIN_LISTEN_ADDRESS=127.0.0.1
VARNISH_ADMIN_LISTEN_PORT=6082
VARNISH_SECRET_FILE=/etc/varnish/secret
VARNISH_MIN_THREADS=50
VARNISH_MAX_THREADS=1000
VARNISH_THREAD_TIMEOUT=120
VARNISH_STORAGE_SIZE=256M
VARNISH_STORAGE="malloc,${VARNISH_STORAGE_SIZE}"
VARNISH_TTL=120
DAEMON_OPTS="-a ${VARNISH_LISTEN_ADDRESS}:${VARNISH_LISTEN_PORT} \
             -f ${VARNISH_VCL_CONF} \
             -T ${VARNISH_ADMIN_LISTEN_ADDRESS}:${VARNISH_ADMIN_LISTEN_PORT} \
             -t ${VARNISH_TTL} \
             -p thread_pool_min=${VARNISH_MIN_THREADS} \
             -p thread_pool_max=${VARNISH_MAX_THREADS} \
             -p thread_pool_timeout=${VARNISH_THREAD_TIMEOUT} \
             -u varnish -g varnish \
             -S ${VARNISH_SECRET_FILE} \
             -s ${VARNISH_STORAGE}"

[root@h101 varnish]# 
~~~

使用 **/etc/init.d/varnish restart** 重启服务

~~~
[root@h101 varnish]# /etc/init.d/varnish restart 
Stopping Varnish Cache:                                    [  OK  ]
Starting Varnish Cache:                                    [  OK  ]
[root@h101 varnish]#
~~~

> **Tip:** 这种情况下用 **/etc/init.d/varnish reload** 是无法重新加载配置的

这时系统里多出了 **80** 端口

~~~
[root@h101 varnish]# netstat  -ant  | grep :80
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      
tcp        1      1 192.168.1.130:34104         103.245.222.133:80          LAST_ACK    
tcp        1      0 192.168.1.130:59686         140.207.194.83:80           CLOSE_WAIT  
tcp        0      0 :::80                       :::*                        LISTEN      
[root@h101 varnish]# 
~~~

再次使用浏览器访问 **http://192.168.100.101/** 或 **http://127.0.0.1/** 就可以获得 **www.boohee.com** 的主页内容

---

其它配置选项都可以在注释中看到相应解释


Item     | Explain
-------- | ---
**-a**  | 指定监听IP和端口,默认为0.0.0.0:6081
**-f** | 指定主VCL配置文件
**-T** | 远程管理端口和地址,默认为127.0.0.1:6082
**-p** | 设定线程参数，最大最小数，timeout值
**-u** | worker工作身份
**-g** | worker工作组
**-S** | 管理的认证密码文件
**-s** | 存储空间大小

**/etc/sysconfig/varnish** 配置文件中，上面对参数进行设置，下面进行引用 ，也可以注释掉原有的，使用自己的配置 

---

VCL
-

**varnish** 是使用 [VCL][vcl] (Varnish Configuration Language) 处理 HTTP 流的，这种语言非常灵活强大与简洁，它从C语言那里继承了很多东西，阅读起来很像C 和 Perl。

> **Note:**  它包含逻辑判断，但不包含任何循环和跳转

这里不打算对VCL 进行详述，一是我自己还没有完全玩转，免得误人子弟 ; 二是这小篇幅也没法有多深入的讲解；三是这种类型的语言都可以在不明白时翻阅[手册][vcl] ,只用熟悉用到的部分，不必求全解

[Varnish Configuration Language][vcl]

---

常用命令浅析
-

[varnishhist][varnishhist]
-

能产生下列效果的统计直方图

~~~
1:1, n = 35                                                                                                                 h101.temp






                                                               #
                                                               #
                                                               #
                                                               #
                                                               #
                                                               #
                                                               #
                                                               #
                                                              ##
                                                              ##
                                                              ##
                                                              ##
                                                              ##
                                                              ###
                                                              ###
                                                              ####
                                                             ######     #
+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-------------
|1e-6         |1e-5         |1e-4         |1e-3         |1e-2         |1e-1         |1e0          |1e1          |1e2
~~~

---

[varnishlog][varnishlog]
-

可以产生如下效果的日志

~~~
[root@h101 varnish]# varnishlog 
*   << BeReq    >> 32778     
-   Begin          bereq 32777 pass
-   Timestamp      Start: 1440000640.182108 0.000000 0.000000
-   BereqMethod    GET
-   BereqURL       /
-   BereqProtocol  HTTP/1.1
-   BereqHeader    Host: 192.168.100.101
-   BereqHeader    Cache-Control: max-age=0
-   BereqHeader    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
-   BereqHeader    User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36
-   BereqHeader    Accept-Encoding: gzip, deflate, sdch
-   BereqHeader    Accept-Language: zh-CN,zh;q=0.8
-   BereqHeader    Cookie: first_visit_at=MZ8nL61DZHxgLUi1N9QnvNijBJkkjMLa%0A; Hm_lvt_7263598dfd4db0dc29539a51f116b23a=1439995511; Hm_lpvt_7263598dfd4db0dc29539a51f116b23a=1439996815
-   BereqHeader    If-None-Match: "c7ea0cef94d50d04d8974eae94b6eb25"
-   BereqHeader    X-Forwarded-For: 192.168.100.1
-   BereqHeader    X-Varnish: 32778
-   VCL_call       BACKEND_FETCH
...
...

~~~

---

[varnishncsa][varnishncsa]
-

可以产生下列格式的日志

~~~
[root@h101 varnish]# varnishncsa 
192.168.100.1 - - [20/Aug/2015:00:12:43 +0800] "GET http://192.168.100.101/ HTTP/1.1" 200 5634 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"
192.168.100.1 - - [20/Aug/2015:00:12:53 +0800] "GET http://192.168.100.101/forums/ HTTP/1.1" 200 8854 "http://192.168.100.101/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"
192.168.100.1 - - [20/Aug/2015:00:12:53 +0800] "GET http://192.168.100.101/house/boohee/sm25_0504.jpg HTTP/1.1" 200 42560 "http://192.168.100.101/forums/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"
192.168.100.1 - - [20/Aug/2015:00:12:59 +0800] "GET http://192.168.100.101/food/ HTTP/1.1" 200 6379 "http://192.168.100.101/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"
...
...
~~~


---

[varnishstat][varnishstat]
-

可以产生下列形式的统计

~~~
Uptime mgt:   0+01:08:38                                                                                                Hitrate n:       10       18       18
Uptime child: 0+01:08:38                                                                                                   avg(n):   0.0000   0.0000   0.0000

  NAME                                                                               CURRENT       CHANGE      AVERAGE       AVG_10      AVG_100     AVG_1000
MAIN.uptime                                                                             4118         1.00         1.00         1.05         1.06         1.06
MAIN.sess_conn                                                                             6         0.00          .           0.00         0.00	 0.00
MAIN.client_req                                                                            8         0.00          .           0.00         0.00	 0.00
MAIN.backend_conn                                                                          3         0.00          .           0.00         0.00	 0.00
MAIN.backend_reuse                                                                         5         0.00          .           0.00         0.00	 0.00
MAIN.backend_toolate                                                                       2         0.00          .           0.00         0.00	 0.00
MAIN.backend_recycle                                                                       8         0.00          .           0.00         0.00	 0.00
MAIN.fetch_length                                                                          2         0.00          .           0.00         0.00	 0.0
...
...

~~~

---

[varnishtop][varnishtop]
-

可以产生下列形式的 **top** 统计信息


~~~
list length 275                                                                                                                                    h101.temp

     3.20 VCL_return     fetch
     3.20 VCL_return     deliver
     2.49 Begin          sess 0 HTTP/1
     1.93 SessClose      RX_TIMEOUT 5.028
     1.73 ReqMethod      GET
     1.73 BereqMethod    GET
     1.73 VCL_call 	 HASH
     1.73 VCL_call       RECV
     1.73 VCL_call       PASS
     1.73 RespHeader     Age: 0
     1.73 VCL_return     pass
     1.73 ReqProtocol    HTTP/1.1
     1.73 RespProtocol   HTTP/1.1
     1.73 BereqProtocol  HTTP/1.1
     1.73 BerespProtocol HTTP/1.1
     1.73 ObjProtocol    HTTP/1.1
     1.73 VCL_call       DELIVER
     1.73 Debug          XXX REF 1
     1.73 VCL_return     lookup
     1.73 VCL_call	 BACKEND_FETCH
     1.73 ReqStart	 192.168.100.1 57938
...
...
~~~

---

[varnishadm][varnishadm]
-

会提供一个交互界面，可用于对运行中的 **varnish** ，进行交互式管理

~~~
[root@h101 varnish]# varnishadm 
200        
-----------------------------
Varnish Cache CLI 1.0
-----------------------------
Linux,2.6.32-504.el6.x86_64,x86_64,-smalloc,-smalloc,-hcritbit
varnish-4.0.3 revision b8c4a34

Type 'help' for command list.
Type 'quit' to close CLI session.

varnish> 
varnish> 
varnish> help
200        
help [<command>]
ping [<timestamp>]
auth <response>
quit
banner
status
start
stop
vcl.load <configname> <filename>
vcl.inline <configname> <quoted_VCLstring>
vcl.use <configname>
vcl.discard <configname>
vcl.list
param.show [-l] [<param>]
param.set <param> <value>
panic.show
panic.clear
storage.list
vcl.show [-v] <configname>
backend.list [<backend_expression>]
backend.set_health <backend_expression> <state>
ban <field> <operator> <arg> [&& <field> <oper> <arg>]...
ban.list

varnish> 
~~~

其它的命令可以参阅 [The Varnish Reference Manual][The Varnish Reference Manual]


---

[varnish]: https://www.varnish-cache.org/
[varnish.docs]: https://www.varnish-cache.org/docs
[glossary]: https://www.varnish-cache.org/docs/4.0/glossary/index.html
[varnish.repo]: https://www.varnish-cache.org/installation/redhat
[vcl]: https://www.varnish-cache.org/docs/4.0/users-guide/vcl.html
[varnishhist]: https://www.varnish-cache.org/docs/4.0/reference/varnishhist.html
[varnishlog]: https://www.varnish-cache.org/docs/4.0/reference/varnishlog.html#display-varnish-logs
[varnishncsa]: https://www.varnish-cache.org/docs/4.0/reference/varnishncsa.html
[varnishstat]: https://www.varnish-cache.org/docs/4.0/reference/varnishstat.html
[varnishtop]:  https://www.varnish-cache.org/docs/4.0/reference/varnishtop.html
[varnishadm]:  https://www.varnish-cache.org/docs/4.0/reference/varnishadm.html
[The Varnish Reference Manual]:  https://www.varnish-cache.org/docs/4.0/reference/index.html#reference-index
