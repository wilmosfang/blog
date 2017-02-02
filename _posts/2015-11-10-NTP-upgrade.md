---
layout: post
title: NTP升级
author: wilmosfang
tags:  upgrade ntp
categories:  ntp
wc: 664 3060 25495
excerpt: follow me
comments: true
---


---

# 前言

**[NTP][ntp]** (Network Time Protocol) 是网络系统中用来协同时间的关键服务，它的作用就是用来同步时间

前不久网络中暴出NTP的漏洞可以被利用，详情可以参看 **[针对网络时间协议（NTP）的攻击将会引起巨大的麻烦][ntp_bug2]** (**[原文][ntp_bug]**) 还有 **[官方申明][ntp_notice]**

这里描述如何对ntp进行升级，详细可以参考 **[官方文档][ntp_doc]**


 > **Tip:** 当前版本 **NTP 4.2.8p4**
 
---

# 概要

* TOC
{:toc}


---


## 下载

**[下载地址][ntp_download]**

~~~
[root@h101 tmp]# wget  http://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz 
--2015-11-09 17:40:04--  http://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz
Resolving www.eecis.udel.edu... 128.4.31.8
Connecting to www.eecis.udel.edu|128.4.31.8|:80... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz [following]
--2015-11-09 17:40:05--  https://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz
Connecting to www.eecis.udel.edu|128.4.31.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7104852 (6.8M) [application/x-gzip]
Saving to: “ntp-4.2.8p4.tar.gz”

100%[============================================================================================>] 7,104,852    244K/s   in 31s     

2015-11-09 17:40:38 (227 KB/s) - “ntp-4.2.8p4.tar.gz” saved [7104852/7104852]

[root@h101 tmp]# 
~~~

对下载包进行校验


~~~
[root@h101 tmp]# wget http://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz.md5 
--2015-11-09 17:41:56--  http://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz.md5
Resolving www.eecis.udel.edu... 128.4.31.8
Connecting to www.eecis.udel.edu|128.4.31.8|:80... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz.md5 [following]
--2015-11-09 17:41:57--  https://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p4.tar.gz.md5
Connecting to www.eecis.udel.edu|128.4.31.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 53 [application/x-gzip]
Saving to: “ntp-4.2.8p4.tar.gz.md5”

100%[============================================================================================>] 53          --.-K/s   in 0s      

2015-11-09 17:41:59 (4.39 MB/s) - “ntp-4.2.8p4.tar.gz.md5” saved [53/53]

[root@h101 tmp]# file ntp-4.2.8p4.tar.gz.md5 
ntp-4.2.8p4.tar.gz.md5: ASCII text
[root@h101 tmp]# cat ntp-4.2.8p4.tar.gz.md5 
6af96862b09324a8ef965ca76b759c8b  ntp-4.2.8p4.tar.gz
[root@h101 tmp]# md5sum   ntp-4.2.8p4.tar.gz   
6af96862b09324a8ef965ca76b759c8b  ntp-4.2.8p4.tar.gz
[root@h101 tmp]#
~~~


---

## 安装

详细的安装方法可以参考 [安装NTP][ntp_build]

~~~
./configure  
echo $?
make 
echo $?
make install 
echo $?
~~~

> **Tip:** **echo $?** 用来确认上一步是否成功执行


注意：**make install** 可能会覆盖一些文件，所以这一步要非常小心，默认会生成在 **/usr/local/bin** 中

~~~
Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]

By default, `make install' will install all the files in
`/usr/local/bin', `/usr/local/lib' etc.  You can specify
an installation prefix other than `/usr/local' using `--prefix',
for instance `--prefix=$HOME'.
~~~

>It is not possible in a software distribution such as this to support every individual computer and operating system with a common executable, even with the same system but different versions and options. Therefore, it is necessary to configure, build and install for each system and version. In almost all cases, these procedures are completely automatic, The user types ./configure, make and install in that order and the autoconfigure system does the rest. 

总之，安装成功后会多出以下文件 

~~~
[root@h101 ntp-4.2.8p4]# ll  /usr/local/bin/ntp*
-rwxr-xr-x 1 root root 2630962 Nov  9 19:53 /usr/local/bin/ntpd
-rwxr-xr-x 1 root root  413656 Nov  9 19:53 /usr/local/bin/ntpdate
-rwxr-xr-x 1 root root  859156 Nov  9 19:53 /usr/local/bin/ntpdc
-rwxr-xr-x 1 root root  632477 Nov  9 19:53 /usr/local/bin/ntp-keygen
-rwxr-xr-x 1 root root  911897 Nov  9 19:53 /usr/local/bin/ntpq
-rwxr-xr-x 1 root root  228107 Nov  9 19:53 /usr/local/bin/ntptime
-rwxr-xr-x 1 root root    3567 Nov  9 19:53 /usr/local/bin/ntptrace
-rwxr-xr-x 1 root root    3207 Nov  9 19:53 /usr/local/bin/ntp-wait
[root@h101 ntp-4.2.8p4]# 
~~~

检查发现，已经是最新的版本了

~~~
[root@h101 ntp-4.2.8p4]# /usr/local/bin/ntpd --help 
ntpd - NTP daemon program - Ver. 4.2.8p4
Usage:  ntpd [ -<flag> [<val>] | --<name>[{=| }<val>] ]... \
		[ <server1> ... <serverN> ]
  Flg Arg Option-Name    Description
   -4 no  ipv4           Force IPv4 DNS name resolution
				- prohibits the option 'ipv6'
   -6 no  ipv6           Force IPv6 DNS name resolution
				- prohibits the option 'ipv4'
   -a no  authreq        Require crypto authentication
				- prohibits the option 'authnoreq'
   -A no  authnoreq      Do not require crypto authentication
				- prohibits the option 'authreq'
   -b no  bcastsync      Allow us to sync to broadcast servers
   -c Str configfile     configuration file name
   -d no  debug-level    Increase debug verbosity level
				- may appear multiple times
   -D Num set-debug-level Set the debug verbosity level
				- may appear multiple times
   -f Str driftfile      frequency drift file name
   -g no  panicgate      Allow the first adjustment to be Big
				- may appear multiple times
   -G no  force-step-once Step any initial offset correction.
   -i --- jaildir        built without --enable-clockctl or --enable-linuxcaps or --enable-solarisprivs
   -I Str interface      Listen on an interface name or address
				- may appear multiple times
   -k Str keyfile        path to symmetric keys
   -l Str logfile        path to the log file
   -L no  novirtualips   Do not listen to virtual interfaces
   -n no  nofork         Do not fork
				- prohibits the option 'wait-sync'
   -N no  nice           Run at high priority
   -p Str pidfile        path to the PID file
   -P Num priority       Process priority
   -q no  quit           Set the time and quit
				- prohibits these options:
				saveconfigquit
				wait-sync
   -r Str propagationdelay Broadcast/propagation delay
      Str saveconfigquit Save parsed configuration and quit
				- prohibits these options:
				quit
				wait-sync
   -s Str statsdir       Statistics file location
   -t Str trustedkey     Trusted key number
				- may appear multiple times
   -u --- user           built without --enable-clockctl or --enable-linuxcaps or --enable-solarisprivs
   -U Num updateinterval interval in seconds between scans for new or dropped interfaces
      Str var            make ARG an ntp variable (RW)
				- may appear multiple times
      Str dvar           make ARG an ntp variable (RW|DEF)
				- may appear multiple times
   -w Num wait-sync      Seconds to wait for first clock sync
				- prohibits these options:
				nofork
				quit
				saveconfigquit
   -x no  slew           Slew up to 600 seconds
      opt version        output version information and exit
   -? no  help           display extended usage information and exit
   -! no  more-help      extended usage information passed thru pager

Options are specified by doubled hyphens and their name or by a single
hyphen and the flag character.


The following option preset mechanisms are supported:
 - examining environment variables named NTPD_*

Please send bug reports to:  <http://bugs.ntp.org, bugs@ntp.org>
[root@h101 ntp-4.2.8p4]# 
~~~

---

## 升级

当前版还是 **Ver. 4.2.6p5**

~~~
[root@h101 etc]# /usr/sbin/ntpd --help 
ntpd - NTP daemon program - Ver. 4.2.6p5
USAGE:  ntpd [ -<flag> [<val>] | --<name>[{=| }<val>] ]...
  Flg Arg Option-Name    Description
   -4     ipv4           Force IPv4 DNS name resolution
	- prohibits these options:
	ipv6
   -6     ipv6           Force IPv6 DNS name resolution
	- prohibits these options:
	ipv4
   -a     authreq        Require crypto authentication
	- prohibits these options:
	authnoreq
   -A     authnoreq      Do not require crypto authentication
	- prohibits these options:
	authreq
   -b     bcastsync      Allow us to sync to broadcast servers
   -c     configfile     configuration file name
   -d     debug-level    Increase output debug message level
				- may appear multiple times
   -D     set-debug-level Set the output debug message level
				- may appear multiple times
   -f     driftfile      frequency drift file name
   -g     panicgate      Allow the first adjustment to be Big
				- may appear multiple times
   -i     jaildir        Jail directory
   -I     interface      Listen on an interface name or address
				- may appear multiple times
   -k     keyfile        path to symmetric keys
   -l     logfile        path to the log file
   -L     novirtualips   Do not listen to virtual interfaces
   -n     nofork         Do not fork
   -N     nice           Run at high priority
   -p     pidfile        path to the PID file
   -P     priority       Process priority
   -q     quit           Set the time and quit
   -r     propagationdelay Broadcast/propagation delay
          saveconfigquit Save parsed configuration and quit
   -s     statsdir       Statistics file location
   -t     trustedkey     Trusted key number
				- may appear multiple times
   -u     user           Run as userid (or userid:groupid)
   -U     updateinterval interval in seconds between scans for new or dropped interfaces
          var            make ARG an ntp variable (RW)
				- may appear multiple times
          dvar           make ARG an ntp variable (RW|DEF)
				- may appear multiple times
   -x     slew           Slew up to 600 seconds
   -m     mlock          Lock memory
   -!     version        Output version information and exit
   -?     help           Display extended usage information and exit
   -!     more-help      Extended usage information passed thru pager

Options are specified by doubled hyphens and their name or by a single
hyphen and the flag character.

The following option preset mechanisms are supported:
 - examining environment variables named NTPD_*



please send bug reports to:  http://bugs.ntp.org, bugs@ntp.org
[root@h101 etc]#
~~~

检查当前运行状态

~~~
[root@h101 etc]# ps faux | grep ntp 
root     57457  0.5  0.0 107668  1436 pts/0    S+   20:42   0:01  |       \_ watch -n .5 ntpstat
root     57453  0.0  0.0 107668  1436 pts/1    S+   20:42   0:00  |       \_ watch -n 4 ntpq -p
root     58106  0.0  0.0 103256   828 pts/2    S+   20:47   0:00  |       \_ grep ntp
ntp      57448  0.2  0.0  30736  2104 ?        Ss   20:41   0:00 ntpd -u ntp:ntp -p /var/run/ntpd.pid -g
[root@h101 etc]# 
~~~

停止NTP服务

~~~
[root@h101 etc]# /etc/init.d/ntpd stop 
Shutting down ntpd:                                        [  OK  ]
[root@h101 etc]#
~~~

备份原NTP server程序

~~~
[root@h101 sbin]# mv ntpd   ntpd.old.4.2.6p5
~~~

更替成新的NTP server

~~~
[root@h101 sbin]# cp  /usr/local/bin/ntpd  .
[root@h101 sbin]# pwd
/usr/sbin
[root@h101 sbin]# ll ntp*
-rwxr-xr-x 1 root root 2630962 Nov  9 21:22 ntpd
-rwxr-xr-x 1 root root  110896 Oct 26 23:32 ntpdate
-rwxr-xr-x 1 root root  250312 Oct 26 23:32 ntpdc
-rwxr-xr-x 1 root root  759720 Oct 26 23:32 ntpd.old.4.2.6p5
-rwxr-xr-x 1 root root  184592 Oct 26 23:32 ntp-keygen
-rwxr-xr-x 1 root root  252424 Oct 26 23:32 ntpq
-rwxr-xr-x 1 root root   72848 Oct 26 23:32 ntptime
[root@h101 sbin]# 
~~~

尝试启动服务

~~~
[root@h101 sbin]# /etc/init.d/ntpd status
ntpd is stopped
[root@h101 sbin]# /etc/init.d/ntpd start 
Starting ntpd: /usr/sbin/ntpd: The 'user' option has been disabled. -- built without --enable-clockctl or --enable-linuxcaps or --enable-solarisprivs
ntpd - NTP daemon program - Ver. 4.2.8p4
Usage:  ntpd [ -<flag> [<val>] | --<name>[{=| }<val>] ]... \
		[ <server1> ... <serverN> ]
Try 'ntpd --help' for more information.
                                                           [FAILED]
[root@h101 sbin]#
~~~

---

### 报错

原因是配置过程中少加了一些权限特性

---

#### 解决方法一

去掉用户和组的参数，再次启动

~~~
[root@h101 sbin]# vim /etc/sysconfig/ntpd
[root@h101 sbin]# cat /etc/sysconfig/ntpd
# Drop root to id 'ntp:ntp' by default.
#OPTIONS="-u ntp:ntp -p /var/run/ntpd.pid -g"
OPTIONS=" -p /var/run/ntpd.pid -g"
[root@h101 sbin]# 
[root@h101 sbin]# /etc/init.d/ntpd start 
Starting ntpd:                                             [  OK  ]
[root@h101 sbin]# /usr/sbin/ntpd --help 
ntpd - NTP daemon program - Ver. 4.2.8p4
Usage:  ntpd [ -<flag> [<val>] | --<name>[{=| }<val>] ]... \
		[ <server1> ... <serverN> ]
  Flg Arg Option-Name    Description
   -4 no  ipv4           Force IPv4 DNS name resolution
				- prohibits the option 'ipv6'
   -6 no  ipv6           Force IPv6 DNS name resolution
				- prohibits the option 'ipv4'
   -a no  authreq        Require crypto authentication
				- prohibits the option 'authnoreq'
   -A no  authnoreq      Do not require crypto authentication
				- prohibits the option 'authreq'
   -b no  bcastsync      Allow us to sync to broadcast servers
   -c Str configfile     configuration file name
   -d no  debug-level    Increase debug verbosity level
				- may appear multiple times
   -D Num set-debug-level Set the debug verbosity level
				- may appear multiple times
   -f Str driftfile      frequency drift file name
   -g no  panicgate      Allow the first adjustment to be Big
				- may appear multiple times
   -G no  force-step-once Step any initial offset correction.
   -i --- jaildir        built without --enable-clockctl or --enable-linuxcaps or --enable-solarisprivs
   -I Str interface      Listen on an interface name or address
				- may appear multiple times
   -k Str keyfile        path to symmetric keys
   -l Str logfile        path to the log file
   -L no  novirtualips   Do not listen to virtual interfaces
   -n no  nofork         Do not fork
				- prohibits the option 'wait-sync'
   -N no  nice           Run at high priority
   -p Str pidfile        path to the PID file
   -P Num priority       Process priority
   -q no  quit           Set the time and quit
				- prohibits these options:
				saveconfigquit
				wait-sync
   -r Str propagationdelay Broadcast/propagation delay
      Str saveconfigquit Save parsed configuration and quit
				- prohibits these options:
				quit
				wait-sync
   -s Str statsdir       Statistics file location
   -t Str trustedkey     Trusted key number
				- may appear multiple times
   -u --- user           built without --enable-clockctl or --enable-linuxcaps or --enable-solarisprivs
   -U Num updateinterval interval in seconds between scans for new or dropped interfaces
      Str var            make ARG an ntp variable (RW)
				- may appear multiple times
      Str dvar           make ARG an ntp variable (RW|DEF)
				- may appear multiple times
   -w Num wait-sync      Seconds to wait for first clock sync
				- prohibits these options:
				nofork
				quit
				saveconfigquit
   -x no  slew           Slew up to 600 seconds
      opt version        output version information and exit
   -? no  help           display extended usage information and exit
   -! no  more-help      extended usage information passed thru pager

Options are specified by doubled hyphens and their name or by a single
hyphen and the flag character.


The following option preset mechanisms are supported:
 - examining environment variables named NTPD_*

Please send bug reports to:  <http://bugs.ntp.org, bugs@ntp.org>
[root@h101 sbin]# 
[root@h101 sbin]# ps faux | grep ntp 
root     57457  0.5  0.0 107668  1436 pts/0    S+   20:42   0:03  |       \_ watch -n .5 ntpstat
root     57453  0.0  0.0 107672  1436 pts/1    S+   20:42   0:00  |       \_ watch -n 4 ntpq -p
root     58752 24.0  0.0  44148  2244 pts/1    R+   20:51   0:00  |           \_ ntpq -p
root     58755  0.0  0.0 103256   828 pts/2    S+   20:51   0:00  |       \_ grep ntp
root     58705  0.2  0.0 122808  2296 ?        Ssl  20:51   0:00 ntpd -p /var/run/ntpd.pid -g
[root@h101 sbin]# ntpq -p 
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 202.120.2.100 ( .INIT.          16 u    -   64    0    0.000    0.000   0.000
 s2a.time.edu.cn .INIT.          16 u    -   64    0    0.000    0.000   0.000
 202.112.0.7     .INIT.          16 u    -   64    0    0.000    0.000   0.000
 gus.buptnet.edu .INIT.          16 u    -   64    0    0.000    0.000   0.000
 mailgw.xanet.ed .INIT.          16 u    -   64    0    0.000    0.000   0.000
 dns1.synet.edu. .INIT.          16 u    -   64    0    0.000    0.000   0.000
*202.112.26.37   202.112.29.82    3 u   28   64    1    5.193  -33.794   0.000
 ntp.glnet.edu.c .INIT.          16 u    -   64    0    0.000    0.000   0.000
 ns.pku.edu.cn   129.6.15.28      2 u   27   64    1  102.864   23.455   0.000
 news.neu.edu.cn .INIT.          16 u    -   64    0    0.000    0.000   0.000
[root@h101 sbin]# 
[root@h101 tmp]# ntpstat  
synchronised to NTP server (202.118.1.130) at stratum 3 
   time correct to within 268 ms
   polling server every 64 s
[root@h101 tmp]# 
~~~

成功升级，只是运行用户和组是root，可能会有安全隐患


---

#### 解决方法二


重编译，并且配置时加入参数 **`--enable-clockctl`**

~~~
[root@h101 ntp-4.2.8p4]# ./configure    --enable-clockctl 
...
...
[root@h101 ntp-4.2.8p4]# echo $?
0
[root@h101 ntp-4.2.8p4]# make 
...
...
[root@h101 ntp-4.2.8p4]# echo $?
0
[root@h101 ntp-4.2.8p4]# make install
[root@h101 ntp-4.2.8p4]# echo $?
0
[root@h101 ntp-4.2.8p4]# 
[root@h101 ntp-4.2.8p4]# ll /usr/local/bin/ntp*
-rwxr-xr-x 1 root root 2633989 Nov 10 10:01 /usr/local/bin/ntpd
-rwxr-xr-x 1 root root  413656 Nov 10 10:01 /usr/local/bin/ntpdate
-rwxr-xr-x 1 root root  859156 Nov 10 10:01 /usr/local/bin/ntpdc
-rwxr-xr-x 1 root root  632477 Nov 10 10:01 /usr/local/bin/ntp-keygen
-rwxr-xr-x 1 root root  911897 Nov 10 10:01 /usr/local/bin/ntpq
-rwxr-xr-x 1 root root  228107 Nov 10 10:01 /usr/local/bin/ntptime
-rwxr-xr-x 1 root root    3567 Nov 10 10:00 /usr/local/bin/ntptrace
-rwxr-xr-x 1 root root    3207 Nov 10 10:00 /usr/local/bin/ntp-wait
[root@h101 ntp-4.2.8p4]# /usr/local/bin/ntpd --help
ntpd - NTP daemon program - Ver. 4.2.8p4
Usage:  ntpd [ -<flag> [<val>] | --<name>[{=| }<val>] ]... \
		[ <server1> ... <serverN> ]
  Flg Arg Option-Name    Description
   -4 no  ipv4           Force IPv4 DNS name resolution
				- prohibits the option 'ipv6'
   -6 no  ipv6           Force IPv6 DNS name resolution
				- prohibits the option 'ipv4'
   -a no  authreq        Require crypto authentication
				- prohibits the option 'authnoreq'
   -A no  authnoreq      Do not require crypto authentication
				- prohibits the option 'authreq'
   -b no  bcastsync      Allow us to sync to broadcast servers
   -c Str configfile     configuration file name
   -d no  debug-level    Increase debug verbosity level
				- may appear multiple times
   -D Num set-debug-level Set the debug verbosity level
				- may appear multiple times
   -f Str driftfile      frequency drift file name
   -g no  panicgate      Allow the first adjustment to be Big
				- may appear multiple times
   -G no  force-step-once Step any initial offset correction.
   -i Str jaildir        Jail directory
   -I Str interface      Listen on an interface name or address
				- may appear multiple times
   -k Str keyfile        path to symmetric keys
   -l Str logfile        path to the log file
   -L no  novirtualips   Do not listen to virtual interfaces
   -n no  nofork         Do not fork
				- prohibits the option 'wait-sync'
   -N no  nice           Run at high priority
   -p Str pidfile        path to the PID file
   -P Num priority       Process priority
   -q no  quit           Set the time and quit
				- prohibits these options:
				saveconfigquit
				wait-sync
   -r Str propagationdelay Broadcast/propagation delay
      Str saveconfigquit Save parsed configuration and quit
				- prohibits these options:
				quit
				wait-sync
   -s Str statsdir       Statistics file location
   -t Str trustedkey     Trusted key number
				- may appear multiple times
   -u Str user           Run as userid (or userid:groupid)
   -U Num updateinterval interval in seconds between scans for new or dropped interfaces
      Str var            make ARG an ntp variable (RW)
				- may appear multiple times
      Str dvar           make ARG an ntp variable (RW|DEF)
				- may appear multiple times
   -w Num wait-sync      Seconds to wait for first clock sync
				- prohibits these options:
				nofork
				quit
				saveconfigquit
   -x no  slew           Slew up to 600 seconds
      opt version        output version information and exit
   -? no  help           display extended usage information and exit
   -! no  more-help      extended usage information passed thru pager

Options are specified by doubled hyphens and their name or by a single
hyphen and the flag character.


The following option preset mechanisms are supported:
 - examining environment variables named NTPD_*

Please send bug reports to:  <http://bugs.ntp.org, bugs@ntp.org>
[root@h101 ntp-4.2.8p4]# 

~~~

停止ntp服务

~~~
[root@h101 sbin]# /etc/init.d/ntpd stop 
Shutting down ntpd:                                        [  OK  ]
[root@h101 sbin]# /etc/init.d/ntpd status
ntpd is stopped
[root@h101 sbin]# 
~~~

替换ntp服务程序

~~~
[root@h101 sbin]# mv ntpd ntpd.noconfig
[root@h101 sbin]# cp  /usr/local/bin/ntpd  . 
[root@h101 sbin]# ll ntpd*
-rwxr-xr-x 1 root root 2633989 Nov 10 10:02 ntpd
-rwxr-xr-x 1 root root  110896 Oct 26 23:32 ntpdate
-rwxr-xr-x 1 root root  250312 Oct 26 23:32 ntpdc
-rwxr-xr-x 1 root root 2630962 Nov  9 21:22 ntpd.noconfig
-rwxr-xr-x 1 root root  759720 Oct 26 23:32 ntpd.old.4.2.6p5
[root@h101 sbin]#
~~~


还原配置

~~~
[root@h101 sbin]# vim /etc/sysconfig/ntpd
[root@h101 sbin]# cat /etc/sysconfig/ntpd 
# Drop root to id 'ntp:ntp' by default.
OPTIONS="-u ntp:ntp -p /var/run/ntpd.pid -g"
#OPTIONS=" -p /var/run/ntpd.pid -g"
[root@h101 sbin]# 
~~~

启动服务

~~~
[root@h101 sbin]# /etc/init.d/ntpd start 
Starting ntpd:                                             [  OK  ]
[root@h101 sbin]# ps faux | grep ntp 
root     28277  0.0  0.0 103256   828 pts/0    S+   10:03   0:00          \_ grep ntp
ntp      28273  0.3  0.0 122812  2328 ?        Ssl  10:03   0:00 ntpd -u ntp:ntp -p /var/run/ntpd.pid -g
[root@h101 sbin]# 
[root@h101 sbin]# ntpq -p 
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
-202.120.2.100 ( 202.118.1.130    3 u    1   64  377    6.135    2.924   1.802
 s2a.time.edu.cn .INIT.          16 u    -  512    0    0.000    0.000   0.000
 202.112.0.7     .INIT.          16 u    -  512    0    0.000    0.000   0.000
 gus.buptnet.edu .INIT.          16 u    -  512    0    0.000    0.000   0.000
 mailgw.xanet.ed .INIT.          16 u    -  512    0    0.000    0.000   0.000
+dns1.synet.edu. 95.222.122.210   2 u    3   64    5  105.878   33.821   3.552
+202.112.26.37   202.112.29.82    3 u    8   64  377    5.129    2.683   0.321
 ntp.glnet.edu.c .INIT.          16 u    -  512    0    0.000    0.000   0.000
-ns.pku.edu.cn   129.6.15.28      2 u    7   64  377   97.361   57.123   1.872
*news.neu.edu.cn 95.222.122.210   2 u   30   64  377   38.277    1.324   2.049
[root@h101 sbin]# ntpstat  
synchronised to NTP server (202.118.1.81) at stratum 3 
   time correct to within 101 ms
   polling server every 64 s
[root@h101 sbin]# 
~~~




---




# 命令汇总


* ./configure    --enable-clockctl 
* ntpstat
* ntpq -p 
* ntptrace   202.120.2.100 


总体思路就是编译出新的NTP服务程序，然后替换掉原有的(做好备份，方便回退)

---
[ntp]:http://www.ntp.org/
[ntp_download]:http://www.ntp.org/downloads.html
[ntp_doc]:https://www.eecis.udel.edu/~mills/ntp/html/index.html
[ntp_notice]:http://support.ntp.org/bin/view/Main/SecurityNotice#Recent_Vulnerabilities
[ntp_bug]:http://arstechnica.com/security/2015/10/new-attacks-on-network-time-protocol-can-defeat-https-and-create-chaos/
[ntp_bug2]:http://www.tuicool.com/articles/aI7J3iZ
[ntp_build]:https://www.eecis.udel.edu/~mills/ntp/html/build.html



