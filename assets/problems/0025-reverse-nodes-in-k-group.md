---
id: 25
title: "Reverse Nodes in k-Group"
slug: reverse-nodes-in-k-group
difficulty: Hard
tags: [Linked List, Recursion]
neetcode150_category: Linked List
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Reverse Nodes in k-Group

Given the `head` of a linked list, reverse the nodes of the list `k` at a time, and return _the modified list_.

`k` is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of `k` then left-out nodes, in the end, should remain as it is.

You may not alter the values in the list's nodes, only nodes themselves may be changed.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex1.jpg)

**Input:** head = \[1,2,3,4,5\], k = 2
**Output:** \[2,1,4,3,5\]

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex2.jpg)

**Input:** head = \[1,2,3,4,5\], k = 3
**Output:** \[3,2,1,4,5\]

**Constraints:**

*   The number of nodes in the list is `n`.
*   `1 <= k <= n <= 5000`
*   `0 <= Node.val <= 1000`

**Follow-up:** Can you solve the problem in `O(1)` extra memory space?

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
func reverseKGroup(head *ListNode, k int) *ListNode {
    
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

function reverseKGroup(head: ListNode | null, k: number): ListNode | null {
    
};
```
