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

知识大乱炖，用于快速回忆知识点，。

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

**Attention**：MRC情况下，ARC无事，自动copy，为了统一可以加copy

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
```
## @dynamic, @systhesis, @propery

https://www.jianshu.com/p/c883687c6405

@dynamic 表示用户自己实现get,set方法；（实例变量也需要自己添加）@systheis如果用户没有实现get,set，系统会自己生产get/set

## bounds和frame的关系

`
frame：描述当前视图在其父视图中的位置和大小。
bounds：描述当前视图在其自身坐标系统中的位置和大小。
center：描述当前视图的中心点在其父视图中的位置。
`
所以改变bounds，frame不会动，但相当于自身坐标系往(-bounds.origin.x, -bounds.origin.y）方向移动，影响的是子视图的frame位置（起点变成了(-bounds.origin.x, -bounds.origin.y））

## postion、anchorPoint和frame之间的关系

首先，view的position等属性就是layer的属性。
```Objective-C
position.x = frame.origin.x + anchorPoint.x * bounds.size.width
position.y = frame.origin.y + anchorPoint.y * bounds.size.height
```
position和anchorPoint互不影响，都只会影响frame

## iOS实现动画有几种方式

UIView animatiedWith

UIView animateKeyframesWithDuration

CAbasicAnimation (针对layer的某一属性)

CATransition(过渡动画)
fade/moveIn/Push/Reval(类似于PPT)


KeyAnimation

## NSTimer问题总结

runloop耗时任务多的时候，会丢失

https://juejin.cn/post/7016633863241728014

## runtime的一些应用

参考文档：
* https://juejin.cn/post/7086379028323516452
* https://blog.csdn.net/weixin_38735568/article/details/95939608

1. 动态添加属性
2. 方法交换（hook系统方法）Method Swizzling
3. 字典转模型(Model to Dictionay)

## iOS数据持久化存储方式

参考：https://juejin.cn/post/6844903440586375181

1. plist(属性列表)文件
2. Preference(偏好设置)
3. NSKeyedArchiver(归档)
4. FMDB(基于SQLite3 封装的一套OC的API库)
5. CoreData

##  NSString property使用strong还是copy，区别和联系是啥？

NSMutableString strong修饰时，会修改原来的内容，造成一些不可预期的效果。

It is still recommended to copy because you want to avoid something passing a mutable string and then changing it without you knowing. A copy guarantees that the string you have will not change.

我们知道NSMutableString是NSString的子类，一个NSString指针可以指向NSMutableString对象，strongString指针指向一个可变字符串是正常的。
如上例子可以看出:

1. 当原字符串是NSString时，字符串是不可变的，不管是Strong还是Copy属性的对象，都是指向原对象，Copy操作只是做了次浅拷贝。
2. 当原字符串是NSMutableString时，Strong属性只是增加了原字符串的引用计数，而Copy属性则是对原字符串做了次深拷贝，产生一个新的对象，且Copy属性对象指向这个新的对象,且这个Copy属性对象的类型始终是NSString，而不是NSMutableString，因此其是不可变的。
3. 这里还有一个性能问题，即在原字符串是NSMutableString，Strong是单纯的增加对象的引用计数，而Copy操作是执行了一次深拷贝，所以性能上会有所差异(虽然不大)。如果原字符串是NSString时，则没有这个问题。

## 苹果登录流程

1. 客户端用appleID/邮箱密码请求,返回个人信息和token
2. 将token和userID给后端
3. 后端向苹果后端请求解密token的公钥
4. 后端用公钥,基于JSW算法验证token
5. 返回客户端告知结果

![Apple Login Procedure](/images/AppleIDLoginProcedure.png)

## iOS消息转发

### 经典崩溃（启发点）

`unrecognized selector sent to instance 0x100524a90`
先来看个很经典的崩溃打印。一般这个日志前部分还会给出所调用的方法，我们可以借此很快找到原因所在，可以说是相当贴心了。然而，
苹果在方便我们的同时，你是否想过这个日志具体是在什么时候打印的，系统是靠什么来捕获这类型即将崩溃的信息，开发者是否也可以捕获呢。这些都要从消息转发流程说起，看完分析或许你心里就有答案了。

这段描述很好，为此引为参考，来源https://juejin.cn/post/6857013884298133517

### 流程速记
• 【快速查找流程】首先，在类的缓存cache中查找指定方法的实现
• 【慢速查找流程】如果缓存中没有找到，则在类的方法列表中查找，如果还是没找到，则去父类链的缓存和方法列表中查找
• 【动态方法决议】如果慢速查找还是没有找到时，第一次补救机会就是尝试一次动态方法决议，即重写`resolveInstanceMethod/resolveClassMethod`方法,`class_addMethod`添加方法
• 【消息转发】如果动态方法决议还是没有找到，则进行消息转发，消息转发中有两次补救机会：快速转发+慢速转发
• 如果转发之后也没有，则程序直接报错崩溃`unrecognized selector sent to instance``

![Message Forawrd Procedure](/images/Message_Forawrd_Procedure.png)

### 应用

    一些容易造成奔溃的方法添加保护，如NSArray访问数组越界的情况，可以用消息转发添加保护。

#### 参考文档

* https://juejin.cn/post/6844903600968171533

## KeyChain

账号密码，token用KeyChain保存，删除App，KeyChain中的数据仍然存在，因为数据是iOS系统保存的

## iOS 通知的线程是在哪个线程

https://www.jianshu.com/p/e368a18ca7c2

## runloop和线程的关系

1. 一条线程对应一个RunLoop对象，每条线程都有唯一一个与之对应的RunLoop对象。
2. 我们只能在当前线程中操作当前线程的RunLoop，而不能去操作其他线程的RunLoop。
3. RunLoop对象在第一次获取RunLoop时创建，销毁则是在线程结束的时候。
4. 主线程的RunLoop对象系统自动帮助我们创建好了(原理如下)，而子线程的RunLoop对象需要我们主动创建。

## NSOperation

参考： https://juejin.cn/post/6844904053906866189

为什么要使用 NSOperation、NSOperationQueue？

1. 可以方便的调用cancel方法来取消某个操作，而 GCD 中的任务是无法被取消的。
2. 可以方指定操作间的依赖关系, 方便的控制执行顺序。
3. 可以通过KVO提供对操作对象的精细控制(如监听当前操作是否被取消或是否已经完成等)。
4. 可以方便的指定操作优先级。
5. 可以自定义NSOperation的子类可以实现操作重用.

## dealloc

```Objective-C
    - (void)dealloc {
        _objc_rootDealloc(self);
    }
```
直接调用_objc_rootDealloc方法来做处理，我们省略一些细节处理，通常情况下，dealloc方法最终会调用objc_dispose方法，内部又调用objc_destructInstance方法来进行析构操作，析构完成后将内存释放掉。

1. dealloc的调用是在最后一次release执行后，但此时实例变量(ivars)并未释放。
2. [super dealloc]会自动调用： 父类的dealloc方法会在子类dealloc方法返回后自动执行。
3. ARC子类的实例变量在根类[NSObject dealloc]中释放。

## load

父类load调用在子类之前，分类的load在主类load调用完后执行
## iOS websocket使用

参考： https://www.jianshu.com/p/80d1440b7012
知名库： https://github.com/facebookincubator/SocketRocket

 主要api:
* open
* close
* sendData/sendString
* didReceviceMessage
* didFailWithError
* didCloseWithCode

## iOS 沙盒

参考：https://www.yuukizoom.top/2021/03/15/iOS%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%B2%99%E7%9B%92%E6%9C%BA%E5%88%B6/

* MyApp.app ： 该目录存放应用本身的数据，包括资源文件和可执行文件。该目录是只读的，如果改变这个目录，将会改变应用程序的签名，应用将会无法启动。不会被iTunes和iCloud同步。

* Documents: 该目录主要存放用户产生的文件，该目录会被iTunes和iCloud同步。

* Library: 苹果建议用来存放默认设置或其他状态信息，该目录除Caches子目录以外会被iTunes和iCloud同步。

* Library/Caches: 主要存放缓存文件，比如音乐缓存，图片缓存等，用户使用过程中的缓存都可以保存在这个目录中，可用于保存可再生文件，应用程序也需要负责删除这些文件，该目录不会被iTunes和iCloud同步。

* Library/Preferences: 存放应用偏好设置文件，该目录会被iTunes和iCloud同步。

* tmp: 存放各种临时文件，保存应用再次启动时不需要的文件。当应用不再需要这些文件时应该主动将其删除。该目录的文件会被系统清理，比如系统磁盘空间不足的时候。该目录不会被iTunes和iCloud同步。

## OOM监控