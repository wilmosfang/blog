---
layout:  post
title:  C++基础(二).封装
author:  wilmosfang
tags: c  c++
categories:  c++
wc: 181  275 4545 
excerpt:  c++ 面向对象之封装
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

## 封装

封装是面向对象的特征之一，是对象和类概念的主要特性

封装是把过程和数据包围起来，对数据的访问只能通过已定义的接口

面向对象计算始于这个基本概念，即现实世界可以被描绘成一系列完全自治、封装的对象，这些对象通过一个受保护的接口访问其他对象

一旦定义了一个对象的特性，则有必要决定这些特性的可见性，即哪些特性对外部世界是可见的，哪些特性用于表示内部状态。在这个阶段定义对象的接口。通常，应禁止直接访问一个对象的实际表示，而应通过操作接口访问对象，这称为信息隐藏

事实上，信息隐藏是用户对封装性的认识，封装则为信息隐藏提供支持。封装保证了模块具有较好的独立性，使得程序维护修改较为容易。对应用程序的修改仅限于类的内部，因而可以将应用程序修改带来的影响减少到最低限度

---

## 代码示例

### 要求

学生成绩有4门，语文、数学、英语、政治，构造一个类，实现

* 1）求平均成绩
* 2）求最高分数
* 3）求不及格的科目

### 代码示例

**`score.cpp`**

~~~
#include <iostream> //cout,endl 相关声明在这里
#include <string.h> //strcpy 相关声明在这里

#define MAX 20
#define NUM 4

using namespace std;  //设定名称空间为 std

class Course  //定义一个课程类
{
public:
  char name[MAX]; //课程名称
  float score;  //课程分数
  void init(char* s="x",float sc=0);  //初始化函数
};

void Course::init(char* s,float sc) //初始化函数的实现
{
  strcpy(name,s);
  score=sc;
}

class Score //定义一个学生分数类
{
private:
  Course  course[NUM];  //里面包含四门课
public:	
  float avg(); //求平均分
  float max(); //求最大分
  void fail(); //求不及格分
  Score(float a=0,float b=0,float c=0,float d=0); //构造函数初始化变量
};

void Score::fail() //不及格函数的实现 
{
  int i=0;
  for(i=0;i<NUM;i++) if(course[i].score < 60) cout <<course[i].name<<" is fail"<<endl;
}

float Score::max() //最大分数函数的实现 
{
  int i=0;
  float max=0;
  for(i=0;i<NUM;i++) max=((max>course[i].score)?max:course[i].score);
  return max;
}

float Score::avg() //平均分数函数的实现 
{
  int i=0;
  float sum=0;
  for(i=0;i<NUM;i++) sum+=course[i].score;
  return sum/i;
}

Score::Score(float a,float b,float c,float d) //构造函数的实现，对四门课程进行名称和分值的初始化
{
  course[0].init((char*)"yuwen",a);
  course[1].init((char*)"math",b);
  course[2].init((char*)"english",c);
  course[3].init((char*)"politics",d);
}

int main() 
{
  Score s(23,30,90,100); //初始化
  cout<<"the avg score is: "<<s.avg()<<endl; 
  cout<<"the max score is: "<<s.max()<<endl;
  s.fail(); //分别打印出平均分，最高分和不及格课程
  return 0;
}
~~~


### 编译执行

~~~
emacs@ubuntu:~/c++$ alias  gtx
alias gtx='g++ -Wall -g -o'
emacs@ubuntu:~/c++$ gtx  score.x score.cpp
emacs@ubuntu:~/c++$ ./score.x 
the avg score is: 60.75
the max score is: 100
yuwen is fail
math is fail
emacs@ubuntu:~/c++$
~~~

编译执行过程中没有报错，从结果来看，符合预期


---

# 总结

弄清下面概念对掌握c++很有帮助

* 名称空间
* 类
* 私有属性
* 公有属性
* 保护属性
* 成员变量
* 成员函数
* 构造函数
* 析构函数

特别是构造函数与析构函数的调用时间需要十分清楚

析构函数根据变量的生命周期，作用域，堆内申请和栈内申请的不同，触发的时机也不尽相同，需要对内存回收的时间有一定的认识才能准确判断

[programming]:http://soft.dog/2016/04/07/thinking-of-programming/
