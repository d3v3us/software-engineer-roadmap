# Go Sync Primitives Deep Dive - Complete Understanding

## Table of Contents
1. [What are Sync Primitives?](#what-are-sync-primitives)
2. [Why Sync Primitives?](#why-sync-primitives)
3. [sync.Mutex and sync.RWMutex](#syncmutex-and-syncrwmutex)
4. [sync.WaitGroup](#syncwaitgroup)
5. [sync.Once](#synconce)
6. [sync.Cond](#synccond)
7. [sync.Pool](#syncpool)
8. [Best Practices](#best-practices)

---

## What are Sync Primitives?

### Definition

**Sync Primitives**: Synchronization primitives in Go's sync package for coordinating goroutines.

**Key Characteristics:**
- **Synchronization**: Coordinate goroutines
- **Thread-safe**: Thread-safe operations
- **Standard library**: Part of standard library
- **Essential**: Essential for concurrency

### Real-World Analogy

**Sync Primitives = Traffic Control:**
- **Traffic**: Goroutines
- **Signals**: Sync primitives
- **Coordination**: Traffic coordination
- **Safety**: Safe traffic flow

**Programming:**
- **Goroutines**: Concurrent execution
- **Sync primitives**: Coordination tools
- **Synchronization**: Synchronize execution
- **Safety**: Thread-safe operations

---

## Why Sync Primitives?

### Benefits

**1. Coordination:**
```
Multiple goroutines
  ↓
Sync primitives
  ↓
Coordinated execution
```

**2. Thread Safety:**
```
Shared resources
  ↓
Sync primitives
  ↓
Thread-safe access
```

**3. Synchronization:**
```
Asynchronous operations
  ↓
Sync primitives
  ↓
Synchronized execution
```

---

## sync.Mutex and sync.RWMutex

### sync.Mutex

**Standard mutex:**
```go
import "sync"

var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}
```

**Characteristics:**
- **Exclusive lock**: Exclusive access
- **Blocking**: Blocks until available
- **Reentrant**: Not reentrant

### sync.RWMutex

**Read-write mutex:**
```go
import "sync"

var mu sync.RWMutex
var data map[string]int

func read(key string) int {
    mu.RLock()
    defer mu.RUnlock()
    return data[key]
}

func write(key string, value int) {
    mu.Lock()
    defer mu.Unlock()
    data[key] = value
}
```

**Characteristics:**
- **Multiple readers**: Multiple readers allowed
- **Single writer**: Single writer at a time
- **Performance**: Better for read-heavy

---

## sync.WaitGroup

### What is WaitGroup?

**WaitGroup**: Waits for collection of goroutines to finish.

**Use Cases:**
- **Goroutine coordination**: Wait for goroutines
- **Synchronization**: Synchronize completion
- **Batch processing**: Wait for batch completion

### WaitGroup Usage

```go
import "sync"

var wg sync.WaitGroup

func main() {
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            // Work
        }(i)
    }
    wg.Wait()  // Wait for all goroutines
}
```

**Methods:**
- **Add(delta)**: Add to counter
- **Done()**: Decrement counter
- **Wait()**: Wait until counter is zero

---

## sync.Once

### What is sync.Once?

**sync.Once**: Ensures function executes only once.

**Use Cases:**
- **Initialization**: One-time initialization
- **Singleton**: Singleton pattern
- **Lazy initialization**: Lazy initialization

### sync.Once Usage

```go
import "sync"

var once sync.Once
var instance *MyStruct

func getInstance() *MyStruct {
    once.Do(func() {
        instance = &MyStruct{}
    })
    return instance
}
```

**Characteristics:**
- **Once**: Executes only once
- **Thread-safe**: Thread-safe
- **Idempotent**: Idempotent operation

---

## sync.Cond

### What is sync.Cond?

**sync.Cond**: Condition variable for goroutine signaling.

**Use Cases:**
- **Signaling**: Signal between goroutines
- **Broadcasting**: Broadcast to multiple goroutines
- **Coordination**: Coordinate goroutines

### sync.Cond Usage

```go
import "sync"

var mu sync.Mutex
var cond = sync.NewCond(&mu)
var ready bool

// Waiter
go func() {
    mu.Lock()
    for !ready {
        cond.Wait()
    }
    mu.Unlock()
}()

// Signaler
go func() {
    mu.Lock()
    ready = true
    cond.Signal()  // or cond.Broadcast()
    mu.Unlock()
}()
```

---

## sync.Pool

### What is sync.Pool?

**sync.Pool**: Pool of temporary objects for reuse.

**Use Cases:**
- **Object reuse**: Reuse temporary objects
- **GC pressure**: Reduce GC pressure
- **Performance**: Better performance

### sync.Pool Usage

```go
import "sync"

var pool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func process() {
    buf := pool.Get().([]byte)
    defer pool.Put(buf)
    // Use buf
}
```

**Characteristics:**
- **Object pool**: Pool of objects
- **Reuse**: Reuse objects
- **GC friendly**: Reduces GC pressure

---

## Best Practices

### 1. Use defer for Unlock

**Why:**
- **Safety**: Ensures unlock
- **Error handling**: Unlocks even on errors
- **Correctness**: Correct behavior

**Guidelines:**
- **Always defer**: Always use defer for unlock
- **Error safety**: Unlocks on errors
- **Consistency**: Consistent pattern

### 2. Use RWMutex for Read-Heavy

**Why:**
- **Performance**: Better performance
- **Concurrency**: More concurrent readers
- **Efficiency**: More efficient

**Guidelines:**
- **Many readers**: Use when many readers
- **Few writers**: Use when few writers
- **Read-heavy**: Use for read-heavy workloads

### 3. Use WaitGroup for Coordination

**Why:**
- **Coordination**: Coordinate goroutines
- **Synchronization**: Wait for completion
- **Clarity**: Clear coordination

**Guidelines:**
- **Add before**: Add before goroutine
- **Done in goroutine**: Call Done in goroutine
- **Wait after**: Wait after starting goroutines

### 4. Use sync.Once for Initialization

**Why:**
- **Once**: Ensures once execution
- **Thread-safe**: Thread-safe initialization
- **Efficiency**: Efficient initialization

**Guidelines:**
- **One-time init**: Use for one-time initialization
- **Singleton**: Use for singleton pattern
- **Lazy init**: Use for lazy initialization

---

## Summary

Sync primitives are essential for goroutine coordination in Go. Understanding mutex, WaitGroup, Once, Cond, Pool, and best practices is crucial for effective concurrent programming.

**Key Takeaways:**
- **Sync primitives**: Synchronization primitives in sync package (synchronization, thread-safe, standard library, essential)
- **sync.Mutex and sync.RWMutex**: Mutex (exclusive lock, blocking) vs RWMutex (multiple readers, single writer, read-heavy)
- **sync.WaitGroup**: Waits for goroutines to finish (Add, Done, Wait methods, goroutine coordination)
- **sync.Once**: Ensures function executes only once (one-time initialization, singleton, lazy initialization)
- **sync.Cond**: Condition variable for signaling (signaling, broadcasting, coordination)
- **sync.Pool**: Pool of temporary objects (object reuse, GC pressure reduction, performance)
- **Best practices**: Use defer for unlock, use RWMutex for read-heavy, use WaitGroup for coordination, use sync.Once for initialization

**Sync Primitives:**
- **Mutex/RWMutex**: Lock-based synchronization
- **WaitGroup**: Goroutine coordination
- **Once**: One-time execution
- **Cond**: Condition signaling
- **Pool**: Object pooling

**Best Practices:**
- Use defer for unlock
- Use RWMutex for read-heavy
- Use WaitGroup for coordination
- Use sync.Once for initialization

**Next Steps:**
- Learn sync primitives
- Practice coordination patterns
- Master synchronization
- Apply best practices

