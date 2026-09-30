---
aliases:
- "MCP 协议是干嘛的,最近为什么突然这么火,解决了什么之前的痛点?"
- "清单 2.2 MCP 协议是干嘛的,最近为什么突然这么火,解决了什么之前的痛点?"
---

# 02 MCP 协议是干嘛的,最近为什么突然这么火,解决了什么之前的痛点?

## 01 核心回答

一句话结论:MCP(Model Context Protocol)是AI 应用连接外部工具/数据的统一开放协议,相当于"AI 界的 USB-C",由 Anthropic 提出，互操作支持范围需按版本核验。

解决的历史痛点:此前每个 Agent 对接工具都要自研适配器,工具格式、鉴权、发现机制各自为政,接入成本高、生态碎片化,更换框架时可能需要重复适配。

它定义了什么:统一了 Server(提供工具/资源/提示词)— Client（宿主 Host 内的协议组件） 的通信规范,工具以标准化 Schema 暴露、动态发现与调用,支持本地与远程。

为什么火了:Anthropic 开源推行,OpenAI/Google 等跟进兼容,有望复用接入，但版本、能力支持以及业务鉴权和观测实现仍需验证。

记忆要点:USB-C 类比 + 三个关键词(统一协议、解耦工具与框架、生态互通);再补一句它和 Function Calling 不冲突,FC 是模型能力、MCP 是接入标准。

## 02 热度与标准化的边界

MCP 源自 Anthropic 提出的开放协议，生态采用情况有时效；“事实标准”“换框架全部重接”“一次接入处处复用”都不能作无条件保证。客户端支持的版本、原语和扩展不同，仍需要兼容测试。

Host 是应用，Client 是其中连接 MCP Server 的协议组件，不应混为一个角色。协议统一工具/资源交互，但业务鉴权、配额、审计实现和风险治理仍由应用与服务端负责。

口述：“它减少重复适配，价值来自可互操作的能力接口；热门与否不替代协议版本和安全验证。”见[[八股/08-RAG与MCP/03-MCP协议与工具/02-MCP使用了哪些协议|传输与授权]]。

## 03 关联追问

- [[八股/07-AI与Agent/07-框架协议与工程化/06-如何理解 MCP？它解决什么问题|如何理解 MCP？它解决什么问题？]]

## 04 参考资料

- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
- [MCP 2025-11-25 Transports 规范](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)
- [MCP 2025-11-25 Authorization 规范](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)

## 05 所属专题

- [[八股/07-AI与Agent/11-Agent工程复习/00-Agent工程复习导航|Agent工程复习导航]]
