---
aliases:
- "Elasticsearch 深分页怎么解决？"
- "ES 1.5 Elasticsearch 深分页怎么解决？"
---

# 05 Elasticsearch 深分页怎么解决？

## 01 核心回答

**问题本质：**每个分片都要先返回 `from + size` 条候选，再由协调节点全局合并排序——页数越深，排序和内存成本越高。

**浅分页：**`from/size` 适合页数较浅、结果量受控的交互式查询。每个分片需要准备前 `from + size` 条候选，协调节点还要做全局排序，因此要设置 `index.max_result_window` 和业务最大页数，避免任意深度请求拖垮集群。

**search_after：**连续翻页使用上一页最后一条记录的完整 sort 值作为游标，下一页只查该位置之后的数据，避免维护巨大 offset。排序字段必须稳定、可比较且有唯一 tie-breaker（无 PIT 时使用带 doc_values 的业务唯一字段；PIT 有 `_shard_doc` 辅助排序，不直接用 `_id` 排序），否则相同 sort 值会导致重复或漏数据。跨请求期间数据变化还要考虑 PIT 固定搜索视图。

**scroll：**Scroll 可用于既有批量遍历流程；官方对新深分页设计推荐 PIT + search_after，它会保留搜索上下文并分批返回结果，不适合高并发用户分页。长时间不释放 scroll 会占用资源，任务结束要主动清理并设置合理 keep alive。

**替代方案：**如果用户只是想跳到很后面的结果，优先提供按时间、状态或关键字筛选，而不是无限翻页；对排行榜和统计类需求使用汇总索引、缓存或数据库游标。最终要同时控制查询扇出、返回字段、排序字段和单页大小。

---

## 02 为什么调大窗口不是解决方案

若查询涉及 S 个分片，每个分片通常需维护约 from+size 个候选，协调端再归并。请求“第 10000 页，每页 20 条”即使只返回 20 条，也可能让各分片处理大量被丢弃的候选。index.max_result_window 默认 10000 是护栏，不是搜索总命中数上限；提高它只放宽限制，不会消除计算和内存成本。

search_after 根据排序游标续查，避免为了跳过前面所有页保留巨大候选集；它仍要执行过滤、排序等工作，不能泛称为常数时间或无成本分页。依据：[官方分页方案](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/paginate-search-results)。

## 03 PIT 与游标的正确配合

1. 为目标索引打开 PIT，确定查询条件、稳定排序与页大小
2. 查询第一页，保存返回的最新 PIT ID 与最后一条命中的完整 sort 数组
3. 后续保持 query、sort 一致，传入 search_after 及 PIT；使用响应中的最新 PIT ID，并设置合理 keep_alive
4. 无更多命中或用户结束后关闭 PIT，避免旧 segment、文件句柄和删除状态跟踪长期占用资源

PIT 固定一个搜索视图，让翻页期间的新增、更新和删除不会造成结果漂移；因此它不等于“每页都看最新数据”。PIT 不是备份，也不是无限期会话，超时或丢失后不能随意拿旧游标在新视图上继续并保证完全相同结果，应按产品语义重新开始或使用预生成导出。

没有 PIT 时，选择唯一且可排序的业务字段作为 tie-breaker。_id 有排序限制，若需要其值排序，应复制到开启 doc_values 的字段；PIT 自带的 _shard_doc 仅在该 PIT 内稳定，不应长期保存作业务排序键。

依据：[PIT API 与资源生命周期](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-open-point-in-time)、[_id 的排序限制](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/mapping-id-field)。

## 04 产品取舍与口述

search_after 天然适合下一页或滚动加载，不直接支持随意跳到第 N 页。若必须跳页，可限制页深、存短期分页锚点，或把固定结果集预生成；锚点需要与查询和视图绑定，不能跨条件复用。大批量导出还应限制并发、只取所需字段，并使用可恢复进度。

口述：“浅分页用 from/size 并保留窗口护栏；深度连续翻页优先 PIT 加 search_after，以完整 sort 数组续查，用 PIT 固定视图并管理过期。它解决的是大 offset 和跨页漂移，不能免费提供任意跳页，也不能直接拿 _id 排序。”

> 返回导航：[[八股/00-总导航|00-总导航]]

## 05 相关问题与延伸

- [[八股/05-消息队列与搜索/04-Elasticsearch/04-Elasticsearch 数据写入与查询原理|Elasticsearch 数据写入与查询原理]]：检索执行与分页一致性

## 06 所属专题

- [[八股/05-消息队列与搜索/04-Elasticsearch/00-Elasticsearch导航|Elasticsearch导航]]
