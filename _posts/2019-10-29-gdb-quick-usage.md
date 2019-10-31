---
layout: post
title: "[工具]GDB使用速查"
---

> last update: 2019-10-29

GDB是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。
现在基本就写C/C++了，GDB成了开发过程中不可或缺的工具，因此记录一下GDB的常用命令和方法，本文长期更新。

<!--more-->

要使用GDB应当在编译过程中加上`-g`参数，以将调试信息编译进可执行文件中，当然没有调试信息的可执行文件
也是可以使用GDB调试的，但是难度很大。

## 启动GDB调试

有以下几种方法可以启动GDB调试，需要注意的是要使用与调试程序相同架构的GDB：

1. 常规方法

```bash
gdb <executable file>
```

如果被调试程序需要参数则可以通过如下方式启动

```bash
gdb --args <executable file> <args>
```

2. 调试已经启动的进程

```bash
gdb <executable file> <pid>
```

3. 调试coredump文件

```bash
gdb <executable file> <core file>
```

> coredump文件包含程序运行时的内存信息，含寄存器状态、堆栈指针、内存管理信息、操作系统flags及其他信息，
算是程序运行时的一个快照，通常会在程序异常终止时产生，是分析程序异常的重要手段。控制coredump文件的产生：
```bash
ulimit -c 0 # 不产生coredump文件
ulimit -c 100 # coredump文件最大为100K
ulimit -c unlimited # 不限制大小
```

## GDB常用命令

|命令|简写|解释|示例|
|:----:|:----:|----|----|
|`file`||指定要调试的可执行文件，如果启动的时候没有指定可以在启动后通过这种方式指定|`(gdb)file a.out`|
|`run`|`r`|开始运行程序，一般GDB启动之后被调试程序并未立即开始运行|`(gdb)r`|
|`continue`|`c`|继续运行程序，直到下一个断点或者终止|`(gdb)c`|
|`kill`|`k`|kill当前正在调试的进程|`(gdb)k`|
|`quit`|`q`|退出GDB|`(gdb)q`|
|`break`|`b`|设置断点|`(gdb)b <path to file>:xxx` 设置断点到某文件的xxx行<br/>`(gdb)b func_name` 设置断点到某个函数<br/>`(gdb)b *<address>` 设置断点为代码中的某个地址|
|`break`|`b <breakpoint> if <condition>`|条件断点|`(gdb)b main if argc > 1` 当main函数的argc参数大于1时断点生效|
|`delete`|`d`|删除断点|`(gdb)d <breakpoint num>`|
|`step`|`s`|单步执行一条语句，且遇到函数调用的话进入函数|`(gdb)s`|
|`stepi`|`si`|单条执行一条指令|`(gdb)si`|
|`next`|`n`|单步执行一条语句，且遇到函数调用不会进入函数，而是直接完成调用|`(gdb)n`|
|`nexti`|`ni`|单步执行一条指令|`(gdb)ni`|
|`finish`||直接执行完当前函数，并打印返回值|`(gdb)finish`|
|`return`||直接就地返回当前函数，可以跟上返回值|`(gdb)return 40`|
|`until`||继续执行，直到代码行数大于当前行数，可用于跳出循环|`(gdb)until`|
|`backtrace`|`bt`|打印函数调用栈|`(gdb)bt`|
|`info`|`i`|查看状态|`(gdb)i registers` 查看所有寄存器<br/>`(gdb)i locals` 查看所有局部变量|
|`disassemble`|`disas`|查看反汇编代码|`(gdb)disas`|

## GDB常用技巧

### GDB调试宏

首先，在编译时加上`-ggdb3`参数，然后使用`i macro <macro name>`命令查看宏定义，用`macro expand <macro>`查看宏展开的样子。

### GDB调试多进程

在调试多进程程序时，GDB默认追踪父进程，可以通过`set follow-fork-mode child`命令（默认是parent）来追踪子进程。默认情况下，GDB追踪一个进程时不会管另一个进程的状态，可以通过`set detach-on-fork off`命令（默认是on）来达到调试一个进程时另一个进程挂起，此时可以通过`i inferiors`来查看所有进程的状态，也可以通过`inferior <num>`来切换调试的进程。