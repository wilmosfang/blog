---
layout: post
title:  Rails MVC 和 CRUD
author: wilmosfang
categories:   ruby
tags:   ruby rails
wc: 1269 2497 30306
excerpt:  创建一个简单的Rails应用，MVC 基础，Rails 的 MVC 框架，创建 controller 和 view，修改页面内容，设置首页，简单访问，添加资源，定义 new 方法，创建模板，添加表单，定义 create 方法，创建 model，进行迁移，保存数据，显示文章，添加链接，数据校验，更新文章，删除文章
comments: true
---



# 前言

**[Rails][rubyonrails]** 是使用 Ruby 语言编写的网页程序开发框架

通过集成开发者需要的常用组件，极大地简化了网页程序的开发

> **Tip:** 类似于 python 的 Django ，perl 的 Dancer


继前面的 **[Ruby on Rails 基础][rails_basic]** ，这里再进一步探究一下其内部运作机制


 **[Rails][rubyonrails]** 的相关基础，详细可以参考 **[官方文档][getting_started]** 和 Ruby China 的 **[Rails 入门][getting_started_cn]** 


> **Tip:** 当前的最新版本为 **Rails 5.0.0.beta3** 发布于 February 27, 2016 4:00 pm


---


# 概要

* TOC
{:toc}


---

## 环境



~~~
[root@h202 blog]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h202 blog]# uname -a 
Linux h202 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h202 blog]# ruby -v 
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
[root@h202 blog]# gem -v 
2.5.1
[root@h202 blog]# rails --version
Rails 4.2.6
[root@h202 blog]# node -v 
v0.10.42
[root@h202 blog]# rvm -v 
rvm 1.27.0 (latest) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
[root@h202 blog]# ls
app  bin  config  config.ru  db  Gemfile  Gemfile.lock  lib  log  public  Rakefile  README.rdoc  test  tmp  vendor
[root@h202 blog]# 
~~~


---


## 运行应用

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

---


## MVC

Rails 是 MVC 框架的 ruby 实现

那什么是 MVC 呢？

MVC (Model-View-Controller) 是一种软件架构，或者说是设计理念，不同语言有不同的实现


~~~
MVC
   +------------------------+
   |                        V
+------+    +----+    +----------+     +-----+     +--+
|client|<---|view|<---|controller|<--->|model|<--->|DB|
+------+    +----+    +----------+     +-----+     +--+
~~~



MVC 框架有什么好处呢？

MVC 分块设计有助于管理复杂的应用程序，因为可以在一段时间内只用关注一个方面；例如，可以在不依赖业务逻辑的情况下专注于视图设计；同时也让应用程序的测试更加容易；MVC 分层同时也简化了分组开发；不同的开发人员可同时开发视图、控制器逻辑和业务逻辑

其核心思想就是模块化，各司其职，分工协作


下面是大体的数据流向图


![mvc1.png](/images/rails/mvc1.png)



* Model（模型）是应用程序中用于处理应用程序数据逻辑的部分，通常模型对象负责在数据库中存取数据
* View（视图）是应用程序中处理数据显示的部分，通常视图是依据模型数据创建的
* Controller（控制器）是应用程序中处理用户交互的部分，通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据


![mvc2.png](/images/rails/mvc2.png)


---


## 创建一个简单页面


### Rails 的 MVC 架构


MVC 角色

![railsmvc1.png](/images/rails/railsmvc1.png)

与数据库的交互

![railsmvc2.png](/images/rails/railsmvc2.jpg)


数据流程

![railsmvc3.png](/images/rails/railsmvc3.png)


对应文件

![railsmvc4.png](/images/rails/railsmvc4.png)


---

### 创建一个控制器和视图

要在 Rails 中显示“My first test” 的静态页面，需要新建一个控制器和视图

控制器用来接受向程序发起的请求

视图的作用是，以人类能看懂的格式显示数据

~~~
[root@h202 blog]# rails generate controller welcome index
Running via Spring preloader in process 11871
      create  app/controllers/welcome_controller.rb
       route  get 'welcome/index'
      invoke  erb
      create    app/views/welcome
      create    app/views/welcome/index.html.erb
      invoke  test_unit
      create    test/controllers/welcome_controller_test.rb
      invoke  helper
      create    app/helpers/welcome_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/welcome.coffee
      invoke    scss
      create      app/assets/stylesheets/welcome.scss
[root@h202 blog]# 
~~~

---

### 修改页面内容


~~~
[root@h202 blog]# vim app/views/welcome/index.html.erb
[root@h202 blog]# cat app/views/welcome/index.html.erb
<h1>My first test</h1>
<p>Find me in app/views/welcome/index.html.erb</p>
[root@h202 blog]#
~~~

---

### 设置首页

路由决定哪个控制器会接受到这个请求

~~~
[root@h202 blog]# vim config/routes.rb 
[root@h202 blog]# grep -v " #" config/routes.rb | grep -v "^$"
Rails.application.routes.draw do
  get 'welcome/index'
  root 'welcome#index'
end
[root@h202 blog]#
~~~

---

### 进行访问


直接刷新页面

![rails4.png](/images/rails/rails4.png)

注意，我修改了配置和服务，但并没有对服务进行重启，而可以直接加载出新的内容，说明 Rails 可以进行动态加载

>In development mode, Rails does not generally require you to restart the server; changes you make in files will be automatically picked up by the server.


下面是访问过程中产生的日志


~~~
Started GET "/" for 192.168.100.1 at 2016-04-22 20:13:15 +0800
Cannot render console from 192.168.100.1! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by WelcomeController#index as HTML
  Rendered welcome/index.html.erb within layouts/application (0.1ms)
Completed 200 OK in 38ms (Views: 37.0ms | ActiveRecord: 0.0ms)
~~~


使用 **`http://192.168.100.202:3000/welcome/index`** 也可以进行访问

![rails5.png](/images/rails/rails5.png)


下面是访问过程中产生的日志


~~~
Started GET "/welcome/index" for 192.168.100.1 at 2016-04-22 20:16:03 +0800
Cannot render console from 192.168.100.1! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by WelcomeController#index as HTML
  Rendered welcome/index.html.erb within layouts/application (0.2ms)
Completed 200 OK in 44ms (Views: 42.9ms | ActiveRecord: 0.0ms)
~~~


---

## 资源的CRUD

资源的创建、读取、更新和删除操作，简称为 CRUD。

我们来尝试创建资源

### 添加资源到 route

~~~
[root@h202 blog]# vim config/routes.rb 
[root@h202 blog]# grep -v ' #' config/routes.rb | grep -v "^$"
Rails.application.routes.draw do
  resources :articles
  root 'welcome#index'
end
[root@h202 blog]# rake routes
      Prefix Verb   URI Pattern                  Controller#Action
    articles GET    /articles(.:format)          articles#index
             POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show
             PATCH  /articles/:id(.:format)      articles#update
             PUT    /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
        root GET    /                            welcome#index
[root@h202 blog]#
~~~

结果展示了当前的一系列 Restfull API 与 Controller#Action 的对应关系

我们尝试访问其中的一个链接，**`/articles/new`**  得到如下反馈


![rails6.png](/images/rails/rails6.png)

报错的原因为没有 **ArticlesController**

---

### 创建控制器

~~~
[root@h202 blog]# bin/rails g controller articles
Running via Spring preloader in process 12913
      create  app/controllers/articles_controller.rb
      invoke  erb
      create    app/views/articles
      invoke  test_unit
      create    test/controllers/articles_controller_test.rb
      invoke  helper
      create    app/helpers/articles_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/articles.coffee
      invoke    scss
      create      app/assets/stylesheets/articles.scss
[root@h202 blog]# ll app/controllers/articles_controller.rb 
-rw-r--r-- 1 root root 53 Apr 22 20:49 app/controllers/articles_controller.rb
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
end
[root@h202 blog]# 
~~~

创建了一个叫 **ArticlesController** 的类，继承自 **ApplicationController**

![rails7.png](/images/rails/rails7.png)

这次报错变了，成了找不到 **new** 方法

---

### 定义 new 方法 

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
  end
end
[root@h202 blog]#
~~~

刷新页面

![rails8.png](/images/rails/rails8.png)

这次报错，是视图中找不到对应的模板

---

### 创建模板

~~~
[root@h202 blog]# vim app/views/articles/new.html.erb
[root@h202 blog]# cat app/views/articles/new.html.erb
<h1>Test blog http://soft.dog/</h1>
[root@h202 blog]# 
~~~

要注意命名，因为 Rails 中 **约定优于配置** 的设计，这里的目录地址，和文件命名都是有意义的

**`app/views`** 是默认的视图存放处

**`articles/new`** 是 **`articles#new`** 方法默认去寻找的视图

**`new.html.erb`** 文件中后面的扩展名 **`.html.erb`** 也包含了意义，html 指定模板类型，erb 指定用来处理模板的程序

再次刷新

![rails9.png](/images/rails/rails9.png)

这次没有报错，获得了我指定的内容

---

### 添加表单


使用 form_for 来构造一个简单表单

~~~
[root@h202 blog]# vim app/views/articles/new.html.erb
[root@h202 blog]# cat app/views/articles/new.html.erb
<h1>Test blog http://soft.dog/</h1>

<%= form_for :article do |f| %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
 
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
 
  <p>
    <%= f.submit %>
  </p>
<% end %>
[root@h202 blog]#
~~~

直接刷新页面

![rails10.png](/images/rails/rails10.png)

一个简单的表单就呈现出来了


不过，通过查看源码，我们可以看到 **action** 部分指向的是当前页面 **`action="/articles/new"`** ， 而这个页面 (Restfull API) 应该是用来进行显示的，而不是进行处理的

![rails11.png](/images/rails/rails11.png)

我们进行一下调整

~~~
[root@h202 blog]# vim app/views/articles/new.html.erb
[root@h202 blog]# cat app/views/articles/new.html.erb
<h1>Test blog http://soft.dog/</h1>

<%= form_for :article, url: articles_path do |f| %>

  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
 
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
 
  <p>
    <%= f.submit %>
  </p>
<% end %>
[root@h202 blog]# 
~~~

指定 **`/articles`** 为对应的 URL

~~~
[root@h202 blog]# bin/rake routes
Running via Spring preloader in process 13088
      Prefix Verb   URI Pattern                  Controller#Action
    articles GET    /articles(.:format)          articles#index
             POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show
             PATCH  /articles/:id(.:format)      articles#update
             PUT    /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
        root GET    /                            welcome#index
[root@h202 blog]# 
~~~

(这里的 **/articles** 明明对应两个方法，**GET、POST** ，有点不太明白，为什么这样指定就一定成了POST请求)

随便填写东西，提交


![rails12.png](/images/rails/rails12.png)


![rails13.png](/images/rails/rails13.png)

原因是 **ArticlesController** 中找不到对应的 **create** 方法


---

### 定义 create 方法

~~~
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
  end
end
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
  end
  def create
  end
end
[root@h202 blog]# 
~~~

重新提交一次

![rails14.png](/images/rails/rails14.png)

这回又是找不到模板，不过变成了找不到 **create** 模板

暂时不尝试去直接解决这个模板问题

我们将刚才获取到的内容直接反馈回来


修改create方法，直接打印获取到的参数

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
  end
  def create
    render plain: params[:article].inspect
  end
end
[root@h202 blog]#
~~~

再次提交一回

![rails15.png](/images/rails/rails15.png)

---

### 创建模型

Rails 提供了一个生成器用来创建模型

~~~
[root@h202 blog]# bin/rails generate model Article title:string text:text
Running via Spring preloader in process 13216
      invoke  active_record
      create    db/migrate/20160422140912_create_articles.rb
      create    app/models/article.rb
      invoke    test_unit
      create      test/models/article_test.rb
      create      test/fixtures/articles.yml
[root@h202 blog]#
~~~

生成的两个文件中包含了这个 model 的结构

~~~
[root@h202 blog]# cat db/migrate/20160422140912_create_articles.rb
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.string :title
      t.text :text

      t.timestamps null: false
    end
  end
end
[root@h202 blog]# cat app/models/article.rb
class Article < ActiveRecord::Base
end
[root@h202 blog]#
~~~

可知这个新生成的 model 继承自 ActiveRecord



---

### 进行迁移

迁移就是将前面定义的model ，落实到数据库中形成表结构

~~~
[root@h202 blog]# bin/rake db:migrate
Running via Spring preloader in process 13248
== 20160422140912 CreateArticles: migrating ===================================
-- create_table(:articles)
   -> 0.0131s
== 20160422140912 CreateArticles: migrated (0.0132s) ==========================

[root@h202 blog]#
~~~

那到底将数据结构定义到了哪里呢

~~~
[root@h202 blog]# cat config/database.yml 
# SQLite version 3.x
#   gem install sqlite3
#
#   Ensure the SQLite 3 gem is defined in your Gemfile
#   gem 'sqlite3'
#
default: &default
  adapter: sqlite3
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db/development.sqlite3

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: db/test.sqlite3

production:
  <<: *default
  database: db/production.sqlite3
[root@h202 blog]# grep -v "^#" config/database.yml 
default: &default
  adapter: sqlite3
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db/development.sqlite3

test:
  <<: *default
  database: db/test.sqlite3

production:
  <<: *default
  database: db/production.sqlite3
[root@h202 blog]# ll db/development.sqlite3 
-rw-r--r-- 1 root root 5120 Apr 22 22:19 db/development.sqlite3
[root@h202 blog]#
~~~


---

### 保存数据

修改 create 方法，对提交的数据进行保存

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
  end
  def create
#    render plain: params[:article].inspect
     @article = Article.new(params[:article])
     @article.save
     redirect_to @article
  end
end
[root@h202 blog]#
~~~

再提交一次

![rails16.png](/images/rails/rails16.png)

我们要明确地告知 Rails 哪些参数可在控制器中使用，否则安全检查通不过

我们再作一下修改

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     @article.save
     redirect_to @article
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]#
~~~

---

### 显示文章

再次提交会产生这样的报错

![rails17.png](/images/rails/rails17.png)

找不到 show 方法

我们定义一下show方法，并且添加相应模板

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb
[root@h202 blog]# cat app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  def new
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     @article.save
     redirect_to @article
  end
  def show
    @article = Article.find(params[:id])
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]# vim app/views/articles/show.html.erb
[root@h202 blog]# cat app/views/articles/show.html.erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
[root@h202 blog]#
~~~

再次加载

![rails18.png](/images/rails/rails18.png)
 
可以成功显示了

---

### 列出所有文章 

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb
[root@h202 blog]# cat app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  def new
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     @article.save
     redirect_to @article
  end
  def show
    @article = Article.find(params[:id])
  end
  def index
    @articles = Article.all
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]# vim app/views/articles/index.html.erb
[root@h202 blog]# cat app/views/articles/index.html.erb
<h1>Listing articles</h1>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
    </tr>
  <% end %>
</table>
[root@h202 blog]# 
~~~

访问 **`/articles`**

![rails19.png](/images/rails/rails19.png)


---

### 添加链接


~~~
[root@h202 blog]# vim app/views/articles/index.html.erb
[root@h202 blog]# cat app/views/articles/index.html.erb
<h1>Link test!!!!</h1>
<%= link_to 'My Blog', controller: 'articles' %>

<%= link_to 'New article', new_article_path %>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
    </tr>
  <% end %>
</table>
[root@h202 blog]# vim app/views/articles/show.html.erb 
[root@h202 blog]# cat app/views/articles/show.html.erb 
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<%= link_to 'Back', articles_path %>
[root@h202 blog]# vim app/views/articles/new.html.erb 
[root@h202 blog]# cat app/views/articles/new.html.erb 
<h1>Test blog http://soft.dog/</h1>

<%= form_for :article, url: articles_path do |f| %>

  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
 
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
 
  <p>
    <%= f.submit %>
  </p>
<% end %>

 
<%= link_to 'Back', articles_path %>
[root@h202 blog]#
~~~


列表页面多出来两个链接，点击 【New article】

![rails20.png](/images/rails/rails20.png)

成功跳转到了添加页面，随便输入点什么，提交

![rails21.png](/images/rails/rails21.png)

自动跳转到了显示页面，点击【Back】

![rails22.png](/images/rails/rails22.png)

跳转回了所有列表页面

![rails23.png](/images/rails/rails23.png)


> **Tip:** 之所以每做一次修改都能直接生效，是因为在开发模式下（默认），每次请求 Rails 都会自动重新加载程序，因此修改之后无需重启服务器

---

### 数据验证

我们常常有对输入进行校验的需求，以避免接受到了无效或不合规范的数据

~~~
[root@h202 blog]# vim app/models/article.rb
[root@h202 blog]# cat app/models/article.rb
class Article < ActiveRecord::Base
  validates :title, presence: true, length: { minimum: 5 }
end
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
    @article = Article.new
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     
     if @article.save
       redirect_to @article
     else
       render 'new'
     end
  end
  def show
    @article = Article.find(params[:id])
  end
  def index
    @articles = Article.all
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]# vim app/views/articles/new.html.erb 
[root@h202 blog]# cat app/views/articles/new.html.erb 
<h1>Test blog http://soft.dog/</h1>

<%= form_for :article, url: articles_path do |f| %>

 <% if @article.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(@article.errors.count, "error") %> prohibited
      this article from being saved:</h2>
    <ul>
    <% @article.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
  <% end %>


  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
 
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
 
  <p>
    <%= f.submit %>
  </p>
<% end %>

 
<%= link_to 'Back', articles_path %>
[root@h202 blog]#  
~~~

在model中添加了基础的校验逻辑，title 字段不能为空，不能小于5个字符

保存成功就直接显示，如果保存失败，就重绘 new 页面，new 页面中加入了对错误信息的显示

并且将之前的值放到新的窗口中，要求继续输入，准备重新提交

![rails24.png](/images/rails/rails24.png)

---

### 更新文章 


定义 edit 方法，并且创建 edit 视图

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
    @article = Article.new
  end
  def edit
    @article = Article.find(params[:id])
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     
     if @article.save
       redirect_to @article
     else
       render 'new'
     end
  end
  def show
    @article = Article.find(params[:id])
  end
  def index
    @articles = Article.all
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]# vim app/views/articles/edit.html.erb
[root@h202 blog]# cat app/views/articles/edit.html.erb
<h1>Editing article</h1>
 
<%= form_for :article, url: article_path(@article), method: :patch do |f| %>
  <% if @article.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(@article.errors.count, "error") %> prohibited
      this article from being saved:</h2>
    <ul>
    <% @article.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
  <% end %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
 
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
 
  <p>
    <%= f.submit %>
  </p>
<% end %>
 
<%= link_to 'Back', articles_path %>
[root@h202 blog]#
~~~

定义 update 方法，并且添加 edit 链接和 show 链接

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
    @article = Article.new
  end
  def edit
    @article = Article.find(params[:id])
  end
  def update
    @article = Article.find(params[:id])
    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     
     if @article.save
       redirect_to @article
     else
       render 'new'
     end
  end
  def show
    @article = Article.find(params[:id])
  end
  def index
    @articles = Article.all
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]# vim app/views/articles/index.html.erb 
[root@h202 blog]# cat app/views/articles/index.html.erb 
<h1>Link test!!!!</h1>
<%= link_to 'My Blog', controller: 'articles' %>

<%= link_to 'New article', new_article_path %>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
    </tr>
  <% end %>
</table>
[root@h202 blog]#
[root@h202 blog]# vim app/views/articles/show.html.erb 
[root@h202 blog]# cat app/views/articles/show.html.erb 
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<%= link_to 'Back', articles_path %>
| <%= link_to 'Edit', edit_article_path(@article) %>
[root@h202 blog]# 
~~~


![rails25.png](/images/rails/rails25.png)


---

### 删除文章


在 controllers 中定义 destory 方法

然后在 index 视图中加入 Destroy 链接

~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
  def new
    @article = Article.new
  end
  def edit
    @article = Article.find(params[:id])
  end
  def update
    @article = Article.find(params[:id])
    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end
  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    redirect_to articles_path
  end
  def create
#    render plain: params[:article].inspect
#    @article = Article.new(params[:article])
     @article = Article.new(article_params)
     
     if @article.save
       redirect_to @article
     else
       render 'new'
     end
  end
  def show
    @article = Article.find(params[:id])
  end
  def index
    @articles = Article.all
  end
  private
    def article_params
        params.require(:article).permit(:title,:text)
    end
end
[root@h202 blog]# vim app/views/articles/index.html.erb 
[root@h202 blog]# cat app/views/articles/index.html.erb 
<h1>Link test!!!!</h1>
<%= link_to 'My Blog', controller: 'articles' %>

<%= link_to 'New article', new_article_path %>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
      <td><%= link_to 'Destroy', article_path(article),method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</table>
[root@h202 blog]# 
~~~

![rails26.png](/images/rails/rails26.png)

点击 【Destroy】 后会根据我们的定义弹出提示

![rails27.png](/images/rails/rails27.png)

![rails28.png](/images/rails/rails28.png)

连续删除几次后所剩无几

![rails29.png](/images/rails/rails29.png)



目前已经通过 Rails 实现了文章的 **新建、显示、列出、更新、删除** 操作


---

# 命令汇总

* **`rails server -b 0.0.0.0`**
* **`rails generate controller welcome index`**
* **`cat app/views/welcome/index.html.erb`**
* **`vim config/routes.rb`**
* **`grep -v " #" config/routes.rb | grep -v "^$"`**
* **`rake routes`**
* **`bin/rails g controller articles`**
* **`cat app/controllers/articles_controller.rb`**
* **`cat app/views/articles/new.html.erb`**
* **`bin/rake routes`**
* **`cat app/controllers/articles_controller.rb`**
* **`bin/rails generate model Article title:string text:text`**
* **`cat db/migrate/20160422140912_create_articles.rb`**
* **`cat app/models/article.rb`**
* **`bin/rake db:migrate`**
* **`cat config/database.yml`**
* **`grep -v "^#" config/database.yml`**
* **`ll db/development.sqlite3`**
* **`cat app/controllers/articles_controller.rb`**
* **`cat app/views/articles/index.html.erb`**
* **`cat app/views/articles/show.html.erb`**
* **`cat app/views/articles/new.html.erb`**
* **`cat app/models/article.rb`**
* **`cat app/controllers/articles_controller.rb`**
* **`cat app/views/articles/new.html.erb`**
* **`cat app/views/articles/edit.html.erb`**
* **`cat app/views/articles/index.html.erb`**


---

[rubyonrails]:http://rubyonrails.org/
[getting_started]:http://guides.rubyonrails.org/getting_started.html
[getting_started_cn]:http://guides.ruby-china.org/getting_started.html
[rails_basic]:http://soft.dog/2016/04/22/rails-basic/

