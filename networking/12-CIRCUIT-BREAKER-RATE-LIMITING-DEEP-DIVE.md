# Circuit Breaker and Rate Limiting Deep Dive - Complete Understanding

## Table of Contents
1. [Circuit Breaker Pattern](#circuit-breaker-pattern)
2. [Rate Limiting](#rate-limiting)
3. [Throttling](#throttling)
4. [Implementation Examples](#implementation-examples)
5. [Best Practices](#best-practices)

---

## Circuit Breaker Pattern

### What is a Circuit Breaker?

**Circuit Breaker**: Design pattern that prevents cascading failures by stopping requests to a failing service.

**Real-World Analogy:**
- **Electrical circuit breaker**: Stops current when overloaded
- **Software circuit breaker**: Stops requests when service fails
- **Protection**: Prevents damage to system

### How Circuit Breaker Works

**Three States:**

**1. Closed (Normal):**
```
Requests → Service
  ↓
Service responds
  ↓
Circuit stays closed
```

**2. Open (Failing):**
```
Requests → Circuit Breaker
  ↓
Circuit open (blocked)
  ↓
Fail fast (don't call service)
```

**3. Half-Open (Testing):**
```
Circuit Breaker → Test request
  ↓
If succeeds: Close circuit
If fails: Open circuit again
```

### State Transitions

**Visual:**
```
Closed → [Failure threshold reached] → Open
  ↑                                        ↓
  └── [Timeout] ←── Half-Open ←── [Test request]
```

**Example:**
```
Closed State:
  - Requests: 100
  - Failures: 5
  - Failure rate: 5% (below threshold)
  - Status: Normal operation

Failure threshold: 50%
Failures increase: 60 failures
Failure rate: 60% (above threshold)
  ↓
Open State:
  - Requests blocked
  - Fail fast
  - No service calls

After timeout (30 seconds):
  ↓
Half-Open State:
  - Allow one test request
  - If succeeds: Close circuit
  - If fails: Open circuit again
```

### Circuit Breaker Implementation

**Basic Implementation:**
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN
    
    def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            # Check if timeout passed
            if time.time() - self.last_failure_time > self.timeout:
                self.state = "HALF_OPEN"
            else:
                raise CircuitBreakerOpenException("Circuit is open")
        
        try:
            result = func(*args, **kwargs)
            # Success
            if self.state == "HALF_OPEN":
                self.state = "CLOSED"
                self.failure_count = 0
            return result
        except Exception as e:
            # Failure
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
            
            raise e
```

### Benefits

**1. Fail Fast:**
- Don't wait for timeout
- Immediate failure response
- Better user experience

**2. Prevent Cascading Failures:**
- Stop calling failing service
- Protect other services
- System stability

**3. Automatic Recovery:**
- Test service periodically
- Recover when service fixed
- No manual intervention

### When to Use

**Use Circuit Breaker When:**
- Calling external services
- Service may fail
- Need to prevent cascading failures
- Want fail-fast behavior

---

## Rate Limiting

### What is Rate Limiting?

**Rate Limiting**: Controlling the rate of requests from clients or to services.

**Purpose:**
- **Prevent abuse**: Stop malicious users
- **Fair usage**: Ensure fair resource distribution
- **Protect services**: Prevent overload
- **Cost control**: Limit API usage

### Rate Limiting Algorithms

**1. Fixed Window:**

**How It Works:**
```
Time: [0-60s] [60-120s] [120-180s]
Limit: 100 requests per window
Count: Track requests in current window
Reset: At window boundary
```

**Example:**
```
Window: 0-60 seconds
Requests: 50, 60, 70, 80, 90, 100
At 60 seconds: Reset counter
New window: 60-120 seconds
```

**Problem:**
- Burst at window boundary
- Can exceed limit briefly

**2. Sliding Window:**

**How It Works:**
```
Track requests in sliding window
Remove old requests as window slides
More accurate than fixed window
```

**Example:**
```
Current time: 100 seconds
Window: 40-100 seconds (60 second window)
Requests in window: Count requests from 40-100s
```

**Benefits:**
- More accurate
- No burst at boundary
- Smooth rate limiting

**3. Token Bucket:**

**How It Works:**
```
Bucket with tokens
Tokens added at rate (e.g., 10/second)
Request consumes token
If no tokens: Request rejected
```

**Example:**
```
Bucket capacity: 100 tokens
Refill rate: 10 tokens/second
Request: Consumes 1 token

If bucket empty: Reject request
If tokens available: Allow request
```

**Benefits:**
- Allows bursts (up to bucket size)
- Smooth rate limiting
- Flexible

**4. Leaky Bucket:**

**How It Works:**
```
Bucket with requests
Requests leak out at constant rate
If bucket full: Reject request
```

**Example:**
```
Bucket capacity: 100 requests
Leak rate: 10 requests/second
Request arrives: Add to bucket

If bucket full: Reject
If space: Add request, process at leak rate
```

**Benefits:**
- Smooth output rate
- No bursts
- Predictable

### Rate Limiting Implementation

**Token Bucket Example:**
```python
import time
from threading import Lock

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.tokens = capacity
        self.last_refill = time.time()
        self.lock = Lock()
    
    def consume(self, tokens=1):
        with self.lock:
            # Refill tokens
            now = time.time()
            elapsed = now - self.last_refill
            self.tokens = min(
                self.capacity,
                self.tokens + elapsed * self.refill_rate
            )
            self.last_refill = now
            
            # Check if enough tokens
            if self.tokens >= tokens:
                self.tokens -= tokens
                return True
            else:
                return False

# Usage
rate_limiter = TokenBucket(capacity=100, refill_rate=10)

def handle_request():
    if rate_limiter.consume():
        # Process request
        return "OK"
    else:
        return "Rate limit exceeded", 429
```

### Rate Limiting Strategies

**1. Per User:**
```
User A: 100 requests/hour
User B: 100 requests/hour
Independent limits
```

**2. Per IP:**
```
IP 1.2.3.4: 1000 requests/hour
IP 5.6.7.8: 1000 requests/hour
```

**3. Per API Key:**
```
API Key 1: 10000 requests/day
API Key 2: 5000 requests/day
```

**4. Global:**
```
All users: 100000 requests/hour
Shared limit
```

### Rate Limiting Headers

**HTTP Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
Retry-After: 60
```

---

## Throttling

### What is Throttling?

**Throttling**: Limiting the rate of processing requests, not just rejecting them.

**Difference from Rate Limiting:**
- **Rate Limiting**: Reject excess requests
- **Throttling**: Queue excess requests, process slowly

### Throttling Strategies

**1. Queue-Based:**
```
Requests → Queue
  ↓
Process at rate (e.g., 10/second)
  ↓
If queue full: Reject new requests
```

**2. Time-Based:**
```
Request arrives
  ↓
Check last request time
  ↓
If too soon: Wait
  ↓
Process when time allows
```

### Implementation

**Queue-Based Throttling:**
```python
from queue import Queue
import threading

class Throttler:
    def __init__(self, max_rate, queue_size=1000):
        self.max_rate = max_rate  # requests per second
        self.queue = Queue(maxsize=queue_size)
        self.processing = False
        self.lock = threading.Lock()
    
    def add_request(self, request):
        try:
            self.queue.put(request, block=False)
            return True
        except Queue.Full:
            return False  # Reject
    
    def process_requests(self):
        while True:
            if not self.queue.empty():
                request = self.queue.get()
                process(request)
            time.sleep(1.0 / self.max_rate)  # Rate limit
```

---

## Implementation Examples

### Circuit Breaker with Hystrix (Java)

```java
@HystrixCommand(fallbackMethod = "fallback")
public String callService() {
    return service.call();
}

public String fallback() {
    return "Service unavailable";
}
```

### Rate Limiting with Redis

```python
import redis

def rate_limit(user_id, limit=100, window=3600):
    r = redis.Redis()
    key = f"rate_limit:{user_id}"
    
    current = r.incr(key)
    if current == 1:
        r.expire(key, window)
    
    if current > limit:
        return False  # Rate limited
    return True
```

---

## Best Practices

### Circuit Breaker

**1. Tune Thresholds:**
- Failure threshold: Based on acceptable failure rate
- Timeout: Based on recovery time
- Test interval: Balance between recovery and load

**2. Monitor:**
- Track state changes
- Monitor failure rates
- Alert on circuit opens

**3. Fallback:**
- Provide fallback response
- Cache previous responses
- Degrade gracefully

### Rate Limiting

**1. Choose Algorithm:**
- Fixed window: Simple, fast
- Sliding window: More accurate
- Token bucket: Allows bursts
- Leaky bucket: Smooth output

**2. Set Appropriate Limits:**
- Based on capacity
- Consider use cases
- Allow legitimate usage
- Prevent abuse

**3. Communicate Limits:**
- Return rate limit headers
- Clear error messages
- Document limits

**4. Handle Gracefully:**
- Return 429 (Too Many Requests)
- Include retry-after header
- Don't crash on limit

---

## Summary

Circuit breaker and rate limiting are essential patterns for building resilient, protected systems.

**Key Takeaways:**
- Circuit Breaker: Prevents cascading failures, three states (closed, open, half-open)
- Rate Limiting: Controls request rate, algorithms (fixed window, sliding window, token bucket, leaky bucket)
- Throttling: Queues requests, processes at rate
- Implementation: Use libraries or implement custom
- Best practices: Tune thresholds, monitor, provide fallbacks, communicate limits

**Next Steps:**
- Implement circuit breaker for external calls
- Add rate limiting to APIs
- Monitor and tune
- Test failure scenarios
- Document limits

