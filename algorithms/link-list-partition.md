---
title: 将链表根据条件分区
date: 2020-08-02 08:00:00
tags: '算法'
categories:
  - ['算法', '链表']
permalink: link-list-partition
photo:
mathjax: false
---

## 简介

将链表根据条件分为左中右三部分, 确保左边链表小于条件值, 中间的链表等于条件值, 右边的链表大于条件值

## 分析

通过简单的标识将链表划分为三个子链表, 然后再拼接起来, 可以简单实现

<!-- more -->

## 实现

```java
public class ListPartition {
    @RequiredArgsConstructor
    @AllArgsConstructor
    static class Node {
        @NonNull
        int value;
        public Node next;
    }

    /**
     * 将链表按照标识分区
     * @param node
     * @param flag
     * @return
     */
    static Node listPartition(Node node, int flag) {
        // small 链表表头标识
        Node sH = null;
        // small 链表表尾标识
        Node sT = null;
        // equal 链表表头标识
        Node eH = null;
        // equal 链表表尾标识
        Node eT = null;
        // big 链表表头标识
        Node bH = null;
        // big 链表表尾标识
        Node bT = null;
        // 保存下一个节点
        Node next;
        // 循环判断并将节点分别加入三个链表中
        while (node != null) {
            next = node.next;
            node.next = null;
            if (node.value < flag) {
                if (sH == null) {
                    sH = node;
                    sT = node;
                } else {
                    // 将符合的节点链接到后面并将表尾指向当前节点
                    sT.next = node;
                    sT = node;
                }
            } else if (node.value == flag) {
                if (eH == null) {
                    eH = node;
                    eT = node;
                } else {
                    eT.next = node;
                    eT = node;
                }
            } else {
                if (bH == null) {
                    bH = node;
                    bT = node;
                } else {
                    bT.next = node;
                    bT = node;
                }
            }
            node = next;
        }
        if (sT != null) {
            // 链接 equal 链表
            sT.next = eH;
            // 如果 equal 链表不存在, 则将 equal 链表的表尾指向 small 链表
            eT = eT == null ? sT : eT;
        }
        if (eT != null) {
            // 链接 big 链表
            eT.next = bH;
        }
        // 返回存在的链表
        return sH != null ? sH : eH != null ? eH : bH;
    }

    public static void main(String[] args) {
        Node node = new Node(7
                , new Node(9
                , new Node(3
                , new Node(8
                , new Node(5
                , new Node(2
                , new Node(5)))))));
        node = listPartition(node, 5);
        while (node != null) {
            System.out.printf(node.value + " > ");
            node = node.next;
        }
    }
}
```

## 结果

```sh
3 > 2 > 5 > 5 > 7 > 9 > 8 >
Process finished with exit code 0
```
