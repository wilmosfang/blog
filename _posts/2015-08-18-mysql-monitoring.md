---
layout: post
title: Mysql 监控 
author: wilmosfang
tags:  monitoring zabbix mysql
categories: zabbix
wc: 413 1448 16383
excerpt: 使用Zabbix监控Mysql的方法 
comments: true
date:   2015-08-18 00:00:00
---

# 前言


大部分生产系统从一开始就要考虑它的 **高可用** 和 **监控** ，数据库更是如此，这里我分享一下Mysql的[监控方法][pmp]

---


# 概要

* TOC
{:toc}

---


## 环境


在 **Centos 6.7** 下面 运行着 **mysql 5.6.25** ([percona][percona] 版本)

~~~
[root@mysql-server packages]# cat /etc/issue
CentOS release 6.7 (Final)
Kernel \r on an \m

[root@mysql-server packages]# uname  -r 
2.6.32-573.1.1.el6.x86_64
[root@mysql-server packages]# mysql -V 
mysql  Ver 14.14 Distrib 5.6.25-73.1, for Linux (x86_64) using  6.0
[root@mysql-server packages]# 
~~~

## 准备插件包


使用[percona][percona]的[repo][pyum]下载下列插件

~~~
[root@mysql-server packages]# ll *zabbix*
-rw-r--r--. 1 root root 30599 Jun 19 17:39 percona-zabbix-templates-1.1.5-1.noarch.rpm
[root@mysql-server packages]# 
~~~

这个包里主要包含：

* 一个 **xml** 模板 : 用来构建mysql监控模板
* 一个 **php** 脚本 : 用来收集mysql状态信息
* 一个 **shell** 脚本 : 用来调用上面的脚本
* 一个mysql 监控插件配置文件 : 用来自定义用户插件

~~~
[root@mysql-server packages]# rpm -qlp percona-zabbix-templates-1.1.5-1.noarch.rpm 
warning: percona-zabbix-templates-1.1.5-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
/var/lib/zabbix/percona
/var/lib/zabbix/percona/scripts
/var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh
/var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php
/var/lib/zabbix/percona/templates
/var/lib/zabbix/percona/templates/userparameter_percona_mysql.conf
/var/lib/zabbix/percona/templates/zabbix_agent_template_percona_mysql_server_ht_2.0.9-sver1.1.5.xml
[root@mysql-server packages]# 
~~~

## 安装插件包


~~~
[root@mysql-server packages]# rpm -ivh  percona-zabbix-templates-1.1.5-1.noarch.rpm 
warning: percona-zabbix-templates-1.1.5-1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:percona-zabbix-template########################################### [100%]

Scripts are installed to /var/lib/zabbix/percona/scripts
Templates are installed to /var/lib/zabbix/percona/templates
[root@mysql-server packages]# 
~~~

## 拷贝配置


拷贝 **userparameter_percona_mysql.conf** 到配置目录


~~~
[root@mysql-server packages]# cp /var/lib/zabbix/percona/templates/userparameter_percona_mysql.conf  /etc/zabbix/zabbix_agentd.d/
[root@mysql-server packages]# 
~~~

## 配置密码


在相应目录下创建密码配置文件 **ss_get_mysql_stats.php.cnf**


~~~
[root@mysql-server scripts]# cat /var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php.cnf
<?php
$mysql_user = 'root';
$mysql_pass = 'xxxxxxx';
[root@mysql-server scripts]# 
~~~

尝试运行一下状态收集脚本

~~~
[root@mysql-server scripts]# /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg 
/var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh: line 35: /usr/bin/php: No such file or directory
ERROR: run the command manually to investigate the problem: /usr/bin/php -q /var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php --host localhost --items gg
[root@mysql-server scripts]# 
~~~

## 安装依赖包


这里提示我们系统里没有安装 **php** ,我们给它装上，同时我们也装上 **php-mysql** ,它提供了php 连接 mysql 需要的DBI

~~~
[root@mysql-server scripts]# yum install php php-mysql 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.pubyun.com
 * extras: mirrors.yun-idc.com
 * updates: mirrors.pubyun.com
Resolving Dependencies
--> Running transaction check
---> Package php.x86_64 0:5.3.3-46.el6_6 will be installed
--> Processing Dependency: php-common(x86-64) = 5.3.3-46.el6_6 for package: php-5.3.3-46.el6_6.x86_64
--> Processing Dependency: php-cli(x86-64) = 5.3.3-46.el6_6 for package: php-5.3.3-46.el6_6.x86_64
---> Package php-mysql.x86_64 0:5.3.3-46.el6_6 will be installed
--> Processing Dependency: php-pdo(x86-64) for package: php-mysql-5.3.3-46.el6_6.x86_64
--> Processing Dependency: libmysqlclient.so.16(libmysqlclient_16)(64bit) for package: php-mysql-5.3.3-46.el6_6.x86_64
--> Processing Dependency: libmysqlclient.so.16()(64bit) for package: php-mysql-5.3.3-46.el6_6.x86_64
--> Running transaction check
---> Package Percona-Server-shared-51.x86_64 0:5.1.73-rel14.12.624.rhel6 will be installed
---> Package php-cli.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package php-common.x86_64 0:5.3.3-46.el6_6 will be installed
---> Package php-pdo.x86_64 0:5.3.3-46.el6_6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================
 Package                          Arch           Version                           Repository                      Size
========================================================================================================================
Installing:
 php                              x86_64         5.3.3-46.el6_6                    updates                        1.1 M
 php-mysql                        x86_64         5.3.3-46.el6_6                    updates                         86 k
Installing for dependencies:
 Percona-Server-shared-51         x86_64         5.1.73-rel14.12.624.rhel6         percona-release-x86_64         2.1 M
 php-cli                          x86_64         5.3.3-46.el6_6                    updates                        2.2 M
 php-common                       x86_64         5.3.3-46.el6_6                    updates                        529 k
 php-pdo                          x86_64         5.3.3-46.el6_6                    updates                         79 k

Transaction Summary
========================================================================================================================
Install       6 Package(s)

Total download size: 6.1 M
Installed size: 19 M
Is this ok [y/N]: y
Downloading Packages:
...
...
Installed:
  php.x86_64 0:5.3.3-46.el6_6                              php-mysql.x86_64 0:5.3.3-46.el6_6                             

Dependency Installed:
  Percona-Server-shared-51.x86_64 0:5.1.73-rel14.12.624.rhel6               php-cli.x86_64 0:5.3.3-46.el6_6              
  php-common.x86_64 0:5.3.3-46.el6_6                                        php-pdo.x86_64 0:5.3.3-46.el6_6              

Complete!
[root@mysql-server scripts]#
~~~

## 测试脚本


装完包后，再次执行测试脚本，就正常返回一个数字了

~~~
[root@mysql-server scripts]# /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg
0
[root@mysql-server scripts]# /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gt
38409
~~~

这个数据从哪里来的呢, 执行脚本的过程中生成了这个文件 **/tmp/localhost-mysql_cacti_stats.txt**

~~~
[root@mysql-server scripts]# cat /tmp/localhost-mysql_cacti_stats.txt
gg:0 gh:0 gi:0 gj:0 gk:3275 gl:7096091357813 gm:0 gn:2 go:0 gp:0 gq:1310712 gr:560473 gs:747264 gt:38409 gu:666299 gv:81269 gw:3884351 gx:4850164 gy:666658 gz:22406662 hg:16685721 hh:0 hi:0 hj:0 hk:0 hl:0 hm:0 hn:0 ho:0 hp:0 hq:61117 hr:79688 hs:72627 ht:3483741 hu:46269304 hv:1477167 hw:10612667 hx:10992203 hy:712017 hz:11704230 ig:0 ih:358979056 ii:0 ij:23 ik:245 il:263 im:2048 in:10245 io:2048 ip:0 iq:2 ir:1 is:0 it:0 iu:1 iv:1 iw:1 ix:2048 iy:8 iz:12 jg:0 jh:0 ji:0 jj:0 jk:0 jl:1 jm:268417400 jn:0 jo:0 jp:0 jq:6 jr:0 js:1 jt:268435456 ju:52 jv:10816055 jw:10511991 jx:6 jy:719085 jz:0 kg:0 kh:0 ki:0 kj:0 kk:0 kl:0 km:0 kn:0 ko:0 kp:3 kq:0 kr:0 ks:0 kt:0 ku:3 kv:0 kw:6 kx:87998 ky:6025810131 kz:8388608 lg:8388608 lh:4347852912824 li:4347852916081 lj:575663419 lk:1048576 ll:0 lm:0 ln:638 lo:0 lp:0 lq:0 lr:0 ls:0 lt:0 lu:1 lv:0 lw:0 lx:0 ly:0 lz:0 mg:0 mh:0 mi:0 mj:0 mk:0 ml:2 mm:38445444 mn:712017 mo:0 mp:0 mq:6 mr:11974037 ms:10 mt:0 mu:11971670 mv:62 mw:0 mx:0 my:0 mz:10992203 ng:10612792 nh:0 ni:0 nj:-1 nk:-1 nl:21978152960 nm:0 nn:72550322 no:1 np:813 nq:815 nr:388741648 ns:2657176 nt:87248076 nu:1197592 nv:53125976 nw:0 nx:-1 ny:-1 nz:-1 og:0 oh:6119424 oi:33554432 oj:0 ok:0 ol:-1 om:-1 on:-1 oo:-1 op:-1 oq:-1 or:-1 os:-1 ot:-1 ou:-1 ov:-1 ow:-1 ox:-1 oy:-1 oz:-1 pg:-1 ph:-1 pi:-1 pj:-1 pk:-1 pl:-1 pm:-1 pn:-1 po:-1 pp:-1 pq:-1 pr:-1 ps:-1 pt:-1 pu:-1 pv:-1 pw:-1 px:-1 py:-1 pz:-1 qg:-1 qh:-1 qi:-1 qj:-1 qk:-1 ql:-1 qm:-1 qn:-1 qo:607839 qp:1964228995[root@mysql-server scripts]# 
[root@mysql-server scripts]# 
~~~

这个脚本并不长，总共只有43行，是对 **ss_get_mysql_stats.php** 的一层包装，罗辑非常简单，看看就知道了

~~~
[root@mysql-server scripts]# wc  -l  /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh 
43 /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh
[root@mysql-server scripts]# cat /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh
#!/bin/sh
# The wrapper for Cacti PHP script.
# It runs the script every 5 min. and parses the cache file on each following run.
# Version: 1.1.5
#
# This program is part of Percona Monitoring Plugins
# License: GPL License (see COPYING)
# Copyright: 2015 Percona
# Authors: Roman Vynar

ITEM=$1
HOST=localhost
DIR=`dirname $0`
CMD="/usr/bin/php -q $DIR/ss_get_mysql_stats.php --host $HOST --items gg"
CACHEFILE="/tmp/$HOST-mysql_cacti_stats.txt"

if [ "$ITEM" = "running-slave" ]; then
    # Check for running slave
    RES=`HOME=~zabbix mysql -e 'SHOW SLAVE STATUS\G' | egrep '(Slave_IO_Running|Slave_SQL_Running):' | awk -F: '{print $2}' | tr '\n' ','`
    if [ "$RES" = " Yes, Yes," ]; then
        echo 1
    else
        echo 0
    fi
    exit
elif [ -e $CACHEFILE ]; then
    # Check and run the script
    TIMEFLM=`stat -c %Y /tmp/$HOST-mysql_cacti_stats.txt`
    TIMENOW=`date +%s`
    if [ `expr $TIMENOW - $TIMEFLM` -gt 300 ]; then
        rm -f $CACHEFILE
        $CMD 2>&1 > /dev/null
    fi
else
    $CMD 2>&1 > /dev/null
fi

# Parse cache file
if [ -e $CACHEFILE ]; then
    cat $CACHEFILE | sed 's/ /\n/g; s/-1/0/g'| grep $ITEM | awk -F: '{print $2}'
else
    echo "ERROR: run the command manually to investigate the problem: $CMD"
fi
[root@mysql-server scripts]# 
~~~

目前是使用 **root** 的身份执行的，但是 **zabbix agent** 是使用 **zabbix** 身份来执行这条命令的，我们尝试使用 **zabbix** 来执行一下，看看效果

~~~
[root@mysql-server scripts]# sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh running-slave 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
0
[root@mysql-server scripts]# sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg
rm: cannot remove `/tmp/localhost-mysql_cacti_stats.txt': Operation not permitted
0
[root@mysql-server scripts]# 
~~~

前一条报错的原因是 **zabbix** 用户在查询 **Slave_IO_Running\|Slave_SQL_Running** 时，没有访问数据库的权限

后一条报错的原因是对于之前使用root生成的 **/tmp/localhost-mysql_cacti_stats.txt** zabbix没有写权限


## 给zabbix赋权


分别来进行处理,先处理写权限问题

~~~
[root@mysql-server scripts]# chown  zabbix.zabbix /tmp/localhost-mysql_cacti_stats.txt  
[root@mysql-server scripts]# sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg 
0
[root@mysql-server scripts]#
~~~

再处理数据库访问权限问题

安装zabbix-agent时自动创建了zabbix用户，这样的用户没有登录权限，并且把 **/var/lib/zabbix** 当自己的家 (一个无家可归的孩子)

~~~
zabbix:x:496:493:Zabbix Monitoring System:/var/lib/zabbix:/sbin/nologin
~~~

修改 **/etc/passwd**，我们把它修改成这样

~~~
zabbix:x:496:493:Zabbix Monitoring System:/home/zabbix:/bin/bash
~~~

然后给 **zabbix** 创建一个家

~~~
[root@mysql-server ~]# mkdir /home/zabbix
[root@mysql-server ~]# cp /etc/skel/.*  /home/zabbix/
cp: omitting directory `/etc/skel/.'
cp: omitting directory `/etc/skel/..'
cp: omitting directory `/etc/skel/.gnome2'
cp: omitting directory `/etc/skel/.mozilla'
[root@mysql-server ~]# chown -R zabbix.zabbix /home/zabbix/
[root@mysql-server ~]# su - zabbix 
[zabbix@mysql-server ~]$ vim .my.cnf
[zabbix@mysql-server ~]$ cat .my.cnf 
[client]
user = root
password = xxxxxx
[zabbix@mysql-server ~]$ 
~~~

再试试,就一切正常了

~~~
[root@mysql-server ~]# sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh running-slave 
1
[root@mysql-server ~]# su - zabbix 
[zabbix@mysql-server ~]$ /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh running-slave
1
[zabbix@mysql-server ~]$ 
~~~

然后重启 **zabbix-agent** ，只有重启，zabbix-agent 才能读到变化后的配置

~~~
[root@mysql-server ~]# /etc/init.d/zabbix-agent  restart 
Shutting down Zabbix agent:                                [  OK  ]
Starting Zabbix agent:                                     [  OK  ]
[root@mysql-server ~]# 
~~~

## 连接测试


在 **zabbix-server** 测试一下连接

~~~
[root@zabbix-server ~]# zabbix_get -s mysql-server -p 10050 -k "MySQL.running-slave" 
1
[root@zabbix-server ~]# zabbix_get -s mysql-server -p 10050 -k "MySQL.Threads-connected" 
1
[root@zabbix-server ~]# zabbix_get -s mysql-server -p 10050 -k "MySQL.max-connections" 
2048
[root@zabbix-server ~]# 
~~~



## 添加模板


[配置][pmp.installation] **Zabbix Server** 

* Grab the latest tarball from the Percona Software Downloads directory to your desktop.
* Unpack it to get zabbix/templates/ folder.
* Import the XML template using Zabbix UI (Configuration -> Templates ->  Import) by additionally choosing “Screens”.
* Create/edit hosts by assigning them “Percona Templates” group and linking the template “Percona MySQL Server Template” (Templates tab).

上面一段直接摘抄于官方文档，因为写得已经很具体了

---

# 总结


*  安装插件包 **percona-zabbix-templates**
*  拷贝配置 **userparameter_percona_mysql.conf**
*  配置密码 **ss_get_mysql_stats.php.cnf**
*  安装依赖包 **php php-mysql**
*  给 **zabbix** 赋权
*  从 **zabbix-server** 进行连接测试
*  添加模板


---

# 命令汇总


* **`mysql -V `**
* **`cp /var/lib/zabbix/percona/templates/userparameter_percona_mysql.conf  /etc/zabbix/zabbix_agentd.d/`**
* **`cat /var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php.cnf`**
* **`/var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg `**
* **`yum install php php-mysql `**
* **`cat /tmp/localhost-mysql_cacti_stats.txt`**
* **`cat /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh`**
* **`sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh running-slave `**
* **`sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg`**
* **`chown  zabbix.zabbix /tmp/localhost-mysql_cacti_stats.txt  `**
* **`mkdir /home/zabbix`**
* **`cp /etc/skel/.*  /home/zabbix/`**
* **`chown -R zabbix.zabbix /home/zabbix/`**
* **`vim .my.cnf`**
* **`zabbix_get -s mysql-server -p 10050 -k "MySQL.running-slave" `**
* **`zabbix_get -s mysql-server -p 10050 -k "MySQL.Threads-connected" `**
* **`zabbix_get -s mysql-server -p 10050 -k "MySQL.max-connections" `**

---

[percona]: https://www.percona.com/
[pmp]: https://www.percona.com/doc/percona-monitoring-plugins/1.1/index.html
[pyum]: https://www.percona.com/doc/percona-server/5.6/installation/yum_repo.html
[pmp.installation]: https://www.percona.com/doc/percona-monitoring-plugins/1.1/zabbix/index.html#installation-instructions


