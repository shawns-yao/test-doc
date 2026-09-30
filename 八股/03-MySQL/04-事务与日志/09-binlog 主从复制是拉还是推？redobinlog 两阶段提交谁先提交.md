---
aliases:
- "binlog 主从复制是拉还是推？redo/binlog 两阶段提交谁先提交？"
- "Mysql 5.9 binlog 主从复制是拉还是推？redo/binlog 两阶段提交谁先提交？"
---

# 09 binlog 主从复制是拉还是推？redo/binlog 两阶段提交谁先提交？

## 01 核心回答


**binlog 是拉还是推：****从库主动拉取（pull）**——从库的 I/O 线程向主库发起请求，主库的 dump 线程把 binlog 推给从库，从库写入 relay log 后由 SQL 线程重放。并非主库无请求主动建连：从库自己控制拉取进度（binlog 位点/GTID），断线后从上次位点续拉。

**两阶段提交顺序（先 redo prepare → binlog → redo commit）：**① **prepare 阶段**：事务执行完，InnoDB 写 redo log 并标记为 prepare 状态；② **commit 阶段**：Server 层写 binlog 并刷盘（`sync_binlog=1` 时）→ 然后 InnoDB 把 redo 标记为 commit，事务才算提交成功。

**关键细节（为什么这个顺序）：**宕机恢复时按日志状态判断：① redo prepare 且对应 binlog 完整持久化提交记录存在 → 事务**补齐提交**（redo commit）；② redo prepare 但 binlog 没写 → 事务**回滚**。在正确刷盘和恢复前提下协调**主库数据与 binlog 的提交决定**；不保证副本此时已接收或应用。binlog 必须介于两者之间才能作为两者的"对账凭证"。

**面试追问：**如果 binlog 写成功但 redo commit 失败会怎样（恢复时按对应完整持久化的 binlog 提交证据补齐决定）；sync_binlog=0 的后果（binlog 未刷盘，宕机可能丢日志）；半同步复制是什么（按配置等待副本接收持久化 ACK，超时可能降级异步，不保证已应用）。

## 02 理解机制与追问

更准确的复制表述是“副本发起连接和位点请求，源端通过持久连接流式发送 binlog”，不是每个事件都重新轮询拉取。接收线程写 relay log，应用线程或并行 worker 再执行，因此“已收到”和“已应用”是两种进度。

## 03 易错点与适用边界

修正恢复判断：必须判断相关事务的完整持久化提交证据，不能只看 binlog 文件是否有任意字节。半同步 ACK 通常证明副本已将事件写入 relay log 并刷盘，不代表业务数据已应用；超时可退回异步。因此半同步不等于读副本立刻一致，也不是防脑裂协议。

## 04 面试口述

副本先请求，源端流式发送，副本接收和应用分离。提交逻辑上先引擎 prepare，再 binlog，再引擎 commit；恢复靠一致的事务提交证据，而复制 ACK 不代表已可读。

## 05 关联问题

- [[八股/03-MySQL/04-事务与日志/07-为什么有 binlog 还需要 redo log？两者分别解决什么问题|两种日志分工]]
- [[八股/03-MySQL/06-高可用与扩展/01-数据库读写分离如何实现|写后读路由]]

## 06 官方参考

- [MySQL 8.4 复制线程](https://dev.mysql.com/doc/refman/8.4/en/replication-threads.html)
- [MySQL 8.4 半同步复制](https://dev.mysql.com/doc/refman/8.4/en/replication-semisync.html)
- [MySQL 8.4 二进制日志参数](https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html)

## 07 所属专题

- [[八股/03-MySQL/04-事务与日志/00-事务与日志导航|事务与日志导航]]
