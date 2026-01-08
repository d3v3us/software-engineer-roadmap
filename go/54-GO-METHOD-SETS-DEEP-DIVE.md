# Go Method Sets Deep Dive - Complete Understanding

## Table of Contents
1. [What are Method Sets?](#what-are-method-sets)
2. [Why Method Sets Matter](#why-method-sets-matter)
3. [Method Set Rules](#method-set-rules)
4. [Value Receiver Method Sets](#value-receiver-method-sets)
5. [Pointer Receiver Method Sets](#pointer-receiver-method-sets)
6. [Interface Satisfaction](#interface-satisfaction)
7. [Common Pitfalls](#common-pitfalls)
8. [Best Practices](#best-practices)

---

## What are Method Sets?

### Definition

**Method Set**: Collection of methods available on a type.

**Key Characteristics:**
- **Type-specific**: Each type has its method set
- **Receiver-dependent**: Depends on receiver type
- **Interface satisfaction**: Determines interface satisfaction
- **Compile-time**: Determined at compile time

### Real-World Analogy

**Method Set = Toolbox:**
- **Type**: Person
- **Toolbox**: Method set
- **Tools**: Methods available
- **Access**: What tools can be used

**Programming:**
- **Type**: Go type
- **Method set**: Available methods
- **Methods**: Functions with receivers
- **Interface**: Interface satisfaction

---

## Why Method Sets Matter?

### Benefits

**1. Interface Satisfaction:**
```
Type with methods
  ↓
Method set
  ↓
Interface satisfaction
```

**2. Type Safety:**
```
Compile-time checking
  ↓
Method set rules
  ↓
Type safety
```

**3. Code Clarity:**
```
Clear method availability
  ↓
Method set
  ↓
Better code understanding
```

---

## Method Set Rules

### Rule 1: Value Receiver Methods

**Value receiver methods:**
- Available on both value and pointer
- Can be called on `T` and `*T`

**Example:**
```go
type Point struct {
    X, Y int
}

func (p Point) Distance() float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}

var p Point
var pp *Point = &p

p.Distance()   // OK: value receiver on value
pp.Distance()  // OK: value receiver on pointer (automatically dereferenced)
```

### Rule 2: Pointer Receiver Methods

**Pointer receiver methods:**
- Available only on pointer
- Can be called only on `*T`

**Example:**
```go
func (p *Point) Move(dx, dy int) {
    p.X += dx
    p.Y += dy
}

var p Point
var pp *Point = &p

p.Move(1, 1)   // ERROR: pointer receiver on value
pp.Move(1, 1)  // OK: pointer receiver on pointer
```

### Rule 3: Method Set Summary

| Receiver Type | Available On | Method Set Contains |
|----------------|--------------|---------------------|
| `(t T)` | `T` and `*T` | Methods with value receiver |
| `(t *T)` | `*T` only | Methods with pointer receiver |

---

## Value Receiver Method Sets

### Characteristics

**Value receiver methods:**
- **Copy**: Receives copy of value
- **No modification**: Cannot modify receiver
- **Both types**: Available on value and pointer

**Example:**
```go
type Counter struct {
    value int
}

func (c Counter) Value() int {
    return c.value
}

func (c Counter) Increment() Counter {
    return Counter{value: c.value + 1}
}

var c Counter
var cp *Counter = &c

c.Value()        // OK
cp.Value()       // OK (automatically dereferenced)
c.Increment()    // OK
cp.Increment()   // OK (automatically dereferenced)
```

### Why Available on Both?

**Go automatically handles:**
```go
var c Counter
var cp *Counter = &c

cp.Value()  // Go automatically does: (*cp).Value()
```

**Benefit:**
- **Convenience**: More convenient to use
- **Flexibility**: Works with both value and pointer
- **Consistency**: Consistent behavior

---

## Pointer Receiver Method Sets

### Characteristics

**Pointer receiver methods:**
- **Reference**: Receives pointer
- **Modification**: Can modify receiver
- **Pointer only**: Available only on pointer

**Example:**
```go
type Counter struct {
    value int
}

func (c *Counter) Increment() {
    c.value++
}

func (c *Counter) SetValue(v int) {
    c.value = v
}

var c Counter
var cp *Counter = &c

c.Increment()   // ERROR: cannot use c (type Counter) as *Counter
cp.Increment()   // OK
cp.SetValue(10)  // OK
```

### Why Pointer Only?

**Reason:**
- **Modification**: Need to modify receiver
- **Efficiency**: Avoid copying large structs
- **Consistency**: Consistent pointer usage

---

## Interface Satisfaction

### Interface Satisfaction Rules

**Rule 1: Value Type**
- Satisfies interface if all methods have value receivers
- Or if all methods have pointer receivers (but must use pointer)

**Rule 2: Pointer Type**
- Satisfies interface if all methods have value receivers
- Or if all methods have pointer receivers

### Example 1: Value Receiver Interface

```go
type Reader interface {
    Read() string
}

type File struct {
    content string
}

func (f File) Read() string {
    return f.content
}

var f File
var fp *File = &f

var r1 Reader = f   // OK: File satisfies Reader
var r2 Reader = fp // OK: *File satisfies Reader (value receiver)
```

### Example 2: Pointer Receiver Interface

```go
type Writer interface {
    Write(string)
}

type Buffer struct {
    data string
}

func (b *Buffer) Write(s string) {
    b.data = s
}

var b Buffer
var bp *Buffer = &b

var w1 Writer = b   // ERROR: Buffer does not satisfy Writer
var w2 Writer = bp  // OK: *Buffer satisfies Writer
```

### Example 3: Mixed Interface

```go
type ReadWriter interface {
    Read() string
    Write(string)
}

type File struct {
    content string
}

func (f File) Read() string {
    return f.content
}

func (f *File) Write(s string) {
    f.content = s
}

var f File
var fp *File = &f

var rw1 ReadWriter = f   // ERROR: File does not satisfy ReadWriter (Write needs pointer)
var rw2 ReadWriter = fp  // OK: *File satisfies ReadWriter
```

---

## Common Pitfalls

### Pitfall 1: Pointer Receiver on Value

**Problem:**
```go
type Counter struct {
    value int
}

func (c *Counter) Increment() {
    c.value++
}

var c Counter
c.Increment()  // ERROR: cannot use c (type Counter) as *Counter
```

**Solution:**
```go
var c Counter
cp := &c
cp.Increment()  // OK

// Or
(&c).Increment()  // OK but awkward
```

### Pitfall 2: Interface Satisfaction

**Problem:**
```go
type Writer interface {
    Write(string)
}

type Buffer struct {
    data string
}

func (b *Buffer) Write(s string) {
    b.data = s
}

func process(w Writer) {
    w.Write("test")
}

var b Buffer
process(b)  // ERROR: Buffer does not satisfy Writer
```

**Solution:**
```go
var b Buffer
process(&b)  // OK: *Buffer satisfies Writer
```

### Pitfall 3: Method Set in Interface

**Problem:**
```go
type Interface interface {
    Method()
}

type Type struct{}

func (t Type) Method() {}

func (t *Type) AnotherMethod() {}

var i Interface = Type{}  // OK
i.AnotherMethod()          // ERROR: Interface does not have AnotherMethod
```

**Solution:**
```go
// Use type assertion or create new interface
type ExtendedInterface interface {
    Interface
    AnotherMethod()
}
```

---

## Best Practices

### 1. Use Pointer Receivers Consistently

**Why:**
- **Consistency**: Consistent method set
- **Flexibility**: Works with both value and pointer
- **Efficiency**: Avoid copying

**Guidelines:**
- **Large structs**: Use pointer receivers
- **Modification**: Use pointer receivers
- **Consistency**: Keep consistent within type

### 2. Use Value Receivers for Immutability

**Why:**
- **Immutability**: Immutable operations
- **Safety**: Cannot modify receiver
- **Simplicity**: Simpler semantics

**Guidelines:**
- **Small types**: Use value receivers
- **Immutable**: Use value receivers
- **Read-only**: Use value receivers

### 3. Understand Interface Satisfaction

**Why:**
- **Correctness**: Correct interface usage
- **Type safety**: Type safety
- **Flexibility**: Flexible design

**Guidelines:**
- **Check method sets**: Verify method sets
- **Pointer types**: Use pointer types when needed
- **Test**: Test interface satisfaction

### 4. Document Method Set Behavior

**Why:**
- **Clarity**: Clear method availability
- **Documentation**: Better documentation
- **Understanding**: Better understanding

**Guidelines:**
- **Document receivers**: Document receiver types
- **Explain availability**: Explain method availability
- **Examples**: Provide examples

---

## Summary

Method sets are fundamental to understanding Go's type system and interface satisfaction. Understanding method set rules, value vs pointer receivers, interface satisfaction, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Method sets**: Collection of methods available on type (type-specific, receiver-dependent, interface satisfaction, compile-time)
- **Method set rules**: Value receiver methods (available on T and *T, automatically dereferenced), pointer receiver methods (available only on *T, must use pointer)
- **Value receiver method sets**: Available on both value and pointer (copy, no modification, both types, automatically dereferenced)
- **Pointer receiver method sets**: Available only on pointer (reference, modification, pointer only, must use pointer)
- **Interface satisfaction**: Value type (satisfies if all methods have value receivers or pointer receivers with pointer), pointer type (satisfies if all methods have value receivers or pointer receivers)
- **Common pitfalls**: Pointer receiver on value (error, use pointer), interface satisfaction (check method sets, use pointer when needed), method set in interface (interface only has declared methods)
- **Best practices**: Use pointer receivers consistently, use value receivers for immutability, understand interface satisfaction, document method set behavior

**Method Set Rules:**
- **Value receiver**: Available on `T` and `*T`
- **Pointer receiver**: Available only on `*T`

**Best Practices:**
- Use pointer receivers consistently
- Use value receivers for immutability
- Understand interface satisfaction
- Document method set behavior

**Next Steps:**
- Learn method set rules
- Practice interface satisfaction
- Understand receiver types
- Apply best practices

