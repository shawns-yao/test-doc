---
aliases:
- "ByteBuffer 的 position limit capacity 和 flip compact 如何理解"
---

# 02 ByteBuffer 的 position limit capacity 和 flip compact 如何理解

## 01 核心回答

capacity 是容量上限，limit 是当前操作边界，position 是下一次相对读写的位置，保持 0 ≤ position ≤ limit ≤ capacity。读写模式不是一个隐藏开关，而是应用通过这些索引约定当前数据范围。

## 02 为什么写完需要 flip

写入时 position 前进，写到哪里就有多少有效数据。flip 把 limit 设为原 position，再把 position 置 0，于是后续从头读取刚写入的内容；若没有 flip，读可能从末尾继续，或把未写入区域当数据。

clear 将 position 置 0、limit 置 capacity，表示可重新写入，不会把底层字节清零。rewind 只回到起点、保留 limit，适合重读当前有效范围。compact 把尚未读完的数据搬到开头，position 放到剩余数据之后，让后续写入能够接续半包；它不是 clear 的同义词。

## 03 部分读写与共享视图

通道一次 read/write 不保证填满或写光缓冲区，应根据返回值和 remaining 继续状态机；-1 表示读到流结束，与 0 的暂时无进展不同。slice/duplicate 可以共享底层数据但具有独立位置状态，这不意味着线程安全；只读视图也不能阻止其他可写视图修改同一内容。

## 04 面试口述与参考

“写完 flip 限定有效区，没读完 compact 留住尾巴继续收，全部消费后 clear 复用；这些操作主要改变索引或搬移数据，不负责消息协议本身。”

- [Buffer 索引不变量与 flip/clear/compact 语义](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/Buffer.html)
- [[八股/01-Java/05-JavaIO/01-BIO NIO AIO 有什么区别？为什么 NIO 不等于非阻塞|非阻塞读写为什么需要状态机]]

- [ByteBuffer.compact 与共享视图](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/ByteBuffer.html#compact())

## 05 所属专题

- [[八股/01-Java/05-JavaIO/00-JavaIO导航|JavaIO导航]]
