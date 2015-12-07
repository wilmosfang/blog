---
layout: post
title: Solr基础操作
categories: linux
excerpt: follow me
comments: true
---

---

#前言

**[Solr][solr]** 是一个使用得非常广泛的高可用，容错性分布式全文检索数据库

更为详细的文档可以参考 **[Resources][solrdoc]**

> **Tip:** 当前版本 **solr-5.3.0**

---

#概要

* TOC
{:toc}


---

##环境需求


**Solr** 运行在 **Java 7** 之上


{% highlight bash %}
[root@h102 solr]# java  -version
java version "1.7.0_65"
OpenJDK Runtime Environment (rhel-2.5.1.2.el6_5-x86_64 u65-b17)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
[root@h102 solr]# 
{% endhighlight %}

>Apache Solr runs of Java 7 or greater, Java 8 is verified to be compatible and may bring some performance improvements. When using Oracle Java 7 or OpenJDK 7, be sure to not use the GA build 147 or update versions u40, u45 and u51! We recommend using u55 or later.
>
>It is also recommended to always use the latest update version of your Java VM, because bugs may affect Solr. An overview of known JVM bugs can be found on http://wiki.apache.org/lucene-java/JavaBugs
>
>With all Java versions it is strongly recommended to not use experimental -XX JVM options.
>
>CPU, disk and memory requirements are based on the many choices made in implementing Solr (document size, number of documents, and number of hits retrieved to name a few). The benchmarks page has some information related to performance on particular platforms.


---

##安装

###下载与解压

**[Solr下载地址][download]**

> **Tip:** 里面列举了很多地址，可以选取一个离自己最近的最快的站点下载


{% highlight bash %}
[root@h102 solr]# tar -zxvf solr-5.3.0.tgz 
solr-5.3.0/LUCENE_CHANGES.txt
solr-5.3.0/contrib/analysis-extras/lib/
solr-5.3.0/contrib/clustering/lib/
solr-5.3.0/contrib/dataimporthandler-extras/lib/
...
...
solr-5.3.0/docs/solr-velocity/resources/titlebar.gif
solr-5.3.0/docs/solr-velocity/resources/titlebar_end.gif
solr-5.3.0/docs/solr-velocity/stylesheet.css
[root@h102 solr]# ls
solr-5.3.0  solr-5.3.0.tgz
[root@h102 solr]# cd solr-5.3.0
[root@h102 solr-5.3.0]# ls
bin  CHANGES.txt  contrib  dist  docs  example  licenses  LICENSE.txt  LUCENE_CHANGES.txt  NOTICE.txt  README.txt  server
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---


##启动solr

{% highlight bash %}
[root@h102 solr-5.3.0]# bin/solr start -e cloud -noprompt

Welcome to the SolrCloud example!

Starting up 2 Solr nodes for your example SolrCloud cluster.

Creating Solr home directory /data/solr/solr-5.3.0/example/cloud/node1/solr
Cloning /data/solr/solr-5.3.0/example/cloud/node1 into
   /data/solr/solr-5.3.0/example/cloud/node2

Starting up Solr on port 8983 using command:
bin/solr start -cloud -p 8983 -s "example/cloud/node1/solr"

Waiting up to 30 seconds to see Solr running on port 8983 [\]  
Started Solr server on port 8983 (pid=3579). Happy searching!
                                                                                                                                                           
Starting up Solr on port 7574 using command:
bin/solr start -cloud -p 7574 -s "example/cloud/node2/solr" -z localhost:9983

Waiting up to 30 seconds to see Solr running on port 7574 [/]  
Started Solr server on port 7574 (pid=3799). Happy searching!
                                                                                                                                                           
Connecting to ZooKeeper at localhost:9983 ...
Uploading /data/solr/solr-5.3.0/server/solr/configsets/data_driven_schema_configs/conf for config gettingstarted to ZooKeeper at localhost:9983

Creating new collection 'gettingstarted' using command:
http://localhost:8983/solr/admin/collections?action=CREATE&name=gettingstarted&numShards=2&replicationFactor=2&maxShardsPerNode=2&collection.configName=gettingstarted

{
  "responseHeader":{
    "status":0,
    "QTime":20494},
  "success":{"":{
      "responseHeader":{
        "status":0,
        "QTime":19880},
      "core":"gettingstarted_shard1_replica2"}}}

Enabling auto soft-commits with maxTime 3 secs using the Config API

POSTing request to Config API: http://localhost:8983/solr/gettingstarted/config
{"set-property":{"updateHandler.autoSoftCommit.maxTime":"3000"}}
Successfully set-property updateHandler.autoSoftCommit.maxTime to 3000


SolrCloud example running, please visit: http://localhost:8983/solr 

[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###配置iptables

修改 **/etc/sysconfig/iptables** 在 **filter** 中加入以下内容，然后reload

{% highlight bash %}
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8983 -j ACCEPT 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 7574 -j ACCEPT 
{% endhighlight %}

---

###管理界面

在本地使用 **http://localhost:8983/solr/** ，或远程使用 **http://ip:8983/solr/** 访问管理界面

![Image_201509091621414.png](/images/solr/Image_201509091621414.png)

> **Tip:** 也可以使用 **7574** 进行访问

![Image_201509091628205.png](/images/solr/Image_201509091628205.png)

---

###当前拓扑

这是当前的拓扑

![Image_201509091630127.png](/images/solr/Image_201509091630127.png)

---

##添加数据


使用 **bin/post** 可以方便的添加数据

{% highlight bash %}
[root@h102 solr-5.3.0]# bin/post -h 

Usage: post -c <collection> [OPTIONS] <files|directories|urls|-d ["...",...]>
    or post -help

   collection name defaults to DEFAULT_SOLR_COLLECTION if not specified

OPTIONS
=======
  Solr options:
    -url <base Solr update URL> (overrides collection, host, and port)
    -host <host> (default: localhost)
    -p or -port <port> (default: 8983)
    -commit yes|no (default: yes)

  Web crawl options:
    -recursive <depth> (default: 1)
    -delay <seconds> (default: 10)

  Directory crawl options:
    -delay <seconds> (default: 0)

  stdin/args options:
    -type <content/type> (default: application/xml)

  Other options:
    -filetypes <type>[,<type>,...] (default: xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log)
    -params "<key>=<value>[&<key>=<value>...]" (values must be URL-encoded; these pass through to Solr update request)
    -out yes|no (default: no; yes outputs Solr response to console)


Examples:

* JSON file: bin/post -c wizbang events.json
* XML files: bin/post -c records article*.xml
* CSV file: bin/post -c signals LATEST-signals.csv
* Directory of files: bin/post -c myfiles ~/Documents
* Web crawl: bin/post -c gettingstarted http://lucene.apache.org/solr -recursive 1 -delay 1
* Standard input (stdin): echo '{commit: {}}' | bin/post -c my_collection -type application/json -out yes -d
* Data as string: bin/post -c signals -type text/csv -out yes -d $'id,value\n1,0.47'

[root@h102 solr-5.3.0]# 
{% endhighlight %}


添加一个目录里所有内容到solr

{% highlight bash %}
[root@h102 solr-5.3.0]# du -sh docs/
70M	docs/
[root@h102 solr-5.3.0]# 
[root@h102 solr-5.3.0]# bin/post -c gettingstarted docs/
java -classpath /data/solr/solr-5.3.0/dist/solr-core-5.3.0.jar -Dauto=yes -Dc=gettingstarted -Ddata=files -Drecursive=yes org.apache.solr.util.SimplePostTool docs/
SimplePostTool version 5.0.0
Posting files to [base] url http://localhost:8983/solr/gettingstarted/update...
Entering auto mode. File endings considered are xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log
Entering recursive mode, max depth=999, delay=0s
Indexing directory docs (3 files, depth=0)
...
...
POSTing file DIHProperties.html (text/html) to [base]/extract
POSTing file RequestInfo.html (text/html) to [base]/extract
POSTing file ClobTransformer.html (text/html) to [base]/extract
Indexing directory docs/solr-dataimporthandler/resources (0 files, depth=2)
3773 files indexed.
COMMITting Solr index changes to http://localhost:8983/solr/gettingstarted/update...
Time spent: 0:02:41.136
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###索引XML

{% highlight bash %}
[root@h102 solr-5.3.0]# bin/post -c gettingstarted example/exampledocs/*.xml
java -classpath /data/solr/solr-5.3.0/dist/solr-core-5.3.0.jar -Dauto=yes -Dc=gettingstarted -Ddata=files org.apache.solr.util.SimplePostTool example/exampledocs/gb18030-example.xml example/exampledocs/hd.xml example/exampledocs/ipod_other.xml example/exampledocs/ipod_video.xml example/exampledocs/manufacturers.xml example/exampledocs/mem.xml example/exampledocs/money.xml example/exampledocs/monitor2.xml example/exampledocs/monitor.xml example/exampledocs/mp500.xml example/exampledocs/sd500.xml example/exampledocs/solr.xml example/exampledocs/utf8-example.xml example/exampledocs/vidcard.xml
SimplePostTool version 5.0.0
Posting files to [base] url http://localhost:8983/solr/gettingstarted/update...
Entering auto mode. File endings considered are xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log
POSTing file gb18030-example.xml (application/xml) to [base]
POSTing file hd.xml (application/xml) to [base]
POSTing file ipod_other.xml (application/xml) to [base]
POSTing file ipod_video.xml (application/xml) to [base]
POSTing file manufacturers.xml (application/xml) to [base]
POSTing file mem.xml (application/xml) to [base]
POSTing file money.xml (application/xml) to [base]
POSTing file monitor2.xml (application/xml) to [base]
POSTing file monitor.xml (application/xml) to [base]
POSTing file mp500.xml (application/xml) to [base]
POSTing file sd500.xml (application/xml) to [base]
POSTing file solr.xml (application/xml) to [base]
POSTing file utf8-example.xml (application/xml) to [base]
POSTing file vidcard.xml (application/xml) to [base]
14 files indexed.
COMMITting Solr index changes to http://localhost:8983/solr/gettingstarted/update...
Time spent: 0:00:23.754
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###索引JSON


{% highlight bash %}
[root@h102 solr-5.3.0]# bin/post -c gettingstarted example/exampledocs/books.json
java -classpath /data/solr/solr-5.3.0/dist/solr-core-5.3.0.jar -Dauto=yes -Dc=gettingstarted -Ddata=files org.apache.solr.util.SimplePostTool example/exampledocs/books.json
SimplePostTool version 5.0.0
Posting files to [base] url http://localhost:8983/solr/gettingstarted/update...
Entering auto mode. File endings considered are xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log
POSTing file books.json (application/json) to [base]
1 files indexed.
COMMITting Solr index changes to http://localhost:8983/solr/gettingstarted/update...
Time spent: 0:00:05.241
[root@h102 solr-5.3.0]# 
{% endhighlight %}


---

###索引CSV

{% highlight bash %}
[root@h102 solr-5.3.0]# bin/post -c gettingstarted example/exampledocs/books.csv
java -classpath /data/solr/solr-5.3.0/dist/solr-core-5.3.0.jar -Dauto=yes -Dc=gettingstarted -Ddata=files org.apache.solr.util.SimplePostTool example/exampledocs/books.csv
SimplePostTool version 5.0.0
Posting files to [base] url http://localhost:8983/solr/gettingstarted/update...
Entering auto mode. File endings considered are xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log
POSTing file books.csv (text/csv) to [base]
1 files indexed.
COMMITting Solr index changes to http://localhost:8983/solr/gettingstarted/update...
Time spent: 0:00:00.512
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

##使用管理界面检索数据



###方法一 


指定集合然后使用 **Query** 或通过 **http://192.168.100.102:7574/solr/#/gettingstarted_shard2_replica1/query**

![Image_201509091659428.png](/images/solr/Image_201509091659428.png)

在 **q** 中输入关键字，然后执行搜索


---


###方法二

通过 **http://ip:8983/solr/gettingstarted/browse** 输入关键字

![Image_2015090921165510.png](/images/solr/Image_2015090921165510.png)

输入关键字 **solr** 的结果


![Image_201509092116299.png](/images/solr/Image_201509092116299.png)

---

##删除数据

{% highlight bash %}
[root@h102 solr-5.3.0]# bin/post -c gettingstarted -d "<delete><id>/data/solr/solr-5.3.0/docs/quickstart.html</id></delete>"
java -classpath /data/solr/solr-5.3.0/dist/solr-core-5.3.0.jar -Dauto=yes -Dc=gettingstarted -Ddata=args org.apache.solr.util.SimplePostTool <delete><id>/data/solr/solr-5.3.0/docs/quickstart.html</id></delete>
SimplePostTool version 5.0.0
POSTing args to http://localhost:8983/solr/gettingstarted/update...
COMMITting Solr index changes to http://localhost:8983/solr/gettingstarted/update...
Time spent: 0:00:00.292
[root@h102 solr-5.3.0]# 
{% endhighlight %}

再使用 **http://192.168.100.102:7574/solr/gettingstarted_shard1_replica1/browse?q=example** 就搜不到了

---

##使用CLI检索数据

使用curl可以快速返回结果

**http://192.168.100.102:7574/solr/gettingstarted_shard1_replica1/select?q=\*%3A\*&wt=json&indent=true**


###任意匹配

{% highlight bash %}
[root@h102 solr-5.3.0]# curl  "http://192.168.100.102:7574/solr/gettingstarted_shard1_replica1/select?q=*%3A*&wt=json&indent=true"
{
  "responseHeader":{
    "status":0,
    "QTime":16,
    "params":{
      "indent":"true",
      "q":"*:*",
      "wt":"json"}},
  "response":{"numFound":3772,"start":0,"maxScore":1.0,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-clustering/org/apache/solr/handler/clustering/carrot2/class-use/LuceneCarrot2StemmerFactory.html",
        "title":["Uses of Class org.apache.solr.handler.clustering.carrot2.LuceneCarrot2StemmerFactory (Solr 5.3.0 API)"],
        "stream_content_type":["text/html"],
        "stream_size":[5204],
        "content_encoding":["UTF-8"],
        "date":["2015-08-17T00:00:00Z"],
        "x_parsed_by":["org.apache.tika.parser.DefaultParser",
          "org.apache.tika.parser.html.HtmlParser"],
        "content_type":["text/html; charset=utf-8"],
        "resourcename":["/data/solr/solr-5.3.0/docs/solr-clustering/org/apache/solr/handler/clustering/carrot2/class-use/LuceneCarrot2StemmerFactory.html"],
        "dc_title":["Uses of Class org.apache.solr.handler.clustering.carrot2.LuceneCarrot2StemmerFactory (Solr 5.3.0 API)"],
        "_version_":1511824584027930624},
       ...
       ...
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-solrj/overview-tree.html",
        "title":["Class Hierarchy (Solr 5.3.0 API)"],
        "stream_content_type":["text/html"],
        "stream_size":[104552],
        "content_encoding":["UTF-8"],
        "date":["2015-08-17T00:00:00Z"],
        "x_parsed_by":["org.apache.tika.parser.DefaultParser",
          "org.apache.tika.parser.html.HtmlParser"],
        "content_type":["text/html; charset=utf-8"],
        "resourcename":["/data/solr/solr-5.3.0/docs/solr-solrj/overview-tree.html"],
        "dc_title":["Class Hierarchy (Solr 5.3.0 API)"],
        "_version_":1511824585009397760}]
  }}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###单关键字匹配

{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&q=foundation"
{
  "responseHeader":{
    "status":0,
    "QTime":45,
    "params":{
      "indent":"true",
      "q":"foundation",
      "wt":"json"}},
  "response":{"numFound":3608,"start":0,"maxScore":0.05791749,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-clustering/org/apache/solr/handler/clustering/carrot2/package-use.html",
        "title":["Uses of Package org.apache.solr.handler.clustering.carrot2 (Solr 5.3.0 API)"],
        "stream_content_type":["text/html"],
        "stream_size":[4597],
        "content_encoding":["UTF-8"],
        "date":["2015-08-17T00:00:00Z"],
        "x_parsed_by":["org.apache.tika.parser.DefaultParser",
          "org.apache.tika.parser.html.HtmlParser"],
        "content_type":["text/html; charset=utf-8"],
        "resourcename":["/data/solr/solr-5.3.0/docs/solr-clustering/org/apache/solr/handler/clustering/carrot2/package-use.html"],
        "dc_title":["Uses of Package org.apache.solr.handler.clustering.carrot2 (Solr 5.3.0 API)"],
        "_version_":1511824583560265728},
     
     ...
     ...
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-test-framework/org/apache/solr/analysis/package-use.html",
        "title":["Uses of Package org.apache.solr.analysis (Solr 5.3.0 API)"],
        "stream_content_type":["text/html"],
        "stream_size":[4399],
        "content_encoding":["UTF-8"],
        "date":["2015-08-17T00:00:00Z"],
        "x_parsed_by":["org.apache.tika.parser.DefaultParser",
          "org.apache.tika.parser.html.HtmlParser"],
        "content_type":["text/html; charset=utf-8"],
        "resourcename":["/data/solr/solr-5.3.0/docs/solr-test-framework/org/apache/solr/analysis/package-use.html"],
        "dc_title":["Uses of Package org.apache.solr.analysis (Solr 5.3.0 API)"],
        "_version_":1511824702977343488}]
  }}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###限定输出

通过 **"numFound":3608** 我们知道一共有 **3608** 个结果，我们可以通过参数限定输出,默认情况下影响输出的参数是以下默认值

Parameter| Value
-------- | ---
start    | 0
rows     | 10
fl       | \*:\*

我们修改一下以它们的值

**q=foundation** 搜索关键字 **foundation**

**fl=id** 只返回 **id** 

**start=30** 开始为第 **30** 条

**rows=5** 只返回 **5** 条


{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&q=foundation&fl=id&start=30&rows=5"
{
  "responseHeader":{
    "status":0,
    "QTime":19,
    "params":{
      "fl":"id",
      "indent":"true",
      "start":"30",
      "q":"foundation",
      "wt":"json",
      "rows":"5"}},
  "response":{"numFound":3608,"start":30,"maxScore":0.05791749,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-uima/deprecated-list.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-analysis-extras/constant-values.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-analysis-extras/deprecated-list.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-clustering/org/apache/solr/handler/clustering/carrot2/class-use/LuceneCarrot2StemmerFactory.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-clustering/org/apache/solr/handler/clustering/carrot2/class-use/LuceneCarrot2TokenizerFactory.html"}]
  }}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###属性匹配

**q=field:value** 可以进行更精细的属性限定

比如只搜索 **\_version\_** 为 **1511824568810995712** 的文档 

{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&q=_version_:1511824568810995712"
{
  "responseHeader":{
    "status":0,
    "QTime":18,
    "params":{
      "indent":"true",
      "q":"_version_:1511824568810995712",
      "wt":"json"}},
  "response":{"numFound":1,"start":0,"maxScore":7.851185,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-morphlines-cell/allclasses-noframe.html",
        "title":["All Classes (Solr 5.3.0 API)"],
        "stream_content_type":["text/html"],
        "stream_size":[1059],
        "content_encoding":["UTF-8"],
        "date":["2015-08-17T00:00:00Z"],
        "x_parsed_by":["org.apache.tika.parser.DefaultParser",
          "org.apache.tika.parser.html.HtmlParser"],
        "content_type":["text/html; charset=utf-8"],
        "resourcename":["/data/solr/solr-5.3.0/docs/solr-morphlines-cell/allclasses-noframe.html"],
        "dc_title":["All Classes (Solr 5.3.0 API)"],
        "_version_":1511824568810995712}]
  }}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

###多关键字匹配（或）

如果要进行多关键字搜索，就使用 **+**  ，例如： **q=ui+test**  或 **q='ui+test'**


{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&fl=id&rows=3&q=test"
{
  "responseHeader":{
    "status":0,
    "QTime":13,
    "params":{
      "fl":"id",
      "indent":"true",
      "q":"test",
      "wt":"json",
      "rows":"3"}},
  "response":{"numFound":204,"start":0,"maxScore":0.48749265,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-test-framework/overview-summary.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-test-framework/org/apache/solr/analysis/package-frame.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-test-framework/org/apache/solr/core/package-frame.html"}]
  }}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&fl=id&rows=3&q=ui"
{
  "responseHeader":{
    "status":0,
    "QTime":8,
    "params":{
      "fl":"id",
      "indent":"true",
      "q":"ui",
      "wt":"json",
      "rows":"3"}},
  "response":{"numFound":31,"start":0,"maxScore":0.3156422,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/servlet/package-summary.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/handler/admin/package-summary.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/core/package-use.html"}]
  }}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&fl=id&rows=3&q=ui+test"
{
  "responseHeader":{
    "status":0,
    "QTime":13,
    "params":{
      "fl":"id",
      "indent":"true",
      "q":"ui test",
      "wt":"json",
      "rows":"3"}},
  "response":{"numFound":234,"start":0,"maxScore":0.17063226,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/changes/Changes.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-test-framework/overview-summary.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-test-framework/org/apache/solr/analysis/package-frame.html"}]
  }}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

> **Tip:** 同时匹配多个关键字的结果会排到更前列


---

###多关键字匹配（且）

**+** **(%2B)** 前缀代表必须包含

{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&fl=id&rows=3&q=%2Bone+%2Bthree"
{
  "responseHeader":{
    "status":0,
    "QTime":11,
    "params":{
      "fl":"id",
      "indent":"true",
      "q":"+one +three",
      "wt":"json",
      "rows":"3"}},
  "response":{"numFound":20,"start":0,"maxScore":0.41965395,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/schema/SimplePreAnalyzedParser.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/util/doc-files/min-should-match.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/handler/DumpRequestHandler.html"}]
  }}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

>By default, when you search for multiple terms and/or phrases in a single query, Solr will only require that one of them is present in order for a document to match. Documents containing more terms will be sorted higher in the results list.
>
>You can require that a term or phrase is present by prefixing it with a "+"; conversely, to disallow the presence of a term or phrase, prefix it with a "-".
>
>To find documents that contain both terms "one" and "three", enter +one +three in the q param in the core-specific Admin UI Query tab. Because the "+" character has a reserved purpose in URLs (encoding the space character), you must URL encode it for curl as "%2B":
>
>To search for documents that contain the term "two" but don't contain the term "one", enter +two -one in the q param in the Admin UI. Again, URL encode "+" as "%2B":


**-** 前缀代表必须不包含指定关键字

{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?wt=json&indent=true&fl=id&rows=3&q=%2Btwo+-three"
{
  "responseHeader":{
    "status":0,
    "QTime":15,
    "params":{
      "fl":"id",
      "indent":"true",
      "q":"+two -three",
      "wt":"json",
      "rows":"3"}},
  "response":{"numFound":118,"start":0,"maxScore":0.33085516,"docs":[
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-analytics/org/apache/solr/analytics/expression/package-use.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-analytics/org/apache/solr/analytics/expression/package-summary.html"},
      {
        "id":"/data/solr/solr-5.3.0/docs/solr-core/org/apache/solr/search/function/distance/package-summary.html"}]
  }}
[root@h102 solr-5.3.0]#
{% endhighlight %}

> **Note:** 所谓的不包含并不是绝对的，比如不包含 **one** 但可以包含 **none** ，结果取决于分词算法


---

###Faceting

####Field facets

信息分组统计

**rows=0** 不打印出检索结果
 
**facet=true** 打开 **facet**

**facet.field=stream_size**  设定 **stream_size** 为聚合对象



{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://192.168.100.102:7574/solr/gettingstarted_shard1_replica1/select?q=*%3A*&wt=json&indent=true&rows=0&facet=true&facet.field=stream_size"
{
  "responseHeader":{
    "status":0,
    "QTime":131,
    "params":{
      "facet":"true",
      "indent":"true",
      "q":"*:*",
      "facet.field":"stream_size",
      "wt":"json",
      "rows":"0"}},
  "response":{"numFound":3772,"start":0,"maxScore":1.0,"docs":[]
  },
  "facet_counts":{
    "facet_queries":{},
    "facet_fields":{
      "stream_size":[
        "4812",22,
        "4823",20,
        "4779",19,
        "4867",18,
        "4790",17,
        "4768",16,
        "4834",16,
        "4746",15,
        "4801",15,
        "4878",15,
        "4920",14,
        "4757",13,
        "4986",13,
        "4724",12,
        "4856",12,
        "4931",12,
        "4964",12,
        "5008",12,
        "5030",12,
        "5063",12,
        "4735",11,
        "4942",11,
        "4975",11,
        "4997",11,
        "4702",10,
        "4845",10,
        "4854",10,
        "4865",10,
        "5094",10,
        "4889",9,
        "5019",9,
        "2737",8,
        "4777",8,
        "4843",8,
        "4900",8,
        "4911",8,
        "4944",8,
        "4953",8,
        "4966",8,
        "2900",7,
        "4821",7,
        "4887",7,
        "4909",7,
        "5041",7,
        "5118",7,
        "5140",7,
        "8989",7,
        "4799",6,
        "4810",6,
        "4832",6,
        "4922",6,
        "4933",6,
        "5017",6,
        "5127",6,
        "5149",6,
        "4691",5,
        "4898",5,
        "5032",5,
        "5052",5,
        "5072",5,
        "5083",5,
        "5162",5,
        "4658",4,
        "4680",4,
        "4713",4,
        "4876",4,
        "4955",4,
        "4977",4,
        "5039",4,
        "5096",4,
        "5105",4,
        "5138",4,
        "5151",4,
        "5173",4,
        "5246",4,
        "5294",4,
        "7305",4,
        "3941",3,
        "4744",3,
        "4788",3,
        "4962",3,
        "5010",3,
        "5050",3,
        "5129",3,
        "5160",3,
        "5184",3,
        "5204",3,
        "5213",3,
        "5224",3,
        "5279",3,
        "6294",3,
        "7523",3,
        "7549",3,
        "7618",3,
        "7983",3,
        "8109",3,
        "9276",3,
        "13358",3,
        "1334",2,
        "1377",2]},
    "facet_dates":{},
    "facet_ranges":{},
    "facet_intervals":{},
    "facet_heatmaps":{}}}
[root@h102 solr-5.3.0]# 
{% endhighlight %}

---

####Range facets

可以使用区间来进一步分组

**facet=true**  打开 facet

**facet.range=stream_size** 以stream_size的分布来分组

**f.stream_size.facet.range.start=0** 从0开始

**f.stream_size.facet.range.end=9000** 9000为上限

**f.stream_size.facet.range.gap=1000** 步进为1000

**facet.range.other=after** 其它的排最后

{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://192.168.100.102:7574/solr/gettingstarted_shard1_replica1/select?q=*%3A*&wt=json&indent=true&rows=0&facet=true&facet.range=stream_size&f.stream_size.facet.range.start=0&&f.stream_size.facet.range.end=9000&f.stream_size.facet.range.gap=1000&facet.range.other=after"
{
  "responseHeader":{
    "status":0,
    "QTime":41,
    "params":{
      "facet.range.other":"after",
      "f.stream_size.facet.range.start":"0",
      "facet":"true",
      "f.stream_size.facet.range.gap":"1000",
      "f.stream_size.facet.range.end":"9000",
      "indent":"true",
      "q":"*:*",
      "facet.range":"stream_size",
      "wt":"json",
      "rows":"0"}},
  "response":{"numFound":3772,"start":0,"maxScore":1.0,"docs":[]
  },
  "facet_counts":{
    "facet_queries":{},
    "facet_fields":{},
    "facet_dates":{},
    "facet_ranges":{
      "stream_size":{
        "counts":[
          "0",16,
          "1000",61,
          "2000",42,
          "3000",12,
          "4000",591,
          "5000",324,
          "6000",107,
          "7000",265,
          "8000",223],
        "gap":1000,
        "after":2131,
        "start":0,
        "end":9000}},
    "facet_intervals":{},
    "facet_heatmaps":{}}}
[root@h102 solr-5.3.0]#
{% endhighlight %}

---

####Pivot facets

其实也就是双重分组

**facet.pivot=stream_size,title**  先根据 **stream_size** 分组 ，再根据 **title** 分组

{% highlight bash %}
[root@h102 solr-5.3.0]# curl "http://localhost:8983/solr/gettingstarted/select?q=*:*&rows=0&wt=json&indent=on&facet=on&facet.pivot=stream_size,title"
{
  "responseHeader":{
    "status":0,
    "QTime":5265,
    "params":{
      "facet":"on",
      "indent":"on",
      "q":"*:*",
      "wt":"json",
      "facet.pivot":"stream_size,title",
      "rows":"0"}},
  "response":{"numFound":3772,"start":0,"maxScore":1.0,"docs":[]
  },
  "facet_counts":{
    "facet_queries":{},
    "facet_fields":{},
    "facet_dates":{},
    "facet_ranges":{},
    "facet_intervals":{},
    "facet_heatmaps":{},
    "facet_pivot":{
      "stream_size,title":[{
          "field":"stream_size",
          "value":4812,
          "count":22,
          "pivot":[{
              "field":"title",
              "value":"Uses of Class org.apache.solr.cloud.OverseerSolrResponse (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.cloud.SocketProxy.Acceptor (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.core.CorePropertiesLocator (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.core.NIOFSDirectoryFactory (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.handler.DumpRequestHandler (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.handler.PingRequestHandler (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.handler.RealTimeGetHandler (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.response.CSVResponseWriter (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.response.PHPResponseWriter (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.response.RawResponseWriter (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.response.XMLResponseWriter (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.DisMaxQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.ExportQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.NestedQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.PrefixQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.ReRankQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.SimpleQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.SolrFieldCacheMBean (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.search.SwitchQParserPlugin (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.servlet.LoadAdminUiServlet (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.util.CryptoKeys.RSAKeyPair (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"Uses of Class org.apache.solr.util.PropertiesInputStream (Solr 5.3.0 API)",
              "count":1}]},
              ...
              ...
                 {
          "field":"stream_size",
          "value":1334,
          "count":2,
          "pivot":[{
              "field":"title",
              "value":"org.apache.solr.client.solrj.beans (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"org.apache.solr.search.join (Solr 5.3.0 API)",
              "count":1}]},
        {
          "field":"stream_size",
          "value":1377,
          "count":2,
          "pivot":[{
              "field":"title",
              "value":"org.apache.solr.handler.loader (Solr 5.3.0 API)",
              "count":1},
            {
              "field":"title",
              "value":"org.apache.solr.store.hdfs (Solr 5.3.0 API)",
              "count":1}]}]}}}
[root@h102 solr-5.3.0]# 
{% endhighlight %}


---

##关闭Solr

正常运行的状态下，会有两个 **java** 分别监听在 **8983** 和 **7574**

{% highlight bash %}
[root@h102 solr-5.3.0]#  ps faux | grep solr 
root     63799  0.0  0.0 103252   828 pts/1    S+   13:45   0:00  |       \_ grep solr
root      3579  0.8 13.0 2002356 513088 pts/0  Sl   Sep09  11:10 java -server -Xss256k -Xms512m -Xmx512m -XX:NewRatio=3 -XX:SurvivorRatio=4 -XX:TargetSurvivorRatio=90 -XX:MaxTenuringThreshold=8 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:ConcGCThreads=4 -XX:ParallelGCThreads=4 -XX:+CMSScavengeBeforeRemark -XX:PretenureSizeThreshold=64m -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=50 -XX:CMSMaxAbortablePrecleanTime=6000 -XX:+CMSParallelRemarkEnabled -XX:+ParallelRefProcEnabled -XX:CMSFullGCsBeforeCompaction=1 -XX:CMSTriggerPermRatio=80 -verbose:gc -XX:+PrintHeapAtGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:/data/solr/solr-5.3.0/example/cloud/node1/solr/../logs/solr_gc.log -DzkClientTimeout=15000 -DzkRun -Djetty.port=8983 -DSTOP.PORT=7983 -DSTOP.KEY=solrrocks -Duser.timezone=UTC -Djetty.home=/data/solr/solr-5.3.0/server -Dsolr.solr.home=/data/solr/solr-5.3.0/example/cloud/node1/solr -Dsolr.install.dir=/data/solr/solr-5.3.0 -Dlog4j.configuration=file:/data/solr/solr-5.3.0/example/resources/log4j.properties -jar start.jar -XX:OnOutOfMemoryError=/data/solr/solr-5.3.0/bin/oom_solr.sh 8983 /data/solr/solr-5.3.0/example/cloud/node1/solr/../logs --module=http
root      3799  0.4  9.4 2006728 369716 pts/0  Sl   Sep09   5:54 java -server -Xss256k -Xms512m -Xmx512m -XX:NewRatio=3 -XX:SurvivorRatio=4 -XX:TargetSurvivorRatio=90 -XX:MaxTenuringThreshold=8 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:ConcGCThreads=4 -XX:ParallelGCThreads=4 -XX:+CMSScavengeBeforeRemark -XX:PretenureSizeThreshold=64m -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=50 -XX:CMSMaxAbortablePrecleanTime=6000 -XX:+CMSParallelRemarkEnabled -XX:+ParallelRefProcEnabled -XX:CMSFullGCsBeforeCompaction=1 -XX:CMSTriggerPermRatio=80 -verbose:gc -XX:+PrintHeapAtGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:/data/solr/solr-5.3.0/example/cloud/node2/solr/../logs/solr_gc.log -DzkClientTimeout=15000 -DzkHost=localhost:9983 -Djetty.port=7574 -DSTOP.PORT=6574 -DSTOP.KEY=solrrocks -Duser.timezone=UTC -Djetty.home=/data/solr/solr-5.3.0/server -Dsolr.solr.home=/data/solr/solr-5.3.0/example/cloud/node2/solr -Dsolr.install.dir=/data/solr/solr-5.3.0 -Dlog4j.configuration=file:/data/solr/solr-5.3.0/example/resources/log4j.properties -jar start.jar -XX:OnOutOfMemoryError=/data/solr/solr-5.3.0/bin/oom_solr.sh 7574 /data/solr/solr-5.3.0/example/cloud/node2/solr/../logs --module=http
[root@h102 solr-5.3.0]# netstat  -ant | grep -E '(8983|7574)'
tcp        0      0 :::7574                     :::*                        LISTEN      
tcp        0      0 :::8983                     :::*                        LISTEN      
tcp        0      0 ::ffff:192.168.100.10:42750 ::ffff:192.168.100.102:8983 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:33948 ::ffff:192.168.100.102:7574 TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.102:8983 ::ffff:192.168.100.10:42749 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.102:8983 ::ffff:192.168.100.10:42745 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.102:7574 ::ffff:192.168.100.10:33936 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:42745 ::ffff:192.168.100.102:8983 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:33949 ::ffff:192.168.100.102:7574 TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.10:42749 ::ffff:192.168.100.102:8983 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:33947 ::ffff:192.168.100.102:7574 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.102:7574 ::ffff:192.168.100.10:33947 ESTABLISHED 
tcp        0      0 ::ffff:127.0.0.1:42313      ::ffff:127.0.0.1:8983       TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.102:7574 ::ffff:192.168.100.10:33938 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:33942 ::ffff:192.168.100.102:7574 TIME_WAIT   
tcp        0      0 ::ffff:127.0.0.1:42307      ::ffff:127.0.0.1:8983       TIME_WAIT   
tcp        0      0 ::ffff:127.0.0.1:42316      ::ffff:127.0.0.1:8983       TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.10:42744 ::ffff:192.168.100.102:8983 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:33938 ::ffff:192.168.100.102:7574 ESTABLISHED 
tcp        0      0 ::ffff:127.0.0.1:42317      ::ffff:127.0.0.1:8983       TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.102:8983 ::ffff:192.168.100.10:42744 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.102:8983 ::ffff:192.168.100.10:42750 ESTABLISHED 
tcp        0      0 ::ffff:127.0.0.1:42327      ::ffff:127.0.0.1:8983       TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.10:33945 ::ffff:192.168.100.102:7574 TIME_WAIT   
tcp        0      0 ::ffff:127.0.0.1:42314      ::ffff:127.0.0.1:8983       TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.10:33936 ::ffff:192.168.100.102:7574 ESTABLISHED 
tcp        0      0 ::ffff:192.168.100.10:33946 ::ffff:192.168.100.102:7574 TIME_WAIT   
tcp        0      0 ::ffff:192.168.100.10:33943 ::ffff:192.168.100.102:7574 TIME_WAIT   
tcp        0      0 ::ffff:127.0.0.1:59150      ::ffff:127.0.0.1:7574       TIME_WAIT   
[root@h102 solr-5.3.0]# 
{% endhighlight %}

使用如下命令停止服务 

{% highlight bash %}
[root@h102 solr-5.3.0]# bin/solr stop -all
Sending stop command to Solr running on port 8983 ... waiting 5 seconds to allow Jetty process 3579 to stop gracefully.
Sending stop command to Solr running on port 7574 ... waiting 5 seconds to allow Jetty process 3799 to stop gracefully.
Solr process 3799 is still running; forcefully killing it now.
Killed process 3799
[root@h102 solr-5.3.0]# 
{% endhighlight %}

原来的两个 **java** 分别监听在 **8983** 和 **7574** ，现在已经没有了

> **Tip:** 监听的端口会马上消失，但是还有一些残留的连接处于 **TIME_WAIT** 状态，timeout 后会自动消失，但此刻管理界面已经无法访问了

{% highlight bash %}
[root@h102 solr-5.3.0]# ps fuax | grep solr 
root     64883  0.0  0.0 103252   828 pts/1    S+   13:56   0:00  |       \_ grep solr
[root@h102 solr-5.3.0]# netstat  -ant | grep -E '(8983|7574)'
[root@h102 solr-5.3.0]#
{% endhighlight %}


---

##删除数据


数据保存到如下路径

{% highlight bash %}
[root@h102 solr-5.3.0]# tree example/cloud/
example/cloud/
├── node1
│   ├── logs
│   │   ├── solr-8983-console.log
│   │   ├── solr_gc.log
│   │   └── solr.log
│   └── solr
│       ├── gettingstarted_shard1_replica1
│       │   ├── core.properties
│       │   └── data
│       │       ├── index
│       │       │   ├── segments_1
│       │       │   └── write.lock
│       │       └── tlog
│       ├── gettingstarted_shard2_replica1
│       │   ├── core.properties
│       │   └── data
│       │       ├── index
│       │       │   ├── segments_1
│       │       │   └── write.lock
│       │       └── tlog
│       ├── solr.xml
│       ├── zoo.cfg
│       └── zoo_data
│           └── version-2
│               └── log.1
└── node2
    ├── logs
    │   ├── solr-7574-console.log
    │   ├── solr_gc.log
    │   └── solr.log
    └── solr
        ├── gettingstarted_shard1_replica2
        │   ├── core.properties
        │   └── data
        │       ├── index
        │       │   ├── segments_1
        │       │   └── write.lock
        │       └── tlog
        ├── gettingstarted_shard2_replica2
        │   ├── core.properties
        │   └── data
        │       ├── index
        │       │   ├── segments_1
        │       │   └── write.lock
        │       └── tlog
        ├── solr.xml
        └── zoo.cfg

24 directories, 23 files
[root@h102 solr-5.3.0]# 
{% endhighlight %}

如果要删除数据，确保服务已经停止的前提下，通过如下方式

{% highlight bash %}
[root@h102 solr-5.3.0]# rm -Rf  example/cloud/
[root@h102 solr-5.3.0]# tree example/cloud/
example/cloud/ [error opening dir]

0 directories, 0 files
[root@h102 solr-5.3.0]# 
{% endhighlight %}


---

#总结


* Launched Solr into SolrCloud mode, two nodes, two collections including shards and replicas
* Indexed a directory of rich text files
* Indexed Solr XML files
* Indexed Solr JSON files
* Indexed CSV content
* Opened the admin console, used its query interface to get JSON formatted results
* Opened the /browse interface to explore Solr's features in a more friendly and familiar interface


---


#附

使用下面的脚本可以快速的准备出试验环境

{% highlight bash %}
date ;
bin/solr start -e cloud -noprompt ;
  open http://localhost:8983/solr ;
  bin/post -c gettingstarted docs/ ;
  open http://localhost:8983/solr/gettingstarted/browse ;
  bin/post -c gettingstarted example/exampledocs/*.xml ;
  bin/post -c gettingstarted example/exampledocs/books.json ;
  bin/post -c gettingstarted example/exampledocs/books.csv ;
  bin/post -c gettingstarted -d "<delete><id>SP2514N</id></delete>" ;
  bin/solr healthcheck -c gettingstarted ;
date ;
{% endhighlight %}


---


[solr]:http://lucene.apache.org/solr/
[solrdoc]:http://lucene.apache.org/solr/resources.html
[download]:http://www.apache.org/dyn/closer.lua/lucene/solr/5.3.0

