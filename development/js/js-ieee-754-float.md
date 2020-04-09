---
title: JavaScript IEEE 754 浮点数运算
date: 2020-04-09 20:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-ieee-754-float
---

# 简介

在 JavaScript 中整数和浮点数都属于 `Number` 数据类型, 所有数字都是以 64 位浮点数形式储存, 即便整数也是如此. 当浮点数做数学运算的时候, 经常会发现一些问题

```js
// 加法 =====================
// 0.1 + 0.2 = 0.30000000000000004
// 0.7 + 0.1 = 0.7999999999999999
// 0.2 + 0.4 = 0.6000000000000001
// 2.22 + 0.1 = 2.3200000000000003
 
// 减法 =====================
// 1.5 - 1.2 = 0.30000000000000004
// 0.3 - 0.2 = 0.09999999999999998
 
// 乘法 =====================
// 19.9 * 100 = 1989.9999999999998
// 19.9 * 10 * 10 = 1990
// 1306377.64 * 100 = 130637763.99999999
// 1306377.64 * 10 * 10 = 130637763.99999999
// 0.7 * 180 = 125.99999999999999
// 9.7 * 100 = 969.9999999999999
// 39.7 * 100 = 3970.0000000000005
 
// 除法 =====================
// 0.3 / 0.1 = 2.9999999999999996
// 0.69 / 10 = 0.06899999999999999
```

# [IEEE 754 规范](https://zh.wikipedia.org/wiki/IEEE_754)

> IEEE 二进制浮点数算术标准 (IEEE 754) 是 20 世纪 80 年代以来最广泛使用的浮点数运算标准, 为许多 CPU 与浮点运算器所采用. 

JavaScript 里的数字是采用 IEEE 754 标准的 64 位双精度浮点数. 

双精度二进制小数, 使用 64 个比特存储. 

| 符号位 | 指数部分 (带偏移)                   | 尾数部分                     |
| ------ | ----------------------------------- | ---------------------------- |
| 1      | 11                                  | 52位长                       |
| S      | Exp                                 | Fraction                     |
| 63     | 62 至 52 偏正值 (实际的指数大小 +1023)  | 51 至 0 位编号 (从右边开始为 0)  |

符号位决定了一个数的正负, 指数部分决定了数值的大小, 小数部分决定了数值的精度.  IEEE 75 4规定, 有效数字第一位默认总是 1, 不保存在 64 位浮点数之中. 也就是说, 有效数字总是 `1.xx…xx` 的形式, 其中 `xx..xx` 的部分保存在 64 位浮点数之中, 最长可能为 52 位. 因此, JavaScript 提供的有效数字最长为 53 个二进制位 (64 位浮点的后 52 位 + 有效数字第一位的 1) . 

<!-- more -->

# 计算过程

- 将十进制转换为二进制
- 两者相加
- 将二进制转换为十进制

由于某些浮点数使用二进制表达是无穷的, 所以在第一和第三步骤都会出现误差的情况, 例如计算 `0.1 + 0.2`

首先, `0.1` 和 `0.2` 将分别转换成二进制

```
0.1 = 0.0001100110011001...(无限)
0.2 = 0.0011001100110011...(无限)
```

IEEE 754 标准的 64 位双精度浮点数的小数部分最多支持 53 位二进制位, 所以两者相加之后得到二进制为: 

```
0.0100110011001100110011001100110011001100110011001100
```

因浮点数小数位的限制而截断的二进制数字, 再转换为十进制, 就成了 `0.30000000000000004`. 所以在进行算术计算时会产生误差. 

# 整数精度问题

在 JavaScript 中 `Number` 类型统一按浮点数处理, 整数是按最大 54 位来算最大 `( 253 - 1, Number.MAX_SAFE_INTEGER, 9007199254740991)` 和最小 `(-(253 - 1), Number.MIN_SAFE_INTEGER, -9007199254740991)` 安全整数范围的. 所以只要超过这个范围, 就会存在被舍去的精度问题. 

例如

```js
console.log(19571992547450991); //=> 19571992547450990
console.log(19571992547450991===19571992547450992); //=> true
```

当然这个问题并不只是在 Javascript 中才会出现, 几乎所有的编程语言都采用了 IEEE-745 浮点数表示法, 任何使用二进制浮点数的编程语言都会有这个问题, 只不过在很多其他语言中已经封装好了方法来避免精度的问题, 而 JavaScript 是一门弱类型的语言, 从设计思想上就没有对浮点数有个严格的数据类型, 所以精度误差的问题就显得格外突出. 

# 解决方法

## 类库

通常这种对精度要求高的计算都应该交给后端去计算和存储, 因为后端有成熟的库来解决这种计算问题. 前端也有几个不错的类库: 

### Math.js

Math.js 是专门为 JavaScript 和 Node.js 提供的一个广泛的数学库. 它具有灵活的表达式解析器, 支持符号计算, 配有大量内置函数和常量, 并提供集成解决方案来处理不同的数据类型, 像数字, 大数字(超出安全数的数字), 复数, 分数, 单位和矩阵. 功能强大, 易于使用. 

官网: http://mathjs.org/

GitHub: https://github.com/josdejong/mathjs

### decimal.js

为 JavaScript 提供十进制类型的任意精度数值. 

官网: http://mikemcl.github.io/decimal.js/

GitHub: https://github.com/MikeMcl/decimal.js

### big.js

官网: http://mikemcl.github.io/big.js

GitHub: https://github.com/MikeMcl/big.js/

这几个类库帮我们解决很多这类问题, 不过通常我们前端做这类运算通常只用于表现层, 应用并不是很多. 所以很多时候, 一个函数能解决的问题不需要引用一个类库来解决. 

## toFixed() 方法

通过使用 `toFixed(digits)` 方法来截取小数点后的位数, 参数 `digits` 表示小数点后数字的个数；介于 0 到 20  (包括) 之间, 实现环境可能支持更大范围. 如果忽略该参数, 则默认为 0. 该数值在必要时进行四舍五入, 另外在必要时会用 0 来填充小数部分, 以便小数部分有指定的位数.  如果数值大于 1e+21, 该方法会简单调用 Number.prototype.toString()并返回一个指数记数法格式的字符串. 

## 使用字符串

当需要计算和表示的是已经超大数字或者超长的浮点数时, 这时候通过字符串截取和拼接进行计算

# 简单使用字符串方法计算浮点数

```js
/**
 ** 加法函数, 用来得到精确的加法结果
 ** 说明: javascript 的加法结果会有误差, 在两个浮点数相加的时候会比较明显. 这个函数返回较为精确的加法结果.
 ** 调用: accAdd(arg1,arg2)
 ** 返回值: arg1 加上 arg2 的精确结果
 **/
function accAdd (arg1, arg2) {
  var r1, r2, m, c
  try {
    r1 = arg1.toString().split(".")[1].length
  }
  catch (e) {
    console.log(e)
    r1 = 0
  }
  try {
    r2 = arg2.toString().split(".")[1].length
  }
  catch (e) {
    console.log(e)
    r2 = 0
  }
  c = Math.abs(r1 - r2)
  m = Math.pow(10, Math.max(r1, r2))
  if (c > 0) {
    var cm = Math.pow(10, c)
    if (r1 > r2) {
      arg1 = Number(arg1.toString().replace(".", ""))
      arg2 = Number(arg2.toString().replace(".", "")) * cm
    } else {
      arg1 = Number(arg1.toString().replace(".", "")) * cm
      arg2 = Number(arg2.toString().replace(".", ""))
    }
  } else {
    arg1 = Number(arg1.toString().replace(".", ""))
    arg2 = Number(arg2.toString().replace(".", ""))
  }
  return (arg1 + arg2) / m
}

// 给 Number 类型增加一个 add 方法, 调用起来更加方便.
Number.prototype.add = function (arg) {
  return accAdd(this, arg)
}

/**
 ** 减法函数, 用来得到精确的减法结果
 ** 说明: javascript 的减法结果会有误差, 在两个浮点数相减的时候会比较明显. 这个函数返回较为精确的减法结果.
 ** 调用: accSub(arg1,arg2)
 ** 返回值: arg1 加上 arg2 的精确结果
 **/
function accSub (arg1, arg2) {
  var r1, r2, m, n
  try {
    r1 = arg1.toString().split(".")[1].length
  }
  catch (e) {
    console.log(e)
    r1 = 0
  }
  try {
    r2 = arg2.toString().split(".")[1].length
  }
  catch (e) {
    console.log(e)
    r2 = 0
  }
  m = Math.pow(10, Math.max(r1, r2)) //last modify by deeka //动态控制精度长度
  n = (r1 >= r2) ? r1 : r2
  return Number(((arg1 * m - arg2 * m) / m).toFixed(n))
}

// 给 Number 类型增加一个 mul 方法, 调用起来更加方便.
Number.prototype.sub = function (arg) {
  return accSub(this, arg)
}

/**
 ** 乘法函数, 用来得到精确的乘法结果
 ** 说明: javascript 的乘法结果会有误差, 在两个浮点数相乘的时候会比较明显. 这个函数返回较为精确的乘法结果.
 ** 调用: accMul(arg1,arg2)
 ** 返回值: arg1 乘以 arg2 的精确结果
 **/
function accMul (arg1, arg2) {
  var m = 0, s1 = arg1.toString(), s2 = arg2.toString()
  try {
    m += s1.split(".")[1].length
  }
  catch (e) {
    console.log(e)
  }
  try {
    m += s2.split(".")[1].length
  }
  catch (e) {
    console.log(e)
  }
  return Number(s1.replace(".", "")) * Number(s2.replace(".", "")) / Math.pow(10, m)
}

// 给 Number 类型增加一个 mul 方法, 调用起来更加方便.
Number.prototype.mul = function (arg) {
  return accMul(this, arg)
}

/**
 ** 除法函数, 用来得到精确的除法结果
 ** 说明: javascript 的除法结果会有误差, 在两个浮点数相除的时候会比较明显. 这个函数返回较为精确的除法结果.
 ** 调用: accDiv(arg1,arg2)
 ** 返回值: arg1 除以 arg2 的精确结果
 **/
function accDiv (arg1, arg2) {
  var t1 = 0, t2 = 0, r1, r2
  try {
    t1 = arg1.toString().split(".")[1].length
  }
  catch (e) {
    console.log(e)
  }
  try {
    t2 = arg2.toString().split(".")[1].length
  }
  catch (e) {
    console.log(e)
  }
  r1 = Number(arg1.toString().replace(".", ""))
  r2 = Number(arg2.toString().replace(".", ""))
  return (r1 / r2) * Math.pow(10, t2 - t1)
}

// 给 Number 类型增加一个 div 方法, 调用起来更加方便.
Number.prototype.div = function (arg) {
  return accDiv(this, arg)
}
```

# 参考

- [JavaScript 浮点数运算的精度问题](https://www.html.cn/archives/7340)