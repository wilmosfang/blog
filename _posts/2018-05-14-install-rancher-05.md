---
layout: post
title: "Install Rancher 5"
author:  wilmosfang
date: 2018-05-16 12:38:26
image: '/assets/img/'
excerpt: '通过 Rancher RKE 安装 k8s 集群'
main-class: 'rancher'
color: '#2980b8'
tags:
 - docker
 - rancher
 - rke
 - kubernetes
categories: 
 - rancher
twitter_text: 'simple process of installation of k8s by RKE'
introduction: 'Installation of k8s by RKE'
---

# 前言

**[Rancher][rancher]** 是一款开源的容器管理软件

**[Rancher][rancher]** 的设计目标的是简化容器的管理操作，提升容器应用的操作效率

因为整合了 k8s 的编排功能, 并且有着非常友好的操作界面，所以在目前的容器技术圈中有着很大的影响力

如果要快速构建一套 **CI/CD** 发布平台， **[Rancher][rancher]** 是一个不错的选择

这里基于前面的工作，演示一下如何通过 RKE 工具构建一个 k8s 集群

参考 **[如何安装RKE][rancher_rke]** 和 **[An Introduction To Rancher Kubernetes Engine (RKE)][introduction_rke]**

> **Tip:** 当前的版本为 **Rancher 2.0 GA** 和 **rke v0.1.7-rc4**

---

## 运行环境

~~~
bolo@rancher:~$ hostnamectl
   Static hostname: rancher
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4bae573cc9bc7f877992f1885ae0c42b
           Boot ID: 6b0d45eadd73407d9716a77f1c9e1486
    Virtualization: oracle
  Operating System: Ubuntu 16.04.4 LTS
            Kernel: Linux 4.4.0-116-generic
      Architecture: x86-64
bolo@rancher:~$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:87:b2:4d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.5/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe87:b24d/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ae:1b:a6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.152/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feae:1ba6/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:9e:19:48:8e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:9eff:fe19:488e/64 scope link 
       valid_lft forever preferred_lft forever
10: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether ba:2b:f8:4a:f1:dc brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::b82b:f8ff:fe4a:f1dc/64 scope link 
       valid_lft forever preferred_lft forever
83: veth7c2720d@if82: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether e2:6b:78:79:c7:61 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::e06b:78ff:fe79:c761/64 scope link 
       valid_lft forever preferred_lft forever
bolo@rancher:~$ dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           17.03.2~ce-0~ubuntu-xenial                 amd64        Docker: the open-source application container engine
bolo@rancher:~$ docker version
Client:
 Version:      17.03.2-ce
 API version:  1.27
 Go version:   go1.7.5
 Git commit:   f5ec1e2
 Built:        Tue Jun 27 03:35:14 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.2-ce
 API version:  1.27 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   f5ec1e2
 Built:        Tue Jun 27 03:35:14 2017
 OS/Arch:      linux/amd64
 Experimental: false
bolo@rancher:~$ 
~~~



## 给 user 赋权

~~~
root@rancher:~# su - bolo
bolo@rancher:~$ docker ps -a 
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.27/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
bolo@rancher:~$ id 
uid=1000(bolo) gid=1000(bolo) groups=1000(bolo),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
bolo@rancher:~$ sudo su - 
root@rancher:~# id bolo
uid=1000(bolo) gid=1000(bolo) groups=1000(bolo),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
root@rancher:~# usermod -aG docker bolo
root@rancher:~# id bolo
uid=1000(bolo) gid=1000(bolo) groups=1000(bolo),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),999(docker)
root@rancher:~# su - bolo
bolo@rancher:~$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bolo@rancher:~$
~~~

此操作要确保在以下三台服务器上都有权限执行

* 192.168.56.152
* 192.168.56.153
* 192.168.56.154

也就是以上三台服务器的 bolo 用户都得分配在 docker 组中，都得具备对于 docker 的操作权限

## 配置 user 互访

~~~
bolo@rancher:~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bolo/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/bolo/.ssh/id_rsa.
Your public key has been saved in /home/bolo/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:azlXFhW33H1oaNXmPzjrTJbb3hBWSginqu1yHBv+Rdk bolo@rancher
The key's randomart image is:
+---[RSA 2048]----+
|          . . +o.|
|           + =.o*|
|          . = +==|
|         . . * oo|
|        S   = E .|
|       ooo + +.o.|
|      .o*+. .++ .|
|      .o=o .+.o..|
|       o... .+.o.|
+----[SHA256]-----+
bolo@rancher:~$ 
bolo@rancher:~$ ssh-copy-id -i bolo@192.168.56.153
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/bolo/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.153 (192.168.56.153)' can't be established.
ECDSA key fingerprint is SHA256:R2LjKHhqC2SPVRzTeLkg/qySIgcKgfnNHxwqXNJcBYI.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bolo@192.168.56.153's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bolo@192.168.56.153'"
and check to make sure that only the key(s) you wanted were added.

bolo@rancher:~$ ssh 'bolo@192.168.56.153'
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

17 packages can be updated.
9 updates are security updates.


Last login: Tue May 15 00:25:46 2018 from 192.168.56.1
bolo@node153:~$ exit
logout
Connection to 192.168.56.153 closed.
bolo@rancher:~$ ssh-copy-id -i bolo@192.168.56.154
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/bolo/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.154 (192.168.56.154)' can't be established.
ECDSA key fingerprint is SHA256:R2LjKHhqC2SPVRzTeLkg/qySIgcKgfnNHxwqXNJcBYI.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bolo@192.168.56.154's password: 
Permission denied, please try again.
bolo@192.168.56.154's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bolo@192.168.56.154'"
and check to make sure that only the key(s) you wanted were added.

bolo@rancher:~$ ssh 'bolo@192.168.56.154'
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

17 packages can be updated.
9 updates are security updates.


Last login: Mon May 14 22:25:59 2018 from 192.168.56.1
bolo@node154:~$ exit
logout
Connection to 192.168.56.154 closed.
bolo@rancher:~$ ssh-copy-id -i bolo@192.168.56.152
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/bolo/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.152 (192.168.56.152)' can't be established.
ECDSA key fingerprint is SHA256:R2LjKHhqC2SPVRzTeLkg/qySIgcKgfnNHxwqXNJcBYI.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bolo@192.168.56.152's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bolo@192.168.56.152'"
and check to make sure that only the key(s) you wanted were added.

bolo@rancher:~$ ssh 'bolo@192.168.56.152'
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

17 packages can be updated.
9 updates are security updates.


Last login: Mon May 14 23:46:24 2018 from 192.168.56.1
bolo@rancher:~$ exit
logout
Connection to 192.168.56.152 closed.
bolo@rancher:~$ 
~~~

因为 RKE 是基于 ssh 进行操作的，所以首先要打通 ssh 的访问


## 打开防火墙

~~~
root@rancher:~# ufw status
Status: inactive
root@rancher:~# ufw enable 
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@rancher:~# ufw status
Status: active
root@rancher:~# ufw allow 6443
Rule added
Rule added (v6)
root@rancher:~# ufw allow 2379
Rule added
Rule added (v6)
root@rancher:~# ufw allow 2380
Rule added
Rule added (v6)
root@rancher:~# ufw allow 22
Rule added
Rule added (v6)
root@rancher:~# ufw status
Status: active

To                         Action      From
--                         ------      ----
6443                       ALLOW       Anywhere                  
2379                       ALLOW       Anywhere                  
2380                       ALLOW       Anywhere                  
22                         ALLOW       Anywhere                  
6443 (v6)                  ALLOW       Anywhere (v6)             
2379 (v6)                  ALLOW       Anywhere (v6)             
2380 (v6)                  ALLOW       Anywhere (v6)             
22 (v6)                    ALLOW       Anywhere (v6)             

root@rancher:~# 
~~~

至少 **6443/2379/2380/22** 这些端口要打开

当然，如果运行 rancher 那 **80/443** 也是要放开的

~~~
bolo@node153:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
6443                       ALLOW       Anywhere                  
2379                       ALLOW       Anywhere                  
2380                       ALLOW       Anywhere                  
22                         ALLOW       Anywhere                  
6443 (v6)                  ALLOW       Anywhere (v6)             
2379 (v6)                  ALLOW       Anywhere (v6)             
2380 (v6)                  ALLOW       Anywhere (v6)             
22 (v6)                    ALLOW       Anywhere (v6)             

bolo@node153:~$
--------
bolo@node154:~$ sudo ufw status
sudo: unable to resolve host node154
Status: active

To                         Action      From
--                         ------      ----
6443                       ALLOW       Anywhere                  
2379                       ALLOW       Anywhere                  
2380                       ALLOW       Anywhere                  
22                         ALLOW       Anywhere                  
6443 (v6)                  ALLOW       Anywhere (v6)             
2379 (v6)                  ALLOW       Anywhere (v6)             
2380 (v6)                  ALLOW       Anywhere (v6)             
22 (v6)                    ALLOW       Anywhere (v6)             

bolo@node154:~$ 
~~~

## 安装 rancher

~~~
bolo@rancher:~$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bolo@rancher:~$ docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
0c3980244a60601363daaeebdc95572660c9ab16667faf606d1fd33300dd3b20
bolo@rancher:~$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                      NAMES
0c3980244a60        rancher/rancher     "rancher --http-li..."   4 seconds ago       Up 3 seconds        0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   relaxed_bartik
bolo@rancher:~$
~~~

## 访问配置

随机生成密码

![rancher](/assets/img/rancher/rancher23.png)

配置服务链接

![rancher](/assets/img/rancher/rancher24.png)

进入主界面

![rancher](/assets/img/rancher/rancher25.png)

导入集群

![rancher](/assets/img/rancher/rancher26.png)

创建后进入集群界面

![rancher](/assets/img/rancher/rancher27.png)

然后会成生导入命令的提示

## 下载 RKE

下载地址在 **[GitHub][rke_dl]**

~~~
bolo@rancher:~$ wget https://github.com/rancher/rke/releases/download/v0.1.7-rc4/rke_linux-amd64
--2018-05-17 00:25:31--  https://github.com/rancher/rke/releases/download/v0.1.7-rc4/rke_linux-amd64
Resolving github.com (github.com)... 13.229.188.59, 13.250.177.223, 52.74.223.119
Connecting to github.com (github.com)|13.229.188.59|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/108337180/f22a5112-584c-11e8-8aaa-24243e6d37b7?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20180516%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180516T162532Z&X-Amz-Expires=300&X-Amz-Signature=97d27eef7fa5718d9f49a6f3a760b8137b98f52dfb66878e53accdb6ddc92456&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Drke_linux-amd64&response-content-type=application%2Foctet-stream [following]
--2018-05-17 00:25:32--  https://github-production-release-asset-2e65be.s3.amazonaws.com/108337180/f22a5112-584c-11e8-8aaa-24243e6d37b7?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20180516%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180516T162532Z&X-Amz-Expires=300&X-Amz-Signature=97d27eef7fa5718d9f49a6f3a760b8137b98f52dfb66878e53accdb6ddc92456&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Drke_linux-amd64&response-content-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 52.216.228.88
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|52.216.228.88|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 31328455 (30M) [application/octet-stream]
Saving to: 'rke_linux-amd64'

rke_linux-amd64                    100%[==============================================================>]  29.88M   252KB/s    in 2m 11s  

2018-05-17 00:27:45 (233 KB/s) - 'rke_linux-amd64' saved [31328455/31328455]

bolo@rancher:~$ sha256sum rke_linux-amd64 
1184b954cc439f6aa7f45fa7e8d71899cec54e18ec04f7e450accd3d0d2489ed  rke_linux-amd64
bolo@rancher:~$ file rke_linux-amd64 
rke_linux-amd64: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
bolo@rancher:~$ 
~~~

给予执行权限

~~~
bolo@rancher:~$ chmod +x rke_linux-amd64 
bolo@rancher:~$ ./rke_linux-amd64 -h 
NAME:
   rke - Rancher Kubernetes Engine, Running kubernetes cluster in the cloud

USAGE:
   rke_linux-amd64 [global options] command [command options] [arguments...]
   
VERSION:
   v0.1.7-rc4
   
AUTHOR(S):
   Rancher Labs, Inc. 
   
COMMANDS:
     up       Bring the cluster up
     remove   Teardown the cluster and clean cluster nodes
     version  Show cluster Kubernetes version
     config   Setup cluster configuration
     etcd     etcd backup/restore operations in k8s cluster
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug, -d    Debug logging
   --help, -h     show help
   --version, -v  print the version
   
bolo@rancher:~$ 
bolo@rancher:~$ ll rke_linux-amd64 
-rwxrwxr-x 1 bolo bolo 31328455 May 16 05:33 rke_linux-amd64*
bolo@rancher:~$
~~~


## 查看 rke 版本

其实帮助里已经包含了版本信息

~~~
bolo@rancher:~$ ./rke_linux-amd64 --version
rke version v0.1.7-rc4
bolo@rancher:~$ ./rke_linux-amd64 --help
NAME:
   rke - Rancher Kubernetes Engine, Running kubernetes cluster in the cloud

USAGE:
   rke_linux-amd64 [global options] command [command options] [arguments...]
   
VERSION:
   v0.1.7-rc4
   
AUTHOR(S):
   Rancher Labs, Inc. 
   
COMMANDS:
     up       Bring the cluster up
     remove   Teardown the cluster and clean cluster nodes
     version  Show cluster Kubernetes version
     config   Setup cluster configuration
     etcd     etcd backup/restore operations in k8s cluster
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug, -d    Debug logging
   --help, -h     show help
   --version, -v  print the version
   
bolo@rancher:~$
~~~

## 配置 cluster.yml

~~~
bolo@rancher:~$ vim cluster.yml
bolo@rancher:~$ cat cluster.yml 
---
nodes:
  - address: 192.168.56.154
    user: bolo
    role:
    - controlplane
    - worker
    - etcd
    port: 22
  - address: 192.168.56.153
    user: bolo
    role:
    - worker
    - etcd
 
services:
  etcd:
    image: rancher/etcd:latest
  kube-api:
    image: rancher/k8s:v1.10.0-rancher1-2
  kube-controller:
    image: rancher/k8s:v1.10.0-rancher1-2
  scheduler:
    image: rancher/k8s:v1.10.0-rancher1-2
  kubelet:
    image: rancher/k8s:v1.10.0-rancher1-2
  kubeproxy: 
    image: rancher/k8s:v1.10.0-rancher1-2
bolo@rancher:~$ 
~~~

这里我只用两台 VM 来构建 k8s 集群

一般建议 3 台或 3 台以上的奇数个节点来运行 k8s 集群

## 构建 k8s 集群

rke 的使用可以参考 **[github][rke_github]**

~~~
bolo@rancher:~$ ls
cluster.yml  rke_linux-amd64
bolo@rancher:~$ ./rke_linux-amd64 up
INFO[0000] Building Kubernetes cluster                  
INFO[0000] [dialer] Setup tunnel for host [192.168.56.154] 
INFO[0000] [dialer] Setup tunnel for host [192.168.56.153] 
INFO[0000] [network] Deploying port listener containers 
INFO[0000] [network] Successfully started [rke-etcd-port-listener] container on host [192.168.56.154] 
INFO[0000] [network] Successfully started [rke-etcd-port-listener] container on host [192.168.56.153] 
INFO[0001] [network] Successfully started [rke-cp-port-listener] container on host [192.168.56.154] 
INFO[0001] [network] Successfully started [rke-worker-port-listener] container on host [192.168.56.154] 
INFO[0001] [network] Successfully started [rke-worker-port-listener] container on host [192.168.56.153] 
INFO[0001] [network] Port listener containers deployed successfully 
INFO[0001] [network] Running etcd <-> etcd port checks  
INFO[0001] [network] Successfully started [rke-port-checker] container on host [192.168.56.154] 
INFO[0001] [network] Successfully started [rke-port-checker] container on host [192.168.56.153] 
INFO[0004] [network] Running control plane -> etcd port checks 
INFO[0004] [network] Successfully started [rke-port-checker] container on host [192.168.56.154] 
INFO[0005] [network] Running control plane -> worker port checks 
INFO[0006] [network] Successfully started [rke-port-checker] container on host [192.168.56.154] 
INFO[0006] [network] Running workers -> control plane port checks 
INFO[0006] [network] Successfully started [rke-port-checker] container on host [192.168.56.153] 
INFO[0006] [network] Successfully started [rke-port-checker] container on host [192.168.56.154] 
INFO[0006] [network] Checking KubeAPI port Control Plane hosts 
INFO[0006] [network] Removing port listener containers  
INFO[0006] [remove/rke-etcd-port-listener] Successfully removed container on host [192.168.56.153] 
INFO[0006] [remove/rke-etcd-port-listener] Successfully removed container on host [192.168.56.154] 
INFO[0007] [remove/rke-cp-port-listener] Successfully removed container on host [192.168.56.154] 
INFO[0007] [remove/rke-worker-port-listener] Successfully removed container on host [192.168.56.153] 
INFO[0007] [remove/rke-worker-port-listener] Successfully removed container on host [192.168.56.154] 
INFO[0007] [network] Port listener containers removed successfully 
INFO[0007] [certificates] Attempting to recover certificates from backup on [etcd,controlPlane] hosts 
INFO[0008] [certificates] Successfully started [cert-fetcher] container on host [192.168.56.154] 
INFO[0008] [certificates] Successfully started [cert-fetcher] container on host [192.168.56.153] 
INFO[0015] [certificates] Certificate backup found on [etcd,controlPlane] hosts 
INFO[0015] [certificates] Regenerating new etcd-192.168.56.154 certificate and key 
INFO[0016] [certificates] Successfully generated new etcd-192.168.56.154 certificate and key 
INFO[0016] [reconcile] Rebuilding and updating local kube config 
INFO[0016] Successfully Deployed local admin kubeconfig at [./kube_config_cluster.yml] 
INFO[0016] [reconcile] Reconciling cluster state        
INFO[0016] [reconcile] This is newly generated cluster  
INFO[0016] [certificates] Deploying kubernetes certificates to Cluster nodes 
INFO[0021] Successfully Deployed local admin kubeconfig at [./kube_config_cluster.yml] 
INFO[0021] [certificates] Successfully deployed kubernetes certificates to Cluster nodes 
INFO[0021] Pre-pulling kubernetes images                
INFO[0021] Kubernetes images pulled successfully        
INFO[0021] [etcd] Building up etcd plane..              
INFO[0022] [etcd] Successfully started [etcd] container on host [192.168.56.154] 
INFO[0022] [etcd] Successfully started [rke-log-linker] container on host [192.168.56.154] 
INFO[0022] [remove/rke-log-linker] Successfully removed container on host [192.168.56.154] 
INFO[0022] [etcd] Successfully started [etcd] container on host [192.168.56.153] 
INFO[0023] [etcd] Successfully started [rke-log-linker] container on host [192.168.56.153] 
INFO[0023] [remove/rke-log-linker] Successfully removed container on host [192.168.56.153] 
INFO[0023] [etcd] Successfully started etcd plane..     
INFO[0023] [controlplane] Building up Controller Plane.. 
INFO[0023] [controlplane] Successfully started [kube-apiserver] container on host [192.168.56.154] 
INFO[0023] [healthcheck] Start Healthcheck on service [kube-apiserver] on host [192.168.56.154] 
INFO[0034] [healthcheck] service [kube-apiserver] on host [192.168.56.154] is healthy 
INFO[0034] [controlplane] Successfully started [rke-log-linker] container on host [192.168.56.154] 
INFO[0034] [remove/rke-log-linker] Successfully removed container on host [192.168.56.154] 
INFO[0035] [controlplane] Successfully started [kube-controller-manager] container on host [192.168.56.154] 
INFO[0035] [healthcheck] Start Healthcheck on service [kube-controller-manager] on host [192.168.56.154] 
INFO[0040] [healthcheck] service [kube-controller-manager] on host [192.168.56.154] is healthy 
INFO[0040] [controlplane] Successfully started [rke-log-linker] container on host [192.168.56.154] 
INFO[0041] [remove/rke-log-linker] Successfully removed container on host [192.168.56.154] 
INFO[0041] [controlplane] Successfully started [kube-scheduler] container on host [192.168.56.154] 
INFO[0041] [healthcheck] Start Healthcheck on service [kube-scheduler] on host [192.168.56.154] 
INFO[0046] [healthcheck] service [kube-scheduler] on host [192.168.56.154] is healthy 
INFO[0047] [controlplane] Successfully started [rke-log-linker] container on host [192.168.56.154] 
INFO[0047] [remove/rke-log-linker] Successfully removed container on host [192.168.56.154] 
INFO[0047] [controlplane] Successfully started Controller Plane.. 
INFO[0047] [authz] Creating rke-job-deployer ServiceAccount 
INFO[0047] [authz] rke-job-deployer ServiceAccount created successfully 
INFO[0047] [authz] Creating system:node ClusterRoleBinding 
INFO[0047] [authz] system:node ClusterRoleBinding created successfully 
INFO[0047] [certificates] Save kubernetes certificates as secrets 
INFO[0049] [certificates] Successfully saved certificates as kubernetes secret [k8s-certs] 
INFO[0049] [state] Saving cluster state to Kubernetes   
INFO[0049] [state] Successfully Saved cluster state to Kubernetes ConfigMap: cluster-state 
INFO[0049] [worker] Building up Worker Plane..          
INFO[0049] [sidekick] Sidekick container already created on host [192.168.56.154] 
INFO[0049] [worker] Successfully started [kubelet] container on host [192.168.56.154] 
INFO[0049] [healthcheck] Start Healthcheck on service [kubelet] on host [192.168.56.154] 
INFO[0049] [worker] Successfully started [nginx-proxy] container on host [192.168.56.153] 
INFO[0050] [worker] Successfully started [rke-log-linker] container on host [192.168.56.153] 
INFO[0050] [remove/rke-log-linker] Successfully removed container on host [192.168.56.153] 
INFO[0051] [worker] Successfully started [kubelet] container on host [192.168.56.153] 
INFO[0051] [healthcheck] Start Healthcheck on service [kubelet] on host [192.168.56.153] 
INFO[0055] [healthcheck] service [kubelet] on host [192.168.56.154] is healthy 
INFO[0055] [worker] Successfully started [rke-log-linker] container on host [192.168.56.154] 
INFO[0056] [remove/rke-log-linker] Successfully removed container on host [192.168.56.154] 
INFO[0056] [worker] Successfully started [kube-proxy] container on host [192.168.56.154] 
INFO[0056] [healthcheck] Start Healthcheck on service [kube-proxy] on host [192.168.56.154] 
INFO[0056] [healthcheck] service [kubelet] on host [192.168.56.153] is healthy 
INFO[0057] [worker] Successfully started [rke-log-linker] container on host [192.168.56.153] 
INFO[0057] [remove/rke-log-linker] Successfully removed container on host [192.168.56.153] 
INFO[0057] [worker] Successfully started [kube-proxy] container on host [192.168.56.153] 
INFO[0057] [healthcheck] Start Healthcheck on service [kube-proxy] on host [192.168.56.153] 
INFO[0061] [healthcheck] service [kube-proxy] on host [192.168.56.154] is healthy 
INFO[0062] [worker] Successfully started [rke-log-linker] container on host [192.168.56.154] 
INFO[0062] [remove/rke-log-linker] Successfully removed container on host [192.168.56.154] 
INFO[0063] [healthcheck] service [kube-proxy] on host [192.168.56.153] is healthy 
INFO[0063] [worker] Successfully started [rke-log-linker] container on host [192.168.56.153] 
INFO[0063] [remove/rke-log-linker] Successfully removed container on host [192.168.56.153] 
INFO[0063] [worker] Successfully started Worker Plane.. 
INFO[0063] [sync] Syncing nodes Labels and Taints       
INFO[0063] [sync] Successfully synced nodes Labels and Taints 
INFO[0063] [network] Setting up network plugin: canal   
INFO[0063] [addons] Saving addon ConfigMap to Kubernetes 
INFO[0063] [addons] Successfully Saved addon to Kubernetes ConfigMap: rke-network-plugin 
INFO[0063] [addons] Executing deploy job..              
INFO[0063] [addons] Setting up KubeDNS                  
INFO[0063] [addons] Saving addon ConfigMap to Kubernetes 
INFO[0063] [addons] Successfully Saved addon to Kubernetes ConfigMap: rke-kubedns-addon 
INFO[0063] [addons] Executing deploy job..              
INFO[0063] [addons] KubeDNS deployed successfully..     
INFO[0063] [ingress] Setting up nginx ingress controller 
INFO[0063] [addons] Saving addon ConfigMap to Kubernetes 
INFO[0063] [addons] Successfully Saved addon to Kubernetes ConfigMap: rke-ingress-controller 
INFO[0063] [addons] Executing deploy job..              
INFO[0063] [ingress] ingress controller nginx is successfully deployed 
INFO[0063] [addons] Setting up user addons              
INFO[0063] [addons] no user addons defined              
INFO[0063] Finished building Kubernetes cluster successfully 
bolo@rancher:~$ echo $?
0
bolo@rancher:~$ 
~~~

此时，再到 153 和 154 上去看容器状态，会多出来很多东西 

~~~
bolo@node154:~$ docker ps -a 
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                      PORTS               NAMES
abb00b4dc12c        66e10c24a484                         "/usr/bin/dumb-ini..."   2 seconds ago       Up 1 second                                     k8s_nginx-ingress-controller_nginx-ingress-controller-4ksj6_ingress-nginx_09cf49e4-590e-11e8-9c35-08002736e2d9_0
faef9537499f        2b736d06ca4c                         "/opt/bin/flanneld..."   3 seconds ago       Up 3 seconds                                    k8s_kube-flannel_canal-24sdj_kube-system_fb7b2bf6-590d-11e8-9c35-08002736e2d9_0
dd2f829839e0        8cfec7659f1d                         "run.sh"                 3 seconds ago       Up 3 seconds                                    k8s_cluster-register_cattle-cluster-agent-7c4d79d459-kqxc2_cattle-system_e50b05bb-5920-11e8-9c35-08002736e2d9_0
af7bfc2aece2        482f47df27e2                         "/install-cni.sh"        4 seconds ago       Up 3 seconds                                    k8s_install-cni_canal-24sdj_kube-system_fb7b2bf6-590d-11e8-9c35-08002736e2d9_0
c6e6ad1c6c0e        rancher/pause-amd64:3.1              "/pause"                 4 seconds ago       Up 3 seconds                                    k8s_POD_cattle-cluster-agent-7c4d79d459-kqxc2_cattle-system_e50b05bb-5920-11e8-9c35-08002736e2d9_0
ce1b408606fe        846921f0fe0e                         "/server"                4 seconds ago       Up 2 seconds                                    k8s_default-http-backend_default-http-backend-564b9b6c5b-pb77k_ingress-nginx_09d33ac3-590e-11e8-9c35-08002736e2d9_0
323e677de712        8cfec7659f1d                         "run.sh"                 4 seconds ago       Up 3 seconds                                    k8s_agent_cattle-node-agent-88tq2_cattle-system_e74d61ba-5920-11e8-9c35-08002736e2d9_0
1fe37fc8dbc6        d94b64ac210d                         "start_runit"            4 seconds ago       Up 3 seconds                                    k8s_calico-node_canal-24sdj_kube-system_fb7b2bf6-590d-11e8-9c35-08002736e2d9_0
4b90e9d84102        rancher/pause-amd64:3.1              "/pause"                 4 seconds ago       Up 4 seconds                                    k8s_POD_cattle-node-agent-88tq2_cattle-system_e74d61ba-5920-11e8-9c35-08002736e2d9_0
502598c3a457        rancher/pause-amd64:3.1              "/pause"                 4 seconds ago       Up 4 seconds                                    k8s_POD_canal-24sdj_kube-system_fb7b2bf6-590d-11e8-9c35-08002736e2d9_0
1ed4fee12d12        5a88c6227f53                         "sh -c 'sysctl -w ..."   4 seconds ago       Exited (0) 4 seconds ago                        k8s_sysctl_nginx-ingress-controller-4ksj6_ingress-nginx_09cf49e4-590e-11e8-9c35-08002736e2d9_0
bca872b91b53        rancher/pause-amd64:3.1              "/pause"                 5 seconds ago       Up 4 seconds                                    k8s_POD_nginx-ingress-controller-4ksj6_ingress-nginx_09cf49e4-590e-11e8-9c35-08002736e2d9_0
45676a4f80ee        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   5 seconds ago       Up 4 seconds                                    kube-proxy
89576caa8640        rancher/pause-amd64:3.1              "/pause"                 5 seconds ago       Up 4 seconds                                    k8s_POD_default-http-backend-564b9b6c5b-pb77k_ingress-nginx_09d33ac3-590e-11e8-9c35-08002736e2d9_0
443c7b4fa488        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   11 seconds ago      Up 11 seconds                                   kubelet
352963a0695c        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   20 seconds ago      Up 19 seconds                                   kube-scheduler
02e5067903b3        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   26 seconds ago      Up 25 seconds                                   kube-controller-manager
1ff5292ed212        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   37 seconds ago      Up 37 seconds                                   kube-apiserver
141dfc01a059        rancher/rke-tools:v0.1.6             "/bin/bash"              37 seconds ago      Created                                         service-sidekick
ec43d413c0b7        rancher/coreos-etcd:v3.1.12          "/usr/local/bin/et..."   39 seconds ago      Up 38 seconds                                   etcd
61d799feb2df        rancher/rke-tools:v0.1.6             "/bin/bash"              53 seconds ago      Exited (0) 52 seconds ago                       cert-fetcher
bolo@node154:~$ 
~~~


~~~
bolo@node153:~$ docker ps -a 
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                     PORTS               NAMES
c9ce041a6547        2b736d06ca4c                         "/opt/bin/flanneld..."   2 minutes ago       Up 2 minutes                                   k8s_kube-flannel_canal-6ts6l_kube-system_fb7bf41e-590d-11e8-9c35-08002736e2d9_0
3a3a3911b42f        482f47df27e2                         "/install-cni.sh"        2 minutes ago       Up 2 minutes                                   k8s_install-cni_canal-6ts6l_kube-system_fb7bf41e-590d-11e8-9c35-08002736e2d9_0
e59205b5f940        66e10c24a484                         "/usr/bin/dumb-ini..."   2 minutes ago       Up 2 minutes                                   k8s_nginx-ingress-controller_nginx-ingress-controller-8z4qm_ingress-nginx_09d084b6-590e-11e8-9c35-08002736e2d9_0
a0feeee64b21        d94b64ac210d                         "start_runit"            2 minutes ago       Up 2 minutes                                   k8s_calico-node_canal-6ts6l_kube-system_fb7bf41e-590d-11e8-9c35-08002736e2d9_0
dbe294ff03a7        6f7f2dc7fab5                         "/sidecar --v=2 --..."   2 minutes ago       Up 2 minutes                                   k8s_sidecar_kube-dns-5ccb66df65-jl27r_kube-system_06d27f0b-590e-11e8-9c35-08002736e2d9_0
36f3ba0eaf72        rancher/pause-amd64:3.1              "/pause"                 2 minutes ago       Up 2 minutes                                   k8s_POD_canal-6ts6l_kube-system_fb7bf41e-590d-11e8-9c35-08002736e2d9_0
c87260bdcac2        8cfec7659f1d                         "run.sh"                 2 minutes ago       Up 2 minutes                                   k8s_agent_cattle-node-agent-x28r9_cattle-system_e936f188-5920-11e8-9c35-08002736e2d9_0
26936192c46a        c2ce1ffb51ed                         "/dnsmasq-nanny -v..."   2 minutes ago       Up 2 minutes                                   k8s_dnsmasq_kube-dns-5ccb66df65-jl27r_kube-system_06d27f0b-590e-11e8-9c35-08002736e2d9_0
c22f5b977cd5        80cc5ea4b547                         "/kube-dns --domai..."   2 minutes ago       Up 2 minutes                                   k8s_kubedns_kube-dns-5ccb66df65-jl27r_kube-system_06d27f0b-590e-11e8-9c35-08002736e2d9_0
f2b404104c54        rancher/pause-amd64:3.1              "/pause"                 2 minutes ago       Up 2 minutes                                   k8s_POD_cattle-node-agent-x28r9_cattle-system_e936f188-5920-11e8-9c35-08002736e2d9_0
b88aea37bd13        5a88c6227f53                         "sh -c 'sysctl -w ..."   2 minutes ago       Exited (0) 2 minutes ago                       k8s_sysctl_nginx-ingress-controller-8z4qm_ingress-nginx_09d084b6-590e-11e8-9c35-08002736e2d9_0
9ecb7c7e3b5d        rancher/pause-amd64:3.1              "/pause"                 2 minutes ago       Up 2 minutes                                   k8s_POD_nginx-ingress-controller-8z4qm_ingress-nginx_09d084b6-590e-11e8-9c35-08002736e2d9_0
405a8cf56b18        e183460c484d                         "/cluster-proporti..."   2 minutes ago       Up 2 minutes                                   k8s_autoscaler_kube-dns-autoscaler-6c4b786f5-g5mlh_kube-system_076206a0-590e-11e8-9c35-08002736e2d9_0
f4537c9d02fc        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   2 minutes ago       Up 2 minutes                                   kube-proxy
80165948a20a        rancher/pause-amd64:3.1              "/pause"                 2 minutes ago       Up 2 minutes                                   k8s_POD_kube-dns-5ccb66df65-jl27r_kube-system_06d27f0b-590e-11e8-9c35-08002736e2d9_0
79431adae40b        rancher/pause-amd64:3.1              "/pause"                 2 minutes ago       Up 2 minutes                                   k8s_POD_kube-dns-autoscaler-6c4b786f5-g5mlh_kube-system_076206a0-590e-11e8-9c35-08002736e2d9_0
b785cccf7f25        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   2 minutes ago       Up 2 minutes                                   kubelet
f0e1eaeba18d        rancher/rke-tools:v0.1.6             "/bin/bash"              2 minutes ago       Created                                        service-sidekick
73ef93643c02        rancher/rke-tools:v0.1.6             "nginx-proxy CP_HO..."   2 minutes ago       Up 2 minutes                                   nginx-proxy
4028257a8e06        rancher/coreos-etcd:v3.1.12          "/usr/local/bin/et..."   3 minutes ago       Up 3 minutes                                   etcd
bolo@node153:~$ 
~~~

## 安装 kubectl

参考下面链接安装 kubectl 工具

**[Install kubectl binary via native package management][kubectl_install]**

~~~
root@rancher:~# apt-get update && apt-get install -y apt-transport-https
Hit:1 https://download.docker.com/linux/ubuntu xenial InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu xenial InRelease                                                                               
Hit:3 http://security.ubuntu.com/ubuntu xenial-security InRelease   
Hit:5 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease  
Hit:6 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease
Hit:4 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
apt-transport-https is already the newest version (1.2.26).
The following package was automatically installed and is no longer required:
  pigz
Use 'apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 17 not upgraded.
root@rancher:~# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
OK
root@rancher:~# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
> deb http://apt.kubernetes.io/ kubernetes-xenial main
> EOF
root@rancher:~# apt-get update
Hit:2 http://security.ubuntu.com/ubuntu xenial-security InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu xenial InRelease
Hit:4 https://download.docker.com/linux/ubuntu xenial InRelease
Hit:5 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease
Hit:6 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
root@rancher:~# apt-get install -y kubectl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  pigz
Use 'apt autoremove' to remove it.
The following NEW packages will be installed:
  kubectl
0 upgraded, 1 newly installed, 0 to remove and 17 not upgraded.
Need to get 0 B/8,909 kB of archives.
After this operation, 54.3 MB of additional disk space will be used.
Selecting previously unselected package kubectl.
(Reading database ... 60315 files and directories currently installed.)
Preparing to unpack .../kubectl_1.10.2-00_amd64.deb ...
Unpacking kubectl (1.10.2-00) ...
Setting up kubectl (1.10.2-00) ...
root@rancher:~# 
root@rancher:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.2", GitCommit:"81753b10df112992bf51bbc2c2f85208aad78335", GitTreeState:"clean", BuildDate:"2018-04-27T09:22:21Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
root@rancher:~# 
~~~

## 测试与 k8s 集群的连接

~~~
bolo@rancher:~$ ls
cluster.yml  kube_config_cluster.yml  rke_linux-amd64
bolo@rancher:~$ mkdir ~/.kube
bolo@rancher:~$ cp kube_config_cluster.yml ~/.kube/config
bolo@rancher:~$ kubectl get node
NAME             STATUS    ROLES                      AGE       VERSION
192.168.56.153   Ready     etcd,worker                3h        v1.10.1
192.168.56.154   Ready     controlplane,etcd,worker   3h        v1.10.1
bolo@rancher:~$
~~~

查看其它配置信息

~~~
bolo@rancher:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://192.168.56.154:6443
  name: local
contexts:
- context:
    cluster: local
    user: kube-admin
  name: Default
current-context: Default
kind: Config
preferences: {}
users:
- name: kube-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
bolo@rancher:~$ 
~~~

## 加入现有的 rancher 集群

~~~
bolo@rancher:~$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                      NAMES
0c3980244a60        rancher/rancher     "rancher --http-li..."   59 minutes ago      Up 59 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   relaxed_bartik
bolo@rancher:~$ kubectl apply -f https://192.168.56.152/v3/import/2g8jvmjk2dlfjrg9287w7lqs957t8tzrnlfzg69bspbhh6zx4gx892.yaml
Unable to connect to the server: x509: certificate signed by unknown authority
bolo@rancher:~$ curl --insecure -sfL https://192.168.56.152/v3/import/2g8jvmjk2dlfjrg9287w7lqs957t8tzrnlfzg69bspbhh6zx4gx892.yaml | kubectl apply -f -
namespace "cattle-system" configured
serviceaccount "cattle" unchanged
clusterrolebinding.rbac.authorization.k8s.io "cattle" configured
secret "cattle-credentials-ec1c4f6" created
deployment.extensions "cattle-cluster-agent" configured
daemonset.extensions "cattle-node-agent" configured
bolo@rancher:~$ 
~~~

过一小会儿，就可以看到这个集群已经 Active 了

![rancher](/assets/img/rancher/rancher28.png)

集群中的信息呈现也十分直观

![rancher](/assets/img/rancher/rancher29.png)


获得了预期的效果

---

# 总结

使用 Rancher RKE 来布署 k8s 集群，不得不说是目前我用过的最简单的办法

这种简单和高效，必然是未来 DevOps 的趋势

* TOC
{:toc}

---

[rancher]:https://rancher.com/
[introduction_rke]:https://rancher.com/an-introduction-to-rke/
[rke_github]:https://github.com/rancher/rke
[rancher_rke]:https://www.kubernetes.org.cn/3280.html
[rke_dl]:https://github.com/rancher/rke/releases
[kubectl_install]:https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-snap-on-ubuntu