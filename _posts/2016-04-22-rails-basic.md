---
layout: post
title:  Ruby on Rails 基础
author: wilmosfang
categories:   ruby
tags:   ruby rvm rails
wc: 1196  3890 44402 
excerpt:  RVM 的安装，ruby版本查看，可用版本查看，ruby的安装，Rails的安装，环境，安装源的替换，创建Rails应用，Rails的目录结构，启停服务，对外服务的配置，服务的访问 
comments: true
---



# 前言

**[Rails][rubyonrails]** 是使用 Ruby 语言编写的网页程序开发框架

通过为开发者提供常用组件，来简化网页程序的开发

> **Tip:** 类似于 python 的 Django ，perl 的 Dancer

Rails 框架有自己的指导思想：

* 不重复造轮子(DRY) 

>Don't Repeat Yourself: DRY is a principle of software development which states that "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system." By not writing the same information over and over again, our code is more maintainable, more extensible, and less buggy

* 约定优于配置 

>Convention Over Configuration: Rails has opinions about the best way to do many things in a web application, and defaults to this set of conventions, rather than require that you specify every minutiae through endless configuration files

这两条编码哲学可以算是历代猴子们的智慧结晶，核心目标只有一个，最大化的减少代码规模，明确核心逻辑，而这样的好处是多多的(编码效率高，Debug也快)

DRY 自不用说，人生苦短，我们要站在巨人的肩膀上攀爬，不要把有限的生命浪费在人家已经反复踩过的坑里

配置如果不在代码内部消化，必然要在外面申明，而配置复杂到一定程度后，本身就已经成为了一门具备独立语法的体系，逻辑不在代码里就在配置里，逻辑是守恒的

这里分享一下 **[Rails][rubyonrails]** 的相关基础，详细可以参考 **[官方文档][getting_started]** 和 Ruby China 的 **[Rails 入门][getting_started_cn]** 


> **Tip:** 当前的最新版本为 **Rails 5.0.0.beta3** 发布于 February 27, 2016 4:00 pm


---


# 概要

* TOC
{:toc}


---

## 环境

~~~
[root@h202 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h202 ~]# uname -a 
Linux h202 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h202 ~]# 
~~~


---

## RVM

**[RVM (Ruby Version Manager)][rvm]** 是一个 CLI 工具，可以用来对 ruby 的多个版本进行安装，隔离和管理

>RVM is a command-line tool which allows you to easily install, manage, and work with multiple ruby environments from interpreters to sets of gems

是玩 ruby 不可多得的好工具

---

### 安装RVM

~~~
[root@h202 ruby]# gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: requesting key D39DC0E3 from hkp server keys.gnupg.net
gpg: key D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[root@h202 ruby]# echo $?
0
[root@h202 ruby]# curl -sSL https://get.rvm.io | bash -s stable
Downloading https://github.com/rvm/rvm/archive/1.27.0.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.27.0/1.27.0.tar.gz.asc
curl: (35) SSL connect error

Could not download 'https://github.com/rvm/rvm/releases/download/1.27.0/1.27.0.tar.gz.asc'.
  curl returned status '35'.

Creating group 'rvm'

Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open shell windows, in rare cases you need to reopen all shell windows.

# Administrator,
#
#   Thank you for using RVM!
#   We sincerely hope that RVM helps to make your life easier and more enjoyable!!!
#
# ~Wayne, Michal & team.

In case of problems: https://rvm.io/help and https://twitter.com/rvm_io
[root@h202 ruby]# echo $?
0
[root@h202 ruby]# 
~~~

---

### 查看可用ruby版本

~~~
[root@h202 ruby]# rvm list known
-bash: rvm: command not found
[root@h202 ruby]# su - root 
[root@h202 ~]# cd ruby/
[root@h202 ruby]# rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.8]
[ruby-]2.2[.4]
[ruby-]2.3[.0]
[ruby-]2.2-head
ruby-head

# for forks use: rvm install ruby-head-<name> --url https://github.com/github/ruby.git --branch 2.2

# JRuby
jruby-1.6[.8]
jruby-1.7[.23]
jruby[-9.0.5.0]
jruby-head

# Rubinius
rbx-1[.4.3]
rbx-2.3[.0]
rbx-2.4[.1]
rbx[-2.5.8]
rbx-head

# Opal
opal

# Minimalistic ruby implementation - ISO 30170:2012
mruby[-head]

# Ruby Enterprise Edition
ree-1.8.6
ree[-1.8.7][-2012.02]

# GoRuby
goruby

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
[root@h202 ruby]# 
~~~

---

### 查看本地的ruby

~~~
[root@h202 ruby]# rvm list

rvm rubies


# No rvm rubies installed yet. Try 'rvm help install'.

[root@h202 ruby]#
~~~

---

### 安装ruby

接上版本号就可以自动安装指定版本的ruby

~~~
[root@h202 ruby]# rvm install 2.3
Searching for binary rubies, this might take some time.
No binary rubies available for: centos/6/x86_64/ruby-2.3.0.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: libyaml-devel, autoconf, gcc-c++, readline-devel, zlib-devel, libffi-devel, openssl-devel, automake, libtool, bison, sqlite-devel.............................
Requirements installation successful.
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-2.3.0, this may take a while depending on your cpu(s)...
ruby-2.3.0 - #downloading ruby-2.3.0, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13.5M  100 13.5M    0     0  97053      0  0:02:26  0:02:26 --:--:-- 75395
ruby-2.3.0 - #extracting ruby-2.3.0 to /usr/local/rvm/src/ruby-2.3.0....
ruby-2.3.0 - #configuring..........................................................
ruby-2.3.0 - #post-configuration..
ruby-2.3.0 - #compiling.....................................................................................
ruby-2.3.0 - #installing...........................
ruby-2.3.0 - #making binaries executable..
Installed rubygems 2.5.1 is newer than 2.4.8 provided with installed ruby, skipping installation, use --force to force installation.
ruby-2.3.0 - #gemset created /usr/local/rvm/gems/ruby-2.3.0@global
ruby-2.3.0 - #importing gemset /usr/local/rvm/gemsets/global.gems................................................
ruby-2.3.0 - #generating global wrappers........
ruby-2.3.0 - #gemset created /usr/local/rvm/gems/ruby-2.3.0
ruby-2.3.0 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.3.0 - #generating default wrappers........
ruby-2.3.0 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.3.0 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
[root@h202 ruby]# echo $?
0
[root@h202 ruby]# rvm list

rvm rubies

=* ruby-2.3.0 [ x86_64 ]

# => - current
# =* - current && default
#  * - default

[root@h202 ruby]# 
[root@h202 ruby]# ruby -v 
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
[root@h202 ruby]# 
~~~

---

## 安装Rails


### 检查环境

检查以下三个软件，确保已经安装

~~~
[root@h202 ruby]# ruby -v 
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
[root@h202 ruby]# gem -v 
2.5.1
[root@h202 ruby]# sqlite3 --version
3.6.20
[root@h202 ruby]#
~~~

---

### 替换安装源

如果不替换源，会很慢，或者根本没法获取包，因为有墙

~~~
[root@h202 ruby]# gem source -l
*** CURRENT SOURCES ***

https://rubygems.org/
[root@h202 ruby]# time gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
https://gems.ruby-china.org/ added to sources
https://rubygems.org/ removed from sources

real	0m31.666s
user	0m1.624s
sys	0m4.582s
[root@h202 ruby]# gem source -l
*** CURRENT SOURCES ***

https://gems.ruby-china.org/
[root@h202 ruby]# 
~~~

---

### 安装 Rails



~~~
[root@h202 ruby]# gem install rails
Fetching: rack-1.6.4.gem (100%)
Successfully installed rack-1.6.4
Fetching: concurrent-ruby-1.0.1.gem (100%)
Successfully installed concurrent-ruby-1.0.1
Fetching: sprockets-3.6.0.gem (100%)
Successfully installed sprockets-3.6.0
Fetching: thread_safe-0.3.5.gem (100%)
Successfully installed thread_safe-0.3.5
Fetching: tzinfo-1.2.2.gem (100%)
Successfully installed tzinfo-1.2.2
Fetching: i18n-0.7.0.gem (100%)
Successfully installed i18n-0.7.0
Fetching: activesupport-4.2.6.gem (100%)
Successfully installed activesupport-4.2.6
Fetching: mini_portile2-2.0.0.gem (100%)
Successfully installed mini_portile2-2.0.0
Fetching: nokogiri-1.6.7.2.gem (100%)
Building native extensions.  This could take a while...
Successfully installed nokogiri-1.6.7.2
Fetching: loofah-2.0.3.gem (100%)
Successfully installed loofah-2.0.3
Fetching: rails-html-sanitizer-1.0.3.gem (100%)
Successfully installed rails-html-sanitizer-1.0.3
Fetching: rails-deprecated_sanitizer-1.0.3.gem (100%)
Successfully installed rails-deprecated_sanitizer-1.0.3
Fetching: rails-dom-testing-1.0.7.gem (100%)
Successfully installed rails-dom-testing-1.0.7
Fetching: rack-test-0.6.3.gem (100%)
Successfully installed rack-test-0.6.3
Fetching: erubis-2.7.0.gem (100%)
Successfully installed erubis-2.7.0
Fetching: builder-3.2.2.gem (100%)
Successfully installed builder-3.2.2
Fetching: actionview-4.2.6.gem (100%)
Successfully installed actionview-4.2.6
Fetching: actionpack-4.2.6.gem (100%)
Successfully installed actionpack-4.2.6
Fetching: sprockets-rails-3.0.4.gem (100%)
Successfully installed sprockets-rails-3.0.4
Fetching: thor-0.19.1.gem (100%)
Successfully installed thor-0.19.1
Fetching: railties-4.2.6.gem (100%)
Successfully installed railties-4.2.6
Fetching: bundler-1.11.2.gem (100%)
Successfully installed bundler-1.11.2
Fetching: arel-6.0.3.gem (100%)
Successfully installed arel-6.0.3
Fetching: activemodel-4.2.6.gem (100%)
Successfully installed activemodel-4.2.6
Fetching: activerecord-4.2.6.gem (100%)
Successfully installed activerecord-4.2.6
Fetching: globalid-0.3.6.gem (100%)
Successfully installed globalid-0.3.6
Fetching: activejob-4.2.6.gem (100%)
Successfully installed activejob-4.2.6
Fetching: mime-types-data-3.2016.0221.gem (100%)
Successfully installed mime-types-data-3.2016.0221
Fetching: mime-types-3.0.gem (100%)
Successfully installed mime-types-3.0
Fetching: mail-2.6.4.gem (100%)
Successfully installed mail-2.6.4
Fetching: actionmailer-4.2.6.gem (100%)
Successfully installed actionmailer-4.2.6
Fetching: rails-4.2.6.gem (100%)
Successfully installed rails-4.2.6
Parsing documentation for rack-1.6.4
Installing ri documentation for rack-1.6.4
Parsing documentation for concurrent-ruby-1.0.1
Installing ri documentation for concurrent-ruby-1.0.1
Parsing documentation for sprockets-3.6.0
Installing ri documentation for sprockets-3.6.0
Parsing documentation for thread_safe-0.3.5
Installing ri documentation for thread_safe-0.3.5
Parsing documentation for tzinfo-1.2.2
Installing ri documentation for tzinfo-1.2.2
Parsing documentation for i18n-0.7.0
Installing ri documentation for i18n-0.7.0
Parsing documentation for activesupport-4.2.6
Installing ri documentation for activesupport-4.2.6
Parsing documentation for mini_portile2-2.0.0
Installing ri documentation for mini_portile2-2.0.0
Parsing documentation for nokogiri-1.6.7.2
Installing ri documentation for nokogiri-1.6.7.2
Parsing documentation for loofah-2.0.3
Installing ri documentation for loofah-2.0.3
Parsing documentation for rails-html-sanitizer-1.0.3
Installing ri documentation for rails-html-sanitizer-1.0.3
Parsing documentation for rails-deprecated_sanitizer-1.0.3
Installing ri documentation for rails-deprecated_sanitizer-1.0.3
Parsing documentation for rails-dom-testing-1.0.7
Installing ri documentation for rails-dom-testing-1.0.7
Parsing documentation for rack-test-0.6.3
Installing ri documentation for rack-test-0.6.3
Parsing documentation for erubis-2.7.0
Installing ri documentation for erubis-2.7.0
Parsing documentation for builder-3.2.2
Installing ri documentation for builder-3.2.2
Parsing documentation for actionview-4.2.6
Installing ri documentation for actionview-4.2.6
Parsing documentation for actionpack-4.2.6
Installing ri documentation for actionpack-4.2.6
Parsing documentation for sprockets-rails-3.0.4
Installing ri documentation for sprockets-rails-3.0.4
Parsing documentation for thor-0.19.1
Installing ri documentation for thor-0.19.1
Parsing documentation for railties-4.2.6
Installing ri documentation for railties-4.2.6
Parsing documentation for bundler-1.11.2
Installing ri documentation for bundler-1.11.2
Parsing documentation for arel-6.0.3
Installing ri documentation for arel-6.0.3
Parsing documentation for activemodel-4.2.6
Installing ri documentation for activemodel-4.2.6
Parsing documentation for activerecord-4.2.6
Installing ri documentation for activerecord-4.2.6
Parsing documentation for globalid-0.3.6
Installing ri documentation for globalid-0.3.6
Parsing documentation for activejob-4.2.6
Installing ri documentation for activejob-4.2.6
Parsing documentation for mime-types-data-3.2016.0221
Installing ri documentation for mime-types-data-3.2016.0221
Parsing documentation for mime-types-3.0
Installing ri documentation for mime-types-3.0
Parsing documentation for mail-2.6.4
Installing ri documentation for mail-2.6.4
Parsing documentation for actionmailer-4.2.6
Installing ri documentation for actionmailer-4.2.6
Parsing documentation for rails-4.2.6
Installing ri documentation for rails-4.2.6
Done installing documentation for rack, concurrent-ruby, sprockets, thread_safe, tzinfo, i18n, activesupport, mini_portile2, nokogiri, loofah, rails-html-sanitizer, rails-deprecated_sanitizer, rails-dom-testing, rack-test, erubis, builder, actionview, actionpack, sprockets-rails, thor, railties, bundler, arel, activemodel, activerecord, globalid, activejob, mime-types-data, mime-types, mail, actionmailer, rails after 613 seconds
32 gems installed
[root@h202 ruby]# echo $?
0
[root@h202 ruby]# rails --version
Rails 4.2.6
[root@h202 ruby]# 
~~~

其实就是一捆gems

> **Tip:** 查看本地有哪些 gem ，可以通过如下方式

~~~
[root@h202 ruby]# gem list

*** LOCAL GEMS ***

actionmailer (4.2.6)
actionpack (4.2.6)
actionview (4.2.6)
activejob (4.2.6)
...
...
rdoc (4.2.1)
rvm (1.11.3.9)
sprockets (3.6.0)
sprockets-rails (3.0.4)
test-unit (3.1.5)
thor (0.19.1)
thread_safe (0.3.5)
tzinfo (1.2.2)
[root@h202 ruby]#
~~~


---

## 创建 Rails 程序


我们创建一个叫 blog 的项目

~~~
[root@h202 ruby]# rails new blog
      create  
      create  README.rdoc
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
      create  app
      create  app/assets/javascripts/application.js
      create  app/assets/stylesheets/application.css
      create  app/controllers/application_controller.rb
      create  app/helpers/application_helper.rb
      create  app/views/layouts/application.html.erb
      create  app/assets/images/.keep
      create  app/mailers/.keep
      create  app/models/.keep
      create  app/controllers/concerns/.keep
      create  app/models/concerns/.keep
      create  bin
      create  bin/bundle
      create  bin/rails
      create  bin/rake
      create  bin/setup
      create  config
      create  config/routes.rb
      create  config/application.rb
      create  config/environment.rb
      create  config/secrets.yml
      create  config/environments
      create  config/environments/development.rb
      create  config/environments/production.rb
      create  config/environments/test.rb
      create  config/initializers
      create  config/initializers/assets.rb
      create  config/initializers/backtrace_silencers.rb
      create  config/initializers/cookies_serializer.rb
      create  config/initializers/filter_parameter_logging.rb
      create  config/initializers/inflections.rb
      create  config/initializers/mime_types.rb
      create  config/initializers/session_store.rb
      create  config/initializers/wrap_parameters.rb
      create  config/locales
      create  config/locales/en.yml
      create  config/boot.rb
      create  config/database.yml
      create  db
      create  db/seeds.rb
      create  lib
      create  lib/tasks
      create  lib/tasks/.keep
      create  lib/assets
      create  lib/assets/.keep
      create  log
      create  log/.keep
      create  public
      create  public/404.html
      create  public/422.html
      create  public/500.html
      create  public/favicon.ico
      create  public/robots.txt
      create  test/fixtures
      create  test/fixtures/.keep
      create  test/controllers
      create  test/controllers/.keep
      create  test/mailers
      create  test/mailers/.keep
      create  test/models
      create  test/models/.keep
      create  test/helpers
      create  test/helpers/.keep
      create  test/integration
      create  test/integration/.keep
      create  test/test_helper.rb
      create  tmp/cache
      create  tmp/cache/assets
      create  vendor/assets/javascripts
      create  vendor/assets/javascripts/.keep
      create  vendor/assets/stylesheets
      create  vendor/assets/stylesheets/.keep
         run  bundle install
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle as root will break this application
for all non-root users on this machine.
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies............

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/rake-11.1.2.gem)
Using i18n 0.7.0
Using json 1.8.3

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/minitest-5.8.4.gem)
Using thread_safe 0.3.5
Using builder 3.2.2
Using erubis 2.7.0
Using mini_portile2 2.0.0
Using rack 1.6.4
Using mime-types-data 3.2016.0221
Using arel 6.0.3

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/debug_inspector-0.0.2.gem)
Using bundler 1.11.2

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/byebug-8.2.4.gem)

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/coffee-script-source-1.10.0.gem)

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/execjs-2.6.0.gem)
Using thor 0.19.1
Using concurrent-ruby 1.0.1

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/multi_json-1.11.2.gem)

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/sass-3.4.22.gem)

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/tilt-2.0.2.gem)

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/spring-1.7.1.gem)

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/sqlite3-1.3.11.gem)
An error occurred while installing rake (11.1.2), and Bundler cannot continue.
Make sure that `gem install rake -v '11.1.2'` succeeds before bundling.
         run  bundle exec spring binstub --all
bundler: command not found: spring
Install missing gem executables with `bundle install`
[root@h202 ruby]# echo $?
0
[root@h202 ruby]# ls
blog
[root@h202 ruby]#
~~~

没有创建成功，但是反馈结果却是成功 (说明这是一批命令，最后一个反馈结果正常)，并且生成一个文件目录

从输出可以看到 **`Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/rake-11.1.2.gem)`**

原因是 bundle 过程中与 gem 安装源连接产生了问题

解决办法是替换成稳定可用且可达的源


~~~
[root@h202 ruby]# ls
blog
[root@h202 ruby]# cd blog/
[root@h202 blog]# ls
app  bin  config  config.ru  db  Gemfile  lib  log  public  Rakefile  README.rdoc  test  tmp  vendor
[root@h202 blog]# head -n 3 Gemfile 
source 'https://rubygems.org'


[root@h202 blog]# vim Gemfile 
[root@h202 blog]# head -n 3 Gemfile
#source 'https://rubygems.org'
source 'https://gems.ruby-china.org/'

[root@h202 blog]# 
~~~


根据提示再次尝试安装


~~~
[root@h202 blog]# gem install rake -v '11.1.2'
Fetching: rake-11.1.2.gem (100%)
Successfully installed rake-11.1.2
Parsing documentation for rake-11.1.2
Installing ri documentation for rake-11.1.2
Done installing documentation for rake after 1 seconds
1 gem installed
[root@h202 blog]# bundle install
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle as root will break this application
for all non-root users on this machine.
Fetching gem metadata from https://gems.ruby-china.org/...........
Fetching version metadata from https://gems.ruby-china.org/...
Fetching dependency metadata from https://gems.ruby-china.org/..
Resolving dependencies..........
Using rake 11.1.2
Using i18n 0.7.0
Using json 1.8.3
Installing minitest 5.8.4
Using thread_safe 0.3.5
Using builder 3.2.2
Using erubis 2.7.0
Using mini_portile2 2.0.0
Using rack 1.6.4
Using mime-types-data 3.2016.0221
Using arel 6.0.3
Installing debug_inspector 0.0.2 with native extensions
Using bundler 1.11.2
Installing byebug 8.2.4 with native extensions
Installing coffee-script-source 1.10.0
Installing execjs 2.6.0
Using thor 0.19.1
Using concurrent-ruby 1.0.1
Installing multi_json 1.11.2
Installing sass 3.4.22
Installing tilt 2.0.2
Installing spring 1.7.1
Installing sqlite3 1.3.11 with native extensions
Installing rdoc 4.2.2
Using tzinfo 1.2.2
Using nokogiri 1.6.7.2
Using rack-test 0.6.3
Using mime-types 3.0
Installing binding_of_caller 0.7.2 with native extensions
Installing coffee-script 2.4.1
Installing uglifier 3.0.0
Using sprockets 3.6.0
Installing sdoc 0.4.1
Using activesupport 4.2.6
Using loofah 2.0.3
Using mail 2.6.4
Using rails-deprecated_sanitizer 1.0.3
Using globalid 0.3.6
Using activemodel 4.2.6
Installing jbuilder 2.4.1
Using rails-html-sanitizer 1.0.3
Using rails-dom-testing 1.0.7
Using activejob 4.2.6
Using activerecord 4.2.6
Using actionview 4.2.6
Using actionpack 4.2.6
Using actionmailer 4.2.6
Using railties 4.2.6
Using sprockets-rails 3.0.4
Installing coffee-rails 4.1.1
Installing jquery-rails 4.1.1
Using rails 4.2.6
Installing sass-rails 5.0.4
Installing web-console 2.3.0
Installing turbolinks 2.5.3
Bundle complete! 12 Gemfile dependencies, 55 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
Post-install message from rdoc:
Depending on your version of ruby, you may need to install ruby rdoc/ri data:

<= 1.8.6 : unsupported
 = 1.8.7 : gem install rdoc-data; rdoc-data --install
 = 1.9.1 : gem install rdoc-data; rdoc-data --install
>= 1.9.2 : nothing to do! Yay!
[root@h202 blog]# echo $?
0
[root@h202 blog]# 
~~~

> **Note:** bundle install 过程中有一个警告，让我们不要使用 root，这样会让其它用户无法操作此应用，其实还有一定安全隐患，这里为图方便，只为了解功能就不去讲究这些了，生产环境下要非常注意
>
> >Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle as root will break this application
for all non-root users on this machine.


---

## Rails 的目录结构


~~~
[root@h202 blog]# tree
.
├── app
│   ├── assets
│   │   ├── images
│   │   ├── javascripts
│   │   │   └── application.js
│   │   └── stylesheets
│   │       └── application.css
│   ├── controllers
│   │   ├── application_controller.rb
│   │   └── concerns
│   ├── helpers
│   │   └── application_helper.rb
│   ├── mailers
│   ├── models
│   │   └── concerns
│   └── views
│       └── layouts
│           └── application.html.erb
├── bin
│   ├── bundle
│   ├── rails
│   ├── rake
│   └── setup
├── config
│   ├── application.rb
│   ├── boot.rb
│   ├── database.yml
│   ├── environment.rb
│   ├── environments
│   │   ├── development.rb
│   │   ├── production.rb
│   │   └── test.rb
│   ├── initializers
│   │   ├── assets.rb
│   │   ├── backtrace_silencers.rb
│   │   ├── cookies_serializer.rb
│   │   ├── filter_parameter_logging.rb
│   │   ├── inflections.rb
│   │   ├── mime_types.rb
│   │   ├── session_store.rb
│   │   └── wrap_parameters.rb
│   ├── locales
│   │   └── en.yml
│   ├── routes.rb
│   └── secrets.yml
├── config.ru
├── db
│   └── seeds.rb
├── Gemfile
├── Gemfile.lock
├── lib
│   ├── assets
│   └── tasks
├── log
├── public
│   ├── 404.html
│   ├── 422.html
│   ├── 500.html
│   ├── favicon.ico
│   └── robots.txt
├── Rakefile
├── README.rdoc
├── test
│   ├── controllers
│   ├── fixtures
│   ├── helpers
│   ├── integration
│   ├── mailers
│   ├── models
│   └── test_helper.rb
├── tmp
│   └── cache
│       └── assets
└── vendor
    └── assets
        ├── javascripts
        └── stylesheets

38 directories, 39 files
[root@h202 blog]# 
~~~



文件/文件夹 | 作用
-------- | ---
app/	|存放程序的控制器、模型、视图、帮助方法、邮件和静态资源文件。本文主要关注的是这个文件夹。
bin/	|存放运行程序的 rails 脚本，以及其他用来部署或运行程序的脚本。
config/	|设置程序的路由，数据库等。详情参阅 **[“设置 Rails 程序”][config]** 一文。
config.ru	|基于 Rack 服务器的程序设置，用来启动程序。
db/	|存放当前数据库的模式，以及数据库迁移文件。
Gemfile, Gemfile.lock	|这两个文件用来指定程序所需的 gem 依赖件，用于 Bundler gem。关于 Bundler 的详细介绍，请访问 **[Bundler 官网][bundler]** 。
lib/	|程序的扩展模块。
log/	|程序的日志文件。
public/	|唯一对外开放的文件夹，存放静态文件和编译后的资源文件。
Rakefile	|保存并加载可在命令行中执行的任务。任务在 Rails 的各组件中定义。如果想添加自己的任务，不要修改这个文件，把任务保存在 lib/tasks 文件夹中。
README.rdoc	|程序的简单说明。你应该修改这个文件，告诉其他人这个程序的作用，如何安装等。
test/	|单元测试，固件等测试用文件。详情参阅 **[“测试 Rails 程序”][testing]** 一文。
tmp/	|临时文件，例如缓存，PID，会话文件。
vendor/	|存放第三方代码。经常用来放第三方 gem。


---

## 启动服务


使用 **rails server** 尝试启动服务

~~~
[root@h202 blog]# ls
app  bin  config  config.ru  db  Gemfile  Gemfile.lock  lib  log  public  Rakefile  README.rdoc  test  tmp  vendor
[root@h202 blog]# rails server
/usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:80:in `rescue in block (2 levels) in require': There was an error while trying to load the gem 'uglifier'. (Bundler::GemRequireError)
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:76:in `block (2 levels) in require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:72:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:72:in `block in require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:61:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:61:in `require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler.rb:99:in `require'
	from /root/ruby/blog/config/application.rb:7:in `<top (required)>'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:78:in `require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:78:in `block in server'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:75:in `tap'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:75:in `server'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:39:in `run_command!'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands.rb:17:in `<top (required)>'
	from bin/rails:4:in `require'
	from bin/rails:4:in `<main>'
[root@h202 blog]# gem install uglifier
Successfully installed uglifier-3.0.0
Parsing documentation for uglifier-3.0.0
Installing ri documentation for uglifier-3.0.0
Done installing documentation for uglifier after 3 seconds
1 gem installed
[root@h202 blog]# rails server
/usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:80:in `rescue in block (2 levels) in require': There was an error while trying to load the gem 'uglifier'. (Bundler::GemRequireError)
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:76:in `block (2 levels) in require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:72:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:72:in `block in require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:61:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:61:in `require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler.rb:99:in `require'
	from /root/ruby/blog/config/application.rb:7:in `<top (required)>'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:78:in `require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:78:in `block in server'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:75:in `tap'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:75:in `server'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:39:in `run_command!'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands.rb:17:in `<top (required)>'
	from bin/rails:4:in `require'
	from bin/rails:4:in `<main>'
[root@h202 blog]# bundle exec spring binstub --all
* bin/rake: spring inserted
* bin/rails: spring inserted
[root@h202 blog]# bundle install
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle as root will break this application
for all non-root users on this machine.
Using rake 11.1.2
Using i18n 0.7.0
Using json 1.8.3
Using minitest 5.8.4
Using thread_safe 0.3.5
Using builder 3.2.2
Using erubis 2.7.0
Using mini_portile2 2.0.0
Using rack 1.6.4
Using mime-types-data 3.2016.0221
Using arel 6.0.3
Using debug_inspector 0.0.2
Using byebug 8.2.4
Using coffee-script-source 1.10.0
Using execjs 2.6.0
Using thor 0.19.1
Using concurrent-ruby 1.0.1
Using multi_json 1.11.2
Using bundler 1.11.2
Using sass 3.4.22
Using tilt 2.0.2
Using spring 1.7.1
Using sqlite3 1.3.11
Using rdoc 4.2.2
Using tzinfo 1.2.2
Using nokogiri 1.6.7.2
Using rack-test 0.6.3
Using mime-types 3.0
Using binding_of_caller 0.7.2
Using coffee-script 2.4.1
Using uglifier 3.0.0
Using sprockets 3.6.0
Using sdoc 0.4.1
Using activesupport 4.2.6
Using loofah 2.0.3
Using mail 2.6.4
Using rails-deprecated_sanitizer 1.0.3
Using globalid 0.3.6
Using activemodel 4.2.6
Using jbuilder 2.4.1
Using rails-html-sanitizer 1.0.3
Using rails-dom-testing 1.0.7
Using activejob 4.2.6
Using activerecord 4.2.6
Using actionview 4.2.6
Using actionpack 4.2.6
Using actionmailer 4.2.6
Using railties 4.2.6
Using sprockets-rails 3.0.4
Using coffee-rails 4.1.1
Using jquery-rails 4.1.1
Using rails 4.2.6
Using sass-rails 5.0.4
Using web-console 2.3.0
Using turbolinks 2.5.3
Bundle complete! 12 Gemfile dependencies, 55 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
[root@h202 blog]# 
[root@h202 blog]# rails server 
/usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:80:in `rescue in block (2 levels) in require': There was an error while trying to load the gem 'uglifier'. (Bundler::GemRequireError)
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:76:in `block (2 levels) in require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:72:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:72:in `block in require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:61:in `each'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler/runtime.rb:61:in `require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/bundler-1.11.2/lib/bundler.rb:99:in `require'
	from /root/ruby/blog/config/application.rb:7:in `<top (required)>'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:78:in `require'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:78:in `block in server'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:75:in `tap'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:75:in `server'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands/commands_tasks.rb:39:in `run_command!'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/railties-4.2.6/lib/rails/commands.rb:17:in `<top (required)>'
	from /root/ruby/blog/bin/rails:9:in `require'
	from /root/ruby/blog/bin/rails:9:in `<top (required)>'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/lib/spring/client/rails.rb:28:in `load'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/lib/spring/client/rails.rb:28:in `call'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/lib/spring/client/command.rb:7:in `call'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/lib/spring/client.rb:30:in `run'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/bin/spring:49:in `<top (required)>'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/lib/spring/binstub.rb:11:in `load'
	from /usr/local/rvm/gems/ruby-2.3.0/gems/spring-1.7.1/lib/spring/binstub.rb:11:in `<top (required)>'
	from /usr/local/rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /usr/local/rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /root/ruby/blog/bin/spring:13:in `<top (required)>'
	from bin/rails:3:in `load'
	from bin/rails:3:in `<main>'
[root@h202 blog]# 
~~~


报错：**`There was an error while trying to load the gem 'uglifier'. (Bundler::GemRequireError)`**

事实上我们系统里已经有了这个gem


~~~
[root@h202 blog]# bundle list | grep uglifier
  * uglifier (3.0.0)
[root@h202 blog]# 
~~~

那为什么会报错呢，原因是 **[uglifier][uglifier]** 这个包，其实是 JS 的一层包装，它需要 JS的运行环境或者JS的解释器

> **[Uglifier][uglifier]** minifies JavaScript files by wrapping UglifyJS to be accessible in Ruby

解决方法是安装 **NodeJS**

~~~
[root@h202 blog]# yum install nodejs
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Install Process
Repository base is listed more than once in the configuration
Loading mirror speeds from cached hostfile
epel/metalink                                                                                                | 3.8 kB     00:00     
 * base: mirrors.aliyun.com
 * epel: mirrors.opencas.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                                                         | 3.7 kB     00:00     
extras                                                                                                       | 3.4 kB     00:00     
percona-release-noarch                                                                                       | 2.5 kB     00:00     
percona-release-x86_64                                                                                       | 2.5 kB     00:00     
updates                                                                                                      | 3.4 kB     00:00     
zabbix                                                                                                       |  951 B     00:00     
zabbix-non-supported                                                                                         |  951 B     00:00     
Resolving Dependencies
--> Running transaction check
---> Package nodejs.x86_64 0:0.10.42-4.el6 will be installed
--> Processing Dependency: libuv.so.0.10()(64bit) for package: nodejs-0.10.42-4.el6.x86_64
--> Running transaction check
---> Package libuv.x86_64 1:0.10.34-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                      Arch                         Version                                 Repository                  Size
====================================================================================================================================
Installing:
 nodejs                       x86_64                       0.10.42-4.el6                           epel                       2.1 M
Installing for dependencies:
 libuv                        x86_64                       1:0.10.34-1.el6                         epel                        57 k

Transaction Summary
====================================================================================================================================
Install       2 Package(s)

Total download size: 2.1 M
Installed size: 7.2 M
Is this ok [y/N]: y
Downloading Packages:
(1/2): libuv-0.10.34-1.el6.x86_64.rpm                                                                        |  57 kB     00:00     
(2/2): nodejs-0.10.42-4.el6.x86_64.rpm                                                                       | 2.1 MB     00:13     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               147 kB/s | 2.1 MB     00:14     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Importing GPG key 0x0608B895:
 Userid : EPEL (6) <epel@fedoraproject.org>
 Package: epel-release-6-8.noarch (@extras)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : 1:libuv-0.10.34-1.el6.x86_64                                                                                     1/2 
  Installing : nodejs-0.10.42-4.el6.x86_64                                                                                      2/2 
  Verifying  : nodejs-0.10.42-4.el6.x86_64                                                                                      1/2 
  Verifying  : 1:libuv-0.10.34-1.el6.x86_64                                                                                     2/2 

Installed:
  nodejs.x86_64 0:0.10.42-4.el6                                                                                                     

Dependency Installed:
  libuv.x86_64 1:0.10.34-1.el6                                                                                                      

Complete!
[root@h202 blog]#
~~~


再次尝试 启动服务

~~~
[root@h202 blog]# rails server
=> Booting WEBrick
=> Rails 4.2.6 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2016-04-22 13:28:17] INFO  WEBrick 1.3.1
[2016-04-22 13:28:17] INFO  ruby 2.3.0 (2015-12-25) [x86_64-linux]
[2016-04-22 13:28:17] INFO  WEBrick::HTTPServer#start: pid=11288 port=3000
...
...
...
~~~

成功启动，在本地启动浏览器，可以进行访问 (无法从外部访问，原因是并未绑定IP)

![rails1.png](/images/rails/rails1.png)


直接使用 **Ctrl + C** 就可以停止此应用 

如果希望从外部访问，可以进行如下配置

**`-b`** 可以绑定服务 **IP**

~~~
[root@h202 blog]# rails server -b 0.0.0.0
=> Booting WEBrick
=> Rails 4.2.6 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2016-04-22 13:47:39] INFO  WEBrick 1.3.1
[2016-04-22 13:47:39] INFO  ruby 2.3.0 (2015-12-25) [x86_64-linux]
[2016-04-22 13:47:39] INFO  WEBrick::HTTPServer#start: pid=11622 port=3000
...
...
...
~~~

打开防火墙

~~~
[root@h202 ~]# netstat -ant | grep 300
tcp        0      0 127.0.0.1:3000              0.0.0.0:*                   LISTEN      
tcp        0      0 ::1:3000                    :::*                        LISTEN      
[root@h202 ~]# 
[root@h202 ~]# vim /etc/sysconfig/iptables
[root@h202 ~]# /etc/init.d/iptables reload 
iptables: Trying to reload firewall rules:                 [  OK  ]
[root@h202 ~]# iptables -L -nv | grep 3000
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:3000 
[root@h202 ~]# 
~~~

启动浏览器，可以进行访问

![rails2.png](/images/rails/rails2.png)

![rails3.png](/images/rails/rails3.png)



---

# 命令汇总

* **`gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`**
* **`curl -sSL https://get.rvm.io | bash -s stable`**
* **`rvm list known`**
* **`rvm list`**
* **`rvm install 2.3`**
* **`ruby -v`**
* **`gem -v`**
* **`sqlite3 --version`**
* **`gem source -l`**
* **`time gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/`**
* **`gem install rails`**
* **`rails --version`**
* **`gem list`**
* **`rails new blog`**
* **`cd blog/`**
* **`head -n 3 Gemfile`**
* **`vim Gemfile`**
* **`gem install rake -v '11.1.2'`**
* **`bundle install`**
* **`tree`**
* **`rails server`**
* **`gem install uglifier`**
* **`bundle exec spring binstub --all`**
* **`bundle list | grep uglifier`**
* **`yum install nodejs`**
* **`rails server -b 0.0.0.0`**
* **`netstat -ant | grep 300`**
* **`iptables -L -nv | grep 3000`**


---

[rubyonrails]:http://rubyonrails.org/
[getting_started]:http://guides.rubyonrails.org/getting_started.html
[getting_started_cn]:http://guides.ruby-china.org/getting_started.html
[rvm]:http://www.rvm.io/
[config]:http://guides.ruby-china.org/configuring.html
[bundler]:http://bundler.io/
[testing]:http://guides.ruby-china.org/testing.html
[uglifier]:https://rubygems.org/gems/uglifier

