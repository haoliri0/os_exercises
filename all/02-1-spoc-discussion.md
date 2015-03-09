#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
 1. 比较UEFI和BIOS的区别。  

> 答：
    BIOS是固化到计算机主板上的程序，对于不同的启动标准它需要有不同的部分来分别对应。  
    UEFI是一个接口，它在所有平台上一致的提供操作系统的启动服务。

 1. 描述PXE的大致启动流程。  

 > 答：
    PXE是一个网络启动的标准，启动时先往BIOS里加网络协议栈，再连接服务器，从服务器上下载内核镜像来执行。

## 3.2 系统启动流程
 1. 了解NTLDR的启动流程。
 1. 了解GRUB的启动流程。
 1. 比较NTLDR和GRUB的功能有差异。
 1. 了解u-boot的功能。

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
 1. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)

```
  + 采分点：说明了Linux的大致数量（上百个），说明了Linux系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
> linux系统调用有200多个，可以在内核目录unistd.h中找到，按功能分类大致可分为：进程控制（如exit，pause），文件系统控制（stat，close，open），系统控制（sysinfo），内存管理（brk），网络管理（gethostid），socket控制（socket，bind），用户管理（getuid），进程间通信（ipc）

 1. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
 
 ```
  + 采分点：说明了ucore的大致数量（二十几个），说明了ucore系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 > uCore的系统调用有如下几个：sys_exit，sys_fork，sys_wait，sys_exec，sys_yield，sys_kill，sys_getpid，sys_putc，sys_pgdir， sys_gettime，sys_lab6_set_priority，sys_sleep，sys_open，sys_close，sys_read，sys_write，sys_seek，sys_fstat，sys_fsync，sys_getcwd，sys_getdirentry，sys_dup。  
uCore的系统调用的主要分类为：进程控制（如sys_exit），文件系统管理（sys_open），系统控制（sys_gettime）


## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 
 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 
> -- objdump指令是主要的作用是反汇编，但是调用objdump时必须加至少一个参数。例如：-f是返回文件头信息;-D是反汇编对应文件中所有的section;-d是反汇编需要执行指令的那些section。例如:objdump -d lab1-ex0.exe就会输出几千条汇编指令。  
-- nm是用来列出目标文件的符号清单。例如指令nm lab1-ex0.exe的输出中有如下一行：  
0000000000407658 T strlen  
其中T表示这是Global Text类型，strlen为相应的符号  
-- file是检测文件类型的命令。例如file lab1-ex0.exe它就会说明这是一个可执行文件。  
Linux系统调用主要通过终端（Terminal）内输入相应指令来实现。输入的指令需要满足一定格式，同时该指令必须存在于Linux的系统调用库中，才能实现对应的功能。  

 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 

 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
> strace常用来跟踪进程执行时的系统调用和所接收的信号  
系统调用分为两步。  首先为从用户空间到系统空间，不同平台有不同的指令来完成从用户空间到系统空间的转换，这类指令被称为操作系统陷入指令。对于Linux来说，通过软中断int0x80实现。通过该指令，系统会转到一个预设的内核空间地址。这个地址就是系统调用处理程序system_call函数。  其次是从system_call函数到系统调用服务例程。system_call函数通过系统调用号查找系统调用表sys_call_table。软中断指令INT 0x80执行时，系统调用号会被放入eax寄存器中，system_call函数可以读取eax寄存器获取，然后将其乘以4，生成偏移地址，然后以sys_call_table为基址，基址加上偏移地址，就可以得到具体的系统调用服务例程的地址。系统调用服务例程只会从堆栈里获取参数，所以在system_call执行前，会先将参数存放在寄存器中，system_call执行时会首先将这些寄存器压入堆栈。system_call退出后，用户可以从寄存器中获得（被修改过的）参数。


## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
 1. ucore的系统调用中返回结果的传递代码分析。
 1. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
 1. 以ucore lab8的answer为例，尝试修改并运行代码，分析ucore应用的系统调用执行过程。
 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
   1. 说明`int`、`iret`、`call`和`ret`的指令准确功能
 
