---
aliases:
- "LangChain / LangGraph / AutoGen / CrewAI 用过哪个?为什么选它,踩过什么坑?"
- "清单 2.1 LangChain / LangGraph / AutoGen / CrewAI 用过哪个?为什么选它,踩过什么坑?"
---

# 01 LangChain / LangGraph / AutoGen / CrewAI 用过哪个?为什么选它,踩过什么坑?

## 01 核心回答

一句话结论:选型看可控性 vs 上手速度:按固定版本比较状态控制、接入与生产治理能力，各框架均有适用范围,没有银弹。

LangGraph:把 Agent 建成状态图(节点=工具/模型,边=转移条件),支持循环、分支、checkpoint 持久化与人工介入,适合需要精细状态与恢复控制的场景，是否最优须验证。

LangChain:集成生态较广,上手快,但抽象层多、隐式逻辑重,需测试版本兼容并控制抽象层带来的调试成本,适合快速验证。

AutoGen:微软出品,强在多 Agent 对话式协作与代码执行,适合研究与多体研究,生产治理能力应按所用版本与需求核验。

CrewAI:角色化 Crew 分工,声明式简单,适合轻量多角色任务,也提供 Flows 状态与控制流能力，需按具体需求比较。

**踩坑点(结合自己的项目讲):**版本升级不兼容、隐式 Chain 内部逻辑难追踪、长任务 token 超限、工具调用失败缺少兜底、checkpoint 恢复不完整导致重复执行。

记忆要点:要可控上 LangGraph、要快速上 LangChain、要对话式多体用 AutoGen、要轻量分工用 CrewAI;有真实经历时讲具体证据；未使用则说明研究或实验范围。

## 02 选型陈述需要版本和证据

不能把 LangGraph 称为普遍“最好、生产首选”，也不能把其他框架统称原型。当前 LangChain 高层 Agent 基于 LangGraph，CrewAI 有支持状态与控制流的 Flows；具体能力和维护状态需看选定版本。

用同一任务对照持久恢复、人工审批、工具契约、观测、成本与升级兼容。踩坑只讲自己真实见过的现象和证据；没有实战就说做过阅读或小规模验证，不用示例虚构经历。

口述模板：“因为任务需要某种控制能力，比较后选了方案 A；已验证的好处是 X，未解决的代价是 Y。”详见[[八股/07-AI与Agent/07-框架协议与工程化/02-AgentRAG 框架如何选型|框架选型维度]]。

## 03 关联追问

- [[八股/07-AI与Agent/07-框架协议与工程化/03-LangChain 和 LangGraph 的关系与区别|LangChain 和 LangGraph 的关系与区别？]]

## 04 参考资料

- [LangChain 官方概览](https://docs.langchain.com/oss/python/langchain/overview)
- [CrewAI Flows](https://docs.crewai.com/en/concepts/flows)

## 05 所属专题

- [[八股/07-AI与Agent/11-Agent工程复习/00-Agent工程复习导航|Agent工程复习导航]]
