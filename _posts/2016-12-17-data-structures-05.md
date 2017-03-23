---
layout:  post
title:  数据结构基础(五).队
author:  wilmosfang
tags:   c 
categories:  c
wc: 267 640 7540
excerpt:  线性表，队
comments: true
---


# 前言

**线性表** 是一种应用广泛和最为基础的数据结构

线性表的特征：对非空表，a(0)是表头，无前驱；a(n-1)是表尾，无后继；其它的每个元素a(i)有且仅有一个直接前驱a(i-1)和一个直接后继a(i+1)

线性表在计算机存储器中的表示一般有两种形式，一种是 **顺序映象**，一种是 **链式映象**

有一个网站 **[VisuAlgo][visualgo]** 能将数据结构进行可视化展示

这里分享一下我在学习线性表过程中的一些笔记，前面一篇用C语言实现了一个简单的 **栈**，这里用C语言实现一个简单的 **队**

---

# 概要

* TOC
{:toc}

---



## 队

队列是限制在两端进行插入操作和删除操作的线性表，允许进行存入操作的一端称为“队尾”，允许进行删除操作的一端称为“队头”，当线性表中没有元素时，称为“空队”，特点是先进先出(FIFO)

> **Tip:** 队是线性表的一种特殊形式

线性表在计算机存储器中的表示一般有两种形式，一种是 **顺序映象**，一种是 **链式映象**，因此队也可以使用这两种方式来实现，顺序表实现的局限性在于，空间大小固定，容易溢出

这里我使用单链表带表头的方式，实现一个简单的队

---

## 代码示例


~~~
#include <stdio.h>
#include <malloc.h>


typedef struct qnode //定义一个队列节点结构体
{
  int score;
  struct qnode *next;
}QN,*NP;   //重命名结构体为QN,指针为NP

typedef struct queue //定义一个队列头结构体
{
  int count; //用来存储所有成员的个数
  struct qnode *front; //指向队头节点
  struct qnode *rear; //指向队尾节点
}QU,*QP; //重命名结构体为QU,指针为QP


QP INIT() //初始化队列
{
  QP head=NULL; 
  head=(QP)malloc(sizeof(QU)); //动态申请内存，创建头节点
  if(NULL == head) //进行判断，申请内存失败则提醒并且返回
  {
    printf("xxx error:no enough memory!\n");
    return NULL;
  }
  head->count=0; //初始化元素个数为0
  head->front=NULL; 
  head->rear=NULL; //初始化队头队尾为空
  return head; //将头节点位置返回
}

int IFEMPTY(const QP head) //判断队列是否为空，为空则返回0
{
  if (0 == head->count) return 0;
  else return -1;
}

int ENQUEUE(const QP head,const int score) //入队操作
{
  NP np=NULL;
  np=(NP)malloc(sizeof(QN)); //动态申请内存，创建节点
  if(NULL == np)  //进行判断，申请内存失败则提醒并且返回
  {
    printf("error:no enough memory!\n");
    return -1;
  }
  np->score=score; //给节点赋值
  np->next=NULL; //插入队尾，后继置空
  if(NULL != head->rear) head->rear->next=np; //避开第一交的插入操作，对第一次插入的节点要进行特殊对待
  head->rear=np; //更新队尾指针
  if(NULL == head->front) head->front=np; //对第一次插入操作进行特殊处理，如果是第一次插入，就将队头也指向这个结节
  head->count++; //及时更新元素个数
  return 0;
}


int DEQUEUE(const QP head) //出队操作
{
  NP np=NULL;
  int res=-1;
  if(0 == IFEMPTY(head)) //操作前进行一下检查，判断此队是否为空
  {
    printf("DEQUEUE error:this is a empty queue!\n");
    return -1;
  }
  np=head->front;  //将地址进行缓存
  head->front=head->front->next; 
  if(NULL == head->front )head->rear=NULL; //对于只有一个节点的出队操作进行特殊处理，将队尾置空
  res=np->score; //取出分数
  free(np); //释放节点空间
  head->count--; //及时更新元素个数
  return res; //将数值返回
}

int FRONT(const QP head) //取出队头数据，但不进行出队操作
{
  if(0 == IFEMPTY(head)) //操作前进行一下检查，判断此队是否为空
  {
    printf("DEQUEUE error:this is a empty queue!\n");
    return -1;
  }
  return head->front->score; //返回队头值
}


int COUNT(const QP head) //计算元素个数
{
  return head->count; //取出元素个数，进行返回
}

int SHOW(const QP head) //打印所有元素
{
  NP np=NULL;
  if(0 == IFEMPTY(head)) //操作前进行一下检查，判断此队是否为空
  {
    printf("DEQUEUE error:this is a empty queue!\n");
    return -1;
  }
  for(np=head->front;np;np=np->next)printf("(%d)",np->score); //遍历元素进行打印
  printf("\n");
  return 0;
}

int SETNULL(const QP head) //清空队列
{
  while(head->count)DEQUEUE(head); //循环出队
  return 0;
}


int main()
{
  ////INIT
  QP que=NULL; 
  que=INIT(); //初始化测试
  ////SHOW
  SHOW(que); //打印测试
  ////ENQUEUE
  printf("add five nodes to this queue 50,80,70,90,100\n"); 
  ENQUEUE(que,50);
  ENQUEUE(que,80);
  ENQUEUE(que,70);
  ENQUEUE(que,90);
  ENQUEUE(que,100); //入队测试
  SHOW(que); 
  ////COUNT
  printf("the total number of nodes in queue is:%d\n",COUNT(que)); //计数测试
  ////DEQUEUE
  printf("the DEQUEUE ops result is:%d\n",DEQUEUE(que));
  SHOW(que);
  printf("the DEQUEUE ops result is:%d\n",DEQUEUE(que));
  SHOW(que);
  printf("the DEQUEUE ops result is:%d\n",DEQUEUE(que)); //出队测试
  SHOW(que);
  printf("the total number of nodes in queue is:%d\n",COUNT(que));  
  SHOW(que);
  ////FRONT
  printf("the front node in queue is:%d\n",FRONT(que));
  SHOW(que);
  printf("the total number of nodes in queue is:%d\n",COUNT(que));
  printf("the front node in queue is:%d\n",FRONT(que)); 
  SHOW(que);
  printf("the total number of nodes in queue is:%d\n",COUNT(que));
  printf("the front node in queue is:%d\n",FRONT(que));  //打印队头测试
  SHOW(que);
  printf("the total number of nodes in queue is:%d\n",COUNT(que));
  ////SETNULL
  printf("ready to clear this queue\n");
  SETNULL(que); //清队测试
  SHOW(que);
  ////ENQUEUE
  printf("add two more nodes into this queue 50,80\n");
  ENQUEUE(que,50);
  ENQUEUE(que,80);  //进一步的入队出队测试
  SHOW(que);
  ////DEQUEUE
  printf("the DEQUEUE ops result is:%d\n",DEQUEUE(que));
  SHOW(que);
  printf("the DEQUEUE ops result is:%d\n",DEQUEUE(que));
  SHOW(que);
  printf("the DEQUEUE ops result is:%d\n",DEQUEUE(que));
  SHOW(que);
  printf("the total number of nodes in queue is:%d\n",COUNT(que));
  SHOW(que);
  return 0;
}
~~~

编译执行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc queue.x queue.c
emacs@ubuntu:~/c$ ./queue.x 
DEQUEUE error:this is a empty queue!
add five nodes to this queue 50,80,70,90,100
(50)(80)(70)(90)(100)
the total number of nodes in queue is:5
the DEQUEUE ops result is:50
(80)(70)(90)(100)
the DEQUEUE ops result is:80
(70)(90)(100)
the DEQUEUE ops result is:70
(90)(100)
the total number of nodes in queue is:2
(90)(100)
the front node in queue is:90
(90)(100)
the total number of nodes in queue is:2
the front node in queue is:90
(90)(100)
the total number of nodes in queue is:2
the front node in queue is:90
(90)(100)
the total number of nodes in queue is:2
ready to clear this queue
DEQUEUE error:this is a empty queue!
add two more nodes into this queue 50,80
(50)(80)
the DEQUEUE ops result is:50
(80)
the DEQUEUE ops result is:80
DEQUEUE error:this is a empty queue!
DEQUEUE error:this is a empty queue!
the DEQUEUE ops result is:-1
DEQUEUE error:this is a empty queue!
the total number of nodes in queue is:0
DEQUEUE error:this is a empty queue!
emacs@ubuntu:~/c$ 
~~~


[visualgo]:https://visualgo.net/
