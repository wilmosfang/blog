---
layout:  post
title: 进程间通讯(四).非亲缘进程间交互信号
author:  wilmosfang
tags:   c 
categories:  c
wc: 310 825 8079
excerpt:  进程间通讯、signal、fifo
comments: true
---


# 前言


**UNIX/Linux** 是多任务的操作系统，通过多个进程分别处理不同事务来实现，如果多个进程要进行协同工作或者争用同一个资源时，互相之间的通讯就很有必要了

进程间通信，Inter process communication，简称 IPC，在 **UNIX/Linux** 下主要有以下几种方式:

* 无名管道 ( pipe )
* 有名管道 ( fifo )
* 信号 ( signal )
* 信号量 ( semaphore )
* 消息队列 ( message queues )
* 共享内存 ( shared memory )
* 套接字 ( socket )

这里分享一下我在学习进程通讯过程中的笔记和心得

> **Tip:** 前面分别演示了 FIFO 和 signal 的功能，FIFO 可以实现非亲缘进程间的通讯，signal可以实现父子进程间发送信号，将两者结合可以实现非亲缘进程间交互信号


---

# 概要

* TOC
{:toc}

---


## 代码示例

### 要求


有A、B两个进程（无亲缘），实现如下功能：


* 1.A进程运行开始3秒后，向B进程发送一个40号信号
* 2.B收到信号后，打印“A，I have received your signal,now I will kill you!”
* 3.然后B向A发送SIGKILL信号使A进程退出。

要求：用signal做

提示：先可以用fifo互相告知对方的pid


### 代码示例


**`signalB.c`**

~~~
#include <stdio.h>
#include <signal.h> //signal,kill 等相关函数的原型声明
#include <unistd.h> //getpid,unlink,access,read,write,pause,close 等相关函数的原型声明
#include <sys/stat.h> //mkfifo 等相关函数的原型声明
#include <errno.h> //EEXIST,errno 等相关函数的原型声明
#include <fcntl.h> //open,O_RDONLY,O_WRONLY 等相关函数的原型声明和宏定义

void trigger(int signum)  //定义一个触发函数，在收到信号后被调用
{
  printf("B:A, I have received your signal , now I will kill you!, the signal is %d\n",signum);  //打印一句话，并且将收到的信号打印出来
}

int main()
{
  pid_t pid=getpid(),opid=0;
  char *rfifo="/tmp/abfifo";
  char *wfifo="/tmp/bafifo";
  int rfd=0,wfd=0,res=-1,sig=9;  //进行各种变量的初始化
  
  unlink(rfifo);
  unlink(wfifo); //删除掉现存的管道文件

  if(-1 == access(rfifo,F_OK))  //如果rfifo不存在，则创建
  {
    if(0 > (mkfifo(rfifo,0600)) && (EEXIST != errno)) //如果创建rfifo失败，并且出错不是文件已经存在，则提示并返回
    {
      printf("cannot create fifo file %s\n",rfifo);
      return res;
    }
  }
  
  if(-1 == access(wfifo,F_OK)) //同样创建wfifo
  {
    if(0 > (mkfifo(wfifo,0600)) && (EEXIST != errno))
    {
      printf("cannot create fifo file %s\n",wfifo);
      return res;
    }
  }
  
  if(-1 == (rfd = open(rfifo,O_RDONLY))) //打开rfifo
  {
    printf("cannot open fifo file:%s\n",rfifo);
    return res;
  }
 
  if(sizeof(pid_t) != read(rfd,&opid,sizeof(pid_t))) //从rfifo中读取pid_t型数值，并且存放到opid中
  {
    printf("read error on %s",rfifo);
    return res;
  }

  if(-1 == (wfd = open(wfifo,O_WRONLY)))  //打开wfifo
  {
    printf("cannot open fifo file:%s\n",wfifo);
    return res;
  }

  if(sizeof(pid_t) !=  write(wfd,&pid,sizeof(pid_t))) //将自己的pid写到wfifo中
  { 
    printf("write error on : %s\n",wfifo);
    return res;
  }

  printf("B:my pid is %d, other process pid is %d\n",pid,opid); //打印出自己的pid和从管道中获取的另一个进程的pid

  signal(40,trigger); //收到40号信号就进行trigger函数处理
  pause(); //在收到信号之前，一直处于阻塞状态

  if(0 == kill(opid,sig))  //给另一个进程发送信号
  {
    printf("B:sent %d signal to %d\n",sig,opid);
  }   
  else  
  {
    perror("B:kill");
  }
  
  
  close(wfd);
  close(rfd); //进行收尾清理
  res=0;
  return res;
}
~~~


**`signalA.c`**

~~~
#include <stdio.h>
#include <signal.h> //signal,kill 等相关函数的原型声明
#include <unistd.h> //getpid,write,read,sleep,close  等相关函数的原型声明
#include <fcntl.h> //open,O_RDONLY,O_WRONLY 等相关函数的原型声明和宏定义

int main()
{
  pid_t pid=getpid(),opid=0;
  char *rfifo="/tmp/bafifo";
  char *wfifo="/tmp/abfifo";
  int rfd=0,wfd=0,res=-1,sig=40; //进行各种变量的初始化
  
  if(-1 == (wfd = open(wfifo,O_WRONLY))) //打开wfifo
  {
    printf("cannot open fifo file:%s\n",wfifo);
    return res;
  }
  
  if(sizeof(pid_t) !=  write(wfd,&pid,sizeof(pid_t))) //将自己的pid写到wfifo中去
  { 
    printf("write error on : %s\n",wfifo);
    return res;
  }

  if(-1 == (rfd = open(rfifo,O_RDONLY))) //打开rfifo
  {
    printf("cannot open fifo file:%s\n",rfifo);
    return res;
  }
    
  if(sizeof(pid_t) != read(rfd,&opid,sizeof(pid_t))) //从rfifo中获取pid_t类型的数据并且存储到opid中
  {
    printf("read error on %s",rfifo);
    return res;
  }
  
  printf("A:my pid is %d, other process pid is %d\n",pid,opid); //将自己的和另一个进程的pid进行输出
  sleep(3); //沉睡3秒
  printf("A:I am ready to sent sig %d to process which pid is %d\n",sig,opid);
  if(0 == kill(opid,sig)) //发送指定信号给另一个进程
  {
    printf("A:sent %d signal to %d\n",sig,opid); 
  }
  else  
  {
    perror("A:kill");
    return res;
  }
  while(1) //一个死循环，用来查看被信号终结的效果
  {
    printf("A:this is A process with pid %d, opid is %d\n",pid,opid);
  }

  close(wfd);
  close(rfd); //进行收尾清理
  res=0;
  return res;
}
~~~


### 编译执行


~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc signalA.x signalA.c
emacs@ubuntu:~/c$ gtc signalB.x signalB.c
emacs@ubuntu:~/c$ 
~~~

先执行signalB.x，因为等待管道输入，所以会在终端挂起

~~~
emacs@ubuntu:~/c$ ./signalB.x 

~~~

执行signalA.x，会等3秒后立即返回

~~~
emacs@ubuntu:~/c$ ./signalA.x 
A:my pid is 19428, other process pid is 19427
A:I am ready to sent sig 40 to process which pid is 19427
A:sent 40 signal to 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
A:this is A process with pid 19428, opid is 19427
已杀死
emacs@ubuntu:~/c$
~~~

这时signalB.x 也有返回了

~~~
emacs@ubuntu:~/c$ ./signalB.x 
B:my pid is 19427, other process pid is 19428
B:A, I have received your signal , now I will kill you!, the signal is 40
B:sent 9 signal to 19428
emacs@ubuntu:~/c$ 
~~~


编译执行过程中没有报错，从结果来看，符合预期


---

## unlink

在 unistd.h 中有关于 unlink 的原型声明

~~~
/* Remove the link NAME.  */
extern int unlink (__const char *__name) __THROW __nonnull ((1));
~~~

它所起的作用就是删除文件


---

# 总结

以下函数可以进行有名管道的创建和信号的控制

* signal
* kill
* pause
* mkfifo


通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础
