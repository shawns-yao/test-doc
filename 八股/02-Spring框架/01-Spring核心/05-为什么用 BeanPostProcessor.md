---
aliases:
- "为什么用 BeanPostProcessor？"
- "Spring框架 1.5 为什么用 BeanPostProcessor？"
---

# 05 为什么用 BeanPostProcessor？

## 01 核心回答

**概念原理：**BeanPostProcessor（BPP）是 Spring 在 Bean 初始化前后提供的扩展点。容器创建并完成属性填充后，会在初始化回调前后调用 `postProcessBeforeInitialization` 和 `postProcessAfterInitialization`，允许对 Bean 做统一检查、包装或替换。

**常见用途：**前置阶段可做默认值填充、校验和标记；后置阶段可返回代理对象，实现 AOP、事务、异步、缓存和监控等能力。`AutowiredAnnotationBeanPostProcessor` 负责处理注入注解，自动代理创建器则在后置阶段判断是否需要生成代理。

**执行位置：**BPP 只覆盖容器管理的 Bean，且发生在初始化回调附近；它不是 Bean 生命周期全部步骤的替代品。实现类本身会被容器优先创建，若依赖其他 Bean，要注意实例化顺序和提前暴露代理的问题。

**使用风险：**后置处理器可以返回不同对象，可能导致类型判断、循环依赖和调试困难；处理器逻辑应尽量轻量、幂等，避免在其中执行远程调用。需要修改 Bean 定义时应区分 `BeanFactoryPostProcessor`，它处理的是 BeanDefinition 而不是实例。

**面试追问：**BPP 与 BeanFactoryPostProcessor 的区别（实例后置处理 vs BeanDefinition 后置处理）；AOP 代理在哪儿生成（通常在 BPP 后置阶段）；为什么 `new` 出来的对象不生效（没有经过容器生命周期）。

---

## 02 不同扩展点处理的东西不同

BeanFactoryPostProcessor 面向定义，可修改 BeanDefinition 的属性；BeanPostProcessor 面向实例，可检查或返回包装对象。AutowiredAnnotationBeanPostProcessor 还实现 InstantiationAwareBeanPostProcessor 扩展，在属性填充阶段处理注入，不能把所有 BPP 工作都挤进 beforeInitialization。

@PostConstruct 通常由一个后置处理器在 beforeInitialization 链中触发，再执行 InitializingBean 和自定义 init。AOP 通常在 afterInitialization 创建最终代理，但循环依赖时也可能经 SmartInstantiationAwareBeanPostProcessor 提供早期引用。

## 03 实际风险与参考

BPP 及其直接依赖会很早初始化，可能不适合被其他自动代理完整处理；在 BPP 中主动 getBean 还可能提前创建目标。顺序应使用受支持的 PriorityOrdered/Ordered 等约定并确认注册方式，不应依赖类路径碰巧扫描的顺序。

- [Spring 容器扩展点](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html)
- [[八股/02-Spring框架/02-Bean生命周期/01-Spring Bean 从实例化到销毁的完整生命周期流程是什么|初始化回调的实际位置]]

## 04 所属专题

- [[八股/02-Spring框架/01-Spring核心/00-Spring核心导航|Spring核心导航]]
