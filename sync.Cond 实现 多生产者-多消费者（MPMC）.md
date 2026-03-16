在 **Go** 中，用 `sync.Cond` 实现 **多生产者-多消费者（MPMC）** 的核心思想是：

- 共享一个 **buffer / queue**
- 用 **mutex** 保护共享数据
- 用 **两个条件变量** 控制状态：
  - `notEmpty` → 队列非空，消费者可以取
  - `notFull` → 队列未满，生产者可以放

这样可以避免：

- busy wait
- 资源竞争
- producer / consumer starvation

---

# 1 基本结构

```go
type Queue struct {
    buf      []int
    capacity int

    mu       sync.Mutex
    notEmpty *sync.Cond
    notFull  *sync.Cond
}
```

初始化：

```go
func NewQueue(cap int) *Queue {
    q := &Queue{
        buf:      make([]int, 0, cap),
        capacity: cap,
    }

    q.notEmpty = sync.NewCond(&q.mu)
    q.notFull = sync.NewCond(&q.mu)

    return q
}
```

---

# 2 Producer（生产者）

逻辑：

1. 如果队列 **满了** → 等待 `notFull`
2. 放入数据
3. 通知消费者 `notEmpty`

```go
func (q *Queue) Put(x int) {
    q.mu.Lock()

    for len(q.buf) == q.capacity {
        q.notFull.Wait()
    }

    q.buf = append(q.buf, x)

    q.notEmpty.Signal()

    q.mu.Unlock()
}
```

关键点：

```
for len(buf) == capacity
    Wait()
```

必须是 **for**，不能是 `if`。

原因：

- spurious wakeup
- 多 goroutine 竞争

---

# 3 Consumer（消费者）

逻辑：

1. 如果队列 **空** → 等待 `notEmpty`
2. 取数据
3. 通知 producer `notFull`

```go
func (q *Queue) Get() int {
    q.mu.Lock()

    for len(q.buf) == 0 {
        q.notEmpty.Wait()
    }

    v := q.buf[0]
    q.buf = q.buf[1:]

    q.notFull.Signal()

    q.mu.Unlock()

    return v
}
```

---

# 4 完整示例

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Queue struct {
	buf      []int
	capacity int

	mu       sync.Mutex
	notEmpty *sync.Cond
	notFull  *sync.Cond
}

func NewQueue(cap int) *Queue {
	q := &Queue{
		buf:      make([]int, 0, cap),
		capacity: cap,
	}
	q.notEmpty = sync.NewCond(&q.mu)
	q.notFull = sync.NewCond(&q.mu)
	return q
}

func (q *Queue) Put(x int) {
	q.mu.Lock()

	for len(q.buf) == q.capacity {
		q.notFull.Wait()
	}

	q.buf = append(q.buf, x)

	q.notEmpty.Signal()

	q.mu.Unlock()
}

func (q *Queue) Get() int {
	q.mu.Lock()

	for len(q.buf) == 0 {
		q.notEmpty.Wait()
	}

	v := q.buf[0]
	q.buf = q.buf[1:]

	q.notFull.Signal()

	q.mu.Unlock()

	return v
}

func main() {
	q := NewQueue(5)

	// producers
	for i := 0; i < 3; i++ {
		go func(id int) {
			for {
				q.Put(id)
				fmt.Println("produce", id)
				time.Sleep(time.Millisecond * 500)
			}
		}(i)
	}

	// consumers
	for i := 0; i < 2; i++ {
		go func(id int) {
			for {
				v := q.Get()
				fmt.Println("consume", id, v)
				time.Sleep(time.Second)
			}
		}(i)
	}

	select {}
}
```

---

# 5 运行逻辑

假设：

```
capacity = 2
```

流程：

```
P1 put -> [1]
P2 put -> [1,2]
P3 put -> full -> wait(notFull)

C1 get -> [2]
      -> signal(notFull)

P3 wake -> put -> [2,3]
```

---

# 6 为什么用两个 Cond

如果只用一个：

问题：

```
producer signal -> 唤醒 producer
consumer signal -> 唤醒 consumer
```

可能：

```
producer wake
queue still full
→ 又 sleep
```

造成 **无效唤醒**。

所以通常设计：

```
notEmpty
notFull
```

---

# 7 高性能系统里的写法

在高并发系统里常见写法：

```
Producer
while queue full
    notFull.Wait()

Consumer
while queue empty
    notEmpty.Wait()
```

Signal 顺序：

```
put -> signal(notEmpty)
get -> signal(notFull)
```

---

# 8 sync.Cond vs channel

Go 官方通常推荐：

```
chan
```

但 `sync.Cond` 更适合：

| 场景                   | 推荐      |
| ---------------------- | --------- |
| 简单 producer consumer | channel   |
| 复杂共享状态           | sync.Cond |
| 高性能队列             | sync.Cond |
| 需要多个条件           | sync.Cond |

---

💡 **面试高频总结**

`sync.Cond` 实现 MPMC：

```
mutex + two condition variables

notEmpty → consumer wait
notFull  → producer wait
```

核心原则：

```
Wait 必须在 for 循环
Wait 自动 unlock
被唤醒后重新 lock
```

---

如果你愿意，我可以再给你一个 **生产级 MPMC ring buffer（无 slice 移动，O(1) 队列）** 的 `sync.Cond` 实现，这是 **很多 Go 高并发系统（调度器 / worker pool）常用写法**。
