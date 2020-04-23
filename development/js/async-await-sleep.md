---
title: Async await 实现 sleep
date: 2020-03-11 21:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: async-await-sleep
---

有时候经常会用到一个暂停的方法, 随着 nodejs 版本从 v7 以后,已经可以很方便的用原生的 `async await` 来实现了

```js
const sleep = (timeout = 3000) => {
  console.log('休眠', timeout, 'ms')
  return new Promise((resolve, reject) => setTimeout(() => {
    resolve()
  }, timeout))
}

(async () => {
  for (let i = 0; i < 10; i ++) {
    console.log(i)
    await sleep()
  }
})
```

比使用 `Promise` 又简洁了一番
