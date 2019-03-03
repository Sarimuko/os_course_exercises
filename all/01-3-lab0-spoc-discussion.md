# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```
我认为":"后面的数字是数字表示位数的意思

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \ //取off的低16位
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \ //取off的高16位
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？
执行上述指令后，从intr开始的64bit从高位开始为
0x00000F0000020003

intr是unsigned类型，实际上是截取最低32位，intr = 0x2003


### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

<<<<<<< HEAD
以下这段汇编代码来自于trap.s，用于发生自陷时从用户态跳转到内核态和处理完中断之后从内核态跳转到用户态。
  ```
  #include <memlayout.h>

  # vectors.S sends all traps here.
  .text
  .globl __alltraps
  __alltraps:
      # push registers to build a trap frame
      # therefore make the stack look like a struct trapframe

      # 将段寄存器压入栈中
      pushl %ds
      pushl %es
      pushl %fs
      pushl %gs

      #将所有通用寄存器压入栈中
      pushal

      # load GD_KDATA into %ds and %es to set up data segments for kernel
      # 设置段寄存器，为内核态做准备
      movl $GD_KDATA, %eax
      movw %ax, %ds
      movw %ax, %es

      # push %esp to pass a pointer to the trapframe as an argument to trap()
      # 将当前栈顶位置（trapframe的起始位置）压入栈顶，作为trap函数调用的参数
      pushl %esp

      # call trap(tf), where tf=%esp
      # 调用函数trap，进行trap处理
      call trap

      # pop the pushed stack pointer
      # 将压入的参数弹出，开始进行trap处理善后工作
      popl %esp

      # return falls through to trapret...
  .globl __trapret
  __trapret:
      # restore registers from stack
      # 恢复所有通用寄存器
      popal

      # restore %ds, %es, %fs and %gs
      # 恢复所有段寄存器
      popl %gs
      popl %fs
      popl %es
      popl %ds

      # get rid of the trap number and error code
      # 将trap时压入栈中的trap number 和 error code弹出，返回用户态
      addl $0x8, %esp
      iret

  ```

请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

```
    .section .text.entry
    .globl _start
_start:
    add t0, a0, 1
    slli t0, t0, 16

    # t0里面放的是bootstack的大小
    
    lui sp, %hi(bootstack)
    addi sp, sp, %lo(bootstack)
    add sp, sp, t0

    # 将sp置为bootstack的栈顶

    call rust_main
    # 启动os

    .section .bss.stack
    .align 12  #PGSHIFT
    .global bootstack

    # 设置bootstack空间
bootstack:
    .space 4096 * 16 * 8
    .global bootstacktop
bootstacktop:

```

#### 练习二
宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

 > 利用宏进行复杂数据结构中的数据访问；
 > 利用宏进行数据类型转换；如 to_struct, 
 > 常用功能的代码片段优化；如  ROUNDDOWN, SetPageDirty

 > 用于翻译汇编代码；如SEG_ASM
 > 用于定义常量；
 > 用于复杂系统函数的简写命名；如va_start

## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。
编译相关问题都很好地解决了，但是对于make debug由于mac 下没有gnome-terminal而失败。

没有解决，网络上的解决方案是用xterm进行替代，但是我本地的尝试并不可行。

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
>>>>>>> 63b87d9ea737d578dcf4c134125b65f2d85a8557
