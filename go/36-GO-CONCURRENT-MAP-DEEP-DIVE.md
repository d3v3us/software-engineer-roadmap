# Go Concurrent Map Deep Dive - Complete Understanding

## Table of Contents
1. [What is Concurrent Map?](#what-is-concurrent-map)
2. [Why Concurrent Maps are Needed](#why-concurrent-maps-are-needed)
3. [Go Map Concurrency Issues](#go-map-concurrency-issues)
4. [Concurrent Map Solutions](#concurrent-map-solutions)
5. [sync.Map](#syncmap)
6. [Custom Concurrent Map](#custom-concurrent-map)
7. [Best Practices](#best-practices)

---

## What is Concurrent Map?

### Definition

**Concurrent Map**: Map data structure that can be safely accessed by multiple goroutines concurrently.

**Key Characteristics:**
- **Thread-safe**: Safe for concurrent access
- **Multiple readers**: Multiple readers supported
- **Multiple writers**: Multiple writers supported
- **Synchronization**: Proper synchronization

### Real-World Analogy

**Concurrent Map = Shared Safe:**
- **Safe**: Map data structure
- **Multiple users**: Multiple goroutines
- **Lock mechanism**: Synchronization
- **Safe access**: Safe concurrent access

**Programming:**
- **Map**: Key-value storage
- **Goroutines**: Concurrent access
- **Synchronization**: Thread-safe access
- **Performance**: Efficient access

---

## Why Concurrent Maps are Needed?

### Problem with Regular Maps

**Go maps are not thread-safe:**
```go
m := make(map[string]int)

// Concurrent write: RACE CONDITION
go func() {
    m["key"] = 1
}()

go func() {
    m["key"] = 2
}()

// Concurrent read/write: RACE CONDITION
go func() {
    value := m["key"]  // Read
}()

go func() {
    m["key"] = 3  // Write
}()
```

**Result:**
- **Race conditions**: Data races
- **Panic**: May panic with "concurrent map writes"
- **Data corruption**: Data corruption possible

---

## Go Map Concurrency Issues

### Issue 1: Concurrent Writes

**Problem:**
```go
m := make(map[string]int)

// Multiple goroutines writing
for i := 0; i < 10; i++ {
    go func(i int) {
        m["key"] = i  // RACE CONDITION
    }(i)
}
```

**Result:** Panic: "fatal error: concurrent map writes"

### Issue 2: Concurrent Read and Write

**Problem:**
```go
m := make(map[string]int)

// Goroutine 1: Reading
go func() {
    value := m["key"]  // RACE CONDITION
}()

// Goroutine 2: Writing
go func() {
    m["key"] = 42
}()
```

**Result:** Race condition, undefined behavior

---

## Concurrent Map Solutions

### Solution 1: Mutex Protection

**Use mutex to protect map:**
```go
type SafeMap struct {
    mu sync.RWMutex
    m  map[string]int
}

func NewSafeMap() *SafeMap {
    return &SafeMap{
        m: make(map[string]int),
    }
}

func (sm *SafeMap) Get(key string) (int, bool) {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    value, ok := sm.m.m[key]
    return value, ok
}

func (sm *SafeMap) Set(key string, value int) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    sm.m[key] = value
}

func (sm *SafeMap) Delete(key string) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    delete(sm.m, key)
}
```

### Solution 2: sync.Map

**Use sync.Map for concurrent access:**
```go
import "sync"

var m sync.Map

// Store
m.Store("key", 42)

// Load
value, ok := m.Load("key")

// Delete
m.Delete("key")

// Range
m.Range(func(key, value interface{}) bool {
    // Process key-value pair
    return true  // Continue iteration
})
```

---

## sync.Map

### What is sync.Map?

**sync.Map**: Thread-safe map implementation in Go standard library.

**Key Characteristics:**
- **Thread-safe**: Safe for concurrent access
- **Optimized**: Optimized for specific use cases
- **No locking**: No explicit locking needed
- **Type-agnostic**: Uses interface{} for values

### When to Use sync.Map

**Use sync.Map when:**
- **Multiple readers**: Many readers, few writers
- **Key stability**: Keys are stable (not frequently added/removed)
- **Per-key operations**: Operations are per-key

**Don't use sync.Map when:**
- **Many writers**: Many concurrent writers
- **Frequent changes**: Frequent key additions/deletions
- **Type safety**: Need type safety

### sync.Map Example

```go
import "sync"

type Cache struct {
    data sync.Map
}

func (c *Cache) Get(key string) (interface{}, bool) {
    return c.data.Load(key)
}

func (c *Cache) Set(key string, value interface{}) {
    c.data.Store(key, value)
}

func (c *Cache) Delete(key string) {
    c.data.Delete(key)
}

func (c *Cache) Range(fn func(key, value interface{}) bool) {
    c.data.Range(fn)
}
```

---

## Custom Concurrent Map

### Sharded Map Implementation

**Sharded map for better performance:**
```go
type ShardedMap struct {
    shards []*Shard
    shardCount int
}

type Shard struct {
    mu sync.RWMutex
    m  map[string]int
}

func NewShardedMap(shardCount int) *ShardedMap {
    shards := make([]*Shard, shardCount)
    for i := range shards {
        shards[i] = &Shard{
            m: make(map[string]int),
        }
    }
    return &ShardedMap{
        shards: shards,
        shardCount: shardCount,
    }
}

func (sm *ShardedMap) getShard(key string) *Shard {
    hash := fnv32(key)
    return sm.shards[hash%uint32(sm.shardCount)]
}

func (sm *ShardedMap) Get(key string) (int, bool) {
    shard := sm.getShard(key)
    shard.mu.RLock()
    defer shard.mu.RUnlock()
    value, ok := shard.m[key]
    return value, ok
}

func (sm *ShardedMap) Set(key string, value int) {
    shard := sm.getShard(key)
    shard.mu.Lock()
    defer shard.mu.Unlock()
    shard.m[key] = value
}
```

---

## Best Practices

### 1. Use sync.Map for Read-Heavy Workloads

**Why:**
- **Optimized**: Optimized for read-heavy workloads
- **Performance**: Better performance
- **Simplicity**: Simpler than custom implementation

**Guidelines:**
- **Many readers**: Use when many readers
- **Few writers**: Use when few writers
- **Stable keys**: Use when keys are stable

### 2. Use Mutex for Write-Heavy Workloads

**Why:**
- **Flexibility**: More flexible
- **Control**: More control
- **Performance**: Better for write-heavy

**Guidelines:**
- **Many writers**: Use when many writers
- **Type safety**: Use when need type safety
- **Custom logic**: Use when need custom logic

### 3. Consider Sharded Maps for High Concurrency

**Why:**
- **Reduced contention**: Less lock contention
- **Better performance**: Better performance
- **Scalability**: Better scalability

**Guidelines:**
- **High concurrency**: Use for high concurrency
- **Many operations**: Use when many operations
- **Performance critical**: Use when performance critical

### 4. Always Protect Map Access

**Why:**
- **Safety**: Prevent race conditions
- **Correctness**: Correct behavior
- **Reliability**: Reliable programs

**Guidelines:**
- **Never unprotected**: Never access map without protection
- **Consistent**: Use consistent protection
- **Document**: Document concurrent access patterns

---

## Summary

Concurrent maps are essential for safe concurrent access in Go. Understanding map concurrency issues, solutions (mutex, sync.Map), and best practices is crucial for writing safe concurrent code.

**Key Takeaways:**
- **Concurrent map**: Map that can be safely accessed by multiple goroutines (thread-safe, multiple readers/writers, proper synchronization)
- **Go map concurrency issues**: Concurrent writes (panic), concurrent read/write (race condition, undefined behavior)
- **Concurrent map solutions**: Mutex protection (RWMutex for read/write separation), sync.Map (thread-safe map, optimized for read-heavy)
- **sync.Map**: Thread-safe map implementation (when to use: many readers few writers, stable keys; when not to use: many writers, frequent changes)
- **Custom concurrent map**: Sharded map implementation (reduced contention, better performance, scalability)
- **Best practices**: Use sync.Map for read-heavy, use mutex for write-heavy, consider sharded maps for high concurrency, always protect map access

**Concurrent Map Solutions:**
- **Mutex protection**: Simple, flexible, type-safe
- **sync.Map**: Optimized for read-heavy, no explicit locking
- **Sharded map**: Better performance, reduced contention

**Best Practices:**
- Use sync.Map for read-heavy workloads
- Use mutex for write-heavy workloads
- Consider sharded maps for high concurrency
- Always protect map access

**Next Steps:**
- Learn sync.Map
- Practice mutex protection
- Understand sharded maps
- Apply best practices

