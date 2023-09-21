---
title: 项目-动态化方案
date: 2023-09-21 16:17:55
tags:
---

## Question:

1. ReactNative的渲染过程了解吗
2. 如何保持双端UI的一致性
3. JS的数据如何转换为OC对应的数据类型
4. 关注哪些性能指标

## 如何保持双端一致性：

1. 应用统一的布局引擎Yoga,保证布局理念和原理
2. 原子组件，文本、图片、Yoga语义效果一直，文本设计师出规范，行高字体大小确定的情况下居中对齐
3. 缩放模式，双端都以375屏幕为基础单位，　iPhone7/iPhone8位标准，其他屏幕大小进行缩放
4. 除了静态的UI对齐，只要语义一致就可以了


## 关注哪些性能指标

https://www.51cto.com/article/681298.html

*  首屏绘制（First Paint，FP）
* 首屏内容绘制（First Contentful Paint，FCP）
* 可交互时间（Time to Interactive，TTI）
* 最大内容绘制 (Largest Contentful Paint，LCP)
* 首次有效绘制（First Meaning Paint, FMP）

