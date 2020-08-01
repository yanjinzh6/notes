---
title: 最大子矩阵
date: 2020-08-01 08:00:00
tags: '算法'
categories:
  - ['算法', '栈']
permalink: maximal-submatrix
photo:
mathjax: true
---

## 简介

给定一个整型矩阵 map，其中的值只有 0 和 1 两种，求其中全是 1 的所有矩形区域中，最大的矩形区域为 1 的数量

例如

```js
[1, 0, 1, 1]
[1, 0, 1, 1]
[1, 1, 1, 0]
```

最大子矩阵大小应该为 4

<!-- more -->

## 分析

这里采用穷举的方式, 计算所有符合的矩阵, 并通过比较大小获得最大值

如何将所有的矩形都列举出来, 可以简单的以矩阵的行为起点计算每一列的高度, 再分析每一列左右边各有多少列能合并为矩形即可

通过上面的方法, 可以得到第一行的矩形分别是 `{1, 0, 1, 1`, 第一二行的矩形是 `{2, 0, 2, 2}` 以此类推, 第一二三行的矩形是 `{3, 1, 3, 0}`

接下来问题便转换为计算各种矩形组合, 这时候需要使用到单调递减栈来进行协助

根据单调递减的特性, 可以知道栈顶到距离左侧最近一个小于栈顶高度的位置, 矩形需要高度一致, 所以该位置右边一位就是矩形的左边

右侧位置只需要计算到当前位置即可, 就算右侧位置还很多不确定, 但是穷举方法会将后面的结果计算出来

## 实现

```java
public class MaxRec {
    static int maxRecSize(int[][] map) {
        if (map == null || map.length == 0 || map[0].length == 0) {
            return 0;
        }
        int maxArea = 0;
        int[] height = new int[map[0].length];
        // 循环统计每一列的值
        for (int i = 0; i < map.length; i ++) {
            for (int j = 0; j < map[0].length; j ++) {
                // 计算值时, 当前值为 0 时直接设置为 0, 因为这样矩形的面积已经被打破了
                height[j] = map[i][j] == 0 ? 0 : height[j] + 1;
            }
            // 仅保存最大值
            maxArea = Math.max(maxRecFromBottom(height, i), maxArea);
        }
        return maxArea;
    }

    static int maxRecFromBottom(int[] height, int line) {
        if (height == null || height.length == 0) {
            return 0;
        }
        int maxArea = 0;
        // 单调递减
        Stack<Integer> stack = new Stack<>();
        for (int i = 0; i < height.length; i ++) {
            // 比较值, 将左右相邻的不小于当前值的索引进行计算
            while (!stack.isEmpty() && height[i] <= height[stack.peek()]) {
                // 推出需要计算的栈顶索引, 当 j 位置值大于 i 位置时, 最右侧也就只到达 i - 1 位置
                // 当 j 位置等于 i 位置的值时, 由于后面计算 i 位置的结果肯定比 j 大, 也不需要考虑
                int j = stack.pop();
                // 由于单调递减的特点, 现在的栈顶索引就是 j 位置左侧距离最近的比 j 位置值小的位置, 最左侧是不能算进来的必须 +1
                // 当 j 位置的值已经是最大时, 说明已经是最左侧了, 所以取 -1 才能计算正确位置
                int k = stack.isEmpty() ? -1 : stack.peek();
                // 这里计算的是 j 的结果, 所以值应该是右侧索引减去左侧索引再乘以高
                int curArea = (i - k - 1) * height[j];
                // 通过高度反向推导位置
                int top = line - (height[j] - 1);
                System.out.println("矩形位置: [{" + top + ", " + (k + 1) + "}, {" + line + ", " + (i - 1) + "}], 大小: " + curArea);
                maxArea = Math.max(maxArea, curArea);
            }
            stack.push(i);
        }
        // 计算最后的值和 0 的值
        while (!stack.isEmpty()) {
            int j = stack.pop();
            int k = stack.isEmpty() ? -1 : stack.peek();
            int curArea = (height.length - k - 1) * height[j];
            // 通过高度反向推导位置
            int top = line - (height[j] - 1);
            System.out.println("矩形位置: [{" + top + ", " + (k + 1) + "}, {" + line + ", " + (height.length - 1) + "}], 大小: " + curArea);
            maxArea = Math.max(maxArea, curArea);
        }
        return maxArea;
    }

    public static void main(String[] args) {
        int[][] map = {
                {1, 0, 1, 1},
                {1, 0, 1, 1},
                {1, 1, 1, 0}};
        for (int[] line : map) {
            System.out.println(Arrays.toString(line));
        }
        System.out.println("最大值: " + maxRecSize(map));
    }
}
```

## 结果

```sh
[1, 0, 1, 1]
[1, 0, 1, 1]
[1, 1, 1, 0]
矩形位置: [{0, 0}, {0, 0}], 大小: 1
矩形位置: [{0, 2}, {0, 2}], 大小: 1
矩形位置: [{0, 2}, {0, 3}], 大小: 2
矩形位置: [{1, 0}, {0, 3}], 大小: 0
矩形位置: [{0, 0}, {1, 0}], 大小: 2
矩形位置: [{0, 2}, {1, 2}], 大小: 2
矩形位置: [{0, 2}, {1, 3}], 大小: 4
矩形位置: [{2, 0}, {1, 3}], 大小: 0
矩形位置: [{0, 0}, {2, 0}], 大小: 3
矩形位置: [{0, 2}, {2, 2}], 大小: 3
矩形位置: [{2, 0}, {2, 2}], 大小: 3
矩形位置: [{3, 0}, {2, 3}], 大小: 0
最大值: 4

Process finished with exit code 0
```

## 小结

对于矩阵大小为 $N*M$ 的矩阵时间复杂度为 $O(N*M)$, 实现过程中需要添加的剪枝逻辑就是判断当前高度是否有效, 还有是否可用通过一定的规则降低计算各种小矩形
