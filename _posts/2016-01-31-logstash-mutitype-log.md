---
layout: post
title:    Logstash 处理多种格式日志
categories:  linux elasticsearch logstash beats filebeat 
wc: 229  459 7789 
excerpt: logstash的配置，filebeat的配置，建议启动顺序，与多类型日志输入的注意事项
comments: true
---



#前言


生产环境下使用 **logstash** 经常会遇到多种格式的日志，比如 mongodb 的慢日志，nginx或apache的访问日志，系统syslog日志，mysql的慢日志等

不同日志的解析方法不一样，产生的输出也不一样，如果使用同一个 input\|filter\|output 流必将导致混乱，最常见的问题就是日志无法获得正确解析 ，message中的内容还是一整条，并没有从中捕获出我们关心的域值，依旧是schemaless的状态，同时tags中会产生 **_grokparsefailure** 的标记，然后多种日志都存到了同一个index中，混乱不以，给后期的日志分析也带来不便

logstash提供了一种判断机制来对不同内容进行判断，然后分别处理

这里简单分享一下 **logstash** 中同时处理 **mysql慢日志** 和 **nginx访问日志** 的配置过程，相关的详细内容可以参考 **[Event Dependent Configuration][edc]** 和 **[Logstash Configuration Examples][lce]**


> **Tip:** 当前的最新版本为 **Logstash 2.1.1** 、**filebeat-1.0.1**

---


#概要

* TOC
{:toc}



---

##软件版本

* percona server 5.6.27-75.0 #是这个版本产生的慢日志，其它版本使用下面的正则，不一定能正常解析
* elasticsearch 2.1.1
* logstash 2.1.1
* filebeat-1.0.1


---

##logstash配置


{% highlight bash %}
[root@logstash-server conf.d]# cat filebeat-logstash-es.conf 
input {
	
	beats{
		port => 5077
		tags => ["nginx_access_log"]
	}
	beats{
                port => 5088
                codec => multiline {
                   pattern => "^# User@Host:"
                   negate => true
                   what => previous
                }
		tags => ["mysql_slow_log"]
        }	
}

filter {

 if "nginx_access_log" in [tags] {
  	grok {
    	match => { "message" => "%{COMBINEDAPACHELOG}" }
  	}
 }
 if "mysql_slow_log" in [tags] {
	grok {
	match => [ "message", "(?m)^#\s+User@Host:\s+%{USER:user}\[[^\]]+\]\s+@\s+%{USER:clienthost}\s+\[(?:%{IP:clientip})?\]\s+Id:\s+%{NUMBER:id:int}\n#\s+Schema:\s+%{USER:schema}\s+Last_errno:\s+%{NUMBER:lasterrorno:int}\s+Killed:\s+%{NUMBER:killedno:int}\n#\s+Query_time:\s+%{NUMBER:query_time:float}\s+Lock_time:\s+%{NUMBER:lock_time:float}\s+Rows_sent:\s+%{NUMBER:rows_sent:int}\s+Rows_examined:\s+%{NUMBER:rows_examined:int}\s+Rows_affected:\s+%{NUMBER:rows_affected:int}\n#\s+Bytes_sent:\s+%{NUMBER:bytes_sent:int}\n\s*(?:use\s+%{USER:usedatabase};\s*\n)?SET\s+timestamp=%{NUMBER:timestamp};\n\s*(?<query>(?<action>\w+)\b.*)\s*(?:\n#\s+Time)?.*$"]
	}
        date {
            match => [ "timestamp", "UNIX" ]
            #remove_field => [ "timestamp" ]
        }

 }

}

output {


    if "nginx_access_log" in [tags] {
	elasticsearch {
		hosts=>["localhost:9200"]
		index=>"%{[@metadata][beat]}-%{+YYYY.MM.dd}"
		document_type => "%{[@metadata][type]}"
	}
    }

    if "mysql_slow_log" in [tags] {
	elasticsearch {
		hosts=>["localhost:9200"]
		index=>"mysql-slow-log-%{+YYYY.MM.dd}"
	}
    }
}
[root@logstash-server conf.d]#
{% endhighlight %}

这里定义了两个输入源，分别在不同端口进行监听

关键是 **tags => ["nginx_access_log"]** ，这是在对自己这个输入源的日志进行打标

然后在处理和输出配置中使用到了 **if "nginx_access_log" in [tags]** 进行判断，用来分别处理

相关的配置基础可以参考 **[Configuring Logstash][lc]**


> **Tip:**  **/opt/logstash/bin/logstash** 命令的 **`-f`** 参数指定的可以是一个文件也可以是一个目录，当是一个目录的时候，目录下面的所有配置文件都会被合并成一个conf，然后再被加载 , 在 **/etc/init.d/logstash** 的启动脚本中有这么一段定义 **LS_CONF_DIR=/etc/logstash/conf.d**  就是在读取 **/etc/logstash/conf.d** 目录下的所有文件，作为自己的默认配置


---

##mysql slowlog的filebeat配置

{% highlight bash %}
[hunter@mysql-slow-log src]$ cat /etc/filebeat/filebeat.yml  | grep -v "#" | grep -v "^$"
filebeat:
  prospectors:
    -
      paths:
        - /var/lib/mysql/mysql-slow-log-slow.log
      input_type: log
      fields:
        testdb: flower
      fields_under_root: true 
  registry_file: /var/lib/filebeat/registry
output:
  logstash:
    hosts: ["logstash-server:5088"]
shipper:
logging:
  files:
[hunter@mysql-slow-log src]$
{% endhighlight %}

这里指定了日志路径，类型，添加了一个域并赋值，然后输出到指定的logstash中

相关的配置基础可以参考 **[Filebeat Configuration Options][filebeat_conf]**

> **Tip:** 默认情况下，filebeat是全量读取日志内容，除非打开 **tail_files: true**，打开后只会读到新追加的内容

---

##nginx accesslog的filebeat配置

{% highlight bash %}
[root@nginx-accesslog filebeat]# cat /etc/filebeat/filebeat.yml  | grep -v "#" | grep -v "^$"
filebeat:
  prospectors:
    -
      paths:
        - /usr/local/nginx/logs/nginxtest-access.log
      input_type: log
      fields:
        testenv: sport
      fields_under_root: true
  registry_file: /var/lib/filebeat/registry
output:
  logstash:
    hosts: ["logstash-server:5077"]
shipper:
logging:
  files:
[root@nginx-accesslog filebeat]# 
{% endhighlight %}

这里指定了日志路径，类型，添加了一个域并赋值，然后输出到指定的logstash中

相关的配置基础可以参考 **[Filebeat Configuration Options][filebeat_conf]**

> **Tip:** 默认情况下，filebeat是全量读取日志内容，除非打开 **tail_files: true**，打开后只会读到新追加的内容


> **Note:** CentOS 5 中 直接使用 **/etc/init.d/filebeat start** 会失败，报错如下

{% highlight bash %}
[root@nginx-accesslog filebeat]# /etc/init.d/filebeat start
Starting filebeat: FATAL: kernel too old
/bin/bash: line 1:  7968 Segmentation fault      filebeat-god -r / -n -p /var/run/filebeat.pid -- /usr/bin/filebeat -c /etc/filebeat/filebeat.yml
                                                           [FAILED]
[root@nginx-accesslog filebeat]#
{% endhighlight %}

原因是 **filebeat-god** 认为内核版本太老了，这是一个 **go** 语言写出来的工具，CentOS 5 的年代 go 语言还不能很好的对它进行支持

解决办法是直接使用 **`/usr/bin/filebeat -e -c /etc/filebeat/filebeat.yml`** 来运行程序， 如果有错误 **`-e`** 可以在终端看到错误输出，而不是syslog中

> **Tip:** 如果要以服务的形式在后台一直运行，可以这样： **`nohup /usr/bin/filebeat -c /etc/filebeat/filebeat.yml &`**  

---

##启动顺序(建议)


* 检查 logstash 配置(确保端口没有被占用，使用 **`-t`** 参数对语法进行检查)
* 启动(重启) logstash
* 检查新打开的端口
* 分别检查filbeat配置
* 启动(重启) filebeat
* 在elasticsearch中检查确认index (关注新生成的index)
* 在kibana中进行检查确认 (关注新生成的记录中 **_grokparsefailure** 标记的比例)
* (如有异常，要重新调试确认，一般要在测试环境中充分调试好)

---


#命令汇总

* **`cat filebeat-logstash-es.conf`**
* **`cat /etc/filebeat/filebeat.yml  | grep -v "#" | grep -v "^$"`**

---

[lc]:https://www.elastic.co/guide/en/logstash/current/configuration.html
[edc]:https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html
[lce]:https://www.elastic.co/guide/en/logstash/current/config-examples.html#using-conditionals
[filebeat_conf]:https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-configuration-details.html
