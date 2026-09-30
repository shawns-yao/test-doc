---
aliases:
- "Spring 循环依赖如何解决（三级缓存）？为什么三级不是两级？"
- "Spring框架 1.3 Spring 循环依赖如何解决（三级缓存）？为什么三级不是两级？"
---

# 03 Spring 循环依赖如何解决（三级缓存）？为什么三级不是两级？

## 01 核心回答


**概念原理：**循环依赖 = A 依赖 B、B 又依赖 A。Spring 用**三级缓存**在允许循环引用时尝试解决**部分单例属性注入**的循环依赖：① 一级：成品单例池（singletonObjects）；② 二级：早期单例（earlySingletonObjects，已实例化未完全初始化）；③ 三级：单例工厂（singletonFactories，存 ObjectFactory 用于生成早期引用）。

**解决流程（A → B → A）：**① 创建 A：实例化（new）→ 放入**三级缓存**（工厂）→ 填充属性发现需要 B；② 创建 B：实例化 → 三级缓存 → 填充属性发现需要 A；③ B 从**三级缓存**取 A 的工厂生成**早期引用**（提前暴露，此时 A 未完成属性填充）→ 放入二级缓存 → B 完成初始化；④ A 拿到 B，完成自己的属性填充和初始化 → 加入一级缓存，并清理相关二级、三级缓存。

**为什么需要三级而不是两级：**二级缓存也能解决循环依赖，但**三级缓存为了支持 AOP 代理**——若 A 需要代理，早期引用必须用代理对象，而不是原始对象。三级缓存放的是 ObjectFactory，在「有人真正需要早期引用」时才调用工厂生成（可在此织入代理）；在这种实现设计下，工厂延迟决定并生成早期引用，二级缓存复用同一早期结果，避免重复代理；如果改为提前生成引用，理论上可采用不同缓存结构。不能断言“两级在数学上必然无法支持 AOP”。

**风险与取舍：**构造器注入的循环依赖**无法解决**（实例化就需要对方，提前暴露来不及）——报 BeanCurrentlyInCreationException；prototype 作用域不缓存也无法解决；循环依赖本质是设计坏味道，能避免就避免（拆依赖/延迟注入 @Lazy）。

**面试追问：**为什么构造器注入循环依赖解决不了（实例化阶段就卡住）；@Lazy 怎么破循环依赖（代理占位，真正调用时才初始化）；循环依赖和 AOP 的关系（三级缓存 + 提前代理）。

---

## 02 为什么有早期对象仍不等于可安全使用

B 注入 A 的早期引用时，A 可能还没注入自身依赖、没执行初始化回调。B 若在构造/初始化中立即调用 A 的业务方法，仍可能遇到未完成状态。三级缓存解决的是依赖解析与对象身份协调，不保证半成品对象适合执行业务。

构造器 A 必须先拿到 B、B 又必须先拿到 A 时，还没有可提前暴露的 A 实例，因此不能靠普通三级缓存完成。@Lazy/ObjectProvider 可以延后依赖解析，但如果在初始化过程中立即解引用，仍可能重新触发循环。

## 03 版本与取舍

Spring Boot 2.6 起默认禁止循环引用，当前 spring.main.allow-circular-references 默认仍为 false。把开关设为 true 也只恢复“尝试解决”，不会解决所有构造器、prototype 或不兼容代理循环。更稳妥的是抽出第三个职责、改事件协作或重新划分依赖方向。

- [Spring 依赖注入：循环依赖边界](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Boot allow-circular-references 默认值](https://docs.spring.io/spring-boot/appendix/application-properties/index.html#application-properties.core.spring.main.allow-circular-references)

- [Spring Boot 2.6 起默认禁止循环引用](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes#circular-references-prohibited-by-default)

## 04 相关问题与延伸

- [[八股/02-Spring框架/02-Bean生命周期/01-Spring Bean 从实例化到销毁的完整生命周期流程是什么|Spring Bean 从实例化到销毁的完整生命周期流程是什么]]：反向关联：此题引用了本题的机制或边界

## 05 所属专题

- [[八股/02-Spring框架/01-Spring核心/00-Spring核心导航|Spring核心导航]]
