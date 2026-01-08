# Go Method Values and Expressions Deep Dive - Complete Understanding

## Table of Contents
1. [What are Method Values?](#what-are-method-values)
2. [What are Method Expressions?](#what-are-method-expressions)
3. [Method Values vs Method Expressions](#method-values-vs-method-expressions)
4. [Method Values Usage](#method-values-usage)
5. [Method Expressions Usage](#method-expressions-usage)
6. [Closures vs Method Values](#closures-vs-method-values)
7. [Performance Implications](#performance-implications)
8. [Best Practices](#best-practices)

---

## What are Method Values?

### Definition

**Method Value**: Function value that binds a method to a specific receiver value.

**Key Characteristics:**
- **Bound receiver**: Receiver value is bound
- **Function value**: Can be stored and called
- **Closure-like**: Similar to closure
- **Convenient**: Convenient for callbacks

### Real-World Analogy

**Method Value = Bound Function:**
- **Method**: Function with receiver
- **Receiver**: Specific instance
- **Bound**: Method bound to instance
- **Callable**: Can be called later

**Programming:**
- **Method**: Method definition
- **Receiver**: Specific receiver value
- **Method value**: Bound method
- **Usage**: Callbacks, function parameters

---

## Why Method Values Matter?

### Benefits

**1. Callbacks:**
```
Method
  ↓
Method value
  ↓
Callback function
```

**2. Function Parameters:**
```
Method
  ↓
Method value
  ↓
Function parameter
```

**3. Convenience:**
```
Bound method
  ↓
Method value
  ↓
Easy to use
```

---

## What are Method Expressions?

### Definition

**Method Expression**: Function value that represents a method, with receiver as first parameter.

**Key Characteristics:**
- **Unbound**: Method not bound to receiver
- **First parameter**: Receiver is first parameter
- **Function type**: Regular function type
- **Flexible**: More flexible than method values

### Real-World Analogy

**Method Expression = Unbound Function:**
- **Method**: Function with receiver
- **Unbound**: Not bound to instance
- **Parameter**: Receiver as parameter
- **Callable**: Can be called with any receiver

**Programming:**
- **Method**: Method definition
- **Method expression**: Unbound method
- **Receiver parameter**: Receiver as first parameter
- **Usage**: Generic function calls

---

## Method Values vs Method Expressions

### Comparison

| Aspect | Method Values | Method Expressions |
|--------|---------------|-------------------|
| **Binding** | Bound to receiver | Unbound |
| **Receiver** | Bound in value | First parameter |
| **Type** | `func()` | `func(T)` or `func(*T)` |
| **Usage** | Callbacks | Generic calls |
| **Flexibility** | Less flexible | More flexible |

### Example Comparison

**Method Value:**
```go
type Counter struct {
    value int
}

func (c *Counter) Increment() {
    c.value++
}

var c Counter
mv := c.Increment  // Method value: bound to c
mv()               // Calls c.Increment()
```

**Method Expression:**
```go
me := (*Counter).Increment  // Method expression: unbound
var c Counter
me(&c)  // Calls Increment on &c
```

---

## Method Values Usage

### Basic Usage

```go
type Button struct {
    label string
}

func (b *Button) Click() {
    fmt.Printf("Button %s clicked\n", b.label)
}

var btn Button
btn.label = "Submit"

// Method value
clickHandler := btn.Click
clickHandler()  // Calls btn.Click()
```

### As Function Parameter

```go
func process(fn func()) {
    fn()
}

var btn Button
btn.label = "Submit"

process(btn.Click)  // Pass method value
```

### In Slices

```go
type Handler func()

var handlers []Handler

var btn1, btn2 Button
btn1.label = "Button 1"
btn2.label = "Button 2"

handlers = append(handlers, btn1.Click, btn2.Click)

for _, handler := range handlers {
    handler()
}
```

### With Goroutines

```go
var btn Button
btn.label = "Submit"

go btn.Click()  // Calls immediately
go btn.Click    // Method value: calls later
```

---

## Method Expressions Usage

### Basic Usage

```go
type Counter struct {
    value int
}

func (c *Counter) Increment() {
    c.value++
}

// Method expression
increment := (*Counter).Increment

var c1, c2 Counter
increment(&c1)  // Calls Increment on c1
increment(&c2)  // Calls Increment on c2
```

### With Different Receivers

```go
type Point struct {
    X, Y int
}

func (p Point) Distance() float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}

// Method expression
distance := Point.Distance

p1 := Point{X: 0, Y: 0}
p2 := Point{X: 3, Y: 4}

d1 := distance(p1)  // Distance of p1
d2 := distance(p2)  // Distance of p2
```

### In Generic Functions

```go
func applyToAll(counters []*Counter, fn func(*Counter)) {
    for _, c := range counters {
        fn(c)
    }
}

increment := (*Counter).Increment
counters := []*Counter{&c1, &c2, &c3}
applyToAll(counters, increment)
```

### With Interfaces

```go
type Incrementer interface {
    Increment()
}

func callIncrement(i Incrementer) {
    i.Increment()
}

// Method expression for interface
increment := (*Counter).Increment
var c Counter
callIncrement(&c)  // Works if Counter implements Incrementer
```

---

## Closures vs Method Values

### Closures

**Closure:**
```go
func makeCounter() func() {
    count := 0
    return func() {
        count++
        fmt.Println(count)
    }
}

fn := makeCounter()
fn()  // 1
fn()  // 2
```

**Characteristics:**
- **Captures variables**: Captures variables
- **State**: Maintains state
- **Flexible**: Very flexible

### Method Values

**Method Value:**
```go
type Counter struct {
    count int
}

func (c *Counter) Increment() {
    c.count++
    fmt.Println(c.count)
}

var c Counter
fn := c.Increment
fn()  // 1
fn()  // 2
```

**Characteristics:**
- **Bound receiver**: Bound to receiver
- **State in receiver**: State in receiver
- **Type-safe**: Type-safe

### When to Use Which?

**Use Closures When:**
- **Multiple variables**: Need to capture multiple variables
- **Complex state**: Complex state management
- **Flexibility**: Need maximum flexibility

**Use Method Values When:**
- **Single receiver**: Single receiver object
- **Type safety**: Want type safety
- **Simplicity**: Simpler code

---

## Performance Implications

### Method Values Performance

**Overhead:**
- **Small**: Small overhead
- **Bound receiver**: Receiver bound at creation
- **Function call**: Normal function call overhead

**Example:**
```go
var c Counter
mv := c.Increment  // Small overhead: creates bound function
mv()               // Normal function call
```

### Method Expressions Performance

**Overhead:**
- **Minimal**: Minimal overhead
- **Unbound**: No receiver binding
- **Function call**: Normal function call with receiver parameter

**Example:**
```go
me := (*Counter).Increment  // Minimal overhead
me(&c)                      // Normal function call
```

### Comparison

**Method Values:**
- **Bound**: Receiver bound at creation
- **Overhead**: Small binding overhead
- **Call**: Direct call

**Method Expressions:**
- **Unbound**: No binding
- **Overhead**: Minimal
- **Call**: Call with receiver parameter

---

## Best Practices

### 1. Use Method Values for Callbacks

**Why:**
- **Convenience**: More convenient
- **Type safety**: Type-safe
- **Readability**: More readable

**Guidelines:**
- **Callbacks**: Use for callbacks
- **Event handlers**: Use for event handlers
- **Function parameters**: Use as function parameters

### 2. Use Method Expressions for Generic Operations

**Why:**
- **Flexibility**: More flexible
- **Reusability**: More reusable
- **Generic**: Works with multiple receivers

**Guidelines:**
- **Generic functions**: Use in generic functions
- **Multiple receivers**: Use with multiple receivers
- **Functional programming**: Use in functional patterns

### 3. Understand Performance Trade-offs

**Why:**
- **Performance**: Better performance
- **Optimization**: Better optimization
- **Efficiency**: More efficient

**Guidelines:**
- **Measure**: Measure performance
- **Optimize**: Optimize when needed
- **Balance**: Balance convenience and performance

### 4. Prefer Method Values for Simplicity

**Why:**
- **Simplicity**: Simpler code
- **Readability**: More readable
- **Maintainability**: Easier to maintain

**Guidelines:**
- **Simple cases**: Use for simple cases
- **Callbacks**: Use for callbacks
- **Clarity**: When clarity is important

---

## Summary

Method values and method expressions are powerful features in Go for working with methods as first-class values. Understanding method values, method expressions, their differences, usage patterns, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Method values**: Function value that binds method to specific receiver (bound receiver, function value, closure-like, convenient)
- **Method expressions**: Function value representing method with receiver as first parameter (unbound, first parameter, function type, flexible)
- **Method values vs method expressions**: Method values (bound to receiver, bound in value, type `func()`, callbacks) vs Method expressions (unbound, first parameter, type `func(T)`, generic calls)
- **Method values usage**: Basic usage, as function parameter, in slices, with goroutines
- **Method expressions usage**: Basic usage, with different receivers, in generic functions, with interfaces
- **Closures vs method values**: Closures (captures variables, maintains state, flexible) vs Method values (bound receiver, state in receiver, type-safe)
- **Performance implications**: Method values (small overhead, bound receiver, normal call) vs Method expressions (minimal overhead, unbound, normal call)
- **Best practices**: Use method values for callbacks, use method expressions for generic operations, understand performance trade-offs, prefer method values for simplicity

**Method Values:**
- **Bound**: Receiver bound at creation
- **Usage**: Callbacks, event handlers

**Method Expressions:**
- **Unbound**: Receiver as parameter
- **Usage**: Generic operations, functional patterns

**Best Practices:**
- Use method values for callbacks
- Use method expressions for generic operations
- Understand performance trade-offs
- Prefer method values for simplicity

**Next Steps:**
- Learn method values
- Learn method expressions
- Practice usage patterns
- Apply best practices

