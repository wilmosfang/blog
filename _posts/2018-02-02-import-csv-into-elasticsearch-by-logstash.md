---
layout: post
title: "import CSV into Elasticsearch by Logstash"
author:  wilmosfang
date: 2018-02-02 15:42:14
image: '/assets/img/'
excerpt: '使用 Logstash 将 CSV 导入 Elasticsearch'
main-class: es
color: '#51bcb2'
tags:
 - logstash
 - es
categories:
 - es
twitter_text: 'import CSV into Elasticsearch'
introduction: 'use Logstash to import CSV file'
---



# 前言

**[Logstash][logstash]** 是一个开源的数据收集加工和传输软件

常与 Elasticsearch 和 Kibana 一起组成 ELK 技术栈，给日志分析带来极大的便利

![logstash](/assets/img/logstash/logstash01.png)

这里分享一下使用 **[Logstash][logstash]** 将 CSV 导入 Elasticsearch 的方法

参考 **[CSV][logstash_csv]** 和 **[date][logstash_date]** 插件

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

## 准备 CSV 文件

~~~
[root@much es]# for i in {10..30} ; do  echo `date +"%Y/01/$i %H:%M:%S"`",$i-1,$i+1,$i+5"; done > test.csv
[root@much es]# vim test.csv
[root@much es]# cat test.csv
a,b,c,d
2018/01/10 23:54:53,10-1,10+1,10+5
2018/01/11 23:54:53,11-1,11+1,11+5
2018/01/12 23:54:53,12-1,12+1,12+5
2018/01/13 23:54:53,13-1,13+1,13+5
2018/01/14 23:54:53,14-1,14+1,14+5
2018/01/15 23:54:53,15-1,15+1,15+5
2018/01/16 23:54:53,16-1,16+1,16+5
2018/01/17 23:54:53,17-1,17+1,17+5
2018/01/18 23:54:53,18-1,18+1,18+5
2018/01/19 23:54:53,19-1,19+1,19+5
2018/01/20 23:54:53,20-1,20+1,20+5
2018/01/21 23:54:53,21-1,21+1,21+5
2018/01/22 23:54:53,22-1,22+1,22+5
2018/01/23 23:54:53,23-1,23+1,23+5
2018/01/24 23:54:53,24-1,24+1,24+5
2018/01/25 23:54:53,25-1,25+1,25+5
2018/01/26 23:54:53,26-1,26+1,26+5
2018/01/27 23:54:53,27-1,27+1,27+5
2018/01/28 23:54:53,28-1,28+1,28+5
2018/01/29 23:54:53,29-1,29+1,29+5
2018/01/30 23:54:53,30-1,30+1,30+5
[root@much es]#
~~~


这个文件只有四列 **a,b,c,d**

其中 a 列包含了时间戳

## 准备配置文件

~~~
[root@much es]# vim test.conf
[root@much es]# cat test.conf
input {
  file {
    path => "/root/es/test.csv"
    start_position => "beginning"
  }
}
filter {
  csv {
     separator => ","
     columns => ["a","b","c","d"]
  }
  date{
     match => [ "a", "yyyy/MM/dd HH:mm:ss" ]
  }
}
output {
  elasticsearch {
     hosts => "http://localhost:9200"
     index => "abcdjustfortest"
     document_type => "csv"
  }
  stdout {codec => rubydebug}
}
[root@much es]#
~~~


这里有几个处理点

* 使用 file 的 input 插件指定文件位置和开始位置
* 使用 csv 的 filter 插件指明分隔符和列名
* 使用 date 的 filter 插件指明时间戳记的格式，将 a 列中的数据取出匹配为此条信息的时间戳记
* 使用 elasticsearch 的 output 插件指明 es 的位置和索引位置
* 同时以 rubydebug 的方式在 console 终端中打印出解析过后的数据



## 指定配置运行

~~~
[root@much es]# /usr/share/logstash/bin/logstash -f test.conf
WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
Could not find log4j2 configuration at path /usr/share/logstash/config/log4j2.properties. Using default config which logs errors to the console
{
      "@version" => "1",
             "c" => "c",
             "a" => "a",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "a,b,c,d",
             "d" => "d",
             "b" => "b",
          "tags" => [
        [0] "_dateparsefailure"
    ],
    "@timestamp" => 2018-02-02T15:58:05.969Z
}
{
      "@version" => "1",
             "c" => "10+1",
             "a" => "2018/01/10 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/10 23:54:53,10-1,10+1,10+5",
             "d" => "10+5",
             "b" => "10-1",
    "@timestamp" => 2018-01-10T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "11+1",
             "a" => "2018/01/11 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/11 23:54:53,11-1,11+1,11+5",
             "d" => "11+5",
             "b" => "11-1",
    "@timestamp" => 2018-01-11T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "12+1",
             "a" => "2018/01/12 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/12 23:54:53,12-1,12+1,12+5",
             "d" => "12+5",
             "b" => "12-1",
    "@timestamp" => 2018-01-12T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "13+1",
             "a" => "2018/01/13 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/13 23:54:53,13-1,13+1,13+5",
             "d" => "13+5",
             "b" => "13-1",
    "@timestamp" => 2018-01-13T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "14+1",
             "a" => "2018/01/14 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/14 23:54:53,14-1,14+1,14+5",
             "d" => "14+5",
             "b" => "14-1",
    "@timestamp" => 2018-01-14T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "15+1",
             "a" => "2018/01/15 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/15 23:54:53,15-1,15+1,15+5",
             "d" => "15+5",
             "b" => "15-1",
    "@timestamp" => 2018-01-15T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "16+1",
             "a" => "2018/01/16 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/16 23:54:53,16-1,16+1,16+5",
             "d" => "16+5",
             "b" => "16-1",
    "@timestamp" => 2018-01-16T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "17+1",
             "a" => "2018/01/17 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/17 23:54:53,17-1,17+1,17+5",
             "d" => "17+5",
             "b" => "17-1",
    "@timestamp" => 2018-01-17T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "18+1",
             "a" => "2018/01/18 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/18 23:54:53,18-1,18+1,18+5",
             "d" => "18+5",
             "b" => "18-1",
    "@timestamp" => 2018-01-18T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "19+1",
             "a" => "2018/01/19 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/19 23:54:53,19-1,19+1,19+5",
             "d" => "19+5",
             "b" => "19-1",
    "@timestamp" => 2018-01-19T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "20+1",
             "a" => "2018/01/20 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/20 23:54:53,20-1,20+1,20+5",
             "d" => "20+5",
             "b" => "20-1",
    "@timestamp" => 2018-01-20T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "21+1",
             "a" => "2018/01/21 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/21 23:54:53,21-1,21+1,21+5",
             "d" => "21+5",
             "b" => "21-1",
    "@timestamp" => 2018-01-21T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "22+1",
             "a" => "2018/01/22 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/22 23:54:53,22-1,22+1,22+5",
             "d" => "22+5",
             "b" => "22-1",
    "@timestamp" => 2018-01-22T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "23+1",
             "a" => "2018/01/23 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/23 23:54:53,23-1,23+1,23+5",
             "d" => "23+5",
             "b" => "23-1",
    "@timestamp" => 2018-01-23T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "24+1",
             "a" => "2018/01/24 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/24 23:54:53,24-1,24+1,24+5",
             "d" => "24+5",
             "b" => "24-1",
    "@timestamp" => 2018-01-24T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "25+1",
             "a" => "2018/01/25 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/25 23:54:53,25-1,25+1,25+5",
             "d" => "25+5",
             "b" => "25-1",
    "@timestamp" => 2018-01-25T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "26+1",
             "a" => "2018/01/26 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/26 23:54:53,26-1,26+1,26+5",
             "d" => "26+5",
             "b" => "26-1",
    "@timestamp" => 2018-01-26T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "27+1",
             "a" => "2018/01/27 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/27 23:54:53,27-1,27+1,27+5",
             "d" => "27+5",
             "b" => "27-1",
    "@timestamp" => 2018-01-27T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "28+1",
             "a" => "2018/01/28 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/28 23:54:53,28-1,28+1,28+5",
             "d" => "28+5",
             "b" => "28-1",
    "@timestamp" => 2018-01-28T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "29+1",
             "a" => "2018/01/29 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/29 23:54:53,29-1,29+1,29+5",
             "d" => "29+5",
             "b" => "29-1",
    "@timestamp" => 2018-01-29T15:54:53.000Z
}
{
      "@version" => "1",
             "c" => "30+1",
             "a" => "2018/01/30 23:54:53",
          "host" => "much",
          "path" => "/root/es/test.csv",
       "message" => "2018/01/30 23:54:53,30-1,30+1,30+5",
             "d" => "30+5",
             "b" => "30-1",
    "@timestamp" => 2018-01-30T15:54:53.000Z
}
...
...
...

~~~

符合预期

可以再去 kibana 里看看数据内容

选择索引

![kibana](/assets/img/kibana/kibana03.png)

选择时间字段

![kibana](/assets/img/kibana/kibana04.png)

选择合适的时间区间，浏览数据

![kibana](/assets/img/kibana/kibana05.png)

查看解析的数据字段

![kibana](/assets/img/kibana/kibana06.png)

以 JSON 格式查看数据

![kibana](/assets/img/kibana/kibana07.png)



---

# 总结


这个简单的实例，很好地演示了 logstash 的管道处理模型

![logstash](/assets/img/logstash/basic_logstash_pipeline.png)


* TOC
{:toc}


---

[logstash]:https://www.elastic.co/products/logstash
[logstash_csv]:https://www.elastic.co/guide/en/logstash/6.1/plugins-filters-csv.html
[logstash_date]:https://www.elastic.co/guide/en/logstash/6.1/plugins-filters-date.html
