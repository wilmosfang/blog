---
layout: default
title: Linux CPU info
comments: true
---

---

前言
=

近期在一次收集系统信息的过程中涉及到cpu的部分有些范迷糊，特别是Linux中cpu的部分，cpuinfo里的东西太多,关于物理cpu、逻辑cpu，超线程如何看，网上看过一些贴子后，弄清楚了，在这里整理分享一下

---

判断方法 
-

Physical id 和 core id不一定是连续的

* 1.具有相同physical id的cpu在同一物理socket中
* 2.具有相同core id的cpu是在同一个core中的线程


> Physical id and core id are not necessarily consecutive but they are unique.
>
> * 1.Any cpu with the same physical id are threads or cores in the same physical socket.
>
> * 2.Any cpu with the same core id are hyperthreads in the same core.


---


#### 总物理核数 = 物理CPU个数 X 每颗物理CPU的核数 

#### 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数 = 物理CPU个数 X 每个物理CPU线程数 = 总线程数


![cpuinfo](https://raw.githubusercontent.com/wilmosfang/blog/gh-pages/images/cpuinfo/cpuinfo.png)


---

查看方法 
-

查看物理CPU个数

~~~
[root@Test ~]# cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
2
[root@Test ~]#
~~~

查看每个物理CPU中core的个数(即核数)

~~~
[root@Test ~]# cat /proc/cpuinfo| grep "cpu cores"| uniq
cpu cores	: 1
[root@Test ~]#
~~~

查看core的个数，同时查看一个core的线程数

~~~
[test@test ~]$ cat /proc/cpuinfo  | grep 'core id'| sort  | uniq -c  
      2 core id		: 0
      2 core id		: 1
      2 core id		: 2
      2 core id		: 3
      2 core id		: 4
      2 core id		: 5
[test@test ~]$ 
~~~

查看一个socket中的线程数

~~~
[test@test ~]$ cat /proc/cpuinfo| grep "siblings" | sort | uniq 
siblings	: 12
[test@test ~]$
~~~

查看是否启用超线程

>若是cpu cores数量和siblings数量一致，则没有启用超线程，不然超线程被启用，倍数就是超线程倍数

~~~
[test@test ~]$ cat /proc/cpuinfo | grep -e "cpu cores"  -e "siblings" | sort | uniq
cpu cores	: 6
siblings	: 12
[test@test ~]$ 
~~~

查看逻辑CPU的个数(超线程总数)

~~~
[root@Test ~]# cat /proc/cpuinfo| grep "processor"| wc -l 
2
[root@Test ~]#
~~~

查看CPU型号

~~~
[root@Test ~]# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c 
      2  QEMU Virtual CPU version (cpu64-rhel6)
[root@Test ~]# 
[test@test ~]# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c  
     12  Intel(R) Xeon(R) CPU E5-2620 0 @ 2.00GHz
[test@test ~]# 
[root@abc ~]# dmidecode -s processor-version 
Intel(R) Xeon(R) CPU E31225 @ 3.10GHz
[root@abc ~]# 
~~~


---

其它
-


查看内核信息

~~~
[root@Test ~]# uname -a 
Linux Test 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@Test ~]# 
~~~


查看版信息

>redhat和centos中有此命令

~~~
[root@Test ~]# lsb_release -a 
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID:	CentOS
Description:	CentOS release 6.6 (Final)
Release:	6.6
Codename:	Final
[root@Test ~]# 
~~~

查看版本信息

~~~
[root@Test ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@Test ~]# 
~~~

查看cpu运行模式

~~~
[root@Test ~]# getconf LONG_BIT
64
[root@Test ~]# 
~~~

64bit支持

~~~
[root@Test ~]# cat /proc/cpuinfo | grep flags | grep ' lm '
flags		: fpu de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pse36 clflush mmx fxsr sse sse2 syscall nx lm unfair_spinlock pni cx16 hypervisor lahf_lm
flags		: fpu de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pse36 clflush mmx fxsr sse sse2 syscall nx lm unfair_spinlock pni cx16 hypervisor lahf_lm
[root@Test ~]# 
~~~

cpu概要信息

~~~
[root@Test ~]# lscpu 
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             2
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 13
Stepping:              3
CPU MHz:               3092.972
BogoMIPS:              6185.94
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              4096K
NUMA node0 CPU(s):     0,1
[root@Test ~]# 
~~~

查看机器型号

~~~
[root@Test ~]# dmidecode | grep "Product Name" 
	Product Name: KVM
[root@Test ~]# 
[root@abc ~]#  dmidecode | grep "Product Name" 
	Product Name: HP Z210 Workstation
	Product Name: 1587h
[root@abc ~]# 
[root@test ~]# dmidecode | grep "Product Name" 
	Product Name: PowerEdge R620
	Product Name: 0D2D5F
[root@test ~]# 
~~~




---

总结
=

processor  逻辑处理器的唯一标识符。 

physical id  每个物理封装的唯一标识符。 

core id  每个内核的唯一标识符。 

siblings  位于相同物理封装中的逻辑处理器的数量。 

cpu cores  位于相同物理封装中的内核数量。


>拥有相同 physical id 的所有逻辑处理器共享同一个物理插座。每个 physical id 代表一个唯一的物理封装。Siblings 表示位于这一物理封装上的逻辑处理器的数量。它们可能支持也可能不支持超线程（HT）技术。每个 core id 均代表一个唯一的处理器内核。所有带有相同 core id 的逻辑处理器均位于同一个处理器内核上。如果有一个以上逻辑处理器拥有相同的 core id 和 physical id，则说明系统支持超线程（HT）技术。如果有两个或两个以上的逻辑处理器拥有相同的 physical id，但是 core id 不同，则说明这是一个多内核处理器。cpu cores 条目也可以表示是否支持多内核。


---

附
=

~~~
以上各项的含义如下：
processor　：体系中逻辑处理核的编号。对于单核处理器，则课认为是其CPU编号，对于多核处理惩罚器则可所以物理核、或者应用超线程技巧虚拟的逻辑核
vendor_id　：CPU制造商      
cpu family　：CPU产品系列代号
model　　　：CPU属于其系列中的哪一代的代号
model name：CPU属于的名字及其编号、标称主频
stepping　  ：CPU属于制造更新版本
cpu MHz　  ：CPU的实际应用主频
cache size   ：CPU二级缓存大小
physical id   ：单个CPU的标号
siblings       ：单个CPU逻辑物理核数
core id        ：当前物理核在其所处CPU中的编号，这个编号不必然连续
cpu cores    ：该逻辑核所处CPU的物理核数
apicid          ：用来区分不合逻辑核的编号，体系中每个逻辑核的此编号必定不合，此编号不必然连续
fpu             ：是否具有浮点运算单位（Floating Point Unit）
fpu_exception  ：是否支撑浮点策画异常
cpuid level   ：履行cpuid指令前，eax存放器中的值，按照不合的值cpuid指令会返回不合的内容
wp             ：注解当前CPU是否在内核态支撑对用户空间的写保护（Write Protection）
flags          ：当前CPU支撑的功能
bogomips   ：在体系内核启动时粗略测算的CPU速度（Million Instructions Per Second）
clflush size  ：每次刷新缓存的大小单位
cache_alignment ：缓存地址对齐单位
address sizes     ：可接见地址空间位数
power management ：对能源经管的支撑
CPU信息中flags各项含义：
fpu： Onboard （x87） Floating Point Unit
vme： Virtual Mode Extension
de： Debugging Extensions
pse： Page Size Extensions
tsc： Time Stamp Counter: support for RDTSC and WRTSC instructions
msr： Model-Specific Registers
pae： Physical Address Extensions: ability to access 64GB of memory; only 4GB can be accessed at a time though
mce： Machine Check Architecture
cx8： CMPXCHG8 instruction
apic： Onboard Advanced Programmable Interrupt Controller
sep： Sysenter/Sy***it Instructions; SYSENTER is used for jumps to kernel memory during system calls， and SY***IT is used for jumps： back to the user code
mtrr： Memory Type Range Registers
pge： Page Global Enable
mca： Machine Check Architecture
cmov： CMOV instruction
pat： Page Attribute Table
pse36： 36-bit Page Size Extensions: allows to map 4 MB pages into the first 64GB RAM， used with PSE.
pn： Processor Serial-Number; only available on Pentium 3
clflush： CLFLUSH instruction
dtes： Debug Trace Store
acpi： ACPI via MSR
mmx： MultiMedia Extension
fxsr： FXSAVE and FXSTOR instructions
sse： Streaming SIMD Extensions. Single instruction multiple data. Lets you do a bunch of the same operation on different pieces of input： in a single clock tick.
sse2： Streaming SIMD Extensions-2. More of the same.
selfsnoop： CPU self snoop
acc： Automatic Clock Control
IA64： IA-64 processor Itanium.
ht： HyperThreading. Introduces an imaginary second processor that doesn’t do much but lets you run threads in the same process a  bit quicker.
nx： No ute bit. Prevents arbitrary code running via buffer overflows.
pni： Prescott New Instructions aka. SSE3
vmx： Intel Vanderpool hardware virtualization technology
svm： AMD “Pacifica” hardware virtualization technology
lm： “Long Mode，” which means the chip supports the AMD64 instruction set
tm： “Thermal Monitor” Thermal throttling with IDLE instructions. Usually hardware controlled in response to CPU temperature.
tm2： “Thermal Monitor 2″ Decrease speed by reducing multipler and vcore.
est： “Enhanced SpeedStep”
~~~

