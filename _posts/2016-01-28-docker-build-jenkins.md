---
layout: post
title:  Docker 中构建 Jenkins
author: wilmosfang
tags:   docker jenkins
categories:   docker
wc: 478 1595 17315
excerpt: 自定义Jenkins镜像，认证登录registry，推送拉取镜像，容器中运行镜像，访问jenkins
comments: true
---



# 前言

**[Docker][docker]** 与 **[Jenkins][jenkins]** 经常会放到一起构建 **CI** (持续集成)系统


这里结合Docker Registry 分享一下在Docker中构建 Jenkins 容器的相关操作，详细可以参阅 **[官方文档][jenkins_build]** 

> **Tip:** 当前的最新版本为 **Docker 1.10** Released on January 15, 2016

---


# 概要

* TOC
{:toc}



---


## 自定义Jenkins镜像


### 准备构建环境

在构建环境中准备相应的证书文件和插件信息


~~~
[root@docker docker]# mkdir build && cd build
[root@docker build]# pwd
/root/docker/build
[root@docker build]# vim plugins
[root@docker build]# cat plugins 
role-strategy:2.2.0
[root@docker build]# cp ../../certs/docker.* . 
[root@docker build]# ll 
total 16
-rw------- 1 root root 1281 Jan 27 13:52 docker.crt
-rw------- 1 root root 1045 Jan 27 13:52 docker.csr
-rw------- 1 root root 1679 Jan 27 13:52 docker.key
-rw-r--r-- 1 root root   20 Jan 27 13:51 plugins
[root@docker build]# 
~~~

> **Tip:** 这里我使用的自签名证书

---

### 创建Dockerfile


~~~
[root@docker build]# vim Dockerfile
[root@docker build]# cat Dockerfile 
FROM jenkins

#New plugins must be placed in the plugins file
COPY plugins /usr/share/jenkins/plugins

#The plugins.sh script will install new plugins
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins

#Copy private key and cert to image
COPY docker.crt /var/lib/jenkins/cert
COPY docker.key /var/lib/jenkins/pk

#Configure HTTP off and HTTPS on, using port 1973
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=1973 --httpsCertificate=/var/lib/jenkins/cert --httpsPrivateKey=/var/lib/jenkins/pk
[root@docker build]# ll 
total 20
-rw------- 1 root root 1281 Jan 27 13:52 docker.crt
-rw------- 1 root root 1045 Jan 27 13:52 docker.csr
-rw-r--r-- 1 root root  497 Jan 27 14:13 Dockerfile
-rw------- 1 root root 1679 Jan 27 13:52 docker.key
-rw-r--r-- 1 root root   20 Jan 27 13:51 plugins
[root@docker build]# 
~~~


> **Note:** **Dockerfile plugins** 还有两个证书文件 (**docker.crt and docker.key**) 必须在同一个目录里，包含 **Dockerfile** 的目录叫作构建环境，文件只有放在构建环境中才能在构建过程中被集成进去



---

### 构建镜像


~~~
[root@docker build]# docker  build -t  ci-infrastructure/jnkns-img  . 
Sending build context to Docker daemon 9.728 kB
Step 1 : FROM jenkins
 ---> fc39417bd5fb
Step 2 : COPY plugins /usr/share/jenkins/plugins
 ---> 0139ec49d08d
Removing intermediate container a9f6f1ead720
Step 3 : RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins
 ---> Running in 1cab523a28c9
Downloading role-strategy:2.2.0
 ---> b65ed720a699
Removing intermediate container 1cab523a28c9
Step 4 : COPY docker.crt /var/lib/jenkins/cert
 ---> aa2257cc683d
Removing intermediate container 7f72a7d983b1
Step 5 : COPY docker.key /var/lib/jenkins/pk
 ---> 0e692408d048
Removing intermediate container b0bfe48b921b
Step 6 : ENV JENKINS_OPTS --httpPort=-1 --httpsPort=1973 --httpsCertificate=/var/lib/jenkins/cert --httpsPrivateKey=/var/lib/jenkins/pk
 ---> Running in 7ae0ddd5277e
 ---> c384253964b5
Removing intermediate container 7ae0ddd5277e
Successfully built c384253964b5
[root@docker build]# echo $?
0
[root@docker build]# docker images 
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ci-infrastructure/jnkns-img   latest              c384253964b5        15 seconds ago      708.2 MB
<none>                        <none>              d34c985f0918        15 hours ago        292.7 MB
<none>                        <none>              9287fae7a16e        32 hours ago        169.3 MB
ubuntu                        latest              8693db7e8a00        7 days ago          187.9 MB
docker-registry:5000/ubuntu   latest              8693db7e8a00        7 days ago          187.9 MB
docker:5000/ubuntu            latest              8693db7e8a00        7 days ago          187.9 MB
h103:5000/ubuntu              latest              8693db7e8a00        7 days ago          187.9 MB
localhost:5000/ubuntu         latest              8693db7e8a00        7 days ago          187.9 MB
192.168.100.104:5000/ubuntu   latest              8693db7e8a00        7 days ago          187.9 MB
h104:5000/ubuntu              latest              8693db7e8a00        7 days ago          187.9 MB
localhost:5000/myfirstimage   latest              8693db7e8a00        7 days ago          187.9 MB
jenkins                       latest              fc39417bd5fb        2 weeks ago         708.1 MB
registry                      2                   683f9cd9cf88        3 weeks ago         224.5 MB
hello-world                   latest              0a6ba66e537a        3 months ago        960 B
[root@docker build]# 
~~~



---

## 推送镜像

~~~
[root@docker build]# docker tag ci-infrastructure/jnkns-img  docker:5000/ci/jnkns-img
[root@docker build]# docker push docker:5000/ci/jnkns-img 
The push refers to a repository [docker:5000/ci/jnkns-img] (len: 1)
unable to ping registry endpoint https://docker:5000/v0/
v2 ping attempt failed with error: Get https://docker:5000/v2/: x509: certificate signed by unknown authority
 v1 ping attempt failed with error: Get https://docker:5000/v1/_ping: x509: certificate signed by unknown authority
[root@docker build]# cd /root/certs
[root@docker certs]# ls
docker.crt  docker.csr  docker.key
[root@docker certs]# cat docker.crt  >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
[root@docker certs]# systemctl stop docker && systemctl start docker
[root@docker certs]# docker push docker:5000/ci/jnkns-img 
The push refers to a repository [docker:5000/ci/jnkns-img] (len: 1)

Post https://docker:5000/v2/ci/jnkns-img/blobs/uploads/: no basic auth credentials
[root@docker certs]# docker login docker:5000
Username: testuser
Password: 
Email: yyghdfz@163.com
WARNING: login credentials saved in /root/.docker/config.json
Login Succeeded
[root@docker certs]# docker push docker:5000/ci/jnkns-img 
The push refers to a repository [docker:5000/ci/jnkns-img] (len: 1)
c384253964b5: Pushed 
0e692408d048: Pushed 
aa2257cc683d: Pushed 
b65ed720a699: Pushed 
0139ec49d08d: Pushed 
fc39417bd5fb: Pushed 
55422ac36eba: Pushed 
b48f4074fc73: Pushed 
53e20479e6a7: Pushed 
585059426ec6: Pushed 
6234bb424ca2: Pushed 
b31b78b6c124: Pushed 
7e844a128314: Pushed 
6842d0a24c05: Pushed 
9afbe4c3ddc8: Pushed 
ff135e80b6aa: Pushed 
05e608b5b672: Pushed 
b12dfca65359: Pushed 
4ee671494b6b: Pushed 
ce2b29af7753: Pushed 
5c63804eac90: Pushed 
523ef1d23f22: Pushed 
latest: digest: sha256:613ef35ff2fff0a26bab66dd9213463b034d4e536e9a6d52cbaeacb767fdf828 size: 87506
[root@docker certs]# 
~~~

推送过程中要注意的地方:

* 确保Registry地址没错，如果有问题可以使用 **`docker tag`** 来调整
* 确保有证书，如果没有，要先导入，然后重启docker
* 确保进行了基础认证，如果没有要进行认证(在没有基础认证的Registry中不必关心这一点)

---


## 拉取镜像


可以使用其它的机器通过 **docker pull** 来测试一下上传的镜像

~~~
[root@h104 certs]# docker pull docker:5000/ci/jnkns-img 
Using default tag: latest
latest: Pulling from ci/jnkns-img
98f6335f9102: Pull complete 
efae71df8aca: Pull complete 
007c4719623e: Pull complete 
0e54d32ff5f2: Pull complete 
5b825467fc4f: Pull complete 
Digest: sha256:613ef35ff2fff0a26bab66dd9213463b034d4e536e9a6d52cbaeacb767fdf828
Status: Downloaded newer image for docker:5000/ci/jnkns-img:latest
[root@h104 certs]#
[root@h104 certs]# docker images 
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker:5000/ci/jnkns-img      latest              5b825467fc4f        35 minutes ago      708.2 MB
ubuntu                        latest              8693db7e8a00        7 days ago          187.9 MB
192.168.100.103:5000/ubuntu   latest              8693db7e8a00        7 days ago          187.9 MB
docker:5000/ubuntu            latest              8693db7e8a00        7 days ago          187.9 MB
docker:5002/ubuntu            latest              8693db7e8a00        7 days ago          187.9 MB
h103:5000/ubuntu              latest              8693db7e8a00        7 days ago          187.9 MB
localhost:5000/myfirstimage   latest              8693db7e8a00        7 days ago          187.9 MB
localhost:5000/ubuntu         latest              8693db7e8a00        7 days ago          187.9 MB
jenkins                       latest              fc39417bd5fb        2 weeks ago         708.1 MB
registry                      2                   683f9cd9cf88        3 weeks ago         224.5 MB
hello-world                   latest              0a6ba66e537a        3 months ago        960 B
[root@h104 certs]# 
~~~


---

## 通过镜像运行容器

~~~
[root@h104 ~]#  docker run -p 1973:1973 --name jenkins01 docker:5000/ci/jnkns-img 
Running from: /usr/share/jenkins/jenkins.war
webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")
Jan 27, 2016 1:27:39 PM winstone.Logger logInternal
INFO: Beginning extraction from war file
Jan 27, 2016 1:27:41 PM winstone.Logger logInternal
INFO: Winstone shutdown successfully
Jan 27, 2016 1:27:41 PM winstone.Logger logInternal
SEVERE: Container startup failed
java.io.IOException: Failed to start a listener: winstone.HttpsConnectorFactory
	at winstone.Launcher.spawnListener(Launcher.java:209)
	at winstone.Launcher.<init>(Launcher.java:149)
	at winstone.Launcher.main(Launcher.java:354)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at Main._main(Main.java:293)
	at Main.main(Main.java:98)
Caused by: java.io.FileNotFoundException: /var/lib/jenkins/cert (Permission denied)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at winstone.HttpsConnectorFactory.start(HttpsConnectorFactory.java:88)
	at winstone.Launcher.spawnListener(Launcher.java:207)
	... 8 more

[root@h104 ~]# 
~~~

### 报错

出现了报错

通过官方的文档，和docker hub中的说明没有找到根本原因

通过google，有人使用keystore解决了这个bug

暂时不使用https，降级构建Dockerfile (去掉https会丢失安全性，之后再回头慢慢研究原因)

注释掉https的相关配置，然后再构建镜像

~~~
[root@docker build]# vim Dockerfile 
[root@docker build]# cat Dockerfile 
FROM jenkins

#New plugins must be placed in the plugins file
COPY plugins /usr/share/jenkins/plugins

#The plugins.sh script will install new plugins
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins

#Copy private key and cert to image
#COPY docker.crt /var/lib/jenkins/cert
#COPY docker.key /var/lib/jenkins/pk

#Configure HTTP off and HTTPS on, using port 1973
#ENV JENKINS_OPTS --httpPort=-1 --httpsPort=1973 --httpsCertificate=/var/lib/jenkins/cert --httpsPrivateKey=/var/lib/jenkins/pk

[root@docker build]# 
[root@docker build]#  docker build -t test/jnkns-img .
Sending build context to Docker daemon 9.728 kB
Step 1 : FROM jenkins
 ---> fc39417bd5fb
Step 2 : COPY plugins /usr/share/jenkins/plugins
 ---> Using cache
 ---> 0139ec49d08d
Step 3 : RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins
 ---> Using cache
 ---> b65ed720a699
Successfully built b65ed720a699
[root@docker build]#
[root@docker build]# docker tag test/jnkns-img docker:5000/ci/jnkns-img2
[root@docker build]# docker push docker:5000/ci/jnkns-img2
The push refers to a repository [docker:5000/ci/jnkns-img2] (len: 1)
b65ed720a699: Pushed 
0139ec49d08d: Pushed 
fc39417bd5fb: Pushed 
0c27fdb0b33b: Pushed 
55422ac36eba: Pushed 
b48f4074fc73: Pushed 
53e20479e6a7: Pushed 
585059426ec6: Pushed 
6234bb424ca2: Pushed 
b31b78b6c124: Pushed 
7e844a128314: Pushed 
6842d0a24c05: Pushed 
9afbe4c3ddc8: Pushed 
ff135e80b6aa: Pushed 
05e608b5b672: Pushed 
b12dfca65359: Pushed 
4ee671494b6b: Pushed 
ce2b29af7753: Pushed 
5c63804eac90: Pushed 
523ef1d23f22: Pushed 
latest: digest: sha256:b0593124c0f6790329649ae4863bb0c1d66b9aa32c9ce260c505a9ccfd185bd2 size: 78740
[root@docker build]# 
~~~


再次构建,构建前要使用 **docker rm** 删掉之前构建失败的容器，或者新容器换个名字，否则会有冲突

~~~
[root@h104 ~]# docker run -p 8080:8080 --name jenkins01 docker:5000/ci/jnkns-img2
Running from: /usr/share/jenkins/jenkins.war
webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")
Jan 27, 2016 1:45:57 PM winstone.Logger logInternal
INFO: Beginning extraction from war file
Jan 27, 2016 1:45:59 PM org.eclipse.jetty.util.log.JavaUtilLog info
INFO: jetty-winstone-2.8
Jan 27, 2016 1:46:01 PM org.eclipse.jetty.util.log.JavaUtilLog info
INFO: NO JSP Support for , did not find org.apache.jasper.servlet.JspServlet
Jenkins home directory: /var/jenkins_home found at: EnvVars.masterEnvVars.get("JENKINS_HOME")
Jan 27, 2016 1:46:03 PM org.eclipse.jetty.util.log.JavaUtilLog info
INFO: Started SelectChannelConnector@0.0.0.0:8080
Jan 27, 2016 1:46:03 PM winstone.Logger logInternal
INFO: Winstone Servlet Engine v2.0 running: controlPort=disabled
Jan 27, 2016 1:46:04 PM jenkins.InitReactorRunner$1 onAttained
INFO: Started initialization
Jan 27, 2016 1:46:20 PM jenkins.InitReactorRunner$1 onAttained
INFO: Listed all plugins
Jan 27, 2016 1:46:20 PM jenkins.InitReactorRunner$1 onAttained
INFO: Prepared all plugins
Jan 27, 2016 1:46:20 PM jenkins.InitReactorRunner$1 onAttained
INFO: Started all plugins
Jan 27, 2016 1:46:20 PM jenkins.InitReactorRunner$1 onAttained
INFO: Augmented all extensions
Jan 27, 2016 1:46:27 PM jenkins.InitReactorRunner$1 onAttained
INFO: Loaded all jobs
Jan 27, 2016 1:46:28 PM hudson.model.AsyncPeriodicWork$1 run
INFO: Started Download metadata
Jan 27, 2016 1:46:28 PM jenkins.util.groovy.GroovyHookScript execute
INFO: Executing /var/jenkins_home/init.groovy.d/tcp-slave-agent-port.groovy
Jan 27, 2016 1:46:28 PM org.jenkinsci.main.modules.sshd.SSHD start
INFO: Started SSHD at port 47557
Jan 27, 2016 1:46:29 PM jenkins.InitReactorRunner$1 onAttained
INFO: Completed initialization
Jan 27, 2016 1:46:29 PM hudson.WebAppMain$3 run
INFO: Jenkins is fully up and running
--> setting agent port for jnlp
--> setting agent port for jnlp... done
Jan 27, 2016 1:46:45 PM hudson.model.UpdateSite updateData
INFO: Obtained the latest update center data file for UpdateSource default
Jan 27, 2016 1:46:46 PM hudson.model.DownloadService$Downloadable load
INFO: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
Jan 27, 2016 1:46:48 PM hudson.model.DownloadService$Downloadable load
INFO: Obtained the updated data file for hudson.tasks.Ant.AntInstaller
Jan 27, 2016 1:47:16 PM hudson.model.DownloadService$Downloadable load
INFO: Obtained the updated data file for hudson.tools.JDKInstaller
Jan 27, 2016 1:47:16 PM hudson.model.AsyncPeriodicWork$1 run
INFO: Finished Download metadata. 48,013 ms
...
...
~~~

另开一个窗口

~~~
[root@h104 ~]# docker ps -a 
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                               NAMES
236348a3c9ff        docker:5000/ci/jnkns-img2   "/bin/tini -- /usr/lo"   7 minutes ago       Up 7 minutes        0.0.0.0:8080->8080/tcp, 50000/tcp   jenkins01
[root@h104 ~]# netstat  -ant | grep 80
tcp6       0      0 :::8080                 :::*                    LISTEN     
[root@h104 ~]# 
~~~


---

## 访问Jenkins


![jenkins1.png](/images/registry/jenkins1.png)


通过下面步骤进入已安装插件列表

**[系统管理]->[管理插件]->[已安装]**

可以看到 **Role-based Authorization Strategy** 插件，版本和我们指定的一样

~~~
[root@docker build]# cat plugins 
role-strategy:2.2.0
[root@docker build]# 
~~~

![jenkins2.png](/images/registry/jenkins2.png)




---

# 命令汇总

* **`cat plugins`**
* **`vim Dockerfile`**
* **`cat Dockerfile`**
* **`docker  build -t  ci-infrastructure/jnkns-img  .`**
* **`docker images`**
* **`docker tag ci-infrastructure/jnkns-img  docker:5000/ci/jnkns-img`**
* **`docker push docker:5000/ci/jnkns-img`**
* **`cat docker.crt  >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem`**
* **`systemctl stop docker && systemctl start docker`**
* **`docker login docker:5000`**
* **`docker push docker:5000/ci/jnkns-img`**
* **`docker pull docker:5000/ci/jnkns-img`**
* **`docker run -p 1973:1973 --name jenkins01 docker:5000/ci/jnkns-img`**
* **`vim Dockerfile`**
* **`cat Dockerfile`**
* **`docker build -t test/jnkns-img .`**
* **`docker tag test/jnkns-img docker:5000/ci/jnkns-img2`**
* **`docker push docker:5000/ci/jnkns-img2`**
* **`docker run -p 8080:8080 --name jenkins01 docker:5000/ci/jnkns-img2`**



---

[docker]:https://www.docker.com/
[jenkins]:http://jenkins-ci.org/
[jenkins_build]:https://docs.docker.com/docker-trusted-registry/quick-start/

