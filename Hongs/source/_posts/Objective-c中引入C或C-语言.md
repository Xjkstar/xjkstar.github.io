---
title: Objective-C中引入C或C++语言
toc: true
tags: [Objective-C,C,C++]
date: 2018-08-24 15:47:21
categories: Xcode
---
Objective-C编程时，C、C++混合编程的处理
<!-- more -->
## 与 C 语言
与 C 语言混合编程没有特殊处理，直接编写 C 语言代码，使用 C 函数

## 与 C++语言
与 C++语言混合编程，需要将.m 文件的后缀改为.mm 文件。头文件后缀不用修改。

## 与 C 语言& C++语言
既有 C 语言，又有 C++语言的混合编程。首先 C++语言需要将文件后缀改为.mm。在.mm 文件中引入外部 C 函数时需要做特殊处理

```Objective-C
#ifdef __cplusplus
extern "C" {
#endif

# 在 .mm文件中引入 C 的头文件，在这里引入，否则 C 的头文件会找不到
    
#ifdef __cplusplus
}
#endif
```