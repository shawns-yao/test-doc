---
aliases:
- "什么时候用单 Agent、多 Agent、Autonomous Agent？"
- "AI 4.6 什么时候用单 Agent、多 Agent、Autonomous Agent？"
---

# 06 什么时候用单 Agent、多 Agent、Autonomous Agent？

## 01 核心回答

**选型：**单 Agent 适合**流程短、目标清晰、风险可控**场景；多 Agent 适合**复杂任务拆解与并行协作**（Planner/Executor/Critic 分工）；Autonomous Agent 强调**长链路自主闭环**，但需要更强安全边界。把 Agent 想象成组织中的员工，要看具体做的事情的大小和复杂度。

**多 Agent 特点：**潜在优势是分工与并行，但复杂度集中在**通信协议、状态一致性和路由收敛**。

Autonomous vs Multi-Agent：Autonomous 强调授权范围内的自主闭环，不限定单体，是否适用需同时评估动态决策收益、风险和运行成本；Multi-Agent 强调多角色分工协作，适合复杂任务拆解、并行处理和交叉审校。工程上最大差异在编排复杂度和故障域：单体方案简单但一旦决策偏差影响全链路；多 Agent 治理更复杂，但可局部隔离和重试。

**降级时机：**任务风险高、动作不可逆或系统指标恶化时，把 Autonomous Agent 降级为半自动——保留检索和生成能力，把最终执行权交给人工审批，先保正确性和可追责。

---

## 02 概念纠正与选型标准

Autonomous 描述自治程度，single/multi 描述智能体数量，是两个维度：单 Agent 或多 Agent 都可能高度自治。不能把 Autonomous 定义为单体，也不能默认它成本低。

是否拆多 Agent 看任务能否独立验收、上下文是否可隔离、并行收益是否超过通信和合并成本。简单串行任务先用单 Agent 或显式工作流；多人审查式角色分工只有带来可测收益才值得加入，同一模型换角色不构成独立专家保证。

口述：“先定自治边界，再决定是否拆角色；结构复杂度和行动权限分开评估。”参见[[八股/07-AI与Agent/02-Agent原理与编排/08-多智能体系统相比单 Agent 有什么优势？引入哪些新复杂性|多 Agent 的收益与代价]]。

## 03 关联追问

- [[八股/07-AI与Agent/12-多Agent复习/01-什么场景下必须上 Multi-Agent,单 Agent 搞不定|什么场景下必须上 Multi-Agent,单 Agent 搞不定?]]

## 04 参考资料

- [Anthropic 关于工作流与 Agent 的工程说明](https://www.anthropic.com/engineering/building-effective-agents)

## 05 直接追问：如何定义降级与恢复门槛

先分硬门槛和运行指标。未获授权、动作不可逆且缺少核验条件、工具返回结果未知时，应立即停在待确认或人工接管状态，不等平均成功率报警。运行指标则按任务风险定义成功率、超时率、循环率与预算的 SLO，并约定统计窗口和最小样本量，避免一次偶发抖动触发全局切换。

恢复不应只看“工具又能连上”：先核验在途动作，回放失败样本，再小流量放开；恢复阈值与触发阈值可留出间隔，避免反复切换。具体百分比必须来自业务基线与风险容忍度，不能把演示数字说成真实线上成果。

来源与改写说明：本节依据 [goehou/agent_java_offer 仓库贡献者的题目与资料](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/01_AI/02_Workflow%E4%B8%8E%E5%A4%9AAgent/01_%E6%A0%B8%E5%BF%83%E9%97%AE%E7%AD%94.md#L11-L21)（[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)）补充。将原资料只提出的降级阈值追问补成可执行判据；不引入未经验证的成功率数字。 许可说明仅对应本节引入的来源内容，不改变本篇其他原有内容的许可。

## 06 所属专题
- [[八股/07-AI与Agent/02-Agent原理与编排/00-Agent原理与编排导航|Agent原理与编排导航]]
