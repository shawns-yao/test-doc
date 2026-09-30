---
aliases:
  - Synchronized是什么
---

# Synchronized 是什么？

## 一句话回答

这是 Java 提供的一个线程同步关键字。

## 核心原理


核心作用：只允许一个线程进入某个受保护的代码区域，并且线程之间能看到彼此对共享数据的修改
即为：给这一段操作加一把锁。谁先拿到锁，谁先执行；其他线程必须等。

所以解决的问题
	原子性/互斥，防止多个线程同时修改共享数据
	可见性：一个线程修改完共享数据之后，后续获取同一把锁的线程能够看到最新的结果

---

## 直观理解

可以把 synchronized 理解成排队，比如存在一个只有一把钥匙的房间
线程 A，B，C 都像进入这个房间，但是规则是拿到钥匙才能进去，假如 A 拿到了钥匙，A 进入，B 和 C 只能在外面等待

---

# 为什么需要 synchronized？

因为很多操作看起来是一句话，但是其实并不是一步完成的，
比如 count++，这其实是三个操作，如果两个线程同时执行这段代码，同时进行修改，最终结果可能加了两次



---

## synchronized 怎么解决这个问题？

把要执行的操作放到 synchronized 保护的范围内，并且这个期间 B 不能进来
需要等到 A 完成之后，然后 B 在拿到锁，
所以 synchronized 的核心是在执行操作的期间，不允许其他竞争同一把锁的线程进入，这个是保证原子性的方式

并且一定是强调是不是同一把锁，因为因为 synchronized 是否能互斥，关键不是有没有写 synchronized，而是两个线程抢的是不是同一个锁对象。

---

## synchronized 修饰普通方法时锁谁？

synchronized 锁的是对象，也就是 this，
比如有一个对象 account，线程 A 调用 account.add()，线程 B 也调用 account.add()，
因为操作的是同一个 account 对象，所以竞争的是同一个对象的锁，所以可以互斥


---

## static synchronized 又锁谁？

如果 synchronized 修饰的是静态方法，那么锁的就不是 this，因为静态方法没有 this，那么锁的是当前类对应的 Class 对象（也就是 Account.class）。


---

## synchronized 不只是保证原子性

synchronized 主要保证的是
	互斥：同一个时刻，只能有一个线程持有同一把锁
	可见性：前一个持锁线程在临界区里做的所有修改，下一个拿到**同一把锁**的线程一定能看到

## 常见追问

### synchronized 是否支持可重入？

支持。同一个线程已经持有某个对象的锁时，再次进入由同一把锁保护的同步方法或同步代码块，不需要重新等待。JVM 会记录重入次数，退出相应次数后才真正释放锁。

### synchronized 是公平锁吗？

synchronized 不提供公平锁和非公平锁的显式配置。线程竞争锁时的获取顺序由 JVM 和底层调度机制决定，不能依赖它实现严格的先来先得。

### synchronized 和 ReentrantLock 有什么区别？

两者都可以实现互斥和可重入，但关注点不同：

- synchronized 是 Java 关键字，使用简单，退出同步范围时自动释放锁。
- [[04-ReentrantLock是什么|ReentrantLock是什么]] 是 Java 类，基于 AQS，需要手动 `unlock()`，但支持可中断、超时获取、公平锁和多个 Condition。
- 简单的互斥场景优先考虑 synchronized；需要更细粒度控制时再考虑 [[04-ReentrantLock是什么|ReentrantLock是什么]]。

### synchronized 的锁升级过程是什么？

锁竞争较低时，JVM 会尽量使用更轻量的方式处理；竞争加剧时可能进入重量级监视器。具体锁实现和版本有关，复习时重点掌握“竞争越激烈，协调成本越高”的基本思路，不要脱离 JDK 版本死记实现细节。

## 相关笔记

- [[04-ReentrantLock是什么|ReentrantLock是什么]]
- [[02-Volatile|Volatile]]
- [[01-Java 内存模型（JMM）是什么？|Java 内存模型（JMM）是什么？]]
- [[05-ThreadLocal会造成内存泄露吗|ThreadLocal会造成内存泄露吗]]
