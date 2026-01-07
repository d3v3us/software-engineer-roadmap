# Go Functions and Methods Deep Dive - Complete Understanding

## Table of Contents
1. [What are Functions?](#what-are-functions)
2. [What are Methods?](#what-are-methods)
3. [Function Variants](#function-variants)
4. [Method Variants](#method-variants)
5. [Functions vs Methods](#functions-vs-methods)
6. [Best Practices](#best-practices)

---

## What are Functions?

### Definition

**Function**: Block of code that performs a specific task.

**Key Characteristics:**
- **Reusable**: Can be called multiple times
- **Parameters**: Can take parameters
- **Return values**: Can return values
- **First-class**: First-class citizens in Go

### Function Declaration

```go
func functionName(param1 type1, param2 type2) returnType {
    // Function body
    return value
}
```

---

## What are Methods?

### Definition

**Method**: Function with a receiver.

**Key Characteristics:**
- **Receiver**: Has a receiver (type)
- **Associated**: Associated with a type
- **Behavior**: Defines behavior for type
- **Object-oriented**: Object-oriented style

### Method Declaration

```go
func (receiver ReceiverType) methodName(param type) returnType {
    // Method body
    return value
}
```

---

## Function Variants

### Multiple Return Values

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

### Named Return Values

```go
func calculate(x, y int) (sum int, product int) {
    sum = x + y
    product = x * y
    return  // Naked return
}
```

### Variadic Functions

```go
func sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}

// Usage
sum(1, 2, 3, 4, 5)
```

### Function as Values

```go
// Function variable
var fn func(int, int) int

fn = func(a, b int) int {
    return a + b
}

result := fn(1, 2)
```

### Closures

```go
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

c := counter()
fmt.Println(c())  // 1
fmt.Println(c())  // 2
```

---

## Method Variants

### Value Receiver

```go
type Rectangle struct {
    Width  float64
    Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}
```

**Characteristics:**
- **Copy**: Receives copy of value
- **No modification**: Cannot modify receiver
- **Small structs**: Good for small structs

### Pointer Receiver

```go
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}
```

**Characteristics:**
- **Reference**: Receives pointer
- **Modification**: Can modify receiver
- **Large structs**: Good for large structs

### Method Sets

**Value Receiver:**
- Methods available on both value and pointer

**Pointer Receiver:**
- Methods available only on pointer

---

## Functions vs Methods

### When to Use Functions

**Use Functions When:**
- **No receiver**: No specific type association
- **Utility**: General utility functions
- **Stateless**: Stateless operations

**Example:**
```go
func Max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### When to Use Methods

**Use Methods When:**
- **Type behavior**: Behavior specific to type
- **Encapsulation**: Encapsulate type behavior
- **Object-oriented**: Object-oriented design

**Example:**
```go
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}
```

---

## Best Practices

### 1. Keep Functions Small

**Why:**
- **Readability**: Better readability
- **Maintainability**: Easier to maintain
- **Testing**: Easier to test

**Guidelines:**
- **Single responsibility**: One responsibility per function
- **Small**: Keep functions small
- **Focused**: Focused functionality

### 2. Use Meaningful Names

**Why:**
- **Clarity**: Clear intent
- **Readability**: Better readability
- **Documentation**: Self-documenting

**Guidelines:**
- **Descriptive**: Use descriptive names
- **Verb**: Use verbs for functions
- **Consistent**: Consistent naming

### 3. Prefer Methods for Type Behavior

**Why:**
- **Encapsulation**: Better encapsulation
- **Organization**: Better organization
- **Clarity**: Clear type behavior

**Guidelines:**
- **Type behavior**: Use methods for type behavior
- **Functions**: Use functions for utilities
- **Appropriate**: Use appropriately

### 4. Use Pointer Receivers When Modifying

**Why:**
- **Modification**: Need to modify receiver
- **Efficiency**: More efficient for large structs
- **Consistency**: Consistent API

**Guidelines:**
- **Modification**: Use pointer receivers when modifying
- **Large structs**: Use for large structs
- **Consistency**: Be consistent

---

## Summary

Functions and methods are fundamental in Go. Understanding function variants, method variants, differences, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Functions**: Blocks of code performing tasks (reusable, parameters, return values, first-class)
- **Methods**: Functions with receivers (associated with type, behavior, object-oriented style)
- **Function variants**: Multiple returns, named returns, variadic, function values, closures
- **Method variants**: Value receiver (copy, no modification) vs Pointer receiver (reference, modification)
- **Functions vs Methods**: Functions (no receiver, utilities) vs Methods (type behavior, encapsulation)
- **Best practices**: Keep functions small, use meaningful names, prefer methods for type behavior, use pointer receivers when modifying

**Key Differences:**
- **Functions**: No receiver, general purpose
- **Methods**: Has receiver, type-specific

**Best Practices:**
- Keep functions small
- Use meaningful names
- Prefer methods for type behavior
- Use pointer receivers when modifying

**Next Steps:**
- Practice function variants
- Learn method receivers
- Master closures
- Apply best practices

