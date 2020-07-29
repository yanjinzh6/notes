---
title: 窗口最大值列表
date: 2020-07-25 17:00:00
tags: '算法'
categories:
  - ['算法', '列表']
permalink: window-max-list
photo:
mathjax: true
---

## 简介

有一个整型数组 arr 和一个大小为 w 的窗口从数组的最左边滑到最右边, 窗口每次向右边滑一个位置

如下数组 [4, 3, 5, 4, 3, 3, 6, 7], 窗口大小为 3 时

```sh
[4, 3, 5], 4, 3, 3, 6, 7, 窗口中最大值为 5
4, [3, 5, 4], 3, 3, 6, 7, 窗口中最大值为 5
4, 3, [5, 4, 3], 3, 6, 7, 窗口中最大值为 5
4, 3, 5, [4, 3, 3], 6, 7, 窗口中最大值为 4
4, 3, 5, 4, [3, 3, 6], 7, 窗口中最大值为 6
4, 3, 5, 4, 3, [3, 6, 7], 窗口中最大值为 7
```

结果应该返回 [5, 5, 5, 4, 6, 7]

<!-- more -->

## 分析

假设数组长度为 N, 窗口大小为 w, 则采用双重循环即可解答, 这样的时间复杂度为 $O(N*w)$

也可以设计一个算法采用如上分析的步骤, 这样就可以使得时间复杂度为 $O(N)$, 关键在于利用双端队列来实现窗口最大值的更新, 原理就是保持当前队列中所有索引对应的值比当前值大, 并且队列长度最多为窗口大小, 这样就可以确定当前窗口的最大值就是队首索引所对应的值

## 实现

```java
public class WindowMaxList {
    static int[] getMaxWindow(int[] arr, int w) {
        assert arr != null && w > 0 && w < arr.length;
        // 使用双端队列来实现窗口最大值的更新, 队首元素保存着当前窗口最大值下标
        LinkedList<Integer> list = new LinkedList<>();
        // 结果列表
        int[] res = new int[arr.length - w + 1];
        // 结果索引
        int index = 0;
        int count = 0;
        for (int i = 0; i < arr.length; i ++) {
            count ++;
            // 当队列不为空时, 取队尾的索引和当前值比较, 小于当前值删除
            // 这里操作主要是确定以当前值为结尾的区间中当前值最小
            int sCount = 0;
            while (!list.isEmpty() && arr[list.peekLast()] < arr[i]) {
                if (sCount > 0) {
                    count ++;
                }
                sCount ++;
                list.pollLast();
            }
            // 添加到队尾
            list.addLast(i);
            // 如果队首的索引超过了窗口大小, 删除
            if (list.peekFirst() == i - w) {
                list.pollFirst();
            }
            // 获取当前窗口的最大值, 也就是队首索引对应的值
            if (i >= w - 1) {
                res[index++] = arr[list.peekFirst()];
            }
        }
        System.out.println(count);
        return res;
    }

    public static void main(String[] args) {
        int[] arr = {4,3,5,4,3,3,6,7,8,1,7,5,9,7,8};
        int[] res = getMaxWindow(arr, 5);
        System.out.println(Arrays.toString(res));
    }
}
```

## 结果

```sh
21
[5, 5, 6, 7, 8, 8, 8, 8, 9, 9, 9]

Process finished with exit code 0
```
