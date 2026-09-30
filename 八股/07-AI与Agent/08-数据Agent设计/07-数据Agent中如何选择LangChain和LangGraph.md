---
aliases:
- "数据 Agent 中如何选择 LangChain 和 LangGraph"
- "Agent 1.6 LangChain vs LangGraph"
---

# 07 数据 Agent 中如何选择 LangChain 和 LangGraph

## 01 核心回答

原题来源：公司标签为字节飞书、得物、招银；由原综合题中的 LangChain vs LangGraph 独立整理。

LangChain 提供模型、工具及高层 Agent 接口，也支持可组合调用；LangGraph 以节点、边和状态实现显式编排，支持条件分支、循环、人机协同和持久化。当前 LangChain 高层 Agent 建立在 LangGraph 之上，不能简单说前者只适合线性固定流程。

需要精细控制循环反思、多 Agent 协作和中断恢复时，可直接设计 LangGraph 状态图；简单标准 Agent 可以先使用高层接口，避免过度建模。

## 02 数据任务中的选择

例如“理解口径 → SQL 草稿 → 只读校验 → 人工确认 → 发布”需要状态和审批版本，可用显式图。每个节点保存可检查的产物，循环纠错有上限；checkpoint 不保证业务动作自动回滚或不重复。

追问“LangGraph 循环怎么防死循环？”用明确成功/阻塞/失败终态、最大步数与预算，并检测是否产生新证据。口述：“先评估控制需求，再选高层接口或自定义图。”

互相参照：[[八股/07-AI与Agent/08-数据Agent设计/06-MCP 和 Skills 有什么区别|能力接入]]；[[八股/07-AI与Agent/08-数据Agent设计/08-数据Agent的短期与长期记忆如何划分|状态与记忆]]；[[八股/07-AI与Agent/07-框架协议与工程化/03-LangChain 和 LangGraph 的关系与区别|框架关系详解]]。

## 03 参考资料

- [LangChain 官方概览](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)

## 04 补充原稿中的组合方式

原稿提到的 LCEL（LangChain Expression Language）是 LangChain 组合 Runnable 链路的一种表达方式，可用于把输入处理、模型调用、解析等连接起来。是否采用它应看所用版本及团队习惯；图状态、循环、检查点和持久化恢复仍需按工作流需求单独设计，不能把 LCEL 名称等同于全部 Agent 编排能力。

## 05 所属专题

- [[八股/07-AI与Agent/08-数据Agent设计/00-数据Agent设计导航|数据Agent设计导航]]
