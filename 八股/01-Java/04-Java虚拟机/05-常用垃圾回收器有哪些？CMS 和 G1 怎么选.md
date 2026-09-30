---
aliases:
- "常用垃圾回收器有哪些？CMS 和 G1 怎么选？"
- "Java 4.5 常用垃圾回收器有哪些？CMS 和 G1 怎么选？"
---

# 05 常用垃圾回收器有哪些？CMS 和 G1 怎么选？

## 01 核心回答


**常用收集器：**① **Serial/ParNew**：单线程/多线程新生代复制，STW；② **Parallel Scavenge + Parallel Old**：吞吐优先（默认 JDK 8），多线程并行回收；③ **CMS**：老年代并发标记清除，追求低停顿（标记-清除，有碎片）；④ **G1**：JDK 9+ 默认，**分区域（Region）**化，可预测停顿（-XX:MaxGCPauseMillis），整体标记-整理 + 局部复制，兼顾吞吐与延迟；⑤ **ZGC/Shenandoah**：超低停顿（毫秒级），大堆场景。

**CMS vs G1：**① CMS 基于标记-清除（碎片化，要配合 Full GC 整理），G1 通过疏散压缩常规 Region，但 Humongous 对象仍可能遇到连续 Region 不足及尾部浪费；② CMS 并发标记用**增量更新**，G1 用**原始快照（SATB）**——都为了处理并发标记期间对象引用变化；③ G1 按历史成本和停顿目标选择回收集合，但该目标不是硬实时上限；④ 堆规模、分配率、存活率和延迟要求共同决定收益，不能仅用 4–8G 作为分界；⑤ JDK 9 起 CMS 废弃，JDK 14 移除——新项目直接 G1。

**怎么选：**追求吞吐（离线计算）→ Parallel；追求低延迟（在线服务）→ G1（默认）；超大堆 + 毫秒级停顿 → ZGC；选型后要压测验证停顿时间与吞吐。

**面试追问：**增量更新和原始快照的区别（记录新增引用 vs 记录删除引用）；G1 的 Remembered Set 是什么（记录 Region 间引用，避免全堆扫描）；CMS 为什么有碎片问题（标记-清除不清整）。

---

## 02 面试先说明版本与指标

CMS/ParNew 是历史对比，JDK 21 不再提供 CMS。G1 常作为在线服务基线；Parallel 更强调吞吐；ZGC 值得在严格尾延迟需求下验证。Shenandoah 的可用性依赖所用发行版和构建，不应默认所有 JDK 包都包含它。

比较要固定业务负载、CPU 配额和总内存预算，观察吞吐、请求 P99/P999、分配停顿、GC CPU 和存活集。更低 GC pause 不一定等于更低端到端延迟，因为并发回收会竞争 CPU，还可能发生分配等待。

## 03 校正与参考

本次保留 CMS 历史知识，修正 G1 无碎片、停顿硬保证和固定堆阈值。

- [Oracle：可用收集器与选择](https://docs.oracle.com/en/java/javase/21/gctuning/available-collectors.html)
- [Oracle：已移除的组件与 CMS](https://docs.oracle.com/en/java/javase/21/migrate/removed-tools-and-components.html)

## 04 相关问题与延伸

- [[八股/01-Java/04-Java虚拟机/08-G1 的核心思想是什么|G1 的核心思想是什么]]：收集器选型与G1机制

## 05 所属专题

- [[八股/01-Java/04-Java虚拟机/00-Java虚拟机导航|Java虚拟机导航]]
