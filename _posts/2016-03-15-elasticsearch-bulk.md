---
layout: post
title:  Elasticsearch 批量导入数据
author: wilmosfang
categories:   elasticsearch
tags:   nosql elasticsearch
wc: 295   744 11369 
excerpt: elasticsearch 批量导入数据，bulk API的相关基础，数据内容格式，可用方法，导入操作，注意事项，内存问题处理 
comments: true
---



# 前言

**[Elasticsearch][elasticsearch]** 是一款非常高效的全文检索引擎。

**[Elasticsearch][elasticsearch]** 可以非常方便地进行数据的多维分析，所以大数据分析领域也经常会见到它的身影，生产环境中绝大部分新产生的数据可以通过应用直接导入，但是历史或初始数据可能会需要单独处理，这种情况下可能遇到需要导入大量数据的情况

这里简单分享一下批量导入数据的操作方法与相关基础，还有可能会碰到的问题，详细内容可以参考 **[官方文档][docs_bulk]** 


> **Tip:** 当前的最新版本为 **Elasticsearch 2.2.0** 

---


# 概要

* TOC
{:toc}


---

## bulk API

ES提供了一个叫 **bulk** 的 **API** 来进行批量操作

它用来在一个API调用中进行大量的索引更新或删除操作，这极大的提升了操作效率


---

## 形式


### API

API 可以是 **`/_bulk, /{index}/_bulk, 或 {index}/{type}/_bulk`** 这三种形式，当索引或类型已经指定后，数据文件中如不明确指定或申明的内容，就会默认使用API中的值

API 以是 **`/_bulk`** 结尾的，并且跟上如下形式的 **JSON** 数据


### 数据内容格式

~~~
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
~~~

> **Note:** 最后的一行也必须以 **`\n`** 结尾

### 可用方法

可用的操作有 **`index, create, delete 和 update`** ：

* **index** 和 **create** 得在操作与元数据(action_and_meta_data)之后另起一行然后接上内容(必须遵循这样的格式 ，后面会演示不这么做导致操作失败的示例)
* **delete** 只用接上元数据就可以了，不必接上内容(原因自不用说，定位到文档就OK了)
* **update** 得接上要变更的局部数据，也得另起一行

### 文本指定

由于是批量操作，所以不太会直接使用命令行的方式手动指定，更多的是使用文件，如果使用文本文件，则得遵循如下格式

~~~
curl -s -XPOST localhost:9200/_bulk --data-binary "@requests"
~~~

> **Tip:**  **requests** 是文件名 , **`-s`** 是静默模式，不产生输出，也可以使用 **`> /dev/null`** 替代


---

## 导入数据

### 尝试不按要求索引数据

~~~
[root@es-bulk tmp]# curl localhost:9200/stuff_orders/order_list/903713?pretty
{
  "_index" : "stuff_orders",
  "_type" : "order_list",
  "_id" : "903713",
  "found" : false
}
[root@es-bulk tmp]# cat test.json 
{"index":{"_index":"stuff_orders","_type":"order_list","_id":903713}}{"real_name":"刘备","user_id":48430,"address_province":"上海","address_city":"浦东新区","address_district":null,"address_street":"上海市浦东新区广兰路1弄2号345室","price":30.0,"carriage":6.0,"state":"canceled","created_at":"2013-10-24T09:09:28.000Z","payed_at":null,"goods":["营养早餐：火腿麦满分"],"position":[121.53,31.22],"weight":70.0,"height":172.0,"sex_type":"female","birthday":"1988-01-01"}
[root@es-bulk tmp]# curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @test.json
{
  "error" : {
    "root_cause" : [ {
      "type" : "action_request_validation_exception",
      "reason" : "Validation Failed: 1: no requests added;"
    } ],
    "type" : "action_request_validation_exception",
    "reason" : "Validation Failed: 1: no requests added;"
  },
  "status" : 400
}
[root@es-bulk tmp]# curl localhost:9200/stuff_orders/order_list/903713?pretty
{
  "_index" : "stuff_orders",
  "_type" : "order_list",
  "_id" : "903713",
  "found" : false
}
[root@es-bulk tmp]#
~~~

产生了报错，并且数据也的确没有加成功，原因是在校验操作请求(**`action_and_meta_data`**)时，由于不符合规范，所以报异常

### 正确导入方法

解决办法是将格式纠正过来，加上换行

~~~
[root@es-bulk tmp]# vim test.json 
[root@es-bulk tmp]# cat test.json 
{"index":{"_index":"stuff_orders","_type":"order_list","_id":903713}}
{"real_name":"刘备","user_id":48430,"address_province":"上海","address_city":"浦东新区","address_district":null,"address_street":"上海市浦东新区广兰路1弄2号345室","price":30.0,"carriage":6.0,"state":"canceled","created_at":"2013-10-24T09:09:28.000Z","payed_at":null,"goods":["营养早餐：火腿麦满分"],"position":[121.53,31.22],"weight":70.0,"height":172.0,"sex_type":"female","birthday":"1988-01-01"}
[root@es-bulk tmp]# curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @test.json
{
  "took" : 36,
  "errors" : false,
  "items" : [ {
    "index" : {
      "_index" : "stuff_orders",
      "_type" : "order_list",
      "_id" : "903713",
      "_version" : 1,
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "status" : 201
    }
  } ]
}
[root@es-bulk tmp]# curl localhost:9200/stuff_orders/order_list/903713?pretty
{
  "_index" : "stuff_orders",
  "_type" : "order_list",
  "_id" : "903713",
  "_version" : 1,
  "found" : true,
  "_source":{"real_name":"刘备","user_id":48430,"address_province":"上海","address_city":"浦东新区","address_district":null,"address_street":"上海市浦东新区广兰路1弄2号345室","price":30.0,"carriage":6.0,"state":"canceled","created_at":"2013-10-24T09:09:28.000Z","payed_at":null,"goods":["营养早餐：火腿麦满分"],"position":[121.53,31.22],"weight":70.0,"height":172.0,"sex_type":"female","birthday":"1988-01-01"}
}
[root@es-bulk tmp]# 
~~~

> **Tip:** 当数据量极大时，这样一个个改肯定不方便，这时可以使用sed脚本，能很方便的进行批量修改


~~~
[root@es-bulk summary]# sed -ir  's/[}][}][{]/\}\}\n\{/' jjjj.json 
[root@es-bulk summary]# less jjjj.json
~~~

其实就是匹配到合适的地方加上一个换行

---

## 内存不足

基本上只要遵循前面的操作方式，理想情况下都会很顺利地将数据导入ES，但是实现环境中，总会有各种意外，我就遇到了其中一种：内存不足


~~~
[root@es-bulk tmp]# time curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @es_data.json > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
 38  265M    0     0   38  102M      0  43.8M  0:00:06  0:00:02  0:00:04 43.9M
curl: (56) Failure when receiving data from the peer

real	0m5.351s
user	0m0.161s
sys	0m0.919s
[root@es-bulk tmp]#
~~~

当时百思不得其解，已经反复确认了数据格式无误，并且随机选取其中一些进行导入测试也没发现问题，但只要整体一导就出问题，而且每次都一样


~~~
[root@es-bulk tmp]# free -m 
             total       used       free     shared    buffers     cached
Mem:          3949       3548        400          0          1        196
-/+ buffers/cache:       3349        599
Swap:         3951        237       3714
[root@es-bulk tmp]#
~~~

系统内存明明还有多余，但是再看到JAVA内存时，就隐约感觉到了原因

~~~
[root@es-bulk tmp]# ps faux | grep elas
root     14479  0.0  0.0 103252   816 pts/1    S+   16:05   0:00          \_ grep elas
495      19045  0.2 25.6 3646816 1036220 ?     Sl   Mar07  25:45 /usr/bin/java -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-2.1.1.jar:/usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch start -p /var/run/elasticsearch/elasticsearch.pid -d -Des.default.path.home=/usr/share/elasticsearch -Des.default.path.logs=/var/log/elasticsearch -Des.default.path.data=/var/lib/elasticsearch -Des.default.path.conf=/etc/elasticsearch
[root@es-bulk tmp]#
~~~

ES和lucene是使用的JAVA，JAVA的内存分配大小决定了它们的发挥空间，这里的初始内存为 **256M** ，这也是大多数情况下的默认配置，但是应对当前的实际数据大小 **265M** 时就不够了，虽然官方说会尽量减小使用buffer，但实测下来，系统应该会是首先尽量使用内存，通过导入内存的方式来起到显著加速的效果，但是内存不够时，就直接报错退出了


解决内存不足有两种思路：

* 1.调整 **Xms** 和 **Xmx** 参数，使其适应业务需求，然后重启服务使之生效
* 2.将原来的数据切小，分批导入

第一种方式，要求停应用和业务，在某些情况下是不具备条件的(得统一协调时间窗口)，那么就尝试使用第二种方式,好在text文档的切分也可以使用sed快速完成

~~~
[root@es-bulk tmp]# sed  -rn '1,250000p' es_data.json  > es_data1.json
[root@es-bulk tmp]# sed  -rn '250001,500000p' es_data.json  > es_data2.json
[root@es-bulk tmp]# sed  -rn '500001,750000p' es_data.json  > es_data3.json
[root@es-bulk tmp]# sed  -rn '750001,943210p' es_data.json  > es_data4.json
[root@es-bulk tmp]# 
[root@es-bulk tmp]# du -sh es_data*.json
71M	es_data1.json
68M	es_data2.json
71M	es_data3.json
58M	es_data4.json
266M	es_data.json
[root@es-bulk tmp]#
[root@es-bulk tmp]# tail es_data1.json
...
...
[root@es-bulk tmp]# tail es_data2.json
...
...
~~~


再依次进行导入，就发现没问题了

~~~
[root@es-bulk tmp]# time curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @es_data1.json > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  101M  100 30.6M  100 70.3M   981k  2253k  0:00:31  0:00:31 --:--:--     0

real	0m33.308s
user	0m0.100s
sys	0m0.390s
[root@es-bulk tmp]#
~~~


---

# 命令汇总


* **`curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @test.json`**
* **`curl localhost:9200/stuff_orders/order_list/903713?pretty`**
* **`sed -ir  's/[}][}][{]/\}\}\n\{/' jjjj.json`**
* **`less jjjj.json`**
* **`time curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @es_data.json > /dev/null`**
* **`free -m`**
* **`ps faux | grep elas`**
* **`sed  -rn '1,250000p' es_data.json  > es_data1.json`**
* **`sed  -rn '250001,500000p' es_data.json  > es_data2.json`**
* **`sed  -rn '500001,750000p' es_data.json  > es_data3.json`**
* **`sed  -rn '750001,943210p' es_data.json  > es_data4.json`**
* **`du -sh es_data*.json`**
* **`tail es_data1.json`**
* **`time curl -XPOST 'localhost:9200/stuff_orders/_bulk?pretty' --data-binary @es_data1.json > /dev/null`**


---





[elasticsearch]:https://www.elastic.co/
[docs_bulk]:https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html
