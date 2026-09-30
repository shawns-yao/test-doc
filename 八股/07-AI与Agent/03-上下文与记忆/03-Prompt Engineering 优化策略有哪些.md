---
aliases:
- "Prompt Engineering 优化策略有哪些？"
- "AI 5.3 Prompt Engineering 优化策略有哪些？"
---

# 03 Prompt Engineering 优化策略有哪些？

## 01 核心回答

把 Prompt 当成**预算管理问题**来做，核心是「高信号、低冗余」五个手段：① **分层提示**——稳定规则放 system、任务信息放 user，避免重复拼接大段固定文本；② **最小必要上下文**——只传当前任务必需信息，历史对话做摘要而非全量回放；③ **检索按需注入**——RAG 只注入 top-k 片段并设 token 上限；④ **结构替代长文本**——用 schema、枚举、字段约束替代长篇说明；⑤ **监测与 A/B**——监控首 token 时延、总 token、成功率和成本，按指标裁剪 Prompt。

**与微调的协同：**两者不是替代关系——Prompt 负责一次交互的快速约束行为和流程编排，微调负责长期固化能力与风格稳定；「Prompt 先行、微调收口」。

---

## 02 Prompt 能约束什么

Prompt 可以提供任务目标、边界、反例和输出规范，但“最多重试两次”“只能读取某目录”等要求若关系安全和成本，应由运行时计数与权限控制强制执行。JSON Schema 能约束形状，不能证明字段真实、完整或业务上可执行。

优化应在固定样本和模型版本上比较，记录每项约束的失败类型；精简文本若导致关键条件消失，token 少并非改进。结构化示例应覆盖容易混淆的正反例，不把其他用户资料放进 few-shot。

口述：“Prompt 优化先减少歧义，再用评测看效果；硬权限和资源边界交给程序，不能靠措辞保证。”

## 03 关联追问

- [[八股/07-AI与Agent/03-上下文与记忆/04-什么是上下文工程？和 Prompt Engineering 的边界|什么是上下文工程？和 Prompt Engineering 的边界？]]

## 04 参考资料

- [Anthropic 上下文工程说明](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [MCP 2025-11-25 Tools 规范](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)

## 05 所属专题

- [[八股/07-AI与Agent/03-上下文与记忆/00-上下文与记忆导航|上下文与记忆导航]]
