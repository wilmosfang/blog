---
layout: post
title:  Docker Registry
author: wilmosfang
tags:   network docker
categories:   docker
wc: 1268 4423 49312
excerpt: 本地Registry服务的部署与销毁，远程Registry服务的部署与销毁，Docker卷，DockerTLS加密，基本认证与访问控制，使用Compose构建容器，docker-compose.yml的编辑与注意事项，Registry部署过程中的常见问题处理
comments: true
---



# 前言

**[Docker][docker]** 是围绕 **Images** 进行管理的

![docker-stages.png](/images/docker/docker-stages.png)

构建一个私有的镜像仓库可以更高效地管理镜像

在 **[Docker][docker]** 中，镜像仓库叫 **[Registry][registry]**

>A registry is a storage and content delivery system, holding named Docker images, available in different tagged versions.


**[Registry][registry]** 是开源的，高弹性的，可以更为容易地对生产测试环境里的镜像进行定制化管理

>The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images. The Registry is open-source, under the permissive Apache license.

这里分享一下 **[Docker Registry][registry]** 的相关基础，详细可以参阅 **[官方文档][registry]** 

> **Tip:** 当前的最新版本为 **Docker 1.10** Released on January 15, 2016

---


# 概要

* TOC
{:toc}


---

## 依赖

Registry 要求构建在不小于 **1.6.0** 版本的 Docker 引擎上

>The Registry is compatible with Docker engine version 1.6.0 or higher


---

## Registry的创建与销毁


### 创建运行Registry

~~~
[root@h103 ~]# docker run -d -p 5000:5000 --name registry registry:2
Unable to find image 'registry:2' locally
2: Pulling from library/registry
fcee8bcfe180: Pull complete 
4cdc0cbc1936: Pull complete 
d9e545b90db8: Pull complete 
c4bea91afef3: Pull complete 
d03a562198ae: Pull complete 
d2e8bfe6f2bc: Pull complete 
51d207c7259b: Pull complete 
7148a81f93cb: Pull complete 
b239a09153bd: Pull complete 
8f1214c20b01: Pull complete 
683f9cd9cf88: Pull complete 
Digest: sha256:a842b52833778977f7b4466b90cc829e0f9aae725aebe3e32a5a6c407acd2a03
Status: Downloaded newer image for registry:2
7716d7899161a529780b55a51b541953275f0d63bb97f9630b2edab26e1d556f
[root@h103 ~]# echo $?
0
[root@h103 ~]# 
~~~

---

### 从Docker Hub拉取镜像

~~~
[root@h103 ~]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
92ec6d044cb3: Verifying Checksum 
2ef91804894a: Download complete 
f80999a1f330: Download complete 
6cc0fc2a5ee3: Download complete 
Pulling repository docker.io/library/ubuntu
8693db7e8a00: Download complete 
f15ce52fc004: Download complete 
c4fae638e7ce: Download complete 
a4c5be5b6e59: Download complete 
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu: this image was pulled from a legacy registry.  Important: This registry version will not be supported in future versions of docker.
[root@h103 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              8693db7e8a00        9 hours ago         187.9 MB
registry            2                   683f9cd9cf88        2 weeks ago         224.5 MB
hello-world         latest              0a6ba66e537a        3 months ago        960 B
[root@h103 ~]# 
~~~


---

### 镜像打标

~~~
[root@h103 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              8693db7e8a00        9 hours ago         187.9 MB
registry            2                   683f9cd9cf88        2 weeks ago         224.5 MB
hello-world         latest              0a6ba66e537a        3 months ago        960 B 
[root@h103 ~]# docker tag ubuntu localhost:5000/myfirstimage 
[root@h103 ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                        latest              8693db7e8a00        9 hours ago         187.9 MB
localhost:5000/myfirstimage   latest              8693db7e8a00        9 hours ago         187.9 MB
registry                      2                   683f9cd9cf88        2 weeks ago         224.5 MB
hello-world                   latest              0a6ba66e537a        3 months ago        960 B
[root@h103 ~]#
~~~





---

### 推送镜像到Registry


~~~
[root@h103 ~]# docker push localhost:5000/myfirstimage 
The push refers to a repository [localhost:5000/myfirstimage] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:a27637294694a32300c5a9b94c9078709ec75216dd875fbdbc89acb0eb803401 size: 6806
[root@h103 ~]# 
~~~


---

### 从Registry拉取镜像


~~~
[root@h103 ~]# docker pull localhost:5000/myfirstimage
Using default tag: latest
latest: Pulling from myfirstimage
Digest: sha256:a27637294694a32300c5a9b94c9078709ec75216dd875fbdbc89acb0eb803401
Status: Image is up to date for localhost:5000/myfirstimage:latest
[root@h103 ~]# echo $?
0
[root@h103 ~]# 
~~~

---

### 销毁Registry


registry和其它实例没有任何区别，使用stop然后rm就可以便捷地进行销毁

~~~
[root@h103 ~]# docker  ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAME
7716d7899161        registry:2          "/bin/registry /etc/d"   22 hours ago        Up 2 minutes        0.0.0.0:5000->5000/tcp   regi
[root@h103 ~]# docker stop 7716d7899161
7716d7899161
[root@h103 ~]# docker  ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NA
7716d7899161        registry:2          "/bin/registry /etc/d"   22 hours ago        Exited (2) 1 seconds ago                       re
[root@h103 ~]# docker rm 7716d7899161
7716d7899161
[root@h103 ~]# docker  ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@h103 ~]# 
~~~

---


## 部署本地Registry服务

~~~
[root@h103 ~]# docker run -d -p 5000:5000 --restart=always --name registry registry:2
4352b16f2582ed0478f3380be5ab4a65487d7adf1698c66f365881e3aefdab68
[root@h103 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4352b16f2582        registry:2          "/bin/registry /etc/d"   7 seconds ago       Up 5 seconds        0.0.0.0:5000->5000/tcp   registry
[root@h103 ~]# docker pull ubuntu &&  docker tag ubuntu localhost:5000/ubuntu
Using default tag: latest
Pulling repository docker.io/library/ubuntu
8693db7e8a00: Download complete 
8693db7e8a00: Pulling image (latest) from docker.io/library/ubuntu 
f15ce52fc004: Download complete 
c4fae638e7ce: Download complete 
Status: Image is up to date for ubuntu:latest
docker.io/library/ubuntu: this image was pulled from a legacy registry.  Important: This registry version will not be supported in future versions of docker.
[root@h103 ~]# echo $?
0
[root@h103 ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
localhost:5000/myfirstimage   latest              8693db7e8a00        30 hours ago        187.9 MB
ubuntu                        latest              8693db7e8a00        30 hours ago        187.9 MB
localhost:5000/ubuntu         latest              8693db7e8a00        30 hours ago        187.9 MB
jenkins                       latest              fc39417bd5fb        12 days ago         708.1 MB
registry                      2                   683f9cd9cf88        2 weeks ago         224.5 MB
hello-world                   latest              0a6ba66e537a        3 months ago        960 B
[root@h103 ~]# docker push localhost:5000/ubuntu 
The push refers to a repository [localhost:5000/ubuntu] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h103 ~]# 
[root@h103 ~]# docker pull localhost:5000/ubuntu
Using default tag: latest
latest: Pulling from ubuntu
Digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3
Status: Image is up to date for localhost:5000/ubuntu:latest
[root@h103 ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
localhost:5000/myfirstimage   latest              8693db7e8a00        31 hours ago        187.9 MB
ubuntu                        latest              8693db7e8a00        31 hours ago        187.9 MB
localhost:5000/ubuntu         latest              8693db7e8a00        31 hours ago        187.9 MB
jenkins                       latest              fc39417bd5fb        12 days ago         708.1 MB
registry                      2                   683f9cd9cf88        2 weeks ago         224.5 MB
hello-world                   latest              0a6ba66e537a        3 months ago        960 B
[root@h103 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4352b16f2582        registry:2          "/bin/registry /etc/d"   28 minutes ago      Up 28 minutes       0.0.0.0:5000->5000/tcp   registry
[root@h103 ~]# docker stop registry && docker rm -v registry
registry
registry
[root@h103 ~]# 
~~~

---

### 存储


默认情况下，registry 中的数据是以docker卷的形式存在于本地文件系统


可以使用 **`-v`** 的参数来指定一个卷的位置，从而实现对数据存储的控制


~~~
[root@h103 ~]# ls
anaconda-ks.cfg  dockerfile
[root@h103 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@h103 ~]# echo `pwd`
/root
[root@h103 ~]# docker  run -d -p 5000:5000 --restart=always --name registry -v `pwd`/data:/var/lib/registry registry:2
f0e1c155d7ad1e0607e33f9f0b9ff23f1d7e4761b88070486425f3137b513540
[root@h103 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
f0e1c155d7ad        registry:2          "/bin/registry /etc/d"   9 seconds ago       Up 6 seconds        0.0.0.0:5000->5000/tcp   registry
[root@h103 ~]# ls
anaconda-ks.cfg  data  dockerfile
[root@h103 ~]# cd data/
[root@h103 data]# ls
[root@h103 data]# cd ..
[root@h103 ~]# docker push localhost:5000/ubuntu
The push refers to a repository [localhost:5000/ubuntu] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h103 ~]# 
[root@h103 ~]# tree data/
data/
└── docker
    └── registry
        └── v2
            ├── blobs
            │   └── sha256
            │       ├── 27
            │       │   └── 2796840645a7bf9739e3859ba390d8adfbfa9bf8ddbce09feb875a1840df7f38
            │       │       └── data
            │       ├── 3b
            │       │   └── 3b52deaaf0edb8a0282a08dd9c9e25da2050a75739b832ecc6e29941394933a6
            │       │       └── data
            │       ├── 45
            │       │   └── 45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3
            │       │       └── data
            │       ├── 4b
            │       │   └── 4bd501fad6defc3af5638b82f7d760f0dc2f2c5f1bcd2cbfd59607b1631bc679
            │       │       └── data
            │       ├── 83
            │       │   └── 8387d9ff0016d004777e511a55e21672e4b6de49e32db2544b8ac0e2ee01d5ed
            │       │       └── data
            │       └── a3
            │           └── a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4
            │               └── data
            └── repositories
                └── ubuntu
                    ├── _layers
                    │   └── sha256
                    │       ├── 3b52deaaf0edb8a0282a08dd9c9e25da2050a75739b832ecc6e29941394933a6
                    │       │   └── link
                    │       ├── 4bd501fad6defc3af5638b82f7d760f0dc2f2c5f1bcd2cbfd59607b1631bc679
                    │       │   └── link
                    │       ├── 8387d9ff0016d004777e511a55e21672e4b6de49e32db2544b8ac0e2ee01d5ed
                    │       │   └── link
                    │       └── a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4
                    │           └── link
                    ├── _manifests
                    │   ├── revisions
                    │   │   └── sha256
                    │   │       └── 45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3
                    │   │           ├── link
                    │   │           └── signatures
                    │   │               └── sha256
                    │   │                   └── 2796840645a7bf9739e3859ba390d8adfbfa9bf8ddbce09feb875a1840df7f38
                    │   │                       └── link
                    │   └── tags
                    │       └── latest
                    │           ├── current
                    │           │   └── link
                    │           └── index
                    │               └── sha256
                    │                   └── 45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3
                    │                       └── link
                    └── _uploads

39 directories, 14 files
[root@h103 ~]# 
~~~


---

## 部署远程Registry服务


### 创建自签名证书


~~~
[root@h104 ~]# cd certs/
[root@h104 certs]# openssl genrsa -out docker.key 2048
Generating RSA private key, 2048 bit long modulus
..............................................................................+++
................................................................................................+++
e is 65537 (0x10001)
[root@h104 certs]# openssl req -new -key docker.key -out docker.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:Shanghai
Locality Name (eg, city) [Default City]:Shanghai
Organization Name (eg, company) [Default Company Ltd]:docker
Organizational Unit Name (eg, section) []:docker         
Common Name (eg, your name or your server's hostname) []:docker-registry
Email Address []:ok@docker.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
[root@h104 certs]# openssl x509 -req -days 365 -in docker.csr -signkey docker.key -out docker.crt
Signature ok
subject=/C=CN/ST=Shanghai/L=Shanghai/O=docker/OU=docker/CN=docker-registry/emailAddress=ok@docker.com
Getting Private key
[root@h104 certs]# ll
total 12
-rw-r--r-- 1 root root 1306 Jan 21 22:04 docker.crt
-rw-r--r-- 1 root root 1058 Jan 21 22:04 docker.csr
-rw-r--r-- 1 root root 1675 Jan 21 22:02 docker.key
[root@h104 certs]# chmod 600 * 
[root@h104 certs]# ll
total 12
-rw------- 1 root root 1306 Jan 21 22:04 docker.crt
-rw------- 1 root root 1058 Jan 21 22:04 docker.csr
-rw------- 1 root root 1675 Jan 21 22:02 docker.key
[root@h104 certs]# cd ..
[root@h104 ~]# 
~~~


### 运行Registry

~~~
[root@h104 ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@h104 ~]# ls
anaconda-ks.cfg  certs  dockerfile
[root@h104 ~]# docker run -d -p 5000:5000 --restart=always --name registry  -v `pwd`/data:/var/lib/registry  -v `pwd`/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker.crt -e REGISTRY_HTTP_TLS_KEY=/certs/docker.key registry:2
b578e321f33f6f2a0c34340b35239d1ce724c4523f3b2266bc01239658fc3f46
[root@h104 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
b578e321f33f        registry:2          "/bin/registry /etc/d"   6 seconds ago       Up 5 seconds        0.0.0.0:5000->5000/tcp   registry
[root@h104 ~]#
~~~



### 尝试push一个镜像


先tag一些镜像出来

其实就是将本地的镜像作一些别名(链接)

~~~
[root@h103 ~]# docker tag ubuntu 192.168.100.104:5000/ubuntu
[root@h103 ~]# docker tag ubuntu h104:5000/ubuntu
[root@h103 ~]# docker tag ubuntu docker-registry:5000/ubuntu 
[root@h103 ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
192.168.100.104:5000/ubuntu   latest              8693db7e8a00        39 hours ago        187.9 MB
h104:5000/ubuntu              latest              8693db7e8a00        39 hours ago        187.9 MB
localhost:5000/myfirstimage   latest              8693db7e8a00        39 hours ago        187.9 MB
localhost:5000/ubuntu         latest              8693db7e8a00        39 hours ago        187.9 MB
docker:5000/ubuntu            latest              8693db7e8a00        39 hours ago        187.9 MB
h103:5000/ubuntu              latest              8693db7e8a00        39 hours ago        187.9 MB
ubuntu                        latest              8693db7e8a00        39 hours ago        187.9 MB
docker-registry:5000/ubuntu   latest              8693db7e8a00        39 hours ago        187.9 MB
jenkins                       latest              fc39417bd5fb        12 days ago         708.1 MB
registry                      2                   683f9cd9cf88        2 weeks ago         224.5 MB
hello-world                   latest              0a6ba66e537a        3 months ago        960 B
[root@h103 ~]#
~~~



再次尝试push

#### 报错1

~~~
[root@h103 ~]# docker push h104:5000/ubuntu
The push refers to a repository [h104:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://h104:5000/v0/
v2 ping attempt failed with error: Get https://h104:5000/v2/: tls: oversized record received with length 20527
 v1 ping attempt failed with error: Get https://h104:5000/v1/_ping: tls: oversized record received with length 20527
[root@h103 ~]# 
~~~

根据官网的解释和方法，我没有成功处理

官网的解释如下：

原因是没有加入证书或证书不被信任，解决办法是从证书入手

有三种方式可以解决：

* 1.买一个SSL证书
* 2.配置docker忽视指定registry的安全

> **`DOCKER_OPTS="--insecure-registry myregistrydomain.com:5000"`** ，然后重启客户端

* 3.导入自签名证书，让docker客户端单向相信这个registry，然后重启客户端




* 解决办法：

最后的解决办法是将registry删除重建，问题就没再出现了

* 可能原因：

所以，我猜测可能是(当时)我构建这个registry的过程中环境变量配置错误了

---

#### 报错234


~~~
[root@h103 ~]# docker push 192.168.100.104:5000/ubuntu
The push refers to a repository [192.168.100.104:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://192.168.100.104:5000/v0/
v2 ping attempt failed with error: Get https://192.168.100.104:5000/v2/: x509: cannot validate certificate for 192.168.100.104 because it doesn't contain any IP SANs
 v1 ping attempt failed with error: Get https://192.168.100.104:5000/v1/_ping: x509: cannot validate certificate for 192.168.100.104 because it doesn't contain any IP SANs
[root@h103 ~]#
~~~

原因是证书中没有指定IP

~~~
[root@h103 ~]# docker push h104:5000/ubuntu
The push refers to a repository [h104:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://h104:5000/v0/
v2 ping attempt failed with error: Get https://h104:5000/v2/: x509: certificate is valid for docker-registry, not h104
 v1 ping attempt failed with error: Get https://h104:5000/v1/_ping: x509: certificate is valid for docker-registry, not h104
[root@h103 ~]#
~~~

原因是证书中指定的主机名为 **docker-registry** 而不是 **h104**

~~~
[root@h103 ~]# vim /etc/hosts
[root@h103 ~]# grep docker-registry  /etc/hosts
192.168.100.104  h104 docker-registry
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker-registry:5000/v0/
v2 ping attempt failed with error: Get https://docker-registry:5000/v2/: x509: certificate signed by unknown authority
 v1 ping attempt failed with error: Get https://docker-registry:5000/v1/_ping: x509: certificate signed by unknown authority
[root@h103 ~]# 
~~~

原因是证书不被信任(自签名证书)


* 解决办法一：

将证书内容导入受信列表，重启docker客户端

~~~
[root@h103 ~]# ll /etc/pki/tls/certs/ca-bundle.crt
lrwxrwxrwx 1 root root 49 Jan 19 16:30 /etc/pki/tls/certs/ca-bundle.crt -> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
[root@h103 ~]# ll /etc/pki/ca-trust/extracted/pem/
total 692
-r--r--r-- 1 root root 217510 Jan 19 16:30 email-ca-bundle.pem
-r--r--r-- 1 root root 211626 Jan 19 16:30 objsign-ca-bundle.pem
-rw-r--r-- 1 root root    897 Apr 23  2015 README
-r--r--r-- 1 root root 267983 Jan 21 21:21 tls-ca-bundle.pem
[root@h103 ~]# scp root@h104:/root/certs/docker.crt /etc/pki/ca-trust/extracted/pem/
root@h104's password: 
docker.crt                                                                                          100% 1306     1.3KB/s   00:00    
[root@h103 ~]# ll /etc/pki/ca-trust/extracted/pem/
total 696
-rw------- 1 root root   1306 Jan 21 23:24 docker.crt
-r--r--r-- 1 root root 217510 Jan 19 16:30 email-ca-bundle.pem
-r--r--r-- 1 root root 211626 Jan 19 16:30 objsign-ca-bundle.pem
-rw-r--r-- 1 root root    897 Apr 23  2015 README
-r--r--r-- 1 root root 267983 Jan 21 21:21 tls-ca-bundle.pem
[root@h103 ~]# cat /etc/pki/ca-trust/extracted/pem/docker.crt >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker-registry:5000/v0/
v2 ping attempt failed with error: Get https://docker-registry:5000/v2/: x509: certificate signed by unknown authority
 v1 ping attempt failed with error: Get https://docker-registry:5000/v1/_ping: x509: certificate signed by unknown authority
[root@h103 ~]# systemctl stop docker && systemctl start docker
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h103 ~]# docker pull  docker-registry:5000/ubuntu
Using default tag: latest
latest: Pulling from ubuntu
Digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3
Status: Image is up to date for docker-registry:5000/ubuntu:latest
[root@h103 ~]# 
~~~

> **Note:** 一定要重启客户端，否则无效



* 解决办法二：


~~~
[root@h103 ~]# ll /etc/pki/ca-trust/source/anchors/
total 0
[root@h103 ~]# scp root@h104:/root/certs/docker.crt  /etc/pki/ca-trust/source/anchors/
root@h104's password: 
docker.crt                                                                                                            100% 1306     1.3KB/s   00:00    
[root@h103 ~]# ll /etc/pki/ca-trust/source/anchors/
total 4
-rw------- 1 root root 1306 Jan 21 23:49 docker.crt
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker-registry:5000/v0/
v2 ping attempt failed with error: Get https://docker-registry:5000/v2/: x509: certificate signed by unknown authority
 v1 ping attempt failed with error: Get https://docker-registry:5000/v1/_ping: x509: certificate signed by unknown authority
[root@h103 ~]# systemctl stop docker && systemctl start docker 
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker-registry:5000/v0/
v2 ping attempt failed with error: Get https://docker-registry:5000/v2/: x509: certificate signed by unknown authority
 v1 ping attempt failed with error: Get https://docker-registry:5000/v1/_ping: x509: certificate signed by unknown authority
[root@h103 ~]# 
[root@h103 ~]# update-ca-trust 
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker-registry:5000/v0/
v2 ping attempt failed with error: Get https://docker-registry:5000/v2/: x509: certificate signed by unknown authority
 v1 ping attempt failed with error: Get https://docker-registry:5000/v1/_ping: x509: certificate signed by unknown authority
[root@h103 ~]# systemctl stop docker && systemctl start docker 
[root@h103 ~]# docker push docker-registry:5000/ubuntu
The push refers to a repository [docker-registry:5000/ubuntu] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h103 ~]# docker pull docker-registry:5000/ubuntu
Using default tag: latest
latest: Pulling from ubuntu
Digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3
Status: Image is up to date for docker-registry:5000/ubuntu:latest
[root@h103 ~]# 
~~~

为什么这么啰嗦地反复测试，是为了说明以下三步必须且只能按照以下步骤完成，否则无法生效


* 拷贝自签证书到 **`/etc/pki/ca-trust/source/anchors/`** 中(只能是这个目录，其它不行)
* 执行 **update-ca-trust** 刷新受信列表
* 重启docker客户端



> **Note:** **Common Name** 要设置得和库(访问域名)的名字一样否则检查证书时会报错，客户端配置完证书要重启才能生效



---

#### 其它报错


类似于下面两种

~~~
[root@h104 ~]# docker push docker:5000/ubuntu 
The push refers to a repository [docker:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker:5000/v0/
v2 ping attempt failed with error: Get https://docker:5000/v2/: dial tcp 192.168.100.103:5000: no route to host
 v1 ping attempt failed with error: Get https://docker:5000/v1/_ping: dial tcp 192.168.100.103:5000: no route to host
[root@h104 ~]# 
~~~


~~~
[root@h104 ~]# docker push docker:5000/ubuntu 
The push refers to a repository [docker:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker:5000/v0/
v2 ping attempt failed with error: Get https://docker:5000/v2/: dial tcp 192.168.100.103:5000: i/o timeout
 v1 ping attempt failed with error: Get https://docker:5000/v1/_ping: dial tcp 192.168.100.103:5000: i/o timeout
[root@h104 ~]# 
~~~

* 故障原因

一般而言，防火墙会在docker服务之前打开，docker服务启动后会在iptables中应用一些策略

~~~
[root@docker ~]# systemctl list-dependencies docker.service | head -n 10
docker.service
● ├─docker.socket
● ├─system.slice
● └─basic.target
●   ├─firewalld.service
●   ├─microcode.service
●   ├─rhel-autorelabel-mark.service
●   ├─rhel-autorelabel.service
●   ├─rhel-configure.service
●   ├─rhel-dmesg.service
[root@docker ~]# 
[root@docker ~]# iptables -L -nv | grep -i docker
  288 46767 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
  224 45545 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           
Chain DOCKER (1 references)
  288 46767 ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.2           tcp dpt:5000
[root@docker ~]# 
~~~



如果单独重载iptables服务，docker这边的配置会丢失

~~~
[root@docker ~]# firewall-cmd --reload
success
[root@docker ~]# iptables -L -nv | grep -i docker
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           
[root@docker ~]# 
~~~

从而导致网络不可达或相关信息的报错

~~~
[root@h104 ~]# docker push docker:5000/ubuntu 
The push refers to a repository [docker:5000/ubuntu] (len: 1)
unable to ping registry endpoint https://docker:5000/v0/
v2 ping attempt failed with error: Get https://docker:5000/v2/: dial tcp 192.168.100.103:5000: no route to host
 v1 ping attempt failed with error: Get https://docker:5000/v1/_ping: dial tcp 192.168.100.103:5000: no route to host
[root@h104 ~]# 
~~~

* 解决办法

就是确保在iptables服务重启后，docker服务也重启一下，以应用docker里的网络策略(最主要的是加载那条 **Chain DOCKER**)

~~~
[root@docker ~]# systemctl stop docker && systemctl  start docker 
[root@docker ~]# iptables -L -nv | grep -i docker 
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           
Chain DOCKER (1 references)
    0     0 ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.2           tcp dpt:5000
[root@docker ~]# 
----------
[root@h104 ~]# docker push docker:5000/ubuntu 
The push refers to a repository [docker:5000/ubuntu] (len: 1)

8693db7e8a00: Image already exists 
a4c5be5b6e59: Image already exists 
c4fae638e7ce: Image already exists 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h104 ~]# 
~~~


> **Tip:** 由docker export出来的端口不必在主机的防火墙filter表中另外打开，因为它的数据进入了forward链中


---

## Registry负载均衡


目前可以使用多个容器共享存储的方式来实现负载均衡

下面的三点要一样:

* 存储空间
* HTTP Secret 证书
* Redis 缓存(如果有的话)


---

## 访问控制

可以使用本地基础认证在TLS加密的基础上进行更细粒度的访问控制

这个机制和http的基础认证是一样的，由于是简单密码，明文传送，所以只有ssl加密的环境中才有安全保障


### 创建密码文件

首先创建一个密码文件

用户名密码：**testuser/testpassword**

~~~
[root@docker ~]# ls
anaconda-ks.cfg  certs  dockerfile
[root@docker ~]# mkdir auth
[root@docker ~]# docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
[root@docker ~]# ll auth/
total 4
-rw-r--r-- 1 root root 71 Jan 22 15:46 htpasswd
[root@docker ~]# cat auth/htpasswd 
testuser:$2y$05$.NF64Yoz4W/VCfM1RrkBw.CT7ji3TbzdgBWjIH6X60MMgNFC.vIy.

[root@docker ~]#
~~~

### 创建一个registry

这个registry

* 指定了卷
* TLS加密
* 基础认证

先清掉docker中同名的registry，然后再创建，否则会报冲突，也可以给这个registry改为其它名字

~~~
[root@docker ~]# docker run -d -p 5000:5000 --restart=always --name registry \
> -v `pwd`/data:/data \
> -v `pwd`/certs:/certs \
> -v `pwd`/auth:/auth \
> -e "REGISTRY_AUTH=htpasswd" \
> -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
> -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
> -e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker.crt" \
> -e "REGISTRY_HTTP_TLS_KEY=/certs/docker.key" \
> registry:2
71de3ba937945006578d495ed09ec36ca141130e1e22b3083018b9d43a251767
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                    NAMES
71de3ba93794        registry:2          "/bin/registry /etc/d"   4 seconds ago       Up 3 seconds               0.0.0.0:5000->5000/tcp   registry
3d9f0915226f        registry:2          "htpasswd -Bbn testus"   5 minutes ago       Exited (0) 5 minutes ago                            prickly_jang
27995af3fa59        registry:2          "htpasswd -Bbn testus"   2 hours ago         Exited (0) 2 hours ago                              gloomy_goldberg
[root@docker ~]#
~~~

### 尝试push一个镜像

~~~
[root@h104 ~]# docker push docker:5000/ubuntu 
The push refers to a repository [docker:5000/ubuntu] (len: 1)

Head https://docker:5000/v2/ubuntu/blobs/sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4: no basic auth credentials
[root@h104 ~]#
~~~

报错是因为没有进行认证

### 进行认证

~~~
[root@h104 ~]# docker login docker:5000
Username: testuser
Password: 
Email: yyghdfz@163.com
WARNING: login credentials saved in /root/.docker/config.json
Login Succeeded
[root@h104 ~]# docker push docker:5000/ubuntu 
The push refers to a repository [docker:5000/ubuntu] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h104 ~]#
~~~


---

## 使用Compose构建容器

**[Docker Compose][compose]** 是一个docker容器编排工具，可以有效完成多容器对接和组合等工作

如果命令行中输入太多参数变得不方便时，也可以使用它来进行单个容器的配置

相关详情可以参考 **[Docker Compose][compose]** 官方说明

**[Overview of Docker Compose][compose_doc]**

以后有机会再进行深入研究

### 下载安装Compose

可以使用下面两种方法进行安装

~~~
[root@h104 ~]# curl -L https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m` > docker-compose
...
...
[root@h104 ~]# wget https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m`
...
...
[root@h104 ~]# 
~~~

下载可以参考 **[Install Docker Compose][compose_install]**

> **Tip:** 如果不翻墙，这个不到10M的文件，可以让人崩溃

~~~
4% [===>                                                                                                           ] 321,090      423B/s  eta 7h 17m 
~~~


### Compose软件基础信息

~~~
[root@docker ~]# ls
anaconda-ks.cfg  auth  certs  data  docker-compose-Linux-x86_64  dockerfile
[root@docker ~]# du -sh docker-compose-Linux-x86_64 
7.6M	docker-compose-Linux-x86_64
[root@docker ~]# file docker-compose-Linux-x86_64 
docker-compose-Linux-x86_64: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=853203ebc6482b8f7e218413e2d0ee3d7d39e234, stripped
[root@docker ~]# chmod +x docker-compose-Linux-x86_64 
[root@docker ~]# ./docker-compose-Linux-x86_64 version
docker-compose version 1.5.2, build 7240ff3
docker-py version: 1.5.0
CPython version: 2.7.9
OpenSSL version: OpenSSL 1.0.1e 11 Feb 2013
[root@docker ~]# ./docker-compose-Linux-x86_64 --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE           Specify an alternate compose file (default: docker-compose.yml)
  -p, --project-name NAME   Specify an alternate project name (default: directory name)
  --x-networking            (EXPERIMENTAL) Use new Docker networking functionality.
                            Requires Docker 1.9 or later.
  --x-network-driver DRIVER (EXPERIMENTAL) Specify a network driver (default: "bridge").
                            Requires Docker 1.9 or later.
  --verbose                 Show more output
  -v, --version             Print version and exit

Commands:
  build              Build or rebuild services
  help               Get help on a command
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pulls service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  unpause            Unpause services
  up                 Create and start containers
  migrate-to-labels  Recreate containers to add labels
  version            Show the Docker-Compose version information
[root@docker ~]# 
~~~

---

### 编辑docker-compose.yml


~~~
[root@docker ~]# ls
anaconda-ks.cfg  auth  certs  data  docker-compose-Linux-x86_64  docker-compose.yml  dockerfile
[root@docker ~]# vim docker-compose.yml 
[root@docker ~]# cat docker-compose.yml 
registry2:
  restart:always
  image:registry:2
  ports: 
    - 5002:5002
  environment:
    REGISTRY_AUTH:htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM:Registry Realm
    REGISTRY_AUTH_HTPASSWD_PATH:/auth/htpasswd
    REGISTRY_HTTP_TLS_CERTIFICATE:/certs/docker.crt
    REGISTRY_HTTP_TLS_KEY:/certs/docker.key
  volumes:
    - /root/data:/var/lib/registry
    - /root/certs:/certs
    - /root/auth:/auth
[root@docker ~]# ./docker-compose-Linux-x86_64 up -d 
ERROR: yaml.scanner.ScannerError: mapping values are not allowed here
  in "./docker-compose.yml", line 4, column 8
[root@docker ~]# 
~~~

#### 报错1

* 原因是 docker-compose.yml 中格式不对
* 解决办法调整格式，加上空格

> **Tip:** 属性后面的值与 **`:`** 之间要有空格 
> 
> **`restart:always`**  是错的
> 
> **`restart: always`**  是对的

---

~~~
[root@docker ~]# vim docker-compose.yml 
[root@docker ~]# cat docker-compose.yml 
registry2:
  restart: always
  image: registry:2
  ports: 
    - 5002:5002
  environment:
    REGISTRY_AUTH:htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM:Registry Realm
    REGISTRY_AUTH_HTPASSWD_PATH:/auth/htpasswd
    REGISTRY_HTTP_TLS_CERTIFICATE:/certs/docker.crt
    REGISTRY_HTTP_TLS_KEY:/certs/docker.key
  volumes:
    - /root/data:/var/lib/registry
    - /root/certs:/certs
    - /root/auth:/auth
[root@docker ~]# ./docker-compose-Linux-x86_64 up -d 
ERROR: Validation failed in file './docker-compose.yml', reason(s):
Service 'registry2' configuration key 'environment' contains an invalid type, it should be an object, or an array
[root@docker ~]# 
~~~


#### 报错2

* 原因是 docker-compose.yml 中environment部分格式不对
* 解决办法：调整格式，加上空格

> **Tip:** environment属性后面的值与 **`:`** 之间要有空格 
> 
> **`REGISTRY_AUTH:htpasswd`**  是错的
> 
> **`REGISTRY_AUTH: htpasswd`**  是对的


---

~~~
[root@docker ~]# vim docker-compose.yml 
[root@docker ~]# cat docker-compose.yml 
registry2:
  restart: always
  image: registry:2
  ports: 
    - 5002:5002
  environment:
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/docker.crt
    REGISTRY_HTTP_TLS_KEY: /certs/docker.key
  volumes:
    - /root/data:/var/lib/registry
    - /root/certs:/certs
    - /root/auth:/auth
[root@docker ~]# ./docker-compose-Linux-x86_64 up -d 
Creating root_registry2_1
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                              NAMES
e870d0a4b904        registry:2          "/bin/registry /etc/d"   7 seconds ago       Up 6 seconds             5000/tcp, 0.0.0.0:5002->5002/tcp   root_registry2_1
71de3ba93794        registry:2          "/bin/registry /etc/d"   4 hours ago         Up 4 hours               0.0.0.0:5000->5000/tcp             registry
3d9f0915226f        registry:2          "htpasswd -Bbn testus"   4 hours ago         Exited (0) 4 hours ago                                      prickly_jang
27995af3fa59        registry:2          "htpasswd -Bbn testus"   7 hours ago         Exited (0) 7 hours ago                                      gloomy_goldberg
[root@docker ~]# 
----------
[root@h104 ~]# docker push docker:5002/ubuntu
The push refers to a repository [docker:5002/ubuntu] (len: 1)
unable to ping registry endpoint https://docker:5002/v0/
v2 ping attempt failed with error: Get https://docker:5002/v2/: dial tcp 192.168.100.103:5002: connection refused
 v1 ping attempt failed with error: Get https://docker:5002/v1/_ping: dial tcp 192.168.100.103:5002: connection refused
[root@h104 ~]# nmap docker

Starting Nmap 6.40 ( http://nmap.org ) at 2016-01-22 20:13 CST
Nmap scan report for docker (192.168.100.103)
Host is up (0.00079s latency).
rDNS record for 192.168.100.103: h103
Not shown: 994 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   closed http
3306/tcp closed mysql
5000/tcp open   upnp
5002/tcp closed rfe
8080/tcp closed http-proxy
MAC Address: 00:0C:29:B6:CC:BA (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.00 seconds
[root@h104 ~]# 
~~~

#### 报错3

* 原因是配置中端口映射不对
* 解决办法：调整port map 为 5002:5000

> **Tip:** **`5000/tcp, 0.0.0.0:5002->5002/tcp`** 意味着容器里监听了5000端口，但是主机与容器的端口映射为5002外->5002内
> 
> 产生问题的根本原因就是容器里并没有监听在5002，所以无法提供服务
> 
> 只要进行正确映射就可以解决问题




~~~
WRONG
        "Ports": {
            "5000/tcp": null,
            "5002/tcp": [
                {
                    "HostIp": "0.0.0.0",
                    "HostPort": "5002"
                }
            ]
        }
----------
RIGHT
        "Ports": {
            "5000/tcp": [
                {
                    "HostIp": "0.0.0.0",
                    "HostPort": "5002"
                }
            ]
        }
~~~



---



~~~
[root@docker ~]# docker stop e870d0a4b904 && docker rm -v e870d0a4b904
e870d0a4b904
e870d0a4b904
[root@docker ~]# vim docker-compose.yml
[root@docker ~]# cat docker-compose.yml 
registry2:
  restart: always
  image: registry:2
  ports: 
    - 5002:5000
  environment:
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/docker.crt
    REGISTRY_HTTP_TLS_KEY: /certs/docker.key
  volumes:
    - /root/data:/var/lib/registry
    - /root/certs:/certs
    - /root/auth:/auth
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                    NAMES
71de3ba93794        registry:2          "/bin/registry /etc/d"   4 hours ago         Up 4 hours               0.0.0.0:5000->5000/tcp   registry
3d9f0915226f        registry:2          "htpasswd -Bbn testus"   4 hours ago         Exited (0) 4 hours ago                            prickly_jang
27995af3fa59        registry:2          "htpasswd -Bbn testus"   7 hours ago         Exited (0) 7 hours ago                            gloomy_goldberg
[root@docker ~]# 
[root@docker ~]# ./docker-compose-Linux-x86_64 up -d 
Creating root_registry2_1
[root@docker ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                    NAMES
b9ef0f189068        registry:2          "/bin/registry /etc/d"   6 seconds ago       Up 4 seconds             0.0.0.0:5002->5000/tcp   root_registry2_1
71de3ba93794        registry:2          "/bin/registry /etc/d"   4 hours ago         Up 4 hours               0.0.0.0:5000->5000/tcp   registry
3d9f0915226f        registry:2          "htpasswd -Bbn testus"   4 hours ago         Exited (0) 4 hours ago                            prickly_jang
27995af3fa59        registry:2          "htpasswd -Bbn testus"   7 hours ago         Exited (0) 7 hours ago                            gloomy_goldberg
[root@docker ~]#
----------
[root@h104 ~]# docker push docker:5002/ubuntu
The push refers to a repository [docker:5002/ubuntu] (len: 1)

Head https://docker:5002/v2/ubuntu/blobs/sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4: no basic auth creden
[root@h104 ~]# docker login docker:5002
Username: testuser
Password: 
Email: yyghdfz@163.com
WARNING: login credentials saved in /root/.docker/config.json
Login Succeeded
[root@h104 ~]# docker push docker:5002/ubuntu
The push refers to a repository [docker:5002/ubuntu] (len: 1)
8693db7e8a00: Pushed 
a4c5be5b6e59: Pushed 
c4fae638e7ce: Pushed 
f15ce52fc004: Pushed 
latest: digest: sha256:45d78ef16a9e6199ffbbc78f71c2c6ef6647f3be6b9721fe3f1b08d6e3fcf6b3 size: 6800
[root@h104 ~]# 
~~~

现在一切正常


> **Tip:** 直接使用 **`docker-compose-Linux-x86_64 up -d`** 时并未指定配置文件， 但其实它在隐性调用当前目录中的 **docker-compose.yml** 文件，这个和 **Dockerfile** 有相似之处，但是可以使用 **`-f, --file FILE`** 参数来覆盖




---

# 命令汇总

~~~
docker run -d -p 5000:5000 --name registry registry:2
docker pull ubuntu
docker tag ubuntu localhost:5000/myfirstimage
docker images
docker push localhost:5000/myfirstimage
docker pull localhost:5000/myfirstimage
docker  ps -a
docker stop 7716d7899161
docker rm 7716d7899161
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker pull ubuntu &&  docker tag ubuntu localhost:5000/ubuntu
docker push localhost:5000/ubuntu
docker pull localhost:5000/ubuntu
docker stop registry && docker rm -v registry
echo `pwd`
docker  run -d -p 5000:5000 --restart=always --name registry -v `pwd`/data:/var/lib/registry registry:2
tree data/
openssl genrsa -out docker.key 2048
openssl req -new -key docker.key -out docker.csr
openssl x509 -req -days 365 -in docker.csr -signkey docker.key -out docker.crt
chmod 600 *
docker run -d -p 5000:5000 --restart=always --name registry  -v `pwd`/data:/var/lib/registry  -v `pwd`/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker.crt -e REGISTRY_HTTP_TLS_KEY=/certs/docker.key registry:2
docker tag ubuntu 192.168.100.104:5000/ubuntu
docker tag ubuntu h104:5000/ubuntu
docker tag ubuntu docker-registry:5000/ubuntu
docker push h104:5000/ubuntu
docker push 192.168.100.104:5000/ubuntu
docker push h104:5000/ubuntu
vim /etc/hosts
grep docker-registry  /etc/hosts
docker push docker-registry:5000/ubuntu
ll /etc/pki/tls/certs/ca-bundle.crt
ll /etc/pki/ca-trust/extracted/pem/
scp root@h104:/root/certs/docker.crt /etc/pki/ca-trust/extracted/pem/
cat /etc/pki/ca-trust/extracted/pem/docker.crt >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
systemctl stop docker && systemctl start docker
docker push docker-registry:5000/ubuntu
docker pull  docker-registry:5000/ubuntu
scp root@h104:/root/certs/docker.crt  /etc/pki/ca-trust/source/anchors/
ll /etc/pki/ca-trust/source/anchors/
update-ca-trust
systemctl stop docker && systemctl start docker
docker push docker-registry:5000/ubuntu
docker pull docker-registry:5000/ubuntu
systemctl list-dependencies docker.service | head -n 10
firewall-cmd --reload
docker push docker:5000/ubuntu
systemctl stop docker && systemctl  start docker
iptables -L -nv | grep -i docker
docker push docker:5000/ubuntu
mkdir auth
docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
cat auth/htpasswd
docker run -d -p 5000:5000 --restart=always --name registry \
docker push docker:5000/ubuntu
docker login docker:5000
docker push docker:5000/ubuntu
curl -L https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m` > docker-compose
wget https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m`
file docker-compose-Linux-x86_64
chmod +x docker-compose-Linux-x86_64
./docker-compose-Linux-x86_64 version
./docker-compose-Linux-x86_64 --help
vim docker-compose.yml
cat docker-compose.yml
./docker-compose-Linux-x86_64 up -d
nmap docker
docker login docker:5002
docker push docker:5002/ubuntu
~~~


---

[docker]:https://www.docker.com/
[registry]:https://docs.docker.com/registry/
[compose]:https://www.docker.com/docker-compose
[compose_install]:https://docs.docker.com/compose/install/
[compose_doc]:https://docs.docker.com/compose/
