---
layout: post
title:  Rails 构建评论功能
author: wilmosfang
categories:   ruby
tags:   ruby rails
wc: 795 1720 19638
excerpt:  使用 ruby on rails 的框架构建评论功能，模型的添加删除，model 的迁移，关联评论，添加路由，生成 controller，代码重构，删除评论，删除关联，基本认证
comments: true
---



# 前言

**[Rails][rubyonrails]** 是使用 Ruby 语言编写的网页程序开发框架

通过集成开发者需要的常用组件，极大地简化了网页程序的开发

> **Tip:** 类似于 python 的 Django ，perl 的 Dancer


继前面的 **[Rails MVC 和 CRUD][rails_crud]** ，这里再进一步添加一个评论功能


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

## 添加删除模型


**`rails`** 命令可以方便的添加删除模型

~~~
[root@h202 blog]# rails --help 
Usage: rails COMMAND [ARGS]

The most common rails commands are:
 generate    Generate new code (short-cut alias: "g")
 console     Start the Rails console (short-cut alias: "c")
 server      Start the Rails server (short-cut alias: "s")
 dbconsole   Start a console for the database specified in config/database.yml
             (short-cut alias: "db")
 new         Create a new Rails application. "rails new my_app" creates a
             new application called MyApp in "./my_app"

In addition to those, there are:
 destroy      Undo code generated with "generate" (short-cut alias: "d")
 plugin new   Generates skeleton for developing a Rails plugin
 runner       Run a piece of code in the application environment (short-cut alias: "r")

All commands can be run with -h (or --help) for more information.
[root@h202 blog]# rails generate model Comment commenter:string body:text
Running via Spring preloader in process 3716
      invoke  active_record
      create    db/migrate/20160427081218_create_comments.rb
      create    app/models/comment.rb
      invoke    test_unit
      create      test/models/comment_test.rb
      create      test/fixtures/comments.yml
[root@h202 blog]# 
[root@h202 blog]# rails destroy model Comment 
Running via Spring preloader in process 3763
      invoke  active_record
      remove    db/migrate/20160427081218_create_comments.rb
      remove    app/models/comment.rb
      invoke    test_unit
      remove      test/models/comment_test.rb
      remove      test/fixtures/comments.yml
[root@h202 blog]#
~~~

---

## 添加一个评论模型



~~~
[root@h202 blog]# rails generate model Comment commenter:string body:text article:references
Running via Spring preloader in process 3787
      invoke  active_record
      create    db/migrate/20160427082552_create_comments.rb
      create    app/models/comment.rb
      invoke    test_unit
      create      test/models/comment_test.rb
      create      test/fixtures/comments.yml
[root@h202 blog]# cat db/migrate/20160427082552_create_comments.rb
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.string :commenter
      t.text :body
      t.references :article, index: true, foreign_key: true

      t.timestamps null: false
    end
  end
end
[root@h202 blog]# cat app/models/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :article
end
[root@h202 blog]# cat test/models/comment_test.rb
require 'test_helper'

class CommentTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
[root@h202 blog]# cat test/fixtures/comments.yml
# Read about fixtures at http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html

one:
  commenter: MyString
  body: MyText
  article_id: 

two:
  commenter: MyString
  body: MyText
  article_id: 
[root@h202 blog]#
~~~

这里产生了四个文件：

File name| Comment
-------- | ---
db/migrate/20160427082552_create_comments.rb|comment表的迁移文件，用于在数据库里产生表结构
app/models/comment.rb| 模型文件
test/models/comment_test.rb|测试文件
test/fixtures/comments.yml| 测试使用的配置，数据或内容


**belongs_to :article** 建立了与 **article** 模型的关联


---

## 进行迁移

这个过程在数据库中生成表结构

~~~
[root@h202 blog]# rake db:migrate
== 20160427082552 CreateComments: migrating ===================================
-- create_table(:comments)
   -> 0.0035s
== 20160427082552 CreateComments: migrated (0.0036s) ==========================

[root@h202 blog]#
~~~

---

## 关联评论

评论在创表的过程中已经构建了与article 的关联，但是article并没与评论关联

调整一下article的model

~~~
[root@h202 blog]# vim app/models/article.rb 
[root@h202 blog]# cat app/models/article.rb 
class Article < ActiveRecord::Base
  has_many :comments
  validates :title, presence: true, length: { minimum: 5 }
end
[root@h202 blog]# 
~~~

---

## 添加路由


~~~
[root@h202 blog]# vim config/routes.rb 
[root@h202 blog]# grep -v " #" config/routes.rb | grep -v "^$"
Rails.application.routes.draw do
  resources :articles do 
    resources :comments
  end
  root 'welcome#index'
end
[root@h202 blog]#
~~~

> **Tip:** 将 **comments** 嵌入到 **articles** 中


---

## 生成控制器


~~~
[root@h202 blog]# rails generate controller Comments
Running via Spring preloader in process 3855
      create  app/controllers/comments_controller.rb
      invoke  erb
      create    app/views/comments
      invoke  test_unit
      create    test/controllers/comments_controller_test.rb
      invoke  helper
      create    app/helpers/comments_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/comments.coffee
      invoke    scss
      create      app/assets/stylesheets/comments.scss
[root@h202 blog]# cat app/controllers/comments_controller.rb
class CommentsController < ApplicationController
end
[root@h202 blog]# cat app/views/comments
cat: app/views/comments: Is a directory
[root@h202 blog]# ll app/views/comments
total 0
[root@h202 blog]# cat test/controllers/comments_controller_test.rb
require 'test_helper'

class CommentsControllerTest < ActionController::TestCase
  # test "the truth" do
  #   assert true
  # end
end
[root@h202 blog]# cat app/helpers/comments_helper.rb
module CommentsHelper
end
[root@h202 blog]# cat app/assets/javascripts/comments.coffee
# Place all the behaviors and hooks related to the matching controller here.
# All this logic will automatically be available in application.js.
# You can use CoffeeScript in this file: http://coffeescript.org/
[root@h202 blog]# cat app/assets/stylesheets/comments.scss
// Place all the styles related to the Comments controller here.
// They will automatically be included in application.css.
// You can use Sass (SCSS) here: http://sass-lang.com/
[root@h202 blog]# 
~~~


File name| Comment
-------- | ---
app/controllers/comments_controller.rb|Comments 控制器文件
app/views/comments| 控制器的视图存放在这个文件夹里，目前是空的
test/controllers/comments_controller_test.rb|控制器测试文件
app/helpers/comments_helper.rb|视图帮助方法文件
app/assets/javascripts/comments.coffee| 控制器的 CoffeeScript 文件
app/assets/stylesheets/comments.scss|控制器的样式表文件


---

## 修改视图和控制器

~~~
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

<h2>Add a comment:</h2>
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>


<%= link_to 'Back', articles_path %>
| <%= link_to 'Edit', edit_article_path(@article) %>
[root@h202 blog]# 
[root@h202 blog]# vim app/controllers/comments_controller.rb 
[root@h202 blog]# cat app/controllers/comments_controller.rb 
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end
 
  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
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

<h2>Comments</h2>
<% @article.comments.each do |comment| %>
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>
 
  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>
<% end %>


<h2>Add a comment:</h2>
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>


<%= link_to 'Back', articles_path %>
| <%= link_to 'Edit', edit_article_path(@article) %>
[root@h202 blog]# 


~~~



---



## 评论效果


多出来两个文本输入框

![rails30.png](/images/rails/rails30.png)

随便输入点内容，进行提交

![rails31.png](/images/rails/rails31.png)

![rails32.png](/images/rails/rails32.png)


---

## 代码重构

如果程序中重复代码达到一定量级，会影响可读性和可维护性，这时我们可以将其中重复部分抽出来，单独成块

~~~
[root@h202 blog]# vim app/views/comments/_comment.html.erb
[root@h202 blog]# cat app/views/comments/_comment.html.erb
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>
 
  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>

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

<h2>Comments</h2>
<%= render @article.comments %>


<h2>Add a comment:</h2>
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>


<%= link_to 'Back', articles_path %>
| <%= link_to 'Edit', edit_article_path(@article) %>
[root@h202 blog]# 
~~~

再次访问，显示效果不变


![rails32.png](/images/rails/rails32.png)


再将评论的表单也抽出

~~~
[root@h202 blog]# vim app/views/comments/_form.html.erb
[root@h202 blog]# cat app/views/comments/_form.html.erb
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
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

<h2>Comments</h2>
<%= render @article.comments %>

<h2>Add a comment:</h2>
<%= render "comments/form" %>

<%= link_to 'Back', articles_path %>
| <%= link_to 'Edit', edit_article_path(@article) %>
[root@h202 blog]#
~~~

再次刷新访问，显示效果不变


![rails32.png](/images/rails/rails32.png)

---

## 删除评论


在comment视图中添加一个删除链接

然后触发Comment 模型进行删除操作

~~~
[root@h202 blog]# vim app/views/comments/_comment.html.erb 
[root@h202 blog]# cat app/views/comments/_comment.html.erb 
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>
 
  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>

<p>
  <%= link_to 'Destroy Comment', [comment.article, comment],
               method: :delete,
               data: { confirm: 'Are you sure?' } %>
</p>
[root@h202 blog]# vim app/controllers/comments_controller.rb 
[root@h202 blog]# cat app/controllers/comments_controller.rb 
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end
 
  def destroy
    @article = Article.find(params[:article_id])
    @comment = @article.comments.find(params[:id])
    @comment.destroy
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
[root@h202 blog]# 
~~~


![rails33.png](/images/rails/rails33.png)


![rails34.png](/images/rails/rails34.png)


![rails35.png](/images/rails/rails35.png)


![rails36.png](/images/rails/rails36.png)


---

## 删除关联评论


如果一篇文章删除了，其中的评论也应该一并删除，可以使用 **dependent** 来实现需求

~~~
[root@h202 blog]# vim app/models/article.rb 
[root@h202 blog]# cat app/models/article.rb 
class Article < ActiveRecord::Base
  has_many :comments, dependent: :destroy
  validates :title, presence: true, length: { minimum: 5 }
end
[root@h202 blog]# 
~~~


---

## 安全

 对文章的修改加入基础认证


~~~
[root@h202 blog]# vim app/controllers/articles_controller.rb 
[root@h202 blog]# cat app/controllers/articles_controller.rb 
class ArticlesController < ApplicationController
###basic auth
  http_basic_authenticate_with name: "soft", password: "dog", except: [:index, :show]

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
[root@h202 blog]# head -n 4 app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
###basic auth
  http_basic_authenticate_with name: "soft", password: "dog", except: [:index, :show]

[root@h202 blog]# 

~~~


对评论的删除加入基础认证


~~~
[root@h202 blog]# vim app/controllers/comments_controller.rb 
[root@h202 blog]# cat app/controllers/comments_controller.rb 
class CommentsController < ApplicationController
###basic auth
  http_basic_authenticate_with name: "soft", password: "dog", only: :destroy
  
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end
 
  def destroy
    @article = Article.find(params[:article_id])
    @comment = @article.comments.find(params[:id])
    @comment.destroy
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
[root@h202 blog]#  head -n 4 app/controllers/comments_controller.rb
class CommentsController < ApplicationController
###basic auth
  http_basic_authenticate_with name: "soft", password: "dog", only: :destroy
  
[root@h202 blog]# 
~~~

这时直接添加或修改文章和删除评论都会触发认证


![rails37.png](/images/rails/rails37.png)


致此，一个可以进行文章增删改查，增减评论，又有基本认证的简单博客系统就搭建起来了

虽然这只是一个小小的demo，但不得不说，ruby on rails 的开发效率是很高效的，原因是大部分本来需要手动完成的事情，这个框架已经帮忙自动完成了，我们需要做的只剩下去填补最基本的对象定义，逻辑关系，展示方式

这个流程是绝大多数管理后台的开发过程，使用rails，竟然只用两篇博客就讲清楚了



---

# 命令汇总

* **`ruby -v`**
* **`gem -v`**
* **`rails --version`**
* **`node -v`**
* **`rvm -v`**
* **`rails server -b 0.0.0.0`**
* **`rails --help`**
* **`rails generate model Comment commenter:string body:text`**
* **`rails destroy model Comment`**
* **`rails generate model Comment commenter:string body:text article:references`**
* **`cat db/migrate/20160427082552_create_comments.rb`**
* **`cat app/models/comment.rb`**
* **`cat test/models/comment_test.rb`**
* **`cat test/fixtures/comments.yml`**
* **`rake db:migrate`**
* **`cat app/models/article.rb`**
* **`vim config/routes.rb`**
* **`grep -v " #" config/routes.rb | grep -v "^$"`**
* **`rails generate controller Comments`**
* **`cat app/controllers/comments_controller.rb`**
* **`cat app/views/comments`**
* **`cat test/controllers/comments_controller_test.rb`**
* **`cat app/helpers/comments_helper.rb`**
* **`cat app/assets/javascripts/comments.coffee`**
* **`cat app/assets/stylesheets/comments.scss`**
* **`cat app/views/articles/show.html.erb`**
* **`cat app/controllers/comments_controller.rb`**
* **`cat app/views/articles/show.html.erb`**
* **`cat app/views/comments/_comment.html.erb`**
* **`cat app/views/articles/show.html.erb`**
* **`cat app/views/comments/_form.html.erb`**
* **`cat app/views/articles/show.html.erb`**
* **`cat app/views/comments/_comment.html.erb`**
* **`cat app/controllers/comments_controller.rb`**
* **`cat app/models/article.rb`**
* **`cat app/controllers/articles_controller.rb`**
* **`head -n 4 app/controllers/articles_controller.rb`**
* **`cat app/controllers/comments_controller.rb`**
* **`head -n 4 app/controllers/comments_controller.rb`**




---

[rubyonrails]:http://rubyonrails.org/
[getting_started]:http://guides.rubyonrails.org/getting_started.html
[getting_started_cn]:http://guides.ruby-china.org/getting_started.html
[rails_crud]:/2016/04/24/rails-crud/


