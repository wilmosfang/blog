---
layout: post
title: c1000k TCP 连接上限测试
author: wilmosfang
tags:  benchmark c1000k
categories:  c1000k
wc: 262 792 6117
excerpt: follow me
comments: true
---


---

# 前言


**[c1000k][c1000k]** 是一套用来测试本地OS TCP连接上限的C/S小工具。

>This is the TCP server-client suit to help you test if your OS supports c1000k(1 million connections).


下面分享一下 **[c1000k][c1000k]** 的基本使用方法，详细可以参阅 [官方文档][c1000k]



---


# 概要

* TOC
{:toc}



---

## 下载和安装

使用下面的方式安装

~~~
wget --no-check-certificate https://github.com/ideawu/c1000k/archive/master.zip
unzip master.zip
cd c1000k-master
make
~~~

安装过程

~~~
[root@h101 c1000k]# wget --no-check-certificate https://github.com/ideawu/c1000k/archive/master.zip
--2015-11-25 13:44:07--  https://github.com/ideawu/c1000k/archive/master.zip
Resolving github.com... 192.30.252.130
Connecting to github.com|192.30.252.130|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/ideawu/c1000k/zip/master [following]
--2015-11-25 13:44:09--  https://codeload.github.com/ideawu/c1000k/zip/master
Resolving codeload.github.com... 192.30.252.146
Connecting to codeload.github.com|192.30.252.146|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/zip]
Saving to: “master.zip”

    [ <=>                                                                        ] 3,075       --.-K/s   in 0s      

2015-11-25 13:44:11 (11.3 MB/s) - “master.zip” saved [3075]

[root@h101 c1000k]# ls
master.zip
[root@h101 c1000k]# du -sh master.zip 
4.0K	master.zip
[root@h101 c1000k]# unzip master.zip 
Archive:  master.zip
f435ee41fca847b71aab679fed1d9b07c979cfba
   creating: c1000k-master/
 extracting: c1000k-master/.gitignore  
  inflating: c1000k-master/Makefile  
  inflating: c1000k-master/README.md  
  inflating: c1000k-master/client.c  
  inflating: c1000k-master/server.c  
[root@h101 c1000k]# ls 
c1000k-master  master.zip
[root@h101 c1000k]# cd c1000k-master/
[root@h101 c1000k-master]# ls
client.c  Makefile  README.md  server.c
[root@h101 c1000k-master]# make 
gcc -std=c99 -O2 -o server server.c
gcc -O2 -o client client.c
[root@h101 c1000k-master]#
~~~

---

## 启动服务

指定一个空闲端口，服务端会顺次打开100个端口进行监听，并且在当前terminal挂起

~~~
[root@h101 c1000k-master]# ./server  8000
server listen on port: 8000
server listen on port: 8001
server listen on port: 8002
server listen on port: 8003
server listen on port: 8004
server listen on port: 8005
server listen on port: 8006
server listen on port: 8007
server listen on port: 8008
server listen on port: 8009
server listen on port: 8010
server listen on port: 8011
server listen on port: 8012
server listen on port: 8013
server listen on port: 8014
server listen on port: 8015
server listen on port: 8016
server listen on port: 8017
server listen on port: 8018
server listen on port: 8019
server listen on port: 8020
server listen on port: 8021
server listen on port: 8022
server listen on port: 8023
server listen on port: 8024
server listen on port: 8025
server listen on port: 8026
server listen on port: 8027
server listen on port: 8028
server listen on port: 8029
server listen on port: 8030
server listen on port: 8031
server listen on port: 8032
server listen on port: 8033
server listen on port: 8034
server listen on port: 8035
server listen on port: 8036
server listen on port: 8037
server listen on port: 8038
server listen on port: 8039
server listen on port: 8040
server listen on port: 8041
server listen on port: 8042
server listen on port: 8043
server listen on port: 8044
server listen on port: 8045
server listen on port: 8046
server listen on port: 8047
server listen on port: 8048
server listen on port: 8049
server listen on port: 8050
server listen on port: 8051
server listen on port: 8052
server listen on port: 8053
server listen on port: 8054
server listen on port: 8055
server listen on port: 8056
server listen on port: 8057
server listen on port: 8058
server listen on port: 8059
server listen on port: 8060
server listen on port: 8061
server listen on port: 8062
server listen on port: 8063
server listen on port: 8064
server listen on port: 8065
server listen on port: 8066
server listen on port: 8067
server listen on port: 8068
server listen on port: 8069
server listen on port: 8070
server listen on port: 8071
server listen on port: 8072
server listen on port: 8073
server listen on port: 8074
server listen on port: 8075
server listen on port: 8076
server listen on port: 8077
server listen on port: 8078
server listen on port: 8079
server listen on port: 8080
server listen on port: 8081
server listen on port: 8082
server listen on port: 8083
server listen on port: 8084
server listen on port: 8085
server listen on port: 8086
server listen on port: 8087
server listen on port: 8088
server listen on port: 8089
server listen on port: 8090
server listen on port: 8091
server listen on port: 8092
server listen on port: 8093
server listen on port: 8094
server listen on port: 8095
server listen on port: 8096
server listen on port: 8097
server listen on port: 8098
server listen on port: 8099


~~~

---

## 运行客户端



~~~
[root@h101 c1000k-master]# ./client 127.0.0.1 8000
connections: 922
error: Connection refused
[root@h101 c1000k-master]# 
~~~

运行完服务端也会跟着退出

~~~
...
...
server listen on port: 8095
server listen on port: 8096
server listen on port: 8097
server listen on port: 8098
server listen on port: 8099
connections: 921
error: Too many open files
[root@h101 c1000k-master]# 
~~~

---

## 结果分析

客户端反馈结果

~~~
connections: 922
error: Connection refused
~~~

说明客户端的第922个连接被拒绝了


服务端反馈结果

~~~
connections: 921
error: Too many open files
~~~

说明服务端只能打开921个连接，无法打开更多，触及了 **max-open-files** 规定的上限，所以退出了

---

[c1000k]:https://github.com/ideawu/c1000k



