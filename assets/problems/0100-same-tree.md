---
id: 100
title: "Same Tree"
slug: same-tree
difficulty: Easy
tags: [Tree, Depth-First Search, Breadth-First Search, Binary Tree]
neetcode150_category: Trees
blind75_category: Tree
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(min(m, n))
space_complexity: O(h)
"@local": [Tree Traversal]
insight: "值相同、左邊相同、右邊相同；兩個 null 算相同、一個 null 算不同是唯一會寫錯的地方"
---

# Same Tree

Given the roots of two binary trees `p` and `q`, write a function to check if they are the same or not.

Two binary trees are considered the same if they are structurally identical, and the nodes have the same value.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/12/20/ex1.jpg)

**Input:** p = \[1,2,3\], q = \[1,2,3\]
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/12/20/ex2.jpg)

**Input:** p = \[1,2\], q = \[1,null,2\]
**Output:** false

**Example 3:**

![](https://assets.leetcode.com/uploads/2020/12/20/ex3.jpg)

**Input:** p = \[1,2,1\], q = \[1,1,2\]
**Output:** false

**Constraints:**

*   The number of nodes in both trees is in the range `[0, 100]`.
*   `-104 <= Node.val <= 104`

## Code Template

### Go
```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func isSameTree(p *TreeNode, q *TreeNode) bool {
    
}
```

### TypeScript
```typescript
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     val: number
 *     left: TreeNode | null
 *     right: TreeNode | null
 *     constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.left = (left===undefined ? null : left)
 *         this.right = (right===undefined ? null : right)
 *     }
 * }
 */

function isSameTree(p: TreeNode | null, q: TreeNode | null): boolean {
    
};
```
