---
layout: post
title:  Beats Dashboards 基础
categories: linux elasticsearch logstash beats filebeat kibana
wc: 543  1179 33266 
excerpt:  Beats Dashboards 的安装与配置方法
comments: true
---



# 前言

**[Beats][beats]** 是作为ELK技术栈前端数据收集平台的一个开源软件


>Beats is the platform for building lightweight, open source data shippers for many types of operational data you want to enrich with Logstash, search and analyze in Elasticsearch, and visualize in Kibana. Whether you’re interested in log files, infrastructure metrics, network packets, or any other type of data, Beats serves as the foundation for keeping a beat on your data

目前有官方支持的三个子产品：**[packetbeat][packetbeat]、[topbeat][topbeat]、[filebeat][filebeat]**

* **[packetbeat][packetbeat]** 侧重于收集网络包
* **[topbeat][topbeat]** 侧重于收集基础架构信息，如CPU，Memory，Progress 相关信息
* **[filebeat][filebeat]** 侧重于收集日志型信息

下面是它们之间的关系

![beats-logstash.png](/images/beats/beats-logstash.png)

关于 **[Beats][beats]** 的更详细信息可以参考 **[Beats Platform][beats_doc]**


**[filebeat][filebeat]** 从目前来讲有更常见的使用场景，它是用来替代 **Logstash Forwarder** 的下一代 **Logstash** 收集器，是为了更快速稳定轻量低耗地进行收集工作，它可以很方便地与 **Logstash** 还有直接与 **Elasticsearch** 进行对接

> **Tip:** filebeat 是在 Logstash Forwarder 的源码基础上演化过来的项目

同时可以方便的使用 **Kibana** 来进行展示，只用导入 **Beats Dashboards** 模板到 **kibana** 里，然后选择默认 **index** 为 **`[filebeat-]YYYY.MM.DD`** ，就可以在界面里进行查看和检索了

下面对 **Beats Dashboards** 的下载安装与配置方法进行分享，详细可以参看 **ELK** 的 **[集成文档][elk]**


> **Tip:** 当前的版本为 **beats-dashboards-1.0.1** 

---



# 概要

* TOC
{:toc}


---

##承前

完成上一篇《Beats 基础》后再看ES里的信息

{% highlight bash %}
[root@h102 ~]# curl localhost:9200/_cat/indices?v
health status index               pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   logstash-2015.12.23   5   1        100            0    235.8kb        235.8kb 
yellow open   filebeat-2015.12.24   5   1       3175            0   1018.6kb       1018.6kb 
yellow open   logstash-2015.12.22   5   1         41            0    126.5kb        126.5kb 
yellow open   .kibana               1   1          2            0      8.3kb          8.3kb 
[root@h102 ~]# 
{% endhighlight %}

发现已经产生了 **`filebeat-*`** 的index，并且有相当数量的文档


---

##下载beats dashboards

{% highlight bash %}
[root@h102 ~]# mkdir beats-dashboards
[root@h102 ~]# cd beats-dashboards/
[root@h102 beats-dashboards]# curl -L -O http://download.elastic.co/beats/dashboards/beats-dashboards-1.0.1.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
  2  763k    2 19025    0     0   5422      0  0:02:24  0:00:03  0:02:21  7136
  5  763k    5 43641    0     0  10651      0  0:01:13  0:00:04  0:01:09 13407
  ...
  ...
  99  763k   99  760k    0     0   7210      0  0:01:48  0:01:48 --:--:--  6970
  99  763k   99  762k    0     0   7123      0  0:01:49  0:01:49 --:--:--  5270
 100  763k  100  763k    0     0   7127      0  0:01:49  0:01:49 --:--:--  6278
[root@h102 beats-dashboards]# ll 
total 764
-rw-r--r-- 1 root root 782123 Dec 24 23:47 beats-dashboards-1.0.1.tar.gz
[root@h102 beats-dashboards]# du -sh beats-dashboards-1.0.1.tar.gz 
764K	beats-dashboards-1.0.1.tar.gz
[root@h102 beats-dashboards]#
{% endhighlight %}

---

##解压并导入


{% highlight bash %}
[root@h102 beats-dashboards]# du -sh beats-dashboards-1.0.1.tar.gz 
764K	beats-dashboards-1.0.1.tar.gz
[root@h102 beats-dashboards]# tar -zxvf beats-dashboards-1.0.1.tar.gz 
beats-dashboards-1.0.1/
beats-dashboards-1.0.1/.gitignore
beats-dashboards-1.0.1/CHANGELOG.md
beats-dashboards-1.0.1/Makefile
beats-dashboards-1.0.1/README.md
beats-dashboards-1.0.1/dashboards/
beats-dashboards-1.0.1/dashboards/dashboard/
beats-dashboards-1.0.1/dashboards/dashboard/HTTP.json
beats-dashboards-1.0.1/dashboards/dashboard/MongoDB-performance.json
beats-dashboards-1.0.1/dashboards/dashboard/MySQL-performance.json
beats-dashboards-1.0.1/dashboards/dashboard/Packetbeat-Dashboard.json
beats-dashboards-1.0.1/dashboards/dashboard/PgSQL-performance.json
beats-dashboards-1.0.1/dashboards/dashboard/Thrift-performance.json
beats-dashboards-1.0.1/dashboards/dashboard/Topbeat-Dashboard.json
beats-dashboards-1.0.1/dashboards/index-pattern/
beats-dashboards-1.0.1/dashboards/index-pattern/[filebeat-]YYYY.MM.DD.json
beats-dashboards-1.0.1/dashboards/index-pattern/[packetbeat-]YYYY.MM.DD.json
beats-dashboards-1.0.1/dashboards/index-pattern/[topbeat-]YYYY.MM.DD.json
beats-dashboards-1.0.1/dashboards/search/
beats-dashboards-1.0.1/dashboards/search/Cache-transactions.json
beats-dashboards-1.0.1/dashboards/search/DB-transactions.json
beats-dashboards-1.0.1/dashboards/search/Default-Search.json
beats-dashboards-1.0.1/dashboards/search/Errors.json
beats-dashboards-1.0.1/dashboards/search/Filesystem-stats.json
beats-dashboards-1.0.1/dashboards/search/HTTP-errors.json
beats-dashboards-1.0.1/dashboards/search/MongoDB-errors.json
beats-dashboards-1.0.1/dashboards/search/MongoDB-transactions-with-write-concern-0.json
beats-dashboards-1.0.1/dashboards/search/MongoDB-transactions.json
beats-dashboards-1.0.1/dashboards/search/MySQL-Transactions.json
beats-dashboards-1.0.1/dashboards/search/MySQL-errors.json
beats-dashboards-1.0.1/dashboards/search/PgSQL-errors.json
beats-dashboards-1.0.1/dashboards/search/PgSQL-transactions.json
beats-dashboards-1.0.1/dashboards/search/Proc-stats.json
beats-dashboards-1.0.1/dashboards/search/Processes.json
beats-dashboards-1.0.1/dashboards/search/RPC-transactions.json
beats-dashboards-1.0.1/dashboards/search/System-stats.json
beats-dashboards-1.0.1/dashboards/search/System-wide.json
beats-dashboards-1.0.1/dashboards/search/Thrift-errors.json
beats-dashboards-1.0.1/dashboards/search/Thrift-transactions.json
beats-dashboards-1.0.1/dashboards/search/Web-transactions.json
beats-dashboards-1.0.1/dashboards/visualization/
beats-dashboards-1.0.1/dashboards/visualization/Average-system-load-across-all-systems.json
beats-dashboards-1.0.1/dashboards/visualization/CPU-usage-per-process.json
beats-dashboards-1.0.1/dashboards/visualization/CPU-usage.json
beats-dashboards-1.0.1/dashboards/visualization/Cache-transactions.json
beats-dashboards-1.0.1/dashboards/visualization/Client-locations.json
beats-dashboards-1.0.1/dashboards/visualization/DB-transactions.json
beats-dashboards-1.0.1/dashboards/visualization/Disk-usage-overview.json
beats-dashboards-1.0.1/dashboards/visualization/Disk-usage.json
beats-dashboards-1.0.1/dashboards/visualization/Errors-count-over-time.json
beats-dashboards-1.0.1/dashboards/visualization/Errors-vs-successful-transactions.json
beats-dashboards-1.0.1/dashboards/visualization/Evolution-of-the-CPU-times-per-process.json
beats-dashboards-1.0.1/dashboards/visualization/HTTP-codes-for-the-top-queries.json
beats-dashboards-1.0.1/dashboards/visualization/HTTP-error-codes-evolution.json
beats-dashboards-1.0.1/dashboards/visualization/HTTP-error-codes.json
beats-dashboards-1.0.1/dashboards/visualization/Latency-histogram.json
beats-dashboards-1.0.1/dashboards/visualization/Memory-usage-per-process.json
beats-dashboards-1.0.1/dashboards/visualization/Memory-usage.json
beats-dashboards-1.0.1/dashboards/visualization/MongoDB-commands.json
beats-dashboards-1.0.1/dashboards/visualization/MongoDB-errors-per-collection.json
beats-dashboards-1.0.1/dashboards/visualization/MongoDB-errors.json
beats-dashboards-1.0.1/dashboards/visualization/MongoDB-in-slash-out-throughput.json
beats-dashboards-1.0.1/dashboards/visualization/MongoDB-response-times-and-count.json
beats-dashboards-1.0.1/dashboards/visualization/MongoDB-response-times-by-collection.json
beats-dashboards-1.0.1/dashboards/visualization/Most-frequent-MySQL-queries.json
beats-dashboards-1.0.1/dashboards/visualization/Most-frequent-PgSQL-queries.json
beats-dashboards-1.0.1/dashboards/visualization/MySQL-Errors.json
beats-dashboards-1.0.1/dashboards/visualization/MySQL-Methods.json
beats-dashboards-1.0.1/dashboards/visualization/MySQL-Reads-vs-Writes.json
beats-dashboards-1.0.1/dashboards/visualization/MySQL-throughput.json
beats-dashboards-1.0.1/dashboards/visualization/Mysql-response-times-percentiles.json
beats-dashboards-1.0.1/dashboards/visualization/Navigation.json
beats-dashboards-1.0.1/dashboards/visualization/Number-of-MongoDB-transactions-with-writeConcern-w-equal-0.json
beats-dashboards-1.0.1/dashboards/visualization/PgSQL-Errors.json
beats-dashboards-1.0.1/dashboards/visualization/PgSQL-Methods.json
beats-dashboards-1.0.1/dashboards/visualization/PgSQL-Reads-vs-Writes.json
beats-dashboards-1.0.1/dashboards/visualization/PgSQL-response-times-percentiles.json
beats-dashboards-1.0.1/dashboards/visualization/PgSQL-throughput.json
beats-dashboards-1.0.1/dashboards/visualization/Process-status.json
beats-dashboards-1.0.1/dashboards/visualization/RPC-transactions.json
beats-dashboards-1.0.1/dashboards/visualization/Reads-versus-Writes.json
beats-dashboards-1.0.1/dashboards/visualization/Response-times-percentiles.json
beats-dashboards-1.0.1/dashboards/visualization/Response-times-repartition.json
beats-dashboards-1.0.1/dashboards/visualization/Servers.json
beats-dashboards-1.0.1/dashboards/visualization/Slowest-MySQL-queries.json
beats-dashboards-1.0.1/dashboards/visualization/Slowest-PgSQL-queries.json
beats-dashboards-1.0.1/dashboards/visualization/Slowest-Thrift-RPC-methods.json
beats-dashboards-1.0.1/dashboards/visualization/System-load.json
beats-dashboards-1.0.1/dashboards/visualization/Thrift-RPC-Errors.json
beats-dashboards-1.0.1/dashboards/visualization/Thrift-requests-per-minute.json
beats-dashboards-1.0.1/dashboards/visualization/Thrift-response-times-percentiles.json
beats-dashboards-1.0.1/dashboards/visualization/Top-10-HTTP-requests.json
beats-dashboards-1.0.1/dashboards/visualization/Top-10-memory-consumers.json
beats-dashboards-1.0.1/dashboards/visualization/Top-10-processes-by-total-CPU-usage.json
beats-dashboards-1.0.1/dashboards/visualization/Top-Thrift-RPC-calls-with-errors.json
beats-dashboards-1.0.1/dashboards/visualization/Top-Thrift-RPC-methods.json
beats-dashboards-1.0.1/dashboards/visualization/Top-processes.json
beats-dashboards-1.0.1/dashboards/visualization/Top-slowest-MongoDB-queries.json
beats-dashboards-1.0.1/dashboards/visualization/Total-number-of-HTTP-transactions.json
beats-dashboards-1.0.1/dashboards/visualization/Total-time-spent-in-each-MongoDB-collection.json
beats-dashboards-1.0.1/dashboards/visualization/Web-transactions.json
beats-dashboards-1.0.1/load.sh
beats-dashboards-1.0.1/release.sh
beats-dashboards-1.0.1/save/
beats-dashboards-1.0.1/save/README.md
beats-dashboards-1.0.1/save/kibana_dump.py
beats-dashboards-1.0.1/save/requirements.txt
beats-dashboards-1.0.1/screenshots/
beats-dashboards-1.0.1/screenshots/MySql-performance.png
beats-dashboards-1.0.1/screenshots/Packetbeat-statistics.png
beats-dashboards-1.0.1/screenshots/PgSql-performance.png
beats-dashboards-1.0.1/screenshots/Thrift-performance.png
beats-dashboards-1.0.1/screenshots/Topbeat-statistics.png
[root@h102 beats-dashboards]# ls
beats-dashboards-1.0.1  beats-dashboards-1.0.1.tar.gz
[root@h102 beats-dashboards]# cd beats-dashboards-1.0.1
[root@h102 beats-dashboards-1.0.1]# ./load.sh 
curl
Loading search Cache-transactions:
{"_index":".kibana","_type":"search","_id":"Cache-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search DB-transactions:
{"_index":".kibana","_type":"search","_id":"DB-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Default-Search:
{"_index":".kibana","_type":"search","_id":"Default-Search","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Errors:
{"_index":".kibana","_type":"search","_id":"Errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Filesystem-stats:
{"_index":".kibana","_type":"search","_id":"Filesystem-stats","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search HTTP-errors:
{"_index":".kibana","_type":"search","_id":"HTTP-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search MongoDB-errors:
{"_index":".kibana","_type":"search","_id":"MongoDB-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search MongoDB-transactions:
{"_index":".kibana","_type":"search","_id":"MongoDB-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search MongoDB-transactions-with-write-concern-0:
{"_index":".kibana","_type":"search","_id":"MongoDB-transactions-with-write-concern-0","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search MySQL-errors:
{"_index":".kibana","_type":"search","_id":"MySQL-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search MySQL-Transactions:
{"_index":".kibana","_type":"search","_id":"MySQL-Transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search PgSQL-errors:
{"_index":".kibana","_type":"search","_id":"PgSQL-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search PgSQL-transactions:
{"_index":".kibana","_type":"search","_id":"PgSQL-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Processes:
{"_index":".kibana","_type":"search","_id":"Processes","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Proc-stats:
{"_index":".kibana","_type":"search","_id":"Proc-stats","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search RPC-transactions:
{"_index":".kibana","_type":"search","_id":"RPC-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search System-stats:
{"_index":".kibana","_type":"search","_id":"System-stats","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search System-wide:
{"_index":".kibana","_type":"search","_id":"System-wide","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Thrift-errors:
{"_index":".kibana","_type":"search","_id":"Thrift-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Thrift-transactions:
{"_index":".kibana","_type":"search","_id":"Thrift-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading search Web-transactions:
{"_index":".kibana","_type":"search","_id":"Web-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Average-system-load-across-all-systems:
{"_index":".kibana","_type":"visualization","_id":"Average-system-load-across-all-systems","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Cache-transactions:
{"_index":".kibana","_type":"visualization","_id":"Cache-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Client-locations:
{"_index":".kibana","_type":"visualization","_id":"Client-locations","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization CPU-usage:
{"_index":".kibana","_type":"visualization","_id":"CPU-usage","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization CPU-usage-per-process:
{"_index":".kibana","_type":"visualization","_id":"CPU-usage-per-process","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization DB-transactions:
{"_index":".kibana","_type":"visualization","_id":"DB-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Disk-usage:
{"_index":".kibana","_type":"visualization","_id":"Disk-usage","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Disk-usage-overview:
{"_index":".kibana","_type":"visualization","_id":"Disk-usage-overview","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Errors-count-over-time:
{"_index":".kibana","_type":"visualization","_id":"Errors-count-over-time","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Errors-vs-successful-transactions:
{"_index":".kibana","_type":"visualization","_id":"Errors-vs-successful-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Evolution-of-the-CPU-times-per-process:
{"_index":".kibana","_type":"visualization","_id":"Evolution-of-the-CPU-times-per-process","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization HTTP-codes-for-the-top-queries:
{"_index":".kibana","_type":"visualization","_id":"HTTP-codes-for-the-top-queries","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization HTTP-error-codes-evolution:
{"_index":".kibana","_type":"visualization","_id":"HTTP-error-codes-evolution","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization HTTP-error-codes:
{"_index":".kibana","_type":"visualization","_id":"HTTP-error-codes","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Latency-histogram:
{"_index":".kibana","_type":"visualization","_id":"Latency-histogram","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Memory-usage:
{"_index":".kibana","_type":"visualization","_id":"Memory-usage","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Memory-usage-per-process:
{"_index":".kibana","_type":"visualization","_id":"Memory-usage-per-process","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MongoDB-commands:
{"_index":".kibana","_type":"visualization","_id":"MongoDB-commands","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MongoDB-errors:
{"_index":".kibana","_type":"visualization","_id":"MongoDB-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MongoDB-errors-per-collection:
{"_index":".kibana","_type":"visualization","_id":"MongoDB-errors-per-collection","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MongoDB-in-slash-out-throughput:
{"_index":".kibana","_type":"visualization","_id":"MongoDB-in-slash-out-throughput","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MongoDB-response-times-and-count:
{"_index":".kibana","_type":"visualization","_id":"MongoDB-response-times-and-count","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MongoDB-response-times-by-collection:
{"_index":".kibana","_type":"visualization","_id":"MongoDB-response-times-by-collection","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Most-frequent-MySQL-queries:
{"_index":".kibana","_type":"visualization","_id":"Most-frequent-MySQL-queries","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Most-frequent-PgSQL-queries:
{"_index":".kibana","_type":"visualization","_id":"Most-frequent-PgSQL-queries","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MySQL-Errors:
{"_index":".kibana","_type":"visualization","_id":"MySQL-Errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MySQL-Methods:
{"_index":".kibana","_type":"visualization","_id":"MySQL-Methods","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MySQL-Reads-vs-Writes:
{"_index":".kibana","_type":"visualization","_id":"MySQL-Reads-vs-Writes","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Mysql-response-times-percentiles:
{"_index":".kibana","_type":"visualization","_id":"Mysql-response-times-percentiles","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization MySQL-throughput:
{"_index":".kibana","_type":"visualization","_id":"MySQL-throughput","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Navigation:
{"_index":".kibana","_type":"visualization","_id":"Navigation","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Number-of-MongoDB-transactions-with-writeConcern-w-equal-0:
{"_index":".kibana","_type":"visualization","_id":"Number-of-MongoDB-transactions-with-writeConcern-w-equal-0","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization PgSQL-Errors:
{"_index":".kibana","_type":"visualization","_id":"PgSQL-Errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization PgSQL-Methods:
{"_index":".kibana","_type":"visualization","_id":"PgSQL-Methods","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization PgSQL-Reads-vs-Writes:
{"_index":".kibana","_type":"visualization","_id":"PgSQL-Reads-vs-Writes","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization PgSQL-response-times-percentiles:
{"_index":".kibana","_type":"visualization","_id":"PgSQL-response-times-percentiles","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization PgSQL-throughput:
{"_index":".kibana","_type":"visualization","_id":"PgSQL-throughput","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Process-status:
{"_index":".kibana","_type":"visualization","_id":"Process-status","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Reads-versus-Writes:
{"_index":".kibana","_type":"visualization","_id":"Reads-versus-Writes","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Response-times-percentiles:
{"_index":".kibana","_type":"visualization","_id":"Response-times-percentiles","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Response-times-repartition:
{"_index":".kibana","_type":"visualization","_id":"Response-times-repartition","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization RPC-transactions:
{"_index":".kibana","_type":"visualization","_id":"RPC-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Servers:
{"_index":".kibana","_type":"visualization","_id":"Servers","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Slowest-MySQL-queries:
{"_index":".kibana","_type":"visualization","_id":"Slowest-MySQL-queries","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Slowest-PgSQL-queries:
{"_index":".kibana","_type":"visualization","_id":"Slowest-PgSQL-queries","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Slowest-Thrift-RPC-methods:
{"_index":".kibana","_type":"visualization","_id":"Slowest-Thrift-RPC-methods","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization System-load:
{"_index":".kibana","_type":"visualization","_id":"System-load","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Thrift-requests-per-minute:
{"_index":".kibana","_type":"visualization","_id":"Thrift-requests-per-minute","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Thrift-response-times-percentiles:
{"_index":".kibana","_type":"visualization","_id":"Thrift-response-times-percentiles","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Thrift-RPC-Errors:
{"_index":".kibana","_type":"visualization","_id":"Thrift-RPC-Errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-10-HTTP-requests:
{"_index":".kibana","_type":"visualization","_id":"Top-10-HTTP-requests","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-10-memory-consumers:
{"_index":".kibana","_type":"visualization","_id":"Top-10-memory-consumers","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-10-processes-by-total-CPU-usage:
{"_index":".kibana","_type":"visualization","_id":"Top-10-processes-by-total-CPU-usage","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-processes:
{"_index":".kibana","_type":"visualization","_id":"Top-processes","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-slowest-MongoDB-queries:
{"_index":".kibana","_type":"visualization","_id":"Top-slowest-MongoDB-queries","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-Thrift-RPC-calls-with-errors:
{"_index":".kibana","_type":"visualization","_id":"Top-Thrift-RPC-calls-with-errors","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Top-Thrift-RPC-methods:
{"_index":".kibana","_type":"visualization","_id":"Top-Thrift-RPC-methods","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Total-number-of-HTTP-transactions:
{"_index":".kibana","_type":"visualization","_id":"Total-number-of-HTTP-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Total-time-spent-in-each-MongoDB-collection:
{"_index":".kibana","_type":"visualization","_id":"Total-time-spent-in-each-MongoDB-collection","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading visualization Web-transactions:
{"_index":".kibana","_type":"visualization","_id":"Web-transactions","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard HTTP:
{"_index":".kibana","_type":"dashboard","_id":"HTTP","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard MongoDB-performance:
{"_index":".kibana","_type":"dashboard","_id":"MongoDB-performance","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard MySQL-performance:
{"_index":".kibana","_type":"dashboard","_id":"MySQL-performance","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard Packetbeat-Dashboard:
{"_index":".kibana","_type":"dashboard","_id":"Packetbeat-Dashboard","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard PgSQL-performance:
{"_index":".kibana","_type":"dashboard","_id":"PgSQL-performance","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard Thrift-performance:
{"_index":".kibana","_type":"dashboard","_id":"Thrift-performance","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading dashboard Topbeat-Dashboard:
{"_index":".kibana","_type":"dashboard","_id":"Topbeat-Dashboard","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading index pattern \[filebeat-\]YYYY.MM.DD:
{"_index":".kibana","_type":"index-pattern","_id":"[filebeat-]YYYY.MM.DD","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading index pattern \[packetbeat-\]YYYY.MM.DD:
{"_index":".kibana","_type":"index-pattern","_id":"[packetbeat-]YYYY.MM.DD","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
Loading index pattern \[topbeat-\]YYYY.MM.DD:
{"_index":".kibana","_type":"index-pattern","_id":"[topbeat-]YYYY.MM.DD","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
[root@h102 beats-dashboards-1.0.1]# 
[root@h102 beats-dashboards-1.0.1]# curl localhost:9200/_cat/indices?v
health status index               pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   logstash-2015.12.23   5   1        100            0    235.8kb        235.8kb 
yellow open   filebeat-2015.12.24   5   1       3179            0        1mb            1mb 
yellow open   logstash-2015.12.22   5   1         41            0    126.5kb        126.5kb 
yellow open   .kibana               1   1         93            0     97.6kb         97.6kb 
[root@h102 beats-dashboards-1.0.1]# 
{% endhighlight %}

**.kibana** 已经变大了，之前是 **8.3kb** 现在是 **97.6kb** ,原因是导入的数据都装到了这里

**load.sh** 实际的作为

{% highlight bash %}
[root@h102 beats-dashboards-1.0.1]# ls
CHANGELOG.md  dashboards  load.sh  Makefile  README.md  release.sh  save  screenshots
[root@h102 beats-dashboards-1.0.1]# wc -l load.sh 
56 load.sh
[root@h102 beats-dashboards-1.0.1]# cat load.sh 
#!/bin/bash

if [ -z "$1" ]; then
    ELASTICSEARCH=http://localhost:9200
else
    ELASTICSEARCH=$1
fi

if [ -z "$2" ]; then
    CURL=curl
else
    CURL="curl --user $2"
fi

echo $CURL
DIR=dashboards

for file in $DIR/search/*.json
do
    name=`basename $file .json`
    echo "Loading search $name:"
    $CURL -XPUT $ELASTICSEARCH/.kibana/search/$name \
        -d @$file || exit 1
    echo
done

for file in $DIR/visualization/*.json
do
    name=`basename $file .json`
    echo "Loading visualization $name:"
    $CURL -XPUT $ELASTICSEARCH/.kibana/visualization/$name \
        -d @$file || exit 1
    echo
done

for file in $DIR/dashboard/*.json
do
    name=`basename $file .json`
    echo "Loading dashboard $name:"
    $CURL -XPUT $ELASTICSEARCH/.kibana/dashboard/$name \
        -d @$file || exit 1
    echo
done

for file in $DIR/index-pattern/*.json
do
    name=`basename $file .json`
    printf -v escape "%q" $name
    echo "Loading index pattern $escape:"

    $CURL -XPUT $ELASTICSEARCH/.kibana/index-pattern/$escape \
        -d @$file || exit 1
    echo
done


[root@h102 beats-dashboards-1.0.1]# 
{% endhighlight %}

---

##选择默认索引

此时进入 **[Settings]** 的 **[indices]** 中就可以看到除了第一次打开 **kibana** 时创建的 **`logstash-*`** 外还多了几个 **Index Patterns** ，**`[filebeat-]YYYY.MM.DD`** 、 **`[packetbeat-]YYYY.MM.DD`** 、  **`[topbeat-]YYYY.MM.DD`** 是从 **beats-dashboards** 导入的

将 **`[filebeat-]YYYY.MM.DD`** 选择成默认index (选择后点击绿色的星)

![beats-dashboard.png](/images/beats/beats-dashboard.png)

然后就可以在 **[Discover]** 中检索看到相应信息

![beats-dashboard2.png](/images/beats/beats-dashboard2.png)

---

##Dashboard

由于导入 **beats-dashboards** 

我们从 **[Dashboard]** 的 **[Load Saved Dashboard]** (文件夹图标) 里可以看到预定义的 Dashboard

![dashboard.png](/images/beats/dashboard.png)

![dashboard2.png](/images/beats/dashboard2.png)

进去之后还没有相应数据

![dashboard3.png](/images/beats/dashboard3.png)

以后导入相应数据就可以很方便地进行查看了


---

#命令总结

* **`curl -L -O http://download.elastic.co/beats/dashboards/beats-dashboards-1.0.1.tar.gz`**
* **`du -sh beats-dashboards-1.0.1.tar.gz`**
* **`tar -zxvf beats-dashboards-1.0.1.tar.gz`**
* **`cd beats-dashboards-1.0.1`**
* **`./load.sh`**
* **`curl localhost:9200/_cat/indices?v`**
* **`cat load.sh`**

---

[beats]:https://www.elastic.co/products/beats
[packetbeat]:https://www.elastic.co/downloads/beats/packetbeat
[topbeat]:https://www.elastic.co/downloads/beats/topbeat
[filebeat]:https://www.elastic.co/downloads/beats/filebeat
[filebeat_doc]:https://www.elastic.co/guide/en/beats/filebeat/current/index.html
[elk]:https://www.elastic.co/guide/en/beats/libbeat/1.0.1/getting-started.html
[beats_doc]:https://www.elastic.co/guide/en/beats/libbeat/1.0.1/index.html

