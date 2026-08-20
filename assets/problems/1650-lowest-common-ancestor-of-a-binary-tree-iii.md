---
id: 1650
title: "Lowest Common Ancestor of a Binary Tree III"
slug: lowest-common-ancestor-of-a-binary-tree-iii
difficulty: Medium
tags: [Hash Table, Two Pointers, Tree, Binary Tree]
neetcode150_category: null
blind75_category: null
date_solved: 2026-05-05
languages: [typescript, golang]
time_complexity: O(h)
space_complexity: O(1)
insight: "兩指標各從 p、q 往上走，到 null 就跳到對方起點。第二輪會在 LCA 對齊"
---

# Lowest Common Ancestor of a Binary Tree III

> LeetCode Premium.

Given two nodes of a binary tree `p` and `q`, return their lowest common ancestor (LCA).

Each node has a reference to its **parent** node. The definition for `Node` is:

```
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node parent;
}
```

According to the [definition of LCA on Wikipedia](https://en.wikipedia.org/wiki/Lowest_common_ancestor): "The lowest common ancestor of two nodes p and q in a tree T is the lowest node that has both p and q as descendants (where we allow **a node to be a descendant of itself**)."

**Example 1:**

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

**Input:** root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
**Output:** 3

**Example 2:**

**Input:** root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
**Output:** 5

**Constraints:**

* The number of nodes in the tree is in the range `[2, 10^5]`.
* `-10^9 <= Node.val <= 10^9`.
* All `Node.val` are unique.
* `p != q`.
* `p` and `q` exist in the tree.

## Code Template

### TypeScript

```typescript
/**
 * class Node {
 *     val: number
 *     left: Node | null
 *     right: Node | null
 *     parent: Node | null
 * }
 */

function lowestCommonAncestor(p: Node | null, q: Node | null): Node | null {

};
```

### Go

```go
/**
 * type Node struct {
 *     Val int
 *     Left *Node
 *     Right *Node
 *     Parent *Node
 * }
 */

func lowestCommonAncestor(p, q *Node) *Node {

}
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
