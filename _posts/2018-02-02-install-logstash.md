---
layout: post
title: "Install Logstash"
author:  wilmosfang
date: 2018-02-02 12:38:04
image: '/assets/img/'
excerpt: 'Logstash 的安装方法'
main-class: es
color: '#51bcb2'
tags:
 - logstash
 - es
categories:
 - es
twitter_text: 'simple process of Logstash installation'
introduction: 'installation method of Logstash'
---



# 前言

**[Logstash][logstash]** 是一个开源的数据收集加工和传输管道

>Logstash is an open source data collection engine with real-time pipelining capabilities

具有强大灵活的数据收集处理能力，并且可以几乎实时地传送给后端数据存储引擎，常与 Elasticsearch 和 Kibana 一起组成 ELK 技术栈，给日志分析带来极大的便利


![logstash](/assets/img/logstash/logstash01.png)

这里分享一下 **[Logstash][logstash]** 的安装方法

参考 **[Installing Logstash][logstash_install]**

> **Tip:** 当前版本 **Version:6.1.3**



---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 71a5a14bde634bfc8c5bafb7d9442f9e
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
    link/ether 08:00:27:d1:5d:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85051sec preferred_lft 85051sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:47:20:56 brd ff:ff:ff:ff:ff:ff
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


## 导入 GPG key


~~~
[root@much ~]# rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
[root@much ~]#
~~~


## 配置 repo

配置 logstash 仓库

~~~
[root@much ~]# vim /etc/yum.repos.d/logstash.repo
[root@much ~]# cat /etc/yum.repos.d/logstash.repo
[logstash-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
[root@much ~]#
~~~

>**Tips:** 也可以在这里直接下载 **[Download Logstash][logstash_dl]**


## 安装软件

~~~
[root@much ~]# yum install logstash
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 8.0 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
jenkins                                                  | 2.9 kB     00:00     
kibana-6.x                                               | 1.3 kB     00:00     
logstash-6.x                                             | 1.3 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/4): logstash-6.x/primary                                |  31 kB   00:05     
(2/4): epel/x86_64/updateinfo                              | 880 kB   00:15     
(3/4): epel/x86_64/primary_db                              | 6.2 MB   00:34     
(4/4): updates/7/x86_64/primary_db                         | 6.0 MB   00:41     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media:
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
logstash-6.x                                                              90/90
Resolving Dependencies
--> Running transaction check
---> Package logstash.noarch 1:6.1.3-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch            Version            Repository           Size
================================================================================
Installing:
 logstash          noarch          1:6.1.3-1          kibana-6.x          114 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 114 M
Installed size: 203 M
Is this ok [y/d/N]: y
Downloading packages:
logstash-6.1.3.rpm                                         | 114 MB   46:52     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 1:logstash-6.1.3-1.noarch                                    1/1
Using provided startup.options file: /etc/logstash/startup.options
Successfully created system startup script for Logstash
  Verifying  : 1:logstash-6.1.3-1.noarch                                    1/1

Installed:
  logstash.noarch 1:6.1.3-1                                                     

Complete!
[root@much ~]#
~~~


## 查看帮助

~~~
[root@much ~]# /usr/share/logstash/bin/logstash -h
WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
Usage:
    bin/logstash [OPTIONS]

Options:
    -n, --node.name NAME          Specify the name of this logstash instance, if no value is given
                                  it will default to the current hostname.
                                   (default: "much")
    -f, --path.config CONFIG_PATH Load the logstash config from a specific file
                                  or directory.  If a directory is given, all
                                  files in that directory will be concatenated
                                  in lexicographical order and then parsed as a
                                  single config file. You can also specify
                                  wildcards (globs) and any matched files will
                                  be loaded in the order described above.
    -e, --config.string CONFIG_STRING Use the given string as the configuration
                                  data. Same syntax as the config file. If no
                                  input is specified, then the following is
                                  used as the default input:
                                  "input { stdin { type => stdin } }"
                                  and if no output is specified, then the
                                  following is used as the default output:
                                  "output { stdout { codec => rubydebug } }"
                                  If you wish to use both defaults, please use
                                  the empty string for the '-e' flag.
                                   (default: nil)
    --modules MODULES             Load Logstash modules.
                                  Modules can be defined using multiple instances
                                  '--modules module1 --modules module2',
                                     or comma-separated syntax
                                  '--modules=module1,module2'
                                  Cannot be used in conjunction with '-e' or '-f'
                                  Use of '--modules' will override modules declared
                                  in the 'logstash.yml' file.
    -M, --modules.variable MODULES_VARIABLE Load variables for module template.
                                  Multiple instances of '-M' or
                                  '--modules.variable' are supported.
                                  Ignored if '--modules' flag is not used.
                                  Should be in the format of
                                  '-M "MODULE_NAME.var.PLUGIN_TYPE.PLUGIN_NAME.VARIABLE_NAME=VALUE"'
                                  as in
                                  '-M "example.var.filter.mutate.fieldname=fieldvalue"'
    --setup                       Load index template into Elasticsearch, and saved searches,
                                  index-pattern, visualizations, and dashboards into Kibana when
                                  running modules.
                                   (default: false)
    --cloud.id CLOUD_ID           Sets the elasticsearch and kibana host settings for
                                  module connections in Elastic Cloud.
                                  Your Elastic Cloud User interface or the Cloud support
                                  team should provide this.
                                  Add an optional label prefix '<label>:' to help you
                                  identify multiple cloud.ids.
                                  e.g. 'staging:dXMtZWFzdC0xLmF3cy5mb3VuZC5pbyRub3RhcmVhbCRpZGVudGlmaWVy'
    --cloud.auth CLOUD_AUTH       Sets the elasticsearch and kibana username and password
                                  for module connections in Elastic Cloud
                                  e.g. 'username:<password>'
    -w, --pipeline.workers COUNT  Sets the number of pipeline workers to run.
                                   (default: 2)
    --experimental-java-execution (Experimental) Use new Java execution engine.
                                   (default: false)
    -b, --pipeline.batch.size SIZE Size of batches the pipeline is to work in.
                                   (default: 125)
    -u, --pipeline.batch.delay DELAY_IN_MS When creating pipeline batches, how long to wait while polling
                                  for the next event.
                                   (default: 5)
    --pipeline.unsafe_shutdown    Force logstash to exit during shutdown even
                                  if there are still inflight events in memory.
                                  By default, logstash will refuse to quit until all
                                  received events have been pushed to the outputs.
                                   (default: false)
    --path.data PATH              This should point to a writable directory. Logstash
                                  will use this directory whenever it needs to store
                                  data. Plugins will also have access to this path.
                                   (default: "/usr/share/logstash/data")
    -p, --path.plugins PATH       A path of where to find plugins. This flag
                                  can be given multiple times to include
                                  multiple paths. Plugins are expected to be
                                  in a specific directory hierarchy:
                                  'PATH/logstash/TYPE/NAME.rb' where TYPE is
                                  'inputs' 'filters', 'outputs' or 'codecs'
                                  and NAME is the name of the plugin.
                                   (default: [])
    -l, --path.logs PATH          Write logstash internal logs to the given
                                  file. Without this flag, logstash will emit
                                  logs to standard output.
                                   (default: "/usr/share/logstash/logs")
    --log.level LEVEL             Set the log level for logstash. Possible values are:
                                    - fatal
                                    - error
                                    - warn
                                    - info
                                    - debug
                                    - trace
                                   (default: "info")
    --config.debug                Print the compiled config ruby code out as a debug log (you must also have --log.level=debug enabled).
                                  WARNING: This will include any 'password' options passed to plugin configs as plaintext, and may result
                                  in plaintext passwords appearing in your logs!
                                   (default: false)
    -i, --interactive SHELL       Drop to shell instead of running as normal.
                                  Valid shells are "irb" and "pry"
    -V, --version                 Emit the version of logstash and its friends,
                                  then exit.
    -t, --config.test_and_exit    Check configuration for valid syntax and then exit.
                                   (default: false)
    -r, --config.reload.automatic Monitor configuration changes and reload
                                  whenever it is changed.
                                  NOTE: use SIGHUP to manually reload the config
                                   (default: false)
    --config.reload.interval RELOAD_INTERVAL How frequently to poll the configuration location
                                  for changes, in seconds.
                                   (default: 3000000000)
    --http.host HTTP_HOST         Web API binding host (default: "127.0.0.1")
    --http.port HTTP_PORT         Web API http port (default: 9600..9700)
    --log.format FORMAT           Specify if Logstash should write its own logs in JSON form (one
                                  event per line) or in plain text (using Ruby's Object#inspect)
                                   (default: "plain")
    --path.settings SETTINGS_DIR  Directory containing logstash.yml file. This can also be
                                  set through the LS_SETTINGS_DIR environment variable.
                                   (default: "/usr/share/logstash/config")
    --verbose                     Set the log level to info.
                                  DEPRECATED: use --log.level=info instead.
    --debug                       Set the log level to debug.
                                  DEPRECATED: use --log.level=debug instead.
    --quiet                       Set the log level to info.
                                  DEPRECATED: use --log.level=quiet instead.
    -h, --help                    print help
[root@much ~]#
~~~


## 输出输入

~~~
[root@much ~]# /usr/share/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'
WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
Could not find log4j2 configuration at path /usr/share/logstash/config/log4j2.properties. Using default config which logs errors to the console
The stdin plugin is now waiting for input:
just for test
2018-02-02T13:52:32.823Z much just for test
hello world
2018-02-02T13:52:36.364Z much hello world
abc test
2018-02-02T13:52:38.933Z much abc test

2018-02-02T13:52:41.270Z much

2018-02-02T13:52:41.461Z much

2018-02-02T13:52:41.657Z much

2018-02-02T13:52:41.819Z much

2018-02-02T13:52:41.988Z much
[root@much ~]#
~~~

这个简单的实例，是把输入的内容直接打印输出到终端

![logstash](/assets/img/logstash/basic_logstash_pipeline.png)

---

# 总结

使用 rpm 包安装的过程非常简单

* TOC
{:toc}


---

[logstash]:https://www.elastic.co/products/logstash
[logstash_install]:https://www.elastic.co/guide/en/logstash/current/installing-logstash.html
[logstash_dl]:https://www.elastic.co/downloads/logstash
