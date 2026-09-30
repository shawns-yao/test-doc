---
aliases:
- "MySQL 除了 `redo log` 还有哪些日志？`redo log` 中保存什么内容？"
- "Mysql 5.6 MySQL 除了 `redo log` 还有哪些日志？`redo log` 中保存什么内容？"
---

# 06 MySQL 除了 `redo log` 还有哪些日志？`redo log` 中保存什么内容？

## 01 核心回答


redo log

InnoDB 层的物理或偏物理恢复日志，记录数据页发生的修改及恢复所需信息。受容量限制并可复用的日志文件集合（8.4 可动态调整容量），用于 **WAL 和崩溃恢复**——解决已提交事务的数据页还没来得及刷盘时如何恢复。

undo log

保存修改前的逻辑信息，用于事务回滚和 MVCC 读取历史版本；清理受活跃事务影响。

binlog

Server 层的二进制归档日志，按事件记录提交后的数据变更（statement/row/mixed 格式），用于主从复制、数据恢复和 CDC。

慢查询日志

记录超过阈值或未使用索引等慢 SQL，用于性能分析；通用查询日志更完整但开销更高。

错误/中继日志

错误日志记录启动、停止、崩溃、恢复和关键错误；中继日志由复制从库接收主库 binlog 后保存，用于后续重放。

`redo log` 保存的不是完整 SQL 文本，而是**存储引擎层面用于重做页面修改的信息**；`binlog` 也不是用来替代 `undo log` 的。事务提交时要协调 `redo log` 和 `binlog`，恢复和复制分别使用不同日志完成各自职责。

**面试追问：**redo log 和 binlog 谁先写（两阶段提交）；为什么 redo 是物理日志（记录页修改，恢复快）；binlog 能替代 redo 吗（不能；但可参与两阶段提交的恢复决策）；redo 和 undo 的分工（redo 管已提交不丢=持久性，undo 管未提交回滚=原子性 + 历史版本读=隔离性）。

## 02 理解机制与追问

redo 面向存储引擎恢复页面修改，undo 支持回滚和历史版本，binlog 面向提交事件与复制，relay log 是副本接收后的待应用日志。不同日志的“写入缓冲区”“写到文件系统”“持久化介质”不是同一时刻。检查问题时要说明到底缺了哪一层保障。

## 03 易错点与适用边界

版本修正：MySQL 8.4 使用 innodb_redo_log_capacity 管理 redo 容量，可动态调整，不应把“固定大小循环文件”理解成永久固定容量或旧版 ib_logfile0/1 布局。redo 可包含未提交事务的页面修改。binlog 不能独立替代 InnoDB 页恢复，但恢复过程会用它协调内部两阶段提交，因此“binlog 完全不参与崩溃恢复”不准确。

## 04 面试口述

我按用途区分：redo 重做页面、undo 撤销和版本链、binlog 复制与时间点恢复、relay log 接收缓冲。日志是否落盘以及是否可以清理，要看各自配置和消费者进度。

## 05 关联问题

- [[八股/03-MySQL/04-事务与日志/04-MVCC 是什么？它如何实现一致性读|MVCC 是什么？它如何实现一致性读]]

- [[八股/03-MySQL/04-事务与日志/07-为什么有 binlog 还需要 redo log？两者分别解决什么问题|日志为什么不能互相替代]]
- [[八股/03-MySQL/04-事务与日志/08-什么情况下需要使用 binlog|binlog 使用场景]]

## 06 官方参考

- [MySQL 8.4 Redo Log](https://dev.mysql.com/doc/refman/8.4/en/innodb-redo-log.html)
- [MySQL 8.4 二进制日志参数](https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html)
- [MySQL 8.4 复制线程](https://dev.mysql.com/doc/refman/8.4/en/replication-threads.html)

## 07 所属专题

- [[八股/03-MySQL/04-事务与日志/00-事务与日志导航|事务与日志导航]]
