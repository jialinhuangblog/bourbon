---
id: 203
title: "Remove Linked List Elements"
slug: remove-linked-list-elements
difficulty: Easy
tags: [Linked List, Recursion]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Remove Linked List Elements

Given the `head` of a linked list and an integer `val`, remove all the nodes of the linked list that has `Node.val == val`, and return _the new head_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/06/removelinked-list.jpg)

**Input:** head = \[1,2,6,3,4,5,6\], val = 6
**Output:** \[1,2,3,4,5\]

**Example 2:**

**Input:** head = \[\], val = 1
**Output:** \[\]

**Example 3:**

**Input:** head = \[7,7,7,7\], val = 7
**Output:** \[\]

**Constraints:**

*   The number of nodes in the list is in the range `[0, 104]`.
*   `1 <= Node.val <= 50`
*   `0 <= val <= 50`

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
func removeElements(head *ListNode, val int) *ListNode {
    
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

function removeElements(head: ListNode | null, val: number): ListNode | null {
    
};
```
