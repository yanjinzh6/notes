---
title: 852. 山脉数组的峰顶索引
date: 2020-02-01 15:20:00
tags: '算法'
categories:
  - ['算法', '题目']
permalink: 852-peak-index-in-a-mountain-array
photo:
---

# 描述

我们把符合下列属性的数组 `A` 称作山脉：

- `A.length >= 3`
- 存在 `0 < i < A.length - 1` 使得 `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]`

给定一个确定为山脉的数组，返回任何满足 `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]` 的 `i` 的值。

**示例 1：**

```sh
输入：[0,1,0]
输出：1
```

**示例 2：**

```sh
输入：[0,2,1,0]
输出：1
```

提示：

1. `3 <= A.length <= 10000`
1. `0 <= A[i] <= 10^6`
1. `A` 是如上定义的山脉

[来源：力扣（LeetCode）](https://leetcode-cn.com/problems/diameter-of-binary-tree)

<!-- more -->

# 想法

第一想法就是二分, 基本上就是时间复杂度 `O(logn)` 的思路了, 当然这里使用遍历也可以很快的获得结果, 甚至很多的情况下遍历会更快

二分法需要注意的就是切换为后部分的列表时需要存储当前位置, 每次叠加位置时需要加上 `1`

# Javascript

```js
/**
 * @param {number[]} A
 * @return {number}
 */
var peakIndexInMountainArray = function(A) {
    let tmp = 0
    function getMax(list) {
        const m = parseInt(list.length / 2)
        if (list[m - 1] < list[m] && list[m] > list[m + 1]) {
            return tmp > 0 ? m + tmp + 1 : m + tmp
        } else if (list[m] < list[m + 1]) {
            if (list.length - m - 1 < 3) {
                if (list[m + 1] < list[list.length - 1]) {
                    return tmp > 0 ? tmp + list.length : tmp + list.length - 1
                } else {
                    return tmp > 0 ? tmp + m + 2 : tmp + m + 1
                }
            }
            if (tmp === 0) {
                tmp += m
            } else {
                tmp += m + 1
            }
            return getMax(list.slice(m + 1, list.length))
        } else if (list[m] < list[m - 1]) {
            if (m < 3) {
                if (list[0] < list[1]) {
                    return tmp > 0 ? tmp + 2 : tmp + 1
                } else {
                    return tmp > 0 ? tmp + 1 : tmp
                }
            }
            return getMax(list.slice(0, m))
        }
    }
    return getMax(A)
};
```

后期进行精简