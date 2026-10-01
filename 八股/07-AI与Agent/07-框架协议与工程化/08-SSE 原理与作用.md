---
aliases:
- "SSE 原理与作用？"
- "AI 9.8 SSE 原理与作用？"
---

# 08 SSE 原理与作用？

## 01 核心回答

**原理：**SSE 是服务端向客户端**单向持续推送事件**的机制，适合流式文本生成——HTTP 长连接，响应头设置 `text/event-stream`，服务端按事件帧持续写入，客户端边接收边渲染。

调用中做的三件事：① 降低首字等待时间（先看到部分结果）；② 过程可视化（当前节点、工具调用中、重试中）；③ 可配合事件 ID 与服务端持久事件实现续传；业务中断和恢复需应用另行设计。

**工程注意：**心跳保活、防代理超时、事件幂等（避免重连后重复展示或重复落库）；事件分类型（token/tool_start/tool_end/final）让前端精确展示过程。

---

## 02 SSE 的恢复不是业务恢复

事件通常由空行分隔，可含 data、event、id 和 retry；浏览器 EventSource 支持重连及 Last-Event-ID 机制，但服务端必须保留可回放事件，才能真正续传。自定义 fetch 流式客户端是否自动重连要看实现。

断开连接不等于取消服务器工作，重复接收事件也不应重复触发业务写入。事件序号用于展示去重，订单幂等键用于业务去重，二者不能混为一谈。代理缓冲未关闭时，服务端已发送也可能无法立即渲染。

按 MCP 2025-11-25 规范，标准远程传输是 Streamable HTTP，可使用 SSE；旧 HTTP+SSE 是历史传输，见[[八股/08-RAG与MCP/03-MCP协议与工具/02-MCP使用了哪些协议|MCP 传输版本差异]]。

---

## 03 关联追问

- [[八股/07-AI与Agent/09-Agent项目实践/12-交易系统为什么优先用 WebSocket？REST 怎么配合|交易系统为什么优先用 WebSocket？REST 怎么配合？]]
- [[八股/07-AI与Agent/09-Agent项目实践/26-用 Python 搭 AI 服务后端|用 Python 搭 AI 服务后端？]]
- [[八股/07-AI与Agent/09-Agent项目实践/39-长时间视频生成怎样向前端报告可靠进度|长时间视频生成怎样向前端报告可靠进度]]

---

## 04 参考资料

- [WHATWG SSE 标准](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [MCP 2025-11-25 Transports 规范](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)

---

## 05 所属专题

- [[八股/07-AI与Agent/07-框架协议与工程化/00-框架协议与工程化导航|框架协议与工程化导航]]
