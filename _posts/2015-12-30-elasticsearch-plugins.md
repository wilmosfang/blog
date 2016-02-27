---
layout: post
title:  Elasticsearch Plugins
author: wilmosfang
categories: linux elasticsearch 
wc: 273  588 7336 
excerpt:  elasticsearch 插件的相关基础
comments: true
---



# 前言

**[Elasticsearch Plugins][plugins]** 是一种灵活的可自定义扩展 **Elasticsearch** 特性的机制

>Plugins are a way to enhance the core Elasticsearch functionality in a custom manner. They range from adding custom mapping types, custom analyzers, native scripts, custom discovery and more.

大体可分为三类：**Java plugins、Site plugins、Mixed plugins**

* **Java plugins** 这一类只包含 **JAR** 文件 ，并且集群中每一个节点都得安装，安装后都得重启才能生效
* **Site plugins** 这一类包含一些 **Javascript, HTML, 和 CSS** 文件，可以直接在ES里使用，只要在一个节点上进行安装，也不必重启就有能生效，使用这种方式进行访问

>http://yournode:9200/_plugin/[plugin name]

* **Mixed plugins** 结合以上的两种特性

下面对 **[Elasticsearch Plugins][plugins]** 的安装与配置方法进行分享，详细可以参看 **ES** 的 **[官方文档][plugins]**


> **Tip:** 当前的版本为 **Elasticsearch 2.1.1** 

---



# 概要

* TOC
{:toc}


---

## Plugin管理


{% highlight bash %}
[root@h102 elasticsearch]# rpm -qa | grep elast
elasticsearch-2.1.1-1.noarch
[root@h102 elasticsearch]# rpm -ql elasticsearch-2.1.1-1.noarch | grep plugin
/usr/share/elasticsearch/bin/plugin
/usr/share/elasticsearch/plugins
[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin -h

NAME

    plugin - Manages plugins

SYNOPSIS

    plugin <command>

DESCRIPTION

    Manage plugins

COMMANDS

    install    Install a plugin

    remove     Remove a plugin

    list       List installed plugins

NOTES

    [*] For usage help on specific commands please type "plugin <command> -h"


[root@h102 elasticsearch]# 
{% endhighlight %}


相对简单，只有三个方法 **install，remove，list**

子命令的帮助查看方法 

{% highlight bash %}
[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin list -h

NAME

    list - List all plugins

SYNOPSIS

    plugin list

DESCRIPTION

    This command lists all installed elasticsearch plugins


[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin remove -h

NAME

    remove - Remove a plugin

SYNOPSIS

    plugin remove <name>

DESCRIPTION

    This command removes an elasticsearch plugin


[root@h102 elasticsearch]#
{% endhighlight %}

当前系统中并没有检测到有插件

{% highlight bash %}
[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin list 
Installed plugins in /usr/share/elasticsearch/plugins:
    - No plugin detected
[root@h102 elasticsearch]# 
{% endhighlight %}


---

## plugin的目录

plugin默认地址为安装目录的 **./plugins** 

{% highlight bash %}
[root@h102 elasticsearch]# rpm -ql elasticsearch-2.1.1-1.noarch | grep plugin
/usr/share/elasticsearch/bin/plugin
/usr/share/elasticsearch/plugins
[root@h102 elasticsearch]# ll  /usr/share/elasticsearch/plugins
total 0
[root@h102 elasticsearch]# 
{% endhighlight %}

> **Tip:** 由于没有插件，所以是空的


但是可以在 **elasticsearch.yml** 中进行修改

{% highlight bash %}
path.plugins: /path/to/custom/plugins/dir
{% endhighlight %}

可以在 **elasticsearch.yml** 中设定强制要求的plugins

{% highlight bash %}
plugin.mandatory: mapper-attachments,lang-python
{% endhighlight %}

设定之后，如果缺少指定的plugin，ES是不会启动的


---

## 安装plugin

详细的安装方法可以使用下面命令进行查看，也可以参考 **[plugin安装方法][installation]**

{% highlight bash %}
[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin install -h
{% endhighlight %}

### 安装head插件

{% highlight bash %}
[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head 
-> Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip ...
Downloading ..................................................................................................................................................................................................................................................................................DONE
Verifying https://github.com/mobz/elasticsearch-head/archive/master.zip checksums if available ...
NOTE: Unable to verify checksum for downloaded plugin (unable to find .sha1 or .md5 file to verify)
Installed head into /usr/share/elasticsearch/plugins/head
[root@h102 elasticsearch]# echo $?
0
[root@h102 elasticsearch]# /usr/share/elasticsearch/bin/plugin list
Installed plugins in /usr/share/elasticsearch/plugins:
    - head
[root@h102 elasticsearch]# ll  /usr/share/elasticsearch/plugins
total 4
drwxr-xr-x 5 root root 4096 Dec 30 22:04 head
[root@h102 elasticsearch]#
{% endhighlight %}

于是使用网页浏览 **`http://localhost:9200/_plugin/head/`** 就可以看到下列界面


![es_plugin_head.png](/images/es_plugin/es_plugin_head.png)

通过点击不同选项卡，和不同分片，可以看到相关信息


目前只能使用localhost进行访问，下面打开外部的访问

---

## 外部访问

修改配置使其可以从外部访问

**network.host** 可以绑定IP到主机IP，也可以对所有 **0.0.0.0** 打开


{% highlight bash %}
[root@h102 elasticsearch]# vim elasticsearch.yml
[root@h102 elasticsearch]# grep host elasticsearch.yml | grep -v "^#"
network.host: 0.0.0.0 
[root@h102 elasticsearch]# /etc/init.d/elasticsearch  restart 
Stopping elasticsearch:                                    [  OK  ]
Starting elasticsearch:                                    [  OK  ]
[root@h102 elasticsearch]# 
{% endhighlight %}


打开防火墙

{% highlight bash %}
[root@h102 ~]# grep 9200 /etc/sysconfig/iptables 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9200 -j ACCEPT 
[root@h102 ~]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h102 ~]# iptables -L -nv | grep 9200
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:9200 
[root@h102 ~]# 
{% endhighlight %}

现在可以从外面进行访问了

![es_plugin_head2.png](/images/es_plugin/es_plugin_head2.png)




---


# 命令汇总


* **`rpm -ql elasticsearch-2.1.1-1.noarch | grep plugin`**
* **`/usr/share/elasticsearch/bin/plugin -h`**
* **`/usr/share/elasticsearch/bin/plugin list -h`**
* **`/usr/share/elasticsearch/bin/plugin remove -h`**
* **`/usr/share/elasticsearch/bin/plugin list`**
* **`ll  /usr/share/elasticsearch/plugins`**
* **`/usr/share/elasticsearch/bin/plugin install -h`**
* **`/usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head`**
* **`echo $?`**
* **`/usr/share/elasticsearch/bin/plugin list`**
* **`ll  /usr/share/elasticsearch/plugins`**
* **`vim elasticsearch.yml`**
* **`grep host elasticsearch.yml | grep -v "^#"`**
* **`/etc/init.d/elasticsearch  restart`**
* **`grep 9200 /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv | grep 9200`**




[plugins]:https://www.elastic.co/guide/en/elasticsearch/plugins/current/intro.html
[installation]:https://www.elastic.co/guide/en/elasticsearch/plugins/current/installation.html
