# Go String Algorithms Deep Dive - Complete Understanding

## Table of Contents
1. [What are String Algorithms?](#what-are-string-algorithms)
2. [Why String Algorithms Matter](#why-string-algorithms-matter)
3. [String Manipulation](#string-manipulation)
4. [String Searching](#string-searching)
5. [Common String Problems](#common-string-problems)
6. [Best Practices](#best-practices)

---

## What are String Algorithms?

### Definition

**String Algorithms**: Algorithms specifically designed for string operations.

**Key Characteristics:**
- **String operations**: Work with Go strings
- **Efficient**: Efficient time and space complexity
- **Pattern matching**: Pattern matching algorithms
- **Text processing**: Text processing algorithms

### Real-World Analogy

**String Algorithms = Text Processing:**
- **Text**: String data
- **Algorithm**: Processing logic
- **Pattern**: Pattern to find
- **Result**: Processed result

**Programming:**
- **String**: Go string (immutable)
- **Algorithm**: Processing algorithm
- **Efficiency**: Time and space efficient

---

## Why String Algorithms Matter?

### Use Cases

**1. Text Processing:**
```
Text analysis
  ↓
String algorithms
  ↓
Efficient processing
```

**2. Pattern Matching:**
```
Search patterns
  ↓
String algorithms
  ↓
Fast matching
```

**3. Data Validation:**
```
Input validation
  ↓
String algorithms
  ↓
Correct validation
```

---

## String Manipulation

### String Reversal

```go
func reverseString(s string) string {
    runes := []rune(s)
    left, right := 0, len(runes)-1
    
    for left < right {
        runes[left], runes[right] = runes[right], runes[left]
        left++
        right--
    }
    
    return string(runes)
}
```

### Palindrome Check

```go
func isPalindrome(s string) bool {
    left, right := 0, len(s)-1
    
    for left < right {
        if s[left] != s[right] {
            return false
        }
        left++
        right--
    }
    
    return true
}
```

### Longest Palindromic Substring

```go
func longestPalindrome(s string) string {
    n := len(s)
    if n == 0 {
        return ""
    }
    
    start, maxLen := 0, 1
    
    for i := 0; i < n; i++ {
        // Odd length
        l, r := i, i
        for l >= 0 && r < n && s[l] == s[r] {
            if r-l+1 > maxLen {
                start = l
                maxLen = r - l + 1
            }
            l--
            r++
        }
        
        // Even length
        l, r = i, i+1
        for l >= 0 && r < n && s[l] == s[r] {
            if r-l+1 > maxLen {
                start = l
                maxLen = r - l + 1
            }
            l--
            r++
        }
    }
    
    return s[start : start+maxLen]
}
```

---

## String Searching

### Longest Substring Without Repeating Characters

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

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### Anagram Check

```go
func isAnagram(s, t string) bool {
    if len(s) != len(t) {
        return false
    }
    
    count := make(map[rune]int)
    
    for _, char := range s {
        count[char]++
    }
    
    for _, char := range t {
        count[char]--
        if count[char] < 0 {
            return false
        }
    }
    
    return true
}
```

### Group Anagrams

```go
func groupAnagrams(strs []string) [][]string {
    groups := make(map[string][]string)
    
    for _, str := range strs {
        key := sortString(str)
        groups[key] = append(groups[key], str)
    }
    
    result := make([][]string, 0, len(groups))
    for _, group := range groups {
        result = append(result, group)
    }
    
    return result
}

func sortString(s string) string {
    runes := []rune(s)
    sort.Slice(runes, func(i, j int) bool {
        return runes[i] < runes[j]
    })
    return string(runes)
}
```

---

## Common String Problems

### Problem 1: Valid Anagram

```go
func isAnagram(s, t string) bool {
    if len(s) != len(t) {
        return false
    }
    
    count := [26]int{}
    
    for i := 0; i < len(s); i++ {
        count[s[i]-'a']++
        count[t[i]-'a']--
    }
    
    for _, c := range count {
        if c != 0 {
            return false
        }
    }
    
    return true
}
```

### Problem 2: Longest Common Prefix

```go
func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    
    prefix := strs[0]
    
    for i := 1; i < len(strs); i++ {
        for len(prefix) > 0 && !strings.HasPrefix(strs[i], prefix) {
            prefix = prefix[:len(prefix)-1]
        }
        if len(prefix) == 0 {
            return ""
        }
    }
    
    return prefix
}
```

### Problem 3: Reverse Words in String

```go
func reverseWords(s string) string {
    words := strings.Fields(s)
    left, right := 0, len(words)-1
    
    for left < right {
        words[left], words[right] = words[right], words[left]
        left++
        right--
    }
    
    return strings.Join(words, " ")
}
```

### Problem 4: Valid Parentheses

```go
func isValid(s string) bool {
    stack := []rune{}
    pairs := map[rune]rune{
        ')': '(',
        '}': '{',
        ']': '[',
    }
    
    for _, char := range s {
        if char == '(' || char == '{' || char == '[' {
            stack = append(stack, char)
        } else {
            if len(stack) == 0 {
                return false
            }
            if stack[len(stack)-1] != pairs[char] {
                return false
            }
            stack = stack[:len(stack)-1]
        }
    }
    
    return len(stack) == 0
}
```

### Problem 5: String to Integer (atoi)

```go
func myAtoi(s string) int {
    s = strings.TrimSpace(s)
    if len(s) == 0 {
        return 0
    }
    
    sign := 1
    i := 0
    
    if s[0] == '-' {
        sign = -1
        i++
    } else if s[0] == '+' {
        i++
    }
    
    result := 0
    for i < len(s) && s[i] >= '0' && s[i] <= '9' {
        digit := int(s[i] - '0')
        if result > (math.MaxInt32-digit)/10 {
            if sign == 1 {
                return math.MaxInt32
            }
            return math.MinInt32
        }
        result = result*10 + digit
        i++
    }
    
    return sign * result
}
```

---

## Best Practices

### 1. Use Runes for Unicode

**Why:**
- **Unicode**: Proper Unicode handling
- **Correctness**: Correct character handling
- **International**: International text support

**Guidelines:**
- **Runes**: Use runes for Unicode strings
- **Range**: Use range for string iteration
- **Conversion**: Convert to []rune when needed

### 2. Use Built-in String Functions

**Why:**
- **Efficiency**: Efficient implementations
- **Simplicity**: Simple to use
- **Standard**: Standard library functions

**Guidelines:**
- **strings package**: Use strings package functions
- **HasPrefix/HasSuffix**: Use for prefix/suffix checks
- **Contains**: Use for substring checks

### 3. Handle Edge Cases

**Why:**
- **Correctness**: Correct behavior
- **Robustness**: Robust code
- **Interviews**: Important in interviews

**Guidelines:**
- **Empty strings**: Handle empty strings
- **Single character**: Handle single character
- **Special characters**: Handle special characters

---

## Summary

String algorithms are essential for text processing in Go. Understanding string manipulation, searching, common problems, and best practices is crucial for effective string problem solving.

**Key Takeaways:**
- **String algorithms**: Algorithms for string operations (efficient, pattern matching, text processing)
- **String manipulation**: Reversal, palindrome check, longest palindromic substring
- **String searching**: Longest substring without repeating, anagram check, group anagrams
- **Common string problems**: Valid anagram, longest common prefix, reverse words, valid parentheses, atoi
- **Best practices**: Use runes for Unicode, use built-in string functions, handle edge cases

**String Algorithm Techniques:**
- **Two pointers**: Efficient string processing
- **Sliding window**: Substring problems
- **Hash maps**: Character counting

**Best Practices:**
- Use runes for Unicode
- Use built-in string functions
- Handle edge cases

**Next Steps:**
- Practice string manipulation
- Learn string searching
- Master common problems
- Apply best practices

