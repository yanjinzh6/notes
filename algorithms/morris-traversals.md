---
title: Morris 遍历
date: 2020-08-02 12:00:00
tags: '算法'
categories:
  - ['算法', '二叉树']
permalink: morris-traversals
photo:
mathjax: true
---

## 简介

Morris 遍历, 由 Joseph Morris 于 1979 年提出, 避免用栈结构, 而是让底层节点指向 null 的空闲指针指向上层的某个节点, 从而完成下层到上层的移动

使用栈结构遍历二叉树额外的空间复杂度受到树高度的影响 $O(h)$,而 Morris 遍历仅使用几个变量实现, 所以可以做到额外空间复杂度 $O(1)$

## 分析

遍历过程如下

- 指定当前节点 cur 初始指向树的根节点
- 如果 cur 为 null 过程结束
- 如果 cur 没有左子树, 则将 cur = cur.right 向右移动
- 如果 cur 有左子树, 则找到 cur 左子树上最右的节点, 标记为 mostRight
  - 如果 mostRight 的右节点指向 null, 将 mostRight 的右节点指向 cur, 将 cur = cur.left 向左移动
  - 如果 mostRight 的右节点指向 cur, 表示 cur 左子树已经遍历, 即该节点作用是替代递归过程中的栈结构, 将 mostRight 的右节点指向 null, 将 cur = cur.right 向右移动

根据上面的过程可以得到所有右边界的所有节点数量为 $O(N)$, 每条右边界都遍历两次, 那么遍历所有节点左子树右边界的总代价为 $O(N)$, 因此 Morris 遍历的时间复杂度是 $O(N)$

- 先序遍历
  - 对于无左子树的节点 (只到达一次的节点), cur 指向的时候直接打印
  - 对于有左子树的节点 (到达两次的节点), 只在 cur 第一次到达的时候打印
- 中序遍历
  - 对于无左子树的节点 (只到达一次的节点), cur 指向的时候直接打印
  - 对于有左子树的节点 (到达两次的节点), 只在 cur 第二次到达的时候打印
- 后序遍历
  - 对于无左子树的节点 (只到达一次的节点), 直接跳过
  - 对于有左子树的节点 (到达两次的节点), cur 第二次到达时逆序打印左子树的右边界
  - 遍历完成后, 逆序打印整棵树的右边界

<!-- more -->

## 实现

```java
public class Morris {
    @RequiredArgsConstructor
    @AllArgsConstructor
    static class Node {
        @NonNull
        int value;
        public Node left;
        public Node right;
    }

    static void morrisPre(Node head) {
        if (head == null) {
            return;
        }
        Node cur = head;
        Node mostRight;
        while (cur != null) {
            mostRight = cur.left;
            if (mostRight != null) {
                // 当前 cur 有左子树
                // 找到 cur 左子树上最右的节点
                while (mostRight.right != null && mostRight.right != cur) {
                    mostRight = mostRight.right;
                }
                // 判断 cur 左子树最右节点的状态
                if (mostRight.right == null) {
                    // 指向 null 则将其指向 cur
                    mostRight.right = cur;
                    // 前序遍历打印
                    System.out.println(cur.value + " ");
                    // cur 向左移动
                    cur = cur.left;
                    // 回到最外层的 while, 继续判断 cur 的情况
                    continue;
                } else {
                    // cur 左子树最右节点指向 cur 则将其指向 null
                    mostRight.right = null;
                }
            } else {
                // 前序遍历打印
                System.out.println(cur.value + " ");
            }
            // cur 如果没有左子树则向右移动
            // 或者 cur 左子树上最右节点的右指针是指向 cur 的则向右移动
            cur = cur.right;
        }
        System.out.println();
    }

    static void morrisIn(Node head) {
        if (head == null) {
            return;
        }
        Node cur = head;
        Node mostRight;
        while (cur != null) {
            mostRight = cur.left;
            if (mostRight != null) {
                while (mostRight.right != null && mostRight.right != cur) {
                    mostRight = mostRight.right;
                }
                if (mostRight.right == null) {
                    mostRight.right = cur;
                    cur = cur.left;
                    continue;
                } else {
                    mostRight.right = null;
                }
            }
            System.out.println(cur.value + " ");
            cur = cur.right;
        }
        System.out.println();
    }

    static void morrisPos(Node head) {
        if (head == null) {
            return;
        }
        Node cur = head;
        Node mostRight;
        while (cur != null) {
            mostRight = cur.left;
            if (mostRight != null) {
                while (mostRight.right != null && mostRight.right != cur) {
                    mostRight = mostRight.right;
                }
                if (mostRight.right == null) {
                    mostRight.right = cur;
                    cur = cur.left;
                    continue;
                } else {
                    mostRight.right = null;
                    // 逆序打印左子树的右边界
                    printEdge(cur.left);
                }
            }
            cur = cur.right;
        }
        // 逆序打印整棵树的右边界
        printEdge(head);
        System.out.println();
    }

    private static void printEdge(Node head) {
        Node tail = reverseEdge(head);
        Node cur = tail;
        while (cur != null) {
            System.out.println(cur.value + " ");
            cur = cur.right;
        }
    }

    private static Node reverseEdge(Node from) {
        Node pre = null;
        Node next;
        while (from != null) {
            next = from.right;
            from.right = pre;
            pre = from;
            from = next;
        }
        return pre;
    }

    public static void main(String[] args) {
        Node node = new Node(1,
                new Node(2,
                        new Node(4), new Node(5)),
                new Node(3,
                        new Node(6), new Node(7)));

        morrisPre(node);
        morrisIn(node);
        morrisPos(node);
    }

}
```

## 结果

```sh
1
2
4
5
3
6
7

4
2
5
1
6
3
7

4
5
2
6
7
3
1


Process finished with exit code 0
```
