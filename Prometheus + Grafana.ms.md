你这个问题我按 **“Golang 项目里怎么落地 Prometheus + Grafana”** 给你一套生产级方案，适合你这种做分布式/云原生的后端场景。

我会讲：

1. 整体架构
2. Go 项目如何接入 Prometheus
3. 自定义业务指标设计
4. Kubernetes 部署
5. Grafana 面板设计
6. 生产优化建议

---

# 一、整体架构

典型架构：

```
Golang Service
     |
     |  /metrics (HTTP)
     ↓
Prometheus (pull)
     ↓
Grafana (query)
```

Prometheus = 负责采集 + 存储
Grafana = 负责可视化
Go 服务 = 暴露 metrics endpoint

---

# 二、Golang 接入 Prometheus

使用官方库：

```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promhttp
```

---

## 1️⃣ 最基础接入

```go
package main

import (
	"net/http"

	"github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
	http.Handle("/metrics", promhttp.Handler())
	http.ListenAndServe(":8080", nil)
}
```

启动后：

```
http://localhost:8080/metrics
```

---

# 三、添加业务指标（重点）

Prometheus 四种核心类型：

| 类型        | 用途         |
| --------- | ---------- |
| Counter   | 只增不减（请求数）  |
| Gauge     | 可增可减（在线人数） |
| Histogram | 延迟统计       |
| Summary   | 延迟百分位      |

---

## 1️⃣ 请求总数 Counter

```go
var httpRequests = prometheus.NewCounterVec(
	prometheus.CounterOpts{
		Name: "http_requests_total",
		Help: "Total number of HTTP requests",
	},
	[]string{"method", "path", "status"},
)

func init() {
	prometheus.MustRegister(httpRequests)
}
```

使用：

```go
httpRequests.WithLabelValues("GET", "/user", "200").Inc()
```

---

## 2️⃣ 请求耗时 Histogram（推荐）

```go
var httpDuration = prometheus.NewHistogramVec(
	prometheus.HistogramOpts{
		Name:    "http_request_duration_seconds",
		Help:    "Request latency",
		Buckets: prometheus.DefBuckets, // 可自定义
	},
	[]string{"method", "path"},
)
```

使用方式：

```go
timer := prometheus.NewTimer(
	httpDuration.WithLabelValues("GET", "/user"),
)
defer timer.ObserveDuration()
```

---

## 3️⃣ 在线人数 Gauge

```go
var onlineUsers = prometheus.NewGauge(
	prometheus.GaugeOpts{
		Name: "online_users",
		Help: "Current online users",
	},
)
```

```go
onlineUsers.Inc()
onlineUsers.Dec()
```

---

# 四、生产级 Middleware（推荐写法）

真实项目你应该做一个 HTTP 中间件自动统计：

```go
func MetricsMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {

		start := time.Now()

		rw := &responseWriter{ResponseWriter: w, statusCode: 200}

		next.ServeHTTP(rw, r)

		duration := time.Since(start).Seconds()

		httpRequests.WithLabelValues(
			r.Method,
			r.URL.Path,
			strconv.Itoa(rw.statusCode),
		).Inc()

		httpDuration.WithLabelValues(
			r.Method,
			r.URL.Path,
		).Observe(duration)
	})
}
```

这样你所有接口自动被监控。

---

# 五、Prometheus 配置

prometheus.yml

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'golang-app'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

如果是 Kubernetes：

```yaml
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
    - role: pod
```

生产推荐用：

* kube-prometheus-stack
* Prometheus Operator

---

# 六、Grafana 配置

1. 添加数据源 → Prometheus
2. 创建 Dashboard
3. 常用查询：

## QPS

```promql
rate(http_requests_total[1m])
```

---

## P95 延迟

```promql
histogram_quantile(0.95,
  rate(http_request_duration_seconds_bucket[5m])
)
```

---

## 错误率

```promql
rate(http_requests_total{status=~"5.."}[1m])
/
rate(http_requests_total[1m])
```

---

# 七、云原生部署方案（你这种背景推荐）

如果你在做 Kubernetes：

### 方式 1：Helm

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

自动部署：

* Prometheus
* Alertmanager
* Grafana
* Node Exporter

---

### 方式 2：ServiceMonitor

如果你用 Prometheus Operator：

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
```

自动发现 Go 服务。

---

# 八、生产级优化建议（高级）

## 1️⃣ 避免 label 爆炸

错误示例：

```
path = /user/123
path = /user/124
```

正确方式：

```
/user/:id
```

否则 Prometheus 会被打爆。

---

## 2️⃣ 指标命名规范

标准：

```
service_subsystem_metric_unit
```

例：

```
order_payment_duration_seconds
```

---

## 3️⃣ 不要乱用 Summary

Summary 无法跨实例聚合。

生产推荐：

```
Histogram + histogram_quantile()
```

---

## 4️⃣ 高并发优化

* 不要在高频路径频繁创建 metric
* metric 必须全局注册一次
* label value 数量要有限

---

# 九、完整企业级架构

```
Golang Services
     ↓
Prometheus
     ↓
Thanos（长期存储）
     ↓
Grafana
     ↓
Alertmanager → Slack/Email
```

大规模系统一般都会加：

* Thanos
* VictoriaMetrics
* Loki（日志）
* Tempo（链路追踪）

---

# 十、如果你是写简历

你可以写：

> 设计并实现基于 Prometheus + Grafana 的微服务监控体系，构建自定义业务指标（QPS、P95、错误率、队列堆积、AI任务耗时），支持 Kubernetes 自动服务发现与告警策略，实现生产环境高可观测性闭环。

---

如果你愿意，我可以给你：

* ✅ 一套完整 Golang + Prometheus + Grafana Docker Compose 示例
* ✅ 一套 Kubernetes 生产部署示例
* ✅ 一套微服务级监控指标设计规范
* ✅ 一套高并发 AI 系统监控指标设计

你想往哪种场景深入？
单体服务？微服务？还是 AI 分布式系统？


在 **Golang 项目中集成 Prometheus + Grafana** 是云原生监控的标准做法，尤其适合你这种做高并发 / 分布式 / K8s 的后端系统。

下面给你一个**从 0 到 1 的完整实践方案**（生产级思路）。

---

# 一、整体架构

```
Go Service  --->  Prometheus  --->  Grafana
       |                |
   /metrics        Time Series DB
```

* **Prometheus**：负责抓取指标
* **Grafana**：负责可视化
* Go 程序通过 `/metrics` 暴露指标

---

# 二、Golang 接入 Prometheus

使用官方库：

```
github.com/prometheus/client_golang/prometheus
```

## 1️⃣ 安装

```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promhttp
```

---

## 2️⃣ 最基础示例

```go
package main

import (
	"net/http"
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
	httpRequestsTotal = prometheus.NewCounter(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Total number of HTTP requests",
		},
	)
)

func main() {
	prometheus.MustRegister(httpRequestsTotal)

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		httpRequestsTotal.Inc()
		w.Write([]byte("Hello World"))
	})

	http.Handle("/metrics", promhttp.Handler())

	http.ListenAndServe(":8080", nil)
}
```

访问：

```
http://localhost:8080/metrics
```

---

# 三、常见指标类型（必须熟练）

### 1️⃣ Counter（计数器）

* 单调递增
* 用于 QPS、错误数

```go
counter := prometheus.NewCounter(...)
counter.Inc()
```

---

### 2️⃣ Gauge（可增可减）

* 当前在线人数
* 内存使用

```go
gauge := prometheus.NewGauge(...)
gauge.Set(100)
gauge.Inc()
gauge.Dec()
```

---

### 3️⃣ Histogram（生产必备）

用于延迟统计

```go
histogram := prometheus.NewHistogram(
	prometheus.HistogramOpts{
		Name:    "http_duration_seconds",
		Buckets: prometheus.DefBuckets,
	},
)

histogram.Observe(0.25)
```

---

### 4️⃣ Summary（一般不推荐）

分位数统计，但在多实例场景不易聚合。

生产建议：**优先 Histogram**

---

# 四、Gin 项目中优雅集成（生产推荐）

如果你用 Gin：

```go
r := gin.Default()

r.GET("/metrics", gin.WrapH(promhttp.Handler()))
```

更专业方式：写 middleware

```go
func PrometheusMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		c.Next()
		duration := time.Since(start).Seconds()
		httpDuration.Observe(duration)
	}
}
```

---

# 五、Prometheus 配置

`prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "go-service"
    static_configs:
      - targets: ["host.docker.internal:8080"]
```

启动：

```bash
docker run -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

访问：

```
http://localhost:9090
```

---

# 六、Grafana 配置

启动：

```bash
docker run -p 3000:3000 grafana/grafana
```

访问：

```
http://localhost:3000
```

默认账号：

```
admin / admin
```

添加 DataSource：

* 类型：Prometheus
* URL：`http://host.docker.internal:9090`

---

# 七、生产级最佳实践（重点）

## 1️⃣ 指标命名规范

遵循 Prometheus 规范：

```
<namespace>_<subsystem>_<metric>_<unit>
```

示例：

```
app_http_requests_total
app_http_request_duration_seconds
```

---

## 2️⃣ 使用 Label（非常重要）

```go
httpRequests := prometheus.NewCounterVec(
	prometheus.CounterOpts{
		Name: "http_requests_total",
	},
	[]string{"method", "path", "status"},
)

httpRequests.WithLabelValues("GET", "/login", "200").Inc()
```

⚠️ 注意：

* ❌ 不要用高基数 label（比如 user_id）
* ❌ 不要把动态 path 当 label

否则 Prometheus 会爆内存。

---

## 3️⃣ 监控必须包含

在生产系统（分布式 / AI / K8s）中建议监控：

### 应用层

* QPS
* 请求延迟 P95 / P99
* 错误率
* Goroutine 数量
* GC 时间
* 内存使用

Go runtime metrics 自动支持：

```go
import _ "github.com/prometheus/client_golang/prometheus/promhttp"
```

---

## 4️⃣ K8s 场景

如果你部署在 Kubernetes：

* 使用 ServiceMonitor（配合 Prometheus Operator）
* 或使用 annotations

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
```

---

# 八、企业级架构升级

如果你做的是：

* AI Agent 系统
* 分布式微服务
* 实时通信
* 多模型推理

推荐升级架构：

```
Go Service
    ↓
Prometheus
    ↓
Thanos / Cortex（分布式存储）
    ↓
Grafana
```

用于：

* 长期存储
* 多集群
* 跨 Region

---

# 九、告警（必须有）

Prometheus AlertManager：

示例规则：

```yaml
- alert: HighErrorRate
  expr: rate(http_requests_total{status="500"}[5m]) > 0.05
  for: 2m
  labels:
    severity: critical
```

---

# 十、面试加分点

如果你写简历可以写：

> 构建基于 Prometheus + Grafana 的可观测体系，监控 QPS、P99 延迟、GC、Goroutine 数，设计高基数防爆策略，实现生产级告警系统。

---

# 🚀 结合你背景的进阶建议

你之前做：

* 分布式
* AI 多 Agent
* K8s 部署

建议增加：

### 1️⃣ OpenTelemetry

统一：

* Metrics
* Trace
* Log

未来趋势是 OTEL + Prometheus。

---

# 如果你愿意，我可以给你：

* ✅ 一个完整 docker-compose 示例
* ✅ 一个企业级监控项目模板
* ✅ 一个适合写进高级 Golang 简历的监控项目描述
* ✅ 帮你设计一个 AI Agent 系统监控方案

你想往**实战代码方向**还是**架构设计方向**深入？
