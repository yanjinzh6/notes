---
title: 二叉树后继节点
date: 2020-08-08 17:00:00
tags: '算法'
categories:
  - ['算法', '二叉树']
permalink: binary-tree-next-node
photo:
mathjax: true
---

## 简介

在二叉树的中序遍历的序列中, Node 节点的下一个节点叫做 Node 的后继节点

## 解析

对于一棵节点分别指向父节点和子节点的树, 有如下结构

```java
@RequiredArgsConstructor
@AllArgsConstructor
class Node {
    @NonNull
    int value;
    public Node parent;
    public Node left;
    public Node right;
}
```

最简单的方法就是根据父节点指针一直获取到根节点, 再进行中序遍历, 这样就可以得到当前节点的后继节点, 这样需要时间复杂度和额外空间复杂度都是 $O(N)$

优化的解法是通过以下规律进行计算, 根据中序遍历可知

- 如果当前节点有右子树, 那么后继节点就是右子树上最左的节点
- 如果当前节点没有右子树, 则判断当前节点是该节点的父节点的哪个子节点
  - 如果是左子节点, 则当前节点的父节点就是该节点的后继节点
  - 如果是右子节点, 那么需要向上找到祖先节点是属于左子节点的情况, 则该祖先节点的父节点就是当前节点的后继节点

可以实现时间复杂度为 $O(L)$, L 为当前节点到后继节点之间的实际距离, 额外空间复杂度为 $O(1)$

<!-- more -->

## 实现

```java
public class NextNode {
    @RequiredArgsConstructor
    @AllArgsConstructor
    static class Node {
        @NonNull
        int value;
        public Node parent;
        public Node left;
        public Node right;
    }

    static Node getNextNode(Node node) {
        if (node == null) {
            return node;
        }
        if (node.right != null) {
            // 当前节点有右子树, 那么后继节点就是右子树上最左的节点
            return getLeftMost(node.right);
        } else {
            Node parent = node.parent;
            while (parent != null && parent.left != node) {
                // 当前节点是右子节点, 向上找到祖先节点是属于左子节点的情况
                node = parent;
                // 该祖先节点的父节点就是当前节点的后继节点
                parent = node.parent;
            }
            // 当前节点是左子节点, 则当前节点的父节点就是该节点的后继节点
            return parent;
        }
    }

    private static Node getLeftMost(Node node) {
        if (node == null) {
            return node;
        }
        // 遍历最左的节点
        while (node.left != null) {
            node = node.left;
        }
        return node;
    }

    public static void main(String[] args) {
        Node node = new Node(1);
        Node node2 = new Node(2);
        Node node3 = new Node(3);
        Node node4 = new Node(4);
        Node node5 = new Node(5);
        Node node6 = new Node(6);
        Node node7 = new Node(7);
        Node node8 = new Node(8);
        Node node9 = new Node(9);

        node.parent = node3;
        node.right = node2;
        node2.parent = node;
        node3.parent = node6;
        node3.left = node;
        node3.right = node4;
        node4.parent = node3;
        node4.right = node5;
        node5.parent = node4;
        node6.left = node3;
        node6.right = node9;
        node7.parent = node8;
        node8.left = node7;
        node8.parent = node9;
        node9.left = node8;
        node9.parent = node6;

        System.out.printf("节点 1 后继节点为 %d\n" +
                "节点 2 后继节点为 %d\n" +
                "节点 3 后继节点为 %d\n" +
                "节点 4 后继节点为 %d\n" +
                "节点 5 后继节点为 %d\n" +
                "节点 6 后继节点为 %d\n" +
                "节点 7 后继节点为 %d\n" +
                "节点 8 后继节点为 %d\n" +
                "节点 9 后继节点为 %d\n",
                getNextNode(node).value,
                getNextNode(node2).value,
                getNextNode(node3).value,
                getNextNode(node4).value,
                getNextNode(node5).value,
                getNextNode(node6).value,
                getNextNode(node7).value,
                getNextNode(node8).value,
                getNextNode(node9));
    }
}
```

## 结果

```sh
节点 1 后继节点为 2
节点 2 后继节点为 3
节点 3 后继节点为 4
节点 4 后继节点为 5
节点 5 后继节点为 6
节点 6 后继节点为 7
节点 7 后继节点为 8
节点 8 后继节点为 9
节点 9 后继节点为 null

Process finished with exit code 0
```
