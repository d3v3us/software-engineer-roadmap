# Go Type Conversions Deep Dive - Complete Understanding

## Table of Contents
1. [What are Type Conversions?](#what-are-type-conversions)
2. [Type Conversions vs Type Assertions](#type-conversions-vs-type-assertions)
3. [Basic Type Conversions](#basic-type-conversions)
4. [String Conversions](#string-conversions)
5. [Type Assertions](#type-assertions)
6. [Type Switches](#type-switches)
7. [Best Practices](#best-practices)

---

## What are Type Conversions?

### Definition

**Type Conversion**: Converting a value from one type to another.

**Key Characteristics:**
- **Explicit**: Must be explicit in Go
- **Type safety**: Compile-time type checking
- **Compatibility**: Types must be compatible
- **No implicit**: No implicit conversions

### Real-World Analogy

**Type Conversion = Translation:**
- **Language A**: Source type
- **Language B**: Target type
- **Translation**: Conversion process
- **Explicit**: Must be explicit

**Programming:**
- **Source type**: Original type
- **Target type**: Desired type
- **Conversion**: Explicit conversion
- **Safety**: Type-safe conversion

---

## Type Conversions vs Type Assertions

### Type Conversions

**What:**
```
Convert between compatible types
  ↓
Numeric types
  ↓
Explicit conversion
```

**Example:**
```go
var x int = 42
var y float64 = float64(x)  // Conversion
```

### Type Assertions

**What:**
```
Extract concrete type from interface
  ↓
Interface types
  ↓
Type assertion
```

**Example:**
```go
var i interface{} = "hello"
s := i.(string)  // Type assertion
```

---

## Basic Type Conversions

### Numeric Conversions

```go
// int to float64
var i int = 42
var f float64 = float64(i)

// float64 to int (truncation)
var f float64 = 3.14
var i int = int(f)  // i = 3

// int to int32
var i int = 42
var i32 int32 = int32(i)

// int32 to int
var i32 int32 = 42
var i int = int(i32)
```

**Rules:**
- **Explicit**: Must be explicit
- **Compatible**: Types must be compatible
- **Loss**: May lose precision

### Boolean Conversions

```go
// No direct boolean conversion
// Must use explicit logic
var b bool = (x != 0)
```

**Note:** Go doesn't allow direct boolean conversions.

---

## String Conversions

### String to Numeric

```go
import "strconv"

// String to int
s := "42"
i, err := strconv.Atoi(s)

// String to int64
i64, err := strconv.ParseInt(s, 10, 64)

// String to float64
f, err := strconv.ParseFloat("3.14", 64)
```

### Numeric to String

```go
import "strconv"

// int to string
i := 42
s := strconv.Itoa(i)

// int64 to string
i64 := int64(42)
s := strconv.FormatInt(i64, 10)

// float64 to string
f := 3.14
s := strconv.FormatFloat(f, 'f', 2, 64)
```

### String to Byte Slice

```go
s := "hello"
b := []byte(s)  // String to []byte
s2 := string(b) // []byte to string
```

---

## Type Assertions

### Basic Type Assertion

```go
var i interface{} = "hello"

// Type assertion
s := i.(string)  // s is string

// Type assertion with ok check
s, ok := i.(string)
if ok {
    // s is string
}
```

**Behavior:**
- **Single value**: Panics if type wrong
- **Two values**: Returns false if type wrong
- **Type check**: Checks interface value type

### Type Assertion Examples

```go
var i interface{} = 42

// Assert to int
if n, ok := i.(int); ok {
    fmt.Println("It's an int:", n)
}

// Assert to string (will fail)
if s, ok := i.(string); ok {
    fmt.Println("It's a string:", s)
} else {
    fmt.Println("Not a string")
}
```

---

## Type Switches

### Type Switch Syntax

```go
func process(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Println("It's an int:", v)
    case string:
        fmt.Println("It's a string:", v)
    case bool:
        fmt.Println("It's a bool:", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

**Characteristics:**
- **Type checking**: Checks multiple types
- **Variable**: Creates variable of correct type
- **Default**: Handles unknown types

### Type Switch Examples

```go
func printType(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case float64:
        fmt.Printf("Float: %f\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case []int:
        fmt.Printf("Slice: %v\n", v)
    default:
        fmt.Printf("Unknown: %T\n", v)
    }
}
```

---

## Best Practices

### 1. Use Explicit Conversions

**Why:**
- **Clarity**: Clear intent
- **Safety**: Type safety
- **Readability**: Better readability

**Guidelines:**
- **Explicit**: Always explicit
- **Clear**: Make conversions clear
- **Document**: Document if needed

### 2. Handle Conversion Errors

**Why:**
- **Reliability**: Reliable conversions
- **Error handling**: Proper error handling
- **Robustness**: Robust code

**Guidelines:**
- **Check errors**: Always check conversion errors
- **Handle failures**: Handle conversion failures
- **Validate**: Validate converted values

### 3. Use Type Assertions Safely

**Why:**
- **Safety**: Avoid panics
- **Reliability**: Reliable code
- **Correctness**: Correct behavior

**Guidelines:**
- **Two-value form**: Use two-value form
- **Check ok**: Always check ok value
- **Handle failures**: Handle assertion failures

### 4. Prefer Type Switches for Multiple Types

**Why:**
- **Clarity**: Clear type handling
- **Efficiency**: Efficient type checking
- **Maintainability**: Easier to maintain

**Guidelines:**
- **Multiple types**: Use for multiple types
- **Default case**: Include default case
- **Clear logic**: Keep logic clear

---

## Summary

Type conversions and assertions are essential in Go. Understanding conversions, assertions, type switches, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Type conversions**: Convert between compatible types (explicit, type-safe)
- **Type assertions**: Extract concrete type from interface (single-value or two-value form)
- **Type switches**: Check multiple types (switch on type)
- **String conversions**: Use strconv package (Atoi, Itoa, ParseInt, FormatInt)
- **Best practices**: Use explicit conversions, handle errors, use assertions safely, prefer type switches

**Conversion Rules:**
- **Explicit**: Must be explicit
- **Compatible**: Types must be compatible
- **No implicit**: No implicit conversions

**Best Practices:**
- Use explicit conversions
- Handle conversion errors
- Use type assertions safely
- Prefer type switches for multiple types

**Next Steps:**
- Practice type conversions
- Learn type assertions
- Master type switches
- Apply best practices

