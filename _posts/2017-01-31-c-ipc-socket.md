---
layout:  post
title: 进程间通讯(七).socket
author:  wilmosfang
tags:   c 
categories:  c
wc:  724  1515 24193 
excerpt: 套接字、 sockaddr 结构体、in_addr 结构体、sockaddr_in 结构体、socket、AF_INET 和 SOCK_STREAM 宏定义、htons、INADDR_ANY 宏定义、setsockopt、bind、listen、accept、recv、send、inet_addr、connect
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

## 套接字

之前的各种通信机制如：pipe，FIFO，message queue，signal ，semaphore ，shared memory 都局限于同一台计算机上的进程间通信

但是要实现不同计算机（通过网络相连）上的进程互相通信，就需要网络进程间通信（network IPC）

套接字允许进程与不同计算机上的以及同一计算机上的其它进程通信

网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket

建立网络通信连接至少要一对端口号(socket)。socket本质是编程接口(API)，对TCP/IP的封装，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口

>Socket的英文原义是“孔”或“插座”。作为BSD UNIX的进程通信机制，取后一种意思。通常也称作"套接字"，用于描述IP地址和端口，是一个通信链的句柄，可以用来实现不同虚拟机或不同计算机之间的通信。在Internet上的主机一般运行了多个服务软件，同时提供几种服务。每种服务都打开一个Socket，并绑定到一个端口上，不同的端口对应于不同的服务。Socket正如其英文原意那样，像一个多孔插座。一台主机犹如布满各种插座的房间，每个插座有一个编号，有的插座提供220伏交流电， 有的提供110伏交流电，有的则提供有线电视节目。 客户软件将插头插到不同编号的插座，就可以得到不同的服务

---

## 代码示例

### 要求

编写一个网络通讯程序，客户端通过指定IP地址的方式向服务端发送一段字符串，服务端收到后显示并且作出响应，然后退出



### 代码示例

**`tcpserver.c`**

~~~
#include <stdio.h> //perror,printf 相关函数在此声明
#include <netinet/in.h> //sockaddr_in,htons,htonl,socket,AF_INET,SOCK_STREAM,INADDR_ANY,SOL_SOCKET,SO_REUSEADDR,bind,listen,accept,recv,send 相关声明和定义在这个文件中
#include <string.h> //memset 相关函数在此声明
#include <unistd.h> //close 相关函数在此声明

#define MAX_CONN 2
#define BUF_SIZE 1024
#define PORT 9000 

int main()
{
  struct sockaddr_in server_sai,client_sai;
  int sfd=0,cfd=0,res=-1,on=1,recvbytes=0,sendbytes=0;
  int addrlen=sizeof(struct sockaddr);
  char buf[BUF_SIZE]; //各种变量定义与初始化

  if(-1 == (sfd=socket(AF_INET,SOCK_STREAM,0))) //创建一个IPV4的TCP socket
  {
    perror("socket");
    return res;
  }
  
  server_sai.sin_family=AF_INET; //IPV4 协议族
  server_sai.sin_port=htons(PORT); //9000端口
  server_sai.sin_addr.s_addr=htonl(INADDR_ANY); //0.0.0.0 的通配监听
  memset(&(server_sai.sin_zero),0,sizeof(server_sai.sin_zero)); //将剩余部分填零
  setsockopt(sfd,SOL_SOCKET,SO_REUSEADDR,&on,sizeof(on)); //closesocket（一般不会立即关闭而经历TIME_WAIT的过程）后想继续重用该socket

  if(-1 == bind(sfd,(struct sockaddr *)&server_sai,sizeof(struct sockaddr))) //将 sfd 和 socket 地址进行绑定
  {
    perror("bind");
    return res;
  }
  
  if (-1 == (listen(sfd,MAX_CONN))) //在sfd上进行监听，最多允许同时有2个请求在队列中排队，此配置正是DDOS的攻击点，协议天然的缺陷在于，不论这个值设多设少，都不会是一个适合的值
  {
    perror("listen");
    return res;
  }
  else printf("Listening...\n");

  if(-1 == (cfd=accept(sfd,(struct sockaddr *)&client_sai,(socklen_t *)&addrlen))) //接受连接，将返回的描述符赋给cfd
  {
    perror("accept");
    return res;
  }

  memset(buf,0,sizeof(buf)); //将缓存置零
  
  if(-1  == (recvbytes = recv(cfd,buf,BUF_SIZE,0))) //从对端接受内容并且存到buf中
  {
    perror("recv");
    return res;
  }
  
  printf("Received a message:%s\n",buf); //将收到的内容输出
  
  if(-1  == (sendbytes = send(cfd,"OK",2,0))) //给客户端回复一个ok
  {
    perror("send");
    return res;
  }

  close(sfd);
  close(cfd); //进行清理，关闭打开的描述符
  res=0;
  return res;
}
~~~

**`tcpclient.c`**

~~~
#include <stdio.h> //printf,sprintf,perror 相关声明在此文件中
#include <string.h> //memset,strlen
#include <unistd.h> //close
#include <arpa/inet.h> //sockaddr_in,socket,AF_INET,SOCK_STREAM,htons,inet_addr,connect,sockaddr,send,recv //相关定义和声明在此文件中

#define MAX_CONN 2
#define BUF_SIZE 1024
#define PORT 9000 

int main(int argc,char *argv[])
{
  struct sockaddr_in server_sai;
  int sfd=0,res=-1,recvbytes=0,sendbytes=0;
  char buf[BUF_SIZE],buf2[5]={0}; //进行变量的定义和初始化
  
  if(argc < 3) //如果参数小于3个就报错，命令后面会分别加上IP地址和消息内容，所以一共是三个参数
  {
    printf("error number of argc:%d\n",argc);
    return res;
  }
  
  memset(buf,0,sizeof(buf)); //对buf清零
  sprintf(buf,"%s",argv[2]); //将要传输的内容(第二个参数)复制到buf中

  if(-1 == (sfd=socket(AF_INET,SOCK_STREAM,0))) //创建一个IPV4的TCP socket
  {
    perror("socket");
    return res;
  }

  server_sai.sin_family=AF_INET;   //IPV4 协议族
  server_sai.sin_port=htons(PORT); //9000端口
  server_sai.sin_addr.s_addr=inet_addr(argv[1]); //使用第一个参数作为IP地址
  memset(&(server_sai.sin_zero),0,sizeof(server_sai.sin_zero)); //将结构体剩余部分填零
  
  if  (-1 == connect(sfd,(struct sockaddr *)&server_sai,sizeof(struct sockaddr))) //使用sfd进行连接
  {
    perror("connect");
    return res;
  }
  
  if (-1 == (sendbytes=send(sfd,buf,strlen(buf),0))) //将buf中的内容写到远端服务端，buf中的内容是命令行中的第二个参数
  {
    perror("send");
    return res;
  }
  
  recvbytes=recv(sfd,buf2,5,0); //从服务端接收数据，写到buf2中
  printf("%d -->%s\n",recvbytes,buf2); //将buf2中的数据显示出来
  close(sfd); //进行清理工作，关闭描述符

  res=0;
  return res;
}
~~~


### 编译执行

~~~
emacs@ubuntu:~/c$ alias  gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc tcpclient.x tcpclient.c ; gtc tcpserver.x tcpserver.c
emacs@ubuntu:~/c$ 
~~~

此时系统中并没有开放9000端口

~~~
emacs@ubuntu:~/c$ netstat -ant  | grep 9000
emacs@ubuntu:~/c$ 
~~~

运行服务端

~~~
emacs@ubuntu:~/c$ ./tcpserver.x 
Listening...

~~~

此时系统中多了一个9000端口

~~~
emacs@ubuntu:~/c$ netstat -ant  | grep 9000
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN     
emacs@ubuntu:~/c$
~~~

运行客户端，会立刻返回

~~~
emacs@ubuntu:~/c$ ./tcpclient.x 127.0.0.1 hello
2 -->OK
emacs@ubuntu:~/c$ 
~~~

服务端会打印信息并且返回

~~~
emacs@ubuntu:~/c$ ./tcpserver.x 
Listening...
Received a message:hello
emacs@ubuntu:~/c$ 
~~~


编译执行过程中没有报错，从结果来看，符合预期

---

## sockaddr 结构体

**`bits/socket.h`** 中有关于 sockaddr 结构体的定义

~~~
/* Structure describing a generic socket address.  */
struct sockaddr
  {
    __SOCKADDR_COMMON (sa_);    /* Common data: address family and length.  */
    char sa_data[14];           /* Address data.  */
  };
~~~

---

## in_addr 结构体


**`netinet/in.h`** 中有关于 in_addr 结构体的定义

~~~
/* Internet address.  */
typedef uint32_t in_addr_t;
struct in_addr
  {
    in_addr_t s_addr;
  };
~~~

---

## sockaddr_in 结构体

**`netinet/in.h`** 中有关于 sockaddr_in 结构体的定义

~~~
/* Structure describing an Internet socket address.  */
struct sockaddr_in
  {
    __SOCKADDR_COMMON (sin_);
    in_port_t sin_port;                 /* Port number.  */
    struct in_addr sin_addr;            /* Internet address.  */

    /* Pad to size of `struct sockaddr'.  */
    unsigned char sin_zero[sizeof (struct sockaddr) -
                           __SOCKADDR_COMMON_SIZE -
                           sizeof (in_port_t) -
                           sizeof (struct in_addr)];
  };
~~~

从 **`sin_zero[sizeof (struct sockaddr) -
                           __SOCKADDR_COMMON_SIZE -
                           sizeof (in_port_t) -
                           sizeof (struct in_addr)]`**  中可以看出

这个字段，是为填补与 **`sockaddr`** 结构体的长度差

二者的占用的内存大小是一致的，因此可以互相转化，从这个意义上说，他们并无区别

---

## socket

**`sys/socket.h`** 中有关于 socket 函数的声明

~~~
/* Create a new socket of type TYPE in domain DOMAIN, using
   protocol PROTOCOL.  If PROTOCOL is zero, one is chosen automatically.
   Returns a file descriptor for the new socket, or -1 for errors.  */
extern int socket (int __domain, int __type, int __protocol) __THROW;
~~~

用于创建一个socket描述符（socket descriptor），它唯一标识一个socket。这个socket描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作

**`__domain`**  即协议域，又称为协议族（family）。常用的协议族有，AF_INET、AF_INET6、AF_LOCAL（或称AF_UNIX，Unix域socket）、AF_ROUTE等等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址（32位的）与端口号（16位的）的组合、AF_UNIX决定了要用一个绝对路径名作为地址

**`__type`**  指定socket类型。常用的socket类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等等

**`__protocol`**  指定协议。常用的协议有，IPPROTO_TCP、IPPTOTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC等，它们分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议


> **Note:**  并不是上面的type和protocol可以随意组合的，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当protocol为0时，会自动选择type类型对应的默认协议



---

## AF_INET 和 SOCK_STREAM 宏定义

**`bits/socket.h`** 中有关于 AF_INET 和 SOCK_STREAM 的宏定义

~~~
emacs@ubuntu:/usr/include$ grep AF_INET bits/socket.h
#define	AF_INET		PF_INET
#define	AF_INET6	PF_INET6
emacs@ubuntu:/usr/include$ grep PF_INET bits/socket.h
#define	PF_INET		2	/* IP protocol family.  */
#define	PF_INET6	10	/* IP version 6.  */
#define	AF_INET		PF_INET
#define	AF_INET6	PF_INET6
emacs@ubuntu:/usr/include$ grep SOCK_STREAM bits/socket.h
  SOCK_STREAM = 1,		/* Sequenced, reliable, connection-based
#define SOCK_STREAM SOCK_STREAM
emacs@ubuntu:/usr/include$
~~~

使用此方法可以获取其它想要的宏定义

---

## htons


**`netinet/in.h`** 中有关于 htons 的定义

~~~
/* Functions to convert between host and network byte order.

   Please note that these functions normally take `unsigned long int' or
   `unsigned short int' values as arguments and also return them.  But
   this was a short-sighted decision since on different systems the types
   may have different representations but the values are always the same.  */

extern uint32_t ntohl (uint32_t __netlong) __THROW __attribute__ ((__const__));
extern uint16_t ntohs (uint16_t __netshort)
     __THROW __attribute__ ((__const__));
extern uint32_t htonl (uint32_t __hostlong)
     __THROW __attribute__ ((__const__));
extern uint16_t htons (uint16_t __hostshort)
     __THROW __attribute__ ((__const__));
~~~

网络字节顺序与系统字节顺序不一定相同

>网络字节顺序（大端顺序）是指一个数在内存中存储的时候“高对低，低对高”（即一个数的高位字节存放于低地址单元，低位字节存放在高地址单元中）。但是计算机的内存存储数据时有可能是大端顺序或者小端顺序

而上面的函数就是用来进行这方面转化工作的

* h：host 本地主机端
* to：就是to，转化为
* n：net 网络端
* l：是 unsigned long (32bit)
* s：是 unsigned short (16bit)

**`ntohl`**  无符号长整型，从网络到本机

**`ntohs`** 无符号短整型，从网络到本机

**`htonl`** 无符号长整型，从本机到网络

**`htons`**  无符号短整型，从本机到网络


---

## INADDR_ANY 宏定义

**`netinet/in.h`** 中有关于 INADDR_ANY 的定义

~~~
/* Address to accept any incoming messages.  */
#define INADDR_ANY              ((in_addr_t) 0x00000000)
/* Address to send to all hosts.  */
#define INADDR_BROADCAST        ((in_addr_t) 0xffffffff)
/* Address indicating an error return.  */
#define INADDR_NONE             ((in_addr_t) 0xffffffff)
~~~

INADDR_ANY 代表通配 0.0.0.0



---

## setsockopt

**`sys/socket.h`** 中有关于 setsockopt 的定义


~~~
/* Set socket FD's option OPTNAME at protocol level LEVEL
   to *OPTVAL (which is OPTLEN bytes long).
   Returns 0 on success, -1 for errors.  */
extern int setsockopt (int __fd, int __level, int __optname,
                       __const void *__optval, socklen_t __optlen) __THROW;
~~~

**`__fd`**  指向一个打开的套接口描述字

**`__level`**  指定选项代码的类型

**`__optname`**  选项名称

**`__optval`**  是一个指向变量的指针，类型为整形

**`__optlen`**  optval 的size大小


标志打开或关闭某个特征的二进制选项


closesocket（一般不会立即关闭而经历TIME_WAIT的过程）后想继续重用该socket

~~~
BOOL bReuseaddr=TRUE;
setsockopt(s,SOL_SOCKET ,SO_REUSEADDR,(const char*)&bReuseaddr,sizeof(BOOL));
~~~

---

## bind

**`sys/socket.h`** 中有关于 bind 的定义

~~~
/* Give the socket FD the local address ADDR (which is LEN bytes long).  */
extern int bind (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)
     __THROW;
~~~

**`__fd`**  指定地址与哪个套接字绑定，这是一个由之前的socket函数调用返回的套接字。调用bind的函数之后，该套接字与一个相应的地址关联，发送到这个地址的数据可以通过这个套接字来读取与使用

**`__addr`** 指定地址。这是一个地址结构，并且是一个已经经过填写的有效的地址结构。调用bind之后这个地址与参数sockfd指定的套接字关联，从而实现上面所说的效果

**`__len`** 正如大多数socket接口一样，内核不关心地址结构，当它复制或传递地址给驱动的时候，它依据这个值来确定需要复制多少数据。这已经成为socket接口中最常见的参数之一了

成功，返回0；出错，返回－1，相应地设定全局变量errno

~~~
EACCESS：地址空间受保护，用户不具有超级用户的权限
EADDRINUSE：指定的地址已经在使用
EBADF：sockfd参数为非法的文件描述符
EINVAL：socket已经和地址绑定
ENOTSOCK：参数sockfd为文件描述符
~~~

> **Tip:**  bind函数并不是总是需要调用的，只有用户进程想与一个具体的地址或端口相关联的时候才需要调用这个函数。如果用户进程没有这个需要，那么程序可以依赖内核的自动的选址机制来完成自动地址选择，而不需要调用bind的函数，同时也避免不必要的复杂度。在一般情况下，对于服务器进程问题需要调用bind函数，对于客户进程则不需要调用bind函数


---

## listen

**`sys/socket.h`** 中有关于 listen 的定义

~~~
/* Prepare to accept connections on socket FD.
   N connection requests will be queued before further requests are refused.
   Returns 0 on success, -1 for errors.  */
extern int listen (int __fd, int __n) __THROW;
~~~

listen函数在一般在调用bind之后-调用accept之前调用。用户在调用socket函数之后，返回一个套接字sockfd. sockfd默认一个主动连接的套接字，也就是此时系统假设用户会对这个套接字调用connect函数，期待它主动与其它进程连接，然后在服务器编程中，用户希望这个套接字可以接受外来的连接请求，也就是被动等待用户来连接。由于系统默认时认为一个套接字是主动连接的，所以需要通过某种方式来告诉系统，用户进程通过系统调用listen来完成这件事

listen函数可使得流套接字sockfd处于监听状态，使得一个进程可以接受其它进程的请求，从而成为一个服务器进程。在TCP服务器编程中listen函数把进程变为一个服务器，并指定相应的套接字变为被动连接

处于监听状态的套接字sockfd将维护一个客户连接请求队列，该队列最多容纳backlog个用户请求

**`__fd`** 套接字

**`__n`** 队列最多同时容纳用户请求的个数

返回：0 成功， -1 失败

---

## accept


**`sys/socket.h`** 中有关于 accept 的定义

~~~
/* Await a connection on socket FD.
   When a connection arrives, open a new socket to communicate with it,
   set *ADDR (which is *ADDR_LEN bytes long) to the address of the connecting
   peer and *ADDR_LEN to the address's actual length, and return the
   new socket's descriptor, or -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int accept (int __fd, __SOCKADDR_ARG __addr,
                   socklen_t *__restrict __addr_len);
~~~

服务器编程中最重要的一步是等待并接受客户的连接，那么这一步在编程中如何完成，accept函数就是完成这一步的。它从内核中取出已经建立的客户连接，然后把这个已经建立的连接返回给用户程序，此时用户程序就可以与自己的客户进行点到点的通信了

**`__fd`**  指定处于监听状态的流套接字，这个套接字用来监听一个端口，当有一个客户与服务器连接时，它使用这个一个端口号，而此时这个端口号正与这个套接字关联。当然客户不知道套接字这些细节，它只知道一个地址和一个端口号

**`__addr`**  返回新创建的套接字的地址结构，它用来接受一个返回值，这返回值指定客户端的地址，当然这个地址是通过某个地址结构来描述的，用户应该知道这一个什么样的地址结构。如果对客户的地址不感兴趣，那么可以把这个值设置为NULL

**`__addr_len`**  新创建的套接字的地址结构的长度，用来接受上述addr的结构的大小，它指明addr结构所占有的字节个数。同样的，它也可以被设置为NULL

如果accept成功返回，则服务器与客户已经正确建立连接了，此时服务器通过accept返回的套接字来完成与客户的通信

返回：非负描述字成功， -1失败

>有人从很远的地方通过一个在侦听 (listen()) 的端口连接 (connect()) 到机器。它的连接将加入到等待接受 (accept()) 的队列中


---

## recv 

**`sys/socket.h`** 中有关于 recv  的声明

~~~
/* Read N bytes into BUF from socket FD.
   Returns the number read or -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern ssize_t recv (int __fd, void *__buf, size_t __n, int __flags);
~~~

从TCP连接的另一端接收数据

**`__fd`**  指定接收端套接字描述符

**`__buf`**  指明一个缓冲区，该缓冲区用来存放recv函数接收到的数据

**`__n`** 指明buf的长度 

**`__flags`**  参数一般置0


返回值： <0 出错 ；==0 对方调用了close API来关闭连接 ；>0 接收到的数据大小

阻塞模式下recv会一直阻塞直到接收到数据，非阻塞模式下如果没有数据就会返回，不会阻塞着读，因此需要循环读取）

可能错误

~~~
EAGAIN：套接字已标记为非阻塞，而接收操作被阻塞或者接收超时
EBADF：sock不是有效的描述词
ECONNREFUSE：远程主机阻绝网络连接
EFAULT：内存空间访问出错
EINTR：操作被信号中断
EINVAL：参数无效
ENOMEM：内存不足
ENOTCONN：与面向连接关联的套接字尚未被连接上
ENOTSOCK：sock索引的不是套接字
~~~


---

## send

**`sys/socket.h`** 中有关于 send 的声明

~~~
/* Send N bytes of BUF to socket FD.  Returns the number sent or -1.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern ssize_t send (int __fd, __const void *__buf, size_t __n, int __flags);
~~~

向TCP连接的另一端发送数据

**`__fd`**  指定发送端套接字描述符

**`__buf`** 指明一个存放应用程序要发送数据的缓冲区

**`__n`** 指明实际要发送的数据的字节数

**`__flags`**  参数一般置0



| flags| 说明| recv|send|
| :------- | :---- | :---: |:---:|
|MSG_DONTROUTE|	绕过路由表查找|| •|
|MSG_DONTWAIT|仅本操作非阻塞|	  •  |  •|
|MSG_OOB　　|发送或接收带外数据|	 •	| •|
|MSG_PEEK  |窥看外来消息|  •  ||
|MSG_WAITALL　|等待所有数据 | •||	  


返回值 ：>0 表示发送的字节数（实际上是拷贝到发送缓冲中的字节数）；==0 对方调用了close API来关闭连接 ；<0 发送失败，错误原因存于全局变量errno中

~~~
EBADF 参数s 非合法的socket处理代码
EFAULT 参数中有一指针指向无法存取的内存空间
ENOTSOCK 参数s为一文件描述词，非socket
EINTR 被信号所中断
EAGAIN 此操作会令进程阻断，但参数s的socket为不可阻断
ENOBUFS 系统的缓冲内存不足
ENOMEM 核心内存不足
EINVAL 传给系统调用的参数不正确
~~~

---

## inet_addr

**`arpa/inet.h`** 中有关于 inet_addr 的声明


~~~
/* Convert Internet host address from numbers-and-dots notation in CP
   into binary data in network byte order.  */
extern in_addr_t inet_addr (__const char *__cp) __THROW;
~~~

将一个ip地址字符串转换成一个整数值，一般的IP地址串格式为：'a.b.c.d' 分成四段

**`__cp`**  字符串指针


---

## connect

**`sys/socket.h`** 中有关于 connect 的声明

~~~
/* Open a connection on socket FD to peer at ADDR (which LEN bytes long).
   For connectionless socket types, just set the default address to send to
   and the only address from which to accept transmissions.
   Return 0 on success, -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int connect (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len);
~~~

用于建立与指定socket的连接

**`__fd`**  标识一个未连接的socket

**`__addr`** 指向要连接套接字的sockaddr结构体的指针

**`__len`** sockaddr结构体的字节长度

返回值 ： 成功则返回0，失败则返回非0，错误码GetLastError()


~~~
EBADF 参数sockfd 非合法socket处理代码
EFAULT 参数serv_addr指针指向无法存取的内存空间
ENOTSOCK 参数sockfd为一文件描述词，非socket
EISCONN 参数sockfd的socket已是连线状态
ECONNREFUSED 连线要求被server端拒绝
ETIMEDOUT 企图连线的操作超过限定时间仍未有响应
ENETUNREACH 无法传送数据包至指定的主机
EAFNOSUPPORT sockaddr结构的sa_family不正确
EALREADY socket为不可阻断且先前的连线操作还未完成
~~~



---

# 总结

以下函数可以进行信号量和线程的创建与控制

* socket
* htons
* setsockopt
* bind
* listen
* accept
* recv
* send
* inet_addr
* connect

通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础

[19901763]:https://www.zhihu.com/question/19901763

