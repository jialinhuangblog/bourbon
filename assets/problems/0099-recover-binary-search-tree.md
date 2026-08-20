---
id: 99
title: "Recover Binary Search Tree"
slug: recover-binary-search-tree
difficulty: Medium
tags: [Tree, Depth-First Search, Binary Search Tree, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Recover Binary Search Tree

You are given the `root` of a binary search tree (BST), where the values of **exactly** two nodes of the tree were swapped by mistake. _Recover the tree without changing its structure_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/28/recover1.jpg)

**Input:** root = \[1,3,null,null,2\]
**Output:** \[3,1,null,null,2\]
**Explanation:** 3 cannot be a left child of 1 because 3 > 1. Swapping 1 and 3 makes the BST valid.

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/10/28/recover2.jpg)

**Input:** root = \[3,1,4,null,null,2\]
**Output:** \[2,1,4,null,null,3\]
**Explanation:** 2 cannot be in the right subtree of 3 because 2 < 3. Swapping 2 and 3 makes the BST valid.

**Constraints:**

*   The number of nodes in the tree is in the range `[2, 1000]`.
*   `-231 <= Node.val <= 231 - 1`

**Follow up:** A solution using `O(n)` space is pretty straight-forward. Could you devise a constant `O(1)` space solution?

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
func recoverTree(root *TreeNode)  {
    
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

/**
 Do not return anything, modify root in-place instead.
 */
function recoverTree(root: TreeNode | null): void {
    
};
```
