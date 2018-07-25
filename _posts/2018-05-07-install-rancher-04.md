---
layout: post
title: "Install Rancher 4"
author:  wilmosfang
date: 2018-05-07 16:31:18
image: '/assets/img/'
excerpt: '安装 Rancher'
main-class: 'rancher'
color: '#2980b8'
tags:
 - docker
 - rancher
 - nginx
categories: 
 - rancher
twitter_text: 'simple process of Rancher installation'
introduction: 'Installation of Nginx'
---

# 前言

**[Rancher][rancher]** 是一款开源的容器管理软件

**[Rancher][rancher]** 的设计目标的是简化容器的管理操作，提升容器应用的操作效率

因为整合了 k8s 的编排功能, 并且有着非常友好的操作界面，所以在目前的容器技术圈中有着很大的影响力

如果要快速构建一套 **CI/CD** 发布平台， **[Rancher][rancher]** 是一个不错的选择

这里基于前面的工作，演示一下如何构建一个 Nginx 应用

参考 **[Quick Start Guide][rancher_guide]**

> **Tip:** 当前的版本为 **Rancher 2.0 GA**

---

## 运行环境

~~~
root@rancher:~# hostnamectl 
   Static hostname: rancher
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4bae573cc9bc7f877992f1885ae0c42b
           Boot ID: 6064b84b6d9646eebbd22e2e4e5227bb
    Virtualization: oracle
  Operating System: Ubuntu 16.04.4 LTS
            Kernel: Linux 4.4.0-116-generic
      Architecture: x86-64
root@rancher:~# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:87:b2:4d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
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
    link/ether 02:42:67:13:8c:f1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:67ff:fe13:8cf1/64 scope link 
       valid_lft forever preferred_lft forever
8: veth15ebd93@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 86:51:91:be:27:25 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::8451:91ff:febe:2725/64 scope link 
       valid_lft forever preferred_lft forever
33: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether 02:cc:d2:e0:e8:91 brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::cc:d2ff:fee0:e891/64 scope link 
       valid_lft forever preferred_lft forever
34: calie57880a6180@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
35: calie52ce405014@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
36: calid27ce70d57a@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 3
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
37: calidc43affed2c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 4
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
root@rancher:~# dpkg -l | grep -i docker
rc  docker                              1.5-1                                      amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                           17.03.2~ce-0~ubuntu-xenial                 amd64        Docker: the open-source application container engine
root@rancher:~# docker version
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
root@rancher:~# docker ps -a
CONTAINER ID        IMAGE                                                                                                                     COMMAND                  CREATED             STATUS                       PORTS                                      NAMES
82f7085ba368        66e10c24a484                                                                                                              "/usr/bin/dumb-ini..."   4 minutes ago       Exited (255) 4 minutes ago                                              k8s_nginx-ingress-controller_nginx-ingress-controller-qtk79_ingress-nginx_f3aa2c14-5211-11e8-8729-08002787b24d_8
6711f3776518        rancher/k8s-dns-sidecar-amd64@sha256:23df717980b4aa08d2da6c4cfa327f1b730d92ec9cf740959d2d5911830d82fb                     "/sidecar --v=2 --..."   19 minutes ago      Up 19 minutes                                                           k8s_sidecar_kube-dns-7dfdc4897f-7rmz5_kube-system_f0767260-5211-11e8-8729-08002787b24d_0
57c430f0f87a        rancher/k8s-dns-dnsmasq-nanny-amd64@sha256:93c827f018cf3322f1ff2aa80324a0306048b0a69bc274e423071fb0d2d29d8b               "/dnsmasq-nanny -v..."   19 minutes ago      Up 19 minutes                                                           k8s_dnsmasq_kube-dns-7dfdc4897f-7rmz5_kube-system_f0767260-5211-11e8-8729-08002787b24d_0
b3e0824d7c1e        rancher/nginx-ingress-controller-defaultbackend@sha256:865b0c35e6da393b8e80b7e3799f777572399a4cff047eb02a81fa6e7a48ed4b   "/server"                19 minutes ago      Up 19 minutes                                                           k8s_default-http-backend_default-http-backend-564b9b6c5b-82qw5_ingress-nginx_f3b0d66b-5211-11e8-8729-08002787b24d_0
7b17a6c3f235        rancher/cluster-proportional-autoscaler-amd64@sha256:77d2544c9dfcdfcf23fa2fcf4351b43bf3a124c54f2da1f7d611ac54669e3336     "/cluster-proporti..."   19 minutes ago      Up 19 minutes                                                           k8s_autoscaler_kube-dns-autoscaler-6c4b786f5-h4vjv_kube-system_f10785c0-5211-11e8-8729-08002787b24d_0
fbcc9b8159f9        8cfec7659f1d                                                                                                              "run.sh"                 20 minutes ago      Up 20 minutes                                                           k8s_cluster-register_cattle-cluster-agent-57cd9bd678-6svfq_cattle-system_f7192201-5211-11e8-8729-08002787b24d_0
4567f07a7adc        rancher/pause-amd64:3.1                                                                                                   "/pause"                 20 minutes ago      Up 20 minutes                                                           k8s_POD_cattle-cluster-agent-57cd9bd678-6svfq_cattle-system_f7192201-5211-11e8-8729-08002787b24d_0
d16684a11cf3        rancher/k8s-dns-kube-dns-amd64@sha256:6d8e0da4fb46e9ea2034a3f4cab0e095618a2ead78720c12e791342738e5f85d                    "/kube-dns --domai..."   20 minutes ago      Up 20 minutes                                                           k8s_kubedns_kube-dns-7dfdc4897f-7rmz5_kube-system_f0767260-5211-11e8-8729-08002787b24d_0
915b08a9cf09        rancher/pause-amd64:3.1                                                                                                   "/pause"                 20 minutes ago      Up 20 minutes                                                           k8s_POD_default-http-backend-564b9b6c5b-82qw5_ingress-nginx_f3b0d66b-5211-11e8-8729-08002787b24d_0
9c015c99e11b        rancher/pause-amd64:3.1                                                                                                   "/pause"                 20 minutes ago      Up 20 minutes                                                           k8s_POD_kube-dns-autoscaler-6c4b786f5-h4vjv_kube-system_f10785c0-5211-11e8-8729-08002787b24d_0
9d470be5afd6        rancher/pause-amd64:3.1                                                                                                   "/pause"                 20 minutes ago      Up 20 minutes                                                           k8s_POD_kube-dns-7dfdc4897f-7rmz5_kube-system_f0767260-5211-11e8-8729-08002787b24d_0
fa941050b9c9        rancher/coreos-flannel@sha256:93952a105b4576e8f09ab8c4e00483131b862c24180b0b7d342fb360bbe44f3d                            "/opt/bin/flanneld..."   20 minutes ago      Up 20 minutes                                                           k8s_kube-flannel_canal-v4tdn_kube-system_ee94dd19-5211-11e8-8729-08002787b24d_0
473cda198333        rancher/calico-cni@sha256:cafcb06d6bd5ed1651e6cc7fe3f9a1848606be3950d7218cf4c9439634ca5342                                "/install-cni.sh"        20 minutes ago      Up 20 minutes                                                           k8s_install-cni_canal-v4tdn_kube-system_ee94dd19-5211-11e8-8729-08002787b24d_0
48fefb1aac54        rancher/calico-node@sha256:21d581d7356f2dba648f2905502a38fd4ae325fd079d377bcf94028bcfa577a3                               "start_runit"            21 minutes ago      Up 21 minutes                                                           k8s_calico-node_canal-v4tdn_kube-system_ee94dd19-5211-11e8-8729-08002787b24d_0
7be9fea39428        8cfec7659f1d                                                                                                              "run.sh"                 22 minutes ago      Up 22 minutes                                                           k8s_agent_cattle-node-agent-58xrp_cattle-system_f71fdab6-5211-11e8-8729-08002787b24d_0
10c41c63a631        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Up 22 minutes                                                           k8s_POD_cattle-node-agent-58xrp_cattle-system_f71fdab6-5211-11e8-8729-08002787b24d_0
c5b945b76851        d1a7302844b3                                                                                                              "sh -c 'sysctl -w ..."   22 minutes ago      Exited (0) 22 minutes ago                                               k8s_sysctl_nginx-ingress-controller-qtk79_ingress-nginx_f3aa2c14-5211-11e8-8729-08002787b24d_0
a9c08d788d8e        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Up 22 minutes                                                           k8s_POD_nginx-ingress-controller-qtk79_ingress-nginx_f3aa2c14-5211-11e8-8729-08002787b24d_0
c3ed57acfee1        3e3f2ccada54                                                                                                              "kubectl apply -f ..."   22 minutes ago      Exited (0) 22 minutes ago                                               k8s_rke-ingress-controller-pod_rke-ingress-controller-deploy-job-2c98g_kube-system_f2bf4df0-5211-11e8-8729-08002787b24d_0
fc3751a5a9c2        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Exited (0) 22 minutes ago                                               k8s_POD_rke-ingress-controller-deploy-job-2c98g_kube-system_f2bf4df0-5211-11e8-8729-08002787b24d_0
bd47246e7243        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Exited (0) 22 minutes ago                                               k8s_POD_rke-kubedns-addon-deploy-job-2hfmb_kube-system_efb9313e-5211-11e8-8729-08002787b24d_1
393a1e244275        3e3f2ccada54                                                                                                              "kubectl apply -f ..."   22 minutes ago      Exited (0) 22 minutes ago                                               k8s_rke-kubedns-addon-pod_rke-kubedns-addon-deploy-job-2hfmb_kube-system_efb9313e-5211-11e8-8729-08002787b24d_0
75579d3d1e98        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Exited (0) 22 minutes ago                                               k8s_POD_rke-kubedns-addon-deploy-job-2hfmb_kube-system_efb9313e-5211-11e8-8729-08002787b24d_0
64db385bcdcd        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Up 22 minutes                                                           k8s_POD_canal-v4tdn_kube-system_ee94dd19-5211-11e8-8729-08002787b24d_0
47b681f1effb        3e3f2ccada54                                                                                                              "kubectl apply -f ..."   22 minutes ago      Exited (0) 22 minutes ago                                               k8s_rke-network-plugin-pod_rke-network-plugin-deploy-job-4vj6d_kube-system_e3bd1d65-5211-11e8-8729-08002787b24d_0
608c56155985        rancher/pause-amd64:3.1                                                                                                   "/pause"                 22 minutes ago      Exited (0) 22 minutes ago                                               k8s_POD_rke-network-plugin-deploy-job-4vj6d_kube-system_e3bd1d65-5211-11e8-8729-08002787b24d_0
3a3c5d38c5b9        rancher/hyperkube:v1.10.1-rancher2                                                                                        "/opt/rke/entrypoi..."   22 minutes ago      Up 22 minutes                                                           kube-proxy
a4b975145ccc        rancher/hyperkube:v1.10.1-rancher2                                                                                        "/opt/rke/entrypoi..."   22 minutes ago      Up 22 minutes                                                           kubelet
faf8a462fd13        rancher/hyperkube:v1.10.1-rancher2                                                                                        "/opt/rke/entrypoi..."   22 minutes ago      Up 22 minutes                                                           kube-scheduler
c9259ead8d90        rancher/hyperkube:v1.10.1-rancher2                                                                                        "/opt/rke/entrypoi..."   23 minutes ago      Up 23 minutes                                                           kube-controller-manager
d3128bad8694        rancher/hyperkube:v1.10.1-rancher2                                                                                        "/opt/rke/entrypoi..."   23 minutes ago      Up 23 minutes                                                           kube-apiserver
ec68d5e209b5        rancher/rke-tools:v0.1.4                                                                                                  "/bin/bash"              23 minutes ago      Created                                                                 service-sidekick
a94ed66eed99        rancher/coreos-etcd:v3.1.12                                                                                               "/usr/local/bin/et..."   23 minutes ago      Up 23 minutes                                                           etcd
50494e50e532        rancher/rke-tools:v0.1.4                                                                                                  "/bin/bash"              25 minutes ago      Exited (0) 25 minutes ago                                               cert-fetcher
627036dedbbf        rancher/rancher-agent:v2.0.0                                                                                              "run.sh -- share-r..."   25 minutes ago      Exited (137) 1 second ago                                               share-mnt
4ffb830d13cc        rancher/rancher                                                                                                           "rancher --http-li..."   47 minutes ago      Up 47 minutes                0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   peaceful_hodgkin
dfaf2be1374d        hello-world                                                                                                               "/hello"                 11 days ago         Exited (0) 11 days ago                                                  angry_montalcini
root@rancher:~# 
~~~

## 创建一个 nginx 服务

选择 Deploy

![rancher](/assets/img/rancher/rancher20.png)

填好名称，镜像，端口，然后 Lunch

![rancher](/assets/img/rancher/rancher21.png)

过一小会儿这个应用就运行起来了

![rancher](/assets/img/rancher/rancher22.png)

我们在本地进行访问

~~~
root@rancher:~# curl http://10.42.0.6:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@rancher:~# 
~~~

获得了预期的效果

---

# 总结

使用 Rancher 来布署应用是一个十分简单的过程

这种简单和高效，必然是未来 DevOps 的趋势


* TOC
{:toc}

---

[rancher]:https://rancher.com/
[rancher_guide]:https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/
[rancher_requirements]:https://rancher.com/docs/rancher/v2.0/en/quick-start-guide/#host-and-node-requirements
[docker_install]:https://docs.docker.com/install/linux/docker-ce/ubuntu/
