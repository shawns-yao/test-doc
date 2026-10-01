---
aliases:
- "Netty 如何体现 Reactor 模型？"
- "计算机网络 5.15 Netty 如何体现 Reactor 模型？"
---

# 15 Netty 如何体现 Reactor 模型？

## 01 核心回答

**模型映射：**Reactor 的核心是**事件驱动分发**：Reactor 线程监听就绪事件，Handler 负责处理读写和协议状态。Netty 常用主从 Reactor，Boss `EventLoopGroup` 负责 accept，新连接注册到 Worker `EventLoop`，Worker 负责 read、decode、业务事件传播和 write。

**Pipeline 处理：**数据进入 `ChannelPipeline` 后依次经过入站和出站 `ChannelHandler`；解码器负责半包/粘包和协议帧，业务 handler 处理请求，编码器把响应写回 channel。每个 channel 通常绑定一个 EventLoop，减少并发访问同一连接状态的锁竞争。

**线程边界：**非 I/O 的重计算、阻塞数据库和远程调用应下沉到独立业务线程池，再通过事件回到 EventLoop；线程池要有界、可监控，并避免在 I/O 线程里调用 `sync`、`await` 或长时间 `join`。

**常见风险：**EventLoop 被阻塞会让同一线程上的多个连接一起超时；业务线程池过小会造成任务排队，过大又会争抢 CPU。需要关注 event loop pending tasks、单次 handler 耗时、写缓冲区水位和连接关闭后的资源释放。

**面试追问：**如果在 I/O 线程里做重计算，会出现什么线上表现？一个连接为什么通常固定到同一个 EventLoop？如何处理写缓冲区高水位和背压？

---

> 返回导航：[[八股/00-总导航|00-总导航]]

## 02 理解补充与边界校订

Channel 绑定 EventLoop 有助于连接内事件串行，但将 handler 放到其他 executor 后必须重新考虑并发、结果顺序和共享状态。写成功通常意味着交给网络栈或完成该异步写操作，不等于对端业务确认。背压要落实到减少生产、控制待发送队列和请求并发，而非仅监控高水位。

---

## 03 依据与延伸阅读

- [Netty ChannelPipeline](https://netty.io/4.1/api/io/netty/channel/ChannelPipeline.html)

---

## 04 相关问题

- [[八股/06-计算机基础/04-TCP与UDP/08-TCP 粘包和半包是什么？如何解决|TCP 粘包和半包是什么？如何解决]]：字节流拆帧与Netty解码
- [[八股/06-计算机基础/06-网络原理与IO/14-IO 多路复用原理怎么讲|IO 多路复用原理怎么讲]]：事件就绪到Reactor处理

---

## 05 所属专题

- [[八股/06-计算机基础/06-网络原理与IO/00-网络原理与IO导航|网络原理与IO导航]]
