---
layout: post
title:  Ruby 操作 Mysql
author: wilmosfang
tags:   mysql ruby
categories:   mysql ruby
wc: 493  1296 12349 
excerpt: mysql2 gem 的安装，创建用户，连接数据库，创建数据库，创建表，插入数据，捕获返回结果并显示，更新数据，删除数据，查询数据，兼容性
comments: true
---


# 前言


使用 Ruby 处理各种任务时难免会和数据库打交道，而 Mysql 又是一款应用极其广泛的数据库

**[RubyGems][rubygems]** 是 Ruby 的武器库，类似于 Perl 的 CPAN，各类封装好的处理逻辑应有尽有，我们可以充分利用这些成品包以减轻开发的工作量，其中的 **mysql2** 的 gem 就可以满足我们的需求

>A simple, fast Mysql library for Ruby, binding to libmysql

这里我分享一下使用 Ruby 来操作 Mysql 数据库的相关基础，详细可以参考 **[mysql2][mysql2]**


> **Tip:**   当前的最新版本为 **mysql2 0.4.4**

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


## 安装 mysql2

~~~
[root@h102 mysql]# gem source -l 
*** CURRENT SOURCES ***

https://gems.ruby-china.org
[root@h102 mysql]# gem install mysql2
Fetching: mysql2-0.4.4.gem (100%)
Building native extensions.  This could take a while...
Successfully installed mysql2-0.4.4
Parsing documentation for mysql2-0.4.4
Installing ri documentation for mysql2-0.4.4
Done installing documentation for mysql2 after 1 seconds
1 gem installed
[root@h102 mysql]# 
~~~

> **Tip:** 确认一下安装源，否则可能被墙，速度慢得没法忍

---

## 连接数据库

### 创建用户

先在目标数据库上创建一个用户，用于测试

> **Tip:** 主要用于功能测试，所以创建一个大权限用户，生产环境下不建议这样

~~~
[root@h105 ~]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.6.27-76.0 Percona Server (GPL), Release 76.0, Revision 5498987

Copyright (c) 2009-2015 Percona LLC and/or its affiliates
Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> GRANT ALL privileges ON *.* TO 'xxx'@'%' IDENTIFIED BY 'xxx';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> 
~~~

---

### 连接数据库

~~~
[root@h102 mysql]# irb
2.3.0 :001 > require 'mysql2'
 => true 
2.3.0 :002 > client = Mysql2::Client.new(:host => "192.168.100.105", :username => "xxx", :password => "xxx")
 => #<Mysql2::Client:0x00000001ba9180 @read_timeout=nil, @query_options={:as=>:hash, :async=>false, :cast_booleans=>false, :symbolize_keys=>false, :database_timezone=>:local, :application_timezone=>nil, :cache_rows=>true, :connect_flags=>2147525125, :cast=>true, :default_file=>nil, :default_group=>nil, :host=>"192.168.100.105", :username=>"xxx", :password=>"xxx"}> 
2.3.0 :003 > client.class
 => Mysql2::Client 
2.3.0 :004 > 
~~~

查看连接是否可用

~~~
2.3.0 :021 > client.ping
 => true 
2.3.0 :022 > 
~~~

查看客户端信息

~~~
2.3.0 :022 > client.info
 => {:id=>50627, :version=>"5.6.27-76.0", :header_version=>"5.6.27-76.0"} 
2.3.0 :023 > 
~~~

查看服务端信息

~~~
2.3.0 :023 > client.server_info
 => {:id=>50627, :version=>"5.6.27-76.0"} 
2.3.0 :024 > 
~~~

> **Tip:**   可以使用的连接选项如下

~~~
Mysql2::Client.new(
  :host,
  :username,
  :password,
  :port,
  :database,
  :socket = '/path/to/mysql.sock',
  :flags = REMEMBER_OPTIONS | LONG_PASSWORD | LONG_FLAG | TRANSACTIONS | PROTOCOL_41 | SECURE_CONNECTION | MULTI_STATEMENTS,
  :encoding = 'utf8',
  :read_timeout = seconds,
  :write_timeout = seconds,
  :connect_timeout = seconds,
  :reconnect = true/false,
  :local_infile = true/false,
  :secure_auth = true/false,
  :default_file = '/path/to/my.cfg',
  :default_group = 'my.cfg section',
  :init_command => sql
  )
~~~

在对安全要求更严格的环境下，可以使用 SSL 加密连接，前提是客户端和服务端都得编译对 SSL 的支持

~~~
Mysql2::Client.new(
  # ...options as above...,
  :sslkey => '/path/to/client-key.pem',
  :sslcert => '/path/to/client-cert.pem',
  :sslca => '/path/to/ca-cert.pem',
  :sslcapath => '/path/to/cacerts',
  :sslcipher => 'DHE-RSA-AES256-SHA',
  :sslverify => true,
  )
~~~





---


## 创建数据库

~~~
2.3.0 :024 > client.query("CREATE DATABASE `testxxx` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci")
 => nil 
2.3.0 :025 >
~~~

在本地进行检查

~~~
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| Syslog             |
| mysql              |
| performance_schema |
| test               |
| testxxx            |
+--------------------+
6 rows in set (0.03 sec)

mysql> show create database testxxx;
+----------+------------------------------------------------------------------------------------------------+
| Database | Create Database                                                                                |
+----------+------------------------------------------------------------------------------------------------+
| testxxx  | CREATE DATABASE `testxxx` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */ |
+----------+------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> 
~~~






---

## 创建表

~~~
2.3.0 :025 > client.query("CREATE table testxxx.test (id int(10),name char(20))")
 => nil 
2.3.0 :026 > 
~~~

在本地进行检查

~~~
mysql> use testxxx;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_testxxx |
+-------------------+
| test              |
+-------------------+
1 row in set (0.01 sec)

mysql> 
mysql> show create table test\G
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `id` int(10) DEFAULT NULL,
  `name` char(20) COLLATE utf8mb4_unicode_ci DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql>
~~~

---

## 插入数据

~~~
2.3.0 :040 > client.query("use testxxx")
 => nil 
2.3.0 :041 > (1..100).each do |i|
2.3.0 :042 >     client.query("insert into test(id,name) values(#{i},'hello#{i}')")
2.3.0 :043?>   end
 => 1..100 
2.3.0 :044 > 
~~~

本地检查

~~~
mysql> show tables;
+-------------------+
| Tables_in_testxxx |
+-------------------+
| test              |
+-------------------+
1 row in set (0.00 sec)

mysql> select * from test limit 10;
+------+---------+
| id   | name    |
+------+---------+
|    1 | hello1  |
|    2 | hello2  |
|    3 | hello3  |
|    4 | hello4  |
|    5 | hello5  |
|    6 | hello6  |
|    7 | hello7  |
|    8 | hello8  |
|    9 | hello9  |
|   10 | hello10 |
+------+---------+
10 rows in set (0.01 sec)

mysql>
~~~

---

##  捕获反馈结果并显示


~~~
2.3.0 :055 > r=client.query("show databases")
 => #<Mysql2::Result:0x00000001c3f810 @query_options={:as=>:hash, :async=>false, :cast_booleans=>false, :symbolize_keys=>false, :database_timezone=>:local, :application_timezone=>nil, :cache_rows=>true, :connect_flags=>2147525125, :cast=>true, :default_file=>nil, :default_group=>nil, :host=>"192.168.100.105", :username=>"xxx", :password=>"xxx"}> 
2.3.0 :056 > r.class
 => Mysql2::Result 
2.3.0 :057 > r.each do |x|
2.3.0 :058 >     puts x
2.3.0 :059?>   end
{"Database"=>"information_schema"}
{"Database"=>"Syslog"}
{"Database"=>"mysql"}
{"Database"=>"performance_schema"}
{"Database"=>"test"}
{"Database"=>"testxxx"}
 => [{"Database"=>"information_schema"}, {"Database"=>"Syslog"}, {"Database"=>"mysql"}, {"Database"=>"performance_schema"}, {"Database"=>"test"}, {"Database"=>"testxxx"}] 
2.3.0 :060 >
~~~

基实就是反馈了所有的数据库列表，类似于以下结果

~~~
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| Syslog             |
| mysql              |
| performance_schema |
| test               |
| testxxx            |
+--------------------+
6 rows in set (0.01 sec)

mysql>
~~~

---

## 更新数据

~~~
2.3.0 :066 > r=client.query("update test set name = 'minitest' where id=12")
 => nil 
2.3.0 :067 > r.class
 => NilClass 
2.3.0 :068 > r=client.query("select * from  test  where id=12")
 => #<Mysql2::Result:0x00000001b3b248 @query_options={:as=>:hash, :async=>false, :cast_booleans=>false, :symbolize_keys=>false, :database_timezone=>:local, :application_timezone=>nil, :cache_rows=>true, :connect_flags=>2147525125, :cast=>true, :default_file=>nil, :default_group=>nil, :host=>"192.168.100.105", :username=>"xxx", :password=>"xxx"}> 
2.3.0 :069 > r.each do |x|
2.3.0 :070 >     puts x 
2.3.0 :071?>   end
{"id"=>12, "name"=>"minitest"}
 => [{"id"=>12, "name"=>"minitest"}] 
2.3.0 :072 >
~~~

本地检查

~~~
mysql> select * from test where id=12;
+------+----------+
| id   | name     |
+------+----------+
|   12 | minitest |
+------+----------+
1 row in set (0.01 sec)

mysql> 
~~~

---

## 删除数据 


~~~
2.3.0 :073 > r=client.query("delete  from  test  where id=12")
 => nil 
2.3.0 :074 > r=client.query("select * from  test  where id=12")
 => #<Mysql2::Result:0x00000001960590 @query_options={:as=>:hash, :async=>false, :cast_booleans=>false, :symbolize_keys=>false, :database_timezone=>:local, :application_timezone=>nil, :cache_rows=>true, :connect_flags=>2147525125, :cast=>true, :default_file=>nil, :default_group=>nil, :host=>"192.168.100.105", :username=>"xxx", :password=>"xxx"}> 
2.3.0 :075 > r.each do |x|
2.3.0 :076 >     puts x 
2.3.0 :077?>   end
 => [] 
2.3.0 :078 >
~~~

---

## 查询数据


~~~
2.3.0 :082 > r=client.query("select * from  test  limit 10")
 => #<Mysql2::Result:0x00000001c72800 @query_options={:as=>:hash, :async=>false, :cast_booleans=>false, :symbolize_keys=>false, :database_timezone=>:local, :application_timezone=>nil, :cache_rows=>true, :connect_flags=>2147525125, :cast=>true, :default_file=>nil, :default_group=>nil, :host=>"192.168.100.105", :username=>"xxx", :password=>"xxx"}> 
2.3.0 :083 > r.each do |x|
2.3.0 :084 >     puts x 
2.3.0 :085?>   end
{"id"=>1, "name"=>"hello1"}
{"id"=>2, "name"=>"hello2"}
{"id"=>3, "name"=>"hello3"}
{"id"=>4, "name"=>"hello4"}
{"id"=>5, "name"=>"hello5"}
{"id"=>6, "name"=>"hello6"}
{"id"=>7, "name"=>"hello7"}
{"id"=>8, "name"=>"hello8"}
{"id"=>9, "name"=>"hello9"}
{"id"=>10, "name"=>"hello10"}
 => [{"id"=>1, "name"=>"hello1"}, {"id"=>2, "name"=>"hello2"}, {"id"=>3, "name"=>"hello3"}, {"id"=>4, "name"=>"hello4"}, {"id"=>5, "name"=>"hello5"}, {"id"=>6, "name"=>"hello6"}, {"id"=>7, "name"=>"hello7"}, {"id"=>8, "name"=>"hello8"}, {"id"=>9, "name"=>"hello9"}, {"id"=>10, "name"=>"hello10"}] 
2.3.0 :086 >
~~~

可以对这个结果集做些手脚，以更方便操作


~~~
2.3.0 :111 > r.class
 => Mysql2::Result 
2.3.0 :112 > r.to_a.class
 => Array 
2.3.0 :113 > r.to_a[1]
 => {"id"=>2, "name"=>"hello2"} 
2.3.0 :114 > r.to_a[1]["id"]
 => 2 
2.3.0 :115 > r.to_a[1]["name"]
 => "hello2" 
2.3.0 :116 > r.to_a[0]["name"]
 => "hello1" 
2.3.0 :117 > r.to_a[9]["id"]
 => 10 
2.3.0 :118 >
~~~

---


## 兼容性

这个 gem 已经在 Linux 和 Mac OS X 上以下版本的 Ruby 中通过测试

* Ruby MRI 1.8.7, 1.9.3, 2.0.0, 2.1.x, 2.2.x, 2.3.x
* Ruby Enterprise Edition (based on MRI 1.8.7)
* Rubinius 2.x, 3.x

这个 gem 已经通过以下版本的 MySQL 和 MariaDB 的测试

* MySQL 5.5, 5.6, 5.7
* MySQL Connector/C 6.0 and 6.1 (primarily on Windows)
* MariaDB 5.5, 10.0, 10.1


---

# 命令汇总


* **`ruby -v`**
* **`gem source -l`**
* **`gem install mysql2`**
* **`irb`**



---


[rubygems]:https://rubygems.org/
[mysql2]:https://rubygems.org/gems/mysql2

