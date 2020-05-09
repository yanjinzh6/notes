---
title: JavaScript 按位操作符
date: 2020-05-03 20:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-bitwise-operators
mathjax: true
---

## 简介

按位操作符 (Bitwise operators) 将其操作数 (operands) 当作32位的比特序列 (由 `0` 和 `1` 组成)

- 按位与 ( AND) `a & b`: 对于每一个比特位, 只有两个操作数相应的比特位都是 `1` 时, 结果才为 `1`, 否则为 `0`
- 按位或 (OR) `a | b`: 对于每一个比特位, 当两个操作数相应的比特位至少有一个 `1` 时, 结果为 `1`, 否则为 `0`
- 按位异或 (XOR) `a ^ b`: 对于每一个比特位, 当两个操作数相应的比特位有且只有一个 `1` 时, 结果为 `1`, 否则为 `0`
- 按位非 (NOT) `~ a`: 反转操作数的比特位, 即 `0` 变成 `1`, `1` 变成 `0`
- 左移 (Left shift) `a << b`: 将 `a` 的二进制形式向左移 `b` (< 32) 比特位, 右边用 `0` 填充
- 有符号右移 `a >> b`: 将 `a` 的二进制表示向右移 `b` (< 32) 位, 丢弃被移出的位
- 无符号右移 `a >>> b`: 将 `a` 的二进制表示向右移 `b` (< 32) 位, 丢弃被移出的位, 并使用 `0` 在左侧填充

<!-- more -->

## 有符号 32 位整数

所有的按位操作符的操作数都会被转成补码 (two's complement) 形式的有符号 `32` 位整数

补码形式是指一个数的负对应值 (negative counterpart) 为数值的所有比特位反转后, 再加 `1`

反转比特位即该数值进行 "非" 位运算, 也即该数值的反码

整数 `314` 的二进制编码 `00000000000000000000000100111010`

`~314`, 即 `314` 的反码 `11111111111111111111111011000101`

`-314`, 即 `314` 的反码再加 `1` `11111111111111111111111011000110`

补码保证了当一个数是正数时, 其最左的比特位是 `0`, 当一个数是负数时, 其最左的比特位是 `1`, 因此, 最左边的比特位被称为符号位 (sign bit)

`0` 是所有比特数字 `0` 组成的整数 `0 (base 10) = 00000000000000000000000000000000 (base 2)`

`-1` 是所有比特数字 `1` 组成的整数 `-1 (base 10) = 11111111111111111111111111111111 (base 2)`

`-2147483648` (十六进制形式: `-0x80000000`) 是除了最左边为 `1` 外, 其他比特位都为 `0` 的整数 `-2147483648 (base 10) = 10000000000000000000000000000000 (base 2)`

`2147483647` (十六进制形式: `0x7fffffff`) 是除了最左边为 `0` 外, 其他比特位都为 `1` 的整数 `2147483647 (base 10) = 01111111111111111111111111111111 (base 2)`

数字 `-2147483648` 和 `2147483647` 是 `32` 位有符号数字所能表示的最小和最大整数

## 按位逻辑操作符

从概念上讲, 按位逻辑操作符按遵守下面规则

- 操作数被转换成 `32` 位整数, 用比特序列 (0和1组成) 表示, 超过 `32` 位的数字会被丢弃
- 第一个操作数的每个比特位与第二个操作数的相应比特位匹配: 第一位对应第一位, 第二位对应第二位, 以此类推
- 位运算符应用到每对比特位, 结果是新的比特值

例如, 以下具有 `32` 位以上的整数将转换为 `32` 位整数:

```
转换前: 11100110111110100000000000000110000000000001
转换后:             10100000000000000110000000000001
```

### & (按位与)

与操作的真值表

| a | b | a AND b |
| -- | -- | -- |
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

```
     9 (base 10) = 00000000000000000000000000001001 (base 2)
    14 (base 10) = 00000000000000000000000000001110 (base 2)
                   --------------------------------
14 & 9 (base 10) = 00000000000000000000000000001000 (base 2) = 8 (base 10)
```

将任一数值 `x` 与 `0` 执行按位与操作, 其结果都为 `0`, 将任一数值 `x` 与 `-1` 执行按位与操作, 其结果都为 `x`

### | (按位或)

或操作的真值表

| a | b | a OR b |
| -- | -- | -- |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

```
     9 (base 10) = 00000000000000000000000000001001 (base 2)
    14 (base 10) = 00000000000000000000000000001110 (base 2)
                   --------------------------------
14 | 9 (base 10) = 00000000000000000000000000001111 (base 2) = 15 (base 10)
```

将任一数值 `x` 与 `0` 进行按位或操作, 其结果都是 `x`, 将任一数值 x 与 `-1` 进行按位或操作, 其结果都为 `-1`

例子

```js
1 | 0 ;                       // 1

1.1 | 0 ;                     // 1

'asfdasfda' | 0 ;             // 0

0 | 0 ;                       // 0

(-1) | 0 ;                    // -1

(-1.5646) | 0 ;               // -1

[] | 0 ;                      // 0

({}) | 0 ;                    // 0

"123456" | 0 ;            // 123456

1.23E2 | 0;               // 123

1.23E12 | 0;              // 1639353344

-1.23E2 | 0;              // -123

-1.23E12 | 0;             // -1639353344
```

### ^ (按位异或)

异或操作真值表

| a | b | a XOR b |
| -- | -- | -- |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

```
     9 (base 10) = 00000000000000000000000000001001 (base 2)
    14 (base 10) = 00000000000000000000000000001110 (base 2)
                   --------------------------------
14 ^ 9 (base 10) = 00000000000000000000000000000111 (base 2) = 7 (base 10)
```

将任一数值 `x` 与 `0` 进行异或操作, 其结果为 `x`, 将任一数值 `x` 与 `-1` 进行异或操作, 其结果为 `~x`

### ~ (按位非)

非操作的真值表

| a | NOT a |
| -- | -- |
| 0 | 1 |
| 1 | 0 |

```
 9 (base 10) = 00000000000000000000000000001001 (base 2)
               --------------------------------
~9 (base 10) = 11111111111111111111111111110110 (base 2) = -10 (base 10)
```

对任一数值 `x` 进行按位非操作的结果为 `-(x + 1)`, 例如, `~5` 结果为 `-6`

与 `indexOf` 一起使用示例

```js
var str = 'rawr';
var searchFor = 'a';

// 这是 if (-1*str.indexOf('a') <= 0) 条件判断的另一种方法
if (~str.indexOf(searchFor)) {
  // searchFor 包含在字符串中
} else {
  // searchFor 不包含在字符串中
}

// (~str.indexOf(searchFor))的返回值
// r == -1
// a == -2
// w == -3
```

## 按位移动操作符

按位移动操作符有两个操作数: 第一个是要被移动的数字, 而第二个是要移动的长度, 移动的方向根据操作符的不同而不同

按位移动会先将操作数转换为大端字节序顺序 (big-endian order) 的 `32` 位整数, 并返回与左操作数相同类型的结果, 右操作数应小于 `32` 位, 否则只有最低 `5` 个字节会被使用

注: **Big-Endian: 高位字节排放在内存的低地址端, 低位字节排放在内存的高地址端, 又称为"高位编址"**

Big-Endian 是最直观的字节序:

- 把内存地址从左到右按照由低到高的顺序写出
- 把值按照通常的高位到低位的顺序写出
- 两者对照, 一个字节一个字节的填充进去

### << (左移)

该操作符会将第一个操作数向左移动指定的位数, 向左被移出的位被丢弃, 右侧用 `0` 补充

`9 << 2` 得到 `36`

```
     9 (base 10): 00000000000000000000000000001001 (base 2)
                  --------------------------------
9 << 2 (base 10): 00000000000000000000000000100100 (base 2) = 36 (base 10)
```

在数字 `x` 上左移 `y` 比特得到 $x * 2^y$

### >> (有符号右移)

该操作符会将第一个操作数向右移动指定的位数, 向右被移出的位被丢弃, 拷贝最左侧的位以填充左侧, 由于新的最左侧的位总是和以前相同, 符号位没有被改变, 所以被称作 "符号传播"

`9 >> 2` 得到 `2`

```
     9 (base 10): 00000000000000000000000000001001 (base 2)
                  --------------------------------
9 >> 2 (base 10): 00000000000000000000000000000010 (base 2) = 2 (base 10)
```

`-9 >> 2` 得到 `-3`, 因为符号被保留了

```
     -9 (base 10): 11111111111111111111111111110111 (base 2)
                   --------------------------------
-9 >> 2 (base 10): 11111111111111111111111111111101 (base 2) = -3 (base 10)
```

### >>> (无符号右移)

该操作符会将第一个操作数向右移动指定的位数, 向右被移出的位被丢弃, 左侧用 `0` 填充, 因为符号位变成了 `0`, 所以结果总是非负的,  (译注: 即便右移 `0` 个比特, 结果也是非负的)

对于非负数, 有符号右移和无符号右移总是返回相同的结果, 例如 `9 >>> 2` 和 `9 >> 2` 一样返回 `2`

```
      9 (base 10): 00000000000000000000000000001001 (base 2)
                   --------------------------------
9 >>> 2 (base 10): 00000000000000000000000000000010 (base 2) = 2 (base 10)
```

但是对于负数却不尽相同,  `-9 >>> 2` 产生 `1073741821` 这和 `-9 >> 2` 不同

```
      -9 (base 10): 11111111111111111111111111110111 (base 2)
                    --------------------------------
-9 >>> 2 (base 10): 00111111111111111111111111111101 (base 2) = 1073741821 (base 10)
```

## 示例

### 标志位与掩码

位运算经常被用来创建、处理以及读取标志位序列——一种类似二进制的变量, 虽然可以使用变量代替标志位序列, 但是这样可以节省内存 (1/32)

掩码 (bitmask) 是一个通过与/或来读取标志位的位序列

```js
// 二进制 0101
var flags = 5;

// 典型的定义每个标志位的原语掩码如下
var FLAG_A = 1; // 0001
var FLAG_B = 2; // 0010
var FLAG_C = 4; // 0100
var FLAG_D = 8; // 1000

// 新的掩码可以在以上掩码上使用逻辑运算创建, 例如, 掩码 1011 可以通过 FLAG_A、FLAG_B 和 FLAG_D 逻辑或得到
var mask = FLAG_A | FLAG_B | FLAG_D; // 0001 | 0010 | 1000 => 1011

// 某个特定的位可以通过与掩码做逻辑与运算得到, 通过与掩码的与运算可以去掉无关的位, 得到特定的位, 例如, 掩码 0100 可以用来检查标志位 C 是否被置位
// 有 FLAG_C
if (flags & FLAG_C) { // 0101 & 0100 => 0100 => true
   // do stuff
}

// 一个有多个位被置位的掩码表达任一/或者的含义, 例如, 以下两个表达是等价的
// (0101 & 0010) || (0101 & 0100) => 0000 || 0100 => true
// 有 FLAG_B 或者 FLAG_C 至少一个
if ((flags & FLAG_B) || (flags & FLAG_C)) {
   // do stuff
}
// 有 FLAG_B 或者 FLAG_C 至少一个
var mask_2 = FLAG_B | FLAG_C; // 0010 | 0100 => 0110
if (flags & mask_2) { // 0101 & 0110 => 0100 => true
   // do stuff
}

// 可以通过与掩码做或运算设置标志位, 掩码中为 1 的位可以设置对应的位, 例如掩码 1100 可用来设置位 C 和 D
// 有 FLAG_C 和 FLAG_D
var mask_3 = FLAG_C | FLAG_D; // 0100 | 1000 => 1100
flags |= mask_3;   // 0101 | 1100 => 1101

// 可以通过与掩码做与运算清除标志位, 掩码中为 0 的位可以设置对应的位, 掩码可以通过对原语掩码做非运算得到, 例如, 掩码 1010 可以用来清除标志位 A 和 C
// 没有 FLAG_A 也没有 FLAG_C
var mask_4 = ~(FLAG_A | FLAG_C); // ~0101 => 1010
flags &= mask_4;   // 1101 & 1010 => 1000

// 如上的掩码同样可以通过 ~FLAG_A & ~FLAG_C 得到 (德摩根定律)
// 没有 FLAG_A 也没有 FLAG_C
var mask_5 = ~FLAG_A & ~FLAG_C;
flags &= mask_5;   // 1101 & 1010 => 1000

// 标志位可以使用异或运算切换, 所有值为 1 的位可以切换对应的位, 例如, 掩码 0110 可以用来切换标志位 B 和 C
// 如果以前没有 FLAG_B, 那么现在有 FLAG_B
// 但是如已经有了一个, 那么现在没有了
// 对 FLAG_C 也是相同的情况
var mask_6 = FLAG_B | FLAG_C;
flags = flags ^ mask_6

// 所有标志位可以通过非运算翻转
// entering parallel universe...
flags = ~flags;    // ~1010 => 0101
```

### 转换片段

```js
// 将一个二进制数的 String 转换为十进制的 Number
var sBinString = "1011";
var nMyNumber = parseInt(sBinString, 2);
alert(nMyNumber); // 打印 11
```

```js
// 将一个十进制的 Number 转换为二进制数的 String
var nMyNumber = 11;
var sBinString = nMyNumber.toString(2);
alert(sBinString); // 打印 1011
```

### 自动化掩码创建

```js
// 从一系列的 Boolean 值创建一个掩码
function createMask () {
  var nMask = 0, nFlag = 0, nLen = arguments.length > 32 ? 32 : arguments.length;
  for (nFlag; nFlag < nLen; nMask |= arguments[nFlag] << nFlag++);
  return nMask;
}
var mask1 = createMask(true, true, false, true); // 11, i.e.: 1011
var mask2 = createMask(false, false, true); // 4, i.e.: 0100
var mask3 = createMask(true); // 1, i.e.: 0001
// etc.

alert(mask1); // 打印 11
```

### 逆算法: 从掩码得到布尔数组

```js
// 从掩码得到得到 Boolean Array
function arrayFromMask (nMask) {
  // nMask 必须介于 -2147483648 和 2147483647 之间
  if (nMask > 0x7fffffff || nMask < -0x80000000) {
    throw new TypeError("arrayFromMask - out of range");
  }
  for (var nShifted = nMask, aFromMask = []; nShifted;
       aFromMask.push(Boolean(nShifted & 1)), nShifted >>>= 1);
  return aFromMask;
}

var array1 = arrayFromMask(11);
var array2 = arrayFromMask(4);
var array3 = arrayFromMask(1);

alert("[" + array1.join(", ") + "]");
// 打印 "[true, true, false, true]", i.e.: 11, i.e.: 1011
```

```js
// 同时测试以上两个算法
var nTest = 19; // our custom mask
var nResult = createMask.apply(this, arrayFromMask(nTest));

alert(nResult); // 19
```

```js
// 仅仅由于教学目的  (因为有 Number.toString(2) 方法) , 这里展示如何修改 arrayFromMask 算法通过 Number 返回二进制的 String, 而非 Boolean Array
function createBinaryString (nMask) {
  // nMask must be between -2147483648 and 2147483647
  for (var nFlag = 0, nShifted = nMask, sMask = ""; nFlag < 32;
       nFlag++, sMask += String(nShifted >>> 31), nShifted <<= 1);
  return sMask;
}

var string1 = createBinaryString(11);
var string2 = createBinaryString(4);
var string3 = createBinaryString(1);

alert(string1);
// 打印 00000000000000000000000000001011, i.e. 11
```

## 来源

- [按位操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators)