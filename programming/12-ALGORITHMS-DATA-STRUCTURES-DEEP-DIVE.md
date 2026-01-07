# Algorithms and Data Structures Deep Dive - Complete Understanding

## Table of Contents
1. [What are Algorithms and Data Structures?](#what-are-algorithms-and-data-structures)
2. [Why They Matter](#why-they-matter)
3. [Time Complexity](#time-complexity)
4. [Space Complexity](#space-complexity)
5. [Common Data Structures](#common-data-structures)
6. [Common Algorithms](#common-algorithms)
7. [Sorting Algorithms](#sorting-algorithms)
8. [Searching Algorithms](#searching-algorithms)
9. [Graph Algorithms](#graph-algorithms)
10. [Dynamic Programming](#dynamic-programming)
11. [Best Practices](#best-practices)

---

## What are Algorithms and Data Structures?

### Algorithm

**Algorithm**: Step-by-step procedure for solving a problem.

**Characteristics:**
- **Input**: Takes input
- **Output**: Produces output
- **Definite**: Definite steps
- **Terminates**: Always terminates

### Data Structure

**Data Structure**: Way of organizing and storing data.

**Characteristics:**
- **Organization**: Organizes data
- **Operations**: Supports operations
- **Efficiency**: Efficient access/modification

### Real-World Analogy

**Algorithm = Recipe:**
- **Recipe**: Algorithm
- **Ingredients**: Input
- **Steps**: Algorithm steps
- **Dish**: Output

**Data Structure = Container:**
- **Container**: Data structure
- **Items**: Data
- **Organization**: How organized
- **Access**: How to access

---

## Why They Matter?

### Performance Impact

**Example:**
```
Search in unsorted array: O(n)
Search in sorted array: O(log n)
Search in hash table: O(1)
  ↓
1000x difference for large n
```

### Scalability

**Example:**
```
O(n²) algorithm: 1 second for 1000 items
  ↓
10 seconds for 10,000 items
  ↓
100 seconds for 100,000 items
  ↓
Not scalable
```

---

## Time Complexity

### Big O Notation

**Big O**: Describes how algorithm performance scales with input size.

**Common Complexities:**
- **O(1)**: Constant time
- **O(log n)**: Logarithmic time
- **O(n)**: Linear time
- **O(n log n)**: Linearithmic time
- **O(n²)**: Quadratic time
- **O(2ⁿ)**: Exponential time

### Examples

**O(1) - Constant:**
```python
def get_first(arr):
    return arr[0]  # Always one operation
```

**O(log n) - Logarithmic:**
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

**O(n) - Linear:**
```python
def find_max(arr):
    max_val = arr[0]
    for item in arr:  # n operations
        if item > max_val:
            max_val = item
    return max_val
```

**O(n²) - Quadratic:**
```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n - i - 1):  # n² operations
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
```

---

## Space Complexity

### What is Space Complexity?

**Space Complexity**: Amount of memory algorithm uses relative to input size.

**Types:**
- **Auxiliary space**: Extra space (excluding input)
- **Total space**: Total space (including input)

### Examples

**O(1) - Constant Space:**
```python
def sum_array(arr):
    total = 0  # One variable
    for item in arr:
        total += item
    return total
```

**O(n) - Linear Space:**
```python
def copy_array(arr):
    result = []  # n elements
    for item in arr:
        result.append(item)
    return result
```

---

## Common Data Structures

### Array

**Characteristics:**
- **Indexed**: Access by index
- **Contiguous**: Contiguous memory
- **Fixed size**: Fixed size (static)

**Operations:**
- **Access**: O(1)
- **Search**: O(n)
- **Insert**: O(n)
- **Delete**: O(n)

### Linked List

**Characteristics:**
- **Nodes**: Nodes with pointers
- **Dynamic**: Dynamic size
- **Non-contiguous**: Non-contiguous memory

**Operations:**
- **Access**: O(n)
- **Search**: O(n)
- **Insert**: O(1) at head
- **Delete**: O(1) at head

### Stack

**Characteristics:**
- **LIFO**: Last In First Out
- **Operations**: Push, Pop, Peek

**Operations:**
- **Push**: O(1)
- **Pop**: O(1)
- **Peek**: O(1)

### Queue

**Characteristics:**
- **FIFO**: First In First Out
- **Operations**: Enqueue, Dequeue, Peek

**Operations:**
- **Enqueue**: O(1)
- **Dequeue**: O(1)
- **Peek**: O(1)

### Hash Table

**Characteristics:**
- **Key-value**: Key-value pairs
- **Hash function**: Hash function
- **Fast lookup**: Fast lookup

**Operations:**
- **Insert**: O(1) average
- **Search**: O(1) average
- **Delete**: O(1) average

### Binary Tree

**Characteristics:**
- **Nodes**: Nodes with children
- **Binary**: Max 2 children
- **Hierarchical**: Hierarchical structure

**Operations:**
- **Search**: O(log n) balanced
- **Insert**: O(log n) balanced
- **Delete**: O(log n) balanced

### Heap

**Characteristics:**
- **Complete tree**: Complete binary tree
- **Heap property**: Parent > children (max heap)
- **Priority**: Priority queue

**Operations:**
- **Insert**: O(log n)
- **Extract max**: O(log n)
- **Peek**: O(1)

---

## Common Algorithms

### Algorithm 1: Binary Search

**What:**
```
Search in sorted array
  ↓
Divide and conquer
  ↓
O(log n)
```

**Use Case:**
- **Sorted data**: Sorted arrays
- **Fast search**: Fast search needed

### Algorithm 2: Merge Sort

**What:**
```
Divide and conquer
  ↓
Merge sorted halves
  ↓
O(n log n)
```

**Use Case:**
- **Stable sort**: Stable sorting
- **Worst case**: Good worst case

### Algorithm 3: Quick Sort

**What:**
```
Divide and conquer
  ↓
Partition around pivot
  ↓
O(n log n) average
```

**Use Case:**
- **Fast average**: Fast average case
- **In-place**: In-place sorting

### Algorithm 4: Breadth-First Search (BFS)

**What:**
```
Graph traversal
  ↓
Level by level
  ↓
O(V + E)
```

**Use Case:**
- **Shortest path**: Shortest path (unweighted)
- **Level order**: Level-order traversal

### Algorithm 5: Depth-First Search (DFS)

**What:**
```
Graph traversal
  ↓
Go deep first
  ↓
O(V + E)
```

**Use Case:**
- **Path finding**: Path finding
- **Cycle detection**: Cycle detection

---

## Sorting Algorithms

### Comparison

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | No |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

### When to Use

**Small data:**
- **Insertion sort**: Insertion sort

**General purpose:**
- **Quick sort**: Quick sort (fast average)
- **Merge sort**: Merge sort (stable, predictable)

**External sorting:**
- **Merge sort**: Merge sort

---

## Searching Algorithms

### Linear Search

**What:**
```
Check each element
  ↓
O(n)
```

**Use Case:**
- **Unsorted**: Unsorted data
- **Small data**: Small datasets

### Binary Search

**What:**
```
Divide and conquer
  ↓
O(log n)
```

**Use Case:**
- **Sorted**: Sorted data
- **Fast**: Fast search needed

### Hash Table Search

**What:**
```
Hash function
  ↓
O(1) average
```

**Use Case:**
- **Fast lookup**: Fast lookup
- **Key-value**: Key-value data

---

## Graph Algorithms

### Shortest Path

**Algorithms:**
- **Dijkstra**: Single source, non-negative weights
- **Bellman-Ford**: Single source, negative weights
- **Floyd-Warshall**: All pairs

### Minimum Spanning Tree

**Algorithms:**
- **Kruskal**: Greedy, O(E log E)
- **Prim**: Greedy, O(E log V)

### Topological Sort

**What:**
```
Order nodes in DAG
  ↓
Dependencies resolved
  ↓
O(V + E)
```

---

## Dynamic Programming

### What is DP?

**Dynamic Programming**: Solve problems by breaking into subproblems and storing results.

**Characteristics:**
- **Overlapping subproblems**: Overlapping subproblems
- **Optimal substructure**: Optimal substructure
- **Memoization**: Store results

### DP Patterns

**1. Fibonacci:**
```
F(n) = F(n-1) + F(n-2)
  ↓
Memoize results
  ↓
O(n) time, O(n) space
```

**2. Longest Common Subsequence:**
```
Compare strings
  ↓
Build table
  ↓
O(mn) time, O(mn) space
```

---

## Best Practices

### 1. Choose Right Data Structure

**Why:**
- **Performance**: Better performance
- **Operations**: Match operations needed
- **Efficiency**: More efficient

**Guidelines:**
- **Fast lookup**: Hash table
- **Ordered**: Tree, heap
- **FIFO**: Queue
- **LIFO**: Stack

### 2. Understand Complexity

**Why:**
- **Scalability**: Understand scalability
- **Performance**: Predict performance
- **Optimization**: Optimize when needed

**Guidelines:**
- **Analyze**: Analyze complexity
- **Measure**: Measure performance
- **Optimize**: Optimize bottlenecks

### 3. Use Standard Libraries

**Why:**
- **Tested**: Well-tested
- **Optimized**: Optimized
- **Maintained**: Maintained

**Guidelines:**
- **Don't reinvent**: Don't reinvent wheel
- **Use libraries**: Use standard libraries
- **Custom when needed**: Custom only when needed

### 4. Profile and Measure

**Why:**
- **Actual performance**: Actual performance
- **Bottlenecks**: Identify bottlenecks
- **Optimization**: Guide optimization

**Tools:**
- **Profilers**: Profiling tools
- **Benchmarks**: Benchmarking
- **Monitoring**: Performance monitoring

---

## Summary

Algorithms and data structures are fundamental to programming. Understanding complexity, common structures, and algorithms is essential for backend engineers.

**Key Takeaways:**
- **Algorithms**: Step-by-step problem solving
- **Data structures**: Organizing and storing data
- **Time complexity**: How performance scales
- **Space complexity**: Memory usage
- **Common structures**: Array, list, stack, queue, hash, tree, heap
- **Common algorithms**: Search, sort, graph, DP
- **Best practices**: Right structure, understand complexity, use libraries, profile

**Data Structures:**
- **Array**: Indexed, O(1) access
- **Linked List**: Dynamic, O(1) insert
- **Hash Table**: O(1) average operations
- **Tree**: O(log n) operations
- **Heap**: Priority queue

**Algorithms:**
- **Search**: Linear O(n), Binary O(log n)
- **Sort**: Quick O(n log n), Merge O(n log n)
- **Graph**: BFS, DFS, Shortest path

**Best Practices:**
- Choose right data structure
- Understand complexity
- Use standard libraries
- Profile and measure

**Next Steps:**
- Learn common structures
- Practice algorithms
- Understand complexity
- Profile code
- Optimize bottlenecks

