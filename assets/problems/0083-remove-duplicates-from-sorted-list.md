---
id: 83
title: "Remove Duplicates from Sorted List"
slug: remove-duplicates-from-sorted-list
difficulty: Easy
tags: [Linked List]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Remove Duplicates from Sorted List

Given the `head` of a sorted linked list, _delete all duplicates such that each element appears only once_. Return _the linked list **sorted** as well_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/04/list1.jpg)

**Input:** head = \[1,1,2\]
**Output:** \[1,2\]

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/01/04/list2.jpg)

**Input:** head = \[1,1,2,3,3\]
**Output:** \[1,2,3\]

**Constraints:**

*   The number of nodes in the list is in the range `[0, 300]`.
*   `-100 <= Node.val <= 100`
*   The list is guaranteed to be **sorted** in ascending order.

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
func deleteDuplicates(head *ListNode) *ListNode {
    
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

function deleteDuplicates(head: ListNode | null): ListNode | null {
    
};
```
