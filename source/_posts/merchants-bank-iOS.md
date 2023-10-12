---
title: 招商银行iOS面试笔试
date: 2023-09-18 10:17:00
tags:
- iOS
- 面试
categories:
- 技术
- 面试
---

## 笔试题

1. 实现属性`@propperty(nonatomic, strong) NSString* name`和`@propperty(nonatomic, copy) NSString* name`的set方法

2. 下列哪些操作可以在iOS后台运行
   a. VOIP
   b. 音乐播放
   c. 地理位置获取
   d. http请求

3. UIView和UIControl的关系

4. [super dealloc]需要在dealloc中调用吗，调用顺序是怎么样的

    a. [super dealloc]在第一行
    b. [super dealloc]在最后一行
    c. 没有关系

5. @propery和@systhesis的作用

6. UIResponder，_____， 以及UIView能响应事件

7. 哪个是单例
    a. NSNotification
    b. UIApplication (sharedApplication)
    c. NSuserDefault (standardUserDefaults)
    d. 
8. 什么情况下使用delegate，什么情况下使用notification

    Delegate:

    消息的发送者(sender)告知接收者(receiver)某个事件将要发生，delegate同意然然后发送者响应事件，delegate机制使得接收者可以改变发送者的行为。通常发送者和接收者的关系是直接的一对多的关系。

    Notification:

    消息的发送者告知接收者事件已经发生或者将要发送，仅此而已，接收者并不能反过来影响发送者的行为。通常发送者和接收者的关系是间接的多对多关系。

    区别:

    a. 效率肯定是delegate比nsnotification高。

    b. delegate方法比notification更加直接，最典型的特征是，delegate方法往往需要关注返回值，也就是delegate方法的结果。比如-windowShouldClose:，需要关心返回的是yes还是no。所以delegate方法往往包含should这个很传神的词。也就是好比你做我的delegate，我会问你我想关闭窗口你愿意吗？你需要给我一个答案，我根据你的答案来决定如何做下一步。相反的，notification最大的特色就是不关心接受者的态度，我只管把通告放出来，你接受不接受就是你的事情，同时我也不关心结果。所以notification往往用did这个词汇，比如NSWindowDidResizeNotification，那么nswindow对象放出这个notification后就什么都不管了也不会等待接受者的反应。

    1）两个模块之间联系不是很紧密，就用notification传值，例如多线程之间传值用notificaiton。

    2）delegate只是一种较为简单的回调，且主要用在一个模块中，例如底层功能完成了，需要把一些值传到上层去，就事先把上层的函数通过delegate传到底层，然后在底层call这个delegate，它们都在一个模块中，完成一个功能，例如说 NavgationController 从 B 界面到A 点返回按钮 (调用popViewController方法) 可以用delegate比较好。

9. 什么情况下使用copy属性
    a.对于不可变属性，推荐使用copy，能防止不可变对象变成可变对象，从而防止值发生不可控变化。 
    b.对于可变属性，推荐使用strong，因为用copy修饰后，会变成不可变对象，再调用可变对象的函数时会crash

## 面试题

1. CocoaPods原理，单独的库是如何打包的？
2. 大文件下载多线程如何实现，读取和下载完成，如何合并各个文件下载的进度
3. load系统是怎么调用的，initalize可以运行时调用吗？
4. 
