幂等性（Idempotency）在分布式系统里非常重要，尤其是你这种做高并发 + 微服务 + 云原生架构的场景。

> **幂等性 = 同一个请求执行一次和执行多次，结果一致**

在 Golang 后端系统里，幂等性通常用于：

- 支付回调
- 下单接口
- 消息队列重复消费
- 重试机制（HTTP retry / gRPC retry）
- 分布式任务执行

---

# 一、常见幂等问题场景

### ❌ 典型问题

```text
用户点击两次“提交订单”
↓
生成两条订单
↓
扣两次钱
```

或者：

```text
MQ 重复投递
↓
库存扣减两次
```

---

# 二、幂等性设计核心思路

## 方案一：唯一约束（最简单、最可靠）

### ✅ 数据库唯一索引

这是最推荐的方式（强一致场景）

```sql
ALTER TABLE orders ADD UNIQUE KEY uniq_order_no (order_no);
```

Golang 代码：

```go
err := db.Create(&order).Error
if err != nil {
    if errors.Is(err, gorm.ErrDuplicatedKey) {
        return nil // 幂等返回成功
    }
    return err
}
```

### ✔ 优点

- 强一致
- 实现简单
- 无额外系统依赖

### ❗适用

- 下单
- 支付单
- 交易流水

---

## 方案二：幂等 Token（推荐 Web 接口使用）

适用于：

- 用户点击类接口
- 防止前端重复提交

### 流程

1. 客户端请求生成 token
2. 服务端保存 token（Redis）
3. 客户端携带 token 请求
4. 服务端验证 token 是否已使用

---

### 示例（Redis）

```go
func CheckIdempotent(token string) bool {
    key := "idempotent:" + token

    // SETNX
    ok, _ := rdb.SetNX(ctx, key, 1, time.Minute*10).Result()
    return ok
}
```

使用：

```go
if !CheckIdempotent(token) {
    return errors.New("duplicate request")
}
```

---

### ✔ 优点

- 非侵入数据库
- 性能高
- 适合高并发

### ❗问题

- 依赖 Redis
- 需要过期策略

---

## 方案三：分布式锁（高并发扣库存）

使用：

- Redis Lock
- etcd
- ZooKeeper

例如 Redis：

```go
lock := redislock.New(client)

lockKey := "lock:order:" + orderID

l, err := lock.Obtain(ctx, lockKey, 5*time.Second, nil)
if err != nil {
    return errors.New("request is processing")
}
defer l.Release(ctx)
```

适用于：

- 同一资源强互斥
- 秒杀
- 库存扣减

---

## 方案四：去重表（高可靠金融场景）

专门设计幂等表：

```sql
CREATE TABLE idempotent_record (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    biz_key VARCHAR(64) UNIQUE,
    created_at DATETIME
);
```

Golang：

```go
err := db.Exec("INSERT INTO idempotent_record(biz_key) VALUES(?)", key).Error
if err != nil {
    return nil // 已存在，说明处理过
}
```

---

# 三、MQ 消费幂等（重点）

在分布式系统中，MQ 必然会重复投递。

## 正确做法：

### 1️⃣ 消费前去重

```go
if Exist(msg.ID) {
    return nil
}
```

### 2️⃣ 业务逻辑必须可重复执行

错误做法：

```go
balance -= 100
```

正确做法：

```go
UPDATE account
SET balance = balance - 100
WHERE id = ? AND balance >= 100;
```

必须保证：

- 原子操作
- 条件更新

---

# 四、不同场景最佳方案总结

| 场景         | 推荐方案            |
| ------------ | ------------------- |
| 下单         | DB 唯一索引         |
| 支付回调     | 唯一索引 + 状态机   |
| 用户重复提交 | 幂等 token          |
| MQ 重复消费  | 去重表 / 唯一索引   |
| 秒杀         | 分布式锁 + 原子扣减 |
| 定时任务     | 任务执行记录表      |

---

# 五、进阶设计（高级架构）

在大型系统中常用：

### 1️⃣ 状态机设计（强烈推荐）

```text
INIT → PROCESSING → SUCCESS → FAILED
```

更新必须：

```sql
UPDATE order
SET status='SUCCESS'
WHERE order_no=? AND status='PROCESSING';
```

确保状态单向流转。

---

### 2️⃣ 幂等 + 重试 + 补偿

幂等不是为了避免错误，而是为了：

> 安全重试

比如：

- gRPC retry
- HTTP retry
- MQ re-consume
- 分布式事务补偿

---

# 六、你这个背景建议（架构级）

你做的是：

- 高并发
- 分布式
- 云原生
- 微服务

建议标准化一个：

## Idempotency Middleware

### 统一：

- Header: X-Idempotency-Key
- Redis 校验
- 自动封装响应缓存

类似 Stripe 的设计。

---

# 七、企业级幂等最佳实践（总结）

✅ 核心原则：

1. 业务必须可重试
2. 状态流转必须单向
3. 数据修改必须原子
4. 所有写操作必须可幂等
5. 不能依赖“不会重复”

---

# 八、一句话总结

> 幂等性 = 允许系统安全重试的能力
> 本质是为了对抗分布式系统的不可靠性

---

如果你愿意，我可以给你：

- ✅ 画一张完整分布式幂等架构图
- ✅ 写一个生产级 Golang 幂等中间件实现
- ✅ 讲大厂支付系统如何设计幂等
- ✅ 结合 Kubernetes + MQ 做完整示例

你想深入哪块？
