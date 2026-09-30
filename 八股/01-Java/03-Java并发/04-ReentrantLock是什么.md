---
aliases:
  - ReentrantLock是什么
---

 
 `ReentrantLock` 和 `synchronized` 做的核心事情很像，都是让多个线程围绕同一把锁竞争，从而保护共享数据。


---
# ReentrantLock 和 synchronized 的区别

synchronized是 java 中的一个关键字，锁的获取和释放由 JVM 帮你管理。核心是线程想进入临界区，需要进行抢锁，抢到才会执行，没有抢到，等待
而 ReentrantLock 是 Java 中的一个类 ，本质和 synchronized 的思路相同
但是需要自己明确的加锁，执行代码，以及解锁

---
# 为什么还需要 ReentrantLock？

原因是，有些时候并不是拿不到锁，就一直等，我们可能还想要，拿不到锁了，就不等了，或者拿不到锁，我最多等待 3 秒，亦或者我希望线程按照比较公平的顺序获得锁
这些事情，ReentrantLock 控制起来更加方便

---
# ReentrantLock 是什么意思？

Reentrant  +  Lock ，Reentrant是可冲入的意思，

可重入锁，就是一个线程已经获取到了这把锁，可以在获得同一把锁，而不会把自己卡死。
比如 A 获取到了锁，在执行过程中又调用了另外一个方法，这个方法中又执行获取锁的操作，那么这个时候不会因为 A 等待 A 自己释放锁导致卡死，因为锁是可重入的

当然 synchronized也是支持可重入的

---
# ReentrantLock 为什么一定推荐 finally 解锁？

Synchronized在代码块结束时，JVM 自动释放锁，哪怕是抛异常，锁一般也会正确释放。

但是 ReentrantLock 需要程序员自己调用 `unlock()`。所以如果代码运行到一半抛异常了，那锁可能一直不释放。其他线程全部卡住。

所以 `finally` 的意义就是：

> 不管业务代码正常结束还是抛异常，都尽量确保锁被释放。


---
# ReentrantLock 和 synchronized 最大区别是什么？

synchronized，是 JVM 管理锁的释放，但是 ReentrantLock 是程序员自己控制，正因为能自己控制，所以 ReentrantLock 能做更多事情。

尝试拿到锁
	synchronized 拿不到锁就一直等
	ReentrantLock 还可以，尝试一下，拿得到就执行，拿不到就算了，这在高并发下的场景很有用

可以响应中断，比如线程 B 正在等待锁，如果这个时候发现这个任务已经不需要锁了，那么 ReentrantLock 提供了可中断获取锁的方式。

公平锁，类似排队买票，谁先等，谁先拿
非公平锁：新来的线程也可能直接抢到锁。ReentrantLock 默认非公平锁，因为因为非公平锁通常吞吐量更高。公平意味着系统需要尽量维护排队顺序，会增加一些调度成本。

## 核心实现方向

ReentrantLock 的底层实现依赖 AQS。可以先记住这条主线：线程尝试修改同步状态，获取失败后进入等待队列，释放锁后唤醒后继线程；可重入则通过同步状态记录同一线程的重入次数。

## 常见追问

### ReentrantLock 为什么可以响应中断？

它提供 `lockInterruptibly()`，线程在等待锁的过程中可以响应中断，从而及时结束已经不再需要的等待。普通 `lock()` 的等待方式不能用同样的方式处理中断。

### ReentrantLock 如何实现超时获取？

可以使用 `tryLock()` 立即尝试，也可以使用带超时时间的 `tryLock(timeout, unit)`。获取失败时可以执行降级、重试或返回，而不是无限等待。

### ReentrantLock 和 synchronized 如何选择？

- 只需要简单互斥、自动释放和较少控制时，优先考虑 [[03-Synchronized是什么|Synchronized是什么]]。
- 需要超时、可中断、公平策略或多个 Condition 时，考虑 ReentrantLock。
- 选择 ReentrantLock 后，要把加锁和解锁范围控制清楚，并保证释放逻辑可靠。

## 相关笔记

- [[03-Synchronized是什么|Synchronized是什么]]
- [[01-Java 内存模型（JMM）是什么？|Java 内存模型（JMM）是什么？]]
- AQS
- [[死锁如何排查]]
