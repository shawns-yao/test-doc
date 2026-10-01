---
aliases:
- "MySQL 事务有哪些特性？ACID 分别如何实现？"
- "Mysql 5.1 MySQL 事务有哪些特性？ACID 分别如何实现？"
---

# 01 MySQL 事务有哪些特性？ACID 分别如何实现？

## 01 核心回答


原子性

事务中的多个操作要么全部成功，要么全部回滚。InnoDB 通过 `undo log` 保存修改前的信息，需要回滚时恢复旧值；单条语句报错是否导致整个事务回滚取决于错误类型。

一致性

事务执行前后都要满足约束和业务规则（主键、唯一键、外键、检查约束、库存不为负）。数据库的锁、日志和隔离机制提供基础保障，业务代码仍要负责跨服务规则和幂等。

隔离性

并发事务互相尽量不可见。InnoDB 通过锁、MVCC 和事务隔离级别控制读写冲突，按所选级别限制脏读、不可重复读和幻读等现象。

持久性

在强持久化配置及存储兑现刷盘承诺的前提下，已提交结果可在宕机后恢复。InnoDB 通过必要的 `redo log` 持久化保障恢复，重启后通过日志恢复脏页。

在复制场景中还要考虑 `binlog`：MySQL 通过内部**两阶段提交**协调 InnoDB 的 `redo log` 和 Server 层的 `binlog`，避免出现存储引擎已提交但复制日志没记录，或日志已记录但引擎没提交的不一致。ACID 是数据库能力和业务约束共同实现的，不是只依赖某一个日志文件。

**面试追问：**redo log 和 undo log 分别保证什么（持久性 vs 原子性）；两阶段提交协调的是什么（redo 和 binlog）；一致性是数据库单独保证的吗（不是，业务也要参与）；隔离性有哪些实现代价（锁、MVCC、并发度与业务语义的权衡）。

---

## 02 理解机制与追问

原子性和持久性关注不同失败时刻：事务未提交要能撤销，提交后则要能重做。脏页可能包含尚未提交事务的修改，因此崩溃恢复不是只重放“已提交数据”；通常先恢复页状态，再由事务状态和 undo 处理未完成事务。业务一致性仍需唯一约束、条件更新或合适隔离级别配合。

---

## 03 易错点与适用边界

持久性必须加配置前提。常见强持久配置为 innodb_flush_log_at_trx_commit=1，并在启用 binlog 时配合 sync_binlog=1，还依赖存储真实兑现刷盘承诺。放宽刷盘、异步副本故障切换、设备损坏的保证各不相同。原“异常就全部回滚”也应区分语句失败与整个事务失败，应用必须按错误类型处理事务。

---

## 04 面试口述

undo 支持撤销和历史版本，redo 支持页恢复，锁与 MVCC 实现隔离；一致性由数据库约束和业务逻辑共同维持。谈提交不丢必须说明刷盘与故障模型。

---

## 05 关联问题

- [[八股/03-MySQL/03-存储引擎与索引结构/05-InnoDB 和 MyISAM 有什么区别？为什么通常选择 InnoDB|InnoDB 和 MyISAM 有什么区别？为什么通常选择 InnoDB]]

- [[八股/03-MySQL/03-存储引擎与索引结构/02-B+ 树索引是怎么更新的|B+ 树索引是怎么更新的]]

- [[八股/03-MySQL/04-事务与日志/07-为什么有 binlog 还需要 redo log？两者分别解决什么问题|redo 和 binlog 分工]]
- [[八股/03-MySQL/04-事务与日志/04-MVCC 是什么？它如何实现一致性读|MVCC]]

---

## 06 官方参考

- [MySQL 8.4 Redo Log](https://dev.mysql.com/doc/refman/8.4/en/innodb-redo-log.html)
- [MySQL 8.4 二进制日志参数](https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html)
- [MySQL 8.4 隔离级别](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html)
- [MySQL 8.4 InnoDB 刷盘参数](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit)

---

## 07 相关问题与延伸

- [[八股/02-Spring框架/01-Spring核心/08-事务传播行为常见用法|事务传播行为常见用法]]：应用事务边界与数据库ACID

---

## 08 所属专题

- [[八股/03-MySQL/04-事务与日志/00-事务与日志导航|事务与日志导航]]
