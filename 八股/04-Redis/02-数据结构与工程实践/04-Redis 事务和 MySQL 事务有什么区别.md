---
aliases:
- "Redis 事务和 MySQL 事务有什么区别？"
- "Redis 2.4 Redis 事务和 MySQL 事务有什么区别？"
---

# 04 Redis 事务和 MySQL 事务有什么区别？

## 01 核心回答


**Redis 事务**

**命令：**`MULTI`、`EXEC`、`DISCARD` 和 `WATCH`。

**特点：**`MULTI` 后命令入队，`EXEC` 按顺序执行且不被其他客户端命令插入；`WATCH` 实现乐观锁式条件提交。

**局限：**没有 MySQL 那样完整的回滚机制，入队语法错误、执行类型错误和业务条件失败需分别处理。

**MySQL 事务**

**边界：**`BEGIN`、`COMMIT`、`ROLLBACK`。

**实现：**InnoDB 通过 undo log、redo log、锁、MVCC 和约束实现 ACID；修改可整体回滚，按隔离级别控制并发读写，宕机后能通过日志恢复已提交结果。

Redis 事务适合短小的多 key 原子操作或条件更新，Lua 脚本可以把判断和修改放到一次执行中；MySQL 事务适合多行、多表和需要持久化的业务。两者分属两个系统，不能自动组成分布式事务，跨库操作仍需消息、补偿、Saga 或状态机设计。

---

## 02 理解机制与追问

EXEC 执行阶段中的命令不被其他客户端命令插入。入队阶段的命令错误可导致 EXEC 拒绝整个事务；执行时类型错误通常只让该命令失败，其他已执行和后续命令不会自动回滚。WATCH 监视键变化，冲突时 EXEC 放弃执行，由客户端重读重试。

---

## 03 易错点与适用边界

Lua 的原子执行同样不等于遇到运行时错误自动撤销已经发生的写入；应先检查参数和类型再修改。MULTI 内无法先拿某条读取的结果再决定客户端后续命令，条件逻辑可用 WATCH 或脚本。Cluster 多键事务/脚本一般要求同槽，并不能顺带事务化 MySQL、MQ 或 HTTP 调用。

---

## 04 面试口述

Redis 提供执行不穿插和乐观检查，但没有关系数据库式的完整回滚语义。我会区分入队错误、执行错误、WATCH 冲突，以及跨系统边界。

---

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/02-Redis 和 MySQL 的区别是什么|Redis 和 MySQL 的区别是什么]]

- [[八股/04-Redis/02-数据结构与工程实践/08-Redis 分布式锁怎么实现|锁与原子执行]]
- [[八股/04-Redis/02-数据结构与工程实践/03-Redis 和 MySQL 的数据一致性如何保证|跨库一致性]]

---

## 06 官方参考

- [Redis 事务与错误处理](https://redis.io/docs/latest/develop/using-commands/transactions/)
- [Redis Lua 脚本执行机制](https://redis.io/docs/latest/develop/programmability/eval-intro/)
- [Redis Cluster 协议与槽迁移](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)

---

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
