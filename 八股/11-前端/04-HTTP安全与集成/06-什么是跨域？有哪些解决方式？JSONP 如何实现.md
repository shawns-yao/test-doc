---
aliases:
- "什么是跨域？有哪些解决方式？JSONP 如何实现？"
- "前端 4.6 什么是跨域？有哪些解决方式？JSONP 如何实现？"
---

# 06 什么是跨域？有哪些解决方式？JSONP 如何实现？

## 01 核心回答


**概念：**跨域 = 浏览器**同源策略**限制（协议 + 域名 + 端口任一不同即跨域），阻止页面读取跨源响应（主要限制跨源读取，不能代替 CSRF 防护）。

**解决方式：**① **CORS（标准方案）**：服务端加 `Access-Control-Allow-Origin` 响应头；简单请求无需预检，但读取响应仍需 CORS 授权，复杂请求（自定义头/非简单方法）先 **OPTIONS 预检**；② **JSONP**：利用 `<script>` 标签不受同源限制，只支持 GET；③ **代理转发**：开发用 Vite/Webpack proxy，生产用 Nginx 反向代理同源化；④ **postMessage**：跨域窗口通信（iframe）；⑤ 同源部署（前后端同域）最省事。

**JSONP 实现：**① 前端动态创建 `<script src="https://api.com/data?callback=cb"></script>`；② 服务端返回 `cb({...数据...})`（JS 函数调用）；③ 前端预定义 `window.cb = (data) => {...}`，脚本加载即执行回调。局限：只能 GET、有 XSS 风险、依赖服务端配合。

**追问：**CORS 预检触发条件（自定义头/非 GET POST/特殊 Content-Type）；Cookie 跨域要什么（Credentials + Allow-Credentials）；为什么 JSONP 不能 POST（script src 只发 GET）。

---

## 02 CORS 不是 CSRF 防线

订正：同源策略主要限制跨源读取，不能阻止所有跨站请求或副作用。符合简单请求条件的请求可在没有预检的情况下发出，服务器缺少允许头时浏览器只是不给脚本读取响应；如果接口已经执行转账或改配置，读不到响应并不能撤销副作用。因此 Cookie 鉴权接口仍需 CSRF Token、来源校验等独立防护。

---

## 03 凭据与代理的边界

携带跨源凭据时，客户端需配置相应凭据模式，服务端需明确允许凭据，并返回匹配的具体源，不能以通配符 * 代替。Cookie 是否实际发送还受 SameSite、Secure、第三方 Cookie 策略和站点关系约束；跨源与跨站不是同一概念。

开发代理只影响开发环境，生产仍需服务端配置或同源反代。postMessage 应验证消息 origin、source 与数据结构，发送时指定准确 targetOrigin。JSONP 会把第三方返回内容作为脚本执行，只适合可信来源的历史兼容场景，不能用于需要安全隔离的数据接口。

---

## 04 参考

- [MDN CORS：简单请求与凭据](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [OWASP CSRF 防护](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

---

## 05 相关问题与延伸

- [[八股/11-前端/04-HTTP安全与集成/07-前端安全需要关注哪些问题|前端安全需要关注哪些问题]]：跨源访问与浏览器安全边界

---

## 06 所属专题

- [[八股/11-前端/04-HTTP安全与集成/00-HTTP安全与集成导航|HTTP安全与集成导航]]
