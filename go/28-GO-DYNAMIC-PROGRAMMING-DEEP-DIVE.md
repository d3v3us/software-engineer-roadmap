# Go Dynamic Programming Deep Dive - Complete Understanding

## Table of Contents
1. [What is Dynamic Programming?](#what-is-dynamic-programming)
2. [Why Use Dynamic Programming?](#why-use-dynamic-programming)
3. [DP Patterns](#dp-patterns)
4. [Common DP Problems](#common-dp-problems)
5. [Optimization Techniques](#optimization-techniques)
6. [Best Practices](#best-practices)

---

## What is Dynamic Programming?

### Definition

**Dynamic Programming**: Solving problems by breaking them into subproblems and storing results.

**Key Characteristics:**
- **Subproblems**: Break into smaller subproblems
- **Memoization**: Store results of subproblems
- **Optimal substructure**: Optimal solution contains optimal subproblems
- **Overlapping subproblems**: Subproblems repeat

### Real-World Analogy

**DP = Building Blocks:**
- **Problem**: Large building
- **Subproblems**: Building blocks
- **Memoization**: Reuse blocks
- **Solution**: Complete building

**Programming:**
- **Problem**: Complex problem
- **Subproblems**: Smaller problems
- **Cache**: Store solutions
- **Efficiency**: Avoid recomputation

---

## Why Use Dynamic Programming?

### Benefits

**1. Efficiency:**
```
Exponential time
  ↓
DP optimization
  ↓
Polynomial time
```

**2. Optimal Solutions:**
```
Greedy may fail
  ↓
DP guarantees optimal
  ↓
Correct solution
```

**3. Reusability:**
```
Reuse subproblems
  ↓
Avoid recomputation
  ↓
Efficient
```

---

## DP Patterns

### Pattern 1: Fibonacci (1D DP)

```go
func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    
    dp := make([]int, n+1)
    dp[0] = 0
    dp[1] = 1
    
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}

// Space-optimized
func fibonacciOptimized(n int) int {
    if n <= 1 {
        return n
    }
    
    prev2, prev1 := 0, 1
    for i := 2; i <= n; i++ {
        curr := prev1 + prev2
        prev2, prev1 = prev1, curr
    }
    
    return prev1
}
```

### Pattern 2: Climbing Stairs

```go
func climbStairs(n int) int {
    if n <= 2 {
        return n
    }
    
    dp := make([]int, n+1)
    dp[1] = 1
    dp[2] = 2
    
    for i := 3; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}
```

### Pattern 3: Coin Change

```go
func coinChange(coins []int, amount int) int {
    dp := make([]int, amount+1)
    for i := range dp {
        dp[i] = amount + 1
    }
    dp[0] = 0
    
    for i := 1; i <= amount; i++ {
        for _, coin := range coins {
            if coin <= i {
                dp[i] = min(dp[i], dp[i-coin]+1)
            }
        }
    }
    
    if dp[amount] > amount {
        return -1
    }
    return dp[amount]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

---

## Common DP Problems

### Problem 1: Longest Common Subsequence

```go
func longestCommonSubsequence(text1, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    
    return dp[m][n]
}
```

### Problem 2: Maximum Subarray (Kadane's)

```go
func maxSubArray(nums []int) int {
    maxSoFar, maxEndingHere := nums[0], nums[0]
    
    for i := 1; i < len(nums); i++ {
        maxEndingHere = max(nums[i], maxEndingHere+nums[i])
        maxSoFar = max(maxSoFar, maxEndingHere)
    }
    
    return maxSoFar
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### Problem 3: Longest Increasing Subsequence

```go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    dp := make([]int, n)
    for i := range dp {
        dp[i] = 1
    }
    
    maxLen := 1
    for i := 1; i < n; i++ {
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] {
                dp[i] = max(dp[i], dp[j]+1)
            }
        }
        maxLen = max(maxLen, dp[i])
    }
    
    return maxLen
}
```

### Problem 4: Edit Distance

```go
func minDistance(word1, word2 string) int {
    m, n := len(word1), len(word2)
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    for i := 0; i <= m; i++ {
        dp[i][0] = i
    }
    for j := 0; j <= n; j++ {
        dp[0][j] = j
    }
    
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if word1[i-1] == word2[j-1] {
                dp[i][j] = dp[i-1][j-1]
            } else {
                dp[i][j] = min(
                    dp[i-1][j],    // delete
                    dp[i][j-1],    // insert
                    dp[i-1][j-1],  // replace
                ) + 1
            }
        }
    }
    
    return dp[m][n]
}

func min(a, b, c int) int {
    if a < b && a < c {
        return a
    }
    if b < c {
        return b
    }
    return c
}
```

### Problem 5: Maximum Product Subarray

```go
func maxProduct(nums []int) int {
    maxProd, minProd, result := nums[0], nums[0], nums[0]
    
    for i := 1; i < len(nums); i++ {
        candidates := []int{
            nums[i],
            maxProd * nums[i],
            minProd * nums[i],
        }
        
        maxProd, minProd = candidates[0], candidates[0]
        for _, v := range candidates {
            if v > maxProd {
                maxProd = v
            }
            if v < minProd {
                minProd = v
            }
        }
        
        if maxProd > result {
            result = maxProd
        }
    }
    
    return result
}
```

---

## Optimization Techniques

### Space Optimization

```go
// 2D DP -> 1D DP
func optimizedDP(n int) int {
    // Instead of dp[i][j], use dp[j] and update in place
    dp := make([]int, n+1)
    // ... optimization logic
    return dp[n]
}
```

### Memoization (Top-Down)

```go
func fibonacciMemo(n int) int {
    memo := make(map[int]int)
    
    var fib func(int) int
    fib = func(n int) int {
        if n <= 1 {
            return n
        }
        if val, exists := memo[n]; exists {
            return val
        }
        memo[n] = fib(n-1) + fib(n-2)
        return memo[n]
    }
    
    return fib(n)
}
```

---

## Best Practices

### 1. Identify DP Pattern

**Why:**
- **Recognition**: Recognize DP problems
- **Pattern matching**: Match to known patterns
- **Solution**: Apply appropriate solution

**Guidelines:**
- **Optimal substructure**: Check for optimal substructure
- **Overlapping subproblems**: Check for overlapping subproblems
- **Pattern**: Identify DP pattern

### 2. Start with Recursive Solution

**Why:**
- **Understanding**: Understand problem structure
- **Base case**: Identify base cases
- **Transition**: Identify state transitions

**Guidelines:**
- **Recursive**: Start with recursive solution
- **Memoize**: Add memoization
- **Iterative**: Convert to iterative if needed

### 3. Optimize Space When Possible

**Why:**
- **Memory**: Reduce memory usage
- **Efficiency**: More efficient
- **Scalability**: Better scalability

**Guidelines:**
- **Space optimization**: Optimize space when possible
- **1D instead of 2D**: Use 1D when possible
- **Variables**: Use variables instead of arrays when possible

---

## Summary

Dynamic programming is essential for solving optimization problems efficiently in Go. Understanding DP patterns, common problems, optimization techniques, and best practices is crucial for effective DP problem solving.

**Key Takeaways:**
- **Dynamic programming**: Solving problems by breaking into subproblems and storing results (optimal substructure, overlapping subproblems)
- **DP patterns**: 1D DP (Fibonacci, climbing stairs), 2D DP (LCS, edit distance), optimization
- **Common DP problems**: Fibonacci, coin change, LCS, max subarray, LIS, edit distance, max product
- **Optimization techniques**: Space optimization (2D to 1D), memoization (top-down), iterative (bottom-up)
- **Best practices**: Identify DP pattern, start with recursive solution, optimize space when possible

**DP Characteristics:**
- **Optimal substructure**: Optimal solution contains optimal subproblems
- **Overlapping subproblems**: Subproblems repeat
- **Memoization**: Store results to avoid recomputation

**Best Practices:**
- Identify DP pattern
- Start with recursive solution
- Optimize space when possible

**Next Steps:**
- Practice DP patterns
- Learn common problems
- Master optimization techniques
- Apply best practices

