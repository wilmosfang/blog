---
layout:  post
title:  Nginx 模块
author:  wilmosfang
tags:   nginx 
categories:  nginx
wc: 838 3028 42238
excerpt: nginx 的模块加载方法，与查看方法 
comments: true
---


# 前言

**[Nginx (engine x)][nginx]** 可以作为 HTTP 和反向代理服务器，也可以作为邮件代理和普通的 TCP/UDP 代理服务器
 
由于其事件驱动的异步通讯机制在当前的web应用场景中性能非常卓越，所以被广泛使用，相关基础可以参考之前的一篇文章 **[nginx基础][nginx_blog]**

> **Tip:** 当前最新版本为 **nginx-1.11.2** 于 2016-07-05 发布

**[Tengine][tengine]** 是由淘宝网发起的Web服务器项目，它在 **[Nginx][nginx]** 的基础上，针对大访问量网站的需求，添加了很多高级功能和特性

相关基础可以参考之前的一篇文章 **[Tengine基础][tengine_blog]**

> **Tip:** 当前最新版本为 **Tengine-2.1.2** 于 2015-12-31 发布

**模块化** 是代码世界中一个极其重要的思想，将特定功能的代码组织在一起，通过统一的接口与主体对接，这样不仅精简了设计，明确了主体逻辑，让软件架构变得更健壮，甚至还能动态地扩展软件能力，和定制化缩减冗余功能，这样的设计可以更好的适应复杂多变的环境需求

很多优秀的软件都引入了这个思想，**[Nginx][nginx]** 也不例外，这里通过 **[Tengine][tengine]** 来介绍一下加载模块的相关基础，详细可以参考 **[Tengine 官方文档][tengine_doc]** 和 **[Nginx 官方文档][nginx_doc]**

---


# 概要

* TOC
{:toc}


---

## 环境

~~~
[root@iZ11b0k6s5lZ ~]# hostnamectl 
   Static hostname: iZ11b0k6s5lZ
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 7d26c16f128042a684ea474c9e2c240f
           Boot ID: 8169ae94d80c4ae4b84c036d604ed2ce
    Virtualization: xen
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.el7.x86_64
      Architecture: x86-64
[root@iZ11b0k6s5lZ ~]#  
~~~


---

## 下载

**[Tengine][tengine]** 的 **[下载地址][tengine_dl]**

~~~
[root@iZ11b0k6s5lZ src]# wget http://tengine.taobao.org/download/tengine-2.1.2.tar.gz
--2016-07-08 11:54:00--  http://tengine.taobao.org/download/tengine-2.1.2.tar.gz
Resolving tengine.taobao.org (tengine.taobao.org)... 120.55.149.135
Connecting to tengine.taobao.org (tengine.taobao.org)|120.55.149.135|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2137295 (2.0M) [application/octet-stream]
Saving to: ‘tengine-2.1.2.tar.gz’

100%[=======================================================================================>] 2,137,295   1.52MB/s   in 1.3s   

2016-07-08 11:54:01 (1.52 MB/s) - ‘tengine-2.1.2.tar.gz’ saved [2137295/2137295]

[root@iZ11b0k6s5lZ src]# md5sum tengine-2.1.2.tar.gz 
7f898a0dbb5162ff1eb19aeb9d53bec3  tengine-2.1.2.tar.gz
[root@iZ11b0k6s5lZ src]#
~~~

> **Tip:** 下载后，可以使用 **md5sum** 计算一下包的值，和官网给出的值进行一下比对，可以起到简单校验的效果


---

## 依赖包

下面是依赖的包

~~~
pcre.x86_64   
pcre-devel.x86_64  
zlib.x86_64  
zlib-devel.x86_64 
openssl.x86_64 
openssl-devel.x86_64
~~~

使用 **yum** 进行安装

~~~
[root@iZ11b0k6s5lZ src]# yum install  pcre.x86_64  pcre-devel.x86_64  zlib.x86_64  zlib-devel.x86_64 openssl.x86_64 openssl-devel.x86_64
Loaded plugins: fastestmirror
base                                                                                                      | 3.6 kB  00:00:00     
epel                                                                                                      | 4.3 kB  00:00:00     
extras                                                                                                    | 3.4 kB  00:00:00     
updates                                                                                                   | 3.4 kB  00:00:00     
updates/7/x86_64/primary_db                                                                               | 5.7 MB  00:00:05     
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyuncs.com
 * epel: mirrors.aliyuncs.com
 * extras: mirrors.aliyuncs.com
 * updates: mirrors.aliyuncs.com
Package pcre-8.32-15.el7_2.1.x86_64 already installed and latest version
Package pcre-devel-8.32-15.el7_2.1.x86_64 already installed and latest version
Package zlib-1.2.7-15.el7.x86_64 already installed and latest version
Package zlib-devel-1.2.7-15.el7.x86_64 already installed and latest version
Package 1:openssl-1.0.1e-51.el7_2.5.x86_64 already installed and latest version
Package 1:openssl-devel-1.0.1e-51.el7_2.5.x86_64 already installed and latest version
Nothing to do
[root@iZ11b0k6s5lZ src]#
~~~

> **Note:** 如果缺少上面的包，在编译检查的过程中，可能会报错


---

## 解压

~~~
[root@iZ11b0k6s5lZ src]# ls
tengine-2.1.2.tar.gz
[root@iZ11b0k6s5lZ src]# tar -zxvf tengine-2.1.2.tar.gz 
tengine-2.1.2/
tengine-2.1.2/good_configure
tengine-2.1.2/configure
tengine-2.1.2/docs/
tengine-2.1.2/docs/modules/
tengine-2.1.2/docs/modules/ngx_http_tfs_module_cn.md
...
...
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/check_interface.t
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/tcp_check.t
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/ssl_hello_check.t
tengine-2.1.2/tests/test-nginx/dso_cases/ngx_http_upstream_check_module/http_check.t
[root@iZ11b0k6s5lZ src]# 
~~~

---

## 配置

与配置参数相关的详细信息可以参考 **[Building nginx from Sources][nginx_conf]**  和 **[Tengine 的编译和安装][tengine_conf]**

这里来看看 **configure** 可以接哪些参数

~~~
[root@iZ11b0k6s5lZ src]# ls
tengine-2.1.2  tengine-2.1.2.tar.gz
[root@iZ11b0k6s5lZ src]# cd tengine-2.1.2
[root@iZ11b0k6s5lZ tengine-2.1.2]# ls
AUTHORS.te  CHANGES     CHANGES.ru  conf       contrib  good_configure  LICENSE  modules   README           src    THANKS.te
auto        CHANGES.cn  CHANGES.te  configure  docs     html            man      packages  README.markdown  tests
[root@iZ11b0k6s5lZ tengine-2.1.2]# ./configure  --help 

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

  --builddir=DIR                     set build directory

  --enable-mods-shared=all           enable all the modules to be shared
  --enable-mods-static=all           enable all the modules to be static

  --dso-path=DIR                     set dso default load path
  --dso-tool-path=DIR                set dso_tool pathname
  --dso-max-modules=*)               set max dso module(default is 256)
  --includedir=DIR                   set C header files[PREFIX/include]

  --with-rtsig_module                enable rtsig module
  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --without-procs                    disable procs module

  --with-file-aio                    enable file AIO support
  --with-ipv6                        enable IPv6 support

  --without-syslog                   disable syslog logging

  --without-dso                      disable dso module load

  --with-http_spdy_module            enable ngx_http_spdy_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_filter_module
  --with-http_xslt_module            enable ngx_http_xslt_filter_module
  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_filter_module
  --with-http_dav_module             enable ngx_http_dav_module
  --with-http_flv_module             enable ngx_http_flv_module
  --with-http_slice_module           enable ngx_http_slice_module
  --with-http_mp4_module             enable ngx_http_mp4_module
  --with-http_gunzip_module          enable ngx_http_gunzip_module
  --with-http_gzip_static_module     enable ngx_http_gzip_static_module
  --with-http_auth_request_module    enable ngx_http_auth_request_module
  --with-http_concat_module          enable ngx_http_concat_module
  --with-http_random_index_module    enable ngx_http_random_index_module
  --with-http_secure_link_module     enable ngx_http_secure_link_module
  --with-http_degradation_module     enable ngx_http_degradation_module
  --with-http_sysguard_module        enable ngx_http_sysguard_module

  --with-http_addition_module=shared enable ngx_http_addition_filter_module (shared)
  --with-http_xslt_module=shared     enable ngx_http_xslt_filter_module (shared)
  --with-http_image_filter_module=shared
                                     enable ngx_http_image_filter_module (shared)
  --with-http_geoip_module=shared    enable ngx_http_geoip_module
  --with-http_sub_module=shared      enable ngx_http_sub_filter_module (shared)
  --with-http_flv_module=shared      enable ngx_http_flv_module (shared)
  --with-http_slice_module=shared    enable ngx_http_slice_module (shared)
  --with-http_mp4_module=shared      enable ngx_http_mp4_module (shared)
  --with-http_concat_module=shared   enable ngx_http_concat_module (shared)
  --with-http_random_index_module=shared
                                     enable ngx_http_random_index_module (shared)
  --with-http_secure_link_module=shared
                                     enable ngx_http_secure_link_module (shared)
  --with-http_sysguard_module=shared enable ngx_http_sysguard_module (shared)
  --with-http_charset_filter_module=shared
                                     enable ngx_http_charset_filter_module (shared)
  --with-http_userid_filter_module=shared
                                     enable ngx_http_userid_filter_module (shared)
  --with-http_footer_filter_module=shared
                                     enable ngx_http_footer_filter_module (shared)
  --with-http_trim_filter_module=shared
                                     enable ngx_http_trim_filter_module (shared)
  --with-http_access_module=shared   enable ngx_http_access_module (shared)
  --with-http_autoindex_module=shared
                                     enable ngx_http_autoindex_module (shared)
  --with-http_map_module=shared      enable ngx_http_map_module (shared)
  --with-http_split_clients_module=shared
                                     enable ngx_http_split_clients_module (shared)
  --with-http_referer_module=shared  enable ngx_http_referer_module (shared)
  --with-http_rewrite_module=shared  enable ngx_http_rewrite_module (shared)
  --with-http_fastcgi_module=shared  enable ngx_http_fastcgi_module (shared)
  --with-http_uwsgi_module=shared    enable ngx_http_uwsgi_module (shared)
  --with-http_scgi_module=shared     enable ngx_http_scgi_module (shared)
  --with-http_memcached_module=shared
                                     enable ngx_http_memcached_module (shared)
  --with-http_limit_conn_module=shared
                                     enable ngx_http_limit_conn_module (shared)
  --with-http_limit_req_module=shared
                                     enable ngx_http_limit_req_module (shared)
  --with-http_empty_gif_module=shared
                                     enable ngx_http_empty_gif_module (shared)
  --with-http_browser_module=shared  enable ngx_http_browser_module (shared)
  --with-http_user_agent_module=shared
                                     enable ngx_http_user_agent_module (shared)
  --with-http_upstream_ip_hash_module=shared
                                     enable ngx_http_upstream_ip_hash_module (shared)
  --with-http_upstream_least_conn_module=shared
                                     enable ngx_http_upstream_least_conn_module (shared)
  --with-http_upstream_session_sticky_module=shared
                                     enable ngx_http_upstream_session_sticky_module (shared)
  --with-http_reqstat_module=shared  enable ngx_http_reqstat_module (shared)

  --without-http_charset_module      disable ngx_http_charset_filter_module
  --without-http_gzip_module         disable ngx_http_gzip_filter_module
  --without-http_ssi_module          disable ngx_http_ssi_module
  --without-http_ssl_module          disable ngx_http_ssl_module
  --without-http_userid_module       disable ngx_http_userid_filter_module
  --without-http_footer_filter_module
                                     disable ngx_http_footer_filter_module
  --without-http_trim_filter_module  disable ngx_http_trim_filter_module
  --without-http_access_module       disable ngx_http_access_module
  --without-http_auth_basic_module   disable ngx_http_auth_basic_module
  --without-http_autoindex_module    disable ngx_http_autoindex_module
  --without-http_geo_module          disable ngx_http_geo_module
  --without-http_map_module          disable ngx_http_map_module
  --without-http_split_clients_module
                                     disable ngx_http_split_clients_module
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
  --without-http_upstream_check_module
                                     disable ngx_http_upstream_check_module
  --without-http_upstream_least_conn_module
                                     disable ngx_http_upstream_least_conn_module
  --without-http_upstream_session_sticky_module
                                     disable ngx_http_upstream_session_sticky_module
  --without-http_upstream_keepalive_module
                                     disable ngx_http_upstream_keepalive_module
  --without-http_upstream_dynamic_module
                                     disable ngx_http_upstream_dynamic_module
  --without-http_upstream_ip_hash_module
                                     disable ngx_http_upstream_ip_hash_module
  --without-http_upstream_consistent_hash_module
                                     disable ngx_http_upstream_consistent_hash_module
  --without-http_user_agent_module   disable ngx_http_user_agent_module
  --without-http_stub_status_module  disable ngx_http_stub_status_module
  --without-http_reqstat_module      disable ngx_http_reqstat_module

  --with-http_dyups_module           enable ngx_http_dyups_module
  --with-http_dyups_lua_api          enable lua api of ngx_http_dyups_module

  --with-http_perl_module            enable ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --without-http-upstream-rbtree     disable using rbtree for upstream lookup

  --with-http_lua_module             enable ngx_http_lua_module (will also enable --with-md5 and --with-sha1)
  --with-http_lua_module=shared      enable ngx_http_lua_module (shared) (will also enable --with-md5 and --with-sha1)
  --with-luajit-inc=PATH             set LuaJIT headers path (where lua.h/lauxlib.h/... are located)
  --with-luajit-lib=PATH             set LuaJIT library path (where libluajit-5.1.{a,so} are located)
  --with-lua-inc=PATH                set Lua headers path (where lua.h/lauxlib.h/... are located)
  --with-lua-lib=PATH                set Lua library path (where liblua.{a,so} are located, only support Lua-5.1.x)

  --with-http_tfs_module             enable ngx_http_tfs_module (will also enable --with-md5)
  --with-http_tfs_module=shared      enable ngx_http_tfs_module (shared) (will also enable --with-md5)
  --with-libyajl-inc=PATH            set libyajl headers path (where yajl.h is located)
  --with-libyajl-lib=PATH            set libyajl library path (where libyajl.{a,so} is located)

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

  --with-google_perftools_module     enable ngx_google_perftools_module
  --with-cpp_test_module             enable ngx_cpp_test_module
  --with-backtrace_module            enable ngx_backtrace_module

  --add-module=PATH                  enable an external module

  --with-cc=PATH                     set C compiler pathname
  --with-cpp=PATH                    set C preprocessor pathname
  --with-link=LINK                   set C linker
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

  --with-jemalloc                    force jemalloc library usage
  --with-jemalloc=DIR                set path to jemalloc library files

  --with-openssl=DIR                 set path to OpenSSL library sources
  --with-openssl-opt=OPTIONS         set additional build options for OpenSSL

  --with-debug                       enable debug logging

[root@iZ11b0k6s5lZ tengine-2.1.2]#
~~~

这里不就其它参数细节进行探讨，主要针对模块的加载

---

### 加载模块

加载一个模块的方法就是在配置的时候加上 **`--with-xxx_xxx_module`** ，禁用一个模块的方法就是在后面加上 **`--without-xxx_xxx_module`**

这里我们启用几个常用的模块:

* **[ngx_http_ssl_module][ngx_http_ssl_module]** : 用来支持 HTTPS
* **[ngx_http_gzip_static_module][ngx_http_gzip_static_module]** : 用来支持文件压缩
* **[ngx_http_stub_status_module][ngx_http_stub_status_module]** : 用来提供基本的状态信息
* **[ngx_http_v2_module][ngx_http_v2_module]** : 用来支 HTTP/2
* **ipv6** : 用来支持 IPV6


加入这几个模块进行编译配置

~~~
[root@iZ11b0k6s5lZ tengine-2.1.2]# ./configure --with-http_ssl_module --with-http_gzip_static_module --with-http_stub_status_module --with-ipv6 --with-http_v2_module 
checking for OS
 + Linux 3.10.0-327.el7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) 
checking for gcc -pipe switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for compiler structure-packing pragma ... found
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
checking for O_PATH ... found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for sched_setaffinity() ... found
checking for crypt_r() ... found
checking for SO_REUSEPORT ... found
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
checking for sysinfo() ... found
checking for getloadavg() ... found
checking for /proc/meminfo ... found
checking for sched_yield() ... found
checking for SO_SETFIB ... not found
checking for SO_ACCEPTFILTER ... not found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... found
checking for TCP_INFO ... found
checking for accept4() ... found
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
checking for AF_INET6 ... found
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

[root@iZ11b0k6s5lZ tengine-2.1.2]# echo $?
0
[root@iZ11b0k6s5lZ tengine-2.1.2]# 
~~~


---

## 编译与安装



~~~
[root@iZ11b0k6s5lZ tengine-2.1.2]# make 
make -f objs/Makefile
make[1]: Entering directory `/usr/local/src/tengine-2.1.2'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
	-o objs/src/core/nginx.o \
	src/core/nginx.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/proc -I objs \
	-o objs/src/core/ngx_log.o \
	src/core/ngx_log.c
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
[root@iZ11b0k6s5lZ tengine-2.1.2]# echo $?
0
[root@iZ11b0k6s5lZ tengine-2.1.2]# make install 
make -f objs/Makefile install
make[1]: Entering directory `/usr/local/src/tengine-2.1.2'
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
test -f '/usr/local/nginx/conf/browsers' 		|| cp conf/browsers '/usr/local/nginx/conf'
cp conf/browsers '/usr/local/nginx/conf/browsers'
test -d '/usr/local/nginx/logs' || 		mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/modules/' 		|| mkdir -p '/usr/local/nginx/modules/'
test -f '/usr/local/nginx/conf/module_stubs' 		|| cp objs/module_stubs '/usr/local/nginx/conf'
cp objs/module_stubs '/usr/local/nginx/conf/module_stubs'
test -d '/usr/local/nginx/sbin' || 		mkdir -p '/usr/local/nginx/sbin'
cp objs/dso_tool '/usr/local/nginx/sbin/dso_tool'
chmod 0755 '/usr/local/nginx/sbin/dso_tool'
test -d '/usr/local/nginx/include' 		|| mkdir -p '/usr/local/nginx/include'
test -f 'src/core/nginx.h' && cp 'src/core/nginx.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_config.h' && cp 'src/core/ngx_config.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_core.h' && cp 'src/core/ngx_core.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_log.h' && cp 'src/core/ngx_log.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_palloc.h' && cp 'src/core/ngx_palloc.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_array.h' && cp 'src/core/ngx_array.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_list.h' && cp 'src/core/ngx_list.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_hash.h' && cp 'src/core/ngx_hash.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_buf.h' && cp 'src/core/ngx_buf.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_queue.h' && cp 'src/core/ngx_queue.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_string.h' && cp 'src/core/ngx_string.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_parse.h' && cp 'src/core/ngx_parse.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_inet.h' && cp 'src/core/ngx_inet.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_file.h' && cp 'src/core/ngx_file.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_crc.h' && cp 'src/core/ngx_crc.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_crc32.h' && cp 'src/core/ngx_crc32.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_murmurhash.h' && cp 'src/core/ngx_murmurhash.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_md5.h' && cp 'src/core/ngx_md5.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_sha1.h' && cp 'src/core/ngx_sha1.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_rbtree.h' && cp 'src/core/ngx_rbtree.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_trie.h' && cp 'src/core/ngx_trie.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_segment_tree.h' && cp 'src/core/ngx_segment_tree.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_radix_tree.h' && cp 'src/core/ngx_radix_tree.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_slab.h' && cp 'src/core/ngx_slab.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_times.h' && cp 'src/core/ngx_times.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_shmtx.h' && cp 'src/core/ngx_shmtx.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_connection.h' && cp 'src/core/ngx_connection.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_cycle.h' && cp 'src/core/ngx_cycle.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_conf_file.h' && cp 'src/core/ngx_conf_file.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_resolver.h' && cp 'src/core/ngx_resolver.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_open_file_cache.h' && cp 'src/core/ngx_open_file_cache.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_crypt.h' && cp 'src/core/ngx_crypt.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_proxy_protocol.h' && cp 'src/core/ngx_proxy_protocol.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event.h' && cp 'src/event/ngx_event.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event_timer.h' && cp 'src/event/ngx_event_timer.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event_posted.h' && cp 'src/event/ngx_event_posted.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event_busy_lock.h' && cp 'src/event/ngx_event_busy_lock.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event_connect.h' && cp 'src/event/ngx_event_connect.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event_pipe.h' && cp 'src/event/ngx_event_pipe.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_time.h' && cp 'src/os/unix/ngx_time.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_errno.h' && cp 'src/os/unix/ngx_errno.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_alloc.h' && cp 'src/os/unix/ngx_alloc.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_files.h' && cp 'src/os/unix/ngx_files.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_channel.h' && cp 'src/os/unix/ngx_channel.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_shmem.h' && cp 'src/os/unix/ngx_shmem.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_process.h' && cp 'src/os/unix/ngx_process.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_setaffinity.h' && cp 'src/os/unix/ngx_setaffinity.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_setproctitle.h' && cp 'src/os/unix/ngx_setproctitle.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_atomic.h' && cp 'src/os/unix/ngx_atomic.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_gcc_atomic_x86.h' && cp 'src/os/unix/ngx_gcc_atomic_x86.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_thread.h' && cp 'src/os/unix/ngx_thread.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_socket.h' && cp 'src/os/unix/ngx_socket.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_os.h' && cp 'src/os/unix/ngx_os.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_user.h' && cp 'src/os/unix/ngx_user.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_pipe.h' && cp 'src/os/unix/ngx_pipe.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_sysinfo.h' && cp 'src/os/unix/ngx_sysinfo.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_process_cycle.h' && cp 'src/os/unix/ngx_process_cycle.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_linux_config.h' && cp 'src/os/unix/ngx_linux_config.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_linux.h' && cp 'src/os/unix/ngx_linux.h' '/usr/local/nginx/include'
test -f 'src/os/unix/ngx_syslog.h' && cp 'src/os/unix/ngx_syslog.h' '/usr/local/nginx/include'
test -f 'src/proc/ngx_proc.h' && cp 'src/proc/ngx_proc.h' '/usr/local/nginx/include'
test -f 'src/event/ngx_event_openssl.h' && cp 'src/event/ngx_event_openssl.h' '/usr/local/nginx/include'
test -f 'src/core/ngx_regex.h' && cp 'src/core/ngx_regex.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http.h' && cp 'src/http/ngx_http.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_request.h' && cp 'src/http/ngx_http_request.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_config.h' && cp 'src/http/ngx_http_config.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_core_module.h' && cp 'src/http/ngx_http_core_module.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_cache.h' && cp 'src/http/ngx_http_cache.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_variables.h' && cp 'src/http/ngx_http_variables.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_script.h' && cp 'src/http/ngx_http_script.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_upstream.h' && cp 'src/http/ngx_http_upstream.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_upstream_round_robin.h' && cp 'src/http/ngx_http_upstream_round_robin.h' '/usr/local/nginx/include'
test -f 'src/http/ngx_http_busy_lock.h' && cp 'src/http/ngx_http_busy_lock.h' '/usr/local/nginx/include'
test -f 'src/http/modules/ngx_http_ssi_filter_module.h' && cp 'src/http/modules/ngx_http_ssi_filter_module.h' '/usr/local/nginx/include'
test -f 'src/http/v2/ngx_http_v2.h' && cp 'src/http/v2/ngx_http_v2.h' '/usr/local/nginx/include'
test -f 'src/http/v2/ngx_http_v2_module.h' && cp 'src/http/v2/ngx_http_v2_module.h' '/usr/local/nginx/include'
test -f 'src/http/modules/ngx_http_ssl_module.h' && cp 'src/http/modules/ngx_http_ssl_module.h' '/usr/local/nginx/include'
test -f 'src/http/modules/ngx_http_reqstat.h' && cp 'src/http/modules/ngx_http_reqstat.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_headers.h'  && cp 'objs/ngx_auto_headers.h' '/usr/local/nginx/include'
test -f 'objs/ngx_auto_config.h' && cp 'objs/ngx_auto_config.h' '/usr/local/nginx/include'
make[1]: Leaving directory `/usr/local/src/tengine-2.1.2'
[root@iZ11b0k6s5lZ tengine-2.1.2]# 
[root@iZ11b0k6s5lZ tengine-2.1.2]# echo $?
0
[root@iZ11b0k6s5lZ tengine-2.1.2]# 
~~~

---

## 查看模块

~~~
[root@iZ11b0k6s5lZ tengine-2.1.2]# /usr/local/nginx/sbin/nginx -V
Tengine version: Tengine/2.1.2 (nginx/1.6.2)
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) 
TLS SNI support enabled
configure arguments: --with-http_ssl_module --with-http_gzip_static_module --with-http_stub_status_module --with-ipv6 --with-http_v2_module
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
    ngx_http_v2_module (static)
    ngx_http_static_module (static)
    ngx_http_gzip_static_module (static)
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
    ngx_http_upstream_keepalive_module (static)
    ngx_http_upstream_dynamic_module (static)
    ngx_http_stub_status_module (static)
    ngx_http_write_filter_module (static)
    ngx_http_header_filter_module (static)
    ngx_http_chunked_filter_module (static)
    ngx_http_v2_filter_module (static)
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
    ngx_http_reqstat_module (static)
    ngx_http_copy_filter_module (static)
    ngx_http_range_body_filter_module (static)
    ngx_http_not_modified_filter_module (static)
[root@iZ11b0k6s5lZ tengine-2.1.2]#
~~~

此时这四种模块就被添加到 tengine 中了

---

# 命令汇总

* **`hostnamectl`**
* **`wget http://tengine.taobao.org/download/tengine-2.1.2.tar.gz`**
* **`md5sum tengine-2.1.2.tar.gz`**
* **`yum install  pcre.x86_64  pcre-devel.x86_64  zlib.x86_64  zlib-devel.x86_64 openssl.x86_64 openssl-devel.x86_64`**
* **`tar -zxvf tengine-2.1.2.tar.gz`**
* **`cd tengine-2.1.2`**
* **`./configure  --help`**
* **`./configure --with-http_ssl_module --with-http_gzip_static_module --with-http_stub_status_module --with-ipv6 --with-http_v2_module`**
* **`make`**
* **`make install`**
* **`echo $?`**
* **`/usr/local/nginx/sbin/nginx -V`**



---


[nginx]:http://nginx.org/
[nginx_blog]:http://soft.dog/2015/10/09/nginx-basic/
[tengine]:http://tengine.taobao.org/index_cn.html
[tengine_blog]:http://soft.dog/2015/11/04/Tengine-basic/
[tengine_doc]:http://tengine.taobao.org/documentation_cn.html
[nginx_doc]:http://nginx.org/en/docs/
[tengine_dl]:http://tengine.taobao.org/download_cn.html
[nginx_conf]:http://nginx.org/en/docs/configure.html
[tengine_conf]:http://tengine.taobao.org/document_cn/install_cn.html
[ngx_http_ssl_module]:http://nginx.org/en/docs/http/ngx_http_ssl_module.html
[ngx_http_gzip_static_module]:http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html
[ngx_http_stub_status_module]:http://nginx.org/en/docs/http/ngx_http_stub_status_module.html
[ngx_http_v2_module]:http://nginx.org/en/docs/http/ngx_http_v2_module.html

