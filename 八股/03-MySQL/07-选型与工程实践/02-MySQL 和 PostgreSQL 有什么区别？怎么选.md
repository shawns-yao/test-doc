---
aliases:
- "MySQL 和 PostgreSQL 有什么区别？怎么选？"
- "Mysql 8.2 MySQL 和 PostgreSQL 有什么区别？怎么选？"
---

# 02 MySQL 和 PostgreSQL 有什么区别？怎么选？

## 01 核心回答

**MySQL**

工程化和生态成熟，常见 OLTP 选型；是否适用取决于需求与团队经验。

**PostgreSQL**

功能"全能型"：高级 SQL、窗口分析、JSONB、数组/范围类型、PostGIS、丰富索引（GIN/GiST/BRIN/表达式/部分索引）；两者都支持 MVCC。

**怎么选：**高并发交易 + 常规 CRUD + 团队经验在 MySQL → MySQL；复杂报表/复杂 SQL、地理空间、半结构化数据、强约束需求 → PostgreSQL；一种可选组合是：现有核心交易用 MySQL，地理/复杂建模模块按需求评估 PostgreSQL（不是通用公司实践定论）。

**追问：**PG 的 JSONB 比 MySQL JSON 好在哪（索引 + 操作符）；两边复制差异（MySQL binlog 复制与 PG WAL/逻辑复制；同步性另看配置）；迁移成本怎么评估（SQL 兼容性 + 运维体系）。

## 02 理解机制与追问

按需求维度比较比品牌印象更有用：事务隔离默认与实现、索引类型、JSON 查询和索引表达方式、扩展生态、复制与恢复、团队运维经验。PostgreSQL 的 GIN/GiST/BRIN 等对应不同访问需求；MySQL 同样支持事务、窗口函数和 JSON，不能把这些共有能力写成 PG 独有。

## 03 易错点与适用边界

修正复制对比：“异步为主 vs 流复制”混合了两个维度。流式描述传输方式，同步/异步描述确认语义；PostgreSQL 流复制也默认异步并可配置同步。MySQL 常见 binlog 复制与 PG 物理 WAL 流复制的内容层次不同，PG 还支持逻辑复制。不存在无需负载证据的统一高并发赢家。

## 04 面试口述

两者都能承担成熟 OLTP。复杂数据类型、索引或扩展需要时重点看 PG，已有 MySQL 体系则核对其是否满足需求；最终比较迁移代价和真实负载，不能只背“简单用 MySQL、复杂用 PG”。

## 05 关联问题

- [[八股/03-MySQL/07-选型与工程实践/01-OLTP、OLAP、HTAP 常用数据库有哪些？怎么选型|工作负载选型]]

## 06 官方参考

- [PostgreSQL 索引类型](https://www.postgresql.org/docs/current/indexes-types.html)
- [PostgreSQL 高可用与复制](https://www.postgresql.org/docs/current/high-availability.html)
- [MySQL 8.4 复制线程](https://dev.mysql.com/doc/refman/8.4/en/replication-threads.html)

## 07 所属专题

- [[八股/03-MySQL/07-选型与工程实践/00-选型与工程实践导航|选型与工程实践导航]]
