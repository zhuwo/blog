---
title: iOS
date: 2023-08-29 13:48:17
tags:
---

## weak实现原理

1. https://www.jianshu.com/p/f331bd5ce8f8

获取oldobj(弱引用所指向的之前对象) unregister，替换为newobj的hashtable, register，
weak表是hash表，对象的weak指针数组，销毁对象的时候所有weakrefernce置为nil