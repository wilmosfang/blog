---
layout: post
title:  Ruby 操作 MongoDB
author: wilmosfang
tags:   mongodb ruby
categories:   ruby
wc: 1081 4848 50732
excerpt:  mongo gem 的安装，兼容性，连接数据库，插入数据，查询数据，修改数据，删除数据，集合操作，数据库操作，索引操作等相关基础
comments: true
---


# 前言


使用 Ruby 处理各种任务时难免会和数据库打交道，而 MongoDB 又是一款应用极其广泛的数据库

**[RubyGems][rubygems]** 是 Ruby 的武器库，类似于 Perl 的 CPAN，各类封装好的处理逻辑应有尽有，我们可以充分利用这些成品包来减轻开发的工作量，其中 **mongo** 的 gem 就可以很好地满足我们的需求

>A Ruby driver for MongoDB

>The MongoDB Ruby driver is the officially supported Ruby driver for MongoDB. It's written in pure Ruby and is optimized for simplicity. It can be used on its own, but it also serves as the basis of several object mapping libraries

这里我分享一下使用 Ruby 来操作 MongoDB 数据库的相关基础，详细可以参考 **[Ruby Driver Tutorial][ruby_driver]**


> **Tip:**   当前的最新版本为 **mongo 2.2.5**

---


# 概要

* TOC
{:toc}

---


## 环境


~~~
[root@h102 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 ~]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 ~]# ruby -v 
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
[root@h102 ~]#
~~~

---


## 安装 mongo


~~~
[root@h102 ~]# gem source -l 
*** CURRENT SOURCES ***

https://gems.ruby-china.org
[root@h102 ~]# gem install mongo
Fetching: bson-4.1.1.gem (100%)
Building native extensions.  This could take a while...
Successfully installed bson-4.1.1
Fetching: mongo-2.2.5.gem (100%)
Successfully installed mongo-2.2.5
Parsing documentation for bson-4.1.1
Installing ri documentation for bson-4.1.1
Parsing documentation for mongo-2.2.5
Installing ri documentation for mongo-2.2.5
Done installing documentation for bson, mongo after 8 seconds
2 gems installed
[root@h102 ~]# gem list | grep mongo
mongo (2.2.5)
[root@h102 ~]#
~~~


> **Tip:** 确认一下安装源，否则可能被墙，速度慢得没法忍

---

## 兼容性


### 不同版本 MongoDB

下面这张表是不同版本 Ruby Driver 与不同版本 MongoDB 的兼容性列表

|Ruby Driver|	MongoDB 2.4	|MongoDB 2.6	|MongoDB 3.0	|MongoDB 3.2|
| :---- | :----: | :---: |:----: | :---: |:----: |
|2.2	|✓|	✓|	✓|	✓|
|2.0	|✓|	✓|	✓|	 |
|1.12	|✓|	✓|	✓|	 |


### 不同版本 Ruby 语言

下面这张表是不同版本 Ruby Driver 与不同版本 Ruby 语言的兼容性列表

|Ruby Driver|	Ruby 1.8.7|	Ruby 1.9|	Ruby 2.0|	Ruby 2.1|	JRuby|
| :---- | :----: | :---: |:----: | :---: |:----: |
|2.0	| 	|✓	|✓	|✓	|✓|
|1.9	|✓	|✓	|✓	|✓	|✓|


### 不同版本 MongoDB 和 不同版本 Ruby 

下面这张表是在不同版本 Ruby 语言，不同版本的 MongoDB 中此 Ruby Driver (mongo 2.2.5) 是否兼容的列表


|Ruby Version	|2.4.x	|2.6.x	|3.0.x|
| :---- | :----: | :---: |:----: | 
|MRI 1.8.x	|No	|No	|No|
|MRI 1.9.x	|Yes	|Yes	|Yes|
|MRI 2.0.x	|Yes	|Yes	|Yes|
|MRI 2.1.x	|Yes	|Yes	|Yes|
|MRI 2.2.x	|Yes	|Yes	|Yes|
|JRuby 1.7.x	|Yes	|Yes	|Yes|


> **Note:** 之所以这么强调兼容性，是要尽量在生产中避免由于兼容产生的隐患，自己写的小工具出现问题还可以随便改换过来，但是生产环境下，不是那么容易获得系统停机窗口的，并且不同版本之间的小差异可能产生调用的失败，在大量代码已经完成的情况下，再次改写是很疼的，所以前期的规划很重要，尽量减少这些潜在隐患发生的可能

---

## 连接数据库

可以使用两种方式连接 mongo

~~~
[root@h102 mysql]# irb
2.3.0 :001 > require 'mongo'
 => true 
2.3.0 :002 > c = Mongo::Client.new(['192.168.100.105:27017'],:database => 'post')
D, [2016-05-25T22:11:05.617907 #36607] DEBUG -- : MONGODB | Adding 192.168.100.105:27017 to the cluster.
 => #<Mongo::Client:0x22859300 cluster=192.168.100.105:27017> 
2.3.0 :003 > c1 = Mongo::Client.new('mongodb://192.168.100.105:27017/post')
D, [2016-05-25T22:11:13.529810 #36607] DEBUG -- : MONGODB | Adding 192.168.100.105:27017 to the cluster.
 => #<Mongo::Client:0x26735480 cluster=192.168.100.105:27017> 
2.3.0 :004 > c.inspect
 => "#<Mongo::Client:0x22859300 cluster=192.168.100.105:27017>" 
2.3.0 :005 > c1.inspect
 => "#<Mongo::Client:0x26735480 cluster=192.168.100.105:27017>" 
2.3.0 :006 > c.itself
 => #<Mongo::Client:0x22859300 cluster=192.168.100.105:27017> 
2.3.0 :007 > c1.itself
 => #<Mongo::Client:0x26735480 cluster=192.168.100.105:27017> 
2.3.0 :008 >
2.3.0 :009 > c.class
 => Mongo::Client 
2.3.0 :010 > c1.class
 => Mongo::Client 
2.3.0 :011 >
~~~


> **Tip:** 创建连接的过程中可以添加很多其它的选项，以修改初始化连接的特性，详细可以参考 **[Client Options][client_options]** 和 **[Ruby Options][ruby_options]** 还有 **[Details on timeout options][timeout_options]**

---

## 插入数据


### 插入一条数据

~~~
2.3.0 :025 > r = c[:abctest].insert_one({name: 'justfortest'})
D, [2016-05-25T22:23:11.090176 #36607] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.insert | STARTED | {"insert"=>"abctest", "documents"=>[{:name=>"justfortest", :_id=>BSON::ObjectId('5745b54ff677048eff545bc5')}], "ordered"=>true}
D, [2016-05-25T22:23:11.095860 #36607] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.insert | SUCCEEDED | 0.005496831s
 => #<Mongo::Operation::Result:26962180 documents=[{"ok"=>1, "n"=>1}]> 
2.3.0 :026 > r.n
 => 1 
2.3.0 :027 > r.class
 => Mongo::Operation::Write::Insert::Result 
2.3.0 :028 >
~~~


查看插入结果的反馈

~~~
2.3.0 :040 > r.ok?
 => true 
2.3.0 :041 > r.one?
 => true 
2.3.0 :042 > r.display
#<Mongo::Operation::Write::Insert::Result:0x0000000336d208> => nil 
2.3.0 :043 > r.inspect
 => "#<Mongo::Operation::Result:26962180 documents=[{\"ok\"=>1, \"n\"=>1}]>" 
2.3.0 :044 > 
~~~


---

### 插入多条数据

~~~
2.3.0 :063 > r = c[:abctest].insert_many([{:name => 'abc'},{:name => 'def' },{:name => 'ghi' },{:name => 'jkl'}])
D, [2016-05-25T22:30:43.872507 #36607] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.insert | STARTED | {"insert"=>"abctest", "documents"=>[{:name=>"abc", :_id=>BSON::ObjectId('5745b713f677048eff545bca')}, {:name=>"def", :_id=>BSON::ObjectId('5745b713f677048eff545bcb')}, {:name=>"ghi", :_id=>BSON::ObjectId('5745b713f677048eff545bcc')}, {:name=>"jkl", :_...
D, [2016-05-25T22:30:43.886054 #36607] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.insert | SUCCEEDED | 0.01328074s
 => #<Mongo::BulkWrite::Result:0x000000033b0058 @results={"n_inserted"=>4, "n"=>4, "inserted_ids"=>[BSON::ObjectId('5745b713f677048eff545bca'), BSON::ObjectId('5745b713f677048eff545bcb'), BSON::ObjectId('5745b713f677048eff545bcc'), BSON::ObjectId('5745b713f677048eff545bcd')]}> 
2.3.0 :064 > r.inspect
 => "#<Mongo::BulkWrite::Result:0x000000033b0058 @results={\"n_inserted\"=>4, \"n\"=>4, \"inserted_ids\"=>[BSON::ObjectId('5745b713f677048eff545bca'), BSON::ObjectId('5745b713f677048eff545bcb'), BSON::ObjectId('5745b713f677048eff545bcc'), BSON::ObjectId('5745b713f677048eff545bcd')]}>" 
2.3.0 :065 > r.class
 => Mongo::BulkWrite::Result 
2.3.0 :066 > 
~~~

> **Tip:** 根据文档中的 **`.n`** 其实已经没有了，如果调用会出现如下报错

~~~
2.3.0 :077 > r.n
NoMethodError: undefined method `n' for #<Mongo::BulkWrite::Result:0x000000033b0058>
	from (irb):77
	from /usr/local/rvm/rubies/ruby-2.3.0/bin/irb:11:in `<main>'
2.3.0 :078 > 
~~~

---


## 查询数据

~~~
[root@h102 ~]# irb
2.3.0 :001 > require 'mongo'
 => true 
2.3.0 :002 > c = Mongo::Client.new(['192.168.100.105:27017'],:database => 'post')
D, [2016-05-26T11:53:51.447336 #5174] DEBUG -- : MONGODB | Adding 192.168.100.105:27017 to the cluster.
 => #<Mongo::Client:0x9375760 cluster=192.168.100.105:27017> 
2.3.0 :003 >  c[:abctest].find(:name => 'abc').each do |x|
2.3.0 :004 >     puts x.class, x["_id"],x["name"]
2.3.0 :005?>   end
D, [2016-05-26T11:54:18.273709 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"abc"}}
D, [2016-05-26T11:54:18.280939 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.006918876s
BSON::Document
5745b6aaf677048eff545bc6
abc
BSON::Document
5745b713f677048eff545bca
abc
 => #<Enumerator: #<Mongo::Cursor:0x13268780 @view=#<Mongo::Collection::View:0x9305440 namespace='post.abctest' @filter={"name"=>"abc"} @options={}>>:each> 
2.3.0 :006 >
~~~

修改查询特性

~~~
2.3.0 :076 > c[:abctest].find().each do |x|
2.3.0 :077 >     printf("%s\t=>\t%s\n",x["_id"],x["name"])
2.3.0 :078?>   end
D, [2016-05-26T13:47:08.884140 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{}}
D, [2016-05-26T13:47:08.886403 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002051126s
5745b4aaf677048eff545bc4	=>	justfortest
5745b54ff677048eff545bc5	=>	justfortest
5745b6aaf677048eff545bc6	=>	abc
5745b6aaf677048eff545bc7	=>	def
5745b6aaf677048eff545bc8	=>	ghi
5745b6aaf677048eff545bc9	=>	jkl
5745b713f677048eff545bca	=>	abc
5745b713f677048eff545bcb	=>	def
5745b713f677048eff545bcc	=>	ghi
5745b713f677048eff545bcd	=>	jkl
 => #<Enumerator: #<Mongo::Cursor:0x9358000 @view=#<Mongo::Collection::View:0x9377420 namespace='post.abctest' @filter={} @options={}>>:each> 
2.3.0 :079 > c[:abctest].find().skip(3).limit(5).each do |x|
2.3.0 :080 >     printf("%s\t=>\t%s\n",x["_id"],x["name"])
2.3.0 :081?>   end
D, [2016-05-26T13:47:28.682917 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{}, "skip"=>3, "limit"=>5}
D, [2016-05-26T13:47:28.686809 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.003669962s
5745b6aaf677048eff545bc7	=>	def
5745b6aaf677048eff545bc8	=>	ghi
5745b6aaf677048eff545bc9	=>	jkl
5745b713f677048eff545bca	=>	abc
5745b713f677048eff545bcb	=>	def
 => #<Enumerator: #<Mongo::Cursor:0x9118740 @view=#<Mongo::Collection::View:0x9128780 namespace='post.abctest' @filter={} @options={"skip"=>3, "limit"=>5}>>:each> 
2.3.0 :082 > c[:abctest].find({:name => 'abc'}).limit(1).each do |x|
2.3.0 :083 >     printf("%s\t=>\t%s\n",x["_id"],x["name"])
2.3.0 :084?>   end
D, [2016-05-26T13:48:38.397344 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"abc"}, "limit"=>1}
D, [2016-05-26T13:48:38.400765 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.003194933s
5745b6aaf677048eff545bc6	=>	abc
 => #<Enumerator: #<Mongo::Cursor:0x7675560 @view=#<Mongo::Collection::View:0x7750520 namespace='post.abctest' @filter={"name"=>"abc"} @options={"limit"=>1}>>:each> 
2.3.0 :085 > 
~~~

> **Tip:** 创建查询的过程中可以添加很多其它选项，以修改查询的特性，详细可以参考 **[Query Options][query_options]** 和 **[Additional Query Operations][query_options2]** 


~~~
2.3.0 :085 > c[:abctest].find({:name => 'abc'}).count
D, [2016-05-26T13:52:22.366743 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.count | STARTED | {"count"=>"abctest", "query"=>{"name"=>"abc"}}
D, [2016-05-26T13:52:22.369058 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.count | SUCCEEDED | 0.0015160509999999998s
 => 2 
2.3.0 :086 > c[:abctest].find.distinct(:name)
D, [2016-05-26T13:53:19.399217 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.distinct | STARTED | {"distinct"=>"abctest", "key"=>"name", "query"=>{}}
D, [2016-05-26T13:53:19.402953 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.distinct | SUCCEEDED | 0.003509715s
 => ["justfortest", "abc", "def", "ghi", "jkl"] 
2.3.0 :087 >
~~~

---


## 修改数据


我们可以直接使用 Collection 来引用操作


~~~
2.3.0 :118 > a = c[:abctest]
 => #<Mongo::Collection:0x12898100 namespace=post.abctest> 
2.3.0 :119 > a.class
 => Mongo::Collection 
2.3.0 :120 > a.inspect
 => "#<Mongo::Collection:0x12898100 namespace=post.abctest>" 
2.3.0 :121 > a.name
 => "abctest" 
2.3.0 :122 > a.namespace
 => "post.abctest" 
2.3.0 :123 > a.itself
 => #<Mongo::Collection:0x12898100 namespace=post.abctest> 
2.3.0 :124 > a.cluster
 => #<Mongo::Cluster:0x9375440 servers=[#<Mongo::Server:0x9373660 address=192.168.100.105:27017>] topology=Single> 
2.3.0 :125 > a.client
 => #<Mongo::Client:0x9375760 cluster=192.168.100.105:27017> 
2.3.0 :126 > a.count
D, [2016-05-26T14:46:11.460020 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.count | STARTED | {"count"=>"abctest", "query"=>{}}
D, [2016-05-26T14:46:11.463027 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.count | SUCCEEDED | 0.002638529s
 => 10 
2.3.0 :127 > 
~~~

---


### 更新一条数据


~~~
2.3.0 :173 > a.find(:name => 'justfortest').each do |x|
2.3.0 :174 >     puts x
2.3.0 :175?>   end
D, [2016-05-26T15:41:11.761639 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T15:41:11.766917 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.005021206s
{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"justfortest"}
{"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest"}
 => #<Enumerator: #<Mongo::Cursor:0x13474400 @view=#<Mongo::Collection::View:0x13478700 namespace='post.abctest' @filter={"name"=>"justfortest"} @options={}>>:each> 
2.3.0 :176 > a.find(:name => 'justfortest').update_one("$inc" => { :newfiled => 1 })
D, [2016-05-26T15:43:04.517881 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{"$inc"=>{:newfiled=>1}}, "multi"=>false, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T15:43:04.520336 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.002116027s
 => #<Mongo::Operation::Result:13420760 documents=[{"ok"=>1, "nModified"=>1, "n"=>1}]> 
2.3.0 :177 > a.find(:name => 'justfortest').each do |x|
2.3.0 :178 >     puts x
2.3.0 :179?>   end
D, [2016-05-26T15:43:13.598368 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T15:43:13.602324 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.003671943s
{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"justfortest", "newfiled"=>1}
{"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest"}
 => #<Enumerator: #<Mongo::Cursor:0x70030381400740 @view=#<Mongo::Collection::View:0x70030381405400 namespace='post.abctest' @filter={"name"=>"justfortest"} @options={}>>:each> 
2.3.0 :180 > 
~~~

---

### 捕获更新反馈结果

~~~
2.3.0 :184 > r = a.find(:name => 'justfortest').update_one("$inc" => { :newfiled => 1 })
D, [2016-05-26T15:45:17.110646 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{"$inc"=>{:newfiled=>1}}, "multi"=>false, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T15:45:17.112755 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.001860819s
 => #<Mongo::Operation::Result:70030381342520 documents=[{"ok"=>1, "nModified"=>1, "n"=>1}]> 
2.3.0 :185 > a.find(:name => 'justfortest').each do |x|
2.3.0 :186 >     puts x
2.3.0 :187?>   end
D, [2016-05-26T15:45:31.859453 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T15:45:31.865512 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.005811567s
{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"justfortest", "newfiled"=>2}
{"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest"}
 => #<Enumerator: #<Mongo::Cursor:0x13244260 @view=#<Mongo::Collection::View:0x13256880 namespace='post.abctest' @filter={"name"=>"justfortest"} @options={}>>:each> 
2.3.0 :188 > r.n
 => 1 
2.3.0 :189 > r.class
 => Mongo::Operation::Write::Update::Result 
2.3.0 :190 > 
2.3.0 :191 > r.ok?
 => true 
2.3.0 :192 >
~~~

还可以直接使用 **update_one**

~~~
2.3.0 :194 > a.update_one({:name => 'justfortest'},{"$inc" => { :newfiled => 1 }})
D, [2016-05-26T15:49:58.408074 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{"$inc"=>{:newfiled=>1}}, "multi"=>false, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T15:49:58.410407 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.002170654s
 => #<Mongo::Operation::Result:12896540 documents=[{"ok"=>1, "nModified"=>1, "n"=>1}]> 
2.3.0 :195 >
2.3.0 :196 >
2.3.0 :197 > a.find(:name => 'justfortest').each do |x|
2.3.0 :198 >     puts x
2.3.0 :199?>   end
D, [2016-05-26T15:50:47.860722 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T15:50:47.862710 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0017079410000000001s
{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"justfortest", "newfiled"=>3}
{"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest"}
 => #<Enumerator: #<Mongo::Cursor:0x70030381883420 @view=#<Mongo::Collection::View:0x70030381888060 namespace='post.abctest' @filter={"name"=>"justfortest"} @options={}>>:each> 
2.3.0 :200 > 
~~~

---

### 更新多条数据


~~~
2.3.0 :232 > a.find(:name => 'justfortest').update_many("$inc" => { :newfiled => 1 })
D, [2016-05-26T15:57:44.801190 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{"$inc"=>{:newfiled=>1}}, "multi"=>true, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T15:57:44.804500 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.002955263s
 => #<Mongo::Operation::Result:70030381761480 documents=[{"ok"=>1, "nModified"=>2, "n"=>2}]> 
2.3.0 :233 > a.find(:name => 'justfortest').to_a
D, [2016-05-26T15:57:54.840148 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T15:57:54.844287 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0039025730000000003s
 => [{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"justfortest", "newfiled"=>4}, {"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest", "newfiled"=>1}] 
2.3.0 :234 > a.update_many({:name => 'justfortest'},{"$inc" => { :newfiled => 1 }})
D, [2016-05-26T15:59:29.274224 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{"$inc"=>{:newfiled=>1}}, "multi"=>true, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T15:59:29.277737 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.0031768710000000004s
 => #<Mongo::Operation::Result:70030381890320 documents=[{"ok"=>1, "nModified"=>2, "n"=>2}]> 
2.3.0 :235 > a.find(:name => 'justfortest').to_a
D, [2016-05-26T15:59:32.245721 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T15:59:32.248552 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002961119s
 => [{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"justfortest", "newfiled"=>5}, {"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest", "newfiled"=>2}] 
2.3.0 :236 >
~~~

---

### 替换一条数据


~~~
2.3.0 :240 > a.find(:name => 'justfortest').replace_one(:name => 'replaceTest')
D, [2016-05-26T16:05:59.441006 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{:name=>"replaceTest"}, "multi"=>false, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T16:05:59.466799 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.025567399s
 => #<Mongo::Operation::Result:9868040 documents=[{"ok"=>1, "nModified"=>1, "n"=>1}]> 
2.3.0 :241 > a.find(:name => 'justfortest').to_a
D, [2016-05-26T16:06:04.461909 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T16:06:04.466557 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0043987639999999995s
 => [{"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"justfortest", "newfiled"=>2}] 
2.3.0 :242 > a.find(:name => 'replaceTest').to_a
D, [2016-05-26T16:06:19.565133 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"replaceTest"}}
D, [2016-05-26T16:06:19.569626 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.004263940999999999s
 => [{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"replaceTest"}] 
2.3.0 :243 > a.replace_one({:name => 'justfortest'},{:name => 'replaceTest'})
D, [2016-05-26T16:12:22.103523 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | STARTED | {"update"=>"abctest", "updates"=>[{"q"=>{"name"=>"justfortest"}, "u"=>{:name=>"replaceTest"}, "multi"=>false, "upsert"=>false}], "ordered"=>true}
D, [2016-05-26T16:12:22.106802 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.update | SUCCEEDED | 0.003047168s
 => #<Mongo::Operation::Result:12647000 documents=[{"ok"=>1, "nModified"=>1, "n"=>1}]> 
2.3.0 :244 > a.find(:name => 'replaceTest').to_a
D, [2016-05-26T16:12:27.783514 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"replaceTest"}}
D, [2016-05-26T16:12:27.786342 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0025952189999999997s
 => [{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"replaceTest"}, {"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"replaceTest"}] 
2.3.0 :245 > a.find(:name => 'justfortest').to_a
D, [2016-05-26T16:12:51.175450 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"justfortest"}}
D, [2016-05-26T16:12:51.180454 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.004738488s
 => [] 
2.3.0 :246 >
~~~

> **Note:**  替换是整个文档替换，可能造成数据丢失，要非常小心



---


### 查找并删除一条数据

~~~
2.3.0 :259 > a.find(:name => 'def').to_a
D, [2016-05-26T16:30:57.159722 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"def"}}
D, [2016-05-26T16:30:57.163661 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.003645691s
 => [{"_id"=>BSON::ObjectId('5745b6aaf677048eff545bc7'), "name"=>"def"}, {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"def"}] 
2.3.0 :260 > a.find(:name => 'def').find_one_and_delete
D, [2016-05-26T16:35:55.478816 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | STARTED | {"findandmodify"=>"abctest", "query"=>{"name"=>"def"}, "remove"=>true}
D, [2016-05-26T16:35:55.550119 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | SUCCEEDED | 0.07105563299999999s
 => {"_id"=>BSON::ObjectId('5745b6aaf677048eff545bc7'), "name"=>"def"} 
2.3.0 :261 > a.find(:name => 'def').to_a
D, [2016-05-26T16:35:59.540622 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"def"}}
D, [2016-05-26T16:35:59.543734 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002624591s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"def"}] 
2.3.0 :262 > 
~~~


---

### 查找并替换一条数据


~~~
2.3.0 :282 > a.find(:name => 'def').to_a
D, [2016-05-26T16:38:16.187679 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"def"}}
D, [2016-05-26T16:38:16.193693 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0056998s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"def"}] 
2.3.0 :283 > a.find(:name => 'def').find_one_and_replace(:name => 'xxx')
D, [2016-05-26T16:38:50.123162 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | STARTED | {"findandmodify"=>"abctest", "query"=>{"name"=>"def"}, "update"=>{:name=>"xxx"}, "new"=>false, "bypassDocumentValidation"=>false}
D, [2016-05-26T16:38:50.126430 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | SUCCEEDED | 0.003046483s
 => {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"def"} 
2.3.0 :284 > a.find(:name => 'def').to_a
D, [2016-05-26T16:38:53.169636 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"def"}}
D, [2016-05-26T16:38:53.174287 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.004290449s
 => [] 
2.3.0 :285 > a.find(:name => 'xxx').to_a
D, [2016-05-26T16:38:56.631101 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"xxx"}}
D, [2016-05-26T16:38:56.634794 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.003439957s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"xxx"}] 
2.3.0 :286 > 
~~~


另外一种形式

~~~
2.3.0 :299 > a.find_one_and_replace({:name => 'xxx'},{:name => 'yyy'})
D, [2016-05-26T16:40:24.705708 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | STARTED | {"findandmodify"=>"abctest", "query"=>{"name"=>"xxx"}, "update"=>{:name=>"yyy"}, "new"=>false, "bypassDocumentValidation"=>false}
D, [2016-05-26T16:40:24.707865 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | SUCCEEDED | 0.0019451739999999998s
 => {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"xxx"} 
2.3.0 :300 > a.find(:name => 'xxx').to_a
D, [2016-05-26T16:40:27.641622 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"xxx"}}
D, [2016-05-26T16:40:27.646195 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.004332849999999999s
 => [] 
2.3.0 :301 > a.find(:name => 'yyy').to_a
D, [2016-05-26T16:40:33.807708 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"yyy"}}
D, [2016-05-26T16:40:33.810583 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0026168429999999998s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"yyy"}] 
2.3.0 :302 > 
~~~


---

### 查找并更新一条数据


~~~
2.3.0 :322 > a.find(:name => 'zzz').to_a
D, [2016-05-26T16:57:07.911844 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"zzz"}}
D, [2016-05-26T16:57:07.913529 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.001439203s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"zzz"}] 
2.3.0 :323 > a.find(:name => 'zzz').find_one_and_update('$set' => {:ui => 'CLI'})
D, [2016-05-26T16:57:43.518176 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | STARTED | {"findandmodify"=>"abctest", "query"=>{"name"=>"zzz"}, "update"=>{"$set"=>{:ui=>"CLI"}}, "new"=>false, "bypassDocumentValidation"=>false}
D, [2016-05-26T16:57:43.522056 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | SUCCEEDED | 0.00366088s
 => {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"zzz"} 
2.3.0 :324 > a.find(:name => 'zzz').to_a
D, [2016-05-26T16:57:46.291361 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"zzz"}}
D, [2016-05-26T16:57:46.294192 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002366816s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"zzz", "ui"=>"CLI"}] 
2.3.0 :325 > 
~~~

另一种形式

可以把结果集进行保存


~~~
2.3.0 :333 > d = a.find_one_and_update({"name"=>"zzz"},{"ui"=>"GUI"})
D, [2016-05-26T17:02:15.633513 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | STARTED | {"findandmodify"=>"abctest", "query"=>{"name"=>"zzz"}, "update"=>{"ui"=>"GUI"}, "new"=>false, "bypassDocumentValidation"=>false}
D, [2016-05-26T17:02:15.637428 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.findandmodify | SUCCEEDED | 0.003628107s
 => {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"zzz", "ui"=>"GUI"} 
2.3.0 :334 > d.class
 => BSON::Document 
2.3.0 :335 > d
 => {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "name"=>"zzz", "ui"=>"GUI"} 
2.3.0 :336 > 
~~~

> **Note:** 虽然用的是 **`find_one_and_update`** 方法，但其实这个过程是在替换，要分外小心

~~~
2.3.0 :340 > a.find({"name"=>"zzz"}).to_a
D, [2016-05-26T17:23:52.321931 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"zzz"}}
D, [2016-05-26T17:23:52.323976 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0018150710000000001s
 => [] 
2.3.0 :341 > a.find({"ui"=>"GUI"}).to_a
D, [2016-05-26T17:24:14.688982 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"ui"=>"GUI"}}
D, [2016-05-26T17:24:14.691677 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002366695s
 => [{"_id"=>BSON::ObjectId('5745b6aaf677048eff545bc8'), "ui"=>"GUI"}, {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "ui"=>"GUI"}] 
2.3.0 :342 > 
~~~


---

## 删除数据 


### 删除一条数据


~~~
2.3.0 :353 > a.find({"ui"=>"GUI"}).to_a
D, [2016-05-26T17:27:34.928755 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"ui"=>"GUI"}}
D, [2016-05-26T17:27:34.930674 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0016437049999999999s
 => [{"_id"=>BSON::ObjectId('5745b6aaf677048eff545bc8'), "ui"=>"GUI"}, {"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "ui"=>"GUI"}] 
2.3.0 :354 > r = a.find({"ui"=>"GUI"}).delete_one
D, [2016-05-26T17:29:53.396121 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | STARTED | {"delete"=>"abctest", "deletes"=>[{"q"=>{"ui"=>"GUI"}, "limit"=>1}], "ordered"=>true}
D, [2016-05-26T17:29:53.398662 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | SUCCEEDED | 0.0023097839999999996s
 => #<Mongo::Operation::Result:12774140 documents=[{"ok"=>1, "n"=>1}]> 
2.3.0 :355 > r.n
 => 1 
2.3.0 :356 > a.find({"ui"=>"GUI"}).to_a
D, [2016-05-26T17:30:10.157727 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"ui"=>"GUI"}}
D, [2016-05-26T17:30:10.160650 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002679249s
 => [{"_id"=>BSON::ObjectId('5745b713f677048eff545bcb'), "ui"=>"GUI"}] 
2.3.0 :357 > a.delete_one("ui"=>"GUI")
D, [2016-05-26T17:30:34.906641 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | STARTED | {"delete"=>"abctest", "deletes"=>[{"q"=>{"ui"=>"GUI"}, "limit"=>1}], "ordered"=>true}
D, [2016-05-26T17:30:34.909964 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | SUCCEEDED | 0.003001837s
 => #<Mongo::Operation::Result:10403640 documents=[{"ok"=>1, "n"=>1}]> 
2.3.0 :358 > a.find({"ui"=>"GUI"}).to_a
D, [2016-05-26T17:30:43.650621 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"ui"=>"GUI"}}
D, [2016-05-26T17:30:43.653355 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002408293s
 => [] 
2.3.0 :359 >
~~~


---

### 删除多条数据

~~~
2.3.0 :359 > a.find({"name"=>"jkl"}).to_a
D, [2016-05-26T17:36:08.230221 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"jkl"}}
D, [2016-05-26T17:36:08.232321 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0016569990000000001s
 => [{"_id"=>BSON::ObjectId('5745b6aaf677048eff545bc9'), "name"=>"jkl"}, {"_id"=>BSON::ObjectId('5745b713f677048eff545bcd'), "name"=>"jkl"}] 
2.3.0 :360 > a.find({"name"=>"jkl"}).delete_many
D, [2016-05-26T17:36:38.044545 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | STARTED | {"delete"=>"abctest", "deletes"=>[{"q"=>{"name"=>"jkl"}, "limit"=>0}], "ordered"=>true}
D, [2016-05-26T17:36:38.048958 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | SUCCEEDED | 0.004232251s
 => #<Mongo::Operation::Result:9183680 documents=[{"ok"=>1, "n"=>2}]> 
2.3.0 :361 > a.find({"name"=>"jkl"}).to_a
D, [2016-05-26T17:36:41.693029 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"jkl"}}
D, [2016-05-26T17:36:41.695807 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002505381s
 => [] 
2.3.0 :362 >
~~~ 

另一种形式


~~~
2.3.0 :362 > a.find({"name"=>"replaceTest"}).to_a
D, [2016-05-26T17:37:51.179071 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"replaceTest"}}
D, [2016-05-26T17:37:51.185426 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.006037441s
 => [{"_id"=>BSON::ObjectId('5745b4aaf677048eff545bc4'), "name"=>"replaceTest"}, {"_id"=>BSON::ObjectId('5745b54ff677048eff545bc5'), "name"=>"replaceTest"}] 
2.3.0 :363 > a.delete_many(:name => "replaceTest")
D, [2016-05-26T17:38:41.258842 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | STARTED | {"delete"=>"abctest", "deletes"=>[{"q"=>{"name"=>"replaceTest"}, "limit"=>0}], "ordered"=>true}
D, [2016-05-26T17:38:41.261118 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.delete | SUCCEEDED | 0.001905545s
 => #<Mongo::Operation::Result:6844980 documents=[{"ok"=>1, "n"=>2}]> 
2.3.0 :364 > a.find({"name"=>"replaceTest"}).to_a
D, [2016-05-26T17:38:43.690739 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{"name"=>"replaceTest"}}
D, [2016-05-26T17:38:43.692741 #5174] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.0017209950000000001s
 => [] 
2.3.0 :365 >
~~~

---


## Collections 操作


### 创建集合

~~~
[root@h102 ~]# irb
2.3.0 :001 > require 'mongo'
 => true 
2.3.0 :002 > c = Mongo::Client.new([ '192.168.100.105:27017' ], :database => 'post')
D, [2016-05-26T22:26:43.917490 #32905] DEBUG -- : MONGODB | Adding 192.168.100.105:27017 to the cluster.
 => #<Mongo::Client:0x7687020 cluster=192.168.100.105:27017> 
2.3.0 :003 > newtable = c[:newtable, :capped => true, :size => 1024]
 => #<Mongo::Collection:0x11552180 namespace=post.newtable> 
2.3.0 :004 >
~~~

此时进行本地检查

~~~
> show tables;
abctest
post
user
users
> 
~~~

还有没生成表

然后我进行创建

~~~
2.3.0 :004 > newtable.create
D, [2016-05-26T22:28:34.677356 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.create | STARTED | {"create"=>"newtable", "capped"=>true, "size"=>1024}
D, [2016-05-26T22:28:34.740613 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.create | SUCCEEDED | 0.062936352s
 => #<Mongo::Operation::Result:11601440 documents=[{"ok"=>1.0}]> 
2.3.0 :005 >
~~~

再看本地

~~~
> show tables;
abctest
newtable
post
user
users
> 
~~~

---

### 删除集合


~~~
2.3.0 :005 > newtable.capped?
D, [2016-05-26T22:29:24.455870 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.collstats | STARTED | {"collstats"=>"newtable"}
D, [2016-05-26T22:29:24.464827 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.collstats | SUCCEEDED | 0.008704707s
 => true 
2.3.0 :006 > newtable.drop
D, [2016-05-26T22:30:47.021025 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.drop | STARTED | {"drop"=>"newtable"}
D, [2016-05-26T22:30:47.030352 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.drop | SUCCEEDED | 0.00903782s
 => #<Mongo::Operation::Result:11770800 documents=[{"ns"=>"post.newtable", "nIndexesWas"=>1, "ok"=>1.0}]> 
2.3.0 :007 > newtable.capped?
D, [2016-05-26T22:31:18.597838 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.collstats | STARTED | {"collstats"=>"newtable"}
D, [2016-05-26T22:31:18.601270 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.collstats | FAILED | Collection [post.newtable] not found. () | 0.003093439s
Mongo::Error::OperationFailure: Collection [post.newtable] not found. ()
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/operation/result.rb:256:in `validate!'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/operation/executable.rb:36:in `block in execute'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/server/connection_pool.rb:108:in `with_connection'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/server/context.rb:63:in `with_connection'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/operation/executable.rb:34:in `execute'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/database.rb:158:in `command'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/mongo-2.2.5/lib/mongo/collection.rb:162:in `capped?'
	from (irb):7
	from /usr/local/rvm/rubies/ruby-2.3.0/bin/irb:11:in `<main>'
2.3.0 :008 >
~~~


本地查看，发现 **newtable** 没了

~~~
> show tables;
abctest
post
user
users
> 
~~~

还可以修改读写的倾向性，可以参考 **[Changing Read/Write Preferences][rw_preferences]**



---


## 数据库操作



### 获取数据库名

~~~
2.3.0 :021 > db1 = c.database
 => #<Mongo::Database:0x11515220 name=post> 
2.3.0 :022 > db1.class
 => Mongo::Database 
2.3.0 :023 > db1.name
 => "post" 
2.3.0 :024 > 
~~~

---

### 获取数据库中集合名

~~~
2.3.0 :024 > db1.collections
D, [2016-05-26T22:49:11.426246 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | STARTED | {"listCollections"=>1, "cursor"=>{}, "filter"=>{:name=>{"$not"=>/system\.|\$/}}}
D, [2016-05-26T22:49:11.429074 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | SUCCEEDED | 0.002431477s
 => [#<Mongo::Collection:0x12300880 namespace=post.users>, #<Mongo::Collection:0x12300840 namespace=post.abctest>, #<Mongo::Collection:0x12300800 namespace=post.user>, #<Mongo::Collection:0x12300760 namespace=post.post>] 
2.3.0 :025 > db1.collection_names
D, [2016-05-26T22:49:18.891294 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | STARTED | {"listCollections"=>1, "cursor"=>{}, "filter"=>{:name=>{"$not"=>/system\.|\$/}}}
D, [2016-05-26T22:49:18.894400 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | SUCCEEDED | 0.002841808s
 => ["users", "abctest", "user", "post"] 
2.3.0 :026 >
~~~

---

### 在数据库中执行命令

~~~
2.3.0 :029 > r = db1.command(:ismaster => 1)
D, [2016-05-26T22:51:50.618824 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.ismaster | STARTED | {"ismaster"=>1}
D, [2016-05-26T22:51:50.620750 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.ismaster | SUCCEEDED | 0.00166754s
 => #<Mongo::Operation::Result:11736140 documents=[{"ismaster"=>true, "maxBsonObjectSize"=>16777216, "maxMessageSizeBytes"=>48000000, "maxWriteBatchSize"=>1000, "localTime"=>2016-05-26 14:51:50 UTC, "maxWireVersion"=>4, "minWireVersion"=>0, "ok"=>1.0}]> 
2.3.0 :030 > r.first
 => {"ismaster"=>true, "maxBsonObjectSize"=>16777216, "maxMessageSizeBytes"=>48000000, "maxWriteBatchSize"=>1000, "localTime"=>2016-05-26 14:51:50 UTC, "maxWireVersion"=>4, "minWireVersion"=>0, "ok"=>1.0} 
2.3.0 :031 >
~~~

---

### 删除数据库

~~~
2.3.0 :035 > db1.name
 => "post" 
2.3.0 :036 > db1.drop
D, [2016-05-26T22:53:10.397049 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.dropDatabase | STARTED | {"dropDatabase"=>1}
D, [2016-05-26T22:53:10.484772 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.dropDatabase | SUCCEEDED | 0.087418383s
 => #<Mongo::Operation::Result:12008520 documents=[{"dropped"=>"post", "ok"=>1.0}]> 
2.3.0 :037 > db1.name
 => "post" 
2.3.0 :038 > db1.collection_names
D, [2016-05-26T22:53:28.778302 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | STARTED | {"listCollections"=>1, "cursor"=>{}, "filter"=>{:name=>{"$not"=>/system\.|\$/}}}
D, [2016-05-26T22:53:28.780969 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | SUCCEEDED | 0.0023564479999999997s
 => [] 
2.3.0 :039 >
~~~

发现名字还在，但是下面的所有集合都没了，到本地看看

~~~
> show tables;
> show dbs
local  0.000GB
>
~~~

本地 post 数据库已经不存在了

---

### 创建数据库

即便一个库不存在，如果往这个库里插入数据，就会连同集合一起，自动被创建

上面的操作过程中已经将 post 数据库删除了，于是我执行下面的语句

~~~
2.3.0 :051 > db1[:abctest].insert_one({name: 'justfortest'})
D, [2016-05-26T22:58:31.161257 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.insert | STARTED | {"insert"=>"abctest", "documents"=>[{:name=>"justfortest", :_id=>BSON::ObjectId('57470f17f677048089c7f028')}], "ordered"=>true}
D, [2016-05-26T22:58:31.197713 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.insert | SUCCEEDED | 0.036217863s
 => #<Mongo::Operation::Result:11204100 documents=[{"ok"=>1, "n"=>1}]> 
2.3.0 :052 > db1.collection_names
D, [2016-05-26T22:58:42.023711 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | STARTED | {"listCollections"=>1, "cursor"=>{}, "filter"=>{:name=>{"$not"=>/system\.|\$/}}}
D, [2016-05-26T22:58:42.060270 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listCollections | SUCCEEDED | 0.036180602s
 => ["abctest"] 
2.3.0 :053 > db1[:abctest].find().to_a
D, [2016-05-26T23:03:22.076758 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | STARTED | {"find"=>"abctest", "filter"=>{}}
D, [2016-05-26T23:03:22.079052 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.find | SUCCEEDED | 0.002194069s
 => [{"_id"=>BSON::ObjectId('57470f17f677048089c7f028'), "name"=>"justfortest"}] 
2.3.0 :054 >
~~~

本地查看一下

~~~
> show dbs
local  0.000GB
post   0.000GB
> use post
switched to db post
> show tables
abctest
> db.abctest.find()
{ "_id" : ObjectId("57470f17f677048089c7f028"), "name" : "justfortest" }
> 
~~~

看来 post 库和 abctest 表外加 "name" : "justfortest" 的记录一同被创建了

---

## 索引操作

### 创建索引 

MongoDB 3.0.0 之后的版本可以并行创建索引，之前的版本只能顺序创建

> Indexes can be created one at a time, or multiples can be created in a single operation. When creating multiples on MongoDB 3.0.0 and higher, the indexes will be created in parallel, otherwise they will be created in order.


#### 创建单个索引

~~~
2.3.0 :054 > db1[:abctest].indexes.create_one({ :name => 1 }, :unique => true)
D, [2016-05-26T23:11:11.304095 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.createIndexes | STARTED | {"reateIndexes"=>"abctest", "indexes"=>[{:key=>{:name=>1}, :unique=>true, :name=>"name_1"}]}
D, [2016-05-26T23:11:11.371189 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.createIndexes | SUCCEEDED | .066713294s
 => #<Mongo::Operation::Result:5602320 documents=[{"createdCollectionAutomatically"=>false, "numIndexesBefore"=>1, "nmIndexesAfter"=>2, "ok"=>1.0}]> 
2.3.0 :055 >
~~~

看本地

~~~
> db.abctest.getIndexes()
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "post.abctest"
	},
	{
		"v" : 1,
		"unique" : true,
		"key" : {
			"name" : 1
		},
		"name" : "name_1",
		"ns" : "post.abctest"
	}
]
> 
~~~

---

#### 创建多个索引

~~~
2.3.0 :056 > db1[:test2].indexes.create_many([{:key => { name: 1 }, :unique => true },{:key => { label: -1 }}])
D, [2016-05-26T23:27:27.426590 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.createIndexes | STARTED | {"createIndexes"=>"test2", "indexes"=>[{:key=>{:name=>1}, :unique=>true, :name=>"name_1"}, {:key=>{:label=>-1}, :name=>"label_-1"}]}
D, [2016-05-26T23:27:27.479450 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.createIndexes | SUCCEEDED | 0.052639038s
 => #<Mongo::Operation::Result:7996660 documents=[{"createdCollectionAutomatically"=>true, "numIndexesBefore"=>1, "numIndexesAfter"=>3, "ok"=>1.0}]> 
2.3.0 :057 > 
~~~

本地查看

~~~
> show tables
abctest
test2
> db.test2.getIndexes()
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "post.test2"
	},
	{
		"v" : 1,
		"unique" : true,
		"key" : {
			"name" : 1
		},
		"name" : "name_1",
		"ns" : "post.test2"
	},
	{
		"v" : 1,
		"key" : {
			"label" : -1
		},
		"name" : "label_-1",
		"ns" : "post.test2"
	}
]
> 
~~~

创建索引过程中还可以加入其它参数，详细参数可以参考 **[索引参数][indexes_opt]**


---

### 查看索引

~~~
2.3.0 :061 > db1[:test2].indexes.to_a
D, [2016-05-26T23:36:39.171127 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | STARTED | {"listIndexes"=>"test2", "cursor"=>{}}
D, [2016-05-26T23:36:39.203225 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | SUCCEEDED | 0.031920452s
 => [{"v"=>1, "key"=>{"_id"=>1}, "name"=>"_id_", "ns"=>"post.test2"}, {"v"=>1, "unique"=>true, "key"=>{"name"=>1}, "name"=>"name_1", "ns"=>"post.test2"}, {"v"=>1, "key"=>{"label"=>-1}, "name"=>"label_-1", "ns"=>"post.test2"}] 
2.3.0 :062 > db1[:test2].indexes.each do |i|
2.3.0 :063 >     p i
2.3.0 :064?>   end
D, [2016-05-26T23:37:24.375182 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | STARTED | {"listIndexes"=>"test2", "cursor"=>{}}
D, [2016-05-26T23:37:24.377380 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | SUCCEEDED | 0.00198784s
{"v"=>1, "key"=>{"_id"=>1}, "name"=>"_id_", "ns"=>"post.test2"}
{"v"=>1, "unique"=>true, "key"=>{"name"=>1}, "name"=>"name_1", "ns"=>"post.test2"}
{"v"=>1, "key"=>{"label"=>-1}, "name"=>"label_-1", "ns"=>"post.test2"}
 => #<Enumerator: #<Mongo::Cursor:0x10950640 @view=#<Mongo::Index::View:0x000000014d5a20 @collection=#<Mongo::Collection:0x10923340 namespace=post.test2>, @batch_size=nil>>:each> 
2.3.0 :065 > 
~~~ 


---

### 删除索引 


#### 删除一个索引

~~~
2.3.0 :078 > db1[:abctest].indexes.to_a
D, [2016-05-26T23:40:04.083421 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | STARTED | {"listIndexes"=>"abctest", "cursor"=>{}}
D, [2016-05-26T23:40:04.086508 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | SUCCEEDED | 0.0028248400000000003s
 => [{"v"=>1, "key"=>{"_id"=>1}, "name"=>"_id_", "ns"=>"post.abctest"}, {"v"=>1, "unique"=>true, "key"=>{"name"=>1}, "name"=>"name_1", "ns"=>"post.abctest"}] 
2.3.0 :079 > db1[:abctest].indexes.drop_one('name_1')
D, [2016-05-26T23:40:30.530875 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.dropIndexes | STARTED | {"dropIndexes"=>"abctest", "index"=>"name_1"}
D, [2016-05-26T23:40:30.563482 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.dropIndexes | SUCCEEDED | 0.032385475s
 => #<Mongo::Operation::Result:7589800 documents=[{"nIndexesWas"=>2, "ok"=>1.0}]> 
2.3.0 :080 > db1[:abctest].indexes.to_a
D, [2016-05-26T23:40:32.946595 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | STARTED | {"listIndexes"=>"abctest", "cursor"=>{}}
D, [2016-05-26T23:40:32.950582 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | SUCCEEDED | 0.0037632599999999996s
 => [{"v"=>1, "key"=>{"_id"=>1}, "name"=>"_id_", "ns"=>"post.abctest"}] 
2.3.0 :081 > 
~~~

#### 删除所有索引


~~~
2.3.0 :087 > db1[:test2].indexes.to_a
D, [2016-05-26T23:42:10.912880 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | STARTED | {"listIndexes"=>"test2", "cursor"=>{}}
D, [2016-05-26T23:42:10.916162 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | SUCCEEDED | 0.0028684260000000003s
 => [{"v"=>1, "key"=>{"_id"=>1}, "name"=>"_id_", "ns"=>"post.test2"}, {"v"=>1, "unique"=>true, "key"=>{"name"=>1}, "name"=>"name_1", "ns"=>"post.test2"}, {"v"=>1, "key"=>{"label"=>-1}, "name"=>"label_-1", "ns"=>"post.test2"}] 
2.3.0 :088 > db1[:test2].indexes.drop_all
D, [2016-05-26T23:42:19.955254 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.dropIndexes | STARTED | {"dropIndexes"=>"test2", "index"=>"*"}
D, [2016-05-26T23:42:19.979831 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.dropIndexes | SUCCEEDED | 0.024125167s
 => #<Mongo::Operation::Result:7885720 documents=[{"nIndexesWas"=>3, "msg"=>"non-_id indexes dropped for collection", "ok"=>1.0}]> 
2.3.0 :089 > db1[:test2].indexes.to_a
D, [2016-05-26T23:42:27.945764 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | STARTED | {"listIndexes"=>"test2", "cursor"=>{}}
D, [2016-05-26T23:42:27.948694 #32905] DEBUG -- : MONGODB | 192.168.100.105:27017 | post.listIndexes | SUCCEEDED | 0.0026218500000000002s
 => [{"v"=>1, "key"=>{"_id"=>1}, "name"=>"_id_", "ns"=>"post.test2"}] 
2.3.0 :090 > 
~~~


---

# 命令汇总

* **`gem source -l`**
* **`gem install mongo`**
* **`gem list | grep mongo`**
* **`irb`**


---


[rubygems]:https://rubygems.org/
[ruby_driver]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#ruby-driver-tutorial
[client_options]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#client-options
[ruby_options]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#ruby-options
[timeout_options]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#details-on-timeout-options
[query_options]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#query-options
[query_options2]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#additional-query-operations
[rw_preferences]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#changing-read-write-preferences
[indexes_opt]:https://docs.mongodb.com/ecosystem/tutorial/ruby-driver-tutorial/#creating-indexes

