---
layout: post
title: Mysql MHA 搭建 (二) mha0.53 安装
author: wilmosfang
categories: linux mha mysql ssh cluster 
wc: 480 1782 20916
excerpt: follow me
comments: true
---



---

前言
=

继前一篇[Mysql MHA 搭建 (一) percona5.1 安装][mha1] , 这篇继续mha 的安装部分

[Mha] 全称 mysql master ha , 是一个开源项目，是一组用perl写的脚本，完成自动（非计划情况）或手动（计划情况下）的mysql master 切换操作，减少数据库宕机时间，保证高可用

>和其它高可用不太一样的是，它只保证一次非计划打击，需要手动恢复环境，然后手动完成同步，然后手动将修复好的节点加入到集群，检查通过后，再次进行监控

---


# 概要

* TOC
{:toc}


---

安装mha
-

配置host到m1，m2，s

~~~
[root@m1 tmp]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.75.11 m1
192.168.75.12 m2
192.168.75.13 s
[root@m1 tmp]# 
~~~

下载必要的安装包，并且拷贝到m2和s上去

>[mha的下载地址][Mha]
>
>[epel的下载地址][epel rpm]

~~~
[root@m1 tmp]# ls 
epel-release-6-8.noarch2.rpm             my.cnf
mha4mysql-manager-0.53-0.el6.noarch.rpm  my.cnf.tt
mha4mysql-manager-0.53.tar.gz            percona-release-0.1-3.noarch.rpm
mha4mysql-node-0.53-0.el6.noarch.rpm
[root@m1 tmp]# 
[root@m1 tmp]# rsync -av * m2:/root/tmp
root@m2's password: 
sending incremental file list

sent 215 bytes  received 12 bytes  41.27 bytes/sec
total size is 250732  speedup is 1104.55
[root@m1 tmp]# rsync -av * s:/root/tmp
root@s's password: 
sending incremental file list

sent 215 bytes  received 12 bytes  50.44 bytes/sec
total size is 250732  speedup is 1104.55
[root@m1 tmp]# 
~~~
给m1,m2,s都安装上epel库

~~~
[root@localhost tmp]# rpm -ivh epel-release-6-8.noarch2.rpm
warning: epel-release-6-8.noarch2.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Preparing...                ########################################### [100%]
   1:epel-release           ########################################### [100%]
[root@localhost tmp]#
~~~

根据角色的不同 , 为m1,m2,s分别安装下列mha软件

|Host|IP|Software|Role|
|:---|:---:|:---|:---:|
|m1|192.168.75.11|node|candidate_master|
|m2|192.168.75.11|node|candidate_master|
|s|192.168.75.13|node,manager|no_master,monitor|

mha-node 的安装有如下[依赖][mha install]

>mha的相关资料都托管在google code上，目前得有相应的代理或vpn进行翻墙操作才能阅览，这是[mha的软件依赖和安装方法][mha install] 

~~~
[root@localhost tmp]# rpm -ivh mha4mysql-node-0.53-0.el6.noarch.rpm
error: Failed dependencies:
        perl(DBD::mysql) is needed by mha4mysql-node-0.53-0.el6.noarch
[root@localhost tmp]#
~~~

解决依赖后，可以正常安装

~~~
[root@localhost tmp]# yum install perl-DBD-MySQL
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
...
...
Installed:
  perl-DBD-MySQL.x86_64 0:4.013-3.el6                                                              

Complete!
[root@localhost tmp]# rpm -ivh mha4mysql-node-0.53-0.el6.noarch.rpm
Preparing...                ########################################### [100%]
   1:mha4mysql-node         ########################################### [100%]
[root@localhost tmp]# 
~~~

mha-manager 依赖mha-node ，安装完node后还有如下[依赖][mha install]

~~~
[root@localhost tmp]# rpm -ivh mha4mysql-manager-0.53-0.el6.noarch.rpm
error: Failed dependencies:
        perl(Config::Tiny) is needed by mha4mysql-manager-0.53-0.el6.noarch
        perl(Log::Dispatch) is needed by mha4mysql-manager-0.53-0.el6.noarch
        perl(Log::Dispatch::File) is needed by mha4mysql-manager-0.53-0.el6.noarch
        perl(Log::Dispatch::Screen) is needed by mha4mysql-manager-0.53-0.el6.noarch
        perl(Parallel::ForkManager) is needed by mha4mysql-manager-0.53-0.el6.noarch
[root@localhost tmp]# 
~~~

解决依赖后，可以正常安装

>这个操作只在s上进行，manager 这个角色起监视和管控的作用，可以是一台备库，也可以专门拿一台没有安装数据库的机器，软件本身的运行并不基于数据库

~~~
[root@localhost tmp]# yum -y install perl-Config-Tiny   perl-Log-Dispatch  perl-Parallel-ForkManager
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.yun-idc.com
 * epel: mirrors.zju.edu.cn
 * extras: mirrors.zju.edu.cn
 * updates: mirrors.btte.net
Resolving Dependencies
...
...
Installed:
  perl-Config-Tiny.noarch 0:2.12-7.1.el6                   perl-Log-Dispatch.noarch 0:2.27-1.el6
  perl-Parallel-ForkManager.noarch 0:0.7.9-1.el6

Dependency Installed:
  perl-Email-Date-Format.noarch 0:1.002-5.el6            perl-MIME-Lite.noarch 0:3.027-2.el6
  perl-MIME-Types.noarch 0:1.28-2.el6                    perl-Mail-Sender.noarch 0:0.8.16-3.el6
  perl-Mail-Sendmail.noarch 0:0.79-12.el6                perl-MailTools.noarch 0:2.04-4.el6
  perl-Params-Validate.x86_64 0:0.92-3.el6               perl-TimeDate.noarch 1:1.16-13.el6

Complete!
[root@localhost tmp]# rpm -ivh mha4mysql-manager-0.53-0.el6.noarch.rpm
Preparing...                ########################################### [100%]
   1:mha4mysql-manager      ########################################### [100%]
[root@localhost tmp]#
~~~

软件安装完成，开始进行系统配置 

---

系统配置
-

mha需要一个有拷贝binlog权限的用户可以对其它机器进行password less登录

使用证书认证的方式可以实现这个效果，[官方文档][mha requirements]关于这一段的描述是使用root来作这个用户，这个的确非常方便，因为权限高，不仅是mha用来作各种操作，后面配合keepalived也会省不少事，但是使用root来进行此类操作毕竟有潜在风险，更何况一些环境下是不允许root直接进行远程登录的，于是我使用了mysql来充当这个用户(与其再创建一个可以读写binlog的用户，不如直接使用mysql用户来的方便)

在安装percona server的过程中，系统自动为软件创建了一个mysql用户，可是这个用户的家在**/var/lib/mysql**中，mysql的默认根目录，一个奇怪的地方

~~~
[root@localhost tmp]# grep mysql /etc/passwd
mysql:x:496:493:Percona Server:/var/lib/mysql:/bin/bash
[root@localhost tmp]# su - mysql
-bash-4.1$ ls
ibdata1  ib_logfile0  ib_logfile1  localhost.localdomain.pid  mysql  mysql.sock  test
-bash-4.1$
~~~

可不可以直接在里面安家呢，还真不可以，因为我试过，使用证书认证的过程中会产生一个**.ssh**的目录，而mysql是个很傻的数据库，它会把数据目录里的所有文件夹都当作自己的一个数据库，数据目录又默认是使用的mysql根，结果数据库里就会产生一个类似**#mysql50#.ssh**的奇怪库，它会导致复制异常，并报错

所以我手动给它创建了一个家，这个要在三台机器上都进行操作

~~~
[root@localhost tmp]# mkdir /home/mysql
[root@localhost tmp]# cp /etc/skel/.* /home/mysql/
cp: omitting directory `/etc/skel/.'
cp: omitting directory `/etc/skel/..'
cp: omitting directory `/etc/skel/.gnome2'
cp: omitting directory `/etc/skel/.mozilla'
[root@localhost tmp]# chown  -R mysql.mysql /home/mysql/
[root@localhost tmp]# chmod -R 700 /home/mysql/
~~~

将**/etc/passwd**中**mysql**的家**/var/lib/mysql**改为**/home/mysql** , 然后以**mysql**身份登录

~~~
[root@localhost tmp]# su - mysql
[mysql@localhost ~]$ ls -a
.  ..  .bash_logout  .bash_profile  .bashrc
[mysql@localhost ~]$
~~~

生成RSA证书

~~~
[mysql@localhost ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/mysql/.ssh/id_rsa):
Created directory '/home/mysql/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/mysql/.ssh/id_rsa.
Your public key has been saved in /home/mysql/.ssh/id_rsa.pub.
The key fingerprint is:
90:d4:0a:0a:35:23:43:14:cc:d4:7d:10:63:d7:26:b2 mysql@localhost.localdomain
The key's randomart image is:
+--[ RSA 2048]----+
|O** .=+o.        |
|.= +o+o+.o       |
| . . .*.o        |
|  .  E..         |
|        S        |
|                 |
|                 |
|                 |
|                 |
+-----------------+
[mysql@localhost ~]$
~~~

相互之间拷贝证书，无密码登录

~~~
[mysql@localhost ~]$ ssh-copy-id  -i .ssh/id_rsa.pub  mysql@m2
mysql@m2's password:
Now try logging into the machine, with "ssh 'mysql@m2'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[mysql@localhost ~]$ ssh-copy-id  -i .ssh/id_rsa.pub  mysql@s
The authenticity of host 's (192.168.75.13)' can't be established.
RSA key fingerprint is d0:e6:d7:01:96:f8:f2:02:fd:b0:fa:3f:1a:00:37:e7.
Are you sure you want to continue connecting (yes/no)? eys
Please type 'yes' or 'no': yes
Warning: Permanently added 's,192.168.75.13' (RSA) to the list of known hosts.^M
mysql@s's password:
Now try logging into the machine, with "ssh 'mysql@s'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[mysql@localhost ~]$
~~~

其它两台也一样，要三方互拷自己的公钥

>这里有一点需要注意，就是s不仅要把自己的公钥拷给m1和m2，还要拷给自己，这一点设计有些让人费解哈，对于manager节点，一上来不判断是谁就直接ssh连起了

mysql用户配置完成，接下来设置mha配置文件

mha有两个配置文件 : 

全局配置文件 **/etc/masterha_default.cnf**  


~~~
[mysql@s ~]$ cat /etc/masterha_default.cnf 
[server default]
user=mhauser
password=xxx
ssh_user=mysql
master_binlog_dir= /var/lib/mysql
remote_workdir=/home/mysql/mha/app1
master_ip_failover_script= /home/mysql/mha/script/master_ip_failover
master_ip_online_change_script= /home/mysql/mha/script/master_ip_online_change
# shutdown_script= /script/masterha/power_manager
# report_script= /script/masterha/send_report
[mysql@s ~]$ 

~~~

全局配置文件中设定了:

数据库层mha操作用户 , 之前在percona server里创建的mhauser用户

系统层操作用户 , 直接使用mysql

binlog目录 , 如果所有server的binlog目录路径一样，可以在此统一设定

远程工作目录 , manager 登录到远程后的工作目录   

自动切换脚本 , mha进入后台监控模式过程中使用的脚本

手动切换脚本 , 手动在线切换使用的脚本

>这些都可以被应用配置文件里的同名参数覆盖

应用配置文件 **/etc/app1.cnf**

~~~
[mysql@s ~]$ cat /etc/app1.cnf 
[server default]
manager_workdir=/home/mysql/mha/app1
manager_log=/home/mysql/mha/app1/manager.log

[server1]
hostname=m1
candidate_master=1

[server2]
hostname=m2
candidate_master=1

[server3]
hostname=s
no_master=1

[mysql@s ~]$
~~~
应用配置文件里设定了:

manager工作目录，这是一个本地目录

manager日志存放路径

集群中server的主机名也可以是IP

还有master候选资格配置


拷完后有一个mha脚本可以进行验证，只有通过ssh检查才能进行下一步，因为mha的整个机制都是建立在此之上

~~~
[mysql@s ~]$ masterha_check_ssh --conf=/etc/app1.cnf 
Sat Mar 28 04:10:09 2015 - [info] Reading default configuratoins from /etc/masterha_default.cnf..
Sat Mar 28 04:10:09 2015 - [info] Reading application default configurations from /etc/app1.cnf..
Sat Mar 28 04:10:09 2015 - [info] Reading server configurations from /etc/app1.cnf..
Sat Mar 28 04:10:09 2015 - [info] Starting SSH connection tests..
Sat Mar 28 04:10:12 2015 - [debug] 
Sat Mar 28 04:10:09 2015 - [debug]  Connecting via SSH from mysql@m1(192.168.75.11:22) to mysql@m2(192.168.75.12:22)..
Sat Mar 28 04:10:10 2015 - [debug]   ok.
Sat Mar 28 04:10:10 2015 - [debug]  Connecting via SSH from mysql@m1(192.168.75.11:22) to mysql@s(192.168.75.13:22)..
Sat Mar 28 04:10:12 2015 - [debug]   ok.
Sat Mar 28 04:10:13 2015 - [debug] 
Sat Mar 28 04:10:10 2015 - [debug]  Connecting via SSH from mysql@m2(192.168.75.12:22) to mysql@m1(192.168.75.11:22)..
Sat Mar 28 04:10:11 2015 - [debug]   ok.
Sat Mar 28 04:10:11 2015 - [debug]  Connecting via SSH from mysql@m2(192.168.75.12:22) to mysql@s(192.168.75.13:22)..
Sat Mar 28 04:10:13 2015 - [debug]   ok.
Sat Mar 28 04:10:14 2015 - [debug] 
Sat Mar 28 04:10:10 2015 - [debug]  Connecting via SSH from mysql@s(192.168.75.13:22) to mysql@m1(192.168.75.11:22)..
Sat Mar 28 04:10:12 2015 - [debug]   ok.
Sat Mar 28 04:10:12 2015 - [debug]  Connecting via SSH from mysql@s(192.168.75.13:22) to mysql@m2(192.168.75.12:22)..
Sat Mar 28 04:10:14 2015 - [debug]   ok.
Sat Mar 28 04:10:14 2015 - [info] All SSH connection tests passed successfully.
[mysql@s ~]$ 
~~~

如果没能通过检测，可以根据日志进行排错

然后进行复制检查

~~~
[mysql@s ~]$ masterha_check_repl --conf=/etc/app1.cnf 
Sat Mar 28 04:49:47 2015 - [info] Reading default configuratoins from /etc/masterha_default.cnf..
Sat Mar 28 04:49:47 2015 - [info] Reading application default configurations from /etc/app1.cnf..
Sat Mar 28 04:49:47 2015 - [info] Reading server configurations from /etc/app1.cnf..
Sat Mar 28 04:49:47 2015 - [info] MHA::MasterMonitor version 0.53.
Sat Mar 28 04:49:48 2015 - [info] Dead Servers:
Sat Mar 28 04:49:48 2015 - [info] Alive Servers:
Sat Mar 28 04:49:48 2015 - [info]   m1(192.168.75.11:3306)
Sat Mar 28 04:49:48 2015 - [info]   m2(192.168.75.12:3306)
Sat Mar 28 04:49:48 2015 - [info]   s(192.168.75.13:3306)
Sat Mar 28 04:49:48 2015 - [info] Alive Slaves:
Sat Mar 28 04:49:48 2015 - [info]   m1(192.168.75.11:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Sat Mar 28 04:49:48 2015 - [info]     Replicating from 192.168.75.12(192.168.75.12:3306)
Sat Mar 28 04:49:48 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Mar 28 04:49:48 2015 - [info]   s(192.168.75.13:3306)  Version=5.1.73-14.12-log (oldest major version between slaves) log-bin:enabled
Sat Mar 28 04:49:48 2015 - [info]     Replicating from 192.168.75.12(192.168.75.12:3306)
Sat Mar 28 04:49:48 2015 - [info]     Not candidate for the new Master (no_master is set)
Sat Mar 28 04:49:48 2015 - [info] Current Alive Master: m2(192.168.75.12:3306)
Sat Mar 28 04:49:48 2015 - [info] Checking slave configurations..
Sat Mar 28 04:49:48 2015 - [info] Checking replication filtering settings..
Sat Mar 28 04:49:48 2015 - [info]  binlog_do_db= , binlog_ignore_db= 
Sat Mar 28 04:49:48 2015 - [info]  Replication filtering check ok.
Sat Mar 28 04:49:48 2015 - [info] Starting SSH connection tests..
Sat Mar 28 04:49:52 2015 - [info] All SSH connection tests passed successfully.
Sat Mar 28 04:49:52 2015 - [info] Checking MHA Node version..
Sat Mar 28 04:49:54 2015 - [info]  Version check ok.
Sat Mar 28 04:49:54 2015 - [info] Checking SSH publickey authentication settings on the current master..
Sat Mar 28 04:49:54 2015 - [info] HealthCheck: SSH to m2 is reachable.
Sat Mar 28 04:49:55 2015 - [info] Master MHA Node version is 0.53.
Sat Mar 28 04:49:55 2015 - [info] Checking recovery script configurations on the current master..
Sat Mar 28 04:49:55 2015 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/home/mysql/mha/save_binary_logs_test --manager_version=0.53 --start_file=mysql-bin.000014 
Sat Mar 28 04:49:55 2015 - [info]   Connecting to mysql@m2(m2).. 
  Creating /home/mysql/mha if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin.000014
Sat Mar 28 04:49:56 2015 - [info] Master setting check done.
Sat Mar 28 04:49:56 2015 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Sat Mar 28 04:49:56 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=m1 --slave_ip=192.168.75.11 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Sat Mar 28 04:49:56 2015 - [info]   Connecting to mysql@192.168.75.11(m1:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to m1-relay-bin.000002
    Temporary relay log file is /var/lib/mysql/m1-relay-bin.000002
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Sat Mar 28 04:49:57 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user=mhauser --slave_host=s --slave_ip=192.168.75.13 --slave_port=3306 --workdir=/home/mysql/mha --target_version=5.1.73-14.12-log --manager_version=0.53 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Sat Mar 28 04:49:57 2015 - [info]   Connecting to mysql@192.168.75.13(s:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to s-relay-bin.000019
    Temporary relay log file is /var/lib/mysql/s-relay-bin.000019
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Sat Mar 28 04:49:58 2015 - [info] Slaves settings check done.
Sat Mar 28 04:49:58 2015 - [info] 
m2 (current master)
 +--m1
 +--s

Sat Mar 28 04:49:58 2015 - [info] Checking replication health on m1..
Sat Mar 28 04:49:58 2015 - [info]  ok.
Sat Mar 28 04:49:58 2015 - [info] Checking replication health on s..
Sat Mar 28 04:49:58 2015 - [info]  ok.
Sat Mar 28 04:49:58 2015 - [info] Checking master_ip_failover_script status:
Sat Mar 28 04:49:58 2015 - [info]   /home/mysql/mha/script/master_ip_failover --command=status --ssh_user=mysql --orig_master_host=m2 --orig_master_ip=192.168.75.12 --orig_master_port=3306 
Sat Mar 28 04:49:58 2015 - [info]  OK.
Sat Mar 28 04:49:58 2015 - [warning] shutdown_script is not defined.
Sat Mar 28 04:49:58 2015 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.
[mysql@s ~]$ 
~~~

这里有一个**[warning] shutdown_script is not defined.**的警告，由于我们暂时不考虑fense的情况，就让它警告吧，不影响使用

这一步是检查主从复制的情况，mha要求所有节点都复制正常，这一步检查容易出问题的是**relay_log_purge**没有设置成**off** , **read_only**在备库上没有设置成**on** , 然后就是各种脚本有问题，根据提示进行修改后检查通过才能进行操作，这个检查过程中已经包含了ssh的通过性检查，前面一步检查不通过，这一步是肯定没法通过的，因为它们调用了同一个检查脚本


如果通过了这一步检查，代表mha已经安装正确，其实此时已经可以进行切换了，目前的程度，mha已经可以确保master的迁移和备库的配置调整，让它自动同步数据并且指向新的master（它会帮忙计算好要同步的file和pos），只是mha本身不负责ip的迁移，这个事情由keepalived来负责，由mha与keepalived一起配合完成自动迁移的全过程

这里有两个脚本还没有涉及到，分别是：


**master_ip_failover_script= /home/mysql/mha/script/master_ip_failover**

**master_ip_online_change_script= /home/mysql/mha/script/master_ip_online_change**

我会在下一节中结合**keepalived**进行阐释

这里是[mha的wiki][mha wiki] , 里面有对mha的详细介绍，不过，极有可能，你打不开，原因是被墙了，想获取真知，想想办法吧

---

[Mha]:http://code.google.com/p/mysql-master-ha/
[mha1]:http://soft.dog/2015/03/26/mysql-HA-build-percona-server5.1-install.html
[epel rpm]:http://rpmfind.net/linux/rpm2html/search.php?query=epel&submit=Search+...
[mha install]:https://code.google.com/p/mysql-master-ha/wiki/Installation
[mha requirements]:https://code.google.com/p/mysql-master-ha/wiki/Requirements
[mha wiki]:https://code.google.com/p/mysql-master-ha/wiki/TableOfContents?tm=6

