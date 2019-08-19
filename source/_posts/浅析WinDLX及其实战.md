---
title: 浅析WinDLX及其实战
date: 2016-04-23 20:38:05
tags:
	- WinDLX
	- 指令集流水线
categories:
	- 计算机体系结构
---


# WinDLX简介 #

> WinDLX模拟器是一个基于Windows操作系统的图形化、交互式DLX指令集模拟器，能够演示DLX指令集流水线是如何工作的。该模拟器可以装载DLX汇编语言程序（后缀为“.s”的文件），然后单步、设断点或是连续执行该程序。CPU的寄存器、流水线、I/O和存储器都用图形进行表示，以形象生动的方式描述DLX流水线的工作过程。模拟器还提供了对流水线操作的统计功能，便于对流水线进行性能分析。

# 环境配置 #

- 主要运行在win3.0上，16bit软件，<font color="#FF0000">不支持64bit系统</font>。所以在使用前需要确保系统环境为32bit，方可兼容此软件，否则需要虚拟机环境。
- 为WinDLX创建目录，例如D:\WINDLX，注意受编译的约束，路径名中<font color="#FF0000">不可以包含汉字</font>。
- 解压WinDLX软件包或拷贝所有的WinDLX文件（至少包含 windlx.exe，windlx.hlp，fact.s 和input.s ）到这个WinDLX 目录。

<!-- more -->

# WinDLX子窗口 #
## Pipeline ##
- IF取指阶段
- ID译码阶段
- MEM访存阶段
- WB写回阶段
- intEX整型加阶段执行
- faddEX浮点加执行阶段
- fmul浮点乘执行阶段
- fdivEX浮点除执行阶段


![](http://i.imgur.com/NvyjyDO.png)

## Code ##

![](http://i.imgur.com/3XBX80r.png)
> 三栏信息，从左到右依次为：地址 (符号或数字)、命令的十六进制机器代码和汇编命令。

![](http://i.imgur.com/1qM62YH.png)
> 按下 F7 键，模拟就向前执行一步，第一行的颜色变成黄色，下一行变成棕色，依次向下执行。

## Clock Cycle Diagram ##

![](http://i.imgur.com/VqE29fz.png)
> 横坐标代表时钟周期，纵坐标为指令集

## Breakpoint ##

> 此为增设断点的子窗口

## Register ##

> 此为寄存器的子窗口

## Statistics ##

> 分析窗口，主要关注RAW stall数据相关，Structral stall结构相关，Control stall控制相关。
> 
> 下文中会主要分析这三者的产生原因及其消除方法。

# WinDLX指令分类 #

1. Load/Store指令：除R0之外，所有通用寄存器与浮点寄存器都可以作为加载或存储之用
2. ALU操作指令：所有的ALU操作都是寄存器–寄存器指令。加，减，与，或，异或，移位，比较指令比较两个寄存器(=, !=, <, >, =<, =>)，如果条件为真，则在目标寄存器置1，否则置0。
3. 分支/跳转指令：所有分支都是条件分支，分支条件由指令测试寄存器为零或非零来指定。
4. 浮点运算指令：浮点加，减，乘，除(浮点格式为IEEE754)

# Trap机制 #

> Trap在DLX程序和系统I/O之间建立了接口。在WinDLX中共定义了5种Trap。

- 0对于Trap指令来说是无效参数，Trap  0用来结束程序
- Trap  1可以为读写打开一个文件，打开的文件在DLX重置或结束之后自动关闭
- Trap  2关闭由Trap  1打开的文件
- Trap  3从文件块读，读入一个文件块或者标准输入的一行
- Trap  4向文件块写，向文存储器或标准输出块写

# 实例讲解 #

## 程序说明 ##

功能：卡特兰数求解<br>
输入：输入一个大于0的自然数n，程序会计算第n项的卡特兰数<br>

测试用例：

- 输入:n=7
- 输出：132

## 卡特兰数讲解 ##

- 一般公式

$$ C_n = \frac{1}{n+1} \dbinom{2n}{n} = \frac{(2n)!}{(n+1)!n!} $$

- 递推公式

![](https://upload.wikimedia.org/math/6/2/1/6217b3c99a3243afcd5d8dbd58186822.png)

它也满足

![](https://upload.wikimedia.org/math/8/a/4/8a49332e4a46b3a2c7accec81160f5e3.png)

卡塔兰数的渐近增长为

![](https://upload.wikimedia.org/math/d/1/9/d19d674c2da2a8c3864d97e0326ba9b4.png)

前20项为（OEIS中的数列A000108）：1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786, 208012, 742900, 2674440, 9694845, 35357670, 129644790, 477638700, 1767263190

## 样例程序 ##

求解第n项卡特兰数的WinDLX样例程序

    ;*********** WINDLX Ex.3: Factorial		*************
	;*********** (c) 1991 Günther Raidl		*************
	;*********** Modified: 1992 Maziar Khosravipour *************
	; class:
	; number:
	; author:
	;--------------------------------------------------------------------------
	; Program begin at symbol main
	; requires module INPUT
	; read a number from stdin and calculate the catalan number (type: double)
	; the result is written to stdout
	;--------------------------------------------------------------------------

		.data
	Prompt: 	.asciiz 	"An integer value >1 : "

	PrintfFormat:	.asciiz 	"Factorial = %g\n\n"
		.align		2
	PrintfPar:	.word		PrintfFormat
	PrintfValue:	.space		1024


		.text
		.global	main
	main:
		;*** Read value from stdin into R1
		addi		r1,r0,Prompt
		jal		InputUnsigned
		
		;*** init values
		subi		r1,r1,1
		movi2fp 	f10,r1		;R1 -> D0	D0..Count register
		cvti2d		f0,f10
		addi		r2,r0,1 	;1 -> D2	D2..result
		movi2fp		f11,r2
		cvti2d		f2,f11
		movd		f4,f2		;1-> D4 	D4..Constant 1
		addd		f6,f0,f4	;D0+1 -> D6
		addd		f8,f0,f0 	;2*D0 -> D8
		addi		r2,r0,1 	;1 -> D10	D10..result
		movi2fp		f11,r2
		cvti2d		f10,f11
		
		;*** Break loop if D0 = 1
	Loop1:	led		f0,f4		;D0<=1 ?
		bfpt		Loop2
		
		;*** Multiplication and next loop1
		multd		f2,f2,f0
		subd		f0,f0,f4
		j		Loop1

		;*** Break loop if D8 = 1
	Loop2:	led		f8,f4		;D8<=1 ?
		bfpt		Finish
		
		;*** Multiplication and next loop2
		multd		f10,f10,f8
		subd		f8,f8,f4
		;multd		f10,f10,f8
		;subd		f8,f8,f4
		;multd		f10,f10,f8
		;subd		f8,f8,f4
		;multd		f10,f10,f8
		;subd		f8,f8,f4
		j		Loop2


	Finish: 	;*** write result to stdout
		multd		f2,f2,f2
		multd		f6,f6,f2
		divd		f10,f10,f6
		sd		PrintfValue,f10
		addi		r14,r0,PrintfPar
		trap		5
				
		;*** end
		trap		0	
		
## 三种相关 ##

- 数据相关：当一条指令需要用到前面指令的执行结果，而这些指令均在流水线中重叠执行时
- 结构相关：当指令在重叠执行过程中，硬件资源满足不了指令重叠执行的要求,发生资源冲突时
- 控制相关：当不满足条件时或者其他改变PC值的跳转指令，通常发生控制相关

## 消除方法 ##

- 数据相关：勾选Enable Forwading，采用重定向技术
- 结构相关：通过增加硬件的数目来解决结构相关
- 控制相关：为了解决控制相关，我们要在分支跳转指令里循环展开，通过代码的冗余来做到消除控制相关
