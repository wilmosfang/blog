---
layout:  post
title:  C++基础(四).继承
author:  wilmosfang
tags:   c c++
categories:  c++
wc: 216  342 7617 
excerpt:  c++ 面向对象之继承
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

## 继承

继承是一种联结类的层次模型，并且允许和鼓励类的重用，它提供了一种明确表述共性的方法

对象的一个新类可以从现有的类中派生，这个过程称为类继承。新类继承了原始类的特性，新类称为原始类的派生类（子类），而原始类称为新类的基类（父类）

派生类可以从它的基类那里继承方法和实例变量，并且类可以修改或增加新的方法使之更适合特殊的需要。这也体现了大自然中一般与特殊的关系

继承性很好的解决了软件的可重用性问题。比如说，所有的Windows应用程序都有一个窗口，它们可以看作都是从一个窗口类派生出来的。但是有的应用程序用于文字处理，有的应用程序用于绘图，这是由于派生出了不同的子类，各个子类添加了不同的特性

---

## 代码示例

### 要求

分别定义Teacher（教师）类和Cadre（干部）类，采用多重继承方式由这两个类派生出新类Teacher_Cadre（教师兼干部）类，要求：

* 1） 在两个基类中都包括姓名、年龄、性别、地址、电话数据成员
* 2） 在Teacher类中还包括数据成员title（职称），在Cadre中还包括post（职务），在Teacher_Cadre中还包括wages（工资）
* 3） 对两个基类的姓名、年龄、性别、地址、电话 数据成员用相同的名字，在引用这些数据成员时指定作用域
* 4） 在类体中声明成员函数，在类外实现
* 5） 在派生类Teacher_Cadre的成员函数show中调用Teacher类的display函数，输出姓名、年龄、性别、地址、电话、职称，然后再用cout输出职务与工资

### 代码示例

**`man.cpp`**

~~~
#include <iostream> 

using namespace std;

class People //创建一个始祖基类
{
public:
  string name;
  int age;
  char sex;
  string address;
  string cellphone; //包含共有的成员属性
  People(string pname="",int page=0,char psex='x',string paddress="paddress",string pcellphone="pcellphone"); //给出有默认值的构造函数
};

People::People(string pname,int page,char psex,string paddress,string pcellphone) //实现构造函数
{
  name=pname;
  age=page;
  sex=psex;
  address=paddress;
  cellphone=pcellphone;
}

class Teacher:public People //创建一个Teacher类，继成自People基类
{
public:
  string title; //Teacher类的额外属性
  void display(); //定义一个显示函数
  Teacher(string tname="",int tage=0,char tsex='x',string taddress="taddress",string tcellphone="tcellphone",string ttitle="ttitle"); //声明一个构造函数
};

Teacher::Teacher(string tname,int tage,char tsex,string taddress,string tcellphone,string ttitle):People(tname,tage,tsex,taddress,tcellphone) //实现构造函数，注意调用基类构造的方法
{
  title=ttitle; //因为已经使用基类构造将其它的成员变量初始化了，只用将这额外的属性进行单独初始化就够了，这即是继成的一个好处，代码复用
}

void Teacher::display() //实现Teacher类的display方法,将成员变量的详细信息打印出来
{
  cout<<"name:\t"<<name<<endl;
  cout<<"age:\t"<<age<<endl;
  cout<<"sex:\t"<<sex<<endl;
  cout<<"address:\t"<<address<<endl;
  cout<<"cellphone:\t"<<cellphone<<endl;
  cout<<"title:\t"<<title<<endl; 
}

class Cadre:public People //创建一个Cadre类，继成自People基类
{
public:
  string post; //Cadre类的额外属性
  Cadre(string cname="",int cage=0,char csex='x',string caddress="caddress",string ccellphone="ccellphone",string cpost="cpost"); //声明Cadre类的构造方法
};

Cadre::Cadre(string cname,int cage,char csex,string caddress,string ccellphone,string cpost):People(cname,cage,csex,caddress,ccellphone) //实现Cadre类的构造函数，注意调用基类构造的方法
{
  post=cpost; //因为已经使用基类构造将其它的成员变量初始化了，只用将这额外的属性进行单独初始化就够了，这即是继成的一个好处，代码复用
}


class Teacher_Cadre:public Teacher,public Cadre //创建一个Teacher_Cadre类，继成自Teacher基类和Cadre基类 
{
public:
  float wages; //Teacher_Cadre类的额外属性
  void show(); //Teacher_Cadre类的额外方法
  Teacher_Cadre(string vname="",int vage=0,char vsex='x',string vaddress="vaddress",string vcellphone="vcellphone",string vtitle="vtitle",string vpost="vpost",float vwages=0); //声明Teacher_Cadre类的构造方法
};

Teacher_Cadre::Teacher_Cadre(string vname,int vage,char vsex,string vaddress,string vcellphone,string vtitle,string vpost,float vwages):Teacher(vname,vage,vsex,vaddress,vcellphone,vtitle),Cadre(vname,vage,vsex,vaddress,vcellphone,vpost) //实现Teacher_Cadre类的构造函数，注意通过调用基类构造的方法来初始化变量
{
  wages=vwages; //因为已经使用基类构造将其它的成员变量初始化了，只用将这额外的属性进行单独初始化就够了，这即是继成的一个好处，代码复用
}

void Teacher_Cadre::show() //实现show方法
{
  display(); //调用继成自基类的方法，显示其它属性
  cout<<"post:\t"<<post<<endl; 
  cout<<"wages:\t"<<wages<<endl; //添加基类显示方法还没覆盖到的新属性
}


int main() 
{
  Teacher_Cadre t("zhang",30,'M',"asasas","121212","profe","post1",3000); //生成一个Teacher_Cadre的对象，并且使用构造方法进行变量的初始化
  
  t.show(); //调用对象的show方法来显示自己的属性
  return 0;
}
~~~


### 编译执行

~~~
emacs@ubuntu:~/c++$ alias gtx
alias gtx='g++ -Wall -g -o'
emacs@ubuntu:~/c++$ gtx teacher.x teacher.cpp
emacs@ubuntu:~/c++$ ./teacher.x 
name:	zhang
age:	30
sex:	M
address:	asasas
cellphone:	121212
title:	profe
post:	post1
wages:	3000
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
* 抽象
* 封装
* 继成
* 多态

特别是构造函数与析构函数的调用时间需要十分清楚

析构函数根据变量的生命周期，作用域，堆内申请和栈内申请的不同，触发的时机也不尽相同，需要对内存回收的时间有一定的认识才能准确判断

派生类调用基类进行初始化时，其实现顺序应该和声明顺序一致，否则会出编译错误，如：

~~~
class Teacher_Cadre:public Teacher,public Cadre
...
...
Teacher_Cadre::Teacher_Cadre(string vname,int vage,char vsex,string vaddress,string vcellphone,string vtitle,string vpost,float vwages):Teacher(vname,vage,vsex,vaddress,vcellphone,vtitle),Cadre(vname,vage,vsex,vaddress,vcellphone,vpost) 
~~~

[programming]:http://soft.dog/2016/04/07/thinking-of-programming/

