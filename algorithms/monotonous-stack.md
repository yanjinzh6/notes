---
title: 单调栈
date: 2020-07-25 19:00:00
tags: '算法'
categories:
  - ['算法', '栈']
permalink: monotonous-stack
photo:
mathjax: true
---

## 简介

单调栈是一种特殊的栈, 栈内的元素都保存一个单调性, 递增或递减, 利用这个特性可以解决一些特殊的问题

## 问题

给定一个数组 arr, 找出每一个元素左边和右边离当前位置最近且比当前值小的位置, 返回所有位置相关信息

例如 `arr = {3, 4, 1, 5, 6, 2, 7}`

返回二维数组

```js
res = {{-1, 2}, {0, 2}, {-1, -1}, {2, 5}, {3, 5}, {2, -1}, {5, -1}}
```

-1 表示不存在

<!-- more -->

## 分析

通过对每个位置向左右方向分别遍历一下可以解决问题, 这样时间复杂度就是 $O(N^2)$

使用单调栈结构可以使得时间复杂度为 $O(N)$, 保证栈里的值都是递增的, 分析情况如下

- 当前值小于栈顶元素, 则推出栈顶, 并且当前值作为右侧索引位置, 当前的栈顶为左侧索引位置
- 当前值大于栈顶元素, 压入栈
- 当前值等于栈顶元素, 使用列表结构保存, 使得列表索引位置是由小到大
- 处理完列表后推出所有的栈元素, 此时元素只有左侧索引位置, 右侧没有小于当前值

## 实现

```java
public class NearLess {
    static int[][] getNearLess(int[] arr) {
        // 结果数组
        int[][] res = new int[arr.length][2];
        // 递增单调栈, 保存索引, 列表结构可以保存数组里相同值的索引
        Stack<List<Integer>> stack = new Stack<>();
        for (int i = 0; i < arr.length; i ++) {
            // 判断栈中的元素, 只要比当前值大就弹出
            while (!stack.isEmpty() && arr[stack.peek().get(0)] > arr[i]) {
                makeRes(stack, res, i);
            }
            if (!stack.isEmpty() && arr[stack.peek().get(0)] == arr[i]) {
                // 如果索引对应的值相同, 那就直接加入到栈顶的列表中
                stack.peek().add(Integer.valueOf(i));
            } else {
                // 剩下大于栈顶的值, 直接入栈
                List<Integer> list = new ArrayList<>(1);
                list.add(i);
                stack.push(list);
            }
        }
        // 剩下的都是右侧没有小于它们值的
        // 对数组进行处理后, 弹出栈里面剩下的值
        while (!stack.isEmpty()) {
            makeRes(stack, res, -1);
        }
        return res;
    }

    static void makeRes(Stack<List<Integer>> stack, int[][] res, int cur) {
        List<Integer> pop = stack.pop();
        // 获取位于栈顶第二位置的列表中, 最后加入的索引就是小于栈顶值并且是左边最靠近的位置
        int leftLessIndex = stack.isEmpty() ? -1 : stack.peek().get(stack.peek().size() - 1);
        for (Integer p: pop) {
            res[p][0] = leftLessIndex;
            // 索引 i 是小于栈顶值并且是右边最靠近的位置
            res[p][1] = cur;
        }
    }

    public static void main(String[] args) {
        int[] arr = {3, 4, 1, 5, 6, 2, 7};
        int[][] res = getNearLess(arr);
        for (int[] r: res) {
            System.out.println(Arrays.toString(r));
        }
    }
}
```

## 结果

```sh
[-1, 2]
[0, 2]
[-1, -1]
[2, 5]
[3, 5]
[2, -1]
[5, -1]

Process finished with exit code 0
```
