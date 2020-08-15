---
title: 二叉树两个节点最近公共祖先
date: 2020-08-08 18:00:00
tags: '算法'
categories:
  - ['算法', '二叉树']
permalink: binary-tree-nearest-parent
photo:
mathjax: true
---

## 简介

判断两个节点最近的公共祖先, 其实就是求这两个节点在同一最小的子树, 最近的公共祖先就是该子树的根节点

## 分析

直接方法是后序遍历二叉树, 每个子树处理方式都相同, 所以可以设计递归方法

- 如果当前树的根节点为空, 那直接返回空
- 如果当前树的根节点为两个节点中一个, 则该根节点就是最近公共祖先
- 如果当前树左右子树都返回空, 说明该子树不存在相应的节点, 也就没有公共祖先, 返回空
- 如果当前树左右子树包含题中的两个节点, 说明该树就是最小的子树, 根节点就是最近的公共祖先
- 如果当前树左右子树中有一个为空
  - 不为空的节点有可能是节点中的一个, 也就是最近公共祖先是题中其一的节点
  - 或者不为空的节点就是得到的最近公共祖先, 返回它就可以了

在带有父节点的二叉树结构中是很容易实现的, 通过遍历父节点判断两个节点最先相同的就是结果, 在不满足的结构中, 可以通过使用哈希表来实现相应的关系, 然后只需要找到相同的映射值就是最近的公共祖先

这一实现建立哈希表的时间复杂度为 $O(N)$, 额外空间复杂度为 $O(N)$, 查询操作时, 时间复杂度为 $O(h)$, h 为二叉树高度

<!-- more -->

## 实现

```java
public class NearestParent {
    @RequiredArgsConstructor
    @AllArgsConstructor
    static class Node {
        @NonNull
        int value;
        public Node left;
        public Node right;
    }

    static Node nearestParent(Node head, Node n, Node n2) {
        // 当前节点为空返回空
        // 如果当前节点就是其中之一, 返回当前节点
        if (head == null || head == n || head == n2) {
            return head;
        }
        // 两个节点一样的随便都是
        if (n == n2) {
            return n;
        }
        // 后序遍历 LRC
        Node left = nearestParent(head.left, n, n2);
        Node right = nearestParent(head.right, n, n2);
        // 如果左右子树都存在则当前树就是拥有两个节点的最小子树
        if (left != null && right != null) {
            // 返回根节点
            return head;
        }
        // 如果都不存在则返回空
        // 如果只有一个存在, 则存在的节点就是最近的公共祖先
        return left != null ? left : right;
    }

    static Map<Node, Node> parentCache = new HashMap<>();

    static void setMap(Node head) {
        if (head == null) {
            return;
        }
        // 将树映射成 left -> parent
        if (head.left != null) {
            parentCache.put(head.left, head);
        }
        // right -> parent
        if (head.right != null) {
            parentCache.put(head.right, head);
        }
        setMap(head.left);
        setMap(head.right);
    }

    static Node queryNarestParent(Node n, Node n2) {
        Set<Node> path = new HashSet<>();
        // 找到其中一个节点的路径
        while (parentCache.containsKey(n)) {
            path.add(n);
            n = parentCache.get(n);
        }
        // 遍历另外一个节点当其祖先节点存在路径中即为最近公共祖先
        while (!path.contains(n2)) {
            n2 = parentCache.get(n2);
        }
        return n2;
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

        node.left = node2;
        node.right = node3;
        node2.left = node4;
        node2.right = node5;
        node3.left = node6;
        node3.right = node7;
        node4.left = node8;
        node7.right = node9;

        System.out.printf("节点 1, 2 公共节点为 %d\n" +
                        "节点 3, 4 公共节点为 %d\n" +
                        "节点 5, 6 公共节点为 %d\n" +
                        "节点 4, 8 公共节点为 %d\n" +
                        "节点 1, 9 公共节点为 %d\n" +
                        "节点 4, 5 公共节点为 %d\n" +
                        "节点 7, 6 公共节点为 %d\n"+
                        "节点 3, 5 公共节点为 %d\n\n",
                nearestParent(node, node, node2).value,
                nearestParent(node, node3, node4).value,
                nearestParent(node, node5, node6).value,
                nearestParent(node, node4, node8).value,
                nearestParent(node, node, node9).value,
                nearestParent(node, node4, node5).value,
                nearestParent(node, node7, node6).value,
                nearestParent(node, node3, node5).value);

        parentCache.put(node, null);
        setMap(node);

        System.out.printf("节点 1, 2 公共节点为 %d\n" +
                        "节点 3, 4 公共节点为 %d\n" +
                        "节点 5, 6 公共节点为 %d\n" +
                        "节点 4, 8 公共节点为 %d\n" +
                        "节点 1, 9 公共节点为 %d\n" +
                        "节点 4, 5 公共节点为 %d\n" +
                        "节点 7, 6 公共节点为 %d\n"+
                        "节点 3, 5 公共节点为 %d\n",
                queryNarestParent(node, node2).value,
                queryNarestParent(node3, node4).value,
                queryNarestParent(node5, node6).value,
                queryNarestParent(node4, node8).value,
                queryNarestParent(node, node9).value,
                queryNarestParent(node4, node5).value,
                queryNarestParent(node7, node6).value,
                queryNarestParent(node3, node5).value);
    }
}
```

## 结果

```sh
节点 1, 2 公共节点为 1
节点 3, 4 公共节点为 1
节点 5, 6 公共节点为 1
节点 4, 8 公共节点为 4
节点 1, 9 公共节点为 1
节点 4, 5 公共节点为 2
节点 7, 6 公共节点为 3
节点 3, 5 公共节点为 1

节点 1, 2 公共节点为 1
节点 3, 4 公共节点为 1
节点 5, 6 公共节点为 1
节点 4, 8 公共节点为 4
节点 1, 9 公共节点为 1
节点 4, 5 公共节点为 2
节点 7, 6 公共节点为 3
节点 3, 5 公共节点为 1

Process finished with exit code 0
```
