---
title: 最大搜索二叉树
date: 2020-08-02 15:00:00
tags: '算法'
categories:
  - ['算法', '二叉树']
permalink: max-search-binary-tree
photo:
mathjax: true
---

## 简介

在一棵包含不同节点的二叉树中找出包含节点最多的搜索二叉树, 也就是找到最大搜索二叉树

## 分析

由于二叉树的结构特点是左右子树也是二叉树, 这样可以通过递归的方式将问题拆分成在每个子树中找到最大搜索二叉树

所以当前问题可以分解为

- 查找左右子树的最大搜索二叉树
- 根据左右子树的最大搜索二叉树信息通过判断确定当前树的最大搜索儿茶树信息
- 如果当前树构成搜索二叉树则返回当前树节点
- 从最底层计算回去, 在根节点处得到整个树的最佳解

这里由于搜索二叉树的性质, 左子树的值小于跟节点, 右子树的值大于根节点, 需要将子树的最大最小值保存起来, 方便与根节点进行比较

<!-- more -->

## 实现

```java
public class MaxSBT {
    @RequiredArgsConstructor
    @AllArgsConstructor
    static class Node {
        @NonNull
        int value;
        public Node left;
        public Node right;
    }

    @AllArgsConstructor
    static class ReturnType {
        Node maxBSTHead;
        int maxBSTSize;
        int min;
        int max;
    }

    static ReturnType process(Node root) {
        if (root == null) {
            // 当子树为空时, 最小值为系统最大, 最大值为系统最小
            return new ReturnType(null, 0, Integer.MAX_VALUE, Integer.MIN_VALUE);
        }
        // 递归得到左子树的全部信息
        ReturnType left = process(root.left);
        // 递归得到右子树的全部信息
        ReturnType right = process(root.right);
        // 获取当前子树根节点, 左子树和右子树最小值
        int min = Math.min(root.value, Math.min(left.min, right.min));
        // 获取当前子树根节点, 左子树和右子树最大值
        int max = Math.max(root.value, Math.max(left.max, right.max));
        // 获取当前子树中左子树和右子树中最大的搜索二叉树的节点数的最大值
        int maxBSTSize = Math.max(left.maxBSTSize, right.maxBSTSize);
        // 获取当前最大的搜索二叉树的节点
        Node maxBSTHead = left.maxBSTSize >= right.maxBSTSize ? left.maxBSTHead : right.maxBSTHead;
        if (left.maxBSTHead == root.left && right.maxBSTHead == root.right && root.value > left.max && root.value < right.min) {
            // 当左子树中的最大搜索二叉树是当前左子树, 右子树中的最大搜索二叉树是当前右子树
            // 并且当前子树根节点大于左子树中的最大值和小于右子树中的最小值
            // 即当前子树是最大搜索二叉树, 节点数量就是左边的数量加上右边的数量再加上根节点
            maxBSTSize = left.maxBSTSize + right.maxBSTSize + 1;
            // 将当前的最大搜索二叉树标识指向根节点
            maxBSTHead = root;
        }
        // 生成新的最大搜索二叉树信息, 将信息返回给上一层节点
        return new ReturnType(maxBSTHead, maxBSTSize, min, max);
    }

    public static void main(String[] args) {
        Node tree = new Node(6,
                new Node(1,
                        new Node(0), new Node(3)), new Node(12,
                new Node(10,
                        new Node(4,
                                new Node(2), new Node(5)), new Node(14,
                        new Node(11), new Node(15))), new Node(13,
                new Node(20), new Node(16))));

        ReturnType returnType = process(tree);
        System.out.printf("最大搜索二叉树根节点: %d\n节点数量: %d\n树最小值: %d\n树最大值: %d", returnType.maxBSTHead.value, returnType.maxBSTSize, returnType.min, returnType.max);
    }
}
```

## 结果

```sh
最大搜索二叉树根节点: 10
节点数量: 7
树最小值: 0
树最大值: 20
Process finished with exit code 0
```
