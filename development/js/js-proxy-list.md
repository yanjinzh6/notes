---
title: js-proxy-list
date: 2021-01-31 15:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-proxy-list
mathjax: false
---

https://zhuanlan.zhihu.com/p/36035973

# 如何监听数组变化?

[! [狒狒神](https://pic4.zhimg.com/v2-b8504ba50420464a4bdd77af476bc73f_xs.jpg? source=172ae18b)](//www.zhihu.com/people/solojiang)

[狒狒神](//www.zhihu.com/people/solojiang) [​](https://www.zhihu.com/question/48510028)

阿里巴巴集团 前端工程师

2 人赞同了该文章

起源: 在 Vue 的数据绑定中会对一个对象属性的变化进行监听, 并且通过依赖收集做出相应的视图更新等等.

问题: 一个对象所有类型的属性变化都能被监听到吗?

之前用 `Object.defineProperty` 通过对象的 `getter/setter` 简单的实现了 [对象属性变化的监听](https://link.zhihu.com/? target= https%3A//github.com/SoloJiang/structure/blob/master/src/action/mvvm.js), 并且去通过依赖关系去做相应的依赖处理.

但是, 这是存在问题的, 尤其是当对象中某个属性的值是数组的时候. 正如 Vue 文档所说:

> 由于 JavaScript 的限制, Vue 无法检测到以下数组变动:

1.  当你使用索引直接设置一项时, 例如 `vm.items[indexOfItem] = newValue`
2.  当你修改数组长度时, 例如 `vm.items.length = newLength`



从 [Vue 源码](https://link.zhihu.com/? target= https%3A//github.com/vuejs/vue/blob/706c67d1d013577fdbfab258bca78557419cba7c/src/observe/array-augmentations.js) 中也可以看到确实是对数组做了特殊处理的. 原因就是 **ES5 及以下的版本无法做到对数组的完美继承** .

## 实验一下?

用之前写好的 `observe` 做了一个简单的实验, 如下:

```
import { observe } from './mvvm'
const data = {
  name: 'Jiang',
  userInfo: {
    gender: 0
  },
  list: []
}
// 此处直接使用了前面写好的 getter/setter observe(data)
data.name = 'Solo'
data.userInfo.gender = 1
data.list.push(1)
console.log(data)
```

结果是这样的:



<img src="https://pic1.zhimg.com/v2-19d79bcee4c0c0ad274da5e515b70c00\_b.jpg" data-caption="" data-size="normal" data-rawwidth="244" data-rawheight="315" class="content\_image" width="244"/>

! [](data: image/svg+ xml; utf8, <svg xmlns='http://www.w3.org/2000/svg' width='244' height='315'> </svg>)



从结果可以看出问题所在,`data` 中 name, userInfo, list 属性的值均发生了变化, 但是数组 list 的变化并没有被 `observe` 监听到. 原因是什么呢? 简单来说, 操作数组的方法, 也就是 `Array.prototype` 上挂载的方法并不能触发该属性的 setter, 因为这个属性并没有做赋值操作.

## 如何解决这个问题?

Vue 中解决这个问题的方法, 是将数组的常用方法进行重写, 通过包装之后的数组方法就能够去在调用的时候被监听到.

在这里, 我想的一种方法与它类似, 大概就是通过原型链去拦截对数组的操作, 从而实现对操作数组这个行为的监听.

实现如下:

```
// 让 arrExtend 先继承 Array 本身的所有属性 const arrExtend = Object.create(Array.prototype)
const arrMethods = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
/**
 * arrExtend 作为一个拦截对象, 对其中的方法进行重写 */
arrMethods.forEach(method => {
  const oldMethod = Array.prototype[method]
  const newMethod = function(...args) {
    oldMethod.apply(this, args)
    console.log(`${method} 方法被执行了 `)
  }
  arrExtend[method] = newMethod
})

export default {
  arrExtend
}
```

需要在 `defineReactive` 函数中添加的代码为:

```
if (Array.isArray(value)) {
    value.__proto__ = arrExtend
 }
```

测试一下:`data.list.push(1)`

我们看看结果:



<img src="https://pic4.zhimg.com/v2-1148ebd82820226916b91b4865ddc97b\_b.jpg" data-caption="" data-size="normal" data-rawwidth="298" data-rawheight="48" class="content\_image" width="298"/>

! [](data: image/svg+ xml; utf8, <svg xmlns='http://www.w3.org/2000/svg' width='298' height='48'> </svg>)



上面代码的逻辑一目了然, 也是 Vue 中实现思路的简化. 将 `arrExtend` 这个对象作为拦截器. 首先让这个对象继承 `Array` 本身的所有属性, 这样就不会影响到数组本身其他属性的使用, 后面对相应的函数进行改写, 也就是在原方法调用后去通知其它相关依赖这个属性发生了变化, 这点和 `Object.defineProperty` 中 `setter` 所做的事情几乎完全一样, 唯一的区别是可以细化到用户到底做的是哪一种操作, 以及数组的长度是否变化等等.

## 还有什么别的办法吗?

ES6 中我们看到了一个让人耳目一新的属性——`Proxy`. 我们先看一下概念:

> 通过调用 new Proxy() , 你可以创建一个代理用来替代另一个对象 (被称为目标), 这个代理对目标对象进行了虚拟, 因此该代理与该目标对象表面上可以被当作同一个对象来对待.
> 代理允许你拦截在目标对象上的底层操作, 而这原本是 JS 引擎的内部能力. 拦截行为使用了一个能够响应特定操作的函数 (被称为陷阱).

`Proxy` 顾名思义, 就是代理的意思, 这是一个能让我们随意玩弄对象的特性. 当我们, 通过 `Proxy` 去对一个对象进行代理之后, 我们将得到一个和被代理对象几乎完全一样的对象, 并且可以对这个对象进行完全的监控.

什么叫完全监控?`Proxy` 所带来的, 是对底层操作的拦截. 前面我们在实现对对象监听时使用了 `Object.defineProperty`, 这个其实是 JS 提供给我们的高级操作, 也就是通过底层封装之后暴露出来的方法.`Proxy` 的强大之处在于, 我们可以直接拦截对代理对象的底层操作. 这样我们相当于从一个对象的底层操作开始实现对它的监听.

改进一下我们的代码?

```
const createProxy = data => {
  if (typeof data === 'object' && data.toString() === '[object Object]') {
    for (let k in data) {
      if (typeof data[k] === 'object') {
        defineObjectReactive(data, k, data[k])
      } else {
        defineBasicReactive(data, k, data[k])
      }
    }
  }
}

function defineObjectReactive(obj, key, value) {
  // 递归   createProxy(value)
  obj[key] = new Proxy(value, {
    set(target, property, val, receiver) {
      if (property !== 'length') {
        console.log('Set %s to %o', property, val)
      }
      return Reflect.set(target, property, val, receiver)
    }
  })
}

function defineBasicReactive(obj, key, value) {
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: false,
    get() {
      return value
    },
    set(newValue) {
      if (value === newValue) return
      console.log(` 发现 ${key} 属性 ${value} -> ${newValue}`)
      value = newValue
    }
  })
}

export default {
  createProxy
}
```

对于一个对象中的基础类型的属性, 我们还是通过 `Object.defineProperty` 来实现响应式的属性, 因为这里并不存在痛点, 但是在实现对 `Object` 类型的属性进行监听的时候, 我采用的是创建代理, 因为我们之前的痛点在于无法去有效监听数组的变化. 当我们使用这种改进方法之后, 我们不用像之前通过重写数组的方法来实现对数组操作的监听了, 因为之前这种方法存在很多的局限性, 我们不能覆盖所有的数组操作, 同时, 我们也不能响应到类似于 `data.array.length = 0` 这种操作. 通过代理实现之后, 一切都不一样了. 我们可以从底层就实现对数组的变化进行监听. 甚至能 `watch` 到数组长度的变化等等各种更加细节的东西. 这无疑解决了很大的问题.

我们调用一下刚才的方法, 试试看?

```
let data = {
  name: 'Jiang',
  userInfo: {
    gender: 0,
    movies: []
  },
  list: []
}
createProxy(data)

data.name = 'Solo'
data.userInfo.gender = 0
data.userInfo.movies.push(' 星际穿越 ')
data.list.push(1)
```

输出为:



<img src="https://pic1.zhimg.com/v2-16e32e276fd8a4be5e002657435ffd58\_b.jpg" data-caption="" data-size="normal" data-rawwidth="520" data-rawheight="176" class="origin\_image zh-lightbox-thumb" width="520" data-original="https://pic1.zhimg.com/v2-16e32e276fd8a4be5e002657435ffd58\_r.jpg"/>

! [](data: image/svg+ xml; utf8, <svg xmlns='http://www.w3.org/2000/svg' width='520' height='176'> </svg>)



结果非常完美~我们实现了对对象所有属性变化的监听 `Proxy` 的骚操作还有很多很多, 比如说将代理当作原型放到原型链上, 这样一来就可以只对子类不含有的属性进行监听, 非常的强大.`Proxy` 可以得到更加广泛的应用, 而且场景很多. 这也是我第一次去使用, 还需要多加巩固 ( ;´Д｀)

编辑于 2018-04-24

[

JavaScript



](//www.zhihu.com/topic/19552521)

​赞同 2​

​2 条评论

​分享

​喜欢​收藏
