---
title: JavaScript Set 的交集, 并集, 差集
date: 2020-04-09 11:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-set-cal
---

> Set 是 es6 新增加的对象, 对于基础的数据类型可以很容易的利用其特点计算交集, 并集, 差集

```js
let setA = new Set([1, 3, 5, 7, 9])
let setB = new Set([4, 5, 6, 7])
// 并集
let unionSet = new Set([...setA, ...setB])
// [1, 3, 4, 5, 6, 7, 9]

// 交集
let intersectionSet = new Set([...setA].filter(e => setB.has(e)))
// [5, 7]

// 差集
let differenceSet = new Set([...[...setA].filter(e => !setB.has(e)), ...[...setB].filter(e => !setA.has(e))])
// [1, 3, 4, 6, 9]
```

由于 js 中对象的相等判断只能是应用的, 例如

```js
let set = new Set()
let obj = { a: 1, b: 2 }
set.add({ a: 1, b: 2 })
// Set(1) {{…}}
set.has({ a: 1, b: 2 })
// false
set.has(obj)
// false
set.add(obj)
// Set(2) {{…}, {…}}
set.has(obj)
// true
```

只有对引用的对象使用 `has` 方法才会返回 `true`, 这种情况只能将对象 `JSON.stringify()`, 但是对象可能会因为键值的排序而导致转换的字符串不一样, 这里在原型中添加一个方法, 将对象按照键值的固定排序转换成字符串, 但是转换字符串本来就是很消耗计算的方法

```js
Object.prototype.stringifySorted = function() {
  let oldObj = this
  let obj = oldObj.length || oldObj.length === 0 ? [] : {}
  for (let key of Object.keys(this).sort((a, b) => a.localeCompare(b))) {
    let type = typeof oldObj[key]
    if (type === "object") {
      obj[key] = oldObj[key].stringifySorted()
    } else {
      obj[key] = oldObj[key]
    }
  }
  return JSON.stringify(obj)
}

let set = new Set()
set.add({ a: 1, b: 2 }.stringifySorted())

console.log(
  `Set contains {b:2, a:1}: ${set.has({ b: 2, a: 1 }.stringifySorted())}`
)
```

参考

- [How to customize object equality for JavaScript Set](https://stackoverflow.com/questions/29759480/how-to-customize-object-equality-for-javascript-set)
- [知乎](https://www.zhihu.com/question/19863166)