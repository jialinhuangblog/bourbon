---
id: 19
title: "Remove Nth Node From End of List"
slug: remove-nth-node-from-end-of-list
difficulty: Medium
tags: [Linked List, Two Pointers]
neetcode150_category: Linked List
blind75_category: Linked List
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Remove Nth Node From End of List

Given the `head` of a linked list, remove the `nth` node from the end of the list and return its head.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

**Input:** head = \[1,2,3,4,5\], n = 2
**Output:** \[1,2,3,5\]

**Example 2:**

**Input:** head = \[1\], n = 1
**Output:** \[\]

**Example 3:**

**Input:** head = \[1,2\], n = 1
**Output:** \[1\]

**Constraints:**

*   The number of nodes in the list is `sz`.
*   `1 <= sz <= 30`
*   `0 <= Node.val <= 100`
*   `1 <= n <= sz`

**Follow up:** Could you do this in one pass?

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
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    
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

function removeNthFromEnd(head: ListNode | null, n: number): ListNode | null {
    
};
```
