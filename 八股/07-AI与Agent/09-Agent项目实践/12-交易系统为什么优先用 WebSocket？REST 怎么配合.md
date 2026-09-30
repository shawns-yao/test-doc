---
aliases:
- "交易系统为什么优先用 WebSocket？REST 怎么配合？"
- "Agent 4.8 交易系统为什么优先用 WebSocket？REST 怎么配合？"
---

# 12 交易系统为什么优先用 WebSocket？REST 怎么配合？

## 01 核心回答

**WebSocket：**行情、订单状态、仓位变化适合**低延迟实时推送**；**REST：**查询历史、配置类接口、回补丢包。

**系统设计：**常是 **WebSocket 收实时流 + REST 做补数/对账**。

---

## 02 为什么实时推送与请求响应搭配

高频行情、订单变化适合订阅式推送，避免频繁轮询；REST 适合快照、历史查询和明确的一次操作。但“交易系统优先用 WebSocket”不是所有路径的通用规则，是否支持下单、限流和可靠性语义取决于交易所接口。

WebSocket 连接内传输有顺序性，不等于跨重连不丢事件。客户端要处理心跳、订阅确认、断线重连、认证过期、背压与消费延迟；监测最后事件时间和序列号，不能只看 socket 仍连接就认为行情新鲜。

## 03 配合方式

启动时取得快照并与缓冲增量衔接；运行中处理序列缺口和必要的 REST 回查；订单事件缺失时按订单 ID 查询权威状态。回补要遵守接口限流，不能故障时同时全量拉取压垮服务。请求超时与流断开都不等于订单失败。

追问“REST 比 WebSocket 一定慢吗？”不一定，比较连接复用、消息模式、服务实现和实际 p95。口述：“按消息模式选协议，用序列、回查和对账补足业务可靠性，而不是把长连接当可靠消息队列。”

相关：[[八股/07-AI与Agent/09-Agent项目实践/13-Order book 的 snapshot + 增量更新怎么保证正确|快照与增量衔接]]。

## 04 关联追问

- [[八股/07-AI与Agent/07-框架协议与工程化/08-SSE 原理与作用|SSE 原理与作用？]]

## 05 参考资料

- [Binance 官方 WebSocket 与订单簿同步文档](https://github.com/binance/binance-spot-api-docs/blob/master/web-socket-streams.md)
- [Binance 官方现货交易接口契约](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/trading-endpoints)

## 06 所属专题

- [[八股/07-AI与Agent/09-Agent项目实践/00-Agent项目实践导航|Agent项目实践导航]]
