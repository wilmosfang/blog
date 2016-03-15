---
layout: post
title:  SQLite 基础
categories:  linux sqlite
wc: 1455  5519 54763 
excerpt: sqlite的下载，安装，基础命令，交互输入，帮助，环境配置，数据类型，数据库与表的创建修改查看，导入导出，数据库的附加分离，基础信息表，和sqlite下的基本SQL操作
comments: true
---



# 前言

**[SQLite][sqlite]** 是一个开源的进程内库，实现了自给自足、无服务端、零配置、事务性的 SQL 数据库引擎

>SQLite is a software library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine.

目前 **[SQLite][sqlite]** 应用极其广泛，我们几乎人手都使用着一个，因为它的轻量特性，IOS和Andriod终端里都嵌入了 **[SQLite][sqlite]** 作为本地数据库，同时它还大量地使用在了各类嵌入式系统中

程序开发的过程中，也可以使用它来替代重量型的RDBMS，类似于 Mysql或Postgresql，来进行简单功能测试

这里分享一下 **[SQLite][sqlite]** 的相关操作基础，详细内容可以参考 **[官方文档][sqlite_doc]** , 网络资料参考了 **[RUNOOB][sqlite_ref]** 


> **Tip:** 当前的最新版本为 **SQLite Release 3.11.1** On 2016-03-03

---


# 概要

* TOC
{:toc}



---

## 下载

**[SQLite][sqlite]** 的 **[下载地址][sqlite_dl]**

{% highlight bash %}
[root@h102 sqlite]# wget http://www.sqlite.org/2016/sqlite-autoconf-3110100.tar.gz
--2016-03-11 23:28:10--  http://www.sqlite.org/2016/sqlite-autoconf-3110100.tar.gz
Resolving www.sqlite.org... 67.18.92.124, 2600:3c00::f03c:91ff:fe96:b959
Connecting to www.sqlite.org|67.18.92.124|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2359545 (2.2M) [application/x-gzip]
Saving to: “sqlite-autoconf-3110100.tar.gz”

100%[============================================================================================>] 2,359,545   16.4K/s   in 2m 3s   

2016-03-11 23:30:14 (18.7 KB/s) - “sqlite-autoconf-3110100.tar.gz” saved [2359545/2359545]

[root@h102 sqlite]# 
[root@h102 sqlite]# ls
sqlite-autoconf-3110100.tar.gz
[root@h102 sqlite]# sha1sum sqlite-autoconf-3110100.tar.gz 
c4b4dcd735a4daf5a2e2bb90f374484c8d4dad29  sqlite-autoconf-3110100.tar.gz
[root@h102 sqlite]# 
{% endhighlight %}




---

## 解压安装


### 解压安装命令

使用下列命令进行解压安装

{% highlight bash %}
tar -zxvf sqlite-autoconf-3110100.tar.gz 
cd sqlite-autoconf-3110100
./configure --prefix=/usr/local/sqlite3.11
make
make install
{% endhighlight %}

### 详细安装过程

{% highlight bash %}
[root@h102 sqlite]# tar -zxvf sqlite-autoconf-3110100.tar.gz 
sqlite-autoconf-3110100/
sqlite-autoconf-3110100/config.sub
sqlite-autoconf-3110100/shell.c
sqlite-autoconf-3110100/sqlite3.c
sqlite-autoconf-3110100/configure.ac
sqlite-autoconf-3110100/sqlite3.1
sqlite-autoconf-3110100/install-sh
sqlite-autoconf-3110100/Makefile.msc
sqlite-autoconf-3110100/compile
sqlite-autoconf-3110100/INSTALL
sqlite-autoconf-3110100/tea/
sqlite-autoconf-3110100/tea/generic/
sqlite-autoconf-3110100/tea/generic/tclsqlite3.c
sqlite-autoconf-3110100/tea/license.terms
sqlite-autoconf-3110100/tea/configure.ac
sqlite-autoconf-3110100/tea/tclconfig/
sqlite-autoconf-3110100/tea/tclconfig/install-sh
sqlite-autoconf-3110100/tea/tclconfig/tcl.m4
sqlite-autoconf-3110100/tea/README
sqlite-autoconf-3110100/tea/pkgIndex.tcl.in
sqlite-autoconf-3110100/tea/aclocal.m4
sqlite-autoconf-3110100/tea/doc/
sqlite-autoconf-3110100/tea/doc/sqlite3.n
sqlite-autoconf-3110100/tea/win/
sqlite-autoconf-3110100/tea/win/rules.vc
sqlite-autoconf-3110100/tea/win/makefile.vc
sqlite-autoconf-3110100/tea/win/nmakehlp.c
sqlite-autoconf-3110100/tea/configure
sqlite-autoconf-3110100/tea/Makefile.in
sqlite-autoconf-3110100/sqlite3ext.h
sqlite-autoconf-3110100/aclocal.m4
sqlite-autoconf-3110100/ltmain.sh
sqlite-autoconf-3110100/config.guess
sqlite-autoconf-3110100/sqlite3.rc
sqlite-autoconf-3110100/sqlite3.h
sqlite-autoconf-3110100/missing
sqlite-autoconf-3110100/depcomp
sqlite-autoconf-3110100/configure
sqlite-autoconf-3110100/Makefile.am
sqlite-autoconf-3110100/Makefile.in
sqlite-autoconf-3110100/sqlite3.pc.in
sqlite-autoconf-3110100/README.txt
[root@h102 sqlite]# ls
sqlite-autoconf-3110100  sqlite-autoconf-3110100.tar.gz
[root@h102 sqlite]# cd sqlite-autoconf-3110100
[root@h102 sqlite-autoconf-3110100]# ls
aclocal.m4    config.sub    depcomp     ltmain.sh    Makefile.msc  shell.c    sqlite3ext.h   sqlite3.rc
compile       configure     INSTALL     Makefile.am  missing       sqlite3.1  sqlite3.h      tea
config.guess  configure.ac  install-sh  Makefile.in  README.txt    sqlite3.c  sqlite3.pc.in
[root@h102 sqlite-autoconf-3110100]#
[root@h102 sqlite-autoconf-3110100]# ./configure --prefix=/usr/local/sqlite3.11
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for style of include used by make... GNU
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
checking dependency style of gcc... gcc3
checking for special C compiler options needed for large files... no
checking for _FILE_OFFSET_BITS value needed for large files... no
checking for gcc... (cached) gcc
checking whether we are using the GNU C compiler... (cached) yes
checking whether gcc accepts -g... (cached) yes
checking for gcc option to accept ISO C89... (cached) none needed
checking whether gcc understands -c and -o together... (cached) yes
checking dependency style of gcc... (cached) gcc3
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking how to print strings... printf
checking for a sed that does not truncate output... /bin/sed
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for fgrep... /bin/grep -F
checking for ld used by gcc... /usr/bin/ld
checking if the linker (/usr/bin/ld) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /usr/bin/nm -B
checking the name lister (/usr/bin/nm -B) interface... BSD nm
checking whether ln -s works... yes
checking the maximum length of command line arguments... 1966080
checking how to convert x86_64-unknown-linux-gnu file names to x86_64-unknown-linux-gnu format... func_convert_file_noop
checking how to convert x86_64-unknown-linux-gnu file names to toolchain format... func_convert_file_noop
checking for /usr/bin/ld option to reload object files... -r
checking for objdump... objdump
checking how to recognize dependent libraries... pass_all
checking for dlltool... no
checking how to associate runtime and link libraries... printf %s\n
checking for ar... ar
checking for archiver @FILE support... @
checking for strip... strip
checking for ranlib... ranlib
checking command to parse /usr/bin/nm -B output from gcc object... ok
checking for sysroot... no
checking for a working dd... /bin/dd
checking how to truncate binary pipes... /bin/dd bs=4096 count=1
checking for mt... no
checking if : is a manifest tool... no
checking how to run the C preprocessor... gcc -E
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
checking for dlfcn.h... yes
checking for objdir... .libs
checking if gcc supports -fno-rtti -fno-exceptions... no
checking for gcc option to produce PIC... -fPIC -DPIC
checking if gcc PIC flag -fPIC -DPIC works... yes
checking if gcc static flag -static works... no
checking if gcc supports -c -o file.o... yes
checking if gcc supports -c -o file.o... (cached) yes
checking whether the gcc linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking whether -lc should be explicitly linked in... no
checking dynamic linker characteristics... GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... yes
checking whether to build static libraries... yes
checking for fdatasync... yes
checking for usleep... yes
checking for fullfsync... no
checking for localtime_r... yes
checking for gmtime_r... yes
checking whether strerror_r is declared... yes
checking for strerror_r... yes
checking whether strerror_r returns char *... no
checking for library containing readline... no
checking for library containing pthread_create... -lpthread
checking for library containing pthread_mutexattr_init... none required
checking for library containing dlopen... -ldl
checking for whether to support dynamic extensions... yes
checking for posix_fallocate... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating sqlite3.pc
config.status: executing depfiles commands
config.status: executing libtool commands
[root@h102 sqlite-autoconf-3110100]# echo $?
0
[root@h102 sqlite-autoconf-3110100]# make 
/bin/sh ./libtool  --tag=CC   --mode=compile gcc -DPACKAGE_NAME=\"sqlite\" -DPACKAGE_TARNAME=\"sqlite\" -DPACKAGE_VERSION=\"3.11.1\" -DPACKAGE_STRING=\"sqlite\ 3.11.1\" -DPACKAGE_BUGREPORT=\"http://www.sqlite.org\" -DPACKAGE_URL=\"\" -DPACKAGE=\"sqlite\" -DVERSION=\"3.11.1\" -DSTDC_HEADERS=1 -DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -DHAVE_DLFCN_H=1 -DLT_OBJDIR=\".libs/\" -DHAVE_FDATASYNC=1 -DHAVE_USLEEP=1 -DHAVE_LOCALTIME_R=1 -DHAVE_GMTIME_R=1 -DHAVE_DECL_STRERROR_R=1 -DHAVE_STRERROR_R=1 -DHAVE_POSIX_FALLOCATE=1 -I.    -D_REENTRANT=1 -DSQLITE_THREADSAFE=1    -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -g -O2 -MT sqlite3.lo -MD -MP -MF .deps/sqlite3.Tpo -c -o sqlite3.lo sqlite3.c
libtool: compile:  gcc -DPACKAGE_NAME=\"sqlite\" -DPACKAGE_TARNAME=\"sqlite\" -DPACKAGE_VERSION=\"3.11.1\" "-DPACKAGE_STRING=\"sqlite 3.11.1\"" -DPACKAGE_BUGREPORT=\"http://www.sqlite.org\" -DPACKAGE_URL=\"\" -DPACKAGE=\"sqlite\" -DVERSION=\"3.11.1\" -DSTDC_HEADERS=1 -DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -DHAVE_DLFCN_H=1 -DLT_OBJDIR=\".libs/\" -DHAVE_FDATASYNC=1 -DHAVE_USLEEP=1 -DHAVE_LOCALTIME_R=1 -DHAVE_GMTIME_R=1 -DHAVE_DECL_STRERROR_R=1 -DHAVE_STRERROR_R=1 -DHAVE_POSIX_FALLOCATE=1 -I. -D_REENTRANT=1 -DSQLITE_THREADSAFE=1 -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -g -O2 -MT sqlite3.lo -MD -MP -MF .deps/sqlite3.Tpo -c sqlite3.c  -fPIC -DPIC -o .libs/sqlite3.o
libtool: compile:  gcc -DPACKAGE_NAME=\"sqlite\" -DPACKAGE_TARNAME=\"sqlite\" -DPACKAGE_VERSION=\"3.11.1\" "-DPACKAGE_STRING=\"sqlite 3.11.1\"" -DPACKAGE_BUGREPORT=\"http://www.sqlite.org\" -DPACKAGE_URL=\"\" -DPACKAGE=\"sqlite\" -DVERSION=\"3.11.1\" -DSTDC_HEADERS=1 -DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -DHAVE_DLFCN_H=1 -DLT_OBJDIR=\".libs/\" -DHAVE_FDATASYNC=1 -DHAVE_USLEEP=1 -DHAVE_LOCALTIME_R=1 -DHAVE_GMTIME_R=1 -DHAVE_DECL_STRERROR_R=1 -DHAVE_STRERROR_R=1 -DHAVE_POSIX_FALLOCATE=1 -I. -D_REENTRANT=1 -DSQLITE_THREADSAFE=1 -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -g -O2 -MT sqlite3.lo -MD -MP -MF .deps/sqlite3.Tpo -c sqlite3.c -o sqlite3.o >/dev/null 2>&1
mv -f .deps/sqlite3.Tpo .deps/sqlite3.Plo
/bin/sh ./libtool  --tag=CC   --mode=link gcc -D_REENTRANT=1 -DSQLITE_THREADSAFE=1    -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -g -O2 -no-undefined -version-info 8:6:8  -o libsqlite3.la -rpath /usr/local/sqlite3.11/lib sqlite3.lo  -ldl -lpthread 
libtool: link: gcc -shared  -fPIC -DPIC  .libs/sqlite3.o   -ldl -lpthread  -g -O2   -Wl,-soname -Wl,libsqlite3.so.0 -o .libs/libsqlite3.so.0.8.6
libtool: link: (cd ".libs" && rm -f "libsqlite3.so.0" && ln -s "libsqlite3.so.0.8.6" "libsqlite3.so.0")
libtool: link: (cd ".libs" && rm -f "libsqlite3.so" && ln -s "libsqlite3.so.0.8.6" "libsqlite3.so")
libtool: link: ar cru .libs/libsqlite3.a  sqlite3.o
libtool: link: ranlib .libs/libsqlite3.a
libtool: link: ( cd ".libs" && rm -f "libsqlite3.la" && ln -s "../libsqlite3.la" "libsqlite3.la" )
gcc -DPACKAGE_NAME=\"sqlite\" -DPACKAGE_TARNAME=\"sqlite\" -DPACKAGE_VERSION=\"3.11.1\" -DPACKAGE_STRING=\"sqlite\ 3.11.1\" -DPACKAGE_BUGREPORT=\"http://www.sqlite.org\" -DPACKAGE_URL=\"\" -DPACKAGE=\"sqlite\" -DVERSION=\"3.11.1\" -DSTDC_HEADERS=1 -DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -DHAVE_DLFCN_H=1 -DLT_OBJDIR=\".libs/\" -DHAVE_FDATASYNC=1 -DHAVE_USLEEP=1 -DHAVE_LOCALTIME_R=1 -DHAVE_GMTIME_R=1 -DHAVE_DECL_STRERROR_R=1 -DHAVE_STRERROR_R=1 -DHAVE_POSIX_FALLOCATE=1 -I.    -D_REENTRANT=1 -DSQLITE_THREADSAFE=1    -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -DSQLITE_ENABLE_EXPLAIN_COMMENTS -g -O2 -MT sqlite3-shell.o -MD -MP -MF .deps/sqlite3-shell.Tpo -c -o sqlite3-shell.o `test -f 'shell.c' || echo './'`shell.c
mv -f .deps/sqlite3-shell.Tpo .deps/sqlite3-shell.Po
gcc -DPACKAGE_NAME=\"sqlite\" -DPACKAGE_TARNAME=\"sqlite\" -DPACKAGE_VERSION=\"3.11.1\" -DPACKAGE_STRING=\"sqlite\ 3.11.1\" -DPACKAGE_BUGREPORT=\"http://www.sqlite.org\" -DPACKAGE_URL=\"\" -DPACKAGE=\"sqlite\" -DVERSION=\"3.11.1\" -DSTDC_HEADERS=1 -DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -DHAVE_DLFCN_H=1 -DLT_OBJDIR=\".libs/\" -DHAVE_FDATASYNC=1 -DHAVE_USLEEP=1 -DHAVE_LOCALTIME_R=1 -DHAVE_GMTIME_R=1 -DHAVE_DECL_STRERROR_R=1 -DHAVE_STRERROR_R=1 -DHAVE_POSIX_FALLOCATE=1 -I.    -D_REENTRANT=1 -DSQLITE_THREADSAFE=1    -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -DSQLITE_ENABLE_EXPLAIN_COMMENTS -g -O2 -MT sqlite3-sqlite3.o -MD -MP -MF .deps/sqlite3-sqlite3.Tpo -c -o sqlite3-sqlite3.o `test -f 'sqlite3.c' || echo './'`sqlite3.c
mv -f .deps/sqlite3-sqlite3.Tpo .deps/sqlite3-sqlite3.Po
/bin/sh ./libtool  --tag=CC   --mode=link gcc -D_REENTRANT=1 -DSQLITE_THREADSAFE=1    -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -DSQLITE_ENABLE_EXPLAIN_COMMENTS -g -O2   -o sqlite3 sqlite3-shell.o sqlite3-sqlite3.o  -ldl -lpthread 
libtool: link: gcc -D_REENTRANT=1 -DSQLITE_THREADSAFE=1 -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_RTREE -DSQLITE_ENABLE_EXPLAIN_COMMENTS -g -O2 -o sqlite3 sqlite3-shell.o sqlite3-sqlite3.o  -ldl -lpthread
[root@h102 sqlite-autoconf-3110100]# echo $?
0
[root@h102 sqlite-autoconf-3110100]# make install 
make[1]: Entering directory `/usr/local/src/sqlite/sqlite-autoconf-3110100'
 /bin/mkdir -p '/usr/local/sqlite3.11/lib'
 /bin/sh ./libtool   --mode=install /usr/bin/install -c   libsqlite3.la '/usr/local/sqlite3.11/lib'
libtool: install: /usr/bin/install -c .libs/libsqlite3.so.0.8.6 /usr/local/sqlite3.11/lib/libsqlite3.so.0.8.6
libtool: install: (cd /usr/local/sqlite3.11/lib && { ln -s -f libsqlite3.so.0.8.6 libsqlite3.so.0 || { rm -f libsqlite3.so.0 && ln -s libsqlite3.so.0.8.6 libsqlite3.so.0; }; })
libtool: install: (cd /usr/local/sqlite3.11/lib && { ln -s -f libsqlite3.so.0.8.6 libsqlite3.so || { rm -f libsqlite3.so && ln -s libsqlite3.so.0.8.6 libsqlite3.so; }; })
libtool: install: /usr/bin/install -c .libs/libsqlite3.lai /usr/local/sqlite3.11/lib/libsqlite3.la
libtool: install: /usr/bin/install -c .libs/libsqlite3.a /usr/local/sqlite3.11/lib/libsqlite3.a
libtool: install: chmod 644 /usr/local/sqlite3.11/lib/libsqlite3.a
libtool: install: ranlib /usr/local/sqlite3.11/lib/libsqlite3.a
libtool: finish: PATH="/usr/local/rvm/gems/ruby-2.2.1/bin:/usr/local/rvm/gems/ruby-2.2.1@global/bin:/usr/local/rvm/rubies/ruby-2.2.1/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/rvm/bin:/root/bin:/sbin" ldconfig -n /usr/local/sqlite3.11/lib
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/sqlite3.11/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the '-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the 'LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the 'LD_RUN_PATH' environment variable
     during linking
   - use the '-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to '/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
 /bin/mkdir -p '/usr/local/sqlite3.11/bin'
  /bin/sh ./libtool   --mode=install /usr/bin/install -c sqlite3 '/usr/local/sqlite3.11/bin'
libtool: install: /usr/bin/install -c sqlite3 /usr/local/sqlite3.11/bin/sqlite3
 /bin/mkdir -p '/usr/local/sqlite3.11/include'
 /usr/bin/install -c -m 644 sqlite3.h sqlite3ext.h '/usr/local/sqlite3.11/include'
 /bin/mkdir -p '/usr/local/sqlite3.11/share/man/man1'
 /usr/bin/install -c -m 644 sqlite3.1 '/usr/local/sqlite3.11/share/man/man1'
 /bin/mkdir -p '/usr/local/sqlite3.11/lib/pkgconfig'
 /usr/bin/install -c -m 644 sqlite3.pc '/usr/local/sqlite3.11/lib/pkgconfig'
make[1]: Leaving directory `/usr/local/src/sqlite/sqlite-autoconf-3110100'
[root@h102 sqlite-autoconf-3110100]# echo $?
0
[root@h102 sqlite-autoconf-3110100]# ll /usr/local/sqlite3.11/
total 16
drwxr-xr-x 2 root root 4096 Mar 12 00:05 bin
drwxr-xr-x 2 root root 4096 Mar 12 00:05 include
drwxr-xr-x 3 root root 4096 Mar 12 00:05 lib
drwxr-xr-x 3 root root 4096 Mar 12 00:05 share
[root@h102 sqlite-autoconf-3110100]# tree /usr/local/sqlite3.11/
/usr/local/sqlite3.11/
├── bin
│   └── sqlite3
├── include
│   ├── sqlite3ext.h
│   └── sqlite3.h
├── lib
│   ├── libsqlite3.a
│   ├── libsqlite3.la
│   ├── libsqlite3.so -> libsqlite3.so.0.8.6
│   ├── libsqlite3.so.0 -> libsqlite3.so.0.8.6
│   ├── libsqlite3.so.0.8.6
│   └── pkgconfig
│       └── sqlite3.pc
└── share
    └── man
        └── man1
            └── sqlite3.1

7 directories, 10 files
[root@h102 sqlite-autoconf-3110100]# 

{% endhighlight %}

### 版本确认

{% highlight bash %}
[root@h102 ~]# /usr/local/sqlite3.11/bin/sqlite3 -version
3.11.1 2016-03-03 16:17:53 f047920ce16971e573bc6ec9a48b118c9de2b3a7
[root@h102 ~]# 
{% endhighlight %}

### 帮助信息

{% highlight bash %}
[root@h102 ~]# /usr/local/sqlite3.11/bin/sqlite3 --help 
Usage: /usr/local/sqlite3.11/bin/sqlite3 [OPTIONS] FILENAME [SQL]
FILENAME is the name of an SQLite database. A new database is created
if the file does not previously exist.
OPTIONS include:
   -ascii               set output mode to 'ascii'
   -bail                stop after hitting an error
   -batch               force batch I/O
   -column              set output mode to 'column'
   -cmd COMMAND         run "COMMAND" before reading stdin
   -csv                 set output mode to 'csv'
   -echo                print commands before execution
   -init FILENAME       read/process named file
   -[no]header          turn headers on or off
   -help                show this message
   -html                set output mode to HTML
   -interactive         force interactive I/O
   -line                set output mode to 'line'
   -list                set output mode to 'list'
   -lookaside SIZE N    use N entries of SZ bytes for lookaside memory
   -mmap N              default mmap size set to N
   -newline SEP         set output row separator. Default: '\n'
   -nullvalue TEXT      set text string for NULL values. Default ''
   -pagecache SIZE N    use N slots of SZ bytes each for page cache memory
   -scratch SIZE N      use N slots of SZ bytes each for scratch memory
   -separator SEP       set output column separator. Default: '|'
   -stats               print memory stats before each finalize
   -version             show SQLite version
   -vfs NAME            use NAME as the default VFS
[root@h102 ~]# 

{% endhighlight %}

比前几个版本多出了不少新的参数

---

## SQLite命令


### 交互式输入


{% highlight bash %}
[root@h102 bin]# ./sqlite3 
SQLite version 3.11.1 2016-03-03 16:17:53
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> 
sqlite> 

{% endhighlight %}

### help

{% highlight bash %}
sqlite> .help
.backup ?DB? FILE      Backup DB (default "main") to FILE
.bail on|off           Stop after hitting an error.  Default OFF
.binary on|off         Turn binary output on or off.  Default OFF
.changes on|off        Show number of rows changed by SQL
.clone NEWDB           Clone data into NEWDB from the existing database
.databases             List names and files of attached databases
.dbinfo ?DB?           Show status information about the database
.dump ?TABLE? ...      Dump the database in an SQL text format
                         If TABLE specified, only dump tables matching
                         LIKE pattern TABLE.
.echo on|off           Turn command echo on or off
.eqp on|off            Enable or disable automatic EXPLAIN QUERY PLAN
.exit                  Exit this program
.explain ?on|off|auto? Turn EXPLAIN output mode on or off or to automatic
.fullschema            Show schema and the content of sqlite_stat tables
.headers on|off        Turn display of headers on or off
.help                  Show this message
.import FILE TABLE     Import data from FILE into TABLE
.indexes ?TABLE?       Show names of all indexes
                         If TABLE specified, only show indexes for tables
                         matching LIKE pattern TABLE.
.limit ?LIMIT? ?VAL?   Display or change the value of an SQLITE_LIMIT
.load FILE ?ENTRY?     Load an extension library
.log FILE|off          Turn logging on or off.  FILE can be stderr/stdout
.mode MODE ?TABLE?     Set output mode where MODE is one of:
                         ascii    Columns/rows delimited by 0x1F and 0x1E
                         csv      Comma-separated values
                         column   Left-aligned columns.  (See .width)
                         html     HTML <table> code
                         insert   SQL insert statements for TABLE
                         line     One value per line
                         list     Values delimited by .separator strings
                         tabs     Tab-separated values
                         tcl      TCL list elements
.nullvalue STRING      Use STRING in place of NULL values
.once FILENAME         Output for the next SQL command only to FILENAME
.open ?FILENAME?       Close existing database and reopen FILENAME
.output ?FILENAME?     Send output to FILENAME or stdout
.print STRING...       Print literal STRING
.prompt MAIN CONTINUE  Replace the standard prompts
.quit                  Exit this program
.read FILENAME         Execute SQL in FILENAME
.restore ?DB? FILE     Restore content of DB (default "main") from FILE
.save FILE             Write in-memory database into FILE
.scanstats on|off      Turn sqlite3_stmt_scanstatus() metrics on or off
.schema ?TABLE?        Show the CREATE statements
                         If TABLE specified, only show tables matching
                         LIKE pattern TABLE.
.separator COL ?ROW?   Change the column separator and optionally the row
                         separator for both the output mode and .import
.shell CMD ARGS...     Run CMD ARGS... in a system shell
.show                  Show the current values for various settings
.stats on|off          Turn stats on or off
.system CMD ARGS...    Run CMD ARGS... in a system shell
.tables ?TABLE?        List names of tables
                         If TABLE specified, only list tables matching
                         LIKE pattern TABLE.
.timeout MS            Try opening locked tables for MS milliseconds
.timer on|off          Turn SQL timer on or off
.trace FILE|off        Output each SQL statement as it is run
.vfsinfo ?AUX?         Information about the top-level VFS
.vfslist               List all available VFSes
.vfsname ?AUX?         Print the name of the VFS stack
.width NUM1 NUM2 ...   Set column widths for "column" mode
                         Negative values right-justify
sqlite> 
{% endhighlight %}

### show


查看当前的默认配置

{% highlight bash %}
sqlite> .show
        echo: off
         eqp: off
     explain: auto
     headers: off
        mode: list
   nullvalue: ""
      output: stdout
colseparator: "|"
rowseparator: "\n"
       stats: off
       width: 
sqlite> 
{% endhighlight %}


---

## 数据类型


Type    | Comment
-------- | ---
NULL|  NULL 值。
INTEGER| 带符号的整数，根据值的大小存储在 1、2、3、4、6 或 8 字节中
REAL| 浮点值，存储为 8 字节的 IEEE 浮点数字
TEXT|文本字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储
BLOB|blob 数据，完全根据它的输入存储


> **Tip:**  SQLite 没有单独的 Boolean 存储类。相反，布尔值被存储为整数 0（false）和 1（true）


> **Note:** SQLite 没有一个单独的用于存储日期和/或时间的存储类，但 SQLite 能够把日期和时间存储为 TEXT、REAL 或 INTEGER 值

Type    | Comment
-------- | ---
TEXT| 格式为 "YYYY-MM-DD HH:MM:SS.SSS" 的日期
REAL| 从公元前 4714 年 11 月 24 日格林尼治时间的正午开始算起的天数
INTEGER|从 1970-01-01 00:00:00 UTC 算起的秒数




---


## 创建数据库


{% highlight bash %}
[root@h102 bin]# ./sqlite3 test.db
SQLite version 3.11.1 2016-03-03 16:17:53
Enter ".help" for usage hints.
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db                         
sqlite>
{% endhighlight %}

---

## 创建表

{% highlight bash %}
sqlite> create table test ( id int primary key not null, name text );
sqlite> .tables
test
sqlite> .schema test
CREATE TABLE test ( id int primary key not null, name text );
sqlite> 
sqlite> insert into test values(1,"hello");
sqlite> insert into test values(2,"hello");         
sqlite> insert into test values(3,"hello");
sqlite> select * from test;
1|hello
2|hello
3|hello
sqlite> 
{% endhighlight %}

> **Tip:** 以点开头的管理命令，如 **`.tables`** 和 **`.schema`** 是不能接 **`;`** 的，而增删改查类操作是必须要以 **`;`** 结尾的


{% highlight bash %}
sqlite> .table
test
sqlite> .table;
Error: unknown command or invalid arguments:  "table;". Enter ".help" for help
sqlite> insert into test values(4,"hello");
sqlite> insert into test values(5,"hello")
   ...> 
   ...> 
   ...> 
   ...> 
   ...> 
   ...> 
   ...> 
   ...> ;
sqlite> insert into test values(5,"hello")
   ...> 
   ...> 
   ...> 
   ...> 
   ...> 
   ...> 
   ...> ;
Error: UNIQUE constraint failed: test.id
sqlite> 
{% endhighlight %}



## 导入导出数据库

### 导出数据库

可以使用这种方式来将sqlite数据转化为SQL

{% highlight bash %}
[root@h102 bin]# ./sqlite3 test.db .dump > test.sql
[root@h102 bin]# cat test.sql 
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE test ( id int primary key not null, name text );
INSERT INTO "test" VALUES(1,'hello');
INSERT INTO "test" VALUES(2,'hello');
INSERT INTO "test" VALUES(3,'hello');
INSERT INTO "test" VALUES(4,'hello');
INSERT INTO "test" VALUES(5,'hello');
COMMIT;
[root@h102 bin]# 
{% endhighlight %}


也可以定向的只dump一个表，但这个操作没法在shell中完成，只能在sqlite中完成

{% highlight bash %}
sqlite> .dump test
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE test ( id int primary key not null, name text );
INSERT INTO "test" VALUES(1,'hello');
INSERT INTO "test" VALUES(2,'hello');
INSERT INTO "test" VALUES(3,'hello');
INSERT INTO "test" VALUES(4,'hello');
INSERT INTO "test" VALUES(5,'hello');
INSERT INTO "test" VALUES(12,'12');
INSERT INTO "test" VALUES(13,'www');
COMMIT;
sqlite> 
{% endhighlight %}


### 导入数据库


{% highlight bash %}
[root@h102 bin]# ls
sqlite3  test.db  test.sql
[root@h102 bin]# ./sqlite3 test_tmp.db < test.sql 
[root@h102 bin]# ./sqlite3 test_tmp.db 
SQLite version 3.11.1 2016-03-03 16:17:53
Enter ".help" for usage hints.
sqlite> .tables 
test
sqlite> .schema test
CREATE TABLE test ( id int primary key not null, name text );
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test_tmp.db                     
1    temp                                                                       
sqlite> select * from test;
1|hello
2|hello
3|hello
4|hello
5|hello
sqlite> 
{% endhighlight %}

---

## 附加数据库

sqlite可以对多个数据库(多个文件)进行操作

{% highlight bash %}
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db                         
sqlite>
sqlite> attach database 'test_tmp.db' as 'abc';
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db                         
2    abc              /usr/local/sqlite3.11/bin/test_tmp.db                     
sqlite> .tables
abc.test  test    
sqlite> select * from abc.test;
1|hello
2|hello
3|hello
4|hello
5|hello
sqlite> select * from test;
1|hello
2|hello
3|hello
4|hello
5|hello
sqlite> create table t2(id int primary key not null, name text );
sqlite> .tables
abc.test  t2        test    
sqlite> create table abc.t2(id int primary key not null, name text );  
sqlite> .tables
abc.t2    abc.test  t2        test    
sqlite> 
sqlite> INSERT INTO abc.t2  VALUES(1,'hello');
sqlite> INSERT INTO abc.t2  VALUES(3,'hello');
sqlite> INSERT INTO abc.t2  VALUES(5,'hello');
sqlite> 
sqlite> INSERT INTO t2 VALUES(2,'hello');
sqlite> INSERT INTO t2 VALUES(4,'hello');
sqlite> select * from abc.t2;
1|hello
3|hello
5|hello
sqlite> select * from t2;    
2|hello
4|hello
sqlite> 

{% endhighlight %}

---

## 分离数据库

无法分离 **main** 和 **temp** 数据库

{% highlight bash %}
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db                         
2    abc              /usr/local/sqlite3.11/bin/test_tmp.db                     
sqlite> detach database main;
Error: cannot detach database main
sqlite> detach database abc;
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db                         
sqlite> 
{% endhighlight %} 

> **Tip:**  可以使用 attach 的方法来创建数据库别名

{% highlight bash %}
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db   
sqlite> .tables
hello  t2     test 
sqlite> attach database 'test.db' as 'new';
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /usr/local/sqlite3.11/bin/test.db                         
1    temp                                                                       
2    new              /usr/local/sqlite3.11/bin/test.db                         
sqlite> .tables
hello      new.hello  new.t2     new.test   t2         test   
sqlite> select * from new.t2;
2|hello
4|hello
sqlite> 
sqlite> detach database temp;
Error: cannot detach database temp
sqlite> detach database main;
Error: cannot detach database main
sqlite> 
{% endhighlight %}

---

## 创建表

{% highlight bash %}
sqlite> .tables 
hello  t2     test 
sqlite> create table ui(
   ...> id int,
   ...> name text,
   ...> age int);
sqlite> .schema ui
CREATE TABLE ui(
id int,
name text,
age int);
sqlite> .tables
hello  t2     test   ui   
sqlite> 

{% endhighlight %}



##  删除表

{% highlight bash %}
sqlite> .tables
company     department  hello       t2          test        ui        
sqlite> drop table t2;
sqlite> .tables
company     department  hello       test        ui        
sqlite>
{% endhighlight %}


## 插入数据


{% highlight bash %}
sqlite> .schema test
CREATE TABLE test ( id int primary key not null, name text );
sqlite> insert into test (id,name) values ( 12,"12");
sqlite> insert into test values (13,"www");
sqlite> .schema company
CREATE TABLE company(
id int primary key not null,
name text not null,
age int not null,
address char(50),
salary real
);
sqlite> insert into company (id,name,age,address,salary) values VALUES (1, 'Paul', 32, 'California', 20000.00 );
Error: near "VALUES": syntax error
sqlite> insert into company (id,name,age,address,salary) values (1, 'Paul', 32, 'California', 20000.00 );
sqlite> insert into company (id,name,age,address,salary) values (2, 'Allen', 25, 'Texas', 15000.00 );
sqlite> insert into company (id,name,age,address,salary) values (3, 'Teddy', 23, 'Norway', 20000.00 );
sqlite> insert into company (id,name,age,address,salary) values (4, 'Mark', 25, 'Rich-Mond ', 65000.00 );
sqlite> insert into company values (5, 'David', 27, 'Texas', 85000.00 );
sqlite> insert into company values (6, 'Kim', 22, 'South-Hall', 45000.00 );
sqlite> insert into company values (7, 'James', 24, 'Houston', 10000.00 );
sqlite> 
{% endhighlight %}


---


## 查询数据

### 调整格式

使用下面的方法来格式化输出

{% highlight bash %}
sqlite> .show
        echo: off
         eqp: off
     explain: auto
     headers: off
        mode: list
   nullvalue: ""
      output: stdout
colseparator: "|"
rowseparator: "\n"
       stats: off
       width: 
sqlite> .headers on
sqlite> .mode column
sqlite> .show
        echo: off
         eqp: off
     explain: auto
     headers: on
        mode: column
   nullvalue: ""
      output: stdout
colseparator: "|"
rowseparator: "\n"
       stats: off
       width: 
sqlite> 
{% endhighlight %}

再进行查询

{% highlight bash %}
sqlite> select * from company;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0   
sqlite> 
{% endhighlight %}

格式明显变工整了

{% highlight bash %}
sqlite> select name,salary,id from company;
name        salary      id        
----------  ----------  ----------
Paul        20000.0     1         
Allen       15000.0     2         
Teddy       20000.0     3         
Mark        65000.0     4         
David       85000.0     5         
Kim         45000.0     6         
James       10000.0     7         
sqlite> 
{% endhighlight %}

### 设置列宽

{% highlight bash %}
sqlite> .width 6 , 15, 10
sqlite> select name,salary,id from company;
name    salary      id             
------  ----------  ---------------
Paul    20000.0     1              
Allen   15000.0     2              
Teddy   20000.0     3              
Mark    65000.0     4              
David   85000.0     5              
Kim     45000.0     6              
James   10000.0     7              
sqlite>
{% endhighlight %}

## 隐藏的信息管理表

{% highlight bash %}
sqlite> .schema sqlite_master
CREATE TABLE sqlite_master (
  type text,
  name text,
  tbl_name text,
  rootpage integer,
  sql text
);
sqlite> 
{% endhighlight %}

这张表里包含了其它表的信息


{% highlight bash %}
sqlite>  select * from sqlite_master;
type        name        tbl_name    rootpage    sql                                                         
----------  ----------  ----------  ----------  ------------------------------------------------------------
table       test        test        2           CREATE TABLE test ( id int primary key not null, name text )
index       sqlite_aut  test        3                                                                       
table       hello       hello       6           CREATE TABLE hello (
id int primary key not null,
age int,
n
index       sqlite_aut  hello       7                                                                       
table       ui          ui          8           CREATE TABLE ui(
id int,
name text,
age int)                
table       company     company     9           CREATE TABLE company(
id int primary key not null,
name text
index       sqlite_aut  company     10                                                                      
table       department  department  11          CREATE TABLE department(
id int primary key not null,
dept c
index       sqlite_aut  department  12                                                                      
sqlite> 
{% endhighlight %}

---

## 算术运算



{% highlight bash %}
sqlite> select 20+30;
20+30 = 50
sqlite> select 20-30;
20-30 = -10
sqlite> select 20*30;
20*30 = 600
sqlite> select 20/30;
20/30 = 0
sqlite> select 20%30;
20%30 = 20
sqlite> 
{% endhighlight %}

---

## 比较运算符

{% highlight bash %}
sqlite> .mode column
sqlite> select * from company; 
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0   
sqlite> select * from company where salary > 60000;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
sqlite> select * from company where salary = 15000;      
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
2           Allen       25          Texas       15000.0   
sqlite> select * from company where salary !=85000;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0   
sqlite> select * from company where salary >= 45000;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
sqlite> 

{% endhighlight %}


---

## 逻辑运算符


{% highlight bash %}
sqlite> SELECT * FROM COMPANY WHERE AGE <= 22 and SALARY >= 40000;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
6           Kim         22          South-Hall  45000.0   
sqlite> SELECT * FROM COMPANY WHERE AGE <= 22 or SALARY >= 40000;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
sqlite> SELECT * FROM COMPANY WHERE AGE IS NOT NULL;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0   
sqlite> SELECT * FROM COMPANY WHERE AGE is null;
sqlite> SELECT * FROM COMPANY WHERE NAME LIKE 'P%';     
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
sqlite> SELECT * FROM COMPANY WHERE NAME GLOB 'T%';
sqlite> SELECT * FROM COMPANY WHERE NAME GLOB 'Ki*';
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
6           Kim         22          South-Hall  45000.0   
sqlite> SELECT * FROM COMPANY WHERE NAME GLOB 'Te%';
sqlite> SELECT * FROM COMPANY WHERE NAME GLOB 'T*'; 
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
3           Teddy       23          Norway      20000.0   
sqlite> SELECT * FROM COMPANY WHERE AGE IN ( 24,32);
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
7           James       24          Houston     10000.0   
sqlite> SELECT * FROM COMPANY WHERE AGE BETWEEN 25 and 32;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
sqlite> SELECT AGE FROM COMPANY WHERE EXISTS (SELECT AGE FROM COMPANY WHERE SALARY > 65000);
age       
----------
32        
25        
23        
25        
27        
22        
24        
sqlite>  SELECT * FROM COMPANY WHERE AGE > (SELECT AGE FROM COMPANY WHERE SALARY > 15000);  
sqlite> SELECT * FROM COMPANY WHERE AGE > (SELECT AGE FROM COMPANY WHERE SALARY <15000); 
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
sqlite> 
{% endhighlight %}

---


## 位运算符


{% highlight bash %}
sqlite> .mode line
sqlite> select 6|5;
  6|5 = 7
sqlite> select 6&5;
  6&5 = 4
sqlite> select (~6);
 (~6) = -7
sqlite> select (6 << 2 );
(6 << 2 ) = 24
sqlite> select (6 >>1);
(6 >>1) = 3
sqlite> 
{% endhighlight %}

---

## 表达式

{% highlight bash %}
sqlite> SELECT ( 22 + 34 ) AS ADDITION;
ADDITION  
----------
56        
sqlite> SELECT COUNT(*) AS "RECORDS" FROM COMPANY;
RECORDS   
----------
7         
sqlite> SELECT CURRENT_TIMESTAMP;
CURRENT_TIMESTAMP  
-------------------
2016-03-14 06:46:16
sqlite>
{% endhighlight %}


---

## UPDATE

{% highlight bash %}
sqlite> select * from COMPANY;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0   
sqlite> UPDATE COMPANY SET ADDRESS = 'Texas' WHERE ID = 6;
sqlite> select * from company where id=6;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
6           Kim         22          Texas       45000.0   
sqlite> UPDATE COMPANY SET ADDRESS = 'test' ,SALARY =10;        
sqlite> select * from COMPANY;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          test        10.0      
2           Allen       25          test        10.0      
3           Teddy       23          test        10.0      
4           Mark        25          test        10.0      
5           David       27          test        10.0      
6           Kim         22          test        10.0      
7           James       24          test        10.0      
sqlite> 
{% endhighlight %}


---


## DELETE

{% highlight bash %}
sqlite> select * from company where id=7;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
7           James       24          test        10.0      
sqlite> delete from company where id=7;
sqlite> select * from company where id=7;
sqlite> select * from company;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          test        10.0      
2           Allen       25          test        10.0      
3           Teddy       23          test        10.0      
4           Mark        25          test        10.0      
5           David       27          test        10.0      
6           Kim         22          test        10.0      
sqlite> delete from company;
sqlite> select * from company;
sqlite>
{% endhighlight %}


---

## LIKE

{% highlight bash %}
sqlite> select * from company where age like '%5';
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
2           Allen       25          Texas       15000.0   
4           Mark        25          Rich-Mond   65000.0   
sqlite> 
sqlite> select * from company where address like '%-%';
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
4           Mark        25          Rich-Mond   65000.0   
6           Kim         22          South-Hall  45000.0   
sqlite> 
{% endhighlight %}


---

## GLOB


GLOB与LIKE类似，都是用来进行模糊匹配的，但是通配符使用的shell的规则 用**`*`** 替代 **`%`** 用 **`？`** 替代 **`_`**


{% highlight bash %}
sqlite> select * from company where age glob '*5';
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
2           Allen       25          Texas       15000.0   
4           Mark        25          Rich-Mond   65000.0   
sqlite> select * from company where address glob '*-*';
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
4           Mark        25          Rich-Mond   65000.0   
6           Kim         22          South-Hall  45000.0   
sqlite>
sqlite> select * from company where age glob '%5';
sqlite>
{% endhighlight %}

---

## LIMIT


{% highlight bash %}
sqlite> select * from company;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0   
sqlite> select * from company limit 3;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
sqlite> select * from company limit 3 offset 3;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
sqlite>
{% endhighlight %}


---

## ORDER BY

{% highlight bash %}
sqlite> select * from company order by salary;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
7           James       24          Houston     10000.0   
2           Allen       25          Texas       15000.0   
1           Paul        32          California  20000.0   
3           Teddy       23          Norway      20000.0   
6           Kim         22          South-Hall  45000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
sqlite> select * from company order by salary desc;
id          name        age         address     salary    
----------  ----------  ----------  ----------  ----------
5           David       27          Texas       85000.0   
4           Mark        25          Rich-Mond   65000.0   
6           Kim         22          South-Hall  45000.0   
1           Paul        32          California  20000.0   
3           Teddy       23          Norway      20000.0   
2           Allen       25          Texas       15000.0   
7           James       24          Houston     10000.0   
sqlite> 
{% endhighlight %}

---

## GROUP BY

{% highlight bash %}
sqlite> select age,sum(salary) from company group by age order by age ;
age         sum(salary)
----------  -----------
22          45000.0    
23          20000.0    
24          30000.0    
25          80000.0    
27          85000.0    
32          20000.0    
44          5000.0     
45          5000.0     
sqlite> select name,sum(salary) from company group by name order by name desc;
name        sum(salary)
----------  -----------
Teddy       20000.0    
Paul        40000.0    
Mark        65000.0    
Kim         45000.0    
James       20000.0    
David       85000.0    
Allen       15000.0    
sqlite>
{% endhighlight %}

---

## HAVING

在最终结果中进行过滤


{% highlight bash %}
sqlite> select name,count(*) from company group by name;
name        count(*)  
----------  ----------
Allen       1         
David       1         
James       3         
Kim         1         
Mark        1         
Paul        2         
Teddy       1         
sqlite> select name,count(*) from company group by name having count(*) < 2;
name        count(*)  
----------  ----------
Allen       1         
David       1         
Kim         1         
Mark        1         
Teddy       1         
sqlite> 
{% endhighlight %}


## DISTINCT

去重


{% highlight bash %}
sqlite> select name from company;
name      
----------
Paul      
Allen     
Teddy     
Mark      
David     
Kim       
James     
Paul      
James     
James     
sqlite> select distinct name from company;
name      
----------
Paul      
Allen     
Teddy     
Mark      
David     
Kim       
James     
sqlite>
{% endhighlight %}


---

## 退出SQLite


**.quit** 和 **.exit** 都可以用来退出sqlite

{% highlight bash %}
[root@h102 bin]# ./sqlite3 
SQLite version 3.11.1 2016-03-03 16:17:53
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> .quit
[root@h102 bin]# ./sqlite3 
SQLite version 3.11.1 2016-03-03 16:17:53
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> .exit
[root@h102 bin]# 
{% endhighlight %}

---

# 命令汇总

* **`wget http://www.sqlite.org/2016/sqlite-autoconf-3110100.tar.gz`**
* **`sha1sum sqlite-autoconf-3110100.tar.gz`**
* **`tar -zxvf sqlite-autoconf-3110100.tar.gz`**
* **`cd sqlite-autoconf-3110100`**
* **`./configure --prefix=/usr/local/sqlite3.11`**
* **`make`**
* **`make install`**
* **`tree /usr/local/sqlite3.11/`**
* **`/usr/local/sqlite3.11/bin/sqlite3 -version`**
* **`/usr/local/sqlite3.11/bin/sqlite3 --help`**
* **`./sqlite3`**
* **`./sqlite3 test.db`**
* **`./sqlite3 test.db .dump > test.sql`**
* **`cat test.sql`**
* **`./sqlite3 test_tmp.db < test.sql`**


---


[sqlite]:http://www.sqlite.org/
[sqlite_doc]:http://www.sqlite.org/docs.html
[sqlite_dl]:http://www.sqlite.org/download.html
[sqlite_ref]:http://www.runoob.com/sqlite/sqlite-tutorial.html
