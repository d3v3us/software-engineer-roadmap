# Retry and Timeout Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [Why Retry and Timeout?](#why-retry-and-timeout)
2. [Retry Patterns](#retry-patterns)
3. [Exponential Backoff](#exponential-backoff)
4. [Jitter in Retries](#jitter-in-retries)
5. [Timeout Patterns](#timeout-patterns)
6. [Combining Retry and Timeout](#combining-retry-and-timeout)
7. [Retry Strategies](#retry-strategies)
8. [When to Retry](#when-to-retry)
9. [When NOT to Retry](#when-not-to-retry)
10. [Implementation Examples](#implementation-examples)
11. [Best Practices](#best-practices)
12. [Common Mistakes](#common-mistakes)

---

## Why Retry and Timeout?

### The Problem

**Transient Failures:**
```
Request → Service
  ↓
Service temporarily unavailable
  ↓
Request fails
  ↓
But service recovers quickly
```

**Problems:**
- **Network glitches**: Temporary network issues
- **Service overload**: Service temporarily overloaded
- **Database locks**: Temporary database locks
- **Timeout**: Request takes too long

**Without Retry:**
```
Request fails → User sees error
(Even though service would work if retried)
```

**With Retry:**
```
Request fails → Retry → Success
User sees success
```

### The Need for Timeout

**Problem:**
```
Request → Service
  ↓
Service hangs (doesn't respond)
  ↓
Request waits forever
  ↓
User waits forever
  ↓
Resources blocked
```

**With Timeout:**
```
Request → Service (with timeout)
  ↓
Service doesn't respond in time
  ↓
Timeout → Fail fast
  ↓
User gets error quickly
  ↓
Resources freed
```

---

## Retry Patterns

### What is Retry?

**Retry**: Attempting an operation again after it fails, with the hope that it will succeed.

**Key Concept:**
- **Transient failures**: Failures that are temporary
- **Retry logic**: Logic to retry failed operations
- **Success on retry**: Operation might succeed on retry

### Simple Retry

**Basic Pattern:**
```python
def simple_retry(func, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise  # Last attempt failed
            # Retry
            continue
```

**Example:**
```python
def call_api():
    response = requests.get("https://api.example.com/data")
    response.raise_for_status()
    return response.json()

# Retry up to 3 times
result = simple_retry(call_api, max_attempts=3)
```

### Retry with Delay

**Add Delay Between Retries:**
```python
import time

def retry_with_delay(func, max_attempts=3, delay=1):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            time.sleep(delay)  # Wait before retry
            continue
```

**Why Delay?**
- **Service recovery**: Give service time to recover
- **Reduce load**: Reduce load on service
- **Avoid thundering herd**: Avoid all retries at once

---

## Exponential Backoff

### What is Exponential Backoff?

**Exponential Backoff**: Increasing delay between retries exponentially.

**Pattern:**
```
Attempt 1: Wait 1 second
Attempt 2: Wait 2 seconds
Attempt 3: Wait 4 seconds
Attempt 4: Wait 8 seconds
...
```

**Formula:**
```
delay = base_delay * (2 ^ attempt_number)
```

### Why Exponential Backoff?

**1. Service Recovery:**
- **More time**: Give service more time to recover
- **Gradual**: Gradually increase wait time
- **Efficient**: Don't retry too quickly

**2. Reduce Load:**
- **Less load**: Reduce load on failing service
- **Spread out**: Spread retries over time
- **Avoid overload**: Avoid overwhelming service

**3. Network Issues:**
- **Network recovery**: Network might need time
- **Congestion**: Network congestion might clear
- **Timeout**: Give time for timeouts to clear

### Implementation

**Basic Exponential Backoff:**
```python
import time
import random

def exponential_backoff(func, max_attempts=5, base_delay=1):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            
            # Calculate delay: base_delay * 2^attempt
            delay = base_delay * (2 ** attempt)
            time.sleep(delay)
            continue
```

**Example:**
```python
def call_api():
    response = requests.get("https://api.example.com/data")
    response.raise_for_status()
    return response.json()

# Retry with exponential backoff
result = exponential_backoff(call_api, max_attempts=5, base_delay=1)
# Delays: 1s, 2s, 4s, 8s
```

---

## Jitter in Retries

### What is Jitter?

**Jitter**: Random variation added to retry delay to prevent synchronized retries.

### The Thundering Herd Problem

**Problem:**
```
1000 clients all retry at same time
  ↓
All retry at: 0s, 1s, 2s, 4s, 8s
  ↓
Synchronized retries
  ↓
Service overwhelmed
  ↓
All fail again
  ↓
Cycle repeats
```

**Visual:**
```
Time: 0s    1s    2s    4s    8s
      ↓     ↓     ↓     ↓     ↓
      [All retry] [All retry] [All retry] [All retry] [All retry]
      (Synchronized - bad!)
```

### Solution: Jitter

**Add Random Variation:**
```
Base delay: 1s
Jitter: ±0.5s
Actual delays: 0.5s, 1.5s, 0.8s, 1.2s, ...
(Spread out - good!)
```

**Types of Jitter:**

**1. Full Jitter:**
```python
delay = random.uniform(0, base_delay * (2 ** attempt))
```

**2. Equal Jitter:**
```python
delay = base_delay * (2 ** attempt) / 2 + random.uniform(0, base_delay * (2 ** attempt) / 2)
```

**3. Decorrelated Jitter:**
```python
# More sophisticated algorithm
# Spreads retries more evenly
```

### Implementation with Jitter

```python
import time
import random

def exponential_backoff_with_jitter(func, max_attempts=5, base_delay=1):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            
            # Exponential backoff
            base = base_delay * (2 ** attempt)
            # Add jitter: random between 0 and base
            delay = random.uniform(0, base)
            time.sleep(delay)
            continue
```

**Benefits:**
- **Spread retries**: Retries spread over time
- **Reduce load**: Reduce load spikes
- **Better success rate**: Better chance of success

---

## Timeout Patterns

### What is Timeout?

**Timeout**: Maximum time to wait for an operation to complete before giving up.

**Purpose:**
- **Fail fast**: Don't wait forever
- **Free resources**: Free resources quickly
- **Better UX**: User gets response quickly

### Types of Timeouts

**1. Connection Timeout:**
```
Time to establish connection
Example: 5 seconds
```

**2. Read Timeout:**
```
Time to read response
Example: 30 seconds
```

**3. Total Timeout:**
```
Total time for entire operation
Example: 60 seconds
```

### Implementation

**Basic Timeout:**
```python
import signal

class TimeoutError(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutError("Operation timed out")

def with_timeout(func, timeout_seconds=30):
    # Set timeout
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(timeout_seconds)
    
    try:
        result = func()
        signal.alarm(0)  # Cancel timeout
        return result
    except TimeoutError:
        raise
    finally:
        signal.alarm(0)  # Always cancel
```

**Using Requests:**
```python
import requests

# Connection timeout: 5 seconds
# Read timeout: 30 seconds
response = requests.get(
    "https://api.example.com/data",
    timeout=(5, 30)  # (connect, read)
)
```

### Timeout Best Practices

**1. Set Appropriate Timeouts:**
```
Fast operations: 1-5 seconds
Medium operations: 10-30 seconds
Slow operations: 60+ seconds
```

**2. Different Timeouts for Different Operations:**
```python
# Fast: Database query
db_timeout = 5

# Medium: API call
api_timeout = 30

# Slow: File processing
file_timeout = 300
```

**3. Timeout Hierarchy:**
```
Total timeout: 60s
  ├── Connection: 5s
  ├── Request: 30s
  └── Processing: 25s
```

---

## Combining Retry and Timeout

### Why Combine?

**Retry Alone:**
```
Request → Timeout (no timeout set)
  ↓
Waits forever
  ↓
Never retries
```

**Timeout Alone:**
```
Request → Timeout → Fail
  ↓
No retry
  ↓
Might have succeeded on retry
```

**Retry + Timeout:**
```
Request → Timeout → Retry → Success
(Best of both)
```

### Implementation

**Retry with Timeout:**
```python
import time
import random
import requests

def retry_with_timeout(url, max_attempts=3, timeout=30, base_delay=1):
    for attempt in range(max_attempts):
        try:
            response = requests.get(url, timeout=timeout)
            response.raise_for_status()
            return response.json()
        except (requests.Timeout, requests.ConnectionError) as e:
            if attempt == max_attempts - 1:
                raise
            
            # Exponential backoff with jitter
            base = base_delay * (2 ** attempt)
            delay = random.uniform(0, base)
            time.sleep(delay)
            continue
        except requests.HTTPError as e:
            # Don't retry on 4xx errors (client errors)
            raise
```

---

## Retry Strategies

### Strategy 1: Fixed Retry

**Fixed Number of Retries:**
```python
max_attempts = 3
# Always retry 3 times
```

**Use When:**
- **Simple cases**: Simple retry needs
- **Predictable**: Predictable failure patterns
- **Low risk**: Low risk of overload

### Strategy 2: Exponential Backoff

**Exponential Delay:**
```python
delay = base_delay * (2 ** attempt)
# Delays: 1s, 2s, 4s, 8s, 16s
```

**Use When:**
- **Service recovery**: Service needs time to recover
- **High load**: High load situations
- **Network issues**: Network congestion

### Strategy 3: Linear Backoff

**Linear Delay:**
```python
delay = base_delay * attempt
# Delays: 1s, 2s, 3s, 4s, 5s
```

**Use When:**
- **Moderate recovery**: Moderate recovery time needed
- **Predictable**: Predictable failure duration
- **Simple**: Simpler than exponential

### Strategy 4: Immediate Retry

**No Delay:**
```python
# Retry immediately
# No delay between retries
```

**Use When:**
- **Very transient**: Very transient failures
- **Quick recovery**: Quick recovery expected
- **Low risk**: Low risk of overload

---

## When to Retry

### Good Cases for Retry

**1. Network Errors:**
```
Connection timeout
Connection refused
Network unreachable
DNS resolution failure
```

**2. Transient Service Errors:**
```
503 Service Unavailable
502 Bad Gateway
504 Gateway Timeout
429 Too Many Requests (with backoff)
```

**3. Database Errors:**
```
Connection pool exhausted
Deadlock
Temporary lock
```

**4. Rate Limiting:**
```
429 Too Many Requests
(Retry after rate limit window)
```

### Retry Decision Matrix

| Error Type | Retry? | Reason |
|------------|--------|--------|
| **Network timeout** | Yes | Transient |
| **503 Service Unavailable** | Yes | Temporary |
| **500 Internal Server Error** | Maybe | Might be transient |
| **400 Bad Request** | No | Client error |
| **401 Unauthorized** | No | Auth issue |
| **404 Not Found** | No | Resource doesn't exist |
| **429 Too Many Requests** | Yes | With backoff |

---

## When NOT to Retry

### Bad Cases for Retry

**1. Client Errors (4xx):**
```
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```
**Reason**: Client error, won't succeed on retry

**2. Idempotency Violations:**
```
Operation not idempotent
Retry might cause duplicate operations
```

**3. Expensive Operations:**
```
Expensive operations
Retry multiplies cost
```

**4. User-Initiated Actions:**
```
User actions
User expects immediate feedback
Don't retry silently
```

### Examples

**Don't Retry:**
```python
# 400 Bad Request - client error
if response.status_code == 400:
    raise ValueError("Invalid request")
    # Don't retry - won't help

# 401 Unauthorized - auth issue
if response.status_code == 401:
    raise AuthenticationError("Not authorized")
    # Don't retry - need to fix auth
```

**Do Retry:**
```python
# 503 Service Unavailable - transient
if response.status_code == 503:
    # Retry - service might recover
    raise RetryableError("Service unavailable")

# Network timeout - transient
except requests.Timeout:
    # Retry - network might recover
    raise RetryableError("Timeout")
```

---

## Implementation Examples

### Python: Tenacity Library

**Simple Retry:**
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10)
)
def call_api():
    response = requests.get("https://api.example.com/data")
    response.raise_for_status()
    return response.json()
```

**Retry on Specific Exceptions:**
```python
from tenacity import retry, retry_if_exception_type, stop_after_attempt

@retry(
    retry=retry_if_exception_type(requests.Timeout),
    stop=stop_after_attempt(3)
)
def call_api():
    response = requests.get("https://api.example.com/data", timeout=30)
    response.raise_for_status()
    return response.json()
```

### Java: Resilience4j

**Retry Configuration:**
```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(3)
    .waitDuration(Duration.ofSeconds(1))
    .retryOnException(e -> e instanceof TimeoutException)
    .build();

Retry retry = Retry.of("apiCall", config);

String result = retry.executeSupplier(() -> {
    return callApi();
});
```

### Go: Custom Implementation

**Retry with Exponential Backoff:**
```go
func retryWithBackoff(fn func() error, maxAttempts int) error {
    baseDelay := time.Second
    
    for attempt := 0; attempt < maxAttempts; attempt++ {
        err := fn()
        if err == nil {
            return nil
        }
        
        if attempt == maxAttempts-1 {
            return err
        }
        
        delay := baseDelay * time.Duration(1<<uint(attempt))
        time.Sleep(delay)
    }
    
    return fmt.Errorf("max attempts reached")
}
```

---

## Best Practices

### 1. Use Exponential Backoff

**Why:**
- **Service recovery**: Give service time to recover
- **Reduce load**: Reduce load on service
- **Better success rate**: Better chance of success

**Implementation:**
```python
delay = base_delay * (2 ** attempt)
```

### 2. Add Jitter

**Why:**
- **Prevent thundering herd**: Prevent synchronized retries
- **Spread load**: Spread load over time
- **Better distribution**: Better retry distribution

**Implementation:**
```python
delay = random.uniform(0, base_delay * (2 ** attempt))
```

### 3. Set Appropriate Timeouts

**Why:**
- **Fail fast**: Don't wait forever
- **Free resources**: Free resources quickly
- **Better UX**: User gets response quickly

**Guidelines:**
- **Fast operations**: 1-5 seconds
- **Medium operations**: 10-30 seconds
- **Slow operations**: 60+ seconds

### 4. Retry Only Transient Failures

**Retry:**
- Network errors
- 5xx server errors
- Timeouts
- Rate limits (with backoff)

**Don't Retry:**
- 4xx client errors
- Authentication errors
- Validation errors
- Non-idempotent operations

### 5. Limit Retry Attempts

**Why:**
- **Prevent infinite loops**: Prevent infinite retries
- **Fail eventually**: Fail eventually if service down
- **Resource usage**: Limit resource usage

**Typical:**
- **3-5 attempts**: For most cases
- **More attempts**: For critical operations
- **Less attempts**: For fast-fail scenarios

### 6. Log Retries

**Why:**
- **Debugging**: Help debug issues
- **Monitoring**: Monitor retry patterns
- **Alerting**: Alert on high retry rates

**Implementation:**
```python
logger.warning(f"Retry attempt {attempt + 1}/{max_attempts} for {operation}")
```

### 7. Use Circuit Breaker

**Why:**
- **Prevent cascading failures**: Stop retrying if service down
- **Fail fast**: Fail fast if service unavailable
- **Resource protection**: Protect resources

**Combine:**
```
Retry → Circuit Breaker
  ↓
If circuit open: Don't retry
If circuit closed: Retry normally
```

---

## Common Mistakes

### Mistake 1: Retrying Everything

**Bad:**
```python
# Retry on all errors
@retry(stop=stop_after_attempt(5))
def call_api():
    response = requests.get(url)
    return response.json()
    # Retries even on 400 Bad Request!
```

**Good:**
```python
# Retry only on retryable errors
@retry(
    retry=retry_if_exception_type((requests.Timeout, requests.ConnectionError)),
    stop=stop_after_attempt(3)
)
def call_api():
    response = requests.get(url)
    response.raise_for_status()
    return response.json()
```

### Mistake 2: No Timeout

**Bad:**
```python
# No timeout - waits forever
response = requests.get(url)
```

**Good:**
```python
# Set timeout
response = requests.get(url, timeout=30)
```

### Mistake 3: Too Many Retries

**Bad:**
```python
# 100 retries - too many!
@retry(stop=stop_after_attempt(100))
def call_api():
    ...
```

**Good:**
```python
# 3-5 retries - reasonable
@retry(stop=stop_after_attempt(3))
def call_api():
    ...
```

### Mistake 4: No Backoff

**Bad:**
```python
# Retry immediately - might overwhelm service
for attempt in range(3):
    try:
        return call_api()
    except:
        continue  # No delay!
```

**Good:**
```python
# Exponential backoff
for attempt in range(3):
    try:
        return call_api()
    except:
        delay = 1 * (2 ** attempt)
        time.sleep(delay)
```

### Mistake 5: Retrying Non-Idempotent Operations

**Bad:**
```python
# Retry payment - might charge twice!
@retry(stop=stop_after_attempt(3))
def process_payment(amount):
    charge_credit_card(amount)  # Not idempotent!
```

**Good:**
```python
# Make idempotent first
def process_payment(amount, idempotency_key):
    if already_processed(idempotency_key):
        return get_existing_payment(idempotency_key)
    return charge_credit_card(amount, idempotency_key)
```

---

## Summary

Retry and timeout patterns are essential for building resilient systems that handle transient failures gracefully.

**Key Takeaways:**
- **Retry**: Attempt operation again after failure
- **Exponential backoff**: Increase delay between retries
- **Jitter**: Add randomness to prevent thundering herd
- **Timeout**: Maximum time to wait for operation
- **Combine**: Use retry and timeout together
- **When to retry**: Transient failures only
- **When not to retry**: Client errors, non-idempotent operations
- **Best practices**: Exponential backoff, jitter, appropriate timeouts, limit attempts

**Retry Strategies:**
- **Fixed retry**: Fixed number of attempts
- **Exponential backoff**: Exponential delay increase
- **Linear backoff**: Linear delay increase
- **Immediate retry**: No delay

**Timeout Types:**
- **Connection timeout**: Time to establish connection
- **Read timeout**: Time to read response
- **Total timeout**: Total operation time

**Best Practices:**
- Use exponential backoff
- Add jitter
- Set appropriate timeouts
- Retry only transient failures
- Limit retry attempts
- Log retries
- Use circuit breaker

**Common Mistakes:**
- Retrying everything
- No timeout
- Too many retries
- No backoff
- Retrying non-idempotent operations

**Next Steps:**
- Implement retry logic
- Add timeouts
- Use exponential backoff
- Add jitter
- Monitor retry patterns
- Apply best practices

