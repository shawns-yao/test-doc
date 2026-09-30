---
aliases:
- "MCP使用了哪些协议"
---

# 02 MCP使用了哪些协议

## 01 核心回答

按 MCP 2025-11-25 规范，消息层使用 JSON-RPC 2.0；标准传输是 stdio 和 Streamable HTTP。SSE 是 HTTP 流式响应可使用的事件格式，不能把旧版 HTTP+SSE 写成当前唯一远程传输。

## 02 三层理解

1. 消息层：request 带 id，response 对应 id，notification 没有响应 id；消息用 UTF-8 编码。工具 schema 用于描述参数与输出约束，不等于实际执行授权。
2. 生命周期：连接后初始化并协商协议版本与能力，再进行 tools/list、tools/call 等交互。客户端不能假设服务端支持所有扩展。
3. 传输层：stdio 常由客户端启动服务端子进程，经标准输入输出交换消息，普通日志应走 stderr；Streamable HTTP 让独立服务端经 HTTP endpoint 接收请求，响应可为 JSON 或 SSE，是否支持额外服务器推送取决于能力与实现。

## 03 旧 SSE 与授权边界

2024-11-05 的 HTTP+SSE 传输已被 Streamable HTTP 替代，但兼容旧客户端仍可能保留旧端点。“SSE 断开”也不天然等于业务取消或状态回滚。

HTTP 授权规范基于 OAuth 体系，授权并非所有 MCP 实现的强制统一前提，stdio 的凭证获取与 HTTP 流程不同。不要把用户给 MCP Server 的访问令牌直接透传给任意下游 API；令牌受众、scope、资源归属和下游凭证需分别校验。会话 ID 不是访问授权的替代品。

## 04 追问与口述

“能用 WebSocket 吗？”可实现自定义传输，但不能把它说成此版本规定的标准传输；需满足协议语义并考虑互操作成本。“为什么 JSON-RPC 不是 REST？”它统一方法调用、结果和通知，HTTP 只是其中一种承载方式。

口述：“MCP 是 JSON-RPC 消息加生命周期和能力约定，常用 stdio 或 Streamable HTTP 承载。SSE 是可选流式机制，授权也要区分传输和实际资源权限。”

相关：[[八股/07-AI与Agent/07-框架协议与工程化/08-SSE 原理与作用|SSE 机制]]；[[八股/07-AI与Agent/07-框架协议与工程化/05-Agent 调用 MCP 的逻辑|Agent 调用流程]]；[[八股/08-RAG与MCP/03-MCP协议与工具/03-为什么使用MCP而不是使用本地Tools或者Skills|选用 MCP 的条件]]

## 05 关联追问

- [[八股/07-AI与Agent/11-Agent工程复习/02-MCP 协议是干嘛的,最近为什么突然这么火,解决了什么之前的痛点|MCP 协议是干嘛的,最近为什么突然这么火,解决了什么之前的痛点?]]

## 06 参考资料

- [MCP 2025-11-25 Transports 规范](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)
- [MCP 2025-11-25 Authorization 规范](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- [MCP 官方安全实践](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices)

## 07 所属专题

- [[八股/08-RAG与MCP/03-MCP协议与工具/00-MCP协议与工具导航|MCP协议与工具导航]]
