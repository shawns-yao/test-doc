---
aliases:
- "为什么选 LangGraph 而不是 CrewAI/AutoGen？合规怎么保证？"
- "Agent 4.24 为什么选 LangGraph 而不是 CrewAI/AutoGen？合规怎么保证？"
---

# 28 为什么选 LangGraph 而不是 CrewAI/AutoGen？合规怎么保证？

## 01 核心回答

为什么 LangGraph：显式状态图及持久化机制可支持恢复与审批；框架能力应按版本对比，医疗合规适用性须另行审查，不能由 LangGraph 或其他框架保证；LangGraph 支持循环边（cycles）+ Human-in-the-Loop，匹配 Critic/Verifier/Auditor 步骤。

**Agent 失败处理：**conditional edges + retry 机制（指数退避）；Gatekeeper Agent 先做风险评估，失败自动路由到 HITL 节点并记录完整日志（Prompt + Output + Error）；检索不到最新指南时标记「证据不足」。

合规（HIPAA/GDPR/NMPA）：按适用规则做数据最小化、身份与访问控制、风险评估、审计和保留治理；姓名/MRN 令牌化不必然匿名，私有化/零保留与检查点也不等于自动合规。

多模态扩展与迭代：若处理医学影像，需先验证模型适用用途、性能与安全并由专业人员审查，不能把通用 Vision 节点输出直接当临床报告；每周收集医生反馈触发 RAG 更新 + Prompt 优化，用 LangSmith 监控每 Agent 的 Latency & Success Rate 自动告警。

---

## 02 框架选择不能证明合规

LangGraph 提供显式状态与持久化/中断能力，可适合可恢复流程；CrewAI 当前也有 Flows、状态与持久化等能力，不能简单断言只能做原型或状态管理弱。应按固定版本比较恢复、审批、观测、部署和升级成本。

HIPAA、GDPR 与中国医疗产品监管并非同一套规则，也不是所有应用同时适用。是否涉及受保护健康信息、产品预期用途、参与主体、部署地与数据跨境等，需要由合规/法律及专业人员确认。checkpoint、私有部署或“零保留”配置都不是合规认证。

## 03 技术措施的边界

把姓名/MRN 替换成 token 可能仍可重识别，不能直接称为匿名化。还要考虑访问控制、身份认证、传输与存储保护、审计、保留删除、供应商协议及风险评估；日志与检查点本身也可能含敏感数据。只读到公开文献与处理患者资料应采用不同数据边界。

医学影像模型输出不能未经验证当作临床报告；引入新模型需要领域性能、安全和适用用途评估，而非简单加一个 Vision Agent 节点。

口述：“选框架是实现可控流程，合规是适用规则、组织流程与技术控制共同满足，不能用框架名字代替审查。”

## 04 关联追问

- [[八股/07-AI与Agent/06-Agent安全/01-如何确保 Agent 安全、可控、可追责|如何确保 Agent 安全、可控、可追责？]]

## 05 参考资料

- [LangChain 官方概览](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)
- [CrewAI Flows](https://docs.crewai.com/en/concepts/flows)
- [HHS HIPAA Security Rule 官方概述](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html)
- [HHS 去标识化官方说明](https://www.hhs.gov/hipaa/for-professionals/special-topics/de-identification/index.html)

## 06 所属专题

- [[八股/07-AI与Agent/09-Agent项目实践/00-Agent项目实践导航|Agent项目实践导航]]

## 07 补充研究型多模态与访问控制示例

研究型多模态原型可参考 CheXagent 的胸部 X 光任务，把候选发现连同输入标识、模型限制和来源整理成结构化产物，再供主流程核对。其官方明确仅供研究、不得用于临床，不能拿来说明系统可以直接自动生成临床报告。

入口 Gatekeeper 可以对敏感字段令牌化，由运行时身份与字段级权限限制谁可读取原始资料，日志也须脱敏；令牌化只减少直接暴露，并不自动构成匿名化或合规证明，仍需处理反向映射保护、授权、审计和保留期限。

- [原始资料](https://github.com/Stanford-AIMI/CheXagent)
