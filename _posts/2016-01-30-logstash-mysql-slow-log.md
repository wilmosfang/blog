---
layout: post
title:    Logstash 处理 Mysql Slow Log
author: wilmosfang
categories:  linux logstash mysql
wc: 411  1158 18484 
excerpt: logstash配置，input，filter，output 插件详解，正则详解，配置运行与测试 
comments: true
---



# 前言


**logstash** 可以处理各类日志，对于Apache和Nginx的访问日志，由于遵循统一标准，在 **[grok patterns][grok_patterns]** 中已经有现成定义， 一条 **COMBINEDAPACHELOG** 就可以匹配

但是对于 **Mysql** 的 **Slow Log** 来说，情况就要复杂得多，主要表现在格式不统一，字段比较随意，有些字段会偶尔出现，偶尔消失，sql内容也长段不一行数不定，所以目前也没有一个成熟的预定好的 **patterns** 可以拿来就用

>可见统一接口，统一规范的重要性，在我看来，统一标准后，可以为更大规模更大范围的协作带来可能，规避很多不必要的重复劳动，节省下来的宝贵时间可以用来做更有挑战和更有价值的事情


下面是不同版本mysql慢日志的格式

* mysql 5.1.36的slowlog：

~~~
# Time: 151202 17:29:24
# User@Host: root[root] @  [192.168.35.89]
# Query_time: 6.578696  Lock_time: 0.000039 Rows_sent: 999424  Rows_examined: 999424
SET timestamp=1449048564;
select * from users_test;

~~~

* percona server 5.5.34 的slowlog：

~~~
# User@Host: root[root] @  [192.168.35.62]
# Thread_id: 1164217  Schema: mgr  Last_errno: 0  Killed: 0
# Query_time: 0.371185  Lock_time: 0.000056  Rows_sent: 0  Rows_examined: 0  Rows_affected: 2  Rows_read: 0
# Bytes_sent: 11
SET timestamp=1449105655;
REPLACE INTO  edgemgr_dbcache(id, type, data, expire_time)  VALUES(UNHEX('ec124ee5766c4a31819719c645dab895'), 'sermap', '{\"storages\":{\"sg1-s1\":[{\"download_port\":9083,\"p2p_port\":9035,\"rtmp_port\":9035,\"addr\":\"{\\\"l\\\":{\\\"https://192.168.35.40:9184/storage\\\":\\\"\\\"},\\\"m\\\":{},\\\"i\\\":{\\\"https://192.168.35.40:9184/storage\\\":\\\"\\\"}}\",\"cpu\":6,\"mem\":100,\"bandwidth\":0,\"disk\":0,\"dead\":0}]},\"lives\":{}}', '2016-01-02 09:20:55');
~~~



* mysql 5.6.17的slowlog：

~~~
# Time: 151202 16:09:54
# User@Host: root[root] @  [192.168.35.89]  Id: 84589
# Query_time: 7.089324  Lock_time: 0.000112 Rows_sent: 1  Rows_examined: 33554432
SET timestamp=1449043794;
select count(*) from t1;
~~~

* percona server 5.6.27 的slowlog：

~~~
# Time: 151217  2:00:56
# User@Host: taobao[taobao] @ regular_exp [192.168.35.23]  Id:  1235
# Schema: bat_db  Last_errno: 0  Killed: 0
# Query_time: 3.101086  Lock_time: 0.181175  Rows_sent: 0  Rows_examined: 360321  Rows_affected: 103560
# Bytes_sent: 58
SET timestamp=1450288856;
create table just_for_temp_case as
  select '2015-12-16' as the_date,
         t2.user_id,
         t1.stuff_no,
         count(*) as buy_times
  from stuff_entries as t1
  join bill as t2
  on t1.orderserino = t2.id
  where t2.notification_ts < '2015-12-17 00:00:00'
    and t2.notification_ts >= '2015-09-18 00:00:00'
  group by t2.user_id,
           t1.stuff_no;
# Time: 151217 18:03:47
# User@Host: taobao[taobao] @ regular_exp [192.168.35.23]  Id:  1667
# Schema: bat_db  Last_errno: 0  Killed: 0
# Query_time: 2.857983  Lock_time: 0.000095  Rows_sent: 1  Rows_examined: 452253  Rows_affected: 0
# Bytes_sent: 64
use bat_db;
SET timestamp=1450346627;
SELECT COUNT(*) FROM `bill`  WHERE `bill`.`type` IN ('manu_title') AND ( bill.real_name = 'testuser');
~~~

展示上面的例子，只是想说明，不同大版本(5.1与5.5)的mysql slow log 格式不一致，相同大版本小版本不同的mysql也不一致，并且不同mysql变种(percona server) 也会不一致，即便版本都一致了，同一个slowlog中的不同记录格式也不尽相同，这就是它麻烦的地方

不过好在logstash有插件机制，使用grok可以通过正则的方式进行自定义，这样就灵活不少，可以根据具体的环境来调配以适应

> **Tip:** 写正则的过程，就是一个不断调校的过程，写完后，测试，再改，再测，再改......绝大部分条目可以匹配后，还要找点不同款的拿来测，尽量作到日志里的任意一条都能被匹配(当然换个版本，可能又得来一次，但方法不变)

这里分享一下logstash中处理mysql日志的配置过程，logstash中正则的相关内容可以参考 **[patterns][patterns]**  和 **[grok predifined patterns][grok_patterns]**

> **Tip:** 当前的最新版本为 **Logstash 2.1.1** 

---


# 概要

* TOC
{:toc}


---

## 环境

* percona server 5.6.27-75.0
* elasticsearch 2.1.1
* logstash 2.1.1
* kibana 4.3.1
* CentOS release 6.6 (Final)
* 2.6.32-504.el6.x86_64


---

## logstash配置

~~~
[root@h102 etc]# cat  logstash-multiline.conf 
input {
  stdin {
    codec => multiline {
      pattern => "^# User@Host:"
      negate => true
      what => previous
    }
  }
}


filter {
  grok {
	match => [ "message", "(?m)^#\s+User@Host:\s+%{USER:user}\[[^\]]+\]\s+@\s+%{USER:clienthost}\s+\[(?:%{IP:clientip})?\]\s+Id:\s+%{NUMBER:id:int}\n#\s+Schema:\s+%{USER:schema}\s+Last_errno:\s+%{NUMBER:lasterrorno:int}\s+Killed:\s+%{NUMBER:killedno:int}\n#\s+Query_time:\s+%{NUMBER:query_time:float}\s+Lock_time:\s+%{NUMBER:lock_time:float}\s+Rows_sent:\s+%{NUMBER:rows_sent:int}\s+Rows_examined:\s+%{NUMBER:rows_examined:int}\s+Rows_affected:\s+%{NUMBER:rows_affected:int}\n#\s+Bytes_sent:\s+%{NUMBER:bytes_sent:int}\n\s*(?:use\s+%{USER:usedatabase};\s*\n)?SET\s+timestamp=%{NUMBER:timestamp};\n\s*(?<query>(?<action>\w+)\b.*)\s*(?:\n#\s+Time)?.*$"]

  } 

  date {
    match => [ "timestamp", "UNIX" ]
    #remove_field => [ "timestamp" ]
  }
}

output {
  elasticsearch { 
  	hosts => ["localhost:9200"] 
        index=>"mysql-slow-log-%{+YYYY.MM.dd}"
	}
  stdout { codec => rubydebug }
}
[root@h102 etc]# 
~~~

### 检测配置

~~~
[root@h102 etc]# /opt/logstash/bin/logstash -f logstash-multiline.conf -t 
Configuration OK
[root@h102 etc]# 
~~~

### 运行logstash

~~~
[root@h102 etc]# /opt/logstash/bin/logstash -f logstash-multiline.conf 
Settings: Default filter workers: 1
Logstash startup completed
...
...
...
~~~

### 输入测试


随便在终端中贴入一段日志，要求完全覆盖完整的一条，然后观察输出

> **Tip:** 不能正好一条，要完全包含完整一条的首尾

~~~
{
       "@timestamp" => "2015-12-16T18:00:59.000Z",
          "message" => "# User@Host: taobao[taobao] @ regular_exp [192.168.35.23]  Id:  1236\n# Schema: bat_db  Last_errno: 0  Killed: 0\n# Query_time: 1.679745  Lock_time: 0.124872  Rows_sent: 0  Rows_examined: 292389  Rows_affected: 1066\n# Bytes_sent: 55\nSET timestamp=1450288859;\ncreate table temp_logstash_regular as\n  select t1.user_id, t2.user_key\n  from kibana_test_repo as t1\n  join users as t2\n  on t1.user_id = t2.id\n  where t1.notification_ts >= '2015-12-16 00:00:00' and\n        t1.notification_ts < '2015-12-17 00:00:00'\n  group by t1.user_id;\n# Time: 151217  2:01:01",
         "@version" => "1",
             "tags" => [
        [0] "multiline"
    ],
             "host" => "h102.temp",
             "user" => "taobao",
       "clienthost" => "regular_exp",
         "clientip" => "192.168.35.23",
               "id" => 1236,
           "schema" => "bat_db",
      "lasterrorno" => 0,
         "killedno" => 0,
       "query_time" => 1.679745,
        "lock_time" => 0.124872,
        "rows_sent" => 0,
    "rows_examined" => 292389,
    "rows_affected" => 1066,
       "bytes_sent" => 55,
        "timestamp" => "1450288859",
            "query" => "create table temp_logstash_regular as\n  select t1.user_id, t2.user_key\n  from kibana_test_repo as t1\n  join users as t2\n  on t1.user_id = t2.id\n  where t1.notification_ts >= '2015-12-16 00:00:00' and\n        t1.notification_ts < '2015-12-17 00:00:00'\n  group by t1.user_id;\n# Time: 151217  2:01:01",
           "action" => "create"
}
~~~

可以正常解析


> **Tip:** 如果无法正常解析, **tags** 里会多出一个 **_grokparsefailure** ，并且无法捕获下面多出来的那些值


~~~
{
    "@timestamp" => "2016-01-29T21:29:06.567Z",
       "message" => "# User@Host: taobao[taobao] @ regular_exp [192.168.35.23]  Id:  1236\\n# Schema: bat_db  Last_errno: 0  Killed: 0\\n# Query_time: 1.679745  Lock_time: 0.124872  Rows_sent: 0  Rows_examined: 292389  Rows_affected: 1066\\n# Bytes_sent: 55\\nSET timestamp=1450288859;\\ncreate table temp_logstash_regular as\\n  select t1.user_id, t2.user_key\\n  from kibana_test_repo as t1\\n  join users as t2\\n  on t1.user_id = t2.id\\n  where t1.notification_ts >= '2015-12-16 00:00:00' and\\n        t1.notification_ts < '2015-12-17 00:00:00'\\n  group by t1.user_id;\\n# Time: 151217  2:01:01",
      "@version" => "1",
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
  stdin {
    codec => multiline {
      pattern => "^# User@Host:"
      negate => true
      what => previous
    }
  }
}
~~~

logstash的流水线模型是 **intpu\|[filter]\|output**，其中 **filter** 部分为可选，但是处理mysql这种复杂的日志，没有filter，还真不行


Item     | Comment
-------- | ---
`input {`| 框定输入源的定义范围
`stdin {`|定义了一个输入源，使用 **[stdin][stdin]** 插件从标准输入读取数据，也就是终端读入(生产中不会这样配置，一般用来进行交互调试)
`codec => multiline {` | 使用 **[multiline][multiline]** 插件来进行处理，因为mysql的日志是多行的
`pattern => "^# User@Host:"`| 匹配以 **# User@Host:** 顶头的行
`negate => true`| 对上面的匹配进行反转(就是实际去匹配不以 **# User@Host:** 顶头的行)
`what => previous`| 把找到的行用于追加，追加到之前的日志条目中，就是认为这一行仍然属于前一个事件的内容


合起来的意思就是 : 所有不以 **# User@Host:** 顶头的条目都追加到以 **# User@Host:** 顶头的条目中，作为同一个事件看待



---

### filter 

~~~
filter {
  grok {
	match => [ "message", "(?m)^#\s+User@Host:\s+%{USER:user}\[[^\]]+\]\s+@\s+%{USER:clienthost}\s+\[(?:%{IP:clientip})?\]\s+Id:\s+%{NUMBER:id:int}\n#\s+Schema:\s+%{USER:schema}\s+Last_errno:\s+%{NUMBER:lasterrorno:int}\s+Killed:\s+%{NUMBER:killedno:int}\n#\s+Query_time:\s+%{NUMBER:query_time:float}\s+Lock_time:\s+%{NUMBER:lock_time:float}\s+Rows_sent:\s+%{NUMBER:rows_sent:int}\s+Rows_examined:\s+%{NUMBER:rows_examined:int}\s+Rows_affected:\s+%{NUMBER:rows_affected:int}\n#\s+Bytes_sent:\s+%{NUMBER:bytes_sent:int}\n\s*(?:use\s+%{USER:usedatabase};\s*\n)?SET\s+timestamp=%{NUMBER:timestamp};\n\s*(?<query>(?<action>\w+)\b.*)\s*(?:\n#\s+Time)?.*$"]

  } 

  date {
    match => [ "timestamp", "UNIX" ]
    #remove_field => [ "timestamp" ]
  }
}
~~~

**filter** 是整个mysql 日志处理的核心部分，就是通过它来抓取信息赋给各个filed

Item     | Comment
-------- | ---
`filter {`|框定处理逻辑的定义范围
`grok {`|定义了一个过滤器，使用 **[grok][grok]** 插件来解析文本，和抓取信息，用于文本结构化
`match => ["message",".*"]`| 用来match哈希 `{"message" => ".*patten.*"}`,然后把正则捕获的值作为事件日志的filed
`date {`| 定义了一个过滤器，使用 **[date][date]** 插件来从fileds中解析出时间，然后把获取的时间值作为此次事件日志的时间戳
`match => [ "timestamp", "UNIX" ]`|取用 **timestamp** 中的时间作为事件日志时间戳，模式匹配为**UNIX**
`#remove_field => [ "timestamp" ]`|一般而言，日志会有一个自己的时间戳 **@timestamp** ,这是logstash或 beats看到日志时的时间点，但是上一步已经将从日志捕获的时间赋给了 **@timestamp** ，所以 timestamp 就是一份冗余的信息,可以使用 **remove_field** 方法来删掉这个字段，但我选择保留



>The date filter is especially important for sorting events and for backfilling old data. If you don’t get the date correct in your event, then searching for them later will likely sort out of order.
>
>In the absence of this filter, logstash will choose a timestamp based on the first time it sees the event (at input time), if the timestamp is not already set in the event. For example, with file input, the timestamp is set to the time of each read.



---

### output 


~~~
output {
  elasticsearch { 
  	hosts => ["localhost:9200"] 
        index=>"mysql-slow-log-%{+YYYY.MM.dd}"
	}
  stdout { codec => rubydebug }
}
~~~


**output** 是经过加工和处理后，事件日志的去向


Item     | Comment
-------- | ---
`output {`|框定出口的定义范围
`elasticsearch {`|  定义了一个出口，使用 **[elasticsearch][elasticsearch]** 插件来进行输出，将结果输出到ES中
`hosts => ["localhost:9200"]`| 指定es的目标地址为 **localhost:9200**
`index=>"mysql-slow-log-%{+YYYY.MM.dd}"`| 指定存到哪个index，如不指定，默认为**logstash-%{+YYYY.MM.dd}**
`stdout { codec => rubydebug }`| 定义了一个出口，使用 **[stdout][stdout]** 插件将信息输出到标准输出，也就是终端，并且使用 **[rubydebug][rubydebug]** 插件处理过后进行展示，也就是行成jason格式 (生产中不会这样配置，一般用来进行交互调试)


---

### 正则


>`"(?m)^#\s+User@Host:\s+%{USER:user}\[[^\]]+\]\s+@\s+%{USER:clienthost}\s+\[(?:%{IP:clientip})?\]\s+Id:\s+%{NUMBER:id:int}\n#\s+Schema:\s+%{USER:schema}\s+Last_errno:\s+%{NUMBER:lasterrorno:int}\s+Killed:\s+%{NUMBER:killedno:int}\n#\s+Query_time:\s+%{NUMBER:query_time:float}\s+Lock_time:\s+%{NUMBER:lock_time:float}\s+Rows_sent:\s+%{NUMBER:rows_sent:int}\s+Rows_examined:\s+%{NUMBER:rows_examined:int}\s+Rows_affected:\s+%{NUMBER:rows_affected:int}\n#\s+Bytes_sent:\s+%{NUMBER:bytes_sent:int}\n\s*(?:use\s+%{USER:usedatabase};\s*\n)?SET\s+timestamp=%{NUMBER:timestamp};\n\s*(?<query>(?<action>\w+)\b.*)\s*(?:\n#\s+Time)?.*$"`


Item     | Comment
-------- | ---
`(?m)`| 打开多行模式的开关
`^#`|以 **#** 字符顶头
`\s+`| 匹配一个或多个空字符
`\s*`| 0个或多个空字符
`%{USER:user}`| 以 **USER** 模式进行正则匹配，结果放在user中
`\[[^\]]+\]`| 以 **[** 开头 以**]**结尾，内容是由一个或多个不是 **]** 的字符填充而成
`\[(?:%{IP:clientip})?\]`|以 **[** 开头 以**]**结尾，内容可能有，也可能无，如果有并且匹配 **IP** 的正则模式，结果放在clientip中
`%{NUMBER:id:int}`| 以 **NUMBER** 模式进行正则匹配，为整数型，结果放在id中
`\n`| 匹配换行符
`%{NUMBER:query_time:float}`| 以 **NUMBER** 模式进行正则匹配，为浮点型，结果放在query_time中
`(?:use\s+%{USER:usedatabase};\s*\n)?`| 这个匹配可能有，也可能无，如果有，就是以use开头，若干空字符，以 **USER** 模式进行正则匹配，结果放在usedatabase中，然后紧接着 **;** ，后面是0个或多个空字符，然后是换行，注意：如果有是整体有，如果无，是整体无
`\b`|代表字单词边界不占位置，只用来指示位置
`.*`| 尽可能多的任意匹配
`(?<query>(?<action>\w+)\b.*)`| 整体匹配，存到query中，以一个或多个字符开头组成的单词，结果存到action中
`(?:\n#\s+Time)?`|内容可能有，也可能无，如果有，是接在一个换行之后，以 **#** 开头，隔着一个或多个空字符，然后是Time
`.*$`|任意匹配直到结尾





预定义正则模式的相关内容可以参考 **[patterns][patterns]**  和 **[grok predifined patterns][grok_patterns]**



---

# 命令汇总


* **`cat  logstash-multiline.conf`**
* **`/opt/logstash/bin/logstash -f logstash-multiline.conf -t`**
* **`/opt/logstash/bin/logstash -f logstash-multiline.conf`**

---

# 附


这里预定义了一些正则表达式如：**USER、IP、NUMBER**

* `USERNAME [a-zA-Z0-9._-]+`

* `IP (?:%{IPV6}|%{IPV4})`

`IPV6 ((([0-9A-Fa-f]{1,4}:){7}([0-9A-Fa-f]{1,4}|:))|(([0-9A-Fa-f]{1,4}:){6}(:[0-9A-Fa-f]{1,4}|((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){5}(((:[0-9A-Fa-f]{1,4}){1,2})|:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){4}(((:[0-9A-Fa-f]{1,4}){1,3})|((:[0-9A-Fa-f]{1,4})?:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){3}(((:[0-9A-Fa-f]{1,4}){1,4})|((:[0-9A-Fa-f]{1,4}){0,2}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){2}(((:[0-9A-Fa-f]{1,4}){1,5})|((:[0-9A-Fa-f]{1,4}){0,3}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){1}(((:[0-9A-Fa-f]{1,4}){1,6})|((:[0-9A-Fa-f]{1,4}){0,4}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(:(((:[0-9A-Fa-f]{1,4}){1,7})|((:[0-9A-Fa-f]{1,4}){0,5}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:)))(%.+)?`

`IPV4 (?<![0-9])(?:(?:[0-1]?[0-9]{1,2}|2[0-4][0-9]|25[0-5])[.](?:[0-1]?[0-9]{1,2}|2[0-4][0-9]|25[0-5])[.](?:[0-1]?[0-9]{1,2}|2[0-4][0-9]|25[0-5])[.](?:[0-1]?[0-9]{1,2}|2[0-4][0-9]|25[0-5]))(?![0-9])`


* `NUMBER (?:%{BASE10NUM})`


`BASE10NUM (?<![0-9.+-])(?>[+-]?(?:(?:[0-9]+(?:\.[0-9]+)?)|(?:\.[0-9]+)))`




---


[patterns]:https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns
[grok_patterns]:https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns
[multiline]:https://www.elastic.co/guide/en/logstash/current/plugins-codecs-multiline.html
[grok]:https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
[date]:https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
[elasticsearch]:https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
[stdout]:https://www.elastic.co/guide/en/logstash/current/plugins-outputs-stdout.html
[stdin]:https://www.elastic.co/guide/en/logstash/current/plugins-inputs-stdin.html
[rubydebug]:https://www.elastic.co/guide/en/logstash/current/plugins-codecs-rubydebug.html

