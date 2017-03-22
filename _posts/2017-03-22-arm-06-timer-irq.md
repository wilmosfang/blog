---
layout:  post
title:  ARM(六).TIMER and IRQ
author:  wilmosfang
tags:   arm c
categories:  arm
wc: 1670  7739 79499  
excerpt:  ARM裸机开发、定时器与中断
comments: true
---


# 前言

**[ARM][arm]** 处理器是英国 Acorn 有限公司设计的低功耗低成本的一款 RISC 微处理器

> **ARM** 全称为 **Acorn RISC Machine**

因为价格与能耗上的明显优势，在手持设备与嵌入式领域有大规模的应用，可以说目前的绝大部分便携或手持电子消费品都是用的 ARM 芯片

前面一篇简单地对 ARM 裸机开发、平台环境与工具、创建项目、按键中断的控制与基础进行了演示

这里再对定时器与中断进行探究

---

# 概要

* TOC
{:toc}

---


## 定时器和中断

### 要求

* 使用 ARM 板上的定时器结合中断对蜂鸣器进行控制

#### Timer 电路原理图

![timer.jpg](/images/timer/timer.jpg)

#### Buzzer 电路原理图

![buzzer.png](/images/tq2440/buzzer.png)


### 创建项目

创建项目的总体过程就是

* 新建文件夹
* 创建项目文件
* 项目中添加入源代码

只有以下几个方面需要稍微注意一下

#### 选择三星 S3C2440A芯片

**Device** 选项卡中确保是正确的设备选型(和头文件相关，寄存器的正确地址决定于此)

![S3C2440A.png](/images/arm_tq/S3C2440A.png)

#### 设定时钟频率和栈大小

**Target** 选项卡中确保时钟频率和板载一致

正确设定内存(只读栈和读写栈，也就是代码区与数据区的大小)

![memory.png](/images/arm_tq/memory.png)

#### 选择H-JTAG ARM 模式

选择正确的模式

![HJTAG_ARM.png](/images/arm_tq/HJTAG_ARM.png)


#### 使用外部工具

![External_tool.png](/images/arm_tq/External_tool.png)


### 代码示例


**`timer_irq.s`**

这是主汇编程序,定义了中断向量表,进行了各种初始化

~~~ini
    	GET		s3c2410_SFR.s		;GET伪指令将s3c2410_SFR.s包含到此文件中,s3c2410_SFR.s是寄存器地址的宏定义
	GET		startup_head.s		;GET伪指令将startup_head.s包含到此文件中,startup_head.s是初始化配置

	IMPORT  kain				;IMPORT伪指令指示编译器当前的符号不是在本源文件中定义的,而是在其它源文件中定义的,在本源文件中可能引用该符号,Main定义在c源文件中
	IMPORT	Handle_Timer0	 		;Handle_EINT0定义在c源文件中
	
	AREA   	RESET, CODE, READONLY		;定义一个名为RESET的只读代码段
	CODE32					;CODE32伪指令指示汇编编译器后面的指令为32位的ARM指令
	ENTRY					;ENTRY伪指令用于指定程序的入口点,一个程序(可以包含多个源文件)中至少要有一个ENTRY,可以有多个ENTRY，但一个源文件中最多只有一个ENTRY

_Startup					;这只是一个普通标号
	EXPORT	VectorBase			;EXPORT声明一个符号VectorBase可以被其它文件引用

VectorBase					;向量基址,下面是自定义的向量集
	B		HandlerReset		;直接跳转到HandlerReset处进行处理
	LDR		PC, (Vect_Table + 4)	;将(Vect_Table + 4)中的地址加载到PC中,也就是跳转到(Vect_Table + 4)中的地址处进行处理
	LDR		PC, (Vect_Table + 8)	;SWI,将(Vect_Table + 8)中的地址加载到PC中,也就是跳转到(Vect_Table + 8)中的地址处进行处理
	LDR		PC, (Vect_Table +12)	;Prefetch Abort,将(Vect_Table + 12)中的地址加载到PC中,也就是跳转到(Vect_Table + 12)中的地址处进行处理
	LDR		PC, (Vect_Table +16)	;Data Abort,将(Vect_Table + 16)中的地址加载到PC中,也就是跳转到(Vect_Table + 16)中的地址处进行处理
	B		.			;Not Assigned,什么事也不做,相当于while(1)
	LDR		PC, (Vect_Table +24)	;IRQ,将(Vect_Table + 24)中的地址加载到PC中,也就是跳转到(Vect_Table + 24)中的地址处进行处理 
	LDR		PC, (Vect_Table +28)	;FIQ,将(Vect_Table + 28)中的地址加载到PC中,也就是跳转到(Vect_Table + 28)中的地址处进行处理 

Vect_Table					;中断向量表,DCD用于分配一片连续的字(4个字节)存储单元并用指定的数据初始化(有点像int型数组)
	DCD		HandlerReset		;相当于(Vect_Table + 0),并且将HandlerReset的值加载到其中
	DCD		HandlerUndef		;相当于(Vect_Table + 4),并且将HandlerUndef的值加载到其中
	DCD		HandlerSWI		;相当于(Vect_Table + 8),并且将HandlerSWI的值加载到其中
	DCD		HandlerPabort		;相当于(Vect_Table + 12),并且将HandlerPabort的值加载到其中
	DCD		HandlerDabort		;相当于(Vect_Table + 16),并且将HandlerDabort的值加载到其中
	DCD		.			;相当于(Vect_Table + 20),并且将.的值加载到其中  
	DCD	    IRQ_Handler			;相当于(Vect_Table + 24),并且将IRQ_Handler的值加载到其中
	DCD		HandlerFIQ		;相当于(Vect_Table + 28),并且将HandlerFIQ的值加载到其中
	DCD		kain			;相当于(Vect_Table + 32),并且将Main的值加载到其中
		
	EXPORT VectorEnd			;EXPORT声明一个符号VectorEnd可以被其它文件引用
VectorEnd	
	LTORG					;声明文字池保存以上向量表(这条命令的实际效用还是有点不是很清楚)

	AREA   	RESET, CODE, READONLY		;定义一个名为RESET的只读代码段
	CODE32
HandlerReset					;定义一个HandlerReset标签(指代了此处的地址)
;/***************************************/
;/* disable interrupt                   */
;/***************************************/
    MRS 	R0, cpsr    			;将状态寄存器cpsr中的值读到R0中
    ORR 	R0, R0, #0xc0			;将R0与(1100 0000)进行或操作,结果放到R0中,这个过程其实是保持其它位不变，将第6(FIQ)位和7(IRQ)位置1，就是禁止所有中断
    MSR 	cpsr_c, R0			;将R0重新存回,也就是关闭了所有中断

;/***************************************/
;/* disable watchdog                    */
;/***************************************/
	LDR		R0, =WTCON		;看门狗配置寄存器地址加载到R0中
	LDR		R1, =0x0         	;将0加载到R1中
	STR		R1, [R0]		;将看门狗配置寄存器中的值置0,也就是关闭看门狗

;/****************************************/
;/* config interrupt                     */
;/****************************************/
	LDR		R0, =INTMSK		;中断配置寄存器地址加载到R0中
	LDR		R1, =0xFFFFFFFF         ;将全1加载到R1中
	STR		R1, [R0]		;将全1加载到中断配置寄存器中，让所有中断屏蔽掉

	LDR		R0, =INTSUBMSK		;子中断配置寄存器地址加载到R0中
	LDR		R1, =0x000007FF		;将(0111 1111 1111)加载到R1中
	STR		R1, [R0]		;将全(0111 1111 1111)加载到子中断配置寄存器中，让所有子中断屏蔽掉

	LDR     R0, =INTPND                 	;中断未决寄存器地址加载到R0中
	LDR     R1, =0xFFFFFFFF			;将全1加载到R1中
	STR     R1, [R0]			;通过写1的方式来清理中断未决寄存器
	
	LDR     R0, =SRCPND                 	;源未决寄存器地址加载到R0中
	LDR     R1, =0xFFFFFFFF			;将全1加载到R1中
	STR     R1, [R0]			;通过写1的方式来清理源未决寄存器

;/****************************************/
;/* config pll                           */
;/****************************************/
	LDR		R0, =LOCKTIME		;锁定时间计数寄存器地址加载到R0中
	LDR		R1, =0x00FFFFFF		;将0x00FFFFFF加载到R1中
	STR		R1, [R0]		;将R1加载到锁定时间计数寄存器中(U_LTIME 为0x00FF,M_LTIME 为0xFFFF)

	LDR		R0, =CLKDIVN	       	;时钟分频控制寄存器地址加载到R0中
	LDR		R1, =((M_HDIVN << 1) | M_PDIVN)	;将((M_HDIVN << 1) | M_PDIVN)加载到R1中，M_HDIVN 和 M_PDIVN 定义在另一个头文件中
	STR		R1, [R0]		;进行配置

;/****************************************/
;/* config mmu                           */
;/****************************************/
	MRC  	p15, 0, R0, c1, c0, 0
	ORR		R0, R0, #R1_nF:OR:R1_iA
	MCR		p15,0,R0,c1,c0,0
	
	MRC		p15, 0, R0, c1, c0, 0
	BIC		R0, R0, #R1_M
	MCR		p15, 0, R0, c1, c0, 0
	
	MRC		p15, 0, R0, c1, c0, 0
	ORR		R0, R0, #R1_I
	MCR		p15, 0, R0, c1, c0, 0
	
	MRC		p15, 0, R0, c1, c0, 0
	ORR		R0, R0, #R1_C
	MCR		p15, 0, R0, c1, c0, 0

;/****************************************/
;/* config pll                           */
;/****************************************/
	LDR		R0, =CLKCON		;时钟发生器控制寄存器地址加载到R0中
	LDR		R1, =0x0007FFF0		;将0x0007FFF0加载到R1中，相应的位置1，就是设定有效，哪些外设要进行有效处理，得查文档
	STR		R1, [R0]		;进行设定

	LDR		R0, =CLKSLOW		;减慢时钟控制寄存器地址加载到R0中
	LDR		R1, =0x00000004		;将0x00000004加载到R1中
	STR		R1, [R0]		;进行设定

	LDR     R0, =UPLLCON  			;UPLL配置寄存器地址加载到R0中 ,USB的PLL就在此配置
	LDR     R1, =((U_MDIV << 12) + (U_PDIV << 4) + U_SDIV) 	;将((U_MDIV << 12) + (U_PDIV << 4) + U_SDIV)加载到R1中 ;Fin=12MHz, Fout=48MHz
	STR     R1, [R0]			;进行设定

	NOP					;NOP为空操作伪指令，NOP伪指令在汇编时将会被代替成ARM中的空操作，比如 MOV R0,R0
	NOP
	NOP
	NOP
	NOP
	NOP
	NOP

	LDR		R0, =MPLLCON        	;MPLL配置寄存器地址加载到R0中
	LDR		R1, =((M_MDIV << 12) + (M_PDIV << 4) + M_SDIV) ;将((M_MDIV << 12) + (M_PDIV << 4) + M_SDIV)加载到R1中
	STR		R1, [R0]		;进行设定

	NOP
	NOP

;/****************************************/
;/* config stack                         */
;/****************************************/
    MSR     CPSR_c, #0x0d2			;将(1101 0010)加载到CPSR_c中(代表禁止所有中断,使用ARM模式,进入中断模式)
    LDR     SP, =IRQStack_BASE			;IRQStack_BASE在另一个文件中定义,将中断模式中的堆栈指针SP指到IRQStack_BASE处

    MSR     CPSR_c, #0x05f			;将(0101 1111)加载到CPSR_c中(代表开启IRQ中断禁止FIQ中断,使用ARM模式,进入系统模式)
    LDR     SP, =UsrStack_BASE			;UsrStack_BASE在另一个文件中定义,将系统模式中的堆栈指针SP指到UsrStack_BASE处

;/***************************************/
;/* enable interrupt                    */
;/***************************************/
	MRS 	R0, cpsr    			;cpsr加载到R0中
    BIC 	R0, R0, #0x80			;BIC将R0中的第7位置0,(将R0跟(1000 0000)的反码(0111 1111)进行与操作,就是对第7位清零)
    MSR 	cpsr_c, R0			;将R0结果保存回cpsr_c中

;/****************************************/
;/* go to c main                         */
;/****************************************/
	LDR		PC, (Vect_Table + 32)	;这里进行跳转，相当于 goto main(由此可知c语言中的main函数之所以叫main,也是类似这样的地方定义的,如果取别的名字比如xxx,那C的代码就都会从xxx函数开始执行)
	NOP
	NOP
	NOP
	NOP

;/****************************************/
;/* Undefined Instruction interrupt entry*/
;/****************************************/
HandlerUndef					;定义一个HandlerUndef,并且啥也不干
	B		.

;/****************************************/
;/* SWI interrupt entry                  */
;/****************************************/
HandlerSWI					;定义一个HandlerSWI,并且啥也不干
	B		.

;/****************************************/
;/* Prefetch Abort interrupt entry       */
;/****************************************/
HandlerPabort					;定义一个HandlerPabort,并且啥也不干
	B		.

;/****************************************/
;/* Data Abort interrupt entry           */
;/****************************************/
HandlerDabort					;定义一个HandlerDabort,并且啥也不干
	B		.
	
;/****************************************/
;/* FIQ interrupt entry                  */
;/****************************************/
HandlerFIQ					;定义一个HandlerFIQ,并且啥也不干
	B		.

;/****************************************/
;/* default irq entry                    */
;/****************************************/
	EXPORT  Default_IRQ_ISR			;EXPORT声明一个符号Default_IRQ_ISR可以被其它文件引用
Default_IRQ_ISR					;定义一个Default_IRQ_ISR,并且啥也不干
	B       .

	PRESERVE8 				;保证堆栈8字节对齐

IRQ_Vecotr                            		;中断向量表
EINT0_Handle       B   Default_IRQ_ISR 		;B   Default_IRQ_ISR代表啥都不干(因为上面对Default_IRQ_ISR中的操作定义就是啥都没干)
EINT1_Handle       B   Default_IRQ_ISR
EINT2_Handle       B   Default_IRQ_ISR
EINT3_Handle       B   Default_IRQ_ISR		
EINT4_7_Handle     B   Default_IRQ_ISR
EINT8_23_Handle    B   Default_IRQ_ISR
CAM_Handle         B   Default_IRQ_ISR
BATFLT_Handle      B   Default_IRQ_ISR
TICK_Handle        B   Default_IRQ_ISR
WDT_AC97_Handle    B   Default_IRQ_ISR
ISR_TIMER0_Handle  B   Handle_Timer0		;收到ISR_TIMER0_Handle中断会跳转到Handle_Timer0进行处理
ISR_TIMER1_Handle  B   Default_IRQ_ISR
ISR_TIMER2_Handle  B   Default_IRQ_ISR
ISR_TIMER3_Handle  B   Default_IRQ_ISR
ISR_TIMER4_Handle  B   Default_IRQ_ISR
ISR_UART2_Handle   B   Default_IRQ_ISR
ISR_LCD_Handle     B   Default_IRQ_ISR
ISR_DMA0_Handle    B   Default_IRQ_ISR
ISR_DMA1_Handle    B   Default_IRQ_ISR
ISR_DMA2_Handle    B   Default_IRQ_ISR
ISR_DMA3_Handle    B   Default_IRQ_ISR
ISR_SDI_Handle     B   Default_IRQ_ISR
ISR_SPI0_Handle    B   Default_IRQ_ISR
ISR_UART1_Handle   B   Default_IRQ_ISR
ISR_NFCON_Handle   B   Default_IRQ_ISR
ISR_USBD_Handle    B   Default_IRQ_ISR
ISR_USBH_Handle    B   Default_IRQ_ISR
ISR_IIC_Handle     B   Default_IRQ_ISR
ISR_UART0_Handle   B   Default_IRQ_ISR		
ISR_SPI1_Handle    B   Default_IRQ_ISR
ISR_RTC_Handle     B   Default_IRQ_ISR
ISR_ADC_Handle     B   Default_IRQ_ISR

IRQ_Handler     PROC
	EXPORT  IRQ_Handler          [WEAK] 	;EXPORT声明一个符号IRQ_Handler可以被其它文件引用，[WEAK] 指定该选项后，如果symbol在所有的源程序中都没有被定义，编译器也不会产生任何错误信息，同时编译器也不会到当前没有被INCLUDE进来的库中去查找该标号

    SUB		LR, LR, #4			;LR连接寄存器(Link Register, LR)，在ARM体系结构中LR的特殊用途有两种：一是用来保存子程序返回地址；二是当异常发生时，LR中保存的值等于异常发生时PC的值减4(或者减2),因此在各种异常模式下可以根据LR的值返回到异常发生前的相应位置继续执行
    STMFD	SP!, {R0-R12,LR}		;保护现场,将{R0-R12,LR}作压栈处理,顺序是寄存器从大到小，SP!意思是每次操作完将SP更新的值还是存回SP
    LDR		R0, =INTOFFSET			;将中断偏移寄存器的地址存到R0中
    LDR		R0, [R0]			;将R0中地址(中断偏移寄存器地址)所代表的寄存器的值存到R0中
    LDR		R1, =IRQ_Vecotr			;将中断向量表的基址存到R1中
    ADD		R1, R1, R0, LSL #2		;将R0逻辑左移2位,加上R1,结果放到R1中,其实就是R1=R1+R0*4,为什么要乘4呢,因为向量表是4字节对其的,所以结果就是相应中断跳转的位置
    LDR		LR, =int_return			;LR中保存int_return作为返回地址
    MOV		PC, R1				;将R1的值(中断入口地址)保存到PC中,即相当于直接跳转到中断处,开始执行中断服务程序

int_return					;返回地址
    LDMFD	SP!,{R0-R12, PC}^		;进行现场恢复,将之前压栈的环境变量从堆栈中读出,覆盖到当前的寄存器中,在LDM指令的寄存器列表中包含有PC时使用'^',那么除了正常的多寄存器传送外,将SPSR拷贝到CPSR中,这可用于异常处理返回,使用'^'后缀进行数据传送且寄存器列表不包含PC时,加载/存储的是用户模式的寄存器,而不是当前模式的寄存器
	
    ENDP

    END
~~~


**`main.c`**

主 c 程序中定义了中断处理程序


~~~c
#include "2440addr.h"	//将"2440addr.h"包含进来,这里面放的是所有寄存器的地址宏

int i;			//定义一个全局的计数器,用来数中断个数

void Timer0Init(void)	//timer0的初始化程序
{
    rTCFG0  = 124;	//TCFG0(定时器配置寄存器0) 设定timer0预标定器值为124,定时器0和1共享一个8位的预分频器(预定标器),定时器2,3,4共享一个8位预分频器(预定标器)
    rTCFG1  = 0x01;	//(0000 0001)TCFG1(定时器配置寄存器1) 设定timer0的分频值为 1/4分频,每个定时器有一个时钟分频器,其可以生成5种不同的分频信号(1/2,1/4,1/8,1/16和TCLK)  
			//定时器输出时钟频率 = PCLK/{prescaler + 1}/{divider}
			//timer0 = PCLK/(124+1)/4 = 101.25MHz/125/4 = 202.5KHz
    rTCNTB0 = 20250;	//timer0定时器计数缓存寄存器设定值为20250(这个值不能超过65535) 20250*1/202.5KHz = 0.1s =100ms 
                                  
	rINTMSK &= ~(BIT_TIMER0);  //打开timer0中断,相当于EnableIrq(BIT_TIMER0);  
	 
	rTCON   = (1<<3)|(0<<2)|(1<<1)|(0<<0); //TCON定时器控制寄存器, timer0 启用 auto reload, 禁用反相器, 手动加载一次 TCNTB 和 TCMPB ,禁用timer0
	rTCON   = (1<<3)|(0<<2)|(0<<1)|(1<<0); //TCON定时器控制寄存器, timer0 启用 auto reload, 禁用反相器, 不用手动加载 TCNTB 和 TCMPB(前面一个操作手动加载了) ,开启timer0


}


void BuzzerInit(void)		//蜂鸣器的初始化函数
{
	rGPBCON = (1 << 0);	//buzzer连接到GPB0,先将其配置为输出模式(01)
	rGPBDAT = (0 << 0);	//将GPB0设定为低电位,关闭蜂鸣器
	i = 0;			//将i置0
}


void Handle_Timer0(void)	// Handle_Timer0中断服务程序
{
	i++;			//i++ ,计数器加1
	if(i==10)		//如果i的值为10(触发了10次timer0中断)
	{
		rGPBDAT = rGPBDAT ^ 0x01;	//翻转GPB0电位(如果蜂鸣器响,则变为不响,如果蜂鸣器不响,则变得响)
		i=0;		//将i置0
	}
	ClearPending(BIT_TIMER0);  	//清理TIMER0中断
}

int kain()			//主函数,这里是kain,而不是main,故意为之,以展示入口函数的自定义性
{
	BuzzerInit();		//蜂鸣器的初始化
	Timer0Init();		//timer0的初始化

	while(1);		//无限循环
}
~~~


### 编译执行

**`[Build]->[Debug]->[Run]`**

编译执行过程中没有报错，从结果来看，符合预期

开发板会每隔一秒响一次，一次持续一秒钟

---

# 附.头文件


**`s3c2410_SFR.s`**

这个文件作为头文件, 定义了 s3c2410 特殊功能寄存器的宏

~~~ini
;********************************************************************************************************
;* 文件: S3C2410SFR.S
;* 描述: s3c2410 特殊功能寄存器定义.(special function register)
;********************************************************************************************************/

        GBLL   BIG_ENDIAN__
BIG_ENDIAN__   SETL   {FALSE}

;=================
; Memory control 
;=================
BWSCON      EQU  0x48000000	;Bus width & wait status
BANKCON0    EQU  0x48000004	;Boot ROM control
BANKCON1    EQU  0x48000008	;BANK1 control
BANKCON2    EQU  0x4800000c	;BANK2 cControl
BANKCON3    EQU  0x48000010	;BANK3 control
BANKCON4    EQU  0x48000014	;BANK4 control
BANKCON5    EQU  0x48000018	;BANK5 control
BANKCON6    EQU  0x4800001c	;BANK6 control
BANKCON7    EQU  0x48000020	;BANK7 control
REFRESH     EQU  0x48000024	;DRAM/SDRAM refresh
BANKSIZE    EQU  0x48000028	;Flexible Bank Size
MRSRB6      EQU  0x4800002c	;Mode register set for SDRAM
MRSRB7      EQU  0x48000030	;Mode register set for SDRAM

;=================
; USB Host
;=================

;=================
; INTERRUPT
;=================
SRCPND       EQU  0x4a000000	;Interrupt request status
INTMOD       EQU  0x4a000004	;Interrupt mode control
INTMSK       EQU  0x4a000008	;Interrupt mask control
PRIORITY     EQU  0x4a00000c	;IRQ priority control   -- May 06, 2002 SOP
INTPND       EQU  0x4a000010	;Interrupt request status
INTOFFSET    EQU  0x4a000014	;Interruot request source offset
SUSSRCPND    EQU  0x4a000018	;Sub source pending
INTSUBMSK    EQU  0x4a00001c	;Interrupt sub mask


;=================
; DMA
;=================
DISRC0       EQU  0x4b000000	;DMA 0 Initial source
DISRCC0      EQU  0x4b000004	;DMA 0 Initial source control
DIDST0       EQU  0x4b000008	;DMA 0 Initial Destination
DIDSTC0      EQU  0x4b00000c	;DMA 0 Initial Destination control
DCON0        EQU  0x4b000010	;DMA 0 Control
DSTAT0       EQU  0x4b000014	;DMA 0 Status
DCSRC0       EQU  0x4b000018	;DMA 0 Current source
DCDST0       EQU  0x4b00001c	;DMA 0 Current destination
DMASKTRIG0   EQU  0x4b000020	;DMA 0 Mask trigger
	
DISRC1       EQU  0x4b000040	;DMA 1 Initial source
DISRCC1      EQU  0x4b000044	;DMA 1 Initial source control
DIDST1       EQU  0x4b000048	;DMA 1 Initial Destination
DIDSTC1      EQU  0x4b00004c	;DMA 1 Initial Destination control
DCON1        EQU  0x4b000050	;DMA 1 Control
DSTAT1       EQU  0x4b000054	;DMA 1 Status
DCSRC1       EQU  0x4b000058	;DMA 1 Current source
DCDST1       EQU  0x4b00005c	;DMA 1 Current destination
DMASKTRIG1   EQU  0x4b000060	;DMA 1 Mask trigger
	
DISRC2       EQU  0x4b000080	;DMA 2 Initial source
DISRCC2      EQU  0x4b000084	;DMA 2 Initial source control
DIDST2       EQU  0x4b000088	;DMA 2 Initial Destination
DIDSTC2      EQU  0x4b00008c	;DMA 2 Initial Destination control
DCON2        EQU  0x4b000090	;DMA 2 Control
DSTAT2       EQU  0x4b000094	;DMA 2 Status
DCSRC2       EQU  0x4b000098	;DMA 2 Current source
DCDST2       EQU  0x4b00009c	;DMA 2 Current destination
DMASKTRIG2   EQU  0x4b0000a0	;DMA 2 Mask trigger
	
DISRC3       EQU  0x4b0000c0	;DMA 3 Initial source
DISRCC3      EQU  0x4b0000c4	;DMA 3 Initial source control
DIDST3       EQU  0x4b0000c8	;DMA 3 Initial Destination
DIDSTC3      EQU  0x4b0000cc	;DMA 3 Initial Destination control
DCON3        EQU  0x4b0000d0	;DMA 3 Control
DSTAT3       EQU  0x4b0000d4	;DMA 3 Status
DCSRC3       EQU  0x4b0000d8	;DMA 3 Current source
DCDST3       EQU  0x4b0000dc	;DMA 3 Current destination
DMASKTRIG3   EQU  0x4b0000e0	;DMA 3 Mask trigger


;==========================
; CLOCK & POWER MANAGEMENT
;==========================
LOCKTIME    EQU  0x4c000000	;PLL lock time counter
MPLLCON     EQU  0x4c000004	;MPLL Control
UPLLCON     EQU  0x4c000008	;UPLL Control
CLKCON      EQU  0x4c00000c	;Clock generator control
CLKSLOW     EQU  0x4c000010	;Slow clock control
CLKDIVN     EQU  0x4c000014	;Clock divider control


;=================
; LCD CONTROLLER
;=================
LCDCON1     EQU  0x4d000000	;LCD control 1
LCDCON2     EQU  0x4d000004	;LCD control 2
LCDCON3     EQU  0x4d000008	;LCD control 3
LCDCON4     EQU  0x4d00000c	;LCD control 4
LCDCON5     EQU  0x4d000010	;LCD control 5
LCDSADDR1   EQU  0x4d000014	;STN/TFT Frame buffer start address 1
LCDSADDR2   EQU  0x4d000018	;STN/TFT Frame buffer start address 2
LCDSADDR3   EQU  0x4d00001c	;STN/TFT Virtual screen address set
REDLUT      EQU  0x4d000020	;STN Red lookup table
GREENLUT    EQU  0x4d000024	;STN Green lookup table 
BLUELUT     EQU  0x4d000028	;STN Blue lookup table
DITHMODE    EQU  0x4d00004c	;STN Dithering mode
TPAL        EQU  0x4d000050	;TFT Temporary palette
LCDINTPND   EQU  0x4d000054	;LCD Interrupt pending
LCDSRCPND   EQU  0x4d000058	;LCD Interrupt source
LCDINTMSK   EQU  0x4d00005c	;LCD Interrupt mask
LPCSEL      EQU  0x4d000060	;LPC3600 Control


;=================
; NAND flash
;=================
NFCONF      EQU  0x4e000000	;NAND Flash configuration
NFCMD       EQU  0x4e000004	;NADD Flash command
NFADDR      EQU  0x4e000008	;NAND Flash address
NFDATA      EQU  0x4e00000c	;NAND Flash data
NFSTAT      EQU  0x4e000010	;NAND Flash operation status
NFECC       EQU  0x4e000014	;NAND Flash ECC


;=================
; UART
;=================
ULCON0       EQU  0x50000000	;UART 0 Line control
UCON0        EQU  0x50000004	;UART 0 Control
UFCON0       EQU  0x50000008	;UART 0 FIFO control
UMCON0       EQU  0x5000000c	;UART 0 Modem control
UTRSTAT0     EQU  0x50000010	;UART 0 Tx/Rx status
UERSTAT0     EQU  0x50000014	;UART 0 Rx error status
UFSTAT0      EQU  0x50000018	;UART 0 FIFO status
UMSTAT0      EQU  0x5000001c	;UART 0 Modem status
UBRDIV0      EQU  0x50000028	;UART 0 Baud rate divisor
	
ULCON1       EQU  0x50004000	;UART 1 Line control
UCON1        EQU  0x50004004	;UART 1 Control
UFCON1       EQU  0x50004008	;UART 1 FIFO control
UMCON1       EQU  0x5000400c	;UART 1 Modem control
UTRSTAT1     EQU  0x50004010	;UART 1 Tx/Rx status
UERSTAT1     EQU  0x50004014	;UART 1 Rx error status
UFSTAT1      EQU  0x50004018	;UART 1 FIFO status
UMSTAT1      EQU  0x5000401c	;UART 1 Modem status
UBRDIV1      EQU  0x50004028	;UART 1 Baud rate divisor

ULCON2       EQU  0x50008000	;UART 2 Line control
UCON2        EQU  0x50008004	;UART 2 Control
UFCON2       EQU  0x50008008	;UART 2 FIFO control
UMCON2       EQU  0x5000800c	;UART 2 Modem control
UTRSTAT2     EQU  0x50008010	;UART 2 Tx/Rx status
UERSTAT2     EQU  0x50008014	;UART 2 Rx error status
UFSTAT2      EQU  0x50008018	;UART 2 FIFO status
UMSTAT2      EQU  0x5000801c	;UART 2 Modem status
UBRDIV2      EQU  0x50008028	;UART 2 Baud rate divisor
	
        [ BIG_ENDIAN__	
UTXH0        EQU  0x50000023	;UART 0 Transmission Hold
URXH0        EQU  0x50000027	;UART 0 Receive buffer
UTXH1        EQU  0x50004023	;UART 1 Transmission Hold
URXH1        EQU  0x50004027	;UART 1 Receive buffer
UTXH2        EQU  0x50008023	;UART 2 Transmission Hold
URXH2        EQU  0x50008027	;UART 2 Receive buffer

        |                   	;Little Endian
UTXH0        EQU  0x50000020	;UART 0 Transmission Hold
URXH0        EQU  0x50000024	;UART 0 Receive buffer
UTXH1        EQU  0x50004020	;UART 1 Transmission Hold
URXH1        EQU  0x50004024	;UART 1 Receive buffer
UTXH2        EQU  0x50008020	;UART 2 Transmission Hold
URXH2        EQU  0x50008024	;UART 2 Receive buffer
        ]


;=================
; PWM TIMER
;=================
TCFG0    EQU  0x51000000	;Timer 0 configuration
TCFG1    EQU  0x51000004	;Timer 1 configuration
TCON     EQU  0x51000008	;Timer control
TCNTB0   EQU  0x5100000c	;Timer count buffer 0
TCMPB0   EQU  0x51000010	;Timer compare buffer 0
TCNTO0   EQU  0x51000014	;Timer count observation 0
TCNTB1   EQU  0x51000018	;Timer count buffer 1
TCMPB1   EQU  0x5100001c	;Timer compare buffer 1
TCNTO1   EQU  0x51000020	;Timer count observation 1
TCNTB2   EQU  0x51000024	;Timer count buffer 2
TCMPB2   EQU  0x51000028	;Timer compare buffer 2
TCNTO2   EQU  0x5100002c	;Timer count observation 2
TCNTB3   EQU  0x51000030	;Timer count buffer 3
TCMPB3   EQU  0x51000034	;Timer compare buffer 3
TCNTO3   EQU  0x51000038	;Timer count observation 3
TCNTB4   EQU  0x5100003c	;Timer count buffer 4
TCNTO4   EQU  0x51000040	;Timer count observation 4


;=================
; USB DEVICE
;=================
        [ BIG_ENDIAN__
FUNC_ADDR_REG       EQU  0x52000143		;Function address
PWR_REG             EQU  0x52000147		;Power management
EP_INT_REG          EQU  0x5200014b		;EP Interrupt pending and clear
USB_INT_REG         EQU  0x5200015b		;USB Interrupt pending and clear
EP_INT_EN_REG       EQU  0x5200015f		;Interrupt enable
USB_INT_EN_REG      EQU  0x5200016f		
FRAME_NUM1_REG      EQU  0x52000173		;Frame number lower byte
FRAME_NUM2_REG      EQU  0x52000177		;Frame number lower byte
INDEX_REG           EQU  0x5200017b		;Register index
MAXP_REG            EQU  0x52000183		;Endpoint max packet
EP0_CSR             EQU  0x52000187		;Endpoint 0 status
IN_CSR1_REG         EQU  0x52000187		;In endpoint control status
IN_CSR2_REG         EQU  0x5200018b		
OUT_CSR1_REG        EQU  0x52000193		;Out endpoint control status
OUT_CSR2_REG        EQU  0x52000197		
OUT_FIFO_CNT1_REG   EQU  0x5200019b		;Endpoint out write count
OUT_FIFO_CNT2_REG   EQU  0x5200019f		
EP0_FIFO            EQU  0x520001c3		;Endpoint 0 FIFO
EP1_FIFO            EQU  0x520001c7		;Endpoint 1 FIFO
EP2_FIFO            EQU  0x520001cb		;Endpoint 2 FIFO
EP3_FIFO            EQU  0x520001cf		;Endpoint 3 FIFO
EP4_FIFO            EQU  0x520001d3		;Endpoint 4 FIFO
EP1_DMA_CON         EQU  0x52000203		;EP1 DMA interface control
EP1_DMA_UNIT        EQU  0x52000207		;EP1 DMA Tx unit counter
EP1_DMA_FIFO        EQU  0x5200020b		;EP1 DMA Tx FIFO counter
EP1_DMA_TTC_L       EQU  0x5200020f		;EP1 DMA total Tx counter
EP1_DMA_TTC_M       EQU  0x52000213		
EP1_DMA_TTC_H       EQU  0x52000217		
EP2_DMA_CON         EQU  0x5200021b		;EP2 DMA interface control
EP2_DMA_UNIT        EQU  0x5200021f		;EP2 DMA Tx unit counter
EP2_DMA_FIFO        EQU  0x52000223		;EP2 DMA Tx FIFO counter
EP2_DMA_TTC_L       EQU  0x52000227		;EP2 DMA total Tx counter
EP2_DMA_TTC_M       EQU  0x5200022b		
EP2_DMA_TTC_H       EQU  0x5200022f		
EP3_DMA_CON         EQU  0x52000243		;EP3 DMA interface control
EP3_DMA_UNIT        EQU  0x52000247		;EP3 DMA Tx unit counter
EP3_DMA_FIFO        EQU  0x5200024b		;EP3 DMA Tx FIFO counter
EP3_DMA_TTC_L       EQU  0x5200024f		;EP3 DMA total Tx counter
EP3_DMA_TTC_M       EQU  0x52000253		
EP3_DMA_TTC_H       EQU  0x52000257		
EP4_DMA_CON         EQU  0x5200025b		;EP4 DMA interface control
EP4_DMA_UNIT        EQU  0x5200025f		;EP4 DMA Tx unit counter
EP4_DMA_FIFO        EQU  0x52000263		;EP4 DMA Tx FIFO counter
EP4_DMA_TTC_L       EQU  0x52000267		;EP4 DMA total Tx counter
EP4_DMA_TTC_M       EQU  0x5200026b		
EP4_DMA_TTC_H       EQU  0x5200026f

        |   ; Little Endian
FUNC_ADDR_REG       EQU  0x52000140		;Function address
PWR_REG             EQU  0x52000144		;Power management
EP_INT_REG          EQU  0x52000148		;EP Interrupt pending and clear
USB_INT_REG         EQU  0x52000158		;USB Interrupt pending and clear
EP_INT_EN_REG       EQU  0x5200015c		;Interrupt enable
USB_INT_EN_REG      EQU  0x5200016c		
FRAME_NUM1_REG      EQU  0x52000170		;Frame number lower byte
FRAME_NUM2_REG      EQU  0x52000174		;Frame number lower byte
INDEX_REG           EQU  0x52000178		;Register index
MAXP_REG            EQU  0x52000180		;Endpoint max packet
EP0_CSR             EQU  0x52000184		;Endpoint 0 status
IN_CSR1_REG         EQU  0x52000184		;In endpoint control status
IN_CSR2_REG         EQU  0x52000188		
OUT_CSR1_REG        EQU  0x52000190		;Out endpoint control status
OUT_CSR2_REG        EQU  0x52000194		
OUT_FIFO_CNT1_REG   EQU  0x52000198		;Endpoint out write count
OUT_FIFO_CNT2_REG   EQU  0x5200019c		
EP0_FIFO            EQU  0x520001c0		;Endpoint 0 FIFO
EP1_FIFO            EQU  0x520001c4		;Endpoint 1 FIFO
EP2_FIFO            EQU  0x520001c8		;Endpoint 2 FIFO
EP3_FIFO            EQU  0x520001cc		;Endpoint 3 FIFO
EP4_FIFO            EQU  0x520001d0		;Endpoint 4 FIFO
EP1_DMA_CON         EQU  0x52000200		;EP1 DMA interface control
EP1_DMA_UNIT        EQU  0x52000204		;EP1 DMA Tx unit counter
EP1_DMA_FIFO        EQU  0x52000208		;EP1 DMA Tx FIFO counter
EP1_DMA_TTC_L       EQU  0x5200020c		;EP1 DMA total Tx counter
EP1_DMA_TTC_M       EQU  0x52000210		
EP1_DMA_TTC_H       EQU  0x52000214		
EP2_DMA_CON         EQU  0x52000218		;EP2 DMA interface control
EP2_DMA_UNIT        EQU  0x5200021c		;EP2 DMA Tx unit counter
EP2_DMA_FIFO        EQU  0x52000220		;EP2 DMA Tx FIFO counter
EP2_DMA_TTC_L       EQU  0x52000224		;EP2 DMA total Tx counter
EP2_DMA_TTC_M       EQU  0x52000228		
EP2_DMA_TTC_H       EQU  0x5200022c		
EP3_DMA_CON         EQU  0x52000240		;EP3 DMA interface control
EP3_DMA_UNIT        EQU  0x52000244		;EP3 DMA Tx unit counter
EP3_DMA_FIFO        EQU  0x52000248		;EP3 DMA Tx FIFO counter
EP3_DMA_TTC_L       EQU  0x5200024c		;EP3 DMA total Tx counter
EP3_DMA_TTC_M       EQU  0x52000250		
EP3_DMA_TTC_H       EQU  0x52000254		
EP4_DMA_CON         EQU  0x52000258		;EP4 DMA interface control
EP4_DMA_UNIT        EQU  0x5200025c		;EP4 DMA Tx unit counter
EP4_DMA_FIFO        EQU  0x52000260		;EP4 DMA Tx FIFO counter
EP4_DMA_TTC_L       EQU  0x52000264		;EP4 DMA total Tx counter
EP4_DMA_TTC_M       EQU  0x52000268		
EP4_DMA_TTC_H       EQU  0x5200026c		
        ]


;=================
; WATCH DOG TIMER
;=================
WTCON     EQU  0x53000000	;Watch-dog timer mode
WTDAT     EQU  0x53000004	;Watch-dog timer data
WTCNT     EQU  0x53000008	;Eatch-dog timer count
		
		
;=================		
; IIC		
;=================		
IICCON    EQU  0x54000000	;IIC control
IICSTAT   EQU  0x54000004	;IIC status
IICADD    EQU  0x54000008	;IIC address
IICDS     EQU  0x5400000c	;IIC data shift
		
		
;=================		
; IIS		
;=================		
IISCON    EQU  0x55000000	;IIS Control
IISMOD    EQU  0x55000004	;IIS Mode
IISPSR    EQU  0x55000008	;IIS Prescaler
IISFCON   EQU  0x5500000c	;IIS FIFO control

        [ BIG_ENDIAN__
IISFIFO    EQU  0x55000012	;IIS FIFO entry
        |                 	;Little Endian
IISFIFO    EQU  0x55000010	;IIS FIFO entry
        ]


;=================
; I/O PORT 
;=================
GPACON      EQU  0x56000000	;Port A control
GPADAT      EQU  0x56000004	;Port A data
							
GPBCON      EQU  0x56000010	;Port B control
GPBDAT      EQU  0x56000014	;Port B data
GPBUP       EQU  0x56000018	;Pull-up control B
							
GPCCON      EQU  0x56000020	;Port C control
GPCDAT      EQU  0x56000024	;Port C data
GPCUP       EQU  0x56000028	;Pull-up control C
							
GPDCON      EQU  0x56000030	;Port D control
GPDDAT      EQU  0x56000034	;Port D data
GPDUP       EQU  0x56000038	;Pull-up control D
							
GPECON      EQU  0x56000040	;Port E control
GPEDAT      EQU  0x56000044	;Port E data
GPEUP       EQU  0x56000048	;Pull-up control E
							
GPFCON      EQU  0x56000050	;Port F control
GPFDAT      EQU  0x56000054	;Port F data
GPFUP       EQU  0x56000058	;Pull-up control F
							
GPGCON      EQU  0x56000060	;Port G control
GPGDAT      EQU  0x56000064	;Port G data
GPGUP       EQU  0x56000068	;Pull-up control G
							
GPHCON      EQU  0x56000070	;Port H control
GPHDAT      EQU  0x56000074	;Port H data
GPHUP       EQU  0x56000078	;Pull-up control H
							
MISCCR      EQU  0x56000080	;Miscellaneous control
DCKCON      EQU  0x56000084	;DCLK0/1 control
EXTINT0     EQU  0x56000088	;External interrupt control register 0
EXTINT1     EQU  0x5600008c	;External interrupt control register 1
EXTINT2     EQU  0x56000090	;External interrupt control register 2
EINTFLT0    EQU  0x56000094	;Reserved
EINTFLT1    EQU  0x56000098	;Reserved
EINTFLT2    EQU  0x5600009c	;External interrupt filter control register 2
EINTFLT3    EQU  0x560000a0	;External interrupt filter control register 3
EINTMASK    EQU  0x560000a4	;External interrupt mask
EINTPEND    EQU  0x560000a8	;External interrupt pending
GSTATUS0    EQU  0x560000ac	;External pin status
GSTATUS1    EQU  0x560000b0	;Chip ID(0x32410000)
GSTATUS2    EQU  0x560000b4	;Reset type
GSTATUS3    EQU  0x560000b8	;Saved data0(32-bit) before entering POWER_OFF mode 
GSTATUS4    EQU  0x560000bc	;Saved data1(32-bit) before entering POWER_OFF mode


;=================
; RTC
;=================
        [ BIG_ENDIAN__
RTCCON    EQU  0x57000043	;RTC control
TICNT     EQU  0x57000047	;Tick time count
RTCALM    EQU  0x57000053	;RTC alarm control
ALMSEC    EQU  0x57000057	;Alarm second
ALMMIN    EQU  0x5700005b	;Alarm minute
ALMHOUR   EQU  0x5700005f	;Alarm Hour
ALMDATE   EQU  0x57000063	;Alarm day      -- May 06, 2002 SOP
ALMMON    EQU  0x57000067	;Alarm month
ALMYEAR   EQU  0x5700006b	;Alarm year
RTCRST    EQU  0x5700006f	;RTC round reset
BCDSEC    EQU  0x57000073	;BCD second
BCDMIN    EQU  0x57000077	;BCD minute
BCDHOUR   EQU  0x5700007b	;BCD hour
BCDDATE   EQU  0x5700007f	;BCD day        -- May 06, 2002 SOP
BCDDAY    EQU  0x57000083	;BCD date       -- May 06, 2002 SOP
BCDMON    EQU  0x57000087	;BCD month
BCDYEAR   EQU  0x5700008b	;BCD year
		
        |                	;Little Endian
RTCCON    EQU  0x57000040	;RTC control
TICNT     EQU  0x57000044	;Tick time count
RTCALM    EQU  0x57000050	;RTC alarm control
ALMSEC    EQU  0x57000054	;Alarm second
ALMMIN    EQU  0x57000058	;Alarm minute
ALMHOUR   EQU  0x5700005c	;Alarm Hour
ALMDATE   EQU  0x57000060	;Alarm day      -- May 06, 2002 SOP
ALMMON    EQU  0x57000064	;Alarm month
ALMYEAR   EQU  0x57000068	;Alarm year
RTCRST    EQU  0x5700006c	;RTC round reset
BCDSEC    EQU  0x57000070	;BCD second
BCDMIN    EQU  0x57000074	;BCD minute
BCDHOUR   EQU  0x57000078	;BCD hour
BCDDATE   EQU  0x5700007c	;BCD day        -- May 06, 2002 SOP
BCDDAY    EQU  0x57000080	;BCD date       -- May 06, 2002 SOP
BCDMON    EQU  0x57000084	;BCD month
BCDYEAR   EQU  0x57000088	;BCD year
        ]                	;RTC


;=================
; ADC
;=================
ADCCON      EQU  0x58000000	;ADC control
ADCTSC      EQU  0x58000004	;ADC touch screen control
ADCDLY      EQU  0x58000008	;ADC start or Interval Delay
ADCDAT0     EQU  0x5800000c	;ADC conversion data 0
ADCDAT1     EQU  0x58000010	;ADC conversion data 1                     
		
		
;=================         		
; SPI           		
;=================		
SPCON0      EQU  0x59000000	;SPI0 control
SPSTA0      EQU  0x59000004	;SPI0 status
SPPIN0      EQU  0x59000008	;SPI0 pin control
SPPRE0      EQU  0x5900000c	;SPI0 baud rate prescaler
SPTDAT0     EQU  0x59000010	;SPI0 Tx data
SPRDAT0     EQU  0x59000014	;SPI0 Rx data
		
SPCON1      EQU  0x59000020	;SPI1 control
SPSTA1      EQU  0x59000024	;SPI1 status
SPPIN1      EQU  0x59000028	;SPI1 pin control
SPPRE1      EQU  0x5900002c	;SPI1 baud rate prescaler
SPTDAT1     EQU  0x59000030	;SPI1 Tx data
SPRDAT1     EQU  0x59000034	;SPI1 Rx data

;=================
; SD Interface
;=================
SDICON      EQU  0x5a000000	;SDI control
SDIPRE      EQU  0x5a000000	;SDI baud rate prescaler
SDICmdArg   EQU  0x5a000000	;SDI command argument
SDICmdCon   EQU  0x5a000000	;SDI command control
SDICmdSta   EQU  0x5a000000	;SDI command status
SDIRSP0     EQU  0x5a000000	;SDI response 0
SDIRSP1     EQU  0x5a000000	;SDI response 1
SDIRSP2     EQU  0x5a000000	;SDI response 2
SDIRSP3     EQU  0x5a000000	;SDI response 3
SDIDTimer   EQU  0x5a000000	;SDI data/busy timer
SDIBSize    EQU  0x5a000000	;SDI block size
SDIDatCon   EQU  0x5a000000	;SDI data control
SDIDatCnt   EQU  0x5a000000	;SDI data remain counter
SDIDatSta   EQU  0x5a000000	;SDI data status
SDIFSTA     EQU  0x5a000000	;SDI FIFO status
SDIIntMsk   EQU  0x5a000000	;SDI interrupt mask
		
        [ BIG_ENDIAN__		
SDIDAT      EQU  0x5a00003f	;SDI data
        |                  	;Little Endian
SDIDAT      EQU  0x5a00003c	;SDI data
        ]                  	;SD Interface

;/********************************************************************************************************
        END
;*********************************************************************************************************/

~~~

**`startup_head.s`**

作为头文件，定义了 ARM 板的初始设置

比如堆栈基址，FCLK:HCLK:PCLK 分频，USB 频率等参数

~~~ini
;input frequency	12.00 MHz
;MPLL的分频配置 		;MPLL=(2*m*Fin)/(p*2^s)
M_MDIV		EQU		127	;m=(MDIV+8)
M_PDIV		EQU		2	;p=(PDIV+2)		
M_SDIV		EQU		1	;s=SDIV
; output frequency		405.00 MHz

; hdivn,pdivn FCLK:HCLK:PCLK
;     0,0         1:1:1 
;     0,1         1:1:2 
;     1,0         1:2:2
;     1,1         1:2:4
M_HDIVN		EQU		1	;HDIVN=01  代表HCLK=FCLK/2
M_PDIVN		EQU		1	;PDIVN=01  代表PCLK=HCLK/2
;所以FCLK:HCLK:PCLK=4:2:1


;Fin=12.0MHz 
;UPLL的分频配置 ; UPLL=(m*Fin)/(p*2^s)
U_MDIV		EQU		56	;m=(MDIV+8)	
U_PDIV		EQU		2	;p=(PDIV+2)
U_SDIV		EQU		2	;s=SDIV
;Fout=48.0MHz

R1_I    EQU     (1<<12)			;Rx_y 这一系列是在定义变量，分别代表R1寄存器上的不同位
R1_C    EQU     (1<<2)
R1_A    EQU     (1<<1)
R1_M    EQU     (1)
R1_iA   EQU     (1<<31)
R1_nF   EQU     (1<<30)


STACK_SIZE	EQU	128		;定义变量栈大小	
SUB_STACK_SIZE	EQU 	128		;定义变量子栈大小
	
STACK_BASE	EQU	(0x00001000)	;定义栈的基址
IRQStack_BASE	EQU	STACK_BASE			;定义IRQ栈的基址
UsrStack_BASE	EQU	(STACK_BASE - SUB_STACK_SIZE)	;定义用户栈的基址


	END
~~~

**`2440addr.h`**

这个文件作为 c 的头文件,定义了各种寄存器的地址宏,和清中断的函数


~~~c
//=============================================================================
// File Name : 2440addr.h
// Function  : S3C2440 Define Address Register
// History
// 0.0 : Programming start (February 15,2002) -- SOP
// Revision	: 03.11.2003 ver 0.0	Attatched for 2440
// 用来定义寄存器地址(寄存器地址宏),给C程序代码引用
//=============================================================================

#ifndef __2440ADDR_H__
#define __2440ADDR_H__

#ifdef __cplusplus
extern "C" {
#endif


// Memory control 
#define rBWSCON    (*(volatile unsigned *)0x48000000)	//Bus width & wait status
#define rBANKCON0  (*(volatile unsigned *)0x48000004)	//Boot ROM control
#define rBANKCON1  (*(volatile unsigned *)0x48000008)	//BANK1 control
#define rBANKCON2  (*(volatile unsigned *)0x4800000c)	//BANK2 cControl
#define rBANKCON3  (*(volatile unsigned *)0x48000010)	//BANK3 control
#define rBANKCON4  (*(volatile unsigned *)0x48000014)	//BANK4 control
#define rBANKCON5  (*(volatile unsigned *)0x48000018)	//BANK5 control
#define rBANKCON6  (*(volatile unsigned *)0x4800001c)	//BANK6 control
#define rBANKCON7  (*(volatile unsigned *)0x48000020)	//BANK7 control
#define rREFRESH   (*(volatile unsigned *)0x48000024)	//DRAM/SDRAM refresh
#define rBANKSIZE  (*(volatile unsigned *)0x48000028)	//Flexible Bank Size
#define rMRSRB6    (*(volatile unsigned *)0x4800002c)	//Mode register set for SDRAM
#define rMRSRB7    (*(volatile unsigned *)0x48000030)	//Mode register set for SDRAM


// USB Host


// INTERRUPT
#define rSRCPND     (*(volatile unsigned *)0x4a000000)	//Interrupt request status
#define rINTMOD     (*(volatile unsigned *)0x4a000004)	//Interrupt mode control
#define rINTMSK     (*(volatile unsigned *)0x4a000008)	//Interrupt mask control
#define rPRIORITY   (*(volatile unsigned *)0x4a00000c)	//IRQ priority control
#define rINTPND     (*(volatile unsigned *)0x4a000010)	//Interrupt request status
#define rINTOFFSET  (*(volatile unsigned *)0x4a000014)	//Interruot request source offset
#define rSUBSRCPND  (*(volatile unsigned *)0x4a000018)	//Sub source pending
#define rINTSUBMSK  (*(volatile unsigned *)0x4a00001c)	//Interrupt sub mask


// DMA
#define rDISRC0     (*(volatile unsigned *)0x4b000000)	//DMA 0 Initial source
#define rDISRCC0    (*(volatile unsigned *)0x4b000004)	//DMA 0 Initial source control
#define rDIDST0     (*(volatile unsigned *)0x4b000008)	//DMA 0 Initial Destination
#define rDIDSTC0    (*(volatile unsigned *)0x4b00000c)	//DMA 0 Initial Destination control
#define rDCON0      (*(volatile unsigned *)0x4b000010)	//DMA 0 Control
#define rDSTAT0     (*(volatile unsigned *)0x4b000014)	//DMA 0 Status
#define rDCSRC0     (*(volatile unsigned *)0x4b000018)	//DMA 0 Current source
#define rDCDST0     (*(volatile unsigned *)0x4b00001c)	//DMA 0 Current destination
#define rDMASKTRIG0 (*(volatile unsigned *)0x4b000020)	//DMA 0 Mask trigger

#define rDISRC1     (*(volatile unsigned *)0x4b000040)	//DMA 1 Initial source
#define rDISRCC1    (*(volatile unsigned *)0x4b000044)	//DMA 1 Initial source control
#define rDIDST1     (*(volatile unsigned *)0x4b000048)	//DMA 1 Initial Destination
#define rDIDSTC1    (*(volatile unsigned *)0x4b00004c)	//DMA 1 Initial Destination control
#define rDCON1      (*(volatile unsigned *)0x4b000050)	//DMA 1 Control
#define rDSTAT1     (*(volatile unsigned *)0x4b000054)	//DMA 1 Status
#define rDCSRC1     (*(volatile unsigned *)0x4b000058)	//DMA 1 Current source
#define rDCDST1     (*(volatile unsigned *)0x4b00005c)	//DMA 1 Current destination
#define rDMASKTRIG1 (*(volatile unsigned *)0x4b000060)	//DMA 1 Mask trigger

#define rDISRC2     (*(volatile unsigned *)0x4b000080)	//DMA 2 Initial source
#define rDISRCC2    (*(volatile unsigned *)0x4b000084)	//DMA 2 Initial source control
#define rDIDST2     (*(volatile unsigned *)0x4b000088)	//DMA 2 Initial Destination
#define rDIDSTC2    (*(volatile unsigned *)0x4b00008c)	//DMA 2 Initial Destination control
#define rDCON2      (*(volatile unsigned *)0x4b000090)	//DMA 2 Control
#define rDSTAT2     (*(volatile unsigned *)0x4b000094)	//DMA 2 Status
#define rDCSRC2     (*(volatile unsigned *)0x4b000098)	//DMA 2 Current source
#define rDCDST2     (*(volatile unsigned *)0x4b00009c)	//DMA 2 Current destination
#define rDMASKTRIG2 (*(volatile unsigned *)0x4b0000a0)	//DMA 2 Mask trigger

#define rDISRC3     (*(volatile unsigned *)0x4b0000c0)	//DMA 3 Initial source
#define rDISRCC3    (*(volatile unsigned *)0x4b0000c4)	//DMA 3 Initial source control
#define rDIDST3     (*(volatile unsigned *)0x4b0000c8)	//DMA 3 Initial Destination
#define rDIDSTC3    (*(volatile unsigned *)0x4b0000cc)	//DMA 3 Initial Destination control
#define rDCON3      (*(volatile unsigned *)0x4b0000d0)	//DMA 3 Control
#define rDSTAT3     (*(volatile unsigned *)0x4b0000d4)	//DMA 3 Status
#define rDCSRC3     (*(volatile unsigned *)0x4b0000d8)	//DMA 3 Current source
#define rDCDST3     (*(volatile unsigned *)0x4b0000dc)	//DMA 3 Current destination
#define rDMASKTRIG3 (*(volatile unsigned *)0x4b0000e0)	//DMA 3 Mask trigger


// CLOCK & POWER MANAGEMENT
#define rLOCKTIME   (*(volatile unsigned *)0x4c000000)	//PLL lock time counter
#define rMPLLCON    (*(volatile unsigned *)0x4c000004)	//MPLL Control
#define rUPLLCON    (*(volatile unsigned *)0x4c000008)	//UPLL Control
#define rCLKCON     (*(volatile unsigned *)0x4c00000c)	//Clock generator control
#define rCLKSLOW    (*(volatile unsigned *)0x4c000010)	//Slow clock control
#define rCLKDIVN    (*(volatile unsigned *)0x4c000014)	//Clock divider control
#define rCAMDIVN    (*(volatile unsigned *)0x4c000018)	//USB, CAM Clock divider control


// LCD CONTROLLER
#define rLCDCON1    (*(volatile unsigned *)0x4d000000)	//LCD control 1
#define rLCDCON2    (*(volatile unsigned *)0x4d000004)	//LCD control 2
#define rLCDCON3    (*(volatile unsigned *)0x4d000008)	//LCD control 3
#define rLCDCON4    (*(volatile unsigned *)0x4d00000c)	//LCD control 4
#define rLCDCON5    (*(volatile unsigned *)0x4d000010)	//LCD control 5
#define rLCDSADDR1  (*(volatile unsigned *)0x4d000014)	//STN/TFT Frame buffer start address 1
#define rLCDSADDR2  (*(volatile unsigned *)0x4d000018)	//STN/TFT Frame buffer start address 2
#define rLCDSADDR3  (*(volatile unsigned *)0x4d00001c)	//STN/TFT Virtual screen address set
#define rREDLUT     (*(volatile unsigned *)0x4d000020)	//STN Red lookup table
#define rGREENLUT   (*(volatile unsigned *)0x4d000024)	//STN Green lookup table 
#define rBLUELUT    (*(volatile unsigned *)0x4d000028)	//STN Blue lookup table
#define rDITHMODE   (*(volatile unsigned *)0x4d00004c)	//STN Dithering mode
#define rTPAL       (*(volatile unsigned *)0x4d000050)	//TFT Temporary palette
#define rLCDINTPND  (*(volatile unsigned *)0x4d000054)	//LCD Interrupt pending
#define rLCDSRCPND  (*(volatile unsigned *)0x4d000058)	//LCD Interrupt source
#define rLCDINTMSK  (*(volatile unsigned *)0x4d00005c)	//LCD Interrupt mask
#define rTCONSEL    (*(volatile unsigned *)0x4d000060)	//LPC3600 Control --- edited by junon
#define PALETTE     0x4d000400				//Palette start address


//Nand Flash
#define rNFCONF		(*(volatile unsigned *)0x4E000000)	//NAND Flash configuration
#define rNFCONT		(*(volatile unsigned *)0x4E000004)	//NAND Flash control
#define rNFCMD		(*(volatile unsigned *)0x4E000008)	//NAND Flash command
#define rNFADDR		(*(volatile unsigned *)0x4E00000C)	//NAND Flash address
#define rNFDATA		(*(volatile unsigned *)0x4E000010)	//NAND Flash data
#define rNFDATA8	(*(volatile unsigned char *)0x4E000010) //NAND Flash data
#define NFDATA		(0x4E000010)      			//NAND Flash data address
#define rNFMECCD0	(*(volatile unsigned *)0x4E000014)	//NAND Flash ECC for Main Area
#define rNFMECCD1	(*(volatile unsigned *)0x4E000018)
#define rNFSECCD	(*(volatile unsigned *)0x4E00001C)	//NAND Flash ECC for Spare Area
#define rNFSTAT		(*(volatile unsigned *)0x4E000020)	//NAND Flash operation status
#define rNFESTAT0	(*(volatile unsigned *)0x4E000024)
#define rNFESTAT1	(*(volatile unsigned *)0x4E000028)
#define rNFMECC0	(*(volatile unsigned *)0x4E00002C)
#define rNFMECC1	(*(volatile unsigned *)0x4E000030)
#define rNFSECC		(*(volatile unsigned *)0x4E000034)
#define rNFSBLK		(*(volatile unsigned *)0x4E000038)	//NAND Flash Start block address
#define rNFEBLK		(*(volatile unsigned *)0x4E00003C)	//NAND Flash End block address


//Camera Interface.  Edited for 2440A                              
#define rCISRCFMT           (*(volatile unsigned *)0x4F000000)        
#define rCIWDOFST           (*(volatile unsigned *)0x4F000004)        
#define rCIGCTRL            (*(volatile unsigned *)0x4F000008)        
#define rCICOYSA1           (*(volatile unsigned *)0x4F000018)
#define rCICOYSA2           (*(volatile unsigned *)0x4F00001C)
#define rCICOYSA3           (*(volatile unsigned *)0x4F000020)        
#define rCICOYSA4           (*(volatile unsigned *)0x4F000024)        
#define rCICOCBSA1          (*(volatile unsigned *)0x4F000028)        
#define rCICOCBSA2          (*(volatile unsigned *)0x4F00002C)        
#define rCICOCBSA3          (*(volatile unsigned *)0x4F000030)        
#define rCICOCBSA4          (*(volatile unsigned *)0x4F000034)
#define rCICOCRSA1          (*(volatile unsigned *)0x4F000038)
#define rCICOCRSA2          (*(volatile unsigned *)0x4F00003C)
#define rCICOCRSA3          (*(volatile unsigned *)0x4F000040)
#define rCICOCRSA4          (*(volatile unsigned *)0x4F000044)
#define rCICOTRGFMT         (*(volatile unsigned *)0x4F000048)
#define rCICOCTRL           (*(volatile unsigned *)0x4F00004C)        
#define rCICOSCPRERATIO     (*(volatile unsigned *)0x4F000050)        
#define rCICOSCPREDST       (*(volatile unsigned *)0x4F000054)
#define rCICOSCCTRL         (*(volatile unsigned *)0x4F000058)
#define rCICOTAREA          (*(volatile unsigned *)0x4F00005C)
#define rCICOSTATUS         (*(volatile unsigned *)0x4F000064)
#define rCIPRCLRSA1         (*(volatile unsigned *)0x4F00006C)
#define rCIPRCLRSA2         (*(volatile unsigned *)0x4F000070)
#define rCIPRCLRSA3         (*(volatile unsigned *)0x4F000074)        
#define rCIPRCLRSA4         (*(volatile unsigned *)0x4F000078)        
#define rCIPRTRGFMT         (*(volatile unsigned *)0x4F00007C)        
#define rCIPRCTRL           (*(volatile unsigned *)0x4F000080)        
#define rCIPRSCPRERATIO     (*(volatile unsigned *)0x4F000084)        
#define rCIPRSCPREDST       (*(volatile unsigned *)0x4F000088)        
#define rCIPRSCCTRL         (*(volatile unsigned *)0x4F00008C)        
#define rCIPRTAREA          (*(volatile unsigned *)0x4F000090)
#define rCIPRSTATUS         (*(volatile unsigned *)0x4F000098)
#define rCIIMGCPT           (*(volatile unsigned *)0x4F0000A0)


// UART
#define rULCON0     (*(volatile unsigned *)0x50000000)	//UART 0 Line control
#define rUCON0      (*(volatile unsigned *)0x50000004)	//UART 0 Control
#define rUFCON0     (*(volatile unsigned *)0x50000008)	//UART 0 FIFO control
#define rUMCON0     (*(volatile unsigned *)0x5000000c)	//UART 0 Modem control
#define rUTRSTAT0   (*(volatile unsigned *)0x50000010)	//UART 0 Tx/Rx status
#define rUERSTAT0   (*(volatile unsigned *)0x50000014)	//UART 0 Rx error status
#define rUFSTAT0    (*(volatile unsigned *)0x50000018)	//UART 0 FIFO status
#define rUMSTAT0    (*(volatile unsigned *)0x5000001c)	//UART 0 Modem status
#define rUBRDIV0    (*(volatile unsigned *)0x50000028)	//UART 0 Baud rate divisor

#define rULCON1     (*(volatile unsigned *)0x50004000)	//UART 1 Line control
#define rUCON1      (*(volatile unsigned *)0x50004004)	//UART 1 Control
#define rUFCON1     (*(volatile unsigned *)0x50004008)	//UART 1 FIFO control
#define rUMCON1     (*(volatile unsigned *)0x5000400c)	//UART 1 Modem control
#define rUTRSTAT1   (*(volatile unsigned *)0x50004010)	//UART 1 Tx/Rx status
#define rUERSTAT1   (*(volatile unsigned *)0x50004014)	//UART 1 Rx error status
#define rUFSTAT1    (*(volatile unsigned *)0x50004018)	//UART 1 FIFO status
#define rUMSTAT1    (*(volatile unsigned *)0x5000401c)	//UART 1 Modem status
#define rUBRDIV1    (*(volatile unsigned *)0x50004028)	//UART 1 Baud rate divisor
#define rULCON2     (*(volatile unsigned *)0x50008000)	//UART 2 Line control
#define rUCON2      (*(volatile unsigned *)0x50008004)	//UART 2 Control
#define rUFCON2     (*(volatile unsigned *)0x50008008)	//UART 2 FIFO control
#define rUMCON2     (*(volatile unsigned *)0x5000800c)	//UART 2 Modem control
#define rUTRSTAT2   (*(volatile unsigned *)0x50008010)	//UART 2 Tx/Rx status
#define rUERSTAT2   (*(volatile unsigned *)0x50008014)	//UART 2 Rx error status
#define rUFSTAT2    (*(volatile unsigned *)0x50008018)	//UART 2 FIFO status
#define rUMSTAT2    (*(volatile unsigned *)0x5000801c)	//UART 2 Modem status
#define rUBRDIV2    (*(volatile unsigned *)0x50008028)	//UART 2 Baud rate divisor

#ifdef __BIG_ENDIAN
#define rUTXH0      (*(volatile unsigned char *)0x50000023)	//UART 0 Transmission Hold
#define rURXH0      (*(volatile unsigned char *)0x50000027)	//UART 0 Receive buffer
#define rUTXH1      (*(volatile unsigned char *)0x50004023)	//UART 1 Transmission Hold
#define rURXH1      (*(volatile unsigned char *)0x50004027)	//UART 1 Receive buffer
#define rUTXH2      (*(volatile unsigned char *)0x50008023)	//UART 2 Transmission Hold
#define rURXH2      (*(volatile unsigned char *)0x50008027)	//UART 2 Receive buffer

#define WrUTXH0(ch) (*(volatile unsigned char *)0x50000023)=(unsigned char)(ch)
#define RdURXH0()   (*(volatile unsigned char *)0x50000027)
#define WrUTXH1(ch) (*(volatile unsigned char *)0x50004023)=(unsigned char)(ch)
#define RdURXH1()   (*(volatile unsigned char *)0x50004027)
#define WrUTXH2(ch) (*(volatile unsigned char *)0x50008023)=(unsigned char)(ch)
#define RdURXH2()   (*(volatile unsigned char *)0x50008027)

#define UTXH0       (0x50000020+3)  				//Byte_access address by DMA
#define URXH0       (0x50000024+3)
#define UTXH1       (0x50004020+3)
#define URXH1       (0x50004024+3)
#define UTXH2       (0x50008020+3)
#define URXH2       (0x50008024+3)

#else //Little Endian
#define rUTXH0 (*(volatile unsigned char *)0x50000020)		//UART 0 Transmission Hold
#define rURXH0 (*(volatile unsigned char *)0x50000024)		//UART 0 Receive buffer
#define rUTXH1 (*(volatile unsigned char *)0x50004020)		//UART 1 Transmission Hold
#define rURXH1 (*(volatile unsigned char *)0x50004024)		//UART 1 Receive buffer
#define rUTXH2 (*(volatile unsigned char *)0x50008020)		//UART 2 Transmission Hold
#define rURXH2 (*(volatile unsigned char *)0x50008024)		//UART 2 Receive buffer

#define WrUTXH0(ch) (*(volatile unsigned char *)0x50000020)=(unsigned char)(ch)
#define RdURXH0()   (*(volatile unsigned char *)0x50000024)
#define WrUTXH1(ch) (*(volatile unsigned char *)0x50004020)=(unsigned char)(ch)
#define RdURXH1()   (*(volatile unsigned char *)0x50004024)
#define WrUTXH2(ch) (*(volatile unsigned char *)0x50008020)=(unsigned char)(ch)
#define RdURXH2()   (*(volatile unsigned char *)0x50008024)

#define UTXH0       (0x50000020)    				//Byte_access address by DMA
#define URXH0       (0x50000024)
#define UTXH1       (0x50004020)
#define URXH1       (0x50004024)
#define UTXH2       (0x50008020)
#define URXH2       (0x50008024)
#endif


// PWM TIMER
#define rTCFG0  (*(volatile unsigned *)0x51000000)	//Timer 0 configuration
#define rTCFG1  (*(volatile unsigned *)0x51000004)	//Timer 1 configuration
#define rTCON   (*(volatile unsigned *)0x51000008)	//Timer control
#define rTCNTB0 (*(volatile unsigned *)0x5100000c)	//Timer count buffer 0
#define rTCMPB0 (*(volatile unsigned *)0x51000010)	//Timer compare buffer 0
#define rTCNTO0 (*(volatile unsigned *)0x51000014)	//Timer count observation 0
#define rTCNTB1 (*(volatile unsigned *)0x51000018)	//Timer count buffer 1
#define rTCMPB1 (*(volatile unsigned *)0x5100001c)	//Timer compare buffer 1
#define rTCNTO1 (*(volatile unsigned *)0x51000020)	//Timer count observation 1
#define rTCNTB2 (*(volatile unsigned *)0x51000024)	//Timer count buffer 2
#define rTCMPB2 (*(volatile unsigned *)0x51000028)	//Timer compare buffer 2
#define rTCNTO2 (*(volatile unsigned *)0x5100002c)	//Timer count observation 2
#define rTCNTB3 (*(volatile unsigned *)0x51000030)	//Timer count buffer 3
#define rTCMPB3 (*(volatile unsigned *)0x51000034)	//Timer compare buffer 3
#define rTCNTO3 (*(volatile unsigned *)0x51000038)	//Timer count observation 3
#define rTCNTB4 (*(volatile unsigned *)0x5100003c)	//Timer count buffer 4
#define rTCNTO4 (*(volatile unsigned *)0x51000040)	//Timer count observation 4


// USB DEVICE
#ifdef __BIG_ENDIAN
<ERROR IF BIG_ENDIAN>
#define rFUNC_ADDR_REG     (*(volatile unsigned char *)0x52000143)	//Function address
#define rPWR_REG           (*(volatile unsigned char *)0x52000147)	//Power management
#define rEP_INT_REG        (*(volatile unsigned char *)0x5200014b)	//EP Interrupt pending and clear
#define rUSB_INT_REG       (*(volatile unsigned char *)0x5200015b)	//USB Interrupt pending and clear
#define rEP_INT_EN_REG     (*(volatile unsigned char *)0x5200015f)	//Interrupt enable
#define rUSB_INT_EN_REG    (*(volatile unsigned char *)0x5200016f)
#define rFRAME_NUM1_REG    (*(volatile unsigned char *)0x52000173)	//Frame number lower byte
#define rFRAME_NUM2_REG    (*(volatile unsigned char *)0x52000177)	//Frame number higher byte
#define rINDEX_REG         (*(volatile unsigned char *)0x5200017b)	//Register index
#define rMAXP_REG          (*(volatile unsigned char *)0x52000183)	//Endpoint max packet
#define rEP0_CSR           (*(volatile unsigned char *)0x52000187)	//Endpoint 0 status
#define rIN_CSR1_REG       (*(volatile unsigned char *)0x52000187)	//In endpoint control status
#define rIN_CSR2_REG       (*(volatile unsigned char *)0x5200018b)
#define rOUT_CSR1_REG      (*(volatile unsigned char *)0x52000193)	//Out endpoint control status
#define rOUT_CSR2_REG      (*(volatile unsigned char *)0x52000197)
#define rOUT_FIFO_CNT1_REG (*(volatile unsigned char *)0x5200019b)	//Endpoint out write count
#define rOUT_FIFO_CNT2_REG (*(volatile unsigned char *)0x5200019f)
#define rEP0_FIFO          (*(volatile unsigned char *)0x520001c3)	//Endpoint 0 FIFO
#define rEP1_FIFO          (*(volatile unsigned char *)0x520001c7)	//Endpoint 1 FIFO
#define rEP2_FIFO          (*(volatile unsigned char *)0x520001cb)	//Endpoint 2 FIFO
#define rEP3_FIFO          (*(volatile unsigned char *)0x520001cf)	//Endpoint 3 FIFO
#define rEP4_FIFO          (*(volatile unsigned char *)0x520001d3)	//Endpoint 4 FIFO
#define rEP1_DMA_CON       (*(volatile unsigned char *)0x52000203)	//EP1 DMA interface control
#define rEP1_DMA_UNIT      (*(volatile unsigned char *)0x52000207)	//EP1 DMA Tx unit counter
#define rEP1_DMA_FIFO      (*(volatile unsigned char *)0x5200020b)	//EP1 DMA Tx FIFO counter
#define rEP1_DMA_TTC_L     (*(volatile unsigned char *)0x5200020f)	//EP1 DMA total Tx counter
#define rEP1_DMA_TTC_M     (*(volatile unsigned char *)0x52000213)
#define rEP1_DMA_TTC_H     (*(volatile unsigned char *)0x52000217)
#define rEP2_DMA_CON       (*(volatile unsigned char *)0x5200021b)	//EP2 DMA interface control
#define rEP2_DMA_UNIT      (*(volatile unsigned char *)0x5200021f)	//EP2 DMA Tx unit counter
#define rEP2_DMA_FIFO      (*(volatile unsigned char *)0x52000223)	//EP2 DMA Tx FIFO counter
#define rEP2_DMA_TTC_L     (*(volatile unsigned char *)0x52000227)	//EP2 DMA total Tx counter
#define rEP2_DMA_TTC_M     (*(volatile unsigned char *)0x5200022b)
#define rEP2_DMA_TTC_H     (*(volatile unsigned char *)0x5200022f)
#define rEP3_DMA_CON       (*(volatile unsigned char *)0x52000243)	//EP3 DMA interface control
#define rEP3_DMA_UNIT      (*(volatile unsigned char *)0x52000247)	//EP3 DMA Tx unit counter
#define rEP3_DMA_FIFO      (*(volatile unsigned char *)0x5200024b)	//EP3 DMA Tx FIFO counter
#define rEP3_DMA_TTC_L     (*(volatile unsigned char *)0x5200024f)	//EP3 DMA total Tx counter
#define rEP3_DMA_TTC_M     (*(volatile unsigned char *)0x52000253)
#define rEP3_DMA_TTC_H     (*(volatile unsigned char *)0x52000257) 
#define rEP4_DMA_CON       (*(volatile unsigned char *)0x5200025b)	//EP4 DMA interface control
#define rEP4_DMA_UNIT      (*(volatile unsigned char *)0x5200025f)	//EP4 DMA Tx unit counter
#define rEP4_DMA_FIFO      (*(volatile unsigned char *)0x52000263)	//EP4 DMA Tx FIFO counter
#define rEP4_DMA_TTC_L     (*(volatile unsigned char *)0x52000267)	//EP4 DMA total Tx counter
#define rEP4_DMA_TTC_M     (*(volatile unsigned char *)0x5200026b)
#define rEP4_DMA_TTC_H     (*(volatile unsigned char *)0x5200026f)

#else  // Little Endian
#define rFUNC_ADDR_REG     (*(volatile unsigned char *)0x52000140)	//Function address
#define rPWR_REG           (*(volatile unsigned char *)0x52000144)	//Power management
#define rEP_INT_REG        (*(volatile unsigned char *)0x52000148)	//EP Interrupt pending and clear
#define rUSB_INT_REG       (*(volatile unsigned char *)0x52000158)	//USB Interrupt pending and clear
#define rEP_INT_EN_REG     (*(volatile unsigned char *)0x5200015c)	//Interrupt enable
#define rUSB_INT_EN_REG    (*(volatile unsigned char *)0x5200016c)
#define rFRAME_NUM1_REG    (*(volatile unsigned char *)0x52000170)	//Frame number lower byte
#define rFRAME_NUM2_REG    (*(volatile unsigned char *)0x52000174)	//Frame number higher byte
#define rINDEX_REG         (*(volatile unsigned char *)0x52000178)	//Register index
#define rMAXP_REG          (*(volatile unsigned char *)0x52000180)	//Endpoint max packet
#define rEP0_CSR           (*(volatile unsigned char *)0x52000184)	//Endpoint 0 status
#define rIN_CSR1_REG       (*(volatile unsigned char *)0x52000184)	//In endpoint control status
#define rIN_CSR2_REG       (*(volatile unsigned char *)0x52000188)
#define rOUT_CSR1_REG      (*(volatile unsigned char *)0x52000190)	//Out endpoint control status
#define rOUT_CSR2_REG      (*(volatile unsigned char *)0x52000194)
#define rOUT_FIFO_CNT1_REG (*(volatile unsigned char *)0x52000198)	//Endpoint out write count
#define rOUT_FIFO_CNT2_REG (*(volatile unsigned char *)0x5200019c)
#define rEP0_FIFO          (*(volatile unsigned char *)0x520001c0)	//Endpoint 0 FIFO
#define rEP1_FIFO          (*(volatile unsigned char *)0x520001c4)	//Endpoint 1 FIFO
#define rEP2_FIFO          (*(volatile unsigned char *)0x520001c8)	//Endpoint 2 FIFO
#define rEP3_FIFO          (*(volatile unsigned char *)0x520001cc)	//Endpoint 3 FIFO
#define rEP4_FIFO          (*(volatile unsigned char *)0x520001d0)	//Endpoint 4 FIFO
#define rEP1_DMA_CON       (*(volatile unsigned char *)0x52000200)	//EP1 DMA interface control
#define rEP1_DMA_UNIT      (*(volatile unsigned char *)0x52000204)	//EP1 DMA Tx unit counter
#define rEP1_DMA_FIFO      (*(volatile unsigned char *)0x52000208)	//EP1 DMA Tx FIFO counter
#define rEP1_DMA_TTC_L     (*(volatile unsigned char *)0x5200020c)	//EP1 DMA total Tx counter
#define rEP1_DMA_TTC_M     (*(volatile unsigned char *)0x52000210)
#define rEP1_DMA_TTC_H     (*(volatile unsigned char *)0x52000214)
#define rEP2_DMA_CON       (*(volatile unsigned char *)0x52000218)	//EP2 DMA interface control
#define rEP2_DMA_UNIT      (*(volatile unsigned char *)0x5200021c)	//EP2 DMA Tx unit counter
#define rEP2_DMA_FIFO      (*(volatile unsigned char *)0x52000220)	//EP2 DMA Tx FIFO counter
#define rEP2_DMA_TTC_L     (*(volatile unsigned char *)0x52000224)	//EP2 DMA total Tx counter
#define rEP2_DMA_TTC_M     (*(volatile unsigned char *)0x52000228)
#define rEP2_DMA_TTC_H     (*(volatile unsigned char *)0x5200022c)
#define rEP3_DMA_CON       (*(volatile unsigned char *)0x52000240)	//EP3 DMA interface control
#define rEP3_DMA_UNIT      (*(volatile unsigned char *)0x52000244)	//EP3 DMA Tx unit counter
#define rEP3_DMA_FIFO      (*(volatile unsigned char *)0x52000248)	//EP3 DMA Tx FIFO counter
#define rEP3_DMA_TTC_L     (*(volatile unsigned char *)0x5200024c)	//EP3 DMA total Tx counter
#define rEP3_DMA_TTC_M     (*(volatile unsigned char *)0x52000250)
#define rEP3_DMA_TTC_H     (*(volatile unsigned char *)0x52000254)
#define rEP4_DMA_CON       (*(volatile unsigned char *)0x52000258)	//EP4 DMA interface control
#define rEP4_DMA_UNIT      (*(volatile unsigned char *)0x5200025c)	//EP4 DMA Tx unit counter
#define rEP4_DMA_FIFO      (*(volatile unsigned char *)0x52000260)	//EP4 DMA Tx FIFO counter
#define rEP4_DMA_TTC_L     (*(volatile unsigned char *)0x52000264)	//EP4 DMA total Tx counter
#define rEP4_DMA_TTC_M     (*(volatile unsigned char *)0x52000268)
#define rEP4_DMA_TTC_H     (*(volatile unsigned char *)0x5200026c)
#endif   // __BIG_ENDIAN


// WATCH DOG TIMER
#define rWTCON   (*(volatile unsigned *)0x53000000)		//Watch-dog timer mode
#define rWTDAT   (*(volatile unsigned *)0x53000004)		//Watch-dog timer data
#define rWTCNT   (*(volatile unsigned *)0x53000008)		//Eatch-dog timer count


// IIC
#define rIICCON		(*(volatile unsigned *)0x54000000)	//IIC control
#define rIICSTAT	(*(volatile unsigned *)0x54000004)	//IIC status
#define rIICADD		(*(volatile unsigned *)0x54000008)	//IIC address
#define rIICDS		(*(volatile unsigned *)0x5400000c)	//IIC data shift
#define rIICLC		(*(volatile unsigned *)0x54000010)	//IIC multi-master line control


// IIS
#define rIISCON  (*(volatile unsigned *)0x55000000)		//IIS Control
#define rIISMOD  (*(volatile unsigned *)0x55000004)		//IIS Mode
#define rIISPSR  (*(volatile unsigned *)0x55000008)		//IIS Prescaler
#define rIISFCON (*(volatile unsigned *)0x5500000c)		//IIS FIFO control
#ifdef __BIG_ENDIAN
#define IISFIFO  ((volatile unsigned short *)0x55000012)	//IIS FIFO entry
#else 														//Little Endian
#define IISFIFO  ((volatile unsigned short *)0x55000010)	//IIS FIFO entry
#endif


//AC97, Added for S3C2440A 
#define rAC_GLBCTRL		*(volatile unsigned *)0x5b000000
#define rAC_GLBSTAT		*(volatile unsigned *)0x5b000004
#define rAC_CODEC_CMD	*(volatile unsigned *)0x5b000008
#define rAC_CODEC_STAT	*(volatile unsigned *)0x5b00000C
#define rAC_PCMADDR		*(volatile unsigned *)0x5b000010
#define rAC_MICADDR		*(volatile unsigned *)0x5b000014
#define rAC_PCMDATA		*(volatile unsigned *)0x5b000018
#define rAC_MICDATA		*(volatile unsigned *)0x5b00001C

#define AC_PCMDATA		0x5b000018
#define AC_MICDATA		0x5b00001C

// I/O PORT 
#define rGPACON    (*(volatile unsigned *)0x56000000)	//Port A control
#define rGPADAT    (*(volatile unsigned *)0x56000004)	//Port A data

#define rGPBCON    (*(volatile unsigned *)0x56000010)	//Port B control
#define rGPBDAT    (*(volatile unsigned *)0x56000014)	//Port B data
#define rGPBUP     (*(volatile unsigned *)0x56000018)	//Pull-up control B

#define rGPCCON    (*(volatile unsigned *)0x56000020)	//Port C control
#define rGPCDAT    (*(volatile unsigned *)0x56000024)	//Port C data
#define rGPCUP     (*(volatile unsigned *)0x56000028)	//Pull-up control C

#define rGPDCON    (*(volatile unsigned *)0x56000030)	//Port D control
#define rGPDDAT    (*(volatile unsigned *)0x56000034)	//Port D data
#define rGPDUP     (*(volatile unsigned *)0x56000038)	//Pull-up control D

#define rGPECON    (*(volatile unsigned *)0x56000040)	//Port E control
#define rGPEDAT    (*(volatile unsigned *)0x56000044)	//Port E data
#define rGPEUP     (*(volatile unsigned *)0x56000048)	//Pull-up control E

#define rGPFCON    (*(volatile unsigned *)0x56000050)	//Port F control
#define rGPFDAT    (*(volatile unsigned *)0x56000054)	//Port F data
#define rGPFUP     (*(volatile unsigned *)0x56000058)	//Pull-up control F

#define rGPGCON    (*(volatile unsigned *)0x56000060)	//Port G control
#define rGPGDAT    (*(volatile unsigned *)0x56000064)	//Port G data
#define rGPGUP     (*(volatile unsigned *)0x56000068)	//Pull-up control G

#define rGPHCON    (*(volatile unsigned *)0x56000070)	//Port H control
#define rGPHDAT    (*(volatile unsigned *)0x56000074)	//Port H data
#define rGPHUP     (*(volatile unsigned *)0x56000078)	//Pull-up control H

#define rGPJCON    (*(volatile unsigned *)0x560000d0)	//Port J control
#define rGPJDAT    (*(volatile unsigned *)0x560000d4)	//Port J data
#define rGPJUP     (*(volatile unsigned *)0x560000d8)	//Pull-up control J

#define rMISCCR    (*(volatile unsigned *)0x56000080)	//Miscellaneous control
#define rDCLKCON   (*(volatile unsigned *)0x56000084)	//DCLK0/1 control
#define rEXTINT0   (*(volatile unsigned *)0x56000088)	//External interrupt control register 0
#define rEXTINT1   (*(volatile unsigned *)0x5600008c)	//External interrupt control register 1
#define rEXTINT2   (*(volatile unsigned *)0x56000090)	//External interrupt control register 2
#define rEINTFLT0  (*(volatile unsigned *)0x56000094)	//Reserved
#define rEINTFLT1  (*(volatile unsigned *)0x56000098)	//Reserved
#define rEINTFLT2  (*(volatile unsigned *)0x5600009c)	//External interrupt filter control register 2
#define rEINTFLT3  (*(volatile unsigned *)0x560000a0)	//External interrupt filter control register 3
#define rEINTMASK  (*(volatile unsigned *)0x560000a4)	//External interrupt mask
#define rEINTPEND  (*(volatile unsigned *)0x560000a8)	//External interrupt pending
#define rGSTATUS0  (*(volatile unsigned *)0x560000ac)	//External pin status
#define rGSTATUS1  (*(volatile unsigned *)0x560000b0)	//Chip ID(0x32440000)
#define rGSTATUS2  (*(volatile unsigned *)0x560000b4)	//Reset type
#define rGSTATUS3  (*(volatile unsigned *)0x560000b8)	//Saved data0(32-bit) before entering POWER_OFF mode 
#define rGSTATUS4  (*(volatile unsigned *)0x560000bc)	//Saved data0(32-bit) before entering POWER_OFF mode 

// Added for 2440
#define rFLTOUT		(*(volatile unsigned *)0x560000c0)	// Filter output(Read only)
#define rDSC0		(*(volatile unsigned *)0x560000c4)	// Strength control register 0
#define rDSC1		(*(volatile unsigned *)0x560000c8)	// Strength control register 1
#define rMSLCON		(*(volatile unsigned *)0x560000cc)	// Memory sleep control register

// RTC
#ifdef __BIG_ENDIAN
#define rRTCCON    (*(volatile unsigned char *)0x57000043)	//RTC control
#define rTICNT     (*(volatile unsigned char *)0x57000047)	//Tick time count
#define rRTCALM    (*(volatile unsigned char *)0x57000053)	//RTC alarm control
#define rALMSEC    (*(volatile unsigned char *)0x57000057)	//Alarm second
#define rALMMIN    (*(volatile unsigned char *)0x5700005b)	//Alarm minute
#define rALMHOUR   (*(volatile unsigned char *)0x5700005f)	//Alarm Hour
#define rALMDATE   (*(volatile unsigned char *)0x57000063)	//Alarm date   //edited by junon
#define rALMMON    (*(volatile unsigned char *)0x57000067)	//Alarm month
#define rALMYEAR   (*(volatile unsigned char *)0x5700006b)	//Alarm year
#define rRTCRST    (*(volatile unsigned char *)0x5700006f)	//RTC round reset
#define rBCDSEC    (*(volatile unsigned char *)0x57000073)	//BCD second
#define rBCDMIN    (*(volatile unsigned char *)0x57000077)	//BCD minute
#define rBCDHOUR   (*(volatile unsigned char *)0x5700007b)	//BCD hour
#define rBCDDATE   (*(volatile unsigned char *)0x5700007f)	//BCD date  //edited by junon
#define rBCDDAY    (*(volatile unsigned char *)0x57000083)	//BCD day   //edited by junon
#define rBCDMON    (*(volatile unsigned char *)0x57000087)	//BCD month
#define rBCDYEAR   (*(volatile unsigned char *)0x5700008b)	//BCD year

#else //Little Endian
#define rRTCCON    (*(volatile unsigned char *)0x57000040)	//RTC control
#define rTICNT     (*(volatile unsigned char *)0x57000044)	//Tick time count
#define rRTCALM    (*(volatile unsigned char *)0x57000050)	//RTC alarm control
#define rALMSEC    (*(volatile unsigned char *)0x57000054)	//Alarm second
#define rALMMIN    (*(volatile unsigned char *)0x57000058)	//Alarm minute
#define rALMHOUR   (*(volatile unsigned char *)0x5700005c)	//Alarm Hour
#define rALMDATE   (*(volatile unsigned char *)0x57000060)	//Alarm date  // edited by junon
#define rALMMON    (*(volatile unsigned char *)0x57000064)	//Alarm month
#define rALMYEAR   (*(volatile unsigned char *)0x57000068)	//Alarm year
#define rRTCRST    (*(volatile unsigned char *)0x5700006c)	//RTC round reset
#define rBCDSEC    (*(volatile unsigned char *)0x57000070)	//BCD second
#define rBCDMIN    (*(volatile unsigned char *)0x57000074)	//BCD minute
#define rBCDHOUR   (*(volatile unsigned char *)0x57000078)	//BCD hour
#define rBCDDATE   (*(volatile unsigned char *)0x5700007c)	//BCD date  //edited by junon
#define rBCDDAY    (*(volatile unsigned char *)0x57000080)	//BCD day   //edited by junon
#define rBCDMON    (*(volatile unsigned char *)0x57000084)	//BCD month
#define rBCDYEAR   (*(volatile unsigned char *)0x57000088)	//BCD year
#endif  //RTC


// ADC
#define rADCCON    (*(volatile unsigned *)0x58000000)	//ADC control
#define rADCTSC    (*(volatile unsigned *)0x58000004)	//ADC touch screen control
#define rADCDLY    (*(volatile unsigned *)0x58000008)	//ADC start or Interval Delay
#define rADCDAT0   (*(volatile unsigned *)0x5800000c)	//ADC conversion data 0
#define rADCDAT1   (*(volatile unsigned *)0x58000010)	//ADC conversion data 1
#define rADCUPDN   (*(volatile unsigned *)0x58000014)	//Stylus Up/Down interrupt status
	
	
// SPI       	
#define rSPCON0    (*(volatile unsigned *)0x59000000)	//SPI0 control
#define rSPSTA0    (*(volatile unsigned *)0x59000004)	//SPI0 status
#define rSPPIN0    (*(volatile unsigned *)0x59000008)	//SPI0 pin control
#define rSPPRE0    (*(volatile unsigned *)0x5900000c)	//SPI0 baud rate prescaler
#define rSPTDAT0   (*(volatile unsigned *)0x59000010)	//SPI0 Tx data
#define rSPRDAT0   (*(volatile unsigned *)0x59000014)	//SPI0 Rx data
	
#define rSPCON1    (*(volatile unsigned *)0x59000020)	//SPI1 control
#define rSPSTA1    (*(volatile unsigned *)0x59000024)	//SPI1 status
#define rSPPIN1    (*(volatile unsigned *)0x59000028)	//SPI1 pin control
#define rSPPRE1    (*(volatile unsigned *)0x5900002c)	//SPI1 baud rate prescaler
#define rSPTDAT1   (*(volatile unsigned *)0x59000030)	//SPI1 Tx data
#define rSPRDAT1   (*(volatile unsigned *)0x59000034)	//SPI1 Rx data


// SD Interface
#define rSDICON     (*(volatile unsigned *)0x5a000000)	//SDI control
#define rSDIPRE     (*(volatile unsigned *)0x5a000004)	//SDI baud rate prescaler
#define rSDICARG    (*(volatile unsigned *)0x5a000008)	//SDI command argument
#define rSDICCON    (*(volatile unsigned *)0x5a00000c)	//SDI command control
#define rSDICSTA    (*(volatile unsigned *)0x5a000010)	//SDI command status
#define rSDIRSP0    (*(volatile unsigned *)0x5a000014)	//SDI response 0
#define rSDIRSP1    (*(volatile unsigned *)0x5a000018)	//SDI response 1
#define rSDIRSP2    (*(volatile unsigned *)0x5a00001c)	//SDI response 2
#define rSDIRSP3    (*(volatile unsigned *)0x5a000020)	//SDI response 3
#define rSDIDTIMER  (*(volatile unsigned *)0x5a000024)	//SDI data/busy timer
#define rSDIBSIZE   (*(volatile unsigned *)0x5a000028)	//SDI block size
#define rSDIDCON    (*(volatile unsigned *)0x5a00002c)	//SDI data control
#define rSDIDCNT    (*(volatile unsigned *)0x5a000030)	//SDI data remain counter
#define rSDIDSTA    (*(volatile unsigned *)0x5a000034)	//SDI data status
#define rSDIFSTA    (*(volatile unsigned *)0x5a000038)	//SDI FIFO status
#define rSDIIMSK    (*(volatile unsigned *)0x5a00003c)	//SDI interrupt mask. edited for 2440A

#ifdef __BIG_ENDIAN  /* edited for 2440A */
#define rSDIDAT    	(*(volatile unsigned *)0x5a00004c)	//SDI data
#define SDIDAT     0x5a00004c  	
#else  	// Little Endian	
#define rSDIDAT    	(*(volatile unsigned *)0x5a000040)	//SDI data 
#define SDIDAT     0x5a000040  
#endif	//SD Interface


// Exception vector
#define pISR_RESET	(*(unsigned *)(_ISR_STARTADDRESS+0x0))
#define pISR_UNDEF	(*(unsigned *)(_ISR_STARTADDRESS+0x4))
#define pISR_SWI	(*(unsigned *)(_ISR_STARTADDRESS+0x8))
#define pISR_PABORT	(*(unsigned *)(_ISR_STARTADDRESS+0xc))
#define pISR_DABORT	(*(unsigned *)(_ISR_STARTADDRESS+0x10))
#define pISR_RESERVED	(*(unsigned *)(_ISR_STARTADDRESS+0x14))
#define pISR_IRQ	(*(unsigned *)(_ISR_STARTADDRESS+0x18))
#define pISR_FIQ	(*(unsigned *)(_ISR_STARTADDRESS+0x1c))
// Interrupt vector
#define pISR_EINT0	(*(unsigned *)(_ISR_STARTADDRESS+0x20))
#define pISR_EINT1	(*(unsigned *)(_ISR_STARTADDRESS+0x24))
#define pISR_EINT2	(*(unsigned *)(_ISR_STARTADDRESS+0x28))
#define pISR_EINT3	(*(unsigned *)(_ISR_STARTADDRESS+0x2c))
#define pISR_EINT4_7	(*(unsigned *)(_ISR_STARTADDRESS+0x30))
#define pISR_EINT8_23	(*(unsigned *)(_ISR_STARTADDRESS+0x34))
#define pISR_CAM	(*(unsigned *)(_ISR_STARTADDRESS+0x38))	// Added for 2440.
#define pISR_BAT_FLT	(*(unsigned *)(_ISR_STARTADDRESS+0x3c))	
#define pISR_TICK	(*(unsigned *)(_ISR_STARTADDRESS+0x40))	
#define pISR_WDT_AC97	(*(unsigned *)(_ISR_STARTADDRESS+0x44))	//Changed to pISR_WDT_AC97 for 2440A 
#define pISR_TIMER0	(*(unsigned *)(_ISR_STARTADDRESS+0x48))
#define pISR_TIMER1	(*(unsigned *)(_ISR_STARTADDRESS+0x4c))
#define pISR_TIMER2	(*(unsigned *)(_ISR_STARTADDRESS+0x50))
#define pISR_TIMER3	(*(unsigned *)(_ISR_STARTADDRESS+0x54))
#define pISR_TIMER4	(*(unsigned *)(_ISR_STARTADDRESS+0x58))
#define pISR_UART2	(*(unsigned *)(_ISR_STARTADDRESS+0x5c))
#define pISR_LCD	(*(unsigned *)(_ISR_STARTADDRESS+0x60))
#define pISR_DMA0	(*(unsigned *)(_ISR_STARTADDRESS+0x64))
#define pISR_DMA1	(*(unsigned *)(_ISR_STARTADDRESS+0x68))
#define pISR_DMA2	(*(unsigned *)(_ISR_STARTADDRESS+0x6c))
#define pISR_DMA3	(*(unsigned *)(_ISR_STARTADDRESS+0x70))
#define pISR_SDI	(*(unsigned *)(_ISR_STARTADDRESS+0x74))
#define pISR_SPI0	(*(unsigned *)(_ISR_STARTADDRESS+0x78))
#define pISR_UART1	(*(unsigned *)(_ISR_STARTADDRESS+0x7c))
#define pISR_NFCON	(*(unsigned *)(_ISR_STARTADDRESS+0x80))	// Added for 2440.
#define pISR_USBD	(*(unsigned *)(_ISR_STARTADDRESS+0x84))
#define pISR_USBH	(*(unsigned *)(_ISR_STARTADDRESS+0x88))
#define pISR_IIC	(*(unsigned *)(_ISR_STARTADDRESS+0x8c))
#define pISR_UART0	(*(unsigned *)(_ISR_STARTADDRESS+0x90))
#define pISR_SPI1	(*(unsigned *)(_ISR_STARTADDRESS+0x94))
#define pISR_RTC	(*(unsigned *)(_ISR_STARTADDRESS+0x98))
#define pISR_ADC	(*(unsigned *)(_ISR_STARTADDRESS+0x9c))


// PENDING BIT
#define BIT_EINT0	(0x1)
#define BIT_EINT1	(0x1<<1)
#define BIT_EINT2	(0x1<<2)
#define BIT_EINT3	(0x1<<3)
#define BIT_EINT4_7	(0x1<<4)
#define BIT_EINT8_23	(0x1<<5)
#define BIT_CAM		(0x1<<6)	// Added for 2440.
#define BIT_BAT_FLT	(0x1<<7)
#define BIT_TICK	(0x1<<8)
#define BIT_WDT_AC97	(0x1<<9)	// Changed from BIT_WDT to BIT_WDT_AC97 for 2440A  
#define BIT_TIMER0	(0x1<<10)
#define BIT_TIMER1	(0x1<<11)
#define BIT_TIMER2	(0x1<<12)
#define BIT_TIMER3	(0x1<<13)
#define BIT_TIMER4	(0x1<<14)
#define BIT_UART2	(0x1<<15)
#define BIT_LCD		(0x1<<16)
#define BIT_DMA0	(0x1<<17)
#define BIT_DMA1	(0x1<<18)
#define BIT_DMA2	(0x1<<19)
#define BIT_DMA3	(0x1<<20)
#define BIT_SDI		(0x1<<21)
#define BIT_SPI0	(0x1<<22)
#define BIT_UART1	(0x1<<23)
#define BIT_NFCON	(0x1<<24)	//Added for 2440.
#define BIT_USBD	(0x1<<25)
#define BIT_USBH	(0x1<<26)
#define BIT_IIC		(0x1<<27)
#define BIT_UART0	(0x1<<28)
#define BIT_SPI1	(0x1<<29)
#define BIT_RTC		(0x1<<30)
#define BIT_ADC		(0x1<<31)
#define BIT_ALLMSK	(0xffffffff)

#define BIT_SUB_ALLMSK	(0x7fff) 	//Changed from 0x7ff to 0x7fff for 2440A 
#define BIT_SUB_AC97	(0x1<<14)	//Added for 2440A 
#define BIT_SUB_WDT	(0x1<<13)	//Added for 2440A 
#define BIT_SUB_CAM_P	(0x1<<12)	//edited for 2440A.
#define BIT_SUB_CAM_C	(0x1<<11)	//edited for 2440A
#define BIT_SUB_ADC	(0x1<<10)	
#define BIT_SUB_TC	(0x1<<9)
#define BIT_SUB_ERR2	(0x1<<8)
#define BIT_SUB_TXD2	(0x1<<7)
#define BIT_SUB_RXD2	(0x1<<6)
#define BIT_SUB_ERR1	(0x1<<5)
#define BIT_SUB_TXD1	(0x1<<4)
#define BIT_SUB_RXD1	(0x1<<3)
#define BIT_SUB_ERR0	(0x1<<2)
#define BIT_SUB_TXD0	(0x1<<1)
#define BIT_SUB_RXD0	(0x1<<0)

//Wait until rINTPND is changed for the case that the ISR is very short.
/*
#define	ClearPending(bit) {\
			rSRCPND = bit;\
			rINTPND = bit;\
			rINTPND;\
		}
*/

#define	EnableIrq(bit)		rINTMSK 	&= ~(bit)
#define	DisableIrq(bit)		rINTMSK 	|= 	(bit)
#define	EnableSubIrq(bit)	rINTSUBMSK 	&= ~(bit)
#define	DisableSubIrq(bit)	rINTSUBMSK	|=	(bit)

__inline void ClearPending(int bit)	//通过置1的方式来清空中断未决寄存器
{
	register i;		
	rSRCPND = bit;			//清空相应位的源未决寄存器
	rINTPND = bit;			//清空相应位的中断未决寄存器
	i = rINTPND;			//空操作相当于NOP
}

__inline void ClearSubPending(int bit)	//通过置1的方式来清空子中断源未决寄存器
{
	register i;
	rSUBSRCPND = bit;		//清空相应位的子中断源未决寄存器
	i = rINTPND;			//空操作相当于NOP
}                       
//Wait until rINTPND is changed for the case that the ISR is very short.

#ifdef __cplusplus
}
#endif
#endif  //__2440ADDR_H__
~~~

## timer

![timer_overview.png](/images/timer/timer_overview.png)



[arm]:http://www.arm.com/
