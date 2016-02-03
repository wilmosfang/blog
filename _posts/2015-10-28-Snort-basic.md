---
layout: post
title: Snort 基础
categories: linux snort security
wc: 1353 5732 62839
excerpt: follow me
comments: true
---


---

# 前言

**[Snort][snort]** 是一款开源的IDS/IPS(Intrusion Detection/Prevention System)软件

下面分享一下 **[Snort][snort]** 的基础操作，详细可以参阅 [官方文档][snortdoc] 和 [Snort 中文手册][doc.cn]

> **Tip:** 当前版本 **Snort 2.9.7.6**  另外 **[Snort 3.0][snort3]** 的测试版也出来了

---

# 概要

* TOC
{:toc}



---

## 安装


### 下载软件包

{% highlight bash %}
[root@h101 src]# wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz 
--2015-10-28 13:43:57--  https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
Resolving www.snort.org... 104.20.60.203, 104.20.59.203, 2400:cb00:2048:1::6814:3bcb, ...
Connecting to www.snort.org|104.20.60.203|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://s3.amazonaws.com/snort-org-site/production/release_files/files/000/002/545/original/daq-2.0.6.tar.gz?AWSAccessKeyId=AKIAIXACIED2SPMSC7GA&Expires=1446014637&Signature=hod%2BhGW%2BYX0PRS9aW2YWcfwV9eM%3D [following]
--2015-10-28 13:43:58--  https://s3.amazonaws.com/snort-org-site/production/release_files/files/000/002/545/original/daq-2.0.6.tar.gz?AWSAccessKeyId=AKIAIXACIED2SPMSC7GA&Expires=1446014637&Signature=hod%2BhGW%2BYX0PRS9aW2YWcfwV9eM%3D
Resolving s3.amazonaws.com... 54.231.9.88
Connecting to s3.amazonaws.com|54.231.9.88|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 514687 (503K) [binary/octet-stream]
Saving to: “daq-2.0.6.tar.gz”


100%[==========================================================================================>] 514,687     30.0K/s   in 66s     

2015-10-28 13:45:07 (7.58 KB/s) - “daq-2.0.6.tar.gz” saved [514687/514687]
 
[root@h101 src]# wget https://www.snort.org/downloads/snort/snort-2.9.7.6.tar.gz
--2015-10-28 13:56:37--  https://www.snort.org/downloads/snort/snort-2.9.7.6.tar.gz
Resolving www.snort.org... 104.20.59.203, 104.20.60.203, 2400:cb00:2048:1::6814:3bcb, ...
Connecting to www.snort.org|104.20.59.203|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://s3.amazonaws.com/snort-org-site/production/release_files/files/000/002/537/original/snort-2.9.7.6.tar.gz?AWSAccessKeyId=AKIAIXACIED2SPMSC7GA&Expires=1446015396&Signature=IyFcmAdHV%2B4A6MIgTis8nNtf2kA%3D [following]
--2015-10-28 13:56:38--  https://s3.amazonaws.com/snort-org-site/production/release_files/files/000/002/537/original/snort-2.9.7.6.tar.gz?AWSAccessKeyId=AKIAIXACIED2SPMSC7GA&Expires=1446015396&Signature=IyFcmAdHV%2B4A6MIgTis8nNtf2kA%3D
Resolving s3.amazonaws.com... 54.231.66.80
Connecting to s3.amazonaws.com|54.231.66.80|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6198052 (5.9M) [binary/octet-stream]
Saving to: “snort-2.9.7.6.tar.gz.1”

100%[==========================================================================================>] 6,198,052   6.11K/s   in 19m 18s   

2015-10-28 14:15:59 (5.22 KB/s) - “snort-2.9.7.6.tar.gz.1” saved [6198052/6198052]

[root@h101 src]# 
{% endhighlight %}

---

### 安装软件包


#### 安装daq-2.0.6

{% highlight bash %}
[root@h101 snort]# ll
total 6560
-rw-r--r-- 1 root root  514687 Oct 28 13:53 daq-2.0.6.tar.gz
-rw-r--r-- 1 root root 6198052 Oct 28 13:53 snort-2.9.7.6.tar.gz
[root@h101 snort]# tar -zxvf daq-2.0.6.tar.gz 
daq-2.0.6/
daq-2.0.6/ChangeLog
daq-2.0.6/missing
daq-2.0.6/daq.dsp
daq-2.0.6/configure
...
...
daq-2.0.6/m4/lt~obsolete.m4
daq-2.0.6/m4/ltoptions.m4
daq-2.0.6/configure.ac
[root@h101 snort]# ll 
total 6564
drwxr-xr-x 6 1000 1000    4096 Jul 17 05:06 daq-2.0.6
-rw-r--r-- 1 root root  514687 Oct 28 13:53 daq-2.0.6.tar.gz
-rw-r--r-- 1 root root 6198052 Oct 28 13:53 snort-2.9.7.6.tar.gz
[root@h101 snort]# 
{% endhighlight %}

##### 安装报错一

{% highlight bash %}
[root@h101 snort]# cd daq-2.0.6
[root@h101 daq-2.0.6]# ls
aclocal.m4  ChangeLog  config.guess  config.sub  configure.ac  daq.dsp  install-sh  m4           Makefile.in  os-daq-modules  sfbpf
api         compile    config.h.in   configure   COPYING       depcomp  ltmain.sh   Makefile.am  missing      README
[root@h101 daq-2.0.6]# ./configure 
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
...
...
checking for getaddrinfo... yes
checking for flex... flex
checking for flex 2.4 or higher... yes
checking for bison... no
configure: WARNING: don't have both flex and bison; reverting to lex/yacc
checking for capable lex... insufficient
configure: error: Your operating system's lex is insufficient to compile
         libsfbpf. You should install both bison and flex.
         flex is a lex replacement that has many advantages,
         including being able to compile libsfbpf.  For more
         information, see http://www.gnu.org/software/flex/flex.html .
[root@h101 daq-2.0.6]# echo $?
1
[root@h101 daq-2.0.6]#
{% endhighlight %}

错误原因是缺少 **bison** 和 **flex** ，不仅要安装它们的rpm包，还要安装开发包

{% highlight bash %}
yum install flex.x86_64  flex-devel.x86_64  bison.x86_64  bison-devel.x86_64 
{% endhighlight %}

解决办法

{% highlight bash %}
[root@h101 daq-2.0.6]# yum install flex-devel.x86_64  bison-devel.x86_64 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package bison-devel.x86_64 0:2.4.1-5.el6 will be installed
---> Package flex-devel.x86_64 0:2.5.35-9.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                           Arch                         Version                              Repository                  Size
======================================================================================================================================
Installing:
 bison-devel                       x86_64                       2.4.1-5.el6                          base                        21 k
 flex-devel                        x86_64                       2.5.35-9.el6                         base                        12 k

Transaction Summary
======================================================================================================================================
Install       2 Package(s)

Total download size: 33 k
Installed size: 43 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): bison-devel-2.4.1-5.el6.x86_64.rpm                                                                      |  21 kB     00:00     
(2/2): flex-devel-2.5.35-9.el6.x86_64.rpm                                                                      |  12 kB     00:00     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 201 kB/s |  33 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : bison-devel-2.4.1-5.el6.x86_64                                                                                     1/2 
  Installing : flex-devel-2.5.35-9.el6.x86_64                                                                                     2/2 
  Verifying  : flex-devel-2.5.35-9.el6.x86_64                                                                                     1/2 
  Verifying  : bison-devel-2.4.1-5.el6.x86_64                                                                                     2/2 

Installed:
  bison-devel.x86_64 0:2.4.1-5.el6                                  flex-devel.x86_64 0:2.5.35-9.el6                                 

Complete!
[root@h101 daq-2.0.6]# 
[root@h101 daq-2.0.6]# yum -y install  bison.x86_64 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package bison.x86_64 0:2.4.1-5.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                       Arch                           Version                              Repository                    Size
======================================================================================================================================
Installing:
 bison                         x86_64                         2.4.1-5.el6                          base                         637 k

Transaction Summary
======================================================================================================================================
Install       1 Package(s)

Total download size: 637 k
Installed size: 2.0 M
Downloading Packages:
bison-2.4.1-5.el6.x86_64.rpm                                                                                   | 637 kB     00:03     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : bison-2.4.1-5.el6.x86_64                                                                                           1/1 
  Verifying  : bison-2.4.1-5.el6.x86_64                                                                                           1/1 

Installed:
  bison.x86_64 0:2.4.1-5.el6                                                                                                          

Complete!
[root@h101 daq-2.0.6]# 
{% endhighlight %}

---

##### 安装报错二

{% highlight bash %}
[root@h101 daq-2.0.6]# ./configure
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
...
...
checking whether TPACKET2_HDRLEN is declared... yes
checking whether PACKET_TX_RING is declared... yes
checking pcap.h usability... no
checking pcap.h presence... no
checking for pcap.h... no
checking for pcap_lib_version in -lpcap... no
checking netinet/in.h usability... yes
checking netinet/in.h presence... yes
checking for netinet/in.h... yes
checking libipq.h usability... no
checking libipq.h presence... no
checking for libipq.h... no
checking for linux/netfilter.h... yes
checking for netinet/in.h... (cached) yes
checking libnetfilter_queue/libnetfilter_queue.h usability... no
checking libnetfilter_queue/libnetfilter_queue.h presence... no
checking for libnetfilter_queue/libnetfilter_queue.h... no
checking for linux/netfilter.h... (cached) yes
checking for pcap.h... (cached) no
checking for pcap_lib_version... checking for pcap_lib_version in -lpcap... (cached) no

    ERROR!  Libpcap library version >= 1.0.0 not found.
    Get it from http://www.tcpdump.org

[root@h101 daq-2.0.6]# echo $?
1
[root@h101 daq-2.0.6]# 
{% endhighlight %}

报错原因是有 **Libpcap** 的依赖关系


解决办法: 安装依赖包

{% highlight bash %}
[root@h101 daq-2.0.6]# yum list all | grep  -i  Libpcap 
libpcap.x86_64                              14:1.4.0-1.20130826git2dbcaa1.el6
libpcap.i686                                14:1.4.0-4.20130826git2dbcaa1.el6
libpcap.x86_64                              14:1.4.0-4.20130826git2dbcaa1.el6
libpcap-devel.i686                          14:1.4.0-4.20130826git2dbcaa1.el6
libpcap-devel.x86_64                        14:1.4.0-4.20130826git2dbcaa1.el6
[root@h101 daq-2.0.6]# yum install  libpcap.x86_64  libpcap-devel.x86_64 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package libpcap.x86_64 14:1.4.0-1.20130826git2dbcaa1.el6 will be updated
---> Package libpcap.x86_64 14:1.4.0-4.20130826git2dbcaa1.el6 will be an update
---> Package libpcap-devel.x86_64 14:1.4.0-4.20130826git2dbcaa1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                       Arch                   Version                                              Repository            Size
======================================================================================================================================
Installing:
 libpcap-devel                 x86_64                 14:1.4.0-4.20130826git2dbcaa1.el6                    base                 114 k
Updating:
 libpcap                       x86_64                 14:1.4.0-4.20130826git2dbcaa1.el6                    base                 131 k

Transaction Summary
======================================================================================================================================
Install       1 Package(s)
Upgrade       1 Package(s)

Total download size: 245 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): libpcap-1.4.0-4.20130826git2dbcaa1.el6.x86_64.rpm                                                       | 131 kB     00:01     
(2/2): libpcap-devel-1.4.0-4.20130826git2dbcaa1.el6.x86_64.rpm                                                 | 114 kB     00:00     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                  96 kB/s | 245 kB     00:02     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : 14:libpcap-1.4.0-4.20130826git2dbcaa1.el6.x86_64                                                                   1/3 
  Installing : 14:libpcap-devel-1.4.0-4.20130826git2dbcaa1.el6.x86_64                                                             2/3 
  Cleanup    : 14:libpcap-1.4.0-1.20130826git2dbcaa1.el6.x86_64                                                                   3/3 
  Verifying  : 14:libpcap-1.4.0-4.20130826git2dbcaa1.el6.x86_64                                                                   1/3 
  Verifying  : 14:libpcap-devel-1.4.0-4.20130826git2dbcaa1.el6.x86_64                                                             2/3 
  Verifying  : 14:libpcap-1.4.0-1.20130826git2dbcaa1.el6.x86_64                                                                   3/3 

Installed:
  libpcap-devel.x86_64 14:1.4.0-4.20130826git2dbcaa1.el6                                                                              

Updated:
  libpcap.x86_64 14:1.4.0-4.20130826git2dbcaa1.el6                                                                                    

Complete!
[root@h101 daq-2.0.6]#
{% endhighlight %}


再次配置，就成功了

{% highlight bash %}
[root@h101 daq-2.0.6]# ./configure
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
checking for visibility support... yes
checking CFLAGS for gcc -Wall... -Wall
checking CFLAGS for gcc -Wwrite-strings... -Wwrite-strings
checking CFLAGS for gcc -Wsign-compare... -Wsign-compare
checking CFLAGS for gcc -Wcast-align... -Wcast-align
checking CFLAGS for gcc -Wextra... -Wextra
checking CFLAGS for gcc -Wformat... -Wformat
checking CFLAGS for gcc -Wformat-security... -Wformat-security
checking CFLAGS for gcc -Wno-unused-parameter... -Wno-unused-parameter
checking CFLAGS for gcc -fno-strict-aliasing... -fno-strict-aliasing
checking CFLAGS for gcc -fdiagnostics-show-option... -fdiagnostics-show-option
checking CFLAGS for gcc -pedantic -std=c99 -D_GNU_SOURCE... -pedantic -std=c99 -D_GNU_SOURCE
checking for getaddrinfo... yes
checking for flex... flex
checking for flex 2.4 or higher... yes
checking for bison... bison
checking linux/if_ether.h usability... yes
checking linux/if_ether.h presence... yes
checking for linux/if_ether.h... yes
checking linux/if_packet.h usability... yes
checking linux/if_packet.h presence... yes
checking for linux/if_packet.h... yes
checking whether TPACKET2_HDRLEN is declared... yes
checking whether PACKET_TX_RING is declared... yes
checking pcap.h usability... yes
checking pcap.h presence... yes
checking for pcap.h... yes
checking for pcap_lib_version in -lpcap... yes
checking netinet/in.h usability... yes
checking netinet/in.h presence... yes
checking for netinet/in.h... yes
checking libipq.h usability... no
checking libipq.h presence... no
checking for libipq.h... no
checking for linux/netfilter.h... yes
checking for netinet/in.h... (cached) yes
checking libnetfilter_queue/libnetfilter_queue.h usability... no
checking libnetfilter_queue/libnetfilter_queue.h presence... no
checking for libnetfilter_queue/libnetfilter_queue.h... no
checking for linux/netfilter.h... (cached) yes
checking for pcap.h... (cached) yes
checking for pcap_lib_version... checking for pcap_lib_version in -lpcap... (cached) yes
checking for libpcap version >= "1.0.0"... yes
checking for net/netmap.h... no
checking for net/netmap_user.h... no
checking whether NETMAP_API is declared... no
checking for dlopen in -ldl... yes
checking for inttypes.h... (cached) yes
checking for memory.h... (cached) yes
checking netdb.h usability... yes
checking netdb.h presence... yes
checking for netdb.h... yes
checking for netinet/in.h... (cached) yes
checking for stdint.h... (cached) yes
checking for stdlib.h... (cached) yes
checking for string.h... (cached) yes
checking sys/ioctl.h usability... yes
checking sys/ioctl.h presence... yes
checking for sys/ioctl.h... yes
checking sys/param.h usability... yes
checking sys/param.h presence... yes
checking for sys/param.h... yes
checking sys/socket.h usability... yes
checking sys/socket.h presence... yes
checking for sys/socket.h... yes
checking sys/time.h usability... yes
checking sys/time.h presence... yes
checking for sys/time.h... yes
checking for unistd.h... (cached) yes
checking for inline... inline
checking for size_t... yes
checking for uint16_t... yes
checking for uint32_t... yes
checking for uint64_t... yes
checking for uint8_t... yes
checking for stdlib.h... (cached) yes
checking for GNU libc compatible malloc... yes
checking for stdlib.h... (cached) yes
checking for unistd.h... (cached) yes
checking for sys/param.h... (cached) yes
checking for getpagesize... yes
checking for working mmap... yes
checking for gethostbyname... yes
checking for getpagesize... (cached) yes
checking for memset... yes
checking for munmap... yes
checking for socket... yes
checking for strchr... yes
checking for strcspn... yes
checking for strdup... yes
checking for strerror... yes
checking for strrchr... yes
checking for strstr... yes
checking for strtoul... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating api/Makefile
config.status: creating os-daq-modules/Makefile
config.status: creating os-daq-modules/daq-modules-config
config.status: creating sfbpf/Makefile
config.status: creating config.h
config.status: executing depfiles commands
config.status: executing libtool commands

Build AFPacket DAQ module.. : yes
Build Dump DAQ module...... : yes
Build IPFW DAQ module...... : yes
Build IPQ DAQ module....... : no
Build NFQ DAQ module....... : no
Build PCAP DAQ module...... : yes
Build netmap DAQ module.... : no

[root@h101 daq-2.0.6]# echo $?
0
[root@h101 daq-2.0.6]#
{% endhighlight %}

然后编译和安装

{% highlight bash %}
[root@h101 daq-2.0.6]# make 
make  all-recursive
make[1]: Entering directory `/tmp/snort/daq-2.0.6'
Making all in api
make[2]: Entering directory `/tmp/snort/daq-2.0.6/api'
/bin/sh ../libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I. -I..     -g -O2 -fvisibility=hidden -Wall -Wwrite-strings -Wsign-compare -Wcast-align -Wextra -Wformat -Wformat-security -Wno-unused-parameter -fno-strict-aliasing -fdiagnostics-show-option -pedantic -std=c99 -D_GNU_SOURCE -MT daq_base.lo -MD -MP -MF .deps/daq_base.Tpo -c -o daq_base.lo daq_base.c
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I.. -g -O2 -fvisibility=hidden -Wall -Wwrite-strings -Wsign-compare -Wcast-align -Wextra -Wformat -Wformat-security -Wno-unused-parameter -fno-strict-aliasing -fdiagnostics-show-option -pedantic -std=c99 -D_GNU_SOURCE -MT daq_base.lo -MD -MP -MF .deps/daq_base.Tpo -c daq_base.c  -fPIC -DPIC -o .libs/daq_base.o
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I.. -g -O2 -fvisibility=hidden -Wall -Wwrite-strings -Wsign-compare -Wcast-align -Wextra -Wformat -Wformat-security -Wno-unused-parameter -fno-strict-aliasing -fdiagnostics-show-option -pedantic -std=c99 -D_GNU_SOURCE -MT daq_base.lo -MD -MP -MF .deps/daq_base.Tpo -c daq_base.c -o daq_base.o >/dev/null 2>&1
mv -f .deps/daq_base.Tpo .deps/daq_base.Plo
...
...

libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I.. -I../api -I../sfbpf -I../sfbpf -DBUILDING_SO -g -O2 -fvisibility=hidden -Wall -Wwrite-strings -Wsign-compare -Wcast-align -Wextra -Wformat -Wformat-security -Wno-unused-parameter -fno-strict-aliasing -fdiagnostics-show-option -pedantic -std=c99 -D_GNU_SOURCE -MT daq_ipfw_la-daq_ipfw.lo -MD -MP -MF .deps/daq_ipfw_la-daq_ipfw.Tpo -c daq_ipfw.c -o daq_ipfw_la-daq_ipfw.o >/dev/null 2>&1
mv -f .deps/daq_ipfw_la-daq_ipfw.Tpo .deps/daq_ipfw_la-daq_ipfw.Plo
/bin/sh ../libtool  --tag=CC   --mode=link gcc -DBUILDING_SO -g -O2 -fvisibility=hidden -Wall -Wwrite-strings -Wsign-compare -Wcast-align -Wextra -Wformat -Wformat-security -Wno-unused-parameter -fno-strict-aliasing -fdiagnostics-show-option -pedantic -std=c99 -D_GNU_SOURCE -module -export-dynamic -avoid-version -shared   -o daq_ipfw.la -rpath /usr/local/lib/daq daq_ipfw_la-daq_ipfw.lo ../sfbpf/libsfbpf.la 
libtool: link: gcc -shared  -fPIC -DPIC  .libs/daq_ipfw_la-daq_ipfw.o   -Wl,-rpath -Wl,/tmp/snort/daq-2.0.6/sfbpf/.libs -Wl,-rpath -Wl,/usr/local/lib ../sfbpf/.libs/libsfbpf.so  -g -O2   -Wl,-soname -Wl,daq_ipfw.so -o .libs/daq_ipfw.so
libtool: link: ( cd ".libs" && rm -f "daq_ipfw.la" && ln -s "../daq_ipfw.la" "daq_ipfw.la" )
make[2]: Leaving directory `/tmp/snort/daq-2.0.6/os-daq-modules'
make[2]: Entering directory `/tmp/snort/daq-2.0.6'
make[2]: Leaving directory `/tmp/snort/daq-2.0.6'
make[1]: Leaving directory `/tmp/snort/daq-2.0.6'
[root@h101 daq-2.0.6]# echo $?
0
[root@h101 daq-2.0.6]#
[root@h101 daq-2.0.6]# make install 
Making install in api
make[1]: Entering directory `/tmp/snort/daq-2.0.6/api'
make[2]: Entering directory `/tmp/snort/daq-2.0.6/api'
 /bin/mkdir -p '/usr/local/lib'
 /bin/sh ../libtool   --mode=install /usr/bin/install -c   libdaq.la libdaq_static.la '/usr/local/lib'
libtool: install: /usr/bin/install -c .libs/libdaq.so.2.0.4 /usr/local/lib/libdaq.so.2.0.4
libtool: install: (cd /usr/local/lib && { ln -s -f libdaq.so.2.0.4 libdaq.so.2 || { rm -f libdaq.so.2 && ln -s libdaq.so.2.0.4 libdaq.so.2; }; })
libtool: install: (cd /usr/local/lib && { ln -s -f libdaq.so.2.0.4 libdaq.so || { rm -f libdaq.so && ln -s libdaq.so.2.0.4 libdaq.so; }; })
libtool: install: /usr/bin/install -c .libs/libdaq.lai /usr/local/lib/libdaq.la
libtool: install: /usr/bin/install -c .libs/libdaq_static.lai /usr/local/lib/libdaq_static.la
libtool: install: /usr/bin/install -c .libs/libdaq.a /usr/local/lib/libdaq.a
libtool: install: chmod 644 /usr/local/lib/libdaq.a
libtool: install: ranlib /usr/local/lib/libdaq.a
libtool: install: /usr/bin/install -c .libs/libdaq_static.a /usr/local/lib/libdaq_static.a
libtool: install: chmod 644 /usr/local/lib/libdaq_static.a
libtool: install: ranlib /usr/local/lib/libdaq_static.a
libtool: finish: PATH="/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/sbin" ldconfig -n /usr/local/lib
...
...
make[2]: Nothing to be done for `install-data-am'.
make[2]: Leaving directory `/tmp/snort/daq-2.0.6/os-daq-modules'
make[1]: Leaving directory `/tmp/snort/daq-2.0.6/os-daq-modules'
make[1]: Entering directory `/tmp/snort/daq-2.0.6'
make[2]: Entering directory `/tmp/snort/daq-2.0.6'
make[2]: Nothing to be done for `install-exec-am'.
make[2]: Nothing to be done for `install-data-am'.
make[2]: Leaving directory `/tmp/snort/daq-2.0.6'
make[1]: Leaving directory `/tmp/snort/daq-2.0.6'
[root@h101 daq-2.0.6]# echo $?
0
[root@h101 daq-2.0.6]# 
{% endhighlight %}

---

#### 安装snort-2.9.7.6

{% highlight bash %}
[root@h101 snort]# tar -zxvf snort-2.9.7.6.tar.gz 
snort-2.9.7.6/
snort-2.9.7.6/depcomp
snort-2.9.7.6/tools/
snort-2.9.7.6/tools/u2streamer/
snort-2.9.7.6/tools/u2streamer/sf_error.h
snort-2.9.7.6/tools/u2streamer/sf_error.c
snort-2.9.7.6/tools/u2streamer/UnifiedLog.h
snort-2.9.7.6/tools/u2streamer/UnifiedLog.c
snort-2.9.7.6/tools/u2streamer/TimestampedFile.h
snort-2.9.7.6/tools/u2streamer/TimestampedFile.c
snort-2.9.7.6/tools/u2streamer/Unified2File.h
snort-2.9.7.6/tools/u2streamer/Unified2File.c
snort-2.9.7.6/tools/u2streamer/Unified2.h
snort-2.9.7.6/tools/u2streamer/Unified2.c
...
...
snort-2.9.7.6/COPYING
snort-2.9.7.6/snort.pc.in
snort-2.9.7.6/config.h.in
snort-2.9.7.6/aclocal.m4
snort-2.9.7.6/configure.in
snort-2.9.7.6/configure
snort-2.9.7.6/Makefile.am
snort-2.9.7.6/Makefile.in
[root@h101 snort]# ll 
total 6568
drwxr-xr-x  6 1000 1000    4096 Oct 28 14:07 daq-2.0.6
-rw-r--r--  1 root root  514687 Oct 28 13:53 daq-2.0.6.tar.gz
drwxr-xr-x 10 root root    4096 Aug 28 13:54 snort-2.9.7.6
-rw-r--r--  1 root root 6198052 Oct 28 13:53 snort-2.9.7.6.tar.gz
[root@h101 snort]# 
{% endhighlight %}

##### 安装报错一

{% highlight bash %}
[root@h101 snort-2.9.7.6]# ./configure  --enable-sourcefire 
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether to enable maintainer-specific portions of Makefiles... no
checking for style of include used by make... GNU
checking for gcc... gcc
...
...
checking for int64_t... yes
checking for boolean... no
checking for INADDR_NONE... yes
checking for __FUNCTION__... yes
checking for pcap_datalink in -lpcap... yes
checking for pcap_lex_destroy... yes
checking for pcap_lib_version... yes
./configure: line 15088: pcre-config: command not found
./configure: line 15094: pcre-config: command not found
checking pcre.h usability... no
checking pcre.h presence... no
checking for pcre.h... no

   ERROR!  Libpcre header not found.
   Get it from http://www.pcre.org
[root@h101 snort-2.9.7.6]# echo $?
1
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}

报错原因为 **pcre** 头文件缺失

解决方法 : 安装 **pcre.x86_64** 和 **pcre-devel.x86_64** 软件包

{% highlight bash %}
[root@h101 snort-2.9.7.6]# yum install  pcre.x86_64  pcre-devel.x86_64  
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package pcre.x86_64 0:7.8-6.el6 will be updated
---> Package pcre.x86_64 0:7.8-7.el6 will be an update
---> Package pcre-devel.x86_64 0:7.8-7.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                           Arch                          Version                            Repository                   Size
======================================================================================================================================
Installing:
 pcre-devel                        x86_64                        7.8-7.el6                          base                        320 k
Updating:
 pcre                              x86_64                        7.8-7.el6                          base                        196 k

Transaction Summary
======================================================================================================================================
Install       1 Package(s)
Upgrade       1 Package(s)

Total download size: 516 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): pcre-7.8-7.el6.x86_64.rpm                                                                               | 196 kB     00:02     
(2/2): pcre-devel-7.8-7.el6.x86_64.rpm                                                                         | 320 kB     00:01     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 138 kB/s | 516 kB     00:03     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : pcre-7.8-7.el6.x86_64                                                                                              1/3 
  Installing : pcre-devel-7.8-7.el6.x86_64                                                                                        2/3 
  Cleanup    : pcre-7.8-6.el6.x86_64                                                                                              3/3 
  Verifying  : pcre-7.8-7.el6.x86_64                                                                                              1/3 
  Verifying  : pcre-devel-7.8-7.el6.x86_64                                                                                        2/3 
  Verifying  : pcre-7.8-6.el6.x86_64                                                                                              3/3 

Installed:
  pcre-devel.x86_64 0:7.8-7.el6                                                                                                       

Updated:
  pcre.x86_64 0:7.8-7.el6                                                                                                             

Complete!
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}

---

##### 安装报错二

{% highlight bash %}
[root@h101 snort-2.9.7.6]# ./configure  --enable-sourcefire 
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether to enable maintainer-specific portions of Makefiles... no
checking for style of include used by make... GNU
checking for gcc... gcc
...
...
checking for SHA256_Init in -lcrypto... no
checking for MD5_Init in -lcrypto... no
checking dnet.h usability... no
checking dnet.h presence... no
checking for dnet.h... no
checking dumbnet.h usability... no
checking dumbnet.h presence... no
checking for dumbnet.h... no

   ERROR!  dnet header not found, go get it from
   http://code.google.com/p/libdnet/ or use the --with-dnet-*
   options, if you have it installed in an unusual place
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}

报错是因为 **libdnet** 头文件缺失

解决办法：安装 **libdnet.x86_64** 和 **libdnet-devel.x86_64**

{% highlight bash %}
[root@h101 snort-2.9.7.6]# yum install  libdnet.x86_64   libdnet-devel.x86_64 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package libdnet.x86_64 0:1.12-6.el6 will be installed
---> Package libdnet-devel.x86_64 0:1.12-6.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                             Arch                         Version                            Repository                  Size
======================================================================================================================================
Installing:
 libdnet                             x86_64                       1.12-6.el6                         epel                        28 k
 libdnet-devel                       x86_64                       1.12-6.el6                         epel                        28 k

Transaction Summary
======================================================================================================================================
Install       2 Package(s)

Total download size: 56 k
Installed size: 125 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): libdnet-1.12-6.el6.x86_64.rpm                                                                           |  28 kB     00:00     
(2/2): libdnet-devel-1.12-6.el6.x86_64.rpm                                                                     |  28 kB     00:00     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 140 kB/s |  56 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : libdnet-1.12-6.el6.x86_64                                                                                          1/2 
  Installing : libdnet-devel-1.12-6.el6.x86_64                                                                                    2/2 
  Verifying  : libdnet-1.12-6.el6.x86_64                                                                                          1/2 
  Verifying  : libdnet-devel-1.12-6.el6.x86_64                                                                                    2/2 

Installed:
  libdnet.x86_64 0:1.12-6.el6                                    libdnet-devel.x86_64 0:1.12-6.el6                                   

Complete!
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}


##### 安装报错三


{% highlight bash %}
[root@h101 snort-2.9.7.6]# ./configure  --enable-sourcefire 
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether to enable maintainer-specific portions of Makefiles... no
checking for style of include used by make... GNU
checking for gcc... gcc
...
...
checking for daq address space ID... no
checking for daq flow ID... no
checking for DAQ_VERDICT_RETRY... no
checking for sparc... no
checking for visibility support... yes
checking zlib.h usability... no
checking zlib.h presence... no
checking for zlib.h... no

   ERROR!  zlib header not found, go get it from
   http://www.zlib.net
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}

报错原因是 **zlib** 的头文件缺失

解决办法是： 安装 **zlib-devel.x86_64**

{% highlight bash %}
[root@h101 snort-2.9.7.6]# yum install  zlib.x86_64  zlib-devel.x86_64  
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * epel: mirrors.opencas.cn
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Package zlib-1.2.3-29.el6.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package zlib-devel.x86_64 0:1.2.3-29.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                          Arch                         Version                               Repository                  Size
======================================================================================================================================
Installing:
 zlib-devel                       x86_64                       1.2.3-29.el6                          base                        44 k

Transaction Summary
======================================================================================================================================
Install       1 Package(s)

Total download size: 44 k
Installed size: 115 k
Is this ok [y/N]: y
Downloading Packages:
zlib-devel-1.2.3-29.el6.x86_64.rpm                                                                             |  44 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : zlib-devel-1.2.3-29.el6.x86_64                                                                                     1/1 
  Verifying  : zlib-devel-1.2.3-29.el6.x86_64                                                                                     1/1 

Installed:
  zlib-devel.x86_64 0:1.2.3-29.el6                                                                                                    

Complete!
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}

再次配置，就成功

{% highlight bash %}
[root@h101 snort-2.9.7.6]# ./configure  --enable-sourcefire 
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether to enable maintainer-specific portions of Makefiles... no
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
checking for gcc option to accept ISO C99... -std=gnu99
checking for gcc -std=gnu99 option to accept ISO Standard C... (cached) -std=gnu99
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
checking whether the shell understands some XSI constructs... yes
checking whether the shell understands "+="... yes
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
checking for ranlib... (cached) ranlib
checking whether byte ordering is bigendian... no
checking for inline... inline
checking for stdbool.h that conforms to C99... yes
checking for _Bool... yes
checking for bison... bison
checking for flex... flex
checking for inttypes.h... (cached) yes
checking math.h usability... yes
checking math.h presence... yes
checking for math.h... yes
checking paths.h usability... yes
checking paths.h presence... yes
checking for paths.h... yes
checking for stdlib.h... (cached) yes
checking for string.h... (cached) yes
checking for strings.h... (cached) yes
checking for unistd.h... (cached) yes
checking wchar.h usability... yes
checking wchar.h presence... yes
checking for wchar.h... yes
checking sys/sockio.h usability... no
checking sys/sockio.h presence... no
checking for sys/sockio.h... no
checking for floor in -lm... yes
checking for ceil in -lm... yes
checking uuid/uuid.h usability... no
checking uuid/uuid.h presence... no
checking for uuid/uuid.h... no
checking for inet_ntoa in -lnsl... yes
checking for socket in -lsocket... no
checking whether printf must be declared... no
checking whether fprintf must be declared... no
checking whether syslog must be declared... no
checking whether puts must be declared... no
checking whether fputs must be declared... no
checking whether fputc must be declared... no
checking whether fopen must be declared... no
checking whether fclose must be declared... no
checking whether fwrite must be declared... no
checking whether fflush must be declared... no
checking whether getopt must be declared... no
checking whether bzero must be declared... no
checking whether bcopy must be declared... no
checking whether memset must be declared... no
checking whether strtol must be declared... no
checking whether strcasecmp must be declared... no
checking whether strncasecmp must be declared... no
checking whether strerror must be declared... no
checking whether perror must be declared... no
checking whether socket must be declared... no
checking whether sendto must be declared... no
checking whether vsnprintf must be declared... no
checking whether snprintf must be declared... no
checking whether strtoul must be declared... no
checking for sigaction... yes
checking for strlcpy... no
checking for strlcat... no
checking for strerror... yes
checking for vswprintf... yes
checking for wprintf... yes
checking for memrchr... yes
checking for inet_ntop... yes
checking for snprintf... yes
checking for malloc_trim... yes
checking for mallinfo... yes
checking size of char... 1
checking size of short... 2
checking size of int... 4
checking size of long int... 8
checking size of long long int... 8
checking size of unsigned int... 4
checking size of unsigned long int... 8
checking size of unsigned long long int... 8
checking for u_int8_t... yes
checking for u_int16_t... yes
checking for u_int32_t... yes
checking for u_int64_t... yes
checking for uint8_t... yes
checking for uint16_t... yes
checking for uint32_t... yes
checking for uint64_t... yes
checking for int8_t... yes
checking for int16_t... yes
checking for int32_t... yes
checking for int64_t... yes
checking for boolean... no
checking for INADDR_NONE... yes
checking for __FUNCTION__... yes
checking for pcap_datalink in -lpcap... yes
checking for pcap_lex_destroy... yes
checking for pcap_lib_version... yes
checking pcre.h usability... yes
checking pcre.h presence... yes
checking for pcre.h... yes
checking for pcre_compile in -lpcre... yes
checking for libpcre version 6.0 or greater... yes
checking for SHA256_Init in -lcrypto... no
checking for MD5_Init in -lcrypto... no
checking dnet.h usability... yes
checking dnet.h presence... yes
checking for dnet.h... yes
checking dumbnet.h usability... no
checking dumbnet.h presence... no
checking for dumbnet.h... no
checking for eth_set in -ldnet... yes
checking for eth_set in -ldumbnet... no
checking for dlsym in -ldl... yes
checking for daq_load_modules in -ldaq_static... yes
checking for daq_hup_apply... yes
checking for daq_acquire_with_meta... yes
checking for daq_dp_add_dc... yes
checking for struct _DAQ_DP_key_t.sa.src_ip4... yes
checking for daq address space ID... no
checking for daq flow ID... no
checking for DAQ_VERDICT_RETRY... no
checking for sparc... no
checking for visibility support... yes
checking zlib.h usability... yes
checking zlib.h presence... yes
checking for zlib.h... yes
checking for inflate in -lz... yes
checking lzma.h usability... no
checking lzma.h presence... no
checking for lzma.h... no
checking for lzma_stream_decoder in -llzma... no
checking for linuxthreads... no
checking for yylex_destroy support... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating snort.pc
config.status: creating Makefile
config.status: creating src/Makefile
config.status: creating src/sfutil/Makefile
config.status: creating src/control/Makefile
config.status: creating src/file-process/Makefile
config.status: creating src/file-process/libs/Makefile
config.status: creating src/side-channel/Makefile
config.status: creating src/side-channel/dynamic-plugins/Makefile
config.status: creating src/side-channel/dynamic-plugins/snort_side_channel.pc
config.status: creating src/side-channel/plugins/Makefile
config.status: creating src/detection-plugins/Makefile
config.status: creating src/dynamic-examples/Makefile
config.status: creating src/dynamic-examples/dynamic-preprocessor/Makefile
config.status: creating src/dynamic-examples/dynamic-rule/Makefile
config.status: creating src/dynamic-plugins/Makefile
config.status: creating src/dynamic-plugins/sf_engine/Makefile
config.status: creating src/dynamic-plugins/sf_engine/examples/Makefile
config.status: creating src/dynamic-plugins/sf_preproc_example/Makefile
config.status: creating src/dynamic-preprocessors/Makefile
config.status: creating src/dynamic-preprocessors/libs/Makefile
config.status: creating src/dynamic-preprocessors/libs/snort_preproc.pc
config.status: creating src/dynamic-preprocessors/ftptelnet/Makefile
config.status: creating src/dynamic-preprocessors/smtp/Makefile
config.status: creating src/dynamic-preprocessors/ssh/Makefile
config.status: creating src/dynamic-preprocessors/sip/Makefile
config.status: creating src/dynamic-preprocessors/reputation/Makefile
config.status: creating src/dynamic-preprocessors/gtp/Makefile
config.status: creating src/dynamic-preprocessors/dcerpc2/Makefile
config.status: creating src/dynamic-preprocessors/pop/Makefile
config.status: creating src/dynamic-preprocessors/imap/Makefile
config.status: creating src/dynamic-preprocessors/sdf/Makefile
config.status: creating src/dynamic-preprocessors/dns/Makefile
config.status: creating src/dynamic-preprocessors/ssl/Makefile
config.status: creating src/dynamic-preprocessors/modbus/Makefile
config.status: creating src/dynamic-preprocessors/dnp3/Makefile
config.status: creating src/dynamic-preprocessors/file/Makefile
config.status: creating src/dynamic-preprocessors/appid/Makefile
config.status: creating src/dynamic-output/Makefile
config.status: creating src/dynamic-output/plugins/Makefile
config.status: creating src/dynamic-output/libs/Makefile
config.status: creating src/dynamic-output/libs/snort_output.pc
config.status: creating src/output-plugins/Makefile
config.status: creating src/preprocessors/Makefile
config.status: creating src/preprocessors/HttpInspect/Makefile
config.status: creating src/preprocessors/HttpInspect/include/Makefile
config.status: creating src/preprocessors/HttpInspect/utils/Makefile
config.status: creating src/preprocessors/HttpInspect/anomaly_detection/Makefile
config.status: creating src/preprocessors/HttpInspect/client/Makefile
config.status: creating src/preprocessors/HttpInspect/files/Makefile
config.status: creating src/preprocessors/HttpInspect/event_output/Makefile
config.status: creating src/preprocessors/HttpInspect/mode_inspection/Makefile
config.status: creating src/preprocessors/HttpInspect/normalization/Makefile
config.status: creating src/preprocessors/HttpInspect/server/Makefile
config.status: creating src/preprocessors/HttpInspect/session_inspection/Makefile
config.status: creating src/preprocessors/HttpInspect/user_interface/Makefile
config.status: creating src/preprocessors/Session/Makefile
config.status: creating src/preprocessors/Stream6/Makefile
config.status: creating src/parser/Makefile
config.status: creating src/target-based/Makefile
config.status: creating doc/Makefile
config.status: creating rpm/Makefile
config.status: creating preproc_rules/Makefile
config.status: creating m4/Makefile
config.status: creating etc/Makefile
config.status: creating templates/Makefile
config.status: creating tools/Makefile
config.status: creating tools/control/Makefile
config.status: creating tools/u2boat/Makefile
config.status: creating tools/u2spewfoo/Makefile
config.status: creating tools/u2openappid/Makefile
config.status: creating tools/u2streamer/Makefile
config.status: creating tools/file_server/Makefile
config.status: creating src/win32/Makefile
config.status: creating config.h
config.status: executing depfiles commands
config.status: executing libtool commands
[root@h101 snort-2.9.7.6]# echo $?
0
[root@h101 snort-2.9.7.6]#
{% endhighlight %}

然后编译和安装

{% highlight bash %}
[root@h101 snort-2.9.7.6]# make 
make  all-recursive
make[1]: Entering directory `/tmp/snort/snort-2.9.7.6'
Making all in src
make[2]: Entering directory `/tmp/snort/snort-2.9.7.6/src'
Making all in sfutil
make[3]: Entering directory `/tmp/snort/snort-2.9.7.6/src/sfutil'
gcc -DHAVE_CONFIG_H -I. -I../.. -I../.. -I../../src -I../../src/sfutil -I/usr/include/pcap -I../../src/output-plugins -I../../src/detection-plugins -I../../src/dynamic-plugins -I../../src/preprocessors -I../../src/preprocessors/portscan -I../../src/preprocessors/HttpInspect/include -I../../src/preprocessors/Session -I../../src/preprocessors/Stream6 -I../../src/target-based -I../../src/control -I../../src/file-process -I../../src/file-process/libs -I../../src/side-channel -I../../src/side-channel/plugins  -DGRE -DMPLS -DPPM_MGR -DNDEBUG -DSOURCEFIRE -DPPM_MGR -DENABLE_REACT -DENABLE_RESPOND -DENABLE_RESPONSE3 -DSF_WCHAR -DTARGET_BASED -DPERF_PROFILING -DPERF_PROFILING -DSNORT_RELOAD -DNO_NON_ETHER_DECODER -DNORMALIZER -DACTIVE_RESPONSE  -g -O2 -DSF_VISIBILITY -fvisibility=hidden -fno-strict-aliasing -Wall -c -o sfghash.o sfghash.c
gcc -DHAVE_CONFIG_H -I. -I../.. -I../.. -I../../src -I../../src/sfutil -I/usr/include/pcap -I../../src/output-plugins -I../../src/detection-plugins -I../../src/dynamic-plugins -I../../src/preprocessors -I../../src/preprocessors/portscan -I../../src/preprocessors/HttpInspect/include -I../../src/preprocessors/Session -I../../src/preprocessors/Stream6 -I../../src/target-based -I../../src/control -I../../src/file-process -I../../src/file-process/libs -I../../src/side-channel -I../../src/side-channel/plugins  -DGRE -DMPLS -DPPM_MGR -DNDEBUG -DSOURCEFIRE -DPPM_MGR -DENABLE_REACT -DENABLE_RESPOND -DENABLE_RESPONSE3 -DSF_WCHAR -DTARGET_BASED -DPERF_PROFILING -DPERF_PROFILING -DSNORT_RELOAD -DNO_NON_ETHER_DECODER -DNORMALIZER -DACTIVE_RESPONSE  -g -O2 -DSF_VISIBILITY -fvisibility=hidden -fno-strict-aliasing -Wall -c -o sfhashfcn.o sfhashfcn.c
...
...
/bin/sh ../../libtool  --tag=CC   --mode=link gcc -g -O2 -DSF_VISIBILITY -fvisibility=hidden -fno-strict-aliasing -Wall  -g -O2 -DSF_VISIBILITY -fvisibility=hidden -fno-strict-aliasing -Wall  -lpcre -L/usr/lib64 -ldnet -o u2spewfoo u2spewfoo-u2spewfoo.o  -lz -ldaq_static -ldnet -lpcre -lpcap -lnsl -lm -lm  -ldl -L/usr/local/lib -ldaq_static_modules  -lsfbpf -lpcap -lsfbpf -lpcap -lz -lpthread -lpthread -lpthread
libtool: link: gcc -g -O2 -DSF_VISIBILITY -fvisibility=hidden -fno-strict-aliasing -Wall -g -O2 -DSF_VISIBILITY -fvisibility=hidden -fno-strict-aliasing -Wall -o u2spewfoo u2spewfoo-u2spewfoo.o  -L/usr/lib64 /usr/local/lib/libdaq_static.a -ldnet -lpcre -lnsl -lm -ldl -L/usr/local/lib /usr/local/lib/libdaq_static_modules.a /usr/local/lib/libsfbpf.so -lpcap -lz -lpthread -Wl,-rpath -Wl,/usr/local/lib -Wl,-rpath -Wl,/usr/local/lib
make[3]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools/u2spewfoo'
make[3]: Entering directory `/tmp/snort/snort-2.9.7.6/tools'
make[3]: Nothing to be done for `all-am'.
make[3]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools'
make[2]: Entering directory `/tmp/snort/snort-2.9.7.6'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6'
make[1]: Leaving directory `/tmp/snort/snort-2.9.7.6'
[root@h101 snort-2.9.7.6]# echo $?
0
[root@h101 snort-2.9.7.6]# 
[root@h101 snort-2.9.7.6]# make install 
Making install in src
make[1]: Entering directory `/tmp/snort/snort-2.9.7.6/src'
Making install in sfutil
make[2]: Entering directory `/tmp/snort/snort-2.9.7.6/src/sfutil'
make[3]: Entering directory `/tmp/snort/snort-2.9.7.6/src/sfutil'
make[3]: Nothing to be done for `install-exec-am'.
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/tmp/snort/snort-2.9.7.6/src/sfutil'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6/src/sfutil'
Making install in win32
make[2]: Entering directory `/tmp/snort/snort-2.9.7.6/src/win32'
make[3]: Entering directory `/tmp/snort/snort-2.9.7.6/src/win32'
make[3]: Nothing to be done for `install-exec-am'.
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/tmp/snort/snort-2.9.7.6/src/win32'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6/src/win32'
Making install in output-plugins
...
...
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools/u2spewfoo'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools/u2spewfoo'
make[2]: Entering directory `/tmp/snort/snort-2.9.7.6/tools'
make[3]: Entering directory `/tmp/snort/snort-2.9.7.6/tools'
make[3]: Nothing to be done for `install-exec-am'.
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools'
make[1]: Leaving directory `/tmp/snort/snort-2.9.7.6/tools'
make[1]: Entering directory `/tmp/snort/snort-2.9.7.6'
make[2]: Entering directory `/tmp/snort/snort-2.9.7.6'
make[2]: Nothing to be done for `install-exec-am'.
 /bin/mkdir -p '/usr/local/share/man/man8'
 /usr/bin/install -c -m 644 snort.8 '/usr/local/share/man/man8'
 /bin/mkdir -p '/usr/local/lib/pkgconfig'
 /usr/bin/install -c -m 644 snort.pc '/usr/local/lib/pkgconfig'
make[2]: Leaving directory `/tmp/snort/snort-2.9.7.6'
make[1]: Leaving directory `/tmp/snort/snort-2.9.7.6'
[root@h101 snort-2.9.7.6]# echo $?
0
[root@h101 snort-2.9.7.6]# 
{% endhighlight %}

#### 包依赖总结


snort-2.9.7.6 依赖以下安装包

{% highlight bash %}
pcre.x86_64  pcre-devel.x86_64   libdnet.x86_64   libdnet-devel.x86_64   zlib.x86_64  zlib-devel.x86_64 daq-2.0.6
{% endhighlight %}


daq-2.0.6 依赖以下安装包

{% highlight bash %}
flex.x86_64  flex-devel.x86_64  bison.x86_64  bison-devel.x86_64  libpcap.x86_64  libpcap-devel.x86_64
{% endhighlight %}

##### 汇总解决依赖

{% highlight bash %}
yum install flex.x86_64  flex-devel.x86_64  bison.x86_64  bison-devel.x86_64  libpcap.x86_64  libpcap-devel.x86_64  pcre.x86_64  pcre-devel.x86_64   libdnet.x86_64   libdnet-devel.x86_64   zlib.x86_64  zlib-devel.x86_64 
{% endhighlight %}

> **Tip:** Make sure the following packages are installed in your CentOS 6.x system via System Administration  Add/Remove Software (requires ‘**root**’ privileges): **gcc** version (4.4.6 including libraries), **flex** (2.5.35), **bison** (2.4.1), **zlib** (1.2.3 including **zlib-devel** ), **libpcap** (1.0.0 including **libpcap-devel** ), **pcre** (7.84 including **pcre-devel** ), **libdnet** (1.11 or 1.12 including **libdnet-devel** ) and **tcpdump** (4.1.0). Versions of these packages already installed may be newer than what is listed here, but should NOT cause any issues when compiling DAQ and/or SNORT.

下载源码包

{% highlight bash %}
wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
wget https://www.snort.org/downloads/snort/snort-2.9.7.6.tar.gz
{% endhighlight %}

安装daq-2.0.6

{% highlight bash %}
tar xvfz daq-2.0.6.tar.gz
cd daq-2.0.6
./configure; make; make install
{% endhighlight %}

安装snort-2.9.7.6

{% highlight bash %}
tar xvfz snort-2.9.7.6.tar.gz
cd snort-2.9.7.6
./configure --enable-sourcefire; make; make install
{% endhighlight %}

> **Tip:** 可以使用 **echo $?** 来检验上一步是否成功返回






---

[snort]:https://www.snort.org/
[snortdoc]:http://manual.snort.org/
[snort3]:https://www.snort.org/snort3
[doc.cn]:http://man.chinaunix.net/network/snort/Snortman.htm



