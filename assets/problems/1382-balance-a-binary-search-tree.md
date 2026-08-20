---
id: 1382
title: "Balance a Binary Search Tree"
slug: balance-a-binary-search-tree
difficulty: Medium
tags: [Divide and Conquer, Greedy, Tree, Depth-First Search, Binary Search Tree, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Balance a Binary Search Tree

Given the `root` of a binary search tree, return _a **balanced** binary search tree with the same node values_. If there is more than one answer, return **any of them**.

A binary search tree is **balanced** if the depth of the two subtrees of every node never differs by more than `1`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/08/10/balance1-tree.jpg)

**Input:** root = \[1,null,2,null,3,null,4,null,null\]
**Output:** \[2,1,3,null,null,null,4\]
**Explanation:** This is not the only correct answer, \[3,1,4,null,2\] is also correct.

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/08/10/balanced2-tree.jpg)

**Input:** root = \[2,1,3\]
**Output:** \[2,1,3\]

**Constraints:**

*   The number of nodes in the tree is in the range `[1, 104]`.
*   `1 <= Node.val <= 105`

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
func balanceBST(root *TreeNode) *TreeNode {
    
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

function balanceBST(root: TreeNode | null): TreeNode | null {
    
};
```
