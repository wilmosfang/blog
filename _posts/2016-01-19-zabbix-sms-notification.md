---
layout: post
title:  Zabbix 短信报警配置
author: wilmosfang
categories:  linux monitoring zabbix
wc: 409 778 11512 
excerpt: 涉及短信API，短信发送脚本，Zabbix Action配置，触发测试   
comments: true
---



# 前言

**[Zabbix][zabbix]** 是一个高效的开源监控解决方案

邮件提醒的方式比较适合办公环境，电脑不在身边时，报警容易被忽视(大家习惯刷微博，刷微信，但不太习惯刷手机邮箱)，这种情况下短信报警对于重要紧急的内容是一种很好的提醒方式

下面分享一下 **[Zabbix][zabbix]** 监控系统短信报警的设定方法，详细可以参阅 **[官方文档][zabbix_doc]**

> **Tip:** 当前的最新版本为 **Zabbix 2.4.7**

---


# 概要

* TOC
{:toc}


---

## 前期准备

首先得有一个发短信的云平台

短信云平台的选择可以参考各类营销短信提供商，为什么选择营销短信提供商，而不是其它的，有以下几点原因

* 1.短信猫要使用电话卡，短信费用不便宜，还要购买和维护相应的硬件设备，性价比不高
* 2.验证短信云平台有模板审查机制，定制的报警模板不能马上生效，要等审查通过，比较局限
* 3.营销短信云平台最为灵活，可以随便自定义，余额管理也比较方便

市面上主要就是这三种方式，相较而言第三种最方便

当然这并非绝对，只是我的一家之言，具体还得看应用场景，比如对安全性有额外要求或局域网络没有外网访问能力的，就可能要调整相应取向的权重

总而言之，我最后选择的是使用营销短信云平台，因为最省事儿

---

### 发送短信API

选择好短信云平台后，就要使用云平台提供的API开发出一个发送短信的工具(脚本)

不同商家提供的API不一样，不能一概而论，所以这里得有一定功底看懂API文档或Demo，然后进行改造使用或干脆自已写一个


下面以正奥通信提供的API为例进行演示

(此刻为 2016.01.18 21:11，之后API可能会有改动，但方法不变)


下面为[API相关文档](http://139.129.128.71/api.html)



<table width="100%" border="1" >

<tr height="40">
          <td ><div align="center"><font size="3">接口文档</font></div></td>
          <td ><div align="center"><font size="4"><a href="http://139.129.128.71/api_doc.rar">正奥通信短信接口规范V1.0.doc</a></font></div></td>
        </tr>
        <tr height="40">
          <td ><div align="center"><font size="3">客户端页面地址</font></div></td>
          <td ><div align="center"><font size="4">http://139.129.128.71:8080/hsms</font></div></td>
        </tr>
	<tr height="40">
          <td ><div align="center"><font size="3">短信发送地址</font></div></td>
          <td ><div align="center"><font size="4">http://139.129.128.71:8086/msgHttp/json/mt</font></div></td>
        </tr>  
       <tr height="40">
          <td ><div align="center"><font size="3">余额查询地址</font></div></td>
          <td ><div align="center"><font size="4">http://139.129.128.71:8086/msgHttp/json/balance</font></div></td>
        </tr>  
        <tr height="40">
          <td ><div align="center"><font size="3">PHP Demo</font></div></td>
          <td ><div align="center"><font size="4"><a href="http://139.129.128.71/PHP_demo.rar">二次开发PHP接口</a></font></div></td>
        </tr>    
       <tr height="40">
          <td ><div align="center"><font size="3">JAVA Demo</font></div></td>
          <td ><div align="center"><font size="4"><a href="http://139.129.128.71/JAVA_demo.rar">二次开发JAVA接口</a></font></div></td>
        </tr>  
       <tr height="40">
          <td ><div align="center"><font size="3">C# Demo</font></div></td>
          <td ><div align="center"><font size="4"><a href="http://139.129.128.71/SmsDemo.rar">二次开发C#接口</a></font></div></td>
        </tr>
</table>



根据API文档和参考Demo我写了一个简单的bash实现

---

### 短信余额脚本

这个是获取短信余额的脚本

~~~
[root@redis-b sms_script]# cat sms_get_balance.bash 
#!/bin/bash

## config area
QTOOLS=/tmp/sms_script/qtools
CURL=/usr/bin/curl
account='xxxxxxx'
password='xxxxxxx'

## auto config
timestamps=`date +%s`
timestamps=$timestamps'000'
url='http://139.129.128.71:8086/msgHttp/json/balance'

## generate args for curl
url_account=`$QTOOLS urlencode $account`
url_pass_temp=`echo -n $password$timestamps|md5sum | awk '{print $1}' `
url_pass=`$QTOOLS urlencode $url_pass_temp`
url_time=`$QTOOLS urlencode $timestamps`

#$CURL -X POST  "$url?account=$url_account&password=$url_pass&timestamps=$url_time"
$CURL -X POST  "$url" -d "account=$url_account&password=$url_pass&timestamps=$url_time"
[root@redis-b sms_script]# ./sms_get_balance.bash 
{
	"Rspcode":0,
	"Count":972
}
[root@redis-b sms_script]# 
~~~

---

### 发送短信脚本

这个是发送短信的脚本

~~~
[root@redis-b sms_script]# cat sms_sent_message.bash 
#!/bin/bash

##
#$1 is phone number list 
#$2 is messages with utf-8
#

help_info()
{
cat <<EOF
usage:$0 <'phone_numbers'>  <'message'> 
 
Example:$0  '18016017077,18016017078'   '你好'

output:
	will get the jason response something like

{
  "Rets": [
    {
      "Rspcode": 0,
      "Msg_Id": " 114445276129660989",
      "Mobile": "18016017077"
    },
    {
      "Rspcode": 0,
      "Msg_Id": " 114445276129660991",
      "Mobile": "18016017078"
    }
  ]
}


More: <'phone_numbers'>  <'message'>  must be specified and only two args
	it need two args and only two args 

EOF
}

## simple check for input args

if [ "$#" -ne "2" ]
then
        help_info
        exit 1
fi 

##
## need to be specified in CLI

mobile="$1"
content="$2"

## config area
## need to be configed manually
QTOOLS=/tmp/sms_script/qtools
CURL=/usr/bin/curl
account='xxxxxxx'
password='xxxxxxx'

## auto config
timestamps=`date +%s`
timestamps=$timestamps'000'
url='http://139.129.128.71:8086/msgHttp/json/mt'

## generate args for curl
url_account=`$QTOOLS urlencode "$account"`
url_pass_temp=`echo -n "$password$mobile$timestamps"|md5sum | awk '{print $1}' `
url_pass=`$QTOOLS urlencode "$url_pass_temp"`
url_time=`$QTOOLS urlencode "$timestamps"`
url_mobile=`$QTOOLS urlencode "$mobile"`
url_content=`$QTOOLS urlencode "$content"`

$CURL -X POST  "$url" -d "account=$url_account&password=$url_pass&mobile=$url_mobile&content=$url_content&timestamps=$url_time"

[root@redis-b sms_script]# ./sms_sent_message.bash 
usage:./sms_sent_message.bash <'phone_numbers'>  <'message'> 
 
Example:./sms_sent_message.bash  '18016017077,18016017078'   '你好'

output:
	will get the jason response something like

{
  "Rets": [
    {
      "Rspcode": 0,
      "Msg_Id": " 114445276129660989",
      "Mobile": "18016017077"
    },
    {
      "Rspcode": 0,
      "Msg_Id": " 114445276129660991",
      "Mobile": "18016017078"
    }
  ]
}


More: <'phone_numbers'>  <'message'>  must be specified and only two args
	it need two args and only two args 

[root@redis-b sms_script]# 
[root@redis-b sms_script]# ./sms_sent_message.bash  '15016017077' '高级报警测试
> abcdefghijklmnopqrstuvwxyz
> hello!
> over'
{
	"Rets":[
	{
		"Rspcode":0,
		"Msg_Id":"214531233876240942",
		"Mobile":"15016017077",
		"Fee":1
	}
		]
}
[root@redis-b sms_script]# 
~~~

几秒钟后手机会收到一条短信

![sms1.png](/images/zabbix_sms/sms1.png)

> **Tip:** 这里有一个命令 **qtools** 不必太计较是怎么来的，只用知道它是用来进行urlencode转换的就可以了
> 
> 也可以使用shell来代替，比如

~~~
echo '报警' | tr -d '\n' | xxd -plain | sed 's/\(..\)/%\1/g'  
echo '报警' |tr -d '\n' |od -An -tx1|tr ' ' %
~~~

只是上面的脚本在处理带有换行的内容时会产生问题，最后都会变成一行，格式就很难看


---

## 配置Zabbix Actions

进入zabbix的Actions创建界面

**[Configuration]->[Actions]->[Create action]**

![sms2.png](/images/zabbix_sms/sms2.png)


**[Action]** 选项卡里进行相关配置，如果不发邮件的话 **Default subject** 和 **Default message** 的内容并不起作用

![sms3.png](/images/zabbix_sms/sms3.png)

**[Conditions]** 里加入一个判断条件，就是 **Trigger severity = Disater** 

这个级别可以根据具体应用场景自定义，也可以加入其它条件用来进行更精确的定位，但方法都一样

![sms4.png](/images/zabbix_sms/sms4.png)

**[Operations]** 里配置Action的具体内容

我设定的是：

* 立即执行
* 调用远程命令的方式
* 目标为本机
* 自定义脚本
* 使用Zabbix server执行
* 命令内容

~~~
/tmp/sms_script/sms_sent_message.bash '1801601xxxx'  'zabbix测试系统报警:{TRIGGER.STATUS}:{HOST.NAME1}:{TRIGGER.NAME}: {ITEM.NAME1} ({HOST.NAME1}:{ITEM.KEY1}): {ITEM.VALUE1}:{EVENT.DATE} {EVENT.TIME}'
~~~

> **Tip:** 可以使用Zabbix提供的宏组合出自已想要的信息，相关的宏信息可以参考 **[Zabbix Macros][macros]**


![sms5.png](/images/zabbix_sms/sms5.png)

添加保存后，**[Actions]** 里就会多出一条记录来

![sms6.png](/images/zabbix_sms/sms6.png)





---

## 触发测试


创建一个 **Disaster** 级别的触发器

参考 **Template ICMP Ping** 创建一个trigger，然后修改一下 **Severity** 的级别

> **Tip:** 简便方法是 **Link** 一下模板，然后 **Unlik** 一下 ，不要选择 **Unlink and clear** 否则又没了

![sms7.png](/images/zabbix_sms/sms7.png)

通过防火墙丢弃icmp包触发报警


~~~
[root@h101 ~]# iptables -L -nv | grep icmp
    7   672 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
   87  9577 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
[root@h101 ~]# grep icmp /etc/sysconfig/iptables
-A INPUT -p icmp -j ACCEPT 
#-A INPUT -p icmp -j DROP 
-A INPUT -j REJECT --reject-with icmp-host-prohibited 
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
[root@h101 ~]# vim /etc/sysconfig/iptables
[root@h101 ~]# grep icmp /etc/sysconfig/iptables
#-A INPUT -p icmp -j ACCEPT 
-A INPUT -p icmp -j DROP 
-A INPUT -j REJECT --reject-with icmp-host-prohibited 
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
[root@h101 ~]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h101 ~]# iptables -L -nv | grep icmp
    0     0 DROP       icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
[root@h101 ~]# 
~~~

过一小会儿(根据检查频度和触发规则而定)，报警被触发了


![sms8.png](/images/zabbix_sms/sms8.png)

几乎同时也收到了短信

![sms9.png](/images/zabbix_sms/sms9.png)


**Events** 里也留下了记录

查看方法是 : **[Monitoring]->[Events]**

![sms10.png](/images/zabbix_sms/sms10.png)



---


# 命令汇总

* **`cat sms_get_balance.bash`**
* **`./sms_get_balance.bash`**
* **`cat sms_sent_message.bash`**
* **`./sms_sent_message.bash`**
* **`./sms_sent_message.bash  '15016017077' '高级报警测试`**
* **`vim /etc/sysconfig/iptables`**
* **`grep icmp /etc/sysconfig/iptables`**
* **`/etc/init.d/iptables reload`**
* **`iptables -L -nv | grep icmp`**


---

[zabbix]:http://www.zabbix.com/
[zabbix_doc]:http://www.zabbix.com/documentation.php
[macros]:https://www.zabbix.com/documentation/2.4/manual/appendix/macros/supported_by_location

