---
id: 235
title: "Lowest Common Ancestor of a Binary Search Tree"
slug: lowest-common-ancestor-of-a-binary-search-tree
difficulty: Medium
tags: [Tree, Depth-First Search, Binary Search Tree, Binary Tree]
neetcode150_category: Trees
blind75_category: Tree
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Lowest Common Ancestor of a Binary Search Tree

Given a binary search tree (BST), find the lowest common ancestor (LCA) node of two given nodes in the BST.

According to the [definition of LCA on Wikipedia](https://en.wikipedia.org/wiki/Lowest_common_ancestor): “The lowest common ancestor is defined between two nodes `p` and `q` as the lowest node in `T` that has both `p` and `q` as descendants (where we allow **a node to be a descendant of itself**).”

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/12/14/binarysearchtree_improved.png)

**Input:** root = \[6,2,8,0,4,7,9,null,null,3,5\], p = 2, q = 8
**Output:** 6
**Explanation:** The LCA of nodes 2 and 8 is 6.

**Example 2:**

![](https://assets.leetcode.com/uploads/2018/12/14/binarysearchtree_improved.png)

**Input:** root = \[6,2,8,0,4,7,9,null,null,3,5\], p = 2, q = 4
**Output:** 2
**Explanation:** The LCA of nodes 2 and 4 is 2, since a node can be a descendant of itself according to the LCA definition.

**Example 3:**

**Input:** root = \[2,1\], p = 2, q = 1
**Output:** 2

**Constraints:**

*   The number of nodes in the tree is in the range `[2, 105]`.
*   `-109 <= Node.val <= 109`
*   All `Node.val` are **unique**.
*   `p != q`
*   `p` and `q` will exist in the BST.

## Code Template

### Go
```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val   int
 *     Left  *TreeNode
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
