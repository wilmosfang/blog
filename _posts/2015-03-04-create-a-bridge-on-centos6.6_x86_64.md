---
layout: default
title: 在Centos6.6中创建一个Bridge
---

##前言

大部分玩**KVM**的人一定都会遇到同一个问题，那就是给当前系统创建一个**Bridge**(Bridge 可以理解为软件模拟出的网桥，网桥又可以理解为简单的交换机，如果还不明白建议用Baidu去Google一把)

##配置

下面配置很少，但是花了我整整一个下午的时间排错，其根本原因不是配置错误，而是与**Network Manager**服务的冲突，关掉它这个世界立刻变得平静而祥和

```bash

[test@test network-scripts]$ grep -v "#" ifcfg-eth0 | cat -s 

DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BRIDGE=br0

[test@test network-scripts]$ grep -v "#" ifcfg-br0 | cat -s 

DEVICE=br0
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.45
NETMASK=255.255.255.0
GATEWAY=192.168.1.2

[test@test network-scripts]$ /etc/init.d/NetworkManager status
NetworkManager is stopped
[test@test network-scripts]$ chkconfig  --list NetworkManager 
NetworkManager 	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[test@test network-scripts]$ grep -v "#" /etc/rc.local | cat -s 

touch /var/lock/subsys/local

ifup br0
[test@test network-scripts]$ 


```

之所以在rc.local中添加**ifup br0**，是因为操作系统重启后理应自动启动的**br0**会启动不了，所以使用这个方法来确保启动

##效果

下面是重启网络后的效果

```bash

[root@test ~]# /etc/init.d/network  restart 
Shutting down interface br0:                               [  OK  ]
Shutting down interface eth0:                              [  OK  ]
Shutting down loopback interface:                          [  OK  ]
Bringing up loopback interface:                            [  OK  ]
Bringing up interface br0:  Determining if ip address 192.168.1.45 is already in use for device br0...
                                                           [  OK  ]
Bringing up interface eth0:                                [  OK  ]
[root@test ~]# ifconfig 
br0       Link encap:Ethernet  HWaddr 2C:27:D7:46:CF:12  
          inet addr:192.168.1.45  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::fc54:ff:fe12:1e4a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1182852 errors:0 dropped:0 overruns:0 frame:0
          TX packets:244945 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:526104157 (501.7 MiB)  TX bytes:25898911 (24.6 MiB)

eth0      Link encap:Ethernet  HWaddr 2C:27:D7:46:CF:12  
          inet6 addr: fe80::2e27:d7ff:fe46:cf12/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1333346 errors:0 dropped:0 overruns:0 frame:0
          TX packets:261701 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:598101735 (570.3 MiB)  TX bytes:28345108 (27.0 MiB)
          Interrupt:20 Memory:fb000000-fb020000 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:39961 errors:0 dropped:0 overruns:0 frame:0
          TX packets:39961 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:89126489 (84.9 MiB)  TX bytes:89126489 (84.9 MiB)

virbr0    Link encap:Ethernet  HWaddr 52:54:00:01:D6:47  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

vnet0     Link encap:Ethernet  HWaddr FE:54:00:12:1E:4A  
          inet6 addr: fe80::fc54:ff:fe12:1e4a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:17764 errors:0 dropped:0 overruns:0 frame:0
          TX packets:121156 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:500 
          RX bytes:1279243 (1.2 MiB)  TX bytes:54399248 (51.8 MiB)

[root@test ~]# ip addr 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 2c:27:d7:46:cf:12 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::2e27:d7ff:fe46:cf12/64 scope link 
       valid_lft forever preferred_lft forever
3: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
    link/ether 2c:27:d7:46:cf:12 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.45/24 brd 192.168.1.255 scope global br0
    inet6 fe80::fc54:ff:fe12:1e4a/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
    link/ether 52:54:00:01:d6:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 500
    link/ether 52:54:00:01:d6:47 brd ff:ff:ff:ff:ff:ff
9: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 500
    link/ether fe:54:00:12:1e:4a brd ff:ff:ff:ff:ff:ff
    inet6 fe80::fc54:ff:fe12:1e4a/64 scope link 
       valid_lft forever preferred_lft forever
[root@test ~]# 

```

##补充

selinux有时也会造成影响，需要关闭

查看方法

```bash
[root@test ~]# sestatus  -v 
SELinux status:                 disabled
[root@test ~]# getenforce 
Disabled
[root@test ~]# cat /etc/se
securetty      security/      selinux/       services       sestatus.conf  setuptool.d/   
[root@test ~]# cat /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
#SELINUX=enforcing
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 


[root@test ~]# 

```

关闭方法 

```bash

setenforce 0

```

