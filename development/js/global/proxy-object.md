---
title: JavaScript 标准内置对象 Proxy
date: 2020-03-25 14:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript', 'global']
permalink: proxy-object
---

# 简介

[Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 对象用于定义基本操作的自定义行为 (如属性查找, 赋值, 枚举, 函数调用等)

这里通过例子简单说明 `Proxy` 的简单使用和实际应用

<!-- more -->

# 语法

```js
let p = new Proxy(target, handler)
```

## 参数

- `target`: 用 `Proxy` 包装的目标对象 (可以是任何类型的对象, 包括原生数组, 函数, 甚至另一个代理)
- `handler`: 一个对象, 其属性是当执行一个操作时定义代理的行为的函数

# 示例

对于一些旧模块, 经常会存在一大堆有规律的回调方法

```js
const oldModule = {
  func: function({ isSuccess, success, error }) {
    if (isSuccess) {
      success({
        message: 'func success'
      })
    } else {
      error({
        message: 'func error'
      })
    }
  },
  func2: function({ isSuccess, success, error }) {
    if (isSuccess) {
      success({
        message: 'func2 success'
      })
    } else {
      error({
        message: 'func2 error'
      })
    }
  }
}
```

每次回调只要传入相应的回调方法就可以了, 但是这样很容易导致回调地狱的出现

```js
oldModule.func({ isSuccess: true, success: data => log.log(data), error: err => log.error(err) })
oldModule.func2({ isSuccess: false, success: data => log.log(data), error: err => log.error(err) })
```

## 简单 Promise

最简单的就是通过返回一个 `Promise` 对象即可

```js
const oldModulePromisify = (name, options) => {
  return new Promise((resolve, reject) => {
    oldModule[name]({
      ...options,
      success: resolve,
      error: reject
    })
  })
}

const proxyTest = async () => {
  let res = await oldModulePromisify('func', { isSuccess: true })
  log.log(res)
  try {
    res = await oldModulePromisify('func2', { isSuccess: false })
    log.log(res)
  } catch (e) {
    log.error(e)
  }
}

proxyTest()
```

这样虽然可以实现, 但基本上是一个个方法去改造, 失去了美观和 IDE 支持

## 通过代理

Proxy 的 [handler](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler) 对象是一个占位符对象, 它包含了用于 Proxy 的陷阱 (Trap) 函数, 这里使用 `get` 方法, 在读取代理对象的某个属性时触, 可以获取到被代理的对象和当前属性名, 其他方法可以通过官方文档了解更多方法

```js
const promisify = (target, name) => (options) => {
  return new Promise((resolve, reject) => {
    target[name]({
      ...options,
      success: resolve,
      error: reject
    })
  })
}

let handler = {
  get: function(target, name) {
    log.debug(target, name)
    return promisify(target, name)
  }
}

const oldModuleProxy = new Proxy(oldModule, handler)

const proxyTest = async () => {
  let res = await oldModuleProxy.func({ isSuccess: true })
  log.log(res)
  try {
    res = await oldModuleProxy.func2({ isSuccess: false })
    log.log(res)
  } catch (e) {
    log.error(e)
  }
}

proxyTest()
```

通过创建一个代理对象, 就可以使用相应的方法, 可以通过 jsDoc, `index.td` 等方式提供 vsCode 的智能提醒
