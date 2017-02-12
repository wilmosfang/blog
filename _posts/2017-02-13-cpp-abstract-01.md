---
layout:  post
title:  C++基础(一).抽象
author:  wilmosfang
tags:   c++
categories:  c++
wc: 209  338 7079 
excerpt:  c++ 面向对象之抽象
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

## 抽象

抽象就是忽略一个主题中与当前目标无关的那些方面，以便更充分地注意与当前目标有关的方面

抽象并不打算了解全部问题，而只是选择其中的一部分，暂时不用部分细节。比如，我们要设计一个学生成绩管理系统，考察学生这个对象时，我们只关心他的班级、学号、成绩等，而不用去关心他的身高、体重这些信息

抽象包括两个方面：

*  **过程抽象**
*  **数据抽象**

**过程抽象**是指任何一个明确定义功能的操作都可被使用者看作单个的实体看待，尽管这个操作实际上可能由一系列更低级的操作来完成

**数据抽象** 定义了数据类型和施加于该类型对象上的操作，并限定了对象的值只能通过使用这些操作修改和观察

---

## 代码示例

### 要求

构建一个运算类，实现两个操作数的加减乘除

### 代码示例

**`calc.cpp`**

~~~
#include <iostream> //count,endl 的相关定义在此申明

using namespace std;  //namespace是指标识符的各种可见范围,命名空间用关键字namespace 来定义,命名空间是C++的一种机制，用来把单个标识符下的大量有逻辑联系的程序实体组合到一起，此标识符作为此组群的名字，C++标准程序库中的所有标识符都被定义于一个名为std的namespace中，(代码中这么声明是为了更简单的调用标准库函数，不用加长串的前缀，或挨个地进行声明)

class Calc //定义一个叫Calc的类，C++中的抽象是通过类的机制来实现的
{
private:
  int a,b; //定义两个私有变量，私有变量从外部不能直接引用，只能通过内部定义的方法来进行修改和查看，所以要想修改和查看只能通过定义的公有方法来进行，这就达到了封装的效果
public:
  int add();
  int sub();
  int mul();
  int div(); //定义几个公有的方法，只有公有属性的变量或方法才能被外部引用
  Calc(int x=1, int y=1); //与类名相同的成员函数叫构造函数，构造函数是特殊的成员函数，只要创建类类型的新对象，都要执行构造函数，一般而言构造函数的工作是保证每个对象的数据成员具有合适的初值，不能有返回值(和类型)，因为要被调用，所以通常作为public成员，创建对象时自动调用
  ~Calc(); //与构造函数相对应，函数名与类名相同，但是前面有一个~符号的成员函数是析构函数，析构函数在对象销毁时被系统自动调用，所以析构函数一般会用来进行清理工作，例如释放分配的内存、关闭打开的文件等，析构函数没有返回值，不需要程序员显式调用（程序员也没法显式调用），而是在销毁对象时自动执行，析构函数没有参数，不能被重载，因此一个类只能有一个析构函数，如果用户没有定义，编译器会自动生成一个默认的析构函数
  void init(int x,int y); //定义一个初始化函数
};

Calc::Calc(int x ,int y) //实现构造函数的细节
{
  a=x; //初始化a值
  b=y; //初始化b值
  cout<<"A new Calc obj is created: a="<<a<<" b="<<b<<endl; //将对象的a,b的初值打印出来
}

Calc::~Calc() //实现析构函数的细节
{
  cout<<"A Calc obj is destroyed: a="<<a<<" b="<<b<<endl; //将对象的a,b的当前值打印出来
}

void Calc::init(int x,int y) //实现初始化函数的细节
{
  a=x; 
  b=y; //给私有变量赋指定的值。对象的私有变量无法从外部直接访问，但是可以被任意的成员函数访问，通过这种间接调用的方式，只公布部分公有成员函数的方式来实现封装的效果，可以减少耦合，提升内聚，使程度更安全和健壮
}

int Calc::add() //实现加法成员函数
{
  return a+b;
}

int Calc::sub() //实现减法成员函数
{
  return a-b;
}

int Calc::mul() //实现乘法成员函数
{
  return a*b;
}

int Calc::div() //实现除法成员函数
{
  return a/b;
}


int main()
{
  Calc c; //生成一个c对象

  c.init(20,5); //将对象进行初始化
 
  cout<<c.add()<<endl; 
  cout<<c.sub()<<endl;
  cout<<c.mul()<<endl;
  cout<<c.div()<<endl; //分别调用各计算函数

  return 0; //程序返回
}

~~~


### 编译执行

~~~
emacs@ubuntu:~/c++$ alias gtx
alias gtx='g++ -Wall -g -o'
emacs@ubuntu:~/c++$ gtx calc.x calc.cpp
emacs@ubuntu:~/c++$ ./calc.x 
A new Calc obj is created: a=1 b=1
25
15
100
4
A Calc obj is destroyed: a=20 b=5
emacs@ubuntu:~/c++$
~~~

编译执行过程中没有报错，从结果来看，符合预期

---

## include 路径

c++ 和 c 的 include 文件夹路径不一样，可以通过下面方式查看 

~~~
emacs@ubuntu:~$ gcc -E -v
Using built-in specs.
Target: i486-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 4.4.3-4ubuntu5.1' --with-bugurl=file:///usr/share/doc/gcc-4.4/README.Bugs --enable-languages=c,c++,fortran,objc,obj-c++ --prefix=/usr --enable-shared --enable-multiarch --enable-linker-build-id --with-system-zlib --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.4 --program-suffix=-4.4 --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-plugin --enable-objc-gc --enable-targets=all --disable-werror --with-arch-32=i486 --with-tune=generic --enable-checking=release --build=i486-linux-gnu --host=i486-linux-gnu --target=i486-linux-gnu
Thread model: posix
gcc version 4.4.3 (Ubuntu 4.4.3-4ubuntu5.1) 
emacs@ubuntu:~$
~~~ 


从中可以获知 **`/usr/include/c++/4.4`** 即为include路径

~~~
emacs@ubuntu:~$ ll /usr/include/c++/4.4/iostream 
-rw-r--r-- 1 root root 2654 2012-03-08 18:14 /usr/include/c++/4.4/iostream
emacs@ubuntu:~$ 
~~~




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
