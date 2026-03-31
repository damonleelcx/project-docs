Kafka 想做到“零丢失”（严格来说是 **尽可能接近 0 丢失**），本质是端到端保证：**生产端 → Broker → 消费端** 每一层都不能出问题。你需要同时配置好这三段，否则任何一段都可能丢数据。

我给你从工程实战角度讲清楚👇

---

# 一、生产端（Producer）：防止“没写进去就丢了”

### 1️⃣ 必开：ACK = all

```properties
acks=all
```

含义：

- 必须等 **ISR（同步副本）全部写成功** 才算成功
- leader 挂了也不会丢

---

### 2️⃣ 重试 + 不限次数

```properties
retries=Integer.MAX_VALUE
delivery.timeout.ms=120000
```

---

### 3️⃣ 防止重复：幂等生产者（关键！）

```properties
enable.idempotence=true
```

Kafka 会保证：

- 重试不会产生重复消息
- 避免“以为失败其实成功”导致重复发送

---

### 4️⃣ 避免乱序（重要）

```properties
max.in.flight.requests.per.connection=1
```

否则：

- 重试可能导致消息乱序

---

# 二、Broker（Kafka 集群）：防止“写进去又丢了”

### 1️⃣ 副本数 ≥ 3

```properties
replication.factor=3
```

---

### 2️⃣ ISR 最小副本数

```properties
min.insync.replicas=2
```

配合：

```properties
acks=all
```

👉 解释：

- 至少 2 个副本成功才算写成功
- 避免只写 leader 就挂掉

---

### 3️⃣ 禁止脏 Leader 选举（关键）

```properties
unclean.leader.election.enable=false
```

否则：

- follower 没同步完也能当 leader → 数据直接丢

---

### 4️⃣ 刷盘策略（是否 fsync）

默认 Kafka：

- 依赖 OS page cache（性能好，但理论上有风险）

如果你要极致安全：

```properties
log.flush.interval.messages=1
```

⚠️ 但：

- 性能会暴跌（通常不建议）

---

# 三、消费者（Consumer）：防止“读到了但没处理成功”

### 1️⃣ 关闭自动提交

```properties
enable.auto.commit=false
```

---

### 2️⃣ 手动提交 offset（处理完再提交）

```go
// pseudo code
process(msg)
commitOffset()
```

---

### 3️⃣ 处理失败必须重试

否则：

- offset 提交了 → 数据永远丢

---

### 4️⃣ 幂等消费（去重）

即使 Kafka 不丢数据，也可能：

- 消费重复（at-least-once）

解决：

- 数据库唯一键
- Redis 去重
- 业务幂等

---

# 四、进阶：真正“接近零丢失”的方案

## ✅ 1️⃣ 事务（Exactly Once）

```properties
enable.idempotence=true
transactional.id=xxx
```

Kafka 支持：

- **EOS（Exactly Once Semantics）**

👉 保证：

- 生产 + 消费 + 处理 一致

---

## ✅ 2️⃣ 消费 + 生产事务（流处理）

典型场景：

- Kafka → 处理 → Kafka

使用：

- Kafka Streams
- 或手动事务

---

## ✅ 3️⃣ 外部存储 + Kafka 双写一致性

常见方案：

- 本地事务表（Outbox Pattern）
- CDC（Debezium）

---

# 五、总结（最关键配置组合）

直接给你一套“工业级配置”👇

### Producer

```properties
acks=all
enable.idempotence=true
retries=Integer.MAX_VALUE
max.in.flight.requests.per.connection=1
```

### Broker

```properties
replication.factor=3
min.insync.replicas=2
unclean.leader.election.enable=false
```

### Consumer

```properties
enable.auto.commit=false
```

---

# 六、现实结论（很重要）

👉 Kafka **不能 100% 绝对零丢失**，因为：

- 磁盘损坏
- 机房级灾难
- 人为误操作

👉 但可以做到：

> **工程上“无限接近零丢失”**

---

# 七、一句话理解

> Kafka 的“零丢失” =
> **多副本 + 强一致写入 + 幂等 + 手动提交 + 不允许脏选举**

---

如果你要，我可以帮你补一版：
👉 **Kafka 面试标准答案（3分钟讲清零丢失）**
👉 或 **结合你 Golang 项目怎么落地配置（含代码）**
