---
title: 543. 二叉树的直径
date: 2020-01-31 15:20:00
tags: '算法'
categories:
  - ['算法', '题目']
permalink: 543-diameter-of-binary-tree
photo:
---

# 描述

给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过根结点。

示例 :
给定二叉树

```sh
          1
         / \
        2   3
       / \     
      4   5    
```

返回 `3`, 它的长度是路径 `[4,2,1,3]` 或者 `[5,2,1,3]`。

注意：两结点之间的路径长度是以它们之间边的数目表示。

[来源：力扣（LeetCode）](https://leetcode-cn.com/problems/diameter-of-binary-tree)

<!-- more -->

# 想法

这是一道简单的题目, 主要是找出左右分支层数之和最高的节点就可以了, 如果能把节点值都设置为唯一, 应该可以使用 `HashMap` 来缓存已经计算过的节点的左右分支长度, 减少计算量.

# Java

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int maxDepth = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        getDepth(root);
        return maxDepth;
    }

    private int getDepth(TreeNode node) {
        if (node == null) {
            return 0;
        }
        int ld = getDepth(node.left);
        int rd = getDepth(node.right);
        maxDepth = Math.max(ld + rd, maxDepth);
        return Math.max(ld, rd) + 1;
    }
}
```

消耗内存比较高, 需要找时间分析一下原因.

# Javascript

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
var diameterOfBinaryTree = function(root) {
    return getDepth(root, 0).maxDepth
};
/**
 * @param {TreeNode} node
 * @param {number} maxDepth
 * @return {object}
 */
function getDepth(node, maxDepth) {
    if (!node) {
        return {
            maxDepth: 0,
            depth: 0
        }
    }
    const left = getDepth(node.left, maxDepth)
    const right = getDepth(node.right, maxDepth)
    maxDepth = Math.max(left.depth + right.depth, left.maxDepth, right.maxDepth)
    depth = Math.max(left.depth, right.depth) + 1
    return {
        maxDepth,
        depth
    }
}
```

由于 js 比较方便的返回, 所以不需要使用全局变量. 但是速度不是很好.