---
id: 328
title: "Odd Even Linked List"
slug: odd-even-linked-list
difficulty: Medium
tags: [Linked List]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Odd Even Linked List

Given the `head` of a singly linked list, group all the nodes with odd indices together followed by the nodes with even indices, and return _the reordered list_.

The **first** node is considered **odd**, and the **second** node is **even**, and so on.

Note that the relative order inside both the even and odd groups should remain as it was in the input.

You must solve the problem in `O(1)` extra space complexity and `O(n)` time complexity.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/10/oddeven-linked-list.jpg)

**Input:** head = \[1,2,3,4,5\]
**Output:** \[1,3,5,2,4\]

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/03/10/oddeven2-linked-list.jpg)

**Input:** head = \[2,1,3,5,6,4,7\]
**Output:** \[2,3,6,7,1,5,4\]

**Constraints:**

*   The number of nodes in the linked list is in the range `[0, 104]`.
*   `-106 <= Node.val <= 106`

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
func oddEvenList(head *ListNode) *ListNode {
    
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

function oddEvenList(head: ListNode | null): ListNode | null {
    
};
```
