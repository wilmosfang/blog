---
layout: post
title:  Logstash  Plugins
author: wilmosfang
categories:  linux logstash ruby
wc: 751  1934 25341 
excerpt:  Logstash 插件相关基础与使用方法
comments: true
---



# 前言


**[Logstash][logstash]** 有一套灵活的插件机制，用来方便地扩展 **[Logstash][logstash]** 的能力和特性

由于 **[Logstash][logstash]** 是使用ruby写的，所以它的插件其实就是各种gem 

>Logstash has a rich collection of input, filter, codec and output plugins. Plugins are available as self-contained packages called gems and hosted on RubyGems.org. The plugin manager accesed via bin/plugin script is used to manage the lifecycle of plugins in your Logstash deployment. You can install, uninstall and upgrade plugins using these Command Line Interface (CLI) described below.

下面分享一下 **[Logstash Plugins][logstash_plugins]** 的基础使用方法


> **Tip:** 当前的最新版本为 **Logstash 2.1.1** 

---


# 概要

* TOC
{:toc}


---

## plugin命令

### 获取帮助


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin --help
Usage:
    bin/plugin [OPTIONS] SUBCOMMAND [ARG] ...

Parameters:
    SUBCOMMAND                    subcommand
    [ARG] ...                     subcommand arguments

Subcommands:
    install                       Install a plugin
    uninstall                     Uninstall a plugin
    update                        Update a plugin
    pack                          Package currently installed plugins
    unpack                        Unpack packaged plugins
    list                          List all installed plugins

Options:
    -h, --help                    print help
[root@h102 ~]# /opt/logstash/bin/plugin list --help
Usage:
    bin/plugin list [OPTIONS] [PLUGIN]

Parameters:
    [PLUGIN]                      Part of plugin name to search for, leave empty for all plugins

Options:
    --installed                   List only explicitly installed plugins using bin/plugin install ... (default: false)
    --verbose                     Also show plugin version number (default: false)
    --group NAME                  Filter plugins per group: input, output, filter or codec
    -h, --help                    print help
[root@h102 ~]# 
{% endhighlight %}

---

### plugin list

{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin list 
logstash-codec-collectd
logstash-codec-dots
logstash-codec-edn
logstash-codec-edn_lines
logstash-codec-es_bulk
logstash-codec-fluent
logstash-codec-graphite
logstash-codec-json
logstash-codec-json_lines
logstash-codec-line
logstash-codec-msgpack
logstash-codec-multiline
logstash-codec-netflow
logstash-codec-oldlogstashjson
logstash-codec-plain
logstash-codec-rubydebug
logstash-filter-anonymize
logstash-filter-checksum
logstash-filter-clone
logstash-filter-csv
logstash-filter-date
logstash-filter-dns
logstash-filter-drop
logstash-filter-fingerprint
logstash-filter-geoip
logstash-filter-grok
logstash-filter-json
logstash-filter-kv
logstash-filter-metrics
logstash-filter-multiline
logstash-filter-mutate
logstash-filter-ruby
logstash-filter-sleep
logstash-filter-split
logstash-filter-syslog_pri
logstash-filter-throttle
logstash-filter-urldecode
logstash-filter-useragent
logstash-filter-uuid
logstash-filter-xml
logstash-input-beats
logstash-input-couchdb_changes
logstash-input-elasticsearch
logstash-input-eventlog
logstash-input-exec
logstash-input-file
logstash-input-ganglia
logstash-input-gelf
logstash-input-generator
logstash-input-graphite
logstash-input-heartbeat
logstash-input-http
logstash-input-imap
logstash-input-irc
logstash-input-jdbc
logstash-input-kafka
logstash-input-log4j
logstash-input-lumberjack
logstash-input-pipe
logstash-input-rabbitmq
logstash-input-redis
logstash-input-s3
logstash-input-snmptrap
logstash-input-sqs
logstash-input-stdin
logstash-input-syslog
logstash-input-tcp
logstash-input-twitter
logstash-input-udp
logstash-input-unix
logstash-input-xmpp
logstash-input-zeromq
logstash-output-cloudwatch
logstash-output-csv
logstash-output-elasticsearch
logstash-output-email
logstash-output-exec
logstash-output-file
logstash-output-ganglia
logstash-output-gelf
logstash-output-graphite
logstash-output-hipchat
logstash-output-http
logstash-output-irc
logstash-output-juggernaut
logstash-output-kafka
logstash-output-lumberjack
logstash-output-nagios
logstash-output-nagios_nsca
logstash-output-null
logstash-output-opentsdb
logstash-output-pagerduty
logstash-output-pipe
logstash-output-rabbitmq
logstash-output-redis
logstash-output-s3
logstash-output-sns
logstash-output-sqs
logstash-output-statsd
logstash-output-stdout
logstash-output-tcp
logstash-output-udp
logstash-output-xmpp
logstash-output-zeromq
logstash-patterns-core
[root@h102 ~]# /opt/logstash/bin/plugin list logstash-output-kafka
logstash-output-kafka
[root@h102 ~]# /opt/logstash/bin/plugin list kafka
logstash-input-kafka
logstash-output-kafka
[root@h102 ~]# /opt/logstash/bin/plugin list --verbose kafka
logstash-input-kafka (2.0.2)
logstash-output-kafka (2.0.1)
[root@h102 ~]# /opt/logstash/bin/plugin list --group codec
logstash-codec-collectd
logstash-codec-dots
logstash-codec-edn
logstash-codec-edn_lines
logstash-codec-es_bulk
logstash-codec-fluent
logstash-codec-graphite
logstash-codec-json
logstash-codec-json_lines
logstash-codec-line
logstash-codec-msgpack
logstash-codec-multiline
logstash-codec-netflow
logstash-codec-oldlogstashjson
logstash-codec-plain
logstash-codec-rubydebug
[root@h102 ~]# 
{% endhighlight %}


---

### plugin uninstall


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin uninstall -h 
Usage:
    bin/plugin uninstall [OPTIONS] PLUGIN

Parameters:
    PLUGIN                        plugin name

Options:
    -h, --help                    print help
[root@h102 ~]# 
{% endhighlight %}

#### 修改镜像源

{% highlight bash %}
[root@h102 logstash]# cd /opt/logstash/
[root@h102 logstash]# vim Gemfile
[root@h102 logstash]# grep source Gemfile
source "https://ruby.taobao.org"
[root@h102 logstash]# 
{% endhighlight %}

> **Note:** 如果不修改，所有涉及插件变更的操作都会报错，原因是 **The Great Wall** ，如下

{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin uninstall logstash-output-kafka
Uninstalling logstash-output-kafka
Error Bundler::InstallError, retrying 1/10
An error occurred while installing arr-pm (0.0.10), and Bundler cannot continue.
Make sure that `gem install arr-pm -v '0.0.10'` succeeds before bundling.
WARNING: SSLSocket#session= is not supported

^C
{% endhighlight %}


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin install logstash-output-kafka
Validating logstash-output-kafka
Unable to download data from https://rubygems.org - Connection reset by peer (https://rubygems.global.ssl.fastly.net/latest_specs.4.8.gz)
ERROR: Installation aborted, verification failed for logstash-output-kafka 
[root@h102 ~]# 
{% endhighlight %}

即便全局的Source指的没问题

{% highlight bash %}
[root@h102 ~]# gem source -l 
*** CURRENT SOURCES ***

https://ruby.taobao.org/
[root@h102 ~]# 
{% endhighlight %}

但局部配置会覆盖此配置，从而实际以无法访问的地址作为自己的镜像源

{% highlight bash %}
[root@h102 logstash]# grep source /opt/logstash/Gemfile
source "https://rubygems.org"
[root@h102 logstash]# 
{% endhighlight %}

报错就是这么产生的

解决方法就是修改此配置到权威且可以访问的地址

详细可以参考 **[Private Gem Repositories][private_rubygem]** 


---


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin list kafka 
logstash-input-kafka
logstash-output-kafka
[root@h102 ~]# /opt/logstash/bin/plugin uninstall logstash-output-kafka
Uninstalling logstash-output-kafka
[root@h102 ~]# /opt/logstash/bin/plugin list kafka 
logstash-input-kafka
[root@h102 ~]#
{% endhighlight %}

---

### plugin install


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin install -h
Usage:
    bin/plugin install [OPTIONS] [PLUGIN] ...

Parameters:
    [PLUGIN] ...                  plugin name(s) or file

Options:
    --version VERSION             version of the plugin to install
    --[no-]verify                 verify plugin validity before installation (default: true)
    --development                 install all development dependencies of currently installed plugins (default: false)
    --local                       force local-only plugin installation. see bin/plugin package|unpack (default: false)
    -h, --help                    print help
[root@h102 ~]# /opt/logstash/bin/plugin list kafka 
logstash-input-kafka
[root@h102 ~]# /opt/logstash/bin/plugin install logstash-output-kafka
Validating logstash-output-kafka
Installing logstash-output-kafka
Installation successful
[root@h102 ~]# /opt/logstash/bin/plugin list kafka 
logstash-input-kafka
logstash-output-kafka
[root@h102 ~]# 
{% endhighlight %}

---

### plugin update


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin update -h
Usage:
    bin/plugin update [OPTIONS] [PLUGIN] ...

Parameters:
    [PLUGIN] ...                  Plugin name(s) to upgrade to latest version

Options:
    --[no-]verify                 verify plugin validity before installation (default: true)
    --local                       force local-only plugin update. see bin/plugin package|unpack (default: false)
    -h, --help                    print help
[root@h102 ~]# /opt/logstash/bin/plugin update logstash-codec-edn_lines
Updating logstash-codec-edn_lines
No plugin updated
[root@h102 ~]#
{% endhighlight %}

由于当前没有更新的 **logstash-codec-edn_lines** ，所以没有更新


更新所有插件

{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin list --verbose 
logstash-codec-collectd (2.0.2)
logstash-codec-dots (2.0.2)
logstash-codec-edn (2.0.2)
logstash-codec-edn_lines (2.0.2)
logstash-codec-es_bulk (2.0.2)
logstash-codec-fluent (2.0.2)
logstash-codec-graphite (2.0.2)
logstash-codec-json (2.0.4)
logstash-codec-json_lines (2.0.2)
logstash-codec-line (2.0.2)
logstash-codec-msgpack (2.0.2)
logstash-codec-multiline (2.0.4)
logstash-codec-netflow (2.0.2)
logstash-codec-oldlogstashjson (2.0.2)
logstash-codec-plain (2.0.2)
logstash-codec-rubydebug (2.0.4)
logstash-filter-anonymize (2.0.2)
logstash-filter-checksum (2.0.2)
logstash-filter-clone (2.0.4)
logstash-filter-csv (2.1.0)
logstash-filter-date (2.0.2)
logstash-filter-dns (2.0.2)
logstash-filter-drop (2.0.2)
logstash-filter-fingerprint (2.0.2)
logstash-filter-geoip (2.0.4)
logstash-filter-grok (2.0.2)
logstash-filter-json (2.0.2)
logstash-filter-kv (2.0.2)
logstash-filter-metrics (3.0.0)
logstash-filter-multiline (2.0.3)
logstash-filter-mutate (2.0.2)
logstash-filter-ruby (2.0.2)
logstash-filter-sleep (2.0.2)
logstash-filter-split (2.0.2)
logstash-filter-syslog_pri (2.0.2)
logstash-filter-throttle (2.0.2)
logstash-filter-urldecode (2.0.2)
logstash-filter-useragent (2.0.3)
logstash-filter-uuid (2.0.3)
logstash-filter-xml (2.0.2)
logstash-input-beats (2.0.3)
logstash-input-couchdb_changes (2.0.2)
logstash-input-elasticsearch (2.0.2)
logstash-input-eventlog (3.0.1)
logstash-input-exec (2.0.4)
logstash-input-file (2.0.3)
logstash-input-ganglia (2.0.4)
logstash-input-gelf (2.0.2)
logstash-input-generator (2.0.2)
logstash-input-graphite (2.0.4)
logstash-input-heartbeat (2.0.2)
logstash-input-http (2.0.2)
logstash-input-imap (2.0.2)
logstash-input-irc (2.0.3)
logstash-input-jdbc (2.0.5)
logstash-input-kafka (2.0.2)
logstash-input-log4j (2.0.4)
logstash-input-lumberjack (2.0.5)
logstash-input-pipe (2.0.2)
logstash-input-rabbitmq (3.1.1)
logstash-input-redis (2.0.2)
logstash-input-s3 (2.0.3)
logstash-input-snmptrap (2.0.2)
logstash-input-sqs (2.0.3)
logstash-input-stdin (2.0.2)
logstash-input-syslog (2.0.2)
logstash-input-tcp (3.0.0)
logstash-input-twitter (2.2.0)
logstash-input-udp (2.0.3)
logstash-input-unix (2.0.4)
logstash-input-xmpp (2.0.3)
logstash-input-zeromq (2.0.2)
logstash-output-cloudwatch (2.0.2)
logstash-output-csv (2.0.2)
logstash-output-elasticsearch (2.2.0)
logstash-output-email (3.0.2)
logstash-output-exec (2.0.2)
logstash-output-file (2.2.0)
logstash-output-ganglia (2.0.2)
logstash-output-gelf (2.0.2)
logstash-output-graphite (2.0.2)
logstash-output-hipchat (3.0.2)
logstash-output-http (2.0.5)
logstash-output-irc (2.0.2)
logstash-output-juggernaut (2.0.2)
logstash-output-kafka (2.0.1)
logstash-output-lumberjack (2.0.4)
logstash-output-nagios (2.0.2)
logstash-output-nagios_nsca (2.0.3)
logstash-output-null (2.0.2)
logstash-output-opentsdb (2.0.2)
logstash-output-pagerduty (2.0.2)
logstash-output-pipe (2.0.2)
logstash-output-rabbitmq (3.0.6)
logstash-output-redis (2.0.2)
logstash-output-s3 (2.0.3)
logstash-output-sns (3.0.2)
logstash-output-sqs (2.0.2)
logstash-output-statsd (2.0.4)
logstash-output-stdout (2.0.3)
logstash-output-tcp (2.0.2)
logstash-output-udp (2.0.2)
logstash-output-xmpp (2.0.2)
logstash-output-zeromq (2.0.2)
logstash-patterns-core (2.0.2)
[root@h102 ~]# /opt/logstash/bin/plugin update 
You are updating logstash-input-jdbc to a new version 3.0.0, which may not be compatible with 2.0.5. are you sure you want to proceed (Y/N)?
y
Updating logstash-codec-collectd, logstash-codec-dots, logstash-codec-edn, logstash-codec-edn_lines, logstash-codec-es_bulk, logstash-codec-fluent, logstash-codec-graphite, logstash-codec-json, logstash-codec-json_lines, logstash-codec-line, logstash-codec-msgpack, logstash-codec-multiline, logstash-codec-netflow, logstash-codec-oldlogstashjson, logstash-codec-plain, logstash-codec-rubydebug, logstash-filter-anonymize, logstash-filter-checksum, logstash-filter-clone, logstash-filter-csv, logstash-filter-date, logstash-filter-dns, logstash-filter-drop, logstash-filter-fingerprint, logstash-filter-geoip, logstash-filter-grok, logstash-filter-json, logstash-filter-kv, logstash-filter-metrics, logstash-filter-multiline, logstash-filter-mutate, logstash-filter-ruby, logstash-filter-sleep, logstash-filter-split, logstash-filter-syslog_pri, logstash-filter-throttle, logstash-filter-urldecode, logstash-filter-useragent, logstash-filter-uuid, logstash-filter-xml, logstash-input-beats, logstash-input-couchdb_changes, logstash-input-elasticsearch, logstash-input-eventlog, logstash-input-exec, logstash-input-file, logstash-input-ganglia, logstash-input-gelf, logstash-input-generator, logstash-input-graphite, logstash-input-heartbeat, logstash-input-http, logstash-input-imap, logstash-input-irc, logstash-input-jdbc, logstash-input-kafka, logstash-input-log4j, logstash-input-lumberjack, logstash-input-pipe, logstash-input-rabbitmq, logstash-input-redis, logstash-input-s3, logstash-input-snmptrap, logstash-input-sqs, logstash-input-stdin, logstash-input-syslog, logstash-input-tcp, logstash-input-twitter, logstash-input-udp, logstash-input-unix, logstash-input-xmpp, logstash-input-zeromq, logstash-output-cloudwatch, logstash-output-csv, logstash-output-elasticsearch, logstash-output-email, logstash-output-exec, logstash-output-file, logstash-output-ganglia, logstash-output-gelf, logstash-output-graphite, logstash-output-hipchat, logstash-output-http, logstash-output-irc, logstash-output-juggernaut, logstash-output-kafka, logstash-output-lumberjack, logstash-output-nagios, logstash-output-nagios_nsca, logstash-output-null, logstash-output-opentsdb, logstash-output-pagerduty, logstash-output-pipe, logstash-output-rabbitmq, logstash-output-redis, logstash-output-s3, logstash-output-sns, logstash-output-sqs, logstash-output-statsd, logstash-output-stdout, logstash-output-tcp, logstash-output-udp, logstash-output-xmpp, logstash-output-zeromq
Error Bundler::InstallError, retrying 1/10
An error occurred while installing jruby-kafka (1.5.0), and Bundler cannot continue.
Make sure that `gem install jruby-kafka -v '1.5.0'` succeeds before bundling.
WARNING: SSLSocket#session= is not supported
Error Bundler::InstallError, retrying 2/10
An error occurred while installing manticore (0.5.2), and Bundler cannot continue.
Make sure that `gem install manticore -v '0.5.2'` succeeds before bundling.
WARNING: SSLSocket#session= is not supported
Updated logstash-codec-json_lines 2.0.2 to 2.0.3
Updated logstash-codec-multiline 2.0.4 to 2.0.6
Updated logstash-codec-netflow 2.0.2 to 2.0.3
Updated logstash-codec-rubydebug 2.0.4 to 2.0.5
Updated logstash-filter-csv 2.1.0 to 2.1.1
Updated logstash-filter-date 2.0.2 to 2.1.1
Updated logstash-filter-fingerprint 2.0.2 to 2.0.3
Updated logstash-filter-geoip 2.0.4 to 2.0.5
Updated logstash-filter-grok 2.0.2 to 2.0.3
Updated logstash-filter-json 2.0.2 to 2.0.3
Updated logstash-filter-kv 2.0.2 to 2.0.3
Updated logstash-filter-mutate 2.0.2 to 2.0.3
Updated logstash-filter-ruby 2.0.2 to 2.0.3
Updated logstash-filter-useragent 2.0.3 to 2.0.4
Updated logstash-filter-xml 2.0.2 to 2.1.1
Updated logstash-input-elasticsearch 2.0.2 to 2.0.3
Updated logstash-input-file 2.0.3 to 2.1.3
Updated logstash-input-imap 2.0.2 to 2.0.3
Updated logstash-input-jdbc 2.0.5 to 3.0.0
Updated logstash-input-kafka 2.0.2 to 2.0.3
Updated logstash-input-log4j 2.0.4 to 2.0.5
Updated logstash-input-rabbitmq 3.1.1 to 3.1.2
Updated logstash-output-elasticsearch 2.2.0 to 2.3.2
Updated logstash-output-file 2.2.0 to 2.2.3
Updated logstash-output-graphite 2.0.2 to 2.0.3
Updated logstash-output-http 2.0.5 to 2.1.0
Updated logstash-output-rabbitmq 3.0.6 to 3.0.7
Updated logstash-output-s3 2.0.3 to 2.0.4
Updated logstash-output-stdout 2.0.3 to 2.0.4
[root@h102 ~]# gem install jruby-kafka -v '1.5.0'
ERROR:  Could not find a valid gem 'jruby-kafka' (= 1.5.0) in any repository
ERROR:  Possible alternatives: jruby-kafka
[root@h102 ~]# gem install manticore -v '0.5.2'
ERROR:  Could not find a valid gem 'manticore' (= 0.5.2) in any repository
ERROR:  Possible alternatives: MINT-core, antidote, antinode, anvil-core, fat_core
[root@h102 ~]# /opt/logstash/bin/plugin list --verbose 
logstash-codec-collectd (2.0.2)
logstash-codec-dots (2.0.2)
logstash-codec-edn (2.0.2)
logstash-codec-edn_lines (2.0.2)
logstash-codec-es_bulk (2.0.2)
logstash-codec-fluent (2.0.2)
logstash-codec-graphite (2.0.2)
logstash-codec-json (2.0.4)
logstash-codec-json_lines (2.0.3)
logstash-codec-line (2.0.2)
logstash-codec-msgpack (2.0.2)
logstash-codec-multiline (2.0.6)
logstash-codec-netflow (2.0.3)
logstash-codec-oldlogstashjson (2.0.2)
logstash-codec-plain (2.0.2)
logstash-codec-rubydebug (2.0.5)
logstash-filter-anonymize (2.0.2)
logstash-filter-checksum (2.0.2)
logstash-filter-clone (2.0.4)
logstash-filter-csv (2.1.1)
logstash-filter-date (2.1.1)
logstash-filter-dns (2.0.2)
logstash-filter-drop (2.0.2)
logstash-filter-fingerprint (2.0.3)
logstash-filter-geoip (2.0.5)
logstash-filter-grok (2.0.3)
logstash-filter-json (2.0.3)
logstash-filter-kv (2.0.3)
logstash-filter-metrics (3.0.0)
logstash-filter-multiline (2.0.3)
logstash-filter-mutate (2.0.3)
logstash-filter-ruby (2.0.3)
logstash-filter-sleep (2.0.2)
logstash-filter-split (2.0.2)
logstash-filter-syslog_pri (2.0.2)
logstash-filter-throttle (2.0.2)
logstash-filter-urldecode (2.0.2)
logstash-filter-useragent (2.0.4)
logstash-filter-uuid (2.0.3)
logstash-filter-xml (2.1.1)
logstash-input-beats (2.0.3)
logstash-input-couchdb_changes (2.0.2)
logstash-input-elasticsearch (2.0.3)
logstash-input-eventlog (3.0.1)
logstash-input-exec (2.0.4)
logstash-input-file (2.1.3)
logstash-input-ganglia (2.0.4)
logstash-input-gelf (2.0.2)
logstash-input-generator (2.0.2)
logstash-input-graphite (2.0.4)
logstash-input-heartbeat (2.0.2)
logstash-input-http (2.0.2)
logstash-input-imap (2.0.3)
logstash-input-irc (2.0.3)
logstash-input-jdbc (3.0.0)
logstash-input-kafka (2.0.3)
logstash-input-log4j (2.0.5)
logstash-input-lumberjack (2.0.5)
logstash-input-pipe (2.0.2)
logstash-input-rabbitmq (3.1.2)
logstash-input-redis (2.0.2)
logstash-input-s3 (2.0.3)
logstash-input-snmptrap (2.0.2)
logstash-input-sqs (2.0.3)
logstash-input-stdin (2.0.2)
logstash-input-syslog (2.0.2)
logstash-input-tcp (3.0.0)
logstash-input-twitter (2.2.0)
logstash-input-udp (2.0.3)
logstash-input-unix (2.0.4)
logstash-input-xmpp (2.0.3)
logstash-input-zeromq (2.0.2)
logstash-output-cloudwatch (2.0.2)
logstash-output-csv (2.0.2)
logstash-output-elasticsearch (2.3.2)
logstash-output-email (3.0.2)
logstash-output-exec (2.0.2)
logstash-output-file (2.2.3)
logstash-output-ganglia (2.0.2)
logstash-output-gelf (2.0.2)
logstash-output-graphite (2.0.3)
logstash-output-hipchat (3.0.2)
logstash-output-http (2.1.0)
logstash-output-irc (2.0.2)
logstash-output-juggernaut (2.0.2)
logstash-output-kafka (2.0.1)
logstash-output-lumberjack (2.0.4)
logstash-output-nagios (2.0.2)
logstash-output-nagios_nsca (2.0.3)
logstash-output-null (2.0.2)
logstash-output-opentsdb (2.0.2)
logstash-output-pagerduty (2.0.2)
logstash-output-pipe (2.0.2)
logstash-output-rabbitmq (3.0.7)
logstash-output-redis (2.0.2)
logstash-output-s3 (2.0.4)
logstash-output-sns (3.0.2)
logstash-output-sqs (2.0.2)
logstash-output-statsd (2.0.4)
logstash-output-stdout (2.0.4)
logstash-output-tcp (2.0.2)
logstash-output-udp (2.0.2)
logstash-output-xmpp (2.0.2)
logstash-output-zeromq (2.0.2)
logstash-patterns-core (2.0.2)
[root@h102 ~]# 
{% endhighlight %}


其中有几个由于找不到包，所以没有更新成功，但是大部分已经获得了更新


---

### plugin pack


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin pack -h
Usage:
    bin/plugin pack [OPTIONS]

Options:
    --tgz                         compress package as a tar.gz file (default: true)
    --zip                         compress package as a zip file (default: false)
    --[no-]clean                  clean up the generated dump of plugins (default: true)
    --overwrite                   Overwrite a previously generated package file (default: false)
    -h, --help                    print help
[root@h102 ~]# /opt/logstash/bin/plugin pack   --tgz  
Packaging plugins for offline usage
Generated at /opt/logstash/plugins_package.tar.gz
[root@h102 ~]# ll -h /opt/logstash/plugins_package.tar.gz
-rw-r--r-- 1 root root 52M Jan  8 16:26 /opt/logstash/plugins_package.tar.gz
[root@h102 ~]# 
{% endhighlight %}

---

### plugin unpack


{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin unpack -h
Usage:
    bin/plugin unpack [OPTIONS] file

Parameters:
    file                          the package file name

Options:
    --tgz                         unpack a packaged tar.gz file (default: true)
    --zip                         unpack a packaged  zip file (default: false)
    -h, --help                    print help
[root@h102 ~]# /opt/logstash/bin/plugin unpack --tgz /opt/logstash/plugins_package.tar.gz
Unpacking /opt/logstash/plugins_package.tar.gz
Unpacked at /opt/logstash/vendor/cache
The unpacked plugins can now be installed in local-only mode using bin/plugin install --local [plugin name]
[root@h102 ~]# echo $?
0
[root@h102 ~]# 
{% endhighlight %}

**pack/unpack** 主要是用来进行离线管理 **plugins** 

{% highlight bash %}
[root@h102 ~]# /opt/logstash/bin/plugin uninstall logstash-input-twitter
Uninstalling logstash-input-twitter
[root@h102 ~]# /opt/logstash/bin/plugin list twitter
ERROR: No plugins found
[root@h102 ~]# /opt/logstash/bin/plugin install --local logstash-input-twitter
Installing logstash-input-twitter
Installation successful
[root@h102 ~]# /opt/logstash/bin/plugin list twitter
logstash-input-twitter
[root@h102 ~]#
{% endhighlight %}

---

## 其它plugin


其它plugin可以参考下面链接

**[Input Plugins][input_plugins]**

**[Filter Plugins][filter_plugins]**

**[Output Plugins][output_plugins]**

**[Codec Plugins][codec_plugins]**

---

# 命令汇总

* **`/opt/logstash/bin/plugin --help`**
* **`/opt/logstash/bin/plugin list --help`**
* **`/opt/logstash/bin/plugin list`**
* **`/opt/logstash/bin/plugin list logstash-output-kafka`**
* **`/opt/logstash/bin/plugin list kafka`**
* **`/opt/logstash/bin/plugin list --verbose kafka`**
* **`/opt/logstash/bin/plugin list --group codec`**
* **`/opt/logstash/bin/plugin uninstall -h`**
* **`/opt/logstash/bin/plugin uninstall logstash-output-kafka`**
* **`/opt/logstash/bin/plugin install logstash-output-kafka`**
* **`gem source -l`**
* **`grep source /opt/logstash/Gemfile`**
* **`/opt/logstash/bin/plugin update -h`**
* **`/opt/logstash/bin/plugin update logstash-codec-edn_lines`**
* **`/opt/logstash/bin/plugin update`**
* **`/opt/logstash/bin/plugin list --verbose`**
* **`/opt/logstash/bin/plugin pack -h`**
* **`/opt/logstash/bin/plugin pack   --tgz`**
* **`ll -h /opt/logstash/plugins_package.tar.gz`**
* **`/opt/logstash/bin/plugin unpack -h`**
* **`/opt/logstash/bin/plugin unpack --tgz /opt/logstash/plugins_package.tar.gz`**
* **`/opt/logstash/bin/plugin uninstall logstash-input-twitter`**
* **`/opt/logstash/bin/plugin install --local logstash-input-twitter`**
* **`/opt/logstash/bin/plugin list twitter`**


---

[logstash]:https://www.elastic.co/products/logstash
[logstash_plugins]:https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html
[private_rubygem]:https://www.elastic.co/guide/en/logstash/current/private-rubygem.html
[input_plugins]:https://www.elastic.co/guide/en/logstash/current/input-plugins.html
[filter_plugins]:https://www.elastic.co/guide/en/logstash/current/filter-plugins.html
[output_plugins]:https://www.elastic.co/guide/en/logstash/current/output-plugins.html
[codec_plugins]:https://www.elastic.co/guide/en/logstash/current/codec-plugins.html


