---
layout: post
title: Temp Blog Name
author: wilmosfang
tags:  template
categories:  template
wc: 115 186 1696
excerpt: 我的blog模板与书写规范，检查与处理流程
comments: true
---

# 前言

用于以后的 **blog** 行文规范

---

# 概要

* TOC
{:toc}


---


# 一级内容标题

---

## 二级内容标题

---


### 三级内容标题

#### 四级内容标题

##### 五级内容标题

###### 六级内容标题

---

代码块

~~~
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
touch /var/lock/subsys/local
~~~


> **Tip:**


~~~
just for test
~~~


> **Note:**

---

# 总结

* **`for i in $(seq 1 10); do  echo -n ! &&  echo "[sms$i.png](/images/zabbix_sms/sms$i.png)" ; done`**

---

# 附录

## 创建流程

* 内容书写
* 截图
* 图片信息过滤处理
* 嵌入图片

~~~
[gituser@tools ~]$ for i in `seq 1 10`; do  echo -n ! &&  echo "[sms$i.png](/images/zabbix_sms/sms$i.png)" ; done 
![sms1.png](/images/zabbix_sms/sms1.png)
![sms2.png](/images/zabbix_sms/sms2.png)
![sms3.png](/images/zabbix_sms/sms3.png)
![sms4.png](/images/zabbix_sms/sms4.png)
![sms5.png](/images/zabbix_sms/sms5.png)
![sms6.png](/images/zabbix_sms/sms6.png)
![sms7.png](/images/zabbix_sms/sms7.png)
![sms8.png](/images/zabbix_sms/sms8.png)
![sms9.png](/images/zabbix_sms/sms9.png)
![sms10.png](/images/zabbix_sms/sms10.png)
[gituser@tools ~]$ 
~~~

---

## 处理流程

* 标记转化
* 过滤检查
* 信息过滤
* 命令汇总
* 编写摘要
* 内容统计
* 检查发布

---

[link]: http://soft.dog/
[temp]: http://soft.dog/
