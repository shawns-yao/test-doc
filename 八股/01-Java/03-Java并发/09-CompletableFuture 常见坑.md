---
aliases:
- "CompletableFuture 常见坑？"
- "Java 2.9 CompletableFuture 常见坑？"
---

# 09 CompletableFuture 常见坑？

## 01 核心回答

**核心认识：**`CompletableFuture` 是一个可组合的异步结果容器，不等于“自动异步”。不带 executor 的 `async` 方法通常使用公共 `ForkJoinPool.commonPool()`，而不带 `async` 的回调可能直接在完成前一个阶段的线程上执行。

**组合语义：**① `thenApply` 是结果转换，返回普通值；② `thenCompose` 用于把两个异步阶段串联，避免产生嵌套的 `CompletableFuture<CompletableFuture<T>>`；③ `thenCombine` 等待两个独立任务后合并结果；④ `allOf` 只表示全部完成，结果需要从原 future 中收集，不能直接得到列表。

**最常见的坑：**① 在回调中调用 `join/get`，阻塞公共线程池，形成线程饥饿；② 把 CPU 密集型任务和网络、数据库等阻塞任务放在同一个线程池；③ 忽略返回的新 future，导致异常或转换结果丢失；④ 误以为 `whenComplete` 能吞掉异常，实际上它主要用于观察，异常仍会继续传播；⑤ 异常只在最终 `join` 时暴露，定位距离真实故障点很远。

**异常与超时：**用 `exceptionally` 提供降级值，用 `handle` 同时处理正常值和异常，用 `whenComplete` 记录日志和指标；对外部调用使用 `orTimeout` 控制最长等待，允许降级时使用 `completeOnTimeout` 返回兜底值。超时不一定能中断底层 HTTP 或数据库调用，底层客户端仍需配置连接、读写和总超时。

**线程池与上下文：**为阻塞 I/O、CPU 计算和低延迟任务配置独立 executor，设置有界队列、拒绝策略和线程命名；在异步边界显式传递 traceId、用户身份和 MDC，上下文不能依赖 ThreadLocal 自动继承。提交任务前还要限制并发度，避免 `allOf` 一次性扇出过多请求。

**重试与取消：**重试必须限定异常类型、次数和退避时间，并确保操作幂等；`cancel` 主要完成 future 的状态变更，不保证已经执行的底层任务被真正终止，因此需要结合任务中断响应和客户端取消接口。生产环境应记录每个阶段耗时、排队长度、成功率、异常类型和超时数量。

**面试追问：**“为什么不用一条链全部异步？”异步只能隐藏等待，不能消除下游容量和线程池约束；“如何保证结果顺序？”为每个任务保留输入序号，完成后按序收集；“如何避免级联超时？”给整条链路设置总预算，再为每个子调用分配更短的剩余超时，并在超时后及时停止继续扇出。

## 02 回调到底在哪里执行

非 Async 回调可能在完成上游的线程执行，也可能在注册时发现上游已完成后由当前调用线程执行，因此别把耗时回调塞进网络事件循环。Async 表示交给 executor 调度，不表示一定获得专用新线程；显式提供 executor 才能控制隔离边界。

allOf 在所有输入完成后完成，任一异常会使组合结果异常，但它不是自动 fail-fast 取消其他任务。anyOf 接受最先完成的结果，包括异常，不等价于“第一个成功结果”。需要最快成功、限并发或失败取消时，应先定义这些语义再组合。

## 03 超时和取消不能只改结果状态

orTimeout/completeOnTimeout 完成的是当前 CompletableFuture，本身不是客户端调用的终止器。cancel(true) 的 mayInterruptIfRunning 在 CompletableFuture 中没有中断执行的效果；应通过真正运行任务的 Future、客户端取消句柄和协作取消标志停止资源消耗。共享同一个 future 的多个消费者还应注意超时方法修改同一对象的影响。

## 04 参考与关联

- [CompletableFuture：执行策略、allOf、anyOf、cancel](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- [[八股/01-Java/03-Java并发/05-ThreadLocal会造成内存泄露吗|异步上下文为什么不会自动传播]]

## 05 所属专题

- [[八股/01-Java/03-Java并发/00-Java并发导航|Java并发导航]]
