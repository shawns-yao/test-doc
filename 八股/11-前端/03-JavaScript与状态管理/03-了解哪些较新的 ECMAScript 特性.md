---
aliases:
- "了解哪些较新的 ECMAScript 特性？"
- "前端 3.3 了解哪些较新的 ECMAScript 特性？"
---

# 03 了解哪些较新的 ECMAScript 特性？

## 01 核心回答


**Optional Chaining**
`a?.b?.c` 安全访问链，ES2020。

**Nullish 合并**
`a ?? b` 只兜底 null/undefined（不兜 0/''），ES2020。

**BigInt**
任意精度整数，ES2020。

**Promise.allSettled**
等所有 Promise 结束（不因失败短路），ES2020。

**Array.at / findLast**
at 负数索引访问属 ES2022；findLast 从后查找属 ES2023。

**Record/Tuple（提案）**
不可变值结构提案，已于 2025 年撤回归档。

**结构化克隆**
`structuredClone()` 结构化克隆，属于 HTML 标准的宿主 API。

**Decorators（提案）**
装饰器提案，阶段以 TC39 当前清单为准，见下方核验。

**回答策略：**挑 3-4 个讲用法 + 场景即可，不用背全——重点展示「知道规范在演进、会查兼容性」。

**追问：**?? 和 || 的区别（0/'' 是否兜底）；allSettled 和 all 的区别（是否短路）；怎么确认特性可用（CanIUse + 目标浏览器矩阵）。

---

## 02 标准归属与版本订正

核验于 2026-09-30：Array.at 属于 ES2022，findLast/findLastIndex 属于 ES2023；structuredClone 是 HTML 标准定义的宿主 API，不是 ES2022 的语言内建。Record/Tuple 提案已于 2025-04 撤回并归档，不能继续说“正在 Stage 2 推进”。Decorators 应以 TC39 当前提案列表为准，本次核验列表列于 Stage 2.7，历史文章的 Stage 3 标签不可直接沿用。

---

## 03 讲特性时带上失效条件

可选链只处理 null/undefined，不会吞掉 getter 自身抛出的异常；BigInt 不能直接与 Number 混合进行大多数算术运算；Promise.all 的拒绝不会自动取消其他任务。structuredClone 可处理循环引用和部分内建类型，但不是任意对象的万能复制工具，函数等不可克隆，原型与属性描述符也不能假设完整保留。

---

## 04 参考

- [ECMAScript 2023：findLast](https://tc39.es/ecma262/2023/multipage/indexed-collections.html#sec-array.prototype.findlast)
- [HTML 标准：structuredClone](https://html.spec.whatwg.org/multipage/structured-data.html#structured-cloning)
- [TC39：Record/Tuple 撤回说明](https://github.com/tc39/proposal-record-tuple/issues/394)
- [TC39 当前提案清单](https://github.com/tc39/proposals)

---

## 05 相关问题与延伸

- [[八股/11-前端/05-场景与表达/04-HardMan 这类任务如何分析和实现|HardMan 这类任务如何分析和实现]]：链式异步任务与语言能力
- [[八股/11-前端/03-JavaScript与状态管理/02-ES6 的含义是什么？ECMAScript 语言规范如何迭代|ES6 的含义是什么？ECMAScript 语言规范如何迭代]]：规范演进与具体特性状态

---

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
