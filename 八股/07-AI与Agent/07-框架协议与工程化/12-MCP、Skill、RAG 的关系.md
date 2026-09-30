---
aliases:
- "MCP、Skill、RAG 的关系？"
- "AI 9.12 MCP、Skill、RAG 的关系？"
---

# 12 MCP、Skill、RAG 的关系？

## 01 核心回答

**Skill** 更像「方法论」或「流程模板」，定义模型遇到某类任务时按什么步骤做——适合封装重复性流程和团队经验；**RAG** 解决「知识从哪里来」——把当前任务相关且可能实时变化的外部信息检索出来喂给模型。

**关系：**Skill 是能力封装，RAG 是知识供给，两者配合不是替代——一个 Skill 内部常规定「先检索、再筛选、再生成、再校验」，其中的「检索」往往就是 RAG。

---

## 02 补齐 MCP 的角色

MCP 管客户端与工具/资源服务的标准交互；Skill 封装使用方法、说明及可选资源/脚本；RAG 检索证据供生成使用。一个 Skill 可以要求先检索知识，检索能力通过 MCP 暴露，也可以直接调用本地函数。

例如“解答产品政策” Skill 定义来源与引用要求，MCP 提供受权限保护的文档查询，RAG 负责挑选材料并形成答案。任何一层都不能用自然语言说明替代真实鉴权。

追问“Skill 和 RAG 是否都只是塞文档？”Skill 主要影响任务方法，RAG 主要提供问题相关事实，但可组合；不是每份加载文档都应视作可信指令。详见[[八股/08-RAG与MCP/03-MCP协议与工具/03-为什么使用MCP而不是使用本地Tools或者Skills|三者的接入取舍]]。

## 03 关联追问

- [[八股/07-AI与Agent/08-数据Agent设计/06-MCP 和 Skills 有什么区别|MCP 和 Skills 的区别]]

## 04 参考资料

- [Agent Skills 规范](https://agentskills.io/specification)
- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
- [RAG 原论文](https://arxiv.org/abs/2005.11401)

## 05 所属专题

- [[八股/07-AI与Agent/07-框架协议与工程化/00-框架协议与工程化导航|框架协议与工程化导航]]
