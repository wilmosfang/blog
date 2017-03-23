---
layout: post
title: 21分钟搭建一个GitHub Blog
author: wilmosfang
tags:  git jekyll 
categories:  git
wc: 101 116 2872
excerpt: 申请 github 账号，fork 出一个blog ，定制 blog , 添加新博文
comments: true
---

## 前言

喜欢写Blog的人，会经历三个阶段。

* 第一阶段，刚接触Blog，觉得很新鲜，试着选择一个免费空间来写。
* 第二阶段，发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。
* 第三阶段，觉得独立博客的管理太麻烦，最好在保留控制权的前提下，让别人来管，自己只负责写文章。

现在就根据我在网上收集到的资料，直接帮你进入第三个阶段

---

# 概要

* TOC
{:toc}


---


## 申请创建一个github帐号

21世纪的公民都应该具备一项基本技能，那就是申请帐号

细节我就不详叙了，这是github的地址：
[GitHub](https://github.com/)

GitHub是个好东西，自由且免费，还有一个所有穷屌丝爱死的特性，没容量上限，不要玩坏了哦！！



---


## fork出一个现有的Blog

如何最快地获得成果？

那就是‘剽窃’别人的胜利果实

当然也可以高尚一点叫作‘分享’，更何况被剽窃者相当乐意与你一起分享呢^_^

有一种哲学叫作：不要去重复发明轮子！

有个项目叫：[StrayBirds](https://github.com/minixalpha/StrayBirds/tree/gh-pages)

进入后，右上角有一个fork，点击fork出一份代码到自己的github中

访问http://${your-github-account}.github.io/StrayBirds/

其中${your-github-account}要替换成自己注册的github帐号

有没有成就感？你已经花费不到21分钟完成了这一原来看似艰巨的任务

其实此刻你已经拥有了一个完全属于自己的Blog

---

## 定制化Blog

虽然已经拥有了一个属于自己的Blog，但是看起来还是像是别人的，接下来进行定制化，让它看起来是自己的


* 1.将_config.yml 中的 username 修改为你的用户名 minixbeta (此处应该是你自己的账户名)
* 2.将_config.yml 中的 baseurl 修改为 /blog (或者其它任何你认为炫酷的名字)
* 3.将_config.yml 中的 title 修改为 我的博客 (或者其它任何你认为逼格的title)
* 4.在项目的 Setting 中将 Repository name 从 StrayBirds 修改为 blog 

由于整个Blog网站的源码都在你手上，你可做更多更深入的修改与定制


---

## 添加新博文

在 `_post` 目录下添加形如 `2015-03-01-title.md` 的文章，用 markdown 格式

markdown是一个有待驯服的小野兽，好待它实在是很简单

告诉你一个快速学习的方法：

两个窗口打开同一个demo，一边是效果，一边是源码，盯着看十分钟，你就懂了！！

撰写博客

你可以记下任何自己觉得值得与人分享的东西



