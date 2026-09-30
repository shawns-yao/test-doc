---
aliases:
- "ReAct 是什么？Agent 的规划能力怎么设计？"
- "AI 4.2 ReAct 是什么？Agent 的规划能力怎么设计？"
---

# 02 ReAct 是什么？Agent 的规划能力怎么设计？

## 01 核心回答

**ReAct：**是 **Thought-Action-Observation 循环**——先思考、再调用工具、再根据观察结果迭代。比单次 CoT 更适合信息不完整任务，因为能**边查边改**；代价是链路更长，线上要控制步数、超时和成本。

**规划方法（线性到搜索）：**① **CoT** 单路径分步推理，成本低但容错弱；② **ToT** 树状思考引入多分支探索和回溯，可能提高成功率但算力开销大；③ **GoT** 图结构允许分支合并和循环优化，适合复杂依赖问题。

工程实践：把「规划」和「执行」解耦——Planner 负责拆解、Executor 负责落地（可为阶段划分，不必是多个 Agent），减少模型一步到位失败。

**口述重点：**规划能力需要任务分解、反馈与可靠验收；更多搜索只是可选手段。

---

## 02 勘误与直接追问

ReAct 将语言层面的推理候选与实际行动、环境观察交替组织；论文中的 Thought 文本不能直接当成模型真实内部思考，也不要求产品披露隐藏思维链。可对外展示简明计划、工具事件和证据。

ToT/GoT 的收益取决于候选生成、评价器和预算，多分支不保证成功率更高。Planner/Executor 也可以只是同一服务的两个阶段，无需一定拆成多 Agent。规划的目标是减少漏项、管理依赖和纠错，不能简化成“越多搜索越聪明”。

“Function Calling 与 ReAct 区别？”前者是结构化表达工具调用的能力，后者是组织多轮推理行动的策略；一次函数调用不是完整 ReAct，ReAct 也可通过 Function Calling 实现。

比较两种规划路线：[[八股/07-AI与Agent/02-Agent原理与编排/18-plan-and-solve 和 Tree-of-Thoughts分别是什么|PS 与 ToT]]

## 03 关联追问

- [[八股/07-AI与Agent/15-Agent专题复盘/01-题目 1ReAct 和纯 Function Calling 到底有什么区别|题目 1:ReAct 和纯 Function Calling 到底有什么区别?]]

## 04 参考资料

- [ReAct 原论文](https://arxiv.org/abs/2210.03629)
- [Tree of Thoughts 原论文](https://arxiv.org/abs/2305.10601)

## 05 所属专题

- [[八股/07-AI与Agent/02-Agent原理与编排/00-Agent原理与编排导航|Agent原理与编排导航]]
