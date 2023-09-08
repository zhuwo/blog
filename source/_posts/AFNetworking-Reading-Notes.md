---
title: AFNetworking-Reading-Notes
date: 2023-09-04 14:25:29
tags:
---


  知名的网络第三方开源库。

  ## 作用

  1. session的任务的回调收拢，和任务绑定进度回调，https的安全处理也包括了


  ## 类

AFHTTPSessionManager继承自AFURLSessionManager，使用AFJSONResponseSerializer，AFSecurityPolicy（https）,AFNetworkReachabilityManager（网络状态）NSURLSessionConfiguration（URLSession配置，关于http协议的很多实现，例如http2.0支持keep-alive，保留tcp连接）

NSURLSession设置的delegateQueue，是回调的时候的处理队列，当设置maxConcurrentOperationCount为1, 可以达到并发的请求串行的进行回调的效果。



## 核心类AFURLSessionManager

首先对所有dataTask,uploadTask,downLoadTask，添加监听代理（可以获取进度）和通知。
taskDidResume/taskDidSuspend，接收taskResume，并发出通知AFNetworkingTaskDidResumeNotification，taskDidSuspend同理

所有代理有统一词典管理mutableTaskDelegatesKeyedByTaskIdentifier。任务结束didCompleteWithError，移除代理，并调用taskDidComplete回调。tasksForKeyPath，获取对应任务，留给子类使用。

hook resume/suspend方法，发送对应通知
    1. 给请求封装了进度的回调，包括上传/下载/数据任务.
    2. 将NSURLSession的所有代理，包装成了block，跟任务绑定进度

接收挑战时（didReceiveChallenge），用securityPolicy进行证书认证等SSL流程。


### _AFURLSessionTaskSwizzling

傀儡类，给localDataTask(sessionTask)添加af_resume方法，af_suspend方法，并hook系统的resume/suspend方法，获取resume和suspend的通知，发送对应通知给AFURLSessionManager。

### AFURLSessionManagerTaskDelegate

主要是记录进度，并持有进度回调block以便任务能添加进度回调，以及完成任务的回调，同时收拢不同任务（比方说都跟下载进度相关有数据任务、下载任务等都统一收拢到downloadProgress）。

通过KVO，用来接收进度变化，并回调进度和完成的任务，包含NSURLSessionTaskDelegate、NSURLSessionDataDelegate（下载、上传）、NSURLSessionDownloadDelegate（下载）。


### AFSecurityPolicy

    猜测大概是https的应用，模式有pinningModeCertificate/publicKey/None，none就是什么也不验证，Certificate验证证书所有字段，包括有效期之内，publicKey只验证证书中的公钥。
    pinningModeCertificate模式下，会建立ssl，认证domain的有效性

### AFNetworkReachabilityManager

    网络状态检测单例，AFNetworkReachabilityStatus网络状态，wifi/4g/无？，初始化就会用linux系统方法获取reachability，然后startMonitoring方法会注册监听网络状态的变化，并发送通知（如果网络状态变化，SCNetworkReachability系列系统方法）。

### AFJSONResponseSerializer

AFJSONResponseSerializer继承自AFHTTPResponseSerializer，拥有acceptableStatusCodes（2**）和acceptableContentTypes（MINE中的），AFJSONResponseSerializer包含"application/json", @"text/json", @"text/javascript"，其中responseObjectForResponse方法，会将错误包装返回

## AFHTTPSessionManager

对外接口，http协议的get/post/put/delete/...

请求使用AFURLRequestSerialization构建，并用父类方法dataTaskWithRequest发起请求

### AFURLRequestSerialization

请求的封装（封装复杂的http请求构建）

### AFURLResponseSerialization

对response的反序列化。


## UIKit

### AFNetworkActivityIndicatorManager

    如何处理网络请求过快？用timer
    如何处理网络请求过多? 用计数器表示

## iOS基础知识

NSLock,NSOperationQueue,SCNetworkReachability
