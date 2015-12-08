---
layout: post
title: Daily Script
categories: linux script
excerpt: follow me
comments: true
---

---

前言
=

家中常备脚本，居家旅行必备良品


---

#概要

* TOC
{:toc}


---

[mysql]
-

\#show

{% highlight bash %}
show databases;
show CREATE DATABASE `abc_qa`
show tables;
show variables like "%format%";
show character set;
show character set like '%utf8%';
show collation like "%utf8%";
SHOW TABLE STATUS FROM `abc` LIKE 'def'\G
SHOW CREATE TABLE `abc`.`def`\G
SHOW INDEX FROM `abc`.`def`\G
show slave status\G
{% endhighlight %}

\#desc

{% highlight bash %}
desc mysql.user

desc mysql.db
{% endhighlight %}

\#显示当前用户

{% highlight bash %}
select user();
{% endhighlight %}

\#基本操作权限

{% highlight bash %}
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, CREATE VIEW, SHOW VIEW ON `db_dev`.* TO 'abc'@'192.168.1.%' IDENTIFIED BY 'xxxxx';
FLUSH PRIVILEGES;
{% endhighlight %}

\#查看用户权限

{% highlight bash %}
show grants for 'care'@'192.168.1.%';
{% endhighlight %}

\# 复制

{% highlight bash %}
CHANGE MASTER TO MASTER_HOST='192.168.x.x', MASTER_USER='xxx',MASTER_PASSWORD='xxx',MASTER_PORT=330x, MASTER_LOG_FILE='mysql-bin.00000x', MASTER_LOG_POS=1;

stop slave;
start slave;
show slave status\G
{% endhighlight %}

\#创建操作

{% highlight bash %}
CREATE DATABASE db_name;
CREATE TABLE tbl_name (id int(4),name char(20));
INSERT INTO tbl_name (col1,col2) VALUES(15,col1*2);
{% endhighlight %}

\#修改操作

{% highlight bash %}
ALTER TABLE chatter_users alter column ip varchar(50) NULL; 
alter table address modify column city char(30);
alter table address modify column city varchar(50);
{% endhighlight %}


\#删除操作

{% highlight bash %}
DROP DATABASE db_name;
DROP TABLE tbl_name;
TRUNCATE TABLE tbl_name;
{% endhighlight %}

\#一致性检查与同步

{% highlight bash %}
#only sp master
pt-table-checksum --nocheck-replication-filters --replicate=pt.checksums h=h102,u=test,p=test,P=3307
pt-table-checksum --nocheck-replication-filters  --nocheck-binlog-format --replicate=pt.check  h=192.168.100.201,u=test,p=test 

#only sp slave
pt-table-sync --replicate pt.checksums   h=192.168.100.202,u=test,p='test'    --sync-to-master --databases=test   --tables=t1 --print
pt-table-sync --replicate pt.checksums   h=192.168.100.202,u=test,p='test'    --sync-to-master --databases=test   --tables=t1 --execute
{% endhighlight %}


\#参数推荐

{% highlight bash %}
pt-variable-advisor 192.168.1.123 --user root --password abc
{% endhighlight %}

\# mysql 信息汇总

{% highlight bash %}
pt-mysql-summary --user root --password abc
{% endhighlight %}

\# mysql 参数对比

{% highlight bash %}
pt-config-diff h=192.168.1.123 h=192.168.1.178 --user=abc --password=abc
{% endhighlight %}



\# mysql 参数调整

{% highlight bash %}
./mysqltuner.pl --forcemem 48288 --forceswap 4094 
{% endhighlight %}

\#查看长时休眠的mysql进程

{% highlight bash %}
select * from  information_schema.processlist where Command="sleep" and Time>86400;
{% endhighlight %}

\#杀进程脚本

{% highlight bash %}
mysql -u root -p -e "select concat('KILL ',ID,';') from information_schema.processlist where COMMAND='Sleep';"  | cat 

mysql -u root -p -e "select concat('KILL ',ID,';') from information_schema.processlist where COMMAND='Sleep' and time > 259200;"  | cat 
{% endhighlight %}

\# mysqladmin 

{% highlight bash %}
mysqladmin flush-hosts -h localhost -P 3306 -u root -p 

mysqladmin -u root -pPASSWORD flush-hosts flush-tables flush-threads flush-logs flush-privileges flush-status
{% endhighlight %}

\#导出数据库

{% highlight bash %}
mysqldump -u root -p  a_qa > a.qa.sql
{% endhighlight %}

\#导入数据库

{% highlight bash %}
use xxx;
source fff.sql;
{% endhighlight %}




---

[smartctl]
-

\#各种检查方法 

{% highlight bash %}
smartctl  -t short  /dev/sda
smartctl -C -t short  /dev/sda
smartctl -i  /dev/sda
smartctl -i  -d ata /dev/sda
smartctl -H /dev/sda
smartctl -a /dev/sda
smartctl  -A /dev/sda
{% endhighlight %}

---

[ansible]
-

\#ansible 基本模块与用法

{% highlight bash %}
ansible test -m command -a 'hostname'
ansible test -m command -a "ls -al /tmp/ansible.cfg"
ansible test -m command -a 'ping'
ansible test -m command -a 'uptime'
ansible test -m copy -a "src=/etc/ansible/ansible.cfg dest=/tmp/ansible.cfg owner=root group=root mode=0644"
ansible test -m copy -a "src=/tmp/rocketzhang_test.sh dest=/tmp/rocketzhang_test.sh owner=root group=root mode=0755"
ansible test -m file -a "path=/tmp/resolv.conf state=absent"
ansible test -m file -a "src=/etc/resolv.conf dest=/tmp/resolv.conf state=link"
ansible test -m ping
ansible test -m shell -a "/tmp/rocketzhang_test.sh"
ansible-doc -l  #list module
{% endhighlight %}

---

[drop_caches]
-


\#drop cache

{% highlight bash %}
sync
echo 3 > /proc/sys/vm/drop_caches
{% endhighlight %}

---

[iotop]
-

\#记录io日志

{% highlight bash %}
iotop -ot > iotop.log
{% endhighlight %}


---

[gcc]
-

{% highlight bash %}
gcc -o t1 -g test.c 
{% endhighlight %}

---

[badblocks]
-

\#坏块检查

{% highlight bash %}
badblocks -s -v -o /root/badblocks.log /dev/sda
{% endhighlight %}



---

[vim]
-

{% highlight bash %}
:set hls #打开高亮
:set nohls #关闭高亮
{% endhighlight %}


---

[sysbench]
-

\#基准测试

{% highlight bash %}
sysbench --test=fileio --num-threads=200 --file-num=2000 --file-total-size=1536G --file-test-mode=rndrw prepare

sysbench --test=fileio --num-threads=200 --file-num=2000 --file-total-size=1536G --file-test-mode=rndrw run  

 sysbench --test=fileio --num-threads=200 --file-num=2000 --file-total-size=1536G --file-test-mode=rndrw cleanup
{% endhighlight %}


---

[pmap]
-

\#根据常驻内核使用大小排序

{% highlight bash %}
pmap -x 14769 | sort -k3 -n
{% endhighlight %}


---

[ss]
-


\#查看socket统计信息

{% highlight bash %}
ss -s
{% endhighlight %}

---

[ssh-keygen]
-

\#根据私钥生成公钥

{% highlight bash %}
ssh-keygen -f ~/.ssh/id_rsa -y 
{% endhighlight %}

\#创建与添加公钥

{% highlight bash %}
ssh-keygen -t rsa
vim authorized_keys 
   (add pub key)
chmod 600 authorized_keys
{% endhighlight %}

---

[iptables]
-


\#SNAT 和 DNAT 

{% highlight bash %}
iptables -t nat -A POSTROUTING  -o eth1 -j MASQUERADE

iptables -t nat  -A PREROUTING -p tcp -m tcp --dport 10222 -j DNAT --to-destination 192.168.1.123:22
{% endhighlight %}

---

[perl]
-

\#模拟交互

{% highlight bash %}
[root@iZ23n5z6svlZ ~]# perl -e 'print "==>";while(<STDIN>){eval($_);print "\n==>";}'
==>$a="lala";

==>print $a."\n";
lala

==>
{% endhighlight %}

\#one line program


{% highlight perl %}
perl -lne 'print $1 if (/(\b\S+map\S+\b)/)' tm |grep srw | sort -u
cat jjkk  | grep -E 'Hostname:|Function:'| perl -nle 'print $1.$2 if(/(Hostname:|ID:|Function:|Description:|Test URI:|Ping Test:|Nukit:|ValidateInternals:|Special:|Rank:|Bladecenter:).*>([^<>]+)(<\/\w+>)+$/) '| sed 's/&nbsp;//'
cat jjkk  | grep -E 'Hostname:|ID:|Function:|Description:|Test URI:|Ping Test:|Nukit:|ValidateInternals:|Special:|Rank:|Bladecenter:'| perl -nle 'print $1.$2 if(/(Hostname:|ID:|Function:|Description:|Test URI:|Ping Test:|Nukit:|ValidateInternals:|Special:|Rank:|Bladecenter:).*>([^<>]+)(<\/\w+>)+$/) '
perl -nle 's/(^\s+|\s+$)//g;print $_ unless (/^$/)' jk
perl -nle 's/(^\s+|\s+$)//g;' -e  'print $_ unless (/^$/)' jk
perl -i.bak  -p -e 'BEGIN{$flag=2}  if (/^~~~/ and $flag == 2) {s/^~~~/startflag/ ; $flag=3 } ; if (/^~~~/ and $flag == 3) {s/^~~~/endflag/ ; $flag=2 }'   jk.md
{% endhighlight %}

---

[paste]
-


{% highlight bash %}
cat u |cut -c 2-11 > time_s
paste <(for i in `cat time_s`; do date -d "1970-01-01 UTC $i seconds" +"%x %X%z %Z" ; done)  <(cat u)
{% endhighlight %}

---

[exec]
-

{% highlight bash %}
exec 8>u.log1
exec 9>u.log2
ll /proc/self/fd
{% endhighlight %}

---

[nmap]
-

{% highlight bash %}
nmap -R -vv -T4 -p 1-65535
nmap -R -vv -T4 -p 1  140.211.161.192/27   | grep 'host down'
nmap -R -vv -T4 -p 1  192.168.1.20/24   | grep 'host down'
{% endhighlight %}

---

[ip]
-

{% highlight bash %}
ip add add 192.168.1.49/24  dev eth0
ip add del 192.168.1.49/24  dev eth0 
ip route flush table test
ip route show table test 
ip route add default via 192.168.55.2 dev eth1 table test
ip rule add from 192.168.55.0/24 table test 
{% endhighlight %}

---

[for]
-

{% highlight bash %}
for i in `cat u`;do ng_find $i >>u.path 2>>u.log; done
for i in `cat u.path`; do sed -i 's=soft.com.soft.com.log=soft.com.log=' $i; done
for i in `cat u.path` ; do  sed -i -r -e 's=/log/zdr/(ndc-\w+-\w+).soft.com.log=/log/zdr/\1.log=' -e 's=/log/(ndc-\w+-\w+).soft.com.log=/log/\1.log=' $i; done 
for i in `cat u.address2`; do nexec -i -l $i ln -s /usr/local/ssl/lib/libssl.so.0.9.8 /usr/lib/libssl.so.0.9.8; nexec -i -l $i ln -s /usr/local/ssl/lib/libcrypto.so.0.9.8  /usr/lib/libcrypto.so.0.9.8; done
(for i in `cat u.path`;do sed -i -r -e "s=(-p\s+'/opt/(\w+)/\S+)=\1 -u \2=" $i;done) >u.log 2>u.log2
(for i in `cat u`;do nexec -i -l $i ln -s /usr/local/ssl/lib/libssl.so.0.9.8 /usr/lib/libssl.so.0.9.8; nexec -i -l $i ln -s /usr/local/ssl/lib/libcrypto.so.0.9.8  /usr/lib/libcrypto.so.0.9.8; done) 1>&8 2>&9
for i in `cat time_s`; do date -d "1970-01-01 UTC $i seconds" +"%x %X%z %Z" ; done;
for i in `cat o2`; do ng_disable_check -H $i -y -q -a wfang01 -c troubleshooting ; done
for i in `cat u.path`; do  sed -i -r -f p $i; done
{% endhighlight %}

---

[sed]
-

{% highlight bash %}
sed -n '/03:00:52/,$p' messages
sed -ri -e 's/(contacts\s+\w+)/\1,sganes/' -e 's/sganes,sganes/sganes/' {n,e}dc-emsimg-app{1..4}.cfg
sed -ri -e 's/nobody/sganes/' -e  's/(contacts\s+\w+)/\1,sganes/' -e 's/sganes,sganes/sganes/' {n,e}dc-emsts-app{1..4}.cfg
sed -i 's=soft.com.soft.com.log=soft.com.log=' ndc-v5-dalprs3.cfg 
sed  -ri -e 's=/log/zdr/(ndc-\w+-\w+).soft.com.log=/log/zdr/\1.log=' -e 's=/log/(ndc-\w+-\w+).soft.com.log=/log/\1.log=' ndc-v5-l2cacheprs8.cfg
sed -r -e "s=(-p\s+'/opt/(\w+)/\S+)=\1 -u \2=" test2
sed -ri -e "s=(-p\s+'/opt/(\w+-\w+)/\S+)=\1 -u \2=" /Monitoring/nagios/etc/objects/VIP/ndc-vwmriskp1.cfg
sed -r 's=(.*?([en]dc[-1-9a-z]+).*?open log file (/\S+):.*?)=\2\t\3=' test|sort -u > u
sed -r "s=with ip (\S+),=\#\1\#=" error |cut -f 2 -d '#' |sort -rn |uniq
sed -r -e "s=(\S*CORE\S*)=\#\1\#="  -e "s=(\S*SQL\S*)=\#\1\#=" error.firewall  |cut -f 2,4 -d '#' |sort -rn | grep CORE |uniq > error.useful
sed -r 's=(to|for|for DataSource)\s+(\S+[._]\S+)=\1 \#\2\#=g'  ui  | grep '#'|cut -f 2 -d '#'| sort -u
sed -e 's!^!http://tmr-master.vip.fucktest.com/cgi-bin/epstatus.cgi?submitted=yes\&input=ep\&name=!' -e 's!$!\&quest=epsync\&swd_package=!' b 
s=(check_remote!check_http!-a\s+\"-H\s+[ne]dc[-0-9a-z]+\s+-u\s+)\S+\s+(.*)=\1'/verify.jsp' -p 1730 \2=

grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'| sed 's/^/add server /'
grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'|  sed -r 's/(.*)/add service test_\1-9000 \1 HTTP 9000 -gslb NONE -maxClient 0 -maxReq 0 -cip ENABLED X-fucktest-Web-Tier-IP -usip NO -useproxyport YES -sp OFF -cltTimeout 180 -svrTimeout 360 -CKA NO -TCPB YES -CMP NO -state DISABLED -downStateFlush DISABLED -appflowLog DISABLED/'
grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'| cut -f 1  | sed -r 's/(.*)/add service test_\1-9000 \1 HTTP 9000 -gslb NONE -maxClient 0 -maxReq 0 -cip ENABLED X-fucktest-Web-Tier-IP -usip NO -useproxyport YES -sp OFF -cltTimeout 180 -svrTimeout 360 -CKA NO -TCPB YES -CMP NO -state DISABLED -downStateFlush DISABLED -appflowLog DISABLED/'
grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'| cut -f 1  | sed -r 's/(.*)/add service test_\1-9000 \1 HTTP 9000 -gslb NONE -maxClient 0 -maxReq 0 -cip ENABLED X-fucktest-Web-Tier-IP -usip NO -useproxyport YES -sp OFF -cltTimeout 180 -svrTimeout 360 -CKA NO -TCPB YES -CMP NO -state DISABLED -downStateFlush DISABLED -appflowLog DISABLED/'
grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'| cut -f 1  | sed -r 's/(.*)/bind lb monitor v3app_ecv  test_\1-9000/'
 ../find-unused-ip.sh  10.91.121.0/24
grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'| cut -f 1  | sed -r 's/(.*)/bind lb monitor v3app_ecv  test_\1-9000/'
grep  'beijing' test  | sed 's/.ecop.beijing.fucktest.com//'| cut -f 1  | sed -r 's/(.*)/bind lb vserver test-app-1-9000  test_\1-9000/'
grep  -v 'beijing' test  | sed 's/.ecop.hangzhou.fucktest.com//'| cut -f 1  | sed -r 's/(.*)/bind lb vserver test-app-1-9000  test_\1-9000/'

sed -r "s=with ip (\S+),=\#\1\#=" orphan |cut -f 2 -d '#' |sort -rn |uniq 
sed -n '331,360p' or2 > 360
sed -n '361,$p' or2 > 381
cat jk| sed '/com$/{
N
s/com\nFunction/com\tFunction/
}'
{% endhighlight %}

---

[awk]
-



{% highlight bash %}
ps faux   | grep "sshd" | grep -v 'grep' | awk 'BEGIN{sum=0;}{sum=sum+$6;}END{print sum*1024;}'
ps faux  | awk 'BEGIN{sum=0}{sum=sum+$6}END{print sum/1024}'
{% endhighlight %}


---

[jekyll]
-

{% highlight bash %}
jekyll server  --host 0.0.0.0 --port 8080
{% endhighlight %}
