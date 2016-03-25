---
layout: post
title:  UnrealIRCd 基础
categories:  linux irc unrealircd
wc: 2022 11273 97455 
excerpt: unrealircd 的下载，检验，安装，依赖，基础配置，启动运行，访问，和整个过程中的注意事项以及报错处理 
comments: true
---

# 前言

**[UnrealIRCd][unrealircd]** 是一款开源的 IRC 服务软件

IRC是 Internet Relay Chat 的英文缩写，即互联网中继聊天

IRC的最大特性是实现了在线实时交谈，类似于QQ

那为什么不直接使用QQ(或其它任何社交类及时通讯软件)呢，当公司内部的敏感信息是通过QQ交流时，你认为有多少安全性呢，我们只能寄期望于提供服务的是一家有原则，有节操有底线的公司，包括所有与通讯信息系统相关的技术人员都是没有窥探欲的 “正人君子”，然而这个大数据时代，有些来自商业基因的原始冲动是很难把持得住的，所以IRC(或基于IRC进行二次开发的聊天工具)自然成了很多技术驱动型公司的内部及时通讯系统

相对于电子邮件或新闻组等沟通方式，IRC速度更快、功能更多，交互性更强，所以也自然成为了很多技术社群的首选，因为它足够强健的安全保护，也成了很多黑客的交流工具(总之就是各种好呀！)

话说回来，有没有不好呢，当然有，得有相应的人力物力资源来维护和支撑，在老板看来这就是成本

这里简单分享一下 **[UnrealIRCd][unrealircd]** 相关的基础，详细内容可以参考 **[官方文档][unrealircd_doc]** 


> **Tip:** 当前的最新版本为 **UnrealIRCd 4.0.2** 发布于 March 11, 2016, 17:25 CET

---


# 概要

* TOC
{:toc}



---

## 下载

撰文此刻最新版的 **[下载地址][unrealircd_dl]** 

{% highlight bash %}
[root@h104 irc]# wget https://www.unrealircd.org/unrealircd4/unrealircd-4.0.2.tar.gz
--2016-03-21 20:45:12--  https://www.unrealircd.org/unrealircd4/unrealircd-4.0.2.tar.gz
Resolving www.unrealircd.org (www.unrealircd.org)... 54.76.160.181
Connecting to www.unrealircd.org (www.unrealircd.org)|54.76.160.181|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4728491 (4.5M) [application/x-gzip]
Saving to: ‘unrealircd-4.0.2.tar.gz’

100%[============================================================================================>] 4,728,491   8.00KB/s   in 5m 56s 

2016-03-21 20:51:10 (13.0 KB/s) - ‘unrealircd-4.0.2.tar.gz’ saved [4728491/4728491]

[root@h104 irc]# ls
unrealircd-4.0.2.tar.gz
[root@h104 irc]# du -sh unrealircd-4.0.2.tar.gz 
4.6M	unrealircd-4.0.2.tar.gz
[root@h104 irc]# ll unrealircd-4.0.2.tar.gz 
-rw-r--r-- 1 root root 4728491 Mar 12 00:00 unrealircd-4.0.2.tar.gz
[root@h104 irc]# 
{% endhighlight %}

---

## 检验

下载完源码包后，强烈建议对包进行一次校验，这是一种最简单高效确认介质可信的办法

这一步并不是必须的，但是体检一下，可以消除很大的安全隐患

{% highlight bash %}
[root@h104 irc]# gpg --keyserver keys.gnupg.net --recv-keys 0xA7A21B0A108FF4A9
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: requesting key 108FF4A9 from hkp server keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 108FF4A9: public key "UnrealIRCd releases (for verification of software downloads only!) <releases@unrealircd.org>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[root@h104 irc]# echo $?
0
[root@h104 irc]#
[root@h104 irc]# wget https://www.unrealircd.org/unrealircd4/unrealircd-4.0.2.tar.gz.asc
--2016-03-21 21:34:44--  https://www.unrealircd.org/unrealircd4/unrealircd-4.0.2.tar.gz.asc
Resolving www.unrealircd.org (www.unrealircd.org)... 54.76.160.181
Connecting to www.unrealircd.org (www.unrealircd.org)|54.76.160.181|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 836 [text/plain]
Saving to: ‘unrealircd-4.0.2.tar.gz.asc’

100%[============================================================================================>] 836         --.-K/s   in 0s      

2016-03-21 21:34:47 (16.7 MB/s) - ‘unrealircd-4.0.2.tar.gz.asc’ saved [836/836]

[root@h104 irc]# cat unrealircd-4.0.2.tar.gz.asc
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2

iQIcBAABCAAGBQJW4uXrAAoJEKeiGwoQj/Sp56cQAJCuU+IMQPMOL+ALlEasjL8w
Dmn0QcIFjB/qFJgGZaMWlb1iAK7QvTtwEf36qgJiqN+VyES2c93EAefmlTI3rDnn
2s/7va3JYLMUDQQPvGyiWzJ6074T+6veK9enj4/izqvCq7fbNx1K/8B1l6Dxy3l+
WHPyrna3NTRTcIYiz7et01XpEAESYuX9w+VQ4RG8kLYQIophUkDCkxgLEISC3Lnj
lt/5j9S0q35j15F2uw16bUMfjLsHhBEuyYCqVt4WJmvsDKvU2gdkNvJigRJ5JEPQ
6dwWY/PAMX5mUpDk//ikZrr78nsclE9fScZq9np0jORUyfIDD9LNhvTVufvU6rZM
2AYM1w0s7pW321KKSKkEswJReBySBScOVWzPrhMNCFwxWVzJzk8o3H6pNP9fdvDV
s2eVIxR1RKP6zatRlvFgxqrQbsuFlF/byng/JOx0tcB/PlR6Fis6JWokUDH8eQTb
3hhL2y0PuO7R+OapeV1h4sjaf8BqYKweuQqtKquNtT61xh3AdcP74gS3yzaw9XuT
Lsw4c2d9pdUFTdvp4LzqJiez6CAPu0ZKOPNFDyoeU3jz7KZTVuHYQR5Sw0r8rHy5
VUzOEe24irfqFRNcA/WlyawkxulMvicUXW5KiCwQlGi1ettZJtskZJUWkQCose8t
iTaXIDooi9ChB03ks0F+
=jWpz
-----END PGP SIGNATURE-----
[root@h104 irc]# 
[root@h104 irc]# gpg --verify unrealircd-4.0.2.tar.gz.asc unrealircd-4.0.2.tar.gz
gpg: Signature made Fri 11 Mar 2016 11:36:11 PM CST using RSA key ID 108FF4A9
gpg: Good signature from "UnrealIRCd releases (for verification of software downloads only!) <releases@unrealircd.org>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 1D2D 2B03 A0B6 8ED1 1D68  A24B A7A2 1B0A 108F F4A9
[root@h104 irc]# 
{% endhighlight %}

这里有一段：

{% highlight bash %}
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 1D2D 2B03 A0B6 8ED1 1D68  A24B A7A2 1B0A 108F F4A9
{% endhighlight %}

说明unrealircd给我们的公钥是没有经过权威认证的自签名证书，没有任何信息可以确保这的确就是来自unrealircd的

那为什么要使用自签名证书呢，因为生成快，还不要钱，测试时我也经常干这事儿，只要我自已信就可以了

还有一种方式就是使用hash结果来判断有否被篡改

{% highlight bash %}
[root@h104 irc]# sha1sum unrealircd-4.0.2.tar.gz
57d59e8ee436dd4d9894789095a9d94358dfd5c6  unrealircd-4.0.2.tar.gz
[root@h104 irc]# sha256sum unrealircd-4.0.2.tar.gz
6af88866c0926fef874b969a09cae547be898c6b528d906fcf1834659531927c  unrealircd-4.0.2.tar.gz
[root@h104 irc]# 
{% endhighlight %}

通过和官方提供的hash结果进行比较来判断是否一致

在http环境下，也有可能连同页面提供的hash结果一同劫持篡改，相对而言https页面里的hash结果更可信一些，更安全的是经ca认证的https

---

## 解压

{% highlight bash %}
[root@h104 irc]# tar -axvf unrealircd-4.0.2.tar.gz
unrealircd-4.0.2/
unrealircd-4.0.2/README.md
unrealircd-4.0.2/Makefile.in
unrealircd-4.0.2/unrealircd.in
unrealircd-4.0.2/autoconf/
...
...
unrealircd-4.0.2/LICENSE
unrealircd-4.0.2/.gitignore
unrealircd-4.0.2/makefile.win32
unrealircd-4.0.2/configure.ac
unrealircd-4.0.2/autogen.sh
unrealircd-4.0.2/.CHANGES.NEW
[root@h104 irc]# 
[root@h104 irc]# ls
unrealircd-4.0.2  unrealircd-4.0.2.tar.gz  unrealircd-4.0.2.tar.gz.asc
[root@h104 irc]# 
{% endhighlight %}

---

## 安装

整个过程使用如下命令

{% highlight bash %}
./Config
make
make install
{% endhighlight %}

详细安装过程


> **Note:** 用的是 **`./Config`** 而不是 **`./configure`** , **`./Config`** 是个shell脚本，就是一层打包，用来收集配置信息，然后调用 **`./configure`**


### 注意

> **Note:** 这里注意，最好不要使用root的身份安装，建议创建一个irc用户，然后以它的身份安装和运行；如果使用root安装，填默认安装路径时最好不要接受 **`/root/unrealircd`** ,会带来很多隐患和问题，最好填一个比较常规的地址，类似于 **`/usr/local/unrealircd/`** ，这样即便后期想转成其它用户身份来管理也相对容易（一旦编译写死，除非重编译，即便把目录拷过去也会找编译时指定的地址，那时就有权限问题了），但是测试环境我会使用root并接受所有默认来演示功能

一切都接受默认

{% highlight bash %}
[root@h104 irc]# ls
unrealircd-4.0.2  unrealircd-4.0.2.tar.gz  unrealircd-4.0.2.tar.gz.asc
[root@h104 irc]# cd unrealircd-4.0.2
[root@h104 unrealircd-4.0.2]# ls
autoconf    Config     configure.ac  extras   LICENSE      makefile.win32  src
autogen.sh  configure  doc           include  Makefile.in  README.md       unrealircd.in
[root@h104 unrealircd-4.0.2]# ./Config 

 _   _                      _ ___________  _____     _
| | | |                    | |_   _| ___ \/  __ \   | |
| | | |_ __  _ __ ___  __ _| | | | | |_/ /| /  \/ __| |
| | | | '_ \| '__/ _ \/  _ | | | | |    / | |    /  _ |
| |_| | | | | | |  __/ (_| | |_| |_| |\ \ | \__/\ (_| |
 \___/|_| |_|_|  \___|\__,_|_|\___/\_| \_| \____/\__,_|

                               Configuration Program
                                for UnrealIRCd 4.0.2
                                    
This program will help you to compile your IRC server, and ask you
questions regarding the compile-time settings of it during the process. 
regarding the setup of it, during the process.

A short installation guide is available online at:
https://www.unrealircd.org/docs/Installing_from_source

Full documentation is available at:
https://www.unrealircd.org/docs/UnrealIRCd_4_documentation
[Enter to continue]
UnrealIRCd 4.0.2 Release Notes
===============================

UnrealIRCd 4 is here!

We have been working hard over the past few years to replace the successful
3.2.x series with a more modern code base. At the same time we have been
incorporating requests from our bug tracker, ideas from ourselves and
many suggestions that came up during the UnrealIRCd survey from Q4 2013.
After 4 alpha versions, 4 beta's and 6 release candidates we are proud to
finally present you the first stable release of UnrealIRCd 4.

UnrealIRCd is far more modular and configurable than before. For a brief
overview of what's new in UnrealIRCd 4 have a look at:
https://www.unrealircd.org/docs/What's_new_in_UnrealIRCd_4

==[ DOCUMENTATION ]==
All documentation has been moved to our wiki:
* Documentation: https://www.unrealircd.org/docs/
* FAQ: https://www.unrealircd.org/docs/FAQ

Be sure not to use any other (older) documentation as it isn't fully
compatible with UnrealIRCd 4. In particular, do NOT use unreal32docs*html.

==[ UPGRADING FROM 3.2.x ]==
If you are upgrading from 3.2.x then there are three important things to know:

1) NEW FILE LOCATIONS
In UnrealIRCd 4 the location of the configuration files and other files have
been changed. On *NIX the directory where you compile the IRCd from
(previously 'Unreal3.2.X', now 'unrealircd-4.0.X') is no longer the same as
the directory where the IRCd will be running from.
By default the IRCd is installed to /home/yourusername/unrealircd on *NIX
On Windows UnrealIRCd will install to C:\Program Files (x86\UnrealIRCd 4

The new directory structure is as follows (both on Windows and *NIX):
conf/      contains all configuration files
logs/      for log files
modules/   all modules (.so files on *NIX, .dll files on Windows)

2) CONFIGURATION FILE CHANGES
There have also been changes in various configuration blocks and settings.
Don't worry, UnrealIRCd can convert your existing 3.2.x configuration files
to UnrealIRCd 4 format. There's no need to start from scratch.

Please read https://www.unrealircd.org/docs/Upgrading_from_3.2.x !!

3) THIRD PARTY MODULES
If you are using 3rd party modules then they will need an update to run on
UnrealIRCd 4. Due to the many core changes in UnrealIRCd 4 it was simply
impossible to make 3.2.x modules work out-of-the-box on 4.x.
Contact your developer for a new version or ask on our Modules forum where
someone may be kind enough to convert the module for you if you ask nicely:
https://forums.unrealircd.org/viewforum.php?f=52

==[ RUNNING A MIXED 3.2.X / 4.X NETWORK ]==
You can run a mixed 3.2.x <-> 4.x network if you a follow a few simple rules:
https://www.unrealircd.org/docs/Running_a_mixed_UnrealIRCd_3.2_and_UnrealIRCd_4_network

==[ END OF THE 3.2.X SERIES ]==
With the release of UnrealIRCd 4 we are deprecating the previous series.
All support for the 3.2.x series will stop after December 31, 2016.
See https://www.unrealircd.org/docs/UnrealIRCd_3.2.x_deprecated

==[ SUPPORT ]==
Before you seek support, please check our Frequently Asked Questions:
https://www.unrealircd.org/docs/FAQ

For support you have two choices:
* Forums: https://forums.unrealircd.org/
* IRC: irc.unrealircd.org / #unreal-support

==[ CHANGES BETWEEN 4.0.1 AND 4.0.2 ]==
The 4.0.2 release comes with the following new features:
* Ability to hide quit messages from *LINEd users (set::hide-ban-reason)
* Blacklist hits are now sent to new snomask +b rather than all ircops

Major issues fixed:
* None

Minor issues fixed:
* prefix-quit was not working
* FreeBSD: fix kevent bug flood in error log
* Incorrect server description in /LINKS
* Logging to syslog was broken
* OS X: Update ./Config to use Homebrew OpenSSL by default
* Don't show UID to client in case of a SVSMODE

==[ CHANGES BETWEEN 4.0.0 AND 4.0.1 ]==
The 4.0.1 release comes with the following minor improvements:
* The blacklist module now supports %ip (=banned IP) in blacklist::reason.
* *NIX: You can use cron again, see https://www.unrealircd.org/docs/Cron_job
* /MODULE now lists only 3rd party modules by default so you don't get flooded.
* *NIX: Added './unrealircd reloadtls' to reload TLS certificate and keys.

Major issue fixed:
* Crash if you removed a listen { } block with active clients on that port
* MODEs set by a server (not by a user) were not always propagated
  correctly accross the network. In practice this only affected /SAMODE
  and possibly some services that don't send MODEs from ChanServ/BotServ.

Minor issues fixed:
* When doing /LIST under mIRC it would hide empty +P channels.
* Servers wouldn't link if link::outgoing::hostname was a CNAME.
* SSL Certificate fingerprint not communicated properly to servers/services.
* *NIX: ./unrealircd [stop|rehash] failed if not installed to ~/unrealircd.
* Windows: IRCd could crash after showing the config error screen on startup.

==[ CHANGES BETWEEN 3.2.X AND 4.X ]==
Below is a summary of the changes between UnrealIRCd 3.2.x and UnrealIRCd 4.
For a complete list of all 1100+ changes you can use 'git log' or have a
look at: https://github.com/unrealircd/unrealircd/commits/unreal40

==[ NEW ]==
* We moved a lot of functionality, including most channel modes, user
  modes and all extended bans into 138 separate modules.
  This makes it...
  A) possible to fully customize what exact functionality you want to load.
     You could even strip down UnrealIRCd to get something close to the
     basic RFC1459 features from the 1990s. (No idea why you would want
     that, but it's possible)
  B) easier for coders to see all source code related to a specific feature
  C) possible to fix bugs and just reload rather than restart the IRCd.

  Have a look at modules.default.conf which contains the "default" set of
  modules that you can load if you just want to load all functionality.
  If you want to customize the list of modules to load then simply make
  a copy of that file, give it a different name, and include that one
  instead. Since the file is fully documented, you can just comment out
  or delete the loadmodule lines of things you don't want to load.
* Oper permissions have changed completely: [A4+]
  * All previous oper levels/ranks no longer exist (Netadmin, Admin, ..)
  * oper::flags has been removed. Instead you must specify an operclass
    in oper::operclass (for example, 'operclass netadmin').
  * In operclass block(s) you define the privileges. You can now control
    exactly what an IRCOp can and cannot do.
    Have a look at operclass.default.conf which ships with UnrealIRCd,
    it contains a number of default operclass blocks suitable for the
    most common situations. See also the operclass block documentation:
    https://www.unrealircd.org/docs/Operclass_block
  * If you ask UnrealIRCd to convert your 3.2.x configuration file then
    it will try to select a suitable operclass for the oper. This will
    not always 100% match your current oper block rights, though.
  * Channel Mode +A (Admin Only) has been removed. You can use the new
    extended ban ~O:<operclass>. This allows you to, for example, create
    an operclass 'netadmin' only channel: /MODE #chan +iI ~O:netadmin*
  * set::hosts has been removed, use oper::vhost instead.
  * Since oper levels have been removed you no longer see things like
    "OperX is a Network Administrator" in /WHOIS by default.
    If you want that, then you can set oper::swhois to
    "is a Network Administrator" (or any other text).
* Entirely rewritten I/O and event loop. This allows the IRCd to scale
  more easily to tens of thousands of clients by using kernel-evented I/O
  mechanisms such as epoll and kqueue.
* Memory pooling has been added to improve memory allocation efficiency
  and performance.
* On-connect DNSBL/RBL checking via the new blacklist block. [B1]
* The Windows version now has IPv6 support too. [B3]
* On all OS's we compile with IPv6 support enabled. You can still
  disable IPv6 at runtime by setting set::options::disable-ipv6. [B3]
* The local nickname length can be modified without recompiling the IRCd
* Channel Mode +d: This will hide joins/parts for users who don't say
  anything in a channel. Whenever a user speaks for the first time they
  will appear to join. Chanops will still see everyone joining normally
  as if there was no +d set.
* If you connect with SSL/TLS with a client certificate then your SSL
  Fingerprint (SHA256 hash) can be seen by yourself and others through
  /WHOIS. The fingerprint is also shared with all servers on the network.
* ExtBan ~S:<certificate fingerprint> for ban exceptions / invex. This
  can be used like +iI ~S:000000000etc.
* bcrypt has been added as a password hashing algorithm and is now the
  preferred algorithm [A3]
* './unreal mkpasswd' will now prompt you for the password to hash [A3]
* Protection against SSL renegotiation attacks [A3]
* When you link two servers the current timestamp is exchanged. If the
  time differs more than 60 seconds then servers won't link and it will
  show a message that you should fix your clock(s). This requires
  version alpha3 (or later) on both ends of the link [A3]
* Configuration file converter that will upgrade your 3.2.x conf to 4.x.
  On *NIX run './unreal upgrade-conf'. On Windows simply try to boot and
  after the config errors screen UnrealIRCd offers the conversion. [A3]
* The IRCd can now better handle unknown channel modes which expect a
  parameter. This can be useful in a scenario where you are slowly
  upgrading all your servers.
* If you want to unset a vhost but keep cloaked then use /MODE yournick -t
* A "crash reporter" was added. When UnrealIRCd is started it will check
  if a previous UnrealIRCd instance crashed and (after booting a new
  instance) it will spit out a report and ask if you want to submit it
  to the UnrealIRCd developers. Doing so will help us a lot as many bugs
  are often not reported. Note that UnrealIRCd will always ask before
  sending any information and never do so automatically. [B3]
* SSL: Support for ECDHE has been added to provide "forward secrecy". [B4]

==[ CHANGED ]==
* Numerics have been removed. Instead we now use SIDs (Server ID's) and
  UIDs (User ID's). SIDs work very similar to server numerics and UIDs 
  help us to fix a number of lag-related race conditions / bugs.
* The module commands.so / commands.dll has been removed. All commands
  (those that are modular) are now in their own module.
* Self-signed certificates are now generated using 4096 bits, a SHA256
  hash and validity of 10 years. [A2]
* Building with SSL (OpenSSL) is now mandatory [A2]
* The link { } block has been restructured, see
  https://www.unrealircd.org/docs/Upgrading_from_3.2.x#Link_block [A3]
* Better yet, check out our secure server linking tutorial:
  https://www.unrealircd.org/docs/Tutorial:_Linking_servers
* If you have no set::throttle block you now get a default of 3:60 [A3]
* password entries in the conf no longer require specifying an auth-type
  like password "..." { md5; };. UnrealIRCd will now auto-detect. [A3]
* You will now see a warning when you link to a non-SSL server. [A3]
* Previously we used POSIX Regular expressions in spamfilters and at
  some other places. We have now moved to PCRE Regular expressions.
  They look very similar, but PCRE is a lot faster.
  For backwards-compatibility we still compile with both regex engines. [A3]
* Spamfilter command syntax has been changed, it now has an extra option
  to indicate the matching method:
  /SPAMFILTER [add|del|remove|+|-] [method] [type] ....
  Where 'method' can be one of:
  * -regex: this is the new fast PCRE2 regex engine
  * -simple: supports just strings and ? and * wildcards (super fast)
  * -posix: the old regex engine for compatibility with 3.2.x.  [A3]
* If you have both 3.2.x and 4.x servers on your network then the
  4.x server will only send spamfilters of type 'posix' to the 3.2.x
  servers because 3.2.x servers don't support the other two types.
  So in a mixed network you probably want to keep using 'posix' for
  a while until all your servers are running UnrealIRCd 4. [A3]
* set::oper-only-stats now defaults to "*"
* oper::from::userhost and vhost::from::userhost are now called
  oper::mask and vhost::mask. The usermask@ part is now optional and
  it supports two syntaxes. For one entry you can use: mask 1.2.3.*;
  For multiple entries the syntax is: mask { 192.168.*; 10.*; };
* Because having both allow::ip and allow::hostname in the same allow
  block was highly confusing (it was an OR-match) you must now choose
  between either allow::ip OR allow::hostname. [A3]
* cgiirc block is renamed to webirc and the syntax has changed [A4]
* set::pingpong-warning is removed, warning always off now [A4]
* More helpful configuration file parse error messages [A4]
* You can use '/OPER username' without password if you use SSL
  certificate (fingerprint) authentication. The same is true for
  '/VHOST username'. [A4]
* You must now always use 'make install' on *NIX [A4]
* Changed (default) directory structure entirely, see the section
  titled 'CONFIGURATION CHANGES' about 100 lines up. [A4]
* badword quit { } is removed, we use badword channel for it. [A4]
* badwords.*.conf is now just one badwords.conf
* To load all default modules you now include modules.default.conf.
  This file was called modules.conf in earlier alpha's.
  The file has been split up in sections and a lot of comments have
  been added to aid the user in deciding whether to load or not to
  load each module. [A4]
* Snomask +s is now (always) IRCOp-only. [A4]
* Previously there was little logic behind what modes halfops could
  set. Now the idea is as follows: halfops should be able to help out
  in case of a flood but not be able to change any 'policy decission
  modes' such as +G, +S, +c, +s. Due to this change halfops can now
  set modes +beiklmntIMKNCR (was: +beikmntI). [A4]
* If no link::hub or link::leaf is specified then assume hub "*". [B1]
* SWHOIS (Special whois title) has been extended in a number of ways:
  * We now "track" who or what set an swhois. This allows us to
    remove the swhois received via oper/vhost on de-oper/de-vhost.
  * You can now have multiple swhois lines
  * Multiple oper::swhois and vhost::swhois items are supported. [B1]
* When trying to link two servers without link::outgoing::options::ssl
  (which is not recommended) we try to use STARTTLS in order to
  'upgrade' the connection to use SSL/TLS anyway. This can be disabled
  via link::outgoing::options::insecure. [B2]
* SSLv3 has now been disabled for security. This also means you can only
  link UnrealIRCd 4 with 3.2.10.3 and later because earlier versions
  used SSLv3 instead of TLS due to an OpenSSL API mistake. [B4]

==[ MODULE CODERS / DEVELOPERS ]==
* A lot of technical documentation for module coders has been added
  at https://www.unrealircd.org/docs/ describing things like how to
  write a module from scratch, the User & Channel Mode System, Commands,
  Command Overrides, Hooks, attaching custom-data to users/channels,
  and more. [A2+]
* For commands: do not read from parv[0] anymore, doing so will lead
  to a crash. Use sptr->name instead. This change is necessary as
  the "name" in parv[0] could possibly point to a UID/SID rather than
  a nick name. Thus, if you would send parv[0] to a non-UID or non-SID
  capable server this would lead to serious issues (not found errors).
* Added MOD_OPT_PERM_RELOADABLE which permits reloading (eg: upgrades)
  but disallows unloading of a module [A3]
* There have been *a lot* of source code cleanups (ALL)
* We now use the information from PROTOCTL CHANMODES= for parameter
  skipping if the channel mode is unknown. Also, when channel modes
  are loaded or unloaded we re-broadcast PROTOCTL CHANMODES=. [B1]
* The server protocol docs have been removed. The protocol is now
  documented at https://www.unrealircd.org/docs/Server_protocol
  See also https://www.unrealircd.org/docs/Server_protocol:Changes
  for a list of changes between the 3.2 and 4.0 server protocol.
* GCC typechecking has been added to make sure your HookAdd... calls
  are adding hook functions with the correct parameter (types).

==[ REMOVED / DROPPED ]==
* Numeric server IDs, see above. [A1]
* PROTOCTL TOKEN and SJB64 are no longer implemented. [A1]
* Ziplinks have been removed. [A1]
* WebTV support. [A3]
* Channel Mode +j was removed and replaced by the configuration setting
  set::anti-flood::join-flood (default: 3 per 90 seconds). [B1]
* /CHATOPS: use /GLOBOPS instead which does the same
  /ADCHAT & /NACHAT: gone as we don't have such oper levels anymore
  Your opers should actually be in an #opers channel. If you also want
  special classes of oper channels like #admins then use +iI ~O:*admin*
* User modes:
  * +N (Network Administrator): see 'Oper permissions' under NEW as for why
  * +a (Services Administrator): same
  * +A (Server Administrator: same
  * +C (Co Administrator): same
  * +O (Local IRC Operator): same
  * +h (HelpOp): all this did was add a line "is available for help" in
    WHOIS. You can use a vhost block with vhost::swhois as a replacement
    or for opers just add an oper::swhois item.
  * +g (failops): we already have snomasks and the +o usermode for this
  * +v (receive infected DCC SEND rejection notices): moved to snomask +D
[Enter to continue]
We will now ask you a number of questions.
You can just press ENTER to accept the defaults!

If you have previously installed UnrealIRCd on this shell then you can specify a
directory here so I can import the build settings and third party modules
to make your life a little easier.
If you install UnrealIRCd for the first time on this shell, then just hit Enter
[] -> 

In what directory do you want to install UnrealIRCd?
(Note: UnrealIRCd 4 will need to be installed somewhere.
 If this directory does not exist it will be created.)
[/root/unrealircd] -> 

What should the default permissions for your configuration files be? (Set this to 0 to disable)
It is strongly recommended that you use 0600 to prevent unwanted reading of the file
[0600] -> 

If you know the path to OpenSSL on your system, enter it here. If not
leave this blank (in most cases it will be detected automatically).
[] -> 

Do you want to enable remote includes?
This allows stuff like this in your configuration file:
include "http://www.somesite.org/files/opers.conf";
[No] -> 

Do you want to enable prefixes for chanadmin and chanowner?
This will give +a the & prefix and ~ for +q (just like +o is @)
Supported by the major clients (mIRC, xchat, epic, eggdrop, Klient,
PJIRC, irssi, CGI:IRC, etc.)
This feature should be enabled/disabled network-wide.
[Yes] -> 

How far back do you want to keep the nickname history?
[2000] -> 

What is the maximum sendq length you wish to have?
[3000000] -> 


How many file descriptors (or sockets) can the IRCd use?
[1024] -> 

Would you like to pass any custom parameters to configure?
See  `./configure --help' and write them here:
[] -> 
./configure --with-showlistmodes --enable-ssl --with-bindir=/root/unrealircd/bin --with-datadir=/root/unrealircd/data --with-pidfile=/root/unrealircd/data/unrealircd.pid --with-confdir=/root/unrealircd/conf --with-modulesdir=/root/unrealircd/modules --with-logdir=/root/unrealircd/logs --with-cachedir=/root/unrealircd/cache --with-docdir=/root/unrealircd/doc --with-tmpdir=/root/unrealircd/tmp --with-scriptdir=/root/unrealircd --with-nick-history=2000 --with-sendq=3000000 --with-permissions=0600 --with-fd-setsize=1024 --enable-dynamic-linking
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking if gcc has a working -pipe... yes
checking for rm... /usr/bin/rm
checking for cp... /usr/bin/cp
checking for touch... /usr/bin/touch
checking for openssl... /usr/bin/openssl
checking for install... /usr/bin/install
checking for gmake... gmake
checking for gmake... /usr/bin/gmake
checking for gunzip... /usr/bin/gunzip
checking for pkg-config... /usr/bin/pkg-config
checking for crypt in -ldescrypt... no
checking for crypt in -lcrypt... yes
checking for socket in -lsocket... no
checking for inet_ntoa in -lnsl... yes
checking for RAND_egd in -lcrypto... no
checking if your system has IPv6 support... yes
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking sys/param.h usability... yes
checking sys/param.h presence... yes
checking for sys/param.h... yes
checking for stdlib.h... (cached) yes
checking stddef.h usability... yes
checking stddef.h presence... yes
checking for stddef.h... yes
checking sys/syslog.h usability... yes
checking sys/syslog.h presence... yes
checking for sys/syslog.h... yes
checking for unistd.h... (cached) yes
checking for string.h... (cached) yes
checking for strings.h... (cached) yes
checking malloc.h usability... yes
checking malloc.h presence... yes
checking for malloc.h... yes
checking sys/rusage.h usability... no
checking sys/rusage.h presence... no
checking for sys/rusage.h... no
checking glob.h usability... yes
checking glob.h presence... yes
checking for glob.h... yes
checking for stdint.h... (cached) yes
checking for inttypes.h... (cached) yes
checking for an ANSI C-conforming const... yes
checking for inline... inline
checking for mode_t... yes
checking for size_t... yes
checking for intptr_t... yes
checking whether time.h and sys/time.h may both be included... yes
checking whether struct tm is in sys/time.h or time.h... time.h
checking for uid_t in sys/types.h... yes
checking size of short... 2
checking size of int... 4
checking size of long... 8
checking for int16_t... yes
checking for u_int16_t... yes
checking for int32_t... yes
checking for u_int32_t... yes
checking size of rlim_t... 0
checking what kind of nonblocking sockets you have... O_NONBLOCK
checking whether gcc needs -traditional... no
checking whether setpgrp takes no argument... yes
checking for snprintf... yes
checking for vsnprintf... yes
checking for strlcpy... no
checking for strlcat... no
checking for strlncat... no
checking for inet_pton... yes
checking for inet_ntop... yes
checking if C99 variable length arrays are supported... yes
checking if we can set the core size to unlimited... yes
checking for vprintf... yes
checking for _doprnt... no
checking for gettimeofday... yes
checking for getrusage... yes
checking for setproctitle... no
checking for setproctitle in -lutil... no
checking for pstat... no
checking what type of signals you have... POSIX
checking for strtoken... no
checking for strtok... yes
checking for strerror... yes
checking for index... yes
checking for strtoul... yes
checking for bcopy... yes
checking for bcmp... yes
checking for bzero... yes
checking for strcasecmp... yes
checking for inet_addr... yes
checking for inet_ntoa... yes
checking for syslog... yes
checking for openssl... not found

Apparently you do not have both the openssl binary and openssl development libraries installed.
Please install the needed binaries and libraries.
The package is often called 'openssl-dev', 'openssl-devel' or 'libssl-dev'
After doing so re-run ./Config
[root@h104 unrealircd-4.0.2]# echo $?
1
[root@h104 unrealircd-4.0.2]#
{% endhighlight %}

### 报错

报错内容

{% highlight bash %}
checking for openssl... not found

Apparently you do not have both the openssl binary and openssl development libraries installed.
Please install the needed binaries and libraries.
The package is often called 'openssl-dev', 'openssl-devel' or 'libssl-dev'
After doing so re-run ./Config
{% endhighlight %}

由于依赖 **openssl-devel** , 在它缺失的情况下就会报找不到的错

{% highlight bash %}
[root@h104 unrealircd-4.0.2]# rpm -qa | egrep -E '(openssl|libssl)'
openssl-1.0.1e-51.el7_2.2.x86_64
openssl098e-0.9.8e-29.el7.centos.2.x86_64
openssl-libs-1.0.1e-51.el7_2.2.x86_64
[root@h104 unrealircd-4.0.2]# 
{% endhighlight %}

解决办法是给系统装上这个包

{% highlight bash %}
[root@h104 unrealircd-4.0.2]# yum install openssl-devel.x86_64
Loaded plugins: fastestmirror, langpacks
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
base                                                                                                           | 3.6 kB  00:00:00     
docker-main-repo                                                                                               | 2.9 kB  00:00:00     
dockerrepo                                                                                                     | 2.9 kB  00:00:00     
extras                                                                                                         | 3.4 kB  00:00:00     
updates                                                                                                        | 3.4 kB  00:00:00     
(1/4): extras/7/x86_64/primary_db                                                                              | 101 kB  00:00:00     
(2/4): docker-main-repo/primary_db                                                                             |  13 kB  00:00:00     
(3/4): dockerrepo/7/primary_db                                                                                 |  13 kB  00:00:00     
(4/4): updates/7/x86_64/primary_db                                                                             | 3.2 MB  00:00:01     
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * extras: mirrors.skyshe.cn
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package openssl-devel.x86_64 1:1.0.1e-51.el7_2.4 will be installed
--> Processing Dependency: openssl-libs(x86-64) = 1:1.0.1e-51.el7_2.4 for package: 1:openssl-devel-1.0.1e-51.el7_2.4.x86_64
--> Processing Dependency: zlib-devel(x86-64) for package: 1:openssl-devel-1.0.1e-51.el7_2.4.x86_64
--> Processing Dependency: krb5-devel(x86-64) for package: 1:openssl-devel-1.0.1e-51.el7_2.4.x86_64
--> Running transaction check
---> Package krb5-devel.x86_64 0:1.13.2-10.el7 will be installed
--> Processing Dependency: libverto-devel for package: krb5-devel-1.13.2-10.el7.x86_64
--> Processing Dependency: libselinux-devel for package: krb5-devel-1.13.2-10.el7.x86_64
--> Processing Dependency: libcom_err-devel for package: krb5-devel-1.13.2-10.el7.x86_64
--> Processing Dependency: keyutils-libs-devel for package: krb5-devel-1.13.2-10.el7.x86_64
---> Package openssl-libs.x86_64 1:1.0.1e-51.el7_2.2 will be updated
--> Processing Dependency: openssl-libs(x86-64) = 1:1.0.1e-51.el7_2.2 for package: 1:openssl-1.0.1e-51.el7_2.2.x86_64
---> Package openssl-libs.x86_64 1:1.0.1e-51.el7_2.4 will be an update
---> Package zlib-devel.x86_64 0:1.2.7-15.el7 will be installed
--> Running transaction check
---> Package keyutils-libs-devel.x86_64 0:1.5.8-3.el7 will be installed
---> Package libcom_err-devel.x86_64 0:1.42.9-7.el7 will be installed
---> Package libselinux-devel.x86_64 0:2.2.2-6.el7 will be installed
--> Processing Dependency: libsepol-devel >= 2.1.9-1 for package: libselinux-devel-2.2.2-6.el7.x86_64
--> Processing Dependency: pkgconfig(libsepol) for package: libselinux-devel-2.2.2-6.el7.x86_64
--> Processing Dependency: pkgconfig(libpcre) for package: libselinux-devel-2.2.2-6.el7.x86_64
---> Package libverto-devel.x86_64 0:0.2.5-4.el7 will be installed
---> Package openssl.x86_64 1:1.0.1e-51.el7_2.2 will be updated
---> Package openssl.x86_64 1:1.0.1e-51.el7_2.4 will be an update
--> Running transaction check
---> Package libsepol-devel.x86_64 0:2.1.9-3.el7 will be installed
---> Package pcre-devel.x86_64 0:8.32-15.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                               Arch                     Version                               Repository                 Size
======================================================================================================================================
Installing:
 openssl-devel                         x86_64                   1:1.0.1e-51.el7_2.4                   updates                   1.2 M
Installing for dependencies:
 keyutils-libs-devel                   x86_64                   1.5.8-3.el7                           base                       37 k
 krb5-devel                            x86_64                   1.13.2-10.el7                         base                      649 k
 libcom_err-devel                      x86_64                   1.42.9-7.el7                          base                       30 k
 libselinux-devel                      x86_64                   2.2.2-6.el7                           base                      174 k
 libsepol-devel                        x86_64                   2.1.9-3.el7                           base                       71 k
 libverto-devel                        x86_64                   0.2.5-4.el7                           base                       12 k
 pcre-devel                            x86_64                   8.32-15.el7                           base                      478 k
 zlib-devel                            x86_64                   1.2.7-15.el7                          base                       50 k
Updating for dependencies:
 openssl                               x86_64                   1:1.0.1e-51.el7_2.4                   updates                   711 k
 openssl-libs                          x86_64                   1:1.0.1e-51.el7_2.4                   updates                   951 k

Transaction Summary
======================================================================================================================================
Install  1 Package  (+8 Dependent packages)
Upgrade             ( 2 Dependent packages)

Total download size: 4.3 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/11): keyutils-libs-devel-1.5.8-3.el7.x86_64.rpm                                                             |  37 kB  00:00:00     
(2/11): libverto-devel-0.2.5-4.el7.x86_64.rpm                                                                  |  12 kB  00:00:00     
(3/11): libcom_err-devel-1.42.9-7.el7.x86_64.rpm                                                               |  30 kB  00:00:00     
(4/11): libsepol-devel-2.1.9-3.el7.x86_64.rpm                                                                  |  71 kB  00:00:00     
(5/11): openssl-1.0.1e-51.el7_2.4.x86_64.rpm                                                                   | 711 kB  00:00:00     
(6/11): krb5-devel-1.13.2-10.el7.x86_64.rpm                                                                    | 649 kB  00:00:01     
(7/11): openssl-libs-1.0.1e-51.el7_2.4.x86_64.rpm                                                              | 951 kB  00:00:01     
(8/11): pcre-devel-8.32-15.el7.x86_64.rpm                                                                      | 478 kB  00:00:00     
(9/11): zlib-devel-1.2.7-15.el7.x86_64.rpm                                                                     |  50 kB  00:00:00     
(10/11): libselinux-devel-2.2.2-6.el7.x86_64.rpm                                                               | 174 kB  00:00:02     
(11/11): openssl-devel-1.0.1e-51.el7_2.4.x86_64.rpm                                                            | 1.2 MB  00:00:22     
--------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                 190 kB/s | 4.3 MB  00:00:23     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Updating   : 1:openssl-libs-1.0.1e-51.el7_2.4.x86_64                                                                           1/13 
  Installing : keyutils-libs-devel-1.5.8-3.el7.x86_64                                                                            2/13 
  Installing : libcom_err-devel-1.42.9-7.el7.x86_64                                                                              3/13 
  Installing : libsepol-devel-2.1.9-3.el7.x86_64                                                                                 4/13 
  Installing : pcre-devel-8.32-15.el7.x86_64                                                                                     5/13 
  Installing : libselinux-devel-2.2.2-6.el7.x86_64                                                                               6/13 
  Installing : libverto-devel-0.2.5-4.el7.x86_64                                                                                 7/13 
  Installing : krb5-devel-1.13.2-10.el7.x86_64                                                                                   8/13 
  Installing : zlib-devel-1.2.7-15.el7.x86_64                                                                                    9/13 
  Installing : 1:openssl-devel-1.0.1e-51.el7_2.4.x86_64                                                                         10/13 
  Updating   : 1:openssl-1.0.1e-51.el7_2.4.x86_64                                                                               11/13 
  Cleanup    : 1:openssl-1.0.1e-51.el7_2.2.x86_64                                                                               12/13 
  Cleanup    : 1:openssl-libs-1.0.1e-51.el7_2.2.x86_64                                                                          13/13 
  Verifying  : zlib-devel-1.2.7-15.el7.x86_64                                                                                    1/13 
  Verifying  : libselinux-devel-2.2.2-6.el7.x86_64                                                                               2/13 
  Verifying  : libverto-devel-0.2.5-4.el7.x86_64                                                                                 3/13 
  Verifying  : 1:openssl-devel-1.0.1e-51.el7_2.4.x86_64                                                                          4/13 
  Verifying  : pcre-devel-8.32-15.el7.x86_64                                                                                     5/13 
  Verifying  : krb5-devel-1.13.2-10.el7.x86_64                                                                                   6/13 
  Verifying  : 1:openssl-1.0.1e-51.el7_2.4.x86_64                                                                                7/13 
  Verifying  : libsepol-devel-2.1.9-3.el7.x86_64                                                                                 8/13 
  Verifying  : 1:openssl-libs-1.0.1e-51.el7_2.4.x86_64                                                                           9/13 
  Verifying  : libcom_err-devel-1.42.9-7.el7.x86_64                                                                             10/13 
  Verifying  : keyutils-libs-devel-1.5.8-3.el7.x86_64                                                                           11/13 
  Verifying  : 1:openssl-libs-1.0.1e-51.el7_2.2.x86_64                                                                          12/13 
  Verifying  : 1:openssl-1.0.1e-51.el7_2.2.x86_64                                                                               13/13 

Installed:
  openssl-devel.x86_64 1:1.0.1e-51.el7_2.4                                                                                            

Dependency Installed:
  keyutils-libs-devel.x86_64 0:1.5.8-3.el7      krb5-devel.x86_64 0:1.13.2-10.el7        libcom_err-devel.x86_64 0:1.42.9-7.el7     
  libselinux-devel.x86_64 0:2.2.2-6.el7         libsepol-devel.x86_64 0:2.1.9-3.el7      libverto-devel.x86_64 0:0.2.5-4.el7        
  pcre-devel.x86_64 0:8.32-15.el7               zlib-devel.x86_64 0:1.2.7-15.el7        

Dependency Updated:
  openssl.x86_64 1:1.0.1e-51.el7_2.4                              openssl-libs.x86_64 1:1.0.1e-51.el7_2.4                             

Complete!
[root@h104 unrealircd-4.0.2]# echo $?
0
[root@h104 unrealircd-4.0.2]#
{% endhighlight %}

再次运行

{% highlight bash %}
[root@h104 unrealircd-4.0.2]# ./Config 

 _   _                      _ ___________  _____     _
| | | |                    | |_   _| ___ \/  __ \   | |
| | | |_ __  _ __ ___  __ _| | | | | |_/ /| /  \/ __| |
| | | | '_ \| '__/ _ \/  _ | | | | |    / | |    /  _ |
| |_| | | | | | |  __/ (_| | |_| |_| |\ \ | \__/\ (_| |
 \___/|_| |_|_|  \___|\__,_|_|\___/\_| \_| \____/\__,_|

                               Configuration Program
                                for UnrealIRCd 4.0.2
                                    
This program will help you to compile your IRC server, and ask you
questions regarding the compile-time settings of it during the process. 
regarding the setup of it, during the process.

A short installation guide is available online at:
https://www.unrealircd.org/docs/Installing_from_source

Full documentation is available at:
https://www.unrealircd.org/docs/UnrealIRCd_4_documentation
[Enter to continue]
UnrealIRCd 4.0.2 Release Notes
===============================

UnrealIRCd 4 is here!

We have been working hard over the past few years to replace the successful
3.2.x series with a more modern code base. At the same time we have been
incorporating requests from our bug tracker, ideas from ourselves and
many suggestions that came up during the UnrealIRCd survey from Q4 2013.
After 4 alpha versions, 4 beta's and 6 release candidates we are proud to
finally present you the first stable release of UnrealIRCd 4.

UnrealIRCd is far more modular and configurable than before. For a brief
overview of what's new in UnrealIRCd 4 have a look at:
https://www.unrealircd.org/docs/What's_new_in_UnrealIRCd_4

==[ DOCUMENTATION ]==
All documentation has been moved to our wiki:
* Documentation: https://www.unrealircd.org/docs/
* FAQ: https://www.unrealircd.org/docs/FAQ

Be sure not to use any other (older) documentation as it isn't fully
compatible with UnrealIRCd 4. In particular, do NOT use unreal32docs*html.

==[ UPGRADING FROM 3.2.x ]==
If you are upgrading from 3.2.x then there are three important things to know:

1) NEW FILE LOCATIONS
In UnrealIRCd 4 the location of the configuration files and other files have
been changed. On *NIX the directory where you compile the IRCd from
(previously 'Unreal3.2.X', now 'unrealircd-4.0.X') is no longer the same as
the directory where the IRCd will be running from.
By default the IRCd is installed to /home/yourusername/unrealircd on *NIX
On Windows UnrealIRCd will install to C:\Program Files (x86\UnrealIRCd 4

The new directory structure is as follows (both on Windows and *NIX):
conf/      contains all configuration files
logs/      for log files
modules/   all modules (.so files on *NIX, .dll files on Windows)

2) CONFIGURATION FILE CHANGES
There have also been changes in various configuration blocks and settings.
Don't worry, UnrealIRCd can convert your existing 3.2.x configuration files
to UnrealIRCd 4 format. There's no need to start from scratch.

Please read https://www.unrealircd.org/docs/Upgrading_from_3.2.x !!

3) THIRD PARTY MODULES
If you are using 3rd party modules then they will need an update to run on
UnrealIRCd 4. Due to the many core changes in UnrealIRCd 4 it was simply
impossible to make 3.2.x modules work out-of-the-box on 4.x.
Contact your developer for a new version or ask on our Modules forum where
someone may be kind enough to convert the module for you if you ask nicely:
https://forums.unrealircd.org/viewforum.php?f=52

==[ RUNNING A MIXED 3.2.X / 4.X NETWORK ]==
You can run a mixed 3.2.x <-> 4.x network if you a follow a few simple rules:
https://www.unrealircd.org/docs/Running_a_mixed_UnrealIRCd_3.2_and_UnrealIRCd_4_network

==[ END OF THE 3.2.X SERIES ]==
With the release of UnrealIRCd 4 we are deprecating the previous series.
All support for the 3.2.x series will stop after December 31, 2016.
See https://www.unrealircd.org/docs/UnrealIRCd_3.2.x_deprecated

==[ SUPPORT ]==
Before you seek support, please check our Frequently Asked Questions:
https://www.unrealircd.org/docs/FAQ

For support you have two choices:
* Forums: https://forums.unrealircd.org/
* IRC: irc.unrealircd.org / #unreal-support

==[ CHANGES BETWEEN 4.0.1 AND 4.0.2 ]==
The 4.0.2 release comes with the following new features:
* Ability to hide quit messages from *LINEd users (set::hide-ban-reason)
* Blacklist hits are now sent to new snomask +b rather than all ircops

Major issues fixed:
* None

Minor issues fixed:
* prefix-quit was not working
* FreeBSD: fix kevent bug flood in error log
* Incorrect server description in /LINKS
* Logging to syslog was broken
* OS X: Update ./Config to use Homebrew OpenSSL by default
* Don't show UID to client in case of a SVSMODE

==[ CHANGES BETWEEN 4.0.0 AND 4.0.1 ]==
The 4.0.1 release comes with the following minor improvements:
* The blacklist module now supports %ip (=banned IP) in blacklist::reason.
* *NIX: You can use cron again, see https://www.unrealircd.org/docs/Cron_job
* /MODULE now lists only 3rd party modules by default so you don't get flooded.
* *NIX: Added './unrealircd reloadtls' to reload TLS certificate and keys.

Major issue fixed:
* Crash if you removed a listen { } block with active clients on that port
* MODEs set by a server (not by a user) were not always propagated
  correctly accross the network. In practice this only affected /SAMODE
  and possibly some services that don't send MODEs from ChanServ/BotServ.

Minor issues fixed:
* When doing /LIST under mIRC it would hide empty +P channels.
* Servers wouldn't link if link::outgoing::hostname was a CNAME.
* SSL Certificate fingerprint not communicated properly to servers/services.
* *NIX: ./unrealircd [stop|rehash] failed if not installed to ~/unrealircd.
* Windows: IRCd could crash after showing the config error screen on startup.

==[ CHANGES BETWEEN 3.2.X AND 4.X ]==
Below is a summary of the changes between UnrealIRCd 3.2.x and UnrealIRCd 4.
For a complete list of all 1100+ changes you can use 'git log' or have a
look at: https://github.com/unrealircd/unrealircd/commits/unreal40

==[ NEW ]==
* We moved a lot of functionality, including most channel modes, user
  modes and all extended bans into 138 separate modules.
  This makes it...
  A) possible to fully customize what exact functionality you want to load.
     You could even strip down UnrealIRCd to get something close to the
     basic RFC1459 features from the 1990s. (No idea why you would want
     that, but it's possible)
  B) easier for coders to see all source code related to a specific feature
  C) possible to fix bugs and just reload rather than restart the IRCd.

  Have a look at modules.default.conf which contains the "default" set of
  modules that you can load if you just want to load all functionality.
  If you want to customize the list of modules to load then simply make
  a copy of that file, give it a different name, and include that one
  instead. Since the file is fully documented, you can just comment out
  or delete the loadmodule lines of things you don't want to load.
* Oper permissions have changed completely: [A4+]
  * All previous oper levels/ranks no longer exist (Netadmin, Admin, ..)
  * oper::flags has been removed. Instead you must specify an operclass
    in oper::operclass (for example, 'operclass netadmin').
  * In operclass block(s) you define the privileges. You can now control
    exactly what an IRCOp can and cannot do.
    Have a look at operclass.default.conf which ships with UnrealIRCd,
    it contains a number of default operclass blocks suitable for the
    most common situations. See also the operclass block documentation:
    https://www.unrealircd.org/docs/Operclass_block
  * If you ask UnrealIRCd to convert your 3.2.x configuration file then
    it will try to select a suitable operclass for the oper. This will
    not always 100% match your current oper block rights, though.
  * Channel Mode +A (Admin Only) has been removed. You can use the new
    extended ban ~O:<operclass>. This allows you to, for example, create
    an operclass 'netadmin' only channel: /MODE #chan +iI ~O:netadmin*
  * set::hosts has been removed, use oper::vhost instead.
  * Since oper levels have been removed you no longer see things like
    "OperX is a Network Administrator" in /WHOIS by default.
    If you want that, then you can set oper::swhois to
    "is a Network Administrator" (or any other text).
* Entirely rewritten I/O and event loop. This allows the IRCd to scale
  more easily to tens of thousands of clients by using kernel-evented I/O
  mechanisms such as epoll and kqueue.
* Memory pooling has been added to improve memory allocation efficiency
  and performance.
* On-connect DNSBL/RBL checking via the new blacklist block. [B1]
* The Windows version now has IPv6 support too. [B3]
* On all OS's we compile with IPv6 support enabled. You can still
  disable IPv6 at runtime by setting set::options::disable-ipv6. [B3]
* The local nickname length can be modified without recompiling the IRCd
* Channel Mode +d: This will hide joins/parts for users who don't say
  anything in a channel. Whenever a user speaks for the first time they
  will appear to join. Chanops will still see everyone joining normally
  as if there was no +d set.
* If you connect with SSL/TLS with a client certificate then your SSL
  Fingerprint (SHA256 hash) can be seen by yourself and others through
  /WHOIS. The fingerprint is also shared with all servers on the network.
* ExtBan ~S:<certificate fingerprint> for ban exceptions / invex. This
  can be used like +iI ~S:000000000etc.
* bcrypt has been added as a password hashing algorithm and is now the
  preferred algorithm [A3]
* './unreal mkpasswd' will now prompt you for the password to hash [A3]
* Protection against SSL renegotiation attacks [A3]
* When you link two servers the current timestamp is exchanged. If the
  time differs more than 60 seconds then servers won't link and it will
  show a message that you should fix your clock(s). This requires
  version alpha3 (or later) on both ends of the link [A3]
* Configuration file converter that will upgrade your 3.2.x conf to 4.x.
  On *NIX run './unreal upgrade-conf'. On Windows simply try to boot and
  after the config errors screen UnrealIRCd offers the conversion. [A3]
* The IRCd can now better handle unknown channel modes which expect a
  parameter. This can be useful in a scenario where you are slowly
  upgrading all your servers.
* If you want to unset a vhost but keep cloaked then use /MODE yournick -t
* A "crash reporter" was added. When UnrealIRCd is started it will check
  if a previous UnrealIRCd instance crashed and (after booting a new
  instance) it will spit out a report and ask if you want to submit it
  to the UnrealIRCd developers. Doing so will help us a lot as many bugs
  are often not reported. Note that UnrealIRCd will always ask before
  sending any information and never do so automatically. [B3]
* SSL: Support for ECDHE has been added to provide "forward secrecy". [B4]

==[ CHANGED ]==
* Numerics have been removed. Instead we now use SIDs (Server ID's) and
  UIDs (User ID's). SIDs work very similar to server numerics and UIDs 
  help us to fix a number of lag-related race conditions / bugs.
* The module commands.so / commands.dll has been removed. All commands
  (those that are modular) are now in their own module.
* Self-signed certificates are now generated using 4096 bits, a SHA256
  hash and validity of 10 years. [A2]
* Building with SSL (OpenSSL) is now mandatory [A2]
* The link { } block has been restructured, see
  https://www.unrealircd.org/docs/Upgrading_from_3.2.x#Link_block [A3]
* Better yet, check out our secure server linking tutorial:
  https://www.unrealircd.org/docs/Tutorial:_Linking_servers
* If you have no set::throttle block you now get a default of 3:60 [A3]
* password entries in the conf no longer require specifying an auth-type
  like password "..." { md5; };. UnrealIRCd will now auto-detect. [A3]
* You will now see a warning when you link to a non-SSL server. [A3]
* Previously we used POSIX Regular expressions in spamfilters and at
  some other places. We have now moved to PCRE Regular expressions.
  They look very similar, but PCRE is a lot faster.
  For backwards-compatibility we still compile with both regex engines. [A3]
* Spamfilter command syntax has been changed, it now has an extra option
  to indicate the matching method:
  /SPAMFILTER [add|del|remove|+|-] [method] [type] ....
  Where 'method' can be one of:
  * -regex: this is the new fast PCRE2 regex engine
  * -simple: supports just strings and ? and * wildcards (super fast)
  * -posix: the old regex engine for compatibility with 3.2.x.  [A3]
* If you have both 3.2.x and 4.x servers on your network then the
  4.x server will only send spamfilters of type 'posix' to the 3.2.x
  servers because 3.2.x servers don't support the other two types.
  So in a mixed network you probably want to keep using 'posix' for
  a while until all your servers are running UnrealIRCd 4. [A3]
* set::oper-only-stats now defaults to "*"
* oper::from::userhost and vhost::from::userhost are now called
  oper::mask and vhost::mask. The usermask@ part is now optional and
  it supports two syntaxes. For one entry you can use: mask 1.2.3.*;
  For multiple entries the syntax is: mask { 192.168.*; 10.*; };
* Because having both allow::ip and allow::hostname in the same allow
  block was highly confusing (it was an OR-match) you must now choose
  between either allow::ip OR allow::hostname. [A3]
* cgiirc block is renamed to webirc and the syntax has changed [A4]
* set::pingpong-warning is removed, warning always off now [A4]
* More helpful configuration file parse error messages [A4]
* You can use '/OPER username' without password if you use SSL
  certificate (fingerprint) authentication. The same is true for
  '/VHOST username'. [A4]
* You must now always use 'make install' on *NIX [A4]
* Changed (default) directory structure entirely, see the section
  titled 'CONFIGURATION CHANGES' about 100 lines up. [A4]
* badword quit { } is removed, we use badword channel for it. [A4]
* badwords.*.conf is now just one badwords.conf
* To load all default modules you now include modules.default.conf.
  This file was called modules.conf in earlier alpha's.
  The file has been split up in sections and a lot of comments have
  been added to aid the user in deciding whether to load or not to
  load each module. [A4]
* Snomask +s is now (always) IRCOp-only. [A4]
* Previously there was little logic behind what modes halfops could
  set. Now the idea is as follows: halfops should be able to help out
  in case of a flood but not be able to change any 'policy decission
  modes' such as +G, +S, +c, +s. Due to this change halfops can now
  set modes +beiklmntIMKNCR (was: +beikmntI). [A4]
* If no link::hub or link::leaf is specified then assume hub "*". [B1]
* SWHOIS (Special whois title) has been extended in a number of ways:
  * We now "track" who or what set an swhois. This allows us to
    remove the swhois received via oper/vhost on de-oper/de-vhost.
  * You can now have multiple swhois lines
  * Multiple oper::swhois and vhost::swhois items are supported. [B1]
* When trying to link two servers without link::outgoing::options::ssl
  (which is not recommended) we try to use STARTTLS in order to
  'upgrade' the connection to use SSL/TLS anyway. This can be disabled
  via link::outgoing::options::insecure. [B2]
* SSLv3 has now been disabled for security. This also means you can only
  link UnrealIRCd 4 with 3.2.10.3 and later because earlier versions
  used SSLv3 instead of TLS due to an OpenSSL API mistake. [B4]

==[ MODULE CODERS / DEVELOPERS ]==
* A lot of technical documentation for module coders has been added
  at https://www.unrealircd.org/docs/ describing things like how to
  write a module from scratch, the User & Channel Mode System, Commands,
  Command Overrides, Hooks, attaching custom-data to users/channels,
  and more. [A2+]
* For commands: do not read from parv[0] anymore, doing so will lead
  to a crash. Use sptr->name instead. This change is necessary as
  the "name" in parv[0] could possibly point to a UID/SID rather than
  a nick name. Thus, if you would send parv[0] to a non-UID or non-SID
  capable server this would lead to serious issues (not found errors).
* Added MOD_OPT_PERM_RELOADABLE which permits reloading (eg: upgrades)
  but disallows unloading of a module [A3]
* There have been *a lot* of source code cleanups (ALL)
* We now use the information from PROTOCTL CHANMODES= for parameter
  skipping if the channel mode is unknown. Also, when channel modes
  are loaded or unloaded we re-broadcast PROTOCTL CHANMODES=. [B1]
* The server protocol docs have been removed. The protocol is now
  documented at https://www.unrealircd.org/docs/Server_protocol
  See also https://www.unrealircd.org/docs/Server_protocol:Changes
  for a list of changes between the 3.2 and 4.0 server protocol.
* GCC typechecking has been added to make sure your HookAdd... calls
  are adding hook functions with the correct parameter (types).

==[ REMOVED / DROPPED ]==
* Numeric server IDs, see above. [A1]
* PROTOCTL TOKEN and SJB64 are no longer implemented. [A1]
* Ziplinks have been removed. [A1]
* WebTV support. [A3]
* Channel Mode +j was removed and replaced by the configuration setting
  set::anti-flood::join-flood (default: 3 per 90 seconds). [B1]
* /CHATOPS: use /GLOBOPS instead which does the same
  /ADCHAT & /NACHAT: gone as we don't have such oper levels anymore
  Your opers should actually be in an #opers channel. If you also want
  special classes of oper channels like #admins then use +iI ~O:*admin*
* User modes:
  * +N (Network Administrator): see 'Oper permissions' under NEW as for why
  * +a (Services Administrator): same
  * +A (Server Administrator: same
  * +C (Co Administrator): same
  * +O (Local IRC Operator): same
  * +h (HelpOp): all this did was add a line "is available for help" in
    WHOIS. You can use a vhost block with vhost::swhois as a replacement
    or for opers just add an oper::swhois item.
  * +g (failops): we already have snomasks and the +o usermode for this
  * +v (receive infected DCC SEND rejection notices): moved to snomask +D
[Enter to continue]
We will now ask you a number of questions.
You can just press ENTER to accept the defaults!


In what directory do you want to install UnrealIRCd?
(Note: UnrealIRCd 4 will need to be installed somewhere.
 If this directory does not exist it will be created.)
[/root/unrealircd] -> 

What should the default permissions for your configuration files be? (Set this to 0 to disable)
It is strongly recommended that you use 0600 to prevent unwanted reading of the file
[0600] -> 

If you know the path to OpenSSL on your system, enter it here. If not
leave this blank (in most cases it will be detected automatically).
[] -> 

Do you want to enable remote includes?
This allows stuff like this in your configuration file:
include "http://www.somesite.org/files/opers.conf";
[No] -> 

Do you want to enable prefixes for chanadmin and chanowner?
This will give +a the & prefix and ~ for +q (just like +o is @)
Supported by the major clients (mIRC, xchat, epic, eggdrop, Klient,
PJIRC, irssi, CGI:IRC, etc.)
This feature should be enabled/disabled network-wide.
[Yes] -> 

How far back do you want to keep the nickname history?
[2000] -> 

What is the maximum sendq length you wish to have?
[3000000] -> 


How many file descriptors (or sockets) can the IRCd use?
[1024] -> 

Would you like to pass any custom parameters to configure?
See  `./configure --help' and write them here:
[] -> 
./configure --with-showlistmodes --enable-ssl --with-bindir=/root/unrealircd/bin --with-datadir=/root/unrealircd/data --with-pidfile=/root/unrealircd/data/unrealircd.pid --with-confdir=/root/unrealircd/conf --with-modulesdir=/root/unrealircd/modules --with-logdir=/root/unrealircd/logs --with-cachedir=/root/unrealircd/cache --with-docdir=/root/unrealircd/doc --with-tmpdir=/root/unrealircd/tmp --with-scriptdir=/root/unrealircd --with-nick-history=2000 --with-sendq=3000000 --with-permissions=0600 --with-fd-setsize=1024 --enable-dynamic-linking
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking if gcc has a working -pipe... yes
checking for rm... /usr/bin/rm
checking for cp... /usr/bin/cp
checking for touch... /usr/bin/touch
checking for openssl... /usr/bin/openssl
checking for install... /usr/bin/install
checking for gmake... gmake
...
...
Configuration summary
=====================

TRE is now configured as follows:

* Compilation environment

  CC       = gcc
  CFLAGS   = -g -O2 -Wall
  CPP      = gcc -E
  CPPFLAGS = 
  LD       = /usr/bin/ld -m elf_x86_64
  LDFLAGS  = 
  LIBS     = 
  Use alloca():                       yes

* TRE options

  Development-time debugging:         no
  System regex ABI compatibility:     no
  Wide character (wchar_t) support:   no (disabled with --disable-wchar)
  Multibyte character set support:    no (disabled with --disable-multibyte)
  Approximate matching support:       yes
  Build and install agrep:            no
...
...
configure: creating ./config.status
config.status: creating Makefile
config.status: creating src/modules/Makefile
config.status: creating src/modules/chanmodes/Makefile
config.status: creating src/modules/usermodes/Makefile
config.status: creating src/modules/snomasks/Makefile
config.status: creating src/modules/extbans/Makefile
config.status: creating src/modules/third/Makefile
config.status: creating unrealircd
config.status: creating include/setup.h

Do you want to generate an SSL certificate for the IRCd?
Only answer No if you already have one.
[Yes] -> Yes
Generating certificate request .. 
/usr/bin/openssl req -new \
              -config src/ssl.cnf -sha256 -out server.req.pem \
              -keyout server.key.pem -nodes
Generating a 4096 bit RSA private key
...........................++
............++
writing new private key to 'server.key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name [US]:CN    
State/Province [New York]:Shanghai
Locality Name (eg, city) []:Shanghai   
Organization Name (eg, company) [IRC geeks]:havefun
Organizational Unit Name (eg, section) [IRCd]:IRCd
Common Name (Full domain of your server) []:h104.example.com
Generating self-signed certificate .. 
/usr/bin/openssl req -x509 -days 3650 -sha256 -in server.req.pem \
               -key server.key.pem -out server.cert.pem
Generating fingerprint ..
/usr/bin/openssl x509 -subject -dates -sha256 -fingerprint -noout \
	-in server.cert.pem
subject= /C=CN/ST=Shanghai/L=Shanghai/O=havefun/OU=IRCd/CN=h104.example.com
notBefore=Mar 21 14:20:53 2016 GMT
notAfter=Mar 19 14:20:53 2026 GMT
SHA256 Fingerprint=05:CD:52:91:5B:22:67:D4:65:8A:06:1A:87:EF:D1:6B:9A:08:9E:FA:F4:B7:C4:43:04:C3:2D:C2:98:37:B6:31
Setting o-rwx & g-rwx for files... 
chmod o-rwx server.req.pem server.key.pem server.cert.pem
chmod g-rwx server.req.pem server.key.pem server.cert.pem
Done!. If you want to encrypt the private key, run
make encpem
Certificate created successfully.

 _______________________________________________________________________
|                                                                       |
|                    UnrealIRCd Compile-Time Config                     |
|_______________________________________________________________________|
|_______________________________________________________________________|
|                                                                       |
| Now all you have to do is type 'make' and let it compile. When that's |
| done, you will receive other instructions on what to do next.         |
|                                                                       |
|_______________________________________________________________________|
|_______________________________________________________________________|
|                        - The UnrealIRCd Team -                        |
|                                                                       |
| * Bram Matthys (Syzop)     syzop@unrealircd.org                       |
| * Travis McArthur (Heero)  heero@unrealircd.org                       |
|_______________________________________________________________________|
[root@h104 unrealircd-4.0.2]# echo $?
0
[root@h104 unrealircd-4.0.2]# 
{% endhighlight %}



> **Tip:** **`./Config`** 脚本会引导我们阅读新版的变更与注意事项，然后收集软件配置，收集配置信息的过程中，可以按需指定，也可以直接回车接受默认值，最后会引导我们创建一个SSL证书，根据具体环境进行设置就好了


然后编译

{% highlight bash %}
[root@h104 unrealircd-4.0.2]# make 
Building src
make[1]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src'
gcc -I/usr/local/src/irc/unrealircd-4.0.2/include -pthread -I/usr/local/src/irc/unrealircd-4.0.2/extras/regexp/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/include -pipe -g -O2 -funsigned-char -fno-strict-aliasing -Wno-pointer-sign -Wl,-export-dynamic   -c timesynch.c
gcc -I/usr/local/src/irc/unrealircd-4.0.2/include -pthread -I/usr/local/src/irc/unrealircd-4.0.2/extras/regexp/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/include -pipe -g -O2 -funsigned-char -fno-strict-aliasing -Wno-pointer-sign -Wl,-export-dynamic   -c res.c
gcc -I/usr/local/src/irc/unrealircd-4.0.2/include -pthread -I/usr/local/src/irc/unrealircd-4.0.2/extras/regexp/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/include -pipe -g -O2 -funsigned-char -fno-strict-aliasing -Wno-pointer-sign -Wl,-export-dynamic   -fPIC -DPIC -shared -DDYNAMIC_LINKING \
	-o account.so account.c
gcc -I/usr/local/src/irc/unrealircd-4.0.2/include -pthread -I/usr/local/src/irc/unrealircd-4.0.2/extras/regexp/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/include -pipe -g -O2 -funsigned-char -fno-strict-aliasing -Wno-pointer-sign -Wl,-export-dynamic   -fPIC -DPIC -shared -DDYNAMIC_LINKING \
	-o operclass.so operclass.c
gcc -I/usr/local/src/irc/unrealircd-4.0.2/include -pthread -I/usr/local/src/irc/unrealircd-4.0.2/extras/regexp/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/include -pipe -g -O2 -funsigned-char -fno-strict-aliasing -Wno-pointer-sign -Wl,-export-dynamic   -fPIC -DPIC -shared -DDYNAMIC_LINKING \
	-o certfp.so certfp.c
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/extbans'
cd third; make all
make[3]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/third'
../../buildmod
Building all third party modules...
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/third'
make[2]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules'
make[1]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src'

* UnrealIRCd compiled successfully
* YOU ARE NOT DONE YET! Run "make install" to install UnrealIRCd !

[root@h104 unrealircd-4.0.2]# echo $?
0
[root@h104 unrealircd-4.0.2]# 
{% endhighlight %}

最后安装

{% highlight bash %}
[root@h104 unrealircd-4.0.2]# make install 
Building src
make[1]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src'
cd modules; make 'CFLAGS=-I/usr/local/src/irc/unrealircd-4.0.2/include -pthread -I/usr/local/src/irc/unrealircd-4.0.2/extras/regexp/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/include -I/usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/include -pipe -g -O2 -funsigned-char -fno-strict-aliasing -Wno-pointer-sign -Wl,-export-dynamic  ' 'CC=gcc' 'IRCDLIBS=-lcrypt -lnsl  -ldl /usr/local/src/irc/unrealircd-4.0.2/extras/regexp/lib/libtre.a     -pthread -L/usr/local/src/irc/unrealircd-4.0.2/extras/pcre2/lib -lpcre2-8   -L../extras/c-ares/lib /usr/local/src/irc/unrealircd-4.0.2/extras/c-ares/lib/libcares.a   ' 'LDFLAGS=' 'IRCDMODE=711' 'BINDIR=/root/unrealircd/bin' 'INSTALL=/usr/bin/install' 'INCLUDEDIR=/usr/local/src/irc/unrealircd-4.0.2/include' 'MANDIR=' 'RM=/usr/bin/rm' 'CP=/usr/bin/cp' 'TOUCH=/usr/bin/touch' 'RES=' 'SHELL=/bin/sh' 'STRTOUL=' 'CRYPTOLIB=-lssl -lcrypto' 'CRYPTOINCLUDES=' 'URL='  all
make[2]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules'
cd chanmodes; make all
make[3]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/chanmodes'
make[3]: Nothing to be done for `all'.
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/chanmodes'
cd usermodes; make all
make[3]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/usermodes'
make[3]: Nothing to be done for `all'.
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/usermodes'
cd snomasks; make all
make[3]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/snomasks'
make[3]: Nothing to be done for `all'.
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/snomasks'
cd extbans; make all
make[3]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/extbans'
make[3]: Nothing to be done for `all'.
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/extbans'
cd third; make all
make[3]: Entering directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/third'
../../buildmod
Building all third party modules...
make[3]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules/third'
make[2]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src/modules'
make[1]: Leaving directory `/usr/local/src/irc/unrealircd-4.0.2/src'

* UnrealIRCd compiled successfully
* YOU ARE NOT DONE YET! Run "make install" to install UnrealIRCd !

/usr/bin/install -m 0700 -d /root/unrealircd/bin
/usr/bin/install -m 0700 src/ircd /root/unrealircd/bin/unrealircd
/usr/bin/install -m 0700 -d /root/unrealircd/doc
/usr/bin/install -m 0600 doc/Authors doc/coding-guidelines doc/tao.of.irc /root/unrealircd/doc
/usr/bin/install -m 0700 -d /root/unrealircd/conf
/usr/bin/install -m 0600 doc/conf/*.conf /root/unrealircd/conf
/usr/bin/install -m 0700 -d /root/unrealircd/conf/aliases
/usr/bin/install -m 0600 doc/conf/aliases/*.conf /root/unrealircd/conf/aliases
/usr/bin/install -m 0700 -d /root/unrealircd/conf/help
/usr/bin/install -m 0600 doc/conf/help/*.conf /root/unrealircd/conf/help
/usr/bin/install -m 0700 -d /root/unrealircd/conf/examples
/usr/bin/install -m 0600 doc/conf/examples/*.conf /root/unrealircd/conf/examples
/usr/bin/install -m 0700 -d /root/unrealircd/conf/ssl
/usr/bin/install -m 0600 doc/conf/ssl/curl-ca-bundle.crt /root/unrealircd/conf/ssl
/usr/bin/install -m 0700 unrealircd /root/unrealircd
/usr/bin/install -m 0700 -d /root/unrealircd/modules
/usr/bin/install -m 0700 src/modules/*.so /root/unrealircd/modules
/usr/bin/install -m 0700 -d /root/unrealircd/modules/usermodes
/usr/bin/install -m 0700 src/modules/usermodes/*.so /root/unrealircd/modules/usermodes
/usr/bin/install -m 0700 -d /root/unrealircd/modules/chanmodes
/usr/bin/install -m 0700 src/modules/chanmodes/*.so /root/unrealircd/modules/chanmodes
/usr/bin/install -m 0700 -d /root/unrealircd/modules/snomasks
/usr/bin/install -m 0700 src/modules/snomasks/*.so /root/unrealircd/modules/snomasks
/usr/bin/install -m 0700 -d /root/unrealircd/modules/extbans
/usr/bin/install -m 0700 src/modules/extbans/*.so /root/unrealircd/modules/extbans
/usr/bin/install -m 0700 -d /root/unrealircd/modules/third
/usr/bin/install: cannot stat ‘src/modules/third/*.so’: No such file or directory

/usr/bin/install -m 0700 -d /root/unrealircd/tmp
/usr/bin/install -m 0700 -d /root/unrealircd/cache
/usr/bin/install -m 0700 -d /root/unrealircd/data
/usr/bin/install -m 0700 -d /root/unrealircd/logs

* UnrealIRCd is now installed.
* Leave this directory and run "cd /root/unrealircd" now
* Directory layout:
 * Base directory: /root/unrealircd
  * Configuration files: /root/unrealircd/conf
  * Log files: /root/unrealircd/logs
  * Modules: /root/unrealircd/modules
* To start/stop UnrealIRCd run: /root/unrealircd/unrealircd"

* Consult the documentation online at:
  * https://www.unrealircd.org/docs/UnrealIRCd_4_documentation
  * https://www.unrealircd.org/docs/FAQ
* You may also wish to install a cron job to ensure UnrealIRCd is always running:
  * https://www.unrealircd.org/docs/Cron_job

Again, be sure to change to the /root/unrealircd directory!
[root@h104 unrealircd-4.0.2]# echo $?
0
[root@h104 unrealircd-4.0.2]# 
{% endhighlight %}


> **Tip:** 由于我以root的身份配置安装(生产中不要这样)，并且接受的默认配置，于是它给安装到了 **/root/unrealircd** 下

{% highlight bash %}
[root@h104 unrealircd-4.0.2]# ll /root/unrealircd/
total 16
drwx------ 2 root root   23 Mar 21 22:57 bin
drwx------ 2 root root    6 Mar 21 22:57 cache
drwx------ 6 root root 4096 Mar 21 22:57 conf
drwx------ 2 root root    6 Mar 21 22:57 data
drwx------ 2 root root   61 Mar 21 22:57 doc
drwx------ 2 root root    6 Mar 21 22:57 logs
drwx------ 7 root root 4096 Mar 21 22:57 modules
drwx------ 2 root root    6 Mar 21 22:20 tmp
-rwx------ 1 root root 7039 Mar 21 22:57 unrealircd
[root@h104 unrealircd-4.0.2]# tree /root/unrealircd/
/root/unrealircd/
├── bin
│   └── unrealircd
├── cache
├── conf
│   ├── aliases
│   │   ├── aliases.conf
│   │   ├── anope.conf
│   │   ├── atheme.conf
│   │   ├── auspice.conf
│   │   ├── cygnus.conf
│   │   ├── epona.conf
│   │   ├── generic.conf
│   │   ├── genericstats.conf
│   │   ├── ircservices.conf
│   │   └── operstats.conf
│   ├── badwords.conf
│   ├── dccallow.conf
│   ├── examples
│   │   ├── example.conf
│   │   ├── example.fr.conf
│   │   └── example.tr.conf
│   ├── help
│   │   ├── help.conf
│   │   ├── help.de.conf
│   │   ├── help.fr.conf
│   │   ├── help.ru.conf
│   │   └── help.tr.conf
│   ├── modules.default.conf
│   ├── operclass.default.conf
│   ├── spamfilter.conf
│   └── ssl
│       ├── curl-ca-bundle.crt
│       ├── server.cert.pem
│       ├── server.key.pem
│       └── server.req.pem
├── data
├── doc
│   ├── Authors
│   ├── coding-guidelines
│   └── tao.of.irc
├── logs
├── modules
│   ├── blacklist.so
│   ├── certfp.so
│   ├── chanmodes
│   │   ├── censor.so
│   │   ├── delayjoin.so
│   │   ├── floodprot.so
│   │   ├── issecure.so
│   │   ├── link.so
│   │   ├── nocolor.so
│   │   ├── noctcp.so
│   │   ├── noinvite.so
│   │   ├── nokick.so
│   │   ├── noknock.so
│   │   ├── nonickchange.so
│   │   ├── nonotice.so
│   │   ├── operonly.so
│   │   ├── permanent.so
│   │   ├── regonly.so
│   │   ├── regonlyspeak.so
│   │   ├── secureonly.so
│   │   └── stripcolor.so
│   ├── cloak.so
│   ├── extbans
│   │   ├── account.so
│   │   ├── certfp.so
│   │   ├── inchannel.so
│   │   ├── join.so
│   │   ├── nickchange.so
│   │   ├── operclass.so
│   │   ├── quiet.so
│   │   ├── realname.so
│   │   └── regnick.so
│   ├── jointhrottle.so
│   ├── m_addmotd.so
│   ├── m_addomotd.so
│   ├── m_admin.so
│   ├── m_away.so
│   ├── m_botmotd.so
│   ├── m_cap.so
│   ├── m_chghost.so
│   ├── m_chgident.so
│   ├── m_chgname.so
│   ├── m_close.so
│   ├── m_connect.so
│   ├── m_cycle.so
│   ├── m_dccallow.so
│   ├── m_dccdeny.so
│   ├── m_eos.so
│   ├── m_globops.so
│   ├── m_help.so
│   ├── m_invite.so
│   ├── m_ison.so
│   ├── m_join.so
│   ├── m_kick.so
│   ├── m_kill.so
│   ├── m_knock.so
│   ├── m_lag.so
│   ├── m_links.so
│   ├── m_list.so
│   ├── m_locops.so
│   ├── m_lusers.so
│   ├── m_map.so
│   ├── m_md.so
│   ├── m_message.so
│   ├── m_mkpasswd.so
│   ├── m_mode.so
│   ├── m_motd.so
│   ├── m_names.so
│   ├── m_netinfo.so
│   ├── m_nick.so
│   ├── m_nopost.so
│   ├── m_opermotd.so
│   ├── m_oper.so
│   ├── m_part.so
│   ├── m_pass.so
│   ├── m_pingpong.so
│   ├── m_protoctl.so
│   ├── m_quit.so
│   ├── m_rping.so
│   ├── m_rules.so
│   ├── m_sajoin.so
│   ├── m_samode.so
│   ├── m_sapart.so
│   ├── m_sasl.so
│   ├── m_sdesc.so
│   ├── m_sendsno.so
│   ├── m_sendumode.so
│   ├── m_server.so
│   ├── m_sethost.so
│   ├── m_setident.so
│   ├── m_setname.so
│   ├── m_silence.so
│   ├── m_sjoin.so
│   ├── m_sqline.so
│   ├── m_squit.so
│   ├── m_starttls.so
│   ├── m_stats.so
│   ├── m_svsfline.so
│   ├── m_svsjoin.so
│   ├── m_svskill.so
│   ├── m_svslusers.so
│   ├── m_svsmode.so
│   ├── m_svsmotd.so
│   ├── m_svsnick.so
│   ├── m_svsnline.so
│   ├── m_svsnolag.so
│   ├── m_svsnoop.so
│   ├── m_svspart.so
│   ├── m_svssilence.so
│   ├── m_svssno.so
│   ├── m_svswatch.so
│   ├── m_swhois.so
│   ├── m_time.so
│   ├── m_tkl.so
│   ├── m_topic.so
│   ├── m_trace.so
│   ├── m_tsctl.so
│   ├── m_umode2.so
│   ├── m_undccdeny.so
│   ├── m_unsqline.so
│   ├── m_userhost.so
│   ├── m_userip.so
│   ├── m_user.so
│   ├── m_vhost.so
│   ├── m_wallops.so
│   ├── m_watch.so
│   ├── m_whois.so
│   ├── m_who.so
│   ├── m_whowas.so
│   ├── snomasks
│   │   └── dccreject.so
│   ├── ssl_antidos.so
│   ├── third
│   ├── usermodes
│   │   ├── bot.so
│   │   ├── censor.so
│   │   ├── noctcp.so
│   │   ├── nokick.so
│   │   ├── privacy.so
│   │   ├── regonlymsg.so
│   │   ├── servicebot.so
│   │   └── showwhois.so
│   └── webirc.so
├── tmp
└── unrealircd

17 directories, 170 files
[root@h104 unrealircd-4.0.2]# 
{% endhighlight %}



---

## 配置 

**`conf/examples/example.conf`** 是一个配置模板，默认系统会读取 **`conf/unrealircd.conf`** 中的配置

于是拷贝过来，直接使用

{% highlight bash %}
[root@h104 unrealircd]# cp conf/examples/example.conf conf/unrealircd.conf
[root@h104 unrealircd]# 
[root@h104 unrealircd]# bin/unrealircd 
WARNING: You are running UnrealIRCd as root and it is not
         configured to drop priviliges. This is VERY dangerous,
         as any compromise of your UnrealIRCd is the same as
         giving a cracker root SSH access to your box.
         You should either start UnrealIRCd under a different
         account than root, or set IRC_USER in include/config.h
         to a nonprivileged username and recompile.
 _   _                      _ ___________  _____     _ 
| | | |                    | |_   _| ___ \/  __ \   | |
| | | |_ __  _ __ ___  __ _| | | | | |_/ /| /  \/ __| |
| | | | '_ \| '__/ _ \/ _` | | | | |    / | |    / _` |
| |_| | | | | | |  __/ (_| | |_| |_| |\ \ | \__/\ (_| |
 \___/|_| |_|_|  \___|\__,_|_|\___/\_| \_| \____/\__,_|
                           v4.0.2

  using TRE 0.8.0 (BSD)
  using OpenSSL 1.0.1e-fips 11 Feb 2013

Loading IRCd configuration..
config error: /root/unrealircd/conf/unrealircd.conf:144: please change the the name and password of the default 'bobsmith' oper block
config error: /root/unrealircd/conf/unrealircd.conf:378: set::cloak-keys: (key 2) Keys should be mixed a-zA-Z0-9, like "a2JO6fh3Q6w4oN3s7"
config error: /root/unrealircd/conf/unrealircd.conf:379: set::cloak-keys: (key 3) Keys should be mixed a-zA-Z0-9, like "a2JO6fh3Q6w4oN3s7"
config error: /root/unrealircd/conf/unrealircd.conf:376: set::cloak-keys: All your 3 keys should be RANDOM, they should not be equal
config error: /root/unrealircd/conf/unrealircd.conf:386: set::kline-address must be an e-mail or an URL
config error: 5 errors encountered
config error: IRCd configuration failed to pass testing
[root@h104 unrealircd]# 
{% endhighlight %}


### 警告

{% highlight bash %}
WARNING: You are running UnrealIRCd as root and it is not
         configured to drop priviliges. This is VERY dangerous,
         as any compromise of your UnrealIRCd is the same as
         giving a cracker root SSH access to your box.
         You should either start UnrealIRCd under a different
         account than root, or set IRC_USER in include/config.h
         to a nonprivileged username and recompile.

{% endhighlight %}

我们是使用的root，如果使用的普通用户，就不会有这样的警告，如：

{% highlight bash %}
[root@h104 unrealircd]# su - irc 
Last login: Tue Mar 22 00:17:00 CST 2016 on pts/0
[irc@h104 ~]$ /usr/local/unrealircd/bin/unrealircd 
 _   _                      _ ___________  _____     _ 
| | | |                    | |_   _| ___ \/  __ \   | |
| | | |_ __  _ __ ___  __ _| | | | | |_/ /| /  \/ __| |
| | | | '_ \| '__/ _ \/ _` | | | | |    / | |    / _` |
| |_| | | | | | |  __/ (_| | |_| |_| |\ \ | \__/\ (_| |
 \___/|_| |_|_|  \___|\__,_|_|\___/\_| \_| \____/\__,_|
                           v4.0.2

  using TRE 0.8.0 (BSD)
  using OpenSSL 1.0.1e-fips 11 Feb 2013

Loading IRCd configuration..
Configuration loaded without any problems.
Loading tunefile..
Initializing SSL..
Dynamic configuration initialized.. booting IRCd.
UnrealIRCd is now listening on the following addresses/ports:
IPv4: *:6900(SSL), *:6697(SSL), *:6667
IPv6: *:6900(SSL), *:6697(SSL), *:6667
UnrealIRCd started.
[irc@h104 ~]$ 
{% endhighlight %}

### 报错

{% highlight bash %}
Loading IRCd configuration..
config error: /root/unrealircd/conf/unrealircd.conf:144: please change the the name and password of the default 'bobsmith' oper block
config error: /root/unrealircd/conf/unrealircd.conf:378: set::cloak-keys: (key 2) Keys should be mixed a-zA-Z0-9, like "a2JO6fh3Q6w4oN3s7"
config error: /root/unrealircd/conf/unrealircd.conf:379: set::cloak-keys: (key 3) Keys should be mixed a-zA-Z0-9, like "a2JO6fh3Q6w4oN3s7"
config error: /root/unrealircd/conf/unrealircd.conf:376: set::cloak-keys: All your 3 keys should be RANDOM, they should not be equal
config error: /root/unrealircd/conf/unrealircd.conf:386: set::kline-address must be an e-mail or an URL
config error: 5 errors encountered
config error: IRCd configuration failed to pass testing
{% endhighlight %}

分别为：

* 1.要求修改操作员角色的用户名和密码
* 2.**cloak-keys** 必须得相互不一样，并且由 **`a-zA-Z0-9`** 混合而成(且必须包含这三种字符形式，少任何一种都会提示不通过)
* 3.**kline-address** 必须得是一个邮箱地址格式

根据定位，依次修改，顺便修改一下访问权限

{% highlight bash %}
[root@h104 unrealircd]# vim conf/unrealircd.conf 
[root@h104 unrealircd]# 
[root@h104 unrealircd]# diff conf/examples/example.conf  conf/unrealircd.conf 
60c60
< 	name "irc.foonet.com";
---
> 	name "h104.example.com";
128c128
< 	ip *@192.0.2.1;
---
> 	ip *@192.168.100.*;
144c144
< oper bobsmith {
---
> oper bob {
147c147
< 	password "test";
---
> 	password "bob";
378,379c378,381
< 		"and another one";
< 		"and another one";
---
> 		#"and another one";
> 		"asflkjweoQQWJQWLKSOSDLFnh234233450flakdfjnaf";
> 		"asflkjweawefwj2o32932nasdfajfAKFAOIF0flakdfjnaf";
> 		#"and another one";
386c388
< 	kline-address "set.this.to.email.address"; /* e-mail or URL shown when a user is banned */
---
> 	kline-address "yyghdfz@163.com"; /* e-mail or URL shown when a user is banned */
[root@h104 unrealircd]# 
{% endhighlight %}

这里只是最基本的，使其可以运行并且能够访问的配置，详细的配置方法，放到以后再作讲解

可以参考 **[UnrealIRCd 配置语法][unrealircd_conf_syntax]**

还可以参考 **[UnrealIRCd 所有配置][unrealircd_conf]**


---

## 启动

尝试启动服务

{% highlight bash %}
[root@h104 unrealircd]# bin/unrealircd 
WARNING: You are running UnrealIRCd as root and it is not
         configured to drop priviliges. This is VERY dangerous,
         as any compromise of your UnrealIRCd is the same as
         giving a cracker root SSH access to your box.
         You should either start UnrealIRCd under a different
         account than root, or set IRC_USER in include/config.h
         to a nonprivileged username and recompile.
 _   _                      _ ___________  _____     _ 
| | | |                    | |_   _| ___ \/  __ \   | |
| | | |_ __  _ __ ___  __ _| | | | | |_/ /| /  \/ __| |
| | | | '_ \| '__/ _ \/ _` | | | | |    / | |    / _` |
| |_| | | | | | |  __/ (_| | |_| |_| |\ \ | \__/\ (_| |
 \___/|_| |_|_|  \___|\__,_|_|\___/\_| \_| \____/\__,_|
                           v4.0.2

  using TRE 0.8.0 (BSD)
  using OpenSSL 1.0.1e-fips 11 Feb 2013

Loading IRCd configuration..
Configuration loaded without any problems.
Loading tunefile..
Initializing SSL..
Dynamic configuration initialized.. booting IRCd.
UnrealIRCd is now listening on the following addresses/ports:
IPv4: *:6900(SSL), *:6697(SSL), *:6667
IPv6: *:6900(SSL), *:6697(SSL), *:6667
UnrealIRCd started.
[root@h104 unrealircd]# 
[root@h104 unrealircd]# ps faux | grep ircd | grep -v grep
root     27340  0.0  0.3 335156  6632 ?        S    23:04   0:00 bin/unrealircd
[root@h104 unrealircd]# ps -Lf 27340
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
root     27340     1 27340  0    1 23:04 ?        S      0:00 bin/unrealircd
[root@h104 unrealircd]# pstree 27340
unrealircd
[root@h104 unrealircd]#
{% endhighlight %}

可以看到正常启动了，并且打开了几个端口

{% highlight bash %}
[root@h104 unrealircd]# netstat -ant | egrep -E '(6900|6697|6667)'
tcp        0      0 0.0.0.0:6697            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:6667            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:6900            0.0.0.0:*               LISTEN     
tcp6       0      0 :::6697                 :::*                    LISTEN     
tcp6       0      0 :::6667                 :::*                    LISTEN     
tcp6       0      0 :::6900                 :::*                    LISTEN     
[root@h104 unrealircd]# 
{% endhighlight %}


为了能对外正常提供服务，防火墙上这几个端口也要打开，打开方法

{% highlight bash %}
[root@h104 unrealircd]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 4000/tcp 80/tcp 8500/tcp 8080/tcp 3306/tcp 2375/tcp 40000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@h104 unrealircd]# firewall-cmd --add-port 6900/tcp
success
[root@h104 unrealircd]# firewall-cmd --add-port 6697/tcp
success
[root@h104 unrealircd]# firewall-cmd --add-port 6667/tcp
success
[root@h104 unrealircd]# firewall-cmd --list-all 
public (default, active)
  interfaces: eno16777736 eno33554960
  sources: 
  services: dhcpv6-client ssh
  ports: 4000/tcp 80/tcp 6667/tcp 8500/tcp 8080/tcp 3306/tcp 6900/tcp 2375/tcp 6697/tcp 40000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@h104 unrealircd]# 
{% endhighlight %}


---

## 访问

下面是使用 **IceChat9** 访问的界面

![icechat9.png](/images/irc/icechat9.png)


下面是使用 **mIRC7.4.3** 访问的界面

![mirc1.png](/images/irc/mirc1.png)

![mirc2.png](/images/irc/mirc2.png)



---


# 命令汇总

* **`wget https://www.unrealircd.org/unrealircd4/unrealircd-4.0.2.tar.gz`**
* **`gpg --keyserver keys.gnupg.net --recv-keys 0xA7A21B0A108FF4A9`**
* **`wget https://www.unrealircd.org/unrealircd4/unrealircd-4.0.2.tar.gz.asc`**
* **`gpg --verify unrealircd-4.0.2.tar.gz.asc unrealircd-4.0.2.tar.gz`**
* **`sha1sum unrealircd-4.0.2.tar.gz`**
* **`sha256sum unrealircd-4.0.2.tar.gz`**
* **`tar -axvf unrealircd-4.0.2.tar.gz`**
* **`cd unrealircd-4.0.2`**
* **`rpm -qa | egrep -E '(openssl|libssl)'`**
* **`yum install openssl-devel.x86_64`**
* **`./Config`**
* **`make`**
* **`make install`**
* **`tree /root/unrealircd/`**
* **`cp conf/examples/example.conf conf/unrealircd.conf`**
* **`bin/unrealircd`**
* **`su - irc`**
* **`/usr/local/unrealircd/bin/unrealircd`**
* **`vim conf/unrealircd.conf`**
* **`diff conf/examples/example.conf  conf/unrealircd.conf`**
* **`bin/unrealircd`**
* **`ps faux | grep ircd | grep -v grep`**
* **`ps -Lf 27340`**
* **`pstree 27340`**
* **`netstat -ant | egrep -E '(6900|6697|6667)'`**
* **`firewall-cmd --add-port 6900/tcp`**
* **`firewall-cmd --add-port 6697/tcp`**
* **`firewall-cmd --add-port 6667/tcp`**
* **`firewall-cmd --list-all`**


---

[unrealircd]:https://www.unrealircd.org/
[unrealircd_doc]:https://www.unrealircd.org/docs/UnrealIRCd_4_documentation
[unrealircd_dl]:https://www.unrealircd.org/download
[unrealircd_conf]:https://www.unrealircd.org/docs/Configuration
[unrealircd_conf_syntax]:https://www.unrealircd.org/docs/Configuration_file_syntax

