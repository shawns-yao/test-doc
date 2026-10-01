---
aliases:
- "checked 异常和 unchecked 异常有什么区别？资源如何可靠关闭"
---

# 05 checked 异常和 unchecked 异常有什么区别？资源如何可靠关闭

## 01 核心回答

Throwable 下有 Error 和 Exception。RuntimeException 及其子类与 Error 及其子类属于 unchecked，编译器不要求声明或捕获；其他异常通常属于 checked，方法应捕获或在签名中声明。这个分类是编译期处理要求，不等于业务上能否恢复。

---

## 02 如何设计异常边界

业务边界应保留异常原因与必要上下文，避免每层重复记录后再抛造成日志噪音。能够恢复才在当前层处理，不能恢复则向适当边界传播；别把异常吞成成功结果。Error 通常表示严重运行问题，不应以 catch Throwable 当作正常业务恢复策略。

捕获 InterruptedException 后若无法继续向上抛，应按任务取消协议处理并通常恢复中断标记，不能悄悄吞掉取消请求。Spring 默认回滚规则与 Java checked 分类有关，但可以配置，见事务题。

---

## 03 关闭资源为什么优先 try-with-resources

实现 AutoCloseable 的资源可由 try-with-resources 按创建的逆序关闭。如果主逻辑与 close 同时抛异常，关闭异常作为 suppressed exception 附在主要异常上，有助保留真正失败原因。不要在 finally 中 return，它可能覆盖原返回值或异常。

finally 也不是任何终止条件下都执行的“物理保证”，进程强杀、JVM 终止等场景不能依赖它释放外部资源，因此远端资源还需要超时和租约等机制。

---

## 04 参考与关联

- [JLS 异常分类与检查](https://docs.oracle.com/javase/specs/jls/se21/html/jls-11.html)
- [[八股/02-Spring框架/01-Spring核心/04-事务失效有哪些场景？同类内部调用为什么失效|异常如何影响事务回滚]]

- [JLS try-with-resources 与 suppressed exception](https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3)

---

## 05 所属专题

- [[八股/01-Java/01-Java基础/00-Java基础导航|Java基础导航]]
