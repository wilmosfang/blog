---
layout: post
title:  Logstash 处理 Mongod Log
author: wilmosfang
tags:   logstash mongodb 
categories:   logstash mongodb 
wc: 408  1350 16251 
excerpt: mongodb日志格式，logstash配置，配置分析，注意事项 
comments: true
---



# 前言

**logstash** 可以处理各类日志

对于 **Mongod** 的 **Log** 来说，情况既简单又复杂

简单性在于 **[mongodb patterns][mongodb_patterns]** 已经都定义好了，拿来就能用；复杂性在于，这样抓出来的信息几乎没有太大价值，无非是实现了一个日志存储的功能，谈不上分析，因为最重要的操作时长未能被抓取，而这个数值是分析慢操作的关键，然而 **Mongod** 日志在不同类别下message部分的格式完全不一样，操作耗时信息是可有可无的

> **Tip:**  grok 预定义的正则匹配可以参考 **[grok patterns][grok_patterns]** ，mongo的日志规范可以参考 **[Mongodb Log][mongodb_log]**，不同版本的格式也是不一样的


这里简单分享一下使用logstash处理 **Mongod** 日志的方法 ，相关的理论基础可以参考 **[grok][plugins_filters_grok]**


> **Tip:** 当前的最新版本为 **MongoDB 3.2.1** ，**Logstash 2.2.1**


---


# 概要

* TOC
{:toc}



---

## 环境

* CentOS release 6.6 (Final)
* 2.6.32-504.el6.x86_64
* logstash 2.1.1
* elasticsearch-2.1.1
* MongoDB version: 3.0.3

---

## 格式


从 **MongoDB 3.0** 开始，MongoDB 日志的格式包含了 **severity level** 和 **component** 

~~~
<timestamp> <severity> <component> [<context>] <message>
~~~

### Timestamp

默认是使用的 **iso8601-local**


### Severity Levels


Level	| Description
----| ---
F   |Fatal
E	|Error
W	|Warning
I	|Informational, for Verbosity Level of 0
D	|Debug, for All Verbosity Levels > 0


### Components

Item| Description
----| ---
ACCESS|Messages related to access control, such as authentication
COMMAND|Messages related to database commands, such as count
CONTROL|Messages related to control activities, such as initialization
GEO|Messages related to the parsing of geospatial shapes, such as verifying the GeoJSON shapes
INDEX|Messages related to indexing operations, such as creating indexes
NETWORK|Messages related to network activities, such as accepting connections
QUERY|Messages related to queries, including query planner activities
REPL|Messages related to replica sets, such as initial sync and heartbeats
SHARDING|Messages related to sharding activities, such as the startup of the mongos
STORAGE|Messages related to storage activities, such as processes involved in the fsync command
JOURNAL|Messages related specifically to journaling activities
WRITE|Messages related to write operations, such as update commands
-|Messages not associated with a named component



> **Tip:** 使用 **`db.getLogComponents()`**  和 **`db.setLogLevel(-1, "query")`** 来查看和设定日志级别



### 关注信息

从下面实例的格式中可以看到

~~~
2014-11-03T18:28:32.450-0500 I NETWORK  [initandlisten] waiting for connections on port 27017
2015-12-25T18:41:47.683+0800 I CONTROL  [signalProcessingThread] pid=37405 port=27017 64-bit host=mongodb-server
2015-12-25T18:51:43.858+0800 I QUERY    [conn425412] query local.oplog.rs query: { ts: { $gte: Timestamp 1450975902000|10 } } planSummary: COLLSCAN cursorid:400229983803 ntoreturn:0 ntoskip:0 nscanned:0 nscannedObjects:102 keyUpdates:0 writeConflicts:0 numYields:11609 nreturned:101 reslen:18110 locks:{ Global: { acquireCount: { r: 11610 } }, MMAPV1Journal: { acquireCount: { r: 11611 } }, Database: { acquireCount: { r: 11610 }, acquireWaitCount: { r: 1 }, timeAcquiringMicros: { r: 165 } }, oplog: { acquireCount: { R: 11610 } } } 1211ms
2015-12-25T20:54:11.336+0800 I JOURNAL  [journal writer] old journal file will be removed: /var/lib/mongo/journal/j._177
2015-12-26T00:46:36.512+0800 I COMMAND  [conn424487] command feed_test_repo.$cmd command: geoNear { geoNear: "users", near: [ 88.598884, 44.102866 ], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms
2015-12-26T02:15:02.218+0800 I QUERY    [conn429640] assertion 13435 not master and slaveOk=false ns:feed_test_repo.notifications query:{ query: {}, orderby: { _id: 1.0 } }
2015-12-26T13:50:20.755+0800 I REPL     [ReplicationExecutor] Member 192.168.100.123:27017 is now in state ARBITER
2015-12-29T01:45:40.781+0800 I STORAGE  [FileAllocator] allocating new datafile /var/lib/mongo/feed_test_repo.107, filling with zeroes...
~~~

参考

~~~
<timestamp> <severity> <component> [<context>] <message>
~~~

* 前四部分(`<timestamp> <severity> <component> [<context>]`)的内容相对固定
* 最后一部分 (`<message>`) 内部比较多变

我们比较关心操作时长，希望可以将这个信息收集进来，这个信息在最后一部分包含，有些内容包含，有些不包含



---

## logstash配置


~~~
[root@h102 etc]# cat logstash-for-mongo.conf  
input {
  stdin {}
  file {
	    type=>"mongolog"
	    path=>"/tmp/xyz.log"
	    start_position => beginning
       }
}

filter {
  grok {
       match => ["message","%{TIMESTAMP_ISO8601:timestamp}\s+%{MONGO3_SEVERITY:severity}\s+%{MONGO3_COMPONENT:component}%{SPACE}(?:\[%{DATA:context}\])?\s+%{GREEDYDATA:body}"]
  } 
  if [body] =~ "ms$"  {  
       grok {
	match => ["body",".*\}(\s+%{NUMBER:spend_time:int}ms$)?"]
       }
 }
 date {
   match => [ "timestamp", "ISO8601" ]
   #remove_field => [ "timestamp" ]
  }
}

output {
  elasticsearch { 
  	hosts => ["localhost:9200"] 
        index=>"mongodb-slow-log-%{+YYYY.MM.dd}"
	}
  stdout { codec => rubydebug }
}
[root@h102 etc]# 
~~~


### 检测配置

~~~
[root@h102 etc]# /opt/logstash/bin/logstash -f  logstash-for-mongo.conf -t
Configuration OK
[root@h102 etc]# 
~~~

### 运行logstash

~~~
[root@h102 etc]# /opt/logstash/bin/logstash -f  logstash-for-mongo.conf
Settings: Default filter workers: 1
Logstash startup completed
...
...
...
~~~

### 输入测试

~~~
2015-12-25T18:41:47.683+0800 I CONTROL  [signalProcessingThread] db version v3.0.3
{
       "message" => "2015-12-25T18:41:47.683+0800 I CONTROL  [signalProcessingThread] db version v3.0.3",
      "@version" => "1",
    "@timestamp" => "2015-12-25T10:41:47.683Z",
          "host" => "h102.temp",
     "timestamp" => "2015-12-25T18:41:47.683+0800",
      "severity" => "I",
     "component" => "CONTROL",
       "context" => "signalProcessingThread",
          "body" => "db version v3.0.3"
}
2015-12-25T20:54:11.336+0800 I JOURNAL  [journal writer] old journal file will be removed: /var/lib/mongo/journal/j._177
{
       "message" => "2015-12-25T20:54:11.336+0800 I JOURNAL  [journal writer] old journal file will be removed: /var/lib/mongo/journal/j._177",
      "@version" => "1",
    "@timestamp" => "2015-12-25T12:54:11.336Z",
          "host" => "h102.temp",
     "timestamp" => "2015-12-25T20:54:11.336+0800",
      "severity" => "I",
     "component" => "JOURNAL",
       "context" => "journal writer",
          "body" => "old journal file will be removed: /var/lib/mongo/journal/j._177"
}
2015-12-26T00:46:36.512+0800 I COMMAND  [conn424487] command feed_test_repo.$cmd command: geoNear { geoNear: "users", near: [ 88.598884, 44.102866 ], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms
{
       "message" => "2015-12-26T00:46:36.512+0800 I COMMAND  [conn424487] command feed_test_repo.$cmd command: geoNear { geoNear: \"users\", near: [ 88.598884, 44.102866 ], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms",
      "@version" => "1",
    "@timestamp" => "2015-12-25T16:46:36.512Z",
          "host" => "h102.temp",
     "timestamp" => "2015-12-26T00:46:36.512+0800",
      "severity" => "I",
     "component" => "COMMAND",
       "context" => "conn424487",
          "body" => "command feed_test_repo.$cmd command: geoNear { geoNear: \"users\", near: [ 88.598884, 44.102866 ], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms",
    "spend_time" => 2584
}
~~~

可以正常解析

> **Tip:** 如果无法正常解析, tags 里会多出一个 **_grokparsefailure** ，并且无法捕获下面多出来的那些值


~~~
18:41:47.683+0800 I CONTROL  [signalProcessingThread] db version v3.0.3
{
       "message" => "18:41:47.683+0800 I CONTROL  [signalProcessingThread] db version v3.0.3",
      "@version" => "1",
    "@timestamp" => "2016-02-15T07:20:19.479Z",
          "host" => "h102.temp",
          "tags" => [
        [0] "_grokparsefailure"
    ]
}
~~~

---

## 配置分析


### input

~~~
input {
  stdin {}
  file {
	    type=>"mongolog"
	    path=>"/tmp/xyz.log"
	    start_position => beginning
       }
}
~~~

Item     | Comment
-------- | ---
`input {`| 框定输入源的定义范围
`stdin {`|定义了一个输入源，使用 **[stdin][stdin]** 插件从标准输入读取数据，也就是终端读入(生产中不会这样配置，一般用来进行交互调试)
`file {`|定义了一个输入源，使用 **[file][file]** 插件从指定文本读取数据
`type=>"mongolog"`| 指定读入数据的类型
`path=>"/tmp/xyz.log"`| 指定输入源文件的地址，必须为绝对路径
`start_position => beginning`| 指定读取特性，默认为跟踪新生成的记录或条目

合起来的意思就是：从终端读取，从 **/tmp/xyz.log** 的开头读取并打上mongolog的类型(从终端读取的没有此类型标签)

---

### filter

~~~
filter {
  grok {
       match => ["message","%{TIMESTAMP_ISO8601:timestamp}\s+%{MONGO3_SEVERITY:severity}\s+%{MONGO3_COMPONENT:component}%{SPACE}(?:\[%{DATA:context}\])?\s+%{GREEDYDATA:body}"]
  } 
  if [body] =~ "ms$"  {  
       grok {
	     match => ["body",".*\}(\s+%{NUMBER:spend_time:int}ms$)?"]
       }
 }
 date {
   match => [ "timestamp", "ISO8601" ]
   #remove_field => [ "timestamp" ]
  }
}
~~~

Item     | Comment
-------- | ---
`filter {`|框定处理逻辑的定义范围
`grok {`|定义了一个过滤器，使用 **[grok][grok]** 插件来解析文本，和抓取信息，用于文本结构化
`match => ["message",".*"]`| 用来match哈希 `{"message" => ".*patten.*"}`,然后把正则捕获的值作为事件日志的filed
`if [body] =~ "ms$"`| 判断 `body` 字段中是否以 `ms` 结尾，如果匹配，就执行定义的代码段
`match => ["body",".*\}(\s+%{NUMBER:spend_time:int}ms$)?"]`| 尝试从body中抽取花费的时间
`date {`| 定义了一个过滤器，使用 **[date][date]** 插件来从fileds中解析出时间，然后把获取的时间值作为此次事件日志的时间戳
`match => [ "timestamp", "ISO8601" ]`|取用 **timestamp** 中的时间作为事件日志时间戳，模式匹配为 **ISO8601** 
`#remove_field => [ "timestamp" ]`|一般而言，日志会有一个自己的时间戳 **@timestamp** ,这是logstash或 beats看到日志时的时间点，但是上一步已经将从日志捕获的时间赋给了 **@timestamp** ，所以 timestamp 就是一份冗余的信息,可以使用 **remove_field** 方法来删掉这个字段，但我选择保留


> **Note:** 这里的 **if** 判断不能省，否则会产生大量的 **_grokparsefailure**

---

### output


~~~
output {
  elasticsearch { 
  	hosts => ["localhost:9200"] 
        index=>"mongodb-slow-log-%{+YYYY.MM.dd}"
	}
  stdout { codec => rubydebug }
}
~~~


Item     | Comment
-------- | ---
`output {`|框定出口的定义范围
`elasticsearch {`|  定义了一个出口，使用 **[elasticsearch][elasticsearch]** 插件来进行输出，将结果输出到ES中
`hosts => ["localhost:9200"]`| 指定es的目标地址为 **localhost:9200**
`index=>"mongodb-slow-log-%{+YYYY.MM.dd}"`| 指定存到哪个index，如不指定，默认为**logstash-%{+YYYY.MM.dd}**
`stdout { codec => rubydebug }`| 定义了一个出口，使用 **[stdout][stdout]** 插件将信息输出到标准输，也就是终端，并且使用 **[rubydebug][rubydebug]** 插件处理过后进行展示，也就是行成jason格式 (生产不会这样配置，一般用来进行交互调试)


---


### 正则


>`%{TIMESTAMP_ISO8601:timestamp}\s+%{MONGO3_SEVERITY:severity}\s+%{MONGO3_COMPONENT:component}%{SPACE}(?:\[%{DATA:context}\])?\s+%{GREEDYDATA:body}`


Item     | Comment
-------- | ---
`%{TIMESTAMP_ISO8601:timestamp}`|将匹配 **TIMESTAMP_ISO8601** 模式的结果放到 **timestamp** 中
`\s+`|匹配一个或多个空字符
`(?:\[%{DATA:context}\])?`| 内容可能有，也可能无，如果有,以 **[** 开头，且以 **]** 结尾，中间的任何内容放到 **context** 中


> **Tip:** 可以参考 **[mongodb patterns][mongodb_patterns]** 中的匹配设置 ，`MONGO3_LOG %{TIMESTAMP_ISO8601:timestamp} %{MONGO3_SEVERITY:severity} %{MONGO3_COMPONENT:component}%{SPACE}(?:\[%{DATA:context}\])? %{GREEDYDATA:message}` ,我将最后的部分存入了body，不然会存到原来的 message 字段中， 使message变成一个列表，内容变成 message中的第二个元素，然后将空格替换成了 `\s+` ，这样会更消耗计算资源，但是更严谨



>`.*\}(\s+%{NUMBER:spend_time:int}ms$)?`


Item     | Comment
-------- | ---
`.*`|匹配任意内容
`\}`|匹配 **}**
`(\s+%{NUMBER:spend_time:int}ms$)?`|内容可能有，也可能无，如果有,以若干个空字符开头，以 **ms** 结尾，将中间的 **int** 类型数值存储到 **spend_time** 中


> **Note:** `\}` 不能省，否则以 `.*` 的贪婪特性会一口气将后面的所有内容都吞噬掉，从而使 `%{NUMBER:spend_time:int}` 匹配不到数据



---

# 命令汇总

* **`cat logstash-for-mongo.conf`**
* **`/opt/logstash/bin/logstash -f  logstash-for-mongo.conf -t`**
* **`/opt/logstash/bin/logstash -f  logstash-for-mongo.conf`**

---

# 附

* [grok patterns][grok_patterns] : grok的预定义模式

* [mongodb patterns][mongodb_patterns] : mongo的预定义模式

* [grok conditionals][grok_conditionals] : grok的条件判断

* [patterns][patterns] : 其它预定义模式

---

[grok_patterns]:https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns
[mongodb_patterns]:https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/mongodb
[mongodb_log]:https://docs.mongodb.org/manual/reference/log-messages/
[plugins_filters_grok]:https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
[grok_conditionals]:https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#conditionals
[stdin]:https://www.elastic.co/guide/en/logstash/current/plugins-inputs-stdin.html
[file]:https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html
[grok]:https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
[date]:https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
[elasticsearch]:https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
[stdout]:https://www.elastic.co/guide/en/logstash/current/plugins-outputs-stdout.html
[rubydebug]:https://www.elastic.co/guide/en/logstash/current/plugins-codecs-rubydebug.html
[patterns]:https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns

