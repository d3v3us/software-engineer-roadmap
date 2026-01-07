# Go Graphs Algorithms Deep Dive - Complete Understanding

## Table of Contents
1. [What are Graphs?](#what-are-graphs)
2. [Why Use Graphs?](#why-use-graphs)
3. [Graph Representation](#graph-representation)
4. [Graph Traversal](#graph-traversal)
5. [Shortest Path Algorithms](#shortest-path-algorithms)
6. [Minimum Spanning Tree](#minimum-spanning-tree)
7. [Best Practices](#best-practices)

---

## What are Graphs?

### Definition

**Graph**: Collection of nodes (vertices) connected by edges.

**Key Characteristics:**
- **Vertices**: Nodes in graph
- **Edges**: Connections between nodes
- **Directed/Undirected**: Can be directed or undirected
- **Weighted/Unweighted**: Can have weights

### Real-World Analogy

**Graph = Network:**
- **Nodes**: Cities
- **Edges**: Roads
- **Weights**: Distances
- **Path**: Route between cities

**Programming:**
- **Vertices**: Data points
- **Edges**: Relationships
- **Algorithms**: Graph algorithms

---

## Why Use Graphs?

### Use Cases

**1. Network Problems:**
```
Social networks
  ↓
Graph structure
  ↓
Graph algorithms
```

**2. Path Finding:**
```
Shortest path
  ↓
Navigation
  ↓
Graph algorithms
```

**3. Dependency Resolution:**
```
Dependencies
  ↓
Graph structure
  ↓
Topological sort
```

---

## Graph Representation

### Adjacency List

```go
type Graph struct {
    Vertices int
    Edges    map[int][]Edge
}

type Edge struct {
    To     int
    Weight int
}

// Create graph
graph := make(map[int][]Edge)
graph[0] = []Edge{{To: 1, Weight: 5}, {To: 2, Weight: 3}}
graph[1] = []Edge{{To: 2, Weight: 2}}
graph[2] = []Edge{{To: 0, Weight: 1}}
```

### Adjacency Matrix

```go
// For unweighted graph
graph := [][]bool{
    {false, true, true},
    {false, false, true},
    {true, false, false},
}

// For weighted graph
graph := [][]int{
    {0, 5, 3},
    {0, 0, 2},
    {1, 0, 0},
}
```

---

## Graph Traversal

### Breadth-First Search (BFS)

```go
func BFS(graph map[int][]int, start int) []int {
    visited := make(map[int]bool)
    queue := []int{start}
    visited[start] = true
    result := []int{}
    
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        result = append(result, node)
        
        for _, neighbor := range graph[node] {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
    
    return result
}
```

### Depth-First Search (DFS)

```go
func DFS(graph map[int][]int, start int) []int {
    visited := make(map[int]bool)
    result := []int{}
    
    var dfs func(int)
    dfs = func(node int) {
        visited[node] = true
        result = append(result, node)
        
        for _, neighbor := range graph[node] {
            if !visited[neighbor] {
                dfs(neighbor)
            }
        }
    }
    
    dfs(start)
    return result
}
```

### Shortest Path in Grid (BFS)

```go
type Point struct {
    X, Y int
}

func shortestPathBFS(grid [][]int, start, end Point) int {
    n, m := len(grid), len(grid[0])
    dirs := []Point{{0, 1}, {1, 0}, {0, -1}, {-1, 0}}
    visited := make([][]bool, n)
    
    for i := range visited {
        visited[i] = make([]bool, m)
    }
    
    queue := []Point{start}
    visited[start.X][start.Y] = true
    steps := 0
    
    for len(queue) > 0 {
        size := len(queue)
        for i := 0; i < size; i++ {
            p := queue[0]
            queue = queue[1:]
            
            if p.X == end.X && p.Y == end.Y {
                return steps
            }
            
            for _, d := range dirs {
                nx, ny := p.X+d.X, p.Y+d.Y
                if nx >= 0 && ny >= 0 && nx < n && ny < m && 
                   !visited[nx][ny] && grid[nx][ny] == 0 {
                    queue = append(queue, Point{nx, ny})
                    visited[nx][ny] = true
                }
            }
        }
        steps++
    }
    
    return -1
}
```

---

## Shortest Path Algorithms

### Dijkstra's Algorithm

```go
import (
    "container/heap"
    "math"
)

type Edge struct {
    To     int
    Weight int
}

type Item struct {
    Node, Dist int
}

type MinHeap []Item

func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i].Dist < h[j].Dist }
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *MinHeap) Push(x interface{}) {
    *h = append(*h, x.(Item))
}

func (h *MinHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

func dijkstra(n int, graph map[int][]Edge, start int) []int {
    dist := make([]int, n)
    for i := range dist {
        dist[i] = math.MaxInt32
    }
    dist[start] = 0
    
    h := &MinHeap{{start, 0}}
    heap.Init(h)
    
    for h.Len() > 0 {
        item := heap.Pop(h).(Item)
        u := item.Node
        
        if item.Dist > dist[u] {
            continue
        }
        
        for _, e := range graph[u] {
            if dist[u]+e.Weight < dist[e.To] {
                dist[e.To] = dist[u] + e.Weight
                heap.Push(h, Item{e.To, dist[e.To]})
            }
        }
    }
    
    return dist
}
```

---

## Minimum Spanning Tree

### Prim's Algorithm

```go
import "container/heap"

type EdgeP struct {
    To     int
    Weight int
}

type ItemP struct {
    Node, Weight int
}

type MinHeapP []ItemP

func (h MinHeapP) Len() int           { return len(h) }
func (h MinHeapP) Less(i, j int) bool { return h[i].Weight < h[j].Weight }
func (h MinHeapP) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *MinHeapP) Push(x interface{}) {
    *h = append(*h, x.(ItemP))
}

func (h *MinHeapP) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

func prim(n int, graph map[int][]EdgeP) int {
    visited := make([]bool, n)
    h := &MinHeapP{{0, 0}}
    heap.Init(h)
    total := 0
    
    for h.Len() > 0 {
        item := heap.Pop(h).(ItemP)
        u := item.Node
        
        if visited[u] {
            continue
        }
        
        visited[u] = true
        total += item.Weight
        
        for _, e := range graph[u] {
            if !visited[e.To] {
                heap.Push(h, ItemP{e.To, e.Weight})
            }
        }
    }
    
    return total
}
```

### Number of Islands (DFS)

```go
func numIslands(grid [][]byte) int {
    n, m := len(grid), len(grid[0])
    
    var dfs func(int, int)
    dfs = func(x, y int) {
        if x < 0 || y < 0 || x >= n || y >= m || grid[x][y] != '1' {
            return
        }
        
        grid[x][y] = '0'
        dfs(x+1, y)
        dfs(x-1, y)
        dfs(x, y+1)
        dfs(x, y-1)
    }
    
    count := 0
    for i := 0; i < n; i++ {
        for j := 0; j < m; j++ {
            if grid[i][j] == '1' {
                count++
                dfs(i, j)
            }
        }
    }
    
    return count
}
```

---

## Best Practices

### 1. Choose Right Representation

**Why:**
- **Efficiency**: Efficient operations
- **Memory**: Memory usage
- **Use case**: Depends on use case

**Guidelines:**
- **Adjacency list**: Sparse graphs
- **Adjacency matrix**: Dense graphs
- **Edge list**: When needed

### 2. Use Appropriate Algorithm

**Why:**
- **Efficiency**: Efficient solution
- **Correctness**: Correct solution
- **Use case**: Matches problem

**Guidelines:**
- **BFS**: Shortest path (unweighted)
- **DFS**: Path finding, cycle detection
- **Dijkstra**: Shortest path (weighted, non-negative)
- **Prim**: Minimum spanning tree

### 3. Handle Edge Cases

**Why:**
- **Correctness**: Correct behavior
- **Robustness**: Robust code
- **Interviews**: Important in interviews

**Guidelines:**
- **Empty graph**: Handle empty graph
- **Single node**: Handle single node
- **Disconnected**: Handle disconnected components

---

## Summary

Graph algorithms are essential for solving network and path-finding problems in Go. Understanding graph representation, traversal, shortest path algorithms, MST, and best practices is crucial for effective graph problem solving.

**Key Takeaways:**
- **Graphs**: Collection of nodes connected by edges (vertices, edges, directed/undirected, weighted/unweighted)
- **Graph representation**: Adjacency list (sparse graphs), adjacency matrix (dense graphs)
- **Graph traversal**: BFS (level-order, shortest path unweighted), DFS (depth-first, path finding)
- **Shortest path algorithms**: Dijkstra's algorithm (weighted, non-negative, O(E log V))
- **Minimum spanning tree**: Prim's algorithm (greedy, O(E log V))
- **Best practices**: Choose right representation, use appropriate algorithm, handle edge cases

**Graph Algorithms:**
- **BFS**: O(V + E) time, shortest path unweighted
- **DFS**: O(V + E) time, path finding
- **Dijkstra**: O(E log V) time, shortest path weighted
- **Prim**: O(E log V) time, MST

**Best Practices:**
- Choose right representation
- Use appropriate algorithm
- Handle edge cases

**Next Steps:**
- Practice graph traversal
- Learn shortest path algorithms
- Master MST algorithms
- Apply best practices

