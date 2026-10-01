---
aliases:
- "Human-in-the-loop 在 LangGraph 怎么落地？"
- "AI 8.3 Human-in-the-loop 在 LangGraph 怎么落地？"
---

# 03 Human-in-the-loop 在 LangGraph 怎么落地？

## 01 核心回答

在关键节点使用 **interrupt 或审批节点**，让系统在「高风险动作前」暂停，等待人工确认后继续——典型场景是资金操作、批量发送、生产变更。这样保留自动化效率，同时把不可逆动作的最终控制权交给人。

---

## 02 LangGraph 恢复语义与审批安全

需要配置持久 checkpointer，并用相同 thread_id 定位运行；interrupt 返回给调用方的是待处理信息，恢复时通过框架的 resume 机制提供审批结果。恢复可能从节点开头重执行，因此审批之前不能夹带不可幂等副作用。

审批记录应包含审批人身份、操作对象、参数/产物版本和有效期，恢复后重新检查权限与资源状态。用户改了参数，旧审批不一定仍有效；超时或无回复只能保持等待/取消，不能当同意。

口述：“HITL 是可恢复的审批状态机，不只是弹确认框；重点是审批和真实执行内容一致。”详细见[[八股/07-AI与Agent/02-Agent原理与编排/21-Agent的checkpoint是什么|checkpoint]]。

---

## 03 关联追问

- [[八股/07-AI与Agent/02-Agent原理与编排/19-当 Agent 遇到无法解决的场景，应该怎么处理|当 Agent 遇到无法解决的场景，应该怎么处理？]]
- [[八股/07-AI与Agent/09-Agent项目实践/16-设计可真实交易的 Agent，权限和风控边界怎么设|设计可真实交易的 Agent，权限和风控边界怎么设？]]

---

## 04 参考资料

- [LangGraph 官方中断与恢复文档](https://docs.langchain.com/oss/python/langgraph/interrupts)
- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)

---

## 05 所属专题

- [[八股/07-AI与Agent/06-Agent安全/00-Agent安全导航|Agent安全导航]]
