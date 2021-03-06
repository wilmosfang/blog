---
layout:  post
title:  C++基础(五).多态
author:  wilmosfang
tags:   c c++
categories:  c++
wc: 240 313 7587
excerpt:  c++ 面向对象之多态，重载、隐藏、重写、重载和重写的区别、隐藏和重写重载的区别、友元、友元函数、友元类
comments: true
---


# 前言

C++语言是C语言的拓展，C语言是面向过程的，C++在C的基础上增加了面向对象的方法

什么是面向对象呢，面向对象就是将数据和对数据的加工方法打包在一起，进行模块化的调用，通过方法来进行数据交换的一种设计方法

> **Tip:** 本人关于程序的认知，可以参看前面写的 **[一个运维人员的编程思维][programming]**

面向对象的程序设计有四个主要特点：

* 抽象
* 封装
* 继承
* 多态

下面就通过C++来对面向对象的核心特性进行分享

---



# 概要

* TOC
{:toc}

---

## 多态

多态性是指允许不同类的对象对同一消息作出响应

比如同样的加法，把两个时间加在一起和把两个整数加在一起肯定完全不同;又比如，同样的选择编辑-粘贴操作，在字处理程序和绘图程序中有不同的效果

多态性包括参数化多态性和包含多态性

多态性语言具有灵活、抽象、行为共享、代码共享的优势，很好的解决了应用程序函数同名问题

---

## 相关概念

### 重载

同一可访问区内被声明的几个具有不同参数列（参数的类型，个数，顺序不同）的同名函数，根据参数列表确定调用哪个函数，重载不关心函数返回类型

### 隐藏

派生类的函数屏蔽了与其同名的基类函数，注意只要同名函数，不管参数列表是否相同，基类函数都会被隐藏

### 重写

重写也叫覆盖，是指派生类中存在重新定义的函数。其函数名，参数列表，返回值类型，所有都必须同基类中被重写的函数一致。只有函数体不同（花括号内），派生类调用时会调用派生类的重写函数，不会调用被重写函数。重写的基类中被重写的函数必须有virtual修饰

### 重载和重写的区别

* 范围区别：重写和被重写的函数在不同的类中，重载和被重载的函数在同一类中
* 参数区别：重写与被重写的函数参数列表一定相同，重载和被重载的函数参数列表一定不同
* virtual的区别：重写的基类必须要有virtual修饰，重载函数和被重载函数可以被virtual修饰，也可以没有

### 隐藏和重写，重载的区别

* 与重载范围不同：隐藏函数和被隐藏函数在不同类中
* 参数的区别：隐藏函数和被隐藏函数参数列表可以相同，也可以不同，但函数名一定同；当参数不同时，无论基类中的函数是否被virtual修饰，基类函数都是被隐藏，而不是被重写


> **Tip:** 引自 **[C++中重载、重写（覆盖）和隐藏的区别][48976097]**

---

## 友元

我们已知道类具备封装和信息隐藏的特性。只有类的成员函数才能访问类的私有成员，程式中的其他函数是无法访问私有成员的。非成员函数能够访问类中的公有成员，但是假如将数据成员都定义为公有的，这又破坏了隐藏的特性。另外，应该看到在某些情况下，特别是在对某些成员函数多次调用时，由于参数传递，类型检查和安全性检查等都需要时间开销，而影响程式的运行效率

为了解决上述问题，提出一种使用友元的方案。友元是一种定义在类外部的普通函数，但他需要在类体内进行说明，为了和该类的成员函数加以区别，在说明时前面加以关键字friend。友元不是成员函数，但是他能够访问类中的私有成员。友元的作用在于提高程式的运行效率，但是，他破坏了类的封装性和隐藏性，使得非成员函数能够访问类的私有成员

友元能够是个函数，该函数被称为友元函数；友元也能够是个类，该类被称为友元类

### 友元函数

友元函数的特点是能够访问类中的私有成员的非成员函数。友元函数从语法上看，他和普通函数相同，即在定义上和调用上和普通函数相同

### 友元类

友元除了前面讲过的函数以外，友元还能够是类，即一个类能够作另一个类的友元。当一个类作为另一个类的友元时，这就意味着这个类的任何成员函数都是另一个类的友元函数


> **Tip:** 引自 **[C++中友元详解][2037327]**


---

## 代码示例

### 要求

用友元函数实现clock类的前置、后置单目运算符重载

~~~
  Clock c；
   (c++).showTime();
   (++c).showTime();
~~~

### 代码示例

**`clock.cpp`**

~~~
#include <iostream> //cout,endl 相关函数的声明在此
#include <stdlib.h> //exit 相关函数的声明在此
#include <stdio.h> //printf 相关函数的声明在此

using namespace std; //使用std命名空间

class Clock //定义一个时钟类
{
public:
  friend Clock operator ++ (Clock &c); //使用友元函数的方式对前置++运算符进行重载
  friend Clock operator ++ (Clock &c,int); //使用友元函数的方式对后置++运算符进行重载，注意这里使用别名的方式来引用被操作的对象，为了区别于前置重载，这里留出一个int类型的参数空位，但并不需要真实传参，(此项仅为C++语言中为示区别的约定，即便是其它类型的单目运算符也用int来占位)
  void showTime(); //声明一个显示方法
  Clock(int h=0,int m=0,int s=0); //Clock的构造函数，给出了默认值
private:
  int hour,minute,second; //成员变量
};

Clock::Clock(int h,int m,int s) //构造函数的实现 
{
  if( h<0 || h >23 ) //对hour参数进行检查
  {
    cout<<"error init Hour:"<<h<<endl;
    exit(1);
  }
  if( m<0 || m>59 ) //对minute参数进行检查
  {
    cout<<"error init Minute:"<<m<<endl;
    exit(1);
  }
  if( s<0 || s>59 ) //对second参数进行检查
  {
    cout<<"error init Second:"<<s<<endl;
    exit(1);
  }

  hour=h;
  minute=m;
  second=s; //进行相应的初始化
}


void Clock::showTime() //showTime的实现 
{
  printf("%02d:%02d:%02d\n",hour,minute,second);
}

Clock operator ++ (Clock &c) //前置++运算的实现 
{
  c.second++;
  if(c.second>=60) //对second溢出进位的判断和处理
  {
    c.second-=60;
    c.minute++;
    if(c.minute>=60) //对minute溢出进位的判断和处理
    {
      c.minute-=60;
      c.hour++;
      c.hour%=24; //对hour溢出进位的判断和处理
    }
  }
  return c; 
}

Clock operator ++ (Clock &c,int) //后置++运算的实现
{
  Clock old=c;
  ++c; //先调用前置++
  return old; //返回++操作之前的状态
}


int main()
{
  Clock c(23,59,59); 

  c.showTime();
  (++c).showTime(); //进行前置++测试
  c++.showTime(); //进行后置++测试
  c.showTime();

  return 0;
}
~~~

### 编译执行

~~~
emacs@ubuntu:~/c++$ alias gtx
alias gtx='g++ -Wall -g -o'
emacs@ubuntu:~/c++$ gtx clock.x clock.cpp
emacs@ubuntu:~/c++$ ./clock.x 
23:59:59
00:00:00
00:00:00
00:00:01
emacs@ubuntu:~/c++$
~~~

编译执行过程中没有报错，从结果来看，符合预期





---

# 总结

弄清下面概念对掌握c++很有帮助

* 友元(函数/类)
* 单目运算符(前置/后置)
* 重载
* 重写
* 隐藏

[programming]:/2016/04/07/thinking-of-programming/
[48976097]:http://blog.csdn.net/zx3517288/article/details/48976097
[2037327]:http://blog.chinaunix.net/uid-790245-id-2037327.html
