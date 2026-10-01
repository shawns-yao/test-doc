---
aliases:
- "OOM 定位的第一步是什么？"
- "Java 4.14 OOM 定位的第一步是什么？"
---

# 14 OOM 定位的第一步是什么？

## 01 核心回答

**第一步：先判 OOM 类型**——不同类型抓证据和排查工具完全不同，判错方向会白费时间（Direct buffer 问题不能只看 heap dump；堆中的包装对象仍可能提供持有链线索）。

**Java heap space**
堆问题 → HeapDump + GC 日志（MAT 看 Dominator Tree/引用链）。

**GC overhead limit**
堆快满且回收效率极差 → 同堆链路。

**Metaspace**
类元数据区 → 看类加载量/ClassLoader 统计（类加载器泄漏）。

**Direct buffer memory**
堆外直接内存 → NMT/Netty buffer、MaxDirectMemorySize。

**unable to create native thread**
线程数/系统资源 → jstack 线程数、ulimit、-Xss。

**最后归类：**泄漏（对象不释放）/ 突增（流量任务峰值）/ 配置不合理（堆、元空间、线程栈、直接内存参数太小）。

**一句话：**OOM 定位不是先调参数，而是先"判类型"，再用对应证据链定位是泄漏、突增还是配置问题。（完整命令链见 4.6）

---

## 02 第一份证据应该包含什么

记录原始错误全文、时间、JDK/收集器、启动参数、容器限额、进程 RSS 与堆占用，并核对是否由内核/容器直接杀死。Java heap space、Direct buffer memory、Metaspace 和无法创建 native 线程对应不同预算，不要先统一加 Xmx；容器内盲目扩大堆反而可能挤压 native 内存。

NMT 主要跟踪 JVM 内部的 native 分配，不覆盖所有第三方 native 库；必须与系统内存地图、BufferPool 指标和具体框架指标联合判断。heap dump 无法看到全部堆外数据，但包装器数量及引用链能帮助定位谁长期持有缓冲区。

---

## 03 下一步与参考

先确定哪类预算耗尽，再判断持续保留、瞬时峰值、资源配额或实际容量不足；完整工具链见 [[八股/01-Java/04-Java虚拟机/06-线上遇到过 OOM 吗？怎么排查？topjpsjstackjmap 各能看什么|OOM 分类型取证流程]]。

- [Oracle NMT 的覆盖边界](https://docs.oracle.com/en/java/javase/21/vm/native-memory-tracking.html)

---

## 04 所属专题

- [[八股/01-Java/04-Java虚拟机/00-Java虚拟机导航|Java虚拟机导航]]
