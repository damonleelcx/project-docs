AI Agent Infra（AI Agent Infrastructure）面试一般会考 **LLM + 分布式系统 + AI工程 + 系统设计**。如果你背景是 **Golang / distributed systems / Kubernetes / system design**（从你最近问的问题看很像），面试通常会围绕 **Agent runtime、tooling、memory、orchestration、scaling**。

我给你整理一个 **AI Agent Infra 面试八股体系**（比较接近大厂/AI startup）。

---

# 1 AI Agent 基础概念

首先一定会问：

### 什么是 AI Agent

AI Agent = **LLM + Tool Use + Memory + Planning + Execution loop**

典型结构：

```
User Input
    ↓
LLM Reasoning (Plan)
    ↓
Tool Selection
    ↓
Tool Execution
    ↓
Observation
    ↓
LLM Next Step
```

核心是 **ReAct loop**

```
Thought → Action → Observation → Thought → ...
```

常见框架：

- LangChain
- AutoGPT
- CrewAI
- Semantic Kernel

---

# 2 AI Agent Infra 架构

面试最常考：**设计一个 agent platform**

典型架构：

```
                +------------------+
User Request →  |  API Gateway     |
                +------------------+
                        |
                        v
                +------------------+
                | Agent Runtime    |
                | (Planner Loop)   |
                +------------------+
                     |      |
         +-----------+      +-----------+
         v                          v
+------------------+        +-------------------+
| Tool Registry    |        | Memory System     |
| (APIs, DB, etc)  |        | Vector DB         |
+------------------+        +-------------------+
         |
         v
+------------------+
| Tool Execution   |
| (Sandbox / RPC)  |
+------------------+

```

核心模块：

1️⃣ Agent Runtime
2️⃣ Tool system
3️⃣ Memory system
4️⃣ LLM gateway
5️⃣ Observability

---

# 3 Agent Runtime (核心考点)

很多公司会问：

**Agent runtime 如何设计？**

关键组件：

```
Agent Runtime
├── Planner
├── Tool Selector
├── Executor
├── Memory Manager
└── Context Builder
```

Loop：

```
while not done:
    context = build_context()
    plan = LLM(context)
    action = parse(plan)
    result = execute(action)
    memory.store(result)
```

关键问题：

### 1 context window overflow

解决：

- summarization
- retrieval memory
- sliding window

---

### 2 deterministic tool execution

LLM 只负责：

```
decide tool
```

Infra 负责：

```
execute tool
```

Example：

```
LLM output:
{
  "tool": "search",
  "args": {"query":"golang grpc"}
}
```

Runtime：

```
toolRegistry["search"](args)
```

---

# 4 Tooling System（重要）

面试常问：

**如何设计 tool system？**

Tool registry：

```
Tool {
   name
   description
   schema
   handler
}
```

Example

```
weather(city string) -> temp
```

注册：

```
registry.register(weather)
```

LLM通过schema调用。

标准协议：

- OpenAI function calling
- Model Context Protocol

---

# 5 Memory System（重点）

Agent memory分3类：

### 1 Short-term memory

当前对话 context

```
chat history
```

---

### 2 Long-term memory

vector DB

常见：

- Pinecone
- Weaviate
- Milvus

流程：

```
text → embedding → vector DB
```

query：

```
query → embedding → similarity search
```

---

### 3 Episodic memory

记录 agent 行为：

```
task
actions
results
```

用于：

```
self improvement
```

---

# 6 Multi-Agent System

很多 startup 会问：

**如何设计 multi-agent system？**

结构：

```
User
  ↓
Manager Agent
  ↓
+-----------+-----------+
Research    Coding      QA
Agent       Agent       Agent
```

常见模式：

1️⃣ hierarchical agents
2️⃣ peer-to-peer agents
3️⃣ blackboard architecture

挑战：

- communication
- deadlock
- coordination

---

# 7 Scaling AI Agents（infra重点）

如果是 infra 岗位，一定会问：

### 如何 scale agent system？

关键点：

#### 1 stateless runtime

```
Agent runtime = stateless
Memory = external
```

#### 2 task queue

```
Agent request
    ↓
Kafka / Redis Queue
    ↓
Workers
```

常见：

- Apache Kafka
- Redis

---

#### 3 LLM gateway

统一调用：

```
OpenAI
Claude
local models
```

Gateway：

```
rate limit
retry
fallback
```

---

# 8 Observability

Agent系统很难debug。

需要：

```
trace
token usage
tool calls
latency
```

常见工具：

- LangSmith
- OpenTelemetry

---

# 9 常见 System Design 面试题

AI agent infra 面试最常见题：

### 1

Design ChatGPT with tool use

---

### 2

Design autonomous coding agent

类似：

- Devin

---

### 3

Design AI research agent

能力：

```
search
read
summarize
cite
```

---

### 4

Design multi-agent workflow system

类似：

- CrewAI

---

# 10 Infra 面试加分点（非常关键）

很多 candidate 不会讲这几个：

### Sandbox tool execution

防止 agent 乱执行：

```
docker sandbox
firecracker vm
```

---

### Deterministic replay

agent debugging：

```
replay same prompt
replay same tool calls
```

---

### Guardrails

防止 hallucination：

```
tool validation
schema validation
policy engine
```

---

# 11 面试官最喜欢问的 5 个问题

1️⃣ **Agent vs Workflow difference**

Agent

```
dynamic decision
```

Workflow

```
static DAG
```

---

2️⃣ **How to reduce hallucination**

- RAG
- tool use
- validation

---

3️⃣ **How to manage context window**

- summarization
- retrieval
- memory layers

---

4️⃣ **How to evaluate agents**

metrics：

```
success rate
tool accuracy
latency
cost
```

---

5️⃣ **How to debug agents**

需要：

```
trace
state snapshot
replay
```

---

💡如果你准备 **AI Infra / AI Agent Infra / AI startup backend 面试**，我可以再给你一份 **更硬核的面试版本**：

- **AI Agent system design（完整设计题）**
- **如何实现 LangChain-like framework**
- **如何自己写 Agent runtime（Golang版本）**
- **AI Agent infra 高频面试题 30 个**

基本是 **OpenAI / Anthropic / AI startup / infra team** 那一套。
