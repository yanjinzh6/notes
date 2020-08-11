---
title: 平衡二叉树
date: 2020-08-08 16:00:00
tags: '算法'
categories:
  - ['算法', '二叉树']
permalink: balanced-binary-tree
photo:
mathjax: true
---

## 简介

判断二叉树是否为平衡二叉树

## 解析

对于平衡二叉树的任何子树, 左子树和右子树的高度差都不超过 1, 这里满足递归的流程, 因为所有子树的结构都是一样的

- 计算条件
  - 获取当前树的高度
  - 获取当前树是否为平衡二叉树
- 判断条件
  - 如果为空树, 则当前根节点的树是平衡二叉树
  - 如果左子树不是平衡二叉树, 则当前根节点的树是不平衡的
  - 如果右子树不是平衡二叉树, 则当前根节点的树是不平衡的
  - 如果左子树和右子树的高度差超过 1, 则当前根节点的树是不平衡的
  - 否则当前根节点的树是平衡二叉树

<!-- more -->

## 实现

```java
public class BalancedBT {
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
        int height;
        boolean isBalanced;
    }

    static ReturnType process(Node root) {
        if (root == null) {
            // 当子树为空时, 高度为 0, 是平衡二叉树
            return new ReturnType(0, true);
        }
        // 递归得到左子树的全部信息
        ReturnType left = process(root.left);
        // 递归得到右子树的全部信息
        ReturnType right = process(root.right);
        // 取左右子树中最高的树再加上当前根节点这一层即为当前树的高度
        int height = Math.max(left.height, right.height) + 1;
        // 左右子树均为平衡二叉树且高度差不超过 1 即当前树是平衡二叉树
        boolean isBalanced = left.isBalanced && right.isBalanced && Math.abs(left.height - right.height) < 2;
        return new ReturnType(height, isBalanced);
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
        System.out.println(returnType.isBalanced);
    }
}
```

## 结果

```sh
false

Process finished with exit code 0
```
