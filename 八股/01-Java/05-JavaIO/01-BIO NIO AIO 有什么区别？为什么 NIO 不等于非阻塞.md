---
aliases:
- "BIO NIO AIO 有什么区别？为什么 NIO 不等于非阻塞"
---

# 01 BIO NIO AIO 有什么区别？为什么 NIO 不等于非阻塞

## 01 核心回答

BIO 通常指调用方阻塞等待完成的传统流式 I/O；NIO 是 Java 的缓冲区/通道 API 家族，可包含阻塞与非阻塞模式，Selector 用于就绪事件多路复用；AIO 指通过回调或 Future 获得操作完成结果的异步通道 API。包名和“阻塞/异步”不是一一对应关系。

---

## 02 就绪通知与完成通知

非阻塞 read 在暂时没有数据时可以返回 0，事件循环用 Selector 等待哪些通道已就绪，再由应用执行读写；就绪不保证完整业务消息已到，也不保证一次写完全部缓冲区。应用要维护半包、剩余字节和背压状态。

异步 I/O 把发起与完成分开，完成处理器收到结果后继续处理。若发起后马上 Future.get，调用线程仍会阻塞等待；API 异步也不保证底层操作系统一定采用某个特定内核机制。

---

## 03 为什么不能把慢业务放在事件循环

Selector 可以让少量线程管理大量连接，但事件循环若执行数据库查询、长计算或阻塞等待，会拖慢它管理的其他连接。网络就绪处理应短小，耗时业务交给合适执行器，并限制队列、并发和可写兴趣事件，避免空转或内存堆积。

FileChannel 不是可注册 Selector 的 SelectableChannel，不能因为同在 NIO 包就认为所有文件操作都非阻塞。JDK 21 虚拟线程则提供另一条“保留阻塞写法、降低线程成本”的路径，仍需控制资源。

---

## 04 参考与关联

- [Selector API：选择器与就绪集合](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/Selector.html)
- [[八股/01-Java/03-Java并发/10-Java 21 虚拟线程是什么？适合什么场景|虚拟线程与事件循环的选择]]

---

## 05 相关问题与延伸

- [[八股/01-Java/05-JavaIO/02-ByteBuffer 的 position limit capacity 和 flip compact 如何理解|ByteBuffer 的 position limit capacity 和 flip compact 如何理解]]：反向关联：此题引用了本题的机制或边界

---

## 06 所属专题

- [[八股/01-Java/05-JavaIO/00-JavaIO导航|JavaIO导航]]
