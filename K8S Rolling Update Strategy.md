To ensure **user requests are not interrupted during a Kubernetes Pod deployment**, you need to combine **rolling updates, readiness checks, graceful shutdown, and connection draining**. These mechanisms ensure old Pods continue serving traffic until new ones are ready and existing requests finish.

Below is the typical production setup.

---

# 1. Use Rolling Update Strategy

The default Deployment strategy in Kubernetes is **RollingUpdate**, which replaces Pods gradually.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

**Meaning**

| Setting             | Effect                                    |
| ------------------- | ----------------------------------------- |
| `maxUnavailable: 0` | Never reduce available pods               |
| `maxSurge: 1`       | Create 1 extra pod before terminating old |

Result:

- New Pod starts
- Wait until ready
- Then terminate old Pod

This ensures **no capacity drop**.

---

# 2. Use Readiness Probe

A Pod should **not receive traffic until it is ready**.

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

Flow:

```
New Pod created
        ↓
Container starts
        ↓
Readiness probe passes
        ↓
Pod added to Service endpoints
        ↓
Receives traffic
```

Without this, Kubernetes may route traffic **before the app is ready**.

---

# 3. Graceful Shutdown with terminationGracePeriodSeconds

When Kubernetes stops a Pod:

1. Remove Pod from Service endpoints
2. Send **SIGTERM**
3. Wait `terminationGracePeriodSeconds`
4. Kill container

Example:

```yaml
spec:
  terminationGracePeriodSeconds: 30
```

Your app should handle **SIGTERM** to finish requests.

Example (Go):

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()

go func() {
    <-ctx.Done()
    log.Println("shutting down server")

    shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    server.Shutdown(shutdownCtx)
}()
```

This allows active HTTP requests to finish.

---

# 4. Use preStop Hook (Connection Draining)

Even after removal from Service, **load balancers may still send traffic briefly**.

Add a delay before termination.

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sleep", "10"]
```

Flow:

```
Pod marked terminating
↓
Removed from Service endpoints
↓
sleep 10s (drain traffic)
↓
SIGTERM sent
↓
Pod shutdown
```

---

# 5. Run Multiple Replicas

Always deploy **more than 1 Pod**.

```yaml
replicas: 3
```

Otherwise a rolling update will cause downtime.

---

# 6. PodDisruptionBudget (optional but recommended)

Prevents too many pods from disappearing.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  minAvailable: 2
```

---

# Complete Production Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    spec:
      terminationGracePeriodSeconds: 30

      containers:
        - name: api
          image: my-api:latest

          readinessProbe:
            httpGet:
              path: /health
              port: 8080

          lifecycle:
            preStop:
              exec:
                command: ["sleep", "10"]
```

---

# Deployment Timeline (What Happens)

```
Old Pod (serving traffic)
        │
        │ Deploy new version
        ↓
New Pod starts
        ↓
Readiness probe passes
        ↓
Traffic starts going to new Pod
        ↓
Old Pod removed from endpoints
        ↓
preStop executes (drain)
        ↓
SIGTERM sent
        ↓
Finish in-flight requests
        ↓
Pod exits
```

Result: **zero request interruption**.

---

💡 **Production tip:**
If you're running behind **NGINX / Envoy / Istio**, they also support **connection draining**, which makes this even safer.

---

If you'd like, I can also explain **how large companies prevent request drops during Kubernetes deploys (Google / Uber style)** — it involves **readiness gates, load balancer draining, and surge rollouts**.
