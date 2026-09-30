# RAG


## 一、RAG 与检索增强

# 1.1 RAG 的工作原理？与微调相比解决什么问题？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">原理（先检索、后生成）：</b>① <b>检索（Retrieve）</b>——用户问题作为 query 在外部知识库（向量数据库）搜索最相关的片段；② <b>增强（Augment）</b>——检索片段与原始问题拼接成增强提示；③ <b>生成（Generate）</b>——增强提示喂给 LLM 生成更准确、更具事实性的回答。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">解决的核心问题：</b>① <b>知识过时</b>——LLM 知识冻结在训练截止点，RAG 连接可随时更新的外部知识库；② <b>幻觉</b>——RAG 提供具体相关上下文，把回答锚定在事实依据上；③ <b>缺乏领域私有知识</b>——RAG 可轻松连接任意私有数据集，不必高成本微调。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">相比微调的优势：</b>① <b>知识更新成本低</b>——更新知识只需改数据库，无需重训 LLM；② <b>可追溯可解释</b>——答案可展示来源文档供核查，微调是黑盒；③ <b>降低幻觉</b>——回答有据可循；④ <b>个性化</b>——可为每个用户动态接入不同知识源。</div>
  </div>
</div>

---

# 1.2 从零搭建一个可用的 RAG 系统？完整流水线？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（离线 + 在线两段）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">离线（数据准备/索引）：</b>① <b>数据加载</b>——PDF/Word/网页/数据库等源加载；② <b>文本切块</b>——按语义完整切块；③ <b>嵌入</b>——用嵌入模型（BERT/BGE/M3E）把块转成向量；④ <b>存储</b>——存入向量数据库（FAISS/ChromaDB/Pinecone）并建索引。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">在线（查询/推理）：</b>① 用户提问 → ② 用<b>相同嵌入模型</b>把问题转查询向量 → ③ 相似度检索 Top-K → ④（可选）<b>重排序</b>精细打分取 Top-N → ⑤ 增强与生成（按模板拼提示喂 LLM）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键不在「跑通」在「可治理」：</b>切块要兼顾语义完整与召回粒度；嵌入模型要看领域适配和检索指标；索引要支持<b>增量更新和版本回切</b>。上线时保留数据版本、embedding 版本和索引版本，保证问题可复现、可回放。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">索引全量重建成本高怎么办：</b>① <b>增量索引</b>——chunk_id + content_hash，只重算新增/变更/删除的分片；② <b>双索引版本化（蓝绿）</b>——新索引验收后原子切换，失败一键回滚；③ <b>热冷分层</b>——热数据实时小索引 + 冷数据大基线索引，夜间异步合并；④ <b>元数据与向量解耦</b>——过滤条件变化不重建向量索引，单独维护元数据过滤层。</div>
  </div>
</div>

---

# 1.3 文本切块策略怎么选？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心权衡：</b>块太大，上下文噪声、Embedding 成本和重排延迟会上升；块太小，语义容易断裂，实体、条件和结论可能被拆开。重叠用于减少边界信息损失，但过大又会造成重复召回和索引膨胀。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">先保留结构：</b>解析标题层级、段落、列表、表格和代码边界，优先按语义单元切分，再对超长单元按 token 长度递归切分。标题路径、文档版本、权限和时间等元数据单独保存，不能只拼进正文。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">参数选择：</b>多数通用场景可从 <code>chunk_size=300-800 tokens</code>、<code>overlap=10%-20%</code> 起步，再根据查询粒度调整；FAQ 和错误码偏小块，制度条款和技术方案需要带标题的中等块，代码按函数/类并保留依赖上下文。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">父子块与多尺度：</b>小块负责精确命中，检索后返回所属父段或章节，避免回答缺少前置条件；也可以同时建立句子级和段落级索引，通过查询类型或召回结果动态选择尺度。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">评估与风险：</b>固定一批带标准证据的 query，比较 Recall@K、nDCG、上下文覆盖率、重复率和端到端答案正确率；特别检查表格跨行、列表跨页、标题与正文分离、版本更新导致的旧块残留。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么不能只按固定字符数切（会破坏语义）；overlap 越大越好吗（不是，成本和重复噪声会上升）；如何处理表格（结构化解析或按行列保留表头，不把视觉文本简单串接）。</div>
  </div>
</div>

---

# 1.4 嵌入模型怎么选？怎么评估？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">选型维度：</b>语言覆盖、领域术语、向量维度、上下文长度、推理吞吐、许可协议、部署位置和数据合规都要考虑。不能只看公开榜单，还要确认模型对中文、代码、数字和专有名词的区分能力。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">服务模型：</b>云端模型通常效果和迭代速度较好，但存在网络时延、调用费用和数据出域问题；开源模型可本地部署、便于量化和版本锁定，却需要承担 GPU、监控和升级成本。生产环境要固定模型版本，并记录模型、维度和归一化方式。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">离线评估：</b>构造 query、相关文档、难负例和权限标签组成的黄金集，计算 Recall@K、Precision@K、MRR、nDCG@K；同时按语言、业务域、查询长度、长尾术语分桶，避免平均分掩盖局部退化。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程指标：</b>记录单条向量生成耗时、批量吞吐、显存/内存、索引构建时间、向量维度带来的存储开销，以及升级模型后的向量兼容性。更换模型通常需要重嵌入，不能直接把不同空间的向量混用。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">注意：</b>STS 或 MTEB 高分不等于业务检索最优，必须做真实 query 的 A/B 或离线回放；还要检查提示词语言、query 长度、归一化和距离函数是否与模型训练假设一致。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>余弦相似度和点积何时等价（向量已 L2 归一化时）；为什么要难负例（区分表面相似但答案不相关的文档）；模型升级怎么回滚（双索引、版本路由和可重建流水线）。</div>
  </div>
</div>

---

# 1.5 如何提升 RAG 检索质量？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">优化路径：</b>按“数据质量 → 查询理解 → 召回 → 重排 → 上下文编排 → 生成约束”逐层定位，不把所有问题都归咎于向量库。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">数据与索引：</b>清理重复、过期和无权限文档，补齐标题、来源、版本、时间等元数据；采用 BM25 + 向量的混合搜索，关键词保证实体和错误码精确命中，向量召回覆盖同义表达，再用 RRF 或归一化分数融合。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">查询改写：</b>对省略主语、代词和多轮指代先做 query rewrite；复杂问题拆成子问题，多查询结果去重合并；HyDE 可用于语义表达差异大的场景，但要防止假设答案把检索方向带偏。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">两阶段排序：</b>先用低成本检索器召回 Top20-100，再用 Cross-Encoder 或轻量 reranker 对 query-document 联合编码，最终取 Top3-8；排序后按文档、章节和版本去重，避免上下文被同一来源占满。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">上下文与生成：</b>使用 Small-to-Large 返回父块，给每段附来源和证据范围；按 token 预算截断，提示模型只依据证据回答并在证据不足时拒答。对结构化字段可先查数据库，再把结果作为可信上下文。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">评估闭环：</b>分别测召回命中、排序位置、上下文有效率、答案正确性和引用准确性；线上按 query 类型、租户、版本和失败原因监控，采用离线回放 + 小流量 A/B，避免只看最终满意度。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>TopK 越大越好吗（召回覆盖与噪声、成本存在权衡）；为什么需要 rerank（向量相似度不等于答案相关性）；如何处理权限（检索前过滤或在索引中写入租户 ACL，不能只在生成后过滤）。</div>
  </div>
</div>

---

# 1.6 什么是 Lost in the Middle？怎么缓解？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">现象与原因：</b>当上下文很长时，模型对位于中间的证据利用率可能低于开头和结尾，即使检索已经命中，也可能出现遗漏、引用错误或只依据首尾片段回答。根因既有模型注意力分布，也有上下文过长、重复和排序不合理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">先减少无效上下文：</b>提高召回和 rerank 质量，控制 TopK，去重相邻或同源片段；按 token 预算动态截断，不为了“保险”把大量低相关文档全部塞给模型。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">再优化排列：</b>将最相关证据放在开头和结尾，把多个证据按相关度交错排列；长文档可先做层级摘要，再把支持结论的原文片段放入最终上下文，并明确来源编号。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">改变任务形态：</b>对超长材料采用 Map-Reduce、分块问答后汇总，或让模型先抽取与问题相关的证据再生成最终答案；提示词要求逐条引用证据、证据不足时明确说明，避免凭记忆补全。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">评估方式：</b>构造“关键证据位于开头/中间/结尾”的对照集，分别统计证据召回率、引用准确率和答案正确率；不能只用整体平均分判断是否改善。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">取舍与追问：</b>长上下文模型或专门微调可以缓解，但成本、延迟和迁移风险更高；面试可继续追问“为什么 TopK 不能无限增大”（噪声和注意力稀释）以及“如何证明是 Lost in the Middle 而不是召回失败”（分别做证据命中与位置对照实验）。</div>
  </div>
</div>

---

# 1.7 什么场景用知识图谱增强 RAG？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">适用场景：</b>① <b>多跳推理</b>——问题需要沿实体关系链多次跳转（如「Llama 2 的作者所在公司的 CEO 是谁」，向量检索几乎无法完成，图谱是几次图遍历）；② <b>强结构关联数据</b>——金融股权结构、医疗药物-基因-疾病网络、供应链；③ <b>需要可解释证据链</b>——金融风控、医疗诊断，图谱查询路径本身就是直观证据链。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">关系：</b>多数业务不是替代而是<b>互补增强</b>——先图谱定位实体与关系，构建更精确的查询，再向量检索补充非结构化上下文，最后汇总给模型综合生成。</div>
  </div>
</div>

---

# 1.8 迭代检索和自适应检索范式？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 迭代式检索（Self-RAG / Corrective-RAG）：</b>单向流水线变成循环自我修正——首次检索生成 → LLM 反思评估信息是否足够 → 不足则主动生成新查询二次检索 → 整合精炼。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 自适应检索（FLARE / Self-Ask）：</b>生成过程中「按需」检索——LLM 边生成边预测，遇到不确定的事实信息（概率分布平坦）时暂停、插入检索占位符、获取信息后继续生成。只在需要时检索，避免预检索大量无关信息。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 多源 Agentic RAG：</b>Agent 判断需要哪些信息，同时调用向量库、知识图谱、SQL 数据库或搜索 API，综合生成答案。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">共同点：</b>提升复杂问题准确率，但增加流程复杂度和时延。</div>
  </div>
</div>

---

# 1.9 搜索系统和 RAG 的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">搜索系统的目标是「<b>找文档</b>」，输出候选结果列表；RAG 的目标是「<b>给答案</b>」，输出基于证据融合后的自然语言结果。两者不是替代关系——RAG 本质是在搜索之上叠加了 LLM 的生成层。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程组合：</b>先检索再生成，并保留引用来源，让用户既能快速拿答案、也能回查证据。</div>
  </div>
</div>

---

# 1.10 RAG 生产环境的挑战与治理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#B26E00;">三类挑战：</b>① <b>数据新鲜度</b>——文档更新后索引滞后导致「答旧不答新」；② <b>稳定性</b>——检索链路抖动造成时延和空召回；③ <b>可解释性</b>——回答不可追溯影响业务信任。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">治理手段：</b>增量更新、索引版本化、灰度发布、回滚开关、失败补偿和可观测看板。</div>
  </div>
</div>

---

# 1.11 如何评估一个 RAG 系统？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（检索 + 生成两阶段）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">检索阶段（找得对、找得全）：</b>① <b>上下文精确率（Context Precision）</b>——检索到的文档有多少真正相关（信噪比）；② <b>上下文召回率（Context Recall）</b>——所有相关文档找回多少（全面性）；③ 其他排名指标——Hit Rate（及格线）、MRR（第一个正确结果速度）、nDCG@k（相关性等级 + 排名）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">生成阶段（忠实、准确、有用）：</b>① <b>Faithfulness/Groundedness（可溯源性）</b>——答案是否完全基于上下文、有无幻觉；② <b>Answer Relevancy</b>——是否直接回答用户问题；③ <b>Answer Correctness</b>——事实是否准确（更严格，原文也可能错）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">自动化框架：</b><b>RAGAS、ARES、TruLens</b> 用 LLM-as-a-Judge 把 Faithfulness/Relevancy 等指标自动化计算，提高评估效率。</div>
  </div>
</div>

---

# 1.12 RAG 部署的实际挑战？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#B26E00;">① 数据维护：</b>分块策略的<b>泛化性</b>——PDF 上效果好的策略处理 HTML/JSON 可能很差；知识库<b>实时更新</b>——源文档修改/删除时可靠更新或废弃对应向量，涉及复杂 ETL。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">② 延迟与成本：</b>「检索 + 生成」两步天然比直接调 LLM 慢，实时场景需极致优化；计算成本（嵌入、向量库、LLM 调用）和存储成本（高维向量索引）持续支出。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">③ 安全隐私：</b><b>访问控制</b>——企业环境必须集成权限体系，用户只能检索有权查看的文档；<b>提示注入</b>——恶意查询或索引文档中的恶意内容可能攻击 RAG 系统。</div>
  </div>
</div>

---

# 1.13 Query Rewrite 怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">目标：</b>问题改写目标是提升<b>检索召回</b>，不是把问题「写得更漂亮」。一般「单轮为主，多轮兜底」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">单轮改写三件事：</b>补全上下文指代、规范术语、拆分复合问题。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">多轮改写：</b>低置信度或召回差的 case 进入——先生成若干等价 query，再并行检索，最后融合去重。加约束避免偏题：<b>不改变用户意图、不引入新事实、保留关键实体词</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">效果评估：</b>召回命中率提升、相关性提升、端到端成功率变化；多轮改写提升有限但成本明显增加时回退单轮。核心原则是「<b>改写为检索服务</b>」。</div>
  </div>
</div>

---

# 1.14 Rerank 和 TopK 怎么设置？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">Rerank 位置：</b>放在「初召回之后、喂模型之前」——先向量检索召回较大候选集（Top50/100），再用重排模型按 query 相关性重新打分，截断成最终 TopK 给大模型。价值是降低噪声上下文、提升答案稳定性。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">TopK 设置：</b>不是固定拍板——事实问答 K 小一点，复杂推理 K 稍大；通过离线评测和线上 AB 找最优点，观察「正确率-时延-成本」三条曲线的拐点。K 过小漏信息，K 过大稀释注意力并拉高 token 成本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实践顺序：</b>先保证召回，再通过重排把信息密度做上来，最后调 K——而不是反过来。</div>
  </div>
</div>

---

# 1.15 向量库选型：PGVector 还是 ES？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">ES 可以做向量检索，但选 PGVector 是因为场景需要「<b>向量检索和关系数据一体化</b>」和更低接入成本：① 数据原本在 PostgreSQL 体系，减少异构系统同步和一致性治理成本；② 规模和 QPS 在 PGVector 可承受范围内，优先工程简洁；③ 很多业务查询是「结构化过滤 + 向量相似度」组合，同库更顺滑。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">ES 的优势：</b>大规模检索和复杂搜索生态——数据量上升到更高量级时，ES 或专用向量库是候选。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">面试表述：</b>选型不是绝对优劣，而是「业务规模、团队运维能力、数据一致性成本」的综合权衡。</div>
  </div>
</div>

---

# 1.16 向量检索算法有哪些？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">暴力计算（brute-force）</b>
        <div style="margin:4px 0 0;">全量比较找最接近的 K 个。结果准确无误差，但只适合小规模数据。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">LSH（局部敏感哈希）</b>
        <div style="margin:4px 0 0;">利用哈希碰撞把相似向量映射到同一桶，避免逐一比较；数据量大时需更多哈希函数保证准确率。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">PQ / IVFPQ（乘积量化）</b>
        <div style="margin:4px 0 0;">把高维向量切分多个子向量做量化编码，显著降低内存占用，适合大规模向量库；量化引入精度损失。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">HNSW（图方法）</b>
        <div style="margin:4px 0 0;">多层图结构建立「高速查询通道」，查询像图里近邻跳转——速度快、召回率高；代价是内存占用大、建索引慢、删除节点不友好。</div>
      </div>
    </div>
  </div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
