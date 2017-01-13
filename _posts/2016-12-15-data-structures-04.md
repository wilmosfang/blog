---
layout:  post
title:  数据结构基础(四).栈
author:  wilmosfang
tags:   c 
categories:  c
wc:  213  404 5836 
excerpt:  线性表，栈
comments: true
---


# 前言

**线性表** 是一种应用广泛和最为基础的数据结构

线性表的特征：对非空表，a(0)是表头，无前驱；a(n-1)是表尾，无后继；其它的每个元素a(i)有且仅有一个直接前驱a(i-1)和一个直接后继a(i+1)

线性表在计算机存储器中的表示一般有两种形式，一种是 **顺序映象**，一种是 **链式映象**

有一个网站 **[VisuAlgo][visualgo]** 能将数据结构进行可视化展示

这里分享一下我在学习线性表过程中的一些笔记，前面一篇用C语言实现了一个简单的 **双链表**，这里用C语言实现一个简单的 **栈**

---

# 概要

* TOC
{:toc}

---



## 栈

栈是限制在一端进行插入操作和删除操作的线性表，允许进行操作的一端称为“栈顶”，另一固定端称为“栈底”，当栈中没有元素时称为“空栈”，特点是后进先出(LIFO)

> **Tip:** 栈是线性表的一种特殊形式

> **Note:** 也有将 **栈** 称为 **堆栈** 的，但其实 **堆** 和 **栈** 在计算机里是代表不同的两种存储机制

线性表在计算机存储器中的表示一般有两种形式，一种是 **顺序映象**，一种是 **链式映象**，因此栈也可以使用这两种方式来实现，顺序表实现的局限性在于，空间大小固定，容易溢出，也就是所谓的 **stack overflow** , 有一个知名的技术知识问答平台就是以此命名的，估计猴子们看到这个会油然一种亲切感

这里我使用单链表不带表头的方式，实现一个简单的栈

---

## 代码示例


~~~
#include <stdio.h>
#include <malloc.h>

typedef struct stack //创建一个栈结构
{
  int score;
  struct stack *next; //节点里只用包含当前节点的值和，下一节点的位置
}ST,*SP,**SPP; //重命名栈节点结构类型为ST,指针为SP，指针的指针为SPP(第一次用这种高级玩法)

int push(SPP tpp,int score) //入栈操作，注意第一个参数为节点指针的指针
{
  SP newnode=NULL;
  if(NULL != *tpp) //不为空的情况下创建新节点，赋值，并且置于栈顶
  {
    newnode=(SP)malloc(sizeof(ST)); //动态申请内存，创建节点
    if(NULL == newnode)  //进行判断，申请内存失败则提醒并且返回
    {
      printf("push error:no enough memory!\n");
      return -1;
    }
    newnode->score=score;
    newnode->next=*tpp; //置于栈顶
    *tpp=newnode; //将栈顶进行更新
    return 0;
  }
  else //为空的情况下，就创建第一个栈节点
  {
    *tpp=(SP)malloc(sizeof(ST)); //动态申请内存，创建节点
    if(NULL == *tpp)  //进行判断，申请内存失败则提醒并且返回
    {
      printf("error:no enough memory!\n");
      return -1;
    }
    (*tpp)->score=score;
    (*tpp)->next=NULL; //初始化栈底
    return 0;
  }
}

int pop(SPP tpp) //出栈操作，注意第一个参数为节点指针的指针
{
  SP sp=NULL;
  int x=0;

  if(NULL != *tpp) //进行判断，是否为空，不为空则返回栈顶的值，并且销毁节点，更新栈顶
  {
    x = (*tpp)->score; //将栈顶的值进行保存
    sp=*tpp; //这里要借用一个结构体指针临时存放一下栈顶位置
    *tpp = sp->next; //更新栈顶位置
    free(sp); //对原来栈顶进行销毁
    return x; //返回栈顶的值
  }
  else //为空则提醒，并且返回
  {
    printf("pop error:this is a empty stack!\n");
    return -1;
  }  
}

int count(SP top) //计算堆栈元素个数
{
  int i=0; 
  if(NULL != top) 
  {
    for(i=0;top;top=top->next)i++; // the same as : i=1;while(top=top->next)i++; 有多种写法时，可以选用一种最为简洁的形式
    return i;
  }
  else return i;
}

int clear(SPP tpp) //清空栈
{
  if(NULL != *tpp)
  {
    while(*tpp)pop(tpp); //只用循环弹出，直到为空，然后提示退出
    printf("this stack has been clean\n");
    return 0;
  }
  else //本来为空的时候，直接提示退出
  {
    printf("this is an empty stack\n");
    return 0;
  }
}

int show(SP top) //打印所有栈元素
{
  for(;top;top=top->next)printf("(%d)",top->score);
  printf("\n");
  return 0;
}



int main()
{
  SP top=NULL; //这个地方一定要赋初值为NULL，否则会有bug，因为对于是否为栈底的判断依据来源于此
  printf("push 50,80,70,90,100 into stack\n");
  push(&top,50);
  push(&top,80);
  push(&top,70);
  push(&top,90);
  push(&top,100); //压栈测试
  printf("show all nodes in stack\n");
  show(top); //打印测试
  printf("the total count is: %d\n",count(top));
  printf("pop stack\n");
  printf("%d\n",pop(&top)); //出栈测试
  show(top);
  printf("the total count is: %d\n",count(top)); //计数测试
  printf("pop stack\n");
  printf("%d\n",pop(&top));
  show(top);
  printf("the total count is: %d\n",count(top));
  printf("pop stack twice\n");
  printf("%d\n",pop(&top));
  printf("%d\n",pop(&top)); //出栈测试
  printf("clear stack\n"); //清栈测试
  clear(&top);
  printf("pop stack twice\n"); 
  printf("%d\n",pop(&top));
  printf("%d\n",pop(&top)); //对空栈进行出栈测试
  return 0;
}
~~~

编译执行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc stack.x  stack.c
emacs@ubuntu:~/c$ ./stack.x 
push 50,80,70,90,100 into stack
show all nodes in stack
(100)(90)(70)(80)(50)
the total count is: 5
pop stack
100
(90)(70)(80)(50)
the total count is: 4
pop stack
90
(70)(80)(50)
the total count is: 3
pop stack twice
70
80
clear stack
this stack has been clean
pop stack twice
pop error:this is a empty stack!
-1
pop error:this is a empty stack!
-1
emacs@ubuntu:~/c$
~~~


[visualgo]:https://visualgo.net/
