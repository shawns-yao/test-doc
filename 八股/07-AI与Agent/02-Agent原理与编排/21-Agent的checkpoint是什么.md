---
aliases:
- "Agent的checkpoint是什么"
---

# 21 Agent的checkpoint是什么

## 01 核心结论

Checkpoint 是 Agent 运行过程的持久化状态快照，用来知道“已经做了什么、下一步从哪里继续”。它不是模型权重，也不是仅保存聊天记录，更不是外部业务系统的自动回滚点。

---

## 02 保存什么与怎样恢复

以 LangGraph 为例，checkpointer 按线程保存图执行的状态历史。完整 checkpoint 通常位于 super-step 边界：同一轮调度的节点可并行执行，节点完成的 pending writes 可用于故障恢复，但不等同一份完整快照。这不是任意代码指令位置的进程快照。典型内容包括状态值、待执行任务、运行元数据和检查点关联信息。thread_id 用来定位会话运行历史；生产要将它与租户身份一起校验，不能仅凭客户端给的 ID 读取别人的状态。

恢复链路是：加载检查点 → 校验运行版本和依赖状态 → 恢复待执行节点 → 再次校验结果。大文件、长工具结果适合放外部存储，检查点保存引用和版本；只保存引用时要确保引用内容仍存在且未被改写。进程内存型 saver 适合调试，跨进程恢复需要真正持久存储和备份策略。

---

## 03 为什么 checkpoint 不保证恰好一次

假设节点已经向外部服务创建订单，但保存检查点前进程崩溃。恢复后节点可能再次执行，造成重复订单。正确做法是持久化业务幂等键、查询外部结果、明确 pending/confirmed/unknown 状态；必要时使用事务消息或补偿流程。单独一份图快照无法和任意第三方 API 原子提交。

LangGraph 中断恢复时节点可能从头重新执行，因此 interrupt 之前的副作用应移出节点、保证幂等或采用受控任务封装。回放也不等于撤销，取消订单是另一次有权限约束的业务动作。

---

## 04 追问与口述

- “checkpoint 与长期记忆区别？”前者面向一次运行恢复，后者面向跨任务知识与偏好；二者可存同一数据库，但生命周期、权限、检索方式不同。
- “升级图后旧 checkpoint 能恢复吗？”要做状态 Schema 版本与迁移兼容测试，不能默认旧状态适配新节点代码。
- 口述：“checkpoint 让任务断了能续，幂等和业务状态核验让续跑不重复做事。二者解决不同层面的可靠性。”

相关：[[八股/07-AI与Agent/02-Agent原理与编排/22-记忆具体存储在哪里|记忆物理存储]]；[[八股/07-AI与Agent/07-框架协议与工程化/04-LangGraph 生产治理怎么做|LangGraph 生产治理]]；[[八股/07-AI与Agent/06-Agent安全/03-Human-in-the-loop 在 LangGraph 怎么落地|中断审批]]

---

## 05 关联追问

- [[八股/07-AI与Agent/02-Agent原理与编排/10-Multi-Agent 实际项目怎么设计？LangGraph 里怎么编排更稳|Multi-Agent 实际项目怎么设计？LangGraph 里怎么编排更稳？]]
- [[八股/07-AI与Agent/02-Agent原理与编排/16-Browser Agent 的关键失败点有哪些？如何做重试和回滚|Browser Agent 的关键失败点有哪些？如何做重试和回滚？]]
- [[八股/07-AI与Agent/02-Agent原理与编排/19-当 Agent 遇到无法解决的场景，应该怎么处理|当 Agent 遇到无法解决的场景，应该怎么处理？]]
- [[八股/07-AI与Agent/03-上下文与记忆/01-Agent 的短期记忆、长期记忆和状态怎么设计|Agent 的短期记忆、长期记忆和状态怎么设计？]]
- [[八股/07-AI与Agent/09-Agent项目实践/14-下单为什么设计 clientOid？怎么做幂等|下单为什么设计 clientOid？怎么做幂等？]]
- [[八股/07-AI与Agent/09-Agent项目实践/34-如果做一个 Agent 创作助手，工作流怎么编排、状态怎么保存、怎么观测|如果做一个 Agent 创作助手，工作流怎么编排、状态怎么保存、怎么观测？]]
- [[八股/07-AI与Agent/14-Agent架构复习/02-如果任务要跑几十步才能完成,怎么保证 Agent 不跑偏、不断链|如果任务要跑几十步才能完成,怎么保证 Agent 不跑偏、不断链?]]

---

## 06 参考资料

- [LangGraph 官方 Checkpointers 详细机制](https://docs.langchain.com/oss/python/langgraph/checkpointers)
- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)
- [LangGraph 官方中断与恢复文档](https://docs.langchain.com/oss/python/langgraph/interrupts)

---

## 07 所属专题

- [[八股/07-AI与Agent/02-Agent原理与编排/00-Agent原理与编排导航|Agent原理与编排导航]]
