---
layout: post
title:   ETL (Extract-Transform-Load) with Kiba
author: wilmosfang
tags:   ruby etl kiba
categories:   ruby
wc: 1091  2591 26770 
excerpt: ETL 项目的创建，解决依赖，库文件的创建，源的定义，打印中间结果，解析数值，转化日期，重命名列，数据有效性检查，定义数据去向
comments: true
---


# 前言

在构建数据仓库，进行数据分析，实现异构数据库之间数据转存的情境下会涉及到数据的 **[ETL(Extract-Transform-Load)][etl]** 

> **Tip:** 一般而言如下情况也可以使用 ETL 来解决：
>
> * 将遗留数据库中的数据迁移到新的数据库中
> * 自动处理数据以生成报表
> * 将多个系统中的所有数据或部分数据同步到一个中来
> * 将数据处理得易于搜索(导入到Elasticsearch 或 Solr 中)
> * 多个数据库中的数据进行聚合处理后将结果保存到一个数据一致的库中
> * 清理脏数据或无效数据 
> * 将数据进行位置分配后显示到地图应用中
> * 为用户实现一个数据导出的服务

ETL主要分三部：

* 数据抽取：(Data extraction)从各类数据源读取数据
* 数据处理：(Data transformation)对数据进行适当的加工处理以适应需求
* 数据装载：(Data loading)将结果保存到合适的地方

整个ETL的过程是像管道流一样进行处理的

>Since the data extraction takes time, it is common to execute the three phases in parallel. While the data is being extracted, another transformation process executes. It processes the already received data and prepares it for loading. As soon as there is some data ready to be loaded into the target, the data loading kicks off without waiting for the completion of the previous phases

Ruby 的 **[kiba][kiba]** gem 可以很容易地实现轻量级的 **ETL**

这里分享一下 **[kiba][kiba]** 的简单使用，详细可以参考 **[官方文档][kiba_doc]** 和 **[How to reformat CSV files with Kiba (in-depth, hands-on tutorial)][kiba_csv]**

> **Tip:**   目前此 gem 的最新版本为 **kiba 0.6.1** 

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

[root@h102 ~]# uname  -a
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 ~]# ruby -v 
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
[root@h102 ~]# gem --version
2.5.1
[root@h102 ~]#  
~~~

这里我们根据 **[How to reformat CSV files with Kiba (in-depth, hands-on tutorial)][kiba_csv]** 中的实验一步步来体验一下 Kiba 的简单使用方法 


---

## 源数据与目标数据

### CSV源数据

~~~
date_facture;montant_eur;numero_commande
7/3/2015;10,96;FA1986
7/3/2015;85,11;FA1987
8/3/2015;6,41;FA1988
~~~

### CSV目标数据

~~~
invoice_number,invoice_date,amount_eur
FA1986,2015-03-07,10.96
FA1987,2015-03-07,85.11
FA1988,2015-03-08,6.41
~~~

### 它们之间的差别

* 列使用 **;** 作为分割，要转化为 **,**
* 价格使用  **,** 作为分割，要转化为 **.**
* 列名要从 **date_facture** 转为 **invoice_date**，从 **montant_eur** 转为 **invoice_number**
* 数据的格式从 **7/3/2015** 转化为 **2015-03-07**

---


## 创建一个ETL项目

~~~
[root@h102 ~]# mkdir kiba
[root@h102 ~]# cd kiba
[root@h102 kiba]# ls
[root@h102 kiba]# 
~~~

---

### 创建一个 Gemfile 用来指定依赖

~~~
[root@h102 kiba]# vim Gemfile
[root@h102 kiba]# cat Gemfile 
source 'https://gems.ruby-china.org'

gem 'kiba', '~> 0.6.0'
gem 'awesome_print'
[root@h102 kiba]# 
~~~

这里的源我们使用 **`source 'https://gems.ruby-china.org'`** 因为 **`'https://rubygems.org'`** 会被墙

`gem 'kiba', '~> 0.6.0'`  是当前最新的 kiba 版本，项目中要使用到

`gem 'awesome_print'`  是一个很好用的打印工具

下面是它和普通打印的区别

~~~
[root@h102 ~]# irb
2.3.0 :001 > require 'awesome_print'
 => true 
2.3.0 :002 > p (1..8).to_a
[1, 2, 3, 4, 5, 6, 7, 8]
 => [1, 2, 3, 4, 5, 6, 7, 8] 
2.3.0 :003 > ap (1..8).to_a
[
    [0] 1,
    [1] 2,
    [2] 3,
    [3] 4,
    [4] 5,
    [5] 6,
    [6] 7,
    [7] 8
]
 => nil 
2.3.0 :004 >
~~~

它可以用很友好(便于人类阅读)地方式展示对象的结构和内容，更详细的用法可以参考 **[awesome_print][awesome_print]**


---

### 安装依赖并且测试

~~~
[root@h102 kiba]# bundle install 
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle
as root will break this application for all non-root users on this machine.
Fetching gem metadata from https://gems.ruby-china.org/..
Fetching version metadata from https://gems.ruby-china.org/.
Resolving dependencies...
Installing awesome_print 1.7.0
Installing kiba 0.6.1
Using bundler 1.12.5
Bundle complete! 2 Gemfile dependencies, 3 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
[root@h102 kiba]# echo "puts 'Hello from Kiba'" > convert-csv.etl
[root@h102 kiba]# bundle exec kiba convert-csv.etl 
Hello from Kiba
[root@h102 kiba]#
~~~

> **Note:** 这里必须确保 **bundler** gem 已经安装好，否则没法使用 bundle 命令

---

## 创建一个库文件

我们采用尽量模块化的思想，将可重用的代码集中放到一个库文件中(common.rb)以便于维护，核心逻辑放到主文件中(convert-csv.etl)


### 加入对 CSV 源的定义

~~~
[root@h102 kiba]# vim common.rb
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end
[root@h102 kiba]#
~~~

> **Note:** 随着项目的累积与扩展，会产生各种各样的源定义，为了便于维护也可以将这些源定义分离出来成为单独的文件

---

### 对 CSV 源进行测试

~~~
[root@h102 kiba]# vim commandes.csv
[root@h102 kiba]# cat commandes.csv 
date_facture;montant_eur;numero_commande
7/3/2015;10,96;FA1986
7/3/2015;85,11;FA1987
8/3/2015;6,41;FA1988
[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol
[root@h102 kiba]# bundle exec kiba convert-csv.etl 
[root@h102 kiba]# 
[root@h102 kiba]# 
~~~


**require_relative** 和 **require** 所起的功能一样，只是引用文件的位置为自身的相对位置而与 **`$LAOD_PATH ($:)`**  路径无关

从对 CSV 源的定义我们知道，**`'commandes.csv'`** 被初始化给了 **`@file`** ，而  **`col_sep: ';', headers: true, header_converters: :symbol`**  被初始化给了 **`@options`**

参数的意思就是：使用 CSV 打开 'commandes.csv' 文件， 这个文件是以 ';' 作为字段分割符的，有头信息，将头信息转化为 ':symbol' 的形式 

> **Tip:** **CSV** 是标准库，其使用方法与相关细节可以参考 **[CSV gem][csv]**

最后的执行结果并没有报加载异常，表明代码可以正常执行

---

### 打印结果 

我们可以将中间结果打印出来，以方便调试

在 common.rb 中定义一个 show_me 方法，加入一段显示逻辑，然后运行kiba项目

~~~
[root@h102 kiba]# vim common.rb 
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end


require 'awesome_print'

def show_me
  transform do |row|
    ap row
    row # always return the row to keep it in the pipeline
  end
end

[root@h102 kiba]# ls
commandes.csv  common.rb  convert-csv.etl  Gemfile  Gemfile.lock
[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol

show_me
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
       :date_facture => "7/3/2015",
        :montant_eur => "10,96",
    :numero_commande => "FA1986"
}
{
       :date_facture => "7/3/2015",
        :montant_eur => "85,11",
    :numero_commande => "FA1987"
}
{
       :date_facture => "8/3/2015",
        :montant_eur => "6,41",
    :numero_commande => "FA1988"
}
[root@h102 kiba]# 
~~~

现在已经可以成功解析使用 **`';'`**  分割的字段，并且以hash的形式打印出来

---

## 解析数值

加入解析数值的类 **ParseFrenchFloat** ，并定义处理逻辑


~~~
[root@h102 kiba]# vim common.rb 
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end


require 'awesome_print'

def show_me
  transform do |row|
    ap row
    row # always return the row to keep it in the pipeline
  end
end


class ParseFrenchFloat
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Float(row[@from].gsub(',', '.'))
    row
  end
end

[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol

# Parse the numbers
transform ParseFrenchFloat, from: :montant_eur, to: :amount_eur

# show details of row contents
show_me
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
       :date_facture => "7/3/2015",
        :montant_eur => "10,96",
    :numero_commande => "FA1986",
         :amount_eur => 10.96
}
{
       :date_facture => "7/3/2015",
        :montant_eur => "85,11",
    :numero_commande => "FA1987",
         :amount_eur => 85.11
}
{
       :date_facture => "8/3/2015",
        :montant_eur => "6,41",
    :numero_commande => "FA1988",
         :amount_eur => 6.41
}
[root@h102 kiba]# 
~~~


其中最主要的就是 **`row[@to] = Float(row[@from].gsub(',', '.'))`**

它的意思就是对 **from** 字段(或 Key) 指向的值进行处理，将其中的 **`,`** 替换为 **`.`**，然后使用 **Float** 转化为浮点数，然后赋予给 **to** 字段，这个字段是新字段，在 row hash 中添加入新的 KV 对

运行的结果正如预期


---

## 转化日期


加入解析数值的类 **ParseFrenchDate** ，并定义处理逻辑

~~~
[root@h102 kiba]# vim common.rb 
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end


require 'awesome_print'

def show_me
  transform do |row|
    ap row
    row # always return the row to keep it in the pipeline
  end
end


class ParseFrenchFloat
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Float(row[@from].gsub(',', '.'))
    row
  end
end


class ParseFrenchDate
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Date.strptime(row[@from], '%d/%m/%Y').to_s
    row
  end
end
[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol

# Parse the numbers
transform ParseFrenchFloat, from: :montant_eur, to: :amount_eur

#Reformat the dates
transform ParseFrenchDate, from: :date_facture, to: :invoice_date

# show details of row contents
show_me
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
       :date_facture => "7/3/2015",
        :montant_eur => "10,96",
    :numero_commande => "FA1986",
         :amount_eur => 10.96,
       :invoice_date => "2015-03-07"
}
{
       :date_facture => "7/3/2015",
        :montant_eur => "85,11",
    :numero_commande => "FA1987",
         :amount_eur => 85.11,
       :invoice_date => "2015-03-07"
}
{
       :date_facture => "8/3/2015",
        :montant_eur => "6,41",
    :numero_commande => "FA1988",
         :amount_eur => 6.41,
       :invoice_date => "2015-03-08"
}
[root@h102 kiba]# 
~~~

其中最主要的就是 **`row[@to] = Date.strptime(row[@from], '%d/%m/%Y').to_s`**

它的意思就是对 **from** 字段(或 Key) 指向的值进行处理，将其中的值以 **`'%d/%m/%Y'`** 模式解析成日期 ，然后转化为字符串格式，然后赋予给 **to** 字段，这个字段是新字段，在 row hash 中添加入新的 KV 对

运行的结果正如预期

---


## 对列进行重命名

加入对列进行重命名的类 **RenameField** ，并定义处理逻辑

~~~
[root@h102 kiba]# vim common.rb 
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end


require 'awesome_print'

def show_me
  transform do |row|
    ap row
    row # always return the row to keep it in the pipeline
  end
end


class ParseFrenchFloat
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Float(row[@from].gsub(',', '.'))
    row
  end
end


class ParseFrenchDate
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Date.strptime(row[@from], '%d/%m/%Y').to_s
    row
  end
end


class RenameField
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = row.delete(@from)
    row
  end
end
[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol

# Parse the numbers
transform ParseFrenchFloat, from: :montant_eur, to: :amount_eur

#Reformat the dates
transform ParseFrenchDate, from: :date_facture, to: :invoice_date

#Rename the remaining column
transform RenameField, from: :numero_commande, to: :invoice_number

# show details of row contents
show_me
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
      :date_facture => "7/3/2015",
       :montant_eur => "10,96",
        :amount_eur => 10.96,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1986"
}
{
      :date_facture => "7/3/2015",
       :montant_eur => "85,11",
        :amount_eur => 85.11,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1987"
}
{
      :date_facture => "8/3/2015",
       :montant_eur => "6,41",
        :amount_eur => 6.41,
      :invoice_date => "2015-03-08",
    :invoice_number => "FA1988"
}
[root@h102 kiba]# 
~~~

其中最主要的就是 **`row[@to] = row.delete(@from)`**

它的意思就是删除 **from** 字段(或 Key) ，将其中的值赋予给 **to** 字段，这个字段是新字段，在 row hash 中添加入新的 KV 对

> **Tip:** 删除 Hash 中的一个 Key 时会反馈其值 

~~~
2.3.0 :016 > row = {:a => "b", :c => "d"}
 => {:a=>"b", :c=>"d"} 
2.3.0 :017 > ap row
{
    :a => "b",
    :c => "d"
}
 => nil 
2.3.0 :018 > tmp = row.delete(:c)
 => "d" 
2.3.0 :019 > ap tmp
"d"
 => nil 
2.3.0 :020 > ap row
{
    :a => "b"
}
 => nil 
2.3.0 :021 >
~~~

最后运行的结果正如预期

---

## 数据有效性检查

为了防止源数据的格式变动或异常造成ETL任务的失败，我们可以对数据进行提前检查，以预防此类问题的发生

这里实现一个简单的空值检测，如果发现空值，就抛出定义的异常信息


这里需要加入一个新的 gem 到 Gemfile 中，并且进行安装

~~~
[root@h102 kiba]# vim Gemfile
[root@h102 kiba]# cat Gemfile
source 'https://gems.ruby-china.org'

gem 'kiba', '~> 0.6.0'
gem 'awesome_print'
gem "facets", require: false
[root@h102 kiba]# bundle install 
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle
as root will break this application for all non-root users on this machine.
Fetching gem metadata from https://gems.ruby-china.org/..
Fetching version metadata from https://gems.ruby-china.org/.
Resolving dependencies...
Using awesome_print 1.7.0
Installing facets 3.1.0
Using kiba 0.6.1
Using bundler 1.12.5
Bundle complete! 3 Gemfile dependencies, 4 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
[root@h102 kiba]# 
~~~

加入对列进行检查的类 **VerifyFieldsPresence** ，并定义处理逻辑 

~~~
[root@h102 kiba]# vim common.rb 
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end


require 'awesome_print'

def show_me
  transform do |row|
    ap row
    row # always return the row to keep it in the pipeline
  end
end


class ParseFrenchFloat
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Float(row[@from].gsub(',', '.'))
    row
  end
end


class ParseFrenchDate
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Date.strptime(row[@from], '%d/%m/%Y').to_s
    row
  end
end


class RenameField
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = row.delete(@from)
    row
  end
end


require 'facets/kernel/blank'

class VerifyFieldsPresence
  def initialize(expected_fields)
    @expected_fields = expected_fields
  end
  
  def process(row)
    @expected_fields.each do |field|
      if row[field].blank?
        raise "Row lacks value for field #{field} - #{row.inspect}"
      end
    end
    row
  end
end
[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol

#verify the source columns are there and provide a non-blank value
transform VerifyFieldsPresence, [:date_facture, :montant_eur, :numero_commande]

# Parse the numbers
transform ParseFrenchFloat, from: :montant_eur, to: :amount_eur

#Reformat the dates
transform ParseFrenchDate, from: :date_facture, to: :invoice_date

#Rename the remaining column
transform RenameField, from: :numero_commande, to: :invoice_number

# show details of row contents
show_me
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
      :date_facture => "7/3/2015",
       :montant_eur => "10,96",
        :amount_eur => 10.96,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1986"
}
{
      :date_facture => "7/3/2015",
       :montant_eur => "85,11",
        :amount_eur => 85.11,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1987"
}
{
      :date_facture => "8/3/2015",
       :montant_eur => "6,41",
        :amount_eur => 6.41,
      :invoice_date => "2015-03-08",
    :invoice_number => "FA1988"
}
[root@h102 kiba]# 
~~~

因为我们的数据都符合预期，所以没有报出异常，现在故意修改一下源数据

将第二条数据的价格删除，然后再运行ETL脚本

~~~
[root@h102 kiba]# vim commandes.csv 
[root@h102 kiba]# cat commandes.csv 
date_facture;montant_eur;numero_commande
7/3/2015;10,96;FA1986
7/3/2015;;FA1987
8/3/2015;6,41;FA1988
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
      :date_facture => "7/3/2015",
       :montant_eur => "10,96",
        :amount_eur => 10.96,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1986"
}
/root/kiba/common.rb:76:in `block in process': Row lacks value for field montant_eur - {:date_facture=>"7/3/2015", :montant_eur=>nil, :numero_commande=>"FA1987"} (RuntimeError)
	from /root/kiba/common.rb:74:in `each'
	from /root/kiba/common.rb:74:in `process'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:35:in `block (3 levels) in process_rows'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:34:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:34:in `block (2 levels) in process_rows'
	from /root/kiba/common.rb:11:in `block in each'
	from /usr/local/rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/csv.rb:1748:in `each'
	from /usr/local/rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/csv.rb:1131:in `block in foreach'
	from /usr/local/rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/csv.rb:1282:in `open'
	from /usr/local/rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/csv.rb:1130:in `foreach'
	from /root/kiba/common.rb:10:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:33:in `block in process_rows'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:32:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:32:in `process_rows'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/runner.rb:13:in `run'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/lib/kiba/cli.rb:13:in `run'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/kiba-0.6.1/bin/kiba:5:in `<top (required)>'
	from /usr/local/rvm/gems/ruby-2.3.0/bin/kiba:23:in `load'
	from /usr/local/rvm/gems/ruby-2.3.0/bin/kiba:23:in `<main>'
	from /usr/local/rvm/gems/ruby-2.3.0/bin/ruby_executable_hooks:15:in `eval'
	from /usr/local/rvm/gems/ruby-2.3.0/bin/ruby_executable_hooks:15:in `<main>'
[root@h102 kiba]# 
~~~

第一条得到成功的处理，但是第二就信息就报错了

报错 **` Row lacks value for field montant_eur - {:date_facture=>"7/3/2015", :montant_eur=>nil, :numero_commande=>"FA1987"} (RuntimeError)`**

运行符合预期

---

## 定义数据去向

~~~
[root@h102 kiba]# vim commandes.csv 
[root@h102 kiba]# cat commandes.csv 
date_facture;montant_eur;numero_commande
7/3/2015;10,96;FA1986
7/3/2015;23,12;FA1987
8/3/2015;6,41;FA1988
[root@h102 kiba]# vim common.rb 
[root@h102 kiba]# cat common.rb 
require 'csv'

class CsvSource
  def initialize(file, options)
    @file = file
    @options = options
  end
  
  def each
    CSV.foreach(@file, @options) do |row|
      yield row.to_hash
    end
  end
end


require 'awesome_print'

def show_me
  transform do |row|
    ap row
    row # always return the row to keep it in the pipeline
  end
end


class ParseFrenchFloat
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Float(row[@from].gsub(',', '.'))
    row
  end
end


class ParseFrenchDate
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = Date.strptime(row[@from], '%d/%m/%Y').to_s
    row
  end
end


class RenameField
  def initialize(from:, to:)
    @from = from
    @to = to
  end
  
  def process(row)
    row[@to] = row.delete(@from)
    row
  end
end


require 'facets/kernel/blank'

class VerifyFieldsPresence
  def initialize(expected_fields)
    @expected_fields = expected_fields
  end
  
  def process(row)
    @expected_fields.each do |field|
      if row[field].blank?
        raise "Row lacks value for field #{field} - #{row.inspect}"
      end
    end
    row
  end
end


class CsvDestination
  def initialize(file, output_fields)
    @csv = CSV.open(file, 'w')
    @output_fields = output_fields

    @csv << @output_fields
  end
  
  def write(row)
    verify_row!(row)
    @csv << row.values_at(*@output_fields) #*
  end

  def verify_row!(row)
    missing_fields = @output_fields - [row.keys & @output_fields].flatten

    if missing_fields.size > 0
      raise "Row lacks required field(s) #{missing_fields}\n#{row}"
    end
  end

  def close
    @csv.close
  end
end

[root@h102 kiba]# vim convert-csv.etl 
[root@h102 kiba]# cat convert-csv.etl 
require_relative 'common'

# read from source CSV file
source CsvSource, 'commandes.csv', col_sep: ';', headers: true, header_converters: :symbol

#verify the source columns are there and provide a non-blank value
transform VerifyFieldsPresence, [:date_facture, :montant_eur, :numero_commande]

# Parse the numbers
transform ParseFrenchFloat, from: :montant_eur, to: :amount_eur

#Reformat the dates
transform ParseFrenchDate, from: :date_facture, to: :invoice_date

#Rename the remaining column
transform RenameField, from: :numero_commande, to: :invoice_number

#define CSV destination
output_fields = [:invoice_number, :invoice_date, :amount_eur]
destination CsvDestination, 'orders.csv', output_fields

# show details of row contents
show_me
[root@h102 kiba]# bundle exec kiba convert-csv.etl
{
      :date_facture => "7/3/2015",
       :montant_eur => "10,96",
        :amount_eur => 10.96,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1986"
}
{
      :date_facture => "7/3/2015",
       :montant_eur => "23,12",
        :amount_eur => 23.12,
      :invoice_date => "2015-03-07",
    :invoice_number => "FA1987"
}
{
      :date_facture => "8/3/2015",
       :montant_eur => "6,41",
        :amount_eur => 6.41,
      :invoice_date => "2015-03-08",
    :invoice_number => "FA1988"
}
[root@h102 kiba]# ls
commandes.csv  common.rb  convert-csv.etl  Gemfile  Gemfile.lock  orders.csv
[root@h102 kiba]# cat orders.csv 
invoice_number,invoice_date,amount_eur
FA1986,2015-03-07,10.96
FA1987,2015-03-07,23.12
FA1988,2015-03-08,6.41
[root@h102 kiba]# 
~~~

到此，一个简单的基于 CSV 源和目标的 ETL 就实现了，下次有机会再分享一下，如何使用 Mysql 或 Elasticsearch 或 Mongodb 来实现相互之间的 ETL

上面的实例中已经涵盖了 **source、transform、process、destination** 的定义和应用，其实还有 **pre_process 和 post_process** 可以定义，它们分别是在 ETL 处理第一行数据之前执行的代码块和 ETL 处理完成最后一行数据之后执行的代码块，详细可以参考 **[官方文档][kiba_doc]**，有机会再单独分享




---

# 命令汇总

* **`gem --version`**
* **`mkdir kiba`**
* **`cat Gemfile`**
* **`irb`**
* **`bundle install`**
* **`echo "puts 'Hello from Kiba'" > convert-csv.etl`**
* **`bundle exec kiba convert-csv.etl`**
* **`vim common.rb`**
* **`vim commandes.csv`**
* **`vim convert-csv.etl`**
* **`bundle exec kiba convert-csv.etl`**
* **`vim Gemfile`**
* **`vim commandes.csv`**
* **`cat orders.csv`**



---


[etl]:https://en.wikipedia.org/wiki/Extract,_transform,_load
[kiba]:https://rubygems.org/gems/kiba
[kiba_doc]:http://www.rubydoc.info/gems/kiba
[kiba_csv]:http://thibautbarrere.com/2015/06/04/how-to-reformat-csv-files-with-kiba/
[awesome_print]:https://rubygems.org/gems/awesome_print
[csv]:http://ruby-doc.org/stdlib-2.3.1/libdoc/csv/rdoc/CSV.html
