---
title: iOS知识点
date: 2023-08-29 13:48:17
tags:
- iOS
- 面试
categories:
- 技术
- iOS
---

知识大乱炖，用于快速回忆知识点，需要时背诵。

## weak实现原理

参考链接：https://www.jianshu.com/p/f331bd5ce8f8

   * weak由runtime持有的weak_table实现，每个对象有对应的weak_entry_t
   * 获取oldobj(弱引用所指向的之前对象) unregister，替换为newobj的hashtable, register，
    weak表是hash表，对象的weak指针数组，销毁对象的时候所有weakrefernce置为nil
## KVO 实现原理

参考：https://juejin.cn/post/6844903593925935117

iOS系统会修改这个对象的isa指针，改为指向一个全新的通过Runtime动态创建的子类(KVONOtification)，子类拥有自己的set方法实现，set方法实现内部会顺序调用willChangeValueForKey方法、原来的setter方法实现、didChangeValueForKey方法，而didChangeValueForKey方法内部又会调用监听器的observeValueForKeyPath:ofObject:change:context:监听方法。
## 如何扩大点击区域

    事件响应链会调用pointInSide，重写pointInSide，edgeInsets扩大范围检测
## OC实现链式调用

https://juejin.cn/post/7010958824933130253

提供返回block的方法， block内部返回self，返回self是实现链式的核心。
## 自动释放池内对象释放时机

https://juejin.cn/post/7010726670181253127

自动释放池对象AutoreleasePool被销毁时，有可能是当前runloop结束进入休眠，会对池内的所有对象发送release指令。
本质是有个双向链表存储所有池内对象，一页放满就新建一页，放入新的页面中。
## Struct和Class的区别

|  Comparision |    \|Class  |  Structure  |
|  ----  | ----  | ----  |
|  Type	 | Classes are reference types.	 | Structures are value types. |
| Inheritance | Classes have an inheritance that allows one class to inherit the characteristics of another.	 | Structures do not support inheritance. |
| Storage	 | Class instances are stored on the heap.	 | Structure properties are stored on the stack. |
| Initializer	 | We have to define the initializer manually.	 | Struct gets a default initializer automatically. |
| Thread−safe	 | Classes are not fully thread−safe.	 | The structure is thread−safe or singleton at all times. |
## Category实现原理

https://juejin.cn/post/6844903602524274696

从源码基本可以看出我们平时使用categroy的方式，对象方法，类方法，协议，和属性都可以找到对应的存储方式。并且我们发现分类结构体中是不存在成员变量的，因此分类中是不允许添加成员变量的。分类中添加的属性并不会帮助我们自动生成成员变量，只会生成get set方法的声明，需要我们自己去实现。


category的方法会在运行时加到原来类的方法列表之前，所有category的方法协议会覆盖原来类的方法和协议等。

category中有load方法，先执行原来的load方法，load方法在类加载时候调用,initalize在类第一次使用的时候调用。
## 关联对象（添加新的成员变量）

https://juejin.cn/post/6844903605347057672


AssociationsManager拥有AssociationsHashMap=>value为ObjectAssociationMap=>valuew为ObjcAssociation（set_assoicated设置的_policy、value）。


![关联对象类图](/images/Associate_Object.png)

通过上图我们可以总结为：一个实例对象就对应一个ObjectAssociationMap，而ObjectAssociationMap中存储着多个此实例对象的关联对象的key以及ObjcAssociation，为ObjcAssociation中存储着关联对象的value和policy策略。
## GCD

https://www.jianshu.com/p/2d57c72016c6
### 主队列上执行同步任务卡死的原因

```Objective-C

dispatch_queue_t queue = dispatch_queue_create("test.queue", DISPATCH_QUEUE_SERIAL);
 dispatch_async(queue, ^{    // 异步执行 + 串行队列
        NSLog(@"add task");
        NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        dispatch_sync(queue, ^{  // 同步执行 + 当前串行队列
            // 追加任务 1
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        });
    });
```
dispatch_sync添加的任务执行完以后才会返回，因为是串行队列，所有只有dispatch_sync任务执行完，才会执行下一个任务【追加任务 1】，所有一直追加任务等待dispatch_sync执行完成
主队列是串行队列，跟这个情况一样。
## block为什么用copy修饰?

默认情况下，block 是存放在栈中即 NSStackBlock ，因此 block 在函数调用结束时，对象会变成 nil，但是对象的指针变成野指针，因此对象继续调用会产生异常。使用 copy 修饰之后，会将 block 对象保存到堆中 NSMallocBlock，它的生命周期会随着对象的销毁而结束的。所以函数调用结束之后指针也会被设置为 nil，再次调用该对象也不会产生异常。
## iOS保证线程安全的几种方式

https://juejin.cn/post/6965770220921159694

* OSSpinLock
* os_unfair_lock
* pthread_mutex
* dispatch_semaphore
* dispatch_queue(DISPATCH_QUEUE_SERIAL)
* NSLock
* NSRecursiveLock
* NSCondition
* NSConditionLock
* @synchronized

## Collection

    NSSet和NSDictionary都是用hash，查找比较高效。

    NSSet对应NSHashTable, NSDictionary对应NSMapTable。
    区别都是后者是可变的，且可以对成员进行弱引用，当对象被销毁时，锁存储的实体可以会被移除。
## 引用计数表保存在哪里 

参考链接： https://www.jianshu.com/p/43571ab79821 
 nonpointer:extra_rc +sidetable_rc;
非nonpointer: SideTable（静态变量）,散列表，SideTables

如果isa中无法存储指针，那么就存储在SideTable中的RefcountMap中

## iOS启动流程

参考文档：https://juejin.cn/post/6951591401528229895

```
1. 解析Info.plist
2. Mach-O（可执行文件）加载
    * dylib loading time（动态库加载耗时）
    * rebase/binding time（偏移修正/符号绑定耗时
    * 加载类扩展（Category）中的方法
    * C++静态对象加载、调用ObjC的 +load 函数
    * 执行声明为__attribute__((constructor))的C函数
3. 程序执行
    * 调用main()
    * 调用UIApplicationMain()
    * 调用applicationWillFinishLaunching
