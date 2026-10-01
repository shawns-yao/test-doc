---
aliases:
- "Workflow 和 Agent 怎么选？"
- "Agent 4.21 Workflow 和 Agent 怎么选？"
---

# 25 Workflow 和 Agent 怎么选？

## 01 核心回答

从任务的稳定性和开放度选：任务路径相对固定（文献检索、摘要生成、引用整理、格式输出）→ 优先 workflow（更稳定、可控、好评估）；任务开放且目标已澄清，需要系统动态拆步骤和选工具 → Agent。不会把 Agent 当默认答案——生产环境经常是 workflow 为主、Agent 为辅，用 Agent 补 workflow 覆盖不到的部分。

---

## 02 按不确定环节安排控制权

如果步骤和分支可提前定义，用工作流直接表达；如果下一步取决于新证据和开放环境，让 Agent 在限定工具与预算内选择。目标不明确时先澄清，不是越模糊越适合直接放权。

混合方案常见：固定入口鉴权与需求确认，中间 Agent 动态检索，固定出口做引用和格式检查，发布前审批。数据导出、交易、权限变化等动作即使处于 Agent 内也继续受硬规则约束。

---

## 03 追问与口述

“什么情况下升级为 Agent？”当固定流程分支维护成本很高，且动态决策带来可测成功率提升时。“如何回退？”保存当前已验证结果，将未完成项交给确定性流程或人工，而不是重头自由规划。

口述：“可确定的事情程序化，不确定的环节模型化，最终权限与验收仍是确定的。”详见[[八股/07-AI与Agent/02-Agent原理与编排/11-Chain、Agent、Workflow 三者怎么区分|Chain Agent Workflow]]。

---

## 04 参考资料

- [Anthropic 关于工作流与 Agent 的工程说明](https://www.anthropic.com/engineering/building-effective-agents)

---

## 05 所属专题

- [[八股/07-AI与Agent/09-Agent项目实践/00-Agent项目实践导航|Agent项目实践导航]]
