---
id: 143
title: "Reorder List"
slug: reorder-list
difficulty: Medium
tags: [Linked List, Two Pointers, Stack, Recursion]
neetcode150_category: Linked List
blind75_category: Linked List
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Reorder List

You are given the head of a singly linked-list. The list can be represented as:

L0 → L1 → … → Ln - 1 → Ln

_Reorder the list to be on the following form:_

L0 → Ln → L1 → Ln - 1 → L2 → Ln - 2 → …

You may not modify the values in the list's nodes. Only nodes themselves may be changed.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/04/reorder1linked-list.jpg)

**Input:** head = \[1,2,3,4\]
**Output:** \[1,4,2,3\]

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/03/09/reorder2-linked-list.jpg)

**Input:** head = \[1,2,3,4,5\]
**Output:** \[1,5,2,4,3\]

**Constraints:**

*   The number of nodes in the list is in the range `[1, 5 * 104]`.
*   `1 <= Node.val <= 1000`

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
func reorderList(head *ListNode)  {
    
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

/**
 Do not return anything, modify head in-place instead.
 */
function reorderList(head: ListNode | null): void {
    
};
```
