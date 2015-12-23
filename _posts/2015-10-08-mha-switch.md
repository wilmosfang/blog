---
layout: post
title: mha 切换(操作流程建议)
categories: linux mha mysql cluster
wc: 87 165 2668
excerpt: follow me
comments: true
---

---

#前言

如果使用的 **[mha][mha]** 构建mysql集群，由于各种原因会遇到需要进行迁移的情况

这里分享一下 **mha切换** 的过程

> **Tip:** 当前版本 **[MHA Manager 0.56][mha]**

---

#概要

* TOC
{:toc}


---

##检查列表


在进行切换之前一定要对数据库状态进行检查，下面是检查列表：

* 1.所有节点数据库必须正常运行
* 2.所有备库复制必须指向主库，且复制状态正常（io , sql 进程状态正常）
* 3.备库上复制状态中 Seconds_Behind_Master 参数不能大于3秒钟
* 4.当前情况下keepalived工作正常，master获得VIP，候选master上keepalived的运行状态正常，并且优先级比master低
* 5.数据库上 relay_log_purge 参数都为 off ，否则有潜在的丢失掉relay log 的风险
* 6.除了主库，其它备库上的 read_only 参数都为 on ,否则有主备数据库不一致的风险

---

##手动切换操作手顺


* 1.关掉后台切换监控 **`masterha_stop --conf=/etc/app1.cnf`** 
* 2.检查数据库运行状态
* 3.检查数据库参数
* 4.检查keepalived工作状态，ip挂载情况
* 5.记录待切slave(候选master)的binlogfile和position(用于之后同步)
* 6.进行 **repl check**
* 7.实施切换
* 8.状态检查(mysql,keepalived,参数,同步状态)
* 9.系统维护，数据库维护
* 10.主备同步(使用之前的不变的slave position)
* 11.将维护好的数据库加入mha集群
* 12.检查数据库运行状态
* 13.检查数据库参数
* 14.修改keepalived.conf文件，降低优先级，使其不与当前主master争抢ip
* 15.启动keepalived,检查ip,检查keepalived运行状态
* 16.记录待切slave(候选master,原master)的binlogfile和position(用于之后同步)
* 17.进行 **repl check**
* 18.实施切换
* 19.恢复mha架构(参考前面步骤)
* 20.进行后台mha监控
* 21.监控观察


---

##可能涉及到的命令

* **`masterha_check_status  --conf=/etc/app1.cnf`**
* **`masterha_stop  --conf=/etc/app1.cnf`**
* **`masterha_check_repl --conf=/etc/app1.cnf`**
* **`nohup   masterha_manager --conf=/etc/app1.cnf  --ignore_last_failover  &`**
* **`masterha_master_switch --master_state=alive --conf=/etc/app1.cnf  --new_master_host=m1 --interactive=0`**
* **`purge_relay_logs --user=root --password=xxxx  --workdir=/data/relay_tmp/`**
* **`show slave status\G`**
* **`show master status;`**
* **`show variables like 'relay_log_purge';`**
* **`show variables like 'read_only';`**


---
[mha]:https://code.google.com/p/mysql-master-ha/

