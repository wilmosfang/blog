---
layout: post
title:  MongoDB 基础
author: wilmosfang
tags:   nosql mongodb
categories:   nosql mongodb
wc: 558  1861 22968 
excerpt: mongodb 软件仓库，mongodb 安装，手动创建 mongodb 实例，mongodb 服务启动，连接，停止
comments: true
---


# 前言

**[MongoDB][mongodb]** 是一个开源的文档型数据库

>MongoDB is an open-source, document database designed for ease of development and scaling

目前的数据库主要分为两大阵营： SQL  和  NoSQL

NoSQL 的出现是为了应对 SQL 在互联网环境中一些力不从心的场景，但是并不能完全取代 SQL 的地位，各有所长，都在取长补短，协作配合，共同应对海量数据管理带来的挑战

> **Tip:** NoSQL 的类型可以参考之前写的一篇博文 **[Neo4j 基础][neo4j]** 的 **[前言][qianyan]** 部分

**[MongoDB][mongodb]** 作为 NoSQL 阵营里文档型存储的最典型代表，虽然其使用内存的方式经常遭人诟病，早期版本的库级锁让人头疼，但是当前的发展势头依然火热，良好的支持，全面的文档和活跃的社区是很多开源项目的典范，技术上的缺陷相信在未来都会获得逐步地改善

MongoDB在生产实践中有很广泛的使用，这里分享一下MongoDB的相关基础，详细可以参考 **[官方文档][mongodb_doc]**


> **Tip:**   当前的最新版本为 **MongoDB 3.2**

---


# 概要

* TOC
{:toc}



---


## 环境


~~~
[root@h105 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h105 ~]# uname -a 
Linux h105 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h105 ~]# 
~~~


---

## 安装 mongodb

在 **[MongoDB Download Center][mongodb_dl]** 中可以选择合适的的版本进行下载


由于我的平台是Centos 6.6 ，我是参考 **[Install MongoDB Community Edition on Red Hat Enterprise or CentOS Linux][mongodb_install]** 的过程来进行安装

之所以使用 yum 而非源码，是图省事儿

目前 mongodb 兼容的 OS 可以参考 **[支持平台][supported_platforms]**

---

### 创建软件仓库

~~~
[root@h105 ~]# cd /etc/yum.repos.d/
[root@h105 yum.repos.d]# ls
Base.repo         CentOS-Debuginfo.repo  CentOS-Media.repo  epel.repo
CentOS-Base.repo  CentOS-fasttrack.repo  CentOS-Vault.repo  epel-testing.repo
[root@h105 yum.repos.d]# vim /etc/yum.repos.d/mongodb-org-3.2.repo
[root@h105 yum.repos.d]# cat /etc/yum.repos.d/mongodb-org-3.2.repo
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc
[root@h105 yum.repos.d]#
~~~

---


### 安装软件


~~~
[root@h105 yum.repos.d]# yum clean all 
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository base is listed more than once in the configuration
Cleaning repos: base epel extras mongodb-org-3.2 updates
Cleaning up Everything
Cleaning up list of fastest mirrors
[root@h105 yum.repos.d]# yum install -y mongodb-org
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Determining fastest mirrors
epel/metalink                                                                                                  | 3.3 kB
 * epel: mirror.pregi.net
 * extras: mirrors.aliyun.com
 * updates: mirrors.skyshe.cn
base                                                                                                           | 4.0 kB
base/primary_db                                                                                                | 4.5 MB
epel                                                                                                           | 4.3 kB
epel/primary_db                                                                                                | 5.9 MB
extras                                                                                                         | 3.4 kB
extras/primary_db                                                                                              |  37 kB
mongodb-org-3.2                                                                                                | 2.5 kB
mongodb-org-3.2/primary_db                                                                                     |  33 kB
updates                                                                                                        | 3.4 kB
updates/primary_db                                                                                             | 5.2 MB
Resolving Dependencies
--> Running transaction check
---> Package mongodb-org.x86_64 0:3.2.6-1.el6 will be installed
--> Processing Dependency: mongodb-org-tools = 3.2.6 for package: mongodb-org-3.2.6-1.el6.x86_64
--> Processing Dependency: mongodb-org-shell = 3.2.6 for package: mongodb-org-3.2.6-1.el6.x86_64
--> Processing Dependency: mongodb-org-server = 3.2.6 for package: mongodb-org-3.2.6-1.el6.x86_64
--> Processing Dependency: mongodb-org-mongos = 3.2.6 for package: mongodb-org-3.2.6-1.el6.x86_64
--> Running transaction check
---> Package mongodb-org-mongos.x86_64 0:3.2.6-1.el6 will be installed
---> Package mongodb-org-server.x86_64 0:3.2.6-1.el6 will be installed
---> Package mongodb-org-shell.x86_64 0:3.2.6-1.el6 will be installed
---> Package mongodb-org-tools.x86_64 0:3.2.6-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================================================
 Package                              Arch                     Version                        Repository               
=======================================================================================================================
Installing:
 mongodb-org                          x86_64                   3.2.6-1.el6                    mongodb-org-3.2          
Installing for dependencies:
 mongodb-org-mongos                   x86_64                   3.2.6-1.el6                    mongodb-org-3.2          
 mongodb-org-server                   x86_64                   3.2.6-1.el6                    mongodb-org-3.2          
 mongodb-org-shell                    x86_64                   3.2.6-1.el6                    mongodb-org-3.2          
 mongodb-org-tools                    x86_64                   3.2.6-1.el6                    mongodb-org-3.2          

Transaction Summary
=======================================================================================================================
Install       5 Package(s)

Total download size: 65 M
Installed size: 206 M
Downloading Packages:
(1/5): mongodb-org-3.2.6-1.el6.x86_64.rpm                                                                      | 5.8 kB
(2/5): mongodb-org-mongos-3.2.6-1.el6.x86_64.rpm                                                               | 6.0 MB
(3/5): mongodb-org-server-3.2.6-1.el6.x86_64.rpm                                                               |  13 MB
(4/5): mongodb-org-shell-3.2.6-1.el6.x86_64.rpm                                                                | 7.3 MB
(5/5): mongodb-org-tools-3.2.6-1.el6.x86_64.rpm                                                                |  38 MB
-----------------------------------------------------------------------------------------------------------------------
Total                                                                                                 499 kB/s |  65 MB
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA1 Signature, key ID ea312927: NOKEY
Retrieving key from https://www.mongodb.org/static/pgp/server-3.2.asc
Importing GPG key 0xEA312927:
 Userid: "MongoDB 3.2 Release Signing Key <packaging@mongodb.com>"
 From  : https://www.mongodb.org/static/pgp/server-3.2.asc
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : mongodb-org-server-3.2.6-1.el6.x86_64                                                                   
  Installing : mongodb-org-mongos-3.2.6-1.el6.x86_64                                                                   
  Installing : mongodb-org-tools-3.2.6-1.el6.x86_64                                                                    
  Installing : mongodb-org-shell-3.2.6-1.el6.x86_64                                                                    
  Installing : mongodb-org-3.2.6-1.el6.x86_64                                                                          
  Verifying  : mongodb-org-shell-3.2.6-1.el6.x86_64                                                                    
  Verifying  : mongodb-org-tools-3.2.6-1.el6.x86_64                                                                    
  Verifying  : mongodb-org-mongos-3.2.6-1.el6.x86_64                                                                   
  Verifying  : mongodb-org-server-3.2.6-1.el6.x86_64                                                                   
  Verifying  : mongodb-org-3.2.6-1.el6.x86_64                                                                          

Installed:
  mongodb-org.x86_64 0:3.2.6-1.el6                                                                                     

Dependency Installed:
  mongodb-org-mongos.x86_64 0:3.2.6-1.el6     mongodb-org-server.x86_64 0:3.2.6-1.el6     mongodb-org-shell.x86_64 0:3.
  mongodb-org-tools.x86_64 0:3.2.6-1.el6     

Complete!
[root@h105 yum.repos.d]#
[root@h105 yum.repos.d]# mongo --version
MongoDB shell version: 3.2.6
[root@h105 yum.repos.d]# 
~~~

> **Tip:**  **mongodb-org** 其实本身不包含任何逻辑代码，只是利用 rpm 的 spec 文件在指定依赖，这样只用安装一个 **mongodb-org** ，yum 就会自动去将其依赖的 **mongodb-org-mongos、mongodb-org-server、mongodb-org-shell、mongodb-org-tools** 都给装上，简化了安装过程


> **Tip:** 要查看这个包的依赖关系可以使用 **`yum deplist mongodb-org`**


> **Note:**  SELinux 可能妨碍 mongodb 的正常工作，要关掉

~~~
[root@h105 ~]# getenforce 
Disabled
[root@h105 ~]# grep -i '^SELINUX' /etc/sysconfig/selinux 
SELINUX=disabled
SELINUXTYPE=targeted 
[root@h105 ~]# 
~~~


---

## 创建 mongo 实例


安装好 **mongodb-org-server** 后，系统中就已经创建好了如下目录或文件


~~~
[root@h105 mongo]# rpm -ql mongodb-org-server
/etc/init.d/mongod
/etc/mongod.conf
/etc/sysconfig/mongod
/usr/bin/mongod
/usr/share/doc/mongodb-org-server-3.2.6
/usr/share/doc/mongodb-org-server-3.2.6/GNU-AGPL-3.0
/usr/share/doc/mongodb-org-server-3.2.6/MPL-2
/usr/share/doc/mongodb-org-server-3.2.6/README
/usr/share/doc/mongodb-org-server-3.2.6/THIRD-PARTY-NOTICES
/usr/share/man/man1/mongod.1
/var/lib/mongo
/var/log/mongodb
/var/log/mongodb/mongod.log
/var/run/mongodb
[root@h105 mongo]#
~~~


其中几个重要的目录如下：


File     | Comment
-------- | ---
/etc/init.d/mongod | 启动脚本
/etc/mongod.conf | 配置文件
/etc/sysconfig/mongod    | 配置文件
/usr/bin/mongod | 服务程序
/var/log/mongodb/mongod.log | 日志文件
/var/lib/mongo | 数据文件目录


> **Tip:** 可以直接使用 **`/etc/init.d/mongod start`**   或 **`service mongod start`** 来启动 mongo 服务 


~~~
[root@h105 ~]# /etc/init.d/mongod start 
Starting mongod: 
                                                           [  OK  ]
[root@h105 ~]# 
~~~


我们这里准备手动创建一个实例，而不是使用现成的，这样可以更加了解其动作机制

---

###  手动创建实例

创建必要目录结构

~~~
[root@h105 ~]# mkdir mongo
[root@h105 ~]# cd mongo/
[root@h105 mongo]# ls
[root@h105 mongo]# mkdir data
[root@h105 mongo]# ls
data
[root@h105 mongo]# mkdir log
[root@h105 mongo]# mkdir conf
[root@h105 mongo]# mkdir bin
[root@h105 mongo]# ls
bin  conf  data  log
[root@h105 mongo]#
~~~

拷贝一份程序代码到 bin 目录中

~~~
[root@h105 mongo]# cp /usr/bin/mongod bin
[root@h105 mongo]# 
~~~

创建一个配置文件

~~~
[root@h105 mongo]# cd conf
[root@h105 conf]# vim mongod.conf 
[root@h105 conf]# cat mongod.conf 
port = 12345
dbpath = data
logpath = log/mongod.log
fork = true
[root@h105 conf]# 
~~~

---

### 启动服务 


~~~
[root@h105 mongo]# ./bin/mongod -f conf/mongod.conf 
about to fork child process, waiting until server is ready for connections.
forked process: 3505
child process started successfully, parent exiting
[root@h105 mongo]#
[root@h105 mongo]# netstat  -ant | grep 1234
tcp        0      0 0.0.0.0:12345               0.0.0.0:*                   LISTEN      
[root@h105 mongo]# ll data/
total 152
-rw-r--r-- 1 root root 16384 May 18 16:57 collection-0-1422751514030254971.wt
drwxr-xr-x 2 root root  4096 May 18 16:57 diagnostic.data
-rw-r--r-- 1 root root 16384 May 18 16:57 index-1-1422751514030254971.wt
drwxr-xr-x 2 root root  4096 May 18 15:57 journal
-rw-r--r-- 1 root root 16384 May 18 16:57 _mdb_catalog.wt
-rw-r--r-- 1 root root     0 May 18 16:57 mongod.lock
-rw-r--r-- 1 root root 32768 May 18 16:57 sizeStorer.wt
-rw-r--r-- 1 root root    95 May 18 15:57 storage.bson
-rw-r--r-- 1 root root    46 May 18 15:57 WiredTiger
-rw-r--r-- 1 root root  4096 May 18 16:57 WiredTigerLAS.wt
-rw-r--r-- 1 root root    21 May 18 15:57 WiredTiger.lock
-rw-r--r-- 1 root root   915 May 18 16:57 WiredTiger.turtle
-rw-r--r-- 1 root root 45056 May 18 16:57 WiredTiger.wt
[root@h105 mongo]# ll log/
total 4
-rw-r--r-- 1 root root 3736 May 18 16:57 mongod.log
[root@h105 mongo]# 
[root@h105 mongo]# tail log/mongod.log 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.511+0800 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/root/mongo/data/diagnostic.data'
2016-05-18T15:57:03.512+0800 I NETWORK  [HostnameCanonicalizationWorker] Starting hostname canonicalization worker
2016-05-18T15:57:03.542+0800 I NETWORK  [initandlisten] waiting for connections on port 12345
[root@h105 mongo]#
~~~

---

### 连接服务


使用mongo客户端来连接服务，mongo 客户端由 **mongodb-org-shell** 包提供

~~~
[root@h105 ~]# rpm -ql mongodb-org-shell-3.2.6-1.el6.x86_64
/usr/bin/mongo
/usr/share/man/man1/mongo.1
[root@h105 ~]# 
~~~

拷贝一份到 bin 中

~~~
[root@h105 mongo]# which mongo
/usr/bin/mongo
[root@h105 mongo]# cp /usr/bin/mongo bin/
[root@h105 mongo]# ll bin/
total 60696
-rwxr-xr-x 1 root root 22561172 May 18 16:54 mongo
-rwxr-xr-x 1 root root 39583898 May 18 15:55 mongod
[root@h105 mongo]# ./bin/mongo --help 
MongoDB shell version: 3.2.6
usage: ./bin/mongo [options] [db address] [file names (ending in .js)]
db address can be:
  foo                   foo database on local machine
  192.169.0.5/foo       foo database on 192.168.0.5 machine
  192.169.0.5:9999/foo  foo database on 192.168.0.5 machine on port 9999
Options:
  --shell                             run the shell after executing files
  --nodb                              don't connect to mongod on startup - no 
                                      'db address' arg expected
  --norc                              will not run the ".mongorc.js" file on 
                                      start up
  --quiet                             be less chatty
  --port arg                          port to connect to
  --host arg                          server to connect to
  --eval arg                          evaluate javascript
  -h [ --help ]                       show this usage information
  --version                           show version information
  --verbose                           increase verbosity
  --ipv6                              enable IPv6 support (disabled by default)
  --disableJavaScriptJIT              disable the Javascript Just In Time 
                                      compiler
  --enableJavaScriptProtection        disable automatic JavaScript function 
                                      marshalling
  --ssl                               use SSL for all connections
  --sslCAFile arg                     Certificate Authority file for SSL
  --sslPEMKeyFile arg                 PEM certificate/key file for SSL
  --sslPEMKeyPassword arg             password for key in PEM file for SSL
  --sslCRLFile arg                    Certificate Revocation List file for SSL
  --sslAllowInvalidHostnames          allow connections to servers with 
                                      non-matching hostnames
  --sslAllowInvalidCertificates       allow connections to servers with invalid
                                      certificates
  --sslFIPSMode                       activate FIPS 140-2 mode at startup

Authentication Options:
  -u [ --username ] arg               username for authentication
  -p [ --password ] arg               password for authentication
  --authenticationDatabase arg        user source (defaults to dbname)
  --authenticationMechanism arg       authentication mechanism
  --gssapiServiceName arg (=mongodb)  Service name to use when authenticating 
                                      using GSSAPI/Kerberos
  --gssapiHostName arg                Remote host name to use for purpose of 
                                      GSSAPI/Kerberos authentication

file names: a list of files to run. files have to end in .js and will exit after unless --shell is specified
[root@h105 mongo]# 
~~~


进行连接


~~~
[root@h105 mongo]# ./bin/mongo 127.0.0.1:12345/test
MongoDB shell version: 3.2.6
connecting to: 127.0.0.1:12345/test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
>
~~~

---

### 停止服务 

~~~
[root@h105 mongo]# ./bin/mongo 127.0.0.1:12345/test
MongoDB shell version: 3.2.6
connecting to: 127.0.0.1:12345/test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-05-18T15:57:03.510+0800 I CONTROL  [initandlisten] 
> db.shutdownServer()
shutdown command only works with the admin database; try 'use admin'
> use admin
switched to db admin
> db.shutdownServer()
server should be down...
2016-05-18T16:57:34.674+0800 I NETWORK  [thread1] trying reconnect to 127.0.0.1:12345 (127.0.0.1) failed
2016-05-18T16:57:34.682+0800 W NETWORK  [thread1] Failed to connect to 127.0.0.1:12345, reason: errno:111 Connection refused
2016-05-18T16:57:34.682+0800 I NETWORK  [thread1] reconnect 127.0.0.1:12345 (127.0.0.1) failed failed 
> ^C
bye
[root@h105 mongo]# 
~~~

查看日志

~~~
[root@h105 mongo]# tail -f log/mongod.log 
2016-05-18T16:57:34.670+0800 I CONTROL  [conn1] now exiting
2016-05-18T16:57:34.670+0800 I NETWORK  [conn1] shutdown: going to close listening sockets...
2016-05-18T16:57:34.670+0800 I NETWORK  [conn1] closing listening socket: 6
2016-05-18T16:57:34.670+0800 I NETWORK  [conn1] closing listening socket: 7
2016-05-18T16:57:34.670+0800 I NETWORK  [conn1] removing socket file: /tmp/mongodb-12345.sock
2016-05-18T16:57:34.670+0800 I NETWORK  [conn1] shutdown: going to flush diaglog...
2016-05-18T16:57:34.670+0800 I NETWORK  [conn1] shutdown: going to close sockets...
2016-05-18T16:57:34.670+0800 I STORAGE  [conn1] WiredTigerKVEngine shutting down
2016-05-18T16:57:34.823+0800 I STORAGE  [conn1] shutdown: removing fs lock...
2016-05-18T16:57:34.823+0800 I CONTROL  [conn1] dbexit:  rc: 0
^C
[root@h105 mongo]#
~~~




---

# 命令汇总

* **`vim /etc/yum.repos.d/mongodb-org-3.2.repo`**
* **`yum clean all`**
* **`yum install -y mongodb-org`**
* **`mongo --version`**
* **`getenforce`**
* **`grep -i '^SELINUX' /etc/sysconfig/selinux`**
* **`rpm -ql mongodb-org-server`**
* **`/etc/init.d/mongod start`**
* **`mkdir mongo`**
* **`cd mongo/`**
* **`mkdir data`**
* **`mkdir log`**
* **`mkdir conf`**
* **`mkdir bin`**
* **`cp /usr/bin/mongod bin`**
* **`cd conf`**
* **`vim mongod.conf`**
* **`./bin/mongod -f conf/mongod.conf`**
* **`netstat  -ant | grep 1234`**
* **`tail log/mongod.log`**
* **`rpm -ql mongodb-org-shell-3.2.6-1.el6.x86_64`**
* **`cp /usr/bin/mongo bin/`**
* **`./bin/mongo --help`**
* **`./bin/mongo 127.0.0.1:12345/test`**
* **`tail -f log/mongod.log`**


---


[mongodb]:https://www.mongodb.com/
[mongodb_doc]:https://docs.mongodb.com/
[neo4j]:http://soft.dog/2016/04/20/neo4j-basic/
[qianyan]:http://soft.dog/2016/04/20/neo4j-basic/#section
[mongodb_dl]:https://www.mongodb.com/download-center?jmp=nav#community
[mongodb_install]:https://docs.mongodb.com/master/tutorial/install-mongodb-on-red-hat/
[supported_platforms]:https://docs.mongodb.com/master/installation/#supported-platforms

