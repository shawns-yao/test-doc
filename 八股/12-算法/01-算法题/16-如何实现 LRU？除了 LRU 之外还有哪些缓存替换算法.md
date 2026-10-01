---
aliases:
- "如何实现 LRU？除了 LRU 之外还有哪些缓存替换算法？"
- "手撕代码 1.16 如何实现 LRU？除了 LRU 之外还有哪些缓存替换算法？"
---

# 16 如何实现 LRU？除了 LRU 之外还有哪些缓存替换算法？

## 01 核心代码

单线程 int 键值缓存，容量 0 不缓存，未命中返回 -1；get 和更新均刷新使用顺序。

```java
import java.util.*;

static class LRU {
    static class Node {
        int key, value; Node prev, next;
        Node(int key, int value) { this.key = key; this.value = value; }
    }
    final int capacity;
    final Map<Integer, Node> map = new HashMap<>();
    final Node head = new Node(0, 0), tail = new Node(0, 0);
    LRU(int capacity) {
        if (capacity < 0) throw new IllegalArgumentException();
        this.capacity = capacity; head.next = tail; tail.prev = head;
    }
    int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        unlink(n); addFirst(n); return n.value;
    }
    void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) { n.value = value; unlink(n); addFirst(n); return; }
        if (capacity == 0) return;
        n = new Node(key, value); map.put(key, n); addFirst(n);
        if (map.size() > capacity) {
            Node old = tail.prev; unlink(old); map.remove(old.key);
        }
    }
    void unlink(Node n) { n.prev.next = n.next; n.next.prev = n.prev; }
    void addFirst(Node n) {
        n.prev = head; n.next = head.next;
        head.next.prev = n; head.next = n;
    }
    // 直接变体：FIFO 命中不移位；LFU 按频次淘汰并定义同频规则；
    // Clock 用访问位+循环指针近似近期使用。TTL 与替换策略独立。
}
```

---

## 02 时间和空间复杂度

get/put 期望时间 O(1)；空间 O(capacity)。并发使用需同步保护 Map 与链表的复合修改。

---

## 03 参考与关联

- [LeetCode 146：LRU 行为与平均 O(1) 要求](https://leetcode.com/problems/lru-cache/)
- [[八股/12-算法/02-题型清单/03-03_链表|03_链表]]：反向关联：此题引用了本题的机制或边界

---

## 04 所属专题

- [[八股/12-算法/01-算法题/00-算法题导航|算法题导航]]
