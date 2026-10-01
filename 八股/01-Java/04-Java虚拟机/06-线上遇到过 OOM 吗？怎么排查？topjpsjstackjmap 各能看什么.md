---
aliases:
- "线上遇到过 OOM 吗？怎么排查？top/jps/jstack/jmap 各能看什么？"
- "Java 4.6 线上遇到过 OOM 吗？怎么排查？top/jps/jstack/jmap 各能看什么？"
---

# 06 线上遇到过 OOM 吗？怎么排查？top/jps/jstack/jmap 各能看什么？

## 01 核心回答


**概念原理：**OOM 排查核心：**先看错误类型，再抓现场，最后用工具分析**。常见类型：`Java heap space`（堆满）、`Metaspace`（类元信息）、`GC overhead limit exceeded`（GC 白忙）、`Unable to create native thread`（线程数超限）。

**排查流程（命令链）：**

1. `top`：看进程 CPU/内存占用，确认哪个进程异常。

2. `jps -l`：列出 JVM 进程和主类，找到目标 PID。

3. `jmap -dump`：**OOM 前抓堆 dump**（或启动参数 `-XX:+HeapDumpOnOutOfMemoryError` 自动抓）。

4. `jstack`：看线程栈（线程卡死/死锁/大量等待）。

5. `jstat -gcutil`：看 GC 频率（是否频繁 Full GC）。

6. **MAT 分析 dump**：Dominator Tree 找大对象、Leak Suspects 找泄漏嫌疑、GC Roots 引用链定位持有者。

7. **结合代码修复**：静态集合、缓存无淘汰、连接未关、ThreadLocal 未 remove。

**关键细节：**无头服务器怎么定位（同命令链，远程 ssh + 工具）；dump 文件可能很大（先 `jmap -histo` 看对象统计再决定是否全量 dump）；OOM 后进程可能还在（GC overhead）也可能已退出——生产要配自动重启 + dump 保留；只背命令名没用，要能讲清每步解决什么问题。

**面试追问：**heap dump 和 thread dump 的区别（堆对象 vs 线程状态）；jmap -histo 看什么（对象类型统计/占用）；OOM 后第一件事做什么（保留现场：dump + 日志 + 指标快照）。

---

## 02 按错误类型选择现场

堆 OOM 关注存活对象与 GC；Metaspace 看类加载量和 ClassLoader；native thread 错误看线程数量、栈内存、进程/容器线程限制；容器 OOMKill 则先看退出原因和内存限制，可能根本没有 Java 异常或 heap dump。

jstat 是周期统计，GC 日志给出事件及耗时，heap dump 给出对象图，thread dump 给出线程等待。不能用线程 dump 证明对象泄漏，也不能用 heap dump 直接代表进程 RSS。建议以 jcmd help 确认目标 JVM 支持的诊断命令。

---

## 03 现场代价与面试表达

GC.heap_dump、对象直方图可能触发停顿或 GC，先核对磁盘空间、服务副本和诊断窗口；NMT 必须预先启用且不是所有 native 内存的总账。若没有亲自经历生产 OOM，应按“我的排查步骤”回答，不把演练或假设说成真实项目经历。

- [jcmd 命令：影响级别与诊断参数](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jcmd.html)
- [Oracle 诊断工具](https://docs.oracle.com/en/java/javase/21/troubleshoot/diagnostic-tools.html)

---

## 04 相关问题与延伸

- [[八股/01-Java/04-Java虚拟机/14-OOM 定位的第一步是什么|OOM 定位的第一步是什么]]：反向关联：此题引用了本题的机制或边界

---

## 05 所属专题

- [[八股/01-Java/04-Java虚拟机/00-Java虚拟机导航|Java虚拟机导航]]
