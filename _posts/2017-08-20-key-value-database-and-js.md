---
layout: post
title: 脑洞，从键值对数据库联想到javascirpt的作用域
date:   2017-08-20 20:30:00 +0800
categories: javascirpt
tag: [c]
---

## 前言

最近笔者阅读redis源代码，从redis简单实现的对象机制，忽然想到javascirpt的语言特性，javascirpt的对象是如何实现的呢？一查资料发觉水真深，javascirpt经过多年优化，v8使用了许多c++的奇技淫巧。我脑洞中提出一个疑问，javascirpt的链式作用域，如何查找，是不是javascirpt编译器也实现了类似键值对数据库的一些策略，比如键空间、字典空间什么的。。。作为一枚刚巩固c语言基础入门c++的新手，像验证这个猜想有些难度，或者说好高骛远了，姑且试试吧反正浪费点时间罢了