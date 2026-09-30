---
aliases:
- "Spring IoC（控制反转）的原理是什么？"
- "Spring框架 1.1 Spring IoC（控制反转）的原理是什么？"
---

# 01 Spring IoC（控制反转）的原理是什么？

## 01 核心回答


**核心含义：**IoC（控制反转）把对象的创建、依赖组装和生命周期管理从业务代码交给 Spring 容器；业务类通常声明“需要什么”，不再负责 `new` 具体实现。依赖方向由“业务类主动找依赖”变为“容器注入依赖”。

**容器流程：**启动时读取注解、配置类或 XML，解析为 `BeanDefinition`；根据定义实例化 Bean，解析构造器/属性依赖，执行后置处理器和初始化回调，最后按作用域缓存并提供给调用方。`ApplicationContext` 是常用的完整容器，底层能力来自 `BeanFactory`。

**注入方式：**构造器注入最适合必需依赖和不可变对象，也便于单元测试；Setter 注入适合可选依赖；字段注入写法简单但隐藏依赖、测试不便，生产代码通常优先构造器注入。多个候选 Bean 要用 `@Qualifier` 或 `@Primary` 消歧。

**价值与边界：**IoC 降低模块耦合，便于替换实现、统一配置和测试，但容器启动、代理和反射会增加理解成本；手动 `new` 出来的对象不受容器管理，也不会自动获得注入、事务或 AOP 能力。

**面试追问：**循环依赖如何处理（允许循环引用且满足单例及早期引用条件时，部分 Setter/字段循环可借助三级缓存，构造器循环依赖通常无法解决）；Bean 默认作用域是什么（singleton）；为什么推荐构造器注入（依赖显式、对象可不变、失败更早）。

---

## 02 控制反转不只是反射创建对象

BeanDefinition 是对象创建与装配的元数据，Bean 是按定义产生的实例；容器先处理定义，再实例化和注入，最后执行初始化与扩展点。反射只是可能采用的手段，工厂方法、显式注册实例和 AOT 生成方式也可以参与创建。

singleton 表示每个容器中每个 Bean 定义通常一个实例，不是全 JVM 永远只有一个对象，也不自动保证线程安全。无状态服务可共享；可变请求数据应避免放在单例字段中。单例直接注入 prototype 时通常只在自身创建时解析一次，需要按次获取则用 ObjectProvider 或作用域代理。

## 03 口述与参考

IoC 解决对象图由谁组装，DI 是常见实现方式；它把依赖显式化并统一管理生命周期，但仍需设计清晰的职责与作用域。

- [Spring 依赖注入与构造器/Setter取舍](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [[八股/02-Spring框架/02-Bean生命周期/01-Spring Bean 从实例化到销毁的完整生命周期流程是什么|对象创建之后的生命周期]]

## 04 相关问题与延伸

- [[八股/02-Spring框架/01-Spring核心/06-ApplicationContext 和 BeanFactory 的区别|ApplicationContext 和 BeanFactory 的区别]]：容器接口能力与IoC依赖管理

## 05 所属专题

- [[八股/02-Spring框架/01-Spring核心/00-Spring核心导航|Spring核心导航]]
