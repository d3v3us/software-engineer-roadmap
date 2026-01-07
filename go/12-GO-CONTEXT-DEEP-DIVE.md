# Go Context Deep Dive - Complete Understanding

## Table of Contents
1. [What is Context?](#what-is-context)
2. [Why Use Context?](#why-use-context)
3. [Context Interface](#context-interface)
4. [Context Creation](#context-creation)
5. [Context Patterns](#context-patterns)
6. [Best Practices](#best-practices)

---

## What is Context?

### Definition

**Context**: Package for carrying deadlines, cancellation signals, and request-scoped values.

**Key Characteristics:**
- **Deadlines**: Carry deadlines
- **Cancellation**: Cancellation signals
- **Values**: Request-scoped values
- **Propagation**: Propagate through call chain

### Real-World Analogy

**Context = Request Information:**
- **Request**: HTTP request
- **Context**: Request information
- **Propagation**: Passed through call chain
- **Cancellation**: Can cancel request

**Programming:**
- **Request**: Function call chain
- **Context**: Request context
- **Cancellation**: Cancel operations
- **Values**: Share values

---

## Why Use Context?

### Problems Context Solves

**1. Cancellation:**
```
Long-running operations
  ↓
Need cancellation
  ↓
Context provides cancellation
```

**2. Timeouts:**
```
Operations with timeouts
  ↓
Need timeout mechanism
  ↓
Context provides deadlines
```

**3. Request Scoping:**
```
Request-scoped values
  ↓
Need to pass values
  ↓
Context carries values
```

---

## Context Interface

### Context Definition

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

**Methods:**
- **Deadline**: Returns deadline if set
- **Done**: Returns channel that closes when context cancelled
- **Err**: Returns error if context cancelled
- **Value**: Returns value for key

---

## Context Creation

### Background Context

```go
ctx := context.Background()  // Root context
```

**Use:**
- **Root context**: Root of context tree
- **Main function**: Main function
- **Tests**: Tests

### TODO Context

```go
ctx := context.TODO()  // Placeholder context
```

**Use:**
- **Placeholder**: When context not available yet
- **Migration**: During migration to context
- **Temporary**: Temporary use

### WithCancel

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()  // Cancel when done
```

**Use:**
- **Cancellation**: Manual cancellation
- **Control**: Control operation cancellation

### WithTimeout

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
```

**Use:**
- **Timeouts**: Operation timeouts
- **Deadlines**: Set deadlines

### WithDeadline

```go
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()
```

**Use:**
- **Specific deadline**: Specific deadline time
- **Absolute time**: Absolute deadline

### WithValue

```go
ctx := context.WithValue(context.Background(), "key", "value")
value := ctx.Value("key")
```

**Use:**
- **Request values**: Request-scoped values
- **Metadata**: Request metadata
- **Tracing**: Tracing information

---

## Context Patterns

### Pattern 1: Cancellation

```go
func operation(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case result := <-doWork():
        return result
    }
}
```

### Pattern 2: Timeout

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

err := operation(ctx)
if err != nil {
    // Handle timeout or cancellation
}
```

### Pattern 3: Request Scoped Values

```go
type key string

const userKey key = "user"

func withUser(ctx context.Context, user string) context.Context {
    return context.WithValue(ctx, userKey, user)
}

func getUser(ctx context.Context) string {
    if user, ok := ctx.Value(userKey).(string); ok {
        return user
    }
    return ""
}
```

### Pattern 4: HTTP Request Context

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Use context in operations
    result, err := operation(ctx)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    w.Write(result)
}
```

---

## Best Practices

### 1. Pass Context as First Parameter

**Why:**
- **Convention**: Go convention
- **Consistency**: Consistent API
- **Clarity**: Clear intent

**Guidelines:**
- **First parameter**: Context as first parameter
- **Consistent**: Consistent across functions
- **Document**: Document if context optional

### 2. Don't Store Context in Structs

**Why:**
- **Request-scoped**: Context is request-scoped
- **Lifetime**: Context lifetime is request lifetime
- **Confusion**: Storing causes confusion

**Guidelines:**
- **Pass explicitly**: Pass context explicitly
- **Don't store**: Don't store in structs
- **Request-scoped**: Keep request-scoped

### 3. Check Context Cancellation

**Why:**
- **Responsiveness**: Responsive cancellation
- **Resource cleanup**: Proper resource cleanup
- **User experience**: Better user experience

**Guidelines:**
- **Check Done**: Check ctx.Done() in loops
- **Return early**: Return early on cancellation
- **Propagate**: Propagate cancellation

### 4. Use Context for Timeouts

**Why:**
- **Timeouts**: Built-in timeout support
- **Deadlines**: Deadline support
- **Consistency**: Consistent timeout handling

**Guidelines:**
- **WithTimeout**: Use WithTimeout for timeouts
- **WithDeadline**: Use WithDeadline for deadlines
- **Always cancel**: Always call cancel

---

## Summary

Context is essential for cancellation, timeouts, and request-scoped values in Go. Understanding context interface, creation, patterns, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Context**: Package for deadlines, cancellation, request-scoped values
- **Context interface**: Deadline, Done, Err, Value methods
- **Context creation**: Background, TODO, WithCancel, WithTimeout, WithDeadline, WithValue
- **Context patterns**: Cancellation, timeout, request-scoped values, HTTP request context
- **Best practices**: Pass as first parameter, don't store in structs, check cancellation, use for timeouts

**Context Use Cases:**
- **Cancellation**: Cancel operations
- **Timeouts**: Operation timeouts
- **Values**: Request-scoped values

**Best Practices:**
- Pass context as first parameter
- Don't store context in structs
- Check context cancellation
- Use context for timeouts

**Next Steps:**
- Practice context patterns
- Learn context propagation
- Master cancellation
- Apply best practices

