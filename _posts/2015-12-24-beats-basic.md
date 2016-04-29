---
layout: post
title:  Beats 基础
author: wilmosfang
categories: linux elasticsearch logstash beats filebeat
wc: 383  1241 13946 
excerpt:  Beats 基础概念与使用方法
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

下面对 **[filebeat][filebeat]** 的基础操作进行分享，详细内容可以参考 **[官方文档][filebeat_doc]**

**ELK** 的 **[集成文档][elk]**


> **Tip:** 当前的版本为 **Filebeat 1.0.1** 

---



# 概要

* TOC
{:toc}



---

## 架构

beats platform

![beats-platform.png](/images/beats/beats-platform.png)

filebeat

![filebeat.png](/images/beats/filebeat.png)




beats是一个使用Golang构建的平台，libbeat是其核心库，用来提供API进行与Elasticsearch，Logstash的连接，还能配置输入特性和实现信息收集等工作

filebeat是构建于beats之上的，应用于日志收集场景的实现

>When you start Filebeat, it starts one or more prospectors that look in the paths you’ve specified for log files. For each log file that the prospector locates, Filebeat starts a harvester. Each harvester reads a single log file for new content and sends the new log data to the spooler, which aggregates the events and sends the aggregated data to the output that you’ve configured for Filebeat.

---

## 安装

~~~
[root@h102 filebeat]# curl -L -O https://download.elastic.co/beats/filebeat/filebeat-1.0.1-x86_64.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 3622k  100 3622k    0     0   9340      0  0:06:37  0:06:37 --:--:-- 14275
[root@h102 filebeat]# ls
filebeat-1.0.1-x86_64.rpm
[root@h102 filebeat]# sha1sum filebeat-1.0.1-x86_64.rpm 
1e9c3e52a9bcd938a2f790bd0f0df728c076ab0e  filebeat-1.0.1-x86_64.rpm
[root@h102 filebeat]# du -sh filebeat-1.0.1-x86_64.rpm 
3.6M	filebeat-1.0.1-x86_64.rpm
[root@h102 filebeat]# rpm -ivh filebeat-1.0.1-x86_64.rpm 
Preparing...                ########################################### [100%]
   1:filebeat               ########################################### [100%]
[root@h102 filebeat]# 
~~~



---

## 配置


~~~
[root@h102 filebeat]# tree /etc/filebeat/
/etc/filebeat/
├── filebeat.template.json
└── filebeat.yml

0 directories, 2 files
[root@h102 filebeat]# grep -v "#" /etc/filebeat/filebeat.yml  | grep -v "^$"
filebeat:
  prospectors:
    -
      paths:
        - /var/log/*.log
      input_type: log
  registry_file: /var/lib/filebeat/registry
output:
  elasticsearch:
    hosts: ["localhost:9200"]
shipper:
logging:
  files:
[root@h102 filebeat]# vim  /etc/filebeat/filebeat.yml
[root@h102 filebeat]# grep -v "#" /etc/filebeat/filebeat.yml  | grep -v "^$"
filebeat:
  prospectors:
    -
      paths:
        - /var/log/*.log
        - /var/log/messages*
      input_type: log
  registry_file: /var/lib/filebeat/registry
output:
  logstash:
    hosts: ["localhost:5044"]
shipper:
logging:
  files:
[root@h102 filebeat]# 
~~~

在默认配置的基础上加入 **/var/log/messages** 以监控系统日志

将输出由ES改为了logstash

相关配置详情可以参看 **[Configuration Options][beats_conf]**


> **Note:** Make sure a file is not defined more than once across all prospectors because this can lead to unexpected behaviour


---

## 导入索引模板到ES

~~~
[root@h102 filebeat]# ls
filebeat.template.json  filebeat.yml
[root@h102 filebeat]# curl -XPUT 'http://localhost:9200/_template/filebeat?pretty' -d@/etc/filebeat/filebeat.template.json
{
  "acknowledged" : true
}
[root@h102 filebeat]# echo $?
0
[root@h102 filebeat]# 
~~~

---

## 运行filebeat 

~~~
[root@h102 filebeat]# /etc/init.d/filebeat start 
Starting filebeat:                                         [  OK  ]
[root@h102 filebeat]# /etc/init.d/filebeat status
filebeat-god (pid  2852) is running...
[root@h102 filebeat]# ps faux | grep filebeat
root      2870  0.0  0.0 103256   828 pts/0    S+   21:48   0:00  |       \_ grep filebeat
root      2852  0.0  0.0  11388   232 pts/0    Sl   21:47   0:00 filebeat-god -r / -n -p /var/run/filebeat.pid -- /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  0.5  0.6 237664 12920 pts/0    Sl   21:47   0:00  \_ /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
[root@h102 filebeat]# 
~~~

线程信息

~~~
[root@h102 filebeat]# ps -Lf 2852
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root      2852     1  2852  0    2 21:47 pts/0    Sl     0:00 filebeat-god -r / -n -p /var/run/filebeat.pid -- /usr/bin/filebeat -c /e
root      2852     1  2854  0    2 21:47 pts/0    Sl     0:00 filebeat-god -r / -n -p /var/run/filebeat.pid -- /usr/bin/filebeat -c /e
[root@h102 filebeat]# ps -Lf 2853
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root      2853  2852  2853  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2855  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2856  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2857  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2858  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2859  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2860  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2861  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
root      2853  2852  2862  0    9 21:47 pts/0    Sl     0:00 /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
[root@h102 filebeat]# pstree -ap 2852
filebeat-god,2852 -r / -n -p /var/run/filebeat.pid -- /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
  ├─filebeat,2853 -c /etc/filebeat/filebeat.yml
  │   ├─{filebeat},2855
  │   ├─{filebeat},2856
  │   ├─{filebeat},2857
  │   ├─{filebeat},2858
  │   ├─{filebeat},2859
  │   ├─{filebeat},2860
  │   ├─{filebeat},2861
  │   └─{filebeat},2862
  └─{filebeat-god},2854
[root@h102 filebeat]# 
~~~

---

## 配置与运行logstash

~~~
[root@h102 etc]# /opt/logstash/bin/logstash -f logstash-filebeat-es-simple.conf -t 
Configuration OK
[root@h102 etc]# cat logstash-filebeat-es-simple.conf
input {
	stdin{}
	beats{port => 5044}
}
output {
	elasticsearch {
		hosts=>"localhost:9200"
		index=>"%{[@metadata][beat]}-%{+YYYY.MM.dd}"
		document_type => "%{[@metadata][type]}"
	}
	stdout {codec=>rubydebug}
}
[root@h102 etc]# /opt/logstash/bin/logstash -f logstash-filebeat-es-simple.conf 
Settings: Default filter workers: 1
Logstash startup completed
...
...
~~~


查看端口信息

~~~
[root@h102 ~]# netstat  -ant | grep 5044
tcp        0      0 :::5044                     :::*                        LISTEN      
[root@h102 ~]# lsof -i :5044
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    3518 root    5u  IPv6  26614      0t0  TCP *:lxi-evntsvc (LISTEN)
[root@h102 ~]# pstree -ap 3518
java,3518 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -Djava.awt.headless=true -XX:CMSInitiatingOccupancyFraction=75-XX:+UseCMSIniti
  ├─{java},3535
  ├─{java},3536
  ├─{java},3537
  ├─{java},3538
  ├─{java},3539
  ├─{java},3540
  ├─{java},3541
  ├─{java},3542
  ├─{java},3543
  ├─{java},3544
  ├─{java},3545
  ├─{java},3546
  ├─{java},3547
  ├─{java},3548
  ├─{java},3550
  ├─{java},3551
  ├─{java},3553
  ├─{java},3554
  ├─{java},3555
  ├─{java},3556
  ├─{java},3557
  └─{java},3558
[root@h102 ~]#
[root@h102 ~]# ps -faux |grep  3518
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root      3577  0.0  0.0 103256   828 pts/2    S+   22:58   0:00  |       \_ grep 3518
root      3518 12.9  9.5 2516488 183444 pts/3  Sl+  22:53   0:36          \_ /usr/bin/java -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -Djava.awt.headless=true -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -Xmx1g -Xss2048k -Djffi.boot.library.path=/opt/logstash/vendor/jruby/lib/jni -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -Djava.awt.headless=true -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/logstash/heapdump.hprof -Xbootclasspath/a:/opt/logstash/vendor/jruby/lib/jruby.jar -classpath : -Djruby.home=/opt/logstash/vendor/jruby -Djruby.lib=/opt/logstash/vendor/jruby/lib -Djruby.script=jruby -Djruby.shell=/bin/sh org.jruby.Main --1.9 /opt/logstash/lib/bootstrap/environment.rb logstash/runner.rb agent -f logstash-filebeat-es-simple.conf
[root@h102 ~]# 
~~~

---

## 查看信息


logstash的配置中加入了 **stdout {codec=>rubydebug}** 是为了方便在终端监视信息(在实际应用中完全没有必要)，经过一番刷屏，最终停了下来

产生了大量如下格式的输出

~~~
...
...
{
       "message" => "Dec 24 22:49:08 h102 filebeat[3383]: registrar.go:157: Registry file updated. 0 states written.",
      "@version" => "1",
    "@timestamp" => "2015-12-24T14:59:26.133Z",
          "beat" => {
        "hostname" => "h102.temp",
            "name" => "h102.temp"
    },
         "count" => 1,
        "fields" => nil,
    "input_type" => "log",
        "offset" => 246594,
        "source" => "/var/log/messages",
          "type" => "log",
          "host" => "h102.temp"
}
{
       "message" => "Dec 24 22:49:08 h102 filebeat[3383]: beat.go:143: Cleaning up filebeat before shutting down.",
      "@version" => "1",
    "@timestamp" => "2015-12-24T14:59:26.133Z",
          "beat" => {
        "hostname" => "h102.temp",
            "name" => "h102.temp"
    },
         "count" => 1,
        "fields" => nil,
    "input_type" => "log",
        "offset" => 246690,
        "source" => "/var/log/messages",
          "type" => "log",
          "host" => "h102.temp"
}
...
...
~~~

再看ES里的信息

~~~
[root@h102 ~]# curl localhost:9200/_cat/indices?v
health status index               pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   logstash-2015.12.23   5   1        100            0    235.8kb        235.8kb 
yellow open   filebeat-2015.12.24   5   1       3175            0   1018.6kb       1018.6kb 
yellow open   logstash-2015.12.22   5   1         41            0    126.5kb        126.5kb 
yellow open   .kibana               1   1          2            0      8.3kb          8.3kb 
[root@h102 ~]# 
~~~

发现已经产生了 **`filebeat-*`** 的index，并且有相当数量的文档


接下来只用导入模板和 **index pattern** 到 **kibana** 里，然后选择默认index 为 **`[filebeat-]YYYY.MM.DD`** ，就可以在界面里进行查看和检索了

---

# 命令汇总

* **`curl -L -O https://download.elastic.co/beats/filebeat/filebeat-1.0.1-x86_64.rpm`**
* **`sha1sum filebeat-1.0.1-x86_64.rpm`**
* **`rpm -ivh filebeat-1.0.1-x86_64.rpm`**
* **`tree /etc/filebeat/`**
* **`vim  /etc/filebeat/filebeat.yml`**
* **`grep -v "#" /etc/filebeat/filebeat.yml  | grep -v "^$"`**
* **`curl -XPUT 'http://localhost:9200/_template/filebeat?pretty' -d@/etc/filebeat/filebeat.template.json`**
* **`/etc/init.d/filebeat start`**
* **`ps -Lf 2852`**
* **`ps -Lf 2853`**
* **`pstree -ap 2852`**
* **`/opt/logstash/bin/logstash -f logstash-filebeat-es-simple.conf -t`**
* **`cat logstash-filebeat-es-simple.conf`**
* **`/opt/logstash/bin/logstash -f logstash-filebeat-es-simple.conf`**
* **`netstat  -ant | grep 5044`**
* **`lsof -i :5044`**
* **`pstree -ap 3518`**
* **`curl localhost:9200/_cat/indices?v`**

---

[beats]:https://www.elastic.co/products/beats
[packetbeat]:https://www.elastic.co/downloads/beats/packetbeat
[topbeat]:https://www.elastic.co/downloads/beats/topbeat
[filebeat]:https://www.elastic.co/downloads/beats/filebeat
[filebeat_doc]:https://www.elastic.co/guide/en/beats/filebeat/current/index.html
[elk]:https://www.elastic.co/guide/en/beats/libbeat/1.0.1/getting-started.html
[beats_doc]:https://www.elastic.co/guide/en/beats/libbeat/1.0.1/index.html
[beats_conf]:https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-configuration-details.html
