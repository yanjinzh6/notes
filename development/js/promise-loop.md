---
title: JavaScript Promise 循环
date: 2020-04-10 11:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: promise-loop
---

多个 promise 一起执行可以使用 `Promise.all()` 方法, 可以保证执行顺序和返回结果顺序, 但是每个 promise 响应的时间是不一样的, 也就会导致需要串行的时候会比较随机执行每个 promise 的处理方法

类似要达到如下的效果, 使用 `Promise.all()` 会导致每个 promise 响应处理随机

```js
Promise.resolve().then(res => {
  // doSomething
  return promises[0]
}).then(res => {
  // doSomething
  return promises[1]
}).then(res => {
  // doSomething
  return promises[2]
})
```

使用 `arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])` 解决循环同步

`callback` 函数接收4个参数:

1. Accumulator (acc) (累计器)
1. Current Value (cur) (当前值)
1. Current Index (idx) (当前索引)
1. Source Array (src) (源数组)

将多个需要执行的 promise 组成数组

```js
[new Promise((re, rj) => re(1)), new Promise((re, rj) => re(2))].reduce((promise, value) => {
  return promise.then((res) => {
    console.log(res, value)
    return Promise.resolve(value)
  })
}, Promise.resolve('init'))
// init Promise {<resolved>: 1}
// 1 Promise {<resolved>: 2}
// Promise {<resolved>: 2}
```

使用 `async await` 一个循环就可以解决