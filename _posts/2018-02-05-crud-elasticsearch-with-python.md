---
layout: post
title: "CRUD Elasticsearch with Python"
author:  wilmosfang
date: 2018-02-05 14:44:27
image: '/assets/img/'
excerpt: '使用 Python 对 ES 进行增删改查'
main-class: python
color: '#265277'
tags:
 - python
 - es
categories:
 - es
twitter_text: 'simple CRUD of Elasticsearch with Python'
introduction: 'CRUD method of Elasticsearch with Python'
---



# 前言

**[Elasticsearch][elasticsearch]** 使用 restful API 来进行数据操作

Python 调用 Elasticsearch API 可以用来简化这个过程

这里分享一下 **[Python Elasticsearch Client][elasticsearch_py]** 简单的 **CRUD** API

参考 **[API Documentation][elasticsearch_api]**

> **Tip:** 当前版本 **elasticsearch (6.1.1)**

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



## 安装 elasticsearch 包

~~~
[root@much ~]# pip list  | grep -i elast
DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a format=(legacy|columns) in your pip.conf under the [list] section) to disable this warning.
[root@much ~]#
[root@much ~]# pip install  elasticsearch
Collecting elasticsearch
  Using cached elasticsearch-6.1.1-py2.py3-none-any.whl
Requirement already satisfied: urllib3<1.23,>=1.21.1 in /usr/lib/python2.7/site-packages (from elasticsearch)
Installing collected packages: elasticsearch
Successfully installed elasticsearch-6.1.1
[root@much ~]# pip list  | grep -i elast
DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a format=(legacy|columns) in your pip.conf under the [list] section) to disable this warning.
elasticsearch (6.1.1)
[root@much ~]#
~~~


## 连接 ES

~~~
[root@much ~]# ipython
Python 2.7.5 (default, Nov  6 2016, 00:28:07)
Type "copyright", "credits" or "license" for more information.

IPython 3.2.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]: from elasticsearch import Elasticsearch

In [2]: es=Elasticsearch('http://elastic:rlziMTdf-+cFW4mN0&pO@localhost:9200/')

In [3]: es
Out[3]: <Elasticsearch([{u'host': 'localhost', u'http_auth': u'elastic:rlziMTdf-+cFW4mN0&pO', u'port': 9200}])>

In [4]:
~~~

使用 **`Elasticsearch('http|https://user:secret@host:port/')`** 连接 ES



## 获取文档

~~~
In [4]: es.get(index='i1',doc_type='t1',id='2')
Out[4]:
{u'_id': u'2',
 u'_index': u'i1',
 u'_source': {u'a': u'a', u'b': u'b', u'c': u'c'},
 u'_type': u't1',
 u'_version': 1,
 u'found': True}

In [5]:
~~~

**`get`** 用来获取内容

**`index`** 指定索引

**`doc_type`** 指定类型

**`id`** 指定 id

如果只想获取文档数据可以使用

~~~
In [30]: es.create(index='test',doc_type='doc',id='1',body=date)
Out[30]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 6,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 1,
 u'result': u'created'}

In [31]: es.get_source(index='test',doc_type='doc',id='1')
Out[31]: {u'1': u'1', u'2': u'2', u'3': u'3'}

In [32]: s
~~~

**`get_source`** 用来获取文档内容



## 创建文档

~~~
In [5]: date={'1':'1','2':'2','3':'3'}

In [6]: date
Out[6]: {'1': '1', '2': '2', '3': '3'}

In [7]: es.create(index='test',doc_type='doc',id='1',body=date)
Out[7]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 0,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 1,
 u'result': u'created'}

In [8]: es.get(index='test',doc_type='doc',id='1')
Out[8]:
{u'_id': u'1',
 u'_index': u'test',
 u'_source': {u'1': u'1', u'2': u'2', u'3': u'3'},
 u'_type': u'doc',
 u'_version': 1,
 u'found': True}

In [9]:
~~~

**`create`** 用来创建文档

**`body`** 用来指定文档内容


## 更新文档

~~~
In [16]: es.get(index='test',doc_type='doc',id='1')
Out[16]:
{u'_id': u'1',
 u'_index': u'test',
 u'_source': {u'1': u'1', u'2': u'2', u'3': u'3'},
 u'_type': u'doc',
 u'_version': 1,
 u'found': True}

In [17]: script={"script":"ctx._source.new_field = '4'"}

In [18]: es.update(index='test',doc_type='doc',id='1',body=script)
Out[18]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 1,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 2,
 u'result': u'updated'}

In [19]: es.get(index='test',doc_type='doc',id='1')
Out[19]:
{u'_id': u'1',
 u'_index': u'test',
 u'_source': {u'1': u'1', u'2': u'2', u'3': u'3', u'new_field': u'4'},
 u'_type': u'doc',
 u'_version': 2,
 u'found': True}

In [20]: script2={"script":"ctx._source.remove('1')"}

In [21]: es.update(index='test',doc_type='doc',id='1',body=script2)
Out[21]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 2,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 3,
 u'result': u'updated'}

In [22]: es.get(index='test',doc_type='doc',id='1')
Out[22]:
{u'_id': u'1',
 u'_index': u'test',
 u'_source': {u'2': u'2', u'3': u'3', u'new_field': u'4'},
 u'_type': u'doc',
 u'_version': 3,
 u'found': True}

In [23]:
~~~

**`update`** 用来更新文档

**`body`** 用来脚本，脚本也要处理成 JSON 格式


## 删除文档

~~~
In [23]: es.delete(index='test',doc_type='doc',id='1')
Out[23]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 3,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 4,
 u'result': u'deleted'}

In [24]: es.get(index='test',doc_type='doc',id='1')
---------------------------------------------------------------------------
NotFoundError                             Traceback (most recent call last)
<ipython-input-24-f0cc6988b370> in <module>()
----> 1 es.get(index='test',doc_type='doc',id='1')

/usr/lib/python2.7/site-packages/elasticsearch/client/utils.pyc in _wrapped(*args, **kwargs)
     74                 if p in kwargs:
     75                     params[p] = kwargs.pop(p)
---> 76             return func(*args, params=params, **kwargs)
     77         return _wrapped
     78     return _wrapper

/usr/lib/python2.7/site-packages/elasticsearch/client/__init__.pyc in get(self, index, doc_type, id, params)
    409                 raise ValueError("Empty value passed for a required argument.")
    410         return self.transport.perform_request('GET', _make_path(index,
--> 411             doc_type, id), params=params)
    412
    413     @query_params('_source', '_source_exclude', '_source_include', 'parent',

/usr/lib/python2.7/site-packages/elasticsearch/transport.pyc in perform_request(self, method, url, headers, params, body)
    312
    313             try:
--> 314                 status, headers_response, data = connection.perform_request(method, url, params, body, headers=headers, ignore=ignore, timeout=timeout)
    315
    316             except TransportError as e:

/usr/lib/python2.7/site-packages/elasticsearch/connection/http_urllib3.pyc in perform_request(self, method, url, params, body, timeout, ignore, headers)
    161         if not (200 <= response.status < 300) and response.status not in ignore:
    162             self.log_request_fail(method, full_url, url, body, duration, response.status, raw_data)
--> 163             self._raise_error(response.status, raw_data)
    164
    165         self.log_request_success(method, full_url, url, body, response.status,

/usr/lib/python2.7/site-packages/elasticsearch/connection/base.pyc in _raise_error(self, status_code, raw_data)
    123             logger.warning('Undecodable raw error response from server: %s', err)
    124
--> 125         raise HTTP_EXCEPTIONS.get(status_code, TransportError)(status_code, error_message, additional_info)
    126
    127

NotFoundError: TransportError(404, u'{"_index":"test","_type":"doc","_id":"1","found":false}')

In [25]:
In [25]: es.create(index='test',doc_type='doc',id='1',body=date)
Out[25]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 4,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 1,
 u'result': u'created'}

In [26]: es.get(index='test',doc_type='doc',id='1')
Out[26]:
{u'_id': u'1',
 u'_index': u'test',
 u'_source': {u'1': u'1', u'2': u'2', u'3': u'3'},
 u'_type': u'doc',
 u'_version': 1,
 u'found': True}

In [27]: es.delete(index='test',doc_type='doc',id='1')
Out[27]:
{u'_id': u'1',
 u'_index': u'test',
 u'_primary_term': 1,
 u'_seq_no': 5,
 u'_shards': {u'failed': 0, u'successful': 1, u'total': 2},
 u'_type': u'doc',
 u'_version': 2,
 u'result': u'deleted'}

In [28]: es.get(index='test',doc_type='doc',id='1')
---------------------------------------------------------------------------
NotFoundError                             Traceback (most recent call last)
<ipython-input-28-f0cc6988b370> in <module>()
----> 1 es.get(index='test',doc_type='doc',id='1')

/usr/lib/python2.7/site-packages/elasticsearch/client/utils.pyc in _wrapped(*args, **kwargs)
     74                 if p in kwargs:
     75                     params[p] = kwargs.pop(p)
---> 76             return func(*args, params=params, **kwargs)
     77         return _wrapped
     78     return _wrapper

/usr/lib/python2.7/site-packages/elasticsearch/client/__init__.pyc in get(self, index, doc_type, id, params)
    409                 raise ValueError("Empty value passed for a required argument.")
    410         return self.transport.perform_request('GET', _make_path(index,
--> 411             doc_type, id), params=params)
    412
    413     @query_params('_source', '_source_exclude', '_source_include', 'parent',

/usr/lib/python2.7/site-packages/elasticsearch/transport.pyc in perform_request(self, method, url, headers, params, body)
    312
    313             try:
--> 314                 status, headers_response, data = connection.perform_request(method, url, params, body, headers=headers, ignore=ignore, timeout=timeout)
    315
    316             except TransportError as e:

/usr/lib/python2.7/site-packages/elasticsearch/connection/http_urllib3.pyc in perform_request(self, method, url, params, body, timeout, ignore, headers)
    161         if not (200 <= response.status < 300) and response.status not in ignore:
    162             self.log_request_fail(method, full_url, url, body, duration, response.status, raw_data)
--> 163             self._raise_error(response.status, raw_data)
    164
    165         self.log_request_success(method, full_url, url, body, response.status,

/usr/lib/python2.7/site-packages/elasticsearch/connection/base.pyc in _raise_error(self, status_code, raw_data)
    123             logger.warning('Undecodable raw error response from server: %s', err)
    124
--> 125         raise HTTP_EXCEPTIONS.get(status_code, TransportError)(status_code, error_message, additional_info)
    126
    127

NotFoundError: TransportError(404, u'{"_index":"test","_type":"doc","_id":"1","found":false}')

In [29]:
~~~

**`delete`** 用来删除文档

---

# 总结

对于单个文档的 **CRUD** 通过 Elasticsearch 的 Python client API 可以很方便地完成

加入其它逻辑就可以很方便地实现更复杂的功能

* TOC
{:toc}


---

[elasticsearch]:https://www.elastic.co/products/elasticsearch
[elasticsearch_py]:https://elasticsearch-py.readthedocs.io/en/master/
[elasticsearch_api]:https://elasticsearch-py.readthedocs.io/en/master/api.html
