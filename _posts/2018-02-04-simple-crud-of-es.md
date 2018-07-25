---
layout: post
title: "simple CRUD of Elasticsearch"
author:  wilmosfang
date: 2018-02-04 15:19:29
image: '/assets/img/'
excerpt: 'ES 的简单增删改查'
main-class: es
color: '#51bcb2'
tags:
 - curl
 - es
categories:
 - es
twitter_text: 'simple CRUD of Elasticsearch'
introduction: 'CRUD method of Elasticsearch'
---


# 前言

**[Elasticsearch][elasticsearch]** 使用 restful API 来进行数据操作

这里分享一下 **[Elasticsearch][elasticsearch]** 简单的 **CRUD** API

参考 **[Document APIs][elasticsearch_crud]**

> **Tip:** 当前版本 **Version:6.1.3**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 71a5a14bde634bfc8c5bafb7d9442f9e
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d1:5d:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85051sec preferred_lft 85051sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:47:20:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 获取 indices

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  http://localhost:9200/_cat/indices?pretty=true
yellow open .watcher-history-7-2018.02.04   c7SPfW83R5u_hLZURXU3sA 1 1   959  0   1.8mb   1.8mb
yellow open .watches                        R-OLTmNVRzqWE6njMyvUzA 1 1     5  0  47.1kb  47.1kb
yellow open .monitoring-es-6-2018.02.04     1iFJjxoCSC2nyfnp0onL9A 1 1 10701 25     8mb     8mb
yellow open abcdjustfortest                 IdqYj5EiTdqHCu6BiA7huQ 5 1    21  0    43kb    43kb
yellow open .monitoring-alerts-6            8D5NlTCxQPa4ULGr7mhKow 1 1     1  0  12.1kb  12.1kb
yellow open .triggered_watches              qJawGoExRdOePEWXUq_mjg 1 1     0  0    43kb    43kb
green  open .security-6                     SypjlJsUTMezjPvwDffRgQ 1 0     3  0   9.9kb   9.9kb
yellow open .monitoring-kibana-6-2018.02.04 buqe4grJToeD_dbmL-cp1Q 1 1   588  0 421.7kb 421.7kb
yellow open .kibana                         PzWyv9kFRBi9stsDJSBskQ 1 1     2  0     7kb     7kb
[root@much ~]#
~~~

* **`-u 'elastic':'rlziMTdf-+cFW4mN0&pO'`** 是在指定用户密码，如果ES进行了密码认证而不指定用户密码,会有如下报错

~~~
[root@much ~]# curl http://localhost:9200/_cat/indices?pretty=true
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "missing authentication token for REST request [/_cat/indices?pretty=true]",
        "header" : {
          "WWW-Authenticate" : "Basic realm=\"security\" charset=\"UTF-8\""
        }
      }
    ],
    "type" : "security_exception",
    "reason" : "missing authentication token for REST request [/_cat/indices?pretty=true]",
    "header" : {
      "WWW-Authenticate" : "Basic realm=\"security\" charset=\"UTF-8\""
    }
  },
  "status" : 401
}
[root@much ~]#
~~~


## 增加文档

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XPUT http://localhost:9200/i1/t1/1?pretty=true  -H 'Content-Type: application/json' -d '{"a":"a","b":"b","c":"c"}'
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
[root@much ~]#
~~~

**`i1`** 为 index 名，不存在就会自动创建

**`t1`** 为 type 名，不存在就会自动创建

**`1`** 为 id 号，不指定会随机分配，已经存在就会更新

**`?`** 用来分隔 API 路径和参数

**`pretty=true`** 为参数，以便于阅读的格式输出

使用 **`PUT`** 的方法添加数据

将 json 数据交给 **`-d`** 参数

这里必须指定内容类型 **`Content-Type`** 为 **`application/json`**, 否则默认的类型可能会报错

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XPUT http://localhost:9200/i1/t1/1?pretty=true  -d
'{"a":"a","b":"b","c":"c"}'
{
  "error" : "Content-Type header [application/x-www-form-urlencoded] is not supported",
  "status" : 406
}
[root@much ~]#
~~~

## 查询文档

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XPUT http://localhost:9200/i1/t1/2?pretty=true  -H 'Content-Type: application/json' -d '{"a":"a","b":"b","c":"c"}'
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "2",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XGET http://localhost:9200/i1/t1/2?pretty=true
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "2",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "a" : "a",
    "b" : "b",
    "c" : "c"
  }
}
[root@much ~]#
~~~

使用 **`GET`** 来查询指定文档内容

一个文档由 **`index/type/id`** 来唯一标识


## 更新文档


对一个已经存在的文档进行再次创建就是一个简单的更新操作

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XPUT http://localhost:9200/i1/t1/1?pretty=true  -H 'Content-Type: application/json' -d '{"a":"a","b":"b","c":"c"}'
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
[root@much ~]#
~~~

可见版本已经变成了版本2, result 为 **updated**


~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XPUT http://localhost:9200/i1/t1/1?pretty=true  -H 'Content-Type: application/json' -d '{"a":"a","b":"b","c":"cx"}'
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XGET http://localhost:9200/i1/t1/1?pretty=true {
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 3,
  "found" : true,
  "_source" : {
    "a" : "a",
    "b" : "b",
    "c" : "cx"
  }
}
[root@much ~]#
~~~

同时修改内容，可现已经发生变更

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XPUT http://localhost:9200/i1/t1/1?pretty=true  -H 'Content-Type: application/json' -d '{"b":"bxx","c":"cx"}'
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XGET http://localhost:9200/i1/t1/1?pretty=true {
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 4,
  "found" : true,
  "_source" : {
    "b" : "bxx",
    "c" : "cx"
  }
}
[root@much ~]#
~~~

## 删除文档

~~~
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XDELETE http://localhost:9200/i1/t1/1?pretty=true
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "_version" : 5,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
[root@much ~]# curl -u 'elastic':'rlziMTdf-+cFW4mN0&pO'  -XGET http://localhost:9200/i1/t1/1?pretty=true
{
  "_index" : "i1",
  "_type" : "t1",
  "_id" : "1",
  "found" : false
}
[root@much ~]#
~~~

使用 **`DELETE`** 方法来删除指定文档


---

# 总结

对于单个文档的 **CRUD** 通过 HTTP 的 **`PUT/GET/DELETE`** 就可以完成


* TOC
{:toc}


---

[elasticsearch]:https://www.elastic.co/products/elasticsearch
[elasticsearch_crud]:https://www.elastic.co/guide/en/elasticsearch/reference/6.1/docs.html
