---
aliases:
- "HashMap 为什么线程不安全？ConcurrentHashMap 如何保证线程安全？哪些操作是 CAS、哪些上锁？"
- "Java 1.2 HashMap 为什么线程不安全？ConcurrentHashMap 如何保证线程安全？哪些操作是 CAS、哪些上锁？"
---

# HashMap 为什么线程不安全？ConcurrentHashMap 如何保证线程安全？哪些操作是 CAS、哪些上锁？

## 01 核心回答


**HashMap 为什么不安全：**
① 并发 put 时两个线程算到同一桶，后写覆盖前写（丢数据）；
② 扩容期间多线程同时 重新 hash，JDK 7 头插法可能形成环形链表导致 get 死循环；
③ 并发结构修改使遍历结果不可依赖；fail-fast 只是尽力检测错误，modCount 不提供同步保障。没有任何同步机制。

**ConcurrentHashMap（JDK 8）实现：**采用 **CAS + synchronized 锁桶头节点**，写路径分四步：

1. **桶为空 → CAS 直接插入**（无锁路径，期望 null 才写入）。

2. **桶非空 → synchronized 锁桶头节点**再操作——锁粒度 = 单个桶，多线程可并发写不同桶。

3. **扩容由多线程协助**（transfer 迁移，分担 rehash）。

4. **size 统计用 CounterCell 分段累加**。JDK 7 则是 Segment 分段锁（继承 ReentrantLock）。

**关键细节：** CAS 比较的是**桶位是否为 null（期望 null 才插入）；读操作不加锁**——Node 的 val 和 next 是 `volatile`，保证可见性；锁只锁头节点，链表/树内操作在锁内完成。

**面试追问：**CAS 失败怎么办（自旋重试）；为什么读不用锁也不会读到脏数据（volatile + 不变性设计）；和 Hashtable 的区别（全表锁 vs 桶锁，并发度差异）。

---

## 02 线程安全保证到哪一层

ConcurrentHashMap 保护映射的单次操作，不自动保护 value 对象内部状态，也不把 get 后 put 组合成原子更新。计数、按条件替换应使用 compute、merge、replace 等适合业务的原子操作，或给 value 自身同步。回调应短小，避免阻塞 I/O、递归更新同一映射和复杂交叉依赖。

get 返回某个更新后的非 null 值时，与相应插入/更新建立 happens-before。迭代是弱一致的，不提供整个 Map 的同一时刻快照；并发时的 size 可用于观测，不能用作“检查空位后插入”的正确性依据。它禁止 null 键和值，让 null 能明确表示当前无映射。

---

## 03 扩容期间怎么找数据

普通写会验证桶头仍然是加锁前看到的节点，避免锁住已经被迁移的旧桶。迁移完成的桶放置 ForwardingNode，访问路径可转向新表，部分写线程参与迁移。树桶还涉及 TreeBin 的读写协调，不能把“所有操作只锁普通头节点”理解成精确源码全景。

---

## 04 面试边界与参考

先解释数据结构如何安全发布，再解释空桶 CAS、非空桶协调和协助扩容，最后明确“单操作安全不等于多步骤事务”。

- [ConcurrentHashMap API：一致性与原子复合操作](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)

---

## 05 相关问题与延伸

- [[八股/01-Java/02-Java集合/01-HashMap 底层数据结构是什么？如何扩容|HashMap 底层数据结构是什么？如何扩容]]：非并发HashMap与并发容器的不同保证

---

## 06 所属专题

- [[八股/01-Java/02-Java集合/00-Java集合导航|Java集合导航]]
