---
aliases:
- "ES6 的含义是什么？ECMAScript 语言规范如何迭代？"
- "前端 3.2 ES6 的含义是什么？ECMAScript 语言规范如何迭代？"
---

# 02 ES6 的含义是什么？ECMAScript 语言规范如何迭代？

## 01 核心回答


**概念：**ECMAScript 是 JS 的**语言规范**（语法/类型/标准库），由 TC39 委员会维护；ES6 = ECMAScript 2015，是**里程碑版本**（此后每年一版，命名改为年份：ES2016/ES2017…）。

**迭代流程（详见下方现行阶段订正）：**Stage 0 想法 → Stage 1 探索 → Stage 2 草案 → Stage 2.7 验证 → Stage 3 实现反馈 → Stage 4 完成。

**ES6 核心新增：**let/const（块级作用域）、箭头函数、类、模板字符串、解构、默认参数、Promise、模块化（import/export）、Symbol/Map/Set；后续 ES2017 另增 async/await。

**追问：**let 和 var 的区别（块级作用域 + 暂时性死区）；箭头函数和普通函数的区别（this 绑定、无 arguments）；迭代怎么跟上（TC39 官网 + 浏览器兼容表）。

## 02 TC39 阶段名称订正

核验于 2026-09-30：当前流程包括 Stage 0、1、2、2.7、3、4。0 是初始想法；1 进入问题与方案探索；2 形成草案方案；2.7 设计原则上完成，进入测试验证；3 推荐实现并收集实践反馈；4 完成并进入规范整合。原文“四阶段”及把 Stage 2 写作候选、Stage 3 写作冻结的叫法不准确。

ES6 专指 2015 版，不等于此后所有现代 JavaScript。async/await 属于 ES2017，应与 ES2015 新增特性分开。标准发布、引擎实现和项目工具链支持是三个时间线；一个提案被编译器支持也不意味着已标准化。

## 03 应用时怎样查证

先查 TC39 正式规范和提案状态，再检查目标 Node/浏览器版本。语法可转译不代表运行时 API 都能低成本 polyfill；项目应明确编译目标与兼容测试范围。

## 04 参考

- [TC39 当前流程与 Stage 2.7](https://tc39.es/process-document/)
- [TC39 已完成提案及纳入年份](https://github.com/tc39/proposals/blob/main/finished-proposals.md)

## 05 相关问题与延伸

- [[八股/11-前端/03-JavaScript与状态管理/04-Reflect 和 Proxy 如何理解|Reflect 和 Proxy 如何理解]]：语言版本与元对象操作能力
- [[八股/11-前端/03-JavaScript与状态管理/03-了解哪些较新的 ECMAScript 特性|了解哪些较新的 ECMAScript 特性]]：规范演进与具体特性状态

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
