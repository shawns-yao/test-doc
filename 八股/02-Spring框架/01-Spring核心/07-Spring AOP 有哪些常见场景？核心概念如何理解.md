---
aliases:
- "Spring AOP 有哪些常见场景？核心概念如何理解？"
- "Spring框架 1.7 Spring AOP 有哪些常见场景？核心概念如何理解？"
---

# 07 Spring AOP 有哪些常见场景？核心概念如何理解？

## 01 核心回答

**常见场景：**① **日志与链路追踪**——统一记录接口入参、耗时、traceId，统计方法 RT、成功率、异常率上报监控系统，不污染业务代码；② **事务管理**——`@Transactional` 本质就是 AOP 在方法前后做事务开启/提交/回滚；③ **权限与鉴权**——在 Controller/Service 入口做统一权限校验，失败直接拦截；④ **审计与合规**——对敏感操作统一留痕（谁在什么时候做了什么）。

**实现原理：**Spring 给目标对象「套代理」，调用先进入代理，再执行切面逻辑，最后调用目标方法。三个关键概念：`JoinPoint`（可被拦截的位置，Spring 里主要是方法执行）、`Pointcut`（匹配哪些方法）、`Advice`（增强逻辑：Before/After/Around/AfterThrowing）。

**代理方式：**Spring Framework 的代理选择取决于接口及 proxyTargetClass 配置；Spring Boot 默认使用类代理，不能仅按“有没有接口”断定最终类型。

**执行链路（最常考）：**外部调用 → 代理对象 → 拦截器链（多个 Advice）→ 目标方法 → 返回/异常处理。`@Around` 可以决定是否继续执行目标方法（`proceed()`）。

**口述重点：**Spring AOP 是「代理 + 拦截器链」机制，把日志、事务、鉴权等横切能力从业务代码里抽出来统一治理。

---

## 02 如何设计一个可用的切面

先明确 JoinPoint 是一次可增强的方法执行，Pointcut 负责选择，Advice 负责动作，Aspect 把选择规则与动作组织在一起，Advisor 是 Spring 中切点与通知的一种组合表达。Around 的 proceed 是继续调用链，既可以不调用，也可以多次调用，但后者可能重复执行业务副作用。

记录耗时需覆盖正常返回与异常，入参日志要避免密码、令牌和大对象；鉴权不能只做“登录存在”而漏掉具体资源权限。需要拿提交成功后的状态来做审计时，方法正常返回也不一定代表事务最终成功，应选择正确的事务完成时点。

---

## 03 与机制题的关联

本题关注应用场景和边界；代理如何创建以及自调用为什么绕过，见 [[八股/02-Spring框架/01-Spring核心/02-Spring AOP（面向切面编程）的原理是什么|Spring AOP 代理与拦截链]]。多个切面排序需做针对性测试，尤其重试、事务和缓存组合。

- [Spring AOP 代理语义](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)

---

## 04 所属专题

- [[八股/02-Spring框架/01-Spring核心/00-Spring核心导航|Spring核心导航]]
