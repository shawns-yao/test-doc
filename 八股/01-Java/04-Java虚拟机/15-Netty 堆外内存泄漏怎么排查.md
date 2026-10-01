---
aliases:
- "Netty 堆外内存泄漏怎么排查？"
- "Java 4.15 Netty 堆外内存泄漏怎么排查？"
---

# 15 Netty 堆外内存泄漏怎么排查？

## 01 核心回答

**概念原理：**`ByteBuffer.allocateDirect()` 是直接内存的一种分配方式；Netty 分配器还可能使用其他 native 分配路径——网络/文件 I/O 时数据本来就会进入堆外区域，直接用堆外可减少一次从堆内到堆外的拷贝；`-XX:MaxDirectMemorySize` 不是所有进程堆外内存的总上限，Netty 的约束还取决于分配器与配置，DirectByteBuffer 失去引用后 GC 时可能回收，**堆外持续泄漏也会触发 OOM**。

**排查流程：**① 先看 direct memory 指标和 Full GC 行为；② 启用泄漏检测（`ResourceLeakDetector`）定位未释放 ByteBuf；③ 排查重点是**引用计数对象是否成对 release**，以及异常分支是否遗漏释放逻辑。

**面试追问：**线上开启最高级泄漏检测的代价是什么？

---

> 返回导航：[[八股/00-总导航|00-总导航]]

## 02 先判断池保留还是未释放

池化 ByteBuf 释放后常归还 arena/chunk 供复用，不一定立即降低 RSS。应联合 usedDirectMemory、活跃分配、流量退潮后的基线和 allocator 指标，不能只凭 RSS 不下降判定泄漏。

Netty 用引用计数管理 ByteBuf。每次 retain 要有对应 ownership 和最终 release；slice/duplicate 共享底层内容且通常不会自动增加引用计数，跨异步边界要正确保留。不要在向下游转交消息后又重复释放，也要检查异常、取消和超时路径。SimpleChannelInboundHandler 等框架可能自动释放，先确认谁是最后消费者。

---

## 03 泄漏检测如何使用

SIMPLE 抽样检测，ADVANCED 额外记录访问轨迹，PARANOID 检测每个分配、代价最高，适合测试或受控诊断窗口。泄漏日志帮助找到分配/访问位置，不一定直接给出漏 release 的行；未被 GC 触达的长期强引用对象也可能尚未产生报告。修复后应重复相同压测并验证流量峰值回落后的基线，不能以“没有报日志”单独证明无泄漏。

参考：
- [Netty 引用计数所有权、派生缓冲区与泄漏排查](https://netty.io/wiki/reference-counted-objects.html)
- [ResourceLeakDetector 级别](https://netty.io/4.1/api/io/netty/util/ResourceLeakDetector.Level.html)

---

## 04 相关问题与延伸

- [[八股/01-Java/04-Java虚拟机/02-Java 内存泄漏的常见原因有哪些？如何排查|Java 内存泄漏的常见原因有哪些？如何排查]]：堆外引用计数与普通内存泄漏的区别

---

## 05 所属专题

- [[八股/01-Java/04-Java虚拟机/00-Java虚拟机导航|Java虚拟机导航]]
