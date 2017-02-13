---
layout:  post
title:  C++基础(三).封装
author:  wilmosfang
tags:   c++
categories:  c++
wc:  180  319 4429 
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

隐藏数据或方法的构造与细节是封装的主要意图

这样会有更高的内聚度，也极大程度地避免了越界篡改产生的潜在风险

因为封装产生的独立性，可以一定程度上更加明晰维护的边界

这是一种经典的模块化思维，降低了大规模项目设计的复杂度，同时也提升了代码的可重用性，节约了人力成本，提升了设计效率

---

## 代码示例

### 要求

构造一个男人类，假设这个男人23岁大学毕业参加工作，年薪4万，随着年龄的增长年薪和消费不断增加，具体按照如下发展

* 年薪=4\*(1+工龄\*10%);
* 消费=1.5\*(1+工龄\*8%);
* 房价每年5%增长=30\*(1+0.05)^(n-1)

问 : 如果他打算买一套现价30万的房子，多少岁可以买的起？


### 代码示例

**`man.cpp`**

~~~
#include <iostream>  //cout,endl
#include <math.h> //pow

using namespace std; //使用std的命名空间

class Man //创建一个Man类
{
private:
  float salary; 
  float cost;
  int age;
public:
  Man() //这是一个隐式的inline函数
  {
    age=23;  //初始年龄为23岁
    salary=4; //初始工资为4万
    cost=1.5; //初始开销为1.5万
  }
  float save(int a) //累计存款函数
  {
    float sum=0;
    int i=0;
    for(i=age;i<=a;i++)sum += sup(i)-cup(i); //将收入与开销的差额进行累计加和
    return sum; 
  }
  float sup(int a){return salary*(1+(a-age)*0.1);}; //某年龄时当年的收入
  float cup(int a){return cost*(1+(a-age)*0.08);}; //某年龄时当年的开销
  int getage(){return age;}; //返回年龄
};


class House //创建一个房子类
{
private:
  float price; //房价单位为万
public:
  House(float a){price=a;}; //inline的构造函数，初始化房价
  float pup(int n){return  price*pow(1+0.05,n-1);}; //第N年的放假
};


int main()
{
  Man m;
  House h(30);
  int i=0;
  cout<<"salary"<<"\t"<<"cost"<<"\t"<<"save"<<"\t"<<"age"<<"\t"<<"savesum"<<"\t"<<"price"<<endl;
  for(i=0;i<100;i++) 
  {
    cout<<m.sup(i+m.getage())<<"\t"<<m.cup(i+m.getage())<<"\t"<<m.sup(i+m.getage())-m.cup(i+m.getage())<<"\t"<<i+m.getage()<<"\t"<<m.save(i+m.getage())<<"\t"<<h.pup(i+1)<< endl;
    if(m.save(i+m.getage()) >= h.pup(i+1)) //如果存款可以买得起房子，就将工龄和年龄打印出来，并且退出循环
    {
      cout<<"the year is:"<<i+1<<" my age is:"<<m.getage()+i<<endl;
      break;
    }
  }
  return 0;
}
~~~


### 编译执行

~~~
emacs@ubuntu:~/c++$ alias gtx
alias gtx='g++ -Wall -g -o'
emacs@ubuntu:~/c++$ gtx man.x man.cpp
emacs@ubuntu:~/c++$ ./man.x 
salary	cost	save	age	savesum	price
4	1.5	2.5	23	2.5	30
4.4	1.62	2.78	24	5.28	31.5
4.8	1.74	3.06	25	8.34	33.075
5.2	1.86	3.34	26	11.68	34.7287
5.6	1.98	3.62	27	15.3	36.4652
6	2.1	3.9	28	19.2	38.2884
6.4	2.22	4.18	29	23.38	40.2029
6.8	2.34	4.46	30	27.84	42.213
7.2	2.46	4.74	31	32.58	44.3237
7.6	2.58	5.02	32	37.6	46.5398
8	2.7	5.3	33	42.9	48.8668
8.4	2.82	5.58	34	48.48	51.3102
8.8	2.94	5.86	35	54.34	53.8757
the year is:13 my age is:35
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
