---
aliases:
- "Redis 数据结构如何用于存储 DAU？为什么使用 Bitmap，而不是 MySQL 或 `COUNT(*)`？"
- "Redis 2.10 Redis 数据结构如何用于存储 DAU？为什么使用 Bitmap，而不是 MySQL 或 `COUNT(*)`？"
---

# 10 Redis 数据结构如何用于存储 DAU？为什么使用 Bitmap，而不是 MySQL 或 `COUNT(*)`？

## 01 核心回答


**方案：**用户 ID 映射为相对紧凑的整数后，把“某用户当天是否活跃”映射到 Bitmap 的一个 bit，例如 `SETBIT dau:2026-09-07 user_id 1`；统计去重活跃人数用 `BITCOUNT`，按天保存 key，用 `BITOP` 计算交集和并集。

**Bitmap 方案**

空间按位占用，重复写入天然幂等，单条位操作和批量统计速度快，适合只关心布尔状态的 DAU。

**注意：**用户 ID 很稀疏时浪费空位，可考虑 Redis Set、Roaring Bitmap 或离线聚合。

**MySQL 方案**

每个用户一行活跃记录会产生更多行和索引维护成本；直接 `COUNT(*)` 需要扫描符合条件的记录，不能天然消除同一用户的重复访问。

需要明确 Bitmap 统计的是去重用户数，不是访问次数；还要处理时区、日期边界、用户 ID 类型、key 过期和故障恢复。Redis 结果如果用于财务、审计或核心运营口径，最好通过访问日志或数仓离线结果定期校验。

---

## 02 理解机制与追问

空间取决于最大偏移，不是活跃人数。1 亿个连续编号的单日位图约 12.5 MB；只活跃 1 万人但最大 ID 很大，也会分配大量空位。多天求留存需每天使用同一用户编号映射，映射重用会把不同用户算成同一个人。BITCOUNT 扫描字节，不能把全量统计当成 O(1)。

---

## 03 易错点与适用边界

单个 String 的大小和 SETBIT 偏移受限，超大 ID 空间应分桶；大偏移首次写入可能触发显著分配延迟。MySQL COUNT(DISTINCT user_id) 或 (date,user_id) 唯一记录也能精确去重，区别在模型、成本和查询需求，不是 MySQL 不会去重。Bitmap 通常不能给每个 bit 单独设 TTL。

---

## 04 面试口述

ID 紧凑、只需是否活跃及交并统计时 Bitmap 合适；稀疏 ID 或需要用户明细则考虑集合或离线明细。先算最大偏移、保留天数和统计开销。

---

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/26-Redis 的 HyperLogLog 是什么|HLL 近似去重]]
- [[八股/04-Redis/02-数据结构与工程实践/17-如何设计一个结构来存储 QQ 号和在线状态|在线状态与 TTL]]

---

## 06 官方参考

- [Redis SETBIT 与空间边界](https://redis.io/docs/latest/commands/setbit/)
- [Redis 紧凑编码与版本](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)

---

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
