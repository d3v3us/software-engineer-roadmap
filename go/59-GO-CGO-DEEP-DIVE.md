# Go CGO Deep Dive - Complete Understanding

## Table of Contents
1. [What is CGO?](#what-is-cgo)
2. [Why Use CGO?](#why-use-cgo)
3. [CGO Basics](#cgo-basics)
4. [CGO Overhead](#cgo-overhead)
5. [Data Transfer Between Go and C](#data-transfer-between-go-and-c)
6. [Memory Management in CGO](#memory-management-in-cgo)
7. [Common Patterns](#common-patterns)
8. [Best Practices](#best-practices)

---

## What is CGO?

### Definition

**CGO**: Mechanism that allows Go programs to call C code and vice versa.

**Key Characteristics:**
- **C interop**: Interoperability with C
- **FFI**: Foreign Function Interface
- **Bridge**: Bridge between Go and C
- **Powerful**: Powerful but complex

### Real-World Analogy

**CGO = Translator:**
- **Go code**: One language
- **C code**: Another language
- **CGO**: Translator
- **Communication**: Enables communication

**Programming:**
- **Go**: Go code
- **C**: C code
- **CGO**: Bridge
- **Integration**: Integration

---

## Why Use CGO?

### Reasons

**1. Legacy Code:**
```
Existing C libraries
  ↓
CGO
  ↓
Use in Go
```

**2. Performance:**
```
Critical performance
  ↓
C code
  ↓
CGO integration
```

**3. System Calls:**
```
System-level operations
  ↓
C code
  ↓
CGO access
```

**4. Hardware Access:**
```
Hardware interfaces
  ↓
C drivers
  ↓
CGO integration
```

---

## CGO Basics

### Basic Example

**C code:**
```go
/*
#include <stdio.h>
void hello() {
    printf("Hello from C\n");
}
*/
import "C"

func main() {
    C.hello()
}
```

### Import C

**Import C package:**
```go
import "C"
```

**Note**: Must be separate import, comment before import.

### Calling C Functions

**Call C function:**
```go
/*
#include <stdlib.h>
*/
import "C"

func main() {
    C.malloc(100)
}
```

### C Types in Go

**C types:**
```go
var x C.int
var y C.float
var s *C.char
```

---

## CGO Overhead

### Overhead Sources

**1. Function Call Overhead:**
```
Go → C call
  ↓
Context switch
  ↓
C execution
  ↓
Context switch back
  ↓
Overhead
```

**2. Data Conversion:**
```
Go types
  ↓
Convert to C types
  ↓
C types
  ↓
Overhead
```

**3. Memory Management:**
```
Go memory
  ↓
C memory
  ↓
Conversion overhead
```

### Measuring Overhead

**Benchmark:**
```go
func BenchmarkCGO(b *testing.B) {
    for i := 0; i < b.N; i++ {
        C.someFunction()
    }
}

func BenchmarkGo(b *testing.B) {
    for i := 0; i < b.N; i++ {
        goFunction()
    }
}
```

**Typical overhead**: 10-100x slower than pure Go.

---

## Data Transfer Between Go and C

### Passing Go Values to C

**Example:**
```go
/*
#include <string.h>
void process(char* str) {
    // Process string
}
*/
import "C"
import "unsafe"

func main() {
    goStr := "Hello"
    cStr := C.CString(goStr)
    defer C.free(unsafe.Pointer(cStr))
    
    C.process(cStr)
}
```

### Getting C Values in Go

**Example:**
```go
/*
char* getString() {
    return "Hello from C";
}
*/
import "C"

func main() {
    cStr := C.getString()
    goStr := C.GoString(cStr)
    fmt.Println(goStr)
}
```

### Type Conversions

**Common conversions:**
```go
// String
cStr := C.CString(goStr)
goStr := C.GoString(cStr)

// Bytes
cBytes := C.CBytes(goBytes)
goBytes := C.GoBytes(cBytes, C.int(len))

// Numbers
cInt := C.int(goInt)
goInt := int(cInt)
```

---

## Memory Management in CGO

### C Memory Allocation

**Allocate in C:**
```go
/*
#include <stdlib.h>
*/
import "C"

ptr := C.malloc(100)
defer C.free(ptr)
```

### Go Memory in C

**Pass Go memory to C:**
```go
goSlice := []byte{1, 2, 3}
cPtr := (*C.uchar)(unsafe.Pointer(&goSlice[0]))
```

**Warning**: Go GC may move memory!

### Memory Safety

**Best practices:**
- **Allocate in C**: Use C.malloc for C-allocated memory
- **Free in C**: Use C.free for C-allocated memory
- **Go memory**: Keep Go memory referenced
- **Lifetime**: Understand lifetime

---

## Common Patterns

### Pattern 1: Wrapper Functions

**Wrap C functions:**
```go
/*
int add(int a, int b) {
    return a + b;
}
*/
import "C"

func Add(a, b int) int {
    return int(C.add(C.int(a), C.int(b)))
}
```

### Pattern 2: Struct Conversion

**Convert structs:**
```go
/*
typedef struct {
    int x;
    int y;
} Point;
*/
import "C"

type Point struct {
    X int
    Y int
}

func toCPoint(p Point) C.Point {
    return C.Point{
        x: C.int(p.X),
        y: C.int(p.Y),
    }
}
```

### Pattern 3: Callbacks

**C callbacks:**
```go
/*
typedef void (*Callback)(int);
void callCallback(Callback cb, int value) {
    cb(value);
}
*/
import "C"

//export goCallback
func goCallback(value C.int) {
    fmt.Println("Callback:", value)
}

func main() {
    C.callCallback(C.Callback(C.goCallback), 42)
}
```

---

## Best Practices

### 1. Minimize CGO Usage

**Why:**
- **Overhead**: CGO has overhead
- **Complexity**: Adds complexity
- **Portability**: Reduces portability

**Guidelines:**
- **Avoid**: Avoid when possible
- **Alternatives**: Consider alternatives
- **Minimize**: Minimize CGO calls

### 2. Batch CGO Calls

**Why:**
- **Overhead**: Reduce overhead
- **Performance**: Better performance
- **Efficiency**: More efficient

**Guidelines:**
- **Batch**: Batch multiple operations
- **Reduce calls**: Reduce number of calls
- **Optimize**: Optimize call patterns

### 3. Manage Memory Carefully

**Why:**
- **Leaks**: Prevent memory leaks
- **Safety**: Ensure safety
- **Correctness**: Correct behavior

**Guidelines:**
- **Free**: Always free C memory
- **Defer**: Use defer for cleanup
- **Track**: Track memory lifetime

### 4. Handle Errors Properly

**Why:**
- **Robustness**: More robust
- **Debugging**: Easier debugging
- **Reliability**: More reliable

**Guidelines:**
- **Check errors**: Check C return values
- **Handle failures**: Handle failures
- **Error conversion**: Convert C errors to Go errors

### 5. Test Thoroughly

**Why:**
- **Correctness**: Ensure correctness
- **Safety**: Ensure safety
- **Reliability**: More reliable

**Guidelines:**
- **Tests**: Comprehensive tests
- **Edge cases**: Test edge cases
- **Platforms**: Test on different platforms

---

## Summary

CGO enables Go programs to call C code and vice versa. Understanding CGO basics, overhead, data transfer, memory management, common patterns, and best practices is crucial for effective C interoperation.

**Key Takeaways:**
- **CGO**: Mechanism for Go-C interop (C interop, FFI, bridge, powerful)
- **CGO basics**: Import C, call C functions, C types in Go
- **CGO overhead**: Function call overhead (context switch), data conversion (type conversion), memory management (conversion overhead), typical overhead (10-100x slower)
- **Data transfer**: Passing Go values to C (C.CString, C.CBytes), getting C values in Go (C.GoString, C.GoBytes), type conversions
- **Memory management**: C memory allocation (C.malloc, C.free), Go memory in C (unsafe.Pointer, GC issues), memory safety (allocate in C, free in C, keep Go memory referenced)
- **Common patterns**: Wrapper functions, struct conversion, callbacks
- **Best practices**: Minimize CGO usage, batch CGO calls, manage memory carefully, handle errors properly, test thoroughly

**CGO Characteristics:**
- **Powerful**: Very powerful
- **Overhead**: Significant overhead
- **Complex**: Complex to use

**Best Practices:**
- Minimize CGO usage
- Batch CGO calls
- Manage memory carefully
- Handle errors properly
- Test thoroughly

**Next Steps:**
- Learn CGO basics
- Understand overhead
- Practice memory management
- Apply best practices

