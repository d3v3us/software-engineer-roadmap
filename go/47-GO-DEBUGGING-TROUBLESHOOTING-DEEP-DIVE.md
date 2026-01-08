# Go Debugging and Troubleshooting Deep Dive - Complete Understanding

## Table of Contents
1. [What is Debugging in Go?](#what-is-debugging-in-go)
2. [Why Debugging Matters](#why-debugging-matters)
3. [Debugging Tools](#debugging-tools)
4. [Common Issues](#common-issues)
5. [Debugging Techniques](#debugging-techniques)
6. [Best Practices](#best-practices)

---

## What is Debugging in Go?

### Definition

**Debugging**: Process of finding and fixing bugs in Go programs.

**Key Characteristics:**
- **Bug finding**: Identify bugs
- **Root cause**: Find root cause
- **Fixing**: Fix the bug
- **Verification**: Verify the fix

### Real-World Analogy

**Debugging = Detective Work:**
- **Crime**: Bug
- **Detective**: Developer
- **Clues**: Logs, errors, behavior
- **Solution**: Fix the bug

**Programming:**
- **Bug**: Software bug
- **Debugging**: Finding and fixing
- **Tools**: Debugging tools
- **Techniques**: Debugging techniques

---

## Why Debugging Matters?

### Benefits

**1. Bug Resolution:**
```
Bugs in code
  ↓
Debugging
  ↓
Fixed bugs
```

**2. Understanding:**
```
Code behavior
  ↓
Debugging
  ↓
Better understanding
```

**3. Quality:**
```
Buggy code
  ↓
Debugging
  ↓
Quality code
```

---

## Debugging Tools

### Tool 1: Delve (dlv)

**Go debugger:**
```bash
# Install
go install github.com/go-delve/delve/cmd/dlv@latest

# Debug
dlv debug main.go

# Commands
(dlv) break main.main
(dlv) continue
(dlv) next
(dlv) print variable
(dlv) locals
(dlv) stack
```

**Features:**
- **Breakpoints**: Set breakpoints
- **Step through**: Step through code
- **Variables**: Inspect variables
- **Stack**: View stack trace

### Tool 2: pprof

**Performance profiling:**
```go
import _ "net/http/pprof"

go func() {
    log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```

**Access:**
```bash
go tool pprof http://localhost:6060/debug/pprof/profile
go tool pprof http://localhost:6060/debug/pprof/heap
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

### Tool 3: Race Detector

**Detect race conditions:**
```bash
go run -race main.go
go test -race ./...
```

**Output:**
- **Race conditions**: Detects race conditions
- **Warnings**: Shows warnings
- **Stack traces**: Stack traces

### Tool 4: go vet

**Static analysis:**
```bash
go vet ./...
```

**Checks:**
- **Common errors**: Common errors
- **Suspicious code**: Suspicious code
- **Best practices**: Best practices

### Tool 5: Logging

**Structured logging:**
```go
import "log"

log.Printf("Debug: %v", value)
log.SetFlags(log.LstdFlags | log.Lshortfile)
```

**Structured logging:**
```go
import "github.com/sirupsen/logrus"

log := logrus.New()
log.WithFields(logrus.Fields{
    "key": "value",
}).Info("Message")
```

---

## Common Issues

### Issue 1: Race Conditions

**Problem:**
```go
var counter int

func increment() {
    counter++  // Race condition!
}

go increment()
go increment()
```

**Solution:**
```go
var counter int
var mu sync.Mutex

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}
```

**Detection:**
```bash
go run -race main.go
```

### Issue 2: Goroutine Leaks

**Problem:**
```go
func leak() {
    ch := make(chan int)
    go func() {
        ch <- 42
    }()
    // Missing: <-ch
    // Goroutine blocked forever
}
```

**Solution:**
```go
func noLeak() {
    ch := make(chan int)
    go func() {
        ch <- 42
    }()
    <-ch  // Receive from channel
}
```

**Detection:**
```bash
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

### Issue 3: Deadlocks

**Problem:**
```go
ch := make(chan int)
ch <- 42  // Deadlock: no receiver
```

**Solution:**
```go
ch := make(chan int)
go func() {
    ch <- 42
}()
value := <-ch
```

**Detection:**
- **Hanging**: Application hangs
- **No progress**: No progress
- **pprof**: Use pprof to check

### Issue 4: Nil Pointer Dereference

**Problem:**
```go
var p *int
fmt.Println(*p)  // Panic: nil pointer dereference
```

**Solution:**
```go
var p *int
if p != nil {
    fmt.Println(*p)
}
```

**Detection:**
- **Panic**: Runtime panic
- **Stack trace**: Stack trace shows location

### Issue 5: Memory Leaks

**Problem:**
```go
func leak() {
    data := make([]byte, 1024*1024)
    // data not released
}
```

**Solution:**
```go
func noLeak() {
    data := make([]byte, 1024*1024)
    // Use data
    // GC will collect when out of scope
}
```

**Detection:**
```bash
go tool pprof http://localhost:6060/debug/pprof/heap
```

---

## Debugging Techniques

### Technique 1: Print Debugging

**Simple logging:**
```go
fmt.Printf("Debug: value=%v\n", value)
log.Printf("Debug: step=%d\n", step)
```

**Structured:**
```go
log.WithFields(logrus.Fields{
    "step": step,
    "value": value,
}).Debug("Processing")
```

### Technique 2: Breakpoints

**Using Delve:**
```bash
dlv debug main.go
(dlv) break main.main
(dlv) continue
(dlv) next
```

**Using IDE:**
- **VS Code**: Go extension
- **GoLand**: Built-in debugger
- **Breakpoints**: Set breakpoints

### Technique 3: Stack Traces

**Print stack:**
```go
import "runtime/debug"

debug.PrintStack()
```

**Panic recovery:**
```go
defer func() {
    if r := recover(); r != nil {
        debug.PrintStack()
    }
}()
```

### Technique 4: Conditional Logging

**Debug levels:**
```go
const (
    DebugLevel = iota
    InfoLevel
    ErrorLevel
)

var logLevel = InfoLevel

func debugLog(msg string) {
    if logLevel <= DebugLevel {
        log.Println(msg)
    }
}
```

### Technique 5: Unit Testing

**Test for bugs:**
```go
func TestFunction(t *testing.T) {
    result := function(input)
    if result != expected {
        t.Errorf("Expected %v, got %v", expected, result)
    }
}
```

---

## Best Practices

### 1. Use Appropriate Tools

**Why:**
- **Efficiency**: More efficient debugging
- **Accuracy**: More accurate results
- **Productivity**: Better productivity

**Guidelines:**
- **Delve**: Use for step debugging
- **pprof**: Use for performance issues
- **Race detector**: Use for race conditions
- **go vet**: Use for static analysis

### 2. Add Strategic Logging

**Why:**
- **Visibility**: Better visibility
- **Debugging**: Easier debugging
- **Monitoring**: Better monitoring

**Guidelines:**
- **Structured**: Use structured logging
- **Levels**: Use log levels
- **Context**: Add context
- **Performance**: Consider performance

### 3. Reproduce Issues

**Why:**
- **Understanding**: Better understanding
- **Fixing**: Easier fixing
- **Testing**: Better testing

**Guidelines:**
- **Reproduce**: Reproduce the issue
- **Isolate**: Isolate the problem
- **Test**: Test the fix

### 4. Use Version Control

**Why:**
- **History**: Code history
- **Revert**: Can revert changes
- **Compare**: Compare versions

**Guidelines:**
- **Commit often**: Commit often
- **Descriptive**: Descriptive commits
- **Branch**: Use branches for debugging

---

## Summary

Debugging and troubleshooting are essential skills in Go development. Understanding debugging tools, common issues, techniques, and best practices is crucial for effective bug resolution.

**Key Takeaways:**
- **Debugging**: Process of finding and fixing bugs (bug finding, root cause, fixing, verification)
- **Debugging tools**: Delve (Go debugger: breakpoints step through variables stack), pprof (performance profiling: CPU memory goroutine profiling), race detector (detect race conditions: go run -race), go vet (static analysis: common errors suspicious code), logging (structured logging: log levels context)
- **Common issues**: Race conditions (concurrent access, use mutex, detect with -race), goroutine leaks (blocked goroutines, ensure channels are used, detect with pprof), deadlocks (circular wait, fix ordering, detect with pprof), nil pointer dereference (accessing nil, check for nil, detect with panic), memory leaks (unreleased memory, GC handles, detect with pprof)
- **Debugging techniques**: Print debugging (simple logging, structured logging), breakpoints (Delve, IDE), stack traces (debug.PrintStack, panic recovery), conditional logging (debug levels), unit testing (test for bugs)
- **Best practices**: Use appropriate tools, add strategic logging, reproduce issues, use version control

**Debugging Tools:**
- **Delve**: Step debugging
- **pprof**: Performance profiling
- **Race detector**: Race condition detection
- **go vet**: Static analysis

**Best Practices:**
- Use appropriate tools
- Add strategic logging
- Reproduce issues
- Use version control

**Next Steps:**
- Learn debugging tools
- Practice debugging techniques
- Master troubleshooting
- Apply best practices

