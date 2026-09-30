---
aliases:
- "ApplicationContext 和 BeanFactory 的区别？"
- "Spring框架 1.6 ApplicationContext 和 BeanFactory 的区别？"
---

# 06 ApplicationContext 和 BeanFactory 的区别？

## 01 核心回答

**定位：**BeanFactory 是最底层的 IOC 容器接口，核心是获取与查询 Bean；注册能力由 BeanDefinitionRegistry 等相关接口提供；ApplicationContext 是在 BeanFactory 之上的**完整企业容器**，功能更全。

BeanFactory

依赖注入、Bean 生命周期管理、基础作用域支持。

ApplicationContext

额外提供：国际化（MessageSource）、事件发布/监听（ApplicationEventPublisher）、资源加载（ResourceLoader）、环境与配置体系（Environment）、自动注册 BeanPostProcessor（更好集成 AOP、事务）。

**加载时机：**BeanFactory 偏**延迟加载**（getBean 时才创建）；ApplicationContext refresh 默认预实例化**非懒加载单例** Bean——启动更「重」，但错误能更早暴露。

**生产选型：**现代 Spring 应用要用到事件、配置、AOP、事务、自动装配等能力，ApplicationContext 开箱即用，开发和治理成本更低。

**口述重点：**BeanFactory 是「能用的最小容器」，ApplicationContext 是「企业级完整容器」；实际项目默认选 ApplicationContext。

---

## 02 延迟加载不是两者不可改变的本质

BeanFactory 实现也能显式预实例化单例，ApplicationContext 中的 lazy Bean 也可延迟创建；真正差异是上下文层自动整合后置处理器、事件、消息、环境与资源加载等应用基础设施。

只手工创建 DefaultListableBeanFactory 并注册 Bean 定义，不代表 @Autowired、事务和各种扩展点都会像完整应用一样自动就绪，必须注册对应处理器。ApplicationContext refresh 负责把这些阶段组织起来。

## 03 面试口述与参考

BeanFactory 提供核心对象工厂契约，ApplicationContext 在其上提供应用级容器体验。是否懒加载是默认行为与配置差异；不要答成“BeanFactory 无法 AOP、ApplicationContext 才能创建对象”。

- [BeanFactory 与 ApplicationContext 的能力对照](https://docs.spring.io/spring-framework/reference/core/beans/beanfactory.html)

## 04 相关问题与延伸

- [[八股/02-Spring框架/01-Spring核心/01-Spring IoC（控制反转）的原理是什么|Spring IoC（控制反转）的原理是什么]]：容器接口能力与IoC依赖管理

## 05 所属专题

- [[八股/02-Spring框架/01-Spring核心/00-Spring核心导航|Spring核心导航]]
