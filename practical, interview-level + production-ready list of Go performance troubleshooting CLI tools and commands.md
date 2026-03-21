Here’s a **practical, interview-level + production-ready list** of Go performance troubleshooting CLI tools and commands. I’ll group them by **what problem you’re diagnosing**, not just list them randomly.

---

# 🧠 1. Built-in Go Profiling (pprof) — THE MOST IMPORTANT

### 🔹 Enable in your app

```go
import _ "net/http/pprof"

go func() {
    http.ListenAndServe("localhost:6060", nil)
}()
```

---

## 🔸 CPU profiling

```bash
go tool pprof http://localhost:6060/debug/pprof/profile
```

- Runs ~30 seconds CPU sampling
- Shows **where CPU time is spent**

Useful commands inside pprof:

```bash
top        # hottest functions
list foo   # show source for function
web        # graph (requires graphviz)
```

---

## 🔸 Heap (memory) profiling

```bash
go tool pprof http://localhost:6060/debug/pprof/heap
```

- Shows **memory allocation hotspots**
- Find leaks / excessive allocations

---

## 🔸 Goroutine dump

```bash
curl http://localhost:6060/debug/pprof/goroutine?debug=2
```

- Shows all goroutines + stack traces
- Detect:
  - deadlocks
  - goroutine leaks
  - blocking calls

---

## 🔸 Block profiling

```bash
go tool pprof http://localhost:6060/debug/pprof/block
```

- Shows **where goroutines are blocked**
- Useful for:
  - lock contention
  - channel blocking

---

## 🔸 Mutex profiling

```bash
go tool pprof http://localhost:6060/debug/pprof/mutex
```

- Shows **lock contention hotspots**

Enable it:

```go
runtime.SetMutexProfileFraction(1)
```

---

## 🔸 Trace (VERY powerful)

```bash
curl -o trace.out http://localhost:6060/debug/pprof/trace?seconds=5
go tool trace trace.out
```

- Full timeline:
  - goroutine scheduling
  - GC pauses
  - network blocking

👉 Best tool for **deep latency debugging**

---

# ⚙️ 2. Runtime Debugging (no pprof UI)

## 🔸 GC stats

```bash
GODEBUG=gctrace=1 ./app
```

Shows:

- GC frequency
- pause time
- heap size

---

## 🔸 Scheduler tracing

```bash
GODEBUG=schedtrace=1000 ./app
```

- Dumps scheduler state every 1s
- Shows:
  - runnable goroutines
  - threads (M)
  - processors (P)

---

## 🔸 Combined

```bash
GODEBUG=gctrace=1,schedtrace=1000 ./app
```

---

# 🔬 3. Benchmarking & Micro Performance

## 🔸 Run benchmarks

```bash
go test -bench=.
```

---

## 🔸 Memory allocation stats

```bash
go test -bench=. -benchmem
```

Shows:

- allocs/op
- bytes/op

---

## 🔸 CPU profiling in benchmark

```bash
go test -bench=. -cpuprofile=cpu.out
go tool pprof cpu.out
```

---

## 🔸 Memory profiling in benchmark

```bash
go test -bench=. -memprofile=mem.out
go tool pprof mem.out
```

---

# 🔍 4. Binary / Compilation Analysis

## 🔸 Escape analysis (VERY IMPORTANT)

```bash
go build -gcflags="-m"
```

- Shows:
  - heap vs stack allocation
  - hidden allocations

---

## 🔸 Inlining decisions

```bash
go build -gcflags="-m -m"
```

---

## 🔸 Assembly inspection

```bash
go tool compile -S file.go
```

- Used for extreme optimization cases

---

# 📊 5. System-Level Tools (Outside Go)

These are critical in real production debugging.

---

## 🔸 top / htop

```bash
top
htop
```

- CPU usage
- memory usage
- thread count

---

## 🔸 perf (Linux)

```bash
perf top
perf record ./app
perf report
```

- Kernel + user-level CPU profiling
- Useful when Go pprof isn’t enough

---

## 🔸 strace

```bash
strace -p <pid>
```

- System calls tracing
- Detect:
  - I/O blocking
  - network delays

---

## 🔸 lsof

```bash
lsof -p <pid>
```

- Open files / sockets
- Detect FD leaks

---

## 🔸 netstat / ss

```bash
ss -lntp
netstat -anp
```

- Network connections
- backlog issues

---

# 🌐 6. Go Execution Tracing (CLI-style)

## 🔸 Run with trace file

```bash
go test -trace trace.out
go tool trace trace.out
```

---

# 🚨 7. Race Condition Detection

```bash
go run -race main.go
go test -race
```

- Detects:
  - data races
  - concurrent writes

---

# 🧪 8. Build & Runtime Info

## 🔸 Go environment

```bash
go env
```

---

## 🔸 Module dependencies

```bash
go mod graph
```

---

## 🔸 Binary size analysis

```bash
go build -o app
ls -lh app
```

---

# 🧭 How to Think (Important)

When debugging performance:

### 🔥 CPU high

→ `pprof profile` → `top`

### 🧠 Memory high

→ `heap profile` + `-benchmem`

### 🧵 Goroutines growing

→ `goroutine dump`

### 🔒 Latency / blocking

→ `trace` + `block profile`

### 🧊 Lock contention

→ `mutex profile`

### 🧬 GC issues

→ `GODEBUG=gctrace=1`

---

# 🧠 Interview Summary (1-liner)

> “In Go, performance troubleshooting mainly revolves around **pprof (CPU/heap/block/mutex), runtime debug flags (GODEBUG), benchmarking tools, and system-level tools like perf/strace**.”

---

If you want, I can go one level deeper and give you:

- 🔥 **Real production debugging playbook (step-by-step)**
- 🔥 **Common performance bugs + how to detect them fast**
- 🔥 **pprof graph reading tricks (what most people don’t know)**

In Go, **逃逸分析 (escape analysis)** is done at **compile time**, and the main CLI command is:

---

# ✅ Core Command

```bash
go build -gcflags="-m"
```

---

# 🔍 What it does

- Shows whether variables are allocated on:
  - **stack** (fast, preferred)
  - **heap** (slower, GC involved)

---

# 🧠 Example Output

```bash
./main.go:10:6: moved to heap: x
./main.go:15:9: &y escapes to heap
```

### Meaning:

- `moved to heap`: compiler decided it must live on heap
- `escapes to heap`: variable escapes current function scope

---

# 🚀 More Detailed Version (Recommended)

```bash
go build -gcflags="-m -m"
```

👉 Adds:

- why it escapes
- inlining decisions
- deeper compiler reasoning

---

# 🔬 Even Better (for large projects)

```bash
go build -gcflags="all=-m"
```

or

```bash
go build -gcflags="all=-m -m"
```

👉 Ensures:

- **all packages** (including dependencies) are analyzed

---

# ⚡ When to Use

Use escape analysis when you suspect:

- too many allocations
- GC pressure
- unexpected performance drop

---

# 🧠 Quick Mental Model

A variable **escapes to heap if**:

- returned from function
- referenced by pointer outside scope
- captured by closure
- stored in interface
- used in goroutine

---

# 🔥 Interview Answer (1-liner)

> “Use `go build -gcflags="-m"` to inspect escape analysis and identify heap allocations.”

---

If you want, I can show:

- 🔥 real code examples that **force escape vs avoid escape**
- 🔥 tricks to reduce heap allocations (very interview-relevant)
