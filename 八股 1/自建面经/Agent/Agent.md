# Agent 面经

## 一、Agent 设计与治理

# 1.1 如果让你提升 Agent 生成 SQL 或数据链路的准确率，你会怎么做？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">把问题拆成<b>“理解需求、检索知识、生成 SQL、执行校验、结果解释”</b>五个环节，逐环加约束和反馈：</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">元数据与约束：</b>先建立统一的指标、表、字段和血缘元数据，使用结构化 Schema 约束模型输出。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">知识检索：</b>生成 SQL 前检索相关表结构、字段含义、样例值和历史正确 SQL，减少幻觉。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">执行前校验：</b>SQL Parser 做语法和权限校验，再通过 <code>EXPLAIN</code>、成本限制和只读连接做安全检查。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">结果校验与修正：</b>执行后进行行数、空值、时间范围和指标口径校验，失败则把错误信息反馈给 Agent 进行<b>有限次数修正</b>。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">5</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">评测闭环：</b>用固定评测集衡量 SQL 执行成功率、结果正确率、关键链路召回率和 Token 成本。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么生成前要检索知识（减少幻觉、对齐口径）；校验失败让 Agent 自纠几次（有限次数，防死循环）；评测指标怎么定（执行率 + 正确率 + 成本）。</div>
  </div>
</div>

---

# 1.2 假设要开发一个需求开发 Agent，输入需求可能是加工数据表、制作 BI 看板或对接数据产品应用。如果由你设计这个 Agent 的整体工作流程，你会怎么做？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 意图识别与拆解：</b>对自然语言需求做意图识别和结构化拆解，提取业务目标、数据范围、指标口径、产出类型和验收条件。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 检索与方案确认：</b>检索数据目录、指标字典、表血缘和历史案例，生成候选方案并让用户确认关键口径。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 分类型计划：</b>确认后进入计划阶段——加工数据表：生成数据模型、SQL 和调度依赖；制作 BI 看板：生成指标、维度、图表和权限配置；对接数据产品：生成接口、字段映射和联调计划。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">④ 安全执行：</b>工具白名单、参数校验、沙箱或测试环境和人工审批。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">⑤ 自动验证与交付：</b>完成后自动运行数据质量校验、接口测试和看板验收，最终输出变更记录和可回滚方案。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么关键口径要人工确认（LLM 理解歧义，一次确认省返工）；沙箱执行的意义（防误改生产）；回滚方案为什么必须（AI 产物不可控，留退路）。</div>
  </div>
</div>

---

# 1.3 现在平台已有 AI 解析日志并给出排查建议。如果交给你从零实现这套告警 AI 诊断能力，你会怎么设计？告警诊断包含哪些分类？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">设计成<b>“告警接入、上下文补全、假设生成、分层取证、根因判断、处置建议、结果反馈”</b>七步链路：</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">告警接入：</b>去重、聚合并关联服务、版本、部署和时间窗口，提取错误指纹、指标异常和最近变更。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">上下文补全：</b>聚合告警相关上下文，构建受限证据上下文（关联版本、变更、时间窗）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">假设生成：</b>Agent 生成多个结构化竞争假设（根因候选/支持征兆/会否证它的证据）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">分层取证：</b>按区分度和取证成本调用日志、Trace、Kubernetes、配置和发布记录等<b>只读工具</b>，逐轮验证和剪枝。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">5</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">根因判断与处置建议：</b>输出必须经过 Schema 校验、证据引用和置信度检查，涉及副作用的操作必须审批。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">6</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">结果反馈：</b>诊断结果回流，持续优化（正确/错误案例进评测集）。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">诊断分类：</b>应用异常、依赖服务异常、数据库与缓存、网络与 DNS、资源与容量、配置或发布变更、数据质量、权限与证书、基础设施故障。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">安全边界：</b>项目中把 LLM 限制为<b>提交建议事件</b>，状态机负责确定性流转，避免模型直接改变生产状态。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么取证工具只读（生产安全 + 可审计）；假设为什么要多个（竞争剪枝防一条路走到黑）；LLM 为什么不直接改状态（不可控 + 审计要求）。</div>
  </div>
</div>

---

# 1.4 数仓领域和机器学习结合，你觉得有哪些潜力场景？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">行为预测</b><br>用户或商品行为预测，用于留存、流失、推荐和营销。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">异常检测</b><br>监控指标突变、数据质量和风控。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">需求预测</b><br>预测与资源调度。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">实体解析</b><br>实体解析与指标口径对齐。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">NL2SQL/BI 问答</b><br>自然语言取数和 BI 问答。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分工：</b>数仓负责提供统一、可追溯的特征和标签，机器学习负责预测或分类。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">落地注意：</b>特征穿越、样本偏差、模型漂移和结果回流。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>特征穿越是什么（用未来数据训练）；模型漂移怎么监测（指标分布 + 定期重训）；结果回流的意义（预测结果回到数仓形成闭环）。</div>
  </div>
</div>

---

# 1.5 数仓或数据领域，有没有印象比较深、值得推荐的 Agent 开源项目？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">推荐方向：</b>优先关注面向 <b>Text-to-SQL、数据目录和指标治理</b>的项目——Vanna AI、DB-GPT、Dataherald（NL2SQL）；DataHub、OpenMetadata（元数据与血缘平台）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">评估标准：</b>选择时不只看 Demo，重点评估 <b>Schema 检索、权限隔离、SQL 校验、执行反馈、评测集和可观测性</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">判断：</b>开源项目适合用来验证交互和基线，生产系统仍需要结合企业元数据、权限和数据质量体系做定制。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么 NL2SQL 项目都强调 Schema 检索（不检索就幻觉）；开源和自研怎么选（开源做基线，核心定制自研）；权限隔离为什么难（企业级行级/列级权限）。</div>
  </div>
</div>

---

# 1.6 MCP 和 Skills 的区别？LangChain 和 LangGraph 的区别？短期记忆和长期记忆怎么划分？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：字节飞书、得物、招银</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MCP vs Skills</b>
        <div style="margin:4px 0 0;"><b>MCP（Model Context Protocol）</b>是工具接入的<b>协议标准</b>：定义模型如何发现、调用外部工具/数据源（类似 USB 接口，一套协议接各种设备）；<b>Skills</b> 是<b>能力封装</b>：把特定任务的指令、流程、参考文档打包成可复用的技能包。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">区别：</b>MCP 解决"怎么连"（标准化工具调用通道），Skills 解决"怎么干"（任务级方法论）；MCP 偏运行时协议，Skills 偏知识与流程资产。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">LangChain vs LangGraph</b>
        <div style="margin:4px 0 0;"><b>LangChain</b>：链式编排（Chain 线性/顺序组合，LCEL 声明式）——适合固定流程；<b>LangGraph</b>：<b>图状态机</b>编排——节点 + 边 + 状态，支持条件分支、循环、人机协同和持久化。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">区别：</b>LangGraph 更擅长复杂 Agent（循环反思、多 Agent 协作、中断恢复）；LangChain 简单流程够用。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">短期记忆 vs 长期记忆</b>
        <div style="margin:4px 0 0;"><b>短期记忆</b>：当前会话上下文（对话历史、槽位、进行中任务状态）——放 Redis/内存，有 TTL；<b>长期记忆</b>：跨会话的用户偏好、历史结论、知识积累——放数据库/向量库，按用户维度持久化。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">划分原则：</b>按<b>生命周期和复用价值</b>——会话内临时状态短期化，跨会话有价值的信息（用户画像、历史决策、错误教训）沉淀长期记忆，检索时按相关性注入。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>MCP 和插件系统的区别（开放协议 vs 平台私有）；LangGraph 的循环怎么防死循环（最大迭代次数 + 条件收敛）；长期记忆怎么避免过期（版本 + 更新时间 + 冲突检测）。</div>
  </div>
</div>

## 二、项目与大模型

---

# 2.1 为什么不直接使用大模型来开发整个系统？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">大模型适合自然语言理解、信息抽取、代码生成和方案建议，但<b>不适合独立承担强确定性、强一致性和高安全要求的核心链路</b>。直接让模型开发整个系统的四类风险：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">随机性与幻觉</b><br>输出存在随机性，不能保证每次生成相同且正确的结果。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">业务约束缺失</b><br>不了解全部业务约束，容易遗漏权限、幂等、事务和异常分支。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">成本与依赖</b><br>调用带来延迟、Token 成本、上下文长度和供应商依赖。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">不可审计副作用</b><br>模型直接执行工具可能越权读写、误删数据或产生不可审计的副作用。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">更合理的架构：</b>“<b>模型负责理解和建议，程序负责约束和执行</b>”——模型把自然语言转换为结构化意图、参数和候选计划，系统通过 Schema 校验、参数范围校验、权限校验、SQL 解析、工具白名单和状态机控制后续流程；高风险操作加用户确认或人工审批，所有调用保留输入、模型版本、提示词、工具参数、执行结果和审计记录。正确性、可回滚性和合规性掌握在确定性系统中。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>模型幻觉怎么防（Schema + 检索 + 校验闭环）；为什么高风险操作要审批（不可审计副作用）；"程序负责执行"具体指什么（状态机/白名单/条件更新）。</div>
  </div>
</div>

---

# 2.2 项目中的大模型应用有哪些具体细节？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">从<b>“输入、检索、推理、执行、校验、反馈”</b>六个环节说明：</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">输入：</b>意图识别、会话上下文整理和槽位提取。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">检索：</b>RAG 获取项目文档、指标口径、接口定义、历史案例和权限信息。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">推理：</b>角色、任务边界、输出格式和失败处理约束模型，要求返回结构化 JSON。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">执行：</b>只允许调用注册过的只读或受控工具（查询日志、指标、知识树、配置版本）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">5</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">校验：</b>模型输出不能直接作为结论——JSON Schema、字段类型、枚举值、权限范围、时间窗口和业务规则校验；SQL/脚本做语法解析、危险操作拦截和成本限制；执行后把工具结果交给校验器或模型做<b>证据比对</b>。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">6</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">反馈：</b>低置信度、证据不足或高风险操作转人工；线上记录 Prompt、模型版本、检索片段、工具调用、耗时、Token、错误和反馈，用固定评测集观察正确率、成功率、延迟和成本。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>证据比对怎么实现（结论与工具返回逐项核对）；转人工的标准（置信度阈值 + 风险等级）；评测集怎么建（真实历史问题标注）。</div>
  </div>
</div>


---

# 2.3 项目最大的亮点是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">亮点概括：</b>“<b>模型理解、知识增强、工具执行和确定性治理</b>”的组合，而不是简单接入一个聊天接口——模型负责把自然语言需求转换成结构化意图和候选方案，RAG 提供领域知识和历史经验，工具层获取实时事实，状态机和规则引擎负责流程推进、权限控制、幂等和副作用隔离。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">三点价值：</b>① 用户可以用自然语言完成原本需要查文档、写 SQL 或排查日志的工作；② 每个结论都能关联<b>检索内容和工具证据</b>，便于解释和复盘；③ 模型出错时不会直接改变生产状态，通过校验、人工确认、灰度和回滚降低风险。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">加分项：</b>补充评测方式——用真实历史问题构建数据集，分别统计意图识别、工具调用、结果正确率、平均延迟和人工接管率。</div>
  </div>
</div>

---

# 2.4 项目中遇到过什么比较难的问题？如何定位和解决？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心难点：</b>把<b>不确定的自然语言请求</b>接入<b>确定性的业务系统</b>——四类问题：① <b>需求表达不完整</b>（同一句话可能缺少对象、时间范围、指标口径或权限信息）；② <b>知识与数据变化</b>（模型检索到的文档可能过期或互相冲突）；③ <b>工具调用不稳定</b>（超时、限流、权限和部分成功）；④ <b>难以单一衡量</b>（模型输出不只用准确率衡量，还要看证据充分性、业务可执行性和风险）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">先拆阶段再定位：</b>把问题拆成输入理解、知识检索、模型生成、工具执行和结果交付几个阶段，每阶段加<b>请求 ID、Trace ID、耗时、输入输出摘要和错误码</b>，分别记录成功率、P95 延迟、Token 消耗和人工接管率。先判断是偶发超时、数据错误、模型幻觉、上下文缺失，还是工具本身的权限、连接和幂等问题——避免只看最终页面的失败提示。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">定位手段：</b>结合应用日志、链路 Trace、模型请求记录、检索命中内容、Prompt 版本和工具返回结果做<b>回放</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分类解决：</b>① 模型输出不稳定 → 结构化输出、少样本示例、字段枚举和结果校验；② 检索不准 → 调整切分、元数据过滤、关键词与向量混合检索和重排序；③ 工具问题 → 超时、有限重试、熔断、幂等键和降级；④ 数据口径问题 → 修复知识版本、字段定义和更新流程。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">闭环：</b>修复后用固定样本和线上失败样本回归验证，并把监控、告警、Runbook 和复盘结论沉淀下来。</div>
  </div>
</div>


---

# 2.5 项目后续考虑迭代哪些功能？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">基础层</b><br>覆盖真实场景的评测集和回放平台，跟踪意图识别准确率、任务完成率、证据命中率、P95 延迟、Token 成本和人工接管率。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">数据层</b><br>知识库的版本、血缘、权限、过期提醒和冲突检测。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">执行层</b><br>工具编排、并行调用、超时重试、缓存、幂等、补偿和多模型路由。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">治理层</b><br>高风险操作审批、审计、灰度、回滚、敏感信息脱敏和提示词注入防护。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">产品层</b><br>多轮澄清、用户反馈、结果解释、历史任务复用和个性化配置。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">优先级：</b>按高频失败场景、风险等级和业务收益排序——先解决影响正确性和稳定性的瓶颈，再扩展功能数量。</div>
  </div>
</div>

## 三、百度补充：Agent 与项目实践

---

# 3.1 在 Agent 项目中，如何做风险闭环？如何判断用户意图？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">风险闭环（六步）：</b>“识别、评估、拦截、处置、验证、复盘”——识别阶段检测提示词注入、敏感数据、越权请求、危险 SQL 和高风险业务动作；评估阶段根据用户身份、资源范围、操作类型、影响范围和模型置信度计算风险等级。处置分级：<b>低风险自动执行、中风险用户二次确认、高风险人工审批或直接拒绝</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">意图判断（不只靠一句自然语言）：</b>① 意图分类（查询/创建/修改/删除/分析/咨询）；② 提取对象、时间、范围、条件、输出格式和权限等<b>槽位</b>；③ 结合历史对话补全信息；④ 检查槽位是否完整、类型是否正确、实体是否存在、用户是否有权限；⑤ 多意图请求拆成任务计划逐个确认。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">审计闭环：</b>把意图、槽位、风险等级、规则命中和执行结果写入审计记录，异常时支持回放、人工接管、撤销或补偿。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>风险等级怎么算（身份 × 资源范围 × 操作类型 × 影响 × 置信度）；提示词注入怎么检测（敏感模式 + 工具参数校验）；撤销/补偿怎么做（状态机 + 幂等）。</div>
  </div>
</div>

---

# 3.2 Redis 和 MySQL 在 Agent 项目中分别承担什么用途？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Redis（低延迟/临时状态）</b>
        <div style="margin:4px 0 0;">会话上下文、槽位补全状态、任务锁、幂等键、限流计数、验证码、短期 Token、热点知识和模型结果缓存。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">边界：</b>读写快、数据结构丰富、支持过期和原子操作，但内存容量有限——<b>不能把唯一事实只放 Redis</b>。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL（持久化/事务）</b>
        <div style="margin:4px 0 0;">用户与权限、Agent 配置、知识条目及版本、工具定义、执行记录、审计日志和反馈结果。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">链路：</b>MySQL 为事实来源，高频读取缓存到 Redis；更新通过事务、版本号、消息通知或延迟双删使缓存最终一致；任务状态设计幂等和恢复机制，避免 Redis 过期或节点故障导致重复执行。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>任务锁放 Redis 挂了怎么办（锁超时 + 幂等 + 数据库状态兜底）；为什么审计必须落 MySQL（持久化 + 事务）；缓存最终一致怎么做（版本号 + 双删）。</div>
  </div>
</div>


---

# 3.3 实习项目中的知识树查询如何保证一致性？如何进行性能优化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">一致性：</b>① 明确知识树的主键、父子关系、排序和<b>版本模型</b>；② 每次发布生成新的版本号或快照 ID，节点内容、边关系和索引状态在<b>同一事务</b>提交；③ 查询时固定读取一个已发布版本——不能让父节点来自新版本、子节点来自旧版本；④ 更新走<b>草稿态 → 审核态 → 发布态</b>，完整校验通过才切换当前版本；⑤ 删除/移动节点检查子树、引用关系和权限，保留历史版本支持回滚。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">性能优化：</b>① <code>parent_id</code>、版本号、状态和排序字段建<b>联合索引</b>，避免逐节点递归查询造成 N+1；② 一次批量读取节点、内存中组装树，或使用<b>物化路径/闭包表</b>减少深层遍历；③ 热点节点和已发布快照放 Redis，<b>缓存键带版本号</b>避免更新后读旧数据；④ 更新成功后通过消息或版本切换淘汰缓存；⑤ 监控慢查询、树深度、查询节点数、缓存命中率和 P95 延迟，用真实数据压测验证。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么节点和边要同事务提交（防止读到半棵树）；闭包表是什么（存所有祖先-后代对，查询快写入重）；缓存键带版本号的意义（版本切换即缓存失效）。</div>
  </div>
</div>

---

# 3.4 进行 AI Coding 时，你会让 AI 做什么，自己负责什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">AI 负责（重复劳动）</b>
        <div style="margin:4px 0 0;">阅读代码、搜索调用链、生成样板代码、接口适配、测试用例初稿、SQL 草稿、文档和重构建议；针对明确的小任务给出多种实现方案。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">使用方式：</b>提供模块边界、接口契约、约束条件和示例，要求先说明修改计划再分步生成，避免一次性改动过大范围。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#B26E00;">我负责（判断）</b>
        <div style="margin:4px 0 0;">需求澄清、架构设计、数据模型、并发与事务边界、权限和安全、技术取舍、关键代码以及最终验收。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">红线：</b>AI 内容必须经过人工审阅、编译、静态检查、单元测试、集成测试和性能验证；支付、权限、敏感数据、并发控制、数据库迁移和生产配置的代码<b>不能直接采纳</b>；注意代码来源、许可证、隐私和提示词泄露风险。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">核心原则：</b>让 AI 提高检索和实现效率，但由<b>人</b>对正确性、可维护性和上线风险负责。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>AI 写的代码怎么验收（评审 + 测试 + 边界审查）；为什么关键代码自己写（正确性和可维护性责任）；提示词泄露风险怎么防（不喂敏感信息给外部模型）。</div>
  </div>
</div>

## 四、项目表达与场景

---

# 4.1 Sub-agent 的收益到底是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（除了并行还有五点）：</b>
    <div style="margin:6px 0 0;">① <b>上下文隔离</b>——每个子代理只看自己需要的信息，减少主上下文污染、幻觉和 prompt 变长问题；② <b>职责专业化</b>——按角色拆分（检索、执行、审校、风控），每个子代理用不同提示词/工具/模型更稳；③ <b>故障隔离</b>——某个子代理跑偏或失败不拖垮整条链路，可局部重试、局部回滚；④ <b>权限最小化</b>——给不同子代理不同权限（只读/可写/高危工具禁用），安全边界更清晰；⑤ <b>成本与性能分层</b>——简单任务给小模型、关键步骤给强模型，「该省省、该强强」。</div>
  </div>
</div>

---

# 4.2 规则知识库和 embedding memory 的差别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（硬规则 vs 软记忆）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">规则库：</b>稳定、可审计、可追责——适合合规/风控边界、权限、计费、交易限制。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">embedding：</b>覆盖广、能处理长尾，但可能召回偏差——适合 FAQ、经验复用、个性化上下文、知识问答。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">工程实践：</b>规则库做「硬约束和最终裁决」，embedding 做「语义检索和补充上下文」。</div>
  </div>
</div>

---

# 4.3 知识库内容过期或错误，怎么避免误导模型？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（三层 + 止损）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 入库：</b>所有知识带<b>版本号、生效/失效时间、来源等级和责任人</b>，过期内容自动降权或下线；费率、风控、政策等高危知识人工审核后才能进高可信索引。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 检索：</b>先做 <b>metadata 硬过滤</b>（只查有效时间、可信来源），再做向量 + 关键词混检和重排序，把相似度、新鲜度、可信度一起打分，避免旧文档因语义相近被误召回。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 生成：</b>要求模型「有证据才回答」必须带引用；证据不足或冲突时直接拒答或提示人工确认。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">④ 线上止损：</b>快速封禁问题文档 ID/来源；持续监控过期命中率、引用覆盖率、冲突率、幻觉率，指标异常一键切高可信索引并回滚快照。</div>
  </div>
</div>

---

# 4.4 为什么是「方案驱动」而不是让 AI 直接从 PRD 出码？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">PRD 直接出码容易「看起来对但逻辑错」——需求有歧义、约束没喂全、上下文缺失。方案驱动让 AI 先产出<b>实现方案</b>（影响分析、涉及模块、改动点、风险），人工确认后再写码：① 把需求歧义在方案阶段暴露；② 先做<b>影响分析</b>避免改错范围；③ 方案可评审、可回滚，出码质量前置把关。</div>
  </div>
</div>

---

# 4.5 AI 出码「看起来对但逻辑错」怎么发现和兜底？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（三步）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 写码时用 spec 规范约束：</b>给 AI 硬性「不能做」的约束、正确范例和团队代码规约；测试时把需求拆成可验证用例，覆盖正常、边界、异常三类（空参数、重复请求、并发冲突、超时重试）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 小流量灰度：</b>不只看接口 200，还看真实业务指标——成功率、错误率、延迟、资金和库存是否异常，确认逻辑在真实流量下成立。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 兜底机制：</b>新逻辑挂 <b>Feature Flag</b>（异常一键关闭切回旧逻辑）；写操作加幂等键防重复扣费/下单；关键链路保留审计日志和 request_id 快速回放定位。</div>
  </div>
</div>

---

# 4.6 灰度切流、在线双跑、回滚怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">灰度切流：</b>新旧链路并行，按用户/流量比例逐步切流，日志和埋点双写对比结果；<b>在线双跑：</b>新链路与旧链路同时执行，结果比对（一致性、质量、耗时），差异告警；<b>回滚：</b>Feature Flag/路由开关一键切回，灰度策略与配置版本绑定。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">关键：</b>避免影响真实用户——比对数据只观测不下发，异常先降级回滚再排查。</div>
  </div>
</div>

---

# 4.7 交易型 Agent 产品的后端架构怎么拆？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">主链路：</b><code>Query 理解 → 任务路由 → 检索/RAG → 工具调用 → 风控校验 → 执行 → 结果解释 → 观测与评测</code>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">面试官想听的：</b>不是「我会用 LangChain/Dify」，而是<b>怎么把分析和执行分层、怎么设护栏、怎么追踪一次用户请求到真实下单的全链路</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">Agent vs 固定 workflow：</b>固定流程适合低风险、强约束任务；Agent/planning 适合不确定查询、多轮推理和多工具决策；交易场景里真正高风险的动作<b>不能完全交给自由规划</b>，必须加硬规则/风控闸门。</div>
  </div>
</div>

---

# 4.8 交易系统为什么优先用 WebSocket？REST 怎么配合？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">WebSocket：</b>行情、订单状态、仓位变化适合<b>低延迟实时推送</b>；<b>REST：</b>查询历史、配置类接口、回补丢包。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">系统设计：</b>常是 <b>WebSocket 收实时流 + REST 做补数/对账</b>。</div>
  </div>
</div>

---

# 4.9 Order book 的 snapshot + 增量更新怎么保证正确？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">流程：</b>先拿 snapshot，再合并增量——价格档同价替换、数量为 0 删除、无同价则按 bid 降序/ask 升序插入。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">正确性校验：</b>定期或每次计算 <b>checksum</b>（如 CRC32），对不上就触发<b>重拉快照</b>。</div>
  </div>
</div>

---

# 4.10 下单为什么设计 clientOid？怎么做幂等？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">clientOid：</b>客户端自定义订单 ID——是客户端与服务端之间<b>唯一的幂等保证字段</b>，不传会有重复下单风险。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">顺着讲：</b>请求去重、重试安全、网络超时后的查询补偿、订单状态最终一致性。</div>
  </div>
</div>

---

# 4.11 交易场景怎么做风控？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（四层）：</b>
    <div style="margin:6px 0 0;">① <b>输入风控</b>——参数校验、黑白名单、用户状态；② <b>策略风控</b>——仓位、杠杆、限价/市价约束；③ <b>执行风控</b>——频控、订单上限、重复单；④ <b>结果风控</b>——成交回执、对账、告警。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">模型抖动降级：</b>先降级到纯行情/规则引擎；保留查询和分析、不开放自动执行；缓存最近一次结构化建议；对外返回明确的「不确定」状态；打通 tracing、成本、时延、成功率监控。</div>
  </div>
</div>

---

# 4.12 设计可真实交易的 Agent，权限和风控边界怎么设？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（九个词对应三层防线）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 权限层（先防误用）：</b><b>read-only 默认</b>——新 Agent 默认只能查行情和持仓不能下单；<b>动作分级授权</b>——查阅、模拟下单、真实下单、调杠杆、撤单分别授权；<b>密钥隔离</b>——按环境/策略/账户拆分 API Key，生产密钥放 KMS/HSM、不落盘、定期轮转，子账户隔离。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 交易前风控层（先校验再执行）：</b><b>额度限制</b>——单笔/单日/单策略限额、最大仓位、最大杠杆；<b>风控前置校验</b>——下单网关校验价格偏离、滑点、频率、风险敞口，不通过直接拒单；<b>双确认</b>——大额开仓/高杠杆/新币对二次确认，可人工审批或双签。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 运行与追责层（可追溯可止损）：</b><b>审计日志</b>——完整记录「输入-推理摘要-风控结果-最终指令-交易回执」；<b>回放</b>——按 request_id 回放决策链；<b>沙箱执行</b>——新策略先模拟盘/影子模式，稳定后再灰度真实资金。</div>
  </div>
</div>

---

# 4.13 Prompt injection 在交易场景最危险的后果？怎么防？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#B26E00;">最危险后果：</b>让 Agent「绕过风控去执行高风险真实下单」，造成快速亏损甚至爆仓，而且是<b>自动连续发生</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">五层防护：</b>① <b>执行权隔离</b>——LLM 只产出「建议」，最终下单必须经过确定性风控引擎；② <b>上下文隔离</b>——外部内容（新闻、社媒、RAG）一律标记 untrusted，永不当系统指令；③ <b>工具白名单 + 参数校验</b>——只允许固定动作，杠杆、数量、币对、方向硬校验；④ <b>高危双确认</b>——大额/高杠杆/新策略人工确认或双签；⑤ <b>运行时止损</b>——异常行为检测（频次突增、风险暴露异常）+ 一键熔断 read-only。</div>
  </div>
</div>

---

# 4.14 Skill/plugin 供应链被污染时怎么止损？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">核心：先保证资金安全和边界，再恢复服务——<b>宁可短时降级，也不能带毒运行</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">止损流程：</b>① 把线上大模型权限只给到<b>只读</b>；② 关闭第三方工具调用，暂停高危任务队列；③ 定位影响面和导致问题的插件；④ 插件回滚到安全版本；⑤ 逐步恢复线上服务。</div>
  </div>
</div>

---

# 4.15 交易平台为什么要做 Agent Hub/MCP？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（解决「各自调 API」三大问题）：</b>
    <div style="margin:6px 0 0;">① <b>开发效率</b>——直接调 API 每家都要重复做鉴权、重试、限流、参数校验和工具封装；Hub + MCP 用统一协议和工具描述，接入周期从「周级」降到「天级」；② <b>风险治理</b>——交易场景高风险，平台必须把权限分级、额度限制、风控前置校验、审计日志、熔断降级做成系统能力，避免各家用自研导致安全合规口径碎片化；③ <b>生态与商业化</b>——Hub 能做插件发现、版本管理、灰度发布、计量计费和质量评分，形成网络效应。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">本质：</b>底层保留原生 API 给高阶团队，上层提供 Agent Hub/MCP 给规模化生态——既保灵活性又保平台可控性。</div>
  </div>
</div>

---

# 4.16 通用大模型为什么不能直接解决专业领域写作？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">专业领域（如医药科研）不是只看语言流畅度的场景，而是<b>强依赖证据链、专业术语、时效性和可追溯性</b>——通用模型能写得像，但不代表写得对。核心不是生成，而是<b>检索-筛选-归因-分析-校验-生成</b>这条链路；没有高质量专业数据、引用约束、工具调用和自检机制，模型很容易出现假引用、旧结论、逻辑跳步和专业误判。</div>
  </div>
</div>

---

# 4.17 设计 AI 写科研综述/研究报告的系统？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（七层）：</b>
    <div style="margin:6px 0 0;">① <b>用户意图识别层</b>——综述、竞争情报、靶点评估、临床方案草拟、文献问答；② <b>规划层（Planner）</b>——拆子问题（检索范围、纳入排除标准、证据分层、结果结构）；③ <b>检索层</b>——同时查内部数据库/专业权威数据源；④ <b>排序层</b>——去重、时间过滤、质量评分、可信度打分；⑤ <b>分析层</b>——主题聚类、观点对比、结论归纳、冲突证据标注；⑥ <b>生成层</b>——按目标格式输出，每段挂来源；⑦ <b>校验层</b>——检查引用真实、结论被证据支持、无越权推断。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">加分点：</b>多租户支持（不同医院独立实例）；评估指标——准确率（与专家一致性）、幻觉率、处理速度、ROI；风险控制——Agent 只做「建议 + 证据链」，所有输出带来源溯源。</div>
  </div>
</div>

---

# 4.18 从 0 到 1 搭 AI 后端服务怎么拆？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（七层）：</b>
    <div style="margin:6px 0 0;">① <b>接入层</b>——API Gateway + 鉴权（JWT）+ 限流 + 灰度路由，长响应用 SSE/WebSocket、短请求走 HTTP；② <b>编排层（Agent Runtime）</b>——状态机/图：意图识别 → 检索/工具 → 生成 → 审核 → 返回，每节点超时/重试/最大步数/熔断/降级；③ <b>模型层</b>——模型路由（大/小模型切换）；④ <b>能力层</b>——Prompt 模板中心、Tool 调用、RAG 检索（向量 + 关键词混检）；⑤ <b>数据层</b>——MySQL（用户/订单/任务状态）、Redis（会话缓存）、对象存储（文件/产物）、向量库（知识检索），关键写操作幂等键；⑥ <b>安全与治理层</b>——内容安全、提示词注入防护、工具白名单、最小权限、审计日志；⑦ <b>可观测与运维层</b>——Metrics + Logs + Traces，重点盯成功率、时延、成本、幻觉率、工具失败率，可灰度、可监控、可回滚。</div>
  </div>
</div>

---

# 4.19 AI 写论文/报告的多步工作流怎么拆？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（6 步 workflow，不是放任 Agent 自由发挥）：</b>
    <div style="margin:6px 0 0;">① <b>意图识别</b>——先定输出格式、篇幅、语气、引用要求；② <b>规划</b>——规划目标输出结构、拆子任务、多 Agent 分工各负责一节；③ <b>资料召回</b>——文献库/知识库/内部资料拿候选（向量 + 关键词 + rerank）；④ <b>证据筛选</b>——去重、时间过滤、来源权重、可信度评分；⑤ <b>内容生成</b>——按目标结构输出正文，每段绑定证据；⑥ <b>后置校验</b>——检查引用真实、结论不越界、格式符合，低置信度标出给用户确认——必须多步 + 反思 + Human review。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">LangGraph 价值：</b>状态机确保每步可回滚、可审计；多 Agent 协作胜过单 Agent；checkpointer 自动保存支持事后追溯。</div>
  </div>
</div>

---

# 4.20 怎么降低「假引用」和幻觉？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（四层系统控制，不依赖 prompt 小技巧）：</b>
    <div style="margin:6px 0 0;">① <b>数据源控制</b>——只允许白名单源进入关键链路（PubMed、官方指南、临床试验注册库、专利库）；② <b>生成前约束</b>——先抽证据再生成，要求引用 ID 先被检索系统验证；③ <b>生成后校验</b>——检查引用是否存在、标题是否匹配、结论能否被原文支撑；④ <b>结果展示约束</b>——把「事实」和「模型推断」分开展示，不确定内容明确标注。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">医疗场景增强：</b>多 Agent 协作验证——Writer 生成后 Critic Agent 模拟「同行评审」（Cross-check 文献、GRADE 证据等级）；RAG（实时抓 PubMed）+ Tool Calling（调用真实数据库）+ Reflection Loop（让 Agent 反思 3 次）；目标幻觉率 &lt;1%，Human + Automated Eval 双重把关。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">本质：</b>降低幻觉是系统设计问题，不是 prompt 小技巧。</div>
  </div>
</div>

---

# 4.21 Workflow 和 Agent 怎么选？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">从<b>任务的稳定性和开放度</b>选：任务路径相对固定（文献检索、摘要生成、引用整理、格式输出）→ 优先 workflow（更稳定、可控、好评估）；任务开放、目标不明确、需要系统自己拆步骤决定调哪些工具 → Agent。不会把 Agent 当默认答案——生产环境经常是 <b>workflow 为主、Agent 为辅</b>，用 Agent 补 workflow 覆盖不到的部分。</div>
  </div>
</div>

---

# 4.22 用 Python 搭 AI 服务后端？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">优先用 <b>FastAPI</b> 搭服务层（适合 AI 应用接口、开发效率高、方便异步）；数据层接 MySQL/Postgres，缓存和会话状态用 Redis；长任务（检索、生成报告、处理文件）引入<b>异步任务机制</b>（任务队列/后台任务）；模型调用层单独封装——统一请求、重试、日志、限流和 fallback；流式输出用 SSE 或 WebSocket；关注可观测性（日志、trace、错误监控、token 成本统计）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分层与契约：</b>API 层只处理入参校验和响应协议；Service 层负责业务编排和规则决策；Repository/Model 层只管数据读写；Workflow 层专注状态转移。层间统一用 DTO 和明确错误码——替换 LLM、切换向量库或升级存储只影响局部层。</div>
  </div>
</div>

---

# 4.23 ToB 和 ToC 的差异？技术上有什么变化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b>ToB</b> 更看重专业能力、准确性、定制化和交付深度——功能不一定花哨，但要稳定、可信、能融入客户流程；<b>ToC</b> 更看重体验、易用性、响应速度、留存和增长。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">ToB 往 ToC 的技术变化：</b>① 交互体验更轻更顺滑；② 系统并发和成本控制更敏感；③ 用户路径更短，不能太依赖复杂配置；④ 评估指标从交付成功扩展到留存、活跃和转化——技术架构要兼顾快速试错和规模化能力。</div>
  </div>
</div>

---

# 4.24 为什么选 LangGraph 而不是 CrewAI/AutoGen？合规怎么保证？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">为什么 LangGraph：</b>状态机（State Machine）每步状态持久化到数据库，支持可回滚、可审计——医疗场景必须满足 HIPAA/NMPA 审计要求；CrewAI 适合快速原型但状态管理弱；LangGraph 支持<b>循环边（cycles）+ Human-in-the-Loop</b>，匹配 Critic/Verifier/Auditor 步骤。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">Agent 失败处理：</b>conditional edges + retry 机制（指数退避）；Gatekeeper Agent 先做风险评估，失败自动路由到 HITL 节点并记录完整日志（Prompt + Output + Error）；检索不到最新指南时标记「证据不足」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">合规（HIPAA/GDPR/NMPA）：</b>Tokenization + 最小化原则——Gatekeeper 在入口把患者姓名/MRN 脱敏成 Token，只有必要 Agent 能看到原始数据，所有 Agent 运行在私有化/零保留（Zero-Retention）环境；每步输出带 audit trail（checkpointer 自动保存）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">多模态扩展与迭代：</b>输入含医学影像时在第 3 步增加 Vision Agent（CheXagent 类模型）输出结构化影像报告合并到主流程；每周收集医生反馈触发 RAG 更新 + Prompt 优化，用 LangSmith 监控每 Agent 的 Latency & Success Rate 自动告警。</div>
  </div>
</div>

---

# 4.25 针对 Excel 表格，RAG 召回质量不好怎么改进？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">① 不要按纯文本切块，做<b>表格结构化切块</b>——sheet → 表 → 行 → 单元格；② 每行<b>补齐表头语义</b>（把行序列化成「日期=xxx, 客户=xxx, 金额=xxx」）；③ 做<b>混合检索</b>——关键词（BM25）+ 向量 + 元数据过滤（sheet、字段、日期）；④ TopK 后加 <b>rerank</b>，减少「看起来相关但答非所问」的片段。</div>
  </div>
</div>

---

# 4.26 单元格字段内容超长超过切分长度怎么处理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">① 对「单元格内部」再分段（句子/段落级），不是硬截断；② 建<b>两级索引</b>——行级摘要索引 + 单元格明细索引；③ 检索先命中摘要，再<b>回源拉明细 chunk</b>；④ 超长字段可存「摘要向量 + 原文指针」，避免一次塞满上下文。</div>
  </div>
</div>

---

# 4.27 向量增量更新后排序变化怎么解？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">通常不用每次全量重建——向量检索排序是<b>查询时实时算相似度</b>，不是全库预排序。实践上用「<b>Base 索引 + Delta 增量索引</b>」，查询时合并结果；只在低峰期做周期性 merge/compact 提升检索效率；只有大规模分布变化时才全量重训或重建。</div>
  </div>
</div>

---

# 4.28 文档时间信息不准（需解析提取）怎么解？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">① 建「<b>业务时间元数据</b>」字段，不只依赖文件更新时间；② 只对<b>新增和变更文档</b>做抽取，历史库按热度异步回填；③ 查询时先用日期索引粗筛，再对 TopN 候选做深度解析；④ 抽取结果持久化并缓存，避免重复算。</div>
  </div>
</div>

---

# 4.29 新字段搜索需要重建索引吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">一般<b>不需要重建向量索引</b>——把检索拆成两层：① <b>语义层（向量索引）</b>——负责「内容相关性」，尽量稳定，不因「按日期/按作者过滤」而变；② <b>结构层（元数据索引）</b>——负责「字段过滤和排序」，新字段上线时扩 schema + 回填该字段即可。查询流程：先按元数据筛候选（date=xxx、author=yyy），再在候选集做向量召回 + 重排——既快又准。</div>
  </div>
</div>

---

# 4.30 如果做一个 Agent 创作助手，工作流怎么编排、状态怎么保存、怎么观测？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">工作流编排：</b>用推荐图或状态机——意图识别 → 任务规划 → 分段执行（过程中可检索编排）→ 任务审校核对 → 不通过则根据审校建议再修改 → 人工审核节点 → 任务发布。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">状态保存（记忆）：</b>短期记忆——一次上下文会话所需信息，存内存或 Redis；长期记忆——用户特征、行为方式、专业知识，存向量库或关系型数据库；日志行为类参考龙虾存本地 MD 文件。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">观测：</b>最小三件套 Log + Metric + Trace；具体指标——系统稳定性（延迟 P95/P99、CPU）、成本（token 监控）、执行步数、业务成功率/超时率/失败错误码分布、Agent 鲁棒性（工具调用失败时表现）、主动性（人工介入情况、推理过程是否符合逻辑、执行是否严格遵守计划）。</div>
  </div>
</div>

---

# 4.31 如何设计一个抢红包系统？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">需求：</b>规则同微信红包、可设置红包权限、考虑数据量和并发。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">架构：</b>Gateway → 红包服务（无状态）→ Redis Cluster（高并发原子扣减）→ MQ → 账务服务 → MySQL（主存）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分工：</b>Redis 负责「抢红包瞬时并发」（原子扣减）；MQ 解耦削峰避免直接打库；MySQL 负责「最终账务与审计」。</div>
  </div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
