---
layout:  post
title: 进程间通讯(六).semaphore and shared memory
author:  wilmosfang
tags:   c 
categories:  c
wc:    912  1847 26729 
excerpt:  信号量、共享内存、信号量限制、共享内存限制、sembuf、SEM_UNDO、shmget、shmat、semget、semop、shmdt、semctl、sembuf 共用体、semid_ds 结构体、seminfo 结构体、shmctl、IPC_X 宏定义、shmid_ds 结构体、ipc_perm 结构体
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

## 信号量

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问代码的临界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它，也就是说信号量是用来调协进程对共享资源的访问的。

信号量是一个特殊的变量，程序对其访问都是原子操作，且只允许对它进行等待（即P(信号变量))和发送（即V(信号变量))信息操作。最简单的信号量是只能取0和1的变量，这也是信号量最常见的一种形式，叫做二进制信号量。而可以取多个正整数的信号量被称为通用信号量。这里主要讨论二进制信号量。

> **Tip:** 引自 **[《Linux进程间通信——使用信号量》][10243617]**

信号量与已经介绍过的IPC机构（管道、FIFO以及消息列队）不同，它是一个计数器，用于为多个进程提供对共享数据对象的访问

为了获得共享资源，进程需要执行下列操作

* 1.测试控制该资源的信号量
* 2.若此信号量的值为正，则进程可以使用该资源，在这种情况下，进程会将信号量值减1，表示它使用了一个资源单位
* 3.否则，若此信号量的值为0，则进程进入休眠状态，直至信号量大于0，进程被唤醒后它返回至步骤1

当进程不再使用由一个信号量控制的共享资源时，该信号量值增1，如果有进程正在休眠等待此信号量，则唤醒它们

为了正确地实现信号量，信号量值的测试及减1操作应该是原子操作，为此信号量通常是在内核中实现的

常用的信号量形式被称为 **二元信号量（binary semaphore）** ，它控制单个资源，其初始值为1，但是一般而言，信号量的初值可以是任意一个正值，该值表明有多少个共享资源单位可供共享应用。

> **Tip:** 引自 **《UNIX环境高级编程》**


---

## 共享内存

顾名思义，共享内存就是允许两个不相关的进程访问同一个逻辑内存。共享内存是在两个正在运行的进程之间共享和传递数据的一种非常有效的方式。不同进程之间共享的内存通常安排为同一段物理内存。进程可以将同一段共享内存连接到它们自己的地址空间中，所有进程都可以访问共享内存中的地址，就好像它们是由用C语言函数malloc分配的内存一样。而如果某个进程向共享内存写入数据，所做的改动将立即影响到可以访问同一段共享内存的任何其他进程。

特别提醒：共享内存并未提供同步机制，也就是说，在第一个进程结束对共享内存的写操作之前，并无自动机制可以阻止第二个进程开始对它进行读取。所以我们通常需要用其他的机制来同步对共享内存的访问，例如前面说到的信号量。

> **Tip:** 引自 **[《Linux进程间通信——使用共享内存》][10253345]**

共享存储允许两个或多个进程共享一个给定的存储区，因为数据不需要在客户进程和服务进程之间复制，所以这是最快的一种IPC，使用共享存储时要掌握的唯一窍门是，在多个进程之间同步访问一个给定的存储区时，若服务器进程正在将数据放入共享存储区，则它在做完这一操作之前，客户进程不应当去取这些数据，通常，信号量用于同步共享存储访问(也可以用记录锁或互斥量)

> **Tip:** 引自 **《UNIX环境高级编程》**

---

## 系统限制

### 信号量限制

系统层面有一些内核参数限制了信号量的大小

~~~
root@ubuntu:~# ipcs -ls

------ Semaphore Limits --------
max number of arrays = 128
max semaphores per array = 250
max semaphores system wide = 32000
max ops per semop call = 32
semaphore max value = 32767

root@ubuntu:~# sysctl -a 2>/dev/null | grep sem
kernel.sem = 250	32000	32	128
root@ubuntu:~#
~~~

表示系统信号量集的最大个数为128个

一个信号量集中最大可以有250个信号量

系统中最多总共有32000个信号量

semop系统调用允许的信号量最大个数为32个


### 共享内存限制

系统层面有一些内核参数限制了共享内存的大小

~~~
root@ubuntu:~# ipcs -lm

------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 32768
max total shared memory (kbytes) = 8388608
min seg size (bytes) = 1

root@ubuntu:~# sysctl -a 2>/dev/null | grep shm
kernel.shmmax = 33554432
kernel.shmall = 2097152
kernel.shmmni = 4096
vm.hugetlb_shm_group = 0
root@ubuntu:~#
~~~

* SHMMAX参数定义共享内存段的最大尺寸（以字节为单位），默认值是32MB。
* SHMALL参数控制着系统一次可以使用的共享内存总量（以页为单位），默认值2097152.该参数值至少应该大于等于SHMMAX/PAGE_SIZE。
* SHMMNI 参数设置系统范围内共享内存段的最大数量，默认值是 4096。

> **Tip:** **`/etc/sysctl.conf`** 中可以进行内核配置

---


## 代码示例

### 要求

* A、B两个进程（非亲缘关系）
* A进程往共享内存中写入任意字符串，B进程读取
* 直到收到“quit”才退出
* 写一次读一次，读一次写一次

注意：

* 1）shmid要保持一致  `IPC_CREAT|0666`
* 2）不能用sleep函数进行同步

提示：

* 1）用信号量配合2个




### 代码示例


**`shmsemA.c`**

~~~
#include <stdio.h>
#include <sys/shm.h> //shmget,shmat,shmdt,shmctl 相关声明都在这里
#include <sys/sem.h> //sembuf,SEM_UNDO,SETALL 相关声明和宏定义都在这里
#include <string.h>  //memset,strcmp  相关函数声明都在这里

#define SHMSIZE 1024

typedef struct sembuf SB; // 将sembuf重命名

union semun  //定义semun共用体作为参数 
{ 
  int val;		 // value for SETVAL  设定一个值可以用
  struct semid_ds *buf;	 // buffer for IPC_STAT & IPC_SET  获取状态可以使用
  unsigned short *array; // array for GETALL & SETALL  设定所有值或获取所有值可以使用
  struct seminfo *__buf; // buffer for IPC_INFO 获取信息可以用
  void *__pad; //万能指针
};

int main() 
{
  int res=-1,shmid=0,semid=0;
  key_t  key=IPC_PRIVATE;
  char *shmaddr=NULL; //定义一个共享内存的指针
  SB sem_p0={0,-1,SEM_UNDO}, //构建对第一个信号量进行P操作的参数
     sem_v0={0,1,SEM_UNDO}, //构建对第一个信号量进行V操作的参数
     sem_v1={1,1,SEM_UNDO}; //构建对第二个信号量进行V操作的参数
  union semun sem_args;  
  unsigned short array[2]={0,0}; //构建数组

  sem_args.array=array; //构建出给两个信号量一起赋值的共用体参数
  

  if(-1 == (key=ftok("/",888))) //生成key
  {
    perror("ftok");
    return res;
  }
  if (0 > (shmid=shmget(key,SHMSIZE,IPC_CREAT|0600))) //使用shmget 创建或获取共享内存ID , 大小为1024个字节 ， or use getpagesize() instead 
  {
    perror("shmget");
    return res;
  }
  else printf("created shared memory :%d\n",shmid); //显示出获取的共享内存ID
  
  if ((char *)0 > (shmaddr=shmat(shmid,0,0))) //通过ID获取地址，并且使用shmaddr进行指向
  {
    perror("shmat");
    return res;
  }
  else printf("attached shared memory:%p\n",shmaddr); //将内存地址打印出来
  
  if (0 > (semid=semget(key,2,IPC_CREAT|0600))) //通过key获取两个信号量的ID
  {
    perror("semget");
    return res;
  }
  else printf("created a sem set with two sems which id is :%d\n",semid); //将信号量ID打印出来

  if (0 > semctl(semid,0,SETALL,sem_args)) //将两个信号量一起赋值为0，设置值存于sem_args中
  {
    perror("semctl");
    return res;
  } 
  else printf("semset has been initialized\n");
  
  if (0 > semop(semid,&sem_v0,1)) //对第一个信号量进行V操作
  {
    perror("semop"); 
    return res;
  }

  memset(shmaddr,0,SHMSIZE); //将共享内存置0

  do
  {
    if (0 > semop(semid,&sem_p0,1)) //对第一个信号量进行P操作
    {
      perror("semop"); 
      return res;
    }
    puts("please enter the message to shm:\n('quit' to exit)");
    if(NULL == (fgets(shmaddr,SHMSIZE,stdin))) //将输入内容写到共享内存中
    {
      perror("fgets");
      return res;
    }
    if (0 > semop(semid,&sem_v1,1)) //对第二个信号量进行V操作
    {
      perror("semop"); 
      return res;
    }
  }while(strcmp(shmaddr,"quit\n"));   //如果输入为quit就退出

  if (0 > (shmdt(shmaddr))) //分离共享内存
  {
    perror("shmdt");
    return res;
  }
  else   printf("Deattach shared-memory\n"); 
  
  if (0 > semop(semid,&sem_p0,1)) //对第一个信号量进行P操作
  {
    perror("semop"); 
    return res;
  }

  if (0 > shmctl(shmid, IPC_RMID, NULL)) //删除和释放共享内存
  {
    perror("shmctl(IPC_RMID)\n");
    return res;
  }
  else printf("Delete shared-memory\n"); 
  res=0;
  return res;
}
~~~


**`shmsemB.c`**

~~~
#include <stdio.h>
#include <sys/shm.h>  //shmget,shmat,shmdt 相关声明都在这里
#include <sys/sem.h>  //sembuf, SEM_UNDO,semget,semop 相关声明和宏定义都在这里
#include <string.h>

#define SHMSIZE 1024

typedef struct sembuf SB;  //将sembuf重命名

int main()
{
  int res=-1,shmid=0,semid=0;
  key_t  key=IPC_PRIVATE;
  char *shmaddr=NULL;
  SB sem_v0={0,1,SEM_UNDO},  //构建对第一个信号量进行V操作的参数
     sem_p1={1,-1,SEM_UNDO}; //构建对第二个信号量进行P操作的参数

  if(-1 == (key=ftok("/",888))) //生成key
  {
    perror("ftok");
    return res;
  }
  if (0 > (shmid=shmget(key,SHMSIZE,IPC_CREAT|0600))) //使用key创建或获取共享内存ID,大小为1024字节 or use getpagesize() instead
  {
    perror("shmget");
    return res;
  }
  else printf("created shared memory :%d\n",shmid); //将共享内存的ID进行显示
  
  if ((char *)0 > (shmaddr=shmat(shmid,0,0))) //通过共享内存的ID获取内存地址
  {
    perror("shmat");
    return res;
  }
  else printf("attached shared memory:%p\n",shmaddr); //将共享内存的地址进行打印
  
  if (0 > (semid=semget(key,2,IPC_CREAT|0600))) //通过key创建两个信号量的ID
  {
    perror("semget");
    return res;
  }
  else printf("created a sem set with two sems which id is :%d\n",semid);

  do
  {
    if (0 > semop(semid,&sem_p1,1))  //对第二个信号量进行P操作
    {
      perror("semop"); 
      return res;
    }
    printf("from shm :%s",shmaddr); //将共享内存中的内容进行打印
    
    if(0 == strcmp(shmaddr,"quit\n"))break; //如果内容为 quit，则直接退出循环
    if (0 > semop(semid,&sem_v0,1)) //对第一个信号量进行V操作
    {
      perror("semop"); 
      return res;
    }
  }while(1);

  if (0 > (shmdt(shmaddr))) //分享共享内存
  {
    perror("shmdt");
    return res;
  }
  else   printf("Deattach shared-memory\n");
  
  if (0 > semop(semid,&sem_v0,1)) //对第一个信号量进行V操作
  {
    perror("semop"); 
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
emacs@ubuntu:~/c$ gtc shmsemA.x  shmsemA.c ; gtc shmsemB.x shmsemB.c 
emacs@ubuntu:~/c$
~~~

执行 **`shmsemB.x`** 会等待输入

~~~
emacs@ubuntu:~/c$ ./shmsemB.x 
created shared memory :1081363
attached shared memory:0xb77e1000
created a sem set with two sems which id is :0


~~~

执行 **`shmsemA.x`** 会等待输入

~~~
emacs@ubuntu:~/c$ ./shmsemA.x 
created shared memory :1081363
attached shared memory:0xb78b8000
created a sem set with two sems which id is :0
semset has been initialized
please enter the message to shm:
('quit' to exit)

~~~
 
在 **`shmsemA.x`** 端输入一些内容

~~~
emacs@ubuntu:~/c$ ./shmsemA.x 
created shared memory :1081363
attached shared memory:0xb78b8000
created a sem set with two sems which id is :0
semset has been initialized
please enter the message to shm:
('quit' to exit)
abc
please enter the message to shm:
('quit' to exit)
123
please enter the message to shm:
('quit' to exit)
456
please enter the message to shm:
('quit' to exit)
quit
Deattach shared-memory
Delete shared-memory
emacs@ubuntu:~/c$

~~~


在 **`shmsemB.x`** 端会进行显示

~~~
emacs@ubuntu:~/c$ ./shmsemB.x 
created shared memory :1081363
attached shared memory:0xb77e1000
created a sem set with two sems which id is :0
from shm :abc
from shm :123
from shm :456
from shm :quit
Deattach shared-memory
emacs@ubuntu:~/c$
~~~

编译执行过程中没有报错，从结果来看，符合预期


---

## sembuf

**`sys/sem.h`** 中有关于sembuf结构体的定义

~~~
/* Structure used for argument to `semop' to describe operations.  */
struct sembuf
{
  unsigned short int sem_num;   /* semaphore number */
  short int sem_op;             /* semaphore operation */
  short int sem_flg;            /* operation flag */
};
~~~

这个结构体定义的其实是 **`semop`**  的第二个参数 

这个结构体中三个成员变量分别表示 :

* **`sem_num`**  信号在信号集中的索引，0代表第一个信号，1代表第二个信号
* **`sem_op`**  操作类型，代表P操作或是V操作
* **`sem_flg`**  该参数可设置为 `IPC_NOWAIT` 或 `SEM_UNDO` 两种状态，`IPC_NOWAIT` 设置信号量操作不等待，一般使用 `SEM_UNDO` ， 代表进程终止或崩溃后，这个操作会自动撤销，恢复到未操作之前的样子，是为了避免进程异常导致的死锁或系统资源耗尽 

---

## SEM_UNDO

**`bits/sem.h`** 中有关于 `SEM_UNDO` 宏的定义

~~~
/* Flags for `semop'.  */
#define SEM_UNDO        0x1000          /* undo the operation on exit */
~~~

`SEM_UNDO` 选项会让内核记录一个与调用进程相关的UNDO记录，如果该进程崩溃，则根据这个进程的UNDO记录自动恢复相应信号量的计数值

---

## shmget 

**`sys/shm.h`** 中有关于 `shmget` 的原型声明


~~~
/* Get shared memory segment.  */
extern int shmget (key_t __key, size_t __size, int __shmflg) __THROW;
~~~

得到一个共享内存标识符或创建一个共享内存对象并返回共享内存标识符

**`__key`** 由ftok生成的key

**`__size`** 共享内存的大小

**`__shmflg`** 当 `shmflg&IPC_CREAT` 为真时，如果内核中不存在键值与key相等的共享内存，则新建一个共享内存；如果存在这样的共享内存，返回此共享内存的标识符；`IPC_CREAT|IPC_EXCL`：如果内核中不存在键值与key相等的共享内存，则新建一个消息队列；如果存在这样的共享内存则报错（shmflg参数为模式标志参数，使用时需要与IPC对象存取权限（如0600）进行或运算(\|)来确定信号量集的存取权限）


函数成功则返回共享内存的标识符，出错则返回-1，错误原因存于error中

有以下几种错误

~~~
EINVAL：参数size小于SHMMIN或大于SHMMAX

EEXIST：预建立key所指的共享内存，但已经存在

EIDRM：参数key所指的共享内存已经删除

ENOSPC：超过了系统允许建立的共享内存的最大值(SHMALL)

ENOENT：参数key所指的共享内存不存在，而参数shmflg未设IPC_CREAT位

EACCES：没有权限

ENOMEM：核心内存不足
~~~

---

## shmat

**`sys/shm.h`** 中有关于 `shmat` 的原型声明

~~~
/* Attach shared memory segment.  */
extern void *shmat (int __shmid, __const void *__shmaddr, int __shmflg)
     __THROW;
~~~

连接共享内存标识符为shmid的共享内存，连接成功后把共享内存区对象映射到调用进程的地址空间，随后可像本地空间一样访问

**`__shmid`** 共享内存标识符

**`__shmaddr`**  指定共享内存出现在进程内存地址的什么位置，直接指定为NULL让内核自己决定一个合适的地址位置

**`__shmflg`**  SHM_RDONLY：为只读模式，其他为读写模式

如果成功则返回附加好的共享内存地址，如果出错，则返回-1，错误原因存于error中


> **Tip:**  fork后子进程继承已连接的共享内存地址；exec后该子进程与已连接的共享内存地址自动脱离(detach)；进程结束后，已连接的共享内存地址会自动脱离(detach)

有以下几种错误

~~~
	
EACCES：无权限以指定方式连接共享内存

EINVAL：无效的参数shmid或shmaddr

ENOMEM：核心内存不足
~~~

---

## semget

**`sys/sem.h`** 中有关于 `semget` 的原型声明

~~~
/* Get semaphore.  */
extern int semget (key_t __key, int __nsems, int __semflg) __THROW;
~~~

得到一个信号量集标识符或创建一个信号量集对象并返回信号量集标识符

**`__key`**  通常要求此值来源于ftok返回的IPC键值

**`__nsems`** 创建信号量集中信号量的个数，该参数只在创建信号量集时有效

**`__semflg`**  IPC_CREAT：当 `semflg&IPC_CREAT`为真时，如果内核中不存在键值与key相等的信号量集，则新建一个信号量集；如果存在这样的信号量集，返回此信号量集的标识符；`IPC_CREAT|IPC_EXCL`：如果内核中不存在键值与key相等的信号量集，则新建一个消息队列；如果存在这样的信号量集则报错


如果成功则返回信号量集的标识符，如果失败则返回-1，错误原因存于error中

有以下几种错误

~~~
EACCESS：没有权限

EEXIST：信号量集已经存在，无法创建

EIDRM：信号量集已经删除

ENOENT：信号量集不存在，同时semflg没有设置IPC_CREAT标志

ENOMEM：没有足够的内存创建新的信号量集

ENOSPC：超出限制
~~~

---

## semop


**`sys/sem.h`** 中有关于 `semop` 的原型声明

~~~
/* Operate on semaphore.  */
extern int semop (int __semid, struct sembuf *__sops, size_t __nsops) __THROW;
~~~

对信号量集标识符为semid中的一个或多个信号量进行P操作或V操作

**`__semid`**   信号量集标识符

**`__sops`**  sembuf 结构体的指针，这个结构体里存放着操作的内容，结构体相关内容可以参看前面的说明

**`__nsops`**  进行操作信号量的个数，即sops结构变量的个数，需大于或等于1。最常见设置此值等于1，只完成对一个信号量的操作

如果成功则返回信号量集的标识，如果出错，则返回-1，错误原因存于error中


有以下几种错误

~~~
E2BIG：一次对信号量个数的操作超过了系统限制

EACCESS：权限不够

EAGAIN：使用了IPC_NOWAIT，但操作不能继续进行

EFAULT：sops指向的地址无效

EIDRM：信号量集已经删除

EINTR：当睡眠时接收到其他信号

EINVAL：信号量集不存在,或者semid无效

ENOMEM：使用了SEM_UNDO，但无足够的内存创建所需的数据结构

ERANGE：信号量值超出范围
~~~


---

## shmdt


**`sys/shm.h`** 中有关于 `shmdt` 的原型声明

~~~
/* Detach shared memory segment.  */
extern int shmdt (__const void *__shmaddr) __THROW;
~~~

与shmat函数相反，是用来断开与共享内存附加点的地址，禁止本进程访问此片共享内存

**`__shmaddr`**  连接的共享内存的起始地址

成功则返回0，出错则返回-1，错误原因存于error中

>本函数调用并不删除所指定的共享内存区，而只是将先前用shmat函数连接（attach）好的共享内存脱离（detach）目前的进程


有以下几种错误

~~~
EINVAL：无效的参数shmaddr
~~~



---

## semctl


**`sys/sem.h`** 中有关于 `semctl` 的原型声明

~~~
/* Semaphore control operation.  */
extern int semctl (int __semid, int __semnum, int __cmd, ...) __THROW;
~~~

在指定的信号集或信号集内的某个信号上执行控制操作

**`__semid`**  信号量集标识符

**`__semnum`**  信号量集数组上的下标，表示某一个信号量

**`__cmd`**  可以取以下的宏


~~~
/* Commands for `semctl'.  */
#define GETPID          11              /* get sempid */
#define GETVAL          12              /* get semval */
#define GETALL          13              /* get all semval's */
#define GETNCNT         14              /* get semncnt */
#define GETZCNT         15              /* get semzcnt */
#define SETVAL          16              /* set semval */
#define SETALL          17              /* set all semval's */
~~~

---

## sembuf 共用体


**`bits/sem.h`** 中有关于 sembuf 的说明

~~~
/* The user should define a union like the following to use it for arguments
   for `semctl'.

   union semun
   {
     int val;                           <= value for SETVAL
     struct semid_ds *buf;              <= buffer for IPC_STAT & IPC_SET
     unsigned short int *array;         <= array for GETALL & SETALL
     struct seminfo *__buf;             <= buffer for IPC_INFO
   };

   Previous versions of this file used to define this union but this is
   incorrect.  One can test the macro _SEM_SEMUN_UNDEFINED to see whether
   one must define the union or not.  */
~~~

这是一个共用体，用作 semctl 的参数，不同的共用体成员可以用于不同的情景中


---

## semid_ds 结构体

**`bits/sem.h`** 中有关于 semid_ds 的说明


~~~
/* Data structure describing a set of semaphores.  */
struct semid_ds
{
  struct ipc_perm sem_perm;             /* operation permission struct */
  __time_t sem_otime;                   /* last semop() time */
  unsigned long int __unused1;
  __time_t sem_ctime;                   /* last time changed by semctl() */
  unsigned long int __unused2;
  unsigned long int sem_nsems;          /* number of semaphores in set */
  unsigned long int __unused3;
  unsigned long int __unused4;
};
~~~

这个结构体的指针可以在semctl中作为参数获取信号量的信息


---

## seminfo 结构体

**`bits/sem.h`** 中有关于 seminfo 结构体的说明

~~~
struct  seminfo
{
  int semmap;
  int semmni;
  int semmns;
  int semmnu;
  int semmsl;
  int semopm;
  int semume;
  int semusz;
  int semvmx;
  int semaem;
};
~~~

这个结构体的指针可以在semctl中作为参数获取信号量的信息


---

## shmctl

**`sys/shm.h`** 中有关于 shmctl 的原型声明

~~~
/* Shared memory control operation.  */
extern int shmctl (int __shmid, int __cmd, struct shmid_ds *__buf) __THROW;
~~~

完成对共享内存的控制

**`__shmid`**    共享内存标识符

**`__cmd`**   `IPC_STAT`：得到共享内存的状态，把共享内存的shmid_ds结构复制到buf中；`IPC_SET`：改变共享内存的状态，把buf所指的shmid_ds结构中的uid、gid、mode复制到共享内存的shmid_ds结构内；`IPC_RMID`：删除这片共享内存

**`__buf`**  共享内存管理结构体指针

如果成功则返回0，如果出错则返回-1，错误原因存于error中

可能的错误有

~~~
EACCESS：参数cmd为IPC_STAT，确无权限读取该共享内存

EFAULT：参数buf指向无效的内存地址

EIDRM：标识符为msqid的共享内存已被删除

EINVAL：无效的参数cmd或shmid

EPERM：参数cmd为IPC_SET或IPC_RMID，却无足够的权限执行
~~~

---

## IPC_X 宏定义


**`bits/ipc.h`** 中有关 IPC_X 的宏定义

~~~
/* Control commands for `msgctl', `semctl', and `shmctl'.  */
#define IPC_RMID        0               /* Remove identifier.  */
#define IPC_SET         1               /* Set `ipc_perm' options.  */
#define IPC_STAT        2               /* Get `ipc_perm' options.  */
#ifdef __USE_GNU
# define IPC_INFO       3               /* See ipcs.  */
#endif
~~~

---

## shmid_ds 结构体

**`bits/shm.h`** 中有关 shmid_ds 的宏定义

~~~
/* Data structure describing a shared memory segment.  */
struct shmid_ds
  {
    struct ipc_perm shm_perm;           /* operation permission struct */
    size_t shm_segsz;                   /* size of segment in bytes */
    __time_t shm_atime;                 /* time of last shmat() */
#if __WORDSIZE == 32
    unsigned long int __unused1;
#endif
    __time_t shm_dtime;                 /* time of last shmdt() */
#if __WORDSIZE == 32
    unsigned long int __unused2;
#endif
    __time_t shm_ctime;                 /* time of last change by shmctl() */
#if __WORDSIZE == 32
    unsigned long int __unused3;
#endif
    __pid_t shm_cpid;                   /* pid of creator */
    __pid_t shm_lpid;                   /* pid of last shmop */
    shmatt_t shm_nattch;                /* number of current attaches */
    unsigned long int __unused4;
    unsigned long int __unused5;
  };

~~~

---

## ipc_perm 结构体

**`bits/ipc.h`** 中有关 ipc_perm 结构体的宏定义

~~~
/* Data structure used to pass permission information to IPC operations.  */
struct ipc_perm
  {
    __key_t __key;                      /* Key.  */
    __uid_t uid;                        /* Owner's user ID.  */
    __gid_t gid;                        /* Owner's group ID.  */
    __uid_t cuid;                       /* Creator's user ID.  */
    __gid_t cgid;                       /* Creator's group ID.  */
    unsigned short int mode;            /* Read/write permission.  */
    unsigned short int __pad1;
    unsigned short int __seq;           /* Sequence number.  */
    unsigned short int __pad2;
    unsigned long int __unused1;
    unsigned long int __unused2;
  };
~~~


---

# 总结

以下函数可以进行信号量和共享内存的创建与控制

* shmget
* shmat
* shmdt
* shmctl
* semget
* semop
* semctl

通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础



[10243617]:http://blog.csdn.net/ljianhui/article/details/10243617
[10253345]:http://blog.csdn.net/ljianhui/article/details/10253345

