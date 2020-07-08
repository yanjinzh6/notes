---
title: JavaScript 中的 this
date: 2020-07-04 11:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-this
---

## 简介

js 中的 this 关键字强大灵活, 指向多变, 如果能熟练掌握, 就可以写出更简洁优雅的代码, 否则, 乱用 this 可能导致许多潜藏的 bug, this 主要了解的是其指向性, 简单的概括, 谁调用了, this 就指向谁, 也就是说, this 的指向是在调用的时候确定的, 调用函数会创建新的属于函数自身的执行上下文, 执行上下文的调用创建阶段会决定 this 的指向

具体调用的规则如下:

- 在函数体中, 简单调用该函数时 (非显示 / 隐式绑定下), 严格模式下 this 绑定到 undefined, 否则绑定到全局对象 window / global 上
- 一般构造函数 new 调用时绑定到新创建的对象上
- 一般由 call / apply / bind 方法显式调用时绑定到指定参数的对象上
- 一般由上下文对象调用时绑定在该对象上
- 箭头函数中, 根据外层上下文绑定的 this 决定 this 指向

<!-- more -->

## 全局环境调用

在浏览器的全局环境中简单调用的, this 默认指向 window, 在 `use strict` 严格模式下 this 为 undefined

```js
function foo() {
  console.log(this)
}
foo()
// Window {parent: Window, opener: null, top: Window, length: 0, frames: Window, …}
function foo2() {
  'use strict'
  console.log(this)
}
foo2()
// undefined
```

另外通过赋值到全局环境中的调用也同上面一样

```js
bar = 2
const foo = {
  bar: 1,
  fn: function() {
    console.log(this)
    console.log(this.bar)
  }
}
const foo2 = foo.fn
foo2()
// 等于执行了 console.log(window) 和 console.log(window.bar)
// Window {parent: Window, opener: null, top: Window, length: 0, frames: Window, …}
// 2
foo.fn()
// 这里 this 指向的是最后调用的对象
// {bar: 1, fn: ƒ}
// 1
```

所以在执行函数时, 如果函数中的 this 是被上一级的对象所调用, 那么 this 指向该对象, 否则指向全局环境或 undefined

## 上下文对象调用

```js
const foo = {
  name: 'foo',
  fn: function() {
    return this
  }
}
console.log(foo.fn( === foo))
// true
```

```js
const foo = {
  name: 'foo',
  bar: {
    name: 'bar',
    fn: function() {
      return this.name
    }
  }
}
console.log(foo.bar.fn())
// 由于 this 指向最后调用的对象, 所以输出 bar
```

```js
const foo = {
  name: 'foo',
  fn: function() {
    return this.name
  }
}
console.log(foo.fn())
// foo
const foo2 = {
  name: 'foo2',
  fn: function() {
    return foo.fn()
  }
}
console.log(foo2.fn())
// this 指向最后调用的对象, 所以是 foo.name
// foo
const foo3 = {
  name: 'foo3',
  fn: function() {
    const fn = foo.fn
    return fn()
  }
}
console.log(foo3.fn())
// this 指向 window
// ''
```

修改 `foo2.fn = foo.fn` 赋值, 即可以在调用 `foo2.fn()` 时 this 指向 `foo2` 对象

## bind/call/apply

三者都会改变相关函数 this 指向, 具体的区别有

- call/apply 直接进行相关函数调用, bind 不会执行相关函数, 而是返回一个新的函数, 这个新的函数已经自动绑定了新的 this 指向
- call 于 apply 主要区别是参数设定上

```js
const target = {}
fn.call(target, 'arg1', 'arg2')
```

相当于

```js
const target = {}
fn.apply(target, ['arg1', 'arg2'])
```

相当于

```js
const target = {}
fn.bind(target, 'arg1', 'arg2')()
```

```js
const foo = {
  name: 'foo',
  fn: function() {
    console.log(this.name)
  }
}
const bar = {
  name: 'bar'
}
console.log(foo.fn.call(bar))
// bar
```

## 构造函数

new 操作符调用构造函数流程

- 创建一个新的对象
- 将构造函数的 this 指向这个新对象
- 为对象添加属性和方法
- 返回新对象

对应的代码流程

```js
var obj = {}
obj.__proto__ = Foo.prototype
Foo.call(obj)
```

```js
function Foo() {
  this.name = 'foo'
}
const foo = new Foo()
console.log(foo.name)
// foo
```

如果构造函数中显示 return 的话, 需要注意如果返回的是对象, 则 this 指向返回的对象, 如果返回的不是对象, 则 this 指向实例

```js
function Foo() {
  this.name = 'foo'
  const o = {}
  return o
}
const foo = new Foo()
// foo 是返回的空对象
console.log(foo.name)
// undefined
```

```js
function Foo() {
  this.name = 'foo'
  return 1
}
const foo = new Foo()
// foo 是返回的目标对象实例 this
console.log(foo.name)
// foo
```

## 箭头函数

箭头函数使用 this 是根据外层 (函数或者全局) 的上下文来决定的

```js
const foo = {
  fn: function() {
    setTimeout(function() {
      console.log(this)
    })
  }
}
console.log(foo.fn())
// this 是在 setTimeout 中的匿名函数里, 因此 this 指向 window 对象
// Window {parent: Window, opener: null, top: Window, length: 0, frames: Window, …}
```

```js
const foo = {
  fn: function() {
    setTimeout(() => {
      console.log(this)
    })
  }
}
console.log(foo.fn())
// this 指向当前对象
// {fn: f}
```

## this 优先级

通过 bind/call/apply 和 new 对 this 绑定的称为显示绑定, 根据调用关系确定的 this 指向称为隐式绑定

```js
function foo(a) {
  console.log(this.a)
}
const bar = {
  a: 1,
  foo: foo
}
const bar2 = {
  a: 2,
  foo: foo
}
bar.foo.call(bar2)
bar2.foo.call(bar)
// call/apply 的显式绑定优先级更高
// 2
// 1
```

```js
function foo(a) {
  this.a = a
}
const obj = {}
const bar = foo.bind(obj)
bar(2)
console.log(obj.a)
// 使用 bind 将 bar 函数中的 this 绑定为 obj 对象, obj.a = 2
// 2
const baz = new bar(3)
// bar 函数本身通过 bind 方法构造的函数, 内部已经将 this 绑定为 obj, 作为构造函数时, 通过 new 调用, 返回的实例已经与 obj 解绑
// new 绑定修改了 bind 绑定中的 this, 所以 new 优先级比显式 bind 绑定更高
console.log(baz.a)
// 3
```

```js
function foo() {
  return a => {
    console.log(this.a)
  }
}
const bar = {
  a: 1
}
const bar2 = {
  a: 2
}
const foo2 = foo.call(bar)
console.log(foo2.call(bar2))
// foo 的 this 绑定到 bar, bar (引用箭头函数) 的 this 也会绑定到 bar, 箭头函数的绑定无法被修改
// 1
```

如果完全使用箭头函数的话, 那绑定一开始就已经确定了, 无法再修改

```js
var a = 123
const foo = () => a => {
  // 一开始 this 绑定到全局 window
  console.log(this.a)
}
const bar = {
  a: 1
}
const bar2 = {
  a: 2
}
const foo2 = foo.call(bar)
console.log(foo2.call(bar2))
// 123
// undefined
```

如果将 `var a = 123` 修改为 `const a = 123`, 将会输出 undefined, 因为使用 const 声明的变量不会挂载到全局对象 window 中, 因此找不到 `window.a`
