---
aliases:
- "Istio 的核心能力有哪些？"
- "系统设计 5.3 Istio 的核心能力有哪些？"
---

# 03 Istio 的核心能力有哪些？

## 01 核心回答

**流量治理：**通过 VirtualService、DestinationRule 等资源实现路由匹配、权重灰度、版本子集、超时、重试、熔断和故障注入；服务发现和负载均衡由数据面代理执行。

**安全与可观测：**支持工作负载身份、mTLS 加密、授权策略、访问日志、指标和分布式追踪。控制面负责配置校验、证书和策略分发，数据面负责每次请求的低延迟执行。

**治理重点：**配置要分环境、分租户、分命名空间管理，发布前做语义校验和影响面检查；错误路由、过宽授权或证书异常可能影响大量服务，因此必须支持版本审计、灰度下发和快速回退。

**面试追问：**为什么 Istio 配置错误会导致全局性故障？

---

## 02 配置如何变成代理行为

以 sidecar 模式为例，路由规则决定请求发往哪个目标/子集，DestinationRule 定义连接池、负载均衡、TLS 与异常实例剔除等策略，控制面把期望配置编译下发给代理。声明资源成功并不保证目标标签匹配或所有代理已接收，应检查代理实际配置与版本。

Istio 的 circuit breaking 常包含连接池资源限制和 outlier detection，不能直接等同于所有应用库的“错误率触发 Closed/Open/Half-Open”算法。重试次数、总超时和并发上限分别保护不同维度，需要和 SDK 策略核对避免重复治理。

## 03 安全 遥测 与版本限制

mTLS 证明通信工作负载身份并加密通道，不自动证明最终用户能读取某张订单；AuthorizationPolicy 与应用对象级权限各有职责。开启 mTLS 也需确认策略模式、覆盖范围和实际连接，不能只看安装了 Istio。

代理可生成网络遥测，但跨应用调用仍需要传播 trace 上下文。Ambient 模式的 L7 能力与路由 API 应依据所用版本/waypoint 配置确认，不应把所有 Sidecar 资源示例无差别复制过去。

口述：“Istio 统一路由、安全和遥测，但我会把声明配置、实际下发和请求行为三层验证。网络身份不替代业务授权，代理重试不替代业务幂等，模式和版本影响具体 API。”

## 04 依据与关联问题

- [Istio 安全模型](https://istio.io/latest/docs/concepts/security/)
- [Istio DestinationRule 参数](https://istio.io/latest/docs/reference/config/networking/destination-rule/)
- [Istio Ambient 数据面](https://istio.io/latest/docs/ambient/architecture/data-plane/)
- [[八股/09-系统设计/05-ServiceMesh/05-引入 Mesh 后排障方式有什么变化|Mesh 排障]]
- [[八股/09-系统设计/04-网关与流量治理/04-网关鉴权与签名校验怎么设计|用户认证与请求签名]]

## 05 所属专题

- [[八股/09-系统设计/05-ServiceMesh/00-ServiceMesh导航|ServiceMesh导航]]
