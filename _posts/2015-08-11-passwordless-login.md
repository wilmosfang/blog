---
layout: post
title: 证书管理碉堡机
categories: linux ssh
wc: 254 639 6129
excerpt: follow me
comments: true
---

#前言

使用碉堡机进行跳转管理是一种有效的安全手段

---

#概要

* TOC
{:toc}


---


##准备


###堡垒机两台


| HOST| PassLogin | PublicIP|PrivateIP|
| :------- | ----: | :---: | :---: |
| a| yes |  yes |yes|
| b| yes |  yes |yes|

> **Tip:** 之所以用两台是为了避免单点故障而登不上所有的机器

###被管理机若干台


| HOST| PassLogin | PublicIP|PrivateIP|
| :------- | ----: | :---: | :---: |
| c| no |  no |yes|
| d| no |  no |yes|


> **Tip:** 被管理机和堡垒机在同一内网，并且互通

---

##操作



###创建用户


操作范围：ALL

{% highlight bash %}
[root@h101 ~]# useradd test 
{% endhighlight %}

###设置密码


操作范围：ALL

{% highlight bash %}
[root@h101 ~]# passwd test
Changing password for user test.
New password: 
BAD PASSWORD: it is too short
BAD PASSWORD: is too simple
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@h101 ~]# 
{% endhighlight %}

###生成密钥


操作范围：a,b(所有堡垒机)

{% highlight bash %}
[test@h101 ~]$ ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/test/.ssh/id_rsa): 
Created directory '/home/test/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/test/.ssh/id_rsa.
Your public key has been saved in /home/test/.ssh/id_rsa.pub.
The key fingerprint is:
23:ac:3b:ff:3e:8f:75:b0:23:c9:2e:b3:e3:8b:a3:63 test@h101.temp
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|                 |
|     .           |
|      o S .      |
|     . o o o     |
|    .   + + .    |
|  E oo+..+ o     |
| ..oo==O=o.      |
+-----------------+
[test@h101 ~]$
{% endhighlight %}

###创建.ssh目录


操作范围：c,d(所有被管理机)

{% highlight bash %}
[test@h102 ~]$ mkdir .ssh
[test@h102 ~]$ chmod 700 .ssh/
[test@h102 ~]$ ll .ssh/ -d 
drwx------. 2 test test 4096 Jun 11 14:45 .ssh/
[test@h102 ~]$ 
{% endhighlight %}

###导入证书


操作范围：c,d(所有被管理机)

使用下列方法依次导入a,b(所有堡垒机证书)

{% highlight bash %}
[test@h102 .ssh]$ cat /tmp/h101_id_rsa.pub  >> ~/.ssh/authorized_keys
[test@h102 .ssh]$ chmod 600 ~/.ssh/authorized_keys
[test@h102 .ssh]$ ll ~/.ssh/authorized_keys
-rw-------. 1 test test 396 Jun 11 14:51 /home/test/.ssh/authorized_keys
[test@h102 .ssh]$ 
{% endhighlight %}

###验证无密码登录


操作范围：a,b(任意一台堡垒机)

> **Tip:** 第一次会提示接收证书到 **known_hosts**

{% highlight bash %}
[test@h101 ~]$ ssh h102
The authenticity of host 'h102 (192.168.100.102)' can't be established.
RSA key fingerprint is 78:c4:6f:3f:08:43:d1:2a:02:bf:ec:f3:9f:e3:89:76.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'h102,192.168.100.102' (RSA) to the list of known hosts.
[test@h102 ~]$ 
{% endhighlight %}

之后，就可以直接登录

{% highlight bash %}
[test@h101 .ssh]$ ssh h102
Last login: Thu Jun 11 14:55:55 2015 from h101.temp
[test@h102 ~]$ 
{% endhighlight %}

> **Note:**  如果之前有使用过这个用户，并且 **known_hosts** 中已经有对被控主机上同一用户的证书记录，可能会产生如下报错

{% highlight bash %}
[test@h101 .ssh]$ ssh h102
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
78:c4:6f:3f:08:43:d1:2a:02:bf:ec:f3:9f:e3:89:76.
Please contact your system administrator.
Add correct host key in /home/test/.ssh/known_hosts to get rid of this message.
Offending key in /home/test/.ssh/known_hosts:2
RSA host key for h102 has changed and you have requested strict checking.
Host key verification failed.
[test@h101 .ssh]$
{% endhighlight %}

解决办法是清掉 **known_hosts** 中的这条记录，让它重新接受

###添加sudo权限


操作范围：ALL

{% highlight bash %}
[root@h102 .ssh]# visudo 
{% endhighlight %}

最后加上下面三条，然后保存退出
{% highlight bash %}
Cmnd_Alias COMSU = /bin/su
User_Alias USERSU = test
USERSU  ALL=(root)  COMSU
{% endhighlight %}

###测试sudo权限


操作范围：ALL(任意一台机器)
在提示输密码的地方输入test用户的密码

{% highlight bash %}
[test@h102 ~]$ sudo su - 

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for test: 
[root@h102 ~]#
{% endhighlight %}

###关闭密码登录


对于其它被管理机，如不直接提供对外服务，尽量避免配置Public IP,并且使用下面方法关闭 **sshd** 的密码登录

{% highlight bash %}
sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
/etc/init.d/sshd  restart
{% endhighlight %}

> **Note:** 重启服务才能使配置生效，但在重启之前，一定要确认使用证书可以认证登录,或者旁边多开着一个已登录的终端，否则除了跑机房通过 **console** 没有别的办法可以远程登录，所有的远程登录权限配置，防火墙配置都要注意此类问题

---


###使用方法


本地生成一个证书，然后加入到堡垒机的 **authorized_keys** 中，然后使用证书书登录堡垒机

通过堡垒机来跳转其它被管理机


---

#注意事项


* 定期更换被管理机证书（两到三个月）
* 定期更换堡垒机密码 （一个月）
* 人员变动更换所有密码与证书（首先在堡垒机上注销此用户）




