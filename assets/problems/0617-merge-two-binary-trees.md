---
id: 617
title: "Merge Two Binary Trees"
slug: merge-two-binary-trees
difficulty: Easy
tags: [Tree, Depth-First Search, Breadth-First Search, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Merge Two Binary Trees

You are given two binary trees `root1` and `root2`.

Imagine that when you put one of them to cover the other, some nodes of the two trees are overlapped while the others are not. You need to merge the two trees into a new binary tree. The merge rule is that if two nodes overlap, then sum node values up as the new value of the merged node. Otherwise, the NOT null node will be used as the node of the new tree.

Return _the merged tree_.

**Note:** The merging process must start from the root nodes of both trees.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/05/merge.jpg)

**Input:** root1 = \[1,3,2,5\], root2 = \[2,1,3,null,4,null,7\]
**Output:** \[3,4,5,5,4,null,7\]

**Example 2:**

**Input:** root1 = \[1\], root2 = \[1,2\]
**Output:** \[2,2\]

**Constraints:**

*   The number of nodes in both trees is in the range `[0, 2000]`.
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
func mergeTrees(root1 *TreeNode, root2 *TreeNode) *TreeNode {
    
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

function mergeTrees(root1: TreeNode | null, root2: TreeNode | null): TreeNode | null {
    
};
```
