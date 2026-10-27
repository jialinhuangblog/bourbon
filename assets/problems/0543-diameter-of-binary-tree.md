---
id: 543
title: "Diameter of Binary Tree"
slug: diameter-of-binary-tree
difficulty: Easy
tags: [Tree, Depth-First Search, Binary Tree]
neetcode150_category: Trees
blind75_category: null
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(h)
"@local": [Tree Traversal]
insight: "最長路徑在某個節點轉彎，長度是左深加右深；回傳的是高度，答案要另外拿變數在回程收"
---

# Diameter of Binary Tree

Given the `root` of a binary tree, return _the length of the **diameter** of the tree_.

The **diameter** of a binary tree is the **length** of the longest path between any two nodes in a tree. This path may or may not pass through the `root`.

The **length** of a path between two nodes is represented by the number of edges between them.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/06/diamtree.jpg)

**Input:** root = \[1,2,3,4,5\]
**Output:** 3
**Explanation:** 3 is the length of the path \[4,2,1,3\] or \[5,2,1,3\].

**Example 2:**

**Input:** root = \[1,2\]
**Output:** 1

**Constraints:**

*   The number of nodes in the tree is in the range `[1, 104]`.
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
func diameterOfBinaryTree(root *TreeNode) int {
    
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

function diameterOfBinaryTree(root: TreeNode | null): number {
    
};
```
