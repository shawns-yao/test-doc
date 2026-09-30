---
aliases:
- "如何把普通 API MCP 化？"
- "AI 9.9 如何把普通 API MCP 化？"
---

# 09 如何把普通 API MCP 化？

## 01 核心回答

**回答（包装、约束、观测三步）：**

① **包装**——把原 API 能力抽象成工具，定义清晰的输入输出 schema（必填项、类型、枚举、错误码）；② **约束**——加权限控制、参数校验和幂等键，明确哪些场景可调用、谁可调用、失败如何处理；③ **观测**——接入日志和指标（调用成功率、P95 时延、错误分布、重试次数）。

**注册与测试：**把工具描述注册到 MCP Server；上线前做两类测试——**契约测试**（保证 schema 不破）、**回放测试**（保证关键 case 稳定）。「MCP 化」不是加一层协议，而是把接口变成可被模型稳定消费的标准能力。

---

## 02 不要只翻译 HTTP 参数

先选一个明确业务意图，把底层 REST 请求映射为工具 schema、业务错误和可验证结果；说明分页、幂等、限流、数据敏感性与审批要求。SDK 能生成协议样板，但不能替接口设计合理粒度。

HTTP MCP Server 需单独验证访问令牌的受众和范围；若后端 API 需要另外的凭证，不能未经区分地透传客户端令牌。stdio 工具也应限制环境凭证与本机权限。

验收包含未授权租户、无效对象 ID、重复写入、超时后已成功、超长返回和恶意参数等负例。口述：“MCP 化的是稳定业务能力，不是把所有管理员接口直接暴露给模型。”

## 03 关联追问

- [[八股/07-AI与Agent/09-Agent项目实践/19-交易平台为什么要做 Agent HubMCP|交易平台为什么要做 Agent Hub/MCP？]]
- [[八股/08-RAG与MCP/03-MCP协议与工具/01-哪些工具适合放入MCP|哪些工具适合放入MCP]]

## 04 参考资料

- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
- [MCP 2025-11-25 Authorization 规范](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- [MCP 官方安全实践](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices)

## 05 所属专题

- [[八股/07-AI与Agent/07-框架协议与工程化/00-框架协议与工程化导航|框架协议与工程化导航]]
