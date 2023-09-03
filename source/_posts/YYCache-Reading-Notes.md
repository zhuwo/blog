---
title: YYCache-Reading-Notes
date: 2023-09-03 12:47:34
tags:
---


  优秀的Cache库，使用LRU实现，拥有较好的性能。

  ## 结构

  YYCache由DiskCache和MemoryCache组成，主要Api: containsObjectForKey，setObjectForKey, removeObjectForKey，removeAllObjects等等。


## MemoryCache

mutex保证线程安全，所有操作都加互斥锁

关于剪枝：
trimToCost：不断的删除末尾节点，知道消耗和小雨costLimit
autoTrimInterval:每隔5s，定时减末尾的枝

LRU:
    所有操作都会将当前节点放到头部节点，保证最近使用的放到最前。

## DiskCache

globalCache存储所有storage的最大NSMapTable
使用YYKVStorage存储到文件或数据库
YYLinkedMapNode：双向链表，存储节点，
YYLinkedMap， _head， _tail

dispatch_semaphore_t保证线程安全

使用LRU删除节点，删除访问时间最老的数据，直到满足大小/个数条件为止

所有操作都是YYKVStorage的操作，添加队列操作，增加一些功能（进度。。。），以及加锁保证线程安全。

### YYKVStorage

saveItemWithKey：优先存文件，存文件失败村sqlite
removeItemForKey: 根据类型，删除文件或数据库内容
removeItemsLargerThan（Size/Time）:数据库字段已经存入，根据条件找到对应FileNames并删除
剩下都是数据库操作



### iOS支持要点

NSMapTable, 对key,value弱引用
NSKeyedArchiver

  