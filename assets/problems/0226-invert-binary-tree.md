---
id: 226
title: "Invert Binary Tree"
slug: invert-binary-tree
difficulty: Easy
tags: [Tree, Depth-First Search, Breadth-First Search, Binary Tree]
neetcode150_category: Trees
blind75_category: Tree
date_solved: 2026-03-29
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(h)
"@local": [Tree Traversal]
insight: "root.Left, root.Right = root.Right, root.Left，遞迴到每個 node"
---

# Invert Binary Tree

Given the `root` of a binary tree, invert the tree, and return _its root_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/14/invert1-tree.jpg)

**Input:** root = \[4,2,7,1,3,6,9\]
**Output:** \[4,7,2,9,6,3,1\]

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/03/14/invert2-tree.jpg)

**Input:** root = \[2,1,3\]
**Output:** \[2,3,1\]

**Example 3:**

**Input:** root = \[\]
**Output:** \[\]

**Constraints:**

*   The number of nodes in the tree is in the range `[0, 100]`.
*   `-100 <= Node.val <= 100`

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
func invertTree(root *TreeNode) *TreeNode {
    
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

function invertTree(root: TreeNode | null): TreeNode | null {
    
};
```
