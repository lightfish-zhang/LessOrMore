---
layout: post
title: 浮点数运算问题和各种编程语言的解决方案
date:   2016-10-20 10:30:00 +0800
categories: bug 
tag: [bug]
---

* content
{:toc}

## 前言
浮点运算的精确度，相信coder们都各自的体会，在各自常使用的语言中，都会不自觉陷入浮点运算的困扰，然后找到并有各自的解决方法．本文收集各种语言的解决方案，并将其放在篇头，探究浮点运算问题的长篇大论置于后面
   
## 常见的语言作浮点运算

- c语言: `printf("%.17f\n", .1+.2)` 打印出 `0.30000000000000004`
- Java: `System.out.println(.1 + .2)` 打印出 `0.30000000000000004`，　`System.out.println(.1F + .2F)` 打印出 `0.3`
- Objective-C: `0.1 + 0.2;` 的结果是 `0.300000012`
- JavaScript: `console.log(.1 + .2)` 打印出`0.30000000000000004`
- PHP 有点意思， `echo .1 + .2` 打印出 `0.3`，PHP有这样一个的隐式转换的规律:

> PHP converts 0.30000000000000004 to a string and shortens it to “0.3”. To achieve the desired floating point result, adjust the precision ini setting: ini_set(“precision”, 17).

- 更多语言的浮点型运算例子请点传送门 [各种语言的浮点运算的结果比较]


## 解决方法



## 浮点预算精度问题的探究



## 参考文献



[各种语言的浮点运算的结果比较]

[各种语言的浮点运算的结果比较]:http://0.30000000000000004.com/