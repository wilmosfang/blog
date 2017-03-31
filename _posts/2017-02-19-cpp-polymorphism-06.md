---
layout:  post
title:  C++基础(六).多态
author:  wilmosfang
tags:   c c++
categories:  c++
wc: 234 374 5471
excerpt:  c++ 面向对象之多态，成员函数运算符重载
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

前面的一篇中使用友元函数的方式实现clock类的前置、后置单目运算符重载，使用成员函数的方式也可以实现重载，这里稍加介绍

---



# 概要

* TOC
{:toc}

---


## 代码示例

### 要求

运算符重载实现 (成员函数形式)  字符串类STR 加法

~~~
int main()
{
STR b("helloworld");
STR b1("world");
STR c,d;
c=b+b1;
c.display(); //打印出helloworldworld

         
STR a(b);
a.display(); //打印出helloworld

return 0;
}
~~~

可以使用`strcat、strcpy、strlen`库函数


### 代码示例

**`str.cpp`**

~~~
#include <iostream> //cout,endl 相关函数的声明
#include <string.h> //strlen,strcpy,strcat 相关函数的声明

using namespace std; //使用std命名空间

class STR
{
public:

  STR(const STR &str); //使用STR类进行构造
  STR(const char * const s = NULL); //使用字符串进行构造，这里得使用 const 进行限定，如果不限定，就有对指向内容进行修改的嫌疑，这样编译器会报警告
  ~STR(); //析构函数，清理现场

  STR operator + (const STR &str); //使用成员函数的形式重载加法运算
  STR operator = (const STR &str); //使用成员函数的形式重载赋值运算

  void display(); //显示函数

protected:
  char * pstr; //字符指针，指向数据空间
  int  slen; //字符串的长度
};


STR::~STR() //析构函数
{
  delete[] pstr; //回收内存
}

STR::STR(const STR &str) //构造函数的实现,使用STR类对象进行构造
{
  slen=strlen(str.pstr); //设定长度
  pstr=new char[slen+1]; //分配内存
  strcpy(pstr,str.pstr); //复制内容
}

STR::STR(const char * const s)  //构造函数的实现,使用字符串进行构造
{
  if(NULL == s)  //若指针为空则构造一个空对象
  {
    pstr=new char[1];
    *pstr='\0';
    slen=0;
    return;
  }

  slen=strlen(s); //不为空则计算长度，并且设定
  pstr=new char[slen+1]; //分配内存
  strcpy(pstr,s); //复制内容
}

void STR::display() //显示内容
{
  cout<<pstr<<endl; //将字符串内容进行显示
}


STR STR::operator + (const STR &str) //对此类的加法运算符进行重载
{
  STR bstr; //构建一个空对象

  bstr.slen=slen+str.slen; //设定长度
  delete[] bstr.pstr; //回收内存,这一步非常必要，否则会逐渐泄露内存，一次一个字节
  bstr.pstr=NULL; //指空，避免野指针
  bstr.pstr=new char[bstr.slen+1]; //根据长度重配内存
  strcpy(bstr.pstr,pstr); //复制主体内容
  strcat(bstr.pstr,str.pstr); //复制被加对象内容

  return bstr; //将新构造的对象进行返回
}



STR STR::operator = (const STR &str) //对此类的赋值运算符进行重载
{
  if(this == &str) return *this; //如果被赋的对象就是自己，什么也不用做，直接返回
  
  delete [] pstr; //否则，回收掉旧有内存
  pstr=NULL; //指空，避免野指针
  slen=strlen(str.pstr); //计算长度，并且赋给自己
  pstr = new char[slen+1]; //根据长度重配内存
  strcpy(pstr,str.pstr); //复制内容
  
  return *this;
}


int main()
{
  STR b("helloworld"); //使用字符串初始化
  STR b1("world"); 
  STR c; //使用空指针初始化

  c=b+b1;  //进行加法运算，和赋值操作
  c.display(); 
  
  STR a(b); //使用对象进行初始化
  a.display(); 

  return 0;
}
~~~

### 编译执行

~~~
emacs@ubuntu:~/c++$ alias gtx
alias gtx='g++ -Wall -g -o'
emacs@ubuntu:~/c++$ gtx str.x  str.cpp
emacs@ubuntu:~/c++$ ./str.x 
helloworldworld
helloworld
emacs@ubuntu:~/c++$
~~~

编译执行过程中没有报错，从结果来看，符合预期


---

## const 限定

使用字符串进行构造的过程中要对字符串的内容使用 `const` 进行限定，如下

~~~
STR(const char * const s = NULL); 
~~~

如果不进行限定，如下

~~~
STR(char * const s = NULL); 
~~~

如果不限定，就有对指向内容进行修改的嫌疑，而字符串常量是处于静态区，并且内容也是固定不变的，这样编译器会报警告

~~~
emacs@ubuntu:~/c++$ gtx str.x  str.cpp
str.cpp: In function ‘int main()’:
str.cpp:90: warning: deprecated conversion from string constant to ‘char*’
str.cpp:91: warning: deprecated conversion from string constant to ‘char*’
emacs@ubuntu:~/c++$ 
~~~

虽然警告内容不是我上面说的意思，但是加上 `const` 后，这个警告就会消除


> **Tip:** 关于`const`的详细用法可以参看  **[C语言深度解剖 (二) 之 const][const]**

---

# 总结

弄清下面概念对掌握c++很有帮助

* 成员函数
* 运算符重载

[programming]:/2016/04/07/thinking-of-programming/
[const]:/2016/11/30/c-deep-02/#const
