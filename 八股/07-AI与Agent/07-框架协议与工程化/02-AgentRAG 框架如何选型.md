---
aliases:
- "Agent/RAG 框架如何选型？"
- "AI 9.2 Agent/RAG 框架如何选型？"
---

# 02 Agent/RAG 框架如何选型？

## 01 核心回答

**回答（问题驱动而非框架驱动）：**

LangChain：提供模型、工具和高层 Agent 等应用抽象，当前高层 Agent 建立在 LangGraph 之上；LlamaIndex：专注外部数据的「数据」框架（Connectors/Indexes/Retrievers/Query Engines），强在数据接入、索引构建和检索优化——适合知识库问答与高质量 RAG。实际项目常组合用：LlamaIndex 管数据能力 + LangChain 做上层编排。

**LangGraph：**面向有状态、多步骤、可恢复任务的图编排框架；常见是「LangChain 组件 + LangGraph 托管状态机」。

RAGFlow 可用于端到端 RAG 平台式体验；Haystack 是组件化 AI 编排框架，不能笼统归为必带 GUI 的平台。选型需按所用版本验证解析、编排、治理与交付方式。策略：初期用 RAGFlow 搭基线验证价值，深度优化时再用代码重构实现。

**评价指标：**任务成功率、资源消耗（token/API 费用/延迟/步骤数）、鲁棒性与可预测性（错误处理、输出一致性、安全评估）——不看单点 Demo 漂亮程度。

---

## 02 框架定位与版本边界

当前 LangChain 提供高层 Agent 能力，其 Agent 运行建立在 LangGraph 之上；LangGraph 也可独立使用。LlamaIndex 不只有索引，亦有工作流和 Agent 能力，因此这些定位是侧重点，不是互斥功能清单。

Haystack 主要是组件化开源 AI 编排框架，不宜与 RAGFlow 一并归为必带 GUI 的端到端平台。选型应实际验证连接器、状态恢复、流式输出、评测集成、权限与升级兼容，而非只比较介绍页。

可改编口述：“先用相同任务做最小基线和故障注入，记录交付成本与生产边界。框架不能替应用保证幂等、权限和数据质量。”

## 03 关联追问

- [[八股/07-AI与Agent/11-Agent工程复习/01-LangChain LangGraph AutoGen CrewAI 用过哪个为什么选它,踩过什么坑|LangChain / LangGraph / AutoGen / CrewAI 用过哪个?为什么选它,踩过什么坑?]]

## 04 参考资料

- [LangChain 官方概览](https://docs.langchain.com/oss/python/langchain/overview)
- [Haystack 官方概览](https://docs.haystack.deepset.ai/docs/intro)

## 05 直接追问：如何迁移框架并避免技术锁定

把业务输入输出、工具契约、证据引用和验收规则放在独立模块，通过适配层连接框架；避免把框架消息对象直接当数据库业务格式。持久状态必须带 schema 与图版本，不能假定新框架能直接读取旧 checkpoint。

迁移时先选一条低风险链路，用固定样本和模拟工具对照旧实现的结果、路由、错误处理、时延及成本；通过后再小流量切换。含真实写操作的影子测试只记录拟执行动作，不能新旧两套同时提交。回退要同时恢复代码、Prompt、配置和兼容状态，旧长任务可排空或保留旧执行器，而不是只切回一个模型名。

框架能否替换，最终用一次受控适配实验验证。接口抽象过度也有成本，优先隔离最可能变化的供应商和运行时边界。

来源与改写说明：本节依据 [goehou/agent_java_offer 仓库贡献者的题目与资料](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/01_AI/08_%E6%A1%86%E6%9E%B6%E5%8D%8F%E8%AE%AE%E4%B8%8E%E5%B7%A5%E7%A8%8B%E5%8C%96/01_%E6%A0%B8%E5%BF%83%E9%97%AE%E7%AD%94.md#L78-L91)（[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)）补充。补充源资料的框架迁移、回滚与技术锁定追问；与原文选型结论衔接，不重复比较框架品牌。 许可说明仅对应本节引入的来源内容，不改变本篇其他原有内容的许可。

## 06 所属专题
- [[八股/07-AI与Agent/07-框架协议与工程化/00-框架协议与工程化导航|框架协议与工程化导航]]

## 07 补充分块操作与框架控制力

RAGFlow 提供模板化分块、分块可视化与人工干预，可用于快速验证检索基线；代码框架便于细化定制，易用性与控制力应在同一任务中比较。这些具体能力不应笼统归到 Haystack 或所有 RAG 框架名下。

- [原始资料](https://github.com/infiniflow/ragflow)
