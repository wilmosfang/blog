---
layout: post
title:  Rails 容器与配置
author: wilmosfang
categories:   docker  
tags:   docker ruby rails 
wc: 661 1367 20401
excerpt:  rails 官方镜像的拉取，rails 镜像的构建，rails 容器的构建，访问操作和注意事项 
comments: true
---



# 前言

**[Rails][rails]** 是使用 Ruby 语言编写的网页程序开发框架

通过集成开发者需要的常用组件，极大的降低了网页程序的开发成本

前面几篇博客中使用 Rails 框架构建了一个具备基本认证功能的简单博客系统，详细可以参考：

* **[Ruby on Rails 基础][rails_basic]**
* **[Rails MVC 和 CRUD][rails_crud]**
* **[Rails 构建评论功能][rails_comments]**

当然，不了解也没关系，因为绝大部分开发的细节都不是运维需要关心的，运维更需要关心的是部署

传统的 **Ruby on Rails** 应用是使用 **Capistrano** 来进行自动化布署的，其实效率已经很高了，那有没有比它更高效的方式呢？

当然有，**Docker** 是 **DevOps** 神器，将 **Rails** 应用 **Docker** 化后，我们可以更进一步降低布署的复杂度，负责发布的运维人员可以退化为 **Docker(码头工人)** 只需要将 **箱子(应用)** 搬到正确的地方就OK了，基本告别了发布过程中由于环境冲突而痛苦Debug的时代

> 运维人员的命运是很奇特的，自已发明的工具来革自己的命，自已编写的软件来跟自己抢饭碗，代替人力就是自动化工具的根本目标，毫无疑问，云时代的来临，大量运维人员将面临“失业”，因为高效平台工具的出现，使企业对运维的总体需求规模小了不止一个量级，或者说一个运维人员可以cover掉以前100(虚指，并无翔实数据源)个运维的产出，运维工种会更为细分，更为专精，但这并非悲观论调，而是进步的表现，总体趋势上来看人力资源节省了，所以聪明的运维会找准定位，适时调整

目前来讲，容器也比较适合运行无状态的服务，类似于web服务的应用层(app layer)，因为这样可以很方便地进行水平扩展，系统的可扩展性，高弹性因此而变得很容易实现

这里分享一下 Docker 化一个 Rails 应用的操作过程和相关基础，详细可以参考 **Docker hub** 中的 **[Rails OFFICIAL REPOSITORY][docker_rails]** 和 **[官方文档][compose_rails]**


> **Tip:** 当前的 Docker 最新版本为 **Docker Version 1.10** ，Rails 最新版本为 **Rails 5.0.0.beta3** ， Docker hub 中的 Rails 官方镜像最新版本为 **Rails 4.2.6**


---


# 概要

* TOC
{:toc}


---

## 环境

~~~ 
[root@h104 ~]# hostnamectl 
   Static hostname: h104
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 12a02f8ee88d4b8e91d54d1390b0b275
           Boot ID: ac91120b8b4446f193e7cc3e25f278e4
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.4.4.el7.x86_64
      Architecture: x86-64
[root@h104 ~]# uname -a 
Linux h104 3.10.0-327.4.4.el7.x86_64 #1 SMP Tue Jan 5 16:07:00 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
[root@h104 ~]# docker --version
Docker version 1.9.1, build a34a1d5
[root@h104 ~]#
~~~

---

## 拉取官方镜像

这个过程很漫长，可以准备点视频或瓜子什么的，实在无聊也可以翻翻我的其它博客 (^ ^) 

~~~
[root@h104 ~]# docker pull rails 
Using default tag: latest
latest: Pulling from library/rails
004814f54a9a: Pull complete 
4786bcc15aac: Pull complete 
b6b57a59043e: Pull complete 
783fdfa6305f: Pull complete 
298958ea032a: Pull complete 
ae0c9441f5a3: Pull complete 
15f206b10e55: Pull complete 
6529e7d7f485: Pull complete 
b0ad7658b188: Pull complete 
b9cc583df59a: Pull complete 
9163bcf48f72: Pull complete 
b0e3fc140041: Pull complete 
57c77d269392: Pull complete 
533d0a2f687a: Pull complete 
ffe115a2f981: Pull complete 
8cda06d14823: Pull complete 
5b3b2ad1e099: Pull complete 
342ff98b0e82: Pull complete 
fc2eabed675c: Pull complete 
afdddae9b2bf: Pull complete 
Digest: sha256:a9c33d16edd9a3819f1ff9662615bef97b3c77d40773c3e7298c856f796cf3d8
Status: Downloaded newer image for rails:latest
[root@h104 ~]#
~~~

系统里多出来一个镜像，是rails的最新版


~~~
[root@h104 ~]# docker images | grep rails
rails                                   latest              afdddae9b2bf        46 hours ago        833.7 MB
[root@h104 ~]# 
~~~

不得不说，还是有点大的 **833.7 MB**，相较而言一个完整的rails应用代码才区区几兆

~~~
[root@h202 ruby]# du -sh blog/
2.0M	blog/
[root@h202 ruby]#
~~~

可见 Rails 框架帮我们完成了多少额外工作，我们的核心代码也因此而精简

反观，Rails的框架依赖有多么臃肿，整个一大胖子，应该也是反映慢的原因之一吧

不过话说回来，正因为这些基础，这个应用可以自立根生，除了系统内核和Docker提供的隔离环境，它的运行不再看其它环境或基础设施的脸色，可以独立运行了

> **Tip:** 同时，它依赖的那么多层基础镜像是可以和其它容器共享的，并非每次都是成倍的磁盘空间需求，大量相似容器的环境中，一定程度上还节约了磁盘空间


可以看看镜像的详细内容

~~~
[root@h104 blog2]# docker inspect afdddae9b2bf
[
{
    "Id": "afdddae9b2bf7469476e271850590aaee2e2c7353121e2801f4c3bd35b30e324",
    "RepoTags": [
        "rails:latest"
    ],
    "RepoDigests": [],
    "Parent": "fc2eabed675c24b767c4d78dfad2c8a525f778e06a81278a169e262e4ccc9eff",
    "Comment": "",
    "Created": "2016-04-27T17:18:03.953130492Z",
    "Container": "af60debb03e6f61c0e0d5875dd3859408151d335c59abf4e9ff986b3ada4c517",
    "ContainerConfig": {
        "Hostname": "bcad5a346f31",
        "Domainname": "",
        "User": "",
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "Tty": false,
        "OpenStdin": false,
        "StdinOnce": false,
        "Env": [
            "PATH=/usr/local/bundle/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "RUBY_MAJOR=2.3",
            "RUBY_VERSION=2.3.1",
            "RUBY_DOWNLOAD_SHA256=b87c738cb2032bf4920fef8e3864dc5cf8eae9d89d8d523ce0236945c5797dcd",
            "RUBYGEMS_VERSION=2.6.3",
            "BUNDLER_VERSION=1.11.2",
            "GEM_HOME=/usr/local/bundle",
            "BUNDLE_PATH=/usr/local/bundle",
            "BUNDLE_BIN=/usr/local/bundle/bin",
            "BUNDLE_SILENCE_ROOT_WARNING=1",
            "BUNDLE_APP_CONFIG=/usr/local/bundle",
            "RAILS_VERSION=4.2.6"
        ],
        "Cmd": [
            "/bin/sh",
            "-c",
            "gem install rails --version \"$RAILS_VERSION\""
        ],
        "Image": "5da847340e289b1357164d7a9f62ac6e67a557c63c4cb2c8823b4cb341776e15",
        "Volumes": null,
        "WorkingDir": "",
        "Entrypoint": null,
        "OnBuild": [],
        "Labels": {}
    },
    "DockerVersion": "1.9.1",
    "Author": "",
    "Config": {
        "Hostname": "bcad5a346f31",
        "Domainname": "",
        "User": "",
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "Tty": false,
        "OpenStdin": false,
        "StdinOnce": false,
        "Env": [
            "PATH=/usr/local/bundle/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "RUBY_MAJOR=2.3",
            "RUBY_VERSION=2.3.1",
            "RUBY_DOWNLOAD_SHA256=b87c738cb2032bf4920fef8e3864dc5cf8eae9d89d8d523ce0236945c5797dcd",
            "RUBYGEMS_VERSION=2.6.3",
            "BUNDLER_VERSION=1.11.2",
            "GEM_HOME=/usr/local/bundle",
            "BUNDLE_PATH=/usr/local/bundle",
            "BUNDLE_BIN=/usr/local/bundle/bin",
            "BUNDLE_SILENCE_ROOT_WARNING=1",
            "BUNDLE_APP_CONFIG=/usr/local/bundle",
            "RAILS_VERSION=4.2.6"
        ],
        "Cmd": [
            "irb"
        ],
        "Image": "5da847340e289b1357164d7a9f62ac6e67a557c63c4cb2c8823b4cb341776e15",
        "Volumes": null,
        "WorkingDir": "",
        "Entrypoint": null,
        "OnBuild": [],
        "Labels": {}
    },
    "Architecture": "amd64",
    "Os": "linux",
    "Size": 54280468,
    "VirtualSize": 833718357,
    "GraphDriver": {
        "Name": "devicemapper",
        "Data": {
            "DeviceId": "294",
            "DeviceName": "docker-253:0-134859501-afdddae9b2bf7469476e271850590aaee2e2c7353121e2801f4c3bd35b30e324",
            "DeviceSize": "107374182400"
        }
    }
}
]
[root@h104 blog2]#
~~~

通过 **`docker inspect afdddae9b2bf`** 可以获得丰富的，容器镜像的细节


> **Tip:**  后面的操作并不依赖于这上面的操作，上面的操作只是为了演示官方 rails 镜像的拉取和相关属性


---

## 拷贝 Rails 应用


从这里开始构建一个可以被反复使用的 Rails 镜像


~~~
[root@h202 ruby]# ls
blog
[root@h202 ruby]# rsync  -av blog/   root@192.168.100.104:/tmp/blog
root@192.168.100.104's password: 
sending incremental file list
created directory /tmp/blog
./
.gitignore
Gemfile
Gemfile.lock
README.rdoc
Rakefile
config.ru
app/
app/assets/
app/assets/images/
app/assets/images/.keep
app/assets/javascripts/
app/assets/javascripts/application.js
app/assets/javascripts/articles.coffee
app/assets/javascripts/comments.coffee
app/assets/javascripts/welcome.coffee
app/assets/stylesheets/
app/assets/stylesheets/application.css
app/assets/stylesheets/articles.scss
app/assets/stylesheets/comments.scss
app/assets/stylesheets/welcome.scss
app/controllers/
app/controllers/application_controller.rb
app/controllers/articles_controller.rb
app/controllers/comments_controller.rb
app/controllers/welcome_controller.rb
app/controllers/concerns/
app/controllers/concerns/.keep
app/helpers/
app/helpers/application_helper.rb
app/helpers/articles_helper.rb
app/helpers/comments_helper.rb
app/helpers/welcome_helper.rb
app/mailers/
app/mailers/.keep
app/models/
app/models/.keep
app/models/article.rb
app/models/comment.rb
app/models/concerns/
app/models/concerns/.keep
app/views/
app/views/articles/
app/views/articles/_form.html.reb
app/views/articles/edit.html.erb
app/views/articles/index.html.erb
app/views/articles/new.html.erb
app/views/articles/show.html.erb
app/views/comments/
app/views/comments/_comment.html.erb
app/views/comments/_form.html.erb
app/views/layouts/
app/views/layouts/application.html.erb
app/views/welcome/
app/views/welcome/index.html.erb
bin/
bin/bundle
bin/rails
bin/rake
bin/setup
bin/spring
config/
config/application.rb
config/boot.rb
config/database.yml
config/environment.rb
config/routes.rb
config/secrets.yml
config/environments/
config/environments/development.rb
config/environments/production.rb
config/environments/test.rb
config/initializers/
config/initializers/assets.rb
config/initializers/backtrace_silencers.rb
config/initializers/cookies_serializer.rb
config/initializers/filter_parameter_logging.rb
config/initializers/inflections.rb
config/initializers/mime_types.rb
config/initializers/session_store.rb
config/initializers/wrap_parameters.rb
config/locales/
config/locales/en.yml
db/
db/development.sqlite3
db/schema.rb
db/seeds.rb
db/migrate/
db/migrate/20160422140912_create_articles.rb
db/migrate/20160427082552_create_comments.rb
lib/
lib/assets/
lib/assets/.keep
lib/tasks/
lib/tasks/.keep
log/
log/.keep
log/development.log
public/
public/404.html
public/422.html
public/500.html
public/favicon.ico
public/robots.txt
test/
test/test_helper.rb
test/controllers/
test/controllers/.keep
test/controllers/articles_controller_test.rb
test/controllers/comments_controller_test.rb
test/controllers/welcome_controller_test.rb
test/fixtures/
test/fixtures/.keep
test/fixtures/articles.yml
test/fixtures/comments.yml
test/helpers/
test/helpers/.keep
test/integration/
test/integration/.keep
test/mailers/
test/mailers/.keep
test/models/
test/models/.keep
test/models/article_test.rb
test/models/comment_test.rb
tmp/
tmp/cache/
tmp/cache/assets/
...
...
tmp/cache/assets/sprockets/v3.0/z_/z_Prv0YfktgUwhZma2rWy0-p7b7X6Fp4yrAvnhlhUP4.cache
tmp/pids/
tmp/sessions/
tmp/sockets/
vendor/
vendor/assets/
vendor/assets/javascripts/
vendor/assets/javascripts/.keep
vendor/assets/stylesheets/
vendor/assets/stylesheets/.keep

sent 902359 bytes  received 4048 bytes  139447.23 bytes/sec
total size is 883817  speedup is 0.98
[root@h202 ruby]# 
~~~


~~~
[root@h104 tmp]# cp blog/ blog2 -r 
[root@h104 tmp]# cd blog2
[root@h104 blog2]# ls
app  bin  config  config.ru  db  Gemfile  Gemfile.lock  lib  log  public  Rakefile  README.rdoc  test  tmp  vendor
[root@h104 blog2]#
~~~



---

## 创建 onbuild Dockerfile


只需要加上一行 **`FROM rails:onbuild`**


创建的位置为 app 项目的根，**Gemfile** 的旁边

~~~
[root@h104 blog2]# ls
app  bin  config  config.ru  db  Gemfile  Gemfile.lock  lib  log  public  Rakefile  README.rdoc  test  tmp  vendor
[root@h104 blog2]# vim Dockerfile
[root@h104 blog2]# cat Dockerfile 
FROM rails:onbuild
[root@h104 blog2]# 
~~~

这个 **ONBUILD** 镜像可以用于大部分的Rails应用，它会完成类似如下的一些工作 ：

* **COPY . /usr/src/app**
* **bundle install**
* **EXPOSE 3000**
* **rails server**


---

## 构建 Rails 容器镜像


注意目录在 app 项目的根一层

~~~
[root@h104 blog2]# ls
app  bin  config  config.ru  db  Dockerfile  Gemfile  Gemfile.lock  lib  log  public  Rakefile  README.rdoc  test  tmp  vendor
[root@h104 blog2]# docker build -t test-rails-app-blog . 
Sending build context to Docker daemon 1.104 MB
Step 1 : FROM rails:onbuild
onbuild: Pulling from library/rails
f502f0e93adb: Pull complete 
41fb86dd2354: Pull complete 
7db4e84aa159: Pull complete 
4e4386f0802f: Pull complete 
2010da638e26: Pull complete 
d63c045b79b9: Pull complete 
8471367ded15: Pull complete 
2b85b1b3b222: Downloading [================>                                  ] 939.4 kB/2.878 MB
2b85b1b3b222: Pull complete 
f9d35d3939ac: Pull complete 
1aa074a974f7: Pull complete 
097c204ce316: Pull complete 
Digest: sha256:e9d8f1a8e16137880b074c60e7c2d6e0ced6bd498d0d871f6c15ffdc619b8e5a
Status: Downloaded newer image for rails:onbuild
# Executing 4 build triggers...
Step 1 : COPY Gemfile /usr/src/app/
Step 1 : COPY Gemfile.lock /usr/src/app/
Step 1 : RUN bundle install
 ---> Running in a6d3a5d93541
Fetching gem metadata from https://gems.ruby-china.org/...........
Fetching version metadata from https://gems.ruby-china.org/...
Fetching dependency metadata from https://gems.ruby-china.org/..
Installing rake 11.1.2
Installing i18n 0.7.0
Using json 1.8.3
Installing minitest 5.8.4
Installing thread_safe 0.3.5
Installing builder 3.2.2
Installing erubis 2.7.0
Installing mini_portile2 2.0.0
Installing rack 1.6.4
Installing mime-types-data 3.2016.0221
Installing arel 6.0.3
Installing debug_inspector 0.0.2 with native extensions
Installing byebug 8.2.4 with native extensions
Installing coffee-script-source 1.10.0
Installing execjs 2.6.0
Installing thor 0.19.1
Installing concurrent-ruby 1.0.1
Installing multi_json 1.11.2
Using bundler 1.11.2
Installing sass 3.4.22
Installing tilt 2.0.2
Installing spring 1.7.1
Installing sqlite3 1.3.11 with native extensions
Installing rdoc 4.2.2
Installing tzinfo 1.2.2
Installing nokogiri 1.6.7.2 with native extensions
Installing rack-test 0.6.3
Installing mime-types 3.0
Installing binding_of_caller 0.7.2 with native extensions
Installing coffee-script 2.4.1
Installing uglifier 3.0.0
Installing sprockets 3.6.0
Installing sdoc 0.4.1
Installing activesupport 4.2.6
Installing loofah 2.0.3
Installing mail 2.6.4
Installing rails-deprecated_sanitizer 1.0.3
Installing globalid 0.3.6
Installing activemodel 4.2.6
Installing jbuilder 2.4.1
Installing rails-html-sanitizer 1.0.3
Installing rails-dom-testing 1.0.7
Installing activejob 4.2.6
Installing activerecord 4.2.6
Installing actionview 4.2.6
Installing actionpack 4.2.6
Installing actionmailer 4.2.6
Installing railties 4.2.6
Installing sprockets-rails 3.0.4
Installing coffee-rails 4.1.1
Installing jquery-rails 4.1.1
Installing rails 4.2.6
Installing sass-rails 5.0.4
Installing web-console 2.3.0
Installing turbolinks 2.5.3
Bundle complete! 12 Gemfile dependencies, 55 gems now installed.
Bundled gems are installed into /usr/local/bundle.
Post-install message from rdoc:
Depending on your version of ruby, you may need to install ruby rdoc/ri data:

<= 1.8.6 : unsupported
 = 1.8.7 : gem install rdoc-data; rdoc-data --install
 = 1.9.1 : gem install rdoc-data; rdoc-data --install
>= 1.9.2 : nothing to do! Yay!
Step 1 : COPY . /usr/src/app
 ---> b5b7ed8d740e
Removing intermediate container dee87f8e4f1f
Removing intermediate container 175758fecfc8
Removing intermediate container a6d3a5d93541
Removing intermediate container 8134ef278d71
Successfully built b5b7ed8d740e
[root@h104 blog2]# 
~~~


完成后系统中多出了两个镜像

~~~
[root@h104 blog2]# docker images | grep rails
test-rails-app-blog                     latest              b5b7ed8d740e        2 hours ago         851.3 MB
rails                                   onbuild             097c204ce316        46 hours ago        779.4 MB
rails                                   latest              afdddae9b2bf        47 hours ago        833.7 MB
[root@h104 blog2]# 
~~~

现在我们可以使用生成的 **test-rails-app-blog** 来创建容器


---

## 创建 Rails 容器

~~~
[root@h104 ~]# docker run --name blog-rails-app -p 8080:3000 -d test-rails-app-blog
b460d005093fc36774ad6cddc8697a0f76c59d6a084db9508f48a5655142e852
[root@h104 ~]# docker ps 
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
b460d005093f        test-rails-app-blog   "rails server -b 0.0."   4 seconds ago       Up 3 seconds        0.0.0.0:8080->3000/tcp   blog-rails-app
[root@h104 ~]#
~~~

---

## 访问操作


访问根 **`http://192.168.100.104:8080/`**  

![rails38.png](/images/rails/rails38.png)

查看所有文章

![rails39.png](/images/rails/rails39.png)

尝试添加一篇新文章，点击连接 **[New article]** ，弹出对话框，提示输入密码

![rails40.png](/images/rails/rails40.png)


输入帐号密码 soft/dog，确认 **[登录]**

![rails41.png](/images/rails/rails41.png)

认证成功，进入填写文章内容的界面，随便输入点东西，然后点击链接 **[Save Article]**

![rails42.png](/images/rails/rails42.png)

保存后就到了此文章的展示窗口，随便添加内容作为评论，然后点击链接 **[Create Comment]**

![rails43.png](/images/rails/rails43.png)

提交后评论如期展示了出来，点击链接 **[Back]**

![rails44.png](/images/rails/rails44.png)

回到了所有文章列表的界面，这时最下面多出了一篇文章

![rails45.png](/images/rails/rails45.png)

点击第一篇的链接 **[Destroy]** 尝试删除它，弹出了对话框，让我确认

![rails46.png](/images/rails/rails46.png)

点击按钮 **[确定]** 后，第一篇文章就被删除了

![rails47.png](/images/rails/rails47.png)


这个博客系统和之前的特性一样，功能上没有任何差别

为了实现简便，这里我们使用的是sqlite，由于保存了数据，所以其实它是有状态的，我们虽然可以开启多个容器，但每个之间由于不共享数据，所以是相互独立的

这可以通过共用数据库来解决，使用统一缓存来存session信息，使用集中的DB来存储数据，应用层不保存数据，这样就可以根据业务需求和业务压力任意扩容和缩容应用层的 Capacity

---

# 命令汇总


* **`docker pull rails`**
* **`docker images | grep rails`**
* **`du -sh blog/`**
* **`docker inspect afdddae9b2bf`**
* **`rsync  -av blog/   root@192.168.100.104:/tmp/blog`**
* **`cp blog/ blog2 -r`**
* **`cd blog2`**
* **`cat Dockerfile`**
* **`docker build -t test-rails-app-blog .`**
* **`docker run --name blog-rails-app -p 8080:3000 -d test-rails-app-blog`**




---


[rails]:http://rubyonrails.org/
[rails_basic]:/2016/04/22/rails-basic/
[rails_crud]:/2016/04/24/rails-crud/
[rails_comments]:/2016/04/27/rails-comments/
[docker_rails]:https://hub.docker.com/_/rails/
[compose_rails]:https://docs.docker.com/compose/rails/
