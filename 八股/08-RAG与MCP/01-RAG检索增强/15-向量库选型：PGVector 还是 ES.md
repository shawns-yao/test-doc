---
aliases:
- "向量库选型：PGVector 还是 ES？"
- "RAG 1.15 向量库选型：PGVector 还是 ES？"
---

# 15 向量库选型：PGVector 还是 ES？

## 01 核心回答

ES 可以做向量检索，但选 PGVector 是因为场景需要「**向量检索和关系数据一体化**」和更低接入成本：① 数据原本在 PostgreSQL 体系，减少异构系统同步和一致性治理成本；② 规模和 QPS 在 PGVector 可承受范围内，优先工程简洁；③ 很多业务查询是「结构化过滤 + 向量相似度」组合，同库更顺滑。

**ES 的优势：**大规模检索和复杂搜索生态——数据量上升到更高量级时，ES 或专用向量库是候选。

**面试表述：**选型不是绝对优劣，而是「业务规模、团队运维能力、数据一致性成本」的综合权衡。

---

## 02 不靠规模标签决定选型

pgvector 提供 PostgreSQL 内的向量能力，可受益于现有 SQL、事务与运维；ES 提供分布式搜索、文本检索与向量组合能力。两者都需要在真实数据、过滤选择性、更新比例、并发和召回目标上压测，不能简单说“小用 PG，大用 ES”。

PG 的近似索引结合过滤可能返回不足，需要调搜索参数或设计索引；ES 的文档写入与可搜索之间受 refresh 等机制影响，也不等于关系事务语义。比较时统一召回率与硬件预算，而不是只比最低延迟。

口述：“我选能满足质量和SLO、同时运维与一致性成本更低的方案，数据原本在哪里只是一个因素。”

## 03 关联追问

- [[八股/08-RAG与MCP/01-RAG检索增强/12-RAG 部署的实际挑战|RAG 部署的实际挑战？]]
- [[八股/08-RAG与MCP/01-RAG检索增强/20-RAG 向量库|RAG 向量库]]

## 04 参考资料

- [pgvector 官方文档](https://github.com/pgvector/pgvector)
- [Elasticsearch 官方 kNN 文档](https://www.elastic.co/docs/solutions/search/vector/knn)

## 05 直接追问：FAISS、pgvector 与 Milvus 如何比较

先区分库与数据库，再比较运行职责：

- FAISS 是相似度检索与索引算法库，可嵌入本地或自建服务，适合算法实验及需要自行控制索引的场景；鉴权、租户隔离、服务高可用和数据同步需应用另行建设，不能把“单机实验”当能力上限
- pgvector 将向量能力放进 PostgreSQL；已有关系数据、SQL 过滤与事务需求时可减少系统间同步，但仍须压测 ANN 与过滤组合的召回和更新开销
- Milvus 提供向量数据库的服务化与分布式部署能力；需要独立扩展检索服务时可作为候选，收益需抵消部署、资源和一致性治理成本，不能笼统称为“扩展性最好”

用同一数据、过滤条件和召回目标比较 p50/p95/p99、吞吐、错误率、资源占用、索引构建/更新耗时及源变更到可搜索的延迟。答案可溯源率与幻觉率属于上层系统指标，不能单独归因于数据库。规模增加也不必自动迁移，先确认现有瓶颈及迁移收益。

官方依据：[FAISS](https://github.com/facebookresearch/faiss/wiki/Getting-started)、[pgvector](https://github.com/pgvector/pgvector)、[Milvus 2.6 架构](https://milvus.io/docs/v2.6.x/architecture_overview.md)。

来源与改写说明：本节依据 [goehou/agent_java_offer 仓库贡献者的题目与资料](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/01_AI/03_RAG/01_%E6%A0%B8%E5%BF%83%E9%97%AE%E7%AD%94.md#L291-L307)（[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)）补充。补充原目标缺少的 FAISS/Milvus 运行职责及上线指标；保留原有 PG 与 ES 对比，不采用原资料绝对优劣和必然迁移建议。 许可说明仅对应本节引入的来源内容，不改变本篇其他原有内容的许可。

## 06 所属专题
- [[八股/08-RAG与MCP/01-RAG检索增强/00-RAG检索增强导航|RAG检索增强导航]]
