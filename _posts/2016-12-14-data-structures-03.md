---
layout:  post
title:  数据结构基础(三)
author:  wilmosfang
tags:   c 
categories:  c
wc:  382   810 11072 
excerpt:  线性表，链表，双向链表
comments: true
---


# 前言

**线性表** 是一种应用广泛和最为基础的数据结构

线性表的特征：对非空表，a(0)是表头，无前驱；a(n-1)是表尾，无后继；其它的每个元素a(i)有且仅有一个直接前驱a(i-1)和一个直接后继a(i+1)

线性表在计算机存储器中的表示一般有两种形式，一种是 **顺序映象**，一种是 **链式映象**

有一个网站 **[VisuAlgo][visualgo]** 能将数据结构进行可视化展示

这里分享一下我在学习线性表过程中的一些笔记，前面一篇用C语言实现了一个简单的单链表，这里用C语言实现一个简单的 **双链表**

---

# 概要

* TOC
{:toc}

---



## 链表结构

将线性表中各元素分布在存储器的不同存储块中，通过地址或指针建立它们之间的联系，所得到的的存储结构为链表结构

链表结构根据指向的特性，分为 **单向链表** 和 **双向链表** 

> **Tips:** 双链表和单链表的区别就是每个节点不仅存储了下一个节点的地址，还存储了上一个节点的地址


> **Tips:** 单双循环链表是它们的变种，将首尾连接就成了循环链表，添加删除节点的操作方法不变


---

## 代码示例


~~~
#include <stdio.h> 
#include <malloc.h>

typedef struct dlist 
{
  int score;
  struct dlist *prev; //相对于单链表，双链表有前置节点
  struct dlist *next; 
}DL,*DP; //重命名双链节点类型为DL，双链指针类型为DP


DP createList() //创建空表
{
  DP head=NULL;
  head=(DP)malloc(sizeof(DL));
  if(NULL == head) //跟进检查，如果申请失败则提醒返回，将NULL放在左边是一种更安全的做法
  {
    printf("error:no enough memory!\n");
    return NULL;
  }
  head->score=0; //初始化头结点的score，这个地方用来存储链表中的元素个数
  head->next=NULL; 
  head->prev=NULL; //由于是空表，将前置和后继节点置空
  return head; //返回此头节点
}

int instNode(DP const head,int pos,int score) //在列表中的指定位置插入给定socre的记录
{
  DP p=NULL,r=head; //给变量进行初始化是一个好习惯，特别是指针，可以有效避免野指针的潜在隐患
  int i=0;  
  if(pos < 1) pos=1; //对插入位置进行校正，位置小于1时，定位到1位置
  if(pos > head->score + 1) pos=head->score + 1; //对插入位置进行校正，位置超出最后一个元素时，定位到末尾位置
  p=(DP)malloc(sizeof(DL)); //申请内存，创建一个节点
  if(NULL == p) //跟进检查，如果申请失败则提醒返回，将NULL放在左边是一种更安全的做法
  {
    printf("error:no enough memory!\n");
    return -1;
  }
  p->score=score; //初始化score为给定值
  for(i=0;i<pos-1;i++) r=r->next;  //定位到插入点前一个元素的位置
  p->next=r->next;
  p->prev=r; 
  if(r->next)r->next->prev=p; //对于链尾情况的特殊照顾
  r->next=p; //挂接新节点，这个过程的关键就是前置结点的next指针一定要最后再修改
  head->score++; //及时跟进最大下标
  return 0;
}
 

int ifEmptyList(const DP head) //用来判断此表是否为空，如果为空则返回0
{
  if(0 == head->score)
  {
    printf("warning:empty list!\n");
    return 0;
  }
  else return -1;
} 

int delNode(DP const head,int pos) //在列表中指定的位置删除一个节点
{
  DP r=head,p=NULL;
  int i=0;
  if(0 == ifEmptyList(head) )return -1; //删除前进行一下检查，判断此表是否为空
  if(1 > pos) pos=1; //对删除位置进行校正，位置小于1时，定位到1位置
  if(pos > r->score) pos=r->score; //对删除位置进行校正，位置超出最后一个元素时，定位到最后一个元素的位置
  for(i=0;i<pos-1;i++) r=r->next; //定位到删除点前一个元素的位置
  p=r->next; 
  if(p->next)p->next->prev=r;  //对于链尾情况的特殊照顾
  r->next=p->next; //断线此节点，这个过程的关键就是前置结点的next指针一定要最后再修改
  free(p); //释放节点空间
  head->score--; //及时更新元素个数
  return 0;
} 

int showList(const DP head) //将列表中的所有元素进行打印
{
  DP r=head;
  if(0 == ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(r=head->next;r;r=r->next) printf("(%d)",r->score); //依次将各节点的score进行显示
  printf("\n");
  return 0;
}

int showNodesAbove(const DP head,int score) //将列表中大于指定分数的节点进行打印
{
  DP r=head;
  int res=-1;
  if(0 == ifEmptyList(head) )return res; //操作前进行一下检查，判断此表是否为空
  for(r=head->next;r;r=r->next)  //遍历表中所有节点
  {
    if(r->score > score) //将满足条件的节点进行打印
    {
      printf("(%d)",r->score); 
      res=0;
    }
  }
  printf("\n");
  return res;
}


int sortListDesc(const DP head) //对链表进行降序排序
{
  DP p=NULL,q=NULL;
  int tmp=0;
  if(0 == ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(p=head->next;p;p=p->next) //冒泡排序的思想进行排序
  {
    for(q=p->next;q;q=q->next)
    {
      if(p->score < q->score)
      {
	tmp=p->score;
	p->score=q->score;
	q->score=tmp;
      }
    }
  }
  return 0;  
}

int sortListAsc(const DP head) //对链表进行升序排序
{
  DP p=NULL,q=NULL;
  int tmp=0;
  if(0 == ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(p=head->next;p;p=p->next) //冒泡排序的思想进行排序
  {
    for(q=p->next;q;q=q->next) 
    {
      if(p->score > q->score)
      {
	tmp=p->score;
	p->score=q->score;
	q->score=tmp;
      }
    }
  }
  return 0;  
}


int filterListBelow(const DP head,int score) //删除掉小于指定分数的记录
{
  DP p=NULL,r=NULL;
  if(0 == ifEmptyList(head) )return -1; //操作前进行一次检查，判断此表是否为空
  for(r=head,p=r->next;p;) //遍历所有节点
  {
    if(p->score < score) //删除掉满足条件的节点
    {
      r->next=p->next;
      if(p->next)p->next->prev=r; //对尾节点要进行特殊处理
      free(p);
      p=r->next;
      head->score--; //及时更新元素个数
    }
    else
    {
      r=r->next;
      p=r->next;
    }
  }
  return 0;
}

int getNodePos(const DP head,const int score) //根据score来获取节点位置，获取满足条件的第一个节点位置
{
  DP p=NULL;
  int i=0;
  if(0 == ifEmptyList(head) )return -1; //操作前进行一下检查，判断此表是否为空
  for(i=1,p=head->next;p;p=p->next,i++)
  {
    if(score == p->score) return i;  //如果找到，就反馈出当前位置
  }
  return -1;
}

int instNodeAfter(DP head,int score,int newscore) //根据score来插入新节点，只插入在第一个满足条件的节点后面
{
  if(-1 ==getNodePos(head,score)) //检查列表中有没有此score的节点，没有就提示并返回
  {
    printf("the score %d not found\n",score);
    return -1;
  }
  else  //如果找到，就进行插入操作
  {
    instNode(head,getNodePos(head,score)+1,newscore); 
    return 1;
  }
}

int modifyNode(DP head,int score,int newvalue) //将列表中指定值的节点修改为新值
{
  DP p=NULL;
  int res=-1;
  if(0 == ifEmptyList(head) )return res; //操作前进行一下检查，判断此表是否为空
  for(p=head->next;p;p=p->next) //遍历所有节点
  {
    if(score == p->score) //将满足条件的节点进行修改，并且打印
    {
      p->score=newvalue;
      printf("(%d)",p->score);  
      res=0;
    }
  }
  printf("\n");
  return res;
}

int clearList(const DP head) //清空表并给出提示
{
  DP p=head;
  while(head->score)delNode(head,0); //使用之前定义的删除函数，进行循环删除，直到链表中元素个数为0
  free(p); //将头节点一并销毁
  printf("the list has been flushed\n");
  return 0;
}


int main()
{
  DP head=NULL;
  //create a empty list
  printf("create a emplty list\n");
  head = createList(); //创建空链测试
  printf("show list\n");
  //show list
  showList(head); //打印测试
  //insert node to list
  printf("insert 5 node into list\n");
  instNode(head,100,70);
  instNode(head,100,80);
  instNode(head,100,50);
  instNode(head,100,100);
  instNode(head,100,50); //添加元素测试
  showList(head);
  //insert a node after the first node which socre is 50
  printf("insert a node after the first node which socre is 50\n");
  instNodeAfter(head,50,60);  //根据score值插入测试
  showList(head); 
  //delete nodes which score is 50 
  printf("delete nodes from list which score is 50\n"); 
  while(-1 != getNodePos(head,50))delNode(head,getNodePos(head,50)); // 删除所有score为50的节点
  showList(head);  
  //show node which score is above 75
  printf("show nodes which score is above 75\n");
  showNodesAbove(head,75); //节点过滤测试
  showList(head);
  //insert nodes on pos 30,5,3,1,-20
  printf("insert nodes on pos  30,5,3,1,-20\n");
  instNode(head,30,11);
  instNode(head,5,22);
  instNode(head,3,33);
  instNode(head,1,44);
  instNode(head,-20,55);  //特殊位置插入测试
  showList(head);
  //sore list desc
  printf("sort list by desc order\n");
  sortListDesc(head); //降序排序测试
  showList(head);
  //sort list asc
  printf("sort list by asc order\n");
  sortListAsc(head); //升序排序测试
  showList(head);
  //delete nodes from list on pos 100,8,5,1,-1 
  printf("delete nodes on pos 100,8,5,1,-1\n");
  delNode(head,100);
  delNode(head,8);
  delNode(head,5);
  delNode(head,1);
  delNode(head,-1); //特殊位置删除测试
  showList(head);
  //delete the record which below 60
  printf("delete the record which below 60\n");
  filterListBelow(head,60); //过滤删除测试
  showList(head);
  //modify the value of a node
  printf("modify node score to 19 which score is 70\n");
  modifyNode(head,70,19); //修改值测试
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
show list
warning:empty list!
insert 5 node into list
(70)(80)(50)(100)(50)
insert a node after the first node which socre is 50
(70)(80)(50)(60)(100)(50)
delete nodes from list which score is 50
(70)(80)(60)(100)
show nodes which score is above 75
(80)(100)
(70)(80)(60)(100)
insert nodes on pos  30,5,3,1,-20
(55)(44)(70)(80)(33)(60)(100)(22)(11)
sort list by desc order
(100)(80)(70)(60)(55)(44)(33)(22)(11)
sort list by asc order
(11)(22)(33)(44)(55)(60)(70)(80)(100)
delete nodes on pos 100,8,5,1,-1
(33)(44)(60)(70)
delete the record which below 60
(60)(70)
modify node score to 19 which score is 70
(19)
(60)(19)
clear the list
the list has been flushed
emacs@ubuntu:~/c$ 
~~~



[visualgo]:https://visualgo.net/

