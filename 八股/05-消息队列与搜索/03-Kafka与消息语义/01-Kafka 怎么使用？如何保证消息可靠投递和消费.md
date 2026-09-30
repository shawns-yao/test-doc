---
aliases:
- "Kafka 怎么使用？如何保证消息可靠投递和消费？"
- "消息队列 3.1 Kafka 怎么使用？如何保证消息可靠投递和消费？"
---

# 01 Kafka 怎么使用？如何保证消息可靠投递和消费？

## 01 核心回答


**概念原理：**Kafka 是分布式日志型消息系统：Topic 分 Partition，Partition 内有序、多副本（Leader/Follower），消费组内分区分配消费。使用：建 Topic → Producer 按 key 路由写分区 → Consumer 从分区拉取并提交 offset。

**生产端可靠性（投递不丢）：**① `acks=all`：Leader 等所有 ISR 副本确认才返回；② `retries` 重试 + `enable.idempotence=true` 幂等（防止重试重复写）；③ 跟踪发送结果；失败或结果不确定时依据稳定事件 ID 重试或持久化补偿。

**存储可靠性：**副本因子 ≥2（生产建议 3），ISR 同步；`min.insync.replicas` 控制最少同步副本数；broker 宕机由 Controller 重新选举 Leader。

**消费端可靠性：**① **先处理后提交** offset（业务成功再 commit），处理失败不提交可重新消费；② 至少一次语义下消费必须**幂等**（重复消费不可避免）；③ 提交时机权衡：处理前提交（快但丢消息）vs 处理后提交（不丢但可能重复）。

**关键细节：**Kafka 保证分区内有序，跨分区不保证——需要全局有序就单分区或按 key 路由；消费组再均衡行为取决于协议，处理过慢超过 `max.poll.interval.ms` 可能失去分区；`commitSync` 只是等待位点提交完成，不能保证业务不丢。

**风险与取舍：**acks=all + 同步副本降低吞吐（高可靠 vs 高吞吐权衡）；重复消费是常态，幂等是刚需；消息积压先查瓶颈，再在分区数、顺序要求和下游容量内扩消费者。

**项目落地：**日志/埋点管道（高吞吐场景）；消费端幂等表 + 处理后提交；监控 lag（消费积压）、ISR 状态和 broker 磁盘；关键业务事件用 acks=all + 幂等 Producer。

**面试追问：**acks=0/1/all 的区别；ISR 是什么（同步副本集合，落后则剔除）；为什么 Kafka 吞吐高（顺序写 + 页缓存 + 零拷贝）；rebalance 是什么（分区重新分配，会短暂停消费）。

---

## 02 可靠性需要三个边界同时成立

生产成功必须以发送结果为准，不能把 send 方法返回等同于 Broker 已接收。异步发送也可可靠，关键是处理 future/回调中的最终错误；超时往往代表结果不确定，不能推断服务端一定没写入。

以副本因子 3、min.insync.replicas=2、acks=all 为例：acks=all 等待当前 ISR 所需的确认，min ISR 则限制“允许接受写入时的同步副本下限”。只配 acks=all 而允许 ISR 缩到 1，保护力度会明显下降。它也不是每条记录在所有副本执行 fsync 的承诺。还需禁用会丢数据的非同步副本选主路径，并核对部署版本的选主机制。依据：[生产者配置](https://kafka.apache.org/41/configuration/producer-configs/)、[Topic 的 min.insync.replicas 和选主配置](https://kafka.apache.org/41/configuration/topic-configs/)。

消费成功要以业务结果为准：先提交位点再写数据库，宕机会漏处理；先写数据库再提交位点，宕机会重复处理。经典消费组并没有让这两个系统原子提交的魔法开关，正确做法通常是后者加业务幂等。

## 03 生产端 in-flight 和超时怎样配合幂等

以 Kafka 4.1 Java Producer 为例，显式启用 `enable.idempotence=true` 时要求 `acks=all`、`retries>0`、`max.in.flight.requests.per.connection<=5`；这些允许值下可保持同分区发送顺序。若关闭幂等、允许重试且 in-flight 大于 1，第一批失败重试而第二批先成功，就可能重排。不能一律把 in-flight 设为 1，也不能把幂等 Producer 当作应用重新调用 send 的业务去重器。

重试还须受总投递预算约束：`delivery.timeout.ms` 覆盖 send 返回后的排队、确认与可重试失败时间，应至少为 `request.timeout.ms + linger.ms`；send 自身等待缓冲/元数据的上限另看 `max.block.ms`。总预算耗尽后保留可重试业务事件及其稳定 ID，不能吞掉最终失败。批量、压缩和 linger 是吞吐/延迟调节，应在可靠性约束成立后压测。依据：[Kafka 4.1 Producer 配置](https://kafka.apache.org/41/configuration/producer-configs/)。

问题来源：[agent_java_offer 原题](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/02_后端/03_Kafka/01_核心问答.md#L37-L40)，Repository contributors，采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)。改动：补齐 in-flight 与幂等、重试、投递超时之间的具体约束。许可范围仅限本次引用/改编内容，不改变本页其他原有内容的许可。

## 04 位点提交的两个常见陷阱

- 提交的是“下一条待消费的位置”，不是刚处理的记录位置。按分区保存连续完成的进度，避免跳过较早但尚未处理成功的记录
- poll 返回一批消息后交给异步线程，若立即提交整批位置，就可能在后续线程失败时永久跳过任务。需要限制批量、跟踪每分区已连续完成区间，并在再均衡撤销分区时停止或收束在途工作

例：分区记录 100、101、102 中 102 先完成，但 101 未完成，不能仅因“最大完成 offset=102”就提交到 103。commitSync 即使成功，也只证明这个危险的位置被保存了。依据：[KafkaConsumer 手动位点控制与线程安全](https://kafka.apache.org/41/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html)。

## 05 按业务 key 分区后还要保证执行顺序

同 key 进入同一分区，只保证其日志与交付顺序；如果消费线程把两条订单事件交给任意工作线程并行执行，后一条仍可能先提交业务结果。可按分区建立串行任务队列，或按业务 key 稳定哈希到固定的本地串行队列；不同队列并行，同一队列一次只执行一个任务。队列和下游并发必须有上限。

按 key 并行后，同分区不同 key 的完成顺序可能变化，因此仍需按分区维护无未完成缺口的提交边界。“安全提交位点”和“同实体副作用顺序”是两个约束，前者不能替代后者。重试也要保持实体顺序：前一个事件失败时，不能先执行依赖它的后一个事件再回补。

增加 Kafka 分区或改变本地队列数量，都可能改变哈希映射；应在旧任务排空后切换，或用稳定路由版本和业务序列校验衔接，不能直接边跑边改。多个生产者同时写同一个 key 时，Broker 的到达顺序也未必等于业务因果顺序，必要时由上游定义事件版本。线程模型依据：[KafkaConsumer 多线程处理](https://kafka.apache.org/41/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html)。

本节问题线索来自 [agent_java_offer Kafka 顺序性补充](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/02_后端/03_Kafka/01_核心问答.md)，Repository contributors，[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)。改动：将按 key 本地排队补为顺序性追问，并加入位点、重试和路由变更边界；许可说明只针对本次引用/改编，不改变其他原有内容。

## 06 故障演练与口述

至少验证发送确认超时、Leader 故障、数据库提交后进程退出、消费者长暂停、再均衡以及积压超过保留期。指标同时看发布错误、ISR、最老待处理事件年龄、位点和业务对账，不能只说“Kafka 有三副本所以不丢”。

口述：“我把可靠性拆成生产、存储和消费。生产跟踪确认并启用幂等，存储用合理副本和 min ISR，消费在业务成功后提交连续位点并做幂等。超时和宕机仍会带来不确定结果，消息保留期和补偿对账也是方案的一部分。”

## 07 相关问题与延伸

- [[八股/05-消息队列与搜索/03-Kafka与消息语义/02-如何设计一个消息队列系统？以 Kafka 为例说明核心组件和链路|如何设计一个消息队列系统？以 Kafka 为例说明核心组件和链路]]：可靠投递与整体组件链路
- [[八股/05-消息队列与搜索/03-Kafka与消息语义/03-至少一次与恰好一次怎么选？消费幂等如何做|至少一次与恰好一次怎么选？消费幂等如何做]]：投递消费保证与端到端边界

- [[八股/05-消息队列与搜索/03-Kafka与消息语义/09-Kafka 消费者如何优雅停机|Kafka 消费者如何优雅停机]]：将投递可靠性落实到停止拉取、在途收束与分区安全位点

## 08 所属专题

- [[八股/05-消息队列与搜索/03-Kafka与消息语义/00-Kafka与消息语义导航|Kafka与消息语义导航]]
