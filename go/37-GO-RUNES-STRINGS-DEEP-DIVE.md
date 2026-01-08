# Go Runes and Strings Deep Dive - Complete Understanding

## Table of Contents
1. [What are Runes and Strings?](#what-are-runes-and-strings)
2. [Why Understanding Runes Matters](#why-understanding-runes-matters)
3. [String Internals](#string-internals)
4. [Rune Internals](#rune-internals)
5. [Runes vs Strings](#runes-vs-strings)
6. [String and Rune Operations](#string-and-rune-operations)
7. [Best Practices](#best-practices)

---

## What are Runes and Strings?

### Definition

**String**: Immutable sequence of bytes representing text.

**Rune**: Integer type (int32) representing a Unicode code point.

**Key Characteristics:**
- **String**: UTF-8 encoded bytes
- **Rune**: Unicode code point
- **Immutable**: Strings are immutable
- **Unicode**: Full Unicode support

### Real-World Analogy

**String = Text:**
- **Text**: Human-readable text
- **Bytes**: Underlying bytes
- **Encoding**: UTF-8 encoding
- **Display**: Displayed text

**Rune = Character:**
- **Character**: Single character
- **Code point**: Unicode code point
- **Integer**: Integer representation
- **Unicode**: Unicode character

---

## Why Understanding Runes Matters?

### Unicode Complexity

**1. Multi-byte Characters:**
```
ASCII: 1 byte per character
UTF-8: 1-4 bytes per character
  ↓
String length ≠ character count
```

**2. String Indexing:**
```
String indexing: Byte position
Character access: Need runes
  ↓
Different operations
```

**3. String Operations:**
```
Byte operations: On bytes
Character operations: On runes
  ↓
Correct operations
```

---

## String Internals

### String Structure

**String in Go:**
```go
type string struct {
    ptr *byte    // Pointer to byte array
    len int      // Length in bytes
}
```

**Characteristics:**
- **Immutable**: Cannot be modified
- **UTF-8**: UTF-8 encoded
- **Byte sequence**: Sequence of bytes
- **Length**: Length in bytes

### String Representation

```go
s := "Hello, 世界"

// String is UTF-8 encoded bytes
// "Hello, " = 7 bytes (ASCII)
// "世界" = 6 bytes (2 UTF-8 characters, 3 bytes each)
// Total: 13 bytes
```

---

## Rune Internals

### Rune Type

**Rune is int32:**
```go
type rune = int32

// Rune represents Unicode code point
var r rune = 'A'        // 65
var r2 rune = '世'      // 19990 (Unicode code point)
```

### Rune Characteristics

- **Unicode code point**: Represents Unicode code point
- **Integer**: Integer type (int32)
- **Character**: Single character
- **32-bit**: 32-bit integer

---

## Runes vs Strings

### Key Differences

| Aspect | String | Rune |
|--------|--------|------|
| Type | string | int32 |
| Size | Variable (bytes) | Fixed (4 bytes) |
| Content | UTF-8 bytes | Unicode code point |
| Immutable | Yes | Yes (value) |
| Indexing | Byte position | Character value |

### String Indexing

```go
s := "Hello, 世界"

// Byte indexing
fmt.Println(s[0])        // 72 (byte 'H')
fmt.Println(s[7])        // 228 (first byte of '世')

// Character access (incorrect)
// s[7] is NOT '世', it's a byte
```

### Rune Conversion

```go
s := "Hello, 世界"

// Convert string to []rune
runes := []rune(s)
fmt.Println(runes[7])    // 19990 (rune '世')

// Convert []rune to string
s2 := string(runes)
```

---

## String and Rune Operations

### Iterating Over Strings

**Byte iteration:**
```go
s := "Hello, 世界"
for i := 0; i < len(s); i++ {
    fmt.Printf("%d: %c (%d)\n", i, s[i], s[i])
}
// Output: Bytes, not characters
```

**Rune iteration:**
```go
s := "Hello, 世界"
for i, r := range s {
    fmt.Printf("%d: %c (%d)\n", i, r, r)
}
// Output: Characters (runes)
```

### String Length

**Byte length vs character count:**
```go
s := "Hello, 世界"

// Byte length
byteLen := len(s)  // 13 bytes

// Character count
runeCount := len([]rune(s))  // 9 characters
runeCount2 := utf8.RuneCountInString(s)  // 9 characters
```

### String Manipulation

**Substring (by bytes):**
```go
s := "Hello, 世界"
sub := s[0:5]  // "Hello" (byte-based)
```

**Substring (by runes):**
```go
s := "Hello, 世界"
runes := []rune(s)
sub := string(runes[0:5])  // "Hello" (rune-based)
```

### Rune Operations

**Rune to string:**
```go
r := '世'
s := string(r)  // "世"
```

**String to runes:**
```go
s := "世界"
runes := []rune(s)  // [19990, 30028]
```

**Rune comparison:**
```go
r1 := 'A'
r2 := 'B'
if r1 < r2 {
    // Comparison works
}
```

---

## Best Practices

### 1. Use range for Character Iteration

**Why:**
- **Correct**: Correct character iteration
- **Unicode**: Proper Unicode handling
- **Simple**: Simple and clear

**Guidelines:**
- **range**: Use range for character iteration
- **Not byte indexing**: Don't use byte indexing for characters
- **Runes**: Use runes for character operations

### 2. Understand Byte vs Character

**Why:**
- **Correctness**: Correct string operations
- **Unicode**: Proper Unicode handling
- **Clarity**: Clear understanding

**Guidelines:**
- **Byte length**: Use len() for byte length
- **Character count**: Use utf8.RuneCountInString() for character count
- **Indexing**: Understand byte vs character indexing

### 3. Use utf8 Package for Unicode Operations

**Why:**
- **Correctness**: Correct Unicode operations
- **Standard library**: Standard library support
- **Reliability**: Reliable operations

**Guidelines:**
- **utf8 package**: Use utf8 package for Unicode operations
- **RuneCountInString**: Use for character count
- **ValidString**: Use for validation

### 4. Be Careful with String Slicing

**Why:**
- **Byte-based**: String slicing is byte-based
- **Character boundaries**: May break character boundaries
- **Corruption**: May corrupt UTF-8 sequences

**Guidelines:**
- **Character boundaries**: Ensure character boundaries
- **Convert to runes**: Convert to []rune for character-based slicing
- **Validate**: Validate UTF-8 sequences

---

## Summary

Understanding runes and strings is essential for proper text handling in Go. Understanding string internals, rune internals, differences, operations, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Strings**: Immutable sequence of UTF-8 bytes (immutable, UTF-8 encoded, byte sequence, length in bytes)
- **Runes**: Integer type (int32) representing Unicode code point (Unicode code point, integer, character, 32-bit)
- **Runes vs strings**: String (variable size, UTF-8 bytes, byte indexing) vs Rune (fixed 4 bytes, Unicode code point, character value)
- **String and rune operations**: Iterating (byte iteration, rune iteration with range), length (byte length vs character count), manipulation (byte-based vs rune-based)
- **Best practices**: Use range for character iteration, understand byte vs character, use utf8 package, be careful with string slicing

**Key Differences:**
- **String**: UTF-8 encoded bytes, variable size, byte indexing
- **Rune**: Unicode code point, fixed 4 bytes, character value

**Best Practices:**
- Use range for character iteration
- Understand byte vs character
- Use utf8 package for Unicode operations
- Be careful with string slicing

**Next Steps:**
- Practice rune operations
- Learn Unicode handling
- Master string operations
- Apply best practices

