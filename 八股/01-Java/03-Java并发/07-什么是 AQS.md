---
aliases:
- "什么是 AQS？"
- "Java 2.7 什么是 AQS？"
---

# 07 什么是 AQS？

## 01 核心回答

**概念原理：**AQS 是 JUC 的同步器框架，核心是 state + CLH 等待队列 + CAS + park/unpark 唤醒——把并发控制从「每个锁自己造轮子」变成「只实现状态规则，其余排队唤醒交给框架」。

**核心组成：**`state` 一个 int 状态位（锁是否被占用、剩余许可数）；等待队列是 CLH 变种双向链表，抢不到资源的线程排队；CAS 原子修改 state；`park/unpark` 挂起与唤醒线程。

**独占锁工作流程：**线程先 CAS 抢 state → 抢到进临界区 → 抢不到入队并 park 挂起 → 持有线程释放后 unpark 队头线程 → 被唤醒线程再尝试 CAS 抢锁。

**实现方式：**只需重写 `tryAcquire/tryRelease`（独占）或 `tryAcquireShared/tryReleaseShared`（共享）——AQS 负责排队、阻塞、唤醒，你只定义规则。

**基于 AQS 的组件：**

① **ReentrantLock**：state = 重入次数，同一时刻只允许一个线程持有。
② **Semaphore**：state = 剩余许可数，获取 -1、释放 +1，最多 N 个线程同时通过。
③ **CountDownLatch**：state = 剩余计数，到 0 一次性放开，不可重置（重置用 CyclicBarrier/Phaser）。
④ **ReentrantReadWriteLock**：state 高低位分别编码读锁和写锁计数。

---

## 02 AQS 的模板与业务规则

AQS 提供一个 volatile int state、同步队列和排队唤醒算法；子类定义 state 的意义以及获取/释放成功条件。它是同步器框架，不是某一把锁，也不默认要求 FIFO 公平获取。实现 tryAcquire 时可以先抢资源，也可以检查前驱等待者后再尝试。

独占成功通常只有一个线程继续；共享成功还可能允许后续线程继续尝试。CountDownLatch 的归零是一次性开闸，Semaphore 的许可可以重复释放获取，所以二者虽然都用共享模式，状态语义完全不同。

---

## 03 等待队列常见误区

同步队列与 Condition 队列不是一个队列。await 先释放同步状态进入条件等待，通知后重新进入获取同步状态的流程。park 可能虚假返回，中断或超时也可能取消等待，所以等待必须循环检查，节点清理和唤醒都不能等同于“把锁直接交给下一人”。

---

## 04 面试口述

AQS 让同步器只关心何时可获取、何时可释放，框架负责失败后的排队、阻塞、取消与唤醒。解释 ReentrantLock、Semaphore、CountDownLatch 时分别说清 state 的含义，比只背 CLH 名称更重要。

---

## 05 官方参考

- [AbstractQueuedSynchronizer 官方设计说明](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/AbstractQueuedSynchronizer.html)
- [[八股/01-Java/03-Java并发/04-ReentrantLock是什么|用 ReentrantLock 理解独占模式与 Condition]]

---

## 06 所属专题

- [[八股/01-Java/03-Java并发/00-Java并发导航|Java并发导航]]
