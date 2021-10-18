---
title: "C++ 内存类型"
date: 2020-09-01T00:24:37+08:00
draft: false
---

# C++内存类型

### (1).rodata段（常量区 ）

存放只读数据，

### (2) .text段

存放已经编译的机器代码

**程序加载运行时，.rodata段和.text段通常合并到一个Segment（Text Segment）中，操作系统将这个Segment的页面只读保护起来，防止意外的改写。**

### (3) .data段（全局区）

存放已初始化的全局变量

### (4) .bss段

存放未初始化的全局变量

**.data和.bss在加载时合并到一个Segment（Data Segment）中，这个Segment是可读可写的。**

### (5) 栈

### (6) 堆



