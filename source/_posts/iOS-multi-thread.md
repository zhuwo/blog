---
title: iOS 多线程问题汇总
date: 2023-09-15 13:54:32
categories:
- 技术
- iOS
tags:
- iOS
- 多线程
---


## iOS实现多线程有哪些方法

1. NSThread
2. GCD
3. NSOperation


## 下面代码有问题吗

```Objective-C
@inteface Person: NSObject
@property(nonatomic, strong) NSString* name;
@property(nonatomic, assign) NSInteger age;
@property(nonatomic, strong) NSObject* object;
@end;


@implementation FBViewController

@property(nonatomic, strong) NSMutableArray* myArray;

- (void)testThread {
    dispatch_queue_t q = dispatch_queue_create("com.queue", DISPATCH_QUEUE_SERIAL);
    for (int i = 0; i < 100; ++i) {
        dispatch_async(q, ^{
            [self.myArray addObject:@(1)];
        });
    }
}
@end

```

改为``@property(atomic, strong) NSMutableArray* myArray``会有问题吗？

    会。因为原子性只能保证读写安全，而该表达式需要三步操作：

    1.先进行get, 读取i的值存入寄存器；
    2.将寄存器的值加1；
    3.使用寄存器修改后的值给i赋值；

### 以下内容会有循环引用吗？不调用testBlock会有吗？


```Objective-C

@inteface NewViewController()
@property(nonatomic, strong) Block* block
@end

@implmentation NewViewController

- (void)testBlock {
    [self.block addblock:^{
        self.view.backgroundColor = [UIColor redColor];
    }];
}

@end

typedef void (^MyBlock)();

@inteface Block
@propery(nonatomic, strong) MyBlock block;
@end

@implmentation

- (void)addBlock:(MyBlock)block {
    self.block = block;
    if (self.block) {
        self.block();
    }
}

@end
```

### 接上边的block，以下代码会有问题吗

```Objective-C
    __weak typeof(self) weakSelf = self;
    [self.block addblock:^{
        __strong typeof(weakSelf) strongSelf = self;
        strongSelf->preson.name = @"";
        strongSelf->preson.object = [NSObject new];
        // weakSelf->person.name = @"jiang";
    }];
```

1. 延迟执行block会有问题吗？不会，因为有strong保证VC不会提前释放
2. self为null会有问题吗
3. weakSelf->直接访问会奔溃（成员变量），.name走的是消息转发不会奔溃

`OC向nil发送消息，是不会崩溃的。 OC的函数调用都是通过objc_msgSend进行消息发送来实现，相对于C/C++来说，操作空指针引起crash问题，
    objc_msgSend通过判断self来决定是否发送消息，self为nil，那么selector也为空，直接返回，所以不会崩溃
`    

### 是在新的子线程执行的吗？以下代码打印顺序是怎么样的？

```Objective-C
- (void)MyLog {
    NSLog(@"3");
}
- (void)testBlock2 {
    NSLog(@"1---%@",[NSThread currentThread]);
    dispatch_queue_t q2 = dispatch_queue_create("com.queue.queue2", DISPATCH_QUEUE_SERIAL);
    dispatch_async(q2, ^{
        NSLog(@"2---%@",[NSThread currentThread]);
        NSLog(@"1");
        [self performSelector:@selector(MyLog) withObject:self afterDelay:2.0];
        NSLog(@"2");
    });
}
```

打印出"12"， 3不会打印，2执行完后，线程就退出了

## 下边代码会有问题吗？

```Objective-C
dispatch_queue_t q3 = dispatch_queue_create("com.queue.queue3", DISPATCH_QUEUE_SERIAL);

for (int i = 0; i < 100; ++i) {
 dispatch_async(q3, ^{
          person.name = @"jiang";
          person.obj = [NSObject new];

    })；
}
```

对象重新赋值，会多次release，造成奔溃。



