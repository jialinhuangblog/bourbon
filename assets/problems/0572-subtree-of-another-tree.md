---
id: 572
title: "Subtree of Another Tree"
slug: subtree-of-another-tree
difficulty: Easy
tags: [Tree, Depth-First Search, String Matching, Binary Tree, Hash Function]
neetcode150_category: Trees
blind75_category: Tree
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(m * n)
space_complexity: O(h)
"@local": [Tree Traversal]
insight: "每個節點呼叫一次 Same Tree；序列化版要補 null 記號跟節點邊界，不然 12 會被讀成 1 和 2"
---

# Subtree of Another Tree

Given the roots of two binary trees `root` and `subRoot`, return `true` if there is a subtree of `root` with the same structure and node values of `subRoot` and `false` otherwise.

A subtree of a binary tree `tree` is a tree that consists of a node in `tree` and all of this node's descendants. The tree `tree` could also be considered as a subtree of itself.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/04/28/subtree1-tree.jpg)

**Input:** root = \[3,4,5,1,2\], subRoot = \[4,1,2\]
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/04/28/subtree2-tree.jpg)

**Input:** root = \[3,4,5,1,2,null,null,null,null,0\], subRoot = \[4,1,2\]
**Output:** false

**Constraints:**

*   The number of nodes in the `root` tree is in the range `[1, 2000]`.
*   The number of nodes in the `subRoot` tree is in the range `[1, 1000]`.
*   `-104 <= root.val <= 104`
*   `-104 <= subRoot.val <= 104`

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
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    
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

function isSubtree(root: TreeNode | null, subRoot: TreeNode | null): boolean {
    
};
```
