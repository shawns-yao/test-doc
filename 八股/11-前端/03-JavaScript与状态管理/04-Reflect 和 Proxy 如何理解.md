---
aliases:
- "`Reflect` 和 `Proxy` 如何理解？"
- "前端 3.4 `Reflect` 和 `Proxy` 如何理解？"
---

# 04 `Reflect` 和 `Proxy` 如何理解？

## 01 核心回答


**概念：****Proxy** 给对象加一层拦截（get/set/has/delete 等 13 种 trap），可拦截规范规定的对象内部操作，受目标能力与不变量限制；**Reflect** 提供与 Proxy trap 一一对应的**默认行为方法**（`Reflect.get` 等），保证拦截后能调用原始逻辑且返回值一致。

**实现流程（典型用法）：**`new Proxy(target, { get(obj, key, receiver) { ...; return Reflect.get(obj, key, receiver); } })`——trap 里先做自定义逻辑，再用 Reflect 走默认行为；Reflect 还统一了返回值（如 `Reflect.set` 返回布尔）和 receiver 绑定。

**应用场景：**① Vue 3 响应式（Proxy 代理 data）；② 数据校验/脱敏（set trap 拦截非法值）；③ 埋点统计（get trap 记录访问）；④ 私有属性模拟（has/get trap 隐藏）；⑤ 撤销代理（Proxy.revocable）。

**追问：**Proxy 和 Object.defineProperty 的区别（全量代理 vs 属性级、Vue2 vs Vue3）；Reflect 为什么存在（统一默认行为 + 函数式调用）；代理能代理函数吗（能，apply trap）。

## 02 Proxy 不能打破对象不变量

订正：Proxy 只能拦截规范规定的内部操作，不能代理“任何操作”。例如类私有字段访问依赖对象身份与私有品牌检查，不是普通属性 get；只有可调用目标才能使用 apply，构造能力也受目标限制。对于不可配置、不可写等属性，trap 不能任意谎报，否则会抛 TypeError。

Reflect.get 的 receiver 决定 getter 中的 this，直接读取 target[key] 与保留 receiver 的语义可能不同。Reflect 并非保证所有操作“返回值一致”：例如 Reflect.set 返回成功布尔值，但底层 getter/setter 仍可能抛异常。

## 03 响应式与安全边界

代理只拦截经由代理对象发生的访问，持有原始对象的人仍可绕过；浅层代理也不会自动代理所有嵌套对象。因此用 get/has 隐藏字段适合封装约定，不能据此声称获得不可绕过的安全隔离。

## 04 参考

- [ECMAScript：Proxy 内部方法及不变量](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-proxy-object-internal-methods-and-internal-slots)

## 05 相关问题与延伸

- [[八股/11-前端/03-JavaScript与状态管理/02-ES6 的含义是什么？ECMAScript 语言规范如何迭代|ES6 的含义是什么？ECMAScript 语言规范如何迭代]]：语言版本与元对象操作能力

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
