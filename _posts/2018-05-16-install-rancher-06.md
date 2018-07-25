---
layout: post
title: "Install Rancher 6"
author:  wilmosfang
date: 2018-05-16 17:48:03
image: '/assets/img/'
excerpt: 'rancher 节点扩容'
main-class: 'rancher'
color: '#2980b8'
tags:
 - docker
 - rancher
 - kubernetes
categories: 
 - rancher
twitter_text: 'Expand capacity of rancher'
introduction: 'expand capacity'
---


# 前言

**[Rancher][rancher]** 是一款开源的容器管理软件

**[Rancher][rancher]** 的设计目标的是简化容器的管理操作，提升容器应用的操作效率

因为整合了 k8s 的编排功能, 并且有着非常友好的操作界面，所以在目前的容器技术圈中有着很大的影响力

如果要快速构建一套 **CI/CD** 发布平台， **[Rancher][rancher]** 是一个不错的选择

这里基于前面的工作，演示一下如何给 **[Rancher][rancher]** 扩容

参考 **[Quick Start Guide][rancher_guide]**

> **Tip:** 当前的版本为 **Rancher 2.0 GA** 和 **rke v0.1.7-rc4**

---

## 运行环境

~~~
bolo@rancher:~$ hostnamectl 
   Static hostname: rancher
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4bae573cc9bc7f877992f1885ae0c42b
           Boot ID: 40c628a7a1a44a769200d9b15464bbb8
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
    link/ether 02:42:69:a1:ca:3a brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:69ff:fea1:ca3a/64 scope link 
       valid_lft forever preferred_lft forever
6: veth2a144ba@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 66:91:b8:1e:52:9f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::6491:b8ff:fe1e:529f/64 scope link 
       valid_lft forever preferred_lft forever
53: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether a6:c9:8a:e0:20:ad brd ff:ff:ff:ff:ff:ff
    inet 10.42.2.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::a4c9:8aff:fee0:20ad/64 scope link 
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
bolo@rancher:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                      NAMES
0c3980244a60        rancher/rancher     "rancher --http-li..."   2 days ago          Up 42 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   relaxed_bartik
bolo@rancher:~$ 
~~~

~~~
bolo@node153:~$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bolo@node153:~$ 
~~~

~~~
bolo@node154:~$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bolo@node154:~$
~~~


## 关闭防火墙

~~~
bolo@node154:~$ sudo ufw disable 
sudo: unable to resolve host node154
Firewall stopped and disabled on system startup
bolo@node154:~$ sudo ufw status
sudo: unable to resolve host node154
Status: inactive
bolo@node154:~$
~~~

因为是测试环境，先关闭防火墙

要在所有节点上执行以确保没有来自网络的阻断

* 192.168.56.152
* 192.168.56.153
* 192.168.56.154

## 访问 rancher

访问主界面

![rancher](/assets/img/rancher/rancher30.png)

创建一个集群

![rancher](/assets/img/rancher/rancher31.png)

集群参数先使用默认的，暂不修改

创建后会生成命令配置

![rancher](/assets/img/rancher/rancher32.png)

在目标机器上执行

~~~
bolo@node154:~$ sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.0 --server https://192.168.56.152 --token 6rfblpm8l85g5m8cscbhhsdfsxmpwzl4mv465vg9brssdbtk7zgfpk --ca-checksum e36067163218cb0c2a5dfe4dee0eb7e0af4f6a6f7392a26ad0d5121f839f120d --address 10.0.2.15 --internal-address 192.168.56.154 --etcd --controlplane --worker
sudo: unable to resolve host node154
[sudo] password for bolo: 
d42ed3f43a5b35085ecae25ad822d571bd8c9b172593b71282f1012e2fd5c728
bolo@node154:~$
~~~

一段时间后集群中的第一个节点加入成功，并且集群状态变为 active 的

![rancher](/assets/img/rancher/rancher33.png)

目标节点中也多了很很容器

~~~
bolo@node154:~$ docker ps -a 
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                            PORTS               NAMES
be90c1bbd125        8cfec7659f1d                         "run.sh"                 5 minutes ago       Up 5 minutes                                          k8s_cluster-register_cattle-cluster-agent-855985d899-k9zq2_cattle-system_7a6f5659-5abc-11e8-8b8c-08002736e2d9_0
c5cf8937ca9b        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_cattle-cluster-agent-855985d899-k9zq2_cattle-system_7a6f5659-5abc-11e8-8b8c-08002736e2d9_0
f3e1b6a6d30c        8cfec7659f1d                         "run.sh"                 5 minutes ago       Up 5 minutes                                          k8s_agent_cattle-node-agent-2zmrr_cattle-system_7a75d97e-5abc-11e8-8b8c-08002736e2d9_0
386fa3371680        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_cattle-node-agent-2zmrr_cattle-system_7a75d97e-5abc-11e8-8b8c-08002736e2d9_0
92ef61a6b0c2        6f7f2dc7fab5                         "/sidecar --v=2 --..."   5 minutes ago       Up 5 minutes                                          k8s_sidecar_kube-dns-7dfdc4897f-hmcxp_kube-system_7137e32c-5abc-11e8-8b8c-08002736e2d9_0
13764c02edd8        c2ce1ffb51ed                         "/dnsmasq-nanny -v..."   5 minutes ago       Up 5 minutes                                          k8s_dnsmasq_kube-dns-7dfdc4897f-hmcxp_kube-system_7137e32c-5abc-11e8-8b8c-08002736e2d9_0
3ef872e46f01        80cc5ea4b547                         "/kube-dns --domai..."   5 minutes ago       Up 5 minutes                                          k8s_kubedns_kube-dns-7dfdc4897f-hmcxp_kube-system_7137e32c-5abc-11e8-8b8c-08002736e2d9_0
54ff549c7f07        e183460c484d                         "/cluster-proporti..."   5 minutes ago       Up 5 minutes                                          k8s_autoscaler_kube-dns-autoscaler-6c4b786f5-rshqq_kube-system_71cb839a-5abc-11e8-8b8c-08002736e2d9_0
831dd754a868        66e10c24a484                         "/usr/bin/dumb-ini..."   5 minutes ago       Up 5 minutes                                          k8s_nginx-ingress-controller_nginx-ingress-controller-rd9n2_ingress-nginx_74552f1f-5abc-11e8-8b8c-08002736e2d9_0
0ddd52744d15        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_kube-dns-autoscaler-6c4b786f5-rshqq_kube-system_71cb839a-5abc-11e8-8b8c-08002736e2d9_0
d0fde8d1e5c2        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_kube-dns-7dfdc4897f-hmcxp_kube-system_7137e32c-5abc-11e8-8b8c-08002736e2d9_0
e4d6304c1f8e        846921f0fe0e                         "/server"                5 minutes ago       Up 5 minutes                                          k8s_default-http-backend_default-http-backend-564b9b6c5b-2hf4j_ingress-nginx_7456b785-5abc-11e8-8b8c-08002736e2d9_0
4aeff92a3873        d1a7302844b3                         "sh -c 'sysctl -w ..."   5 minutes ago       Exited (0) 5 minutes ago                              k8s_sysctl_nginx-ingress-controller-rd9n2_ingress-nginx_74552f1f-5abc-11e8-8b8c-08002736e2d9_0
06c33cd01f69        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_default-http-backend-564b9b6c5b-2hf4j_ingress-nginx_7456b785-5abc-11e8-8b8c-08002736e2d9_0
7a6aed63c5d8        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_nginx-ingress-controller-rd9n2_ingress-nginx_74552f1f-5abc-11e8-8b8c-08002736e2d9_0
5cecd149890c        3e3f2ccada54                         "kubectl apply -f ..."   5 minutes ago       Exited (0) 5 minutes ago                              k8s_rke-ingress-controller-pod_rke-ingress-controller-deploy-job-zgjqk_kube-system_73858961-5abc-11e8-8b8c-08002736e2d9_0
a9b6d57190ab        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Exited (0) 5 minutes ago                              k8s_POD_rke-ingress-controller-deploy-job-zgjqk_kube-system_73858961-5abc-11e8-8b8c-08002736e2d9_0
7a7699aa4897        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Exited (137) 5 minutes ago                            k8s_POD_rke-kubedns-addon-deploy-job-f9vpc_kube-system_7081921e-5abc-11e8-8b8c-08002736e2d9_1
e76e6d53d1de        3e3f2ccada54                         "kubectl apply -f ..."   5 minutes ago       Exited (0) 5 minutes ago                              k8s_rke-kubedns-addon-pod_rke-kubedns-addon-deploy-job-f9vpc_kube-system_7081921e-5abc-11e8-8b8c-08002736e2d9_0
add07fc038e0        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Exited (0) 5 minutes ago                              k8s_POD_rke-kubedns-addon-deploy-job-f9vpc_kube-system_7081921e-5abc-11e8-8b8c-08002736e2d9_0
cdd68cf894f4        2b736d06ca4c                         "/opt/bin/flanneld..."   5 minutes ago       Up 5 minutes                                          k8s_kube-flannel_canal-858q6_kube-system_6e2b2243-5abc-11e8-8b8c-08002736e2d9_0
cffaaad4a121        482f47df27e2                         "/install-cni.sh"        5 minutes ago       Up 5 minutes                                          k8s_install-cni_canal-858q6_kube-system_6e2b2243-5abc-11e8-8b8c-08002736e2d9_0
e229b74f62c9        d94b64ac210d                         "start_runit"            5 minutes ago       Up 5 minutes                                          k8s_calico-node_canal-858q6_kube-system_6e2b2243-5abc-11e8-8b8c-08002736e2d9_0
8ff696533637        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_canal-858q6_kube-system_6e2b2243-5abc-11e8-8b8c-08002736e2d9_0
9c19708aa5ca        3e3f2ccada54                         "kubectl apply -f ..."   5 minutes ago       Exited (0) 5 minutes ago                              k8s_rke-network-plugin-pod_rke-network-plugin-deploy-job-pnfhc_kube-system_6d77f88a-5abc-11e8-8b8c-08002736e2d9_0
909099f73320        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Exited (0) 5 minutes ago                              k8s_POD_rke-network-plugin-deploy-job-pnfhc_kube-system_6d77f88a-5abc-11e8-8b8c-08002736e2d9_0
68ede732514a        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   5 minutes ago       Up 5 minutes                                          kube-proxy
f28e7a288478        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   6 minutes ago       Up 6 minutes                                          kubelet
efcdb18da1ff        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   6 minutes ago       Up 6 minutes                                          kube-scheduler
8429869deb70        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   6 minutes ago       Up 6 minutes                                          kube-controller-manager
37373ed9997b        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   6 minutes ago       Up 6 minutes                                          kube-apiserver
1bca6c509b8a        rancher/rke-tools:v0.1.4             "/bin/bash"              6 minutes ago       Created                                               service-sidekick
cf7bb65d4652        rancher/coreos-etcd:v3.1.12          "/usr/local/bin/et..."   6 minutes ago       Up 6 minutes                                          etcd
3e69b4bf203d        rancher/rke-tools:v0.1.4             "/bin/bash"              6 minutes ago       Exited (0) 6 minutes ago                              cert-fetcher
2b7658323fc1        rancher/rancher-agent:v2.0.0         "run.sh -- share-r..."   7 minutes ago       Exited (137) About a minute ago                       share-mnt
bolo@node154:~$ 
~~~

## 扩容一个节点

重复之前的操作生成命令

![rancher](/assets/img/rancher/rancher34.png)

在目标节点上执行命令

~~~
bolo@node153:~$ sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.0 --server https://192.168.56.152 --token 6rfblpm8l85g5m8cscbhhsdfsxmpwzl4mv465vg9brssdbtk7zgfpk --ca-checksum e36067163218cb0c2a5dfe4dee0eb7e0af4f6a6f7392a26ad0d5121f839f120d --address 10.0.2.4 --internal-address 192.168.56.153 --etcd --worker
[sudo] password for bolo: 
5fe309cdc6dc6724d86dfb215680e1292e0d86bcb94b5760c9e8243ecd50f86c
bolo@node153:~$ 
~~~

目标节点上开始布署环境

![rancher](/assets/img/rancher/rancher35.png)

过一小会儿，加入成功

![rancher](/assets/img/rancher/rancher36.png)

可以看到，现在集群中已经有两个节点了

![rancher](/assets/img/rancher/rancher37.png)

此节点中也多出一堆容器来

~~~
bolo@node153:~$ docker ps -a 
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                            PORTS               NAMES
0fb12c6b52b7        66e10c24a484                         "/usr/bin/dumb-ini..."   5 minutes ago       Up 5 minutes                                          k8s_nginx-ingress-controller_nginx-ingress-controller-mgbgb_ingress-nginx_db5d8d42-5abd-11e8-9894-08002736e2d9_0
1a973d9b5a88        2b736d06ca4c                         "/opt/bin/flanneld..."   5 minutes ago       Up 5 minutes                                          k8s_kube-flannel_canal-5k8m6_kube-system_db5ac46e-5abd-11e8-9894-08002736e2d9_0
7ec9023aa2a1        d1a7302844b3                         "sh -c 'sysctl -w ..."   5 minutes ago       Exited (0) 5 minutes ago                              k8s_sysctl_nginx-ingress-controller-mgbgb_ingress-nginx_db5d8d42-5abd-11e8-9894-08002736e2d9_0
589b38cb5379        482f47df27e2                         "/install-cni.sh"        5 minutes ago       Up 5 minutes                                          k8s_install-cni_canal-5k8m6_kube-system_db5ac46e-5abd-11e8-9894-08002736e2d9_0
e5c2b55d0d4d        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_nginx-ingress-controller-mgbgb_ingress-nginx_db5d8d42-5abd-11e8-9894-08002736e2d9_0
1c59ca552e97        8cfec7659f1d                         "run.sh"                 5 minutes ago       Up 5 minutes                                          k8s_agent_cattle-node-agent-jhqpz_cattle-system_db623f67-5abd-11e8-9894-08002736e2d9_0
cf9903c1df1e        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   5 minutes ago       Up 5 minutes                                          kube-proxy
64e0371f255f        d94b64ac210d                         "start_runit"            5 minutes ago       Up 5 minutes                                          k8s_calico-node_canal-5k8m6_kube-system_db5ac46e-5abd-11e8-9894-08002736e2d9_0
b50184f249ff        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_cattle-node-agent-jhqpz_cattle-system_db623f67-5abd-11e8-9894-08002736e2d9_0
d6dbc24bc2c9        rancher/pause-amd64:3.1              "/pause"                 5 minutes ago       Up 5 minutes                                          k8s_POD_canal-5k8m6_kube-system_db5ac46e-5abd-11e8-9894-08002736e2d9_0
8e2561437b47        rancher/hyperkube:v1.10.1-rancher2   "/opt/rke/entrypoi..."   5 minutes ago       Up 5 minutes                                          kubelet
bd19143868bf        rancher/rke-tools:v0.1.4             "/bin/bash"              5 minutes ago       Created                                               service-sidekick
cac4c6d923f2        rancher/rke-tools:v0.1.4             "nginx-proxy CP_HO..."   5 minutes ago       Up 5 minutes                                          nginx-proxy
918d3ebcd9df        rancher/coreos-etcd:v3.1.12          "/usr/local/bin/et..."   6 minutes ago       Up 6 minutes                                          etcd
a482c922364b        rancher/rancher-agent:v2.0.0         "run.sh -- share-r..."   6 minutes ago       Exited (137) About a minute ago                       share-mnt
bolo@node153:~$ 
~~~

## 扩容第二个节点

重复之前的操作生成命令

![rancher](/assets/img/rancher/rancher38.png)

在目标节点上执行命令

~~~
bolo@rancher:~$ sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.0 --server https://192.168.56.152 --token 6rfblpm8l85g5m8cscbhhsdfsxmpwzl4mv465vg9brssdbtk7zgfpk --ca-checksum e36067163218cb0c2a5dfe4dee0eb7e0af4f6a6f7392a26ad0d5121f839f120d --address 10.0.2.5 --internal-address 192.168.56.152 --etcd --worker
[sudo] password for bolo: 
5bee1cbf1212721ab66eb934f859873d10b8cb94beca1f4bcaeed35f6572d7c9
bolo@rancher:~$ 
~~~


目标节点上开始布署环境

![rancher](/assets/img/rancher/rancher39.png)

过一小会儿，加入成功

![rancher](/assets/img/rancher/rancher40.png)

可以看到，现在集群中已经有三个节点了

![rancher](/assets/img/rancher/rancher41.png)

此节点中也多出一堆容器来

到此，rancher 集群的扩容就完成了

使用同样的办法，可以继续扩更多节点进来

**Note:** 如果要清掉重来，一定要使用 **`rke remove`** 来清理一些内部数据，否则再建的过程会遇到问题，同时建议顺便也清掉 **`~/.kube/config`** 文件，它是用来连接 k8s 集群的配置文件

**Tip:** 如果使用 rke 来构建一个类似的 k8s 集群，可以参考下面的配置

~~~
nodes:
  - address: 192.168.56.154
    user: bolo
    role:
    - controlplane
    - worker
    - etcd
    port: 22
  - address: 192.168.56.152
    user: bolo
    role:
    - worker
    - etcd
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

---

# 总结

使用 Rancher 自带的命令生成工具来扩展 k8s 集群非常简单而有效

越来越喜欢 rancher 了

后面有空再研究一下 ceph ，就可以基于这三种技术构建出一套小型的私有云环境了

* TOC
{:toc}

---

[rancher]:https://rancher.com/
[rancher_guide]:https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/