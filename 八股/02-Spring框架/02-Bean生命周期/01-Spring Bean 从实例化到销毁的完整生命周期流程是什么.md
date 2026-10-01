---
aliases:
- "Spring Bean 从实例化到销毁的完整生命周期流程是什么？"
- "Spring框架 2.1 Spring Bean 从实例化到销毁的完整生命周期流程是什么？"
---

# 01 Spring Bean 从实例化到销毁的完整生命周期流程是什么？

## 01 核心回答


以单例 Bean 为例，流程大致是：

1. Spring 读取 BeanDefinition，实例化 Bean。

2. 通过依赖注入填充属性。

3. 执行 `Aware` 接口回调，让 Bean 获取 BeanName、BeanFactory 等容器信息。

4. 执行 `BeanPostProcessor` 的前置处理。

5. 初始化回调通常按 `@PostConstruct`、`InitializingBean.afterPropertiesSet()`、自定义 `init-method` 的顺序；其中 PostConstruct 实际由前置 BPP 链内的处理器触发。

6. 执行 `BeanPostProcessor` 的后置处理，AOP 代理通常在这一阶段生成。

7. Bean 放入单例池并对外提供。

8. 容器关闭时执行 `@PreDestroy`、`DisposableBean.destroy()` 和自定义 `destroy-method`。

实际顺序要结合具体处理器和配置确认；构造器注入发生在属性填充前，原型 Bean 默认由容器创建但不负责完整销毁。

---

## 02 实例化、属性填充和初始化为什么分开

构造器运行只说明对象实例已存在，依赖字段和初始化逻辑可能还没完成。BeanNameAware/BeanFactoryAware 等回调帮助对象了解容器；ApplicationContextAware 等还由对应处理器完成。初始化后容器暴露的对象可能是包装代理，不一定与最初 new 的引用相同。

若同时配置多种不同初始化回调，通常先 PostConstruct，再 afterPropertiesSet，再自定义 init；同一方法被多个机制指向时容器会避免不必要的重复调用。销毁对应 PreDestroy、DisposableBean 和自定义 destroy。Spring 6+ 的 PostConstruct/PreDestroy 注解来自 jakarta.annotation，旧 javax 包需注意迁移。

---

## 03 生命周期边界

prototype 创建后交给调用方，容器不自动跟踪完整销毁；正常关闭上下文才能可靠触发已注册的销毁回调，kill -9 等强制结束不能依赖这些回调。初始化回调里也不要假设最终代理已完全可用，尤其依赖事务增强的初始化操作应单独设计。

---

## 04 参考与关联

- [Spring Bean 初始化/销毁回调](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html)
- [[八股/02-Spring框架/01-Spring核心/05-为什么用 BeanPostProcessor|BPP 各扩展阶段的位置]]
- [[八股/02-Spring框架/01-Spring核心/03-Spring 循环依赖如何解决（三级缓存）？为什么三级不是两级|为什么早期引用还是半成品]]

---

## 05 相关问题与延伸

- [[八股/02-Spring框架/03-SpringBoot/01-Spring Boot 自动装配如何工作|Spring Boot 自动装配如何工作]]：反向关联：此题引用了本题的机制或边界
- [[八股/02-Spring框架/01-Spring核心/01-Spring IoC（控制反转）的原理是什么|Spring IoC（控制反转）的原理是什么]]：反向关联：此题引用了本题的机制或边界

---

## 06 所属专题

- [[八股/02-Spring框架/02-Bean生命周期/00-Bean生命周期导航|Bean生命周期导航]]
