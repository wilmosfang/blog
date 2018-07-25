---
layout: post
title: "ETL from CSV to Elasticsearch"
author:  wilmosfang
date: 2018-02-09 15:09:05
image: '/assets/img/'
excerpt: '从 CSV 导数据到 Elasticsearch'
main-class: python
color: '#265277'
tags:
 - python
 - es
 - etl
 - pip
categories:
 - python
twitter_text: 'import data from CSV to Elasticsearch'
introduction: 'python ETL from CSV to Elasticsearch'
---



## 前言

当有大量数据要从 CSV 导入到 Elasticsearch 中时一般有两种方式来完成

* 1.使用 logstash 加上 csv filter 的方式来导入
* 2.编写脚本来完成

对于第一种方式，只要定义好字段名，指定输入源文件，相对简单，但定制空间比较受 logstash 的功能约束

对于第二种方式，相对灵活，但是更复杂一点，需要借助各种库 API，也要理清数据抽取，变换处理与导入的逻辑流程

这里演示一下如何傅用 python 来将 CSV 导出到 Elasticsearch

> **Tip:** 需要借助 **Elasticsearch** 的 python 客户端

---

# 操作


## 环境

~~~
[root@much sf_script]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: a6eba448fd814d6dad2f7cb92465f567
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much sf_script]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 80921sec preferred_lft 80921sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much sf_script]# python -V
Python 2.7.5
[root@much sf_script]# rpm -qa | grep elast
elasticsearch-6.2.1-1.noarch
[root@much sf_script]# rpm -qa | grep kibana
kibana-6.2.1-1.x86_64
[root@much sf_script]#
~~~


>**Note:** Kibana 的版本要与 elasticsearch 版本兼容，否则会报错

## 安装 pip

~~~
[root@much ~]# yum install  python2-pip.noarch
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package python2-pip.noarch 0:8.1.2-5.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package              Arch            Version               Repository     Size
================================================================================
Installing:
 python2-pip          noarch          8.1.2-5.el7           epel          1.7 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 1.7 M
Installed size: 7.2 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/epel/packages/python2-pip-8.1.2-5.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for python2-pip-8.1.2-5.el7.noarch.rpm is not installed
python2-pip-8.1.2-5.el7.noarch.rpm                         | 1.7 MB   00:05     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:
ipa-client-4.4.0-14.el7.centos.7.x86_64 has installed conflicts freeipa-client: ipa-client-4.4.0-14.el7.centos.7.x86_64
ipa-client-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-client-common: ipa-client-common-4.4.0-14.el7.centos.7.noarch
ipa-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-common: ipa-common-4.4.0-14.el7.centos.7.noarch
  Installing : python2-pip-8.1.2-5.el7.noarch                               1/1
  Verifying  : python2-pip-8.1.2-5.el7.noarch                               1/1

Installed:
  python2-pip.noarch 0:8.1.2-5.el7                                              

Complete!
[root@much ~]#
~~~



## 安装 Elasticsearch Client API

~~~
[root@much ~]# pip install elasticsearch
Collecting elasticsearch
  Downloading elasticsearch-6.1.1-py2.py3-none-any.whl (59kB)
    100% |████████████████████████████████| 61kB 326kB/s
Collecting urllib3<1.23,>=1.21.1 (from elasticsearch)
  Downloading urllib3-1.22-py2.py3-none-any.whl (132kB)
    100% |████████████████████████████████| 133kB 347kB/s
Installing collected packages: urllib3, elasticsearch
  Found existing installation: urllib3 1.10.2
    Uninstalling urllib3-1.10.2:
      Successfully uninstalled urllib3-1.10.2
Successfully installed elasticsearch-6.1.1 urllib3-1.22
You are using pip version 8.1.2, however version 9.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
[root@much ~]#
~~~

有提示安装新版本的 pip

~~~
[root@much ~]# pip install --upgrade pip
Collecting pip
  Downloading pip-9.0.1-py2.py3-none-any.whl (1.3MB)
    100% |████████████████████████████████| 1.3MB 273kB/s
Installing collected packages: pip
  Found existing installation: pip 8.1.2
    Uninstalling pip-8.1.2:
      Successfully uninstalled pip-8.1.2
Successfully installed pip-9.0.1
[root@much ~]# pip -V
pip 9.0.1 from /usr/lib/python2.7/site-packages (python 2.7)
[root@much ~]#
~~~


## 查看当前索引

~~~
[root@much sf_script]# curl http://localhost:9200/_cat/indices?pretty=true
green open .kibana FEw09koKTymzBRmFlyCThA 1 0 4 0 20kb 20kb
[root@much sf_script]#
~~~

## 编写脚本

~~~
[root@much sf_script]# vim csv2es.py
[root@much sf_script]# cat csv2es.py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

#import
import csv
import sys
import os
from optparse import OptionParser
from elasticsearch import Elasticsearch
from elasticsearch import helpers

reload(sys)
sys.setdefaultencoding('utf-8')

#args parser
usage =  "Usage: %prog <-i index> <-t type> [options] arg"
parser = OptionParser(usage)
parser.add_option("-i","--index",dest="index",help="(mandatory)the index ready to import")
parser.add_option("-t","--type",dest="dtype",help="(mandatory)the type ready to import")
parser.add_option("-f","--csv",dest="csv",help="(mandatory)the csv file ready to import")
parser.add_option("-s","--server",dest="server",default="localhost",help="the Elasticsearch host, default is %default")
parser.add_option("-P","--port",dest="port",default="9200",help="the Elasticsearch port, default is %default")
parser.add_option("-u","--user",dest="user",help="the user of elasticsearch")
parser.add_option("-p","--password",dest="password",help="the password of the elasticsearch user")
(options,args)=parser.parse_args()

#check args
if options.csv == None:
    exit("Error: you need to specified target csv file tobe import")
if not os.path.exists(options.csv):
    exit("Error: %s not found"%options.csv)
if options.index == None:
    exit("Error: you need to specified target index tobe import")
if options.dtype == None:
    exit("Error: you need to specified target type tobe import")
if options.user == None or options.password == None:
    es_url = 'http://' + options.server + ':' + options.port + '/'
else:
    es_url = 'http://' + options.user + ':' + options.password + '@' + options.server + ':' + options.port + '/'


es = Elasticsearch(es_url)

def to_utf8(record):
    for i in record:
        record[i]=str(record[i]).encode('utf-8')
    return record

def etl_csv_to_es(indexName,typeName,csvFile):
    actions = []
    count = 0
    for row in csv.DictReader(open(csvFile,'rb')):
        action = {"_index":indexName,"_type":typeName,"_source":to_utf8(row)}
        actions.append(action)
        count += 1
    helpers.bulk(es,actions)
    es.indices.flush(index=[indexName])
    return (True,count)

#main
if __name__ == "__main__":
    (res,num) = etl_csv_to_es(options.index,options.dtype,options.csv)
    if res:
        print "%d items import secussfully"%num
    else:
        print "import fail"
[root@much sf_script]#
~~~

## 准备测试数据

~~~
[root@much sf_script]# vim y.csv
[root@much sf_script]# cat y.csv
x,y,z,p,q
2018/01/02 23:44:23,a,b,c,d
2018/01/02 23:44:24,a,b,c,d
2018/01/02 23:44:25,a,b,c,d
2018/01/02 23:44:26,a,b,c,d
2018/01/02 23:44:27,a,b,c,d
2018/01/02 23:44:28,a,b,c,d
2018/01/02 23:44:29,a,b,c,d
2018/01/02 23:44:30,a,b,c,d
2018/01/02 23:44:31,a,b,c,d
2018/01/02 23:44:32,a,b,c,d
2018/01/02 23:44:33,a,b,c,d
2018/01/02 23:44:34,a,b,c,d
2018/01/02 23:44:35,a,b,c,d
2018/01/02 23:44:36,a,b,c,d
2018/01/02 23:44:37,a,b,c,d
2018/01/02 23:44:38,a,b,c,d
2018/01/02 23:44:39,a,b,c,d
2018/01/02 23:44:40,a,b,c,d
2018/01/02 23:44:41,a,b,c,d
2018/01/02 23:44:42,a,b,c,d
2018/01/02 23:44:43,a,b,c,d
2018/01/02 23:44:44,a,b,c,d
2018/01/02 23:44:45,a,b,c,d
2018/01/02 23:44:46,a,b,c,d
2018/01/02 23:44:47,a,b,c,d
2018/01/02 23:44:48,a,b,c,d
2018/01/02 23:44:49,a,b,c,d
2018/01/02 23:44:50,a,b,c,d
2018/01/02 23:44:51,a,b,c,d
2018/01/02 23:44:52,a,b,c,d
2018/01/02 23:45:23,a,b,c,d
2018/01/02 23:45:24,a,b,c,d
2018/01/02 23:45:25,a,b,c,d
2018/01/02 23:45:26,a,b,c,d
2018/01/02 23:45:27,a,b,c,d
2018/01/02 23:45:28,a,b,c,d
2018/01/02 23:45:29,a,b,c,d
2018/01/02 23:45:30,a,b,c,d
2018/01/02 23:45:31,a,b,c,d
2018/01/02 23:45:32,a,b,c,d
2018/01/02 23:45:33,a,b,c,d
2018/01/02 23:45:34,a,b,c,d
2018/01/02 23:45:35,a,b,c,d
2018/01/02 23:45:36,a,b,c,d
2018/01/02 23:45:37,a,b,c,d
2018/01/02 23:45:38,a,b,c,d
2018/01/02 23:45:39,a,b,c,d
2018/01/02 23:45:40,a,b,c,d
2018/01/02 23:45:41,a,b,c,d
[root@much sf_script]#
~~~

## 运行脚本

~~~
[root@much sf_script]# ./csv2es.py -h
Usage: csv2es.py <-i index> <-t type> [options] arg

Options:
  -h, --help            show this help message and exit
  -i INDEX, --index=INDEX
                        (mandatory)the index ready to import
  -t DTYPE, --type=DTYPE
                        (mandatory)the type ready to import
  -f CSV, --csv=CSV     (mandatory)the csv file ready to import
  -s SERVER, --server=SERVER
                        the Elasticsearch host, default is localhost
  -P PORT, --port=PORT  the Elasticsearch port, default is 9200
  -u USER, --user=USER  the user of elasticsearch
  -p PASSWORD, --password=PASSWORD
                        the password of the elasticsearch user
[root@much sf_script]# ./csv2es.py -i indextest -t typetest -f y.csv
49 items import secussfully
[root@much sf_script]#
~~~

可以在命令行中进行验证

~~~
[root@much sf_script]# curl http://localhost:9200/_cat/indices?pretty=true
yellow open indextest NiccLPLgStO7o6YjofKS-g 5 1 49 0 29kb 29kb
green  open .kibana   FEw09koKTymzBRmFlyCThA 1 0  4 0 20kb 20kb
[root@much sf_script]#
~~~

## 从 kibana 中查看数据


![kibana](/assets/img/kibana/kibana14.png)


---

# 总结

相对于使用 logstash 此脚本可以不用操心列名的问题，因为它会自动将表头与内容处理成哈希(字典)，只要确保表头与此列是对应关系，列的数量变化都是兼容的，(logstash 需要针对不同的数据源，处理 filter csv 插件中的列名)


* TOC
{:toc}


---
