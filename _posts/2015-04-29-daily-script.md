---
layout: default
title: Daily Script
comments: true
---

---

前言
=

家中常备脚本，居家旅行必备良品


---


[mysql]
-

\# 表结构信息 

~~~
SHOW TABLE STATUS FROM `abc` LIKE 'def'\G
SHOW CREATE TABLE `abc`.`def`\G
SHOW INDEX FROM `abc`.`def`\G
~~~

\# 复制

~~~
CHANGE MASTER TO MASTER_HOST='192.168.x.x', MASTER_USER='xxx',MASTER_PASSWORD='xxx', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1;

stop slave;
start slave;
show slave status\G
~~~

