---
aliases:
- ReentrantLock是什么
---

`ReentrantLock` 和 `synchronized` 做的核心事情很像，都是让多个线程围绕同一把锁竞争，从而保护共享数据。

---
# 04 ReentrantLock 是什么？

synchronized是 java 中的一个关键字，锁的获取和释放由 JVM 帮你管理。核心是线程想进入临界区，需要进行抢锁，抢到才会执行，没有抢到，等待
而 ReentrantLock 是 Java 中的一个类 ，本质和 synchronized 的思路相同
但是需要自己明确的加锁，执行代码，以及解锁

---
## 01 为什么还需要 ReentrantLock？

原因是，有些时候并不是拿不到锁，就一直等，我们可能还想要，拿不到锁了，就不等了，或者拿不到锁，我最多等待 3 秒，亦或者我希望线程按照比较公平的顺序获得锁
这些事情，ReentrantLock 控制起来更加方便

---
## 02 ReentrantLock 是什么意思？

Reentrant  +  Lock ，Reentrant是可重入的意思，

可重入锁，就是一个线程已经获取到了这把锁，可以再获得同一把锁，而不会把自己卡死。
比如 A 获取到了锁，在执行过程中又调用了另外一个方法，这个方法中又执行获取锁的操作，那么这个时候不会因为 A 等待 A 自己释放锁导致卡死，因为锁是可重入的

当然 synchronized也是支持可重入的

---
## 03 ReentrantLock 为什么一定推荐 finally 解锁？

Synchronized在代码块结束时，JVM 自动释放锁，哪怕是抛异常，锁一般也会正确释放。

但是 ReentrantLock 需要程序员自己调用 `unlock()`。所以如果代码运行到一半抛异常了，那锁可能一直不释放。其他线程全部卡住。

所以 `finally` 的意义就是：

> 不管业务代码正常结束还是抛异常，都尽量确保锁被释放。

---
## 04 ReentrantLock 和 synchronized 最大区别是什么？

synchronized，是 JVM 管理锁的释放，但是 ReentrantLock 是程序员自己控制，正因为能自己控制，所以 ReentrantLock 能做更多事情。

尝试拿到锁
synchronized 拿不到锁就一直等
ReentrantLock 还可以，尝试一下，拿得到就执行，拿不到就算了，这在高并发下的场景很有用

可以响应中断，比如线程 B 正在等待锁，如果这个时候发现这个任务已经不需要锁了，那么 ReentrantLock 提供了可中断获取锁的方式。

公平锁，类似排队买票，谁先等，谁先拿
非公平锁：新来的线程也可能直接抢到锁。ReentrantLock 默认非公平锁，因为非公平锁通常吞吐量更高。公平意味着系统需要尽量维护排队顺序，会增加一些调度成本。

## 05 核心实现方向

ReentrantLock 的底层实现依赖 AQS。可以先记住这条主线：线程尝试修改同步状态，获取失败后进入等待队列，释放锁后唤醒后继线程；可重入则通过同步状态记录同一线程的重入次数。

## 06 常见追问

### 06.1 ReentrantLock 为什么可以响应中断？

它提供 `lockInterruptibly()`，线程在等待锁的过程中可以响应中断，从而及时结束已经不再需要的等待。普通 `lock()` 的等待方式不能用同样的方式处理中断。

### 06.2 ReentrantLock 如何实现超时获取？

可以使用 `tryLock()` 立即尝试，也可以使用带超时时间的 `tryLock(timeout, unit)`。获取失败时可以执行降级、重试或返回，而不是无限等待。

### 06.3 ReentrantLock 和 synchronized 如何选择？

- 只需要简单互斥、自动释放和较少控制时，优先考虑 [[八股/01-Java/03-Java并发/03-Synchronized是什么|Synchronized是什么]]。
- 需要超时、可中断、公平策略或多个 Condition 时，考虑 ReentrantLock。
- 选择 ReentrantLock 后，要把加锁和解锁范围控制清楚，并保证释放逻辑可靠。

## 07 相关笔记

- [[八股/01-Java/03-Java并发/03-Synchronized是什么|Synchronized是什么]]
- [[八股/01-Java/03-Java并发/01-Java 内存模型（JMM）是什么？|Java 内存模型（JMM）是什么？]]
- AQS
- [[八股/01-Java/03-Java并发/08-死锁如何排查|死锁如何排查]]

## 08 旧面经的详细补充


**synchronized**

JVM 关键字，基于 Monitor，锁实现随 HotSpot 版本变化；使用简单（方法/代码块）；自动释放锁（异常也释放）。

**局限：**等待进入监视器不能取消或超时，不提供公平性配置；可使用对象的 wait/notify 条件等待，但一个对象只有一个等待集。

**ReentrantLock**

JDK 类，基于 **AQS**（CLH 队列 + CAS 状态）。支持：**可中断**（`lockInterruptibly`）、**超时**（`tryLock(timeout)`）、**公平锁**（构造参数）、**多个 Condition**（精确唤醒）、可查询锁状态。

**代价：**必须手动 `unlock`（finally 中释放）；API 复杂。

**怎么选：**简单互斥用 `synchronized`（简洁 + 自动释放）；需要超时/中断/公平/多条件（如生产者消费者精确唤醒）用 `ReentrantLock`；两者都是可重入锁。不要把 HotSpot 的锁升级路径套在 ReentrantLock 上；后者由 AQS 管理同步状态与等待。

**面试追问：**可重入怎么实现（AQS 的 state 计数）；公平锁为什么慢（队列唤醒 + 上下文切换）；Condition 和 wait/notify 的区别（可有多个条件队列 vs 每个监视器一个等待集；两者都支持单个或全部通知）。

---

## 09 AQS 如何支持重入与公平

state 为 0 时尝试 CAS 获取；成功后记录持有线程。当前持有者重入时增加 state，unlock 递减，只有减到 0 才真正释放并推动等待者。失败线程进入 AQS 同步队列，必要时 park；被唤醒之后仍要重新竞争，唤醒本身不授予锁。

公平构造参数倾向于让等待时间最长者先获取，通常降低吞吐量，但不保证操作系统的线程调度公平。特别是无参 tryLock() 即使在公平锁上也允许插队；带超时版本遵守相应公平获取规则。这是 API 明确规定的例外。

## 10 Condition 的完整等待流程

持锁检查条件，不满足则 await：保存重入状态、释放锁、进入该 Condition 的等待队列。signal 将符合条件的等待者推进重新竞争锁的流程；等待者真正恢复前必须重新持有锁。由于存在虚假唤醒和其他线程抢先改变条件，应循环检查谓词。signal 和 await 一般都需持有创建该 Condition 的锁。

多个 Condition 能区分“队列非空”和“队列未满”等等待原因，减少无关唤醒；它并不保证点名任意线程，也不省掉业务条件检查。

## 11 使用与纠错要点

只在成功获取锁后进入对应的 finally 解锁范围。tryLock 失败时不能 unlock；非持有线程解锁会抛 IllegalMonitorStateException。不要把查询锁状态当作下一步操作必然安全的条件，因为状态随时会改变。

本次修正旧文“wait/notify 只能全量唤醒”“两种锁都走锁升级”的不准确对比。

- [ReentrantLock：公平性、tryLock 与使用约定](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html)
- [Condition：条件队列与虚假唤醒](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/Condition.html)
- 进一步理解队列：[[八股/01-Java/03-Java并发/07-什么是 AQS|AQS 同步状态与等待队列]]

## 12 相关问题与延伸

- [[八股/01-Java/03-Java并发/02-Volatile|Volatile]]：反向关联：此题引用了本题的机制或边界
- [[八股/01-Java/03-Java并发/05-ThreadLocal会造成内存泄露吗|ThreadLocal会造成内存泄露吗]]：反向关联：此题引用了本题的机制或边界
- [[八股/01-Java/03-Java并发/06-乐观锁与悲观锁的核心区别是什么？分别如何实现（CAS、synchronized），各自适用于什么场景|乐观锁与悲观锁的核心区别是什么？分别如何实现（CAS、synchronized），各自适用于什么场景]]：反向关联：此题引用了本题的机制或边界

## 13 所属专题

- [[八股/01-Java/03-Java并发/00-Java并发导航|Java并发导航]]
