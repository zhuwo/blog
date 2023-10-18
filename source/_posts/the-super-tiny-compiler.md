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

## 词法分析 *Lexical Analysis*

扫描输入，变为：
```javascript
    [
      { type: 'paren',  value: '('        },
      { type: 'name',   value: 'add'      },
      { type: 'number', value: '2'        },
      { type: 'paren',  value: '('        },
      { type: 'name',   value: 'subtract' },
      { type: 'number', value: '4'        },
      { type: 'number', value: '2'        },
      { type: 'paren',  value: ')'        },
      { type: 'paren',  value: ')'        },
    ]
 ```

 实际程序：
    扫描字符，状态机
    如果遇到数字，就是数字；遇到(就是(。

```javascript
function tokenizer(input) {

  // A `current` variable for tracking our position in the code like a cursor.
  let current = 0;

  // And a `tokens` array for pushing our tokens to.
  let tokens = [];

  // We start by creating a `while` loop where we are setting up our `current`
  // variable to be incremented as much as we want `inside` the loop.
  //
  // We do this because we may want to increment `current` many times within a
  // single loop because our tokens can be any length.
  while (current < input.length) {

    // We're also going to store the `current` character in the `input`.
    let char = input[current];

    // The first thing we want to check for is an open parenthesis. This will
    // later be used for `CallExpression` but for now we only care about the
    // character.
    //
    // We check to see if we have an open parenthesis:
    if (char === '(') {

      // If we do, we push a new token with the type `paren` and set the value
      // to an open parenthesis.
      tokens.push({
        type: 'paren',
        value: '(',
      });

      // Then we increment `current`
      current++;

      // And we `continue` onto the next cycle of the loop.
      continue;
    }

    // Next we're going to check for a closing parenthesis. We do the same exact
    // thing as before: Check for a closing parenthesis, add a new token,
    // increment `current`, and `continue`.
    if (char === ')') {
      tokens.push({
        type: 'paren',
        value: ')',
      });
      current++;
      continue;
    }

    // Moving on, we're now going to check for whitespace. This is interesting
    // because we care that whitespace exists to separate characters, but it
    // isn't actually important for us to store as a token. We would only throw
    // it out later.
    //
    // So here we're just going to test for existence and if it does exist we're
    // going to just `continue` on.
    let WHITESPACE = /\s/;
    if (WHITESPACE.test(char)) {
      current++;
      continue;
    }

    // The next type of token is a number. This is different than what we have
    // seen before because a number could be any number of characters and we
    // want to capture the entire sequence of characters as one token.
    //
    //   (add 123 456)
    //        ^^^ ^^^
    //        Only two separate tokens
    //
    // So we start this off when we encounter the first number in a sequence.
    let NUMBERS = /[0-9]/;
    if (NUMBERS.test(char)) {

      // We're going to create a `value` string that we are going to push
      // characters to.
      let value = '';

      // Then we're going to loop through each character in the sequence until
      // we encounter a character that is not a number, pushing each character
      // that is a number to our `value` and incrementing `current` as we go.
      while (NUMBERS.test(char)) {
        value += char;
        char = input[++current];
      }

      // After that we push our `number` token to the `tokens` array.
      tokens.push({ type: 'number', value });

      // And we continue on.
      continue;
    }

    // We'll also add support for strings in our language which will be any
    // text surrounded by double quotes (").
    //
    //   (concat "foo" "bar")
    //            ^^^   ^^^ string tokens
    //
    // We'll start by checking for the opening quote:
    if (char === '"') {
      // Keep a `value` variable for building up our string token.
      let value = '';

      // We'll skip the opening double quote in our token.
      char = input[++current];

      // Then we'll iterate through each character until we reach another
      // double quote.
      while (char !== '"') {
        value += char;
        char = input[++current];
      }

      // Skip the closing double quote.
      char = input[++current];

      // And add our `string` token to the `tokens` array.
      tokens.push({ type: 'string', value });

      continue;
    }

    // The last type of token will be a `name` token. This is a sequence of
    // letters instead of numbers, that are the names of functions in our lisp
    // syntax.
    //
    //   (add 2 4)
    //    ^^^
    //    Name token
    //
    let LETTERS = /[a-z]/i;
    if (LETTERS.test(char)) {
      let value = '';

      // Again we're just going to loop through all the letters pushing them to
      // a value.
      while (LETTERS.test(char)) {
        value += char;
        char = input[++current];
      }

      // And pushing that value as a token with the type `name` and continuing.
      tokens.push({ type: 'name', value });

      continue;
    }

    // Finally if we have not matched a character by now, we're going to throw
    // an error and completely exit.
    throw new TypeError('I dont know what this character is: ' + char);
  }

  // Then at the end of our `tokenizer` we simply return the tokens array.
  
```

 ## 语法分析 *Syntactic Analysis* 

输入：token数组
输出；ast

walk递归函数：
  1. 遇到Number\String，直接返回对应JSON结构
  2. 遇到`(`,构建表达式，循环一直push，walk返回的结果，直到遇到`)`，返回表达式node

总结：
  后缀表达式的解析，相对简单

```javascript
  let current = 0;
  function walk() {
    let token = tokens[current];
    if (token.type === 'number') {
      current++;
      return {
        type: 'NumberLiteral',
        value: token.value
      }
    }
    if (token.type === 'string') {
      current++;
      return {
        type: 'StringLiteral',
        value: token.value
      }
    }
    if (token.type == 'paren' && token.value == '(') {
      token = tokens[++current];
      let node = {
        type: 'CallExpression',
        name: token.name,
        params:[]
      }
      token = tokens[++current];
      while (token.type !== 'paren' ||
            token.type === 'paren' && token.value !== ')') {
          node.params.push(walk());
          token = tokens[current];
      }
      ++current;
      return node;
    }
    throw TypeError(token.type)
  }
  let ast = {
    type: 'Program',
    body:[]
  }

  while(current < token.length()) {
    ast.body.push(walk());
  }

  return ast;

```


 
```javascript
{
     type: 'Program',
     body: [{
       type: 'CallExpression',
       name: 'add',
       params: [{
         type: 'NumberLiteral',
         value: '2',
       }, {
         type: 'CallExpression',
         name: 'subtract',
         params: [{
           type: 'NumberLiteral',
           value: '4',
         }, {
           type: 'NumberLiteral',
           value: '2',
         }]
       }]
     }]
   }
```

## 变换 Transformation

任何想对原始AST进行的处理，包括增加、删除、替换节点

## 遍历 Traversal

   1. Program - Starting at the top level of the AST
   2. CallExpression (add) - Moving to the first element of the Program's body
   3. NumberLiteral (2) - Moving to the first element of CallExpression's params
   4. CallExpression (subtract) - Moving to the second element of CallExpression's params
   5. NumberLiteral (4) - Moving to the first element of CallExpression's params
   6. NumberLiteral (2) - Moving to the second element of CallExpression's params

## 延伸扩展

1. 优先级如何处理
2. 语义如何处理



