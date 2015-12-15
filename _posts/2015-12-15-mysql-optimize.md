---
layout: post
title:  Mysql 优化存储
categories: linux  mysql 
excerpt: 使用optimize table对数据库进行优化
comments: true
date:   2015-12-15 17:37:00
---


---

#前言


Mysql长期使用会产生碎片，使用 **optimize table** 可以有效减少碎片


>Reorganizes the physical storage of table data and associated index data, to reduce storage space and improve I/O efficiency when accessing the table. The exact changes made to each table depend on the storage engine used by that table. This statement does not work with views. This statement requires SELECT and INSERT privileges for the table.

大量的增删改操作过后，数据文件中会留下大量间隙块，出于性能的考虑，这些间隙是不会被立刻回收的，**optimize table** 在 InnoDB 的存储引擎里是被映射成了 **ALTER TABLE ... FORCE** ，其实就是强制以原来的定义重建表，导入数据，更新索引和释放空闲空间，从而一定程度上也优化了读写性能

目前来说此操作只对 innodb 和myisam有效，下面分享一下它的基本操作，详细内容可以参考 **[官方文档][optimize]** 

> **Note:** 此操作会锁表，也会产生大量磁盘IO，并且持续时间非常长，生产环境慎用，一般可以用在slave上

我的场景也假定在slave上进行此操作

---


#概要

* TOC
{:toc}


---

##检查一致性

操作之前进行一致性检查，以确保主备一致

**[Percona][percona]** 的 **[percona-toolkit][percona-toolkit]** 中提供一个叫 **[pt-table-checksum][pt-table-checksum]** 的工具，可以有效地进行一致性检查

{% highlight bash %}
[root@opti-master checkdb]# pt-table-checksum --nocheck-replication-filters --nocheck-binlog-format --replicate=ptcheck.checksum --databases youku_db,jd_db,elearning_db,bat_db   h=opti-master,u=ptcheck --ask-pass
Enter MySQL password: 
{% endhighlight %}

此工具要求用户 **ptcheck** 对所检查的库或表有读权限和对 **ptcheck.checksum** 有写权限 ，有在 **ptcheck** 库中创建表的权限

根据提示输入密码就会开始进行检查

> **Note:** 此时指定的host要是master

> **Note:** **[pt-table-checksum][pt-table-checksum]** 需要 **Term::ReadKey** 的支持，如果缺少，会报下面的错误 

{% highlight bash %}
12-15T14:22:37 Cannot read response; is Term::ReadKey installed? Can't locate Term/ReadKey.pm in @INC (@INC contains: /usr/lib64/perl5/site_perl/5.8.8/x86_64-linux-thread-multi /usr/lib/perl5/site_perl/5.8.8 /usr/lib/perl5/site_perl /usr/lib64/perl5/vendor_perl/5.8.8/x86_64-linux-thread-multi /usr/lib/perl5/vendor_perl/5.8.8 /usr/lib/perl5/vendor_perl /usr/lib64/perl5/5.8.8/x86_64-linux-thread-multi /usr/lib/perl5/5.8.8 .) at /usr/bin/pt-table-checksum line 2612.
{% endhighlight %}

解决方法是安装这个包

{% highlight bash %}
[root@opti-master checkdb]# yum install perl-TermReadKey.x86_64  
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * epel: mirrors.ustc.edu.cn
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package perl-TermReadKey.x86_64 0:2.30-4.el5 set to be updated
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================
 Package                            Arch                     Version                        Repository              Size
=========================================================================================================================
Installing:
 perl-TermReadKey                   x86_64                   2.30-4.el5                     epel                    32 k

Transaction Summary
=========================================================================================================================
Install       1 Package(s)
Upgrade       0 Package(s)

Total download size: 32 k
Is this ok [y/N]: y
Downloading Packages:
perl-TermReadKey-2.30-4.el5.x86_64.rpm                                                            |  32 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Finished Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing     : perl-TermReadKey                                                                                  1/1 

Installed:
  perl-TermReadKey.x86_64 0:2.30-4.el5                                                                                   

Complete!
[root@opti-master checkdb]#
{% endhighlight %}

---

##获取一致性检查结果

**[percona-toolkit][percona-toolkit]** 中提供一个叫 **[pt-table-sync][pt-table-sync]** 的工具，可以获取一致性检查结果

{% highlight bash %}
[root@opti-master checkdb]# pt-table-sync --replicate ptcheck.checksum  h=opti-slave,u=ptcheck --ask-pass  --sync-to-master --databases=youku_db,jd_db,elearning_db,bat_db --print > /tmp/users.sql
Enter password for opti-slave: [root@opti-master checkdb]# 
[root@opti-master checkdb]# cat /tmp/users.sql 

[root@opti-master checkdb]#
{% endhighlight %}

其实这个工具是用来根据 **[pt-table-checksum][pt-table-checksum]** 生成的结果来同步差异部分的，但是如若不使用 **`--execute`** 就不会执行，使用 **`--print`** 可以打印出差异的部分，我们就是通过有无差异的记录条目来确认一致性

从结果来看，是空的，说明主备数据是一致的

> **Note:** 此时指定的host要是slave，也就是待检查的对象

---

##停止复制

在待优化的slave上停止复制

{% highlight bash %}
mysql> show slave status\G
mysql> stop slave;
{% endhighlight %}

停止复制后，最好再使用一个文本记录一下当前的position，以避免窗口信息丢失后，又执行了reset slave命令产生不良后果

---

##生成优化语句

{% highlight bash %}
mysql> select concat('optimize table ',TABLE_SCHEMA,'.',TABLE_NAME,';')  from information_schema.TABLES where (ENGINE='MyISAM' or ENGINE='InnoDB') and TABLE_SCHEMA!='information_schema' and TABLE_SCHEMA!='mysql'  into  outfile  "/tmp/optimize.sql";
Query OK, 365 rows affected (0.09 sec)

mysql> 
{% endhighlight %}

结果是类似这样的

{% highlight bash %}
[root@opti-slave tmp]# cat optimize.sql 
optimize table azheng_db.answers;
optimize table azheng_db.feedbacks;
optimize table azheng_db.logged_exceptions;
optimize table azheng_db.question_answers;
optimize table azheng_db.questions;
optimize table azheng_db.rule_answers;
optimize table azheng_db.rule_feedbacks;
...
...
...
{% endhighlight %}

---

##执行优化


{% highlight bash %}
[root@opti-slave hunter]# time nohup  mysql -u root -p  < optimize.sql   2>&1  >> optim.log
nohup: redirecting stderr to stdout
Enter password: 



{% endhighlight %}

输入密码后，就开始了优化过程 

可以另开一个终端进行监视


{% highlight bash %}
[root@opti-slave hunter]# tail -f optim.log 
bat_db.spec_items	optimize	status	OK
Table	Op	Msg_type	Msg_text
...
...
...
{% endhighlight %}

也可以使用 **show processlist** 在数据库里查看当前状态

{% highlight bash %}
mysql> show processlist;
+----+------+-----------+------+---------+------+-------------------+-----------------------------------------+-----------+---------------+
| Id | User | Host      | db   | Command | Time | State             | Info                                    | Rows_sent | Rows_examined |
+----+------+-----------+------+---------+------+-------------------+-----------------------------------------+-----------+---------------+
| 24 | root | localhost | NULL | Query   |   21 | copy to tmp table | optimize table func_db.sport_stat_items |         0 |             0 |
| 29 | root | localhost | NULL | Query   |    0 | init              | show processlist                        |         0 |             0 |
+----+------+-----------+------+---------+------+-------------------+-----------------------------------------+-----------+---------------+
2 rows in set (0.01 sec)

mysql> 
{% endhighlight %}


---

##优化脚本

一般此过程会非常漫长，可以写一个脚本来后台运行，或简单的控制一下IO

{% highlight bash %}
[hunter@opti-slave ~]$ cat opti.bash 
#!/bin/bash 


cat /path/to/optimize.sql | while read LINE
do 
   mysql -u root -pxxxxx -e "$LINE"    
   sleep 10;
done
[hunter@opti-slave ~]$ 
{% endhighlight %}


使用如下方式调用


{% highlight bash %}
time nohup bash opti.bash  >> /path/to/optimize.log   2>&1 &
{% endhighlight %}

通过监控 **optimize.log** 来判断执行完成状态

也可以通过查看监控，IOPS很能反映问题

---

##恢复备份

优化完成后，立刻恢复备份

{% highlight bash %}
start slave;
{% endhighlight %}

通过对比前后数据文件大小，可以明显看到优化效果

一般少也能缩减5%的空间，平均在10%左右，我自己经历最明显效果的是减少了32%的空间，对于一个大库来说，能节省不少磁盘空间，并且对查询性能也有一定优化效果

---

#命令总结


* **`pt-table-checksum --nocheck-replication-filters --nocheck-binlog-format --replicate=ptcheck.checksum --databases youku_db,jd_db,elearning_db,bat_db   h=opti-master,u=ptcheck --ask-pass`**
* **`pt-table-sync --replicate ptcheck.checksum  h=opti-slave,u=ptcheck --ask-pass  --sync-to-master --databases=youku_db,jd_db,elearning_db,bat_db --print > /tmp/users.sql`**
* **`time nohup  mysql -u root -p  < optimize.sql   2>&1  >> optim.log`**


---


[optimize]:http://dev.mysql.com/doc/refman/5.6/en/optimize-table.html
[percona]:https://www.percona.com/
[percona-toolkit]:https://www.percona.com/software/mysql-tools/percona-toolkit
[pt-table-checksum]:https://www.percona.com/doc/percona-toolkit/2.2/pt-table-checksum.html
[pt-table-sync]:https://www.percona.com/doc/percona-toolkit/2.2/pt-table-sync.html

