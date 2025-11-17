---
title: ios性能监控
date: 2025-11-14 19:35:10
categories:
- 技术
- iOS
tags:
- iOS
- APM
---


## Runloop 整体过程

1. Notify observers that the run loop has been entered.
2. Notify observers that any ready timers are about to fire.
3. Notify observers that any input sources that are not port based are about to fire.
4. Fire any non-port-based input sources that are ready to fire.
5. If a port-based input source is ready and waiting to fire, process the event immediately. Go to step 9.
6. Notify observers that the thread is about to sleep.
7. Put the thread to sleep until one of the following events occurs:
    * An event arrives for a port-based input source.
    * A timer fires.
    * The timeout value set for the run loop expires. (runloop 超时了)
    * The run loop is explicitly woken up.
8. Notify observers that the thread just woke up.
9. Process the pending event.
    * If a user-defined timer fired, process the timer event and restart the loop. Go to step 2.
    * If an input source fired, deliver the event.
    * If the run loop was explicitly woken up but has not yet timed out, restart the loop. Go to step 2.

代码：

```objc
// 1.进入loop
__CFRunLoopRun(runloop, currentMode, seconds, returnAfterSourceHandled)

// 2.RunLoop 即将触发 Timer 回调。
__CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeTimers);
// 3.RunLoop 即将触发 Source0 (非port) 回调。
__CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeSources);
// 4.RunLoop 触发 Source0 (非port) 回调。
sourceHandledThisLoop = __CFRunLoopDoSources0(runloop, currentMode, stopAfterHandle)
// 5.执行被加入的block
__CFRunLoopDoBlocks(runloop, currentMode);

// 6.RunLoop 的线程即将进入休眠(sleep)。
__CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);

// 7.调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
__CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort)


// 进入休眠

// 8.RunLoop 的线程刚刚被唤醒了。
__CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopAfterWaiting

// 9.如果一个 Timer 到时间了，触发这个Timer的回调
__CFRunLoopDoTimers(runloop, currentMode, mach_absolute_time())

// 10.如果有dispatch到main_queue的block，执行bloc
 __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
 
 // 11.如果一个 Source1 (基于port) 发出事件了，处理这个事件
__CFRunLoopDoSource1(runloop, currentMode, source1, msg);

// 12.RunLoop 即将退出
__CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);

```


## 监控代码

```objc

- (void)startMonitor {
    
    CFRunLoopObserverContext context = {0,(__bridge void*)self,NULL,NULL};
    runLoopObserver = CFRunLoopObserverCreate(kCFAllocatorDefault,kCFRunLoopAllActivities,YES,0,&runLoopObserverCallBack,&context);
    
    //创建子线程监控
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        //子线程开启一个持续的 loop 用来进行监控
        while (YES) {
            long semaphoreWait = dispatch_semaphore_wait(dispatchSemaphore, dispatch_time(DISPATCH_TIME_NOW, 3 * NSEC_PER_SEC));
            if (semaphoreWait != 0) {
                if (!runLoopObserver) {
                    timeoutCount = 0;
                    dispatchSemaphore = 0;
                    runLoopActivity = 0;
                    return;
                }
                //BeforeSources 和 AfterWaiting 这两个状态能够检测到是否卡顿
                if (runLoopActivity == kCFRunLoopBeforeSources || runLoopActivity == kCFRunLoopAfterWaiting) {
                    //将堆栈信息上报服务器的代码放到这里
                } //end activity
            }// end semaphore wait
            timeoutCount = 0;
        }// end while
    });

}

static void runLoopObserverCallBack(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info) {
    BGPerformanceMonitor *monitor = (__bridge BGPerformanceMonitor*)info;
    
    // 记录状态值
    monitor->activity = activity;
    
    // 发送信号
    dispatch_semaphore_t semaphore = monitor->semaphore;
    long st = dispatch_semaphore_signal(semaphore);
    NSLog(@"dispatch_semaphore_signal:st=%ld,time:%@",st,[BGPerformanceMonitor getCurTime]);
}

```

kCFRunLoopBeforeSources和kCFRunLoopAfterWaiting之间，有人可能会问kCFRunLoopAfterWaiting之后也有一些方法调用，为什么不监测呢，我的理解，大部分导致卡顿的的方法是在kCFRunLoopBeforeSources和kCFRunLoopAfterWaiting之间，比如source0主要是处理App内部事件，App自己负责管理(出发),如UIEvent(Touch事件等，GS发起到RunLoop运行再到事件回调到UI)、CFSocketRef。


然后再runLoopObserverCallBack中每次都 signal， 如果一个状态卡很久就可以认为是卡顿了，
这边可以做一些策略，比方说连续 5 次超过 80ms，或者单次超过 250ms。

