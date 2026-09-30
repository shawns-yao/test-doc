---
aliases:
- "OLTP、OLAP、HTAP 常用数据库有哪些？怎么选型？"
- "Mysql 8.1 OLTP、OLAP、HTAP 常用数据库有哪些？怎么选型？"
---

# 01 OLTP、OLAP、HTAP 常用数据库有哪些？怎么选型？

## 01 核心回答

**OLTP（事务型）**

MySQL（成熟 OLTP 生态）、PostgreSQL（功能强）、TiDB（MySQL 协议 + 分布式扩展）、OceanBase（金融）、SQL Server/Oracle（传统企业）。

**OLAP（分析型）**

ClickHouse（实时报表/日志）、Doris/StarRocks（实时分析 + 明细查询）、Trino/Presto（联邦 SQL 查询引擎，非独立主存储）、Snowflake/BigQuery（云数仓）。

**HTAP（混合）**

TiDB（行列混合）、OceanBase、SingleStore；PostgreSQL + 扩展方案组合。

**选型原则：**交易核心链路优先 OLTP（一致性和延迟优先）；经营分析/报表优先 OLAP（扫描聚合性能优先）；既要实时写又要近实时分析才考虑 HTAP（评估成本和复杂度）。

**追问：**ClickHouse 为什么分析快（列式存储 + 向量化）；TiDB 怎么做到分布式事务（Raft + Percolator 模型）；HTAP 的代价（资源竞争 + 治理复杂）。

## 02 理解机制与追问

OLTP 优先处理短事务、点查、小范围更新；OLAP 优先吞吐式扫描和聚合；HTAP 试图让同一事实数据兼顾两者。TiDB 的 TiKV 行存与 TiFlash 列存副本说明“混合”通常需要不同存储/执行路径及一致性协调，不是把报表直接跑在交易主库上就完成。

## 03 易错点与适用边界

原列举中的 Trino/Presto 是分布式 SQL 查询引擎，不等同于拥有自身主存储的数据库。市场排名、行业标签不能作为性能证据。选型要补 RPO/RTO、事务边界、分析新鲜度、总成本、运维能力及代表性压测；HTAP 也需验证分析负载对交易尾延迟的影响。

## 04 面试口述

我先按交易延迟、分析扫描、新鲜度和故障恢复目标拆需求，再比较可用产品能力。协议兼容并不代表所有 SQL、事务与运维行为完全兼容。

## 05 关联问题

- [[八股/03-MySQL/03-存储引擎与索引结构/10-RocksDB 了解吗？它和 MySQL 的存储方式有什么不同|RocksDB 了解吗？它和 MySQL 的存储方式有什么不同]]

- [[八股/03-MySQL/07-选型与工程实践/02-MySQL 和 PostgreSQL 有什么区别？怎么选|MySQL 与 PostgreSQL]]
- [[八股/03-MySQL/06-高可用与扩展/05-跨分片 join、分页、排序、count 怎么处理|OLTP 不宜承载的跨片分析]]

## 06 官方参考

- [TiDB TiFlash 架构](https://docs.pingcap.com/tidb/stable/tiflash-overview/)
- [RocksDB 官方架构与事务能力](https://github.com/facebook/rocksdb/wiki/RocksDB-Overview)

## 07 所属专题

- [[八股/03-MySQL/07-选型与工程实践/00-选型与工程实践导航|选型与工程实践导航]]
