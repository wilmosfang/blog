---
layout: post
title: GoAccess access日志分析工具
author: wilmosfang
tags:  linux nginx log
categories:  log
wc: 264 872 11009
excerpt: web access log 日志分析工具 goaccess 的安装、报告的方法
comments: true
---

# 前言

Web运维难免涉及到access log的分析工作,虽然使用awk可以实现绝大部分的工作，但是效率不高，也不具备可视化的效果，报告形式很单调

在网上经过一翻搜索后，发现了一款分析软件[GoAccess](http://goaccess.io/),非常好用，可以自动生成报表，结合一下其它工具，可以定期给自己发一个access log分析报表


---

# 概要

* TOC
{:toc}


---


## 安装GoAccess

如果配置了epel库，可以直接使用yum安装，过程非常简单

~~~
[root@Test-slave ~]# yum -y install  goaccess 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * epel: mirrors.yun-idc.com
 * extras: mirrors.skyshe.cn
 * updates: mirrors.skyshe.cn
Resolving Dependencies
--> Running transaction check
---> Package goaccess.x86_64 0:0.8.5-1.el6 will be installed
--> Processing Dependency: libGeoIP.so.1()(64bit) for package: goaccess-0.8.5-1.el6.x86_64
--> Running transaction check
---> Package GeoIP.x86_64 0:1.5.1-5.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================
 Package                Arch                 Version                     Repository          Size
==================================================================================================
Installing:
 goaccess               x86_64               0.8.5-1.el6                 epel                82 k
Installing for dependencies:
 GeoIP                  x86_64               1.5.1-5.el6                 epel                21 M

Transaction Summary
==================================================================================================
Install       2 Package(s)

Total download size: 21 M
Installed size: 42 M
Downloading Packages:
(1/2): GeoIP-1.5.1-5.el6.x86_64.rpm                                        |  21 MB     02:03     
(2/2): goaccess-0.8.5-1.el6.x86_64.rpm                                     |  82 kB     00:00     
--------------------------------------------------------------------------------------------------
Total                                                             172 kB/s |  21 MB     02:03     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : GeoIP-1.5.1-5.el6.x86_64                                                       1/2 
  Installing : goaccess-0.8.5-1.el6.x86_64                                                    2/2 
  Verifying  : goaccess-0.8.5-1.el6.x86_64                                                    1/2 
  Verifying  : GeoIP-1.5.1-5.el6.x86_64                                                       2/2 

Installed:
  goaccess.x86_64 0:0.8.5-1.el6                                                                   

Dependency Installed:
  GeoIP.x86_64 0:1.5.1-5.el6                                                                      

Complete!
[root@Test-slave ~]# goaccess  -V
GoAccess - 0.8.5.
For more details visit: http://goaccess.io
Copyright (C) 2009-2014 GNU GPL'd, by Gerardo Orellana
[root@Test-slave ~]#
~~~

如果是源码安装，这里提供了[下载和安装方法](http://goaccess.io/download)

当前的最新版本是0.9


## 终端生成报表

* 1.命令行键入 **goaccess -f access.log**


~~~
[root@Test-slave tmp]# goaccess -f access.log 
~~~

* 2.弹出一个交互窗口

~~~
GoAccess  
                       +--------------------------------------------------+
                       | Log Format Configuration                         |
                       | [SPACE] to toggle - [ENTER] to proceed           |
                       |                                                  |
                       | [ ] NCSA Combined Log Format                     |
                       | [ ] NCSA Combined Log Format with Virtual Host   |
                       | [ ] Common Log Format (CLF)                      |
                       | [ ] Common Log Format (CLF) with Virtual Host    |
                       | [ ] W3C                                          |
                       | [ ] CloudFront (Download Distribution)           |
                       |                                                  |
                       | Log Format - [c] to add/edit format              |
                       |                                                  |
                       |                                                  |
                       | Date Format - [d] to add/edit format             |
                       |                                                  |
                       +--------------------------------------------------+

~~~

* 3.选择一种格式，由于我的access.log最匹配第一种格式，于是选择了它，这是goaccess能进行正常解析的基础

~~~
GoAccess
                       +--------------------------------------------------+
                       | Log Format Configuration                         |
                       | [SPACE] to toggle - [ENTER] to proceed           |
                       |                                                  |
                       | [x] NCSA Combined Log Format                     |
                       | [ ] NCSA Combined Log Format with Virtual Host   |
                       | [ ] Common Log Format (CLF)                      |
                       | [ ] Common Log Format (CLF) with Virtual Host    |
                       | [ ] W3C                                          |
                       | [ ] CloudFront (Download Distribution)           |
                       |                                                  |
                       | Log Format - [c] to add/edit format              |
                       | %h %^[%d:%^] "%r" %s %b "%R" "%u"                |
                       |                                                  |
                       | Date Format - [d] to add/edit format             |
                       | %d/%b/%Y                                         |
                       +--------------------------------------------------+


~~~

* 3.经过一段时间的解析，出现下面的Dashboard

~~~
 Dashboard - Overall Analyzed Requests                                   [Active Panel: Visitors]

  Total Requests  1278268 Unique Visitors 49690  Referrers    107473 Log Size  301.37 MiB
  Failed Requests 0       Unique Files    920353 Unique 404   2030   Bandwidth 13.81 GiB
  Generation Time 23      Excl. IP Hits   0      Static Files 17     Log File  access.log
                       | [ ] NCSA Combined Log Format with Virtual Host   |
 1 - Unique visitors per day - Including spiders                                      Total: 1/1
 Hits having the same IP, date and agent are a unique visit.

  49690 100.00%   13.81 GiB 26/Mar/2015 ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||

~~~

* 4.使用上下翻页可以看到其它的统计信息

~~~
 Dashboard - Overall Analyzed Requests                                                          [Active Panel: Visitors]

  Total Requests  1278268 Unique Visitors 49690  Referrers    107473 Log Size  301.37 MiB
  Failed Requests 0       Unique Files    920353 Unique 404   2030   Bandwidth 13.81 GiB
  Generation Time 23      Excl. IP Hits   0	 Static Files 17     Log File  access.log

 6 - Operating Systems                                                                                     Total: 52/52
 Top Operating Systems sorted by unique visitors

  33241 66.90% Windows   ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
  7640  15.38% Macintosh ||||||||||||||||||||||
  6988  14.06% Android   ||||||||||||||||||||
  1421  2.86%  Unknown   ||||
  247   0.50%  Linux     |
  122   0.25%  Unix-like |
  27    0.05%  Others    |

 7 - Browsers                                                                                            Total: 300/626
 Top Browsers sorted by unique visitors

  23131 46.55% Chrome   |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
  10370 20.87% MSIE     |||||||||||||||||||||||||||||||||||||||||||
  8934  17.98% Safari   |||||||||||||||||||||||||||||||||||||
 [F1]Help [O]pen detail view  0 - Thu Mar 26 17:30:05 2015                                          [Q]uit GoAccess 0.8.
~~~

* 5.下面是请求源的地理信息

~~~
 Dashboard - Overall Analyzed Requests                                                          [Active Panel: Visitors]

  Total Requests  1278268 Unique Visitors 49690  Referrers    107473 Log Size  301.37 MiB
  Failed Requests 0       Unique Files    920353 Unique 404   2030   Bandwidth 13.81 GiB
  Generation Time 23      Excl. IP Hits   0	 Static Files 17     Log File  access.log

 Continent > Country sorted by unique visitors

  46602 93.79% AS Asia
  2003  4.03%  NA North America
  613   1.23%  EU Europe
  320   0.64%  OC Oceania
  122   0.25%  -- Location Unknown
  17    0.03%  SA South America
  13    0.03%  AF Africa

 12 - HTTP Status Codes                                                                                    Total: 13/13
 Top HTTP Status Codes sorted by hits

  1211257 94.76% 2xx Success
  32348   2.53%  3xx Redirection
  21406   1.67%  5xx Server Error
  13243   1.04%  4xx Client Error
 [F1]Help [O]pen detail view  0 - Thu Mar 26 17:30:05 2015                                          [Q]uit GoAccess 0.8.
~~~


是不是很强大，至少我是这么觉得的，下面展示更强大的显示效果


## 生成html报告


* 1.命令行键入 **goaccess -f access.log  --date-format='%d/%b/%Y'  --log-format='%h %^[%d:%^] "%r" %s %b "%R" "%u"' > abc.html**

必须指定至少下面几个参数

* a valid IPv4/6 **%h**
* a valid date **%d**
* server status code **%s**
* the request **%r**

* 2.使用任何一款浏览器打开abc.html

![goaccess-dashboard](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/goaccess/goaccess-dashboard.png)

![goaccess-host](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/goaccess/goaccess-host.png)

![goaccess-os](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/goaccess/goaccess-os.png)

![goaccess-code](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/goaccess/goaccess-code.png)


# 附

参考链接

[GoAccess](http://goaccess.io/)

[Install GoAccess](http://goaccess.io/download)

[Man Page for GoAccess](http://goaccess.io/man)
