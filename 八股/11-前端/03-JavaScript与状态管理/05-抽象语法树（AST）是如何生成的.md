---
aliases:
- "抽象语法树（AST）是如何生成的？"
- "前端 3.5 抽象语法树（AST）是如何生成的？"
---

# 05 抽象语法树（AST）是如何生成的？

## 01 核心回答


**概念：**AST 是源代码的结构化树形表示——每个节点是一个语法单元（变量声明、函数调用、字面量），是编译器/转译器的中间产物。

**生成流程（两步）：**① **词法分析（Tokenizer）**：源码按规则切成 token 流（关键字/标识符/运算符/字面量，带位置信息）；② **语法分析（Parser）**：按文法规则把 token 流组装成 AST（递归下降/LL/LR），语法错误在此阶段报出。常见工具：Acorn（部分构建工具与插件使用，依版本而定）、babel-parser、esprima、TypeScript 自带 parser。

**AST 的用途：**① Babel/TS 编译（AST 转换 → 生成目标代码）；② ESLint/Prettier（AST 分析 + 格式化）；③ 代码压缩混淆（terser）；④ IDE 智能提示/重构（基于 AST）；⑤ 手写解析器/DSL 时同样思路。

**追问：**词法和语法的区别（切 token vs 组结构）；AST 和 CST 的区别（是否保留无关语法细节）；怎么调试 AST（AST Explorer 可视化）。

---

## 02 从源码到 AST 不是语义正确性证明

词法分析识别 token，语法分析结合优先级、结合性与文法构造树；实际解析器常边取 token 边解析，不一定先存完整 token 数组。得到 AST 只证明语法可解析，名称解析、类型检查、控制流和运行时行为仍需后续阶段处理。

代码变换需要理解绑定作用域，不能只按同名 Identifier 全局替换，否则会误改遮蔽变量。保留 source map、注释及位置元数据有助于调试；不同工具 AST 的节点类型也不完全一致，Babel 格式与 ESTree 有公开差异。

---

## 03 工具版本订正

Acorn、Babel、TypeScript 都有常见解析器实现，但不能永久断言“Vite 就使用 Acorn”：当前 Vite 8 的 Rolldown/Oxc 工具链已不同，应按具体版本和阶段判断。

---

## 04 参考

- [Babel parser：语法模式与 AST 差异](https://babeljs.io/docs/babel-parser)
- [Vite 8 工具链变更](https://vite.dev/blog/announcing-vite8)

---

## 05 相关问题与延伸

- [[八股/11-前端/02-构建渲染与性能/01-vite build 做了哪些事情|vite build 做了哪些事情]]：源码解析与构建转换

---

## 06 所属专题

- [[八股/11-前端/03-JavaScript与状态管理/00-JavaScript与状态管理导航|JavaScript与状态管理导航]]
