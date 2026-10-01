---
aliases:
- "Elasticsearch 怎么使用？"
- "ES 1.1 Elasticsearch 怎么使用？"
---

# 01 Elasticsearch 怎么使用？

## 01 核心回答


Elasticsearch 是一个基于 Lucene 的分布式搜索和分析引擎，常用于全文检索、日志检索、商品搜索、指标聚合和可观测性分析。使用时通常先创建索引并定义 Mapping，明确字段类型、是否分词、分词器、日期格式和是否建立索引；然后通过单条写入或 Bulk API 导入文档。业务查询使用 Query DSL，可以组合全文匹配、精确过滤、范围查询、排序、高亮和聚合。

典型链路是业务数据先写入 MySQL，再通过消息队列、CDC 或定时同步写入 Elasticsearch，避免把 ES 当作唯一事实来源。查询时，关键词检索使用 `match`，精确值过滤使用 `term`，多个条件使用 `bool` 组合；深分页应避免过大的 `from + size`，优先使用 `PIT + search_after`；Scroll 可维护既有批量任务，但不是新设计深分页的默认推荐。聚合查询可以按字段分桶并计算数量、平均值、最大值等指标。

工程中还需要关注索引模板、别名、分片和副本、刷新间隔、批量大小、Mapping 变更、冷热数据和生命周期管理。写入后默认不是立即对搜索可见，而是在刷新后近实时可见；更新文档底层通常会形成新版本并标记旧文档删除。排查问题时重点看集群健康、节点资源、分片分布、慢查询、写入拒绝、段合并和 JVM 内存。

---

## 02 从商品搜索说明使用步骤

先把查询需求映射成字段：商品标题用 text 做全文检索，商品编号和状态用 keyword 做精确筛选，价格和时间用数值/date 做范围与排序。若标题同时需要分词搜索和精确聚合，可设置 text 与 keyword 多字段；不要把所有值都交给动态 Mapping 猜类型。

查询时 match 会按字段搜索分析器分析输入，term 匹配的是已有词项，不会替用户输入重新做完整分词。商品状态、店铺、价格等确定条件放在 bool.filter 中，相关性关键词放在 query 评分部分。按 text 字段做精确排序或聚合要重新审视建模，通常使用相应 keyword 子字段。

依据：[字段映射](https://www.elastic.co/docs/manage-data/data-store/mapping)、[Keyword 字段](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/keyword)。

---

## 03 同步链路比调用 API 更重要

MySQL 到 ES 要明确全量初始化、增量来源、删除事件、版本顺序和重试。用业务 ID 作为稳定文档 ID 可以避免重试新增出多份文档，但无法独自阻止旧事件覆盖新状态；需要有序消费、业务版本校验或适用的版本控制。ES 的 if_seq_no/if_primary_term 处理 ES 内部并发修改，不能直接把 MySQL 的版本号当作这两个参数。

Bulk 返回 HTTP 成功不代表每条操作都成功，应检查 items，按错误类型处理失败项：映射错误修复数据，临时拒绝做有限退避重试，避免无差别重发整批。写成功但搜索不可见时先判断 refresh，而不是马上补写一份。

依据：[Bulk API 的逐项结果](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-bulk)、[乐观并发控制](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/optimistic-concurrency-control)。

---

## 04 变更与口述

已存在字段的类型通常不能直接改，常见做法是建新索引、回填数据、同步追平后切别名。别名切换的原子性不自动解决迁移期间漏增量、删除遗漏或旧事件覆盖，切换前后要核对条数、抽样字段、关键查询和延迟。

口述：“我先围绕查询定义 Mapping，再用 Bulk 和增量事件维护搜索副本，查询结合全文匹配与精确过滤。重点防止批量部分失败、重复/乱序同步和刷新延迟；改字段结构时用新索引加别名切换，并验证增量追平。”

- [Mapping 更新限制](https://www.elastic.co/docs/manage-data/data-store/mapping/update-mappings-examples)
- [别名及多动作切换](https://www.elastic.co/docs/manage-data/data-store/aliases)

---

## 05 相关问题与延伸

- [[八股/05-消息队列与搜索/04-Elasticsearch/02-Elasticsearch 有哪些核心特性|Elasticsearch 有哪些核心特性]]：使用路径与搜索引擎能力

---

## 06 所属专题

- [[八股/05-消息队列与搜索/04-Elasticsearch/00-Elasticsearch导航|Elasticsearch导航]]
