---
title: 矩阵路径问题
date: 2020-08-09 16:00:00
tags: '算法'
categories:
  - ['算法', '动态规划']
permalink: matrix-path-sum
photo:
mathjax: true
---

## 简介

计算通过矩阵左上角到右下角的路径距离, 每次只能向右或者向下走, 要求出最短的路径, 如下矩阵

$$
\left[
  \begin{array}{cccc}
    1&3&5&9 \\\\
    8&1&3&4 \\\\
    5&0&6&1 \\\\
    8&8&4&0
  \end{array}
\right]
$$

路径 $1, 3, 1, 0, 6, 1, 0$ 是所有路径中路径和最小的, 等于 12

## 分析

经典的动态规划问题, 需要求解出所有局部最优解, 当前情况下的最优解就是计算每一步两个条件哪个更好, 时间复杂度为 $O(M \times N)$, 额外空间复杂度为 $O(min\{M, N\})$

<!-- more -->

## 实现

```java
public class Test {
    static int matrixSmallPath(int[][] m) {
        if (m == null || m.length == 0 || m[0] == null || m[0].length == 0) {
            return 0;
        }
        // 获取行列数
        // 数字大的为 more
        int more = Math.max(m.length, m[0].length);
        // 数字小的为 less
        int less = Math.max(m.length, m[0].length);
        // 判断行数是不是大于等于列数
        boolean rowMore = more == m.length;
        // 使用行列数中的最小值作为辅助数字的长度
        int[] arr = new int[less];
        // 设定起点
        arr[0] = m[0][0];
        // 从少的一边遍历, 如果行数少, 则从上到下, 列数少, 从左到右
        for (int i = 1; i < less; i ++) {
            // 第一行或者列只有一个方向可以走
            arr[i] = arr[i - 1] + (rowMore ? m[0][i] : m[i][0]);
        }
        System.out.println(Arrays.toString(arr));
        for (int i = 1; i < more; i ++) {
            // 从第二行或者列处理, 这时最靠近左边或者上边的项只有一个方向走
            arr[0] = arr[0] + (rowMore ? m[i][0] : m[0][i]);
            for (int j = 1; j < less; j ++) {
                // 直接全部遍历
                arr[j] = Math.min(arr[j - 1], arr[j]) + (rowMore ? m[i][j] : m[j][i]);
            }
            System.out.println(Arrays.toString(arr));
        }
        // 返回最后一项
        return arr[less - 1];
    }

    public static void main(String[] args) {
        int[][] matrix = {
                {1, 3, 5, 9},
                {8, 1, 3, 4},
                {5, 0, 6, 1},
                {8, 8, 4, 0}
        };

        System.out.println(matrixSmallPath(matrix));
    }
}
```

## 结果

```sh
[1, 4, 9, 18]
[9, 5, 8, 12]
[14, 5, 11, 12]
[22, 13, 15, 12]
12

Process finished with exit code 0
```
