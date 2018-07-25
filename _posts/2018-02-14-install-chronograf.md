---
layout: post
title: "Install Chronograf"
author:  wilmosfang
date: 2018-02-14 14:00:52
image: '/assets/img/'
excerpt: '安装 Chronograf'
main-class: influxdb
color: '#4591ed'
tags:
 - chronograf
 - influxdb
categories:
 - influxdb
twitter_text: 'simple process of Chronograf installation'
introduction: 'installation of Chronograf'
---


## 前言

**[InfluxDB][influxdb]** 是一款高性能的时序数据库，见长于标量的时序存储

类似于 **Elasticsearch** 的 **ELK** 技术栈，**InfluxDB** 也有一套 **[TICK][tick]** 技术栈

其中 **[Chronograf][chronograf]** 是展示数据的前端 UI 组件

>Chronograf is the user interface component of InfluxData’s TICK Stack. It makes the monitoring and alerting for your infrastructure easy to setup and maintain. It is simple to use and includes templates and libraries to allow you to rapidly build dashboards with real-time visualizations of your data

这里分享一下 **[Chronograf][chronograf]** 的安装方法

参考 **[Installing Chronograf][chronograf_install]**

> **Tip:** 当前的版本为 **chronograf-1.4.1.2**

---

# 操作


## 环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 1f9d9f1fc29440c8874b993d9455c898
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e3:df:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83032sec preferred_lft 83032sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d3:ec:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 配置仓库

~~~
[root@much ~]# cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
> [influxdb]
> name = InfluxDB Repository - RHEL \$releasever
> baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
> enabled = 1
> gpgcheck = 1
> gpgkey = https://repos.influxdata.com/influxdb.key
> EOF
[influxdb]
name = InfluxDB Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
[root@much ~]# cat /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
[root@much ~]#
~~~

## 依赖

在 **[TICK][tick]** 技术栈中，**[Chronograf][chronograf]** 的正常工作依赖 **[InfluxDB][influxdb]**

**Telegraf** 是建议安装的，但并非必要，因为它可以为系统不断提供新数据

**Kapacitor** 是可选安装的，数据处理与报警处理等高级功能才需要依赖它


## 安装软件

~~~
[root@much telegraf]# rpm -qa | grep influx
influxdb-1.4.2-1.x86_64
[root@much telegraf]# yum list all | grep influx
influxdb.x86_64                         1.4.2-1                        @influxdb
telegraf.x86_64                         1.5.2-1                        @influxdb
chronograf.x86_64                       1.4.0.0-1                      influxdb
kapacitor.x86_64                        1.4.0-1                        influxdb
pcp-export-pcp2influxdb.x86_64          3.11.8-7.el7                   base     
[root@much telegraf]# yum install chronograf.x86_64
Loaded plugins: fastestmirror, langpacks
base                                                                                   | 3.6 kB  00:00:00     
c7-media                                                                               | 3.6 kB  00:00:00     
epel/x86_64/metalink                                                                   | 7.3 kB  00:00:00     
epel                                                                                   | 4.7 kB  00:00:00     
extras                                                                                 | 3.4 kB  00:00:00     
influxdb                                                                               | 2.5 kB  00:00:00     
kibana-6.x                                                                             | 1.3 kB  00:00:00     
updates                                                                                | 3.4 kB  00:00:00     
(1/2): epel/x86_64/updateinfo                                                          | 880 kB  00:00:03     
(2/2): epel/x86_64/primary_db                                                          | 6.2 MB  00:00:29     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package chronograf.x86_64 0:1.4.0.0-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================
 Package                    Arch                   Version                     Repository                Size
==============================================================================================================
Installing:
 chronograf                 x86_64                 1.4.0.0-1                   influxdb                 7.6 M

Transaction Summary
==============================================================================================================
Install  1 Package

Total download size: 7.6 M
Installed size: 7.6 M
Is this ok [y/d/N]: y
Downloading packages:
chronograf-1.4.0.0.x86_64.rpm                                                          | 7.6 MB  00:00:22     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : chronograf-1.4.0.0-1.x86_64                                                                1/1
Created symlink from /etc/systemd/system/multi-user.target.wants/chronograf.service to /usr/lib/systemd/system/chronograf.service.
  Verifying  : chronograf-1.4.0.0-1.x86_64                                                                1/1

Installed:
  chronograf.x86_64 0:1.4.0.0-1                                                                               

Complete!
[root@much telegraf]# echo $?
0
[root@much telegraf]# rpm -qa | grep chronograf
chronograf-1.4.0.0-1.x86_64
[root@much telegraf]#
~~~

可以看看有些什么东西

~~~
[root@much telegraf]# rpm -ql chronograf-1.4.0.0-1.x86_64
/etc/logrotate.d/chronograf
/usr/bin/chronograf
/usr/lib/chronograf/scripts/chronograf.service
/usr/lib/chronograf/scripts/init.sh
/usr/share/chronograf/canned/apache.json
/usr/share/chronograf/canned/consul.json
/usr/share/chronograf/canned/consul_agent.json
/usr/share/chronograf/canned/consul_cluster.json
/usr/share/chronograf/canned/consul_election.json
/usr/share/chronograf/canned/consul_http.json
/usr/share/chronograf/canned/consul_leadership.json
/usr/share/chronograf/canned/consul_serf_events.json
/usr/share/chronograf/canned/cpu.json
/usr/share/chronograf/canned/disk.json
/usr/share/chronograf/canned/diskio.json
/usr/share/chronograf/canned/docker.json
/usr/share/chronograf/canned/docker_blkio.json
/usr/share/chronograf/canned/docker_net.json
/usr/share/chronograf/canned/elasticsearch.json
/usr/share/chronograf/canned/haproxy.json
/usr/share/chronograf/canned/influxdb_database.json
/usr/share/chronograf/canned/influxdb_httpd.json
/usr/share/chronograf/canned/influxdb_queryExecutor.json
/usr/share/chronograf/canned/influxdb_write.json
/usr/share/chronograf/canned/kubernetes_node.json
/usr/share/chronograf/canned/kubernetes_pod_container.json
/usr/share/chronograf/canned/kubernetes_pod_network.json
/usr/share/chronograf/canned/kubernetes_system_container.json
/usr/share/chronograf/canned/load.json
/usr/share/chronograf/canned/mem.json
/usr/share/chronograf/canned/memcached.json
/usr/share/chronograf/canned/mesos.json
/usr/share/chronograf/canned/mongodb.json
/usr/share/chronograf/canned/mysql.json
/usr/share/chronograf/canned/net.json
/usr/share/chronograf/canned/netstat.json
/usr/share/chronograf/canned/nginx.json
/usr/share/chronograf/canned/nsq_channel.json
/usr/share/chronograf/canned/nsq_server.json
/usr/share/chronograf/canned/nsq_topic.json
/usr/share/chronograf/canned/phpfpm.json
/usr/share/chronograf/canned/ping.json
/usr/share/chronograf/canned/postgresql.json
/usr/share/chronograf/canned/processes.json
/usr/share/chronograf/canned/procstat.json
/usr/share/chronograf/canned/rabbitmq.json
/usr/share/chronograf/canned/redis.json
/usr/share/chronograf/canned/riak.json
/usr/share/chronograf/canned/varnish.json
/usr/share/chronograf/canned/win_cpu.json
/usr/share/chronograf/canned/win_mem.json
/usr/share/chronograf/canned/win_net.json
/usr/share/chronograf/canned/win_system.json
/usr/share/chronograf/canned/win_websvc.json
/var/lib/chronograf
/var/log/chronograf
[root@much telegraf]#
~~~


## 启动服务

~~~
[root@much telegraf]# systemctl status chronograf.service
● chronograf.service - Open source monitoring and visualization UI for the entire TICK stack.
   Loaded: loaded (/usr/lib/systemd/system/chronograf.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://www.influxdata.com/time-series-platform/chronograf/
[root@much telegraf]# systemctl start chronograf.service
[root@much telegraf]# systemctl status chronograf.service
● chronograf.service - Open source monitoring and visualization UI for the entire TICK stack.
   Loaded: loaded (/usr/lib/systemd/system/chronograf.service; enabled; vendor preset: disabled)
   Active: active (running) since 三 2018-02-14 22:58:22 CST; 2s ago
     Docs: https://www.influxdata.com/time-series-platform/chronograf/
 Main PID: 5066 (chronograf)
   CGroup: /system.slice/chronograf.service
           └─5066 /usr/bin/chronograf --host 0.0.0.0 --port 8888 -b /var/lib/chronograf/chronograf-v1.db -c...

2月 14 22:58:22 much systemd[1]: Started Open source monitoring and visualization UI for the entire T...ack..
2月 14 22:58:22 much systemd[1]: Starting Open source monitoring and visualization UI for the entire ...k....
2月 14 22:58:22 much chronograf[5066]: time="2018-02-14T22:58:22+08:00" level=info msg="Serving chrono...rver
2月 14 22:58:22 much chronograf[5066]: time="2018-02-14T22:58:22+08:00" level=info msg="Reporting usag...ime"
Hint: Some lines were ellipsized, use -l to show in full.
[root@much telegraf]# ps faux | grep chronograf
root      5084  0.0  0.0 112660  1020 pts/0    S+   22:58   0:00          \_ grep --color=auto chronograf
chronog+  5066  0.5  0.2 218840 10852 ?        Ssl  22:58   0:00 /usr/bin/chronograf --host 0.0.0.0 --port 8888 -b /var/lib/chronograf/chronograf-v1.db -c /usr/share/chronograf/canned
[root@much telegraf]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN      1318/java           
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      1318/java           
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1544/dnsmasq        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1317/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1313/cupsd          
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN      1320/influxd        
tcp        0      0 10.0.2.15:50254         104.131.151.204:443     ESTABLISHED 5066/chronograf     
tcp        0      0 127.0.0.1:49498         127.0.0.1:8086          ESTABLISHED 4943/telegraf       
tcp        0      0 192.168.56.208:22       192.168.56.1:44842      ESTABLISHED 1876/sshd: root@pts
tcp6       0      0 :::8086                 :::*                    LISTEN      1320/influxd        
tcp6       0      0 :::22                   :::*                    LISTEN      1317/sshd           
tcp6       0      0 :::8888                 :::*                    LISTEN      5066/chronograf     
tcp6       0      0 :::3000                 :::*                    LISTEN      1319/grafana-server
tcp6       0      0 127.0.0.1:8086          127.0.0.1:49498         ESTABLISHED 1320/influxd        
[root@much telegraf]#
~~~

可以看到 **chronograf** 监听在了本地的 **8888** 端口

## 打开防火墙

~~~
[root@much telegraf]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 3000/tcp 8080/tcp 5601/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much telegraf]# firewall-cmd --add-port 8888/tcp --permanent
success
[root@much telegraf]# firewall-cmd --reload
success
[root@much telegraf]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports: 3000/tcp 8888/tcp 8080/tcp 5601/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

[root@much telegraf]#
~~~


## 进行访问


**Username** 和 **Password** 可以为空

![chronograf](/assets/img/chronograf/chronograf01.png)

查看 **hostlist**

![chronograf](/assets/img/chronograf/chronograf02.png)

进入可以查看主机的负载信息(由telegraf在默认配置下收集的标量信息)

![chronograf](/assets/img/chronograf/chronograf03.png)

也可以交互式探索数据

![chronograf](/assets/img/chronograf/chronograf04.png)

非常直观易用，上面是查询语句，中间是标量信息，下面是图形



---

# 总结

**chronograf** 作为 influxdb 的前端数据展示组件，与 influxdb 的对接非常简单(默认不作配置都是直接连接的本地 influxdb)

界面简洁美观，和 **grafana** 风格比较像

* TOC
{:toc}


---


[influxdb]:https://portal.influxdata.com/
[chronograf]:https://www.influxdata.com/time-series-platform/chronograf/
[tick]:https://www.influxdata.com/time-series-platform/
[chronograf_install]:http://docs.influxdata.com/chronograf/v1.4/introduction/installation/
