---
aliases:
- "手写 SQL：分组 Top N / 平均分大于 80 / 每科都不低于 80"
- "Mysql 6.1 手写 SQL：分组 Top N / 平均分大于 80 / 每科都不低于 80"
---

# 01 如何查询分组 Top N 并处理并列排名

## 01 核心回答


**思路：**每组 Top N 的核心是窗口函数中的 `PARTITION BY` 与组内排序，再外层筛选排名；输入需先汇总时才先 `GROUP BY`。并列是否全部保留决定选 ROW_NUMBER、RANK 还是 DENSE_RANK。下例保留为**先汇总后的全局 Top 3 对照**，没有 PARTITION BY，因此不是每组前三。

```sql
-- ③ 2025 年消费额前三的用户（user × order 两表）
SELECT user_id, total
FROM (
    SELECT u.id AS user_id,
           SUM(o.amount) AS total,
           ROW_NUMBER() OVER (ORDER BY SUM(o.amount) DESC) AS rn
    FROM user u
    JOIN orders o ON u.id = o.user_id
    WHERE YEAR(o.create_time) = 2025
    GROUP BY u.id
) t
WHERE rn <= 3;
```

**关键细节：**分组 TopN 核心 = PARTITION BY + 有序窗口 + 外层过滤；是否 GROUP BY 取决于输入是否需聚合；`RANK` 和 `DENSE_RANK` 的区别（并列是否占位）；HAVING vs WHERE（聚合后 vs 聚合前）。

**面试追问：**取每组前三用 ROW_NUMBER 还是 RANK（看是否允许并列占位）；DISTINCT 和 GROUP BY 去重区别；JOIN 后 GROUP BY 要注意什么（连接字段唯一性防结果放大）。

## 02 理解机制与追问

保留的对照 SQL 是“先按用户聚合全年消费，再做全局前三”，不是每个业务分组的 Top N。真正每组 Top N 的关键是窗口 PARTITION BY 分组键；若明细行直接排名，不需要先 GROUP BY。ROW_NUMBER 精确取 N 行需唯一 tie-breaker；RANK 保留并列但会跳号，DENSE_RANK 取前 N 个不同排名，返回行数可能超过 N。

## 03 易错点与适用边界

平均分与每科达标已分别拆为独立问题；本题的原 YEAR(create_time) 条件可改为对应年份半开时间区间以利普通索引；保留原示例作为语义对照。

## 04 面试口述

先定义分组、并列和缺失数据口径。明细每组排名用分区窗口；消费排名先聚合再排。对照示例按用户聚合消费后全局排名；组内排名则明确写出分区维度。

## 05 关联问题

- [[八股/03-MySQL/05-SQL查询/03-如何查询平均分大于 80 的学生|平均分与 HAVING]]
- [[八股/03-MySQL/05-SQL查询/04-如何查询每科成绩都不低于 80 的学生|每科达标与缺失口径]]
- [[八股/03-MySQL/05-SQL查询/02-左连接和右连接的区别？两表内联查询 on 和 where 的区别|JOIN 后行数放大]]
- [[八股/03-MySQL/02-索引与查询优化/07-时间戳函数为什么可能导致索引失效？如何改写|日期条件改写]]

## 06 官方参考

- [MySQL 8.4 窗口函数](https://dev.mysql.com/doc/refman/8.4/en/window-function-descriptions.html)
- [MySQL 8.4 范围访问与 Skip Scan](https://dev.mysql.com/doc/refman/8.4/en/range-optimization.html)

## 07 所属专题

- [[八股/03-MySQL/05-SQL查询/00-SQL查询导航|SQL查询导航]]
