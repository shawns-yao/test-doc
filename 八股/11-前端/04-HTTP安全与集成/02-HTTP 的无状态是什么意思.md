---
aliases:
- "HTTP 的无状态是什么意思？"
- "前端 4.2 HTTP 的无状态是什么意思？"
---

# 02 HTTP 的无状态是什么意思？

## 01 核心回答


**概念：**无状态 = HTTP 协议**不要求保留跨请求会话上下文**，应用仍可用 Session、数据库等维护业务状态。好处：易于水平扩展（任何节点可处理任意请求）、协议简单。

**会话维持方案（弥补无状态）：**① **Cookie + Session**：服务端存 Session（内存/Redis），Cookie 带 sessionId；② **Token（JWT）**：可减少会话查询，撤销等机制仍可能维护服务端状态，签名令牌自包含用户信息，放 Header/Cookie；③ **本地存储**：localStorage/IndexedDB 存客户端状态。

**前后端配合：**前端把登录态（Cookie/Token）随每次请求携带；服务端鉴权后放行——"无状态协议 + 有状态业务"靠会话机制桥接。

**追问：**Session 和 JWT 的区别（服务端存储 vs 自包含签名）；无状态为什么利于扩容（无粘性，任意节点处理）；Token 放哪安全（HttpOnly Cookie 防 XSS 读取）。

## 02 无状态不等于服务端没有数据库

订正：HTTP 无状态指协议不要求依赖此前请求的会话上下文来解释当前请求，并不禁止服务器保存用户资料、购物车、日志或 Session。TCP 长连接、HTTP 连接复用与应用会话状态也属于不同层面。

JWT 可以让部分身份校验不查 Session，但刷新令牌、撤销列表、账户禁用和权限更新仍可能需要服务端状态。localStorage 只是浏览器存储方式，本身既不能证明身份，也不能代替服务器鉴权。

## 03 Cookie 的两种风险

HttpOnly 能阻止脚本直接读取 Cookie，却不能阻止 XSS 代码在同源页面发起已登录请求；浏览器自动携带 Cookie 又涉及 CSRF，需要另行处理。无状态架构是否利于扩容，要看共享会话、缓存与一致性设计，不能仅由采用 JWT 得出结论。

## 04 参考

- [RFC 9110：HTTP 无状态性质](https://www.rfc-editor.org/rfc/rfc9110.html)
- [OWASP：Cookie 会话与 CSRF 防护](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

## 05 相关问题与延伸

- [[八股/11-前端/04-HTTP安全与集成/08-为什么选择 JWT？登录鉴权流程是什么？JWT 的 JSON 中一般包含哪些属性|为什么选择 JWT？登录鉴权流程是什么？JWT 的 JSON 中一般包含哪些属性]]：无状态协议与应用认证状态

## 06 所属专题

- [[八股/11-前端/04-HTTP安全与集成/00-HTTP安全与集成导航|HTTP安全与集成导航]]
