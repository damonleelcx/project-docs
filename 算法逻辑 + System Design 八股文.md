“**算法逻辑 + System Design 八股文**”通常指面试中需要**背熟、结构化回答的一套套路**。在技术面试（特别是大厂/互联网公司）里，面试官通常希望看到 **清晰的思考框架 + 常见设计模式**。下面我给你整理一份**高频八股文框架**，算法和系统设计都适用。

---

# 一、算法题「八股文」回答模板

面试时建议 **固定 5 步结构**：

### 1️⃣ Clarify Question（澄清问题）

先确认题意：

- 输入规模？
- 是否允许重复？
- 是否需要最优解？
- 是否有负数 / 边界情况？

例子：

```
Input size?
Are there duplicates?
What should I return if no solution exists?
```

---

### 2️⃣ Brute Force（暴力解法）

面试官非常看重这一步。

结构：

- 最简单思路
- 时间复杂度
- 为什么不够好

例子：

```
Brute force solution:
Check every pair.

Time complexity: O(n^2)
Space complexity: O(1)

But this is too slow when n is large.
```

---

### 3️⃣ Optimization（优化思路）

说明 **关键 insight**。

常见套路：

| 技巧           | 关键词     |
| -------------- | ---------- |
| HashMap        | 空间换时间 |
| Two Pointers   | 排序数组   |
| Sliding Window | 子数组问题 |
| Binary Search  | 单调性     |
| Heap           | TopK       |
| DFS/BFS        | 图/树      |
| DP             | 最优子结构 |

示例：

```
We can use a HashMap to store previous values.
This reduces time complexity to O(n).
```

---

### 4️⃣ Algorithm（算法步骤）

清晰步骤：

```
1. Initialize hashmap
2. Iterate through array
3. Check if target - num exists
4. Return indices
```

---

### 5️⃣ Complexity Analysis（复杂度分析）

标准回答：

```
Time Complexity: O(n)
Space Complexity: O(n)
```

---

### 算法题完整模板

面试时可以按这个说：

```
1. Clarify the problem
2. Discuss brute force
3. Identify optimization
4. Explain algorithm
5. Analyze complexity
6. Walk through example
7. Write code
```

---

# 二、System Design 八股文（超高频）

系统设计面试其实也有套路。

经典结构：

```
1 Clarify requirements
2 Estimate scale
3 High level design
4 Core components
5 Data model
6 Scaling
7 Bottleneck
8 Tradeoffs
```

---

# 1️⃣ Clarify Requirements（需求澄清）

问清：

### Functional Requirements

系统要做什么？

例：

- 用户发帖
- 点赞
- 搜索

### Non-functional Requirements

- QPS
- Latency
- Availability
- Consistency

---

# 2️⃣ Capacity Estimation（容量估算）

常见面试题：

```
Daily active users
Requests per second
Storage
Bandwidth
```

例子：

```
10M users
1 request/sec per user

QPS ≈ 10M
```

---

# 3️⃣ High Level Design（高层架构）

画架构图：

典型架构：

```
Client
   ↓
Load Balancer
   ↓
API Servers
   ↓
Cache
   ↓
Database
   ↓
Message Queue
```

组件：

| 组件          | 作用         |
| ------------- | ------------ |
| Load Balancer | 分流         |
| API Server    | 业务逻辑     |
| Cache         | 降低 DB 压力 |
| DB            | 持久化       |
| Queue         | 异步         |

---

# 4️⃣ Data Model（数据模型）

例如：

用户表

```
User
-----
id
name
email
```

帖子表

```
Post
-----
id
user_id
content
timestamp
```

---

# 5️⃣ Scaling（扩展）

高频八股：

### Cache

```
Redis
Memcached
```

用途：

- 减少 DB 压力
- 提升 latency

---

### Database Sharding

按：

- user_id
- hash
- region

---

### Replication

```
Master → Write
Slave → Read
```

读写分离。

---

# 6️⃣ Bottleneck（瓶颈）

常见瓶颈：

| 组件    | 问题    |
| ------- | ------- |
| DB      | QPS瓶颈 |
| Cache   | 热key   |
| Network | 带宽    |
| Queue   | 积压    |

---

# 7️⃣ Reliability（可靠性）

高频八股：

### Circuit Breaker

防止雪崩。

### Retry

失败重试。

### Idempotency

幂等。

---

# 8️⃣ Tradeoffs（取舍）

必须提：

### CAP theorem

CAP theorem

```
Consistency
Availability
Partition tolerance
```

分布式系统只能保证其中两个。

---

# 三、常见 System Design 面试题

高频题：

| 题目                 | 核心知识    |
| -------------------- | ----------- |
| Design Twitter       | Feed fanout |
| Design YouTube       | 视频存储    |
| Design Rate Limiter  | Redis       |
| Design URL Shortener | hash        |
| Design Chat System   | WebSocket   |
| Design Uber          | GeoHash     |

---

# 四、系统设计「万能回答套路」

可以直接背：

```
1. Clarify requirements
2. Estimate scale
3. High level architecture
4. Data model
5. APIs
6. Scaling strategies
7. Bottlenecks
8. Tradeoffs
```

---

# 五、算法 + System Design 面试思维

大厂面试其实看三件事：

### 1 结构化思考

说话必须有框架。

### 2 Tradeoff

没有绝对最优设计。

### 3 Communication

边想边说。

---

# 六、如果你想走「分布式系统 / infra / backend」

你最近在研究：

- Docker
- Kubernetes
- Raft
- gRPC

这些非常适合 **system design面试**。

建议重点准备：

```
一致性
缓存
分布式锁
限流
消息队列
微服务
```

---

如果你愿意，我可以再给你整理一份 **大厂最常问的 20 个 System Design 八股文（含标准答案）**，比如：

- Rate Limiter
- Distributed Lock
- Feed System
- Chat System
- Distributed Cache
- API Gateway

基本就是 **Google / Meta / ByteDance / Amazon 面试题库**。
