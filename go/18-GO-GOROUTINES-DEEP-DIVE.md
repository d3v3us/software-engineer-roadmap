# Go Goroutines Deep Dive - Complete Understanding

## Table of Contents
1. [What are Goroutines?](#what-are-goroutines)
2. [Why Use Goroutines?](#why-use-goroutines)
3. [Goroutine Creation](#goroutine-creation)
4. [Goroutine Lifecycle](#goroutine-lifecycle)
5. [Goroutine Scheduling](#goroutine-scheduling)
6. [Goroutine Patterns](#goroutine-patterns)
7. [Best Practices](#best-practices)

---

## What are Goroutines?

### Definition

**Goroutine**: Lightweight thread managed by Go runtime.

**Key Characteristics:**
- **Lightweight**: Very small stack (2KB initially)
- **Managed**: Managed by Go runtime
- **Efficient**: Efficient context switching
- **Scalable**: Can create millions

### Real-World Analogy

**Goroutine = Worker:**
- **Workers**: Multiple workers
- **Tasks**: Different tasks
- **Coordination**: Coordination needed
- **Efficiency**: Efficient work distribution

**Programming:**
- **Goroutines**: Concurrent execution units
- **Tasks**: Different tasks
- **Coordination**: Channels for coordination
- **Efficiency**: Efficient execution

---

## Why Use Goroutines?

### Benefits

**1. Concurrency:**
```
Multiple tasks
  ↓
Concurrent execution
  ↓
Better utilization
```

**2. Efficiency:**
```
Lightweight
  ↓
Low overhead
  ↓
Many goroutines
```

**3. Simplicity:**
```
Simple syntax
  ↓
Easy to use
  ↓
Productive
```

---

## Goroutine Creation

### Basic Creation

```go
// Launch goroutine
go function()

// Anonymous function
go func() {
    // code
}()

// With parameters
go func(x int) {
    // code
}(42)
```

### Goroutine with Channels

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch
```

---

## Goroutine Lifecycle

### Lifecycle Stages

```
Goroutine created
  ↓
Scheduled by runtime
  ↓
Executes
  ↓
Completes or blocked
  ↓
Terminated
```

### Goroutine States

**1. Runnable:**
- Ready to execute
- Waiting for CPU

**2. Running:**
- Currently executing
- On OS thread

**3. Blocked:**
- Waiting for channel
- Waiting for I/O

---

## Goroutine Scheduling

### M:N Scheduler

**Model:**
- **M goroutines**: Many goroutines
- **N OS threads**: Fewer OS threads
- **Efficient**: Efficient scheduling

**Benefits:**
- **Efficiency**: Efficient CPU utilization
- **Scalability**: Scalable to many goroutines
- **Performance**: Good performance

### Work Stealing

**Mechanism:**
- **Idle threads**: Steal work from busy threads
- **Load balancing**: Balance load across threads
- **Efficiency**: Efficient CPU utilization

---

## Goroutine Patterns

### Pattern 1: Simple Goroutine

```go
go func() {
    fmt.Println("Hello from goroutine")
}()
```

### Pattern 2: Worker Pool

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        results <- process(job)
    }
}

// Usage
jobs := make(chan int, 100)
results := make(chan int, 100)

for w := 1; w <= 10; w++ {
    go worker(w, jobs, results)
}
```

### Pattern 3: Fan-out/Fan-in

```go
// Fan-out: Distribute work
func fanOut(input <-chan int, outputs []chan int) {
    for value := range input {
        for _, out := range outputs {
            out <- value
        }
    }
}

// Fan-in: Collect results
func fanIn(inputs []<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            out <- n
        }
    }
    
    wg.Add(len(inputs))
    for _, c := range inputs {
        go output(c)
    }
    
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

---

## Best Practices

### 1. Don't Create Too Many Goroutines

**Why:**
- **Overhead**: Each goroutine has overhead
- **Resources**: Consumes resources
- **Performance**: Too many can hurt performance

**Guidelines:**
- **Worker pools**: Use worker pools
- **Limit**: Limit number of goroutines
- **Reuse**: Reuse goroutines when possible

### 2. Use Channels for Communication

**Why:**
- **Safety**: Prevents race conditions
- **Clarity**: Clear communication patterns
- **Synchronization**: Built-in synchronization

**Guidelines:**
- **Channels**: Use channels for communication
- **Avoid shared memory**: Avoid shared memory
- **Go philosophy**: Follow Go philosophy

### 3. Use sync.WaitGroup for Coordination

**Why:**
- **Coordination**: Coordinate goroutines
- **Synchronization**: Wait for completion
- **Clarity**: Clear coordination

**Guidelines:**
- **WaitGroup**: Use for waiting
- **Add before**: Add before goroutine
- **Done in goroutine**: Call Done in goroutine

### 4. Handle Goroutine Leaks

**Why:**
- **Memory leaks**: Can cause memory leaks
- **Resource leaks**: Can cause resource leaks
- **Performance**: Can hurt performance

**Guidelines:**
- **Close channels**: Close channels properly
- **Context cancellation**: Use context for cancellation
- **Monitor**: Monitor goroutine counts

---

## Summary

Goroutines are essential for concurrency in Go. Understanding goroutine creation, lifecycle, scheduling, patterns, and best practices is crucial for effective concurrent programming.

**Key Takeaways:**
- **Goroutines**: Lightweight threads managed by Go runtime (lightweight, efficient, scalable)
- **Goroutine creation**: `go` keyword (function, anonymous function, with parameters)
- **Goroutine lifecycle**: Created → Scheduled → Executes → Completes/Blocked → Terminated
- **Goroutine scheduling**: M:N scheduler (many goroutines, fewer threads, work stealing)
- **Goroutine patterns**: Simple goroutine, worker pool, fan-out/fan-in
- **Best practices**: Don't create too many, use channels, use WaitGroup, handle leaks

**Goroutine Benefits:**
- **Concurrency**: Multiple tasks concurrently
- **Efficiency**: Lightweight and efficient
- **Simplicity**: Simple syntax

**Best Practices:**
- Don't create too many goroutines
- Use channels for communication
- Use sync.WaitGroup for coordination
- Handle goroutine leaks

**Next Steps:**
- Practice goroutine creation
- Learn goroutine patterns
- Master coordination
- Apply best practices

