---
layout:  post
title:  数据结构基础（一）
author:  wilmosfang
tags:   c 
categories:  c
wc:  282  610 5935 
excerpt:  数据结构，顺序表
comments: true
---


# 前言

数据结构由 **数据的值** 与 **数据之间的关系** 所构成

**数据结构** 本质就是 **数据的表示** ， **算法** 本质就是 **数据处理** 

数据的 **表示** 和 **处理** 构成了计算系统的所有内涵

> **Note:**  小生关于程序的其它看法，在 **[《一个运维人员的编程思维》][thinking-of-programming]** 中有所详述，欢迎前来拍砖

* 常见的数据结构可以进行如下划分：

|      |  |    | ||
| :------- | :---- | :--- |:---|
| 逻辑结构 | 线性结构 |  线性表    |顺序表、单向链表、双向链表|
| |  |  栈    ||
| |  |  队   ||
| | 非线性结构 |  集合    ||
| |  |  树形结构    ||
| |  |  图形结构    ||
| 存储结构    | 顺序存储   |     ||
| | 链式存储   |     ||


* 对于数据的常见运算包含如下一些内容：

对于元素的操作：增、删、改、查

对于集合的操作：建、排、销


这里分享一下我在学习线性表过程中的一些笔记


---

# 概要

* TOC
{:toc}

---

## 线性表

线性表的特征：对非空表，a(0)是表头，无前驱；a(n-1)是表尾，无后继；其它的每个元素a(i)有且仅有一个直接前驱a(i-1)和一个直接后继a(i+1)

线性表作为一种基本的数据结构类型，在计算机存储器中的表示一般有两种形式，一种是顺序映象，一种是链式映象

---

## 顺序表

在一片连续的存储空间中，逻辑上相邻的元素存储位置也相邻的线性表，就是顺序表

正由于顺序表的物理特性，致其增删慢而查改快，同时存储密度高

---

## 代码示例


~~~
emacs@ubuntu:~/c$ cat 5.c
#include <stdio.h>
#include <malloc.h>
#define LEN 100

typedef struct line
{
  int date[LEN];
  int last;
}LIST,*LP;

int ifEmptyList(const LP head)
{
  if(-1 == head->last) return 0;
  else return -1;
}

int showList(const LP head)
{
  int i=0;
  for(i=0;i < head->last+1;i++)printf("[%d]",head->date[i]);
  printf("\n");
  return 0;
}

int instNode(const LP head,int pos,int value)
{
  int i=0;
  if(head->last >= LEN-1) 
  {
    printf("the list is full\n");
    return -1;
  }
  if(pos < 0) pos=0;
  if(pos > head->last+1) pos= head->last+1;
  for(i=head->last;i>pos;i--) head->date[i+1]=head->date[i];
  head->date[pos]=value;
  head->last++;
  return 0;
}

int delNode(const LP head,int pos)
{
  int i=0;
  if(0 == ifEmptyList(head))return -1;
  if(0 > pos)pos=0;
  if(pos > head->last) pos=head->last;
  for(i=pos;i < head->last ;i++)head->date[i]=head->date[i+1];
  head->last--;
  return 0;
}

int sortListAsc(const LP head)
{
  int i=0,j=0,tmp=0;
  if(0 == ifEmptyList(head))return -1;
  for(i=0;i < head->last;i++)
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

int sortListDesc(const LP head)
{
  int i=0,j=0,tmp=0;
  if(0 == ifEmptyList(head))return -1;
  for(i=0;i < head->last;i++)
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

int modifyNode(const LP head,int pos,int value)
{
  if(0 == ifEmptyList(head))return -1;
  if(pos > head->last || pos < 0)
  {
    printf("the pos is out of range [0,%d],and pos is %d\n",head->last,pos);
    return -1;
  }
  head->date[pos]=value;
  return 0;
}

int getNode(const LP head,int pos)
{
  if(0 == ifEmptyList(head))return -1;
  if(pos > head->last || pos < 0)
  {
    printf("the pos is out of range [0,%d],and pos is %d\n",head->last,pos);
    return -1;
  }
  printf("the value of Node on pos %d  is [%d]\n",pos,head->date[pos]);
  return 0;
}

int clearList(const LP head)
{
  head->last=-1;
  return 0;
}

int main()
{
  //creat a list
  LIST list;
  LP l=&list;
  l->last = -1;
  //show list
  printf("show this list\n");
  showList(l);
  //add a node to list
  printf("add 9 nodes to list\n");
  instNode(l,100,3);
  instNode(l,100,2);
  instNode(l,100,7);
  instNode(l,100,9);
  instNode(l,100,19);
  instNode(l,100,88);
  instNode(l,100,99);
  instNode(l,100,11);
  instNode(l,100,44);
  showList(l);
  //del a node from list
  printf("delete 3 nodes(-7,4,19) from list\n");
  delNode(l,-7);
  delNode(l,4);
  delNode(l,19);
  showList(l);
  //sort list order by desc
  printf("sort list order by desc \n");
  sortListDesc(l);
  showList(l);
  printf("sort list order by asc\n");
  //sort list order by asc
  sortListAsc(l);
  showList(l);
  //modify a node value
  printf("modify node on 3 postion(3,-9,40)\n");
  modifyNode(l,3,99);
  modifyNode(l,-9,99);
  modifyNode(l,40,99);
  showList(l);
  //get a node value
  printf("get node on 3 postion(-1,2,90)\n");
  getNode(l,-1);
  getNode(l,2);
  getNode(l,90);
  showList(l);
  //clear list
  printf("clear list\n");
  clearList(l);
  showList(l);
  return 0;
}
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc 5.x 5.c
emacs@ubuntu:~/c$ 
emacs@ubuntu:~/c$ ./5.x
show this list

add 9 nodes to list
[3][2][7][9][19][88][99][11][44]
delete 3 nodes(-7,4,19) from list
[2][7][9][19][99][11]
sort list order by desc 
[99][19][11][9][7][2]
sort list order by asc
[2][7][9][11][19][99]
modify node on 3 postion(3,-9,40)
the pos is out of range [0,5],and pos is -9
the pos is out of range [0,5],and pos is 40
[2][7][9][99][19][99]
get node on 3 postion(-1,2,90)
the pos is out of range [0,5],and pos is -1
the value of Node on pos 2  is [9]
the pos is out of range [0,5],and pos is 90
[2][7][9][99][19][99]
clear list

emacs@ubuntu:~/c$ 
~~~




[thinking-of-programming]:http://soft.dog/2016/04/07/thinking-of-programming/#section-11

