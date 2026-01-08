# Go Rate Limiting Deep Dive - Complete Understanding

## Table of Contents
1. [What is Rate Limiting?](#what-is-rate-limiting)
2. [Why Rate Limiting Matters](#why-rate-limiting-matters)
3. [Rate Limiting Algorithms](#rate-limiting-algorithms)
4. [Go Rate Limiting Implementation](#go-rate-limiting-implementation)
5. [Rate Limiting Patterns](#rate-limiting-patterns)
6. [Best Practices](#best-practices)

---

## What is Rate Limiting?

### Definition

**Rate Limiting**: Controlling the rate of requests or operations to prevent abuse and ensure fair resource usage.

**Key Characteristics:**
- **Request control**: Controls request rate
- **Abuse prevention**: Prevents abuse
- **Resource protection**: Protects resources
- **Fair usage**: Ensures fair usage

### Real-World Analogy

**Rate Limiting = Speed Limit:**
- **Road**: API/service
- **Speed limit**: Rate limit
- **Traffic**: Requests
- **Control**: Control traffic flow

**Programming:**
- **Service**: API or service
- **Rate limit**: Maximum requests per time
- **Requests**: Incoming requests
- **Protection**: Resource protection

---

## Why Rate Limiting Matters?

### Benefits

**1. Abuse Prevention:**
```
Unlimited requests
  ↓
Rate limiting
  ↓
Prevent abuse
```

**2. Resource Protection:**
```
Limited resources
  ↓
Rate limiting
  ↓
Protect resources
```

**3. Fair Usage:**
```
Multiple users
  ↓
Rate limiting
  ↓
Fair resource sharing
```

---

## Rate Limiting Algorithms

### Algorithm 1: Fixed Window

**Fixed time window:**
```go
type FixedWindowLimiter struct {
    limit    int
    window   time.Duration
    requests map[string][]time.Time
    mu       sync.Mutex
}

func (l *FixedWindowLimiter) Allow(key string) bool {
    l.mu.Lock()
    defer l.mu.Unlock()
    
    now := time.Now()
    windowStart := now.Truncate(l.window)
    
    if l.requests[key] == nil {
        l.requests[key] = []time.Time{}
    }
    
    // Remove old requests
    validRequests := []time.Time{}
    for _, t := range l.requests[key] {
        if t.After(windowStart) {
            validRequests = append(validRequests, t)
        }
    }
    l.requests[key] = validRequests
    
    if len(l.requests[key]) >= l.limit {
        return false
    }
    
    l.requests[key] = append(l.requests[key], now)
    return true
}
```

### Algorithm 2: Sliding Window

**Sliding time window:**
```go
type SlidingWindowLimiter struct {
    limit    int
    window   time.Duration
    requests map[string][]time.Time
    mu       sync.Mutex
}

func (l *SlidingWindowLimiter) Allow(key string) bool {
    l.mu.Lock()
    defer l.mu.Unlock()
    
    now := time.Now()
    windowStart := now.Add(-l.window)
    
    if l.requests[key] == nil {
        l.requests[key] = []time.Time{}
    }
    
    // Remove old requests
    validRequests := []time.Time{}
    for _, t := range l.requests[key] {
        if t.After(windowStart) {
            validRequests = append(validRequests, t)
        }
    }
    l.requests[key] = validRequests
    
    if len(l.requests[key]) >= l.limit {
        return false
    }
    
    l.requests[key] = append(l.requests[key], now)
    return true
}
```

### Algorithm 3: Token Bucket

**Token bucket algorithm:**
```go
type TokenBucketLimiter struct {
    capacity     int
    tokens       int
    refillRate   int           // tokens per second
    lastRefill   time.Time
    mu           sync.Mutex
}

func NewTokenBucketLimiter(capacity, refillRate int) *TokenBucketLimiter {
    return &TokenBucketLimiter{
        capacity:   capacity,
        tokens:     capacity,
        refillRate: refillRate,
        lastRefill: time.Now(),
    }
}

func (l *TokenBucketLimiter) Allow() bool {
    l.mu.Lock()
    defer l.mu.Unlock()
    
    now := time.Now()
    elapsed := now.Sub(l.lastRefill)
    
    // Refill tokens
    tokensToAdd := int(elapsed.Seconds()) * l.refillRate
    if tokensToAdd > 0 {
        l.tokens = min(l.capacity, l.tokens+tokensToAdd)
        l.lastRefill = now
    }
    
    if l.tokens > 0 {
        l.tokens--
        return true
    }
    
    return false
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

---

## Go Rate Limiting Implementation

### Using golang.org/x/time/rate

**Standard rate limiter:**
```go
import "golang.org/x/time/rate"

// Create limiter: 10 requests per second
limiter := rate.NewLimiter(10, 10)

// Allow request
if !limiter.Allow() {
    // Rate limit exceeded
    return errors.New("rate limit exceeded")
}

// Wait for token
ctx := context.Background()
err := limiter.Wait(ctx)
if err != nil {
    // Context cancelled or timeout
    return err
}
```

### HTTP Middleware

**Rate limiting middleware:**
```go
func rateLimitMiddleware(limiter *rate.Limiter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            if !limiter.Allow() {
                http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
                return
            }
            next.ServeHTTP(w, r)
        })
    }
}

// Usage
limiter := rate.NewLimiter(10, 10)
mux := http.NewServeMux()
mux.HandleFunc("/", handler)
http.ListenAndServe(":8080", rateLimitMiddleware(limiter)(mux))
```

---

## Rate Limiting Patterns

### Pattern 1: Per-IP Rate Limiting

```go
type PerIPLimiter struct {
    limiters map[string]*rate.Limiter
    mu       sync.RWMutex
    rate     rate.Limit
    burst    int
}

func NewPerIPLimiter(r rate.Limit, burst int) *PerIPLimiter {
    return &PerIPLimiter{
        limiters: make(map[string]*rate.Limiter),
        rate:     r,
        burst:    burst,
    }
}

func (p *PerIPLimiter) GetLimiter(ip string) *rate.Limiter {
    p.mu.RLock()
    limiter, exists := p.limiters[ip]
    p.mu.RUnlock()
    
    if exists {
        return limiter
    }
    
    p.mu.Lock()
    defer p.mu.Unlock()
    
    limiter = rate.NewLimiter(p.rate, p.burst)
    p.limiters[ip] = limiter
    return limiter
}
```

### Pattern 2: Distributed Rate Limiting

```go
// Using Redis for distributed rate limiting
func distributedRateLimit(key string, limit int, window time.Duration) (bool, error) {
    conn := redisPool.Get()
    defer conn.Close()
    
    key := fmt.Sprintf("ratelimit:%s", key)
    
    count, err := redis.Int(conn.Do("INCR", key))
    if err != nil {
        return false, err
    }
    
    if count == 1 {
        conn.Do("EXPIRE", key, int(window.Seconds()))
    }
    
    return count <= limit, nil
}
```

---

## Best Practices

### 1. Choose Right Algorithm

**Why:**
- **Use case**: Depends on use case
- **Performance**: Different performance characteristics
- **Accuracy**: Different accuracy levels

**Guidelines:**
- **Fixed window**: Simple, but can allow bursts
- **Sliding window**: More accurate, smoother
- **Token bucket**: Smooth rate, allows bursts

### 2. Set Appropriate Limits

**Why:**
- **Balance**: Balance between protection and usability
- **Resources**: Consider available resources
- **Users**: Consider user needs

**Guidelines:**
- **Reasonable limits**: Set reasonable limits
- **Different tiers**: Different limits for different users
- **Monitor**: Monitor and adjust limits

### 3. Return Clear Error Messages

**Why:**
- **User experience**: Better user experience
- **Debugging**: Easier debugging
- **Compliance**: API compliance

**Guidelines:**
- **HTTP 429**: Use HTTP 429 (Too Many Requests)
- **Retry-After**: Include Retry-After header
- **Clear message**: Clear error message

### 4. Monitor Rate Limiting

**Why:**
- **Effectiveness**: Monitor effectiveness
- **Adjustment**: Adjust limits as needed
- **Abuse detection**: Detect abuse patterns

**Guidelines:**
- **Metrics**: Track rate limit metrics
- **Alerts**: Set up alerts for abuse
- **Analysis**: Analyze rate limit patterns

---

## Summary

Rate limiting is essential for protecting resources and preventing abuse in Go applications. Understanding rate limiting algorithms, implementations, patterns, and best practices is crucial for effective rate limiting.

**Key Takeaways:**
- **Rate limiting**: Controlling request rate (request control, abuse prevention, resource protection, fair usage)
- **Rate limiting algorithms**: Fixed window (simple, can allow bursts), sliding window (more accurate, smoother), token bucket (smooth rate, allows bursts)
- **Go rate limiting implementation**: golang.org/x/time/rate (standard limiter, HTTP middleware)
- **Rate limiting patterns**: Per-IP rate limiting (per user/IP), distributed rate limiting (Redis, shared state)
- **Best practices**: Choose right algorithm, set appropriate limits, return clear error messages, monitor rate limiting

**Rate Limiting Algorithms:**
- **Fixed window**: Simple, time-based window
- **Sliding window**: More accurate, sliding window
- **Token bucket**: Smooth rate, token-based

**Best Practices:**
- Choose right algorithm
- Set appropriate limits
- Return clear error messages
- Monitor rate limiting

**Next Steps:**
- Learn rate limiting algorithms
- Practice implementations
- Set up rate limiting
- Apply best practices

