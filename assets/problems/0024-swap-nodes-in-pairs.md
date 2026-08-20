---
id: 24
title: "Swap Nodes in Pairs"
slug: swap-nodes-in-pairs
difficulty: Medium
tags: [Linked List, Recursion]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Swap Nodes in Pairs

Given a linked list, swap every two adjacent nodes and return its head. You must solve the problem without modifying the values in the list's nodes (i.e., only nodes themselves may be changed.)

**Example 1:**

**Input:** head = \[1,2,3,4\]

**Output:** \[2,1,4,3\]

**Explanation:**

![](https://assets.leetcode.com/uploads/2020/10/03/swap_ex1.jpg)

**Example 2:**

**Input:** head = \[\]

**Output:** \[\]

**Example 3:**

**Input:** head = \[1\]

**Output:** \[1\]

**Example 4:**

**Input:** head = \[1,2,3\]

**Output:** \[2,1,3\]

**Constraints:**

*   The number of nodes in the list is in the range `[0, 100]`.
*   `0 <= Node.val <= 100`

## Code Template

### Go
```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func swapPairs(head *ListNode) *ListNode {
    
}
```

### TypeScript
```typescript
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     val: number
 *     next: ListNode | null
 *     constructor(val?: number, next?: ListNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.next = (next===undefined ? null : next)
 *     }
 * }
 */

function swapPairs(head: ListNode | null): ListNode | null {
    
};
```
