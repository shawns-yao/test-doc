---
aliases:
- "InnoDB 和 MyISAM 有什么区别？为什么通常选择 InnoDB？"
- "Mysql 3.5 InnoDB 和 MyISAM 有什么区别？为什么通常选择 InnoDB？"
---

# 05 InnoDB 和 MyISAM 有什么区别？为什么通常选择 InnoDB？

## 01 核心回答


**InnoDB**

支持事务和 ACID，通过 `redo log`、`undo log`、锁和 MVCC 实现崩溃恢复、回滚和并发控制。

支持**行级锁**、外键和一致性读；使用聚簇索引（主键叶子存完整行）。适合订单、账户、库存等需要并发写入和数据可靠性的业务。

**MyISAM**

不支持事务和 MVCC，主要使用**表级锁**，写操作容易阻塞整张表；异常宕机后恢复能力弱于 InnoDB。

索引和数据分离存储，索引叶子保存记录地址；表行数可从元数据直接获得，但这不代表复杂条件统计一定更快。

现代通用业务通常优先选择 InnoDB，因为它在**事务、并发、故障恢复和数据完整性**方面更适合线上系统。只有在明确不需要事务、主要只读，并经过实际测试确认收益的特殊场景，才考虑其他引擎；不能只依据“某种引擎查询快”做选择。

**面试追问：**MyISAM 的 COUNT(*) 为什么快（元数据直接存行数）；表级锁和行级锁的并发差异；这两种引擎谁支持外键（InnoDB；不能扩展为所有引擎只有它支持）。

## 02 理解机制与追问

InnoDB 的优势不是某个单独操作更快，而是把事务、锁、崩溃恢复与并发读写整合起来。MyISAM 可从元数据返回无条件总行数；InnoDB 同一时刻不同事务可能看见不同记录集合，因此不能用一个全局行数准确回答所有事务的 COUNT(*)。

## 03 易错点与适用边界

修正原“只有 InnoDB 支持外键”：若只比较本题两种引擎，InnoDB 支持、MyISAM 不支持；放到 MySQL 全部引擎，NDB 也有外键能力。MyISAM 并非完全没有读写并发优化，但缺少事务与 MVCC 的关键边界不变。不能将无条件计数优势扩展到 WHERE 条件计数。

## 04 面试口述

线上交易通常选择 InnoDB，因为事务、并发控制和恢复能力匹配业务。MyISAM 的个别读取特性不能抵消账户、库存等场景对事务正确性的要求。

## 05 关联问题

- [[八股/03-MySQL/04-事务与日志/01-MySQL 事务有哪些特性？ACID 分别如何实现|ACID 如何实现]]

## 06 官方参考

- [MySQL 8.4 聚簇与二级索引](https://dev.mysql.com/doc/refman/8.4/en/innodb-index-types.html)
- [MySQL 8.4 隔离级别](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html)
- [MySQL 8.4 外键约束及存储引擎](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html)

## 07 所属专题

- [[八股/03-MySQL/03-存储引擎与索引结构/00-存储引擎与索引结构导航|存储引擎与索引结构导航]]
