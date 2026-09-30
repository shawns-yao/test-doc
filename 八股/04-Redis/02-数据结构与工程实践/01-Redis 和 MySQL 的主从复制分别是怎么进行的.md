---
aliases:
- "Redis 和 MySQL 的主从复制分别是怎么进行的？"
- "Redis 2.1 Redis 和 MySQL 的主从复制分别是怎么进行的？"
---

# 01 Redis 和 MySQL 的主从复制分别是怎么进行的？

## 01 核心回答


**Redis 主从复制**

**首次同步：**副本主动连接主节点，主节点生成或复用 RDB 快照，副本加载后再接收复制积压缓冲区中的增量命令。

**持续复制：**同步完成后，主节点把后续写命令持续发送给副本，副本按顺序执行。

**断线与切换：**积压缓冲区包含断点命令时部分重同步，否则重新全量同步；Sentinel/Cluster 负责故障检测、选主和切换，但切换期间可能短暂丢写或读旧数据。

**MySQL 主从复制**

**链路：**主库把已提交事务写入 binlog，从库 I/O 线程拉取并写入 relay log，SQL 线程或并行复制线程读取重放。

**断点续传：**GTID 标识事务，简化断点续传和主从切换。

两者默认多为异步复制，主库提交成功不等于副本已经追平；更强一致性需求须设计具体协议；半同步、等待副本确认或读主只解决部分风险，并监控复制延迟、复制错误和数据校验结果。

## 02 理解机制与追问

Redis 部分重同步依赖复制 ID 与 offset 可衔接，且所需增量仍在 backlog；offset 相同但历史不同不能直接续传。全量同步时快照期间的新增写也要传给副本，不能只加载 RDB 就认为追平。MySQL 则把接收 relay log 和应用事务分成不同阶段。

## 03 易错点与适用边界

修正“半同步/等待确认即可强一致”：Redis WAIT 改善副本接收确认，不把系统变成共识型强一致数据库；MySQL 半同步也不保证副本已应用。Redis 主节点返回、AOF 持久化、副本接收、故障后保留是不同保证，必须分别说明。

## 04 面试口述

我会比较快照/增量、断点标识、接收与应用阶段，并说明两者异步复制下的丢失窗口。确认副本收到了，不等于任何副本立即可读新值。

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/28-Redis 哨兵机制是什么|Sentinel 切换]]
- [[八股/04-Redis/02-数据结构与工程实践/14-Redis 或数据库多个节点之间如何处理数据一致性|跨节点一致性]]

## 06 官方参考

- [Redis 复制机制](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)
- [Redis WAIT 复制确认](https://redis.io/docs/latest/commands/wait/)
- [MySQL 8.4 复制线程](https://dev.mysql.com/doc/refman/8.4/en/replication-threads.html)
- [MySQL 8.4 半同步确认边界](https://dev.mysql.com/doc/refman/8.4/en/replication-semisync.html)

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
