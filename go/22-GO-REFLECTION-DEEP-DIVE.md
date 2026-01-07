# Go Reflection Deep Dive - Complete Understanding

## Table of Contents
1. [What is Reflection?](#what-is-reflection)
2. [Why Use Reflection?](#why-use-reflection)
3. [Reflection Basics](#reflection-basics)
4. [Reflection Operations](#reflection-operations)
5. [Reflection Use Cases](#reflection-use-cases)
6. [Best Practices](#best-practices)

---

## What is Reflection?

### Definition

**Reflection**: Ability of program to examine and modify its own structure at runtime.

**Key Characteristics:**
- **Runtime**: Runtime type information
- **Inspection**: Inspect types and values
- **Modification**: Modify values
- **Meta-programming**: Meta-programming capability

### Real-World Analogy

**Reflection = Mirror:**
- **Object**: Code structure
- **Mirror**: Reflection
- **Inspection**: See structure
- **Modification**: Modify if needed

**Programming:**
- **Types**: Type information
- **Values**: Value information
- **Inspection**: Runtime inspection
- **Modification**: Runtime modification

---

## Why Use Reflection?

### Use Cases

**1. Generic Code:**
```
Before generics
  ↓
Reflection for generics
  ↓
Flexible code
```

**2. Serialization:**
```
JSON/XML encoding
  ↓
Reflection to inspect
  ↓
Automatic encoding
```

**3. Frameworks:**
```
ORM frameworks
  ↓
Reflection to map
  ↓
Database mapping
```

---

## Reflection Basics

### reflect Package

```go
import "reflect"

// Get type
t := reflect.TypeOf(value)

// Get value
v := reflect.ValueOf(value)
```

### Type Information

```go
type Person struct {
    Name string
    Age  int
}

t := reflect.TypeOf(Person{})
fmt.Println(t.Name())        // "Person"
fmt.Println(t.Kind())         // reflect.Struct
fmt.Println(t.NumField())     // 2
```

### Value Information

```go
p := Person{Name: "Alice", Age: 30}
v := reflect.ValueOf(p)

fmt.Println(v.Field(0))  // "Alice"
fmt.Println(v.Field(1))  // 30
```

---

## Reflection Operations

### Inspecting Structs

```go
func inspectStruct(s interface{}) {
    v := reflect.ValueOf(s)
    t := reflect.TypeOf(s)
    
    for i := 0; i < v.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("%s: %v\n", field.Name, value.Interface())
    }
}
```

### Modifying Values

```go
p := &Person{Name: "Alice", Age: 30}
v := reflect.ValueOf(p).Elem()

// Modify field
nameField := v.FieldByName("Name")
if nameField.IsValid() && nameField.CanSet() {
    nameField.SetString("Bob")
}
```

### Calling Methods

```go
v := reflect.ValueOf(obj)
method := v.MethodByName("MethodName")
if method.IsValid() {
    result := method.Call([]reflect.Value{})
}
```

---

## Reflection Use Cases

### Use Case 1: JSON Encoding

```go
// Reflection used internally by json.Marshal
func marshal(v interface{}) []byte {
    // Uses reflection to inspect struct
    // Reads struct tags
    // Encodes to JSON
}
```

### Use Case 2: ORM Mapping

```go
// Reflection to map struct to database
func mapToDB(s interface{}) map[string]interface{} {
    // Uses reflection to inspect struct
    // Maps fields to database columns
}
```

### Use Case 3: Validation

```go
// Reflection to validate struct
func validate(s interface{}) error {
    // Uses reflection to inspect struct
    // Validates fields based on tags
}
```

---

## Best Practices

### 1. Use Reflection Sparingly

**Why:**
- **Performance**: Reflection is slower
- **Complexity**: More complex code
- **Type safety**: Loses type safety

**Guidelines:**
- **Last resort**: Use as last resort
- **Alternatives**: Consider alternatives
- **Generics**: Use generics when possible (Go 1.18+)

### 2. Cache Reflection Results

**Why:**
- **Performance**: Reflection is expensive
- **Efficiency**: Cache improves efficiency
- **Reuse**: Reuse reflection results

**Guidelines:**
- **Cache types**: Cache type information
- **Reuse**: Reuse reflection results
- **Performance**: Consider performance impact

### 3. Handle Errors

**Why:**
- **Runtime errors**: Reflection can panic
- **Safety**: Safe reflection usage
- **Robustness**: Robust code

**Guidelines:**
- **Check validity**: Check IsValid()
- **Check settability**: Check CanSet()
- **Handle panics**: Handle potential panics

---

## Summary

Reflection provides runtime type inspection in Go. Understanding reflection basics, operations, use cases, and best practices is crucial for advanced Go programming.

**Key Takeaways:**
- **Reflection**: Runtime type inspection and modification (reflect package)
- **Reflection basics**: reflect.TypeOf (type information), reflect.ValueOf (value information)
- **Reflection operations**: Inspecting structs, modifying values, calling methods
- **Reflection use cases**: Generic code (before generics), serialization (JSON/XML), frameworks (ORM)
- **Best practices**: Use sparingly, cache results, handle errors

**Reflection Characteristics:**
- **Runtime**: Runtime type information
- **Inspection**: Inspect types and values
- **Modification**: Modify values

**Best Practices:**
- Use reflection sparingly
- Cache reflection results
- Handle errors

**Next Steps:**
- Learn reflection basics
- Practice reflection operations
- Understand use cases
- Apply best practices

