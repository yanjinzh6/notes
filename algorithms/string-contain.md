---
title: 字符串包含问题
date: 2020-05-04 8:00:00
tags: '算法'
categories:
  - ['算法', '字符串']
permalink: string-contain
photo:
---

## 问题描述

有两个字符串 `str1` 和 `str2`, 如何判断 `str2` 中的所有字符是否被字符串 `str1` 包含, 即 `str2` 中所有字符是否都在 `str1` 中出现

## 分析

通过散列表可以简单的实现, 时间复杂度为 `O(n+m)`, 空间复杂度为 `O(n)`, 通过 `HashSet.add(e)` 方法添加 `str1` 所有字符, 通过 `HashSet.contains(e)` 可以判断 `str2` 中的字符是否包含在 `str1` 中

如果只是简单判断 `26` 个字符的字符串序列, 可以使用位运算来实现, 通过 `str1` 进行或运算计算出一个位运算的结果, 然后与 `str2` 的每个字符进行与运算就可以确定是否包含了, 这样时间复杂度与散列表一样, 但是空间复杂度可以实现 `O(1)`

## 位运算简单实现

通过将 `str1` 字符串转换为一个二进制数, 由 `A` 到 `Z` 依次从低位到高位排列

例如, `A` 可以转换成二进制 `0001`, `B` 转换为 `0010`, `ABD` 转换为 `1011`, 然后分别与 `str2` 中的字符进行与运算, 运算结果为 `0` 即表示当前字符不在 `str1` 中

例如 `ABD` 与 `C` 的与运算结果

```
  1011
& 0100
  ----
  0000
```

结果为 `0` 即表示 `ABD` 中不包含 `C`

```java
    public static boolean contain(String str, String str2) {
        int res = 0;
        for (int i = 0; i < str.length(); i ++) {
            res |= (1 << str.charAt(i) - 'A');
        }
        for (int j = 0; j < str2.length(); j ++) {
            if ((res & (1 << str2.charAt(j) - 'A')) == 0) {
                System.out.println("false");
                return false;
            }
        }
        System.out.println("true");
        return true;
    }
    public static void main(String[] args) {
        String str = "ABCDEFGH", str2 = "BDFG", str3 = "BFHK";
        contain(str, str2);
        contain(str, str3);
    }
```
