---
aliases:
- "ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化？"
- "Java 1.3 ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化？"
---

# 03 ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化？

## 01 核心回答


**ArrayList（动态数组）**

底层 Object []，随机访问 `O(1)`；尾部插入摊还 `O(1)`（扩容 1.5 倍）；**中间插入/删除 O(n)**（元素搬移）。

**代价：** 扩容时复制数组；删除后不缩容。

**LinkedList（双向链表）**

头尾操作 `O(1)`；**中间插入定位 O(n)**（要遍历找节点），找到后插入 O(1)。

**代价：** 每个节点存前后指针，内存占用高；随机访问 O(n)；缓存不友好（节点分散）。

**中间插入优化：
① 已有迭代器定位且连续插删时可考虑 **LinkedList**；按随机下标插入仍需 O(n) 定位。**CopyOnWriteArrayList** 不能作为中间插入或写入性能优化；
② 批量插入用 `addAll`（一次搬移）；
③ 根据语义换结构：双端队列可用 `ArrayDeque`，按键有序可考虑树/跳表；二者都不是通用按下标插入列表的替代；
④ 先收集到临时列表再整体合并，避免逐条 insert。最终应结合随机访问、位置是否已知、插删分布和内存成本选择，并以实际吞吐测试为准。

**面试追问：**为什么实际项目 ArrayList 用得更多（随机访问 + 缓存友好 + 尾部追加为主）；LinkedList 真的适合队列吗（ArrayDeque 更优，LinkedList 有节点开销）；ArrayList 扩容机制（1.5 倍 + Arrays.copyOf）。

---

## 02 为什么链表未必比数组插入快

复杂度必须把“寻找位置”和“修改结构”一起算。ArrayList 在索引 i 插入需要移动后面 n-i 个引用，可能还要扩容；LinkedList 按索引定位同样线性，并额外分配节点。连续内存复制通常有较好局部性，链表指针跳转和 GC 成本可能抵消常数次连边的优势。队列场景优先考虑 `ArrayDeque`，因为其头尾操作语义更匹配且没有链表节点开销。

批量 addAll 能将多次尾部搬移合并成一次。ensureCapacity 只减少扩容次数，不消除中间插入的搬移。1.5 倍是常见 OpenJDK 增长策略，不是 ArrayList API 的永久承诺；超过容量上限或批量需求更大时还要处理最小容量。

---

- [ArrayList API：复杂度与容量](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html)
- [CopyOnWriteArrayList API：写时复制的适用条件](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html)

- [[八股/01-Java/01-Java基础/04-Java 泛型为什么使用类型擦除？extends 和 super 怎么选|集合接口的泛型读写约束]]

---

## 03 相关问题与延伸

- [[八股/01-Java/02-Java集合/04-哪些集合是线程安全的？怎么把 ArrayList 变线程安全？说说 CopyOnWriteArrayList|哪些集合是线程安全的？怎么把 ArrayList 变线程安全？说说 CopyOnWriteArrayList]]：列表选择与线程安全代价

---

## 04 所属专题

- [[八股/01-Java/02-Java集合/00-Java集合导航|Java集合导航]]
