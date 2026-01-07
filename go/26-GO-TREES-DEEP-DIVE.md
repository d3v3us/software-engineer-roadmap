# Go Trees Deep Dive - Complete Understanding

## Table of Contents
1. [What are Trees?](#what-are-trees)
2. [Why Use Trees?](#why-use-trees)
3. [Tree Implementation](#tree-implementation)
4. [Tree Traversal](#tree-traversal)
5. [Common Tree Problems](#common-tree-problems)
6. [Best Practices](#best-practices)

---

## What are Trees?

### Definition

**Tree**: Hierarchical data structure with nodes and edges.

**Key Characteristics:**
- **Root**: Top node
- **Nodes**: Elements with children
- **Leaves**: Nodes without children
- **Hierarchical**: Parent-child relationships

### Real-World Analogy

**Tree = Family Tree:**
- **Root**: Ancestor
- **Nodes**: Family members
- **Edges**: Relationships
- **Hierarchy**: Generations

**Programming:**
- **Root**: Top node
- **Nodes**: Data with children
- **Traversal**: Visit nodes

---

## Why Use Trees?

### Advantages

**1. Hierarchical Data:**
```
Natural hierarchy
  ↓
Tree structure
  ↓
Efficient representation
```

**2. Fast Search:**
```
Binary Search Tree
  ↓
O(log n) search
  ↓
Efficient
```

**3. Flexible Structure:**
```
Different tree types
  ↓
Different use cases
  ↓
Flexible
```

---

## Tree Implementation

### Binary Tree Node

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}
```

### Creating Tree

```go
// Create tree
root := &TreeNode{Val: 1}
root.Left = &TreeNode{Val: 2}
root.Right = &TreeNode{Val: 3}
root.Left.Left = &TreeNode{Val: 4}
root.Left.Right = &TreeNode{Val: 5}
```

### Binary Search Tree

```go
type BSTNode struct {
    Val   int
    Left  *BSTNode
    Right *BSTNode
}

func insert(root *BSTNode, val int) *BSTNode {
    if root == nil {
        return &BSTNode{Val: val}
    }
    
    if val < root.Val {
        root.Left = insert(root.Left, val)
    } else {
        root.Right = insert(root.Right, val)
    }
    
    return root
}
```

---

## Tree Traversal

### Inorder Traversal

```go
func inorderTraversal(root *TreeNode) []int {
    var result []int
    var dfs func(*TreeNode)
    
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        dfs(node.Left)
        result = append(result, node.Val)
        dfs(node.Right)
    }
    
    dfs(root)
    return result
}
```

### Preorder Traversal

```go
func preorderTraversal(root *TreeNode) []int {
    var result []int
    var dfs func(*TreeNode)
    
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        result = append(result, node.Val)
        dfs(node.Left)
        dfs(node.Right)
    }
    
    dfs(root)
    return result
}
```

### Postorder Traversal

```go
func postorderTraversal(root *TreeNode) []int {
    var result []int
    var dfs func(*TreeNode)
    
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        dfs(node.Left)
        dfs(node.Right)
        result = append(result, node.Val)
    }
    
    dfs(root)
    return result
}
```

### Level Order Traversal (BFS)

```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil {
        return [][]int{}
    }
    
    var result [][]int
    queue := []*TreeNode{root}
    
    for len(queue) > 0 {
        levelSize := len(queue)
        level := []int{}
        
        for i := 0; i < levelSize; i++ {
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        
        result = append(result, level)
    }
    
    return result
}
```

---

## Common Tree Problems

### Problem 1: Maximum Depth

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    
    leftDepth := maxDepth(root.Left)
    rightDepth := maxDepth(root.Right)
    
    return max(leftDepth, rightDepth) + 1
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### Problem 2: Same Tree

```go
func isSameTree(p *TreeNode, q *TreeNode) bool {
    if p == nil && q == nil {
        return true
    }
    
    if p == nil || q == nil {
        return false
    }
    
    return p.Val == q.Val && 
           isSameTree(p.Left, q.Left) && 
           isSameTree(p.Right, q.Right)
}
```

### Problem 3: Invert Binary Tree

```go
func invertTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    
    root.Left, root.Right = root.Right, root.Left
    invertTree(root.Left)
    invertTree(root.Right)
    
    return root
}
```

### Problem 4: Validate Binary Search Tree

```go
func isValidBST(root *TreeNode) bool {
    return validate(root, nil, nil)
}

func validate(node *TreeNode, min, max *int) bool {
    if node == nil {
        return true
    }
    
    if min != nil && node.Val <= *min {
        return false
    }
    
    if max != nil && node.Val >= *max {
        return false
    }
    
    return validate(node.Left, min, &node.Val) && 
           validate(node.Right, &node.Val, max)
}
```

### Problem 5: Lowest Common Ancestor

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || root == p || root == q {
        return root
    }
    
    left := lowestCommonAncestor(root.Left, p, q)
    right := lowestCommonAncestor(root.Right, p, q)
    
    if left != nil && right != nil {
        return root
    }
    
    if left != nil {
        return left
    }
    
    return right
}
```

---

## Best Practices

### 1. Use Recursion for Tree Problems

**Why:**
- **Natural fit**: Trees are recursive structures
- **Clarity**: Clear and readable
- **Efficiency**: Efficient for tree operations

**Guidelines:**
- **Recursion**: Use recursion for tree traversal
- **Base case**: Handle base case (nil node)
- **Recursive case**: Process node and recurse

### 2. Handle Nil Nodes

**Why:**
- **Safety**: Prevents panics
- **Correctness**: Correct behavior
- **Robustness**: Robust code

**Guidelines:**
- **Check nil**: Always check for nil nodes
- **Base case**: Handle nil in base case
- **Edge cases**: Handle empty tree

### 3. Use Helper Functions

**Why:**
- **Clarity**: Clear function signatures
- **Reusability**: Reusable code
- **Maintainability**: Easier to maintain

**Guidelines:**
- **Helper functions**: Use for complex operations
- **Recursive helpers**: Use for recursive operations
- **Validation**: Use for validation logic

---

## Summary

Trees are essential hierarchical data structures in Go. Understanding implementation, traversal, common problems, and best practices is crucial for effective tree manipulation.

**Key Takeaways:**
- **Trees**: Hierarchical data structure (root, nodes, leaves, parent-child relationships)
- **Tree implementation**: Binary tree node (Val, Left, Right), creating trees, BST implementation
- **Tree traversal**: Inorder, preorder, postorder (DFS), level order (BFS)
- **Common tree problems**: Maximum depth, same tree, invert tree, validate BST, LCA
- **Best practices**: Use recursion, handle nil nodes, use helper functions

**Tree Advantages:**
- **Hierarchical data**: Natural hierarchy representation
- **Fast search**: O(log n) in BST
- **Flexible structure**: Different tree types

**Best Practices:**
- Use recursion for tree problems
- Handle nil nodes
- Use helper functions

**Next Steps:**
- Practice tree traversal
- Learn common problems
- Master recursive thinking
- Apply best practices

