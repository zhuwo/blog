---
title: producer_consumer
date: 2023-09-09 10:42:40
categories:
- 技术
- 操作系统
tags: 
- 多线程
---


iOS 实现解决生产者消费者问题


1. 先是生产者要解决生产者过剩的问题， 消费者没有可消费的产品要等待
2. 考虑CPU，++操作可能不是原子操作，需要对齐进行加锁，解决线程不安全问题


```Objective-C


@proppery(nonatomic, strong) dispatch_semaphore_t pro_semaphore;
@proppery(nonatomic, strong) dispatch_semaphore_t cus_semaphore;

@proppery(nonatomic, strong) NSInterger producer_count;
@proppery(nonatomic, strong) NSLock lock;

#define BUFFER_SIZE 5;


- (void)init {
    if (self = [super init]) {
        self.pro_semaphore = dispatch_semaphore_create(BUFFER_SIZE);
        self.cus_semaphore = dispatch_semaphore_create(0);
        self.producer_count = 0;
        self.lock = [[NSLock alloc] init];

    }
    return self;
}


- (void)producer {
    dispath_async(dispath_global_queue(0, 0), ^{
        dispatch_semaphore_wait(_pro_semaphore);
        [self.lock lock];
        producer_count++;
        [self.lock unlock];
        dispatch_semaphore_signal(_cus_semaphore);
    });

}

- (void)consumer {
     dispath_async(dispath_global_queue(0, 0), ^{
        dispatch_semaphore_wait(_cus_semaphore);
        [self.lock lock];
        producer_count--;
        [self.lock unlock];
        dispatch_semaphore_signal(_pro_semaphore);
    });
}


```



## 参考

* [GCD 解决生产者消费者问题](https://crmo.github.io/2019/06/16/GCD%20%E8%A7%A3%E5%86%B3%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85%E9%97%AE%E9%A2%98/)

