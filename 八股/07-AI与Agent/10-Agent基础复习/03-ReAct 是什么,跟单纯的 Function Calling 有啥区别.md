---
aliases:
- "ReAct 是什么,跟单纯的 Function Calling 有啥区别?"
- "清单 1.3 ReAct 是什么,跟单纯的 Function Calling 有啥区别?"
---

# 03 ReAct 是什么,跟单纯的 Function Calling 有啥区别?

## 01 核心回答

一句话结论:ReAct 是"思考→行动→观察"的多轮推理范式,将推理候选与行动、观察交替组织，不要求向用户披露隐藏思维链;Function Calling 是模型一次输出"调哪个工具+参数"的接口能力,本身不含多轮推理循环。

**ReAct:**每一轮先输出 Thought(推理),再选 Action(调工具),拿到 Observation(结果)进入下一轮,可回溯纠错,适合多步推理、易出错需修正的复杂任务。

Function Calling:模型在生成时按注入的工具 Schema 输出结构化调用(名称+参数),生成调用意图，由运行时校验并执行,不强制暴露中间推理,可用于单次或多轮任务，成本与延迟取决于整体控制流程。

记忆要点:两者常搭配使用——用 Function Calling 做工具解析与执行,用 ReAct 思路做多步规划,不是二选一(展开见下方"题目 1")。

## 02 接口与策略可以组合

Function Calling 描述模型输出工具调用结构的能力，运行时真正执行。它可以被多轮调用，也可用于复杂规划系统，并不天然限制单步或保证更低延迟。ReAct 描述推理候选、行动与观察交替的组织方式。

论文中的 Thought 是生成的中间文本，不能等同忠实内部思维；产品可只展示计划摘要、动作与证据，不要求暴露隐藏思维链。用相同模型、工具和预算比较终态成功率，才有意义。

口述：“Function Calling 是调用表达方式，ReAct 是交互循环，两者属于不同层。”详解与追问见[[八股/07-AI与Agent/15-Agent专题复盘/01-题目 1ReAct 和纯 Function Calling 到底有什么区别|ReAct 对比复盘]]。

## 03 参考资料

- [ReAct 原论文](https://arxiv.org/abs/2210.03629)
- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)

## 04 所属专题

- [[八股/07-AI与Agent/10-Agent基础复习/00-Agent基础复习导航|Agent基础复习导航]]
