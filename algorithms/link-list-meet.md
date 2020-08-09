---
title: 链表相交
date: 2020-08-02 10:00:00
tags: '算法'
categories:
  - ['算法', '链表']
permalink: link-list-meet
photo:
mathjax: false
---

## 简介

判断两个链表相交有多种情况

- 两个链表都没有环
  - 两个链表存在相同的结尾则证明链表相交
  - 不存在相同的结尾则证明不相交
- 一个链表有环一个没有则不相交
- 两个链表都有环
  - 如果两个链表的入环点是一样的, 则相交点在入环前的同一点
  - 如果两个链表的入环点不一样并且两个入环点能互通则表示链表相交, 相交的点就是入环点
  - 如果两个链表的入环点不一样并且两个入环点不相通则链表不相交

## 分析

需要将问题分解为三个阶段

- 判断链表是否有环, 如果链表没环, 那么遍历链表肯定会有终点
  - 设置两个指针 slow 和 fast, 默认指向链表的表头 head, 随后遍历过程中 slow 每次移动一个节点, fast 每次移动两个节点
  - 如果 fast 先到终点, 即 next 为 null 的节点, 则表示链表无环
  - 如果 slow 和 fast 在某个位置相遇时, 则表示链表中存在环
  - 当链表中存在环时, 通过将 fast 重新指向表头 head, 然后两个指针每次均移动一个节点遍历链表, 当指针相遇时即为链表第一次入环的节点
- 判断两个无环链表是否相交
  - 分别遍历统计链表长度和找到最后一个节点
  - 如果两个链表的最后一个节点不相同则链表不相交
  - 如果两个链表的最后一个节点相同则表示链表相交, 两个链表从相交节点到最后节点都是相同的
  - 将较长的链表先从表头遍历去掉长度差, 然后再一同遍历到第一个相同的节点即为第一个相交的节点
- 判断两个有环链表是否相交
  - 如果两个入环的节点相同, 则两个链表的第一个相交的节点在入环的节点前
  - 分别遍历统计链表到入环节点的长度
  - 将较长的链表先从表头遍历去掉长度差, 然后再一同遍历到第一个相同的节点即为第一个相交的节点
  - 如果两个入环的节点不相同, 则判断这两个入环的节点能否相通, 相通了则表示链表相交, 两个入环点都可以作为第一个相交的节点, 不相通则表示链表不相交

<!-- more -->

## 实现

```java
public class ListMet {
    @RequiredArgsConstructor
    @AllArgsConstructor
    static class Node {
        @NonNull
        int value;
        public Node next;
    }

    static Node getLoopNode(Node head) {
        if (head == null || head.next == null || head.next.next == null) {
            return null;
        }
        // slow 节点每次指向下一个节点
        Node slow = head.next;
        // fast 节点每次指向后面第二个节点
        Node fast = head.next.next;
        while (slow != fast) {
            if (fast.next == null || fast.next.next == null) {
                // fast 节点指向终点
                return null;
            }
            fast = fast.next.next;
            slow = slow.next;
        }
        // slow 和 fast 相遇后 fast 指向表头 head
        fast = head;
        while (slow != fast) {
            // 改为 slow 和 fast 一样每次指向下一个节点
            slow = slow.next;
            fast = fast.next;
        }
        // 当 slow 和 fast 再次相遇时即为第一个入环的节点
        return slow;
    }

    static Node getNoLoopMet(Node head, Node head2) {
        if (head == null || head2 == null) {
            return null;
        }
        // 设置两个节点进行遍历
        Node cur = head;
        Node cur2 = head2;
        // 计算两个链表的长度差
        int n = 0;
        while (cur.next != null) {
            n ++;
            cur = cur.next;
        }
        while (cur2.next != null) {
            n --;
            cur2 = cur2.next;
        }
        // 如果两个链表没有同一个终点则表示链表是不相交的
        if (cur != cur2) {
            return null;
        }
        return getMetNode(head, head2, n);
    }

    static Node getMetNode(Node head, Node head2, int n) {
        // 通过两个节点分别指向表头 head
        Node cur = n > 0 ? head : head2;
        Node cur2 = cur == head ? head2 : head;
        // 获取差值的绝对值
        n = Math.abs(n);
        // 链表相交是尾部相同, 所以长链表先跳过相应的长度
        while (n != 0) {
            n --;
            cur = cur.next;
        }
        // 两个链表一同遍历, 直到相等就是第一个相交的点
        while (cur != cur2) {
            cur = cur.next;
            cur2 = cur2.next;
        }
        return cur;
    }

    static Node getBothLoopMet(Node head, Node loop, Node head2, Node loop2) {
        Node cur;
        Node cur2;
        if (loop == loop2) {
            // 如果两个链表的入环点相同, 则相交的点在入环点之前
            cur = head;
            cur2 = head2;
            int n = 0;
            // 通过确定入环前的链表来计算这段链表中相交的第一个交点
            while (cur != loop) {
                n ++;
                cur = cur.next;
            }
            while (cur2 != loop2) {
                n --;
                cur2 = cur2.next;
            }
            return getMetNode(head, head2, n);
        } else {
            // 如果两个链表的入环点不同, 则判断入环点是否相通
            cur = loop.next;
            while (cur != loop) {
                if (cur == loop2) {
                    // 入环点是相通的则表示相交, 返回其中一个入环点
                    return loop;
                }
                cur = cur.next;
            }
        }
        return null;
    }

    static Node getIntersectNode(Node head, Node head2) {
        if (head == null || head2 == null) {
            return null;
        }
        // 判断是否有环
        Node loop = getLoopNode(head);
        Node loop2 = getLoopNode(head2);
        // 如果都无环则判断交点
        if (loop == null && loop2 == null) {
            return getNoLoopMet(head, head2);
        }
        // 如果都有环则判断交点
        if (loop != null && loop2 != null) {
            return getBothLoopMet(head, loop, head2, loop2);
        }
        // 一个有环一个无环则不相交
        return null;
    }

    public static void main(String[] args) {
        Node node = new Node(1
                , new Node(2
                , new Node(3)));
        Node f = new Node(4
                , new Node(5
                , new Node(6)));
        Node f2 = new Node(7
                , new Node(8
                , new Node(9)));
        f2.next.next.next = f;
        node.next.next.next = f2;

        Node node2 = new Node(11
                , new Node(12
                , new Node(13
                , new Node(14))));
        node2.next.next.next.next = f;

        Node intersectNode = getIntersectNode(node, node2);
        if (intersectNode == null) {
            System.out.println("null");
        } else {
            System.out.println(intersectNode.value);
        }
    }
}
```

## 结果

```sh
4

Process finished with exit code 0
```
