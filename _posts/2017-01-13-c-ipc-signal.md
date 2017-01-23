---
layout:  post
title: 进程间通讯(三).signal
author:  wilmosfang
tags:   c 
categories:  c
wc: 271  706 7459 
excerpt:  进程间通讯、signal、kill、pause
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


---

# 概要

* TOC
{:toc}

---

## signal

信号是软件中断，它提供了一种处理异步事件的方法

在 **`signal.h`** 中有关于 **`signal()`** 函数的原型声明

~~~
/* Set the handler for the signal SIG to HANDLER, returning the old
   handler, or SIG_ERR on error.
   By default `signal' has the BSD semantic.  */
__BEGIN_NAMESPACE_STD
#ifdef __USE_BSD
extern __sighandler_t signal (int __sig, __sighandler_t __handler)
     __THROW;
#else
~~~

第一个参数其实是一个整数

第二个参数是一个函数地址，并且不能带参数，第一个整型信号会被当作参数传给第二个函数

这个函数的返回值也是一个函数地址，其实就是第二个函数的地址

**`void ( *signal( int sig, void (* handler)( int )))( int );`**  

这个定义的确看起来有点晕


---

## 预定义信号

系统中也有一些预定义的信号

~~~
emacs@ubuntu:/usr/include$ kill -l 
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	
emacs@ubuntu:/usr/include$
~~~

详细的定义在 **`asm/signal.h、asm-generic/signal.h、bits/signum.h`** 中都有描述

~~~
emacs@ubuntu:/usr/include$ grep SIGINT *   -r 
asm/signal.h:#define SIGINT		 2
asm-generic/signal.h:#define SIGINT		 2
bits/signum.h:#define	SIGINT		2	/* Interrupt (ANSI).  */
linux/reboot.h: * CAD_OFF     Ctrl-Alt-Del sequence sends SIGINT to init task.
python2.6/pyconfig.h:#define HAVE_SIGINTERRUPT 1
rpcsvc/rex.x:const SIGINT = 2;	/* interrupt */
rpcsvc/rex.h:#define SIGINT 2
emacs@ubuntu:/usr/include$
~~~

信号是异步事件的经典实例，产生信号的事件对进程而言是随机出现的，进程不能简单地测试一个变量来判断是否发生了一个信号，而是必须告诉内核，在此信号发生时，请执行下列操作

在某个信号出现时，可以让内核按下列三种方式之一进行处理

* 1.忽略此信号：大多数信号都可以使用这种方式进行处理，但有两种信号决不能被忽略，它们分别是 **SIGKILL** 和 **SIGSTOP**
* 2.捕捉信号：为了做到这一点，要通知内核在某种信号发生时，调用一个用户函数
* 3.执行系统默认动作：对大多数信号的系统默认动作是终止该进程


下面通过一个例子，演示一下 signal 的使用方法

---



## 代码示例

### 要求

有A、B两个进程（父子），实现如下功能：

* 1.A进程运行开始3秒后，向B进程发送一个40号信号
* 2.B收到信号后，打印 “A，I have received your signal,now I will kill you!”
* 3.B然后向A发送SIGKILL信号使A进程退出

要求：用signal实现

提示：注意信号量的选用




### 代码示例


~~~
#include <stdio.h>
#include <signal.h> //signal，kill 函数的原型声明在里面
#include <unistd.h>

void trigger(int signum)  //定义一个触发函数，在收到信号后被调用
{
  printf("T:A,I have received your signal , now I will kill you!, the signal is %d\n",signum); //打印一句话，并且将收到的信号打印出来
}


int main()
{
  pid_t ret=0;
  int res=-1; // 初始化变量
  printf("X:befor fork\n"); 
  ret=fork(); //分裂出新进程
  printf("X:after fork:%d\n",ret);
  if(0 == ret) //A ，也就是子进程
  {
    int sig=40;
    pid_t pid=getpid(),ppid=getppid(); //获取自己的和父亲的进程号
    sleep(3); //沉睡3秒
    printf("A:this is child, my pid is %d , I am ready to sent sig %d to process which pid is %d\n",pid,sig,ppid); //打印出当前的状态
    if(0 == kill(ppid,sig)) //给父进程发送指定信号，在这里kill并不是杀死的意义，而是发送信号的意义
    {
      printf("A:sent %d signal to %d\n",sig,ppid);
    }
    else  //进行容错处理
    {
      perror("A:kill");
      return res;
    }
  }
  else if(0 < ret) //B ，也就是父进程
  {
    pid_t pid=getpid(),cpid=ret; //获取自己的和子进程的进程号
    int sig=9;
    printf("B:this is father, pid is %d, cpid is %d\n",pid,cpid); //将当前状态进行显示
    signal(40,trigger); //收到40号信号后，执行trigger函数
    pause(); //在收到信号之前，一直处于阻塞状态
    if(0 == kill(cpid,sig)) //给子进程发送信号
    {
      printf("B:sent %d signal to %d\n",sig,cpid);
    }
    else  
    {
      perror("B:kill");
    }
  }
  else //对于fork失败的容错处理
  {
    perror("X:fork\n");
    return res;
  }
  res=0;
  return res;  
}
~~~


### 编译执行


~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc signal.x signal.c
emacs@ubuntu:~/c$ ./signal.x 
X:befor fork
X:after fork:17082
B:this is father, pid is 17081, cpid is 17082
X:after fork:0
A:this is child, my pid is 17082 , I am ready to sent sig 40 to process which pid is 17081
A:sent 40 signal to 17081
T:A,I have received your signal , now I will kill you!, the signal is 40
B:sent 9 signal to 17082
emacs@ubuntu:~/c$
~~~

编译执行过程中没有报错，从结果来看，符合预期


---

## kill

在 **`signal.h`** 中有关于 kill 的原型声明

~~~
/* Send signal SIG to process number PID.  If PID is zero,
   send SIG to all processes in the current process's process group.
   If PID is < -1, send SIG to all processes in process group - PID.  */
#ifdef __USE_POSIX
extern int kill (__pid_t __pid, int __sig) __THROW;
#endif /* Use POSIX.  */
~~~

从注释中可以获取它的用法



---

## pause

在 **`unistd.h`** 中有关于 pause 的原型声明

~~~
/* Suspend the process until a signal arrives.
   This always returns -1 and sets `errno' to EINTR.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int pause (void);
~~~

直到收到一个信号，这个函数是一直阻塞的


---

# 总结

以下函数可以进行有名管道的创建

* signal
* kill
* pause


通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础
