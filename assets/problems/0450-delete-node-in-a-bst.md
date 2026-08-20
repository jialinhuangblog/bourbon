---
id: 450
title: "Delete Node in a BST"
slug: delete-node-in-a-bst
difficulty: Medium
tags: [Tree, Binary Search Tree, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Delete Node in a BST

Given a root node reference of a BST and a key, delete the node with the given key in the BST. Return _the **root node reference** (possibly updated) of the BST_.

Basically, the deletion can be divided into two stages:

1.  Search for a node to remove.
2.  If the node is found, delete the node.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/09/04/del_node_1.jpg)

**Input:** root = \[5,3,6,2,4,null,7\], key = 3
**Output:** \[5,4,6,2,null,null,7\]
**Explanation:** Given key to delete is 3. So we find the node with value 3 and delete it.
One valid answer is \[5,4,6,2,null,null,7\], shown in the above BST.
Please notice that another valid answer is \[5,2,6,null,4,null,7\] and it's also accepted.
![](https://assets.leetcode.com/uploads/2020/09/04/del_node_supp.jpg)

**Example 2:**

**Input:** root = \[5,3,6,2,4,null,7\], key = 0
**Output:** \[5,3,6,2,4,null,7\]
**Explanation:** The tree does not contain a node with value = 0.

**Example 3:**

**Input:** root = \[\], key = 0
**Output:** \[\]

**Constraints:**

*   The number of nodes in the tree is in the range `[0, 104]`.
*   `-105 <= Node.val <= 105`
*   Each node has a **unique** value.
*   `root` is a valid binary search tree.
*   `-105 <= key <= 105`

**Follow up:** Could you solve it with time complexity `O(height of tree)`?

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
func deleteNode(root *TreeNode, key int) *TreeNode {
    
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

function deleteNode(root: TreeNode | null, key: number): TreeNode | null {
    
};
```
