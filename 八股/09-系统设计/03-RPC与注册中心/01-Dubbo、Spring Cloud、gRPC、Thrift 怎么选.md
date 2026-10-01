---
aliases:
- "Dubbo、Spring Cloud、gRPC、Thrift 怎么选？"
- "系统设计 3.1 Dubbo、Spring Cloud、gRPC、Thrift 怎么选？"
---

# 01 Dubbo、Spring Cloud、gRPC、Thrift 怎么选？

## 01 核心回答

优先按「**协议性能、生态成熟度、团队技术栈**」选型：Java 内网高性能调用常见 Dubbo/gRPC；跨语言统一协议常用 gRPC/Thrift；若团队已重度 Spring 生态且治理组件齐全，Spring Cloud 落地成本最低。

**Dubbo vs Spring Cloud：**Dubbo 是服务框架，支持多种协议，包括基于 HTTP/2、兼容 gRPC 的 Triple，不能仅等同于旧 dubbo TCP 协议；Spring Cloud 更偏**整套分布式系统解决方案**（组件全、与 Spring 生态结合深）——需要分开比较服务框架、治理生态、传输协议与序列化，它们可以组合使用。

**面试追问：**如果核心链路从 HTTP 改 gRPC，网关、监控和鉴权要改哪些点？

---

## 02 先把比较对象放在同一层

Spring Cloud 是分布式系统组件生态，不是一个与 gRPC 对等的线协议；gRPC 提供基于契约的跨语言调用与流式通信；Thrift 组合 IDL、代码生成和可选协议/传输；Dubbo 还覆盖服务发现、治理与多种调用协议。Spring 应用完全可以使用 gRPC 或 Dubbo，不必把四者当互斥套餐。

选型先验证团队语言、已有治理能力、浏览器/移动端接入、单向或双向流、IDL 演进、网关支持、TLS 与认证、监控和可调试性。二进制协议不自动胜过 HTTP JSON，消息大小、压缩、连接复用和业务耗时会改变结果。

---

## 03 迁移到 gRPC 要验证哪些链路

代理是否完整支持所需 HTTP/2 特性、流式调用和 trailers；认证如何携带 metadata；deadline 和取消能否跨调用传播；状态码怎样映射成原有业务错误；新旧 proto 是否兼容；负载均衡是否出现长连接偏斜。浏览器直接调用还需适合的接入方案，不能假设和服务间客户端一样。

监控迁移也要改变判定口径：HTTP/2 的 200 不能直接当作 RPC 成功，要读取最终 gRPC status，并按 service/method 统计调用量、错误类别和延迟；把一次逻辑调用与重试产生的多次 attempt 分开，避免把重试吞吐误认为业务吞吐。取消和 deadline exceeded 应保留各自原因，再按业务 SLO 定义是否计入错误。

对于流式 RPC，既观察从建立到结束的整个调用时长，也观察活跃流数、消息/字节吞吐、首次响应和长时间无进展；不能等一条持续很久的流结束后才发现服务停滞。通过客户端/服务端 interceptor 或受支持的自动埋点传递 metadata 中的追踪上下文，确认跨异步执行和代理后的 span 仍能关联。具体指标名称及自动采集能力按语言库版本核对，欠缺的流级指标需补业务埋点，避免将用户 ID 放进指标标签。

口述：“我先看语言、契约、流式需求与治理生态，再评估实际负载。已有 Spring Cloud 不妨碍选择 gRPC；Dubbo 也不能只按旧 TCP 协议评价。最终方案要连网关、安全、观测和升级一起验证。”

---

## 04 依据与关联问题

- [gRPC 指标中的逻辑调用与 attempt](https://grpc.io/docs/guides/opentelemetry-metrics/)
- [gRPC HTTP/2 状态与 trailers](https://grpc.github.io/grpc/core/md_doc__p_r_o_t_o_c_o_l-_h_t_t_p2.html)
- [gRPC metadata 与追踪上下文](https://grpc.io/docs/guides/metadata/)

- [Dubbo Triple 协议目标](https://dubbo.apache.org/en/overview/reference/protocols/triple/)
- [Spring Cloud 项目定位](https://spring.io/projects/spring-cloud/)
- [gRPC 核心概念](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [Thrift 官方文档](https://thrift.apache.org/docs/)
- [[八股/09-系统设计/03-RPC与注册中心/02-RPC 一次调用的完整链路是什么|RPC 完整调用链路]]
- [[八股/09-系统设计/03-RPC与注册中心/05-序列化协议取舍看什么|序列化与契约演进]]

---

## 05 所属专题

- [[八股/09-系统设计/03-RPC与注册中心/00-RPC与注册中心导航|RPC与注册中心导航]]
