---
aliases:
- "Kafka 的消息去重分哪三层？"
- "消息队列 3.6 Kafka 的消息去重分哪三层？"
---

# 06 Kafka 的消息去重分哪三层？

## 01 核心回答

**回答（三层理解）：**

**① 生产端去重**
`enable.idempotence=true` + acks=all：Kafka 用生产者 ID、epoch 和分区内序号识别重试批次，防网络重试重复写入。

**② Kafka 内部端到端**
"消费一条再生产一条"链路将输出记录与输入 offset 放在同一事务（transactional.id），下游 read_committed，才构成 Kafka 到 Kafka 的原子处理边界。

**③ 业务幂等（最易忽略）**
消费后写 DB/发券/扣库存时，Kafka 本身不能替业务去重——消息带全局唯一 eventId，消费端将幂等记录与业务变更原子提交，已成功处理的重复事件才可跳过；坚持"处理成功再提交 offset"。

**追问：**事务消息的代价（性能 + 协调器）；为什么业务幂等必须做（Kafka 只管自己内部）；幂等表怎么防并发（唯一索引兜底）。

---

## 02 三层分别识别哪一种重复

第一层识别协议重试：同一次 send 的批次确认丢失后，Producer 重发，Broker 能用协议序号判断是否已写入。应用主动再次 send 相同内容、业务系统重新生成事件，或独立生产者发送同一业务事实，不会因为内容相同就自动被去重。

第二层识别处理提交边界：输入记录可能因崩溃重新执行，但已中止的事务输出对 read_committed 消费者不可见。只有输出与消费位点原子提交，才能防止“输出成功但位点没提交”形成可见的重复结果。仅配置 transactional.id 加 read_committed，漏掉 sendOffsetsToTransaction，仍然有窗口。

第三层识别业务操作：数据库唯一约束、条件更新或外部接口的幂等键判断“这个操作是否已成功”，不依赖它经由哪一个 Kafka offset 到达。三层是分析框架，不是 Kafka 自动提供的三个配置按钮。

---

## 03 失败例子与事务身份

假设读 offset 20 后写输出 Topic，再用普通 commitSync 提交输入位点。如果输出已提交、进程却在位点提交前宕机，重启后会再次处理 20。正确的 Kafka 内部方案是将输出与下一输入位点放入同一事务；失败时中止事务并重新处理。

transactional.id 应能标识一个逻辑生产任务，并在恢复时延续其身份；不同活跃任务不能随意共用同一 ID，否则会发生 fencing。反过来，每次随机生成新 ID 也可能失去对旧实例的正确隔离。具体生命周期遵循所用客户端或框架。

若同一事务里还发了短信，中止 Kafka 事务不会撤回短信。因此外部操作仍要独立设计幂等与补偿；log compaction 按 key 清理旧日志也不等于阻止消费者看到重复。

---

## 04 口述与依据

“我先问重复从哪里来。协议重试用幂等生产者；Kafka 读写链路用输出和位点同事务；外部业务副作用用业务幂等键。三者处理不同故障窗口，不能说 enable.idempotence=true 就端到端恰好一次。”

- [幂等 Producer 配置约束](https://kafka.apache.org/41/configuration/producer-configs/)
- [KafkaProducer 事务、fencing 与 sendOffsetsToTransaction](https://kafka.apache.org/41/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html)
- [read_committed 消费隔离](https://kafka.apache.org/41/configuration/consumer-configs/)

---

## 05 相关问题与延伸

- [[八股/05-消息队列与搜索/03-Kafka与消息语义/03-至少一次与恰好一次怎么选？消费幂等如何做|至少一次与恰好一次怎么选？消费幂等如何做]]：恰好一次边界与分层去重

---

## 06 所属专题

- [[八股/05-消息队列与搜索/03-Kafka与消息语义/00-Kafka与消息语义导航|Kafka与消息语义导航]]
