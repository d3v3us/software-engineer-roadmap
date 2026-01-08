# Go Anti-patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What are Anti-patterns?](#what-are-anti-patterns)
2. [Why Anti-patterns Matter](#why-anti-patterns-matter)
3. [Concurrency Anti-patterns](#concurrency-anti-patterns)
4. [Error Handling Anti-patterns](#error-handling-anti-patterns)
5. [Memory Anti-patterns](#memory-anti-patterns)
6. [Interface Anti-patterns](#interface-anti-patterns)
7. [Channel Anti-patterns](#channel-anti-patterns)
8. [Best Practices](#best-practices)

---

## What are Anti-patterns?

### Definition

**Anti-patterns**: Common patterns that seem correct but lead to problems.

**Key Characteristics:**
- **Common mistakes**: Frequently made mistakes
- **Seem correct**: Look correct at first
- **Cause problems**: Lead to bugs or performance issues
- **Avoidable**: Can be avoided with knowledge

### Real-World Analogy

**Anti-patterns = Bad Habits:**
- **Habits**: Programming patterns
- **Bad habits**: Anti-patterns
- **Problems**: Cause issues
- **Learning**: Learn to avoid

**Programming:**
- **Patterns**: Code patterns
- **Anti-patterns**: Bad patterns
- **Issues**: Bugs, performance
- **Best practices**: Good patterns

---

## Why Anti-patterns Matter?

### Benefits

**1. Avoid Bugs:**
```
Anti-patterns
  ↓
Bugs
  ↓
Avoid anti-patterns
```

**2. Better Performance:**
```
Anti-patterns
  ↓
Performance issues
  ↓
Avoid anti-patterns
```

**3. Maintainability:**
```
Anti-patterns
  ↓
Hard to maintain
  ↓
Avoid anti-patterns
```

---

## Concurrency Anti-patterns

### 1. Goroutine Leak

**Anti-pattern:**
```go
func leakExample() {
    ch := make(chan int)
    go func() {
        // Goroutine blocks forever
        <-ch
    }()
    // Channel never receives, goroutine leaks
}
```

**Problem:**
- Goroutine never exits
- Memory leak
- Resource waste

**Solution:**
```go
func noLeak() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    ch := make(chan int)
    go func() {
        select {
        case <-ch:
            // Process
        case <-ctx.Done():
            return
        }
    }()
}
```

### 2. Shared Mutable State

**Anti-pattern:**
```go
var counter int

func increment() {
    counter++ // Race condition!
}
```

**Problem:**
- Race conditions
- Data corruption
- Unpredictable behavior

**Solution:**
```go
var counter int64
var mu sync.Mutex

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}

// Or use atomic
func incrementAtomic() {
    atomic.AddInt64(&counter, 1)
}
```

### 3. Unbuffered Channel Deadlock

**Anti-pattern:**
```go
func deadlock() {
    ch := make(chan int)
    ch <- 1 // Blocks forever if no receiver
    fmt.Println(<-ch)
}
```

**Problem:**
- Deadlock
- Program hangs

**Solution:**
```go
func noDeadlock() {
    ch := make(chan int, 1) // Buffered
    ch <- 1
    fmt.Println(<-ch)
    
    // Or use goroutine
    go func() {
        ch <- 1
    }()
    fmt.Println(<-ch)
}
```

---

## Error Handling Anti-patterns

### 1. Ignoring Errors

**Anti-pattern:**
```go
func ignoreError() {
    file, _ := os.Open("file.txt") // Error ignored!
    defer file.Close()
}
```

**Problem:**
- Errors ignored
- Silent failures
- Hard to debug

**Solution:**
```go
func handleError() {
    file, err := os.Open("file.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()
}
```

### 2. Wrapping Without Context

**Anti-pattern:**
```go
func wrapError() error {
    if err := doSomething(); err != nil {
        return err // No context
    }
    return nil
}
```

**Problem:**
- No context
- Hard to debug
- Lost information

**Solution:**
```go
func wrapWithContext() error {
    if err := doSomething(); err != nil {
        return fmt.Errorf("failed to do something: %w", err)
    }
    return nil
}
```

### 3. Panic for Control Flow

**Anti-pattern:**
```go
func panicFlow() {
    if condition {
        panic("error") // Don't use panic for control flow
    }
}
```

**Problem:**
- Panic for non-exceptional cases
- Hard to recover
- Unexpected behavior

**Solution:**
```go
func returnError() error {
    if condition {
        return errors.New("error")
    }
    return nil
}
```

---

## Memory Anti-patterns

### 1. Large Slice Reallocation

**Anti-pattern:**
```go
func reallocation() {
    var s []int
    for i := 0; i < 1000000; i++ {
        s = append(s, i) // Many reallocations
    }
}
```

**Problem:**
- Many reallocations
- Performance issues
- Memory waste

**Solution:**
```go
func preallocate() {
    s := make([]int, 0, 1000000) // Preallocate capacity
    for i := 0; i < 1000000; i++ {
        s = append(s, i)
    }
}
```

### 2. Memory Leak with Maps

**Anti-pattern:**
```go
func leakMap() {
    m := make(map[string]*LargeStruct)
    for i := 0; i < 1000; i++ {
        m[fmt.Sprintf("key%d", i)] = &LargeStruct{}
    }
    // Map never cleared, memory leak
}
```

**Problem:**
- Memory leak
- Growing memory usage

**Solution:**
```go
func noLeakMap() {
    m := make(map[string]*LargeStruct)
    for i := 0; i < 1000; i++ {
        m[fmt.Sprintf("key%d", i)] = &LargeStruct{}
    }
    // Clear when done
    for k := range m {
        delete(m, k)
    }
}
```

### 3. Unnecessary Pointer Usage

**Anti-pattern:**
```go
func unnecessaryPointer(s *string) { // String is small, no need for pointer
    fmt.Println(*s)
}
```

**Problem:**
- Unnecessary indirection
- Heap allocation
- Performance overhead

**Solution:**
```go
func useValue(s string) { // Pass by value for small types
    fmt.Println(s)
}
```

---

## Interface Anti-patterns

### 1. Overly Broad Interfaces

**Anti-pattern:**
```go
type Everything interface {
    DoA()
    DoB()
    DoC()
    DoD()
    // Too many methods
}
```

**Problem:**
- Hard to implement
- Tight coupling
- Less flexible

**Solution:**
```go
type Reader interface {
    Read([]byte) (int, error)
}

type Writer interface {
    Write([]byte) (int, error)
}

// Small, focused interfaces
```

### 2. Interface in Wrong Package

**Anti-pattern:**
```go
// In consumer package
type Database interface {
    Save(User) error
}
```

**Problem:**
- Interface in wrong place
- Dependency inversion violation

**Solution:**
```go
// In provider package
type Database interface {
    Save(User) error
}

// Consumer uses interface
```

### 3. Empty Interface Abuse

**Anti-pattern:**
```go
func process(data interface{}) { // Too generic
    // Type assertions everywhere
}
```

**Problem:**
- Loss of type safety
- Runtime errors
- Hard to maintain

**Solution:**
```go
func process[T any](data T) { // Use generics
    // Type-safe
}
```

---

## Channel Anti-patterns

### 1. Closing Channel Multiple Times

**Anti-pattern:**
```go
func closeMultiple() {
    ch := make(chan int)
    close(ch)
    close(ch) // Panic!
}
```

**Problem:**
- Panic
- Race conditions

**Solution:**
```go
func closeOnce() {
    ch := make(chan int)
    once := sync.Once{}
    once.Do(func() {
        close(ch)
    })
}
```

### 2. Sending to Closed Channel

**Anti-pattern:**
```go
func sendClosed() {
    ch := make(chan int)
    close(ch)
    ch <- 1 // Panic!
}
```

**Problem:**
- Panic
- Unexpected behavior

**Solution:**
```go
func checkClosed() {
    ch := make(chan int)
    close(ch)
    select {
    case ch <- 1:
        // Sent
    default:
        // Channel closed or full
    }
}
```

### 3. Range Over Channel Without Close

**Anti-pattern:**
```go
func rangeNoClose() {
    ch := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            ch <- i
        }
        // Forgot to close!
    }()
    for v := range ch { // Blocks forever
        fmt.Println(v)
    }
}
```

**Problem:**
- Deadlock
- Blocking forever

**Solution:**
```go
func rangeWithClose() {
    ch := make(chan int)
    go func() {
        defer close(ch) // Always close
        for i := 0; i < 10; i++ {
            ch <- i
        }
    }()
    for v := range ch {
        fmt.Println(v)
    }
}
```

---

## Best Practices

### 1. Learn Common Anti-patterns

**Why:**
- **Avoid mistakes**: Avoid common mistakes
- **Better code**: Write better code
- **Experience**: Learn from others

**Guidelines:**
- **Study**: Study common anti-patterns
- **Review**: Review code for anti-patterns
- **Learn**: Learn from mistakes

### 2. Use Linters

**Why:**
- **Detect**: Detect anti-patterns
- **Automate**: Automated detection
- **Consistency**: Consistent code

**Guidelines:**
- **golangci-lint**: Use golangci-lint
- **go vet**: Use go vet
- **Static analysis**: Use static analysis

### 3. Code Review

**Why:**
- **Catch**: Catch anti-patterns
- **Learn**: Learn from reviews
- **Improve**: Improve code quality

**Guidelines:**
- **Review**: Review code regularly
- **Feedback**: Give feedback
- **Learn**: Learn from feedback

### 4. Use Race Detector

**Why:**
- **Detect races**: Detect race conditions
- **Concurrency**: Find concurrency issues
- **Safety**: Ensure safety

**Guidelines:**
- **Test**: Test with race detector
- **CI/CD**: Use in CI/CD
- **Development**: Use in development

---

## Summary

Anti-patterns are common mistakes that seem correct but cause problems. Understanding concurrency anti-patterns, error handling anti-patterns, memory anti-patterns, interface anti-patterns, channel anti-patterns, and best practices is crucial for writing better Go code.

**Key Takeaways:**
- **Anti-patterns**: Common mistakes (seem correct, cause problems, avoidable)
- **Concurrency anti-patterns**: Goroutine leak (never exits, memory leak), shared mutable state (race conditions), unbuffered channel deadlock (blocks forever)
- **Error handling anti-patterns**: Ignoring errors (silent failures), wrapping without context (no context), panic for control flow (unexpected behavior)
- **Memory anti-patterns**: Large slice reallocation (many reallocations), memory leak with maps (never cleared), unnecessary pointer usage (heap allocation)
- **Interface anti-patterns**: Overly broad interfaces (hard to implement), interface in wrong package (dependency inversion), empty interface abuse (loss of type safety)
- **Channel anti-patterns**: Closing channel multiple times (panic), sending to closed channel (panic), range over channel without close (deadlock)
- **Best practices**: Learn common anti-patterns, use linters, code review, use race detector

**Anti-patterns to Avoid:**
- **Goroutine leaks**: Always use context/timeout
- **Race conditions**: Use mutex/atomic
- **Error ignoring**: Always handle errors
- **Memory leaks**: Clear resources
- **Channel issues**: Always close channels

**Best Practices:**
- Learn common anti-patterns
- Use linters
- Code review
- Use race detector

**Next Steps:**
- Study anti-patterns
- Review code
- Apply best practices
- Learn from mistakes

