---
aliases:
- "LangChain 和 LangGraph 的关系与区别？"
- "AI 9.3 LangChain 和 LangGraph 的关系与区别？"
---

# 03 LangChain 和 LangGraph 的关系与区别？

## 01 核心回答

LangChain 更偏**组件库和表达式编排**（Prompt、Model、Retriever、Tools、Runnable），适合快速搭建链路；LangGraph 是面向**有状态、多步骤、可恢复任务**的图编排框架，适合 Agent 工作流。两者不是替代关系，常见是「LangChain 组件 + LangGraph 状态机编排」。

升级时机：当流程从「线性调用」变成「有状态、有分支、可恢复、可治理」时，若高层接口不足以清晰表达状态与恢复，可直接使用 LangGraph 图。

---

## 02 当前关系与直接追问

LangChain 的高层 Agent API 基于 LangGraph 运行，不宜再把 LangChain 仅描述成线性 Chain 库；LangGraph 是更底层的状态与执行编排能力，也能不用 LangChain 独立构建。

选择高层接口还是直接写图，取决于是否需要自定义状态、路由、检查点与中断控制。Runnable 组合不等于只能线性，框架名称也不决定系统是否自治。

口述：“先用高层 API 快速起步，需要精细控制状态生命周期和恢复路径时下沉到图层。”参考[[八股/07-AI与Agent/07-框架协议与工程化/04-LangGraph 生产治理怎么做|图的生产治理]]。

## 03 关联追问

- [[八股/07-AI与Agent/08-数据Agent设计/07-数据Agent中如何选择LangChain和LangGraph|数据 Agent 中如何选择 LangChain 和 LangGraph]]
- [[八股/07-AI与Agent/11-Agent工程复习/01-LangChain LangGraph AutoGen CrewAI 用过哪个为什么选它,踩过什么坑|LangChain / LangGraph / AutoGen / CrewAI 用过哪个?为什么选它,踩过什么坑?]]

## 04 参考资料

- [LangChain 官方概览](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)

## 05 所属专题

- [[八股/07-AI与Agent/07-框架协议与工程化/00-框架协议与工程化导航|框架协议与工程化导航]]
