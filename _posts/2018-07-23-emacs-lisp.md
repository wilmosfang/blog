---
layout: post
title: "Emacs Lisp"
author:  wilmosfang
date: 2018-07-23 12:54:30
image: '/assets/img/'
excerpt: 'Emacs Lisp 15 分钟入门'
main-class: 'lisp'
color:
tags:
  - lisp
  - emacs
categories:
  - lisp
twitter_text:  'Learn Emacs Lisp in 15 minitus'
introduction:  'Learn Emacs Lisp in 15 minitus'
---

# 前言 #

Lisp 是一门很古老的语言

>LISP 是具有悠久历史的计算机编程语言家族，有独特和完全括号的前缀符号表示法，起源于1958年，是现今第二悠久而仍广泛使用的高级编程语言，只有 FORTRAN 编程语言比它更早一年，LISP 编程语族已经演变出许多种方言，现代最著名的通用编程语种是 Common Lisp 和 Scheme , 以上解释来自 WIKI

Emacs Lisp 是 Lisp 的一个分支

>Emacs Lisp，一种直译式的脚本语言，为LISP的方言之一，GNU Emacs与XEmacs文字编辑器都使用这个编程语言来扩展它们的功能，它的直译器是以C语言来实作的，它受到Maclisp的影响很大，但是跟Common Lisp与Scheme有所不同

最近我迷上了 Emacs 

所以顺藤摸瓜，竟然搭进去了一门语言

>万万没想到，为了了解一个编辑器，竟然搭进去了一门语言

好在这门语言结构比较简单清晰

以致于可以用 15 分钟入个门

>**Tip:** 这一篇纯属于摘抄，不是原创，只是好东西忍不住拿出来分享，如果原作者有意见，可以随时联系我，下线此文章，我完全尊重原作者的意见

原文，请参考 **[Emacs Lisp 15 分钟入门][jobbole]**

-------------------------------------------------------------------------------

## 操作 ##

### 系统环境 ###

~~~
wilmos@Nothing:~$ hostname
Nothing
wilmos@Nothing:~$ hostnamectl 
   Static hostname: Nothing
         Icon name: computer-laptop
           Chassis: laptop
        Machine ID: bba3eb544f8748cf884490544d268807
           Boot ID: 6c0f498d0757429591682b0574cf9e5c
  Operating System: Ubuntu Kylin 16.04.4 LTS
            Kernel: Linux 4.8.0-58-generic
      Architecture: x86-64
wilmos@Nothing:~$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp109s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state DOWN group default qlen 1000
    link/ether 80:fa:5b:3b:d5:f4 brd ff:ff:ff:ff:ff:ff
3: wlp110s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 84:ef:18:f3:92:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.6/24 brd 192.168.1.255 scope global dynamic wlp110s0
       valid_lft 6613sec preferred_lft 6613sec
    inet6 fe80::e6c0:2896:5b4a:1712/64 scope link 
       valid_lft forever preferred_lft forever
4: vboxnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.1/24 brd 192.168.56.255 scope global vboxnet0
       valid_lft forever preferred_lft forever
    inet6 fe80::800:27ff:fe00:0/64 scope link 
       valid_lft forever preferred_lft forever
5: vboxnet1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 0a:00:27:00:00:01 brd ff:ff:ff:ff:ff:ff
wilmos@Nothing:~$ 
~~~

### 软件环境 ###


~~~
wilmos@Nothing:~$ emacs --version
GNU Emacs 26.1
Copyright (C) 2018 Free Software Foundation, Inc.
GNU Emacs comes with ABSOLUTELY NO WARRANTY.
You may redistribute copies of GNU Emacs
under the terms of the GNU General Public License.
For more information about these matters, see the file named COPYING.
wilmos@Nothing:~$ 
~~~


### 代码 ###


直接上代码

跟着代码边学边做

~~~lisp
;; This gives an introduction to Emacs Lisp in 15 minutes (v0.2d)
;;
;; 英文原作者: Bastien / @bzg2 / http://bzg.fr
;; 中文翻译: iamxuxiao
;; 
;; 
;; 如何安装 Emacs 
;; 
;; Debian: apt-get install emacs (or see your distro instructions)
;; MacOSX: http://emacsformacosx.com/emacs-builds/Emacs-24.3-universal-10.6.8.dmg
;; Windows: http://ftp.gnu.org/gnu/windows/emacs/emacs-24.3-bin-i386.zip
;;
;; More general information can be found at:
;; http://www.gnu.org/software/emacs/#Obtaining
;; 免责声明：
;;
;; Going through this tutorial won't damage your computer unless
;; you get so angry that you throw it on the floor. In that case,
;; I hereby decline any responsability. Have fun!

== 启动Emacs, 缓冲区和工作模式==
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 
;; 第一步首先启动Emacs: (在windows中可以双击emacs图标，在Linux中可以输入% emacs & )，
;; 然后在键盘上键入q 跳过系统欢迎的信息，
;; 先观察在Emacs屏幕的底部，会给出一堆关于当前的工作情况的信息，其中灰色的一行叫做状态行，
;; 在其中你会发现 *scratch* 的字样，这表示你当前的缓冲区(buffer)的名字。
;; 缓冲区也叫做工作区，在Emacs中打开一个文件，实际只是在Emacs中构造该文件的一个副本，放到缓冲区中，
;; 在Emacs中对该文件的编辑也是针对该副本的编辑，唯有保存改动时，Emacs才会把缓冲区中的内容在复制到原文件中去。
;; 状态行下面的那行，叫做辅助输入区(minibuffer),该minibuffer用于显示计算结果，以及和用户做交互。
;;
;; 
;; 如何切换Emacs的工作模式 
;; Emacs有各种各样功能各异的模式，工作模式的含义其实就是Emacs对当前的文本编辑工作
;; 更加的敏感，比如高亮和缩进，并且支持一些特殊的命令。
;; 为了实验本教程中的lisp命令，我们要让Emacs工作在lisp-interaction-mode工作模式下，
;; 这个模式可以让我们在缓冲区中和Emacs进行互动，并且直接执行Lisp命令,得到结果。
;; 进入lisp-interaction-mode的方法： 把光标移动到辅助输入区，键入M-x lisp-interaction-mode 
;; 然后回车。

== 表达式，变量和函数 ==

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;
;; 冒号在Lisp中表示注释
;; 在Elisp中做运算，调用函数的最简单的方式是
;; (function arg1 arg2) 
;; 这相当于通常的function(arg1,arg2)，下面的表达式，对两个数字进行加法运算
(+ 2 2)

;; Elisp中表达式可以通过括号来嵌套
(+ 2 (+ 1 1))

;; 在lisp-interaction-mode模式中，我们可以直接计算一个表达式,计算的方法是
(+ 3 (+ 1 2))
;; ^ 把光标放在这里，并且键入Ctrl-j (之后将简写成C-j)
;; C-j是一个快捷命令，在后台，该快捷键将调用求值命令，并且把计算的结果
;; 插入到当前的缓冲区中

;; 如果不希望Emacs在缓冲区中插入计算结果，我们还可以在表达式的末尾使用C-x C-e组合键
;; C-x C-e的意思是: 先按下Ctrl-x 再按下Ctrl-e 
;; 这个命令会让Emacs在辅助缓冲区，也就是Emacs窗口的最底部那行显示计算结果

;; ELisp中的赋值函数是是setq，下面的表达式给变量my-name赋值"Bastien"
(setq my-name "Bastien")
;; ^ 把光标停在这里，再键入C-x C-e

;; 下面insert函数的作用是在光标所在出插入字符Hello
(insert "Hello!")
;; ^ 把光标停在这里，再键入C-x C-e

;; insert函数还可以两个常量字符，比如
(insert "Hello" " world!")

;; insert函数还可以接受变量作为参数，我们之前已经给my-name变量赋过值了
;; 所以下面命令的输出结果是 "Hello, I am Bastien"
(insert "Hello, I am " my-name)

;; defun命令用来定义一个函数,语法是
;; (defun 函数名 (参数列表) (函数体))
(defun hello () (insert "Hello, I am " my-name))
;; ^ 把光标停在这里，再键入C-x C-e 执行defun命令来定义函数
;; 通过defun命令，你已经在Emacs中安装了这个hello函数，这个函数就成为了Emacs的一部分，知道你退出Emacs或者改变hello的定义

;; 从下面开始，我们将不再提醒读者使用C-x C-e来定义函数和执行ELisp指令

;; 在Elisp中直接输入函数的名称就是调用该函数。
;; 下面的命令的输入结果是: Hello, I am Bastien
(hello)

;; 前面定义的hello函数不接受任何参数,过于简单，
;; 现在我们重新定义hello函数，让它接受一个参数name。 
(defun hello (name) (insert "Hello " name))

;; 然后调用新的hello函数，并且提供一个参数。
;; 下面命令的输出结果是"Hello you"
(hello "you")

== progn,let和交互式函数== 
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;
;; 执行switch-to-buffer-other-window命令，将在在一个新的窗口中打开一个buffer
;; 该buffer命名叫做 test, 并且把光标移到新的buffer的窗口中。
(switch-to-buffer-other-window "*test*")

;; 要回到原来的buffer中，可以使用鼠标点击原来的buffer
;; 或者使用组合键 C-x o 
;; C-x o的意思是: 先按下Ctrl-x 再按下o

;; 如果要执行一系列的指令，可以使用流程函数progn，把函数命令连接起来.
;; 下面的命令,先打开一个新的buffer,再执行hello函数，该hello函数的参数是"you"
(progn
(switch-to-buffer-other-window "*test*")
(hello "you"))

;; 如果要清空一个buffer,可以调用erase-buffer函数。下面的命令先清空test buffer,再调用hello函数做打印
(progn
(switch-to-buffer-other-window "*test*")
(erase-buffer)
(hello "there"))

;; 在这一系列的质量后面再添加调用一个other-window函数，这样在hello函数被调用完毕之后
;; 光标自动回到之前的buffer窗口中
(progn
(switch-to-buffer-other-window "*test*")
(erase-buffer)
(hello "you")
(other-window 1))

;; let函数用来做局部变量的定义 下面的一系列命令中
;; let函数首先定义local-name变量的值为“you”
;; 然后接着执行括号中其它的语句块部分，这个功能和progn类似
(let ((local-name "you"))
(switch-to-buffer-other-window "*test*")
(erase-buffer)
(hello local-name)
(other-window 1))

;; format函数可以用做格式化的输出 其中%s表示该s的地方将被之后提供的一个字符串,即visitor替换
;; \n表示换行
(format "Hello %s!\n" "visitor")

;; 现在我们利用format函数来改进之前定义的hello函数
(defun hello (name)
(insert (format "Hello %s!\n" name)))

;; 执行这个函数结果是"Hello you"，并且光标换到下一行
(hello "you")

;; 下面我们再设计一个greeting函数，该函数接受一个参数name,
;; 在函数体的内部又使用了let函数，给一个局部变量your-name赋值
;; 最后把参数和局部变量格式化的打印出来
(defun greeting (name)
(let ((your-name "Bastien"))
(insert (format "Hello %s!\n\nI am %s."
name 
your-name ; 局部变量
))))

;; 执行greeting函数，并提供"you"字符串作为参数
(greeting "you")

;; read-from-minibuffer函数提供和用户交互的功能，这个函数可以帮助Elisp程序从用户处得到输入
(read-from-minibuffer "Enter your name: ")

;; 比如如果我们希望greeting函数能够从用户处得到姓名，并且做打印格式化的欢迎信息。
;; 可以先调用read-from-minibuffer在minibuffer中提示用户输入姓名，
;; 然后把得到的结果赋给局部变量your-name，
;; 最后insert函数在当前buffer中插入格式化的输出
(defun greeting (from-name)
(let ((your-name (read-from-minibuffer "Enter your name: ")))
(insert (format "Hello!\n\nI am %s and you are %s."
from-name ; 格式化输出参数1
your-name ; 格式化输出参数2
))))

;; 执行这个函数
(greeting "Bastien")

;; 再稍加改进greeting 把结果打印在新的buffer中
(defun greeting (from-name)
(let ((your-name (read-from-minibuffer "Enter your name: ")))
(switch-to-buffer-other-window "*test*")
(erase-buffer)
(insert (format "Hello %s!\n\nI am %s." your-name from-name))
(other-window 1)))

;; 执行这个函数
(greeting "Bastien")

== 列表和综合实例 ==

;; Lisp中使用括号构造列表，使用setq给变量赋值。
;; 下面的命令先构造一个列表，再把这个列表赋给list-of-names变量
(setq list-of-names '("Sarah" "Chloe" "Mathilde"))
;; ^这里的单引号表示这是一个列表

;; 如果想要得到列表中的第一个元素，可以使用car函数
(car list-of-names)

;; 如果想要得到列表中的除第一个元素以外的其它元素，可以使用cdr函数
(cdr list-of-names)

;; 以后push函数可以在列表的头部插入新的元素，所以下面的命令将改变list-of-name中元素的个数
(push "Stephanie" list-of-names)

;; mapcar函数对列表中的把列表中的每一个元素分别取出来，赋给hello函数
(mapcar 'hello list-of-names)

;; 重新定义greeting函数，在一个新的，清空的buffer中，对list-of-names列表中的每一个元素，调用hello函数
;; 调用完毕之后，再让光标回到原的buffer中
(defun greeting ()
(switch-to-buffer-other-window "*test*")
(erase-buffer)
(mapcar 'hello list-of-names)
(other-window 1))

;;执行这个函数，我们将得到一个名叫test的buffer，其中的内容是
;; Hello Stephanie!
;; Hello Sarah!
;; Hello Chloe!
;; Hello Mathilde!
;; 暂时先不要关闭这个buffer!后面还有用！ 
(greeting)

;; 下面我们对buffer做一些更有意思的事情！
;; 定义一个replace-hello-by-bonjour函数，顾名思义，就是把hello替换成bonjour
;; 该函数首先把光标移到一个叫做test的buffer中
;; 再把光标移到该buffer的开头
;; 从头开始搜索字符串Hello,并且替换成Bonjour
;; 结束之后在把光标移会到一开始的buffer中。
(defun replace-hello-by-bonjour ()
(switch-to-buffer-other-window "*test*")
(goto-char (point-min)) ;该函数把光标移到buffer的开头
(while (search-forward "Hello")
(replace-match "Bonjour"))
(other-window 1))

;; 其中 (search-forward "Hello") 在当前的buffer中做前向搜索
;; (while x y) 当x 的条件满足时执行y指令 ，当x返回nil时，while循环结束

;; 执行这个函数 替换test buffer中的hello
(replace-hello-by-bonjour)

;; test buffer中的结果如下
;; Bonjour Stephanie!
;; Bonjour Sarah!
;; Bonjour Chloe!
;; Bonjour Mathilde!

;; 在minibuff中，还会有一条错误信息 "Search failed: Hello".
;; 把(search-forward "Hello")一句换成如下就不会有错误信息了
;; (search-forward "Hello" nil t)

;; 其中 nil参数表示 搜索的区域不加限制，直到buffer结束
;; 其中t参数指示search-foward函数 跳过错误信息 直接退出

;; 新hello-to-bonjour如下：
(defun hello-to-bonjour ()
(switch-to-buffer-other-window "*test*")
(erase-buffer)
;; 对list-of-names列表中的每个元素 使用hello函数
(mapcar 'hello list-of-names)
(goto-char (point-min))
;; 搜索Hello替换成Bonjour
(while (search-forward "Hello" nil t)
(replace-match "Bonjour"))
(other-window 1))

;; 执行这个函数
(hello-to-bonjour)

;; 下面的boldify-names 函数 ，
;; 首先把光标挪到名叫test的buffer的开头，
;; 然后使用regular expression 搜索 “Bonjour + 其它任何内容” 的pattern，
;; 然后对找到的字符加粗。 
(defun boldify-names ()
(switch-to-buffer-other-window "*test*")
(goto-char (point-min))
(while (re-search-forward "Bonjour \<span class='MathJax_Preview'>\(.+\\)</span>!" nil t)
(add-text-properties (match-beginning 1) ;返回匹配模式中，最先匹配的位置
(match-end 1) ;返回最后匹配的位置
(list 'face 'bold)))
(other-window 1))

;; 执行这个函数 
(boldify-names)

== 帮助和参考==

;; 在Emacs中我们可以通过如下的方式得到变量和函数的帮助信息
;; C-h v a-variable RET
;; C-h f a-function RET
;;
;; 下面的命令将打开整个Emacs Manual
;;
;; C-h i m elisp RET
;;
;; Emacs Lisp 教程
;; https://www.gnu.org/software/emacs/manual/html_node/eintr/index.html

;; Thanks to these people for their feedback and suggestions:
;; - Wes Hardaker
;; - notbob
;; - Kevin Montuori
;; - Arne Babenhauserheide
;; - Alan Schmitt
;; - LinXitoW
;; - Aaron Meurer
~~~


有没有感觉很神奇

至少，在我完成上面的练习之后感觉到了震惊

我就是跟着一步步做，从而在很短时间里对这门语言的基础有一个清晰的了解的

结构比较一脉相承

然后为了印证我的所学，我自己写了一个小函数，用于这篇文章中输入两个由  `~~~` 来标识的代码块

~~~lisp
(local-set-key (kbd "C-c C-`") '( lambda () (interactive)
                                           (insert "~~~\n\n~~~")
                                          (evil-previous-line)
                                          ))
~~~


这段代码的作用就是，在我按下 **Ctrl-c Ctrl-`** 的时候，会自动输出一个 markdown 的代码框，然后将光标定位到要输入代码的位置


-------------------------------------------------------------------------------

# 总结 #

Lisp 真的很简洁优雅

只是括号看起来有点怪

它提供了编程的另一种思维方式

很值得花时间了解一下

* TOC
{:toc}


-------------------------------------------------------------------------------


[jobbole]: http://blog.jobbole.com/44932/
