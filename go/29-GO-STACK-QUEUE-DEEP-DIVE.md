# Go Stack and Queue Deep Dive - Complete Understanding

## Table of Contents
1. [What are Stack and Queue?](#what-are-stack-and-queue)
2. [Why Use Stack and Queue?](#why-use-stack-and-queue)
3. [Stack Implementation](#stack-implementation)
4. [Queue Implementation](#queue-implementation)
5. [Common Problems](#common-problems)
6. [Best Practices](#best-practices)

---

## What are Stack and Queue?

### Definition

**Stack**: LIFO (Last In First Out) data structure.

**Queue**: FIFO (First In First Out) data structure.

**Key Characteristics:**
- **Stack**: Last element added is first removed
- **Queue**: First element added is first removed
- **Operations**: Push/Pop (stack), Enqueue/Dequeue (queue)
- **Efficiency**: O(1) operations

### Real-World Analogy

**Stack = Stack of Plates:**
- **Add**: Add plate on top
- **Remove**: Remove top plate
- **LIFO**: Last in, first out

**Queue = Line of People:**
- **Add**: Join at end
- **Remove**: Leave from front
- **FIFO**: First in, first out

---

## Why Use Stack and Queue?

### Use Cases

**Stack:**
- **Function calls**: Call stack
- **Expression evaluation**: Postfix evaluation
- **Backtracking**: DFS, undo operations
- **Parentheses matching**: Valid parentheses

**Queue:**
- **BFS**: Breadth-first search
- **Task scheduling**: Process scheduling
- **Buffering**: Message buffering
- **Level-order traversal**: Tree traversal

---

## Stack Implementation

### Array-Based Stack

```go
type Stack struct {
    items []int
}

func NewStack() *Stack {
    return &Stack{items: []int{}}
}

func (s *Stack) Push(item int) {
    s.items = append(s.items, item)
}

func (s *Stack) Pop() (int, bool) {
    if len(s.items) == 0 {
        return 0, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

func (s *Stack) Peek() (int, bool) {
    if len(s.items) == 0 {
        return 0, false
    }
    return s.items[len(s.items)-1], true
}

func (s *Stack) IsEmpty() bool {
    return len(s.items) == 0
}

func (s *Stack) Size() int {
    return len(s.items)
}
```

### Generic Stack (Go 1.18+)

```go
type Stack[T any] struct {
    items []T
}

func NewStack[T any]() *Stack[T] {
    return &Stack[T]{items: []T{}}
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```

---

## Queue Implementation

### Array-Based Queue

```go
type Queue struct {
    items []int
}

func NewQueue() *Queue {
    return &Queue{items: []int{}}
}

func (q *Queue) Enqueue(item int) {
    q.items = append(q.items, item)
}

func (q *Queue) Dequeue() (int, bool) {
    if len(q.items) == 0 {
        return 0, false
    }
    item := q.items[0]
    q.items = q.items[1:]
    return item, true
}

func (q *Queue) Peek() (int, bool) {
    if len(q.items) == 0 {
        return 0, false
    }
    return q.items[0], true
}

func (q *Queue) IsEmpty() bool {
    return len(q.items) == 0
}

func (q *Queue) Size() int {
    return len(q.items)
}
```

### Circular Queue

```go
type CircularQueue struct {
    items []int
    front int
    rear  int
    size  int
    cap   int
}

func NewCircularQueue(capacity int) *CircularQueue {
    return &CircularQueue{
        items: make([]int, capacity),
        front: 0,
        rear:  -1,
        size:  0,
        cap:   capacity,
    }
}

func (q *CircularQueue) Enqueue(item int) bool {
    if q.size == q.cap {
        return false
    }
    q.rear = (q.rear + 1) % q.cap
    q.items[q.rear] = item
    q.size++
    return true
}

func (q *CircularQueue) Dequeue() (int, bool) {
    if q.size == 0 {
        return 0, false
    }
    item := q.items[q.front]
    q.front = (q.front + 1) % q.cap
    q.size--
    return item, true
}
```

---

## Common Problems

### Problem 1: Valid Parentheses

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

### Problem 2: Daily Temperatures

```go
func dailyTemperatures(temperatures []int) []int {
    n := len(temperatures)
    result := make([]int, n)
    stack := []int{}
    
    for i := 0; i < n; i++ {
        for len(stack) > 0 && temperatures[i] > temperatures[stack[len(stack)-1]] {
            idx := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            result[idx] = i - idx
        }
        stack = append(stack, i)
    }
    
    return result
}
```

### Problem 3: Implement Queue using Stacks

```go
type MyQueue struct {
    input  []int
    output []int
}

func Constructor() MyQueue {
    return MyQueue{
        input:  []int{},
        output: []int{},
    }
}

func (q *MyQueue) Push(x int) {
    q.input = append(q.input, x)
}

func (q *MyQueue) Pop() int {
    q.move()
    item := q.output[len(q.output)-1]
    q.output = q.output[:len(q.output)-1]
    return item
}

func (q *MyQueue) Peek() int {
    q.move()
    return q.output[len(q.output)-1]
}

func (q *MyQueue) Empty() bool {
    return len(q.input) == 0 && len(q.output) == 0
}

func (q *MyQueue) move() {
    if len(q.output) == 0 {
        for len(q.input) > 0 {
            q.output = append(q.output, q.input[len(q.input)-1])
            q.input = q.input[:len(q.input)-1]
        }
    }
}
```

### Problem 4: Implement Stack using Queues

```go
type MyStack struct {
    queue []int
}

func Constructor() MyStack {
    return MyStack{queue: []int{}}
}

func (s *MyStack) Push(x int) {
    s.queue = append(s.queue, x)
    for i := 0; i < len(s.queue)-1; i++ {
        s.queue = append(s.queue, s.queue[0])
        s.queue = s.queue[1:]
    }
}

func (s *MyStack) Pop() int {
    item := s.queue[0]
    s.queue = s.queue[1:]
    return item
}

func (s *MyStack) Top() int {
    return s.queue[0]
}

func (s *MyStack) Empty() bool {
    return len(s.queue) == 0
}
```

---

## Best Practices

### 1. Use Built-in Slices

**Why:**
- **Simplicity**: Simple implementation
- **Efficiency**: Efficient operations
- **Native**: Native Go support

**Guidelines:**
- **Slices**: Use slices for stack/queue
- **Append**: Use append for push/enqueue
- **Slicing**: Use slicing for pop/dequeue

### 2. Handle Empty Cases

**Why:**
- **Safety**: Prevent panics
- **Correctness**: Correct behavior
- **Robustness**: Robust code

**Guidelines:**
- **Check empty**: Always check if empty
- **Return error**: Return error/bool for empty
- **Handle gracefully**: Handle gracefully

### 3. Consider Performance

**Why:**
- **Efficiency**: Efficient operations
- **Scalability**: Better scalability
- **Optimization**: Optimize when needed

**Guidelines:**
- **O(1) operations**: Maintain O(1) operations
- **Pre-allocate**: Pre-allocate when size known
- **Circular queue**: Use circular queue for fixed size

---

## Summary

Stack and queue are essential data structures in Go. Understanding implementation, common problems, and best practices is crucial for effective stack and queue usage.

**Key Takeaways:**
- **Stack**: LIFO data structure (last in, first out, O(1) operations)
- **Queue**: FIFO data structure (first in, first out, O(1) operations)
- **Stack implementation**: Array-based stack, generic stack (Go 1.18+)
- **Queue implementation**: Array-based queue, circular queue
- **Common problems**: Valid parentheses, daily temperatures, implement queue/stack using other
- **Best practices**: Use built-in slices, handle empty cases, consider performance

**Stack and Queue Use Cases:**
- **Stack**: Function calls, expression evaluation, backtracking, parentheses matching
- **Queue**: BFS, task scheduling, buffering, level-order traversal

**Best Practices:**
- Use built-in slices
- Handle empty cases
- Consider performance

**Next Steps:**
- Practice stack operations
- Learn queue operations
- Master common problems
- Apply best practices

