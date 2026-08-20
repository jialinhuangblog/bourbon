---
id: 110
title: "Balanced Binary Tree"
slug: balanced-binary-tree
difficulty: Easy
tags: [Tree, Depth-First Search, Binary Tree]
neetcode150_category: Trees
blind75_category: null
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(h)
"@local": [Tree Traversal]
insight: "一趟後序回傳高度，不合格就回傳 -1 一路往上傳；-1 落在高度值域外面才敢這樣用"
---

# Balanced Binary Tree

Given a binary tree, determine if it is **height-balanced**.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/06/balance_1.jpg)

**Input:** root = \[3,9,20,null,null,15,7\]
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/10/06/balance_2.jpg)

**Input:** root = \[1,2,2,3,3,null,null,4,4\]
**Output:** false

**Example 3:**

**Input:** root = \[\]
**Output:** true

**Constraints:**

*   The number of nodes in the tree is in the range `[0, 5000]`.
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
func isBalanced(root *TreeNode) bool {
    
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

function isBalanced(root: TreeNode | null): boolean {
    
};
```
