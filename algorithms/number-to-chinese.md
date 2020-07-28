---
title: 数字转中文
date: 2020-07-25 12:00:00
tags: '算法'
categories:
  - ['算法', '字符串']
permalink: number-to-chinese
photo:
mathjax: true
---

## 简介

现在的各种报表页面都有需求将自动统计的数字转换为中文, 这里简单介绍中文数字的特点以及转换的相应算法#

## 中文数字特点

中文数字采用十进制, 用汉字 "零一二三四五六七八九" 表示基本记数, 用 "数字 + 权位" 的方式组成数字, 比如阿拉伯数字 `100`, 用中文计数就是 "一百" , 其中 "一" 是数字,  "百" 是数字的权位, 中文常用的数字权位有 "分"  "角"  "十"  "百"  "千"  "万"  "亿" 等, 对于更大的单位, 用 "万亿" 、 "百万亿" 、 "千万亿" 、 "亿亿" 来堆叠计数

对应如下

| 中文数字 | 阿拉伯数字 |
| -- | -- |
| 分 | 0.01 |
| 角 | 0.1 |
| 十 | 10 |
| 百 | 100 |
| 千 | 1000 |
| 万 | 10000 |
| 亿 | 100000000 |

<!-- more -->

### 权位

权位是这个数字的量值, 相当于阿拉伯数字中的数位, 只有最低位数是没有权位的, 例如个位数, 万位数, **万位数之所以有一万这样的转换, 需要注意这里的万是小节中的计量单位**

例如: 中文数字 "一千二百三十" , 数字 "一" 的权位是 "千" , 对应的阿拉伯数字的值是 $1×1000$, 数字 "二" 的权位是 "百" , 对应的阿拉伯数字的值是 $2×100$, 数字 "三" 的权位是 "十" , 对应的阿拉伯数字的值是 $3×10$, 整个数字的值就是 $1×1000 + 2×100 + 3×10 = 1230$

### 小节

中文计数以 "万" 为小节 (欧美的计数习惯是以 "千" 为小节) , 每一个小节都有一个节权位, 万以下的节没有节权位 (或节权位是空) , 万以上的节权位就是万, 再大就是亿 (即万的一万倍) , 每个小节内部以 "十百千" 为权位独立计数,  "十百千" 这几个权位是不能连续出现的, 比如 "二十百" 和 "一千千" 都是非法的中文数字, 但是 "万" 和 "亿" 作为节权位时可以和其他权位连在一起使用, 比如 "二十亿" 和 "五千万" 都是合法的中文数字

### 零

阿拉伯数字中数字的权位依靠数字在整个数字长度中的偏移位置确定, 因此数字中间出现的 `0` 用于标记数字的偏移位置, 即便是连续出现的 `0` 也不能省略, 中文计数方式中每个数字的权位都直接跟在数字后面, 因此可以用一个 "零" 代表连续出现的若干个 `0`

中文数字对 "零" 的使用总结起来有以下三条规则

- 规则 1: 以 `10000` 为小节, 小节的结尾即使是 `0`, 也不使用 "零"
- 规则 2: 小节内两个非 `0` 数字之间要使用 "零"
- 规则 3: 当小节的 "千" 位是 `0` 时, 若本小节的前一小节无其他数字, 则不用 "零" , 否则就要用 "零"

## 转换算法

### 数据结构设计

- 使用数组代表单个数字的转换, 因为阿拉伯数字 `0～9` 与相应的中文数字是一一对应的
- 使用数组代表权位
  - 数组的 `0` 号位置为空, 对应上面所讲到的最低位数没有权位
  - 数字中每个小节中的位移对应着相应的权位
- 使用数组代表小节
  - 数组的 `0` 号位置为空, 对应上面所讲到的万以下的节没有节权位
  - 对于 32 位正数能表达的最大数来说, 最大节权是 "万亿" 已经足够了, 如果要转换更大的数, 可以延伸这个节权表的定义, 比如增加 "亿亿"
  - 数字中每满 4 个位移即累计一个小节数组的位移
- 使用数组表示小数

### 零的处理

对每个小节进行处理

- 连续多个零字只保留一个
- 如果这一小节只有零字则去掉这一小节
- 如果结尾为 "零" 则去掉零字

### 小数的处理

现实生活中计算金额的小数都是通过 ["银行家算法"](https://zh.wikipedia.org/wiki/%E9%93%B6%E8%A1%8C%E5%AE%B6%E7%AE%97%E6%B3%95) 取前两位数, 即 "四舍六入五取偶", 这样只需要考虑到 "分"  "角" 即可

如果有存在 `0` 的情况, 直接返回空字符串, 其他的根据位移从数组获取

当没有小数时, 可以加上 "元整" 的标识

### 负数的处理

负数需要将第一位置转换为某个标识即可

## 实现

```js
/**
 * @description 数字转中文数码
 *
 * @param {Number|String}   number  数字, 包含负数和小数
 * @param {String}          type    文本类型, lower|upper, 默认 upper
 *
 */
export default (number, {type = 'upper', unitName = '元'} = {}) => {
  if (typeof number === 'undefined') {
    // 不存在
    return '零'
  }
  // 配置
  const confs = {
    lower: {
      num: ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九'],
      unit: ['', '十', '百', '千'],
      level: ['', '万', '亿']
    },
    upper: {
      num: ['零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖'],
      unit: ['', '拾', '佰', '仟'],
      level: ['', '万', '亿']
    },
    decimal: {
      unit: ['分', '角']
    },
    maxNumber: 999999999999.99
  }

  // 过滤不合法参数
  if (Number(number) > confs.maxNumber) {
    console.error(
      `The maxNumber is ${confs.maxNumber}. ${number} is bigger than it!`
    )
    return false
  }

  const conf = confs[type]
  const numbers = String(Number(number).toFixed(2)).split('.')
  const integer = numbers[0].split('')
  const decimal = (typeof numbers[1] === 'undefined' || Number(numbers[1]) === 0) ? [] : numbers[1].split('')

  // 四位分级
  const levels = integer.reverse().reduce((pre, item, idx) => {
    let level = pre[0] && pre[0].length < 4 ? pre[0] : []
    let value =
      item === '-' ? '[负]' : item === '0' ? conf.num[item] : conf.num[item] + conf.unit[idx % 4]
    level.unshift(value)

    if (level.length === 1) {
      pre.unshift(level)
    } else {
      pre[0] = level
    }

    return pre
  }, [])

  // 整数部分
  const _integer = levels.reduce((pre, item, idx) => {
    let _level = conf.level[levels.length - idx - 1]
    let _item = item.join('').replace(/(零)\1+/g, '$1') // 连续多个零字的部分设置为单个零字

    // 如果这一级只有一个零字, 则去掉这级
    if (_item === '零') {
      _item = ''
      _level = ''

      // 否则如果末尾为零字, 则去掉这个零字
    } else if (_item[_item.length - 1] === '零') {
      _item = _item.slice(0, _item.length - 1)
    }

    // 为负数时去掉 level
    if (_item === '[负]') {
      _level = ''
    }

    return pre + _item + _level
  }, '')

  // 小数部分
  let _decimal = decimal
    .map((item, idx) => {
      const unit = confs.decimal.unit
      const _unit = item !== '0' ? unit[unit.length - idx - 1] : ''

      if (decimal.length === (idx + 1) && _unit === '') {
        // 最后一个为 0
        return ''
      }

      return `${conf.num[item]}${_unit}`
    })
    .join('')

  // 如果是整数, 则补个整字
  return (_integer === '' ? `零${unitName}` : _integer === '[负]' ? _integer : `${_integer + unitName}`) + (_decimal || '整')
}
```
