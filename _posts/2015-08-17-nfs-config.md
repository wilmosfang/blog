---
layout: post
title: NFS 配置
categories: linux nfs
wc: 485 1341 14805
excerpt: follow me
comments: true
---

前言
=

NFS(Network File System)是Linux中使用非常频繁的一文件系统共享方式，今天重新研究了一下，略有收货，分享出来

---

# 概要

* TOC
{:toc}


---


依赖
-

NFS依赖于RPC(Remote Procedure Call)，也就是NFS服务运行之前，要确保RPC的正常运行，因为NFS要向RPC注册端口。

> **Tip:** 类似NFS的还有NIS，这一类服务也叫RPC server,它们在启动时会随机选取一个端口，然后主动向RPC注册，所以RPC就知道所有PRC server的服务端口，然后RPC固定在111端口进行监听，客户端连接时，首先向服务端的111询问RPC server的服务端口，获得真正端口后，再去连接真实服务端口。

---

## 包


RPC 服务：rpcbind (Centos6.x 下) / portmap (Centos5.x 下)

NFS 服务：nfs-utils

{% highlight bash %}
[root@Centos6.x ~]# rpm -qa | grep -E '(rpcbind|nfs|portmap)'
nfs-utils-lib-1.1.5-11.el6.x86_64
nfs-utils-1.2.3-64.el6.x86_64
nfs4-acl-tools-0.3.3-7.el6.x86_64
rpcbind-0.2.0-11.el6.x86_64
[root@Centos6.x ~]# 


[root@Centos5.x ~]# rpm -qa | grep -E '(rpcbind|nfs|portmap)' 
nfs-utils-1.0.9-70.el5
nfs-utils-lib-1.0.8-7.9.el5
portmap-4.0-65.2.2.1
[root@Centos5.x ~]# 
{% endhighlight %}

> **Tip:**   NFS 会产生以下进程
 rpc.nfsd 主服务
 rpc.mountd 相关权限审核
 rpc.lockd 管理文件锁
 rpc.statd 文件一致性检查 

 ---

## 权限


服务端和客户端都是根据用户名来查UID，GID然后通过UID，GID来判别读写权限

满足以下条件才能进行正常操作

*  UID有相应权限(用户ID层面)
*  NFS服务有相应权限 (exportfs 配置)
*  服务端文件系统有相应权限 (文件系统层面)


---

## 配置


NFS配置文件
-

NFS使用 **/etc/exports** 作为配置文件

{% highlight bash %}
[root@test ~]# cat /etc/exports 
/data/nfs     192.168.1.115(rw,sync,no_root_squash)  192.168.1.0/24(ro)
[root@test ~]# 
{% endhighlight %}

**rw** :代表可读写

**sync** :代表同步到硬盘，相比async更慢，但更可靠

**no_root_squash** :代表root不进行匿名替换，保留root权限


> **Tip:**  使用 **man exports** 可以看到更详细的权限配置 
可以使用/255.255.255.0 来代替/24

---

## 启动服务


分别启动下面几个服务

*  **/etc/init.d/rpcbind** (Centos6 与Centos5不同 ,Centos5 下是portmap)
* **/etc/init.d/nfs**
* **/etc/init.d/nfslock**

{% highlight bash %}
[root@test data]# /etc/init.d/rpcbind  status
rpcbind (pid  2338) is running...
[root@test data]# /etc/init.d/nfs status
rpc.svcgssd is stopped
rpc.mountd is stopped
nfsd is stopped
rpc.rquotad is stopped
[root@test data]# /etc/init.d/nfs start 
Starting NFS services:  [  OK  ]

Starting NFS quotas: [  OK  ]

Starting NFS mountd: [  OK  ]

Starting NFS daemon: [  OK  ]

Starting RPC idmapd: [  OK  ]

[root@test ~]# /etc/init.d/nfslock  status
rpc.statd (pid  2360) is running...
[root@test ~]# 
{% endhighlight %}

成功启动后111端口是监听状态

{% highlight bash %}
[root@test ~]# netstat -an | grep 111
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      
tcp        0      0 :::111                      :::*                        LISTEN      
udp        0      0 0.0.0.0:111                 0.0.0.0:*                               
udp        0      0 :::111                      :::*                                    
[root@test ~]# 
{% endhighlight %}


> **Tip:**  NFS的启动信息默认会写到 **/var/log/messages** 里


NFS的服务会打开很多端口(并且其中一些端口是随机的)

{% highlight bash %}
[root@test data]# netstat -tulnp | grep -E '(rpc|nfs)'
tcp        0      0 0.0.0.0:34146               0.0.0.0:*                   LISTEN      37900/rpc.mountd    
tcp        0      0 0.0.0.0:44934               0.0.0.0:*                   LISTEN      37900/rpc.mountd    
tcp        0      0 0.0.0.0:56230               0.0.0.0:*                   LISTEN      2360/rpc.statd      
tcp        0      0 0.0.0.0:875                 0.0.0.0:*                   LISTEN      37895/rpc.rquotad   
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      2338/rpcbind        
tcp        0      0 0.0.0.0:48980               0.0.0.0:*                   LISTEN      37900/rpc.mountd    
tcp        0      0 :::51424                    :::*                        LISTEN      37900/rpc.mountd    
tcp        0      0 :::35175                    :::*                        LISTEN      37900/rpc.mountd    
tcp        0      0 :::111                      :::*                        LISTEN      2338/rpcbind        
tcp        0      0 :::45202                    :::*                        LISTEN      2360/rpc.statd      
tcp        0      0 :::41402                    :::*                        LISTEN      37900/rpc.mountd    
udp        0      0 0.0.0.0:34189               0.0.0.0:*                               37900/rpc.mountd    
udp        0      0 0.0.0.0:60207               0.0.0.0:*                               37900/rpc.mountd    
udp        0      0 0.0.0.0:817                 0.0.0.0:*                               2338/rpcbind        
udp        0      0 0.0.0.0:48316               0.0.0.0:*                               2360/rpc.statd      
udp        0      0 0.0.0.0:54461               0.0.0.0:*                               37900/rpc.mountd    
udp        0      0 127.0.0.1:840               0.0.0.0:*                               2360/rpc.statd      
udp        0      0 0.0.0.0:875                 0.0.0.0:*                               37895/rpc.rquotad   
udp        0      0 0.0.0.0:111                 0.0.0.0:*                               2338/rpcbind        
udp        0      0 :::54948                    :::*                                    37900/rpc.mountd    
udp        0      0 :::817                      :::*                                    2338/rpcbind        
udp        0      0 :::43853                    :::*                                    37900/rpc.mountd    
udp        0      0 :::52049                    :::*                                    37900/rpc.mountd    
udp        0      0 :::111                      :::*                                    2338/rpcbind        
udp        0      0 :::36981                    :::*                                    2360/rpc.statd      
[root@test data]# 
{% endhighlight %}

使用下面方法可以看到本机的rpc信息

{% highlight bash %}
[root@test data]# rpcinfo -p localhost
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  48316  status
    100024    1   tcp  56230  status
    100011    1   udp    875  rquotad
    100011    2   udp    875  rquotad
    100011    1   tcp    875  rquotad
    100011    2   tcp    875  rquotad
    100005    1   udp  54461  mountd
    100005    1   tcp  34146  mountd
    100005    2   udp  34189  mountd
    100005    2   tcp  44934  mountd
    100005    3   udp  60207  mountd
    100005    3   tcp  48980  mountd
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    2   tcp   2049  nfs_acl
    100227    3   tcp   2049  nfs_acl
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    2   udp   2049  nfs_acl
    100227    3   udp   2049  nfs_acl
    100021    1   udp  38623  nlockmgr
    100021    3   udp  38623  nlockmgr
    100021    4   udp  38623  nlockmgr
    100021    1   tcp  59092  nlockmgr
    100021    3   tcp  59092  nlockmgr
    100021    4   tcp  59092  nlockmgr
[root@test data]# 

{% endhighlight %}

其中

* rpcbind 在UDP TCP的111上
* NFS在UDP TCP的2049上
* 其它服务都是在UDP TCP的随机端口上进行监听(rpcbind知道在哪里)

那为什么是三个NFS 服务呢

{% highlight bash %}
[root@test ~]# rpcinfo -u  localhost nfs 
program 100003 version 2 ready and waiting
program 100003 version 3 ready and waiting
program 100003 version 4 ready and waiting
[root@test ~]# rpcinfo -t  localhost nfs 
program 100003 version 2 ready and waiting
program 100003 version 3 ready and waiting
program 100003 version 4 ready and waiting
[root@test ~]# 

[root@abc ~]# rpcinfo -t test nfs 
program 100003 version 2 ready and waiting
program 100003 version 3 ready and waiting
program 100003 version 4 ready and waiting
[root@abc ~]# rpcinfo -u test nfs 
program 100003 version 2 ready and waiting
program 100003 version 3 ready and waiting
program 100003 version 4 ready and waiting
[root@abc ~]# 
{% endhighlight %}

因为为了更好的兼容，NFS开启了三个版本进行响应

---

## 查看NFS共享


使用 **showmount** 查看NFS共享

这个工具同样依赖rpcbind

{% highlight bash %}
[root@nfs-server ~]# showmount  -a localhost
All mount points on localhost:
192.168.1.215:/data/nfs
[root@nfs-server ~]# showmount  -e localhost
Export list for localhost:
/data/nfs 192.168.1.0/24
[root@nfs-server ~]# cat /etc/exports 
/data/nfs     192.168.1.215(rw,sync,no_root_squash)  192.168.1.0/24(ro)
[root@nfs-server ~]# 

[root@nfs-client ~]# showmount -e nfs-server 
Export list for nfs-server:
/data/nfs 192.168.1.0/24
[root@nfs-client ~]# showmount -a nfs-server 
All mount points on nfs-server:
192.168.1.215:/data/nfs
[root@nfs-client ~]#
{% endhighlight %}

如果想查看当前比较全面的参数，可以通过 **/var/lib/nfs/etab**


{% highlight bash %}
[root@nfs-server ~]# cat /var/lib/nfs/etab 
/data/nfs	192.168.1.215(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_ocks,acl,anonuid=65534,anongid=65534,sec=sys,rw,no_root_squash,no_all_squash)
/data/nfs	192.168.1.0/24(ro,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_lcks,acl,anonuid=65534,anongid=65534,sec=sys,ro,root_squash,no_all_squash)
[root@nfs-server ~]# 
{% endhighlight %}

---

## 重载NFS配置


使用 **exportfs** 对NFS进行重载

{% highlight bash %}
[root@nfs-server ~]# showmount -e
Export list for nfs-server:
/data/nfs 192.168.1.0/24
[root@nfs-server ~]# exportfs  -ua
[root@nfs-server ~]# showmount -e
Export list for nfs-server:
[root@nfs-server ~]# 
{% endhighlight %}

使用 **exportfs -ra** 可以对 **/etc/exports** 里的内容进行重载

{% highlight bash %}
[root@nfs-server ~]# exportfs  -ra
[root@nfs-server ~]# showmount -e
Export list for nfs-server:
/data/nfs 192.168.1.0/24
[root@nfs-server ~]# 
{% endhighlight %}

---

## NFS防火墙

由于 **RPC** 类的服务，都会随机注册端口，这样就给防火墙的设置造成了困扰，NFS 提供了一个配置文件，可以将要申请注册的端口进行固定~

编辑 **/etc/sysconfig/nfs** , 取消下面行的注释

{% highlight bash %}
[root@nfs-server ~]# grep -v "^#" /etc/sysconfig/nfs 
RQUOTAD_PORT=875
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892
[root@nfs-server ~]# 
{% endhighlight %}

重启NFS服务，然后看看 **rpcinfo**

{% highlight bash %}
[root@nfs-server ~]# rpcinfo  -p | grep -E '(rquota|mount|nlock)'
    100011    1   udp    875  rquotad
    100011    2   udp    875  rquotad
    100011    1   tcp    875  rquotad
    100011    2   tcp    875  rquotad
    100005    1   udp    892  mountd
    100005    1   tcp    892  mountd
    100005    2   udp    892  mountd
    100005    2   tcp    892  mountd
    100005    3   udp    892  mountd
    100005    3   tcp    892  mountd
    100021    1   udp  32769  nlockmgr
    100021    3   udp  32769  nlockmgr
    100021    4   udp  32769  nlockmgr
    100021    1   tcp  32803  nlockmgr
    100021    3   tcp  32803  nlockmgr
    100021    4   tcp  32803  nlockmgr
[root@nfs-server ~]# 
{% endhighlight %}

发现本来随机的端口都已经变成了自己指定的了

然后打开对这些端口的访问权限

编辑 **/etc/sysconfig/iptables** , 加入下列行, 然后 reload 

{% highlight bash %}
-A INPUT -i em1 -p tcp -s 192.168.1.0/24 -m multiport --dport 111,2049,875,892,32769,32803 -j ACCEPT
-A INPUT -i em1 -p udp -s 192.168.1.0/24 -m multiport --dport 111,2049,875,892,32769,32803 -j ACCEPT
{% endhighlight %}

或者直接输入

**`iptables -A INPUT -i em1 -p tcp -s 192.168.1.0/24 -m multiport --dport 111,2049,875,892,32769,32803 -j ACCEPT`**
**`iptables -A INPUT -i em1 -p udp -s 192.168.1.0/24 -m multiport --dport 111,2049,875,892,32769,32803 -j ACCEPT`**

> **Tip:** **iptables** 的配配置需要root的权限，不加udp的部分，会导致部分client无法正常showmount和挂载


---

## 客户端挂载


确保 **/etc/init.d/portmap** (Centos 5.x) 或 **/etc/init.d/rpcbind** (Centos 6.x)

**/etc/init.d/nfslock** 都是启动状态

{% highlight bash %}
[root@nfs-client ~]# mount -t nfs -o intr nfs-server:/data/nfs /mnt/nfs/ 
[root@nfs-client ~]# df  -h 
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda5              95G   22G   69G  25% /
/dev/sda6             429G  215G  192G  53% /opt
/dev/sda3             284G   99G  172G  37% /var
/dev/sda1             494M   36M  434M   8% /boot
tmpfs                  24G     0   24G   0% /dev/shm
nfs-server:/data/nfs
                      1.7T  585G  1.1T  36% /mnt/nfs
[root@nfs-client ~]# 
{% endhighlight %}


查看本地挂载参数
-

使用 **mount** 不加参数查看


Option | Default
-------- | ---
suid/nosuid| suid
rw/ro| rw
dev/nodev|dev
exec/noexec|exec
user/nouser|nouser
auto/noauto|auto


NFS额外的挂载参数

Option| Default
-------- | ---
fg/bg|fg
soft/hard|hard
intr|-
rsize/wsize|rsize=1024,wsize=1024





注意
=

rpcbind (Centos6.x 下) / portmap (Centos5.x 下)

---

总结
=

**cat /etc/exports**

**no_root_squash** :代表root不进行匿名替换，保留root权限

**/etc/init.d/rpcbind** (Centos6 与Centos5不同 ,Centos5 下是portmap)

**/etc/init.d/nfs**

**/etc/init.d/nfslock**

**netstat -tulnp \| grep -E '(rpc\|nfs)'**

NFS的启动信息默认会写到 **/var/log/messages** 里

**rpcinfo -p localhost**

**rpcinfo -t test nfs**

**showmount  -a localhost**

**showmount  -e localhost**

**cat /var/lib/nfs/etab**

**exportfs  -ra**

**grep -v "^#" /etc/sysconfig/nfs**

**rpcinfo  -p \| grep -E '(rquota\|mount\|nlock)'**

**`iptables -A INPUT -i em1 -p tcp -s 192.168.1.0/24 -m multiport --dport 111,2049,875,892,32769,32803 -j ACCEPT`**

**`iptables -A INPUT -i em1 -p udp -s 192.168.1.0/24 -m multiport --dport 111,2049,875,892,32769,32803 -j ACCEPT`**

**mount -t nfs -o intr nfs-server:/data/nfs /mnt/nfs/**



