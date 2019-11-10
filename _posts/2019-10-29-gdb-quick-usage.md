---
layout: post
title: "[工具]GDB使用速查"
---

> last update: 2019-11-10

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

|命令|简写|解释|
|:----:|:----:|----|
|`file`||指定要调试的可执行文件，如果启动的时候没有指定可以在启动后通过这种方式指定|
|`continue`|`c`|继续运行程序，直到下一个断点或者终止|
|`kill`|`k`|kill当前正在调试的进程|
|`quit`|`q`|退出GDB|
|`break`|`b`|设置断点|
|`watch`||设置观察点|
|`catch`||设置捕获点|
|`delete`|`d`|删除断点|
|`step`|`s`|单步执行一条语句，且遇到函数调用的话进入函数|
|`stepi`|`si`|单条执行一条指令|
|`next`|`n`|单步执行一条语句，且遇到函数调用不会进入函数，而是直接完成调用|
|`nexti`|`ni`|单步执行一条指令|
|`finish`||直接执行完当前函数，并打印返回值|
|`return`||直接就地返回当前函数，可以跟上返回值|
|`until`||继续执行，直到代码行数大于当前行数，可用于跳出循环|
|`backtrace`|`bt`|打印函数调用栈|
|`info`|`i`|查看状态|
|`disassemble`|`disas`|查看反汇编代码|

## GDB常用技巧

### Breakpoint, Watchpoint, Catchpoint

Breakpoint的作用是让程序执行到某个特定的地方停止，可以通过以下方式设置断点：

- `b <function>`，在函数入口处设置断点
- `b <+/-offset>`，在当前停止处向前或向后偏移offset处设置断点
- `b <file:linenumber>`，在源文件的第几行处设置断点，省略文件路径则使用当前源文件
- `b <file:function>`，在指定源文件中的函数入口处设置断点
- `b ... if condition`，设置条件断点，当满足某条件是程序停止

Watchpoint的作用主要用于监视某个表达式，当其值被操作时程序停止，如：

- `watch expr`，例，`watch *(int *)0x1000000`当0x1000000这个地址存放的int类型变量被写入时停止
- `rwatch expr`，当表达式被读取时停止
- `awatch expr`，当表达式被写入或读取时停止

Catchpoint的作用是当发生某个事件时停止，支持如下事件：

```
catch assert -- Catch failed Ada assertions
catch catch -- Catch an exception
catch exception -- Catch Ada exceptions
catch exec -- Catch calls to exec
catch fork -- Catch calls to fork
catch handlers -- Catch Ada exceptions
catch load -- Catch loads of shared libraries
catch rethrow -- Catch an exception
catch signal -- Catch signals by their names and/or numbers
catch syscall -- Catch system calls by their names
catch throw -- Catch an exception
catch unload -- Catch unloads of shared libraries
catch vfork -- Catch calls to vfork
```

### GDB调试宏

首先，在编译时加上`-ggdb3`参数，然后使用`i macro <macro name>`命令查看宏定义，用`macro expand <macro>`查看宏展开的样子。

### GDB调试多进程

在调试多进程程序时，GDB默认追踪父进程，可以通过`set follow-fork-mode child`命令（默认是parent）来追踪子进程。默认情况下，GDB追踪一个进程时不会管另一个进程的状态，可以通过`set detach-on-fork off`命令（默认是on）来达到调试一个进程时另一个进程挂起，此时可以通过`i inferiors`来查看所有进程的状态，也可以通过`inferior <num>`来切换调试的进程。