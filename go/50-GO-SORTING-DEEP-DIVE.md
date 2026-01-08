# Go Sorting Deep Dive - Complete Understanding

## Table of Contents
1. [What is Sorting?](#what-is-sorting)
2. [Why Sorting Matters](#why-sorting-matters)
3. [Sort Package](#sort-package)
4. [Sorting Primitives](#sorting-primitives)
5. [Sorting Custom Types](#sorting-custom-types)
6. [Best Practices](#best-practices)

---

## What is Sorting?

### Definition

**Sorting**: Arranging elements in a specific order (ascending or descending).

**Key Characteristics:**
- **Order**: Arranges in order
- **Comparison**: Based on comparison
- **In-place**: Usually modifies original
- **Efficient**: Efficient algorithms

### Real-World Analogy

**Sorting = Organizing:**
- **Items**: Elements to sort
- **Order**: Specific order
- **Organization**: Organized collection
- **Efficiency**: Efficient organization

**Programming:**
- **Elements**: Data elements
- **Sorting**: Arranging in order
- **Algorithm**: Sorting algorithm
- **Performance**: Performance matters

---

## Why Sorting Matters?

### Benefits

**1. Organization:**
```
Unordered data
  ↓
Sorting
  ↓
Ordered data
```

**2. Search Efficiency:**
```
Sorted data
  ↓
Binary search
  ↓
Fast search
```

**3. Data Presentation:**
```
Raw data
  ↓
Sorting
  ↓
Presentable data
```

---

## Sort Package

### Import

```go
import "sort"
```

### Available Functions

**1. Sort primitives:**
- `sort.Ints(slice []int)`
- `sort.Float64s(slice []float64)`
- `sort.Strings(slice []string)`

**2. Generic sort:**
- `sort.Sort(data Interface)`
- `sort.Slice(slice interface{}, less func(i, j int) bool)`
- `sort.SliceStable(slice interface{}, less func(i, j int) bool)`

**3. Search:**
- `sort.Search(n int, f func(int) bool) int`
- `sort.SearchInts(a []int, x int) int`
- `sort.SearchStrings(a []string, x string) int`

---

## Sorting Primitives

### Sorting Integers

```go
import "sort"

numbers := []int{3, 1, 4, 1, 5, 9, 2, 6}
sort.Ints(numbers)
// numbers is now [1, 1, 2, 3, 4, 5, 6, 9]
```

**Characteristics:**
- **In-place**: Modifies original slice
- **Ascending**: Sorts in ascending order
- **Efficient**: Efficient sorting

### Sorting Floats

```go
numbers := []float64{3.14, 2.71, 1.41, 1.73}
sort.Float64s(numbers)
// numbers is now [1.41, 1.73, 2.71, 3.14]
```

### Sorting Strings

```go
words := []string{"banana", "apple", "cherry"}
sort.Strings(words)
// words is now ["apple", "banana", "cherry"]
```

---

## Sorting Custom Types

### Method 1: Implement sort.Interface

**sort.Interface requires:**
```go
type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

**Example:**
```go
type Person struct {
    Name string
    Age  int
}

type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

func main() {
    people := []Person{
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35},
    }
    sort.Sort(ByAge(people))
    // Sorted by age
}
```

### Method 2: Use sort.Slice

**Simpler approach:**
```go
type Person struct {
    Name string
    Age  int
}

people := []Person{
    {"Alice", 30},
    {"Bob", 25},
    {"Charlie", 35},
}

// Sort by age
sort.Slice(people, func(i, j int) bool {
    return people[i].Age < people[j].Age
})

// Sort by name
sort.Slice(people, func(i, j int) bool {
    return people[i].Name < people[j].Name
})
```

### Method 3: Sort by Multiple Fields

**Sort by multiple criteria:**
```go
sort.Slice(people, func(i, j int) bool {
    if people[i].Age != people[j].Age {
        return people[i].Age < people[j].Age
    }
    return people[i].Name < people[j].Name
})
```

---

## Best Practices

### 1. Use sort.Slice for Simplicity

**Why:**
- **Simplicity**: Simpler code
- **Flexibility**: More flexible
- **Readability**: More readable

**Guidelines:**
- **sort.Slice**: Use sort.Slice when possible
- **sort.Interface**: Use when need stable sort
- **Choose**: Choose based on needs

### 2. Use sort.SliceStable for Stability

**Why:**
- **Stability**: Maintains relative order
- **Predictability**: More predictable
- **Correctness**: Correct for equal elements

**Guidelines:**
- **Equal elements**: Use when equal elements matter
- **Stability**: Use when stability needed
- **Performance**: Slightly slower but stable

### 3. Pre-allocate When Possible

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Memory**: Better memory usage

**Guidelines:**
- **Known size**: Pre-allocate when size known
- **Capacity**: Set appropriate capacity
- **Optimize**: Optimize when needed

### 4. Consider Performance for Large Data

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Scalability**: Better scalability

**Guidelines:**
- **Large data**: Consider performance for large data
- **Algorithm**: Choose appropriate algorithm
- **Measure**: Measure performance

---

## Summary

Sorting is essential for organizing data in Go. Understanding sort package, sorting primitives, custom types, and best practices is crucial for effective Go programming.

**Key Takeaways:**
- **Sorting**: Arranging elements in specific order (order, comparison, in-place, efficient)
- **Sort package**: Available functions (sort primitives: Ints Float64s Strings, generic sort: Sort Slice SliceStable, search: Search SearchInts SearchStrings)
- **Sorting primitives**: Sorting integers (in-place ascending efficient), sorting floats, sorting strings
- **Sorting custom types**: Implement sort.Interface (Len Less Swap methods), use sort.Slice (simpler approach, flexible), sort by multiple fields (multiple criteria)
- **Best practices**: Use sort.Slice for simplicity, use sort.SliceStable for stability, pre-allocate when possible, consider performance for large data

**Sorting Benefits:**
- **Organization**: Organized data
- **Search efficiency**: Fast search
- **Data presentation**: Presentable data

**Best Practices:**
- Use sort.Slice for simplicity
- Use sort.SliceStable for stability
- Pre-allocate when possible
- Consider performance for large data

**Next Steps:**
- Learn sort package
- Practice sorting primitives
- Master custom type sorting
- Apply best practices

