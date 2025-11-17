---
title: iOS Crash上报原理
date: 2025-11-12 19:49:15
tags:
- iOS
- 面试
categories:
- 技术
- iOS
---


## 捕获异常类型

1. Objective-C NSException （OC异常）
2. Mach异常（底层）
3. 内存异常（OOM）
4. Signal 异常（对Mach的封转）
5. C++异常
6. Zoombine
7. 死锁DeadLock

## 大致代码分布

intall， 将配置中的所有异常处理安装，会分别调用每个API。

KSCrashMonitor_XXXException收集和处理不同异常，回调到KSCrashMonitor的notifyException和handleException， notify是接收到异常，handleException是所有处理都结束，context 收集完毕；onExceptionEvent调用KSCrashReportC中的report方法记录异常。


##  Objective-C NSException （OC异常）

1. 通过NSSetUncaughtExceptionHandler 获取异常处理
2. 获取CPU状态和task所有线程，获取所有线程？后续转换为堆栈？
3. 尝试从callStackReturnAddresses，异常获取异常堆栈，如果不行，就用系统方法backtrace， 获取当前线程的调用堆栈
4. 记录异常名、异常 userInfo、Reason等

补充：shouldRecordAllThreads为true时候，会将所有线程的上述状态记录到context中


## Mach异常捕获

关心的异常：

EXC_MASK_BAD_ACCESS | EXC_MASK_BAD_INSTRUCTION | EXC_MASK_ARITHMETIC | EXC_MASK_SOFTWARE | EXC_MASK_BREAKPOINT

简单来说，就是：
1. 保存系统的异常处理：task_get_exception_ports
2. 新建一个自己的异常处理：task_set_exception_ports
3. 拦截后，上报处理后，还原为保存的异常处理器，
如果可以处理：发送‘KERN_SUCCESS’消息，线程会重新执行**错误指令**， 原来的异常处理会执行，又出错，然后，结束
如果不能处理，发送KERN_FAILURE， 内核会把权限交给 host port， 然后结束此次崩溃

### startNewExceptionHandler 添加自己的异常处理器

![自定义Mach异常捕获](/images/mach_exception_handler.png) 

1. 初始化ExceptionContext：清除上下文结构体并设置基本信息
2. 分配Mach端口：通过mach_port_allocate创建接收端口
3. 插入端口权限：使用mach_port_insert_right添加发送权限
4. 设置异常端口为ctx->exceptionPort：通过task_set_exception_ports注册异常处理
5. 保存恢复点：调用saveExceptionPortsRestorePoint保存当前状态
6. 创建线程：使用pthread_create创建异常处理线程  （线程监听端口ctx->exceptionPort， 就能处理异常信息）
7. 分离线程：调用pthread_detach使线程独立运行
8. 添加保留线程：通过ksmc_addReservedThread标记为保留线程


代码
`kern_return_t kr = mach_msg(&ctx->request->Head, MACH_RCV_MSG | MACH_RCV_LARGE, 0, ctx->request->Head.msgh_size,
                                ctx->exceptionPort, MACH_MSG_TIMEOUT_NONE, MACH_PORT_NULL);`监听了exceptionPort的消息，

一旦异发生就会受到消息
								

## Signal异常

FatalSignal 
[SIGABRT, SIGBUS, SIGFPE, SIGILL, SIGPIPE, SIGSEGV, SIGSYS, SIGTRAP, SIGTERM]

步骤：
1. 分配并设置备用信号栈signalStack, 针对堆栈溢出的场景 （当前线程栈可能已损坏，无法在当前栈上安全执行信号处理器。使用备用栈能增加处理器运行成功写报告的概率）
2. previousSignalHandlers存储就的信号行为（系统自带的）， 
3. 循环，用sigaction接管关心的异常信号，用handleSignal处理
4. 如果接管信号失败，就reverse damage， 将信号处理替换为存到previousSignalHandlers的信号处理


信号处理，替换成以下自己的信号处理，handleSignal
```c
static void handleSignal(int sigNum, siginfo_t *signalInfo, void *userContext)
```
### handleSignal 

ksmc_getContextForSignal(userContext, &machineContext);将信号的寄存器快照userContext，拷贝到了machineContext中， 主要是STRUCT_MCONTEXT_L machineContext （包含异常的信息）

同时记录信号的信息，包括signum(对应信号的具体枚举)，错误地址fault_Adress, 信号码sig_code

## C++异常捕获（CPPException）

set_terminate设置异常捕获的回调， `CPPExceptionTerminate`。
originalTerminateHandler保存之前的系统回调

### CPPExceptionTerminate

* __cxxabiv1::__cxa_current_exception_type()获取异常类型

* 排除NSException，让originalTerminateHandler处理。（NSException 异常用 C++异常实现，会重复）

* rethrow异常， 捕获不同异常类型处理
```c++
try {
    throw;
} catch (std::exception *exc){

}
```
包含exception类型、string类型、以及其他各种基础类型。

* descriptionBuff存储异常的字符串， 最后记录到crashReason

* ksmc_getContextForThread 线程信息都会记录到Context中

__cxa_throw 重新抛出异常， __cxa_throw  内部会调用 unwind，清理函数调用栈，所以重新__cxa_throw， 先捕获stackTrace， 再调用会原来的__cxa_throw
```c++
void __cxa_throw(void *thrown_exception, std::type_info *tinfo, void (*dest)(void *)) KS_KEEP_FUNCTION_IN_STACKTRACE
{
    static cxa_throw_type orig_cxa_throw = NULL;
    if (g_state.cxaSwapEnabled == false) {
        captureStackTrace(thrown_exception, tinfo, dest);
    }
    unlikely_if(orig_cxa_throw == NULL) { orig_cxa_throw = (cxa_throw_type)dlsym(RTLD_NEXT, "__cxa_throw"); }
    orig_cxa_throw(thrown_exception, tinfo, dest);
    KS_THWART_TAIL_CALL_OPTIMISATION
    __builtin_unreachable();
}
```

## 主线程 DeadLock

主要代码
```objc
- (void)runMonitor
{
    BOOL cancelled = NO;
    do {
        // Only do a watchdog check if the watchdog interval is > 0.
        // If the interval is <= 0, just idle until the user changes it.
        @autoreleasepool {
            NSTimeInterval sleepInterval = g_watchdogInterval;
            BOOL runWatchdogCheck = sleepInterval > 0;
            if (!runWatchdogCheck) {
                sleepInterval = kIdleInterval;
            }
            [NSThread sleepForTimeInterval:sleepInterval];
            cancelled = self.monitorThread.isCancelled;
            if (!cancelled && runWatchdogCheck) {
                if (self.awaitingResponse) {
                    [self handleDeadlock];
                } else {
                    [self watchdogPulse];
                }
            }
        }
    } while (!cancelled);
}

- (void)watchdogPulse
{
    __block id blockSelf = self;
    self.awaitingResponse = YES;
    dispatch_async(dispatch_get_main_queue(), ^{
        [blockSelf watchdogAnswer];
    });
}

- (void)watchdogAnswer
{
    self.awaitingResponse = NO;
}
```

`watchdogPulse`子线程awaitingResponse置为true，dispatch到主线程置为 false，下一次，子线程休眠5s后，如果awaitingResponse还是true，说明主线程没有响应，则认为死锁，用handleDeadlock处理

### handleDeadlock

记录主线程的线程信息

缺陷：主线程任务多，卡顿等行为也有类似现象，会被误认为是死锁进行处理

## 


### handleException

和OC异常类似，只不过，先会通过waitForException中的mach_msg，捕获异常消息， 直接将消息起点给了ExceptionRequest。

ExceptionRequest实际是__Request__exception_raise_t，

```c
typedef struct {
		mach_msg_header_t Head;
		/* start of the kernel processed data */
		mach_msg_body_t msgh_body; //消息体
		mach_msg_port_descriptor_t thread; // 线程
		mach_msg_port_descriptor_t task; // 任务
		/* end of the kernel processed data */
		NDR_record_t NDR;
		exception_type_t exception; // 异常类型
		mach_msg_type_number_t codeCnt;
		integer_t code[2];
	} __Request__exception_raise_t __attribute__((unused));
```

task，可以获取到所有相关的线程，

context->

request->exception 错误类型获取，
request->code[0] 错误码，code
request->code[1] 子错误码，subcode

ksmc_getContextForThread复用函数，和KSCrashMonitor_NSException一样，获取thread， threadList。


stackCursor初始化，大概是为了还原堆栈用的。

## Zombie Monitor

僵尸对象监控器通过一种精巧的内存追踪机制来发现对已释放对象的非法访问。它通过替换 NSObject 和 NSProxy 的 dealloc 方法，在对象被释放时记录下对象的地址和类型信息。这些信息被存储在一个经过优化的哈希表中，当程序试图访问一个已释放的对象时，监控器能够快速定位到这个对象的原始类型，帮助开发者准确定位内存访问问题。除了跟踪对象的释放信息，该监控器还能捕获对象释放过程中可能发生的 NSException 异常，为内存问题的诊断提供更全面的信息。

![Zombie Monitor](/images/Zombie_Monitor.png) 

## Memory Monitor

内存状态：

* Normal (< 25%): Full functionality, optimal performance
* Warning (25-50%): Begin reducing memory usage, optimize caches
* Urgent (50-75%): Significant memory reduction needed
* Critical (75-90%): Aggressive cleanup required
* Terminal (> 90%): Imminent termination risk, emergency measures


ksmemory_map, 会把g_memoryURL对应的文件和内存g_memory映射，当内存写入的时候，文件也会被写入；所以启动的时候直接读取ksmemory_map。

因为OOM的处理，在信号、Mach异常等异常的后边，所以OOM上报是上一次APP崩溃的时候是OOM情况。

### KSCrashAppMemoryTracker
1. 包装了Memory Pressure, 比didReceiveMemoryWarning更有用，支持不同级别的内存压力level。
2. 支持内存级别（memory level）

大致流程：
dispatch source（调度源）监听内存压力`DISPATCH_SOURCE_TYPE_MEMORYPRESSURE`，监听到pressure变化的时候，通知observers。

timer 监听，DISPATCH_SOURCE_TYPE_TIMER，每100ms触发hearBeat, 内存变化超过1MB，通知observers，发出通知。


footprint 占比超过 95%，认为是KSCrashAppMemoryStateTerminal，表示即将或者已经OOM。

### 其他OOM监控方法

可以 hook系统的malloc方法， malloc_logger，然后根据它里边的参数信息判断是否分配过大的内存。



## 辅助函数

```c
bool kscm_addMonitor(const KSCrashMonitorAPI *api)
{
    static KSCrash_ExceptionHandlerCallbacks exceptionCallbacks = {
        .notify = notifyException,
        .handle = handleException,
    };
    api->init(&exceptionCallbacks);
}
```
setMonitor 的时候，就注册了 callback回调， notify 和 handle

notifify猜测是刚开始处理异常的时候调用的，没有研读内容
handle 是获取上下文内容以后调用，g_state.callbacks.handle(monitorCtx);

### notifyException

处理异常的一些状态：
1. no other exceptions， 正常状态
2. a non-fatal exception 正在处理，不用关心
3. a fatal exception正在处理，block 当前处理线程
4. 在同一个线程遇到一样的异常，按 recrash 处理
5. 同样的线程递归崩溃， 陷入了崩溃的循环，则需要exit APP, 不能abort，会进一步抛出异常
6. shouldRecordAllThreads的时候，将所有相关线程挂起（以方便记录）

### handleException

1. Context添加更详细的上下文信息addContextualInfoToEvent， 比方说Mach_Exception）， 添加信号类型：EXC_BAD_ACCESS、EXC_BAD_INSTRUCTION、EXC_ARITHMETIC （针对 Mach_Exception）
2. onExceptionEvent， 异常发生处理，进行上报
3. isFatal异常，则将自己卸载(uninstall), 让其他异常处理库能进行
4. 将挂起的线程重新恢复

## 实现

由于崩溃发生时网络可能不可用，建议将崩溃信息先写入本地存储（如文件系统或数据库），然后在下次启动时再尝试上传

## WriteReport

1. Mach信息和Signal信息都会尝试输出，比较底层

StackCursor 通过 Context 的 machineContext， 然后用KSCPU_arm64的方法可以把堆栈复现

## KSCPU_arm64

获取寄存器内容, pc, sp, lr(link register)...

## KSCrashAppStateTracker

APP状态变化，可以添加Observer

## 其他

* KSCrashMonitor_AppState， app状态： forground\background\isactive 等等
* KSCrashMonitor_System， 系统信息：os版本、app版本、uuid 等等
* KSCrashMonitor_User， 自定义的异常上报




## 参考

* [APM - iOS Crash 异常捕获原理](https://juejin.cn/post/7370526031700377652)

Task相关概念
* [MacOSX Mach IPC 入门](https://kiprey.github.io/2021/12/mach_ipc_intro/)