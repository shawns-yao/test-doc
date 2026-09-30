# AI 与数据

## 一、Python 与机器学习

# 1.1 Python 工作中常用哪些包？分别用于什么场景？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:grid;grid-template-columns:120px 1fr;column-gap:12px;row-gap:6px;margin:6px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">数据处理</div>
      <div><code>pandas</code>、<code>numpy</code>。</div>
      <div style="color:#3A5FBF;font-weight:600;">可视化</div>
      <div><code>matplotlib</code>、<code>seaborn</code>。</div>
      <div style="color:#3A5FBF;font-weight:600;">机器学习</div>
      <div><code>scikit-learn</code>；深度学习按项目用 PyTorch。</div>
      <div style="color:#3A5FBF;font-weight:600;">HTTP/服务</div>
      <div><code>requests</code>、<code>httpx</code>、FastAPI。</div>
      <div style="color:#3A5FBF;font-weight:600;">校验/配置</div>
      <div><code>pydantic</code>、<code>yaml</code>。</div>
    </div>
    <div style="margin:8px 0 0;">选择包时会考虑<b>生态、性能、团队维护成本和部署环境</b>，不会为了使用库而引入不必要的依赖。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>pandas 和 numpy 的区别（表格 vs 数组计算）；FastAPI 和 Flask 的区别（异步 + 类型校验）；为什么不用 requests 而用 httpx（异步支持）。</div>
  </div>
</div>

---

# 1.2 Python 是动态类型语言吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>Python 是<b>动态类型</b>语言——变量本身不声明固定类型，类型信息绑定在运行时对象上。同一个变量可以先后指向不同类型的对象（<code>x = 1</code> 后再 <code>x = "1"</code>，是重新绑定）；函数参数可接收不同类型对象，能否执行取决于运行时对象是否支持相应操作。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">动态类型 ≠ 弱类型：</b>Python 是<b>强类型</b>语言——<code>1 + "1"</code> 不会隐式拼接，而是抛 <code>TypeError</code>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程实践：</b>类型注解、<code>mypy</code>、IDE 检查和单元测试可提前发现类型问题——但注解主要用于<b>静态分析</b>，不会把 Python 变成编译期强制类型。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>开发灵活、迭代快，代价是部分错误推迟到运行时——大型项目用类型注解、接口约束、测试和代码评审补足风险。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>动态类型和鸭子类型的区别（运行时行为 vs 接口约定）；mypy 是运行时生效吗（不是，静态分析）；和 Java 的编译期检查差异。</div>
  </div>
</div>

---

# 1.3 机器学习了解哪些算法？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">监督学习</b>
        <div style="margin:4px 0 0;">线性回归、逻辑回归、决策树、随机森林、GBDT、XGBoost、SVM。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">无监督学习</b>
        <div style="margin:4px 0 0;">K-Means、层次聚类、PCA、异常检测。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">深度学习</b>
        <div style="margin:4px 0 0;">CNN、RNN、Transformer 的基本思想。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">表述方式：</b>从<b>问题类型、损失函数、训练方式、过拟合处理、评估指标和适用场景</b>说明，不只罗列算法名称。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>决策树和随机森林的关系（Bagging 集成）；GBDT 和 XGBoost 的区别（二阶导数 + 正则）；过拟合怎么处理（正则/交叉验证/早停）。</div>
  </div>
</div>

---

# 1.4 数理统计和数据结构掌握到什么水平？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">数理统计</b>
        <div style="margin:4px 0 0;">概率分布、期望方差、条件概率、贝叶斯公式、抽样估计、假设检验、置信区间、相关回归——能够理解指标波动和实验结果。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">数据结构与算法</b>
        <div style="margin:4px 0 0;">数组、链表、栈、队列、哈希表、树、堆、图和常见排序查找——能分析时间/空间复杂度，用 Java 或 Python 完成常见题目。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">诚实边界：</b>不熟的高级算法明确边界并说明学习计划。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>置信区间怎么解释（区间估计而非点估计）；假设检验 p 值含义；哈希冲突怎么解决（链地址/开放寻址）。</div>
  </div>
</div>

## 二、学习与发展

---

# 2.1 你的学习渠道有哪些？如何保障学习效率？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">渠道：</b>官方文档、源码和 RFC 等一手资料；课程和书籍；技术社区中的真实案例；项目中的问题复盘。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">方法：</b>先带着<b>具体问题</b>学习 → 用小实验或代码验证 → 沉淀成可复用笔记；每周按<b>“输入、实践、输出、复盘”</b>检查完成情况，用间隔复习和面试口述检验是否真正掌握。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>怎么证明学透了（能口述 + 能写代码）；遇到看不懂的文档怎么办（拆小目标 + 找最小可运行例子）。</div>
  </div>
</div>

## 四、Agent 原理与编排

---

# 4.1 Agent 的定义与核心组件？软件 Agent 与具身 Agent 有何不同？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">定义与组件：</b>Agent 是「以 LLM 为决策中枢、可在环境中闭环执行任务的系统」，通常由四层构成：<b>规划层、记忆层、工具层和执行控制层</b>。和普通对话模型相比，关键差异是它能<b>持续感知、决策、行动并根据反馈修正</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">软件 Agent vs 具身 Agent：</b>从纯粹软件环境（调用 API、读写文件）进入真实/模拟物理环境（机器人、游戏）即<b>具身智能体（Embodied Agent）</b>。核心区别是「感知与行动的不确定性」——软件 Agent 处理结构化、可控接口；具身 Agent 面对<b>高维噪声输入、部分可观测状态和连续动作误差</b>，需要实时闭环。更关键的是安全后果：软件错误多数可回滚，<b>物理动作可能不可逆</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">评估标准变化：</b>从「答对」升级为「<b>安全、稳定、可恢复</b>」，系统设计重点从回答质量转为安全与工程兜底。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>真正拉开差距的是工程控制面——状态管理、异常恢复和可观测性。</div>
</div>

---

# 4.2 ReAct 是什么？Agent 的规划能力怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">ReAct：</b>是 <b>Thought-Action-Observation 循环</b>——先思考、再调用工具、再根据观察结果迭代。比单次 CoT 更适合信息不完整任务，因为能<b>边查边改</b>；代价是链路更长，线上要控制步数、超时和成本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">规划方法（线性到搜索）：</b>① <b>CoT</b> 单路径分步推理，成本低但容错弱；② <b>ToT</b> 树状思考引入多分支探索和回溯，成功率更高但算力开销大；③ <b>GoT</b> 图结构允许分支合并和循环优化，适合复杂依赖问题。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程实践：</b>把「规划」和「执行」<b>解耦</b>——Planner 负责拆解、Executor 负责落地（多角色 Agent 划分），减少模型一步到位失败。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>规划能力本质是用更多搜索换更高成功率。</div>
</div>

---

# 4.3 AI Agent 的基本实现路径是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（控制面 + 数据面）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">控制面：</b>状态机/图编排——定义节点、路由条件、重试策略和结束条件；<b>数据面</b>是每个节点具体怎么跑。<b>大模型 API</b> 负责推理与决策，<b>RAG</b> 提供外部知识，<b>MCP</b> 标准化工具调用。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">典型回路：</b>模型判断当前任务是否需要知识检索或调用工具 → 需要则先走 RAG 拿上下文，再走 MCP 调用外部能力（搜索、数据库查询、系统操作）→ 工具结果回填状态 → 再交给模型继续推理 → 直到满足结束条件。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>① <b>节点输入输出契约清晰</b>——每个节点只读必要状态、只写自己产物，避免上下文污染；② 线上加<b>步数上限、超时、幂等键和降级策略</b>；③ 打<b>全链路 trace</b>，区分是 Prompt 问题、检索问题还是工具问题。</div>
  </div>
</div>

---

# 4.4 Agent 当前的局限是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">短板主要是「<b>稳定性、可控性、成本</b>」三件事：① <b>稳定性</b>——任务链路长、依赖外部工具多，任何一环抖动都会影响成功率；② <b>可控性</b>——模型是概率系统，复杂场景下会出现误判、过度调用工具甚至越权风险；③ <b>成本</b>——多轮推理 + 检索 + 工具调用叠加，token 和外部 API 成本上升很快。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">为什么没爆款：</b>很多场景还没跨过「可用到可靠」的工程门槛。Agent 更适合<b>流程明确、容错可设计、收益可量化</b>的场景（内容生产、客服分流、内部效率工具）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">做亮点的关键：</b>不是堆模型，而是<b>做强编排、评测和治理</b>，把失败率和成本打下来。</div>
  </div>
</div>

---

# 4.5 如果设计一个 Agent 产品，你会从什么方向切入？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">优先选「<b>高频、可量化、可回滚</b>」的业务场景（如企业知识问答 + 任务执行助手），设计思路是<b>先确定目标指标，再反推功能</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">四步设计：</b>① 定义<b>北极星指标</b>——任务完成率、人工介入率、平均处理时长、单位任务成本；② <b>能力拆分</b>——意图识别、检索增强、工具执行、结果审校、异常兜底；③ <b>安全与治理</b>——最小权限、敏感操作二次确认、全链路审计；④ <b>上线策略</b>——小流量灰度、在线 AB、失败快速回滚。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">经验：</b>先做「<b>半自动</b>」而不是「全自动」——把高风险动作放人工审批，先把稳定性和信任建立起来，再逐步放开自治程度。</div>
  </div>
</div>

---

# 4.6 什么时候用单 Agent、多 Agent、Autonomous Agent？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">选型：</b>单 Agent 适合<b>流程短、目标清晰、风险可控</b>场景；多 Agent 适合<b>复杂任务拆解与并行协作</b>（Planner/Executor/Critic 分工）；Autonomous Agent 强调<b>长链路自主闭环</b>，但需要更强安全边界。把 Agent 想象成组织中的员工，要看具体做的事情的大小和复杂度。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">多 Agent 特点：</b>优势是上限高，但复杂度集中在<b>通信协议、状态一致性和路由收敛</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">Autonomous vs Multi-Agent：</b>Autonomous 强调单体自主闭环，适合流程清晰、低风险、成本敏感任务；Multi-Agent 强调多角色分工协作，适合复杂任务拆解、并行处理和交叉审校。工程上最大差异在<b>编排复杂度和故障域</b>：单体方案简单但一旦决策偏差影响全链路；多 Agent 治理更复杂，但可局部隔离和重试。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">降级时机：</b>任务风险高、动作不可逆或系统指标恶化时，把 Autonomous Agent 降级为半自动——保留检索和生成能力，把最终执行权交给人工审批，先保正确性和可追责。</div>
  </div>
</div>

---

# 4.7 构建复杂 Agent 最主要的挑战是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">难点不止模型能力，而是<b>端到端系统可靠性</b>，通常集中在四类：① <b>规划鲁棒性不足</b>——卡在重复的思考-行动循环、对工具失败没有备用方案、过早认为任务完成；② <b>评估难复现</b>——开放式任务没有唯一正确答案，指标难定义、环境不可复现、人工评估成本高；③ <b>成本时延高</b>——复杂任务数十上百次 LLM 调用，API 费用和延迟难以规模化；④ <b>安全可控不足</b>——权限管理困难、提示词注入、行为不可预测。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">面试表述：</b>「先做可控再做聪明」——加步数上限、权限边界、故障回退和全链路日志，优先保底稳定性。</div>
  </div>
</div>

---

# 4.8 多智能体系统相比单 Agent 有什么优势？引入哪些新复杂性？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">多智能体系统就是把一个复杂目标拆成多个角色协同完成（规划、执行、审校分工），提高复杂任务成功率。优势：① <b>专业化</b>——每个 Agent 设定不同角色和专长，可基于专门知识和工具微调；② <b>并行化</b>——复杂任务分解后分配给不同 Agent 同时处理，缩短总时间。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">新增复杂性：</b>① <b>通信协议</b>——Agent 之间如何有效沟通，需要标准化的消息格式确保相互理解意图、状态和知识；② <b>任务协调、状态一致性</b>——谁负责、谁失败、谁回滚都要定义清楚，没有明确编排规则时多 Agent 反而比单 Agent 更不稳定。常用框架：LangGraph、OpenAI Agents SDK、AutoGen、CrewAI。</div>
  </div>
</div>

---

# 4.9 A2A 框架与普通 Agent 框架的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">A2A（Agent-to-Agent）最关键在「<b>协议层</b>」——它是一个通讯协议（类似 HTTP 那样的底层协议），关注<b>多个异构 Agent 之间的通信和协作</b>，试图定义一套通用的标准、协议和语言，让不同开发者、不同技术栈、不同目标构建的 Agent 能相互发现、理解和协作。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">一句话总结：</b>前者解决「群体协作标准」，后者解决「个体执行能力」；多团队、多技术栈协作时，A2A 的价值明显放大。</div>
  </div>
</div>

---

# 4.10 Multi-Agent 实际项目怎么设计？LangGraph 里怎么编排更稳？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（角色分工 + 状态编排 + 失败恢复）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 角色分工：</b>典型角色是 Planner（规划）、Executor（执行）、Critic（审校）、Tool Agent（外部能力）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 状态编排：</b>状态层用图或状态机管理节点流转；每个 Agent 有明确<b>输入输出契约</b>，避免互相污染上下文。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 失败恢复：</b>每个节点配超时、重试、步数上限、防循环；高风险任务接<b>人工接管（human-in-the-loop）</b>；上线后通过 trace 做全链路观测，重点看任务成功率、时延、成本和重试率。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">LangGraph 编排（主控图 + 专家节点）：</b>Planner 只拆解任务、Dispatcher 负责路由、Executor 只执行、Critic 只审校。① 全局状态只放<b>最小字段</b>，大文本走引用；② 图上配置<b>最大步数、重试上限、超时和预算</b>；③ 写操作加<b>幂等键</b>并做 checkpoint——失败也能从节点恢复，不重复副作用；④ 上线后按任务成功率、改写率、时延和成本做节点级观测。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>让 Agent 像微服务一样可组合、可回放、可治理；多 Agent 最容易坏在状态编排层（Orchestration），失败大多发生在「交接」。</div>
</div>

---

# 4.11 Chain、Agent、Workflow 三者怎么区分？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b>Chain</b> 是固定顺序管道，适合确定性流程；<b>Agent</b> 是模型驱动决策，适合开放任务；<b>Workflow</b>（特别是 LangGraph）是显式状态机，适合把 Agent 决策放进可控节点里。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">LangGraph 的价值：</b>把「多 Agent 协作」从隐式流程变成<b>显式状态机</b>——明确状态与节点边界（减少上下文串台）、条件路由可控（避免无意回环）、checkpointer 支持中断后续跑、关键节点可 interrupt 人工审核、每步输入输出可追踪。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">框架不替你解决的坑：</b>I/O 契约没定义清楚、工具调用非幂等（重试导致重复副作用）、没有超时/最大步数/重试上限、状态字段设计混乱。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">限制 Agent 自由度：</b>工具白名单和最小权限放行 + 限制最大步数、最大调用次数、超时和成本 + 高风险动作人工审批 + 全链路监控告警——可追溯、可回滚、可治理。</div>
  </div>
</div>

---

# 4.12 Agent 工具调用的完整业务流程？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（判定-执行-回填-收敛）：</b>
    <div style="margin:6px 0 0;">① <b>判定</b>——模型基于当前状态判定是否需要工具，产出结构化调用计划；② <b>执行</b>——执行层做<b>参数校验、权限校验和幂等校验</b>，通过后调用工具；③ <b>回填</b>——执行结果统一封装为成功/失败事件回填到状态；④ <b>收敛</b>——模型吸收结果决定下一步：继续调用、换工具、降级还是结束。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>每一步都要可观测——调用前后都打 trace，记录入参摘要、耗时、错误码和重试次数。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">线上稳定三件套：</b>超时与重试策略、熔断与降级策略、人工接管开关——外部依赖抖动时先止损再恢复，而不是把失败直接暴露给用户。</div>
  </div>
</div>

---

# 4.13 节点失败或意图识别错误如何处理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">节点失败：</b>每个节点都有超时 + 最大次数，超过就走<b>失败分支/降级路径/人工介入节点</b>；做<b>错误分级</b>——可重试（超时、429、临时网络）和不可重试（参数错、权限错、业务校验失败）；用 <b>checkpoint + 幂等键</b> 保证断点恢复时不重复副作用（不重复下单/发消息）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">意图识别错误：</b>意图识别后先走一层<b>校验层/风险控制层</b>——高风险意图必须二次确认；执行过程中做<b>一致性校验</b>，若检索/工具返回与意图不一致，触发路由和反思。</div>
  </div>
</div>

---

# 4.14 OpenClaw 这类项目对 Agent 工作流的启发？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">① <b>Gateway 统一网关入口</b>——跨渠道消息先标准化再路由，避免每个渠道各写一套逻辑；② <b>会话隔离不串台</b>——sessionKey、dmScope、每会话串行执行和队列并发上限，核心是防串台、防并发写冲突；③ <b>三层记忆系统</b>——「文件化 + 可检索」：MEMORY.md 做长期记忆、memory/*.md 做日记忆、配 memory_search，可追溯可审计；④ <b>可托管运行</b>——心跳机制保证主动思考工作，定时任务块机制确保周期任务统一调度。</div>
  </div>
</div>

---

# 4.15 多步骤研究型 Agent 的核心链路怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">设计成「<b>规划 → 并行采集 → 证据归并 → 结论校验 → 交付输出</b>」五段：先把问题拆成子任务，再并行拉取多源信息，做去重和可信度排序，最后生成可追溯结论。每段都有<b>输入输出契约和失败恢复点</b>，避免长链路一步错全盘错。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">与普通 RAG 的差异：</b>普通 RAG 是单跳检索增强（回答一个问题）；Wide Research 是<b>任务型流程系统</b>——多轮规划、并行采集、阶段归纳、动态重规划。前者强调召回与生成质量，后者强调<b>任务成功率、过程可解释性、执行可靠性与恢复能力</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">为什么复杂任务依赖编排而非单次模型能力：</b>复杂任务失败通常不在「某一句答错」，而在流程失控——顺序不对、依赖缺失、证据冲突没处理。编排能力决定任务能否按阶段收敛、能否回滚重试、能否追踪责任。</div>
  </div>
</div>

---

# 4.16 Browser Agent 的关键失败点有哪些？如何做重试和回滚？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#B26E00;">高频失败点：</b>DOM 变化、元素不可见、会话过期、跳转异常和限流。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">处理做法：</b>「<b>动作前置校验 + 失败分类重试</b>」——可重试错误做指数退避，不可重试立即降级；关键写操作加幂等键，失败后按<b>检查点回滚</b>到上一步，不从头盲跑。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">动态页面/登录态/反爬：</b>动态页面用<b>显式等待和稳定锚点</b>，不依赖脆弱选择器；登录态通过安全会话托管和失效检测自动刷新；遇到反爬或验证码时<b>切人工节点</b>，不强行绕过。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">为什么可观测性特别重要：</b>浏览器任务是长链路状态机，失败常发生在中间节点——没有步骤级日志、截图、DOM 快照和耗时指标，无法判断是页面变化、网络抖动还是策略错误。</div>
  </div>
</div>

---

# 4.17 团队协作场景下 Agent 如何做会话隔离与权限控制？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">会话隔离：</b>上下文主键设计为「<b>workspace/channel/thread/user</b>」——线程内共享任务状态，线程间严格隔离；权限采用<b>角色映射到动作级策略</b>（谁可触发外部调用、谁可审批敏感动作）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">群聊上下文污染治理：</b>① 限制上下文来源——默认只读当前线程和被 @ 消息；② 消息分级——低价值闲聊不进入任务记忆；③ 关键事实落<b>结构化状态</b>而非自然语言堆叠；④ 冲突时以最近确认状态为准，保留证据引用。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">为什么审计要求更高：</b>协作平台涉及多人协同和组织责任，任何自动动作都可能影响业务结果与合规边界——必须回答清楚「谁发起、谁审批、执行了什么、影响了什么」，审计链完整才能可追责、可证明。</div>
  </div>
</div>

## 五、上下文工程与记忆

---

# 5.1 Agent 的短期记忆、长期记忆和状态怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">短期记忆：</b>承载当前任务上下文、中间结果和工具返回，保证任务连贯——生命周期短，通常跟会话/任务绑定，强调时效和低延迟，适合放缓存或状态存储（快速读写 + 过期清理）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">长期记忆：</b>承载跨会话偏好、稳定事实和历史经验——生命周期长，强调可检索、可更新和一致性，更像知识资产，用向量库 + 结构化库组合（RAG 范式检索），配版本和来源管理，也可用 SQL 或知识图谱存结构化事实。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">State 设计原则：</b>① <b>最小化</b>——只放决策必需字段（任务目标、中间结果、证据、错误码、重试计数、下一步动作）；② <b>分层</b>——业务数据、过程数据、观测数据分开；③ 节点只读必要字段、只写自己负责字段，避免相互污染；④ <b>大对象外部化</b>——长文本/检索结果/工具返回存 DB/向量库，State 放引用 ID。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">记忆写入策略（防噪声）：</b>① <b>分级写入</b>——先入候选区，只有「高频出现 + 高置信 + 可复用」才转正；② <b>写前去重合并</b>——语义相似度 + 关键字段去重，已有就更新；③ <b>生命周期</b>——带来源/时间/版本/TTL，长期不命中自动降权或清理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">长期记忆过期与纠错：</b>① TTL + 最后命中时间 + 重要度，超期归档/清理；② <b>冲突检测与版本化</b>——事实写入先检测冲突，不直接覆盖，保留旧版、标记新版、可回滚；③ <b>验证闭环</b>——关键记忆二次验证（多次出现、可信来源、用户确认），提供用户纠正入口。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">本质区别：</b>短期记忆服务「当前决策」，长期记忆服务「持续个性化与累积学习」。</div>
  </div>
</div>

---

# 5.2 LLM 效果不好时如何分层调优？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（三层法）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">L1 Prompt 调优（先做，成本最低）：</b>① 定义执行协议——角色、边界、输出格式、拒答策略；② 结构化输出——JSON Schema/函数调用；③ 少量高质量 few-shot（正例 + 反例）；④ 上下文工程——按 Gather-Select-Structure-Compress 组织；⑤ 建评测集——50-200 条真实样本，按准确率、格式通过率、幻觉率评估；⑥ 提示词版本化——A/B、灰度、可回滚。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">L2 检索与工具链路（第二层）：</b>① RAG 数据治理（清洗、去重、切块、元数据）；② 检索策略（hybrid + rerank）；③ 查询改写（multi-query、HyDE）；④ 工具编排（路由器决定直接答/检索/调工具，信心阈值 + 兜底）；⑤ 结果校验（schema 校验、失败重试或降级）；⑥ 观测闭环（检索命中率、工具成功率、端到端成功率、时延、成本）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">L3 训练调优（最后做）：</b>① 触发条件——Prompt+RAG 已到瓶颈仍有稳定性/领域表达问题；② 先 SFT（真实任务轨迹数据）；③ 参数高效微调优先 LoRA/QLoRA；④ 再做偏好优化 DPO/RLHF；⑤ 训练数据覆盖高频失败样本、难例、边界例；⑥ 上线策略——影子流量 + 小流量灰度 + 回归评测，避免微调后退化。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">原则：</b>先 Prompt 与上下文工程，再检索/工具链路，再模型微调——先改低成本高收益环节。很多线上问题来自上下文构造和评估缺失，不是模型不够强。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">微调后变差的排查顺序：</b>先回滚模型流量止损（卸载新 LoRA adapter），再按<b>数据（标注噪声/分布漂移/样本冲突）、训练（学习率/epoch/过拟合/LoRA 参数）、评测（离线集不代表线上/口径错误）、系统（RAG/工具链路变了）</b>四层对照排查，最后小流量灰度重放上线。</div>
  </div>
</div>

---

# 5.3 Prompt Engineering 优化策略有哪些？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">把 Prompt 当成<b>预算管理问题</b>来做，核心是「高信号、低冗余」五个手段：① <b>分层提示</b>——稳定规则放 system、任务信息放 user，避免重复拼接大段固定文本；② <b>最小必要上下文</b>——只传当前任务必需信息，历史对话做摘要而非全量回放；③ <b>检索按需注入</b>——RAG 只注入 top-k 片段并设 token 上限；④ <b>结构替代长文本</b>——用 schema、枚举、字段约束替代长篇说明；⑤ <b>监测与 A/B</b>——监控首 token 时延、总 token、成功率和成本，按指标裁剪 Prompt。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">与微调的协同：</b>两者不是替代关系——Prompt 负责一次交互的快速约束行为和流程编排，微调负责长期固化能力与风格稳定；「Prompt 先行、微调收口」。</div>
  </div>
</div>

---

# 5.4 什么是上下文工程？和 Prompt Engineering 的边界？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">定义：</b>上下文窗口不是「越大越好」，核心是<b>上下文工程</b>——如何挑选、组织、压缩、更新上下文。从聊天机器人走向可执行任务的 Agent 时，Prompt Engineering 不够用了，必须升级为 Context Engineering：Agent 的上下文是持续演化的状态系统（工具结果、网页观察、代码反馈、阶段结论、失败重试、长期记忆）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">与 Prompt 的边界：</b>Prompt Engineering 是「单轮输出控制」（角色、格式、约束），Context Engineering 是「多轮状态治理」（信息进入、保留、淘汰和回取）。前者解决「怎么说」，后者解决「基于什么说」，两者必须同时做。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">四件事：</b>上下文<b>采集、选择、压缩、治理</b>（预算/监控/回滚）。「状态优先于文本」——模型每一步读到可执行状态，而不是冗长叙述。</div>
  </div>
</div>

---

# 5.5 如何突破上下文窗口限制？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心：</b>真正的瓶颈不是模型一次能读多少 token，而是长链路任务中信息不断累积、注意力被噪声占据——出现早期信息遗忘、证据链断裂、结论自相矛盾。解决思路是「超越窗口」：把上下文从单个大文本改造成<b>分层、分阶段、可检索的外部系统</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">做法：</b>① <b>任务分解</b>——拆成子目标，每阶段只携带当前决策必需信息；② <b>外部记忆</b>——历史过程通过摘要和结构化记录沉淀，需要时按证据索引回取，不全量回灌；③ <b>阶段摘要</b>——每阶段产出结构化结论、证据引用和未决问题清单；④ <b>动态重规划</b>——新证据出现时更新任务树和优先级。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">避免前后矛盾：</b>每阶段先读「摘要状态」再行动；做一致性检查（关键实体、数字、时间线）和冲突告警，必要时触发回溯检索。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">一句话：</b>窗口是资源约束，上下文工程是资源调度——Agent 的稳定性来自工程设计，而不是模型参数单点突破。</div>
  </div>
</div>

---

# 5.6 RAG 和 Agent 的 Prompt 有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">目标差异：</b>RAG Prompt 核心是「<b>基于给定证据回答</b>」——强调引用、边界和事实一致性；Agent Prompt 核心是「<b>驱动任务执行</b>」——强调计划、工具选择、状态推进和失败处理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">结构（六段式）：</b>角色、任务、上下文、约束、示例、输出格式。RAG 里<b>上下文段最重</b>——「仅依据检索片段回答，缺失就明确说不知道」；Agent 里<b>工具规范最重</b>——何时调用工具、入参格式、异常分支和停止条件。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">与模型交流两点：</b>① 结构化输入输出（JSON Schema）减少歧义；② 把策略写成可观测规则（「最多重试 2 次，超过则降级」）——把提示词从文案变成可执行协议，便于调试和复盘。</div>
  </div>
</div>

---

# 5.7 超长文本（如整本代码库）如何治理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">核心思路：不把整库塞进上下文，而是「<b>索引化 + 检索化 + 压缩化 + 增量化</b>」。除了分块，还可：</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">① 代码图索引：</b>用 AST、依赖图、调用图做「代码图检索」，比纯文本更准；② <b>混合检索</b>——BM25（关键词）+ 向量（语义）+ 图（关系）联合召回；③ <b>重排与预算控制</b>——召回后 rerank 只留最相关片段，按 token 预算动态调 TopK；④ <b>层级摘要</b>——文件摘要、模块摘要、仓库摘要三级压缩，先读摘要再按需下钻；⑤ <b>增量更新</b>——只重建变更文件（基于 git diff），避免每次全量 embedding。</div>
  </div>
</div>

## 六、模型调优与微调

---

# 6.1 LLM 基本原理与后训练体系？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（三段讲）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">原理：</b>LLM 是基于 Transformer 架构的大规模语言模型——单 encoder 是 BERT（理解/相关性/意图识别），单 decoder 是大语言生成模型，核心能力是根据已有上下文<b>预测下一个 token</b>。海量语料预训练后学到知识、模式和推理能力。同一个模型在不同系统提示词下表现差异大——LLM 不是执行固定代码，而是「在当前上下文条件下做下一个 token 的概率决策」，系统提示词本身就是最强条件之一。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">训练三阶段：</b>① <b>预训练</b>——大规模文本 next-token prediction，学语言规律和世界知识；② <b>指令微调（SFT）</b>——让模型学会遵循指令和对话格式；③ <b>后训练（RLHF/DPO）</b>——让输出符合人类偏好和安全要求。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">RLHF 经典链路：</b>SFT → <b>奖励建模（RM）</b>（学习人类偏好，评估回答质量）→ <b>强化学习（PPO）</b>优化策略。核心目标是把「可读」提升为「符合人类偏好和安全要求」。RLAIF 用强大 AI 模型（GPT-4）替代人工标注偏好数据，效果接近 RLHF 且成本大幅降低。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">DPO：</b>去掉奖励模型和在线 RL 环节，训练更稳、成本更低、迭代更快；但需要强探索和长时序优化的任务里 PPO 仍有价值。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">局限与配合：</b>会幻觉、可控性有限、成本和延迟高——上线要配合 Prompt 约束、RAG、工具调用、评估和监控。</div>
  </div>
</div>

---

# 6.2 LoRA/QLoRA 的原理、流程和上线方法？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">原理：</b>LoRA 不改大模型原参数，只学一个「<b>低秩增量</b>」——在冻结基座模型参数前提下训练低秩适配器；QLoRA 进一步把基座模型做 <b>4-bit 量化</b>，显著降低显存和训练成本。LoRA 做的是模型输出行为和风格稳定性、一致性。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">落地流程：</b>① 先定义<b>验收指标</b>；② 高质量数据构建，尤其补齐线上失败样本；③ 训练阶段重点调 <b>target modules 和 rank</b> 等关键参数；④ 评估不只看任务指标，还看安全和回归；⑤ 上线采用灰度 A/B 和可回滚策略。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">注意：</b>微调收益主要由<b>数据质量和评估体系</b>决定——这两块不扎实，微调容易出现局部提升但整体退化；数据量没有绝对值，常见从几千到几万条高质量指令数据起步。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">RAG 和 LoRA 分工：</b>RAG 解决「事实与时效」，LoRA 解决「行为与风格」；通常先做 RAG，再用 LoRA 补稳定性和格式一致性。微调不是万能药，若问题本质是检索错召回，先优化 RAG 往往更有效。</div>
  </div>
</div>

---

# 6.3 Agent 微调数据集怎么构建？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">Agent 微调的重点不是「最终答案」，而是「<b>决策轨迹</b>」——教会模型如何更好地「思考」和「使用工具」，本质是<b>行为克隆（Behavioral Cloning）</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">数据构造：</b>让强教师模型生成高质量 Thought/Action/Observation 决策轨迹，再过滤失败样本，结合人工修正构造训练集。数据来源一般包括：<b>合成任务、真实线上失败案例和专家示范</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实践：</b>先做行为克隆打基础，再用偏好数据做策略优化，提升稳定性。</div>
  </div>
</div>

---

# 6.4 LoRA 在 Agent 微调中的应用场景？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">LoRA 适合「行为和表达对齐」，不适合「实时知识更新」。Agent 微调常见场景：① <b>工具调用格式稳定化</b>——让模型更稳定地产生 JSON/function call；② <b>角色风格一致化</b>——客服、投研、写作助手的语气和输出结构；③ <b>领域术语适配</b>——金融、法律、医疗等垂类表达更专业；④ <b>多租户个性化</b>——每个客户挂不同 LoRA adapter，而不是训多套大模型。</div>
  </div>
</div>

---

# 6.5 最近关注的前沿技术或论文？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（以 LinearRAG 为例）：</b>
    <div style="margin:6px 0 0;">印象最深的一篇是 LinearRAG——它指出现在主流的「先建图关系库做多步推理，再进行向量检索」的方式比较重：构建重型准确的图数据库和复杂推理本身就很复杂。论文主张<b>在业务里优先做轻量结构化索引 + 两阶段检索加重排</b>，先拿到稳定收益——类似广告召回中的<b>两段式检索</b>：先用用户问题找到相关实体，再由实体去找信息。</div>
  </div>
</div>

---

# 6.6 BERT 与 LLM 的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">BERT 可以理解为「<b>擅长理解文本、不擅长自由生成</b>」的 Transformer 模型：<b>Encoder-only</b>，偏「理解」；主流 LLM（GPT 类）是 <b>Decoder-only</b>，自回归逐词生成，偏「生成 + 推理」。结果上：BERT 常用于打分/判别，LLM 常用于对话、生成、Agent。</div>
  </div>
</div>

---

# 6.7 NLP、TFRecord、TensorFlow 与 Transformer 的关系？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">NLP：</b>不是单一模型，而是一组「让机器处理文本」的能力集合——文本理解（分词、实体识别、意图识别、语义匹配、情感/分类）、文本生成（标题、摘要、问答）、文本检索与排序（相关性打分、重排）、文本质量与安全（去重、错别字、违禁内容识别）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">TFRecord：</b>TensorFlow 常用的训练数据二进制格式——把原始样本清洗、特征化后序列化成 .tfrecord 文件再喂给训练。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">TensorFlow vs Transformer：</b>「框架」与「模型架构」的关系——TensorFlow 是深度学习框架（训练/推理工具链），Transformer 是神经网络架构（模型怎么设计，如自注意力）。前者解决「怎么高效训练和上线」，后者定义模型内部计算方式。</div>
  </div>
</div>

## 七、评测与监控

---

# 7.1 如何设计 LLM/Agent 评估体系？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（离线 + 在线 + 人评三位一体）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">离线：</b>看能力覆盖——事实性、推理、安全、格式；<b>在线：</b>看业务指标——成功率、转化、时延、成本、稳定性；<b>人评：</b>校准自动评估偏差。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">Agent 还要看过程指标：</b>步数、工具成功率、重试率、中断率。上线后通过持续监控与坏例回流形成闭环，避免模型漂移无人感知。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">四维度评价 Agent：</b>① <b>效果</b>——任务完成率、答案正确率、引用准确率；② <b>效率</b>——首 token 时延、端到端时延、单任务 token 和外部 API 成本；③ <b>稳定性</b>——工具调用成功率、重试率、超时率、回环率；④ <b>安全性</b>——越权调用、敏感操作拦截率、审计完整性。评测采用「离线集 + 线上流量」双轨，最重要的是<b>坏例回流</b>——把失败 case 结构化记录并分类到 Prompt、检索、工具、编排四层。</div>
  </div>
</div>

---

# 7.2 为什么 BLEU/ROUGE 评 LLM 不够？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">指标原理：</b>BLEU 主要统计候选文本与参考答案的 n-gram 精确率并加 brevity penalty；ROUGE 更偏向召回参考答案中的词或片段。它们适合参考答案稳定的翻译、摘要场景，但不能代表开放式问答的真实质量。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">主要局限：</b>① 词面不同但语义正确时得分可能低；② 词面相似但事实错误时得分可能高；③ 无法判断引用是否支持结论、推理步骤是否正确、是否遵守安全策略；④ 对答案长度、模板和语言差异敏感。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">替代组合：</b>使用 Embedding/BERTScore 做语义相似度，使用事实核验、引用准确率和人工 Rubric 检查正确性，再加入安全、拒答、风格和延迟等业务指标。词面指标可以保留作辅助，不应作为唯一上线门槛。</div>
  </div>
</div>

---

# 7.3 常用 LLM 综合基准测试？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">能力覆盖：</b><b>MMLU</b>考察多学科知识与选择题推理，<b>BIG-Bench</b>覆盖大量任务和泛化能力，<b>GSM8K</b>偏小学数学推理，<b>HumanEval</b>评估代码生成并常用 <code>pass@k</code>，还可根据需要加入 TruthfulQA、GPQA、MMMU 等事实、专业和多模态测试。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">使用边界：</b>不同基准的数据污染、题型格式和语言分布会影响排名；单一分数不能代表生产能力，也不能直接推断 Agent 的工具调用、长链路稳定性和业务成本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程做法：</b>公开基准用于横向了解能力，业务黄金集用于纵向版本回归，再补充延迟、Token 成本、失败率和安全测试。固定数据版本、提示词、采样参数和评分脚本，保证结果可复现。</div>
  </div>
</div>

---

# 7.4 什么是 LLM-as-a-Judge？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">定义：</b>LLM-as-a-Judge 是让一个评审模型按照明确 Rubric，对候选回答进行打分、排序或判断胜负。流程通常是准备问题和参考标准 → 生成候选答案 → 脱敏并随机化顺序 → 评审模型输出分数与理由 → 与人工标注抽样校准。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">Rubric 设计：</b>把“好答案”拆成事实正确、任务完成、证据支持、表达清晰、安全合规等维度，规定 0-2 或 1-5 分锚点，并要求评审给出结构化 JSON，避免只返回模糊总分。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">偏见与校准：</b>模型可能偏好更长、更像自己或排在前面的答案，也可能被格式和措辞影响。应做双盲、顺序打乱、位置交换、多评审模型交叉验证，并持续计算与人工标注的一致率；高风险结论必须人工复核。</div>
  </div>
</div>

---

# 7.5 如何评估事实性、推理能力、安全性？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">事实性：</b>准备带标准证据和引用范围的问答集，分别测答案正确率、证据支持率、引用准确率和拒答正确率；加入不存在实体、过期信息和证据冲突样本，检查模型是否承认不知道而不是编造。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">推理能力：</b>用数学、逻辑、代码和多步规划任务评估最终结果，同时检查中间步骤是否满足约束、是否出现循环或跳步。生产评测应关注可验证的中间产物，不把未经验证的思维链文本当作正确证明。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">安全性：</b>覆盖提示注入、越权工具调用、隐私泄露、危险内容和拒答绕过等红队样本。核心指标包括有害请求拦截率、正常请求误伤率、敏感信息泄露率、工具越权率和人工升级率，并按攻击类型分桶。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">评估闭环：</b>离线固定集用于版本回归，线上监控按模型、场景和风险等级采样；任何安全指标下降都要阻断发布或回滚，不能用平均任务成功率抵消高风险缺陷。</div>
  </div>
</div>

---

# 7.6 Agent 评估与 LLM 评估的差异？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">基础 LLM 是单轮「输入-输出」评估；Agent 是<b>多步交互系统</b>，过程影响结果、状态持续变化，复杂度高很多。对 LLM 评估像「产品质量检测」，对 Agent 评估像「路况复杂的真实驾驶测试」——不仅要看是否到达终点，还要看过程中的效率、安全和应对突发。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">过程指标六项：</b>步骤数、总时延、Token/费用、工具调用成功率、重试率、异常恢复时长——很多系统「能做对但代价太高」线上不可用，过程指标直接指导优化方向。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">鲁棒性与自主性：</b>错误处理能力（工具报错能否纠正）、抗干扰能力（噪声/误导信息下成功率下降多少）；人工干预次数越少越自主、行为可解释性、计划遵循程度。</div>
  </div>
</div>

---

# 7.7 Agent 能力基准测试有哪些？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">① <b>WebArena</b>——网页浏览与操作（电商/论坛/写作工具环境，程序化判断终态）；② <b>AgentBench</b>——通用 Agent 综合评估（OS 终端、SQL 数据库、知识图谱、文字冒险等八种环境）；③ <b>GAIA</b>——模拟人类用真实工具完成复杂任务（网页搜索 + 代码 + 文件操作并用）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">环境构建：</b>可控隔离沙箱（Docker 封装浏览器/终端/文件系统），任务以<b>高层次目标</b>给出（不给具体步骤），附带可程序化验证的成功标准——评估规划、工具使用和鲁棒性，而不仅是最终文本质量。</div>
  </div>
</div>

---

# 7.8 人工评估准则与流程怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">先把维度<b>原子化</b>（准确性、完整性、安全性、简洁性），给每档分数明确定义和示例；流程上做<b>盲评、多评审、冲突仲裁和一致性统计</b>（如 Kappa），避免主观漂移。模型对比任务用<b>成对比较</b>比绝对打分更稳定。人工评估不是「凭感觉」，而是标准化实验流程。</div>
  </div>
</div>

---

# 7.9 上线后如何持续监控与评估？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（采集/监控 → 分析 → 迭代闭环）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 采集和监控：</b>记录请求完整交互数据（输入、中间思考、输出、调用工具、延迟、token）；嵌入<b>用户反馈机制</b>（顶踩/打分）；间接指标——输出长度、代码块比例、JSON 格式错误率、拒绝率，过程指标——平均步数、工具调用频率、工具失败率。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 审核与分析：</b>自动化——定期抽样生产流量评估、裁判模型自动打分、与黄金评估集对比；定期人工审计——随机样本、用户反馈坏例、监控异常案例深入分析。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 反馈闭环与迭代：</b>把失败案例和用户不喜欢的案例清洗标注后持续加入评估集和微调数据集，定期再训练；新版本用 AB 测试小流量验证是否优于旧版本。</div>
  </div>
</div>

---

# 7.10 LangChain/LangGraph 上线看哪些指标？成功率下降怎么定位？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（四类指标）：</b>
    <div style="margin:6px 0 0;">① <b>任务成功率</b>（业务结果）；② <b>链路稳定性</b>（错误率/重试率/中断率）；③ <b>性能与成本</b>（p95 时延、token 成本）；④ <b>质量指标</b>（幻觉率、格式通过率、人工接管率）。没有这套指标就无法判断是模型问题还是编排问题。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">成功率下降漏斗定位（编排 → 工具 → 检索 → Prompt）：</b>① <b>图编排层</b>——看 loop_rate、step_count、p95 timeout、retry_rate：步数暴涨/回环增多/超时增多 = 路由或终止条件问题；② <b>工具层</b>——看 success_rate、5xx/429、schema 校验失败率、p95 延迟：调用失败或慢 = 工具可用性/契约问题；③ <b>检索层</b>——看 hit@k/recall@k、空召回率、索引新鲜度：召回质量掉则答案成功率同步下降；④ <b>Prompt 层</b>——同一检索结果 + 同一工具返回下做 Prompt A/B：前面都正常但输出格式/拒答边界变差才定位 Prompt 回归。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">症状速判：</b>步数/超时突然上升 → 图编排；工具报错和慢调用上升 → 工具；空召回或错召回上升 → 检索；链路都健康但答案质量掉 → Prompt。</div>
  </div>
</div>

---

# 7.11 意图识别准确率如何定义与计算？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">准确率不给拍脑袋数字，必须先定义口径——离线看标注测试集的 Accuracy、F1、混淆矩阵；在线看业务口径的<b>任务完成率、误路由率和人工接管率</b>。「准确率 95%」要明确是 Top1 还是 TopK、是否包含低置信度拒答样本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">计算流程：</b>构建分层数据集（高频、长尾、对抗样本）→ 固定版本评测 → 上线后埋点还原「预测意图 - 真实落地结果」按周回归。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">强调：</b>单看准确率不够，要看<b>错误代价</b>——低频误判触发高风险动作时要提高阈值并强制二次确认，宁可多澄清一次也不能错执行业务。</div>
  </div>
</div>

## 八、安全与风控

---

# 8.1 如何确保 Agent 安全、可控、可追责？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（分层安全）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">① 模型层——对齐训练：</b>RLHF/DPO 微调基础 LLM 遵循「有用、诚实、无害」原则，是所有安全措施的基石；② <b>系统层——最小权限与工具白名单</b>：只给 Agent 完成任务所必需的最少工具和权限；③ <b>执行层——沙箱隔离与资源限额</b>：Agent 生成的代码/命令在受控沙箱（Docker/虚拟机）中执行，即使被劫持破坏范围也限制在沙箱内；④ <b>业务层——高风险动作人类确认（HITL）</b>：执行「删除文件、发送邮件、金融交易」等敏感操作前，Agent 生成执行计划并暂停等待人类明确批准。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">再配合：</b>护栏规则、红队测试、注入攻击测试、全链路审计日志和告警阈值。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">核心原则：</b>「先可控再智能」——即使模型偶发漂移，也不能直接触达不可逆动作。</div>
  </div>
</div>

---

# 8.2 什么是红队测试？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">红队测试用<b>攻击者视角</b>主动找 Agent 系统弱点，评估和提升安全性与鲁棒性——价值是发现常规测试覆盖不到的<b>高风险边界情况</b>，对具备执行能力的 Agent 尤其关键。常见手法：越权请求、提示注入、越狱诱导、工具滥用。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">闭环：</b>红队结果应回流到规则护栏、权限策略和训练数据，形成持续加固闭环。</div>
  </div>
</div>

---

# 8.3 Human-in-the-loop 在 LangGraph 怎么落地？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">在关键节点使用 <b>interrupt 或审批节点</b>，让系统在「高风险动作前」暂停，等待人工确认后继续——典型场景是资金操作、批量发送、生产变更。这样保留自动化效率，同时把不可逆动作的最终控制权交给人。</div>
  </div>
</div>

---

# 8.4 为什么 Agent 项目必须做执行沙箱？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">原因：</b>Agent 一旦具备网页操作、代码执行和外部调用能力，风险从「答错」升级为「<b>误执行</b>」——沙箱把执行面与核心系统隔离，限制破坏范围，避免高权限误操作直达生产环境。没有沙箱，很多企业场景即使效果好也不敢上线。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">沙箱三价值：</b>① <b>隔离</b>——浏览器操作、代码执行、文件读写都在受控环境完成；② <b>可复现</b>——任务过程可追踪、可回放，便于排障和合规审计；③ <b>可治理</b>——权限边界、资源配额、网络策略和操作白名单。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">权限控制与审计：</b>最小权限白名单（可访问域名、可用工具、可写目录）+ CPU/内存/时长限额 + 网络出口策略；关键动作记录为结构化事件日志，绑定任务 ID 和操作者，支持按步骤回放。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">一句话：</b>Agent 的生产门槛不是会不会回答，而是能不能安全执行。</div>
  </div>
</div>

## 九、框架协议与工程化

---

# 9.1 工具调用如何做稳定性与可靠性治理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">Function Calling 本质：</b>给模型一份结构化工具说明（名称、功能、参数 Schema），让它在对话中先判断「要不要调工具」，再输出结构化调用参数（JSON：函数名 + 参数对象）；Agent 编排层执行工具后把结果回填给模型做二次生成。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">从「会调」走向「稳调」三层治理：</b>① <b>输入层</b>——schema 校验与默认值补齐；② <b>调用层</b>——超时、重试、熔断、并发限额；③ <b>结果层</b>——幂等键、去重与统一错误码。工具返回统一结构（status/data/error），禁止「字符串拼接式」返回。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">重试策略：</b>区分可重试（超时、429、临时网络——不改输入也有机会成功）与不可重试（参数错、权限错——输入不变就必失败）错误，避免高危写操作被重复执行；配回退策略（备用工具/降级答案）避免单工具故障拖垮全链路。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">防越权：</b>① <b>最小权限 + 白名单</b>——只给当前任务必需工具，每个工具用短期、范围受限凭证；② <b>调用前二次鉴权</b>——服务端再校验角色权限、资源归属、参数合法性；③ <b>高风险动作强制人工确认 + 可追溯</b>——全链路审计 + 告警 + 一键熔断。</div>
  </div>
</div>

---

# 9.2 Agent/RAG 框架如何选型？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（问题驱动而非框架驱动）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">LangChain：</b>通用 LLM 应用「编排」框架（Chains/Agents/Memory/Callbacks），强在工作流编排、多工具协作和 Agent 执行控制——适合构建复杂的多步骤 Agent；<b>LlamaIndex：</b>专注外部数据的「数据」框架（Connectors/Indexes/Retrievers/Query Engines），强在数据接入、索引构建和检索优化——适合知识库问答与高质量 RAG。实际项目常组合用：LlamaIndex 管数据能力 + LangChain 做上层编排。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">LangGraph：</b>面向有状态、多步骤、可恢复任务的图编排框架；常见是「LangChain 组件 + LangGraph 托管状态机」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">RAGFlow/Haystack（端到端 RAG 平台）：</b>「开箱即用」、对业务人员友好——自动化和可视化、智能分块、GUI。选择它为了「效率与易用性」；选择 LangChain/LlamaIndex 为了「灵活性与控制力」。策略：初期用 RAGFlow 搭基线验证价值，深度优化时再用代码重构实现。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">评价指标：</b>任务成功率、资源消耗（token/API 费用/延迟/步骤数）、鲁棒性与可预测性（错误处理、输出一致性、安全评估）——不看单点 Demo 漂亮程度。</div>
  </div>
</div>

---

# 9.3 LangChain 和 LangGraph 的关系与区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">LangChain 更偏<b>组件库和表达式编排</b>（Prompt、Model、Retriever、Tools、Runnable），适合快速搭建链路；LangGraph 是面向<b>有状态、多步骤、可恢复任务</b>的图编排框架，适合 Agent 工作流。两者不是替代关系，常见是「LangChain 组件 + LangGraph 状态机编排」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">升级时机：</b>当流程从「线性调用」变成「有状态、有分支、可恢复、可治理」时，Chain 的表达力不够，就该升级到 LangGraph 图。</div>
  </div>
</div>

---

# 9.4 LangGraph 生产治理怎么做？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（可控、可恢复、可治理四件事）：</b>
    <div style="margin:6px 0 0;">① <b>流程组件化（LCEL）</b>——Prompt/Model/Parser/Retriever 都抽成 Runnable 组合成可复用链路，便于替换与 A/B；收益：统一抽象、可组合复用、节点级可观测、内建 streaming/batch/并行/重试/fallback、单测粒度细、变更成本低、团队协作稳；② <b>图层硬约束</b>——条件边 + 终止条件 + 最大步数/重试/超时/置信度阈值，把「结束条件」设计成硬约束而不是靠模型自觉停机；③ <b>Checkpoint + 幂等键</b>——长任务中断后从最近状态恢复，不必整条链重跑；和任务 ID 绑定支持幂等重放（Checkpoint 只保证「从哪继续」，幂等设计才保证「不重复副作用」）；④ <b>节点级观测</b>——定位故障来源。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">死循环止血：</b>第一步<b>流量止血</b>——关闭工作流入口（feature flag/路由开关），请求切到降级路径（单轮回答或人工兜底）；然后批量中断在跑实例（按 run_id/thread_id 取消）、临时加硬阈值再恢复小流量。</div>
  </div>
</div>

---

# 9.5 Agent 调用 MCP 的逻辑？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（四阶段）：</b>
    <div style="margin:6px 0 0;">① <b>工具发现</b>——通过 MCP 拿到可用工具清单和 schema，告诉模型「有哪些能力可选」；② <b>工具决策</b>——模型根据当前任务状态输出调用意图（工具名、参数、预期结果）；③ <b>执行与回填</b>——运行工具，拿到结果或错误码，标准化写回状态；④ <b>结果吸收</b>——模型读取工具结果，决定继续调用、改计划还是结束。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程三点：</b>参数校验（防模型传错字段）、防越权（白名单 + 权限上下文）、幂等（重试不产生重复副作用）——让 MCP 是「可控、可审计、可恢复」地调工具。</div>
  </div>
</div>

---

# 9.6 如何理解 MCP？它解决什么问题？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">理解：</b>MCP 是「模型与外部工具之间的<b>标准化协议层</b>」。解决的核心问题：不同模型、不同框架、不同工具之间<b>接入成本高、格式不统一、可移植性差</b>——用 MCP 后，工具对外暴露统一描述和调用方式，模型侧按同一协议发现和调用。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">现状不足：</b>① 工具描述质量参差不齐，schema 不规范时调用成功率下降；② 安全与权限治理在复杂场景下不够细；③ 跨语言、跨环境的调试和观测链路不统一。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">优化方向：</b>工具规范治理（版本化 + 契约测试）、权限与审计（最小权限 + 操作留痕）、观测体系（调用成功率、延迟、错误分类统一上报）——让 MCP 从「协议可用」升级到「生产可用」。</div>
  </div>
</div>

---

# 9.7 MCP API 与普通大模型 API 的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">普通大模型 API 本质是「输入文本，输出文本」；MCP API 是「输入任务上下文，输出可执行工具行为」。三点区别：① <b>交互对象</b>——普通 API 主要和模型交互，MCP 还要和工具生态交互；② <b>数据结构</b>——普通 API 重点是 prompt 和 response，MCP 强调工具 schema、参数校验、执行结果和错误码；③ <b>工程目标</b>——普通 API 追求回答质量，MCP 追求「可执行性 + 可治理性」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">关系：</b>两者不是替代而是组合——模型负责决策，MCP 负责执行和回填；建议把工具调用封成统一执行器，避免业务代码里散落调用逻辑。</div>
  </div>
</div>

---

# 9.8 SSE 原理与作用？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">原理：</b>SSE 是服务端向客户端<b>单向持续推送事件</b>的机制，适合流式文本生成——HTTP 长连接，响应头设置 <code>text/event-stream</code>，服务端按事件帧持续写入，客户端边接收边渲染。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">调用中做的三件事：</b>① 降低首字等待时间（先看到部分结果）；② 过程可视化（当前节点、工具调用中、重试中）；③ 便于中断和恢复（前端断开后按会话状态继续）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程注意：</b>心跳保活、防代理超时、事件幂等（避免重连后重复展示或重复落库）；事件分类型（token/tool_start/tool_end/final）让前端精确展示过程。</div>
  </div>
</div>

---

# 9.9 如何把普通 API MCP 化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（包装、约束、观测三步）：</b>
    <div style="margin:6px 0 0;">① <b>包装</b>——把原 API 能力抽象成工具，定义清晰的输入输出 schema（必填项、类型、枚举、错误码）；② <b>约束</b>——加权限控制、参数校验和幂等键，明确哪些场景可调用、谁可调用、失败如何处理；③ <b>观测</b>——接入日志和指标（调用成功率、P95 时延、错误分布、重试次数）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">注册与测试：</b>把工具描述注册到 MCP Server；上线前做两类测试——<b>契约测试</b>（保证 schema 不破）、<b>回放测试</b>（保证关键 case 稳定）。「MCP 化」不是加一层协议，而是把接口变成可被模型稳定消费的标准能力。</div>
  </div>
</div>

---

# 9.10 为什么选 Spring AI？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">① 与 Spring 生态天然兼容——依赖注入、配置管理、监控链路复用，接入成本低；② 对模型调用、Prompt 模板、向量存储、工具调用有统一抽象，减少重复造轮子；③ 对 Java 后端团队友好，便于把 AI 能力和原有业务服务一体化部署与治理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">选型原则：</b>先看团队熟练度和交付周期，再看框架能力边界，必要时允许混合方案（LangChain/LangGraph 在 Agent 编排上生态更活跃）而不是框架宗教。</div>
  </div>
</div>

---

# 9.11 多模型支持架构怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（统一网关 + 策略路由 + 观测闭环）：</b>
    <div style="margin:6px 0 0;">① <b>统一网关</b>——标准化请求与响应，屏蔽不同模型 SDK 差异；② <b>策略路由</b>——按任务类型、时延预算、成本上限和稳定性动态选择模型；③ <b>观测闭环</b>——记录每次调用的质量、耗时和成本，反哺路由策略。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实现：</b>抽象 <code>ModelProvider</code> 接口，统一超时、重试、熔断和降级，支持主备模型切换；高风险任务配置「双模型交叉校验」或「主模型失败后回退模板回复」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">强调：</b>多模型不是越多越好，关键是「可灰度、可监控、可回滚」，否则只是把复杂度前移。</div>
  </div>
</div>

---

# 9.12 MCP、Skill、RAG 的关系？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b>Skill</b> 更像「方法论」或「流程模板」，定义模型遇到某类任务时按什么步骤做——适合封装重复性流程和团队经验；<b>RAG</b> 解决「知识从哪里来」——把当前任务相关且可能实时变化的外部信息检索出来喂给模型。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">关系：</b>Skill 是能力封装，RAG 是知识供给，两者配合不是替代——一个 Skill 内部常规定「先检索、再筛选、再生成、再校验」，其中的「检索」往往就是 RAG。</div>
  </div>
</div>

---

# 9.13 Subagent 与 Agent Team 模式的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">Subagent：</b>处理轻量级、短周期事务——主 Agent 拆任务并与各 Sub Agent 通信（方案设计、前端开发、后台开发、测试），任务执行完后 Subagent 被回收，周期短。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">Agent Team：</b>负责大型任务，适合上下游、多团队多角色并行协作——团队间共享状态、记忆和任务看板，有团队级的路由、评审、投票机制，像现实中多团队协作开发大项目。</div>
  </div>
</div>

---

# 9.14 什么是 Harness Engineering？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念：</b>Harness（驾驭工程）的核心是<b>系统性地设计约束、反馈链路和持续改进机制</b>，使 AI Agent 在规模化、长周期的软件开发中可靠运行——包含系统提示词、工具、文件系统、沙箱、编排逻辑和检查机制，本质上回答三个问题：AI 在哪干活？用什么干活？怎么知道干得对不对？</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">本质：</b>把工作方法和判断标准转化为 AI 能理解和执行的环境——AI 出错时反问「我的环境里缺了什么」；工程师角色从「写代码」转向「设计环境/标准、建立反馈」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实践细节：</b>除了开发 Agent，并行起一个<b>测试 Agent</b>打通反馈链路——测试 Agent 能读所有工具链的可观测日志、端到端模仿人在浏览器验证每个功能、收集报错信息给开发修正。Linter 错误信息面向 AI 重写——每条报错包含 AI 决策所需完整信息：<b>规则意图</b>（这条规则防什么问题）、<b>具体修复方向</b>（不是「你错了」而是「你应该这样改」）、<b>上下文约束</b>（依赖方向等架构规则违反时说明是哪条链路）。</div>
  </div>
</div>

---

# 9.15 Claude Code 架构设计的启发？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（六课）：</b>
    <div style="margin:6px 0 0;">① <b>Agent 工程特别重要</b>——真正调用 LLM API 的部分不到 5%，剩下 95% 是安全检查、权限管理、上下文压缩、错误恢复、多 Agent 协调；② <b>提示词分层管理</b>——静态层走缓存节省费用、动态层注入实时环境信息，每个工具配一份给 AI 看的使用手册；③ <b>安全的重视</b>——默认所有工具状态「不安全、会写入」，除非明确声明安全；④ <b>上下文管理是生命线</b>——上下文窗口是策略不是开关，区分哪些内容可丢、哪些必须保留；⑤ <b>记忆系统要「精确」不「贪多」</b>——宁可漏掉可能有用的记忆，也不把不相关记忆塞进上下文污染判断；⑥ <b>多 Agent 协作给每个 Agent 明确「身份」</b>——协调者只协调、执行者只执行，职责边界在设计阶段写死。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">工程细节补充：</b>Claude Code 共 51 万行代码；为让 BashTool 安全运行写了 18 个安全文件、9 层审查流程（默认所有工具「不安全、会写入」，按需放开）；上下文三层递进压缩——先清理旧工具调用结果（对话主线保留）→ token 用量接近 87% 自动触发压缩 → 最后让 AI 生成全局摘要替换历史；子 Agent 注入强硬指令（「你是工人不是经理，不许派活，直接干，汇报不超过 500 字」）防止子 Agent 无限生成子 Agent。</div>
  </div>
</div>


---

> 返回导航：[[00-总导航|00-总导航]]
