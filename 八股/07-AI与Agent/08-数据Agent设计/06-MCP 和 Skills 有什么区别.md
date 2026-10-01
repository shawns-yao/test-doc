---
aliases:
- "06-MCP 和 Skills 的区别？LangChain 和 LangGraph 的区别？短期记忆和长期记忆怎么划分"
- "MCP 和 Skills 的区别？LangChain 和 LangGraph 的区别？短期记忆和长期记忆怎么划分？"
- "Agent 1.6 MCP 和 Skills 的区别？LangChain 和 LangGraph 的区别？短期记忆和长期记忆怎么划分？"
---

# 06 MCP 和 Skills 的区别

## 01 核心回答


MCP（Model Context Protocol）是客户端接入外部工具、资源等能力的开放协议，统一发现和调用的消息约定。Skills 是任务能力封装，把指令、流程、参考文档及可选脚本打包为可复用材料。MCP 侧重“怎么连”，Skill 侧重“怎么干”，两者可以组合。

原综合题中独立的框架和记忆问题已分开：[[八股/07-AI与Agent/08-数据Agent设计/07-数据Agent中如何选择LangChain和LangGraph|数据Agent中如何选择LangChain和LangGraph]]；[[八股/07-AI与Agent/08-数据Agent设计/08-数据Agent的短期与长期记忆如何划分|数据Agent的短期与长期记忆如何划分]]。原复合问题与旧文件名已保留为 aliases，便于追溯。

---

## 02 直接追问

“与插件系统区别？”插件可能把 MCP、Skill 和界面封在一起，不能把所有插件都认定为私有协议。“加载 Skill 是否就能访问数据库？”不能，实际能力和凭证仍需独立授权；Skill 中的脚本也要符合执行限制。

在数据 Agent 中，Skill 可规定指标口径校验流程，MCP 提供数据库或元数据工具，本地函数做纯计算。是否引入 MCP 看跨客户端复用与隔离需求，而不是认为它自动提高 SQL 准确率。

相关：[[八股/08-RAG与MCP/03-MCP协议与工具/03-为什么使用MCP而不是使用本地Tools或者Skills|本地工具 MCP Skill 的取舍]]。

---

## 03 关联追问

- [[八股/07-AI与Agent/07-框架协议与工程化/12-MCP、Skill、RAG 的关系|MCP、Skill、RAG 的关系？]]

---

## 04 参考资料

- [Agent Skills 规范](https://agentskills.io/specification)
- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)

---

## 05 所属专题

- [[八股/07-AI与Agent/08-数据Agent设计/00-数据Agent设计导航|数据Agent设计导航]]
