---
aliases:
- "Redis 和 MySQL 的区别是什么？"
- "Redis 2.2 Redis 和 MySQL 的区别是什么？"
---

# 02 Redis 和 MySQL 的区别是什么？

## 01 核心回答


**Redis**

**定位：**内存优先的键值数据库，单次访问路径短。

**能力：**String、Hash、List、Set、Sorted Set、Bitmap、Stream 等数据结构；支持 RDB、AOF 持久化。

**适用：**缓存、计数、排行榜、会话、限流、分布式锁和临时状态。

**限制：**容量和成本受内存限制，复杂关系查询、Join、约束和强事务能力不如 MySQL。

**MySQL**

**定位：**关系型数据库，以表、行、列和 SQL 为核心。

**能力：**事务、ACID、索引、Join、约束、权限和复杂查询。

**适用：**订单、账户、商品等核心事实数据的持久化来源。

**成本：**磁盘 I/O、锁、日志和查询优化，单次访问通常比 Redis 更重。

实际系统常让 MySQL 保存最终事实，Redis 承担缓存和高并发读写。使用 Redis 时要设计过期、淘汰、持久化、故障恢复和缓存一致性；使用 MySQL 时要关注索引、事务边界、慢查询、连接池和备份，不能简单地用一个替代另一个。

---

## 02 理解机制与追问

比较要对齐数据模型与恢复目标。Redis 的简单原子数据结构操作路径短，MySQL 的索引和 Buffer Pool 也可能使查询主要在内存完成；差别不等于“一个内存，一个每次都磁盘”。Redis 支持持久化和复制，但事务错误、隔离与恢复语义不同。

---

## 03 易错点与适用边界

不能从 Redis 适合缓存推导出它只能做缓存，也不能从支持 AOF 推导出任何配置都适合作为唯一事实源。使用 Redis 存不可重建数据时，需要有明确 RPO/RTO、淘汰策略、备份和恢复演练。跨 Redis/MySQL 写入不会自动共享事务。

---

## 04 面试口述

MySQL 常承载关系事实和事务约束，Redis 常承载数据结构化的低延迟访问。按模型、一致性和故障恢复选型，不能只按单次 GET 与 SQL 的耗时比较。

---

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/16-RocksDB、Redis 和其他 KV 存储有什么区别|RocksDB、Redis 和其他 KV 存储有什么区别]]

- [[八股/04-Redis/02-数据结构与工程实践/04-Redis 事务和 MySQL 事务有什么区别|事务语义差异]]
- [[八股/04-Redis/02-数据结构与工程实践/32-为什么不把所有查询都塞到 Redis|哪些查询值得缓存]]

---

## 06 官方参考

- [Redis 事务与错误处理](https://redis.io/docs/latest/develop/using-commands/transactions/)
- [Redis RDB 与 AOF 持久化](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [Redis 紧凑编码与版本](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)

---

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
