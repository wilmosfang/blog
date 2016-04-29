---
layout: post
title:  Elasticsearch Delete By Query Plugin  
author: wilmosfang
categories:  linux nosql elasticsearch
wc: 607  1606 16293 
excerpt: Elasticsearch Delete By Query Plugin 的安装卸载方法，操作注意事项  
comments: true
---



# 前言


**[Elasticsearch][elasticsearch]** 的使用过程中常常要删除具备一定特性的一批数据(documents)

* 传统方法：使用 **`_search`** API搜出来，然后通过脚本处理后使用 **`DELETE`** 方法一个个删除
* 批量操作：使用 **`scroll`** API搜出来，然后通过 **`bulk`** 进行批量删除
* 最便捷方法：使用 **Delete By Query** 方法，直接进行删除

前面两种方法都特别繁琐，很显然最后一种方法最便捷，但问题是 **Delete By Query** API在 **1.5.3** 的版本中因为潜在的安全与性能隐患就已经被废弃了，这里给出了 **[原因][reason]**


> **Delete By Query API**
>
>Deprecated in 1.5.3.
>
>Delete by Query will be removed in 2.0: it is problematic since it silently forces a refresh which can quickly cause OutOfMemoryError during concurrent indexing, and can also cause primary and replica to become inconsistent. Instead, use the scroll/scan API to find all matching ids and then issue a bulk request to delete them..

但好在废除这个API的同时又提供了一个 **delete-by-query plugin** 来解决这个问题

这里在 **ES2.1** 中分享一下 **Delete By Query** 的操作过程，详细可以参阅 **[官方文档][plugins_delete_by_query]** 

> **Tip:** 当前的最新版本为 **Elasticsearch 2.1.1** 

---


# 概要

* TOC
{:toc}


---

## 背景


ES版本为2.1.1

~~~
[root@esdbqp bin]# ps faux | grep elasticsearch |grep -v grep 
492      16600 28.8  1.2 4968568 411068 ?      Sl   23:39   2:17 /usr/bin/java -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-2.1.1.jar:/usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch start -p /var/run/elasticsearch/elasticsearch.pid -d -Des.default.path.home=/usr/share/elasticsearch -Des.default.path.logs=/var/log/elasticsearch -Des.default.path.data=/var/lib/elasticsearch -Des.default.path.conf=/etc/elasticsearch
[root@esdbqp bin]# 
~~~

ES中有一些被标记为 **multiline** 的记录，我想删掉它们

~~~
[root@esdbqp bin]# curl "http://localhost:9200/filebeat-*/_search?pretty=true&q=tags:multiline"
{
  "took" : 44,
  "timed_out" : false,
  "_shards" : {
    "total" : 165,
    "successful" : 165,
    "failed" : 0
  },
  "hits" : {
    "total" : 4214,
    "max_score" : 9.93189,
    "hits" : [ {
      "_index" : "filebeat-2016.01.28",
      "_type" : "log",
      "_id" : "AVKLW2hTFoV7cO-dan6I",
      "_score" : 9.93189,
      ...
      ...
      ...
~~~

当前的插件情况

~~~
[root@esdbqp bin]# /usr/share/elasticsearch/bin/plugin list
Installed plugins in /usr/share/elasticsearch/plugins:
    - No plugin detected
[root@esdbqp bin]# 
~~~

### 报错1

当前尝试使用 **Delete By Query API**

~~~
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-*/log/_query?pretty=true&q=tags:multiline"
{
  "error" : {
    "root_cause" : [ {
      "type" : "invalid_index_name_exception",
      "reason" : "Invalid index name [filebeat-*], must not contain the following characters [\\, /, *, ?, \", <, >, |,  , ,]",
      "index" : "filebeat-*"
    } ],
    "type" : "invalid_index_name_exception",
    "reason" : "Invalid index name [filebeat-*], must not contain the following characters [\\, /, *, ?, \", <, >, |,  , ,]",
    "index" : "filebeat-*"
  },
  "status" : 400
}
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-2016.01.02/log/_query?pretty=true&q=tags:multiline"
{
  "found" : false,
  "_index" : "filebeat-2016.01.02",
  "_type" : "log",
  "_id" : "_query",
  "_version" : 1,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  }
}
[root@esdbqp bin]#
~~~

 说明在 ES2.1.1中的确已经删掉了这个API 
 
> **Tip:**在1.5.3的版本之前，上面的操作是直接可以成功的

---

## 安装插件

~~~
[root@esdbqp bin]#  /usr/share/elasticsearch/bin/plugin list
Installed plugins in /usr/share/elasticsearch/plugins:
    - No plugin detected
[root@esdbqp bin]#  /usr/share/elasticsearch/bin/plugin install delete-by-query
-> Installing delete-by-query...
Trying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/delete-by-query/2.1.1/delete-by-query-2.1.1.zip ...
ERROR: failed to download out of all possible locations..., use --verbose to get detailed information
[root@esdbqp bin]#  /usr/share/elasticsearch/bin/plugin install delete-by-query
-> Installing delete-by-query...
Trying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/delete-by-query/2.1.1/delete-by-query-2.1.1.zip ...
Downloading ..DONE
Verifying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/delete-by-query/2.1.1/delete-by-query-2.1.1.zip checksums if available ...
Downloading .DONE
Installed delete-by-query into /usr/share/elasticsearch/plugins/delete-by-query
[root@esdbqp bin]#  /usr/share/elasticsearch/bin/plugin list
Installed plugins in /usr/share/elasticsearch/plugins:
    - delete-by-query
[root@esdbqp bin]# 
~~~

### 报错2

> **Tip:**  由于网络原因，有时一次可能安装不成功，所以可以多试两次

---

## 重启ES服务

安装后得重启，否则不会生效，如果是一个集群，那每一个节点安装完插件后都得重启

>The plugin must be installed on every node in the cluster, and each node must be restarted after installation.

~~~
[root@esdbqp bin]# /etc/init.d/elasticsearch restart 
Stopping elasticsearch:                                    [  OK  ]
Starting elasticsearch:                                    [  OK  ]
[root@esdbqp bin]#
~~~

> **Note:**  一定得重启


---

## 调用API

刚才启动后，因为插件还没完全加载，如果立即调用 **Delete By Query API** 虽然不会报最前面的错，但是也不会反馈想要的结果


### 报错3

一般会反馈如下结果

~~~
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-*/log/_query?pretty=true&q=tags:multiline"
{
  "error" : {
    "root_cause" : [ ],
    "type" : "search_phase_execution_exception",
    "reason" : "all shards failed",
    "phase" : "query",
    "grouped" : true,
    "failed_shards" : [ ]
  },
  "status" : 503
}
[root@esdbqp bin]#
~~~

稍等过后，再次尝试，就可以进行删除操作

~~~
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-2016.01.28/log/_query?q=tags:multiline"
{"took":200,"timed_out":false,"_indices":{"_all":{"found":110,"deleted":110,"missing":0,"failed":0},"filebeat-2016.01.28:{"found":110,"deleted":110,"missing":0,"failed":0}},"failures":[]}[root@esdbqp bin]# curl -XDELETE "http://localhost:920/filebeat-2016.01.28/log/_query?pretty=true&q=tags:multiline"
{
  "took" : 0,
  "timed_out" : false,
  "_indices" : {
    "_all" : {
      "found" : 0,
      "deleted" : 0,
      "missing" : 0,
      "failed" : 0
    }
  },
  "failures" : [ ]
}
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-2016.01.28/log/_query?pretty=true&q=tags:multiline"
{
  "took" : 0,
  "timed_out" : false,
  "_indices" : {
    "_all" : {
      "found" : 0,
      "deleted" : 0,
      "missing" : 0,
      "failed" : 0
    }
  },
  "failures" : [ ]
}
[root@esdbqp bin]#
~~~



也可以使用如下DSL

~~~
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-2016.01.28/log/_query?pretty=true" -d '
{
 "query": {
   "term": {
      "tags": "multiline"
    }
  }
}'
{
  "took" : 0,
  "timed_out" : false,
  "_indices" : {
    "_all" : {
      "found" : 0,
      "deleted" : 0,
      "missing" : 0,
      "failed" : 0
    }
  },
  "failures" : [ ]
}
[root@esdbqp bin]#
~~~

我觉得CLI下面这种方法比较麻烦


> **Tip:** 
>
> * 不加 **pretty=true** 输出仍然为jason，但因为没有换行，显示会很凌乱
> * 事实上这个插件也还是使用的 **scroll** 加 **bulk** 的方式在内部进行实现的


>Internally, the query is used to execute an initial scroll request. As hits are pulled from the scroll API, they are passed to the Bulk API for deletion.


执行结果显示正常删除了110条记录

进行查看，剩下4104条，而原来总共4214条，确实是删掉了这么多

~~~
[root@esdbqp bin]# curl "http://localhost:9200/filebeat-*/log/_search?pretty=true&q=tags:multiline" | head -n 20 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12576  100 1  0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
{
  "took" : 120,
  "timed_out" : false,
  "_shards" : {
    "total" : 165,
    "successful" : 165,
    "failed" : 0
  },
  "hits" : {
    "total" : 4104,
    "max_score" : 9.852164,
    "hits" : [ {
      "_index" : "filebeat-2016.01.29",
      "_type" : "log",
      "_id" : "AVKLW366FoV7cO-daoBe",
      "_score" : 9.852164,
      ...
      ...
      ...
~~~

上面是对单个索引里的内容进行删除操作

现在对索引进行批量操作

> **Tip:** 这个操作在前面没有安装插件的情况下，是会报错的

~~~
[root@esdbqp bin]# curl -XDELETE "http://localhost:9200/filebeat-*/log/_query?pretty=true&q=tags:multiline"
{
  "took" : 8468,
  "timed_out" : false,
  "_indices" : {
    "_all" : {
      "found" : 4104,
      "deleted" : 4104,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2015.12.31" : {
      "found" : 136,
      "deleted" : 136,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2015.12.30" : {
      "found" : 129,
      "deleted" : 129,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.13" : {
      "found" : 133,
      "deleted" : 133,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.12" : {
      "found" : 147,
      "deleted" : 147,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.11" : {
      "found" : 138,
      "deleted" : 138,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.10" : {
      "found" : 111,
      "deleted" : 111,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.09" : {
      "found" : 108,
      "deleted" : 108,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.08" : {
      "found" : 133,
      "deleted" : 133,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.07" : {
      "found" : 143,
      "deleted" : 143,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.29" : {
      "found" : 35,
      "deleted" : 35,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.06" : {
      "found" : 155,
      "deleted" : 155,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.05" : {
      "found" : 140,
      "deleted" : 140,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.27" : {
      "found" : 99,
      "deleted" : 99,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.04" : {
      "found" : 132,
      "deleted" : 132,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.26" : {
      "found" : 126,
      "deleted" : 126,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.03" : {
      "found" : 127,
      "deleted" : 127,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.25" : {
      "found" : 114,
      "deleted" : 114,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2015.12.29" : {
      "found" : 189,
      "deleted" : 189,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2015.12.28" : {
      "found" : 308,
      "deleted" : 308,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.02" : {
      "found" : 99,
      "deleted" : 99,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.24" : {
      "found" : 66,
      "deleted" : 66,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.01" : {
      "found" : 95,
      "deleted" : 95,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.23" : {
      "found" : 102,
      "deleted" : 102,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.22" : {
      "found" : 120,
      "deleted" : 120,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.21" : {
      "found" : 104,
      "deleted" : 104,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.20" : {
      "found" : 135,
      "deleted" : 135,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.19" : {
      "found" : 126,
      "deleted" : 126,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.18" : {
      "found" : 147,
      "deleted" : 147,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.17" : {
      "found" : 122,
      "deleted" : 122,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.16" : {
      "found" : 114,
      "deleted" : 114,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.15" : {
      "found" : 124,
      "deleted" : 124,
      "missing" : 0,
      "failed" : 0
    },
    "filebeat-2016.01.14" : {
      "found" : 147,
      "deleted" : 147,
      "missing" : 0,
      "failed" : 0
    }
  },
  "failures" : [ ]
}
[root@esdbqp bin]# 
~~~

结果显示成功删除了4104个记录

进行查看

~~~
[root@esdbqp bin]# curl "http://localhost:9200/filebeat-*/log/_search?pretty=true&q=tags:multiline"
{
  "took" : 43,
  "timed_out" : false,
  "_shards" : {
    "total" : 165,
    "successful" : 165,
    "failed" : 0
  },
  "hits" : {
    "total" : 0,
    "max_score" : null,
    "hits" : [ ]
  }
}
[root@esdbqp bin]# 
~~~

已经没有被标记为 **multiline** 的记录了


---







---

# 命令汇总

* **`curl "http://localhost:9200/filebeat-*/_search?pretty=true&q=tags:multiline"`**
* **`/usr/share/elasticsearch/bin/plugin install delete-by-query`**
* **`/usr/share/elasticsearch/bin/plugin list`**
* **`/etc/init.d/elasticsearch restart`**
* **`curl -XDELETE "http://localhost:9200/filebeat-2016.01.28/log/_query?pretty=true&q=tags:multiline"`**
* **`curl -XDELETE "http://localhost:9200/filebeat-2016.01.28/log/_query?pretty=true" -d '`**
* **`curl -XDELETE "http://localhost:9200/filebeat-*/log/_query?pretty=true&q=tags:multiline"`**
* **`curl "http://localhost:9200/filebeat-*/log/_search?pretty=true&q=tags:multiline"`**
* **`/usr/share/elasticsearch/bin/plugin remove delete-by-query`**



---

# 附

## 删除插件

~~~
[root@esdbqp bin]# /usr/share/elasticsearch/bin/plugin list
Installed plugins in /usr/share/elasticsearch/plugins:
    - delete-by-query
[root@esdbqp bin]# /usr/share/elasticsearch/bin/plugin remove delete-by-query
-> Removing delete-by-query...
Removed delete-by-query
[root@esdbqp bin]# /usr/share/elasticsearch/bin/plugin list
Installed plugins in /usr/share/elasticsearch/plugins:
    - No plugin detected
[root@esdbqp bin]# /etc/init.d/elasticsearch restart 
Stopping elasticsearch:                                    [  OK  ]
Starting elasticsearch:                                    [  OK  ]
[root@esdbqp bin]# 
~~~

> **Note:** 一定要重启服务，才能生效

---

[elasticsearch]:https://www.elastic.co/
[plugins_delete_by_query]:https://www.elastic.co/guide/en/elasticsearch/plugins/2.1/plugins-delete-by-query.html#plugins-delete-by-query
[reason]:https://www.elastic.co/guide/en/elasticsearch/plugins/2.1/delete-by-query-plugin-reason.html#delete-by-query-plugin-reason
