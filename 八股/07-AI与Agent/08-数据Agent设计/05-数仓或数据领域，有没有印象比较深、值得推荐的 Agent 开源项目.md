---
aliases:
- "数仓或数据领域，有没有印象比较深、值得推荐的 Agent 开源项目？"
- "Agent 1.5 数仓或数据领域，有没有印象比较深、值得推荐的 Agent 开源项目？"
---

# 05 数仓或数据领域，有没有印象比较深、值得推荐的 Agent 开源项目？

## 01 核心回答


研究方向：Text-to-SQL 可参考 Vanna、DB-GPT、Dataherald 等历史或当前项目，元数据治理可参考 DataHub、OpenMetadata；维护与许可需分别核实，Vanna 官方仓库已归档，见下文核验。

**评估标准：**选择时不只看 Demo，重点评估 **Schema 检索、权限隔离、SQL 校验、执行反馈、评测集和可观测性**。

**判断：**开源项目适合用来验证交互和基线，生产系统仍需要结合企业元数据、权限和数据质量体系做定制。

**面试追问：**为什么 NL2SQL 项目都强调 Schema 检索（不检索就幻觉）；开源和自研怎么选（开源做基线，核心定制自研）；权限隔离为什么难（企业级行级/列级权限）。

---

## 02 项目定位与时效核验

截至 2026-09-30 的官方仓库核验，vanna-ai/vanna 已于 2026-03-29 归档为只读，可用于学习 Text-to-SQL 的交互与权限设计，但不能无条件推荐为持续维护中的生产依赖。DB-GPT 可作为数据应用与 Text-to-SQL 工程参考；DataHub、OpenMetadata 首先是元数据治理平台，不应一概称为同类 Agent 框架。Dataherald 等候选需在选型时另查维护与许可状态。

## 03 面试怎么讲得具体

只挑实际阅读或运行过的项目，解释请求如何进入、如何获得 schema、生成 SQL 后如何校验，以及用户身份在哪一步影响查询。未用过就明确说“研究过设计”，不要背“踩坑经历”。

生产试验用合成或授权数据，测试跨租户、错误 schema、超时、复杂 JOIN 和权限负例；记录依赖锁定、升级路线与退出成本。仓库 star 和 Demo 准确率不能代替这些验证。

口述：“我会区分数据底座与 Agent 框架，挑一个具体机制讲清楚，再说明当前维护状态和生产差距。”

## 04 关联追问

- [[八股/07-AI与Agent/08-数据Agent设计/01-如果让你提升 Agent 生成 SQL 或数据链路的准确率，你会怎么做|如果让你提升 Agent 生成 SQL 或数据链路的准确率，你会怎么做？]]

## 05 参考资料

- [Vanna 官方仓库与归档状态](https://github.com/vanna-ai/vanna)
- [DB-GPT 官方仓库](https://github.com/eosphoros-ai/DB-GPT)
- [DataHub 官方仓库](https://github.com/datahub-project/datahub)
- [OpenMetadata 标准](https://docs.open-metadata.org/v2.0.x/main-concepts/metadata-standard)

## 06 所属专题

- [[八股/07-AI与Agent/08-数据Agent设计/00-数据Agent设计导航|数据Agent设计导航]]
