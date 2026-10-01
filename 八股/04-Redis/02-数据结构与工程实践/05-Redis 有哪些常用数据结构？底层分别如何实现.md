---
aliases:
- "Redis 有哪些常用数据结构？底层分别如何实现？"
- "Redis 2.5 Redis 有哪些常用数据结构？底层分别如何实现？"
---

# 05 Redis 有哪些常用数据结构？底层分别如何实现？

## 01 核心回答


String

保存字符串、整数或二进制数据，整数可以直接执行自增，适合缓存、计数和锁值。

Hash

保存字段到值的映射，适合用户、商品等对象；小对象会采用更紧凑的编码，数据变大后转为哈希表。

List

支持两端插入和弹出，适合队列和栈；现代 Redis 通常使用 quicklist 组织多个紧凑列表节点。

Set

保存不重复的无序成员，适合去重、标签和集合运算；底层可能是整数集合、listpack（7.2 起）或哈希表，取决于版本和数据。

Sorted Set

成员带 score，支持排名和范围查询，常见实现是字典加跳表。

Bitmap

基于 String 的位操作，适合签到、活跃标记和布尔状态。

HyperLogLog

使用固定且较小的空间估算基数，适合允许误差的 UV 统计。

Stream

提供消息 ID、消费组和确认机制，适合轻量消息流。

选择结构要结合访问模式、元素规模、是否需要排序或集合运算、持久化和一致性要求。还要关注单 key 过大、热 key、阻塞命令、过期策略和内存碎片，不能只看命令是否方便。

---

## 02 理解机制与追问

逻辑类型与物理编码要分开。String 常用整数或 SDS 字符串编码；Hash 小对象可用 listpack，大对象用字典；ZSet 小集合可用 listpack，大集合用字典加跳表。Redis 7.x List 常见 quicklist 连接紧凑节点。Stream 利用有序 ID 索引和紧凑节点支撑范围访问与消费组状态。

---

## 03 易错点与适用边界

版本修正：Hash/ZSet 从 Redis 7.0 采用 listpack 替代相关 ziplist 紧凑编码；Redis 7.2 的 Set 还可使用 listpack，因此“Set 只有 intset/hashtable”不完整。实际用 OBJECT ENCODING 和配置核实。Bitmap/HLL/Geo 是特定语义与实现方式，不应把它们全当作彼此独立的底层对象类别。

---

## 04 面试口述

我先按访问需求选逻辑结构，再解释小对象紧凑编码和大对象常规结构之间的转换；编码随版本和阈值变化，不能背一张永不过期的映射表。

---

## 05 关联问题

- [[八股/04-Redis/02-数据结构与工程实践/06-Redis 的 Sorted Set 底层是怎么实现的？新旧数据结构如何转换？并发读写和删除失败如何处理|ZSet 编码转换]]
- [[八股/04-Redis/02-数据结构与工程实践/33-Redis Hash 的渐进式扩容|字典 rehash]]

---

## 06 官方参考

- [Redis 紧凑编码与版本](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)
- [Redis ZADD 行为与复杂度](https://redis.io/docs/latest/commands/zadd/)
- [Redis HyperLogLog](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/)
- [Redis 地理空间类型](https://redis.io/docs/latest/develop/data-types/geospatial/)

---

## 07 所属专题

- [[八股/04-Redis/02-数据结构与工程实践/00-数据结构与工程实践导航|数据结构与工程实践导航]]
