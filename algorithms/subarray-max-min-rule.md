---
title: 最大值减去最小值小于或等于 num 的子数组数量
date: 2020-08-01 12:00:00
tags: '算法'
categories:
  - ['算法', '队列']
permalink: subarray-max-min-rule
photo:
mathjax: true
---

## 简介

给定长度为 N 的数组 arr 和整数 num, 计算符合条件的子数组

$$
max(arr[i \dots j]) - min(arr[i \dots j]) \leq num, (0 \leq i \leq j \leq N)
$$

$max(arr[i \dots j])$ 表示子数组 $arr[i \dots j]$ 中的最大值, $min(arr[i \dots j])$ 表示最小值

<!-- more -->

## 分析

一般的解决方法可以通过穷举所有的子数组, 并且遍历所有子数组中的最大值和最小值, 统计出符合的结果, 这样的时间复杂度为 $O(N^3)$

下面通过双端队列实现时间复杂度为 $O(N)$ 的方法

根据问题可以得到以下结论

- 如果子数组 $arr[i \dots j]$ 满足条件, 则 $arr[i \dots j]$ 中的每一个子数组 $arr[k \dots m], (i \leq k \leq m \leq j)$ 都满足条件
- 如果子数组 $arr[i \dots j]$ 不满足条件, 则所有包含 $arr[i \dots j]$ 的数组 $arr[k \dots m], (k \leq i \leq j \leq m)$ 都不满足条件

所以通过给定子数组的第一个元素, 并且遍历数组, 然后取所有符合的子数组的结果集相加就可以

通过 j 不断增加遍历数组, 使得 $arr[i \dots j]$ 不满足条件, 这样便可以得到 $arr[i \dots j-1], arr[i \dots j-2], arr[i \dots j-3] \dots arr[i \dots i]$ 都是符合要求的, 总数为 $j-i$

通过 i 不断增加遍历数组, 可以得到 $arr[0 \dots j], arr[0 \dots j] \dots arr[j \dots j]$ 所有的满足的结果, 相加起来就是答案

### 实现

```java
public class Test6 {
    static int getNum(int[] arr, int num) {
        if (arr == null || arr.length == 0) {
            return 0;
        }
        // 更新当前窗口子数组的最小值
        LinkedList<Integer> min = new LinkedList<>();
        // 更新当前窗口子数组的最大值
        LinkedList<Integer> max = new LinkedList<>();
        // i...j 范围数组
        // res 记录结果
        int i = 0, j = 0, res = 0;
        while (i < arr.length) {
            // j++ 向右遍历
            while (j < arr.length) {
                // 确保不重复 j
                if (min.isEmpty() || min.peekLast() != j) {
                    // 递增队列, 保持队列中所有值都不小于当前值
                    while (!min.isEmpty() && arr[min.peekLast()] >= arr[j]) {
                        min.pollLast();
                    }
                    min.addLast(j);
                    // 递减队列, 保持队列中所有值都不大于当前值
                    while (!max.isEmpty() && arr[max.peekLast()] <= arr[j]) {
                        max.pollLast();
                    }
                    max.addLast(j);
                }
                // 如果不满足条件则停止, 此时的 arr[i, j] 是满足 i 开头的最大子数组, 所以包含的子数组都满足
                if (arr[max.getFirst()] - arr[min.getFirst()] > num) {
                    break;
                }
                System.out.println(arr[max.getFirst()] + " - " + arr[min.getFirst()] + " <= " + num);
                j++;
            }
            // 以 i 作为第一个元素的子数组, 满足条件的数量为 j - i 个
            res += j - i;
            // 将子数组第一个元素变为 i + 1
            if (min.peekFirst() == i) {
                min.pollFirst();
            }
            if (max.peekFirst() == i) {
                max.pollFirst();
            }
            i ++;
        }
        return res;
    }

    public static void main(String[] args) {
        int[] arr = {1, 5, 8};
        System.out.println(getNum(arr, 4));
    }
}
```

## 结果

```sh
1 - 1 <= 4
5 - 1 <= 4
8 - 5 <= 4
5

Process finished with exit code 0
```
