---
layout:  post
title:  数据结构基础(二).单链表
author:  wilmosfang
tags:   c 
categories:  c
wc: 370 740 10814
excerpt:  线性表，链表，单向链表
comments: true
---


# 前言

线性表是一种应用广泛和最为基础的数据结构

线性表的特征：对非空表，a(0)是表头，无前驱；a(n-1)是表尾，无后继；其它的每个元素a(i)有且仅有一个直接前驱a(i-1)和一个直接后继a(i+1)

线性表在计算机存储器中的表示一般有两种形式，一种是顺序映象，一种是链式映象

有一个网站 **[VisuAlgo][visualgo]** 能将数据结构进行可视化展示

这里分享一下我在学习线性表过程中的一些笔记，前面一篇用C语言实现了一个简单的顺序表，这里用C语言实现一个简单的单向链表

---

# 概要

* TOC
{:toc}

---



## 链表结构

将线性表中各元素分布在存储器的不同存储块中，通过地址或指针建立它们之间的联系，所得到的的存储结构为链表结构

链表结构根据指向的特性，分为 **单向链表** 和 **双向链表** 

> **Tips:** 单双循环链表是它们的变种

线性表的顺序存储结构有存储密度高和能随机存取的优点，但有以下不足：

* 插入删除操作比较耗时，因为相应的后续元素要在存储器中成片移动
* 要求系统提供较大的连续存储空间

线性表的链式存储结构可以有效克服以上不足，但代价就是存储密度低，也无法随机存取

> **Tips:** 线性表的链式存储结构和顺序存储结构优劣是互置的，之所以存储密度低，是因为这种形式的节点中不仅要存值，逻辑关系也需要消耗额外空间，节点关系是通过在数据节点中存储下一节点的位置信息来实现的，但这种开销换来了足够的灵活度和增删效率


---

## 代码示例


~~~
#include <stdio.h>
#include <malloc.h>

typedef struct student 
{
  int ID;
  int score;
  struct student *next; //存放下一节点的位置
}STU,*STUP; //将定义的结构体重命名为STU类型，此类指针重命名为STUP


STUP createList() //创建空表
{
  STUP head=NULL;
  head=(STUP)malloc(sizeof(STU)); //申请内存
  if(NULL == head) //跟进检查，如果申请失败则提醒返回，将NULL放在左边是一种更安全的做法
  {
    printf("error:no enough memory!\n"); 
    return NULL;
  }
  head->ID=0; //初始化，虽然头节点的这个值无用，但是给变量赋初值是一种更安全的实践
  head->score=0; //设定初值为0，头节点的这个值还有另外的意思，用来记录链表中的元素个数
  head->next=NULL; //由于是空表，将下一节点位置置空
  return head; //返回此头节点
}

int instNode(STUP const head,int id,int score,int pos) //在列表中的指定位置插入给定ID和socre的记录
{
  STUP p=NULL,r=head;
  int i=0;
  if(pos < 1) pos=1; //对插入位置进行校正，位置小于1时，定位到1位置
  if(pos > head->score + 1) pos=head->score + 1; //对插入位置进行校正，位置超出最后一个元素时，定位到末尾位置
  p=(STUP)malloc(sizeof(STU)); //申请内存，创建一个节点
  if(NULL == p) //跟进检查，如果申请失败则提醒返回，将NULL放在左边是一种更安全的做法
  {
    printf("error:no enough memory!\n");
    return -1;
  }
  p->ID=id; //初始化id为给定值
  p->score=score; //初始化score为给定值
  for(i=0;i<pos-1;i++) r=r->next; //定位到插入点前一个元素的位置
  p->next=r->next; //挂上新节点
  r->next=p;  //接入新节点,及插入新节点
  head->score++; //及时跟进最大下标
  return 0;
 } 

int ifEmptyList(const STUP head) //用来判断此表是否为空，如果为空则返回-1
{
  if(0 == head->score) //判断方法为检查元素个数是否为0
  {
    printf("warning:empty list!\n");
    return -1;
  }
  return 0;
} 

int delNode(STUP const head,int pos) //在列表中指定的位置删除一个节点
{
  STUP r=head,p=NULL;
  int i=0;
  if(0 != ifEmptyList(head) )return -1; //删除前进行一下检查，判断此表是否为空
  if(1 > pos) pos=1; //对删除位置进行校正，位置小于1时，定位到1位置
  if(pos > r->score) pos=r->score; //对删除位置进行校正，位置超出最后一个元素时，定位到最后一个元素的位置
  for(i=0;i<pos-1;i++) r=r->next; //定位到删除点前一个元素的位置
  p=r->next;
  r->next=p->next;
  free(p); //对指定位置节点进行删除
  head->score--; //及时更新元素个数
  return 0;
} 

int showList(const STUP head) //将列表中的所有元素进行打印
{
  STUP r=head;
  if(0 != ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(r=head->next;r;r=r->next) printf("(%03d,%d)",r->ID,r->score); //依次将各节点的ID和score进行显示
  printf("\n");
  return 0;
}

int searchNode(const STUP head,int score) //搜索列表中指定分数的节点
{
  STUP r=NULL;
  int res=-1;
  if(0 != ifEmptyList(head) )return res; //操作前进行一下检查，判断此表是否为空
  for(r=head->next;r;r=r->next)  //遍历所有节点
  {
   if (r->score >= score) //依次对各节点的score进行比较和判断，显示满足条件的节点信息
   {
	 printf("(%03d,%d)",r->ID,r->score); 
     res=0;
   }
  }
  printf("\n");
  return res;
}

int sortListDesc(const STUP head) //对链表进行降序排序
{
  STUP p=NULL,q=NULL;
  int tmp=0;
  if(0 != ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(p=head->next;p;p=p->next) //冒泡排序的思想进行排序
  {
    for(q=p->next;q;q=q->next)
    {
      if(p->score < q->score)
      {
	tmp=p->score;
	p->score=q->score;
	q->score=tmp;
	
	tmp=p->ID;
	p->ID=q->ID;
	q->ID=tmp;
      }
    }
  }
  return 0;  
}

int sortListAsc(const STUP head) //对链表进行升序排序
{
  STUP p=NULL,q=NULL;
  int tmp=0;
  if(0 != ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(p=head->next;p;p=p->next)  //冒泡排序的思想进行排序
  {
    for(q=p->next;q;q=q->next)
    {
      if(p->score > q->score)
      {
	tmp=p->score;
	p->score=q->score;
	q->score=tmp;
	
	tmp=p->ID;
	p->ID=q->ID;
	q->ID=tmp;
      }
    }
  }
  return 0;  
}


int filterList(const STUP head,int score) //删除掉小于指定分数的记录
{
  STUP p=NULL,q=NULL;
  if(0 != ifEmptyList(head) )return -1; //操作前进行一次检查，判断此表是否为空
  for(p=head,q=p->next;q;) //遍历所有节点
  {
    if(q->score < score) //删除掉满足条件的节点
    {
      p->next=q->next;
      free(q);
      q=p->next;
      head->score--; //及时更新元素个数
    }
    else
    {
      p=p->next;
      q=p->next;
    }
  }
  return 0;
}

int getNode(const STUP head,const int id) //获取表中指定ID元素的值
{
  STUP p=NULL;
  int res=-1;
  if(0 != ifEmptyList(head) )return res; //操作前进行一次检查，判断此表是否为空
  for(p=head->next;p;p=p->next)  //遍历所有节点
  {
    if(id == p->ID) //打印满足条件的节点信息
    {
      printf("(%03d,%d)",p->ID,p->score);
      res=0;
    }
  }
  printf("\n");
  return res;
}

int modifyNode(STUP head,int id,int score) //将表中指定ID的元素修改为新值
{
  STUP p=NULL;
  int res=-1;
  if(0 != ifEmptyList(head) )return res; //操作前进行一次检查，判断此表是否为空
  for(p=head->next;p;p=p->next) //遍历所有节点
  {
    if(id == p->ID) //将满足条件的节点分数进行更新，并且打印
    {
      p->score=score;
      printf("(%03d,%d)",p->ID,p->score);
      res=0;
    }
  }
  printf("\n");
  return res;
}

int clearList(const STUP head) //清空表
{
  STUP p=head;
  while(head->score)delNode(head,0); //使用之前定义的删除函数，进行循环删除，直到链表中元素个数为0
  free(p); //将头节点一并销毁
  printf("the list has been flushed\n");
  return 0;
}


int main()
{
  STUP head=NULL;
  //create a empty list
  printf("create a emplty list\n"); 
  head = createList(); //创建链表测试
  //insert node to list
  printf("insert 5 node into list\n");
  instNode(head,1,40,100);
  instNode(head,2,55,100);
  instNode(head,3,90,100);
  instNode(head,4,55,100);
  instNode(head,5,95,100);   //添加元素测试
  //show list
  printf("show list\n");
  showList(head); //打印测试
  //insert node to specil postion
  printf("insert nodes on pos 1,7,3,20,-5\n");
  instNode(head,6,75,1);
  instNode(head,7,50,7);
  instNode(head,8,50,3);
  instNode(head,9,90,20);
  instNode(head,10,80,-5);  //特殊位置插入测试
  showList(head);
  //search node from list which score above 85
  printf("search nodes which score above 85\n");
  searchNode(head,85);  //搜索测试
  //get node from list
  printf("get node by id which is 8\n");
  getNode(head,8); //查询元素测试
  //sore list desc
  printf("sort list by desc order\n");
  sortListDesc(head); //降序排序测试
  showList(head);
  //sort list asc
  printf("sort list by asc order\n");
  sortListAsc(head); //升序排序测试
  showList(head);
  //delete node from list
  printf("delete three node on pos 20,10,5,1,-4\n");
  delNode(head,20);
  delNode(head,10);
  delNode(head,5);
  delNode(head,1);
  delNode(head,-4); //分别取五个有代表性的位置进行删除测试
  showList(head);
  //delete the record which below 60
  printf("delete the record which below 60\n");
  filterList(head,60); //过滤测试
  showList(head);
  //modify the value of a node
  printf("modify node score to 19 which id is 10\n");
  modifyNode(head,10,19); //修改值测试
  showList(head);
  //clear the list
  printf("clear the list\n");
  clearList(head); //清表测试
  return 0;
}
~~~

编译执行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc toblog.x toblog.c
emacs@ubuntu:~/c$ ./toblog.x 
create a emplty list
insert 5 node into list
show list
(001,40)(002,55)(003,90)(004,55)(005,95)
insert nodes on pos 1,7,3,20,-5
(010,80)(006,75)(001,40)(008,50)(002,55)(003,90)(004,55)(005,95)(007,50)(009,90)
search nodes which score above 85
(003,90)(005,95)(009,90)
get node by id which is 8
(008,50)
sort list by desc order
(005,95)(003,90)(009,90)(010,80)(006,75)(002,55)(004,55)(007,50)(008,50)(001,40)
sort list by asc order
(001,40)(008,50)(007,50)(004,55)(002,55)(006,75)(010,80)(009,90)(003,90)(005,95)
delete three node on pos 20,10,5,1,-4
(007,50)(004,55)(006,75)(010,80)(009,90)
delete the record which below 60
(006,75)(010,80)(009,90)
modify node score to 19 which id is 10
(010,19)
(006,75)(010,19)(009,90)
clear the list
the list has been flushed
emacs@ubuntu:~/c$ 
~~~



[visualgo]:https://visualgo.net/
