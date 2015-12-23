---
layout: post
title: 安装jekyll (include rvm,ruby,gem)
categories: linux rvm ruby jekyll
wc: 531 1714 19693
excerpt: follow me
comments: true
---





##前言

我前面有一篇文章可以[21分钟搭建一个GitHub Blog](http://wilmosfang.github.io/blog/2015/03/02/build-a-githubblog-in-21minutes.html)

这里我们深入到jekyll层面看看,如何在本地也生成一个blog,也就是在本地干和github上一样的事情

这里涉及到rvm,ruby,gem和jekyll的安装,绝对干货,因为里面已经包含了几个我踩的坑,坑对面的屌丝看过来,看过来,看过来……这里有表演很精彩……

---

#概要

* TOC
{:toc}


---

##系统环境

{% highlight bash %}
[root@Test ~]# lsb_release  -a 
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID:	CentOS
Description:	CentOS release 6.6 (Final)
Release:	6.6
Codename:	Final
[root@Test ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@Test ~]# uname -a 
Linux Test 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@Test ~]# 
{% endhighlight %}

##安装过程


* 1.安装gcc

{% highlight bash %}
[root@Test ~]# yum -y install gcc
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: ftp.sjtu.edu.cn
 * epel: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirrors.skyshe.cn
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.4.7-11.el6 will be installed
--> Processing Dependency: cpp = 4.4.7-11.el6 for package: gcc-4.4.7-11.el6.x86_64
--> Processing Dependency: cloog-ppl >= 0.15 for package: gcc-4.4.7-11.el6.x86_64
--> Running transaction check
---> Package cloog-ppl.x86_64 0:0.15.7-1.2.el6 will be installed
--> Processing Dependency: libppl_c.so.2()(64bit) for package: cloog-ppl-0.15.7-1.2.el6.x86_64
--> Processing Dependency: libppl.so.7()(64bit) for package: cloog-ppl-0.15.7-1.2.el6.x86_64
---> Package cpp.x86_64 0:4.4.7-11.el6 will be installed
--> Processing Dependency: libmpfr.so.1()(64bit) for package: cpp-4.4.7-11.el6.x86_64
--> Running transaction check
---> Package mpfr.x86_64 0:2.4.1-6.el6 will be installed
---> Package ppl.x86_64 0:0.10.2-11.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================
 Package                  Arch                  Version                         Repository           Size
==========================================================================================================
Installing:
 gcc                      x86_64                4.4.7-11.el6                    base                 10 M
Installing for dependencies:
 cloog-ppl                x86_64                0.15.7-1.2.el6                  base                 93 k
 cpp                      x86_64                4.4.7-11.el6                    base                3.7 M
 mpfr                     x86_64                2.4.1-6.el6                     base                157 k
 ppl                      x86_64                0.10.2-11.el6                   base                1.3 M

Transaction Summary
==========================================================================================================
Install       5 Package(s)
...
...
  Verifying  : ppl-0.10.2-11.el6.x86_64                                                               1/5 

  Verifying  : cpp-4.4.7-11.el6.x86_64                                                                2/5 

  Verifying  : gcc-4.4.7-11.el6.x86_64                                                                3/5 

  Verifying  : cloog-ppl-0.15.7-1.2.el6.x86_64                                                        4/5 

  Verifying  : mpfr-2.4.1-6.el6.x86_64                                                                5/5 

Installed:
  gcc.x86_64 0:4.4.7-11.el6                                                                               

Dependency Installed:
  cloog-ppl.x86_64 0:0.15.7-1.2.el6       cpp.x86_64 0:4.4.7-11.el6       mpfr.x86_64 0:2.4.1-6.el6      
  ppl.x86_64 0:0.10.2-11.el6             

Complete!
[root@Test ~]# 

{% endhighlight %}

* 1号坑:安装ruby(用于展示坑的大小,不要跟着操作,因为这是白操作)

系统默认安装好了ruby

{% highlight bash %}
[root@Test ~]# yum list ruby
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: ftp.sjtu.edu.cn
 * epel: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirrors.skyshe.cn
Installed Packages
ruby.x86_64                                   1.8.7.374-4.el6_6                                   @updates
[root@Test ~]# ruby -v 
ruby 1.8.7 (2013-06-27 patchlevel 374) [x86_64-linux]
[root@Test ~]#
{% endhighlight %}

安装rubygems

其中的这些包都是根据错误提示一个个加上去的，这个过程就略过了，至于为什么要安装gem呢，是网上的很多声称实践过的先驱们探索出来的,方便用来安装jekyll用的


{% highlight bash %}
[root@Test ~]# yum -y install rubygems ruby-devel rubygems-devel
{% endhighlight %}

安装jekyll(坑来了，坑来了)

{% highlight bash %}
[root@Test ~]# gem install jekyll  
ERROR:  Error installing jekyll:
	redcarpet requires Ruby version >= 1.9.2.
[root@Test ~]# ruby  -v 
ruby 1.8.7 (2013-06-27 patchlevel 374) [x86_64-linux]
[root@Test ~]#
{% endhighlight %}

按说也就是一个版本问题，升级不得了呗，结果这个系统最高的也就是1.8.7，于是从官网下载源码2.2.1和1.9.3进行尝试，卸载掉ruby，用源码安装成功

{% highlight bash %}
[root@Test ~]# ruby  -v 
ruby 1.9.3p545 (2014-02-24 revision 45159) [x86_64-linux]
[root@Test ~]# 
{% endhighlight %}
可这个不是问题的关键，关键是由于此系统中的gem依赖1.8.7，结果一安装又把ruby覆盖掉了回到了1.8.7,解决办法是源码编译高版本的gem...,当然也不知道到时候又有什么别的依赖....,太浪费时间了

干脆放弃了这个方法

* 2.安装rvm

有了rvm，妈妈再也不担心我的ruby版本了

{% highlight bash %}
[root@Test ~]# curl -L get.rvm.io | bash -s stable 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
184   184  184   184    0     0    323      0 --:--:-- --:--:-- --:--:--   654
184   184  184   184    0     0    323      0 --:--:-- --:--:-- --:--:--   652
  0   184    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 22817  100 22817    0     0  10969      0  0:00:02  0:00:02 --:--:-- 52452
Downloading https://github.com/wayneeseguin/rvm/archive/1.26.10.tar.gz
Downloading https://github.com/wayneeseguin/rvm/releases/download/1.26.10/1.26.10.tar.gz.asc
gpg: Signature made Tue 03 Feb 2015 12:09:00 AM CST using RSA key ID BF04FF17
gpg: Can't check signature: No public key
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found.
Assuming you trust Michal Papis import the mpapis public key (downloading the signatures).

GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.26.10.tgz' - 'https://github.com/wayneeseguin/rvm/releases/download/1.26.10/1.26.10.tar.gz.asc'!
try downloading the signatures:

    gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

the key can be compared with:

    https://rvm.io/mpapis.asc
    https://keybase.io/mpapis

[root@Test ~]# echo $?
2
[root@Test ~]#
{% endhighlight %}

GPG signature verification failed

{% highlight bash %}
[root@Test ~]# gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39 
DC0E3
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: requesting key D39DC0E3 from hkp server keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[root@Test ~]# echo $?
0
[root@Test ~]#
{% endhighlight %}

再次rvm安装就成功了

{% highlight bash %}
[root@Test ~]# curl -L get.rvm.io | bash -s stable 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
184   184  184   184    0     0    307      0 --:--:-- --:--:-- --:--:--   613
184   184  184   184    0     0    307      0 --:--:-- --:--:-- --:--:--   613
  0   184    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 22817  100 22817    0     0  11904      0  0:00:01  0:00:01 --:--:-- 68519
Downloading https://github.com/wayneeseguin/rvm/archive/1.26.10.tar.gz
Downloading https://github.com/wayneeseguin/rvm/releases/download/1.26.10/1.26.10.tar.gz.asc
gpg: Signature made Tue 03 Feb 2015 12:09:00 AM CST using RSA key ID BF04FF17
gpg: Good signature from "Michal Papis (RVM signing) <mpapis@gmail.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
     Subkey fingerprint: 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/rvm/archives/rvm-1.26.10.tgz'

Upgrading the RVM installation in /usr/local/rvm/
Upgrade of RVM in /usr/local/rvm/ is complete.

# Administrator,
#
#   Thank you for using RVM!
#   We sincerely hope that RVM helps to make your life easier and more enjoyable!!!
#
# ~Wayne, Michal & team.

In case of problems: http://rvm.io/help and https://twitter.com/rvm_io

Upgrade Notes:

  * No new notes to display.

[root@Test ~]# echo $?
0
[root@Test ~]#
{% endhighlight %}

切换一下环境，就可以看到rvm命令了

{% highlight bash %}
[root@Test ~]# su - 
[root@Test ~]# rvm -v 
rvm 1.26.10 (latest) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
[root@Test ~]#
{% endhighlight %}


* 3.安装ruby


{% highlight bash %}
[root@Test ~]# rvm install 2.2.1
Searching for binary rubies, this might take some time.
Found remote file https://rvm_io.global.ssl.fastly.net/binaries/centos/6/x86_64/ruby-2.2.1.tar.bz2
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: libyaml-devel, autoconf, gcc-c++, readline-devel, zlib-devel, libffi-devel, openssl-devel, automake, libtool, bison, sqlite-devel................
Requirements installation successful.
...
...
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.2.1 - #validate archive
ruby-2.2.1 - #extract
ruby-2.2.1 - #validate binary
ruby-2.2.1 - #setup
ruby-2.2.1 - #gemset created /usr/local/rvm/gems/ruby-2.2.1@global
ruby-2.2.1 - #importing gemset /usr/local/rvm/gemsets/global.gems....................................
ruby-2.2.1 - #generating global wrappers........
ruby-2.2.1 - #gemset created /usr/local/rvm/gems/ruby-2.2.1
ruby-2.2.1 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.2.1 - #generating default wrappers........
[root@Test ~]# ruby -v 
ruby 2.2.1p85 (2015-02-26 revision 49769) [x86_64-linux]
[root@Test ~]#

{% endhighlight %}

> 有点慢, 整个过程花费了6分钟左右

安装好ruby，gem也跟着安装上去了

{% highlight bash %}
[root@Test ~]# gem -v 
2.4.6
[root@Test ~]#
{% endhighlight %}

* 4.安装jekyll

2号坑来了

{% highlight bash %}
[root@Test ~]# gem install jekyll
^CERROR:  Interrupted
[root@Test ~]# 
{% endhighlight %}

为什么我会去中断，原因是等太久了，具体多少时间没统计，反正超出了我的忍耐

原因是我使用的安装源太远

{% highlight bash %}
[root@Test ~]# gem sources -l 
*** CURRENT SOURCES ***

https://rubygems.org/
[root@Test ~]#
{% endhighlight %}

IP Address : 54.186.104.15 - 1 other site is hosted on this server

IP Location : Oregon - Portland - Amazon.com Inc.

解决办法是替换安装源地址

{% highlight bash %}
[root@Test ~]# gem sources -a https://ruby.taobao.org/
https://ruby.taobao.org/ added to sources
[root@Test ~]# gem sources -l 
*** CURRENT SOURCES ***

https://rubygems.org/
https://ruby.taobao.org/
[root@Test ~]# gem sources --remove https://rubygems.org/
https://rubygems.org/ removed from sources
[root@Test ~]# gem sources -l 
*** CURRENT SOURCES ***

https://ruby.taobao.org/
[root@Test ~]# 
{% endhighlight %}

再次安装jekyll,这回我使用time统计了一下共花费了2分半钟左右

{% highlight bash %}
[root@Test ~]#  time gem install jekyll
Fetching: liquid-2.6.2.gem
Fetching: liquid-2.6.2.gem ( 35%)
Fetching: liquid-2.6.2.gem ( 70%)
Fetching: liquid-2.6.2.gem (100%)
Fetching: liquid-2.6.2.gem (100%)
Successfully installed liquid-2.6.2
...
...
Parsing documentation for jekyll-2.5.3
Installing ri documentation for jekyll-2.5.3
Done installing documentation for liquid, kramdown, mercenary, safe_yaml, colorator, yajl-ruby, posix-spawn, pygments.rb, redcarpet, blankslate, parslet, toml, jekyll-paginate, jekyll-gist, coffee-script-source, execjs, coffee-script, jekyll-coffeescript, sass, jekyll-sass-converter, hitimes, timers, celluloid, rb-fsevent, ffi, rb-inotify, listen, jekyll-watch, fast-stemmer, classifier-reborn, jekyll after 27 seconds
31 gems installed

real	2m21.194s
user	0m46.456s
sys	0m2.128s
[root@Test ~]# echo $?
0
[root@Test ~]#
{% endhighlight %}

* 5.安装nodejs

为什么要安装nodejs呢，因为jekyll要依赖它，不安装会报错，下面是我没安装，常试直接运行的结果 

{% highlight bash %}
[root@Test tmp]# jekyll  new myblog
/usr/local/rvm/gems/ruby-2.2.1/gems/execjs-2.4.0/lib/execjs/runtimes.rb:45:in `autodetect': Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)
	from /usr/local/rvm/gems/ruby-2.2.1/gems/execjs-2.4.0/lib/execjs.rb:5:in `<module:ExecJS>'
	from /usr/local/rvm/rubies/ruby-2.2.1/lib/ruby/site_ruby/2.2.0/rubygems/core_ext/kernel_require.rb:54:in `require'
	from /usr/local/rvm/gems/ruby-2.2.1/gems/jekyll-2.5.3/bin/jekyll:6:in `<top (required)>'
...
...
	from /usr/local/rvm/gems/ruby-2.2.1/bin/jekyll:23:in `load'
	from /usr/local/rvm/gems/ruby-2.2.1/bin/jekyll:23:in `<main>'
	from /usr/local/rvm/gems/ruby-2.2.1/bin/ruby_executable_hooks:15:in `eval'
	from /usr/local/rvm/gems/ruby-2.2.1/bin/ruby_executable_hooks:15:in `<main>'
[root@Test tmp]#  
{% endhighlight %}

安装nodejs

{% highlight bash %}
[root@Test tmp]#  yum -y install nodejs 
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.skyshe.cn
 * epel: mirrors.zju.edu.cn
 * extras: mirrors.zju.edu.cn
 * updates: mirrors.zju.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package nodejs.x86_64 0:0.10.33-1.el6 will be installed
--> Processing Dependency: v8(x86-64) < 1:3.15 for package: nodejs-0.10.33-1.el6.x86_64
--> Processing Dependency: v8(x86-64) >= 1:3.14.5.7 for package: nodejs-0.10.33-1.el6.x86_64
--> Processing Dependency: libv8.so.3()(64bit) for package: nodejs-0.10.33-1.el6.x86_64
--> Processing Dependency: libuv.so.0.10()(64bit) for package: nodejs-0.10.33-1.el6.x86_64
--> Processing Dependency: libhttp_parser.so.2()(64bit) for package: nodejs-0.10.33-1.el6.x86_64
--> Processing Dependency: libcares19.so.2()(64bit) for package: nodejs-0.10.33-1.el6.x86_64
--> Running transaction check
---> Package c-ares19.x86_64 0:1.9.1-5.el6.3 will be installed
---> Package http-parser.x86_64 0:2.0-4.20121128gitcd01361.el6 will be installed
---> Package libuv.x86_64 1:0.10.29-1.el6 will be installed
---> Package v8.x86_64 1:3.14.5.10-14.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================
 Package                Arch              Version                                   Repository       Size
==========================================================================================================
Installing:
 nodejs                 x86_64            0.10.33-1.el6                             epel            500 k
Installing for dependencies:
 c-ares19               x86_64            1.9.1-5.el6.3                             epel             73 k
 http-parser            x86_64            2.0-4.20121128gitcd01361.el6              epel             22 k
 libuv                  x86_64            1:0.10.29-1.el6                           epel             56 k
 v8                     x86_64            1:3.14.5.10-14.el6                        epel            3.0 M

Transaction Summary
==========================================================================================================
Install       5 Package(s)

Total download size: 3.7 M
Installed size: 12 M
...
...
  Verifying  : http-parser-2.0-4.20121128gitcd01361.el6.x86_64                                        1/5 

  Verifying  : 1:v8-3.14.5.10-14.el6.x86_64                                                           2/5 

  Verifying  : 1:libuv-0.10.29-1.el6.x86_64                                                           3/5 

  Verifying  : c-ares19-1.9.1-5.el6.3.x86_64                                                          4/5 

  Verifying  : nodejs-0.10.33-1.el6.x86_64                                                            5/5 

Installed:
  nodejs.x86_64 0:0.10.33-1.el6                                                                           

Dependency Installed:
  c-ares19.x86_64 0:1.9.1-5.el6.3            http-parser.x86_64 0:2.0-4.20121128gitcd01361.el6           
  libuv.x86_64 1:0.10.29-1.el6               v8.x86_64 1:3.14.5.10-14.el6                                

Complete!
[root@Test tmp]#

{% endhighlight %}

* 6.生成blog

{% highlight bash %}
[root@Test tmp]# jekyll  new myblog
New jekyll site installed in /root/tmp/myblog. 
[root@Test tmp]# 
{% endhighlight %}


* 7.运行jekyll server

{% highlight bash %}
[root@Test tmp]# cd myblog/
[root@Test myblog]# jekyll  server
Configuration file: /root/tmp/myblog/_config.yml
            Source: /root/tmp/myblog
       Destination: /root/tmp/myblog/_site
      Generating... 
                    done.
 Auto-regeneration: enabled for '/root/tmp/myblog'
Configuration file: /root/tmp/myblog/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
{% endhighlight %}

于是使用firefox打开 http://127.0.0.1:4000/ 就会显现

![jekyll_init_page](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/install_jekyll/jekyll_install.png)

下面这个是将我的blog clone到本地后的运行效果

![jekyll_init_page](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/install_jekyll/myblog_local_page.png)

感觉棒棒哒！！

---

#总结

* 1.安装gcc
* 2.安装rvm
* 3.安装ruby
* 4.安装jekyll
* 5.安装nodejs
* 6.生成blog
* 7.运行jekyll server 

---

#注意

* 1.尽量使用rvm,这是一款ruby神器
* 2.使用合适的安装源,别让自己的青春虚度

