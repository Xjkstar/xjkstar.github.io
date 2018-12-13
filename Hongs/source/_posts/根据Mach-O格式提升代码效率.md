---
title: 《Overview of the Mach-O Executable Format》中文翻译
toc: true
tags: [Mach-O,iOS,性能优化]
date: 2018-12-12 21:29:01
categories: Mach-O
---
[《Overview of the Mach-O Executable Format》](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/MachOOverview.html#//apple_ref/doc/uid/20001860-BAJGJEJC) 的中文翻译，来自Apple 官方文档《Code Size Performance Guidelines》。本文描述如何根据Mach-O文件格式提升代码执行效率。***版权所有，转载须注明出处！***
<!-- more -->
## Overview of the Mach-O Executable Format
Mach-O 是 OS X 上 Native 二进制可执行文件格式，是装运代码的首选格式。一个可执行文件格式决定了二进制文件中code和data被读入内存的指令。这些 code、data 指令会影响内存使用、页面寻址，因此直接影响到你的程序运行性能。

一个Mach-O二进制文件由多个 segment 组成。每个 segment 包含1个或多个 section。code和不同类型的data 分别记录在这些 section 中。segments 是 page 对齐的，而 sections 不保证 page 对齐。segment 的大小由其下所有 section 的 bytes 计算得出，并且最后一个虚拟内存的页面边界向上取整。因此，一个 segment 的 size 总是4096(4k)的整数倍，segment size 最小是 4096 bytes。

Mach-O的 segment、section 按功能命名。segment命名格式，双下划线+大写字母（比如，\_\_TEXT）；section 命名格式，双下滑线+小写字幕（比如，\_\_text）。

一个Mach-O文件可能包含多个 segment，但只有2个与性能相关：\_\_TEXT、\_\_DATA。

## The \_\_TEXT Segment: Read Only
\_\_TEXT segment 是一个只读区域，包含可执行代码、常量数据。按照规范，编译工具创建每个可执行文件时，至少需要包含一个只读的segment——\_\_TEXT segment。基于此segment的只读性，内核将这部分信息映射到内存只需一次即可。当此 segment 映射到内存后，需要这部分内容的线程可以共享。（主要是frameworks、动态库的情况）。只读性还意味着，组成\_\_TEXT的pages 不会被保存到后台存储。当内核需要释放物理内存时，会释放1个或多个\_\_TEXT的pages，然后在需要时从硬盘上重新读取。

表1列举了\_\_TEXT中一些常用的sections，完整的 segments 列表见 [Mach-O Runtime Architecture]
(http://math-atlas.sourceforge.net/devel/assembly/MachORuntime.pdf)。

| Section | 说明 |
|---|---|
| \_\_text | 编译后的机器码 |
| \_\_const | 全局常量 |
| \_\_cstring | 字符串常量(代码中引用的) |
| \_\_picsymbol\_stub | 动态链接器(dyld)使用的无需重定向的代码存根例程 |

## The \_\_DATA Segment: Read/Write
\_\_DATA segment 包含可执行文件中非常量的数据。此 segment 可读可写。基于可读属性，一个 framework 或动态库为每个连接的线程逻辑拷贝一份。当内存分页可读写时，内核将其标记为 copy-on-write。这项技术会延迟拷贝分页单元，直到某一个动态线程需要写入此分页。此时，内核会给这个线程创建一个私有拷贝分页。

\_\_DATA segment 包含了很多 sections，有些只给动态链接库使用。表2列举了\_\_DATA中一些常用的 sections，完整的 segments 列表见 [Mach-O Runtime Architecture]
(http://math-atlas.sourceforge.net/devel/assembly/MachORuntime.pdf)

| Section | 说明 |
|---|---|
| \_\_data | 已初始化的全局变量(例，ina a = 1; 或 static int a = 1;) |
| \_\_const | 需要重定向的常量数据(例，char * const p = "foo";) |
| \_\_bss | 未初始化的静态变量(例，static int a;) |
| \_\_common | 未初始化的外部全局变量(例，int a;外部代码块) |
| \_\_dyld | 占位 section，给动态链接库使用 |
| \_\_la\_symbol\_ptr | 懒加载符号指针，可执行文件调用的未定义方法的符号指针 |
| \_\_nl\_symbol\_ptr | 非懒加载符号指针，可执行文件依赖的未定义方法的符号指针 |

## Mach-O Performance Implications
Mach-O 可执行文件中的\_\_TEXT、\_\_DATA segments 对程序性能有直接影响。每个segment 配置的方法、目标都不一样。但有一点相同：更有效的利用内存。

大部分典型的 Mach-O 文件包含可执行代码，位与\_\_TEXT,\_\_text section 中。注意到\_\_TEXT是只读的，是直接映射到可执行文件的。因此，如果内核需要回收被\_\_text 占用的物理内存时，不需要保存到后台存储，不需要分页。只需释放内存，待之后再次被引用时，再从硬盘读取。虽然这样操作比内存切换要高效——因为这样只需要读写1次硬盘，比2次少——但这样任然不够高效，当许多分页需要重硬盘读取时会非常明显。

解决此问题的一种办法是通过程序重新排序提高代码引用定位效率，详见[Improving Locality of Reference](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/ImprovingLocality.html#//apple_ref/doc/uid/20001862-CJBJFIDD)。这个方法将 methods、funtions聚集在一起重新排序，根据被执行来源、被调用次数、调用频率的顺序。当\_\_text section的分页这样逻辑分布时，绝大多数情况下可以减少调用内存的释放、加载的次数。比如，当你把所有 launch-time 初始化相关的方法放在1、2个分页中，那么初始化完成后这些分页就不必再加载了。

与\_\_TEXT segment不同，\_\_DATA segment可写，所以\_\_DATA segment不可共享。一个 framework 的非常量全局变量执行时可能会冲突，因为链接到 framework 的各个线程都各自拥有一份这些变量的拷贝。此问题最主要解决的方案，是尽可能这些非常量全局变量声明为常量，从而移入\_\_TEXT,\_\_const section 。[Reducing Shared Memory Pages](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/SharedPages.html#//apple_ref/doc/uid/20001863-CJBJFIDD)中介绍了此方法以及更多技术方案。对于一个应用来说这通常不是问题，因为一个应用中\_\_DATA的 sections 不会与其他应用共享。

编译器将\_\_DATA segment 中不同类型的非常量全局数据记录在不同 section 中。这些类型是，未初始化的静态数据、按ANSI C 规范"临时定义"但未声明 extern 的符号。未初始化的静态数据记录在 \_\_DATA,\_\_bss 中。"临时定义"的符号记录在\_\_DATA,\_\_common 中。

ANSI C 和 C++标准要求系统将未初始化的静态变量置0。（其他类型的未初始化变量保持未初始化状态）。因为未初始化的静态数据、"临时定义"的符号记录在不同 section ，操作系统需要分别处理它们。因为这些变量在不同 sections，大概率分布在不同分页上，所以需要分别切换来切换去，导致你的代码执行变慢。此问题的解决方案，在[Reducing Shared Memory Pages](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/SharedPages.html#//apple_ref/doc/uid/20001863-CJBJFIDD)中说明的，统一非常量全局数据到\_\_DATA的一个 section 中。