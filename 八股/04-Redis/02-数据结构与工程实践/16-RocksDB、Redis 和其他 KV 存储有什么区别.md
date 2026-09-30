---
aliases:
- "RocksDB、Redis 和其他 KV 存储有什么区别？"
- "Redis 2.16 RocksDB、Redis 和其他 KV 存储有什么区别？"
---

# 16 RocksDB、Redis 和其他 KV 存储有什么区别？

## 01 核心回答


**RocksDB**

嵌入式 KV 引擎，LSM Tree：写入先落 WAL 和 MemTable，再刷成 SSTable。适合本地状态、写密集和需要磁盘容量的场景。

**代价：**应用需自己管理进程生命周期、键编码、二级索引、备份、服务化和访问权限；关注 Compaction、写放大、读放大和 Block Cache。

**Redis 与其他 KV**

Redis 是独立的内存优先服务，提供丰富数据结构、过期、网络协议、复制和低延迟访问，适合缓存、计数、会话、排行榜和在线状态。

其他 KV 可能基于 B+ 树、LSM Tree、列族或共识协议，分别在范围查询、写吞吐、强一致、容量、扩容和运维成本上有不同取舍。

选择要看读写比例、点查与范围查询、数据规模、延迟目标、事务、复制、故障恢复、成本和团队能力。不能只比较单次 `GET` 的 QPS，还要验证升级、备份、恢复、节点故障和数据校验流程。

## 02 理解机制与追问

比较分三层：RocksDB 是嵌入式有序 KV 引擎；Redis 是带网络与数据结构语义的服务；基于 RocksDB 的分布式数据库还叠加路由、复制和共识。它们未必互斥。RocksDB 有事务接口、批写和快照，Redis 有事务/脚本但错误回滚语义不同。

## 03 易错点与适用边界

不能用“磁盘所以慢、内存所以快”替代测量。RocksDB Block Cache 命中时也可在内存完成读取，Redis 持久化、复制、大 key 操作也会增加延迟。压测须达到 compaction 稳态并覆盖重启与故障，不只报告短时间热缓存 GET 吞吐。

## 04 面试口述

我先对齐引擎与数据库服务的层次，再按访问模式、容量、事务和恢复能力比较；单命令峰值不足以决定实际选型。

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/22-NoSQL 和 KV 存储有什么区别？各自适合什么场景|NoSQL 和 KV 存储有什么区别？各自适合什么场景]]

- [[八股/04-Redis/02-数据结构与工程实践/15-基于 Redis 协议的数据库了解吗|Redis 协议兼容]]
- [[八股/04-Redis/02-数据结构与工程实践/02-Redis 和 MySQL 的区别是什么|Redis 与 MySQL]]

## 06 官方参考

- [RocksDB 官方架构](https://github.com/facebook/rocksdb/wiki/RocksDB-Overview)
- [Apache Kvrocks 官方介绍](https://kvrocks.apache.org/)
- [Redis RDB 与 AOF 持久化](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
