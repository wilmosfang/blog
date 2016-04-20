---
layout: post
title:  Elasticsearch 映射
categories:  linux nosql elasticsearch  
wc:  563  1229 15243 
excerpt: elasticsearch mapping 的相关基础，查看mapping，查看索引mapping，查看字段mapping，类型报错，创建或指定mapping，更新mapping，字段冲突
comments: true
---



# 前言


**[Elasticsearch][elasticsearch]** 是一个 schemeless 的文档型数据库

ES 不像一般 RDBMS (mysql，postgresql) 一样，字段类型必须提前定义，但是不定义字段类型，并不代表没有字段类型，如果不提前人为指定，ES会在索引数据的时候自动判断以加上类型，一但加上，后面索引文档同字段的数据就默认遵循此类型，如果类型不同，就会报错

这有好处，一般使用场景下开发人员 **不用在意这些细节** 了，大部分场景中也基本够用，不会有大问题

但是有时还是会产生意外，比如同样对于 "123" ，以字符串索引和以整型索引是不一样的，忽略了有类型这件事，也并不会报错，大量数据导入后，检索的结果不符合预期时就很麻烦

 其次，一旦类型被自动构建好，有了数据后，相关字段的部分索引属性就不能变更了，比如之前一个字段默认是 **analyzed** 的，之后就再也没法改为 **not_analyzed**

所以根据具体场景，对ES的索引进行一定的 **scheme设计** ，以避免此类问题是很有必要的

这里简单分享一下 **Elasticsearch Mapping** 的相关操作和基础，详细可以参考 **[Get Mapping][get_mapping]** 和 **[Mapping][mapping]**


> **Tip:**当前的最新版本为 **Elasticsearch 2.3.1** ，我这里是拿 **2.1.1** 来演示

---


# 概要

* TOC
{:toc}


---

## 环境


系统版本和ES版本

{% highlight bash %}
[root@h102 st]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 st]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 st]# curl 'localhost:9200/_cat/nodes?h=v'
2.1.1 
[root@h102 st]#
{% endhighlight %}

---

## 查看mapping

首先创建一个索引，并加入一条数据

{% highlight bash %}
[root@h102 ~]# curl -XPUT 'localhost:9200/abc/test/1?pretty' -d '{"name":"joke","age":12}'
{
  "_index" : "abc",
  "_type" : "test",
  "_id" : "1",
  "_version" : 1,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
[root@h102 ~]# curl 'localhost:9200/abc/test/1?pretty'
{
  "_index" : "abc",
  "_type" : "test",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source":{"name":"joke","age":12}
}
[root@h102 ~]#
{% endhighlight %}

---

###  查看索引的mapping

{% highlight bash %}
[root@h102 ~]# curl 'localhost:9200/abc/_mapping?pretty'
{
  "abc" : {
    "mappings" : {
      "test" : {
        "properties" : {
          "age" : {
            "type" : "long"
          },
          "name" : {
            "type" : "string"
          }
        }
      }
    }
  }
}
[root@h102 ~]#
{% endhighlight %}

可以看到虽然我没有手动指定字段类型，但ES根据我指定的输入内容自动判断 **age** 类型为 **long** ， **name** 类型为 **string**

还是比较符合我的初衷的

查看API为 

{% highlight bash %}
host:port/{index}/_mapping/{type}
{% endhighlight %}

**`{index}`** 和 **`{type}`** 中可以使用逗号作为分割来指定一个名称列表，以同时指定多个想查看的对象 . 如果要代表所有的索引 可以在 **`{index}`** 中使用 **`_all`** 


> **Tip:** 可以直接查看索引的所有信息

{% highlight bash %}
[root@h102 ~]# curl 'localhost:9200/abc?pretty'
{
  "abc" : {
    "aliases" : { },
    "mappings" : {
      "test" : {
        "properties" : {
          "age" : {
            "type" : "long"
          },
          "name" : {
            "type" : "string"
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1460103933993",
        "uuid" : "VpdJUsGfRQqV0CUPY7PPjw",
        "number_of_replicas" : "1",
        "number_of_shards" : "5",
        "version" : {
          "created" : "2010199"
        }
      }
    },
    "warmers" : { }
  }
}
[root@h102 ~]# 
{% endhighlight %}

---

### 查看字段的mapping

可以限定只查看指定的字段类型，而不是所有字段

{% highlight bash %}
[root@h102 ~]# curl 'localhost:9200/abc/_mapping/test/field/age?pretty'
{
  "abc" : {
    "mappings" : {
      "test" : {
        "age" : {
          "full_name" : "age",
          "mapping" : {
            "age" : {
              "type" : "long"
            }
          }
        }
      }
    }
  }
}
[root@h102 ~]#
{% endhighlight %}

查看API为 

{% highlight bash %}
host:port/{index}/{type}/_mapping/field/{field}
{% endhighlight %}

**`{index}`** 、 **`{type}`** 和 **`{field}`** 中可以使用逗号作为分割来指定一个名称列表，以同时指定多个想查看的对象 . 如果要代表所有的索引 可以在 **`{index}`** 中使用 **`_all`**


### 补充特性


#### 匹配符

可以使用逗号作为分割来指定一个名称列表，同时也可以使用匹配符

{% highlight bash %}
[root@h102 ~]# curl 'localhost:9200/abc/_mapping/t*/field/a*?pretty'
{
  "abc" : {
    "mappings" : {
      "test" : {
        "age" : {
          "full_name" : "age",
          "mapping" : {
            "age" : {
              "type" : "long"
            }
          }
        }
      }
    }
  }
}
[root@h102 ~]#
{% endhighlight %}


#### 嵌套文档

这里的示例比较简单，如果是嵌套文档怎么办呢，在json中，这种情况可不少见

这时用 **`.`** 来进行指定

{% highlight bash %}
{
     "article": {
         "properties": {
             "id": { "type": "string" },
             "title":  { "type": "string"},
             "abstract": { "type": "string"},
             "author": {
                 "properties": {
                     "id": { "type": "string" },
                     "name": { "type": "string" }
                 }
             }
         }
     }
}
{% endhighlight %}

**author.id**  指代 **author** 中的 **id**

**author.name** 指代 **author** 中的 **name**

> **Tip:**  ES是使用Lucene 实现索引的，而Lucene并不懂多层对象，Lucene只是将它们看作一个个的扁平的 Key-Value 对， 为了让它可以处理多层对象，ES将嵌套的多层结构映射成了点分多层结构，user中的id和name 分别被当成 user.id 和 user.name 来处理

> **Note:** 在ES中列表是没有顺序的，类似于集合的概念


#### 显示所有默认属性

加上 **`include_defaults=true`**  就可以将隐藏的默认属性都显示出来

{% highlight bash %}
[root@h102 ~]# curl 'localhost:9200/abc/_mapping/test/field/age?pretty&include_defaults=true'
{
  "abc" : {
    "mappings" : {
      "test" : {
        "age" : {
          "full_name" : "age",
          "mapping" : {
            "age" : {
              "type" : "long",
              "boost" : 1.0,
              "index" : "not_analyzed",
              "store" : false,
              "doc_values" : true,
              "term_vector" : "no",
              "norms" : {
                "enabled" : false
              },
              "index_options" : "docs",
              "analyzer" : "_long/16",
              "search_analyzer" : "_long/max",
              "similarity" : "default",
              "fielddata" : { },
              "ignore_malformed" : false,
              "coerce" : true,
              "precision_step" : 16,
              "null_value" : null,
              "include_in_all" : false
            }
          }
        }
      }
    }
  }
}
[root@h102 ~]#
{% endhighlight %}


---

## 类型报错


我们尝试添加一条数据类型的记录到 **name** 中

{% highlight bash %}
[root@h102 ~]# curl -XPUT 'localhost:9200/abc/test/2?pretty' -d '{"name":12,"age":23}'
{
  "_index" : "abc",
  "_type" : "test",
  "_id" : "2",
  "_version" : 1,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
[root@h102 ~]# curl 'localhost:9200/abc/test/2?pretty'
{
  "_index" : "abc",
  "_type" : "test",
  "_id" : "2",
  "_version" : 1,
  "found" : true,
  "_source":{"name":12,"age":23}
}
[root@h102 ~]# 
[root@h102 ~]# curl 'localhost:9200/abc/_mapping/t*/field/n*?pretty'
{
  "abc" : {
    "mappings" : {
      "test" : {
        "name" : {
          "full_name" : "name",
          "mapping" : {
            "name" : {
              "type" : "string"
            }
          }
        }
      }
    }
  }
}
[root@h102 ~]#
{% endhighlight %}

成功了，说明数据类型被转化为了字符串类型

我们再尝试添加一条字符串类型的数据到 **age** 中

{% highlight bash %}
[root@h102 ~]# curl -XPUT 'localhost:9200/abc/test/3?pretty' -d '{"name":"testtype","age":"lili"}'
{
  "error" : {
    "root_cause" : [ {
      "type" : "mapper_parsing_exception",
      "reason" : "failed to parse [age]"
    } ],
    "type" : "mapper_parsing_exception",
    "reason" : "failed to parse [age]",
    "caused_by" : {
      "type" : "number_format_exception",
      "reason" : "For input string: \"lili\""
    }
  },
  "status" : 400
}
[root@h102 ~]# curl 'localhost:9200/abc/test/3?pretty'
{
  "_index" : "abc",
  "_type" : "test",
  "_id" : "3",
  "found" : false
}
[root@h102 ~]#
{% endhighlight %}

报类型不匹配的错误

我们尝试进行修改

{% highlight bash %}
[root@h102 ~]# curl -XPUT 'localhost:9200/abc/_mapping/test?update_all_types&pretty' -d  '{"properties" : {"age" : {"type" : "string"}}}'
{
  "error" : {
    "root_cause" : [ {
      "type" : "merge_mapping_exception",
      "reason" : "Merge failed with failures {[mapper [age] of different type, current_type [long], merged_type [string]]}"
    } ],
    "type" : "merge_mapping_exception",
    "reason" : "Merge failed with failures {[mapper [age] of different type, current_type [long], merged_type [string]]}"
  },
  "status" : 400
}
[root@h102 ~]# 
[root@h102 ~]# curl 'localhost:9200/abc/_mapping/test/field/age?pretty'
{
  "abc" : {
    "mappings" : {
      "test" : {
        "age" : {
          "full_name" : "age",
          "mapping" : {
            "age" : {
              "type" : "long"
            }
          }
        }
      }
    }
  }
}
[root@h102 ~]# 
{% endhighlight %}

结论是：**修改不了** 

> **Tip:** 但可以使用这个方法添加额外属性(就是原来没有指定的特性)，对于已经设定好的，就无能为力了，唯一的解决办法就是清空重来，如果此时数据量已经很大了，想想都疼

所以使用之初就应该进行一翻慎重考虑，必要的 **scheme设计** 可以有效解决这类问题


---

## 创建mapping


使用 **PUT mapping API** 可以在一个索引中创建符合指定mapping的类型(type，其实翻译过来反而怪怪的)，或者在一个现有的类型中添加指定mapping的字段

{% highlight bash %}
[root@h102 ~]# curl -XPUT 'localhost:9200/def?pretty'  -d '{"mappings": {"test": {"properties": {"userid": {"type": "integer"}}}}}'
{
  "acknowledged" : true
}
[root@h102 ~]# curl 'localhost:9200/def/_mapping?pretty'
{
  "def" : {
    "mappings" : {
      "test" : {
        "properties" : {
          "userid" : {
            "type" : "integer"
          }
        }
      }
    }
  }
}
[root@h102 ~]# curl -XPUT 'localhost:9200/def/_mapping/test?pretty'  -d '{"properties": {"city": {"type": "string"}}}'
{
  "acknowledged" : true
}
[root@h102 ~]# curl 'localhost:9200/def/_mapping?pretty'
{
  "def" : {
    "mappings" : {
      "test" : {
        "properties" : {
          "city" : {
            "type" : "string"
          },
          "userid" : {
            "type" : "integer"
          }
        }
      }
    }
  }
}
[root@h102 ~]# 
{% endhighlight %}

更为详细的mapping属性可以参考 **[Mapping][mapping]** ，字段类型可以参考 **[Field datatypes][mapping_type]**

**PUT mapping API**  

{% highlight bash %}
PUT /{index}/_mapping/{type}
{ body }
{% endhighlight %}


* **`{index}`** 可以是以逗号分割的多个索引或匹配符.
* **`{type}`** 是类型名.
* **`{body}`** 中包含了准备应用的映射内容.

---

## 更新mapping

总体而言，一般情况下现有字段的mapping是不能被更新的

但以下几种情况例外：


* 新属性可以被添加到对象的数据类型区域中
* 新的多字段可以被添加到现存字段中
* 文档值可以禁用，但不能启用
* **ignore_above** 参数可以被更新


---

## 字段冲突

在同一个索引中，即便是在不同类型(type)下，相同名字的字段必须拥有相同的mapping，因为在内部的实现中，不同的type如果有相同字段名其实就是在使用相同的字段(基础支持)

所以说索引才是字段类型的名称空间，而类型(type)并不是

在同一索引中，除非使用 **update_all_types** 参数，否则在不同的type中对一个名字相同的字段进行属性更新时会抛出异常，这个操作事实上会更新这一索引中不同type里所有叫这个名字的字段属性


我的看法是，既然目前ES对一个现成的字段更新不能很好地支持，那么就不要去尝试导入数据后更新这条路，保有的数据越多，就越头疼，直接在事先规划好，设计好，通过充分的考虑根据需求进行指定，就能免去这此问题

---



# 命令汇总


* **`uname -a`**
* **`cat /etc/issue`**
* **`curl 'localhost:9200/_cat/nodes?h=v'`**
* **`curl -XPUT 'localhost:9200/abc/test/1?pretty' -d '{"name":"joke","age":12}'`**
* **`curl 'localhost:9200/abc/test/1?pretty'`**
* **`curl 'localhost:9200/abc/_mapping?pretty'`**
* **`curl 'localhost:9200/abc?pretty'`**
* **`curl 'localhost:9200/abc/_mapping/test/field/age?pretty'`**
* **`curl 'localhost:9200/abc/_mapping/t*/field/a*?pretty'`**
* **`curl 'localhost:9200/abc/_mapping/test/field/age?pretty&include_defaults=true'`**
* **`curl -XPUT 'localhost:9200/abc/test/2?pretty' -d '{"name":12,"age":23}'`**
* **`curl 'localhost:9200/abc/test/2?pretty'`**
* **`curl 'localhost:9200/abc/_mapping/t*/field/n*?pretty'`**
* **`curl -XPUT 'localhost:9200/abc/test/3?pretty' -d '{"name":"testtype","age":"lili"}'`**
* **`curl 'localhost:9200/abc/test/3?pretty'`**
* **`curl -XPUT 'localhost:9200/abc/_mapping/test?update_all_types&pretty' -d  '{"properties" : {"age" : {"type" : "string"}}}'`**
* **`curl 'localhost:9200/abc/_mapping/test/field/age?pretty'`**
* **`curl -XPUT 'localhost:9200/def?pretty'  -d '{"mappings": {"test": {"properties": {"userid": {"type": "integer"}}}}}'`**
* **`curl 'localhost:9200/def/_mapping?pretty'`**
* **`curl -XPUT 'localhost:9200/def/_mapping/test?pretty'  -d '{"properties": {"city": {"type": "string"}}}'`**
* **`curl 'localhost:9200/def/_mapping?pretty'`**


---


[elasticsearch]:https://www.elastic.co/products/elasticsearch
[get_mapping]:https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html
[mapping]:https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html
[mapping_type]:https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html
