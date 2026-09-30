---
aliases:
- "01-SpringBoot"
- "SpringBoot"
---

# 01 Spring Boot 自动装配如何工作

## 01 Spring Boot 的定位

### 01.1 单体与微服务都能使用

它本质上不是“微服务框架”，而是：

> 用来快速构建独立 Spring 应用的工程框架。

Spring Boot 本身并不等于微服务，它的核心目标是简化 Spring 应用的配置和启动。

既可以用于普通单体、多模块项目，也可以作为微服务体系中每个独立服务的基础框架；微服务还需要服务治理、注册发现、配置管理等能力。

---
## 02 自动装配入口

问题：为什么我只加个依赖，Spring Boot 就知道应该创建哪些 Bean？

入口是@SpringBootApplication，核心可以理解成三个东西
- @SpringBootConfiguration
- @EnableAutoConfiguration
- @ComponentScan

其中自动装配的核心是@EnableAutoConfiguration

---

## 03 从候选配置到 Bean 的完整链路

EnableAutoConfiguration 导入自动配置选择逻辑，读取自动配置候选列表，结合 exclusions、排序和条件筛选，将符合条件的配置注册为 BeanDefinition，再进入正常容器生命周期。它不是扫描整个依赖包就无条件创建所有对象。

现代 Spring Boot 使用 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports 声明候选自动配置；Boot 2.7 支持这一方式，Boot 3+ 不再用 spring.factories 中的 EnableAutoConfiguration 键注册自动配置。spring.factories 仍可能用于其他扩展点，不应说该文件彻底不存在。

ConditionalOnClass 检查依赖类是否在类路径，ConditionalOnMissingBean 让用户自定义 Bean 优先，ConditionalOnProperty 根据配置开关选择。starter 主要整理依赖；自动配置依赖这些条件作决定，所以“加 starter”不是“启动所有功能”。

## 04 为什么我的自动配置没有生效

先看依赖与候选是否存在，再看 exclusions、条件不匹配原因、用户已有 Bean、配置值以及配置类的先后关系。启用 debug 可查看条件评估报告；有 Actuator 时可在合适权限控制下查看 conditions 端点。不要只反复添加 ComponentScan，自动配置导入和业务组件扫描是两条不同路径。

## 05 口述版与关联

Spring Boot 通过约定、依赖和条件式自动配置减少重复配置，同时允许应用通过显式 Bean 和属性覆盖默认选择。Bean 生命周期与 AOP 是自动配置产物继续参与的容器机制，分别见 [[八股/02-Spring框架/02-Bean生命周期/01-Spring Bean 从实例化到销毁的完整生命周期流程是什么|Bean 生命周期]] 和 [[八股/02-Spring框架/01-Spring核心/02-Spring AOP（面向切面编程）的原理是什么|AOP 代理]]，不在这里重复保留空题。

- [Boot 自动配置用法与条件报告](https://docs.spring.io/spring-boot/reference/using/auto-configuration.html)
- [Boot 自定义自动配置、imports 与条件注解](https://docs.spring.io/spring-boot/reference/features/developing-auto-configuration.html)

## 06 相关问题与延伸

- [[八股/02-Spring框架/01-Spring核心/09-配置治理的最佳实践|配置治理的最佳实践]]：外部配置与自动装配的条件

## 07 所属专题

- [[八股/02-Spring框架/03-SpringBoot/00-SpringBoot导航|SpringBoot导航]]
