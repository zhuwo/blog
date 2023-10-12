---
title: The super tiny compiler研究
date: 2023-10-12 14:03:17
tags:
- 表达式
- compiler
categories:
- 技术
- 编译
---

[The super tiny compiler](git@github.com:jamiebuilds/the-super-tiny-compiler.git)

Github非常有名的超级简单的编译器实现。

## 测试

从作者的测试着手，了解编译器的功能。

### 输入输出

```javascript
const input  = '(add 2 (subtract 4 2))';
const output = 'add(2, subtract(4, 2));';
```

输入input是个后缀表达式，后缀表达式的操作符为啥以`(`开头呢，我猜测是为了方便判断是操作符，一个操作的开头都是操作符，结束的地方以`)`结束一个简单操作，比如：`(subtract 4 2)`。
输出output是个中缀表达式。

### 完整编译过程

    1. tokenizer，词法分析，inuput输入=>token数组
    2. parser，语法分析，token数组=>AST
    3. transformer,语法树替换，AST=>newAST
    4. codeGenerator, newAST=>output
    5. compiler, 输入input=>输出output

## 


