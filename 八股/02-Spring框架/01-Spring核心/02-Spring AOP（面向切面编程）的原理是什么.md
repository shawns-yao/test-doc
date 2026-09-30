---
aliases:
- "Spring AOP（面向切面编程）的原理是什么？"
- "Spring框架 1.2 Spring AOP（面向切面编程）的原理是什么？"
---

# 02 Spring AOP（面向切面编程）的原理是什么？

## 01 核心回答


**核心思想：**AOP 把日志、事务、权限、监控等横切逻辑从业务代码中抽离，在不修改核心业务的情况下统一织入。Spring AOP 主要拦截 Spring Bean 的方法执行，适合处理重复、与主业务相对独立的逻辑。

**关键概念：**`JoinPoint` 是可拦截位置；`Pointcut` 决定匹配哪些方法；`Advice` 是增强逻辑，包括 Before、After、AfterReturning、AfterThrowing 和 Around。多个切面会形成拦截器链，按优先级执行。

**代理方式：**Spring Framework 未强制类代理时，有合适接口可使用 JDK 动态代理；无接口或配置类代理时使用 CGLIB。Spring Boot 默认类代理的配置不能与 Framework 默认混为一谈。调用链是“外部调用 → 代理对象 → 拦截器链 → 目标方法 → 返回/异常处理”，`@Around` 通过 `proceed()` 决定是否继续执行。

**典型应用：**`@Transactional` 由事务拦截器完成开启、提交和回滚；日志切面记录 traceId、耗时和异常；权限切面在入口校验身份与资源；审计切面记录敏感操作。切面应保持幂等、低延迟，并避免吞掉业务异常。

**常见失效点：**同类内部调用没有经过代理，注解可能不生效；对象由 `new` 创建、不在 Spring 容器中也不会被织入；`private`、`final` 方法和自调用场景要结合代理类型判断。需要时拆分 Bean 或改用编程式方案。

---

## 02 为什么 this 调用绕过增强

调用方持有的是代理，代理执行拦截链后再进入目标对象。目标对象内部的 this 指向目标自身，调用另一个方法时没有回到代理入口，所以不会新增该方法的 advice。即使是类代理，也不要假设普通 Spring AOP 会自动拦截 self-invocation。

JDK 代理暴露接口方法；CGLIB 通过子类覆盖方法，因此 final 类不能这样代理，final/private 方法不能被覆盖增强。Spring AOP 只支持方法执行连接点，不自动支持构造器调用或字段访问；AspectJ 编译期/加载期织入是另一种机制，其自调用边界不同。

## 03 调用链的取舍

多个 around advice 是嵌套调用，外层进入早、退出晚，顺序会影响事务、重试和缓存。例如在同一事务内重试与每次重试新建事务的结果不同。应给必要的切面明确顺序并测试异常传播；不要吞异常后假装事务层能自动识别失败。

- [Spring 代理机制与自调用](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
- [[八股/02-Spring框架/01-Spring核心/04-事务失效有哪些场景？同类内部调用为什么失效|代理边界如何影响事务]]

- [Spring Boot 默认类代理与配置开关](https://docs.spring.io/spring-boot/reference/features/aop.html)
- [[八股/02-Spring框架/01-Spring核心/07-Spring AOP 有哪些常见场景？核心概念如何理解|如何选择切点、通知和实际横切场景]]

## 04 相关问题与延伸

- [[八股/02-Spring框架/03-SpringBoot/01-Spring Boot 自动装配如何工作|Spring Boot 自动装配如何工作]]：反向关联：此题引用了本题的机制或边界

## 05 所属专题

- [[八股/02-Spring框架/01-Spring核心/00-Spring核心导航|Spring核心导航]]
