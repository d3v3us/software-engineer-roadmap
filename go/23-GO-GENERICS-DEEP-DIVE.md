# Go Generics Deep Dive - Complete Understanding

## Table of Contents
1. [What are Generics?](#what-are-generics)
2. [Why Generics?](#why-generics)
3. [Generic Functions](#generic-functions)
4. [Generic Types](#generic-types)
5. [Type Constraints](#type-constraints)
6. [Best Practices](#best-practices)

---

## What are Generics?

### Definition

**Generics**: Ability to write code that works with multiple types.

**Key Characteristics:**
- **Type parameters**: Type parameters
- **Reusability**: Reusable code
- **Type safety**: Type safety maintained
- **Go 1.18+**: Available from Go 1.18

### Real-World Analogy

**Generics = Template:**
- **Template**: Generic code
- **Types**: Different types
- **Reuse**: Reuse same template
- **Flexibility**: Flexible code

**Programming:**
- **Generic code**: Code with type parameters
- **Types**: Multiple types
- **Reuse**: Reusable across types
- **Type safety**: Type-safe

---

## Why Generics?

### Problems Generics Solve

**1. Code Duplication:**
```
Same logic
  ↓
Different types
  ↓
Code duplication
```

**2. Type Safety:**
```
interface{} loses
  ↓
Type safety
  ↓
Generics maintain
```

**3. Performance:**
```
Reflection is slow
  ↓
Generics are fast
  ↓
Better performance
```

---

## Generic Functions

### Basic Generic Function

```go
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Usage
numbers := []int{1, 2, 3, 4, 5}
doubled := Map(numbers, func(n int) int {
    return n * 2
})
```

### Generic Function with Constraints

```go
func Max[T comparable](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

---

## Generic Types

### Generic Struct

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```

### Generic Interface

```go
type Container[T any] interface {
    Add(T)
    Get(int) T
    Size() int
}
```

---

## Type Constraints

### Built-in Constraints

**any:**
```go
func Print[T any](value T) {
    fmt.Println(value)
}
```

**comparable:**
```go
func Contains[T comparable](slice []T, item T) bool {
    for _, v := range slice {
        if v == item {
            return true
        }
    }
    return false
}
```

### Custom Constraints

```go
type Numeric interface {
    int | int64 | float64
}

func Sum[T Numeric](numbers []T) T {
    var sum T
    for _, n := range numbers {
        sum += n
    }
    return sum
}
```

---

## Best Practices

### 1. Use Generics When Appropriate

**Why:**
- **Clarity**: Clear when appropriate
- **Performance**: Better performance than reflection
- **Type safety**: Maintains type safety

**Guidelines:**
- **Type parameters**: Use when need type parameters
- **Reusability**: Use for reusable code
- **Avoid overuse**: Don't overuse generics

### 2. Keep Constraints Simple

**Why:**
- **Clarity**: Clear constraints
- **Flexibility**: More flexible
- **Maintainability**: Easier to maintain

**Guidelines:**
- **Simple constraints**: Keep constraints simple
- **Built-in**: Use built-in constraints when possible
- **Custom**: Use custom constraints when needed

### 3. Prefer Generics Over Reflection

**Why:**
- **Performance**: Better performance
- **Type safety**: Type safety
- **Compile-time**: Compile-time checks

**Guidelines:**
- **Generics**: Prefer generics over reflection
- **Performance**: Better performance
- **Type safety**: Maintains type safety

---

## Summary

Generics enable type-parameterized code in Go (Go 1.18+). Understanding generic functions, types, constraints, and best practices is crucial for modern Go programming.

**Key Takeaways:**
- **Generics**: Code that works with multiple types (type parameters, reusability, type safety, Go 1.18+)
- **Generic functions**: Functions with type parameters (Map, Filter, etc.)
- **Generic types**: Types with type parameters (Stack, Queue, etc.)
- **Type constraints**: Constraints on type parameters (any, comparable, custom constraints)
- **Best practices**: Use when appropriate, keep constraints simple, prefer over reflection

**Generics Benefits:**
- **Code reuse**: Reusable code
- **Type safety**: Maintains type safety
- **Performance**: Better than reflection

**Best Practices:**
- Use generics when appropriate
- Keep constraints simple
- Prefer generics over reflection

**Next Steps:**
- Learn generic syntax
- Practice generic functions
- Master type constraints
- Apply best practices

