

# 1.Spring Boot 是用来做微服务还是普通 Java 分模块？


它本质上不是“微服务框架”，而是：

> 用来快速构建独立 Spring 应用的工程框架。


Spring Boot 本身并不等于微服务，它的核心目标是简化 Spring 应用的配置和启动。

既可以用于普通单体、多模块项目，也可以作为微服务体系中每个独立服务的基础框架；微服务还需要服务治理、注册发现、配置管理等能力。


---
## 2.Spring Boot 自动装配机制？

问题：为什么我只加个依赖，Spring Boot 就知道应该创建哪些 Bean？

入口是@SpringBootApplication，核心可以理解成三个东西
- @SpringBootConfiguration
- @EnableAutoConfiguration
- @ComponentScan

其中自动装配的核心是@EnableAutoConfiguration




---
# 3.Bean 生命周期？







---
# 4.AOP 代理什么时候产生？

