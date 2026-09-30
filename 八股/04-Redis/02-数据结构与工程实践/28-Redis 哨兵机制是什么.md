---
aliases:
- "Redis 哨兵机制是什么？"
- "Redis 2.28 Redis 哨兵机制是什么？"
---

# 28 Redis 哨兵机制是什么？

## 01 核心回答

**概念：**Sentinel 解决主从模式 **master 宕机后服务不可用**的问题——独立进程，监控多个 master-slave 集群，通过**多哨兵投票确认故障并自动选主切换**，在多数派可用、候选副本和网络等条件满足时帮助恢复服务。

**选型：**高可用用 **Sentinel**；要横向扩容/分片用 **Redis Cluster**——Sentinel 管"主挂了自动切"，Cluster 管"数据分散 + 高可用"。

**追问：**哨兵怎么判断主节点挂了（主观下线 + 多哨兵客观下线投票）；切换期间会丢写吗（可能，异步复制未同步部分）；客户端怎么感知切换（哨兵通知 + 客户端重连新主）。

## 02 理解机制与追问

主观下线是单个 Sentinel 的判断，客观下线需达到配置的 quorum；真正发起故障转移还需获多数 Sentinel 授权。quorum 和多数派选举不是同一个阈值概念。客户端从 Sentinel 获取当前主地址并重连，而不是 Sentinel 自己代理所有数据请求。

## 03 易错点与适用边界

少数分区通常不能完成合法故障转移；部署在同一故障域会削弱多 Sentinel 的意义。异步复制仍可能丢已确认写，旧主隔离与客户端写入路径需约束。Sentinel 不负责数据分片，也不替代 RDB/AOF 与备份；恢复可用不等于任何时刻无丢写。

## 04 面试口述

Sentinel 负责监控、故障判定和协调提升副本；要区分 quorum 与多数派授权，并让客户端正确发现新主。它提高可用性，不自动提高复制为强一致。

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/01-Redis 和 MySQL 的主从复制分别是怎么进行的|复制窗口]]
- [[八股/04-Redis/02-数据结构与工程实践/23-Redis 的持久化机制有哪些|持久化]]

## 06 官方参考

- [Redis Sentinel 高可用](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)
- [Redis 复制机制](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
