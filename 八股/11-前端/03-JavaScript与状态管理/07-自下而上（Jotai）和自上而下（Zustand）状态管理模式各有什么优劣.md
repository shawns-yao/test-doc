---
aliases:
- "自下而上（Jotai）和自上而下（Zustand）状态管理模式各有什么优劣？"
- "前端 3.7 自下而上（Jotai）和自上而下（Zustand）状态管理模式各有什么优劣？"
---

# 07 自下而上（Jotai）和自上而下（Zustand）状态管理模式各有什么优劣？

## 01 核心回答


**Jotai（自下而上，原子化）**

状态拆成**最小原子**（atom），组件按需组合派生（atom 依赖 atom），粒度细、只重渲染用到该原子的组件。

**优点：**天然按需订阅、可细粒度订阅、支持独立 store 隔离、易组合测试。

**缺点：**状态分散在原子网络，跨模块梳理依赖关系需要心智成本；原子划分不当会碎片化。

**Zustand（自上而下，store 中心）**

一个/几个**集中 store**（hooks 风格 `create`），组件选择订阅自己需要的切片（selector），默认按 selector 结果相等性判断，浅比较需显式选择。

**优点：**结构清晰、样板少、可在组件外读写、中间件（persist/devtools）丰富。

**缺点：**selector 不当会订阅过宽导致重渲染；大 store 组织不当会变"全局变量袋"。

**怎么选：**状态关联松散、粒度细 → Jotai；状态聚合明确、要全局访问 → Zustand；两者都比 Redux 轻量，面试能讲清差异即可。

**追问：**为什么 Zustand 比 Context 性能好（selector 订阅 vs 全量重渲染）；原子派生是什么（derived atom 自动计算）；持久化怎么做（persist 中间件）。

---

## 02 Store 与比较策略订正

Jotai 不是“没有 store”：atom 通常是状态定义，具体值存在 store 中，既有默认 store，也可创建独立 store 并用 Provider 隔离。底层订阅粒度细并不自动保证全部应用更快，派生计算和 atom 划分仍需实测。

Zustand 默认不会自动对所有 selector 结果做浅比较；当前官方指南按 Object.is 描述结果变化，返回新对象或数组时需使用稳定引用、useShallow 或相应比较 API。Zustand v5 对不稳定 selector 输出有额外注意事项，不能把 v4 的自定义 equalityFn 用法直接照搬。

---

## 03 如何做取舍

比较跨域业务更新是否集中、派生依赖是否复杂、是否需要请求隔离、调试和持久化迁移。两种模式都能组织大应用，也都可能过度全局化。持久化缓存只应存可恢复且适合落盘的数据，并考虑版本迁移和登录用户切换，不能把持久化自动等同安全会话管理。

---

## 04 参考

- [Jotai store 官方文档](https://jotai.org/docs/core/store)
- [Zustand selector 与 useShallow 官方指南](https://github.com/pmndrs/zustand/blob/main/docs/learn/guides/prevent-rerenders-with-use-shallow.md)
- [Zustand v5 迁移说明](https://github.com/pmndrs/zustand/blob/main/docs/reference/migrations/migrating-to-v5.md)

---

## 05 相关问题与延伸

- [[八股/11-前端/03-JavaScript与状态管理/06-React 状态管理怎么做|React 状态管理怎么做]]：状态管理机制与库选型

---

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
