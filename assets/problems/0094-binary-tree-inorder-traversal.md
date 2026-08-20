---
id: 94
title: "Binary Tree Inorder Traversal"
slug: binary-tree-inorder-traversal
difficulty: Easy
tags: [Stack, Tree, Depth-First Search, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Binary Tree Inorder Traversal

Given the `root` of a binary tree, return _the inorder traversal of its nodes' values_.

**Example 1:**

**Input:** root = \[1,null,2,3\]

**Output:** \[1,3,2\]

**Explanation:**

![](https://assets.leetcode.com/uploads/2024/08/29/screenshot-2024-08-29-202743.png)

**Example 2:**

**Input:** root = \[1,2,3,4,5,null,8,null,null,6,7,9\]

**Output:** \[4,2,6,5,7,1,3,9,8\]

**Explanation:**

![](https://assets.leetcode.com/uploads/2024/08/29/tree_2.png)

**Example 3:**

**Input:** root = \[\]

**Output:** \[\]

**Example 4:**

**Input:** root = \[1\]

**Output:** \[1\]

**Constraints:**

*   The number of nodes in the tree is in the range `[0, 100]`.
*   `-100 <= Node.val <= 100`

**Follow up:** Recursive solution is trivial, could you do it iteratively?

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
func inorderTraversal(root *TreeNode) []int {
    
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

function inorderTraversal(root: TreeNode | null): number[] {
    
};
```
