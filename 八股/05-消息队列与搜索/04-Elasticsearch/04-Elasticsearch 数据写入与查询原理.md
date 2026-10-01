---
aliases:
- "Elasticsearch 数据写入与查询原理？"
- "ES 1.4 Elasticsearch 数据写入与查询原理？"
---

# 04 Elasticsearch 数据写入与查询原理？

## 01 核心回答

**写入路由：**客户端先访问协调节点，节点根据文档 ID 或路由值计算目标主分片；主分片执行 Mapping 校验、分析器分词和写入。自定义 routing 可以让同一业务实体落在同一分片，但会放大热点风险，需要结合数据分布评估。

**写入落盘：**文档先进入内存缓冲区并生成 Lucene segment，Refresh 后 segment 对 Query 可见；常见 Elastic Stack 默认活跃索引周期约为 1 秒，但以部署形态及配置为准，因此是近实时而非立即可见。为支持故障恢复，操作还写入 `translog`，持久化策略由 `index.translog.durability` 控制。主分片将操作复制给 in-sync 副本，并等待成功或完成失败副本移出同步集合；确认与可用性受副本状态影响。

**查询流程：**`Get` 按 ID 定位单个分片，通常可实时读取最新版本；`Query` 需要在相关分片执行条件匹配和评分。常见 `QUERY_THEN_FETCH` 分两阶段：Query Phase 各分片返回局部 topN 的文档 ID 和排序值，协调节点合并全局结果；Fetch Phase 再向命中的分片拉取完整文档。

**倒排索引细节：**Analyzer 把文本转换成 term，倒排表保存 term 到文档 ID 的 Posting List；Term Index 通常以 FST 等结构减少内存，Posting List 使用文档 ID 压缩并支持交并集。segment 会在后台合并，合并期间会产生额外磁盘 I/O 和 CPU，旧文档会标记删除并在合并后回收。

**一致性边界：**同一主分片内的写入顺序由 primary 负责，副本追平存在时间窗口；Refresh 只影响搜索可见性，不等于数据持久化。需要等待搜索可见时可以使用 `refresh=wait_for`；显式 refresh 立即刷新，二者都不是提升持久化等级的开关，但会增加资源消耗，不能在高吞吐写入中滥用。

---

## 02 必须区分 Refresh Flush 与 Merge

- Refresh 打开新的可搜索 segment，使近期操作对搜索可见，不要求完成一次 Lucene commit
- Translog 记录尚需在恢复时重放的操作。默认 durability=request 下，成功确认前会按要求对 translog 执行 fsync；改为 async 会引入最近操作的丢失窗口
- Flush 完成 Lucene commit 并切换 translog generation，减少恢复所需重放的日志，通常由系统自动执行
- Merge 合并 segment 并回收符合条件的已删除文档，消耗 CPU 和 I/O；不是“每次更新都立刻重写整个索引”

因此“写入返回成功”与“全文搜索看得到”可以同时出现不同状态。它们分别由可靠写入链路和搜索器刷新控制。依据：[近实时搜索](https://www.elastic.co/docs/manage-data/data-store/near-real-time-search)、[Translog 与 flush](https://www.elastic.co/docs/reference/elasticsearch/index-settings/translog)。

---

## 03 从确认语义解释一致性

主分片先校验并执行写入，再向当前 in-sync 副本复制；某副本失败时，要由集群完成相应的同步集合变更后才能在规定条件下继续确认。不能把它描述成任意选择“确认一个还是确认多个副本”的 Kafka acks 模型。wait_for_active_shards 是开始处理前的活跃副本数量检查，也不等于事后绝对保证所有配置副本都写成功。

refresh=wait_for 通常等待发生刷新，refresh=true 主动刷新；后者高频使用会产生大量小 segment，增加合并成本。wait_for 也受资源、刷新设置和监听器上限影响，不是无代价强一致开关。按 ID 的实时 GET 与 search 的近实时语义应分别解释。

依据：[ES 主副本写入协议](https://www.elastic.co/docs/deploy-manage/distributed-architecture/reading-and-writing-documents)、[refresh 参数](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/refresh-parameter)。

---

## 04 查询链路与口述

QUERY_THEN_FETCH 的第一阶段只收集排序所需候选及元信息，第二阶段再拉最终文档，避免所有分片一开始就返回完整正文。代价是全局 TopN 需要各分片局部候选，深分页会扩大候选集；聚合、高亮、脚本和复杂评分还可能成为额外成本，不能以“命中倒排索引”推导整个请求都便宜。

口述：“写入先路由主分片，执行索引和 translog，再复制到同步副本并按协议确认。Refresh 决定搜索可见，Flush 决定 Lucene 提交与日志切换，Merge 决定段整理。查询先各分片找候选，再全局合并和抓取正文；GET、search、持久化是三个不同维度。”

追问：并发修改怎样避免覆盖？对常规支持序列号的索引，用读取获得的 if_seq_no 和 if_primary_term 做乐观并发检查，冲突后重新读取并按业务决定重试。依据：[乐观并发控制](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/optimistic-concurrency-control)。

---

## 05 相关问题与延伸

- [[八股/05-消息队列与搜索/04-Elasticsearch/02-Elasticsearch 有哪些核心特性|Elasticsearch 有哪些核心特性]]：搜索能力背后的写入和查询链路
- [[八股/05-消息队列与搜索/04-Elasticsearch/03-Elasticsearch 分词器有哪些？为什么选择 IK 分词器|Elasticsearch 分词器有哪些？为什么选择 IK 分词器]]：文本分析影响索引与查询
- [[八股/05-消息队列与搜索/04-Elasticsearch/05-Elasticsearch 深分页怎么解决|Elasticsearch 深分页怎么解决]]：检索执行与分页一致性

---

## 06 所属专题

- [[八股/05-消息队列与搜索/04-Elasticsearch/00-Elasticsearch导航|Elasticsearch导航]]
