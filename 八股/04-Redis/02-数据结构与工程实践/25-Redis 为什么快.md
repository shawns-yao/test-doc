---
aliases:
- "Redis 为什么快？"
- "Redis 2.25 Redis 为什么快？"
---

# 25 Redis 为什么快？

## 01 核心回答

**基于内存**
常见数据操作主要访问内存；持久化、复制和系统缺页仍可能产生 I/O。

**单线程事件循环**
命令串行执行，减少数据结构并发锁竞争，但不消除系统调度与后台线程。

**高效数据结构**
哈希表、跳表、紧凑结构（现代版本多用 listpack，ziplist 属于旧版语境）针对场景优化。

**I/O 多路复用**
epoll 事件驱动，单线程扛海量连接；pipeline 减少 RTT。

**追问：**单线程不怕慢命令吗（怕，大 key 长操作或长脚本会拖住执行路径；BLPOP 等通常只等待调用客户端）；为什么不用多线程（CPU、网络和内存均可能成为瓶颈；网络线程和后台任务按版本配置核实）。

---

## 02 理解机制与追问

快来自常用操作以内存数据结构为主、执行路径较短、多路复用减少连接管理成本，以及批量命令/pipeline 摊薄 RTT。数据结构按小对象紧凑编码与大集合查询需求优化，避免每次都做复杂 SQL 规划。性能瓶颈仍可落在 CPU、内存带宽、网络或慢命令上。

---

## 03 易错点与适用边界

修正三个绝对说法：持久化/复制/缺页可产生 I/O，不能说完全无磁盘；操作系统仍会调度线程，不能说无上下文切换；Redis 并非整个进程只有一个线程。普通数据命令以串行执行语义为主要模型，网络线程与后台任务能力随版本变化，不应把 Redis 6 的介绍固定当成所有新版本结论。

---

## 04 面试口述

Redis 低延迟主要靠内存、短执行路径和高效网络模型，但大集合、长脚本、fork 和磁盘压力都可能破坏尾延迟。是否快必须用同负载和持久化配置测量。

---

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/13-Redis 网络编程了解吗|网络模型]]
- [[八股/04-Redis/02-数据结构与工程实践/30-大 key 的风险与治理|大 key 风险]]

---

## 06 官方参考

- [Redis 延迟诊断](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency/)
- [Redis 紧凑编码与版本](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)
- [Redis RDB 与 AOF 持久化](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)

---

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
