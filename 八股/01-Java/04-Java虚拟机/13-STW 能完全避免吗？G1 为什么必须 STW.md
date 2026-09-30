---
aliases:
- "STW 能完全避免吗？G1 为什么必须 STW？"
- "Java 4.13 STW 能完全避免吗？G1 为什么必须 STW？"
---

# 13 STW 能完全避免吗？G1 为什么必须 STW？

## 01 核心回答

**不能完全避免。**即使是低停顿收集器，也有根扫描、重标记等阶段需要短暂停顿。工程目标不是"零 STW"，而是把停顿压到业务可接受范围并保持可预测。

**G1 哪些阶段必须 STW：**① **Concurrent Start 所附带的年轻代暂停及其初始标记工作**；② **Young/Mixed 回收中的对象复制（Evacuation Pause）**；③ Remark 和 Cleanup 暂停；注意 Concurrent Root Region Scan 本身是并发阶段。

**为什么必须停：**GC 线程搬对象、改引用，业务线程同时读写对象——不停顿做关键切换会出现"对象被搬走但引用还没统一修正"的不一致。STW 的作用是短时间内拿到安全一致的内存视图。

**G1 的目标：**不是"无停顿"而是"**可控停顿**"——通过 Region 回收和回收集预测尽量让停顿贴近目标，不能保证每次都在目标内（MaxGCPauseMillis），而不是像传统 Full GC 一次长暂停。

**追问：**ZGC 怎么做到几乎无 STW（并发转移 + 读屏障，见 4.10）；哪些收集器停顿可预测（G1/ZGC vs Parallel）。

---

## 02 为什么不是所有 GC 都必须同样停

“移动对象必然全程 STW”不是普遍定律。G1 的疏散设计在暂停内搬迁并修正相关引用，而 ZGC 用屏障与并发重定位协调应用访问，所以能把更多工作移出暂停。比较时应说明具体收集器机制，不能把 G1 的实现限制当成理论不可能。

STW 也不只来自 GC；安全点操作、去优化或诊断活动都可能影响停顿。对当前主流收集器，应回答仍存在短同步阶段，但“所有未来实现都绝不可能零 STW”不是规范承诺。

## 03 校正依据

本次纠正把 Concurrent Root Region Scan 归成 STW 的误导，保留根处理需要一致性视图的直觉。

- [G1 官方周期：Concurrent Start、Root Region Scan、Remark、Cleanup](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html)
- [[八股/01-Java/04-Java虚拟机/10-ZGC 是什么？为什么停顿低|ZGC 如何并发搬迁对象]]

## 04 所属专题

- [[八股/01-Java/04-Java虚拟机/00-Java虚拟机导航|Java虚拟机导航]]
