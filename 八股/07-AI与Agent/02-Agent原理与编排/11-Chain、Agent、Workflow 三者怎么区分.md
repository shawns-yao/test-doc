---
aliases:
- "Chain、Agent、Workflow 三者怎么区分？"
- "AI 4.11 Chain、Agent、Workflow 三者怎么区分？"
---

# 11 Chain、Agent、Workflow 三者怎么区分？

## 01 核心回答

**Chain** 是固定顺序管道，适合确定性流程；**Agent** 是模型驱动决策，适合开放任务；**Workflow**（特别是 LangGraph）是显式状态机，适合把 Agent 决策放进可控节点里。

**LangGraph 的价值：**把「多 Agent 协作」从隐式流程变成**显式状态机**——明确状态与节点边界（减少上下文串台）、条件路由可控（避免无意回环）、checkpointer 支持中断后续跑、关键节点可 interrupt 人工审核、每步输入输出可追踪。

**框架不替你解决的坑：**I/O 契约没定义清楚、工具调用非幂等（重试导致重复副作用）、没有超时/最大步数/重试上限、状态字段设计混乱。

**限制 Agent 自由度：**工具白名单和最小权限放行 + 限制最大步数、最大调用次数、超时和成本 + 高风险动作人工审批 + 全链路监控告警——可追溯、可回滚、可治理。

---

## 02 不要把概念绑定死在框架名上

Chain 常表示按固定顺序组合调用；Workflow 可以有条件分支、并行和循环，不一定是单线；Agent 的特点是模型在边界内动态决定下一步。实际系统可在一个工作流节点中运行 Agent，也可在 Agent 工具里执行固定链。

LangGraph 可以实现确定性工作流或 Agent，不是只服务多 Agent。判断该选哪种形态，应看分支是否事先已知、是否需要动态工具选择以及恢复粒度；如果明确的业务规则就能完成路由，不必强行让模型决策。

口述：“固定规则尽量写成可测试流程，不确定环节留给模型，执行边界仍由程序控制。”

## 03 关联追问

- [[八股/07-AI与Agent/09-Agent项目实践/25-Workflow 和 Agent 怎么选|Workflow 和 Agent 怎么选？]]
- [[八股/07-AI与Agent/10-Agent基础复习/01-Agent 和普通 Chatbot、和传统 RPA 的本质区别是什么|Agent 和普通 Chatbot、和传统 RPA 的本质区别是什么?]]

## 04 参考资料

- [LangChain 官方概览](https://docs.langchain.com/oss/python/langchain/overview)
- [Anthropic 关于工作流与 Agent 的工程说明](https://www.anthropic.com/engineering/building-effective-agents)

## 05 所属专题

- [[八股/07-AI与Agent/02-Agent原理与编排/00-Agent原理与编排导航|Agent原理与编排导航]]
