---
layout: post
title: "Install OpenVPN"
author:  wilmosfang
date: 2018-03-26 15:09:32
image: '/assets/img/'
excerpt: '安装 OpenVPN'
main-class:  'tools'
color: '#808080'
tags:
 - tools
 - openvpn
 - easyrsa
categories:
 - tools
twitter_text: 'simple process of OpenVPN installation'
introduction: 'installation of OpenVPN'
---



## 前言

**[OpenVPN][openvpn]** 是一款开源的 **VPN(Virtual private network)** 软件

>OpenVPN is a full-featured SSL VPN which implements OSI layer 2 or 3 secure network extension using the industry standard SSL/TLS protocol, supports flexible client authentication methods based on certificates, smart cards, and/or username/password credentials, and allows user or group-specific access control policies using firewall rules applied to the VPN virtual interface. 

在不安全的公共网络中访问公司的内部资源，穿越放火墙访问墙外的资源，都是 VPN 显身手的地方

因为 **[OpenVPN][openvpn]** 特性比较全面，在初创的小公司中完全可以替代一台专业的 VPN 硬件，以节省初期的成本，特别是技术驱动型的公司，能用技术简单解决的问题就不要砸钱来解决

这里演示一下如何构建 **[OpenVPN][openvpn]** 服务的过程

参考 **[HOWTO][openvpn_install]**

> **Tip:** 当前的版本为 **openvpn 2.4.5**

---

# 操作


## 环境

~~~
[root@vpn ~]# hostnamectl 
   Static hostname: vpn
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 38ac177a008e493ba5a4c65d521eff88
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@vpn ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85957sec preferred_lft 85957sec
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
[root@vpn ~]#
~~~


## 安装 epel-release 软件库

~~~
[root@vpn ~]# rpm -qa | grep epel 
[root@vpn ~]# yum list all | grep epel 
epel-release.noarch                         7-9                        extras   
[root@vpn ~]# yum install epel-release.noarch
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.nhanhoa.com
 * c7-media: 
 * extras: centos-hcm.viettelidc.com.vn
 * updates: centos-hcm.viettelidc.com.vn
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-9 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch             Version         Repository        Size
================================================================================
Installing:
 epel-release           noarch           7-9             extras            14 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 14 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-9.noarch.rpm                                |  14 kB   00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-9.noarch                                      1/1 
  Verifying  : epel-release-7-9.noarch                                      1/1 

Installed:
  epel-release.noarch 0:7-9                                                     

Complete!
[root@vpn ~]# 
~~~


## 安装软件包

安装 **openvpn** 和 **easy-rsa**

~~~
[root@vpn ~]# yum list all | egrep "(openvpn|easy-rsa)"
NetworkManager-openvpn.x86_64           1:1.2.6-1.el7                  epel     
NetworkManager-openvpn-gnome.x86_64     1:1.2.6-1.el7                  epel     
easy-rsa.noarch                         3.0.3-1.el7                    epel     
kde-plasma-networkmanagement-openvpn.x86_64
openvpn.x86_64                          2.4.5-1.el7                    epel     
openvpn-auth-ldap.x86_64                2.0.3-15.el7                   epel     
openvpn-devel.x86_64                    2.4.5-1.el7                    epel     
[root@vpn ~]# yum install openvpn easy-rsa
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.nhanhoa.com
 * c7-media: 
 * epel: mirror.smartmedia.net.id
 * extras: centos-hcm.viettelidc.com.vn
 * updates: centos-hcm.viettelidc.com.vn
Resolving Dependencies
--> Running transaction check
---> Package easy-rsa.noarch 0:3.0.3-1.el7 will be installed
---> Package openvpn.x86_64 0:2.4.5-1.el7 will be installed
--> Processing Dependency: libcrypto.so.10(OPENSSL_1.0.2)(64bit) for package: openvpn-2.4.5-1.el7.x86_64
--> Processing Dependency: liblz4.so.1()(64bit) for package: openvpn-2.4.5-1.el7.x86_64
--> Processing Dependency: libpkcs11-helper.so.1()(64bit) for package: openvpn-2.4.5-1.el7.x86_64
--> Running transaction check
---> Package lz4.x86_64 0:1.7.3-1.el7 will be installed
---> Package openssl-libs.x86_64 1:1.0.1e-60.el7_3.1 will be updated
--> Processing Dependency: openssl-libs(x86-64) = 1:1.0.1e-60.el7_3.1 for package: 1:openssl-1.0.1e-60.el7_3.1.x86_64
---> Package openssl-libs.x86_64 1:1.0.2k-8.el7 will be an update
---> Package pkcs11-helper.x86_64 0:1.11-3.el7 will be installed
--> Running transaction check
---> Package openssl.x86_64 1:1.0.1e-60.el7_3.1 will be updated
---> Package openssl.x86_64 1:1.0.2k-8.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch           Version                Repository    Size
================================================================================
Installing:
 easy-rsa              noarch         3.0.3-1.el7            epel          31 k
 openvpn               x86_64         2.4.5-1.el7            epel         517 k
Installing for dependencies:
 lz4                   x86_64         1.7.3-1.el7            epel          82 k
 pkcs11-helper         x86_64         1.11-3.el7             epel          56 k
Updating for dependencies:
 openssl               x86_64         1:1.0.2k-8.el7         base         492 k
 openssl-libs          x86_64         1:1.0.2k-8.el7         base         1.2 M

Transaction Summary
================================================================================
Install  2 Packages (+2 Dependent packages)
Upgrade             ( 2 Dependent packages)

Total download size: 2.3 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for base
warning: /var/cache/yum/x86_64/7/epel/packages/easy-rsa-3.0.3-1.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for easy-rsa-3.0.3-1.el7.noarch.rpm is not installed
(1/6): easy-rsa-3.0.3-1.el7.noarch.rpm                     |  31 kB   00:00     
(2/6): lz4-1.7.3-1.el7.x86_64.rpm                          |  82 kB   00:00     
(3/6): pkcs11-helper-1.11-3.el7.x86_64.rpm                 |  56 kB   00:00     
(4/6): openssl-1.0.2k-8.el7.x86_64.rpm                     | 492 kB   00:00     
(5/6): openssl-libs-1.0.2k-8.el7.x86_64.rpm                | 1.2 MB   00:00     
(6/6): openvpn-2.4.5-1.el7.x86_64.rpm                      | 517 kB   00:06     
--------------------------------------------------------------------------------
Total                                              383 kB/s | 2.3 MB  00:06     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:openssl-libs-1.0.2k-8.el7.x86_64                           1/8 
  Updating   : 1:openssl-1.0.2k-8.el7.x86_64                                2/8 
  Installing : pkcs11-helper-1.11-3.el7.x86_64                              3/8 
  Installing : lz4-1.7.3-1.el7.x86_64                                       4/8 
  Installing : openvpn-2.4.5-1.el7.x86_64                                   5/8 
  Installing : easy-rsa-3.0.3-1.el7.noarch                                  6/8 
  Cleanup    : 1:openssl-1.0.1e-60.el7_3.1.x86_64                           7/8 
  Cleanup    : 1:openssl-libs-1.0.1e-60.el7_3.1.x86_64                      8/8 
  Verifying  : openvpn-2.4.5-1.el7.x86_64                                   1/8 
  Verifying  : 1:openssl-1.0.2k-8.el7.x86_64                                2/8 
  Verifying  : easy-rsa-3.0.3-1.el7.noarch                                  3/8 
  Verifying  : 1:openssl-libs-1.0.2k-8.el7.x86_64                           4/8 
  Verifying  : pkcs11-helper-1.11-3.el7.x86_64                              5/8 
  Verifying  : lz4-1.7.3-1.el7.x86_64                                       6/8 
  Verifying  : 1:openssl-1.0.1e-60.el7_3.1.x86_64                           7/8 
  Verifying  : 1:openssl-libs-1.0.1e-60.el7_3.1.x86_64                      8/8 

Installed:
  easy-rsa.noarch 0:3.0.3-1.el7           openvpn.x86_64 0:2.4.5-1.el7          

Dependency Installed:
  lz4.x86_64 0:1.7.3-1.el7           pkcs11-helper.x86_64 0:1.11-3.el7          

Dependency Updated:
  openssl.x86_64 1:1.0.2k-8.el7        openssl-libs.x86_64 1:1.0.2k-8.el7       

Complete!
[root@vpn ~]# echo $?
0
[root@vpn ~]# 
~~~


## easy-rsa 的目录结构

~~~
[root@vpn ~]# rpm -ql easy-rsa-3.0.3-1.el7.noarch
/usr/share/doc/easy-rsa-3.0.3
/usr/share/doc/easy-rsa-3.0.3/COPYING.md
/usr/share/doc/easy-rsa-3.0.3/ChangeLog
/usr/share/doc/easy-rsa-3.0.3/README.quickstart.md
/usr/share/doc/easy-rsa-3.0.3/vars.example
/usr/share/easy-rsa
/usr/share/easy-rsa/3
/usr/share/easy-rsa/3.0
/usr/share/easy-rsa/3.0.3
/usr/share/easy-rsa/3.0.3/easyrsa
/usr/share/easy-rsa/3.0.3/openssl-1.0.cnf
/usr/share/easy-rsa/3.0.3/x509-types
/usr/share/easy-rsa/3.0.3/x509-types/COMMON
/usr/share/easy-rsa/3.0.3/x509-types/ca
/usr/share/easy-rsa/3.0.3/x509-types/client
/usr/share/easy-rsa/3.0.3/x509-types/san
/usr/share/easy-rsa/3.0.3/x509-types/server
/usr/share/licenses/easy-rsa-3.0.3
/usr/share/licenses/easy-rsa-3.0.3/gpl-2.0.txt
[root@vpn ~]# tree /usr/share/easy-rsa/
/usr/share/easy-rsa/
├── 3 -> 3.0.3
├── 3.0 -> 3.0.3
└── 3.0.3
    ├── easyrsa
    ├── openssl-1.0.cnf
    └── x509-types
        ├── ca
        ├── client
        ├── COMMON
        ├── san
        └── server

4 directories, 7 files
[root@vpn ~]# 
~~~



## openvpn 的目录结构

~~~
[root@vpn ~]# rpm -ql openvpn-2.4.5-1.el7.x86_64
/etc/openvpn
/etc/openvpn/client
/etc/openvpn/server
/run/openvpn-client
/run/openvpn-server
/usr/lib/systemd/system/openvpn-client@.service
/usr/lib/systemd/system/openvpn-server@.service
/usr/lib/systemd/system/openvpn@.service
/usr/lib/tmpfiles.d/openvpn.conf
/usr/lib64/openvpn
/usr/lib64/openvpn/plugins
/usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so
/usr/lib64/openvpn/plugins/openvpn-plugin-down-root.so
/usr/sbin/openvpn
/usr/share/doc/openvpn-2.4.5
/usr/share/doc/openvpn-2.4.5/AUTHORS
/usr/share/doc/openvpn-2.4.5/COPYING
/usr/share/doc/openvpn-2.4.5/COPYRIGHT.GPL
/usr/share/doc/openvpn-2.4.5/ChangeLog
/usr/share/doc/openvpn-2.4.5/Changes.rst
/usr/share/doc/openvpn-2.4.5/README
/usr/share/doc/openvpn-2.4.5/README.auth-pam
/usr/share/doc/openvpn-2.4.5/README.down-root
/usr/share/doc/openvpn-2.4.5/README.systemd
/usr/share/doc/openvpn-2.4.5/contrib
/usr/share/doc/openvpn-2.4.5/contrib/OCSP_check
/usr/share/doc/openvpn-2.4.5/contrib/OCSP_check/OCSP_check.sh
/usr/share/doc/openvpn-2.4.5/contrib/README
/usr/share/doc/openvpn-2.4.5/contrib/openvpn-fwmarkroute-1.00
/usr/share/doc/openvpn-2.4.5/contrib/openvpn-fwmarkroute-1.00/README
/usr/share/doc/openvpn-2.4.5/contrib/openvpn-fwmarkroute-1.00/fwmarkroute.down
/usr/share/doc/openvpn-2.4.5/contrib/openvpn-fwmarkroute-1.00/fwmarkroute.up
/usr/share/doc/openvpn-2.4.5/contrib/pull-resolv-conf
/usr/share/doc/openvpn-2.4.5/contrib/pull-resolv-conf/client.down
/usr/share/doc/openvpn-2.4.5/contrib/pull-resolv-conf/client.up
/usr/share/doc/openvpn-2.4.5/management-notes.txt
/usr/share/doc/openvpn-2.4.5/sample
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/README
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/client.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/firewall.sh
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/home.up
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/loopback-client
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/loopback-server
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/office.up
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/openvpn-shutdown.sh
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/openvpn-startup.sh
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/roadwarrior-client.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/roadwarrior-server.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/server.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/static-home.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/static-office.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/tls-home.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/tls-office.conf
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/xinetd-client-config
/usr/share/doc/openvpn-2.4.5/sample/sample-config-files/xinetd-server-config
/usr/share/doc/openvpn-2.4.5/sample/sample-scripts
/usr/share/doc/openvpn-2.4.5/sample/sample-scripts/auth-pam.pl
/usr/share/doc/openvpn-2.4.5/sample/sample-scripts/bridge-start
/usr/share/doc/openvpn-2.4.5/sample/sample-scripts/bridge-stop
/usr/share/doc/openvpn-2.4.5/sample/sample-scripts/ucn.pl
/usr/share/doc/openvpn-2.4.5/sample/sample-scripts/verify-cn
/usr/share/doc/openvpn-2.4.5/sample/sample-windows
/usr/share/doc/openvpn-2.4.5/sample/sample-windows/sample.ovpn
/usr/share/man/man8/openvpn.8.gz
/var/lib/openvpn
[root@vpn ~]# tree /etc/openvpn
/etc/openvpn
├── client
└── server

2 directories, 0 files
[root@vpn ~]# 
~~~



## 调整 openvpn 配置

~~~
[root@vpn ~]# cp  /usr/share/doc/openvpn-2.4.5/sample/sample-config-files/server.conf /etc/openvpn/
[root@vpn ~]# grep  -v "#" /etc/openvpn/server.conf | grep -v ';'| cat -s

port 1194

proto udp

dev tun

ca ca.crt
cert server.crt

dh dh2048.pem

server 10.8.0.0 255.255.255.0

ifconfig-pool-persist ipp.txt

keepalive 10 120

cipher AES-256-CBC

persist-key
persist-tun

status openvpn-status.log

verb 3

explicit-exit-notify 1
[root@vpn ~]# 
[root@vpn ~]# vim /etc/openvpn/server.conf 
[root@vpn ~]# grep  -v "#" /etc/openvpn/server.conf | grep -v ';'| cat -s

local 192.168.56.210

port 1194

proto udp

dev tun

ca ca.crt
cert server.crt

dh dh2048.pem

server 10.8.0.0 255.255.255.0

ifconfig-pool-persist ipp.txt

push "redirect-gateway def1 bypass-dhcp"

push "dhcp-option DNS 8.8.8.8"

keepalive 10 120

cipher AES-256-CBC

comp-lzo

max-clients 100

user nobody
group nobody

persist-key
persist-tun

status openvpn-status.log

log-append  openvpn.log

verb 3

explicit-exit-notify 1
[root@vpn ~]#  
~~~



## 准备 easy-rsa 环境


~~~
[root@vpn ~]# mkdir -p  /etc/openvpn/easy-rsa/keys
[root@vpn ~]# cp -a /usr/share/easy-rsa/3/* /etc/openvpn/easy-rsa/
[root@vpn ~]# tree /etc/openvpn/easy-rsa/
/etc/openvpn/easy-rsa/
├── easyrsa
├── keys
├── openssl-1.0.cnf
└── x509-types
    ├── ca
    ├── client
    ├── COMMON
    ├── san
    └── server

2 directories, 7 files
[root@vpn ~]# 
~~~



## 配置 easy-rsa


~~~
[root@vpn ~]# cp  /usr/share/doc/easy-rsa-3.0.3/vars.example /etc/openvpn/easy-rsa/vars
[root@vpn ~]# cat /etc/openvpn/easy-rsa/vars  | grep -v "^#"  | cat -s 

if [ -z "$EASYRSA_CALLER" ]; then
	echo "You appear to be sourcing an Easy-RSA 'vars' file." >&2
	echo "This is no longer necessary and is disallowed. See the section called" >&2
	echo "'How to use this file' near the top comments for more details." >&2
	return 1
fi

[root@vpn ~]# vim  /etc/openvpn/easy-rsa/vars
[root@vpn ~]# cat /etc/openvpn/easy-rsa/vars  | grep -v "^#"  | cat -s 

if [ -z "$EASYRSA_CALLER" ]; then
	echo "You appear to be sourcing an Easy-RSA 'vars' file." >&2
	echo "This is no longer necessary and is disallowed. See the section called" >&2
	echo "'How to use this file' near the top comments for more details." >&2
	return 1
fi

set_var EASYRSA_REQ_COUNTRY	"CN"
set_var EASYRSA_REQ_PROVINCE	"Shanghai"
set_var EASYRSA_REQ_CITY	"pudong"
set_var EASYRSA_REQ_ORG	 	"testORG"
set_var EASYRSA_REQ_EMAIL	"me@example.com"
set_var EASYRSA_REQ_OU		"testOU"

[root@vpn ~]# 
~~~



## 初始化 pki

~~~
[root@vpn easy-rsa]# pwd
/etc/openvpn/easy-rsa
[root@vpn easy-rsa]# ls
easyrsa  keys  openssl-1.0.cnf  vars  x509-types
[root@vpn easy-rsa]# ./easyrsa init-pki

Note: using Easy-RSA configuration from: ./vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/pki

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# ls
easyrsa  keys  openssl-1.0.cnf  pki  vars  x509-types
[root@vpn easy-rsa]# tree pki/
pki/
├── private
└── reqs

2 directories, 0 files
[root@vpn easy-rsa]# 
~~~


## 创建 ca

~~~
[root@vpn easy-rsa]# ./easyrsa build-ca

Note: using Easy-RSA configuration from: ./vars
Generating a 2048 bit RSA private key
............................+++
............+++
writing new private key to '/etc/openvpn/easy-rsa/pki/private/ca.key.H5XqMsPPWo'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:testca

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easy-rsa/pki/ca.crt

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# 
~~~

这里的 **Common Name** 是 CA 服务器的 

配置签发密码




## 创建服务端证书

~~~
[root@vpn easy-rsa]# ./easyrsa gen-req server nopass

Note: using Easy-RSA configuration from: ./vars
Generating a 2048 bit RSA private key
..................+++
....................+++
writing new private key to '/etc/openvpn/easy-rsa/pki/private/server.key.rApaa1e3aS'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa/pki/reqs/server.req
key: /etc/openvpn/easy-rsa/pki/private/server.key

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# 
~~~


这里的 **Common Name** 是服务器的，与 CA 的不同


## 通过 CA 证书来签发 server 证书

~~~
[root@vpn easy-rsa]# ./easyrsa sign server server

Note: using Easy-RSA configuration from: ./vars


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 3650 days:

subject=
    commonName                = server


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from ./openssl-1.0.cnf
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Apr  2 15:35:21 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/easy-rsa/pki/issued/server.crt

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# 
~~~


此过程中有一步是需要确认的，代表确认签发服务证书

因为需要 CA 证书来签发服务证书，所以这里需要输入 CA 证书的密码，代表授权




## 创建Diffie-Hellman


~~~
[root@vpn easy-rsa]# ./easyrsa gen-dh

Note: using Easy-RSA configuration from: ./vars
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
.......................................................................................................................................+...............+...........................................................................................................................................................................................................................................+....................+....................................................................................................................................................................................................................................+..............................................................................................+..............+....................................................+..........................................................+...........................................................................+..............................++*++*

DH parameters of size 2048 created at /etc/openvpn/easy-rsa/pki/dh.pem

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# 
~~~



## 准备创建客户端证书


先创拷贝过来 easy-rsa 目录

~~~
[root@vpn tmp]# cd client/
[root@vpn client]# ls
[root@vpn client]# cp -a /usr/share/easy-rsa/3/* /tmp/client/
[root@vpn client]# cd /tmp/client/
[root@vpn client]# ls
easyrsa  openssl-1.0.cnf  x509-types
[root@vpn client]# ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /tmp/client/pki

[root@vpn client]# echo $?
0
[root@vpn client]#
~~~


## 创建客户端证书请求文件

~~~
[root@vpn client]# ./easyrsa gen-req testclient
Generating a 2048 bit RSA private key
................................................................................................................+++
.......................................................................................................+++
writing new private key to '/tmp/client/pki/private/testclient.key.dtCoPvK5Ne'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [testclient]:

Keypair and certificate request completed. Your files are:
req: /tmp/client/pki/reqs/testclient.req
key: /tmp/client/pki/private/testclient.key

[root@vpn client]# echo $?
0
[root@vpn client]# 
~~~


## 导入客户端证书请求文件


~~~
[root@vpn client]# tree pki
pki
├── private
│   └── testclient.key
└── reqs
    └── testclient.req

2 directories, 2 files
[root@vpn client]# cd /etc/openvpn/easy-rsa/
[root@vpn easy-rsa]# ./easyrsa import-req /tmp/client/pki/reqs/testclient.req  testclient

Note: using Easy-RSA configuration from: ./vars

The request has been successfully imported with a short name of: testclient
You may now use this name to perform signing operations on this request.

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# 
~~~


## 签发客户端证书

~~~
[root@vpn easy-rsa]# ./easyrsa sign client testclient

Note: using Easy-RSA configuration from: ./vars


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 3650 days:

subject=
    commonName                = testclient


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from ./openssl-1.0.cnf
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'testclient'
Certificate is to be certified until Apr  2 16:06:55 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/easy-rsa/pki/issued/testclient.crt

[root@vpn easy-rsa]# echo $?
0
[root@vpn easy-rsa]# 
~~~


## 证书使用情况

~~~
[root@vpn easy-rsa]# tree pki
pki
├── ca.crt
├── certs_by_serial
│   ├── 53A817EECB6D232ABCD565377589EEEF.pem
│   └── CF5609BE48BBCE40D153A0B1181B9141.pem
├── dh.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│   ├── server.crt
│   └── testclient.crt
├── private
│   ├── ca.key
│   └── server.key
├── reqs
│   ├── server.req
│   └── testclient.req
├── serial
└── serial.old

4 directories, 16 files
[root@vpn easy-rsa]# tree /tmp/client/pki/
/tmp/client/pki/
├── private
│   └── testclient.key
└── reqs
    └── testclient.req

2 directories, 2 files
[root@vpn easy-rsa]# 
~~~


## 拷贝证书到一起


### 拷贝服务端证书

~~~
[root@vpn easy-rsa]# cp pki/ca.crt  pki/private/server.key  pki/issued/server.crt  pki/dh.pem  /etc/openvpn/
[root@vpn easy-rsa]# ll /etc/openvpn/
total 32
-rw-------. 1 root root     1151 Apr  6 00:14 ca.crt
drwxr-x---. 2 root openvpn     6 Mar  2 00:21 client
-rw-------. 1 root root      424 Apr  6 00:14 dh.pem
drwxr-xr-x. 5 root root       97 Apr  5 23:16 easy-rsa
drwxr-x---. 2 root openvpn     6 Mar  2 00:21 server
-rw-r--r--. 1 root root    10952 Apr  5 22:56 server.conf
-rw-------. 1 root root     4525 Apr  6 00:14 server.crt
-rw-------. 1 root root     1704 Apr  6 00:14 server.key
[root@vpn easy-rsa]#
~~~

### 拷贝客户端证书

~~~
[root@vpn easy-rsa]# cp pki/ca.crt pki/issued/testclient.crt  /tmp/client/pki/private/testclient.key  /tmp/client/
[root@vpn easy-rsa]# ll /tmp/client/
total 60
-rw-------. 1 root root  1151 Apr  6 00:17 ca.crt
-rwxr-xr-x. 1 root root 35985 Aug 22  2017 easyrsa
-rw-r--r--. 1 root root  4560 Sep  3  2015 openssl-1.0.cnf
drwx------. 4 root root    45 Apr  5 23:48 pki
-rw-------. 1 root root  4418 Apr  6 00:17 testclient.crt
-rw-------. 1 root root  1834 Apr  6 00:17 testclient.key
drwxr-xr-x. 2 root root    69 Apr  5 22:14 x509-types
[root@vpn easy-rsa]#
~~~


## 启动服务

~~~
[root@vpn openvpn]# openvpn  --config  /etc/openvpn/server.conf 
...
...
...
~~~

查看日志 **`/etc/openvpn/openvpn.log`**

~~~
Fri Apr  6 00:31:22 2018 OpenVPN 2.4.5 x86_64-redhat-linux-gnu [Fedora EPEL patched] [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Mar  1 2018
Fri Apr  6 00:31:22 2018 library versions: OpenSSL 1.0.2k-fips  26 Jan 2017, LZO 2.06
Fri Apr  6 00:31:22 2018 Diffie-Hellman initialized with 2048 bit key
Fri Apr  6 00:31:22 2018 ROUTE_GATEWAY 10.0.2.2/255.255.255.0 IFACE=enp0s3 HWADDR=08:00:27:f9:30:bb
Fri Apr  6 00:31:22 2018 TUN/TAP device tun0 opened
Fri Apr  6 00:31:22 2018 TUN/TAP TX queue length set to 100
Fri Apr  6 00:31:22 2018 do_ifconfig, tt->did_ifconfig_ipv6_setup=0
Fri Apr  6 00:31:22 2018 /sbin/ip link set dev tun0 up mtu 1500
Fri Apr  6 00:31:22 2018 /sbin/ip addr add dev tun0 local 10.8.0.1 peer 10.8.0.2
Fri Apr  6 00:31:22 2018 /sbin/ip route add 10.8.0.0/24 via 10.8.0.2
Fri Apr  6 00:31:22 2018 Could not determine IPv4/IPv6 protocol. Using AF_INET
Fri Apr  6 00:31:22 2018 Socket Buffers: R=[212992->212992] S=[212992->212992]
Fri Apr  6 00:31:22 2018 UDPv4 link local (bound): [AF_INET]192.168.56.210:1194
Fri Apr  6 00:31:22 2018 UDPv4 link remote: [AF_UNSPEC]
Fri Apr  6 00:31:22 2018 GID set to nobody
Fri Apr  6 00:31:22 2018 UID set to nobody
Fri Apr  6 00:31:22 2018 MULTI: multi_init called, r=256 v=256
Fri Apr  6 00:31:22 2018 IFCONFIG POOL: base=10.8.0.4 size=62, ipv6=0
Fri Apr  6 00:31:22 2018 IFCONFIG POOL LIST
Fri Apr  6 00:31:22 2018 Initialization Sequence Completed
~~~


查看日志 **`/etc/openvpn/openvpn-status.log`**


~~~
OpenVPN CLIENT LIST
Updated,Fri Apr  6 00:36:32 2018
Common Name,Real Address,Bytes Received,Bytes Sent,Connected Since
ROUTING TABLE
Virtual Address,Common Name,Real Address,Last Ref
GLOBAL STATS
Max bcast/mcast queue length,0
END
~~~


查看进程状态

~~~
[root@vpn openvpn]# ps faux | grep openvpn
nobody    5289  0.0  0.1  74948  3968 pts/0    S+   00:31   0:00  |               \_ openvpn --config /etc/openvpn/server.conf
root      5197  0.0  0.0 107936   608 pts/1    S+   00:29   0:00  |       \_ tail -f openvpn.log
root      5451  0.0  0.0 112648  1012 pts/2    S+   00:38   0:00          \_ grep --color=auto openvpn
[root@vpn openvpn]# netstat  -anup 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
udp        0      0 0.0.0.0:53194           0.0.0.0:*                           800/avahi-daemon: r 
udp        0      0 0.0.0.0:56287           0.0.0.0:*                           1256/dhclient       
udp        0      0 192.168.122.1:53        0.0.0.0:*                           1740/dnsmasq        
udp        0      0 0.0.0.0:67              0.0.0.0:*                           1740/dnsmasq        
udp        0      0 0.0.0.0:68              0.0.0.0:*                           1256/dhclient       
udp        0      0 192.168.56.210:1194     0.0.0.0:*                           5289/openvpn        
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           800/avahi-daemon: r 
udp6       0      0 :::9398                 :::*                                1256/dhclient       
[root@vpn openvpn]# 
~~~



到此 openvpn 服务端的配置与安装已经完成了



---

# 总结


因为涉及到 CA 的证书签发流程，所以显得有些复杂



* TOC
{:toc}


---



[openvpn]:https://openvpn.net/
[openvpn_install]:https://openvpn.net/index.php/open-source/documentation/howto.html
[openvpn_software]:https://openvpn.net/index.php/access-server/overview.html
[openvpn_as]:https://openvpn.net/index.php/access-server/download-openvpn-as-sw.html



