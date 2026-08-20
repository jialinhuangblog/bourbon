---
id: 206
title: "Reverse Linked List"
slug: reverse-linked-list
difficulty: Easy
tags: [Linked List, Recursion]
neetcode150_category: Linked List
blind75_category: Linked List
date_solved: 2026-03-28
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(1)
insight: "head.Next.Next = head; head.Next = nil"
"@local": [Linked List]
---

# Reverse Linked List

Given the `head` of a singly linked list, reverse the list, and return _the reversed list_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)

**Input:** head = \[1,2,3,4,5\]
**Output:** \[5,4,3,2,1\]

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)

**Input:** head = \[1,2\]
**Output:** \[2,1\]

**Example 3:**

**Input:** head = \[\]
**Output:** \[\]

**Constraints:**

*   The number of nodes in the list is the range `[0, 5000]`.
*   `-5000 <= Node.val <= 5000`

**Follow up:** A linked list can be reversed either iteratively or recursively. Could you implement both?

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
func reverseList(head *ListNode) *ListNode {
    
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

function reverseList(head: ListNode | null): ListNode | null {
    
};
```
