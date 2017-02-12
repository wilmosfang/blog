---
layout:  post
title: 网络编程(二).UDP
author:  wilmosfang
tags:   c 
categories:  c
wc:  519   979 15958 
excerpt:  网络编程，UDP、UDP 编程步骤、TCP和UDP的区别
comments: true
---


# 前言

不同计算机中的进程间通讯奠定了当前网络世界的基础

网络进程间通信是通过 socket 实现的

目前世界上最为流行的就是 `TCP/IP` 协议栈

这个协议栈中有两种通讯方式

* TCP
* UDP

UDP 的通讯过程如下：

![udp_sockets.jpg](/images/udp_sockets.jpg)

这里分享一下我在学习UDP网络编程过程中的笔记和心得

---

# 概要

* TOC
{:toc}

---

## UDP

UDP不提供复杂的控制机制，利用IP提供 **面向无连接** 的通信服务。并且它是将应用程序发来的数据在收到的那一刻，立刻按照原样发送到网络上的一种机制。即使是出现网络拥堵的情况下，UDP也无法进行流量控制等避免网络拥塞的行为

此外，传输途中如果出现了丢包，UDP也不负责重发。甚至当出现包的到达顺序乱掉时也没有纠正的功能。如果需要这些细节控制，那么不得不交给由采用UDP的应用程序去处理

换句话说，UDP将部分控制转移到应用程序去处理，自己却只提供作为传输层协议的最基本功能。UDP有点类似于用户说什么听什么的机制，但是需要用户充分考虑好上层协议类型并制作相应的应用程序


> **Tip:** TCP和UDP是OSI模型中的运输层中的协议。TCP提供可靠的通信传输，而UDP则常被用于让广播和细节控制交给应用的通信传输

---

## UDP 编程步骤

### 服务器端

UDP编程的服务器端一般步骤是：

* 1、创建一个socket，用函数socket()； 
* 2、设置socket属性，用函数setsockopt();* 可选 
* 3、绑定IP地址、端口等信息到socket上，用函数bind(); 
* 4、循环接收数据，用函数recvfrom(); 
* 5、关闭网络连接；

### 客户端

UDP编程的客户端一般步骤是：


* 1、创建一个socket，用函数socket()； 
* 2、设置socket属性，用函数setsockopt();* 可选 
* 3、绑定IP地址、端口等信息到socket上，用函数bind();* 可选 
* 4、设置对方的IP地址和端口等属性; 
* 5、发送数据，用函数sendto(); 
* 6、关闭网络连接； 


> **Tip:** 引自 **[《TCP和UDP的最完整的区别》][52117463]**

---

## 代码示例

### 要求

客户端用UDP将一幅图片或者文件（1M以上）上传到另一台PC上（服务器），并且用diff测试大小。（注意分包）

### 代码示例

**`udpserver.c`**

~~~
#include <stdio.h> //perror,printf  相关函数的声明包含在内
#include <netinet/in.h> //sockaddr_in,socket,AF_INET,SOCK_DGRAM,htons,htonl,INADDR_ANY,setsockopt,SOL_SOCKET,SO_REUSEADDR,bind,recvfrom,sendto 相关声明和定义包含在内
#include <string.h> //memset 相关函数的声明包含在内
#include <unistd.h> //write,close 相关函数的声明包含在内
#include <fcntl.h> //open,O_RDWR,O_CREAT,O_TRUNC 相关函数的声明和宏定义包含在内

#define MAX_CONN 2
#define BUF_SIZE 1024
#define PORT 9000 

int main()
{
  struct sockaddr_in server_sai,client_sai;
  int sfd=0,res=-1,on=1,recvbytes=0,sendbytes=0,writebytes=0,fa=0;
  int addrlen=sizeof(struct sockaddr);
  char buf[BUF_SIZE];
  char *filename="/tmp/x.download"; //进行各种变量的定义和初始化

  if(-1 == (sfd=socket(AF_INET,SOCK_DGRAM,0))) //创建一个IPV4的UDP socket
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
  
  if (-1==(fa=open(filename,O_RDWR|O_CREAT|O_TRUNC,0644))) //以写的方式打开目标文件，也就是服务端数据的存放处
  {
    printf("cannot open file:%s\n",filename);
    return res;
  }

  memset(buf,0,sizeof(buf)); //将缓存置零

  int i=0; //设定一个变量来追踪分包数，可选项

  do
  {
    i++; //每分一包，加一次，可选项
    if(-1 == (recvbytes = recvfrom(sfd,buf,sizeof(buf),0,(struct sockaddr *)&client_sai,(socklen_t *)&addrlen))) //从远端获取数据，放到buf中
    {
      perror("recvfrom");
      return res;
    }
    if(-1 == (writebytes = write(fa,buf,recvbytes))) //将buf中的数据写到文件中
    {
      printf("write error on:%s\n",filename);
      return res;
    }
    if(-1  == (sendbytes = sendto(sfd,"DONE",4,0,(struct sockaddr *)&client_sai,sizeof(struct sockaddr)))) //发送一个确认信息给远端，来同步节奏
    {
      perror("sendto");
      return res;
    }
  }while(recvbytes == sizeof(buf)); //如果读到的数据小于一整块，就意味着数据已经读完，跳出循环

  printf("i:%d\nrecvbytes:%d\n",i,recvbytes); //将分包的数目和最后一包数据的大小显示出来，可选项
  
  if(-1  == (sendbytes = sendto(sfd,"DONE",4,0,(struct sockaddr *)&client_sai,sizeof(struct sockaddr)))) //发送一个确认信息给远端
  {
    perror("sendto");
    return res;
  }
  
  close(fa);
  close(sfd); //进行清理操作，关闭所有描述符

  res=0;
  return res;
}
~~~

**`udpclient.c`**

~~~
#include <stdio.h> //printf,sprintf,perror 相关函数在此声明
#include <string.h> //memset 相关函数在此声明
#include <unistd.h> //read,close 相关函数在此声明
#include <arpa/inet.h> //sockaddr_in,socket,AF_INET,SOCK_DGRAM,htons,inet_addr,sendto,recvfrom  相关函数和宏在此声明和定义
#include <fcntl.h> //open,O_RDONLY 相关函数和宏在此声明和定义

#define BUF_SIZE 1024
#define PORT 9000 

int main(int argc,char *argv[])
{
  struct sockaddr_in server_sai;
  int sfd=0,res=-1,recvbytes=0,sendbytes=0,readbytes=0,fa=0;
  int addrlen=sizeof(struct sockaddr);
  char buf[BUF_SIZE],buf2[5]={0};
  char *filename=argv[2]; //进行变量的定义和初始化
  
  if(argc != 3)
  {
    printf("error number of argc:%d\n",argc);
    return res;
  }

  if (-1==(fa=open(argv[2],O_RDONLY,0644))) //将最后一个参数作为文件名，打开文件
  {
    printf("cannot open file:%s\n",filename);
    return res;
  }

  if(-1 == (sfd=socket(AF_INET,SOCK_DGRAM,0))) //创建一个IPV4的UDP socket
  {
    perror("socket");
    return res;
  }

  server_sai.sin_family=AF_INET; //IPV4 协议族
  server_sai.sin_port=htons(PORT); //9000端口
  server_sai.sin_addr.s_addr=inet_addr(argv[1]); //使用第一个参数作为IP地址
  
  memset(&(server_sai.sin_zero),0,sizeof(server_sai.sin_zero)); //将结构体剩余部分填零
  
  memset(buf,0,sizeof(buf)); //将buf清零

  do
  {
    if(-1 == (readbytes=read(fa,buf,sizeof(buf)))) //从指定文件中读取数据写到buf中
    {
      printf("read error on:%s\n",filename);
      return res;
    }
    if (-1 == (sendbytes=sendto(sfd,buf,readbytes,0,(struct sockaddr *)&server_sai,sizeof(struct sockaddr)))) //将buf中的数据写到远端
    {
      perror("sendto");
      return res;
    }
    if (-1 == (recvbytes=recvfrom(sfd,buf2,5,0,(struct sockaddr *)&server_sai,(socklen_t *)&addrlen))) //从远端获取信息，用于同步节奏
    {
      perror("recvfrom");
      return res;
    }
  }while(readbytes == sizeof(buf)); //如果读取的数据不再是一整块，就意味着已经读完，随即跳出循环

  
  if (-1 == (recvbytes=recvfrom(sfd,buf2,5,0,(struct sockaddr *)&server_sai,(socklen_t *)&addrlen))) //从远端读取数据到buf2中
  {
    perror("recvfrom");
    return res;
  }

  printf("%d --> %s\n",recvbytes,buf2);   //将接收到的字节数和数据内容打印出来
  
  close(sfd);
  close(fa); //进行清理工作，关闭描述符
  
  res=0;
  return res;
}
~~~


### 编译执行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc udpserver.x udpserver.c; gtc udpclient.x udpclient.c
emacs@ubuntu:~/c$ 
~~~

此时系统中并没有开放9000端口

~~~
emacs@ubuntu:~/c$ netstat -anu | grep 9000
emacs@ubuntu:~/c$ 
~~~

运行服务端

~~~
emacs@ubuntu:~/c$ ./udpserver.x 

~~~

此时系统中多了一个9000端口

~~~
emacs@ubuntu:~/c$ netstat -anu | grep 9000
udp        0      0 0.0.0.0:9000            0.0.0.0:*                          
emacs@ubuntu:~/c$ 
~~~

服务端也并没有 `/tmp/x.download` 这个文件

~~~
emacs@ubuntu:~/c$ ll /tmp/x.download 
ls: 无法访问/tmp/x.download: 没有那个文件或目录
emacs@ubuntu:~/c$  
~~~

运行客户端，会立刻返回

~~~
emacs@ubuntu:~/c$ du -sh 4.png 
8.6M	4.png
emacs@ubuntu:~/c$ ./udpclient.x 127.0.0.1 4.png 
4 --> DONE
emacs@ubuntu:~/c$  
~~~

服务端会打印信息并且返回，对比两个文件也没有差异

~~~
emacs@ubuntu:~/c$ ./udpserver.x 
i:8786
recvbytes:860
emacs@ubuntu:~/c$ diff /tmp/x.download 4.png 
emacs@ubuntu:~/c$
~~~

编译执行过程中没有报错，从结果来看，符合预期

---

## recvfrom

**`sys/socket.h`** 中有关于 recvfrom 的声明

~~~
/* Read N bytes into BUF through socket FD.
   If ADDR is not NULL, fill in *ADDR_LEN bytes of it with tha address of
   the sender, and store the actual size of the address in *ADDR_LEN.
   Returns the number of bytes read or -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern ssize_t recvfrom (int __fd, void *__restrict __buf, size_t __n,
                         int __flags, __SOCKADDR_ARG __addr,
                         socklen_t *__restrict __addr_len);
~~~

从套接口上接收数据，并捕获数据发送源的地址

**`__fd`**  标识一个已连接套接口的描述字

**`__buf`** 接收数据缓冲区

**`__n`** 缓冲区长度

**`__flags`** 调用操作方式

**`__addr`** （可选）指针，指向装有源地址的缓冲区

**`__addr_len`**  （可选）指针，指向`__addr`缓冲区长度值

返回值：>0 返回读入的字节数； ==0 连接已中止； <0 返回SOCKET_ERROR错误，应用程序可通过WSAGetLastError()获取相应错误代码

~~~
EBADF 参数s非合法的socket处理代码
EFAULT 参数中有一指针指向无法存取的内存空间
ENOTSOCK 参数s为一文件描述词，非socket
EINTR 被信号所中断
EAGAIN 此动作会令进程阻断，但参数s的socket为不可阻断
ENOBUFS 系统的缓冲内存不足
ENOMEM 核心内存不足
EINVAL 传给系统调用的参数不正确
~~~


---

## sendto 

**`sys/socket.h`** 中有关于 sendto 的声明

~~~
/* Send N bytes of BUF on socket FD to peer at address ADDR (which is
   ADDR_LEN bytes long).  Returns the number sent, or -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern ssize_t sendto (int __fd, __const void *__buf, size_t __n,
                       int __flags, __CONST_SOCKADDR_ARG __addr,
                       socklen_t __addr_len);
~~~

适用于发送未建立连接的UDP数据包

**`__fd`** 一个标识套接口的描述字

**`__buf`** 包含待发送数据的缓冲区

**`__n`** buf缓冲区中数据的长度

**`__flags`** 调用方式标志位

**`__addr`** （可选）指针，指向目的套接口的地址

**`__addr_len`** 所指地址的长度


返回值 ：>0 返回所发送数据的总数（请注意这个数字可能小于len中所规定的大小）；==0 连接已中止 ；<0 返回SOCKET_ERROR错误，应用程序可通过WSAGetLastError()获取相应错误代码


~~~
EBADF 参数s非法的socket处理代码
EFAULT 参数中有一指针指向无法存取的内存空间
ENOTSOCK 参数 s为一文件描述词，非socket
EINTR 被信号所中断
EAGAIN 此动作会令进程阻断，但参数s的socket为不可阻断的
ENOBUFS 系统的缓冲内存不足
EINVAL 传给系统调用的参数不正确
~~~


---

## SOCK_DGRAM

**`bits/socket.h`** 中有关于 SOCK_DGRAM 的定义

~~~
/* Types of sockets.  */
enum __socket_type
{
  SOCK_STREAM = 1,              /* Sequenced, reliable, connection-based
                                   byte streams.  */
#define SOCK_STREAM SOCK_STREAM
  SOCK_DGRAM = 2,               /* Connectionless, unreliable datagrams
                                   of fixed maximum length.  */
#define SOCK_DGRAM SOCK_DGRAM
  SOCK_RAW = 3,                 /* Raw protocol interface.  */
#define SOCK_RAW SOCK_RAW
  SOCK_RDM = 4,                 /* Reliably-delivered messages.  */
#define SOCK_RDM SOCK_RDM
  SOCK_SEQPACKET = 5,           /* Sequenced, reliable, connection-based,
                                   datagrams of fixed maximum length.  */
#define SOCK_SEQPACKET SOCK_SEQPACKET
  SOCK_DCCP = 6,                /* Datagram Congestion Control Protocol.  */
#define SOCK_DCCP SOCK_DCCP
  SOCK_PACKET = 10,             /* Linux specific way of getting packets
                                   at the dev level.  For writing rarp and
                                   other similar things on the user level. */
#define SOCK_PACKET SOCK_PACKET

  /* Flags to be ORed into the type parameter of socket and socketpair and
     used for the flags parameter of paccept.  */

  SOCK_CLOEXEC = 02000000,      /* Atomically set close-on-exec flag for the
                                   new descriptor(s).  */
#define SOCK_CLOEXEC SOCK_CLOEXEC
  SOCK_NONBLOCK = 04000         /* Atomically mark descriptor(s) as
                                   non-blocking.  */
#define SOCK_NONBLOCK SOCK_NONBLOCK
};
~~~

里面规定 **`SOCK_DGRAM`** 为一种不连接，不可靠的传输模式




---

# 附：TCP和UDP的区别

> **Tip:** 引自 **[《TCP和UDP的最完整的区别》][52117463]**



~~~
TCP与UDP基本区别
  1.基于连接与无连接
  2.TCP要求系统资源较多，UDP较少； 
  3.UDP程序结构较简单 
  4.流模式（TCP）与数据报模式(UDP); 
  5.TCP保证数据正确性，UDP可能丢包 
  6.TCP保证数据顺序，UDP不保证 
　　
UDP应用场景
  1.面向数据报方式
  2.网络数据大多为短消息 
  3.拥有大量Client
  4.对数据安全性无特殊要求
  5.网络负担非常重，但对响应速度要求高
 
具体编程时的区别
  1.socket()的参数不同 
  2.UDP Server不需要调用listen和accept 
  3.UDP收发数据用sendto/recvfrom函数 
  4.TCP：地址信息在connect/accept时确定 
  5.UDP：在sendto/recvfrom函数中每次均 需指定地址信
  6.UDP：shutdown函数无效

TCP与UDP区别总结
  1.TCP面向连接（如打电话要先拨号建立连接）;UDP是无连接的，即发送数据之前不需要建立连接
  2.TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达;UDP尽最大努力交付，即不保证可靠交付
  3.TCP面向字节流，实际上是TCP把数据看成一连串无结构的字节流;UDP是面向报文的,UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如IP电话，实时视频会议等）
  4.每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信
  5.TCP首部开销20字节;UDP的首部开销小，只有8个字节
  6.TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道
~~~


---

# 总结

以下函数可以进行socket的创建与控制，是UDP网络编程的基础

* socket
* setsockopt
* bind
* recvfrom
* sendto

通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础

[52117463]:http://blog.csdn.net/li_ning_/article/details/52117463

