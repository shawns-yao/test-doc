---
aliases:
- "`EXPLAIN` 应该重点关注哪些字段？"
- "Mysql 4.5 `EXPLAIN` 应该重点关注哪些字段？"
---

# 06 `EXPLAIN` 应该重点关注哪些字段？

## 01 核心回答


重点关注字段：

id / select_type

判断查询块、子查询和 UNION 的执行结构。

table / partitions

当前访问的表、派生表以及命中的分区。

type

访问方式，常见选择性从强到弱可粗略列为 `system`、`const`、`eq_ref`、`ref`、`range`、`index`、`ALL`。出现 `ALL` 不一定错误——小表全扫可能更便宜，但大表要重点关注。

possible_keys / key / key_len

候选索引、实际选择的索引以及使用的索引长度，可辅助判断联合索引使用到哪些部分。

ref

索引列与常量或其他表字段如何比较。

rows / filtered

预计扫描行数和条件过滤比例，两者结合可估算传给下一步的数据量。

Extra

关注 `Using index`（覆盖索引）、`Using index condition`（ICP）、`Using where`、`Using temporary`、`Using filesort` 等信息。

`EXPLAIN` 主要是**优化器估算**，不等于真实执行情况。条件允许时用 `EXPLAIN ANALYZE` 对比估算行数、实际行数、循环次数和耗时；估算偏差大可能是统计信息过旧、字段相关性强或数据分布倾斜。不能只看到“使用了索引”就认为查询已经优化，还要关注实际扫描、回表和结果集大小。

**面试追问：**type 从 ref 退化成 ALL 说明什么（访问方式发生变化，需要结合成本和实际数据判断）；Using filesort 一定慢吗（不一定慢；表示额外排序，可能在内存完成）；key_len 能看出什么（联合索引用了几列）。

---

## 02 理解机制与追问

传统格式适合快速看访问方式；TREE 能看执行器父子关系。阅读实际计划时从叶子向根追踪输入行数、过滤后行数和 loops，某个小内表扫描若重复十万次也可能是瓶颈。actual time 通常含子节点工作，不能把各节点耗时简单求和；多次循环时注意指标的平均值语义。

---

## 03 易错点与适用边界

修正原追问：Using filesort 只表示额外排序，不一定慢，也不意味着一定落盘。type 的顺序只是经验线索；小表 ALL 可能优于大范围 ref。key_len 包含类型长度、可空标记等信息，不能单独证明每列都有效过滤；要与范围、条件、实际行数一起看。MySQL 8.4 的 EXPLAIN ANALYZE 使用 TREE 格式。

---

## 04 面试口述

我重点看实际访问路径、行数估算偏差、loops 和最耗时节点。索引名、filesort 或 ALL 都只是线索，结论要由总工作量和真实耗时支撑。

---

## 05 关联问题

- [[八股/03-MySQL/02-索引与查询优化/05-MySQL 慢查询如何优化|慢查询优化闭环]]

---

## 06 官方参考

- [MySQL 8.4 EXPLAIN 与 EXPLAIN ANALYZE](https://dev.mysql.com/doc/refman/8.4/en/explain.html)
- [MySQL 8.4 索引条件下推](https://dev.mysql.com/doc/refman/8.4/en/index-condition-pushdown-optimization.html)

---

## 07 所属专题

- [[八股/03-MySQL/02-索引与查询优化/00-索引与查询优化导航|索引与查询优化导航]]
