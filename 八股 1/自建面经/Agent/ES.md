# Elasticsearch

## 一、基础

# 1.1 Elasticsearch 怎么使用？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">Elasticsearch 是一个基于 Lucene 的分布式搜索和分析引擎，常用于全文检索、日志检索、商品搜索、指标聚合和可观测性分析。使用时通常先创建索引并定义 Mapping，明确字段类型、是否分词、分词器、日期格式和是否建立索引；然后通过单条写入或 Bulk API 导入文档。业务查询使用 Query DSL，可以组合全文匹配、精确过滤、范围查询、排序、高亮和聚合。</div>
    <div style="margin:8px 0 0;">典型链路是业务数据先写入 MySQL，再通过消息队列、CDC 或定时同步写入 Elasticsearch，避免把 ES 当作唯一事实来源。查询时，关键词检索使用 <code>match</code>，精确值过滤使用 <code>term</code>，多个条件使用 <code>bool</code> 组合；深分页应避免过大的 <code>from + size</code>，可以使用 <code>search_after</code>，批量遍历历史数据可以根据场景使用 Scroll。聚合查询可以按字段分桶并计算数量、平均值、最大值等指标。</div>
    <div style="margin:8px 0 0;">工程中还需要关注索引模板、别名、分片和副本、刷新间隔、批量大小、Mapping 变更、冷热数据和生命周期管理。写入后默认不是立即对搜索可见，而是在刷新后近实时可见；更新文档底层通常会形成新版本并标记旧文档删除。排查问题时重点看集群健康、节点资源、分片分布、慢查询、写入拒绝、段合并和 JVM 内存。</div>
  </div>
</div>

---

# 1.2 Elasticsearch 有哪些核心特性？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">第一，全文检索能力强。</b>Elasticsearch 使用倒排索引，把词项映射到包含该词的文档，可以快速完成关键词检索，并支持相关性评分、高亮、模糊匹配、同义词和多字段查询。精确查询和范围过滤通常可以利用倒排索引、BKD Tree 或列式存储结构提高效率。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">第二，天然支持分布式。</b>一个索引可以拆成多个主分片，分布在不同节点上；每个主分片可以配置副本，用于故障恢复和分担查询。协调节点接收请求后把查询分发到相关分片，再合并各分片结果。增加节点后可以重新分配分片，实现水平扩展，但分片数量过多也会增加内存、调度和恢复成本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">第三，提供近实时搜索和聚合分析。</b>文档写入后经过 Refresh 才对搜索可见，因此是近实时而不是严格实时。它支持 Terms、Date Histogram、Metrics 等聚合，适合日志统计、行为分析和多维筛选。除此之外还具有 REST API、动态 Mapping、索引别名、快照恢复和生命周期管理等能力。</div>
    <div style="margin:8px 0 0;">需要注意的是，Elasticsearch 不擅长复杂事务、强一致关系和大量跨实体 Join。它通常作为搜索或分析副本，核心业务数据仍由关系数据库保存，并通过同步机制处理最终一致性。</div>
  </div>
</div>

---

# 1.3 Elasticsearch 分词器有哪些？为什么选择 IK 分词器？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">Elasticsearch 的 Analyzer 通常由字符过滤器、Tokenizer 和 Token Filter 组成。常见内置分析器包括：<code>standard</code> 按 Unicode 规则切词并转小写；<code>simple</code> 按非字母字符切分；<code>whitespace</code> 按空格切分；<code>keyword</code> 把整个字段当成一个词项，适合编号、状态和标签；还有 <code>pattern</code>、语言分析器、<code>ngram</code> 和 <code>edge_ngram</code> 等。也可以组合大小写转换、停用词、同义词和词干处理构建自定义分析器。</div>
    <div style="margin:8px 0 0;">中文没有天然空格，标准分词器通常会按单字或不符合业务语义的方式切分，因此中文搜索经常使用 IK 分词器。<code>ik_smart</code> 倾向于较粗粒度切分，词项少、索引体积较小，适合偏精确的搜索；<code>ik_max_word</code> 会尽可能细粒度切分，召回率较高，但词项更多、索引和查询成本也更高。实际项目可以为同一字段建立多个子字段，索引和查询时按场景选择不同分析器。</div>
    <div style="margin:8px 0 0;">选择 IK 的原因是它对中文词语识别、扩展词典和停用词支持较好，接入成本也较低。但分词器不能只看默认效果，业务专有名词、人名、缩写和新词需要维护自定义词典。还要保证索引阶段和查询阶段的分词策略兼容，并用真实搜索样本评估召回率、准确率、零结果率和搜索延迟。</div>
  </div>
</div>

---

# 1.4 Elasticsearch 数据写入与查询原理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">写入路由：</b>客户端先访问协调节点，节点根据文档 ID 或路由值计算目标主分片；主分片执行 Mapping 校验、分析器分词和写入。自定义 routing 可以让同一业务实体落在同一分片，但会放大热点风险，需要结合数据分布评估。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">写入落盘：</b>文档先进入内存缓冲区并生成 Lucene segment，Refresh 后 segment 对 Query 可见；默认 refresh interval 约为 1 秒，因此是近实时而非立即可见。为支持故障恢复，操作还写入 <code>translog</code>，持久化策略由 <code>index.translog.durability</code> 控制。主分片成功后按副本级别复制，副本确认策略会影响延迟和可用性。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">查询流程：</b><code>Get</code> 按 ID 定位单个分片，通常可实时读取最新版本；<code>Query</code> 需要在相关分片执行条件匹配和评分。常见 <code>QUERY_THEN_FETCH</code> 分两阶段：Query Phase 各分片返回局部 topN 的文档 ID 和排序值，协调节点合并全局结果；Fetch Phase 再向命中的分片拉取完整文档。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">倒排索引细节：</b>Analyzer 把文本转换成 term，倒排表保存 term 到文档 ID 的 Posting List；Term Index 通常以 FST 等结构减少内存，Posting List 使用文档 ID 压缩并支持交并集。segment 会在后台合并，合并期间会产生额外磁盘 I/O 和 CPU，旧文档会标记删除并在合并后回收。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">一致性边界：</b>同一主分片内的写入顺序由 primary 负责，副本追平存在时间窗口；Refresh 只影响搜索可见性，不等于数据持久化。需要更强确认时可以使用 <code>refresh=wait_for</code> 或显式 refresh，但会增加资源消耗，不能在高吞吐写入中滥用。</div>
  </div>
</div>

---

# 1.5 Elasticsearch 深分页怎么解决？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">问题本质：</b>每个分片都要先返回 <code>from + size</code> 条候选，再由协调节点全局合并排序——页数越深，排序和内存成本越高。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">浅分页：</b><code>from/size</code> 适合页数较浅、结果量受控的交互式查询。每个分片需要准备前 <code>from + size</code> 条候选，协调节点还要做全局排序，因此要设置 <code>index.max_result_window</code> 和业务最大页数，避免任意深度请求拖垮集群。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">search_after：</b>实时连续翻页使用上一页最后一条记录的 sort 值作为游标，下一页只查该位置之后的数据，避免维护巨大 offset。排序字段必须稳定、可比较且有唯一 tie-breaker（通常加入 <code>_id</code>），否则相同 sort 值会导致重复或漏数据。跨请求期间数据变化还要考虑 PIT 固定搜索视图。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">scroll：</b>批量导出、离线重建和全量遍历可以使用 Scroll，它会保留搜索上下文并分批返回结果，不适合高并发用户分页。长时间不释放 scroll 会占用资源，任务结束要主动清理并设置合理 keep alive。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">替代方案：</b>如果用户只是想跳到很后面的结果，优先提供按时间、状态或关键字筛选，而不是无限翻页；对排行榜和统计类需求使用汇总索引、缓存或数据库游标。最终要同时控制查询扇出、返回字段、排序字段和单页大小。</div>
  </div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
