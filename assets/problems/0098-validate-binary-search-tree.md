---
id: 98
title: "Validate Binary Search Tree"
slug: validate-binary-search-tree
difficulty: Medium
tags: [Tree, Depth-First Search, Binary Search Tree, Binary Tree]
neetcode150_category: Trees
blind75_category: Tree
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Validate Binary Search Tree

Given the `root` of a binary tree, _determine if it is a valid binary search tree (BST)_.

A **valid BST** is defined as follows:

*   The left subtree of a node contains only nodes with keys **strictly less than** the node's key.
*   The right subtree of a node contains only nodes with keys **strictly greater than** the node's key.
*   Both the left and right subtrees must also be binary search trees.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/12/01/tree1.jpg)

**Input:** root = \[2,1,3\]
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/12/01/tree2.jpg)

**Input:** root = \[5,1,4,null,null,3,6\]
**Output:** false
**Explanation:** The root node's value is 5 but its right child's value is 4.

**Constraints:**

*   The number of nodes in the tree is in the range `[1, 104]`.
*   `-231 <= Node.val <= 231 - 1`

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
func isValidBST(root *TreeNode) bool {
    
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

function isValidBST(root: TreeNode | null): boolean {
    
};
```
