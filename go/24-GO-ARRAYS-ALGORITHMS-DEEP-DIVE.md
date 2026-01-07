# Go Arrays Algorithms Deep Dive - Complete Understanding

## Table of Contents
1. [What are Array Algorithms?](#what-are-array-algorithms)
2. [Why Array Algorithms Matter](#why-array-algorithms-matter)
3. [Two Pointers Technique](#two-pointers-technique)
4. [Sliding Window Technique](#sliding-window-technique)
5. [Array Manipulation](#array-manipulation)
6. [Common Patterns](#common-patterns)
7. [Best Practices](#best-practices)

---

## What are Array Algorithms?

### Definition

**Array Algorithms**: Algorithms specifically designed for array/slice operations in Go.

**Key Characteristics:**
- **Array/Slice operations**: Work with Go slices
- **Efficient**: Efficient time and space complexity
- **Patterns**: Common patterns and techniques
- **Interview**: Common interview questions

### Real-World Analogy

**Array Algorithms = Tools:**
- **Array**: Raw material
- **Algorithm**: Tool
- **Problem**: Task to solve
- **Solution**: Efficient solution

**Programming:**
- **Slice**: Go slice
- **Algorithm**: Processing logic
- **Efficiency**: Time and space efficient

---

## Why Array Algorithms Matter?

### Use Cases

**1. Interview Preparation:**
```
Technical interviews
  ↓
Array problems
  ↓
Common questions
```

**2. Real-World Applications:**
```
Data processing
  ↓
Array operations
  ↓
Efficient algorithms
```

**3. Performance:**
```
Efficient solutions
  ↓
Better performance
  ↓
Scalable code
```

---

## Two Pointers Technique

### What is Two Pointers?

**Two Pointers**: Technique using two pointers to traverse array efficiently.

**Use Cases:**
- **Sorted arrays**: Working with sorted arrays
- **Pair finding**: Finding pairs
- **Reversing**: Reversing arrays
- **Partitioning**: Partitioning arrays

### Example: Two Sum

```go
func twoSum(nums []int, target int) []int {
    left, right := 0, len(nums)-1
    
    for left < right {
        sum := nums[left] + nums[right]
        if sum == target {
            return []int{left, right}
        } else if sum < target {
            left++
        } else {
            right--
        }
    }
    return []int{}
}
```

### Example: Remove Duplicates

```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    
    slow := 0
    for fast := 1; fast < len(nums); fast++ {
        if nums[fast] != nums[slow] {
            slow++
            nums[slow] = nums[fast]
        }
    }
    return slow + 1
}
```

---

## Sliding Window Technique

### What is Sliding Window?

**Sliding Window**: Technique using a window that slides through array.

**Use Cases:**
- **Subarray problems**: Finding subarrays
- **Maximum/minimum**: Finding max/min in window
- **Fixed size**: Fixed-size window problems
- **Variable size**: Variable-size window problems

### Example: Maximum Sum Subarray

```go
func maxSubArray(nums []int) int {
    maxSum, currentSum := nums[0], nums[0]
    
    for i := 1; i < len(nums); i++ {
        currentSum = max(nums[i], currentSum+nums[i])
        maxSum = max(maxSum, currentSum)
    }
    
    return maxSum
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### Example: Longest Substring Without Repeating Characters

```go
func lengthOfLongestSubstring(s string) int {
    charMap := make(map[byte]int)
    maxLen, left := 0, 0
    
    for right := 0; right < len(s); right++ {
        if idx, exists := charMap[s[right]]; exists && idx >= left {
            left = idx + 1
        }
        charMap[s[right]] = right
        maxLen = max(maxLen, right-left+1)
    }
    
    return maxLen
}
```

---

## Array Manipulation

### Reversing Array

```go
func reverse(nums []int) {
    left, right := 0, len(nums)-1
    for left < right {
        nums[left], nums[right] = nums[right], nums[left]
        left++
        right--
    }
}
```

### Rotating Array

```go
func rotate(nums []int, k int) {
    n := len(nums)
    k = k % n
    reverse(nums)
    reverse(nums[:k])
    reverse(nums[k:])
}
```

### Finding Maximum/Minimum

```go
func findMax(nums []int) int {
    max := nums[0]
    for i := 1; i < len(nums); i++ {
        if nums[i] > max {
            max = nums[i]
        }
    }
    return max
}
```

---

## Common Patterns

### Pattern 1: Kadane's Algorithm

```go
func maxSubArray(nums []int) int {
    maxSoFar, maxEndingHere := nums[0], nums[0]
    
    for i := 1; i < len(nums); i++ {
        maxEndingHere = max(nums[i], maxEndingHere+nums[i])
        maxSoFar = max(maxSoFar, maxEndingHere)
    }
    
    return maxSoFar
}
```

### Pattern 2: Dutch National Flag

```go
func sortColors(nums []int) {
    left, curr, right := 0, 0, len(nums)-1
    
    for curr <= right {
        if nums[curr] == 0 {
            nums[left], nums[curr] = nums[curr], nums[left]
            left++
            curr++
        } else if nums[curr] == 2 {
            nums[curr], nums[right] = nums[right], nums[curr]
            right--
        } else {
            curr++
        }
    }
}
```

### Pattern 3: Product of Array Except Self

```go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    result := make([]int, n)
    
    // Left products
    result[0] = 1
    for i := 1; i < n; i++ {
        result[i] = result[i-1] * nums[i-1]
    }
    
    // Right products
    right := 1
    for i := n - 1; i >= 0; i-- {
        result[i] *= right
        right *= nums[i]
    }
    
    return result
}
```

---

## Best Practices

### 1. Use Two Pointers for Sorted Arrays

**Why:**
- **Efficiency**: O(n) time complexity
- **Space**: O(1) space complexity
- **Clarity**: Clear and readable

**Guidelines:**
- **Sorted arrays**: Use for sorted arrays
- **Pair finding**: Finding pairs efficiently
- **Reversing**: Reversing operations

### 2. Use Sliding Window for Subarray Problems

**Why:**
- **Efficiency**: Efficient subarray processing
- **Pattern**: Common pattern
- **Optimization**: Optimizes nested loops

**Guidelines:**
- **Subarray problems**: Use for subarray problems
- **Fixed/variable size**: Both fixed and variable size
- **Optimization**: Optimize nested loops

### 3. Handle Edge Cases

**Why:**
- **Correctness**: Correct behavior
- **Robustness**: Robust code
- **Interviews**: Important in interviews

**Guidelines:**
- **Empty arrays**: Handle empty arrays
- **Single element**: Handle single element
- **Boundary conditions**: Check boundaries

---

## Summary

Array algorithms are essential for efficient array/slice operations in Go. Understanding two pointers, sliding window, array manipulation, common patterns, and best practices is crucial for solving array problems effectively.

**Key Takeaways:**
- **Array algorithms**: Algorithms for array/slice operations (efficient, common patterns, interview questions)
- **Two pointers technique**: Using two pointers to traverse array (sorted arrays, pair finding, O(n) time)
- **Sliding window technique**: Window that slides through array (subarray problems, max/min, optimization)
- **Array manipulation**: Common operations (reversing, rotating, finding max/min)
- **Common patterns**: Kadane's algorithm, Dutch National Flag, product except self
- **Best practices**: Use two pointers for sorted arrays, use sliding window for subarrays, handle edge cases

**Array Algorithm Techniques:**
- **Two pointers**: O(n) time, O(1) space
- **Sliding window**: Efficient subarray processing
- **Patterns**: Common problem patterns

**Best Practices:**
- Use two pointers for sorted arrays
- Use sliding window for subarray problems
- Handle edge cases

**Next Steps:**
- Practice two pointers
- Learn sliding window
- Master common patterns
- Solve array problems

