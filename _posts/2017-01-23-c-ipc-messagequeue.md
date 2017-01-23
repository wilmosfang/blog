---
layout:  post
title: 进程间通讯(五).message queue
author:  wilmosfang
tags:   c 
categories:  c
wc:  605  1489 19484 
excerpt:  消息队列、参数限制、msgmni、msgmax、msgmnb、ftok、key_t、IPC_PRIVATE、msg.h 所包含的头文件、msgget、msgsnd、msgrcv、msgctl、msqid_ds、ipc_perm、fgets
comments: true
---


# 前言


**UNIX/Linux** 是多任务的操作系统，通过多个进程分别处理不同事务来实现，如果多个进程要进行协同工作或者争用同一个资源时，互相之间的通讯就很有必要了

进程间通信，**Inter process communication**，简称 **IPC**，在 **UNIX/Linux** 下主要有以下几种方式:

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

## 消息队列


系统层面的消息队列是消息的链接表，存储在内核中，由消息队列标识符标识

一个消息可以看成一个记录，具有特定的格式以及特定的类别

>对消息队列有写权限的进程可以向消息队列中按照一定的规则添加新消息；对消息队列有读权限的进程则可以从消息队列中读走消息。消息队列是随内核持续的。也就是说进程的退出，如果不自主去释放资源，消息队列是会悄无声息的存在的。所以较管道来说，消息队列的生命周期更加持久。消息队列提供了一种从一个进程向另一个进程发送一个数据块的方法。每个数据块都被认为是有一个类型，接收者进程接收的数据块可以有不同的类型值。我们可以通过发送消息 来避免命名管道的同步和阻塞问题。消息队列与管道不同的是，消息队列是基于消息的， 而管道是基于字节流的，且消息队列的读取不一定是先入先出。消息队列与命名管道有一 样的不足，就是每个消息的最大长度是有上限的（MSGMAX），每个消息队列的总的字节数是有上限的（MSGMNB），系统上消息队列的总数也有一个上限（MSGMNI）

在某个进程往一个队列写入消息之前，并不需要另外某个进程在该队列上等待消息的到达

pipe 和 FIFO 最后一次关闭发生时，仍在该管道或FIFO上的数据将被丢弃，消息队列，除非内核自举或显式删除，否则其一直存在

管道和FIFO都是随进程持续的，XSI IPC(消息队列、信号量、共享内存)都是随内核持续的

消息队列的链表结构类型于下：

~~~
/* Obsolete, used only for backwards compatibility and libc5 compiles */
struct msqid_ds {
        struct ipc_perm msg_perm;
        struct msg *msg_first;          /* first message on queue,unused  */
        struct msg *msg_last;           /* last message in queue,unused */
        __kernel_time_t msg_stime;      /* last msgsnd time */
        __kernel_time_t msg_rtime;      /* last msgrcv time */
        __kernel_time_t msg_ctime;      /* last change time */
        unsigned long  msg_lcbytes;     /* Reuse junk fields for 32 bit */
        unsigned long  msg_lqbytes;     /* ditto */
        unsigned short msg_cbytes;      /* current number of bytes on queue */
        unsigned short msg_qnum;        /* number of messages in queue */
        unsigned short msg_qbytes;      /* max number of bytes on queue */
        __kernel_ipc_pid_t msg_lspid;   /* pid of last msgsnd */
        __kernel_ipc_pid_t msg_lrpid;   /* last receive pid */
};
~~~


>与系统级别相对应的还有应用层面的MQ，(使用一样的思想进行应用解耦)，开源产品有rabbitmq ：MQ全称为Message Queue, 消息队列（MQ）是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如远程过程调用的技术。排队指的是应用程序通过 队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求


---

## 系统限制


系统层面有一些内核参数限制了消息队列的大小

~~~
root@ubuntu:~# sysctl -a 2> /dev/null  | grep msg 
kernel.msgmax = 8192
kernel.msgmni = 1730
kernel.msgmnb = 16384
kernel.auto_msgmni = 1
fs.mqueue.msg_max = 10
fs.mqueue.msgsize_max = 8192
root@ubuntu:~# vim /etc/sysctl.conf 
root@ubuntu:~# 
~~~

### msgmni 

msgmni 定义了系统范围内的消息队列上限。与信号量一样，消息队列也拥有一个相关的标识符。在系统初始化阶段里，内核创建一个指向消息队列标识符结构的指针数组。该数组的项数由 msgmni确定。对于每个消息队列，Linux 内核为标识符分配44B，为消息队列数据结构分配 96B。为了获得更多的消息队列资源，可以动态增加 msgmni 取值。和信号量一样，消息队列标识符的最大数目也受限于IPCMNI。msgmni的默认上限为 16B，这可能不足以保证一些大型数据库应用平滑地运行。如果在系统上要运行数据库应用的话，推荐默认上限值是 128B


### msgmax 

msgmax 限制进程可以发送的消息长度。该参数由 Msgsnd()函数加以应用。如果待发送消息的长度超过该值，则返回一个错误。该参数可以在运行时调整


---

## msgmnb


msgmnb 确定一个消息队列的容量。该参数的取值存储在消息队列标识符结构的某个域中，用于确定是否存在着对新消息进行排队的空间。msgmnb 值可以动态修改，默认为16384。修改其取值会影响到所有新的消息队列的容量。用户可以通过 Msgctl()系统调用来增加现有消息队列的容量


> **Tip:** **`/etc/sysctl.conf`** 中可以进行内核配置

---


## 代码示例

### 要求

* 1.A、B两个进程（非亲缘关系），A进程往消息队列中写入任意字符串（以START开头则为有效的），B进程读取
* 2.直到收到“quit”才退出




### 代码示例


**`msgqueA.c`**

~~~
#include <stdio.h>
#include <sys/msg.h> //key_t,ftok,msgget,msgsnd,IPC_CREAT 等相关声明都在这里面定义
#include <unistd.h> //getpid 的函数声明在这个头文件里
#include <string.h> //strncmp,strcmp,strlen 的函数声明在这个头文件里
#define BUFSZ 1024


typedef struct message //此结构体用于存放消息，从中可以看到消息的两个字段
{
  long msg_type; //消息类型，以整型值进行标示
  char msg_text[BUFSZ]; //消息内容
}MSG; //取了一个别名

int main() 
{
  int res=-1,qid=0;
  key_t key=IPC_PRIVATE; //IPC_PRIVATE 就是0  
  int len=0; //赋初值是一个好习惯
  MSG msg;
  
  if(-1 == (key=ftok("/",18))) // 通过ftok获取一个key，两个进程可以通过这个key来获取队列信息
  {
    perror("ftok");
    return res;
  }

  if(-1==(qid=msgget(key,IPC_CREAT|0600))) //创建一个消息队列，将id存到qid中
  {
    perror("msgget");
    return res;
  }
  
  printf("open queue %d\n",qid); //将qid进行显示
  
  while(1)
  {
    puts("please enter the message to queue:\n(message start with 'START' will be valid,'quit' to exit)"); 
    if(NULL == (fgets(msg.msg_text,BUFSZ,stdin))) //从标准输入中获取信息放到 msg.msg_text 中
    {
      perror("fgets");
      return res;
    }
    msg.msg_type=getpid(); //将消息的类型设置为本进程的ID
    if( 0== strncmp(msg.msg_text,"START",5) || 0== strcmp(msg.msg_text,"quit\n")) //如果内容是以 START 开头，并且不是 quit，就将这条信息发送
    {
      len=strlen(msg.msg_text);
      if (0 > msgsnd(qid,&msg,len,0)) //发送信息
      {
	perror("msgsnd");
	return res;
      }
    }
    if ( 0 == strcmp(msg.msg_text,"quit\n") ) break; //如果是quit 就进行退出
  }    
  res=0;
  return res;
}
~~~


**`msgqueB.c`**

~~~
#include <stdio.h>
#include <sys/msg.h> //key_t,ftok,msgget,msgrcv,msgctl,IPC_RMID 相关声明在这个头文件中有所包含
#include <string.h> //memset,strcmp 相关函数声明在这个头文件中有所包含
#define BUFSZ 1024


typedef struct message  //此结构体用于存放消息，从中可以看到消息的两个字段
{
  long msg_type; //消息类型，以整型值进行标示
  char msg_text[BUFSZ]; //消息内容体
}MSG; //取了一个别名

int main()
{
  int res=-1,qid=0;
  key_t key=IPC_PRIVATE;  //IPC_PRIVATE 就是0
  MSG msg;
  
  if(-1 == (key=ftok("/",18))) //调用ftok使用相同的参数生成key，用于获取一样的队列ID
  {
    perror("ftok");
    return res;
  }
  if(-1==(qid=msgget(key,IPC_CREAT|0600))) //创建一个消息队列，将id存到qid中(如果已经存在，则获取它的ID)
  {
    perror("msgget");
    return res;
  }
  printf("open queue %d\n",qid);
  
  do
  {
    memset(msg.msg_text,0,BUFSZ); //将msg.msg_text的内容清零
    if(0 > msgrcv(qid,&msg,BUFSZ,0,0)) //从消息队列中获取信息并且存到msg中
    {
      perror("msgrcv");
      return res;
    } 
    printf("the message from process %ld is %s",msg.msg_type,msg.msg_text); //将信息内容在终端进行打印
  }while(strcmp(msg.msg_text,"quit\n")); //如果内容为quit就进行跳出
  

   if( 0 > msgctl(qid,IPC_RMID,NULL) ) //将队列删除，如果不删除，在进程退出后，消息将依旧保留在内核中，直到重启系统，消息的持久性，界于进程与磁盘之间
   {
     perror("msgctl");
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
emacs@ubuntu:~/c$ gtc  msgqueA.x msgqueA.c ; gtc msgqueB.x msgqueB.c
emacs@ubuntu:~/c$ 
~~~

执行 **`msgqueB.x`** 会等待输入

~~~
emacs@ubuntu:~/c$ ./msgqueB.x 
open queue 98304

~~~

执行 **`msgqueA.x`** 会等待输入

~~~
emacs@ubuntu:~/c$ ./msgqueA.x 
open queue 98304
please enter the message to queue:
(message start with 'START' will be valid,'quit' to exit)
~~~
 
在 **`msgqueA.x`** 端输入一些内容

~~~
emacs@ubuntu:~/c$ ./msgqueA.x 
open queue 98304
please enter the message to queue:
(message start with 'START' will be valid,'quit' to exit)
STARTtest
please enter the message to queue:
(message start with 'START' will be valid,'quit' to exit)
123
please enter the message to queue:
(message start with 'START' will be valid,'quit' to exit)
START123
please enter the message to queue:
(message start with 'START' will be valid,'quit' to exit)
quit
emacs@ubuntu:~/c$
~~~


在 **`msgqueB.x`** 端会进行显示

~~~
emacs@ubuntu:~/c$ ./msgqueB.x 
open queue 98304
the message from process 23670 is STARTtest
the message from process 23670 is START123
the message from process 23670 is quit
emacs@ubuntu:~/c$ 
~~~

编译执行过程中没有报错，从结果来看，符合预期



---

## ftok

此函数的原型在 **`sys/ipc.h`** 中

~~~
/* Generates key for System V style IPC.  */
extern key_t ftok (__const char *__pathname, int __proj_id) __THROW;
~~~

>The ftok function uses the identity of the  file  named  by  the  given pathname  (which  must  refer  to an existing, accessible file) and the least significant 8 bits of proj_id (which must be nonzero) to generate  a  key_t  type  System  V  IPC  key

此函数把从pathname导出的信息与id的低序8位组合成一个整数IPC键


---

## key_t



~~~
emacs@ubuntu:/usr/include$ grep key_t sys/ipc.h
#ifndef __key_t_defined
typedef __key_t key_t;
# define __key_t_defined
extern key_t ftok (__const char *__pathname, int __proj_id) __THROW;
emacs@ubuntu:/usr/include$ grep key_t bits/types.h
__STD_TYPE __KEY_T_TYPE __key_t;	/* Type of an IPC key.  */
emacs@ubuntu:/usr/include$ grep __KEY_T_TYPE  bits/typesizes.h
#define __KEY_T_TYPE		__S32_TYPE
emacs@ubuntu:/usr/include$ grep __S32_TYPE bits/types.h
#define	__S32_TYPE		int
emacs@ubuntu:/usr/include$
~~~

可知 **`key_t`** 其实就是 **`int`**

**`key_t <= __KEY_T_TYPE <= __S32_TYPE <= int`**


---

## IPC_PRIVATE

**`bits/ipc.h`** 中有关于 IPC_X 的宏定义

~~~
/* Mode bits for `msgget', `semget', and `shmget'.  */
#define IPC_CREAT       01000           /* Create key if key does not exist. */
#define IPC_EXCL        02000           /* Fail if key exists.  */
#define IPC_NOWAIT      04000           /* Return error on wait.  */

/* Control commands for `msgctl', `semctl', and `shmctl'.  */
#define IPC_RMID        0               /* Remove identifier.  */
#define IPC_SET         1               /* Set `ipc_perm' options.  */
#define IPC_STAT        2               /* Get `ipc_perm' options.  */
#ifdef __USE_GNU
# define IPC_INFO       3               /* See ipcs.  */
#endif

/* Special key values.  */
#define IPC_PRIVATE     ((__key_t) 0)   /* Private key.  */
~~~

所以 **`IPC_PRIVATE`** 其实代表的是 0 


---

## msg.h 所包含的头文件

~~~
emacs@ubuntu:/usr/include$ grep include sys/msg.h 
#include <features.h>
#include <stddef.h>
#include <sys/ipc.h>
#include <bits/msq.h>
#include <time.h>
emacs@ubuntu:/usr/include$ grep include sys/ipc.h 
#include <features.h>
#include <bits/ipctypes.h>
#include <bits/ipc.h>
emacs@ubuntu:/usr/include$
~~~

---

## msgget

**`msgget`**  的原型定义在 **`sys/msg.h`** 中

~~~
/* Get messages queue.  */
extern int msgget (key_t __key, int __msgflg) __THROW;
~~~

**`__msgflg`**  是读写权限的组合，相关的宏在 **`bits/ipc.h`** 中有所定义

返回值为一个整型，即消息队列的ID

---

## msgsnd

**`msgsnd`**  的原型定义在 **`sys/msg.h`** 中

~~~
/* Send message to message queue.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int msgsnd (int __msqid, __const void *__msgp, size_t __msgsz,
                   int __msgflg);
~~~

**`__msqid`** 消息队列的ID

**`__msgp`** 消息结构体的指针

**`__msgsz`** 消息内容的长度

**`__msgflg`**  控制函数行为的标志 , 一般取0 , 表示忽略

---

## msgrcv 

**`msgrcv`**  的原型定义在 **`sys/msg.h`** 中

~~~
/* Receive message from message queue.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern ssize_t msgrcv (int __msqid, void *__msgp, size_t __msgsz,
                       long int __msgtyp, int __msgflg);
~~~

**`__msqid`** 消息队列的ID

**`__msgp`** 消息结构体的指针

**`__msgsz`** 消息内容的长度

**`__msgtyp`** 消息类型，可以实现一种简单的接收优先级

>如果 msgtype == 0，就获取队列中的第一个消息
>
>如果 msgtype > 0，将获取具有相同消息类型的第一个信息
>
>如果 msgtype < 0，就获取类型等于或小于msgtype的绝对值的第一个消息

**`__msgflg`**  

>这个参数依然是是控制函数行为的标志，取值可以是：0,表示忽略；IPC_NOWAIT，如果消息队列为空，则返回一个ENOMSG，并将控制权交回调用函数的进程。如果不指定这个参数，那么进程将被阻塞直到函数可以从队列中得到符合条件的消息为止。如果一个client 正在等待消息的时候队列被删除，EIDRM 就会被返回。如果进程在阻塞等待过程中收到了系统的中断信号，EINTR 就会被返回。MSG_NOERROR，如果函数取得的消息长度大于msgsz，将只返回msgsz 长度的信息，剩下的部分被丢弃了。如果不指定这个参数，E2BIG 将被返回，而消息则留在队列中不被取出。当消息从队列内取出后，相应的消息就从队列中删除了。



函数调用成功时，该函数返回放到接收缓存区中的字节数，消息被复制到由msgp指向的用户分配的缓存区中，然后删除消息队列中的对应消息; 失败时返回-1


---

## msgctl 

**`msgctl`**  的原型定义在 **`sys/msg.h`** 中
 
~~~
/* Message queue control operation.  */
extern int msgctl (int __msqid, int __cmd, struct msqid_ds *__buf) __THROW;
~~~

**`__msqid`** 消息队列的ID

**`__cmd`** 将要采取的动作

**`bits/ipc.h`** 中有关于 IPC_X 的宏定义

~~~
#define IPC_RMID        0               /* Remove identifier.  */ //删除消息队列
#define IPC_SET         1               /* Set `ipc_perm' options.  */ // 如果进程有足够的权限，就把消息列队的当前关联值设置为msgid_ds结构中给出的值 
#define IPC_STAT        2               /* Get `ipc_perm' options.  */ //把msgid_ds结构中的数据设置为消息队列的当前关联值，即用消息队列的当前关联值覆盖msgid_ds的值
~~~


**`__buf`**  msqid_ds 结构体指针


> 对删除消息队列的处理不是很完善，因为每个消息队列没有维护引用计数（打开文件有这种计数器），所以在队列被删除以后，仍在使用这一队列的进程在下次对队列进行操作时会出错返回


函数成功时返回0，失败时返回-1


---

## msqid_ds

在 **`bits/msq.h`** 中有关于 **`msqid_ds`** 的定义

~~~
/* Structure of record for one message inside the kernel.
   The type `struct msg' is opaque.  */
struct msqid_ds
{
  struct ipc_perm msg_perm;	/* structure describing operation permission */
  __time_t msg_stime;		/* time of last msgsnd command */
#if __WORDSIZE == 32
  unsigned long int __unused1;
#endif
  __time_t msg_rtime;		/* time of last msgrcv command */
#if __WORDSIZE == 32
  unsigned long int __unused2;
#endif
  __time_t msg_ctime;		/* time of last change */
#if __WORDSIZE == 32
  unsigned long int __unused3;
#endif
  unsigned long int __msg_cbytes; /* current number of bytes on queue */
  msgqnum_t msg_qnum;		/* number of messages currently on queue */
  msglen_t msg_qbytes;		/* max number of bytes allowed on queue */
  __pid_t msg_lspid;		/* pid of last msgsnd() */
  __pid_t msg_lrpid;		/* pid of last msgrcv() */
  unsigned long int __unused4;
  unsigned long int __unused5;
};
~~~

---

## ipc_perm

**`bits/ipc.h`** 中有关于 **`ipc_perm`** 的定义

~~~
/* Data structure used to pass permission information to IPC operations.  */
struct ipc_perm
  {
    __key_t __key;			/* Key.  */
    __uid_t uid;			/* Owner's user ID.  */
    __gid_t gid;			/* Owner's group ID.  */
    __uid_t cuid;			/* Creator's user ID.  */
    __gid_t cgid;			/* Creator's group ID.  */
    unsigned short int mode;		/* Read/write permission.  */
    unsigned short int __pad1;
    unsigned short int __seq;		/* Sequence number.  */
    unsigned short int __pad2;
    unsigned long int __unused1;
    unsigned long int __unused2;
  };
~~~


> **Tip:** 消息队列原来的实施目的是提供高于一般速度的IPC，但现在与其它形式的IPC相比，在速度方面已经没有什么差别了，考虑到使用消息队列可能带来的问题，在新的应用程序中不应当再使用它们

---

## fgets

**`stdio.h`** 中有关于 **`fgets`** 的原型声明

~~~
/* Get a newline-terminated string of finite length from STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern char *fgets (char *__restrict __s, int __n, FILE *__restrict __stream)
     __wur;
~~~



---

# 总结

以下函数可以进行有名管道的创建和信号的控制

* ftok
* msgget
* msgsnd
* msgrcv
* msgctl


通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础


