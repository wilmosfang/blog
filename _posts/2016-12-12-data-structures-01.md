---
layout:  post
title:  数据结构基础(一).顺序表
author:  wilmosfang
tags:   c 
categories:  c
wc: 297 640 8746
excerpt:  数据结构，线性表，顺序表
comments: true
---


# 前言

**数据** 是能被计算机识别、存储和处理的 **信息载体**

数据结构由 **数据的值** 与 **数据之间的关系** (也是数值的一种) 所构成

**数据结构** 本质就是 **数据表示** (数据的结构)， **算法** 本质就是 **数据处理** (数据的加工方法)

数据的 **表示** 和 **处理** 构成了计算系统的所有内涵

> **Note:**  小生关于程序的其它看法，在 **[《一个运维人员的编程思维》][thinking-of-programming]** 中有所详述，欢迎前来拍砖


**数据的逻辑结构**：

* 集合
* 线性
* 树形
* 图状

> **Tip:** 其中的线性结构中又包含了 **表、栈、队**

**数据的存储结构**：

* 顺序
* 链式
* 索引
* 散列


**数据结构的基础运算**：

对于元素(个体)的操作：增加、删除、修改、查询

对于集合(群体)的操作：创建、排序、注销


> **Tip:**  数据结构主要在研究非数值性程序设计中计算机操作的对象及其相互间的关系运算，所以其代表的外延远不止上面所说的几种，上面所述只是最最常见的一些形式，其它的形式可以由它们组合而成

有一个网站 **[VisuAlgo][visualgo]** 能将数据结构进行可视化展示

这里分享一下我在学习线性表过程中的一些笔记

---

# 概要

* TOC
{:toc}

---

## 线性表

线性表的特征：对非空表，a(0)是表头，无前驱；a(n-1)是表尾，无后继；其它的每个元素a(i)有且仅有一个直接前驱a(i-1)和一个直接后继a(i+1)

线性表在计算机存储器中的表示一般有两种形式，一种是顺序映象，一种是链式映象


---

## 顺序表

在一片连续的存储空间中，逻辑上相邻的元素存储位置也相邻的线性表，就是顺序表

正由于顺序表的物理特性，致其 **增删慢** 而 **查改快** ，同时 **存储密度高**

> **Tips:** 之所以存储密度高，是因为这种形式只用存值，逻辑关系不需要消耗额外空间，关系信息是通过存储空间的先后关系推倒出来的

---

## 代码示例


~~~
#include <stdio.h>  
#define LEN 100 //定义此表的最大长度

typedef struct line 
{
  int date[LEN]; //用来存放数据
  int last; //用来存放最后下标
}LIST,*LP; //为顺序表结构体重命名为LIST，为指针重命名为LP

int ifEmptyList(const LP head)  //用来判断此表是否为空
{
  if(-1 == head->last) return 0; //如果下标为-1就代表为空返回0，否则返回-1
  else return -1;
}

int showList(const LP head) //将列表中的所有元素进行打印
{
  int i=0;
  for(i=0;i < head->last+1;i++)printf("[%d]",head->date[i]);  //这里边际条件如果不是 head->last+1 就会忽略掉最后一个元素
  printf("\n");
  return 0;
}

int instNode(const LP head,int pos,int value)  //在列表中指定的位置插入一个值
{
  int i=0;
  if(head->last >= LEN-1) //通过最大下标判断此列表有没有满，如果满了，就警告退出
  {
    printf("the list is full\n");
    return -1;
  }
  if(pos < 0) pos=0; //对插入位置进行校正，位置为负时，定位到0位置
  if(pos > head->last+1) pos= head->last+1; //对插入位置进行校正，位置超出最后一个元素时，定位到末尾位置
  for(i=head->last;i>=pos;i--) head->date[i+1]=head->date[i]; //对数据进行迁移，这也是顺序表插入元素过程中，最耗时的步骤
  head->date[pos]=value;  //将值进行插入
  head->last++; //及时跟进最大下标
  return 0;
}

int delNode(const LP head,int pos) //在列表中指定的位置删除一个值
{
  int i=0;
  if(0 == ifEmptyList(head))return -1; //删除前进行一下检查，判断此表是否为空
  if(0 > pos)pos=0; //对删除位置进行校正，位置为负时，定位到0位置
  if(pos > head->last) pos=head->last; //对删除位置进行校正，位置超出最后一个元素时，定位到最后一个元素的位置
  for(i=pos;i < head->last ;i++)head->date[i]=head->date[i+1]; //对数据进行迁移，这也是顺序表删除元素过程中，最耗时的步骤
  head->last--; //及时跟进最大下标
  return 0;
}

int sortListAsc(const LP head) //对顺序表进行升序排序
{
  int i=0,j=0,tmp=0; 
  if(0 == ifEmptyList(head))return -1; //排序前进行一下检查，判断此表是否为空
  for(i=0;i < head->last;i++) //冒泡排序的思想进行排序
  {
    for(j=i+1;j < head->last+1;j++)
    {
      if(head->date[i] > head->date[j]) 
      {
	    tmp=head->date[i];
	    head->date[i]=head->date[j];
	    head->date[j]=tmp;
      }
    }
  }
  return 0;
}

int sortListDesc(const LP head) //对顺序表进行降序排序
{
  int i=0,j=0,tmp=0;
  if(0 == ifEmptyList(head))return -1; //排序前进行一下检查，判断此表是否为空
  for(i=0;i < head->last;i++) //冒泡排序的思想进行排序
  {
    for(j=i+1;j < head->last+1;j++)
    {
      if(head->date[i] <  head->date[j])
      {
	    tmp=head->date[i];
	    head->date[i]=head->date[j];
	    head->date[j]=tmp;
      }
    }
  }
  return 0;
}

int modifyNode(const LP head,int pos,int value) //表中指定位置的元素修改为新值
{
  if(0 == ifEmptyList(head))return -1; //修改前进行一下检查，判断此表是否为空
  if(pos > head->last || pos < 0) //对指定位置进行有效性检查，如果超出范围，进行提醒
  {
    printf("the pos is out of range [0,%d],and pos is %d\n",head->last,pos);
    return -1;
  }
  head->date[pos]=value; //指定位置的元素修改为新值
  return 0;
}

int getNode(const LP head,int pos) //获取表中指定位置元素的值
{
  if(0 == ifEmptyList(head))return -1; //查询前进行一下检查，判断此表是否为空
  if(pos > head->last || pos < 0) //对指定位置进行有效性检查，如果超出范围，进行提醒
  {
    printf("the pos is out of range [0,%d],and pos is %d\n",head->last,pos);
    return -1;
  }
  printf("the value of Node on pos %d  is [%d]\n",pos,head->date[pos]); //输出指定位置元素的值
  return 0;
}

int clearList(const LP head) //清空表
{
  head->last=-1; //因为最大下标对表里的元素有合法性的约束力，所以直接置-1就轻易的进行了清表操作，其实数据并没删除，但是已经不被认为合法
  return 0;
}

int main()
{
  //creat a list
  LIST list; //创建一个表
  LP l=&list; 
  l->last = -1; //将表的最大下标置-1，代表此表为空
  //show list
  printf("show this list\n"); 
  showList(l); //打印测试
  //add nodes to list
  printf("add 9 nodes to list\n"); 
  instNode(l,-100,3);
  instNode(l,0,2);
  instNode(l,100,7);
  instNode(l,100,9);
  instNode(l,2,19);
  instNode(l,5,88);
  instNode(l,100,99);
  instNode(l,100,11);
  instNode(l,100,44); //添加元素测试
  showList(l);
  //del a node from list
  printf("delete 5 nodes(-7,0,4,5,19) from list\n");
  delNode(l,-7);
  delNode(l,0);
  delNode(l,4);
  delNode(l,5);
  delNode(l,19); //分别取五个有代表性的位置进行删除测试
  showList(l);
  //sort list order by desc
  printf("sort list order by desc \n"); 
  sortListDesc(l); //降序排序测试
  showList(l);
  printf("sort list order by asc\n");
  //sort list order by asc
  sortListAsc(l); //升序排序测试
  showList(l);
  //modify a node value
  printf("modify node on 3 postion(-9,3,40)\n");
  modifyNode(l,-9,99);
  modifyNode(l,3,99);
  modifyNode(l,40,99); //修改值测试
  showList(l);
  //get a node value
  printf("get node on 3 postion(-1,2,90)\n");
  getNode(l,-1);
  getNode(l,2);
  getNode(l,90); //查询测试
  showList(l);
  //clear list
  printf("clear list\n");
  clearList(l); //清表测试
  showList(l);
  return 0;
}
~~~

编译执行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc toblog.x toblog.c
emacs@ubuntu:~/c$ ./toblog.x 
show this list

add 9 nodes to list
[2][3][19][7][9][88][99][11][44]
delete 5 nodes(-7,0,4,5,19) from list
[19][7][9][88]
sort list order by desc 
[88][19][9][7]
sort list order by asc
[7][9][19][88]
modify node on 3 postion(-9,3,40)
the pos is out of range [0,3],and pos is -9
the pos is out of range [0,3],and pos is 40
[7][9][19][99]
get node on 3 postion(-1,2,90)
the pos is out of range [0,3],and pos is -1
the value of Node on pos 2  is [19]
the pos is out of range [0,3],and pos is 90
[7][9][19][99]
clear list

emacs@ubuntu:~/c$ 
~~~



[thinking-of-programming]:/2016/04/07/thinking-of-programming/#section-11
[visualgo]:https://visualgo.net/
