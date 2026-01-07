# Caching Deep Dive - Complete Understanding

## Table of Contents
1. [What is Caching and Why Do We Need It?](#what-is-caching-and-why-do-we-need-it)
2. [Cache Hierarchy - Multiple Levels](#cache-hierarchy---multiple-levels)
3. [Cache Replacement Policies](#cache-replacement-policies)
4. [LRU Cache - Least Recently Used](#lru-cache---least-recently-used)
5. [LFU Cache - Least Frequently Used](#lfu-cache---least-frequently-used)
6. [In-Memory Caching - Redis, Memcached](#in-memory-caching---redis-memcached)
7. [Cache Stampede Problem](#cache-stampede-problem)
8. [Cache Invalidation Strategies](#cache-invalidation-strategies)
9. [Distributed Caching](#distributed-caching)
10. [Cache Coherence and Consistency](#cache-coherence-and-consistency)

---

## What is Caching and Why Do We Need It?

### The Problem

**Slow Data Access:**
```
Database query: 100ms
Disk read: 10ms
Network request: 50ms
```

**Problem:**
- **Repeated access**: Same data accessed many times
- **Waste**: Re-fetching same data
- **Slow**: Users wait

### The Solution: Caching

**Cache**: Fast storage for frequently accessed data.

**Benefits:**
- **Speed**: Much faster than original source
- **Reduced load**: Less pressure on source
- **Better performance**: Faster responses

### Real-World Analogy

**Cache = Frequently Used Items on Desk:**
- **Desk (cache)**: Fast access, limited space
- **Filing cabinet (database)**: Slow access, unlimited space
- **Strategy**: Keep frequently used items on desk
- **Result**: Faster access to common items

---

## Cache Hierarchy - Multiple Levels

### CPU Cache Hierarchy

**Levels:**
```
L1 Cache: Fastest, smallest (KB)
L2 Cache: Medium, medium (MB)
L3 Cache: Slower, larger (MB)
RAM: Slowest, largest (GB)
```

**Access Times:**
```
L1: 1 cycle (nanoseconds)
L2: 10 cycles
L3: 40 cycles
RAM: 100+ cycles
```

### Application Cache Hierarchy

**Levels:**
```
CPU Cache: Hardware cache
Application Cache: In-memory (Redis, Memcached)
Database Cache: Query cache
CDN Cache: Edge cache
Browser Cache: Client cache
```

---

## Cache Replacement Policies

### The Problem

**Cache Full:**
```
Cache has limited size
New item needs to be cached
Which item to evict?
```

### Policies

**1. FIFO (First In First Out):**
```
Evict oldest item
Simple but not optimal
```

**2. LRU (Least Recently Used):**
```
Evict item not used for longest time
Good performance
```

**3. LFU (Least Frequently Used):**
```
Evict item used least often
Good for stable access patterns
```

**4. Random:**
```
Evict random item
Simple but unpredictable
```

**5. Optimal:**
```
Evict item that won't be used for longest time
Optimal but requires future knowledge
```

---

## LRU Cache - Least Recently Used

### How LRU Works

**Principle:** Evict item not used for longest time.

**Implementation:**
```
Data structures:
  - HashMap: O(1) lookup
  - Doubly Linked List: O(1) insertion/deletion, maintains order

Operations:
  - Get: Move to front (most recently used)
  - Put: Add to front, evict from back if full
```

### LRU Implementation

**Python Example:**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key in self.cache:
            # Move to end (most recently used)
            self.cache.move_to_end(key)
            return self.cache[key]
        return -1
    
    def put(self, key, value):
        if key in self.cache:
            # Update and move to end
            self.cache[key] = value
            self.cache.move_to_end(key)
        else:
            if len(self.cache) >= self.capacity:
                # Remove least recently used (first item)
                self.cache.popitem(last=False)
            self.cache[key] = value
```

### Visual Representation

```
Cache (size 3):
[Most Recent] → [Item 2] → [Item 1] → [Least Recent]
     ↑                              ↑
   Head                           Tail

After accessing Item 1:
[Most Recent] → [Item 1] → [Item 2] → [Item 3]
     ↑                              ↑
   Head                           Tail
```

### Thread-Safe LRU

**Problem:**
```
Multiple threads access cache
Race conditions
Data corruption
```

**Solution:**
```python
import threading

class ThreadSafeLRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
        self.lock = threading.Lock()
    
    def get(self, key):
        with self.lock:
            # ... same as before
```

---

## LFU Cache - Least Frequently Used

### How LFU Works

**Principle:** Evict item used least often.

**Implementation:**
```
Track access count for each item
Evict item with lowest count
If tie: Use LRU among tied items
```

### LFU vs LRU

**LRU:**
- **Recent access**: Matters more
- **Use case**: Temporal locality

**LFU:**
- **Frequency**: Matters more
- **Use case**: Stable access patterns

---

## In-Memory Caching - Redis, Memcached

### Redis

**What is Redis?**
- **In-memory data store**: Very fast
- **Data structures**: Strings, lists, sets, hashes, sorted sets
- **Persistence**: Can persist to disk
- **Advanced features**: Pub/sub, transactions, Lua scripting

**Use Cases:**
- **Caching**: Fast cache
- **Session storage**: User sessions
- **Real-time**: Leaderboards, counters
- **Message queue**: Pub/sub

**Example:**
```python
import redis

r = redis.Redis()
r.set('user:123', '{"name": "Alice"}')
user = r.get('user:123')
```

### Memcached

**What is Memcached?**
- **Simple key-value store**: Very fast
- **Distributed**: Multiple servers
- **No persistence**: Memory only
- **Simple**: Easy to use

**Use Cases:**
- **Simple caching**: Key-value cache
- **Session storage**: Simple sessions
- **Database caching**: Cache query results

**Example:**
```python
import memcache

mc = memcache.Client(['127.0.0.1:11211'])
mc.set('user:123', '{"name": "Alice"}')
user = mc.get('user:123')
```

### Redis vs Memcached

| Aspect | Redis | Memcached |
|--------|-------|-----------|
| **Data Types** | Many (strings, lists, etc.) | Key-value only |
| **Persistence** | Yes | No |
| **Complexity** | More features | Simpler |
| **Use Case** | Advanced caching | Simple caching |

---

## Cache Stampede Problem

### What is Cache Stampede?

**Problem:**
```
Cache expires
Many requests for same key
All miss cache
All query database simultaneously
Database overloaded
```

### Example

**Scenario:**
```
Cache key: "popular_product" expires
1000 requests arrive simultaneously
All see cache miss
All query database
Database overwhelmed
```

### Solutions

**1. Lock Per Key:**
```python
lock = Lock(key)
if not cache.has(key):
    with lock:
        if not cache.has(key):  # Double check
            value = db.query()
            cache.set(key, value)
return cache.get(key)
```

**2. Probabilistic Early Expiration:**
```
Refresh cache before expiration
Randomly refresh some items early
Reduces simultaneous misses
```

**3. Background Refresh:**
```
Refresh in background before expiration
Always have fresh data
No cache misses
```

---

## Cache Invalidation Strategies

### TTL (Time To Live)

**How it works:**
```
Cache item expires after TTL
Automatic invalidation
Simple
```

**Example:**
```
cache.set('user:123', data, ttl=3600)  # 1 hour
After 1 hour: Automatically expired
```

### Manual Invalidation

**Explicit deletion:**
```
cache.delete('user:123')
Immediate invalidation
```

### Event-Based Invalidation

**Invalidate on update:**
```
User updated in database
Invalidate cache for that user
cache.delete('user:123')
```

### Tag-Based Invalidation

**Invalidate by tags:**
```
Cache items tagged
Invalidate all items with tag
Useful for related data
```

---

## Distributed Caching

### The Challenge

**Multiple Servers:**
```
Server 1: Has cache
Server 2: Doesn't have cache
Inconsistent
```

### Solutions

**1. Shared Cache:**
```
All servers use same cache (Redis cluster)
Consistent across servers
```

**2. Cache Replication:**
```
Replicate cache to all servers
Consistent but more memory
```

**3. Cache Sharding:**
```
Shard cache across servers
Each server handles subset
```

---

## Cache Coherence and Consistency

### The Problem

**Multiple Caches:**
```
Database: Source of truth
Cache 1: Has old data
Cache 2: Has new data
Inconsistent
```

### Solutions

**1. Write-Through:**
```
Write to cache and database
Always consistent
Slower writes
```

**2. Write-Back:**
```
Write to cache first
Write to database later
Faster but risk of data loss
```

**3. Invalidate on Write:**
```
Write to database
Invalidate cache
Next read fetches fresh data
```

---

## Summary

Caching is essential for performance. Understanding cache policies, implementation, and distributed caching is crucial for backend engineers.

**Key Takeaways:**
- Caching speeds up repeated access
- Multiple cache levels (CPU, application, database, CDN)
- LRU is common replacement policy
- Redis and Memcached for in-memory caching
- Cache stampede must be prevented
- Cache invalidation strategies important
- Distributed caching for scale
- Balance consistency and performance

**Next Steps:**
- Understand your caching needs
- Choose appropriate cache policy
- Implement cache stampede prevention
- Plan cache invalidation
- Monitor cache performance
- Optimize cache hit rates

