---
id: 105
title: "Construct Binary Tree from Preorder and Inorder Traversal"
slug: construct-binary-tree-from-preorder-and-inorder-traversal
difficulty: Medium
tags: [Array, Hash Table, Divide and Conquer, Tree, Binary Tree]
neetcode150_category: Trees
blind75_category: Tree
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Construct Binary Tree from Preorder and Inorder Traversal

Given two integer arrays `preorder` and `inorder` where `preorder` is the preorder traversal of a binary tree and `inorder` is the inorder traversal of the same tree, construct and return _the binary tree_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/tree.jpg)

**Input:** preorder = \[3,9,20,15,7\], inorder = \[9,3,15,20,7\]
**Output:** \[3,9,20,null,null,15,7\]

**Example 2:**

**Input:** preorder = \[-1\], inorder = \[-1\]
**Output:** \[-1\]

**Constraints:**

*   `1 <= preorder.length <= 3000`
*   `inorder.length == preorder.length`
*   `-3000 <= preorder[i], inorder[i] <= 3000`
*   `preorder` and `inorder` consist of **unique** values.
*   Each value of `inorder` also appears in `preorder`.
*   `preorder` is **guaranteed** to be the preorder traversal of the tree.
*   `inorder` is **guaranteed** to be the inorder traversal of the tree.

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
func buildTree(preorder []int, inorder []int) *TreeNode {
    
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

function buildTree(preorder: number[], inorder: number[]): TreeNode | null {
    
};
```
