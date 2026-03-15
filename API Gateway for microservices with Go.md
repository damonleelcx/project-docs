Building an **API Gateway for microservices with Go** is a great systems design + backend project. A gateway typically handles:

* Routing requests to services
* Authentication / authorization
* Rate limiting
* Logging
* Load balancing
* Request aggregation
* Circuit breaking
* Observability

Below is a **clean architecture approach** and a **minimal implementation** you can extend.

---

# 1. Basic Architecture

```
Client
   |
   v
+------------------+
|    API Gateway   |
|------------------|
| Auth Middleware  |
| Rate Limiter     |
| Router           |
| Load Balancer    |
| Logging          |
+------------------+
      |       |
      v       v
+---------+ +---------+
| UserSvc | | OrderSvc|
+---------+ +---------+
```

Example routing:

```
/users/*   -> user-service
/orders/*  -> order-service
/payments/* -> payment-service
```

---

# 2. Recommended Go Stack

| Feature           | Library                  |
| ----------------- | ------------------------ |
| HTTP framework    | `gin` or `chi`           |
| Reverse proxy     | `net/http/httputil`      |
| Service discovery | `consul` / `etcd`        |
| Rate limit        | `golang.org/x/time/rate` |
| Auth              | `jwt-go`                 |
| Logging           | `zap`                    |
| Tracing           | `OpenTelemetry`          |

---

# 3. Project Structure

```
api-gateway/
│
├── cmd/
│   └── gateway/
│       └── main.go
│
├── internal/
│   ├── router/
│   │   └── router.go
│   │
│   ├── proxy/
│   │   └── reverse_proxy.go
│   │
│   ├── middleware/
│   │   ├── auth.go
│   │   ├── ratelimit.go
│   │   └── logging.go
│   │
│   └── config/
│       └── config.go
│
└── go.mod
```

---

# 4. Reverse Proxy (Core of Gateway)

Go provides a built-in reverse proxy.

### reverse_proxy.go

```go
package proxy

import (
	"net/http"
	"net/http/httputil"
	"net/url"
)

func NewReverseProxy(target string) http.Handler {
	url, _ := url.Parse(target)

	proxy := httputil.NewSingleHostReverseProxy(url)

	originalDirector := proxy.Director

	proxy.Director = func(req *http.Request) {
		originalDirector(req)
		req.Host = url.Host
	}

	return proxy
}
```

---

# 5. Router

### router.go

```go
package router

import (
	"net/http"

	"github.com/go-chi/chi/v5"
	"gateway/internal/proxy"
)

func NewRouter() http.Handler {

	r := chi.NewRouter()

	userService := proxy.NewReverseProxy("http://localhost:8081")
	orderService := proxy.NewReverseProxy("http://localhost:8082")

	r.Mount("/users", userService)
	r.Mount("/orders", orderService)

	return r
}
```

---

# 6. Rate Limiting Middleware

### ratelimit.go

```go
package middleware

import (
	"net/http"
	"sync"

	"golang.org/x/time/rate"
)

var visitors = make(map[string]*rate.Limiter)
var mu sync.Mutex

func getLimiter(ip string) *rate.Limiter {
	mu.Lock()
	defer mu.Unlock()

	limiter, exists := visitors[ip]
	if !exists {
		limiter = rate.NewLimiter(5, 10)
		visitors[ip] = limiter
	}

	return limiter
}

func RateLimit(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {

		ip := r.RemoteAddr
		limiter := getLimiter(ip)

		if !limiter.Allow() {
			http.Error(w, "rate limit exceeded", http.StatusTooManyRequests)
			return
		}

		next.ServeHTTP(w, r)
	})
}
```

---

# 7. JWT Authentication Middleware

### auth.go

```go
package middleware

import (
	"net/http"
	"strings"

	"github.com/golang-jwt/jwt/v5"
)

var secret = []byte("secret")

func JWTAuth(next http.Handler) http.Handler {

	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {

		auth := r.Header.Get("Authorization")

		if auth == "" {
			http.Error(w, "missing token", http.StatusUnauthorized)
			return
		}

		tokenString := strings.Split(auth, " ")[1]

		_, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
			return secret, nil
		})

		if err != nil {
			http.Error(w, "invalid token", http.StatusUnauthorized)
			return
		}

		next.ServeHTTP(w, r)
	})
}
```

---

# 8. Main Entry

### main.go

```go
package main

import (
	"log"
	"net/http"

	"gateway/internal/router"
)

func main() {

	r := router.NewRouter()

	log.Println("API Gateway running on :8080")

	err := http.ListenAndServe(":8080", r)
	if err != nil {
		log.Fatal(err)
	}
}
```

---

# 9. Example Request Flow

```
Client
  |
  | GET /users/123
  |
API Gateway :8080
  |
  | middleware (auth, rate limit)
  |
Reverse Proxy
  |
  v
User Service :8081
```

---

# 10. Advanced Features (Production)

### Service Discovery

Instead of static URLs:

```
user-service -> consul lookup
```

Libraries:

* Consul
* Etcd
* Kubernetes API

---

### Load Balancing

```
users -> [svc1, svc2, svc3]
```

Algorithm:

* round robin
* least connections
* weighted

---

### Circuit Breaker

Use

```
github.com/sony/gobreaker
```

Prevents cascading failures.

---

### Request Aggregation

Example:

```
GET /dashboard

gateway calls:
- user service
- order service
- payment service
```

Returns one combined response.

---

### Observability

Add:

* Prometheus metrics
* OpenTelemetry tracing
* Structured logs

---

# 11. Real-World Gateways (for reference)

Well-known gateways include:

* Kong
* Envoy
* NGINX
* Traefik

Go-based gateway examples:

* KrakenD
* Oathkeeper

---

# 12. Advanced Version (Good for Interviews)

A **production-level Go API Gateway** could include:

```
API Gateway
 ├── dynamic routing (etcd/consul)
 ├── JWT authentication
 ├── RBAC authorization
 ├── distributed rate limiting (Redis)
 ├── circuit breaker
 ├── request retries
 ├── caching layer
 ├── metrics + tracing
 └── hot reload config
```

---

✅ If you'd like, I can also show you a **"Senior-level / FAANG-style API Gateway in Go" architecture** that includes:

* dynamic service discovery
* load balancing
* distributed rate limiting
* circuit breaker
* retry
* observability

This is the **kind of system design that gets asked in infra / platform interviews.**
