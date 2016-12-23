---
layout:  post
title:  数据结构基础(六)
author:  wilmosfang
tags:   c 
categories:  c
wc:  189  278 5047 
excerpt:  二叉树，前序遍历，中序遍历，后序遍历，层次遍历
comments: true
---


# 前言


除了前面几篇中介绍到的线性表，还有一种特别常见的数据结构 **树** ，它有如下特征：

* 每个节点有零个或多个子节点
* 没有父节点的节点称为根节点
* 每一个非根节点有且只有一个父节点
* 除了根节点外，每个子节点可以分为多个不相交的子树

国家或公司的层级结构，家族的族谱，树叶的脉络都是这种结构

计算机中的文件层次，数据库中的索引也都是这种结构的应用

这里分享一下我在学习树型数据结构过程中的一些笔记，前面一篇用C语言实现了一个简单的 **队**，这里用C语言实现一个简单的 **二叉树**，并且实现它的几种常见遍历方法

> **Tip:** 有一个网站 **[VisuAlgo][visualgo]** 能将数据结构进行可视化展示

---

# 概要

* TOC
{:toc}

---



## 二叉树

二叉树是树的一种特殊情况，它有如下特征：

* 每个结点最多有两棵子树
* 左子树和右子树，次序不可以颠倒
* 非空二叉树的第n层上至多有2^(n-1)个元素
* 深度为h的二叉树至多有2^h-1个结点

二叉树中又有两种特殊的情况：

* 满二叉树：所有终端都在同一层次，且非终端结点的度数为2，在满二叉树中若其深度为h，则其所包含的结点数必为2^h-1
* 完全二叉树：除了最大的层次即成为一棵满二叉树且层次最大那层所有的结点均向左靠齐，即集中在左面的位置上，不能有空位置


树与线性表一样，在计算机存储器中的表示一般有两种形式，一种是 **顺序映象**，一种是 **链式映象** ，顺序存储的局限性在于，空间大小固定，容易溢出，也不够灵活

这里我使用带指针的结构体来实现一个简单的树：

---

## 代码示例


~~~
#include <stdio.h>
#include <malloc.h> //要使用到动态内存分配的时候得带上这个头文件
#define MAXSIZE 100

typedef struct tree //定义一个树的结构体
{
  int data;
  struct tree *l; //左子树指针
  struct tree *r; //右子树指针
}TN,*TP; //重命名此结构体结点为TN，重命名指针为TP

TP initTree() //初始化二叉树，手动生成一个简单的二叉树
{
  TP pa,pb,pc,pd,pe,pf; //定义六个节点的指针
  pa=(TP)malloc(sizeof(TN));
  pb=(TP)malloc(sizeof(TN));
  pc=(TP)malloc(sizeof(TN));
  pd=(TP)malloc(sizeof(TN));
  pe=(TP)malloc(sizeof(TN));
  pf=(TP)malloc(sizeof(TN)); //分别分配六段内存
  
  pa->l=pb;
  pa->r=pc;
  pa->data='A'; //初始化A节点

  pb->l=pd;
  pb->r=pe;
  pb->data='B'; //初始化B节点
  
  pc->l=pf;
  pc->r=NULL;
  pc->data='C'; //初始化C节点
  
  pd->l=NULL;
  pd->r=NULL;
  pd->data='D'; //初始化D节点
  
  pe->l=NULL;
  pe->r=NULL;
  pe->data='E'; //初始化E节点
  
  pf->l=NULL;
  pf->r=NULL;
  pf->data='F'; //初始化F节点
  
  return pa;  //反馈a节点作为根节点
}

void preOrder(TP root) //前序遍历
{
  if(NULL == root) return; 
  printf("%c",root->data); //先打印节点值
  preOrder(root->l); //再依次进行左子树递归
  preOrder(root->r); //再依次进行右子树递归

}

void inOrder(TP root) //中序遍历
{
  if(NULL == root) return;
  inOrder(root->l);  //先进行左子树递归
  printf("%c",root->data); //再打印节点值
  inOrder(root->r);  //再依次进行右子树递归
}

void postOrder(TP root) //后序遍历
{
  if(NULL == root) return;
  postOrder(root->l); //先进行左子树递归
  postOrder(root->r); //再依次进行右子树递归
  printf("%c",root->data); //再打印节点值
}

void noOrder(TP root) //层次遍历
{
  TP que[MAXSIZE]; //定义一个节点的指针数组
  int e=0,h=0; //定义队列首尾的下标
  if(NULL == root) return; 
  for(e=0;e<MAXSIZE;e++)que[e]=NULL; //初始化为空
  h=e=0; //首尾置0
  que[e]=root; //将根存入第一个数组元素中
  while (NULL != que[h]) //如果队首节点不为空就循环执行下列代码
  {
    printf("%c",que[h]->data);  //打印出节点值
    if(NULL != que[h]->l) //如果左子叶存在则将其存入队列尾部
      que[++e]=que[h]->l;
    if(NULL != que[h]->r) //如果右子叶存在则将其存入队列尾部
      que[++e]=que[h]->r;
    h++; //处理完一个节点，就向队列中下一个节点移动
  }
}

int main()
{
  TP root=initTree(); //初始化测试
  preOrder(root); //前序遍历测试
  printf("\n");
  inOrder(root); //中序遍历测试
  printf("\n");
  postOrder(root); //后序遍历测试
  printf("\n");
  noOrder(root); //层次遍历测试
  printf("\n");
  return 0;
}
~~~

编译执行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc tree.x tree.c
emacs@ubuntu:~/c$ ./tree.x 
ABDECF
DBEAFC
DEBFCA
ABCDEF
emacs@ubuntu:~/c$ 
~~~

[visualgo]:https://visualgo.net/

