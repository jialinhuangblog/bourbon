---
id: 236
title: "Lowest Common Ancestor of a Binary Tree"
slug: lowest-common-ancestor-of-a-binary-tree
difficulty: Medium
tags: [Tree, Depth-First Search, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: 2026-04-16
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(h)
insight: "後序 DFS，左右各回傳找到的節點，兩側都非 null 就是 LCA"
---

# Lowest Common Ancestor of a Binary Tree

Given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree.

According to the [definition of LCA on Wikipedia](https://en.wikipedia.org/wiki/Lowest_common_ancestor): “The lowest common ancestor is defined between two nodes `p` and `q` as the lowest node in `T` that has both `p` and `q` as descendants (where we allow **a node to be a descendant of itself**).”

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

**Input:** root = \[3,5,1,6,2,0,8,null,null,7,4\], p = 5, q = 1
**Output:** 3
**Explanation:** The LCA of nodes 5 and 1 is 3.

**Example 2:**

![](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

**Input:** root = \[3,5,1,6,2,0,8,null,null,7,4\], p = 5, q = 4
**Output:** 5
**Explanation:** The LCA of nodes 5 and 4 is 5, since a node can be a descendant of itself according to the LCA definition.

**Example 3:**

**Input:** root = \[1,2\], p = 1, q = 2
**Output:** 1

**Constraints:**

*   The number of nodes in the tree is in the range `[2, 105]`.
*   `-109 <= Node.val <= 109`
*   All `Node.val` are **unique**.
*   `p != q`
*   `p` and `q` will exist in the tree.

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
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
  
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

function lowestCommonAncestor(root: TreeNode | null, p: TreeNode | null, q: TreeNode | null): TreeNode | null {
	
};
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
