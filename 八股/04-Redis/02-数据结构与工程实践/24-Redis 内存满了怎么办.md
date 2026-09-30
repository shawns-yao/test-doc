---
aliases:
- "Redis 内存满了怎么办？"
- "Redis 2.24 Redis 内存满了怎么办？"
---

# 24 Redis 内存满了怎么办？

## 01 核心回答


**先确认原因：**是 `used_memory` 达到 `maxmemory`，还是 RSS、内存碎片、持久化 fork 或宿主机整体内存不足。检查 `INFO memory`、`MEMORY STATS`、大 key、热 key、客户端缓冲区、复制缓冲区、AOF/RDB 重写和淘汰统计，不能只执行 `FLUSHDB`；线上保留变更前后的监控和采样，避免误删关键数据。

**缓存场景：**配置合适的 `maxmemory-policy`（按 TTL 或访问频率淘汰），给不同业务设置合理 TTL；清理无效 key、压缩 value、拆分大 key、限制单 key 列表和批量删除；可扩容、增加分片或把低频数据迁移到持久化存储，但扩容前评估复制、迁移和热点分布。

**非缓存场景：**保存任务、订单状态等不能随意淘汰的数据时，先暂停非关键写入、扩容或迁移，不能直接开启随机淘汰。检查碎片率、fork 期间写时复制额外内存、AOF 重写和备份空间，必要时错峰执行。

长期治理要建立 key TTL、容量预算、增长趋势、淘汰命中、写入拒绝和恢复演练的监控闭环。

## 02 理解机制与追问

maxmemory 是淘汰控制阈值，不等于进程 RSS 的绝对上限。分配器碎片、复制/AOF 缓冲、fork 子进程和 COW 都需预留空间。先观察增长来自键数量、值大小、缓冲还是碎片；减少业务数据后 RSS 也未必立即等比例下降。

## 03 易错点与适用边界

noeviction 通常让需要新增内存的写命令报错，读和删除仍可继续，并非所有命令全部不可用。volatile-* 只考虑有过期时间的键，可能没有足够候选而拒写；allkeys-* 会影响所有候选数据，不应对不可重建状态随意开启。过期删除与内存淘汰是两套机制。

## 04 面试口述

先区分数据量达到限制和进程总体内存压力，再按数据是否可丢决定淘汰、限写或扩容。设置 maxmemory 时必须为运行开销与持久化峰值留余量。

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/30-大 key 的风险与治理|大 key 治理]]
- [[八股/04-Redis/02-数据结构与工程实践/23-Redis 的持久化机制有哪些|持久化开销]]

## 06 官方参考

- [Redis 淘汰策略](https://redis.io/docs/latest/develop/reference/eviction/)
- [Redis RDB 与 AOF 持久化](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [Redis 紧凑编码与版本](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
