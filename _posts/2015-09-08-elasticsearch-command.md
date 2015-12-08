---
layout: post
title: Elasticsearch 常用命令
categories: linux elasticsearch nosql
excerpt: follow me
comments: true
---

---

#前言

**[Elasticsearch][elasticsearch]** 是一个使用得非常广泛的分布式全文检索数据库

主要有以下特性:

* Distributed, scalable, and highly available
* Real-time search and analytics capabilities
* Sophisticated RESTful API

更为详细的文档可以参考 **[Elasticsearch Reference][doc]**

下面对它常用的一些命令进行分享

---

#概要

* TOC
{:toc}


---

##确认相同java版本

{% highlight bash %}
java -version
{% endhighlight %}

> **Note:** 注意是一个 **-** 而不是两个 ; Elasticsearch 集群中的各节点要使用相同的java版本

---

##配置

详细配置可以参考 **[Configuration][configuration]**

###系统配置

####File Descriptors

建议配置文件句柄到 32k 或 64k

可以使用 **ulimit** 也可以在 **/etc/security/limits.conf** 中进行配置 

使用如下方法查看每个节点的最大文件句柄数，关注 **max_file_descriptors** 属性

{% highlight bash %}
curl localhost:9200/_nodes/process?pretty
{% endhighlight %}


---

####Virtual memory

ES会使用很多的内存映射来存储索引，默认情况下操作系统对 mmap 数量配置得很少，可能会导致内存溢出的异常

查看

{% highlight bash %}
sysctl  vm.max_map_count
{% endhighlight %}


修改

{% highlight bash %}
sysctl -w vm.max_map_count=262144
{% endhighlight %}

> **Tip:** 也可以在 **/etc/sysctl.conf** 中进行配置

---


####swap 

使用如下方法禁用 **swap**

{% highlight bash %}
sudo swapoff -a
{% endhighlight %}

> **Tip:** 也可以在 **/etc/fstab** 中进行配置，注释掉包含swap的那一行


---

####swappiness

查看

{% highlight bash %}
sysctl vm.swappiness 
{% endhighlight %}

修改

{% highlight bash %}
sysctl -w vm.swappiness=0
{% endhighlight %}

---

###ES配置

####mlockall

在 **config/elasticsearch.yml** 中配置

{% highlight bash %}
bootstrap.mlockall: true
{% endhighlight %}

使用下面命令进行查看，关注 **mlockall** 的值

{% highlight bash %}
curl http://localhost:9200/_nodes/process?pretty
{% endhighlight %}

> **Note:** 如果JVM尝试去分配多于可用内存的情况下，mlockall可能会导致异常退出

---

####cluster.name

在 **config/elasticsearch.yml** 中配置

这个是用来发现和自动加入集群的配置

{% highlight bash %}
cluster.name: abctest
{% endhighlight %}

> **Note:** 生产环境下一定要手动指定，否则默认加入 **elasticsearch** 集群其它新生成的节点在没配置的情况下很容易就加入了这个集群，产生意外

---

####node.name


这个用来指定节点名

{% highlight bash %}
node.name: "ES node1"
{% endhighlight %}

> **Note:**  生产环境中，尽量手动指定，默认情况下ES会从3000个名字中随机挑选一个用来命名这个节点，但是这显然不便于管理

---

####network.host

用来同时设定 **network.bind_host** 和 **network.publish_host**

{% highlight bash %}
network.host: 10.10.10.200
{% endhighlight %}

* **network.bind_host** :指定绑定IP，用来与客户端通信
* **network.publish_host** :指定发布IP，用来与节点同步通信

---

##iptables


**/etc/sysconfig/iptables** 的 **filter**中要加入以下几条

{% highlight bash %}
-A INPUT -d 224.2.2.4/32 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9300 -j ACCEPT
{% endhighlight %}

第一条开放组播，用于相互发现

第二条开放默认的服务端口，用于提供服务

第二条开放默认的同步端口，用于节点之间通讯




---


##启动服务

{% highlight bash %}
/data/elasticsearch-1.2.2/bin/elasticsearch -d -Xms512m -Xmx512m
{% endhighlight %}


---

##停止服务

---

##常用查询命令

###查询健康状态

{% highlight bash %}
[root@esvm03 ~]# curl localhost:9200/_cat/health?v
epoch      timestamp cluster status node.total node.data shards pri relo init unassign 
1441722453 22:27:33  escluster  green           3         3     40  20    0    0        0 
[root@esvm03 ~]# 
{% endhighlight %}

> **Tip:** **green** 代表正常  **red** 代表数据有丢失  **yellow** 代表数据完整但缺少副本集


---


###查询节点信息

{% highlight bash %}
[root@esvm03 ~]# curl localhost:9200/_cat/nodes?v
host          ip            heap.percent ram.percent load node.role master name      
esvm01        10.10.10.200           23          78 0.05 d         *      ES esvm01 
esvm02        10.10.10.201           20          39 0.01 d         m      ES esvm02 
esvm03 	      10.10.10.203           49          24 0.01 d         m      ES esvm03 
[root@esvm03 ~]# 
{% endhighlight %}

---

###查询分配信息

{% highlight bash %}
[root@esvm03 ~]# curl 'localhost:9200/_cat/allocation?v'
shards disk.used disk.avail disk.total disk.percent host          ip            node      
    13    20.9gb    274.2gb    295.2gb            7 esvm01        10.10.10.200 ES esvm01 
    14    15.6gb     82.7gb     98.4gb           15 esvm02        10.10.10.201 ES esvm02 
    13   100.5gb      1.6tb      1.6tb            5 esvm03 	  10.10.10.203 ES esvm03 
[root@esvm03 ~]# 
{% endhighlight %}


---

###查询索引信息

{% highlight bash %}
[root@esvm03 ~]# curl localhost:9200/_cat/indices?v
health index      pri rep docs.count docs.deleted store.size pri.store.size 
green  posts        5   1   14762492       309181        4gb            2gb 
green  topics       5   1      69015            0     15.3mb          7.6mb 
green  users        5   1      16810            6     11.2mb          5.6mb 
green  bet_orders   5   1      96720           17     93.2mb         46.6mb 
[root@esvm03 ~]# 
{% endhighlight %}

---

###查询节点负载

{% highlight bash %}
[root@esvm03 ~]# curl 'localhost:9200/_cat/fielddata?v'
id                     host          ip            node      total created_at 
sp7UBOgfSGKJaw_XPhyu-Q esvm02        10.10.10.201 ES esvm02 3.3mb      3.3mb 
LNX8Y35TTEarNaW8zZYVeQ esvm01        10.10.10.200 ES esvm01 4.4mb      4.4mb 
NeyBeQw1Qn6jsuJInDxJCQ esvm03 	     10.10.10.203 ES esvm03    0b         0b 
[root@esvm03 ~]# 
{% endhighlight %}

---

###查询master信息


{% highlight bash %}
[root@esvm03 ~]# curl 'localhost:9200/_cat/master?v'
id                     host   ip            node      
LNX8Y35TTEarNaW8zZYVeQ esvm01 10.10.10.200 ES esvm01 
[root@esvm03 ~]# 
{% endhighlight %}

---

###查询等待中的任务信息

{% highlight bash %}
[root@esvm03 ~]# curl 'localhost:9200/_cat/pending_tasks?v'
insertOrder timeInQueue priority source 
[root@esvm03 ~]# 
{% endhighlight %}


---

###查询插件信息

{% highlight bash %}
[root@esvm03 ~]# curl 'localhost:9200/_cat/plugins?v'
name      component      version type url 
ES esvm01 analysis-mmseg NA      j        
ES esvm02 analysis-mmseg NA      j        
ES esvm03 analysis-mmseg NA      j        
[root@esvm03 ~]# 
{% endhighlight %}

---

###查询恢复信息

{% highlight bash %}
[root@esvm03 ~]# curl 'localhost:9200/_cat/recovery?v'
index      shard time  type       stage source_host target_host   repository snapshot files files_percent bytes     bytes_percent 
users      0     405   relocation done  esvm01      esvm03        n/a        n/a      28    100.0%        1176005   100.0%        
users      0     52    replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        1176005   100.0%        
users      1     358   relocation done  esvm01      esvm03 	  n/a        n/a      39    100.0%        1194365   100.0%        
users      1     64    replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        1194365   100.0%        
users      2     671   replica    done  esvm01      esvm03 	  n/a        n/a      26    100.0%        1177678   100.0%        
users      2     6     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
users      3     385   replica    done  esvm01      esvm03 	  n/a        n/a      14    100.0%        1168462   100.0%        
users      3     29    gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
users      4     5     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
users      4     66    replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        1187941   100.0%        
topics     0     514   replica    done  esvm01      esvm03   	  n/a        n/a      32    100.0%        1611760   100.0%        
topics     0     5     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
topics     1     1096  relocation done  esvm01      esvm03 	  n/a        n/a      38    100.0%        1612483   100.0%        
topics     1     203   replica    done  esvm01      esvm02        n/a        n/a      17    100.0%        1459163   100.0%        
topics     2     725   relocation done  esvm01      esvm03 	  n/a        n/a      20    100.0%        1606817   100.0%        
topics     2     161   replica    done  esvm01      esvm02        n/a        n/a      14    100.0%        1466632   100.0%        
topics     3     4     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
topics     3     94    replica    done  esvm01      esvm02        n/a        n/a      7     100.0%        1469455   100.0%        
topics     4     4     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
topics     4     445   replica    done  esvm01      esvm02        n/a        n/a      7     100.0%        1480195   100.0%        
posts      0     25371 relocation done  esvm01      esvm03	  n/a        n/a      190   100.0%        422296109 100.0%        
posts      0     906   replica    done  esvm01      esvm02        n/a        n/a      56    100.0%        369011722 100.0%        
posts      1     32351 replica    done  esvm01      esvm03 	  n/a        n/a      202   100.0%        422354945 100.0%        
posts      1     853   gateway    done  esvm01      esvm01        n/a        n/a      60    100.0%        13380484  100.0%        
posts      2     24603 replica    done  esvm01      esvm03 	  n/a        n/a      178   100.0%        422154544 100.0%        
posts      2     723   gateway    done  esvm01      esvm01        n/a        n/a      48    100.0%        26706801  100.0%        
posts      3     905   gateway    done  esvm01      esvm01        n/a        n/a      52    100.0%        26862482  100.0%        
posts      3     1396  replica    done  esvm01      esvm02        n/a        n/a      57    100.0%        368898357 100.0%        
posts      4     236   gateway    done  esvm01      esvm01        n/a        n/a      64    100.0%        26563718  100.0%        
posts      4     1324  replica    done  esvm01      esvm02        n/a        n/a      51    100.0%        368808964 100.0%        
bet_orders 0     10210 replica    done  esvm01      esvm03 	  n/a        n/a      23    100.0%        4884128   100.0%        
bet_orders 0     14    gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
bet_orders 1     1446  relocation done  esvm01      esvm03 	  n/a        n/a      39    100.0%        4897417   100.0%        
bet_orders 1     80    replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        4438870   100.0%        
bet_orders 2     3378  relocation done  esvm01      esvm03 	  n/a        n/a      36    100.0%        4874906   100.0%        
bet_orders 2     126   replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        4441795   100.0%        
bet_orders 3     3     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
bet_orders 3     206   replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        4435120   100.0%        
bet_orders 4     6     gateway    done  esvm01      esvm01        n/a        n/a      0     0.0%          0         0.0%          
bet_orders 4     59    replica    done  esvm01      esvm02        n/a        n/a      1     100.0%        4430414   100.0%        
[root@esvm03 ~]# 
{% endhighlight %}

---

###查询线程池信息

{% highlight bash %}
[root@esvm03 ~]# curl localhost:9200/_cat/thread_pool?v
host          ip            bulk.active bulk.queue bulk.rejected index.active index.queue index.rejected search.active search.queue search.rejected 
esvm01        10.10.10.200           0          0             0            0           0              0             0            0               0 
esvm02        10.10.10.201           0          0             0            0           0              0             0            0               0 
esvm03        10.10.10.203           0          0             0            0           0              0             0            0               0 
[root@esvm03 ~]# 
{% endhighlight %}

---

###查看分片信息

{% highlight bash %}
[root@esvm03 ~]# curl localhost:9200/_cat/shards?v
index      shard prirep state      docs   store ip            node      
topics     4     p      STARTED   13804   1.5mb 10.10.10.200 ES esvm01 
topics     4     r      STARTED   13804   1.5mb 10.10.10.201 ES esvm02 
topics     0     r      STARTED   13804   1.5mb 10.10.10.203 ES esvm03 
topics     0     p      STARTED   13804   1.5mb 10.10.10.200 ES esvm01 
topics     3     p      STARTED   13803   1.5mb 10.10.10.200 ES esvm01 
topics     3     r      STARTED   13803   1.5mb 10.10.10.201 ES esvm02 
topics     1     p      STARTED   13802   1.5mb 10.10.10.203 ES esvm03 
topics     1     r      STARTED   13802   1.5mb 10.10.10.201 ES esvm02 
topics     2     p      STARTED   13804   1.5mb 10.10.10.203 ES esvm03 
topics     2     r      STARTED   13804   1.5mb 10.10.10.201 ES esvm02 
users      4     p      STARTED    3378   1.1mb 10.10.10.200 ES esvm01 
users      4     r      STARTED    3378   1.1mb 10.10.10.201 ES esvm02 
users      0     p      STARTED    3352   1.1mb 10.10.10.203 ES esvm03 
users      0     r      STARTED    3352   1.1mb 10.10.10.201 ES esvm02 
users      3     r      STARTED    3358   1.1mb 10.10.10.203 ES esvm03 
users      3     p      STARTED    3358   1.1mb 10.10.10.200 ES esvm01 
users      1     p      STARTED    3371   1.1mb 10.10.10.203 ES esvm03 
users      1     r      STARTED    3371   1.1mb 10.10.10.201 ES esvm02 
users      2     r      STARTED    3351   1.1mb 10.10.10.203 ES esvm03 
users      2     p      STARTED    3351   1.1mb 10.10.10.200 ES esvm01 
posts      4     p      STARTED 2953598 408.8mb 10.10.10.200 ES esvm01 
posts      4     r      STARTED 2953598 410.4mb 10.10.10.201 ES esvm02 
posts      0     p      STARTED 2952734 410.8mb 10.10.10.203 ES esvm03 
posts      0     r      STARTED 2952734 410.8mb 10.10.10.201 ES esvm02 
posts      3     p      STARTED 2951829 410.7mb 10.10.10.200 ES esvm01 
posts      3     r      STARTED 2951829 410.7mb 10.10.10.201 ES esvm02 
posts      1     r      STARTED 2953276 410.9mb 10.10.10.203 ES esvm03 
posts      1     p      STARTED 2953277   411mb 10.10.10.200 ES esvm01 
posts      2     r      STARTED 2951441 410.6mb 10.10.10.203 ES esvm03 
posts      2     p      STARTED 2951441 410.7mb 10.10.10.200 ES esvm01 
bet_orders 4     p      STARTED   19351   9.3mb 10.10.10.200 ES esvm01 
bet_orders 4     r      STARTED   19351   9.3mb 10.10.10.201 ES esvm02 
bet_orders 0     r      STARTED   19351   9.3mb 10.10.10.203 ES esvm03 
bet_orders 0     p      STARTED   19351   9.3mb 10.10.10.200 ES esvm01 
bet_orders 3     p      STARTED   19351   9.3mb 10.10.10.200 ES esvm01 
bet_orders 3     r      STARTED   19351   9.3mb 10.10.10.201 ES esvm02 
bet_orders 1     p      STARTED   19350   9.3mb 10.10.10.203 ES esvm03 
bet_orders 1     r      STARTED   19350   9.3mb 10.10.10.201 ES esvm02 
bet_orders 2     p      STARTED   19350   9.3mb 10.10.10.203 ES esvm03 
bet_orders 2     r      STARTED   19350   9.3mb 10.10.10.201 ES esvm02 
[root@esvm03 ~]# 
{% endhighlight %}


---

###查询段信息

{% highlight bash %}
curl 'http://localhost:9200/_cat/segments?v'
{% endhighlight %}


太长就不列出来了


---

###关掉一个节点

{% highlight bash %}
curl -XPOST 'http://localhost:9200/_cluster/nodes/_local/_shutdown'
curl -XPOST 'http://localhost:9200/_cluster/nodes/nodeId1,nodeId2/_shutdown'
curl -XPOST 'http://localhost:9200/_cluster/nodes/_master/_shutdown'
{% endhighlight %}

###关掉所有节点

{% highlight bash %}
curl -XPOST 'http://localhost:9200/_shutdown'
curl -XPOST 'http://localhost:9200/_cluster/nodes/_shutdown'
curl -XPOST 'http://localhost:9200/_cluster/nodes/_all/_shutdown'
{% endhighlight %}




---


[elasticsearch]:https://www.elastic.co/products/elasticsearch
[doc]:https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
[configuration]:https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration.html#settings
