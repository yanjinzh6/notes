---
title: JavaScript 标准内置对象 Promise
date: 2020-04-07 17:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript', 'global']
permalink: Promise-object
---

## 简介

[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象用于表示一个异步操作的最终完成 (或失败), 及其结果值.

```js
// 官方示例
const promise1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve('foo');
  }, 300);
});

promise1.then(function(value) {
  console.log(value);
  // expected output: "foo"
});

console.log(promise1);
// expected output: [object Promise]
```

<!-- more -->

## 语法

```js
/**
 *
 */
new Promise( function(resolve, reject) {...} /* executor */  );
```

- `executor`: 带有 `resolve` 和 `reject` 两个参数的函数. `Promise` 构造函数执行时立即调用 `executor` 函数, `resolve` 和 `reject` 两个函数作为参数传递给 `executor` (`executor` 函数在 `Promise` 构造函数返回所建 `promise` 实例对象前被调用) . `resolve` 和 `reject` 函数被调用时, 分别将 `promise` 的状态改为 `fulfilled` (完成) 或 `rejected` (失败) .  `executor` 内部通常会执行一些异步操作, 一旦异步操作执行完毕 (可能成功/失败), 要么调用 `resolve` 函数来将 `promise` 状态改成 `fulfilled`, 要么调用 `reject` 函数将 `promise` 的状态改为 `rejected`. 如果在 `executor` 函数中抛出一个错误, 那么该 `promise` 状态为 `rejected`. `executor` 函数的返回值被忽略.

## 方法

- [Promise.all(iterable)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
  - 处理多个 promise 对象, 返回一个新的 `Promise` 对象
  - 该对象会顺序执行参数列表
  - 仅当 iterable 参数对象里所有的 promise 都成功时才会成功
  - 其中任何一个 promise 失败则立即失败
  - 执行时间根据不同的 promise 响应时间不一样
  - 执行结果会一一对应到返回值里, 顺序与 iterable 参数的顺序保持一致
  - 当 promise 失败时返回的是触发失败的 promise 的错误信息
  - 如果传入的参数是一个空的可迭代对象, 则返回一个已完成 (already resolved) 状态的 `Promise`
  - 如果传入的参数不包含任何 promise, 则返回一个异步完成 (asynchronously resolved   ) `Promise`. 注意: Google Chrome 58 在这种情况下返回一个已完成 (already resolved) 状态的 `Promise`.
- [Promise.race(iterable)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
  - 处理多个 promise 对象, 返回一个新的 `Promise` 对象
  - 其中任何一个 promise 成功或失败后都会马上返回
  - 如果传的迭代是空的, 则返回的 promise 将永远等待
  - 如果参数包含一个或多个非 `Promise` 对象和/或已解决/拒绝的 `Promise`, 则返回第一个值
- [Promise.reject(reason)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)
  - 返回一个状态为失败的 `Promise` 对象, 并将给定的失败信息传递给对应的处理方法
- [Promise.resolve(value)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)
  - 将给定的 value 返回为一个 `Promise` 对象
  - 如果这个值是一个 promise, 那么将返回这个 promise
  - 如果这个值是 thenable (即带有"then" 方法), 返回的 promise 会 "跟随" 这个 thenable 的对象, 采用它的最终状态, 否则返回的 promise 将以此值完成
  - 此函数将类 promise 对象的多层嵌套展平

```js
// thenable 例子
// Resolve一个thenable对象
var p1 = Promise.resolve({
  then: function(onFulfill, onReject) { onFulfill("fulfilled!"); }
});
console.log(p1 instanceof Promise) // true, 这是一个Promise对象

p1.then(function(v) {
    console.log(v); // 输出"fulfilled!"
  }, function(e) {
    // 不会被调用
});

// Thenable在callback之前抛出异常
// Promise rejects
var thenable = { then: function(resolve) {
  throw new TypeError("Throwing");
  resolve("Resolving");
}};

var p2 = Promise.resolve(thenable);
p2.then(function(v) {
  // 不会被调用
}, function(e) {
  console.log(e); // TypeError: Throwing
});

// Thenable在callback之后抛出异常
// Promise resolves
var thenable = { then: function(resolve) {
  resolve("Resolving");
  throw new TypeError("Throwing");
}};

var p3 = Promise.resolve(thenable);
p3.then(function(v) {
  console.log(v); // 输出"Resolving"
}, function(e) {
  // 不会被调用
});
```

## 原型

### 属性

`Promise.prototype.constructor`: 返回被创建的实例函数.  默认为 `Promise` 函数.

### 原型方法

- [Promise.prototype.catch(onRejected)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)
  - `then` 方法 `onRejected` 的语法糖
  - 用于 promise 中的错误处理, 当前面的 promise 出现错误或者主动调用了 `onRejected` 方法时, 会在这里处理
  - 返回一个 `Promise` 对象
- [Promise.prototype.then(onFulfilled, onRejected)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)
  - `onFulfilled` 已解决
  - `onRejected` 已拒绝
  - 返回一个 `Promise` 对象
- [Promise.prototype.finally(onFinally)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally)
  - 无论当前 promise 状态是完成或者失败都会调用
  - 返回一个 `Promise` 对象
