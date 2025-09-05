---
title: algorithms
date: 2023-07-12 09:41:30
categories:
- 技术
- 算法
tags: 
- algorithms
---

## 算法和数据结构

### 动态规划

1. 寻找最优子结构
2. 递归定义最优解的值
3. 计算最优解的值，通常使用自底向上的方法
4. 利用计算出的信息构造一个最优解

### 回溯

1. DFS+遍历探索每一层的每一种可能
2. 画出所有树状链路就能写出代码
2. 效率比较低

### 字符串匹配

目标字符串T，和匹配模式P（子字符串）

#### 有限自动机

1. Pattern内部自己构建有限自动机，目标是加上的新的字符以后，最长前缀字符串和目前扫描到的后缀字符串一样
2. 基于构建好的自动机，O(n)扫描字符串，如果中间达到终态，匹配成功；否则，匹配失败
3. 构建有限自动机：对于模式P，和字符集，扫描模式，当前字符a,对于Pi，找寻Pi a的最长前缀Pk，则加字符a，转移到k的状态
4. 算法的复杂度为扫描O(n) + 构建有限自动机的复杂度

有限自动机的输入：当前状态q, 以及新的输入T[i], 输出就是新的状态

```
m = P.length
FINITE-AUTOMATON-MATCHER(T, delta, m)
    n = T.length
    q = 0
    for i = 1 to n
        q = delta(q, T[i])
        if (q == m) 
            print "Pattern occurs with shift " i - m
```
```
COMPUTE-TRANSITION-FUNCTION(P, 字符集)
m = P.length
for q = 0 to m
    for each character a 属于 字符集
        k = min (m + 1, q + 2)
        repeat
            k = k - 1
        until Pk 是 Pq.a的后缀
        delta(q, a) = k
return delta
```

=> KMP是有限自动机的一种有效实现：

用\pi添加额外的存储空间，表示P[q]的后缀中最长真前缀，也就是用模式内部的匹配形式，来表达有限自动机。算法执行流程，就是当前字符不匹配的时候，逐步回退到\pi的地方，再去判断，如何符合就继续，直到回退到开头的形式，这样就利用了模式字符串之前就有的模式子串。
理解：用Pattern字符串内部的最长前缀子串，来探索的匹配新的字符，直到最新的输入匹配到子串或当前串，直到匹配的长度达到Pattern的长度。

```
KMP-Matcher(T, P)
    n = T.length
    m = P.length
    \pi  = Compute_Prefix_Function(P)
    for i from 1 to n
        while q > 0 and P[q+1] != T[i]
            q = \pi[q]
        if P[q+1] == T[i]
            q == q + 1
        if q == m
            Print "Positon at " m - i
            q = \pi[q] // Find next Mathcher

Compute_Prefix_Function(P)
    n = P.length
    k = 0
    let \pi be an new array from 1 to n
    \pi[1] = 0
    for q from 2 to n
        while k > 0 and P[k+1] != P[q]
            k = \pi[k]
        if P[k+1] == P[q]
            k = k + 1
        \pi[q] = k

```

## 图论

存储方式：邻接矩阵，邻接链表。

搜索算法，记录前驱节点，白灰黑三色记录状态，白色记录未被访问，灰色，访问过但是没有访问完，黑色，访问完成。

### 广度优先搜索

除了记录前驱，还可以记录深度

用队列实现，遍历完所有相邻节点后，标记为黑。

### 深度优先搜索

一般递归实现，直到递归回来，节点才标记为黑色。