---
layout: post
title: "Install Jekyll"
author:  wilmosfang
date: 2018-02-06 14:25:17
image: '/assets/img/'
excerpt: '十分钟构建一个 Jekyll 博客'
main-class: jekyll
color: '#B31917'
tags:
 - jekyll
 - ruby
 - rvm
 - nodejs
 - gem
categories:
 - jekyll
twitter_text: 'simple process of Jekyll installation'
introduction: 'installation method of Jekyll'
---

# 前言

这里演示一下如何用十分钟搭建一个 **[Jekyll][jekyll]** 博客

> **Tip:** 当前最新版本 **Jekyll 1.4.3**

---

# 操作


## 环境

~~~
[root@ci ~]# hostnamectl
   Static hostname: ci
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 8d13a50988cc5c4972347415eddf7d47
           Boot ID: 10b2ba6eee6941b78a7e0b2fa9c42e8c
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@ci ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:b7:37:f1 brd ff:ff:ff:ff:ff:ff
    inet 10.144.126.20/18 brd 10.144.127.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@ci ~]#
~~~


## 安装 gcc

~~~
[root@ci ~]# yum install gcc
Loaded plugins: fastestmirror, langpacks
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
jenkins                                                  | 2.9 kB     00:00     
os                                                       | 3.6 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/5): epel/7/x86_64/group_gz                              | 266 kB   00:00     
(2/5): epel/7/x86_64/updateinfo                            | 880 kB   00:00     
(3/5): extras/7/x86_64/primary_db                          | 166 kB   00:00     
(4/5): epel/7/x86_64/primary_db                            | 6.2 MB   00:00     
(5/5): updates/7/x86_64/primary_db                         | 6.0 MB   00:00     
Determining fastest mirrors
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.8.5-16.el7_4.1 will be installed
--> Processing Dependency: libgomp = 4.8.5-16.el7_4.1 for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: cpp = 4.8.5-16.el7_4.1 for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: libgcc >= 4.8.5-16.el7_4.1 for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: libmpfr.so.4()(64bit) for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Running transaction check
---> Package cpp.x86_64 0:4.8.5-16.el7_4.1 will be installed
---> Package libgcc.i686 0:4.8.5-11.el7 will be updated
---> Package libgcc.x86_64 0:4.8.5-11.el7 will be updated
---> Package libgcc.i686 0:4.8.5-16.el7_4.1 will be an update
---> Package libgcc.x86_64 0:4.8.5-16.el7_4.1 will be an update
---> Package libgomp.x86_64 0:4.8.5-11.el7 will be updated
---> Package libgomp.x86_64 0:4.8.5-16.el7_4.1 will be an update
---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
---> Package mpfr.x86_64 0:3.1.1-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package         Arch           Version                   Repository       Size
================================================================================
Installing:
 gcc             x86_64         4.8.5-16.el7_4.1          updates          16 M
Installing for dependencies:
 cpp             x86_64         4.8.5-16.el7_4.1          updates         5.9 M
 libmpc          x86_64         1.0.1-3.el7               os               51 k
 mpfr            x86_64         3.1.1-4.el7               os              203 k
Updating for dependencies:
 libgcc          i686           4.8.5-16.el7_4.1          updates         106 k
 libgcc          x86_64         4.8.5-16.el7_4.1          updates          98 k
 libgomp         x86_64         4.8.5-16.el7_4.1          updates         154 k

Transaction Summary
================================================================================
Install  1 Package  (+3 Dependent packages)
Upgrade             ( 3 Dependent packages)

Total download size: 23 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/7): cpp-4.8.5-16.el7_4.1.x86_64.rpm                     | 5.9 MB   00:00     
(2/7): libgcc-4.8.5-16.el7_4.1.i686.rpm                    | 106 kB   00:00     
(3/7): libgcc-4.8.5-16.el7_4.1.x86_64.rpm                  |  98 kB   00:00     
(4/7): libgomp-4.8.5-16.el7_4.1.x86_64.rpm                 | 154 kB   00:00     
(5/7): libmpc-1.0.1-3.el7.x86_64.rpm                       |  51 kB   00:00     
(6/7): mpfr-3.1.1-4.el7.x86_64.rpm                         | 203 kB   00:00     
(7/7): gcc-4.8.5-16.el7_4.1.x86_64.rpm                     |  16 MB   00:00     
--------------------------------------------------------------------------------
Total                                               34 MB/s |  23 MB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : mpfr-3.1.1-4.el7.x86_64                                     1/10
  Installing : libmpc-1.0.1-3.el7.x86_64                                   2/10
  Installing : cpp-4.8.5-16.el7_4.1.x86_64                                 3/10
  Updating   : libgcc-4.8.5-16.el7_4.1.x86_64                              4/10
  Updating   : libgomp-4.8.5-16.el7_4.1.x86_64                             5/10
  Installing : gcc-4.8.5-16.el7_4.1.x86_64                                 6/10
  Updating   : libgcc-4.8.5-16.el7_4.1.i686                                7/10
  Cleanup    : libgcc-4.8.5-11.el7                                         8/10
  Cleanup    : libgcc-4.8.5-11.el7                                         9/10
  Cleanup    : libgomp-4.8.5-11.el7.x86_64                                10/10
  Verifying  : cpp-4.8.5-16.el7_4.1.x86_64                                 1/10
  Verifying  : libgcc-4.8.5-16.el7_4.1.i686                                2/10
  Verifying  : mpfr-3.1.1-4.el7.x86_64                                     3/10
  Verifying  : libgomp-4.8.5-16.el7_4.1.x86_64                             4/10
  Verifying  : libgcc-4.8.5-16.el7_4.1.x86_64                              5/10
  Verifying  : libmpc-1.0.1-3.el7.x86_64                                   6/10
  Verifying  : gcc-4.8.5-16.el7_4.1.x86_64                                 7/10
  Verifying  : libgcc-4.8.5-11.el7.x86_64                                  8/10
  Verifying  : libgomp-4.8.5-11.el7.x86_64                                 9/10
  Verifying  : libgcc-4.8.5-11.el7.i686                                   10/10

Installed:
  gcc.x86_64 0:4.8.5-16.el7_4.1                                                 

Dependency Installed:
  cpp.x86_64 0:4.8.5-16.el7_4.1           libmpc.x86_64 0:1.0.1-3.el7          
  mpfr.x86_64 0:3.1.1-4.el7              

Dependency Updated:
  libgcc.i686 0:4.8.5-16.el7_4.1          libgcc.x86_64 0:4.8.5-16.el7_4.1      
  libgomp.x86_64 0:4.8.5-16.el7_4.1      

Complete!
[root@ci ~]#
~~~


## 安装 rvm


~~~
[root@ci ~]# rvm -v
-bash: rvm: command not found
[root@ci ~]# curl -L get.rvm.io | bash -s stable
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    330      0 --:--:-- --:--:-- --:--:--   330
100 24090  100 24090    0     0  32957      0 --:--:-- --:--:-- --:--:-- 32957
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: directory `/root/.gnupg' created
gpg: new configuration file `/root/.gnupg/gpg.conf' created
gpg: WARNING: options in `/root/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/root/.gnupg/pubring.gpg' created
gpg: Signature made 2017年09月11日 星期一 04时59分21秒 CST using RSA key ID BF04FF17
gpg: Can't check signature: No public key
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found. Assuming you trust Michal Papis import the mpapis public key (downloading the signatures).

GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.29.3.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

the key can be compared with:

    https://rvm.io/mpapis.asc
    https://keybase.io/mpapis

NOTE: GPG version 2.1.17 have a bug which cause failures during fetching keys from remote server. Please downgrade or upgrade to newer version (if available) or use the second method described above.

[root@ci ~]# gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: requesting key D39DC0E3 from hkp server keys.gnupg.net
gpgkeys: key 409B6B1796C275462A1703113804BB82D39DC0E3 can't be retrieved
gpg: no valid OpenPGP data found.
gpg: Total number processed: 0
[root@ci ~]# echo $?
2
[root@ci ~]# gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: requesting key D39DC0E3 from hkp server keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[root@ci ~]# echo $?
0
[root@ci ~]#  curl -L get.rvm.io | bash -s stable  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    342      0 --:--:-- --:--:-- --:--:--   342
100 24090  100 24090    0     0  34290      0 --:--:-- --:--:-- --:--:-- 34290
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: Signature made 2017年09月11日 星期一 04时59分21秒 CST using RSA key ID BF04FF17
gpg: Good signature from "Michal Papis (RVM signing) <mpapis@gmail.com>"
gpg:                 aka "Michal Papis <michal.papis@toptal.com>"
gpg:                 aka "[jpeg image of size 5015]"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
     Subkey fingerprint: 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/rvm/archives/rvm-1.29.3.tgz'
Creating group 'rvm'

Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
[root@ci ~]# echo $?
0
[root@ci ~]# rvm -v
-bash: rvm: command not found
[root@ci ~]# su -
Last login: 二 2月  6 21:08:01 CST 2018 from 112.211.20.186 on pts/1
Last failed login: 二 2月  6 21:10:40 CST 2018 from 58.218.198.161 on ssh:notty
There were 19 failed login attempts since the last successful login.
[root@ci ~]# rvm -v
rvm 1.29.3 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
[root@ci ~]#
~~~

## 安装 ruby

~~~
[root@ci ~]# rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.7]
[ruby-]2.3[.4]
[ruby-]2.4[.1]
ruby-head

# for forks use: rvm install ruby-head-<name> --url https://github.com/github/ruby.git --branch 2.2

# JRuby
jruby-1.6[.8]
jruby-1.7[.27]
jruby[-9.1.13.0]
jruby-head

# Rubinius
rbx-1[.4.3]
rbx-2.3[.0]
rbx-2.4[.1]
rbx-2[.5.8]
rbx-3[.84]
rbx-head

# Opal
opal

# Minimalistic ruby implementation - ISO 30170:2012
mruby-1.0.0
mruby-1.1.0
mruby-1.2.0
mruby-1[.3.0]
mruby[-head]

# Ruby Enterprise Edition
ree-1.8.6
ree[-1.8.7][-2012.02]

# Topaz
topaz

# MagLev
maglev[-head]
maglev-1.0.0

# Mac OS X Snow Leopard Or Newer
macruby-0.10
macruby-0.11
macruby[-0.12]
macruby-nightly
macruby-head

# IronRuby
ironruby[-1.1.3]
ironruby-head
[root@ci ~]# rvm install ruby
Searching for binary rubies, this might take some time.
Found remote file https://rvm_io.global.ssl.fastly.net/binaries/centos/7/x86_64/ruby-2.4.1.tar.bz2
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: patch, autoconf, automake, bison, gcc-c++, libffi-devel, libtool, patch, readline-devel, sqlite-devel, zlib-devel, libyaml-devel, openssl-devel.....................................
Requirements installation successful.
ruby-2.4.1 - #configure
ruby-2.4.1 - #download
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14.1M  100 14.1M    0     0  2105k      0  0:00:06  0:00:06 --:--:-- 2448k
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.4.1 - #validate archive
ruby-2.4.1 - #extract
ruby-2.4.1 - #validate binary
ruby-2.4.1 - #setup
ruby-2.4.1 - #gemset created /usr/local/rvm/gems/ruby-2.4.1@global
ruby-2.4.1 - #importing gemset /usr/local/rvm/gemsets/global.gems...............
ruby-2.4.1 - #generating global wrappers........
ruby-2.4.1 - #gemset created /usr/local/rvm/gems/ruby-2.4.1
ruby-2.4.1 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.4.1 - #generating default wrappers........
[root@ci ~]# echo $?
0
[root@ci ~]#
~~~


## 替换 gem 源

~~~
[root@ci ~]# gem -v
2.6.14
[root@ci ~]# gem source -l
*** CURRENT SOURCES ***

https://rubygems.org/
[root@ci ~]# gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/  
https://gems.ruby-china.org/ added to sources
https://rubygems.org/ removed from sources
[root@ci ~]# gem source -l
*** CURRENT SOURCES ***

https://gems.ruby-china.org/
[root@ci ~]#
~~~


## 安装 jekyll

~~~
[root@ci ~]# gem install jekyll
Fetching: public_suffix-3.0.1.gem (100%)
Successfully installed public_suffix-3.0.1
Fetching: addressable-2.5.2.gem (100%)
Successfully installed addressable-2.5.2
Fetching: colorator-1.1.0.gem (100%)
Successfully installed colorator-1.1.0
Fetching: http_parser.rb-0.6.0.gem (100%)
Building native extensions.  This could take a while...
Successfully installed http_parser.rb-0.6.0
Fetching: eventmachine-1.2.5.gem (100%)
Building native extensions.  This could take a while...
Successfully installed eventmachine-1.2.5
Fetching: em-websocket-0.5.1.gem (100%)
Successfully installed em-websocket-0.5.1
Fetching: concurrent-ruby-1.0.5.gem (100%)
Successfully installed concurrent-ruby-1.0.5
Fetching: i18n-0.9.3.gem (100%)
Successfully installed i18n-0.9.3
Fetching: rb-fsevent-0.10.2.gem (100%)
Successfully installed rb-fsevent-0.10.2
Fetching: ffi-1.9.21.gem (100%)
Building native extensions.  This could take a while...
Successfully installed ffi-1.9.21
Fetching: rb-inotify-0.9.10.gem (100%)
Successfully installed rb-inotify-0.9.10
Fetching: sass-listen-4.0.0.gem (100%)
Successfully installed sass-listen-4.0.0
Fetching: sass-3.5.5.gem (100%)
Successfully installed sass-3.5.5
Fetching: jekyll-sass-converter-1.5.2.gem (100%)
Successfully installed jekyll-sass-converter-1.5.2
Fetching: ruby_dep-1.5.0.gem (100%)
Successfully installed ruby_dep-1.5.0
Fetching: listen-3.1.5.gem (100%)
Successfully installed listen-3.1.5
Fetching: jekyll-watch-2.0.0.gem (100%)
Successfully installed jekyll-watch-2.0.0
Fetching: kramdown-1.16.2.gem (100%)
Successfully installed kramdown-1.16.2
Fetching: liquid-4.0.0.gem (100%)
Successfully installed liquid-4.0.0
Fetching: mercenary-0.3.6.gem (100%)
Successfully installed mercenary-0.3.6
Fetching: forwardable-extended-2.6.0.gem (100%)
Successfully installed forwardable-extended-2.6.0
Fetching: pathutil-0.16.1.gem (100%)
Successfully installed pathutil-0.16.1
Fetching: rouge-3.1.1.gem (100%)
Successfully installed rouge-3.1.1
Fetching: safe_yaml-1.0.4.gem (100%)
Successfully installed safe_yaml-1.0.4
Fetching: jekyll-3.7.2.gem (100%)
Successfully installed jekyll-3.7.2
Parsing documentation for public_suffix-3.0.1
Installing ri documentation for public_suffix-3.0.1
Parsing documentation for addressable-2.5.2
Installing ri documentation for addressable-2.5.2
Parsing documentation for colorator-1.1.0
Installing ri documentation for colorator-1.1.0
Parsing documentation for http_parser.rb-0.6.0
Installing ri documentation for http_parser.rb-0.6.0
Parsing documentation for eventmachine-1.2.5
Installing ri documentation for eventmachine-1.2.5
Parsing documentation for em-websocket-0.5.1
Installing ri documentation for em-websocket-0.5.1
Parsing documentation for concurrent-ruby-1.0.5
Installing ri documentation for concurrent-ruby-1.0.5
Parsing documentation for i18n-0.9.3
Installing ri documentation for i18n-0.9.3
Parsing documentation for rb-fsevent-0.10.2
Installing ri documentation for rb-fsevent-0.10.2
Parsing documentation for ffi-1.9.21
Installing ri documentation for ffi-1.9.21
Parsing documentation for rb-inotify-0.9.10
Installing ri documentation for rb-inotify-0.9.10
Parsing documentation for sass-listen-4.0.0
Installing ri documentation for sass-listen-4.0.0
Parsing documentation for sass-3.5.5
Installing ri documentation for sass-3.5.5
Parsing documentation for jekyll-sass-converter-1.5.2
Installing ri documentation for jekyll-sass-converter-1.5.2
Parsing documentation for ruby_dep-1.5.0
Installing ri documentation for ruby_dep-1.5.0
Parsing documentation for listen-3.1.5
Installing ri documentation for listen-3.1.5
Parsing documentation for jekyll-watch-2.0.0
Installing ri documentation for jekyll-watch-2.0.0
Parsing documentation for kramdown-1.16.2
Installing ri documentation for kramdown-1.16.2
Parsing documentation for liquid-4.0.0
Installing ri documentation for liquid-4.0.0
Parsing documentation for mercenary-0.3.6
Installing ri documentation for mercenary-0.3.6
Parsing documentation for forwardable-extended-2.6.0
Installing ri documentation for forwardable-extended-2.6.0
Parsing documentation for pathutil-0.16.1
Installing ri documentation for pathutil-0.16.1
Parsing documentation for rouge-3.1.1
Installing ri documentation for rouge-3.1.1
Parsing documentation for safe_yaml-1.0.4
Installing ri documentation for safe_yaml-1.0.4
Parsing documentation for jekyll-3.7.2
Installing ri documentation for jekyll-3.7.2
Done installing documentation for public_suffix, addressable, colorator, http_parser.rb, eventmachine, em-websocket, concurrent-ruby, i18n, rb-fsevent, ffi, rb-inotify, sass-listen, sass, jekyll-sass-converter, ruby_dep, listen, jekyll-watch, kramdown, liquid, mercenary, forwardable-extended, pathutil, rouge, safe_yaml, jekyll after 34 seconds
25 gems installed
[root@ci ~]# echo $?
0
[root@ci ~]# jekyll --help
jekyll 3.7.2 -- Jekyll is a blog-aware, static site generator in Ruby

Usage:

  jekyll <subcommand> [options]

Options:
        -s, --source [DIR]  Source directory (defaults to ./)
        -d, --destination [DIR]  Destination directory (defaults to ./_site)
            --safe         Safe mode (defaults to false)
        -p, --plugins PLUGINS_DIR1[,PLUGINS_DIR2[,...]]  Plugins directory (defaults to ./_plugins)
            --layouts DIR  Layouts directory (defaults to ./_layouts)
            --profile      Generate a Liquid rendering profile
        -h, --help         Show this message
        -v, --version      Print the name and version
        -t, --trace        Show the full backtrace when an error occurs

Subcommands:
  docs                  
  import                
  build, b              Build your site
  clean                 Clean the site (removes site output and metadata file) without building.
  doctor, hyde          Search site and print specific deprecation warnings
  help                  Show the help message, optionally for a given subcommand.
  new                   Creates a new Jekyll site scaffold in PATH
  new-theme             Creates a new Jekyll theme scaffold
  serve, server, s      Serve your site locally
[root@ci ~]#
~~~


## 安装 nodejs

~~~
[root@ci ~]# yum install epel-release
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch             Version           Repository      Size
================================================================================
Installing:
 epel-release           noarch           7-11              epel            15 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-11.noarch.rpm                               |  15 kB   00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-11.noarch                                     1/1
  Verifying  : epel-release-7-11.noarch                                     1/1

Installed:
  epel-release.noarch 0:7-11                                                    

Complete!
[root@ci ~]# yum -y install nodejs
Loaded plugins: fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package nodejs.x86_64 1:6.12.3-1.el7 will be installed
--> Processing Dependency: npm = 1:3.10.10-1.6.12.3.1.el7 for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: http-parser >= 2.7.0 for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: libuv >= 1:1.9.1 for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: libhttp_parser.so.2()(64bit) for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: libicudata.so.50()(64bit) for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: libicui18n.so.50()(64bit) for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: libicuuc.so.50()(64bit) for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Processing Dependency: libuv.so.1()(64bit) for package: 1:nodejs-6.12.3-1.el7.x86_64
--> Running transaction check
---> Package http-parser.x86_64 0:2.7.1-5.el7_4 will be installed
---> Package libicu.x86_64 0:50.1.2-15.el7 will be installed
---> Package libuv.x86_64 1:1.10.2-1.el7 will be installed
---> Package npm.x86_64 1:3.10.10-1.6.12.3.1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch        Version                        Repository    Size
================================================================================
Installing:
 nodejs           x86_64      1:6.12.3-1.el7                 epel         4.6 M
Installing for dependencies:
 http-parser      x86_64      2.7.1-5.el7_4                  updates       28 k
 libicu           x86_64      50.1.2-15.el7                  os           6.9 M
 libuv            x86_64      1:1.10.2-1.el7                 epel         109 k
 npm              x86_64      1:3.10.10-1.6.12.3.1.el7       epel         2.5 M

Transaction Summary
================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 14 M
Installed size: 50 M
Downloading packages:
(1/5): http-parser-2.7.1-5.el7_4.x86_64.rpm                |  28 kB   00:00     
(2/5): libuv-1.10.2-1.el7.x86_64.rpm                       | 109 kB   00:00     
(3/5): npm-3.10.10-1.6.12.3.1.el7.x86_64.rpm               | 2.5 MB   00:00     
(4/5): nodejs-6.12.3-1.el7.x86_64.rpm                      | 4.6 MB   00:00     
(5/5): libicu-50.1.2-15.el7.x86_64.rpm                     | 6.9 MB   00:00     
--------------------------------------------------------------------------------
Total                                               27 MB/s |  14 MB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : http-parser-2.7.1-5.el7_4.x86_64                             1/5
  Installing : libicu-50.1.2-15.el7.x86_64                                  2/5
  Installing : 1:libuv-1.10.2-1.el7.x86_64                                  3/5
  Installing : 1:npm-3.10.10-1.6.12.3.1.el7.x86_64                          4/5
  Installing : 1:nodejs-6.12.3-1.el7.x86_64                                 5/5
  Verifying  : 1:libuv-1.10.2-1.el7.x86_64                                  1/5
  Verifying  : 1:npm-3.10.10-1.6.12.3.1.el7.x86_64                          2/5
  Verifying  : libicu-50.1.2-15.el7.x86_64                                  3/5
  Verifying  : http-parser-2.7.1-5.el7_4.x86_64                             4/5
  Verifying  : 1:nodejs-6.12.3-1.el7.x86_64                                 5/5

Installed:
  nodejs.x86_64 1:6.12.3-1.el7                                                  

Dependency Installed:
  http-parser.x86_64 0:2.7.1-5.el7_4     libicu.x86_64 0:50.1.2-15.el7          
  libuv.x86_64 1:1.10.2-1.el7            npm.x86_64 1:3.10.10-1.6.12.3.1.el7    

Complete!
[root@ci ~]#
~~~


## 创建 alias

~~~
[root@ci ~]# vim ~/.bashrc
[root@ci ~]# source ~/.bashrc
[root@ci ~]# alias runblog
alias runblog='jekyll server  --host 0.0.0.0 --port 80'
[root@ci ~]#
~~~

## 安装 jekyll-paginate

~~~
[root@ci ~]# gem install  jekyll-paginate
Fetching: jekyll-paginate-1.1.0.gem (100%)
Successfully installed jekyll-paginate-1.1.0
Parsing documentation for jekyll-paginate-1.1.0
Installing ri documentation for jekyll-paginate-1.1.0
Done installing documentation for jekyll-paginate after 0 seconds
1 gem installed
[root@ci ~]#
~~~

## clone 博客

~~~
[git@ci ~]$ mkdir git
[git@ci ~]$ cd git/
[git@ci git]$ ls
[git@ci git]$ git clone https://github.com/wilmosfang/biscuits.git
Cloning into 'biscuits'...
remote: Counting objects: 520, done.
remote: Compressing objects: 100% (210/210), done.
remote: Total 520 (delta 142), reused 234 (delta 80), pack-reused 220
Receiving objects: 100% (520/520), 6.64 MiB | 2.06 MiB/s, done.
Resolving deltas: 100% (223/223), done.
[git@ci git]$ echo $?
0
[git@ci git]$ du -sh biscuits/
16M	biscuits/
[git@ci git]$ cd biscuits/
[git@ci biscuits]$ ls
about.html     CNAME        gulpfile.js  _layouts      robots.txt   src
assets         _config.yml  _includes    package.json  search.json  tags.html
category       favicon.ico  index.html   _posts        series.html
category.html  feed.xml     initpost.sh  README.md     sitemap.xml
[git@ci biscuits]$ git pull
Already up-to-date.
[git@ci biscuits]$
~~~


## 运行博客

>如下是在本地执行的效果

~~~
[root@much biscuits]# runblog
Configuration file: /home/bolo/git/biscuits/_config.yml
            Source: /home/bolo/git/biscuits
       Destination: /home/bolo/git/biscuits/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.462 seconds.
 Auto-regeneration: enabled for '/home/bolo/git/biscuits'
    Server address: http://0.0.0.0:80/
  Server running... press ctrl-c to stop.
...
...
...
~~~

紧接着打开防火墙，就可以对外提供访问了


---

# 总结

整个过程行云流水，一气呵成(因为我已经做过四次类似搭建了^_^)


* TOC
{:toc}


---

[jekyll]:https://www.jekyll.com.cn/
