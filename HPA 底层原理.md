Kubernetes 里的 **Horizontal Pod Autoscaler（HPA）** 是用于 **根据负载自动扩缩 Pod 数量** 的控制器。它的底层本质是一个 **控制循环（control loop）+ 指标系统（metrics pipeline）**。

我从 **架构 → 工作流程 → 核心算法 → 关键组件** 四部分讲清楚。

---

# 一、HPA 架构（核心组件）

HPA 主要依赖 4 个组件：

```
                +--------------------+
                |  Metrics Server    |
                | (collect metrics)  |
                +---------+----------+
                          |
                          v
+-------------------+   API   +--------------------+
|  Kubernetes API   | <-----> |  HPA Controller    |
|  Server           |         | (control loop)     |
+---------+---------+         +---------+----------+
          |                             |
          v                             v
+-------------------+        +--------------------+
| Deployment / RS   |        |    Pods            |
| (scale target)    |        | CPU / Memory usage |
+-------------------+        +--------------------+
```

关键组件：

1️⃣ **Metrics Server**

- 从 kubelet 获取 metrics
- 提供 API：

```
/apis/metrics.k8s.io
```

例如：

```
kubectl top pod
```

就是从 metrics-server 来的。

---

2️⃣ **HPA Controller**

运行在 **kube-controller-manager** 里。

核心逻辑：

```
每15秒执行一次:
1 获取 metrics
2 计算 desired replicas
3 调整 Deployment replicas
```

---

3️⃣ **Scale Subresource**

HPA 不直接操作 Pod，而是修改：

```
Deployment
ReplicaSet
StatefulSet
```

的：

```
spec.replicas
```

通过 API：

```
/scale
```

---

# 二、HPA 工作流程

完整流程：

```
用户创建 HPA
      │
      ▼
HPA Controller 启动控制循环
      │
      ▼
每15s获取 metrics
      │
      ▼
计算 desired replicas
      │
      ▼
修改 Deployment.spec.replicas
      │
      ▼
ReplicaSet 创建/删除 Pod
```

流程图：

```
Metrics Server
     │
     ▼
CPU / Memory usage
     │
     ▼
HPA Controller
     │
     ▼
desired replicas
     │
     ▼
Deployment.spec.replicas
     │
     ▼
ReplicaSet
     │
     ▼
Pods scale up/down
```

---

# 三、HPA 核心算法（最关键）

HPA 的核心公式：

```
desiredReplicas = ceil(
    currentReplicas × currentMetric / desiredMetric
)
```

例子：

当前：

```
replicas = 3
CPU target = 50%
current CPU = 80%
```

计算：

```
desiredReplicas = ceil(3 × 80 / 50)
                = ceil(4.8)
                = 5
```

所以：

```
3 → 5 pods
```

---

### 再举一个 scale down

```
replicas = 10
CPU target = 50%
current CPU = 20%
```

```
desiredReplicas = ceil(10 × 20 / 50)
                = ceil(4)
                = 4
```

系统：

```
10 → 4 pods
```

---

# 四、Metrics 类型

HPA 支持三类 metrics：

---

## 1 Resource Metrics（最常用）

来自：

```
metrics-server
```

例如：

```
CPU
Memory
```

示例：

```yaml
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## 2 Custom Metrics

来自：

```
Prometheus Adapter
```

例如：

```
QPS
request latency
queue length
```

API：

```
custom.metrics.k8s.io
```

---

## 3 External Metrics

来自外部系统：

```
Kafka lag
SQS queue
Cloud metrics
```

API：

```
external.metrics.k8s.io
```

---

# 五、HPA 的稳定机制（防止抖动）

HPA 不会每次都立刻 scale。

有 **3 个稳定机制**。

---

### 1 tolerance

默认：

```
10%
```

如果变化小于 10% 不扩缩。

例如：

```
target = 50%
current = 53%
```

不会 scale。

---

### 2 stabilization window

避免频繁 scale down。

默认：

```
scaleDownWindow = 5 min
```

系统会取：

```
过去5分钟最大推荐值
```

---

### 3 scale rate limit

限制扩缩速度。

例如：

```
每分钟最多 +4 pods
```

---

# 六、HPA Controller 源码核心逻辑

核心代码在：

```
pkg/controller/podautoscaler/horizontal.go
```

核心函数：

```
reconcileAutoscaler()
```

伪代码：

```go
loop every 15s:

  currentReplicas = getCurrentReplicas()

  metrics = getMetrics()

  desiredReplicas = calculateReplicas()

  if desiredReplicas != currentReplicas:
        scale()
```

---

# 七、HPA v2 架构（现代版本）

HPA v2 支持：

```
multiple metrics
```

例如：

```
CPU
AND
QPS
```

系统会：

```
取最大值
```

例子：

```
CPU -> 5 pods
QPS -> 8 pods
```

最终：

```
replicas = 8
```

---

# 八、HPA 最大问题

HPA 最大缺点是：

### 冷启动问题

Pod 启动慢：

```
metrics升高
→ 扩容
→ pod启动30s
→ 请求已经超时
```

解决方法：

1️⃣ **预留 buffer**

```
targetCPU = 40%
```

2️⃣ **结合 HPA + VPA**

- HPA：scale pod
- VPA：scale resource

3️⃣ **使用 KEDA**

---

# 九、生产架构（真实公司）

真实生产通常是：

```
Prometheus
     │
     ▼
Prometheus Adapter
     │
     ▼
Custom Metrics API
     │
     ▼
HPA
     │
     ▼
Deployment
```

例如根据：

```
QPS
Kafka lag
Queue size
```

自动扩容。

---

# 十、一句话总结

HPA 本质是：

```
Metrics → Control Loop → Desired Replicas → Scale
```

核心公式：

```
desiredReplicas =
ceil(currentReplicas × currentMetric / targetMetric)
```

---

如果你在准备 **云原生 / 系统设计 / AI infra 面试**，我可以再给你讲三个更硬核的东西（面试官很爱问）：

1️⃣ **HPA + KEDA 的架构原理**
2️⃣ **为什么 HPA 会 scale 抖动（flapping）**
3️⃣ **如何设计一个 Kubernetes HPA 替代系统（system design）**
