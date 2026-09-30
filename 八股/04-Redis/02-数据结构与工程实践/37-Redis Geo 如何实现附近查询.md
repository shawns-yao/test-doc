---
aliases:
- "Redis Geo 如何实现附近查询"
---

# 37 Redis Geo 如何实现附近查询

## 01 核心回答

Geo 保存经纬度并检索附近的人、店或骑手，底层利用 Sorted Set 和地理编码。GEOADD 写入位置，GEODIST 计算距离，GEOSEARCH 按圆形半径或矩形范围查找；较新的使用方式优先评估 GEOSEARCH，而非只记旧 GEORADIUS。

## 02 查询机制

以 GeoHash 式空间编码把经纬度位交错，形成有序分值，利用附近网格筛出候选，再用真实距离或几何范围过滤。空间邻近可以帮助缩小候选，但一段 score 不等于精确圆形；结果排序、数量限制与精确筛选有独立成本。相同成员的位置更新会替换其空间分值。

## 03 易错点与边界

注意经度在前、纬度在后，并确认业务坐标系统一致；Redis Geo 支持的纬度范围不覆盖全部极区。球面距离是近似，复杂多边形、投影、道路距离和导航不应直接当作 Geo 的基本能力。限制返回量与查询半径，避免一个大 key 成为热点；迁移和分片时跨区查询需应用合并。

## 04 面试追问与口述

为什么能检索附近？空间编码给出候选、有序集合辅助定位，再做范围/距离过滤。它适合基础附近检索，并非完整 GIS；如果需求是路网距离或复杂空间关系，应评估专用地理系统。

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/06-Redis 的 Sorted Set 底层是怎么实现的？新旧数据结构如何转换？并发读写和删除失败如何处理|Sorted Set 底层]]
- [[八股/04-Redis/02-数据结构与工程实践/26-Redis 的 HyperLogLog 是什么|HLL 近似基数，独立主题]]
- [[八股/04-Redis/02-数据结构与工程实践/30-大 key 的风险与治理|大 key 与热点治理]]

## 06 官方参考

- [Redis Geospatial](https://redis.io/docs/latest/develop/data-types/geospatial/)
- [Redis GEOADD 坐标约束](https://redis.io/docs/latest/commands/geoadd/)
- [Redis GEOSEARCH](https://redis.io/docs/latest/commands/geosearch/)

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
