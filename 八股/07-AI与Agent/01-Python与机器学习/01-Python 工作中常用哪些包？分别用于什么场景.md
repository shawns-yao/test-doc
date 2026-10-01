---
aliases:
- "Python 工作中常用哪些包？分别用于什么场景？"
- "AI 1.1 Python 工作中常用哪些包？分别用于什么场景？"
---

# 01 Python 工作中常用哪些包？分别用于什么场景？

## 01 核心回答


数据处理

`pandas`、`numpy`。

可视化

`matplotlib`、`seaborn`。

机器学习

`scikit-learn`；深度学习按项目用 PyTorch。

HTTP/服务

`requests`、`httpx`、FastAPI。

校验/配置

`pydantic`、`yaml`。

选择包时会考虑**生态、性能、团队维护成本和部署环境**，不会为了使用库而引入不必要的依赖。

**面试追问：**pandas 和 numpy 的区别（表格 vs 数组计算）；FastAPI 和 Flask 的区别（异步 + 类型校验）；为什么不用 requests 而用 httpx（异步支持）。

---

## 02 进一步讲清选择依据

pandas 的核心是带索引的表格操作，NumPy 侧重同类型多维数组和向量化运算，二者有重叠但不能简单理解成“一个快、一个慢”。Web 框架支持 async 也不意味着业务自动非阻塞：在协程里执行同步网络请求或重 CPU 计算仍会阻塞。

追问“实际用过哪些”时，只讲自己能解释输入输出、错误处理和依赖版本的包。可改编回答：“数据清洗会比较 pandas 与 SQL 下推；并发网络请求考虑异步客户端和连接池，但同步小脚本不为异步而异步。”不要把本页列举当成个人使用经历。

---

## 03 关联追问

- [[八股/07-AI与Agent/01-Python与机器学习/02-Python 是动态类型语言吗|Python 是动态类型语言吗？]]
- [[八股/07-AI与Agent/09-Agent项目实践/26-用 Python 搭 AI 服务后端|用 Python 搭 AI 服务后端？]]

---

## 04 参考资料

- [Python 数据模型](https://docs.python.org/3/reference/datamodel.html)

---

## 05 所属专题

- [[八股/07-AI与Agent/01-Python与机器学习/00-Python与机器学习导航|Python与机器学习导航]]
