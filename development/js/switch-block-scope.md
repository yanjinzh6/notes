---
title: JavaScript switch 语句中的块级作用域
date: 2020-04-14 16:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: switch-block-scope
---

## switch 块级作用域

ES6 或 TS 引入了块级作用域, 通过 let 和 const, class 等可以定义块级作用域里的变量, 块级作用域内的变量不存在变量提升, 且存在暂时性死区. 常用的 if 和 for 语句都可以定义块级变量, switch 语句中的块级作用域是在整个 switch 语句中, 而不是对每个 case 都生成一个独立都块级作用域

```js
let number = 1;
switch(number){
  case 1:
    let name = 'Jony';
  default:
    console.log(name)
    // 输出 Jony
}
```

### 解决

如果需要每个 case 作用域单独出来, 可以如下处理

```js
let number = 1;
switch(number){
case 1:
{ let name = 'Jony';}
default:
console.log(name)
}
```

## 可能存在的问题

我们知道了 switch 语句, 整个 switch 语句的顶层是一个块级作用域, 但是还要注意 case 的特殊性, 在 case 中声明的变量, 并不会提升到块级作用域中.

```js
let number = 2;
switch(number){
  case 2:
    name = 'yu';
    // 相当于 window.name = 'yu'
    break;
}
```

```js
let number = 2;
switch(number){
  case 1:
    let name = 'jony';
    break;
  case 2:
    name = 'yu';
    // Uncaught ReferenceError: name is not defined
    break;
}
```

原因: case 里面定义的块级虽然不会存在变量提升, 但是会存在暂时性死区, 也就是说如果 `let name = 'jony'` 没有执行, 也就是 name 定义的过程没有执行, 那么 name 在整个块级作用域内都是不可用的, 都是 `undefined`

## 规范和检测

通常 es6 转换为 es5 后是不存在上面的问题的, 如果是服务端直接跑 es6 脚本的需要通过 `eslint` 规范来解决这种非法使用的问题

## 引用

- [聊聊 switch 语句中的块级作用域](https://github.com/forthealllight/blog/issues/44)
