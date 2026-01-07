# Go Slices and Arrays Deep Dive - Complete Understanding

## Table of Contents
1. [What are Arrays?](#what-are-arrays)
2. [What are Slices?](#what-are-slices)
3. [Arrays vs Slices](#arrays-vs-slices)
4. [Slice Operations](#slice-operations)
5. [Slice Internals](#slice-internals)
6. [Best Practices](#best-practices)

---

## What are Arrays?

### Definition

**Array**: Fixed-size collection of elements of the same type.

**Characteristics:**
- **Fixed size**: Size determined at declaration
- **Value type**: Copied when assigned
- **Memory**: Contiguous memory allocation
- **Indexed**: Zero-based indexing

### Array Declaration

```go
// Array of 5 integers
var arr [5]int

// Array with values
arr := [5]int{1, 2, 3, 4, 5}

// Array with inferred size
arr := [...]int{1, 2, 3, 4, 5}

// Array with specific indices
arr := [5]int{1: 10, 3: 30}
```

**Key Points:**
- **Size**: Part of type `[5]int` vs `[10]int` are different types
- **Zero values**: Elements initialized to zero value
- **Immutable size**: Cannot change size after declaration

---

## What are Slices?

### Definition

**Slice**: Dynamic, flexible view into an underlying array.

**Characteristics:**
- **Dynamic size**: Can grow and shrink
- **Reference type**: Points to underlying array
- **Flexible**: More flexible than arrays
- **Common**: More commonly used than arrays

### Slice Declaration

```go
// Nil slice
var slice []int

// Slice literal
slice := []int{1, 2, 3, 4, 5}

// Slice from array
arr := [5]int{1, 2, 3, 4, 5}
slice := arr[:]

// Slice with make
slice := make([]int, 5)        // Length 5
slice := make([]int, 5, 10)    // Length 5, capacity 10
```

**Slice Structure:**
```go
type slice struct {
    ptr    *T      // Pointer to underlying array
    len    int     // Length (number of elements)
    cap    int     // Capacity (max elements without reallocation)
}
```

---

## Arrays vs Slices

### Comparison Table

| Aspect | Arrays | Slices |
|--------|--------|--------|
| Size | Fixed | Dynamic |
| Type | Value type | Reference type |
| Copy | Full copy | Reference copy |
| Usage | Rare | Common |
| Memory | Contiguous | Points to array |

### When to Use Each

**Use Arrays When:**
- **Fixed size**: Size known at compile time
- **Value semantics**: Need value semantics
- **Performance**: Slight performance benefit
- **Rare**: Rarely used in practice

**Use Slices When:**
- **Dynamic size**: Size varies
- **Common case**: Most common use case
- **Flexibility**: Need flexibility
- **Standard**: Standard practice in Go

---

## Slice Operations

### Append Operation

```go
slice := []int{1, 2, 3}

// Append single element
slice = append(slice, 4)

// Append multiple elements
slice = append(slice, 5, 6, 7)

// Append slice
other := []int{8, 9}
slice = append(slice, other...)
```

**Behavior:**
- **Growth**: Automatically grows capacity if needed
- **New slice**: Returns new slice (may be same or new)
- **Efficient**: Efficient growth strategy

### Slice Slicing

```go
slice := []int{1, 2, 3, 4, 5}

// Slice from index 1 to 3 (exclusive)
subSlice := slice[1:3]  // [2, 3]

// Slice from start
subSlice := slice[:3]   // [1, 2, 3]

// Slice to end
subSlice := slice[2:]   // [3, 4, 5]

// Full slice
subSlice := slice[:]    // [1, 2, 3, 4, 5]
```

**Important:**
- **Shares memory**: Shares underlying array
- **Capacity**: Capacity is from start of slice to end of array
- **Modification**: Modifying slice affects original

### Copy Operation

```go
src := []int{1, 2, 3, 4, 5}
dst := make([]int, 3)

// Copy elements
n := copy(dst, src)  // n = 3, dst = [1, 2, 3]
```

**Behavior:**
- **Copies elements**: Copies actual elements
- **Independent**: Creates independent slice
- **Length**: Copies min(len(dst), len(src))

---

## Slice Internals

### Slice Header

**Structure:**
```go
type slice struct {
    ptr *T    // Pointer to underlying array
    len int   // Length
    cap int   // Capacity
}
```

**Memory Layout:**
```
Slice Header (24 bytes on 64-bit)
├── ptr (8 bytes) → Points to array
├── len (8 bytes) → Length
└── cap (8 bytes) → Capacity
```

### Capacity Growth

**Growth Strategy:**
```
Small slices (< 1024): 2x growth
Large slices (>= 1024): 1.25x growth
```

**Example:**
```go
slice := make([]int, 0, 1)
// Append until capacity grows
// 1 → 2 → 4 → 8 → 16 → ...
```

---

## Best Practices

### 1. Prefer Slices Over Arrays

**Why:**
- **Flexibility**: More flexible
- **Common**: Standard practice
- **Dynamic**: Dynamic sizing

**Guidelines:**
- **Use slices**: Use slices in most cases
- **Arrays rarely**: Use arrays only when needed
- **Flexibility**: Prefer flexibility

### 2. Pre-allocate Capacity When Known

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Memory**: Less memory reallocation

**Guidelines:**
- **Known size**: Pre-allocate when size known
- **make with cap**: Use make with capacity
- **Avoid growth**: Avoid unnecessary growth

### 3. Be Careful with Slice Slicing

**Why:**
- **Shared memory**: Slices share memory
- **Modifications**: Modifications affect original
- **Unexpected behavior**: Can cause unexpected behavior

**Guidelines:**
- **Be aware**: Be aware of shared memory
- **Copy if needed**: Copy if need independent slice
- **Document**: Document if sharing is intentional

### 4. Use copy for Independent Slices

**Why:**
- **Independence**: Creates independent slice
- **Safety**: Safe modifications
- **Clarity**: Clear intent

**Guidelines:**
- **Independent**: Use copy for independent slices
- **Modifications**: Use when need to modify independently
- **Clarity**: Makes intent clear

---

## Summary

Slices and arrays are fundamental data structures in Go. Understanding arrays, slices, differences, operations, internals, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Arrays**: Fixed-size collection (value type, fixed size, rarely used)
- **Slices**: Dynamic view into array (reference type, dynamic size, commonly used)
- **Arrays vs Slices**: Arrays (fixed, value type) vs Slices (dynamic, reference type)
- **Slice operations**: Append (grows automatically), slicing (shares memory), copy (independent)
- **Slice internals**: Slice header (ptr, len, cap), capacity growth strategy
- **Best practices**: Prefer slices, pre-allocate capacity, be careful with slicing, use copy for independence

**Key Differences:**
- **Size**: Arrays fixed, slices dynamic
- **Type**: Arrays value type, slices reference type
- **Usage**: Arrays rare, slices common

**Best Practices:**
- Prefer slices over arrays
- Pre-allocate capacity when known
- Be careful with slice slicing
- Use copy for independent slices

**Next Steps:**
- Practice slice operations
- Understand slice internals
- Learn capacity management
- Apply best practices

