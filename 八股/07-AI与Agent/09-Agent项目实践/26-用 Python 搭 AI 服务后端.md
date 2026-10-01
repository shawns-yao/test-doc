---
aliases:
- "用 Python 搭 AI 服务后端？"
- "Agent 4.22 用 Python 搭 AI 服务后端？"
---

# 26 用 Python 搭 AI 服务后端？

## 01 核心回答

优先用 **FastAPI** 搭服务层（适合 AI 应用接口、开发效率高、方便异步）；数据层接 MySQL/Postgres，缓存和会话状态用 Redis；长任务（检索、生成报告、处理文件）引入**异步任务机制**（任务队列/后台任务）；模型调用层单独封装——统一请求、重试、日志、限流和 fallback；流式输出用 SSE 或 WebSocket；关注可观测性（日志、trace、错误监控、token 成本统计）。

**分层与契约：**API 层只处理入参校验和响应协议；Service 层负责业务编排和规则决策；Repository/Model 层只管数据读写；Workflow 层专注状态转移。层间统一用 DTO 和明确错误码——替换 LLM、切换向量库或升级存储只影响局部层。

---

## 02 异步接口与可靠后台任务不同

FastAPI 的 async 适合非阻塞 I/O，但同步 HTTP 客户端、重 CPU 解析或模型推理会阻塞事件循环，应使用合适的线程/进程或独立 worker。框架自带 BackgroundTasks 适合轻量请求后工作，不应默认承担崩溃后必须续跑的长任务。

重要长任务由持久队列和状态存储承接，worker 做幂等处理、重试、租约与超时控制。API 返回任务 ID，客户端通过查询或 SSE 观察进度；worker 崩溃后可核验上次副作用再恢复。

---

## 03 服务治理

复用连接池，控制模型 API 并发与供应商配额，取消请求时明确是否继续后台工作；区分客户端超时、模型超时和任务截止时间。上传文件与产物设权限、大小和保留期，日志不含密钥或整段私人材料。

口述：“HTTP 层负责交互，持久任务系统负责可靠执行，异步语法不等于任务有恢复能力。”

---

## 04 关联追问

- [[八股/07-AI与Agent/01-Python与机器学习/01-Python 工作中常用哪些包？分别用于什么场景|Python 工作中常用哪些包？分别用于什么场景？]]
- [[八股/07-AI与Agent/07-框架协议与工程化/08-SSE 原理与作用|SSE 原理与作用？]]
- [[八股/07-AI与Agent/09-Agent项目实践/22-从 0 到 1 搭 AI 后端服务怎么拆|从 0 到 1 搭 AI 后端服务怎么拆？]]

---

## 05 参考资料

- [FastAPI BackgroundTasks 官方说明](https://fastapi.tiangolo.com/tutorial/background-tasks/)
- [LangGraph 官方持久化文档](https://docs.langchain.com/oss/python/langgraph/persistence)

---

## 06 所属专题

- [[八股/07-AI与Agent/09-Agent项目实践/00-Agent项目实践导航|Agent项目实践导航]]
