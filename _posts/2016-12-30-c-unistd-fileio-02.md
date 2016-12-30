---
layout:  post
title:  文件I/O (二)
author:  wilmosfang
tags:   c 
categories:  c
wc: 488  1779 19138 
excerpt:  文件IO库的常用函数、函数原型、宏定义、内存对齐、xxd、ASCII 码
comments: true
---


# 前言

当前的计算系统除了包括对数据有 **加工和处理** 以外还有 **搬运**

这个 **搬运** 代表着 **输入和输出** ，及 **input/output** ，简称 **I/O**

**UNIX/Linux** 的缔造者们将数据的 **来源和目标** 都抽象为 **文件**，所以在 **UNIX/Linux** 系统中 **一切皆文件**

**一切皆文件** 不仅仅对磁盘，还包括鼠标，键盘，显示器这些设备，那么对这些设备的操作也都抽象成了对 **文件的I/O操作**

关于 **标准I/O** 可以参看之前的文章 **[《标准I/O (一)》][c_stdio_01]** ，类Unix系统中除了 **标准I/O** 还有 **文件I/O**，可以完成相同工作，关于 **文件I/O** 还有它们之间的区别可以参看之前的文章 **[《文件I/O (一)》][c_fileio_01]**，关于C语言的API(linux)可以参看 **[Linux C API 参考手册][linux_c_api]** 在线文档

这里分享一下我在学习 **文件 I/O** 库过程中的笔记和心得


---

# 概要

* TOC
{:toc}

---

## 文件IO库的常用函数


下面是一些 **文件IO库** 中的常用函数


~~~
int open( const char *pathname, int flags)
int open( const char *pathname, int flags, mode_t mode)
ssize_t read(int fd, void *buf, size_t count)
ssize_t write(int fd, const void *buf, size_t count)
off_t lseek(int fildes, off_t offset, int whence)
int close(int fd)
~~~


---

## 代码示例

### 要求

结构体定义

~~~
struct stu
{
  int id;
  char name[5];
  int score;
};
~~~

* 1）手工输入5个学生信息，并将结果存入文件f1中
* 2）找出f1中学生分数最高的那个人（有可能多个并列第一），将这个人的信息写入文件f2.

要求：用非缓冲IO实现


### 代码示例

~~~
#include <stdio.h> 
#include <unistd.h> //文件IO函数包含其中,缺少这个头文件read,write,close 会报错
#include <fcntl.h> //open函数包含其中,还有一些重要的宏定义

typedef struct student  //student 结构体定义
{
  int id;
  char name[5];
  int score;
}ST;

int main()
{
  int res=-1,fa=0,fb=0,maxscore=0,tscore=0,i=0;
  char *fileA="f1";
  char *fileB="f2"; //变量定义与初始化
  ST stutmp,stu[5]={
    {11,"xiao",18},
    {13,"hong",88},
    {16,"tuna",98},
    {19,"tony",60},
    {90,"duno",98}
  }; 
  
  if (-1==(fa=open(fileA,O_RDWR|O_CREAT|O_TRUNC,0644))) //以读写方式打开文件A
  {
    printf("cannot open file:%s\n",fileA);
    return res;
  }
  if (-1==(fb=open(fileB,O_RDWR|O_CREAT|O_TRUNC,0644))) //以读写方式打开文件B
  {
    printf("cannot open file:%s\n",fileB);
    return res;
  }
  if(sizeof(ST)*5 != write(fa,stu,sizeof(ST)*5)) //将结构体数组所有内容写到文件A ,顺利的情况下会返回实际写入的字节数，利用这个特性来判断有没写成功
  {
    printf("write error on:%s\n",fileA);
    return res;
  }

  lseek(fa,sizeof(ST)-sizeof(int),SEEK_SET); //将文件指针定位到分数的部分，SEEK_SET 代表的是0，这个宏是在fcntl.h中定义的，意思是偏移量相对位置为文件的开头
  if(sizeof(int)!=read(fa,&maxscore,sizeof(int))) //将分数读到maxscore中，作为初始的最大分数
  {
    printf("read error on:%s\n",fileA);
    return res;
  }
  lseek(fa,sizeof(ST)-sizeof(int),SEEK_SET); //重新将文件指针定位到分数的部分
  for(i=0;i<5;i++)
  {
    if(sizeof(int) != read(fa,&tscore,sizeof(int))) //将分数写到tscore中
    {
      printf("read error on:%s\n",fileA);
      return res;
    }
    if(maxscore < tscore) maxscore=tscore; //将分数与初始的maxscore进行比较，如果当前分数较大，则替换掉maxscore中的值
    lseek(fa,sizeof(ST)-sizeof(int),SEEK_CUR); //从当前位置开始，定位到下一个分数处，SEEK_CUR代表的是1，这个宏是在fcntl.h中定义的，意思是偏移量相对位置为当前位置
  }

  lseek(fa,sizeof(ST)-sizeof(int),SEEK_SET); //重新将文件指针定位到第一个分数的位置
  for(tscore=0,i=0;i<5;i++)
  {
    if(sizeof(int) != read(fa,&tscore,sizeof(int))) //将分数写到tscore中
    {
      printf("read error on:%s\n",fileA);
      return res;
    }
    if(maxscore == tscore) //如果tscore与maxscore相等，就读取这个结构体的内容，并将这个结构体的内容写到文件B中
    {
      lseek(fa,-1*sizeof(ST),SEEK_CUR); //从当前位置后退一个结构体的长度，注意 -1*与SEEK_CUR的用法
      if (sizeof(ST) != read(fa,&stutmp,sizeof(ST))) //读取这个结构体的内容到stutmp中
      {
	printf("read error on:%s\n",fileA); 
	return res;
      }
      if(-1 == write(fb,&stutmp,sizeof(ST))) //将stutmp中的内容写到文件B中
      {
	printf("write error on:%s\n",fileB); 
	return res;
      }
    }  
    lseek(fa,sizeof(ST)-sizeof(int),SEEK_CUR); //定位到下一个分数的部分
  }

  close(fa);
  close(fb);
  res=0;
  return res;
}
~~~

> **Note:** 文件打开数是一种系统资源，是有上限的，虽然程序退出后，系统会帮忙清理，但在程序设计中，打开文件，使用完后进行手动关闭是一种很好的习惯，这样可以有效避免缓存未刷新的潜在隐患，也可以更加节约资源





### 编译执行


~~~
emacs@ubuntu:~/c$ ll f1
ls: 无法访问f1: 没有那个文件或目录
emacs@ubuntu:~/c$ ll f2
ls: 无法访问f2: 没有那个文件或目录
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc savetofile.x savetofile.c
emacs@ubuntu:~/c$ ./savetofile.x 
emacs@ubuntu:~/c$ ll f1
-rw-r--r-- 1 emacs emacs 80 2016-12-30 04:08 f1
emacs@ubuntu:~/c$ ll f2
-rw-r--r-- 1 emacs emacs 32 2016-12-30 04:08 f2
emacs@ubuntu:~/c$ xxd f1
0000000: 0b00 0000 7869 616f 0000 0000 1200 0000  ....xiao........
0000010: 0d00 0000 686f 6e67 0000 0000 5800 0000  ....hong....X...
0000020: 1000 0000 7475 6e61 0000 0000 6200 0000  ....tuna....b...
0000030: 1300 0000 746f 6e79 0000 0000 3c00 0000  ....tony....<...
0000040: 5a00 0000 6475 6e6f 0000 0000 6200 0000  Z...duno....b...
emacs@ubuntu:~/c$ xxd f2
0000000: 1000 0000 7475 6e61 0000 0000 6200 0000  ....tuna....b...
0000010: 5a00 0000 6475 6e6f 0000 0000 6200 0000  Z...duno....b...
emacs@ubuntu:~/c$ 
~~~

编译执行过程中没有报错，从结果来看，f1、f2文件中的内容变化也符合预期

---

## 小技巧

### 宏定义

在写代码的过程偶尔会用到一些宏，这些宏多定义在头文件中，通过查看头文件，就可以获取相关信息

如我们想知道 **O_RDWR** 的定义

~~~
emacs@ubuntu:~$ grep O_RDWR  /usr/include/* -r
/usr/include/asm-generic/fcntl.h:#define O_RDWR		00000002
/usr/include/bits/fcntl.h:#define O_RDWR		     02
/usr/include/linux/fs.h: * to O_WRONLY and O_RDWR via the strange trick in __dentry_open()
/usr/include/linux/smbno.h:#define SMB_O_RDWR	0x0002
emacs@ubuntu:~$
~~~


我们想知道 **`SEEK_SET、SEEK_CUR、SEEK_END`** 的宏定义



~~~
emacs@ubuntu:~$ grep SEEK_SET  /usr/include/* 
/usr/include/fcntl.h:# define SEEK_SET	0	/* Seek from beginning of file.  */
/usr/include/libio.h:   beginning of the file (if W is SEEK_SET),
/usr/include/stdio.h:#define SEEK_SET	0	/* Seek from beginning of file.  */
/usr/include/unistd.h:# define SEEK_SET	0	/* Seek from beginning of file.  */
/usr/include/unistd.h:# define L_SET		SEEK_SET
/usr/include/unistd.h:   beginning of the file (if WHENCE is SEEK_SET),
emacs@ubuntu:~$ grep SEEK_CUR  /usr/include/* 
/usr/include/fcntl.h:# define SEEK_CUR	1	/* Seek from current position.  */
/usr/include/libio.h:   the current position (if W is SEEK_CUR),
/usr/include/stdio.h:#define SEEK_CUR	1	/* Seek from current position.  */
/usr/include/unistd.h:# define SEEK_CUR	1	/* Seek from current position.  */
/usr/include/unistd.h:# define L_INCR		SEEK_CUR
/usr/include/unistd.h:   the current position (if WHENCE is SEEK_CUR),
emacs@ubuntu:~$ grep SEEK_END  /usr/include/* 
/usr/include/fcntl.h:# define SEEK_END	2	/* Seek from end of file.  */
/usr/include/libio.h:   or the end of the file (if W is SEEK_END).
/usr/include/stdio.h:#define SEEK_END	2	/* Seek from end of file.  */
/usr/include/unistd.h:# define SEEK_END	2	/* Seek from end of file.  */
/usr/include/unistd.h:# define L_XTND		SEEK_END
/usr/include/unistd.h:   or the end of the file (if WHENCE is SEEK_END).
emacs@ubuntu:~$ 
~~~

我们还可以使用这种方式来查看函数原型

如我们想知道 **`lseek`** 函数的原型

~~~
emacs@ubuntu:~$ grep lseek  /usr/include/* 
/usr/include/_G_config.h:#define _G_LSEEK64	__lseek64
/usr/include/unistd.h:/* Values for the WHENCE argument to lseek.  */
/usr/include/unistd.h:extern __off_t lseek (int __fd, __off_t __offset, int __whence) __THROW;
/usr/include/unistd.h:extern __off64_t __REDIRECT_NTH (lseek,
/usr/include/unistd.h:				 lseek64);
/usr/include/unistd.h:#  define lseek lseek64
/usr/include/unistd.h:extern __off64_t lseek64 (int __fd, __off64_t __offset, int __whence)
emacs@ubuntu:~$
~~~


> **Tip:** 如果我们事先知道一个函数来自于哪一个头文件，就可以进一步地缩小范围，有时一个函数的头文件里并没有直接包含，可能是这个头文件所include的文件中包含，多时可能达到4到5层

---

## 内存对齐

在定义有结构体的代码中，要留意内存对齐的问题

哪什么是内存对齐呢，我们可以看看下面的一个例子：

~~~
#include <stdio.h>

int main()
{
  struct st1
  {
    char a;
    int b;
    char c;
  };

  struct st2
  {
    char a;
    char c;
    int b;
  };
  
  printf("size of st1:%d\nsize of st2:%d\n",sizeof(struct st1),sizeof(struct st2));
  return 0;
}
~~~

我们将它编译运行

~~~
emacs@ubuntu:~/c$ alias gtc
alias gtc='gcc -Wall -g -o'
emacs@ubuntu:~/c$ gtc duiqi.x duiqi.c 
emacs@ubuntu:~/c$ ./duiqi.x 
size of st1:12
size of st2:8
emacs@ubuntu:~/c$ 
~~~

从结果来看，包含同样内容的两个结构体，占用的内存却是不一样的，而区别只在于它们内部元素的排列方式不一样

**1-4-1** 的顺序占用了12个字节，**1-1-4** 的顺序占用了8个字节

这就是内存对齐的效果

在C语言中，结构是一种复合数据类型，其构成元素既可以是基本数据类型（如int、long、float等）的变量，也可以是一些复合数据类型（如数组、结构、联合等）的数据单元。在结构中，编译器为结构的每个成员按其自然边界（alignment）分配空间。各个成员按照它们被声明的顺序在内存中顺序存储，第一个成员的地址和整个结构的地址相同。

为了使CPU能够对变量进行快速的访问,变量的起始地址应该具有某些特性,即所谓的”对齐”.比如4字节的int型,其起始地址应该位于4字节的边界上,即起始地址能够被4整除

字节对齐的作用不仅是便于cpu快速访问，同时合理的利用字节对齐可以有效地节省存储空间。

对于32位机来说，4字节对齐能够使cpu访问速度提高，比如说一个long类型的变量，如果跨越了4字节边界存储，那么cpu要读取两次，这样效率就低了。但是在32位机中使用1字节或者2字节对齐，反而会使变量访问速度降低。所以这要考虑处理器类型，另外还得考虑编译器的类型。在vc中默认是4字节对齐的，GNU gcc 也是默认4字节对齐


---


## xxd

xxd是一个很好用的命令，可以用来查看二进制文件

~~~
emacs@ubuntu:~/c$ xxd f1
0000000: 0b00 0000 7869 616f 0000 0000 1200 0000  ....xiao........
0000010: 0d00 0000 686f 6e67 0000 0000 5800 0000  ....hong....X...
0000020: 1000 0000 7475 6e61 0000 0000 6200 0000  ....tuna....b...
0000030: 1300 0000 746f 6e79 0000 0000 3c00 0000  ....tony....<...
0000040: 5a00 0000 6475 6e6f 0000 0000 6200 0000  Z...duno....b...
emacs@ubuntu:~/c$ xxd f2
0000000: 1000 0000 7475 6e61 0000 0000 6200 0000  ....tuna....b...
0000010: 5a00 0000 6475 6e6f 0000 0000 6200 0000  Z...duno....b...
emacs@ubuntu:~/c$ 
~~~

结合前面的代码，从这个二进制编码里，我们可以看出很多有价值的信息

* 1.这是一个小端序的系统(数据的低字节保存在内存的低地址中)
* 2.每一个结构体占用了16字节
* 3.**0-3** 对应 **int** 的存储位置，**4-8** 对应 **char[5]** 的存储位置，**12-15** 对应 **int** 的存储位置
* 4.**9-11** 被空置了

这个命令将 ASCII 可显示的部分进行了显示，无法显示的都转化成了点

---

## ASCII 码

在Linux中使用man命令可以看到一份完整的ASCII码表

~~~
emacs@ubuntu:~/c$ man ascii
~~~

从中截取以下内容

~~~
       Oct   Dec   Hex   Char                        Oct   Dec   Hex   Char
       ────────────────────────────────────────────────────────────────────────
       000   0     00    NUL '\0'                    100   64    40    @
       001   1     01    SOH (start of heading)      101   65    41    A
       002   2     02    STX (start of text)         102   66    42    B
       003   3     03    ETX (end of text)           103   67    43    C
       004   4     04    EOT (end of transmission)   104   68    44    D
       005   5     05    ENQ (enquiry)               105   69    45    E
       006   6     06    ACK (acknowledge)           106   70    46    F
       007   7     07    BEL '\a' (bell)             107   71    47    G
       010   8     08    BS  '\b' (backspace)        110   72    48    H
       011   9     09    HT  '\t' (horizontal tab)   111   73    49    I
       012   10    0A    LF  '\n' (new line)         112   74    4A    J
       013   11    0B    VT  '\v' (vertical tab)     113   75    4B    K
       014   12    0C    FF  '\f' (form feed)        114   76    4C    L
       015   13    0D    CR  '\r' (carriage ret)     115   77    4D    M
       016   14    0E    SO  (shift out)             116   78    4E    N
       017   15    0F    SI  (shift in)              117   79    4F    O
       020   16    10    DLE (data link escape)      120   80    50    P
       021   17    11    DC1 (device control 1)      121   81    51    Q
       022   18    12    DC2 (device control 2)      122   82    52    R
       023   19    13    DC3 (device control 3)      123   83    53    S
       024   20    14    DC4 (device control 4)      124   84    54    T
       025   21    15    NAK (negative ack.)         125   85    55    U
       026   22    16    SYN (synchronous idle)      126   86    56    V
       027   23    17    ETB (end of trans. blk)     127   87    57    W
       030   24    18    CAN (cancel)                130   88    58    X
       031   25    19    EM  (end of medium)         131   89    59    Y
       032   26    1A    SUB (substitute)            132   90    5A    Z
       033   27    1B    ESC (escape)                133   91    5B    [
       034   28    1C    FS  (file separator)        134   92    5C    \  '\\'
       035   29    1D    GS  (group separator)       135   93    5D    ]
       036   30    1E    RS  (record separator)      136   94    5E    ^
       037   31    1F    US  (unit separator)        137   95    5F    _
       040   32    20    SPACE                       140   96    60    `
       041   33    21    !                           141   97    61    a
       042   34    22    "                           142   98    62    b
       043   35    23    #                           143   99    63    c
       044   36    24    $                           144   100   64    d
       045   37    25    %                           145   101   65    e
       046   38    26    &                           146   102   66    f
       047   39    27    ´                           147   103   67    g
       050   40    28    (                           150   104   68    h
       051   41    29    )                           151   105   69    i
       052   42    2A    *                           152   106   6A    j
       053   43    2B    +                           153   107   6B    k
       054   44    2C    ,                           154   108   6C    l
       055   45    2D    -                           155   109   6D    m
       056   46    2E    .                           156   110   6E    n
       057   47    2F    /                           157   111   6F    o
       060   48    30    0                           160   112   70    p
       061   49    31    1                           161   113   71    q
       062   50    32    2                           162   114   72    r
       063   51    33    3                           163   115   73    s
       064   52    34    4                           164   116   74    t
       065   53    35    5                           165   117   75    u
       066   54    36    6                           166   118   76    v
       067   55    37    7                           167   119   77    w
       070   56    38    8                           170   120   78    x
       071   57    39    9                           171   121   79    y
       072   58    3A    :                           172   122   7A    z
       073   59    3B    ;                           173   123   7B    {
       074   60    3C    <                           174   124   7C    |
       075   61    3D    =                           175   125   7D    }
       076   62    3E    >                           176   126   7E    ~
       077   63    3F    ?                           177   127   7F    DEL
~~~

其中的可见字符对应表

~~~
   Tables
       For convenience, let us give more compact  tables  in  hex
       and decimal.

          2 3 4 5 6 7       30 40 50 60 70 80 90 100 110 120
        -------------      ---------------------------------
       0:   0 @ P ` p     0:    (  2  <  F  P  Z  d   n   x
       1: ! 1 A Q a q     1:    )  3  =  G  Q  [  e   o   y
       2: " 2 B R b r     2:    *  4  >  H  R  \  f   p   z
       3: # 3 C S c s     3: !  +  5  ?  I  S  ]  g   q   {
       4: $ 4 D T d t     4: "  ,  6  @  J  T  ^  h   r   |
       5: % 5 E U e u     5: #  -  7  A  K  U  _  i   s   }
       6: & 6 F V f v     6: $  .  8  B  L  V  `  j   t   ~
       7: ´ 7 G W g w     7: %  /  9  C  M  W  a  k   u  DEL
       8: ( 8 H X h x     8: &  0  :  D  N  X  b  l   v
       9: ) 9 I Y i y     9: ´  1  ;  E  O  Y  c  m   w
       A: * : J Z j z
       B: + ; K [ k {
       C: , < L \ l |
       D: - = M ] m }
       E: . > N ^ n ~
       F: / ? O _ o DEL
~~~


---

# 总结

以下这些函数可以应对绝大部分的IO需求

* open
* close
* read
* write
* lseek


通过各方面资料弄懂其参数的意义和返回值的类型，是熟练掌握的基础


[c_stdio_01]:http://soft.dog/2016/12/26/c-stdio-01/
[c_fileio_01]:http://soft.dog/2016/12/30/c-unistd-fileio-01/
[linux_c_api]:http://www.kancloud.cn/wizardforcel/linux-c-api-ref/98469
