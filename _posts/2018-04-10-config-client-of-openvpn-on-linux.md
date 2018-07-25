---
layout: post
title: "Config Client of OpenVPN on linux"
author:  wilmosfang
date: 2018-04-10 14:59:26
image: '/assets/img/'
excerpt: 'Linux 下安装配置 OpenVPN 客户端'
main-class:  'tools'
color: '#808080'
tags:
 - tools
 - openvpn
categories:
 - tools
twitter_text: 'simple process of OpenVPN client configuration'
introduction: 'Configuration of OpenVPN'
---


## 前言

**[OpenVPN][openvpn]** 是一款开源的 **VPN(Virtual private network)** 软件

主要用在不安全的公共网络中访问公司的内部资源，或者穿越放火墙访问墙外的资源

因为 **[OpenVPN][openvpn]** 特性比较全面，在初创的小公司中完全可以替代一台专业的 VPN 硬件，以节省初期的成本，特别是技术驱动型的公司，能用技术简单解决的问题就不要砸钱来解决

前面演示了如何搭建服务端，这里演示一下如何构建 **[OpenVPN][openvpn]** 客户端的过程

参考 **[HOWTO][openvpn_client_install]**

> **Tip:** 当前的版本为 **openvpn 2.4.5**

---

# 操作


## 环境

~~~
[root@h208 ~]# hostnamectl 
   Static hostname: h208
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: ad31c8143bbc4aae8c41fb911a9c7bad
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@h208 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84717sec preferred_lft 84717sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h208 ~]# 
~~~


## 安装 openvpn 软件


首先确保 **epel-release** 已经正常安装

~~~
[root@h208 ~]# rpm -qa | grep vpn
[root@h208 ~]# yum list all | grep openvpn.x86_64
NetworkManager-openvpn.x86_64           1:1.2.6-1.el7                  epel     
kde-plasma-networkmanagement-openvpn.x86_64
openvpn.x86_64                          2.4.5-1.el7                    epel     
[root@h208 ~]# yum install openvpn.x86_64
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media: 
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: centos.exabytes.com.my
Resolving Dependencies
--> Running transaction check
---> Package openvpn.x86_64 0:2.4.5-1.el7 will be installed
--> Processing Dependency: liblz4.so.1()(64bit) for package: openvpn-2.4.5-1.el7.x86_64
--> Processing Dependency: libpkcs11-helper.so.1()(64bit) for package: openvpn-2.4.5-1.el7.x86_64
--> Running transaction check
---> Package lz4.x86_64 0:1.7.3-1.el7 will be installed
---> Package pkcs11-helper.x86_64 0:1.11-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch           Version                Repository    Size
================================================================================
Installing:
 openvpn               x86_64         2.4.5-1.el7            epel         517 k
Installing for dependencies:
 lz4                   x86_64         1.7.3-1.el7            epel          82 k
 pkcs11-helper         x86_64         1.11-3.el7             epel          56 k

Transaction Summary
================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 655 k
Installed size: 1.7 M
Is this ok [y/d/N]: y
Downloading packages:
(1/3): lz4-1.7.3-1.el7.x86_64.rpm                          |  82 kB   00:00     
(2/3): openvpn-2.4.5-1.el7.x86_64.rpm                      | 517 kB   00:00     
(3/3): pkcs11-helper-1.11-3.el7.x86_64.rpm                 |  56 kB   00:00     
--------------------------------------------------------------------------------
Total                                              427 kB/s | 655 kB  00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : pkcs11-helper-1.11-3.el7.x86_64                              1/3 
  Installing : lz4-1.7.3-1.el7.x86_64                                       2/3 
  Installing : openvpn-2.4.5-1.el7.x86_64                                   3/3 
  Verifying  : openvpn-2.4.5-1.el7.x86_64                                   1/3 
  Verifying  : lz4-1.7.3-1.el7.x86_64                                       2/3 
  Verifying  : pkcs11-helper-1.11-3.el7.x86_64                              3/3 

Installed:
  openvpn.x86_64 0:2.4.5-1.el7                                                  

Dependency Installed:
  lz4.x86_64 0:1.7.3-1.el7           pkcs11-helper.x86_64 0:1.11-3.el7          

Complete!
[root@h208 ~]# tree /etc/openvpn/
/etc/openvpn/
├── client
└── server

2 directories, 0 files
[root@h208 ~]# 
~~~



## 拷贝证书

~~~
[root@vpn client]# pwd
/tmp/client
[root@vpn client]# ll
total 60
-rw-------. 1 root root  1151 4月   6 00:17 ca.crt
-rwxr-xr-x. 1 root root 35985 8月  22 2017 easyrsa
-rw-r--r--. 1 root root  4560 9月   3 2015 openssl-1.0.cnf
drwx------. 4 root root    45 4月   5 23:48 pki
-rw-------. 1 root root  4418 4月   6 00:17 testclient.crt
-rw-------. 1 root root  1834 4月   6 00:17 testclient.key
drwxr-xr-x. 2 root root    69 4月   5 22:14 x509-types
[root@vpn client]# scp ca.crt testclient.crt  testclient.key  root@192.168.56.208:/etc/openvpn/client/
The authenticity of host '192.168.56.208 (192.168.56.208)' can't be established.
ECDSA key fingerprint is 0c:52:20:0a:00:e3:1a:5d:c6:fc:79:b3:e8:6e:d6:f1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.208' (ECDSA) to the list of known hosts.
root@192.168.56.208's password: 
ca.crt                                                                                    100% 1151     1.1KB/s   00:00    
testclient.crt                                                                            100% 4418     4.3KB/s   00:00    
testclient.key                                                                            100% 1834     1.8KB/s   00:00    
[root@vpn client]# 
~~~

在客户端进行查看

~~~
[root@h208 ~]# tree /etc/openvpn/client/
/etc/openvpn/client/
├── ca.crt
├── testclient.crt
└── testclient.key

0 directories, 3 files
[root@h208 ~]# 
~~~



## 打开服务端防火墙


~~~
[root@vpn openvpn]# firewall-cmd  --add-port 1194/udp   --permanent 
success
[root@vpn openvpn]#
[root@vpn openvpn]# firewall-cmd  --reload
success
[root@vpn openvpn]# firewall-cmd  --list-all 
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 1194/udp 8080/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@vpn openvpn]#
~~~


## 打开 openvpn server 的内核转发

~~~
[root@vpn openvpn]# sysctl  -a | grep  forwarding
net.ipv4.conf.all.forwarding = 1
net.ipv4.conf.all.mc_forwarding = 0
net.ipv4.conf.default.forwarding = 1
net.ipv4.conf.default.mc_forwarding = 0
net.ipv4.conf.enp0s3.forwarding = 1
net.ipv4.conf.enp0s3.mc_forwarding = 0
net.ipv4.conf.enp0s8.forwarding = 1
net.ipv4.conf.enp0s8.mc_forwarding = 0
net.ipv4.conf.lo.forwarding = 1
net.ipv4.conf.lo.mc_forwarding = 0
net.ipv4.conf.tun0.forwarding = 1
net.ipv4.conf.tun0.mc_forwarding = 0
net.ipv4.conf.virbr0.forwarding = 1
net.ipv4.conf.virbr0.mc_forwarding = 0
net.ipv4.conf.virbr0-nic.forwarding = 1
net.ipv4.conf.virbr0-nic.mc_forwarding = 0
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.all.mc_forwarding = 0
net.ipv6.conf.default.forwarding = 0
net.ipv6.conf.default.mc_forwarding = 0
net.ipv6.conf.enp0s3.forwarding = 0
net.ipv6.conf.enp0s3.mc_forwarding = 0
net.ipv6.conf.enp0s8.forwarding = 0
net.ipv6.conf.enp0s8.mc_forwarding = 0
net.ipv6.conf.lo.forwarding = 0
net.ipv6.conf.lo.mc_forwarding = 0
net.ipv6.conf.tun0.forwarding = 0
net.ipv6.conf.tun0.mc_forwarding = 0
net.ipv6.conf.virbr0.forwarding = 0
net.ipv6.conf.virbr0.mc_forwarding = 0
net.ipv6.conf.virbr0-nic.forwarding = 0
net.ipv6.conf.virbr0-nic.mc_forwarding = 0
[root@vpn openvpn]#
~~~


## 打开 server 端防火墙的转发

~~~
[root@vpn openvpn]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 81776sec preferred_lft 81776sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link tentative dadfailed 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/none 
    inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
       valid_lft forever preferred_lft forever
[root@vpn openvpn]# ip route 
default via 10.0.2.2 dev enp0s3  proto static  metric 100 
10.0.2.0/24 dev enp0s3  proto kernel  scope link  src 10.0.2.15  metric 100 
10.8.0.0/24 via 10.8.0.2 dev tun0 
10.8.0.2 dev tun0  proto kernel  scope link  src 10.8.0.1 
169.254.0.0/16 dev enp0s8  scope link  metric 1003 
192.168.56.0/24 dev enp0s8  proto kernel  scope link  src 192.168.56.210 
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1 
[root@vpn openvpn]# 
[root@vpn openvpn]# ping www.baidu.com
PING www.wshifen.com (103.235.46.39) 56(84) bytes of data.
64 bytes from 103.235.46.39 (103.235.46.39): icmp_seq=1 ttl=63 time=23.1 ms
64 bytes from 103.235.46.39 (103.235.46.39): icmp_seq=2 ttl=63 time=20.4 ms
^C
--- www.wshifen.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1008ms
rtt min/avg/max/mdev = 20.482/21.818/23.154/1.336 ms
[root@vpn openvpn]# 
~~~

在通道上打开转发

并且打开 SNAT

~~~
[root@vpn openvpn]# iptables -A FORWARD -i tun0 -j ACCEPT
[root@vpn openvpn]# iptables -t nat -A POSTROUTING -s 10.8.0.1/24 -o  enp0s3 -j MASQUERADE 
[root@vpn openvpn]# 
~~~



## 客户端禁用公网出口

~~~
[root@h208 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86391sec preferred_lft 86391sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h208 ~]# ip route 
default via 10.0.2.2 dev enp0s3  proto static  metric 100 
10.0.2.0/24 dev enp0s3  proto kernel  scope link  src 10.0.2.15  metric 100 
169.254.0.0/16 dev enp0s8  scope link  metric 1003 
192.168.56.0/24 dev enp0s8  proto kernel  scope link  src 192.168.56.208 
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1 
[root@h208 ~]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=63 time=32.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=63 time=31.8 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 31.897/32.313/32.730/0.453 ms
[root@h208 ~]# ifdown enp0s3
Device 'enp0s3' successfully disconnected.
[root@h208 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h208 ~]# ip route 
169.254.0.0/16 dev enp0s8  scope link  metric 1003 
192.168.56.0/24 dev enp0s8  proto kernel  scope link  src 192.168.56.208 
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1 
[root@h208 ~]# ping 8.8.8.8
connect: Network is unreachable
[root@h208 ~]# ping 114.114.114.114
connect: Network is unreachable
[root@h208 ~]# ping 8.8.8.8
connect: Network is unreachable
[root@h208 ~]# 
~~~

## OpenVPN 客户端配置

~~~
[root@h208 client]# pwd
/etc/openvpn/client
[root@h208 client]# ll
total 20
-rw------- 1 root root 1151 4月  10 23:24 ca.crt
-rw-r--r-- 1 root root  213 4月  10 23:30 client.ovpn
-rw------- 1 root root 4418 4月  10 23:24 testclient.crt
-rw------- 1 root root 1834 4月  10 23:24 testclient.key
[root@h208 client]# vim client.ovpn 
[root@h208 client]# cat client.ovpn 
client
dev tun
proto udp
remote 192.168.56.210 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert testclient.crt 
key testclient.key
remote-cert-tls server
#tls-auth ta.key 1
comp-lzo
verb 3
[root@h208 client]# 
~~~

这里的配置项要与服务端相对应


## 启动客户端

~~~
[root@h208 client]# openvpn  --config client.ovpn
Wed Apr 11 00:10:38 2018 OpenVPN 2.4.5 x86_64-redhat-linux-gnu [Fedora EPEL patched] [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Mar  1 2018
Wed Apr 11 00:10:38 2018 library versions: OpenSSL 1.0.2k-fips  26 Jan 2017, LZO 2.06
Enter Private Key Password: ******
Wed Apr 11 00:10:46 2018 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Wed Apr 11 00:10:46 2018 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.56.210:1194
Wed Apr 11 00:10:46 2018 Socket Buffers: R=[212992->212992] S=[212992->212992]
Wed Apr 11 00:10:46 2018 UDP link local: (not bound)
Wed Apr 11 00:10:46 2018 UDP link remote: [AF_INET]192.168.56.210:1194
Wed Apr 11 00:10:46 2018 TLS: Initial packet from [AF_INET]192.168.56.210:1194, sid=b444e4f7 9d494f37
Wed Apr 11 00:10:46 2018 VERIFY OK: depth=1, CN=testca
Wed Apr 11 00:10:46 2018 VERIFY KU OK
Wed Apr 11 00:10:46 2018 Validating certificate extended key usage
Wed Apr 11 00:10:46 2018 ++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
Wed Apr 11 00:10:46 2018 VERIFY EKU OK
Wed Apr 11 00:10:46 2018 VERIFY OK: depth=0, CN=server
Wed Apr 11 00:10:46 2018 WARNING: 'link-mtu' is used inconsistently, local='link-mtu 1542', remote='link-mtu 1558'
Wed Apr 11 00:10:46 2018 WARNING: 'cipher' is used inconsistently, local='cipher BF-CBC', remote='cipher AES-256-CBC'
Wed Apr 11 00:10:46 2018 WARNING: 'keysize' is used inconsistently, local='keysize 128', remote='keysize 256'
Wed Apr 11 00:10:46 2018 Control Channel: TLSv1.2, cipher TLSv1/SSLv3 ECDHE-RSA-AES256-GCM-SHA384, 2048 bit RSA
Wed Apr 11 00:10:46 2018 [server] Peer Connection Initiated with [AF_INET]192.168.56.210:1194
Wed Apr 11 00:10:47 2018 SENT CONTROL [server]: 'PUSH_REQUEST' (status=1)
Wed Apr 11 00:10:47 2018 PUSH: Received control message: 'PUSH_REPLY,redirect-gateway def1 bypass-dhcp,dhcp-option DNS 8.8.8.8,route 10.8.0.1,topology net30,ping 10,ping-restart 120,ifconfig 10.8.0.6 10.8.0.5,peer-id 0,cipher AES-256-GCM'
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: timers and/or timeouts modified
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: --ifconfig/up options modified
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: route options modified
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: --ip-win32 and/or --dhcp-option options modified
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: peer-id set
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: adjusting link_mtu to 1625
Wed Apr 11 00:10:47 2018 OPTIONS IMPORT: data channel crypto options modified
Wed Apr 11 00:10:47 2018 Data Channel: using negotiated cipher 'AES-256-GCM'
Wed Apr 11 00:10:47 2018 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Wed Apr 11 00:10:47 2018 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Wed Apr 11 00:10:47 2018 ROUTE: default_gateway=UNDEF
Wed Apr 11 00:10:47 2018 TUN/TAP device tun0 opened
Wed Apr 11 00:10:47 2018 TUN/TAP TX queue length set to 100
Wed Apr 11 00:10:47 2018 do_ifconfig, tt->did_ifconfig_ipv6_setup=0
Wed Apr 11 00:10:47 2018 /sbin/ip link set dev tun0 up mtu 1500
Wed Apr 11 00:10:47 2018 /sbin/ip addr add dev tun0 local 10.8.0.6 peer 10.8.0.5
Wed Apr 11 00:10:47 2018 NOTE: unable to redirect default gateway -- Cannot read current default gateway from system
Wed Apr 11 00:10:47 2018 /sbin/ip route add 10.8.0.1/32 via 10.8.0.5
Wed Apr 11 00:10:47 2018 Initialization Sequence Completed
...
...
...
~~~

这个过程需要输入客户端私钥(证书)的密码


**Initialization Sequence Completed** 就代表成功了



## 通过隧道访问公网

~~~
[root@h208 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
9: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/none 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
       valid_lft forever preferred_lft forever
[root@h208 ~]# 
[root@h208 ~]# ip route 
10.8.0.1 via 10.8.0.5 dev tun0 
10.8.0.5 dev tun0  proto kernel  scope link  src 10.8.0.6 
169.254.0.0/16 dev enp0s8  scope link  metric 1003 
192.168.56.0/24 dev enp0s8  proto kernel  scope link  src 192.168.56.208 
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1 
[root@h208 ~]# ping 8.8.8.8
connect: Network is unreachable
[root@h208 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
9: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/none 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
       valid_lft forever preferred_lft forever
[root@h208 ~]# ping 8.8.8.8
connect: Network is unreachable
[root@h208 ~]# ping  10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=0.699 ms
64 bytes from 10.8.0.1: icmp_seq=2 ttl=64 time=1.36 ms
64 bytes from 10.8.0.1: icmp_seq=3 ttl=64 time=1.34 ms
^C
--- 10.8.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.699/1.134/1.360/0.307 ms
[root@h208 ~]# ip route add  0.0.0.0/1 via 10.8.0.5 dev tun0
[root@h208 ~]# ip route 
0.0.0.0/1 via 10.8.0.5 dev tun0 
10.8.0.1 via 10.8.0.5 dev tun0 
10.8.0.5 dev tun0  proto kernel  scope link  src 10.8.0.6 
169.254.0.0/16 dev enp0s8  scope link  metric 1003 
192.168.56.0/24 dev enp0s8  proto kernel  scope link  src 192.168.56.208 
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1 
[root@h208 ~]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=35.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=61 time=36.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=61 time=35.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=61 time=38.1 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 35.309/36.330/38.156/1.182 ms
[root@h208 ~]# 
[root@h208 ~]# ping 114.114.114.114
PING 114.114.114.114 (114.114.114.114) 56(84) bytes of data.
64 bytes from 114.114.114.114: icmp_seq=1 ttl=61 time=232 ms
64 bytes from 114.114.114.114: icmp_seq=2 ttl=61 time=254 ms
^C
--- 114.114.114.114 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 232.480/243.625/254.770/11.145 ms
[root@h208 ~]# 
~~~

从不能访问公网到可以访问公网，只添加了一条默认路由

其实这条默认路由也可以由 server 端直接推送过来

其实，这条路由朝外，就是在穿透防火墙访问外网资源

如果这条路由朝内，就是在穿透防火墙访问内网资源，这两者并无本质区别

致此，已经完成了 linux 下 openvpn 客户端的构建与配置


---

# 总结


客户端的配置相对简单，在服务端管理好转发是关键


* TOC
{:toc}


---



[openvpn]:https://openvpn.net/
[openvpn_client_install]:https://openvpn.net/index.php/access-server/docs/admin-guides/182-how-to-connect-to-access-server-with-linux-clients.html
