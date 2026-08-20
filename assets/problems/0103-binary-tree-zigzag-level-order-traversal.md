---
id: 103
title: "Binary Tree Zigzag Level Order Traversal"
slug: binary-tree-zigzag-level-order-traversal
difficulty: Medium
tags: [Tree, Breadth-First Search, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: 2026-02-28
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "BFS + flip append direction each level"
---

# Binary Tree Zigzag Level Order Traversal

Given the `root` of a binary tree, return _the zigzag level order traversal of its nodes' values_. (i.e., from left to right, then right to left for the next level and alternate between).

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg)

**Input:** root = \[3,9,20,null,null,15,7\]
**Output:** \[\[3\],\[20,9\],\[15,7\]\]

**Example 2:**

**Input:** root = \[1\]
**Output:** \[\[1\]\]

**Example 3:**

**Input:** root = \[\]
**Output:** \[\]

**Constraints:**

*   The number of nodes in the tree is in the range `[0, 2000]`.
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
func zigzagLevelOrder(root *TreeNode) [][]int {
    
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

function zigzagLevelOrder(root: TreeNode | null): number[][] {
    
};
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
