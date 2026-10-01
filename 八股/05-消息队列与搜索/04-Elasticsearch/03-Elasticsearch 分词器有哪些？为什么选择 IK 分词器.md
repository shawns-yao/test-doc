---
aliases:
- "Elasticsearch 分词器有哪些？为什么选择 IK 分词器？"
- "ES 1.3 Elasticsearch 分词器有哪些？为什么选择 IK 分词器？"
---

# 03 Elasticsearch 分词器有哪些？为什么选择 IK 分词器？

## 01 核心回答


Elasticsearch 的 Analyzer 通常由字符过滤器、Tokenizer 和 Token Filter 组成。常见内置分析器包括：`standard` 按 Unicode 规则切词并转小写；`simple` 按非字母字符切分；`whitespace` 按空格切分；`keyword` 把整个字段当成一个词项，适合编号、状态和标签；还有 `pattern`、语言分析器等；`ngram` 和 `edge_ngram` 属于可组合使用的 tokenizer/token filter，不应直接列作同名内置 analyzer。也可以组合大小写转换、停用词、同义词和词干处理构建自定义分析器。

中文没有天然空格，标准分词器通常会按单字或不符合业务语义的方式切分，因此中文搜索可以评估 IK 等中文分析方案；IK 是第三方插件，不是 Elasticsearch 官方内置分析器。`ik_smart` 倾向于较粗粒度切分，词项少、索引体积较小，适合偏精确的搜索；`ik_max_word` 会尽可能细粒度切分，通常产生更多候选词项，可能提高召回，但实际准确率、索引与查询成本要用业务样本验证。实际项目可以为同一字段建立多个子字段，索引和查询时按场景选择不同分析器。

选择 IK 的原因是它对中文词语识别、扩展词典和停用词支持较好，接入成本也较低。但分词器不能只看默认效果，业务专有名词、人名、缩写和新词需要维护自定义词典。还要保证索引阶段和查询阶段的分词策略兼容，并用真实搜索样本评估召回率、准确率、零结果率和搜索延迟。

---

## 02 分析器的三段流水线

一个 analyzer 由零或多个 char filter、一个 tokenizer、零或多个 token filter 构成。字符过滤先处理原文，例如移除 HTML；tokenizer 决定切分边界与位置；token filter 再处理大小写、停用词或同义词。whitespace 不自动小写，而 simple 会小写，因此“都是按空格分词”不足以说明行为。

还需区分 keyword analyzer 与 keyword 字段类型：前者输出一个完整词项，后者是一种适合精确值查询、排序和聚合的字段建模选择，不是给 text 换个 analyzer 就自动获得相同结构。

依据：[官方内置分析器与自定义组成](https://www.elastic.co/docs/reference/text-analysis/analyzer-reference)、[Keyword 字段](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/keyword)。

---

## 03 选择 IK 的证据与版本边界

IK 由 INFINI Labs 维护，提供 ik_smart 与 ik_max_word 及词典扩展；安装必须匹配 Elasticsearch 版本，托管环境是否允许该插件也要事先验证。官方另有 Smart Chinese 插件可作为对照。选择哪一个应由真实中文词、专有名词、数字字母混排、短语和错误输入的评测决定。

常见组合是索引使用较细切分、查询使用较粗切分，但不是通用最优解。两个模式并不能简单理解为严格包含关系，短语匹配还依赖 position 等信息；必须用 _analyze 检查实际词项和位置，再用标注查询集验证召回与排序。

修改词典只影响之后进行的分析，不会自动把已存倒排索引重新切词。若改变索引侧分词语义，应评估重建索引；查询侧词典变化则可能立即改变检索结果，适合灰度、回归与回滚。

依据：[IK 官方项目说明与兼容版本](https://github.com/infinilabs/analysis-ik)、[Elastic Smart Chinese 插件](https://www.elastic.co/docs/reference/elasticsearch/plugins/analysis-smartcn)。

---

## 04 追问与口述

- edge_ngram 为什么不能盲用？它适合前缀输入联想，gram 范围决定索引膨胀和可匹配长度；中文词义切分是另一个问题
- 分词器改好后旧商品仍搜不到怎么办？检查旧文档是否已经按新规则重建、查询是否命中正确字段，不能只重启服务
- 同义词是不是越多越好？歧义会降低精确度，并改变评分与短语行为，需要定向扩展和评测

口述：“我先区分 analyzer 和 tokenizer，再说明中文切词与业务词典需求。IK 是一个可评估的第三方方案，用 _analyze 和真实检索集验证，而不是认定 ik_max_word 永远最好。上线同时管理插件版本、词典一致性以及旧索引重建。”

补充依据：[Edge n-gram tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html)。

---

## 05 相关问题与延伸

- [[八股/05-消息队列与搜索/04-Elasticsearch/04-Elasticsearch 数据写入与查询原理|Elasticsearch 数据写入与查询原理]]：文本分析影响索引与查询

---

## 06 所属专题

- [[八股/05-消息队列与搜索/04-Elasticsearch/00-Elasticsearch导航|Elasticsearch导航]]
