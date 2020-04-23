---
title: 抛出 UnhandledPromiseRejectionWarning 异常问题
date: 2020-03-24 17:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: unhandled-promise-rejection-warning
---

一个 `Promise` 是一个异步操作的状态机, 其可能处于这三种状态之一

- `pending`: 异步操作还在执行中
- `fulfilled`: 异步操作已经完成
- `rejected`: 异步操作执行失败

在 Node.js 6.6.0 中增加了一个特性: 对 `Promise` 中未处理的 `rejection` 默认会输出 `UnhandledPromiseRejectionWarning` 提示

<!-- more -->

```js
const test = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject('reject'), 500)
  })
}
await test()
```

运行时会出现错误

```sh
node test.js
(node:47122) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): error
(node:47122) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code
```

原因是在 `Promise` 中主动 `reject` 或者 `throw new Error('exception!')`, 如果没有主动 `.catch()` 的, 就会提示错误, 当 `.catch()` 中出现错误没有继续 `.catch()` 的情况下也会报错

```js
await test().catch(err => log.error(err))
await test().catch(err => log.error(e.message)).catch(err => log.error(err))
```

使用 `try catch` 也可以比较方便的处理 `Promise` 中的异常

```js
try {
  await test()
} catch (e) {
  log.error(err)
}
```

可以通过捕获 `unhandledRejection` 事件来处理 `UnhandledPromiseRejectionWarning` 异常, 这样会把所有未主动处理的 `UnhandledPromiseRejectionWarning` 异常

```js
process.on('unhandledRejection', error => {
  log.error(error)
  process.exit(1)
})
```
