---
aliases:
- "Multi-Agent 实际项目怎么设计？LangGraph 里怎么编排更稳？"
- "AI 4.10 Multi-Agent 实际项目怎么设计？LangGraph 里怎么编排更稳？"
---

# 10 Multi-Agent 实际项目怎么设计？LangGraph 里怎么编排更稳？

## 01 核心回答

**回答（角色分工 + 状态编排 + 失败恢复）：**

**① 角色分工：**典型角色是 Planner（规划）、Executor（执行）、Critic（审校）、Tool Agent（外部能力）。

**② 状态编排：**状态层用图或状态机管理节点流转；每个 Agent 有明确**输入输出契约**，避免互相污染上下文。

**③ 失败恢复：**每个节点配超时、重试、步数上限、防循环；高风险任务接**人工接管（human-in-the-loop）**；上线后通过 trace 做全链路观测，重点看任务成功率、时延、成本和重试率。

**LangGraph 编排（主控图 + 专家节点）：**Planner 只拆解任务、Dispatcher 负责路由、Executor 只执行、Critic 只审校。① 全局状态只放**最小字段**，大文本走引用；② 图上配置**最大步数、重试上限、超时和预算**；③ 写操作加**幂等键**并做 checkpoint——失败后可恢复；副作用是否重复还取决于业务幂等和结果核验；④ 上线后按任务成功率、改写率、时延和成本做节点级观测。

**口述重点：**让 Agent 像微服务一样可组合、可回放、可治理；多 Agent 最容易坏在状态编排层（Orchestration），失败大多发生在「交接」。

---

## 02 恢复与状态的关键细节

Checkpoint 只保存运行状态，不保证外部副作用只发生一次。节点调用成功但检查点未落盘时，恢复可能重放；必须用业务幂等键、持久结果记录和状态回查处理这一窗口。不能把“有 checkpoint”直接说成“不会重复执行”。

并行节点写同一字段时，应定义 reducer 或单写者规则；列表拼接、集合合并和覆盖更新的语义不同。子 Agent 产物需通过验收后合入主状态，权限不应因层层委派而扩大。

追问“Critic 能确保正确吗？”它只是一个评价组件，关键产物仍需测试、规则或事实证据验证。参见[[八股/07-AI与Agent/02-Agent原理与编排/21-Agent的checkpoint是什么|checkpoint 的能力边界]]。

---

## 03 关联追问

- [[八股/07-AI与Agent/07-框架协议与工程化/04-LangGraph 生产治理怎么做|LangGraph 生产治理怎么做？]]
- [[八股/07-AI与Agent/12-多Agent复习/02-Agent 之间怎么通信、怎么分工听说过 Orchestrator 模式吗|Agent 之间怎么通信、怎么分工?听说过 Orchestrator 模式吗?]]

---

## 04 参考资料

- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)

---

## 05 所属专题

- [[八股/07-AI与Agent/02-Agent原理与编排/00-Agent原理与编排导航|Agent原理与编排导航]]
