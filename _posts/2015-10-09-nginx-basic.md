---
layout: post
title: nginx基础
author: wilmosfang
tags:  nginx upgrade
categories:  nginx upgrade
wc: 1092 4425 43801
excerpt: follow me
comments: true
---

---

# 前言

**[Nginx][nginx]** ("engine x") 是一个高性能的 HTTP 和反向代理服务器

以下是目前市面上几个主流开源web服务器的综合对比

![Image_201510091316161.png](/images/nginx/Image_201510091316161.png)

从中可见Nginx的相对优势


> **Tip:** 当前版本 **nginx-1.9.5**

下面分享nginx的基础操作，详细可以参阅 [官方文档][nginxdoc]

---

# 概要

* TOC
{:toc}




---

## 下载



~~~
[root@h102 src]# wget http://nginx.org/download/nginx-1.9.5.tar.gz 
--2015-10-09 14:09:41--  http://nginx.org/download/nginx-1.9.5.tar.gz
Resolving nginx.org... 206.251.255.63, 2606:7100:1:69::3f
Connecting to nginx.org|206.251.255.63|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 884023 (863K) [application/octet-stream]
Saving to: “nginx-1.9.5.tar.gz”

100%[============================================================================================>] 884,023     19.9K/s   in 59s     

2015-10-09 14:10:42 (14.5 KB/s) - “nginx-1.9.5.tar.gz” saved [884023/884023]
 
[root@h102 src]# ls
nginx-1.9.5.tar.gz
[root@h102 src]# 
~~~

---

## 安装

### 解压

~~~
[root@h102 src]# tar -zxvf nginx-1.9.5.tar.gz 
nginx-1.9.5/
nginx-1.9.5/auto/
nginx-1.9.5/conf/
nginx-1.9.5/contrib/
nginx-1.9.5/src/
nginx-1.9.5/configure
nginx-1.9.5/LICENSE
nginx-1.9.5/README
nginx-1.9.5/html/
nginx-1.9.5/man/
nginx-1.9.5/CHANGES.ru
nginx-1.9.5/CHANGES
...
...
nginx-1.9.5/auto/cc/clang
nginx-1.9.5/auto/cc/acc
nginx-1.9.5/auto/cc/bcc
nginx-1.9.5/auto/cc/ccc
nginx-1.9.5/auto/cc/conf
nginx-1.9.5/auto/cc/gcc
nginx-1.9.5/auto/cc/icc
nginx-1.9.5/auto/cc/msvc
nginx-1.9.5/auto/cc/name
nginx-1.9.5/auto/cc/owc
nginx-1.9.5/auto/cc/sunc
[root@h102 src]# ls
nginx-1.9.5  nginx-1.9.5.tar.gz
[root@h102 src]# 
~~~

---

### 配置


~~~
[root@h102 src]# cd nginx-1.9.5
[root@h102 nginx-1.9.5]# ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  man  README  src
[root@h102 nginx-1.9.5]# ./configure 
checking for OS
 + Linux 2.6.32-504.el6.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
checking for gcc -pipe switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for unistd.h ... found
checking for inttypes.h ... found
checking for limits.h ... found
checking for sys/filio.h ... not found
checking for sys/param.h ... found
checking for sys/mount.h ... found
checking for sys/statvfs.h ... found
checking for crypt.h ... found
checking for Linux specific features
checking for epoll ... found
checking for EPOLLRDHUP ... found
checking for O_PATH ... not found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for sched_setaffinity() ... found
checking for crypt_r() ... found
checking for sys/vfs.h ... found
checking for nobody group ... found
checking for poll() ... found
checking for /dev/poll ... not found
checking for kqueue ... not found
checking for crypt() ... not found
checking for crypt() in libcrypt ... found
checking for F_READAHEAD ... not found
checking for posix_fadvise() ... found
checking for O_DIRECT ... found
checking for F_NOCACHE ... not found
checking for directio() ... not found
checking for statfs() ... found
checking for statvfs() ... found
checking for dlopen() ... not found
checking for dlopen() in libdl ... found
checking for sched_yield() ... found
checking for SO_SETFIB ... not found
checking for SO_REUSEPORT ... found
checking for SO_ACCEPTFILTER ... not found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... not found
checking for TCP_INFO ... found
checking for accept4() ... found
checking for eventfd() ... found
checking for int size ... 4 bytes
checking for long size ... 8 bytes
checking for long long size ... 8 bytes
checking for void * size ... 8 bytes
checking for uint64_t ... found
checking for sig_atomic_t ... found
checking for sig_atomic_t size ... 4 bytes
checking for socklen_t ... found
checking for in_addr_t ... found
checking for in_port_t ... found
checking for rlim_t ... found
checking for uintptr_t ... uintptr_t found
checking for system byte ordering ... little endian
checking for size_t size ... 8 bytes
checking for off_t size ... 8 bytes
checking for time_t size ... 8 bytes
checking for setproctitle() ... not found
checking for pread() ... found
checking for pwrite() ... found
checking for sys_nerr ... found
checking for localtime_r() ... found
checking for posix_memalign() ... found
checking for memalign() ... found
checking for mmap(MAP_ANON|MAP_SHARED) ... found
checking for mmap("/dev/zero", MAP_SHARED) ... found
checking for System V shared memory ... found
checking for POSIX semaphores ... not found
checking for POSIX semaphores in libpthread ... found
checking for struct msghdr.msg_control ... found
checking for ioctl(FIONBIO) ... found
checking for struct tm.tm_gmtoff ... found
checking for struct dirent.d_namlen ... not found
checking for struct dirent.d_type ... found
checking for sysconf(_SC_NPROCESSORS_ONLN) ... found
checking for openat(), fstatat() ... found
checking for getaddrinfo() ... found
checking for PCRE library ... not found
checking for PCRE library in /usr/local/ ... not found
checking for PCRE library in /usr/include/pcre/ ... not found
checking for PCRE library in /usr/pkg/ ... not found
checking for PCRE library in /opt/local/ ... not found

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

[root@h102 nginx-1.9.5]# 
~~~

#### 报错1

**the HTTP rewrite module requires the PCRE library**

缺少 **PCRE** 库

解决办法是：

* 1.使用 **--without-http_rewrite_module** 不添加此模块
* 2.安装 **PCRE** 模块到系统中(其实是安装此模块的开发包)

#### 解决依赖

由此可见 **pcre** 已经在系统中有安装，只是它的开发包没有

~~~
[root@h102 nginx-1.9.5]# yum list all | grep -i pcre
pcre.x86_64                             7.8-6.el6                @anaconda-CentOS-201410241409.x86_64/6.6
ghc-pcre-light.x86_64                   0.4-7.el6                epel           
ghc-pcre-light-devel.x86_64             0.4-7.el6                epel           
pcre.i686                               7.8-7.el6                base           
pcre.x86_64                             7.8-7.el6                base           
pcre-devel.i686                         7.8-7.el6                base           
pcre-devel.x86_64                       7.8-7.el6                base           
pcre-static.x86_64                      7.8-7.el6                base           
[root@h102 nginx-1.9.5]# 
~~~

对此包进行更新，并且使用yum安装它的开发包

~~~
[root@h102 nginx-1.9.5]# yum install  pcre-devel.x86_64 pcre.x86_64  
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * epel: mirrors.opencas.cn
 * extras: mirrors.pubyun.com
 * updates: mirrors.pubyun.com
Resolving Dependencies
--> Running transaction check
---> Package pcre.x86_64 0:7.8-6.el6 will be updated
---> Package pcre.x86_64 0:7.8-7.el6 will be an update
---> Package pcre-devel.x86_64 0:7.8-7.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                           Arch                          Version                          Repository                   Size
====================================================================================================================================
Installing:
 pcre-devel                        x86_64                        7.8-7.el6                        base                        320 k
Updating:
 pcre                              x86_64                        7.8-7.el6                        base                        196 k

Transaction Summary
====================================================================================================================================
Install       1 Package(s)
Upgrade       1 Package(s)

Total download size: 516 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): pcre-7.8-7.el6.x86_64.rpm                                                                             | 196 kB     00:00     
(2/2): pcre-devel-7.8-7.el6.x86_64.rpm                                                                       | 320 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               797 kB/s | 516 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
** Found 6 pre-existing rpmdb problem(s), 'yum check' output follows:
perl-DBD-MySQL-4.013-3.el6.x86_64 has missing requires of libmysqlclient.so.16()(64bit)
perl-DBD-MySQL-4.013-3.el6.x86_64 has missing requires of libmysqlclient.so.16(libmysqlclient_16)(64bit)
ruby-mysql-2.8.2-1.el6.x86_64 has missing requires of libmysqlclient.so.16()(64bit)
ruby-mysql-2.8.2-1.el6.x86_64 has missing requires of libmysqlclient.so.16(libmysqlclient_16)(64bit)
ruby193-rubygem-mysql2-0.3.11-4.el6.x86_64 has missing requires of libmysqlclient_r.so.16()(64bit)
ruby193-rubygem-mysql2-0.3.11-4.el6.x86_64 has missing requires of libmysqlclient_r.so.16(libmysqlclient_16)(64bit)
  Updating   : pcre-7.8-7.el6.x86_64                                                                                            1/3 
  Installing : pcre-devel-7.8-7.el6.x86_64                                                                                      2/3 
  Cleanup    : pcre-7.8-6.el6.x86_64                                                                                            3/3 
  Verifying  : pcre-7.8-7.el6.x86_64                                                                                            1/3 
  Verifying  : pcre-devel-7.8-7.el6.x86_64                                                                                      2/3 
  Verifying  : pcre-7.8-6.el6.x86_64                                                                                            3/3 

Installed:
  pcre-devel.x86_64 0:7.8-7.el6                                                                                                     

Updated:
  pcre.x86_64 0:7.8-7.el6                                                                                                           

Complete!
[root@h102 nginx-1.9.5]#
~~~

重新配置，就可以配置成功

~~~
[root@h102 nginx-1.9.5]# ./configure 
checking for OS
 + Linux 2.6.32-504.el6.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
checking for gcc -pipe switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for unistd.h ... found
checking for inttypes.h ... found
checking for limits.h ... found
checking for sys/filio.h ... not found
checking for sys/param.h ... found
checking for sys/mount.h ... found
checking for sys/statvfs.h ... found
checking for crypt.h ... found
checking for Linux specific features
checking for epoll ... found
checking for EPOLLRDHUP ... found
checking for O_PATH ... not found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for sched_setaffinity() ... found
checking for crypt_r() ... found
checking for sys/vfs.h ... found
checking for nobody group ... found
checking for poll() ... found
checking for /dev/poll ... not found
checking for kqueue ... not found
checking for crypt() ... not found
checking for crypt() in libcrypt ... found
checking for F_READAHEAD ... not found
checking for posix_fadvise() ... found
checking for O_DIRECT ... found
checking for F_NOCACHE ... not found
checking for directio() ... not found
checking for statfs() ... found
checking for statvfs() ... found
checking for dlopen() ... not found
checking for dlopen() in libdl ... found
checking for sched_yield() ... found
checking for SO_SETFIB ... not found
checking for SO_REUSEPORT ... found
checking for SO_ACCEPTFILTER ... not found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... not found
checking for TCP_INFO ... found
checking for accept4() ... found
checking for eventfd() ... found
checking for int size ... 4 bytes
checking for long size ... 8 bytes
checking for long long size ... 8 bytes
checking for void * size ... 8 bytes
checking for uint64_t ... found
checking for sig_atomic_t ... found
checking for sig_atomic_t size ... 4 bytes
checking for socklen_t ... found
checking for in_addr_t ... found
checking for in_port_t ... found
checking for rlim_t ... found
checking for uintptr_t ... uintptr_t found
checking for system byte ordering ... little endian
checking for size_t size ... 8 bytes
checking for off_t size ... 8 bytes
checking for time_t size ... 8 bytes
checking for setproctitle() ... not found
checking for pread() ... found
checking for pwrite() ... found
checking for sys_nerr ... found
checking for localtime_r() ... found
checking for posix_memalign() ... found
checking for memalign() ... found
checking for mmap(MAP_ANON|MAP_SHARED) ... found
checking for mmap("/dev/zero", MAP_SHARED) ... found
checking for System V shared memory ... found
checking for POSIX semaphores ... not found
checking for POSIX semaphores in libpthread ... found
checking for struct msghdr.msg_control ... found
checking for ioctl(FIONBIO) ... found
checking for struct tm.tm_gmtoff ... found
checking for struct dirent.d_namlen ... not found
checking for struct dirent.d_type ... found
checking for sysconf(_SC_NPROCESSORS_ONLN) ... found
checking for openat(), fstatat() ... found
checking for getaddrinfo() ... found
checking for PCRE library ... found
checking for PCRE JIT support ... not found
checking for md5 in system md library ... not found
checking for md5 in system md5 library ... not found
checking for md5 in system OpenSSL crypto library ... found
checking for sha1 in system md library ... not found
checking for sha1 in system OpenSSL crypto library ... found
checking for zlib library ... found
creating objs/Makefile

Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + md5: using system crypto library
  + sha1: using system crypto library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

[root@h102 nginx-1.9.5]# echo $?
0
[root@h102 nginx-1.9.5]# 
~~~

> **Tip:** 以下是可配置的选项，不加参数会按默认特性配置

~~~
[root@h102 nginx-1.9.5]# ./configure  --help 

  --help                             print this message

  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

  --user=USER                        set non-privileged user for
                                     worker processes
  --group=GROUP                      set non-privileged group for
                                     worker processes

  --build=NAME                       set build name
  --builddir=DIR                     set build directory

  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --with-threads                     enable thread pool support

  --with-file-aio                    enable file AIO support
  --with-ipv6                        enable IPv6 support

  --with-http_ssl_module             enable ngx_http_ssl_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_module
  --with-http_xslt_module            enable ngx_http_xslt_module
  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_module
  --with-http_dav_module             enable ngx_http_dav_module
  --with-http_flv_module             enable ngx_http_flv_module
  --with-http_mp4_module             enable ngx_http_mp4_module
  --with-http_gunzip_module          enable ngx_http_gunzip_module
  --with-http_gzip_static_module     enable ngx_http_gzip_static_module
  --with-http_auth_request_module    enable ngx_http_auth_request_module
  --with-http_random_index_module    enable ngx_http_random_index_module
  --with-http_secure_link_module     enable ngx_http_secure_link_module
  --with-http_degradation_module     enable ngx_http_degradation_module
  --with-http_stub_status_module     enable ngx_http_stub_status_module

  --without-http_charset_module      disable ngx_http_charset_module
  --without-http_gzip_module         disable ngx_http_gzip_module
  --without-http_ssi_module          disable ngx_http_ssi_module
  --without-http_userid_module       disable ngx_http_userid_module
  --without-http_access_module       disable ngx_http_access_module
  --without-http_auth_basic_module   disable ngx_http_auth_basic_module
  --without-http_autoindex_module    disable ngx_http_autoindex_module
  --without-http_geo_module          disable ngx_http_geo_module
  --without-http_map_module          disable ngx_http_map_module
  --without-http_split_clients_module disable ngx_http_split_clients_module
  --without-http_referer_module      disable ngx_http_referer_module
  --without-http_rewrite_module      disable ngx_http_rewrite_module
  --without-http_proxy_module        disable ngx_http_proxy_module
  --without-http_fastcgi_module      disable ngx_http_fastcgi_module
  --without-http_uwsgi_module        disable ngx_http_uwsgi_module
  --without-http_scgi_module         disable ngx_http_scgi_module
  --without-http_memcached_module    disable ngx_http_memcached_module
  --without-http_limit_conn_module   disable ngx_http_limit_conn_module
  --without-http_limit_req_module    disable ngx_http_limit_req_module
  --without-http_empty_gif_module    disable ngx_http_empty_gif_module
  --without-http_browser_module      disable ngx_http_browser_module
  --without-http_upstream_hash_module
                                     disable ngx_http_upstream_hash_module
  --without-http_upstream_ip_hash_module
                                     disable ngx_http_upstream_ip_hash_module
  --without-http_upstream_least_conn_module
                                     disable ngx_http_upstream_least_conn_module
  --without-http_upstream_keepalive_module
                                     disable ngx_http_upstream_keepalive_module
  --without-http_upstream_zone_module
                                     disable ngx_http_upstream_zone_module

  --with-http_perl_module            enable ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --http-log-path=PATH               set http access log pathname
  --http-client-body-temp-path=PATH  set path to store
                                     http client request body temporary files
  --http-proxy-temp-path=PATH        set path to store
                                     http proxy temporary files
  --http-fastcgi-temp-path=PATH      set path to store
                                     http fastcgi temporary files
  --http-uwsgi-temp-path=PATH        set path to store
                                     http uwsgi temporary files
  --http-scgi-temp-path=PATH         set path to store
                                     http scgi temporary files

  --without-http                     disable HTTP server
  --without-http-cache               disable HTTP cache

  --with-mail                        enable POP3/IMAP4/SMTP proxy module
  --with-mail_ssl_module             enable ngx_mail_ssl_module
  --without-mail_pop3_module         disable ngx_mail_pop3_module
  --without-mail_imap_module         disable ngx_mail_imap_module
  --without-mail_smtp_module         disable ngx_mail_smtp_module

  --with-stream                      enable TCP proxy module
  --with-stream_ssl_module           enable ngx_stream_ssl_module
  --without-stream_limit_conn_module disable ngx_stream_limit_conn_module
  --without-stream_access_module     disable ngx_stream_access_module
  --without-stream_upstream_hash_module
                                     disable ngx_stream_upstream_hash_module
  --without-stream_upstream_least_conn_module
                                     disable ngx_stream_upstream_least_conn_module
  --without-stream_upstream_zone_module
                                     disable ngx_stream_upstream_zone_module

  --with-google_perftools_module     enable ngx_google_perftools_module
  --with-cpp_test_module             enable ngx_cpp_test_module

  --add-module=PATH                  enable an external module

  --with-cc=PATH                     set C compiler pathname
  --with-cpp=PATH                    set C preprocessor pathname
  --with-cc-opt=OPTIONS              set additional C compiler options
  --with-ld-opt=OPTIONS              set additional linker options
  --with-cpu-opt=CPU                 build for the specified CPU, valid values:
                                     pentium, pentiumpro, pentium3, pentium4,
                                     athlon, opteron, sparc32, sparc64, ppc64

  --without-pcre                     disable PCRE library usage
  --with-pcre                        force PCRE library usage
  --with-pcre=DIR                    set path to PCRE library sources
  --with-pcre-opt=OPTIONS            set additional build options for PCRE
  --with-pcre-jit                    build PCRE with JIT compilation support

  --with-md5=DIR                     set path to md5 library sources
  --with-md5-opt=OPTIONS             set additional build options for md5
  --with-md5-asm                     use md5 assembler sources

  --with-sha1=DIR                    set path to sha1 library sources
  --with-sha1-opt=OPTIONS            set additional build options for sha1
  --with-sha1-asm                    use sha1 assembler sources

  --with-zlib=DIR                    set path to zlib library sources
  --with-zlib-opt=OPTIONS            set additional build options for zlib
  --with-zlib-asm=CPU                use zlib assembler sources optimized
                                     for the specified CPU, valid values:
                                     pentium, pentiumpro

  --with-libatomic                   force libatomic_ops library usage
  --with-libatomic=DIR               set path to libatomic_ops library sources

  --with-openssl=DIR                 set path to OpenSSL library sources
  --with-openssl-opt=OPTIONS         set additional build options for OpenSSL

  --with-debug                       enable debug logging

[root@h102 nginx-1.9.5]# 
~~~

---

### 编译

~~~
[root@h102 nginx-1.9.5]# make 
make -f objs/Makefile
make[1]: Entering directory `/usr/local/src/nginx-1.9.5'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs \
		-o objs/src/core/nginx.o \
		src/core/nginx.c
...
...
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs \
		-o objs/ngx_modules.o \
		objs/ngx_modules.c
cc -o objs/nginx \
	objs/src/core/nginx.o \
	objs/src/core/ngx_log.o \
...
...
	objs/src/http/modules/ngx_http_upstream_keepalive_module.o \
	objs/src/http/modules/ngx_http_upstream_zone_module.o \
	objs/ngx_modules.o \
	-lpthread -lcrypt -lpcre -lcrypto -lcrypto -lz
make[1]: Leaving directory `/usr/local/src/nginx-1.9.5'
make -f objs/Makefile manpage
make[1]: Entering directory `/usr/local/src/nginx-1.9.5'
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
		-e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
		-e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
		-e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
		< man/nginx.8 > objs/nginx.8
make[1]: Leaving directory `/usr/local/src/nginx-1.9.5'
[root@h102 nginx-1.9.5]# echo $?
0
[root@h102 nginx-1.9.5]# 
~~~

---

### 安装


~~~
[root@h102 nginx-1.9.5]# make install 
make -f objs/Makefile install
make[1]: Entering directory `/usr/local/src/nginx-1.9.5'
test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'
test -d '/usr/local/nginx/sbin' 		|| mkdir -p '/usr/local/nginx/sbin'
test ! -f '/usr/local/nginx/sbin/nginx' 		|| mv '/usr/local/nginx/sbin/nginx' 			'/usr/local/nginx/sbin/nginx.old'
cp objs/nginx '/usr/local/nginx/sbin/nginx'
test -d '/usr/local/nginx/conf' 		|| mkdir -p '/usr/local/nginx/conf'
cp conf/koi-win '/usr/local/nginx/conf'
cp conf/koi-utf '/usr/local/nginx/conf'
cp conf/win-utf '/usr/local/nginx/conf'
test -f '/usr/local/nginx/conf/mime.types' 		|| cp conf/mime.types '/usr/local/nginx/conf'
cp conf/mime.types '/usr/local/nginx/conf/mime.types.default'
test -f '/usr/local/nginx/conf/fastcgi_params' 		|| cp conf/fastcgi_params '/usr/local/nginx/conf'
cp conf/fastcgi_params 		'/usr/local/nginx/conf/fastcgi_params.default'
test -f '/usr/local/nginx/conf/fastcgi.conf' 		|| cp conf/fastcgi.conf '/usr/local/nginx/conf'
cp conf/fastcgi.conf '/usr/local/nginx/conf/fastcgi.conf.default'
test -f '/usr/local/nginx/conf/uwsgi_params' 		|| cp conf/uwsgi_params '/usr/local/nginx/conf'
cp conf/uwsgi_params 		'/usr/local/nginx/conf/uwsgi_params.default'
test -f '/usr/local/nginx/conf/scgi_params' 		|| cp conf/scgi_params '/usr/local/nginx/conf'
cp conf/scgi_params 		'/usr/local/nginx/conf/scgi_params.default'
test -f '/usr/local/nginx/conf/nginx.conf' 		|| cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf'
cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf.default'
test -d '/usr/local/nginx/logs' 		|| mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/logs' || 		mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/html' 		|| cp -R html '/usr/local/nginx'
test -d '/usr/local/nginx/logs' || 		mkdir -p '/usr/local/nginx/logs'
make[1]: Leaving directory `/usr/local/src/nginx-1.9.5'
[root@h102 nginx-1.9.5]# echo $?
0
[root@h102 nginx-1.9.5]# 
~~~

---

## Nginx操作


nginx的目录结构

~~~
[root@h102 local]# tree /usr/local/nginx/
/usr/local/nginx/
├── conf
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html
│   ├── 50x.html
│   └── index.html
├── logs
└── sbin
    └── nginx

4 directories, 18 files
[root@h102 local]# 
~~~

---

### 启动

查看配置

~~~
[root@h102 conf]#  grep -v  "#" /usr/local/nginx/conf/nginx.conf  | grep -v "^$" 
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
[root@h102 conf]# 
~~~

启动nginx

~~~
[root@h102 logs]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
[root@h102 logs]# 
~~~

两种方法查看进程号

* 1.使用 **ps**
* 2.使用 **pid** 文件

~~~
[root@h102 logs]# ps fuax | grep nginx 
root     11761  0.0  0.0 103256   824 pts/0    S+   15:23   0:00          \_ grep nginx
root     11756  0.0  0.0  24316   676 ?        Ss   15:23   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   11757  0.0  0.0  24728  1244 ?        S    15:23   0:00  \_ nginx: worker process                                          
[root@h102 logs]# cat /usr/local/nginx/logs/nginx.pid 
11756
[root@h102 logs]# 
~~~

此刻就可以使用浏览器进行访问了

![Image_201510091537433.png](/images/nginx/Image_201510091537433.png)

> **Note:** 本地要打开防火墙 **-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT**



---

### 停止

nginx是通过给Nginx主进程发系统信号的方式来停止的

#### 从容停止

~~~
[root@h102 logs]# ps faux | grep nginx 
root     11909  0.0  0.0 103256   828 pts/0    S+   15:31   0:00          \_ grep nginx
root     11756  0.0  0.0  24316   676 ?        Ss   15:23   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   11757  0.0  0.0  24728  1244 ?        S    15:23   0:00  \_ nginx: worker process                                          
[root@h102 logs]# kill -QUIT 11756 
[root@h102 logs]# ps faux | grep nginx 
root     11914  0.0  0.0 103256   824 pts/0    S+   15:32   0:00          \_ grep nginx
[root@h102 logs]# 
~~~

#### 快速停止

~~~
[root@h102 logs]# ps faux | grep nginx 
root     11947  0.0  0.0 103256   828 pts/0    S+   15:42   0:00          \_ grep nginx
root     11923  0.0  0.0  24316   676 ?        Ss   15:34   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   11924  0.0  0.0  24728  1512 ?        S    15:34   0:00  \_ nginx: worker process                                          
[root@h102 logs]# kill -TERM 11923
[root@h102 logs]# ps faux | grep nginx 
root     11950  0.0  0.0 103256   824 pts/0    S+   15:42   0:00          \_ grep nginx
[root@h102 logs]# 
~~~


~~~
[root@h102 logs]# ps faux | grep nginx 
root     11961  0.0  0.0 103256   828 pts/0    S+   15:43   0:00          \_ grep nginx
root     11957  0.0  0.0  24316   680 ?        Ss   15:43   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   11958  0.0  0.0  24728  1516 ?        S    15:43   0:00  \_ nginx: worker process                                          
[root@h102 logs]# kill  -INT 11957 
[root@h102 logs]# ps faux | grep nginx 
root     11964  0.0  0.0 103256   828 pts/0    S+   15:43   0:00          \_ grep nginx
[root@h102 logs]# 
~~~

#### 强制停止


~~~
[root@h102 logs]# ps faux | grep nginx 
root     11974  0.0  0.0 103256   828 pts/0    S+   15:45   0:00          \_ grep nginx
root     11971  0.0  0.0  24316   680 ?        Ss   15:45   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   11972  0.0  0.0  24728  1248 ?        S    15:45   0:00  \_ nginx: worker process                                          
[root@h102 logs]# kill -9 11971 
[root@h102 logs]# ps faux | grep nginx 
root     11976  0.0  0.0 103256   828 pts/0    S+   15:45   0:00          \_ grep nginx
nobody   11972  0.0  0.0  24728  1340 ?        S    15:45   0:00 nginx: worker process                                          
[root@h102 logs]#
~~~

强制停止比较暴力，会导致worker进程仍然停留在系统中，并且还能被访问

---

### 检查配置

重启之前最好先检查一下配置，避免由于配置不合理而导致的服务不可用

~~~
[root@h102 nginx]# sbin/nginx -h 
nginx version: nginx/1.9.5
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

[root@h102 nginx]# sbin/nginx  -t -c conf/nginx.conf
nginx: [emerg] unexpected end of file, expecting ";" or "}" in /usr/local/nginx/conf/nginx.conf:120
nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
[root@h102 nginx]# sbin/nginx  -t -c conf/nginx.conf
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@h102 nginx]#
~~~

---

### 重启

~~~
[root@h102 nginx]# ps fuax | grep nginx 
root     49758  0.0  0.0 103256   828 pts/0    S+   16:40   0:00          \_ grep nginx
root     44512  0.0  0.0  24316   676 ?        Ss   16:39   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   44513  0.0  0.0  24728  1244 ?        S    16:39   0:00  \_ nginx: worker process                                          
[root@h102 nginx]# kill -HUP 44512 
[root@h102 nginx]# ps fuax | grep nginx 
root     62744  0.0  0.0 103256   828 pts/0    S+   16:40   0:00          \_ grep nginx
root     44512  0.0  0.0  24316  1352 ?        Ss   16:39   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   60962  0.0  0.0  24732  1364 ?        S    16:40   0:00  \_ nginx: worker process                                          
[root@h102 nginx]# 
~~~

---

## Nginx信号控制

  
信号     | 控制
-------- | ---
**TERM** **INT** | 快速关闭
**QUIT** | 从容关闭
**HUP**  | 平滑重启，重新加载配置文件
**USR1** | 重新打开日志文件，在切割日志时用途较大
**USR2** | 平滑升级可执行程序
**WINCH** | 从容关闭工作进程


## Nginx 版本变更


### 准备好另一个版本的Nginx


根据上面的步骤准备好另一个版本的Nginx

~~~
wget  http://nginx.org/download/nginx-1.8.0.tar.gz
tar -zxvf nginx-1.8.0.tar.gz 
./configure 
make 
~~~

此时 **objs** 目录中有一个不同版本的 **nginx** 

~~~
[root@h102 nginx-1.8.0]# ll objs/
total 3276
-rw-r--r-- 1 root root   16436 Oct  9 19:15 autoconf.err
-rw-r--r-- 1 root root   36750 Oct  9 19:15 Makefile
-rwxr-xr-x 1 root root 3234515 Oct  9 19:15 nginx
-rw-r--r-- 1 root root    5253 Oct  9 19:15 nginx.8
-rw-r--r-- 1 root root    5952 Oct  9 19:15 ngx_auto_config.h
-rw-r--r-- 1 root root     657 Oct  9 19:14 ngx_auto_headers.h
-rw-r--r-- 1 root root    3812 Oct  9 19:15 ngx_modules.c
-rw-r--r-- 1 root root   30344 Oct  9 19:15 ngx_modules.o
drwxr-xr-x 8 root root    4096 Oct  9 19:15 src
[root@h102 nginx-1.8.0]# file objs/nginx 
objs/nginx: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, not stripped
[root@h102 nginx-1.8.0]# objs/nginx -v 
nginx version: nginx/1.8.0
[root@h102 nginx-1.8.0]# objs/nginx -V
nginx version: nginx/1.8.0
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
configure arguments:
[root@h102 nginx-1.8.0]# 
~~~

查看当前运行Nginx版本

~~~
[root@h102 sbin]# pwd
/usr/local/nginx/sbin
[root@h102 sbin]# ls
nginx
[root@h102 sbin]# du -sh nginx 
3.2M	nginx
[root@h102 sbin]# fuser nginx 
nginx:                5500e  5501e
[root@h102 sbin]# ./nginx  -V 
nginx version: nginx/1.9.5
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
configure arguments:
[root@h102 sbin]# ps faux | grep nginx 
root      5635  0.0  0.0 103256   828 pts/0    S+   20:21   0:00          \_ grep nginx
root      5500  0.0  0.0  24316   676 ?        Ss   20:01   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5501  0.0  0.0  24728  1244 ?        S    20:01   0:00  \_ nginx: worker process        
[root@h102 sbin]# 
~~~

---

### 替换Nginx可执行文件

~~~
[root@h102 sbin]# ls
nginx
[root@h102 sbin]# mv nginx  nginx.old
[root@h102 sbin]# ls
nginx.old
[root@h102 sbin]# fuser nginx.old 
nginx.old:            5500e  5501e
[root@h102 sbin]# cp /usr/local/src/nginx-1.8.0/objs/nginx . 
[root@h102 sbin]# ls
nginx  nginx.old
[root@h102 sbin]# ./nginx -V 
nginx version: nginx/1.8.0
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
configure arguments:
[root@h102 sbin]# 
~~~

### 使用新版本Nginx测试配置


~~~
[root@h102 sbin]# ./nginx -t -c /usr/local/nginx/conf/nginx.conf
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@h102 sbin]# 
~~~

### 平滑替换Nginx可执行程序

~~~
[root@h102 sbin]# kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
[root@h102 sbin]# cat /usr/local/nginx/logs/nginx.pid
5651
[root@h102 sbin]# ll /usr/local/nginx/logs/
total 16
-rw-r--r-- 1 root root 2061 Oct  9 17:07 access.log
-rw-r--r-- 1 root root  419 Oct  9 20:23 error.log
-rw-r--r-- 1 root root    5 Oct  9 20:23 nginx.pid
-rw-r--r-- 1 root root    5 Oct  9 20:01 nginx.pid.oldbin
[root@h102 sbin]# cat /usr/local/nginx/logs/nginx.pid
5651
[root@h102 sbin]# cat /usr/local/nginx/logs/nginx.pid.oldbin 
5500
[root@h102 sbin]# ps faux | grep nginx 
root      5659  0.0  0.0 103256   824 pts/0    S+   20:24   0:00          \_ grep nginx
root      5500  0.0  0.0  24316   848 ?        Ss   20:01   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5501  0.0  0.0  24728  1244 ?        S    20:01   0:00  \_ nginx: worker process        
root      5651  0.0  0.0  24316  1820 ?        S    20:23   0:00  \_ nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5653  0.0  0.0  24740  1252 ?        S    20:23   0:00      \_ nginx: worker process        
[root@h102 sbin]# 
~~~

此时两个master并存


### 从容关闭旧版本Nginx worker进程


~~~
[root@h102 sbin]# ps fuax | grep nginx 
root      5730  0.0  0.0 103256   828 pts/0    S+   20:43   0:00          \_ grep nginx
root      5500  0.0  0.0  24316   848 ?        Ss   20:01   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5501  0.0  0.0  24728  1244 ?        S    20:01   0:00  \_ nginx: worker process        
root      5651  0.0  0.0  24316  1820 ?        S    20:23   0:00  \_ nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5653  0.0  0.0  24740  1532 ?        S    20:23   0:00      \_ nginx: worker process 
[root@h102 sbin]# cat /usr/local/nginx/logs/nginx.pid.oldbin 
5500
[root@h102 sbin]# kill -WINCH `cat /usr/local/nginx/logs/nginx.pid.oldbin`
[root@h102 sbin]# ps fuax | grep nginx 
root      5738  0.0  0.0 103256   828 pts/0    S+   20:45   0:00          \_ grep nginx
root      5500  0.0  0.0  24316   852 ?        Ss   20:01   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
root      5651  0.0  0.0  24316  1820 ?        S    20:23   0:00  \_ nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5653  0.0  0.0  24740  1532 ?        S    20:23   0:00      \_ nginx: worker process        
[root@h102 sbin]# 
~~~

---

### 重新启动旧版本Nginx worker进程


~~~
[root@h102 sbin]# kill -HUP `cat /usr/local/nginx/logs/nginx.pid.oldbin` 
[root@h102 sbin]# ps fuax | grep nginx 
root      5748  0.0  0.0 103256   828 pts/0    S+   20:49   0:00          \_ grep nginx
root      5500  0.0  0.0  24316   852 ?        Ss   20:01   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
root      5651  0.0  0.0  24316  1820 ?        S    20:23   0:00  \_ nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5653  0.0  0.0  24740  1532 ?        S    20:23   0:00  |   \_ nginx: worker process        
nobody    5746  0.0  0.0  24728  1244 ?        S    20:49   0:00  \_ nginx: worker process        
[root@h102 sbin]# 
~~~

---

### 彻底关闭旧版本Nginx worker进程

~~~
[root@h102 sbin]# kill -WINCH `cat /usr/local/nginx/logs/nginx.pid.oldbin` 
[root@h102 sbin]# ps fuax | grep nginx 
root      5759  0.0  0.0 103256   828 pts/0    S+   20:52   0:00          \_ grep nginx
root      5500  0.0  0.0  24316   852 ?        Ss   20:01   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
root      5651  0.0  0.0  24316  1820 ?        S    20:23   0:00  \_ nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5653  0.0  0.0  24740  1532 ?        S    20:23   0:00      \_ nginx: worker process        
[root@h102 sbin]# kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin` 
[root@h102 sbin]# ps fuax | grep nginx 
root      5762  0.0  0.0 103256   828 pts/0    S+   20:52   0:00          \_ grep nginx
root      5651  0.0  0.0  24316  1820 ?        S    20:23   0:00 nginx: master process sbin/nginx -c conf/nginx.conf
nobody    5653  0.0  0.0  24740  1532 ?        S    20:23   0:00  \_ nginx: worker process        
[root@h102 sbin]# 
~~~

版本切换成功,整个切换过程服务都是可用状态


> **Tip:** 有没有注意到其实我是在进行版本降级 **从 nginx-1.9.5 降到了 nginx-1.8.0** ，所以其实和版本是否更新无关，可以更新也可以更旧





---

[nginx]:http://nginx.org/
[nginxdoc]:http://nginx.org/en/docs/
