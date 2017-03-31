---
layout:  post
title:  文件I/O (一).非缓冲IO实现mycopy
author:  wilmosfang
tags:   c 
categories:  c
wc: 194 384 5896
excerpt:  文件IO、open/read/write/lseek/close、非缓冲IO
comments: true
---


# 前言

当前的计算系统除了包括对数据有 **加工和处理** 以外还有 **搬运**

这个 **搬运** 代表着 **输入和输出** ，及 **input/output** ，简称 **I/O**

**UNIX/Linux** 的缔造者们将数据的 **来源和目标** 都抽象为 **文件**，所以在 **UNIX/Linux** 系统中 **一切皆文件**

**一切皆文件** 不仅仅对磁盘，还包括鼠标，键盘，显示器这些设备，那么对这些设备的操作也都抽象成了对 **文件的I/O操作**

关于 **标准I/O** 可以参看之前的文章 **[《标准I/O (一)》][c_stdio_01]** ，类Unix系统中除了 **标准I/O** 还有 **文件I/O**，可以完成相同工作，关于C语言的API(linux)可以参看 **[Linux C API 参考手册][linux_c_api]** 在线文档

这里分享一下我在学习 **文件 I/O** 库过程中的笔记和心得


---

# 概要

* TOC
{:toc}

---


## 文件I/O

**文件I/O** 可以实现 **标准I/O** 一样的功能，包括打开文件，读取文件，写入文件，关闭文件等操作

**文件I/O** 主要包含：**`open/read/write/lseek/close`** 几个函数，大多数操作都可以由这几个函数来完成，相对于 **标准I/O** 封装得更为简单
 
**文件I/O** 也被称为不带缓冲的I/O(unbuffered I/O)，每一个read和write都调用内核中的一个系统调用

> **Note:** 之所以是不带缓冲的，也是相对于标准I/O而言，标准I/O库使用了缓冲技术，而这正是产生很多问题，引起许多混淆的部分，文件I/O进行了有效的规避，缓冲区由开发者自己来定义和管理

> **Tip:** **文件I/O** 并不是ISO C的组成部分，而 **标准I/O** 属于ISO C的组成部分


---

## 文件IO库的常用函数


下面是一些 **文件IO库中的常用函数**


~~~
int open( const char *pathname, int flags)
int open( const char *pathname, int flags, mode_t mode)
ssize_t read(int fd, void *buf, size_t count)
ssize_t write(int fd, const void *buf, size_t count)
off_t lseek(int fildes, off_t offset, int whence)
int close(int fd)
~~~


---

## IO库的比较


| I/O库    | 文件I/O | 标准I/O  |
| :------- | :---- | :--- |
| **缓冲方式**| 非缓冲I/O |  缓冲I/O |
| **操作对象**| 文件描述符   | 流(FILE *)  |
| **打开**   | open()   |  fopen()/freopen()/fdopen() |
|**读**|read()|fread()/fgetc()/fgets()|
|**写**|write()|fwrite()/fputc()/fputs()|
|**定位**|lseek()|fseek()/ftell()/rewind()/fsetpos()/fgetpos()|
|**关闭**|close()|fclose()|


---

## 代码示例

使用主函数传参的方式，实现图片的拷贝

`main(int argc,char *argv[])`


~~~
#./mycopy  a.jpg  b.jpg
# diff a.jpg b.jpg
~~~

代码示例

~~~
#include <stdio.h>  //标准IO函数
#include <unistd.h> //文件IO函数包含其中,缺少这个头文件read,write,close 会报错
#include <fcntl.h> //open函数包含其中,还有一些重要的宏定义


int main(int argc,char *argv[]) //带参数的主函数
{
  int fr=0,fw=0,rres=0,res=-1; 
  char tmpc='\0';
  char *fileA="/home/emacs/file/a.png";
  char *fileB="/home/emacs/file/b.png"; //定义与初始化各种变量
  
  if(3 != argc)  //进行参数检查，不符合则提示并返回
  {
    printf("argument number error: need only two args <fileA> and <fileB>\n");
    return res;
  }
  fileA=argv[1]; //将第一个参数作为A文件
  fileB=argv[2]; //将第二个参数作为B文件
  
  if(-1 == (fr=open(fileA,O_RDONLY))) //以只读的方式打开A文件
  {
    printf("cannot open file:%s\n",fileA);
    return res;
  }
  if(-1 == (fw=open(fileB,O_RDWR|O_CREAT|O_TRUNC,0600))) //以读写的方式打开B文件
  {
    printf("cannot open file:%s\n",fileB);
    return res;
  }
  while( 1 == (rres=read(fr,&tmpc,sizeof(char)))) //循环读出A文件中的内容，一次读取一个字符的长度(这个长度可以适当加长以减少读取次数来提升读取效率)
  {
    if (1 != write(fw,&tmpc,sizeof(char))) //将读出的内容依次写到文件B中
    {
      printf("write error on:%s\n",fileB);
      break;
    }
  }
  if(0 == rres) //如果返回结果是0,就代表读取完毕正常结束
  {
    printf("copy done:from %s to %s\n",fileA,fileB);
    res=0;
  }
  if(-1 == rres) //如果返回结果是-1,就代表读取出错
  {
    printf("read error on %s\n",fileA);
  }
  close(fr);
  close(fw); //回收文件描述符，刷新到硬盘
  return res;
}
~~~

> **Note:** 文件打开数是一种系统资源，是有上限的，虽然程序退出后，系统会帮忙清理，但在程序设计中，打开文件，使用完后进行手动关闭是一种很好的习惯，这样可以有效避免缓存未刷新的潜在隐患，也可以更加节约资源


### 编译执行

~~~
emacs@ubuntu:~/c$ alias gtc 
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc mycopy.x mycopy.c
emacs@ubuntu:~/c$ du -sh /home/emacs/file/a.png 
88K	/home/emacs/file/a.png
emacs@ubuntu:~/c$ du -sh /home/emacs/file/b.png 
du: 无法访问"/home/emacs/file/b.png": 没有那个文件或目录
emacs@ubuntu:~/c$ ./mycopy.x /home/emacs/file/a.png  /home/emacs/file/b.png 
copy done:from /home/emacs/file/a.png to /home/emacs/file/b.png
emacs@ubuntu:~/c$ diff /home/emacs/file/a.png  /home/emacs/file/b.png
emacs@ubuntu:~/c$
~~~

编译执行过程中没有报错，从结果来看，b.png 文件中的内容变化也符合预期


---

# 总结

以下这些函数可以应对绝大部分的IO需求

* open
* close
* read
* write
* lseek


通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础


[c_stdio_01]:/2016/12/26/c-stdio-01/
[linux_c_api]:http://www.kancloud.cn/wizardforcel/linux-c-api-ref/98469
