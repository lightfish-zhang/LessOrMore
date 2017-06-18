---
layout: post
title: gcc与gdb的使用方法与技巧的备忘录
date:   2016-10-15 11:30:00 +0800
categories: c/cpp 
tag: [编译]
---

* content
{:toc}

## 前言

gcc和gdb的使用方法与技巧，总结下当备忘录，以免想起使用再查找资料的麻烦。

## 关于gcc

### 如何看`man gcc`

在Linux环境下编程，使用c语言的感觉是得心应手的。不管是常用命令还是c语言的系统回调，gcc的使用方法只要`man gcc`命令即可查阅，但是刚看文档的新手可能看不太懂，我大概注解下，不定时更新。

- 开始`man gcc`，然后可见以下描述，gcc是GNU项目的c或者c++的编译器，大概有哪些命令

```
NAME
       gcc - GNU project C and C++ compiler

SYNOPSIS
       gcc [-c|-S|-E] [-std=standard]
           [-g] [-pg] [-Olevel]
           [-Wwarn...] [-Wpedantic]
           [-Idir...] [-Ldir...]
           [-Dmacro[=defn]...] [-Umacro]
           [-foption...] [-mmachine-option...]
           [-o outfile] [@file] infile...
```

gcc的命令详解可以往下看`DESCRIPTION`章节，我这里做中文的简单描述。

- 编译过程：预处理-编译-汇编-链接，分步的编译的话，产生的文件的后缀名规范如下
    + `file.c`，`file.cc`，`file.cpp`是c/cpp的源文件，等待被预处理
    + `file.h` 头文件，也是源文件，用于声明函数，函数实质上是多个输入一个输出的代码块，编译时需要与`file.c`等源文件进行预处理，是因为编译器需要根据声明的类型，给输入值和输出值分配内存大小。常用于闭源或者已经编译好的`file.o`文件与头文件`file.h`一起使用。
    + 