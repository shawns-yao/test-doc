---
aliases:
- "RocksDB 了解吗？它和 MySQL 的存储方式有什么不同？"
- "Mysql 3.10 RocksDB 了解吗？它和 MySQL 的存储方式有什么不同？"
---

# 10 RocksDB 了解吗？它和 MySQL 的存储方式有什么不同？

## 01 核心回答


**RocksDB**

嵌入式持久化 KV 引擎，应用通过库接口在本地进程中读写键值，不像 MySQL 天然提供独立数据库服务、SQL、关系模型和完整查询优化器。

主要采用 LSM Tree：写入先进 WAL 和 MemTable，再刷成 SSTable，后台 Compaction 合并文件，对高吞吐顺序写和大规模 KV 数据友好。

**代价：**应用需自己设计键编码、二级索引、数据模型、备份和服务化能力，关注 Compaction、写放大、Block Cache 和 SSTable 数量。

**MySQL（InnoDB）**

关系数据库：表、行、SQL、事务、二级索引、Join、权限和客户端连接协议；InnoDB 用 B+ 树组织索引页，Buffer Pool、Redo、Undo 和 MVCC 保证事务与恢复。

适合复杂条件查询、关系建模和强事务业务；维护关系约束和多个索引会增加写入成本。

RocksDB 更适合作为**本地状态存储、存储系统底层引擎、流处理状态后端或写密集型 KV 场景**；MySQL 更适合作为业务事实数据库。

**面试追问：**RocksDB 为什么适合做流处理状态后端（本地嵌入式 + 高写吞吐）；RocksDB 有 SQL 吗（没有，要自己建模）；什么场景选 RocksDB 而不是 MySQL（写密集 KV、嵌入场景）。

## 02 理解机制与追问

RocksDB 是库而 MySQL 是数据库服务，比较时先对齐层次：与 RocksDB 更接近的是 InnoDB 这类引擎，SQL、复制、权限、路由通常由更上层提供。RocksDB 有 WriteBatch、快照以及乐观/悲观事务接口，不能因它是 KV 就说它没有事务。

## 03 易错点与适用边界

WAL 可配置关闭，写入是否 fsync 也可配置，返回成功不自动等于断电不丢。快照和 iterator 还会延长旧版本/文件保留，必须释放。MySQL 发行版或扩展可以集成不同引擎，因此“RocksDB 与 MySQL”不必是互斥选择；本题主要比较 RocksDB 与 InnoDB。

## 04 面试口述

RocksDB 适合做嵌入式有序 KV 引擎，MySQL 提供完整关系数据库服务。选 RocksDB 要准备补上服务化与运维能力，并明确 WAL、同步写和事务配置。

## 05 关联问题

- [[八股/03-MySQL/03-存储引擎与索引结构/09-了解 LSM 树吗？它和 B+ 树有什么区别|LSM 机制]]
- [[八股/03-MySQL/07-选型与工程实践/01-OLTP、OLAP、HTAP 常用数据库有哪些？怎么选型|数据库选型维度]]

## 06 官方参考

- [RocksDB 官方架构与事务能力](https://github.com/facebook/rocksdb/wiki/RocksDB-Overview)

## 07 所属专题

- [[八股/03-MySQL/03-存储引擎与索引结构/00-存储引擎与索引结构导航|存储引擎与索引结构导航]]
