---
aliases:
- "React 状态管理怎么做？"
- "前端 3.6 React 状态管理怎么做？"
---

# 06 React 状态管理怎么做？

## 01 核心回答


**分层选型（由简到繁）：**① **组件内状态**：useState/useReducer——局部 UI 状态；② **跨组件（Props 提升）**：状态提升到共同父组件 + props 传递；③ **跨层级共享**：Context（避免层层传 props，但值变化导致所有消费组件重渲染）；④ **全局/复杂状态**：外部状态库——Redux（集中 store + action/reducer，适合复杂业务）、Zustand（轻量 hooks 风格）、Jotai（原子化）；⑤ **服务端状态**：React Query/SWR（缓存、请求去重、失效更新）。

**选择原则：**状态越局部越好——先 useState/提升，真需要全局再上库；服务端数据用 React Query，客户端全局态用 Zustand/Context；Redux 适合团队规范强、状态复杂的大型应用。

**追问：**Context 的性能问题（所有消费者重渲染，拆分 Context、稳定无语义变化的 Provider value；memo 不能屏蔽 Context 更新）；Redux 和 Zustand 区别（样板代码 vs 简洁）；什么是受控组件（value + onChange 双向）。

## 02 Context 更新与 memo 的边界

订正：Context Provider 的 value 通过 Object.is 判断是否变化，消费该 Context 的组件会收到更新。React.memo 不能阻止组件接收新的 Context 值；它主要针对 props 相等的父组件重渲染场景。可以拆分 Context，减少订阅范围，并稳定没有语义变化的 value 对象，不能把 memo 当作普遍修复。

## 03 状态来源与一致性

区分本地交互状态、可由现有状态推导的值、URL 中可分享的状态及服务端数据。能推导的值尽量不再存一份，否则容易失去同步；请求缓存的 key 需包含影响结果的参数与用户范围。异步请求要处理过期响应覆盖新结果、取消和错误状态，不能只讨论把数据放在哪个 store。

受控组件是由 React 状态控制 value 并通过事件更新，不代表底层自动“双向绑定”。SSR 场景还要防止跨请求共享用户状态。

## 04 参考

- [React useContext：Object.is 与 memo 限制](https://react.dev/reference/react/useContext)

## 05 相关问题与延伸

- [[八股/11-前端/04-HTTP安全与集成/09-主题切换是怎么实现的？组件库底层如何实现主题切换|主题切换是怎么实现的？组件库底层如何实现主题切换]]：共享主题状态与组件更新
- [[八股/11-前端/03-JavaScript与状态管理/07-自下而上（Jotai）和自上而下（Zustand）状态管理模式各有什么优劣|自下而上（Jotai）和自上而下（Zustand）状态管理模式各有什么优劣]]：状态管理机制与库选型

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
