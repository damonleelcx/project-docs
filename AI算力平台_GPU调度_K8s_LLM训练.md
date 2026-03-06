这个岗位（**AI算力平台 / GPU调度 / K8s / LLM训练**）其实是 **云原生 + 分布式系统 + AI Infra** 的结合。
你的简历 **非常贴近岗位**（Go / K8s / Kafka / 云原生 / LLM integration），面试基本会围绕：

1️⃣ **云原生与Kubernetes底层**
2️⃣ **分布式系统设计**
3️⃣ **算力调度 / GPU资源管理**
4️⃣ **LLM / AI基础设施理解**
5️⃣ **项目深挖**

我帮你整理了一套 **大疆这类岗位最可能问的 40+ 面试题 + 思路答案**。

---

# 一、自我介绍（必问）

### Q1 请做一个自我介绍

核心结构（30秒）：

**背景 → 技术 → 项目 → 和岗位匹配**

示例：

> 我叫李承熙，目前是PPL的高级Golang工程师，有6年后端开发经验，主要方向是云原生和分布式系统。
>
> 过去几年我主要做两件事情：
>
> 第一是 **云原生微服务架构设计**，我主导把核心系统从单体拆成20多个Golang微服务，并部署在AWS EKS上，通过Kafka构建事件驱动架构，系统吞吐提升3倍。
>
> 第二是 **AI能力接入和平台化**，我设计过一个基于LLM的智能语音流程引擎，通过Golang流程编排层调用Bedrock实现意图识别和任务路由。
>
> 我对 **Kubernetes资源调度、分布式系统可靠性、AI基础设施**比较感兴趣，所以看到大疆这个算力平台岗位非常匹配，希望能参与GPU算力平台的建设。

核心点：

- Go
- K8s
- 分布式
- AI infra

---

# 二、为什么想加入大疆（高概率）

### Q2 为什么想来大疆？

回答逻辑：

**技术吸引 + 行业 + 个人成长**

示例：

> 我比较关注大疆的技术方向，大疆在机器人、自动驾驶、AI视觉领域都有非常深的技术积累。
>
> 我自己这几年一直在做云原生和AI系统结合的方向，比如LLM服务架构、Kubernetes部署和自动扩缩容。
>
> 算力平台是AI时代最核心的基础设施之一，特别是GPU资源调度和大模型训练平台，这部分技术挑战非常大。
>
> 我希望未来能深入 **AI infra / GPU scheduling / 大模型平台**，所以这个岗位和我的技术发展方向非常契合。

---

# 三、Kubernetes 深度问题（核心）

### Q3 Kubernetes 调度流程是什么？

核心流程：

```
Pod 创建
   ↓
Scheduler
   ↓
1 过滤节点 (Filter)
2 评分节点 (Score)
   ↓
选择 Node
   ↓
kubelet 创建容器
```

关键组件：

- **kube-scheduler**
- **etcd**
- **kubelet**
- **container runtime**

---

### Q4 Kubernetes 如何支持 GPU？

方式：

```
Device Plugin
```

流程：

1 注册 GPU device plugin
2 kubelet 发现 GPU
3 Scheduler 按资源调度

GPU resource example：

```
resources:
  limits:
    nvidia.com/gpu: 1
```

调度逻辑：

```
Node GPU available
     ↓
Scheduler 分配
     ↓
容器绑定 GPU
```

---

### Q5 如果 GPU 利用率低怎么办？

算力平台典型问题。

解决方案：

**1 GPU共享**

例如：

```
NVIDIA MIG
```

把 GPU 切成多个实例

---

**2 GPU time-slicing**

多个任务共享GPU

---

**3 优先级调度**

```
高优先级训练任务
低优先级推理任务
```

---

**4 Job queue**

类似：

```
Kubernetes + Volcano
```

---

# 四、AI算力平台（非常可能）

### Q6 如何设计 AI 训练平台？

核心模块：

```
用户提交任务
      ↓
任务调度
      ↓
资源分配
      ↓
任务执行
      ↓
日志 / metrics
```

架构：

```
User
 ↓
API Server
 ↓
Job Queue
 ↓
Scheduler
 ↓
Kubernetes
 ↓
GPU Node
```

组件：

- Kubernetes
- Volcano scheduler
- GPU operator
- Prometheus

---

### Q7 如何提高 GPU 利用率？

方法：

1️⃣ Job Queue

```
FIFO
Priority Queue
```

2️⃣ Gang Scheduling

大模型训练需要多个GPU

```
8 GPU together
```

3️⃣ Preemption

低优先级任务让出资源

---

# 五、分布式系统（一定问）

### Q8 Kafka 如何保证消息不丢？

关键：

```
acks=all
replication factor >= 3
min.insync.replicas=2
```

生产者：

```
acks=all
retries
idempotent producer
```

消费者：

```
手动 commit
```

---

### Q9 Kafka 如何保证顺序？

Kafka 顺序保证：

```
partition 内有序
```

解决方案：

```
same key -> same partition
```

---

### Q10 如何设计一个高并发系统？

原则：

```
无状态
水平扩展
异步化
缓存
限流
```

架构：

```
Load Balancer
       ↓
Stateless Service
       ↓
Message Queue
       ↓
Workers
```

---

# 六、LLM / AI问题

### Q11 LLM 推理服务如何设计？

典型架构：

```
Client
  ↓
API Gateway
  ↓
Inference Service
  ↓
Model Server
  ↓
GPU
```

优化：

- batching
- caching
- streaming response

---

### Q12 为什么 LLM 推理要 batch？

因为 GPU：

```
并行计算
```

batch 可以：

```
提高 GPU utilization
```

---

### Q13 LLM 训练流程

基本流程：

```
Dataset
  ↓
Tokenization
  ↓
Distributed Training
  ↓
Checkpoint
  ↓
Evaluation
```

框架：

- PyTorch
- DeepSpeed
- Megatron

---

# 七、Golang 深度问题

### Q14 Go 如何实现 worker pool？

结构：

```
job channel
worker goroutines
```

代码思路：

```go
jobs := make(chan Job)

for i := 0; i < workerNum; i++ {
    go worker(jobs)
}
```

---

### Q15 Go 如何避免 goroutine 泄露？

方法：

- context cancel
- close channel
- timeout

---

### Q16 Go map 是线程安全吗？

不是。

解决：

```
sync.RWMutex
```

或

```
sync.Map
```

---

# 八、项目深挖（最关键）

### Q17 为什么要拆微服务？

回答：

```
单体问题：

1 扩展困难
2 发布风险
3 团队协作困难
```

拆分后：

```
独立扩展
独立部署
故障隔离
```

---

### Q18 Kafka 为什么比 RabbitMQ 适合？

原因：

```
高吞吐
日志结构
可回溯
```

适合：

```
event streaming
```

---

### Q19 为什么选 Kubernetes？

优点：

```
自动扩缩容
自愈能力
声明式部署
生态丰富
```

---

# 九、场景设计题（非常可能）

### Q20 设计一个 GPU 调度系统

核心模块：

```
Task Submit
Scheduler
Resource Manager
Worker Node
```

调度策略：

```
priority
fair share
gang scheduling
```

---

# 十、反问面试官（非常重要）

最后一定问：

推荐：

**1**

> 目前大疆算力平台 GPU规模大概是多少？

**2**

> 调度系统是基于 Kubernetes 原生 scheduler 还是自研？

**3**

> LLM 训练和推理平台是否统一？

**4**

> 算力平台未来规划是什么？

---

# 最后给你一个重点

大疆这个岗位最重要的是：

### 三个能力

**1 云原生**

K8s
调度
容器

**2 分布式系统**

高并发
消息系统
可靠性

**3 AI infra**

GPU
LLM
训练

---

如果你愿意，我可以再帮你整理 **一份“大疆算力平台面试必杀题（20题）”**：

例如：

- **如何实现 GPU 调度器**
- **如何设计分布式训练平台**
- **Kubernetes scheduler plugin 如何写**
- **LLM 推理平台架构**

这套基本是 **大疆 / 字节 / 阿里 AI infra 面试真题级别**。

---

实现 **GPU 调度器（GPU Scheduler）** 本质是解决三个问题：

1️⃣ **GPU 资源发现（Resource Discovery）**
2️⃣ **任务调度（Scheduling Policy）**
3️⃣ **资源分配与隔离（Allocation & Isolation）**

在实际工程中，GPU调度一般不会从零写，而是 **基于 Kubernetes Scheduler 扩展**。很多AI平台（如 Kubeflow、Volcano）都是这么做的。

我从 **系统设计 + 核心算法 + 工程实现** 三个层面讲清楚，这基本是面试官想听的答案。

---

# 一、GPU调度系统整体架构

典型 AI 算力平台架构：

```
User
 ↓
Job Submit API
 ↓
Job Queue
 ↓
GPU Scheduler
 ↓
Kubernetes
 ↓
GPU Node
```

组件说明：

| 模块             | 作用              |
| ---------------- | ----------------- |
| Job API          | 提交训练/推理任务 |
| Job Queue        | 任务排队          |
| Scheduler        | 选择GPU节点       |
| Resource Manager | GPU资源管理       |
| Worker Node      | 执行任务          |

GPU Scheduler 的职责：

```
选择哪台机器
选择哪些 GPU
何时运行
```

---

# 二、GPU资源发现

GPU首先需要被 Kubernetes 识别。

实现方式：

### 1 Device Plugin

Kubernetes GPU支持是通过 **Device Plugin**。

例如 NVIDIA：

```
nvidia-device-plugin
```

节点注册 GPU：

```
nvidia.com/gpu: 8
```

Pod 请求 GPU：

```yaml
resources:
  limits:
    nvidia.com/gpu: 2
```

kube-scheduler 会根据资源调度。

---

# 三、GPU调度核心流程

调度流程一般分 3 步：

```
1 过滤节点 (Filter)
2 评分节点 (Score)
3 绑定节点 (Bind)
```

### 1 Filter

过滤不满足资源要求的节点：

例如：

```
GPU >= requested
GPU memory >= required
node healthy
```

示例：

```
node1 GPU=8 free=2
node2 GPU=4 free=0
node3 GPU=8 free=4
```

任务需要：

```
GPU=2
```

可用：

```
node1
node3
```

---

### 2 Score

对候选节点打分。

常见策略：

**1 GPU利用率**

尽量填满节点：

```
bin packing
```

示例：

```
node1 free=2
node3 free=4
```

优先：

```
node1
```

避免 GPU 碎片。

---

**2 数据局部性**

如果数据在某个节点：

```
prefer same node
```

减少数据传输。

---

**3 网络拓扑**

大模型训练需要：

```
NVLink
same rack
```

避免跨机通信。

---

### 3 Bind

Scheduler 选择节点：

```
Pod -> Node
```

然后：

```
kubelet
```

创建容器。

---

# 四、GPU高级调度策略

AI算力平台不会只做简单调度。

一般会有：

---

## 1 Gang Scheduling（最重要）

大模型训练需要 **多个GPU同时启动**。

例如：

```
8 GPU
```

如果只分到4个：

```
任务无法运行
```

解决：

```
all or nothing
```

示例：

```
request = 8 GPU

node1 = 4
node2 = 4
node3 = 8
```

选择：

```
node3
```

实现方式：

```
Volcano scheduler
```

---

## 2 优先级调度

任务优先级：

```
P0 线上推理
P1 训练任务
P2 实验任务
```

调度策略：

```
高优先级抢占低优先级
```

叫：

```
preemption
```

---

## 3 GPU共享

GPU很贵，需要共享。

方式：

### MIG（Multi Instance GPU）

例如：

```
A100 80GB
```

可以切：

```
7 个 GPU实例
```

实现：

```
NVIDIA MIG
```

---

### Time slicing

多个任务轮流使用 GPU。

---

# 五、GPU调度算法

常见算法：

### 1 Bin Packing

目标：

```
减少 GPU 碎片
```

例子：

```
node1 free=2
node2 free=8
```

任务：

```
2 GPU
```

优先：

```
node1
```

---

### 2 Fair Scheduling

公平分配资源：

例如：

```
team A
team B
```

各占：

```
50%
```

---

### 3 Dominant Resource Fairness (DRF)

考虑多资源：

```
CPU
GPU
Memory
```

公平调度。

---

# 六、工程实现方式

有三种实现方式：

---

# 方式1：Kubernetes Scheduler Plugin（推荐）

写一个 **scheduler plugin**。

K8s调度框架：

```
Filter
Score
Reserve
Bind
```

插件实现：

```
ScorePlugin
FilterPlugin
```

示例：

```go
func (p *GPUScheduler) Score(
    ctx context.Context,
    state *framework.CycleState,
    pod *v1.Pod,
    nodeName string,
) (int64, *framework.Status) {

    gpuFree := getFreeGPU(nodeName)

    score := calculateScore(gpuFree)

    return score, framework.NewStatus(framework.Success)
}
```

---

# 方式2：自研调度器

架构：

```
Job queue
↓
custom scheduler
↓
kubernetes API
```

流程：

```
scheduler
→ patch pod nodeName
```

---

# 方式3：使用 Volcano

开源项目：

```
Volcano
```

特点：

- gang scheduling
- GPU scheduling
- queue

很多 AI 平台用这个。

---

# 七、生产级GPU算力平台架构

成熟平台一般是：

```
User
 ↓
Kubeflow / API
 ↓
Job Queue
 ↓
Volcano Scheduler
 ↓
Kubernetes
 ↓
GPU Node
```

监控：

```
Prometheus
Grafana
DCGM exporter
```

---

# 八、GPU调度系统优化

真实生产环境要解决：

### 1 GPU碎片

例如：

```
剩1 GPU everywhere
```

解决：

```
bin packing
```

---

### 2 GPU利用率低

解决：

```
batch scheduling
time sharing
```

---

### 3 GPU资源浪费

解决：

```
idle GPU reclaim
```

---

# 九、面试总结回答模板

如果面试官问：

**如何实现 GPU 调度器？**

可以这样回答：

> GPU调度器本质上是解决GPU资源管理和任务调度的问题。在工程实现上通常基于Kubernetes Scheduler扩展实现。
>
> 首先通过Device Plugin让Kubernetes识别GPU资源，例如nvidia.com/gpu。
>
> 在调度阶段，Scheduler会经历Filter和Score两个阶段：Filter负责筛选满足GPU资源要求的节点，Score阶段根据策略进行打分，例如GPU利用率、数据局部性或网络拓扑。
>
> 对于AI训练场景，还需要支持Gang Scheduling，因为很多训练任务需要多个GPU同时启动。
>
> 在工程实现上，可以通过Kubernetes Scheduler Plugin实现自定义调度策略，例如编写FilterPlugin和ScorePlugin来实现GPU bin-packing策略。
>
> 在生产环境中通常会结合Volcano或Kubeflow构建完整AI算力平台，实现任务队列、优先级调度以及GPU资源共享等能力。

---

如果你愿意，我可以再给你一套 **AI算力平台最难的面试题（大疆 / 字节 AI Infra 真题级）**，比如：

- **如何设计万卡GPU集群调度系统**
- **LLM推理平台架构设计**
- **Kubernetes Scheduler源码解析**
- **GPU碎片问题如何解决**

这些基本是 **60-150万 AI Infra 面试必问题**。

设计 **分布式训练平台（Distributed Training Platform）** 是 AI Infra / 算力平台岗位最经典的系统设计题之一。面试官通常希望听到：

- **整体架构**
- **训练任务调度**
- **GPU资源管理**
- **分布式通信**
- **训练容错与恢复**
- **平台化能力**

我给你一个 **面试级完整答案结构（从0到1设计）**。

---

# 一、问题本质

分布式训练平台要解决的问题：

```text
1 用户提交训练任务
2 平台分配GPU资源
3 启动分布式训练
4 管理训练过程
5 监控与日志
6 容错与恢复
```

核心目标：

- 高 GPU 利用率
- 高可靠性
- 易用性
- 可扩展

---

# 二、整体架构设计

一个典型的分布式训练平台架构：

```
           用户
            │
        Web / CLI
            │
        API Gateway
            │
       Training Service
            │
        Job Queue
            │
        Scheduler
            │
        Kubernetes
            │
     GPU Worker Nodes
```

核心组件：

| 模块        | 作用                 |
| ----------- | -------------------- |
| API Server  | 接收训练任务         |
| Job Manager | 管理训练任务生命周期 |
| Scheduler   | GPU调度              |
| Kubernetes  | 容器调度             |
| Worker Node | 执行训练             |
| Storage     | 数据与模型存储       |
| Monitoring  | 监控训练             |

---

# 三、训练任务提交流程

用户提交训练任务：

```yaml
job:
  name: llama-train
  gpu: 8
  cpu: 16
  memory: 64Gi
  dataset: s3://dataset
```

任务执行流程：

```
User submit
     ↓
API Server
     ↓
Job Queue
     ↓
Scheduler
     ↓
Allocate GPU
     ↓
Launch training pods
     ↓
Distributed training start
```

---

# 四、GPU资源调度

调度策略是平台核心。

### 1 GPU资源发现

通过 Kubernetes **device plugin**

```
nvidia.com/gpu
```

节点资源：

```
node1 GPU=8
node2 GPU=8
node3 GPU=4
```

---

### 2 Gang Scheduling（必须）

大模型训练：

```
需要 8 GPU
```

必须 **同时启动**

```
all or nothing
```

实现方式：

- Volcano scheduler
- K8s scheduler plugin

---

### 3 Bin Packing

目标：

```
减少 GPU 碎片
```

示例：

```
node1 free=2
node2 free=8
```

任务：

```
2 GPU
```

优先 node1。

---

# 五、分布式训练启动

训练框架：

常见：

- PyTorch Distributed
- DeepSpeed
- Horovod

启动方式：

```
torchrun
```

例如：

```
torchrun \
--nnodes=2 \
--nproc_per_node=4 \
train.py
```

平台需要做：

```
自动生成 hostfile
自动设置环境变量
启动 worker pods
```

---

# 六、分布式通信

GPU之间需要通信。

常见通信库：

```
NCCL
```

通信拓扑：

```
AllReduce
Ring
Tree
```

示意：

```
GPU1 → GPU2 → GPU3 → GPU4
```

平台需要考虑：

- RDMA
- NVLink
- 网络带宽

优化策略：

```
same rack scheduling
```

减少网络延迟。

---

# 七、数据存储

训练数据通常很大。

存储方式：

```
Object Storage
```

例如：

- S3
- MinIO
- HDFS

数据加载：

```
dataset → worker
```

优化：

```
数据缓存
local SSD
```

避免重复下载。

---

# 八、训练监控

需要实时监控训练：

监控内容：

```
GPU utilization
loss
learning rate
throughput
```

监控系统：

- Prometheus
- Grafana
- TensorBoard

例如：

```
DCGM exporter
```

监控 GPU。

---

# 九、训练容错

大模型训练可能运行 **几天甚至几周**。

必须支持：

```
checkpoint
```

流程：

```
save checkpoint
→ object storage
```

如果任务失败：

```
restart from checkpoint
```

实现方式：

```
Kubernetes restart
```

或

```
Job controller
```

---

# 十、多租户资源管理

算力平台通常支持多个团队。

例如：

```
Team A
Team B
Team C
```

调度策略：

### Fair Scheduling

```
资源公平
```

例如：

```
GPU quota
```

---

### Priority Scheduling

```
P0 推理
P1 训练
P2 实验
```

---

# 十一、GPU利用率优化

GPU很贵。

常见优化：

### 1 GPU共享

方式：

```
MIG
```

例如：

```
A100 → 7 instances
```

---

### 2 Time slicing

多个任务共享 GPU。

---

### 3 自动扩缩容

基于：

```
job queue
```

自动增加 GPU 节点。

---

# 十二、生产级架构

成熟平台通常是：

```
User
 ↓
Kubeflow
 ↓
Training Operator
 ↓
Volcano Scheduler
 ↓
Kubernetes
 ↓
GPU Nodes
```

常用组件：

| 组件       | 用途    |
| ---------- | ------- |
| Kubeflow   | ML平台  |
| Volcano    | GPU调度 |
| Prometheus | 监控    |
| Argo       | 工作流  |

---

# 十三、面试回答模板（推荐）

如果面试官问：

**如何设计分布式训练平台？**

你可以这样答：

> 分布式训练平台主要解决大模型训练的资源调度、任务管理和训练管理问题。整体架构通常分为任务管理层、调度层和执行层。
>
> 用户首先通过API或CLI提交训练任务，任务进入Job Queue。调度器根据任务GPU需求进行资源调度。对于大模型训练通常需要Gang Scheduling，保证多个GPU同时启动。
>
> 在执行层通常基于Kubernetes运行训练容器，通过Device Plugin管理GPU资源。训练任务一般使用PyTorch Distributed或DeepSpeed框架，通过NCCL进行GPU间通信。
>
> 在平台能力方面，还需要提供训练监控、日志收集以及checkpoint机制保证训练任务可以容错恢复。
>
> 在资源管理方面，通过bin-packing策略提高GPU利用率，并结合优先级调度和配额机制实现多租户资源公平使用。
>
> 在生产环境中通常会结合Kubeflow和Volcano构建完整的AI训练平台。

---

# 最后给你一个面试加分点 ⭐

如果你补一句：

> 在超大规模GPU集群中，还需要解决GPU碎片问题和跨机通信优化问题，比如优先调度NVLink拓扑一致的GPU节点，以及通过RDMA减少AllReduce通信开销。

面试官基本会觉得：

**你懂 AI Infra。**

---

如果你愿意，我可以继续帮你准备 **一个真正难的面试题**（AI Infra 面试几乎必问）：

**“如何设计万卡 GPU 集群调度系统？”**

这个问题基本是：

- 大疆
- 字节
- 阿里
- OpenAI Infra

级别面试题。

---

在面试里问 **“Kubernetes scheduler plugin 如何写”**，其实是在考察：

1. 你是否理解 **Kubernetes 调度流程**
2. 是否知道 **Scheduler Framework**
3. 能否实现 **自定义调度策略（例如 GPU / bin packing）**

核心思想是：
**通过实现 Scheduler Framework 的接口，在调度生命周期的不同阶段插入自己的逻辑。**

下面给你一个 **工程级 + 面试级完整答案**。

---

# 一、Kubernetes Scheduler Framework

从 Kubernetes v1.19 开始，调度器支持 **插件化架构**。

调度生命周期：

```text
Pod -> Scheduling Queue
        ↓
     PreFilter
        ↓
       Filter
        ↓
       Score
        ↓
      Reserve
        ↓
       Bind
```

插件可以插入这些阶段。

常见插件接口：

| 阶段      | 作用         |
| --------- | ------------ |
| PreFilter | 调度前预处理 |
| Filter    | 过滤节点     |
| Score     | 节点评分     |
| Reserve   | 预留资源     |
| Bind      | 绑定节点     |

---

# 二、Scheduler Plugin 的基本结构

一个插件通常是一个 Go struct。

```go
type GPUScheduler struct {
    handle framework.Handle
}
```

插件需要实现对应接口，例如：

```go
framework.FilterPlugin
framework.ScorePlugin
```

---

# 三、实现 FilterPlugin（过滤节点）

Filter 用于 **筛选符合条件的节点**。

例如：
只允许 GPU >= 任务需求。

示例代码：

```go
func (p *GPUScheduler) Filter(
    ctx context.Context,
    state *framework.CycleState,
    pod *v1.Pod,
    nodeInfo *framework.NodeInfo,
) *framework.Status {

    node := nodeInfo.Node()

    gpu := node.Status.Capacity["nvidia.com/gpu"]

    required := getPodGPURequest(pod)

    if gpu.Value() < required {
        return framework.NewStatus(framework.Unschedulable, "not enough GPU")
    }

    return framework.NewStatus(framework.Success)
}
```

逻辑：

```
Node GPU >= Pod GPU request
        ↓
allowed
```

---

# 四、实现 ScorePlugin（节点打分）

Score 阶段用于 **选择最优节点**。

例如实现 **GPU bin-packing**：

优先 GPU 剩余最少但足够的节点。

示例：

```go
func (p *GPUScheduler) Score(
    ctx context.Context,
    state *framework.CycleState,
    pod *v1.Pod,
    nodeName string,
) (int64, *framework.Status) {

    nodeInfo, err := p.handle.SnapshotSharedLister().NodeInfos().Get(nodeName)
    if err != nil {
        return 0, framework.AsStatus(err)
    }

    gpuFree := getFreeGPU(nodeInfo)

    score := int64(100 - gpuFree)

    return score, framework.NewStatus(framework.Success)
}
```

例如：

```
Node1 free GPU = 2
Node2 free GPU = 8
```

Score：

```
Node1 = 98
Node2 = 92
```

优先 Node1。

---

# 五、实现 Score Normalize

Kubernetes要求评分 **0-100**。

```go
func (p *GPUScheduler) NormalizeScore(
    ctx context.Context,
    state *framework.CycleState,
    pod *v1.Pod,
    scores framework.NodeScoreList,
) *framework.Status {

    var max int64 = 0

    for _, s := range scores {
        if s.Score > max {
            max = s.Score
        }
    }

    for i := range scores {
        scores[i].Score = scores[i].Score * framework.MaxNodeScore / max
    }

    return framework.NewStatus(framework.Success)
}
```

---

# 六、注册插件

插件需要注册到 scheduler。

```go
func NewGPUScheduler(
    obj runtime.Object,
    handle framework.Handle,
) (framework.Plugin, error) {

    return &GPUScheduler{
        handle: handle,
    }, nil
}
```

---

# 七、在 scheduler config 启用

配置文件：

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
    plugins:
      filter:
        enabled:
          - name: GPUScheduler
      score:
        enabled:
          - name: GPUScheduler
```

---

# 八、编译自定义 Scheduler

步骤：

### 1 Fork Kubernetes Scheduler

或使用 scheduler framework example。

### 2 编译

```bash
make kube-scheduler
```

### 3 部署

替换默认 scheduler：

```
kube-scheduler
```

或者：

```
custom scheduler
```

---

# 九、真实生产用法

很多公司不会自己写完整调度器，而是：

方案1：

```
Scheduler Plugin
```

方案2：

```
Volcano Scheduler
```

方案3：

```
Kubeflow Training Operator
```

例如 GPU调度：

```
Volcano + Kubernetes
```

---

# 十、真实GPU调度插件会做什么

生产级 plugin 会考虑：

### 1 GPU拓扑

例如：

```
NVLink
```

尽量选择同一节点 GPU。

---

### 2 NUMA

避免跨 NUMA。

---

### 3 GPU碎片

使用：

```
bin packing
```

---

### 4 数据局部性

数据在：

```
node1
```

优先 node1。

---

# 十一、面试回答模板

如果面试官问：

**Kubernetes scheduler plugin 如何写？**

推荐回答：

> Kubernetes 从 1.19 开始提供 Scheduler Framework，允许通过插件机制扩展调度策略。调度流程包括 PreFilter、Filter、Score、Reserve 和 Bind 等阶段。
>
> 如果需要实现自定义调度逻辑，例如 GPU 调度，可以编写一个 Scheduler Plugin，实现 FilterPlugin 和 ScorePlugin 接口。
>
> Filter 阶段主要用于过滤不满足条件的节点，例如 GPU 数量不足的节点。Score 阶段对剩余节点进行打分，例如实现 bin-packing 策略，提高 GPU 利用率。
>
> 插件实现后需要在 kube-scheduler 的配置文件中注册并启用，然后编译自定义 scheduler 部署到集群中。
>
> 在生产环境中，这种机制通常用于实现 GPU 调度优化、拓扑感知调度或者数据局部性调度等策略。

---

# 十二、面试加分点 ⭐

如果你补一句：

> 在 AI 算力平台中，通常会结合 Scheduler Plugin 实现 GPU bin-packing 或拓扑感知调度，同时配合 Volcano 实现 Gang Scheduling 支持分布式训练任务。

面试官基本会认为：

**你是真的做过云原生调度系统。**

---

如果你愿意，我可以继续帮你准备 **一个超级高频 AI Infra 面试题**：

**“GPU碎片问题如何解决？”**

这个问题是：

- 大疆
- 字节
- 阿里
- NVIDIA

算力平台岗位 **几乎必问**。

---

**LLM 推理平台架构（LLM Inference Platform）** 是 AI Infra / 算力平台岗位非常常见的系统设计题。面试官主要看你是否理解：

- 推理请求流量特点
- GPU资源管理
- 模型部署方式
- 推理性能优化（最重要）
- 多模型管理

下面给你一套 **面试级架构 + 关键技术点**。

---

# 一、LLM 推理平台解决的问题

LLM 推理平台主要解决：

```
1 模型部署
2 推理请求处理
3 GPU资源管理
4 推理性能优化
5 多租户管理
```

目标：

- **低延迟**
- **高吞吐**
- **高 GPU 利用率**
- **支持多模型**

---

# 二、整体架构

典型 LLM 推理平台：

```
Client
   │
API Gateway
   │
Load Balancer
   │
Inference Router
   │
Model Serving Layer
   │
GPU Workers
   │
Model Storage
```

组件说明：

| 组件             | 作用     |
| ---------------- | -------- |
| API Gateway      | 请求入口 |
| Inference Router | 路由模型 |
| Model Serving    | 推理服务 |
| GPU Workers      | 执行推理 |
| Model Storage    | 模型存储 |

---

# 三、请求流程

完整推理流程：

```
User Request
    ↓
API Gateway
    ↓
Inference Router
    ↓
Batch Queue
    ↓
GPU Worker
    ↓
Token Generation
    ↓
Streaming Response
```

关键：

```
Token streaming
```

用户可以实时看到输出。

---

# 四、模型部署方式

LLM 推理通常使用：

| 框架         | 用途             |
| ------------ | ---------------- |
| vLLM         | 高性能推理       |
| TensorRT-LLM | NVIDIA优化       |
| TGI          | HuggingFace 推理 |

例如：

```
vLLM
```

特点：

```
PagedAttention
continuous batching
```

大幅提升吞吐。

---

# 五、GPU推理优化（核心）

LLM推理瓶颈：

```
GPU memory
```

主要优化：

---

## 1 Continuous Batching

传统方式：

```
request1
request2
request3
```

一个 batch。

问题：

```
等待时间长
```

Continuous batching：

```
GPU batch 动态加入请求
```

示例：

```
step1: req1 req2
step2: req1 req2 req3
step3: req1 req3
```

好处：

```
GPU始终满载
```

---

## 2 KV Cache

LLM生成 token 时：

```
attention KV
```

会缓存。

结构：

```
KV Cache
```

存储：

```
GPU memory
```

减少重复计算。

---

## 3 PagedAttention

问题：

```
KV cache fragmentation
```

解决：

```
paged memory
```

类似：

```
virtual memory
```

优势：

```
减少显存碎片
```

这是 **vLLM核心技术**。

---

# 六、多模型管理

推理平台通常需要支持：

```
多个模型
```

例如：

```
llama
mistral
qwen
```

管理方式：

### Model Registry

存储：

```
S3
MinIO
```

---

### Model Loader

流程：

```
download model
load GPU
```

优化：

```
lazy loading
```

按需加载模型。

---

# 七、GPU资源管理

GPU非常昂贵，需要调度。

策略：

### 1 Model per GPU

```
1 GPU = 1 model
```

简单但利用率低。

---

### 2 Multi Model GPU

```
多个模型共享 GPU
```

例如：

```
vLLM
```

支持。

---

### 3 MIG

NVIDIA：

```
MIG
```

GPU切片。

---

# 八、推理扩展（Scaling）

推理平台需要自动扩容。

方案：

### Kubernetes

```
Deployment
```

自动扩容：

```
HPA
```

基于：

```
QPS
GPU utilization
```

---

### Queue-based scaling

根据：

```
request queue
```

自动扩容。

---

# 九、推理延迟优化

LLM latency 关键指标：

```
TTFT
```

Time To First Token

优化方法：

### 1 Prompt caching

缓存 prompt embedding。

---

### 2 Prefix caching

多个请求共享 prefix。

---

### 3 Speculative decoding

使用：

```
small model
```

预测 token。

---

# 十、监控系统

需要监控：

```
GPU utilization
token/s
latency
QPS
```

技术：

- Prometheus
- Grafana

GPU监控：

```
DCGM exporter
```

---

# 十一、生产级架构

成熟平台通常：

```
Client
  ↓
API Gateway
  ↓
Router
  ↓
Inference Service (vLLM)
  ↓
Kubernetes
  ↓
GPU Nodes
```

存储：

```
S3 / MinIO
```

监控：

```
Prometheus + Grafana
```

---

# 十二、LLM 推理平台最大挑战

### 1 GPU显存

例如：

```
70B model
```

需要：

```
> 140GB
```

解决：

```
tensor parallel
```

---

### 2 GPU碎片

解决：

```
dynamic batching
```

---

### 3 多租户

解决：

```
quota
priority
```

---

# 十三、面试回答模板

如果面试官问：

**如何设计 LLM 推理平台？**

推荐回答：

> LLM 推理平台主要解决模型部署、推理请求处理以及 GPU 资源管理问题。整体架构通常包括 API Gateway、请求路由层、模型服务层以及 GPU Worker 节点。
>
> 用户请求首先进入 API Gateway，然后通过 Router 根据模型类型路由到对应的推理服务。推理服务通常使用高性能框架，例如 vLLM 或 TensorRT-LLM。
>
> 在推理优化方面，关键技术包括 continuous batching 提高 GPU 利用率，以及 KV Cache 减少重复计算。vLLM 还通过 PagedAttention 技术解决 KV cache 的显存碎片问题。
>
> 在资源管理方面，平台通常运行在 Kubernetes 上，通过 GPU device plugin 管理 GPU，并通过 HPA 实现自动扩缩容。
>
> 同时平台需要支持多模型管理，通过模型仓库存储模型权重，并支持按需加载。
>
> 最终通过 Prometheus 和 Grafana 对 GPU 利用率、token 生成速度和请求延迟进行监控。

---

# 十四、面试加分点 ⭐

如果你补一句：

> 在高并发推理场景中，还可以通过 prefix caching 和 speculative decoding 进一步降低延迟，提高 token throughput。

面试官基本会认为：

**你真的懂 LLM infra。**

---

如果你愿意，我可以继续帮你整理一个 **AI Infra 面试最难的问题之一**：

**“如何设计一个支持 1000+ GPU 的 LLM 推理集群？”**

这是：

- 大疆
- 字节
- OpenAI Infra

级别的系统设计题。

---

设计 **万卡 GPU 集群调度系统（10,000+ GPUs Scheduler）** 是 AI Infra / 算力平台岗位最顶级的系统设计题之一。面试官主要考察：

- 大规模 **资源调度能力**
- **GPU拓扑感知调度**
- **训练任务 Gang Scheduling**
- **GPU碎片问题**
- **公平调度 / 多租户**

我给你一个 **面试级完整架构答案**。

---

# 一、问题规模

万卡 GPU 集群典型规模：

```
1000+ nodes
10000+ GPUs
100+ concurrent jobs
```

任务类型：

```
LLM training
LLM inference
data processing
```

核心目标：

```
高 GPU 利用率
低调度延迟
公平调度
稳定性
```

---

# 二、整体架构

典型 GPU 集群架构：

```
User
  │
Job API
  │
Job Queue
  │
Scheduler
  │
Placement Engine
  │
Kubernetes / Resource Manager
  │
GPU Nodes
```

组件：

| 组件             | 作用         |
| ---------------- | ------------ |
| Job Manager      | 管理训练任务 |
| Scheduler        | 调度任务     |
| Placement Engine | 选择GPU      |
| Resource Manager | 管理节点     |
| Node Agent       | 上报资源     |

---

# 三、GPU资源管理

每个节点资源：

```
node1
 GPU: 8
 CPU: 64
 memory: 256GB
```

调度器维护：

```
cluster state
```

例如：

```
Node1 free GPU=4
Node2 free GPU=8
Node3 free GPU=2
```

通常存储在：

```
in-memory cache
```

更新来源：

```
node heartbeat
```

---

# 四、调度流程

调度流程：

```
Job submit
   ↓
Job Queue
   ↓
Scheduler
   ↓
Placement algorithm
   ↓
Reserve GPUs
   ↓
Launch Pods
```

核心问题：

```
如何选择GPU
```

---

# 五、Gang Scheduling（关键）

大模型训练需要：

```
N GPUs simultaneously
```

例如：

```
8 GPU
16 GPU
64 GPU
```

必须：

```
all or nothing
```

否则：

```
deadlock
```

实现方式：

```
gang scheduling
```

例如：

```
Volcano scheduler
```

---

# 六、GPU拓扑感知调度

GPU通信成本很高。

典型拓扑：

```
NVLink
PCIe
InfiniBand
```

通信速度：

```
NVLink > PCIe > Network
```

调度优先级：

```
same GPU node
same rack
same cluster
```

示例：

```
Job needs 8 GPU
```

优先：

```
single node
```

否则：

```
2 nodes
```

---

# 七、GPU碎片问题（核心难点）

碎片示例：

```
node1: 2 GPU free
node2: 2 GPU free
node3: 2 GPU free
node4: 2 GPU free
```

任务：

```
8 GPU
```

无法运行。

解决方法：

---

## 1 Bin Packing

优先填满节点：

```
pack workloads
```

例如：

```
job1 → node1
job2 → node1
```

减少碎片。

---

## 2 Defragmentation

后台：

```
job migration
```

整理 GPU。

---

## 3 GPU slicing

使用：

```
MIG
```

适用于推理。

---

# 八、多级调度架构

万卡集群不能单一 scheduler。

典型架构：

```
Global Scheduler
       │
Queue Scheduler
       │
Node Scheduler
```

说明：

### Global Scheduler

负责：

```
queue priority
fair scheduling
```

---

### Placement Engine

负责：

```
choose GPU nodes
```

---

### Node Agent

负责：

```
launch containers
```

---

# 九、多租户公平调度

AI公司通常有多个团队：

```
Team A
Team B
Team C
```

需要：

```
fair share
```

常见算法：

### Dominant Resource Fairness (DRF)

保证：

```
公平资源分配
```

---

# 十、任务优先级

任务优先级：

```
P0 线上推理
P1 训练任务
P2 实验任务
```

调度策略：

```
preemption
```

抢占 GPU。

---

# 十一、调度性能优化

万卡集群问题：

```
scheduler bottleneck
```

优化：

### 1 Cache cluster state

避免频繁访问 etcd。

---

### 2 Incremental scheduling

只调度变化部分。

---

### 3 Parallel scheduling

多线程调度。

---

# 十二、故障处理

节点可能失败：

```
node crash
```

解决：

### Heartbeat

检测：

```
node health
```

---

### Job recovery

使用：

```
checkpoint
```

恢复训练。

---

# 十三、生产级架构（真实AI公司）

真实AI平台：

```
User
  ↓
AI Platform API
  ↓
Job Manager
  ↓
Volcano Scheduler
  ↓
Kubernetes
  ↓
GPU Cluster
```

GPU资源：

```
NVIDIA device plugin
```

---

# 十四、监控系统

必须监控：

```
GPU utilization
job queue
network bandwidth
```

工具：

```
Prometheus
Grafana
```

GPU metrics：

```
DCGM exporter
```

---

# 十五、最大技术挑战

万卡 GPU 调度难点：

### 1 GPU碎片

GPU利用率下降。

---

### 2 网络瓶颈

AllReduce通信。

---

### 3 调度延迟

任务排队。

---

### 4 多租户资源争抢

公平性问题。

---

# 十六、面试回答模板（推荐）

如果面试官问：

**如何设计万卡 GPU 集群调度系统？**

可以这样回答：

> 万卡 GPU 集群调度系统的核心目标是提高 GPU 利用率，同时支持大规模分布式训练任务。整体架构通常分为任务管理层、调度层和资源执行层。
>
> 用户首先提交训练任务到 Job Manager，任务进入调度队列。调度器根据任务的 GPU 需求进行资源分配。对于大模型训练任务，需要支持 Gang Scheduling，保证多个 GPU 可以同时启动。
>
> 在 GPU 选择方面，需要实现拓扑感知调度，优先选择同一节点或 NVLink 互联的 GPU，以降低分布式通信开销。
>
> 在大规模集群中还需要解决 GPU 碎片问题，通常通过 bin-packing 策略减少碎片，同时结合后台 defragmentation 机制整理资源。
>
> 在多租户环境中，可以通过 Dominant Resource Fairness 实现公平资源分配，并结合优先级和抢占机制保障关键任务运行。
>
> 最终整个调度系统运行在 Kubernetes 上，可以结合 Volcano Scheduler 实现 AI 训练任务调度。

---

# 十七、面试官最喜欢的加分点 ⭐

如果你补一句：

> 在超大规模 GPU 集群中，调度器还需要考虑 GPU 拓扑感知调度，例如优先调度 NVLink 连接的 GPU，以优化 NCCL AllReduce 性能。

基本可以判断：

**面试官会认为你具备 AI Infra 架构能力。**

---

如果你愿意，我可以再帮你整理 **AI算力平台最容易问到的 25 个系统设计面试题 + 答案**（基本覆盖大疆 / 字节 / 阿里 AI Infra 面试）。

---

GPU 碎片（GPU Fragmentation）是 **大规模 GPU 集群调度系统**（例如 LLM 推理 / 训练平台）中非常核心的问题。特别是在 **万卡 GPU 集群**里，如果不处理好，**GPU 利用率可能从 80% 掉到 30%-40%**。

GPU 碎片主要分为两类：

---

# 一、GPU 碎片的两种类型

### 1 资源碎片（Resource Fragmentation）

GPU 资源被拆成无法满足任务需求的碎片。

例子：

```
节点1: 1 GPU 空闲
节点2: 1 GPU 空闲
节点3: 1 GPU 空闲
节点4: 1 GPU 空闲
```

但任务需要：

```
Job A: 4 GPU
```

虽然总共 4 GPU，但 **无法调度**。

这是 **Kubernetes / Slurm / Ray 集群最常见问题**。

---

### 2 显存碎片（Memory Fragmentation）

GPU Memory 被切碎。

例子：

```
GPU0 Memory 80GB

已用:
10GB
10GB
10GB
10GB
10GB
```

任务需要：

```
连续 40GB
```

但 allocator 找不到连续空间。

在 **LLM inference** 特别严重。

---

# 二、GPU 碎片的 6 大解决方案（工业界）

---

# 1 GPU Packing（任务打包）

核心思想：

**尽量把任务塞满同一节点**

例如：

```
Node1: 8 GPU
Node2: 8 GPU
```

任务：

```
A: 1 GPU
B: 1 GPU
C: 1 GPU
D: 1 GPU
```

调度策略：

坏策略：

```
Node1: A
Node2: B
Node1: C
Node2: D
```

好策略：

```
Node1: A B C D
Node2: empty
```

这样：

未来任务

```
E: 4 GPU
```

还能调度。

---

### Kubernetes scheduler plugin 实现

Score plugin：

```
Score = GPU使用率
```

使用率越高 → score 越高。

---

# 2 Bin Packing Scheduler

类似：

```
First Fit Decreasing
Best Fit
```

策略：

```
优先放入最满的节点
```

数学模型：

```
maximize node_utilization
minimize node_count
```

---

# 3 GPU Defragmentation（重调度）

Google Borg / Kubernetes Volcano 都会做：

**Periodic Rebalance**

步骤：

1 定期检测碎片
2 迁移低优先级任务
3 合并 GPU

例如：

```
Node1: A(1GPU) B(1GPU)
Node2: C(1GPU) D(1GPU)
```

目标：

```
Node1: A B C D
Node2: empty
```

需要：

```
checkpoint / resume
```

LLM inference 很适合。

---

# 4 GPU Slice（MIG）

对于

```
A100
H100
```

可以使用：

**MIG（Multi Instance GPU）**

例如：

```
1x A100 80GB
```

切成：

```
7 x 10GB GPU
```

这样小任务不会浪费 GPU。

---

缺点：

LLM训练一般不用。

---

# 5 显存级调度（Memory Aware Scheduling）

很多系统：

只看 GPU 数量：

```
GPU=1
```

但 LLM inference 其实需要：

```
GPU=1
VRAM=30GB
```

所以 scheduler 应该：

```
GPU
VRAM
SM
NVLink
```

一起调度。

例如：

```
GPU0 memory free 40GB
GPU1 memory free 10GB
```

任务需要：

```
30GB
```

只允许调度 GPU0。

---

# 6 GPU Sharing（Time Slice）

对于小任务：

可以 GPU time slicing：

例如：

```
NVIDIA MPS
CUDA context switch
```

或者

```
vGPU
```

例如：

```
GPU0
 ├ job A 30%
 ├ job B 30%
 └ job C 40%
```

常用于：

```
LLM inference
embedding service
```

---

# 三、LLM 推理平台最常见解决方案

LLM 推理平台通常结合：

```
GPU packing
+
memory aware
+
continuous batching
```

例如：

系统：

```
vLLM
TensorRT-LLM
TGI
```

它们通过：

```
KV Cache manager
continuous batching
```

提高 GPU 利用率。

---

# 四、Kubernetes GPU 碎片解决方案

常见方案：

### 1 Volcano Scheduler

支持：

```
gang scheduling
bin packing
GPU sharing
```

---

### 2 kube-batch

适合 AI 训练。

---

### 3 NVIDIA GPU Operator

提供：

```
MIG
GPU metrics
device plugin
```

---

### 4 自定义 Scheduler Plugin

实现：

```
Score Plugin
Preemption Plugin
Permit Plugin
```

评分逻辑：

```
score = gpu_used_ratio * 0.6
      + memory_used_ratio * 0.4
```

---

# 五、万卡 GPU 集群的真实架构

顶级 AI 公司通常这样做：

```
                Global Scheduler
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   Region Scheduler Region Scheduler Region Scheduler
        │              │              │
  Cluster Scheduler Cluster Scheduler Cluster Scheduler
        │
    Node Scheduler
```

每层解决不同问题：

```
global     -> 多数据中心
cluster    -> bin packing
node       -> memory packing
```

---

# 六、最先进 GPU 调度技术（大厂）

Google / Meta / OpenAI 类似系统：

### 1 Elastic Training

动态增加 GPU：

```
4 GPU -> 8 GPU -> 16 GPU
```

减少碎片。

---

### 2 GPU Topology Aware

考虑：

```
NVLink
PCIe
NUMA
```

否则：

训练性能会掉 50%。

---

### 3 Job Preemption

低优先级任务被杀掉：

```
research job
```

让位给：

```
production inference
```

---

# 七、如果是 LLM 平台（最优解）

我推荐的架构：

```
GPU Scheduler
    │
    ├─ Bin Packing
    ├─ Memory Aware
    ├─ Topology Aware
    ├─ Preemption
    └─ Defragmentation
```

结合：

```
vLLM
Ray
Kubernetes
```

---

# 八、一个真实 GPU 调度算法

调度评分：

```
score(node) =

  0.35 * gpu_utilization
+ 0.25 * memory_utilization
+ 0.20 * topology_score
+ 0.10 * locality
+ 0.10 * job_affinity
```

选择：

```
max(score)
```

---

如果你愿意，我可以再给你讲 **三个非常硬核的内容（AI infra 面试级别）**：

1️⃣ **Kubernetes GPU Scheduler Plugin 完整实现（Go 代码）**
2️⃣ **万卡 GPU 集群调度架构设计（OpenAI / Google 级别）**
3️⃣ **LLM 推理平台完整架构（vLLM + Ray + Kubernetes）**

这三个基本就是 **AI Infra / Staff Engineer 面试核心题**。
