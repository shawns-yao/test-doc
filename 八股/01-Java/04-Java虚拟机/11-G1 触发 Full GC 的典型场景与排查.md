---
aliases:
- "G1 触发 Full GC 的典型场景与排查？"
- "Java 4.11 G1 触发 Full GC 的典型场景与排查？"
---

# 11 G1 触发 Full GC 的典型场景与排查？

## 01 核心回答

**to-space exhausted**
复制存活对象时没有足够可用 Region（evacuation failure）。

**老年代增长太快**
并发标记没完成就快打满（回收跟不上分配）。

**Humongous 大对象**
超过 Region 一半的对象申请失败或碎片严重。

**Metaspace 压力**
元数据分配压力可能请求 GC/类卸载，是否进入 Full GC 需看 GC cause 与后续分配结果。

**优化方向：**先看 Full GC 前后**老年代占用曲线和分配速率**（不只看停顿时间）；加堆/预留空间（`-Xmx`、`G1ReservePercent`）；避免大对象（必要时调 `G1HeapRegionSize`）；限制显式 GC（`DisableExplicitGC`）；查内存泄漏和晋升压力。

**追问：**To-space exhausted 和晋升失败的关系（复制没空间）；大对象为什么触发 Full GC（Humongous 直接进老年代占大块）；DisableExplicitGC 有风险吗（System.gc 失效，RMI 等依赖的要小心）。

---

## 02 先建立时间线，再调参数

连续查看 Full GC 之前的并发周期是否及时开始/结束、Old/Humongous 增长、空闲 Region 和 evacuation failure。to-space exhausted 是复制目标空间不足的信号，可能导致后续 Full GC，但不能说出现该行就必然当场 Full GC。

若长期活跃集接近上限，提前标记也回收不出足够空间；若短时分配峰值太陡，应限并发、削峰或增加余量；若大量 Humongous 对象，则优先减少大对象和确认连续 Region。显式 System.gc、诊断操作也可能请求 Full GC，需核对调用来源。

## 03 参数取舍与依据

扩大 G1ReservePercent 会保留更多疏散余量，同时压缩正常可用容量；调整 IHOP 要先理解自适应行为，不能只套固定百分比。G1HeapRegionSize 的变化同时影响大对象分类和回收粒度。一次只改有证据支持的因素并回放同负载。

- [G1 调优：Full GC、疏散失败和 Humongous](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-garbage-collector-tuning.html)

## 04 相关问题与延伸

- [[八股/01-Java/04-Java虚拟机/08-G1 的核心思想是什么|G1 的核心思想是什么]]：分区回收策略及退化条件
- [[八股/01-Java/04-Java虚拟机/12-GC 日志快速看什么|GC 日志快速看什么]]：Full GC原因与日志证据

## 05 所属专题

- [[八股/01-Java/04-Java虚拟机/00-Java虚拟机导航|Java虚拟机导航]]
