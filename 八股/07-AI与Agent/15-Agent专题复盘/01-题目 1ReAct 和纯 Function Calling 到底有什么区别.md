---
aliases:
- "题目 1:ReAct 和纯 Function Calling 到底有什么区别?"
- "清单 7.extra27 题目 1:ReAct 和纯 Function Calling 到底有什么区别?"
---

# 01 题目 1:ReAct 和纯 Function Calling 到底有什么区别?

## 01 核心回答

**回答思路:**

先说结论——ReAct 是一种"思考-行动-观察"循环的范式,将推理候选、工具行动(Action)与环境观察(Observation)交替组织；不要求披露隐藏思维链;

而纯 Function Calling 更多是模型直接判断"该不该调、调哪个",不强制暴露中间推理链路。

可以补一句:ReAct 更适合需要多步推理、容易出错要回头修正的复杂任务,Function Calling 可承载单步或多轮工具决策,两者其实经常是搭配着用的,不是二选一。

---

## 02 可直接口述的完整版本

“ReAct 是交替组织推理候选、行动与观察的策略，Function Calling 是模型表达工具调用的接口能力。运行时可以把 Function Calling 放进 ReAct 风格循环中：先选工具，执行器校验并执行，再将结果送回模型。两者没有互斥关系。”

继续追问时强调：Function Calling 可多轮、多工具；ReAct 论文的中间文字不等于真实内部思维，产品不必披露隐藏思维链。是否适合任务，应测同预算下成功率和失败恢复，而不是认为写出更多推理文字一定更稳。

例子可改编：查一个确定字段通常一次工具足够；比较多个相互矛盾来源可能需要观察反馈后补查。相关：[[八股/07-AI与Agent/10-Agent基础复习/03-ReAct 是什么,跟单纯的 Function Calling 有啥区别|基础对比]]；[[八股/07-AI与Agent/02-Agent原理与编排/02-ReAct 是什么？Agent 的规划能力怎么设计|规划策略]]。

---

## 03 参考资料

- [ReAct 原论文](https://arxiv.org/abs/2210.03629)
- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)

---

## 04 所属专题

- [[八股/07-AI与Agent/15-Agent专题复盘/00-Agent专题复盘导航|Agent专题复盘导航]]
