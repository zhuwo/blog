---
title: C++的一些简要操作笔记
date: 2023-10-01 14:34:29
categories:
- 技术
- C++
tags:
- C++
- CPP
---


##  清空queue c++ clear queue

```c++
std::queue<int> empty;
std::swap( q, empty );
```

## 交换vector两个位置的值

```c++
std::swap(nums[a], nums[b]);
```

## 二维vector初始化

```c++
vector(numCourses, vector<int>());
```