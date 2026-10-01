---
aliases:
- "JavaScript 模块化有哪些方案？Node.js 模块化和浏览器模块化有什么区别？"
- "前端 3.1 JavaScript 模块化有哪些方案？Node.js 模块化和浏览器模块化有什么区别？"
---

# 01 JavaScript 模块化有哪些方案？Node.js 模块化和浏览器模块化有什么区别？

## 01 核心回答


**方案演进**
IIFE/全局变量 → CommonJS（Node）→ AMD/CMD（浏览器早期）→ **ES Module（ESM，标准）**→ UMD（兼容层）。

**CommonJS vs ESM**
CJS：require/module.exports，**同步加载**、运行时确定、导出值或对象引用；ESM：import/export，**静态分析**（可 Tree-shaking）、异步加载、实时绑定。

**Node vs 浏览器：**Node 同时支持 CommonJS 和 ESM，按**文件扩展名与包配置等规则**识别；浏览器模块必须 ESM（`<script type="module">`），**异步加载**（网络请求），支持动态 import。Node 的 require 同步执行仍有阻塞成本，浏览器原生 ESM 通过模块加载器获取依赖；不要与普通经典脚本的加载执行方式混淆。

**追问：**ESM 为什么能 Tree-shaking（静态 import 分析，CJS 运行时 require 做不到）；循环依赖谁更危险（CJS 半成品导出）；动态 import 干什么用（路由懒加载）。

---

## 02 加载机制与绑定语义订正

Node 同时支持 CommonJS 和 ESM，具体按扩展名、package.json 的 type、运行参数及版本规则识别，不能一概说所有 Node 模块都默认 CJS。同步 require 会占用当前线程，本地磁盘并不代表无阻塞成本；现代 Node 对同步 ESM 图的 require 互操作也有版本与顶层 await 限制。

CJS 导出的是 module.exports 的值，对象可被共享引用，不能笼统理解成深拷贝；后续替换导出对象与修改原对象属性也不同。ESM 的静态 import 建立实时绑定，但顶层读取尚未初始化的循环依赖仍可能触发暂时性死区，不能认为 ESM 循环依赖永远安全。

---

## 03 浏览器与构建器的职责

浏览器原生模块按 URL 解析和加载，裸包名通常需要 import map 或构建器处理。动态 import 返回 Promise，适合按需加载；静态 ESM 的链接、求值和顶层 await 不能简单压缩成“所有 ESM 代码都异步运行”。Tree-shaking 除模块格式外还依赖副作用与构建器分析。

---

## 04 参考

- [Node 官方：包类型与模块系统](https://nodejs.org/api/packages.html)

---

## 05 相关问题与延伸

- [[八股/11-前端/02-构建渲染与性能/01-vite build 做了哪些事情|vite build 做了哪些事情]]：模块图与构建产物

---

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
