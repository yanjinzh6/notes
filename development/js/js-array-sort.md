---
title: js-array-sort
date: 2021-03-13 18:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-array-sort
photo:
---

[js-array-sort](https://segmentfault.com/a/1190000017042511)

大家都知道 `js` 自带了一个排序方法 `sort`, 很多时候需要排序的时候也都直接使用了 `sort` 方法来排序. 然而有时候 `sort` 的结果和预期结果还是有些差异的.

看下面的代码

```
[1, 23, 2, 3].sort()
```

自然语言情况下, 我们期望的 排序结果应该是 `[1,2,3,23]` , 然而实际结果 `[1, 2, 23, 3]` .

首先我们来看下各个浏览器 ( js 引擎) 的排序算法.

| 浏览器 | 使用的 JavaScript 引擎 | 排序算法 | 源码地址 |
| --- | --- | --- | --- |
| Google Chrome | V8 | 插入排序和快速排序 | [`sort` 源码实现](https://github.com/v8/v8/blob/master/src/js/array.js#L768) |
| Mozilla Firefox | SpiderMonkey | 归并排序 | [`sort` 源码实现](https://github.com/mozilla/gecko-dev/blob/master/js/src/builtin/Array.js) |
| Safari | Nitro(JavaScriptCore ) | 归并排序和桶排序 | [`sort` 源码实现](https://github.com/WebKit/webkit/blob/master/Source/JavaScriptCore/builtins/ArrayPrototype.js#L423) |
| Microsoft Edge 和 IE(9+) | Chakra | 快速排序 | [`sort` 源码实现](https://github.com/Microsoft/ChakraCore/blob/master/lib/Common/DataStructures/QuickSort.h) |
| _注: 上述表格数据来源于 [排序算法](https://segmentfault.com/a/1190000010648740)_ |  |  |  |

以 `v8` 为例, 它分别使用了 插入排序和 快速排序, 分别使用的场景 可以查看源码, 也可以看 `justjavac` 大佬的 [这篇文章](https://segmentfault.com/a/1190000010630780) , 同时这篇文章也分析了一个排序的坑.

当 `sort` 没传递 `compareFn` 的时候, 就会使用默认的 `compareFn` , 而 `sort` 很多的坑都源于这里. 根据 `justjavac` 大大的那篇文章里的分析和 [`ecma` 标准里的声明](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-array.prototype.sort), 我们得知 `compareFn` 会在内部尝试调用 `toString` 将比较的对象转化成字符串.

在转化成字符串这一步之后, 坑就出现了. 当 `x` 和 `y` (这里 `x`, `y` 都已经是字符串了) 进行比较的, 其实是依据字典序来比较的, 而比较的对象其实数据的 `charCode`.

字典序的比较方法引用一下 维基百科上的说法:

> 是先按照第一个字母, 以 a, b, c……z 的顺序排列; 如果第一个字母一样, 那么比较第二个, 第三个乃至后面的字母. 如果比到最后两个单词不一样长 (比如, sigh 和 sight), 那么把短者排在前.

因此, 在 `[1, 23, 2, 3].sort()` 中, 比较的时候其实是 `'1','23','2','3',` 来进行比较的 (注意引号, 是字符串). 下面是这些字符串的 `charCode` 表:

| 1 | 23 | 2 | 3 |
| --- | --- | --- | --- |
| 49 | 50 51 | 50 | 51 |

`'1'` 和 `'23'` 比较, 先比较第一位 `charCode`, 49 小于 50, 直接判定成功,`'1'`<`'23'` , 而 `'23'` 和 `'2'` 比较, 第一位 `charCode` 相同, 判断不了, 看第二位; 而第二位上 一个是 `51`, 一个是 " 空 " 的, 因此 2 < 23. 而 `'23'` 和 `'3'` 比较, 由于 `2` 的 `charCode` 小于 `3` 的, 因此也直接判定成功,23 < 3.

类比可以考虑一下我们小学时候使用的字典 , 字典前面的拼音索引, `a`i 的拼音也始终在 `ba` 前面, 因为 a 的字典序小于 `b` .

同样的, 来看下英文: `['a','ba','b','c','ca']` .

| a | b | ba | c | ca |
| --- | --- | --- | --- | --- |
| 97 | 98 | 98 97 | 99 | 97 |

根据上面的规则, 我们可以很容易推断出排序结果: `["a", "b", "ba", "c", "ca"]`

最后看看中文的: `[' 啊 ',' 次 ',' 比例 ',' 中 ',' 毓 ',' 比侊 ']`

| 啊 | 次 | 比例 | 中 | 毓 | 比侊 |
| --- | --- | --- | --- | --- | --- |
| 21834 | 27425 | 27604 20363 | 20013 | 27603 | 27604 20362 |

同样的, 根据上面的规则, 结果也是一目了然:`[" 中 ", " 啊 ", " 次 ", " 毓 ", " 比侊 ", " 比例 "]`.

> 中文的 `charCode` 可以通过 `charCodeAt` 来获取.

这样的排序结果很多时候并不是我们需要的, 因此 `sort` 也允许我们传入一个 `compareFn` 来实现自定义排序规则, 通过返回 -1,0,1 来觉得排序结果的大小顺序. 对于可以比较的数值, 我们可以通过 `x-y` 的方式来处理, 那么中文又该怎么办呢? 能不能按照拼音的顺序来排序?

答案是肯定的!

String 现在有了一个 [localeCompare](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare) 的方法用于本地化比较.

```
[' 啊 ',' 次 ',' 比例 ',' 中 ',' 毓 ',' 比侊 '].sort((x, y) => x.localeCompare(y))
// [" 啊 ", " 比侊 ", " 比例 ", " 次 ", " 毓 ", " 中 "]
```

在 MDN 中, 在涉及到大量比较的时候更推荐的是 [Intl.Collator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Collator). **Intl** 对象是 ECMAScript 国际化 API 的一个命名空间, 它提供了精确的字符串对比, 数字格式化, 日期和时间格式化.

```
const collator = new Intl.Collator()
[' 啊 ',' 次 ',' 比例 ',' 中 ',' 毓 ',' 比侊 '].sort(collator.compare)
// [" 啊 ", " 比侊 ", " 比例 ", " 次 ", " 毓 ", " 中 "]
```

当然 `localeCompare` 和 `Intl.Collator` 允许传入参数指定 `locale`, 有兴趣的可以去 MDN 上看看用法.

最后, 内置 `sort` 的默认 `compareFn` 方法是基于字典序排序的, 而字典序比较的对象是 数据的 `charCode`. 当 `sort` 默认的排序行为和预期不一样或者无法满足需求的时候, 我们可以传入自定义的 `compareFn` 来进行排序. 对于中文或者需要本地化比较的场景下, 可以使用 `String.localeCompare` 或者 `Intl.Collator` 来进行比较.

最后的最后, 如有错误, 欢迎大家指出, 一起讨论进步.
