# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。


- BIOS 启动固件，计算机加电时直接跳过去执行的内容，放在最下面1MB的空间
	- 基本输入输出
	- 系统设置信息
	- 开机后自检
	- 系统自启动

	- 使用中断调用实现功能，只支持CPU实模式

- BIOS执行
	- 硬件自检
	- 硬件卡初始化
	- 系统BIOS
	- 按照指定顺序启动

	- 先加载主引导记录
		- 启动代码， 分区表， 结束标志 0x550xAA
		- 加载活动分区引导扇区并跳转
		- 分区引导扇区
			- 跳到启动代码
			- 文件格式
			- 启动代码 (记录真实的加载程序在哪里)
			- 0x55 0xAA
	- 加载程序
		- 加载配置信息
		- 加载操作系统内核


- 启动过程
	- BIOS从磁盘读引导扇区加载程序（512Byte)
		- 磁盘有多个分区，BIOS里面要记录分区信息和主引导信息
		- 从活动分区的主引导扇区加载加载程序
	- 跳到加载程序
	- 加载程序读操作系统
	- 跳到操作系统




## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？
读入的是加载程序。
一方面因为此时CPU运行在实模式，可使用的内存只有1MB，因此不能完全读入操作系统内核映像而是通过加载程序来实现
另一方面，操作系统可能被保存在不同的文件系统，不同的分区之下，通过加载程序来识别不同的文件系统和分区来进行系统内核加载
并且加载程序还需要进行一些自检操作

- 比较UEFI和BIOS的区别。
UEFI需要检查引导记录的签名，从而保证安全性。（可信启动流程）
UEFI希望能够实现所有平台一致的系统启动，而BIOS需要考虑更多的向前兼容。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？
0x550xAA
- x86中在UEFI中的可信启动有什么作用？
保证只有可信的代码才会被执行。
- RV中BBL的启动过程大致包括哪些内容？

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
中断是I/O对操作系统的请求
异常是指令执行出错
系统调用是应用程序主动对操作系统提出的请求

-  中断、异常和系统调用的处理流程有什么异同？
中断是异步处理
异常是同步处理
系统调用可异步可同步

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
	- 进程管理
   		- sys_exit
    	- sys_fork
    	- sys_wait
    	- sys_exec
    	- sys_yield
    	- sys_kill
    	- sys_getpid
    	- sys_sleep
    - I/O
    	- sys_putc
    - 内存管理
    	- sys_pgdir
    - 文件管理
    	- sys_fstat
    	- sys_fsync
    	- sys_seek
    	- sys_dup
    	- sys_getcwd
    	- sys_getdirentry
    	- sys_read
    	- sys_write
    	- sys_open
    	- sys_close
    - 其他
    	- sys_gettime
    	- sys_lab6_set_priority



## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
```
void
syscall(void) {
    struct trapframe *tf = current->tf;
    uint32_t arg[5];
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    print_trapframe(tf);
    panic("undefined syscall %d, pid = %d, name = %s.\n",
            num, current->pid, current->name);
}
```
系统调用的触发方式和中断，异常相同，都通过trap进入。中断号被放在%eax中，并在进入trap处理时被放入到中断帧中。返回值也放在%eax中。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
用户态中，调用系统调用调用的函数为
```
static inline int
syscall(int num, ...) {
    va_list ap;
    va_start(ap, num);
    uint32_t a[MAX_ARGS];
    int i, ret;
    for (i = 0; i < MAX_ARGS; i ++) {
        a[i] = va_arg(ap, uint32_t);
    }
    va_end(ap);

    asm volatile (
        "int %1;"
        : "=a" (ret)
        : "i" (T_SYSCALL),
          "a" (num),
          "d" (a[0]),
          "c" (a[1]),
          "b" (a[2]),
          "D" (a[3]),
          "S" (a[4])
        : "cc", "memory");
    return ret;
}
```
直接被汇编成int指令，并指定了系统调用参数保存的地址。
中断被触发后，操作系统跳转到中断处理例程，完成现场保护等过程后进入到内核态部分
```
void
trap(struct trapframe *tf) {
    // dispatch based on what type of trap occurred
    // used for previous projects
    if (current == NULL) {
        trap_dispatch(tf);
    }
    else {
        // keep a trapframe chain in stack
        struct trapframe *otf = current->tf;
        current->tf = tf;
    
        bool in_kernel = trap_in_kernel(tf);
    
        trap_dispatch(tf);
    
        current->tf = otf;
        if (!in_kernel) {
            if (current->flags & PF_EXITING) {
                do_exit(-E_KILLED);
            }
            if (current->need_resched) {
                schedule();
            }
        }
    }
}
```
首先trap进行初步处理，然后进入到trap_dispatch()，然后进入到syscall()
```
void
syscall(void) {
    struct trapframe *tf = current->tf;
    uint32_t arg[5];
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    print_trapframe(tf);
    panic("undefined syscall %d, pid = %d, name = %s.\n",
            num, current->pid, current->name);
}
```
这里读取出来被保存的栈帧内容，并实际上执行syscall操作。

- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
 	- 系统调用是操作系统提供给用户程序的接口，工作在内核态。而函数调用是用户程序中进行的自定义接口，运行在用户态。
 	- 系统调用使用int和iret，函数调用使用call和ret
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
	- int 会进入内核态，将中断号压入栈中，然后调用__all_traps形成中断帧，然后调用trap()根据中断帧进行系统调用处理。
	- iret 会从内核态转换到用户态
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
