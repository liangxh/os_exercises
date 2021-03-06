# lab1 SPOC思考题

## 个人思考题

NOTICE
- 有"w2l2"标记的题是助教要提交到学堂在线上的。
- 有"w2l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。
---

请描述ucore OS配置和驱动外设时钟的准备工作包括哪些步骤？ (w2l2)
```
  + 采分点：说明了ucore OS在让外设时钟正常工作的主要准备工作
  - 答案没有涉及如下3点；（0分）
  - 描述了对IDT的初始化，包了针时钟中断的中断描述符的设置（1分）
  - 除第二点外，进一步描述了对8259中断控制器的初始过程（2分）
  - 除上述两点外，进一步描述了对8253时钟外设的初始化，或描述了对EFLAG操作使能中断（3分）
 ```
- [x]  

于init.c kern_init()中, 
首先调用pic_init(), 对8259A中断控制器进行初始化, 先对中断屏蔽, 再设置master和slave.
后调用idt_init(), 对中断向量逐个进行初始化, 并以lidt设置IDT的所在地址 (  lidt(&idt_pd); ).
最后调用clock_init(), 设置8253时钟, 并对时钟使能(  pic_enable(IRQ_TIMER); ).

lab1中完成了对哪些外设的访问？ (w2l2)
 ```
  + 采分点：说明了ucore OS访问的外设
  - 答案没有涉及如下3点；（0分）
  - 说明了时钟（1分）
  - 除第二点外，进一步说明了串口（2分）
  - 除上述两点外，进一步说明了并口，或说明了CGA，或说明了键盘（3分）
 ```
- [x]  

根据kern_init, 有相关外设的初始化:
cons_init(), 其中有cga_init()对CGA(作为显示器), 有serial_init()对串口, 有kbd_init()对键盘初始化;有clock_init()对时钟进行初始化. 另外还有硬盘, 使读写数据.


lab1中的cprintf函数最终通过哪些外设完成了对字符串的输出？ (w2l2)
 ```
  + 采分点：说明了cprintf函数用到的3个外设
  - 答案没有涉及如下3点；（0分）
  - 说明了串口（1分）
  - 除第二点外，进一步说明了并口（2分）
  - 除上述两点外，进一步说明了CGA（3分）
 ```
- [x]  

cprintf的函数调用过程如下:
int cprintf() →int vcprintf() → void cputch() → cons_putc() → lpt_putc(c); cga_putc(c); serial_putc(c);
其中lpt_putc把c输至并口, 然后cga_putc(c)把c打印到屏幕, 最后serial_putc把c输至串口. 对字符串中每个字符都依次进行这三步.

---

## 小组思考题

---

lab1中printfmt函数用到了可变参，请参考写一个小的linux应用程序，完成实现定义和调用一个可变参数的函数。(spoc)
- [x]  

	#include <stdio.h>
	#include <stdlib.h>
	#include <stdarg.h>
	
	void myprintf(char* format, ...){
		char* flag = format;
		va_list pArg;
		va_start(pArg, format);
	
		while (*flag){
			switch(*flag){
			case 'd': printf("INT : %d\n", *(int*)pArg); va_arg(pArg, int); break;
			case 'f': printf("FLOAT : %lf\n", *(double*)pArg); va_arg(pArg, double); break;
			case 'c': printf("CHAR : %c\n", *(char*)pArg); va_arg(pArg, int); break;	
			}
			flag++;	
		}
	}
	
	
	void myprintf2(char* format, ...){
		char* flag = format;
		char* pArg = (char*)(&format +1);
		while (*flag){
			switch(*flag){
			case 'd': printf("INT : %d\n", *(int*)pArg); pArg += sizeof(int); break;
			case 'f': printf("FLOAT : %lf\n", *(double*)pArg); pArg += sizeof(double); break;
			case 'c': printf("CHAR : %c\n", *(char*)pArg); pArg += sizeof(int); break;	
			}
			flag++;	
		}
	}
	
	int main(int argc, char** argv){
		myprintf("fdcdf", 10.4, 4, 'A', 10, 4.0);	
		myprintf2("fdcdf", 10.4, 4, 'A', 10, 4.0);	
		return 0;
	}



如果让你来一个阶段一个阶段地从零开始完整实现lab1（不是现在的填空考方式），你的实现步骤是什么？（比如先实现一个可显示字符串的bootloader（描述一下要实现的关键步骤和需要注意的事项），再实现一个可加载ELF格式文件的bootloader（再描述一下进一步要实现的关键步骤和需要注意的事项）...） (spoc)
- [x]  

> 步骤如下:
1) 实现可以使cga显示字符串的bootloader, 使显示执行过程中的信息及调试信息.
2) 实现读取硬盘的功能, 确保正确读取内容.
3) 尝试加载ELF格式文件.
4) 进行ucore入口.
5) 实现用户态和内核态转换, 为进入中断服务作准备.
6) 实现键盘中断, 使运行过程中可与系统进行交互, 更方便于后续调试.
7) 实现其他中断.


如何能获取一个系统调用的调用次数信息？如何可以获取所有系统调用的调用次数信息？请简要说明可能的思路。(spoc)
- [x]  

> 1) 对每个系统调用配置一个记数器(寄存器), 并把调用记录保存在文件中即可.
2) strace -c ./a.out
不理太解问题什么意思(?)

如何裁减lab1, 实现一个可显示字符串"THU LAB1"且依然能够正确加载ucore OS的bootloader？如果不能完成实现，请说明理由。
- [x]  

> 

对于ucore_lab中的labcodes/lab1，我们知道如果在qemu中执行，可能会出现各种稀奇古怪的问题，比如reboot，死机，黑屏等等。请通过qemu的分析功能来动态分析并回答lab1是如何执行并最终为什么会出现这种情况？
- [x]  

> 

对于ucore_lab中的labcodes/lab1,如果出现了reboot，死机，黑屏等现象，请思考设计有效的调试方法来分析常在现背后的原因。
- [x]  

> 

---

## 开放思考题

---

如何修改lab1, 实现在出现除零错误异常时显示一个字符串的异常服务例程的lab1？
- [x]  

> 


在lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
- [x]  

>

GRUB是一个通用的bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
- [x]  

>


如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？
- [x]  

>  

---
