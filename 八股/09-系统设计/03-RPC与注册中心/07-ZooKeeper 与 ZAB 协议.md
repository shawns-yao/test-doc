---
aliases:
- "ZooKeeper 与 ZAB 协议？"
- "系统设计 3.7 ZooKeeper 与 ZAB 协议？"
---

# 07 ZooKeeper 与 ZAB 协议？

## 01 核心回答

**角色与数据模型：**集群角色三种——**Leader**（负责写）、**Follower**（同步数据并提供读）、Observer（不参与投票，可接收客户端读写请求；写请求转交投票集群处理）。数据模型是 znode 树：**持久节点**适合放元数据；临时节点在会话过期或会话关闭后移除，短暂 TCP 断开不等于会话过期，适合分布式锁、服务在线状态。客户端与 ZK 建 TCP 长连接（session），sessionTimeout 内重连可延续原 session。

**Watcher：**对 znode 注册监听，节点变化时收到回调通知——分布式锁的羊群效应规避就靠监听前驱节点。

**ZAB 协议：**「带顺序保证的主从同步协议」——Leader 收到事务请求先写本地日志，把 proposal 按顺序同步给 follower；follower 先写磁盘再返回 ack；Leader 收到过半 ack 后发出 commit；follower 收到 commit 后才让新数据可见。关键点：proposal 带全局递增的 `zxid` 保证顺序；过半写机制；Leader 崩溃后需要足够投票成员可互通，并完成选主和历史同步，才能恢复推进。

**一致性定位：**ZK 写入是线性一致的，普通本地读可能陈旧，并提供顺序相关保证；不能把整个系统简化为最终一致。`sync()` 后读也不应无条件宣称为严格线性一致读。

---

## 02 ZAB 为什么需要 epoch 与恢复阶段

多数派确认让已提交事务与未来选主集合相交，zxid 的 epoch 区分不同 Leader 任期，计数器给同任期事务排序。新 Leader 不能一选出就随意写，而要建立正确历史并同步足够参与者，避免已经提交的记录消失或旧 Leader 的未决历史乱入。

故障容忍数按投票成员和 quorum 配置计算，Observer 不计入普通多数派。ZAB 是原子广播/复制协议，不能因为广播阶段看起来有 proposal/commit 就等同于任意跨数据库事务的 2PC。

---

## 03 会话 Watch 与读语义的坑

连接断开时客户端应进入不确定状态，不再凭旧锁身份执行危险写入；会话过期后重新连接获得的新会话不继承旧临时节点。即使锁节点被删除，暂停中的旧进程也可能恢复并访问外部资源，因此关键写入还需 fencing token 等下游校验。

传统 Watch 是一次性通知，需要重新注册；新版本也有持久 Watch，必须指明使用方式。通知意味着数据可能变化，不是完整事件日志；断连期间重连应读取当前状态并核对版本。

官方 Internals 明确普通读不是 quorum 操作，sync 当前也并非 quorum 操作，因此不提供无条件最新读保证。需要严格读语义时应按文档验证适当的 quorum 操作和协议，不机械背“sync 后就强一致”。

口述：“ZAB 保证复制写历史顺序和故障恢复，ZooKeeper 的写、普通读与会话 Watch 各有不同语义。临时节点看会话过期，Observer 不投票，外部资源安全不能只靠拿到过一把锁。”

---

## 04 依据与关联问题

- [ZooKeeper Programmer Guide](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html)
- [ZooKeeper Internals 精确一致性保证](https://zookeeper.apache.org/doc/current/zookeeperInternals.html)
- [ZooKeeper Observer 指南](https://zookeeper.apache.org/doc/current/zookeeperObservers.html)
- [[八股/09-系统设计/03-RPC与注册中心/03-服务注册中心的核心职责是什么|注册中心租约]]
- [[八股/09-系统设计/01-高并发与可用性/16-分布式服务接口请求的顺序性如何保证|分布式请求顺序]]

- [[八股/09-系统设计/02-一致性与高可用/11-Raft 如何保证选主与日志复制安全|Raft与ZAB的比较入口]]
- [[八股/09-系统设计/01-高并发与可用性/19-如何设计可靠的分布式定时任务调度系统|如何设计可靠的分布式定时任务调度系统]]

---

## 05 所属专题

- [[八股/09-系统设计/03-RPC与注册中心/00-RPC与注册中心导航|RPC与注册中心导航]]
