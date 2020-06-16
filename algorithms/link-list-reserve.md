---
title: 反转单向和双向链表
date: 2020-06-13 8:00:00
tags: '算法'
categories:
  - ['算法', '链表']
permalink: link-list-reserve
photo:
mathjax: true
---

## 问题

在时间复杂度 $O(n)$, 额外空间复杂度 $O(1)$ 的情况下翻转长度为 N 的链表

## 解析

实现过程比较简单, 主要通过两个额外的变量来记录翻转的链表的当前位置和已经翻转的链表, 通过迭代一遍链表即可翻转成功, 主要代码是 `head.next = pre;`, 通过将当前链表的 `next` 指针指向已经翻转的链表来得到当前环节已经翻转的链表

<!-- more -->

## Java 实现

```java
@Slf4j
public class LinkListDemo {
    static class Node {
        int value;
        Node next;

        public Node(int value, Node next) {
            this.value = value;
            this.next = next;
        }
    }

    static Node reserve(Node head) {
        Node pre = null;
        Node next = null;
        while (head != null) {
            next = head.next;
            head.next = pre;
            pre = head;
            head = next;
        }
        return pre;
    }

    static class TwoWayNode {
        int value;
        TwoWayNode pre;
        TwoWayNode next;

        public TwoWayNode(int value) {
            this.value = value;
        }
    }

    static TwoWayNode reserve(TwoWayNode head) {
        TwoWayNode pre = null;
        TwoWayNode next = null;
        while (head != null) {
            next = head.next;
            head.next = pre;
            head.pre = next;
            pre = head;
            head = next;
        }
        return pre;
    }

    public static void main(String[] args) {
        Node head = new Node(1, new Node(2, new Node(3, null)));
        Node pre = reserve(head);
        log.info("{} {}", head, pre);

        TwoWayNode f = new TwoWayNode(1);
        TwoWayNode s = new TwoWayNode(2);
        TwoWayNode t = new TwoWayNode(3);

        f.next = s;
        s.next = t;
        t.pre = s;
        s.pre = f;

        TwoWayNode p = reserve(f);
        log.info("{} {}", f, p);
    }
}
```
