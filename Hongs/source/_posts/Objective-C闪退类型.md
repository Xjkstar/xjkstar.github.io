---
title: Objective-C闪退类型
toc: true
tags: [iOS,Crash]
date: 2018-09-18 11:49:19
categories:
---
iOS的6个闪退类型解释
<!-- more -->
## 符号
定义于头文件 <signal.h>

```
#define SIGTERM /*implementation defined*/
#define SIGSEGV /*implementation defined*/
#define SIGINT /*implementation defined*/
#define SIGILL /*implementation defined*/
#define SIGABRT /*implementation defined*/
#define SIGFPE /*implementation defined*/
```

## 解释
上面每个宏常量都展开成拥有相异值的整数常量表达式，表示发送给程序的不同信号。

|常量|解释|
|---|---|
|SIGTERM|发送给程序的终止请求|
|SIGSEGV|非法内存访问（段错误）|
|SIGINT|外部中断，通常为用户所发动|
|SIGILL|非法程序映像，例如非法指令|
|SIGABRT|异常终止条件，例如 abort() 所起始的|
|SIGFPE|错误的算术运算，如除以零|

## 引用
- C11 standard (ISO/IEC 9899:2011):
    - 7.14/3 Signal handling <signal.h> (p: 265)
- C99 standard (ISO/IEC 9899:1999):
    - 7.14/3 Signal handling <signal.h> (p: 246)
- C89/C90 standard (ISO/IEC 9899:1990):
    - 4.7 SIGNAL HANDLING <signal.h>

## 来源
[cppreference.com](https://zh.cppreference.com/w/c/program/SIG_types)