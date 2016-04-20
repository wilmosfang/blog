---
layout: post
title: memcached基础
author: wilmosfang
categories: linux nosql memcached
wc: 1133 4849 46722
excerpt: memcached 的安装、基础管理命令、基本操作命令、状态查看
comments: true
---


# 前言

**[memcached][memcached]**  是一个自由开源的，高性能分布式内存对象缓存系统

>Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls


(更多特性参考[memcached][memcached])

更为详细的文档可以参考 **[memcached][memcached]** 

这里分享一下 **[memcached]** 的相关基础

> **Tip:** 当前版本 **memcached -v1.4.24**

---

# 概要

* TOC
{:toc}


---

## 下载源码包



{% highlight bash %}
[root@h101 src]# wget  http://www.memcached.org/files/memcached-1.4.24.tar.gz
--2015-09-23 14:21:12--  http://www.memcached.org/files/memcached-1.4.24.tar.gz
Resolving www.memcached.org... 173.255.253.96
Connecting to www.memcached.org|173.255.253.96|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 354917 (347K) [application/x-tar]
Saving to: “memcached-1.4.24.tar.gz”

60% [=====================================================>                                     ] 213,988     34.2K/s  eta 16s     
[root@h101 src]# 
{% endhighlight %}

---

## 解压

{% highlight bash %}
[root@h101 src]# ls
memcached-1.4.24.tar.gz
[root@h101 src]# tar -zxvf memcached-1.4.24.tar.gz 
memcached-1.4.24/
memcached-1.4.24/sasl_defs.c
memcached-1.4.24/solaris_priv.c
memcached-1.4.24/AUTHORS
memcached-1.4.24/config.guess
...
...
memcached-1.4.24/Makefile.am
memcached-1.4.24/COPYING
memcached-1.4.24/cache.c
memcached-1.4.24/Makefile.in
[root@h101 src]# ls
memcached-1.4.24  memcached-1.4.24.tar.gz
[root@h101 src]# 
{% endhighlight %}

---

## 安装

此时直接配置会出错

{% highlight bash %}
[root@h101 src]# cd memcached-1.4.24
[root@h101 memcached-1.4.24]# ls
aclocal.m4  compile       daemon.c    items.h         memcached_dtrace.d  protocol_binary.h  slabs.h         timedrun.c
assoc.c     config.guess  depcomp     jenkins_hash.c  memcached.h         README.md          solaris_priv.c  trace.h
assoc.h     config.h.in   doc         jenkins_hash.h  memcached.spec      sasl_defs.c        stats.c         util.c
AUTHORS     config.sub    hash.c      m4              missing             sasl_defs.h        stats.h         util.h
cache.c     configure     hash.h      Makefile.am     murmur3_hash.c      scripts            t               version.m4
cache.h     configure.ac  install-sh  Makefile.in     murmur3_hash.h      sizes.c            testapp.c
ChangeLog   COPYING       items.c     memcached.c     NEWS                slabs.c            thread.c
[root@h101 memcached-1.4.24]# ./configure 
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking for style of include used by make... GNU
checking dependency style of gcc... gcc3
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for icc in use... no
checking for clang in use... no
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking whether __SUNPRO_C is declared... no
checking for gcc option to accept ISO C99... -std=gnu99
checking sasl/sasl.h usability... no
checking sasl/sasl.h presence... no
checking for sasl/sasl.h... no
checking for gcov... /usr/bin/gcov
checking for main in -lgcov... yes
checking for library containing clock_gettime... -lrt
checking for library containing socket... none required
checking for library containing gethostbyname... none required
checking for libevent directory... configure: error: libevent is required.  You can get it from http://www.monkey.org/~provos/libevent/

      If it's already installed, specify its path using --with-libevent=/dir/

[root@h101 memcached-1.4.24]#
{% endhighlight %}

提示缺少 **libevent** 

{% highlight bash %}
[root@h101 memcached-1.4.24]# rpm -qa | grep libevent
libevent-1.4.13-4.el6.x86_64
[root@h101 memcached-1.4.24]# rpm -ql  libevent-1.4.13-4.el6.x86_64 
/usr/lib64/libevent-1.4.so.2
/usr/lib64/libevent-1.4.so.2.1.3
/usr/lib64/libevent_core-1.4.so.2
/usr/lib64/libevent_core-1.4.so.2.1.3
/usr/lib64/libevent_extra-1.4.so.2
/usr/lib64/libevent_extra-1.4.so.2.1.3
/usr/share/doc/libevent-1.4.13
/usr/share/doc/libevent-1.4.13/README
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

发现已经 **libevent** 安装了

使用 **--with-libevent=/usr/lib64/** 再试一次

{% highlight bash %}
[root@h101 memcached-1.4.24]# ./configure --with-libevent=/usr/lib64/
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking for style of include used by make... GNU
checking dependency style of gcc... gcc3
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for icc in use... no
checking for clang in use... no
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking whether __SUNPRO_C is declared... no
checking for gcc option to accept ISO C99... -std=gnu99
checking sasl/sasl.h usability... no
checking sasl/sasl.h presence... no
checking for sasl/sasl.h... no
checking for gcov... /usr/bin/gcov
checking for main in -lgcov... yes
checking for library containing clock_gettime... -lrt
checking for library containing socket... none required
checking for library containing gethostbyname... none required
checking for libevent directory... configure: error: libevent is required.  You can get it from http://www.monkey.org/~provos/libevent/

      If it's already installed, specify its path using --with-libevent=/dir/

[root@h101 memcached-1.4.24]#
{% endhighlight %}

问题依旧，原来是缺少 **libevent-devel** 开发包

---

### 解决依赖

{% highlight bash %}
[root@h101 memcached-1.4.24]# yum -y install libevent-devel.x86_64  
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Determining fastest mirrors
epel/metalink                                                                                                  | 4.5 kB     00:00     
 * base: mirrors.pubyun.com
 * epel: mirrors.opencas.cn
 * extras: mirrors.pubyun.com
 * updates: mirrors.pubyun.com
base                                                                                                           | 3.7 kB     00:00     
base/primary_db                                                                                                | 4.6 MB     00:00     
base123                                                                                                        | 4.0 kB     00:00 ... 
base123/primary_db                                                                                             | 4.5 MB     00:00 ... 
epel                                                                                                           | 4.3 kB     00:00     
epel/primary_db                                                                                                | 5.7 MB     00:21     
extras                                                                                                         | 3.4 kB     00:00     
extras/primary_db                                                                                              |  32 kB     00:00     
updates                                                                                                        | 3.4 kB     00:00     
updates/primary_db                                                                                             | 1.9 MB     00:00     
Resolving Dependencies
--> Running transaction check
---> Package libevent-devel.x86_64 0:1.4.13-4.el6 will be installed
--> Processing Dependency: libevent-headers = 1.4.13-4.el6 for package: libevent-devel-1.4.13-4.el6.x86_64
--> Processing Dependency: libevent-doc = 1.4.13-4.el6 for package: libevent-devel-1.4.13-4.el6.x86_64
--> Running transaction check
---> Package libevent-doc.noarch 0:1.4.13-4.el6 will be installed
---> Package libevent-headers.noarch 0:1.4.13-4.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                               Arch                        Version                            Repository                 Size
======================================================================================================================================
Installing:
 libevent-devel                        x86_64                      1.4.13-4.el6                       base                       74 k
Installing for dependencies:
 libevent-doc                          noarch                      1.4.13-4.el6                       base                      194 k
 libevent-headers                      noarch                      1.4.13-4.el6                       base                       30 k

Transaction Summary
======================================================================================================================================
Install       3 Package(s)

Total download size: 297 k
Installed size: 1.4 M
Downloading Packages:
(1/3): libevent-devel-1.4.13-4.el6.x86_64.rpm                                                                  |  74 kB     00:00     
(2/3): libevent-doc-1.4.13-4.el6.noarch.rpm                                                                    | 194 kB     00:00     
(3/3): libevent-headers-1.4.13-4.el6.noarch.rpm                                                                |  30 kB     00:00     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 1.7 MB/s | 297 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : libevent-doc-1.4.13-4.el6.noarch                                                                                   1/3 
  Installing : libevent-devel-1.4.13-4.el6.x86_64                                                                                 2/3 
  Installing : libevent-headers-1.4.13-4.el6.noarch                                                                               3/3 
  Verifying  : libevent-headers-1.4.13-4.el6.noarch                                                                               1/3 
  Verifying  : libevent-devel-1.4.13-4.el6.x86_64                                                                                 2/3 
  Verifying  : libevent-doc-1.4.13-4.el6.noarch                                                                                   3/3 

Installed:
  libevent-devel.x86_64 0:1.4.13-4.el6                                                                                                

Dependency Installed:
  libevent-doc.noarch 0:1.4.13-4.el6                              libevent-headers.noarch 0:1.4.13-4.el6                             

Complete!
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

### 安装

{% highlight bash %}
[root@h101 memcached-1.4.24]# ./configure 
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking for style of include used by make... GNU
checking dependency style of gcc... gcc3
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for icc in use... no
checking for clang in use... no
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking whether __SUNPRO_C is declared... no
checking for gcc option to accept ISO C99... -std=gnu99
checking sasl/sasl.h usability... no
checking sasl/sasl.h presence... no
checking for sasl/sasl.h... no
checking for gcov... /usr/bin/gcov
checking for main in -lgcov... yes
checking for library containing clock_gettime... -lrt
checking for library containing socket... none required
checking for library containing gethostbyname... none required
checking for libevent directory... (system)
checking for library containing umem_cache_create... no
checking for library containing gethugepagesizes... no
checking for stdbool.h that conforms to C99... yes
checking for _Bool... yes
checking for inttypes.h... (cached) yes
checking for sasl_callback_ft... no
checking for print macros for integers (C99 section 7.8.1)... yes
checking for an ANSI C-conforming const... yes
checking for socklen_t... yes
checking for endianness... little
checking for htonll... no
checking for library containing pthread_create... none required
checking for mlockall... yes
checking for getpagesizes... no
checking for memcntl... no
checking for sigignore... yes
checking for clock_gettime... yes
checking for accept4... yes
checking for alignment... none
checking for GCC atomics... yes
checking for setppriv... no
checking umem.h usability... no
checking umem.h presence... no
checking for umem.h... no
checking for xml2rfc... no
checking for xsltproc... /usr/bin/xsltproc
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating doc/Makefile
config.status: creating config.h
config.status: executing depfiles commands
[root@h101 memcached-1.4.24]# echo $?
0
[root@h101 memcached-1.4.24]# make 
make  all-recursive
make[1]: Entering directory `/usr/local/src/memcached-1.4.24'
Making all in doc
make[2]: Entering directory `/usr/local/src/memcached-1.4.24/doc'
make  all-am
make[3]: Entering directory `/usr/local/src/memcached-1.4.24/doc'
make[3]: Nothing to be done for `all-am'.
make[3]: Leaving directory `/usr/local/src/memcached-1.4.24/doc'
make[2]: Leaving directory `/usr/local/src/memcached-1.4.24/doc'
make[2]: Entering directory `/usr/local/src/memcached-1.4.24'
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-memcached.o -MD -MP -MF .deps/memcached-memcached.Tpo -c -o memcached-memcached.o `test -f 'memcached.c' || echo './'`memcached.c
mv -f .deps/memcached-memcached.Tpo .deps/memcached-memcached.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-hash.o -MD -MP -MF .deps/memcached-hash.Tpo -c -o memcached-hash.o `test -f 'hash.c' || echo './'`hash.c
mv -f .deps/memcached-hash.Tpo .deps/memcached-hash.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-jenkins_hash.o -MD -MP -MF .deps/memcached-jenkins_hash.Tpo -c -o memcached-jenkins_hash.o `test -f 'jenkins_hash.c' || echo './'`jenkins_hash.c
mv -f .deps/memcached-jenkins_hash.Tpo .deps/memcached-jenkins_hash.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-murmur3_hash.o -MD -MP -MF .deps/memcached-murmur3_hash.Tpo -c -o memcached-murmur3_hash.o `test -f 'murmur3_hash.c' || echo './'`murmur3_hash.c
mv -f .deps/memcached-murmur3_hash.Tpo .deps/memcached-murmur3_hash.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-slabs.o -MD -MP -MF .deps/memcached-slabs.Tpo -c -o memcached-slabs.o `test -f 'slabs.c' || echo './'`slabs.c
mv -f .deps/memcached-slabs.Tpo .deps/memcached-slabs.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-items.o -MD -MP -MF .deps/memcached-items.Tpo -c -o memcached-items.o `test -f 'items.c' || echo './'`items.c
mv -f .deps/memcached-items.Tpo .deps/memcached-items.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-assoc.o -MD -MP -MF .deps/memcached-assoc.Tpo -c -o memcached-assoc.o `test -f 'assoc.c' || echo './'`assoc.c
mv -f .deps/memcached-assoc.Tpo .deps/memcached-assoc.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-thread.o -MD -MP -MF .deps/memcached-thread.Tpo -c -o memcached-thread.o `test -f 'thread.c' || echo './'`thread.c
mv -f .deps/memcached-thread.Tpo .deps/memcached-thread.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-daemon.o -MD -MP -MF .deps/memcached-daemon.Tpo -c -o memcached-daemon.o `test -f 'daemon.c' || echo './'`daemon.c
mv -f .deps/memcached-daemon.Tpo .deps/memcached-daemon.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-stats.o -MD -MP -MF .deps/memcached-stats.Tpo -c -o memcached-stats.o `test -f 'stats.c' || echo './'`stats.c
mv -f .deps/memcached-stats.Tpo .deps/memcached-stats.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-util.o -MD -MP -MF .deps/memcached-util.Tpo -c -o memcached-util.o `test -f 'util.c' || echo './'`util.c
mv -f .deps/memcached-util.Tpo .deps/memcached-util.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.  -DNDEBUG   -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached-cache.o -MD -MP -MF .deps/memcached-cache.Tpo -c -o memcached-cache.o `test -f 'cache.c' || echo './'`cache.c
mv -f .deps/memcached-cache.Tpo .deps/memcached-cache.Po
gcc -std=gnu99  -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing   -o memcached memcached-memcached.o memcached-hash.o memcached-jenkins_hash.o memcached-murmur3_hash.o memcached-slabs.o memcached-items.o memcached-assoc.o memcached-thread.o memcached-daemon.o memcached-stats.o memcached-util.o memcached-cache.o    -levent -lrt 
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-memcached.o -MD -MP -MF .deps/memcached_debug-memcached.Tpo -c -o memcached_debug-memcached.o `test -f 'memcached.c' || echo './'`memcached.c
mv -f .deps/memcached_debug-memcached.Tpo .deps/memcached_debug-memcached.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-hash.o -MD -MP -MF .deps/memcached_debug-hash.Tpo -c -o memcached_debug-hash.o `test -f 'hash.c' || echo './'`hash.c
mv -f .deps/memcached_debug-hash.Tpo .deps/memcached_debug-hash.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-jenkins_hash.o -MD -MP -MF .deps/memcached_debug-jenkins_hash.Tpo -c -o memcached_debug-jenkins_hash.o `test -f 'jenkins_hash.c' || echo './'`jenkins_hash.c
mv -f .deps/memcached_debug-jenkins_hash.Tpo .deps/memcached_debug-jenkins_hash.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-murmur3_hash.o -MD -MP -MF .deps/memcached_debug-murmur3_hash.Tpo -c -o memcached_debug-murmur3_hash.o `test -f 'murmur3_hash.c' || echo './'`murmur3_hash.c
mv -f .deps/memcached_debug-murmur3_hash.Tpo .deps/memcached_debug-murmur3_hash.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-slabs.o -MD -MP -MF .deps/memcached_debug-slabs.Tpo -c -o memcached_debug-slabs.o `test -f 'slabs.c' || echo './'`slabs.c
mv -f .deps/memcached_debug-slabs.Tpo .deps/memcached_debug-slabs.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-items.o -MD -MP -MF .deps/memcached_debug-items.Tpo -c -o memcached_debug-items.o `test -f 'items.c' || echo './'`items.c
mv -f .deps/memcached_debug-items.Tpo .deps/memcached_debug-items.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-assoc.o -MD -MP -MF .deps/memcached_debug-assoc.Tpo -c -o memcached_debug-assoc.o `test -f 'assoc.c' || echo './'`assoc.c
mv -f .deps/memcached_debug-assoc.Tpo .deps/memcached_debug-assoc.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-thread.o -MD -MP -MF .deps/memcached_debug-thread.Tpo -c -o memcached_debug-thread.o `test -f 'thread.c' || echo './'`thread.c
mv -f .deps/memcached_debug-thread.Tpo .deps/memcached_debug-thread.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-daemon.o -MD -MP -MF .deps/memcached_debug-daemon.Tpo -c -o memcached_debug-daemon.o `test -f 'daemon.c' || echo './'`daemon.c
mv -f .deps/memcached_debug-daemon.Tpo .deps/memcached_debug-daemon.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-stats.o -MD -MP -MF .deps/memcached_debug-stats.Tpo -c -o memcached_debug-stats.o `test -f 'stats.c' || echo './'`stats.c
mv -f .deps/memcached_debug-stats.Tpo .deps/memcached_debug-stats.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-util.o -MD -MP -MF .deps/memcached_debug-util.Tpo -c -o memcached_debug-util.o `test -f 'util.c' || echo './'`util.c
mv -f .deps/memcached_debug-util.Tpo .deps/memcached_debug-util.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.    -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT memcached_debug-cache.o -MD -MP -MF .deps/memcached_debug-cache.Tpo -c -o memcached_debug-cache.o `test -f 'cache.c' || echo './'`cache.c
mv -f .deps/memcached_debug-cache.Tpo .deps/memcached_debug-cache.Po
gcc -std=gnu99 -fprofile-arcs -ftest-coverage -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing   -o memcached-debug memcached_debug-memcached.o memcached_debug-hash.o memcached_debug-jenkins_hash.o memcached_debug-murmur3_hash.o memcached_debug-slabs.o memcached_debug-items.o memcached_debug-assoc.o memcached_debug-thread.o memcached_debug-daemon.o memcached_debug-stats.o memcached_debug-util.o memcached_debug-cache.o   -lgcov  -levent -lrt 
gcc -std=gnu99 -DHAVE_CONFIG_H -I.     -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT sizes.o -MD -MP -MF .deps/sizes.Tpo -c -o sizes.o sizes.c
mv -f .deps/sizes.Tpo .deps/sizes.Po
gcc -std=gnu99  -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing   -o sizes sizes.o  -levent -lrt 
gcc -std=gnu99 -DHAVE_CONFIG_H -I.     -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT testapp.o -MD -MP -MF .deps/testapp.Tpo -c -o testapp.o testapp.c
mv -f .deps/testapp.Tpo .deps/testapp.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.     -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT util.o -MD -MP -MF .deps/util.Tpo -c -o util.o util.c
mv -f .deps/util.Tpo .deps/util.Po
gcc -std=gnu99 -DHAVE_CONFIG_H -I.     -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT cache.o -MD -MP -MF .deps/cache.Tpo -c -o cache.o cache.c
mv -f .deps/cache.Tpo .deps/cache.Po
gcc -std=gnu99  -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing   -o testapp testapp.o util.o cache.o  -levent -lrt 
gcc -std=gnu99 -DHAVE_CONFIG_H -I.     -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing -MT timedrun.o -MD -MP -MF .deps/timedrun.Tpo -c -o timedrun.o timedrun.c
mv -f .deps/timedrun.Tpo .deps/timedrun.Po
gcc -std=gnu99  -g -O2 -pthread -pthread -Wall -Werror -pedantic -Wmissing-prototypes -Wmissing-declarations -Wredundant-decls -fno-strict-aliasing   -o timedrun timedrun.o  -levent -lrt 
make[2]: Leaving directory `/usr/local/src/memcached-1.4.24'
make[1]: Leaving directory `/usr/local/src/memcached-1.4.24'
[root@h101 memcached-1.4.24]# echo $?
0
[root@h101 memcached-1.4.24]# make install 
make  install-recursive
make[1]: Entering directory `/usr/local/src/memcached-1.4.24'
Making install in doc
make[2]: Entering directory `/usr/local/src/memcached-1.4.24/doc'
make  install-am
make[3]: Entering directory `/usr/local/src/memcached-1.4.24/doc'
make[4]: Entering directory `/usr/local/src/memcached-1.4.24/doc'
make[4]: Nothing to be done for `install-exec-am'.
 /bin/mkdir -p '/usr/local/share/man/man1'
 /usr/bin/install -c -m 644 memcached.1 '/usr/local/share/man/man1'
make[4]: Leaving directory `/usr/local/src/memcached-1.4.24/doc'
make[3]: Leaving directory `/usr/local/src/memcached-1.4.24/doc'
make[2]: Leaving directory `/usr/local/src/memcached-1.4.24/doc'
make[2]: Entering directory `/usr/local/src/memcached-1.4.24'
make[3]: Entering directory `/usr/local/src/memcached-1.4.24'
 /bin/mkdir -p '/usr/local/bin'
  /usr/bin/install -c memcached '/usr/local/bin'
 /bin/mkdir -p '/usr/local/include/memcached'
 /usr/bin/install -c -m 644 protocol_binary.h '/usr/local/include/memcached'
make[3]: Leaving directory `/usr/local/src/memcached-1.4.24'
make[2]: Leaving directory `/usr/local/src/memcached-1.4.24'
make[1]: Leaving directory `/usr/local/src/memcached-1.4.24'
[root@h101 memcached-1.4.24]# echo $?
0
[root@h101 memcached-1.4.24]# ll /usr/local/bin/ | grep memcached 
-rwxr-xr-x 1 root root 358710 Sep 23 14:45 memcached
[root@h101 memcached-1.4.24]#
{% endhighlight %}

---

## 启动


以下为 **memcached** 的参数

{% highlight bash %}
[root@h101 memcached-1.4.24]# /usr/local/bin/memcached  -h 
memcached 1.4.24
-p <num>      TCP port number to listen on (default: 11211)
-U <num>      UDP port number to listen on (default: 11211, 0 is off)
-s <file>     UNIX socket path to listen on (disables network support)
-A            enable ascii "shutdown" command
-a <mask>     access mask for UNIX socket, in octal (default: 0700)
-l <addr>     interface to listen on (default: INADDR_ANY, all addresses)
              <addr> may be specified as host:port. If you don't specify
              a port number, the value you specified with -p or -U is
              used. You may specify multiple addresses separated by comma
              or by using -l multiple times
-d            run as a daemon
-r            maximize core file limit
-u <username> assume identity of <username> (only when run as root)
-m <num>      max memory to use for items in megabytes (default: 64 MB)
-M            return error on memory exhausted (rather than removing items)
-c <num>      max simultaneous connections (default: 1024)
-k            lock down all paged memory.  Note that there is a
              limit on how much memory you may lock.  Trying to
              allocate more than that would fail, so be sure you
              set the limit correctly for the user you started
              the daemon with (not for -u <username> user;
              under sh this is done with 'ulimit -S -l NUM_KB').
-v            verbose (print errors/warnings while in event loop)
-vv           very verbose (also print client commands/reponses)
-vvv          extremely verbose (also print internal state transitions)
-h            print this help and exit
-i            print memcached and libevent license
-V            print version and exit
-P <file>     save PID in <file>, only used with -d option
-f <factor>   chunk size growth factor (default: 1.25)
-n <bytes>    minimum space allocated for key+value+flags (default: 48)
-L            Try to use large memory pages (if available). Increasing
              the memory page size could reduce the number of TLB misses
              and improve the performance. In order to get large pages
              from the OS, memcached will allocate the total item-cache
              in one large chunk.
-D <char>     Use <char> as the delimiter between key prefixes and IDs.
              This is used for per-prefix stats reporting. The default is
              ":" (colon). If this option is specified, stats collection
              is turned on automatically; if not, then it may be turned on
              by sending the "stats detail on" command to the server.
-t <num>      number of threads to use (default: 4)
-R            Maximum number of requests per event, limits the number of
              requests process for a given connection to prevent 
              starvation (default: 20)
-C            Disable use of CAS
-b            Set the backlog queue limit (default: 1024)
-B            Binding protocol - one of ascii, binary, or auto (default)
-I            Override the size of each slab page. Adjusts max item size
              (default: 1mb, min: 1k, max: 128m)
-F            Disable flush_all command
-o            Comma separated list of extended or experimental options
              - (EXPERIMENTAL) maxconns_fast: immediately close new
                connections if over maxconns limit
              - hashpower: An integer multiplier for how large the hash
                table should be. Can be grown at runtime if not big enough.
                Set this based on "STAT hash_power_level" before a 
                restart.
              - tail_repair_time: Time in seconds that indicates how long to wait before
                forcefully taking over the LRU tail item whose refcount has leaked.
                Disabled by default; dangerous option.
              - hash_algorithm: The hash table algorithm
                default is jenkins hash. options: jenkins, murmur3
              - lru_crawler: Enable LRU Crawler background thread
              - lru_crawler_sleep: Microseconds to sleep between items
                default is 100.
              - lru_crawler_tocrawl: Max items to crawl per slab per run
                default is 0 (unlimited)
              - lru_maintainer: Enable new LRU system + background thread
              - hot_lru_pct: Pct of slab memory to reserve for hot lru.
                (requires lru_maintainer)
              - warm_lru_pct: Pct of slab memory to reserve for warm lru.
                (requires lru_maintainer)
              - expirezero_does_not_evict: Items set to not expire, will not evict.
                (requires lru_maintainer)
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

启动一个 **memcached** 后台进程

{% highlight bash %}
[root@h101 memcached-1.4.24]# /usr/local/bin/memcached  -d  -m 1024 -p 12345 -u cc -c 512 -t 10 
[root@h101 memcached-1.4.24]# ps faux | grep memcached
root      8745  0.0  0.0 103252   828 pts/0    S+   15:17   0:00  |       \_ grep memcached
cc        8732  0.1  0.0 786084  1644 ?        Ssl  15:17   0:00 /usr/local/bin/memcached -d -m 1024 -p 12345 -u cc -c 512 -t 10
[root@h101 memcached-1.4.24]# 
[root@h101 memcached-1.4.24]# netstat  -ant | grep 12345
tcp        0      0 0.0.0.0:12345               0.0.0.0:*                   LISTEN      
tcp        0      0 :::12345                    :::*                        LISTEN      
[root@h101 memcached-1.4.24]# 
{% endhighlight %}


Option     | Comment
------- | ---
-d  | 以服务的形式后台运行
-m 1024 | 最大内存分配额度1024M
-p 12345| TCP监听端口(默认为11211)
-u cc |以cc用户身份运行此服务
-c 512 |最大并行连接数
-t 10 |线程数(默认为4)



## 停止

停止是比较简单粗暴的，直接使用kill

{% highlight bash %}
[root@h101 memcached-1.4.24]# ps faux | grep mem
root       777  0.0  0.0      0     0 ?        S    13:43   0:00  \_ [vmmemctl]
root      8822  0.0  0.0 103252   828 pts/0    S+   15:48   0:00  |       \_ grep mem
cc        8732  0.0  0.0 786084  1644 ?        Ssl  15:17   0:00 /usr/local/bin/memcached -d -m 1024 -p 12345 -u cc -c 512 -t 10
[root@h101 memcached-1.4.24]# kill  8732
[root@h101 memcached-1.4.24]# ps faux | grep mem
root       777  0.0  0.0      0     0 ?        S    13:43   0:00  \_ [vmmemctl]
root      8825  0.0  0.0 103252   828 pts/0    S+   15:49   0:00  |       \_ grep mem
[root@h101 memcached-1.4.24]# netstat  -ant | grep 12345
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

---

## 进程状态检查


使用下列方法可以检查线程数量

{% highlight bash %}
[root@h101 memcached-1.4.24]# ps fuax | grep mem
root       777  0.0  0.0      0     0 ?        S    13:43   0:00  \_ [vmmemctl]
root      8857  0.0  0.0 103252   824 pts/0    S+   15:52   0:00  |       \_ grep mem
cc        8835  0.0  0.0 786084  1640 ?        Ssl  15:50   0:00 /usr/local/bin/memcached -d -m 1024 -p 12345 -u cc -c 512 -t 10
[root@h101 memcached-1.4.24]# netstat  -ant | grep 12345
tcp        0      0 0.0.0.0:12345               0.0.0.0:*                   LISTEN      
tcp        0      0 :::12345                    :::*                        LISTEN      
[root@h101 memcached-1.4.24]# pstree -p  8835 
memcached(8835)─┬─{memcached}(8836)
                ├─{memcached}(8837)
                ├─{memcached}(8838)
                ├─{memcached}(8839)
                ├─{memcached}(8840)
                ├─{memcached}(8841)
                ├─{memcached}(8842)
                ├─{memcached}(8843)
                ├─{memcached}(8844)
                ├─{memcached}(8845)
                └─{memcached}(8846)
[root@h101 memcached-1.4.24]# ps -mp 8835
  PID TTY          TIME CMD
 8835 ?        00:00:00 memcached
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
    - -        00:00:00 -
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

修改部分参数，重启，发现如期进行了调整

{% highlight bash %}
[root@h101 memcached-1.4.24]# kill  8835 
[root@h101 memcached-1.4.24]# netstat  -ant | grep 12345
[root@h101 memcached-1.4.24]# /usr/local/bin/memcached -d -m 1024 -p 12354 -u test  -c 512 -t 5
[root@h101 memcached-1.4.24]# ps faux | grep mem 
root       777  0.0  0.0      0     0 ?        S    13:43   0:00  \_ [vmmemctl]
root      8925  0.0  0.0 103252   828 pts/0    S+   16:04   0:00  |       \_ grep mem
test      8917  0.4  0.0 407052  1360 ?        Ssl  16:04   0:00 /usr/local/bin/memcached -d -m 1024 -p 12354 -u test -c 512 -t 5
[root@h101 memcached-1.4.24]# netstat  -ant | grep 12354 
tcp        0      0 0.0.0.0:12354               0.0.0.0:*                   LISTEN      
tcp        0      0 :::12354                    :::*                        LISTEN      
[root@h101 memcached-1.4.24]# pstree -p 8917
memcached(8917)─┬─{memcached}(8918)
                ├─{memcached}(8919)
                ├─{memcached}(8920)
                ├─{memcached}(8921)
                ├─{memcached}(8922)
                └─{memcached}(8923)
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

---

## 常用命令


### telnet

使用 **telnet** 连接实例

{% highlight bash %}
[root@h101 memcached-1.4.24]# telnet localhost 12354
Trying ::1...
Connected to localhost.
Escape character is '^]'.

ERROR

ERROR
{% endhighlight %}

---

### stats

查看状态

{% highlight bash %}
stats
STAT pid 8917
STAT uptime 1183
STAT time 1442996635
STAT version 1.4.24
STAT libevent 1.4.13-stable
STAT pointer_size 64
STAT rusage_user 0.014997
STAT rusage_system 0.105983
STAT curr_connections 12
STAT total_connections 14
STAT connection_structures 13
STAT reserved_fds 25
STAT cmd_get 0
STAT cmd_set 0
STAT cmd_flush 0
STAT cmd_touch 0
STAT get_hits 0
STAT get_misses 0
STAT delete_misses 0
STAT delete_hits 0
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT touch_hits 0
STAT touch_misses 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 24
STAT bytes_written 21
STAT limit_maxbytes 1073741824
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT threads 5
STAT conn_yields 0
STAT hash_power_level 16
STAT hash_bytes 524288
STAT hash_is_expanding 0
STAT malloc_fails 0
STAT bytes 0
STAT curr_items 0
STAT total_items 0
STAT expired_unfetched 0
STAT evicted_unfetched 0
STAT evictions 0
STAT reclaimed 0
STAT crawler_reclaimed 0
STAT crawler_items_checked 0
STAT lrutail_reflocked 0
END
{% endhighlight %}

---

#### stats items

{% highlight bash %}
stats items
STAT items:1:number 4
STAT items:1:age 1736
STAT items:1:evicted 0
STAT items:1:evicted_nonzero 0
STAT items:1:evicted_time 0
STAT items:1:outofmemory 0
STAT items:1:tailrepairs 0
STAT items:1:reclaimed 0
STAT items:1:expired_unfetched 0
STAT items:1:evicted_unfetched 0
STAT items:1:crawler_reclaimed 0
STAT items:1:crawler_items_checked 0
STAT items:1:lrutail_reflocked 0
END
{% endhighlight %}


---

#### stats cachedump slab\_id limit\_num

 查看内容

{% highlight bash %}
stats cachedump 1 0
ITEM a [8 b; 1442995392 s]
ITEM abc [8 b; 1442995392 s]
ITEM def [6 b; 1442995392 s]
ITEM username [8 b; 1442995392 s]
END
{% endhighlight %}

>通过stats items 和 stats cachedump slab\_id limit\_num配合get命令可以遍历memcached的记录。



---

#### stats slabs/sizes/reset

{% highlight bash %}
stats slabs
STAT 1:chunk_size 96
STAT 1:chunks_per_page 10922
STAT 1:total_pages 1
STAT 1:total_chunks 10922
STAT 1:used_chunks 4
STAT 1:free_chunks 10918
STAT 1:free_chunks_end 0
STAT 1:mem_requested 305
STAT 1:get_hits 21
STAT 1:cmd_set 19
STAT 1:delete_hits 1
STAT 1:incr_hits 0
STAT 1:decr_hits 0
STAT 1:cas_hits 1
STAT 1:cas_badval 1
STAT 1:touch_hits 0
STAT active_slabs 1
STAT total_malloced 1048512
END
stats sizes
STAT 96 4
END
stats reset
RESET
{% endhighlight %}




---

### version

{% highlight bash %}
version
VERSION 1.4.24
{% endhighlight %}

---

### 存储命令

存储命令的格式：

{% highlight bash %}
<command name> <key> <flags> <exptime> <bytes>
<data block>
{% endhighlight %}


参数说明如下：


Item     | Value
-------- | ---
command name |	set/add/replace
key	|  查找关键字
flags	|客户机使用它存储关于键值对的额外信息
exptime |	该数据的存活时间，0表示永远
bytes	|存储字节数
data block|存储的数据块（可直接理解为key-value结构中的value）


#### set/get

设定KEY 而不论是否存在

{% highlight bash %}
set abc 0 0 8 
123456789
CLIENT_ERROR bad data chunk
ERROR
get abc
END
set abc 0 0 8  
12345678
STORED
get abc
VALUE abc 0 8
12345678
END
set def 0 0 6 
ab


STORED
get def
VALUE def 0 6
ab


END
{% endhighlight %}

---

#### delete 

删除存在的KEY

{% highlight bash %}
get abc   
VALUE abc 0 8
12345678
END
delete abc
DELETED
get abc
END
delete ioio
NOT_FOUND
{% endhighlight %}

---

#### add

添加不存在的KEY

{% highlight bash %}
add abc 0 0 8 
qwertyui
STORED
get abc
VALUE abc 0 8
qwertyui
END
add abc 0 0 8
iuytrewq
NOT_STORED
get abc
VALUE abc 0 8
qwertyui
END
{% endhighlight %}


---


#### replace

替换已存在的KEY

{% highlight bash %}
get abc
VALUE abc 0 8
qwertyui
END
replace abc 0 0 9 
asdfghjkl
STORED
get abc
VALUE abc 0 9
asdfghjkl
END
replace ui 0 0 8
asdfghjkl
CLIENT_ERROR bad data chunk
ERROR
{% endhighlight %}

---

#### gets

查看修改tag

{% highlight bash %}
get abc
VALUE abc 0 9
asdfghjkl
END
gets abc
VALUE abc 0 9 8
asdfghjkl
END
set abc 0 0 8 
zxcvbnml
STORED
get abc
VALUE abc 0 8
zxcvbnml
END
gets abc
VALUE abc 0 8 9
zxcvbnml
END
{% endhighlight %}


---

#### cas

cas即checked and set的意思，只有当最后一个参数和gets所获取的参数匹配时才能存储，否则返回“EXISTS”

{% highlight bash %}
gets a
VALUE a 0 8 11
lkjhgfds
END
cas a 0 0 8 12
asdfghjk
EXISTS
gets a
VALUE a 0 8 11
lkjhgfds
END
cas a 0 0 8 11
asdfdsas
STORED
gets a 
VALUE a 0 8 12
asdfdsas
END
{% endhighlight %}


---

#### append

在现有的缓存数据后添加缓存数据，如现有缓存的key不存在服务器响应为ERROR

{% highlight bash %}
get a
VALUE a 0 8
asdfdsas
END
append a 0 0 8 
lkjhhjkl
STORED
get a 
VALUE a 0 16
asdfdsaslkjhhjkl
END
append uu
ERROR
{% endhighlight %}

---

#### prepend

和append非常类似，但它的作用是在现有的缓存数据前添加缓存数据

{% highlight bash %}
get a 
VALUE a 0 16
asdfdsaslkjhhjkl
END
prepend a 0 0 8
nmjhnmjh
STORED
get a
VALUE a 0 24
nmjhnmjhasdfdsaslkjhhjkl
END
prepend pp
ERROR
{% endhighlight %}


---

#### flush_all

flush\_all 实际上没有立即释放项目所占用的内存，而是在随后陆续有新的项目被储存时执行（这是由memcached的懒惰检测和删除机制决定的）。


flush\_all 效果是它导致所有更新时间早于 flush_all 所设定时间的项目，在被执行取回命令时命令被忽略。


{% highlight bash %}
get a 
VALUE a 0 24
nmjhnmjhasdfdsaslkjhhjkl
END
flush_all 
OK
get a
END
get b
END
get c
END
get username 
END
{% endhighlight %}

---

### quit

{% highlight bash %}
ERROR

ERROR
quit
Connection closed by foreign host.
[root@h101 memcached-1.4.24]# 
{% endhighlight %}

---

# 命令汇总

* **`wget  http://www.memcached.org/files/memcached-1.4.24.tar.gz`**
* **`tar -zxvf memcached-1.4.24.tar.gz`**
* **`cd memcached-1.4.24`**
* **`./configure`**
* **`rpm -ql  libevent-1.4.13-4.el6.x86_64`**
* **`yum -y install libevent-devel.x86_64`**
* **`./configure`**
* **`make`**
* **`make install`**
* **`ll /usr/local/bin/ | grep memcached`**
* **`/usr/local/bin/memcached  -h`**
* **`/usr/local/bin/memcached  -d  -m 1024 -p 12345 -u cc -c 512 -t 10`**
* **`kill  8732`**
* **`netstat  -ant | grep 12345`**
* **`pstree -p  8835`**
* **`ps -mp 8835`**
* **`telnet localhost 12354`**


---

[memcached]:http://memcached.org/
