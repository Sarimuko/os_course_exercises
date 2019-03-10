# lec4: lab1 SPOC思考题

##**提前准备**
（请在上课前完成）

 - 完成lec4的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
 - 了解x86的保护模式，段选择子，全局描述符，全局描述符表，中断描述符表等概念，以及如何读写，设置等操作
 - 了解Linux中的ELF执行文件格式
 - 了解外设:串口，并口，时钟，键盘,CGA，已经如何对这些外设进行编程
 - 了解x86架构中的mem地址空间和io地址空间
 - 了解x86的中断处理过程（包括硬件部分和软件部分）
 - 了解GCC的x86/RV内联汇编
 - 了解C语言的函数可变参数编程
 	```
 	va_list
 	va_start(AP, lastarg)
 	va_arg(AP, TYPE)
 	```
 - 了解qemu的启动参数的含义
 	done

 - 在piazza上就lec3学习中不理解问题进行提问
 - 学会使用 qemu
 - 在linux系统中，看看 /proc/cpuinfo的内容

## 思考题

### 启动顺序

1. x86段寄存器的字段含义和功能有哪些？
	- 代码段寄存器
	- 数据段寄存器
	- 堆栈段寄存器
	- 附加段寄存器
2. x86描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中存在这些字段？对应的访问条件是什么？
	https://blog.csdn.net/better0332/article/details/3416749
	- CPL current privilege level 当前执行代码处于的特权级
	- RPL request privilege level 进程对段访问的请求权限，对于每个进程，每个进程对于段的访问权限。是可变的，并且会削弱CPL的访问权限。如果该进程的CPL=0，但是其某个段选择子对于这个段的RPL=3，那么实际上访问的特权级只是3。
	- DPL descriptor privilege level 段描述符的特权级，规定访问的最低特权级
3. 分析可执行文件格式elf的格式（无需回答）
```
#ifndef __LIBS_ELF_H__
#define __LIBS_ELF_H__

#include <defs.h>

#define ELF_MAGIC    0x464C457FU            // "\x7FELF" in little endian

/* file header */
struct elfhdr {

	//magic 本来应该是16位长的一个数组，但是前4位应该是文件标识，因此直接使用一个常量来表示。
    uint32_t e_magic;     // must equal ELF_MAGIC
    uint8_t e_elf[12];

    uint16_t e_type;      // 1=relocatable, 2=executable, 3=shared object, 4=core image
    uint16_t e_machine;   // 3=x86, 4=68K, etc.
    uint32_t e_version;   // file version, always 1
    uint32_t e_entry;     // entry point if executable
    uint32_t e_phoff;     // file position of program header or 0
    uint32_t e_shoff;     // file position of section header or 0
    uint32_t e_flags;     // architecture-specific flags, usually 0
    uint16_t e_ehsize;    // size of this elf header
    uint16_t e_phentsize; // size of an entry in program header
    uint16_t e_phnum;     // number of entries in program header or 0
    uint16_t e_shentsize; // size of an entry in section header
    uint16_t e_shnum;     // number of entries in section header or 0
    uint16_t e_shstrndx;  // section number that contains section name strings
};

/* program section header, 32 bit version?*/
struct proghdr {
    uint32_t p_type;   // loadable code or data, dynamic linking info,etc.
    uint32_t p_offset; // file offset of segment
    uint32_t p_va;     // virtual address to map segment
    uint32_t p_pa;     // physical address, not used
    uint32_t p_filesz; // size of segment in file
    uint32_t p_memsz;  // size of segment in memory (bigger if contains bss）
    uint32_t p_flags;  // read/write/execute bits
    uint32_t p_align;  // required alignment, invariably hardware page size
};

#endif /* !__LIBS_ELF_H__ */

```

### 4.1 C函数调用的实现

### 4.2 x86中断处理过程

1. x86/RV中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？
	用户态中断需要转换到内核态，有一个栈的转换，内核态压栈会将用户栈的位置压栈保存。
	而内核态中断仍然处于内核栈中，只需要保存现场部分的压栈。
2. 为什么在用户态的中断响应要使用内核堆栈？
	因为中断的处理需要在内核态中处理。**保护中断处理例程代码的安全**
3. x86中trap类型的中断门与interrupt类型的中断门有啥设置上的差别？如果在设置中断门上不做区分，会有什么可能的后果?
	Interrupt门调用会导致中断被屏蔽
	调用trap门则不会
	为了防止重复触发中断

### 4.3 练习四和五 ucore内核映像加载和函数调用栈分析

1. ucore中，在kdebug.c文件中用到的函数`read_ebp`是内联的，而函数`read_eip`不是内联的。为什么要设计成这样？
因为ebp是直接使用汇编指令获得的，如果不使用内联函数的话，会导致此时出现了函数调用，被读出的ebp并不是要被读到的ebp而是新分配的函数栈的ebp。而eip不能使用汇编指令直接获得，只能使用函数调用将eip压栈，从而读出被保存在栈中的eip，因此不能使用内联函数而是使用函数调用。

### 4.4 练习六 完善中断初始化和处理

1. CPU加电初始化后中断是使能的吗？为什么？
不是，我们可以在trap.c中看到，需要建立起来中段描述符表和例程之后才会打开中断。

## 开放思考题

1. 在ucore/rcore中如何修改lab1, 实现在出现除零异常时显示一个字符串的异常服务例程？
2. 在ucore lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
3. GRUB是一个通用的x86 bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
4. 如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？

## 课堂实践
### 练习一
在Linux系统的应用程序中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。

### 练习二
在ucore/rcore内核中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。
