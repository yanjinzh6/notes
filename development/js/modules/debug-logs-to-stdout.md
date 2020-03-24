---
title: Debug 模块输出到 stdout
date: 2020-03-24 21:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript', 'debug']
permalink: debug-logs-to-stdout
---

`debug` 模块默认输出到 `stderr`, 在使用 `pm2` 的时候很不友好, 所以需要实现一下绑定到 `stdout` 中

```js
const logFactory = require('debug')

module.exports = (namespace) => {
  const log = logFactory(`${namespace}:log`)
  log.log = console.log.bind(console)
  const debug = logFactory(`${namespace}:debug`)
  debug.log = console.debug.bind(console)
  const error = logFactory(`${namespace}:error`)
  return {
    log,
    debug,
    error
  }
}

const log = require('./log')('namespace')
log.log('this is log')
// namespace:log this is log
log.debug('this is debug')
// namespace:debug this is debug
log.error('this is error')
// namespace:error this is error
// log 和 debug 输出到 stdout, error 输出到 stderr
```

[参考](https://github.com/visionmedia/debug/issues/503)
