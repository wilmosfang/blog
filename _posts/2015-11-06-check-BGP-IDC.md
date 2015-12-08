---
layout: post
title: BGP数据中心鉴别方法
categories: linux network
excerpt: follow me
comments: true
---


---

#前言

BGP(Border Gateway Protocol)是一种在自治系统之间动态交换路由信息的路由协议。

BGP数据中心，就是具备BGP线路的机房。

BGP双线优点：

* 安全：由于BGP协议本身具有冗余备份、消除环路的特点，所以当IDC服务商有多条BGP互联线路时可以实现路由的相互备份
* 单ip多线路：服务器只需要设置一个IP地址，最佳访问路由是由网络上的骨干路由器根据路由跳数与其它技术指标来确定的，不会对占用服务器的任何系统资源
* 更快速：使用BGP协议还可以使网络具有很强的扩展性可以将IDC网络与其他运营商互联，做到所有互联运营商的用户访问都很快

目前市面上模拟单IP多线效果的机房很多，他们都冒充自己是BGP机房，下面分享一下鉴别方法

[参考文档][BGP_ref]

---

#概要

* TOC
{:toc}



---

##鉴别方法 

主要分三步：

* 1.查询AS号
* 2.查询出口IP归属
* 3.查询AS是几线BGP

---

###查询AS号


使用traceroute命令查询网站IP/域名对应的AS号


 > **Tip:** **traceroute** 是通过 **traceroute-2.0.14-2.el6.x86_64** 提供的

{% highlight bash %}
[root@h101 ~]# which  traceroute 
/bin/traceroute
[root@h101 ~]# rpm -qf  /bin/traceroute 
traceroute-2.0.14-2.el6.x86_64
[root@h101 ~]# 
{% endhighlight %}

使用 **-A** 参数可以跟踪AS号

{% highlight bash %}
-A  --as-path-lookups       Perform AS path lookups in routing registries and
                              print results directly after the corresponding
                              addresses
{% endhighlight %}

在此以51IDC 提供的安畅云BGP ip 为例

![bgp_idc_check.png](/images/bgp_idc_check.png)

{% highlight bash %}
[root@h101 ~]# traceroute -A  103.21.118.104 
traceroute to 103.21.118.104 (103.21.118.104), 30 hops max, 60 byte packets
 1  192.168.2.254 (192.168.2.254) [*]  1.143 ms 192.168.2.75 (192.168.2.75) [*]  2.110 ms  0.634 ms
 2  58.246.136.1 (58.246.136.1) [AS17621]  2.788 ms  3.335 ms  2.156 ms
 3  112.64.252.85 (112.64.252.85) [AS17621]  3.550 ms  3.351 ms  3.257 ms
 4  139.226.204.37 (139.226.204.37) [*]  5.140 ms  4.882 ms  8.076 ms
 5  219.158.99.186 (219.158.99.186) [AS4837]  35.485 ms  35.481 ms  35.051 ms
 6  58.241.0.230 (58.241.0.230) [AS4837]  11.218 ms  13.538 ms  13.192 ms
 7  58.241.1.66 (58.241.1.66) [AS4837]  28.338 ms  28.574 ms  28.291 ms
 8  58.241.33.198 (58.241.33.198) [AS4837]  14.777 ms  436.646 ms  437.189 ms
 9  * * *
10  * * *
11  103.21.118.104 (103.21.118.104) [AS58879]  32.317 ms  32.508 ms  32.312 ms
12  * * *
13  103.21.118.104 (103.21.118.104) [AS58879]  32.252 ms  32.214 ms  32.592 ms
14  * * *
...
...
29  * * *
30  * * *
[root@h101 ~]# 
{% endhighlight %}

---

###查询出口IP归属

使用whois命令查询AS号的出口IP归属

> **Tip:** **whois** 是通过 **jwhois-4.0-19.el6.x86_64** 提供的

{% highlight bash %}
[root@h101 ~]# which whois
/usr/bin/whois
[root@h101 ~]# rpm -qf /usr/bin/whois
jwhois-4.0-19.el6.x86_64
[root@h101 ~]# 
{% endhighlight %}

直接加IP进行查询

{% highlight bash %}
[root@h101 ~]# whois  103.21.118.104
[Querying whois.arin.net]
[Redirected to whois.apnic.net]
[Querying whois.apnic.net]
[whois.apnic.net]
% [whois.apnic.net]
% Whois data copyright terms    http://www.apnic.net/db/dbcopyright.html

% Information related to '103.21.116.0 - 103.21.119.255'

inetnum:        103.21.116.0 - 103.21.119.255
netname:        ANCHNET
descr:          ShangHai AnchNet Tec, Inc
descr:          2F,Building 4,N0.1 West Hulan Road,ShangHai,PRC
country:        CN
admin-c:        XC1415-AP
tech-c:         XC1415-AP
mnt-by:         MAINT-CNNIC-AP
mnt-irt:        IRT-CNNIC-CN
mnt-routes:     MAINT-CNNIC-AP
status:         ALLOCATED PORTABLE
changed:        hm-changed@apnic.net 20120926
source:         APNIC

irt:            IRT-CNNIC-CN
address:        Beijing, China
e-mail:         ipas@cnnic.cn
abuse-mailbox:  ipas@cnnic.cn
admin-c:        IP50-AP
tech-c:         IP50-AP
auth:           # Filtered
remarks:        Please note that CNNIC is not an ISP and is not
remarks:        empowered to investigate complaints of network abuse.
remarks:        Please contact the tech-c or admin-c of the network.
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20110428
source:         APNIC

person:         Xiaozhong Cheng
address:        2F,Building 4,N0.1 West Hulan Road,ShangHai,PRC
country:        CN
phone:          +86-21-60832266-6603
e-mail:         x.z.cheng@anchnet.com
nic-hdl:        XC1415-AP
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20120306
source:         APNIC

% This query was served by the APNIC Whois Service version 1.69.1-APNICv1r6 (WHOIS1)


[root@h101 ~]# 
{% endhighlight %}

从结果信息得知 **103.21.116.0 - 103.21.119.255** 段的ip 都属于ANCHNET 安畅网络公司

{% highlight bash %}
inetnum:        103.21.116.0 - 103.21.119.255
netname:        ANCHNET
{% endhighlight %}

---

###查询AS是几线BGP



{% highlight bash %}
[root@h101 ~]# whois AS58879
[Querying whois.radb.net]
[whois.radb.net]
aut-num:    AS58879
as-name:    MYASNAME-1
descr:      ʏº£°²³©θç¿Ƽ¼¹ɷޓя
admin-c:    jiang yuanming
tech-c:     jiang yuanming
mnt-by:     MAINT-AS58879
changed:    jiangym@51idc.com 20151025  #01:52:26Z
source:     RADB

aut-num:        AS58879
as-name:        ANCHNET
descr:          Shanghai Anchang Network Security Technology Co.,Ltd.
descr:          Building 4,NO.1 West Hulan Road,Shanghai,PRC
country:        CN
admin-c:        AUTO1-WD
tech-c:         XC1-AUTO
mnt-by:         MAINT-AP-CNISP
mnt-irt:        IRT-CNISP-CN
changed:        ip@cnisp.org.cn 20131202
source:         APNIC
[root@h101 ~]# 
{% endhighlight %}

从结果看出，这并不是一个多线BGP网络

---

##其它案例


###上海有孚BGP

{% highlight bash %}
[root@h101 ~]# traceroute -A  175.102.17.5
traceroute to 175.102.17.5 (175.102.17.5), 30 hops max, 60 byte packets
 1  192.168.2.75 (192.168.2.75) [*]  0.507 ms 192.168.2.254 (192.168.2.254) [*]  1.333 ms  0.830 ms
 2  58.246.136.1 (58.246.136.1) [AS17621]  1.986 ms  4.136 ms  3.902 ms
 3  112.64.252.85 (112.64.252.85) [AS17621]  3.654 ms  3.513 ms  3.402 ms
 4  139.226.203.6 (139.226.203.6) [*]  4.634 ms  6.031 ms  4.995 ms
 5  112.65.207.45 (112.65.207.45) [AS17621]  8.638 ms 140.207.207.54 (140.207.207.54) [*]  7.542 ms 112.65.207.45 (112.65.207.45) [AS17621]  8.617 ms
 6  112.65.207.165 (112.65.207.165) [AS17621]  7.082 ms 140.207.207.57 (140.207.207.57) [*]  7.946 ms 112.65.207.165 (112.65.207.165) [AS17621]  7.794 ms
 7  211.95.77.26 (211.95.77.26) [AS9800]  7.598 ms  7.529 ms  7.277 ms
 8  * * *
 ...
 ...
 30  * * *
[root@h101 ~]# 
[root@h101 ~]# whois 175.102.17.5 
[Querying whois.arin.net]
[Redirected to whois.apnic.net]
[Querying whois.apnic.net]
[whois.apnic.net]
% [whois.apnic.net]
% Whois data copyright terms    http://www.apnic.net/db/dbcopyright.html

% Information related to '175.102.0.0 - 175.102.255.255'

inetnum:        175.102.0.0 - 175.102.255.255
netname:        YOVOLE
descr:          Shanghai Yovole Networks Inc.
descr:          12F,No.323,Guoding Road,Yangpu District,Shanghai,P.R.China
country:        CN
admin-c:        YW6230-AP
tech-c:         YW6230-AP
mnt-by:         MAINT-CNNIC-AP
mnt-lower:      MAINT-CNNIC-AP
mnt-routes:     MAINT-CNNIC-AP
mnt-irt:        IRT-CNNIC-CN
status:         ALLOCATED PORTABLE
changed:        hm-changed@apnic.net 20110304
source:         APNIC

irt:            IRT-CNNIC-CN
address:        Beijing, China
e-mail:         ipas@cnnic.cn
abuse-mailbox:  ipas@cnnic.cn
admin-c:        IP50-AP
tech-c:         IP50-AP
auth:           # Filtered
remarks:        Please note that CNNIC is not an ISP and is not
remarks:        empowered to investigate complaints of network abuse.
remarks:        Please contact the tech-c or admin-c of the network.
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20110428
source:         APNIC

person:         Lv Xin
address:        12F,No.323,Guoding Road,Yangpu District,Shanghai,P.R.China
country:        CN
phone:          +86-21-51263688-3101
e-mail:         lvxin@yovole.com
nic-hdl:        YW6230-AP
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20150401
source:         APNIC

% This query was served by the APNIC Whois Service version 1.69.1-APNICv1r6 (WHOIS2)


[root@h101 ~]# whois  AS9800
[Querying whois.radb.net]
[whois.radb.net]
aut-num:      AS9800
as-name:      CHINAUNICOM-BACKBONE
descr:        No.133,Xi'dan North Street
descr:        Beijing
descr:        100032
remarks:      for backbone of chinaunicom
admin-c:      LJ14-SAVVIS
tech-c:       CUN1-SAVVIS
mnt-by:       MAINT-AS9800
changed:      rabbi@bj.cnuninet.net 20030804
source:       SAVVIS

aut-num:      AS9800
as-name:      UNICOM
descr:        CHINA UNICOM
country:      CN
import:       from AS701
              action pref=10;
              accept ANY
import:       from AS1239
              action pref=10;
              accept ANY
import:       from AS4637
              action pref=10;
              accept ANY
import:       from AS4862
              action pref=10;
              accept ANY
import:       from AS6453
              action pref=10;
              accept ANY
import:       from AS9225
              action pref=10;
              accept ANY
import:       from AS10099
              action pref=10;
              accept ANY
import:       from AS17773
              action pref=10;
              accept ANY
export:       to AS701
              announce ANY
export:       to AS1239
              announce ANY
export:       to AS4637
              announce ANY
export:       to AS4862
              announce ANY
export:       to AS6453
              announce ANY
export:       to AS9225
              announce ANY
export:       to AS10099
              announce ANY
export:       to AS17773
              announce ANY
admin-c:      XZ67-AP
tech-c:       XZ67-AP
mnt-by:       MAINT-CNNIC-AP
mnt-lower:    MAINT-CNNIC-AP
mnt-routes:   MAINT-CNNIC-AP
changed:      ipas@cnnic.cn 20090424
source:       APNIC
[root@h101 ~]# 
{% endhighlight %}

被防火墙阻挡，但AS9800是一个多线BGP


---


###上海Ucloud BGP

{% highlight bash %}
[root@h101 ~]# traceroute -A  101.52.131.7 
traceroute to 101.52.131.7 (101.52.131.7), 30 hops max, 60 byte packets
 1  192.168.2.75 (192.168.2.75) [*]  0.507 ms 192.168.2.254 (192.168.2.254) [*]  0.507 ms  0.523 ms
 2  58.246.136.1 (58.246.136.1) [AS17621]  2.187 ms  3.021 ms  2.763 ms
 3  112.64.252.85 (112.64.252.85) [AS17621]  3.735 ms  3.256 ms  3.148 ms
 4  * 112.64.243.169 (112.64.243.169) [AS17621]  912.949 ms *
 5  112.64.246.210 (112.64.246.210) [AS17621]  4.607 ms  6.117 ms  5.995 ms
 6  211.95.40.26 (211.95.40.26) [AS9800]  45.173 ms  44.825 ms  44.528 ms
 7  101.52.139.230 (101.52.139.230) [AS45079]  13.975 ms  14.775 ms  15.628 ms
 8  * * *
 9  101.52.131.7 (101.52.131.7) [AS45079]  6.399 ms  6.288 ms  6.132 ms
[root@h101 ~]# whois 101.52.131.7 
[Querying whois.arin.net]
[Redirected to whois.apnic.net]
[Querying whois.apnic.net]
[whois.apnic.net]
% [whois.apnic.net]
% Whois data copyright terms    http://www.apnic.net/db/dbcopyright.html

% Information related to '101.52.128.0 - 101.52.131.255'

inetnum:        101.52.128.0 - 101.52.131.255
netname:        TrueStarNET
descr:          Shanghai OneTong Com. Ltd.
descr:          No.777 West GuangZhong Road,Shanghai
country:        CN
admin-c:        ZY1115-AP
tech-c:         ZL863-AP
mnt-by:         MAINT-CNNIC-AP
mnt-lower:      MAINT-CNNIC-AP
mnt-routes:     MAINT-CNNIC-AP
mnt-irt:        IRT-CNNIC-CN
status:         ALLOCATED PORTABLE
changed:        hm-changed@apnic.net 20071121
source:         APNIC

irt:            IRT-CNNIC-CN
address:        Beijing, China
e-mail:         ipas@cnnic.cn
abuse-mailbox:  ipas@cnnic.cn
admin-c:        IP50-AP
tech-c:         IP50-AP
auth:           # Filtered
remarks:        Please note that CNNIC is not an ISP and is not
remarks:        empowered to investigate complaints of network abuse.
remarks:        Please contact the tech-c or admin-c of the network.
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20110428
source:         APNIC

person:         Zhe Lian
nic-hdl:        ZL863-AP
e-mail:         morris@true-star.com
address:        26D WEST BUILDI EAST BEIJING ROAD
phone:          +86-021-61023603-6615
fax-no:         +86-021-63820122
country:        CN
changed:        ipas@cnnic.cn 20090224
mnt-by:         MAINT-CNNIC-AP
source:         APNIC

person:         Zhanwei Yang
nic-hdl:        ZY1115-AP
e-mail:         zwya@true-star.com
address:        26D WEST BUILDI EAST BEIJING ROAD
phone:          +86-021-61023603-6612
fax-no:         +86-021-63820122
country:        CN
changed:        ipas@cnnic.cn 20090224
mnt-by:         MAINT-CNNIC-AP
source:         APNIC

% Information related to '101.52.0.0/16AS45079'

route:          101.52.0.0/16
descr:          GDS CHANGAN SERVICES Ltd.
origin:         AS45079
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20150714
source:         APNIC

% This query was served by the APNIC Whois Service version 1.69.1-APNICv1r6-SNAPSHOT (WHOIS4)


[root@h101 ~]# whois AS45079
[Querying whois.radb.net]
[whois.radb.net]
aut-num:        AS45079
as-name:        GDSNET
descr:          GDS CHANGAN SERVICES Ltd.
descr:          F16,Electronic Science Plaza,No.12A,Jiuxianqiao Road,
descr:          Chaoyang District,Beijing 100102 China
country:        CN
admin-c:        ML2076-AP
tech-c:         BW808-AP
mnt-by:         MAINT-CNNIC-AP
mnt-lower:      MAINT-CNNIC-AP
mnt-irt:        IRT-CNNIC-CN
changed:        ipas@cnnic.net.cn 20140324
source:         APNIC
[root@h101 ~]# 
{% endhighlight %}


并不是一个多线BGP

---

###北京Ucloud BGP

{% highlight bash %}
[root@h101 ~]# traceroute -A  120.132.92.156 
traceroute to 120.132.92.156 (120.132.92.156), 30 hops max, 60 byte packets
 1  192.168.2.75 (192.168.2.75) [*]  0.604 ms  0.672 ms  0.250 ms
 2  58.246.136.1 (58.246.136.1) [AS17621]  2.351 ms  2.300 ms  3.001 ms
 3  112.64.252.85 (112.64.252.85) [AS17621]  3.238 ms  3.281 ms  3.155 ms
 4  * * *
 5  112.64.243.77 (112.64.243.77) [AS17621]  5.362 ms 139.226.204.9 (139.226.204.9) [*]  4.964 ms 112.64.243.69 (112.64.243.69) [AS17621]  7.537 ms
 6  219.158.12.85 (219.158.12.85) [AS4134/AS4837]  30.428 ms  31.655 ms  31.502 ms
 7  123.126.0.66 (123.126.0.66) [AS4134]  31.739 ms  31.556 ms  32.036 ms
 8  123.126.6.237 (123.126.6.237) [AS4134]  31.587 ms  31.524 ms  31.410 ms
 9  61.148.146.158 (61.148.146.158) [AS4808/AS4814]  31.181 ms  31.040 ms  30.878 ms
10  61.48.75.2 (61.48.75.2) [AS4814]  31.668 ms  31.639 ms  31.415 ms
11  * * *
12  * * *
13  180.150.176.222 (180.150.176.222) [AS59089]  784.985 ms  788.087 ms  786.583 ms
14  * * *
15  * * *
16  * * *
17  120.132.92.156 (120.132.92.156) [AS59089]  104.907 ms  104.844 ms  104.714 ms
18  * * *
...
...
30  * * *
[root@h101 ~]# whois 120.132.92.156 
[Querying whois.arin.net]
[Redirected to whois.apnic.net]
[Querying whois.apnic.net]
[whois.apnic.net]
% [whois.apnic.net]
% Whois data copyright terms    http://www.apnic.net/db/dbcopyright.html

% Information related to '120.132.32.0 - 120.132.95.255'

inetnum:        120.132.32.0 - 120.132.95.255
netname:        CloudVsp
descr:          CloudVsp.Inc
descr:          NO.18 Building University of Technology
descr:          Beijing Economic-Technological Development Area
admin-c:        HL2919-AP
tech-c:         XM632-AP
country:        CN
mnt-by:         MAINT-CNNIC-AP
mnt-lower:      MAINT-CNNIC-AP
mnt-irt:        IRT-CNNIC-CN
mnt-routes:     MAINT-CNNIC-AP
status:         ALLOCATED PORTABLE
changed:        hm-changed@apnic.net 20140702
source:         APNIC

irt:            IRT-CNNIC-CN
address:        Beijing, China
e-mail:         ipas@cnnic.cn
abuse-mailbox:  ipas@cnnic.cn
admin-c:        IP50-AP
tech-c:         IP50-AP
auth:           # Filtered
remarks:        Please note that CNNIC is not an ISP and is not
remarks:        empowered to investigate complaints of network abuse.
remarks:        Please contact the tech-c or admin-c of the network.
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn 20110428
source:         APNIC

person:         Huakun Li
nic-hdl:        HL2919-AP
e-mail:         lihuakun@cloudvsp.com
address:        NO.18 Building University of Technology
address:        Beijing Economic-Technological Development Area
phone:          +86-18101125590
fax-no:         +86-10-87529719
country:        CN
changed:        ipas@cnnic.net.cn 20140421
mnt-by:         MAINT-CNNIC-AP
source:         APNIC

person:         Xiaobing Mao
nic-hdl:        XM632-AP
e-mail:         maoxiaobing@cloudvsp.com
address:        NO.18 Building University of Technology
address:        Beijing Economic-Technological Development Area
phone:          +86-10-87120550
fax-no:         +86-10-87529719
country:        CN
changed:        ipas@cnnic.net.cn 20150120
mnt-by:         MAINT-CNNIC-AP
source:         APNIC

% Information related to '120.132.92.0/22AS59089'

route:          120.132.92.0/22
descr:          CloudVsp.Inc
country:        CN
origin:         AS59089
notify:         lihuakun@cloudvsp.com
mnt-by:         MAINT-CNNIC-AP
changed:        ipas@cnnic.cn  20141029
source:         APNIC

% This query was served by the APNIC Whois Service version 1.69.1-APNICv1r6 (WHOIS1)


[root@h101 ~]# whois AS59089
[Querying whois.radb.net]
[whois.radb.net]
aut-num:        AS59089
as-name:        CloudVsp
descr:          NO.18 Building University of Technology
descr:          Beijing Economic-Technological Development Area
country:        CN
import:         from AS24138
                action pref=10;
                accept ANY
import:         from AS56048
                action pref=10;
                accept ANY
export:         to AS24138
                announce AS59089
export:         to AS56048
                announce AS59089
admin-c:        XJ1597-AP
tech-c:         XM632-AP
mnt-by:         MAINT-CNNIC-AP
mnt-routes:     MAINT-CNNIC-AP
mnt-irt:        IRT-CNNIC-CN
changed:        ipas@cnnic.cn 20141021
source:         APNIC
[root@h101 ~]# 
{% endhighlight %}


可知AS59089分别和AS24138、AS56048建立了BGP连接，是一个双线BGP


---

###广东Ucloud BGP

{% highlight bash %}
[root@h101 ~]# traceroute -A  114.119.43.116 
traceroute to 114.119.43.116 (114.119.43.116), 30 hops max, 60 byte packets
 1  192.168.2.75 (192.168.2.75) [*]  0.524 ms  0.977 ms  0.396 ms
 2  58.246.136.1 (58.246.136.1) [AS17621]  3.100 ms  2.933 ms  4.497 ms
 3  112.64.252.85 (112.64.252.85) [AS17621]  3.468 ms  3.359 ms  3.597 ms
 4  * * *
 5  139.226.204.9 (139.226.204.9) [*]  5.436 ms 139.226.201.73 (139.226.201.73) [*]  10.078 ms 139.226.204.17 (139.226.204.17) [*]  9.429 ms
 6  219.158.99.145 (219.158.99.145) [AS4837]  7.101 ms  7.314 ms  7.239 ms
 7  219.158.7.194 (219.158.7.194) [AS4134/AS4837]  37.171 ms  37.075 ms  36.785 ms
 8  120.82.0.182 (120.82.0.182) [AS17816]  38.078 ms  38.014 ms  37.852 ms
 9  * * *
 ...
 ...
 30  * * *
[root@h101 ~]# whois  114.119.43.116 
[Querying whois.arin.net]
[Redirected to whois.apnic.net]
[Querying whois.apnic.net]
[whois.apnic.net]
% [whois.apnic.net]
% Whois data copyright terms    http://www.apnic.net/db/dbcopyright.html

% Information related to '114.119.0.0 - 114.119.127.255'

inetnum:        114.119.0.0 - 114.119.127.255
netname:        SACCL
descr:          Shenzhen Aosida Communication Co., Ltd.
descr:          808,8th Building,No 4 Nanyou Industry,NanShan District
country:        CN
admin-c:        SACC1-AP
tech-c:         SACC1-AP
mnt-by:         APNIC-HM
mnt-lower:      MAINT-SACCL-CN
mnt-routes:     MAINT-SACCL-CN
mnt-irt:        IRT-SACCL-CN
status:         ALLOCATED PORTABLE
changed:        hm-changed@apnic.net 20140627
source:         APNIC

irt:            IRT-SACCL-CN
address:        808,8th Building,No 4 Nanyou Industry,NanShan District, ShenZhen Guangdong Province 518000
e-mail:         sherry998877@163.com
abuse-mailbox:  sherry998877@163.com
admin-c:        SACC1-AP
tech-c:         SACC1-AP
auth:           # Filtered
mnt-by:         MAINT-SACCL-CN
changed:        hm-changed@apnic.net 20140603
source:         APNIC

role:           Shenzhen Aosida Communication Co Ltd administra
address:        808,8th Building,No 4 Nanyou Industry,NanShan District, ShenZhen Guangdong Province 518000
country:        CN
phone:          +86-0755-86158808
fax-no:         +86-0755-86158808
e-mail:         sherry998877@163.com
admin-c:        SACC1-AP
tech-c:         SACC1-AP
nic-hdl:        SACC1-AP
mnt-by:         MAINT-SACCL-CN
changed:        hm-changed@apnic.net 20140603
source:         APNIC

% Information related to '114.119.0.0/17AS17816'

route:          114.119.0.0/17
descr:          China Unicom CHINA169 Guangdong Province network
descr:          Addresses from CNNIC
country:        CN
origin:         AS17816
mnt-by:         MAINT-CNCGROUP-RR
changed:        abuse@cnc-noc.net 20090202
source:         APNIC

% This query was served by the APNIC Whois Service version 1.69.1-APNICv1r6 (WHOIS1)


[root@h101 ~]# whois AS17816
[Querying whois.radb.net]
[whois.radb.net]
aut-num:        AS17816
as-name:        CHINA169-GZ
descr:          China Unicom  IP network China169 Guangdong province
country:        CN
member-of:      AS4837:AS-CNCGROUP
import:         from AS4837
                        action pref=100;
                        accept ANY
import:         from AS17490
                       action pref=100;
                       accept AS17490
export:         to AS4837
                          announce AS17816
export:         to AS17490
                          announce AS17816
admin-c:        CH1302-AP
tech-c:         CH1302-AP
mnt-by:         MAINT-CNCGROUP
mnt-routes:     MAINT-CNCGROUP-RR
changed:        abuse@cnc-noc.net 20101021
source:         APNIC
[root@h101 ~]# 
{% endhighlight %}

被防火墙阻截，但AS17816是一个双线BGP

###结论
  

只有北京Ucloud是多线BGP，其它都不是



---

#命令汇总


* traceroute -A  103.21.118.104
* whois  103.21.118.104
* whois AS58879

---

[BGP_ref]:http://support.huawei.com/ecommunity/bbs/10228617.html


