---
layout:  post
title:  LDD(一).helloworld
author:  wilmosfang
tags:   arm c ldd
categories:  ldd
wc: 193  431 7448
excerpt:  Linux Device Drivers，Linux 驱动开发，hello world demo，内核模块/驱动的开发安装与卸载
comments: true
---


# 前言


Linux 作为目前使用最为广泛的操作系统，关键在于其具备优异特性的内核

> **Tip:** Linux 其实就是指的内核，各种发行版本无非是在内核的基础之上打包了一套软件，多了这层包裹后，系统就变得更加易用，也可以更好的服务于特定领域

Linux 内核运行在单独的内核地址空间，是一种单内核的理念 (有时称之为宏内核 Macrokernel 或 Monolithickernel )，所有事情都运行在内核态，直接调用函数，无需消息传递，避免了IPC机制带来的额外开销，还避免了内核空间到用户空间的上下文切换，因而性能优异，同时在设计上又汲取了微内核(Microkernelkernel) 的精华：模块化设计、抢占式内核、支持内核线程以及动态装载内核模块的能力，从而在灵活性上又得以拓展

由于设计方法上同时汲取了单内核与微内核的思想，所以也有部分人称其为混合内核

![kernal.jpg](/images/kernal.jpg)

Linux 作为单内核保证性能的同时还能兼具动态加载卸载模块的特性给我的印象最深刻

这里通过简单的一个例子来深入到 linux 内核的里面，看看 linux 内核模块的开发，加载，卸载等相关基础

---

# 概要

* TOC
{:toc}



---



## 代码示例

实现一个加载和卸载时打印消息的内核模块


**`hello.c`**

~~~
#include <linux/module.h> 	//printk,KERN_INFO 等在此文件中申明和定义

MODULE_LICENSE ("GPL"); 	//这条不能少，如果少了，编译的过程不会报错，但是加载模块的过程会报license问题

int init_module (void)   	//insmod 过程中此模块执行的函数
{
  printk (KERN_INFO "Hello world\n"); //打印输出，并且返回
  return 0;
}

void cleanup_module (void) 	//rmmod 过程中此模块执行的函数
{
	  printk (KERN_INFO "Goodbye world\n"); //打印输出
}
~~~

> **Note:** 如果不加  `include <linux/module.h>`  会有如下报错

~~~
emacs@ubuntu:~/driver/ex0_hello$ make 
make -C /opt/linux-2.6.32.10/ M=/home/emacs/driver/ex0_hello modules
make[1]: 正在进入目录 `/opt/linux-2.6.32.10'
  CC [M]  /home/emacs/driver/ex0_hello/hello.o
/home/emacs/driver/ex0_hello/hello.c:3: error: expected declaration specifiers or '...' before string constant
/home/emacs/driver/ex0_hello/hello.c:3: warning: data definition has no type or storage class
/home/emacs/driver/ex0_hello/hello.c:3: warning: type defaults to 'int' in declaration of 'MODULE_LICENSE'
/home/emacs/driver/ex0_hello/hello.c:3: warning: function declaration isn't a prototype
/home/emacs/driver/ex0_hello/hello.c: In function 'init_module':
/home/emacs/driver/ex0_hello/hello.c:7: error: implicit declaration of function 'printk'
/home/emacs/driver/ex0_hello/hello.c:7: error: 'KERN_INFO' undeclared (first use in this function)
/home/emacs/driver/ex0_hello/hello.c:7: error: (Each undeclared identifier is reported only once
/home/emacs/driver/ex0_hello/hello.c:7: error: for each function it appears in.)
/home/emacs/driver/ex0_hello/hello.c:7: error: expected ')' before string constant
/home/emacs/driver/ex0_hello/hello.c: In function 'cleanup_module':
/home/emacs/driver/ex0_hello/hello.c:13: error: 'KERN_INFO' undeclared (first use in this function)
/home/emacs/driver/ex0_hello/hello.c:13: error: expected ')' before string constant
make[2]: *** [/home/emacs/driver/ex0_hello/hello.o] 错误 1
make[1]: *** [_module_/home/emacs/driver/ex0_hello] 错误 2
make[1]:正在离开目录 `/opt/linux-2.6.32.10'
make: *** [modules] 错误 2
emacs@ubuntu:~/driver/ex0_hello$
~~~


> **Note:** 如果不加 MODULE_LICENSE 宏来定义授权，在加载模块的过程中会有如下报错


~~~
# insmod hello.ko 
hello: module license 'unspecified' taints kernel.
Disabling lock debugging due to kernel taint
Hello world
#
~~~


**`Makefile`** 

~~~
ifeq ($(KERNELRELEASE),)  #第一次KERNELRELEASE并没有定义，所以直接执行下面的代码，来确定内核目录，和保存当前工作目录

KERNELDIR ?= /opt/linux-2.6.32.10/ #定义内核目录 ?= 的意义是如果KERNELDIR没有被定义过，那么KERNELDIR的值就是/opt/linux-2.6.32.10/,如果KERNELDIR先前被定义过了，那么这条语句将什么都不做

PWD := $(shell pwd) #保存当前的目录到PWD中

modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules 

modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:  #执行make clean 时要触发的操作
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions *.order *.symvers

.PHONY: modules modules_install clean

else	#第二次KERNELRELEASE已经定义，所以直接进行下面的操作来合成模块
    obj-m := hello.o #这一步完成了最核心的步骤，表明有一个模块要从目标文件hello.o建立，目标文件建立后结果模块命名为hello.ko
endif
~~~


---

## 编译执行

~~~
emacs@ubuntu:~/driver/ex0_hello$ make 
make -C /opt/linux-2.6.32.10/ M=/home/emacs/driver/ex0_hello modules
make[1]: 正在进入目录 `/opt/linux-2.6.32.10'
  CC [M]  /home/emacs/driver/ex0_hello/hello.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/emacs/driver/ex0_hello/hello.mod.o
  LD [M]  /home/emacs/driver/ex0_hello/hello.ko
make[1]:正在离开目录 `/opt/linux-2.6.32.10'
emacs@ubuntu:~/driver/ex0_hello$ echo $?
0
emacs@ubuntu:~/driver/ex0_hello$
~~~

---

## 安装


~~~
# ls
hello.ko
# lsmod 
# insmod hello.ko 
Hello world
#
~~~

## 卸载

~~~
# lsmod 
hello 594 0 - Live 0xbf006000
# rmmod hello    
Goodbye world
rmmod: module 'hello' not found
# lsmod 
# ls
hello.ko
# 
~~~

---

## Linux 的一些内核特性

* Linux支持动态加载内核模块: 尽管Linux内核也是单内核，可是允许在需要的时候动态地卸除和加载部分内核代码
* Linux支持对称多处理（SMP）机制: 尽管许多Unix的变体也支持SMP，但传统的Unix并不支持这种机制
* Linux内核可以抢占（preemptive）: 与传统的Unix不同，Linux内核具有允许在内核运行的任务优先执行的能力。在其他各种Unix产品中，只有Solaris和IRIX支持抢占，但是大多数传统的Unix内核不支持抢占
* Linux对线程支持的实现比较有意思, 内核并不区分线程和其他的一般进程。对于内核来说，所有的进程都一样—只不过其中的一些共享资源而已
* Linux提供具有设备类的面向对象的设备模型、热插拔事件，以及用户空间的设备文件系统（sysfs）
* Linux忽略了一些被认为是设计得很拙劣的Unix特性，像STREAMS，它还忽略了那些实际上已经根本不会使用的过时标准
* Linux体现了自由这个词的精髓

> **Tip:** 现有的 Linux 特性集就是 Linux 公开开发模型自由发展的结果。如果一个特性没有任何价值或者创意很差，没有任何人会被迫去实现它。相反的，在 Linux 的发展过程中已经形成了一种值得称赞的务实态度：任何改变都要针对现实中确实存在的问题，经过完善的设计并有正确简洁的实现。于是，许多其他现代 Unix 系统包含的特性，如内核换页机制，都被毫不迟疑的引入进来

