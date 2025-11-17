---
title: ios-mesasge-send
categories:
- 技术
- iOS
tags:
- iOS
- 消息转发
- runtime
---


总体流程：
1. 查找缓存
2. 按继承链查找
3. 动态决议，添加方法
4. 快速转发
5. 慢速转发， methodSignatureForSelector, forwardInvocation:invocation  invocation.selector


## 方法缓存

class->cache_t

catch_t _buckets

buckets: buckect_t[]

buckect_t 包含 SEL, IMP

通过Hash存储，如果有就直接用
慢方法查找的时候，插入 buckets

线性探测解决 hash 冲突，根据key也就是sel判断是否找到， 扩容的时候重新调整。




## 实例对象

```objc
Ivar object_getInstanceVariable(id obj, const char *name, void **value)
{
    if (obj  &&  name  &&  !obj->isTaggedPointer()) {
        Ivar ivar;
        if ((ivar = class_getInstanceVariable(obj->ISA(), name))) {
            if (value) *value = (void *)object_getIvar(obj, ivar);
            return ivar;
        }
    }
    if (value) *value = nil;
    return nil;
}

id object_getIvar(id obj, Ivar ivar)
{
    if (!obj  ||  !ivar  ||  obj->isTaggedPointer()) return nil;

    ptrdiff_t offset;
    objc_ivar_memory_management_t memoryManagement;
    _class_lookUpIvar(obj->ISA(), ivar, offset, memoryManagement);

    id *location = (id *)((char *)obj + offset); // offset 获取到内容

    if (memoryManagement == objc_ivar_memoryWeak) {
        return objc_loadWeak(location);
    } else {
        return *location;
    }
}
```

从上述代码可以看书，对象的实例对象在某个固定的字段后边（ISA），然后根据类的ivars分配内存， 所以可以直接根据offset获取到内容。

```c
isa 

 uintptr_t nonpointer        : 1;                                       \
      uintptr_t has_assoc         : 1;                                       \
      uintptr_t has_cxx_dtor      : 1;                                       \
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ \
      uintptr_t magic             : 6;                                       \
      uintptr_t weakly_referenced : 1;                                       \
      uintptr_t deallocating      : 1;                                       \
      uintptr_t has_sidetable_rc  : 1;                                       \
      uintptr_t extra_rc          : 19


```
shiftcls左移三位找到类对象地址。

在 64 位系统中，类对象的地址通常是 8 字节对齐的，因此低 3 位总是 0。为了节省空间，runtime 将地址右移 3 位后存储在 shiftcls 中

## 方法列表

class_ro_t 只读，编译期间确认的
class_rw_t 可读写，动态运行过程中

