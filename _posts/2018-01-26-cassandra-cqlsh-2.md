---
layout: post
title: "Cassandra Cqlsh 2"
author:  wilmosfang
date: 2018-01-26 13:04:39
image: '/assets/img/'
excerpt: 'Cqlsh 的简单用法'
main-class: cassandra
color: '#90bd50'
tags:
 - cassandra
 - cqlsh
categories:
 - cassandra
twitter_text: 'the usage of Cqlsh'
introduction: 'client of cassandra'
---



# 前言


**[Cassandra][cassandra]** 是一款开源分布式数据库软件，可以提供高容错，高性能，高可用，高弹性，可线性扩展的特性

在 **CAP** 理论中，它很好地实践了 **AP** 牺牲了 **C**, 它是一个最终一致性数据库

**[Cqlsh][cqlsh]** 是 **[Cassandra][cassandra]** 的客户端

接着上一篇，下面分享一下 **[Cqlsh][cqlsh]** 的简单使用方法

参考 **[Cassandra Tools][cassandra_tools]**

> **Tip:** 当前版本 **Cassandra 3.11.1** 和 **cqlsh 5.0.1**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: f13f6477c2db43f9a7febda448d0468e
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
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84011sec preferred_lft 84011sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 依赖

* Python 2.7

>For using cqlsh, the latest version of Python 2.7

~~~
[root@much ~]# python -V
Python 2.7.5
[root@much ~]#
~~~

## 路径

~~~
[root@much ~]# rpm -ql cassandra | grep cqlsh | grep bin
/usr/bin/cqlsh
/usr/bin/cqlsh.py
[root@much ~]#
~~~

在安装 **cassandra** 的过程中就已经自带了客户端


## 连接

~~~
[root@much ~]# cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh>
~~~

默认会连接本地的 9042 号端口

## 常用命令

### TRACING

用于跟踪一条语句的执行过程

>Enables or disables tracing for queries. When tracing is enabled, once a query completes, a trace of the events during the query will be printed.


~~~
cqlsh> TRACING on
Now Tracing is enabled
cqlsh> SELECT listen_address from system.local ;

 listen_address
----------------
      127.0.0.1

(1 rows)

Tracing session: 254857b0-0299-11e8-9160-dd31d8caf67e

 activity                                                                                                                       | timestamp                  | source    | source_elapsed | client
--------------------------------------------------------------------------------------------------------------------------------+----------------------------+-----------+----------------+-----------
                                                                                                             Execute CQL3 query | 2018-01-26 21:02:21.356000 | 127.0.0.1 |              0 | 127.0.0.1
                                                Parsing SELECT listen_address from system.local ; [Native-Transport-Requests-1] | 2018-01-26 21:02:21.360000 | 127.0.0.1 |           3809 | 127.0.0.1
                                                                              Preparing statement [Native-Transport-Requests-1] | 2018-01-26 21:02:21.360000 | 127.0.0.1 |           4215 | 127.0.0.1
                                                                        Computing ranges to query [Native-Transport-Requests-1] | 2018-01-26 21:02:21.360000 | 127.0.0.1 |           4541 | 127.0.0.1
 Submitting range requests on 1 ranges with a concurrency of 1 (30.07059 rows per range expected) [Native-Transport-Requests-1] | 2018-01-26 21:02:21.361000 | 127.0.0.1 |           4779 | 127.0.0.1
                                                            Submitted 1 concurrent range requests [Native-Transport-Requests-1] | 2018-01-26 21:02:21.361000 | 127.0.0.1 |           4935 | 127.0.0.1
                  Executing seq scan across 2 sstables for (min(-9223372036854775808), min(-9223372036854775808)) [ReadStage-2] | 2018-01-26 21:02:21.363000 | 127.0.0.1 |           7084 | 127.0.0.1
                                                                                Read 1 live and 0 tombstone cells [ReadStage-2] | 2018-01-26 21:02:21.364000 | 127.0.0.1 |           8537 | 127.0.0.1
                                                                                                               Request complete | 2018-01-26 21:02:21.366302 | 127.0.0.1 |          10302 | 127.0.0.1


cqlsh> TRACING off
Disabled Tracing.
cqlsh> SELECT listen_address from system.local ;

 listen_address
----------------
      127.0.0.1

(1 rows)
cqlsh>
~~~



### PAGING

用来配置会话里的页显条数

不加参数时打开默认为每页 100 条，也可以在打开时直接指定一页的条目数

>Enables paging, disables paging, or sets the page size for read queries. When paging is enabled, only one page of data will be fetched at a time and a prompt will appear to fetch the next page. Generally, it’s a good idea to leave paging enabled in an interactive session to avoid fetching and printing large amounts of data at once.

~~~
cqlsh> PAGING on
Now Query paging is enabled
Page size: 100
cqlsh> PAGING off
Disabled Query paging.
cqlsh> PAGING 5
Page size: 5
cqlsh> select * from system_schema.columns ;

 keyspace_name | table_name                     | column_name | clustering_order | column_name_bytes        | kind          | position | type
---------------+--------------------------------+-------------+------------------+--------------------------+---------------+----------+-----------
   system_auth | resource_role_permissons_index |    resource |             none |       0x7265736f75726365 | partition_key |        0 |      text
   system_auth | resource_role_permissons_index |        role |              asc |               0x726f6c65 |    clustering |        0 |      text
   system_auth |                   role_members |      member |              asc |           0x6d656d626572 |    clustering |        0 |      text
   system_auth |                   role_members |        role |             none |               0x726f6c65 | partition_key |        0 |      text
   system_auth |               role_permissions | permissions |             none | 0x7065726d697373696f6e73 |       regular |       -1 | set<text>

---MORE---
 keyspace_name | table_name       | column_name  | clustering_order | column_name_bytes          | kind          | position | type
---------------+------------------+--------------+------------------+----------------------------+---------------+----------+-----------
   system_auth | role_permissions |     resource |              asc |         0x7265736f75726365 |    clustering |        0 |      text
   system_auth | role_permissions |         role |             none |                 0x726f6c65 | partition_key |        0 |      text
   system_auth |            roles |    can_login |             none |       0x63616e5f6c6f67696e |       regular |       -1 |   boolean
   system_auth |            roles | is_superuser |             none | 0x69735f737570657275736572 |       regular |       -1 |   boolean
   system_auth |            roles |    member_of |             none |       0x6d656d6265725f6f66 |       regular |       -1 | set<text>

---MORE---
 keyspace_name | table_name | column_name    | clustering_order | column_name_bytes              | kind          | position | type
---------------+------------+----------------+------------------+--------------------------------+---------------+----------+--------------------
   system_auth |      roles |           role |             none |                     0x726f6c65 | partition_key |        0 |               text
   system_auth |      roles |    salted_hash |             none |       0x73616c7465645f68617368 |       regular |       -1 |               text
 system_schema | aggregates | aggregate_name |              asc | 0x6167677265676174655f6e616d65 |    clustering |        0 |               text
 system_schema | aggregates | argument_types |              asc | 0x617267756d656e745f7479706573 |    clustering |        1 | frozen<list<text>>
 system_schema | aggregates |     final_func |             none |         0x66696e616c5f66756e63 |       regular |       -1 |               text

---MORE---
cqlsh>
~~~



### EXPAND

将默认为行显转换为列显

在行显而列数太多的时候，使用这种方式，展示会更友好

>Enables or disables vertical printing of rows. Enabling EXPAND is useful when many columns are fetched, or the contents of a single column are large.

~~~
cqlsh> EXPAND
Expanded output is currently disabled. Use EXPAND ON to enable.
cqlsh> EXPAND on
Now Expanded output is enabled
cqlsh> select * from system_schema.columns limit 1;

@ Row 1
-------------------+--------------------------------
 keyspace_name     | system_auth
 table_name        | resource_role_permissons_index
 column_name       | resource
 clustering_order  | none
 column_name_bytes | 0x7265736f75726365
 kind              | partition_key
 position          | 0
 type              | text

(1 rows)
cqlsh> select * from system_schema.columns limit 2;

@ Row 1
-------------------+--------------------------------
 keyspace_name     | system_auth
 table_name        | resource_role_permissons_index
 column_name       | resource
 clustering_order  | none
 column_name_bytes | 0x7265736f75726365
 kind              | partition_key
 position          | 0
 type              | text

@ Row 2
-------------------+--------------------------------
 keyspace_name     | system_auth
 table_name        | resource_role_permissons_index
 column_name       | role
 clustering_order  | asc
 column_name_bytes | 0x726f6c65
 kind              | clustering
 position          | 0
 type              | text

(2 rows)
cqlsh> EXPAND off
Disabled Expanded output.
cqlsh> select * from system_schema.columns limit 1;

 keyspace_name | table_name                     | column_name | clustering_order | column_name_bytes  | kind          | position | type
---------------+--------------------------------+-------------+------------------+--------------------+---------------+----------+------
   system_auth | resource_role_permissons_index |    resource |             none | 0x7265736f75726365 | partition_key |        0 | text

(1 rows)
cqlsh>
~~~

### CLEAR

用来清屏

~~~
cqlsh> help clear

        CLEAR/CLS [cqlsh only]

        Clears the console.

cqlsh> help cls

        CLEAR/CLS [cqlsh only]

        Clears the console.

cqlsh> cls































cqlsh>
~~~

### DESCRIBE

显示指定对象的描述信息

>Prints a description (typically a series of DDL statements) of a schema element or the cluster. This is useful for dumping all or portions of the schema.

~~~
cqlsh> help describe

        DESCRIBE [cqlsh only]

        (DESC may be used as a shorthand.)

          Outputs information about the connected Cassandra cluster, or about
          the data objects stored in the cluster. Use in one of the following ways:

        DESCRIBE KEYSPACES

          Output the names of all keyspaces.

        DESCRIBE KEYSPACE [<keyspacename>]

          Output CQL commands that could be used to recreate the given keyspace,
          and the objects in it (such as tables, types, functions, etc.).
          In some cases, as the CQL interface matures, there will be some metadata
          about a keyspace that is not representable with CQL. That metadata will not be shown.

          The '<keyspacename>' argument may be omitted, in which case the current
          keyspace will be described.

        DESCRIBE TABLES

          Output the names of all tables in the current keyspace, or in all
          keyspaces if there is no current keyspace.

        DESCRIBE TABLE [<keyspace>.]<tablename>

          Output CQL commands that could be used to recreate the given table.
          In some cases, as above, there may be table metadata which is not
          representable and which will not be shown.

        DESCRIBE INDEX <indexname>

          Output the CQL command that could be used to recreate the given index.
          In some cases, there may be index metadata which is not representable
          and which will not be shown.

        DESCRIBE MATERIALIZED VIEW <viewname>

          Output the CQL command that could be used to recreate the given materialized view.
          In some cases, there may be materialized view metadata which is not representable
          and which will not be shown.

        DESCRIBE CLUSTER

          Output information about the connected Cassandra cluster, such as the
          cluster name, and the partitioner and snitch in use. When you are
          connected to a non-system keyspace, also shows endpoint-range
          ownership information for the Cassandra ring.

        DESCRIBE [FULL] SCHEMA

          Output CQL commands that could be used to recreate the entire (non-system) schema.
          Works as though "DESCRIBE KEYSPACE k" was invoked for each non-system keyspace
          k. Use DESCRIBE FULL SCHEMA to include the system keyspaces.

        DESCRIBE TYPES

          Output the names of all user-defined-types in the current keyspace, or in all
          keyspaces if there is no current keyspace.

        DESCRIBE TYPE [<keyspace>.]<type>

          Output the CQL command that could be used to recreate the given user-defined-type.

        DESCRIBE FUNCTIONS

          Output the names of all user-defined-functions in the current keyspace, or in all
          keyspaces if there is no current keyspace.

        DESCRIBE FUNCTION [<keyspace>.]<function>

          Output the CQL command that could be used to recreate the given user-defined-function.

        DESCRIBE AGGREGATES

          Output the names of all user-defined-aggregates in the current keyspace, or in all
          keyspaces if there is no current keyspace.

        DESCRIBE AGGREGATE [<keyspace>.]<aggregate>

          Output the CQL command that could be used to recreate the given user-defined-aggregate.

        DESCRIBE <objname>

          Output CQL commands that could be used to recreate the entire object schema,
          where object can be either a keyspace or a table or an index or a materialized
          view (in this order).

cqlsh> DESCRIBE cluster

Cluster: Test Cluster
Partitioner: Murmur3Partitioner

cqlsh> DESCRIBE schema

cqlsh> DESCRIBE keyspaces

system_traces  system_schema  system_auth  system  system_distributed

cqlsh> DESCRIBE tables

Keyspace system_traces
----------------------
events  sessions

Keyspace system_schema
----------------------
tables     triggers    views    keyspaces  dropped_columns
functions  aggregates  indexes  types      columns        

Keyspace system_auth
--------------------
resource_role_permissons_index  role_permissions  role_members  roles

Keyspace system
---------------
available_ranges          peers               batchlog        transferred_ranges
batches                   compaction_history  size_estimates  hints             
prepared_statements       sstable_activity    built_views   
"IndexInfo"               peer_events         range_xfers   
views_builds_in_progress  paxos               local         

Keyspace system_distributed
---------------------------
repair_history  view_build_status  parent_repair_history

cqlsh>
~~~


### EXIT

退出 cqlsh

~~~
cqlsh> help exit

        EXIT/QUIT [cqlsh only]

        Exits cqlsh.

cqlsh> exit
[root@much ~]#
~~~



---

# 总结

CQL 的这些命令还是比较简单好用的，目前为止还没涉及到操作数据

* TOC
{:toc}


---

[cassandra]:http://cassandra.apache.org/
[cqlsh]:http://cassandra.apache.org/doc/latest/tools/cqlsh.html
[cassandra_tools]:http://cassandra.apache.org/doc/latest/tools/index.html
