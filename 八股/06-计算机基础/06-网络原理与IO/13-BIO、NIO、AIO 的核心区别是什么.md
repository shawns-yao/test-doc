---
aliases:
- "BIO、NIO、AIO 的核心区别是什么？"
- "计算机网络 5.13 BIO、NIO、AIO 的核心区别是什么？"
---

# 13 BIO、NIO、AIO 的核心区别是什么？

## 01 核心回答

**概念原理：**BIO 是**同步阻塞**，调用线程等待 I/O 完成，传统模型是一连接一线程；NIO 支持 Channel 的阻塞与非阻塞模式，典型多路复用方案使用**非阻塞 Channel + Selector** 处理多个连接的就绪事件；AIO 是**异步非阻塞**，提交 I/O 后立即返回，由系统在完成后通过回调或 completion handler 通知。

**维度对比：**BIO 编程直观但线程数随连接数增长，容易受线程栈和上下文切换限制；NIO 通过少量事件循环线程支撑大量连接，但状态管理和半包拆包更复杂；AIO 理论上把等待交给系统，实际效果取决于操作系统、JDK 实现和底层文件/网络支持。

**选型与误区：**高并发网络服务通常采用 NIO/Netty，平衡性能与工程复杂度；阻塞数据库调用即使放在 NIO 服务里仍会阻塞业务线程，因此还要配合独立线程池。非阻塞不等于异步，NIO 的 selector 线程仍然同步地处理就绪事件。

**面试追问：**Java 生态里为什么 AIO 在业务系统里不如 NIO 常见？NIO 服务如何处理粘包、半包和慢客户端？

---

## 02 理解补充与边界校订

BIO/NIO/AIO 要分别讲调用是否等待，以及通知的是就绪还是完成。Java NIO 包含阻塞模式 Channel，不能把整个包名直接等同于非阻塞；Selector 才是典型多路复用组件。虚拟线程可用阻塞写法提高大量等待任务的并发密度，因此「一连接一线程必然不可扩展」也需限定为平台线程模型。

---

## 03 为什么不能简单说 AIO 一定比 NIO 更好

AIO 交付的是完成通知，但异步接口背后仍可能用线程池完成等待、执行或派发回调；具体路径由操作系统和 JDK provider 决定。`AsynchronousChannelGroup` 本身关联线程池，完成处理器应快速返回，不能把“异步”理解成完全没有线程成本。依据：[Java 21 AsynchronousChannelGroup](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/AsynchronousChannelGroup.html)。

工程选型还要看协议编解码、连接管理、TLS、背压、可观测性和团队现有框架。NIO/Netty 的事件循环能把这些能力组织在成熟管线中；AIO 的回调、取消、超时和缓冲区所有权同样需要设计。是否更快或更普及不能只凭名称下结论，应在目标平台、消息大小、并发和尾延迟要求下压测；已有同步业务也可评估虚拟线程，而不是仅在 NIO/AIO 中二选一。

问题来源：[agent_java_offer 原题](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/02_后端/10_网络I_O与发布治理/01_核心问答.md)，Repository contributors，采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)。改动：补答原题中的 AIO 选型追问，保留平台与实现边界，不把采用率当作未经验证的事实。归属及非商业许可说明仅针对本次引用/改编内容，不改变本页其他原有内容的许可。

---

## 04 依据与延伸阅读

- [Java Selector API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/Selector.html)

---

## 05 相关问题

- [[八股/06-计算机基础/06-网络原理与IO/07-零拷贝是什么？用户态协议栈能解决什么问题|零拷贝是什么？用户态协议栈能解决什么问题]]：拷贝成本与通知模型

---

## 06 所属专题

- [[八股/06-计算机基础/06-网络原理与IO/00-网络原理与IO导航|网络原理与IO导航]]
