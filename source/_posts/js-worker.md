---
title: js_worker
date: 2023-11-10 15:40:06
tags:
- javascript
- 多线程
categories:
- 技术
- javascript
---


## Worker理解

    简而言之：JS上的多线程处理。
    JS的传统模型只在一个主线程运行代码，遇到复杂计算的过程容易引起页面卡顿，为此引入Worker，worker中的代码，单独在一个字线程进行跑，和原生开发一样，可以避免复杂计算影响主线程渲染页面。

## Worker使用

1. 创建worker

```javascript
    const worker = new Worker("path to worker js file")
```
2. 事件触发，给worker发消息

```javascript
    worker.postMessage(message)
```

3. worker接收消息，并处理
```javascript
    onmessage = function(e) {
        // TO DO Complex Compute
    }
```

## 注意点

1. Worker中的API是JS的一个子集，并不是所有API都能跑的： https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers



## demo
    
https://mdn.github.io/dom-examples/web-workers/simple-web-worker/
