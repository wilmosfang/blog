---
layout: post
title:  Kibana 基础
author: wilmosfang
tags:   kibana 
categories:   kibana 
wc: 472  1273 16087 
excerpt:  Kibana 相关基础与使用方法，下载安装，配置，启动，访问，导入数据，可以用来构建视图和仪表盘
comments: true
---



# 前言


**[Kibana][kibana]** 是设计用来与Elasticsearch配合进行数据分析与数据可视化的开源软件

>Kibana is an open source analytics and visualization platform designed to work with Elasticsearch. You use Kibana to search, view, and interact with data stored in Elasticsearch indices. You can easily perform advanced data analysis and visualize your data in a variety of charts, tables, and maps.


**[Kibana][kibana]** 可以与ES进行无缝对接，是ELK的重要组员，下面分享一下 **[Kibana][kibana]**  的基础使用方法，详细可以参阅 **[官方文档][kibana_doc]**


> **Tip:** 当前的最新版本为 **Kibana 4.3.1**

---


# 概要

* TOC
{:toc}



---

## 下载


**Kibana** 的 **[下载地址][kibana_download]**

~~~
[root@h101 kibana]# wget https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz
--2015-12-22 17:42:45--  https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz
Resolving download.elastic.co... 50.16.251.137, 50.17.224.164, 23.21.65.76, ...
Connecting to download.elastic.co|50.16.251.137|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 30408272 (29M) [application/octet-stream]
Saving to: “kibana-4.3.1-linux-x64.tar.gz”

100%[============================================================================================>] 30,408,272  15.6K/s   in 20m 39s 

2015-12-22 18:03:36 (24.0 KB/s) - “kibana-4.3.1-linux-x64.tar.gz” saved [30408272/30408272]

[root@h101 kibana]# sha1sum kibana-4.3.1-linux-x64.tar.gz 
115ba22882df75eb5f07330b7ad8781a57569b00  kibana-4.3.1-linux-x64.tar.gz
[root@h101 kibana]# 
~~~

可以与 **[kibana-4.3.1-linux-x64.tar.gz.sha1.txt][hash]** 相对照，避免不完整或不一致




---

## 解压安装


非常简单，只用解压，就相当于安装了

~~~
[root@h101 kibana]# ls
kibana-4.3.1-linux-x64.tar.gz
[root@h101 kibana]# tar -zxvf kibana-4.3.1-linux-x64.tar.gz 
kibana-4.3.1-linux-x64/
kibana-4.3.1-linux-x64/bin/
kibana-4.3.1-linux-x64/config/
kibana-4.3.1-linux-x64/installedPlugins/
kibana-4.3.1-linux-x64/LICENSE.txt
kibana-4.3.1-linux-x64/node/
kibana-4.3.1-linux-x64/node_modules/
kibana-4.3.1-linux-x64/optimize/
kibana-4.3.1-linux-x64/package.json
kibana-4.3.1-linux-x64/README.txt
kibana-4.3.1-linux-x64/src/
...
...
kibana-4.3.1-linux-x64/node/bin/node
kibana-4.3.1-linux-x64/node/bin/npm
kibana-4.3.1-linux-x64/config/kibana.yml
kibana-4.3.1-linux-x64/bin/kibana
kibana-4.3.1-linux-x64/bin/kibana.bat
[root@h101 kibana]# echo $?
0
[root@h101 kibana]# 
~~~

目录结构

~~~
[root@h101 kibana]# ll 
total 29700
drwxr-xr-x 10 cc   games     4096 Dec 16 22:57 kibana-4.3.1-linux-x64
-rw-r--r--  1 root root  30408272 Jan  8 23:18 kibana-4.3.1-linux-x64.tar.gz
[root@h101 kibana]# ll kibana-4.3.1-linux-x64
total 44
drwxr-xr-x  2 cc games 4096 Jan  8 23:28 bin
drwxr-xr-x  2 cc games 4096 Jan  8 23:28 config
drwxr-xr-x  2 cc games 4096 Dec 16 22:55 installedPlugins
-rw-r--r--  1 cc games  563 Dec 16 22:55 LICENSE.txt
drwxrwxr-x  6 cc games 4096 Jan  8 23:28 node
drwxr-xr-x 79 cc games 4096 Jan  8 23:28 node_modules
drwxr-xr-x  3 cc games 4096 Jan  8 23:28 optimize
-rw-r--r--  1 cc games  701 Dec 16 22:56 package.json
-rw-r--r--  1 cc games 2266 Dec 16 22:55 README.txt
drwxr-xr-x  8 cc games 4096 Jan  8 23:28 src
drwxr-xr-x  2 cc games 4096 Dec 16 22:55 webpackShims
[root@h101 kibana]# tree kibana-4.3.1-linux-x64/bin/ kibana-4.3.1-linux-x64/config/
kibana-4.3.1-linux-x64/bin/
├── kibana
└── kibana.bat
kibana-4.3.1-linux-x64/config/
└── kibana.yml

0 directories, 3 files
[root@h101 kibana]# 
~~~

---

## 配置

修改 **config/kibana.yml** 中的 **elasticsearch** 配置

~~~
[root@h101 kibana]# cd kibana-4.3.1-linux-x64/config/
[root@h101 config]# ls
kibana.yml
[root@h101 config]# grep -v "^#" kibana.yml | grep -v "^$"
[root@h101 config]# vim kibana.yml 
[root@h101 config]# grep -v "^#" kibana.yml | grep -v "^$"
elasticsearch.url: "http://h102:9200"
[root@h101 config]#
~~~

指向可用的ES

---

## 启动


直接使用 **bin/kibana** 就可以启动服务到前台运行

~~~
[root@h101 kibana-4.3.1-linux-x64]# bin/kibana 
  log   [23:42:21.246] [info][status][plugin:kibana] Status changed from uninitialized to green - Ready
  log   [23:42:21.327] [info][status][plugin:elasticsearch] Status changed from uninitialized to yellow - Waiting for Elasticsearch
  log   [23:42:21.359] [info][status][plugin:kbn_vislib_vis_types] Status changed from uninitialized to green - Ready
  log   [23:42:21.380] [info][status][plugin:markdown_vis] Status changed from uninitialized to green - Ready
  log   [23:42:21.397] [info][status][plugin:metric_vis] Status changed from uninitialized to green - Ready
  log   [23:42:21.407] [info][status][plugin:spyModes] Status changed from uninitialized to green - Ready
  log   [23:42:21.437] [info][status][plugin:statusPage] Status changed from uninitialized to green - Ready
  log   [23:42:21.458] [info][status][plugin:table_vis] Status changed from uninitialized to green - Ready
  log   [23:42:21.481] [info][status][plugin:elasticsearch] Status changed from yellow to green - Kibana index ready
  log   [23:42:21.510] [info][listening] Server running at http://0.0.0.0:5601
  ...
  ...
~~~

会默认监听在 **0.0.0.0:5601** 


> **Note:** 确认本地的 **5601** 是开放状态

~~~
[root@h101 ~]# iptables -L -nv | grep 5601
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:5601 
[root@h101 ~]# 
~~~

---

## 访问


在浏览器中输入 **http://192.168.100.101:5601/** 就可以成功访问了

![kibana.png](/images/kibana/kibana.png)

查看 Kibana 的状态

![kibana_status.png](/images/kibana/kibana_status.png)

---

## 导入实验数据


准备数据

~~~
[root@h101 date]# curl -XPUT http://h102:9200/shakespeare -d '{
 "mappings" : {
  "_default_" : {
   "properties" : {
    "speaker" : {"type": "string", "index" : "not_analyzed" },
    "play_name" : {"type": "string", "index" : "not_analyzed" },
    "line_id" : { "type" : "integer" },
    "speech_number" : { "type" : "integer" }
   }
  }
 }
}
';
{"acknowledged":true}[root@h101 date]#
[root@h101 date]# curl -XPUT http://h102:9200/logstash-2015.05.18 -d '{
>   "mappings": {
>     "log": {
>       "properties": {
>         "geo": {
>           "properties": {
>             "coordinates": {
>               "type": "geo_point"
>             }
>           }
>         }
>       }
>     }
>   }
> }
> ';
{"acknowledged":true}[root@h101 date]# curl -XPUT http://h102:9200/logstash-2015.05.19 -d '
> {
>   "mappings": {
>     "log": {
>       "properties": {
>         "geo": {
>           "properties": {
>             "coordinates": {
>               "type": "geo_point"
>             }
>           }
>         }
>       }
>     }
>   }
> }
> ';
{"acknowledged":true}[root@h101 date]#
[root@h101 date]# curl -XPUT http://h102:9200/logstash-2015.05.20 -d '
> {
>   "mappings": {
>     "log": {
>       "properties": {
>         "geo": {
>           "properties": {
>             "coordinates": {
>               "type": "geo_point"
>             }
>           }
>         }
>       }
>     }
>   }
> }
> ';
{"acknowledged":true}[root@h101 date]#
[root@h101 date]# ls
accounts.zip  logs.jsonl.gz  shakespeare.json
[root@h101 date]# unzip accounts.zip 
Archive:  accounts.zip
  inflating: accounts.json           
[root@h101 date]# gunzip logs.jsonl.gz 
[root@h101 date]# ls
accounts.json  accounts.zip  logs.jsonl  shakespeare.json
[root@h101 date]# du -s * 
240	accounts.json
60	accounts.zip
52100	logs.jsonl
24628	shakespeare.json
[root@h101 date]#
~~~


> **Tip:** 实验数据的地址，可以参考 **[Getting Started][getting-started]** ， 可以使用 **wget** 获取

~~~
https://www.elastic.co/guide/en/kibana/3.0/snippets/shakespeare.json
https://github.com/bly2k/files/blob/master/accounts.zip?raw=true
https://download.elastic.co/demos/kibana/gettingstarted/logs.jsonl.gz
~~~


导入数据

~~~
[root@h101 date]# curl -XPOST 'h102:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
...
...
[root@h101 date]# curl -XPOST 'h102:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json > /dev/null
...
...
[root@h101 date]# curl -XPOST 'h102:9200/_bulk?pretty' --data-binary @logs.jsonl  > /dev/null
...
...
[root@h101 date]#
~~~


> **Tip:** 之所以加上 **`> /dev/null`** 是因为如果不加，会刷屏，相当影响加载速度

再看一眼ES状态

~~~
[root@h101 date]# curl 'h102:9200/_cat/indices?v'
health status index                           pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   logstash-2015.05.19               5   1       4624            0     30.8mb         30.8mb 
yellow open   filebeat-2015.12.24               5   1       3182            0        1mb            1mb 
yellow open   logstash-2015.05.18               5   1       4631            0     28.9mb         28.9mb 
yellow open   logstash-2016.12.22               5   1          1            0     11.6kb         11.6kb 
yellow open   logstash-2016.12.23               5   1          3            0     33.8kb         33.8kb 
yellow open   logstash-2016.01.05               5   1         42            0    214.5kb        214.5kb 
yellow open   filebeat-2016.01.05               5   1       4198            0    910.5kb        910.5kb 
yellow open   logstash-2015.12.23               5   1        100            0    235.8kb        235.8kb 
yellow open   logstash-2015.12.22               5   1         41            0    126.5kb        126.5kb 
yellow open   %{[@metadata][beat]}-2016.01.05   5   1         71            0    100.1kb        100.1kb 
yellow open   logstash-2015.05.20               5   1       4750            0     28.5mb         28.5mb 
yellow open   .kibana                           1   1         94            0    102.3kb        102.3kb 
yellow open   bank                              5   1       1000            0    451.1kb        451.1kb 
yellow open   shakespeare                       5   1     111396            0     18.9mb         18.9mb 
[root@h101 date]# 
~~~

可见 **bank、shakespeare、logstash-2015.05.18、logstash-2015.05.19、logstash-2015.05.20** 都已经加载进来了，其它的是我自己生成的数据，不用理会


---

## 简单使用

由于kibana是一个数据可视化工具，绝大部操作都是在 WEB GUI 里完成，如果使用Blog的形式进行展示会产生大量的截图，并且文字描述起来比较吃力，此类最好的演示形式其实是视频，但由于条件有限，这里只给出官方文档的链接

(我这是在给自己的偷懒找一个合理的解释)


有了以上的测试数据后，下面的试验都可以在WEB GUI里完成

**[定义索引与检索数据][tutorial_define_index]**

定义索引匹配和简单检索数据的方法 

**[数据图形化][tutorial_visualizing]**

简单的饼状图，柱状图，GIS分布图(地图)的数据展示方法

**[仪表板][tutorial_dashboard]**

仪表板的拼接方法

> **Tip:** 有时啃啃官网，才能体会到，外国人写文档要严谨很多

看完了基础方法就可以开始使用了

---

## 高级使用


想更深入地了解每一个功能细节，还是要看看高级用法

**[插件][kibana_plugins]**

kibana也有插件机制，目前来看还是比较方便的，只是没有 **list** 命令，不过可以通过网页的状态查看

~~~
[root@h101 kibana-4.3.1-linux-x64]# bin/kibana plugin --help 

  Usage: plugin [options]

  Maintain Plugins

  Options:

    -h, --help                              output usage information
    -i, --install <org>/<plugin>/<version>  The plugin to install
    -r, --remove <plugin>                   The plugin to remove
    -q, --quiet                             Disable all process messaging except errors
    -s, --silent                            Disable all process messaging
    -u, --url <url>                         Specify download url
    -c, --config <path>                     Path to the config file
    -t, --timeout <duration>                Length of time before failing; 0 for never fail
    -d, --plugin-dir <path>                 The path to the directory where plugins are stored

[root@h101 kibana-4.3.1-linux-x64]#
~~~

**[检索数据][discover]**

基本的检索方法，时间范围的设定，自动刷新周期的设定，展示结果的分享，结果的保存，过滤条件的设定，jason定义条件，文档的内容查看，字段的统计设定，


**[数据图形化][visualize]**

主要分三步：

* 1.选择图形类别
* 2.选择数据源
* 3.进行聚合定制

图形类别目前有 **区块型，表格型，线型，Markdown型，数值型，饼型，拼图(GIS地图)型，直方图型**

还有过滤条件的设定，jason定义条件，自动刷新周期的设定


**[仪表板][dashboard]**

仪表板的拼接，页面的布局，过滤条件的设定，jason定义条件，自动刷新周期的设定


![kibana_dashboard.png](/images/kibana/kibana_dashboard.png)


**[设置][settings]**

索引的设定重载与删除，时间格式，默认索引的设定，管理字段，字段的格式定义，颜色定义，脚本字段定义，高级配置，服务参数配置，对象管理

**[安全与生产][production]**

配置Kibana对es的用户密码访问，启用SSL通讯，ES的细粒度权限管控，kibana对es的负载均衡方法

---

# 命令汇总

* **`wget https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz`**
* **`sha1sum kibana-4.3.1-linux-x64.tar.gz`**
* **`tar -zxvf kibana-4.3.1-linux-x64.tar.gz`**
* **`tree kibana-4.3.1-linux-x64/bin/ kibana-4.3.1-linux-x64/config/`**
* **`vim kibana.yml`**
* **`grep -v "^#" kibana.yml | grep -v "^$"`**
* **`bin/kibana`**
* **`iptables -L -nv | grep 5601`**
* **`curl -XPUT http://h102:9200/shakespeare -d '{`**
* **`curl -XPUT http://h102:9200/logstash-2015.05.18 -d '{`**
* **`unzip accounts.zip`**
* **`gunzip logs.jsonl.gz`**
* **`curl -XPOST 'h102:9200/bank/account/_bulk?pretty' --data-binary @accounts.json`**
* **`curl -XPOST 'h102:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json > /dev/null`**
* **`curl -XPOST 'h102:9200/_bulk?pretty' --data-binary @logs.jsonl  > /dev/null`**
* **`curl 'h102:9200/_cat/indices?v'`**
* **`bin/kibana plugin --help`**


---

[kibana]:https://www.elastic.co/products/kibana
[kibana_doc]:https://www.elastic.co/guide/en/kibana/current/index.html
[hash]:https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz.sha1.txt
[kibana_download]:https://www.elastic.co/downloads/kibana
[getting-started]:https://www.elastic.co/guide/en/kibana/current/getting-started.html
[tutorial_define_index]:https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html
[tutorial_visualizing]:https://www.elastic.co/guide/en/kibana/current/tutorial-visualizing.html
[tutorial_dashboard]:https://www.elastic.co/guide/en/kibana/current/tutorial-dashboard.html
[kibana_plugins]:https://www.elastic.co/guide/en/kibana/current/kibana-plugins.html
[discover]:https://www.elastic.co/guide/en/kibana/current/discover.html
[visualize]:https://www.elastic.co/guide/en/kibana/current/visualize.html
[dashboard]:https://www.elastic.co/guide/en/kibana/current/dashboard.html
[settings]:https://www.elastic.co/guide/en/kibana/current/settings.html
[production]:https://www.elastic.co/guide/en/kibana/current/production.html

