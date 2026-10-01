---
aliases:
- "哪些集合是线程安全的？怎么把 ArrayList 变线程安全？说说 CopyOnWriteArrayList。"
- "Java 1.4 哪些集合是线程安全的？怎么把 ArrayList 变线程安全？说说 CopyOnWriteArrayList。"
---

# 04 哪些集合是线程安全的？怎么把 ArrayList 变线程安全？说说 CopyOnWriteArrayList。

## 01 核心回答


**线程安全集合：**① **遗留类**：`Hashtable`、`Vector`（全方法 synchronized，性能差）；② **并发包**：`ConcurrentHashMap`、`CopyOnWriteArrayList`、`CopyOnWriteArraySet`、`ConcurrentLinkedQueue`、`BlockingQueue` 系列；③ **包装类**：`Collections.synchronizedList/synchronizedSet/synchronizedMap`（包装后方法加锁）。

**CopyOnWriteArrayList 原理（写时复制）：**写操作（add/set/remove）先**复制一份新数组**，在副本上修改，然后用 `volatile` 数组引用替换旧数组；**读操作完全不加锁**（直接读当前数组引用）。适合**读多写少**场景（如监听器列表、配置缓存）。

**关键细节：**迭代器是**快照式**（遍历开始时的数组），迭代期间其他线程的修改不可见——固定数组快照语义（应与 ConcurrentHashMap 的弱一致遍历区分）；写操作 O(n) 复制成本高，写频繁场景性能差；元素不能为 null？可以（允许 null）。

**风险与取舍：**写多读少别用（每次写全量复制）；synchronizedList 读写都加锁（读并发差）；选型：读多写少 → COW，写多时按 Map/List/Queue 语义选择相应并发结构或外部锁。

**面试追问：**COW 的读操作有没有锁（没有）；和 synchronizedList 的区别（读无锁 vs 读加锁）；为什么迭代器不抛 ConcurrentModificationException（快照迭代）。

---

## 02 包装后为什么遍历仍可能出错

synchronizedList 将单个方法调用串行化，但迭代器跨多次方法访问，遍历期间必须按 API 约定锁住包装后的列表，并要求所有线程都通过同一包装器操作。原 ArrayList 引用若仍被旁路修改，包装器保护就失效。contains 后 add 也不是天然原子；需要同一锁或专用原子 API。

COW 写线程相互协调后发布新数组，读线程看到的是某个完整数组版本。旧迭代器持有旧数组直到不再使用，因此大数组、长时间迭代和频繁写入会增加内存压力；快照只复制元素引用，不把元素深拷贝成不可变对象。其迭代器不支持 remove/set/add。

---

## 03 面试口述版与参考

“线程安全”要分别回答单方法、复合操作、迭代和元素对象。读远多于写且容忍迭代快照时选 COW；强一致多步骤操作需要明确锁范围。没有哪一种并发 Map 可以不改变语义地替代 List。

- [Collections.synchronizedList：遍历同步要求](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html#synchronizedList(java.util.List))
- [CopyOnWriteArrayList：快照迭代](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html)

---

## 04 相关问题与延伸

- [[八股/01-Java/02-Java集合/03-ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化|ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化]]：列表选择与线程安全代价

---

## 05 所属专题

- [[八股/01-Java/02-Java集合/00-Java集合导航|Java集合导航]]
