#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
 1. 比较UEFI和BIOS的区别。

> 1) UEFI使操作系统可以自动从预启动的操作环境中加载到一种操作系统上,　相比BIOS需要从硬盘多次读取扇区来加载操作系统要快得多;　
2) UEFI对引导记录的可信性进行检查,　只有通过检查的引导记录才会被读进去并赋予控制权,　减少了安全的风险;
3) UEFI本身具备文件系统的支持,　可直接读取FAT分区.
 
 2. 描述PXE的大致启动流程。

> 1) 通过UDP协议发送受限广播(255.255.255.255),目标端口67, 是DHCP命令中的DHCP discover,带PXEClient标志,并在UDP 68端口监听。
2) DHCP服务器在67端口收到请求后,通过UDP协议发送受限广播(255.255.255.255)，目标端口68,是DHCP命令中的DHCP offer,附带客户端ip,网关,tftp服务器ip, 子网掩码, 可引导的文件名等.
3) 客户机在68端口收到服务器回复的信息,客户机配置自身ip，从此时起，客户机可以使用单播通讯了。
4) 客户端通过tftp协议发送获取引导文件的命令,并将获取的引导文件存放在地址 0:7c00 （第3步中获取的引导文件名）.
5) 跳转到0:7c00开始执行引导文件中的代码,从此时起cpu控制权己交给引导文件.

## 3.2 系统启动流程
 1. 了解NTLDR的启动流程。

> NTLDR(全称NT Loader)是系统加载程序, 其文件是Windows NT/windows 2000/windows XP/windows server 2003的引导文件, 整个启动过程如下:
1) 电源自检程序开始运行;
2) 主引导记录被装入内存，并且程序开始执行;
3) 活动分区的引导扇区被装入内存;
4)　NTLDR从引导扇区被装入并初始化;
5) 将处理器的实模式改为32位平滑内存模式;
6) NTLDR开始运行在NTLDR内部的小文件系统驱动程序(用于读FAT或NTFS);
7) NTLDR读boot.ini文件;
8) NTLDR装载所选操作系统;
如果windows NT/windows 2000/windows XP/windows server 2003这些操作系统被选择，NTLDR运行Ntdetect。
对于其他的操作系统，NTLDR装载并运行Bootsect.dos然后向它传递控制。
windows NT过程结束。
9) Ntdetect搜索计算机硬件并将列表传送给NTLDR，以便将这些信息写进\\HKE Y_LOCAL_MACHINE\HARDWARE中;
10) 然后NTLDR装载Ntoskrnl.exe，Hal.dll和系统信息集合;
11) Ntldr搜索系统信息集合，并装载设备驱动配置以便设备在启动时开始工作;
12) Ntldr把控制权交给Ntoskrnl.exe，这时,启动程序结束,装载阶段开始.
 
2. 了解GRUB的启动流程。

> Grub的代码分成两部分,　一部分在第一扇区的主引导记录MBR中,　另一部分则在磁盘的某个位置中.
在计算机启动,　BIOS完成启动自检后
1) 读取MBR然后跳转至Grub的第一部分代码,　其中提供了第二部分代码的信息;
2) 跳转至Grub的第二部分代码,　此为核心部分,　负责读取/boot/grub/grub.conf文件;
3) 根据grub.conf显示操作系统的菜单,　提供用户交互界面;
4) 根据用户的选择把控制权交给内核程序, 由内核程序启动操作系统.
 
3. 比较NTLDR和GRUB的功能有差异。

> NTLDR的设计只供引导windows系统, 且只能引导在硬盘上的操作系统；
GRUB属于第三方操作系统引导器, 一方面支持不同的操作系统包括windows, linux, dos, 另一方面可以引导硬盘, 光盘, 网络上的操作系统, 灵活性要比NTLDR强.
 
4. 了解u-boot的功能。

> U-Boot用於加載內核, 它的啟動過程可分為兩個階段, 負責功能如下:
第一階段
1) 硬件設備的初始化, 包括串口, I/O設備等; 
2) 加載U-Boot第二階段代碼到內存空間; 
3) 設置棧; 
4) 跳转至第二階段代碼的入口; 
第二階段
1) 初始化本階段使用的硬件設備; 
2) 檢查系统內存映射; 
3) 將內核從閃存讀取至內存中; 
4) 為內核設置啟動參數; 
5) 調用內核.

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
 2. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)

```
  + 采分点：说明了Linux的大致数量（上百个），说明了Linux系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
  
 > Linux的系统调用共250多个, 其根据功能可分为以下各大类
一、进程控制： 
二、文件系统控制
  	1) 文件读写操作
	  2) 文件系统操作
三、系统控制
四、内存管理
五、网络管理
六、socket控制
七、用户管理
八、进程间通信
  	1) 信号
  	2) 消息
 	3) 管道
  	4) 信号量
  	5) 共享内存 
 
 
 3. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
 
 ```
  + 采分点：说明了ucore的大致数量（二十几个），说明了ucore系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
> Ucore中包括的系统调用(22个)及根据其功能分类如下
>
一、进程控制:
[SYS_exit]              sys_exit, // 中止进程 
[SYS_fork]              sys_fork, // 创建一个新进程
[SYS_wait]              sys_wait, // 等待子进程终止
[SYS_exec]              sys_exec, // 运行可执行文件
[SYS_yield]             sys_yield, // 进程主动让出处理器,并将自己等候调度队列队尾
[SYS_kill]              sys_kill, // 立即中止当前进程 
[SYS_getpid]            sys_getpid, // 获取进程标识号
[SYS_putc]              sys_putc,
[SYS_pgdir]             sys_pgdir,
[SYS_gettime]           sys_gettime, // 取时间
[SYS_lab6_set_priority] sys_lab6_set_priority,
[SYS_sleep]             sys_sleep,   // 睡眠
>
二、文件系统控制:
1、文件读写操作
[SYS_open]              sys_open,
[SYS_close]             sys_close,
[SYS_read]              sys_read,
[SYS_write]             sys_write,
[SYS_seek]              sys_seek,
[SYS_fsync]             sys_fsync,
[SYS_dup]               sys_dup,
2、文件系统操作
[SYS_fstat]             sys_fstat,
[SYS_getcwd]            sys_getcwd,
[SYS_getdirentry]       sys_getdirentry

 
## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 

 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 
> objdump 可以显示二进制文件可能带有的附加信息的附加信息，如-d指令可以对那些还有指令机器码的section进行反汇编并输出, -s则显示指定section的完整内容.
> nm 可以列出文件的符号清单.
> file 可以文件类型和属性信息, 如file Makefile 输出Makefile: makefile script, ASCII text, 显示为makefile脚本

> lab1-ex0.s的main如下
main:
        movl    $SYS_write,%eax
        movl    $STDOUT,%ebx
        movl    $hello,%ecx
        movl    $12,%edx
        int     $0x80
        ret
>显示出调用sys_write的过程, 首先把调用参数保存于寄存器中, 再透过int 0x80把控制权交还系统由它来完成打印$hello至stdout的操作.

 
 2. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 

 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
 > strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。
 
 > 用户程序中执行系统调用，先通过调用C库中的函数，函数中透过软中断INT 0x80进行系统调用, 此指令使跳转到一个预设的内核空间地址，它指向系统调用处理程序，即system_call函数. INT 0x80指令执行前，系统调用号会被放入eax寄存器中,system_call函数透过读取eax寄存器，将其乘以4以生成偏移地址，然后以sys_call_table为基址，加上偏移地址就可以得到具体的系统调用服务例程的地址了. 当系统调用完成后，把控制权交回到发起调用的用户进程前，内核会有一次调度. 如果发现有优先级更高的进程或当前进程的时间片用完，那么会选择优先级更高的进程或重新选择进程执行.
 
## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
 2. ucore的系统调用中返回结果的传递代码分析。
 3. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
 4. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
 2. 说明`int`、`iret`、`call`和`ret`的指令准确功能
 
