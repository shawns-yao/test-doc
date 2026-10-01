---
aliases:
- "什么是 XSS 和 CSRF？分别如何防护？"
- "安全 1.2 什么是 XSS 和 CSRF？分别如何防护？"
---

# 02 什么是 XSS 和 CSRF？分别如何防护？

## 01 核心回答


**概念原理（XSS）：**跨站脚本攻击，攻击者把脚本或可执行内容注入页面，在受害者浏览器中执行。三种类型：**反射型**（请求参数即时回显）、**存储型**（恶意内容存数据库后展示）、**DOM 型**（前端脚本不安全地操作 DOM）。

**XSS 防护：**按输出上下文做编码——HTML、属性、URL、JavaScript 和 CSS 上下文不能使用同一种转义；富文本必须使用可信的白名单清洗器，避免直接使用 `innerHTML` 拼接不可信内容；配置 CSP 限制脚本来源；Cookie 设置 `HttpOnly`、`Secure` 和合适的 `SameSite`；避免把 Token 放在可被脚本读取的位置。前端框架的默认转义有帮助，但危险 API、模板拼接和第三方组件仍需人工审查。

**概念原理（CSRF）：**攻击者诱导已登录浏览器向目标站点发起跨站请求，浏览器会自动携带 Cookie，使服务端误以为请求来自用户本人。

**CSRF 防护：**CSRF Token、校验 `Origin/Referer`、Cookie 的 `SameSite`、关键操作二次确认、避免用 GET 修改数据。

**关键细节：**CORS 只控制浏览器读取响应，不能单独替代 CSRF 防护；XSS 与 CSRF 维度不同——XSS 是"信任了不可信输入"，CSRF 是"信任了浏览器自动携带的凭证"；XSS 窃取 Token 会使 CSRF 防护失效，所以先防 XSS。

**风险与取舍：**富文本场景必须放行 HTML 时过滤最难（标签黑名单易绕过，用白名单清洗器）；全局 CSP 可能影响第三方脚本；`SameSite=Strict` 可能影响正常跨站跳转场景。

**项目落地：**前端框架默认转义 + 富文本白名单清洗；登录 Cookie 全开 `HttpOnly; Secure; SameSite=Lax`；写操作接口统一 CSRF Token 校验；安全头（CSP、X-Frame-Options）网关统一配置；敏感操作加二次验证。

**面试追问：**存储型和反射型 XSS 的区别（持久化 vs 一次性）；HttpOnly 防什么（XSS 读 Cookie）；CSRF 为什么第三方站点也能发请求（浏览器自动带 Cookie）；JWT 放 Header 为什么不怕 CSRF（凭证不在 Cookie 里，不会自动携带）。

---

## 02 理解补充与边界校订

本题比较两种攻击的信任来源，可保留同篇，但防线不能互相替代。HttpOnly 阻止脚本读 Cookie，不阻止 XSS 以用户身份发请求；SameSite 也不处理同站不同源的所有风险。Authorization Header 的 token 降低传统自动带 Cookie 的 CSRF 风险，却仍需防 XSS、错误的跨域允许配置和 token 泄露。

---

## 03 依据与延伸阅读

- [OWASP XSS 防护](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP CSRF 防护](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

---

## 04 相关问题

- [[八股/10-工程实践/02-Web安全/01-什么是 SQL 注入？如何防止|什么是 SQL 注入？如何防止]]：数据库与浏览器的解释边界

---

## 05 所属专题

- [[八股/10-工程实践/02-Web安全/00-Web安全导航|Web安全导航]]
