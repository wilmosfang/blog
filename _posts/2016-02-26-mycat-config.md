---
layout: post
title:  Mycat 配置
author: wilmosfang
categories:  linux mysql mycat
wc: 474   714 12584 
excerpt:  mycat的概念和配置解析 
comments: true
---



# 前言

**[Mycat][mycat]** 是一个数据库分库分表中间件

**MyCAT** 是作为通用代理设计的，后端是以 **Mysql协议** 和 **JDBC** 的方式连接数据库，可以支持 **Oracle、DB2、SQL Server 、 mongodb、mysql**


![mycat_middle.jpg](/images/mycat/mycat_middle.jpg)


这里简单分享一下 **[Mycat][mycat]** 中的概念及配置的相关基础 ，详细内容可以参考 **[官方文档][mycat_doc]** 、 **[Mycat-Server][mycat_git]**  和 **[Get Start][mycat_wiki]**


> **Tip:** 当前的最新版本为 **Mycat server 1.5 GA** 

---


# 概要

* TOC
{:toc}



---

## 概念


### 数据库中间件



![db_midware.png](/images/mycat/db_midware.png)


Mycat 其实只是一个数据中间件，或数据库代理

> **Tip:** 所有难搞定的事情都可以通过中间件有效处理，中间件能有效解耦并专注于特定领域问题，**LVM、LVS、MQ** 都是这个思路（房屋中介，银行也都是这个思路）

所以Mycat没有存储引擎，本身并不存储数据，只是起到了请求分析，拆解，路由与结果聚合的作用，为前端应用提供统一接口，Mycat 与后端的数据库集群有机组合才一起构成一个分布式数据库系统



---

### 逻辑库(schema)


![mycat_schema.png](/images/mycat/mycat_schema.png)


类似于LVM中VG的概念(VG由一个或多个PV构成)，逻辑库是由一个或多个后端数据库构成的，展示给应用的是一个单一视图，是分布式数据库在逻辑上的一个抽象

---

### 逻辑表(table)

* **逻辑表**

与数据库中表相对应的，分布式数据表在逻辑上的一个抽象

* **分片表**

数据表切分后的一个部分(原表的一个真子集)

* **非分片表**

没有分片的表，就是非分片表

* **ER表**

保留了实体关系特性的表，就是ER表

关系型数据库是基于实体关系模型的相关理论来构建的数据库，表与表间有依赖关系，通过表分组(Table Group) 让有依赖的表在同一实例库中从而避免了数据Join不会跨库操作

* **全局表**

全局表是所有分片上都有一份完整拷贝的表

字典表或符合字典特性的表可以被设置为全局表

有以下特点的表，被称作字典表：

* 变动不频繁
* 数据量总体变化不大
* 数据规模不大(很少超过十万条记录)
* 会与其它表发生关联

这类表可以通过冗余来解决join问题，也就是所有的分片都放上一份数据的拷贝来避免跨分片联查


> **Tip:** 数据冗余和表分组是解决跨分片数据join的好思路，也是数据切分规划的重要规则


![mycat_table.png](/images/mycat/mycat_table.png)


---

### 分片节点(dataNode)

每个表分片所在的数据库就是分片节点

---

### 节点主机(dataHost)

分片节点所在的服务器就是节点主机


> **Tip:** 尽量将读写压力高的分片节点均衡放在不同的节点主机上，以避免单节点主机并发数限制

---

### 分片规则(rule)

分片规则就是切分数据的规则

---

### 全局序列号(sequence)

保证数据全局唯一性标识的外部机制就是全局序列号

> **Tip:** 单机(单实例)环境下的主键约束在分布式环境中将失效，因此得通过外部机制以全局视角来保证数据唯一性


---

### 多租户

多租户技术也叫多重租凭技术，就是在确保用户间数据隔离的前提下实现在多用户环境中共用相同系统或程序等软硬件资源的一种软件架构技术

![mycat_multitenant.png](/images/mycat/mycat_multitenant.png)

> **Tip:** **web** 中广范使用的 **VirtualHost** 就是一种典型的多租户技术，多租户技术是为了更充分的使用到现有资源，同时不失权限控制的一种技术

* 独立数据库
* 共享数据库，隔离数据架构
* 共享数据库，共享数据架构


隔离级别越来越低，共享程度越来越高，均摊成本越来越低

### 整体关系


![mycat_arch.jpg](/images/mycat/mycat_arch.jpg)


---

## 配置


Mycat 的大部分配置都是以 **XML** 的格式设定的

~~~
[root@h102 mycat]# ll conf/schema.xml 
-rwxrwxrwx 1 root root 4129 Feb 17 10:30 conf/schema.xml
[root@h102 mycat]# ll conf/rule.xml 
-rwxrwxrwx 1 root root 4510 Feb 17 10:30 conf/rule.xml
[root@h102 mycat]# ll conf/server.xml 
-rwxrwxrwx 1 root root 2507 Feb 17 10:30 conf/server.xml
[root@h102 mycat]# ll conf/wrapper.conf 
-rwxrwxrwx 1 root root 4318 Feb 24 21:20 conf/wrapper.conf
[root@h102 mycat]#  
~~~


Conf | Comment
-------- | ---
conf/wrapper.conf| JVM运行环境配置
conf/server.xml| 用来定义系统相关变量
conf/schema.xml| 用来定义逻辑库，表，分片节点
conf/rule.xml| 用来定义分片规则



---


### wrapper.conf

我们使用这个文件来配置JVM的相关运行参数

~~~
[root@h102 conf]# cat wrapper.conf | egrep "(Xm|MaxDirectMemorySize)"
#wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G
wrapper.java.additional.5=-XX:MaxDirectMemorySize=256m
#wrapper.java.additional.10=-Xmx4G
wrapper.java.additional.10=-Xmx512m
#wrapper.java.additional.11=-Xms1G
wrapper.java.additional.11=-Xms128m
[root@h102 conf]# 
~~~

以上配置是常用的对JVM内存的控制

---


### server.xml

XML的格式就是各类标签

#### 注释

这个标签用来框定注释范围

~~~
<!--
...
...
-->
~~~


#### mycat:server

这个标签用来框定服务配置范围

~~~
<mycat:server xmlns:mycat="http://org.opencloudb/">
</mycat:server>
~~~


#### system

这个标签用来框定系统配置范围，用来保存几乎所有mycat需要的系统配置信息(其在代码内直接的映射类为 **SystemConfig** )

~~~
<system>
</system>
~~~

#### property


用来设定服务的具体参数

~~~
<property name="defaultSqlParser">druidparser</property>
<property name="processors">2</property>
<property name="serverPort">8066</property> 
<property name="managerPort">9066</property>
<property name="bindIp">0.0.0.0</property> 
~~~


#### user

用来设定一个租户，与相关权限

这里设定了一个 **cc** 的租户，密码为 **cc** , 可以访问 **cctest** 的数据库(schema)

~~~
<user name="cc">
	<property name="password">cc</property>
	<property name="schemas">cctest</property>
</user>
~~~

---

### schema.xml


#### mycat:schema

这个标签用来框定shema的配置范围

~~~
<mycat:schema xmlns:mycat="http://org.opencloudb/">
</mycat:schema>
~~~

#### schema


用来配置一个逻辑库(schema)

这里配置了一个名叫 **cctest** 的逻辑库，不检查SQL，默认limit为100(sql中不添加limit的情况下，mycat会隐式添加，以避免返回太多结果)，其中包含两个逻辑表，**catworld** 和 **catworld4** ，**catworld** 有三个分片，使用 **mod-long** 的规则，**catworld4** 有四个分片，使用 **mod4-long** 的分片规则

~~~
<schema name="cctest" checkSQLschema="false" sqlMaxLimit="100">
	       <table name="catworld"  dataNode="sd1,sd2,sd3"  rule="mod-long" />
	       <table name="catworld4"  dataNode="sd1,sd2,sd3,sd4"  rule="mod4-long" />
</schema>
~~~


Attribute | Comment
-------- | ---
checkSQLschema| 隐式删除schema前缀
sqlMaxLimit| 隐式添加limit语句


#### table 

Attribute | Comment
-------- | ---
dataNode| 指定所属数据节点
rule| 指定分片规则


#### dataNode


这个标签用来定义数据节点(数据分片存放的地方)

~~~
<dataNode name="sd1" dataHost="h101" database="my1" />
<dataNode name="sd2" dataHost="h101" database="my2" />
<dataNode name="sd3" dataHost="h101" database="my3" />
<dataNode name="sd4" dataHost="h202" database="my4" />
~~~


Attribute | Comment
-------- | ---
dataHost| 指定所属数据库实例
database| 指定数据库实例上的实际数据库名(一定要和真实库一样的名字，这个不是被标签定义的，是要提前在实例中手动创建的)



#### dataHost 

节点主机的相关配置

~~~
<dataHost name="h101" maxCon="100" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<writeHost host="h101M1" url="192.168.100.101:3306" user="root" password="mysql">
		<!-- can have multi read hosts -->
		</writeHost>
</dataHost>
<dataHost name="h202" maxCon="100" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<writeHost host="h202M1" url="192.168.100.202:3306" user="root" password="mysql">
		<!-- can have multi read hosts -->
		</writeHost>
</dataHost>
~~~

Attribute | Comment
-------- | ---
maxCon| 一个读写实例链接池的最大连接数
minCon| 一个读写实例链接池的最小连接数，初始化连接池的大小
balance|负载均衡类型：**0** 代表不开启读写分离机制，只使用writeHost; **1** 代表readHost与writeHost分担读请求; **2** 代表随机分配读请求和1类似; **3** 代表只由readHost来承担读请求
writeType|负载均衡类型：**0** 代表发到第一个writeHost,挂了后切到还生存的第二个writeHost,重新启动后以切换后的为准，也就是不漂回；**1** 代表写操作随机发送到writeHost,这样不安全；
dbType|后端数据库类型
dbDriver|mysql系可以使用native,其它系列得使用JDBC
switchType|切换类型：**-1** 代表不切换，**1** 代表自动切换， **2** 代表基于主从同步状态决定是否切换
slaveThreshold|slave读的安全边界，如果**`Seconds_Behind_Master`** 大于这个值，这台slave会被临时剔除，以免被读


#### heartbeat

里面包含一个语句，用语句执行成功与否来判定数据库的可用性

#### writeHost/readHost

Attribute | Comment
-------- | ---
host| 一个主机标识，便于区分，不必和真实主机名一致
url|后端实例连接地址
user|连接账户
password|连接密码


> **Tip:** 用户名和密码要提前在各实例中赋予相应连接和操作权限


---

### rule.xml

此配置用来定义分片规则

#### mycat:rule

框定rule的配置范围

~~~
<mycat:rule xmlns:mycat="http://org.opencloudb/">
</mycat:rule>
~~~

#### tableRule

定义一个分片规则

定义了一个 **mod-long** 的分片规则，对 **id** 列进行分片，使用 **mod-long** 算法；定义了一个 **mod4-long** 的分片规则，对 **id** 列进行分片，使用 **mod4-long** 算法

~~~
<tableRule name="mod-long">
                <rule>
                        <columns>id</columns>
                        <algorithm>mod-long</algorithm>
                </rule>
</tableRule>
<tableRule name="mod4-long">
                <rule>
                        <columns>id</columns>
                        <algorithm>mod4-long</algorithm>
                </rule>
</tableRule>
~~~


#### function


~~~
<function name="mod-long" class="org.opencloudb.route.function.PartitionByMod">
                <!-- how many data nodes -->
                <property name="count">3</property>
</function>
<function name="mod4-long" class="org.opencloudb.route.function.PartitionByMod">
                <!-- how many data nodes -->
                <property name="count">4</property>
</function>
~~~

Attribute | Comment
-------- | ---
class| 使用的类
property|通过 count=3/4 来指定分片数(指定模数)




## 注意

XML中定义的标签有顺序，如果不按照顺序进行配置，会报错

比如 **schema.xml** 中的顺序为

* 1.定义 **schema**
* 2.定义 **dataNode**
* 3.定义 **dataHost**

如果不按顺序，会无法启动mycat，并且 **mycat.log** 中会报错


这里只对一套简单基础的配置进行了分析，只涵盖了一小部分，还未涵盖到的，可以参考 **[官方文档][mycat_doc]**

---

# 命令汇总


* **`ll conf/schema.xml`**
* **`ll conf/rule.xml`**
* **`ll conf/server.xml`**
* **`ll conf/wrapper.conf`**
* **`cat wrapper.conf | egrep "(Xm|MaxDirectMemorySize)"`**


---





[mycat]:http://www.mycat.org.cn/
[mycat_doc]:http://www.mycat.org.cn/document/mycat1.5.2.pdf
[mycat_git]:https://github.com/MyCATApache/Mycat-Server
[mycat_wiki]:https://github.com/MyCATApache/Mycat-Server/wiki

