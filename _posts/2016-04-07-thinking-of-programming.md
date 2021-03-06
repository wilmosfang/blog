---
layout: post
title:  一个运维人员的编程思维
author: wilmosfang
categories:    thinking 
tags:    thinking ruby python php perl  
wc: 557 662 29201
excerpt: 运维人员的优秀品质，懒惰，编码能力，编程思维，运维指导思想，程序到底是什么，数据到底是什么，程序能干什么，工具意识，DRY，流的思想 
comments: true
---




# 前言


作为一个运维人员，虽然不必像开发一样得精通一门或好几门语言，但是基本的编码能力还是要有的，如果懂得一些基础的编程技巧，就可以给自己的日常工作省不少事儿，一些重复性的工作也可以交由代码来完成，使自己的工作不必那么枯燥，同时也少了很多潜在的风险，因为相对于机器，人的速度太慢了，人并不擅于处理重复性的工作，人也更容易出错

---

## 懒惰

**我一直都觉得懒惰是一个运维工程师应该具备的优秀品质**

一个优秀的运维工程师应该有大量的闲暇来思考和优化现有的技术架构，学习先进的技术与理念，更透彻地理解业务需求，让技术架构能够更好的服务于当前和未来的业务发展，而不是频繁被眼前的琐事打断，视野局限在一个非常狭窄的范围

那如何才能有大量的闲暇呢？

将事情交给机器做，然后通过持续优化现有系统架构，就可以逐步脱离疲于奔命的处境

---

## 编码能力

**想懒惰，首先得付出一点点勤奋将自己打磨得具备懒惰的能力**

而这种能力就是编码能力，有了编码能力，机器就会乖乖听话，按照 **寡人** 的旨意，唯命是从

一般而言运维常用到的会是shell、perl、python、ruby

它们有一个共同特点，就是都属于解释型语言，解释性语言是在运行的时候才将程序翻译成机器语言，相较于编译型语言(C，C++，Golang)要慢至少一个量级，但是绝大部分的运维场景中，对于速度的要求并没那么苛刻(远远比不上一个客户端或应用服务对响应的要求，响应速度严重影响 **用户体验** )，甚至基于当前主流的软硬件平台我们基本感知不到两者之间的明显差异，然而解释型语言的简洁和灵活却给运维带来了很多便利

---

## 编程思维

在这里我也并不准备就编译型和解释型展开太多，也不想就哪一种运维常用到的语言进行深入的剖析，相关的网站和书籍多的是，比我讲的更专业，这里我只想分享一下一个运维人员的编程思维


> **Tip:** 当然并不代表对开发人员就毫无用处，思维是比较抽象的，大部分情况下是脱离具体应用场景的，因为它来自于更广泛的观察，正因如此，也可以反过来指导更广泛的生产实践


---


# 概要

* TOC
{:toc}



---

## 总体方向


下面是总体的进阶方向

**复杂的事情简单化，简单的事情重复化，重复的事情自动化**

完成了上面的，就基本满足要求了

但是要想更进一层就涉及到 **可视化** 和 **智能化**


---

### **复杂的事情简单化** 

主旨就是对于一些难以一蹴而就的事情，进行分步处理

**将事情拆解成小的、简单的步骤后往往就变得可行**

**怎样将一头大象装进冰箱里？** 就是一个很好的问题，不要习惯性的反弹回来：“这怎么可能！！！”，而是先初步将这个任务进行一下分解

分解方式因人而异，并无定法，主要以自己或团队最为高效方便的姿势来拆解，当然也可以参考一些最佳实践，这个过程就是在提升其总体的操作可行性

这里分成三个步骤：

* 第一步，打开冰箱门
* 第二步，把大象装进去
* 第三步，关上冰箱门

第二步不明显扯淡么，对的，不过先不要反弹回来嘛，其实第二步又可以当成 **一头大象** ，将家用冰箱更换成集装箱型的工业运输冰箱，或仍然使用家用冰箱，但将大象拆解，都是可以尝试的方向

就这么一步步拆解下去，最后就将难以处理的事情，变得可行

现实场景中往往比这个要复杂很多，也可能因为中间步骤的调整牵连到了前后步骤的变动，但这个不重要，重要的是这个思路，拆解问题这个方向是在持续提升一件事情的可行性

> **Tip:** 当然现实中，并非给出了技术可行的解决方案就一定会执行，因为解决方案的制定和实施都会产生成本，更高层面的管理者眼里还有一个经济可行性的考量，当然这就扯远了，今天只论技术思想

运维中很好体现这一思想的就是Ansible的Playbook

下面是一个安装 apache http server的示例：

~~~
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: name=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running (and enable it at boot)
    service: name=httpd state=started enabled=yes
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
~~~

总体而言就是将一个模糊的安装需求，分解成了一个个有条理的可以执行的操作，完成这些操作后，整体的安装就完成了

> **Tip:** 这里不就细节展开，对Playbook感兴趣可以参考 **[Intro to Playbooks][playbooks_intro]**


---

### **简单的事情重复化**

如何将一个简单的事情变得可重复呢，方法就是将简单的操作标准化，以便于反复调用或反复执行

标准的意义在于统一规范后，对接成本变低，为更大规模更大范围的协作带来了可能，同时尽量避免了个体的不确定性给系统带来的潜在隐患


>因为这篇主要讲思想，所以我得扯远一点，拿一点和运维看起来没太多直接关联的例子来说明 **标准** 的意义
>
>秦始皇的伟大在于他统一了度量衡统一了文字，改革开放有一项重要的举措就是统一了公共交流用语(就是普通话)，为什么计算机技术是当今世界上发展最为迅猛的技术？为什么外国各种组织都醉心于制定各种ISOxxx？
>
>其实仔细想想就会发现，这些基础标准的制定，虽然一定程度上让所谓的 “传统文化” (方言文化或区域文化) 受到很严重的摧残和挤压，诞生了很多 “非物质文化遗产”，但是这些标准构建出了共识，要知道社会的发展归根结底是人与人的协作，当更多人有机会，或能以更低成本参与交流和创造的时候，规模效应发展出更高层次的文明才有了可能，所以这些基础标准事实上将人们整体推向了一个协作发展的快车道
>
>大家应该都有听过下面一句话：
>
> >一流的企业卖标准，二流的企业卖品牌，三流的企业卖产品，四流的企业卖苦力；
> >
> >一流的厂商卖规则，二流的厂商卖技术，三流的厂商卖产品，四流的厂商卖力气
>
>这绝不是一句口号或空话，可以细心品味其中的意义


那又该如何应用在生产实践中呢

* 小团体个体间的潜规则(约定)，公司层面的规章制度，国家层面的法律系统
* A4的大小，火车铁轨的宽度，集装箱的尺寸，USB接口的规格
* 大规模协作的公司都会注重流程
* 关键操作都会有指导手册
* 操作封装成基础工具库，对外提供正确使用工具的文档
* 异构系统间接口的预先定义

太多了，很难穷举，但通过上面几个例子应该可以看到这些实践后面的思想和努力的方向

运维中很好体现这一思想的就是 Docker

如今Docker很火热，而容器技术并非由Docker首创，Docker出现之前容器技术其实已经相对成熟并且被用在了很多生产实践中，而Docker的贡献就在于，将容器范畴的技术进行了封装和标准化，和以前发明集装箱的思路是一样的，集装箱并没有创造出一个从来都没有的东西，而只是规定了一个铁盒子的尺寸和操作方法


> **Tip:** 这里不就细节展开，对Docker感兴趣可以参考 **[Docker][docker]**


---

### **重复的事情自动化**

主旨就是尽量交给机器来完成

这个很明确，人生苦短，如果一件事，被反复要求处理，正好这件事可以交给机器完成，自然可以给自己省下大笔光阴

> **Tip:**  有那么一句话 **人生苦短，我用Python** ，虽然明知本意是在给 Python 打广告，但仍然很有道理呀

人不仅速度慢，还容易出错，也容易有情绪(得克服自己的怠惰，坚持每天感受早晨四点半的洛杉矶才能成就科比，但是机器是可以不睡觉的，被用到报废都不会有怨言)，绝大部分逻辑处理，人的速度是根本没法和机器匹敌的，只要我们可以将它序列化，代码化，就可以自动化

一个合格的运维工程师不会反复人肉生成报表，懒惰的优秀品质会驱使他使用脚本来完成

一个合格的运维工程师不会深夜起床趁业务低点进行数据备份，懒惰的优秀品质会驱使他使用定时脚本来完成

一个合格的运维工程师不会盯着各种日志和性能曲线来关注系统健康状态，懒惰的优秀品质会驱使他使用脚本来触发通知

还有很多我觉得一个合格的运维工程师应该尽量使用脚本而不是手动来完成的，其实这些看起来再正常不过了，但是如果这方面还没有充分涉及到，说明这个运维还有很大的提升空间

想懒惰，首先得付出一点点勤奋来将自己打磨得具备懒惰的能力，而这种能力就是编码能力

> **Tip:** 相较于手动，使用脚本对能力有更高的要求，因为交互式比较直观，使用脚本要求对整个过程和可能产生的情况了然于胸，对处理流程理解更深刻，而人往往喜欢徘徊在自己的能力舒适区，惯性与惰性驱使人拒绝成长


各种语言都可以被用来写脚本，但运维用得较多的主要是 shell、perl、python、ruby (也有用php和js的，但相对小众)

shell准确来说是一个类别，有各种版本，我个人比较喜欢用bash

查看本地shell和当前shell

~~~
[root@h102 ~]# cat /etc/shells 
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/tcsh
/bin/csh
/usr/bin/tmux
[root@h102 ~]# echo $SHELL
/bin/bash
[root@h102 ~]#
~~~

shell 结合 crontab ，sed，awk，grep，正则还有管道就已经可以应付绝大部分的日常处理

但是要进行更灵活和复杂的逻辑处理 shell 就有些力不从心了(不是不能完成，只是会很啰嗦)

perl 有强大的文本处理能力，即便是一行 perl 脚本也可以完成相当复杂的处理，一般我会把一些常用到的写出来后，收集保存起来，以便下次再用

python 不得不说是目前最为主流的运维脚本语言，各种库都非常丰富，拿来就能用，省力又省心

ruby 是一门懒人都会喜欢的语言，因为真的很方便，个人感觉，它的每一个对象都有十八般武艺，信手拈来就能用，我们可以花更多时间在思考要什么，而不是如何获取

其它语言也有应用场景，总体来说对于一个运维人员，实现一个功能，哪种方便就用哪个，因为它们都只是实现自动化的一种工具，对于某一种工具太过偏执而浪费了时间就得不偿失了

我并不是一个语言专家，也并不打算成为一个语言专家，所以也不会去分享各种语言之间相互区别的独道特性，在这里分享的只是我的思想，以上语言我都有接触，总体思路就是：完成特定任务哪种语言更容易实现就用哪个，有时甚至会穿插使用，我不是一个有 **纯种语言代码洁癖** 的人


---

### **可视化**

主旨就是尽量将数据和结果进行图像化展示

人类在漫长的进化历程中，对于视觉信号的处理能力远远强于文字符号的处理能力

>因为文字符号的意义需要翻译和理解，并且是在人类有了文明之后才开发的能力，而人类这一系物种进化出了眼睛(或更早的光感神经)后，就一直在接受和加工处理视觉信号，**百闻不如一见** 就说明了人们可以在看到一的瞬间就获取极大量的信息，关于趋势和规律如果是直接从海量的数字中获得，会很难懂很费解，但如果图形化后，就能 **一目了然** 

数据可视化是一种统领海量数据的有效方法

各种监控的图形化展示(dashboard)就是最好的应用

推荐使用一款叫 Gnuplot 的图形生成软件，可以将数据处理成想要的展示形式

这里只是show一下它的展示能力，它可以根据基础数据简单高效地生成各种图像形式

~~~
gnuplot> f(x,y)=sin(sqrt(x*x+y*y))/sqrt(x*x+y*y)
gnuplot> splot f(x,y)
gnuplot> set isosamples 100
gnuplot> set xyplane 0.2
gnuplot> replot
gnuplot>
~~~


![gnuplot.png](/images/gnuplot/gnuplot.png)


---

### **智能化**

是不是可视化就到了最高境界了，其实还没完

目前在我看来，比可视化更高一层的境界就是 **智能化**

当前的实现方式就是大数据分析，大数据分析是一种通过过去和现在，知道未来的一种方法

或者对于自己和环境的过去和现在进行更深层理解以支持决策，或自动决策的一种方法

Growth hacking 就是一个很典型的例子，通过关键动作的大数据分析，和AB测试以数据来驱动增长

因为我也在学习的过程中，所以只能提供思想层面的东西，给不了特别具体的应用案例

> **Tip:** 其实人的经验就是大数据分析的一种，大数据不求给出精准的答案，只求能给出一种概率或明确的倾向，以便作出更好的决策


---

## 程序


既然是讲一个运维人员的编程思维，那就回到程序这个核心概念(脚本也是程序的一种)

我们花那么多时间精力是要整出一个什么玩意儿

---

### 概念


什么是程序？

>程序（Program）是为实现特定目标或解决特定问题而用计算机语言编写的命令序列的集合

这貌似一个再简单不过的问题，但这里我还是想分享一下自己的理解，在此仅代表一家之言(不过欢迎与我交流和探讨)

上面的定义绝对没错，但是视野却过于狭窄，运维人员头脑中永远都要有宏观的系统观和架构观，系统永远都不是静止的，而上面的文字却将程序定义为了一具尸体(一个标本)，不能与其它组件交互的程序是没有任何意义的

我也不打算重新给出自己的定义去推翻任何哪个权威的定义(语言总有它描述不到的地方，于是沦为了口水战)，我只是尝试将程序放回到它的活动场景中看看它们  **到底是什么，到底在干什么**

---

###  程序到底是什么

计算机只能存储和处理代表0和1的电位序列(可以高0低1，也可以高1低0)，其实哪些代表数据，哪些代表对数据的处理，计算机也不知道，它自己完全没有能力分辨，而这些都是由人来指定的

>**数据** 和 **指令** (**对数据的处理方式**) 有什么本质区别么，并没有，在机算机眼里，都不过是一串电位序列，它们所代表的意义是由人来指定的，人会以电位序列的方式指定计算机如何使用自己的逻辑门或哪一批逻辑门对当前寄存器中的这串序列(另一串)进行加工，然后将产生的结果放到哪里(所谓的 **哪里** ，其实又是一串序列标定的寄存器地址)，在计算机的世界里一切皆电位序列

它们的本质都是数据，只是其中一部分被人为指定成被加工的对象，另一部分被人为映射成CPU里的加工方法(CPU载入了这串序列后，就被驱动得应用对应的逻辑门，CPU绝对是被动的)

那么这里程序(准确来说是写着操作序列的文档)就分化为了两部分 ：**数据** 和 **加工方法**

其实仔细想想，目前为止的所有编程语言(机器语言、汇编语言，高级语言，不论是编译型还是解释型语言)无不是在围绕这两类进行优化和调整，不断重组，以期带来能更高效利用有限计算资源的方法(编程人员的人力资源也囊括在内)

---

### 程序到底在干什么

其实很简单： **就是输入数据，加工数据然后输出数据**

> **Tip:** 准确来说数据的移动(输入输出或加载返回)也算是数据加工的一种

归根结底， **程序就是在对数据进行加工处理**

我们将 **数据的移动** 和 **数据的加工** 分开来看，是为了便于理解，因为移动要求除了位置不能对数据内容产生修改，而加工处理可能产生内容变化

---

### 数据

那么我们平时习以为常的 int，long，float，double，char，boolean，指针，string，array，hash 是不是可以再追问一下，它们真是我们直观感受的那样么？它们到底是什么？

#### 类型

计算机只能存储和处理代表0和1的电位序列，所以上面的数据类型无一例外的都被表示成了01序列，那01序列如何表示成以上各种不同类型呢 ， 如何进行各种计算的呢

* 整数是二进制(01序列)到十进制之间的映射
* 字符是字符编码(01序列)与字符集之间的映射
* 简单的布尔型就是01本身与真假之间的映射
* 指针是二进制(01序列)到寄存器或内存地址之间的映射
* 浮点数的01序列不同区段代表浮点数的不同部分

> **Tip:** 浮点数的表达要复杂很多，得引入数据结构的概念，为了便于表达，各种组织约定指定长度的01序列中哪几位代表正负，哪几位代表指数，哪几位代表尾数，而这些信息组合在一起共同描述了一个浮点数
>
>感兴趣可以参考 **IEEE754** 中单精度二进制浮点数的原理

其它的数据类型都是在此基础上组合而成

其实这里都遵循同一个思路，就是一串01序列，它代表的意义是什么完全是人为来指定的，不同的翻译(或映射方法)就会表达出不同意义，指定错了翻译方法，在人看来就是乱码或产生异常(而计算机很无辜，它都是听话在干活，它都是被动的，当然它也感受不到委屈)


#### 加工

可以表达出数据远远不够，我们的目的是为了使用机器对它们进行计算(加工)，那计算机是如何对一串01序列进行计算的呢

计算机可以通过电子(或晶体)二级管和三级管的组合对高低电位进行与、或、非的逻辑处理，而所有的加减乘除都可以转化为与、或、非的逻辑处理，其它的处理又可以转化为加减乘除的处理，如果可以将数据转化为高低电位一切就迎刃而解了，正好常见的数据都可以人为约定成01序列，01序列又可以转化为高低电位，这样一切就顺理成章了，计算机就是这样加工和处理数据的

---

### 有什么卵用

上面说了那么多有什么卵用？ 根本感觉不到这些对生产有什么指导意义呀， 我只是为了进行一下科普么？

（感觉像是废话，因为你可能全都懂！！！）

绝对不是的，对事物的透彻认知有助于我们拨开浮云，直击本质，不必被那么多的概念弄得晕头转向(在写代码的过程中时刻都有着清晰的思路和方向)

冗述那么多我到底想说什么，我们得知道计算机其实有多么单纯，它只能干两件事：**表达数据** ， **加工数据**

带上这副眼镜，我们再来看看各种编程语言中的各种概念：

什么叫 **数据结构** ？ =>  数据的约定与表达方式

什么叫 **算法** ？  =>  数据的加工方法

什么叫 **数据类型** ？ =>  数据的约定与表达方式

什么叫 **变量** ？ =>  数据容身之所(其实是临时的)，可以直接代表数据

什么叫 **属性** ？ =>  数据

什么叫 **传参** ？ =>  传递数据

什么叫 **返回值** ？ =>  数据加工的结果，还是数据

什么叫 **方法** ？  =>  数据的加工步骤

什么叫 **函数** ？ =>  数据的加工步骤

什么叫 **模块** ？ =>  打包好的数据和对数据的加工步骤

什么叫 **面向对象** ？ =>  数据和对数据的加工方法打包在一起，通过数据加工的方式来完成数据传递

什么叫 **面向过程** ？ =>  数据和对数据的加工方法糅合在一起，尝试描述整个流程，遍历每种可能的结果

太多了，无法穷举，但上面的例子已经可以看出一种思想，就是这些无非都是 **数据** 与 **加工方法** (也可以叫 **算法** ) 的各种组合形式而已

给我们编写代码又有怎样的指导意义呢？

我们得分清哪些是待处理的数据，哪些是对这些数据的处理方法，尽要不要糅合在一起

输入(请求)是什么，要进行怎样的处理，输出(响应)是什么

与我们编写的程序进行交互的对象(可以是人也可能是另一个程序)，会有怎样的请求或响应

两个处理过程中对接的数据是使用相同的表达方式么，是否有鸡同鸭讲的可能

可以将一些复杂的处理过程进行封装打包，以便整体调用，逻辑会更清晰

可以将一些会多处用到的处理过程进行封装打包，以方便反复调用

还有很多，都可以基于此两点进行不断生发，并且再也不必拘泥于一种语言了，其它语言都大同小异(具体就是数据表达和加工处理有方言上的区别而已，而方言这种东西更像是肌肉记忆，不是一两百个字可以讲清楚的)

> **Tip:**  是不是感觉有点虚呀，没办法，思想，心法，内功都是这样的，只有用到时才会明白它的好用

---

## 工具


**君子性非异也，善假于物也**

主旨就是要有工具意识

人从最初的状态到今天，是用工具来划分时代的：**石器时代、青铜时代、铁器时代、蒸汽时代、电气时代，信息时代**，可见工具的重要性，作为生产力的基础(另一个是人自身)，工具可以极大改变整体的生产效率和资源分配格局

充分使用现有工具是运维人员必备的基本素质，工具可以极大拓展和提升个体的能力边界

编写脚本就是一个创造工具的过程

---

### DRY

是不是所有的工具脚本都要偏执地亲自来编写？ 

 **DRY(Don't Repeat Yourself)不要重复发明轮子** ，因为使用工具的初衷是为了提升工作效率，编写脚本本身也是一种成本支出(探究语言和编写过程都得花费一定时间)，当这种支出超过一定边界时反而成为了负担(降低了整体工作效率，反而得不偿失)，如果正好有人已经完成了相同或类似的工作，拿来稍作修改就可以用岂不是更好，更符合使用工具的初衷？

**节省了时间就等于拓展了生命容量**

事实上，人就是一种不断重蹈覆辙的动物，很多事情都是在反复发生，表现在同一空间中历史上的不同时刻，或同一时刻空间中不同个体的身上，所以，很多路已经被前人或他人走过，拿过来直接 **借鉴** 就好了嘛，何必亲自去踩一回坑呢

那反映在工作中如何使用呢？

在shell 中对于排序的需求并不必自己写一个排序函数，直接使用sort就可以了，报表和汇总处理可以使用awk，替换可以使用sed ，过滤信息可以使用 grep ，定时执行也不必去循环检查时间，直接使用crontab，自己要做的只是将这些现成的工具拼接起来，处理目标数据，获取想要的结果就可以了

perl 有 cpan ，python 有 pip ，ruby 有 gem

如果登录到这些公共仓库中看一看，就会发现很多要花费大量时间来实现的复杂处理，都已经被人提前实现了，越通用的，越先被完成，我们要做的就是拿过来用就可以了

那是不是单纯依赖上面的成品包就可以不用自己具备编程能力了呢，当然不是，大部分不代表所有，必然有没被提前实现的方法，如果自己碰到了就得亲自操刀，封装好一点，还能回馈开源社区，即便被提前实现，依旧得具备可以正确使用的能力，至少也得知道如何配置调用，并且与现有的代码集成，其实当项目安全等级要求高的时候，还要有代码审查的能力

总而言之，一定程度的编码能力是绕不过的，但是不要事必躬亲，核心思想是充分利用好现有工具，根本目的是提高工作效率


---

## 流

流是集上面所有思想与一体的智慧

很早以前 Unix的设计思想中就包含以下三点：

>* 1.一个程序只做一件事情，并且把它做好
>* 2.程序之间能够协同工作
>* 3.程序处理文本流，因为它是一个通用的接口

其实前两点已经很直白，专注，模块化，松耦合，分工协作，这里不准备详细地展开，但是对于第三点，很多人持有保留意见，主要就是针对 **文本流**，因为它对于人类是友好的，但是对于计算机会产生很多不必要的开销或容易被误用直接引发异常

这里我把它抽象为流，是 **数据流** ，并不局限于 **文本流**

回到程序，我们知道计算机只能干两件事：**数据表达** 和 **数据加工**

它是按照下面的流程进行处理的：

输入数据，处理数据，输出数据

(或 加载数据，加工数据，保存数据)

仔细观察就会发现这个处理流程像极了流水线上的其中一个环节，如果将很多个这样的环节对接起来就成了整条流水线，这条流水线可以随时被重新拼装，加入步骤，减少步骤以实现对任意数据的任意加工处理，这其实就是一个程序做的事情

> **Tip:** 其实一台计算机中不是只有一个地方能对数据进行处理，除了CPU以外，内存，硬盘，raid卡，显示芯片，南桥芯片，北桥芯片，网卡芯片都能对数据进行处理，只不过CPU更通用也可以被任意编程，而其它芯片是定向优化处理过的，只针对具体应用场景或特定功能特性，不能随意被编程

大而化之，可以这么看待一个程序，它接受一个对象(可以是人也可以另一个程序)的操作(请求)，然后进行处理，最后反馈结果(响应)

既然如此，继续扩大范围，我们的系统架构是否也遵循这一思想呢，当然遵循，现的比较流行的 **Restful + 微服务** 架构可以看成一种网状流系统，**SOA** 架构可以看成是星型流系统(由MQ或信息总线来统一调度和管理，这样就更为松耦合，更强扩展性，比较适合大规模和分布式)，还有一些现成的Web框架如 **MVC** ，就是一种星型流的框架，**DDD** 中的 **CQRS** 就是一种线(或环)型流的框架

系统的架构或应用的框架可以故意往线型流的方向去靠拢，也可以不太在意，设计得为更随意(没有最好的架构，得看具体场景下的需求和实现成本)

这里有一个典型遵循线型流思想的软件架构

**ELK** (Elasticsearch , Logstash , Kibana)

![beats-logstash.png](/images/beats/beats-logstash.png)

> **Tip:** 如果对ELK感兴趣，详细内容可以参考 **[ELK][elk]**

其中 **Logstash** 又是一个遵循线型流思想的软件

Logstash 是这样的处理模型

~~~
input threads | filter worker threads | output worker
~~~

![deploy_2.png](/images/logstash/deploy_2.png)



日常的工作中，可以多使用管道，能节省下大笔的时间

我只想获得以M为单位的空余内存大小

~~~
[root@h102 ~]# free -m 
             total       used       free     shared    buffers     cached
Mem:          1869       1269        600          2        213        367
-/+ buffers/cache:        688       1180
Swap:         3999          0       3999
[root@h102 ~]# free -m  | grep Mem | awk '{print $4}'
600
[root@h102 ~]#
~~~


---

# 总结


上面都是一些显而易见的道理，但是从显而易见或司空见惯的事物中挖掘出营养却是一个非常值得努力的方向

因为这些司空见惯的的事物太多了，但我们未必真懂得其中的内涵或蕴藏的智慧，哪怕只深掘一层，将会发现遍地都是宝藏

作为运维，不仅要学习各种招式(层出不穷的新技术)，还要不断修炼自己的内功(持续提炼和总结)，才能逐渐以不变应万变，适应这个日新月异的环境

这条路很长，没有尽头，我依旧在途中，我很乐意将自己看到的风景拿出来与大家分享

纸上得来终觉浅，绝知此事要躬行


---


[playbooks_intro]:http://docs.ansible.com/ansible/playbooks_intro.html
[docker]:https://www.docker.com/
[elk]:https://www.elastic.co/
